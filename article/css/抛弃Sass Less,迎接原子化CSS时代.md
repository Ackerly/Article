# 抛弃 Sass / Less ，迎接原子化 CSS 时代
## 什么是原子CSS
你可能听说过各种 CSS 方法，如 BEM, OOCSS…
``` 
<button class="button button--state-danger">Danger button</button>
```
人们真的很喜欢 Tailwind CSS 和它的 实用工具优先（utility-first） 的概念。这与 Functional CSS 和 Tachyon 这个库的理念非常接近。
``` 
<button
  class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
>
  Button
</button>
```
原子 CSS 就像是实用工具优先（utility-first）CSS 的一个极端版本: 所有 CSS 类都有一个唯一的 CSS 规则。原子 CSS 最初是由 Thierry Koblentz (Yahoo!)在 2013 年挑战 CSS 最佳实践时使用的.
``` 
/* 原子 CSS */
.bw-2x {
  border-width: 2px;
}
.bss {
  border-style: solid;
}
.sans {
  font-style: sans-serif;
}
.p-1x {
  padding: 10px;
}
/* 不是原子 CSS 因为这个类包含了两个规则 */
.p-1x-sans {
  padding: 10px;
  font-style: sans-serif;
}
```
使用实用工具/原子 CSS，把结构层和表示层结合起来:当需要改变按钮颜色时，直接修改 HTML，而不是 CSS.  
这种紧密耦合在现代 CSS-in-JS 的 React 代码库中也得到了承认，但似乎 是 CSS 世界里最先对传统的关注点分离有一些异议.  
CSS 权重也不是什么问题，因为我们使用的是最简单的类选择器。   
通过 html 标签来添加样式，发现了一些有趣的事儿：
- 增加新功能的时候，样式表的增长减缓了。
- 可以到处移动 html 标签，并且能确保样式也同样生效
- 可以删除新特性，并且确保样式也同时被删掉了

可以肯定的缺点是，html 有点臃肿。对于服务器渲染的 web 应用程序来说可能是个缺点，但是类名中的高冗余使得 gzip 可以压缩得很好。同时它可以很好地处理之前重复的 css 规则。  
一旦你的实用工具/原子 CSS 准备好了，它将不会有太大的变化或增长。可以更有效地缓存它(你可以将它附加到 vendor.css 中，重新部署的时候它也不会失效)。它还具有相当好的可移植性，可以在任意其他应用程序中使用。   
## 实用工具/原子CSS的限制
人们通常手工编写实用工具/原子 CSS，精心制定命名约定。但是很难保证这个约定易于使用、保持一致性，而且不会随着时间的推移而变得臃肿。  
这个 CSS 可以团队协作开发并保持一致性吗?它受巴士因子的影响吗?  
> 巴士系数是软件开发中关于软件专案成员之间讯息与能力集中、未被共享的衡量指标，也有些人称作“货车因子”、“卡车因子”（lottery factor/truck factor）。一个专案或计划至少失去若干关键成员的参与（“被巴士撞了”，指代职业和生活方式变动、婚育、意外伤亡等任意导致缺席的缘由）即导致专案陷入混乱、瘫痪而无法存续时，这些成员的数量即为巴士系数。

