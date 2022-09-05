# React组件设计过程-仿抖音订单组件
**组件设计思路**  
组件中需要实现的业务有：  
- tab切换：  
点击tab，该tab添加上红色下划线样式，并将该tab状态下的订单展示在下方。
- 设置loading状态：  
在数据还在请求中时，显示loading图标
- 搜索订单：  
在当前tab下搜索商品标题含有输入内容的订单。
- 删除订单：  
删除指定订单，由于数据是在fastmock中请求得到，因此删除只相对于前端。
- 实现Empty（空状态）组件  
当当前状态下订单数量为 0 时，显示该组件，否则显示列表组件。

可以划分出5个组件模块组成整个页面：  
1. 页面级别组件<Myorder/>,它是其他组件的父组件；
2. 显示数据列表组件<OrderList/>，单个数据组件<OrderNote/>；
3. 空状态组件<EmptyItem/>；
4. 推荐商品列表组件<RecommendList/>。  
在<Myoeder/>组件中请求数据，将对应的数组数据通过props传给<OrderList/>组件和<RecommendList/>组件;<OrderList/>组件再将单个数据传给<OrderNote/>组件。这样就规范的完成了父组件请求数据，子组件搭建样式的分工合作了。  

**实现 Myorder 组件**  
这个页面级别组件包括固定在顶部的搜索框+导航栏，以及OrderList和RecommendList组件，因此可以写出如下组件框架：  
``` 
import React from 'react'
import OrderList from '../OrderList'
import RecommendList from '../RecommendList'
import { OrderWrapper } from './style'
import fanhui from '../../assets/images/fanhui.svg'
import gengduo from '../../assets/images/gengduo.svg'
import sousuo from '../../assets/images/sousuo.svg'

export default function Myorder() {
  return (
    <OrderWrapper>
      // 搜索 + 导航栏 部分
      <div className="head">
        <div className="searchOrder">
          <img src={fanhui} alt="返回"/>
          <div className='searchgroup'>
            <input 
              placeholder="搜索订单" 
            />
            <img className="searchimg" src={sousuo} alt="搜索"/>
          </div>
          <img src={gengduo} alt="更多"/>
        </div>
        <ul>
          <li>全部</li>
          <li>待支付</li>
          <li>待发货</li>
          <li>待收货/使用</li>
          <li>评价</li>
          <li>退款</li>
        </ul> 
      </div>
    
      // 订单列表组件
      <OrderList/>
      
      // 推荐列表组件
      <RecommendList/>
    </OrderWrapper>
  )
}
```
**实现tab切换效果**  
当点击某个tab时，如'待支付'，这个tab要有红色下划线效果。实现原理其实很简单，就是当我们触发该tab的点击事件时，就将我们事先写好的active样式加到该tab上。  
两种方案：  
- 第一种实现方法是定义一个状态tab来控制每个<li>的className的内容：  
``` 
import React,{ useState} from 'react'
import { OrderWrapper } from './style'

export default function Myorder() {
  const [tab,setTab] = useState('全部');
  const changeTab= (target) => {
    setTab(target);
  }
  
  return (
      <OrderWrapper>
          ...
          <ul>
              <li className={tab=='全部'?'active':''} onClick={changeTab.bind(null,'全部')}>全部</li>
              <li className={tab=='待支付'?'active':''} onClick={changeTab.bind(null,'待支付')}>待支付</li>
              <li className={tab=='待发货'?'active':''} onClick={changeTab.bind(null,'待发货')}>待发货</li>
              <li className={tab=='待收货/使用'?'active':''} onClick={changeTab.bind(null,'待收货/使用')}>待收货/使用</li>
              <li className={tab=='评价'?'active':''} onClick={changeTab.bind(null,'评价')}>评价</li>
              <li className={tab=='退款'?'active':''} onClick={changeTab.bind(null,'退款')}>退款</li>
            </ul>  
          ...
      </OrderWrapper>
  )
}
```
这种方法有一个明显的缺点，就是只能为其添加一个样式名，当有多个样式类名时，就会出问题了
- 第二种方法就是用 classnames 了，也是比较推荐的方法，写法也比较简单
``` 
import classnames from 'classnames'
import { OrderWrapper } from './style'

export default function Myorder() {
  const [tab,setTab] = useState('全部');
  const changeTab= (target) => {
    setTab(target);
  }
  
  return (
      <OrderWrapper>
          ...
          <ul>
              <li className={classnames({active:tab==="全部"})} onClick={changeTab.bind(null,'全部')}>全部</li>
              <li className={classnames({active:tab==="待支付"})} onClick={changeTab.bind(null,'待支付')}>待支付</li>
              <li className={classnames({active:tab==="待发货"})} onClick={changeTab.bind(null,'待发货')}>待发货</li>
              <li className={classnames({active:tab==="待收货/使用"})} onClick={changeTab.bind(null,'待收货/使用')}>待收货/使用</li>
              <li className={classnames({active:tab==="评价"})} onClick={changeTab.bind(null,'评价')}>评价</li>
              <li className={classnames({active:tab==="退款"})} onClick={changeTab.bind(null,'退款')}>退款</li>
            </ul>  
          ...
      </OrderWrapper>
  )
}
```
**获取数据**  
这里准备了两个接口，用于获取订单数据和推荐商品数据。为了便于管理，将数据请求封装在api文件中：
- 第一个接口获取订单数据。需要根据 tab状态筛选获取的数据：  
``` 
import axios from 'axios'

// 请求订单数据
export const getOrder = ({tab}) => 
    axios
    .get('https://www.fastmock.site/mock/759aba4bef0b02794e330cccc1c88555/beers/order') 
    .then ( res => {
            let result=res.data;
            if(tab){ 
                switch(tab) {
                    case "待支付":
                        result=result.filter(item => item.state=="待支付");
                        break;
                    case "待发货":
                        result=result.filter(item => item.state=="待发货");
                        break;
                    case "待收货/使用":
                        result=result.filter(item => item.state=="待收货/使用");
                        break;
                    case "评价":
                        result=result.filter(item => item.state=="评价");
                        break;
                    case "退款":
                        result=result.filter(item => item.state=="退款");
                        break;
                    default:
                        break;
                }
            }
            return Promise.resolve({
                result
            });
        }
    )
```
- 第二个接口获取推荐商品数据：
``` 
 import axios from 'axios'

    // 请求推荐商品数据
    export const getCommend = () => 
               axios.get('https://www.fastmock.site/mock/759aba4bef0b02794e330cccc1c88555/beers/goods')

```
接口准备好了，接下来将数据分配给子组件，数据如何在页面上显示的任务就交给子组件<OrderList/>和<Recommend/>完成  
``` 
import React,{useEffect, useState} from 'react'
import { OrderWrapper } from './style'
import OrderList from './OrderList'
import RecommendList from './RecommendList'

export default function Myorder() {
  const [list,setList] =useState([]);
  const [recommend,setRecommend] = useState([]);
  // 从接口中获取推荐商品数据
  useEffect(()=> {
    (async()=> {
      const {data} = await getCommend();
      setRecommend([...data]);
    })()
  })
  // 从接口中获取订单数据，每次tab切换都重新拉取
  useEffect(()=>{
    (async()=>{
      const {result} = await getOrder({tab});
      setList([
        ...result
      ])
    })()
  },[tab])
  
  return (
      <OrderWrapper>
          ...
          {list.length>0 && <OrderList list={list}/>}
          {recommend.length>0 && <RecommendList recommend={recommend}/>}
      </OrderWrapper>
  ）
}
```
**实现搜索功能**  
搜索功能应该在对应的tab下进行，因此我们可以将输入的内容设置为一个状态，每次改变就根据tab内容和输入内容重新获取数据：  
api接口对订单数据的请求的封装中增加一个query限制：
``` 
export const getOrder = ({tab,query}) => 
    axios
    .get('https://www.fastmock.site/mock/759aba4bef0b02794e330cccc1c88555/beers/order') 
    .then ( res => {
            let result=res.data;
            if(tab){
                switch(tab) {
                    case "待支付":
                        result=result.filter(item => item.state=="待支付");
                        break;
                    case "待发货":
                        result=result.filter(item => item.state=="待发货");
                        break;
                    case "待收货/使用":
                        result=result.filter(item => item.state=="待收货/使用");
                        break;
                    case "评价":
                        result=result.filter(item => item.state=="评价");
                        break;
                    case "退款":
                        result=result.filter(item => item.state=="退款");
                        break;
                    default:
                        break;
                }
            }
            if(query) {
                result = result.filter(item => item.title.includes(query));
            }
            return Promise.resolve({
                result
            });
        }
    )
```
而在组件的实现上，由于页面没有添加点击搜索的按钮，如果将input中的value直接和query状态绑定的话，每次用户输入一个字就会进行一次查询，触发太频繁，性能不够好，用户体验也不好。所以这里我的想法是每次输入完按下enter才进行搜索  
但是React中无法直接对input的enter事件进行处理。于是我在网上查阅到两种处理方式，第一种是通过 e.nativeEvent 来获取keyCode判断是否为 13 ，第二中方法是通过addEventListener注册事件来处理，要慎用。这里采用第一种方法来实现：  
``` 
import React,{useState} from 'react'
import { OrderWrapper } from './style'

export default function Myorder() {
  const [query,setQuery] = useState('');
  const handleEnterKey = (e) => {
    if(e.nativeEvent.keyCode === 13){
      setQuery(e.target.value);
    }
  }
  
   return (
       <OrderWrapper>
             ...
            <input 
              placeholder="搜索订单" 
              onKeyPress={handleEnterKey}
            />
           ...
        </div>
       </OrderWrapper>
   )
}
```
**设置loading状态**  
在数据请求过程之，页面会空白，为了提升视觉上的效果，在这个时间段我们就设置一个loading样式，这个样式组件直接使用reacct-weui的Toast组件。  
增加一个loading状态来来控制Toast的显示  
``` 
import React,{useEffect, useState} from 'react'
import { OrderWrapper } from './style'
import WeUI from 'react-weui'
const {
  Toast
} = WeUI;

export default function Myorder() {
  const [loading,setLoading]=useState(false);
  useEffect(()=>{
    setLoading(true);
    (async()=>{
      const {result} = await getOrder({tab});
      setList([
        ...result
      ])
      setLoading(false);
    })()
  },[tab])
  
  return (
      <OrderWrapper>
          ...
          <Toast show={loading} icon="loading">加载中...</Toast>
          { list.length>0 && <OrderList list={list}}
          ...
      <OrderWrapper>
  )
}
```
**实现Empty(空状态)组件**  
空状态 组件，顾名思义就是当请求到的数据为空或者是数据长度为 0 时，就显示该组件。这个组件实现起来比较简单，直接写在myorder组件中，用styled-components实现效果  
``` 
import React,{useEffect, useState} from 'react'
import { OrderWrapper,EmptyItem } from './style'
import OrderList from './OrderList'
import empty from '../../assets/images/empty.png'

export default function Myorder() {
  const [list,setList] = useState([]);
  ...
  
  return (
     <OrderWrapper>
         ...
          {list.length>0&&<OrderList list={list} deleteOrder={deleteOrder}/>}
          {list.length==0&&loading==false&&
            <EmptyItem>
               <h3>美好生活&nbsp;&nbsp;触手可得</h3>
              <img src={empty} />
              <h2>暂无订单</h2>
              <p>你还没有产生任何订单</p>
            </EmptyItem>
          }
        ...
     </OrderWrapper>
  )
}
```
**实现 OederList 组件**  
这个组件只需要将父组件myorder传进来的数组数据通过 map 分配给 OederNote，另外删除功能在它的子组件OrderNote上触发，需要通过它解构出deleteOrder函数传给OrderNote  
``` 
import React from 'react'
import { OrderListWrapper } from './style'

export default function OrderList({list,deleteOrder}) {
  return (
    <OrderListWrapper>
      <h3>美好生活&nbsp;&nbsp;触手可得</h3>
      {
        list.map(item => (
            <OrderNote key={item.id} data={item} deleteOrder={()=>deleteOrder(item.id)}/>
        ))
      }
    </OrderListWrapper>
  )
}
```
**实现 OrderNote 组件**  
该组件主要负责实现订单的展示效果，这里只展示部分代码
``` 
import React from 'react'
import { NoteWrapper } from './style'

const OrderNote = (props) => {
    const { data } =props;
    const { deleteOrder } =props
    return (
        <NoteWrapper>
                 ...
                <div className="btngroup">
                    <button onClick={deleteOrder}>删除订单</button>
                    <button>查看相似</button>
                </div>
            </div>
        </NoteWrapper>
    )
```
这个组件可以触发删除订单的业务，具体如何删除我们只需要在父组件myOrder实现，然后将函数传递到OrderNote触发  
在myOrder组件添加deleteOrder函数：  
``` 
import React from 'react'
import OrderList from './OrderList'

export default function Myorder() {
  const deleteOrder = (id) => {
      setList(list.filter(order => order.id!==id));
  }
  ...
  
    return (
        <OrderWrapper>
            ...
             {list.length>0&&<OrderList list={list} deleteOrder={deleteOrder}/>}
             ...
        </OrderWrapper>
    )
}
```
**实现 RecommendList 组件**  
该组件也是对从父组件Myorder获取来的数据进行展示，主要是做样式上的功夫。使用多列布局，将页面分为两列，并且不固定每个数据盒子的高度。  
- 最外层列表盒子加上属性： column-count:2; 将页面分为两列
- 列表中的每一个单独的小盒子添加属性：break-inside:avoid; 控制文本块分解成单独的列，以免项目列表的内容跨列，破坏整体的布局**
- 图片的宽度设置：width:100%

原文: 
[超详细的React组件设计过程-仿抖音订单组件](https://mp.weixin.qq.com/s/kZfar7nbnR_qWi5jXB4LUw)
