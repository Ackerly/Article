# Vue 性能指标逐渐开始反超 React 了
Chrome 所统计的网站性能指标主要来源于三个维度：    
- LCP[1]（Largest Contentful Paint）
- FID[2]（First Input Delay）
- CLS[3]（Cumulative Layout Shift）

Chrome 的用户体验报告里主要统计到的前端开发框架有：Vue、React、Angular JS、Angular、Preact、Svelte、Next.js、Nuxt.js、SvelteKit ...   

**各大框架性能如何**  
Preact 只有 4kb 左右，而 React 有 32kb 左右，都知道前者是后者的轻量级的替代方案，从图中数据可以看出，这两者性能差不多，甚至在 PC 端的图表数据上来看 Preact 还优于 React  
在业界流传着一句话：大型应用选React，小型应用选Vue  
**资源大小**  
用户报告其实也统计了各个网站 JS 资源下载情况，这也是跟网站性能有所关联的，毕竟资源过大或多或少也会减慢页面的渲染速度，尤其是 JS 文件，需要下载再解析  
基本上每个框架构建的网站所需要下载的 JS 资源大小都达到了 1000kb 甚至以上，毕竟 SPA 应用会一次性把所有的文件都下载下来  
Svelte 好像是最优的？这是意料之中的，毕竟跟每个框架的设计有关，Svelte 选择了纯编译时（官方所说的无 runtime），也就是最终编译成直接操作原生 DOM 的代码，那么所需要下载的 JS 资源肯定相较于其它框架是少一些的  
Preact 明明是 React 轻量替代方案，展现的数据来看，Preact 确实最"重"的


参考:  
[Vue 性能指标逐渐开始反超 React 了](https://mp.weixin.qq.com/s/DPSAJCdlfg064Dgyv6HisA)