需要预先开发好一个不错的实用工具/原子样式表，然后才能开始开发新功能.
如果实用工具/原子 CSS 是由别人制作的，你将不得不首先学习类命名约定(即使你知道 CSS 的一切)。这种约定是有主观性的，很可能你不喜欢它。  
有时，你需要一些额外的 CSS，而实用工具/原子 CSS 并不提供这些 CSS。没有约定好的方法来提供这些一次性样式。  
## Tailwind
Tailwind 使用的方法是非常便捷的，并且解决了上述一些问题。  
它通过 Utility-First 的理念来解决 CSS 的一些缺点，通过抽象出一组类名 -> 原子功能的集合，来避免你为每个 div 都写一个专有的 class，然后整个网站重复写很多重复的样式。  
传统卡片样式写法：
``` 
<div class="chat-notification">
    <div class="chat-notification-logo-wrapper">
        <img class="chat-notification-logo" src="/img/logo.svg" alt="ChitChat Logo"></img>
    </div>
    <div class="chat-notification-content">
        <h4 class="chat-notification-title">ChitChat</h4>
        <p class="chat-notification-message">You have a new message!</p>
    </div>
</div>
<style>
    .chat-notification {
        display: flex;
        max-width: 24rem;
        margin: 0 auto;
        padding: 1.5rem;
        border-radius: 0.5rem;
        background-color: #fff;
        box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
    }
    .chat-notification-logo-wrapper {
        flex-shrink: 0;
    }
    .chat-notification-logo {
        height: 3rem;
        width: 3rem;
    }
    .chat-notification-content {
        margin-left: 1.5rem;
        padding-top: 0.25rem;
    }
    .chat-notification-title {
        color: #1a202c;
        font-size: 1.25rem;
        line-height: 1.25;
    }
    .chat-notification-message {
        color: #718096;
        font-size: 1rem;
        line-height: 1.5;
    }
</style>
```
Tailwind 卡片样式写法：
``` 
<div class="p-6 max-w-sm mx-auto bg-white rounded-xl shadow-md flex items-center space-x-4">
    <div class="flex-shrink-0">
        <img class="h-12 w-12" src="/img/logo.svg" alt="ChitChat Logo"></img>
    </div>
    <div>
        <h4 class="text-xl font-medium text-black">ChitChat</h4>
        <p class="text-gray-500">You have a new message!</p>
    </div>
</div>
```
它并不是真的为所有网站提供一些唯一的实用工具 CSS，取而代之的是，它提供了一些公用的命名约定。通过一个配置文件，你可以为你的网站生成一套专属的实用工具 CSS。  
Tailwind 提供了一套强大的构建系统，比如默认情况下它提供了一些响应式的断点值：
``` 
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'sm': '640px',
      // => @media (min-width: 640px) { ... }

      'md': '768px',
      // => @media (min-width: 768px) { ... }

      'lg': '1024px',
      // => @media (min-width: 1024px) { ... }

      'xl': '1280px',
      // => @media (min-width: 1280px) { ... }
    }
  }
}
```
你可以随时在配置文件中更改这些断点，比如你所需要的小屏幕 sm 可能指的是更小的 320px，那么你想要在小屏幕时候采用 flex 布局，还是照常写 sm:flex，遵循同样的约定，只是这个 sm 已经被你修改成适合于项目需求的值了。  
再比如说，Tailwind 里的 spacing 掌管了 margin、padding、width 等各个属性里的代表空间占用的值，默认是采用了 rem 单位，当你在配置里这样覆写后：  
``` 
// tailwind.config.js
module.exports = {
  theme: {
    spacing: {
      '1': '8px',
      '2': '12px',
      '3': '16px',
      '4': '24px',
      '5': '32px',
      '6': '48px',
    }
  }
}
```
你再去写 h-6（height）, m-2（margin）, mb-4（margin-bottom），后面数字的意义就被你改变了。  
也许从桌面端换到移动端项目，这个 6 代表的含义变成了 6rem，但是这套约定却深深的印在你的脑海里，成为你知识的一部分了。  
Tailwind 的知识可以迁移到其他应用程序，即使它们使用的类名并不完全相同。这让我想起了 React 的「一次学习，到处编写」理念。    
Tailwind 提供的类名能覆盖他们 90% - 95% 的需求。这个覆盖面似乎已经足够广了，并不需要经常写一次性的 CSS 了。  
**为什么要使用原子 CSS 而不是 Tailwind CSS?强制执行原子 CSS 规则的一个规则，一个类名，有什么好处**
Tailwind 是一个优秀的解决方案，但仍然有一些问题没有解决:
- 需要学习一套主观的命名约定
- CSS 规则插入顺序仍然很重要
- 未使用的规则可以轻松删除吗
- 我们如何处理剩下的一次性样式?

与 Tailwind 相比，手写原子 CSS 可能不是最方便的。
## 和CSS-in-JS比较
CSS-in-JS 和实用工具/原子 CSS 有密切关系。这两种方法都提倡使用标签进行样式化。以某种方式试图模仿内联样式，这让它们有了很多相似的特性(比如在移动某些功能的时候更有信心)。  
Christopher Chedeau 一直致力于推广 React 生态系统中 CSS-in-JS 理念。在很多次演讲中，他都解释了 CSS 的问题:
1. 全局命名空间
2. 依赖
3，无用代码消除
4. 代码压缩
5. 共享常量
6. 非确定性（Non-Deterministic）解析
7. 隔离

实用工具/原子 CSS 也解决了其中的一些问题，但也确实没法解决所有问题（特别是样式的非确定性解析)。
## 探索原子 CSS-in-JS
原子 CSS-in-JS 可以被视为是“自动化的原子 CSS”：
- 你不再需要创建一个 class 类名约定
- 通用样式和一次性样式的处理方式是一样的
- 能够提取页面所需要的的关键 CSS，并进行代码拆分
- 有机会修复 JS 中 CSS 规则插入顺序的问题

两个大规模的原子 CSS-in-JS 的部署使用
- React-Native-Web at Twitter
- Stylex at Facebook

也可以看这些库:
- Styletron
- Fela
- Style-Sheet
- cxs
- otion
- css-zero
- ui-box
- style9
- stitches
- catom

