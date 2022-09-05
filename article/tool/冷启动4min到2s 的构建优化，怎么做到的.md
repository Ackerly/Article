# 冷启动4min到2s 的构建优化，怎么做到的
**项目背景**  
一个 ToB 的 Web 单页应用经过多年的迭代，目前已经累积有大几十万行的业务代码，30+ 路由模块，整体的代码量和复杂度还是比较高的  
项目整体是基于 Vue + TypeScirpt，而构建工具，由于最早项目是经由 vue-cli 初始化而来，所以自然而然使用的是 Webpack  
随着项目体量越来越大，在开发阶段将项目跑起来，也就是通过 npm run serve 的单次冷启动时间，以及在项目发布时候的 npm run build 的耗时都会越来越久。  
打包构建优化也是伴随项目的成长需要持续不断去做的事情。在早期，项目体量比较小的时，构建优化的效果可能还不太明显，而随着项目体量的增大，构建耗时逐渐增加，如何尽可能的降低构建时间，则显得越来越重要：  
1. 大项目通常是团队内多人协同开发，单次开发时的冷启动时间的降低，乘上人数及天数，经年累月节省下来的时间非常可观，能较大程度的提升开发效率、提升开发体验
2. 大项目的发布构建的效率提升，能更好的保证项目发布、回滚等一系列操作的准确性、及时性

**瓶颈分析**  
使用SMP 插件，统计各模块耗时数据  
> speed-measure-webpack-plugin 是一款统计 webpack 打包时间的插件，不仅可以分析总的打包时间，还能分析各阶段loader 的耗时，并且可以输出一个文件用于永久化存储数据

``` 
// 安装
npm install --save-dev speed-measure-webpack-plugin
```
``` 
// 使用方式
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();
config.plugins.push(smp());
```

**开发阶段构建耗时**  
对于 npm run serve，也就是开发阶段而言，在没有任何缓存的前提下，单次冷启动整个项目的时间达到了惊人的 4 min  
**生产阶段构建耗时**  
对于 npm run build，也就是实际线上生产环境的构建，看看总体的耗时7min  
优化分为两个方向：  
1. 基于开发阶段 npm run serve 的优化  
   在开发阶段，核心目标是在保有项目所有功能的前提下，尽可能提高构建速度，保证开发时的效率，所以对于 Live 才需要的一些功能，譬如代码混淆压缩、图片压缩等功能是可以不开启的，并且在开发阶段，我们需要热更新
2. 基于生产阶段 npm run build 的优化  
   在生产打包阶段，尽管构建速度也非常重要，但是一些在开发时可有可无的功能必须加上，譬如代码压缩、图片压缩。因此，生产构建的目标是在于保证最终项目打包体积尽可能小，所需要的相关功能尽可能完善的前提下，同时保有较快的构建速度。  

基于上述的一些分析，本文将从如下几个方面探讨对构建效率优化的探索：  
1. 基于 Webpack 的一些常见传统优化方式
2. 分模块构建
3. 基于 Vite 的构建工具切换
4. 基于 Es-build 插件的构建效率优化

**为什么这么慢**  
对于一个单页应用，构建过程中的大部分时间都消耗在编译 JavaScript 文件及 CSS 文件的各类  Loader 上。  
Webpack 的构建流程，主要时间花费在递归遍历各个入口文件，并基于入口文件不断寻找依赖逐个编译再递归处理的过程，每次递归都需要经历 String->AST->String 的流程，然后通过不同的 loader 处理一些字符串或者执行一些 JavaScript 脚本，由于 NodeJS 单线程的特性以及语言本身的效率限制，Webpack 构建慢一直成为它饱受诟病的原因。  
基于上述 Webpack 构建的流程及提到的一些问题，整体的优化方向就变成了：  
1. 缓存
2. 多进程
3. 寻路优化
4. 抽离拆分
5. 构建工具替换

**基于 Webpack 的传统优化方式**  
构建过程中的大部分时间都消耗在递归地去编译 JavaScript 及 CSS 的各类  Loader 上，并且会受限于 NodeJS 单线程的特性以及语言本身的效率限制  
如果不替换掉 Webpack 本身，语言本身（NodeJS）的执行效率是没法优化的，只能在其他几个点做文章  
在最早期所做的都是一些比较常规的优化手段，这里简单介绍最为核心的几个：
1. 缓存
2. 多进程
3. 寻址优化

