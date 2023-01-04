# 打包JavaScript库的现代化指南
**输出 esm、cjs 和 umd 格式**  
esm 是 “EcmaScript module” 的缩写。  
cjs 是 “CommonJS module” 的缩写。  
umd 是 “Universal Module Definition” 的缩写，它可以在 <script> 标签中执行、被 CommonJS 模块加载器加载、被 AMD 模块加载器加载。  
esm 被认为是 “未来”，但 cjs 仍然在社区和生态系统中占有重要地位。esm 对打包工具来说更容易正确地进行 treeshaking，因此对于库来说，拥有这种格式很重要。或许在将来的某一天，你的库只需要输出 esm。  
umd 已经与 CommonJS 模块加载器兼容 —— 所以为什么还要同时具备 cjs 和 umd 输出呢？一个原因是，与 umd 文件相比，CommonJS 文件在对依赖进行条件导入时通常表现更好；例如：  
``` 
if (process.env.NODE_ENV === "production") {
   module.exports = require("my-lib.production.js");
 } else {
   module.exports = require("my-lib.development.js");
 }
```
上面的例子，当使用 CommonJS 模块时，只会引入 production 或 development 包中的一个。但是，对于 UMD 模块，最终可能会将两个包全部引入。  
最后还需要注意是，开发者可能会在其应用中同时使用 cjs 和 esm，发生双包危险。dual package hazard 一文介绍了一些缓解该问题的方法，利用 package.json#exports 进行 package exports 也可以帮助防止这种情况的发生。  
**输出多文件**  
通过保留文件结构更好地支持 treeshaking  
如果你对你的库使用了打包工具或编译器，可以对其进行配置以保留源文件目录结构。这样可以更容易地对特定文件进行 side effects 标记，有助于开发者的打包工具进行 threeshaking。  
一个例外是，如果你要创建一个不依赖任何打包工具可以直接在浏览器中使用的产出（通常是 umd 格式，但也可能是现代的 esm 格式）。在这种情况下，最好让浏览器请求一个大文件，而不是请求多个小文件。此外，你应该进行代码压缩并为其创建 sourcemap。  

**要不要压缩代码**  
你可以将一些层面的代码压缩应用到你的库中，这取决于你对你的代码最终通过开发者的打包工具后的大小的追求程度。  
例如，大多数编译器已经配置了删除空白符等其他简单的优化，即使是来自 NPM 模块的代码（在这里指的是你的库）。使用 terser —— 一个流行的 JavaScript 代码压缩工具 —— 这类压缩工具可以将包的最终大小减少 95%。在某些情况下，你可能会对这些优化感到满意，且不需要你来付出任何努力。  
但如果在发布前对你的库进行代码压缩，这可以得到一些额外的好处，但需要深入了解压缩工具的配置和副作用。压缩工具通常不会将这类压缩用于 NPM 模块，因此，如果你不自己来做的话，你会错过这些节省。  
最后，如果你正创建一个不依赖任何打包工具可以直接在浏览器中使用的产出（通常是 umd 格式，但也可以是现代的 esm 格式）。在这种情况下，你应该对代码进行压缩，并创建 sourcemap，并输出到一个单文件。  
**创建 sourcemap**  
对源代码进行任何形式的编译，都将导致未来某个异常的位置，无法与源码对应起来。为了帮助未来的自己，创建 sourcemap，即使只进行了很少的编译工作。  

**创建 TypeScript 类型**  
随着使用 TypeScript 的开发者数量不断增长，将类型内置到你的库中将有助于改善开发体验 (DX)。此外，不使用 TypeScript 的开发者在使用支持类型的编辑器（例如 VSCode，它使用类型来支持其 Intellisense 功能）时也会获得更好的 DX。  
但是，创建类型并不意味着你必须使用 TypeScript 来编写你的库。  
一种选择是继续在源代码中使用 JavaScript，然后通过 JSDoc 注释来支持类型。然后，你可以将 TypeScript 配置为仅从你的 JavaScript 源代码中构建类型文件。  
另一种选择是直接在 index.d.ts 文件中编写 TypeScript 类型文件。  
获得类型文件后，请确保设置了 package.json#exports 和 package.json#types 字段.  

**外置框架**  
不要将 React、Vue 等框架打包在你的库中  
当构建的库依赖某个框架（例如 React、Vue 等），或是作为另一个库的插件，你可能需要将框架配置到 “externals” 中。这可以使你的库引用这个框架，但不会将其打包到最终的产出中。这会避免产生一些 bug，并减少库的体积。  
你应该还需要将框架添加到库的 package.json 的 peer dependencies 中，这将帮助开发者发现你依赖于某个框架。  

