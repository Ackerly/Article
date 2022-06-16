# 使用 content-visibility 优化渲染性能
**何为 content-visibility**  
content-visibility：属性控制一个元素是否渲染其内容，它允许用户代理（浏览器）潜在地省略大量布局和渲染工作，直到需要它为止  
有几个常见的取值  
``` 
content-visibility: visible;
content-visibility: hidden;
content-visibility: auto;
```
- content-visibility: visible：默认值，没有任何效果，相当于没有添加 content-visibility，元素的渲染与往常一致。
- content-visibility: hidden：与 display: none 类似，用户代理将跳过其内容的渲染。（这里需要注意的是，跳过的是内容的渲染）
- content-visibility: auto：如果该元素不在屏幕上，并且与用户无关，则不会渲染其后代元素。

除 content-visibility 之外，还有一个与之配套的属性 -- contain-intrinsic-size  
contain-intrinsic-size：控制由 content-visibility 指定的元素的自然大小。  

**利用 content-visibility: hidden 优化展示切换性能**  
ontent-visibility: hidden，它通常会拿来和 display: none 做比较，但是其实它们之间还是有很大的不同的。  
假设有两个 DIV 包裹框：  
``` 
<div class="g-wrap">
    <div>1111</div>
    <div class="hidden">2222</div>
</div>
```
设置两个 div 为 200x200 的黑色块：  
``` 
.g-wrap > div {
    width: 200px;
    height: 200px;
    background: #000;
}
```
给其中的 .hidden 设置 content-visibility: hidden，看看会发生什么：  
``` 
.hidden {
    content-visibility: hidden;
}
```
这里添加了 content-visibility: hidden 之后，消失的只是添加了该元素的 div 的子元素消失不见，而父元素本身及其样式，还是存在页面上的  
如果我们去掉设置了 content-visibility: hidden 的元素本身的 width、height、padding、margin 等属性，则元素看上去就如同设置了 display: none 一般，在页面上消失不见了  
content-visibility: hidden 的作用是什么呢？  
设置了 content-visibility: hidden 的元素，其元素的子元素将被隐藏，但是，它的渲染状态将会被缓存。所以，当 content-visibility: hidden 被移除时，用户代理无需重头开始渲染它和它的子元素。  
如果我们将这个属性应用在一些一开始需要被隐藏，但是其后在页面的某一时刻需要被渲染，或者是一些需要被频繁切换显示、隐藏状态的元素上，其渲染效率将会有一个非常大的提升  
**利用 content-visibility: auto 实现懒加载或虚拟列表**  
content-visibility: auto 的作用是，如果该元素不在屏幕上，并且与用户无关，则不会渲染其后代元素。是不是与 LazyLoad 非常类似？  
假设存在这样一个 HTML 结构，含有大量的文本内容：  
``` 
<div class="g-wrap">
    <div class="paragraph">...</div>
    // ... 包含了 N 个 paragraph
    <div class="paragraph">...</div>
</div>
```
由于没有对页面内容进行任何处理，因此，所有的 .paragraph 在页面刷新的一瞬间，都会进行渲染  
现代浏览器愈加趋于智能，基于这种场景，其实我们非常希望对于仍未看到，仍旧未滚动到的区域，可以延迟加载，只有到我们需要展示、滚动到该处时，页面内容才进行渲染  
基于这种场景，content-visibility: auto 就应运而生了，它允许浏览器对于设置了该属性的元素进行判断，如果该元素当前不处于视口内，则不渲染该元素。  
基于上述的代码，只需要最小化，添加这样一段代码：  
``` 
.paragraph {
    content-visibility: auto;
}
```
对比下添加了 content-visibility: auto 和没有添加 content-visibility: auto 的两种效果下文本的整体高度：  
有着非常明显的差异，这是因为，设置了 content-visibility: auto 的元素，在非可视区域内，目前并没有被渲染，因此，右侧内容的高度其实是比正常状态下少了一大截的。  
由于下方的元素在滚动的过程中，出现在视口范围内才被渲染，因此，滚动条出现了明显的飘忽不定的抖动现象。（当然这也是使用了 content-visibility: auto 的一个小问题之一），不过明显可以看出，这与我们通常使用 JavaScript 实现的懒加载或者延迟加载非常类似。  
与懒加载不同的是，在向下滚动的过程中，上方消失的已经被渲染过且消失在视口的元素，也会因为消失在视口中，重新被隐藏。因此，即便页面滚动到最下方，整体的滚动条高度还是没有什么变化的。  
**content-visibility 是否能够优化渲染性能**  
对于一个存在巨量 HTML 内容的页面，往下翻，根本翻不到尽头,如果不对这个页面做任何处理，首次渲染可能需要好几秒时间，如果给这个页面的每个段落，添加上 content-visibility: auto，整体的耗时DOMContentLoaded 的时间骤降至了几百ms，而花费在 Rendering 上的，直接优化到了 百ms内  
因此，content-visibility: auto 对于长文本、长列表功能的优化是显而易见的。
**利用 contain-intrinsic-size 解决滚动条抖动问题**  
用 content-visibility: auto 处理长文本、长列表的时候。在滚动页面的过程中，滚动条一直在抖动，这不是一个很好的体验。  
contain-intrinsic-size：控制由 content-visibility 指定的元素的自然大小。  
如果我们不使用 contain-intrinsic-size，只对视口之外的元素使用 content-visibility: auto，那么视口外的元素高度通常就为 0。  
那么实际的滚动效果，滚动条就是抖动的，所以，我们可以同时利用上 contain-intrinsic-size，如果能准确知道设置了 content-visibility: auto 的元素在渲染状态下的高度，就填写对应的高度。如果无法准确知道高度，也可以填写一个大概的值  
如此之后，浏览器会给未被实际渲染的视口之外的 .paragraph 元素一个高度，避免出现滚动条抖动的现象  
**content-visibility 的一些其他问题**
由于该属性属于渐进增强一类的功能，即便失效，也完全不影响页面本身的展示。  
利用 content-visibility: auto 只能解决部分场景，在海量 DOM 的场景下的实际效果，还有待进一步的实测。  

参考:  
[使用 content-visibility 优化渲染性能](https://mp.weixin.qq.com/s/webv8u3M43Jy3BVCxwEQFg)
