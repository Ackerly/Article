# 弄懂react的基本使用和高级特性
## React的基本使用
### JSX基本使用
**变量、表达式**  
第一种类型：获取变量、插值  
``` 
import React from'react'

class JSXBaseDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'ces',
            imgUrl: 'https://p3-passport.byteacctimg.com/img/user-avatar/cc88f43a329099d65898aff670ea1171~300x300.image',
            flag: true
        }
    }
    render() {
        // 获取变量 插值
        const pElem = <p>{this.state.name}</p>
        return pElem
    }
}

exportdefault JSXBaseDemo
```
react 中插值的形式是单个花括号 {} 的形式。最终浏览器显示的效果如下
第二种类型：表达式
``` 
import React from'react'

class JSXBaseDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'ces',
            imgUrl: 'https://p3-passport.byteacctimg.com/img/user-avatar/cc88f43a329099d65898aff670ea1171~300x300.image',
            flag: true
        }
    }
    render() {
        // 表达式
        const exprElem = <p>{this.state.flag ? 'yes' : 'no'}</p>
        return exprElem
    }
}

exportdefault JSXBaseDemo
```
**class和style**  
如果要给某一个标签设置类名，那么会给该标签加上一个 class 。而在 react 中，如果想要给一个标签加上一个类，那么需要给其加上 className 。具体代码如下：  
``` 
import React from'react'
import'./style.css'
import List from'../List'

class JSXBaseDemo extends React.Component {
    render() {
        // class
        const classElem = <p className="title">设置 css class</p>

        // style
        const styleData = { fontSize: '30px',  color: 'blue' }
        const styleElem1 = <p style={styleData}>设置 style</p>
        // 内联写法，注意 {{ 和 }}
        const styleElem2 = <p style={{ fontSize: '30px',  color: 'blue' }}>设置 style</p>
       
		// 返回结果
		return [ classElem, styleElem1, styleElem2 ]
    }
}

exportdefault JSXBaseDemo
```
同时需要注意的是，在 react 中，如果要在标签里面写「内联样式」，那么需要使用「双花括号」 {{}} 来表示  

**子元素和组件**  
第一种类型：子元素  
对于子元素来说，它可以在标签里面进行使用。如下代码所示：
``` 
import React from'react'

class JSXBaseDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'test',
            imgUrl: 'https://p3-passport.byteacctimg.com/img/user-avatar/cc88f43a329099d65898aff670ea1171~300x300.image',
            flag: true
        }
    }
    render() {
        // 子元素
        const imgElem = <div>
            <p>我的头像</p>
            <img src="xxxx.png"/>
            <img src={this.state.imgUrl}/>
        </div>
        return imgElem
    }
}

exportdefault JSXBaseDemo
```
第二种类型：加载组件  
``` 
import React from'react'
import'./style.css'
import List from'../List'

class JSXBaseDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'test',
            imgUrl: 'https://p3-passport.byteacctimg.com/img/user-avatar/cc88f43a329099d65898aff670ea1171~300x300.image',
            flag: true
        }
    }
    render() {
        // 加载组件
        const componentElem = <div>
            <p>JSX 中加载一个组件</p>
            <hr/>
            <List/>
        </div>
        return componentElem
    }
}

exportdefault JSXBaseDemo
```
List.js 组件的代码如下：  
``` 
import React from'react'

class List extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'React',
            list: ['a', 'b', 'c']
        }
    }
    render() {
        return<div>
            <p onClick={this.changeName.bind(this)}>{this.state.name}</p>
            <ul>{
                this.state.list.map((item, index) => {
                    return <li key={index}>{item}</li>
                })
            }</ul>
            <button onClick={this.addItem.bind(this)}>添加一项</button>
        </div>
    }
    changeName() {
        this.setState({
            name: 'test'
        })
    }
    addItem() {
        this.setState({
            list: this.state.list.concat(`${Date.now()}`) // 使用不可变值
        })
    }
}

exportdefault List
```
**原生 html**  
``` 

import React from'react'

class JSXBaseDemo extends React.Component {
    render() {
        // 原生 html
        const rawHtml = '<span>富文本内容<i>斜体</i><b>加粗</b></span>'
        const rawHtmlData = {
            // 把 rawHtml 赋值给 __html
            __html: rawHtml // 注意，必须是这种格式
        }
        const rawHtmlElem = <div>
            <p dangerouslySetInnerHTML={rawHtmlData}></p>
            <p>{rawHtml}</p>
        </div>
        return rawHtmlElem
    }
}

exportdefault JSXBaseDemo
```
如果要在 react 中使用原生 html ，那么必须使用 const rawHtmlData = { __html: rawHtml } 这种形式，才能将原生 html 代码给解析出来。否则的话， react 是无法正常将原生 html 解析出来的  

### 条件判断
**if else**  
``` 
import React from'react'
import'./style.css'

class ConditionDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            theme: 'black'
        }
    }
    render() {
        const blackBtn = <button className="btn-black">black btn</button>
        const whiteBtn = <button className="btn-white">white btn</button>

        // if else
        if (this.state.theme === 'black') {
            return blackBtn
        } else {
            return whiteBtn
        }
    }
}

exportdefault ConditionDemo
```
style.css 的代码  
``` 
.title {
    font-size: 30px;
    color: red;
}

.btn-white {
    color: #333;
}
.btn-black {
    background-color: #666;
    color: #fff;;
}
```
theme 设置为 black 时，最终显示的效果就是「黑色」。如果把 theme 设置为其他状态，那么最终显示的效果就是「白色」

