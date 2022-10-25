# Clamp、Max和 Min CSS 函数的用例
**流体尺寸和定位**  
当容器的宽度变小时，我们希望缩小图像的大小以适应可用空间。我们可以通过使用宽度或高度的百分比值（例如：宽度：20%）来做到这一点，但这并没有给我们太多的控制权。  
我们希望能够有一个流体大小，它同时尊重最小值和最大值，这就是clamp来救援的地方！  
CSS:  
``` 
.section-image {
  width: clamp(70px, 80px + 15%, 180px);
}
```
通过设置最小、首选和最大宽度，图像将根据其容器宽度缩小或增长，这是由于使用了固定值和百分比 80px + 15% 的混合。  

**装饰元素**  
两侧有两个装饰元素。在移动设备上，它们会占用太多空间，因此我们只想展示其中的一小部分。  
CSS:  
``` 
.decorative--1 {
  left: 0;
}

.decorative--2 {
  right: 0;
}

@media (max-width: 600px) {
  .decorative--1 {
    left: -8rem;
  }

  .decorative--2 {
    right: -8rem;
  }
}
```
虽然这可行，但我们可以使用带有 CSS clamp() 函数的无媒体查询解决方案。  
CSS:  
``` 
@media (max-width: 600px) {
  .decorative--1 {
    left: clamp(-8rem, -10.909rem + 14.55vw, 0rem);
  }

  .decorative--2 {
    right: clamp(-8rem, -10.909rem + 14.55vw, 0rem);
  }
}
```
剖析一下上面的 CSS:  
- 设置最小左偏移为-8rem，最大值为0rem
-  CSS clamp() 来决定首选值并尊重我们设置的最小值和最大值。

**流体高度**  
通过媒体查询或使用视口单元来改变高度  
CSS
``` 
.hero {
  min-height: 250px;
}

@media (min-width: 800px) {
  .hero {
    min-height: 500px;
  }
}
```
混合使用固定值和视口单位，需要注意不要在较大的视口上设置很大的高度，然后，设置一个最大高度。  
CSS  
``` 
.hero {
  min-height: calc(350px + 20vh);
}

@media (min-width: 2000px) {
  .hero {
    min-height: 600px;
  }
}
```
使用 CSS clamp()，可以只用一个 CSS 声明来设置最小、首选和最大高度。  
``` 

.hero {
  min-height: clamp(250px, 50vmax, 500px);
}
```
调整屏幕大小时，高度会根据视口宽度逐渐变化。在上面的示例中，50vmax 表示“视口最大尺寸的 50%。  
**加载条**  
条形按钮应该从左到右进行动画处理，反之亦然。在 CSS 中，按钮可以绝对定位在左侧。  
CSS:  
``` 
.loading-thumb {
  left: 0%;
}
```
可以使用 CSS calc() 减去按钮宽度，它会起作用，但这不是 100% 灵活的。  
CSS:  
``` 
.loading-thumb {
  /* 40px represents the thumb width. */
  left: calc(100% - 40px);
}
```
探索如何使用 CSS 变量和比较函数来改进 CSS  
CSS:  
``` 
.loading-thumb {
  --loading: 0%;
  --loading-thumb-width: 40px;
  position: absolute;
  top: 4px;
  left: clamp(
    0%,
    var(--loading),
    var(--loading) - var(--loading-thumb-width)
  );
  width: var(--loading-thumb-width);
  height: 16px;
}
```
上述 CSS 的工作原理：  
- 将最小值设置为 0%。
- 首选值是 --loading CSS 变量的当前值
- 最大值表示当前加载减去按钮宽度

圆圈必须在最右侧结束，如果我们不注意这一点，它最终会吹出手柄宽度的一半
在这种情况下，我们可以使用 CSS clamp() 函数  
``` 
.loading-progress {
  width: clamp(10px, var(--loading), var(--loading) - 10px);
}
```
最小值等于半圆宽度，优选值是当前加载百分比，最大值是半圆减去当前百分比的结果。  
**动态线分隔符**  
两个部分之间有一个行分隔符,移动设备上，该分隔符应变为水平  
使用边框和弹性框，这个方法是带有边框的伪元素可以扩展以填充垂直和水平状态的可用空间。  
``` 
.section {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.section:before {
  content: "";
  border: 1px solid #d3d3d3;
  align-self: stretch;
}

@media (min-width: 700px) {
  .section {
    align-items: center;
    flex-direction: row;
  }
}
```
通过使用 CSS clamp 来实现  
``` 
.section {
  --breakpoint: 400px;
  display: flex;
  flex-wrap: wrap;
}

.section:before {
  content: "";
  border: 2px solid lightgrey;
  width: clamp(0px, (var(--breakpoint) - 100%) * 999, 100%);
}
```
剖析一下上面的 CSS：  
- 0px：最小值，用于垂直分隔符。它为零，因为我们使用的是 CSS 边框
- (var(--breakpoint) - 100%) * 999 根据视口宽度在 0px 或 100% 之间切换

**条件边界半径**  
CSS max() 比较函数根据视口宽度将卡片的半径从 0px 切换到 8px  
``` 
.card {
  border-radius: max(
    0px,
    min(8px, calc((100vw - 4px - 100%) * 9999))
  );
}
```
剖析一下上面的 CSS：  
- max() 函数，用于比较 0px 和 min() 的计算值，它将选择较大的值。
- min() 函数在 8px 和 calc((100vw - 4px - 100%) * 9999) 的计算值之间进行比较，这将导致非常大的正数或负数
- 9999 是一个很大的数字，强制该值为 0px 或 8px

有了上面的内容，当卡片占据整个视口宽度时，它的半径为零，或者在更大的屏幕上为 8px。  

**CSS 文章标题**  
构建CSS 文章标题时，需要一种方法来为内容添加动态填充，同时，在较小的视口上保持最小值。  
文章标题不包含在包装元素中，因此我们需要一种方法来模拟内容实际上被包装并与下面的内容对齐。  
在 CSS 中使用以下公式的方法：  
> 动态填充 = (视口宽度 - 包装宽度) / 2

CSS
``` 
:root {
  --wrapper-width: 1100px;
  --wrapper-padding: 16px;
  --space: max(
    1rem,
    calc(
      (
          100vw - calc(var(--wrapper-width) - var(--wrapper-padding) *
                2)
        ) / 2
    )
  );
}

.article-header {
  padding-left: var(--space);
}
```
最小填充为 1rem，然后，它将根据视口宽度动态变化  

**间距**  
需要根据视口宽度更改组件或网格的间距。不带 CSS 比较功能  
``` 
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: min(2vmax, 32px);
}
```

原文:  
[Clamp()、Max() 和 Min() CSS 函数的用例](https://mp.weixin.qq.com/s/e-0UctN4nnRvHwH1m0JUTw)
