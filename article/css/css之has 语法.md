# css之has 语法
## 语法
:has 伪类用于根据元素内容选择元素。它应用于我们想应用规则的元素上，并将其传递给应该包含的元素的选择器  
``` 
/* 这里我们选择任何包含 `h1` 的具有 `post` 类的元素 */
.post:has(h1) {
  background-color: teal;
}
```
## 使用 :has 作为父选择器
将 :has 作为父选择器可以简化许多情况。以下是一些可能的示例：  
在应用的某些页面上，你可能想要更改 body 元素的全局字体大小或背景颜色。在引入 :has 伪类之前，我们通常需要通过后端根据页面类型切换某些 HTML 类。然而，通过父选择器，现在可以轻松实现：  
``` 
body:has(.container.legal-mentions) {
  font-size: 80%;
}
```
在博客文章列表中，如果文章包含图片，我们希望这些文章的边距发生变化：  
``` 
.post:has(img) {
  margin-left: 0;
}
```
## 进一步使用组合器
在 has 中使用 子代组合器 >，以确保我们选择的是直接子元素。例如，要选择具有 hr 元素作为直接子元素的 div 元素，可以使用选择器 div:has(>hr)。  
可以使用 相邻兄弟组合器 + 来选择紧跟在另一个元素后面的元素。例如，要选择一个标题后面跟着一个副标题，可以使用 title:has(+.subtitle)。  
## 与其他伪类组合
可以把 has 与 hover 结合使用来实现这一点。例如，如果我们希望在容器中的任何链接 hover 时都有边框，可以使用以下代码：  
``` 
.container:has(a:hover) {
  border: 2px solid pink;
}
```
## 浏览器支持
:has 伪类仅在 Firefox 中缺失。然而，它在一个标志后面，所以很快应该会被支持！

原文:  
[:has 语法，终于可以用了](https://mp.weixin.qq.com/s/5QhxGVd1C1nmELxRC9Fglg)
