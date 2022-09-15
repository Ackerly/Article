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
**Left组件**  
在开始拖拽组件之前，需要让Left组件渲染一些默认组件以供拖拽，由于前面已经对el-config.jsx进行了配置，所以在Left中可以直接取出并使用  
``` 
// 渲染el-config.jsx中的组件
import config from '../../el-config'

const Left = () => {
    return (
        <section className="left">
            {
                config.map((v, i) => (
                    <div
                        className="left-config"
                        draggable
                        key={i}
                        onDragStart={() => dragStart(i)}
                    >
                        {v.el()}
                    </div>
                ))
            }
    </section>
}
```
为了美观一点，将每个组件都渲染在.left-config这个div中，并给予.left-config一定的样式。其次为这个div开启dragable属性，当为一个元素指定了dragable属性(也可以手动指定为true)后，这个元素就可以被拖拽了，并且当开始拖拽时，会触发它的onDragStart事件，要注意的是，这个事件会作用于被拖拽的元素身上，也就是.left-config身上  
上面config.map(... )中出现了一个v.el()，现在来说说它的作用是什么。在React中，无论是函数式组件还是类式组件，最后其实都是通过“人工”调用的，比如  
``` 
const A = () => null
const B = class B extends React.PureComponents {
 render{
            return null
        }
    }
```
使用A、B两个组件的形式也很简单，无非就是<A/>、<B/>，换句话说，在执行<A/>的同时，React发现这是一个函数式组件，然后调用它，即A()，然后对A函数的参数、返回值进行一番处理，随后得到了null；类式组件亦是如此，只不过调用的形式发生了变化，即A()变为new B()，然后根据B返回的实例进行下一步操作，下面举例说明  
``` 
import { useState } from 'react'

const A = () => {
    const [v] = useState(0)
    return <p>{v}</p>
}

// A() 与 <A/> 并无太大区别
const V = () => A()

export default V
```
A与V均为函数式组件，A组件返回的一个具体的dom元素，而V组件则是手动调用了A函数，这与直接书写<A/>并无太大区别，这种情况就适用于想渲染某个组件，但又不想立即渲染，所以可以使用这种形式来“缓存”下组件，使其不要立即渲染  
以按钮组件举例说明，其配置项的v.el正是props => <button {...props} >按钮</button>，所以此处{v.el()}的目的正是要渲染这个按钮组件，这里只需简单了解下v.el的函数体即可，后面会对它进行详细讲解  
现在为.left-config，也就是Left组件区添加事件  
``` 
import { useContext } from 'react'

import context from '../../Context'
import config from '../../el-config'

const Left = () => {
    const { setCurDrag, editor } = useContext(context)
    // 开始拖拽时触发
    const dragStart = (i) => {
        console.log('dragStart...')
        setCurDrag({
            key: editor.length,
            ...config[i],
        })
    }
    return (
     <session className="left"></session>
    )
}
```
此时注入onDragStart事件，当此事件被触发时，会调用dragStart函数，并且将i传递进去，这个i在此处就是指的当前拖拽的元素是哪个元素(在el-config.jsx中，比如索引0代表按钮，1代表输入框)，在dragStart中只需要根据i来拿出对应的配置，然后修改curDrag即可达到效果   
**Center组件**  
Center组件  
``` 
import { Fragment, useContext } from 'react'

import context from '../../Context'

const Center = () => {
    const { editor, setEditor, curDrag, setCurDrag } = useContext(context)
    // 移动时触发
    const dragOver = e => {
        console.log('dragOver...')
        e.preventDefault()
    }
    // 进入时触发
    const dragEnter = () => {
        console.log('dragEnter...')
    }
    // 完成本次拖拽时触发
    const drop = (e) => {
        console.log('drag...', e)
        const position = {
            top: e.nativeEvent.offsetY,
            left: e.nativeEvent.offsetX,
        }
        const nextEl = { ...curDrag, position }
        setCurDrag(nextEl)
        setEditor([...editor, nextEl])
    }
    return (
        <section
            className="center"
            onDragEnter={dragEnter}
            onDragOver={dragOver}
            onDrop={drop}
        >
            {
                editor.map((v, i) => (
                    <Fragment key={i}>
                        {v.el({ style: { ...v.position, ...v.style }, className: 'static' })}
                    </Fragment>
                ))
            }
        </section>
    )
}

export default Center
```
为Center组件绑定onDragEnter、onDragOver、onDrop事件，此三个事件已在上面进行了详细说明，需要注意的是它们会作用于目标元素身上，也就是类名为.center的这个session元素上，例如从Left组件拖拽一个元素至Center组件中，Left组件就是当前元素，Center组件就是目标元素  
onDrop事件会在拖拽完成时触发，其drop函数对当前拖拽元素的位置进行确定，通过event得到offsetX、offsetY之后，对editor添加新的元素，并且对curDrag进行补充，因为curDrag本身就是配置项了，但该配置项并没有当前元素的具体位置，所以无法渲染  

