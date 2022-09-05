# CSS属性选择器
## 属性选择器
- 选择包含title属性的div
``` 
div[title]
```
- 选择包含title属性的子元素，只需要加个空格
``` 
div [title]
```
- 选择title内容是dna的元素
``` 
div[title="dna"]
```
- 选择title属性包含dna单词的元素
> 注意dna需要是单词，也就是用空格分割，比如“my beautiful dna”或“mutating dna is fun”
``` 
div[title~="dna"]
```
- 以dna结尾的元素
``` 
div[title$="dna"]
```
- 以dna开头
``` 
div[title^="dna"]
```
- 匹配dna或dna-zh，但不希望匹配dna-er
> 这种场景一般用在国际化，比如en en-us就可以用|=“en”
``` 
div[title|="dna"]
```
- 只要包含dna这三个字符就选中
``` 
div[title*="dna"]
```
- 使用i标识匹配时大小写不敏感
``` 
div[title*="dna" i]
```
- 如果找到一个a标签，拥有title属性并且className以genes结尾
``` 
a[title][class$="genes"]
```
## 获取标签的值
用attr标识符拿到当前选择器中元素的属性，比如当hover状态时，在文字尾部显示其title属性：
``` 
.joke:hover:after {
    content: "Answer:" attr(title);
    display: block;
}
```
## 其它用法
**根据输入框类型设置样式**
``` 
input[type="email"] {
  color: papayawhip;
}
input[type="tel"] {
  color: thistle;
}
```
**改变下载标签的icon**
``` 
a[download][href$="pdf"]:after {
  content: url(pdf-icon.svg);
}
```
**重写来代码**
``` 
<div bgcolor="#000000" color="#FFFFFF">Old, holey genes</div>
div[bgcolor="#000000"] {
  /*override*/
  background-color: #222222 !important;
}
```
不过过多的这种写法会导致难以维护  
**结合新标签功能**
比如details标签是html原生的手风琴折叠组件
``` 
<details> <summary>List of Genes</summary> Roddenberry Hackman </details>
```
使用属性选择器，定义打开时的样式
``` 
details[open] {
  background-color: hotpink;
}
```
**为没有async标记的script标签着色**
``` 
script[src]:not([async]) {
  display: block;
  width: 100%;
  height: 1em;
  background-color: red;
}
script:after {
  content: attr(src);
}
```
**为JS事件着色(比如触发的鼠标事件可以作为选择器)**
```
[OnMouseOver] {
  color: burlywood;
}
[OnMouseOver]:after {
  content: "JS: " attr(OnMouseOver);
}
```
**选中隐藏元素**
``` 
[hidden],
[type="hidden"] {
  display: block;
}
```
## 总结
虽然CSS选择器强大，但是回到CSS Module或者css-in-js的工程代码里，我们往往难以做太多的实践
1. DOM结构改变  
业务开发中，大量的需求导致过一段事件DOM结构就面目全非了，就算是普通的圣杯布局，老版本用Table布局，后面用div + flex重构，你会当心之前table选择器在某一天全部失效。最大的原因在于视觉界面对应的实现方式太多，不仅标签各异，CSS属性还有table、block、flex、grid可选，同时grid属性会导致视觉结构与DOM结构不完全对应。  
如果今天用css选择器做了一套完全贴合现在DOM结构的CSS文件，这个CSS文件也许是后面DOM结构改动的噩梦
2. 全局样式的覆盖  
仅对属性做全局覆盖，的确可以绕开DOM结构的限制，但是这样的全局样式覆盖并不是所有团队。  
比如一个团队对项目通用样式做了抽象，比如一个按钮通过添加animate就可以显现非常炫酷的按钮与动画效果，页面流畅，用户体验统一，前端代码简洁优雅。但是当团队水平参差不齐，有人使用table，有人使用试验阶段css属性，有人抽象一个全局样式文件，有人私下注入全局样式，最后总有人的样式被全局覆盖，最后不得不页面入口写上*：unset清空各种奇怪的样式干扰。
3.JS模块化思维的影响  
一个项目安装几百个npm依然可以正常运行，因为第三方包都遵循模块化，同事说不产生副作用，但是如果npm定义不同规范的CSS样式覆盖，项目不一定能正常运行。  
大部分UI组件库是自带样式的，但很多时候我盟用一个UI组件仅仅为了在某处地方使用，不想接受他带来的全局样式污染，好的组件库往往css使用很收敛，尽量不要对用户环境造成影响。  
css modules或者是css-in-js，让每个组件的className都唯一，做到标签粒度的隔离

在一个确定的环境中，比如一个组件，一个独立的模块，比较适合css选择器，但是要注意作用域，如果没有达成共识，最好不要放到全局样式中。  
CSS属性选择器的强大功能需要良好的项目管理做支撑，或者通过技术手段，比如shadow dom做支撑，不过shadow dom支持度仍然很低，所以使用编译工具做隔离，在某种程度上模拟了CSS选择器，承当CSS选择器+shadow dom的功能。  
一些样式都用className控制也许是shadow dom出来前的一种妥协方案。

原文: 
[精读<<使用CSS属性选择器>>](https://github.com/ascoders/weekly/blob/master/%E5%89%8D%E6%B2%BF%E6%8A%80%E6%9C%AF/81.%E7%B2%BE%E8%AF%BB%E3%80%8A%E4%BD%BF%E7%94%A8%20CSS%20%E5%B1%9E%E6%80%A7%E9%80%89%E6%8B%A9%E5%99%A8%E3%80%8B.md)
