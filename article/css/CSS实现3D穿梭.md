# CSS实习3D穿梭
**绘制图形**  
``` 
<div class="g-container">
  <div class="g-group">
      <div class="item item-right"></div>
      <div class="item item-left"></div>   
      <div class="item item-top"></div>
      <div class="item item-bottom"></div> 
      <div class="item item-middle"></div>    
  </div>
</div>

body {
  background: #000;
}
.g-container {
  position: relative;
}
.g-group {
  position: absolute;
  width: 100px;
  height: 100px;
  left: -50px;
  top: -50px;
  transform-style: preserve-3d;
}
.item {
  position: absolute;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, .5);
}
.item-right {
  background: red;
  transform: rotateY(90deg) translateZ(50px);
}
.item-left {
  background: green;
  transform: rotateY(-90deg) translateZ(50px);
}
.item-top {
  background: blue;
  transform: rotateX(90deg) translateZ(50px);
}
.item-bottom {
  background: deeppink;
  transform: rotateX(-90deg) translateZ(50px);
}
.item-middle {
  background: rgba(255, 255, 255, 0.5);
  transform: rotateX(180deg) translateZ(50px);
}
```
给父元素 .g-container 设置一个极小的 perspective，譬如，设置一个 perspective: 4px
``` 
.g-container {
  position: relative;
+ perspective: 4px;
}
// ...其余样式保持不变
```
由于 perspective 生效，原本的平面效果变成了 3D 的效果。将 perspective设置非常之下，每个 item 的 transform: translateZ(50px) 设置的又比较大，图片在视觉上被拉伸的非常厉害。但是整体是充满整个屏幕的。  
给父元素增加一个动画，通过控制父元素的 translateZ() 进行变化即可：  
``` 
.g-container{
  position: relative;
  perspective: 4px;
  perspective-origin: 50% 50%;
}

.g-group{
  position: absolute;
  // ... 一些定位高宽代码
  transform-style: preserve-3d;
  + animation: move 8s infinite linear;
}

@keyframes move {
  0%{
    transform: translateZ(-50px) rotate(0deg);
  }
  100%{
    transform: translateZ(50px) rotate(0deg);
  }
}
```
继续优化：  
1. 通过叠加两组同样的效果，一组比另一组通过负的 animation-delay 提前行进，使两组动画衔接起来
2. 再通过透明度的变化，隐藏掉 item-middle 迎面飞来的突兀感
3. 通过父元素的滤镜 hue-rotate 控制图片的颜色变化

``` 
<div class="g-container">
  <div class="g-group">
      <div class="item item-right"></div>
      <div class="item item-left"></div>   
      <div class="item item-top"></div>
      <div class="item item-bottom"></div> 
      <div class="item item-middle"></div>    
  </div>
  <!-- 增加一组动画 -->
  <div class="g-group">
      <div class="item item-right"></div>
      <div class="item item-left"></div>   
      <div class="item item-top"></div>
      <div class="item item-bottom"></div>   
      <div class="item item-middle"></div>    
  </div>
</div>
```
``` 
.g-container{
  perspective: 4px;
  position: relative;
  // hue-rotate 变化动画，可以让图片颜色一直变换
  animation: hueRotate 21s infinite linear;
}

.g-group{
  transform-style: preserve-3d;
  animation: move 12s infinite linear;
}
// 设置负的 animation-delay，让第二组动画提前进行
.g-group:nth-child(2){
  animation: move 12s infinite linear;
  animation-delay: -6s;
}
.item {
  background: url(https://z3.ax1x.com/2021/08/20/fLwuMd.jpg);
  background-size: cover;
  opacity: 1;
  // 子元素的透明度变化，减少动画衔接时候的突兀感
  animation: fade 12s infinite linear;
  animation-delay: 0;
}
.g-group:nth-child(2) .item {
  animation-delay: -6s;
}
@keyframes move {
  0%{
    transform: translateZ(-500px) rotate(0deg);
  }
  100%{
    transform: translateZ(500px) rotate(0deg);
  }
}
@keyframes fade {
  0%{
    opacity: 0;
  }
  25%,
  60%{
    opacity: 1;
  }
  100%{
    opacity: 0;
  }
}
@keyframes hueRotate {
  0% {
    filter: hue-rotate(0);
  }
  100% {
    filter: hue-rotate(360deg);
  }
}
```
**继续优化，不使用图片**  
使用 box-shadow，去替换实际的星空图，也是在一个 div 标签内实现，借助了 SASS 的循环函数：
``` 
<div></div>

// CSS
@function randomNum($max, $min: 0, $u: 1) {
	@return ($min + random($max)) * $u;
}

@function randomColor() {
    @return rgb(randomNum(255), randomNum(255), randomNum(255));
}

@function shadowSet($maxWidth, $maxHeight, $count) {
    $shadow : 0 0 0 0 randomColor();
    
    @for $i from 0 through $count {         
        $x: #{random(10000) / 10000 * $maxWidth};
        $y: #{random(10000) / 10000 * $maxHeight};

        
        $shadow: $shadow, #{$x} #{$y} 0 #{random(5)}px randomColor();
    }
    
    @return $shadow;
}

body {
    background: #000;
}

div {
    width: 1px;
    height: 1px;
    border-radius: 50%;
    box-shadow: shadowSet(100vw, 100vh, 500);
}
```
替换 .item 这个 class，只列出修改的部分
``` 
// 原 CSS，使用了一张星空图
.item {
  position: absolute;
  width: 100%;
  height: 100%;
  background: url(https://z3.ax1x.com/2021/08/20/fLwuMd.jpg);
  background-size: cover;
  animation: fade 12s infinite linear;
}

// 修改后的 CSS 代码
.item {
  position: absolute;
  width: 100%;
  height: 100%;
  background: #000;
  animation: fade 12s infinite linear;
}
.item::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 1px;
  height: 1px;
  border-radius: 50%;
  box-shadow: shadowSet(100vw, 100vh, 500);
}
```
通过调整动画的时间，perspective 的值，每组元素的 translateZ() 变化距离，可以得到各种不一样的观感和效果，

原文:  
[3D 穿梭效果？使用 CSS 轻松搞定](https://juejin.cn/post/7028757824695959588)