> 换句话说，App中有三个状态，即editor、curDrag、selectedEl。Left与Center通过curDrag相关联，用于确定拖拽的组件是哪个组件；Right与Center通过selectedEl相关联，用于确定当前修改的组件是哪个组件；而Center则全部享有三个状态

el: props => <button {...props} >按钮</button>    
可以看到传递的arguments原封不动的移动到了button身上，这也是为什么位置会被渲染出来的原因，即  
v.el({ style: { ...v.position, ...v.style }, className: 'static' })  
相当于  
``` 
<button
    style={{top: '', left: '', ...}}
    className='static'
>按钮</button>
```  
类.static则是添加了一层position: absolute，与center的position: relative相对应，而offsetX、offsetY又对应着它们每个人的top、left，所以就可以保证拖拽的效果实现  
实现了从组件区拖拽一个组件到画布区（先不考虑属性区是如何完成的），但这种实现方式真的可取吗？  
假设今天(2022/08/03)刚把这个低代码平台实现了，但产品下午上班第一件事就告诉你：“小张啊，左边这个绿色区域不太美观，你能不能把每个组件的背景色换成和右边的红色一样啊”。现在尝试转换成代码的思路，”左边 组件背景色改为红色“ -> ”组件区中每个组件的背景颜色为红色"，嗯？这不是很简单吗，只要把属性区的background-color拷贝一下给组件区不就完成了吗，说干就干，改动后的代码如下  
``` 
.left-config {
    background-color: #DF7861;
}
```
拖拽的时候按理说不应该只有按钮组件过来吗，怎么背景色也过来了呢，是不是拖拽的是背景色，而不是按钮两个字？如果用户只能拖拽“按钮”两个字的这个区域，那体验感就大打折扣了（因为可拖拽的区域太小了）。  
在不完成需求的前提下，即使把背景色由红色换为透明色，类似的问题还是会出现，现在来分析一下为什么会出现这个问题,以下是.left-config的CSS样式  
``` 
.left-config {
    width: 100%;
    height: 200px;
    display: flex;
    justify-content: center;
    align-items: center;
    user-select: none;
    margin-bottom: 50px;
    border-radius: 5px;
    border: 1px solid #fff;
    position: relative;
    background-color: #DF7861;
    transition: all 0.3s;

    &:hover {
        opacity: 0.45;
        box-shadow: 0 0 10px 1px #00000038;
    }

    &::after {
        content: '';
        width: 100%;
        height: 100%;
        position: absolute;
        top: 0;
        left: 0;
        background-color: transparent;
        opacity: 0.1;
    }
}
```
.left-config有一个after伪元素，但罪魁祸首不是它，这个伪元素的目的只是在拖拽时确保不会触发已渲染组件的事件，下面的效果为不使用伪元素，可以看到，如果在组件区域拖拽input框，此时的输入框是会获取到焦点的  
添加这层伪元素的目的就是防止触发已渲染组件的默认行为。而红色背景则是因为拖拽属性的问题，拖拽属性是绑定给.left-config的，此处可以理解成，当拖拽.left-config时，会将.left-config的区域暂时生成一张快照，然后这张快照就是在屏幕上显示的拖拽区域，而背景色虽为红色，但也属于构成这张快照的一员，所以拖拽时自然就会被显示了。那应该怎么解决这个问题呢？如果把dragable属性给每个组件(.static)，那拖拽的区域是有限的，比如当拖拽按钮组件时，只有在按钮这个“小方块”中拖拽才能触发onDragStart事件；当拖拽输入框组件时，只有在输入框这个“小长条”中拖拽才能触发onDragStart事件，这样一来，用户的体验也会大大下降，所以应该如何解决呢？  
解决方案就是不用H5的drag，因为太局限了，无法达到一些特殊的产品需求，或者说使用时无法给出最优的体验感  

