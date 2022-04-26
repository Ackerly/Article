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
**MPA下的 ServiceWorker 治理**  
现代 Web App 项目主要有两种架构形式存在：SPA(Single Page Application) 和 MPA(Multiple Page Application)  
MPA 这种架构的模式在现如今的大型 Web App 非常常见，这种 Web App 相比较于 SPA 能够承受更重的业务体量，并且利于大型 Web App 的后期维护和扩展，它往往会有多个团队去维护。  
假设我们有一个 MPA 的站点：  
``` 
.
|-- app1
|   |-- app1-service-worker.js
|   `-- index.html
|-- app2
|   `-- index.html
|-- index.html
`-- root-service-worker.js
```
app1 和 app2 分别由不同的团队维护。  
如果我们在根目录 '/' 注册了 root-service-worker.js，去完成一些通用的功能，例如：「日志收集」、「静态资源缓存」等。  
然后 app1 团队利用 ServiceWorker 的能力开发了一些特定的功能需要，例如 app1 的「离线化功能」。  
他们在 app1/index.html 目录注册了 app1-service-worker.js。  
访问 app1/* 下的所有页面，ServiceWorker 控制权会交给 app1-service-worker.js，也就是只有app1的「离线化功能」在工作，而原来的「日志收集」、「静态缓存」等功能会失效。  
解决这个问题有两种方案：  
- 封装「日志收集」、「静态资源缓存」功能，app1-service-worker.js引入并使用这些功能。
- 把「离线化功能」整合到 root-service-worker.js，只允许注册该 ServiceWorker。

关于方案一，封装通用功能这是正确的，但是主域下的功能可能完全没办法一一拆解，并且后续主域的 ServiceWorker 更新了新功能，子域下的 ServiceWorker 还需要主动去更新和升级。  
关于方案二，显然可以解决方案一的问题，但是其他应用，例如 app2 可能不需要「离线化功能」。 基于此，我们引入方案三：功能整合到主域，支持功能的组合按照作用域隔离。  
基于 GlacierJS 的话代码上可能会是这样的  
``` 
const mainPlugins = [
  new Collector(); // 日志收集功能
  new AssetsCache(); // 静态资源缓存功能
];

glacier.use('/', mainPlugins)；
glacier.use('/app1', [
  ...mainPlugins,
  new Offiline(),  // 离线化功能
])；
```
## 资源缓存
ServiceWorker 一个很核心的能力就是能结合 CacheAPI 进行灵活的缓存资源，从而达到优化站点的加载速度、弱网访问、离线应用等。  
对于静态资源有五种常用的缓存策略：  
- stale-while-revalidate: 该模式允许您使用缓存（如果可用）尽快响应请求，如果没有缓存则回退到网络请求，然后使用网络请求来更新缓存，它是一种比较安全的缓存策略。
- cache-first: 离线 Web 应用程序将严重依赖缓存，但对于非关键且可以逐渐缓存的资源，**「缓存优先」**是最佳选择。如果缓存中有响应，则将使用缓存的响应来满足请求，并且根本不会使用网络。如果没有缓存响应，则请求将由网络请求完成，然后响应会被缓存，以便下次直接从缓存中提供下一个请求。
- network-first: 对于频繁更新的请求，**「网络优先」**策略是理想的解决方案。默认情况下，它会尝试从网络获取最新响应。如果请求成功，它会将响应放入缓存中。如果网络未能返回响应，则将使用缓存的响应。
- network-only: 如果您需要从网络满足特定请求，network-only 模式会将资源请求进行透传到网络
- cache-only: 该策略确保从缓存中获取响应。这种场景不太常见，它一般匹配着「预缓存」策略会比较有用。

那这些策略中，我们应该使用哪种呢？答案是根据资源的种类具体选择。  
例如一些资源如果只是在 Web 应用发布的时候才会更新，我们就可以使用 cache-first 策略，例如一些 JS、样式、图片等。
而 index.html 作为页面的加载的主入口，更加适宜使用 stale-while-revalidate 策略。  
以 GlacierJS 的缓存插件（@glacierjs/plugin-assets-cache）为例：  
``` 
// in service-worker.js
importScripts("//cdn.jsdelivr.net/npm/@glacierjs/core/dist/index.min.js");
importScripts('//cdn.jsdelivr.net/npm/@glacierjs/sw/dist/index.min.js');
importScripts('//cdn.jsdelivr.net/npm/@glacierjs/plugin-assets-cache/dist/index.min.js');

