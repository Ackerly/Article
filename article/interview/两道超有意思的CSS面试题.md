# 两道超有意思的CSS面试题
``` 
<div>
    <p id="a">First Paragraph</p>
</div>

// css
p#a {
    color: green;
}
div::first-line {
    color: blue;
}
```
标签 <p> 内的文字的颜色，是 green 还是 blue 呢？  
有趣的是，这里的最终结果是蓝色，也就是 color: blue 生效了。  
正常而言，ID 选择器的优先级不应该比伪类选择器高么？为什么这里反而是伪类选择器的优先级更高呢？  
并且，打开调试模式，我们定位到 <p> 元素上，只看到了 color: green 生效，没找到 div::first-line 的样式定义  
只有再向上一层，找到 <div> 的样式规则，才能看到这样一条规则  
<p> 标签继承了父元素 <div> 的这条规则，并且作用到了自身第一行元素之上，覆盖了原本的 ID 选择器内定义的 color: green  

**验证**  
下面的 DEMO 展示了 ::first-line 样式和各种选择器共同作用时的优先级对比，甚至包括了 !important 规则：  
- 第 1 段通过标签选择器设置为灰色
- 第 2 段通过类选择器设置为灰色
- 第 3 段通过 ID 选择器设置为灰色
- 第 4 段通过 !important bash 设置为灰色

综上的同时，每一段同时都使用了 ::first-line 选择器。  
``` 
<h2>::first-line vs. tag selector</h2>
<p>This paragraph ...</p>  

<h2>::first-line vs class selector</h2>
<p class="p2">This paragraph color i...</p>  

<h2>::first-line vs ID selector</h2>
<p id="p3">This paragraph color is set ...</p>  

<h2>::first-line vs !important</h2>
<p id="p4">This paragraph color is ....</p>  

// css
p {
  color: #444;
}
p::first-line {
  color: deepskyblue;
}

.p2 {
  color: #444;
}
.p2::first-line {
  color: tomato;
}

#p3 {
  color: #444;
}
#p3::first-line {
  color: firebrick;
}

#p4 {
  color: #444 !important;
}
#p4::first-line {
  color: hotpink;
}
```
无论是什么选择器，优先级都没有 ::first-line 高.究其原因，在于，::first-line 其实是个伪元素而不是一个伪类，被其选中的内容其实会被当成元素的子元素进行处理，类似于 ::before，::after 一样，因此，对于父元素的 color 规则，对于它而言只是一种级联关系，通过 ::first-line 本身定义的规则，优先级会更高！  

## 再来一题，MDN 的错误例子？
在 MDN 介绍 :not 的页面，有这样一个例子：  
``` 
:not(p) {
  color: blue;
}
```
:not(p) 可以选择任何不是 <p> 标签的元素。然而，上面的 CSS 选择器，在如下的 HTML 结构，实测的结果不太对劲。  
``` 
<p>p</p>
<div>div</div>
<span>span</span>
<h1>h1</h1>
```
<p> 元素的颜色仍旧是 color: blue?这是由于 :not(p) 同样能够选中 <body>，那么 <body> 的 color 即变成了 blue，由于 color 是一个可继承属性，<p> 标签继承了 <body> 的 color 属性，导致看到的 <p> 也是蓝色。  

把它改成一个不可继承的属性  
``` 
:not(p) {
  border: 1px solid;
}
```
<p> 没有边框体现，没有问题！

原文:  
[两道超有意思的 CSS 面试题，试试你的基础](https://mp.weixin.qq.com/s/tVyVT5IZxfzCRnIdZLzq8w)
