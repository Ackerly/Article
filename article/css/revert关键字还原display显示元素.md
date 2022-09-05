# 使用CSS revert全局关键字还原display显示元素
## 需求与问题描述
需求：列表默认最多显示3项，点击更多按钮显示剩余列表项。
``` 
// CSS
li:nth-child(n+4) {
    display: none;
}
// HTML
<ul id="ul">
  <li>选项1</li>
  <li>选项2</li>
  <li>选项3</li>
  <li>选项4</li>
  <li>选项5</li>
</ul>
<p><button id="b1">更多</button></p>
```
点击更多按钮，让隐藏的<li>元素显示。不少前端开发会想到使用使用下面代码实现
``` 
li.style.display = 'block';
```
这样会导致列表的项目符号没有了。原因是<li>元素默认的display计算值不是'block'，而是'list-item'。  
可以使用revert关键字代替，不管使用标签，全部使用浏览器默认的display计算值。  
## revert关键字是干嘛的
revert关键字可以让当前元素的样式还原成浏览器内置的样式。  
revert关键字有时候会和CSS all属性一起使用，可以将某个控件元素完全还原为浏览器默认的样子。
例如<progress>进度条效果在iOS端很好看，很有质感，无需自定义样式，则我们就可以all:revert一键还原成系统默认的界面样式。
``` 
/* 仅iOS Safari有效 */
@supports (-webkit-overflow-scrolling: touch) {
    progress {
        all: revert;
    }
}
```
例如写了自定义样式的按钮可以使用all：revert还原浏览器原生样式的按钮
## 其他display场景
revert控制元素显示最适合用在公用的组件开发中，例如实现一个选项卡切换，或者手风琴切换效果，其中就有对元素的显示控制。可以告别传统的 display:block 显示，也无需判断原始的display计算值再设置，直接使用 display:revert。这样，元素显示的时候，究竟是内联、块状直接通过HTML标签区分即可。


原文: 
[使用CSS revert全局关键字还原display显示元素](https://www.zhangxinxu.com/wordpress/2021/05/css-revert-display/)