const { GlacierSW } = self['@glacierjs/sw'];
const { AssetsCacheSW, Strategy } = self['@glacierjs/plugin-assets-cache'];

const glacierSW = new GlacierSW();

glacierSW.use(new AssetsCacheSW({
    routes: [{
        // capture as string: store index.html with stale-while-revalidate strategy.
        capture: 'https://mysite.com/index.html',
        strategy: Strategy.STALE_WHILE_REVALIDATE,
    }, {
        // capture as RegExp: store all images with cache-first strategy
        capture: /\.(png|jpg)$/,
        strategy: Strategy.CACHE_FIRST
    }, {
        // capture as function: store all stylesheet with cache-first strategy
        capture: ({ request }) => request.destination === 'style',
        strategy: Strategy.CACHE_FIRST
    }],
}));
```
**远程控制**  
基于 ServiceWorker 的原理，一旦在浏览器安装上了，如果遇到紧急线上问题，唯有发布新的 ServiceWorker 才能解决问题。但是 ServiceWorker 的安装是有时延的，再加上有些团队从修改代码到发布的流程，这个反射弧就很长了。我们有什么办法能缩短对于线上问题的反射弧呢？  
怎么去获取配置呢？  
方案一，如果我们在主线程中获取配置：  
- 需要用户主动刷新页面才会生效。
- 做不到轻量的功能关闭，什么意思呢，我们会有开关的场景，主线程只能通过卸载或者清理缓存去实现「关闭」，这个太重了。
方案二，如果我们在 ServiceWorker 线程去获取配置：  
- 可以实现轻量功能关闭，透传请求就行了。
- 但是如果遇到要干净的清理用户环境的需要，去卸载 ServiceWorker 的时候，就会导致主进程每次注册，到了 ServiceWorker 就卸载，造成频繁安装卸载。

所以我们的 最后方案 是 「基于双线程的实时配置获取」。  
主线程也要获取配置，然后配置前面要加上防抖保护，防止 onFetch 事件短时间并发的问题。  
使用 Glacier 的插件 @glacierjs/plugin-remote-controller 可以轻松实现远程控制：  
```  
// in ./remote-controller-sw.ts
import { RemoteControllerSW } from '@glacierjs/plugin-remote-controller';
import { GlacierSW } from '@glacierjs/sw';
import { options } from './options';

const glacierSW = new GlacierSW();
glacierSW.use(new RemoteControllerSW({
  fetchConfig: () => getMyRemoteConfig();
}));

