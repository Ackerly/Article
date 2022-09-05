# nextTick的使用
## 用法
在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。
``` 
// 修改数据
vm.msg = 'Hello'
// DOM 还没有更新
Vue.nextTick(function () {
  // DOM 更新了
})

// 作为一个 Promise 使用 (2.1.0 起新增，详见接下来的提示)
Vue.nextTick()
  .then(function () {
    // DOM 更新了
  })
```
## vm.$nextTick和Vue.nextTick的区别
Vue.nextTick内部函数的this指向window
``` 
Vue.nextTick(function () {
    console.log(this); // window
})
```
vm.$nextTick内部函数的this指向Vue实例对象
``` 
vm.$nextTick(function () {
    console.log(this); // vm实例
})
```
## nextTick的实现
``` 
export const nextTick = (function () {
  // callbacks存放所有的回调函数 也就是dom更新之后我们希望执行的回调函数
  var callbacks = []
  // pending可以理解为上锁 也可以理解为挂起 这里的意思是不上锁
  var pending = false
  // 使用哪种异步函数去执行：MutationObserver，setImmediate还是setTimeout
  var timerFunc
  // 会执行所有的回调函数 
  function nextTickHandler () {
    pending = false
    // 之所以要slice复制一份出来是因为有的cb执行过程中又会往callbacks中加入内容
    // 比如$nextTick的回调函数里又有$nextTick
    // 这些是应该放入到下一个轮次的nextTick去执行的，
    // 所以拷贝一份当前的，遍历执行完当前的即可，避免无休止的执行下去
    var copies = callbacks.slice(0)
    // 清空回调函数 因为全部都拿出来执行了
    callbacks = []
    // 执行所有的回调函数
    for (var i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }
    
  // ios9.3以上的WebView的MutationObserver有bug，
  // 所以在hasMutationObserverBug中存放了是否是这种情况
  if (typeof MutationObserver !== 'undefined' && !hasMutationObserverBug) {
    // 随便声明一个变量作为文本的节点
    var counter = 1
    var textNode = document.createTextNode(counter)
    // 创建一个MutationObserver，observer监听到dom改动之后后执行回调nextTickHandler
    var observer = new MutationObserver(nextTickHandler)
    // 调用MutationObserver的接口，监测文本节点的字符改变
    observer.observe(textNode, {
      characterData: true
    })
    // 每次一执行timerFunc，就变化文本节点的字符，这样就会被observer监听到，然后执行nextTickHandler，nextTickHandler就会执行callback中的回调函数。
    timerFunc = function () {
      // 都会让文本节点的内容在0/1之间切换
      counter = (counter + 1) % 2
      // 切换之后将新值赋值到那个我们用observer观测的文本节点上去
      textNode.data = counter
    }
  } else {
    // webpack默认会在代码中插入setImmediate的垫片
    // 没有MutationObserver就优先用setImmediate，不行再用setTimeout
    // setImmediate是一个宏任务，但是他执行的速度比setTimeout快一点，只在IE下有，主要为了兼容IE。
    const context = inBrowser
      ? window
      : typeof global !== 'undefined' ? global : {}
    timerFunc = context.setImmediate || setTimeout
  }
  //这里返回的才是nextTick的内容
  return function (cb, ctx) {
    //有没有传入第二个参数
    var func = ctx
      //有的话就改变回调函数的this为第二个参数
      ? function () { cb.call(ctx) }
      //没有的话就直接把回调函数赋值给func
      : cb
    //把回调函数放到callbacks里面，等待dom更新之后执行
    callbacks.push(func)
    // 如果pending为true，就表明本轮事件循环中已经执行过timerFunc了
    if (pending) return
    // 上锁
    pending = true
    // 执行异步函数，在异步函数中执行所有的回调
    timerFunc(nextTickHandler, 0)
  }
})()
```
第一次在代码中调用nextTick把函数推入callbacks，然后调用异步函数nextTickHandle的过程，设置pending = true。所以寿面再调用nextTick都是执行到callbacks.push(func),等执行完再执行callbacks列表。
**nextTick的意义**  
同步任务中多次改变DOM。那么在所有同步任务执行完毕之后，就说明数据修改已经结束了，改变DOM的函数我都执行过了，已经得到了最终要渲染的DOM数据，所以这个时候可放心更新DOM了。nextTick的回调函数都是在microtask中执行的。这样就可以尽量避免重复的修改渲染某个DOM元素，另一方面也能够将DOM操作聚集，减少渲染的次数，提升DOM渲染效率。

原文:  
[深入了解vm.$nextTick和Vue.nextTick](https://juejin.cn/post/6844903973061656590)
