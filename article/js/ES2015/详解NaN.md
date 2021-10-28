# 详解NaN
## NaN和Number.NaN
**特点1 typeof是数字**  
``` 
typeof NaN // number
typeof Number.NaN // number
```
**特点2 我不等于我自己**  
``` 
NaN == NaN  // false
Number.NaN == NaN // false
NaN === NaN // false
Number.NaN === NaN  // false

+0 == -0 // true
Object.is(+0, -0) // fasle
```
**NaN的描述信息**  
是一个值，新的ES标准中， 不可配置，不可枚举。 也就是说不可以被删除delete，不可以被改写， 也不可以改写配置。
``` 
delete NaN // false
NaN = 1 // 1
NaN == 1 // false

delete Number.NaN // false
Number.NaN = 1 // 1
Number.NaN == 1 // false
```
尝试改写:  
使用Reflect.defineProperty 而不使用Object.defineProperty,因为前者能准确告诉你是否成功，而后者返回的被定义的对象。
``` 
const success =Reflect.getOwnPropertyDescriptor(window, 'NaN'), {
    writable: true,
    configurable: true,
})
console.log(success) // false
Reflect.getOwnPropertyDescriptor(window, 'NaN')
// {value: NaN, writable: false, enumerable: false, configurable: false}
```
结果是无法改写
## 常见场景
类型转换是典型的场景
``` 
let print = console.log;
// parseInt 
print(isNaN(parseInt("zz123"))) // true

// parseFloat 
print(isNaN(parseFloat("zz123"))) // true

// 直接Number初始化
print(isNaN(Number("zz123"))) // true

// 数字运算
print(isNaN(0 / 0 ))  // true
print(isNaN( 1 * "zz123" )) // true
print(Math.sqrt(-1)) // true
```
## isNaN
isNaN() 是一个全局方法，其本质是检查 toNumber 返回值， 如果是NaN，就返回 true，反之返回 false 。
``` 
Number.isNaN =  function (val){
   return Object.is(Number(val), NaN); 
}
```
先获取原始类型的值，再转为Number。取原值会根据条件执行不同的方法
1. 最优先调用 Symbol.toPrimitive, 如果存在
2. 根据条件决定是先调用 valueOf 还是toString

 valueOf的返回，可以直接影响isNaN的值
``` 
let print = console.log;
var person = {
    age: 10,
    name: "tom",
    valueOf(){
        return this.name 
    }
}
print(isNaN(person))  // true


let print = console.log;
var person = {
    age: 10,
    name: "tom",
    valueOf(){
        return this.age 
    }
}
print(isNaN(person))  // false
```
isNaN是可以被删除的，但是不可被枚举：  
``` 
delete isNaN // true
typeof // undefined

isNaN = 1  // 1
isNaN == 1  //true
```
## Number.isNaN
判断一个值是否是数字，并且值等于NaN.可以语义化为：
``` 
Number.isNaN = function(val){
   if(typeof val !== "number"){
       return false
   }
   return Object.is(val, NaN);
}

```
## isNaN和Number.isNaN的区别
Number.isNaN是严格判断, 必须严格等于NaN。是不是NaN这个值。  
isNaN是通过内部的 toNumber 转换结果来判定的。Number转换的返回值是不是NaN  
Number.isNaN是ES6的语法，固然存在一定的兼容性问题。  
## Object.is
判断两个值是否属于同一个值，其能准确的判断NaN
``` 
let print = console.log;
print(Object.is(NaN, NaN)); // true
print(Object.is("123", NaN)) // false
```
## 严格判断NaN汇总
Number.isNaN (ES6)
``` 
Number.isNaN(NaN) // true
Number.isNaN(1) // false
```
Object.is (ES6)
``` 
function isNaNVal(val){
    return Object.is(val, NaN);
}
isNaNVal(NaN) // true
isNaNVal(1) // false
```
自身比较 (ES5)
``` 
function isNaNVal(val){
    return val !== val;
}
isNaNVal(NaN) // true
isNaNVal(1) // false
```
typeof + NaN (ES5)  
MDN推荐的垫片，有些兼容低版本的库就是这么实现的, 也是ES标准的精准表达
``` 
function isNaNVal(val){
    return typeof val === 'number' && isNaN(val)
}
```
综合的垫片
``` 
if(!("isNaN" in Number)) {
    Number.isNaN = function (val) {
      return typeof val === 'number' && isNaN(val)
    }
}
```
## 深究数组的indexOf与includes
``` 
var arr=[NaN];
arr.indexOf(NaN) // -1
arr.includes(NaN) // true
```
**includes**   
ES标准的Array.prototype.includes 比较值相等调用的是内部的 SameValueZero ( x, y )方法，其会检查值第一值是不是数字，如果是数字，调用的是 Number::sameValueZero(x, y), 其具体比较步骤：
``` 
1. If x is NaN and y is NaN, return true.
2. If x is +0𝔽 and y is -0𝔽, return true.
3. If x is -0𝔽 and y is +0𝔽, return true.
4. If x is the same Number value as y, return true.
5. Return false.
```
其先对NaN进行了比较，所以能检查NaN, 这里还有一个额外信息，比较的时候+0和-0是相等的， 要区分+0和-0还得用Object.is
**indexOf**  
ES标准中 Array.prototype.indexOf 值比较调用的是IsStrictlyEqual(searchElement, elementK), 其如果检查到第一个值为数字，调用的 Number::equal(x, y).
比对逻辑  
``` 
1. If x is NaN, return false.
2. If y is NaN, return false.
3. If x is the same Number value as y, return true.
4. If x is +0𝔽 and y is -0𝔽, return true.
5. If x is -0𝔽 and y is +0𝔽, return true.
6. Return false.
```
任何一个为NaN，就直接返回false，必然不能严格的检查NaN.
## Number::sameValueZero 和 Number::sameValue  
**区别**  
Number::sameValueZero 不区分+0 -0， Number::sameValue 则区分。
``` 
Object.is(+0, -0) // false, 区分+0,-0
[-0].includes(+0) // true，不区分+0,-0
```
**BigInt::sameValue和 BigInt:samgeValueZero**  
除了Number有， BigInt也有相似的比较。
``` 
Object.is(BigInt(+0),BigInt(-0))   // true
Object.is(-0n,0n) // true
Object.is(-0,0) // false
[BigInt(+0)].includes(BigInt(-0))  // false
```
核心解释：  
BigInt::sameValue ( x, y ) 调用 BigInt::equal ( x, y )  
> BigInt::equal ( x, y )  
> 1. If ℝ(x) = ℝ(y), return true; otherwise return false.

而R(x)是
> 从 Number 或 BigInt x 到数学值的转换表示为“ x 的数学值”或 R(x)。+ 0F 和-0F的数学值为0

参考：
[NaN你都未必懂，花五分钟让你懂得不能再懂](https://juejin.cn/post/7023168824975294500)
