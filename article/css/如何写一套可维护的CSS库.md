# 如何写一套可维护的CSS库
## OOCSS
面向对象的CSS,
- （分离结构和主题）减少对 HTML 结构的依赖
- （分离容器和内容）增加样式的复用性

在 OOCSS 的观念中，强调重复使用 class，而应该避免使用 id 作为 CSS 的选择器。 OOCSS追求元件的复用，其class命名更为抽象，一般不体现具体事物，而注重表现层的抽取。

## SMACSS
smacss通过一个灵活的思维过程来检查你的设计过程和方式是否符合你的架构  
设计的主要规范有三点：
- 为css分类（SMACSS认为css有5个类别，分别是： 1 Base 2 Layout 3 Module 4 State 5 Theme or Skin）
- 命名规范
- 最小化适配深度

### css分类
**Base**  
基础规范,描述的是任何场合下，页面元素的默认外观。它的定义不会用到class和ID。css reset也属于此类。常见的如normalize.css,CSS Tools  
**Layout**  
布局规范,元素是有层次级别之分的，Layout Rules属于较高的一层，它可以作为层级较低的Module Rules元素的容器。左右分栏、栅格系统等都属于布局规范。布局是一个网站的基本，无论是左右还是居中，甚至其他什么布局，要实现页面的基本浏览功能，布局必不可少。SMACSS还约定了一个前缀l-/layout-来标识布局的class。举个最普遍的例子。  
``` 
.layout-header {}
.layout-container {}
.layout-sidebar {}
.layout-content {}
.layout-footer {}
```
**Module**  
模块规范,模块是SMACSS最基本的思想，同时也是大部分CSS理论的基本，将样式模块化就能达到复用和可维护的目的，但是SMACSS提出了更具体的模块化方案。SMACSS中的模块具有自己的一个命名，隶属于模块下的类皆以该模块为前缀，例子如下：
``` 
.todolist{}
.todolist-title{}
.todolist-image{}
.todolist-article{}
```
可以看到todolist作为一个模块，包含了title，image，article等组件，同时还可以加上如.todolist-background-danger等修饰类，在模块内可以使用其名称做前缀任意组织模块结构，但目的是让其变得更易用，提高可扩展性和灵活度，如果只是为了修饰而修饰，写出大量没有任何复用性的类，便是一种弄巧成拙的做法。  
**State**  
描述的是任一元素在特定状态下的外观。例如，一个消息框可能有success和error等状态。与OOCSS抽取修饰类的方式的不同，SMACSS是抽取更高级别的样式类，得到更强的复用性，如隐藏某个元素的写法：  
``` 
.is-hidden{
    display: none;
}
```
**Theme**  
主题规范,描述了页面主题外观，一般是指颜色、背景图。Theme Rules可以修改前面4个类别的样式，且应和前面4个类别分离开来（便于切换，也就是“换肤”）.SMACSS的Theme不要求使用单独的class命名，也就是说，你可以在Module Rules中定义.header{ }然后在Theme Rules中也用.header { }来定义需要修改的部分(后加载覆盖前加载样式内容)
### 命名规范
按照前面5种的划分:
- Base(Pass)
- Layout用l-或layout-这样的前缀，例如：.l-header、.l-sidebar。
- Module用模块本身的命名，例如图文排列的.media、.media-image。
- State用is-前缀，例如：.is-active、.is-hidden。
- Theme如果作为单独class，用theme-前缀，例如.theme-a-background、.theme-a-shadow。

