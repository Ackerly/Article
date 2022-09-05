# NodeJS落地WebSocket实践
## 网络协议进化
HTTP 协议是前端最熟悉的网络通信协议。我们通常的打开网页，请求接口，都属于 HTTP 请求。  
HTTP 请求的特点是：请求-> 响应。客户端发起请求，服务端收到请求后进行响应，一次请求就完成了。也就是说，HTTP 请求必须由客户端发起，服务端才能被动响应。
除此之外，发起 HTTP 请求之前，还需要通过三次握手建立 TCP 连接。HTTP/1.0 的特点是，每通信一次，都要经历 “三步走” 的过程 —— TCP 连接 -> HTTP 通信 -> 断开 TCP 连接。  
这样的每一次请求都是独立的，一次请求完成连接就会断开。  
HTTP1.1 对请求过程做了优化。TCP 连接建立之后，可以进行多次 HTTP 通信，等到一个时间段无 HTTP 请求发起 TCP 才会断开连接，这就是 HTTP/1.1 带来的长连接技术。  
即便如此，通信方式依然是客户端发起，服务端响应，这个根本逻辑不会变。随着应用交互的复杂，有一些场景是必须要实时获取服务端消息的。比如即时聊天，比如消息推送，用户并不会主动发起请求，但是当服务器有了新消息，客户端需要立刻知道并且反馈给用户。  
HTTP 不支持服务端主动推送，但是这些场景又急需解决方案，于是早期出现了轮询（polling）。轮询是客户端定时向服务器发起请求，检测服务端是否有更新，如果有则返回新数据。  
轮询方式虽然简单粗暴，但很显然有两个弊端：  
1. 请求消耗太大。客户端不断请求，浪费流量和服务器资源，给服务器造成压力。
2. 不能保证及时。客户端需要平衡及时性和性能，请求间隔必然不能太小，因此会有延迟。

随着 HTML5 推出 WebSocket，即时通讯场景终于迎来了根本解决方案。WebSocket 是全双工通信协议，当客户端与服务端建立连接之后，双方可以互相发送数据，这样的话就不需要客户端通过轮询这种低效的方式获取数据，服务端有新消息直接推送给客户端即可。  
传统 HTTP 连接方式如下：  
``` 
## 普通连接
http://localhost:80/test
## 安全连接
https://localhost:80/test
```
WebSocket 是另一种协议，连接方式如下：  
``` 
## 普通连接
ws://localhost:80/test
## 安全连接
wss://localhost:80/test
```
但是 WebSocket 也不是完全脱离 HTTP 的，若要建立 WebSocket 连接，则必须要客户端主动发起一个建立连接的 HTTP 请求，连接成功之后客户端与服务端才能进行双向通信。  

