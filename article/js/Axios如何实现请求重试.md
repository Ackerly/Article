# Axios 如何实现请求重试
## 拦截器实现请求重试的方案
Axios 提供了 请求拦截器和响应拦截器 来分别处理请求和响应，它们的作用如下：
- 请求拦截器：该类拦截器的作用是在请求发送前统一执行某些操作，比如在请求头中添加 token 字段。
- 响应拦截器：该类拦截器的作用是在接收到服务器响应后统一执行某些操作，比如发现响应状态码为 401 时，自动跳转到登录页。

 Axios 中设置拦截器很简单，通过 axios.interceptors.request 和 axios.interceptors.response 对象提供的 use 方法，就可以分别设置请求拦截器和响应拦截器：  
 ``` 
 export interface AxiosInstance {
   interceptors: {
     request: AxiosInterceptorManager<AxiosRequestConfig>;
     response: AxiosInterceptorManager<AxiosResponse>;
   };
 }
 
 export interface AxiosInterceptorManager<V> {
   use(onFulfilled?: (value: V) => V | Promise<V>, 
     onRejected?: (error: any) => any): number;
   eject(id: number): void;
 }
 ```
对于请求重试的功能来说，希望让用户不仅能够设置重试次数，而且可以设置重试延时时间。当请求失败的时候，若该请求的配置对象配置了重试次数，而 Axios 就会重新发起请求进行重试操作。为了能够全局进行请求重试，接下来我们在响应拦截器上来实现请求重试功能，具体代码如下所示：
``` 
axios.interceptors.response.use(null, (err) => {
  let config = err.config;
  if (!config || !config.retryTimes) return Promise.reject(err);
  const { __retryCount = 0, retryDelay = 300, retryTimes } = config;
  // 在请求对象上设置重试次数
  config.__retryCount = __retryCount;
  // 判断是否超过了重试次数
  if (__retryCount >= retryTimes) {
    return Promise.reject(err);
  }
  // 增加重试次数
  config.__retryCount++;
  // 延时处理
  const delay = new Promise((resolve) => {
    setTimeout(() => {
      resolve();
    }, retryDelay);
  });
  // 重新发起请求
  return delay.then(function () {
    return axios(config);
  });
});
```


参考:  
[Axios 如何实现请求重试？](https://juejin.cn/post/6973812686584807432?utm_source=gold_browser_extension)
