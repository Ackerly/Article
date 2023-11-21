# 什么叫JSON-RPC
JSON-RPC（JavaScript Object Notation Remote Procedure Call）是一种远程过程调用（RPC）协议，用于在网络上通过使用 JSON（JavaScript Object Notation）作为数据交换格式来进行通信。它允许在不同的计算机或不同的进程之间调用函数或方法，就像调用本地函数一样。  
JSON-RPC 的基本思想是通过 HTTP 或其他传输协议发送 JSON 格式的数据，以触发在远程服务器上执行特定操作的请求，然后服务器会处理这些请求并返回相应的 JSON 响应。这种方法使得分布式系统之间的通信变得更加简单，开发人员可以通过类似调用本地函数的方式来调用远程的功能，从而降低了跨网络通信的复杂性。  
JSON-RPC 的数据结构通常包括以下部分：  
- 方法名（method）：要调用的远程函数或方法的名称
- 参数（params）：传递给远程函数的参数，通常是一个数组或对象
- 请求 ID（id）：标识此次请求的唯一标识符，用于匹配请求与响应

基本的 JSON-RPC 请求示例：  
``` 
{
   "jsonrpc": "2.0",
   "method": "add",
   "params": [2, 3],
   "id": 1
 }
```
对应的 JSON-RPC 响应示例：  
``` 
 {
   "jsonrpc": "2.0",
   "result": 5,
   "id": 1
 }
```
JSON-RPC 的实现可以在多种编程语言中找到，许多编程框架和库都提供了对 JSON-RPC 的支持，使得开发人员可以更轻松地构建基于这种思想的分布式系统。  

## 例子
当使用前端请求应用 JSON-RPC 时，通常会使用 JavaScript 来构建请求并处理响应。以下是一个简单的示例，演示如何在前端使用 JSON-RPC 进行远程方法调用：  
假设我们有一个服务端提供了一个简单的加法函数，我们可以通过 JSON-RPC 来调用它。服务端期望接收两个参数，并返回它们的和。  
服务端方法示例（Node.js）：  
``` 
// server.js
 const http = require('http');
 const { parse } = require('querystring');

 const server = http.createServer((req, res) => {
   if (req.method === 'POST') {
     let body = '';

     req.on('data', chunk => {
       body += chunk.toString();
     });

     req.on('end', () => {
       const requestData = JSON.parse(body);
       if (requestData.method === 'add') {
         const result = requestData.params[0] + requestData.params[1];
         const response = {
           jsonrpc: '2.0',
           result: result,
           id: requestData.id
         };
         res.setHeader('Content-Type', 'application/json');
         res.end(JSON.stringify(response));
       }
     });
   }
 });

 server.listen(3000, () => {
   console.log('Server is listening on port 3000');
 });
```
在上述代码中，我们监听端口 3000，并根据收到的 JSON-RPC 请求进行加法运算，然后返回结果。  
前端请求示例：  
``` 
<!-- index.html -->
 <!DOCTYPE html>
 <html>
 <head>
   <title>JSON-RPC Frontend</title>
 </head>
 <body>
   <button id="calculateButton">Calculate</button>
   <p id="result"></p>

   <script>
     document.getElementById('calculateButton').addEventListener('click', async () => {
       const requestData = {
         jsonrpc: '2.0',
         method: 'add',
         params: [2, 3],
         id: 1
       };

       const response = await fetch('http://localhost:3000', {
         method: 'POST',
         body: JSON.stringify(requestData)
       });

       const responseData = await response.json();
       document.getElementById('result').textContent = `Result: ${responseData.result}`;
     });
   </script>
 </body>
 </html>
```
在上述前端代码中，我们在点击按钮时发送 JSON-RPC 请求到服务器，然后在页面上显示返回的结果。  
## JSON-RPC 与 RESTful 的区别
JSON-RPC 和 RESTful 是两种不同的远程通信协议和架构风格，用于在客户端和服务器之间进行通信。它们在设计理念、工作方式和使用场景上有一些区别。下面是它们的主要区别：  
**通信方式**  
- JSON-RPC: JSON-RPC 是一种远程过程调用（RPC）协议，它的主要目标是在客户端和服务器之间触发特定的操作，就像调用本地函数一样。通常，它使用 HTTP 或其他传输协议来发送 JSON 格式的请求和响应。
- RESTful: REST（Representational State Transfer）是一种基于资源的通信风格，它使用 HTTP 方法（GET、POST、PUT、DELETE 等）来对资源进行操作。RESTful 通信是状态无关的，客户端可以通过 HTTP 请求对资源进行增、删、改、查等操作。

**URL 设计**  
- JSON-RPC: JSON-RPC 请求通常不依赖于 URL 路径，而是通过请求中的方法名来指定要执行的操作。请求主体包含了方法名和参数。
- RESTful: RESTful API 的 URL 设计通常使用资源路径来表示资源和操作。不同的 HTTP 方法（如 GET、POST、PUT、DELETE）用于执行不同的操作，而 URL 路径用于指定资源

**状态和无状态**  
- JSON-RPC: JSON-RPC 通常是有状态的，因为每个请求都可以包含方法调用和参数。服务器可以维护一些状态信息以处理请求。它不依赖于 HTTP 状态码来表示操作结果。  
- RESTful: RESTful 通信是无状态的，每个请求应该包含所有必要的信息，服务器不应该依赖于之前的请求状态。HTTP 状态码用于表示操作结果（例如，200 表示成功，404 表示未找到，500 表示服务器错误等）

**格式和约定**  
- JSON-RPC: JSON-RPC 请求和响应都使用 JSON 格式，但它具有一组规范的方法调用和错误处理约定
- RESTful: RESTful 通信可以使用不同的数据格式，包括 JSON、XML 等。它没有强制性的约定，通常需要在 API 文档中明确指定资源的结构和操作方式

**灵活性**  
- JSON-RPC: JSON-RPC 通常更紧凑，适用于执行特定的操作，对于需要定制化操作的情况较为合适
- RESTful: RESTful 通常更灵活，适用于处理不同类型的资源和操作，可以更好地与 HTTP 协议集成

选择使用 JSON-RPC 还是 RESTful 取决于你的应用需求。如果你主要关注方法的调用和返回结果，而且希望在通信中更少地处理状态信息，JSON-RPC 可能更适合。如果你需要构建更复杂的分布式系统，且资源的状态管理是一个重要考虑因素，那么 RESTful 可能更合适。  
RESTful 通常更适合公开的、基于资源的 API，而 JSON-RPC 更适合需要精确控制和操作的 RPC 场景。

原文:  
[什么叫JSON-RPC?](https://mp.weixin.qq.com/s/WYpLRpafXEsrL4bXNRAZXA)