## Socket.IO
Socket.IO 是目前 Node.js 在生产环境中开发 WebSocket 应用最好的选择。它功能强大，高性能，低延迟，并且可以一步集成到 express 框架中。  
Socket.IO 并不是一个纯粹的 WebSocket 框架。它是将 Websocket 和轮询机制以及其它的实时通信方式封装成了通用的接口，以实现更高效的双向通信。  
既然 Socket.IO 在 WebSocket 的基础上做了那么多的优化，并且非常成熟，那为什么还要搭一个原生 WebSocket 服务？  
Socket.IO 不能通过原生的 ws 协议连接。比如你在浏览器试图通过 ws://localhost:8080/test-socket 这种方式连接 Socket.IO 服务，是连接不上的。因为 Socket.IO 的服务端必须通过 Socket.IO 的客户端连接，不支持默认的 WebSocket 方式连接。  
## ws 模块实现
ws 是 Node.js 下一个简单快速，并且定制程度极高的 WebSocket 实现方案，同时包含了服务端和客户端。  
用 ws 搭建起来的服务端，浏览器可以通过原生 WebSocket 构造函数直接连接，非常便捷。ws 客户端则是模拟浏览器的 WebSocket 构造函数，用于连接其他 WebSocket 服务器进行通信。  
注意：ws 只能在 Node.js 环境中使用，浏览器中不可用，浏览器请直接使用原生 WebSocket 构造函数。  
第一步，安装 ws：
``` 
$ npm install ws
```
安装好后，我们先搭建一个 ws 服务端。
**服务端**  
搭建 websocket 服务器需要用 WebSocketServer 构造函数。  
``` 
const { WebSocketServer } = require('ws')
const wss = new WebSocketServer({
  port: 8080
})
wss.on('connection', (ws, req) => {
  console.log('客户端已连接：', req.socket.remoteAddress)
  ws.on('message', data => {
    console.log('收到客户端发送的消息：', data)
  })
  ws.send('我是服务端') // 向当前客户端发送消息
})
```
运行：
``` 
node ws-server.js
```
这样一个监听 8080 端口的 WebSocket 服务器就已经跑起来了
**客户端**  
建好了 WebSocket 服务器，现在在前端连接并监听消息：
``` 
var ws = new WebSocket('ws://localhost:8080')

ws.onopen = function(mevt) {
  console.log('客户端已连接')
}
ws.onmessage = function(mevt) {
  console.log('客户端收到消息: ' + evt.data)
  ws.close()
}
ws.onclose = function(mevt) {
  console.log('连接关闭')
}
```
浏览器连接成功后，收到服务端主动推送过来的消息，然后浏览器可以主动关闭连接。
Node.js 环境下我们看 ws 模块如何发起连接：  
``` 
Node.js 环境下我们看 ws 模块如何发起连接：
```
代码与浏览器的逻辑一模一样，只是写法稍有些不同。另外浏览器端监听 message 事件的回调函数，参数是一个 MessageEvent 的实例对象，服务端发来的实际数据需要通过 mevt.data 获取。在 ws 客户端，这个参数就是服务端的实际数据，直接获取即可。  
## Express 集成
ws 模块一般不会单独使用，更优的方案是集成到现有的框架中。集成到 Express 框架的优点是，我们不需要单独监听一个端口，使用框架启动的端口即可，并且我们还可以指定访问到某个路由，才发起 WebSocket 连接。  
express-ws 模块已经做好了大部分的集成工作，只要安装，然后在入口文件引入：  
``` 
var expressWs = require('express-ws')(app)
```
和 Express 的 Router 一样，express-ws 也支持注册全局路由和局部路由。  
先看全局路由，通过 [host]/test-ws 连接：  
``` 
app.ws('/test-ws', (ws, req) => {
  ws.on('message', msg => {
    ws.send(msg)
  })
})
```
局部路由则是注册在一个路由组下面的子路由。配置一个名为 websocket 的路由组并指向 websocket.js 文件，代码如下：
``` 
// websocket.js
var router = express.Router()

router.ws('/test-ws', (ws, req) => {
  ws.on('message', msg => {
    ws.send(msg)
  })
})

module.exports = router
```
连接 [host]/websocket/test-ws 就可以访问到这个子路由。  
路由组的作用是定义一个 websocket 连接组，不同需求连接这个组下的不同子路由。比如可以将 单聊 和 群聊 设置为两个子路由，分别处理各自的连接通信逻辑。  
完整代码如下：  
``` 
var express = require('express')
var app = express()
var wsServer = require('express-ws')(app)
var webSocket = require('./websocket.js')

app.ws('/test-ws', (ws, req) => {
  ws.on('message', msg => {
    ws.send(msg)
  })
})

app.use('/websocket', webSocket)

app.listen(3000)
```
实际开发中获取常用信息的小方法：  
``` 
// 客户端的IP地址
req.socket.remoteAddress
// 连接参数
req.query
```
## WebSocket 实例
WebSocket 实例是指客户端连接对象，以及服务端连接的第一个参数。  
``` 
var ws = new WebSocket('ws://localhost:8080')
app.ws('/test-ws', (ws, req) => {}
```
ws 就是 WebSocket 实例，表示建立的连接。  
**浏览器**  
浏览器的 ws 对象中包含的信息如下：  
``` 
{
  binaryType: 'blob'
  bufferedAmount: 0
  extensions: ''
  onclose: null
  onerror: null
  onmessage: null
  onopen: null
  protocol: ''
  readyState: 3
  url: 'ws://localhost:8080/'
}
```
四个监听属性，用于定义函数：  
- onopen：连接建立后的函数
- onmessage：收到服务端推送消息的函数
- onclose：连接关闭的函数
- onerror：连接异常的函数