**三元表达式**  
``` 
import React from'react'
import'./style.css'

class ConditionDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            theme: 'black'
        }
    }
    render() {
        
        const blackBtn = <button className="btn-black">black btn</button>
        const whiteBtn = <button className="btn-white">white btn</button>

        // 三元运算符
        return<div>
             { this.state.theme === 'black' ? blackBtn : whiteBtn }
        </div>
    }
}

exportdefault ConditionDemo
```
**逻辑运算符 && ||**  
``` 
import React from'react'
import'./style.css'

class ConditionDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            theme: 'black'
        }
    }
    render() {
        
        const blackBtn = <button className="btn-black">black btn</button>
        const whiteBtn = <button className="btn-white">white btn</button>

        // &&
        return<div>
            { this.state.theme === 'black' && blackBtn }
        </div>
    }
}

exportdefault ConditionDemo
```
### 渲染列表
**map 和 key**  
``` 
import React from'react'

class ListDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            list: [
                {
                    id: 'id-1',
                    title: '标题1'
                },
                {
                    id: 'id-2',
                    title: '标题2'
                },
                {
                    id: 'id-3',
                    title: '标题3'
                }
            ]
        }
    }
    render() {
        return<ul>
            { /* 类似于vue中的v-for */
                this.state.list.map(
                    (item, index) => {
                        // 这里的 key 和 Vue 的 key 类似，必填，不能是 index 或 random
                        return <li key={item.id}>
                            index {index}; id {item.id}; title {item.title}
                        </li>
                    }
                )
            }
        </ul>
    }
}

exportdefault ListDemo

```
### React的事件
**bind this**  
``` 
import React from'react'

class EventDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'zhangsan'
        }

        // 修改方法的 this 指向
        // 使用这个写法，组件初始化时，只执行一次bind
        this.clickHandler1 = this.clickHandler1.bind(this)
    }
    render() {
        // this - 使用 bind
        return<p onClick={this.clickHandler1}>
            {this.state.name}
        </p>
    }
    clickHandler1() {
        // console.log('this....', this) // this 默认是 undefined
        this.setState({
            name: 'lisi'
        })
    }
}

exportdefault EventDemo
```
另外一种关于 this 的绑定方法  
``` 
import React from'react'

class EventDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'zhangsan'
        }
    }
    render() {
       // this - 使用静态方法
        return<p onClick={this.clickHandler2}>
            clickHandler2 {this.state.name}
        </p>
    }
    // 静态方法，this 指向当前实例
    clickHandler2 = () => {
        this.setState({
            name: 'lisi'
        })
    }
}

exportdefault EventDemo
```
**关于 event 参数**  
``` 
import React from'react'

class EventDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'zhangsan'
        }
    }
    render() {
       // event
        return<a href="https://imooc.com/" onClick={this.clickHandler3}>
            click me
        </a>
    }
    // 获取 event （重点）
    clickHandler3 = (event) => {
        event.preventDefault() // 阻止默认行为
        event.stopPropagation() // 阻止冒泡
        console.log('target', event.target) // 指向当前元素，即当前元素触发
        console.log('current target', event.currentTarget) // 指向当前元素，假象！！！

        // 注意，event 其实是 React 封装的。可以把 __proto__.constructor 看成是 SyntheticEvent 组合事件
        console.log('event', event) // 不是原生的 Event ，原生的是 MouseEvent
        console.log('event.__proto__.constructor', event.__proto__.constructor)

        // 原生 event 如下。其 __proto__.constructor 是 MouseEvent
        console.log('nativeEvent', event.nativeEvent)
        console.log('nativeEvent target', event.nativeEvent.target)  // 指向当前元素，即当前元素触发
        console.log('nativeEvent current target', event.nativeEvent.currentTarget) // 指向 document ！！！

    }
}

exportdefault EventDemo
```
需要注意的点是：  
- event 是合成事件 SyntheticEvent ，它能够模拟出来 DOM 事件所有的能力；
- event 是React 封装出来的，而 event.nativeEvent 是原生事件对象；
- 所有的事件，都会被挂载到 document 上；
- React 中的事件，和 DOM 事件不一样，和 Vue 事件也不一样

**传递自定义参数**  
``` 
import React from'react'

class EventDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'zhangsan',
            list: [
                {
                    id: 'id-1',
                    title: '标题1'
                },
                {
                    id: 'id-2',
                    title: '标题2'
                },
                {
                    id: 'id-3',
                    title: '标题3'
                }
            ]
        }
    }
    render() {
       // 传递参数 - 用 bind(this, a, b)
        return<ul>{this.state.list.map((item, index) => {
            return <li key={item.id} onClick={this.clickHandler4.bind(this, item.id, item.title)}>
                index {index}; title {item.title}
            </li>
        })}</ul>
    }
    // 传递参数
    clickHandler4(id, title, event) {
        console.log(id, title)
        console.log('event', event) // 最后追加一个参数，即可接收 event
    }
}

exportdefault EventDemo
```
this.clickHandler4.bind(this, item.id, item.title) 这种形式来对 react 中的事件进行参数传递  

注意点:
- React 16 将事件绑定到 document 上；
- React 17 将事件绑定到 root 组件上；

