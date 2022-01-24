# 学习lodash源码整体架构
**匿名函数执行**  
``` 
;(function() {

}.call(this));
```
暴露 lodash
``` 
var _ = runInContext();
```
**runInContext 函数**  
``` 
var runInContext = (function runInContext(context) {
 // 浏览器中处理context为window
 // ...
 function lodash(value) {}{
  // ...
  return new LodashWrapper(value);
 }
 // ...
 return lodash;
});
```
申明了一个runInContext函数。里面有一个lodash函数，最后处理返回这个lodash函数，的返回值 new LodashWrapper(value)。  
**LodashWrapper 函数**  
``` 
function LodashWrapper(value, chainAll) {
 this.__wrapped__ = value;
 this.__actions__ = [];
 this.__chain__ = !!chainAll;
 this.__index__ = 0;
 this.__values__ = undefined;
}
```
__wrapped__：存放参数value。  
__actions__：存放待执行的函数体func， 函数参数 args，函数执行的this 指向 thisArg。  
__chain__、undefined两次取反转成布尔值false，不支持链式调用。和underscore一样，默认是不支持链式调用的。  
__index__：索引值 默认 0。  
__values__：主要clone时使用。  

往下搜索源码，LodashWrapper， 会发现这两行代码  
``` 
LodashWrapper.prototype = baseCreate(baseLodash.prototype);
LodashWrapper.prototype.constructor = LodashWrapper;
```
往上找baseCreate、baseLodash
**baseCreate 原型继承**  
``` 
//  立即执行匿名函数
// 返回一个函数，用于设置原型 可以理解为是 __proto__
var baseCreate = (function() {
 // 这句放在函数外，是为了不用每次调用baseCreate都重复申明 object
 // underscore 源码中，把这句放在开头就申明了一个空函数 `Ctor`
 function object() {}
 return function(proto) {
  // 如果传入的参数不是object也不是function 是null
  // 则返回空对象。
  if (!isObject(proto)) {
   return {};
  }
  // 如果支持Object.create方法，则返回 Object.create
  if (objectCreate) {
   // Object.create
   return objectCreate(proto);
  }
  // 如果不支持Object.create 用 ployfill new
  object.prototype = proto;
  var result = new object;
  // 还原 prototype
  object.prototype = undefined;
  return result;
 };
}());

// 空函数
function baseLodash() {
 // No operation performed.
}

// Ensure wrappers are instances of `baseLodash`.
lodash.prototype = baseLodash.prototype;
// 为什么会有这一句？因为上一句把lodash.prototype.construtor 设置为Object了。这一句修正constructor
lodash.prototype.constructor = lodash;

LodashWrapper.prototype = baseCreate(baseLodash.prototype);
LodashWrapper.prototype.constructor = LodashWrapper;
```
** isObject 函数**  
判断typeof value不等于null，并且是object或者function
``` 
function isObject(value) {
 var type = typeof value;
 return value != null && (type == 'object' || type == 'function');
}
```
## mixin
mixin 具体用法
``` 
_.mixin([object=lodash], source, [options={}])
```
> 添加来源对象自身的所有可枚举函数属性到目标对象。 如果 object 是个函数，那么函数方法将被添加到原型链上。

> 注意: 使用 _.runInContext 来创建原始的 lodash 函数来避免修改造成的冲突。

**参数**  
> [object=lodash] (Function|Object): 目标对象。

> source (Object): 来源对象。

> [options={}] (Object): 选项对象。

> [options.chain=true] (boolean): 是否开启链式操作。

**返回**  
``` 
(*): 返回 object.
```

