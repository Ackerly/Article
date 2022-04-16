# 了解SourceMap
>  我们项目的代码在经过编译打包后，会将开发时多个文件的代码合并到同一份文件中，而且还会经过各种压缩，合并，代码丑化等等操作，转换完最终生成的代码才会用于线上环境，所以我们线上实际运行的代码跟我们开发时的代码是有非常大的不同，如果此时出现了bug，那么我们只能定位到转换后代码的位置，但此时的代码已经面目全非了

虽然这种代码对计算机非常友好，但是我们debug将会变得很困难，这时候就需要sourcemap了

**什么是SourceMap**  
Sourcemap 就是一个信息文件，它里面存储着代码转换前后的对应位置信息，也就是转换压缩后的代码所对应的转换前的源代码位置，是源代码和生产代码的映射， Sourcemap 解决了在打包过程中，代码经过压缩，去空格以及 babel 编译转化后，由于代码之间差异性过大，debug 困难的问题  
项目在开发完进行build后，在打包文件夹里除了有js，css，图片等资源，一定还见过 .js.map文件，这种就是sourcemap文件  
点开一个打包后的js文件，拉到最后一行，可以看到 //# sourceMappingURL=main.js.map  
有了这行，就可以启用sourcemap，这个sourceMappingURL就是标记了该文件的sourcemap的地址，这个sourcemap文件可以放在本地，也可以放在网络上  
**SourceMap的生成**  
生成的方法有很多，而且有很多前端的工具都支持，如webpack，uglifyjs，gulp等  
**SourceMap的属性**  
```
const value = 123;
console.log(value);
```
用webpack打包后的代码
```
console.log(123);
```
生成的sourcemap文件  
```
{
   version : 3,
   file :  bundle.js ,
   mappings :  AACAA,QAAQC,IADM ,
   sources : [
     webpack://studysourcemap/./test.js 
  ],
   sourcesContent : [
     const value = 123;\nconsole.log(value); 
  ],
   names : [
     console ,
     log 
  ],
   sourceRoot :   
}
```
每个属性的含义如下:  
- version：遵循的是哪一个sourcemap版本的规范（下面会浅提一下）
- sources：转换前的源文件url数组（数组是因为存在多个文件合并的情况）
- names：在mappings中引用的标识符数组（可以理解为转换前代码的所有变量名和属性名）
- sourceRoot：源文件的根路径
- sourcesContent：转换前源文件的原始内容，也是一个数组
- mappings：记录源码和编译后代码的位置信息的base64 VLQ 字符串，是最重要的内容
- file：生成的与该sourcemap文件关联的文件名，也就是打包编译后的文件名

**SourceMap的版本**  
- 2009年，google介绍他的一个编译器Cloure Compiler时，也顺便推出了一个调试插件Closure Inspector，可以方便调试编译后的代码，这个就是 sourcemap 的雏形
- 2010年，Closure Compiler Source Map 2.0中，共同制定了一些标准，已决定使用base64编码，但是生成的map文件要比现在大很多
- 2011年，第三代出炉， Source Map Revision 3 Proposal，也就是我们现在用的 sourcemap 的版本，这也就是为什么我们上面map文件的 version=3 了，这一版对算法进行了优化，大大缩小了map文件的体积

第一版生成的map文件大概有转化后文件的10倍大，第二版则将体积减少了20%～30%，第三版又在v2的基础上体积减少了一半  
正是因为有了第三代 Source Map Revision 3 Proposal 这个标准，不同的打包工具和浏览器才能使用sourcemap，github上的一个根据这个标准生成sourcemap的库 https://github.com/mozilla/source-map  

**SourceMap的原理**  
这里主要关注mappings和names属性，mappings属性是一个很长的字符串，它分成三个部分  
1. 分号(;)，表示行对应，生成的文件的每一行用分号(;)分隔，一个分号代表转换后源码的一行
2. 逗号(,)，位置对应，每一段用逗号(,)分隔，一个逗号对应转换后源码的一个位置
3. 英文字母，每一段由1，4或5块可变长度的字段组成，记录原始代码的位置信息

举一个简单的例子，假设有如下的mappings属性  
```
mappings :  AACAA;QAAQC,IADM ,
```
有一个分号，说明有两行代码，分号前 AACAA 是第一行，后面 QAAQC,IADM 是第二行  
第二行有一个逗号，说明这一行分为两段，QAAQC 和 IADM  
分号跟逗号大家应该都没什么疑问，主要就是英文字母这一块的意义位置对应的原理,每一段最多有5个部分:
- 第一部分，表示这个位置在（转换后的代码）的第几列
- 第二部分，表示这个位置属于sources属性中的哪一个文件
- 第三部分，表示这个位置属于转换前代码的第几行
- 第四部分，表示这个位置属于转换前代码的第几列
- 第五部分，表示这个位置属于names属性中的哪一个变量

