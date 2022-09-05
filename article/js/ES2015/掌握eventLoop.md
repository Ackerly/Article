# 掌握eventLoop
## 什么是事件循环
JavaScript执行函数的时候会有调用栈来处理函数的执行顺序，对于异步代码，通过任务队列进行处理，这个过程就是事件循环。任务队列分为macro-task（宏任务）与micro-task（微任务）
宏任务：
- script（整体代码）
- setTimeout
- setInterval
- I/O
- UI render

微任务：
- process.nextTick
- Promise
- Async/Await
- MutationObserver(html5新特性)  

执行宏任务，然后执行宏任务产生的微任务，若微任务的执行过程产生新的微任务，则继续执行微任务，微任务执行完毕后再回到宏任务执行下一轮循环

## async/await执行顺序
await 产生一个微任务，但是它使执行await之后直接跳出async,执行其他代码，其他代码执行完毕后，再回到async函数去执行剩下的代码，然后把await后面的代码注册到微任务队列当中。
```
console.log('script start')

async function async1() {
await async2()
console.log('async1 end')
}
async function async2() {
console.log('async2 end')
}
async1()

setTimeout(function() {
console.log('setTimeout')
}, 0)

new Promise(resolve => {
console.log('Promise')
resolve()
})
.then(function() {
console.log('promise1')
})
.then(function() {
console.log('promise2')
})

console.log('script end')

```
执行步骤：
1. 执行代码，输出script start
2. 执行async1，async2，然后输出async2 end，保留async1函数的上下文，跳出async1函数
3. 遇到setTimeout，产生宏任务
4. 执行Promise，输出Promise，遇到then，产生一个微任务
5. 继续执行代码，输出script end
6. 代码执行完毕，执行当前宏任务产生的文任务，输出promise1，微任务遇到then，产生一个新的微任务
7. 执行生产的为微任务，微任务执行完毕，执行权回到async1
8. 执行await，实际产生一个promise，即
```
let promise_ = new Promise((resolve,reject){ resolve(undefined)})
```
执行完成，执行await的语句，输出async1 end
最后执行下一个宏任务，即执行setTimeout，输出setTimeout

**注意：新版chrome优化了，await变得更快**

原文:  
[说说事件循环机制](https://juejin.cn/post/6844904079353708557)
