# 从Lint工具窥探前端的clean-code
## Prettier and ESLint
**什么是Prettier**  
一般提起代码规范大家比较容易想到的是代码呈现出的格式（风格），对于前端来说比如：每行代码空 2 格 / 4 格、要不要写分号、箭头函数单个参数要不要括号和一行最长多少字母等等格式问题，这个可以对应于 Prettier - Code for Formatter。  
Prettier 的特点是：用户自定义不多的 样式美化 工具。Prettier 的英文名也是「更漂亮的」。也就是说它只是美化 代码样式 并不会检查 代码质量 。这就是它与 ESLint 最大的区别。  
**为什么使用Prettier**  
基本开源的大型前端项目都会用 Prettier，比如：Ant Design Vue、elementUI、Vue3 和 React 等等。因为他们每天需要处理来自世界各地的 pull requests，如果没有 Prettier 应该天天都脑充血。。。。Prettier 官网给到的 Why Prettier 理由：  
- Building and enforcing a style guide：不需要讨论制定规则，不需要学习任何规则，Prettier 强制执行一个固定的代码风格（用户可自定义的规则不多），或许没有 100% 按照我们自己的意愿格式化所有代码，但是这样开箱即用更加提高效率。
- Helping Newcomers：对新手友好，好的代码格式可以避免很多低级错误。
- Writing code：格式化代码的设置非常友好，option+shift+F 一键美化，写代码的时候不需要花费大量的时间和精力来格式化代码。或者和编辑器相结合，设置 formate on save，在保存时自动格式化。
- Easy to adopt：Prettier 团队在 2017 年的时候就已经不接受增加更多配置项的 issue 了，具体看 issue。这样做就是为了保持它开箱即用的特点，并且保证新格式化的代码风格不会引起重大争议并被大家轻松接受。

**什么是ESLint**  
Lint 工具是一种软件质量保证工具，早在零几年微软就开始启用 Lint 来检查程序，而 ESLint 是属于 Lint 的一种 前端代码质量保证工具 。感兴趣的可以看看 Linter 进化史。现在的 ESLint 除了对代码质量的控制，比如：不使用 var 变量、 switch 必须要设置 default 等等之外，也有一些代码风格的控制，比如：是否使用分号和使用 tab 还是空格等等。  

**为什么使用ESLint**  
代码检查是一种静态分析来寻找有问题的代码，不依赖于具体的编码风格。对于大多数强类型语言，在编译时就会通过内置的检查工具对代码进行检查。但是 JavaScript 是一个动态的弱类型语言，没有编译过程，需要一个检查低质量代码的工具，这就是 ESLint 出现的初衷。ESLint 的所有规则与 Prettier 相比都是可选择的 unopinionated 。你可以使用默认规则或者自定义规则，也可以选择将规则开启 / 关闭  

**怎么使用ESLint**  
基本的使用方法，在项目中安装：  
``` 
npm install eslint --save-dev
```
然后使用 eslint 创建一个配置文件  
``` 
./node_modules/.bin/eslint --init
```
然后在配置文件 .eslintrc.js 中 extends:"eslint:recommended" 就可以使用 ESLint 官方推荐的规则，也是非常方便的。也可以自定义需要的规则：  
``` 
{
   "rules": {
     "semi": ["error", "always"],
      "quotes": ["error", "double"]
   }
 }
```
就可以使用 eslint 去校验文件：  
``` 
./node_modules/.bin/eslint  文件路径.js
```
在 ESLint 没有 --fix 来自动修复未通过规则问题的功能时，需要和 Prettier 结合使用才方便修改，现在也可以不需要 Prettier。不过 --fix 只会修复 AST 语法树 层面所感知的问题，比如：将 const b = data.a 转换为解构形式 const { b } = data 、修改代码的缩进、增加 / 减少分号等，其他的动态层面比如：虽然强制使用 === 但是 fix 时并不会将 == 转换为 === ，因为这两个意图是有可能不一样的。  
在平时项目开发中有两种方式去快速修复 ESLint 问题：  
- 命令行里运行 eslint 检查代码文件的同时加上 --fix
- 在 VSCode 里安装 eslint 插件，进入项目 vscode 会自动检测项目的 eslint 配置文件 .eslintrc ，然后点击 allow 表示同意 eslint 作为默认的 formate 代码的配置。最后就可以愉快的使用 formate on save 或者 option+shift+F 自动格式化了