**缓存优化**  
cache-loader: 在一些性能开销较大的 loader 之前添加 cache-loader，以便将结果缓存到磁盘里  
_HardSourceWebpackPlugin_  
HardSourceWebpackPlugin 为模块提供中间缓存，缓存默认存放的路径是 node_modules/.cache/hard-source，配置了 HardSourceWebpackPlugin 之后，首次构建时间并没有太大的变化，但是第二次开始，构建时间将会大大的加快。  
首先安装依赖：  
``` 
npm install hard-source-webpack-plugin -D
```
修改 vue.config.js 配置文件：  
``` 
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin');
module.exports = {
  ...
  configureWebpack: (config) => {
    // ...
    config.plugins.push(new HardSourceWebpackPlugin());
  },
  ...
}
```
配置了 HardSourceWebpackPlugin 的首次构建时间，和预期的一样，并没有太大的变化，但是第二次构建从平均 4min 左右降到了平均 20s 左右，提升的幅度非常的夸张，当然，这个也因项目而异，但是整体而言，在不同项目中实测发现它都能比较大的提升开发时二次编译的效率。  

**设置 babel-loader 的 cacheDirectory 以及 DLL**  
打开 babel-loader 的 cacheDirectory 的配置，当有设置时，指定的目录将用来缓存 loader 的执行结果。之后的 webpack 构建，将会尝试读取缓存，来避免在每次执行时，可能产生的、高性能消耗的 Babel 重新编译过程。  
DLL 文件为动态链接库，在一个动态链接库中可以包含给其他模块调用的函数和数据。  
为什么要用 DLL？  
原因在于包含大量复用模块的动态链接库只需要编译一次，在之后的构建过程中被动态链接库包含的模块将不会在重新编译，而是直接使用动态链接库中的代码  
由于动态链接库中大多数包含的是常用的第三方模块，例如 Vue、React、React-dom，只要不升级这些模块的版本，动态链接库就不用重新编译  
DLL 的配置非常繁琐，并且最终收效甚微,可以借助了 autodll-webpack-plugin  

**HappyPack**  
HappyPack 可利用多进程对文件进行打包, 将任务分解给多个子进程去并行执行，子进程处理完后，再把结果发送给主进程，达到并行打包的效、HappyPack 并是所有的 loader 都支持, 比如 vue-loader 就不支持  
可以通过 Loader Compatibility List 来查看支持的 loaders。需要注意的是，创建子进程和主进程之间的通信是有开销的，当你的 loader 很慢的时候，可以加上 happypack。否则，可能会编译的更慢  

**寻址优化**  
它的核心即在于，合理设置 loader 的 exclude 和 include 属性  
- 通过配置 loader 的 exclude 选项，告诉对应的 loader 可以忽略某个目录
- 通过配置 loader 的 include 选项，告诉 loader 只需要处理指定的目录，loader 处理的文件越少，执行速度就会更快

这肯定是有用的优化手段，只是对于一些大型项目而言，这类优化对整体构建时间的优化不会特别明显  

## 分模块构建
webpack 构建流程：  
- entry-option：读取 webpack 配置，调用 new Compile(config) 函数准备编译
- run：开始编译
- make：从入口开始分析依赖，对依赖模块进行 build
- before-resolve：对位置模块进行解析
- build-module：开始构建模块
- normal-module-loader：生成 AST 树
- program：遍历 AST 树，遇到 require 语句收集依赖
- seal：build 完成开始优化
- emit：输出 dist 目录

随着项目体量地不断增大，耗时大头消耗在第 7 步，递归遍历 AST，解析 require，如此反复直到遍历完整个项目  
对于单次单个开发而言，极大概率只是基于这整个大项目的某一小个模块进行开发即可  
如果我们可以在收集依赖的时候，跳过我们本次不需要的模块，或者可以自行选择，只构建必要的模块，那么整体的构建时间就可以大大减少。这也就是我们要做的 -- 分模块构建。  
举个栗子，假设我们的项目一共有 6 个大的路由模块 A、B、C、D、E、F，当新需求只需要在 A 模块范围内进行优化新增，那么我们在开发阶段启动整个项目的时候，可以跳过 B、C、D、E、F 这 5 个模块，只构建 A 模块即可  
假设原本每个模块的构建平均耗时 3s，原本 18s 的整体冷启动构建耗时就能下降到 3s。  

