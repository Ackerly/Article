# CSS流与宽高
## 流  
“流” 跟现实世界的 “水流” 有异曲同工的表现。所谓 “流”，就是 CSS 世界中引导元素排列和定位的一条看不见的 “水流”。  
> div作为块级元素就像是铺满容器的水，注意是铺满；而 <span> 作为内联元素就像是漂浮在水中的木头。如果没有人为的干预（比如魔鬼 float），元素总是会按照既定的流（块级元素自上而下，行内元素从左到右），有顺序有组织地排列

**流体布局**  
既然流是布局的机制，那么利用流的机制和特性就可以实现流体布局。使用流体布局从一定程度上可以帮助我们编写精简且巧妙的 CSS ，保持布局的强扩展性和韧性  

**块级元素和内联元素**  
块级元素：block-level element  
常见的块级元素有 <div>、<li>、<table>、<p>,、<h1> - <h6> 等，需要注意是，“块级元素” 和 “display: block” 不是一个概念。这里需要注意一下块级元素的基本特征：一个水平流上只能单独显示一个元素，多个块级元素则换行显示  
除此之外，块级元素还有可以控制高度、行高、以及宽度默认为包含该块级容器的 100%。而在这些列举的块级元素中 <li> 元素默认的 display 值是 list-item，<table> 元素默认的 display 值是 table，但是它们均是 “块级元素”，因为它们都符合了块级元素的基本特征  
下面就来仔细的看看上面提到的 “display: block”、“display:list-item”、“display: table”：  

> display: block
> 1. 如果不指定宽高，默认会继承父元素的宽度，并且独占一行，即使宽度有剩余也会独占一行。例子中，宽度继承于父元素 body。
> 2. 高度一般以子元素撑开的高度为准，当然也可以自己设置宽度和高度。例子中高度被两个 <p> 子元素撑开

> display: list-item
默认会把元素作为列表显示，要完全模仿列表的话还需要加上 list-style-position，list-style-type。
“盒子” 魔术：为什么 list-item 元素会出现项目符号？所有的 “块级元素” 都有一个 “主块级盒子”，list-item 除此之外还有一个 “附加盒子”。list-item 元素会出现项目符号是因为生成了一个附加的盒子，学名 “标记盒子”（marker box），专门用来放圆点、数字这些项目符号。

> display: table
作为块级表格来显示（类似 table），表格前后带有换行符。使用基于表格的 CSS 布局，使我们能够轻松定义一个单元格的边界、背景等样式， 而不会产生因为使用了 table 那样的制表标签所导致的语义化问题

内联元素：inline element  
与块级元素负责结构不同，内联元素负责内容。比如 <a>、<span>、<i> 都是常见的内联元素。内联元素最大的特点就是：可以和文字在一行显示，除此之外，它的高，行高及外边距和内边距不可改变  
每个元素都有两个盒子，外在盒子和内在盒子。外在盒子负责让元素可以一行显示，还是只能换行显示；内在盒子负责宽高、内容呈现什么的，也叫容器盒子  
按照 display 的属性值不同，值为 block 的元素的盒子实际由 外在的 “块级盒子” 和 内在的 “块级容器盒子” 组成，值为 inline-block 的元素则由 外在的 “内联盒子” 和 内在的 “块级容器盒子” 组成，值为 inline 的元素则内外均是 “内联盒子”  

## 流与 width/height
width/height 作用在内在盒子，也就是 “容器盒子”。  
**width: auto**  
宽的默认值是 auto，至少包含了以下 4 种不同的宽度表现：  
1. 充分利用可用空间，fill-available。比如像 <div> 这样的块级元素，它们的宽度默认是包含与它们的父级容器宽度的 100%。  
2. 收缩与包裹，fit-content。指的是父元素的宽度会收缩到和内部元素宽度一样。比如浮动元素，position 为 abosolute/fixed，inline-block 等。  
3. 收缩到最小，min-content，指的是内部元素最大的最小宽度，如 table-layout 为 auto 的表格
4. 超出容器限制。内容超出了父容器，如果明确设定 width 或者内联元素开启了 white-space: nowrap 属性，一般都不会出现这个情况

**width: 100%，失去流动性的宽度**
尺寸超出的原因是，在标准的盒子模型下，元素的宽度 = 内容宽度 + 内边距 + 边框宽度。  
给子元素 <a> 标签设置了 width: 100%，此时的 内容宽度 已经等于外元素的宽度，所以超出的尺寸是设置的 margin 和 padding。  
去掉 margin 和 padding 后（其实这里改变 box-sizing 从 content-box 到 border-box 也可以解决）
所以，width: 100% 就不像 “水流” 那样完全利用容器空间，即所谓的 “流动性丢失”。  

