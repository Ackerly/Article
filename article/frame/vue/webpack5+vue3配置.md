# webpack5+Vue3配置
## 集成Webpack5
webpack-dev-server 更新到 4 之后，很多配置有问题
- webpack-merge：webpack配置合并模块
- webpack：打包工具，又成构建工具,利用 watch 配置可以不使用webpack-dev-middleware插件
- webpack-cli：命令行工具
- webpack-dev-server：热加载【webpack 官方提供的一个小型 Express 服务器】
- 时间 2021/1/16 升级 webpack-dev-server 版本 3.11.0 与 webpack5 不兼容

**重大改变**  
devServer 移除 log, logLevel, logTime, noInfo, quiet, reporter and warn 选项等 // webpack.docschina.org/configurati… 移除了 quiet
## rule配置
### 配置写法规范eslint
- eslint-friendly-formatter: 可以让eslint的错误信息出现在终端上
- eslint eslint-loader:对js vue文件eslint校验规则
``` 
npm i eslint eslint-loader eslint-friendly-formatter -D
```
webpack配置
``` 
{
    test: /\.(js|vue)$/,
    loader: 'eslint-loader',
    enforce: 'pre',/* 前置解析，代码解析先过eslint */
    include: [[resolve('src'), resolve('test')]],/* 包括解析src test */
    options: {
      formatter: require('eslint-friendly-formatter')/* 可以让eslint的错误信息出现在终端上 */
  }
}
```
### 解析高级语法babel-loader
- babel-loader :为了解析 Es6 等高级语法
- @babel/core：babel的核心
- @babel/preset-env:一系列插件的集合,包含了我们在 babel6 中常用的 es2015,es2016, es2017 等最新的语法转化插件
``` 
npm i babel-loader @babel/core @babel/preset-env -D
```
webpack配置
``` 
 {
     test: /\.js$/,
     loader: 'babel-loader',
     exclude: /node_modules/,
     include: [resolve('src'), resolve('test')]
 }
```
.babelrc
``` 
{
    "presets": [
      "@babel/preset-env"
    ]
}
```
plugin插件
- html-webpack-plugin  
自动生产成本一个html文件
``` 
npm i html-webpack-plugin -D
```
- extract-text-webpack-plugin  
不想把样式打在脚本中，而是想独立 css 出来，然后在页面上外链 css，这时候我们需要 extract-text-webpack-plugin
``` 
npm i extract-text-webpack-plugin@v4.0.0-beta.0  –D
```
- CSS解析
``` 
npm i css-loader postcss-loader style-loader -D
```
- css添加浏览器前缀，压缩CSS，postcss-loader
    - postcss-import 版本太高需要先降等
    - postcss-url 版本太高需要先降等
    - autoprefixer 版本太高需要先降等
