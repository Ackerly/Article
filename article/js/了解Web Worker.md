# 了解Web Worker
JavaScript 是单线程的语言。当我们面临需要大量计算的场景时（比如视频解码等），UI 线程就会被阻塞，甚至浏览器直接卡死。现在前端遇到大量计算的场景越来越多，为了有更好的体验，HTML5 中提出了 Web Worker 的概念。Web Worker 可以使脚本运行在新的线程中，它们独立于主线程，可以进行大量的计算活动，而不会影响主线程的 UI 渲染。当计算结束之后，它们可以把结果发送给主线程，从而形成了高效、良好的用户体验。Web Worker 是一个统称，具体可以细分为普通的 Worker、SharedWorker 和 ServiceWorker 等，接下来我们一一介绍其使用方法和适合的场景。  
## 普通 Worker
1. 创建 Worker 通过 new 的方式来生成一个实例，参数为 url 地址，该地址必须和其创建者是同源的。
``` 
const worker = new Worker('./worker.js'); // 参数是 url，这个 url 必须与创建者同源 
```
2. Worker 的方法
   - onmessage 主线程中可以在 Worker 上添加 onmessage 方法，用于监听 Worker 的信息。
   ``` 
   const worker = new Worker('./worker.js');
    worker.onmessage = function (messageEvent) {
    console.log(messageEvent)
    }
   ```
   - onmessageerror 主线程中可以在 Worker 上添加 onmessageerror 方法，用于监听 Worker 的错误信息。
   ``` 
   const worker = new Worker('./worker.js');
    worker.onmessageerror = function (messageEvent) {
    console.log(messageEvent)
    }
   ```
   - postMessage() 主线程通过此方法给 Worker 发送消息，发送参数的格式不限（可以是数组、对象、字符串等），可以根据自己的业务选择。
   ``` 
   const worker = new Worker('./worker.js');
   worker.postMessage({ type: 'start', payload: { count: 666 } }); // 发送信息给worker
   ```
   - terminate() 主线程通过此方法终止 Worker 的运行。
   ``` 
   const worker = new Worker('./worker.js');
   worker.terminate();
   ```
3. 通信  
Worker 的作用域跟主线程中的 Window 是相互独立的，并且 Worker 中是获取不到 DOM 元素的。所以在 Worker 中你无法使用 Window 变量。取而代之的是可以用 self 来表示全局对象。比较常用的方法是 onmessage、postMessage，主要用来跟主线程进行通信。  
  ``` 
  // 监听事件，主线程可以通过 postMessage 发送信息过来
  self.onmessage = (messageEvent) => {
  const { type, payload } = messageEvent.data;
    switch (type) {
      case 'start':
        // 通过 type 去区分不同的业务逻辑，payload 是传过来的数据
        const result = 0;
        // ....,通过一系列处理之后，把最终的结果发送给主线程
        this.postMessage(result);
        break;
    }
  };
  ```
  从 messageEvent.data 中获取从主线程传递过来的数据。为了业务的扩展性，这边是以 type 去区分不同的业务，payload 承载数据源，通过处理之后把结果发给主线程。主线程的 onmessage 回调函数中就能收到这个结果了。  
4. Worker 中引用其他脚本的方式  
跟常用的 JavaScript 一样，Worker 中也是可以引入其他的模块的。但是方式不太一样，是通过 importScripts 来引入。
``` 
// Worker.js
importScripts('constant.js');
// 下面就可以获取到 constant.js 中的所有变量了

// constant.js
// 可以在 Worker 中使用
const a = 111;

// 不可以在 Worker 中使用，原因未知
const b = function () {
  console.log('test');
};

// 可以在 Worker 中使用
function c() {
  console.log('test');
}
```
5. 调试方法(Worker 的调试在浏览器控制台中有专门展示的地方)
6. 常见使用场景
    - 一般的视频网站 以优酷为例，当我们开始播放优酷视频的时候，就能看到它会调用 Worker，解码的代码应该写在 Worker 里面。
    - 需要大量计算的网站 比如 imgcook 这个网站，它能在前端解析 sketch 文件，这部分解析的逻辑就写在 Worker 里。

## SharedWorker
SharedWorker 是一种特定的 Worker。从它的命名就能知道，它是一种共享数据的 Worker。它可以同时被多个浏览器环境访问。这些浏览器环境可以是多个 window, iframes 或者甚至是多个 Worker，只要这些 Workers 处于同一主域。为跨浏览器 tab 共享数据提供了一种解决方案。  
1. 创建 SharedWorker  
   创建的方法跟上面普通 Worker 完全一模一样。
    ``` 
   const worker = new SharedWorker("./shareWorker.js"); // 参数是 url，这个 url 必须与创建者同源 
    ```
2. SharedWorker 的方法  
   haredWorker 的方法都在 port 上，这是它与普通 Worker 不同的地方。
   - port.onmessage  
     主线程中可以在 worker 上添加 onmessage 方法，用于监听 SharedWorker 的信息
     示例：
    ``` 
    const sharedWorker = new SharedWorker('./shareWorker.js');
    sharedWorker.port.onmessage = function (messageEvent) {
    console.log(messageEvent)
    }
    ```
   - port.postMessage()  
   主线程通过此方法给 SharedWorker 发送消息，发送参数的格式不限  
   示例：  
   ``` 
    const sharedWorker = new SharedWorker('./shareWorker.js');
    sharedWorker.port.postMessage({ type: 'increase', payload: { count: 666 } });
   ```
   - port.start()  
   主线程通过此方法开启 SharedWorker 之间的通信  
    ``` 
    const sharedWorker = new SharedWorker('./shareWorker.js');
    sharedWorker.port.start()
    ```
    - port.close()  
    主线程通过此方法关闭 SharedWorker  
   ``` 
    const sharedWorker = new SharedWorker('./shareWorker.js');
    sharedWorker.port.close()
   ```
