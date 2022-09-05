# vue2和vue3的nextTick实现原理
## JS执行机制
 JS 是单线程的，一次只能干一件事，即同步，就是说所有的任务都需要排队，后面的任务需要等前面的任务执行完才能执行，如果前面的任务耗时过长，后面的任务就需要一直等，这是非常影响用户体验的，所以才出现了异步的概念
- 同步任务：指排队在主线程上依次执行的任务
- 异步任务：不进入主线程，而进入任务队列的任务，又分为宏任务和微任务
- 宏任务：渲染事件、请求、script、setTimeout、setInterval、Node中的setImmediate 等
- 微任务：Promise.then、MutationObserver(监听DOM)、Node 中的 Process.nextTick等

执行栈中的同步任务执行完后，就会去任务队列中拿一个宏任务放到执行栈中执行，执行完该宏任务中的所有微任务，再到任务队列中拿宏任务，即一个宏任务、所有微任务、渲染、一个宏任务、所有微任务、渲染...(不是所有微任务之后都会执行渲染)，如此形成循环，即事件循环  
nextTick 就是创建一个异步任务，那么它自然要等到同步任务执行完成后才执行
##Vue2
### nextTick 用法
``` 
<template>
  <div>{{ name }}</div>
</template>
<script>
export default {
  data() {
    return {
      name: ""
    }
  },
  mounted() {
    console.log(this.$el.clientHeight) // 0
    this.name = "沐华"
    console.log(this.$el.clientHeight) // 0
    this.$nextTick(() => {
      console.log(this.$el.clientHeight) // 18
    });
  }
};
</script>
```
### 原理分析
执行 this.name = '沐华' 的时候，就会触发 Watcher 更新，watcher 会把自己放到一个队列,然后调用nextTick
> 用队列的原因是比如多个数据变更就更新视图多次的话，性能上就不好了，所以对视图更新做一个异步更新的队列，避免重复计算和不必要的DOM操作，在下一轮事件循环的时候刷新队列，并执行已去重的任务(nextTick的回调函数)，更新视图

