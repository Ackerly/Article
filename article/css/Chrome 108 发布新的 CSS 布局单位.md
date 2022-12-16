# Chrome 108 ：发布新的 CSS 布局单位
**关于移动端适配，你必须要知道的**  
在响应式布局中，我们经常会用到两个视口相关的单位：  
- vw(Viewport's width)：1vw 等于视觉视口的 1%
- vh(Viewport's height) : 1vh 为视觉视口高度的 1%

另外还有两个相关的衍生单位：  
- vmin : vw 和 vh 中的较小值
- vmax : 选取 vw 和 vh 中的较大值

如果我们将一个元素的宽度设置为 100vw 高度设置为 100vh，它将完全覆盖视觉视口  
这些单位有很好的浏览器兼容性，也在桌面端布局中得到了很好的应用。  
但是，在移动设备上的表现就差强人意了，移动设备的视口大小会受动态工具栏（例如地址栏和标签栏）存在与否的影响。视口大小可能会更改，但 vw 和 vh 的大小不会。因此，尺寸过大的 100vh 元素可能会从视口中溢出。  
当网页向下滚动时，这些动态工具栏可能又会自动缩回。在这种状态下，尺寸为 100vh 的元素又可以覆盖整个视口。  
为了解决这个问题，CSS 工作组规定了视口的各种状态。  
- Large viewport（大视口）：视口大小假设任何动态工具栏都是收缩状态。
- Small Viewport（小视口）：视口大小假设任何动态工具栏都是扩展状态。

新的视口也分配了单位：  
- 代表 Large viewport 的单位以 lv 为前缀：lvw、lvh、lvi、lvb、lvmin、lvmax。
- 代表 Small Viewport 的单位以 sv 为前缀：svw、svh、svi、svb、svmin、svmax。

> 除非调整视口本身的大小，否则这些视口百分比单位的大小是固定的。

除了 Large viewport 和 Small Viewport ，还有一个  Dynamic viewport（动态视口）  
- 当动态工具栏展开时，动态视口等于小视口的大小。
- 当动态工具栏被缩回时，动态视口等于大视口的大小。

相应的，它的视口单位以 dv 为前缀：dvw, dvh, dvi, dvb, dvmin, dvmax。  


原文:  
[Chrome 108 ：发布新的 CSS 布局单位](https://mp.weixin.qq.com/s/qqrIMrEbSa19WyxVryudfA)
