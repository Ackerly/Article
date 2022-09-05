# 搞懂12个Hooks
**自定义Hooks是什么**  
react-hooks是React16.8以后新增的钩子API，目的是增加代码的可复用性、逻辑性，最主要的是解决了函数式组件无状态的问题，这样既保留了函数式的简单，又解决了没有数据管理状态的缺陷  
自定义hooks是在react-hooks基础上的一个扩展，可以根据业务、需求去制定相应的hooks,将常用的逻辑进行封装，从而具备复用性  
**如何设计一个自定义Hooks**  
hooks本质上是一个函数，而这个函数,主要就是逻辑复用。hooks的驱动条件就是props的修改，useState、useReducer的使用是无状态组件更新的条件，从而驱动自定义hooks  
**通用模式**  
自定义hooks的名称是以use开头，我们设计为：  
``` 
const [ xxx, ...] = useXXX(参数一，参数二...)
```
**简单的小例子：usePow**  
写一个简单的小例子来了解下自定义hooks  
``` 
// usePow.ts
const Index = (list: number[]) => {

  return list.map((item:number) => {
    console.log(1)
    return Math.pow(item, 2)
  })
}

export default Index;

// index.tsx
import { Button } from 'antd-mobile';
import React,{ useState } from 'react';
import { usePow } from '@/components';

const Index:React.FC<any> = (props)=> {
  const [flag, setFlag] = useState<boolean>(true)
  const data = usePow([1, 2, 3])
  
  return (
    <div>
      <div>数字：{JSON.stringify(data)}</div>
      <Button color='primary' onClick={() => {setFlag(v => !v)}}>切换</Button>
       <div>切换状态：{JSON.stringify(flag)}</div>
    </div>
  );
}

export default Index;
```
通过 usePow 给所传入的数字平方, 用切换状态的按钮表示函数内部的状态,但是点击切换按钮会触发console.log(1)  
这样明显增加了性能开销，理想状态肯定不希望做无关的渲染，所以做自定义 hooks的时候一定要注意，需要减少性能开销,为组件加入 useMemo试试：
``` 
import { useMemo } from 'react';

const Index = (list: number[]) => {
  return useMemo(() => list.map((item:number) => {
    console.log(1)
    return Math.pow(item, 2)
  }), []) 
}
export default Index;
```
此时就已经解决了这个问题，所以要非常注意一点，一个好用的自定义hooks,一定要配合useMemo、useCallback等 Api 一起使用。  
## 玩转React Hooks
**useMemo**  
当一个父组件中调用了一个子组件的时候，父组件的 state 发生变化，会导致父组件更新，而子组件虽然没有发生改变，但也会进行更新  
简单的理解下，当一个页面内容非常复杂，模块非常多的时候，函数式组件会从头更新到尾，只要一处改变，所有的模块都会进行刷新，这种情况显然是没有必要  
理想的状态是各个模块只进行自己的更新，不要相互去影响，那么此时用useMemo是最佳的解决方案  
这里要尤其注意一点，只要父组件的状态更新，无论有没有对子组件进行操作，子组件都会进行更新，useMemo就是为了防止这点而出现的  
memo的作用是结合了pureComponent纯组件和 componentShouldUpdate功能，会对传入的props进行一次对比，然后根据第二个函数返回值来进一步判断哪些props需要更新。  
useMemo与memo的理念上差不多，都是判断是否满足当前的限定条件来决定是否执行callback函数，而useMemo的第二个参数是一个数组，通过这个数组来判定是否更新回调函数  
这种方式可以运用在元素、组件、上下文中，尤其是利用在数组上，先看一个例子：  
``` 
useMemo(() => (
        <div>
            {
                list.map((item, index) => (
                    <p key={index}>
                        {item.name}
                    </>
                )}
            }
        </div>
    ),[list])
```
从上面看出 useMemo只有在list发生变化的时候才会进行渲染，从而减少了不必要的开销  
总结一下useMemo的好处：  
- 可以减少不必要的循环和不必要的渲染
- 可以减少子组件的渲染次数
- 通过特地的依赖进行更新，可以避免很多不必要的开销，但要注意，有时候在配合 useState拿不到最新的值，这种情况可以考虑使用 useRef解决

