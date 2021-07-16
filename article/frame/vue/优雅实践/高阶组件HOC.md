# 高阶组件 HOC
## 概述
``` 
<template>
    <div v-if="error">failed to load</div>
    <div v-else-if="loading">loading...</div>
    <div v-else>hello {{result.name}}!</div>
</template>

<script>
export default {
  data() {
    return {
        result: {
          name: '',
        },
        loading: false,
        error: false,
    },
  },
  async created() {
      try {
        // 管理loading
        this.loading = true
        // 取数据
        const data = await this.$axios('/api/user')  
        this.data = data
      } catch (e) {
        // 管理error
        this.error = true  
      } finally {
        // 管理loading
        this.loading = false
      }
  },
}
</script>
```
对于异步请求的时候都要有loading、error状态，都需要取数据的逻辑，并且要管理这些状态。HOC（high order component）也就是高阶组件，用来处理这些。
## 什么是高阶组件
高阶组件就是一个函数接受一个组件为参数，返回一个包装后的组件。  
**React中**  
在React里，组件是Class，所以高阶组件有时候会用装饰器语法来实现，因为装饰器的本质也是接受一个Class返回一个新的Class。在React的世界里，高阶组件就是f(Class) -> 新的Class。  
**在Vue中**  
在Vue中，组件是一个对象，所以高阶组件就是一个函数接受一个对象，返回一个新的包装好的对象。高阶组件就是f（object）-> 新的object
## 智能组件和木偶组件
木偶组件: 就是像牵线木偶一样，只根据外部传入的props去渲染相应的视图，而不管这个数据是从哪里来的。  
智能组件: 一般包在木偶组件的外部，通过请求等方式获取数据，传入给木偶组件，控制它的渲染。
## 实现
1. 高阶组件接受木偶组件和请求的方法作为参数
2. 在mounted生命周期中请求到数据
3. 把请求的数据通过props传递给木偶组件  

把loadding、error等状态，还有加载中、加载错误等对应的视图都要在新返回的包装组件，即return的那个新的对象中定义好。
``` 
const withPromise = (wrapped, promiseFn) => {
  return {
    name: "with-promise",
    data() {
      return {
        loading: false,
        error: false,
        result: null,
      };
    },
    async mounted() {
      this.loading = true;
      const result = await promiseFn().finally(() => {
        this.loading = false;
      });
      this.result = result;
    },
  };
};
```
在参数中：
1. wrapped就是需要被包裹的组件对象
2.promiseFunc也就是请求对应的函数，需要返回一个 Promise。