**实现拖拽——定位**  
回顾一下使用H5 Drag的实现方式，在Left组件中通过onDragStart得知要拖拽组件了，在Center组件中通过onDrop来得知此次拖拽完毕，并在drop函数中做出一系列反馈  
那改为定位应该如何去做呢？这里提供一个思路就是，在Left组件中按下鼠标并移动鼠标时，就等同于要拖拽组件了，可以理解为类似onDragStart事件的处理过程。然后在Center组件中注入鼠标移动事件、鼠标松开事件，这样来看的话，一次完整的拖拽组件过程就是  
- 鼠标在组件区按下并移动
- 鼠标在画布区移动
- 鼠标在画布区松开

只需要在鼠标松开时，拿到对应的数据(配置)进行渲染即可  

**Left组件**  
先在Left组件中，使用onMouseDown与onMouseMove替代onDragStart  
``` 
const Left = () => {
    const [move, setMove] = useState(false)
    // 鼠标按下
    const handleMouseDown = () => {
        console.log('handleMouseDown...')
        setMove(true)
    }
    // 鼠标移动
    const handleMouseMove = (i) => {
        if (!move) return
        console.log('left handleMouseMove...')
        setCurDrag({
            key: editor.length,
            ...config[i]
        })
        setMove(false)
    }
    return (
        <section className="left">
            {
                config.map((v, i) => (
                    <div
                        className="left-config"
                        key={i}
                        onMouseDown={handleMouseDown}
                        onMouseMove={() => handleMouseMove(i)}
                    >
                        {v.el()}
                    </div>
                ))
            }
        </section>
    )
}
```
使用move来标识是否处于移动状态，如果处于移动状态，才能去修改curDrag，否则只要鼠标移动上来，curDrag就会一直被修改；当鼠标按下时，就代表用户要进行拖拽了，所以设置move为true，此时便会顺利通过handleMouseMove中的if校验，其handleMouseMove函数会收到参数i，这个i则标识当前拖拽的组件是哪个组件，然后设置curDrag的值，其中key为当前被拖拽组件的标识，因为如果画布中的组件全部都是按钮时，无法单纯的通过配置进行确定当前操作的组件是哪个组件，而...config[i]则是从el-config.jsx中取出对应的配置，然后通过setCurDrag来更新curDrag。调用setCurDrag时，Center就会拿到新的curDrag，然后去渲染当前组件  
**Center组件**  
从组件区拖拽一个组件到画布区都经历了什么  
- 鼠标从组件区按下：代表将要拖拽新组件到画布区
- 鼠标在组件区移动：代表已经要往画布区中生成新组件了
- 鼠标移入画布区：显示灰色遮罩层，并显示弹窗，弹窗中有当前被拖拽组件的名字
- 鼠标在画布区移动：灰色遮罩层依然显示，并且弹窗跟随鼠标进行移动
- 鼠标在画布区松开：灰色遮罩层隐藏，并且鼠标落点处会生成新组件，然后属性区显示当前组件的属性

现在前两步已经实现了，来看后三步应该怎么做。既然drag已经被抛弃了，那Center组件就有必要进行重写了，一个基础的Center组件如下  
``` 
import { useState, useContext } from 'react'

import context from '../../Context'

const Center = () => {
    const { editor, curDrag, setEditor, setCurDrag, setSelectedEl } = useContext(context)
    // 注册事件
    return (
        <div
            className="center"
            onMouseMove={handleMouseMove}
            onMouseUp={handleMouseUp}
        >
            {
                editor.map((v, i) => (
                    <div
                        key={i}
                    >{v.el({ style: { ...v.position, ...v.style }, className: 'static' })}</div>
                ))
            }
        </div>
    )
}

export default Center
```
**遮罩层**  
``` 
const Center = () => {
    // 是否有选中某个组件
    const [selected, setSelected] = useState(false)
    return (
        <div
            className="center"
            onMouseMove={handleMouseMove}
            onMouseUp={handleMouseUp}
        >
   {selected ? <div className="shield"></div> : null}
            {
                editor.map((v, i) => (
                    <div
                        key={i}
                    >{v.el({ style: { ...v.position, ...v.style }, className: 'static' })}</div>
                ))
            }
        </div>
    )
}
```
新增加一个selected状态，决定灰色遮罩层是否显示，当在画布区选中某个元素，或由组件区拖拽元素至画布区时，selected应为true。  

