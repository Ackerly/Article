# 从一次富文本编辑器升级探索React元素渲染
**背景**  
内容后台早期使用的富文本编辑器为 wangEditor v4，按照一般的开发经验来说，对于富文本编辑器这一类的第三方工具升级，除了对工具的 API 进行必要的功能开发之外，通常不会有其他的影响。  
**二级标题富文本编辑器升级暴露的问题**  
内容后台的图文内容结构比较特殊，一篇内容中可以插入多正文内容，也就是说在图文编辑的页面中可能同时会存在多个富文本编辑器。在升级完富文本编辑器之后，在页面中添加多个正文内容（即插入多个富文本编辑器，一个富文本编辑器对应一个正文内容），此时，如果选中第一个富文本编辑器，则后面创建的富文本编辑器都会从页面中消失。  
内容后台页面中，使用的是 React Hook，也是 React 官方推荐的使用方式。其中正文内容为一个组件，其中维护一个名为 contentList 的数据，用于表示当前具有的内容。在正文内容的层级之下，还有富文本编辑器组件与图片上传组件等子组件。富文本编辑器组件在监听到 change 事件时，会通过回调函数的方式，向父组件，也就是正文内容组件传递信息，同时在正文内容组件中对 contentList 数据进行维护。  
为什么使用 v4 版本富文本编辑器时没有遇到这个问题，而使用 v5 版本时会遇到呢？  
经过验证，在进行上述操作时，v5 版富文本编辑器在焦点变换的时候，不但会触发 focus/blur 事件，同时还会触发 change 事件，而在组件中并没有对 focus 和 blur 事件进行监听，也就是说 change 事件的触发会导致出现上面的问题。当然，只是触发富文本编辑器的 change 事件还不够，另外一个条件是，当页面存在多个富文本编辑器时，必须要触发前面的富文本编辑器的 change 事件。  
由于 v4 版本富文本编辑器仅仅在对内容进行编辑时才会触发 change 事件，而运营同学的习惯往往又是正向顺序编辑操作  
**问题的原因是什么**  
先来执行过程的第一步，点击“添加正文”按钮，此时页面会在正文内容组件中创建一个富文本编辑器子组件。页面在打开后经过了若干次渲染，而正文内容组件则被渲染了 6 次。点击按钮后，由于更新了 contentList 数据，正文内容组件又进行了几次渲染，其中创建第一个富文本编辑器发生在第 7 次渲染过程中。同时，第一个富文本编辑器在创建完成之后触发了一次 change 事件，并调用了正文内容组件的回调函数，而且渲染次数与创建时的渲染次数是相同的。  
接下来创建第二个富文本编辑器。同样地，更新过 contentList 数据之后，正文内容组件也进行了几次渲染，而第二个富文本组件的创建则发生在第 11 次渲染过程之中。与创建第一个富文本编辑器一样，第二个富文本编辑器在创建完毕之后，正文内容组件的回调函数也被调用了一次。  
这时，将光标移入第一个富文本编辑器中，触发它的 change 事件。所触发的正文内容组件回调函数，仍旧是第一个富文本编辑器第一次创建时所在的渲染过程，也就是在第 7 次渲染过程。
由于正文内容组件中富文本编辑器的个数是由 contentList 数据来控制的，显而易见地，在创建一个富文本编辑器后，contentList 中有了一条数据；创建第二个富文本编辑器后，contentList 中有了两条数据；在触发第一个富文本编辑器的 change 事件后，contentList 中又只剩下了一条数据，这时页面中仅存在一个富文本编辑器了。  
**React 元素渲染的特点**  
在 React 的官方文档中，可以看到对元素渲染更新的说明：  
React 元素是不可变对象。一旦被创建，你就无法更改它的子元素或者属性。一个元素就像电影的单帧：它代表了某个特定时刻的 UI。  
React DOM 会将元素和它的子元素与它们之前的状态进行比较，并只会进行必要的更新来使 DOM 达到预期的状态。  
React 渲染机制的特点就是：每一帧都拥有独立的状态。如何理解这个特点，先来看一个 React 的例子：  
``` 
function Parent() {
  const [count, setCount] = setState(0);

  const tick = () => {
    setCount(count + 1);
  };

  setInterval(tick, 1000);

  return (
    <div>
      <Child count={count} />
    </div>
  );
}


function Child(props) {
  const { count } = props;

  const handleClick = () => {
    setTimeout(() => {
      alert(count);
    }, 5000);
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleClick}>alert button</button>
    </div>
  );
}
```
这段代码中，创建了一个名为 Parent 的函数组件和一个名为 Child 的函数组件，其中 Child 组件的 count 属性由 Parent 组件传入，初始值为 0，每隔一秒增加 1。点击 Child 组件中的“alert count”按钮，将延迟 5 秒弹出 count 的值。实际操作后会发现，弹窗中出现的值，与页面中展示的 count 值并不相同，而是等于点击按钮那一时刻 count 的值。  
由于 Child 是函数组件，在每一次渲染时，都会接收一个 props 参数，这个 props 是函数作用域下的变量。当 Child 组件被创建时，执行类似如下的代码完成一次渲染：  
``` 
const props_0 = { count: 0 };

const handleClick_0 = () => {
  setTimeout(() => {
    alert(props_0.count);
  }, 5000);
};


return (
  <div>
    <p>{props_0.count}</p>
    <button onClick={handleClick_0}>alert count</button>
  </div>
);
```
当 Parent 组件传入的 count 变为 1，React 会再次调用 Child 函数，执行第二次渲染，这个时候 count 的值是 1：  
``` 
const props_1 = { count: 1 };

const handleClick_1 = () => {
  setTimeout(() => {
    alert(props_1.count);
  }, 5000);
};

return (
  <div>
    <p>{props_1.count}</p>
    <button onClick={handleClick_1}>alert count</button>
  </div>
);
```
由于 props 是 Child 函数作用域下的变量，可以说对于这个函数的每一次调用，都产生了新的 props 变量，它在声明时被赋予了当前的属性，他们相互间互不影响。  
由于 props 是 Child 函数作用域下的变量，可以说对于这个函数的每一次调用，都产生了新的 props 变量，它在声明时被赋予了当前的属性，他们相互间互不影响。  
例如，在第 1 秒的时候点击“alert count”按钮，此时 props.count 的值为 1，handleClick 函数中的 count 的值也为 1。当时间到达第 6 秒时，props.count 的值变为了 6，此时刚刚执行的定时器回调函数开始执行，闭包中的 count 的值仍旧为 1，页面弹窗中出现的值就是 1。  
**如何解决这个问题**  
由于富文本编辑器为了提升视图渲染的稳定性，引入了虚拟 DOM 技术，而虚拟 DOM 技术往往是通过对比两次数据的异同来判断是否需要更新视图的，由此可以断定，正是在 React 中使用了虚拟 DOM 技术，导致了富文本编辑器在页面多次渲染之后仍保留了创建时的状态。  
要解决上述问题，需要引入 useRef。useRef 通常有两种作用：一是作为多次渲染之间的纽带；二是获取 DOM 元素。这里我们只需要 useRef 的第一种作用。我们先来看看 useRef 在 React 返回值的类型定义:  
``` 
interface MutableRefObject<T> {
  current: T;
}
```
可以看到 useRef 返回值是一个包括属性 current 类型为范型 的一个object。它与直接在函数组件中定义一个 { current: null } 的区别就是：useRef会在所有的render中保持对返回值的唯一引用。因为所有对 ref 的赋值和取值拿到的都是最终的状态，并不会因为不同的render而存在不同的隔离。也就是说，我们可以把useRef的返回值想象成一个全局变量。  
改写一些上面的代码：  
``` 
function Child(props) {
  const { count } = props;
  const countRef = useRef(count);

  const handleClick = () => {
    setTimeout(() => {
      alert(countRef.current);
    }, 5000);
  };

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleClick}>alert button</button>
    </div>
  );
}
```
此时再按照之前的步骤去操作，会发现弹窗展示出的值就是最新的值，与页面是保持一致的，而并非是渲染隔离的值。  
在我们的内容后台中使用 useRef，也能确保正文内容组件中 contentList 始终保持最新，在多次触发 change 事件之后，不会有页面异常的情况出现了  

参考:  
[从一次富文本编辑器升级探索React元素渲染](https://mp.weixin.qq.com/s/AaweMKSxnK5lVkBMtTYwNQ)
