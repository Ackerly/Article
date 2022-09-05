# 如何实现一个比较完美的reduce函数
## 实现基础版本  
reduce函数接受一个运行函数和一个初始的默认值,reduce函数的重点就是要将结果再次传入执行函数中进行处理  
``` 
function myReduce(data, iteratee, memo) {
   for(let i = 0; i < data.length; i++) {
    memo = iteratee(memo, data[i], i, data)
   }
   return memo
}
```
**增加this绑定**  
reduce函数可以指定自定义对象绑定this,可以使用call对函数进行重新绑定  
``` 
function myReduce(data, iteratee, memo, context) {
   // 重置iteratee函数的this指向
   iteratee = bind(iteratee, context)
  
   for(let i = 0; i < data.length; i++) {
    memo = iteratee(memo, data[i], i, data)
   }
   return memo
  }
  
  // 绑定函数 使用call进行绑定
  function bind(fn, context) {
    // 返回一个匿名函数，执行时重置this指向
    return function(memo, value, index, collection) {
      return fn.call(context, memo, value, index, collection);
    };
  }
```
**增加对第二个参数默认值的支持**  
reduce函数的第三个参数也是可选值，如果没有传递第三个参数，那么直接使用传入数据的第一个位置初始化  
``` 
function myReduce(data, iteratee, memo, context) {
   // 重置iteratee函数的this指向
   iteratee = bind(iteratee, context)
   // 判断是否传递了第三个参数
   let initial  = arguments.length >= 3; // 新增
   // 初始的遍历下标
   let index = 0 // 新增
  
   if(!initial) { // 新增
    // 如果用户没有传入默认值，那么就取数据的第一项作为默认值
    memo = data[index] // 新增
    // 所以遍历就要从第二项开始
    index += 1 // 新增
   }
  
   for(let i = index; i < data.length; i++) { // 修改
    memo = iteratee(memo, data[i], i, data)
   }
   return memo
  }
  
  // 绑定函数 使用call进行绑定
  function bind(fn, context) {
    return function(memo, value, index, collection) {
      return fn.call(context, memo, value, index, collection);
    };
  }
```
**支持对象**  
js原生的reduce函数是不支持对象这种数据结构的，那么如何完善我们的reduce函数呢？  
其实只需要取出对象中所有的key，然后遍历key就可以了  
``` 
function myReduce(data, iteratee, memo, context) {
 
   iteratee = bind(iteratee, context)
   // 取出所有的key值
   let _keys = !Array.isArray(data) && Object.keys(data) // 新增
   // 长度赋值
   let len = (_keys || data).length // 新增
 
   let initial  = arguments.length >= 3;
   let index = 0
  
   if(!initial) {
    // 如果没有设置默认值初始值，那么取第一个值的操作也要区分对象/数组
    memo = data[ _keys ? _keys[index] : index] // 修改
    index += 1
   }
  
   for(let i = index; i < len; i++) {
    // 取key值
    let currentKey = _keys ? _keys[i] : i // 新增
    memo = iteratee(memo, data[currentKey], currentKey, data) // 修改
   }
   return memo
  }
  
  function bind(fn, context) {
    return function(memo, value, index, collection) {
      return fn.call(context, memo, value, index, collection);
    };
  }
```
**reduceRight**  
reduceRight函数的功能也很简单，就是在遍历的时候倒序进行操作，例如：  
``` 
et total = [ 0, 1, 2, 3 ].reduce(
    function (pre, cur) {
      pre.push(cur + 1)
      return pre
    },
    []
  )
  
  // [1, 2, 3, 4]
```
只需要初始操作的值更改为最后一个元素的位置就可以了：  
``` 
// 加入一个参数dir，用于标识正序/倒序遍历
function myReduce(data, iteratee, memo, context, dir) { //修改

  iteratee = bind(iteratee, context)
  let _keys = !Array.isArray(data) && Object.keys(data)
  let len = (_keys || data).length

  let initial  = arguments.length >= 3;

  // 定义下标
  let index = dir > 0 ? 0 : len - 1 // 修改
 
  if(!initial) {
   memo = data[ _keys ? _keys[index] : index]
   // 定义初始值
   index += dir // 修改
  }
  // 每次修改只需步进指定的值
  for(;index >= 0 && index < len; index += dir) { // 修改
   let currentKey = _keys ? _keys[index] : index
   memo = iteratee(memo, data[currentKey], currentKey, data)
  }
  return memo
 }
 
 function bind(fn, context) {
   if (!context) return fn;
   return function(memo, value, index, collection) {
     return fn.call(context, memo, value, index, collection);
   };
 }
```
调用的时候直接传入最后一个参数为1 / \-1即可  
``` 
myReduce([1, 2, 3, 4], function(pre, cur) {
     console.log(cur)
 }, [], 1)
 
 myReduce([1, 2, 3, 4], function(pre, cur) {
     console.log(cur)
 }, [], -1)
```
最后将整个函数进行重构抽离成为一个单独的函数：  
``` 
function createReduce(dir) {
     function reduce() {
   // ....
     }
 
     return function() {
   return reduce()
     }
 }
```
最终的代码如下：  
``` 
function createReduce(dir) {
 
     function reduce(data, fn, memo, initial) {
         let _keys = Array.isArray(data) && Object.keys(data),
             len = (_keys || data).length,
             index = dir > 0 ? 0 : len - 1;

         if (!initial) {
             memo = data[_keys ? _keys[index] : index]
             index += dir
         }

         for (; index >= 0 && index < len; index += dir) {
             let currentKey = _keys ? _keys[index] : index
             memo = fn(memo, data[currentKey], currentKey, data)
         }
         return memo
     }


     return function (data, fn, memo, context) {
         let initial = arguments.length >= 3
         return reduce(data, bind(fn, context), memo, initial)
     }
 }

 function bind(fn, context) {
     if (!context) return fn;
     return function (memo, value, index, collection) {
         return fn.call(context, memo, value, index, collection);
     };
 }

 let reduce = createReduce(1)
 let reduceRight = createReduce(-1)
```

原文: 
[面试官直呼内行！如何实现一个比较完美的reduce函数？](https://mp.weixin.qq.com/s/8ozhv8F9CHjJZSktVLfUxw)