**Pop组件**  
既然弹窗也是Center的子组件，为了方便管理，不如将其抽离出去，将弹窗命名为Pop，现在项目结构如下  
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
    |   |   |-- Pop              // 新增
    |   |       |-- index.jsx
    |   |-- Left
    |   |   |-- index.jsx
    |   |-- Right
    |       |-- index.jsx
    |-- Context
        |-- index.jsx
```
Pop组件的基本职责是什么：  
- 鼠标在画布区移动时，需要进行跟随
- 显示当前被拖拽组件的名称

鼠标在画布中移动时，在Pop中拿不到精确的位置，而又需要Pop跟随鼠标，因为在Center中可以拿到具体的位置，所以可以使用最简单的父传子通信方式，由Center下发位置，Pop去更改位置，而对于在Pop中如何更改自身位置，此处选择使用Ref，代码如下  
``` 
import { useRef, useEffect } from 'react'

const Pop = ({ setHandOver, text }) => {
    // 通过ref保存Pop元素
    const ref = useRef()
    // 修改位置的方法
    const changePosition = {
        top: v => ref.current.style.top = ${v}px,
        left: v => ref.current.style.left = ${v}px,
    }
    useEffect(() => {
        // 向上传递
        setHandOver({ ...changePosition, ref: ref.current })
    }, [])
    return (
        <div
            className="pop"
            ref={ref}
        >{text}</div>
    )
}

