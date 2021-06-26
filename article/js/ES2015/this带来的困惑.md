# This带来的困惑
## 概要
初学者可能突然将this弄丢导致程序出错，甚至react中也要使用bind的方式，使回调可以访问到setState等函数。  
class带来的困惑主要在于this，因为成员函数会挂到prototype，虽然多个实例共享了引用，但因此带来的隐患就是this的不确定性。js有许多this丢失情况，比如隐式绑定、别名丢失隐式绑定、回调丢失隐式绑定、显示绑定、new 绑定，箭头函数改变this作用范围等等。  
由于在prototype中对象依赖this，如果this丢失，就访问不到原型链，会引发报错。

## this丢失的情况
### 默认绑定
在严格模式和非严格模式下，默认绑定有所区别，非严格模式this会绑定到上级作用域，而use strict时不会绑定到window。
``` 
function foo(){
  console.log(this.count) // 1
  console.log(foo.count) // 2
}
var count = 1
foo.count = 2
foo()
```
```
function foo(){
  "use strict"
  console.log(this.count) // TypeError: count undefined
}
var count = 1
foo()
```
### 隐式绑定
函数被对象引用起来调用时，this会绑定到期依附的对象上
``` 
function foo(){
  console.log(this.count) // 2
}
var obj = {
  count: 2,
  foo: foo
}
obj.foo()
```
### 别名丢失隐式绑定
调用函数引用时，this会根据调用者环境而定
``` 
function foo(){
  console.log(this.count) // 1
}
var count = 1
var obj = {
  count: 2,
  foo: foo
}
var bar = obj.foo // 函数别名
bar()
```
### 回调丢失隐式绑定
类似react默认的情况，将函数传递给子组件，其调用时，this会丢失。
``` 
function foo(){
  console.log(this.count) // 1
}
var count = 1
var obj = {
  count: 2,
  foo: foo
}
setTimeout(obj.foo)
```
## this绑定修复
### bind显示绑定
``` 
function foo(){
  console.log(this.count) // 1
}
var obj = {
  count: 1
}
foo.call(obj)

var bar = foo.bind(obj)
bar()
```
### 使用call绑定
``` 
function foo(){
  setTimeout(() => {
    console.log(this.count) // 2
  })
}
var obj = {
  count: 2
}
foo.call(obj)
```


参考:  
[精读《This 带来的困惑》](https://github.com/ascoders/weekly/blob/master/%E5%89%8D%E6%B2%BF%E6%8A%80%E6%9C%AF/13.%E7%B2%BE%E8%AF%BB%E3%80%8AThis%20%E5%B8%A6%E6%9D%A5%E7%9A%84%E5%9B%B0%E6%83%91%E3%80%8B.md)
