# Iterator迭代器模式
提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。  
这种设计模式要解决的根本问题是，聚合的种类有很多，比如对象、链表、数组、甚至自定义结构，但遍历这些结构时，不同结构的遍历方式又不同，所以我们必须了解每种结构的内部定义才能遍历。  
比如数组我们可以利用 length + for 循环，对象我们可以 Object.keys，链表比较麻烦，需要内部暴露出元素的 next 以操作指向下一个元素。  
迭代器模式可以做到用同一种 API 遍历任意类型聚合对象，且不用关心聚合对象的内部结构。  
这种模式和 Array.from 有点像，但真正的迭代器在 JS 里是 obj【Symbol.iterator】()，也就是一个对象实现了 【Symbol.interator】，就认为是可遍历的。  
## 举例
**generator**  
generator 天生为迭代器的 API：
``` 
function* func () {
  yield 'a';
  yield 'b';
  return 'c';
}

var run = func();
run.next() // {value: "a", done: false}
run.next() // {value: "b", done: false}
run.next() // {value: "c", done: true}
```
无需关心 generator 内部是何种存储结构，只需要调用 .next()，并根据返回的 done 来判断是否遍历完即可。在 generator 的场景中，迭代器不仅用来遍历聚合，还用于执行代码。  
**数组迭代器**  
用迭代器的方式遍历数组：  
``` 
const arr = [1, 2, 3]
const run = arr[Symbol.iterator]()

run.next() // {value: 1, done: false}
run.next() // {value: 2, done: false}
run.next() // {value: 2, done: false}
run.next() // {value: undefined, done: true}
```
可能有人觉得这是画蛇添足，因为遍历数组用 for 循环更方便，但这就是设计模式与非设计模式思维的区别，重要的不是用熟悉简单的 API 快速满足需求，设计模式关注的是如何统一、抽象、低耦合的编码。  
**Map 迭代器**  
Map 对象也可以用迭代器方式遍历：
``` 
const citys = new Map([['北京', 1], ['上海', 2], ['杭州', 3]])
const run = citys.entries()

run.next() // {value: ['北京', 1], done: false}
run.next() // {value: ['上海', 2], done: false}
run.next() // {value: ['杭州', 3], done: false}
run.next() // {value: undefined, done: true}
```
## 解释
虽然用迭代器遍历数组看上去比 for 循环麻烦一点，但当我们把所有聚合类型放到一起看时，可以发现只有迭代器的 API 是最统一的，是唯一一个不需要关心聚合类型就可以完成遍历的方案。

![image](./../../assets/images/design%20patterns/Iterator.png)

-  Aggregate: 聚合，需要定义创建迭代器的接口。比如前端规范的 [Symbol.iterator]()，或者这里定义的 CreateIterator()。
- Iterator: 迭代器，定义了访问与遍历的 API。

迭代器的实现时要考虑的因素包括：  
- 健壮性。即迭代过程中增加、删除元素后，还能正常遍历。或者遍历空聚合时也要能正常工作。
- 外部控制迭代还是内部。即类似 KOA 由插件调用 next() 控制迭代，还是由外层统一控制迭代。  
- 如何定义遍历算法。即便对于对象这种简单场景，也存在深度优先和广度优先、冒泡与捕获这几种遍历顺序，迭代器可以提供选择或者拓展的方式，自定义遍历算法。  

``` 
// 定义聚合接口
interface Aggregate{
  getIterator: () => Iterator
}

// 定义迭代器接口
interface Iterator {
  // 指向下一个
  next: () => void
}

// 定义一个聚合
class List implements Aggregate {
  // 存储元素
  public values: string[]

  // 游标
  public index: number

  getIterator() {
    return new ConcreteIterator(this);
  }
}

// List 的迭代器
class ConcreteIterator implements Iterator {
  constructor(list: List) {
    this.list = list
  }

  next() {
    return this.list.values[this.list.index] // 注意边界情况，这里就不展开
    this.list.index++
  }
}
```
## 缺点
如果你只是遍历数组，直接用 for 循环会比迭代器方便很多，没必要为了用设计模式而用设计模式。迭代器仅在以下情况可以考虑用于数组：  
- 这个数组比较特殊，是 N 维数组，需要一次性遍历完，那么可以用迭代器。
- 同时遍历数组和其他类型的聚合，则不论数组还是其他聚合，都用相同的迭代器模式遍历最好。

参考:  
[Iterator（迭代器模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/182.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Iterator%20%E8%BF%AD%E4%BB%A3%E5%99%A8%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