``` 
export function queueWatcher (watcher: Watcher) {
  ...
  // 因为每次派发更新都会引起渲染，所以把所有 watcher 都放到 nextTick 里调用
  nextTick(flushSchedulerQueue)
}
```
参数 flushSchedulerQueue 方法就会被放入事件循环，主线程任务的行完后就会执行这个函数，对 watcher 队列排序、遍历、执行 watcher 对应的 run 方法，然后 render，更新视图。  
this.name = '沐华' 的时候，任务队列可以简单理解成这样 [flushSchedulerQueue]，  
然后下一行 console.log(...)，由于会更新视图的任务 flushSchedulerQueue 在任务队列里没有执行，所以无法拿到更新后的视图  
然后执行到 this.$nextTick(fn) 的时候，添加一个异步任务，这时的任务队列可以简单理解成这样 [flushSchedulerQueue, fn]  
然后同步任务就执行完了，接着按顺序执行任务队列里的任务，第一个任务执行就会更新视图，后面自然能得到更新后的视图了  
### nextTick 源码剖析  
源码分为两部分，一是判断当前环境能使用的最合适的 API 并保存异步函数，二是调用异步函数 执行回调队列  
**环境判断**  
判断用哪个宏任务或微任务，因为宏任务耗费的时间是大于微任务的，所以成先使用微任务，判断顺序如下Promise>MutationObserver>setImmediate>setTimeout
``` 
export let isUsingMicroTask = false // 是否启用微任务开关
const callbacks = [] // 回调队列
let pending = false // 异步控制开关，标记是否正在执行回调函数

// 该方法负责执行队列中的全部回调
function flushCallbacks () {
  // 重置异步开关
  pending = false
  // 防止nextTick里有nextTick出现的问题
  // 所以执行之前先备份并清空回调队列
  const copies = callbacks.slice(0)
  callbacks.length = 0
  // 执行任务队列
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}
let timerFunc // 用来保存调用异步任务方法
// 判断当前环境是否支持原生 Promise
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  // 保存一个异步任务
  const p = Promise.resolve()
  timerFunc = () => {
    // 执行回调函数
    p.then(flushCallbacks)
    // ios 中可能会出现一个回调被推入微任务队列，但是队列没有刷新的情况
    // 所以用一个空的计时器来强制刷新任务队列
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  // 不支持 Promise 的话，在支持MutationObserver的非 IE 环境下
  // 如 PhantomJS, iOS7, Android 4.4
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // 使用setImmediate，虽然也是宏任务，但是比setTimeout更好
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  // 以上都不支持的情况下，使用 setTimeout
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```
环境判断结束就会得到一个延迟回调函数 timerFunc,然后进入核心的 nextTick
### nextTick()
Vue.nextTick() 或者 this.$nextTick() 都是调用 nextTick() 这个方法  
主要逻辑就是：
- 把传入的回调函数放进回调队列 callbacks
- 执行保存的异步任务 timeFunc，就会遍历 callbacks 执行相应的回调函数了
``` 
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  // 把回调函数放入回调队列
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    // 如果异步开关是开的，就关上，表示正在执行回调函数，然后执行回调函数
    pending = true
    timerFunc()
  }
  // 如果没有提供回调，并且支持 Promise，就返回一个 Promise
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```
可以看到最后有返回一个 Promise 是可以让我们在不传参的时候用的，如下
``` 
this.$nextTick().then(()=>{ ... })
```
## Vue3
### nextTick 用法
``` 
 <template>
     <div ref="test">{{name}}</div>
     <el-button @click="handleClick">按钮</el-button>
 </template>
 <script setup>
     import { ref, nextTick } from 'vue'
     const name = ref("沐华")
     const test = ref(null)
     async function handleClick(){
         name.value = '掘金'
         console.log(test.value.innerText) // 沐华
         await nextTick()
         console.log(test.value.innerText) // 掘金
     }
     return { name, test, handleClick }
 </script>
```
事件循环的原理还是一样，只是加了几个专门维护队列的方法，以及关联到 effect
### nextTick 源码剖析
``` 
const resolvedPromise: Promise<any> = Promise.resolve()
let currentFlushPromise: Promise<void> | null = null

export function nextTick<T = void>(this: T, fn?: (this: T) => void): Promise<void> {
  const p = currentFlushPromise || resolvedPromise
  return fn ? p.then(this ? fn.bind(this) : fn) : p
}
```
nextTick 接受一个函数为参数，同时会创建一个微任务  
页面调用 nextTick 的时候，会执行该函数，把我们的参数 fn 赋值给 p.then(fn)，在队列的任务完成后，fn 就执行了  
执行顺序是这样的：queueJob -> queueFlush -> flushJobs -> nextTick参数的 fn
入口函数 queueJob调用的地方
``` 
function baseCreateRenderer(){
  const setupRenderEffect: SetupRenderEffectFn = (...) => {
    const effect = new ReactiveEffect(
      componentUpdateFn,
      () => queueJob(instance.update), // 当作参数传入
      instance.scope
    )
  }
}
```
ReactiveEffect 这边接收过来的形参就是 scheduler，最终被用到了下面这里
``` 
export function triggerEffects(
  ...
  if (effect.scheduler) {
    effect.scheduler()
  } else {
    effect.run()
  }
}
```
### queueJob()
负责维护主任务队列，接受一个函数作为参数，为待入队任务，会将参数 push 到 queue 队列中，有唯一性判断。会在当前宏任务执行结束后，清空队列
``` 
const queue: SchedulerJob[] = []

export function queueJob(job: SchedulerJob) {
  // 主任务队列为空 或者 有正在执行的任务且没有在主任务队列中  && job 不能和当前正在执行任务及后面待执行任务相同
  if ((!queue.length ||
      !queue.includes( job, isFlushing && job.allowRecurse ? flushIndex + 1 : flushIndex )
      ) && job !== currentPreFlushParentJob
  ) {
    // 可以入队就添加到主任务队列
    if (job.id == null) {
      queue.push(job)
    } else {
      // 否则就删除
      queue.splice(findInsertionIndex(job.id), 0, job)
    }
    // 创建微任务
    queueFlush()
  }
}
```
### queueFlush()
负责尝试创建微任务，等待任务队列执行
``` 
let isFlushing = false // 是否正在执行
let isFlushPending = false // 是否正在等待执行
const resolvedPromise: Promise<any> = Promise.resolve() // 微任务创建器
let currentFlushPromise: Promise<void> | null = null // 当前任务

function queueFlush() {
  // 当前没有微任务
  if (!isFlushing && !isFlushPending) {
    // 避免在事件循环周期内多次创建新的微任务
    isFlushPending = true
    // 创建微任务，把 flushJobs 推入任务队列等待执行
    currentFlushPromise = resolvedPromise.then(flushJobs)
  }
}
```
### flushJobs()
负责处理队列任务，主要逻辑如下：
- 先处理前置任务队列
- 根据 Id 排队队列
- 遍历执行队列任务
- 执行完毕后清空并重置队列
- 执行后置队列任务
- 如果还有就递归继续执行

