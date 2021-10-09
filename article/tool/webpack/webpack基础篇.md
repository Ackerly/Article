# webpack基础篇
## webpack是什么
webpack 是一个现代 JavaScript 应用程序的静态模块打包器，当 webpack 处理应用程序时，会递归构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将这些模块打包成一个或多个 bundle。
## webpack的核心概念
- entry: 入口
- output: 输出
- loader: 模块转换器，用于把模块原内容按照需求转换成新内容
- 插件(plugins): 扩展插件，在webpack构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要做的事情
## 初始化项目
1. 新建一个文件夹
2. 使用npm init -y 进行初始化（可以使用yarn）
3. 安装webpack、webpack-cli
    ``` 
    npm install webpack webpack-cli -D
    ```
    webpack V4.0.0 开始， webpack 是开箱即用的，在不引入任何配置文件的情况下就可以使用。
4. 新建 src/index.js 文件,随便写点内容
5. npx webpack --mode=development 进行构建，默认是 production 模式
6. 项目下多了个 dist 目录，里面有一个打包出来的文件 main.js。webpack 有默认的配置，如默认的入口文件是./src，默认打包到dist/main.js。更多的默认配置可以查看: node_modules/webpack/lib/WebpackOptionsDefaulter.js。查看 dist/main.js 文件，可以看到，src/index.js 并没有被转义为低版本的代码
## 将JS转义为低版本
**babel-loader**  
JS代码向低版本转换，我们需要使用 babel-loader。安装一下依赖
``` 
npm install @babel/core @babel/preset-env @babel/plugin-transform-runtime -D
npm install @babel/runtime @babel/runtime-corejs3
```
新建webpack.config.js  
``` 
//webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: ['babel-loader'],
                exclude: /node_modules/ //排除 node_modules 目录
            }
        ]
    }
}
```
loader 指定 include 或是 exclude，指定其中一个即可，因为 node_modules 目录通常不需要我们去编译，排除后，有效提升编译效率。  
**创建一个 .babelrc**  
``` 
{
    "presets": ["@babel/preset-env"],
    "plugins": [
        [
            "@babel/plugin-transform-runtime",
            {
                "corejs": 3
            }
        ]
    ]
}
```
新执行 npx webpack --mode=development，查看 dist/main.js，会发现已经被编译成了低版本的JS代码。  
**webpack中配置 babel**  
``` 
//webpack.config.js
module.exports = {
    // mode: 'development',
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ["@babel/preset-env"],
                        plugins: [
                            [
                                "@babel/plugin-transform-runtime",
                                {
                                    "corejs": 3
                                }
                            ]
                        ]
                    }
                },
                exclude: /node_modules/
            }
        ]
    }
}
```
补充说明：  
- loader需要配置module.rules 中，rules 是一个数组
- loader 的格式为:
   ```
   {
       test: /\.jsx?$/,//匹配规则
       use: 'babel-loader'
   }
   ```
   或者
   ``` 
   //适用于只有一个 loader 的情况
   {
       test: /\.jsx?$/,
       loader: 'babel-loader',
       options: {
           //...
       }
   }
   ```
test 字段是匹配规则，针对符合规则的文件进行处理。  
use 字段有几种写法  
- 可以是一个字符串，例如上面的 use: 'babel-loader'
- use 字段可以是一个数组，例如处理CSS文件是，use: ['style-loader', 'css-loader']
- use 数组的每一项既可以是字符串也可以是一个对象，当我们需要在webpack 的配置文件中对 loader 进行配置，就需要将其编写为一个对象，并且在此对象的 options 字段中进行配置，如：
    ``` 
    rules: [
        {
            test: /\.jsx?$/,
            use: {
                loader: 'babel-loader',
                options: {
                    presets: ["@babel/preset-env"]
                }
            },
            exclude: /node_modules/
        }
    ]
    ```
