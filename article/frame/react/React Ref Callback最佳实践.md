# React Ref Callback：最佳实践
## ref callback
ref 属性除了接受 ref 对象之外，还可以接受函数也就是 ref callback。在该函数中，DOM 元素作为其唯一参数。  
与 effect 函数一样，React 在组件周期中的某些时刻中调用它。当创建 DOM 元素之后会立即执行 ref callback（参数是 DOM 元素），在删除元素时也会再次调用 ref callback，只不过这时的参数是 null。  
如果 ref callback 被定义为内联函数，React 将在每次渲染时调用它两次，第一次的参数是 null，第二次的参数是 DOM 元素。  
虽然内联 ref callback 被调用两次可能会令人惊讶，如果从 React 的角度来看，这种行为是合理的。它保证了一致性，因为每次渲染都会创建新的函数实例，它可能是一个完全不同的函数。这些函数可能会依赖 props 或 state，而这些 props 或 state 也可能在此期间发生了变化。  
React 需要清除旧的 ref callback（参数是 null），然后设置新的回调（参数是 DOM 元素）。这样可以根据条件来设置 ref 属性的值，甚至在 React 元素之间交换它们。  
这可能会导致一些不必要的调用。在大多数情况下，这不是引起什么问题。如果你不想执行这些不必要的调用，可以通过在 useCallback 中包装 ref callback 或将函数移出组件来避免这种行为。  
## 使用场景
只有在访问底层的 DOM 元素时，才需要 ref callback。那么，ref callback 何时有用？答案是，当您想要在 React 中对 DOM 元素执行操作时，请使用 ref callback。这可以归结为以下四种情况。  
- 当元素挂载或更新时，调用 DOM 元素上的方法来执行一些操作
- “通知” DOM 元素的更改，当 DOM 元素的某些属性发生更改时，重新读取该元素
- 将 DOM 元素设置到 state 中，以便在渲染期间访问它
- 共享 DOM 元素；使用 DOM 元素执行多项操作

**DOM 元素挂载并滚动到它所在的位置**  
可以在 ref callback 中调用 DOM 元素上的方法，以执行滚动或聚焦等 DOM 操作。例如，自动滚动到列表中的最后一项：  
``` 
// On first render and on unmount there
 // is no DOM element so `el` will be `null`
 const scrollTo = (el) => {
   if (el) {
     el.scrollIntoView({ behavior: "smooth" });
   }
 };

 function List({ data }) {
   return (
     <ul>
       {data.map((d, i) => {
         const isLast = i === data.length - 1;
         return (
           <li
             key={d.name}
             // ref callback to scroll to the last list element
             ref={isLast ? scrollTo : undefined}
           >
             {d.name}
           </li>
         );
       })}
     </ul>
   );
 }
```
管理 DOM 是 React 的工作，避免执行 DOM 上的可变方法， 比如（insert, remove, set, replace 等），对于 focus 和 scroll 等非破坏性的操作则允许我们开发实现。  
**当 DOM 元素变化时的重新渲染**  
通过 React 访问某些 DOM 元素属性时，可以使用 ref callback。当我们读取一个 DOM 元素属性，比如滚动位置，或者在 ref callback 中调用一个获取元素信息的方法，比如 getBoundingClientRect()，并将该信息设置到 state 中  
_测量 DOM 元素_  
``` 
const [size, setSize] = useState();

 const measureRef = useCallback((node) => {
   setSize(node.getBoundingClientRect());
 }, []);

 return <div ref={measureRef}>{children}</div>;
```
没有选择使用 useRef，因为当 ref 是一个对象时，它并不会把当前 ref 值的变化情况通知到我们。使用 callback ref 可以确保即便被测量的节点在子组件延迟显示 (比如为了响应一次点击)，我们依然能够在父组件接收到相关的信息，以便更新测量结果。注意到我们传递了 [] 作为 useCallback 的依赖列表。这确保了 ref callback 不会在再次渲染时改变，因此 React 不会在非必要的时候调用它。  
_在 render 中访问 DOM 元素_  
如果在 ref 回调中将 DOM 元素设置到 state，它将触发新的渲染，因为这正是设置 state 的作用。但是它不会陷入无限渲染循环，因为 setState 是一个稳定的函数，因此 ref callback 仅在挂载和卸载时调用。  
不使用 useRef？答案是，因为不允许在渲染过程中访问 ref 对象。对于渲染中的 DOM 元素，必须通过 state 来访问。接下来举几个例子进行介绍。  