### 表单
**受控组件**  
``` 
import React from'react'

class FormDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'test',
            info: '个人信息',
            city: 'GuangDong',
            flag: true,
            gender: 'female'
        }
    }
    render() {

        // 受控组件
        return<div>
            <p>{this.state.name}</p>
            <label htmlFor="inputName">姓名：</label> {/* 用 htmlFor 代替 for */}
            <input id="inputName" value={this.state.name} onChange={this.onInputChange}/>
        </div>
    }
    onInputChange = (e) => {
        this.setState({
            name: e.target.value
        })
    }
}

exportdefault FormDemo
```
在 react 中，通过使用 onChange 事件来手动修改 state 里面的值。  
**input textarea select 用value**  
textarea 相关的代码：  
``` 
import React from'react'

class FormDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'test',
            info: '个人信息',
            city: 'GuangDong',
            flag: true,
            gender: 'female'
        }
    }
    render() {

        // textarea - 使用 value
        return<div>
            <textarea value={this.state.info} onChange={this.onTextareaChange}/>
            <p>{this.state.info}</p>
        </div>
    }
    onTextareaChange = (e) => {
        this.setState({
            info: e.target.value
        })
    }
}

exportdefault FormDemo
```
textarea 也是用 value 和 onChange 来对「值」进行绑定  
继续来看 select 。具体代码如下：  
``` 
import React from'react'

class FormDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'test',
            info: '个人信息',
            city: 'GuangDong',
            flag: true,
            gender: 'female'
        }
    }
    render() {

        // textarea - 使用 value
        return<div>
            <textarea value={this.state.info} onChange={this.onTextareaChange}/>
            <p>{this.state.info}</p>
        </div>
    }
    onSelectChange = (e) => {
        this.setState({
            city: e.target.value
        })
    }
}

exportdefault FormDemo
```
**checkbox radio 用 checked**  
``` 
import React from'react'

class FormDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'test',
            info: '个人信息',
            city: 'GuangDong',
            flag: true,
            gender: 'female'
        }
    }
    render() {

        // checkbox
        return<div>
            <input type="checkbox" checked={this.state.flag} onChange={this.onCheckboxChange}/>
            <p>{this.state.flag.toString()}</p>
        </div>
    }
    onCheckboxChange = () => {
        this.setState({
            flag: !this.state.flag
        })
    }
}

exportdefault FormDemo
```
radio 也是类似，如下代码所示：
``` 
import React from'react'

class FormDemo extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: 'test',
            info: '个人信息',
            city: 'GuangDong',
            flag: true,
            gender: 'female'
        }
    }
    render() {

        // radio
        return<div>
            male <input type="radio" name="gender" value="male" checked={this.state.gender === 'male'} onChange={this.onRadioChange}/>
            female <input type="radio" name="gender" value="female" checked={this.state.gender === 'female'} onChange={this.onRadioChange}/>
            <p>{this.state.gender}</p>
        </div>
    }
    onRadioChange = (e) => {
        this.setState({
            gender: e.target.value
        })
    }
}

exportdefault FormDemo
```
### 组件使用
``` 
import React from'react'
import PropTypes from'prop-types'

class Input extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            title: ''
        }
    }
    render() {
        return<div>
            <input value={this.state.title} onChange={this.onTitleChange}/>
            <button onClick={this.onSubmit}>提交</button>
        </div>
    }
    onTitleChange = (e) => {
        this.setState({
            title: e.target.value
        })
    }
    onSubmit = () => {
        const { submitTitle } = this.props
        submitTitle(this.state.title)

        this.setState({
            title: ''
        })
    }
}
// props 类型检查
Input.propTypes = {
    submitTitle: PropTypes.func.isRequired
}

class List extends React.Component {
    constructor(props) {
        super(props)
    }
    render() {
        const { list } = this.props

        return<ul>{list.map((item, index) => {
            return <li key={item.id}>
                <span>{item.title}</span>
            </li>
        })}</ul>
    }
}
// props 类型检查
// 通过propTypes可以清楚地知道list需要一个什么类型的数据
List.propTypes = {
    list: PropTypes.arrayOf(PropTypes.object).isRequired
}

class Footer extends React.Component {
    constructor(props) {
        super(props)
    }
    render() {
        return<p>
            {this.props.text}
            {this.props.length}
        </p>
    }
    componentDidUpdate() {
        console.log('footer did update')
    }
    shouldComponentUpdate(nextProps, nextState) {
        if (nextProps.text !== this.props.text
            || nextProps.length !== this.props.length) {
            returntrue// 可以渲染
        }
        returnfalse// 不重复渲染
    }

    // React 默认：父组件有更新，子组件则无条件也更新！！！
    // 性能优化对于 React 更加重要！
    // SCU 一定要每次都用吗？—— 需要的时候才优化（SCU即shouldComponentUpdate）
}

// 父组件
class TodoListDemo extends React.Component {
    constructor(props) {
        super(props)
        // 状态（数据）提升
        // list的数据需要放在父组件
        this.state = {
            list: [
                {
                    id: 'id-1',
                    title: '标题1'
                },
                {
                    id: 'id-2',
                    title: '标题2'
                },
                {
                    id: 'id-3',
                    title: '标题3'
                }
            ],
            footerInfo: '底部文字'
        }
    }
    render() {
        return<div>
            <Input submitTitle={this.onSubmitTitle}/>
            <List list={this.state.list}/>
            <Footer text={this.state.footerInfo} length={this.state.list.length}/>
        </div>
    }
    onSubmitTitle = (title) => {
        this.setState({
            list: this.state.list.concat({
                id: `id-${Date.now()}`,
                title
            })
        })
    }
}

exportdefault TodoListDemo
```
**props 传递数据**  
最后一个 TodoListDemo 是父组件，其他都是子组件。在 Input 组件和 List 组件中，我们将 props 属性的内容，以 this.props.xxx 的方式，传递给「父组件」。   
**props 传递函数**  
React 在传递函数这一部分和 vue 是不一样的。对于 vue 来说，如果有一个父组件要传递函数给子组件，子组件如果想要触发这个函数，那么需要使用事件传递和 $emit 的方式来解决  
定位到 Input 组件中，将 submitTitle 以函数的形式，传递给父组件中的 onSubmitTitle  
**props 类型检查**  
使用 react 中的 PropTypes ，可以对当前所使用的属性进行一个类型检查。比如说：submitTitle: PropTypes.func.isRequired 表明的是， submitTitle 是一个函数，并且是一个必填项。  

