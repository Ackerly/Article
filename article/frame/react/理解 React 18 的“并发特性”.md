# 理解 React 18 的“并发特性”
React v18.0 引入了一项期待已久的新功能——并发模式  
## 什么是 React 并发模式
React 并发的基本原理是重构渲染过程，使 在渲染下一个视图时，当前视图保持响应性。并发模式是 React 团队提升应用性能的一个提案。它的想法是将渲染过程分成可中断的工作单元。  
在幕后，这将通过在`requestIdleCallback()`调用中包装组件渲染来实现，让应用在渲染过程中保持响应性。  
如果将类似如下的“阻塞模式”实现：  
``` 
function renderBlocking(Component) {
    for (let Child of Component) {
        renderBlocking(Child);
    }
}
```
将会这样实现“并发模式”：  
``` 
function renderConcurrent(Component) {
    // 如果状态已经过时，打断渲染进程
    if (isCancelled) return;

    for (let Child of Component) {
        // 等待浏览器不忙（没有要处理的输入）
        requestIdleCallback(() => renderConcurrent(Child));
    }
}
```
在最初使用requestIdleCallback之后，React 切换到了 requestAnimationFrame，之后又切换到了用户空间计时器。  
**没有模式，只有特性**  
由于向后兼容性的原因，并发模式并没有实现。React 团队转向了并发特性，一组新的 API，用于选择性地启用并发渲染。到目前为止，React 已经引入了两个新的 hook 来实现并发渲染。  

_useTransition_  
useTransition hook 返回两个元素：  
1. 布尔标志 isPending，如果并发渲染正在进行则为 true
2. 函数 startTransition，用于分发一个新的并发渲染

为了使用它，请在 setState 调用中使用 startTransition 回调   
``` 
function MyCounter() {
    const [isPending, startTransition] = useTransition();
    const [count, setCount] = useState(0);
    const increment = useCallback(() => {
        startTransition(() => {
            // 并发运行这个更新操作
            setCount(count => count + 1);
        });
    }, []);

    return (
        <>
            <button onClick={increment}>Count {count}</button>
            <span>{isPending ? "Pending" : "Not Pending"}</span>
            // 受益于并发特性的组件
            <ManySlowComponents count={count} />
        </>
    )
}
```

> 有 200 多个 Input 框，在 Non-Transition 的情况下，点击 Count 按钮后会感觉到 UI 界面明显有卡顿，而 Transition 模式则非常流畅。

从概念上讲，状态更新检测它们是否被包装在 startTransition 中，以决定是安排阻塞渲染还是并发渲染。  
```
 function startTransition(stateUpdates) {
  isInsideTransition = true;
  stateUpdates();
  isInsideTransition = false;
}

function setState() {
  if (isInsideTransition) {
    // 安排并发渲染
  } else {
    // 安排阻塞渲染
  }
}
```
useTransition 的一个重要警告是，它不能用于受控输入。对于这些情况，最好使用 useDeferredValue。  
**useDeferredValue**  
useDeferredValue hook 是一个便捷的 hook，适用于你没有机会将状态更新包装在 startTransition 中，但仍希望并发运行更新的情况。  
其中一个例子是，子组件从父组件接收一个新值。从概念上看，useDeferredValue 是一个防抖动效果，可以实现如下:  
``` 
function useDeferredValue<T>(value: T) {
    const [isPending, startTransition] = useTransition();
    const [state, setState] = useState(value);
    useEffect(() => {
        // 当输入更改时，分发并发渲染
        startTransition(() => {
            setState(value);
        });
    }, [value])

    return state;
}
```
它的使用方式与输入防抖动 hook 相同：  
``` 
function Child({ value }) {
    const deferredValue = useDeferredValue(value);
    // ...
}
```

**并发特性和 Suspense**  
useTransition 和 useDeferredValue hook 除了可选择启用并发渲染之外，还有一个作用是等待 Suspense 组件完成。  




原文:  
[理解 React 18 的“并发特性”，就这么简单](https://mp.weixin.qq.com/s/PhIGvHTwiIIspyktA_PWHw)