## clean-code
因为 JS 语言并没有编译过程，在没有使用 ESLint 之前并没有发现原来自己的代码有很多隐藏的 bug。  
举个例子
for in 和 for of 的区别是什么？  
for in 更适合遍历对象而不是数组（遍历键名 key）
for in 更适合遍历对象而不是数组（遍历键名 key）
- 遍历的 index（键名）是字符串型的索引，不能直接加减
- 会遍历 所有的可枚举属性 ，包括原型上的属性 (可以加 hasOwnProperty 阻止遍历到原型)
- 遍历顺序有可能不是按照实际数组的内部顺序
for of 更适合数组（遍历键值 value）  
- 遍历的只是数组内的元素，而不包括原型属性
- 适用遍历数 / 数组 / 字符串 / Map/Set 等拥有迭代器对象的集合，但是 不能遍历对象 ，因为对象没有迭代器对象。

**什么是clean-code**  
clean-code-javascript 的规则一共分为九个章节，分别是：变量、函数、对象和数据结构、类、测试、异步、错误处理、格式化和注释。在每一个章节中又分为具体的规则。  
举个例子：函数这一章有一条规则是尽量采用函数式编程，因为函数式的编程具有更干净且便于测试的特点。
Bad：  
``` 
const programmerOutput = [
   {
     name: 'Uncle Bobby',
     linesOfCode: 500
   }, {
     name: 'Suzie Q',
     linesOfCode: 1500
   }
   ......
 ];

 // 使用for循环计算programmerOutput对象里linesOfCode的总数
 var totalOutput = 0;
 for (var i = 0; i < programmerOutput.length; i++) {
   totalOutput += programmerOutput[i].linesOfCode;
 }
```
Good：  
``` 
// 使用高阶函数计算总数
 var totalOutput = programmerOutput
   .map((programmer) => programmer.linesOfCode)
   .reduce((acc, linesOfCode) => acc + linesOfCode, 0);
```
但是很多时候并不是上述这样一定要将循环转变为函数式编程的形式，以 Vue 的源码为例：  
``` 
 // pacakages/template-explorer
 monaco.editor.setModelMarkers(
         editor.getModel()!,
         `@vue/compiler-dom`,
         // 传入的参数使用链式调用直接计算
         errors.filter(e => e.loc).map(formatError)
   )
 // packages/complier-core
 const flagNames = Object.keys(PatchFlagNames) //使用Object.keys遍历对象
             // 高阶函数进行链式调用
             .map(Number)
             .filter(n => n > 0 && patchFlag & n)
             .map(n => PatchFlagNames[n])
             .join(`, `)
```
相对于「for 循环」，上述的函数式编程方式其实在 vue 源码的占比很小很小。因为很多时候在循环的逻辑比较复杂的情况下，都会采用 for 循环的方式去写，而函数式编程就用在可以一条语句完成需要的功能的时候。  
对于 new coder 来说还是需要 good coder 在不同场景下的 code review 来提高代码质量。这是视情况而定的。  
但是对于 for-in 迭代器，ESLint 有两条规则 no-iterator 和 no-restricted-syntax 用来限制使用 for-in 迭代器，推荐使用 map/reduce/find/some/every/forEach... 等高阶函数完成数组操作，Object.keys/Object.values/Object.entries 完成遍历对象生成所需数组的操作。  

**为什么使用clean-code**  
熵增定律是：在一个孤立系统里，如果没有外力做功，其总混乱度（熵）会不断增大，最后到达最混乱无序的状态。  
所以从熵增原则来看代码的话：一个项目的代码（孤立的系统），如果没有保证代码质量的环节（外力做功），随着时间推移代码量越来越大的情况下，代码质量肯定是会变得越来越混乱甚至到最后无法正常去维护。一旦项目代码的熵超过了一定的界限，但是代码本身的产品还需要不停更新迭代时，就需要去重构代码，将其迁移到最开始整洁、简洁的结构中（降低熵值）。  

**怎么使用clean-code**  
第一步：Pre-commit 阶段和 lint 工具结合（对应本文第一部分），例如：husky/pre-commit + lint-staged 工具构建本地提交之前的检查。现目前 React 已经将 lint-staged 集成到了脚手架 create-react-app 中。  
第二步：团队代码 code review（对应本文第二部分）。GitHub 上许多流行项目采用 「PR (Pull Request) 」工作流的方式，一个 PR 至少经过三人次 review 通过才能合入。不过这一块就比较主观了，模式不固定需要根据实际情况去制定。  

原文:  
[从Lint工具窥探前端的clean-code](https://mp.weixin.qq.com/s/8JwAlMU6DUWTxzFYKxTH3A)