**useCallback**  
useCallback与useMemo极其类似,可以说是一模一样，唯一不同的是useMemo返回的是函数运行的结果，而useCallback返回的是函数  
注意：这个函数是父组件传递子组件的一个函数，防止做无关的刷新，其次，这个组件必须配合memo,否则不但不会提升性能，还有可能降低性能  
``` 
import React, { useState, useCallback } from 'react';
import { Button } from 'antd-mobile';

const MockMemo: React.FC<any> = () => {
const [count,setCount] = useState(0)
const [show,setShow] = useState(true)

const  add = useCallback(()=>{
  setCount(count + 1)
},[count])

return (
  <div>
    <div style={{display: 'flex', justifyContent: 'flex-start'}}>
      <TestButton title="普通点击" onClick={() => setCount(count + 1) }/>
      <TestButton title="useCallback点击" onClick={add}/>
    </div>
    <div style={{marginTop: 20}}>count: {count}</div>
    <Button onClick={() => {setShow(!show)}}> 切换</Button>
  </div>
)
}

const TestButton = React.memo((props:any)=>{
console.log(props.title)
return <Button color='primary' onClick={props.onClick} style={props.title === 'useCallback点击' ? {        marginLeft: 20
} : undefined}>{props.title}</Button>
})

export default MockMemo;
```
可以看到，当点击切换按钮的时候，没有经过 useCallback封装的函数会再次刷新，而经过 useCallback包裹的函数不会被再次刷新

**useRef**  
useRef 可以获取当前元素的所有属性，并且返回一个可变的ref对象，并且这个对象只有current属性，可设置initialValue  
通过useRef获取对应的属性值：  
``` 
import React, { useState, useRef } from 'react';

const Index:React.FC<any> = () => {
  const scrollRef = useRef<any>(null);
  const [clientHeight, setClientHeight ] = useState<number>(0)
  const [scrollTop, setScrollTop ] = useState<number>(0)
  const [scrollHeight, setScrollHeight ] = useState<number>(0)

  const onScroll = () => {
    if(scrollRef?.current){
      let clientHeight = scrollRef?.current.clientHeight; //可视区域高度
      let scrollTop  = scrollRef?.current.scrollTop;  //滚动条滚动高度
      let scrollHeight = scrollRef?.current.scrollHeight; //滚动内容高度
      setClientHeight(clientHeight)
      setScrollTop(scrollTop)
      setScrollHeight(scrollHeight)
    }
  }

  return (
    <div >
      <div >
        <p>可视区域高度：{clientHeight}</p>
        <p>滚动条滚动高度：{scrollTop}</p>
        <p>滚动内容高度：{scrollHeight}</p>
      </div>
      <div style={{height: 200, overflowY: 'auto'}} ref={scrollRef} onScroll={onScroll} >
        <div style={{height: 2000}}></div>
      </div>
         </div>
  );
};

export default Index;
```
_缓存数据_  
除了获取对应的属性值外，useRef还有一点比较重要的特性，那就是 缓存数据  
封装一个合格的自定义hooks的时候需要结合useMemo、useCallback等Api，但我们控制变量的值用useState 有可能会导致拿到的是旧值，并且如果他们更新会带来整个组件重新执行，这种情况下，我们使用useRef将会是一个非常不错的选择  
在react-redux的源码中，在hooks推出后，react-redux用大量的useMemo重做了Provide等核心模块，其中就是运用useRef来缓存数据，并且所运用的 useRef() 没有一个是绑定在dom元素上的，都是做数据缓存用的  
简单的来看一下：  
``` 
// 缓存数据
    /* react-redux 用userRef 来缓存 merge之后的 props */ 
    const lastChildProps = useRef() 
    
    // lastWrapperProps 用 useRef 来存放组件真正的 props信息 
    const lastWrapperProps = useRef(wrapperProps) 
    
    //是否储存props是否处于正在更新状态 
    const renderIsScheduled = useRef(false)

    //更新数据
    function captureWrapperProps( 
        lastWrapperProps, 
        lastChildProps, 
        renderIsScheduled, 
        wrapperProps, 
        actualChildProps, 
        childPropsFromStoreUpdate, 
        notifyNestedSubs 
    ) { 
        lastWrapperProps.current = wrapperProps 
        lastChildProps.current = actualChildProps 
        renderIsScheduled.current = false 
   }
```
react-redux 用重新赋值的方法，改变了缓存的数据源，减少了不必要的更新，如过采取useState势必会重新渲染  
**useLatest**  
useRef 可以拿到最新值，可以进行简单的封装，这样做的好处是：可以随时确保获取的是最新值，并且也可以解决闭包问题  
``` 
import { useRef } from 'react';

   const useLatest = <T>(value: T) => {
     const ref = useRef(value)
     ref.current = value

     return ref
   };

   export default useLatest;
```
**结合useMemo和useRef封装useCreatio**  
useCreation ：是 useMemo 或 useRef的替代品。换言之，useCreation这个钩子增强了 useMemo 和 useRef，让这个钩子可以替换这两个钩子。  
- useMemo的值不一定是最新的值，但useCreation可以保证拿到的值一定是最新的值
- 对于复杂常量的创建，useRef容易出现潜在的的性能隐患，但useCreation可以避免