### setState
**不可变值**  
所谓不可变值，即所设置的值永不改变。那这个时候，就需要去创建一个副本，来设置 state 的值  
state要在构造函数中定义  
``` 
import React from'react'

// 函数组件，默认没有 state
class StateDemo extends React.Component {
    constructor(props) {
        super(props)

        // 第一，state 要在构造函数中定义
        this.state = {
            count: 0
        }
    }
    render() {
        return<div>
            <p>{this.state.count}</p>
        </div>
    }
}

exportdefault StateDemo
```
第二点，不要直接修改state，要使用不可变值  
``` 
import React from'react'

// 函数组件，默认没有 state
class StateDemo extends React.Component {
    constructor(props) {
        super(props)

        // 第一，state 要在构造函数中定义
        this.state = {
            count: 0
        }
    }
    render() {
        return<div>
            <p>{this.state.count}</p>
        	<button onClick={this.increase}>累加</button>
        </div>
    }
    increase= () => {
        // 第二，不要直接修改 state，使用不可变值
        // this.state.count++ // 错误写法，会直接修改原来的值
        this.setState({
            count: this.state.count + 1// ShouldComponentUpdate → SCU
        })
    }
}

exportdefault StateDemo
```
在 react 中操作数组的值  
``` 
ist5Copy = this.state.list5.slice()
list5Copy.splice(2, 0, 'a') // 中间插入/删除
this.setState({
    list1: this.state.list1.concat(100), // 追加
    list2: [...this.state.list2, 100], // 追加
    list3: this.state.list3.slice(0, 3), // 截取
    list4: this.state.list4.filter(item => item > 100), // 筛选
    list5: list5Copy // 其他操作
})
// 注意，不能直接对 this.state.list 进行 push pop splice 等，这样违反不可变值
```
在 react 中操作对象的值  
``` 
// 不可变值 - 对象
this.setState({
    obj1: Object.assign({}, this.state.obj1, {a: 100}),
    obj2: {...this.state.obj2, a: 100}
})
// 注意，不能直接对 this.state.obj 进行属性设置，即 this.state.obj.xxx 这样的形式，这种形式会违反不可变值
```
**可能是异步更新**  
``` 
class StateDemo extends React.Component {
    constructor(props) {
        super(props)

        this.state = {
            count: 0
        }
    }
    render() {
        return<div>
            <p>{this.state.count}</p>
        	<button onClick={this.increase}>累加</button>
        </div>
    }
    increase= () => {
        // setState 可能是异步更新（也有可能是同步更新） 
        this.setState({
            count: this.state.count + 1
        }, () => {
            // 联想 Vue $nextTick - DOM
            console.log('count by callback', this.state.count) // 回调函数中可以拿到最新的 state
        })
        console.log('count', this.state.count) // 异步的，拿不到最新值
    }
}

exportdefault StateDemo
```
this.state 前半部分并不能同一时间得到更新，所以它是异步操作。而后面的箭头函数中的内容可以得到同步更新，所以后面函数的部分是同步操作  
setTimeout 在 setState 中是同步的  
``` 
import React from'react'

class StateDemo extends React.Component {
    constructor(props) {
        super(props)

        this.state = {
            count: 0
        }
    }
    render() {
        return<div>
            <p>{this.state.count}</p>
        	<button onClick={this.increase}>累加</button>
        </div>
    }
    increase= () => {
        // setTimeout 中 setState 是同步的
        setTimeout(() => {
            this.setState({
                count: this.state.count + 1
            })
            console.log('count in setTimeout', this.state.count)
        }, 0)
    }
}

exportdefault StateDemo
```
如果是自己定义的 DOM 事件，那么在 setState 中是同步的，用在 componentDidMount 中,如果是销毁事件，那么用在 componentWillMount 生命周期中。代码如下:  
``` 
import React from'react'

class StateDemo extends React.Component {
    constructor(props) {
        super(props)

        this.state = {
            count: 0
        }
    }
    render() {
        return<div>
            <p>{this.state.count}</p>
        	<button onClick={this.increase}>累加</button>
        </div>
    }
    bodyClickHandler = () => {
        this.setState({
            count: this.state.count + 1
        })
        console.log('count in body event', this.state.count)
    }
    componentDidMount() {
        // 自己定义的 DOM 事件，setState 是同步的
        document.body.addEventListener('click', this.bodyClickHandler)
    }
    componentWillUnmount() {
        // 及时销毁自定义 DOM 事件
        document.body.removeEventListener('click', this.bodyClickHandler)
        // clearTimeout
    }
}

exportdefault StateDemo
```
**可能会被合并**  
setState 在传入对象时，更新前会被合并。来看一段代码：
``` 
import React from'react'

class StateDemo extends React.Component {
    constructor(props) {
        super(props)

        this.state = {
            count: 0
        }
    }
    render() {
        return<div>
            <p>{this.state.count}</p>
        	<button onClick={this.increase}>累加</button>
        </div>
    }
    increase= () => {
        // 传入对象，会被合并（类似 Object.assign ）。执行结果只一次 +1
        this.setState({
            count: this.state.count + 1
        })
        this.setState({
            count: this.state.count + 1
        })
        this.setState({
            count: this.state.count + 1
        })
    }
}

exportdefault StateDemo
```
一下子多个三个 setState ，那结果应该是 +3 才是。但其实，如果传入的是对象，那么结果会把三个合并为一个，最终只执行一次
还有另外一种情况，如果传入的是「函」数，那么结果不会被合并。来看一段代码：  
``` 
import React from'react'

class StateDemo extends React.Component {
    constructor(props) {
        super(props)

        this.state = {
            count: 0
        }
    }
    render() {
        return<div>
            <p>{this.state.count}</p>
        	<button onClick={this.increase}>累加</button>
        </div>
    }
    increase= () => {
        // 传入函数，不会被合并。执行结果是 +3
        this.setState((prevState, props) => {
            return {
                count: prevState.count + 1
            }
        })
        this.setState((prevState, props) => {
            return {
                count: prevState.count + 1
            }
        })
        this.setState((prevState, props) => {
            return {
                count: prevState.count + 1
            }
        })
    }
}

exportdefault StateDemo
```
如果传入的是函数，那么结果一下子就执行三次了。  
**组件生命周期**  
react 的组件生命周期，有单组件声明周期和父子组件声明周期。其中，父子组件生命周期与 Vue 类似  

