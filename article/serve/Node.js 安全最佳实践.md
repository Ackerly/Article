# Node.js 安全最佳实践
**计时攻击**  
计时攻击可能会让攻击者获取到一些潜在的敏感信息，例如，测量应用程序响应请求所需的时间。这种攻击并不是特定于 Node.js 的，几乎可以针对所有运行时。  
我们的程序代码中可能会存在一些时间段敏感的操作，比如我们需要校验一个用户的密码是否正确。  
我们可能会从数据库检索出来的用户信息中比较密码。对于相同的长度值，使用内置字符串比较可能需要更长的时间。这种比较在以可接受的数量运行时会增加请求的响应时间。通过比较请求响应时间，攻击者可以在大量请求中猜测密码的长度和值。  
**缓解措施**  
_crypto API_  
crypto API 提供了一个 timingSafeEqual 函数，当你需要进行比较的值比较敏感时，它可一采用恒定时间算法进行比较。  
对于密码比较，你可以使用 crypto 模块上提供的 scrypt 。  
避免在可变时间操作中使用密钥，包括密钥分支，并且当攻击者可能位于同一基础设施（例如同一台云机器）上时，使用密钥作为内存索引。用 JavaScript 编写恒定时间的代码还是很困难的，对于加密应用程序，推荐使用内置的加密 API 或 WebAssembly。  

## 恶意第三方模块
目前，在 Node.js 中，任何包都可以访问网络、文件系统，他们可以将任何数据发送到任何地方。  
所有运行在 Node.js 进程中的代码都能够通过使用 eval() 加载和运行额外的任意代码。所有具有文件系统写访问权限的代码都可以通过写入加载的新文件或现有文件来实现相同的目的。  
Node.js 有一个实验性的 策略机制来声明加载的资源是否是不受信任的。  
我们应该确保使用通用工作流或 npm script 固定依赖版本、自动检查漏洞。在安装依赖包之前，请确保这个还是在维护的并包含你期望的所有内容。注意，Github 源代码并不总是与发布的包相同，最好在 node_modules 中验证一下。  
**供应链攻击**  
供应链攻击一般指控制上游包的攻击者可以发布包含恶意代码的新版本。如果我们的 Node.js 应用程序依赖于这个包，而没有严格确定哪个版本可以安全使用，则该包可以自动更新到最新的恶意版本，从而危及应用程序。  
供应链攻击攻击最近在 Node.js 的依赖生态中频发发生，比如前段时间的 node-ipc，针对俄罗斯和白俄罗斯 IP，会尝试覆盖当前目录、父目录和根目录的所有文件，把所有内容替换成 ❤。  
这主要还是因为 Node.js 生态对依赖项的规范过于松懈了，比如允许不需要的更新，我们可能悄无声息的在某一次上线中为我们的程序带来了巨大的危机。  
虽然我们可以在 package.json 中指定依赖项确切的版本号或范围，但这只能保证直接依赖的固定，我们仍然无法保障间接依赖的不确定性更新。  

**缓解措施**  
- 防止 npm 使用 ——ignore-scripts 执行任意脚本
- 可以使用 npm config set ignore-scripts true 全局禁用它
- 将 lock 文件将依赖版本固定到特定的不可变版本，而不是一个范围（当然后续要手动定期更新）
- 将 npm audit 引入 CI 流程，自动检查漏洞
诸如 Socket 之类的工具可以用来分析带有静态分析的包，以发现诸如网络或文件系统访问之类的风险行为
- 使用 npm ci 代替 npm install，这将强制执行 lockfile，避免它与 package.json 文件之间的不一致会导致错误
- 仔细检查 package.json 文件中依赖项名称中的错误 / 错别字。

## 内存访问冲突
基于内存或基于堆的攻击取决于代码中的内存管理错误和可利用的内存分配器的组合。与所有运行时一样，如果项目运行在共享的机器上，Node.js 很容易受到这些攻击。使用 secure heap 有助于防止由于指针溢出和不足而导致敏感信息泄漏。  

**缓解措施**  
- 根据程序的时机情况使用 ——secure-heap=n ，其中 n 是分配的最大字节大小；
- 不要在共享机器上运行比较重要的应用程序

**猴子修补**  
猴子补丁指的是为了改变现有的行为在运行时修改属性，比如：  
``` 
// eslint-disable-next-line no-extend-native
 Array.prototype.push = function (item) {
   // overriding the global [].push
 };
```

