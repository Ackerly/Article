# CSS 奇思妙想边框动画
## 边框长度变化  
借用了元素的两个伪元素。两个伪元素分别只设置上、左边框，下、右边框，通过 hover 时改变两个伪元素的高宽即可。  
``` 
div {
    position: relative;
    border: 1px solid #03A9F3;
    
    &::before,
    &::after {
        content: "";
        position: absolute;
        width: 20px;
        height: 20px;
    }
    
    &::before {
        top: -5px;
        left: -5px;
        border-top: 1px solid var(--borderColor);
        border-left: 1px solid var(--borderColor);
    }
    
    &::after {
        right: -5px;
        bottom: -5px;
        border-bottom: 1px solid var(--borderColor);
        border-right: 1px solid var(--borderColor);
    }
    
    &:hover::before,
    &:hover::after {
        width: calc(100% + 9px);
        height: calc(100% + 9px);
    }
}
```
## 虚线边框动画
使用 dashed 关键字，可以方便的创建虚线边框  
``` 
div {
    border: 1px dashed #333;
}
```
当然，我们的目的是让边框能够动起来。使用 dashed 关键字是没有办法的。但是实现虚线的方式在 CSS 中有很多种，譬如渐变就是一种很好的方式：  
``` 
div {
    background: linear-gradient(90deg, #333 50%, transparent 0) repeat-x;
    background-size: 4px 1px;
    background-position: 0 0;
}
```
渐变支持多重渐变，我们把容器的 4 个边都用渐变表示即可：  
``` 
div {
    background: 
        linear-gradient(90deg, #333 50%, transparent 0) repeat-x,
        linear-gradient(90deg, #333 50%, transparent 0) repeat-x,
        linear-gradient(0deg, #333 50%, transparent 0) repeat-y,
        linear-gradient(0deg, #333 50%, transparent 0) repeat-y;
    background-size: 4px 1px, 4px 1px, 1px 4px, 1px 4px;
    background-position: 0 0, 0 100%, 0 0, 100% 0;
}
```
我们的虚线边框动画其实算是完成了一大半了。虽然 border-style: dashed 不支持动画，但是渐变支持呀。我们给上述 div 再加上一个 hover 效果，hover 的时候新增一个 animation 动画，改变元素的 background-position 即可。  
``` 
div:hover {
    animation: linearGradientMove .3s infinite linear;
}

@keyframes linearGradientMove {
    100% {
        background-position: 4px 0, -4px 100%, 0 -4px, 100% 4px;
    }
}
```
hover 上去的时候，边框就能动起来，因为整个动画是首尾相连的，无限循环的动画看起来就像是虚线边框在一直运动  
这里还有另外一个小技巧，如果我们希望虚线边框动画是从其他边框，过渡到虚线边框，再行进动画。完全由渐变来模拟也是可以的，如果想节省一些代码，使用 border 会更快一些，譬如这样：  
``` 
div {
    border: 1px solid #333;
    
    &:hover {
        border: none;
        background: 
            linear-gradient(90deg, #333 50%, transparent 0) repeat-x,
            linear-gradient(90deg, #333 50%, transparent 0) repeat-x,
            linear-gradient(0deg, #333 50%, transparent 0) repeat-y,
            linear-gradient(0deg, #333 50%, transparent 0) repeat-y;
        background-size: 4px 1px, 4px 1px, 1px 4px, 1px 4px;
        background-position: 0 0, 0 100%, 0 0, 100% 0;
    }
}
```
由于 border 和 background 在盒子模型上位置的差异，视觉上会有一个很明显的错位的感觉  
要想解决这个问题，可以把 border 替换成 outline，因为 outline 可以设置 outline-offset。便能完美解决这个问题：  
``` 
div {
    outline: 1px solid #333;
    outline-offset: -1px;
    
    &:hover {
        outline: none;
    }
}
```
## 渐变的其他妙用
``` 
div {
    position: relative;

    &::after {
        content: '';
        position: absolute;
        left: -50%;
        top: -50%;
        width: 200%;
        height: 200%;
        background-repeat: no-repeat;
        background-size: 50% 50%, 50% 50%;
        background-position: 0 0, 100% 0, 100% 100%, 0 100%;
        background-image: linear-gradient(#399953, #399953), linear-gradient(#fbb300, #fbb300), linear-gradient(#d53e33, #d53e33), linear-gradient(#377af5, #377af5);
    }
}
```
这里运用了元素的伪元素生成的这个图形，并且，宽高都是父元素的 200%，超出则 overflow: hidden。  
接下来，给它加上旋转：  
``` 
div {
    animation: rotate 4s linear infinite;
}

@keyframes rotate {
 100% {
  transform: rotate(1turn);
 }
}
```
最后，再利用一个伪元素，将中间遮罩起来，一个 Nice 的边框动画就出来了 (动画会出现半透明元素，方便示意看懂原理)  
## 改变渐变的颜色
``` 
div::after {
    content: '';
    position: absolute;
    left: -50%;
    top: -50%;
    width: 200%;
    height: 200%;
    background-color: #fff;
    background-repeat: no-repeat;
    background-size: 50% 50%;
    background-position: 0 0;
    background-image: linear-gradient(#399953, #399953);
}
```
不过如果是单线条，有个很明显的缺陷，就是边框的末尾是一个小三角而不是垂直的，可能有些场景不适用或者 PM 接受不了。  
## conic-gradient 的妙用
上述主要用到了的是线性渐变 linear-gradient 。使用角向渐变 conic-gradient 其实完全也可以实现一模一样的效果。  
试着使用 conic-gradient 也实现一次，这次换一种暗黑风格。核心代码如下：  
``` 
.conic {
 position: relative;
 
 &::before {
  content: '';
  position: absolute;
  left: -50%;
  top: -50%;
  width: 200%;
  height: 200%;
  background: conic-gradient(transparent, rgba(168, 239, 255, 1), transparent 30%);
  animation: rotate 4s linear infinite;
 }
}
@keyframes rotate {
 100% {
  transform: rotate(1turn);
 }
}
```
## clip-path 的妙用
clip-path 本身是可以进行坐标点的动画的，从一个裁剪形状变换到另外一个裁剪形状。  
利用这个特点，我们可以巧妙的实现这样一种 border 跟随效果。伪代码如下：  
``` 
div {
    position: relative;

    &::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        border: 2px solid gold;
        animation: clippath 3s infinite linear;
    }
}

@keyframes clippath {
    0%,
    100% {
        clip-path: inset(0 0 95% 0);
    }
    25% {
        clip-path: inset(0 95% 0 0);
    }
    50% {
        clip-path: inset(95% 0 0 0);
    }
    75% {
        clip-path: inset(0 0 0 95%);
    }
}

```
因为会裁剪元素，借用伪元素作为背景进行裁剪并动画即可，使用 clip-path 的优点了，切割出来的边框不会产生小三角。同时，这种方法，也是支持圆角 border-radius 的。  
## overflow 的妙用
两个核心点：
1. 用 overflow: hidden，把原本在容器外的一整个元素隐藏了起来
2. 利用了 transform-origin，控制了元素的旋转中心

