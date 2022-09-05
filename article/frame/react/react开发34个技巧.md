# react开发34个技巧
## 组件通讯
子组件
``` 
import React from "react";
import PropTypes from "prop-types";
import { Button } from "antd";

export default class EightteenChildOne extends React.Component {
  static propTypes = { //propTypes校验传入类型,详情在技巧11
    name: PropTypes.string
  };

  click = () => {
    // 通过触发方法子传父
    this.props.eightteenChildOneToFather("这是 props 改变父元素的值");
  };

  render() {
    return (
      <div>
        <div>这是通过 props 传入的值{this.props.name}</div>
        <Button type="primary" onClick={this.click}>
          点击改变父元素值
        </Button>
      </div>
    );
  }
}
```
父组件
``` 
<EightteenChildOne name={'props 传入的 name 值'} eightteenChildOneToFather={(mode)=>this.eightteenChildOneToFather(mode)}></EightteenChildOne> 

// 或者
<EightteenChildOne name={'props 传入的 name 值'} eightteenChildOneToFather={this.eightteenChildOneToFather.bind(this)}></EightteenChildOne> 
```
props传多个值时：  
传统写法
``` 
const {dataOne,dataTwo,dataThree} = this.state
<Com dataOne={dataOne} dataTwo={dataTwo} dataThree={dataThree}>
```
升级写法  
``` 
<Com {...{dataOne,dataTwo,dataThree}}>
```
### props升级版
原理:子组件里面利用 props 获取父组件方法直接调用,从而改变父组件的值  
调用父组件方法改变该值  
``` 
// 父组件
state = {
  count: {}
}
changeParentState = obj => {
    this.setState(obj);
}
// 子组件
onClick = () => {
    this.props.changeParentState({ count: 2 });
}
```
### Provider,Consumer和Context
Context在 16.x 之前是定义一个全局的对象,类似 vue 的 eventBus,如果组件要使用到该值直接通过this.context获取
``` 
//根组件
class MessageList extends React.Component {
  getChildContext() {
    return {color: "purple",text: "item text"};
  }

  render() {
    const {messages} = this.props || {}
    const children = messages && messages.map((message) =>
      <Message text={message.text} />
    );
    return <div>{children}</div>;
  }
}

MessageList.childContextTypes = {
  color: React.PropTypes.string
  text: React.PropTypes.string
};

//中间组件
class Message extends React.Component {
  render() {
    return (
      <div>
        <MessageItem />
        <Button>Delete</Button>
      </div>
    );
  }
}

//孙组件(接收组件)
class MessageItem extends React.Component {
  render() {
    return (
      <div>
        {this.context.text}
      </div>
    );
  }
}

MessageItem.contextTypes = {
  text: React.PropTypes.string //React.PropTypes在 15.5 版本被废弃,看项目实际的 React 版本
};

class Button extends React.Component {
  render() {
    return (
      <button style={{background: this.context.color}}>
        {this.props.children}
      </button>
    );
  }
}

Button.contextTypes = {
  color: React.PropTypes.string
};
```
16.x 之后的Context使用了Provider和Customer模式,在顶层的Provider中传入value，在子孙级的Consumer中获取该值，并且能够传递函数，用来修改context
声明一个全局的 context 定义,context.js
``` 
import React from 'react'
let { Consumer, Provider } = React.createContext();//创建 context 并暴露Consumer和Provider模式
export {
    Consumer,
    Provider
}
```
父组件导入
``` 
// 导入 Provider
import {Provider} from "../../utils/context"

<Provider value={name}>
  <div style={{border:'1px solid red',width:'30%',margin:'50px auto',textAlign:'center'}}>
    <p>父组件定义的值:{name}</p>
    <EightteenChildTwo></EightteenChildTwo>
  </div>
</Provider>
```
子组件
``` 
// 导入Consumer
import { Consumer } from "../../utils/context"
function Son(props) {
  return (
    //Consumer容器,可以拿到上文传递下来的name属性,并可以展示对应的值
    <Consumer>
      {name => (
        <div
          style={{
            border: "1px solid blue",
            width: "60%",
            margin: "20px auto",
            textAlign: "center"
          }}
        >
        // 在 Consumer 中可以直接通过 name 获取父组件的值
          <p>子组件。获取父组件的值:{name}</p>
        </div>
      )}
    </Consumer>
  );
}
export default Son;
```
### EventEmitter
使用 events 插件定义一个全局的事件机制
### 路由传参  
params
``` 
<Route path='/path/:name' component={Path}/>
<link to="/path/2">xxx</Link>
this.props.history.push({pathname:"/path/" + name});
```
query
``` 
<Route path='/query' component={Query}/>
<Link to={{ pathname : '/query' , query : { name : 'sunny' }}}>
this.props.history.push({pathname:"/query",query: { name : 'sunny' }});
读取参数用: this.props.location.query.name
```
state
``` 
<Route path='/sort ' component={Sort}/>
<Link to={{ pathname : '/sort ' , state : { name : 'sunny' }}}> 
this.props.history.push({pathname:"/sort ",state : { name : 'sunny' }});
读取参数用: this.props.location.query.state 
```
search
``` 
<Route path='/web/search ' component={Search}/>
<link to="web/search?id=12121212">xxx</Link>
this.props.history.push({pathname:`/web/search?id ${row.id}`});
读取参数用: this.props.location.search
```
react-router-dom: v4.2.2有 bug,传参跳转页面会空白,刷新才会加载出来  
优缺点
- params在HashRouter和BrowserRouter路由中刷新页面参数都不会丢失
- state在BrowserRouter中刷新页面参数不会丢失，在HashRouter路由中刷新页面会丢失
- query：在HashRouter和BrowserRouter路由中刷新页面参数都会丢失
- query和 state 可以传对象

