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



参考:  
[初识图片裁切 object-view-box 属性](https://mp.weixin.qq.com/s/YNEMmPyO2rKNR2euLgxF3g)
