# webpack进阶
## 静态资源拷贝
在 public/index.html 中引入了 public 目录下的 js 或 css 文件。这个时候，如果直接打包，那么在构建出来之后，肯定是找不到对应的 js / css 了。
``` 
├── public
│   ├── config.js
│   ├── index.html
│   ├── js
│   │   ├── base.js
│   │   └── other.js
│   └── login.html
```
在 index.html 中引入了 ./js/base.js
``` 
<!-- index.html -->
<script src="./js/base.js"></script>
```
运行npm run dev，会发现有找不到该资源文件的报错信息
CopyWebpackPlugin，它的作用就是将单个文件或整个目录复制到构建目录。
安装依赖
``` 
npm install copy-webpack-plugin -D
```
修改配置(将 public/js 目录拷贝至 dist/js 目录)
``` 
//webpack.config.js
const CopyWebpackPlugin = require('copy-webpack-plugin');
module.exports = {
    //...
    plugins: [
        new CopyWebpackPlugin([
            {
                from: 'public/js/*.js',
                to: path.resolve(__dirname, 'dist', 'js'),
                flatten: true,
            },
            //还可以继续配置其它要拷贝的文件
        ])
    ]
}
```
flatten 这个参数，设置为 true，那么它只会拷贝文件，而不会把文件夹路径都拷贝上  
要拷贝一个目录下的很多文件，但是想过滤掉某个或某些文件，那么 CopyWebpackPlugin 还为我们提供了 ignore 参数。
``` 
//webpack.config.js
const CopyWebpackPlugin = require('copy-webpack-plugin');
module.exports = {
    //...
    plugins: [
        new CopyWebpackPlugin([
            {
                from: 'public/js/*.js',
                to: path.resolve(__dirname, 'dist', 'js'),
                flatten: true,
            }
        ], {
            ignore: ['other.js']
        })
    ]
}
```
这里忽略掉 js 目录下的 other.js 文件，使用 npm run build 构建， dist/js 下不会出现 other.js 文件。
## ProvidePlugin
ProvidePlugin 的作用就是不需要 import 或 require 就可以在项目中到处使用  
ProvidePlugin 是 webpack 的内置插件，使用方式如下：
``` 
new webpack.ProvidePlugin({
  identifier1: 'module1',
  identifier2: ['module2', 'property2']
});
```
默认寻找路径是当前文件夹 ./** 和 node_modules，当然啦可以指定全路径。  
React 使用的时候，要在每个文件中引入 React，不然立刻抛错。还有就是 jquery, lodash 这样的库，可能在多个文件中使用，但是懒得每次都引入，修改下 webpack 的配置:
``` 
const webpack = require('webpack');
module.exports = {
    //...
    plugins: [
        new webpack.ProvidePlugin({
            React: 'react',
            Component: ['react', 'Component'],
            Vue: ['vue/dist/vue.esm.js', 'default'],
            $: 'jquery',
            _map: ['lodash', 'map']
        })
    ]
}
```
这样配置之后，就可以在项目中随心所欲的使用 $、_map了，并且写 React 组件时，也不需要 import React 和 Component 了，还可以把 React 的 Hooks 都配置在这里。  
Vue 的配置后面多了一个 default，这是因为 vue.esm.js 中使用的是 export default 导出的，对于这种，必须要指定 default。React 使用的是 module.exports 导出的，因此不要写 default。  
项目启动了 eslint 的话，修改下 eslint 的配置文件，增加以下配置
``` 
{
    "globals": {
        "React": true,
        "Vue": true,
        //....
    }
}
```
## 抽离CSS
有些时候，可能会有抽离CSS的需求，即将CSS文件单独打包，这可能是因为打包成一个JS文件太大，影响加载速度，也有可能是为了缓存  
安装 loader
``` 
npm install mini-css-extract-plugin -D
```
> mini-css-extract-plugin 和 extract-text-webpack-plugin 相比:
1. 异步加载
2. 不会重复编译(性能更好)
3. 更容易使用
4. 只适用CSS

修改配置文件
``` 
//webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = {
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name].css'
            //将css文件放在单独目录下
            //publicPath:'../'   //如果你的output的publicPath配置的是 './' 这种相对路径，那么如果将css文件放在单独目录下，记得在这里指定一下publicPath 
        })
    ],
    module: {
        rules: [
            {
                test: /\.(le|c)ss$/,
                use: [
                    MiniCssExtractPlugin.loader, //替换之前的 style-loader
                    'css-loader', {
                        loader: 'postcss-loader',
                        options: {
                            plugins: function () {
                                return [
                                    require('autoprefixer')({
                                        "overrideBrowserslist": [
                                            "defaults"
                                        ]
                                    })
                                ]
                            }
                        }
                    }, 'less-loader'
                ],
                exclude: /node_modules/
            }
        ]
    }
}
```
重新编译：npm run build，目录结构如下所示:
``` 
├── dist
│   ├── assets
│   │   ├── alita_e09b5c.jpg
│   │   └── thor_e09b5c.jpeg
│   ├── css
│   │   ├── index.css
│   │   └── index.css.map
│   ├── bundle.fb6d0c.js
│   ├── bundle.fb6d0c.js.map
│   └── index.html
```
根目录下新建一个 .browserslistrc 文件，多个 loader 共享配置,内容如下
``` 
last 2 version
> 0.25%
not dead
```
修改webpack.config.js：
``` 
//webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = {
    //...
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name].css' 
        })
    ],
    module: {
        rules: [
            {
                test: /\.(c|le)ss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader', {
                        loader: 'postcss-loader',
                        options: {
                            plugins: function () {
                                return [
                                    require('autoprefixer')()
                                ]
                            }
                        }
                    }, 'less-loader'
                ],
                exclude: /node_modules/
            },
        ]
    }
}
```
**将抽离出来的css文件进行压缩**  
使用 mini-css-extract-plugin，CSS 文件默认不会被压缩，如果想要压缩，需要配置 optimization，首先安装 optimize-css-assets-webpack-plugin.
``` 
npm install optimize-css-assets-webpack-plugin -D
```
修改webpack配置：
``` 
//webpack.config.js
const OptimizeCssPlugin = require('optimize-css-assets-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    //....
    plugins: [
        new OptimizeCssPlugin()
    ],
}
```
将 OptimizeCssPlugin 直接配置在 plugins 里面，那么 js 和 css 都能够正常压缩，如果将这个配置在 optimization，那么需要再配置一下 js 的压缩  
抽离之后，修改 css 文件时，第一次页面会刷新，但是第二次页面不会刷新
## 按需加载
很多时候我们不需要一次性加载所有的JS文件，而应该在不同阶段去加载所需要的代码。webpack内置了强大的分割代码的功能可以实现按需加载。  
比如，在点击了某个按钮之后，才需要使用使用对应的JS文件中的代码，需要使用 import() 语法：
``` 
document.getElementById('btn').onclick = function() {
    import('./handle').then(fn => fn.default());
}
```
import() 语法，需要 @babel/plugin-syntax-dynamic-import 的插件支持，但是因为当前 @babel/preset-env 预设中已经包含了 @babel/plugin-syntax-dynamic-import，因此不需要再单独安装和配置。  
webpack 遇到 import(****) 这样的语法的时候，会这样处理：
- 以**** 为入口新生成一个 Chunk
- 当代码执行到 import 所在的语句时，才会加载该 Chunk 所对应的文件
## 热更新
1. 配置 devServer 的 hot 为 true
2. 在 plugins 中增加 new webpack.HotModuleReplacementPlugin()
``` 
//webpack.config.js
const webpack = require('webpack');
module.exports = {
    //....
    devServer: {
        hot: true
    },
    plugins: [
        new webpack.HotModuleReplacementPlugin() //热更新插件
    ]
}
```
HotModuleReplacementPlugin 之后，会发现，修改代码，仍然是整个页面都会刷新。不希望整个页面都刷新，还需要修改入口文件：
``` 
if(module && module.hot) {
    module.hot.accept()
}
```
## 多页应用打包
``` 
//webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
    entry: {
        index: './src/index.js',
        login: './src/login.js'
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[hash:6].js'
    },
    //...
    plugins: [
        new HtmlWebpackPlugin({
            template: './public/index.html',
            filename: 'index.html' //打包后的文件名
        }),
        new HtmlWebpackPlugin({
            template: './public/login.html',
            filename: 'login.html' //打包后的文件名
        }),
    ]
}
```
置多个 HtmlWebpackPlugin，那么 filename 字段不可缺省，否则默认生成的都是 index.html，如果你希望 html 的文件名中也带有 hash，那么直接修改 fliename 字段即可，例如: filename: 'login.[hash:6].html'。
``` 
├── dist
│   ├── 2.463ccf.js
│   ├── assets
│   │   └── thor_e09b5c.jpeg
│   ├── css
│   │   ├── index.css
│   │   └── login.css
│   ├── index.463ccf.js
│   ├── index.html
│   ├── js
│   │   └── base.js
│   ├── login.463ccf.js
│   └── login.html
```
，查看 index.html 和 login.html 会发现，都同时引入了 index.f7d21a.js 和 login.f7d21a.js，通常这不是我们想要的，希望，index.html 中只引入 index.f7d21a.js，login.html 只引入 login.f7d21a.js。  
HtmlWebpackPlugin 提供了一个 chunks 的参数，可以接受一个数组，配置此参数仅会将数组中指定的js引入到html文件中，此外，如果你需要引入多个JS文件，仅有少数不想引入，还可以指定 excludeChunks 参数，它接受一个数组。
``` 
//webpack.config.js
module.exports = {
    //...
    plugins: [
        new HtmlWebpackPlugin({
            template: './public/index.html',
            filename: 'index.html', //打包后的文件名
            chunks: ['index']
        }),
        new HtmlWebpackPlugin({
            template: './public/login.html',
            filename: 'login.html', //打包后的文件名
            chunks: ['login']
        }),
    ]
}
```
npm run build，可以看到 index.html 中仅引入了 index 的 JS 文件，而 login.html 中也仅引入了 login 的 JS 文件
## resolve 配置
resolve 配置 webpack 如何寻找模块所对应的文件
1. modules  
resolve.modules 配置 webpack 去哪些目录下寻找第三方模块，默认情况下，只会去 node_modules 下寻找，如果你我们项目中某个文件夹下的模块经常被导入，不希望写很长的路径，那么就可以通过配置 resolve.modules 来简化。
``` 
//webpack.config.js
module.exports = {
    //....
    resolve: {
        modules: ['./src/components', 'node_modules'] //从左到右依次查找
    }
}
```
这样配置之后，import Dialog from 'dialog'，会去寻找 ./src/components/dialog，不再需要使用相对路径导入。如果在 ./src/components 下找不到的话，就会到 node_modules 下寻找。
2. alias  
resolve.alias 配置项通过别名把原导入路径映射成一个新的导入路径
``` 
//webpack.config.js
module.exports = {
    //....
    resolve: {
        alias: {
            'react-native': '@my/react-native-web' //这个包名是我随便写的哈
        }
    }
}
```
3. extensions  
适配多端的项目中，可能会出现 .web.js, .wx.js，例如在转web的项目中，希望首先找 .web.js，如果没有，再找 .js。我们可以这样配置:
``` 
//webpack.config.js
module.exports = {
    //....
    resolve: {
        extensions: ['web.js', '.js'] //当然，你还可以配置 .json, .css
    }
}
```
``` 
import dialog from '../dialog';
```
首先寻找 ../dialog.web.js ，如果不存在的话，再寻找 ../dialog.js。这在适配多端的代码中非常有用，否则就需要根据不同的平台去引入文件(以牺牲了速度为代价)。  
配置 extensions，我们就可以缺省文件后缀，在导入语句没带文件后缀时，会自动带上extensions 中配置的后缀后，去尝试访问文件是否存在，因此要将高频的后缀放在前面，并且数组不要太长，减少尝试次数。如果没有配置 extensions，默认只会找对对应的js文件。
4.enforceExtensio  
resolve.enforceExtension 为 true，那么导入语句不能缺省文件后缀。
5.mainFields  
有一些第三方模块会提供多份代码，例如 bootstrap，可以查看 bootstrap 的 package.json 文件：
``` 
{
    "style": "dist/css/bootstrap.css",
    "sass": "scss/bootstrap.scss",
    "main": "dist/js/bootstrap",
}
```
resolve.mainFields 默认配置是 ['browser', 'main']，即首先找对应依赖 package.json 中的 brower 字段，如果没有，找 main 字段。  
如：import 'bootstrap' 默认情况下，找得是对应的依赖的 package.json 的 main 字段指定的文件，即 dist/js/bootstrap。  
假设希望，import 'bootsrap' 默认去找 css 文件的话，可以配置 resolve.mainFields 为:
``` 
//webpack.config.js
module.exports = {
    //....
    resolve: {
        mainFields: ['style', 'main'] 
    }
}
```
##  区分不同的环境
创建多个配置文件，如: webpack.base.js、webpack.dev.js、webpack.prod.
- webpack.base.js 定义公共的配置
- webpack.dev.js：定义开发环境的配置
- webpack.prod.js：定义生产环境的配置

webpack-merge 专为 webpack 设计，提供了一个 merge 函数，用于连接数组，合并对象
``` 
npm install webpack-merge -D
```
``` 
const merge = require('webpack-merge');
merge({
    devtool: 'cheap-module-eval-source-map',
    module: {
        rules: [
            {a: 1}
        ]
    },
    plugins: [1,2,3]
}, {
    devtool: 'none',
    mode: "production",
    module: {
        rules: [
            {a: 2},
            {b: 1}
        ]
    },
    plugins: [4,5,6],
});
//合并后的结果为
{
    devtool: 'none',
    mode: "production",
    module: {
        rules: [
            {a: 1},
            {a: 2},
            {b: 1}
        ]
    },
    plugins: [1,2,3,4,5,6]
}
```
webpack.config.base.js 中是通用的 webpack 配置，以 webpack.config.dev.js 为例，如下：
``` 
//webpack.config.dev.js
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.config.base');

module.exports = merge(baseWebpackConfig, {
    mode: 'development'
    //...其它的一些配置
});
```
修改我们的 package.json，指定对应的 config 
``` 
//package.json
{
    "scripts": {
        "dev": "cross-env NODE_ENV=development webpack-dev-server --config=webpack.config.dev.js",
        "build": "cross-env NODE_ENV=production webpack --config=webpack.config.prod.js"
    },
}
```
## 定义环境变量
使用 webpack 内置插件 DefinePlugin 来定义环境变量。DefinePlugin 中的每个键，是一个标识符.
- 如果 value 是一个字符串，会被当做 code 片段
- 如果 value 不是一个字符串，会被stringify
- 如果 value 是一个对象，正常对象定义即可
- 如果 key 中有 typeof，它只针对 typeof 调用定义
``` 
//webpack.config.dev.js
const webpack = require('webpack');
module.exports = {
    plugins: [
        new webpack.DefinePlugin({
            DEV: JSON.stringify('dev'), //字符串
            FLAG: 'true' //FLAG 是个布尔类型
        })
    ]
```
``` 
//index.js
if(DEV === 'dev') {
    //开发环境
}else {
    //生产环境
}
```
## 利用webpack解决跨域问题
前端在3000端口，服务端在4000端口，我们通过 webpack 配置的方式去实现跨域。
本地创建一个 server.js
``` 
let express = require('express');

let app = express();

app.get('/api/user', (req, res) => {
    res.json({name: 'ces'});
});

app.listen(4000);
```
执行代码(run code)，可以在浏览器中访问到此接口: http://localhost:4000/api/user。  
在 index.js 中请求 /api/user，修改 index.js 如下:  
``` 
//需要将 localhost:3000 转发到 localhost:4000（服务端） 端口
fetch("/api/user")
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(err => console.log(err));
```
**配置代理**
修改webpack配置
``` 
//webpack.config.js
module.exports = {
    //...
    devServer: {
        proxy: {
            "/api": "http://localhost:4000"
        }
    }
}
```
后端提供的接口并不包含 /api，即：/user，/info、/list 等，配置代理时不可能罗列出每一个api。  
修改我们的服务端代码  
尽管后端的接口并不包含 /api，我们在请求后端接口时，仍然以 /api 开头，在配置代理时，去掉 /api，修改配置:
``` 
//webpack.config.js
module.exports = {
    //...
    devServer: {
        proxy: {
            '/api': {
                target: 'http://localhost:4000',
                pathRewrite: {
                    '/api': ''
                }
            }
        }
    }
}
```
## 前端模拟数据
mocker-api 为 REST API 创建模拟 API。在没有实际 REST API 服务器的情况下测试应用程序时，它会很有用。
1. 安装 mocker-api:
``` 
npm install mocker-api -D
```
2. 新建mock文件夹，新建 mocker.js.文件
``` 
module.exports = {
    'GET /user': {name: 'ces'},
    'POST /login/account': (req, res) => {
        const { password, username } = req.body
        if (password === '888888' && username === 'admin') {
            return res.send({
                status: 'ok',
                code: 0,
                token: 'sdfsdfsdfdsf',
                data: { id: 1, name: 'ces' }
            })
        } else {
            return res.send({ status: 'error', code: 403 })
        }
    }
}
```
3, 修改 webpack.config.base.js
``` 
const apiMocker = require('mocker-api');
module.export = {
    //...
    devServer: {
        before(app){
            apiMocker(app, path.resolve('./mock/mocker.js'))
        }
    }
}
```
4. 重启 npm run dev

参考:
[带你深度解锁Webpack系列(进阶篇)](https://juejin.cn/post/6844904084927938567)
