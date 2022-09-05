# 如何使用 HTTPS 进行本地开发
**使用 mkcert 为本地网站开启 HTTPS（推荐）**  
要为本地开发网站开启 HTTPS 并访问 https://localhost 或 https://mysite.example（自定义主机名），您需要 TLS 证书。但并非任何证书都会被浏览器接受：证书需要由您的浏览器信任的实体签名，这些实体称之为可信证书颁发机构 (CA)  
需要创建一个证书，并使用受您的设备和浏览器本地信任的 CA 对其进行签名。可以使用工具 mkcert 通过几个命令来实现这个目的。下面介绍了它的工作原理：  
- 如果您使用 HTTPS 在浏览器中打开本地运行的网站，浏览器将检查本地开发服务器的证书。
- 在看到证书已由 mkcert 生成的证书颁发机构签名后，浏览器会检查它是否已注册为受信任的证书颁发机构。
- mkcert 已被列为受信任的颁发机构，所以浏览器会信任该证书并创建 HTTPS 连接。

mkcert（和类似工具）具备下列几种优势：  
- mkcert 专门用于创建与浏览器认为有效相兼容的证书。它会保持更新，来满足需求和最佳实践。因此您无需运行具备复杂配置或参数的 mkcert 命令，就可以生成正确的证书！
- mkcert 是跨平台的工具。团队中的任何人都可以使用

许多操作系统可能会提供用于生成证书的库，例如 openssl 。与 mkcert 和类似工具不同，此类库可能无法始终生成正确的证书，或可能需要运行复杂的命令，并且不一定能够跨平台使用。  

警告  
> 运行 mkcert -install 时，切勿导出或分享由 mkcert 自动创建的 rootCA-key.pem 。获得此文件的攻击者可以对您可能正在访问的任何网站进行路径攻击。他们可以拦截从您的电脑到任何网站（银行、医保供应商或社交网络）的安全请求。如果您需要知道 rootCA-key.pem 的位置以确保其安全，请运行 mkcert -CAROOT 。  
> 仅将 mkcert 用于开发目的，并且永远不要要求最终用户运行 mkcert 命令。
> 开发团队：所有团队成员都应该单独安装和运行 mkcert（而不是存储和共享 CA 和证书）。

**设置**  
1. 安装 mkcert（仅一次）  
   按照操作说明在操作系统上安装 mkcert。例如，在 macOS 上：
    ```
    brew install mkcert
    brew install nss # if you use Firefox
    ```
2. 将 mkcert 添加到本地根 CA   
   在终端运行以下命令：
    ```
     mkcert -install
    ```
   这会生成本地证书颁发机构 (CA)。mkcert 生成的本地 CA 仅在您的设备上本地受信。
3. 为网站生成一个由 mkcert 签名的证书
   在终端中，导航到网站的根目录或用来保存证书的任何目录。然后运行：
    ``` 
    mkcert localhost
   ```
   如果自定义主机名是 mysite.example，请运行：
    ``` 
    mkcert mysite.example
   ```
   上面的命令执行了两个操作：  
   - 为您指定的主机名生成证书
   - 让 mkcert（您在步骤 2 中添加为本地 CA）签署此证书。  
     
   为您指定的主机名生成证书
   让 mkcert（您在步骤 2 中添加为本地 CA）签署此证书。