``` 
// 每次重渲染，都会执行实例化 Subject 的过程，即便这个实例立刻就被扔掉了
   const a = useRef(new Subject()) 
   
// 通过 factory 函数，可以避免性能隐患
const b = useCreation(() => new Subject(), []) 
```
如何封装一个useCreation,首先要明白以下三点:  
- 先确定参数，useCreation 的参数与useMemo的一致，第一个参数是函数，第二个参数参数是可变的数组
- 值要保存在 useRef中，这样可以将值缓存，从而减少无关的刷新
- 更新值的判断，怎么通过第二个参数来判断是否更新 useRef里的值。

实现一个useCreation  
``` 
import { useRef } from 'react';
import type { DependencyList } from 'react';

const depsAreSame = (oldDeps: DependencyList, deps: DependencyList):boolean => {
  if(oldDeps === deps) return true
  
  for(let i = 0; i < oldDeps.length; i++) {
    // 判断两个值是否是同一个值
    if(!Object.is(oldDeps[i], deps[i])) return false
  }

  return true
}

const useCreation = <T>(fn:() => T, deps: DependencyList)=> {

  const { current } = useRef({ 
    deps,
    obj:  undefined as undefined | T ,
    initialized: false
  })

  if(current.initialized === false || !depsAreSame(current.deps, deps)) {
    current.deps = deps;
    current.obj = fn();
    current.initialized = true;
  }

  return current.obj as T
} 

export default useCreation;
```
在useRef判断是否更新值通过initialized 和 depsAreSame来判断，其中depsAreSame通过存储在 useRef下的deps(旧值) 和 新传入的 deps（新值）来做对比，判断两数组的数据是否一致，来确定是否更新    
_验证 useCreation_  
``` 
import React, { useState } from 'react';
    import { Button } from 'antd-mobile';
    import { useCreation } from '@/components';

    const Index: React.FC<any> = () => {
      const [_, setFlag] = useState<boolean>(false)

      const getNowData = () => {
        return Math.random()
      }

      const nowData = useCreation(() => getNowData(), []);

      return (
        <div style={{padding: 50}}>
          <div>正常的函数：{getNowData()}</div>
          <div>useCreation包裹后的：{nowData}</div>
          <Button color='primary' onClick={() => {setFlag(v => !v)}}> 渲染</Button>
        </div>
      )
    }

    export default Index;
```
**useEffect**  
可以使用useEffect来模拟下class的componentDidMount和componentWillUnmount的功能  
**useMount**  
只是简化了使用useEffect的第二个参数
``` 
import { useEffect } from 'react';

const useMount = (fn: () => void) => {

  useEffect(() => {
    fn?.();
  }, []);
};

export default useMount;
```
**useUnmount**  
就是使用useRef来确保所传入的函数为最新的状态，所以可以结合上述讲的useLatest结合使用  
``` 
import { useEffect, useRef } from 'react';

    const useUnmount = (fn: () => void) => {

      const ref = useRef(fn);
      ref.current = fn;

      useEffect(
        () => () => {
          fn?.()
        },
        [],
      );
    };

    export default useUnmount;
```
**结合useMount和useUnmount做个小例子**  
``` 
import { Button, Toast } from 'antd-mobile';
import React,{ useState } from 'react';
import { useMount, useUnmount } from '@/components';

const Child = () => {

  useMount(() => {
    Toast.show('首次渲染')
  });

  useUnmount(() => {
    Toast.show('组件已卸载')
  })

  return <div>你好，我是小杜杜</div>
}

const Index:React.FC<any> = (props)=> {
  const [flag, setFlag] = useState<boolean>(false)

  return (
    <div style={{padding: 50}}>
      <Button color='primary' onClick={() => {setFlag(v => !v)}}>切换 {flag ? 'unmount' : 'mount'}</Button>
      {flag && <Child />}
    </div>
  );
}

export default Index;
``` 
**useUpdate**  
``` 
import { useCallback, useState } from 'react';

const useUpdate = () => {
  const [, setState] = useState({});

  return useCallback(() => setState({}), []);
};

export default useUpdate;

//示例：
import { Button } from 'antd-mobile';
import React from 'react';
import { useUpdate } from '@/components';


const Index:React.FC<any> = (props)=> {
  const update = useUpdate();

  return (
    <div style={{padding: 50}}>
      <div>时间：{Date.now()}</div>
      <Button color='primary' onClick={update}>更新时间</Button>
    </div>
  );
}

export default Index;
```
## 案例
### useReactive
useReactiv: 一种具备响应式的useState  
useState可以定义变量其格式为： const [count, setCount] = useState<number>(0),通过setCount来设置，count来获取，使用这种方式才能够渲染视图  
像这样 let count = 0; count =7 此时count的值就是7，也就是说数据是响应式的  
那么可不可以将 useState也写成响应式的呢？我可以自由设置count的值,并且可以随时获取到count的最新值，而不是通过setCount来设置。  
现一个具备 响应式 特点的 useState 也就是 useRective,提出以下疑问:  
- 这个钩子的出入参该怎么设定？
- 如何将数据制作成响应式（毕竟普通的操作无法刷新视图）？
- 如何使用TS去写，完善其类型？
- 如何更好的去优化？

