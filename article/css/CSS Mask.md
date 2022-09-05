# CSS MASK
在 CSS 中，mask 属性允许使用者通过遮罩或者裁切特定区域的图片的方式来隐藏一个元素的部分或者全部可见区域。  
## 语法
使用 mask 的方式是借助图片，类似这样：
``` 
{
    /* Image values */
    mask: url(mask.png);                       /* 使用位图来做遮罩 */
    mask: url(masks.svg#star);                 /* 使用 SVG 图形中的形状来做遮罩 */
}
```
mask 还可以接受一个类似 background 的参数，也就是渐变。类似如下使用方法：
``` 
{
    mask: linear-gradient(#000, transparent)                      /* 使用渐变来做遮罩 */
}
```
创造了一个从黑色到透明渐变色，将它运用到实际中，代码类似这样：
``` 
{
    background: url(image.png) ;
    mask: linear-gradient(90deg, transparent, #fff);
}
```
图片与 mask 生成的渐变的 transparent 的重叠部分，将会变得透明

## 使用 MASK 进行图片裁切
**使用 mask 实现图片切角遮罩**  
使用线性渐变，实现一个简单的切角图形：
``` 
.notching{
    width: 200px;
    height: 120px;
    background:
    linear-gradient(135deg, transparent 15px, deeppink 0)
    top left,
    linear-gradient(-135deg, transparent 15px, deeppink 0)
    top right,
    linear-gradient(-45deg, transparent 15px, deeppink 0)
    bottom right,
    linear-gradient(45deg, transparent 15px, deeppink 0)
    bottom left;
    background-size: 50% 50%;
    background-repeat: no-repeat;
}
```
将上述渐变运用到 mask 之上，而 background 替换成一张图片，就可以得到运用了切角效果的图片：
``` 
background: url(image.png);
                       mask:
                           linear-gradient(135deg, transparent 15px, #fff 0)
                           top left,
                           linear-gradient(-135deg, transparent 15px, #fff 0)
                           top right,
                           linear-gradient(-45deg, transparent 15px, #fff 0)
                           bottom right,
                           linear-gradient(45deg, transparent 15px, #fff 0)
                           bottom left;
                       mask-size: 50% 50%;
                       mask-repeat: no-repeat;
```
## 多张图片下使用mask
假设有两张图片，使用 mask，可以很好将他们叠加在一起进行展示。最常见的一个用法：  
``` 
div {
    position: relative;
    background: url(image1.jpg);

    &::before {
        position: absolute;
        content: "";
        top: 0;left: 0; right: 0;bottom: 0;
        background: url(image2.jpg);
        mask: linear-gradient(45deg, #000 50%, transparent 50%);
    }
}
```
两张图片，一张完全重叠在另外一张之上，然后使用 mask: linear-gradient(45deg, #000 50%, transparent 50%) 分割两张图片：
使用的 mask 的渐变，是完全的实色变化，没有过度效果.稍微修改一下 mask 内的渐变：  
``` 
{
- mask: linear-gradient(45deg, #000 50%, transparent 50%)
+ mask: linear-gradient(45deg, #000 40%, transparent 60%)
}
```
## 使用 MASK 进行转场动画
使用线性渐变 mask:linear-gradient() 进行切换,通过动态的去改变 mask 的值来实现图片的显示/转场效果
``` 
div {
    background: url(image1.jpg);
    animation: maskMove 2s linear;
}

@keyframes {
  0% {
    mask: linear-gradient(45deg, #000 0%, transparent 5%, transparent 5%);
  }
  1% {
    mask: linear-gradient(45deg, #000 1%, transparent 6%, transparent 6%);
  }
  ...
  100% {
    mask: linear-gradient(45deg, #000 100%, transparent 105%, transparent 105%);
  }
}
```
借助 SASS/LESS 等预处理器进行操作
``` 
div {
    position: relative;
    background: url(image2.jpg) no-repeat;

    &::before {
        position: absolute;
        content: "";
        top: 0;left: 0; right: 0;bottom: 0;
        background: url(image1.jpg);
        animation: maskRotate 1.2s ease-in-out;
    }
}

@keyframes maskRotate {
    @for $i from 0 through 100 { 
        #{$i}% {
            mask: linear-gradient(45deg, #000 #{$i + '%'}, transparent #{$i + 5 + '%'}, transparent 1%);
        }
    }
}
```
**使用角向渐变 mask: conic-gradient() 进行切换**  
除了 mask: linear-gradient()，使用径向渐变或者角向渐变也都是可以的  
``` 
@keyframes maskRotate {
    @for $i from 0 through 100 { 
        #{$i}% {
            mask: conic-gradient(#000 #{$i - 10 + '%'}, transparent #{$i + '%'}, transparent);
        }
    }
}
```
## mask 碰撞滤镜与混合模式
**mask & 滤镜 filter: contrast()**  
利用多重径向渐变，实现这样一张图
``` 
{
  background: radial-gradient(#000, transparent);
  background-size: 20px 20px;
}
```
利用 filter: contrast() 对比度滤镜
``` 
html,body {
    width: 100%;
    height: 100%;
    filter: contrast(5);
}

div {
    position: relative;
    width: 100%;
    height: 100%;
    background: #fff;
    
    &::before {
        content: "";
        position: absolute;
        top: 0; right: 0; bottom: 0; left: 0;
        background: radial-gradient(#000, transparent);
        background-size: 20px 20px;
    }
}
```
叠加上不同的 mask 遮罩。即可得到各种有意思的图形效果  
``` 
body {
    filter: contrast(5);
}

div {
    position: relative;
    background: #fff;
    
    &::before {
        background: radial-gradient(#000, transparent);
        background-size: 20px 20px;
      + mask: linear-gradient(-180deg, rgba(255, 255, 255, 1), rgba(255, 255, 255, .5));
    }
}
```
叠加了一个线性渐变的 mask linear-gradient(-180deg, rgba(255, 255, 255, 1), rgba(255, 255, 255, .5))，注意，两个渐变颜色都是带透明度的。或者换一个径向渐变：
``` 
{
    mask: repeating-radial-gradient(circle at 35% 65%, #000, rgba(0, 0, 0, .5), #000 25%);
}
```
添加上动画
``` 
div {
    ...
    
    &::before {
        background: radial-gradient(#000, transparent);
        background-size: 20px 20px;
        mask: repeating-radial-gradient(circle at 35% 65%, #000, rgba(0, 0, 0, .5), #000 25%);
        animation: maskMove 15s infinite linear;
    }
}

@keyframes maskMove {
    @for $i from 0 through 100 { 
        #{$i}% {
            mask: repeating-radial-gradient(circle at 35% 65%, #000, rgba(0, 0, 0, .5), #000 #{$i + 10 +  '%'});
        }
    }
}
```
加上filter: hue-rotate() 色相滤镜吗。可以让颜色也变化起来

**mask & 滤镜 filter: contrast() & 混合模式**  
以通过多嵌套一层层级，再增加一个容器背景色，再叠加上混合模式，产生不一样的效果  
先不添加使用 mask，重新构造一下结构
``` 
// HTML
<div class="wrap">
    <div class="inner"></div>
</div>
// CSS
.wrap {
    position: relative;
    height: 100%;
    background: linear-gradient(45deg, #f44336, #ff9800, #ffeb3b, #8bc34a, #00bcd4, #673ab7);
}

.inner {
    height: 100%;
    background: #000;
    filter: contrast(700%);
    mix-blend-mode: multiply;
    
    &::before {
        content: "";
        position: absolute;
        top: 0; right: 0; bottom: 0; left: 0;
        background: radial-gradient(#fff, transparent);
        background-size: 12px 12px;
    }
}
```
添加上 mask
``` 
.wrap {
    background: linear-gradient(45deg, #f44336, #ff9800, #ffeb3b, #8bc34a, #00bcd4, #673ab7);
}

.inner {
    ...
    filter: contrast(700%);
    mix-blend-mode: multiply;
    
    &::before {
        background: radial-gradient(#fff, transparent);
        background-size: 12px 12px;
      + mask: linear-gradient(#000, rgba(0, 0, 0, .5));
    }
}
```
这里叠加的是 mix-blend-mode: multiply ，可以尝试其他混合模式，得到其他不一样的效果.譬如，叠加 mix-blend-mode: difference，等等.
## mask 与图片
图片与 mask 生成的渐变的 transparent 的重叠部分，将会变得透明.也可以作用于 mask 属性传入的图片。也就是说，mask 是可以传入图片素材的，并且遵循 background-image 与 mask 图片的透明重叠部分，将会变得透明。  
使用了逐帧动画，快速切换每一帧的 mask ：  
``` 
.img1 {
    background: url(image1.jpg) no-repeat left top;
}

.img2 {
    mask: url(https://i.imgur.com/AYJuRke.png);
    mask-size: 3000% 100%;
    animation: maskMove 2s steps(29) infinite;
}

.img2::before {
    background: url(image2.jpg) no-repeat left top;
}

@keyframes maskMove {
    from {
        mask-position: 0 0;
    }
    to {
        mask-position: 100% 0;
    }
}
```

原文:
[奇妙的 CSS MASK](https://github.com/chokcoco/iCSS/issues/80)
