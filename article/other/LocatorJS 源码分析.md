# LocatorJS 源码分析
## 准备工作
**1. 拉取源码**  
把源码拉到本地，该项目把 Chrome 插件和示例都放在 apps 文件夹里面，packages 里面放着 babel-jsx、locatorjs、runtime 等公共代码。可以看到这个项目是使用 lerna 管理的多包项目，那需要用 lerna bootstrap 进行依赖安装。  
**2. 启动项目**  
依赖安装完毕，然后使用 yarn dev 启动项目。可以看到 turbo 编译速度飞快，端口信息很快就被日志刷没了。于是只能通过代码里查看，具体的示例在哪个端口。  
**3. 加载 Chrome 插件**  
因为我们是采用的 Chrome 插件和 react devtools 进行代码分析。所以必须安装这 2 个插件。react devtools 如果没有安装，可以去 Chrome 插件市场里搜索安装。locatorjs Chrome 插件我们加载开发版本进行代码调试。（进去 Chrome 扩展管理 -> 加载我们项目 apps/extension/build/development_chrome 这个文件夹）  

## 源码分析
1. Chrome 插件  
   打开 apps/extension/src/pages 这个文件，发现里面就是常规的插件代码。
    - Background 放着插件的后台代码，目前里面为空
    - Content 放着插件与浏览器内容页面的代码，与页面代码一起执行
    - Popup 放着插件 popup 页面的代码

这个插件的执行入口就在 content.ts。  


1.1 apps/extension/src/pages/Content/index.ts  
在 content.ts 中把 hook.bundle.js 注入到当前页面中，把 client.bundle.js 赋值给 document.documentElement.dataset.locatorClientUrl，其他代码都是一些监听事件，可以先不管。  

1.2 apps/extension/src/pages/Hook/index.ts  
hook.bundle.js 就是 hook.ts 打包之后的文件名，它主要做了 2 件事情，第一，确保 react devtools 已经被安装了；第二，注入 runtime script。  

1.3 apps/extension/src/pages/Hook/insertRuntimeScript.ts  
该文件主要是监听 DOMContentLoaded，然后注入我们上面 document.documentElement.dataset.locatorClientUrl 这个文件，也就是 client.bundle.js。  

1.4  apps/extension/src/pages/ClientUI/index.ts  
ClientUI 文件夹，直接引入了 @locator/runtime，也就是 runtime 代码。


原文:  
[LocatorJS 源码分析](https://mp.weixin.qq.com/s/xNROdWPMQJVz_cQfFx6CIw)
