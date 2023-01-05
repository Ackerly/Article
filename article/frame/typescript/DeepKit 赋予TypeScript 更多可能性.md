# DeepKit —— 赋予 TypeScript 更多可能性
Deepkit 的第三方库，可以将 TypeScript 的类型信息保留到运行时进行消费。  
## TypeScript 带来的
传统开发上，Javascript 基本没有提供任何类型保护，所有的类型错误都需要在运行时才能发现，而TypeScript 为开发者提供了一套静态类型检查的方案，它提倡开发者在源码中主动声明类型信息，并与对应的变量和操作相匹配，并在编译阶段进行检查，类型相关的错误在编译时就暴露出来，一方面使代码更规范了，一方面也极大程度地规避了许多代码错误，提高了代码的健壮性。  
TypeScirpt 拥有完备的类型系统。但很可惜，它在这方面的能力在运行时几乎完全不存在。TypeScript Compiler在编译源码时会删除类型信息，不对运行时造成任何开销。  
但其实在许多场景下，运行时的类型信息都是极具价值的！  

## 为什么需要运行时类型
**数据校验**  
数据校验并不是局限于传统前端所关注的表单校验，需要数据校验的场景数不胜数，比如：  
- 在编写服务的时候，若我们需要实现一个接口。对于我们来说，传入的参数是未知的，我们永远不知道业务方会给我传来什么奇奇怪怪的参数。如果我们不对参数进行校验的话，后面的代码逻辑随时可能崩溃。而参数校验自然就需要在运行时消费参数的类型定义信息
- 数据库，一张表中所有字段的类型都是有严格定义的，所以在数据写入数据库时，需要校验写入的数据是否符合字段的类型定义，这也需要运行时的类型信息

**序列化与反序列化**  
序列化是将数据类型转换为适合传输或存储的格式的过程。反序列化是撤消此操作的过程，这个过程需要保证是无损的。对于前端开发者来说，接触的最多的应该就是 JSON.parse() 和 JSON.stringify() 这两个方法。在简单场景下，用这两个方法做序列化和反序列化可能没有问题，但是在复杂场景中就不一定了，因为这两个方法并不能保证数据是无损的。  
例如下面这个场景  
``` 
const date = new Date();
const dateString = JSON.stringify(date);//"2022-11-02T17:49:03.240Z"
const dateJson = JSON.parse(dateString);//"2022-11-02T17:49:03.240Z"
```
对于日期类型的数据，先用 JSON.stringify(date) 将其序列化成了适合传输的格式，再用JSON.parse(dateString) 反序列化，发现日期这个类型在过程中已经丢失，最后反序列化的结果为一个字符串，这显然是不符合预期的。因此，在序列化和反序列化的过程中，类型信息也十分重要。  
而 DeepKit 使将 TypeScript 类型保留到运行时成为现实。  

## 快速开始
### 前置  
使用 DeepKit 需要安装两个包：
- @deepkit/type：提供运行时可以使用的方法
- @deepkit/type-compiler：类型编译器，介入TypeScript 编译流程，保留类型信息。可以放在 package.json 的devDependencies中，因为这个类型编译器只需要编译阶段使用。

``` 
npm install --save @deepkit/type
npm install --save-dev @deepkit/type-compiler
```
然后需要在 tsconfig.json 中配置 "reflection": true 。如果需要使用装饰器，还需要加入"experimentalDecorators": true 参数  
``` 
// tsconfig.json
{
    "compilerOptions":{
        "module":"CommonJS",
        "target":"es6",
        "moduleResolution":"node",
        "experimentalDecorators":true
    },
    "reflection":true，
}
```
### 类型信息
DeepKit 定义了两种用于描述运行时的类型信息的数据结构，分别是类型对象和反射类。  
**类型对象**  
使用 typeOf 方法可以快速获取某个类型对应的类型对象。  
``` 
import { typeOf } from '@deepkit/type';
type Title<T> = T extends true ? string : number;

typeOf<Title<true>>();
//Type {kind: 5, typeName: 'Title', typeArguments: [{kind: 7}]}
```
可以看到一个类型对象的基本数据结构（当然，这还不是它的全貌）。详细的类型对象定义：https://github.com/deepkit/deepkit-framework/blob/feature/autotype/packages/type/src/reflection/type.ts#L21-L452
- kind：ReflectionKind，表示传入的类型。例子中对应 Title 的类型
- typeName：string，如果用到了类型别名，会返回这个字段，标识该类型
- typeArguments：当我们用了泛型时，传递进去的类型信息也会保留到类型对象中，会返回typeArguments 字段中记录的就是对应的类型信息

