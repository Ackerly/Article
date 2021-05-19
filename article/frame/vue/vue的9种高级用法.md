# VUE的9种高级用法
## 函数组件
函数组件是无状态的，没有生命周期或方法，因此无法实例化，通过在SFC添加functional：true属性或者在模板添加functional。因为没有实例引用，所以渲染性能提升了不少。
```
<template functional>
  <div class="book">
    {{props.book.name}} {{props.book.price}}
  </div>
</template>

<script>
Vue.component('book', {
  functional: true,
  props: {
    book: {
      type: () => ({}),
      required: true
    }
  },
  render: function (createElement, context) {
    return createElement(
      'div',
      {
        attrs: {
          class: 'book'
        }
      },
      [context.props.book]
    )
  }
})
</script>
```
## 深层选择器
修改第三方组件的CSS，这些都是scoped，移除scope或打开一个新的样式  
用法：深层选择器 >>> /deep/ ::v-deep
```
<style scoped>
>>> .scoped-third-party-class {
  color: gray;
}
</style>

<style scoped>
/deep/ .scoped-third-party-class {
  color: gray;
}
</style>

<style scoped>
::v-deep .scoped-third-party-class {
  color: gray;
}
</style>
```
## 高级“watcher”
**立即执行**
使用immediate
```
watch:{
    value:{
        handle:function(o,n){},
        immediate:true
    }
}
```
**深度监听**
使用deep：true
```
watch:{
    value: {
        handle: () {},
        deep:true
    }
}
```
**多个handlers**
watch可以设置为数组，支持类型为String，Function，Object。触发后注册的watch处理程序将被一一调用
```
watch: {
  value: [
    'printValue',
    function (val, oldVal) {
      console.log(val)
    },
    {
      handler: 'printValue',
      deep: true
    }
  ]
},
methods : {
  printValue () {
    console.log(this.value)
  }
}
```
**订阅多个变量突变**
watcher不能监听多个变量，可以将目标组在一起作为一个新的computed，并监视这个新的变量
```
computed: {
  multipleValues () {
    return {
      value1: this.value1,
      value2: this.value2,
    }
  }
},
watch: {
  multipleValues (val, oldVal) {
    console.log(val)
  }
}
```
4.事件参数:$event
**原生事件**
与默认事件（DOM事件或窗口事件）相同
```
<template>
  <input type="text" @input="handleInput('hello', $event)" />
</template>

<script>
export default {
  methods: {
    handleInput (val, e) {
      console.log(e.target.value) // hello
    }
  }
}
</script>

```
**自定义事件**
参数是从子组件捕获的值
```
<template>
  <input type="text" @input="handleInput('hello', $event)" />
</template>

<script>
export default {
  methods: {
    handleInput (val, e) {
      console.log(e.target.value) // hello
    }
  }
}
</script>
```
## 路由器参数解耦
处理组件中路由器参数的方式
```
export default {
  methods: {
    getRouteParamsId() {
      return this.$route.params.id
    }
  }
}
```
组件内部使用$route会对某个URL产生强耦合，限制了组件的灵活性  
通过向路由添加props
``` 
const router = new VueRouter({
  routes: [{
    path: '/:id',
    component: Component,
    props: true
  }]
})
export default {
  props: ['id'],
  methods: {
    getParamsId() {
      return this.id
    }
  }
}
```
传入函数以返回自定义props
```
const router = new VueRouter({
  routes: [{
    path: '/:id',
    component: Component,
    props: router => ({ id: route.query.id })
  }]
})
```
## 自定义组件的双向绑定
v-model是双向绑定，input是默认的更新时间，可以通过$emit更新该值
```
<my-checkbox v-model="val"></my-checkbox>

<template>
  <input type="checkbox" :value="value" @input="handleInputChange(value)" />
</template>

<script>
export default {
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    handleInputChange (val) {
      console.log(val)
    }
  }
}
</script>
```
使用sync修饰符，仅触发update:<your_prop> 通过事件系统对属性进行突变。
```
<custom-component :value.sync="value" />
```
## 组件声明周期Hook
监听子组件的生命周期
```
<script>
export default {
  mounted () {
    this.$emit('onMounted')
  }
}
</script>

<!-- Parent -->
<template>
  <Child @onMounted="handleOnMounted" />
</template>
```
另一种是使用@hook:mount在Vue内部系统中使用
```
<template>
  <Child @hook:mounted="handleOnMounted" />
</template>
```
## 事件监听APIS
通常要在beforeDestroy中使用this.timer获取计时器ID来进行销毁
```
export default {
  data () {
    return {
      timer: null
    }
  },
  mounted () {
    this.timer = setInterval(() => {
      console.log(Date.now())
    }, 1000)
  },
  beforeDestroy () {
    clearInterval(this.timer)
  }
}
```
使用$once来放弃不必要的东西
``` 
export default {
  mounted () {
    let timer = null
    timer = setInterval(() => {
      console.log(Date.now())
    }, 1000)
    this.$once('hook:beforeDestroy', () => {
      clearInterval(timer)
    })
  }
}
```
## 以编程方式挂载组件
可以通过全局上下文来进行调用，比如通过$popup() 或 $modal.open() 打开弹出窗口或模态窗口。
``` 
import Vue from 'vue'
import Popup from './popup'

const PopupCtor = Vue.extend(Popup)

const PopupIns = new PopupCtr()

PopupIns.$mount()

document.body.append(PopupIns.$el)

Vue.prototype.$popup = Vue.$popup = function () {
  PopupIns.open()
}
```

参考：[听说你熟练使用Vue，那这9种Vue技术你掌握了吗？](https://juejin.cn/post/6862560722531352583?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News%3Fcontent_source_url%3Dhttps%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
