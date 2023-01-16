# Didact 解析
## 简介
以 Didact 作为引导，可以更好地解决以下的问题：  
1. 引导我们逐步理解 Didact，且这个过程与理解 React 是一致的
2. 代码清晰，理解难度小，便于入门和上手
3. 沿用了 React 的设计理念与思想，React 从 16.8 以后，代码在变但思想没变，学习 Didact 可以帮助我们了解 React 的理念与思想

## 原理与源码解析
**从一个简单的例子开始**  
``` 
// 一个jsx element
const element = <h1 title="foo">Hello</h1>;
// 一个根结点容器
const container = document.getElementById("root");
// 将element渲染到容器里
ReactDOM.render(element, container);
```
这段代码用 React 实现了一个最简单的应用：将一个 h1 渲染到 root 容器里  
之所以我们可以在.jsx 文件里写类似 html 标签的 JSX，是因为 babel 会在打包时将 JSX 转义为 React.createElement(args)或者_jsx(args)，本质上 JSX 写法就是这两个方法的语法糖，本文中都将以 createElement 为例：  
``` 
const element = <h1 title="foo">Hello</h1>;
// 会被babel转义为如下
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
);
```
所以，要实现一个简单的 React，就必须实现 createElement 与 render 两个方法，我们这里可以统一到 Didact 里，也就是：  
``` 
function createElement() {/** do sth */}
function render() {/** do sth */}
const Didact = {
  createElement,
  render
};
```
**实现 createElement 方法**  
举一个上面例子的变形：  
``` 
const element = (
  <h1 title="foo">
    <div>123</div>
    Hello
  </h1>
);
```
对于此 element，需要三个字段来对它进行描述：
- 类型 —— h1
- 属性 —— title="foo"
- 孩子 —— <div>123</div> 与 Hello

所以，createElement 的入参可以设定为 type、props、children，其中 children 参数可以有多个，返回类型为 DidactElement，可以将 children 合并到 props 中管理，事实上在 React 中我们也是通过const { children } = props; 来获取孩子 elements，这里对照 Didact 源码来看：  
``` 
type DidactElement = {
  type: string | function; // 本篇文章只实现原生组件和函数式组件
  props: { children: DidactElement[]; [props: string]: any; };
};
/**
* @param type
  元素类型，可能是原生的div、h1等，也可能是Function Component等
* @param props
  元素属性，原生节点中对应DOM properties，FC中对应组件的传值
  @param ...children
  孩子elements，数目不确定
  @return DidactElement
*/
function createElement(
  type: string,
  props: Record<string, any>,
  ...children: DidactElement[]
): DidactElement {
  return {
    type,
    props: {
      ...props,
      // 此处要判断传入的是jsx element还是文本，文本另作处理
      children: children.map(child =>
        (typeof child === 'object' ? child : createTextElement(child))
      )
    },
  };
}

// 文本节点处理的方法，相比jsx element，其type和props都比较固定，且无children
function createTextElement(text: string): DidactElement {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: []
    }
  };
}
```
现在有了 createElement 方法，接下来可以用 babel 来做 transform，只需要加一行注释，构建时 babel 就会将 JSX 转义为 Didact.createElement()：  
``` 
/** @jsx Didact.createElement */
const element = (
  <h1 title="foo">
    <div>123</div>
    Hello
  </h1>
);
```
**实现一个原始的 render 方法**  
实现 createElement 方法后，可以开始着手 render 方法。通过上面的例子可知，render 的入参有两个，渲染的元素 element 和容器 container。我们可以通过 DOM 操作基本实现一个原始的 render 方法：  
``` 
function render(element: DidactElement, container: HTMLElement) {
  // 根据element创建真实的节点
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
  // 将DOM Properties加入到创建的节点上
  Object.keys(element.props)
    .filter(key => key !== "children")
    .forEach(name => {
      dom[name] = element.props[name]
    })；
  // 递归，将每一个孩子节点渲染出来，注意此时容器为当前节点
  element.props.children.forEach(child =>
    render(child, dom)
  );
  // 将节点插入到容器中
  container.appendChild(dom);
}
```
由此，一个静态的 JSX --> DOM 的转化成型了，在接下来的章节中，我们会实现 Didact 赖以工作的核心机制 —— Fiber 架构，所以原始的 render 方法也会随着后续的讲解来重写、完善。  

