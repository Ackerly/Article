# 理解Render阶段Fiber树的初始化与更新
react框架可以用来表示，输入状态 —> 吐出ui。  
**react架构是什么？**  
可以分为如下三层：  
1. scheduler（调度器）：用来分发优先级更高的任务。
2. render阶段（协调器）：找出哪些节点发生了变化，并且给相应的fiber打上标签
3. commit阶段（渲染器）：将打好标签的节点渲染到视图上。遍历effectList执行对应的dom操作或部分生命周期


1. 输入: 将每一次更新(如: 新增, 删除, 修改节点之后)视为一次更新需求(目的是要更新DOM节点).
2. 注册调度任务: react-reconciler收到更新需求之后, 并不会立即构造fiber树, 而是去调度中心scheduler注册一个新任务task, 即把更新需求转换成一个task.  
3. 执行调度任务(输出): 调度中心scheduler通过任务调度循环来执行task
   1. fiber构造循环是task的实现环节之一, 循环完成之后会构造出最新的 fiber 树.
   2. commitRoot是task的实现环节之二, 把最新的 fiber 树最终渲染到页面上, task完成.

主干逻辑就是输入到输出这一条链路, 为了更好的性能(如批量更新, 可中断渲染等功能), react在输入到输出的链路上做了很多优化策略, 任务调度循环和fiber构造循环相互配合就可以实现可中断渲染.  

主干逻辑就是输入到输出这一条链路, 为了更好的性能(如批量更新, 可中断渲染等功能), react在输入到输出的链路上做了很多优化策略, 任务调度循环和fiber构造循环相互配合就可以实现可中断渲染.  

