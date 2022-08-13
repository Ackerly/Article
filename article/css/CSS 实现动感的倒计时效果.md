# CSS 实现动感的倒计时效果  
**数字的变化**  
在以前，数字的变化可能需要创建多个标签，然后改变位移来实现  
``` 
<count-down>
  <span>5</span>
   <span>4</span>
   <span>3</span>
   <span>2</span>
   <span>1</span>
 </count-down>
```
这种方式需要创建多个标签，略微繁琐，也不易扩展。现在有更简洁的方式可以实现了，那就是 CSS @property。这是干什么的呢？简单来讲，可以自定义属性，在这个例子中，可以让数字像颜色一样进行过渡和动画，可能不太懂，直接看例子吧  
假设 HTML 是这样的  
``` 
<count-down style="--t: 5"></count-down>
```
通过 CSS 变量将数字渲染到页面，这里需要借助伪元素和计数器  
``` 
count-down::after{
   counter-reset: time var(--t);
   content: counter(time);
 }
```
如何让这个数字变化呢？可以用到 CSS 动画  
``` 
@keyframes count {
     to {
         --t: 0     }
 }
 count-down::after{
     --t: 5;
     counter-reset: time var(--t);
     content: counter(time);
     animation: count 5s forwards;
 }
```
现在的效果仅仅是 5 秒后，数字从 5 变成了 0，并没有 5 => 4 => 3 => 2 => 1 这种阶段变化。然后最重要的一步来了，加上以下自定义属性  
``` 
@property --t {
     syntax: '<integer>';
     inherits: false;
     initial-value: 0;
 }
```
通过 @property 定义后，这个变量 --t 本身可以单独设置动画了，就像颜色变化一样。  
使用计数器的好处是可以随意更换类型，比如将上面的阿拉伯数字换成中文计数，只需要更换计数器类型就行了  
``` 
count-down::after{
     --t: 5;
     counter-reset: time var(--t);
     content: counter(time, cjk-decimal); /*中日韩十进制数*/
     animation: count 5s forwards;
 }
```
**倒计时的终点**  
上面的计数器最后的终点是 “0”，显然我们需要一些特定的提示，比如 “Go~”  
如何改变最后一帧的状态呢？这里有两种方式：  
- 通过动画覆盖
- 通过计数器覆盖

首先来看第一种方式，这个比较好理解，重新定义一个动画，在倒计时结束后，将最后一帧重置一下  
``` 
@keyframes stop {
     to {
         content: 'Go~';
     }
 }
 count-down::after{
     --t: 5;
     counter-reset: time var(--t);
     content: counter(time);
     animation: count 5s forwards,
     stop 5s step-end forwards;
 }
```
注意这里动画函数是 step-end，为啥是这个呢？step-end 也可写作 steps(1,end)，你可以理解为在整个动画只有两种状态，在运行过程中，都是初始状态，只有到达最后一帧才改变状态  

第二种方式，通过自定义计数器来实现。原理其实和 JS 思维有些类似，当数字为 0 时，让计数器指定一个特殊的值，具体实现如下  
``` 
@counter-style stop {
     system: cyclic;
     symbols: "Go~";
     range: 0 0;
 }
```
这里有个 range 属性，表示计数器的范围，由于这里只需要指定为 0，所以是区间 0 0。然后是 system，表示计算系统，这里为 cyclic，表示循环使用开发者提供的一套字符，字符由 symbos 定义。然后 symbos 表示计算符号，也就是具体展示的字符，这里指定为 Go~ 就行了。  
应用:  
``` 
count-down::after{
    /**/
     counter-reset: time var(--t);
     content: counter(time, stop); /*自定义计数器*/
 }
```

**缩放和透明度变化**  
这两个动画其实是同时进行的，可以放在一个动画里  
``` 
@keyframes shark {
     0%{
         opacity: 1;
         transform: scale(1);
     }

     50%{
         opacity: 0;
         transform: scale(0.4);
     }
 }
```
设置动画时长为 1s，循环 5 次  
``` 
count-down::after{
     --t: 5;
     counter-reset: time var(--t);
     content: counter(time);
     animation: count 5s steps(5) forwards,
     shark 1s 5;
 }
```
是不是稍微有些突兀？因为数字的变化是突然的，需要将数字的变化隐藏到透明度为 0 的时候，为了达到这种效果，只需要将闪烁动画延迟 0.5 秒即可  
``` 
count-down::after{
     --t: 5;
     counter-reset: time var(--t);
     content: counter(time);
     animation: count 5s steps(5) forwards,
     shark 1s .5s 5; /*延迟 0.5s*/
 }
```
不过还有优化的空间。比如现在数字动画有些太连贯了，如果希望数字出现后稍微停留一小会，或者说希望出现的慢一点，消失的快一点，如何处理呢？其实这比想象中的要容易许多，只需要改一下关键帧位置就行了，如下  
``` 
@keyframes shark {
     0%{
         opacity: 1;
         transform: scale(1);
     }

     20%{
         opacity: 0;
         transform: scale(0.4);
     }
 }
```
同时，延迟的时间也需要改成 0.8 秒  
完整代码如下:  
``` 
@property --t {
     syntax: '<integer>';
     inherits: false;
     initial-value: 0;
 }
 @counter-style stop {
     system: cyclic;
     symbols: "Go~";
     range: infinite 0;
 }
 html,body{
     margin: 0;
     height: 100%;
     display: grid;
     place-content: center;
 }
 count-down{
     display: flex;
     align-items: center;
     justify-content: center;
     font-family: Consolas, Monaco, monospace;
     font-size: 120px;
 }
 count-down::after{
     --t: 5;
     --dur: 1;
     counter-reset: time var(--t);
     content: counter(time, stop);
     animation: count calc( var(--t) * var(--dur) * 1s ) steps(var(--t)) forwards,
     shark calc(var(--dur) * 1s) calc(var(--dur) * .8s) calc(var(--t));
 }
 @keyframes count {
     to {
         --t: 0;
     }
 }
 @keyframes shark {
     0%{
         opacity: 1;
         transform: scale(1);
     }

     20%{
         opacity: 0;
         transform: scale(0.4);
     }
 }
```
**其他动画效果**  
除了缩放效果，还可以有一些位移的动画  
``` 
@keyframes shark {
     0%{
         opacity: 1;
         transform: translateY(0);
     }

     20%{
         opacity: 0;
         transform: translateY(100px);
     }
 }
```
是不是有点奇怪？动画不够连贯，一会向下一会向上，有没有办法消失和出现都是从上到下的呢？当然也是可以的，实现如下  
``` 
@keyframes shark {
     0%{
         opacity: 1;
         transform: translateY(0);
     }

     20%{
         opacity: 0;
         transform: translateY(100px);
     }

     21%{
         opacity: 0;
         transform: translateY(-100px);
     }
 }
```
这里多加了一个非常 “邻近” 的关键帧，表示在透明状态下，“迅速” 改变位移，这样在数字出现时的动画就感觉是从上到下的，整体更为流畅，效果如下
还可以调整一下前面的缩放效果，让出来的时候更大，效果也更为震撼  
``` 
@keyframes shark {
     0%{
         opacity: 1;
         transform: scale(1);
     }

     20%{
         opacity: 0;
         transform: scale(.4);
     }

     21%{
         opacity: 0;
         transform: scale(5);
     }
 }
```


参考：  
[动画合成小技巧！CSS 实现动感的倒计时效果](https://mp.weixin.qq.com/s/Wo0_UE_Sa-CP3UuX-y4cPA)