## React高级特性
### 函数组件
``` 
// class 组件
class List extends React.Component {
    constructor(props) {
        super(props)
    }
    redner() {
        const { list } = this.props
        
        return<ul>{list.map((item, index) => {
            return <li key={item.id}>
                	<span>{item.title}</span>
                </li>
        })}</ul>
    }
}
```
函数组件的形式如下：  
``` 
// 函数组件
function List(props) {
    const { list } = this.props
    
    return<ul>{list.map((item, idnex) => {
        return <li key={item.id}>
            	<span>{item.title}</span>
            </li>
    })}</ul>
}
```
函数组件具有以下特点：
- 只是一个「纯函数」，它输入的是 props ，输出的是 JSX ；
- 函数组件「没有实例」，「没有生命周期」，也「没有 state」 ；
- 函数组件「不能扩展其他方法」

### 非受控组件
非受控组件，就是 input 里面的值，不受到 state 的控制  
**input**  
``` 
import React from'react'

class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: '星期一研究室',
            flag: true,
        }
        this.nameInputRef = React.createRef() // 创建 ref
    }
    render() {
        // input defaultValue
        return<div>
            {/* 使用 defaultValue 而不是 value ，使用 ref */}
            <input defaultValue={this.state.name} ref={this.nameInputRef}/>
            {/* state 并不会随着改变 */}
            <span>state.name: {this.state.name}</span>
            <br/>
            <button onClick={this.alertName}>alert name</button>
        </div>

    }
    alertName = () => {
        const elem = this.nameInputRef.current // 通过 ref 获取 DOM 节点
        alert(elem.value) // 不是 this.state.name
    }
}

exportdefault App
```
如果是非受控组件，那么需要使用 defaultValue 去控制组件的值。且最终 input 框里面的内容不论我们怎么改变，都不会影响到 state 的值  
**checkbox**  
``` 
import React from'react'

class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: '星期一研究室',
            flag: true,
        }
    }
    render() {
        // checkbox defaultChecked
        return<div>
            <input
                type="checkbox"
                defaultChecked={this.state.flag}
            />
           	<p>state.name: { this.state.flag === true ? 'true' : 'false' }</p>
        </div>
    }
}

exportdefault App
```
复选框如果当非受控组件来使用的使用，那么使用 defaultCkecked 来对值进行控制  
**file**  
``` 
import React from'react'

class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            name: '星期一研究室',
            flag: true,
        }
        this.fileInputRef = React.createRef()
    }
    render() {
        // file
        return<div>
            <input type="file" ref={this.fileInputRef}/>
            <button onClick={this.alertFile}>alert file</button>
        </div>
    }
    alertFile = () => {
        const elem = this.fileInputRef.current // 通过 ref 获取 DOM 节点
        alert(elem.files[0].name)
    }
}

exportdefault App
```
通过 ref 去获取 DOM 节点，接着去获取到文件的名字。像 file 这种类型的固定，值并不会一直固定的，所以也是一个非受控组件

**总结梳理**  
非受控组件的几大使用场景  
- 必须手动操作 DOM 元素， setState 并无法手动操作 DOM 元素；
- 文件上传类型 <input type=file> ；
- 某些富文本编辑器，需要传入 DOM 元素

受控组件 vs 非受控组件的区别如下：  
- 优先使用受控组件，符合 React 设计原则；
- 必须操作 DOM 时，再使用非受控组件

### Protals
**为什么要用 Protals**  
组件默认会按照既定层次嵌套渲染。类似下面这样：  
``` 
<div id="root">
    <div>
        <div>
            <div class="model">Modal内容</div>
        </div>
    </div>
</div>
```
这样不断嵌套，但里面却只有一层区域的内容是有用的。从某种程度上来说，是非常不好的。那我们想做的事情是，如何让组件渲染到父组件以外呢？  
这个时候就需要用到 Protals  
**如何使用**  
``` 
import React from'react'
import ReactDOM from'react-dom'
import'./style.css'

class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
        }
    }
    render() {
        // 正常渲染
        // return <div className="modal">
        //     {this.props.children} {/* vue slot */}
        // </div>

        // 使用 Portals 渲染到 body 上。
        // fixed 元素要放在 body 上，有更好的浏览器兼容性。
        return ReactDOM.createPortal(
            <div className="modal">{this.props.children}</div>,
            document.body // DOM 节点
        )
    }
}

exportdefault App
```
style.css 代码如下：  
``` 
.modal {
    position: fixed;
    width: 300px;
    height: 100px;
    top: 100px;
    left: 50%;
    margin-left: -150px;
    background-color: #000;
    /* opacity: .2; */
    color: #fff;
    text-alig
}
```
通过使用 ReactDOM.createPortal() ，来创建 Portals 。最终 modals 节点成功脱离开「父组件」，并渲染到组件外部  
**使用场景**  
protals 常用于解决一些 css 兼容性问题。通常使用场景有：  
- overflow:hidden; 触发 bfc ；
- 父组件 z-index 值太小；
- position:fixed 需要放在 body 第一层级