**分模块构建打包的原理**  
Webpack 是静态编译打包的，Webpack 在收集依赖时会去分析代码中的 require（import 会被 bebel 编译成 require） 语句，然后递归的去收集依赖进行打包构建。  
我们要做的，就是通过增加一些配置，简单改造下我们的现有代码，使得 Webpack 在初始化遍历整个路由模块收集依赖的时候，可以跳过我们不需要的模块。  
假设我们的路由大致代码如下：  
``` 
import Vue from 'vue';
import VueRouter, { Route } from 'vue-router';

// 1. 定义路由组件.
// 这里简化下模型，实际项目中肯定是一个一个的大路由模块，从其他文件导入
const moduleA = { template: '<div>AAAA</div>' }
const moduleB = { template: '<div>BBBB</div>' }
const moduleC = { template: '<div>CCCC</div>' }
const moduleD = { template: '<div>DDDD</div>' }
const moduleE = { template: '<div>EEEE</div>' }
const moduleF = { template: '<div>FFFF</div>' }

// 2. 定义一些路由
// 每个路由都需要映射到一个组件。
// 我们后面再讨论嵌套路由。
const routesConfig = [
  { path: '/A', component: moduleA },
  { path: '/B', component: moduleB },
  { path: '/C', component: moduleC },
  { path: '/D', component: moduleD },
  { path: '/E', component: moduleE },
  { path: '/F', component: moduleF }
]

const router = new VueRouter({
  mode: 'history',
  routes: routesConfig,
});

// 让路由生效 ...
const app = Vue.createApp({})
app.use(router)
```
每次启动项目时，可以通过一个前置命令行脚本，收集本次需要启动的模块，按需生成需要的 routesConfig 即可  
实现方式:  
- IgnorePlugin 插件
- webpack-virtual-modules 配合 require.context
- NormalModuleReplacementPlugin 插件进行文件替换

最终选择了使用 NormalModuleReplacementPlugin 插件进行文件替换的方式，原因在于它对整个项目的侵入性非常小，只需要添加前置脚本及修改 Webpack 配置，无需改变任何路由文件代码。总结而言，该方案的两点优势在于：  
- 无需改动上层代码
- 通过生成临时路由文件的方式，替换原路由文件，对项目无任何影响

