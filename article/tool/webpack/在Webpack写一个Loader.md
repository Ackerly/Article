# 在Webpack写一个Loader.
**为什么需要Loader**  
Webpack 它只能处理 js 和 JSON 文件。面对 css 文件还有一些图片等等，Webpack 它自己是不能够处理的，它需要loader 处理其他类型的文件并将它们转换为有效的模块以供应用程序使用并添加到依赖关系图中
**Loader是什么**  
loader本质上是一个node模块，符合Webpack中一切皆模块的思想。由于它是一个 node 模块，它必须导出一些东西。loader本身就是一个函数，在该函数中对接收到的内容进行转换，然后返回转换后的结果  

## 常见的loader
### css-loader 和 style-loader  
**安装依赖**  
``` 
npm install css-loader style-loader
```
**使用加载器**  
``` 
module.exports = {
    // ...
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader'],
        }],
    },
}；
```
其中module.rules代表模块的处理规则。每个规则可以包含很多配置项  
test 可以接收正则表达式或元素为正则表达式的数组。只有与正则表达式匹配的模块才会使用此规则。在此示例中，/\.css$/ 匹配所有以 .css 结尾的文件。  
use 可以接收一个包含规则使用的加载器的数组。如果只配置了一个css-loader，当只有一个loader时也可以为字符串  
css-loader 的作用只是处理 CSS 的各种加载语法（@import 和 url() 函数等），如果样式要工作，则需要 style-loader 将样式插入页面  
style-loader加到了css-loader前面，这是因为在Webpack打包时是按照数组从后往前的顺序将资源交给loader处理的，因此要把最后生效的放在前面  
还可以这样写成对象的形式，里面options传入配置  
``` 
module.exports = {
    // ...
    module: {
        rules: [{
            test: /\.css$/,
            use: [
                'style-loader',
                  {
                    loader: 'css-loader',
                    options: {
                        // css-loader 配置项
                 },
               }
            ],
        }],
    },
}；
```
exclude与include  
include代表该规则只对正则匹配到的模块生效  
exclude的含义是，所有被正则匹配到的模块都排除在该规则之外  
**babel-loader**
babel-loader 这个loader十分的重要，把高级语法转为ES5，常用于处理 ES6+ 并将其编译为 ES5。它允许我们在项目中使用最新的语言特性（甚至在提案中），而无需特别注意这些特性在不同平台上的兼容性。  
主要的三个模块:  
- babel-loader：使 Babel 与 Webpack 一起工作的模块
- @babel/core：Babel核心模块。
- @babel/preset-env：是Babel官方推荐的preseter，可以根据用户设置的目标环境，自动添加编译ES6+代码所需的插件和补丁

**安装**  
``` 
npm install babel-loader @babel/core @babel/preset-env
```
配置  
``` 
rules: [
  {
    test: /\.js$/,
    exclude: /node_modules/, //排除掉，不排除拖慢打包的速度
    use: {
      loader: 'babel-loader',
      options: {
        cacheDirectory: true, // 启用缓存机制以防止在重新打包未更改的模块时进行二次编译
        presets: [[
          'env', {
            modules: false, // 将ES6 Module的语法交给Webpack本身处理
          }
        ]],
      },
    },
  }
],
```
**html-loader**  
html-loader 用于将 HTML 文件转换为字符串并进行格式化，它允许我们通过 JS 加载一个 HTML 片段。  
安装  
``` 
npm install html-loader
```
配置  
``` 
rules: [
    {
        test: /\.html$/,
        use: 'html-loader',
    }
],

// index.js
import otherHtml from './other.html';
document.write(otherHtml);
```
**file-loader**  
用于打包文件类型的资源，比如对png、jpg、gif等图片资源使用file-loader，然后就可以在JS中加载图片了  
安装  
``` 
npm install file-loader
```
配置  
``` 
const path = require('path');
module.exports = {
    entry: './index.js',
    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'bundle.js',
    },
    module: {
        rules: [
            {
                test: /\.(png|jpg|gif)$/,
                use: 'file-loader',
            }
        ],
    },
}；
```
**url-loader**  
与file-loader很相似，但是唯一的区别是用户可以设置文件大小阈值。大于阈值时返回与file-loader相同的publicPath，小于阈值时返回文件base64编码。  
安装  
``` 
npm install url-loader
```
配置  
``` 
rules: [
    {
        test: /\.(png|jpg|gif)$/,
        use: {
            loader: 'url-loader',
            options: {
                limit: 1024,
                name: '[name].[ext]',
                publicPath: './assets/',
            },
        },
    }
],
```
**ts-loader**  
类似于 babel-loader，是一个连接 Webpack 和 Typescript 的模块  
安装  
``` 
npm install ts-loader typescript
```
loader配置，主要的配置还是在 tsconfig.json 中  
``` 
rules: [
    {
        test: /\.ts$/,
        use: 'ts-loader',
    }
],
```
**vue-loader**  
用来处理vue组件,还要安装vue-template-compiler来编译Vue模板  
安装  
``` 
npm install vue-loader  vue-template-compiler 

rules: [
    {
        test: /\.vue$/,
        use: 'vue-loader',
    }
],
```
## 写一个简单的Loader
**初始化项目**  
先创建一个项目文件夹后进行初始化    
``` 
npm init -y
```
安装依赖  
安装依赖： Webpack 和 Webpack脚手架 和 热更新服务器  
``` 
npm install webpack@4.39.2 webpack-cli@3.3.6 webpack-dev-server@3.11.0 -D
```
新建一个index.html文件  
``` 
// dist/index.html
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <meta http-equiv="X-UA-Compatible" content="ie=edge" />
        <title></title>
    </head>
    <body>
        <script src="./bundle.js"></script>
    </body>
</html>
```
新建一个入口文件 index.js 文件  
``` 
// src/index.js  

document.write('hello world')
```
创建 webpack.config.js 配置文件,配置出口和入口文件,配置devServer服务  
``` 
const path = require('path')

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
    },
    devServer: {
        contentBase: './dist',
        overlay: {
            warnings: true,
            errors: true,
        },
        open: true,
    },
}
```
在 package.json 中配置启动命令  
``` 
"scripts": {
    "dev": "Webpack-dev-server"
  }
```
启动 npm run dev  
devServer帮我们启动一个服务器，每次修改index.js不需要自己在去打包，而是自动帮我们完成这项任务  
页面内容就是我们index.js编写的内容被打包成在dist/bundle.js引入到index.html了  
**实现一个简单的 loader**  


参考:  
[手把手教你在 Webpack 写一个 Loader](https://mp.weixin.qq.com/s/2wDp4cRr6uWmitrsCPLOmQ)
