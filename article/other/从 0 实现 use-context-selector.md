# 从 0 实现 use-context-selector
## React context 的问题
在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但此种用法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。  
当 Context Provider 接收的 value 发生变化的时候，React 会向下深度优先遍历组件树，找到消费了该 Context 的组件并标志为需要更新，在组件更新的 render 阶段，这些消费了该 Context 的组件就会重新渲染，读取到最新的 Context 值。  
通常传递给 Context Provider 的 value 是一个对象，对象里包含多个字段，然而这种常见的场景却可能导致多次不必要的重复渲染。以下述代码为例：  
``` 
import { createContext, useState, useContext } from "react";

const Context = createContext(null);

// 使用 Context.Provider 将 count1 和 count2 传递到子孙组件
const StateProvider = ({ children }) => {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);
  return (
    <Context.Provider
      value={{
        count1,
        setCount1,
        count2,
        setCount2,
      }}
    >
      {children}
    </Context.Provider>
  );
};

// 只用到 count1 和 setCount1
const Counter1 = () => {
  const { count1, setCount1 } = useContext(Context);
  return (
    <div>
      <span>count1: {count1}</span>
      <button onClick={() => setCount1((n) => n + 1)}>add count1</button>
    </div>
  );
};

// 只用到 count2 和 setCount2
const Counter2 = () => {
  const { count2, setCount2 } = useContext(Context);
  return (
    <div>
      <span>count2: {count2}</span>
      <button onClick={() => setCount2((n) => n + 1)}>add count2</button>
    </div>
  );
};

const App = () => {
  return (
    <StateProvider>
      <Counter1 />
      <Counter2 />
    </StateProvider>
  );
};

export default App;
```
在上面的例子中，通过 Provider 的 value 向下传递了一个对象，对象中包含 count1、setCount1、count2、setCount2，Counter1 组件只用到了其中的 count1 和 setCount1，Counter2 组件只用到了其中的 count2 和 setCount2。  
但很不幸运的是，count1 发生变化，Counter2 组件会意外地重新渲染，count2 发生变化，Counter1 组件也会意外地重新渲染。  
这是因为每一次 count1 或 count2 的改变，都会引起 StateProvider 重新渲染，那么传到 Provider 的 value 就会是全新的对象，在 React 内部通过 Object.is 对比新旧 value 时，发现 value 发生变化，就会向下深度优先遍历组件树，找到 Counter1 和 Counter2 组件并将它们标志为需要重新渲染。虽然最终渲染结果不会有错误，但是 render 阶段却做了多余的重复渲染和 reconcile（协调，可以理解为 diff）。  
## 实现 use-context-selector
为了解决上述问题，社区中出现了一个解决方案：use-context-selector[1]。它可以让我们从 context value 中选择你会用到的状态，且只有在这些被选择的状态更新时，才会使组件重新渲染。  
使用 use-context-selector 改造一下上面的例子：  
``` 
import { useState } from "react";
+ import { createContext, useContextSelector } from 'use-context-selector';

// 注意这里用的是 use-context-selector 的 createContext
const Context = createContext(null);

// 使用 Context.Provider 将 count1 和 count2 传递到子孙组件
const StateProvider = ({ children }) => {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);
  return (
    <Context.Provider
      value={{
        count1,
        setCount1,
        count2,
        setCount2,
      }}
    >
      {children}
    </Context.Provider>
  );
};

const Counter1 = () => {
- const { count1, setCount1 } = useContext(Context);
+ const count1 = useContextSelector(Context, v => v.count1);
+ const setCount1 = useContextSelector(Context, v => v.setCount1);
  return (
    <div>
      <span>count1: {count1}</span>
      <button onClick={() => setCount1((n) => n + 1)}>add count1</button>
    </div>
  );
};

const Counter2 = () => {
- const { count2, setCount2 } = useContext(Context);
+ const count2 = useContextSelector(Context, v => v.count2);
+ const setCount2 = useContextSelector(Context, v => v.setCount2);
  return (
    <div>
      <span>count2: {count2}</span>
      <button onClick={() => setCount2((n) => n + 1)}>add count2</button>
    </div>
  );
};

const App = () => {
  return (
    <StateProvider>
      <Counter1 />
      <Counter2 />
    </StateProvider>
  );
};

export default App;
```
使用 use-context-selector 改造我们的示例代码之后，count1 的变化只会导致 Counter1 组件的重新渲染，count2 的变化也只会导致 Counter2 组件的重新渲染。  

