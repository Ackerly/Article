# 用BEM和现代CSS选择器控制级联
BEM提供了几个好处，最大的好处是它有助于避免 CSS 级联中的特异性冲突。这是因为，如果使用得当，任何以 BEM 格式编写的选择器都应该有相同的特异性得分（0,1,0）。  
> 实际上有一些例外情况，在这些情况下增加特异性被认为是完全可以接受的。例如：:hover 和 :focus 伪类。它们的特异性得分为 0,2,0。另一种是伪元素，如 ::before 和 ::after，它们的特异性得分为 0,1,1。不过，对于本文的其余部分，让我们假设我们不希望出现任何其他的特异性变动。  

## 什么是现代 CSS 选择器
CSS Selectors Level 4规范为我们提供了一些强大的新方法来选择元素。包括：:is()，:where() 和 :not()，所有的现代浏览器都支持它们，并且现在几乎可以安全地用于任何项目。  
:is() 和 :where() 基本上是一样的，除了它们如何影响特异性。具体来说，:where() 的特异性得分总是0,0,0。甚至 :where(button#widget.some-class) 没有特异性。同时，:is() 的特异性是其参数列表中特异性最高的元素。  
功能强大得令人难以置信的 :has() 伪类也正在迅速获得浏览器支持,然而，在撰写本文时，浏览器对 :has() 的支持还不足以用于生产。  
在 BEM 中加入一个伪类，然后…  
``` 
.something:not(.something--special) {
  /* 给所有 somethings 添加样式, 除了special somethings */
}
```
对于BEM，理想情况下希望我们的选择器都具有 0,1,0 的特异性得分。为什么 0,2,0 不好？考虑一个类似的例子，如下：  
``` 
.something:not(a) {
  color: red;
}
.something--special {
  color: blue;
}
```
尽管第二个选择器在源顺序中排在最后，但第一个选择器的更高特异性 (0,1,1) 胜出，.something--special 元素的颜色将被设置为红色。上述中，假设您的 BEM 编写正确，并且所选元素在 HTML 中应用了 .something 基类和 .something--special 修饰类。  

## :where() 允许我们嵌套
通过一些随意的编码，我们可能会得到这样的CSS  
``` 
.card { ... }

.card--featured {
  /* etc. */  
  .card__title { ... }
  .card__title { ... }
}

.card__title { ... }
.card__img { ... }
```
当它是“featured”卡片（使用 .card--featured 类）时，卡片的标题和图片需要采用不同的样式。但是，正如我们现在所知，上面的代码产生的特异性得分与我们系统的其他部分不一致。   
一个比较古板的写法可能会这样做：
``` 
.card { ... }
.card--featured { ... }
.card__title { ... }
.card__title--featured { ... }
.card__img { ... }
.card__img--featured { ... }
```
HTML也有一个缺点。经验丰富的 BEM 作者可能痛苦地意识到有条件地将修饰类应用于多个元素所需的笨重模板逻辑。在这个例子中，HTML模板需要有条件地将 --featured 修饰类添加到三个元素（.card、.card__title 和 .card__img）中，尽管在实际例子中可能更多。这将产生许多if语句。  
:where() 选择器可以帮助我们在不增加特异性得分的情况下编写更少的模板逻辑，以及更少的 BEM 类。  
``` 
.card { ... }
.card--featured { ... }

.card__title { ... }
:where(.card--featured) .card__title { ... }

.card__img { ... }
:where(.card--featured) .card__img { ... }
```
在 Sass 中情况差不多（注意后面的“&”）：  
``` 
.card { ... }
.card--featured { ... }
.card__title { 
  /* etc. */ 
  :where(.card--featured) & { ... }
}
.card__img { 
  /* etc. */ 
  :where(.card--featured) & { ... }
}
```
## 非BEM HTML
有时您需要处理超出您控制范围的 HTML。例如，一个第三方脚本，它注入了您需要设置样式的 HTML。标记通常不是用 BEM 类名编写的。在某些情况下，这些样式根本不使用类，而是使用 ID！  
我们能使用到 :where()。这个解决方案有点 hack，因为我们需要引用 DOM 树中更上层某个已知存在的元素的类。  
``` 
#widget {
  /* etc. */
}

/* ✅ 特异性得分: 0,1,0 */
.page-wrapper :where(#widget) {
  /* etc. */
}
```
引用父元素感觉有点风险和受限。如果父类发生了改变或者因为某种原因不存在了怎么办?更好的解决方案（但可能同样 hack）是使用 :is()。请记住，:is() 的特异性等于其选择器列表中特异性最高的选择器。  
可以引用一个虚构的类和 <body> 标签，而不是像上面的例子那样使用 :where() 并且引用我们知道（或希望！）存在的类。  
``` 
/* ✅ 特异性得分: 0,1,0 */
:is(.dummy-class, body) :where(#widget) {
  /* etc. */
}
```
一直存在的 body 将帮助我们选择 #widget 元素，在同一个 :is() 中存在的 .dummy-class 类会为 body 选择器提供与类相同的特异性分数（0,1,0）…并且 :where() 的使用确保了选择器不会获得更多特异性。


原文:  
[用BEM和现代CSS选择器控制级联](https://mp.weixin.qq.com/s/VN_iz2NCINo9IB6GAnUxBA)
