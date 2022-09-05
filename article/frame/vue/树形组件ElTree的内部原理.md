# 树形组件ElTree的内部原理
## 设计思想
设计Tree组件的时候是采用两颗树进行互相映射的方案进行设计的，一颗树是用户自定义节点构成的树RawNode，另一颗是内部进行渲染的树TreeNode。当RawNode某个节点的值变更后Mapper就会得到通知，然后通过通知的内容对TreeNode进行更改。  
Mapper需要完成:
- 节点转换与映射
- 变更监听
- 响应更改

注意：
- 在RawNode变更后Mapper要对TreeNode进行修改，但是在修改TreeNode后不能在通知变更去修改RawNode
- 在监听节点的时候，需要对存储子节点的数组进行监听(children)

## 测试先行
> 每次在开发类或者函数之前肯定要先想接口，然后再实现，而TDD只是将内心的活动带到了现实里，这样做有几点好处
> 1. 之后可以自动测试
> 2. 理清楚了接口
> 3. 在你对代码大动刀戈的时候他可以起到一定的指引作用

- 首先需要一个TreeNode类作为树的节点
- 一个可以监听对象的工具类Watcher
- 一个可以事件通知的工具类Event
- 一个需要对象映射Mapper类

### TreeNode Spec
需要传入id、label、children来实现节点的创建
``` 
describe('TreeNode.js', ()=>{
    it('init a node', ()=>{ // 初始化一个节点
        const root = new TreeNode(1, 'Node1', [
            new TreeNode(2, 'Node2', [])
        ])
        
        expect(root.id).toBe(1)
        expect(root.label).toBe('Node1')
        expect(root.children[0].id).toBe(2)
        expect(root.children[0].label).toBe('Node')
    })
})
```
### Watcher Spec
对对象的操作进行拦截，才能知道这个对象的变化
- 监听节点内哪个属性修改了什么值
- 监听存放孩子节点的数组
    - 节点的增加
    - 节点的修改
    - 节点的删除
    