假设文件a.js有一行代码 I Love SourceMap，最终打包后输出的文件为bundle.js，内容为 Javascript is awesome  
那么怎么表示映射关系  
以 Love 为例，它原始的位置为(0,2)，输出后是awesome，位置为(0,14)，那么我们可以这样来映射  

|  输出后的单词   | 映射关系  |
|  ----  | ----  |
| Javascript  | 0｜0｜a.js｜0｜7｜SourceMap |
| is  | 0｜11｜a.js｜0｜0｜I |
| awesome  | 0｜14｜a.js｜0｜2｜Love |

可以优化一下，把a.js和最后面的单词提出来各放到一个数组里，用sources记录所有的原始文件名，names记录原始文件中的所有单词，然后用下标表示他们  
很多时候，我们输出的文件其实是只有一行的，所以可以把输出文件的行号省略掉  
考虑到，如果文件特别大的话，那么行列的数值可能会特别大，所以可以考虑用相对位置来代替绝对位置来表示，只用绝对位置表示第一个单词的位置，后面的都使用相对前一个单词的位置  

|  原始单词   | 输入位置  |输出单词  |输出位置  |映射  |
|  ----  | ----  | ----  | ----  |----  |
| I  | (0,0) 绝对位置 | is  | (0,11)绝对位置|11｜0｜0｜0｜0|
| Love  | (0,2) 相对I的位置 | awesome  | (0,3)相对is的位置  |3｜0｜0｜2｜1 |
| SourceMap  | (0,5) 相对Love的位置 | Javascript  | (0,-14) 相对awesome的位置  -14｜0｜0｜5｜2  |

可以得到这么一个初步的map文件
```
names: ['I', 'Love', 'SourceMap'],
sources: ['a.js'],
mappings: [11｜0｜0｜0｜0, 3｜0｜0｜2｜1, -14｜0｜0｜5｜2]
```
但是mappings这里十分难看，而且还需要用｜来分隔，多占一个位置，用 vlq 编码就可以解决分隔数字的问题，他的核心思路是在连续的数字上做标记，我们先来理解一下，拿上面 mappings 属性的第一个为例，去掉｜，然后在连续的字符上加上一个标记  
```
110000
```
从左往右开始读取，数字1有标记，说明还有连续，再取下一个，是1，这个1没被标记，第一个数结束，所以第一个数是11  
继续往下，0没被标记，说明是一个完整的数字，第二个数就是0  
依此类推，最终就能得到11，0，0，0，0  
而 vlq 利用6位二进制数进行存储，其中第一位就表示是否连续的标识位，最后一位表示正数还是负数（0正数，1负数） ，中间只有4位，因此一个单元表示的范围为[-15,15]，如果超过了就要用连续标识位了  
第三步（按...5554分割），最右边4位是因为他还需要额外多表示一位符号位，其余的都可以用5位来表示数值  
倒数第二步倒顺序，是因为VLQ表示数据字节组的顺序是倒过来的  
最终我们可以得到他们的 vlq 编码  

|  十进制数值   | vlq编码  | vlq编码每一段对应的数值  |
|  ----  | ----  | ----  |
| 5  | 001010 | 10  |
| -19  | 100111 000001 | 39和1 |

就可以得到5和-19的base64 vlq编码了，因为5的 vlq 编码数值是10,同理-19可以得到n和B，最终能得到5和-19的 base64 vlq 编码分别是K和nB
这里有一个网站可以自己转换验证一下https://www.murzwin.com/base64vlq.html  
回过头为最开始那个简单的js文件手动生成一下map文件来验证一下  
```
const value = 123;
console.log(value);
```
打包后的代码
```
console.log(123);
//# sourceMappingURL=bundle.js.map
```
sources和names是可以先确定好的  
```
{
     sources : [ a.js ],
     names : [ console ,  log ],
}
```
得到 base64 vlq 编码
|    | 原始位置  |  输出位置   | sources索引  |  names索引   | 映射  |  每一部分的vlq编码   | base64 vlq编码  |
|  ----  | ----  | ----  | ----  |  ----  | ----  | ----  | ----  |
| console  | (1,0)绝对 | (0,0)绝对  | 0 | 0  | 0 | 0  | 1 |
| log  | (0,8)相对 |(0,8)相对  | 0 | 1  | 8 | 0  | 0 |
| 123  | (-1,6)相对 |(0,4)相对  | 0 | 无  | 4 | 0  | -1 |

得到最终的map文件  
```
{
     sources : [ a.js ],
     names : [  console ,  log ],
     mappings :  AACAA,QAAQC,IADM ,
    // ...其他的
}
```

参考：  
[来聊聊SourceMap](https://mp.weixin.qq.com/s/ELIzEaHoo8gpcICLYEj-4w)