``` 
npm install postcss-loader postcss-url@v9.0.0 autoprefixer@v9.8.6 postcss-import@12.0.1 -D
```
- friendly-errors-webpack-plugin  
友好的错误提示插件，识别某些类别的 webpack 错误，并清理，聚合和优先级，以提供更好的开发人员体验
``` 
npm i friendly-errors-webpack-plugin@2.0.0-beta.2 -D
```
- node-notifier
系统级别的消息提醒
``` 
npm install --save node-notifier
```
- terser-webpack-plugin  
如果你使用的是 webpack v5 或以上版本，你不需要安装这个插件。webpack v5 自带最新的 terser-webpack-plugin
``` 
npm install terser-webpack-plugin --save-dev
```
- cache-loader
性能开销较大的 loader 前面添加 cache-loader，将结果缓存在磁盘中减少编译时间
``` 
npm install cache-loader --save-dev
```
## 安装Vue3
``` 
npm install vue@next vue-loader@next
```
## 总体配置
build/utils.js
``` 
/* 项目信息 */
const pkg = require('../package.json')
/* 该插件的主要是为了抽离css样式,防止将样式打包在js中引起页面样式加载错乱的现象 */
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const path = require('path')

/* 把资源文件放到目录下 */
exports.getAssetPath = function (filePath, options = {}) {
  return options.assetsDir
    ? path.posix.join(options.assetsDir, filePath)
    : filePath
}

/* 解析css */
exports.cssLoaders = function (options) {
  options = options || {}

  /* 解析.css文件 */
  const cssLoader = {
    loader: 'css-loader',
    options: {
      // https://github.com/vuejs/vue-style-loader/issues/50
      // https://github.com/vuejs/vue-style-loader#differences-from-style-loader
      // esModule: false, // 默认为true  vue-style-loader是style-loader的一层封装---要不使用style-loader，要不esModule: false
      // tips 暂且用style-loader来替代vue-style-loader
      sourceMap: options.sourceMap
    }
  }

  /* 解析前缀css */
  const postcssLoader = {
    loader: 'postcss-loader',
    options: {
      sourceMap: options.sourceMap
    }
  }

  // 生成用于提取文本插件的加载程序字符串
  const generateLoaders = (loader, loaderOptions) => {
    /* 是否使用解析前缀 */
    const loaders = options.usePostCSS
      ? [cssLoader, postcssLoader]
      : [cssLoader]
    if (loader) {
      /* 存在loader，往里面添加loader的解析 */
      loaders.push({
        loader: loader + '-loader',
        /* options的配置基础配置与传递参数配置合并 */
        options: Object.assign({}, loaderOptions, {
          sourceMap: options.sourceMap
        })
      })
    }

    // 当指定该选项时就会提取CSS到单独文件
    // (生产构建期间的情况)
    if (options.extract) {
      return ExtractTextPlugin.extract({
        use: loaders,
        fallback: 'style-loader' // 暂且用style-loader来替代vue-style-loader https://github.com/vuejs/vue-style-loader#differences-from-style-loader
      })
    } else {
      return ['style-loader'].concat(loaders) // 暂且用style-loader来替代vue-style-loader https://github.com/vuejs/vue-style-loader#differences-from-style-loader
    }
  }

  // https://vue-loader.vuejs.org/en/configurations/extract-css.html
  /* 只对生产应用CSS提取，以便在开发期间获得CSS热重新加载 */
  return {
    css: generateLoaders(),
    postcss: generateLoaders(),
    less: generateLoaders('less'),
    sass: generateLoaders('sass', { indentedSyntax: true }),
    scss: generateLoaders('sass'),
    stylus: generateLoaders('stylus')
  }
}

// 把.vue文件内部的style加载成独立css文件的 加载器
exports.styleLoaders = function (options) {
  const output = []
  /* 预处理css类型有stylus scss sass postcss css */
  const loaders = exports.cssLoaders(options)
  /* 把所有类型进行 */
  for (const extension in loaders) {
    const loader = loaders[extension]
    output.push({
      test: new RegExp('\\.' + extension + '$'),
      use: loader
    })
  }

  return output
}

/* 错误信息回调 */
exports.createNotifierCallback = function () {
  /* 系统级别的消息 */
  const notifier = require('node-notifier')

  return (severity, errors) => {
    /* 类型不是error就直接return */
    if (severity !== 'error') {
      return
    }
    /* 获取第一个错误 */
    const error = errors[0]

    const filename = error.file && error.file.split('!').pop()
    notifier.notify({
      title: pkg.name /* 项目名称 */,
      message: severity + ': ' + error.name /* 级别+错误信息 */,
      subtitle: filename || '' /* 错误文件 */,
      icon: path.join(__dirname, 'logo.png') /* logo */
    })
  }
}
```
build/webpack.base.conf.js
``` 
const webpack = require('webpack')
const path = require('path')
/* 个人配置信息 */
const config = require('../config')

/* 工具 */
const utils = require('./utils')

/* 配置合并 */
const { merge } = require('webpack-merge')

/* 基础配置 */
const baseWebpackConfig = require('./webpack.base.conf')

/* 模板 */
const HtmlWebpackPlugin = require('html-webpack-plugin')
/* 友好的错误提示插件 */
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')

/* 端口被占用 +1 */
const portfinder = require('portfinder')

const HOST = process.env.HOST
const PORT = process.env.PORT && Number(process.env.PORT)

/* 开发配置 */
const devWebpackConfig = merge(baseWebpackConfig, {
  /* https://github.com/vuejs/vue-style-loader/issues/50 */
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.dev.cssSourceMap
    })
  },
  // 开发模式如何生成source map
  devtool: config.dev.devtool,

  // these devServer options should be customized in /config/index.js
  /* 个人信息配置选项 */
  devServer: {
    /* 当使用 HTML5 History API 时，任意的 404 响应都可能需要被替代为 index.html */
    historyApiFallback: {
      rewrites: [
        {
          from: /.*/,
          to: path.posix.join(config.dev.assetsPublicPath, 'index.html')
        }
      ]
    },
    hot: true /* 启用 webpack 的模块热替换特性 */,
    compress: true /* 一切服务都启用gzip 压缩 */,
    host: HOST || config.dev.host /* 域名 */,
    port: PORT || config.dev.port /* 端口 */,
    open: config.dev.autoOpenBrowser /* 是否自动打开浏览器 */,
    overlay: config.dev.errorOverlay
      ? {
          warnings: false /* 警告不覆盖 */,
          errors: true /* 错误全屏覆盖 */
        }
      : false /* 当出现编译器错误或警告时，在浏览器中显示全屏覆盖。默认情况下禁用。如果您只想显示编译器错误 */,
    proxy: config.dev.proxyTable /* 跨域配置 */
  },
  // https://webpack.docschina.org/configuration/stats/#stats-presets  移除了quiet
  // 除了初始启动信息外，什么都不会写入控制台。 这也意味着来自webpack的错误或警告是不可见的。
  stats: {
    preset: 'errors-only'
  },
  // 在第一个错误出现时抛出失败结果，而不是容忍它。-- 这将迫使 webpack 退出其打包过程
  bail: true,
  // 在运行 webpack 时，通过使用 --progress 标志，来验证文件修改后，是否没有通知 webpack。如果进度显示保存，但没有输出文件，则可能是配置问题，而不是文件监视问题
  watchOptions: {
    poll: config.dev.poll,
    ignored: /node_modules/
  },
  plugins: [
    /* 环境设置为development */
    new webpack.DefinePlugin({
      'process.env': require('../config/dev.env'),
      // https://github.com/JeffreyWay/laravel-mix/issues/2514
      // https://github.com/vuejs/vue-next/tree/master/packages/vue#bundler-build-feature-flags
      __VUE_OPTIONS_API__: 'true',
      __VUE_PROD_DEVTOOLS__: 'false'
    }),

    new webpack.HotModuleReplacementPlugin() /* 开启webpack热更新功能 */,
    new FriendlyErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    })
  ],
  optimization: {
    splitChunks: {
      chunks: 'all'
    },
    /* 当开启 HMR 的时候使用该插件会显示模块的相对路径，建议用于开发环境。 */
    moduleIds: 'named' /* NamedModulesPlugin模块 迁移 */,
    /* webpack编译出错跳过报错阶段,在编译结束后报错 */
    emitOnErrors: true /* NoEmitOnErrorsPlugin模块迁移 */
  }
})

module.exports = new Promise((resolve, reject) => {
  /* 存在环境变量端口就是用环境变量的 或者是设置好的config */
  portfinder.basePort = process.env.PORT || config.dev.port
  portfinder.getPort((err, port) => {
    if (err) {
      reject(err)
    } else {
      // publish the new Port, necessary for e2e tests
      process.env.PORT = port
      // 替换端口
      devWebpackConfig.devServer.port = port

      // 添加有好的信息提示
      devWebpackConfig.plugins.push(
        new FriendlyErrorsPlugin({
          compilationSuccessInfo: {
            messages: [
              '写这段代码的时候，只有上帝和我知道它是干嘛的    \n    现在，只有上帝知道',
              `Your application is running here: http://${devWebpackConfig.devServer.host}:${port}`
            ]
          },
          onErrors: config.dev.notifyOnErrors
            ? utils.createNotifierCallback()
            : undefined,
          clearConsole: true
        })
      )
      /* 返回开发配置 */
      resolve(devWebpackConfig)
    }
  })
})
```
build/webpack.pro.conf.js
```
const path = require('path')

