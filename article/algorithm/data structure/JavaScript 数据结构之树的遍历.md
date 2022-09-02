# 树的遍历
树的遍历有三种方式：  
- 中序遍历
- 先序遍历
- 后序遍历

**中序遍历**  
中序遍历是以从小到大的顺序访问二叉搜索树（BST）所有节点的遍历方式，该方式常常用来对树进行排序。  
实现：  
``` 
inOrderTraverse(callback) {
  this.inOrderTraverseNode(this.root, callback)
}
```
inOrderTraverse 方法接受一个回调函数，这个回调函数的作用是对每个遍历到的节点进行操作。因为遍历树用到递归，所以定义inOrderTraverseNode 作为递归函数，从根节点开始遍历。  
``` 
inOrderTraverseNode(node, callback) {
  if(node != null) {
    this.inOrderTraverseNode(node.left, callback)
    callback(node.key)
    this.inOrderTraverseNode(node.right, callback)
  }
}
```
这个函数首先判断传入的 node 是否为空，不为空才进行遍历，这是递归的终止条件。因为节点有左右两个子节点，所以分别调用递归函数，然后再调用 callback 回调函数，将节点的 key 作为参数传递  

**先序遍历**  
先序遍历是遵循先遍历根节点的原则，再递归遍历左右侧子节点，实现如下：
``` 
preOrderTraverse(callback) {
  this.preOrderTraverseNode(this.root, callback)
}
```
preOrderTraverseNode 方法的实现如下：  
``` 
preOrderTraverseNode(node, callback) {
  if(node != null) {
    callback(node.key)
    this.preOrderTraverseNode(node.left, callback)
    this.preOrderTraverseNode(node.right, callback)
  }
}
```
先序遍历 与 中序遍历 的区别仅仅是递归函数内代码执行顺序的不同。中序遍历的顺序是 左-中-右，先序遍历的执行顺序是 中-左-右。  

**后序遍历**  
遍历和前面两个的遍历逻辑基本一样，不同的也是递归函数内的执行顺序。  
``` 
postOrderTraverseNode(node, callback) {
  if(node != null) {
    this.preOrderTraverseNode(node.left, callback)
    this.preOrderTraverseNode(node.right, callback)
    callback(node.key)
  }
}
```
后序遍历遵循 左-右-中 的执行顺序  

**查找某个值**  
所谓“暴力查找”就是不考虑性能直接遍历整棵树，直到找到某个节点。暴力查找也不用考虑用哪种遍历方式，直接遍历就行了，就好像 JavaScript 当中的 ForEach 一样。  
比如要查找值为 25 的节点：  
``` 
bin_tree.postOrderTraverse((key)=> {
  if(key == 25) {
    console.log('找到啦！')
  }
})
```


参考:  
[怒肝 JavaScript 数据结构 — 树的遍历](https://mp.weixin.qq.com/s/bX7IJQgrFS6eUn5ZvgyIUQ)