### context
**使用场景**  
经常会有一些场景出现切换的频率很频繁，比如语言切换、或者是主题切换，那如何把对应的切换信息给有效地传递给每个组件呢？  
使用 props ，又有点繁琐；使用 redux ，又太小题大做了。因此，这个需要我们可以用 react 中的 context  
**举例阐述**  
``` 
import React from'react'

// 创建 Context 填入默认值（任何一个 js 变量）
const ThemeContext = React.createContext('light')

// 底层组件 - 函数是组件
function ThemeLink (props) {
    // const theme = this.context // 会报错。函数式组件没有实例，即没有 this

    // 函数式组件可以使用 Consumer
    return<ThemeContext.Consumer>
        { value => <p>link's theme is {value}</p> }
    </ThemeContext.Consumer>
}

// 底层组件 - class 组件
class ThemedButton extends React.Component {
    // 指定 contextType 读取当前的 theme context。
    // static contextType = ThemeContext // 也可以用 ThemedButton.contextType = ThemeContext
    render() {
        const theme = this.context // React 会往上找到最近的 theme Provider，然后使用它的值。
        return<div>
            <p>button's theme is {theme}</p>
        </div>
    }
}
ThemedButton.contextType = ThemeContext // 指定 contextType 读取当前的 theme context。

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar(props)
    return (
        <div>
            <ThemedButton />
            <ThemeLink />
        </div>
    )
}

class App extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            theme: 'light'
        }
    }
    render() {
        return<ThemeContext.Provider value={this.state.theme}>
            <Toolbar />
            <hr/>
            <button onClick={this.changeTheme}>change theme</button>
        </ThemeContext.Provider>
    }
    changeTheme = () => {
        this.setState({
            theme: this.state.theme === 'light' ? 'dark' : 'light'
        })
    }
}

exportdefault App
```
首先创建了一个 Context ，也就是 ThemeContext ，并传入了 light 值。  
其次，核心在 <Toolbar /> 组件。Toolbar 现有组件为 ThemedButton 和 ThemeLink 。其中，我们先指定 ThemedButton 的 contextType 去读取当前的 ThemeContext ，那么就取到了默认值 light 。  
接着，来到了 ThemeLink 组件。ThemeLink 是一个「函数式组件」，因此，我们可以直接使用 ThemeContext.Consumer 来对其进行传值。  
上面两个组件的值都取到了，但那只是 ThemeContext 的初始值。取到值了之后呢，我们还要修改值， React 会往上找到最近的 ThemeContext.Provider ，通过 value={this.state.theme} 这种方式，去修改和使用 ThemeContext 最终使用的值 。   
**异步组件（懒加载）**  
在项目开发时，我们总是会不可避免的去加载一些大组件，这个时候就需要用到「异步加载」。在 vue 中，我们通常使用 import() 来加载异步组件，但在 react 就不这么使用了。  
React 通常使用 React.lazy 和 React.Suspense 来加载大组件  
``` 
import React from'react'

const ContextDemo = React.lazy(() =>import('./ContextDemo'))

class App extends React.Component {
    constructor(props) {
        super(props)
    }
    render() {
        return<div>
            <p>引入一个动态组件</p>
            <hr />
            <React.Suspense fallback={<div>Loading...</div>}>
                <ContextDemo/>
            </React.Suspense>
        </div>

        // 1. 强制刷新，可看到 loading （看不到就限制一下 chrome 的网速，Performance的network）
        // 2. 看 network 的 js 加载
    }
}

exportdefault App
```
首先我们使用 import 去导入我们要加载的组件。之后使用 React.lazy 去将这个组件给进行注册，也就是 ContextDemo 。最后使用 React.Suspense 来加载 ContextDemo 。至此，我们完成了该异步组件的加载。  
### 性能优化
**shouldComponentUpdate（简称SCU）**  
``` 
shouldComponentUpdate(nextProps, nextState) {
    if (nextState.count !== this.state.count
       	|| nextProps.text !== this.props.length) {
        returntrue// 可以渲染
    }
    returnfalse// 不重复渲染
}
```
在 React 中，默认的是，当父组件有更新，子组件也无条件更新。那如果每回都触发更新，肯定不太好  
这个时候我们需要用到 shouldComponentUpdate ，判断当属性有发生改变时，可以触发渲染。当属性不发生改变时，也就是前后两次的值相同时，就不触发渲染  
SCU 的使用方式具体如下：  
- SCU默认返回 true ，即 React 默认重新渲染所有子组件；
- 必须配合 「“不可变值”」 一起使用；
- 可先不用 SCU ，有性能问题时再考虑使用

**PureComponent和React.memo**  
PureComponent 在 react 中的使用形式如下：  
``` 
class List extends React.PureComponent {
    constructor(props) {
        super(props)
    }
    render() {
        const { list } = this.props

        return<ul>{list.map((item, index) => {
            return <li key={item.id}>
                <span>{item.title}</span>
            </li>
        })}</ul>
    }
    shouldComponen
```
如果我们使用了 PureComponent ，那么 SCU 会进行浅层比较，也就是一层一层的比较下去。  

memo ，顾名思义是备忘录的意思。在 React 中的使用形式如下：
``` 
function MyComponent(props) {
    /* 使用props 渲染 */
}

function areEqual(prevProps, nextProps) {
    /*
    如果把 nextProps传入render方法的返回结果 与
     preProps传入render方法的返回结果 一致的话，则返回true，
    否则返回false
    */
}
exportdefault React.memo(MyComponent, areEqual);
```
memo ，可以说是函数组件中的 PureComponent 。同时，使用 React.memo() 的形式，将我们的函数组件和 areEqual 的值进行比较，最后返回一个新的函数
在 React 中，浅比较已经使用于大部分情况，一般情况下，尽量不要做深度比较  
**不可变值**  
在 React 中，用于做不可变值的有一个库：Immutable.js，这个库有以下几大特点：  
- 彻底拥抱“不可变值”
- 基于共享数据（不是深拷贝），速度好
- 有一定的学习和迁移成本，按需使用

``` 
const map1 = Immutable.Map({ a: 1, b: 2, c: 3 })
const map2 = map1.set('b', 50)
map1.get('b') // 2
map2.get('b') // 50
```
### 关于组件公共逻辑的抽离
在 React 中，对于组件公共逻辑的抽离主要有三种方式要了解。具体如下：  
- mixin ，已被 React 弃用
- 高阶组件 HOC
- Render Props

