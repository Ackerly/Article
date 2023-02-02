# React技巧之Suspense
Suspense 直译为悬念，推迟应用程序的呈现，直到数据已获取并准备好显示。React 不是一次渲染整个应用程序，而是为每个等待数据的组件提供一个占位符。 当所有的组件都准备好渲染或者你访问相关的路由时，页面会立马展示。  

## 方法
Suspense 的作用与 ErrorBounary 相似。ErrorBounary 用于包装可能抛出错误的组件。 当子组件抛出错误（例如网络请求失败）时，切换呈现显示自定义错误 UI。Suspense 则是当子组件抛出 Promise 时，切换呈现的加载 UI。它们都基于 JavaScirpt 的 throw 语法，将 Suspense 和 ErrorBounary 可视化  

**Suspense Use Cases**  
React 仅推荐在生产环境中使用 Suspense 结合 React.lazy 动态加载组件。用户请求较大的 JavaScript 时，加载时间决定了页面展示的快慢。尤其是在弱网条件下，代码拆分和延迟加载非常有用。Suspense 和 React.lazy 用于在用户加载延迟时显示有用的加载状态。React.lazy 允许动态 import 组件，通过 webpack 的 Require.ensure 功能实现按需加载组件，组件加载过程中会返回一个 Promise。  
通过 React.lazy 的方式引入三个组件，Card、Comment 和 Label。为了看到明显的效果，将开发者工具的网络设置为 slow 3G。  
``` 
// App.js
import React, { Suspense } from 'react';
import { BrowserRouter as Router, Switch, Route, Link } from 'react-router-dom';
​
const Card = React.lazy(() => import('./components/Card'));
const Comment = React.lazy(() => import('./components/Comment'));
const Label = React.lazy(() => import('./components/Label'));
​
function App() {
  return (
    <Router basename="/">
      <ul>
        <li>
          <Link to="/">Home</Link>
        </li>
        <li>
          <Link to="/label">About</Link>
        </li>
        <li>
          <Link to="/comment">Dashboard</Link>
        </li>
      </ul>
​
      <hr />
      <Suspense fallback={<p>Loading component...</p>}>
        <Switch>
          <Route path="/" exact component={Card} />
          <Route path="/comment" component={Comment} />
          <Route path="/label" component={Label} />
        </Switch>
      </Suspense>
    </Router>
  );
}
​
export default App;
```
Suspense 接受一个 fallback 组件，它允许将任何 React 组件显示为自定义加载状态。通过 React.lazy 和 Suspense 可以实现代码分割，避免因体积过大而导致加载时间过长。在点击跳转页面后，会发起一个请求获取代码。  
## Suspense Source Code and Mechanism
React 介绍 Suspense 的工作流程：  
1. 在 render 方法中，从缓存中读取一个值；
2. 如果该值已被缓存，则返回该值；
3. 如果该值尚未缓存，缓存抛出一个 Promise；
4. 当 Promise 状态变为 resolved，回到步骤1

React.lazy 将普通的组件变成一个可缓存的组件（lazyComponent），Suspense 实现了 React 从缓存中读取，等待读取成功后展示。根据以上四步结合源码分析：  
**1.从缓存中读取**
``` 
let Component = readLazyComponentType(elementType);
```
**2 & 3. 已经被缓存了 ? 返回值 : 抛出 Promise**  
``` 
export function readLazyComponentType<T>(lazyComponent: LazyComponent<T>): T {
  initializeLazyComponentType(lazyComponent);
  if (lazyComponent._status !== Resolved) {
    throw lazyComponent._result;
  }
  return lazyComponent._result;
}
```
React 在这一步 throw Promise，并通过 ctor 捕获：  
``` 
export const Uninitialized = -1;
export const Pending = 0;
export const Resolved = 1;
export const Rejected = 2;
​
export function initializeLazyComponentType(
  lazyComponent: LazyComponent<any>,
): void {
  if (lazyComponent._status === Uninitialized) {
    lazyComponent._status = Pending;
    const ctor = lazyComponent._ctor;
    const thenable = ctor();
    lazyComponent._result = thenable;
    thenable.then(
      moduleObject => {
        if (lazyComponent._status === Pending) {
          const defaultExport = moduleObject.default;
          // dev only 
          lazyComponent._status = Resolved;
          lazyComponent._result = defaultExport;
        }
      },
      error => {
        if (lazyComponent._status === Pending) {
          lazyComponent._status = Rejected;
          lazyComponent._result = error;
        }
      },
    );
  }
}
```
React.lazy 导入一个 Thenable 的组件  
``` 
export function lazy<T, R>(ctor: () => Thenable<T, R>): LazyComponent<T> {
  let lazyType = {
    $$typeof: REACT_LAZY_TYPE,
    _ctor: ctor,
    // default: Uninitialized
    _status: -1,
    _result: null,
  };
  // dev code
  return lazyType;
}
```
**4.当 Promise 状态变为 resolved，React 重试**  
``` 
import React from 'react';
​
class CommonFallBack extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      pending: null
    }
  }
  componentDidCatch(err) {
    console.log("catch!")
    if (this.props.children.type ) {
      const thenable = this.props.children.type._result;
      this.setState({pending : thenable}, () => {
        thenable.then( (res) => {
          this.setState({ pending: null});
        }) 
      });
    }
  }
  render() {
    return this.state.pending ? <div>Loading because of catch</div> : this.props.children;
  }
}
​
export default CommonFallBack;
```

## Future
Suspense 的另一个应用是，根据数据请求的状态来展示自定义加载文案。因为 React 还在实验中，所以我们不展开讨论。Suspense 的路线图，React 实验 Suspense 用于数据请求  


原文:  
[React 技巧: Suspense](https://juejin.cn/post/7191879317482111037)