**前置知识：Concurrent 模式**  
上述 render 函数中，会递归处理孩子 elements 直到生成完整 DOM，JS 线程释放，GUI 线程才会进行渲染工作。这个递归的过程是不可中断的，如果 Element Tree 结构非常复杂，则会导致 JS 执行时间过长，GUI 线程工作阻塞，界面会掉帧卡顿，用户使用体验会大打折扣。  
所以，想要提升用户的体验，有一种思路就是在每一帧的时间内，预留一定时间给 JS 线程，让他处理 JS，没有处理完成的工作留到下一帧中继续进行，而剩下的时间给 GUI 线程来进行渲染，保证用户看到的画面是流畅的，这也是 time slice(时间切片)的思想。  
浏览器有一个实验性的 API —— requestIdleCallback，这个 API 可以使我们在浏览器当前帧有空闲时间时执行 JS，通过此方法我们可以拿到当前帧的剩余可用时间，一般用法如下：  
``` 
type Deadline = {
  timeRemaining: () => number; // 当前剩余的可用时间，即该帧剩余时间
  didTimeout: boolean; // 是否超时
}
// 该方法执行callback时会传入该参数
function work(deadline: Deadline) {
  // 如果剩余可用时间>1ms或者没有超时，即说明当前帧还有剩余时间，可以执行JS
  if (deadline.timeRemaining() > 1 || deadline.didTimeout) {
     // do sth
  }
  // 如果没有剩余时间，则等待下一帧；或者完成工作后继续等待下一轮执行
  requestIdleCallback(work);
}
requestIdleCallback(work);
```
但此用法还是有问题，第 9 行的 do something 一旦开始依旧不会停止，所以如果把上面提到的递归的过程放到这里，依旧会一直执行无法中断。  
造成这种情况的本质原因是所有的递归工作都耦合在一起，因此我们第 8 行只判断了一次。如果可以把这个工作分割成一个一个的单元，这样每个单元超时的概率会远远小于总工作超时的概率。  
因此，我们将递归的形式转变为循环的形式，对一个个单元进行处理，而在循环中我们每一次都可以进行第 8 行的判断，就可以解决这个问题了，如下所示：  
``` 
function work(deadline: Deadline) {
  // 循环地、一个一个单元地进行工作
  while (deadline.timeRemaining() > 1 || deadline.didTimeout) {
     // 只做一个单元的工作
  }
  // 退出循环后，等待下一轮的空闲时间可以继续上面的工作
  requestIdleCallback(work);
}
requestIdleCallback(work);
```
同时，这也是 Fiber 架构的核心思想之一  

**Fiber 架构与两个阶段**  
在主流的 Vue、React 框架中，都沿用了虚拟 DOM 这一思想，即用 JavaScript 对象来描述浏览器的 DOM，用一个 JS 的树形结构来描述一个真实的 DOM 树。  
在 React 中用 Fiber 这种数据结构来实现虚拟 DOM 这一思想 —— 先根据 Element Tree 生成 Fiber Tree，再根据 Fiber Tree 增删改真实的 DOM Tree。  
因此，综合上述知识，我们可以引出自 v16 之后 React 采取的新的全新的架构 —— Fiber 架构，这个架构由三个部分组成：  
- Scheduler（调度器）—— 调度任务的优先级，高优任务优先进入 Reconciler；
- Reconciler（协调器）—— 负责找出变化的组件，可以被中断，对应 Render 阶段；
- Renderer（渲染器）—— 负责将变化的组件渲染到页面上，不被中断，对应 Commit 阶段

