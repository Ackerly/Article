# vivo 商品中台的可视化微前端实践
在电商领域内，商品是一个重要组成部分，与其对应的商品管理系统，则负责商品的新建、编辑、复制等功能。随着商品管理系统的成熟稳定和业务上的扩展需求，催化出了商品中台的诞生。它可以将现有商品功能最大效率的复用在很多业务上（公司内业务、公司外业务等）。而不是仅限于当前团队的业务使用。  
在设计商品中台的前端系统时，使用了微前端和可视化技术，其可以达到如下效果：  
- 可视化技术可以让各个业务方的运营等相关人员，直观的看到其配置的数据在页面上的展示效果；
- 微前端可以帮助商品中台更快更好的适配到各个业务方的项目中。

## 可视化技术
核心能力如下:  
- 右侧所有的变动，都能在左侧得到实时更新和展示，如主图、 sku 组合、价格、图文详情、商品参数等功能
- 左侧的可视化区域是一个标准的 h5 页面，可以把它看成一个子页面，它与外层的父页面在 ui 上是完全隔离的，同时在数据上又是共享的

## 可视化技术原理
可视化整体技术原理图如下:  
- 子窗口用 iframe 展示
- 子窗口用 vuex 做状态管理
- 子窗口和父窗口通过共享状态 （ vue store ）来完成数据通信

为什么不使用 postMessage:
- 父窗口含有大量逻辑：父窗口需要将 vuex 的数据进行处理，然后通过 postMessage 进行传输；
- 数据通信方式不纯粹：vuex 和 postMessage 组合在一起，互相转换，使数据通信更加复杂和难以控制
- 不支持 Vue.set , Vue.delete 等
- postMessage 只能同步字符串，不能 fn

在数据通信方面，没有使用 postMessage ，而是使用 vuex 替换掉 postMessage ，来完成 iframe 通信  
使用 vuex 完成 iframe 数据通信是如何实现的呢:
这个问题的答案就是 uni-render 。通过它，可以做到让子窗口通过 iframe 展示的同时，父子窗口共享 store  

**uni-render**  
uni-render 是一个让父子窗口可以不用 postMessage 就能共享 vue store 的技术方案。它包含以下关键内容：  
- 将 iframe 当成一个 dom 节点
- 父窗口渲染子窗口（ iframe ）暴露的组件
- 父子窗口共享 vue store

结合商品中台配置可视化区域做一个通俗解释：  
- 首先我们把 vue 项目设置为多页应用，页面分别是商品预览页、商品管理页；
- 其次，调整 vue 入口，每个页面对应一个入口；
- 编写 iframe 组件和沙箱 vue
- 在商品管理页入口将沙箱 vue 和 store 挂载到 global 对象下
- 在商品预览入口将 global.parent 下的沙箱 vue 和 store 分别挂到 window 下和 global 下
- 其他的内容按照 vue 多页写法正常编写

这样可以让用 iframe 做展示容器的商品预览页和商品管理页共享 store  
为什么要使用沙箱 vue 呢？  
这是因为 vue 的单例机制，子窗口（商品管理页）由父窗口（商品管理页） new Vue 渲染的， 因此在子窗口中使用 use 、 filter 、 mixin 、 全局指令 、 全局组件等， 会覆盖父窗口 vue 对象。所以需要隔离出一个干净的和 vue 一样的 vue ，然后用隔离出的沙箱 vue 来渲染子窗口（商品预览页）的内容。这样就可以 达到父子窗口的 vue 互不影响。  
沙箱 vue 的实现非常巧妙：  
在 Function 上挂一个 $$clone 函数，这样 vue 下就会有 $$clone 函数，通过执行 Vue.$$clone() ，将 vue 的各种属性挂载到 SandboxVue 上。同时返回 SandboxVue 。即可得到一个干净的沙箱 vue 。  
注意：这里的 vue 指的是 vue2 ，目前 vue3 不是单例机制，在 vue3 中是不需要沙箱 vue 的。  
大多数 h5 、 pc 建站的数据通信方案都绕不开 postMessage 。而我们通过 uni-render ，让父窗口和 iframe 子窗口的数据通信不再需要 postMessage ，同时只使用 vue 生态中的 vuex 做数据通信。这带来了非常多的好处，好处如下：
- 统一数据通信方案
- 对 store 数据的 watch 、 computed 、更加纯正，数据通信功能更加强大
- 精简代码，和 postMessage 永久告别
- 支持同步函数，前方的路是星辰大海

