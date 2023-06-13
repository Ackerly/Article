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
打开ClientUI 文件夹，只有一句代码, 直接引入了 @locator/runtime，也就是 runtime 代码。  

**2、locator/runtime**  
2.1  packages/runtime/src/index.ts   
从代码中可以看到，不管我们是 Chrome 插件引入还是通过 @locator/runtime 引入，最终都会执行 initRuntime 函数，无非就是参数不一样。  

2.2  packages/runtime/src/initRuntime.ts  
它通过 shadow Dom 的方案去添加了一个 locatorjs 自己的全局样式和容器，来隔离 CSS，以免对页面进行影响。  
它又引入了 components/Runtime 组件，并且考虑到了 SSR 的情况。我们是客户端渲染，所以走的是第二种情况。  

2.3  packages/runtime/src/components/Runtime.tsx  
最终渲染的就是 Runtime 这个函数组件。该组件使用了 solidjs。函数最上面是一些变量的声明。  
接着是在 document 上绑定了 mouseover、keydown、keyup、click、 scroll 事件    
当我们把鼠标浮到元素上，并且按住 option 按键的时候，就显示出当前元素的表框。这时候触发的就是 mouseover 和 keydown 事件。  
mouseover 处理函数，就把当前的 element 选中；keydown 处理函数使 holdingModKey 为 true。在这种情况下，页面渲染的就是 MaybeOutline 组件  

2.4  packages/runtime/src/components/MaybeOutline.tsx  
在 MaybeOutline 组件里面，主要的就是获取当前 element 的信息。然后根据 elementInfo 去渲染红色的外边框。  

2.5 packages/runtime/src/adapters/getElementInfo.tsx  
接着我们来看 getElementInfo 这个方法到底做了什么。可以看到，它用来适配器设计模式。根据适配 id 的不同，走不同的逻辑。现在我们来看 react 项目是怎么获取元素信息的。  

2.6 packages/runtime/src/adapters/react/reactAdapter.ts  
这里最重要的函数就是 findFiberByHtmlElement，通过命名也能知道。就是通过 html 元素去查询 fiber 节点。  

2.7 packages/runtime/src/adapters/react/findFiberByHtmlElement.ts  
通过 findFiberByHostInstance 就可以找到当前元素的 fiber 节点。  
_debugSource 里面居然自带了当前元素位置信息，还有 lineNumber、columnNumber，难怪可以具体定位到代码中。有了这个信息，跳转到 vscode 还不是轻而易举。  
跳转的事件发生在 click 处理函数中，最终调用了 window.open 方法。vscode:// 这个是协议跳转，electron 本身就支持。  

2.8 @babel/plugin-transform-react-jsx-source  
从 babel 官网的示例可以看出，这个 plugin 可以把当前 tag 的位置信息添加到 source 上。然后通过 React.createElement 把 source 属性挂到 _source 下面。  
然后在创建 fiber 的时候，把元素的 _source 添加到 _debugSource。  

2.9 packages/runtime/src/components/ComponentOutline.tsx  
定位到 ComponentOutline.tsx,它是通过 bbox 的属性计算出了整个元素的外边框。bbox 又是通过 fiber 元素的 getBoundingClientRect 计算出来的。  



原文:  
[LocatorJS 源码分析](https://mp.weixin.qq.com/s/xNROdWPMQJVz_cQfFx6CIw)
