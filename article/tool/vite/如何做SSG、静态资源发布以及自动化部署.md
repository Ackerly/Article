# 如何做SSG、静态资源发布以及自动化部署
**运作流程**  
重构后，从开发到部署更新的运作流程图如下，日常只需要维护 GitHub 仓库的代码，其他的都是自动化完成。  
**重构的价值**  
1. 提前开荒 Vite 2.0 ，为公司后续的业务提前踩坑，可以为团队进行技术选型提供帮助，因为之前我在做 JSSDK、Vue Plugin 的时候，已经开始脱离 Webpack，用 Rollup 作为构建工具，而 Vite 正是基于 Rollup ，不仅构建速度非常快，而且也像 Webpack 一样提供了热更新，对于一线开发来说，体验上是非常好的，而且它还是 Vue 团队大力推广的新工具  
2. 了解一下当前的一些新生的前端工具，比如 UI 框架方面之前一直停留在适合 B 端产品的 Ant-Design、 Vuetify 、 饿了么等等，说实话我做 B 端产品的时候才会用，像 CSS 预处理器之前也一直停留在 Sass / Less / Stylus 三驾马车，这一次用上了 PostCSS Language + CSS Variable  
3. 多了解一下生产环境的服务端开发,公司业务几乎没有机会让自己实操服务端，所以大部分情况下都是在跑本机的 Server，很多场景是开发环境下遇不到的，要想进步，还是要多在生产环境磨练。  
4. 接触更多优秀的开源作品，比如代码语法高亮之前一直只知道 highlight.js,这一次是用了 prism[8] ，更小巧，颗粒度更细  
5. 享受从 0 到 1 搭建脚手架的一个过程，目前这个版本算是实现一个简易版的 VuePress ，但是如果一直使用开箱即用的 VuePress ，很多时候并没有想去了解那些功能是怎么实现的，或者用哪些工具可以实现想要的功能

## 技术栈的选择  
由于重构的最终目标还是要保持网站的 SEO 能力，所以肯定不能使用默认的 SPA 应用模式，要走服务端渲染，所以技术栈方面只需要考虑两条线：  
**基于 SSR**  
SSR ， Nuxt、Vapper等一些比较流行的开箱即用的 SSR 框架目前都还在弄 Vue 2.0，甚至部分框架看起来有点 “弃坑” 的趋势  
加上搞 SSR 的话，服务器成本比较高,那么退而求次就是上 SSG 。
**基于 SSG**  
SSG 也是有考虑过一些开箱即用的 SSG 框架，比如用的人最多的 Hexo,但是很多独立博客都是基于 Hexo 的，模板也千篇一律，缺乏个人特色。
Saber 还是以 Vue 2.0 为主，所以使用了VitePress，他们的新版本都是基于 Vue 3.0，而且已经可以用了  
**最终敲定**  
最终用到的核心技术是：  
- Vite 2.0 —— 超快的构建工具
- Vue 3.0 —— 更强大更灵活的 Vue
- SSG —— 服务端渲染方案，利于 SEO 进行内容收录
- PWA —— 构建离线应用

## 重构过程分析
**构建工具**
Vue-CLI对 Vue 3.0 的支持已经非常好了，之所以选择 Vite，一方面是它的构建速度真的比 Webpack要快好多，另一方面是，自从 Vue 3.0 推出以来， Vue 官方团队就一直在投入精力优化和宣传 Vite，尽管 1.0 版本的功能和生态不如人意，但超快的构建速度已经体现了出来。
而且在生态方面，Vite 2.0 的各种支持都算很完善了，不得不说整个春节期间，Vue 团队的人都在忙着给 Vite 2.0 干活   
**服务端渲染**  
个人博客之前一直选择用 WordPress ，一方面除了有 LNMP 一键部署等快速搭建方案，和各种各样的模板之外，主要也是归功于 WP 对 SEO 的支持也是非常好  
用 Vue 3.0 重新开发 SPA 应用肯定会丢失 SEO，所以才有了前面的 技术栈的选择，本次是通过 SSG 方案来落地服务端渲染。  
**项目架构规划**  
要考虑的一些东西:  
1. 对外展示的网站结构要保持不变，也就是原来的页面地址要尽量一样，避免用户访问不到原来的内容
2. 对实在不能保持原样的 URL ，或者要废弃的页面，需要做 301 重定向
3. 降低后续更新的构建和部署成本，尽量自动化，减少人工操作
4. 数据需要无缝迁移，不能有丢失
5. 减少服务器压力，把大部分资源消耗放在开源平台上（诸如 Github、jsdelivr CDN 等等）

