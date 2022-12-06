# React 数据获取：避免条件竞争
**Promise 简介**  
当 JavaScript 执行代码时，通常是一步一步地同步执行。Promise 是为数不多的异步执行方法之一。有了 Promise，就可以直接触发一个任务并立即进入下一步，而无需等待任务完成。当任务完成后，Promise 会通知我们。  
数据获取是 Promise 最重要和最广泛使用的场景之一。不管是直接使用 fetch 还是使用像 axios 这样的第三方类库，Promise 的行为都是一样的。  
从代码的角度来看，如下所示：  
``` 
console.log('first step'); // will log FIRST

 fetch('/some-url') // create promise here
   .then(() => { // wait for Promise to be done
       // log stuff after the promise is done
       console.log('second step') // will log THIRD (if successful)
     }
   )
   .catch(() => {
     console.log('something bad happened') // will log THIRD (if error happens)
   })

 console.log('third step') // will log SECOND
```
整个过程是：在 fetch('/some-url') 中创建一个 Promise，然后在 .then 和 .catch 中处理后续的事情。  
**Promise 和条件竞争**  
页面的左侧是选项卡 tabs 的列，切换 tabs 就会导航到新页面并发送一个数据请求，请求返回的数据渲染在右侧。如果快速的来回切换 tabs 就会发现这里存在奇怪问题：右侧的内容会闪烁，数据看起来是随机出现的。  
这个问题是怎么发生的？让我们看看页面的实现。页面有两个部分。一个是根 App 组件，它管理 “page” 的状态，并渲染了导航按钮和实际的 Page 组件。  
``` 
const App = () => {
   const [page, setPage] = useState("1");

   return (
     <>
       <!-- left column buttons -->
       <button onClick={() => setPage("1")}>Issue 1</button>
       <button onClick={() => setPage("2")}>Issue 2</button>

       <!-- the actual content -->
       <Page id={page} />
     </div>
   );
 };
```
Page 组件接受 page 的 id 作为属性，发送请求以获取数据，然后渲染数据。Page 组件的实现（没有加载状态）如下所示：  
``` 
 const Page = ({ id }: { id: string }) => {
   const [data, setData] = useState({});

   // pass id to fetch relevant data
   const url = `/some-url/${id}`;

   useEffect(() => {
     fetch(url)
       .then((r) => r.json())
       .then((r) => {
         // save data from fetch request to state
         setData(r);
       });
   }, [url]);

   // render data
   return (
     <>
       <h2>{data.title}</h2>
       <p>{data.description}</p>
     </>
   );
 };
```
通过 id，我们确定从哪个 url 获取数据。然后我们在 useEffect 中发送数据请求，并将结果数据存储在 state 中 —— 这一切都很标准。那么，条件竞争和奇怪问题是什么原因导致的呢？  

**条件竞争的起因**  
这一切归结为两件事：Promise 和 React 生命周期。从生命周期的角度来看，会发生这些事情：  
- App 组件被装载（mounted）
- Page 组件被装载，并默认的 prop 为 “1”
- Page 组件中的 useEffect 首次被执行

然后 Promise 开始执行：useEffect 中的 fetch 是一个 promise 异步操作。它负责发送实际的数据请求，而 React 继续它自己的工作，而不等待结果。大约 2s 后数据请求完成，在 .then 中调用 setData 并将数据保存在 state 中，Page 组件渲染出最新的数据，最终我们在屏幕上看到这些数据。  

如果在渲染和完成所有内容后，单击导航按钮，将看到以下事件：  
- App 组件将其状态更改为另一个 page
- state 触发 App 组件的重新渲染
- Page 组件也将重新渲染
- Page 组件中的 useEffect 依赖于 id；由于 id 已变化，将再次触发 useEffect
- useEffect 将用新 id 触发 fecth 请求，大约 2s 后将再次调用 setData，Page 组件更新，我们在屏幕上看到新数据

如果 id 第一次变化触发的 fetch 尚未返回，这时单击导航按钮切换 tabs 会发生什么？  
- App 组件将再次触发页面的重新渲染
- useEffect 也将再次被触发（id 已发生变化！）
- fetch 将再次被执行，React 将一如既往地继续工作
- 这时第一次数据请求完成。它仍然可以访问到 Page 组件的 setData（组件只是被更新，组件仍然是之前的组件）
- 第一次请求成功后的 setData 将被触发，Page 组件将使用该次请求的数据更新自身
- 第二次请求完成。它仍然在那里，挂在后台，就像任何 Promise 一样。该组件依然使用了同一个 Page 组件的 setData，它将被触发，Page 再次更新自身，这一次使用的是第二次获取的数据。  

