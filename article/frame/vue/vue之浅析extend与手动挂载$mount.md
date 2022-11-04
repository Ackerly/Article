# vue之浅析extend与手动挂载$mount
## 为什么使用
### 普通组件的局限性
**普通组件的引用方式**  
ButtonCounter.vue  
``` 
<template>
  <button v-on:click="count++">You clicked me {{ count }} times.</button>
</template>

<script>
export default {
  name: "ButtonCounter",
  components: {},
  data() {
    return {
        count: 0
    }
  }
}
</script>
```
view.vue  
``` 
<template>
  <ButtonCounter />
</template>

<script>
import ButtonCounter from "./ButtonCounter.vue"
export default {
  name: "VueAppComponent2",
  components: {
    ButtonCounter
  }
}
</script>
```
**普通组件的特点**  
由于 vue 框架多用于单页面应用，因此通常整个应用只有一个根节点，即 #app，如下：  
``` 
// main.js
import Vue from "vue"
import App from "./App.vue"

new Vue({
  render: h => h(App)
}).$mount("#app")
```
因此，项目内的所有内容全部在 #app 根节点下渲染。  
其次，通常引入的组件会进行局部注册，模板引用组件标签，这导致组件只能在预设的位置进行渲染。  
**普通组件的问题**  
如下场景普通组件无法很好的解决：  
1. 组件模版是从服务端异步获取到的，并需要动态渲染
2. 实现类似原生的调用。例如，调用 window.alert() 就可以弹出弹窗

## 如何实现 
### vue2 的实现 
``` 
// CreateButtomCounter.js

import Vue from "vue"
import ButtonCounter from "./ButtonCounter.vue"

export const CreateButtomCounter = () => {
  const ButtonCounterCom = Vue.extend(ButtonCounter)
  const instance = new ButtonCounterCom().$mount()  
  const dom = document.querySelector("#root")
  dom.appendChild(instance.$el)
}
```
**view.vue**  
``` 
<template>
  <div>
    <button @click="CreateButtomCounter">点击加载组件</button>
    <div id="root"></div>
  </div>
</template>
<script>
import { CreateButtomCounter } from "./CreateButtomCounter.js"
export default {
  name: "VueAppComponent",
  methods: {
    CreateButtomCounter
  }
}
</script>
```

### 浅析
**new Vue**  
特点：  
1. 可以通过自身components引用vue.extend构造，通过自身data向构造传参
2. 可以通过自身component引用组件模版，通过自身data向组件传参  
使用范围：仅限于自身

``` 
new Vue({
  el: "#app1",
  data: {
    msg: "vue实例参数"
  },
  components: {
    dt1: {
      template: "#dt1"
    },
    vueapple: Profile //【引入构造】
  }
});
```

**Vue.extend()**  
通过参数传递一个配置对象（这个配置对象就是我们模板中export default的那个对象，例如data,methods,props等等）都可以传递，接下来该函数会根据我们的配置对象生成一个继承自Vue的子组件构造函数  
特点：只能通过自身初始化结构  
使用范围：  
1. 挂载在某元素下
2. 被vue实例的components引用
3. Vue.component组件引用

``` 
const ButtonCounterCom = Vue.extend(ButtonCounter)
```
这里创建了一个构造器，可以解决普通模板的问题。

**$mount()**  
$mount 方法支持传入 2 个参数，第一个是 el，它表示挂载的元素，可以是字符串，也可以是 DOM 对象，如果是字符串在浏览器环境下会调用 query 方法转换成 DOM 对象的。第二个参数是和服务端渲染相关，在浏览器环境下不需要传第二个参数。  
``` 
const instance = new ButtonCounterCom().$mount()
```
这里，$mount() 不带参数，会把组件在内存中渲染完毕。因为没有挂载到节点上，因此显示不了组件。此时的 instance 已经是一个标准的 Vue 组件实例，因此它的 $el 属性也可以被访问  
``` 
dom.appendChild(instance.$el)
```
instance.$el 拿到的就是组件对应的dom元素,可以直接操作dom。  


原文:  
[vue之浅析extend与手动挂载$mount](https://mp.weixin.qq.com/s/eUexOK4BpPvT7Wn8AwLqZw)
