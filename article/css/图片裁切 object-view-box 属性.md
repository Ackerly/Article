# 图片裁切 object-view-box 属性
我们有一个需要裁剪的图像。请注意，我们只想要该图像的特定部分。
可以通过以下方式之一来解决这个问题  
- 使用 <img> 并将其包裹在一个额外的元素中
- 使用图像作为 background-image 并修改位置和大小

包在一个额外的元素中  
常见的解决这个问题的方法,步骤如下：  
- 将图像包裹在另一个元素中（在我们的例子中是 <figure>）
- 添加 position: relative 和 overflow: hidden
- 为图片添加了 position: absolute，并对定位和尺寸值进行了调整

``` 
<figure>
     <img src="img/brownies.jpg" alt="">
 </figure>
 
 figure {
     position: relative;
     width: 300px;
     aspect-ratio: 1;
     overflow: hidden;
     border-radius: 15px;
 }

 img {
     position: absolute;
     left: -23%;
     top: 0;
     right: 0;
     bottom: 0;
     width: 180%;
     height: 100%;
     object-fit: cover;
 }
```
将图像作为背景  
使用一个 <div> 将图片作为背景，然后我们改变位置和大小值  
``` 
<div class="brownies"></div>

.brownies {
   width: 300px;
   aspect-ratio: 3 / 2;
   background-image: url("brownies.jpg");
   background-size: 700px auto;
   background-position: 77% 68%;
   background-repeat: no-repeat;
 }
```
**引入Object-View-Box**  
> object-view-box属性在一个元素上指定了一个 "视图框"，类似于<svg viewBox>属性，在元素的内容上进行缩放或平移

该属性的值是 <basic-shape-rect> = <inset()> | <rect()> | <xywh()>  
有了 object-view-box，就能用inset从四边（上、右、下、左）画一个矩形，然后应用object-fit: cover来避免变形。  
``` 
 <img src="img/brownies.jpg" alt="">
 img {
     aspect-ratio: 1;
     width: 300px;
     object-view-box: inset(25% 20% 15% 0%);
 }
```
**图像的内在尺寸**  
内在大小是默认的图像宽度和高度。处理的图像大小为 1194 × 1194 px.  
``` 
img {
     aspect-ratio: 1;
     width: 300px;
 }
```
**使用 inset**  
inset()值将基于原始图像的宽度和高度，从而形成一个裁剪过的图像。它将帮助我们绘制一个嵌入的矩形并控制四个边缘，类似于处理margin或padding  
inset 值定义了一个嵌入的矩形。我们可以控制四个边缘，就像我们处理margin或padding一样  
``` 
img {
     aspect-ratio: 1;
     width: 300px;
     object-view-box: inset(25% 20% 15% 0%);
 }
```
上述内容的背后的样子，值 25%、20%、15%和0% 的值分别代表顶部、右侧、底部和左侧  
**修复图像失真**  
如果图像的尺寸是正方形的，那么裁剪后的结果将是变形的。  
这可以使用 object-fit 属性来解决
``` 
img {
     aspect-ratio: 1;
     width: 300px;
     object-view-box: inset(25% 20% 15% 0%);
     object-fit: cover;
 }
```
**放大或缩小**  
可以使用 inset 来放大或缩小图像，过渡或动画不能与object-view-box工作  
也可以用一个负的 inset 值来缩小

原文: 
[初识图片裁切 object-view-box 属性](https://mp.weixin.qq.com/s/YNEMmPyO2rKNR2euLgxF3g)
