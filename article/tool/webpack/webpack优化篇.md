# webpack优化篇
随着项目越来越大，构建速度可能会越来越慢，构建出来的js的体积也越来越大，此时就需要对 Webpack 的配置进行优化。
## 量化
有时，我们以为的优化是负优化，这时，如果有一个量化的指标可以看出前后对比，那将会是再好不过的一件事。speed-measure-webpack-plugin 插件可以测量各个插件和loader所花费的时间，对比前后的信息，来确定优化的效果。
speed-measure-webpack-plugin 的使用很简单，直接用其来包裹 Webpack 的配置:
``` 
//webpack.config.js
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();

const config = {
    //...webpack配置
}

module.exports = smp.wrap(config);
```
## exclude/include
通过 exclude、include 配置来确保转译尽可能少的文件。exclude 的优先级高于 include，在 include 和 exclude 中使用绝对路径数组，尽量避免 exclude，更倾向于使用 include。
``` 
//webpack.config.js
const path = require('path');
module.exports = {
    //...
    module: {
        rules: [
            {
                test: /\.js[x]?$/,
                use: ['babel-loader'],
                include: [path.resolve(__dirname, 'src')]
            }
        ]
    },
}
```
## cache-loader
在一些性能开销较大的 loader 之前添加 cache-loader，将结果缓存中磁盘中。默认保存在 node_modueles/.cache/cache-loader 目录下.  
安装依赖
``` 
npm install cache-loader -D
```
cache-loader 的配置放在其他 loader 之前
``` 
module.exports = {
    //...
    
    module: {
        //我的项目中,babel-loader耗时比较长，所以我给它配置了`cache-loader`
        rules: [
            {
                test: /\.jsx?$/,
                use: ['cache-loader','babel-loader']
            }
        ]
    }
}
```
只打算给 babel-loader 配置 cache 的话，也可以不使用 cache-loader，给 babel-loader 增加选项 cacheDirectory。
cacheDirectory：默认值为 false。当有设置时，指定的目录将用来缓存 loader 的执行结果。之后的 Webpack构建，将会尝试读取缓存，来避免在每次执行时，可能产生的、高性能消耗的 Babel 重新编译过程。设置空值或者true的话，使用默认缓存目录：node_modules/.cache/babel-loader。开启 babel-loader的缓存和配置 cache-loader，构建时间很接近。
## happypack
由于有大量文件需要解析和处理，构建是文件读写和计算密集型的操作，特别是当文件数量变多后，Webpack 构建慢的问题会显得严重。HappyPack 把任务分解给多个子进程去并发的执行，子进程处理完后再把结果发送给主进程。  
安装：
``` 
npm install happypack -D
```
修改配置文件
``` 
const Happypack = require('happypack');
module.exports = {
    //...
    module: {
        rules: [
            {
                test: /\.js[x]?$/,
                use: 'Happypack/loader?id=js',
                include: [path.resolve(__dirname, 'src')]
            },
            {
                test: /\.css$/,
                use: 'Happypack/loader?id=css',
                include: [
                    path.resolve(__dirname, 'src'),
                    path.resolve(__dirname, 'node_modules', 'bootstrap', 'dist')
                ]
            }
        ]
    },
    plugins: [
        new Happypack({
            id: 'js', //和rule中的id=js对应
            //将之前 rule 中的 loader 在此配置
            use: ['babel-loader'] //必须是数组
        }),
        new Happypack({
            id: 'css',//和rule中的id=css对应
            use: ['style-loader', 'css-loader','postcss-loader'],
        })
    ]
}
```
happypack 默认开启 CPU核数 - 1 个进程，当然，我们也可以传递 threads 给 Happypack。  
当 postcss-loader 配置在 Happypack 中，必须要在项目中创建 postcss.config.js
``` 
//postcss.config.js
module.exports = {
    plugins: [
        require('autoprefixer')()
    ]
}
```
## thread-loader
thread-loader 放置在其它 loader 之前，那么放置在这个 loader 之后的 loader 就会在一个单独的 worker 池中运行。  
worker 池(worker pool)中运行的 loader 是受到限制的。例如：
- 这些 loader 不能产生新的文件
- 这些 loader 不能使用定制的 loader API（也就是说，通过插件）
- 这些 loader 无法获取 webpack 的选项设置

