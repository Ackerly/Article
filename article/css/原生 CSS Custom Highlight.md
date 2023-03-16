# 原生 CSS Custom Highlight
CSS Custom Highlight API，有了它，可以在不改变 dom 结构的情况下自定义任意文本的样式  
**伪元素 ::highlight()**  
CSS 部分，一个新的伪元素，非常简单  
``` 
::highlight(custom-highlight-name) {
   color: red
 }
```
和::selection 这类伪元素比较类似，仅支持部分文本相关样式，如下  
- 文本颜色: color
- 背景颜色: background-color
- 文本修饰: text-decoration
- 文本阴影: text-shadow
- 文本描边: -webkit-text-stroke
- 文本填充: -webkit-text-fill-colo

仅仅知道这个伪类是没用的，她还需要一个 “参数”，也就是上面的 custom-highlight-name，表示高亮的名称，那这个是怎么来的呢？或者换句话说，如何去标识页面中需要自定义样式的那部分文本呢？  
这就需要借助下面的内容了，看看如何生成这个 “参数”，这才是重点  

## CSS Custom Highlight API
大部分操作其实和这个原理是相同的，只是把拿到的选区做了进一步处理，具体分以下几步  
**1. 创建选区（重点）**  
通过 Range 对象创建文本选择范围，就像用鼠标滑过选区一样，这也是最复杂的一部分，例如  
``` 
const parentNode = document.getElementById("foo");

 const range1 = new Range();
 range1.setStart(parentNode, 10);
 range1.setEnd(parentNode, 20);

 const range2 = new Range();
 range2.setStart(parentNode, 40);
 range2.setEnd(parentNode, 60);
```
这样可以得到选区对象 range1、range2  
**2. 创建高亮**  
将创建的选区高亮实例化，需要用到 Highlight 对象  
``` 
const highlight = new Highlight(range1, range2, ...);
```
当然也可以根据需求创建多个  
``` 
const highlight1 = new Highlight(user1Range1, user1Range2);
 const highlight2 = new Highlight(user2Range1, user2Range2, user2Range3);
```
这样可以得到高亮对象 highlight1、highlight2  
**注册高亮**  
需要将实例化的高亮对象通过 CSS.Highlight 注册到页面  
有点类似于 Map 对象的操作  
``` 
CSS.highlights.set("highlight1", highlight1);
CSS.highlights.set("highlight2", highlight2);
```
目前兼容性比较差，所以需要额外判断一下  
``` 
if (CSS.highlights) {
   //...支持CSS.highlights
 }
```
上面注册的 key 名，highlight1 就是上一节提到的高亮名称，也就是 CSS 中需要的 “参数”  
**4. 自定义样式**  

``` 
::highlight(highlight1) {
   background-color: yellow;
   color: black;
 }
```
以上就是全部过程了，稍显复杂，但是还是比较好理解的，关键是第一步创建选区的过程，最为复杂  

