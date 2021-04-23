# Vue为什么不用index作为key

## key的作用是什么
1. key的作用是唯一标识，用来表示Vue组件的唯一性，更好的区分能过高效的更新虚拟DOM
2. 如果不使用key,Vue会使用最小化element移动，尝试尽最大程度在同适当的地方对相同类型的element，做patch或者reuse
3. 使用key的话，Vue根据key记录element，拥有key的element如果不出现的话会直接remove或者destoryed。
4. 拥有同一个parent的children必须有同一个unique key，重复的key会导致render error。
在patch过程中，执行patch vnode，vnode执行updateChildren会更新两个新旧子元素，通过key判断这两个节点是否是同一个节点，如果没有加key就会认为是同一个节点，就会强硬更新，性能就会很差。
## 为什么不要使用index作为key
Diff算法根据key来进行不同的处理
1. 如果不是相同节点的话销毁vnode，渲染新的node
2. 如果新的vnode不是文字vnode，开始对children进行对比
3. 如果有新children没有旧children，直接addVnodes添加新子节点
4. 如果有旧children没有新children，直接removeVnodes删除旧子节点
5. 如果新旧children都存在，使用sameVnode进行比较
   ```
   function sameVnode (a, b) {
      return (
        a.key === b.key && (
          (
            a.tag === b.tag &&
            a.isComment === b.isComment &&
            isDef(a.data) === isDef(b.data) &&
            sameInputType(a, b)
          )
        )
      )
    }
   ```

### 为什么不要以index作为key
oldChild
```
[
  {
    tag: "item",
    key: 0,
    props: {
      num: 1
    }
  },
  {
    tag: "item",
    key: 1,
    props: {
      num: 2
    }
  },
  {
    tag: "item",
    key: 2,
    props: {
      num: 3
    }
  }
];
```
reverse操作
```
[
  {
    tag: "item",
    key: 0,
    props: {
      num: 3
    }
  },
  {
    tag: "item",
    key: 1,
    props: {
      num: 2
    }
  },
  {
    tag: "item",
    key: 2,
    props: {
      num: 1
    }
  }
];
```
在进行patchVnode时候，检查props有没有变更，有的话通过props.num = 3，更新这个响应式的值，触发dep.notify,触发子组件视图的重新渲染
会触发以下几个钩子：
1. updateAttrs
2. updateClass
3. updateDOMListeners
4. updateDOMProps
5. updateStyle
6. updateDirectives

节点删除时，diff不会关心子组件的内部实现，自会关心传递给子组件的属性是否更新
```
[
  // 第一个被删了
  {
    tag: "li",
    key: 0,
    // 这里其实上一轮子组件对应的是第二个 假设子组件的text是2
  },
  {
    tag: "li",
    key: 1,
    // 这里其实子组件对应的是第三个 假设子组件的text是3
  },
];
```
这样将导致：
1. 原来的第一个节点text：1复用
2. 第二个节点text：2直接复用
3. 少了text3这个节点


### 为什么不用随机数作为key
因为随机数会造成每次渲染都是创建新节点，销毁纠结点，不能实现节点的复用，导致性能下降

参考文章：  
[为什么 Vue 中不要用 index 作为 key？](https://juejin.cn/post/6844904113587634184)
