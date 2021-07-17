# less进阶学习(1)
## 使用变量
### 基本使用
``` 
// bad
a,
.link {
  color: #428bca;
}
.widget {
  color: #fff;
  background: #428bca;
}
```
``` 
// good
@link-color:        #428bca; // sea blue
@link-color-hover:  darken(@link-color, 10%);

// Usage
a,
.link {
  color: @link-color;
}
a:hover {
  color: @link-color-hover;
}
.widget {
  color: #fff;
  background: @link-color;
}
```
### 变量插值  
1.Selectors
``` 
// Variables
@my-selector: banner;

// Usage
.@{my-selector} {
  font-weight: bold;
  line-height: 40px;
  margin: 0 auto;
}
```
Compiles to:
``` 
.banner {
  font-weight: bold;
  line-height: 40px;
  margin: 0 auto;
}
```
2.URLs  
``` 
// Variables
@images: "../img";

// Usage
body {
  color: #444;
  background: url("@{images}/white-sand.png");
}
```
3.Import Statements
``` 
// Variables
@themes: "../../src/themes";

// Usage
@import "@{themes}/tidal-wave.less";
```
4.Properties
``` 
// Variables
@themes: "../../src/themes";

// Usage
@import "@{themes}/tidal-wave.less";
```
Complies to:
``` 
.widget {
  color: #0ee;
  background-color: #999;
}
```
### 引用变量
``` 
@primary:  green;
@secondary: blue;

.section {
  @color: primary;

  .element {
    color: @@color;
  }
}
```
Complies to
``` 
.section .element {
  color: green;
}
```
变量会从点前范围向上搜索，当一个一个变量声明两次，取决去当前块的最后一次声明
```
@var: 0;
.class {
  @var: 1;
  .brass {
    @var: 2;
    three: @var;
    @var: 3;
  }
  one: @var;
}
```
Compiles to:
``` 
.class {
  one: 1;
}
.class .brass {
  three: 3;
}
```
另外一个例子：
``` 
.header {
  --color: white;
  color: var(--color);  // the color is black
  --color: black;
}
```
### 属性作为变量（NEW）
```
.widget {
  color: #efefef;
  background-color: $color;
}
```
Compiles to:
``` 
.widget {
  color: #efefef;
  background-color: #efefef;
}
```
``` 
.block {
  color: red; 
  .inner {
    background-color: $color; 
  }
  color: blue;  
}
```
Compiles to:
``` 
.block {
  color: red; 
  color: blue;  
} 
.block .inner {
  background-color: blue; 
}
```
### 默认变量
``` 
// library
@base-color: green;
@dark-color: darken(@base-color, 10%);

// use of library
@import "library.less";
@base-color: red;
```
equal to
```
@base-color: green;
@dark-color: darken(red, 10%);
```
## 父选择器
### 使用&操作符代表父级选择器
``` 
a {
  color: blue;
  &:hover {
    color: green;
  }
}
```
Compiles to:
``` 
a {
  color: blue;
}

a:hover {
  color: green;
}
```
### 使用&产生复杂的类名
``` 
.button {
  &-ok {
    background-image: url("ok.png");
  }
  &-cancel {
    background-image: url("cancel.png");
  }

  &-custom {
    background-image: url("custom.png");
  }
}
```
Compiles to:
``` 
.button-ok {
  background-image: url("ok.png");
}
.button-cancel {
  background-image: url("cancel.png");
}
.button-custom {
  background-image: url("custom.png");
}
```
### 使用多个&
``` 
.link {
  & + & {
    color: red;
  }

  & & {
    color: green;
  }

  && {
    color: blue;
  }

  &, &ish {
    color: cyan;
  }
}
```
Compiles to:
``` 
.link + .link {
  color: red;
}
.link .link {
  color: green;
}
.link.link {
  color: blue;
}
.link, .linkish {
  color: cyan;
}
```
注意：&代表的是所有父类选择器而不是最近父类选择器
``` 
.grand {
  .parent {
    & > & {
      color: red;
    }

    & & {
      color: green;
    }

    && {
      color: blue;
    }

    &, &ish {
      color: cyan;
    }
  }
}
```
Compiles to:
``` 
.grand .parent > .grand .parent {
  color: red;
}
.grand .parent .grand .parent {
  color: green;
}
.grand .parent.grand .parent {
  color: blue;
}
.grand .parent,
.grand .parentish {
  color: cyan;
}
```
### 后置&符号
``` 
.header {
  .menu {
    border-radius: 5px;
    .no-borderradius & {
      background-image: url('images/button-background.png');
    }
  }
}
```
Compiles to:
``` 
.header .menu {
  border-radius: 5px;
}
.no-borderradius .header .menu {
  background-image: url('images/button-background.png');
}
```
### 组合&符号
``` 
p, a, ul, li {
  border-top: 2px dotted #366;
  & + & {
    border-top: 0;
  }
}
```
Compiles to:
``` 
p,
a,
ul,
li {
  border-top: 2px dotted #366;
}
p + p,
p + a,
p + ul,
p + li,
a + p,
a + a,
a + ul,
a + li,
ul + p,
ul + a,
ul + ul,
ul + li,
li + p,
li + a,
li + ul,
li + li {
  border-top: 0;
}
```
## Extend
``` 
nav ul {
  &:extend(.inline);
  background: blue;
}
.inline {
  color: red;
}
```
Compiles to:
``` 
nav ul {
  background: blue;
}
.inline,
nav ul {
  color: red;
}
```
### 拓展到选择器
extend附加到一个选择器看起来像一个普通的伪类选择器作为参数。选择器可以包含多个extend条款,但所有的extend必须在选择器后面。
- extend必须位于选择器后面，如：pre:hover:extend(div pre)
- 允许选择器与extend之间存在空格
- 允许多个extend，pre:hover:extend(div pre):extend(.bucket tr) 相当于pre:hover:extend(div pre, .bucket tr)
- pre:hover:extend(div pre).nth-child(odd)，不允许这样的写法，因为extend必须在最后面
### 精确匹配拓展
extend可能导致伪类的顺序不同
``` 
link:hover:visited {
  color: blue;
}
.selector:extend(link:visited:hover) {}
```
Compiles to:
``` 
link:hover:visited {
  color: blue;
}
```
### extend不能匹配nth表达式
``` 
:nth-child(1n+3) {
  color: blue;
}
.child:extend(:nth-child(n+3)) {}
```
Compiles to:
``` 
:nth-child(1n+3) {
  color: blue;
}
```
引号类型对于extend不是很重要
``` 
[title=identifier] {
  color: blue;
}
[title='identifier'] {
  color: blue;
}
[title="identifier"] {
  color: blue;
}

.noQuote:extend([title=identifier]) {}
.singleQuote:extend([title='identifier']) {}
.doubleQuote:extend([title="identifier"]) {}
```
Compiles to:
``` 
[title=identifier],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
}

[title='identifier'],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
}

[title="identifier"],
.noQuote,
.singleQuote,
.doubleQuote {
  color: blue;
```
### extend “all”
匹配选择器所有内容
``` 
.test {
  &:hover {
    color: green;
  }
}

.replacement:extend(.test all) {}
```
Compiles to:
``` 
.test:hover,
.replacement:hover {
  color: green;
}
```
### 选择器插入extend
extend无法匹配包括变量的选择器，如果选择器包括变量，extend将被忽略
``` 
.bucket {
  color: blue;
}
.some-class:extend(@{variable}) {} // interpolated selector matches nothing
@variable: .bucket;
```
Compiles to:
``` 
.bucket {
  color: blue;
}
```
### 作用域/extend 嵌套@media
:extend嵌套在@media只会匹配在同层级的@media
``` 
@media print {
  .screenClass:extend(.selector) {} // extend inside media
  .selector { // this will be matched - it is in the same media
    color: black;
  }
}
.selector { // ruleset on top of style sheet - extend ignores it
  color: red;
}
```
Compiles to:
``` 
@media print {
  .selector,
  .screenClass { /*  ruleset inside the same media was extended */
    color: black;
  }
}
.selector { /* ruleset on top of style sheet was ignored */
  color: red;
}
```
另外的例子：
``` 
@media screen {
  .screenClass:extend(.selector) {} // extend inside media
  @media (min-width: 1023px) {
    .selector {  // ruleset inside nested media - extend ignores it
      color: blue;
    }
  }
}
```
Compiles to:
``` 
@media screen and (min-width: 1023px) {
  .selector { /* ruleset inside another nested media was ignored */
    color: blue;
  }
}
```
### 重复检测
``` 
.alert-info,
.widget {
  /* declarations */
}

.alert:extend(.alert-info, .widget) {}
```
Compiles to:
``` 
.alert-info,
.widget,
.alert,
.alert {
  /* declarations */
}
```
### extend使用cases
``` 
<a class="animal bear">Bear</a>
.animal {
  background-color: black;
  color: white;
}
.bear {
  background-color: brown;
}
```
使用extend简化
``` 
<a class="bear">Bear</a>
.animal {
  background-color: black;
  color: white;
}
.bear {
  &:extend(.animal);
  background-color: brown;
}
```
### 使用迷信导致重复，使用extend简化
使用mixin
``` 
.my-inline-block() {
  display: inline-block;
  font-size: 0;
}
.thing1 {
  .my-inline-block;
}
.thing2 {
  .my-inline-block;
}
```
Compiles to:
``` 
.thing1 {
  display: inline-block;
  font-size: 0;
}
.thing2 {
  display: inline-block;
  font-size: 0;
}
```
使用extend
``` 
.my-inline-block {
  display: inline-block;
  font-size: 0;
}
.thing1 {
  &:extend(.my-inline-block);
}
.thing2 {
  &:extend(.my-inline-block);
}
```
Compiles to:
``` 
.my-inline-block,
.thing1,
.thing2 {
  display: inline-block;
  font-size: 0;
}
```
### 结合样式/更高级的Mixin
另一mixin只能使用简单的选择器,如果你有两个不同的html,但需要应用相同的样式都可以使用扩展关系两个方面。
``` 
li.list > a {
  // list styles
}
button.list-style {
  &:extend(li.list > a); // use the same list styles
}
```
## Merge
Merge功能允许聚合来自多个属性值为一个属性下逗号或空格分隔的列表。merge等属性是有用的背景和变换。
**添加逗号**
``` 
.mixin() {
  box-shadow+: inset 0 0 10px #555;
}
.myclass {
  .mixin();
  box-shadow+: 0 0 20px black;
}
```
Compiles to:
``` 
.myclass {
  box-shadow: inset 0 0 10px #555, 0 0 20px black;
}
```
**添加空格**
``` 
.mixin() {
  transform+_: scale(2);
}
.myclass {
  .mixin();
  transform+_: rotate(15deg);
}
```
Compiles to:
``` 
.myclass {
  transform: scale(2) rotate(15deg);
}
```