const webpack = require('webpack')
/* 配置 */
const config = require('../config')

/* 工具 */
const utils = require('./utils')

/* 合并配置 */
const { merge } = require('webpack-merge')
/* 基础配置 */
const baseWebpackConfig = require('./webpack.base.conf')
/* 赋值静态资源文件到其他目录 */
const CopyWebpackPlugin = require('copy-webpack-plugin')
/* HTML */
const HtmlWebpackPlugin = require('html-webpack-plugin')

/* 清理dist */
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

/* 压缩 */
const TerserPlugin = require('terser-webpack-plugin')

const webpackConfig = merge(baseWebpackConfig, {
  /* https://github.com/vuejs/vue-style-loader/issues/50 */
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.dev.cssSourceMap
    })
  },
  output: {
    filename: utils.getAssetPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.getAssetPath('js/[id].[chunkhash].js')
  },
  devtool: config.build.productionSourceMap
    ? config.build.devtool
    : false /* 是否生成SourceMap */,
  // https://webpack.docschina.org/configuration/stats/#stats-presets  移除了quiet
  // 除了初始启动信息外，什么都不会写入控制台。 这也意味着来自webpack的错误或警告是不可见的
  stats: {
    preset: 'errors-warnings'
  },
  /* 压缩js */
  optimization: {
    minimize: true,
    splitChunks: {
      chunks: 'all',
      // 待会直接自己试一下
      cacheGroups: {
        libs: {
          name: 'chunk-libs',
          test: /[\\/]node_modules[\\/]/,
          priority: 10,
          chunks: 'initial'
        },
        defaultVendors: {
          test: /\/src\//,
          name: 'rise',
          chunks: 'all',
          reuseExistingChunk: true
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    },
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true // 去除console.log
          }
        }
      })
    ]
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': require('../config/prod.env'),
      // https://github.com/JeffreyWay/laravel-mix/issues/2514
      // https://github.com/vuejs/vue-next/tree/master/packages/vue#bundler-build-feature-flags
      __VUE_OPTIONS_API__: 'true',
      __VUE_PROD_DEVTOOLS__: 'false'
    }),

    // see https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      },
      // 决定了 script 标签的引用顺序。默认有四个选项，'none', 'auto', 'dependency', '{function}'
      chunksSortMode: 'auto'
    }),

    // 复制自定义的静态文件
    new CopyWebpackPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, '../static'),
          to: config.build.assetsSubDirectory,
          globOptions: {
            ignore: ['.*']
          }
        }
      ]
    }),

    new CleanWebpackPlugin()
  ]
})