``` 
enum ReflectionKind {
  never,    //0
  any,     //1
  unknown, //2
  void,    //3
  object,  //4
  string,  //5
  number,  //6
  boolean, //7
  symbol,  //8
  bigint,  //9
  null,    //10
  undefined, //11

  //... and even more
}
```
**反射类**  
反射类多用于 类/接口/对象类型等等比较复杂的场景  
``` 
import { ReflectionClass } from '@deepkit/type';

interface User {
    id: number;
    username: string;
}

const reflection = ReflectionClass.from<User>();

reflection.getProperty('id'); //ReflectionProperty，记录id类型信息

reflection.getProperty('id').name; //'id'
reflection.getProperty('id').type; //{kind: ReflectionKind.number}
reflection.getProperty('id').isOptional(); //false
reflection.removeProperty('id');
reflection.getProperty('id');//Error: No property id found in User
```
对于复杂场景，我们可以通过 ReflectionClass.from 方法得到类型对应的放射类实例 ReflectionClass ，通过调用ReflectionClass中的方法可以获取更深层次的类型信息，也可以对类型信息做一些操作。  
**验证**  
需要数据验证的场景数不胜数，接口参数校验，数据库实现等都高度依赖数据校验，以此保证数据的安全性。  
DeepKit 提供了is和validate两个函数，用于校验一个值是否符合类型定义。
``` 
interface People {
  name: string
  age: number,
  info?: {
    address?: string,
    phone: number
  }
}

const peopleA = {
    name: 'Jack',
    age: 20,
}

const peopleB = {
    name: 'Peter',
    age: 18,
    info: {}
}

is<People>(peopleA)//true
is<People>(peopleB)//false
```
is 函数接收类型信息，并对参数中的数据进行校验，返回一个布尔值。如上面的例子，定义了一个 People 的 interface，并对 peopleA 和 peopleB 两个数据进行校验，可以看出 peopleA 是符合 People 的 定义的，所以返回is<People>(peopleA)会返回 true 。peopleB 中的 info 属性缺少了必填的 phone 字段，因此is<People>(peopleB) 会返回 false 。  
``` 
validate<People>(peopleA)//[]

validate<People>(peopleB)
// [{
//   path: 'info.phone',
//   code: 'type',
//   message: 'Not a number'
// }]
```
validate 函数和 is 函数的用法类似，区别是 validate 函数并不是返回一个布尔值 ，而是一个包含错误信息的数组。  
- path：错误路径，指向出错的具体属性
- code：错误类型，目前好像只有 type 一种。
- message：具体的错误信息。

**序列化**  
DeepKit 中 serialize/deserialize 两个方法，为用户提供了序列化/反序列化的能力  
``` 
import { serialize } from '@deepkit/type';

class MyModel {
  id: number = 0;
  created: Date = new Date;

  constructor(public name: string) {
  }
}

const model = new MyModel('Peter');

const jsonObject = serialize<MyModel>(model);
//{
//  id: 0,
//  created: 2022-11-02T17:49:03.240Z,
//  name: 'Peter'
//}
```
serialize 方法接收类型信息和需要序列化的数据，将数据序列化为符合类型定义的JSON对象。  
``` 
const myModel = deserialize<MyModel>({
    id: 5,
    created: 'Sat Oct 13 2018 14:17:35 GMT+0200',
    name: 'Peter',
});

is<Date>(myModel.created)// true
```
deserialize 方法接收类型信息和需要反序列化的数据，将数据反序列化为符合类型信息定义的数据。代码中的 created 字段会被反序列化为 Date 字段。  
**类型装饰器**  
> 装饰器本质上就是一个函数，可以在运行时对被装饰对象进行自定义的加工处理。

