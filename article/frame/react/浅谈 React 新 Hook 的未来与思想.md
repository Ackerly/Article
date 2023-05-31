# 浅谈 React 新 Hook 的未来与思想
**useContext -> use(Context)**  
尽管这个 Hook 确实可以在控制流中调用，但将其命名为 use 似乎并不合适。既然其入参是 Promise，为何不直接称其为 usePromise 呢？  
Promise 作为入参仅仅是这个 Hook 宏伟蓝图的第一步。根据相关提案和计划，这个 Hook 将接受被称为“usable”的类型作为入参。  
第一个官方实现的“usable”数据结构是 React Context。原先的 useContext(Context) 调用可以等价地替换为 use(Context)。  
这样做的好处是能在控制流中调用。至于缺点，如果组件本身编写得不够规范，不够“纯”，依赖 useContext 进行重渲染，而不是像纯函数那样仅读取并展示数据，那么组件的逻辑和流程可能会受到影响。  

## 消费外部资源的问题
仅仅作为一个可以在控制流中调用的 useContext 仍无法满足 use 的远大抱负。在 React 团队的设想中，所有外部资源，包括非 React 资源，都可以通过 use 被使用。而这才是 React 新 hook 的重点。  
以 rxjs 中的 BehaviorSubject 为例，这是一个非常典型的非 React 资源。下面，我们将通过讲述 useBehaviorSubject 设计演变过程来进行阐述。  
简单的 useBehaviorSubject 示例如下：  
``` 
function useBehaviorSubject(subject) {
  const [, setState] = useState(0)
  
  useEffect(() => {
    const subscription = subject.subscribe(() => {
      // force rerender
      setState(x => x + 1);
    });
    return () => subscription.unsubscribe()
  }, [subject])
  
  // return the value of this subject.
  return subject.getValue()
}
```
这样的问题在于，每次为 subject 发送一个新值时，都会导致重新渲染，即使这个新值和旧值完全相同。对于遵循函数式编程的组件来说，这无疑会导致许多不必要的重绘。  
为了减少无效重绘，一种可选的做法是通过 useState 将值存储在 fiber 结构上。这样，在每次执行 setState 时，通过 React 自带的差异比较（diff）机制，可以避免许多无效的重新渲染。  
``` 
function useBehaviorSubject(subject) {
  const [state, setState] = useState(subject.getValue())
  
  useEffect(() => {
    const subscription = subject.subscribe(() => {
      setState(subject.getValue())
    })
    return () => subscription.unsubscribe();
  }, [subject])
  
  // or return state.
  return subject.getValue()
}
```
在上述代码示例中，如果新旧两值相同，React 将不会触发组件的重新渲染，从而实现较好的性能。  
在并发模式（concurrent mode）中，使用 useEffect 订阅外部资源可能会导致“tearing”问题。为了解决这个问题，我们只能在外部资源更新时临时关闭时间分片（time-slicing）功能，并同步地更新组件。这也解释了为什么 useSyncExternalStore 中存在 sync 关键字。  
``` 
function useBehaviorSubject(subject) {
  return useSyncExternalStore(subject.subscribe, subject.getValue);
}
```
由于这样的调用会导致时间分片被临时关闭，因此，如果有一系列外部资源像火车式地进行连续更新，这将阻塞 React 组件的重绘，甚至可能导致页面卡顿。这实际上又让我们回到了并发模式原本想要解决的问题，即外部资源的存在使得解决方案又回到了原点。  
另一个缺陷是，state 在 fiber 上存储了一份数据。如果使用像 immer 这样的不可变库，或是新的 ES 提案中的 Record 或 Tuple，每次更改都会生成一份新的对象。如果代码编写得不够好，那么这种方法可能会导致相对于第一种方法使用更多的内存。因此，这是一种以空间换取时间的解决方案。  
从 React 源码中，可以清晰地看到 useSyncExternalStore 和 useState 一样，将值写到了 fiber 上。虽然许多算法都包含这样的思想，但如果能够进行优化，无疑可以节省一定的内存。  
综合来看，React 在与外部资源交互时面临以下挑战：
- 存在无效渲染
- 必须临时关闭并发模式
- 以空间换取时间

