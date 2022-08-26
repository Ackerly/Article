# 由esbuild JavaScript API 看跨语言调用
esbuild 是使用 Go 语言实现的打包工具，配置简单，编译速度快，被广泛应用。esbuild 提供了 JavaScript API，完全兼容 JavaScript 生态。  
例子  
``` 
import { transformSync } from 'esbuild';

 transformSync('let x: number = 1', {
   loader: 'ts',
 });
```
你也许会发出这样的疑问：esbuild 是 Go 语言实现的，但在 JavaScript 中，我们却可以把它当做一个普通的 npm 包调用？它是如何实现跨语言调用的？  

**进程间通信**  
vite 是 JavaScript 实现的构建工具，依赖了 esbuild。先随便新建个 vite 项目，然后执行 yarn run dev 启动 vite。使用 ps 命令找到当前进程 ID，再使用 pstree 来观察进程之间的关系。  

初步实现  
有如下两种使用方式：  
``` 
# 1
 cat src/main.ts | esbuild --loader=ts

 # 2
 esbuild src/main.ts
```
执行命令后，会在控制台输出编译之后的代码。将上述 shell 改成等效的 Node 代码  
``` 
import { spawn } from 'child_process';

 const esbuild = spawn('./node_modules/esbuild-darwin-64/bin/esbuild', [
   'package.json',
 ]);

 esbuild.stdout.on('data', (chunk) => {
   console.log(chunk.toString());
 });
```
当前的实现方式缺点很明显：
- 一次只能编译一个文件
- 编译多个文件的话，频繁创建进程

改进  
更好的方式是 esbuild 在启动之后一直保持运行，向该进程 stdin 写入数据，同时对进程的 stdout 进行解码  
esbuild 启动方式是这样的：  
``` 
esbuild --service=0.13.15 --ping
```
加上 --ping 参数可以让 esbuild 进程保持运行不退出，从而不断地接受来自 stdin 的请求  
**通信协议**  
模型构建  
借助进程的 stdin/stdout 可以实现通信。在此之上，还需要定义传输的数据。esbuild 定义 Packet 结构来描述传输的数据：  
``` 
export interface Packet {
   id: number;
   isRequest: boolean; // 一般都为 true
   value: Value;
 }
```
其中 value 字段表示协议的内容。比如 TransformRequest 表示文件转换请求，TransformResponse 则代表了转换请求的响应。  
``` 
type Value = TransformRequest | TransformResponse | Others;

 export interface TransformRequest {
   command: 'transform';
   flags: string[];
   input: string;
   // ...
 }

 export interface TransformResponse {
   code: string;
   // ...
 }
```
序列化与反序列化  
序列化是将对象的状态信息转换成可取用格式，以留待后续其他的程序能读取出来，反序列化对象的状态，重新创建该对象。接触最多是 JSON 的序列化与反序列化。类似的，需要将 Packet 对象进行序列化与反序列化。  
esbuild 通信的消息采用定长编码。如下图所示，消息的前 4 个字节为消息体长度，后面跟着消息的内容。  
``` 
+---+---+---+---+---+---+---+---+---+----+
 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | .. |
 +---+---+---+---+---+---+---+---+---+----+
 |     length    |          body          |
 +---------------+------------------------+
```
不同的数据，对应着不同的转换规则。可以将数据分为两类：  
- 定长：boolean/uint32/null
- 不定长数据：array/string/object

数据生成的字节编码格式如下。用 1 字节表明数据的类型，用可选的 4 个字节表明数据的长度，最后一部分就是数据的实际内容。  
``` 
 +------+---+---+---+---+---+---+---+----+
 |   1  | 2 | 3 | 4 | 5 | 6 | 7 | 8 | .. |
 +------+---+---+---+---+---+---+---+----+
 | type |    <length>   |     content    |
 +------+---------------+----------------+

```
还有一点需要注意的是字节序，esbuild 协议采用的是小端序。下方的例子可以帮助我们理解字节序。  
``` 
const buffer = Buffer.from([0x01, 0x02, 0x03, 0x04]);
 buffer.readUInt32LE(); // = 67305985
 buffer.readUInt32BE(); // = 16909060 = 0x01020304
```
以字符串 abc 为例，序列化之后如下：  
``` 
+-----+----+----+----+----+----+----+----+
 |  1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  |
 +-----+----+----+----+----+----+----+----+
 |  03 | 03 | 00 | 00 | 00 | 61 | 62 | 63 |
 +-----+----+----+----+----+----+----+----+
 | str |     length: 3     | a  | b  | c  |
 +-----+-------------------+----+----+----+
```

## 实现
**编解码器 (Codec)**  
在实现编解码器前，先实现一个能自动扩容的 Buffer 来屏蔽对数据的读写细节。  
``` 
export class PacketBuffer {
   private buffer: Buffer;
   scale() {}
   writeUint32() {}
   readUint32() {}
 }
```
实现 encode 与 decode 方法  
``` 
export function encode(packet: Packet): Buffer {
   const buffer = new PacketBuffer();
   buffer.writeUint32(id);
   // ...
   return buffer.slice();
 }

 export function decode(buffer: PacketBuffer): Packet {
   // ...
   return packet;
 }
```
**流的读写**  
进程通信时，处理的数据都是面向流的，需要将 Packet 编码后写入流或者将流转换成 Packet。写的过程比较简单，读的过程稍微复杂一下，采用两个 Transform 流来处理。先使用 FixedLengthTransform 将流按照固定长度进行切分，接下来使用 PacketTransform 来完成字节流到 Packet 的转换。  
``` 
import { pipeline } from 'stream';

 // 将流按照包体的长度进行拆分
 export class FixedLengthTransform extends Transform {}

 // 将流转换成 Packet 对象
 export class PacketTransform extends Transform {}

 const transform = pipeline(
   esbuild.stdout,
   new FixedLengthTransform(),
   new PacketTransform(),
   handlerError(),
 );

 transform.on('data', (packet) => {});
```
**API 封装**  
把上面的过程组装在一起，就可以像 esbuild 这个 npm 包一样对外暴露 JavaScript API 了。用户按照自己的需要调用相应的函数即可。  
``` 
export class API {
   transformTs(code: string) {}
   transformJson(code: string) {}
   transform(code: string, option: Option) {}
 }
```

参考:    
[由 esbuild JavaScript API 看跨语言调用](https://mp.weixin.qq.com/s/ukZxF_W6dahisVW_Xf6-TA)
