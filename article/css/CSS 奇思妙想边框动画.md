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




原文:  
[CSS 奇思妙想边框动画](https://mp.weixin.qq.com/s/NDJEexaiDcfEXeNMQrA79A)
