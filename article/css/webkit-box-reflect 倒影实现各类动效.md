# webkit-box-reflect 倒影实现各类动效
-webkit-box-reflect 是一个非常有意思的属性，它让 CSS 有能力像镜子一样，反射元素原本绘制的内容  
**-webkit-box-reflect 基本用法**  
``` 
div {
    -webkit-box-reflect: below;
}
```
below 可以是 below | above | left | right 代表下上左右，也就是有 4 个方向可以选。  
假设我们有如下一张图片：  
``` 
// HTML
<div></div>

// CSS
div {
    background-image: url('https://images.pokemontcg.io/xy2/12_hires.png');
    -webkit-box-reflect: right;
}
```
**设置倒影距离**  
在方向后面，还可以接一个具体的数值大小，表示倒影与原元素间的距离  
``` 
div {
    background-image: url('https://images.pokemontcg.io/xy2/12_hires.png');
    -webkit-box-reflect: right 10px;
}
```
加上 10px 之后，倒影与原元素间将间隔 10px：
**设置倒影虚实**  
能再设置一个渐变值，利用这个渐变值，可以实现倒影的一个虚化效果  
``` 
div {
    background-image: url('https://images.pokemontcg.io/xy2/12_hires.png');
    -webkit-box-reflect: below 2px linear-gradient(transparent, rgba(0, 0, 0, .5));
}
```
这里的渐变就是给倒影的图片添加了一个 MASK 属性，MASK 属性的 transparent 部分，图片将变得透明，而实色部分，则保持原图
**使用 -webkit-box-reflect 实现一些有意思的动效**
- 按钮中运用 -webkit-box-reflect
- 在文字中运用 -webkit-box-reflect
- 在 3D 中运用 -webkit-box-reflect

**使用 -webkit-box-reflect 创造艺术图案**

-webkit-box-reflect 可以生成倒影，那么我们利用它进行不断的套娃，一层叠一层，那么只需要生成一个基本的元素，就可以利用倒影产生出各种不同的效果。  
假设有如下结构：
``` 
<div class="g-wrap1">
    <div class="g-wrap2">
        <div class="g-wrap3">
            <div class="g-wrap4"></div>
        </div>
    </div>
</div>
```
给 .g-wrap4 实现一个图形，例如这样：
``` 
.g-wrap4 {
    background: 
        radial-gradient(circle at 0 0, #000 30%, transparent 30%, transparent 40%, #000 40%, #000 50%, transparent 50%),
        radial-gradient(circle at 100% 100%, #000 10%, transparent 10%, transparent 30%, #000 30%, #000 40%, transparent 40%);
}
```
然后就是 4 层套娃， 首先给 .g-wrap4 加上一层倒影 -webkit-box-reflect：
``` 
.g-wrap4 {
    -webkit-box-reflect: right 0px;
}
```
继续套娃，给 .g-wrap3 加上一层倒影 -webkit-box-reflect：
``` 
.g-wrap4 {
    -webkit-box-reflect: right 0px;
}
.g-wrap3 {
    -webkit-box-reflect: below 0px;
}
```
给 .g-wrap2 加上一层倒影 -webkit-box-reflect：  
``` 
.g-wrap4 {
    -webkit-box-reflect: right 0px;
}
.g-wrap3 {
    -webkit-box-reflect: below 0px;
}
.g-wrap2 {
    -webkit-box-reflect: left 0px;
}
```
给 .g-wrap1 加上一层倒影 -webkit-box-reflect：  
``` 
.g-wrap4 {
    -webkit-box-reflect: right 0px;
}
.g-wrap3 {
    -webkit-box-reflect: below 0px;
}
.g-wrap2 {
    -webkit-box-reflect: left 0px;
}
.g-wrap1 {
    -webkit-box-reflect: above 0px;
}
```

原文:  
[巧用 -webkit-box-reflect 倒影实现各类动效](https://github.com/chokcoco/iCSS/issues/100)
