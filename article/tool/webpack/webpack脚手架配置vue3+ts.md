<!-- TOC -->

- [用webpack脚手架配置vue3 + ts](#用webpack脚手架配置vue3--ts)
  - [安装webpack、vue3、ts](#安装webpackvue3ts)
  - [安装vue-loader支持vue文件的加载](#安装vue-loader支持vue文件的加载)
  - [配置webpack.config.js](#配置webpackconfigjs)
  - [组件模板分离](#组件模板分离)

<!-- /TOC -->
# 用webpack脚手架配置vue3 + ts
## 安装webpack、vue3、ts
``` 
npm install webpack vue-next typescript --save-dev
```
## 安装vue-loader支持vue文件的加载
``` 
npm install vue-loader
```
## 配置webpack.config.js
``` 
var path = require('path'),
    HtmlPlugin = require('html-webpack-plugin'),
    VueLoaderPlugin = require('vue-loader').VueLoaderPlugin;
    
module.exports = env => {
    return {
        entry: './src/bootstrap/main.ts',
        module: {
            rules: [
                {
                    test: /.vue$/,
                    use: 'vue-loader'
                },
                {
                    test: /.ts$/,
                    use: [{
                        loader: 'ts-loader',
                        options: {
                            appendTsSuffixTo: [/.vue$/],
                        }
                    }]
                }
            ]
        },
        plugins: [new VueLoaderPlugin()]
    }
}
```
ts-loader配置appendTsSuffixTo值为[/.vue$/]，新版本vue-loader需要配置plugins.  
添加typescript识别vue文件的vue.d.ts
``` 
declare module '*.vue' {
    import { ComponentOptions } from 'vue'
    const component: ComponentOptions;
    export default component;
}
```
## 组件模板分离
1、利用vue2的方式引入template
``` 
import { defineComponent } from 'vue';
export default defineComponent({
    template: require('app.html')
    // 或者
    render() {
        return require('app.html')
    }
})
```
将文件由字符串的方式引入进来，集成到组件的代码中，这样需要再webpack中配置
```
resolve:{
    alias:{
       'vue':'vue/dist/vue.esm.js'
    }
 }
```
2. vue-tsx-loader
安装
``` 
npm install vue-tsx-loader --save-dev
```
配置webpack
``` 
{
    test: /\.tsx$/,
    use: [
        'vue-loader', 'vue-tsx-loader?template=html'
    ]
}
```