其中 Scheduler 由上述提到的 requestIdleCallback 来实现（实际上 React 受限于浏览器的兼容性和功能上的微小差异选择了自己实现）  
Fiber Tree 是一个树形结构，与传统树形结构相比，Fiber Tree 的每一个父节点的 child 指针只指向长子，长子与兄弟之间再通过 sibling 连接，而且所有的孩子都有一个 parent 指针指向父节点。  
在 Didact 中会有两棵 Fiber Tree，一棵是 Current Fiber Tree，即当前正在呈现的 DOM 对应的 Fiber Tree，还有一棵是 WorkInProgress Fiber Tree，是正在进行构建的树，两棵树中对应的节点通过 Fiber 节点上的 alternate 指针连接。  
至此可以得出 Fiber 节点的数据结构即：  
``` 
type Fiber = {
  type?: string | function; // Fiber Root 不需要此类型，详见
  props: { children: DidactElement[]; [props: string]: any; };
  dom: HTMLElement | null;
  alternate: Fiber | null;
  parent?: Fiber;
  child?: Fiber;
  sibling?: Fiber;
  effectTag?: string;
  hooks?: hook[];
};
// 下面将会用到
type hook = <T>(
  initialValue?: T | (() => T)
) => [T | undefined, (value: T) => T];
```

![](../../assets/images/other/didact work.jpg)

整个工作流程中的入口就是 workloop 函数，可以对照图和源码看一下 workloop 做了什么事情：
``` 
let nextUnitOfWork: Fiber | null = null;
let currentRoot: Fiber | null = null;
let wipRoot: Fiber | null = null;
let deletions: Fiber[] | null = null;

function workLoop(deadline: Deadline) {
  let shouldYield = false;
  // 每进行一个单元的工作，就会进行判断，可随时中断，此时是Scheduler和Reconciler在工作
  // 中断后继续执行时，nextUnitOfWork是全局变量，因此可以取到上一次的中断前的nextUnitOfWork继续进行工作
  // 这就是递归变循环的好处，真正的践行了time slice的思想
  while (nextUnitOfWork && !shouldYield) {
    // dfs遍历
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    );
    shouldYield = deadline.timeRemaining() < 1;
  }
  // 当wip fiber构建完成，就可以进入commit阶段了，此时是Renderer在工作
  if (!nextUnitOfWork && wipRoot) {
    commitRoot();
  }
  requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);
```