**面向现代浏览器**  
使用现代的新特性，如果有需要，让开发者支持旧的浏览器这篇 web.dev 上的文章提供了一个很好的案例，并提供了相关的指导原则：  
- 当使用你的库时，能够让开发者去支持老版本的浏览器
- 输出多个产出来支持不同版本的浏览器

举个例子，如果你使用 TypeScript，你可以创建两个版本的包代码：  
- 通过在 tsconfig.json 中设置 "target"="esnext"，生成一个用现代 JavaScript 的 esm 版本
- 通过在 tsconfig.json 中设置 "target"="es5" 生成一个兼容低版本 JavaScript 的 umd 版本

有了这些设置，大多数用户将获得现代版本的代码，但那些使用老的打包工具配置或使用 <script> 加载代码的用户，将获得进行了额外编译来支持老版本浏览器的版本。  

**必要的编译**  
编译 TypeScript、将 JSX 转换为函数调用  
如果库的源码是需要进行编译的形式，如 TypeScript、React 或 Vue 组件等，那么你库需要输出的是编译后的代码。  
例如：
- 你的 TypeScript 代码应该输出为 JavaScript
- 你的 React 组件，例如 <Example />，应该在输出中使用 jsx() 或 createElement() 来替换 JSX 语法

**维护 changelog**  
只要能让开发者了解到有哪些变更和对他们的影响，至于是通过自动化工具还是通过亲自动手的方式来处理，这都无关紧要。理想情况下，库的每次版本变更都应该在 changelog 中进行相应的更新。  

**拆分出你的 CSS 文件**  
如果你正在创建一个 CSS 库（如 Bootstrap、Tailwind 等），最简单的方式就是提供单一文件，包含库的所有功能。然而，在这种情况下，你的 CSS 产出最终可能会变得很大，影响开发者网站的性能。为了避免这种情况，库通常会提供自定义生成 CSS 产出的功能，让产出中只包含开发者正在使用的必要 CSS（例如，参考 Bootstrap 和 Tailwind 是怎么做的）。  
如果 CSS 只是你的库的一部分（例如，具有默认样式的组件库），那么最好将 CSS 按组件分离单独构建产出，在使用相应的组件时按需导入。这方面的一个例子是 react-component。  

**配置 package.json**  
_设置 name 字段_  
name 字段将决定你的包在 npm 上的名字，开发者可以通过这个名字去安装并使用你的库。  
注意，库的命名是有限制的，如果你的代码库属于某个组织，你还可以创建一个命名空间。  
name 和 version 的组合为库每次迭代创建一个唯一标识。  

_设置 version 字段_  
通过更改 version 来对你的库发布更新  
name 和 version 的组合为你的库在 npm 上创建一个唯一标识。当你更新库中的代码时，你可以更新 version 字段并发布以允许开发者获取该新代码。  
推荐使用 semver 版本控制策略，但要注意的是有些库选择 calver 或使用他们自己特有的版本控制策略。无论你选择使用哪种策略，都应该记录下来，以便开发者了解你的库是如何进行版本控制的。  
还应该在 changelog 中记录你的更改。  

_定义你的 exports_  
exports 为你的库定义公共 API  
package.json 中的 exports 字段 - 有时被称为 “package exports” - 是一个非常有用的补充，尽管它确实引入了一些复杂性。它做的最重要的两件事是：  
- 定义哪些东西可以从你的库中导入，哪些则不可以，以及可导入的内容的名字。如果没有在 exports 中被列出，那么开发者就不可以 import 或 require 它们。换句话说，exports 的表现像是给你的库用户查看的公共 API，帮助定义哪些是外部的哪些是内部的。
- 允许你根据不同的条件（你可以定义）去选择那个文件是被导入的，例如 “文件是被 import 还是被 require？开发人员需要的是 development 版本的库还是 production 版本等等。

