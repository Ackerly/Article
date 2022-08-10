# 基于微前端qiankun的多页签缓存方案实践
## 多页签是什么  
常见的浏览器多页签、编辑器多页签，从产品角度来说，就是为了能够实现用户访问可记录，快速定位工作区等作用；那对于单页应用，可以通过实现多页签，对用户的访问记录进行缓存，从而提供更好的用户体验。  
前端可以通过多种方式实现多页签，常见的方案有两种：  
- 通过 CSS 样式 display:none 来控制页面的显示隐藏模块的内容；
- 将模块序列化缓存，通过缓存的内容进行渲染（与 vue 的 keep-alive 原理类似，在单页面应用中应用广泛）

相对于第一种方式，第二种方式将 DOM 格式存储在序列化的 JS 对象当中，只渲染需要展示的 DOM 元素，减少了 DOM 节点数，提升了渲染的性能，是当前主流的实现多页签的方式  
**单页面应用实现多页签**  
vue 框架提供了 keep-alive 来支持缓存相关的需求，使用 keep-alive 即可实现多页签的基本功能，但是为了支持更多的功能，在其基础上重新封装了 vue-keep-alive 组件  
相对较于 keep-alive 通过 include、exclude 对缓存进行控制，vue-keep-alive 使用更原生的发布订阅方式来删除缓存，可以实现更完整的多页签功能，例如同个路由可以根据参数的不同派生出多个路由实例（如打开多个详情页页签）以及动态删除缓存实例等功能  
vue-keep-alive 自定义的拓展实现：  
``` 
created() {
   // 动态删除缓存实例监听
   this.cache = Object.create(null);
   breadCompBus.$on('removeTabByKey', this.removeCacheByKey);
   breadCompBus.$on('removeTabByKeys', (data) => {
     data.forEach((item) => {
       this.removeCacheByKey(item);
     });
   });
 }
```
vue-keep-alive 组件即可传入自定义方法，用于自定义 vnode.key，支持同一匹配路由中派生多个实例  
``` 
// 传入`vue-keep-alive`的自定义方法
 function updateComponentsKey(key, name, vnode) {
   const match = this.$route.matched[1];

   if (match && match.meta.multiNodeKey) {
     vnode.key = match.meta.multiNodeKey(key, this.$route);
     return vnode.key;
   }

   return key;
 }
```
**使用 qiankun 进行微前端改造后，多页签缓存有什么不同**  
qiankun 是由蚂蚁金服推出的基于 Single-Spa 实现的前端微服务框架，本质上还是路由分发式的服务框架，不同于原本 Single-Spa 采用 JS Entry 用的方案，qiankun 采用 HTML Entry 方式进行了替代优化  
使用 qiankun 进行微前端改造后，页面被拆分为一个基座应用和多个子应用，每个子应用都运行在独立的沙箱环境中  
相对于单页面应用中通过 keep-alive 管控组件实例的方式，拆分后的各个子应用的 keep-alive 并不能管控到其他子应用的实例，我们需要缓存对所有的应用生效，那么只能将缓存放到基座应用中  
存在几个问题：  
- 加载：主应用需要在什么时候，用什么方式来加载子应用实例？
- 渲染：通过缓存实例来渲染子应用时，是通过 DOM 显隐方式渲染子应用还是有其他方式？
- 通信：关闭页签时，如何判断是否完全卸载子应用，主应用应该使用什么通信方式告诉子应用

## 方案选择
**方案一：多个子应用同时存在**  
实现思路：  
- 在 dom 上通过 v-show 控制显示哪一个子应用，及 display:none; 控制不同子应用 dom 的显示隐藏。
- url 变化时，通过 loadMicroApp 手动控制加载哪个子应用，在页签关闭时，手动调用 unmount 方法卸载子应用

示例：  
``` 
<template>
   <div id="app">
   <header>
     <router-link to="/app-vue-hash/">app-vue-hash</router-link>
     <router-link to="/app-vue-history/">app-vue-history</router-link>
     <router-link to="/about">about</router-link>
   </header>
   <div id="appContainer1" v-show="$route.path.startsWith('/app-vue-hash/')"></div>
   <div id="appContainer2" v-show="$route.path.startsWith('/app-vue-history/')"></div>
   <router-view></router-view>
 </div>
 </template>

 <script>
 import { loadMicroApp } from 'qiankun';

 const apps = [
 {
   name: 'app-vue-hash',
   entry: 'http://localhost:1111',
   container: '#appContainer1',
   props: { data : { store, router } }
 },
 {
   name: 'app-vue-history',
   entry: 'http://localhost:2222',
   container: '#appContainer2',
   props: { data : store }
 }
 ]

 export default {
   mounted() {
     // 优先加载当前的子项目
     const path = this.$route.path;
     const currentAppIndex = apps.findIndex(item => path.includes(item.name));
     if(currentAppIndex !== -1){
       const currApp = apps.splice(currentAppIndex, 1)[0];
       apps.unshift(currApp);
     }
     // loadMicroApp 返回值是 app 的生命周期函数数组
     const loadApps = apps.map(item => loadMicroApp(item))
     // 当 tab 页关闭时，调用 loadApps 中 app 的 unmount 函数即可
   },
 }
 </script>
```
方案优势：  
- loadMicroApp 是 qiankun 提供的 API，可以方便快速接入；
- 该方式不卸载子应用，页签切换速度比较快