最常用的是 onmessage 属性，赋值为一个函数来监听服务端消息：  
``` 
ws.onmessage = mevt => {
  console.log('消息：', mevt.data)
}
```
有一个关键属性是 readyState，表示连接状态，值为一个数字。并且每个值都可以用常量表示，对应关系和含义如下：  
- 0: 常量 WebSocket.CONNECTING，表示正在连接
- 1: 常量 WebSocket.OPEN，表示已连接
- 2: 常量 WebSocket.CLOSING，表示正在关闭
- 3: 常量 WebSocket.CLOSED，表示已关闭

最重要的还有 send 方法用于发送信息，向服务端发送数据：  
``` 
ws.send('要发送的信息')
```
**服务端**  
服务端的 ws 对象表示当前发起连接的一个客户端，基本属性与浏览器大致相同。不过因为服务端是 Node.js 实现，因此会有更丰富的支持。比如下面两种监听事件的写法效果是一样的：
``` 
// Node.js 环境
ws.onmessage = str => {
  console.log('消息：', str)
}
ws.on('message', str => {
  console.log('消息：', mevt.data)
})
```
## 消息广播
WebSocket 服务器不会只有一个客户端连接，消息广播的意思就是把信息发给所有已连接的客户端，像一个大喇叭一样，所有人都听得到，经典场景就是热点推送。  
广播之前，就必须要解决一个问题，如何获取当前已连接（在线）的客户端？ws 模块提供了快捷的获取方法：  
``` 
var wss = new WebSocketServer({ port: 8080 })
// 获取所有已连接客户端
wss.clients
```
express-ws 获取：  
``` 
var wsServer = expressWebSocket(app)
var wss = wsServer.getWss()
// 获取所有已连接客户端
wss.clients
```
wss.clients 就是由所有在线客户端的 WebSocket 实例组成的一个 Set 集合。  
获取当前在线客户端的数量：  
``` 
wss.clients.size
```
简单粗暴的实现广播：  
``` 
wss.clients.forEach(client => {
  if (client.readyState === 1) {
    client.send('广播数据')
  }
})
```
这是非常简单，基础的实现方式。但是如果此刻在线客户有 10000 个，那么这个循环多半会卡死吧。因此才会有像 socket.io 这样的库，对基础功能做了大量优化和封装，提高并发性能。  
上面的广播属于全局广播，就是将消息发给所有人。然而还有另一种场景，比如一个 5 人的群聊小组聊天，这时的广播只是给这 5 人小团体发消息，因此这也叫 局部广播。  
局部广播的实现要复杂一些，一般会揉合具体的业务场景。需要在客户端连接时，对客户端数据做持久化处理了。比如用 Redis 存储在线客户端的状态和数据，这样检索分类更快，效率更高。  
局部广播实现，那一对一私聊就更容易了。找到两个客户端对应的 WebSocket 实例互发消息就行。  
## 安全与认证
建好的 WebSocket 服务器，默认任何客户端都可以连接，这在生产环境肯定是不行的。我们要对 WebSocket 服务器做安全保障，主要是从两个方面入手：  
- Token 连接认证
- wss 支持

