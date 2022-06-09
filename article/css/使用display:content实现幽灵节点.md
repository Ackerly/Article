# 使用display:content实现幽灵节点
**基本用法**  
设置了该属性值的元素本身将不会产生任何盒子，但是它保留其子代元素的正常展示。  
看个简单的例子。有如下简单三层结构：  
``` 
<div class="container">
    <div class="wrap">
        <div class="inner"></div>
    </div>
</div>
```
简单的 CSS 如下：  
``` 
.container {
    width: 200px;
    height: 200px;
    background: #bbb;
}

.wrap {
    border: 2px solid red;
    padding: 20px;
    box-sizing: border-box;
}

.inner {
    border: 2px solid green;
    padding: 20px;
    box-sizing: border-box;
}
```
给中间层的容器添加上 display: contents  
``` 
<div class="container">
    <div class="wrap" style="display: contents">
        <div class="inner"></div>
    </div>
</div>
```
没有了中间层的 border: 2px solid red 的红色边框，整个 .wrap div 好像不存在一样，但是它的子元素却是正常的渲染了。  
这个属性适用于那些充当遮罩（wrapper）的元素，这些元素本身没有什么作用，可以被忽略的一些布局场景。也就是幽灵 DOM 节点  
**充当无语义的包裹框**  
写 DOM 结构时，经常需要输出一段模板，或者需要一些无语义的幽灵节点  
``` 
return (
    <div class="wrap">
        <h2>Title</h2>
        <div>...</div>
    </div>
)
```
只是想输出 .wrap div 内的内容，但是由于框架要求，输出的 JSX 模板必须包含在一个父元素之下，所以不得已，需要添加一个 .wrap 进行包裹，但是这个 .wrap 本身是没有任何样式的  
如果输出的元素是要放在其他 display: flex、display: grid 容器之下，加了一层无意义的 .wrap 之后，整个布局又需要重新进行调整  
一种方法是使用框架提供的容器 <React.Fragment>，它不会向页面插入任何多余节点  
> 在 Vue 中类似的是 <template> 元素， <template> 也是不会被渲染在 DOM 树中，查看页面结构也无法看到，但是 display: contents 是存在于页面结构中的，只是没有生成任何盒子。

在 Vue 中类似的是 <template> 元素， <template> 也是不会被渲染在 DOM 树中，查看页面结构也无法看到，但是 display: contents 是存在于页面结构中的，只是没有生成任何盒子。  
``` 
return (
    <div class="wrap" style="display: contents">
        <h2>Title</h2>
        <div>...</div>
    </div>
)
```
它既起到了包裹的作用，但是在实际渲染中，这个 div 其实没有生成任何盒子，一举两得。并且像一些 flex 布局、grid 布局，也不会受到影响  

**让代码更加符合语义化**  
页面上充斥了大量的可点击按钮，或者点击触发相应功能的文字等元素。但是，从语义上而言，它们应该是一个一个的 <button>，但是实际上，更多时候我们都是使用了 <p>、<div>、<a> 等标签进行了模拟，给他们加上了相应的点击事情而已。  
像是下面这样，虽然没什么问题，但是相对而言不那么符合语义化：  
``` 
<p class="button">
    Button
</p>
<p class="button">
    Click Me
</p>
```
``` 
button {
    width: 120px;
    line-height: 64px;
    text-align: center;
    background-color: #ddd;
    border: 2px solid #666;
}
```
使用 <button> 的原因有很多，<button> 相对 div 而言没那么好控制，且会引入很多默认样式。但是，有了 display: contents，我们可以让我们的代码既符合语义化，同时不需要去解决 <button> 带来的一些样式问题：  
``` 
<p class="button">
    <button style="display: contents">
        Button
    </button>
</p>
<p class="button">
    <button style="display: contents">
        Click Me
    </button>
</p>
```
添加了 <button style="display: contents">Click Me</button> 的包裹，不会对样式带来什么影响，button 也不会实际渲染在页面结构中，但是页面的结构语义上好了不少  

**在替换元素及表单元素中一些有意思的现象**  
display: contents 并非在所有元素下的表现都一致。  
对于可替换元素及大部分表单元素，使用 display: contents 的作用类似于 display: none  
也就是说对于一些常见的可替换元素、表单元素：
- <br>
- <canvas>
- <object>
- <audio>
- <iframe>
- <img>
- <video>
- <frame>
- <input>
- <textarea>
- <select>

作用了 display: contents 相当于使用了 display: none ，元素的整个框和内容都没有绘制在页面上  
<button> 的一些异同  
与其他表单元素不一样，正常而言，添加了 display: contents 相当于被隐藏，不会被渲染。但是实际运用过程中发现，<button></button> 如果包裹了内容，其一些可继承样式还是会被子内容继承。这个实际使用的过程中需要注意一下  



参考:  
[冷知识！使用 display: contents 实现幽灵节点？](https://mp.weixin.qq.com/s/DhkQNO8Hv1zZP9Fu7uSw-g)