条件竞争出现了！点击导航按钮到新 page 后，我们看到了页面内容发生闪烁：首先渲染第一次请求的数据，然后替换成第二次请求的数据。  
如果第二次数据请求在第一次数据请求之前完成，这种效果就更有趣了。我们将首先看到下一页的正确内容，然后它将被上一页的不正确内容替换。  

**解决条件竞争：强制重新挂载**  
严格来说，这并不是一个解决方案，它更多的是用来解释为什么这些竞争条件不会经常发生，以及为什么在常规页面导航中我们碰不到竞争条件。我们用下面的代码来实现上面的功能：  
``` 
const App = () => {
   const [page, setPage] = useState('issue');

   return (
     <>
       {page === 'issue' && <Issue />}
       {page === 'about' && <About />}
     </>
   )
 }
```
没有向下传递 prop，Issue 和 About 组件使用各自的 url 来完成数据请求。在 useEffect 中进行数据获取，这与之前的实现方式完全相同：  
``` 
const About = () => {
   const [about, setAbout] = useState();

   useEffect(() => {
     fetch("/some-url-for-about-page")
       .then((r) => r.json())
       .then((r) => setAbout(r));
   }, []);
   ...
 }
```
这样在 tabs 导航时不会出现竞争条件。无论操作的次数和速度有多快，应用程序都能够正常运行。这是为什么呢？  
答案在这里：{page==='issue'&&<issue/>}。当 page 的值更改时，Issue 和 About 不会重新渲染，而是重新装载。当值从 issue 更改为 about 时，Issue 组件将自行卸载，About 组件将装载在它的位置上。  
从数据获取的角度来看：  
- App 组件首先被渲染，然后装载 Issue 组件并开始获取数据
- 在数据请求过程中导航到下一个页面，App 组件会卸载 Issue 组件并装载 About 组件，然后开始它自己的数据获取

当 React 卸载一个组件时，意味着它已经不存在了。彻底消失了，从屏幕上消失了，没有人可以访问它，里面发生的一切，包括它的状态都丢失了。与前面的代码相比，我们在前面的代码中编写了 ＜Page id＝{Page} /＞，此 Page 组件从未被卸载，我们只是在导航时重新使用它。   
所以回到卸载的场景。当 Issue 的数据请求在 About 组件上完成时，Issue 组件的 .then 将尝试调用其 setIssue 来设置 state。但组件已经不存在了，从 React 的角度来看，它已经不存在。因此，这个 Promise 也将消失，它得到的数据也随之消失。  
在组件消失之后完成异步操作（如数据获取），会出现“无法对未安装的组件执行 React 状态更新” 这个警告。当然，现在这个警告已经被 React 删除掉了。  
无论如何，理论上，这种方法可以用于解决的条件竞争问题：我们所需要做的是在导航变化时强制重新装载 Page 组件。为此，我们可以使用 “key” 属性：  
但是，并不推荐使用这个方案来解决条件竞争问题，它会引起一些潜在问题：可能会使性能受到影响，焦点和状态会出现意外错误，以及触发不必要的 useEffect。它并没有从根本上解决问题，而是将问题掩盖起来。但在某些场景下，如果小心使用，也不失为一种解决问题的方法。  

**解决条件竞争：丢弃错误数据**  
一种解决条件竞争更友好的方法，是确保传入到 .then callback 的数据与当前被 “选中” 的 id 匹配；而不是清除整个已存在的 Page 组件。  
如果返回的数据中包括了用于生成 url 的 “id”，我们可以比较它们是否一致。如果它们不一致，则忽略它们。这里需要做的是，要跳出 React 生命周期和函数的本地作用域，在 useEffect 访问到 “最新” 的 id。React ref 非常适合这个场景：  
``` 
const Page = ({ id }) => {
   // create ref
   const ref = useRef(id);

   useEffect(() => {
     // update ref value with the latest id
     ref.current = id;

     fetch(`/some-data-url/${id}`)
       .then((r) => r.json())
       .then((r) => {
         // compare the latest id with the result
         // only update state if the result actually belongs to that id
         if (ref.current === r.id) {
           setData(r);
         }
       });
   }, [id]);
 }
```
如果返回的数据结果中没有包含 id，我们可以比较 url：  
``` 
const Page = ({ id }) => {
   // create ref
   const ref = useRef(id);

   useEffect(() => {
     // update ref value with the latest url
     ref.current = url;

     fetch(`/some-data-url/${id}`)
       .then((result) => {
         // compare the latest url with the result's url
         // only update state if the result actually belongs to that url
         if (result.url === ref.current) {
           result.json().then((r) => {
             setData(r);
           });
         }
       });
   }, [url]);
 }
```
**解决条件竞争：丢弃之前的数据**  
如果不喜欢上面的解决方案，或者认为使用 ref 来做这样的事情。还有另一种方法：useEffect 有一个叫做 “cleanup” 的清除函数，在这个函数中我们可以清理订阅之类的东西。在我们的场景中可以用来控制数据获取。它的语法如下：  
``` 
// normal useEffect
 useEffect(() => {

   // "cleanup" function - function that is returned in useEffect
   return () => {
     // clean something up here
   }
 // dependency - useEffect will be triggered every time url has changed
 }, [url]);
```
“cleanup” 的函数在组件卸载后执行，或者在每次依赖项变化引起重新渲染之前运行。因此，重新渲染期间的执行顺序如下：  
- url 发生变化
- 触发 “cleanup” 函数
- useEffect 的实际内容被触发