**使用 NormalModuleReplacementPlugin 生成新的路由配置文件**  
利用 NormalModuleReplacementPlugin 插件，可以不修改原来的路由配置文件，在编译阶段根据配置生成一个新的路由配置文件然后去使用它，这样做的好处在于对整个源码没有侵入性  
NormalModuleReplacementPlugin 插件的作用在于，将目标源文件的内容替换为我们自己的内容  
简单修改 Webpack 配置，如果当前是开发环境，利用该插件，将原本的 config.ts 文件，替换为另外一份，代码如下：  
``` 
// vue.config.js
if (process.env.NODE_ENV === 'development') {
  config.plugins.push(new webpack.NormalModuleReplacementPlugin(
      /src\/router\/config.ts/,
      '../../dev.routerConfig.ts'
    )
  )
}
```
上面的代码功能是将实际使用的 config.ts 替换为自定义配置的 dev.routerConfig.ts 文件，那么 dev.routerConfig.ts 文件的内容又是如何产生的呢，其实就是借助了 inquirer 与 EJS 模板引擎，通过一个交互式的命令行问答，选取需要的模块，基于选择的内容，动态的生成新的 dev.routerConfig.ts 代码，这里直接上代码。  
改造一下启动脚本，在执行 vue-cli-service serve 前，先跑一段我们的前置脚本：  
``` 
{
  // ...
  "scripts": {
    - "dev": "vue-cli-service serve",
    + "dev": "node ./script/dev-server.js && vue-cli-service serve",
  },
  // ...
}
```
而 dev-server.js 所需要做的事，就是通过 inquirer 实现一个交互式命令，用户选择本次需要启动的模块列表，通过 ejs 生成一份新的 dev.routerConfig.ts 文件。  
``` 
// dev-server.js
const ejs = require('ejs');
const fs = require('fs');
const child_process = require('child_process');
const inquirer = require('inquirer');
const path = require('path');

const moduleConfig = [
    'moduleA',
    'moduleB',
    'moduleC',
    // 实际业务中的所有模块
]

//选中的模块
const chooseModules = [
  'home'
]

function deelRouteName(name) {
  const index = name.search(/[A-Z]/g);
  const preRoute = '' + path.resolve(__dirname, '../src/router/modules/') + '/';
  if (![0, -1].includes(index)) {
    return preRoute + (name.slice(0, index) + '-' + name.slice(index)).toLowerCase();
  }
  return preRoute + name.toLowerCase();;
}

function init() {
  let entryDir = process.argv.slice(2);
  entryDir = [...new Set(entryDir)];
  if (entryDir && entryDir.length > 0) {
    for(const item of entryDir){
      if(moduleConfig.includes(item)){
        chooseModules.push(item);
      }
    }
    console.log('output: ', chooseModules);
    runDEV();
  } else {
    promptModule();
  }
}

const getContenTemplate = async () => {
  const html = await ejs.renderFile(path.resolve(__dirname, 'router.config.template.ejs'), { chooseModules, deelRouteName }, {async: true});
  fs.writeFileSync(path.resolve(__dirname, '../dev.routerConfig.ts'), html);
};

function promptModule() {
  inquirer.prompt({
    type: 'checkbox',
    name: 'modules',
    message: '请选择启动的模块, 点击上下键选择, 按空格键确认(可以多选), 回车运行。注意: 直接敲击回车会全量编译, 速度较慢。',
    pageSize: 15,
    choices: moduleConfig.map((item) => {
      return {
        name: item,
        value: item,
      }
    })
  }).then((answers) => {
    if(answers.modules.length===0){
      chooseModules.push(...moduleConfig)
    }else{
      chooseModules.push(...answers.modules)
    }
    runDEV();
  });
}

init();
```
模板代码的简单示意：  
``` 
// 模板代码示意，router.config.template.ejs
import { RouteConfig } from 'vue-router';

<% chooseModules.forEach(function(item){%>
import <%=item %> from '<%=deelRouteName(item) %>';
<% }) %>
let routesConfig: Array<RouteConfig> = [];
/* eslint-disable */
  routesConfig = [
    <% chooseModules.forEach(function(item){%>
      <%=item %>,
    <% }) %>
  ]

export default routesConfig;
```
dev-server.js 的核心在于启动一个 inquirer 交互命令行服务，让用户选择需要构建的模块  
实现了分模块构建，按需进行依赖收集。如果单次开发只涉及固定的模块，单次项目冷启动的时间，可以从原本的 4min+ 下降到 18s 左右，而有缓存状态下二次构建 1 个模块，仅仅需要 4.5s，属于一个比较大的提升。  
受限于 Webpack 所使用的语言的性能瓶颈，要追求更快的构建性能，我们不可避免的需要把目光放在其他构建工具上。这里，我们的目光聚焦在了 Vite 与 esbuild 上。  

**使用 Vite 优化开发时构建**  
Vite，一个基于浏览器原生 ES 模块的开发服务器。利用浏览器去解析 imports，在服务器端按需编译返回，完全跳过了打包这个概念，服务器随起随用。同时不仅有 Vue 文件支持，还搞定了热更新，而且热更新的速度不会随着模块增多而变慢。  
由于 Vite 本身特性的限制，目前只适用于在开发阶段替代 Webpack。我们都知道 Vite 非常快，它主要快在什么地方？
1. 项目冷启动更快
2. 热更新更快

**Webpack 与 Vite 冷启动的区别**  
Webpack 启动时，从入口文件出发，调用所有配置的 Loader 对模块进行编译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件,这一过程是非常非常耗时的  
Vite 通过在一开始将应用中的模块区分为 依赖 和 源码 两类，改进了开发服务器启动时间。它快的核心在于两点：  
1. 使用 Go 语言的依赖预构建：Vite 将会使用 esbuild 进行预构建依赖。esbuild 使用 Go 编写，并且比以 JavaScript 编写的打包器预构建依赖快 10-100 倍。依赖预构建主要做了什么呢？
   1. 开发阶段中，Vite 的开发服务器将所有代码视为原生 ES 模块。因此，Vite 必须先将作为 CommonJS 或 UMD 发布的依赖项转换为 ESM
   2. Vite 将有许多内部模块的 ESM 依赖关系转换为单个模块，以提高后续页面加载性能。如果不编译，每个依赖包里面都可能含有多个其他的依赖，每个引入的依赖都会又一个请求，请求多了耗时就多
