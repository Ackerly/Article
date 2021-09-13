# Vue3源码中再谈nextTick
定义：在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。  
使用：
``` 
import { createApp, nextTick } from 'vue'
const app = createApp({
  setup() {
    const message = ref('Hello!')
    const changeMessage = async newMessage => {
      message.value = newMessage
      // 这里获取DOM的value是旧值
      await nextTick()
      // nextTick 后获取DOM的value是更新后的值
      console.log('Now DOM is updated')
    }
  }
}) 
```
## JS执行机制
- 同步 在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务
- 异步 不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行

运行机制：
- 所有同步任务都在主线程上执行，形成一个执行栈
- 主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。
- 一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
- 主线程不断重复上面的第三步

## nextTick
实现原理：基于语言执行机制实现，直接创建一个异步任务，那么nextTick自然就达到在同步任务后执行的目的
``` 
const p = Promise.resolve()
export function nextTick(fn?: () => void): Promise<void> {
  return fn ? p.then(fn) : p
} 
```
## queueJob and queuePostFlushCb
queueJob 维护job列队，有去重逻辑，保证任务的唯一性，每次调用去执行 queueFlush queuePostFlushCb 维护cb列队，被调用的时候去重，每次调用去执行 queueFlush
``` 
const queue: (Job | null)[] = []
export function queueJob(job: Job) {
  // 去重 
  if (!queue.includes(job)) {
    queue.push(job)
    queueFlush()
  }
}

export function queuePostFlushCb(cb: Function | Function[]) {
  if (!isArray(cb)) {
    postFlushCbs.push(cb)
  } else {
    postFlushCbs.push(...cb)
  }
  queueFlush()
} 
```
## queueFlush
开启异步任务(nextTick)处理 flushJobs
``` 
function queueFlush() {
  // 避免重复调用flushJobs
  if (!isFlushing && !isFlushPending) {
    isFlushPending = true
    nextTick(flushJobs)
  }
} 
```
## flushJobs
处理列队，先对列队进行排序，执行queue中的job，处理完后再处理postFlushCbs, 如果队列没有被清空会递归调用flushJobs清空队列
``` 
function flushJobs(seen?: CountMap) {
  isFlushPending = false
  isFlushing = true
  let job
  if (__DEV__) {
    seen = seen || new Map()
  }

  // Sort queue before flush.
  // This ensures that:
  // 1. Components are updated from parent to child. (because parent is always
  //    created before the child so its render effect will have smaller
  //    priority number)
  // 2. If a component is unmounted during a parent component's update,
  //    its update can be skipped.
  // Jobs can never be null before flush starts, since they are only invalidated
  // during execution of another flushed job.
  queue.sort((a, b) => getId(a!) - getId(b!))

  while ((job = queue.shift()) !== undefined) {
    if (job === null) {
      continue
    }
    if (__DEV__) {
      checkRecursiveUpdates(seen!, job)
    }
    callWithErrorHandling(job, null, ErrorCodes.SCHEDULER)
  }
  flushPostFlushCbs(seen)
  isFlushing = false
  // some postFlushCb queued jobs!
  // keep flushing until it drains.
  if (queue.length || postFlushCbs.length) {
    flushJobs(seen)
  }
} 
```
 queueJob 及 queuePostFlushCb 是怎么被调用的
``` 
//  renderer.ts
function createDevEffectOptions( instance: ComponentInternalInstance ): ReactiveEffectOptions {
  return {
    scheduler: queueJob,
    onTrack: instance.rtc ? e => invokeArrayFns(instance.rtc!, e) : void 0,
    onTrigger: instance.rtg ? e => invokeArrayFns(instance.rtg!, e) : void 0
  }
}

// effect.ts
const run = (effect: ReactiveEffect) => {
  ...

  if (effect.options.scheduler) {
    effect.options.scheduler(effect)
  } else {
    effect()
  }
} 
```
当响应式对象发生改变后，执行 effect 如果有 scheduler 这个参数，会执行这个 scheduler 函数，并且把 effect 当做参数传入
## 为什么使用nextTick
``` 
{{num}}
for(let i=0; i<100000; i++){
    num = i
} 
```
如果没有 nextTick 更新机制，那么 num 每次更新值都会触发视图更新，有了nextTick机制，只需要更新一次，所以为什么有nextTick存在，


参考:  
[从 Vue3 源码中再谈 nextTick](https://segmentfault.com/a/1190000038921474?content_source_url=https://github.com/vue3/vue3-News)