## 使用 use 消费外部资源
在许多复杂的组件中，无效渲染往往是由于不得不在顶层调用 Hook 监听外部资源，但实际上并未在某些控制流中使用这些资源。如果能将 Hook 放置到真正需要这些资源的控制流中调用，就可以优化这些无效渲染。  
React 团队更倾向于将缓存这个问题留给用户层处理，而不是让 React 擦屁股。因此，在发送新值之前，只需判断新旧值是否相等，就可以避免在 fiber 上存储旧值，从而在节省空间的同时避免无效的强制渲染，实现 "小孩子才做选择，我都要" 的境界。进一步讲，不在 fiber 上存值还带来了诸多优势，这些优势相信无需赘述。  
关于并发模式，我们追求的是更精细的调度，即资源级或原子级的调度。实际上，道理很简单：只需将资源本身作为入参传递，当资源更新时，将与该资源相关的所有更新作为一个批次进行处理。这样就避免了由于 useSyncExternalStore 粗粒度更新导致的并发失效问题。  
``` 
function useBehaviorSubject(subject) {
  return use(subject, (callback) => {
    let subscription;
    subscription = subject.subscribe(() => {
      callback();
      subscription?.unsubscribe()
    })
  }, subject.getValue)
}
```
使用 use 获取外部资源的例子如上所述，与 useSyncExternalStore 非常相似，但有两点不同：第一是 subject 会作为第一个参数传入，当然另一种可能性是手动将其转换成所需的 “usable” 类型；第二是在值不能挂在 fiber 上的同时，effect 也不能挂在 fiber 上，因此需要在每次更新后手动执行 unsubscribe。  

**思想及其优势**  
从时间角度来说，软件工程认为，无非就四种解决方案：
- 减少复杂度，压缩执行时间。从业务角度，这和具体的业务息息相关，React 很难给出什么具体的方案。从算法角度，通常要以空间换取时间。React 能做的，就是在优化执行时间的同时，避免使用额外空间
- 时间切片，这点就是 React 的并发模式，能改进的就是使时间切片更加精准，并且减少不得不关闭时间切片的情况，上文也做了讨论
- React 希望通过 use 的推出来鼓励请求的预载
- 懒计算。use 这个新 Hook 的核心就在于此，把 Hook 的调用挪到真正需要数据的 block 中。换言之，就是精准注册重新渲染的可能性

懒计算是软件工程的通用术语，在这里，我们换成“懒订阅”来更加精准地描述取值时再进行订阅的行为。通过懒订阅，我们可以减少不必要的订阅操作，从而提高应用性能。这种思想在 React 的新 Hook 中得到了体现，有望引发一场 Hook 革命，带来更优化的应用程序。  

## Demo
虽然这个 Hook 还没有上线，但是我们已经可以充分利用这个 Hook 的核心思想了，那就是懒订阅。虽然目前的 Hook 都不能放到控制流中，但是我们可以绕个弯子，当真正读取数据的时候，再去监听数据源，从而注册渲染可能性，从而避免因为控制流程的存在，Hook 的返回值没有被消费，从而造成无效重绘的情况。  
这个优化后的 useBehaviorSubject 针对懒订阅的思想进行了实现。当 getValue 被调用时，才会触发数据源的订阅。同时，在每次重新渲染时，都会执行取消订阅操作。  
``` 
function useBehaviorSubject(subject) {
  const subscriptionRef = useRef(null);
  
  const [value, setValue] = useState(subject.getValue())
  
  if(subscriptionRef.current) {
    // unsubscribe in each rerender.
    // because rerender may go into blocks without actually getting value
    subscriptionRef.current.unsubscribe()
  }
  
  useEffect(() => {
    // make sure unsubscribe when unmount
    return () => subscriptionRef.current?.unsubscribe()
  }, [])
  
  return function getValue() {
    if(!subscriptionRef.current) {
      // only subscribe after getting value
      subscriptionRef.current = subject.subscribe(() => {
        setValue(subject.getValue())
      })
    } 
    
    return subject.getValue()
  }
}
```
除了返回一个显式的 getValue 函数，我们还可以选择返回一个隐式带有 get 方法的对象，以实现类似的效果。  
这两种方法之间存在一些细微差别。例如，当作为 props 传递时，getValue 函数不会被调用，因此只有在子组件调用时才触发监听逻辑。而带有 get 方法的 value 在被读取时就会触发监听逻辑。  
若采用 Proxy 实现带有 get 方法的 value，在处理数字、字符串等原始类型时可能会面临一些挑战。在这种情况下，我们可能需要将这些原始类型自动包装成对象，以便更好地支持这些类型。   
另一方面，直接使用 value 更符合直觉，而且对于提供 value/onChange 的基础组件库有更好的支持。  

