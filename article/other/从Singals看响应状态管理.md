# 从 Signals 看响应式状态管理
Preact 引入了 Signals，提供了快速的响应式状态原语（或者叫原子吧），那么 Signals 有以下几点：  
- 感觉上像是使用原始数据结构
- 能根据值的变化自动更新
- 直接更新 DOM （换句话来说无 VDOM）
- 没有依赖数组

Signals 的用法：  
```
 import { signal } from "@preact/signals";
 const count = signal(0);

 function Counter() {
   const value = count.value;

   const increment = () => {
     count.value++;
   }

   return (
     <div>
       <p>Count: {value}</p>
       <button onClick={increment}>click me</button>
     </div>
   );
 }
```
跟 SolidJS 的 createSignal 非常相似，而且两者有很多共同点（下面再说），另外通过.value 访问属性非常类似于 Vue 中的 Ref。Signals 可以在一个应用从小到大，在越来越复杂的逻辑迭代后，依然能保证性能。Singals 提供了细粒度状态管理的好处，而无需通过 memorize 或者其他 tricks 方式去优化，Signals 跳过了数据在组件树中的传递，而是直接更新所引用的组件。这样开发者就能降低使用心智，保证性能最佳。  
hooks state 作为原子和 Signals 的性能比对, Signals 有以下几点：
- 默认惰性求值（lazy evaluate）- 只有被使用到的才会被监听和更新
- 最佳更新策略
- 最佳依赖追踪策略 - 不像 hooks 需要指定依赖
- 直接访问状态值，不需要 selector 或其他 hooks
  
 Signals 是可以独立于组件外的，跟 hooks 方式不一样,Signals 可以单独品尝使用，不用依附 UI 框架。

**其他框架**  
SolidJS 响应式数据例子：  
```
import { createSignal, onCleanup } from "solid-js";
 import { render } from "solid-js/web";

 const App = () => {
   const [count, setCount] = createSignal(0);
   const timer = setInterval(() => setCount(count() + 1), 1000);
   onCleanup(() => clearInterval(timer));
   return <div>{count()}</div>;
 };

 render(() => <App />, document.getElementById("app"));
```
SolidJS 响应式也是也 Signal 作为基础，createSignal 既可以用于组件内，也可以用于组件外，这个跟 Preact 中类似。一方面可以将 Signal 作为组件的 local state，也可以定义为 global State。SolidJS 中也有以下相似点：  
- 响应式细粒度更新
- 无需定义 dependencies
- 惰性取值

SoidJS 与 Mobx 和 Vue 的响应式非常相似，但是不会处理 VDOM，而是直接更新 DOM。所以 SolidJS 的性能表现也比较不错  

**响应式状态管理三要素**  
_信号: Signals_  
表示一个响应式数据的单元  
```
 const [count, setCount] = createSignal(0);

 console.log(count()); //0

 setCount(5);
 console.log(count()); //5
```
这里取值用的是 function，有些地方用的是 .value，意味着也可以通过 Object 的 getter, setter 或者 Proxy 去进行数据处理  

_反应: Reactions_  
分地方叫 Effect ，也就是副作用，当然也有用 actions 的，下方是一个基本例子：  
``` 
const [count, setCount] = createSignal(0);
 createEffect(() => console.log("The count is", count()));
 setCount(5);
```
反应也就是在数据更新时的监听器，作为响应式数据的基础，也是必不可少的一环  

_衍生: Derivations_  
这里是指数据的衍生状态，本质上也可以认为是 Signals 的变种，常见命令可能有 computed, memo 等  
``` 
const [firstName, setFirstName] = createSignal("John");
 const [lastName, setLastName] = createSignal("Smith");

 const fullName = createMemo(() => {
   return `${firstName()} ${lastName()}`
 });

 console.log(fullName);
```
衍生能缓存计算结果，避免重复的计算，并且也能自动追踪依赖以及同步更新  

