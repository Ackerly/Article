# 解释器模式
给定一个语言，定义它的文法的一种表示，并定义一个解释器。这个解释器使用该表示来解释语言中的句子。  
## 举例
**SQL 解释器**  
SQL 是一种描述语言，所以也适用于解释器模式。不同的 SQL 方言有不同的语法，我们可以根据某种特定的 SQL 方言定制一套适配它的文法表达式，再利用 antlr 解析为一颗语法树。  
**代码编译器**  
程序语言也因为其天然是字符串的原因，和 SQL、日常语言都类似，需要一种模式解析后才能工作。不同的语言有不同的文法表示，我们只需要一个类似 antlr 的通用解释器，通过传入不同的文法表示，返回不同的对象结构。  
**自然语言处理**  
自然语言处理也是解释器的一种，首先自然语言处理一般只能处理日常语言的子集，因此先定义好支持的范围，再定义一套分词系统与文法表达式，并将分词后的结果传入灌入了此文法表达式的解释器，这样解释器可以返回结构化数据，根据结构化数据再进行分析与加工。  
## 解释
对于给定的语言，可以是 SQL、代码或自然语言，“定义它的文法的一种表示” 即文法可以有多种表示，只需定义一种。要注意的是，不同文法执行效率会有差异。  
“并定义一个解释器”，这个解释器就是类似 antlr 的东西，传给它一个文法表达式，就可以解析句子了。即：解释器(语言, 文法) = 抽象语法树。  
可以直接把文法定义耦合到解释器里，但这样做会导致语法复杂时，解释器难以维护。比较好的方式是定义一套与解释器解耦的文法表达式，通过预处理器最终生成解释器。  

![image](./../../assets/images/design%20patterns/Interpreter.png)  

Context 是其他上下文变量，AbstractExpression 是抽象语法表达式。  
TerminalExpression（终结符）与 NonterminalExpression(非终结符) 都继承于 AbstractExpression，终结符指的是没有后续展开的符号，非终结符相反，所以非终结符又指向了 AbstractExpression，如此递归。  
假设我们要实现以下文法：  
``` 
sum    ::= number + number
number ::= 1 | 2
```
表达一个最简单的加法文法，其中加法表达式 sum 和 number 都是非终结符,而 +、1、2 是终结符
``` 
// 抽象表达式
class AbstractExpression {
  interpret (text: string) {}
}

// 终结符表达式
class TerminalExpression extends AbstractExpression {
  constructor(values: string[]) {
    this.values = values
  }

  interpret(value: string) {
    // 值必须是其中之一
    return this.values.includes(value)
  }
}

// 非终结符表达式
class NonterminalExpression extends AbstractExpression {
  constructor(left: TerminalExpression, right: TerminalExpression) {
    this.left = left
    this.right = right
  }

  interpret(value: string) {
    if (value.indexOf("+") === -1) {
      // 必须包含 + 号
      return false
    }

    const splitValue = value.split('+')

    return this.left.interpret(splitValue[0]) 
      && this.right.interpret(splitValue[1])
  }
}

// 调用
const context = new Context()
const terminal = new TerminalExpression(["1", "2"])
const add = new AddExpression(terminal, terminal)

add.interpreter("1 + 1") // true
add.interpreter("1 + 2") // true
add.interpreter("1 + 3") // false
add.interpreter("2 - 1") // false
```
遇到非终结符则继续调用，只有终结符才能直接判断
## 缺点
上面的例子是比较低效场景，因为当语法复杂后，类的数目会明显增多，难以维护，此时需要用一个通用语法解析器

参考:  
[Interpreter（解释器模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/181.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Interpreter%20%E8%A7%A3%E9%87%8A%E5%99%A8%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