以上四个小问题，最关键的就是第二个，我们如何将数据弄成响应式，想要弄成响应式，就必须监听到值的变化，在做出更改，也就是说，我们对这个数进行操作的时候，要进行相应的拦截，这时就需要ES6的一个知识点：Proxy  
Proxy：接受的参数是对象，所以第一个问题也解决了，入参就为对象。那么如何去刷新视图呢？这里就使用上述的useUpdate来强制刷新，使数据更改。  
至于优化这一块，使用上文说的useCreation就好，再配合useRef来放initialState即可  
``` 
import { useRef } from 'react';
import { useUpdate, useCreation } from '../index';

const observer = <T extends Record<string, any>>(initialVal: T, cb: () => void): T => {

 const proxy = new Proxy<T>(initialVal, {
    get(target, key, receiver) {
      const res = Reflect.get(target, key, receiver);
      return typeof res === 'object' ? observer(res, cb) : Reflect.get(target, key);
    },
    set(target, key, val) {
      const ret = Reflect.set(target, key, val);
      cb();
      return ret;
    },
  });

  return proxy;
}

const useReactive = <T extends Record<string, any>>(initialState: T):T => {
  const ref = useRef<T>(initialState);
  const update = useUpdate();

  const state = useCreation(() => {
    return observer(ref.current, () => {
      update();
    });
  }, []);

  return state
};

export default useReactive;
```
因为不知道会传递什么类型的initialState所以在这需要使用泛型，接受的参数是对象，可就是 key-value 的形式，其中 key 为 string，value 可以是 任意类型，所以我们使用 Record<string, any>
拦截这块,只需要拦截设置（set） 和 获取（get） 即可，其中：  
- 设置这块，需要改变是图，也就是说需要，使用useUpdate来强制刷新
- 获取这块，需要判断其是否为对象，是的话继续递归，不是的话返回就行

