<!-- TOC -->

- [Vue插件开发](#vue插件开发)
  - [功能](#功能)
  - [使用](#使用)
  - [开发插件](#开发插件)

<!-- /TOC -->
# Vue插件开发
## 功能
- 添加全局方法或属性
- 添加全局资源，如指令，过滤器，过渡
- 通过全局混入添加一些组件选项
- 添加Vue实例方法，把他们添加到Vue.prototype
- 一个库，提供自己的API

## 使用
- 通过全局方法使用插件
```
// 调用 `MyPlugin.install(Vue)`
Vue.use(MyPlugin)

new Vue({
  // ...组件选项
})
```
- 传入可选参数
```
Vue.use(MyPlugin, { someOption: true })
```
- Vue.use会自动阻止多次注册相同的插件，即使多次调用也只会注册一次

## 开发插件
插件应该暴露一个install，第一个参数是Vue构造器，第二个是一个可选的选项对象
```
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }

  // 2. 添加全局资源
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  })

  // 3. 注入组件选项
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
    ...
  })

  // 4. 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
}
```
