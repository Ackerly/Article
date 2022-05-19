# 10个CSS技巧
## 打字效果
通过 steps() 属性来实现分割文本的效果  
首先，你必须指定 step() 中传入的数量  
第二步，我们使用 @keyframes 去声明什么时候开始执行动画。  
> 如果你在文本 Typing effect for text 后面添加内容，而不改变 step() 中的数字，将不会产生这种效果。

## 透明图片阴影效果
否使用过 box-shadow 为透明的图片添加阴影，却让其看起来像添加了一个边框一样？然而解决方案是使用 drop-shadow。  
drop-shadow 的工作方式是，其遵循给给定图片的 Alpha 通道。因此阴影是基于图片的内部形状，而不是显示在图片外面。  
## 自定义 Cursor
不需要强迫你站点访问者使用独特的光标。至少，不是出于用户体验的目的。不过，关于 cursor 属性要说明的是，它可以让你展示图片，这相当于以照片的格式显示提示信息。  
一些用户案例，包括比较两个不同的照片，你无需在视图窗口渲染这些照片。比如：cursor 属性可以用在你的设计中，节省空间。因为你可以在特定的 div 元素中锁定特定的光标，所以在此 div 这外可以无效。

## 使用 attr() 展示 tooltip
attr() 属性工作的方式:  
- 使用 tooltip class 去标志哪个元素需要展示 tooltip 信息。然后为该元素添加你喜欢的样式，
- 创建一个 :before 伪元素，它将包含内容 content，指向特定的 attr()。这里指 attr(tooltip-data)
- 创建一个 :hover 伪类，当用户鼠标移动道元素上时，它将设置 opacity 为 1

可以包含自定义的样式。这取决于你设定的 tooltp 的数据，你也许需要调整其宽度或者边距。一旦你设定了 tooptip-data arrt() 类，你可以在你设计的其他部分应用。  
## 纯 CSS 实现核算清单
使用 checkbox 输入类型，加上一个 :checked 伪类。当 :checked 返回 true 的情况时，使用 transform 属性更改状态。  
可以使用这种方法实现各种目标。比如，当用户点点击指定的复选框时候，切花到隐藏其内容。在输入 input 类型的单选和复选框使用，当然，这也可以应用到 <option> 和 <select> 元素。  
## 使用 :is() 和 :where() 添加元素样式
:is() 和 :where() 属性可以用于同时设置多种设计元素的样式。但是，更重要的是，你可以使用这些属性去查询你需单独处理的元素。
## 使用关键帧实现手风琴下拉效果
``` 
// HTML
<main>
  <details open>
    <summary>Accordion Tab #1</summary>
    <div class="tab-content">
      <p>your text goes here</p>
    </div>
  </details>

    <details>
    <summary>Accordion Tab #2</summary>
    <div class="tab-content">
      <p>your text goes here</p>
    </div>
  </details>
      <details>
    <summary>Accordion Tab #3</summary>
    <div class="tab-content">
      <p>your text goes here</p>
    </div>
  </details>
   </main>
   
   // CSS
   /* .tab-content can be styled as you like */
main {
  max-width: 400px;
  margin: 0 auto;
}
p {
    text-align: justify;
    font-family: monospace;
    font-size: 13px;
}
summary {
  font-size: 1rem;
  font-weight: 600;
  background-color: #f3f3f3;
  color: #000;
  padding: 1rem;
  margin-bottom: 1rem;
  outline: none;
  border-radius: 0.25rem;
  cursor: pointer;
  position: relative;
}
details[open] summary ~ * {
  animation: sweep .5s ease-in-out;
}
@keyframes sweep {
  0%    {opacity: 0; margin-top: -10px}
  100%  {opacity: 1; margin-top: 0px}
}
details > summary::after {
  position: absolute;
  content: "+";
  right: 20px;
}
details[open] > summary::after {
  position: absolute;
  content: "-";
  right: 20px;
}
details > summary::-webkit-details-marker {
  display: none;
}
```

## 侧边栏的 Hover 效果
以使用 CSS 就可以实现一个动态 Hover 效果的侧边栏呢？当然，这得多亏 transform 和 :hover 属性。  
## 使用 first-letter 实现首字母大写
:first-letter 伪类去实现首字母大写的效果。这个类可以让我们更自由的添加样式。所以，你可以调整大写字母的样式以符合你的站点设计风格。  
当特定元素在页面中第一次出现，我们可以使用 first-of-type 单独进行添加样式。  
## 使用 ::before 添加按钮的图标
``` 
// html 
<div class="card">
  <div class="card-body">
    <a href="" target="_blank" class="btn btn-docu" rel="noopener">Documentation</a>
  </div>
</div>
// CSS
.card .card-body .btn {
  display: block;
  width: 200px;
  height: 48px;
  line-height: 48px;
  background-color: blue;
  border-radius: 4px;
  text-align: center;
  color: #fff;
  font-weight: 700;
}
.card .card-body .btn-docu:before {
content:"\0000a0";
display:inline-flex;
height:24px;
width:24px;
line-height:24px;
margin:0px 10px 0px 0px;
position:relative;
top:0px;
left:0px;
background:url(https://stackdiary.com/docu.svg) no-repeat left center transparent;
background-size:100% 100%;
```

参考:
[10 个不错的 CSS 小技巧](https://juejin.cn/post/7089997204252786702)