方案不足：  
- 子应用切换时不销毁 DOM，会导致 DOM 节点和事件监听过多，严重时会造成页面卡顿；
- 子应用切换时未卸载，路由事件监听也未卸载，需要对路由变化的监听做特殊的处理

**方案二：同一时间仅加载一个子应用，同时保存其他应用的状态**  
实现思路：
- 通过 registerMicroApps 注册子应用，qiankun 会通过自动加载匹配的子应用；
- 参考 keep-alive 实现方式，每个子应用都缓存自己实例的 vnode，下次进入子应用时可以直接使用缓存的 vnode 直接渲染为真实 DOM

方案优势：  
- 同一时间，只是展示一个子应用的 active 页面，可减少 DOM 节点数；
- 非 active 子应用卸载时同时会卸载 DOM 及不需要的事件监听，可释放一定内存

方案不足：  
- 没有现有的 API 可以快速实现，需要自己管理子应用缓存，实现较为复杂；
- DOM 渲染多了一个从虚拟 DOM 转化为真实 DOM 的一个过程，渲染时间会比第一种方案稍多

vue 关键渲染节点
- compile：对 template 进行编译，将 AST 转化后生成 render function;
- render：生成 VNODE 虚拟 DOM;
- patch ：将虚拟 DOM 转换为真实 DOM;

方案二相对于方案一，就是多了最后 patch 的过程  
**最终选择**  
根据两种方案优势与不足的评估，同时根据我们项目的具体情况，最终选择了方案二进行实现，具体原因如下：  
- 过多的 DOM 及事件监听，会造成不必要的内存浪费，同时我们的项目主要以编辑器展示和数据展示为主，单个页签内内容较多，会更倾向于关注内存使用情况；
- 方案二在子应用二次渲染时多了一个 patch 过程，渲染速度不会慢多少，在可接受范围内

## 具体实现
**从组件级别的缓存到应用级别的缓存**  
> 在 vue 中，keep-alive 组件通过缓存 vnode 的方式，实现了组件级别的缓存，对于通过 vue 框架实现的子应用来说，它其实也是一个 vue 实例，那么我们同样也可以做到通过缓存 vnode 的方式，实现应用级别的缓存

通过分析 keep-alive 源码，了解到 keep-alive 是通过在 render 中进行缓存命中，返回对应组件的 vnode，并在 mounted 和 updated 两个生命周期钩子中加入对子组件 vnode 的缓存  
``` 
// keep-alive核心代码
 render () {
   const slot = this.$slots.default   const vnode: VNode = getFirstComponentChild(slot)
   const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions   if (componentOptions) {
     // 更多代码...
     // 缓存命中
     if (cache[key]) {
       vnode.componentInstance = cache[key].componentInstance       // make current key freshest
       remove(keys, key)
       keys.push(key)
     } else {
       // delay setting the cache until update
       this.vnodeToCache = vnode       this.keyToCache = key     }
     // 设置keep-alive，防止再次触发created等生命周期
     vnode.data.keepAlive = true
   }
   return vnode || (slot && slot[0])
 }
 // mounted和updated时缓存当前组件的vnode
 mounted() {
   this.cacheVNode()
 }
 updated() {
   this.cacheVNode()
 }
```
相对于 keep-alive 需要在 mounted 和 updated 两个生命周期中对 vnode 缓存进行更新，在应用级的缓存中，只需要在子应用卸载时，主动对整个实例的 vnode 进行缓存即可  
``` 
// 父应用提供unmountCache方法
 function unmountCache() {
   // 此处永远只会保存首次加载生成的实例
   const needCached = this.instance?.cachedInstance || this.instance;
   const cachedInstance = {};
   cachedInstance._vnode = needCached._vnode;
   // keepalive设置为必须 防止进入时再次created，同keep-alive实现
   if (!cachedInstance._vnode.data.keepAlive) cachedInstance._vnode.data.keepAlive = true;
   // 省略其他代码...

   // loadedApplicationMap用于是key-value形式，用于保存当前应用的实例
   loadedApplicationMap[this.cacheKey] = cachedInstance;
   // 省略其他代码...

   // 卸载实例
   this.instance.$destroy();
   // 设置为null后可进行垃圾回收
   this.instance = null;
 }

 // 子应用在qiankun框架提供的卸载方法中，调用unmountCache
 export async function unmount() {
   console.log('[vue] system app unmount');
   mainService.unmountCache();
 }
```
**移花接木 —— 将 vnode 重新挂载到一个新实例上**  
将 vnode 缓存到内存中后，再将原有的 instance 卸载，重新进入子应用时，就可以使用缓存的 vnode 进行 render 渲染  
``` 
// 创建子应用实例，有缓存的vnode则使用缓存的vnode
 function newVueInstance(cachedNode) {
   const config = {
     router: this.router,
     store: this.store,
     render: cachedNode ? () => cachedNode : instance.render, // 优先使用缓存vnode
   });
   return new Vue(config);
 }

 // 实例化子应用实例，根据是否有缓存vnode确定是否传入cachedNode
 this.instance = newVueInstance(cachedNode);
 this.instance.$mount('#app');
```
这里不禁就会有些疑问：  
- 如果我们每次进入子应用时，都重新创建一个实例，那么为什么还要卸载，直接不卸载就可以了吗？
- 将缓存 vnode 使用到一个新的实例上，不会有什么问题吗