## 商品中台微前端
**为什么要做微前端**  
主要有以下两个目的：  
- 将商品中台更快、更好的嵌入到各个业务方项目中
- 为后面主应用的设计做准备

把商品中台项目设计成了微前端架构，它可以很好的解决前端中台化所面临的各种问题  
## 商品中台微前端设计
微前端领域最主流的技术方案有以下两种：
- single-spa 技术方案
- iframe 技术方案

**qiankun 方案设计架构**  
为什么使用 qiankun ，最核心的原因是：在国内，使用最多的微前端框架就是 qiankun 。整体效果也不错。  
qiankun 的核心在于创建微应用容器。  
**接口跨域处理**  
解决接口跨域，主要有以下两种方式：  
- 主应用转发：接口的 host 与主应用一致，由主应用根据路径关键字 cmmdy 进行转发
- 微应用配置：微应用服务端配置允许跨域

**路由适配**  
微应用 router 需要添加 baseUrl ，并且要与主应用关键字 activeRule 保持一致。  
``` 
const KEY = 'product'
router = new VueRouter({
  mode: 'history',
  base: IN_CMS ? `/main/goods/${KEY}` : `/${KEY}`,
  routes
})
```

**数据通信**  
主应用与微应用之间如何通信？通信这块，主要有两种方案:
- initGlobalState：也是运行时通信(官方方案)；
- window：挂载到 window 下

initGlobalState 方案的优缺点如下：  
- 优点：api 提供了数据的 change 事件，双方均能监听到数据变化
- 缺点：微应用加载时，获取初始数据的时机太晚 ，不适合用作微应用数据的初始化

window 方案的优缺点如下：  
- 优点：微应用代码全周期内均可以获取数据，很好的避免官方方案中获取数据太晚的问题。
- 缺点：需要自己处理对数据变化的监听。

**环境区分**  
主要有以下两种场景：  
- 区分 qiankun 与非 qiankun 技术栈：使用 window.__POWERED_BY_QIANKUN__ 即可判断
- 区分同样使用 qiankun 的不同主应用：主应用与微应用之间约定参数，通过 window 对象或者生命周期函数中的 props 对象传递，来进行判断

**本地联调**  
本地没有主应用的服务，怎么实现主应用与微应用间的快速联调？解决方案如下：
主应用注册微应用时，将 entry 设置为从 localstorage 中获取，在 localstorage 中手动修改入口 entry 的值为微服务的本地地址，就可以实现本地的联调。核心代码如下：  
``` 
const timestamp = new Date().getTime()
const initEntry = (subSys) => {
  const LS_KEY_ENTRY = `__entry__${subSys}`
  const customEntry = localStorage.getItem(LS_KEY_ENTRY)
  if (customEntry) {
    return `${customEntry}`
  }
  if (subSys === 'goods') {
    return `//vshop-commodity.vivo.com.cn/goods/?t=${timestamp}`
  }
  return `${location.origin}/${subSys}/?t=${timestamp}`
}
```
通过上述代码，即可在主应用中对入口地址进行动态适配，达到灵活联调的目的。  
**踩坑经验分享**  
uni-render 相遇 qiankun 跨域问题  
现象：项目接入主应用，uni-render 控制的预览页面空白，控制台报跨域错误。  
原因：iframe 预览页面为商品中台域名，而子应用接入主应用后为主应用域名，从而导致跨域。  
解决方案：主应用、子应用 html 入口文件头部设置 document.domain ，使两者 domain 保持一致  

uni-render 、qiankun 、 ueditor ”化学反应“  
问题一：  
现象：qiankun 子应用中富文本组件 ueditor 功能异常。  
原因：qiankun 对 ueditor 劫持，导致 ueditor 某些变量无法获取到。  
解决方案：在主应用中，通过 excludeAssetFilter 让 ueditor 的静态资源不要被 qiankun 劫持处理。  
问题二：  
现象：子应用中 ueditor 的请求 url 报错。  
原因：ueditor 的请求 url 没加主应用请求前缀  
解决方案：子应用环境中，通过 ue.getActionUrl 给 ueditor 的请求 url 增加前缀。  
问题三：  
现象：子应用中 ueditor 单图上传失败。  
原因：子应用设置了 domain ， ueditor 的单图上传是通过 iframe 实现的，但是 iframe 没有设置 domain ，导致上传失败。
解决方案：重写 ueditor 的单图上传，将 iframe 改为 xhr 上传。  



参考:
[vivo 商品中台的可视化微前端实践](https://mp.weixin.qq.com/s/zYKwgfzC8Z-teo8OxznrmA)
