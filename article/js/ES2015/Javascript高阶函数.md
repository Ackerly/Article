# Javascript高阶函数
## AOP
**什么是AOP**
把和核心业务逻辑模块无关的功能抽取出来，再通过“动态织入”的方式掺到业务模块种。一般包括日志统计，安全控制，异常处理等。
```
/**
* 织入执行前函数
* @param {*} fn 
*/
Function.prototype.aopBefore = function(fn){
  console.log(this)
  // 第一步：保存原函数的引用
  const _this = this
  // 第四步：返回包括原函数和新函数的“代理”函数
  return function() {
    // 第二步：执行新函数，修正this
    fn.apply(this, arguments)
    // 第三步 执行原函数
    return _this.apply(this, arguments)
  }
}
/**
* 织入执行后函数
* @param {*} fn 
*/
Function.prototype.aopAfter = function (fn) {
  const _this = this
  return function () {
    let current = _this.apply(this,arguments)// 先保存原函数
    fn.apply(this, arguments) // 先执行新函数
    return current
  }
}
/**
* 使用函数
*/
let aopFunc = function() {
  console.log('aop')
}
// 注册切面
aopFunc = aopFunc.aopBefore(() => {
  console.log('aop before')
}).aopAfter(() => {
  console.log('aop after')
})
// 真正调用
aopFunc()
```

## 柯里化
**什么是柯里化**
接受一些参数，但是不会立即求值，而是返回另一个函数，参数在函数的形成的闭包保存起来，待到真正求值的时候，之前传入的参数一次性求值。
柯里化前：
```
// 未柯里化的函数计算开销
let totalCost = 0
const cost = function(amount, mounth = '') {
 console.log(`第${mounth}月的花销是${amount}`)
 totalCost += amount
 console.log(`当前总共消费：${totalCost}`)
}
cost(1000, 1) // 第1个月的花销
cost(2000, 2) // 第2个月的花销
// ...
cost(3000, 12) // 第12个月的花销
```
柯里化后
```
// 通用curring函数
const curring = function(fn) {
 let args = []
 return function () {
   if (arguments.length === 0) {
     console.log('curring完毕进行计算总值')
     return fn.apply(this, args)
   } else {
     let currentArgs = Array.from(arguments)[0]
     console.log(`暂存${arguments[1] ? arguments[1] : '' }月，金额${arguments[0]}`)
     args.push(currentArgs)
     // 返回正被执行的 Function 对象，也就是所指定的 Function 对象的正文，这有利于匿名函数的递归或者保证函数的封装性
     return arguments.callee
   }
 }
}
// 求值函数
let costCurring = (function() {
 let totalCost = 0
 return function () {
   for (let i = 0; i < arguments.length; i++) {
     totalCost += arguments[i]
   }
   console.log(`共消费：${totalCost}`)
   return totalCost
 }
})()
// 执行curring化
costCurring = curring(costCurring)
costCurring(2000, 1)
costCurring(2000, 2)
costCurring(9000, 12)
costCurring()
```
## 函数节流
**通过函数节流，限制函数触发频率**
```
const throttle = function (fn, interval = 500) {
 let timer = null, // 计时器 
     isFirst = true // 是否是第一次调用
 return function () {
   let args = arguments, _me = this
   // 首次调用直接放行
   if (isFirst) {
     fn.apply(_me, args)
     return isFirst = false
   }
   // 存在计时器就拦截
   if (timer) {
     return false
   }
   // 设置timer
   timer = setTimeout(function (){
    console.log(timer)
    window.clearTimeout(timer)
    timer = null
    fn.apply(_me, args)
   }, interval)
 }
}
```
## 分时函数
短时间内页面插入大量的DOM节点，导致浏览器卡顿，通过使用分时函数，分批插入
```
const timeChunk = function(list, fn, count = 1){
 let insertList = [], // 需要临时插入的数据
     timer = null // 计时器
 const start = function(){
   // 对执行函数逐个进行调用
   for (let i = 0; i < Math.min(count, list.length); i++) {
     insertList = list.shift()
     fn(insertList)
   }
 }
 return function(){
   timer = setInterval(() => {
     if (list.length === 0) {
       return window.clearInterval(timer)
     }
     start()
   },200)
 }
}
```

## 惰性加载函数
**处理各个浏览器的差异性，比如通用事件绑定函数**
```
let addEventLazy = function(el, type, handler) {
 if (window.addEventListener) {
   // 一旦进入分支，便在函数内部修改函数的实现
   addEventLazy = function(el, type, handler) {
     el.addEventListener(type, handler, false)
   }
 } else if (window.attachEvent) {
   addEventLazy = function(el, type, handler) {
     el.attachEvent(`on${type}`, handler)
   }
 }
 addEventLazy(el, type, handler)
}
```
原文:[JavaScript中高阶函数的魅力](https://juejin.cn/post/6844903668651819016)