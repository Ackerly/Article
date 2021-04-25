# HTML语义化标签
HTML语义标签
- article  
定义：指独立的,自包含的内容
```$xslt
<article>
<h2>Google Chrome</h2>
<p>Google Chrome is a web browser developed by Google, released in 2008. Chrome is the world's most popular web browser today!</p>
</article>

<article>
<h2>Mozilla Firefox</h2>
<p>Mozilla Firefox is an open-source web browser developed by Mozilla. Firefox has been the second most popular web browser since January, 2018.</p>
</article>

<article>
<h2>Microsoft Edge</h2>
<p>Microsoft Edge is a web browser developed by Microsoft, released in 2015. Microsoft Edge replaced Internet Explorer.</p>
</article>
```
- aside  
定义：article 以外的内容。aside 的内容应该与 article 的内容相关
```$xslt
<p>Me and my family visited The Epcot center this summer.</p>
<aside>
<h4>Epcot Center</h4>
The Epcot Center is a theme park in Disney World, Florida.
</aside>
```
- details  
定义：指定用户可以按需打开和关闭的详细信息，
```$xslt
<details>
  <summary>Epcot Center</summary>
  <p>Epcot is a theme park at Walt Disney World Resort featuring exciting attractions, international pavilions, award-winning fireworks and seasonal special events.</p>
</details>
```
- figcaption  
定义：定义 figure 元素的标题。
- figure  
定义：指定自包含的内容，比如插图，图表，图片，代码清单等
```$xslt
<figure>
  <img src="pic_trulli.jpg" alt="Trulli">
  <figcaption>Fig1. - Trulli, Puglia, Italy.</figcaption>
</figure>
```
- footer  
定义：页脚 
通常包含：作者信息，版权信息，联系信息，网站地图，回到顶部链接，相关文件
- header  
定义：表示内容或介绍一组导航链接的容器  
通常包含：
    - 一个或多个标题元素
    - 标徽或图标
    - 作者信息
```$xslt
<article>
  <header>
    <h1>A heading here</h1>
    <p>Posted by John Doe</p>
    <p>Some additional information here</p>
  </header>
  <p>Lorem Ipsum dolor set amet....</p>
</article>
```
- main
定义：文档的主要内容，并且元素中的内容对于文档来说应当是唯一的。它不应包含在文档中重复出现的内容，比如侧栏、导航栏、版权信息、站点标志或搜索表单
```$xslt
<main>
  <h1>Web Browsers</h1>
  <p>Google Chrome、Firefox 以及 Internet Explorer 是目前最流行的浏览器。</p>

  <article>
    <h1>Google Chrome</h1>
    <p>Google Chrome 是由 Google 开发的一款免费的开源 web 浏览器，于 2008 年发布。</p>
  </article>

  <article>
    <h1>Internet Explorer</h1>
    <p>Internet Explorer 由微软开发的一款免费的 web 浏览器，发布于 1995 年。</p>
  </article>

  <article>
    <h1>Mozilla Firefox</h1>
    <p>Firefox 是一款来自 Mozilla 的免费开源 web 浏览器，发布于 2004 年。</p>
  </article>
</main> 
```
- mark  
定义：定义有记号的文本。
```$xslt
<p>Do not forget to buy <mark>milk</mark> today.</p>
```
- nav  
定义：定义导航链接
```$xslt
<nav>
  <a href="/html/">HTML</a> |
  <a href="/css/">CSS</a> |
  <a href="/js/">JavaScript</a> |
  <a href="/jquery/">jQuery</a>
</nav>
```
- section  
定义：内容的主题分组，通常带有标题  
```$xslt
<section>
<h1>WWF</h1>
<p>The World Wide Fund for Nature (WWF) is an international organization working on issues regarding the conservation, research and restoration of the environment, formerly named the World Wildlife Fund. WWF was founded in 1961.</p>
</section>

<section>
<h1>WWF's Panda symbol</h1>
<p>The Panda has become the symbol of WWF. The well-known panda logo of WWF originated from a panda named Chi Chi that was transferred from the Beijing Zoo to the London Zoo in the same year of the establishment of WWF.</p>
</section>
```
- summary  
定义：为 <details> 元素定义可见的标题
- time  
定义：表示时间

参考:
[w3schools](https://www.w3schools.com/html/html5_semantic_elements.asp)
