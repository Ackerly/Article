# 什么时候使用Map或胜过Object
## 为什么对象不符合 Hash Map 的使用情况  
在 Hash Map 中使用对象最明显的缺点是，对象只允许键是字符串和 symbol。任何其他类型的键都会通过 toString 方法被隐含地转换为字符串  
``` 
const foo = []
const bar = {}
const obj = {[foo]: 'foo', [bar]: 'bar'}

console.log(obj) // {"": 'foo', [object Object]: 'bar'}
```
更重要的是，使用对象做 Hash Map 会造成混乱和安全隐患  

## 不必要的继承
在ES6之前，获得 hash map 的唯一方法是创建一个空对象：  
``` 
const hashMap = {}
```
在创建时，这个对象不再是空的。尽管 hashMap 是用一个空的对象字面量创建的，但它自动继承了 Object.prototype。这就是为什么我们可以在 hashMap 上调用hasOwnProperty、toString、constructor 等方法，尽管我们从未在该对象上明确定义这些方法。  
由于原型继承，现在有两种类型的属性被混淆了：存在于对象本身的属性，即它自己的属性，以及存在于原型链的属性，即继承的属性。  
因此，需要一个额外的检查（例如hasOwnProperty）来确保一个给定的属性确实是用户提供的，而不是从原型继承的。  
除此之外，由于属性解析机制在 JavaScrip t中的工作方式，在运行时对 Object.prototype 的任何改变都会在所有对象中引起连锁反应。这就为原型污染攻击打开了大门，这对大型的JavaScript 应用程序来说是一个严重的安全问题。  
除此之外，由于属性解析机制在 JavaScrip t中的工作方式，在运行时对 Object.prototype 的任何改变都会在所有对象中引起连锁反应。这就为原型污染攻击打开了大门，这对大型的JavaScript 应用程序来说是一个严重的安全问题。  

## 名称冲突
当一个对象自己的属性与它的原型上的属性有名称冲突时，它就会打破预期，从而使程序崩溃。  
例如，我们有一个函数 foo，它接受一个对象。  
``` 
function foo(obj) {
	//...
	for (const key in obj) {
		if (obj.hasOwnProperty(key)) {
			
		}
	}
}
```
obj.hasOwnProperty(key)有一个可靠性风险：考虑到属性解析机制在JavaScript中的工作方式，如果 obj 包含一个开发者提供的具有相同名称的 hasOwnProperty 属性，那就会对Object.prototype.hasOwnProperty产生影响。因此，我们不知道哪个方法会在运行时被准确调用。  
可以做一些防御性编程来防止这种情况。例如，我们可以从 Object.prototype 中 "借用""真正的 hasOwnProperty 来代替:  
``` 
function foo(obj) {
	//...
	for (const key in obj) {
		if (Object.prototype.hasOwnProperty.call(obj, key)) {
			// ...
		}
	}
}
```
还有一个更简短的方法就是在一个对象的字面量上调用该方法，如{}.hasOwnProperty.call(key)，不过这也挺麻烦的。这就是为什么还会新出一个静态方法Object.hasOwn 的原因了。  
## 次优的人机工程学
Object 没有提供足够的人机工程学，不能作为 hash map 使用，许多常见的任务不能直观地执行。  
**size**  
Object 并没有提供方便的API来获取 size，即属性的数量。而且，对于什么是一个对象的 size ，还有一些细微的差别:  
- 如果只关心字符串、可枚举的键，那么可以用 Object.keys() 将键转换为数组，并获得其length
- 如果k只想要不可枚举的字符串键，那么必须得使用 Object.getOwnPropertyNames 来获得一个键的列表并获得其 length
- 如果只对 symbol 键感兴趣，可以使用 getOwnPropertySymbols 来显示 symbol 键。或者可以使用 Reflect.ownKeys 来一次获得字符串键和 symbol 键，不管它是否是可枚举的

上述所有选项的运行时复杂度为O(n)，因为我们必须先构造一个键的数组，然后才能得到其长度。  

