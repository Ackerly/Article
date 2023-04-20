# 别整一坨 CSS 代码了，试试这几个实用函数
## Clamp(), Max(), 和 Min() 函数
clamp() 函数的作用是把一个值限制在一个上限和下限之间，当这个值超过最小值和最大值的范围时，在最小值和最大值之间选择一个值使用。它接收三个参数：最小值、首选值、最大值。  

**流体的尺寸和定位**  
当容器的宽度变小时，我们要缩小图片的尺寸，这样才不会变形。一般使用百分比单位来解决，如 width: 20%，但是这种方式没有给我们太多的控制。  
我们希望能够有一个流体尺寸，要求有最小值和最大值，这就是 clamp 出场的地方。  
``` 
.section-image {
  width: clamp(70px, 80px + 15%, 180px);
}
```
**装饰性元素**  
有时候，我们需要在页面边角加一些修饰元素，该修饰元素需要具有响应式  
为了做到这，我们可以使用媒体查询：  
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
```
虽然这样做可以，但我们可以 clamp()函数，这样更简洁：  
``` 
 .decorative--1 {
    left: clamp(-8rem, -10.909rem + 14.55vw, 0rem);
  }

  .decorative--2 {
    right: clamp(-8rem, -10.909rem + 14.55vw, 0rem);
  }
```

**流体高度**  
有时候，我们页面的主区的高度需要根据视口大小而变化。这种场景，我们倾向于通过媒体查询或使用视口单位来改变这种情况。  
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
也可以混合使用固定值和视口单位:  
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
但需要注意在较大的视口上高度不能太过高，所以我们需要设置一个最大高度,使用CSS clamp()，我们可以只用一个CSS声明来设置最小、首选和最大高度。  
``` 
.hero {
  min-height: clamp(250px, 50vmax, 500px);
}
```
当调整屏幕大小时，我们会看到，高度会根据视口宽度逐渐改变。在上面的例子中，50vmax表示着视口最大尺寸的 50%。  

**Loading Bar**  
进度条一般是从左到右一个加载过程，在 CSS 中，我们可以定位在左边：  
``` 
.loading-thumb {
  left: 0%;
}
```
为了将进度条定位到最右边，可以使用 left: 100%，但这会带来一个问题。进度条会跑到容器外：  
``` 
.loading-thumb {
  left: 100%;
}
```
这是正常的情况，100% 是从进度条的末端开始的，而进度条本身也有自己的宽度，所以实际宽度会大于容器的宽度。  
可以使用 calc() 来减去的进度条宽度，这样就可以了，但这并不是100%有效：  
``` 
.loading-thumb {
  /* 40px represents the thumb width. */
  left: calc(100% - 40px);
}
```
如何利用CSS变量和比较函数来更好地实现：  
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
上面的步骤如下：
1. 首先，我们设定一个最小值为 0%
2. 首选值是 --loading CSS变量的当前值
3. 最大值代表当前的加载量减去进度条件的宽度

这里的CSS clamp()为我们提供了这个组件的三种不同的状态信息，这个方案很 nice  
不仅如此，我们还可以以相同的方式来处理不同UI  
``` 
.loading-progress {
  width: clamp(10px, var(--loading), var(--loading) - 10px);
}
```
最小值等于圆圈宽度的一半，首选值是当前的加载百分比，最大值是当前百分比与圆圈一半的减去结果。  
**动态分割器**  
在两个区域之间有一个行分隔符,在移动端上，这个分隔符应该变成水平的  
解决方案是使用一个边框和flex。思路是，边框作为伪元素，以填补垂直和水平状态的可用空间：  
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
可以使用 clamp 而不需要媒体查询的解决方案：  
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

作者：王大冶
链接：https://juejin.cn/post/7147849664518160415
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
剖析一下上面的CSS:  
- 0px：最小值，用于垂直分隔符。它的值是 0，因为我们使用的是一个CSS边框
- (var(--breakpoint) - 100%) * 999 是一个个切换器，根据视口宽度在 0px或 100% 之间切换

**动态 border Radius**  
使用CSS max()函数，根据视口宽度，将卡片的border-radius 从 0px 切换到 8px。  
``` 
.card {
  border-radius: max(
    0px,
    min(8px, calc((100vw - 4px - 100%) * 9999))
  );
}
```
剖析一下上面的CSS:  
- 有一个 max() 函数，在 0px 和 min()的计算值之间进行比较，并选择较大的值。
- min() 函数在 8px 和 calc((100vw - 4px - 100%) * 9999 的计算值之间进行比较,这会得到一个非常大的正数或负数。
- 9999 是一个很大的数字，这样 min 的值都是 8px

**间距**  
我们可能需要根据视口宽度来改变一个组件或一个网格的间距。有了CS函数就不一样了,我们只需要设置一次。  
``` 
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: min(2vmax, 32px);
}
```


原文:  
[别整一坨 CSS 代码了，试试这几个实用函数](https://juejin.cn/post/7147849664518160415)
