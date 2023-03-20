# 可在控制流中调用--react 新hook
## 介绍
**前置条件**  
这个新 Hook 一般来说得和 Suspense 一起使用，因此，需要准备这样的 wrapper 组件，用以包裹下文提到的绝大多数组件。  
``` 
function LoadingErrorWrapper({ children }) {
  return <ErrorBoundary>
    <Suspense fallback={<Loading />}>
      {children}
    </Suspense>
  </ErrorBoundary>
}
```
在实际情况中，这个组件可能还包含错误上报、错误恢复、防止 Loading 闪烁等一系列功能  

**最简例子**  
``` 
function Note({id}) {
  // This fetches a note asynchronously, but to the component author it looks
  // like a synchronous operation.
  const note = use(fetchNote(id));
  return (
    <div>
      <h1>{note.title}</h1>
      <section>{note.body}</section>
    </div>
  );
}
```
熟悉 useSwr 的同学可能发现了，这样的调用和 useSwr 非常相似，同样都是把获取数据的函数传递给 Hook。有点不同的是，React 官方的版本计划接收一个 Promise 而非一个返回 Promise 的函数。  

**缓存 Promise**  
在 useSwr 等库中，有 cacheKey 的概念，当 cacheKey 发生改变，则重新发起查询，最终引起 rerender。  
在 use 中，我们需要怎么实现对应的功能呢？目前，官方的回答是，使用 useMemo 缓存 Promise，当 Promise 发生了改变，则相当于之前说的 cacheKey 发生了改变。  
``` 
function Note({id}) {
  // return new promise when id changes
  const fetchNotePromise = useMemo(() => fetchNote(id), [id]);
  const note = use(fetchNotePromise);
  return (
    <div>
      <h1>{note.title}</h1>
      <section>{note.body}</section>
    </div>
  );
}
```
这样就可以实现，当 id 改变时，重新执行 fetchNode 这个功能了。  
不过，官方认为，似乎可以有更好的方式来实现相应的功能，所以也许会有新的提案补充进来。  
useSwr 都已经实现了，就是这个 hook 独一无二的特性——可以在控制流中调用，也就是 if、switch、for 等表达式。

## 设计动机
**无缝支持 JS 生态**  
React 在设计之初就和 JS 本身结合得很好，JSX 相较于模版语言而言通常被认为有更好的灵活性和可读性。  
React 对于 Promise 一直没有很好的支持，如果需要在 useEffect 中调用，可能还需要包括一层 IIFE，因为 useEffect 的入参函数不能以 Promise 作为返回值，这在社区中还引起了吐槽  
那么这次的新 Hook 因为接收 Promise 作为参数，所以就能够很好地弥补刚刚说的问题，从而能够更好地支持 JS 生态。  

**避免客户端组件与服务端组件过于相似**  
在 SSR 提出之初，React 和社区认为，服务端组件和客户端组件应该是一回事，甚至应该在服务端模拟一个 DOM 来抹平相关差异。  
但是随着开发经验的积累，大家发现，两者最好还是有些区别，不然很可能发生一些问题，比如在开发组件时漏了它作为服务端组件的场景，但是在之后被意外地当成服务端组件调用，然后报错崩溃。  
因此，React 团队建议，同样是获取并展示数据的场景，服务端组件应该使用 Async Component，而客户端组件可以选用本文讲述的新 Hook——use  
**鼓励提前请求和缓存**  
众所周知，Promise 在构造的时候就会调用里面的函数了，它并没有采用函数式中的懒加载的概念。  
而使用use 分为两步，第一步构造 Promise，第二步把结果传递给 use。  
第一步和第二步没有必要放在一起调用，可以把第一步放到状态管理库里，也可以把第二步放到控制流里。  
当提前调用第一步，而需要展示的时候再调用第二步的时候，就是“提前请求”，React 团队认为这能够极大地提升用户体验，减少用户等待时间。  
当 Promise 改变时，use 会在其 resolve 后 rerender 组件，因此要求开发者使用 useMemo 等形式缓存 Promise 或其返回值，这就是 React 团队希望的，鼓励开发者进行“缓存”。  