4. 配置服务器
   现在需要告诉服务器使用 HTTPS（因为默认情况下开发服务器倾向使用 HTTP）并使用您刚刚创建的 TLS 证书  
   具体的操作取决于您的服务器。下面列出了几个例子  
   使用节点：
   server.js （替换 {PATH/TO/CERTIFICATE...} 和 {PORT} ）：  
    ``` 
     const https = require('https');
    const fs = require('fs');
    const options = {
    key: fs.readFileSync('{PATH/TO/CERTIFICATE-KEY-FILENAME}.pem'),
    cert: fs.readFileSync('{PATH/TO/CERTIFICATE-FILENAME}.pem'),
    };
    https
    .createServer(options, function (req, res) {
    // server code
    })
    .listen({PORT});
   ```
   使用 http-server：
   按如下方式启动服务器（替换 {PATH/TO/CERTIFICATE...} ）：   
   ``` 
    http-server -S -C {PATH/TO/CERTIFICATE-FILENAME}.pem -K {PATH/TO/CERTIFICATE-KEY-FILENAME}.pem
   ```
    -S 会用 HTTPS 运行服务器，-C 用来设置证书，-K 用来设置密钥
   使用 React 开发服务器：  
   按下列方式编辑 package.json ，并替换 {PATH/TO/CERTIFICATE...} ：  
    ``` 
    "scripts": {
    "start": "HTTPS=true SSL_CRT_FILE={PATH/TO/CERTIFICATE-FILENAME}.pem SSL_KEY_FILE={PATH/TO/CERTIFICATE-KEY-FILENAME}.pem react-scripts start"
   ```
   例如，如果您按照下列操作在网站根目录中为 localhost 创建了证书：  
    ``` 
   |-- my-react-app
     |-- package.json
     |-- localhost.pem
     |-- localhost-key.pem
     |--...
    ```
   start 脚本应该是这样的：
    ``` 
    "scripts": {
     "start": "HTTPS=true SSL_CRT_FILE=localhost.pem SSL_KEY_FILE=localhost-key.pem react-scripts start"
   ```

**在本地网站开启 HTTPS：其他方法**  
_自签名证书_  
可以不使用 mkcert 这样的本地证书颁发机构，而是自己签署证书  
这种方法的一些缺点：  
- 浏览器不信任您的证书颁发机构身份，因此会显示警告，您需要手动绕过。在 Chrome 中，您可以使用标志 #allow-insecure-localhost localhost 自动绕过此警告。这确实有些麻烦。
- 如果您的网络环境不安全，此举会带来潜在风险。
- 自签名证书的行为方式与受信任证书的行为方式不同。
- 它不一定比使用 mkcert 这样的本地 CA 更方便或更快捷。
- 如果您没有在浏览器上下文中使用此技术，则可能需要禁用服务器的证书验证。在生产中忘记重新启用它会带来潜在风险

> 如果您没有指定任何证书，那么 React 和 Vue 的开发服务器 HTTPS 选项会在后台创建一个自签名证书。这样虽然很快捷，但您会收到浏览器警告，并遇到与上面列出的与自签名证书相关的其他问题。可以使用前端框架的内置 HTTPS 选项并指定由 mkcert 或类似工具创建的本地可信证书。

**为什么浏览器不信任自签名证书**  
如果您使用 HTTPS 在浏览器中打开本地运行的网站，浏览器将检查本地开发服务器的证书。当它看到证书由您签名时，它会检查您是否已注册为受信任的证书颁发机构。因为您不是，所以浏览器不能信任此证书；它会警告您的连接不安全。您可以自行承担风险。如果选择这样，那么将创建 HTTPS 连接。  

**由常规证书颁发机构签署的证书**  
还可以找到基于由实际证书颁发机构（而不是本地机构）签署证书的技术。  
如果您在考虑使用这些技术，请记住以下几点：  
- 与 mkcert 这样的本地 CA 技术相比，您需要投入更多的设置工作
- 需要使用由自己控制且有效的域名。这表示实际的证书颁发机构无法用于：
  - localhost 和其他保留域名，例如 example 或 test 。
  - 您无法控制的任何域名。
  - 无效的顶级域。请参阅有效顶级域的列表。

**反向代理**  
使用 HTTPS 访问本地运行网站的另一种方法是使用反向代理，例如 ngrok  
需要考虑的几点：  
- 一旦与任何用户分享了使用反向代理创建的 URL，他们都可以访问您的本地开发网站。这在向客户演示您的项目时非常方便！但如果您的项目很敏感，这可能是一个缺点。
- 可能需要考虑定价。
- 浏览器新引入的安全措施可能会影响这些工具的表现

**标志（不推荐）**  
使用了 mysite.example 这样的自定义主机名，那么可以在 Chrome 中使用标志来强制将 mysite.example 认作是安全的。请避免使用这种方法，因为：  
- 需要 100% 确定 mysite.example 始终解析为本地地址，否则可能会泄露生产凭据
- 此方法不支持跨浏览器调试

原文: 
[如何使用 HTTPS 进行本地开发](https://mp.weixin.qq.com/s/uAh_9gIth2HNS67y2z8pew)
