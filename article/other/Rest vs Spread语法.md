# Rest vs Spread语法
## Spread
... 作为 Spread 含义时，效果为扩散对象的属性：
``` 
const obj = {
  a: 1,
  b: 2,
  c: 3,
};

const newObj = {
  ...obj,
};

console.log(newObj);
// { a: 1, b: 2, c: 3 }
```
... 符号很形象的表示了把对象中所有属性拿出来平铺的含义。说到平铺，Spread 放在函数参数时，也表示将对象中每个 properties 拿出来作为平铺参数：
``` 
const arr = [1, 2, 3];

const sum = (a, b, c) => a + b + c;

console.log(sum(...arr)); // Outputs: 6
//                 ^
//          sum(1, 2, 3)
```
## Rest
... 作 Rest 含义时，表示将多个值收集为一个数组，如用在函数定义的位置：
``` 
const sum = (...args) => {
  return args.reduce((acc, curr) => acc + curr, 0);
  //      ^
  //  [1, 2, 3]
};

console.log(sum(1, 2, 3));
// 6
```
当然也可以在 ... 前面放置其他变量，这样 ... 仅聚合剩余的变量。... 之后不能再定义变量或者 ...
``` 
const sum = (a, b, ...restOfArguments) => {
  return a + b + restOfArguments.reduce((acc, curr) => acc + curr, 0);
  //     ^   ^          ^
  //     1   2      [3, 4, 5]
};

console.log(sum(1, 2, 3, 4, 5));
// 15
```
## Rest 处理 Set 与 Map
Set 与 Map 都可以通过数组模式赋初值：
``` 
const mySet = new Set(["a", "b", "c"]);
const myMap = new Map([
  ["a", 1],
  ["b", 2],
  ["c", 3],
]);
```
在 ... 符号作 Rest 用途时，可以将其解构为数组：
``` 
[...mySet] // ['a', 'b', 'c']
[...myMap] // [ ['a', 1], ['b', 2], ['c', 3] ]
```
特别的，Map 与 Set 仅支持数组方式解构，不支持对象模式解构：
``` 
{...mySet} // {}
{...myMap} // {}
```
但对于一个普通数组，是同时支持数组与对象模式解构的：
``` 
const arr = ['a', 'b', 'c']
[...arr] // ['a', 'b', 'c']
{...arr} // {0: 'a', 1: 'b', 2: 'c'}
```
这是因为数组变量有潜在的下标，这些下标可以转换为对象的 Key，而 Map Set 不存在下标，所以转换为对象找不到 Key，因此就不支持对象模式的解构。  
更具体的原因与对象的可迭代性有关，虽然 Map 与 Set 都支持迭代，但如果用 for key of 来测试，会发现它们的 key 是 undefined。
## Spread 会丢失 get() 与 set()
Spread 并不代表完整复制整个对象，它能拷贝这个对象属性定义中的瞬时值，比如：
``` 
const obj = {
    a: 1,
    b: get() { return 2 }
}
const newObj = { ...obj }
```
newObj.b 属性不再是 get() 方法，而是固定值 2，这在 get() 函数内返回非固定值，或希望懒加载代码时会产生问题。  
Spread 毕竟不是在定义对象，更恰当的理解应该是 “访问对象”，所以访问的结果就是执行 get()。

## Rest 会跳过不可枚举属性
``` 
const err = new Error('error')
{...error} // {}
```
Error 拥有两个不可枚举属性 message 与 stack，所以不会被 Rest 收集到，遇到这种场景可以使用其他方式，如直接访问 error.message。

原文:  
[精读《Rest vs Spread 语法》](https://mp.weixin.qq.com/s/cSp-uDJCln1mi-9UhHrFIg)