export default Pop
```
在Pop组件挂载后，通过父组件提供的setHandOver将修改Pop位置的方法、以及Pop元素自身都传递上去，这样一来Center组件便可以操控Pop组件了，当在Center组件中要生成新的拖拽组件时，也只需通过handOver获取Pop的位置即可  

**handOver**  
Center添加setHandOver  
``` 
const Center = () => {
    const { curDrag } = useContext(context)
    // 获取Pop中的数据
    const [handOver, setHandOver] = useState(null)
    return (
        <Pop
           setHandOver={setHandOver}
           text={text}
       />
    )
}
```
这样一来Pop组件始终都会被挂载，而不受curDrag的限制，当不需要Pop组件时，将定位设置为负值即可；让Pop始终挂载是因为Center组件始终需要这个dom元素去承担显示位置、移动位置、显示组件名称等任务  
handOver中就拥有了top、left方法，当用户拖拽一个元素至画布区时，只需要在Center中拿到鼠标的位置然后通过top、left方法进行修改弹窗位置即可，这样便会实现弹窗跟随鼠标进行移动的效果了  

**完成Center组件**
_onMouseMove_
为Center注入鼠标移动事件  
``` 
const Center = () => {
    // 当前被拖拽组件的key
    const { key } = curDrag
    // 鼠标移动事件
    const handleMouseMove = e => {
        if (!(typeof (key) === 'number')) return
        console.log('center handleMouseMove...')
        const top = e.nativeEvent.offsetY
        const left = e.nativeEvent.offsetX
        handOver.top(top)
        handOver.left(left)
        setSelected(true)
    }
    return (
     <div
            className="center"
            onMouseMove={handleMouseMove}
     ></div>
    )
}
```
因为onMouseMove事件是给Center组件的，所以不可能一直被触发，需要有个边界判断进行限制，而通过组件区拖拽组件至画布区的这种方式，Center组件会收到完整的curDrag，包含当前被拖拽组件的key，所以在onMouseMove事件中通过判断key是否为Number类型来决定是否执行事件处理逻辑  
_onMouseUp_  
onMouseUp事件与onMouseMove事件同样进行一次边界判断   
``` 
const Center = () => {
    // 存储最后的位置
    const [lastPosition, setLastPosition] = useState({ top: 0, left: 0 })
    const [handOver, setHandOver] = useState(null)
    // 鼠标松开事件
    const handleMouseUp = e => {
        if (!(typeof (key) === 'number')) return
        console.log('handleMouseUp...')
        e.preventDefault()
        e.stopPropagation()
        const top = parseFloat(handOver.ref.style.top)
        const left = parseFloat(handOver.ref.style.left)
        const curPosition = { top, left }
        if (top <= -10 && left <= -101) {
            const { top, left } = lastPosition
            curPosition.top = top
            curPosition.left = left
            changePosition(true, true)
            return
        } else { setLastPosition({ top, left }) }
        const position = { top: curPosition.top, left: curPosition.left }
        changePosition(position)
    }
    return (
     <div
            className="center"
            onMouseUp={handleMouseUp}
     ></div>
    )
}
```
lastPosition则是用于存储当前元素最后的位置，因为经过多次实践发现，有些自带焦点的组件（无论是UI组件库，还是原生dom元素），比如按钮，按钮是可以获取到焦点的，而像一个p标签，或是一张图片，是获取不到焦点的，这些自带焦点的组件在触发自身的默认事件时会发生位置错乱的现象，所以使用lastPosition存储它们最后的位置，以便获取到焦点时造成不必要的位置偏移  
在onMouseUp事件中，通过handOver中存储的Pop组件的ref，来得到弹窗的具体位置，并做一次简单的判断，如果当前的这个松开事件是一个焦点事件，那就不更改位置，而是使用上次缓存的位置，这样保证了其焦点获取时不会发生布局错乱的现象；如果这个松开事件不是一个焦点事件，则缓存一下这个组件的位置，然后通过changePosition去渲染最新的位置；因为在它们获取到焦点时，会发生一次偏移，而偏移的数量就是判断的标准  
_changePosition_  
``` 
const Center = () => {
    // 修改位置的方法（Pop与新组件的位置）
    const changePosition = (position, require) => {
        handOver.top(0)
        handOver.left(-101)
        setSelected(false)
        setCurDrag({})
        setSelectedEl({ ...curDrag, style: { ...curDrag.style } })
        if (require) return
        const isHave = editor.find(v => v.key === key)
        if (!isHave) return setEditor([...editor, { ...curDrag, position }])
        const arrs = editor.map(v => v.key === key ? { ...v, position } : v)
        setEditor(arrs)
    }
}
```
参数position决定了要渲染的最新位置，require决定了是否进行渲染，因为当自带焦点的组件事件被触发时，是不需要渲染位置的；在更改位置之前，首先调用handOver中的top、left方法使Pop组件“复位”，因为本次拖拽已经结束了；setSelected(false)则是代表当前未选中组件，以此来取消灰色遮罩层的显示；setCurDrag({})则是代表拖拽完毕，不要再进去Center中的事件了；setSelectedEl则是通知属性区去展示当前组件对应的属性；isHave的作用则是判断此次修改是针对的新组件还是旧组件，然后根据不同的情况去渲染不同的editor  
**完成Pop组件**  
``` 
import { useRef, useEffect } from 'react'

const Pop = () => {
    const ref = useRef()
    const changePosition = {
        top: v => ref.current.style.top = ${v}px,
        left: v => ref.current.style.left = ${v}px,
    }
    // 鼠标移动至Pop上时，必须对其作出处理
    const handleMouseMove = e => {
        e.stopPropagation()
        const top = e.nativeEvent.target.style.top
        const left = e.nativeEvent.target.style.left
        changePosition.top(parseFloat(top) + e.nativeEvent.offsetY)
        changePosition.left(parseFloat(left) + e.nativeEvent.offsetX)
    }
    return (
        <div
            onMouseMove={handleMouseMove}
            ref={ref}
        ></div>
    )
}