**高阶组件 HOC**  
``` 
// 高阶组件不是一种功能，而是一种设计模式
// 1.传入一个组件 Component
const HOCFactory = (Component) => {
    class HOC extends React.Component {
        // 在此定义多个组件的公共逻辑
        render() {
            // 2.返回拼接的结果
            return<Component {this.props} />
        }
    }
    return HOC
}
const EnhancedComponent1 = HOCFactory(WrappedComponent1)
const EnhancedComponent2 = HOCFactory(WrappedComponent2)
```
高阶组件 HOC 是传入一个组件，返回一个新的组件  
``` 
import React from'react'

// 高阶组件
const withMouse = (Component) => {
    class withMouseComponent extends React.Component {
        constructor(props) {
            super(props)
            this.state = { x: 0, y: 0 }
        }
  
        handleMouseMove = (event) => {
            this.setState({
                x: event.clientX,
                y: event.clientY
            })
        }
  
        render() {
            return (
                <div style={{ height: '500px' }} onMouseMove={this.handleMouseMove}>
                    {/* 1. 透传所有 props 2. 增加 mouse 属性 */}
                    <Component {...this.props} mouse={this.state}/>
                </div>
            )
        }
    }
    return withMouseComponent
}

const App = (props) => {
    const { x, y } = props.mouse // 接收 mouse 属性
    return (
        <div style={{ height: '500px' }}>
            <h1>The mouse position is ({x}, {y})</h1>
        </div>
            )
}

exportdefault withMouse(App) // 返回高阶函数
```
在上面的代码中，定义了高阶组件 withMouse ，之后它通过 <Component {...this.props} mouse={this.state}/> 这种形式，将参数props和props 的 mouse 属性给透传出来，供子组件 App 使用。  
在 react 中，还有一个比较常见的高阶组件是 redux connect 。用一段代码来演示：  
``` 
import { connect } from'react-redux';

// connect 是高阶组件
const VisibleTodoList = connect(
	mapStateToProps,
    mapDispatchToProps
)(TodoList)

exportdefault VisibleTodoList
```
connect 的源码具体如下：  
``` 
exportconst connect =(mapStateToProps, mapDispatchToProps) =>(WrappedComponent) => {
    class Connect extends Component {
        constructor() {
            super()
            this.state = {
                allProps: {}
            }
        }
        /* 中间省略 N 行代码 */
        render () {
			return<WrappedComponent {...this.state.allProps} />
        }
    }
    return Connect
}
```
Connect 也是同样地，传入一个组件，并返回一个组件  

**Render Props**  
``` 
// Render Props 的核心思想
// 1.通过一个函数，将class组件的state作为props，传递给纯函数组件
class Factory extends React.Component {
    constructor() {
        tihs.state = {
            /* state 即多个组件的公共逻辑的数据 */
        }
    }
    /* 2.修改 state */
    render() {
        return<div>{this.props.render(this.state)}</div>
    }
}

const App = () => {
    // 3.在这里使用高阶组件，同时将高阶组件中的render属性传递进来
    <Factory render={
        /* render 是一个函数组件 */
        (props) => <p>{props.a}{props.b} …</p>
    } />
}

exportdefault App;
```
高阶组件 HOC 中，最终返回的结果也是一个高阶组件。但在 Render Props 中，我们把 Factory 包裹在定义的 App 组件中，最终再把 App 返回。  
值得注意的是，在 Vue 中有类似于高阶组件的用法，但没有像 Render Props 类似的用法  

``` 
import React from'react'
import PropTypes from'prop-types'

class Mouse extends React.Component {
    constructor(props) {
        super(props)
        this.state = { x: 0, y: 0 }
    }
  
    handleMouseMove = (event) => {
      this.setState({
        x: event.clientX,
        y: event.clientY
      })
    }
  
    render() {
      return (
        <div style={{ height: '500px' }} onMouseMove={this.handleMouseMove}>
            {/* 将当前 state 作为 props ，传递给 render （render 是一个函数组件） */}
            {this.props.render(this.state)}
        </div>
      )
    }
}
Mouse.propTypes = {
    render: PropTypes.func.isRequired // 必须接收一个 render 属性，而且是函数
}

const App = (props) => (
    <div style={{ height: '500px' }}>
        <Mouse render={
            /* render 是一个函数组件 */
            ({ x, y }) => <h1>The mouse position is ({x}, {y})</h1>
        }/>
        
    </div>
)

/**
 * 即，定义了 Mouse 组件，只有获取 x y 的能力。
 * 至于 Mouse 组件如何渲染，App 说了算，通过 render prop 的方式告诉 Mouse 。
 */

exportdefault App
```
在上面的代码中，通过 this.props.render(this.state) 这种形式，将 Mouse 组件中的属性传递给 App ，并让 App 成功使用到 Mouse 的属性值  

**HOC vs Render Props**  
- HOC：模式简单，但会增加组件层级
- Render Props：代码简洁，学习成本较高

## Redux和React-router
### Redux
**Redux概念简述**  
react来说，它是一个非视图层的轻量级框架，如果要用它来传递数据的话，则要先父传子，然后再慢慢地一层一层往上传递  
但如果用 redux 的话，假设我们想要「某个组件的数据」，那这个组件的数据则会通过 redux 来存放到 store 中进行管理。之后呢，通过 store ，再来将数据一步步地往下面的组件进行传递  
可以视 Redux 为 Reducer 和 Flux 的结合  
**Redux的工作流程**  
Redux ，实际上就是一个数据层的框架，它把所有的数据都放在了 store 之中  
可以看到中间的 store ，它里面就存放着所有的数据。继续看 store 向下的箭头，然后呢，每个组件都要向 store 里面去拿数据  
用一个例子来梳理，具体如下：
- 整张图上有一个 store ，它存放着所有的数据，也就是「存储数据的公共区域」；
- 每个组件，都要从 store 里面拿数据；
- 假设现在有一个场景，模拟我们要在图书馆里面借书。那么我们可以把 react Component 理解为借书人，之后呢，借书人要去找图书馆管理员才能借到这本书。而借书这个过程中数据的传递，就可以把它视为是 Action Creators ，可以理解为 「“你想要借什么书”」 这句话。
- Action Creatures 去到 store 。这个时候我们把 store 当做是「图书馆管理员」，但是，图书馆管理员是没有办法记住所有图书的数据情况的。一般来说，它都需要一个记录本，你想要借什么样的书，那么她就先查一下；又或者你想要还什么书，她也要查一下，需要放回什么位置上。
- 这个时候就需要跟 reducers 去通信，我们可以把 reducers 视为是一个「记录本」，图书馆管理员用这个「记录本」来记录需要的数据。管理员 store 通过 reducer 知道了应该给借书人 Components 什么样的数据。