**mixin 衍生的函数 keys**  
在 mixin 函数中 其实最终调用的就是 Object.keys  
``` 
function keys(object) {
 return isArrayLike(object) ? arrayLikeKeys(object) : baseKeys(object);
}
```
**mixin 衍生的函数 baseFunctions**  
返回函数数组集合  
``` 
function baseFunctions(object, props) {
 return arrayFilter(props, function(key) {
  return isFunction(object[key]);
 });
}
```
**mixin 衍生的函数 isFunction**  
判断参数是否是函数  
``` 
function isFunction(value) {
 if (!isObject(value)) {
  return false;
 }
 // The use of `Object#toString` avoids issues with the `typeof` operator
 // in Safari 9 which returns 'object' for typed arrays and other constructors.
 var tag = baseGetTag(value);
 return tag == funcTag || tag == genTag || tag == asyncTag || tag == proxyTag;
}
```
**mixin 衍生的函数 arrayEach**  
类似 [].forEarch  
``` 
function arrayEach(array, iteratee) {
 var index = -1,
  length = array == null ? 0 : array.length;

 while (++index < length) {
  if (iteratee(array[index], index, array) === false) {
   break;
  }
 }
 return array;
}
```
**mixin 衍生的函数 arrayPush**  
类似 [].push  
``` 
function arrayPush(array, values) {
 var index = -1,
  length = values.length,
  offset = array.length;

 while (++index < length) {
 array[offset + index] = values[index];
 }
 return array;
}
```
**mixin 衍生的函数 copyArray**  
拷贝数组  
``` 
function copyArray(source, array) {
 var index = -1,
  length = source.length;

 array || (array = Array(length));
 while (++index < length) {
  array[index] = source[index];
 }
 return array;
}
```
**mixin 源码解析**  
lodash 源码中两次调用 mixin
``` 
// Add methods that return wrapped values in chain sequences.
lodash.after = after;
// code ... 等 153 个支持链式调用的方法

// Add methods to `lodash.prototype`.
// 把lodash上的静态方法赋值到 lodash.prototype 上
mixin(lodash, lodash);

// Add methods that return unwrapped values in chain sequences.
lodash.add = add;
// code ... 等 152 个不支持链式调用的方法


// 这里其实就是过滤 after 等支持链式调用的方法，获取到 lodash 上的 add 等 添加到lodash.prototype 上。
mixin(lodash, (function() {
 var source = {};
 // baseForOwn 这里其实就是遍历lodash上的静态方法，执行回调函数
 baseForOwn(lodash, function(func, methodName) {
  // 第一次 mixin 调用了所以赋值到了lodash.prototype
  // 所以这里用 Object.hasOwnProperty 排除不在lodash.prototype 上的方法。也就是 add 等 152 个不支持链式调用的方法。
  if (!hasOwnProperty.call(lodash.prototype, methodName)) {
   source[methodName] = func;
  }
 });
 return source;
// 最后一个参数options 特意注明不支持链式调用
}()), { 'chain': false });
```
把lodash上的静态方法赋值到lodash.prototype上。分两次第一次是支持链式调用（lodash.after等 153个支持链式调用的方法），第二次是不支持链式调用的方法（lodash.add等152个不支持链式调用的方法）。  

``` 
var result = _.chain([1, 2, 3, 4, 5])
.map(el => {
 console.log(el); // 1, 2, 3
 return el + 1;
})
.take(3)
.value();
// lodash中这里的`map`仅执行了`3`次。
// 具体功能也很简单 数组 1-5 加一，最后获取其中三个值。
console.log('result:', result);
```
这里lodash聪明的知道了最后需要几个值，就执行几次map循环，对于很大的数组，提升性能很有帮助。  
这里的map方法，添加 LazyWrapper 的方法到 lodash.prototype存储下来，最后调用 value时再调用  
**添加 LazyWrapper 的方法到 lodash.prototype**  
主要是如下方法添加到到 lodash.prototype 原型上
``` 
// "constructor"
["drop", "dropRight", "take", "takeRight", "filter", "map", "takeWhile", "head", "last", "initial", "tail", "compact", "find", "findLast", "invokeMap", "reject", "slice", "takeRightWhile", "toArray", "clone", "reverse", "value"]
```
链式调用最后都是返回实例对象，实际的处理数据的函数都没有调用，而是被存储存储下来了，最后调用value方法，才执行这些函数。  
lodash.prototype.value 即 wrapperValue  
``` 
function baseWrapperValue(value, actions) {
 var result = value;
 // 如果是lazyWrapper的实例，则调用LazyWrapper.prototype.value 方法，也就是 lazyValue 方法
 if (result instanceof LazyWrapper) {
  result = result.value();
 }
 // 类似 [].reduce()，把上一个函数返回结果作为参数传递给下一个函数
 return arrayReduce(actions, function(result, action) {
  return action.func.apply(action.thisArg, arrayPush([result], action.args));
 }, result);
}
function wrapperValue() {
 return baseWrapperValue(this.__wrapped__, this.__actions__);
}
lodash.prototype.toJSON = lodash.prototype.valueOf = lodash.prototype.value = wrapperValue;
```
如果是惰性求值，则调用的是 LazyWrapper.prototype.value 即 lazyValue。  


参考:  
[学习 lodash 源码整体架构，打造属于自己的函数式编程类库](https://juejin.cn/post/6844903939062628360)
