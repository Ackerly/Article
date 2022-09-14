# 开发模式 "Development Mode" 是如何工作的
如果你的代码库即便是稍许有些复杂，你可能已经采取了某种办法，针对开发和生产环境分别进行打包，从而于不同环境运行不同的代码。  
针对开发和生产模式分别打包并运行不同的代码，这样的做法很有用。在开发模式下，React 包含了很多以帮助你发现潜在 bug 的警告 （warnings）。然而，用于检查这些错误的那部分代码往往会增加程序包的体积、拖慢应用运行速度。  
在开发模式下这个 “缓慢” 尚可接受。实际上，在开发阶段使程序运行慢一些还或许有一点好处，那就是，它部分地中和了开发机（往往很快）与多数用户机（较慢）的性能差异。  
而在生产模式，我们则不愿意付出这个性能代价。因此，我们在生产模式下忽略掉这些检查。这是怎么实现的呢？  
如何使开发模式下运行的代码异乎生产环境，确切的方式取决于你的脚本构建管道（build pipeline）（以及你是否设有这样的构建管道）。在 Facebook 它看起来像这样：  
``` 
if (__DEV__) {
   doSomethingDev();
 } else {
   doSomethingProd();
 }
```
__DEV__ 不是一个真实的变量。它是一个常量， 在各个模块（modules）穿连到一起后被替换。结果就是如下面这样：  
``` 
// In development:
 if (true) {
   doSomethingDev(); //
 } else {
   doSomethingProd();
 }

 // In production:
 if (false) {
   doSomethingDev();
 } else {
   doSomethingProd(); //
```
在生产模式下，使用压缩工具（例如：terser）对代码进行处理。多数的 JavaScript 压缩工具部分地实现了死码消除（dead code elimination），比如，去掉 if (false) 条件分支。因此在生产模式下，应该只能看到：  
``` 
// In production (after minification):
 doSomethingProd();
```
然而，如果你使用当下流行的 webpack 打包工具，可能就未使用 __DEV__这个魔术常量（magic constant），这时你可以遵循另一些套路。例如，通常可表达相同意思的方式像是这样：  
``` 
if (process.env.NODE_ENV !== 'production') {
   doSomethingDev();
 } else {
   doSomethingProd();
 }
```
这正是 React 和 Vue 前端库被打包的时候所采用的表达方式。（它分别提供 development 和 production 两个单文件构建版本，分别为 .js 和 .min.js）  
这个惯常的做法源自 Node.js。在 Node.js 中，有一个全局变量 process ，它在属性 process.env 对象中暴露了系统的环境变量。然而，当你注意在前端代码中观察这种用法，会发现并没有真正引入 process 这个变量。  
实际上，在构建的时候，process.env.NODE_ENV 这整个表达式会被一个文本替换，就像神奇的 __DEV__ 变量一样：  
``` 
// In development:
 if ('development' !== 'production') { // true
   doSomethingDev(); // 
 } else {
   doSomethingProd();
 }

 // In production:
 if ('production' !== 'production') { // false
   doSomethingDev();
 } else {
   doSomethingProd(); // 
 }
```
由于这整个表达式是一个常量（'production' !== 'production' 一定是 false），代码压缩工具也会移除条件的否定分支。  
``` 
// In production (after minification):
 doSomethingProd();
```
对于复杂的表达式，这种的方式将不会奏效：  
```  
let mode = 'production';
 if (mode !== 'production') {
   // not guaranteed to be eliminated
 }
```
JavaScript 因为是动态语言，静态分析工具不会那么智能。当它看到是变量 mode ，而不是一个静态的表达式，比如 false 的或者 'production' !== 'production'，它通常无视之。  
类似，JavaScript 中的死码消除对于使用 import 而产生的跨模块边界的情形，不会很好地起作用：  
``` 
// not guaranteed to be eliminated
 import {someFunc} from 'some-module';

 if (false) {
   someFunc();
 }
```
因此，你需要编写一些具体的代码，以使条件是绝对静态的表达，并且确保条件下的全部代码是你想消除的。  
打包工具需要对 process.env.NODE_ENV 进行替换，并且需要明确你要在哪种模式下将其构建到项目中。  
几年之前，人们往往忘记配置环境。你可能经常看到一些处于开发模式下的项目被部署到生产环境上。这非常糟糕，因为，它会让网站加载和运行得更慢。  
在过去的两年，这种状况得到极大改善。例如，webpack 添加了一个方便的 mode 选项，以替代之前对 process.env.NODE_ENV 的手动配置。React DevTools 现在也会在开发模式下将图标显示为红色，以使更加显眼、甚至用于做相关报告。  
直接使用像 Create React App、Next/Nuxt、Vue CLI、Gatsby 等等工具来初始化项目的话，开发与生产构建分别交由两个命令执行（例如：npm start 和 npm run build），这使得用户对此两种模式更加不易混淆。尤其，只能生产构建才能部署，因此开发者不会再犯错误。  
总有声音认为 production 模式应该是默认模式，而 development 模式则应为可配置项。个人来说，我不认为这个观点有任何说服力。从 development 模式下的 warnings 中获益的，往往是那些使用 xx 库的新手。他们不知道需要将环境切换到开发模式，这将导致那些 warnings 能检测到的大量潜在的 bug 被遗漏。  
性能问题很糟糕。但是将问题重重的使用体验抛给终端用户同样糟糕。例如，React key warning 帮助我们避免掉一些 bug，比如：将消息发送给错误的人或者购买到错误的产品。关闭这个 warning 对你和你的用户都存在极大的风险。倘若它默认为关闭的状态，那么当你开启它的时候，已经积攒了大量的 warnings 以待清理！因此大多数人会将其切换为开启状态。这也就是为什么此种检测当从最初即被开启，而非日后为之  
最后，即使开发模式下的 warnings 为可选配置，且开发者知道在开发时需要早早地开启它们，我们还是会回到最初的那个问题。有些人将会把开发模式下的版本部署到生产环境。  
赖那些可根据所处阶段是调试或部署，来显示和使用正确模式的工具。几十年来，除 web 浏览器之外的其它环境（移动、PC 或者服务器）都有各自的方式加载和区分开发（development）和生产（production）的构建。  
是时候为 JavaScript 运行环境提供一个首要配置项，来区分 development 和 production 模式，而不是各个库采用和依赖一些临时规范。  
来看看这段代码：  
``` 
 if (process.env.NODE_ENV !== 'production') {
   doSomethingDev();
 } else {
   doSomethingProd();
 }
```
如果在前端代码中不存在 process 这个对象，为什么像 React 和 Vue 这样的库在 npm 构建的时候需要依赖它呢？  
在浏览器中使用 <script> 加载的 React 和 Vue 的构建包并不依赖这个。但是，你必须自己来选择是使用 development 模式下的构建包 .js 还是 production 模式下的构建包 .min.js。下边的内容旨在探讨使用打包工具（webpack、rollup），基于 ES6 的 import 模块加载规范，加载 React 或 Vue 库时的情况。  
就像编程中的众多问题，这个规范（convention）的形成有历史原因。我们都在使用这个，只是因为现在很多工具在遵循它。如果换作其它的方式，则其代价不小且无所益处。  
那么这背后的历史如何呢？  
在 import 和 export 语法得以标准化之前的数年，已经存在了很多完整的方式来表达模块之间的关系。Node.js 极大推广了 require() 和 module.exports，这就是 CommonJS 规范。  
最初，往 npm 仓库发布的代码仅提供给 Node.js 来使用。Express曾是最为流行的基于 NodeJs 的服务端框架，它使用了 NODE_ENV 环境变量来开启 production 模式。其它一些 npm 包也采取了同样的规范。  
早期的 JavaScript 打包工具，像 browserify ，想将 npm 仓库中的代码应用到前端项目中。是的，那之前，没有人将 npm 包用于前端开发！你能想象吗？）因此，他们把 NodeJs 生态下的这个规范扩展到了前端。  
原先的 “envify” 代码变换（transform）发布于 2013 年。React 在那个时候做了开源，且 npm 搭配 browserify 看起来是最优秀的前端（基于 CommonJS 加载规范）打包解决方案。  
React 从一开始就提供了 npm 构建（另加独立文件构建，<script> 标签访问）。随着 React 日益流行，借 CommonJs 规范进行 JavaScript 模块化的实践和使用 npm 发布前端代码亦得以风行。  
React 在 production 模式下，需要移除仅用于 development 阶段时的那部分代码。Browserify 已经提供解决此类问题的方案，因此 React 也采用了使用 process.env.NODE_ENV 作为环境变量的打包（npm）规范。往后，很多其它的工具和库，包括 webpack 和 VUE，都是如此。  
截止 2019 年，browserify 已经失去了一些关注度。然而，构建阶段替换 process.env.NODE_ENV 以 'development' 或 'production'，作为一个规范流行了起来。  
为什么 React 在 GitHub 上的源代码中，你可看到 __DEV__ 用于魔术变量。但是在 npm 仓库中 React 使用的是 process.env.NODE_ENV。  
以往，我们曾在源码中使用 __DEV__ 来适应 Facebook 的源码（内部的规范）。很长一段时间，React 直接被拷贝到 Facebook 的代码库中，因此，它需要维持一致的规则。对于 npm 代码库，我们在构建发布之前，会将 __DEV__ 检查替换为 process.env.NODE_ENV !== 'production' 文本表达。  
这种依赖 Node.js 包规范的代码形式在 npm 上能正常工作，但破坏了 Facebook 的规则，或者反过来说也是。  
React 16 以来，改变了这种方式。为每一种环境构建文件包（包括支持 <script> 标签引入、npm 和 Facebook 的内部代码规范）。因此，甚至 npm 仓库上的 CommonJs 代码也会提前为 development 和 production 模式分别编译独立文件。  
当 React 源代码里说 if (__DEV__)，实际上生成了两个文件。其中一个已经被预编译为 __DEV__ = true，另外一个，则被预编译为 __DEV__ = false。在入口点判断需要输出哪个文件包。  
例如：  
``` 
 if (process.env.NODE_ENV === 'production') {
   module.exports = require('./cjs/react.production.min.js');
 } else {
   module.exports = require('./cjs/react.development.js');
 }

```
打包工具将 ‘development’ 或 ‘production’ 以字符串形式插入到代码中做环境判断的地方，继而，代码压缩工具抛开 development-only 的那些依赖（require）  
react.production.min.js 和 react.development.js 二者都不会再有基于 process.env.NODE_ENV 的环境检查了。这非常棒，因为在 Node.js 环境中运行的时候，访问 process.env 会导致系统变慢。提前为两种模式编译好文件包，也能让更加一致地来优化文件大小，而不用考虑使用何种打包和压缩工具。  



参考:  
[开发模式 "Development Mode" 是如何工作的](https://mp.weixin.qq.com/s/81L1RRiCdqGMJmk-iUZldg)