**width 值作用的细节**  
当我们给一个 <div> 元素设定宽度的时候，这个宽度是如何作用到它上面的呢？比如在 div { width: 100px; } 中 100px 的宽度是如何作用到这个 <div> 元素上的。  
当我们在谈论它们是在一行还是换行显示时，实际上是谈论的外在盒子。而内在盒子实际是负责了元素的宽高和内容。  
内在盒子由内到外又可分为：content box、padding box、border box 和 margin box。  
content box 是环绕着 content 给定宽高的矩形，所以 width: 100px 作用在了 content box 上  
但这种宽设定却让流动性消失了，当我们给定元素设定 width: auto，元素的宽就会 “自适应” 地铺满容器，而给定了 width 值会让这种流动性消失。所以 “无宽度” 准则会让布局更灵活、容错性更高。从另一方面来说，如果没有精准去计算 border、padding 和 content 的宽度，很容易因为空间不足，导致页面布局错位的问题。  
**宽度分离原则**  
“宽度分离原则”，就是 CSS 中的 width 属性不与影响宽度的 padding/border（有时候包括 margin）属性共存，也就是不能出现以下的组合：  
``` 
first-div {
  width: 100px;
  border: 1px solid;
}

/* 或者 */

div {
  width: 100px;
  padding: 20px;
}
```
假如想在第一个 div 上添加 padding，那么加上 padding: 20px; 的属性：  
``` 
.first-div {
  width: 100px;
  border: 1px solid;
  padding: 20px;
}
```
在未加上 padding 之前，这个 div 的宽高是 102px，加上 padding 后变成了 142px，比之前的大了 40px，显然布局很容易出问题。为了不影响之前的布局，我们还需要通过计算减去 40px 的 padding 大小才能和之前的大小保持一致：  
``` 
.box {  
  width: 60px; /* 通过计算，减去 40 像素 */
  padding: 20px;  
  border: 1px solid;
}
```
good case
如果我们使用了宽度分离，事情就会轻松很多：
``` 
/* 在first-div外嵌套一层 */
.first-div-father {
  width: 102px;
}

.first-div {
  border: 1px solid;
}
```
嵌套一层标签，父元素定宽，子元素因为 width 使用的是默认值 auto，所以会如水流般自动填满父级容器。因此，子元素的 content box 宽度就是 100px，和上面直接设置 width 为 100px 的表现一样。  
**改变 width/height 作用细节的 box-sizing**  
box-sizing 属性改变了 width 作用的盒子。“内在盒子” 的 4 个盒子分别是 content box、padding box、border box 和 margin box。默认情况下，width 是作用在 content box 上的，box-sizing 的作用就是可以把 width 作用的盒子变成其他几个，因此，理论上，box-sizing 可以有下面这些写法：  
.box1 {box-sizing: content-box;} → 默认值  
.box2 {box-sizing: padding-box;} → Firefox 曾经支持  
.box3 {box-sizing: border-box;} → 全部支持  
.box4 {box-sizing: margin-box;} →从未支持  

为何 box-sizing 不支持 margin-box  
如果我们使用 width 或 height 设定好了尺寸，此时设置 margin 值，其 offset 尺寸会不会有变化。显然不会，一个本身并不会改变元素尺寸的盒子，没有让 box-sizing 支持的道理。  

box-sizing 被发明出来最大的初衷应该是解决替换元素宽度自适应问题，比如 textarea 和 input  

## height
**height:auto**  
height: auto 要比 width: auto 简单而单纯得多，原因在于，CSS 的默认流是水平方向的，宽度是稀缺的，高度是无限的。height: auto 的表现也基本上就是：有几个元素盒子，每个多高，然后一加，就是最终的高度值了。  
**height: 100%**  
对于 height 属性，如果父元素 height 为 auto，只要子元素在文档流中，其百分比值完全就被忽略了。只要经过一定的实践，我们都会发现对于普通文档流中的元素，百分比高度值要想起作用，其父级必须有一个可以生效的高度值。  
**为何父级没有具体高度值的时候，height: 100% 会无效呢**  
一个错误的说法：死循环  
例如，一个 <div> 有一个高度为 100px 的子元素，此时，这个 <div> 被子元素撑起来后高度就是 100px，假设此时插入第二个子元素，高度设为 height: 100%，那么第二个子元素的高度就是 100px。但是， <div> 的 height 已经变成了 200px，而第二个子元素的高又会变成 200px。如此反复形成了逻辑上的死循环，然而这种说法是错误的  

正确的解释：浏览器的顺序渲染  
首先浏览器渲染的基本原理：先渲染父元素，后渲染子元素，是有先后顺序的。  
如果包含块的高度没有显式指定（即高度由内容决定），并且该元素不是绝对定位，则计算值为 auto，所以高度计算出来是 'auto' * 100 / 100 = NaN。  
那如何让元素支持 height: 100% 的效果呢？  
1. 设定显式的高度值，比如设置 height: 600px，或者可以生效的百分比值高度；
2. 使用绝对定位。

使用绝对定位时，需要注意绝对定位的宽高百分比计算是相对于 padding box 的，也就是说会把 padding 大小值计算在内，但是，非绝对定位元素则是相对于 content box 计算的

原文: 
[《CSS 世界》读书笔记-流与宽高](https://mp.weixin.qq.com/s/0YGg7KHm3UcjkzURWyQJng)
