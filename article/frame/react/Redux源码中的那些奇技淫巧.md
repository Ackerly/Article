# Redux源码中的那些奇技淫巧
## 随机字符串生成器
redux中内置了一些action类型,有时候在内部实现时需要生成唯一的action类型  
redux中的内置action类型  
``` 
// dispatch的用法
dispatch({ type: ActionTypes.REPLACE })
// 三种内置actionType:  init replace UNKNOWN_ACTION
const ActionTypes = {
  INIT: `@@redux/INIT${/* #__PURE__ */ randomString()}`,
  REPLACE: `@@redux/REPLACE${/* #__PURE__ */ randomString()}`,
  PROBE_UNKNOWN_ACTION: () => `@@redux/PROBE_UNKNOWN_ACTION${randomString()}`
}
```
其中,为了保证actionType的唯一性,使用了拼接randomString的方式  
``` 
// 使用randomString函数创建随机string,进行拼接
const randomString = () =>
Math.random().toString(36).substring(7).split('').join('.')
```
使用Math.random, 在进行各种字符串处理,生成一个随机字符串  
## 自定义KindOf函数获取变量类型
此函数用于获取一个变量的类型名 比如"string","Array","Promise"等,在源码里用于控制台打印。  
一般来说,我们单纯用JS自带的typeof能判断的类型极为有限, 有时候我们还需要判断Array,甚至是Promise,WeakMap等特殊结构体。  
``` 
// 用于获取类型的函数
function kindOf(val: any): string {
  // 通过void 0获取undefined
  if (val === void 0) return 'undefined'
  if (val === null) return 'null'
  // 通过typeof判断一些类型 
  const type = typeof val
  switch (type) {
    case 'boolean':
    case 'string':
    case 'number':
    case 'symbol':
    case 'function': {
      return type
    }
  }
  // 通过Array.isArray判断数组
  // 封装的isDate函数用于判断日期 (内部使用instanceof Date)
  // 封装的isError函数用于判断Error (内部使用instanceof Error)
  if (Array.isArray(val)) return 'array'
  if (isDate(val)) return 'date'
  if (isError(val)) return 'error'
	
  // 通过val.constructor.name获取特殊对象类型
  const constructorName = ctorName(val)
  switch (constructorName) {
    case 'Symbol':
    case 'Promise':
    case 'WeakMap':
    case 'WeakSet':
    case 'Map':
    case 'Set':
      return constructorName
  }
	
  // 其他通过Object.prototype.toString+正则获取类型
  return Object.prototype.toString
    .call(val)
    .slice(8, -1)
    .toLowerCase()
    .replace(/\s/g, '')
}
```
## 通过isDispatching标记给函数上锁
使用dispatch去触发同步reducer时, 会同步改变state, 称其为mutation形式的变更。 而同步执行的函数在单线程的js中是不会有交集的。  
但是由于reducer是用户写的, redux内部为了保险, 依然给reducer的执行上了锁  
通过isDispatching这个标签,确保同时只能有一个reducer执行  
``` 
// 外层
let isDispatching = false

function dispatch(action: A) {
   // 判断isDispatching
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }
    // 执行reducer 改变状态
    try {
      isDispatching = true  // 改变isDispatching
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false // 改变isDispatching
    }
}
```
原因: 是为了不让用户在reducer方法中执行其他可能会破环正常数据流程的方法，比如在reducer中再次dispatch，会导致死循环，这也时Redux为了保护而进行限制的一种体现。  
## 判断是否是简单对象 isPlainObject
redux需要保证我们传入的action对象是一个简单对象,所以使用此函数来进行判断  
主要逻辑就是通过检查对象最底层的prototype  
``` 
export default function isPlainObject(obj: any): boolean {
  // 过滤掉非object情况
  if (typeof obj !== 'object' || obj === null) return false
  // 通过while循环深入查找到最底层的原型链
  let proto = obj
  while (Object.getPrototypeOf(proto) !== null) {
    proto = Object.getPrototypeOf(proto)
  }
  // 如果最底层的prototype等于当前object的prototype 则是简单对象
  return Object.getPrototypeOf(obj) === proto
}
```
这也是比较常用的对原型链的运用,如果需要过滤掉一些包装对象,或者是复杂代理对象,可以使用此函数
## compose将多个函数转换为嵌套结构
Redux可以通过applyMiddleware给dispatch的过程传入多个中间件,而compose函数用于将这些中间件函数转化为嵌套结构,更符合中间件的执行方式  
``` 
// 例子 compose(A, B, C)(args)转换为A(B(C(arg)))的形式
compose(fn1,fn2,fn3...)
// 转化为
fn1(fn2(fn3(args)))
// 巧用Array.reduce转化多个平级函数为嵌套结构
compose(...funcs: Function[]) {
  return funcs.reduce(
    (a, b) =>
      (...args: any) =>
        a(b(...args))
  )
}
```

原文:  
[Redux源码中的那些奇技淫巧](https://juejin.cn/post/7191023152296624183)
