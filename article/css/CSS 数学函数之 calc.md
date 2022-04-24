# CSS 数学函数之 calc
calc() 此 CSS函数允许在声明 CSS 属性值时执行一些计算，语法类似于：
```
{
    width: calc(100% - 80px);
}
``` 
一些需要注意的点：  
- + 和 - 运算符的两边必须要有空白字符。比如，calc(50% -8px) 会被解析成为一个无效的表达式，必须写成calc(8px + -50%)
- * 和 / 这两个运算符前后不需要空白字符，但如果考虑到统一性，仍然推荐加上空白符
- 用 0 作除数会使 HTML 解析器抛出异常
- 涉及自动布局和固定布局的表格中的表列、表列组、表行、表行组和表单元格的宽度和高度百分比的数学表达式，auto 可视为已指定
- calc() 函数支持嵌套，但支持的方式是：把被嵌套的 calc() 函数全当成普通的括号。（所以，函数内直接用括号就好了。）
- calc() 支持与 CSS 变量混合使用

一个最常见的例子，页面结构如下：  
```
<div class="g-container">
    <div class="g-content">Content</div>
    <div class="g-footer">Footer</div>
</div>
```
页面的 g-footer 高为 80px，我们希望不管页面多长，g-content 部分都可以占满剩余空间  
这种布局使用 flex 的弹性布局可以轻松实现，当然，也可以使用 calc() 实现：
```
.g-container {
    height: 100vh;
}
.g-content {
    height: calc(100vh - 80px);
}
.g-footer {
    height: 80px;
}
```
**Calc 中的加减法与乘除法的差异**  
calc() 中的加减法与乘除法的差异：
```
{
    font-size: calc(1rem + 10px);
    width: calc(100px + 10%);
}
```
加减法两边的操作数都是需要单位的，而乘除法，需要一个无单位数，仅仅表示一个倍率：  
```
{
    width: calc(100% / 7);
    animation-delay: calc(1s * 3);
}
```
**Calc 的嵌套**  
calc() 函数是可以嵌套使用的，像是这样：
```
{
  width: calc(100vw - calc(100% - 64px));
}
```
内部的 calc() 函数可以退化写成一个括号即可 ()，所以上述代码等价于：  
```
{
  width: calc(100vw - (100% - 64px));
}
```
嵌套内的 calc()，calc 几个函数字符可以省略  
**Calc 内不同单位的混合运算**  
calc() 支持不同单位的混合运算，对于长度，只要是属于长度相关的单位都可以进行混合运算，包含这些：  
- px
- %
- em
- rem
- in
- mm
- cm
- pt
- pc
- ex
- ch
- vh
- vw
- vmin
- vmax

