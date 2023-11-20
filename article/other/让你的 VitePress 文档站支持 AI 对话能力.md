# 让你的 VitePress 文档站支持 AI 对话能力
## 技术原理
第一部分：一个将文档提交到数据库的 CLI 工具，只需要在项目中配置下 documate.json 就行了。  
document.json 配置文件非常简单，只有三个配置项：  
- root: 描述使用项目的哪个目录下的文件生成内容，默认是根目录
- include: 描述哪些是需要处理的文档文件，默认是 Markdown 文本。
- backend: 指定上传文档内容到后端保存的接口，OpenAI 根据这些内容来提供回答

第二部分：一个封装好的拿来即用的问答 UI 组件，直接 import 组件就可以使用，目前提供了 Vue 组件以及不依赖任何框架的原生 JavaScript 的 UI。看 issue 规划，React 版本社区已经在支持中了，相信很快就会发布；  
第三部分：一个提供问答服务、可一键完成部署的 AI Ask Server，可以在 AirCode 直接完成部署，也可以在自己的服务器上部署；  
问答服务有了，UI 组件有了，数据也有了，那就可以尝试着玩耍了，整个配置过程大概可以在 15min 内完成。  

**如何接入**  
你想创建一个全新的 VitePress 项目并包含 AI 对话能力，可以使用下面的命令：  
``` 
npm create documate@latest --template vitepress
```
创建完成后可直接跳到第 3 步「构建上传和搜索后端 API」继续配置。  
如果要给已有的 VitePress 项目添加 AI 对话能力，则按照以下步骤进行。  
_1. 初始化_  
在你的 VitePress 项目根目录下使用以下命令进行初始化：  
``` 
 npx @documate/documate init --framework vue
```
该命令会创建一个 documate.json 配置文件。  
``` 
{
   "root": ".",
   "include": [
     "**/*.md"
   ],
   "backend": ""
 }
```
并且添加了一个 documate:upload 命令用于上传文档生成知识库，后面会介绍到具体用法。  
``` 
{
   "scripts": {
     "docs:dev": "vitepress dev",
     "docs:build": "vitepress build",
     "docs:preview": "vitepress preview",
     "documate:upload": "documate upload"
   },
   "dependencies": {
     "@documate/vue": "^0.2.3"
   },
   "devDependencies": {
     "@documate/documate": "^0.1.0"
   }
 }
```
_2. 给项目添加 UI 入口_  
在文件 .vitepress/theme/index.js 中添加如下代码，如果没有则需要先手动创建这个文件。VitePress 在 Extending the Default Theme 文档中介绍了如何定制自己的主题。  
``` 
import { h } from 'vue'
 import DefaultTheme from 'vitepress/theme'

 // Load component and style
 import Documate from '@documate/vue'
 import '@documate/vue/dist/style.css'

 export default {
   ...DefaultTheme,
   // Add Documate UI to the slot
   Layout: h(DefaultTheme.Layout, null, {
     'nav-bar-content-before': () => h(Documate, {
       endpoint: '',
     }),
   }),
 }
```
上面的代码会在导航栏中添加一个 AI 对话框的 UI。在本地使用 npm run docs:dev 启动服务后，你可以在左上角找到 Ask AI 的按钮。如果没有看到 Ask AI 按钮，可以检查下上面的代码是否都正确添加，并确保从 @documate/vue/dist/style.css 导入了 CSS 样式文件。  
_3. 构建上传和搜索后端 API_  
Documate 的后端代码用于上传文档内容生成知识库，以及接收用户的问题并返回流式的回答。  
进入 GitHub 中的 backend 文件夹，点击其中的「Deploy to AirCode」，快速复制并部署一份自己的后端代码。  
如果是第一次使用 AirCode（一个在线编写和部署 Node.js 应用的平台），会被重定向到登录页面。建议选择 GitHub 登录，会更快一些。  
创建应用之后，在 Envrionments 标签页中设置 OPENAI_API_KEY 环境变量为你的 OpenAI Key 值。你可以在 OpenAI 控制台 中获取到 API Key。  
点击顶部栏上的「Deploy」按钮，将所有的函数部署到线上，部署成功后可以得到每个函数的调用 URL。  
这里会使用到 upload.js 和 ask.js 两个函数，一个用于文档内容上传，另一个用于回答问题。  
_4. 设置 API 接口_  
在 AirCode Dashboard 中，选择部署后的 upload.js 文件，复制 URL 并添加到 documate.json 的 backend 字段中：  
``` 
// documate.json
 {
   "root": ".",
   "include": [ "**/*.md" ],
   "backend": "替换为你的 upload.js 的 URL"
 }
```
同上操作，在 AirCode 中选择已部署的 ask.js 文件，复制调用 URL，修改 .vitepress/theme/index.js 文件 endpoint 值。  
``` 
// .vitepress/theme/index.js
 import { h } from 'vue'
 import DefaultTheme from 'vitepress/theme'
 import Documate from '@documate/vue'
 import '@documate/vue/dist/style.css'

 export default {
   ...DefaultTheme,
   Layout: h(DefaultTheme.Layout, null, {
     'nav-bar-content-before': () => h(Documate, {
         // Replace the URL with your own one
         endpoint: '替换为你的 ask.js 的 URL',
       },
     ),
   }),
 }
```
_5. 运行项目_  
通过下面的命令将内容上传到后端，生成文档知识库:  
``` 
npm run documate:upload
```
命令完成后，本地启动项目，点击左上角 Ask AI 按钮，在弹出对话框后输入问题就可以得到基于你文档内容的回答。  
``` 
npm run docs:dev
```



[原文](https://mp.weixin.qq.com/s/OOZQvJnqiunW1DKAMyoOzg)
