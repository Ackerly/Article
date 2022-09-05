# 编写高质量JavaScript代码的基本要点
## 最小全局变量
全局变量的问题在于所有代码都共享了这些全局变量，他们在同一个全局命名空间，所以当两个不同定义同名但不同作用的全局变量的时候，命名冲突在所难免。
Web页面包含不是该页面开发者所写的代码的情况：
- 第三方JavaScript库
- 广告的脚本代码
- 第三方用户跟踪和分析脚本代码
- 不同类型的小组件，标志和按钮

JavaScript两个特征
- 不需要声明就可以使用变量
- JavaScript有隐含的全局概念，不声明的变量都会成为一个全局对象属性
``` 
function sum(x, y) {
   // 不推荐写法: 隐式全局变量 
   result = x + y;
   return result;
}
```
因为result没有声明，在调用函数后就多了一个全局命名空间，解决方发生是使用var,let,const声明变量。  
另外的一种创建的隐式变量的例子：
``` 
// 反例，勿使用 
function foo() {
   var a = b = 0;
   // ...
}
```
a是本地变量，b是全局变量，原因是从右到左进行赋值，先是b = 0，b是未声明的。这个表达式返回值是0，然后就是var a = 0，定义了局部变量a。
## var的副作用
隐式全局变量和明确定义的全局变量有些小的差异，就是能否使用delete操作符让变量未定义。
- 通过var 创建的全局变量（函数外）时不能被删除的
- 无var的隐式变量（无论是否在函数中创建）是能被删除的