export default Pop
```
给Pop组件添加onMouseMove的原因是为了避免在画布区拖动时造成的“丢帧”情况，并且需要在该事件中取消事件冒泡，如果不取消事件冒泡，则当鼠标移动至Pop中时，Pop组件会立即发生位置错乱，当然，你也可以不取消事件冒泡，而是使用元素与鼠标分离的方式，也就是拖拽时始终保证鼠标位于Pop组件的下方并有一段有效间隔，随后在该事件中对Pop的位置做出处理  
**主动拖拽画布中的组件**  
一个完整的Center组件就基本实现了，但还有个问题，就是无法在画布区主动拖拽元素，也就是说，现在只能通过组件区拖拽组件至画布区，无法在画布区主动拖拽已有的组件，由于前面已经实现了大概，所以在Center组件中直接添加一个onMouseDown即可  
``` 
const Center = () => {
    // 鼠标按下（可以理解成由组件间拖拽组件至画布区）
    const handleMouseDown = i => {
        console.log('handleMouseDown...')
        setCurDrag(editor.find(v => v.key === i))
    }
    return (
     <div className="center">
            {
                editor.map((v, i) => (
                    <div
                        key={i}
                        onMouseDown={() => handleMouseDown(i)}
                    >{vel()}</div>
                ))
            }
     </div>
    )
}
```
onMouseDown事件要绑定给画布区中的每个组件，如果绑定给Center就无法确定当前要拖拽的组件是哪个组件了，而handleMouseDown事件处理函数的逻辑也相对简单，只是让curDrag变为当前拖拽的元素，然后便可畅通无阻的进入onMouseMove和onMouseUp事件，这样也就实现了在画布区主动拖拽元素的效果  

**搭建属性区**  
属性区中根据selectedEl来渲染当前组件的对应的属性，而是否展示为空则是根据selectedEl.text来决定，如果text取值为undefined则说明当前未选中某个组件，此时只需展示空数据即可，如果text不为空，那么就渲染当前组件内置好的style属性。  
``` 
import { useContext } from 'react'

import context from '../../Context'

// 渲染selectedEl所对应的属性
const Right = () => {
    const { setEditor, editor, selectedEl, setSelectedEl } = useContext(context)
    const styleKeys = Object.keys(selectedEl.style)
    return (
        <section className="right">
            {
                !selectedEl.text ? <h2>暂未选择组件</h2> : (
                    <div>
                        <h2>{selectedEl.text}</h2>
                        {
                            styleKeys.map((v2, i2) => (
                                <div
                                    className="right-configs"
                                    key={i2}
                                >
                                    <p>{v2}</p>
                                    <input
                                        type="text"
                                        onChange={e => handleChange(e, v2)}
                                    />
                                </div>
                            ))
                        }
                    </div>
                )
            }
        </section>
    )
}

export default Right
```
当确定选中了组件，并且在更改输入框的值时，会触发其onChange属性，在handleChange函数中会收到两个形参，event用于取出输入框的值，name则代表要修改的style属性是谁  
``` 
const Right = () => {
    // 修改当前组件的属性
    const handleChange = (e, name) => {
        const value = e.nativeEvent.target.value
        const next = v => ({
            ...v,
            style: {
                ...v.style,
                [name]: value
            }
        })
        const result = editor.map(v => v.key === selectedEl.key ? next(selectedEl) : v)
        setEditor(result)
        setSelectedEl(next(selectedEl))
    }
```
通过key取出当前操作的组件是哪个组件，这种类似的操作在前面已经用过多次，所以不再进行赘述。找出操作的组件并更新其配置后，此时修改editor、selectedEl，修改editor的目的是让画布重新渲染，修改selectedEl则是让属性区也得到最新的属性  

**浅谈低代码**  
1. 代码在某种程度上可以分担程序员的一些工作，比如一个简单的H5页面，完全可以通过低代码平台进行实现，无论是设计还是产品，中间都省去了一堆繁琐的步骤，但比较复杂的H5页面，单单借助低代码平台是很难完成的，所以说低代码平台更像是给这些不太懂编程而又想快速实现符合产品本身需求的这类人所准备的
2. 复杂的低代码平台，或者说由此所生产出来的网页，它的可维护性、可扩展性和灵活性，是有待考究的，比如要对原网页进行二次开发怎么办、网页中有一个复杂的请求模块应该如何处理、如何去测试这个网页的功能是否正常、...... 等等这些问题都是要考虑的
3. 

参考：  
[低代码都做了什么？怎么实现 Low-Code？](https://mp.weixin.qq.com/s/a_J2jjkh_aAzJ0gUsBeoTg)
