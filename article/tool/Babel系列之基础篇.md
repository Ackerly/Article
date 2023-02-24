# Babel系列之基础篇
**Babel 是什么**  
Babel 官网是这样定义的：Babel is a JavaScript compiler。Babel 是一套解决方案，主要用来把 ECMAScript 2015+ 的代码转化为浏览器或者其它环境支持的代码。它主要可以做以下事情：  
- 语法转换
- 为目标环境提供 Polyfill 解决方案
- 源码转换

**Babel 的使用**  
初始化一个基础项目并安装依赖。  
``` 
npm install --save-dev @babel/core @babel/cli @babel/preset-env
```

## Babel 原理
babel 作为一个编译器，主要做的工作内容如下：  
1. 解析源码，生成 AST
2. 对 AST 进行转换，生成新的 AST
3. 根据新的 AST 生成目标代码

**Parse（解析）阶段**  
Parse 阶段可以细分为两个阶段：词法分析（Lexical Analysis, LA）和语法分析（Syntactic Analysis, SA）  
_词法分析_  
词法分析是对代码进行分词，把代码分割成被称为 Tokens 的东西。Tokens 是一个数组，由一些代码的碎片组成，比如数字、标点符号、运算符号等等，例如这样：  
``` 
// 代码
const a = 1;

// Tokens https://esprima.org/demo/parse.html#
[
    {
        "type": "Keyword",
        "value": "const"
    },
    {
        "type": "Identifier",
        "value": "a"
    },
    {
        "type": "Punctuator",
        "value": "="
    },
    {
        "type": "Numeric",
        "value": "1"
    },
    {
        "type": "Punctuator",
        "value": ";"
    }
]
```
_语法分析_  
词法分析之后，代码就已经变成了一个 Tokens 数组，现在需要通过语法分析把 Tokens 转化为 AST。  
在 babel 中，以上工作是通过 @babel/parser 来完成的，它基于ESTree 规范，但也存在一些差异。最终生成的 AST 结构中有很多相似的元素，它们都有一个 type 属性（可以通过官网提供的说明文档来查看所有类型），这样的元素被称作节点。一个节点通常含有若干属性，可以用于描述 AST 的节点信息。  
_Transform（转换）阶段_  
转换阶段，Babel 对 AST 进行深度优先遍历，对于 AST 上的每一个分支 Babel 都会先向下遍历走到尽头，然后再向上遍历退出刚遍历过的节点，然后寻找下一个分支。在遍历的过程中，可以增删改这些节点，从而转换成实际需要的 AST。  
以上是 babel 转换阶段操作节点的思路，具体实现是：babel 维护一个称作 Visitor 的对象，这个对象定义了用于 AST 中获取具体节点的方法，如果匹配上一个 type，就会调用 visitor 里的方法  
一个简单的 Visitor 对象如下：  
``` 
const visitor = {
    FunctionDeclaration(path, state) {
        console.log('我是函数声明');
    }
};
```
在遍历 AST 的过程中，如果当前节点的类型匹配 visitor 中的类型，就会执行对应的方法。上面提到，遍历 AST 节点的时候会遍历两次（进入和退出），因此，上面的 Vistor 也可以这样写：  
``` 
const visitor = {
    FunctionDeclaration: {
        enter(path, state) {
            console.log('enter');
        },
        exit(path, state) {
            console.log('exit');
        }
    }
};
```
> Visitor 中的每个函数接收 2 个参数：path 和 state。  
> path：表示两个节点之间连接的对象，对象包含：当前节点、父级点、作用域等元信息，以及增删改查 AST 的 api。  
> state：遍历过程中 AST 节点之间传递数据的方式，插件可以从 state 中拿到 opts，也就是插件的配置项。

例如使用上面 visitor 遍历如下代码时：  
``` 
// 源码
function test() {
  console.log(1)
}
```

**Generator（生成）阶段**  


原文:  
[Babel 系列【基础篇】](https://mp.weixin.qq.com/s/GcozDbrrFmVqt0fjtjqn8g)
