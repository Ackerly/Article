# CSS 自定义形状网格布局  
在正常的开发中，我们会遇到很多元素块排列对齐的需求，如九宫格抽奖，多张图片上传后等分布局预览，微信朋友圈多张图片展示等。这都是正常的正方形很规整的布局。  
如果图像不是完全正方形，而是形状像六边形或菱形怎么办？结合已经研究过的 CSS 网格技术，并加入一些 CSS clip-path和mask魔法，可以想象的任何形状创建精美的图像网格！  
**六边形 CSS 网格**    
使用clip-path在图像上使用来创建六边形形状，并将它们全部放在同一个网格区域中，以便它们重叠。  
```
 .gallery {
  --s: 150px; /* controls the size */
  display: grid;
}

.gallery > img {
  grid-area: 1/1;
  width: var(--s);
  aspect-ratio: 1.15;
  object-fit: cover;
  clip-path: polygon(25% 0%, 75% 0%, 100% 50%, 75% 100%, 25% 100%, 0 50%);
}
```
此时所有的图像都是六边形并且重叠在一起。所以看起来我们只有一个六边形的图像元素，但实际上有七个。下一步将把图像平移到它们正确放置的网格上。  
保留其中一张图像在中心位置。其余图像使用 CSS translate 平移在它周围。这是我为网格中的每个图像提出的模拟公式：  
```
translate((height + gap)*sin(0deg), (height + gap)*cos(0))
translate((height + gap)*sin(60deg), (height + gap)*cos(60deg))
translate((height + gap)*sin(120deg), (height + gap)*cos(120deg))
translate((height + gap)*sin(180deg), (height + gap)*cos(180deg))
translate((height + gap)*sin(240deg), (height + gap)*cos(240deg))
translate((height + gap)*sin(300deg), (height + gap)*cos(300deg))
```
经过一些计算和优化后，得到以下最终 CSS  
```
.gallery {
  --s: 150px; /* control the size */
  --g: 10px;  /* control the gap */
  display: grid;
}
.gallery > img {
  grid-area: 1/1;
  width: var(--s);
  aspect-ratio: 1.15;
  object-fit: cover;
  clip-path: polygon(25% 0%, 75% 0%, 100% 50% ,75% 100%, 25% 100%, 0 50%);
  transform: translate(var(--_x,0), var(--_y,0));
}
.gallery > img:nth-child(1) { --_y: calc(-100% - var(--g)); }
.gallery > img:nth-child(7) { --_y: calc( 100% + var(--g)); }
.gallery > img:nth-child(3),
.gallery > img:nth-child(5) { --_x: calc(-75% - .87*var(--g)); }
.gallery > img:nth-child(4),
.gallery > img:nth-child(6) { --_x: calc( 75% + .87*var(--g)); }
.gallery > img:nth-child(3),
.gallery > img:nth-child(4) { --_y: calc(-50% - .5*var(--g)); }
.gallery > img:nth-child(5), 
.gallery > img:nth-child(6) { --_y: calc( 50% + .5*var(--g)); }
```
每个图像都由基于这些公式的--_x和变量转换。--_y只有第二张图片 ( nth-child(2)) 在任何选择器中未定义，因为它位于中心。  
**CSS 菱形网格**  
首先在 CSS 中定义一个 2×2 的图像网格：  
```
.gallery {
  --s: 150px; /* controls the size */

  display: grid;
  gap: 10px;
  grid: auto-flow var(--s) / repeat(2, var(--s));
  place-items: center;
}
.gallery > img {
  width: 100%; 
  aspect-ratio: 1;
  object-fit: cover;
}
```
然后设置旋转，将它们都旋转45deg，但方向相反。  
```
.gallery {
  transform: rotate(45deg);
}
.gallery > img {
  transform: rotate(-45deg);
}
```
向负方向旋转图像可防止它们与网格一起旋转，因此它们保持笔直。现在，应用 clip-path 从它们中剪出菱形。  
此时的图像并没有按我们的预期的间距排列，需要纠正图像的大小以使它们适合在一起。否则，它们的间距会很远，以至于看起来不像图像网格。  
```
.gallery > img {
  width: 141%; /* 100%*sqrt(2) = 141% */
  aspect-ratio: 1;
  object-fit: cover;
  transform: rotate(-45deg);
  clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
}
```
**三角形的 CSS 网格**  
找出clip-path我们想要的形状。对于这个网格，每个元素都有自己的clip-path值，而最后两个网格使用一致的形状。这一次，就像我们正在处理几个不同的三角形形状，它们组合在一起形成一个矩形的图像网格。  
使用以下 CSS 将它们放置在 3×2 网格中：  
```
.gallery {
  display: grid;
  gap: 10px; 
  grid-template-columns: auto auto auto; /* 3 columns */
  place-items: center;
}
.gallery > img {
  width: 200px; /* controls the size */
  aspect-ratio: 1;
  object-fit: cover;
}
/* the clip-path values */
.gallery > img:nth-child(1) { clip-path: polygon(0 0, 50% 0, 100% 100% ,0 100%); }
.gallery > img:nth-child(2) { clip-path: polygon(0 0, 100% 0, 50% 100%); }
.gallery > img:nth-child(3) { clip-path: polygon(50% 0, 100% 0, 100% 100%, 0 100%); }
.gallery > img:nth-child(4) { clip-path: polygon(0 0, 100% 0, 50% 100%, 0 100%); }
.gallery > img:nth-child(5) { clip-path: polygon(50% 0, 100% 100%, 0% 100%); }
.gallery > img:nth-child(6) { clip-path: polygon(0 0, 100% 0 ,100% 100%, 50% 100%); } }
```
最后一点是使中间列的宽度等于0消除图像之间的空间。在菱形网格中遇到了同样的间距问题，但对我们使用的形状采用了不同的方法：  
```
grid-template-columns: auto 0 auto;
```
**拼图风格的 CSS 网格**  
现在设置网格应该是小菜一碟，所以把注意力集中在mask上。需要两个渐变来创建最终的拼图形状。一个渐变创建一个圆形（绿色部分），另一个渐变创建红色区域并填充半圆白色区域。  
```
--g: 6px; /* controls the gap */
--r: 42px;  /* control the circular shapes */

background: 
  radial-gradient(var(--r) at left 50% bottom var(--r), green 95%, #0000),
  radial-gradient(calc(var(--r) + var(--g)) at calc(100% + var(--g)) 50%, #0000 95%, red)
  top/100% calc(100% - var(--r)) no-repeat;
```
两个变量控制形状。--g变量控制网格间隙，相对不是最重要的。重要的是考虑间隙之间如何正确放置我们的圆圈，以便在组装整个拼图时它们完美重叠。该--r变量则控制拼图形状的圆形部分的大小。  
然后我们使用相同的 CSS 值并针对不同的位置稍加调整来创建其他三个形状  
此时整体拼图形状好了，但没有按我们的预期重叠在一起。因为每个图像都被限制在它所在的网格单元中，所以现在形状有点混乱是对的   
需要通过增加图像的高度/宽度来创建溢出。必须增加第一个和第四个图像的高度，同时增加第二个和第三个图像的宽度。使用--r变量来增加它们。  
```
.gallery > img:is(:nth-child(1),:nth-child(4)) {
  width: 100%;
  height: calc(100% + var(--r));
}
.gallery > img:is(:nth-child(2),:nth-child(3)) {
  height: 100%;
  width: calc(100% + var(--r));
}
```
此时左边两张图片按预期展示了，但默认情况下，我们的图像要么在右侧（如果我们增加宽度）重叠，要么在底部（如果我们增加高度）重叠。但这不是我们想要的第二张和第四张图片。解决方法是在这两个图像上使用place-self: end  



参考:  
[那些你不知道的 CSS 自定义形状网格布局](https://mp.weixin.qq.com/s/ey73FvOeIgSUjaeDZc872Q)