2. 按需编译返回：Vite 以 原生 ESM 方式提供源码。这实际上是让浏览器接管了打包程序的部分工作：Vite 只需要在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入代码，即只在当前屏幕上实际使用时才会被处理。

**Webpack 与 Vite 热更新的区别**  
使用 Vite 的另外一个大的好处在于，它的热更新也是非常迅速的。  
- Webpack-complier：Webpack 的编译器，将 Javascript 编译成 bundle（就是最终的输出文件）
- HMR Server：将热更新的文件输出给 HMR Runtime
- Bunble Server：提供文件在浏览器的访问，也就是我们平时能够正常通过 localhost 访问我们本地网站的原因
- HMR Runtime：开启了热更新的话，在打包阶段会被注入到浏览器中的 bundle.js，这样 bundle.js 就可以跟服务器建立连接，通常是使用 Websocket ，当收到服务器的更新指令的时候，就去更新文件的变化
- bundle.js：构建输出的文件

Webpack 热更新的大致原理是，文件经过 Webpack-complier 编译好后传输给 HMR Server，HMR Server 知道哪个资源 (模块) 发生了改变，并通知 HMR Runtime 有哪些变化，HMR Runtime 就会更新我们的代码，这样浏览器就会更新并且不需要刷新。  
Webpack 热更新机制主要耗时点在于，Webpack 的热更新会以当前修改的文件为入口重新 build 打包，所有涉及到的依赖也都会被重新加载一次  
Vite 号称 热更新的速度不会随着模块增多而变慢。它的主要优化点在哪呢？  
Vite 实现热更新的方式与 Webpack 大同小异，也通过创建 WebSocket 建立浏览器与服务器建立通信，通过监听文件的改变向客户端发出消息，客户端对应不同的文件进行不同的操作的更新  
Vite 通过 chokidar 来监听文件系统的变更，只用对发生变更的模块重新加载，只需要精确的使相关模块与其临近的 HMR 边界连接失效即可，这样 HMR 更新速度就不会因为应用体积的增加而变慢而 Webpack 还要经历一次打包构建。所以 HMR 场景下，Vite 表现也要好于 Webpack。  
通过不同的消息触发一些事件。做到浏览器端的即时热模块更换（热更新）。通过不同事件，触发更细粒度的更新（目前只有 Vue 和 JS，Vue 文件又包含了 template、script、style 的改动），做到只更新必须的文件，而不是全量进行更新。在些事件分别是：  
- connected: WebSocket 连接成功
- vue-reload: Vue 组件重新加载（当修改了 script 里的内容时）
- vue-rerender: Vue 组件重新渲染（当修改了 template 里的内容时）
- style-update: 样式更新
- style-remove: 样式移除
- js-update: js 文件更新
- full-reload: fallback 机制，网页重刷新

基于 Vue-cli 4 的 Vue2 项目改造，大致只需要：  
1. 安装 Vite
2. 配置 index.html（Vite 解析 <script type="module" src="..."> 标签指向源码）
3. 配置 vite.config.js
4. package.json 的 scripts 模块下增加启动命令 "vite": "vite"

以命令行方式运行 npm run vite时，Vite 会自动解析项目根目录下名为 vite.config.js 的文件，读取相应配置。而对于 vite.config.js 的配置，整体而言比较简单：  
1. Vite 提供了对 .scss, .sass, .less, 和 .stylus 文件的内置支持
2. 天然的对 TS 的支持，开箱即用
3. 基于 Vue2 的项目支持，可能不同的项目会遇到不同的问题，根据报错逐步调试即可，譬如通过一些官方插件兼容 .tsx、.jsx

遇到的一些小问题：  
1. tsx 中使用装饰器导致的编译问题，通过魔改了 @vitejs/plugin-vue-jsx，使其支持 Vue2 下的 jsx
2. 由于 Vite 仅支持 ESM 语法，需要将代码中的模块引入方式由 require 改为 import
3. Sass 预处理器无法正确解析样式中的 /deep/，可使用 ::v-deep 替换
4. 其他一些小问题，譬如 Webpack 环境变量的兼容，SVG iCON 的兼容

解决完上述的一些问题后，我们成功地将开发时基于 Webpack 的构建打包迁移到了 Vite，效果也非常惊人，全模块构建耗时只有 2.6s
至此，开发阶段的构建耗时从原本的 4.5min 优化到了 2.6s  