**缓解措施**  
--frozen-intrinsics 标志可以启用实验性的 冻结内置函数，启用后所有内置的 JavaScript 对象和函数都被递归冻结。下面的代码片段不会覆盖 Array.prototype.push 的默认行为  
``` 
// eslint-disable-next-line no-extend-native
 Array.prototype.push = function (item) {
   // overriding the global [].push
 };

 // Uncaught:
 // TypeError <Object <Object <[Object: null prototype] {}>>>:
 // Cannot assign to read only property 'push' of object ''
```
但需要注意的是，你仍然可以使用 globalThis 定义新的全局变量并替换现有的全局变量：  
``` 
 > globalThis.foo = 3; foo; // you can still define new globals
 3
 > globalThis.Array = 4; Array; // However, you can also replace existing globals
 4
```
Object.freeze(globalThis) 可用于保证不会替换任何全局变量。  

## 原型污染
原型污染是指通过滥用 _proto_、 constructor、prototype 和其他从内置原型继承的其他属性来修改或将属性注入 JavaScript 语言项的攻击，这是一种继承自 JavaScript 语言的潜在漏洞。  
比如下面的代码，一个外部传入的数据可能会影响到我们整个 Node.js 服务的 Object 对象的默认行为：  
``` 
const a = {"a": 1, "b": 2};
 const data = JSON.parse('{"__proto__": { "polluted": true}}');

 const c = Object.assign({}, a, data);
 console.log(c.polluted); // true

 // Potential DoS
 const data2 = JSON.parse('{"__proto__": null}');
 const d = Object.assign(a, data2);
 d.hasOwnProperty('b'); // Uncaught TypeError: d.hasOwnProperty is not a function
```
**缓解措施**  
- 避免不安全的递归合并https://gist.github.com/DaniAkash/b3d7159fddcff0a9ee035bd10e34b277#file-unsafe-merge-js
- 为外部 / 不受信任的请求实施 JSON 模式验证
- 使用 Object.create(null) 创建没有原型的对象
- 使用 Object.freeze(MyObject.prototype) 冻结原型
- 使用 --disable-proto 标志禁用 Object.prototype.__proto__ 属性
- 检查属性是否直接存在于对象上，而不是从使用 Object.hasOwn(obj, keyFromObj)
- 避免使用 Object.prototype 中的方法

## 路径引入漏洞
Node.js 按照模块解析算法加载模块。因此，它默认假定请求（require）模块的目录是受信任的。  
也就是说，这意味着以下应用程序行为是预期的。假设有以下目录结构：  
``` 
app/
   server.js
   auth.js
   auth
```
如果 server.js 使用 require('./auth') ，它将遵循模块解析算法并加载 auth 而不是 auth.js。  

**缓解措施**  
具有完整性检查的实验性策略机制可以避免上述威胁。对于上述目录，可以使用以下 policy.json  
``` 
{
   "resources": {
     "./app/auth.js": {
       "integrity": "sha256-iuGZ6SFVFpMuHUcJciQTIKpIyaQVigMZlvg9Lx66HV8="
     },
     "./app/server.js": {
       "dependencies": {
         "./auth" : "./app/auth.js"
       },
       "integrity": "sha256-NPtLCQ0ntPPWgfVEgX46ryTNpdvTWdQPoZO3kHo0bKI="
     }
   }
 }
```
当引入 auth 模块时，系统将验证完整性，如果与预期的不匹配则抛出错误。  
``` 
» node --experimental-policy=policy.json app/server.js
 node:internal/policy/sri:65
       throw new ERR_SRI_PARSE(str, str[prevIndex], prevIndex);
       ^

 SyntaxError [ERR_SRI_PARSE]: Subresource Integrity string "sha256-iuGZ6SFVFpMuHUcJciQTIKpIyaQVigMZlvg9Lx66HV8=%" had an unexpected "%" at position 51
     at new NodeError (node:internal/errors:393:5)
     at Object.parse (node:internal/policy/sri:65:13)
     at processEntry (node:internal/policy/manifest:581:38)
     at Manifest.assertIntegrity (node:internal/policy/manifest:588:32)
     at Module._compile (node:internal/modules/cjs/loader:1119:21)
     at Module._extensions..js (node:internal/modules/cjs/loader:1213:10)
     at Module.load (node:internal/modules/cjs/loader:1037:32)
     at Module._load (node:internal/modules/cjs/loader:878:12)
     at Module.require (node:internal/modules/cjs/loader:1061:19)
     at require (node:internal/modules/cjs/helpers:99:18) {
   code: 'ERR_SRI_PARSE'
 }
```
注意：始终建议使用 --policy-integrity 来避免策略突变。  