## 彩虹文本
彩虹文本效果。总共 7 种颜色，文字依次变色，不断循环，而且仅有一个标签  
``` 
<p id="rainbow-text">CSS Custom Highlight API</p>
```
这里总共有 7 种颜色，所以需要创建 7 个高亮区域，可以先定义高亮 CSS，如下  
``` 
::highlight(rainbow-color-1) { color: #ad26ad;  text-decoration: underline; }
 ::highlight(rainbow-color-2) { color: #5d0a99;  text-decoration: underline; }
 ::highlight(rainbow-color-3) { color: #0000ff;  text-decoration: underline; }
 ::highlight(rainbow-color-4) { color: #07c607;  text-decoration: underline; }
 ::highlight(rainbow-color-5) { color: #b3b308;  text-decoration: underline; }
 ::highlight(rainbow-color-6) { color: #ffa500;  text-decoration: underline; }
 ::highlight(rainbow-color-7) { color: #ff0000;  text-decoration: underline; }
```
下面通过循环，创建 7 个高亮区域  
``` 
const textNode = document.getElementById("rainbow-text").firstChild;

 if (CSS.highlights) {

   const highlights = [];
   for (let i = 0; i < 7; i++) {
     // 给每个颜色实例化一个Highlight对象
     const colorHighlight = new Highlight();
     highlights.push(colorHighlight);

     // 注册高亮
     CSS.highlights.set(`rainbow-color-${i + 1}`, colorHighlight);
   }

   // 遍历文本节点
   for (let i = 0; i < textNode.textContent.length; i++) {
     // 给每个字符创建一个选区
     const range = new Range();
     range.setStart(textNode, i);
     range.setEnd(textNode, i + 1);

     // 添加到高亮
     highlights[i % 7].add(range);
   }
 }
```
## 文本搜索高亮
浏览器的搜索功能，ctrl+f 就可以快速对整个网页就行查找，查找到的关键词会添加黄色背景的高亮  
有了 CSS Custom Highlight API ，完全可以手动实现一个和原生浏览器一模一样的搜索高亮功能。  
目前为止，还无法自定义原生搜索高亮的黄色背景，以后可能会开放  
假设 HTML 结构是这样的，一个搜索框和一堆文本  
``` 
label>搜索 <input id="query" type="text"></label>
 <article>
   <p>
     阅文旗下囊括 QQ 阅读、起点中文网、新丽传媒等业界知名品牌，汇聚了强大的创作者阵营、丰富的作品储备，覆盖 200 多种内容品类，触达数亿用户，已成功输出《庆余年》《赘婿》《鬼吹灯》《全职高手》《斗罗大陆》《琅琊榜》等大量优秀网文 IP，改编为动漫、影视、游戏等多业态产品。
   </p>
   <p>
     《盗墓笔记》最初连载于起点中文网，是南派三叔成名代表作。2015年网剧开播首日点击破亿，开启了盗墓文学 IP 年。电影于2016年上映，由井柏然、鹿晗、马思纯等主演，累计票房10亿元。
   </p>
   <p>
     庆余年》是阅文集团白金作家猫腻的作品，自2007年在起点中文网连载，持续保持历史类收藏榜前五位。改编剧集成为2019年现象级作品，播出期间登上微博热搜百余次，腾讯视频、爱奇艺双平台总播放量突破160亿次，并荣获第26届白玉兰奖最佳编剧（改编）、最佳男配角两项大奖。
   </p>
   <p>《鬼吹灯》是天下霸唱创作的经典悬疑盗墓小说，连载于起点中文网。先后进行过漫画、游戏、电影、网络电视剧的改编，均取得不俗的成绩，是当之无愧的超级IP。</p>
 </article>
```
``` 
const query = document.getElementById("query");
 const article = document.querySelector("article");

 // 创建 createTreeWalker 迭代器，用于遍历文本节点，保存到一个数组
 const treeWalker = document.createTreeWalker(article, NodeFilter.SHOW_TEXT);
 const allTextNodes = [];
 let currentNode = treeWalker.nextNode();
 while (currentNode) {
   allTextNodes.push(currentNode);
   currentNode = treeWalker.nextNode();
 }

 // 监听inpu事件
 query.addEventListener("input", () => {
   // 判断一下是否支持 CSS.highlights
   if (!CSS.highlights) {
     article.textContent = "CSS Custom Highlight API not supported.";
     return;
   }

   // 清除上个高亮
   CSS.highlights.clear();

   // 为空判断
   const str = query.value.trim().toLowerCase();
   if (!str) {
     return;
   }

   // 查找所有文本节点是否包含搜索词
   const ranges = allTextNodes
     .map((el) => {
       return { el, text: el.textContent.toLowerCase() };
     })
     .map(({ text, el }) => {
       const indices = [];
       let startPos = 0;
       while (startPos < text.length) {
         const index = text.indexOf(str, startPos);
         if (index === -1) break;
         indices.push(index);
         startPos = index + str.length;
       }

       // 根据搜索词的位置创建选区
       return indices.map((index) => {
         const range = new Range();
         range.setStart(el, index);
         range.setEnd(el, index + str.length);
         return range;
       });
     });

   // 创建高亮对象
   const searchResultsHighlight = new Highlight(...ranges.flat());

   // 注册高亮
   CSS.highlights.set("search-results", searchResultsHighlight);
 });
```
通过 CSS 设置高亮的颜色  
``` 
::highlight(search-results) {
   background-color: #f06;
   color: white;
 }
```
还可以将高亮效果改成波浪线  
``` 
::highlight(search-results) {
   text-decoration: underline wavy #f06;
 }
```
除了避免 dom 操作带来的便利外，性能也能得到极大的提升，毕竟创建、移除 dom 也是性能大户  
在 10000 个节点的情况下，使用highlight和span两者相差 100 倍的差距！而且数量越大，性能差距越明显，甚至直接导致浏览器卡死！ 