利用 JavaScript 函数和闭包，可写出这样的代码：  
``` 
useEffect(() => {
   // local variable for useEffect's run
   let isActive = true;

   // do fetch here

   return () => {
     // local variable from above
     isActive = false;
   }
 }, [url]);
```
引入了一个局部布尔变量 isActive，并在 useEffect 运行时将其设置为 true，在 “cleanup” 清理时将其设为 false。每次重新渲染时都会重新创建 useEffect 中的函数，因此最近一次 useEffect 运行的 isActive 将始终重置为 true。但是在它之前运行的 “cleanup” 函数仍然可以访问前一个函数的作用域，它会将其重置为 false。这也是 JavaScript 闭包的工作原理。  
fetch Promise 虽然是异步的，但仍然只存在于该闭包中，并且只能访问 useEffect 运行的局部变量。因此，当我们在 .then 回调中检查 isActive 布尔值时，只有在最近的一次运行中，尚未执行清理函数，才会将变量设置为 true。所以现在需要做的只是检查是否处于活动的闭包中，如果是，则设置状态。如果不是，什么都不做，数据将再次凭空消失。  
``` 
 useEffect(() => {
     // set this closure to "active"
     let isActive = true;

     fetch(`/some-data-url/${id}`)
       .then((r) => r.json())
       .then((r) => {
         // if the closure is active - update state
         if (isActive) {
           setData(r);
         }
       });

     return () => {
       // set this closure to not active before next re-render
       isActive = false;
     }
   }, [id]);
```
**解决条件竞争：取消之前的请求**  
可以取消之前的所有请求，而不是清理或比较数据结果。如果它们永远都不会完成数据请求，那么也就永远不会使用这些过时数据，条件竞争的问题也就不复存在。  
可以使用 AbortController，实现的代码也相对比较简单：在 useEffect 中创建 AbortController 并在清理函数中调用 .abort()。  
``` 
useEffect(() => {
     // create controller here
     const controller = new AbortController();

     // pass controller as signal to fetch
     fetch(url, { signal: controller.signal })
       .then((r) => r.json())
       .then((r) => {
         setData(r);
       });

     return () => {
       // abort the request here
       controller.abort();
     };
   }, [url]);
```
在每次重新渲染时，正在进行的请求将被取消，只允许最新的数据请求被解析并更新到 state。  
中止正在进行的请求，会导致 Promise 被 reject，因此我们需要捕获错误以消除控制台中的可怕警告。正确处理 Promise reject 是一个很好的开发习惯，这是任何场景下都应该做的事情，与是否使用 AbortController 无关。由 AbortController 引起的 reject 会抛出特定类型的错误，因此很容易将它与其它常规错误区分开。  
``` 
fetch(url, { signal: controller.signal })
   .then((r) => r.json())
   .then((r) => {
     setData(r);
   })
   .catch((error) => {
     // error because of AbortController
     if (error.name === 'AbortError') {
       // do nothing
     } else {
       // do something, it's a real error!
     }
   });
```
**Async/await 会改变什么吗？**  
Async/await 只是编写 Promise 的语法糖。从执行的角度来看，它们只是转换为 “同步” 函数，但不会改变它们的异步特性。下面的 Promise 代码：  
``` 
 fetch('/some-url')
   .then(r => r.json())
   .then(r => setData(r));
```
与下面 async/await 代码等价：  
``` 
const response = await fetch('/some-url');
 const result = await response.json();
 setData(result);
```
使用 async/await 代替 “传统” 的 Promise，同样会存在条件竞争问题，前面介绍的解决方案也依然适用，只是语法略有差异。


原文:  
[React 数据获取：避免条件竞争](https://mp.weixin.qq.com/s/gTcKgh83lLNGiLTjQGfmcw)
