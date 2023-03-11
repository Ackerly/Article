# 了解Next.js
## Next.js 可以带给我们什么？
Next.js 是一个 React web 应用框架，主要做的事情有以下几点：  
1. 完善的工程化机制
2. 良好的开发和构建性能
3. 智能文件路由系统
4. 多种渲染模式来保证页面性能体验
5. 可扩展配置
6. 提供其他多方面性能优化方案
7. 提供性能数据，让开发者更好的分析性能。
8. 提供的其他常用功能或者扩展，比如使用 mdx 来编写页面的功能等等

**完善的工程化机制**  
不需要自己去配置 webpack 方案，它已经内置了以下工程化基础：  
- babel 内置，支持JS代码向后兼容
- postcss 内置，支持CSS代码向后兼容
- browserslist 支持配置兼容的浏览器信息，配合 babel 和 postcss 工作。
- TypeScript 可选择使用，保证代码的质量，以及可阅读性和可维护性
- eslint 可选择使用，检测代码格式，可自定义规则。vscode 编写代码，或者build打包时都会有提示
- prettier 可通过扩展使用，格式化代码，可自定义规则
- css modules 内置
- css-in-js 可扩展使用
- tailwind css 可扩展使用

也做了一些打包优化功能：  
- tree shaking
- 代码压缩
- 页面自动静态化
- 按需打包第三方 es 包（通过设置 transpilePackages 属性，让部分包可以被 next-babel 打包）
- 异步动态加载组件，和 React.lazy 功能一样，只不过实现得更早

**良好的开发和构建性能**  
开发模式 next.js 都是动态编译，启动的时候并不会去编译页面，访问才去动态编译，而且因为是多页应用，页面非共用部分互不影响，不会因为工程的业务模块变多而导致页面访问变慢。  
构建性能也很快，十几个页面，全部打包只用了 10.52 秒，有两个页面还动态请求了接口，不然还会更短一些。  

**智能文件路由系统**  
写在 pages 目录或者 src/pages 下的 js 文件都会被认为是页面，也会当成页面来打包，路由定义了一套动态路由[的规则  
只要是特定目录下的 js 文件都会当成页面，这样会造成不经意把非页面的 js 文件也打包成了页面，这样肯定是不太好的，因此 Next.js 13 才会推广出 app 目录下的新模式，但还不太稳定，有些功能还没实现，因此如果在老模式下还是需要注意，可以这样来处理文件结构：  
``` 
./src
├── common // 公共部分
├── views // 页面内容，所有代码都写在这个文件
│   └── home // 首页
│       ├── index.tsx
│       ├── List.tsx
│       ├── util.ts
│       └── style.module.css
├── pages
│   ├── _app.tsx
│   ├── _document.tsx
│   ├── api
│   └── home.tsx // 只对 src/views/home 的导出进行转发
└── styles
    ├── Home.module.css
    └── globals.css
```
智能文件路由系统虽然有一点缺陷，但优点更多，它也是 Next.js 最重要的特性之一，因为它在一定程度上规范了代码组织结构，这也是比较重要的。  

### 多种渲染模式来保证页面性能体验
**可扩展配置**  
Next.js 的可配置性真的是一个很强大的特色，它准备了一套默认就很好用的默认配置，但这些配置基本上用户都可以 完全 控制它（完全做一个保留，但大型工程基本上都是可以支撑的）。  
分析一下它的可配置功能
- 配置文件 next.config.js 中暴露了 webpack 实例，因此你可以完全控制 webpack
- 配置文件 next.config.js 中支持配置自定义配置，你可以把一些公用的不变的配置写在 serverRuntimeConfig 或者 publicRuntimeConfig 中，前者只会出现在服务端，后者会暴露到客户端。
- 可 自定义 server，你可以在启动服务的时候做一些自己想要做的处理，比如 node.js 性能监控等等
  不自定义 server ，也可以使用它提供的 middreware机制来拦截请求或者校验权限等事项
- 自定义 APP，也就是 _app.js，它用于处理多个页面公共部分
- 自定义 Document，也就是_document.js，用于自定义配置 html 生成内容，比如插入 Google 分析脚本。  
- 自定义错误界面也就是 404 或者 500 错误状态的页面
- 可自定义 `babel` 和 `postcss`等工程化规则配置

**提供其他多方面性能优化方案**  
Next.js 除了页面默认静态化（不使用getServerSideProps/getStaticPath/getInitialProps），还提供了其他方面的优化方案：  
- 图片优化：大部分页面基本上都会有或多或少的图片，图片往往是影响页面性能体验的重要因素之一，因为现在一张图片的体积很可能比页面的所有代码体积还打，因此关注性能就一定要关注图片
- 字体优化：字体文件一般也比较大，这一块还不是很了解，因为时间问题，后续了解完再更新一下。

**提供性能数据**  
Next.js 提供了获取应用性能数据的方法 reportWebVitals，只能在 App 组件中使用。  
``` 
// _app.tsx
export function reportWebVitals(metric: NextWebVitalsMetric) {
  console.log(metric)
}
```

**提供的其他常用功能或扩展**  
其他功能：
- `API Routes` ，Next.js 支持在 pages/api 目录下编写接口，可通过接口去实现 ISR 增量静态化功能，前端用于编写 BFF 接口应该也是一个不错的方案，但注意不能在 getStaticProps/getStaticPaths 中去请求，打包的时候请求不了
- `next/amp`: 用于支持开发 AMP 应用

扩展：
- `@next/mdx`: 用于支持使用 mdx 来编写页面，也就是如果要开发一个 md 的文档工程，直接接入它即可

**谈谈 React 单页应用**  
用过 React 的应该都知道官方提供了一个 Create React App 脚手架，可以很方便的搭建一个单页应用，但是其实真的不太好用（我不知道大家有没有这种感觉），因此我们可能还要去社区找一些完整的模板，或者新建以后，我们还要去新增各种配置和规划页面结构规范，比如路由、redux、请求封装，不能做到真正的开箱即用。而且两年前就已经传出不怎么维护的消息，将近一年没有新版本了。  
上一次版本还是在去年4月份，然后在官方github 论坛上，很多都在说 CRA 已经被放弃了。  
再加上 React 官方文档现在其实也在推荐其他的搭建方案，比如 Next.js 和 Gatsby  
其实 CRA 放弃没放弃，关系也不是特别大，反正也可以暴露了 webpack 配置，但需要自己定制的内容太多，而且并没不能完全的去体现出 React 的能力，比如不支持服务端渲染，也不支持 React 18 中的 Serevr Component/Client Component/Shared Component 等等新概念，因此 CAR 还是会逐渐被 Next.js 或者 Gatsby.js 给逐渐取代。  
CAR 的结果如此，后续的前端开发的方向应该会更偏向 Next.js 或者 Gatsby.js，不过这些主要是在C端或者B工程的市场会表现更好，在不那么重视首屏性能的管理端应用来说，影响不会那么大。  



原文:  
[Next.js了解篇｜一文带你梳理清楚 Next.js 的功能](https://mp.weixin.qq.com/s/xeRYh1sbt3iW0uF84qDgpw)
