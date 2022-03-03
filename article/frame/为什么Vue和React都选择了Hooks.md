# 为什么Vue和React都选择了Hooks
**hooks 的定义**  
"hooks" 直译是 “钩子”，它并不仅是 react，甚至不仅是前端界的专用术语，而是整个行业所熟知的用语。通常指：  
> 系统运行到某一时期时，会调用被注册到该时机的回调函数。

比较常见的钩子有：windows 系统的钩子能监听到系统的各种事件，浏览器提供的 onload 或addEventListener 能注册在浏览器各种时机被调用的方法。  
在 react@16.x 之前，当我们谈论 hooks 时，我们可能谈论的是“组件的生命周期”。但是现在，hooks 则有了全新的含义,以 react 为例，hooks 是：  
一系列以 “use” 作为开头的方法，它们提供了让你可以完全避开 class式写法，在**函数式组件**中完成生命周期、状态管理、逻辑复用等几乎全部组件开发工作的能力。
```
import { useState, useEffect, useCallback } from 'react';
// 比如以上这几个方法，就是最为典型的 Hooks
```
而在 vue 中hooks 的定义可能更模糊,在 vue **组合式API**里，以 “use” 作为开头的，一系列提供了组件复用、状态管理等开发能力的方法,如：useSlots、 useAttrs、 useRouter 等。 
```
import { useSlots, useAttrs } from 'vue';
import { useRouter } from 'vue-router';
// 以上这些方法，也是 vue3 中相关的 Hook！
```
但主观来说vue 组合式API其本身就是“vue hooks”的关键一环，起到了 react hooks里对生命周期、状态管理的核心作用。（如 onMounted、 ref 等等）。  
**命名规范 和 指导思想**  
在 react 官方文档里
- 假设任何以「use」开头并紧跟着一个大写字母的函数就是一个 Hook
- 只在 React 函数组件中调用 Hook，而不在普通函数中调用 Hook
- 只在最顶层使用 Hook，而不要在循环，条件或嵌套函数中调用 Hook。
  
## 为什么需要 hooks**  
**更好的状态复用**  
在 class 组件模式下，状态逻辑的复用是一件困难的事情。  
假设有如下需求：  
> 当组件实例创建时，需要创建一个 state 属性：name，并随机给此 name 属性附一个初始值。除此之外，还得提供一个 setName 方法。可以在组件其他地方开销和修改此状态属性。  
> 更重要的是: 这个逻辑要可以复用，在各种业务组件里复用这个逻辑

在拥有 Hooks 之前，首先会想到的解决方案一定是 mixin,代码如下：
```
// 混入文件：name-mixin.js
export default {
  data() {
    return {
      name: genRandomName() // 假装它能生成随机的名字
    }
  },
  methods: {
    setName(name) {
      this.name = name
    }
  }
}
// 组件：my-component.vue
<template>
  <div>{{ name }}</div>
<template>
<script>
import nameMixin from './name-mixin';
export default {
  mixins: [nameMixin],
  // 通过mixins, 你可以直接获得 nameMixin 中所定义的状态、方法、生命周期中的事件等
  mounted() {
    setTimeout(() => {
      this.setName('Tom')
    }, 3000)
  }
}
<script>
```
mixins 虽然提供了这种状态复用的能力，但它的弊端实在太多了
**难以追溯的方法与属性 **
2. 





参考：
[浅谈：为啥vue和react都选择了Hooks🏂？](https://juejin.cn/post/7066951709678895141?utm_source=gold_browser_extension)
