# 如何从0设计开发一款JS-SDK.md
前端SDK其实很常见了，比如：  
UI组件库：通过封装一系列组件，通过配置帮助开发者调用  
- Antd
- ElementUI

JS类库：通过实现一类常用的方法，便于开发处理数据，也不用再考虑兼容性
- lodash
- moment

JS类库：通过实现一类常用的方法，便于开发处理数据，也不用再考虑兼容性  
- lodash
- moment

监控统计工具：通过API，来监听前端系统的报错、统计数据  
- Sentry
- 百度统计等

## 开发前的设计
### 设计原则
**满足一类功能的需要**  
SDK一般都是为了满足一类业务的需要，所以设计之初要明确业务范围  

**最小可用性原则**  
即能用确定的方法实现，就不要再去搞复杂的内容。比如获取DOM，如果GetElementById可以实现，就不要再设计一下GetElementsByTagName、 document.querySelector等方法封装，除非有其他的开发需要无法满足。  

**最少依赖原则**  
SDK减少依赖，要避免Lodash、JQuery、Moment、Dayjs等库，尽可能自行实现必要的方法，或者引入尽量小的库。否则会导致SDK打包后过大，或者更新版本带来的兼容问题
一切都要根据实际情况，有些SDK是时间的各种处理，自己处理时间的成本太高，不妨引入小型的Dayjs时间库  

**足够稳定、向后兼容**  
减少BreakChange，绝不能导致载体应用崩溃，同时做好文档说明

**易扩展**  
模块化实现方法，尽量小的封装函数，保持函数功能的单一性原则，这样就可以更好的增加SDK的能力  

### 要实现的功能
首先要明确我们写的SDK是用来做什么的？比如实现的是用户H5页面的一键登录和号码检测。那么需要暴露两个实例，供其他开发者使用，为了满足易扩展的原则，将声明两个类，来实现（如果每个实例都很多能力，可以拆分成两个SDK也是可以的）

### 构建工具和技术选择
提供的SDK一般都要提供压缩和未压缩版本，未压缩可以用来帮助开发调用，查找问题。压缩版本可以使用在生产环境，减少http损耗。所以要借助构建工具来集成这部分的能力。  
可供选择的压缩工具有很多：webpack、Rollup、Gulp。如果是纯类库的压缩，当然是Rollup更好，压缩更彻底，如果是有DOM和样式，那么使用webpack功能更强大

### 单元测试
SDK的设计原则有一条：足够稳定、向后兼容，最少依赖原则。
这就意味着要少写Bug，所以一定要引入单元测试  
``` 
describe('common test', () => {
    test('osIsPc', () => {
        expect(osIsPc()).toBeBoolean(true, false);
    })
    test("isWifi", () => {
        expect(isWifi()).toBeBoolean(true, false);
    })
})
```

### SDK支持的引入方式  
浏览器js模块化常见的几种方式包括：amd\cmd\es6 modules\umd  
**静态资源引入**  
``` 
<script src="/sdk/v1/phoneserver"></script>
```
**支持amd引入**  
``` 
define([jquery.js, lodash.js], function($, _){
    console.log("jquery and lodash", $, _)
})
```
**支持cmd引入**  
``` 
define(function(require){
    const lodash = require('./a.js')
    console.log("lodash", lodash)
})
```
**支持es6引入**  
``` 
import { PhoneServer } from 'phone-server-sdk'
```
在webpack中配置umd方式打包，然后就可以支持上面的多种引入方式  
``` 
output: {
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].js',
    library: 'Phone-JS-SDK',
    libraryTarget: 'umd'
}
```
### 版本维护和更新
- 管理好版本号
- 记录好更新日志

SDK版本更新，每个版本都会存在差异，而用户使用的版本肯定也太一样，所以记录好版本更新日志可以减少非技术问题。  
通过静态文件导出的SDK要同时部署多个版本，不能随时下线老版本。  

### 其他的注意点
- 代码混淆
- 开发环境配置和代码格式
- 上传NPM
- CDN部署
- 依赖的三方库如何打包进SDK
仅支持静态引入的库如何处理
如何全局共享库方法
- 针对有后端API交互的SDK，需要考虑
API要限流、限制次数、防止盗刷
日志监控和数据上报

## 项目实践
**项目需要实现的能力**  
- 构建工具构建，配置开发环境、ts配置
- 实现类库的相关方法（版本记录、帮助命令等）
- 实现一键登录的方法（预期的功能方法）
- 实现号码检测的方法
- 单元测试
- 上传npm，支持导入