**react-redux**  
React-redux 中要了解的几个点是 Provider 、 Connect 、 mapStateToProps 和 mapDisptchToProps  
``` 
import React from'react'
import { Provider } from'react-redux'
import { createStore } from'redux'
import todoApp from'./reducers'
import App from'./components/App'

let store = createStore(todoApp)

exportdefaultfunction() {
    return<Provider store={store}>
        	<App />
        </Provider>
}
```
react-redux 提供了 Provider 的能力，可以看到最后部分的代码， Provider 将 <App /> 包裹起来，其实也就是说为它包裹的所有组件提供 store 能力，这也是 Provider 发挥的作用  

``` 
import { connect } from'react-redux'
import { toggleTodo } from'../actions'
import TodoList from'../components/TodoList'

// 不同类型的 todo 列表
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case'SHOW_ALL':
      return todos
    case'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = state => {
  // state 即 vuex 的总状态，在 reducer/index.js 中定义
  return {
    // 根据完成状态，筛选数据
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
const mapDispatchToProps = dispatch => {
  return {
    // 切换完成状态
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  }
}

// connect 高阶组件，将 state 和 dispatch 注入到组件 props 中
const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

exportdefault VisibleTodoList
```
在上面的代码中， connect 将 state 和 dispatch 给注入到组件的 props 中，将属性传递给到 TodoList 组件  
**异步action**  
redux 中的同步 action 如下代码所示：  
``` 
// 同步 action
exportconst addTodo = text => {
    // 返回 action 对象
    return {
        type: 'ADD_TODO',
        id: nextTodoId++,
        text
    }
}
```
redux 中的异步action 如下代码所示：  
``` 
// 异步 action
exportconst addTodoAsync = text => {
    // 返回函数，其中有 dispatch 参数
    return(dispatch) => {
        // ajax 异步获取数据
        fetch(url).thne(res => {
            // 执行异步 action
            dispatch(addTodo(res.text))
        })
    }
}
```
## React-router
### 路由模式  
React-router 和 vue-router 一样，都是两种模式。具体如下：  
- hash 模式（默认），如 http://abc.com/#/user/10
- H5 history 模式，如 http://abc.com/user/20
- 后者需要 server 端支持，因此无特殊需求可选择前者

hash模式的路由配置如下代码所示：  
``` 
import React from'react'
import {
    HashRouter as Router,
    Switch,
    Route
} from'react-router-dom'

function RouterComponent() {
    return(
    	<Router>
        	<Switch>
        		<Route exact path="/">
        			<Home />
        		</Route>
        		<Route exact path="/project/:id">
        			<Project />
        		</Route>
        		<Route path="*">
        			<NotFound />
        		</Route>
        	</Switch>
        </Router>
    )
}
```
History模式的路由配置如下：  
``` 
import React from'react'
import {
    BrowserRouter as Router,
    Switch,
    Route
} from'react-router-dom'

function RouterComponent() {
    return(
    	<Router>
        	<Switch>
        		<Route exact path="/">
        			<Home />
        		</Route>
        		<Route exact path="/project/:id">
        			<Project />
        		</Route>
        		<Route path="*">
        			<NotFound />
        		</Route>
        	</Switch>
        </Router>
    )
}
```
hash 和 history 的区别在于import中的 HashRouter 和 BrowserRouter  
### 路由配置
**动态路由**  
现在有父组件 RouterComponent  
``` 
function RouterComponent() {
    return(
    	<Router>
        	<Switch>
        		<Route exact path="/">
        			<Home />
        		</Route>
        		<Route exact path="/project/:id">
        			<Project />
        		</Route>
        		<Route path="*">
        			<NotFound />
        		</Route>
        	</Switch>
        </Router>
    )
}
```
在这个组件中还有一个 Project 组件，需要进行动态传参  
子组件 Project 组件时如何进行动态传参的。具体代码如下：  
``` 
import React from'react'
import { Link, useParams } from'react-router-dom'

function Project() {
    // 获取 url 参数，如 '/project/100'
    const { id } = useParams()
    console.log('url param id', id)
    
    return (
    	<div>
        	<Link to="/">首页</Link>
        </div>
    )
}
```
在 React 中，通过 const { id } = useParams() 这样的形式，来进行动态传参。  
另外一种情况是跳转路由  
``` 
import React from'react'
import { useHistory } from'react-router-dom'

function Trash() {
    let history = useHistory()
    function handleClick() {
        history.push('/')
    }
    return (
    	<div>
        	<Button type="primary" onClick={handleClick}>回到首页</Button>
        </div>
    )
}
```
通过使用 useHistory ，让点击事件跳转到首页中  
**懒加载**  
``` 
import { BrowserRouter as Router, Route, Switch } from'react-router-dom';
import React, { Suspence, lazy } from'react';

const Home = lazy(() =>import('./routes/Home'));
const About lazy(() =>import('./routes/About'));

const App = () => {
    <Router>
    	<Suspense fallback={<div>Loading……</div>}>
        	<Switch>
        		<Route exact path="/" component={Home}/>
                 <Route path="/about" component={About}/>
        	</Switch>
        </Suspense>
    </Router>
}
```
在 React 中，可以直接用 lazy() 包裹，对页面的内容进行懒加载。当然，还有另外一种情况是，加载类似于首页初次加载页面 Loading 的那种效果，在 react 中可以使用 <Suspense> 来解决

原文: 
[探秘react，一文弄懂react的基本使用和高级特性](https://mp.weixin.qq.com/s/U_1MG7uk3GdbSfxhmZ0VZA)