**Render 阶段**  
核心方法就是 performUnitOfWork，接下来直接对照源码进行解析：  
``` 
/**
 * 此方法的作用：
 * 1. 将当前fiber的props.children中的elements生成fiber节点并连接起来
 * 2. 返回长子节点，即fiber.child；若没有孩子则返回兄弟节点或叔辈节点
*/
function performUnitOfWork(fiber: Fiber) {
  // 这两个update函数实现功能1
  const isFunctionComponent =
    fiber.type instanceof Function;
  if (isFunctionComponent) {
    updateFunctionComponent(fiber);
  } else {
    updateHostComponent(fiber);
  }
  // 实现功能2
  if (fiber.child) {
    return fiber.child;
  }
  // 如果没有孩子节点了，则返回兄弟节点或者叔辈节点，即dfs的过程
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    nextFiber = nextFiber.parent;
  }
}

function updateHostComponent(fiber: Fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber);
  }
  // 上述功能1的具体实现
  reconcileChildren(fiber, fiber.props.children);
}
function updateFunctionComponent(fiber: Fiber) {
  // do sth
}
 // 当标记为"PLACEMENT"时，即新增DOM，需要执行此函数创建一个DOM
function createDom(fiber: Fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type)
  // 同时将属性注入
  updateDom(dom, {}, fiber.props);
  return dom;
}
// 接收fiber节点及其孩子elements作为参数，将孩子elements生成孩子fiber并与fiber节点连接
function reconcileChildren(wipFiber: Fiber, elements: DidactElement[]) {
  let index = 0;
  // 要进行工作的节点对应的old fiber节点，初始值为长子节点
  let oldFiber =
    wipFiber.alternate && wipFiber.alternate.child;
  // 前面完成的wip fiber的孩子节点，用来做兄弟之间的连接
  let prevSibling: Fiber | null = null;
  // 循环将孩子们生成fiber，并与current fiber节点进行对比打上标记
  while (
    index < elements.length ||
    oldFiber !== null
  ) {
    const element = elements[index];
    let newFiber: Fiber | null = null;

    const sameType =
      oldFiber &&
      element &&
      element.type == oldFiber.type;

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: wipFiber,
        alternate: oldFiber,
        effectTag: "UPDATE"
      };
    }
    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: wipFiber,
        alternate: null,
        effectTag: "PLACEMENT"
      };
    }
    /**
     删除的时候比较特殊，因为删除后新的树中就不应该有该Fiber节点了，所以只能在
     老的树中的Fiber节点上存储标记，并用一个全局的数组deletions存储这些节点
    */
    if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION";
      deletions.push(oldFiber);
    }
    // 结束当前element建fiber的工作，将oldFiber更新为下一轮的oldFiber
    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }
    // 如果是长子节点则将连接父节点与长子节点，不是则连成前一轮的节点的兄弟节点
    if (index === 0) {
      wipFiber.child = newFiber;
    } else if (element) {
      prevSibling.sibling = newFiber;
    }
    // 更新prevSibling节点，并进入下一轮
    prevSibling = newFiber;
    index++;
  }
}
```
至此，从 DidactElement --> Fiber 的工作完成，找到了变化的组件，并打上了相应的标记，后续在 commit 阶段，会根据不同的标记进行相应的 DOM 操作。  
原始 render 方法可以进行重写了：
``` 
function render(element: DidactElement, container: HTMLElement) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
    alternate: currentRoot,
  };
  deletions = [];
  /**
   我们只需要将nextUnitOfWork置为根Fiber节点，nextUnitOfWork && !shouldYield
   的判断条件就会满足，workloop方法就会开始构建Fiber Tree并最终生成DOM树
  */
  nextUnitOfWork = wipRoot;
}
```
**Commit 阶段**  
Fiber Tree 构建完成后，nextUnitOfWork 会变为 null，Render 阶段结束，开始执行 commitRoot 方法：用 WIP Fiber Tree 及其对应的 DOM Tree 来替换掉 Current Fiber Tree 及其对应的 DOM Tree。  
具体的 commitRoot 方法直接对应源码进行解析：  
``` 
function commitRoot() {
  // 执行删操作
  deletions.forEach(commitWork);
  // 执行增和改操作
  commitWork(wipRoot.child);
  // 操作完DOM后，用WIP Fiber Tree替换掉Current Fiber Tree
  currentRoot = wipRoot;
  // 将WIP Fiber Tree置空
  wipRoot = null;
}

// 进行具体的增删改操作
function commitWork(fiber: Fiber) {
  // 递归出口
  if (!fiber) {
    return;
  }

  let domParentFiber = fiber.parent;
  // 如果父Fiber没有dom，则找到第一个有dom的祖先Fiber节点，参考图
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent;
  }
  const domParent = domParentFiber.dom;

  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom !== null
  ) {
    // 这一步的dom是在Render阶段就建好了，所以只需插入
    domParent.appendChild(fiber.dom);
  } else if (
    fiber.effectTag === "UPDATE" &&
    fiber.dom !== null
  ) {
    // 更新DOM的具体逻辑
    updateDom(
      fiber.dom,
      fiber.alternate.props,
      fiber.props
    );
  } else if (fiber.effectTag === "DELETION") {
    commitDeletion(fiber, domParent);
  }
  // dfs模式进行递归
  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

function commitDeletion(fiber: Fiber, domParent: HTMLElement) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom);
  } else {
    // 与上面相似的逻辑，没有dom就对第一个有dom的子Fiber进行删除操作
    commitDeletion(fiber.child, domParent);
  }
}
const isEvent = (key: string) => key.startsWith('on');
const isProperty = (key: string) => key !== 'children' && !isEvent(key);
const isNew =
  (prev: Record<string, any>, next: Record<string, any>) => (key: string) =>
    prev[key] !== next[key];
const isGone =
  (prev: Record<string, any>, next: Record<string, any>) => (key: string) =>
    !(key in next);

function updateDom(
  dom: HTMLElement,
  prevProps: Record<string, any>,
  nextProps: Record<string, any>
) {
  // 移除旧的监听事件
  Object.keys(prevProps)
    .filter(isEvent)
    .filter(key => !(key in nextProps) || isNew(prevProps, nextProps)(key))
    .forEach(name => {
      const eventType = name.toLowerCase().substring(2);
      dom.removeEventListener(eventType, prevProps[name]);
    });

  // 增加新的监听事件
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      const eventType = name.toLowerCase().substring(2);
      dom.addEventListener(eventType, nextProps[name]);
    });

  // 移除旧的DOM properties
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach(name => {
      dom[name] = '';
    });

  // 增加新的的DOM properties
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name];
    });
}
```