**模板开发**  
基于 Vue 3.0 的项目，主要的模板肯定还是 Vue 文件，站点的主要结构、页面的布局、美化等等都是基于 .vue 文件，只需要按照原来的习惯，路由页面放在你的 src/views 文件夹下，组件模板放置于 src/components 下，就可以自动生成路由访问。  
同时也加入了 .md 文件的支持，用于书写 Markdown 格式的内容，日常记录博客会更方便，并且像 VuePress 那样，同时支持在 Markdown 里嵌套 Vue，让博客的定制更加灵活。  
整个项目的路由页面、组件结构，跟平时开发 Vue 项目是完全一样的，无缝切换。  
``` 
src 
├─components 
│ ├─Footer.vue 
│ └─Header.vue 
└─views 
│ ├─[page].vue
├─article 
│ └─rewrite-in-vite.md 
│ ├─about.md 
└─index.vue
```
几个非常方便的 Vite 插件：  
vite-plugin-pages：能够自动读取指定目录下的 Vue / Md 文件生成 Vue 路由，只需要管理好 views 文件夹的层级关系，无需再单独维护路由配置
vite-plugin-md：一个能让 Markdown 文件像 Vue 组件一样导入使用的插件，它也基于 markdown-it，支持进行一系列 md 生态扩展  
vite-plugin-components：可以像 VuePress 一样，无需 import，会自动根据组件的标签名去 components 目录下寻找组件
**样式处理器**  
有设计稿的时候通常借助 CSS 预处理器（目前常用 Stylus），借助他们的变量 、 嵌套书写，以及 Mixin 、 Extend 等功能，避免写原生 CSS 带来的烦恼。  
没有设计稿的时候，会用上 Ant Design 等 UI 框架来帮我减少页面设计上的一些时间浪费，但这些框架通常更适合用在 B 端产品  
去年底在知乎刷到过一篇 如何评价 CSS 框架 TailwindCSS？了解到一款全新的 CSS 框架 Tailwind CSS，乍一看很像是在开历史的倒车，回归原子类 className ，评价也是褒贬不一  
目前感受到的好处就是：  
> 延续 CSS 的属性命名，你需要什么属性自己放，也就是自己必须有一定的 CSS 基础，特别是在多端适配方面，不用担心框架用久了自己不会写 CSS 的问题

要实现一个容器内完全居中，手写 CSS 是：
``` 
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100px;
  height: 100px;
}
```
Tailwind CSS 的写法是：  
``` 
<div class="flex justify-center items-center w-40 h-40"></div>
```
写法跟你在 VSCode 里自动补全代码时，敲入的命令非常接近，不像传统的 UI 框架一样，你写个标签就自动生成按钮，都不知道它是怎么写出来的  
> 支持 CSS tree-shaking ，构建后的文件非常迷你

传统的 Atom CSS ，引入了就得整包引入，而 Tailwind 可以借助 PostCSS ，可以在最终项目构建的时候，抽离出我们用到的样式，用不到的会被直接扔掉。  
Tailwind核心样式文件在配置 Purge 之前构建出来大概有 6M 多，Purge 之后只有 24K ！  
> 可以组合使用，类似于 CSS 预处理器的 Extend

写一个通用的图片样式，让图片具备自适配不变型的效果，我只需要借助 @apply 像这样子：  
``` 
.img {
  @apply w-full h-full object-cover;
}
```
编译出来的效果  
``` 
img {
  width: 100%;
  height: 100%;
  -o-object-fit: cover;
  object-fit: cover;
}
```
> 支持目前主流的暗黑模式，通过 dark:xxxxx 的前缀就可以轻松定制两款皮肤

> 用了 Tailwind 之后，几乎可以不用写 Sass / Stylus 了，那么问题来了：如何弥补 CSS 预处理器提供的一些功能