可能还需要一些其它的 babel 的插件和预设，例如 @babel/preset-react，@babel/plugin-proposal-optional-chaining 等，不过，babel 的配置并非本文的重点，
## mode
将 mode 增加到 webpack.config.js 中:
``` 
module.exports = {
    //....
    mode: "development",
    module: {
        //...
    }
}
```
mode 配置项，告知 webpack 使用相应模式的内置优化。
mode配置项支持以下两个配置:
- development：将 process.env.NODE_ENV 的值设置为 development，启用 NamedChunksPlugin 和 NamedModulesPlugin
- production：将 process.env.NODE_ENV 的值设置为 production，启用 FlagDependencyUsagePlugin, FlagIncludedChunksPlugin, ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin 和 UglifyJsPlugin  
## 浏览器中查看页面
安装依赖
``` 
npm install webpack-dev-server -D
```
修改下咱们的 package.json 文件的 scripts：
``` 
"scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server",
    "build": "cross-env NODE_ENV=production webpack"
},
```
配置了 html-webpack-plugin 的情况下， contentBase 不会起任何作用  
可以在 webpack.config.js 中进行 webpack-dev-server 的其它配置，例如指定端口号，设置浏览器控制台消息，是否压缩等等:
``` 
//webpack.config.js
module.exports = {
    //...
    devServer: {
        port: '3000', //默认是8080
        quiet: false, //默认不启用
        inline: true, //默认开启 inline 模式，如果设置为false,开启 iframe 模式
        stats: "errors-only", //终端仅打印 error
        overlay: false, //默认不启用
        clientLogLevel: "silent", //日志等级
        compress: true //是否启用 gzip 压缩
    }
}
```
- 启用 quiet 后，除了初始启动信息之外的任何内容都不会被打印到控制台。这也意味着来自 webpack 的错误或警告在控制台不可见 
- stats: "errors-only" ， 终端中仅打印出 error，注意当启用了 quiet 或者是 noInfo 时，此属性不起作用
- 启用 overlay 后，当编译出错时，会在浏览器窗口全屏输出错误，默认是关闭的。
- clientLogLevel: 当使用内联模式时，在浏览器的控制台将显示消息，如：在重新加载之前，在一个错误之前，或者模块热替换启用时。如果你不喜欢看这些信息，可以将其设置为 silent (none 即将被移除)。
## devtool
evtool 中的一些设置，可以帮助我们将编译后的代码映射回原始源代码。不同的值会明显影响到构建和重新构建的速度.
``` 
//webpack.config.js
module.exports = {
    devtool: 'cheap-module-eval-source-map' //开发环境下使用
}
```
生产环境可以使用 none 或者是 source-map，使用 source-map 最终会单独打包出一个 .map 文件，我们可以根据报错信息和此 map 文件，进行错误解析，定位到源代码。  
source-map 和 hidden-source-map 都会打包生成单独的 .map 文件，区别在于，source-map 会在打包出的js文件中增加一个引用注释，以便开发工具知道在哪里可以找到它。hidden-source-map 则不会在打包的js中增加引用注释。  
但是一般不会直接将 .map 文件部署到CDN，因为会直接映射到源码，更希望将.map 文件传到错误解析系统，然后根据上报的错误信息，直接解析到出错的源码位置。
## 处理样式文件
ebpack 不能直接处理 css，需要借助 loader。如果是 .css，我们需要的 loader 通常有： style-loader、css-loader，考虑到兼容性问题，还需要 postcss-loader，而如果是 less 或者是 sass 的话，还需要 less-loader 和 sass-loader，这里配置一下 less 和 css 文件(sass 的话，使用 sass-loader即可):
``` 
npm install style-loader less-loader css-loader postcss-loader autoprefixer less -D
```
``` 
//webpack.config.js
module.exports = {
    //...
    module: {
        rules: [
            {
                test: /\.(le|c)ss$/,
                use: ['style-loader', 'css-loader', {
                    loader: 'postcss-loader',
                    options: {
                        plugins: function () {
                            return [
                                require('autoprefixer')({
                                    "overrideBrowserslist": [
                                        ">0.25%",
                                        "not dead"
                                    ]
                                })
                            ]
                        }
                    }
                }, 'less-loader'],
                exclude: /node_modules/
            }
        ]
    }
}
```
- style-loader 动态创建 style 标签，将 css 插入到 head 中
- css-loader 负责处理 @import 等语句
- postcss-loader 和 autoprefixer，自动生成浏览器兼容性前缀 
- less-loader 负责处理编译 .less 文件,将其转为 css
**注意**  
loader 的执行顺序是从右向左执行的，也就是后面的 loader 先执行，上面 loader 的执行顺序为: less-loader ---> postcss-loader ---> css-loader ---> style-loader  
loader 其实还有一个参数，可以修改优先级，enforce 参数，其值可以为: pre(优先执行) 或 post (滞后执行)。  
## 图片/字体文件处理
使用 url-loader 或者 file-loader 来处理本地的资源文件。url-loader 和 file-loader 的功能类似，但是 url-loader 可以指定在文件大小小于指定的限制时，返回 DataURL  
安装依赖  
``` 
npm install url-loader -D
```
webpack.config.js 中进行配置：
``` 
//webpack.config.js
module.exports = {
    //...
    modules: {
        rules: [
            {
                test: /\.(png|jpg|gif|jpeg|webp|svg|eot|ttf|woff|woff2)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 10240, //10K
                            esModule: false 
                        }
                    }
                ],
                exclude: /node_modules/
            }
        ]
    }
}
```
limit 的值大小为 10240，即资源大小小于 10K 时，将资源转换为 base64，超过 10K，将图片拷贝到 dist 目录。esModule 设置为 false，否则，<img src={require('XXX.jpg')} /> 会出现 <img src=[Module Object] />  
将资源转换为 base64 可以减少网络请求次数，但是 base64 数据较大，如果太多的资源是 base64，会导致加载变慢，因此设置 limit 值时，需要二者兼顾。  
默认情况下，生成的文件的文件名就是文件内容的 MD5 哈希值并会保留所引用资源的原始扩展名  
可以通过 options 参数进行修改
``` 
//....
use: [
    {
        loader: 'url-loader',
        options: {
            limit: 10240, //10K
            esModule: false,
            name: '[name]_[hash:6].[ext]'
        }
    }
]
```
当本地资源较多时，我们有时会希望它们能打包在一个文件夹下，这也很简单，我们只需要在 url-loader 的 options 中指定 outpath，如: outputPath: 'assets'
## 处理html中的本地图片
安装 html-withimg-loader
``` 
npm install html-withimg-loader -D
```
修改 webpack.config.js：
``` 
module.exports = {
    //...
    module: {
        rules: [
            {
                test: /.html$/,
                use: 'html-withimg-loader'
            }
        ]
    }
}
```
使用 html-withimg-loader 处理图片之后，html 中就不能使用 vm, ejs 的模板了.如果想继续在 html 中使用 <% if(htmlWebpackPlugin.options.config.header) { %> 这样的语法，但是呢，又希望能使用本地图片.下面这样编写图片的地址，并且删除html-withimg-loader的配置即可。
``` 
<!-- index.html -->
<img src="<%= require('./thor.jpeg') %>" />
```
## 入口配置
``` 
//webpack.config.js
module.exports = {
    entry: './src/index.js' //webpack的默认配置
}
```
entry 的值可以是一个字符串，一个数组或是一个对象,为数组时，表示有“多个主入口”，想要多个依赖文件一起注入时，会这样配置。例如:
``` 
entry: [
    './src/polyfills.js',
    './src/index.js'
]
```
polyfills.js 文件中可能只是简单的引入了一些 polyfill，例如 babel-polyfill，whatwg-fetch 等，需要在最前面被引入
## 出口配置
output 选项可以控制 webpack 如何输出编译文件
``` 
const path = require('path');
module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'), //必须是绝对路径
        filename: 'bundle.js',
        publicPath: '/' //通常是CDN地址
    }
}
```
给文件名加上 hash
``` 
//webpack.config.js
module.exports = {
    output: {
        path: path.resolve(__dirname, 'dist'), //必须是绝对路径
        filename: 'bundle.[hash].js',
        publicPath: '/' //通常是CDN地址
    }
}
```
## 每次打包前清空dist目录
安装插件clean-webpack-plugin
``` 
npm install clean-webpack-plugin -D
```
以前，clean-webpack-plugin 是默认导出的，现在不是，所以引用的时候，需要注意一下。另外，现在构造函数接受的参数是一个对象，可缺省。
``` 
//webpack.config.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    //...
    plugins: [
        //不需要传参数喔，它可以找到 outputPath
        new CleanWebpackPlugin() 
    ]
}
```
**希望dist目录下某个文件夹不被清空**  
clean-webpack-plugin 为我们提供了参数 cleanOnceBeforeBuildPatterns。
``` 
//webpack.config.js
module.exports = {
    //...
    plugins: [
        new CleanWebpackPlugin({
            cleanOnceBeforeBuildPatterns:['**/*', '!dll', '!dll/**'] //不删除dll目录下的文件
        })
    ]
}
```

参考:
[带你深度解锁Webpack系列(基础篇)](https://juejin.cn/post/6844904079219490830)
