# CSS columns实现两端对齐布局
``` 
<div class="container">
    <div class="zhang"></div>
    <div class="xin"></div>
    <div class="xu"></div>
</div>

.container {
    columns: 3;
    column-gap: 30px;
}
.container > div {
    padding: 50px;
    background: deepskyblue;
}
```
## columns实现的优缺点
**优点**  
相比Flex布局和Grid布局的space-between值的两端对齐效果，使用CSS columns布局实现的优点除了代码少了一点之外，最大的优点是保护了HTML元素原本的display计算值。  
例如，浏览器默认状态下，<li>元素会出现项目符号，例如圆点，或数字序号。  
**缺点**  
适合单行元素的两端对齐效果，如果列表元素有很多行，则columns布局就不太好处理，一是列表的流向优先垂直方向，二是容易出现列表垂直分列的意外场景。

原文: 
[CSS columns轻松实现两端对齐布局效果](https://www.zhangxinxu.com/wordpress/2020/05/css-columns-justify-content/)