``` 
function flushJobs(seen?: CountMap) {
  isFlushPending = false // 是否正在等待执行
  isFlushing = true // 正在执行
  if (__DEV__) seen = seen || new Map() // 开发环境下
  flushPreFlushCbs(seen) // 执行前置任务队列
  // 根据 id 排序队列，以确保
  // 1. 从父到子，因为父级总是在子级前面先创建
  // 2. 如果父组件更新期间卸载了组件，就可以跳过
  queue.sort((a, b) => getId(a) - getId(b))
  try {
    // 遍历主任务队列，批量执行更新任务
    for (flushIndex = 0; flushIndex < queue.length; flushIndex++) {
      const job = queue[flushIndex]
      if (job && job.active !== false) {
        if (__DEV__ && checkRecursiveUpdates(seen!, job)) {
          continue
        }
        callWithErrorHandling(job, null, ErrorCodes.SCHEDULER)
      }
    }
  } finally {
    flushIndex = 0 // 队列任务执行完，重置队列索引
    queue.length = 0 // 清空队列
    flushPostFlushCbs(seen) // 执行后置队列任务
    isFlushing = false  // 重置队列执行状态
    currentFlushPromise = null // 重置当前微任务为 Null
    // 如果主任务队列、前置和后置任务队列还有没被清空，就继续递归执行
    if ( queue.length || pendingPreFlushCbs.length || pendingPostFlushCbs.length ) {
      flushJobs(seen)
    }
  }
}
```
### flushPreFlushCbs()
负责执行前置任务队列，
``` 
export function flushPreFlushCbs( seen?: CountMap, parentJob: SchedulerJob | null = null) {
  // 如果待处理的队列不为空
  if (pendingPreFlushCbs.length) {
    currentPreFlushParentJob = parentJob
    // 保存队列中去重后的任务为当前活动的队列
    activePreFlushCbs = [...new Set(pendingPreFlushCbs)]
    // 清空队列
    pendingPreFlushCbs.length = 0
    // 开发环境下
    if (__DEV__) { seen = seen || new Map() }
    // 遍历执行队列里的任务
    for ( preFlushIndex = 0; preFlushIndex < activePreFlushCbs.length; preFlushIndex+ ) {
      // 开发环境下
      if ( __DEV__ && checkRecursiveUpdates(seen!, activePreFlushCbs[preFlushIndex])) {
        continue
      }
      activePreFlushCbs[preFlushIndex]()
    }
    // 清空当前活动的任务队列
    activePreFlushCbs = null
    preFlushIndex = 0
    currentPreFlushParentJob = null
    // 递归执行，直到清空前置任务队列，再往下执行异步更新队列任务
    flushPreFlushCbs(seen, parentJob)
  }
}
```
### flushPostFlushCbs()
负责执行后置任务队列
``` 
let activePostFlushCbs: SchedulerJob[] | null = null

export function flushPostFlushCbs(seen?: CountMap) {
  // 如果待处理的队列不为空
  if (pendingPostFlushCbs.length) {
    // 保存队列中去重后的任务
    const deduped = [...new Set(pendingPostFlushCbs)]
    // 清空队列
    pendingPostFlushCbs.length = 0
    // 如果当前已经有活动的队列，就添加到执行队列的末尾，并返回
    if (activePostFlushCbs) {
      activePostFlushCbs.push(...deduped)
      return
    }
    // 赋值为当前活动队列
    activePostFlushCbs = deduped
    // 开发环境下
    if (__DEV__) seen = seen || new Map()
    // 排队队列
    activePostFlushCbs.sort((a, b) => getId(a) - getId(b))
    // 遍历执行队列里的任务
    for ( postFlushIndex = 0; postFlushIndex < activePostFlushCbs.length; postFlushIndex++ ) {
      if ( __DEV__ && checkRecursiveUpdates(seen!, activePostFlushCbs[postFlushIndex])) {
        continue
      }
      activePostFlushCbs[postFlushIndex]()
    }
    // 清空当前活动的任务队列
    activePostFlushCbs = null
    postFlushIndex = 0
  }
}
```

原文: 
[一次弄懂 Vue2 和 Vue3 的 nextTick 实现原理](https://juejin.cn/post/7021688091513454622)
