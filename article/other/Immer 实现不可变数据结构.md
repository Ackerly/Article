# Immer 实现不可变数据结构
为什么在写 React的时候，总是强调不可变数据，有时一些复杂的，嵌套层级深的数据，在写法上也要兼顾，层层解构和组合，以此来保证不可变数据。  
在 redux 时代，reducer 当中最常看到的就是通过 数组 concat 方法 返回新状态，Array.prototype.concat 方法确实也可以实现不可变数据结构，下面这个例子，可以看到 concat 方法可以 满足不可变数据，返回新对象，尽可能高性能的复用内存。  
``` 
const user = { name: "cyan" };
const userList = [user];
const newUserList = userList.concat({ gender: "men" });

console.log(newUserList[0] === user); // true
console.log(newUserList === userList); // false
```
为什么不用 cancat，为什么用 Immer 呢?数据结构复杂的时候，比如下面这种情况，concat 就显得有些捉襟见肘。还需要通过解构，组合，如果是数据结构再复杂一点呢？ 所以 Immer 出现了。  
``` 
const user = {
  list: [
    {
      name: "cyan",
    },
  ],
};
```
## 为什么需要不可变数据
### 从 React 角度来讲  
第一：React UI 更新的原则就是 Immutable， 试想你在修改状态的时候, 直接将 state.push("balabala"), 然后 setState(state), React UI 会更新吗？答案是否定的。因为 React State 在 Diff 更新的时候，是通过 shallowEqual 去比较，比较结果是 false 才去更新，如果你是可变数据的，即便数组的内容已经 通过 push 方法改变了。但数组的地址不会变，比较结果还是 true，React 对视图不会进行更新。  
第二：有了不可变数据的依赖，React 程序才知道什么时候，或者应不应该需要优化组件渲染和昂贵计算，什么时候，或者应不应该更新视图。  

### 从不可变数据 本身来讲  
不可变数据 本身会为应用程序带来：可预测性的程序，高性能的程序，允许状态追踪的程序。
**可预测性**  
可变数据虽然方便，但是屏蔽或者说隐藏了改变带来了副作用，比如，你直接向数组中 push 一个元素，用起来是很舒服，但是别忘了，整个应用程序，任何一个地方都可以修改这个数组，而且无法预测这个数组在应用程序的哪个地方，进行了什么样的操作，这样一来，便会造成难以排查的 bug，反观不可变数据，任何一次操作，都会返回一个全新的数据结构，这样一来，针对该数据结构的控制会变得简单容易理解方便维护，比如借助第三方工具 redux，你完全可以知道你在哪个地方，进行了什么样的操作，让数据最后变成了什么样子。  
**高性能**  
尽管向不可变对象添加值意味着需要创建一个新实例，其中需要复制现有值，并且需要将新值添加到新对象中，这会消耗内存，但 使用 Immer 或者 Immutable.js 创建的不可变对象可以最大幅度的利用结构共享来减少内存  

**状态可追踪**  
用过 redux 开发者工具的同伴都知道，每一次 dispatch，在应用程序的哪个地方，触发了什么样的 action，最终得到了什么的结果，在 redux 工具当中一目了然。可变数据无法做到这样的可追踪。  
useMemo 等一系列钩子优化组件和计算，也可以归结为不可变数据的状态的可追踪，有了不可变数据的依赖，React 程序 才知道什么时候，或者应不应该需要优化组件渲染和昂贵计算，什么时候，或者应不应该更新视图。  

_Immer 不可变数据结构 性能究竟好在哪了？？_  
面这种情况，通过 user.list...修改 name 值，很抱歉，Immer帮不了你，Immer 再处理的时候，还是会层层创建新对象，这种情况和深克隆差不多。  
``` 
const user = {
  list: {
    student: {
      s1: {
        cyan: {
          name: "cyan",
        },
      },
    },
  },
};
```
Immer 什么时候能帮助你? 如果是以下的情况 Immer 就开始发光了。比如你向 user.list 通过 immer push 一个新数组，或者修改了 每一个user.list[0].name, Immer 就会帮助你，怎么帮，Immer 只会创建一个 list[0], 为新对象，其他 list [i] 的数据结构内存都会复用，其次 Immer 为了实现不可变数据，也会递归改变发生变化元素的父元素  

**Immer 解决了什么问题？**  
解决了在 React 当中强调的不可变数据 (immutable) 的性能问题，可以不使用深拷贝 这样浪费性能的操作来完成这一过程。  
mmer 原理：Immer 通过 递归式的 proxy 对象代理 和 浅拷贝，提高了不可变数据的性能，尽可能的复用数据结构当中其他节点的内存。既满足了性能要求，又使得数据达到了不可变数据的要求。  