``` 
describe('Watcher.js', ()=>{
    it('listern a node prop change', ()=>{
        const root = {
            label: 'Node1',
        }
        const watcher = new Watcher(root)
        const _root = watcher.proxy // 拿到代理后的对象
        
        const changeHandler = jest.fn()
        watcher.bindHandler('change', changeHandler)
        const addHandler = jest.fn()
        watcher.bindHandler('add', addHandler)
        
        _root.label = "Test"
        expect(changeHandler).toHaveBeenCalledTimes(1)
       	
        _root.disabled = true
        expect(addHandler).toHaveBeenCalledTimes(1)
    })
    
    it('listen a node children node pointer and length change', ()=>{
        const root = {
            label: 'Node1',
            children:[
            	{
                    label: 'Node2'
                }
            ]
        }
        const watcher = new Watcher(root)
        const _root = watcher.proxy // 拿到代理后的对象
        
        const childrenChangeHandler = jest.fn()
        watcher.bindHandler('array/change', childrenChangeHandler)
        const addHandler = jest.fn()
        watcher.bindHandler('array/', addHandler)
        
        _root.label = 'Test'
        expect(childrenChangeHandler).toHaveBeenCalledTimes(1)
        expect(childrenChangeHandler).toHaveBeenNthCalledWith(1, {
          target: root,
          key: 'label',
          value: 'Test',
          currentNode: root
        })
        
        _root.disabled = true
        expect(addHandler).toHaveBeenCalledTimes(1)
        expect(childrenChangeHandler).toHaveBeenNthCalledWith(1, {
          target: root,
          key: 'disabled',
          value: true,
          currentNode: root
        })
    })
})
```
### Event Spec
实现事件的监听和发送
``` 
describe('Event.js', ()=>{
	it('listen a event', ()=>{
        const event = new Event()
        const cb = jest.fn()
        event.on('ev1', cb)
        event.emit('ev1', 1, 2, 3)

        expect(cb).toHaveBeenCalledTimes(1)
        expect(cb).toHaveBeenCalledWith(1, 2, 3)
    })
})
```
### Mapper Spec
- 节点转换与映射
- 变更监听
- 响应更改
``` 
describe('Mapper.js', () => {
  it('mapper a tree', () => {
    const rawNode = {
      text: 'Node1',
      childs: [
        {
          text: 'Node11',
          childs: [
            {
              text: 'Node111',
              childs: []
            }
          ]
        }
      ]
    }

    const mapper = new TreeMapper(rawNode, {
      label: 'text',
      children: 'childs'
    })

    const rawNodeProxy = mapper.rawNodeProxy
    const treeNodeProxy = mapper.treeNodeProxy
    
    rawNodeProxy.text = "Test"

    expect(rawNodeProxy.text).toEqual(treeNodeProxy.label)
    expect(rawNodeProxy.childs[0].text).toEqual(treeNodeProxy.children[0].label)
    expect(rawNodeProxy.childs[0].childs[0].text).toEqual(
      treeNodeProxy.children[0].children[0].label
    )
  })
  ...
})
```
## 实现原理
### TreeNode
``` 
class TreeNode {
  constructor(id, label, children) {
    this.id = id
    this.label = label;
    this.children = children ?? [];
  }
}
```
### Watcher
通过Proxy进行代理拦截，然后通过Event推送出去
``` 
class Watcher {
  constructor(target) {
    this.event = new Event(); // 用于变更通知
    this.toProxy = new WeakMap(); // WeakMap 有个特别好的特性，可以自动移除未引用的对象
    this.toRaw = new WeakMap();
    this.proxy = this.reactive(target, target);
  }

  reactive(target, lastTarget) { // 嵌套响应式
    if (!isObject(target) || this.toRaw.has(target)) { // 如果当前是代理，或者不是对象则返回
      return target;
    }

    if (this.toProxy.has(target)) { // 如果当前对象以及代理则返回代理
      return this.toProxy.get(target);
    }

    const currentNode = isArray(lastTarget) ? target : lastTarget; // 获取当前的节点

    const handler = {
      get: this.createGetter(),
      set: this.createSetter(currentNode),
      deleteProperty: this.createDeleteProperty(currentNode),
    };

    const observer = new Proxy(target, handler);
    this.toProxy.set(target, observer); // 建立原始对象和代理
    this.toRaw.set(observer, target);   // 对象的映射关系
    return observer;
  }

  bindHandler(type, callback) { // 绑定一个通知
    this.event.on(type, callback);
  }

  createGetter() {
    return (target, key) => {
      const res = Reflect.get(target, key);
      return isObject(res) ? this.reactive(res, target) : res;
        	 // 如果是对象，则继续嵌套代理，如果不是对象则返回这个值
    };
  }

  createSetter(currentNode) {
    return (target, key, value) => {
      if (this.toRaw.has(value)) { // 如果写入的是已经被代理的对象，则先转换为普通对象
        value = this.toRaw.get(value);
      }
      if (isArray(target)) { 
        if (key === "length") {
          this.event.emit("array/changeLength", {
            target,
            key,
            value,
            currentNode,
          });
        } else {
          if (Reflect.has(target, key)) {
            // 修改
            this.event.emit("array/change", {
              target,
              key,
              value,
              currentNode,
            });
          } else {
            // 新增
            this.event.emit("array/append", {
              target,
              key,
              value,
              currentNode,
            });
          }
        }
      } else {
        if (Reflect.has(target, key)) {
          // 修改
          this.event.emit("change", { target, key, value, currentNode });
        } else {
          // 新增
          this.event.emit("add", { target, key, value, currentNode });
        }
      }
      return Reflect.set(target, key, value);
    };
  }

  createDeleteProperty(currentNode) {
    return (target, key) => {
      if (isArray(target)) {
        this.event.emit("array/delete", { target, key, currentNode });
      } else {
        this.event.emit("delete", { target, key, currentNode });
      }
      return Reflect.deleteProperty(target, key);
    };
  }
}
```
### Event
``` 
class Event {
  constructor() {
    this.events = new Map();
  }

  on(name, callback) {
    if (!this.events.has(name)) {
      this.events.set(name, new Set([callback]));
      return;
    }
    this.events.get(name).add(callback);
  }

  emit(name, ...args) {
    if (this.events.has(name)) {
      this.events.get(name).forEach((cb) => cb(...args));
    }
  }
}
```
### Mapper
- 转换RawNode -> TreeNode
- 对RawNode进行Watcher监听
- 对TreeNode进行Watcher监听
- 当发生变更通知后，对原数据进行修改
``` 
class Mapper {
  constructor(rawNode, keyMap) {
    this.toTreeNode = new WeakMap();
    this.toRawNode = new WeakMap();
    this.toRawNodeKey = keyMap;
    this.toTreeNodeKey = reversalNodeKeyMap(keyMap); // 反向 NodeKey
    // 初始化

    this.rawNode = rawNode;
    this.treeNode = this.convertToTreeNode(rawNode);
    // 生成TreeNode

    this.rawNodeWatcher = new Watcher(this.rawNode);
    this.treeNodeWatcher = new Watcher(this.treeNode);
    this.withRawNodeHandler();
    this.withTreeNodeHandler();
    // 对 rawNode 与 treeNode 分别进行响应式处理
  }

  convertToTreeNode(rawNode) {
    const treeNode = new TreeNode(
      rawNode[this.toRawNodeKey.id],
      rawNode[this.toRawNodeKey.label],
      this.convertToTreeNodes(rawNode[this.toRawNodeKey.children]),
      { isChecked: rawNode[this.toRawNodeKey.isChecked] }
    );
    this.toTreeNode.set(rawNode, treeNode);
    this.toRawNode.set(treeNode, rawNode);
    return treeNode;
  }

  convertToRawNode(treeNode) {
    const rawNode = {
      [this.toRawNodeKey.id]: treeNode.id,
      [this.toRawNodeKey.label]: treeNode.label,
      [this.toRawNodeKey.children]: this.convertToRawNodes(treeNode.children),
    };
    this.toTreeNode.set(rawNode, treeNode);
    this.toRawNode.set(treeNode, rawNode);
    return rawNode;
  }

  convertToTreeNodes(rawNodes) {
    return rawNodes?.map((node) => this.convertToTreeNode(node));
  }

  convertToRawNodes(treeNodes) {
    return treeNodes?.map((node) => this.convertToRawNode(node));
  }

  withRawNodeHandler() {
    this.rawNodeWatcher.bindHandler(
      "array/append",
      ({ currentNode, value }) => {
        const currentTreeNode = this.toTreeNode.get(currentNode);
        this.forTreeNodeAppendChild(
          currentTreeNode,
          this.convertToTreeNode(value)
        );
      }
    );

    this.rawNodeWatcher.bindHandler("array/delete", ({ currentNode, key }) => {
      const currentTreeNode = this.toTreeNode.get(currentNode);
      this.forTreeNodeRemoveChild(currentTreeNode, key);
    });

    this.rawNodeWatcher.bindHandler(
      "array/change",
      ({ currentNode, key, value }) => {
        const currentTreeNode = this.toTreeNode.get(currentNode);
        this.forTreeNodeUpdateChild(
          currentTreeNode,
          key,
          this.toTreeNode.get(value) ?? this.convertToTreeNode(value)
        );
      }
    );

    this.rawNodeWatcher.bindHandler("change", ({ currentNode, key, value }) => {
      const currentTreeNode = this.toTreeNode.get(currentNode);
      this.forTreeNodeUpdateValue(
        currentTreeNode,
        this.toTreeNodeKey[key],
        value
      );
    });

    this.rawNodeWatcher.bindHandler("add", ({ currentNode, key, value }) => {
      const currentTreeNode = this.toTreeNode.get(currentNode);
      this.forTreeNodeUpdateValue(
        currentTreeNode,
        this.toTreeNodeKey[key],
        value
      );
    });
  }

  withTreeNodeHandler() {
    this.treeNodeWatcher.bindHandler(
      "array/append",
      ({ currentNode, value }) => {
        const currentRawNode = this.toRawNode.get(currentNode);
        this.forRawNodeAppendChild(
          currentRawNode,
          this.convertToRawNode(value)
        );
      }
    );

    this.treeNodeWatcher.bindHandler("array/delete", ({ currentNode, key }) => {
      const currentRawNode = this.toRawNode.get(currentNode);
      this.forRawNodeRemoveChild(currentRawNode, key);
    });

    this.treeNodeWatcher.bindHandler(
      "array/change",
      ({ currentNode, key, value }) => {
        const currentRawNode = this.toRawNode.get(currentNode);
        this.forRawNodeUpdateChild(currentRawNode, key, value);
      }
    );

    this.treeNodeWatcher.bindHandler(
      "change",
      ({ currentNode, key, value }) => {
        const currentRawNode = this.toRawNode.get(currentNode);
        this.forRawNodeUpdateValue(
          currentRawNode,
          this.toRawNodeKey[key],
          value
        );
      }
    );

    this.treeNodeWatcher.bindHandler("add", ({ currentNode, key, value }) => {
      const currentRawNode = this.toRawNode.get(currentNode);
      this.forRawNodeUpdateValue(currentRawNode, this.toRawNodeKey[key], value);
    });
  }

  forTreeNodeAppendChild(currentTreeNode, newTreeNode) {
    currentTreeNode.children.push(newTreeNode);
  }

  forTreeNodeUpdateValue(currentTreeNode, key, value) {
    if (key === "children") {
      currentTreeNode[key] = this.convertToTreeNodes(value);
    } else {
      currentTreeNode[key] = value;
    }
  }

  forTreeNodeRemoveChild(currentTreeNode, index) {
    // TODO: 这里还得对toRawNode 与 toTreeNode 进行处理，不然会内存泄漏
    currentTreeNode.children.splice(index, 1);
  }

  forTreeNodeUpdateChild(currentTreeNode, index, childNode) {
    currentTreeNode.children[index] = childNode;
  }

  forRawNodeAppendChild(currentRawNode, newRawNode) {
    currentRawNode[this.toRawNodeKey.children].push(newRawNode);
  }

  forRawNodeUpdateValue(currentRawNode, key, value) {
    if (key === this.toRawNodeKey["children"]) {
      currentRawNode[key] = this.convertToRawNodes(value);
    } else if (Reflect.has(currentRawNode, key)) {
      currentRawNode[key] = value;
    }
  }

  forRawNodeRemoveChild(currentRawNode, index) {
    currentRawNode[this.toRawNodeKey.children].splice(index, 1);
  }

  forRawNodeUpdateChild(currentRawNode, index, childNode) {
    currentRawNode[this.toRawNodeKey.children][index] = childNode;
  }
}
```

原文: 
[【Vue3组件化源码】树形组件ElTree的内部原理](https://juejin.cn/post/6926144123669839880?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
