# 探索在 JS 中，为什么要在函数前面加！
**简介**  
函数的声明方式有这两种  
``` 
function msg(){alert('msg');}//声明式定义函数

var msg = function(){alert('msg');}//函数赋值表达式定义函数
```
其实还有第三种声明方式，Function构造函数  
``` 
var msg = new function(msg) {
  alert('msg')
}
```
等同于  
``` 
function msg(msg) {
  alert('msg')
}
```
函数的调用方式通常是方法名(),但是，如果我们尝试为一个“定义函数”末尾加上()，解析器是无法理解的。  
``` 
function msg(){
  alert('message');
}();//解析器是无法理解的
```
定义函数的调用方式应该是 print(); 那为什么将函数体部分用()包裹起来就可以了呢？  
原来，使用括号包裹定义函数体，解析器将会以函数表达式的方式去调用定义函数。 也就是说，任何能将函数变成一个函数表达式的作法，都可以使解析器正确的调用定义函数。而 ! 就是其中一个，而 + \- || ~ 都有这样的功能。  
但是请注意如果用括号包裹函数体，然后立即执行。这种方式只适用一次性调用该函数，涉及到了一个作用域问题，当你想复用该函数的时候，会出现函数未定义问题  
可如果你想复用该函数的话，就可按先声明函数，然后再调用函数，在同一个父级作用域下，可以复用该函数，如下：  
``` 
var msg = function(msg) {}
msg();
```
## 自执行匿名函数：
在很多js代码中我们常常会看见这样一种写法：  
``` 
(function( window, undefined ) {
    // code
})(window);
```
这种写法我们称之为自执行匿名函数。正如它的名字一样，它是自己执行自己的，前一个括号是一个匿名函数，后一个括号代表立即执行  
前面也提到 + \- || ~这些运算符也同样有这样的功能  
``` 
(function () { /* code */ } ()); 
!function () { /* code */ } ();  
~function () { /* code */ } (); 
-function () { /* code */ } ();
+function () { /* code */ } ();
```
1. ( ) 没什么实际意义，不操作返回值
2. ! 对返回值的真假取反
3. 对返回值进行按位取反（所有正整数的按位取反是其本身+1的负数，所有负整数的按位取反是其本身+1的绝对值，零的按位取反是 -1。其中，按位取反也会对返回值进行强制转换，将字符串5转化为数字5，然后再按位取反。false被转化为0，true会被转化为1。其他非数字或不能转化为数字类型的返回值，统一当做0处理）
4.  ~ +、- 是对返回值进行数学运算 ( 可见返回值不是数字类型的时候 +、- 会将返回值进行强制转换,字符串强制转换后为NaN)

