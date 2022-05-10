# JavaScript 基础题
## 第1题 — 原型
``` 
function Animal(){ 
  this.type = "animal"
}
   
function Dog(){ 
  this.name = "dog"
}
 
Dog.prototype = new Animal()
 
var PavlovPet = new Dog(); 
 
console.log(PavlovPet.__proto__ === Dog.prototype)
console.log(Dog.prototype.__proto__ === Animal.prototype)
```
console.log 打印出的值是 true, true
>所有的对象都有 [[prototype]] 属性（通过 _proto_ 访问），该属性对应对象的原型；所有的函数对象都有 prototype 属性，该属性的值会被赋值给该函数创建的对象的 _proto_ 属性。

## 第2题 — 小心“排序” *
``` 
var arr = [5, 22, 14, 9];

console.log(arr.sort());
```
console.log 打印出的值[14, 22, 5, 9]
不传入排序函数，sort 函数会将每个元素转换成字符串，然后根据它们的 UTF-16 值排序。
## 第3题 — 异步循环
``` 
for (let i = 0; i < 3; i++) {
  const log = () => {
    console.log(i)
  }
  setTimeout(log, 100)
}
```
console.log 打印出的值0,1,2
## 第4题 — numbers里面有啥
``` 
const length = 4
const numbers = []
for (var i = 0; i < length; i++);{
  numbers.push(i + 1)
}
 
console.log(numbers)
```
console.log 打印出的值是5
注意:小括号和大括号之间有个；
## 第5题 — 长度为0
``` 
const clothes = ['shirt', 'socks', 'jacket', 'pants', 'hat']
clothes.length = 0
```

console.log 打印出的值是undefined
注意：长度赋值为 0 就相当于从数组中删除所有元素。
## 第6题 — 变量定义
``` 
var a = 1
function output () {
    console.log(a)
    var a = 2
    console.log(a)
}
console.log(a)
output()
console.log(a)
```
console.log 打印出的值是1, undefined, 2,1
## 第7题 — 找到值了吗
``` 
function foo() {
    let a = b = 0
    a++
    return a
}
 
foo()
console.log(typeof a)
console.log(typeof b)
```
console.log 打印出的值是undefined, b
## 第8题 — 类型转换
``` 
console.log(+true)
console.log(!"ConardLi")
```
console.log 打印出的值是1,true
## 第9题 — ESM
``` 
// module.js 
export default () => "Hello world"
export const name = "c"

// index.js 
import * as data from "./module"

console.log(data)
```
console.log打印出的值是() => "Hello world"和name = "c"
## 第10题 — 对象做 key
``` 
const a = {};
const b = { key: "b" };
const c = { key: "c" };

a[b] = 123;
a[c] = 456;

console.log(a[b]);
```
console.log 打印出的值是123

参考:
[进来做几道 JavaScript 基础题找找自信](https://mp.weixin.qq.com/s/h-oZm_4o_2RcEy99UsdYKg)
