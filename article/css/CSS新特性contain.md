# CSS新特性contain
contain 属性允许我们指定特定的 DOM 元素和它的子元素，让它们能够独立于整个 DOM 树结构之外。目的是能够让浏览器有能力只对部分元素进行重绘、重排，而不必每次都针对整个页面。  
**contain 语法**  
``` 
  /* No layout containment. */
  contain: none;
  /* Turn on size containment for an element. */
  contain: size;
  /* Turn on layout containment for an element. */
  contain: layout;
  /* Turn on style containment for an element. */
  contain: style;
  /* Turn on paint containment for an element. */
  contain: paint;

  /* Turn on containment for layout, paint, and size. */
  contain: strict;
  /* Turn on containment for layout, and paint. */
  contain: content;
```
## contain: size
contain: size: 设定了 contain: size 的元素的渲染不会受到其子元素内容的影响。  
``` 
// html
<div class="container">
   
</div>
// style
.container {
    width: 300px;
    padding: 10px;
    border: 1px solid red;
}

p {
    border: 1px solid #333;
    margin: 5px;
    font-size: 14px;
}
// JS
$('.container').on('click', e => {
    $('.container').append('<p>Coco</p>')
})
```
容器 .container 的高度是会随着元素的增加而增加的,给容器 .container 添加一个 contain: size，也就会出现上述说的：设定了 contain: size 的元素的渲染不会受到其子元素内容的影响，子元素的变化不再影响父元素的样式布局。
## contain: paint
设定了 contain: paint 的元素即是设定了布局限制，也就是说告知 User Agent，此元素的子元素不会在此元素的边界之外被展示，因此，如果元素不在屏幕上或以其他方式设定为不可见，则还可以保证其后代不可见不被渲染。  
设定了 contain: paint 的元素的子元素不会在此元素的边界之外被展示，这个特点有点类似与 overflow: hidden，也就是明确告知用户代理，子元素的内容不会超出元素的边界，所以超出部分无需渲染。  
``` 
// html
<div class="container">
    <p>Coco</p>
</div>
// style
.container {
    contain: paint;
    border: 1px solid red;
}

p{
    left: -100px;
}
使用 contain: paint， 如果元素处于屏幕外，那么用户代理就会忽略渲染这些元素，从而能更快的渲染其它内容。
```
## contain: layout
contain: layout 的元素即是设定了布局限制，也就是说告知 User Agent，此元素内部的样式变化不会引起元素外部的样式变化  
启用 contain: layout 可以潜在地将每一帧需要渲染的元素数量减少到少数，而不是重新渲染整个文档，从而为浏览器节省了大量不必要的工作，并显着提高了性能。  
使用 contain：layout，开发人员可以指定对该元素任何后代的任何更改都不会影响任何外部元素的布局，反之亦然。  
浏览器仅计算内部元素的位置（如果对其进行了修改），而其余DOM保持不变。因此，这意味着帧渲染管道中的布局过程将加快。 
实际上设定了 contain: layout 的指定元素，改元素的任何后代的任何更改还是会影响任何外部元素的布局  
##  contain: strict | contain: content
这两个属性稍微有点特殊，效果是上述介绍的几个属性的聚合效果：  
- contain: strict：同时开启 layout、style、paint 以及 size 的功能，它相当于 contain: size layout paint
- contain: content：同时开启 layout、style 以及 paint 的功能，它相当于 contain: layout paint

参考:  
[CSS新特性contain，控制页面的重绘与重排](https://juejin.cn/post/6958990366888607757?utm_source=gold_browser_extension)