**iterate**  
可以使用 for...in循环。但它会读取到继承的可枚举属性。  
``` 
Object.prototype.foo = 'bar'

const obj = {id: 1} 

for (const key in obj) {
	console.log(key) // 'id', 'foo'
}
```
不能对一个对象使用 for ... of，因为默认情况下它不是一个可迭代的对象，除非我们明确定义 Symbol.iterator 方法在它上面。  
可以使用 Object.keys、Object.values 和 Object.entry 来获得一个可枚举的字符串键（或/和值）的列表，并通过该列表进行迭代，这引入了一个额外的开销步骤。  
还有一个是 插入对象的键的顺序并不是按我们的顺序来的，这是一个很蛋疼的地方。在大多数浏览器中，整数键是按升序排序的，并优先于字符串键，即使字符串键是在整数键之前插入的：  
``` 
const obj = {}

obj.foo = 'first'
obj[2] = 'second'
obj[1] = 'last'

console.log(obj) // {1: 'last', 2: 'second', foo: 'first'}
```
**clear**  
没有简单的方法来删除一个对象的所有属性，必须用 delete 操作符一个一个地删除每个属性，这在历史上是众所周知的慢。  

**检查属性是否存在**  
不能依靠点/括号符号来检查一个属性的存在，因为值本身可能被设置为 undefined。相反，得使用 Object.prototype.hasOwnProperty 或 Object.hasOwn。  
``` 
const obj = {a: undefined}

Object.hasOwn(obj, 'a') // true
```

## Map
ES6 为我们带来了 Map，首先，与只允许键值为字符串和 symbols 的 Object 不同，Map 支持任何数据类型的键。  
但更重要的是，Map 在用户定义的和内置的程序数据之间提供了一个干净的分离，代价是需要一个额外的 Map.prototype.get 来获取对应的项。  
Map 也提供了更好的人机工程学。Map 默认是一个可迭代的对象。这说明可以用 for ... of 轻松地迭代一个 Map，并做一些事情，比如使用嵌套的解构来从 Map 中取出第一个项。  
``` 
const [[firstKey, firstValue]] = map
```
与 Object 相比，Map 为各种常见任务提供了专门的API:  
- Map.prototype.has 检查一个给定的项是否存在，与必须在对象上使用Object.prototype.hasOwnProperty/Object.hasOwn 相比，不那么尴尬了
- Map.prototype.get 返回与提供的键相关的值。有的可能会觉得这比对象上的点符号或括号符号更笨重。不过，它提供了一个干净的用户数据和内置方法之间的分离
- Map.prototype.size 返回 Map 中的项的个数，与获取对象大小的操作相比，这明显好太多了。此外，它的速度也更快
- Map.prototype.clear 可以删除 Map 中的所有项，它比 delete 操作符快得多

## 性能差异
大多数情况下，Map 要比 Object 快。有些人声称通过从 Object 切换到 Map 可以看到明显的性能提升。  
在 LeetCode 上也证实了这种想法，对于数据量大的 Object 会超时，但 Map 上则不会。  

## 测试
测试用例有一个表格，主要测试 Object 和 Map 在插入、迭代和删除数据的速度。  
插入和迭代的性能是以每秒的操作来衡量的。这里使用了一个实用函数 measureFor，它重复运行目标函数，直到达到指定的最小时间阈值（即用户界面上的 duration 输入字段）。它返回这样一个函数每秒钟被执行的平均次数。  
``` 
function measureFor(f, duration) {
  let iterations = 0;
  const now = performance.now();
  let elapsed = 0;
  while (elapsed < duration) {
    f();
    elapsed = performance.now() - now;
    iterations++;
  }

  return ((iterations / elapsed) * 1000).toFixed(4);
}
```
至于删除，只是要测量使用 delete  操作符从一个对象中删除所有属性所需的时间，并与相同大小的 Map 使用 Map.prototype.delete 的时间进行比较。也可以使用Map.prototype.clear，但这有悖于基准测试的目的，因为我知道它肯定会快得多。  
在这三种操作中，我更关注插入操作，因为它往往是我在日常工作中最常执行的操作。对于迭代性能，很难有一个全面的基准，因为我们可以对一个给定的对象执行许多不同的迭代变体。这里我只测量 for ... in 循环。  
在这里使用了三种类型的 key  
- 字符串，例如：Yekwl7caqejth7aawelo4
- 整数字符串，例如：123
- 由 Math.random().toString() 生成的数字字符串，例如：0.4024025689756525