_响应式特点_  
响应式数据管理会存储不同节点之间的链接关系，当每次节点更新之后，会重新检查链接关系。如果不在关联，就会解绑链接，取消依赖。  
下方的例子更能体现：  
``` 
const [firstName, setFirstName] = createSignal("a");
 const [lastName, setLastName] = createSignal("b");
 const [showFullName, setShowFullName] = createSignal(true);

 const displayName = createMemo(() => {
   if (!showFullName()) return firstName();
   return `${firstName()} ${lastName()}`
 });

 createEffect(() => console.log("名称：", displayName()));
 // a b
 setShowFullName(false);
 // a
 setLastName("c");
 // nothging change
 setShowFullName(true);
 // a c
```
另外响应式还有一点就是同步更新，同步更新避免了状态不一致的问题,的预测性和可测试性。在响应式数据更新的基础上，有些也会加入比如批量更新，批量更新在避免重复执行反应和衍生上大有好处，大大避免了一些多余额外的执行消耗  

**手动实现一个**  
响应式状态管理核心还是用的观察者模式，当 Signals 更新时，Reactions 会订阅到数据变化从而更新数据。  
_Signals_  
实现一个基础的数据更新与读取  
``` 
const createSignal = (value) => {
   const setter = (newValue) => value = newValue;
   return [() => value, setter]
 }

 const [name, setName] = createSignal('a')
 console.log(name());
 setName('b')
 console.log(name());
```
加上订阅逻辑，重新更改下：  
``` 
// context 包含Reactions中的执行方法和Signal依赖
 const context = [];

 const createSignal = (value) => {
   const subscriptions = new Set();
   const readFn = () => {
     const running = context.pop();
     if (running) {
       subscriptions.add({
         execute: running.execute
       });
       running.deps.add(subscriptions);
     }
     return value;
   };
   const writeFn = (newValue) => {
     value = newValue;
     for (const sub of [...subscriptions]) {
       sub.execute();
     }
   };
   return [readFn, writeFn];
 };

 const [name, setName] = createSignal("a");
 console.log(name());
 setName("b");
 console.log(name());
```
在读取的时候会获取当前执行的上下文，拿到 Reactions 的方法，并且方法依赖里增加当前 Signals，这样 Reactions 就能订阅到这个 Signals，当 Signals 更新时，会执行所包含的订阅方法。  
_Reactions_  
``` 
const createEffect = (fn) => {
   const execute = () => {
     context.push(running);
     fn();
     context.pop(running);
   }

   const running = {
     execute,
     deps: new Set()
   }
   execute();
 }
 const [name, setName] = createSignal("a");
 createEffect(() => console.log(name()))
 setName("b");
```
随着 Reactions 每次执行，running 的 deps 会逐步累加，所以需要在执行前，清空 deps。  
``` 
const createEffect = (fn) => {
   const execute = () => {
     running.deps.clear();
     context.push(running);
     try {
       fn();
     } finally {
       context.pop(running);
     }
   };

   const running = {
     execute,
     deps: new Set()
   };
   execute();
 };
```
_Derivations_  
``` 
const createMemo = (fn) => {
   const [memo, setMemo] = createSignal();
   createEffect(() => setMemo(fn()));
   return memo;
 }
```
衍生是一种特殊的 Signals，所以直接返回 Signal，另外 Reactions 是可以追踪订阅到 Signals 的变化，所以在 Reactions 函数里设置 Derivations 的值就可以了。  

**回到 React**  
valtio 更符合 Signals 在 React 中的实现  
- 无需 dependency
- 外部 store，无需关联组件

作为 React 相关的框架，最后都会去做 VDOM 更新，跟 SolidJS 直接更新不一样，当然我们也期望 React 官方能做一些改变去优化现有的开发体验，比如：React Forget/useEvent

原文:  
[从 Signals 看响应式状态管理](https://mp.weixin.qq.com/s/Yi7ujuAY6TIE_H3pDYnzwA)