以 字符串、数字、布尔、数组、函数、计算属性几个方面去验证一下：  
``` 
import { Button } from 'antd-mobile';
    import React from 'react';
    import { useReactive } from '@/components'

    const Index:React.FC<any> = (props)=> {

      const state = useReactive<any>({
        count: 0,
        name: '小杜杜',
        flag: true,
        arr: [],
        bugs: ['小杜杜', 'react', 'hook'],
        addBug(bug:string) {
          this.bugs.push(bug);
        },
        get bugsCount() {
          return this.bugs.length;
        },
   })

return (
<div style={{padding: 20}}>
  <div style={{fontWeight: 'bold'}}>基本使用：</div>
   <div style={{marginTop: 8}}> 对数字进行操作：{state.count}</div>
   <div style={{margin: '8px 0', display: 'flex',justifyContent: 'flex-start'}}>
     <Button color='primary' onClick={() => state.count++ } >加1</Button>
     <Button color='primary' style={{marginLeft: 8}} onClick={() => state.count-- } >减1</Button>
     <Button color='primary' style={{marginLeft: 8}} onClick={() => state.count = 7 } >设置为7</Button>
   </div>
   <div style={{marginTop: 8}}> 对字符串进行操作：{state.name}</div>
   <div style={{margin: '8px 0', display: 'flex',justifyContent: 'flex-start'}}>
     <Button color='primary' onClick={() => state.name = '小杜杜' } >设置为小杜杜</Button>
     <Button color='primary' style={{marginLeft: 8}} onClick={() => state.name = 'Domesy'} >设置为Domesy</Button>
   </div>
   <div style={{marginTop: 8}}> 对布尔值进行操作：{JSON.stringify(state.flag)}</div>
   <div style={{margin: '8px 0', display: 'flex',justifyContent: 'flex-start'}}>
     <Button color='primary' onClick={() => state.flag = !state.flag } >切换状态</Button>
   </div>
   <div style={{marginTop: 8}}> 对数组进行操作：{JSON.stringify(state.arr)}</div>
   <div style={{margin: '8px 0', display: 'flex',justifyContent: 'flex-start'}}>
     <Button color="primary" onClick={() => state.arr.push(Math.floor(Math.random() * 100))} >push</Button>
     <Button color="primary" style={{marginLeft: 8}} onClick={() => state.arr.pop()} >pop</Button>
     <Button color="primary" style={{marginLeft: 8}} onClick={() => state.arr.shift()} >shift</Button>
     <Button color="primary" style={{marginLeft: 8}} onClick={() => state.arr.unshift(Math.floor(Math.random() * 100))} >unshift</Button>
     <Button color="primary" style={{marginLeft: 8}} onClick={() => state.arr.reverse()} >reverse</Button>
     <Button color="primary" style={{marginLeft: 8}} onClick={() => state.arr.sort()} >sort</Button>
   </div>
   <div style={{fontWeight: 'bold', marginTop: 8}}>计算属性：</div>
   <div style={{marginTop: 8}}>数量：{ state.bugsCount } 个</div>  
   <div style={{margin: '8px 0'}}>
             <form
               onSubmit={(e) => {
                 state.bug ? state.addBug(state.bug) : state.addBug('domesy')
                 state.bug = '';
                 e.preventDefault();
               }}
             >
               <input type="text" value={state.bug} onChange={(e) => (state.bug = e.target.value)} />
               <button type="submit"  style={{marginLeft: 8}} >增加</button>
               <Button color="primary" style={{marginLeft: 8}} onClick={() => state.bugs.pop()}>删除</Button>
             </form>

           </div>
           <ul>
             {
               state.bugs.map((bug:any, index:number) => (
                 <li key={index}>{bug}</li>
               ))
             }
           </ul>
        </div>
      );
    }

    export default Index;
```
### 案例2: useEventListener
监听各种事件的时候需要做监听，如：监听点击事件、键盘事件、滚动事件等，将其统一封装起来，方便后续调用  
useEventListener的入参可分为三个  
- 第一个event是事件（如：click、keydown）
- 第二个回调函数（所以不需要出参）
- 第三个就是目标（是某个节点还是全局）

需要注意一点就是在销毁的时候需要移除对应的监听事件
``` 
import { useEffect } from 'react';

    const useEventListener = (event: string, handler: (...e:any) => void, target: any = window) => {

      useEffect(() => {
        const targetElement  = 'current' in target ? target.current : window;
        const useEventListener = (event: Event) => {
          return handler(event)
        }
        targetElement.addEventListener(event, useEventListener)
        return () => {
          targetElement.removeEventListener(event, useEventListener)
        }
      }, [event])
    };

    export default useEventListener;
```
这里把target默认设置成了window，至于为什么要这么写：'current' in target是因为我们用useRef拿到的值都是 ref.current  
**优化**  
- 首先需要hasInitRef来存储是否是第一次进入，通过它来判断初始化存储
- 然后考虑有几个参数需要存储，从上述代码上来看，可变的变量有两个，一个是event，另一个是target，其次，我们还需要存储对应的卸载后的函数，所以存储的变量应该有3个
- 接下来考虑一下什么情况下触发更新，也就是可变的两个参数：event和 target
- 最后在卸载的时候可以考虑使用useUnmount，并执行存储对应的卸载后的函数 和把hasInitRef还原