## React-Native-Web
 React-Native-Web 是一个常规的 CSS-in-JS 库，它自带一些原始的 React 组件。所有你写 View 组件的地方，都可以用 div 替换。  
**Stylex**
Stylex 是一个新的 CSS-in-JS 库
## 可扩展性
在 Atomic CSS 的加成下，Twitter 和 Facebook 的 CSS体积都大幅减少了，现在它的增长遵循的是对数曲线。不过，简单的应用则会多了一些 初始体积。  
Facebook 分享了具体数字:  
- 旧的网站仅仅首页就用了 413Kb 的 CSS
- 新的网站整个站点只用了 74Kb，包括暗黑模式
## 源码和输出
React-Native-Web 会扩展 CSS 语法糖，比如 margin: 0，会被输出为 4 个方向的 margin 原子规则。以一个组件为例，来看看旧版传统 CSS 和新版原子 CSS 输出的区别。
``` 
<Component1 classNames="class1" /> <Component2 classNames="class2" />
.class1 {
  background-color: mediumseagreen;
  cursor: default;
  margin-left: 0px;
}
.class2 {
  background-color: thistle;
  cursor: default;
  jusify-content: flex-start;
  margin-left: 0px;
}
```
两个样式中 cursor 和 margin-left 是一模一样的，但它在输出中都会占体积。  
再来看看原子 CSS 的输出：  
``` 
<Component1 classNames="classA classC classD" />
<Component2 classNames="classA classB classD classE" />

class a {
  cursor: default;
}
class b {
  background-color: mediumseagreen;
}
class C {
  background-color: thistle;
}
class D {
  margin-left: 0px;
}
class E {
  jusify-content: flex-start;
}
```
虽然标签上的类名变多了，但是 CSS 的输出体积会随着功能的增多而减缓增长，因为出现过一次的 CSS Rule 就不会再重复出现了。
## CSS规则顺序
与手写的工具/原子 CSS 不同，JS 库能让样式不依赖于 CSS 规则的插入顺序。在规则冲突的情况下，生效的不是标签上 class attribute 中的最后一个类，而是样式表中最后插入的规则。  
在实际场景中，这些库避免在同一个元素上写入多个规则冲突的类。它们会确保标签上书写在最后的类名生效。其他的被覆盖的类名则被规律掉，甚至压根不会出现在 DOM 上。
``` 
const styles = pseudoLib.create({
  red: {color: "red"},
  blue: {color: "blue"},
});

// 只会输出 blue 相关的 CSS
<div style={[styles.red, styles.blue]}>
  Always blue!
</div>

// 只会输出 red 相关的 CSS
<div style={[styles.blue, styles.red]}>
  Always red!
</div>
```
> 注意:只有使用最严格的原子 CSS 库才能实现这种可预测的行为。

如果一个类里有多个 CSS 规则，并且只有其中的一个 CSS 规则被覆盖，那么 CSS-in-JS 库没办法进行相关的过滤，这也是原子 CSS 的优势之一。  
如果一个类只有一个简单的 CSS 规则，如 margin: 0，而覆盖的是 marginTop: 10。像 margin: 0 这样的简写语法被扩展为 4 个不同的原子类，这个库就能更加轻松的过滤掉不该出现在 DOM 上的类名。
## 仍然喜欢 Tailwind？
只要你熟悉所有的 Tailwind 命名约定，你就可以很高效的完成 UI 编写。一旦你熟悉了这个设定，就很难回到手写每个 CSS 规则的时代了，就像你写 CSS-in-JS 那样。  
没什么能阻止你在原子 CSS-in-JS 的框架上建立你自己的抽象 CSS 规则，Styled-system 就能在 CSS-in-JS 库里完成一些类似的事情。它基于一些约定创造出一些原子规则，在 emotion 中使用它试试：
``` 
import styled from '@emotion/styled';
import { typography, space, color } from 'styled-system';

const Box = styled('div')(typography, space, color);
```
等效于：
``` 
<Box
  fontSize={4}
  fontWeight="bold"
  p={3}
  mb={[4, 5]}
  color="white"
  bg="primary"
>
  Hello
</Box>
```
甚至有可能在 JS 里复用一些 Tailwind 的命名约定，如果你喜欢的话。  
先看些 Tailwind 的代码：
``` 
<div className="absolute inset-0 p-4 bg-blue-500" />
```
我们在谷歌上随便找一个方案，比如 react-native-web-tailwindcss：  
``` 
import { t } from 'react-native-tailwindcss';

<View style={[t.absolute, t.inset0, t.p4, t.bgBlue500]} />;
```
就生产力的角度而言，并没有太大的不同。甚至可以用 TS 来避免错别字。

原文: 
[Facebook 重构：抛弃 Sass / Less ，迎接原子化 CSS 时代](https://github.com/sl1673495/blogs/issues/69)
