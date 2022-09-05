# 使用JSX/TSX 开发Vue3组件
## 直接使用TSX
vue3可以直接使用tsx开发，唯一需要处理的就是children
``` 
<div>
    <p>1</p>
    <p>1</p>
</div> 

// tsx写法
<div>
    {
      [
        <p>1</p>,
        <p>1</p>
      ]
    }
</div> 
```
封装一个工具函数
``` 
function JSXFactory(tag: any, props: any, ...children: any) {
  if (
    typeof tag !== 'string' &&
    typeof tag !== 'symbol' &&
    !tag.__isTeleport &&
    !tag.__isKeepAlive
  ) {
    // Component
    children = children[0]
  }
  return createVNode(tag, props, children)
}
```
 tsconfig.json 中配置 jsxFactory,但是这个函数限制在为组件传递slots只能
 ``` 
 const App = {
     setup() {
         const slots = {
             foo: () => <p>foo</p>,
             bar: () => <p>bar</p>,
         }
         return () => <Hello>{ slots }</Hello>
     }
 }
 ```
有了 JSXFactory 工具函数之后要用jsx 插件的原因是使用了jsx语法后丢失了很多模板中提供的便利能力，例如事件修饰符、v-model 之类的， 因此 jsx 插件还是有必要的，但是不是必须的。
## 既支持JSX又支持TSX
tsx 中不支持 amespaced attribute,但babel是支持，可以在jsx这么写
``` 
<p v-on:click={ handler } ></p>
```
但是 tsx 中则不行，为了语法统一，我决定不允许在属性名中使用 :，而是使用 - 替代 
``` 
<p v-on-click={ handler } ></p>
```
无论是 jsx 还是 tsx，修饰符都不允许使用 .，而是使用 _ 代替：
``` 
 <p v-on-click_stop={ handler } ></p>
```
于事件，vue-next-jsx 支持全部的模板中可用语法，例如：
``` 
<div v-on-click_middle={ handler }></div>
<div v-on-click_stop={ handler }></div>
<div v-on-click_right={ handler }></div>
<div v-on-keyup_esc={ handler }></div>

<div v-on={ obj }></div>
```
## v-model
Vue2 中的 .sync 被 v-model:foo 代替了，例如：
``` 
<!-- Vue2 -->
<Comp :foo.sync="val" />
<!-- Vue3 -->
<Comp v-model:foo="val" />
```
在 jsx 中，把 : 换成 - ：
``` 
<Comp v-model-foo_a_b={ refVal.value }/> 
```
## v-bind
在 j/tsx 中不需要 v-bind，直接使用 jsxExpressionContainer 和 jsxSpreadAttribute 代替：
``` 
<Comp foo={ refVal.value } { ...props } bar="bar" />
```
## slots
v-slot会导致类型丢失
``` 
<Comp>
    <template v-slot-foo="props">
    </template>
</Comp> 
```
这里的 props 没有类型，它就是一个字符串,推荐像如下这样为组件传递插槽
```
<Comp>{ mySlots }</Comp> 
```
插槽 mySlots 可以自行构建：
``` 
const mySlots = {
    default: () => [<p>默认插槽</p>]
} 
```
## KeepAlive 和 Teleport
这两个组件比较特殊，他们的子节点不会作为 slots 存在，而是当做正常的 children,vue-next-jsx 会进行处理。
## Optimization mode
在 babel.config.json 中打开优化模式：
``` 
{
  "presets": [
    "@babel/env"
  ],
  "plugins": [
    ["@hcysunyang/vue-next-jsx", {
      // 开启优化模式
      "optimizate": true
    }]
  ]
}
```
可以查看 vue-next-jsx 的测试用例生成的 ，并与 Vue3 Compiler 对比，他们的行为是一致的，包括及其复杂的情况。
## 指定 source
source 指的是 ImportDeclaration 语句的 source，例如：
``` 
import { createApp } from 'vue'
```
这里的 source 就是 vue ，但是你可能安装的不是 vue 而是 @vue/runtime-dom ，这时你需要指定 source：
``` 
{
  "presets": [
    "@babel/env"
  ],
  "plugins": [
    ["@hcysunyang/vue-next-jsx", {
      // 指定 source
      "source": "@vue/runtime-dom"
    }]
  ]
} 
```
## v-html / v-text
在 jsx 中支持这两个指令意义不大，全当顺手，它们的使用与在模板中相同
``` 
<p v-html={ refHtml.value }></p>
<p v-text={ refText.value }></p>
```

原文: 
[使用 JSX/TSX 开发 Vue3 组件](https://juejin.cn/post/6914517242298236942?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