使用render函数把传入的wrapped木偶组件给包裹起来，这样就形成智能组件获取数据 -> 木偶组件消费数据，这样的数据流动了。
``` 
const withPromise = (wrapped, promiseFn) => {
  return {
    data() { ... },
    async mounted() { ... },
    render(h) {
      return h(wrapped, {
        props: {
          result: this.result,
          loading: this.loading,
        },
      });
    },
  };
};
```
木偶组件：
``` 
const view = {
  template: `
    <span>
      <span>{{result?.name}}</span>
    </span>
  `,
  props: ["result", "loading"],
};
```
用withPromise 包裹view
``` 
/ 假装这是一个 axios 请求函数
const request = () => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ name: "ssh" });
    }, 1000);
  });
};

const hoc = withPromise(view, request)
```
在父组件中渲染
``` 
<div id="app">
  <hoc />
</div>

<script>
 const hoc = withPromise(view, request)

 new Vue({
    el: 'app',
    components: {
      hoc
    }
 })
</script>
```
加上加载中和加载失败的视图
``` 
const withPromise = (wrapped, promiseFn) => {
  return {
    data() { ... },
    async mounted() { ... },
    render(h) {
      const args = {
        props: {
          result: this.result,
          loading: this.loading,
        },
      };

      const wrapper = h("div", [
        h(wrapped, args),
        this.loading ? h("span", ["加载中……"]) : null,
        this.error ? h("span", ["加载错误"]) : null,
      ]);

      return wrapper;
    },
  };
};
```
## 完善
目前高阶组件还缺少的一些功能
1. 要拿到子组件上定义的参数，作为初始化发送请求的参数
2. 要监听子组件中请求参数的变化，并且重新发送请求。
3. 外部组件传递给 hoc 组件的参数现在没有透传下去。
使用ref
``` 
const withPromise = (wrapped, promiseFn) => {
  return {
    data() { ... },
    async mounted() {
      this.loading = true;
      // 从子组件实例里拿到数据
      const { requestParams } = this.$refs.wrapped
      // 传递给请求函数
      const result = await promiseFn(requestParams).finally(() => {
        this.loading = false;
      });
      this.result = result;
    },
    render(h) {
      const args = {
        props: {
          result: this.result,
          loading: this.loading,
        },
        // 这里传个 ref，就能拿到子组件实例了，和平常模板中的用法一样。
        ref: 'wrapped'
      };

      const wrapper = h("div", [
        this.loading ? h("span", ["加载中……"]) : null,
        this.error ? h("span", ["加载错误"]) : null,
        h(wrapped, args),
      ]);

      return wrapper;
    },
  };
};
```
第二点子组件参数变化时，父组件要响应式重新发送请求，并且把心数据带给子组件
``` 
const withPromise = (wrapped, promiseFn) => {
  return {
    data() { ... },
    methods: {
      // 请求抽象成方法
      async request() {
        this.loading = true;
        // 从子组件实例里拿到数据
        const { requestParams } = this.$refs.wrapped;
        // 传递给请求函数
        const result = await promiseFn(requestParams).finally(() => {
          this.loading = false;
        });
        this.result = result;
      },
    },
    async mounted() {
      // 立刻发送请求，并且监听参数变化重新请求
      this.$refs.wrapped.$watch("requestParams", this.request.bind(this), {
        immediate: true,
      });
    },
    render(h) { ... },
  };
};
```
第三点透传属性，只要渲染子组件的时候吧$attrs、listeners、$scopedSlots 传递下去即可。
``` 
const withPromise = (wrapped, promiseFn) => {
  return {
    ...,
    render(h) {
      const args = {
        props: {
          // 混入 $attrs
          ...this.$attrs,
          result: this.result,
          loading: this.loading,
        },

        // 传递事件
        on: this.$listeners,

        // 传递 $scopedSlots
        scopedSlots: this.$scopedSlots,
        ref: "wrapped",
      };

      const wrapper = h("div", [
        this.loading ? h("span", ["加载中……"]) : null,
        this.error ? h("span", ["加载错误"]) : null,
        h(wrapped, args),
      ]);

      return wrapper;
    },
  };
};
```
## 组合
### 函数式compose
``` 
function compose(...funcs) {
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```
compose(a, b, c) 返回的是一个新的函数，这个函数会把传入的几个函数 嵌套执行
### 循环式compose
返回一个函数，把传入的函数数组从右往左的执行，并且上一个函数的返回值会作为下一个函数执行的参数。
``` 
function compose(...args) {
  return function(arg) {
    let i = args.length - 1
    let res = arg
    while(i >= 0) {
     let func = args[i]
     res = func(res)
     i--
    }
    return res
  }
}
```
### 改造withPromise
比如 compose(a, b) 来说，b(arg) 返回的值就会作为 a 的参数，进一步调用 a(b(args))，这需要保证 compose里接受的函数，每一项的参数都只有一个。高阶化withPromise，让它返回一个只接受一个参数的函数
``` 
const withPromise = (promiseFn) => {
  // 返回的这一层函数 wrap，就符合我们的要求，只接受一个参数
  return function wrap(wrapped) {
    // 再往里一层 才返回组件
    return {
      mounted() {},
      render() {},
    }
  }
}
```
## 真实的业务场景
vue-router 可以配置异步路由，但是在网速很慢的情况下，这个异步路由对应的 chunk 也就是组件代码，要等到下载完成后才会进行跳转。这段下载异步组件的时间我们想让页面展示一个 Loading 组件，让交互更加友好。
```
const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
})

new VueRouter({
    routes: [{
        path: '/',
        component: () => import('./MyComponent.vue')
        component: AsyncComponent
    }]
})

```
但是上面代码根本不支持，因为在 vue-router 的实现中不会去帮你渲染 Loading 组件。  
可以使用vue-router 先跳转到一个 容器组件，这个 容器组件 帮我们利用 Vue 内部的渲染机制去渲染 AsyncComponent ，渲染出 loading 状态了。  
由于 vue-router 的 component 字段接受一个 Promise，因此我们把组件用 Promise.resolve 包裹一层。
``` 
function lazyLoadView (AsyncView) {
  const AsyncHandler = () => ({
    component: AsyncView,
    loading: require('./Loading.vue').default,
    error: require('./Timeout.vue').default,
    delay: 400,
    timeout: 10000
  })

  return Promise.resolve({
    functional: true,
    render (h, { data, children }) {
      // 这里用 vue 内部的渲染机制去渲染真正的异步组件
      return h(AsyncHandler, data, children)
    }
  })
}
  
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: () => lazyLoadView(import('./Foo.vue'))
    }
  ]
})
```

参考:  
[Vue 进阶必学之高阶组件 HOC](https://juejin.cn/post/6844904116603486221)