**ReactElement, Fiber, DOM 三者的关系**  
``` 
function Test() {
  const [showName, setShowName] = useState(true);
  return (
    <div>
      <div>今天肯德基疯狂星期八，和我一起玩彩虹六？</div>
      <ul>
        <li>抱枕一号</li>
        {showName && <li>抱枕二号</li>}
      </ul>
      <div
        onClick={() => {
          setShowName(false);
        }}
      >
        点击让高启强少一个小弟
      </div>
    </div>
  );
}
```
所有采用JSX语法书写的节点, 都会被编译器转换, 最终会以React.createElement(...)的方式, 创建出来一个与之对应的ReactElement对象  
这也是为什么在每个使用JSX的JS文件中，你必须显式的声明 import React from 'react';(17版本后不需要）否则在运行时该模块内就会报未定义变量 React的错误  

**ReactElement数据结构和内存结构**  
_数据结构_  
``` 
export type ReactElement = {
  // 用于辨别ReactElement对象形式
  $$typeof: any,

  // 内部属性
  type: any, // 表明其种类
  key: any,
  ref: any,
  props: any,

  // ReactFiber 记录创建本对象的Fiber节点, 还未与Fiber树关联之前, 该属性为null
  _owner: any,

  // __DEV__ dev环境下的一些额外信息, 如文件路径, 文件名, 行列信息等
  _store: {validated: boolean, ...},
  _self: React$Element<any>,
  _shadowChildren: any,
  _source: Source,
};
```
**Fiber 对象数据结构**  
_数据结构_  
``` 
export type Fiber = {|
  tag: WorkTag,
  key: null | string, // 和ReactElement组件的 key 一致.
  elementType: any,//一般来讲和ReactElement组件的 type 一致 比如div ul
  type: any, // 一般来讲和fiber.elementType一致. 一些特殊情形下, 比如在开发环境下为了兼容热更新
  stateNode: any, // 真实DOM是谁
  return: Fiber | null, //爹是谁
  child: Fiber | null, //孩子是谁
  sibling: Fiber | null, //兄弟是谁
  index: number, 
  ref:
    | null
    | (((handle: mixed) => void) & { _stringRef: ?string, ... })
    | RefObject, //指向在ReactElement组件上设置的 ref
  pendingProps: any, // 从`ReactElement`对象传入的 props. 用于和`fiber.memoizedProps`比较可以得出属性是否变动
  memoizedProps: any, // 上一次生成子节点时用到的属性, 生成子节点之后保持在内存中
  updateQueue: mixed, // 存储state更新的队列, 当前节点的state改动之后, 都会创建一个update对象添加到这个队列中.
  memoizedState: any, // 用于输出的state, 最终渲染所使用的state
  dependencies: Dependencies | null, // 该fiber节点所依赖的(contexts, events)等
  mode: TypeOfMode, // 二进制位Bitfield,继承至父节点,影响本fiber节点及其子树中所有节点. 与react应用的运行模式有关(有ConcurrentMode, BlockingMode, NoMode等选项).

  // 优先级相关
  lanes: Lanes, // 本fiber节点的优先级
  childLanes: Lanes, // 子节点的优先级
  alternate: Fiber | null, // 双fiber缓存 指向内存中的另一个fiber, 每个被更新过fiber节点在内存中都是成对出现(current和workInProgress)
|};
```

**React的启动过程发生了啥**  
``` 
ReactDOM.render(<App />, document.getElementById('root'), dom => {});
```
在没有进入render阶段（react-reconciler包）之前,reactElement(<App/>)和 DOM 对象div#root之间没有关联。  
在react初始化的时候，会创建三个全局对象，在三个对象创建完毕的时候，react初始化完毕。  
1. ReactDOMRoot对象
   1. 属于react-dom包，该对象暴露有render,unmount方法, 通过调用该实例的ReactDOM.render方法, 可以引导 react 应用的启动
2. fiberRoot对象
   1. 属于react-reconciler包,在运行过程中的全局上下文, 保存 fiber 构建过程中所依赖的全局状态
   2. 其大部分实例变量用来存储fiber构造循环过程的各种状态，react 应用内部, 可以根据这些实例变量的值, 控制执行逻辑
3. HostRootFiber对象
   1. 属于react-reconciler包，这是 react 应用中的第一个 Fiber 对象, 是 Fiber 树的根节点, 节点的类型是HostRoot

这 3 个对象是 react 体系得以运行的基本保障, 除非卸载整个应用，否则不会再销毁  

## 双缓冲fiber技术**  
fiber树的构造过程, 就是把ReactElement转换成fiber树的过程. 但是在这个过程中, 内存里会同时存在 2 棵fiber树:  
- 其一: 代表当前界面的fiber树(已经被展示出来, 挂载到fiberRoot.current上). 如果是初次构造(初始化渲染), 页面还没有渲染, 此时界面对应的 fiber 树为空(fiberRoot.current = null).
- 其二: 正在构造的fiber树(即将展示出来, 挂载到HostRootFiber.alternate上, 正在构造的节点称为workInProgress). 当构造完成之后, 重新渲染页面, 最后切换fiberRoot.current = workInProgress, 使得fiberRoot.current重新指向代表当前界面的fiber树.

## fiber 树构造方式    
1. 初次创建: 在React应用首次启动时, 界面还没有渲染, 此时并不会进入对比过程, 相当于直接构造一棵全新的树
2. 对比更新: React应用启动后, 界面已经渲染. 如果再次发生更新, 创建新fiber之前需要和旧fiber进行对比. 最后构造的 fiber 树有可能是全新的, 也可能是部分更新的

在深度优先遍历中, 每个fiber节点都会经历 2 个阶段
1. 探寻阶段 beginWork
2. 回溯阶段 completeWork

**beginWork探寻阶段发生了什么**  
1. 创建节点：根据 ReactElement对象创建所有的fiber节点, 最终构造出fiber树形结构(设置return和sibling指针)
2. 给节点打标签：设置fiber.flags(二进制形式变量, 用来标记 fiber节点 的增,删,改状态, 等待completeWork阶段处理)
3. 设置真实DOM的局部状态：设置fiber.stateNode局部状态(如Class类型节点: fiber.stateNode=new Class())

**completeWork回溯阶段发生了什么**  
1. 调用completeWork
   1. 给fiber节点(tag=HostComponent, HostText)创建 DOM 实例, 设置fiber.stateNode局部状态(如tag=HostComponent, HostText节点: fiber.stateNode 指向这个 DOM 实例)
   2. 为 DOM 节点设置属性, 绑定事件(合成事件原理)
   3. 设置fiber.flags标记
2. 把当前 fiber 对象的副作用队列(firstEffect和lastEffect)添加到父节点的副作用队列之后, 更新父节点的firstEffect和lastEffect指针
3. 识别beginWork阶段设置的fiber.flags, 判断当前 fiber 是否有副作用(增,删,改), 如果有, 需要将当前 fiber 加入到父节点的effects队列, 等待commit阶段处理

## 初次创建 
生成时期的 beginWork 和 completeWork 执行过程  
``` 
// 将最新的fiber树挂载到root.finishedWork节点上 下面绿色粗线表示指针
  const finishedWork: Fiber = (root.current.alternate: any);
  root.finishedWork = finishedWork;
  root.finishedLanes = lanes;
  // 进入commit阶段
  commitRoot(root);
```
fiber树构造循环负责构造新的fiber树, 构造过程中同时标记fiber.flags, 最终把所有被标记的fiber节点收集到一个副作用队列中, 这个副作用队列被挂载到根节点上(HostRootFiber.alternate.firstEffect). 此时的fiber树和与之对应的DOM节点都还在内存当中, 等待commitRoot阶段进行渲染  

## 对比更新的时候发生了什么
**优化原则**
1. 只对同级节点进行对比，如果DOM节点跨层级移动，则react不会复用
   1. 我们可以从同级的节点数量将Diff分为两类：
      1. 当newChild类型为JSX对象、number、string，代表同级只有一个节点
   2. 不同类型的元素会产出不同的结构，会销毁老的结构，创建新的结构
   3. 可以通过key标示移动的元素
   4. 类型一致的节点才有继续diff的必要性
      1. 单节点对应演示,可以去浏览器的Elements->Properties查看
       
**diff算法介绍**   
1. 单节点
   1. 如果是新增节点, 直接新建 fiber, 没有多余的逻辑
   2. 如果是对比更新
      1. 如果key和type都相同，则复用
      2. 否则新建

2. 多节点
   1. 多节点一般会存在两轮遍历，第一轮寻找公共序列，第二轮遍历剩余非公共序列
   2. 第一次循环
      1. key不同导致不可复用，立即跳出整个遍历，第一轮遍历结束
      2. key相同type不同导致不可复用，会将oldFiber标记为DELETION，并继续遍历
      3. 如果newChildren遍历完（即i === newChildren.length - 1）或者oldFiber遍历完（即oldFiber.sibling === null），跳出遍历，第一轮遍历结束。
      4. let i = 0，遍历newChildren，将newChildren[i]与oldFiber比较，判断DOM节点是否可复用
      5. 如果可复用，i++，继续比较newChildren[i]与oldFiber.sibling，可以复用则继续遍历
    
   3. 第二次循环: 遍历剩余非公共序列, 优先复用 oldFiber 序列中的节点
      1. 如果newChildren与oldFiber同时遍历完，diff结束
      2. 如果 newChildren没遍历完，oldFiber遍历完，意味着没有可以复用的节点了，遍历剩下的newChildren为生成的workInProgress fiber依次标记Placement
      3. 如果newChildren遍历完，oldFiber没遍历完，意味着有节点被删除了，需要遍历剩下的oldFiber，依次标记Deletion
      4. 如果newChildren与oldFiber都没遍历完
         1. 先去`声明map数据结构`，遍历一遍老节点，把老fiber的key做映射 \{元素的key：老的fiber节点\}
         2. 继续遍历新`jsx`，如果`map`有`key`，会把`key`从`map`中删除，说明可以复用，把当前节点标记为`更新`。新地位高的不动，新地位低的动（中间插入链表比链表屁股插入费劲）所以地位低的动动。
         3. `lastPlaceIndex`指针，指向最后一个不需要动的老节点的`key`。每次新jsx复用到节点，`lastPlaceIndex`会指向老节点的最后一个成功复用的老`fiber`节点。
         4. 如果新复用的节点key小于`lastPlaceIndex`，说明老`fiber`节点的顺序在新`jsx`之前，需要挪动位置接到新`jsx`节点后面。如果`jsx`没有复用的老`fiber`，直接插入新的
         5. `map`中只剩还没被复用的节点，等着新的`jsx`数组遍历完，`map`里面的`fiber`节点全部设置为删
         

原文:  
[深入理解Render阶段Fiber树的初始化与更新](https://mp.weixin.qq.com/s/b1lmYPuz9HMCEE_oxWq6Cg)