隐式全局变量不是真正的全局变量，它是全局对象的属性。可以通过delete操作符删除，变量时不能的。
## 访问全局变量
浏览器中全局对象可有通过window属性在代码的任何位置访问。但在其他环境这个方便的属性可能是其他名字（甚至不可用）。如果需要在硬编码的window标识符下访问全局对象
``` 
var global = (function () {
   return this;
}());
```
这种方法可以随时获取全局变量，this总指向全局对象，但是不适合ECMAScript5严格模式。严格模式下必须采用不同的形式，比如开发一个JavaScript库，将代码包裹在一个即时函数中，然后从全局作用域中，传递一个引用指向this作为你即时函数的参数。
## 预解析
在函数任意位置声明多个var，会被提升到函数，这种即为变量提升，
``` 
// 反例
myname = "global"; // 全局变量
function func() {
    alert(myname); // "undefined"
    var myname = "local";
    alert(myname); // "local"
}
func();
```
等同于
``` 
myname = "global"; // global variable
function func() {
   var myname; // 等同于 -> var myname = undefined;
   alert(myname); // "undefined"
   myname = "local";
   alert(myname); // "local"}
func()
```
## for循环
``` 
// 次佳的循环
for (var i = 0; i < myarray.length; i++) {
   // 使用myarray[i]做点什么
}
```
这种形式的循环的不足在于每次循环的时候数组的长度都要去获取下。这回降低你的代码，尤其当myarray不是数组，而是一个HTMLCollection对象的时候。集合的麻烦在于它们实时查询基本文档（HTML页面）。这意味着每次你访问任何集合的长度，你要实时查询DOM，而DOM操作一般都是比较昂贵的。
``` 
for (var i = 0, max = myarray.length; i < max; i++) {
   // 使用myarray[i]做点什么
}
```
在这个循环过程中，只检索了一次长度值
``` 
var i, myarray = [];
for (i = myarray.length; i–-;) {
   // 使用myarray[i]做点什么
}
```
- 少了一个变量
- 向下数到0，和0做比较和数组长度或其他不是0的东西作比较更有效率
## for-in循环
for-in循环应该用在非数组对象的遍历上，使用for-in循环数组（因为JavaScript中数组也是对象），但这是不推荐的。因为如果数组对象已被自定义的功能增强，就可能发生逻辑错误。在for-in中，属性列表的顺序（序列）是不能保证的。
``` 
// 1.
// for-in 循环
for (var i in man) {
   if (man.hasOwnProperty(i)) { // 过滤
      console.log(i, ":", man[i]);
   }
}
/* 控制台显示结果
hands : 2
legs : 2
heads : 1
*/
// 2.
// 反面例子:
// for-in loop without checking hasOwnProperty()
for (var i in man) {
   console.log(i, ":", man[i]);
}
/*
控制台显示结果
hands : 2
legs : 2
heads : 1
clone: function()
*/
```
for-in如果不使用hasOwnProperty会遍历原型链的方法属性
## （不）扩展内置原型
增加内置的构造函数原型（如Object(), Array(), 或Function()）挺诱人的，但是这严重降低了可维护性，因为它让你的代码变得难以预测。  
不增加内置原型是最好的。你可以指定一个规则，
## switch模式(switch Pattern)
``` 
var inspect_me = 0,
    result = '';
switch (inspect_me) {
case 0:
   result = "zero";
   break;
case 1:
   result = "one";
   break;
default:
   result = "unknown";
}
```
- 每个case和switch对齐
- 每个case中代码缩进
- 每个case以break清除结束
- 避免贯穿（故意忽略break）。如果你非常确信贯穿是最好的方法，务必记录此情况，因为对于有些阅读人而言，它们可能看起来是错误的。
- 以default结束switch：确保总有健全的结果，即使无情况匹配。
## 避免隐式类型转换
JavaScript的变量在比较的时候会隐式类型转换。例如false == 0 或 "" == 0返回的结果是true。为避免引起混乱的隐含类型转换，在你比较值和表达式类型的时候始终使用===和!==操作符。
## 避免eval
此方法接受任意的字符串，并当作JavaScript代码来处理。。如果代码是在运行时动态生成，有一个更好的方式不使用eval而达到同样的目 标。例如，用方括号表示法来访问动态属性会更好更简单
``` 
// 反面示例
var property = "name";
alert(eval("obj." + property));

// 更好的
var property = "name";
alert(obj[property]);
```
使用eval()也带来了安全隐患，因为被执行的代码（例如从网络来）可能已被篡改。当处理Ajax请求得到的JSON 相应的时候,最好使用JavaScript内置方法来解析JSON相应，以确保安全和有效。   
new Function()构造就类似于eval(),如果你绝对必须使用eval()，你 可以考虑使用new Function()代替。有一个小的潜在好处，因为在新Function()中代码是在局部函数作用域中运行。var 定义的变量都不会自动变成全局变量。另一种方法来阻止自动全局变量是封装eval()调用到一个即时函数中。
``` 
console.log(typeof un);    // "undefined"
console.log(typeof deux); // "undefined"
console.log(typeof trois); // "undefined"

var jsstring = "var un = 1; console.log(un);";
eval(jsstring); // logs "1"

jsstring = "var deux = 2; console.log(deux);";
new Function(jsstring)(); // logs "2"

jsstring = "var trois = 3; console.log(trois);";
(function () {
   eval(jsstring);
}()); // logs "3"

console.log(typeof un); // number
console.log(typeof deux); // "undefined"
console.log(typeof trois); // "undefined"
```
un作为全局变量污染了命名空间。  
eval()和Function构造不同的是eval()可以干扰作用域链，而Function()更安分守己些。不管你在哪里执行 Function()，它只看到全局作用域。所以其能很好的避免本地变量污染。
``` 
(function () {
   var local = 1;
   eval("local = 3; console.log(local)"); // logs "3"
   console.log(local); // logs "3"
}());

(function () {
   var local = 1;
   Function("console.log(typeof local);")(); // logs undefined
}());
```
上面例子eval()可以访问和修改它外部作用域中的变量，这是 Function做不到。
## parseInt()下的数值转换
使用parseInt()你可以从字符串中获取数值，该方法接受另一个基数参数，这经常省略。当字符串以”0″开头的时候就有可能会出问题，在ECMAScript 3中，开头为”0″的字符串被当做8进制处理了，但这已在ECMAScript 5中改变了。
## 编码规范
建立和遵循编码规范是很重要的，这让你的代码保持一致性，可预测，更易于阅读和理解
## 缩进
代码没有缩进基本上就不能读了。唯一糟糕的事情就是不一致的缩进，因为它看上去像是遵循了规范，但是可能一路上伴随着混乱和惊奇。重要的是规范地使用缩进。
## 花括号
花括号（亦称大括号，下同）应总被使用，即使在它们为可选的时候。在in或是for中如果语句仅一条，花括号是不需要的，但是你还是应该总是使用它们，这会让代码更有持续性和易于更新。
``` 
// 糟糕的实例
for (var i = 0; i < 10; i += 1)
   alert(i);
```
继续增加代码
``` 
// 糟糕的实例
for (var i = 0; i < 10; i += 1)
   alert(i);
   alert(i + " is " + (i % 2 ? "odd" : "even"));
```
``` 
// 好的实例
for (var i = 0; i < 10; i += 1) {
   alert(i);
}
```
## 空格
空格的使用同样有助于改善代码的可读性和一致性。在写英文句子的时候，在逗号和句号后面会使用间隔。  
适合使用空格的地方：
- for循环分号分开后的的部分：如for (var i = 0; i < 10; i += 1) {...}
- for循环中初始化的多变量(i和max)：for (var i = 0, max = 10; i < max; i += 1) {...}
- 分隔数组项的逗号的后面：var a = [1, 2, 3];
- 对象属性逗号的后面以及分隔属性名和属性值的冒号的后面：var o = {a: 1, b: 2};
- 限定函数参数：myFunc(a, b, c)
- 函数声明的花括号的前面：function myFunc() {}
- 匿名函数表达式function的后面：var myFunc = function () {};
- +, -, *, =, <, >, <=, >=, ===, !==, &&, ||, +=等前后
## 命名规范
构造函数使用首字母大写
## 分隔单词(
当你的变量或是函数名有多个单词的时候，最好单词的分离遵循统一的规范，有一个常见的做法被称作“驼峰(Camel)命名法”，就是单词小写，每个单词的首字母大写。  
对于构造函数，可以使用大驼峰式命名法(upper camel case)，如MyConstructor()。对于函数和方法名称，你可以使用小驼峰式命名法(lower camel case)，像是myFunction(), calculateArea()和getFirstName()。  
变量不是函数的时候可以使用下划线连接，例如，first_name, favorite_bands,和old_company_name。
## 注释
注 释可以给你代码未来的阅读者以诸多提示，阅读者需要的是（不要读太多的东西）仅注释和函数属性名来理解你的代码。例如，当你有五六行程序执行特定的任务， 如果你提供了一行代码目的以及为什么在这里的描述的话，阅读者就可以直接跳过这段细节。没有硬性规定注释代码比，代码的某些部分（如正则表达式）可能注释 要比代码多。


原文: 
[编写高质量JavaScript代码的基本要点](https://www.cnblogs.com/TomXu/archive/2011/12/28/2286877.html)