为什么在切换子应用时，要卸载掉原来的子应用实例，有两个考虑方面：  
其一，是对内存的考量，我们需要的其实仅仅是 vnode，而不是整个实例，缓存整个实例是方案一的实现方案，所以，我们仅需要缓存我们需要的对象即可；  
其二，卸载子应用实例可以移除不必要的事件监听，比如 vue-router 对 popstate 事件就进行了监听，我们在其他子应用操作时，并不希望原来的子应用也对这些事件进行响应，那么在子应用卸载时，就可以移除掉这些监听。

**解决应用级缓存方案的问题**  
vue-router 相关问题  
- 在实例卸载后对路由变化监听失效；
- 新的 vue-router 对原有的 router params 等参数记录失效

需要明确这两个问题的原因：  
第一个是因为在子应用卸载时移除了对 popstate 事件的监听，那么需要做的就是重新注册对 popstate 事件的监听，这里可以通过重新实例化一个 vue-router 解决；  
第二问题是因为通过重新实例化 vue-router 解决第一个问题之后，实际上是一个新的 vue-router，需要做的就是不仅要缓存 vnode，还需要缓存 router 相关的信息。  
大致的解决实现如下：  
``` 
// 实例化子应用vue-router
 function initRouter() {
   const { router: originRouter } = this.baseConfig;
   const config = Object.assign(originRouter, {
     base: `app-kafka/`,
   });
   Vue.use(VueRouter);
   this.router = new VueRouter(config);
 }

 // 创建子应用实例，有缓存的vnode则使用缓存的vnode
 function newVueInstance(cachedNode) {
   const config = {
     router: this.router, // 在vue init过程中，会重新调用vue-router的init方法，重新启动对popstate事件监听
     store: this.store,
     render: cachedNode ? () => cachedNode : instance.render, // 优先使用缓存vnode
   });
   return new Vue(config);
 }

 function render() {
   if（isCache） {
     // 场景一、重新进入应用（有缓存）
     const cachedInstance = loadedApplicationMap[this.cacheKey];

     // router使用缓存命中
     this.router = cachedInstance.$router;
     // 让当前路由在最初的Vue实例上可用
     this.router.apps = cachedInstance.catchRoute.apps;
     // 使用缓存vnode重新实例化子应用
     const cachedNode = cachedInstance._vnode;
     this.instance = this.newVueInstance(cachedNode);
   } else {
     // 场景二、首次加载子应用/重新进入应用（无缓存）
     this.initRouter();
     // 正常实例化
     this.instance = this.newVueInstance();
   }
 }

 function unmountCache() {
   // 省略其他代码...
   cachedInstance.$router = this.instance.$router;
   cachedInstance.$router.app = null;
   // 省略其他代码...
 }
```

_父子组件通信_  
多页签的方式增加了父子组件通信的频率，qiankun 有提供 setGlobalState 通信方式，但是在单应用模式下，同一时间仅支持和一个子应用进行通行，对于 unmount 的子应用来说，无法接收到父应用的通信，因此，对于不同的场景，我们需要更加灵活的通信方式  
子应用 —— 父应用：使用 qiankun 自带通信方式:  
从子到父的通信场景较为简单，一般只有路由变化时进行上报，并且仅为激活状态的子应用才会上报，可直接使用 qiankun 自带通信方式；
父应用 —— 子应用：使用自定义事件通信:  
父应用到子应用，不仅需要和 active 状态的子应用通信，还需要和当前处于缓存中子应用通信；  
因此，父应用到子应用，通过自定义事件的方式，能够实现父应用和多个子应用的通信。  
``` 
// 自定义事件发布
 const evt = new CustomEvent('microServiceEvent', {
   detail: {
     action: { name: action, data },
     basePath, // 用于子应用唯一标识
   },
 });
 document.dispatchEvent(evt);

 // 自定义事件监听
 document.addEventListener('microServiceEvent', this.listener);
```
_缓存管理，防止内存泄露_  
应用级缓存  
子应用 vnode、router 等属性，子应用切换时缓存；  
页面级缓存  
- 通过 vue-keep-alive 缓存组件的 vnode；
- 删除页签时，监听 remove 事件，删除页面对应的 vnode；
- vue-keep-alive 组件中所有缓存均被删除时，通知删除整个子应用缓存

## 现有问题
**暂时只支持 vue 框架的实例缓存**  
该方案也是基于 vue 现有特性支持实现的，在 react 社区中对于多页签实现并没有统一的实现方案  

参考:  
[基于微前端qiankun的多页签缓存方案实践](https://mp.weixin.qq.com/s/aHDcrrc3oLwaLpXakIwjEA)