**更加抽象**  
考虑到请求是一件非常常见且抽象的事情，React 团队一直想把它集成到 React 中去。  
但是抽象是有成本的，盲目的抽象可能会增加许多约定，限制可能性与灵活性，甚至变成历史债。  
但是不能因噎废食，如果设计得好，考虑得周全，那么把请求集成到 React 中去当然是一件美事。  

**编译期优化**  
自推出 Hook 起，社区就有讨论，是否应该把 Hook 的第二个参数 dependency 交给编译器去做，当然最终结论是通过 eslint 插件的形式，把选择权放到每一位开发者手中  
不过编译期优化的潜力还是巨大的，它可以节约不必要的运行时开销。因此在这个提案中，React 团队又重新把这个概念搬了出来，期望之后他们能交出完美的答卷  

## 限制
尽管 use 无视了很多 React Hook 的规则，比如其他 Hook 不能在控制流中调用，而 use 可以，但是它也必须遵循以下两个规则：  
**必须在 React 组件中调用**  
因为 use 还是会引起 rerender 的，所以它必须和一个组件绑定，也就是必须在 React 组件中调用，这也是为什么 React 团队仍然把它称之为 Hook 的理由  

**父函数必须是 React 组件或 Hook**  
提案指出，use 位于的那个函数必须是 React 组件，并给出了以下的例子：  
``` 
function ItemsWithForLoop() {
  const items = [];
  for (const id of ids) {
    // ✅ This works! The parent function is a component.
    const data = use(fetchThing(id));
    items.push(<Item key={id} data={data} />);
  }
  return items;
}

function ItemsWithMap() {
  return ids.map((id) => {
    // ❌ The parent closure is not a component or Hook!
    // This will cause a compiler error.
    const data = use(fetchThing(id));
    return <Item key={id} data={data} />;
  });
}
```

## 具体实现
虽然官方还没有编写代码，但是在提案中已经对具体实现进行了拆解。  
因为 Promise 本身没有成员变量来获取状态，因此考虑在 Promise 上挂载 status、value、reason 三个字段，分别代表当前 Promise 的状态、resolve 后的 value，reject 后的错误。  
当然，这个挂载行为只会针对传入 use 的 Promise，而不会污染全局的原型链。  
因此，代码大致如下：  
``` 
function use(promise) {
  if(promise.status === 'rejected') {
    // 由 ErrorBoundary 承接错误
    throw promise.reason;
  }
  if(promise.status === 'fulfilled') {
    // 正常返回
    return promise.value
  }
  promise.status = 'pending';
  // 抛出 promise，由 Suspense 处理。
  // 也是一个新的 Promise 进入会走的流程。
  throw promise.then(value => {
    promise.status = 'fulfilled';
    promise.value = value;
    tryRerender();
  }).catch(reason => {
    promise.status = 'rejected';
    promise.reason = reason;
  })
}
```
第 16 行有 tryRerender()，之所以需要尝试“重新渲染”，是因为 Suspense 似乎会加上类似于 debounce 的功能。如果没有这个功能，在 Promise 以非常快的速度 resolve 时，会发生“闪屏”现象，而加上 debounce 会有效防止“闪屏”的发生，有效提高用户体验。但是这个功能的存在就意味着这个 Hook 就必须读取 fiber，就必须像其他 Hook 一样和 React 组件绑定了。  
而之所以是“尝试”重新渲染，是因为这是一个异步行为，此刻组件可能因为被 Suspense 等情况而卸载了，所以需要防御式编程下。  
因为它只依赖 filber 提供的重新渲染能力，而不用像其他 Hook 一样往 filber 上挂东西，因此，它可以随意在控制流中调用。  

## 争议
Phil KarIton 说，“计算机科学只存在两个难题：缓存失效和命名。”  
官方认为，因为它还是像其他 Hook 一样依赖 fiber，所以必须在运行时和一个具体的 React 组件绑定，因此它一定是个 Hook，换言之，得以 “use” 作为前缀。从另一方面来说，它的限制与其他 Hook 不同，其中最大的不同是可以在控制流中随意调用，这赋予了它无与伦比的灵活性和可能性，因此官方希望找到一个独一无二的名字来把它和其他“普通” Hook 区分开。综上，思来想去，不如就用 “use” 本身作为名字吧。  

原文:  
[可在控制流中调用！React 新 hook 尝鲜](https://mp.weixin.qq.com/s/j6I-LR9ck_HFkrlHVImvRg)