``` 
import { useEffect } from 'react';
    import type { DependencyList } from 'react';
    import { useRef } from 'react';
    import useLatest from '../useLatest';
    import useUnmount from '../useUnmount';

    const depsAreSame = (oldDeps: DependencyList, deps: DependencyList):boolean => {
      for(let i = 0; i < oldDeps.length; i++) {
        if(!Object.is(oldDeps[i], deps[i])) return false
      }
      return true
    }

    const useEffectTarget = (effect: () => void, deps:DependencyList, target: any) => {

      const hasInitRef = useRef(false); // 一开始设置初始化
      const elementRef = useRef<(Element | null)[]>([]);// 存储具体的值
      const depsRef = useRef<DependencyList>([]); // 存储传递的deps
      const unmountRef = useRef<any>(); // 存储对应的effect

      // 初始化 组件的初始化和更新都会执行
      useEffect(() => {
        const targetElement  = 'current' in target ? target.current : window;

        // 第一遍赋值
        if(!hasInitRef.current){
          hasInitRef.current = true;

          elementRef.current = targetElement;
          depsRef.current = deps;
          unmountRef.current = effect();
          return
        }
        // 校验变值: 目标的值不同， 依赖值改变
        if(elementRef.current !== targetElement || !depsAreSame(deps, depsRef.current)){
                 //先执行对应的函数
          unmountRef.current?.();
          //重新进行赋值
          elementRef.current = targetElement;
          depsRef.current = deps; 
          unmountRef.current = effect();
        }
      })

      useUnmount(() => {
        unmountRef.current?.();
        hasInitRef.current = false;
      })
    }

    const useEventListener = (event: string, handler: (...e:any) => void, target: any = window) => {
      const handlerRef = useLatest(handler);

      useEffectTarget(() => {
        const targetElement  = 'current' in target ? target.current : window;

        //  防止没有 addEventListener 这个属性
        if(!targetElement?.addEventListener) return;

        const useEventListener = (event: Event) => {
          return handlerRef.current(event)
        }
        targetElement.addEventListener(event, useEventListener)
        return () => {
          targetElement.removeEventListener(event, useEventListener)
        }
      }, [event], target)
    };

    export default useEventListener; 
```
- 在这里只用useEffect是因为，在更新和初始化的情况下都需要使用
- 必须要防止没有 addEventListener这个属性的情况，监听的目标有可能没有加载出来

**验证**  
验证一下useEventListener是否能够正常的使用，顺变验证一下初始化、卸载的  
``` 
import React, { useState, useRef } from 'react';
import { useEventListener } from '@/components'
import { Button } from 'antd-mobile';

const Index:React.FC<any> = (props)=> {

  const [count, setCount] = useState<number>(0)
  const [flag, setFlag] = useState<boolean>(true)
  const [key, setKey] = useState<string>('')
  const ref = useRef(null);

  useEventListener('click', () => setCount(v => v +1), ref)
  useEventListener('keydown', (ev) => setKey(ev.key));

  return (
    <div style={{padding: 20}}>
      <Button color='primary' onClick={() => {setFlag(v => !v)}}>切换 {flag ? 'unmount' : 'mount'}</Button>
      {
        flag && <div>
          <div>数字：{count}</div>
          <button ref={ref} >加1</button>
          <div>监听键盘事件：{key}</div>
        </div>
      }

    </div>
  );
}

export default Index;
```
**useHover**
useHover：监听 DOM 元素是否有鼠标悬停  
只需要通过 useEventListener来监听mouseenter和mouseleave即可，在返回布尔值就行了：  
``` 
import { useState } from 'react';
import useEventListener  from '../useEventListener';

interface Options {
  onEnter?: () => void;
  onLeave?: () => void;
}

const useHover = (target:any, options?:Options): boolean => {

  const [flag, setFlag] = useState<boolean>(false)
  const { onEnter, onLeave } = options || {};

  useEventListener('mouseenter', () => {
    onEnter?.()
    setFlag(true)
  }, target)

  useEventListener('mouseleave', () => {
    onLeave?.()
    setFlag(false)
  }, target)

  return flag
};

export default useHover;
```
### 有关时间的Hooks
**useTimeout**  
useTimeout：一段时间内，执行一次  
传递参数只要函数和延迟时间即可，需要注意的是卸载的时候将定时器清除下就OK了  
``` 
import { useEffect } from 'react';
import useLatest from '../useLatest';


const useTimeout = (fn:() => void, delay?: number): void => {

  const fnRef = useLatest(fn)

  useEffect(() => {
    if(!delay || delay < 0) return;

    const timer = setTimeout(() => {
      fnRef.current();
    }, delay)

    return () => {
      clearTimeout(timer)
    }
  }, [delay])

};

export default useTimeout;
```
**useInterval**  
useInterval: 每过一段时间内一直执行  
大体上与useTimeout一样，多了一个是否要首次渲染的参数immediate  
``` 
import { useEffect } from 'react';
import useLatest from '../useLatest';


const useInterval = (fn:() => void, delay?: number, immediate?:boolean): void => {

  const fnRef = useLatest(fn)

  useEffect(() => {
    if(!delay || delay < 0) return;
    if(immediate) fnRef.current();

    const timer = setInterval(() => {
      fnRef.current();
    }, delay)

    return () => {
      clearInterval(timer)
    }
  }, [delay])

};

export default useInterval;
```
**useCountDown**  
useCountDown：简单控制倒计时的钩子  
这个钩子需要什么：  
- 要做倒计时的钩子首先需要一个目标时间（targetDate），控制时间变化的秒数（interval默认为1s），然后就是倒计时完成后所触发的函数（onEnd）
- 返参就更加一目了然了，返回的是两个时间差的数值（time），再详细点可以换算成对应的天、时、分等（formattedRes）

