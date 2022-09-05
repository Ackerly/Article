# Vue3.0七大亮点
## 性能比2快1.2~2倍
### diff算法优化
- vue2中虚拟DOM是全量比较的。  
- Vue3中增加了静态标记PatchFlag。创建vnode的时候，内用是否可以变化，为其添加PatchFlag。diff的时候，只会比较PatchFlag的节点。PatchFlag是有类型的，比如一个可变化文本节点，会将其添加PatchFlag枚举值为TEXT的静态标记。在diff的时候，只需要对比文本内容。需要对比的内容更少了。PathFlag还有动态class、动态style、动态属性、动态key属性等枚举值。
### render阶段的静态提升（render阶段生成虚拟dom树阶段）
- vue2中数据变化是就会re-render组件，所有vnode重新创建一遍，形成新的vdom树
- vue3对不参与更新的vnode会做静态提升，只会创建一次，在rerender时直接复用
- 静态提升在第一次render不参与与更新的vnode节点的时候，保存它们的引用。re-render新时直接引用，不用创建
### 事件侦听缓存
- vue2中@click="onClick"被当做动态属性，diff时需要对比
- vue3中如果事件不会变化会将onClick缓存，该节点不会标记PatchFlag。在render和diff两个阶段事件侦听属性都节约了不必要的性能消耗。
## 按需编译，比Vue2更小
``` 
import { computed, watch, nextTick } from "vue";
```
如果没有用到watch，编译时会被tree shaking掉
## Compostion API：组合API/注入API
- Vue2使用的是Option风格，逻辑分散，需要分别在data/methods/mounted反复切换到对应位置，然后进行更改
- vue3中使用setup，比如实现count功能，把逻辑放在counter文件
``` 
import useCounter from './counter'

setup(){
  let {count,add} = useCounter() 
  return {
    count,add,
  }
}
```
## 更好的TS支持
- vue2不适合TS，因为vue2的option是个简单对象，而ts是一种类型系统，面向对象的语法
- vue结合TS的实践中用vue-class-component强化vue组件，让script支持TypeScript装饰器，vue-property-decorator 来增加更多结合 Vue 特性的装饰器，最终TS组件写法和js组件写法差别挺大
- vue3中量身打造defineComponent函数、在TS下更好利用类型腿短。Composition API代码风格中，比较有滴啊表性的API就是ref和reactive，很好支持了类型声明。
``` 
import { defineComponent, ref } from 'vue'
const Component = defineComponent({
    props: {
        success: { type: String },
        student: {
          type: Object as PropType<Student>,
          required: true
       }
    },
    setup() {
      const year = ref(2020)
      const month = ref<string | number>('9')
     
      month.value = 9 // OK
     const result = year.value.split('') // => Property 'split' does not exist on type 'number'
 }
```
## 自定义渲染API
vue官方实现的 createApp 会给我们的 template 映射生成 html 代码，但是要是你不想渲染生成到 html ，而是要渲染生成到 canvas 之类的不是html的代码的时候，那就需要用到 Custom Renderer API 来定义自己的 render 渲染生成函数了。
``` 
// 你自己实现一个createApp，比如是渲染到canvas的。
import { createApp } from "./runtime-render";
import App from "./src/App"; // 根组件

createApp(App).mount('#app');
```
## 更先进的组件
**Fragment组件**
``` 
<template>
  <div>Hello</div>
  <div>World</div>
</template>
```
vue2中组件必须有一个根节点，在vue3不用，vue3会创建一个虚拟Fragment节点,Fragment是虚拟的，不会在DOM树中出现  
**Suspense组件**
``` 
<Suspense>
  <template >
    <Suspended-component />
  </template>
  <template #fallback>
    Loading...
  </template>
</Suspense>
```
Suspended-component完全渲染之前，备用内容会被显示出来。如果是异步组件，Suspense可以等待组件被下载，或者在设置函数中执行一些异步操作。
## 更快的开发体验
在使用webpack作为开发构建工具时，npm run dev都要等一会，项目越大等的时间越长。热重载页有几秒的延迟，但是如果用vite来做vue3的开发构建工具，npm run dev 秒开，热重载也很快。
vite的原理还是用了浏览器支持import关键字了，启动项目不用webpack构建工具先构建了，浏览器直接请求路由对应的代码文件，代理服务器针对单个文件进行编译并返回。如果请求的文件里还import了其他文件，同理浏览器继续发请求，代理服务器返回。就这样实现了npm run dev时无需编译，实时请求实时编译。

原文: 
[Vue3.0七大亮点是什么？？](https://juejin.cn/post/6968261755063500831?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
