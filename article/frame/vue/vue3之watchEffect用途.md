# Vue3的watchEffect用途
## 与 watch 有什么不同
- watchEffect 不需要指定监听的属性，它会自动的收集依赖， 只要回调中引用到了 响应式的属性，那么当这些属性变更的时候，这个回调都会执行，而 watch 只能监听指定的属性而做出变更
- watch 可以获取到新值与旧值（更新前的值），而 watchEffect 是拿不到的
- watchEffect 如果存在的话，在组件初始化的时候就会执行一次用以收集依赖（与computed同理），而后收集到的依赖发生变化，这个回调才会再次执行，而 watch 不需要，因为他一开始就指定了依赖。

## 进阶 
**停止监听**  
watchEffect 会返回一个用于停止这个监听的函数，例如：
``` 
const stop = watchEffect(() => {
  /* ... */
})

// later
stop()
```
watchEffect  setup 或者 生命周期里面注册的话，在组件取消挂载的时候会自动的停止掉。  
**使 side effect 无效**  
不可预知的接口请求就是一个 side effect,假设现在用一个用户ID去查询用户的详情信息，然后监听了这个用户ID， 当用户ID 改变的时候就会去发起一次请求，用watch 就可以做到。但是如果在请求数据的过程中，用户ID发生了多次变化，那么就会发起多次请求，而最后一次返回的数据将会覆盖掉之前返回的所有用户详情。这不仅会导致资源浪费，还无法保证 watch回调执行的顺序。而使用 watchEffect 我们就可以做到。
``` 
watchEffect(() => {
      // 异步api调用，返回一个操作对象
      const apiCall = someAsyncMethod(props.userID)

      onInvalidate(() => {
        // 取消异步api的调用。
        apiCall.cancel()
      })
})
```
onInvalidate(fn)传入的回调会在 watchEffect 重新运行或者 watchEffect 停止的时候执行

原文: 
[浅谈Vue3的watchEffect用途](https://www.codenong.com/s1190000023669309/?content_source_url=https://github.com/vue3/vue3-News)