运算肯定是消耗性能的，早年间，有这样一段 CSS 代码，可以直接让 Chrome 浏览器崩溃 Crash：  
```
<div></div>

// CSS
div {
  --initial-level-0: calc(1vh + 1% + 1px + 1em + 1vw + 1cm);

  --level-1: calc(var(--initial-level-0) + var(--initial-level-0));
  --level-2: calc(var(--level-1) + var(--level-1));
  --level-3: calc(var(--level-2) + var(--level-2));
  --level-4: calc(var(--level-3) + var(--level-3));
  --level-5: calc(var(--level-4) + var(--level-4));
  --level-6: calc(var(--level-5) + var(--level-5));
  --level-7: calc(var(--level-6) + var(--level-6));
  --level-8: calc(var(--level-7) + var(--level-7));
  --level-9: calc(var(--level-8) + var(--level-8));
  --level-10: calc(var(--level-9) + var(--level-9));
  --level-11: calc(var(--level-10) + var(--level-10));
  --level-12: calc(var(--level-11) + var(--level-11));
  --level-13: calc(var(--level-12) + var(--level-12));
  --level-14: calc(var(--level-13) + var(--level-13));
  --level-15: calc(var(--level-14) + var(--level-14));
  --level-16: calc(var(--level-15) + var(--level-15));
  --level-17: calc(var(--level-16) + var(--level-16));
  --level-18: calc(var(--level-17) + var(--level-17));
  --level-19: calc(var(--level-18) + var(--level-18));
  --level-20: calc(var(--level-19) + var(--level-19));
  --level-21: calc(var(--level-20) + var(--level-20));
  --level-22: calc(var(--level-21) + var(--level-21));
  --level-23: calc(var(--level-22) + var(--level-22));
  --level-24: calc(var(--level-23) + var(--level-23));
  --level-25: calc(var(--level-24) + var(--level-24));
  --level-26: calc(var(--level-25) + var(--level-25));
  --level-27: calc(var(--level-26) + var(--level-26));
  --level-28: calc(var(--level-27) + var(--level-27));
  --level-29: calc(var(--level-28) + var(--level-28));
  --level-30: calc(var(--level-29) + var(--level-29));

  --level-final: calc(var(--level-30) + 1px);

    border-width: var(--level-final);                                 
    border-style: solid;
}
```
从 --level-1 到 --level-30，每次的运算量都是成倍的增长，最终到 --level-final 变量，展开将有 2^30 = 1073741824 个 --initial-level-0 表达式的内容。  
并且，每个 --initial-level-0 表达式的内容 -- calc(1vh + 1% + 1px + 1em + 1vw + 1cm)，在浏览器解析的时候，也已经足够复杂。  
calc 是可以进行不同单位的混合运算的，另外一个就是注意具体使用的时候如果计算量巨大，可能会导致性能上较大的消耗。  
不要将长度单位和非长度单位混合使用  
**Calc 搭配 CSS 自定义变量使用**  
calc() 函数非常重要的一个特性就是能够搭配 CSS 自定义以及 CSS @property 变量一起使用  
```
:root {
    --width: 10px;
}
div {
    width: calc(var(--width));
}
```
**calc 搭配自定义变量时候的默认值**  
```
<div class="g-container">
    <div class="g-item" style="--delay: 0"></div>
    <div class="g-item" style="--delay: 0.6"></div>
    <div class="g-item"></div>
    <div class="g-item" style="--delay: 1.8"></div>
    <div class="g-item" style="--delay: 2.4"></div>
</div>

// CSS
{
    animation-delay: calc(var(--delay) * -1s);
}
```
由于 HTML 标签没有传入 --delay 的值，并且在 CSS 中向上查找也没找到对应的值，此时，animation-delay: calc(var(--delay) * -1s) 这一句其实是无效的，相当于 animation-delay: 0  
基于这种情况，可以利用 CSS 自定义变量 var() 的 fallback 机制：  
```
{
    // (--delay, 1) 中的 1 是个容错机制
    animation-delay: calc(var(--delay, 1) * -1s);
}
```
如果没有读取到任何 --delay 值，就会使用默认的 1 与 -1s 进行运算。  
**Calc 字符串拼接**  
很多人在使用 CSS 的时候，会尝试字符串的拼接，像是这样：
```
<div style="--url: 'bsBD1I.png'"></div>

// CSS
:root {
    --urlA: 'url(https://s1.ax1x.com/2022/03/07/';
    --urlB: ')';
}
div {
    width: 400px;
    height: 400px;
    background-image: calc(var(--urlA) + var(--url) + var(--urlB));
}
```
这里想利用 calc(var(--urlA) + var(--url) + var(--urlB)) 拼出完整的在 background-image 中可使用的 URL url(https://s1.ax1x.com/2022/03/07/bsBD1I.png)。
然而，这是不被允许的（无法实现的）。calc 的没有字符串拼接的能力。  
唯一可能完成字符串拼接的是在元素的伪元素的  content 属性中。但是也不是利用 calc。
来看这样一个例子，这是错误的：  
```
:root {
    --stringA: '123';
    --stringB: '456';
    --stringC: '789';
}

div::before {
    content: calc(var(--stringA) + var(--stringB) + var(--stringC));
}
```
此时，不需要 calc，直接使用自定义变量相加即可,正确的写法：
```
:root {
    --stringA: '123';
    --stringB: '456';
    --stringC: '789';
}
div::before {
    content: var(--stringA) + var(--stringB) + var(--stringC);
}
```

参考:  
[CSS 数学函数之 calc](https://mp.weixin.qq.com/s/BB_Sk03m8e17aQZrVfv7HA)