# 构建可控,可靠,可扩展的PWA应用
## 概述
PWA （Progressive Web App）指的是使用指定技术和标准模式来开发的 Web 应用，让 Web 应用具有原生应用的特性和体验。比如我们觉得本地应用使用便捷，响应速度更加快等。  
PWA 的关键技术有两个：  
- Manifest：浏览器允许你提供一个清单文件，从而实现 A2HS
- ServiceWorker：通过对网络请求的代理，从而实现资源缓存、站点加速、离线应用等场景。

其次还有诸如：消息推送、WebStream、Web蓝牙、Web分享、硬件访问等API。出于浏览器厂商的支持不一，普及度还不高。  
使用 ServiceWorker 来优化用户体验，已经成为Web前端优化的主流技术。  
## 工具与框架  
2018 年之前，主流的工具是：  
- google/sw-toolbox: 提供了一套工具，用于方便的构建 ServiceWorker。
- google/sw-precache: 提供在构建阶段，注入资源清单到 ServiceWorker 中，从而实现预缓存功能。
- baidu/Lavas: 百度开发的基于 Vue 的 PWA 集成解决方案。

后来由于 Google 开发了更加优秀的工具集 Workbox，sw-toolbox 和 sw-precache 得以退出舞台。  

**痛点**  
Workbox 提供了一套工具集合，用以帮助我们管理 ServiceWorker ，它对 CacheStorage 的封装，也得以让我们更轻松的去管理资源。  
在构建实际的 PWA 应用的时候，还需要关心很多问题：  
- 如何组织工程和代码？
- 如何进行单元测试？
- 如何解决 MPA (Multiple Page Application) 应用间的 ServiceWorker 作用域冲突问题？
- 如何远程控制我们的 ServiceWorker？
- 最优的资源缓存方案？
- 如何监控我们的 ServiceWorker，收集数据？

由于 Workbox 的定位是 「Library」，而我们需要一个 「Framework」 去为这些通用问题提供统一的解决方案。并且， 我们希望它是渐进式（Progressive）的，就犹如 PWA 所提倡的那样。  

## 代码解耦
**是什么问题？**  
当我们的 ServiceWorker 程序代码越来越多的时候，会造成代码臃肿，管理混乱，复用困难。
同时一些常见的实现，如：远程控制、进程通讯、数据上报等，希望能实现按需插拔式的复用，这样才能达到「渐进式」的目的。  
ServiceWorker 在运行时提供了一系列事件，常用的有：  
```
self.addEventListener('install', event => { });
self.addEventListener('activate', event => { });
self.addEventListener("fetch", event => { });
self.addEventListener('message', event => { });
```
有多个功能实现都要监听相同的事件，就会导致同个文件的代码越来越臃肿：
``` 
self.addEventListener('install', event => {
  // 远程控制模块 - 配置初始化
  ...
  // 资源预缓存模块 - 缓存资源
  ...
  // 数据上报模块 - 收集事件
  ...
});
  
self.addEventListener('activate', event => {
  // 远程控制模块 - 刷新配置
  ...
  // 数据上报模块 - 收集事件
  ...
});
  
self.addEventListener("fetch", event => {
  // 远程控制模块 - 心跳检查
  ...
  // 资源缓存模块 - 缓存匹配
  ...
  // 数据上报模块 - 收集事件
  ...
});

self.addEventListener('message', event => {
  // 数据上报模块 - 收集事件
  ...
});
```
进行模块化
``` 
import remoteController from './remoete-controller.ts';  // 远程控制模块
import assetsCache from './assets-cache.ts';  // 资源缓存模块
import collector from './collector.ts';  // 数据收集模块
import precache from './pre-cache.ts';  // 资源预缓存模块

self.addEventListener('install', event => {
  // 远程控制模块 - 配置初始化
  remoteController.init(...);
  // 资源预缓存模块 - 缓存资源
  assetsCache.store(...);
  // 数据上报模块 - 收集事件
  collector.log(...);
});
  
self.addEventListener('activate', event => {
  // 远程控制模块 - 刷新配置
  remoteController.refresh(..);
  // 数据上报模块 - 收集事件
  collector.log(...);
});
  
self.addEventListener("fetch", event => {
  // 远程控制模块 - 心跳检查
  remoteController.heartbeat(...);
  // 资源缓存模块 - 缓存匹配
  assetsCache.match(...);
  // 数据上报模块 - 收集事件
  collector.log(...);
});

self.addEventListener('message', event => {
  // 数据上报模块 - 收集事件
  collector.log(...);
});
```
模块化能减少主文件的代码量，同时也一定程度上对功能进行了解耦，但是这种方式还存在一些问题：  
- 复用困难：当要使用一个模块的功能时，要在多个事件中去正确的调用模块的接口。同样，要去掉一个模块事，也要多个事件中去修改。
- 使用成本高：模块暴露各种接口，使用者必须了解透彻模块的运转方式，以及接口的使用，才能很好的使用。
- 解耦有限：如果模块更多，甚至要解决同域名下多个前端应用的命名空间冲突问题，就会显得捉襟见肘。

