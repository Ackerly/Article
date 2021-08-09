# vite入门
## 安装
- 使用NPM
``` 
npm init @vitejs/app
```
- 使用yarn
``` 
yarn create @vitejs/app
```
安装CSS预编译语言，三者选一
- 安装scss
``` 
npm install -D sass
```
- 安装less
``` 
npm install -D less
```
- .安装stylus
``` 
npm install -D stylus
```

## Glob Import
Vite支持特殊的import.meta.glob从文件系统中导入多个模块
``` 
const modules = import.meta.glob('./dir/*.js')
```
等同于
``` 
// code produced by vite
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```
通过遍历对象的键访问响应的模块
``` 
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```
匹配文件默认通过动态导入延迟加载，在构建过程分割为单独的块。如果要导入所有的模块可以使用import.meta.globEager
``` 
const modules = import.meta.globEager('./dir/*.js')
```
等同于
``` 
// code produced by vite
import * as __glob__0_0 from './dir/foo.js'
import * as __glob__0_1 from './dir/bar.js'
const modules = {
  './dir/foo.js': __glob__0_0,
  './dir/bar.js': __glob__0_1
}
```
## Web Assembly
预编译的.wasm文件可以直接导入--默认的导出监视一个初始化函数，返回wasm实例的exports对象的承诺
``` 
import init from './example.wasm'

init().then((exports) => {
  exports.test()
})
```
init函数可以接受传入WebAssembly的imports对象。
``` 
init({
  imports: {
    someFunc: () => {
      /* ... */
    }
  }
}).then(() => {
  /* ... */
})
```
在生产版本中，小于assetInlineLimit的.wasm文件将作为base64字符串内联。否则，它们将作为资产复制到dist目录中，并按需获取。
## Web Workers
导入请求中附加？worker,可以直接导入web worker脚本。默认的导出将是一个自定义的worker构造函数
``` 
import MyWorker from './worker?worker'
const worker = new MyWorker()
```
worker脚本也可以使用import语句来代替importScripts()——注意，在开发过程中，这依赖于浏览器本地支持，目前只在Chrome中工作，但在生产版本中，它被编译掉了。  
默认情况下，worker脚本将在生产构建中作为单独的块发出。如果你想将worker内联为base64字符串，添加内联查询:
``` 
import MyWorker from './worker?worker&inline'
```
## 打包优化
### 动态导入Polyfill
Vite使用ES动态导入作为代码分割点。生成的代码还将使用动态导入来加载异步块。然而，本机ESM动态导入支持是在ESM之后通过脚本标记实现的，并且这两个特性在浏览器支持方面存在差异。Vite会自动注入一个轻量级的动态导入填充来消除这种差异。  
如果你的目标浏览器只支持本机动态导入，可以通过build.polyfillDynamicImport显式禁用此特性。
### CSS代码分离
Vite自动提取模块在一个异步块中使用的CSS，并为它生成一个单独的文件。当相关的异步块被加载时，CSS文件通过标签自动加载，并且异步块保证只在CSS加载后才被计算，以避免FOUC。  
如果想把所有CSS提取到一个文件中，可以通过设置build禁用CSS代码分割，通过设置build.cssCodeSplit为假。
### 生成预加载指令
Vite自动生成<link rel="modulepreload"> 指令，用于条目块和它们在构建的HTML中直接导入。
### 异步块加载优化