## HTTP 请求走私攻击（HTTP Request Smuggling）
这是一种涉及两个 HTTP 服务器（通常是代理服务和 Node.js 应用服务）的攻击。客户端发送 HTTP 请求，这个请求首先通过前端服务器（代理），然后重定向到后端服务器（应用程序）。当前端和后端对模糊的 HTTP 请求的解释不同时，攻击者就有可能发送前端看不到但后端会看到的恶意消息，有效地通过代理服务器进行了 “走私” 。  
通俗地理解就是：攻击者发送一个语句模糊的请求，就有可能被解析为两个不同的 HTTP 请求，第二请求可能会 “逃过” 正常的安全设备的检测，使攻击者可以绕过安全控制，未经授权访问敏感数据并直接危害其他应用程序用户。  
由于这种攻击产生的根本原因是 Node.js 与另一个 HTTP 服务器解释 HTTP 请求的方式不同，我们可以认为它是 Node.js、前端服务器两者的漏洞 。如果 Node.js 解释请求的方式是符合 HTTP 规范的，那么它就不被认为是 Node.js 中的漏洞。  

**缓解措施**
- 在创建 HTTP 服务器时，不要使用 insecureHTTPParser 选项；
- 前端服务器的配置要尽量规范化，避免歧义请求；
- 持续监控 Node.js 和前端服务器中是否存在新的 HTTP 请求走私漏洞；
- 使用 HTTP/2 端到端并尽可能禁用 HTTP 降级。

## HTTP 服务拒绝访问
由于我们错误的代码逻辑或者错误的配置可能会导致 HTTP 服务无法访问，参考下面的代码：  
``` 
const net = require('net');

 const server = net.createServer(function(socket) {
   // socket.on('error', console.error) // this prevents the server to crash
   socket.write('Echo server\r\n');
   socket.pipe(socket);
 });

 server.listen(5000, '0.0.0.0');
```
我们的 WebServer 没有正确的处理 Socket 错误，当发送的请求量过大时，我们的服务就会崩溃。  
**缓解措施**  
- 使用反向代理接收请求并将请求转发到 Node.js 应用程序。反向代理可以提供缓存、负载平衡、IP 黑名单等功能，从而降低 DoS 攻击生效的可能性；
- 正确配置服务器超时，以便可以放弃空闲或速度太慢的连接。例如 http.Server 中的 headersTimeout、requestTimeout timeout keepAliveTimeout；
- 限制每台主机打开的 Socket 总数，可以参考 http 中的 agent.maxSockets、agent.maxTotalSockets、agent.maxFreeSockets、server.maxRequestsPerSocket

## DNS 重绑定
这是一种针对在使用 --inspect 启用调试检查器的情况下运行的 Node.js 应用程序的攻击。  
由于在 Web 浏览器中打开的网站可以发出 WebSocket 和 HTTP 请求，它们可以针对本地运行的调试检查器。这通常会被现代浏览器实施的同源策略所阻止，这个策略会禁止脚本访问来自不同来源的资源（意味着恶意网站无法读取从本地 IP 地址请求的数据）。  
但是，通过 DNS 重绑定，攻击者可以暂时控制其请求的来源，使它们看起来像是来自本地 IP 地址。这是通过控制网站和用于解析其 IP 地址的 DNS 服务器来完成的。详细可以了解：https://en.wikipedia.org/wiki/DNS_rebinding  
**缓解措施**  
通过附加一个 process.on(‘SIGUSR1’, …) 侦听器来禁用 SIGUSR1 信号上的检查器
不要在生产环境中运行 inspector 协议  

## NPM 敏感信息泄漏
在包发布期间，包含在当前目录中的所有文件和文件夹都会被推送到 npm 注册表中，如果我们的开发目录中包含了一些敏感信息，它们都会被泄露出去。  
我们可以通过用 .npmignore 和 .gitignore 定义一个阻止列表或者在 package.json 中定义一个 allowlist 来控制这种行为  

**缓解措施**  
- 使用 npm publish——dry-run 列出所有要发布的文件，确保在发布包之前进行检查；
- 创建和维护诸如 .gitignore 和 .npmignore 这样的忽略文件也很重要。在这些文件中，你可以指定不应该发布哪些文件 / 文件夹；

原文:  
[Node.js 安全最佳实践](https://mp.weixin.qq.com/s/aWOuZeYUriznoROrairrDg)