**Token 连接认证**  
HTTP 请求接口我们一般会做 JWT 认证，在请求头中带一个指定 Header，将一个 token 字符串传过去，后端会拿这个 token 做校验，校验失败则返回 401 错误阻止请求。  
WebSocket 建立连接的第一步是客户端发起一个 HTTP 的连接请求，那么在这个 HTTP 请求上做验证，如果验证失败，则中断 WebSocket 的连接创建，不就可以了？  
因为要在 HTTP 层做校验，所以用 http 模块创建服务器，关掉 WebSocket 服务的端口。  
``` 
var server = http.createServer()
var wss = new WebSocketServer({ noServer: true })

server.listen(8080)
```
当客户端通过 ws:// 连接服务端时，服务端会进行协议升级，也就是将 http 协议升级成 websocket 协议，此时会触发 upgrade 事件：  
``` 
server.on('upgrade', (request, socket) => {
  // 用 request 获取参数做验证
  // 1. 验证不通过判断
  if ('验证失败') {
    socket.write('HTTP/1.1 401 Unauthorized\r\n\r\n')
    socket.destroy()
    return
  }
  // 2. 验证通过，继续建立连接
  wss.handleUpgrade(request, socket, _, ws => {
    wss.emit('connection', ws, request)
  })
})

// 3. 监听连接
wss.on('connection', (ws, request) => {
  console.log('客户端已连接')
  ws.send('服务端信息')
})
```
这样服务端认证添加完毕，具体的认证方法结合客户端的传参方式来定。  
WebSocket 客户端连接不支持自定义 Header，因此不能用 JWT 的方案，可用方案有两种：  
- Basic Auth
- Quary 传参

Basic Auth 认证简单说就是账号+密码认证，而且账号密码是带在 URL 里的。  
设我有账号是 ruims，密码是 123456，那么客户端连接是这样：  
``` 
var ws = new WebSocket('ws://ruims:123456@localhost:8080')
```
服务端就会收到这样一个请求头：  
``` 
wss.on('connection', (ws, req) => {
  if(req.headers['authorization']) {
    let auth = req.headers['authorization']
    console.log(auth)
    // 打印的值：Basic cnVpbXM6MTIzNDU2
  }
}
```
cnVpbXM6MTIzNDU2 就是 ruims:123456 的 base64 编码，服务端可以获取到这个编码来做认证。  
Quary 传参比较简单，就是普通的 URL 传参，可以带一个短一点的加密字符串过去，服务端获取到该字符串然后做认证：  
``` 
var ws = new WebSocket('ws://localhost:8080?token=cnVpbXM6MTIzNDU2')
```
服务端获取参数：  
``` 
wss.on('connection', (ws, req) => {
  console.log(req.query.token)
}
```
**wss 支持**  
WebSocket 客户端使用 ws:// 协议连接，那 wss 是什么意思？和 https 原理一摸一样，https 表示安全的 http 协议，组成是 HTTP + SSL。 wss 则表示安全的 ws 协议，组成是 WS + SSL  
为什么一定要用 wss 呢？除了安全性，还有一个关键原因是：如果你的 web 应用是 https 协议，你在当前应用中使用 WebSocket 就必须是 wss 协议，否则浏览器拒绝连接。  
配置 wss 直接在 https 配置中加一个 location 即可，nginx 配置：  
``` 
location /websocket {
  proxy_pass http://127.0.0.1:8080;
  proxy_redirect off;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection upgrade;
}
```
客户端连接就变成了这样：
``` 
var ws = new WebSocket('wss://[host]/websocket')
```
## BFF 应用
BFF全称是 Backend For Frontend，意思是为前端服务的后端，在实际应用架构中属于前端和后端的一个 中间层。  
现在后端的主流架构是微服务，微服务情况下 API 会划分的非常细，商品服务就是商品服务，通知服务就是通知服务。当你想在商品上架时给用户发一个通知，可能至少需要调两个接口。  
这样的话对前端其实是不友好的，于是后来出现了 BFF 中间层，相当于一个后端请求的中间代理站，前端可以直接请求 BFF 的接口，然后 BFF 再向后端接口请求，将需要的数据组合起来，一次返回前端。  
WebSocket 的知识在 BFF 层如何应用呢？
应用场景至少有 4 个：  
1. 查看当前在线人数，在线用户信息
2. 登录新设备，其他设备退出登录
3. 检测网络连接/断开
4. 站内消息，小圆点提示

这些功能以前是在后端实现的，并且会与其他业务功能耦合。现在有了 BFF，那么 WebSocket 完全可以在这一层实现，让后端可以专注核心数据逻辑。  
原文:  
[前端架构师破局技能，NodeJS 落地 WebSocket 实践](https://mp.weixin.qq.com/s/05OlojnO11cnafn4o6qCmQ)