**搭建基础架构，配置webpack**  
``` 
module.exports = {
    entry: {
        sdk: [path.resolve(__dirname, '../src/index.ts')]
    },
    resolve: {
        extensions: ['.tsx', '.ts', '.js'],
    },
    output: {
        path: path.resolve(__dirname, '../dist'),
        filename: '[name].js',
        library: 'SDK',
        libraryTarget: 'umd'
    },
    module: {
      rules: [
        {
          test: /\.ts?$/,
          exclude: /node_modules/,
          use: [
              {
                loader: 'babel-loader',
                options: {
                  presets: ['@babel/preset-env'],
                },
              },
              {
                loader: 'ts-loader',
                options: {
                  compilerOptions: {
                    noEmit: false,
                  },
                },
              },
            ],
        },
      ]
    }
```

**实现类库的相关方法**  
项目的主要目录结构  
- src源代码
- scripts 是webpack的相关配置
- public 是用来调试和打包的目录
- tests 单元测试

LibInfo.ts 用来实现库的一些方法，比如获取版本号，帮助文档，展示依赖版本等  
PhoneNumberLogin.ts 一键登录类  
PhoneNumberAuth.ts 号码认证类  
ajax.ts 简单封装的ajax请求  
index.ts 作为入口文件  
``` 
js-sdk                   
├─ __tests__                                       
│  └─ utils                     
│     ├─ ajax.test.js           
│     └─ commont.test.js                      
├─ public                       
│  ├─ index.html                
│  └─ sdk.js                    
├─ scripts                                     
│  ├─ webpack.base.config.js    
│  ├─ webpack.dev.config.js     
│  └─ webpack.prod.config.js    
├─ src                          
│  ├─ lib                       
│  │  ├─ LibInfo.ts             
│  │  ├─ PhoneNumberAuth.ts     
│  │  ├─ PhoneNumberLogin.ts    
│  │  └─ Init.ts                
│  ├─ utils                             
│  │  ├─ ajax.ts                
│  │  ├─ common.ts                       
│  │  └─ interface.ts           
│  └─ index.ts                  
├─ Readme.md                    
├─ index.d.ts                   
├─ jest.config.js               
├─ package.json                 
├─ tsconfig.json                          
└─ yarn.lock              
```

**实现SDK的接口**  
_在调用之前，我们需要引用第三方库，而且是md5加密的（如下），无法直接下载本地使用，所以考虑直接插入head中_  
``` 
export const scriptInit = (src: string, callback?: Function) => {
    const script:any = document.createElement('script'),
        fn = callback || function(){};
    script.type = 'text/javascript';
    //IE
    if(script.readyState){
        script.onreadystatechange = function(){
            if( script.readyState == 'loaded' || script.readyState == 'complete' ){
                script.onreadystatechange = null;
                fn();
            }
        };
    }else{
        //其他浏览器
        script.onload = function(){
            fn();
        };
    }
    script.src = src;
    document.getElementsByTagName('head')[0].appendChild(script);
}
```
_以实现意见登录号码为例，新建PhoneNumberLogin.ts_  
``` 
const loginPhoneUrl = `http://test.com`
export class PhoneNumberLogin {
    constructor(options:AppInfo){
        this.Init()
    }
    private Init(){
        // 引入第三方依赖的script
        scriptInit(loginPhoneUrl)
    } 
    // 处理一键登录的接口逻辑
    public LoginApp(options){
        return options
    }
}
```
这样每一个小的功能点都放在一个类中，不对外的设置为私有方法，对外的可以设置为公共方法，其他的通过引用就可以让SDK保持良好的可扩展性  
_在index.ts中抛出方法_  
``` 
export * from './lib/PhoneNumberLogin.ts'
```
_在项目中使用_  
script导入，一般都需要申请域名，那么就需要考虑容灾，防止一台机器挂掉，服务不可用，一般考虑CDN部署  
``` 
const { PhoneNumberLogin } = Phone-JS-SDK
const PhoneServer = new PhoneNumberLogin()
```
_ES6 Modules导入_  
``` 
const { PhoneNumberLogin } from "Phone-JS-SDK"
const PhoneServer = new PhoneNumberLogin()
```
_上传NPM_  



参考:  
[如何从0设计开发一款JS-SDK](https://juejin.cn/post/7111880557914488846)