**具体实现**  
createContext
``` 
import { createContext as createContextOrig, useLayoutEffect, useRef } from "react";

const createProvider = (ProviderOrig) => {
  const ContextProvider = ({ value, children }) => {
    const contextValue = useRef();
    if (!contextValue.current) {
      const listeners = new Set();
      contextValue.current = {
        value,
        listeners,
      };
    }
    useLayoutEffect(() => {
      contextValue.current.value = value;
      contextValue.current.listeners.forEach((listener) => {
        listener({ v: value });
      });
    }, [value]);
    return <ProviderOrig value={contextValue.current}>{children}</ProviderOrig>;
  };

  return ContextProvider;
};

function createContext(defaultValue) {
  const context = createContextOrig({
    value: defaultValue,
    listeners: new Set(),
  });
  context.Provider = createProvider(context.Provider);
  delete context.Consumer;
  return context;
}
```
需要保证传给 Context Provider 的 value 对象地址不变，这样在 React 内部做新旧 value 比较的时候（通过 Object.is）才能得出 value 无变化的结果（避免消费者组件的意外更新）  
代码里重写了 context 对象上默认的 Provider 组件，在我们的自定义 Provider 组件中，通过 useRef 创建了 contextValue，并在首次渲染时给 contextValue 赋一个初始对象，后续就不再更改 contextValue 的引用了。在 contextValue 对象中，包含了最新的 value 和 listeners。  
在自定义 Provider 接收到新的 value 时，更新 contextValue 内部的 value 属性，同时调用所有 listeners，并将最新的 value 传给每一个 listener。  
_useContextSelector_  
``` 
import { useContext as useContextOrig, useLayoutEffect, useReducer } from "react";

function useContextSelector(context, selector) {
  const contextValue = useContextOrig(context);
  const { value, listeners } = contextValue;

  const selected = selector(value);

  const [state, dispatch] = useReducer(
    (prev, action) => {
      const { v } = action;
      if (Object.is(prev[0], v)) {
        return prev;
      }
      const nextSelected = selector(v);
      if (Object.is(prev[1], nextSelected)) {
        return prev;
      }
      return [v, nextSelected];
    },
    [value, selected]
  );

  useLayoutEffect(() => {
    listeners.add(dispatch);
    return () => {
      listeners.delete(dispatch);
    };
  }, [listeners]);

  return selected;
}
```
1. 通过 React 原生的 useContext，拿到了 contextValue 对象上的 value 和 listeners
2. 在每一次渲染中，根据最新的 selector 和 value 计算出选择值 selected
3. 运行 useReducer 得到 dispatch 函数，并将它添加到 listeners 中
4. 如果 value 发生变化，就会执行 listeners 收集到的所有 dispatch 函数，并将最新的 value 作为参数传给 dispatch 函数，dispatch 触发 reducer 的内部逻辑，对比 value 和 选择值 selected 有无变化。在没有变化的情况下，返回上一次的 state，state 相同，React 就不会触发当前组件的重新渲染。