``` 
import { useState, useEffect, useMemo } from 'react';
    import useLatest from '../useLatest';
    import dayjs from 'dayjs';

    type DTime = Date | number | string | undefined;

    interface Options {
      targetDate?: DTime;
      interval?: number;
      onEnd?: () => void;
    }

    interface FormattedRes {
      days: number;
      hours: number;
      minutes: number;
      seconds: number;
      milliseconds: number;
    }

    const calcTime = (time: DTime) => {
      if(!time) return 0

      const res = dayjs(time).valueOf() - new Date().getTime(); //计算差值

      if(res < 0) return 0

      return res
    }

    const parseMs = (milliseconds: number): FormattedRes => {
      return {
        days: Math.floor(milliseconds / 86400000),
        hours: Math.floor(milliseconds / 3600000) % 24,
        minutes: Math.floor(milliseconds / 60000) % 60,
        seconds: Math.floor(milliseconds / 1000) % 60,
        milliseconds: Math.floor(milliseconds) % 1000,
      };
    };

    const useCountDown = (options?: Options) => {

      const { targetDate, interval = 1000, onEnd } = options || {};

      const [time, setTime] = useState(() =>  calcTime(targetDate));
      const onEndRef = useLatest(onEnd);

      useEffect(() => {

        if(!targetDate) return setTime(0)

        setTime(calcTime(targetDate))

        const timer = setInterval(() => {
          const target = calcTime(targetDate);

          setTime(target);
          if (target === 0) {
            clearInterval(timer);
            onEndRef.current?.();
          }
        }, interval);
        return () => clearInterval(timer);
      },[targetDate, interval])

      const formattedRes = useMemo(() => {
        return parseMs(time);
      }, [time]);

      return [time, formattedRes] as const
    };

    export default useCountDown;
```
验证  
``` 
import React, { useState } from 'react';
    import { useCountDown } from '@/components'
    import { Button, Toast } from 'antd-mobile';

    const Index:React.FC<any> = (props)=> {

      const [_, formattedRes] = useCountDown({
        targetDate: '2022-12-31 24:00:00',
      });

      const { days, hours, minutes, seconds, milliseconds } = formattedRes;

      const [count, setCount] = useState<number>();

      const [countdown] = useCountDown({
        targetDate: count,
        onEnd: () => {
          Toast.show('结束')
        },
      });

      return (
        <div style={{padding: 20}}>
          <div> 距离 2022-12-31 24:00:00 还有 {days} 天 {hours} 时 {minutes} 分 {seconds} 秒 {milliseconds} 毫秒</div>
          <div>
            <p style={{marginTop: 12}}>动态变化：</p>
            <Button color='primary' disabled={countdown !== 0} onClick={() => setCount(Date.now() + 3000)}>
              {countdown === 0 ? '开始' : `还有 ${Math.round(countdown / 1000)}s`}
            </Button>
            <Button style={{marginLeft: 8}} onClick={() => setCount(undefined)}>停止</Button>
          </div>
        </div>
      );
    }
export default Index;
```

原文: 
[搞懂这12个Hooks，保证让你玩转React](https://mp.weixin.qq.com/s/Q2DDI1wm22zPiMUZlpAYZw)