所有的键都是随机生成的，所以我们不会碰到V8实现的内联缓存。我还在将整数和数字键添加到对象之前，使用 toString 明确地将其转换为字符串，以避免隐式转换的开销。  
最后，在基准测试开始之前，还有一个至少100ms的热身阶段，在这个阶段，我们反复创建新的对象和 Map，并立即丢弃。  
从大小为 100 个属性/项的 Object 和 Map 开始，一直到 5000000，并让每种类型的操作持续运行 10000ms，看看它们之间的表现如何。下面是测试结果：  
**string keys**  
当键为（非数字）字符串时，Map 在所有操作上都优于 Object。  
但细微之处在于，当数量并不真正多时（低于100000），Map 在插入速度上 是Object 的两倍，但当规模超过 100000 时，性能差距开始缩小。  
对于几百或几千个数据的规模，Map 的性能至少是 Object 的两倍。因此，我们是否应该就此打住，并开始重构我们的代码库，全部采用 Map？  
这不太靠谱......或者至少不能期望我们的应用程序变得快 2 倍。记住我们还没有探索其他类型的键。下面我们看一下整数键。  

**integer keys**  
我之所以特别想在有整数键的对象上运行基准，是因为V8在内部优化了整数索引的属性，并将它们存储在一个单独的数组中，可以线性和连续地访问。  
先尝试在 [0, 1000] 范围内的整数键。Object 这次的表现超过了 Map。它们的插入速度比 Map 快65%，迭代速度快16%。  
扩大范围，使键中的最大整数为 1200。现在 Map 的插入速度开始比 Object 快一点，迭代速度快 5 倍。  
现在，只增加了整数键的范围，而不是 Object 和 Map 的实际大小。让我们加大 size，看看这对性能有什么影响。  
当属性 size 为 1000 时，Object 最终比 Map 的插入速度快 70%，迭代速度慢2倍。  
随着 size 的增长，以一些相对较小的整数作为键值，Object 在插入方面比Map 更有性能，在删除方面总是大致相同，迭代速度慢4或5倍。  
Object 在插入时开始变慢的最大整数键的阈值会随着 Object 的大小而增长。例如，当对象只有100个条数据，阈值是1200；当它有 10000 个条目时，阈值似乎是 24000 左右。  

**numeric keys**  
这里的数字键特指由 Math.random().toString() 生成的数字字符串。  
结果与那些字符串键的情况类似。Map 开始时比 Object 快得多（插入和删除快2倍，迭代快4-5倍），但随着我们规模的增加，差距也越来越小。  

## 内存使用情况
由于无法控制浏览器环境中的垃圾收集器，这里决定在 Node 中运行基准测试。  
这里创建了一个小脚本来测量它们各自的内存使用情况，并在每次测量中手动触发了完全的垃圾收集。用 node --expose-gc 运行它，就得到了以下结果。  
``` 
{
  object: {
    'string-key': {
      '10000': 3.390625,
      '50000': 19.765625,
      '100000': 16.265625,
      '500000': 71.265625,
      '1000000': 142.015625
    },
    'numeric-key': {
      '10000': 1.65625,
      '50000': 8.265625,
      '100000': 16.765625,
      '500000': 72.265625,
      '1000000': 143.515625
    },
    'integer-key': {
      '10000': 0.25,
      '50000': 2.828125,
      '100000': 4.90625,
      '500000': 25.734375,
      '1000000': 59.203125
    }
  },
  map: {
    'string-key': {
      '10000': 1.703125,
      '50000': 6.765625,
      '100000': 14.015625,
      '500000': 61.765625,
      '1000000': 122.015625
    },
    'numeric-key': {
      '10000': 0.703125,
      '50000': 3.765625,
      '100000': 7.265625,
      '500000': 33.265625,
      '1000000': 67.015625
    },
    'integer-key': {
      '10000': 0.484375,
      '50000': 1.890625,
      '100000': 3.765625,
      '500000': 22.515625,
      '1000000': 43.515625
    }
  }
}
```
很明显，Map 比 Object 消耗的内存少20%到50%，这并不奇怪，因为 Map 不像 Object 那样存储属性描述符，比如 writable/enumerable/configurable  


原文:  
[在 JavaScript 中，什么时候使用 Map 或胜过 Object](https://juejin.cn/post/7141174031411052581)