借助 PostCSS Language和 CSS Variable，可以轻松的书写像 CSS 预处理一样的嵌套和变量
``` 
a {
  color: var(--fg-deeper);
  text-decoration: none;

  &:hover {
    border-bottom: 1px solid var(--fg-light);
  }
}
```
独立的文件使用 .postcss 或者 .pcss 作为文件后缀，在 Vue 组件里则使用 <style lang="postcss"></style> 来指定 PostCSS Language 。  
**SEO 优化**  
虽然前面的 服务端渲染帮我们解决了空 HTML 文档的问题，但要更好的进行 SEO 优化，还需要落实到具体的页面上去。  
比如页面的 title 、 description 、 keyword 等等，这里我是用到了以下两个工具来帮我实现每个页面的 TKD 定制.  
gray-matter：支持对 .md 文件的 TKD 优化，你可以在 Markdown 文件的最前面加入这样的代码，即可实现对页面展示对应的 TKD 信息。  
``` 
---
title: 这是页面的标题
desc: 这是页面的描述
keywords: 关键词1,关键词2,关键词3
---
 Markdown 内容…
```
@vueuse/head：可以让你在 .vue 文件里实现优化，在 Vue 组件里的 script 部分，写入以下的代码，就可以实现 TKD 信息的配置。  
``` 
import { useHead } from '@vueuse/head'

useHead({
  meta: [
    {
      name: 'title',
      content: '这是页面的标题'
    },
    {
      name: 'description',
      content: '这是页面的描述'
    },
    {
      name: 'keywords',
      content: '关键词1,关键词2,关键词3'
    }
  ]
})
```
还可以扩展更多的信息上去，具体都在各自对应的 Github 仓库的 README 里有详细的说明。  
当然，SEO 优化远远不止这一点，包括 robots 、 链接语义化 、减少死链 、 旧地址重定向等等  
**静态资源处理**  
静态资源指 js 、 css 、 img 这些资源,虽然 WordPress 虽然有配置 CDN 的插件，但是 CDN 平台诸如七牛、又拍云，免费额度只针对 http , 都是需要付费才可以使用 https  
改版后是把项目托管到了 Github ，先天优势存在，那么就要多考虑一下利用 Github 提供的免费服务了！  
开发过 NPM 包的同学，或者日常使用 NPM 插件比较细心的同学，应该能够发现发布在 NPM 上的包都自动部署到了 CDN 平台，诸如 jsdelivr 、 unpkg 、cdnjs 等等，那么 Github 和这些 CDN 能关联吗？答案是可以的，而且其中对国内访问速度最友好的 jsdelivr ，支持度最高！  
最后把所有静态资源都指向了 jsdelivr CDN ，它无需你自己再做任何部署工作，只需要把代码文件更新到你的 GitHub 仓库里，就会自动同步到 jsdelivr 。  
访问格式为在 jsdelivr CDN 官网有案例说明，更多用法可以查看官网的文档 Features - jsdelivr，为了避免项目源码过大，可以单独创建一个类似 assets-storage这样的仓库用来存储这些静态资源，在仓库的 README 也有简单介绍下如何引用 CDN 地址和清除 CDN 缓存。  
回到项目里，只需要在 vite.config.ts里修改 base 的路径即可  
``` 
export default defineConfig({
  base: isDev
    ? '/'
    : 'https://cdn.jsdelivr.net/gh/chengpeiquan/chengpeiquan.com@gh-pages/'
})
```
这种方式如果你用平时的命令行或者老乌龟界面工具来提交文件，始终还是比较麻烦，这里推荐一个现成的图床工具 PicGo ，支持多个平台的 CDN 服务，其中就有 Github 。  
**资源导出**  
资源导出主要是指原来的那些图片，WordPress 的上传资源都存放在 /wp-content/uploads/ 目录下，阿里云非常方便的就是，你可以连 SFTP 上去把这些文件直接拖下来就可以了  
重新传到 Github 上又非常简单，克隆你的仓库下来后，放到指定的文件夹里，重新提交就可以了  
**爬虫编写**  
这一部分主要针对原来的文章，虽然我之前的 WordPress 就开启了 Markdown 编辑器支持，但如 SEO 优化里提到的，缺少很多 TKD 信息配置，而且里面的图片地址也都要更换为 CDN 的路径，所以就算用现成工具去处理 HTML / XML 转 Markdown，都还要去补充这些信息，也比较繁琐。  
所以是借助了 Node 编写了个静态爬虫，在爬取过程中对一些内容进行追加、转换。  
**数据统计**  
百度统计：vue-baidu-analytics
友盟统计：vue-cnzz-analytics

