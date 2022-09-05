# 为什么不需要再Vue3中使用Vuex
Vue3公开了底层的响应式系统，并引入构建应用程序的新方法。新的响应式系统功能强大，可用于共享状态管理。
## 你需要一个共享状态吗
需要共享状态和数据的情况
- 多个组件使用相同的数据
- 多个模块需要单独接入数据
- 深层嵌套的组件需要与其他组件进行数据通讯

## 新的解决方案
共享状态需要符合两个标准：
- 响应式：当状态改变时，使用他们的组件也相应更新
- 可用性：可以在任何组件中访问
### 响应式
Vue3对外暴露其响应式系统的众多行数，可以使用reactive创建一个reactive变量（也可以使用ref函数）
``` 
import { reactive } from 'vue';

export const state = reactive({ counter: 0 });
```
响应式函数返回的对象是一个Proxy对象，它可以监听其属性的更改。当在组件模板中使用时，每当响应值发生变化时，组件就会重新渲染：
``` 
<template>
  <div>{{ state.counter }}</div>
  <button type="button" @click="state.counter++">Increment</button>
</template>

<script>
  import { reactive } from 'vue';
  export default {
    setup() {
      const state = reactive({ counter: 0 });
      return { state };
    }
  };
</script>
```
### 可用性
多组件共享状态，使用Vue3的provide与inject：
``` 
import { reactive, provide, inject } from 'vue';

export const stateSymbol = Symbol('state');
export const createState = () => reactive({ counter: 0 });

export const useState = () => inject(stateSymbol);
export const provideState = () => provide(
  stateSymbol, 
  createState()
);
```
将Symbol作为键值传递给provide时，该值将通过inject方法使任何组件可用。键在提供和检索时使用相同的Symbol名称. 
如果你在最上层的组件上提供值，那么它将在所有组件中都可用，或者你也可以在应用的主文件上调用provide
``` 
import App from './App.vue';
import { stateSymbol, createState } from './store';

const app = createApp(App);
app.provide(stateSymbol, createState());
app.mount('#app');

<script>
  import { useState } from './state';
  export default {
    setup() {
      return { state: useState() };
    }
  };
</script>
```
### 更健壮的方案
上面的方案有一个缺点：你不知道谁做的状态更改，因为状态可以直接更改，不受任何限制。  
可以通过readonly函数包装状态，使其成为受保护的状态，需要修改状态时通过单独的函数来处理。
``` 
import { reactive, readonly } from 'vue';

export const createStore = () => {
  const state = reactive({ counter: 0 });
  const increment = () => state.counter++;

  return { increment, state: readonly(state) };
}
```
外部代码只能访问状态，只有导出的函数才可以修改状态。通过受保护的状态避免不必要的修改，新的解决方案相对而言更接近于Vuex。

原文: 
[为什么不需要在Vue3中使用Vuex](https://juejin.cn/post/6856718746694713352?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