看到的动画只是原本现象的一小部分，通过特定的裁剪、透明度的变化、遮罩等，让我们最后只看到了原本现象的一部分。  

## border-image 的妙用
以利用 border-image-slice 及 border-image-repeat 的特性，得到类似的边框图案：  
``` 
div {
  width: 200px;
  height: 120px;
  border: 24px solid;
  border-image: url(image-url);
  border-image-slice: 32;
  border-image-repeat: round;
}
```
在这个基础上，可以随便改变元素的高宽，如此便能扩展到任意大小的容器边框中  
利用 border-image 的边框动画，使用一个运动背景图，就能得到运动的边框图  

## border-image 使用渐变
可以利用 border-image + filter + clip-path 实现渐变变换的圆角边框：  
``` 
.border-image-clip-path {
    width: 200px;
    height: 100px;
    border: 10px solid;
    border-image: linear-gradient(45deg, gold, deeppink) 1;
    clip-path: inset(0px round 10px);
    animation: huerotate 6s infinite linear;
    filter: hue-rotate(360deg);
}

@keyframes huerotate {
    0% {
        filter: hue-rotate(0deg);
    }
    100% {
        filter: hue-rotate(360deg);
    }
}
```

原文:  
[CSS 奇思妙想边框动画](https://mp.weixin.qq.com/s/NDJEexaiDcfEXeNMQrA79A)
