# proposal-extractors
proposal-extractors 是一个关于解构能力增强的提案，支持在直接解构时执行自定义逻辑  
**概述**  
```
const [first, second] = arr;
const { name, age } = obj;
```
以上就是解构带来的便利，如果没有解构语法，相同的实现我们需要这么做：  
```
const first = arr[0];
const second = arr[1];
const name = obj.name;
const age = obj.age;
```
较为原始的方法可以在对象赋值时进行一些加工，比如：  
```
const first = someLogic(arr[0]);
const second = someLogic(arr[1]);
const name = someLogic(obj.name);
const age = someLogic(obj.age);
```
解构语法就没那么简单了，想要实现类似的效果，需要退化到多行代码实现，冗余度甚至超过非解构语法  
```
const [first: firstTemp, second: secondTemp] = arr
const {name: nameTemp, age: ageTemp} = obj
const first = someLogic(firstTemp)
const second = someLogic(secondTemp)
const name = someLogic(nameTemp)
const age = someLogic(ageTemp)
```
proposal-extractors 提案就是用来解决这个问题，希望保持解构语法优雅的同时，加一些额外逻辑：  
```
const SomeLogic(first, second) = arr // 解构数组
const SomeLogic{name, age} = obj // 解构对象
```
稍稍有点别扭，使用 () 解构数组，使用 {} 解构对象。我们再看 SomeLogic 的定义：  
```
const SomeLogic = {
  [Symbol.matcher]: (value) => {
    return { matched: true, value: value.toString() + "特殊处理" };
  },
};
```
这样我们拿到的 first、second、name、age 变量就都变成字符串了，且后缀增加了 '特殊处理' 这四个字符。  
为什么用 () 表示数组解构呢？主要是防止出现赋值歧义：  
```
// 只有一项时，[] 到底是下标含义还是解构含义呢？
const SomeLogic[first] = arr
```

proposal-extractors 提案提到了 BindingPattern 与 AssignmentPattern：  
```
// binding patterns
const Foo(y) = x;           // instance-array destructuring
const Foo{y} = x;           // instance-object destructuring
const [Foo(y)] = x;         // nesting
const [Foo{y}] = x;         // ..
const { z: Foo(y) } = x;    // ..
const { z: Foo{y} } = x;    // ..
const Foo(Bar(y)) = x;      // ..
const X.Foo(y) = x;         // qualified names (i.e., a.b.c)

// assignment patterns
Foo(y) = x;                 // instance-array destructuring
Foo{y} = x;                 // instance-object destructuring
[Foo(y)] = x;               // nesting
[Foo{y}] = x;               // ..
({ z: Foo(y) } = x);        // ..
({ z: Foo{y} } = x);        // ..
Foo(Bar(y)) = x;            // ..
X.Foo(y) = x;               // qualified names (i.e., a.b.c)
```
BindingPattern 相比 AssignmentPattern 只是前面多了一个 const 标记。那么 BindingPattern 与 AssignmentPattern 分别表示什么含义呢？  
BindingPattern 与 AssignmentPattern 是解构模式下的特有概念。  
BindingPattern 需要用 const let 等变量定义符描述。比如下面的例子，生成了 a、d 两个新对象，我们称这两个对象被绑定了（binding）  
```
const obj = { a: 1, b: { c: 2 } };
const {
  a,
  b: { c: d },
} = obj;
// Two variables are bound: `a` and `d`
```
AssignmentPattern 无需用变量定义符描述，只能用已经定义好的变量，所以可以理解为对这些已经存在的变量赋值。比如下面的例子，将对象的 a b 分别绑定到数组 numbers 的每一项。  
```
const numbers = [];
const obj = { a: 1, b: 2 };
({ a: numbers[0], b: numbers[1] } = obj);
```
proposal-extractors 是针对解构的增强提案，自然也要支持 BindingPattern 与 AssignmentPattern 这两种模式。


原文:  
[精读《proposal-extractors》](https://mp.weixin.qq.com/s/GTry9tnXZ3fMZyJMsmEwjQ)
