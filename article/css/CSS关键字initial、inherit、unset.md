# CSS关键字initial、inherit和unset
CSS属性通常有initial（默认）、inherit（继承）和unset几个值
## initial
initial关键字用于设置css属性为它的默认值，可用于任何css样式（IE不支持该关键字）
## inherit
每一个CSS都有一个特性，属性是默认继承或者默认不继承。  
**可继承属性**
- 所有元素可继承：visibility 和 cursor
- 内联元素可继承：letter-spacing、word-spacing、white-space、line-height、color、font、 font-family、font-size、font-style、font-variant、font-weight、text- decoration、text-transform、direction
- 块状元素可继承：text-indent和text-align
- list-style、list-style-type、list-style-position、list-style-image
- border-collapse
## unset
unset是不设置，是关键字initial和inherit的组合。  
一个css属性设置了unset的话：
1. 该属性是默认继承属性，该值等同于inherit
2. 该属性是非继承属性，该值等同与initial

unset的一些妙用：  


原文: 
[谈谈 CSS 关键字 initial、inherit 和 unset](https://www.cnblogs.com/coco1s/p/6733022.html)
