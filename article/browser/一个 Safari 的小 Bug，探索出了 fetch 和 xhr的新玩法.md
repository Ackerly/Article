# 一个 Safari 的小 Bug，探索出了 fetch 和 xhr的新玩法
> 打开 A 页面之后，就自动跳转到登录页面了，但是打开其他页面是正常的

在 Chrome 中会正常展示数据，在 Safari 中会提示 request error。  

**现象描述**  
因为有一个接口由于请求地址不对，接口返回了 301，需要重定向到新的接口：  
- 前端请求的地址：/api/user/list
- 后端需要的地址：/api/user/list-new

在 Safari 中当浏览器收到 3XX 的重定向状态码后，会自动对新的地址发起请求（也就是响应体中 Location 的地址）  
然而 Safari 浏览器在自动发起新的请求时，没有携带自定义的 Authorization 请求头，所以导致接口鉴权失败，返回了 401（Unauthorized）
前端在收到接口响应后，由于响应体里面也返回了未登录的业务 code，就自动跳转到了登录页面  
Safari 在发起重定向请求时，虽然没有带上 Authorization 请求头，但是会带上 cookie，这也说明了为什么在改造为 JWT 之前，Safari 能正常使用的原因  
Chrome 中进行了相同的测试，发现 Chrome 在发起重定向请求时，会携带 Authorization 请求头，所以能够正常使用  

**猜测可能**  
> 尽管标准要求浏览器在收到该响应并进行重定向时不应该修改 http method 和 body，但是有一些浏览器可能会有问题。所以最好是在应对 GET 或 HEAD 方法时使用 301，其他情况使用 308 来替代 301。
--- MDN：301 Moved Permanently

通过 Proxyman 将请求的响应码改为 308 后，发现 Safari 依旧不会携带 Authorization 请求头  

**解决方案**   
_目前 Safari 15.4（iOS 15.4, macOS 12.3） 已经修复了此问题，所以升级版本即可解决_  
_存储到 cookie_  
  通过把 token 放到 cookie 中存储来解决这个问题的，因为 Safari 重定向时，虽然不会携带 Authorization，但是会把 cookie 带上

_手动处理_  
XMLHttpRequest
按照规定 XMLHttpRequest 在收到重定向请求时，会自动对新 URL 发起请求，并且规范中没有提供阻止重定向的方法  
但是可以通过 responseURL 属性获取到重定向的 URL：  
``` 
xhr.onreadystatechange = function () {
   if (this.readyState === this.DONE) {
     console.log("responseURL", this.responseURL);
   }
 };
```
responseURL 这个属性有 2 种取值逻辑，当本次请求：  
- 没有触发重定向时，打印的是本次请求的 URL。
- 触发重定向时，打印的是重定向的 URL

但是规范中提到，他有可能是空字符串  
如果你一定要终止重定向请求，那么可以通过 responseURL 和原始的请求 URL 进行对比，如果不同，则表明存在重定向，但是不推荐使用这种逻辑判断，因为这不是官方标准。另外，这里的 status 取到的是重定向后的值，所以不能用它对比。  
通过这种逻辑进行重定向判断的，需要注意以下两点：  
- 通过 abort 终止重定向请求后，需要在 onload 事件中做一层判断，因为 Safari 在请求终止后，还是会进入到 onload 事件中。可通过 status 进行判断，终止之后的请求，status 的值为 0。
- 通过 abort 终止重定向请求后，浏览器还是会对重定向的新 URL 发起请求，服务器也会正常处理并响应，所以需要注意此请求是否有「副作用」

Hack 思路  
- 通过 Fetch 阻止浏览器自动重定向。
- 通过 XMLHttpRequest 获取重定向的 URL。
- 自动对重定向的 URL 发起请求

使用上述方法，虽然在 Safari 中可完美运行，但是控制台还是会打印 401 的错误  
另外一个需要注意的点是：最好根据浏览器做一层判断，如果是 Safari，则将 redirect 设置为 manual，否则不进行处理。这样可以避免 Chrome 发起过多的无用请求（Chrome 总共会发出 5 个请求）  

参考：  
[一个 Safari 的小 Bug，探索出了 fetch 和 xhr的新玩法](https://mp.weixin.qq.com/s/jqRxNAI5C2NdxVX-pthFpg)