### onRef
原理:onRef 通讯原理就是通过 props 的事件机制将组件的 this(组件实例)当做参数传到父组件,父组件就可以操作子组件的 state 和方法  
EightteenChildFour.jsx
``` 
export default class EightteenChildFour extends React.Component {
  state={
      name:'这是组件EightteenChildFour的name 值'
  }

  componentDidMount(){
    this.props.onRef(this)
    console.log(this) // ->将EightteenChildFour传递给父组件this.props.onRef()方法
  }

  click = () => {
    this.setState({name:'这是组件click 方法改变EightteenChildFour改变的name 值'})
  };

  render() {
    return (
      <div>
        <div>{this.state.name}</div>
        <Button type="primary" onClick={this.click}>
          点击改变组件EightteenChildFour的name 值
        </Button>
      </div>
    );
  }
}
```
eighteen.jsx
``` 
<EightteenChildFour onRef={this.eightteenChildFourRef}></EightteenChildFour>

eightteenChildFourRef = (ref)=>{
  console.log('eightteenChildFour的Ref值为')
  // 获取的 ref 里面包括整个组件实例
  console.log(ref)
  // 调用子组件方法
  ref.click()
}
```
### ref
原理:就是通过 React 的 ref 属性获取到整个子组件实例,再进行操作  
EightteenChildFive.jsx
``` 
// 常用的组件定义方法
export default class EightteenChildFive extends React.Component {
  state={
      name:'这是组件EightteenChildFive的name 值'
  }

  click = () => {
    this.setState({name:'这是组件click 方法改变EightteenChildFive改变的name 值'})
  };

  render() {
    return (
      <div>
        <div>{this.state.name}</div>
        <Button type="primary" onClick={this.click}>
          点击改变组件EightteenChildFive的name 值
        </Button>
      </div>
    );
  }
}
```
eighteen.jsx
``` 
// 钩子获取实例
componentDidMount(){
    console.log('eightteenChildFive的Ref值为')
      // 获取的 ref 里面包括整个组件实例,同样可以拿到子组件的实例
    console.log(this.refs["eightteenChildFiveRef"])
  }

// 组件定义 ref 属性
<EightteenChildFive ref="eightteenChildFiveRef"></EightteenChildFive>
```
### 事件通讯插件
redux、Mobx、flux
### hooks
hooks 是利用 userReducer 和 context 实现通讯,下面模拟实现一个简单的 redux， 核心文件分为 action,reducer,types  
action.js
``` 
import * as Types from './types';

export const onChangeCount = count => ({
    type: Types.EXAMPLE_TEST,
    count: count + 1
})
```
reducer.js
``` 
import * as Types from "./types";
export const defaultState = {
  count: 0
};
export default (state, action) => {
  switch (action.type) {
    case Types.EXAMPLE_TEST:
      return {
        ...state,
        count: action.count
      };
    default: {
      return state;
    }
  }
};
```
types.js
``` 
export const EXAMPLE_TEST = 'EXAMPLE_TEST';
```
eightteen.jsx
``` 
export const ExampleContext = React.createContext(null);//创建createContext上下文

// 定义组件
function ReducerCom() {
  const [exampleState, exampleDispatch] = useReducer(example, defaultState);

  return (
    <ExampleContext.Provider
      value={{ exampleState, dispatch: exampleDispatch }}
    >
      <EightteenChildThree></EightteenChildThree>
    </ExampleContext.Provider>
  );
}
```
EightteenChildThree.jsx // 组件
``` 
import React, {  useEffect, useContext } from 'react';
import {Button} from 'antd'

import {onChangeCount} from '../../pages/TwoTen/store/action';
import { ExampleContext } from '../../pages/TwoTen/eighteen';

const Example = () => {

    const exampleContext = useContext(ExampleContext);

    useEffect(() => { // 监听变化
        console.log('变化执行啦')
    }, [exampleContext.exampleState.count]);

    return (
        <div>
            <p>值为{exampleContext.exampleState.count}</p>
            <Button onClick={() => exampleContext.dispatch(onChangeCount(exampleContext.exampleState.count))}>点击加 1</Button>
        </div>
    )
}

export default Example;
```
hooks其实就是对原有React 的 API 进行了封装,暴露比较方便使用的钩子

