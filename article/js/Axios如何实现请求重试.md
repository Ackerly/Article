# Axios 如何实现请求重试
## 拦截器实现请求重复的方案
Axios 是一个基于 Promise 的 HTTP 客户端，同时支持浏览器和 Node.js 环境。对于浏览器环境来说，Axios 底层是利用 XMLHttpRequest 对象来发起 HTTP 请求。如果要取消请求的话，我们可以通过调用 XMLHttpRequest 对象上的 abort 方法来取消请求  
``` 
let xhr = new XMLHttpRequest();
xhr.open("GET", "https://developer.mozilla.org/", true);
xhr.send();
setTimeout(() => xhr.abort(), 300);
```
对于 Axios 来说，我们可以通过 Axios 内部提供的 CancelToken 来取消请求：
``` 
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.post('/user/12345', {
  name: 'semlinker'
}, {
  cancelToken: source.token
})

source.cancel('Operation canceled by the user.'); // 取消请求，参数是可选的
```
也可以通过调用 CancelToken 的构造函数来创建 CancelToken
``` 
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    cancel = c;
  })
});

cancel(); // 取
```
**判断请求重复**  
当请求方式、请求 URL 地址和请求参数都一样时，我们就可以认为请求是一样的。因此在每次发起请求时，我们就可以根据当前请求的请求方式、请求 URL 地址和请求参数来生成一个唯一的 key，同时为每个请求创建一个专属的 CancelToken，然后把 key 和 cancel 函数以键值对的形式保存到 Map 对象中，使用 Map 的好处是可以快速的判断是否有重复的请求：  
``` 
import qs from 'qs'

const pendingRequest = new Map();
// GET -> params；POST -> data
const requestKey = [method, url, qs.stringify(params), qs.stringify(data)].join('&'); 
const cancelToken = new CancelToken(function executor(cancel) {
  if(!pendingRequest.has(requestKey)){
    pendingRequest.set(requestKey, cancel);
  }
})
```
当出现重复请求的时候，可以使用 cancel 函数来取消前面已经发出的请求，在取消请求之后，还需要把取消的请求从 pendingRequest 中移除。  
**取消重复请求**  
使用 Axios 的拦截器机制来实现取消重复请求的功能。Axios 为开发者提供了请求拦截器和响应拦截器，它们的作用如下：  
- 请求拦截器：该类拦截器的作用是在请求发送前统一执行某些操作，比如在请求头中添加 token 字段。
- 响应拦截器：该类拦截器的作用是在接收到服务器响应后统一执行某些操作，比如发现响应状态码为 401 时，自动跳转到登录页。

**定义辅助函数**  
generateReqKey：用于根据当前请求的信息，生成请求 Key；
``` 
function generateReqKey(config) {
  const { method, url, params, data } = config;
  return [method, url, Qs.stringify(params), Qs.stringify(data)].join("&");
}
```
addPendingRequest：用于把当前请求信息添加到pendingRequest对象中；
``` 
const pendingRequest = new Map();
function addPendingRequest(config) {
  const requestKey = generateReqKey(config);
  config.cancelToken = config.cancelToken || new axios.CancelToken((cancel) => {
    if (!pendingRequest.has(requestKey)) {
       pendingRequest.set(requestKey, cancel);
    }
  });
}
```
removePendingRequest：检查是否存在重复请求，若存在则取消已发的请求。
``` 
function removePendingRequest(config) {
  const requestKey = generateReqKey(config);
  if (pendingRequest.has(requestKey)) {
     const cancelToken = pendingRequest.get(requestKey);
     cancelToken(requestKey);
     pendingRequest.delete(requestKey);
  }
}
```
**设置请求拦截器**  
``` 
axios.interceptors.request.use(
  function (config) {
    removePendingRequest(config); // 检查是否存在重复请求，若存在则取消已发的请求
    addPendingRequest(config); // 把当前请求信息添加到pendingRequest对象中
    return config;
  },
  (error) => {
     return Promise.reject(error);
  }
);
```
**设置响应拦截器**  
``` 
axios.interceptors.response.use(
  (response) => {
     removePendingRequest(response.config); // 从pendingRequest对象中移除请求
     return response;
   },
   (error) => {
      removePendingRequest(error.config || {}); // 从pendingRequest对象中移除请求
      if (axios.isCancel(error)) {
        console.log("已取消的重复请求：" + error.message);
      } else {
        // 添加异常处理
      }
      return Promise.reject(error);
   }
);
```

## 适配器实现请求重试的方案
Axios 引入了适配器，使得它可以同时支持浏览器和 Node.js 环境。对于浏览器环境来说，它通过封装 XMLHttpRequest API 来发送 HTTP 请求，而对于 Node.js 环境来说，它通过封装 Node.js 内置的 http 和 https 模块来发送 HTTP 请求。  
 Axios 内置的 xhrAdapter 适配器定义在 lib/adapters/xhr.js 文件中：
 ``` 
 module.exports = function xhrAdapter(config) {
   return new Promise(function dispatchXhrRequest(resolve, reject) {
     var requestData = config.data;
     var requestHeaders = config.headers;
 
     var request = new XMLHttpRequest();
     // 省略大部分代码
     var fullPath = buildFullPath(config.baseURL, config.url);
     request.open(config.method.toUpperCase(), buildURL(fullPath, config.params, config.paramsSerializer), true);
     // Set the request timeout in MS
     request.timeout = config.timeout;
 
     // Listen for ready state
     request.onreadystatechange = function handleLoad() { ... }
 
     // Send the request
     request.send(requestData);
   });
 };
 ```
 xhrAdapter 适配器是一个函数对象，它接收一个 config 参数并返回一个 Promise 对象。而在 xhrAdapter 适配器内部，最终会使用 XMLHttpRequest API 来发送 HTTP 请求。
 **定义 retryAdapterEnhancer 函数**  
为了让用户能够更灵活地控制请求重试的功能，我们定义了一个 retryAdapterEnhancer 函数，该函数支持两个参数：  
- adapter：预增强的 Axios 适配器对象；
- options：缓存配置对象，该对象支持 2 个属性，分别用于配置不同的功能：
    - times：全局设置请求重试的次数；
    - delay：全局设置请求延迟的时间，单位是 ms
    
具体实现
``` 
function retryAdapterEnhancer(adapter, options) {
  const { times = 0, delay = 300 } = options;

  return async (config) => {
    const { retryTimes = times, retryDelay = delay } = config;
    let __retryCount = 0;
    const request = async () => {
      try {
        return await adapter(config);
      } catch (err) {
        // 判断是否进行重试
        if (!retryTimes || __retryCount >= retryTimes) {
          return Promise.reject(err);
        }
        __retryCount++; // 增加重试次数
        // 延时处理
        const delay = new Promise((resolve) => {
          setTimeout(() => {
            resolve();
          }, retryDelay);
         });
         // 重新发起请求
         return delay.then(() => {
           return request();
         });
        }
      };
   return request();
  };
}
```
**使用 retryAdapterEnhancer 函数**  
创建 Axios 对象并配置 adapter 选项
``` 
const http = axios.create({
  baseURL: "http://localhost:3000/",
  adapter: retryAdapterEnhancer(axios.defaults.adapter, {
    retryDelay: 1000,
  }),
});
```
使用 http 对象发送请求
``` 
// 请求失败不重试
function requestWithoutRetry() {
  http.get("/users");
}

// 请求失败重试
function requestWithRetry() {
  http.get("/users", { retryTimes: 2 });
}
```

原文: 
[Axios 如何实现请求重试？](https://juejin.cn/post/6973812686584807432?utm_source=gold_browser_extension)
