# JS代码越来越难读了
**对一个值执行连续操作**  
在 JavaScript 中对一个值执行连续操作（例如函数调用）时，目前有两种基本方式：  
- 将值作为参数传递给具体操作（如果有多个操作，则嵌套操作），例如：three(two(one(value)))；
- 将函数作为值上的方法调用（如果有多个方法，则为链式调用），例如：value.one().two().three()。

在 2020 年 JS 状态调查中，“你认为 JavaScript 目前缺少什么？“ 问题中，希望拥有管道操作符 答案排行第四名。  
如果是嵌套写法的话，简单的嵌套还好，但是当嵌套变得很深的时候就有点难以阅读了。嵌套的执行流程是从右到左移动的，而不是我们正常阅读代码从左到右的方向。另外，我们在很多括号之间找到一个位置添加一些参数也比较困难。比如下面的代码：  
``` 
console.log(
  chalk.dim(
    `$ ${Object.keys(envars)
      .map(envar =>
        `${envar}=${envars[envar]}`)
      .join(' ')
    }`,
    'node',
    args.join(' ')));
```
对于链式调用，只有我们把方法指定为值的实例方法时才能用，这让它具有很大的局限性。当然，如果你的库设计的很好（比如 jQuery） 还是挺好用的。  
**管道式编程**  
Unix 操作系统有一个管道机制，可以把前一个操作的值传给后一个操作。这个机制非常有用，使得简单的操作可以组合成为复杂的操作。许多语言都有管道的实现，举个简单的例子：  
``` 
function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}
function hello (str) {
  return str + ' Hello!';
}
```
两个简单的函数，想要嵌套执行，传统写法和管道写法分别如下：  
``` 
/ 传统的写法
exclaim(hello('conardli'))
// "Conardli Hello!"

// 管道的写法
'conardli'
  |> capitalize
  |> hello
// "Conardli Hello!"
```
**两个互相竞争的提案**  
在 ES 中有两个相互竞争的提案：  
Microsoft 提出的 F# ：是一种函数式编程语言，其核心基于 OCaml，这个运算符可以很方便的写出柯里化风格的代码。  
Meta 提出的 Hack：大致是 PHP 的静态类型版本。这个管道运算符专注于柯里化函数以外的语言特性。  
目前来看，Meta 提出的 Hack 应该更收社区的欢迎，Microsoft 提出的 F# 已经多次被 TC39 打回去了。  
**Hack 管道运算符**  
下面是一个 Hack 管道运算符 |> 的简单示例：  
``` 
ConardLi' |> console.log(%)  // ConardLi
```
管道运算符 |> 的左侧是一个表达式，它被计算并成为特殊变量 % 的值。我们可以在右侧使用该变量。返回右侧的执行结果。前面的例子等价于：  
``` 
console.log('ConardLi') // ConardLi
```
一些和其他写法配合的例子：  
``` 
value |> someFunction(1, %, 3) // function calls
value |> %.someMethod() // method call
value |> % + 1 // operator
value |> [%, 'b', 'c'] // Array literal
value |> {someProp: %} // object literal
value |> await % // awaiting a Promise
value |> (yield %) // yielding a generator value
```
更复杂点的例子，一个嵌套函数调用：  
``` 
const y = h(g(f(x)));
```
Hack pipe 操作符可以让我们更好地表达这段代码的意思：  
``` 
const y = x |> f(%) |> g(%) |> h(%);
```
**F# 管道运算符**  


参考:  
[JS 代码越来越难读了 ...](https://mp.weixin.qq.com/s/Ng-38P0jvvWbsPSwAjrJdA)
