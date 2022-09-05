<!-- TOC -->

- [6大Web安全攻防解析](#6大web安全攻防解析)
  - [XSS](#xss)
    - [非持久型XSS（反射型XSS）](#非持久型xss反射型xss)
    - [持久型XSS（存储型XSS）](#持久型xss存储型xss)
    - [防御XSS](#防御xss)
    - [HttpOnly Cookie](#httponly-cookie)
  - [CSRF](#csrf)
    - [如何防御](#如何防御)
  - [点击劫持](#点击劫持)
    - [特点](#特点)
    - [点击劫持的原理](#点击劫持的原理)
    - [如何防御](#如何防御-1)
  - [URL跳转漏洞](#url跳转漏洞)
    - [原理](#原理)
    - [实现方式](#实现方式)
    - [如何防御](#如何防御-2)
  - [SQL注入](#sql注入)
    - [原理](#原理-1)
    - [危害](#危害)
    - [如何防御](#如何防御-3)
  - [OS命令注入攻击](#os命令注入攻击)
    - [原理](#原理-2)
    - [如何防御](#如何防御-4)

<!-- /TOC -->
# 6大Web安全攻防解析
## XSS
XSS (Cross-Site Scripting)，跨站脚本攻击.跨站脚本攻击是指通过存在安全漏洞的Web网站注册用户的浏览器内运行非法的HTML标签或JavaScript进行一种攻击。  
跨站脚本攻击造成的影响：
- 利用虚假输入表单骗取用户个人信息
- 利用脚本窃取用户的Cookie值，被害者在不知情的情况下，帮助攻击者发送恶意请求。
- 显示伪造的文章或图片  

**XSS的原理是恶意攻击者往Web页面里插入恶意可执行网页脚本代码，当用户浏览该页之时，嵌入其中Web里面的脚本代码会被执行，从而可以达到攻击者盗取用户信息或其他侵犯用户安全隐私的目的**
XSS的攻击方式千变万化，可以分为几种类型
### 非持久型XSS（反射型XSS）
非持久型XSS漏洞，通过别人发送带有恶意脚本代码参数的URL，当地址被打开时，特有的恶意代码参数被HTML解析、执行。  
攻击者可以通过URL（类似https://xxx.com/xxx?default=<script>alert(document.cookie)</script>）注入可执行的脚本代码。不过Chrome内置了一些XSS过滤，可以防止大部分反射性XSS攻击。  
非持久型XSS攻击有一下几点特征：
- 即时性，不经过服务器存储，直接通过HTTP的GET和POST请求节能完成一次攻击，拿到用户隐私数据。
- 攻击者需要诱骗点击，必须通过用户点击链接才能发起
- 反馈率低，所以较难发现和响应修复
- 盗取用户敏感保密信息

防止出现非持久型XSS漏洞
- Web页面渲染的所有内容或者渲染的数据都必须来自服务端
- 尽量不要从URL，document.referrer,document.forms等这种DOM API中获取数据直接渲染
- 尽量不要使用 eval, new Function()，document.write()，document.writeln()，window.setInterval()，window.setTimeout()，innerHTML，document.createElement() 等可执行字符串的方法。
- 如果做不到以上几点，必须对涉及DOM喧嚷的方法传入的字符串参数做escape转义
- 前端渲染的时候对任何的字段都必需要做escape转义编码

### 持久型XSS（存储型XSS）
持久型XSS漏洞，一般存在于Form表单提交等交互功能，如文章留言，提交文本信息等。黑客利用XSS漏洞。将内容经正常功能提交进入数据库持久保存，当前端页面获得数据库中读出的注入代码时，恰好将其渲染执行。  
注入页面方式和非持久型XSS漏洞类似，只不过持久型的不是来源于URL，referer，forms等，而是来源后端从数据库中读出来的数据。持久型XSS攻击不需要诱骗点击，黑客只需要在提交表单的地方完成注入即可，但是这种XSS攻击成本相对还是很高。  
攻击成功需要同时满足以下几个条件：
- POST请求提交表单后端没做转义直接入库。
- 后端从数据库取出数据没做转义直接输出给前端
- 前端拿到后端数据没做转义直接渲染成DOM

持久型XSS有以下几个特点：
- 持久性，植入在数据库中
- 盗取用户敏感私密信息
- 危害面广

### 防御XSS
1. CSP  
CSP本质上就是建立白名单，开发者明确告诉浏览器那些外部资源可以加载和执行。只需要配置规则，如何拦截是浏览器自己实现。  
开启CSP的两种方式：
- 设置HTTP Header中的Content-Security-Policy
- 设置meta标签的方式

设置HTTP Header：
- 只允许加载本站资源
``` 
Content-Security-Policy: default-src 'self'
```
- 只允许加载HTTPS协议图片
``` 
Content-Security-Policy: img-src https://*
```
- 允许加载任何来源框架
``` 
Content-Security-Policy: child-src 'none'
```
对于这种方式来说，只要开发者配置了正确的规则，那么即使网站存在漏洞，攻击者也不能执行它的攻击代码，并且 CSP 的兼容性也不错。
2. 转义字符
用户的输入永远不可信任的，最普遍的做法就是转义输入输出的内容，对于引号、尖括号、斜杠进行转义
``` 
function escape(str) {
  str = str.replace(/&/g, '&amp;')
  str = str.replace(/</g, '&lt;')
  str = str.replace(/>/g, '&gt;')
  str = str.replace(/"/g, '&quto;')
  str = str.replace(/'/g, '&#39;')
  str = str.replace(/`/g, '&#96;')
  str = str.replace(/\//g, '&#x2F;')
  return str
}
```
但是对于显示富文本来说，显然不能通过上面的办法来转义所有字符，因为这样会把需要的格式也过滤掉。对于这种情况，通常采用白名单过滤的办法，当然也可以通过黑名单过滤，但是考虑到需要过滤的标签和标签属性实在太多，更加推荐使用白名单的方式。
```
const xss = require('xss')
let html = xss('<h1 id="title">XSS Demo</h1><script>alert("xss");</script>')
// -> <h1>XSS Demo</h1>&lt;script&gt;alert("xss");&lt;/script&gt;
console.log(html)
```
上面使用js-xss来实现，输出中保留了h1标签且过滤script标签。  
### HttpOnly Cookie
预防XSS攻击窃取用户cookie最有效的防御手段。Web应用程序在设置cookie时，将其属性设置为HttpOnly，可以避免该网页的cookie被客户端恶意JavaScript窃取，保护用户cookie信息。

## CSRF
跨站请求伪造，利用用户已登录的身份，在用户毫不知情的情况下，以用户的名义完成非法操作。  
CSRF攻击必须的三个条件：
- 用户登录了站点A，在本地记录了cookie
- 在用户没有登出站点A的情况下，访问了恶意攻击者提供的引诱危险站点B（B站点没有要求访问站点A）
- 站点A没有做任何CSRF防御

例如：
登入转账页面后，突然眼前一亮惊现"XXX隐私照片，不看后悔一辈子"的链接，耐不住内心躁动，立马点击了该危险的网站，但当这页面一加载，便会执行脚本代码来提交转账请求，从而将钱转给黑客。
### 如何防御
防范CSRF攻击可以遵循以下几种规则：
- Get请求不对数据进行修改
- 不让第三方网站访问到用户Cookie
- 阻止第三方网站请求接口
- 请求附带验证信息，比如验证码或者Token

1. SameSite
可以对 Cookie 设置 SameSite 属性。该属性表示 Cookie 不随着跨域请求发送，可以很大程度减少 CSRF 的攻击，但是该属性目前并不是所有浏览器都兼容。
2. Referer Check
HTTP Referer是header的一部分，当浏览器向web服务器发送请求时，一般会带上Referer信息告诉服务器是从哪个页面链接过来的，服务器籍此可以获得一些信息用于处理。可以通过检查请求的来源来防御CSRF攻击。正常请求的referer具有一定规律，如在提交表单的referer必定是在该页面发起的请求。所以通过检查http包头referer的值是不是这个页面，来判断是不是CSRF攻击。  
在某些情况下如从https跳转到http，浏览器处于安全考虑，不会发送referer，服务器就无法进行check了。若与该网站同域的其他网站有XSS漏洞，那么攻击者可以在其他网站注入恶意脚本，受害者进入了此类同域的网址，也会遭受攻击。出于以上原因，无法完全依赖Referer Check作为防御CSRF的主要手段。但是可以通过Referer Check来监控CSRF攻击的发生。
3. Anti CSRF Token
发送请求时在HTTP 请求中以参数的形式加入一个随机产生的token，并在服务器建立一个拦截器来验证这个token。服务器读取浏览器当前域cookie中这个token值，会进行校验该请求当中的token和cookie当中的token值是否都存在且相等，才认为这是合法的请求。否则认为这次请求是违法的，拒绝该次服务。  
token可以在用户登陆后产生并放于session或cookie中，然后在每次请求时服务器把token从session或cookie中拿出，与本次请求中的token 进行比对。由于token的存在，攻击者无法再构造出一个完整的URL实施CSRF攻击。但在处理多个页面共存问题时，当某个页面消耗掉token后，其他页面的表单保存的还是被消耗掉的那个token，其他页面的表单提交时会出现token错误。
4. 验证码
应用程序和用户进行交互过程中，特别是账户交易这种核心步骤，强制用户输入验证码，才能完成最终请求。。但增加验证码降低了用户的体验，网站不能给所有的操作都加上验证码。所以只能将验证码作为一种辅助手段，在关键业务点设置验证码。

## 点击劫持
攻击者将需要攻击的网站通过 iframe 嵌套的方式嵌入自己的网页中，并将 iframe 设置为透明，在页面中透出一个按钮诱导用户点击。
### 特点
- 隐蔽性较高，骗取用户操作
- "UI-覆盖攻击"
- 利用iframe或者其它标签的属性
### 点击劫持的原理
用户在登陆 A 网站的系统后，被攻击者诱惑打开第三方网站，而第三方网站通过 iframe 引入了 A 网站的页面内容，用户在第三方网站中点击某个按钮（被装饰的按钮），实际上是点击了 A 网站的按钮。
### 如何防御
1. X-FRAME-OPTIONS
X-FRAME-OPTIONS是一个 HTTP 响应头，在现代浏览器有一个很好的支持。这个 HTTP 响应头 就是为了防御用 iframe 嵌套的点击劫持攻击。
该响应头有三个值可选，
- DENY，表示页面不允许通过 iframe 的方式展示
- SAMEORIGIN，表示页面可以在相同域名下通过 iframe 的方式展示
- ALLOW-FROM，表示页面可以在指定来源的 iframe 中展示

2. JavaScript 防御
对于某些远古浏览器来说，并不能支持上面的这种方式，那我们只有通过 JS 的方式来防御点击劫持了。
``` 
<head>
  <style id="click-jack">
    html {
      display: none !important;
    }
  </style>
</head>
<body>
  <script>
    if (self == top) {
      var style = document.getElementById('click-jack')
      document.body.removeChild(style)
    } else {
      top.location = self.location
    }
  </script>
</body>
```
以上代码的作用就是当通过 iframe 的方式加载页面时，攻击者的网页直接不显示所有内容了。
## URL跳转漏洞
借助未验证的URL跳转，将应用程序引导到不安全的第三方区域，从而导致安全问题
### 原理
黑客利用URL跳转漏洞来诱导安全意识低的用户点击，导致用户信息泄露或者资金流失。其原理是黑客构建恶意链接（链接需要进行伪装尽可能迷惑）发在QQ群或者是浏览量多的贴吧/论坛中。  
安全意识低的用户点击后进过服务器或者浏览器解析后跳到恶意的网站中。 
恶意链接需要进行伪装经常的做法是熟悉链接后面加上一个恶意的网址，迷惑用户。  
例如：
``` 
http://gate.baidu.com/index?act=go&url=http://t.cn/RVTatrd
http://qt.qq.com/safecheck.html?flag=1&url=http://t.cn/RVTatrd
http://tieba.baidu.com/f/user/passport?jumpUrl=http://t.cn/RVTatrd
```
### 实现方式
- Header头跳转
``` 
<?php
$url=$_GET['jumpto'];
header("Location: $url");
?>
```
- JavaScript跳转
- META标签跳转
### 如何防御
1. referer的限制
如果确定传递URL参数进入的来源，我们可以通过该方式实现安全限制，保证URL的有效性，避免恶意用户自己生成跳转链接
2.加入有效性验证Token
保证所有生成的链接都是来自我们可信域，通过在生成的链接加入用户不可控的Token对生成的链接进行校验，避免用户生成自己的恶意链接从而被利用，如果功能比较开放，可能导致有一定的限制。
## SQL注入
SQL是常见的Web安全漏洞，攻击者利用这个漏洞，可以访问或修改数据，或者利用潜在的数据库漏洞进行攻击
### 原理
前端
``` 
<form action="/login" method="POST">
    <p>Username: <input type="text" name="username" /></p>
    <p>Password: <input type="password" name="password" /></p>
    <p><input type="submit" value="登陆" /></p>
</form>
```
后端
``` 
let querySQL = `
    SELECT *
    FROM user
    WHERE username='${username}'
    AND psw='${password}'
`;
```
如果有一个恶意攻击者输入的用户名是admin‘ --，密码随意输入，就可以直接登录系统
之前预想的SQL语句是
``` 
SELECT * FROM user WHERE username='admin' AND psw='password'
```
恶意攻击用奇怪的用户名将SQL语句变成：
``` 
SELECT * FROM user WHERE username='admin' --' AND psw='xxxx'
```
在SQL中’--是闭合和注释的意思，--注释后面的内容，所以查询语句变成：
``` 
SELECT * FROM user WHERE username='admin'
```
SQL注入过程包括以下几个过程
- 获取用户请求参数
- 拼接到代码当中
- SQL语句按照我们构造参数的语义执行成功

SQL注入的必备条件：
- 可以控制输入的数据
- 服务器执行的代码拼接了控制的数据

### 危害
1. 获取数据库信息
- 管理员后台用户名和密码
- 获取其他数据库敏感信息：用户名、密码、手机号码、身份证、银行卡信息......
- 整个数据库：脱库
2. 获取服务器权限
3.植入Webshell，获取服务器后门
4，读取服务器敏感文件
### 如何防御
- 严格限制Web应用的数据库的操作全向，给用户提供仅仅能满足其工作的最低权限，从而最大限度的减少注入攻击对数据库的危害
- 后端代码检查输入的数据是否符合预期，严格限制变量的类型，例如使用正则表达式进行一些匹配处理。
- 对进入数据库的特殊字符（'，"，\，<，>，&，*，; 等）进行转义处理，或编码转换。
- 所有的查询语句建议使用数据库提供的参数化查询接口，参数化的语句使用参数而不是将用户输入变量嵌入到 SQL 语句中，即不要直接拼接 SQL 语句。
## OS命令注入攻击
OS命令注入攻击指通过Web应用，执行非法的操作系统命令达到攻击的目的。
### 原理
黑客构造命令提交给web应用程序，web应用程序提取黑客构造的命令，拼接到被执行的命令中，因黑客注入的命令打破了原有命令结构，导致web应用执行了额外的命令，最后web应用程序将执行的结果输出到响应页面中。
例如：
``` 
// 以 Node.js 为例，假如在接口中需要从 github 下载用户指定的 repo
const exec = require('mz/child_process').exec;
let params = {/* 用户输入的参数 */};
exec(`git clone ${params.repo} /some/path`);
```
如果 params.repo 传入的是 https://github.com/admin/admin.github.io.git 从指定的 git repo 上下载到想要的代码。
如果 params.repo 传入的是 https://github.com/xx/xx.git && rm -rf /* && 恰好服务是用 root 权限就会把库给删除了。
### 如何防御
- 后端对前端提交内容进行规则限制（比如正则表达式）
- 在调用系统命令前对所有传入参数进行命令行参数转义过滤
- 不要直接拼接命令语句，借助一些工具做拼接、转义预处理

原文:  
[常见六大Web安全攻防解析](https://github.com/ljianshu/Blog/issues/56)