### 最小化适配深度
``` 
/* depth 1 */
.sidebar ul h3 { }

/* depth 2 */
.sub-title { }
```
两段css的区别在于html和css的耦合度(这一点上和OOCSS的分离容器和内容的原则不谋而合)。可以想到，由于上面的样式规则使用了继承选择符，因此对于html的结构实际是有一定依赖的。如果html发生重构，就有可能不再具有这些样式。对应的，下面的样式规则只有一个选择符，因此不依赖于特定html结构，只要为元素添加class，就可以获得对应样式。  
继承选择符是有用的，它可以减少因相同命名引发的样式冲突（常发生于多人协作开发）。在不造成样式冲突的允许范围之内，尽可能使用短的、不限定html结构的选择符。这就是SMACSS的最小化适配深度的意义。  
## BEMCSS
BEM 分别代表着：Block（块）、Element（元素/子块/组成部分）、Modifier（修饰符），是一种组件化的 CSS 命名方法和规范，由俄罗斯 Yandex团队所提出。其目的是将用户界面划分成独立的（模）块，使开发更为简单和快速，利于团队协作开发。  
特点：  
- 组件化/模块化的开发思路。书写方式解耦化，不会造成命名空间的污染，如：.xxx ul li 写法带来的潜在嵌套风险。
- 命名方式化扁平，避免样式层级过多而导致的解析效率降低，渲染开销变大。
- 组件结构独立化，减少样式冲突，可以将已开完成的组件快速应用到新项目中。
- 有着较好的维护性、易读性、灵活性。

**Block（块）**  
是一个独立的实体，即通常所说的模块或组件。例：header、menu、search  
块名需能清晰的表达出，其用途、功能或意义，具有唯一性。块名称之间用-连接。每个块名前应增加一个前缀，这前缀在 CSS 中有命名空间（如：m-、u-、分别代表：mod 模块、ui 元件）。每个块在逻辑上和功能上都相互独立。由于块是独立的，可以在应用开发中进行复用，从而降低代码重复并提高开发效率。块可以放置在页面上的任何位置，也可以互相嵌套。同类型的块，在显示上可能会有一定的差异，所以不要定义过多的外观显示样式，主要负责结构的呈现。这样就能确保块在不同地方复用和嵌套时，增加其扩展性。综上所述，最终我们可以把BEM规则最终定义成：  
构建一个 search 组件写法 .m-search{} 结构  
如果打算开发一套框架，可以使用具有代表性的缩写，用来表示命名空间：Element UI(el-)、Ant Design(ant-)、iView(ivu-)。  
**Element（元素）**  
是块中的组成部分，对应块中的子元素/子节点。  
例：header title、menu item、list item  元素名需能简单的描述出，其结构、布局或意义，并且在语义上与块相关联。块与元素之间用__连接。不能与块分开单独使用。块的内部元素，都被认为是块的子元素。一个块中元素的类名必须用父级块的名称作为前缀，search 组件中包含 input 和 button，是列表中的一个子元素。  
写法 .m-search{} .m-search__input{} .m-search__button{} 结构  
``` 
<!-- search 组件 -->
<form class="m-search">
    <!-- input 是 search 组件的子元素 -->
    <input class="m-search__input">
    <!-- button 是 search 组件的子元素 -->
    <button class="m-search__button">Search</button>
</form>
```
原则上书写时不会出现两层以上的嵌套，所有样式都为平级，嵌套只出现在 .m-block_active ，状态激活时的情况。  
**Modifier（修饰符）**  
定义块和元素的外观、状态或类型。例：color、disabled、size  修饰符需能直观易懂表达出，其外观、状态或行为。修饰符用_连接块与元素。修饰符不能单独使用。在必要时可进行扩展，书写成：block__elem_modifier_modifier，第一个modifier表示其命名空间。情景假定 search 组件有多种外观，选择其中一种。并且在用户未输入内容时，button 显示为禁用样式。
写法 .m-search{} .m-search_dark{} .m-search__input{} .m-search__button{} .m-search__button_disabled{} 结构
``` 
<!-- dark 表明 search 组件的外观 -->
<form class="m-search m-search-form_dark">
    <input class="m-search__input">
    <!-- disabled 表明 search__button 的状态 -->
    <button class="m-search__button m-search__button_disabled">Search</button>
</form>
```


参考:  
[如何写出一套可维护的CSS库？](https://juejin.cn/post/6958690548009926687?utm_source=gold_browser_extension)