## 代码高亮编辑器
假设 HTML 结构是这样的，很简单，就一个纯文本的标签  
``` 
<pre class="editor" id="code">ul{
   min-height: 0;
 }
 .sub {
   display: grid;
   grid-template-rows: 0fr;
   transition: 0.3s;
   overflow: hidden;
 }
 :checked ~ .sub {
   grid-template-rows: 1fr;
 }
 .txt{
   animation: color .001s .5 linear forwards;
 }
 @keyframes color {
   from {
     color: var(--c1)
   }
   to{
     color: var(--c2)
   }
 }</pre>
```
简单修饰一下，设置为可编辑元素  
``` 
.editor{
   white-space: pre-wrap;
   -webkit-user-modify: read-write-plaintext-only; /* 读写纯文本 */
 }
```
如何让这些代码高亮呢？  
这就需要对内容进行关键词分析提取了，我们可以用现有的代码高亮库，比如 highlight.js 
``` 
hljs.highlight(pre.textContent, {
    language: 'css'
  })._emitter.rootNode.children
```
对代码内容进行遍历了，方法也是类似的，如下  
``` 
const nodes = pre.firstChild
 const text = nodes.textContent
 const highlightMap = {}
 let startPos = 0;
 words.filter(el => el.scope).forEach(el => {
   const str = el.children[0]
   const scope = el.scope
   const index = text.indexOf(str, startPos);
   if (index < 0) {
     return
   }
   const item = {
     start: index,
     scope: scope,
     end: index + str.length,
     str: str
   }
   if (highlightMap[scope]){
     highlightMap[scope].push(item)
   } else {
     highlightMap[scope] = [item]
   }
   startPos = index + str.length;
 })
 Object.entries(highlightMap).forEach(function([k,v]){
   const ranges = v.map(({start, end}) => {
     const range = new Range();
     range.setStart(nodes, start);
     range.setEnd(nodes, end);
     return range;
   });
   const highlight = new Highlight(...ranges.flat());
   CSS.highlights.set(k, highlight);
 })
 }
 highlights(code)
 code.addEventListener('input', function(){
   highlights(this)
 })
```
最后，根据不同的类型，定义不同的颜色就行了，如下  
``` 
::highlight(built_in) {
     color: #c18401;
   }
 ::highlight(comment) {
   color: #a0a1a7;
   font-style: italic;
   }
 ::highlight(number),
 ::highlight(selector-class){
     color: #986801;
   }
 ::highlight(attr) {
     color: #986801;
   }
 ::highlight(string) {
     color: #50a14f;
   }
 ::highlight(selector-pseudo) {
     color: #986801;
   }
 ::highlight(attribute) {
     color: #50a14f;
   }
 ::highlight(keyword) {
     color: #a626a4;
   }
```

原文:  
[原生 CSS Custom Highlight 终于来了~](https://mp.weixin.qq.com/s/42R18FerTJ7_P17aNp8OMA)