|  钩子名   | 作用  |
|  ----  | ----  |
| useState  | 初始化和设置状态 |
| useEffect  | componentDidMount，componentDidUpdate和componentWillUnmount和结合体,所以可以监听useState定义值的变化 |
| useContext  | 定义一个全局的对象,类似 context |
| useReducer  | 可以增强函数提供类似 Redux 的功能 |
| useCallback  | 记忆作用,共有两个参数，第一个参数为一个匿名函数，就是我们想要创建的函数体。第二参数为一个数组，里面的每一项是用来判断是否需要重新创建函数体的变量，如果传入的变量值保持不变，返回记忆结果。如果任何一项改变，则返回新的结果 |
| useMemo  | 作用和传入参数与 useCallback 一致,useCallback返回函数,useDemo 返回值 |
| useRef  | 获取 ref 属性对应的 dom |
| useMutationEffect  | 作用与useEffect相同，但在更新兄弟组件之前，它在React执行其DOM改变的同一阶段同步触发 |
| useLayoutEffect  | 作用与useEffect相同，但在所有DOM改变后同步触发 |
useImperativeMethods
``` 
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeMethods(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```
### slot
将父组件的标签传给子组件,类似vue 的 v-slot,有些组件只是共用部分dom 逻辑,里面有部分逻辑是独立的
``` 
// 父组件文件
import SlotChild from 'SlotChild'

<SlotChild
slot={<div>这是父组件的 slot</div>}>
</SlotChild>

// 子组件
子组件直接获取 this.props.slot 就可获取到内容
```
### 对比
|  钩子名   | 作用  |  缺点  |
|  ----  | ----  | ----  |
| props  | 不需要引入外部插件 | 兄弟组件通讯需要建立共同父级组件,麻烦 |
| props 升级版  | 不需要引入外部插件,子传父,不需要在父组件用方法接收 | 同props |
| Provider,Consumer和Context  | 不需要引入外部插件,跨多级组件或者兄弟组件通讯利器 | 状态数据状态追踪麻烦 |
| EventEmitter  | 可支持兄弟,父子组件通讯 | 要引入外部插件 |
| 路由传参  | 可支持兄弟组件传值,页面简单数据传递非常方便 | 父子组件通讯无能为力 |
| onRef  | 可以在获取整个子组件实例,使用简单 | 兄弟组件通讯麻烦,官方不建议使用 |
| ref  | 同 onRef | 同 onRef |
| redux  | 建立了全局的状态管理器,兄弟父子通讯都可解决 | 引入了外部插件 |
| mobx  | 建立了全局的状态管理器,兄弟父子通讯都可解决 | 引入了外部插件 |
| flux  | 建立了全局的状态管理器,兄弟父子通讯都可解决 | 引入了外部插件 |
| hooks  | 16.x 新的属性,可支持兄弟,父子组件通讯 | 需要结合 context 一起使用 |
| slot  | 支持父向子传标签 |
redux , mobx和flux对比