要达到我们目的：「渐进式」，我们需要对代码的组织再优化一下。  
**插件化实现**  
可以把 ServiceWorker 的一系列事件的控制权交出去，各模块通过插件的方式来使用这些事件。  
洋葱模型是「插件化」的很好的思想，但是它是 「一维」 的，Koa 完成一次网络请求的应答，各个中间件只需要监听一个事件。  
在 ServiceWorker 中，除了上面提及到的常用四个事件，他还有更多事件，如：SyncEvent, NotificationEvent。所以，还要多弄几个「洋葱」去满足更多的事件。  
同时由于 PWA 应用的代码一般会运行在两个线程：主线程、ServiceWorker 线程。最后，我们去封装原生的事件，去提供插件化支持，从而有了：「多维洋葱插件系统」  
对原生事件和生命周期进行封装之后，我们为每一个插件提供更优雅的生命周期钩子函数  
基于 GlacierJS 的话，可以很容易做到模块的插件化  
在 ServiceWorker 线程的主文件中注册插件：  
``` 
import { GlacierSW } from '@glacierjs/sw';
import RemoteController from './remoete-controller.ts';  // 远程控制模块
import AssetsCache from './assets-cache.ts';  // 资源缓存模块
import Collector from './collector.ts';  // 数据收集模块
import Precache from './pre-cache.ts';  // 资源预缓存模块
import MyPluginSW from './my-plugin.ts'

const glacier = new GlacierSW();

glacier.use(new Log(...));
glacier.use(new RemoteController(...));
glacier.use(new AssetsCache(...));
glacier.use(new Collector(...));
glacier.use(new Precache(...));

glacier.listen();
```
在插件中，可以通过监听事件去收归一个独立模块的逻辑：  
``` 
import { ServiceWorkerPlugin } from '@glacierjs/sw';
import type { FetchContext, UseContext  } from '@glacierjs/sw';

export class MyPluginSW implements ServiceWorkerPlugin {
    constructor() {...}
    public async onUse(context: UseContext) {...}
    public async onInstall(event) {...}
    public async onActivate() {...}
    public async onFetch(context: FetchContext) {...}
    public async onMessage(event) {...}
    public async onUninstall() {...}
}
```
## 作用域冲突
ServiceWorker 的作用域有两个关键特性  
- 默认的作用域是注册时候的 Path。
- 同个路径下同时间只能有一个 ServiceWorker 得到控制权

**作用域缩小与扩大**  
关于第一个特性，例如注册 Service Worker 文件为 /a/b/sw.js，则 scope 默认为 /a/b/：  
``` 
if (navigator.serviceWorker) {
    navigator.serviceWorker.register('/a/b/sw.js').then(function (reg) {
        console.log(reg.scope);
        // scope => https://yourhost/a/b/
    });
}
```
可以在注册的的时候指定 scope 去向下缩小作用域，例如：  
``` 
if (navigator.serviceWorker) {
    navigator.serviceWorker.register('/a/b/sw.js', {scope: '/a/b/c/'})
        .then(function (reg) {
            console.log(reg.scope);
            // scope => https://yourhost/a/b/c/
        });
}
```
可以通过服务器对 ServiceWorker 文件的响应设置 Service-Worker-Allowed 头部，去扩大作用域。  


参考:
[如何构建可控,可靠,可扩展的 PWA 应用](https://mp.weixin.qq.com/s/4fuP1puANOOGdGj0oAny1Q)
