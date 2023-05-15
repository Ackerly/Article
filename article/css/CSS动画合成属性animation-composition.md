# CSS动画合成属性animation-composition
## 从 CSS 抛物线运动说起  
众所周知，抛物线运动是一个水平方向上匀速、垂直方向上匀加速的合成运动,这个其实用 CSS 动画也很好实现，水平和垂直两个方向的位移动画分别用不同的动画缓存函数  
这里简单介绍一下  
实现这样的效果需要一个嵌套结构  
``` 
<div class="ball-x">
   <div class="ball-y"></div>
 </div>
```
设置不同的动画缓冲函数  
``` 
.ball-x {
   animation-timing-function: linear; /*匀速直线运动*/
 }
 .ball-y {
   animation-timing-function: cubic-bezier(.55, 0, .85, .36);  /*大致匀加速运动*/
 }
```

## 动画合成属性
``` 
/* 单个值 */
 animation-composition: replace;
 animation-composition: add;/*追加*/
 animation-composition: accumulate;/*混合*/
 /* 多个值，暂不讨论 */
 animation-composition: replace, add;
 animation-composition: add, accumulate;
 animation-composition: replace, add, accumulate;
```
主要是这 3 个关键词：
- replace：覆盖（默认值）。动画会覆盖原有属性。
- add：追加。动画追加到原有属性后面
- accumulate：累加。动画会和原有属性相同的部分进行累加

假设有一个元素，默认有一些样式  
``` 
div{
   transform-origin: 50% 50%;
  transform: translateX(50px) rotate(45deg);
 }
```
然后，给一个动画，关键帧是这样的  
``` 
@keyframes adjust {
   to {
     transform: translateX(100px);
   }
 }
```
第一个 replace，也就是默认效果。其实就是直接将 transform 中的 translateX(50px) rotate(45deg) 变成了 translateX(100px)。  
第二个 add。可以理解成直接在 transform 后追加，也就是最后变成了 translateX(50px) rotate(45deg) translateX(100px)，等同于先向右移动 50px，然后旋转 45deg，再向右移动 100px。
第三个 accumulate。可以理解成将已有的 translateX (50px) 累加，最后结果是 translateX (150px) rotate (45deg)  

## 再来看抛物线运动
假设水平、垂直两个方向的动画关键帧是这样的
``` 
@keyframes ballMoveX {
     100%{
         transform: translateX(300px)
     }
 }
 @keyframes ballMoveY {
     100% {
         transform: translateY(300px)
     }
 }
```
然后小球将这两个动画合起来  
``` 
.ball{
     animation:
     ballMoveX 1s linear infinite alternate,
     ballMoveY 1s cubic-bezier(.55, 0, .85, .36) infinite alternate;
 }
```
可以得到一个很奇怪的动画,原因其实是这两个属性冲突了，后面的动画覆盖了前面的，导致动画的结束点其实是 translateY(300px)  
像这种情况下，用动画合成属性就非常合适了  
``` 
.ball{
     ...
       animation-composition: add; /*accumulate也行*/
 }
```
add 和 accumulate 都行，因为 translateX 和 translateY 并不能累加，只能追加。  

## 兼容性和总结
这方面 Safari 居然跑在了前头，然后 Chrome 也是最近得到了正式支持，Firefox 目前仍然是实验支持，不过离正式支持也不远了。  


原文:  
[了解一下全新的CSS动画合成属性animation-composition](https://mp.weixin.qq.com/s/YHiu13rHK1PM_8Oytf7BaQ)