**Immer 的基本使用**  
``` 
import { produce } from "./index.js";
const obj = { age: 17 };
const baseState = { name: { age: 10 }, list: [obj, 1], gender:"men" };

const newState = produce(baseState, (draft) => {
  draft.gender = "men";
  draft.name.age = 18;
  draft.list.push(4);
});
```
从 immer 源码角度 来解释上面的过程  
draft.gender = “men”  
immer.produce 方法会给 baseState 通过 proxy 搭建一层拦截（通过 createProxy ，createProxyProxy 函数 )并且创建一个 state（draft_：Object.assign 浅拷贝之后对象, base_:源对象, modified） 对象，然后执行我们传入的第二个回调函数，当我们执行 draft.gender = “men” 的时候，会来到 拦截的 set 方法当中，这个方法会将 state 对象的 modified_ 属性变为 true, 表示该对象已经改变，然后将 state 当中的 gender 变为 新值，然后推出之后，会判断是否 mutated，如果改变了，返回 Object.assign() 浅拷贝之后的 draft，作为一个新的对象返回。  

draft.name.age = 18  
immer.produce 方法会给 baseState 通过 proxy 搭建一层拦截（createProxy），( createProxyProxy )并且创建一个 state（draft_：Object.assign 浅拷贝之后对象, base_:源对象, modified） 对象，然后执行我们传入的第二个回调函数，当我们执行 draft.name 的时候,会来到拦截的 get 方法当中，将 draft.name 取出来之后，发现是对象，递归执行 createProxyProxy 在创建 state（draft_：Object.assign 浅拷贝之后对象{age: 17}, base_:源对象 {age: 18}, modified）对象，注册修改父亲的回调函数，然后执行 draft.name.age = 18，会来到 set 方法，这个方法会将 state 对象的 modified_ 属性变为 true, 表示该对象已经改变，然后将 state 当中的 age 变为 新值。返回浅拷贝之后新对象的 draft，调用改变父引用的回调函数，上回溯将 draft 对象返回。  
``` 
### Immer 源码实现
import { isArray, isFunction, isObject } from "./is.js";

let INTERNAL = Symbol("INTERNAL");

export function produce(baseState, producer) {
  let proxyState = toProxy(baseState);
  producer(proxyState);
  const internal = proxyState[INTERNAL];
  return internal.mutated ? internal.draftState : internal.baseState;
}

function createDraftState(baseState) {
  if (isObject(baseState)) {
    return Object.assign({}, baseState);
  } else if (isArray(baseState)) {
    return [...baseState];
  } else {
    return baseState;
  }
}

export function toProxy(baseState, callParentCopy) {
  let keyToProxy = {};
  let internal = {
    /* 浅拷贝 指向 baseState */
    draftState: createDraftState(baseState),
    keyToProxy,
    mutated: false, //是否变更
    baseState,
  };

  return new Proxy(baseState, {
    get(target, key) {
      debugger;
      if (key === INTERNAL) {
        return internal;
      }
      let value = target[key];
      if (isObject(value) || isArray(value)) {
        if (key in keyToProxy) {
          return keyToProxy[key];
        } else {
          keyToProxy[key] = toProxy(value, () => {
            internal.mutated = true;
            const proxyChild = keyToProxy[key];
            let { draftState } = proxyChild[INTERNAL];
						// 递归改变父引用
            internal.draftState[key] = draftState;
            callParentCopy && callParentCopy();
          });
        }
        return keyToProxy[key];
      } else if (isFunction(value)) {
        internal.mutated = true;
        callParentCopy && callParentCopy();
        return value.bind(internal.draftState);
      }

      return internal.mutated
        ? internal.draftState[key]
        : internal.baseState[key];
    },

    set(t, key, value) {
      debugger;
      internal.mutated = true;
      const { draftState } = internal;
      draftState[key] = value;
      callParentCopy && callParentCopy();
      return true;
    },
  });
}

const isArray = (val) => Array.isArray(val);

const isObject = (val) =>
  Object.prototype.toString.call(val) === "[object Object]";

const isFunction = (val) => typeof val === "function";

export { isArray, isFunction, isObject };
```

**Immutablejs 是什么**  
和 Immer 解决了 一样的问题，但是 Immer 和 React hook 可以 更加天然的结合，使用更加方便，Immutablejs 实现了一套自己的 API 和 数据结构，和使用 Immer 比较起来，稍显复杂。  
Immutablejs 提供多种数据结构 List Set Map Stack Queue，并且方便的对数据结构/复杂嵌套的数据结构进行 immutable 操作，push slice 不会直接改变对象，而是返回一个新的不可变对象。让不可变的方式变得更加方便。  

原文:  
[来聊聊 Immer 实现不可变数据结构](https://juejin.cn/post/7177987710836015162)