module.exports = webpackConfig
```
config/dev.env.js
``` 
const { merge } = require('webpack-merge')
const prodEnv = require('./prod.env')

module.exports = merge(prodEnv, {
  NODE_ENV: '"development"'
})
```
config/prod.env.js
``` 
module.exports = {
  NODE_ENV: '"production"'
}
```
config/index.js
``` 
const path = require('path')

module.exports = {
  /* 编译生产模式 */
  build: {
    // path
    assetsSubDirectory: 'static',
    assetsPublicPath: '/' /* 资源公共文件 */,

    assetsRoot: path.resolve(
      __dirname,
      '../dist'
    ) /* 输出文件夹--通常是build */,

    productionSourceMap: false, // 是否生成sourcemap
    devtool: 'source-map'
  },
  /* 编译开发模式 */
  dev: {
    // path
    assetsSubDirectory: 'static' /* 静态文件资源 */,
    assetsPublicPath: '/',

    /* 跨域 */
    proxyTable: {},
    // 服务器设置
    host: 'localhost', // 可以被process.env.HOST覆盖 【优先级比环境配置低】
    port: 8080, // 可以被process.env.PORT覆盖【优先级比环境配置低】，如果端口正在使用中，则将确定一个空闲端口
    autoOpenBrowser: false /* 是否自动打开浏览器 */,
    errorOverlay: true /* 代码错误是否覆盖全屏浏览器上 */,
    notifyOnErrors: true /* 开启错误提示 */,
    poll: false, // 监视文件 https://webpack.docschina.org/configuration/watch/

    showEslintErrorsInOverlay: true, // eslint错误是否显示在console终端上
    useEslint: true /* 是否使用eslint */,

    /**
     * Source Maps
     */
    devtool: 'eval-cheap-module-source-map' /* 开发者控制台的信息如何展示 */,

    /* CSS Sourcemaps */
    cssSourceMap: false /* 是否生成css文件map */
  }
}
```

参考:  
[webpack5都来了，还不学习配置一下webpack5+Vue3的配置嘛](https://juejin.cn/post/6922265673074737165?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
