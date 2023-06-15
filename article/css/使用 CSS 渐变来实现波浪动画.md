# 使用 CSS 渐变来实现波浪动画
## 波浪的原理
可以分解一下波浪的原理，看似有点复杂，又是贝塞尔曲线，又是上下震动，其实都是视觉错觉，本质上是一个水平方向的周期性运动，曲线都是固定的  

## 曲面的绘制
提到曲面，可以想到径向渐变，并且是椭圆的。  
首先来看左边这个曲面，其实就是一个透明到纯色的径向渐变，用代码实现就是  
``` 
div{
  background: radial-gradient(100% 57% at top ,#0000 100%,#2196F3 100.5%) no-repeat;
  background-size: 50% 100%;
  background-position: 0 100%;
  background-repeat: no-repeat;
}
```
为了自适应容器，这里都采用的是百分比单位  
用同样的方式，可以绘制出又半部分，为了方便管理，可以用 CSS 变量来代替  
``` 
div{
  --c: #2196F3;
  --w1: radial-gradient(100% 57% at top ,#0000 100%,var(--c) 100.5%) no-repeat;
  --w2: radial-gradient(100% 57% at bottom,var(--c) 100%,#0000 100.5%) no-repeat;
  background: var(--w1),var(--w2);
  background-position: 0% 100%, 100% 100%;
  background-size: 50% 100%;
}
```
这个过程中，需要细微调整一下两个曲面的位置关系，确保能够完美的衔接在一起  
## 波浪动画
其实就是一个重复水平运动，在这里只需要改变background-position-x就行了。  
不过需要注意的是，为了保证动画的连贯性，需要再“复制”一份完全相同的，避免在动画首尾处出现“空档”  
就体现出 CSS 变量的好处了，直接再写两个变量即可，如下  
``` 
div{
  --w1: radial-gradient(100% 57% at top ,#0000 100%,var(--c) 100.5%) no-repeat;
  --w2: radial-gradient(100% 57% at bottom, red 100%,#0000 100.5%) no-repeat;
  background: var(--w1),var(--w2),var(--w1),var(--w2); /*两份*/
  background-position: -200% 100%, -100% 100%, 0% 100%, 100% 100%;
  background-size: 50% 100%;
  animation: m 1s infinite linear; /*无限循环动画*/
}
```
然后是动画关键帧，改变background-position-x即可
``` 
@keyframes m {
  0%  {background-position-x:-200%, -100%, 0%, 100%}
  100%{background-position-x:  0%, 100%, 200%, 300%}
}
```
将颜色都改成相同后，由于看不清左右的运动，只能看到上下在晃动，就感觉非常像一个波浪了  
下面是完整代码  
``` 
.wave{
  width: 400px;
  height: 200px;
  outline: 2px dashed gray;
  --c: #2196F3;
  --w1: radial-gradient(100% 57% at top ,#0000 100%,var(--c) 100.5%) no-repeat;
  --w2: radial-gradient(100% 57% at bottom, var(--c) 100%,#0000 100.5%) no-repeat;
  background: var(--w1),var(--w2),var(--w1),var(--w2);
  background-position-x: -200%, -100%, 0%, 100%;
  background-position-y: 100%;
  background-size: 50.5% 100%;
  animation: m 1s infinite linear;
}
@keyframes m {
  0%  {background-position-x:-200%, -100%, 0%, 100%}
  100%{background-position-x:  0%, 100%, 200%, 300%}
}
```
## 文字波浪动画
相比于其他实现，渐变的最大优势是不占用任何标签，包括伪元素，这样即使在非常苛刻的情况下也能使用，比如文章开头的文字波浪效果  
由于只是背景，直接像普通的渐变文字那样使用就行了，完整代码如下  
``` 
.txt{
  --c: #2196F3;
  --w1: radial-gradient(100% 57% at top ,#0000 100%,var(--c) 100.5%) no-repeat;
  --w2: radial-gradient(100% 57% at bottom, var(--c) 100%,#0000 100.5%) no-repeat;
  background: var(--w1),var(--w2),var(--w1),var(--w2);
  background-position-x: -200%, -100%, 0%, 100%;
  background-position-y: 100%;
  background-size: 50.5% 100%;
  animation: m 1s infinite linear;
  font-size: 100px;
  font-weight: bold;
  color: transparent;
  -webkit-background-clip: text; /*文本裁切*/
  -webkit-text-stroke: 2px var(--c);
}
@keyframes m {
  0%  {background-position-x:-200%, -100%, 0%, 100%}
  100%{background-position-x:  0%, 100%, 200%, 300%}
}
```


原文:  
[巧妙使用 CSS 渐变来实现波浪动画](https://mp.weixin.qq.com/s/JGg1E_-J875qgxNx0BYoJw)