关于这部分的内容 NodeJS 团队和 Webpack 团队提供了一些很优秀的文档。在此列出一个涵盖大部分常见场景的例子：
``` 
{
   "exports": {
     ".": {
       "types": "index.d.ts",
       "module": "index.js",
       "import": "index.js",
       "require": "index.cjs",
       "default": "index.js"
     },
     "./package.json": "./package.json"
   }
 }
```
了解这些字段的含义以及选择这个例子的原因：  
- "."：表示你的库的默认入口
- 解析过程是从上往下的，并在找到匹配的字段后立即停止；所以入口的顺序是非常重要的
- types：字段应始终放在第一位，帮助 TypeScript 查找类型文件
- module：是一个 “非官方” 字段，它被 Webpack 和 Rollup 等打包工具所支持。它应该被放在 import 和 require 之前，并且指向 esm 格式的产出 -- 如果你的源代码是纯 esm 的，它也可以指向你的源代码。正如在格式部分中指出的那样，它旨在帮助打包工具只包含你的库的一个副本，无论它是通过 import：还是 require 方式引入的。
- import：用于当有人通过 import 使用你的库时
- require：用于当有人通过 require 使用你的库时
- default：字段用于兜底，在没有任何条件匹配时使用。虽然目前可能并不会匹配到它，但为了面对 “未知的未来场景”，使用它是好的

当一个打包工具或者运行时支持 exports 字段的时候，那么 package.json 中的顶级字段 main、types、module 还有 browser 将被忽略，被 exports 取代。但是，对于尚不支持 exports 字段的工具或运行时来说，设置这些字段仍然很重要。  
如果你有一个 "development" 和一个 "production" 的产出（例如，你有一些警告在 development 产出中有但在 production 产出中没有），那么你可以通过在 exports 字段中 "development" 和 "production" 来设置它们。注意一些打包工具例如 webpack 和 vite 将会自动识别这些导出条件，而 Rollup 也可以通过配置来识别它们，你需要提醒开发者在他们自己打包工具的配置中去做这些事。  

_列出要发布的 files_  
files 定义你的 NPM 包中要包含哪些文件  
files 决定 npm CLI 在打包库时哪些文件和目录包含到最终的 NPM 包中。  
例如，如果你将代码从 TypeScript 编译为 JavaScript，你可能就不想在 NPM 包中包含 TypeScript 的源代码。（相反，你应该包含 sourcemap）。  
files 可以接受一个字符串数组（如果需要，这些字符串可以包含类似 glob 的语法），例如：  
``` 
{
   "files": ["dist"]
 }
```
注意，文件数组不接受相对路径表示；"files": ["./dist"] 将无法正常工作。  
验证你已正确设置 files 的一种好方法是运行 npm publish --dry-run，它将根据此设置列出将会包含的文件。  

_为你的 JS 文件设置默认的模块 type_  
type 规定你的 .js 文件使用哪个模块系统  
运行时和打包工具需要一种方法来确定你的 .js 文件采用哪种模块系统 —— ESM 还是 CommonJS。因为 CommonJS 首先出现，所以它被打包工具视为默认的 - 但你可以通过在你的 package.json 中添加 "type" 来控制这种行为。  
可以选择 "type":"module" 或 "type":"commonjs"，也可以不添加该字段（默认为 CommonJS），但仍强烈建议你进行设置，显式地声明你正在使用哪一个。  
可以通过几个技巧在项目中混用模块类型：  
- .mjs 文件总是 ESM 模块，即使你的 package.json 有 "type": "commonjs"（或者没有 type）
- .cjs 文件总是 CommonJS 模块，即使你的 package.json 有 "type": "module"

可以在子目录下添加其他 package.json 文件；运行时和打包工具将向上遍历文件目录，直到寻找到最近的 package.json。这意味着你可以有两个不同的文件夹，都使用 .js 文件，但每个文件夹都有自己的 package.json 并设置为不同的 type 以获得基于 CommonJS 和 ESM 的文件夹。  