但 use-context-selector 不是完美的，它也有一些问题：  
_问题一_  
当 selector 返回一个包含多个字段的对象时，useContextSelector 的表现和 React 原生的 useContext 表现几乎一样，即 contextValue.current.value 发生变化，始终导致该消费者组件重新渲染。  
比如将 Counter1 组件的代码更改为：  
``` 
const Counter1 = () => {
const { count1, setCount1 } = useContextSelector(Context, v => ({
  count1: v.count1,
  setCount1: v.setCount1
});
  return (
    <div>
      <span>count1: {count1}</span>
      <button onClick={() => setCount1((n) => n + 1)}>add count1</button>
    </div>
  );
};
```
更改后的代码更符合编码习惯，因为我们只需要调用一次 useContextSelector 就得到了 count1 和 setCount1  
但很不幸的是，每次调用 selector 返回的都是一个新对象，在 useContextSelector 内部通过 Object.is 进行新旧 selected 值的对比时，始终会得到新旧 selected 值不相同的结果，因此，每次 contextValue.current.value 发生变化都会引发当前组件的重新渲染。  
_问题二_  
假如一个组件通过 useContextSelector 选择了 A、B 两个字段，但根据组件里的某个内部状态，实际上只用到了 A 字段，在这种情况下，B 字段的值其实不会影响组件的渲染结果，所以合理情况下，B 字段的变化不应该导致组件的重新渲染。然而在实际情况中，B 字段发生变化，仍然会导致组件重新渲染。  
为了说明这个问题，可以将 Counter1 组件改为这样：  
``` 
const Counter1 = () => {
  const count1 = useContextSelector(Context, v => v.count1);
  const setCount1 = useContextSelector(Context, v => v.setCount1);
+ const [showCount1, setShowCount1] = useState(false);
  return (
    <div>
-     <span>count1: {count1}</span>
+     {showCount1 && <span>count1: {count1}</span>}
      <button onClick={() => setCount1((n) => n + 1)}>add count1</button>
+     <button onClick={() => setShowCount1(v => !v)}>toggle count1 display</button>
    </div>
  );
};
```
上面的代码中，showCount1 默认是 false，即默认不展示 count1，所以 count1 的值根本不影响渲染结果。但是在这种情况下，count1 的改变还是会导致 Counter1 组件的重新渲染。  
_问题三_  
当在 Counter1 和 Counter2 组件之间来回点击 add count1 和 add count2 按钮，即使每次点击只更改 count1 或 count2，但 Counter1 和 Counter2 组件都会重新渲染。  
在 React 中，一个组件其实会对应两个 fiber，一个保存当前视图对应的相关信息，称为 current fiber；一个保存接下来要变化的视图对应的相关信息，称为 wip fiber。  
当组件触发更新后，会在组件对应的两个 fiber 上都标记需要更新。当组件 render 完成后，会把 wip fiber 上的更新标记清除。当视图完成渲染后，current fiber 与 wip fiber 会交换位置（也就是说本次更新的 wip fiber 会变为下次更新的 current fiber）  
当我们第一次点击 add count1 的时候，Counter1 组件对应 current fiber 和 wip fiber 同时标记更新。组件渲染完成后，wip fiber 的更新标记被清除，但此时 current fiber 还存在更新标记。完成渲染后，current fiber 和 wip fiber 会互换位置。此时变成了：wip fiber 存在更新，current fiber 不存在更新。  
当点击 add count2 的时候，由于 Counter1 组件的 wip fiber 存在更新，所以即使本次没有修改 count1，但 Counter1 组件仍然会重新渲染，就出现了 Counter1 和 Counter2 组件同时重新渲染的情况。  
_问题四_  
useContextSelector 的表现和 React 原生的 useContext 表现几乎一样，即 contextValue.current.value 发生变化，始终导致所有消费者组件重新渲染  
在 useContextSelector 中，用到了 useReducer。在 React 17 中，调用 useReducer 返回的 dispatch 函数时，假如当前组件不存在其他更新，那么本次 dispatch 调用就是组件的第一个更新，在触发重新渲染之前，React 会通过传入 useReducer 的 reducer 函数立刻算出最新的状态，然后和当前状态进行比较，如果两者一致，则跳过该组件的渲染更新。  
``` 
import { useContext } from 'react';
import { useContextSelector } from 'use-context-selector';

// 在 React 18 中，以下两个实现方式的表现都是一样的

const Counter1 = () => {
 const { count1, setCount1 } = useContext(Context);
  return (
    <div>
      <span>count1: {count1}</span>
      <button onClick={() => setCount1((n) => n + 1)}>add count1</button>
    </div>
  );
};

const Counter1 = () => {
  const count1 = useContextSelector(Context, v => v.count1);
  const setCount1 = useContextSelector(Context, v => v.setCount1);
  return (
    <div>
      <span>count1: {count1}</span>
      <button onClick={() => setCount1((n) => n + 1)}>add count1</button>
    </div>
  );
};
```
**优化版**  
- 对于问题一，可以给 useContextSelector 增加第三个参数 equalityFn，该参数默认是 shallowEqual，也就是说，默认情况下，会对新旧 selected 值做浅比较，避免了 useContextSelector 返回对象时的性能问题  
- 对于问题二，暂时无解，需要使用类似 Vue 的 Object.defineProperty 或 Proxy 等劫持/代理方案，才能知道 selected 值有无被使用
- 对于问题三，这种现象只会在使用了 useState 或 useReducer 的情况下才会出现，那么就不依赖这两个 hook 了。参考 react-redux 中 useSelector 的实现，可以使用 React 18 的新 hook：useSyncExternalStore。在 React 17 中，可以使用 use-sync-external-store这个 npm 包，它是 useSyncExternalStore 的向后兼容垫片。  
- 对于问题四，在改为使用 useSyncExternalStore 后，也不再依赖 useReducer，自然就没有这个问题了

