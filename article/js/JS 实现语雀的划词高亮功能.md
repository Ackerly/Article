# JS 实现语雀的划词高亮功能
划词评论的库大部分的实现都是需要破坏页面 DOM 结构的，也就是需要在页面 DOM 结构中拆分文本包裹一个 mask 标签，但是由于我们做的是在线富文本文档功能，文本的内容是可以再编辑的，如果评论破坏了 DOM 结构这样对我们编辑的时候编辑器解析就不是很友好。语雀实现的划词评论功能是基于 canvas 实现的，与页面结构完全解耦  

## 实现思路  
主要思路：生成一个 canvas 元素，让 canvas 元素与需要划词高亮功能的文本容器元素等宽高，并且重叠在文本容器上，划词的时候获取划词区域的文本节点相对于文本容器的位置信息，然后通过这些位置信息进行高亮背景的渲染。  

## 实现细节  
**让 canvas 与文本容器元素重叠**  
让 canvas 与文本容器元素重叠最好的实现方式就是将 canvas 做为文本容器的直接子节点，然后设置文本容易为相对定位，将 canvas 设置为绝对定位，然后将 top、left、right、bottom 都设置为 0，这样就可以时刻保证 canvas 元素与文本容器元素始终等宽高，且 canvas 重叠在文本容器上。不过这种实现方式也有一个问题，我们把 canvas 的层级提高了，盖住了文本容器中的其他文本节点，这样就没办法进行划词了，所以这时候我们需要给 canvas 再添加一个 css 属性：pointerEvents: 'none'，这样就可以让 canvas 不响应鼠标事件，从而让底部文本节点可以正常划词了。  
**获取划词区域文本节点的位置信息**  
获取划词区域信息需要使用 document.getSelection().getRangeAt(0) 来获得当前划词区域的 range 对象，在这个对象上可以获取到划词区域的起始和终止文本节点以及偏移量信息。  
``` 
const {
  startContainer, // 起始节点
  startOffset, // 起始节点偏移量
  endContainer, // 终止节点
  endOffset // 终止节点偏移量
} = document.getSelection().getRangeAt(0)
```
虽然我们拿到了节点信息，但是怎么获得具体的位置信息呢？这时候就需要借助 Range 对象的强大功能了  
``` 
// 创建一个 range 对象
const range = document.createRange()
// 设置需要获取位置信息的文本节点以及偏移量
range.setStart(startContainer, startOffset)
range.setEnd(startContainer, startContainer.textContent.length)
// 通过 getBoundingClientRect 获取位置信息
const rect = range.getBoundingClientRect()
```
通过创建 range 对象我们可以获得任何一个文本节点中的任何一段文本相对与整个页面的位置信息，然后再通过减去文本容器元素相对于整个页面的位置信息，我们就可以得到划词区域文本相对与文本容器的位置信息了。  
**获取头尾中间的文本节点**  
通过 document.getSelection().getRangeAt(0) 获得了划词头尾节点的信息，但是头尾中间如果有其他的文本节点我们也需要进行背景高亮，那么中间的文本节点我们该怎么获得呢？这里是从头节点开始进行深度优先遍历，遍历到尾节点为止，然后收集遍历过程中的所有文本节点，这样我们就得到了整个划词区域内的所有文本节点，然后通过上面第 2 点的办法我们也可以得到所有文本节点的位置信息。  
``` 
function getTextNodesByDfs(start: Text, end: Text) {
  if (start === end) return []
  const iterator = nodeDfsGenerator(start, false)
  const textNodes: Text[] = []
  iterator.next()
  let value = iterator.next().value
  while (value && value !== end) {
    if (node.nodeType === 3) {
      textNodes.push(value)
    }
    value = iterator.next().value
  }
  if (!value) {
    return []
  }
  return textNodes
}
// 返回节点的深度优先迭代器
// 对于有子节点的 Node 会遍历到两次，不过 Text Node 肯定没有子节点，所以不会重复统计到
function * nodeDfsGenerator(node: Node, isGoBack = false): Generator<Node, void, Node> {
  yield node
  // isGoBack 用于判断是否属于子节点遍历结束回退到父节点，如果是那么该节点不再遍历其子节点
  if (!isGoBack && node.childNodes.length > 0) {
    yield * nodeDfsGenerator(node.childNodes[0], false)
  } else if (node.nextSibling) {
    yield * nodeDfsGenerator(node.nextSibling, false)
  } else if (node.parentNode) {
    yield * nodeDfsGenerator(node.parentNode, true)
  }
}
```
**处理跨行文本节点的位置信息**  
第 2 点获取划词区域文本节点的位置信息的方案还有缺陷，对于跨行的文本节点我们如果仍然采用一个 range 去获取位置信息，那么得到位置信息是错误的,range 只能是一个矩形，并没有办法表示我们跨行选中时的不规则图形的位置信息  
既然一个 range 不行，那么多个呢？所以我们的解决思路就是将一个跨行的 range 拆分成多个不跨行的 range。  
使用的办法是通过二分法的方式去找到每一行的最后一个文本节点去拆分，怎么判断两个字符是否在同一行采用的创建一个单位长度的 range?比较 range 位置信息中的 top 是否相同来进行判断  
``` 
// 将一个跨行的 range 切割为多个不跨行的 range
function splitRange(node: Text, startOffset: number, endOffset: number): Range[] {
  const range = document.createRange()
  const rowTop = getCharTop(node, startOffset)
  // 字符数小于两个不用判断是否跨行
  // 头尾高度一致说明在同一行
  if ((endOffset - startOffset < 2) || rowTop === getCharTop(node, endOffset - 1)) {
    range.setStart(node, startOffset)
    range.setEnd(node, endOffset)
    return [range]
  } else {
    const last = findRowLastChar(rowTop, node, startOffset, endOffset - 1)
    range.setStart(node, startOffset)
    range.setEnd(node, last + 1)
    const others = splitRange(node, last + 1, endOffset)
    return [range, ...others]
  }
}

// 二分法找到 range 某一行的最右字符
function findRowLastChar(top: number, node: Text, start: number, end: number): number {
  if (end - start === 1) {
    return getCharTop(node, end) === top ? end : start
  }
  const mid = (end + start) >> 1
  return getCharTop(node, mid) === top
    ? findRowLastChar(top, node, mid, end)
    : findRowLastChar(top, node, start, mid)
}

// 获取 range 某个字符位置的 top 值
function getCharTop(node: Text, offset: number) {
  return getCharRect(node, offset).top
}

// 获取 range 某个字符位置的 DOMRect
function getCharRect(node: Text, offset: number) {
  const range = document.createRange()
  range.setStart(node, offset)
  range.setEnd(node, offset + 1 > node.textContent!.length ? offset : offset + 1)
  return range.getBoundingClientRect()
}
```
这样位置信息的问题我们就彻底解决了，接下来就可以使用这些信息去canvas 上渲染我们想要的高亮背景效果了  
**划词信息持久化与返显**  
实现了高亮的功能，但是设想如果我们做的是划词评论功能，那么肯定还涉及到将划词信息保存到后端，但是我们这一切的开头都是从系统提供的一个 range 对象开始的，但是 range 对象上的 startContainer 和 endContainer 是保存着 DOM 节点的引用，这肯定没办法序列化存储到后端的，所以我们需要一种方式能让我们准确的找到我们想要的文本节点  
语雀中的每一个文本标签都有一个固定的 id，这样他们实现起来就很简单了，只需要保存对应的 id 就行，但是采用这种方式就需要你对页面的每个文本标签都设置一个文本 id，这样显然与我们最初与页面文本结构解耦的想法不符了，所以这里采用的是类似 XPath 的方式进行储存，对于头尾节点，保存一个路径数组，里面储存的是从文本容器通过 childNodes 属性遍历下去找到该节点的信息，这样对于任何的页面结构我们都可以使用了。  
``` 
// 获取从文本容器到文本节点的路径信息，用于存储
function getPath(textNode: Text) {
  const path = [0]
  let parentNode = textNode.parentNode
  let cur: Node = textNode
  while (parentNode) {
    if (cur === parentNode.firstChild) {
      // this.root 为文本容器
      if (parentNode === this.root) {
        break
      } else {
        cur = parentNode
        parentNode = cur.parentNode
        path.unshift(0)
      }
    } else {
      cur = cur.previousSibling!
      path[0]++
    }
  }
  return parentNode ? path : null
}

// 根据路径信息获取文本节点，用于返显
function getNodeByPath(path: number[]) {
  // this.root 为文本容器
  let node: Node = this.root
  for (let i = 0; i < path.length; i++) {
    if (node && node.childNodes && node.childNodes[path[i]]) {
      node = node.childNodes[path[i]]
    } else {
      return null
    }
  }
  return node
}
```

原文:  
[纯 JS 实现语雀的划词高亮功能](https://mp.weixin.qq.com/s/F2OVWaMLFg8EofNr3M-8sg)
