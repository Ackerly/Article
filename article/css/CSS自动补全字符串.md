# CSS自动补全字符串
很多时候都会碰到字符串补全的需求，典型的例子就时间或者日期中的补零操作，例如  
``` 
2021-12-31
2022-03-03
```  
通常的做法是  
``` 
if (num < 10) {
   num = '0' + num
 }
```
JS 中出现了原生的补全方法 padStart () 和 padEnd ()，如下  
``` 
'3'.padStart(2, '0')
 // 结果是 ’03‘
 '12'.padStart(2, '0')
 // 结果是 ’12‘
```

CSS实现方案
**flex-end 对齐**  
``` 
span>2</span>
 -
 <span>28</span>
 
 // CSS
 span{
   font-family: Consolas, Monaco, monospace;
 }
```
需要在数字前用伪元素生成一个 “0”,给元素设置一个固定宽度，这里由于是等宽字体，所以可以直接设置为 2ch，注意这个 ch 单位，它表示字符 0 的宽度,然后设置右对齐就行了  
``` 
span{
   /**/
   display: inline-flex;
   width: 2ch;
   justify-content: flex-end;
 }
```
原理很简单，在 2 个字符宽度的空间里放置 3 个字符，以右对齐的方式，是不是就自动把最左边的 0 给挤出去了？然后超出隐藏就可以了  
完整代码如下  
``` 
span::before{
   content: '0'
 }
 span{
   display: inline-flex;
   width: 2ch;
   justify-content: flex-end;
   overflow: hidden;
 }
```

**CSS 变量动态计算**  
由于 CSS 无法获取标签的文本内容，所以这里需要构建一个 CSS 变量传递下去，如下  
``` 
<span style="--num:2">2</span>
 -
 <span style="--num:12">28</span>
```
通过 var(--num) 拿到变量以后，就可以进行一系列的逻辑判断了，那么，如何在小于 10 的情况下自动补零呢？  
需要在数字前用伪元素生成一个 “0”  
``` 
span::before{
   content: '0'
 }
```
根据 CSS 变量动态隐藏这个伪元素就行了，先设置透明度，如下  
``` 
span::before{
   /**/
   opacity: calc(10 - var(--num));
 }
```
具体的逻辑就是:  
- 当 --num 等于 10 时，透明度的计算值就是 0，直接按照 0 来渲染
- 当 --num 大于 10 时，透明度的计算值就是负数值，会按照 0 来渲染
- 当 --num 小于 10 时，透明度的计算值就是大于等于 1 的值，会按照 1 来渲染

如何消除这个位置呢？方法有很多，这里采用 margin-left 的方式，如下  
``` 
 span::before{
   /**/
   margin-left: clamp(-1ch, calc((9 - var(--num)) * 1ch),0ch)；
 }
```
这里用到了 clamp，可以理解为一个区间，有 3 个值 [Min, Val, Max]，前后分别是最小、最大值，中间是可变值（注意这里是和 9 比较），所以这里的逻辑就是  
当 --num 大于等于 10 时，假设为 15，中间 calc 值计算为 -5ch，clamp 取值为最小值 -1ch
当 --num 小于 10 时，假设为 5，中间 calc 值计算为 5ch，clamp 取值为最大值 0ch
所以，最终的表现就是当大于等于 10 时 margin-left 为 - 1ch，小于 10 的时候 margin-left 为 0
完整代码如下:  
``` 
span::before{
   content: '0';
   opacity: calc(10 - var(--num));
   margin-left: clamp(-1ch, calc((9 - var(--num)) * 1ch),0ch)；
 }
```
**定义计数器样式**  
利用计数器也能实现这一效果，首先看默认的计数器效果，需要隐藏原有的文字，利用计数器让 CSS 变量通过伪元素展示出来,借助 content 属性显示 CSS var 变量值，如下  
``` 
span::before{
   counter-reset: num var(--num);
   content: counter(num);
 }
```
接下来需要用到 counter 的第 2 个参数 <counter-style>，计数器样式。和属性 list-style-type相通的，可以定义序列的样式，比如按照小写英文字母的顺序  
``` 
list-style-type: lower-latin;
```
需要一个 10 以内自动补零的计数器，刚好有个现成的，叫做 decimal-leading-zero，翻译过来就是，十进制前置零  
``` 
list-style-type: decimal-leading-zero;
```
补上这个参数就行了，完整代码如下  
``` 
span::before{
   counter-reset: num var(--num);
   content: counter(num, decimal-leading-zero);
 }
```
**计数器的扩展**  
计数器只适用于 2 位数的，如果需要 3 位数的怎么办呢？例如  
``` 
 001、002、...、010、012、...、098、099、100
```
JS 中的 padStart 可以指定填充后的位数  
``` 
'1'.padStart(3, '0')
 // 结果是 ’001‘
 '99'.padStart(3, '0')
 // 结果是 ’099‘
 '101'.padStart(3, '0')
 // 结果是 ’101‘
```
CSS 中也是有这样的能力的，叫做 @counter-style/pad，严格来说，这才是官方的补全方案，语法也非常类似  
``` 
pad: 3 "0";
```
这个需要用在自定义计数器上，也就是 @counter-style，假设定义一个计数器叫做 pad-num，实现如下  
``` 
@counter-style pad-num {
     system: extends numeric;
     pad: 3 "0";
 }
```
这里的 system 表示 “系统”，就是内置的一些计数器，比如这里用到了 extends numeric，后面的 numeric 表示数字技术系统，前面的 extends 表示扩展，以这个为基础，然后 pad: 3 "0" 就和 JS 的意义一样了，表示不足 3 位的地方补 “0”  
然后运用到计数器中：  
当然，这个兼容性略差，根据实际需求即可  

原文:  
[CSS 也能自动补全字符串？](https://mp.weixin.qq.com/s/tzcE15njxxHD4ihkwtoDTg)