至此，所有的 DOM 的增、删、改操作都已完成，但是还待完善：  
1. 只支持 Host Component。Didact 最终的目标是支持 Host Component 和 Function Component
2. 只有 render 函数执行第一次渲染 —— mount，而没有提供 update 的入口。接下来一小节中，实现 State Hook 功能，将会为我们提供 setState 函数作为 update 的入口

**Function Component 与 State Hook**  
_无状态的 Function Component_  
Function Component 实质上就是一个函数，它的 Fiber 节点的 type 就是该函数，我们通过执行这个函数来拿到它返回的 elements，也就是它的子 elements，要实现 Function Component，只需要在这里做工作，我们可以直接看源码：  
``` 
let wipFiber: Fiber | null = null;
let hookIndex: number | null = null;

function updateFunctionComponent(fiber: Fiber) {
  wipFiber = fiber;
  // 用数组来存储hooks，在React中用链表，下一小节会用到
  hookIndex = 0;
  wipFiber.hooks = [];
  // 与Host Componenet区别之一就是children需要通过执行函数获得
  const children = [fiber.type(fiber.props)];
  reconcileChildren(fiber, children);
}
```

_实现 useState_  
useState 实现原理其实很简单：
- 执行 setState 导致重渲染
- 重渲染的时候，会再次执行 Function Component 函数
- 执行函数时，会执行 useState，拿到最新的 state，return 的 JSX 中使用了最新的 state
- state 的流转过程：JSX -> Element -> Fiber -> DOM，最终视图刷新

直接看 Didact 源码：  
``` 
// 此hooks在执行updateFunctionComponent的第10行执行FC的函数时执行
function useState<T>(initial?: T) {
  // 取之前旧树上的对应hook（通过hookIndex保证顺序）
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex];
  // state初值为上一次的state或者初值（第一次调用时）
  // queue用来存储所有的setState动作以便进行批处理
  const hook = {
    state: oldHook ? oldHook.state : initial,
    queue: [],
  };
  // 更新state
  const actions = oldHook ? oldHook.queue : [];
  actions.forEach(action => {
    hook.state = action(hook.state);
  });
  // 执行此函数后，会触发重新构建Fiber树
  const setState = (action: (value: T) => T) => {
    // 上一步中执行的就是action方法，此处会将其推入queue
    hook.queue.push(action);
    // 执行更新，做的工作和render方法相似，因此setState是update的入口
    wipRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    };
    nextUnitOfWork = wipRoot;
    deletions = [];
  };
  // 执行后将该hook推入新树的hooks数组
  wipFiber.hooks.push(hook);
  // 为处理下一个hook作准备
  hookIndex++;
  // 注意此时返回了一个函数setState
  // setState用到了函数中的局部变量hook，因此形成了一个闭包
  return [hook.state, setState];
}
```
1. 初次挂载时，第 15 行开始没有任何 action 执行
2. 一切就绪后，初次挂载时的 WIP Fiber Tree 变成了 Current Fiber Tree，而且 hooks 都已经已经挂载到了对应的 Function Component Fiber 节点上
3. 触发 setState，就会触发构建一棵新的 WIP Fiber Tree，此时便会：
   1. 执行 Function Component 的函数本身，理所当然地又执行了 useState；
   2. 第 4 行从 Function Component 对应的 Current Fiber 节点上拿到的上一次的 hook；
   3. 第 15 行开始执行我们在刚才 setState 第 22 行存储的所有 action 来计算出最新的 state 值
   4. 最后还是同样的逻辑，将[state, setState]返回出去，此时的 state 已经是最新值


原文:  
[How to build your own React: Didact 解析](https://mp.weixin.qq.com/s/EL3Qu6SB9pct4d_vF7XVtQ)