**IIFE（Imdiately Invoked Function Expression 立即执行的函数表达式）**  
``` 
function(){
    alert('IIFE');
}
```
把这个代码放在console中执行会报没有函数名错误  
因为这是一个匿名函数，要想让它正常运行就必须给个函数名，然后通过函数名调用。  
其实在匿名函数前面加上这些符号后，就把一个函数声明语句变成了一个函数表达式，是表达式就会在script标签中自动执行。  
所以现在很多对代码压缩和编译后，导出的js文件通常如下：  
``` 
(function(e,t){"use strict";function n(e){var t=e.length,n=st.type(e);return st.isWindow(e)?!1:1===e.nodeType&&t?!0:"array"===n||"function"!==n&&(0===t||"number"==typeof t&&t>0&&t-1 in e)}function r(e){var t=Tt[e]={};return st.each(e.match(lt)||[],function(e,n){t[n]=!0}),t}function i(e,n,r,i){if(st.acceptData(e)){var o,a,s=st.expando,u="string"==typeof n,l=e.nodeType,c=l?st.cache:e,f=l?e[s]:e[s]&&s;if(f&&c[f]&&(i||c[f].data)||!u||r!==t)return f||(l?e[s]=f=K.pop()||st.guid++:f=s),c[f]||(c[f]={},l||(c[f].toJSON=st.noop)),("object"==typeof n||"function"==typeof n)&&(i?c[f]=st.extend(c[f],n):c[f].data=st.extend(c[f].data,n)),o=c[f],i||(o.data||(o.data={}),o=o.data),r!==t&&(o[st.camelCase(n)]=r),u?(a=o[n],null==a&&(a=o[st.camelCase(n)])):a=o,a}}function o(e,t,n){if(st.acceptData(e)){var r,i,o,a=e.nodeType,u=a?st.cache:e,l=a?e[st.expando]:st.expando;if(u[l]){if(t&&(r=n?u[l]:u[l].data)){st.isArray(t)?t=t.concat(st.map(t,st.camelCase)):t in r?t=[t]:(t=st.camelCase(t),t=t in r?[t]:t.split(" "));for(i=0,o=t.length;o>i;i++)delete r[t[i]];if(!(n?s:st.isEmptyObject)(r))return}(n||(delete u[l].data,s(u[l])))&&(a?st.cleanData([e],!0):st.support.deleteExpando||u!=u.window?delete u[l]:u[l]=null)}}}function a(e,n,r){if(r===t&&1===e.nodeType){var i="data-"+n.replace(Nt,"-$1").toLowerCase();if(r=e.getAttribute(i),"string"==typeof r){try{r="true"===r?!0:"false"===r?!1:"null"===r?null:+r+""===r?+r:wt.test(r)?st.parseJSON(r):r}catch(o){}st.data(e,n,r)}else r=t}return r}function s(e){var t;for(t in e)if(("data"!==t||!st.isEmptyObject(e[t]))&&"toJSON"!==t)return!1;return!0}function u(){return!0}function l(){return!1}function c(e,t)
```
**解析器**  
程序在运行之前需要经过编译或解释的过程，把源程序翻译成为字节码，但是在翻译之前，需要把字符串形式的程序源码解析为语法树或者抽象语法树等数据结构，这就需要用到解析器  

## 那么什么是解析器？
析器（Parser），一般是指把某种格式的文本（字符串）转换成某种数据结构的过程。最常见的解析器（Parser），是把程序文本转换成编译器内部的一种叫做抽象语法树（AST）的数据结构，此时也叫做语法分析器（Parser）。也有一些简单的解析器（Parser），用于处理CSV、JSON，XML之类的格式  
JS解析器在执行第一步预解析的时候，会从代码的开始搜索直到结尾，只去查找var、function和参数等内容。一般把第一步称之为“JavaScript的预解析”。而且，当找到这些内容时，所有的变量，在正式运行代码之前，都提前赋了一个值：未定义；所有的函数，在正式运行代码之前，都是整个函数块。让解析器识别到是一个表达式，那就得加上特殊符号来让其解析器识别出来，比如刚才提到的特殊运算符。  
解析过程大致如下：
1. “找一些东西”: var、 function、 参数；(也被称之为预解析)  
   备注：如果遇到重名分为以下两种情况：遇到变量和函数重名了，只留下函数；遇到函数重名了，根据代码的上下文顺序，留下最后一个。
2. 逐行解读代码
   备注：表达式可以修改预解析的值 （可以自行查阅文档，这就是后面说到的内容）    

## 函数调用
函数声明加一个()就可以调用函数了  
``` 
function msg(){
    alert('IIFE');
}()
```
按上面在console中执行发现出错了,因为这样的代码混淆了函数声明和函数调用，以这种方式声明的函数 `msg`，就应该以 `msg()` 的方式调用。  
若改成(function msg())()就是这样的一个结构体：（函数体)(IIFE)，能被Javascript的解析器识别并正常执行  
从Js解析器的预解析过程了解到：  
解析器都能识别一种模式:使用括号封装函数。对于解析器来说，这几乎总是一个积极的信号，即函数需要立即执行。如果解析器看到一个左括号，紧接着是一个函数声明，它将立即解析这个函数。可以通过显式地声明立即执行的函数来帮助解析器加快解析速度  
那么也就是说，括号的作用，就是将一个函数声明，让解析器识别为一个表达式，最后由程序执行这个函数  
任何消除函数声明和函数表达式间歧义的方法，都可以被Javascript解析器正确识别。赋值，逻辑，甚至是逗号，各种操作符，只要是解析器支持且用来识别的特殊符号都可以用作消除歧义的方式方法，而!function() 和（function()）, 都是其中转换成表达式的一种方式。  


原文:  
[探索在 JS 中，为什么要在函数前面加！](https://mp.weixin.qq.com/s/3bhpz6lZI85miNcvxr_vzw)