_列出哪些模块有 sideEffects_  
设置 sideEffects 来允许 treeshaking  
创建一个 “纯模块” 带来的优点与创建一个纯函数十分类似；打包工具能够对你的库更好的进行 treeshaking。  
通过设置 sideEffects 让打包工具知道你的模块是否是 “纯” 的。不设置这个字段，打包工具将不得不假设你所有的模块都是有副作用。  
sideEffects 可以设为 false，表示没有任何模块具有副作用，也可以设置为字符串数组来列出哪些文件具有副作用。例如：  
``` 
{
   // 所有模块都是“纯”的
   "sideEffects": false
 }
```
或  
``` 
{
   // 除了 "module.js"，所有模块都是“纯”的
   "sideEffects": ["module.js"]
 }
```
所以，什么让一个模块具有副作用？例如修改一个全局变量，发送 API 请求，或导出 CSS，而且开发人员不需要做任何事情这些动作就会被执行。例如：  
``` 
// 具有副作用的模块

 export const myVar = "hello";

 window.example = "testing";
```
导入 myVar 时，你的模块自动设置 window.example。  
例如：  
``` 
import { myVar } from "library";

 console.log(window.example);
 // 打印 "testing"
```
在某些情况下，如 polyfill，这种行为是有意的。然而，如果我们想让这个模块是 “纯” 的，我们可以将对 window.example 的赋值移动到一个函数中。例如：  
``` 
// 一个“纯”模块

 export const myVar = "hello";

 export function setExample() {
   window.example = "testing";
 }
```
现在这是一个 “纯” 模块。注意，从开发者的角度来看会有不同：  
``` 
import { myVar, setExample } from "library";

 console.log(window.example);
 // 打印 "undefined"

 setExample();

 console.log(window.example);
 // 打印 "testing"
```
_设置 main 字段_  
main 定义 CommonJS 入口  
main 是一个当打包工具或运行时不支持 package.json#exports 时的兜底方案；如果打包工具或运行时支持 package exports，则不会使用 main。  
main 应该指向一个兼容 CommonJS 格式的产出；它应该与 package exports 中的 require 保持一致。  
_设置 module 字段_  
module 定义 ESM 入口  
module 是一个当打包工具或运行时不支持 package.json#exports 时的兜底方案；如果打包工具或运行时支持 package exports，则不会使用 module。  
module 应该指向一个兼容 ESM 格式的产出；它应该与 package exports 中的 module 或 import 保持一致。  
_设置给 CDN 使用的附加字段_  
支持 CDN，例如 unpkg 和 jsdelivr  
为让你的库在 CDN 上 “以默认的方式正常工作”，如 unpkg 和 jsdelivr，你可以设置它们的特定字段指向你的 umd 产出。例如：  
``` 
{
   "unpkg": "./dist/index.umd.js",
   "jsdelivr": "./dist/index.umd.js"
 }
```
_设置 browser 字段_  
browser 指向能在浏览器中工作的产出  
browser 是一个当打包工具或运行时不支持 package.json#exports 时的兜底方案；如果打包工具或运行时支持 package exports， 则不会使用 browser。  
browser 应该指向能在浏览器中工作的 esm 产出。但是，只有在为浏览器和服务器（等其他非浏览器环境）创建不同的产出时，才需要设置该字段。如果你没有为多个环境创建多个产出，或者你的产出是 “纯 JavaScript” 或 “通用” 的，可以在任何 JavaScript 环境中运行，那么你就不需要设置 browser 字段。  
如果你确实需要设置该字段，这里有一个优秀的指南，介绍了配置它的不同方法。  
注意，browser 字段不应该指向 umd 产出，因为那样的话，你的库就不会被打包工具（如 Webpack）进行 treeshaking，这些打包工具会优先考虑这个字段，而不是其他字段，比如 module 和 main。  
_设置 types 字段_  
types 定义 TypeScript 类型  
types 是一个当打包工具或运行时不支持 package.json#exports 时的兜底方案；如果打包工具或运行时支持 package exports，则不会使用 types。  
types 应该指向你的 TypeScript 入口文件，例如 index.d.ts；它应该与 package exports 中的 types 字段指向同一个文件。  
_列出 peerDependencies_  
如果你依赖别的框架或库，将它设置为 peer dependency  
你应该外置框架。然而，这样做后，你的库只有在开发人员自行安装你需要的框架后才能工作。设置 peerDependencies 让他们知道他们需要安装的框架。- 例如，如果你在创建一个 React 库：  
``` 
{
   "peerDependencies": {
     "react": "^18.2.0",
     "react-dom": "^18.2.0"
   }
 }
```
你应该以书面形式来体现这些依赖；例如，npm v3-v6 不安装 peer dependencies，而 npm v7+ 将自动安装 peer dependencies。  
_说明你的库使用哪个许可证_  
> 开源许可证用于保护贡献者和用户。没有这种保护，企业和有经验的开发者不会使用该项目。

当你决定了许可证，关于许可证的 npm 文档中描述了许可证字段的格式。例如：  
``` 
{
   "license": "MIT"
 }
```
除此之外，你可以在项目的根目录下创建一个 LICENSE.txt 文件，并将许可证的文本复制到这里。

原文:  
[打包JavaScript库的现代化指南](https://mp.weixin.qq.com/s/m-i_Et6YqkZ0aj537vN2_A)
