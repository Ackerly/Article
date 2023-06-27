# CSS中@scope简介
## 基于邻近性的样式
不仅仅依靠源代码的顺序和特异性来确定样式，还可以根据元素的邻近性来覆盖样式。请猜测以下示例中按钮的颜色会是什么：  
``` 
@scope (.blue) {
     button {
         background-color: blue;
     }
 }

 @scope (.green) {
     button {
       background-color: green;
     }
 }

 @scope (.red) {
     button {
       background-color: red;
     }
 }
```
示例 1：  
``` 
<div class="red">
     <div class="green">
         <div class="blue">
             <button>Click</button>
         </div>
     </div>
 </div>
```
示例 2：  
``` 
<div class="blue">
     <div class="green">
         <div class="red">
             <button>Click</button>
         </div>
     </div>
 </div>
```
为了确保足够的颜色对比度以便轻松阅读，链接文本在浅色背景上是深蓝色，在深色背景上是浅蓝色。这样我们就不需要在每个链接上都加上一个类，这样做既繁琐又容易出现不一致性。  
``` 
.theme-white {
   background-color: white;
   color: black;
   }

 .theme-white a {
   color: #00528a;
 }

 .theme-black {
   background-color: black;
   color: white;
 }

 .theme-black a {
   color: #35adce;
 }
```
这种方法运作得还不错，但存在一个问题：嵌套。CSS 并不根据最近的 HTML 祖先元素来确定应用哪种样式，它只按照 CSS 文件中的源代码顺序进行处理。根据您定义样式的顺序，如果您将一个白色区块嵌套在一个黑色区块中，或者将一个黑色区块嵌套在一个白色区块中，链接的颜色将不再正确。在 @scope 出现之前，实际上没有解决这个问题的方法。  
``` 
<div class="theme-white">
   <a href="example.com">This link is the correct color</a>
 </div>

 <div class="theme-black">
   <a href="example.com">This link is the correct color</a>

   <div class="theme-white">
     <a href="example.com">This link is the wrong color</a>
   </div>

 </div>
```
使用 @scope，可以解决这个问题：  
``` 
.theme-white {
    background-color: white;
    color: black;
  }

 .theme-gray {
    background-color: #f5f5f5;
    color: black;
  }

 @scope (.theme-white, .theme-gray) {
   a {
     color: #00528a;
   }
 }

 .theme-black {
   background-color: black;
   color: white;
 }

 @scope (.theme-black) {
   a {
     color: #35adce;
   }
 }
```
无论链接是在白色或灰色背景上，它都会显示为深蓝色。而在黑色背景上，它会显示为更亮的浅蓝色。  
可以选择重写之前的 CSS，使用:scope 伪类来引用当前作用域的根元素。在下面的示例中，:scope 将选择具有.theme-black 类的任何元素。  
``` 
@scope (.theme-black) {
    :scope {
      background-color: black;
      color: white;
    }
    a {
      color: #35adce;
    }
 }
```
:scope 并不是一个新的伪类。它在浏览器中已经存在多年，但在样式表中使用时通常没有实际意义，因为在 @scope 块之外，它始终与:root 相同（:root 选择文档的根元素，即 <html> 元素）。  
## 为选择器设置较低的边界
看一下语法：  
``` 
@scope (.component) to (.content) {
   p {
     color: red;
   }
 }
```
第二个选择器设置了一个较低的边界，即从这个点停止样式化  
``` 
<div class="component">

   <p>In scope.</p>

   <div class="content">
     <p>Out of scope.</p>
   </div>

 </div>
```
如果在.content 内部有一个段落（<p>），它将不会被选择  
一个 @scope 可以有任意多个 “空洞”（holes）：  
``` 
 @scope (.component) to (.content, .slot, .child-component) {
   p {
     color: red;
   }
 }
```


原文:  
[CSS中@scope的简介](https://mp.weixin.qq.com/s/b5RJ4ZArlbzOmM3Zz1VDhg)
