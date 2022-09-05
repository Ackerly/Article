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
### 更好的状态复用 
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
**难以追溯的方法与属性**
```
export default {
  mixins: [ a, b, c, d, e, f, g ], // 当然，这只是表示它混入了很多能力
  mounted() {
    console.log(this.name)
    // mmp!这个 this.name 来自于谁？我难道要一个个混入看实现？
  }
}
```
**覆盖、同名**  
同时想混入 mixin-a.js 和 mixin-b.js 以同时获得它们能力的时候，不幸的事情发生了：  
由于这两个 mixin 功能都定义了 this.name 作为属性,导致第一个name被覆盖，这种时候，你会深深怀疑，mixins 究竟是不是一种科学的复用方式。  
**代价很大**  
如果我的需求发生了改变，不再是一个简单的状态 name，而是分别需要 firstName 和 lastName。此时 name-mixin.js 混入的能力就会非常尴尬因为无法两次 mixins 同一个文件  
当然也是有解决方案的如：  
```
// 动态生成mixin
function genNameMixin(key, funcKey) {
  return {
    data() {
      return {
        [key]: genRandomName()
      }
    },
    methods: {
      [funcKey]: function(v) {
        this.[key] = v
      } 
    }
  }
}

export default {
  mixins: [
    genNameMixin('firstName', 'setFirstName'),
    genNameMixin('lastName', 'setLastName'),
  ]
}
```
通过动态生成 mixin 完成了能力的复用，但这样一来，无疑更加地增大了程序的复杂性，降低了可读性。  
**Hook 的状态复用写法：**  
```
// 单个name的写法
const { name, setName } = useName();

// 多个的写法
const { name : firstName, setName : setFirstName } = useName();

const { name : secondName, setName : setSecondName } = useName();
```
相比于 mixins的优点：
1. 方法和属性好追溯，谁产生的，哪儿来的一目了然
2. 没有重名、覆盖问题，内部的变量在闭包内，返回的变量支持定义别名
3. 多次使用

### 代码组织
一个页面中，N件事情的代码在一个组件内互相纠缠确实是在 Hooks 出现之前非常常见的一种状态。那么 Hooks 写法在代码组织上究竟能带来怎样的提升呢？  
无论是 vue 还是 react, 通过 Hooks 写法都能做到，将“分散在各种声明周期里的代码块”，通过 Hooks 的方式将相关的内容聚合到一起。  
这样带来的好处是显而易见的：“高度聚合，可阅读性提升”。伴随而来的便是 “效率提升，bug变少”。
### 比 class 组件更容易理解
在 react 的 class 写法中，随处可见各种各样的 .bind(this)。vue的computed: { a: () => { this } } 里的 this 也是 undefined。绑定虽然“必要”，但并不是“优点”，反而是“故障高发”地段。在Hooks 写法中，就完全不必担心 this 的问题了
## 开始使用hooks
**环境和版本**  
在 react 项目中， react 的版本需要高于 16.8.0。  
在 vue 项目中， vue3.x 是最好的选择，但 vue2.6+ 配合 @vue/composition-api，也可以开始享受“组合式API”的快乐。
**react 的 Hooks 写法**
```
// my-component.js
import { useState, useEffect } from 'React'

export default () => {
  // 通过 useState 可以创建一个 状态属性 和一个赋值方法
  const [ name, setName ] = useState('')

  // 通过 useEffect 可以对副作用进行处理
  useEffect(() => {
    console.log(name)
  }, [ name ])

  // 通过 useMemo 能生成一个依赖 name 的变量 message
  const message = useMemo(() => {
    return `hello, my name is ${name}`
  }, [name])

  return <div>{ message }</div>
}
```
**vue 的 Hooks 写法**
```
<template>
  <div>
    {{ message }}
  </div>
</template>
<script setup>
import { computed, ref } from 'vue'
// 定义了一个 ref 对象
const name = ref('')
// 定义了一个依赖 name.value 的计算属性
const message = computed(() => {
  return `hello, my name is ${name.value}`
})
</script>
```
**Reac实践**  
```
const { name, setName } = useName();
// 随机生成一个状态属性 name，它有一个随机名作为初始值
// 并且提供了一个可随时更新该值的方法 setName

import React from 'react';

export const useName = () => {
  // 这个 useMemo 很关键
  const randomName = React.useMemo(() => genRandomName(), []);
  const [ name, setName ] = React.useState(randomName)

  return {
    name,
    setName
  }
}
```
为什么不直接这样写：
```
const [ name, setName ] = React.useState(genRandomName())
```
因为这样写是不对的，每次使用该 Hook 的函数组件被渲染一次时，genRandom() 方法就会被执行一次，虽然不影响 name 的值，但存在性能消耗，甚至产生其他 bug。  
可以通过 React.useState(() => randomName()) 传参来避免重复执行，这样就不需要 useMemo 了  
**Vue实践**  
```
import { ref } from 'vue';

export const useName = () => {
  const name = ref(genRandomName())
  const setName = (v) => {
    name.value = v
  }
  return {
    name,
    setName
  }
}
```
**vue 和 react 自定义 Hook 的异同**  
相似点： 总体思路是一致的 都遵照着 "定义状态数据"，"操作状态数据"，"隐藏细节" 作为核心思路。    
差异点：vue3 的组件里， setup 是作为一个早于 “created” 的生命周期存在的，无论如何，在一个组件的渲染过程中只会进入一次。
React函数组件 则完全不同，如果没有被 memorized，它们可能会被不停地触发，不停地进入并执行方法，因此需要开销的心智相比于vue其实是更多的。

原文:
[浅谈：为啥vue和react都选择了Hooks🏂？](https://juejin.cn/post/7066951709678895141?utm_source=gold_browser_extension)