## 节约无效重绘
在表格或表单业务的底层，通常可能会有一个根据不同 type 进行分发的组件。使用前文提到的 useBehaviorSubject 后，该组件可能如下所示：  
``` 
function Dispatcher() {
  const getInputValue = useBehaviorSubject(input$)
  switch(type) {
    case 'input':
      return <Input value={getInputValue()} onChange={e => input$.next(e.target.value)} />
    case ...:
       return ...
  }
}
```
在这个例子中，懒订阅的优势在于仅在实际需要获取输入值时才会订阅 input$。这意味着如果组件的 type 发生变化而不需要输入值时，getInputValue 不会触发订阅，从而避免了无效的重绘。这种方式可以显著提高性能，特别是在具有复杂控制流的组件中。  
假设页面上有 1000 个这样的组件，其中 400 个进入了 input 分支，另外 600 个进入了其他分支。那么，当 input$ 发生改变时，只有那 400 个进入 input 分支的会重新渲染，而剩下的那 600 个不会重新渲染。  
针对这个问题，将每个 switch case 拆成一个组件，并将取值的 Hook 放到对应的组件中也能达到性能优化的目的。然而，在大部分情况下，这个组件通常要么非常简单，简单到拆分都显得画蛇添足，要么非常复杂，各种业务逻辑耦合在一起，各种数据和逻辑互相依赖，没有办法拆分。所以，在这样的场景下，使用懒订阅无疑是雪中送炭。  

## 无需重绘获取最新值
在许多应用中，我们需要实现一个提交按钮，点击后将页面上的表单项发送给后端。如果不充分考虑性能优化，可能会导致每次表单发生变化时，提交按钮都会触发重绘。  
使用上述的 Hook 来实现此类需求，可以确保在获取到最新值的同时，不会触发任何不必要的重绘。  
``` 
function SubmitButton() {
  const getForm = useBehaviorSubject(form$);
  
  // When we submit, request backend with the latest form.
  return <Button onClick={() => {
    submit(getForm())
  }}>Submit</Button>
}
```
由于懒订阅会在读取值的时候订阅，在重新渲染时取消订阅，因此，无论提交函数是否会引发与 form$ 变更相关的重绘，form$ 都不会被订阅。这意味着，form$ 的变化不会导致组件重绘。换句话说，在这个例子中，SubmitButton 组件不会因为自身状态改变而触发重绘，从而实现了极致的性能优化。  
这只是一个用以说明优势的例子，实际上我们可以使用 form$.getValue()直接获取值。然而，在复杂情况下，useBehaviorSubject(form$) 可能仅是一个复杂 Hook 中的一部分，因此在这个复杂 Hook 的调用方可能无法直接获取到form$本身。换句话说，这个例子展示了一种全新的可能性，类似于 redux 文档中使用 redux 实现 todo-list 一样。  


原文:  
[Hook 革命！浅谈 React 新 Hook 的未来与思想](https://mp.weixin.qq.com/s/DZVMvq_wwtjjCckci-tVaQ)