**React Portal**  
React portal 主要用于解决组件树和 DOM 树的结构之间不一致的问题。portal 将 DOM 树上不同位置上的组件连接到一起，最为常使用的场景就是将 Modal 弹窗覆盖整个视窗。  
``` 
// Assume an empty div with id 'modal' is in your HTML
 const modalEl = document.getElementById("modal");

 function Modal({ children, ...props }) {
   return ReactDOM.createPortal(
     <ModalBase {...props}>
       {children}
     </ModalBase>,
     modalEl
   );
 }
```
可以使用 document.getElementById() 来获取 DOM 元素，前提是你能保证它是存在的。或许你不想通过 HTML 来控制 Modal，而是希望能 portal 到一个 React 创建的 DOM 元素上。  
这就需要在进行 render 时访问到相应的 DOM 元素，使用 ref callback 可以实现这个功能。  
``` 
function Parent() {
   const [modalElement, setModalElement] = useState(null);

   return (
     <div>
       <div id="modal-location" ref={setModalElement} />
       {/* Imagine that the modal container and the
           Modal itself are farther apart in the component tree */}
       <Modal modalElement={modalElement}>Warning</Modal>
     </div>
   )
 }

 function Modal({ children, modalElement, ...props }) {
   return modalElement
     ? ReactDOM.createPortal(
         <ModalBase {...props}>{children}</ModalBase>,
         modalElement
       )
     : null;
```
modalElement 的值是 null，所以需要在创建 portal 之前做一下判断。  

**非受控复合组件**  
非受控复合组件（Uncontrolled Compound Components）是一种高级的 React 模式，其核心是 ref callback 来处理 React portal。  
复合组件（Compound Component）是将多个组件组合到一起工作，进而形成一个能够展示的 UI。复合组件将复杂的功能拆分为更小的块，并且它们在一起共同完成整个复杂功能。这样就可以避免产生一个有很多 props 的 “上帝组件”。  
这种模式与 HTML 元素组合比较类似，比如：  
- <select> 中包含多个 <option>
- <table> 组件会由 <thead> 和 <tbody> 组成
- <details> 元素中会包含 <summary>

对于一些共享组件，如对话框或侧边栏，页面上只能有一个，提升 state 会使这些 state “过于全局”。这些 state 将通过许多中间组件连接起来，而这些中间组件实际上并不需要知道它。它会污染整个链条上的组件，并会使代码变得混乱。  
可以把 React state 看作是悬挂在组件树上的绳子。绳子的长度代表了定义 state 的组件到使用 state 的 UI 之间的距离。所以，当你的绳子长度越长、数量越多时，它们就越容易被缠在一起。所以要尽量缩短绳子的长度，同时控制绳子的数量。  
强行将组件树和数据流适配成 DOM 树的形状。反过来，我们可以通过组件树适应数据流来反转控制。使用 Portal，我们可以重置组件间的距离，并保持 DOM 结构不变。这样就可以将拉近相关组件的距离，即便是在 DOM 树中的离得很远。这使我们的 React 代码更容易理解。  
非受控复合组件的实现过程：  
- ref callback 定位到元素位置。获取 DOM 元素并将其置于 state 中。这里我们不能使用 ref 对象，因为我们需要在 render 中使用它，而且要在设置它的时候触发更新。
- 用 Context 共享元素位置。将这个 DOM 元素放存放在 context 中，以便 context 中的所有组件都可以访问 DOM 中的这个位置。
- 使用 Portal 连接到该位置。从 context 中获取对 DOM 元素，并将组件进行 portal

**共享 DOM Ref**  
经常会出现不止一个消费者需要访问 DOM 元素。假设你想测量一个 <div> 的宽度，并将其交给另一个 React 之外的库来处理 DOM 内容。对于 React 来说，这样的元素就是一个黑盒。React 既不知道它的内部有什么，也不关心它是什么。这个元素完全交给另外一个库来管理。  
``` 
import useMeasure from "react-use-measure";
 import * as Plot from "@observablehq/plot";

 export function BoxPlot({ data }) {
   const [measureRef, { width, height }] = useMeasure({ debounce: 5 });

   const plotRef = useRef<HTMLDivElement | null>(null);

   useEffect(() => {
     const boxPlot = Plot.plot({
       width: Math.max(150, width),
       marks: [
         Plot.boxX(data),
       ],
     });

     plotRef.current.append(boxPlot);

     return () => boxPlot.remove();
   }, [data, width]);

   const initBoxPlot = useCallback((el: HTMLDivElement | null) => {
     plotRef.current = el;
     measureRef(el);
   }, []);

   return <div ref={initBoxPlot} />;
 }
```


原文:  
[React Ref Callback：最佳实践](https://mp.weixin.qq.com/s/qW3L4y_kmQISRJPOvFlMEw)
