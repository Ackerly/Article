# JavaScript 数据结构之双端队列
**什么是双端队列**  
队列是遵循先进先出（FIFO）原则的一组有序集合。队列的规则是在尾部添加新元素，从头部移除最近的元素。  
你去买早点，看到前面有人，你肯定要从最后面排队，排在最前面的人先买到早点，然后才能轮到下一个人。  
但是可能会有这种场景:  
1. 在最后面排着队呢，突然有急事，不得不赶紧离开
2. 付完钱刚离开，突然发现没拿吸管，赶快跑到最前面拿一根  

这两种情况从队列的角度来说，是违反“先进先出”的原则的，因为最后入列的新元素，竟然从队列尾部出列了；本该出列的第一个元素，竟然又从队列头部入列了！这就相当于在队列中，又发生了栈的“后进先出”的情况。这就是双端队列，双端队列约等于 队列+栈。  
看具体的概念：双端队列，英文名 deque，是一种允许我们同时从头部和尾部添加和移除元素的特殊队列。可以把它看作是队列和栈相结合的一种数据结构。  
双端队列在计算机世界里最常见的应用是——撤销。比如你在做文本编辑，每一个操作都被记录在了一个双端队列中。正常情况下，你输入的文字会源源不断从这个队列的尾部插入。当满足某种条件之后，比如最多1000字，你可以把最前面的文字从队列头部移除。  
但是你突然发现有个字写错了，此时是可以在键盘上用 ctrl+z 撤销的，这个撤销操作就是把最后面的文字从队列尾部出列，表示这个文字被移除了。  
计算机和现实生活中的双端队列，都同时遵循了“先进先出”和“后进先出”的原则。  

**实现双端队列**  
``` 
const Deque = (()=> {
  let _start;
  let _end;
  let _items;
  class Deque {
    constructor() {
      _start = 0;
      _end = 0;  
      _items = 0;
    }
  }
  return Deque
})()
```
items 属性用来存储双端队列的元素，因为双端队列也是一种队列，队列基本的方法如 isEmpty，clear， size 和 toString  
双端队列相比于普通队列有以下几个方法：  
- addFront()：从双端队列头部添加元素
- addBack()：从双端队列尾部添加元素（与队列的 enqueue 方法一样）
- removeFront()：从双端队列头部移除元素（与队列的 dequeue 方法一样）
- removeBack()：从双端队列尾部移除元素（与栈的 pop 方法一样）
- peekFront()：返回双端队列的第一个元素（与队列的 peek 方法一样）
- peekBack()：返回双端队列的最后一个元素（与栈的 peek 方法一样）

这些方法只有 addFront() 是双端队列独有的，其他方法都能在栈和队列的实现中找到。具体实现:  
``` 
addFront(item) {
  if(this.isEmpty()) {
    return this.addBack(item)
  }
  if(_start>0) {
    _start--;
    _items[_start] = item
  } else {
    for(let i = 0; i<_end; i++) {
      _items[i+1] = _items[i]
    }
    _end++;
    _start = 0;
    _items[0] = item
  }
}
```
如上代码，实现在双端队列头部入列，分三种情况。
_情况一：队列为空_  
如果队列为空，则从队列的头部和尾部插入元素是一样的。所以这种情况我们直接调用从尾部插入元素的方法即可  
_情况二：start 的值大于0_  
tart 的值大于 0 表示该双端队列已经有元素出列，这种情况下，我们只需要把新添加的元素放到最后出列的元素的位置即可。这个位置的 key 值就是 start - 1  
_情况三：start 的值等于0_  
start 的值等于 0 表示双端队列中没有元素出列。此时要在头部新添加一个元素，那么就将队列中所有元素的 key 值往后挪一位，也就是 +1。让新元素的 key 值为0。  
其他方法都在栈和队列的文章里介绍过，完整代码如下：  
``` 
const Deque = (() => {
  let _start
  let _end
  let _items
  class Deque {
    constructor() {
      _start = 0
      _end = 0
      _items = {}
    }
    // 头部入列
    addFront(item) {
      if (this.isEmpty()) {
        return this.addBack(item)
      }
      if (_start > 0) {
        _start--
        _items[_start] = item
      } else {
        for (let i = _end; i > 0; i--) {
          _items[i] = _items[i-1]
        }
        _end++
        _start = 0
        _items[0] = item
      }
    }
    // 尾部入列
    addBack(item) {
      _items[_end] = item
      _end++;
    }
    // 头部出列
    removeFront() {
      if (this.isEmpty()) {
        return undefined
      }
      let item = _items[_start]
      delete _items[_start]
      _start++;
      return item
    }
    // 尾部出列
    removeBack() {
      if (this.isEmpty()) {
        return undefined
      }
      _end--;
      let item = _items[_end]
      delete _items[_end]
      return item
    }
    // 头部第一个元素
    peekFront() {
      if (this.isEmpty()) {
        return undefined
      }
      return _items[_start]
    }
    // 尾部第一个元素
    peekBack() {
      if (this.isEmpty()) {
        return undefined
      }
      return _items[_end]
    }
    size() {
      return _end - _start
    }
    isEmpty() {
      return _end - _start === 0
    }
    clear() {
      _items = {}
      _start = 0
      _end = 0
    }
    toString() {
      let arr = Object.values(_items)
      return arr.toString()
    }
  }
  return Deque
})()
```


原文:  
[怒肝 JavaScript 数据结构 — 双端队列篇](https://mp.weixin.qq.com/s/aJv_Vz6NMocQExgLlLglvw)
