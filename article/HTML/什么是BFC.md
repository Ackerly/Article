# 什么是BFC
BFC 全称：Block Formatting Context， 名为 "块级格式化上下文"。BFC它决定了元素如何对其内容进行定位，以及与其它元素的关系和相互作用，当涉及到可视化布局时，Block Formatting Context提供了一个环境，HTML在这个环境中按照一定的规则进行布局。  
BFC是一个完全独立的空间（布局环境），让空间里的子元素不会影响到外面的布局。那么怎么使用BFC呢，BFC可以看做是一个CSS元素属性  
**怎样触发BFC**  
简单列举几个触发BFC使用的CSS属性：  
- overflow: hidden
- display: inline-block
- position: absolute
- position: fixed
- display: table-cell
- display: flex

**BFC的规则**  
- BFC就是一个块级元素，块级元素会在垂直方向一个接一个的排列
- BFC就是页面中的一个隔离的独立容器，容器里的标签不会影响到外部标签
- 垂直方向的距离由margin决定， 属于同一个BFC的两个相邻的标签外边距会发生重叠
- 计算BFC的高度时，浮动元素也参与计算

**BFC解决了什么问题**  
1. 使用Float脱离文档流，高度塌陷  
    设置完float结果脱离文档流，使container高度没有被撑开，从而背景颜色没有颜色出来，解决此问题可以给container触发BFC
2. Margin边距重叠
   两个盒子的margin外边距设置的是10px，可结果显示两个盒子之间只有10px的距离，这就导致了margin塌陷问题，这时margin边距的结果为最大值，而不是合，为了解决此问题可以使用BFC规则（为元素包裹一个盒子形成一个完全独立的空间，做到里面元素不受外面布局影响），或者简单粗暴方法一个设置margin，一个设置padding。
3. 两栏布局
   第二个div元素为300px宽度，但是被第一个div元素设置Float脱离文档流给覆盖上去了，解决此方法我们可以把第二个div元素设置为一个BFC

原文:  
[面试官：请说说什么是BFC？大白话讲清楚](https://juejin.cn/post/6950082193632788493)