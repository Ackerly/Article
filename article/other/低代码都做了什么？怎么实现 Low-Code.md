# 低代码都做了什么？怎么实现 Low-Code？
**什么是低代码**  
低代码(Low-Code)从字面意思来讲，低就是少;在一般标准或平均程度之下，那低代码自然就是少代码，也就是说不需要付出太多的代码成本  
想要从0实现一个可以在Web中访问的网页，那最好要掌握的技术必然是Html、Css、JavaScript，大部分情况下仅仅有了以上三种技术的加持是不够的，为了让所生产出来的Web页面有着高维护性和高灵活性，一般要根据网页中的功能进行模块划分，以及确定页面的整个层次、结构等，其次再去考虑UI等设计类问题  
通过上述描述可以发现，实现一个Web页面的学习成本，对于一个并不长期从事于网页开发的用户来说，这种开发方式所带来的学习成本是较大的；而低代码则不同，它不需要付出太多的代码成本，换句话说，即使零代码基础也可以轻松构建Web页面  

**低代码的优点**  
随着一个个可视化编辑工具或者说低代码平台的问世，低代码的优点也逐渐突出，比如上手速度快、开发效率高，不需要去考虑高学习成本所带来的负担。有着页面可视化的加持，网页的布局、设计、UI等尽在掌握之中。通过拖拽组件的方式，可以在最短时间内实现最初的想法或设计，从而不用将过多的精力投入至编程中  

**实现Low-Code的方式**  
实现一个Web版低代码平台的方式大概有哪些  
- 由浏览器完成构建、渲染；服务端则提供一些依赖包
- 由服务端完成构建；浏览器则只负责渲染

大概通过以下步骤来从0至1实现低代码平台  
- 快速完成静态页面布局
- 确定能够被拖拽的组件有哪些，以及它们的存在方式是什么
- 选择一种适用于各组件间进行通信的方式
- 实现拖拽
- 渲染被拖拽组件
- 确保组件属性更改后可以实时映射至画布区域