3. 通信  
   SharedWorker 跟普通的 Worker 一样，可以用 self 来表示全局对象。不同之处是，它需要等 port 连接成功之后，利用 port 的onmessage、postMessage，来跟主线程进行通信。当你打开多个窗口的时候，SharedWorker 的作用域是公用的，这也是其特点。  
    ``` 
   // index.js
    const worker = new SharedWorker('./shareWorker.js');
    
    worker.port.start(); // 开启端口
    
    // 发送信息给 shareWorker
    worker.port.postMessage({ type: 'increase', payload: { count: 666 } });
    
    // 接受 shareWorker 发过来的数据
    worker.port.onmessage = function (val) {
    console.log(val.data)
    };
    
    // shareWorker.js
    let count = 666;
    
    port.onmessage = (messageEvent) => {
    const { type, payload } = messageEvent.data;
    
    switch (type) {
    case 'increase':
    port.postMessage(++count);
    break;
    case 'decrease':
    port.postMessage(--count);
    break;
    }
    };
    ```
4. Worker 中引用其他脚本  
   这个与普通的 Worker 方法一样，使用 importScripts
5. 调试方法  
   在浏览器中查看和调试 SharedWorker 的代码，需要输入 chrome://inspect/

## ServiceWorker
ServiceWorker 一般作为 Web 应用程序、浏览器和网络之间的代理服务。他们旨在创建有效的离线体验，拦截网络请求，以及根据网络是否可用采取合适的行动，更新驻留在服务器上的资源。他们还将允许访问推送通知和后台同步 API。  
1. 创建 ServiceWorker
``` 
// index.js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', function () {
    navigator.serviceWorker
      .register('./serviceWorker.js', { scope: '/page/' })
      .then(
      function (registration) {
        console.log('ServiceWorker registration successful with scope: ',
                    registration.scope);
      },
      function (err) {
        console.log('ServiceWorker registration failed: ', err);
      }
    );
  });
}
```
只要创建了 ServiceWorker，不管这个创建 ServiceWorker 的 html 是否打开，这个 ServiceWorker 是一直存在的。它会代理范围是根据 scope 决定的，如果没有这个参数，则其代理范围是创建目录同级别以及子目录下所有页面的网络请求。代理的范围可以通过 registration.scope 查看。  
2. 安装 ServiceWorker  
``` 
// serviceWorker.js
const CACHE_NAME = 'cache-v1';
// 需要缓存的文件
const urlsToCache = [
  '/style/main.css',
  '/constant.js',
  '/serviceWorker.html',
  '/page/index.html',
  '/serviceWorker.js',
  '/image/131.png',
];
self.oninstall = (event) => {
  event.waitUntil(
    caches
    .open(CACHE_NAME) // 这返回的是 promise
    .then(function (cache) {
      return cache.addAll(urlsToCache); // 这返回的是 promise
    })
  );
};
```
在 install 事件的回调中，我们打开了名字为 cache-v1 的缓存，它返回的是一个 promise。在打开缓存之后，我们需要把要缓存的文件 add 进去，基本上所有类型的资源都可以进行缓存，例子中缓存了 css、js、html、png。如果所有缓存数据都成功，就表示 ServiceWorker 安装成功；如果控制台提示 Uncaught (in promise) TypeError: Failed to execute 'Cache' on 'addAll': Request failed，则表示安装失败。
3. 缓存和返回请求
``` 
self.onfetch = (event) => {
  event.respondWith(
    caches
    .match(event.request) // 此方法从服务工作线程所创建的任何缓存中查找缓存的结果
    .then(function (response) {
      // response 为匹配到的缓存资源，如果没有匹配到则返回 undefined，需要 fetch 资源
      if (response) {
        return response;
      }
      return fetch(event.request);
    })
  );
};
```
在 fetch 事件的回调中，我们去匹配 cache 中的资源。如果匹配到，则使用缓存资源；没有匹配到则用 fetch 请求。正因为 ServiceWorker 可以代理网络请求，所以为了安全起见，规范中规定它只能在 https 和 localhost 下才能开启。  
4. 调试方法  
   在浏览器中查看和调试 ServiceWorker 的代码，需要输入 chrome://inspect/#service-workers  
5. 效果  
   缓存了 131.png。切换到离线模式，131 图片还是能显示。这个图片它存到哪里去了？实际上它会把文件自动存到浏览器的 Cache Storage 中。我们打开浏览器可以看到。  
6. 常见使用场景  
   缓存资源文件，加快渲染速度  

|  类型   | Worker         | SharedWorker   | ServiceWorker  |
|  ----  |------------|  ----  | ----  |
| 通信方式  | postMessage        | port.postMessage  | 单向通信，通过addEventListener 监听 serviceWorker 的状态 |
| 使用场景  | 适合大量计算的场景        | 适合跨 tab、iframes之间共享数据  | 缓存资源、网络优化 |
| 兼容性  | >= IE 10   >= Chrome 4| 不支持 IE、Safari、Android、iOS>= Chrome 4  | 不支持 IE>= Chrome 40 |

参考:
[Web Worker](https://mp.weixin.qq.com/s/4cKldlMlTPrSJG3an7PB5g)
