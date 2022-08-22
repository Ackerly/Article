# web reflow
网页渲染会经历 DOM -> CSSOM -> Layout(重排 or reflow) -> Paint(重绘) -> Composite(合成)  
排除 JS、DOM、CSSOM、Composite 可能导致的性能问题外，剩下的就是我们这次关注的重点，reflow 了。从顺序上可以看出来，重排后一定重绘，而重绘不一定触发重排。  
**概述**  
什么时候会触发 Layout(reflow) 呢？一般来说，当元素位置发生变化时就会。但也不尽然，因为浏览器会自动合并更改，在达到某个数量或时间后，会合并为一次 reflow，而 reflow 是渲染页面的重要一步，打开浏览器就一定会至少 reflow 一次，所以我们不可能避免 reflow。  
为什么要注意 reflow 导致的性能问题呢？这是因为某些代码可能导致浏览器优化失效，即明明能合并 reflow 时没有合并，这一般出现在我们用 js API 访问某个元素尺寸时，为了保证拿到的是精确值，不得不提前触发一次 reflow，即便写在 for 循环里。  
当然也不是每次访问元素位置都会触发 reflow，在浏览器触发 reflow 后，所有已有元素位置都会记录快照，只要不再触发位置等变化，第二次开始访问位置就不会触发 reflow。现在要解释的是，这个 ”触发位置等变化“，到底有哪些？  

**获得盒子模型信息**  
- elem.offsetLeft, elem.offsetTop, elem.offsetWidth, elem.offsetHeight, elem.offsetParent
- elem.clientLeft, elem.clientTop, elem.clientWidth, elem.clientHeight
- elem.getClientRects(), elem.getBoundingClientRect()

获取元素位置、宽高的一些手段都会导致 reflow，不存在绕过一说，因为只要获取这些信息，都必须 reflow 才能给出准确的值  

**滚动**  
- elem.scrollBy(), elem.scrollTo()
- elem.scrollIntoView(), elem.scrollIntoViewIfNeeded()
- elem.scrollWidth, elem.scrollHeight
- elem.scrollLeft, elem.scrollTop 访问及赋值

对 scrollLeft 赋值等价于触发 scrollTo，所有导致滚动产生的行为都会触发 reflow，目前主要推测是滚动条出现会导致可视区域变窄，所以需要 reflow。  

**focus()**  
可以根据源码看一下注释，主要是这一段：  
``` 
// Ensure we have clean style (including forced display locks).
GetDocument().UpdateStyleAndLayoutTreeForNode(this)
```
即在聚焦元素时，虽然没有拿元素位置信息的诉求，但指不定要被聚焦的元素被隐藏或者移除了，此时必须调用 UpdateStyleAndLayoutTreeForNode 重排重绘函数，确保元素状态更新后才能继续操作  
还有一些其他 element API：  
- elem.computedRole, elem.computedName
- elem.innerText (源码)

innerText 也需要重排后才能拿到正确内容  

**获取 window 信息**  
- window.scrollX, window.scrollY
- window.innerHeight, window.innerWidth
- window.visualViewport.height / width / offsetTop / offsetLeft (源码)

和元素级别一样，为了拿到正确宽高和位置信息，必须重排。  

**document 相关**  
- document.scrollingElement 仅重绘
- document.elementFromPoint
elementFromPoint 因为要拿到精确位置的元素，必须重排

**Form 相关**  
- inputElem.focus()
- inputElem.select(), textareaElem.select()
focus、select 触发重排的原因和 elem.focus 类似。

**鼠标事件相关**  
- mouseEvt.layerX, mouseEvt.layerY, mouseEvt.offsetX, mouseEvt.offsetY (源码)

鼠标相关位置计算，必须依赖一个正确的排布，所以必须触发 reflow。  
**getComputedStyle**  
getComputedStyle 通常会导致重排和重绘，是否触发重排取决于是否访问了位置相关的 key 等因素  

**Range 相关**  
- range.getClientRects(), range.getBoundingClientRect()

获取选中区域的大小，必须 reflow 才能保障精确性  

**SVG**  
大量 SVG 方法会引发重排，就不一一枚举了，总之使用 SVG 操作时也要像操作 dom 一样谨慎。

**contenteditable**  
被设置为 contenteditable 的元素内，包括将图像复制到剪贴板在内，大量操作都会导致重排。

**repaint-reflow-restyle**  
repaint-reflow-restyle 提到现代浏览器会将多次 dom 操作合并，但像 IE 等其他内核浏览器就不保证有这样的实现了，因此给出了一个安全写法：  
``` 
// bad
var left = 10,
    top = 10;
el.style.left = left + "px";
el.style.top  = top  + "px";
 
// better 
el.className += " theclassname";
 
// or when top and left are calculated dynamically...
 
// better
el.style.cssText += "; left: " + left + "px; top: " + top + "px;";
```
比如用一次 className 的修改，或一次 cssText 的修改保证浏览器一定触发一次重排。但这样可维护性会降低很多，不太推荐。  
**avoid large complex layouts**  
avoid large complex layouts 重点强调了读写分离，首先看下面的 bad case：  
``` 
function resizeAllParagraphsToMatchBlockWidth() {
  // Puts the browser into a read-write-read-write cycle.
  for (var i = 0; i < paragraphs.length; i++) {
    paragraphs[i].style.width = box.offsetWidth + 'px';
  }
}
```
在 for 循环中不断访问元素宽度，并修改其宽度，会导致浏览器执行 N 次 reflow。  
虽然当 JavaScript 运行时，前一帧中的所有旧布局值都是已知的，但当你对布局做了修改后，前一帧所有布局值缓存都会作废，因此当下次获取值时，不得不重新触发一次 reflow。  
而读写分离的话，就代表了集中读，虽然读的次数还是那么多，但从第二次开始就可以从布局缓存中拿数据，不用触发 reflow 了。  
另外还提到 flex 布局比传统 float 重排速度快很多（3ms vs 16ms），所以能用 flex 做的布局就尽量不要用 float 做。  
**really fixing layout thrashing**  
really fixing layout thrashing 提到了用 fastdom 实践读写分离：  
``` 
ids.forEach(id => {
  fastdom.measure(() => {
    const top = elements[id].offsetTop
    fastdom.mutate(() => {
      elements[id].setLeft(top)
    })
  })
})
```
fastdom 是一个可以在不分离代码的情况下，分离读写执行的库，尤其适合用在 reflow 性能优化场景。每一个 measure、mutate 都会推入执行队列，并在 window.requestAnimationFrame 时机执行。  


参考:  
[精读《web reflow》](https://mp.weixin.qq.com/s/uaNTKwdXDSVLixF4Rz1H3Q)