// 其中 getMyRemoteConfig 用于获取你存在远端的配置，返回的格式规定如下：
const getMyRemoteConfig = async () => {
    const config: RemoteConfig = {
        // 全局关闭，卸载 ServiceWorker
        switch: true,
      
        // 缓存功能开关
        assetsEnable: true,

                // 精细控制特定缓存
        assetsCacheRoutes: [{
            capture: 'https://mysite.com/index.html',
            strategy: Strategy.STALE_WHILE_REVALIDATE,
        }],
    },
}
```
## 数据收集
ServiceWorker 发布之后，我们需要保持对线上情况的把控。对于一些必要的统计指标，我们可能需要进行上统计和上报。  
@glacierjs/plugin-collector 内置了五个常见的数据事件：  
- ServiceWorker 注册：SW_REGISTER
- ServiceWorker 安装成功：SW_INSTALLED
- ServiceWorker 控制中：SW_CONTROLLED
- 命中 onFetch 事件：SW_FETCH
- 命中浏览器缓存：CACHE_HIT of CacheFrom.Window
- 命中 CacheAPI 缓存：CACHE_HIT of CacheFrom.SW

基于以上数据的收集，我们就可以得到一些常见的通用指标：  
- ServiceWorker 安装率 = SW_REGISTER / SW_INSTALLED
- ServiceWorker 控制率 = SW_REGISTER / SW_CONTROLLED
- ServiceWorker 缓存命中率 = SW_FETCH / CACHE_HIT (of CacheFrom.SW)

在 ServiceWorker 线程中注册 plugin-collector：  
``` 
import { AssetsCacheSW } from '@glacierjs/plugin-assets-cache';
import { CollectorSW } from '@glacierjs/plugin-collector';
import { GlacierSW } from '@glacierjs/sw';

const glacierSW = new GlacierSW();

// should use plugin-assets-cache first in order to make CollectedDataType.CACHE_HIT work.
glacierSW.use(new AssetsCacheSW({...}));
glacierSW.use(new CollectorSW());
```
在主线程中注册 plugin-collector，并且监听数据事件，进行数据上报：  
``` 
import {
  CollectorWindow,
  CollectedData,
  CollectedDataType,
} from '@glacierjs/plugin-collector';
import { CacheFrom } from '@glacierjs/plugin-assets-cache';
import { GlacierWindow } from '@glacierjs/window';

const glacierWindow = new GlacierWindow('./service-worker.js');

glacierWindow.use(new CollectorWindow({
    send(data: CollectedData) {
      const { type, data } = data;

      switch (type) {
        case CollectedDataType.SW_REGISTER:
          myReporter.event('sw-register-count');
          break;

        case CollectedDataType.SW_INSTALLED:
          myReporter.event('sw-installed-count');
          break;

        case CollectedDataType.SW_CONTROLLED:
          myReporter.event('sw-controlled-count');
          break;

        case CollectedDataType.SW_FETCH:
          myReporter.event('sw-fetch-count');
          break;

        case CollectedDataType.CACHE_HIT:
          // hit service worker cache
          if (data?.from === CacheFrom.SW) {
            myReporter.event(`sw-assets-count:hit-sw-${data?.url}`);
          }

          // hit browser cache or network
          if (data?.from === CacheFrom.Window) {
            myReporter.event(`sw-assets-count:hit-window-${data?.url}`);
          }
          break;
      }
    },
}));
```
其中 myReporter.event 是你可能会实现的数据上报库  
## 单元测试
ServiceWorker 测试可以分解为常见的测试组.
在顶层的是 「集成测试」，在这一层，我们检查整体的行为，例如：测试页面可加载，ServiceWorker注册，离线功能等。集成测试是最慢的，但是也是最接近现实情况的。  
再往下一层的是 「浏览器单元测试」，由于 ServiceWorker 的生命周期，以及一些 API 只有在浏览器环境下才能有，所以我们使用浏览器去进行单元测试，会减少很多环境的问题。  
接着是 「ServiceWorker 单元测试」，这种测试也是在浏览器环境中注册了测试用的 ServiceWorker 为前提进行的单元测试。
最后一种是 「模拟 ServiceWorker」，这种测试粒度会更加精细，精细到某个类某个方法，只检测入参和返回。这意味着没有了浏览器启动成本，并且最终是一种可预测的方式测试代码的方式。
但是模拟 ServiceWorker 是一件困难的事情，如果 mock 的 API 表面不正确，则在集成测试或者浏览器单元测试之前问题不会被发现。我们可以使用 service-worker-mock 或者 MSW 在 NodeJS 环境中进行 ServiceWorker 的单元测试。

参考:
[如何构建可控,可靠,可扩展的 PWA 应用](https://mp.weixin.qq.com/s/4fuP1puANOOGdGj0oAny1Q)