安装依赖
``` 
npm install thread-loader -D
```
修改配置:
``` 
module.exports = {
    module: {
        //项目中,babel-loader耗时比较长，所以我给它配置 thread-loader
        rules: [
            {
                test: /\.jsx?$/,
                use: ['thread-loader', 'cache-loader', 'babel-loader']
            }
        ]
    }
}
```
## 开启 JS 多进程压缩
很多 webpack 优化的文章上会提及多进程压缩的优化，不管是 webpack-parallel-uglify-plugin 或者是 uglifyjs-webpack-plugin 配置 parallel.但是没必要单独安装这些插件，它们并不会让你的 Webpack 构建速度提升。  
Webpack 默认使用的是 TerserWebpackPlugin，默认就开启了多进程和缓存，构建时，你的项目中可以看到 terser 的缓存文件 node_modules/.cache/terser-webpack-plugin。
## HardSourceWebpackPlugin
HardSourceWebpackPlugin 为模块提供中间缓存，缓存默认的存放路径是: node_modules/.cache/hard-source。  
配置 hard-source-webpack-plugin，首次构建时间没有太大变化，但是第二次开始，构建时间大约可以节约 80%。  
安装依赖
``` 
npm install hard-source-webpack-plugin -D
```
修改webpack配置
``` 
//webpack.config.js
var HardSourceWebpackPlugin = require('hard-source-webpack-plugin');
module.exports = {
    //...
    plugins: [
        new HardSourceWebpackPlugin()
    ]
}
```
[HardSourceWebpackPlugin文档中](https://www.npmjs.com/package/hard-source-webpack-plugin) 列出了一些你可能会遇到的问题以及如何解决，例如热更新失效，或者某些配置不生效等。
## noParse   
如果一些第三方模块没有AMD/CommonJS规范版本，可以使用 noParse 来标识这个模块，这样 Webpack 会引入这些模块，但是不进行转化和解析，从而提升 Webpack 的构建性能 ，例如：jquery 、lodash。  
noParse 属性的值是一个正则表达式或者是一个 function。 
``` 
//webpack.config.js
module.exports = {
    //...
    module: {
        noParse: /jquery|lodash/
    }
}
```
## resolve
resolve 配置 webpack 如何寻找模块所对应的文件。假设我们确定模块都从根目录下的 node_modules 中查找，我们可以配置:
``` 
//webpack.config.js
const path = require('path');
module.exports = {
    //...
    resolve: {
        modules: [path.resolve(__dirname, 'node_modules')],
    }
}
```
如果你配置了上述的 resolve.moudles ，可能会出现问题，例如，依赖中还存在 node_modules 目录，那么就会出现，对应的文件明明在，但是却提示找不到。因此呢，不推荐配置这个。    
另外，resolve 的 extensions 配置，默认是 ['.js', '.json']，如果你要对它进行配置，记住将频率最高的后缀放在第一位，并且控制列表的长度，以减少尝试次数。
## IgnorePlugin
webpack 的内置插件，作用是忽略第三方包指定目录。例如: moment (2.24.0版本) 会将所有本地化内容和核心功能一起打包，我们就可以使用 IgnorePlugin 在打包时忽略本地化内容。
``` 
//webpack.config.js
module.exports = {
    //...
    plugins: [
        //忽略 moment 下的 ./locale 目录
        new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)
    ]
}
```
在使用的时候，如果需要指定语言，那么需要手动的去引入语言包，例如，引入中文语言包:
``` 
import moment from 'moment';
import 'moment/locale/zh-cn';// 手动引入
```
index.js 中只引入 moment，打包出来的 bundle.js 大小为 263KB，如果配置了 IgnorePlugin，单独引入 moment/locale/zh-cn，构建出来的包大小为 55KB。
## externals
希望通过 import 的方式去引用(如 import $ from 'jquery')，并且希望 webpack 不会对其进行打包，此时就可以配置 externals
``` 
//webpack.config.js
module.exports = {
    //...
    externals: {
        //jquery通过script引入之后，全局中即有了 jQuery 变量
        'jquery': 'jQuery'
    }
}
```
## DllPlugin
DllPlugin 和 DLLReferencePlugin 可以实现拆分 bundles，并且可以大大提升构建速度，DllPlugin 和 DLLReferencePlugin 都是 webpack 的内置模块。  
DllPlugin 将不会频繁更新的库进行编译，当这些依赖的版本没有变化时，就不需要重新编译。新建一个 webpack 的配置文件，来专门用于编译动态链接库，例如名为: webpack.config.dll.js，这里我们将 react 和 react-dom 单独打包成一个动态链接库。
``` 
//webpack.config.dll.js
const webpack = require('webpack');
const path = require('path');

module.exports = {
    entry: {
        react: ['react', 'react-dom']
    },
    mode: 'production',
    output: {
        filename: '[name].dll.[hash:6].js',
        path: path.resolve(__dirname, 'dist', 'dll'),
        library: '[name]_dll' //暴露给外部使用
        //libraryTarget 指定如何暴露内容，缺省时就是 var
    },
    plugins: [
        new webpack.DllPlugin({
            //name和library一致
            name: '[name]_dll', 
            path: path.resolve(__dirname, 'dist', 'dll', 'manifest.json') //manifest.json的生成路径
        })
    ]
}
```
package.json 的 scripts 中增加:
``` 
{
    "scripts": {
        "dev": "NODE_ENV=development webpack-dev-server",
        "build": "NODE_ENV=production webpack",
        "build:dll": "webpack --config webpack.config.dll.js"
    },
}
```
npm run build:dll，可以看到 dist 目录如下，之所以将动态链接库单独放在 dll 目录下，主要是为了使用 CleanWebpackPlugin 更为方便的过滤掉动态链接库。  
manifest.json 用于让 DLLReferencePlugin 映射到相关依赖上。  
webpack 的主配置文件: webpack.config.js 的配置：  
``` 
//webpack.config.js
const webpack = require('webpack');
const path = require('path');
module.exports = {
    //...
    devServer: {
        contentBase: path.resolve(__dirname, 'dist')
    },
    plugins: [
        new webpack.DllReferencePlugin({
            manifest: path.resolve(__dirname, 'dist', 'dll', 'manifest.json')
        }),
        new CleanWebpackPlugin({
            cleanOnceBeforeBuildPatterns: ['**/*', '!dll', '!dll/**'] //不删除dll目录
        }),
        //...
    ]
}
```
## 抽离公共代码
抽离公共代码是对于多页应用来说的，如果多个页面引入了一些公共模块，那么可以把这些公共的模块抽离出来，单独打包。公共代码只需要下载一次就缓存起来了，避免了重复下载。  
配置 optimization.splitChunks
``` 
//webpack.config.js
module.exports = {
    optimization: {
        splitChunks: {//分割代码块
            cacheGroups: {
                vendor: {
                    //第三方依赖
                    priority: 1, //设置优先级，首先抽离第三方模块
                    name: 'vendor',
                    test: /node_modules/,
                    chunks: 'initial',
                    minSize: 0,
                    minChunks: 1 //最少引入了1次
                },
                //缓存组
                common: {
                    //公共模块
                    chunks: 'initial',
                    name: 'common',
                    minSize: 100, //大小超过100个字节
                    minChunks: 3 //最少引入了3次
                }
            }
        }
    }
}
```
即使是单页应用，同样可以使用这个配置，例如，打包出来的 bundle.js 体积过大，可以将一些依赖打包成动态链接库，然后将剩下的第三方依赖拆出来。这样可以有效减小 bundle.js 的体积大小。  
runtimeChunk 的作用是将包含 chunk 映射关系的列表从 main.js 中抽离出来，在配置了 splitChunk 时，记得配置 runtimeChunk.
``` 
module.exports = {
    //...
    optimization: {
        runtimeChunk: {
            name: 'manifest'
        }
    }
}
```
最终构建出来的文件中会生成一个 manifest.js 
**借助 webpack-bundle-analyzer 进一步优化**  
在做 webpack 构建优化的时候，vendor 打出来超过了1M，react 和 react-dom 已经打包成了DLL  
借助 webpack-bundle-analyzer 查看一下是哪些包的体积较大
安装依赖
``` 
npm install webpack-bundle-analyzer -D
```
修改配置
``` 
//webpack.config.prod.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.config.base');
module.exports = merge(baseWebpackConfig, {
    //....
    plugins: [
        //...
        new BundleAnalyzerPlugin(),
    ]
})
```
npm run build 构建，会默认打开： http://127.0.0.1:8888/，可以看到各个包的体积  
进一步对 vendor 进行拆分，将 vendor 拆分成了4个(使用 splitChunks 进行拆分即可)。  
``` 
module.exports = {
    optimization: {
    concatenateModules: false,
    splitChunks: {//分割代码块
      maxInitialRequests:6, //默认是5
      cacheGroups: {
        vendor: {
          //第三方依赖
          priority: 1,
          name: 'vendor',
          test: /node_modules/,
          chunks: 'initial',
          minSize: 100,
          minChunks: 1 //重复引入了几次
        },
        'lottie-web': {
          name: "lottie-web", // 单独将 react-lottie 拆包
          priority: 5, // 权重需大于`vendor`
          test: /[\/]node_modules[\/]lottie-web[\/]/,
          chunks: 'initial',
          minSize: 100,
          minChunks: 1 //重复引入了几次
        },
        //...
      }
    },
  },
}
```
## webpack自身的优化
**tree-shaking**  
如果使用ES6的import 语法，那么在生产环境下，会自动移除没有使用到的代码。
**scope hosting 作用域提升**  
变量提升，可以减少一些变量声明。在生产环境下，默认开启。  
speed-measure-webpack-plugin 和 HotModuleReplacementPlugin 不能同时使用，否则会报错
**babel 配置的优化**  
不配置 @babel/plugin-transform-runtime 时，babel 会使用很小的辅助函数来实现类似 _createClass 等公共方法。默认情况下，它将被注入(inject)到需要它的每个文件中。但是这样的结果就是导致构建出来的JS体积变大。  
使用 @babel/plugin-transform-runtime，@babel/plugin-transform-runtime 是一个可以重复使用 Babel 注入的帮助程序，以节省代码大小的插件。  
在 .babelrc 中增加 @babel/plugin-transform-runtime 的配置
``` 
{
    "presets": [],
    "plugins": [
        [
            "@babel/plugin-transform-runtime"
        ]
    ]
}
```

原文:  
[带你深度解锁Webpack系列(优化篇)](https://juejin.cn/post/6844904093463347208)