## 优化生产构建
生产发布是基于 GitLab 及 Jenkins 的完整 CI/CD 流。  
生产环境构建时间较长， build 平均耗时约 9 分钟，整体发布构建时长在 15 分钟左右，整体构建环节耗时过长， 效率低下，严重影响测试以及回滚 。  
Build base 阶段的优化，涉及到环境准备，镜像拉取，依赖的安装。前端能发挥的空间不大，这一块主要和 SRE 团队沟通，共同进行优化，可以做的有增加缓存处理、外挂文件系统、将依赖写进容器等方式。  
主要关注 Build Region 阶段，也就是核心关注如何减少 npm run build 的时间。  
一般而言， 代码编译  时间和代码规模正相关。根据以往优化经验，代码静态检查可能会占据比较多时间，目光锁定在 eslint-loader 上。
在生产构建阶段，eslint 提示信息价值不大，考虑在 build 阶段去除，步骤前置。可以通过 esbuild-loader 插件去替代非常耗时的 babel-loader、ts-loader 等 loader。  
整体优化方向就是：  
1. 改写打包脚本，引入 esbuild 插件
2. 优化构架逻辑，减少 build 阶段不必要的检查

**优化构架逻辑，减少 build 阶段不必要的检查**  
在生产构建阶段，eslint 提示信息价值不大，考虑在 build 阶段去除，步骤前置。  
比如在 git commit 的时候利用 lint-staged 及 git hook 做检查， 或者利用 CI 在 git merge 的时候加一条流水线任务，专门做静态检查。  
Gitlab CI 的代码：  
``` 
// .gitlab-ci.yml
stages:
  - eslint

eslint-job:
  image: node:14.13.0
  stage: eslint
  script:
    - npm run lint 
    - echo 'eslint success'
  retry: 1
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "test"'
```
通过 .gitlab-ci.yml 配置文件，指定固定的时机进行 lint 指令，前置步骤。  

**改写打包脚本，引入 esbuild 插件**  
这里借助的是 esbuild-loader，它把 esbuild 的能力包装成 Webpack 的 loader 来实现 Javascript、TypeScript、CSS 等资源的编译。以及提供更快的资源压缩方案。
接入起来也非常简单。项目是基于 Vue CLi 的，主要修改 vue.config.js，改造如下：  
``` 
// vue.config.js
const { ESBuildMinifyPlugin } = require('esbuild-loader');

module.exports = {
  // ...

  chainWebpack: (config) => {
    // 使用 esbuild 编译 js 文件
    const rule = config.module.rule('js');

    // 清理自带的 babel-loader
    rule.uses.clear();

    // 添加 esbuild-loader
    rule
      .use('esbuild-loader')
      .loader('esbuild-loader')
      .options({
        loader: 'ts', // 如果使用了 ts, 或者 vue 的 class 装饰器，则需要加上这个 option 配置， 否则会报错：ERROR: Unexpected "@"
        target: 'es2015',
        tsconfigRaw: require('./tsconfig.json')
      })

    // 删除底层 terser, 换用 esbuild-minimize-plugin
    config.optimization.minimizers.delete('terser');

    // 使用 esbuild 优化 css 压缩
    config.optimization
      .minimizer('esbuild')
      .use(ESBuildMinifyPlugin, [{ minify: true, css: true }]);
  }
}
```
移除 ESLint，以及接入 esbuild-loader 这一番组合拳打完，本地单次构建可以优化到 90 秒。  

**前端工程化的演进及后续规划**  
在项目体量差不多的情况下，他们的生产构建耗时（npm run build）在 2 分钟出头，细究其原因在于：  
1. 项目是 React + TSX，我这次优化的项目是 Vue，在文件的处理上就需要多过一层 vue-loader；
2. 项目采用了微前端，对项目对了拆分，主项目只需要加载基座相关的代码，子应用各自构建。需要构建的主应用代码量大大减少，这是主要原因；

还有许多可以尝试的方向，譬如：  
1. 对项目进行微前端拆分，将相对独立的模块拆解出来，做到独立部署
2. 基于 Jenkinks 构建时，在 Build Base 阶段优化的提升，譬如将构建流程前置，结合 CDN 做快速回滚，以及将依赖预置进 Docker 容器中，减少在容器中每次 npm install 时间的消耗等

原文: 
[冷启动 4min -> 2s 的构建优化，怎么做到的](https://mp.weixin.qq.com/s/biC7qsBEJYWzNEe33I3TOw)