**服务端开发**  
服务端之前是 WordPress 所依赖的 Nginx + PHP + MySQL ，这一次重构也把服务端直接换了，更换为 Node.JS + Express ，通过 PM2 守护进程来运行在阿里云。  
服务器系统是 CentOS 7,关键命令参考:  
1. 清除缓存然后升级系统和软件
``` 
sudo yum clean all sudo yum makecache sudo yum update sudo yum upgrade -y
```
2. 安装 NPM 并通过 stable 安装最新版本的 Node
``` 
sudo yum install npm sudo npm install -g n sudo n stable
```
3. 全局安装 yarn[
``` 
npm i -g yarn
```
4. 全局安装 pm2,这个是必须要装的，否则我们的终端一关，服务就停了，需要通过 PM2 来守护进程，当然也可以用 forever  
``` 
yarn global add pm2
```

注意：  
1. 因为服务端变了，如果原来有开启 HTTPS，记得重新配置你的 SSL 证书
2. 域名也要重新做 301 重定向（HTTP 强切 HTTPS ， WWW 强切无 3W 等
3. 检查之前是否有在推广的的链接挂掉了，也要重新 301 到新地址 （比如 RSS 源之前是 /feed/ ，现在是 /feed.xml）
4. 最重要的，配置上对路由 history 模式的支持

**自动化部署**  
代码托管在 GitHub 的好处就是 GitHub Actions 可以帮我们实现 CI / CD，通过配置分支的 push 或者 pull_request 等行为来实现自动触发项目的构建打包，并实现一键部署到阿里云服务器  
workflow 里所有以 secrets.XXXXXX 的格式均为仓库独立配置的密钥变量，在仓库的 settings > Actions secrets 里添加。  
关键环节:  
1. on 是指分支行为，配置了合并分支才会触发，因为平时都是托管在 develop 分支，包括未开发完毕的功能，写一半的文章草稿，只有确认可以发布的代码，才会合并到 main 进行更新
2. jobs 是触发自动打包 / 发布一系列行为的各种操作，从上到下按顺序处理，其中的 ACCESS_TOKEN 是 GitHub 的 Token，请来 Personal access tokens创建，创建后只会显示一次，请保存好，后面涉及到 Token 的地方可以重复使用同一个 Token，请勿泄露！
3. gh-pages 分支是打包完毕后的文件，推送到阿里云服务器的也是这个分支下的所有文件，之所以托管一份在 GitHub，是因为部署了 CDN 支持，JS / CSS 文件是需要读取这个分支的 CDN 文件
4. 部署到阿里云的环节，配置的 SERVER_SSH_KEY 是自己服务器的密钥对，如果是使用阿里云的 ECS ，可以参考 创建 SSH 密钥对， 创建后还需要绑定给实例才能激活生效，绑定操作请参考 绑定 SSH 密钥对
5. SERVER_IP 是自己服务器的公网 IP，这个其实可以不用配置为密钥变量，因为 ping 一下你的域名也知道是什么 IP ，只是因为有两台服务器，所以配置为变量可以方便的通过 SERVER_IP 和 SERVER_IP_TEST 去切换，其他变量其实也有一个 TEST 版本
6. 最后的 TARGET 是你在服务器上，node 服务器所安装的目录

**离线应用构建**  
使用 Vue-CLI 创建新项目的时候，可以了解到有一个选项是关于 PWA 的，Vite 官方团队也对 PWA 做了支持，通过 vite-plugin-pwa可以方便的实现一个离线应用的配置。  
需要特别注意的点：  
1. 因为使用了 CDN，所以 scope 和 manifest.start_url 选项需要显式指定，否则资源会读取出错
2. 基于上面提到的路径问题，从 v0.5.3 开始，配置 CDN 的同时，也需要显式指定 base 选项，避免出现 404

原文: 
[重构于 Vite：我如何做 SSG、静态资源发布以及自动化部署](https://mp.weixin.qq.com/s/fpc1zTvPTMJ4UiGFrpHxHw)