使用 useSyncExternalStore 实现了监听浏览器网络访问状态的功能  
``` 
import { useSyncExternalStore } from 'react';

// 一个可以展示当前网络状态的组件
export default function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return <h1>{isOnline ? 'Online' : 'Disconnected'}</h1>;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```
在 subscribe 函数中，监听了 online 和 offline 事件（当 navigator.onLine 更改时，浏览器会触发 window 对象上的online和offline事件）  
subscribe 函数的调用时机：  
``` 
useEffect(() => {
  const handleStoreChange = () => {/**...*/}
  return subscribe(handleStoreChange)
}, [subscribe])
```
online 和 offline 事件处理函数就是 handleStoreChange，该函数会根据 getSnapshot 算出最新的 snapshot 值，和上一次算出的 snapshot 值做对比，如果发生了变化，则触发更新。在重新渲染时，useSyncExternalStore 就会返回最新的 snapshot 值，即最新的 navigator.onLine 值。  
use-context-selector 的优化版：  
``` 
import {
  createContext as createContextOrig,
  useContext as useContextOrig,
  useLayoutEffect,
  useRef,
  useCallback,
  useSyncExternalStore,
} from "react";
import shallowEqual from "shallowequal";

const createProvider = (ProviderOrig) => {
  const ContextProvider = ({ value, children }) => {
    const contextValue = useRef();
    if (!contextValue.current) {
      const listeners = new Set();
      contextValue.current = {
        value,
        listeners,
      };
    }
    useLayoutEffect(() => {
      contextValue.current.value = value;
      contextValue.current.listeners.forEach((listener) => {
        // 这里不同了，不再需要给 listener 传入参数
        listener();
      });
    }, [value]);
    return <ProviderOrig value={contextValue.current}>{children}</ProviderOrig>;
  };

  return ContextProvider;
};

function createContext(defaultValue) {
  const context = createContextOrig({
    value: defaultValue,
    listeners: new Set(),
  });
  context.Provider = createProvider(context.Provider);
  delete context.Consumer;
  return context;
}

// 基于 useSyncExternalStore 实现 useContextSelector
function useContextSelector(context, selector, equalityFn = shallowEqual) {
  const contextValue = useContextOrig(context);
  const { value, listeners } = contextValue;

  const subscribe = useCallback(
    (callback) => {
      listeners.add(callback);
      return () => listeners.delete(callback);
    },
    [listeners]
  );

  const lastSnapshot = useRef(selector(value));

  const getSnapshot = () => {
    const nextSnapshot = selector(contextValue.value);

    if (equalityFn(lastSnapshot.current, nextSnapshot)) {
      return lastSnapshot.current;
    }

    lastSnapshot.current = nextSnapshot;
    return nextSnapshot;
  };

  return useSyncExternalStore(subscribe, getSnapshot);
}
```



原文:  
[从 0 实现 use-context-selector](https://mp.weixin.qq.com/s/vp7Jfxh7AmseD6L1NVpvZA)