|  方法   | 介绍  |
|  ----  | ----  |
| redux  | 1.核心模块:Action,Reducer,Store;2. Store 和更改逻辑是分开的;3. 只有一个 Store;4. 带有分层 reducer 的单一 Store;5. 没有调度器的概念;6. 容器组件是有联系的;7. 状态是不可改变的;8.更多的是遵循函数式编程思想|
| mobx  | 1.核心模块:Action,Reducer,Derivation;2.有多个 store;3.设计更多偏向于面向对象编程和响应式编程，通常将状态包装成可观察对象，一旦状态对象变更，就能自动获得更新 |
| flux  | 1.核心模块:Store,ReduceStore,Container;2.有多个 store; |
## require.context()
``` 
const path = require('path')
const files = require.context('@/components/home', false, /\.vue$/)
const modules = {}
files.keys().forEach(key => {
  const name = path.basename(key, '.vue')
  modules[name] = files(key).default || files(key)
})
```
## Decorator
decorator是ES7的一个新特性，可以修改class的属性
``` 
import React from 'react'
import Test from '../../utils/decorators'

@Test
//只要Decorator后面是Class，默认就已经把Class当成参数隐形传进Decorator了。
class TwentyNine extends React.Component{
    componentDidMount(){
        console.log(this,'decorator.js') // 这里的this是类的一个实例
        console.log(this.testable)
    }
    render(){
        return (
            <div>这是技巧23</div>
        )
    }
}

export default TwentyNine
```
decorators.js
``` 
function testable(target) {
  console.log(target)
  target.isTestable = true;
  target.prototype.getDate = ()=>{
    console.log( new Date() )
  }
}

export default testable
```
## 使用 if...else
有些时候需要根据不同状态值页面显示不同内容
``` 
import React from "react";

export default class Four extends React.Component {
  state = {
    count: 1
  };
  render() {
    let info
    if(this.state.count===0){
      info=(
        <span>这是数量为 0 显示</span>
      )
    } else if(this.state.count===1){
      info=(
        <span>这是数量为 1 显示</span>
      )
    }
    return (
      <div>
        {info}
      </div>
    );
  }
}
```
##  state 值改变的五种方式
方式 1
```
let {count} = this.state
this.setState({count:2})
```
方式 2:callBack
```
this.setState(({count})=>({count:count+2}))
```
方式 3:接收 state 和 props 参数
```
this.setState((state, props) => {
    return { count: state.count + props.step };
});
```
方式 4:hooks
```
const [count, setCount] = useState(0)
// 设置值
setCount(count+2)
```
方式 5:state 值改变后调用
``` 
this.setState(
    {count:3},()=>{
        //得到结果做某种事
    }
)
```
## 监听states 变化
16.x 之前使用componentWillReceiveProps
``` 
componentWillReceiveProps (nextProps){
  if(this.props.visible !== nextProps.visible){
      //props 值改变做的事
  }
}
```
注意:有些时候componentWillReceiveProps在 props 值未变化也会触发,因为在生命周期的第一次render后不会被调用，但是会在之后的每次render中被调用 = 当父组件再次传送props  
16.x 之后使用getDerivedStateFromProps,16.x 以后componentWillReveiveProps也未移除  
``` 
export default class Six extends React.Component {
  state = {
    countOne:1,
    changeFlag:''
  };
  clickOne(){
    let {countOne} = this.state
    this.setState({countOne:countOne+1})
  };
  static getDerivedStateFromProps (nextProps){
    console.log('变化执行')
    return{
      changeFlag:'state 值变化执行'
    }
  }
  render() {
    const {countOne,changeFlag} = this.state
    return (
      <div>
        <div>
         <Button type="primary" onClick={this.clickOne.bind(this)}>点击加 1</Button><span>countOne 值为{countOne}</span>
        <div>{changeFlag}</div>
        </div>
      </div>
    );
  }
}
```
## 组件定义方法  
ES5 的Function 定义  
``` 
function FunCom(props){
  return <div>这是Function 定义的组件</div>
}
ReactDOM.render(<FunCom name="Sebastian" />, mountNode)

// 在 hooks 未出来之前,这个是定义无状态组件的方法,现在有了 hooks 也可以处理状态
```
ES5的 createClass 定义  
``` 
const CreateClassCom = React.createClass({
  render: function() {
  return <div>这是React.createClass定义的组件</div>
  }
});
```
ES6 的 extends
``` 
class Com extends React.Component {
  render(){
    return(<div>这是React.Component定义的组件</div>)
  }
}
```
调用
``` 
export default class Seven extends React.Component {
  render() {
    return (
      <div>
        <FunCom></FunCom>
        <Com></Com>
      </div>
    );
  }
}
```
ES5的 createClass是利用function模拟class的写法做出来的es6; 通过es6新增class的属性创建的组件此组件创建简单.
## 通过 ref 属性获取 component
this.refs[属性名获取] 也可以作用到组件上,从而拿到组件实例
``` 
class RefOne extends React.Component{
  componentDidMount() {
    this.refs['box'].innerHTML='这是 div 盒子,通过 ref 获取'
  }
  render(){
    return(
      <div ref="box"></div>
    )
  }
```
回调函数,在dom节点或组件上挂载函数，函数的入参是dom节点或组件实例
``` 
class RefTwo extends React.Component{
  componentDidMount() {
    this.input.value='这是输入框默认值';
    this.input.focus();
  }
  render(){
    return(
      <input ref={comp => { this.input = comp; }}/>
    )
  }
}
```
React.createRef() React 16.3版本后，使用此方法来创建ref。将其赋值给一个变量，通过ref挂载在dom节点或组件上，该ref的current属性,将能拿到dom节点或组件的实例
``` 
class RefThree extends React.Component{
  constructor(props){
    super(props);
    this.myRef=React.createRef();
  }
  componentDidMount(){
    console.log(this.myRef.current);
  }
  render(){
    return <input ref={this.myRef}/>
  }
}
```
React.forwardRef,React 16.3版本后提供的，可以用来创建子组件，以传递ref
``` 
class RefFour extends React.Component{
  constructor(props){
    super(props);
    this.myFourRef=React.forwardRef();
  }
  componentDidMount(){
    console.log(this.myFourRef.current);
  }
  render(){
    return <Child ref={this.myFourRef}/>
  }
}
```
子组件通过React.forwardRef来创建，可以将ref传递到内部的节点或组件，进而实现跨层级的引用。
## static 使用
声明静态方法的关键字
``` 
export default class Nine extends React.Component {
  static update(data) {
    console.log('静态方法调用执行啦')
  }
  render() {
    return (
      <div>
        这是 static 关键字技能
      </div>
    );
  }
}

Nine.update('2')
```
注意:
1. ES6的class，我们定义一个组件的时候通常是定义了一个类，而static则是创建了一个属于这个类的属性或者方法
2. 组件则是这个类的一个实例，component的props和state是属于这个实例的，所以实例还未创建
3. 所以static并不是react定义的，而加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用,所以也是无法访问到 this
4. getDerivedStateFromProps也是通过静态方法监听值,详情请见技巧 6
## constructor和super
class方式定义的类默认都有一个constructor函数， 这个函数是构造函数的主函数， 该函数体内部的this指向生成的实例  
super关键字用于访问和调用一个对象的父对象上的函数
``` 
export default class Ten extends React.Component {
  constructor() { // class 的主函数
    super() // React.Component.prototype.constructor.call(this),其实就是拿到父类的属性和方法
    this.state = {
      arr:[]
    }
  }  
  render() {
    return (
      <div>
        这是技巧 10
      </div>
    );
  }
}
```
## PropTypes
场景:检测传入子组件的数据类型  
类型检查PropTypes自React v15.5起已弃用，请使用prop-types
``` 
// 旧的写法
class PropTypeOne extends React.Component {
  render() {
    return (
      <div>
        <div>{this.props.email}</div>
        <div>{this.props.name}</div>
      </div>
    );
  }
}

PropTypeOne.propTypes = {
  name: PropTypes.string, //值可为array,bool,func,number,object,symbol
  email: function(props, propName, componentName) { //自定义校验
    if (
      !/^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/.test(
        props[propName]
      )
    ) {
      return new Error(
        "组件" + componentName + "里的属性" + propName + "不符合邮箱的格式"
      );
    }
  },
};
```
利用 ES7 的静态属性关键字 static
``` 
class PropTypeTwo extends React.Component {
  static propTypes = {
      name:PropTypes.string
  };
  render() {
    return (
      <div>
        <div>{this.props.name}</div>
      </div>
    );
  }
}
```
## 使用类字段声明语法
在不使用构造函数的情况下初始化本地状态，并通过使用箭头函数声明类方法，而无需额外对它们进行绑定
``` 
class Counter extends Component {
  state = { value: 0 };

  handleIncrement = () => {
    this.setState(prevState => ({
      value: prevState.value + 1
    }));
  };

  handleDecrement = () => {
    this.setState(prevState => ({
      value: prevState.value - 1
    }));
  };

  render() {
    return (
      <div>
        {this.state.value}

        <button onClick={this.handleIncrement}>+</button>
        <button onClick={this.handleDecrement}>-</button>
      </div>
    )
  }
}
```
## 异步组件
安装 react-loadable ,babel插件安装 syntax-dynamic-import. react-loadable是通过webpack的异步import实现的
``` 
const Loading = () => {
  return <div>loading</div>;
};

const LoadableComponent = Loadable({
  loader: () => import("../../components/TwoTen/thirteen"),
  loading: Loading
});

export default class Thirteen extends React.Component {
  render() {
    return <LoadableComponent></LoadableComponent>;
  }
}
```
## 动态组件
利用三元表达式判断组件是否显示
``` 
class FourteenChildOne extends React.Component {
    render() {
        return <div>这是动态组件 1</div>;
    }
}

class FourteenChildTwo extends React.Component {
    render() {
        return <div>这是动态组件 2</div>;
    }
}

export default class Fourteen extends React.Component {
  state={
      oneShowFlag:true
  }
  tab=()=>{
      this.setState({oneShowFlag:!this.state.oneShowFlag})
  }
  render() {
    const {oneShowFlag} = this.state
    return (<div>
        <Button type="primary" onClick={this.tab}>显示组件{oneShowFlag?2:1}</Button>
        {oneShowFlag?<FourteenChildOne></FourteenChildOne>:<FourteenChildTwo></FourteenChildTwo>}
    </div>);
  }
}
```
## 递归组件
利用React.Fragment或者 div 包裹循环
``` 
class Item extends React.Component {
  render() {
    const list = this.props.children || [];
    return (
      <div className="item">
        {list.map((item, index) => {
          return (
            <React.Fragment key={index}>
              <h3>{item.name}</h3>
              {// 当该节点还有children时，则递归调用本身
              item.children && item.children.length ? (
                <Item>{item.children}</Item>
              ) : null}
            </React.Fragment>
          );
        })}
      </div>
    );
  }
}
```
## 受控组件和不受控组件
受控组件:组件的状态通过React 的状态值 state 或者 props 控制  
``` 
class Controll extends React.Component {
  constructor() {
    super();
    this.state = { value: "这是受控组件默认值" };
  }
  render() {
    return <div>{this.state.value}</div>;
  }
}
```
不受控组件:组件不被 React的状态值控制,通过 dom 的特性或者React 的ref 来控制
``` 
class NoControll extends React.Component {
  render() {
    return <div>{this.props.value}</div>;
  }
}
```
导入代码:
``` 
export default class Sixteen extends React.Component {
  componentDidMount() {
    console.log("ref 获取的不受控组件值为", this.refs["noControll"]);
  }
  render() {
    return (
      <div>
        <Controll></Controll>
        <NoControll
          value={"这是不受控组件传入值"}
          ref="noControll"
        ></NoControll>
      </div>
    );
  }
}
```
## 高阶组件(HOC)
将组件作为参数或者返回一个组件的组件
``` 
// 属性代理
import React,{Component} from 'react';

const Seventeen = WrappedComponent =>
  class extends React.Component {
    render() {
      const props = {
        ...this.props,
        name: "这是高阶组件"
      };
      return <WrappedComponent {...props} />;
    }
  };

class WrappedComponent extends React.Component {
  state={
     baseName:'这是基础组件' 
  }
  render() {
    const {baseName} = this.state
    const {name} = this.props
    return <div>
        <div>基础组件值为{baseName}</div>
        <div>通过高阶组件属性代理的得到的值为{name}</div>
    </div>
  }
}

export default Seventeen(WrappedComponent)
```
``` 
// 反向继承,利用 super 改变改组件的 this 方向,继而就可以在该组件处理容器组件的一些值
  const Seventeen = (WrappedComponent)=>{
    return class extends WrappedComponent {
        componentDidMount() {
            this.setState({baseName:'这是通过反向继承修改后的基础组件名称'})
        }
        render(){
            return super.render();
        }
    }
}

class WrappedComponent extends React.Component {
  state={
     baseName:'这是基础组件' 
  }
  render() {
    const {baseName} = this.state
    return <div>
        <div>基础组件值为{baseName}</div>
    </div>
  }
}

export default Seventeen(WrappedComponent);
```
## .元素是否显示
``` 
flag?<div>显示内容</div>:''
 flag&&<div>显示内容</div>
```
## Dialog 组件创建
通过 state 控制组件是否显示
``` 
 class NineteenChildOne extends React.Component {
  render() {
    const Dialog = () => <div>这是弹层1</div>;

    return this.props.dialogOneFlag && <Dialog />;
  }
}
```
通过ReactDom.render创建弹层-挂载根节点外层,通过原生的createElement,appendChild, removeChild和react 的ReactDOM.render,ReactDOM.unmountComponentAtNode来控制元素的显示和隐藏
NineteenChild.jsx
``` 
import ReactDOM from "react-dom";

class Dialog {
  constructor(name) {
    this.div = document.createElement("div");
    this.div.style.width = "200px";
    this.div.style.height = "200px";
    this.div.style.backgroundColor = "green";
    this.div.style.position = "absolute";
    this.div.style.top = "200px";
    this.div.style.left = "400px";
    this.div.id = "dialog-box";
  }
  show(children) {
    // 销毁
    const dom = document.querySelector("#dialog-box");
    if(!dom){ //兼容多次点击
      // 显示
      document.body.appendChild(this.div);
      ReactDOM.render(children, this.div);
    }
  }
  destroy() {
    // 销毁
    const dom = document.querySelector("#dialog-box");
    if(dom){//兼容多次点击
      ReactDOM.unmountComponentAtNode(this.div);
      dom.parentNode.removeChild(dom);
    }
  }
}
export default {
  show: function(children) {
    new Dialog().show(children);
  },
  hide: function() {
    new Dialog().destroy();
  }
};
```
nineteen.jsx
``` 
twoSubmit=()=>{
    Dialog.show('这是弹层2')
  }

  twoCancel=()=>{
    Dialog.hide()
  }
```
## React.memo
当类组件的输入属性相同时，可以使用 pureComponent 或 shouldComponentUpdate 来避免组件的渲染
``` 
import React from "react";

function areEqual(prevProps, nextProps) {
  /*
  如果把 nextProps 传入 render 方法的返回结果与
  将 prevProps 传入 render 方法的返回结果一致则返回 true，
  否则返回 false
  */
  if (prevProps.val === nextProps.val) {
    return true;
  } else {
    return false;
  }
}

// React.memo()两个参数,第一个是纯函数,第二个是比较函数
export default React.memo(function twentyChild(props) {
  console.log("MemoSon rendered : " + Date.now());
  return <div>{props.val}</div>;
}, areEqual);
```
## React.PureComponent
作用:
1. React.PureComponent 和 React.Component类似，都是定义一个组件类。
2. 不同是React.Component没有实现shouldComponentUpdate()，而 React.PureComponent通过props和state的浅比较实现了。
3. React.PureComponent是作用在类中,而React.memo是作用在函数中。
4. 如果组件的props和state相同时，render的内容也一致，那么就可以使用React.PureComponent了,这样可以提高组件的性能
``` 
class TwentyOneChild extends React.PureComponent{  //组件直接继承React.PureComponent
  render() {
    return <div>{this.props.name}</div>
  }
}

export default class TwentyOne extends React.Component{
    render(){
        return (
            <div>
              <TwentyOneChild name={'这是React.PureComponent的使用方法'}></TwentyOneChild>
            </div>
        )
    }
}
```
## React.Component
基于ES6 class的React组件,React允许定义一个class或者function作为组件，那么定义一个组件类，就需要继承React.Component
``` 
export default class TwentyTwo extends React.Component{ //组件定义方法
    render(){
        return (
            <div>这是技巧22</div>
        )
    }
}
```
## 在JSX 打印 falsy 值
定义:
1. falsy 值 (虚值) 是在 Boolean 上下文中认定为 false 的值;
2. 值有 0,"",'',``,null,undefined,NaN
``` 
export default class TwentyThree extends React.Component{
    state={myVariable:null}
    render(){
        return (
            <div>{String(this.state.myVariable)}</div>
        )
    }
}
```
虚值如果直接展示,会发生隐式转换,为 false,所以页面不显示
## ReactDOM.createPortal
组件的render函数返回的元素会被挂载在它的父级组件上,createPortal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案
``` 
import React from "react";
import ReactDOM from "react-dom";
import {Button} from "antd"

const modalRoot = document.body;

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement("div");
    this.el.style.width = "200px";
    this.el.style.height = "200px";
    this.el.style.backgroundColor = "green";
    this.el.style.position = "absolute";
    this.el.style.top = "200px";
    this.el.style.left = "400px";
  }

  componentDidMount() {
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(this.props.children, this.el);
  }
}

function Child() {
  return (
    <div className="modal">
      这个是通过ReactDOM.createPortal创建的内容
    </div>
  );
}

export default class TwentyFour extends React.Component {
  constructor(props) {
    super(props);
    this.state = { clicks: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      clicks: prevState.clicks + 1
    }));
  }

  render() {
    return (
      <div>
          <Button onClick={this.handleClick}>点击加1</Button>
        <p>点击次数: {this.state.clicks}</p>
        <Modal>
          <Child />
        </Modal>
      </div>
    );
  }
}
```
这样元素就追加到指定的元素下面啦
## 在 React 使用innerHTML
有些后台返回是 html 格式字段,就需要用到 innerHTML 属性
``` 
export default class TwentyFive extends React.Component {
  render() {
    return (
      <div dangerouslySetInnerHTML={{ __html: "<span>这是渲染的 HTML 内容</span>" }}></div>
    );
  }
}
```
## React.createElement
``` 
// 源码
export default class TwentySix extends React.Component {
  render() {
    return (
      <div>
        {React.createElement(
          "div",
          { id: "one", className: "two" },
          React.createElement("span", { id: "spanOne" }, "这是第一个 span 标签"),
          React.createElement("br"),
          React.createElement("span", { id: "spanTwo" }, "这是第二个 span 标签")
        )}
      </div>
    );
  }
}
```
原理:实质上 JSX 的 dom 最后转化为 js 都是React.createElement
``` 
// jsx 语法
<div id='one' class='two'>
    <span id="spanOne">this is spanOne</span>
    <span id="spanTwo">this is spanTwo</span>
</div>

// 转化为 js
React.createElement(
  "div",
 { id: "one", class: "two" },
 React.createElement( "span", { id: "spanOne" }, "this is spanOne"), 
 React.createElement("span", { id: "spanTwo" }, "this is spanTwo")
);
```
## React.cloneElement
复制组件,给组件传值或者添加属性
``` 
React.Children.map(children, child => {
  return React.cloneElement(child, {
    count: _this.state.count
  });
});
```
## React.Fragment
聚合一个子元素列表，并且不在DOM中增加额外节点
``` 
render() {
    const { info } = this.state;
    return (
      <div>
        {info.map((item, index) => {
          return (
            <React.Fragment key={index}>
              <div>{item.name}</div>
              <div>{item.age}</div>
            </React.Fragment>
          );
        })}
      </div>
    );
  }

```
## 循环元素
``` 
{arr.map((item,index)=>{
  return(
    <div key={item.id}>
      <span>{item.name}</span>
      <span>{item.age}</span>
    </div>
  )
})}
```
## 给 DOM 设置和获取自定义属性
``` 
export default class Thirty extends React.Component {
  click = e => {
    console.log(e.target.getAttribute("data-row"));
  };

  render() {
    return (
      <div>
        <div data-row={"属性1"} data-col={"属性 2"} onClick={this.click}>
          点击获取属性
        </div>
      </div>
    );
  }
```
## 绑定事件
``` 
import React from "react";
import { Button } from 'antd'

export default class Three extends React.Component {
  state = {
    flag: true,
    flagOne: 1
  };
  click(data1,data2){
    console.log('data1 值为',data1)
    console.log('data2 值为',data2)
  }
  render() {
    return (
      <div>
        <Button type="primary" onClick={this.click.bind(this,'参数 1','参数 2')}>点击事件</Button>
      </div>
    );
  }
}
```
## React-Router
### V3和 V4的区别
1. V3或者说V早期版本是把router 和 layout components 分开;
2. V4是集中式 router,通过 Route 嵌套，实现 Layout 和 page 嵌套,Layout 和 page 组件 是作为 router 的一部分;
3. 在V3 中的 routing 规则是 exclusive，意思就是最终只获取一个 route;
4. V4 中的 routes 默认是 inclusive 的，这就意味着多个;   可以同时匹配和呈现.如果只想匹配一个路由，可以使用Switch，在  中只有一个  会被渲染，同时可以再在每个路由添加exact，做到精准匹配
Redirect，浏览器重定向，当多有都不匹配的时候，进行匹配
### 使用
``` 
import { HashRouter as Router, Switch  } from "react-router-dom";

class App extends React.Component{
    render(){
        const authPath = '/login' // 默认未登录的时候返回的页面，可以自行设置
        let authed = this.props.state.authed || localStorage.getItem('authed') // 如果登陆之后可以利用redux修改该值
        return (
            <Router>
                <Switch>
                    {renderRoutes(routes, authed, authPath)}
                </Switch>
            </Router>
        )
    }
}
```
V4是通过 Route 嵌套，实现 Layout 和 page 嵌套,Switch切换路由的作用
## 样式引入方法
内联方式
``` 
import React from 'react';

const Header = () => {

    const heading = '头部组件'

    return(
        <div style={{backgroundColor:'orange'}}>
            <h1>{heading}</h1>
        </div>
    )
}

或者
import React from 'react';

const footerStyle = {
    width: '100%',
    backgroundColor: 'green',
    padding: '50px',
    font: '30px',
    color: 'white',
    fontWeight: 'bold'
}

export const Footer = () => {
    return(
        <div style={footerStyle}>
            底部组件
        </div>
    )
}
```
## 动态绑定 className
``` 
render(){
  const flag=true
  return (
    <div className={flag?"active":"no-active"}>这是技巧 34</div>
  )
}
```

原文: 
[React 开发必须知道的 34 个技巧](https://juejin.cn/post/6844903993278201870)