DeepKit 中提供了一套类型装饰器，这里的类型装饰器和 TypeScript 的装饰器并不相同，TypeScript 多用于对类的装饰，类型装饰器顾名思义是对类型的装饰。这些类型装饰器可以被当作一个正常的 TypeScript 类型使用。  
举一个简单的例子  
``` 
import { integer } from '@deepkit/type';

// case 1
type count = integer;
is<count>(1) // true
is<count>(1.1) // false
```
对定义 count 类型为 integer（整型），可以看到，1.1这个浮点数类型并没有通过校验。  
除此之外，DeepKit 还实现了如 PrimaryKey（主键）,maxLength/minLength（最小/最大长度）等功能的类型装饰器。我们可以把这些类型装饰器看作对于 TypeScript 类型的拓展，这些类型装饰器使 TypeScript 能够实现数据库级别的类型定义。也正是基于这套拓展后的运行时类型，验证和序列化可以有更多的约束，DeepKit 也实现了一套高性能的 ORM 。  

## More
@deepKit/type 给我们提供了一套运行时调用类型信息的方案。除此之外，DeepKit 的作者还基于类型信息和反射机制实现了更多的能力。
- 事件系统：@deepkit/events
- HTTP 库：@deepkit/http
- RPC服务：@deepkit/rpc
- 数据库ORM：@deepkit/orm
- 模版引擎：@deepkit/templat ，但与react不兼容
- 大一统框架：@deepkit/framework ，集成了上述能力的 node 框架

## 如何保证性能
**类型缓存**  
在未使用泛型的情况下，DeepKit 会对使用到的类型对象进行缓存  
``` 
//  case1
type MyType = string;

typeOf<MyType>() === typeOf<MyType>(); //true

//  case2
type MyType<T> = T;

typeOf<MyType<string>>() === typeOf<MyType<string>>();//false
```
可以看到，对于 case1 ，Mytype 对应的类型对象会被缓存，因此两次typeOf<MyType>() 的结果相等；但是对于泛型来说，我们无法确定传入的 T 具体是什么类型（理论上会有无限种），因此不会结果进行缓存，每次都会创建一个新的类型对象。   
**类型编译器**  
DeepKit 的核心原理是一个类型编译器，它会介入TypeScript 的编译流程，保留类型信息， 在这个过程中，Deepkit 的类型编译器会读取源码中的类型信息，产生相关的字节码（为了使它尽可能小），并将其插入 AST 中，将其转化为另一个包含这些字节码信息的 TypeScript AST。  
在运行时，DeepKit 会有一个迷你虚拟机，负责解析和执行这些字节码，最后会返回一个类型对象。  
在 DeepKit 官方提供的性能图中，可以看到 DeepKit 在数据读写上的表现是比较优秀的，这也归功于 DeepKit 提供的 运行时类型信息，这种预先知晓类型信息的机制可以使 序列化/验证等更加快速高效。  

## 总结
DeepKit 是市场上第一个在 JavaScript 运行时提供全套 TypeScript 类型的解决方案。它使前端/服务端可以共用一套TypeScript定义的数据模型，并且使用基于 TypeScript 实现的一套反射机制。  
但它依旧存在一些不足，比如 不支持外部类型，若代码中使用的类型信息来自第三方，且第三方库也没有经过 deepkit 的类型编译器的话，外部类型的类型信息在运行时也会全部丢失。  

原文:   
[DeepKit —— 赋予 TypeScript 更多可能性](https://mp.weixin.qq.com/s/mrm9zNq0q8cYJDXatHh6-w)