## 实现
**确定界面**  
直接采用最简单的布局，即左、中、右方式的三栏布局，左栏与右栏均为固定宽度，中间一栏自适应。不过左栏和右栏这俩名字听起来也不优雅，所以将左栏称之为组件区，代表要拖拽的组件；中间栏称为画布区，即发挥灵魂画手的地方；右栏则为属性区，用来设置画布中某个组件的样式或属性  
组件区定义为Left，画布区定义为Center，属性区定义为Right；然后由App组件对其统一管理。此时的项目结构为  
``` 
|-- Low-Code
    |-- package-lock.json
    |-- package.json
    |-- README.md
    |-- public
    |   |-- favicon.ico
    |   |-- index.html
    |-- src
        |-- App.jsx
        |-- index.js
        |-- components
        |   |-- Center
        |   |   |-- index.jsx
        |   |-- Left
        |   |   |-- index.jsx
        |   |-- Right
        |       |-- index.jsx
```
**思路**  
App(#root)为flex布局，Left与Right的宽度固定为300px，由于Center是自适应宽度，所以不设置固定宽度，而是将其flex-grow设为2，这样便达到了圣杯布局的效果  
``` 
|-- src
    |-- App.css               // 新增
    |-- App.jsx
    |-- App.less              // 新增
    |-- index.js
    |-- components
    |   |-- Center
    |   |   |-- index.jsx
    |   |-- Left
    |   |   |-- index.jsx
    |   |-- Right
    |       |-- index.jsx
```
``` 
// App样式如下  
#root {
  width: 100%;
  height: 100%;
  display: flex;
}

// Left样式如下
#root .left {
  width: 300px;
  height: 100%;
  padding: 15px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: #94B49F;
}

// Center样式如下
#root .center {
  height: 100%;
  flex-grow: 2;
  position: relative;
  background-color: #FCF8E8;
  box-shadow: inset 0 0 4px 2px #00000042;
}

// Right样式如下
#root .right {
  width: 300px;
  height: 100%;
  padding: 15px;
  background-color: #DF7861;
}
```
**引入**  
App组件代码如下  
``` 
import Left from './components/Left'
import Right from './components/Right'
import Center from './components/Center'

import './App.css'

const App = () => {
  return (
    <Left />
    <Center />
    <Right />
  )
```
Left、Center、Right组件中的代码均为以下示例模板  
``` 
const Xxx = () => {
    return <session className="xxx"></session>
}

export default Xxx
```
**配置项**  
Low-Code能否实现的关键问题之一就是：组件区、画布区、属性区之间应该通过何种方式进行关联？此处并不是指组件间通信方式，因为这并不是个棘手的问题  

## 一次拖拽，两处显示 & 一处修改，两处显示
**一次拖拽，两处显示**  
在组件区拖拽完成后，画布区和属性区显示当前被拖拽组件，以及它所对应的属性  
**一处修改，两处显示**  
在属性区完成更改属性后，画布区需要根据最新的属性重新渲染当前组件，属性区则也要展示当前组件所对应的最新属性  
在已经实现拖拽的前提下，设想现有一个Button组件，鼠标从组件区将Button拖至画布区，然后画布区呈现当前拖拽的Button组件，并且属性区会显示Button组件的一系列属性。如果此时主动(拖拽)修改画布区中Button组件的位置，则Center要重新渲染最新的位置，要注意的是，画布区中其它组件的位置不能发生变动，位置发生变化的只能是当前组件，即Button  
类似的问题又来了，当修改属性区中的某一属性时，只有当前组件所对应的属性会发生变化，其它组件即使有同名属性也不会发生变化  
现在可以正视这个问题了，不管是组件的位置变化还是属性更改，其影响的区域甚多，而又不可能让每个区域都单独遵从某个逻辑，所以综上所述，此时应使用一个规则，来管理所有的变化，所有的区域都遵循当前规则  
比如  
``` 
{
    text: "按钮",
    el: <Button/>,
    style: {
        width: '',
        height: '',
    },
}
```
text代表当前组件的名称，el代表当前要渲染的组件，style代表当前组件的css样式  
所有区域都共用以上规则，假设在属性区中修改Button组件的宽度，此时只需要改变style中的width即可，然后其它区域拿到的都是修改后的width，这样一来，问题不就解决了吗。但不能直接叫人家规则吧，除去低代码的概念，现在看看之前的“新”概念都是用什么实现的？  
数据可视化使用svg或canvas来完成直观的数据展示，而对数据的管理则使用了JSON，低代码中的拖拽其实也逃不过注入鼠标事件的结局，在低代码中除去事件外，更需要一个东西来统领组件区、画布区、属性区，那这个东西，就称之为配置。即使用一项项的配置来决定要在组件区渲染的组件有哪些、画布区应展示的组件是什么、属性区可以修改的属性有哪些、......  
无论在何处修改当前组件的位置、属性，只需要更改当前组件所对应的配置即可，这样其它区域也都会遵从最新的配置来进行渲染。所以此时在项目中新增一个el-config.jsx文件，里面存放组件区中的组件，此文件中的配置就决定了低代码平台中的组件区拥有哪些组件、画布中可以渲染哪些组件，以及属性区可以更改哪些属性等  
``` 
|-- src
    |-- App.css
    |-- App.jsx
    |-- App.less
    |-- el-config.jsx         // 新增
    |-- index.js
    |-- components
    |   |-- Center
    |   |   |-- index.jsx
    |   |-- Left
    |   |   |-- index.jsx
    |   |-- Right
    |       |-- index.jsx
    
const config = [
    {
        text: "按钮",
        el: props => <button {...props} >按钮</button>,
        style: {
            width: '',
            height: '',
            backgroundColor: '',
        }
    },
    {
        text: "输入框",
        el: props => <input {...props} />,
        style: {
            width: '',
            height: '',
            backgroundColor: '',
        }
    },
]

export default config
```
**选择组件通信方式**  
组件通信的方式有很多种，比如父传子、PubSub，甚至是redux等，而现在的需求是多个组件共用一个配置，当这个配置项发生变化时，其它组件必须重新渲染，以此显示最新的组件状态。所以决定使用context来完成各组件通信，即  
在项目中新增一个Context文件夹，用于存放创建的context环境上下文  
``` 
|-- src
    |-- App.css
    |-- App.jsx
    |-- App.less
    |-- el-config.jsx
    |-- index.js
    |-- components
    |   |-- Center
    |   |   |-- index.jsx
    |   |-- Left
    |   |   |-- index.jsx
    |   |-- Right
    |       |-- index.jsx
    |-- Context            // 新增
        |-- index.jsx
```
``` 
import React from 'react'
export default React.createContext()
```
通过上述代码就创建好了一个context环境上下文，然后在组件区、画布区、属性区的共同区域使用Provider向内传递数据，其被包裹的组件只需通过useContext来接收数据即可，当共同区域中的数据发生变化时，Provider下的组件会拿到最新的值，并重新渲染  
**App组件结合Context**  
App组件通过Provider将组件区、画布区、属性区进行包裹并向下传递数据  
``` 
import { useContext } from 'react'

import context from './Context'

const App = () => {
  // 存储画布中的组件
  const [editor, setEditor] = useState([])
  // 存储当前拖拽的组件
  const [curDrag, setCurDrag] = useState({})
  // 存储当前操作的组件
  const [selectedEl, setSelectedEl] = useState({ style: {} })
  return (
    <Provider value={{
      editor, setEditor,
      curDrag, setCurDrag,
      selectedEl, setSelectedEl,
    }}>
      <Left />
      <Center />
      <Right />
    </Provider>
  )
}
```
需要注意的是，组件区、画布区、属性区会始终操作这些传递下去的数据，而当调用setXxx时，其下的组件都会被重新渲染，下面是对于各状态的说明  
- editor用于存储画布中的元素，默认为空数组；当在画布中新增元素时，只需对该数组执行push操作即可
- curDrag为当前正在拖拽的元素；当开始拖拽新元素时，立即使用setCurDrag更新curDrag的值，并可借此通知画布区将要新增的组件是哪个组件
- selectedEl则让属性区展示当前被拖拽组件的属性

**实现拖拽——H5 Drag**  
在Html5提供的一系列drag API加持下，可以轻松实现在页面中对元素发起的任意拖拽，而一个好的Low-Code更不能丢弃拖拽的优雅性，所以在此处用drag来实现拖拽组件的效果，但本篇文章并不会使用drag来完成拖拽组件，而是采用定位来完成，具体原因会详细说明
对drag Api进行简单回顾  
- onDragStart：当要拖拽的元素开始被拖拽时触发，此事件作用于被拖拽元素上
- onDragEnter：当要拖拽的元素进入目标元素时触发，此事件作用于目标元素上
- onDragOver：当要拖拽的元素在目标元素上移动时触发，此事件作用于目标元素上
- onDragDrop：当要拖拽的元素在目标元素上松开鼠标时触发，此事件作用于目标元素上

在onDragOver中必须取消默认事件，即event.preventDefault()，否则onDrop事件不会被触发  

## 注入事件


参考：  
[低代码都做了什么？怎么实现 Low-Code？](https://mp.weixin.qq.com/s/a_J2jjkh_aAzJ0gUsBeoTg)
