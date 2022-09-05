# babel知识点  
**困扰**  
在 babel@6 时候，最常收到反馈之一就是 regeneratorRuntime is not defined  
到了 babel@7，最常收到反馈之一 Cannot find module 'core-js/library/fn/**'.  
总结来讲：Babel 在编译大家的代码时候，会依据大家配置的 preset or plugin 注入一些模块依赖，而这些模块依赖是大家需要在 pkg.dependencies 里面体现出来的，否则很可能出现的问题就是加载不到具体的文件或者加载错误的版本的文件。  
根本的原因是什么：其实大家对  
- @babel/preset-env
- @babel/plugin-transform-runtime
- @babel/runtime
- core-js
- @babel/polyfills
- babel-polyfills

## 预备知识
**@babel/preset-env**  
preset-env 主要做的是转换 JavaScript 最新的 Syntax（指的是 const let ... 等）， 而作为可选项 preset-env 也可以转换 JavaScript 最新的 API （指的是比如 数组最新的方法 filter 、includes，Promise 等等）。  
- 在 babel@6 年代，我们使用的是 stage，那 stage 其实只会翻译 Syntax，而 API 则交给 babel-plugin-transform-runtime 或者 babel-polyfill 来实现。（这也是为什么大家在老项目中可以看到有引入 babel-polyfill 的原因）
- 在 babel@7 年代，我们废弃了 stage，使用的 preset-env，同时他也可以提供 polyfill 的能力

几个困惑:  
- preset-env 如何减小包体积的
- 有 preset-env polyfill 能力了，为啥还要有 @babel/plugin-transform-runtime，这是必须的吗
- 有了 preset-env polyfill 能力了，我还要 @babel/polyfill 吗

preset-env 的三个关键参数:  
- targets:  该参数决定了项目需要适配到的环境，比如可以申明适配到的浏览器版本，这样 babel 会根据浏览器的支持情况自动引入所需要的 polyfill  
- useBuiltIns("usage" | "entry" | false, defaults to false):  这个参数决定了 preset-env 如何处理 polyfills.  
  
    false: 这种方式下，不会引入 polyfills，你需要人为在入口文件处@import '@babel/polyfill';
但如上这种方式在 @babel@7.4 之后被废弃了，取而代之的是在入口文件处自行 import 如下代码
    ``` 
    import 'core-js/stable';
    import 'regenerator-runtime/runtime';
    ```
    不推荐采用 false，这样会把所有的 polyfills 全部打入，造成包体积庞大
- usage:  在项目的入口文件处不需要 import 对应的 polyfills 相关库。babel 会根据用户代码的使用情况，并根据 targets 自行注入相关 polyfills。
- entry:  在项目的入口文件处 import 对应的 polyfills 相关库
    ``` 
    import 'core-js/stable';
    import 'regenerator-runtime/runtime';
    ```
    此时 babel 会根据当前 targets 描述，把需要的所有的 polyfills 全部引入到你的入口文件
    
**corejs**  
corejs 并不是特殊概念，而是浏览器的 polyfill 都由它来管  
举个例子  
``` 
const one = Symbol('one');
```
Babel
``` 
"use strict";
require("core-js/modules/es.symbol.js");
require("core-js/modules/es.symbol.description.js");
require("core-js/modules/es.object.to-string.js");
var one = Symbol('one');
```
corejs-2 不会维护了，所有浏览器新 feature 的 polyfill 都会维护在 corejs-3 上。总结下：用 corejs-3，开启 proposals: true，proposals 为真那样我们就可以使用 proposals 阶段的 API 了。  

**总结**  
使用 preset-env 注入的 polyfill 是会污染全局的，但是如果是自己的应用其实是在可控的。所以这里推荐业务项目这么使用 .babelrc  
``` 
[
    {
        "presets": [
            [
                "@babel/preset-env",
                {
                    "targets": {
                        "chrome": "58"      
// 按自己需要填写
                    },
                    "useBuiltIns": "entry",
                    "corejs": {
                        "version": 3,
                        "proposals": true
                    }
                }
            ]
        ],
        "plugins": []
    }
]

// 入口文件代码
import 'core-js/stable';
import 'regenerator-runtime/runtime';
```
这样配置的原因是：targets 下设置我们业务项目所需要支持的最低环境配置，useBuiltIns 设置为 entry 为，将最低环境不支持的所有 polyfill 都引入到入口文件（即使你在你的业务代码中并未使用）。这是一种兼顾最终打包体积和稳妥的方式，因为很难保证引用的三方包有处理好 polyfill 这些问题。当然如果你能充分保证你的三方依赖 polyfill 处理得当，那么也可以把 useBuiltIns 设置为 usage。  
存在的两个问题：  
- 还是会有一定程度的代码重复
- 对项目，polyfill 会污染全局可以接受，但是作为 Library 我更希望它不会污染全局环境

babel/plugin-transform-runtime就是复用 babel 注入的关联代码，
如果是业务项目开发者：@babel/plugin-transform-runtime ，建议关闭 corejs，polyfill 的引入由 @babel/preset-env 完成，即开启 useBuiltIns  
``` 
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets": {
                    "chrome": 58
                },
                "useBuiltIns": "entry",
                "corejs": {
                    "version": 3,
                    "proposals": true
                }
            }
        ]
    ],
    "plugins": [
        [
            "@babel/plugin-transform-runtime",
            {
                "corejs": false
            }
        ]
    ]
}
```
如果是 Library 开发者：@babel/plugin-transform-runtime ，建议开启 corejs，polyfill 由 @babel/plugin-transform-runtime 引入。@babel/preset-env 关闭 useBuiltIns。  
``` 
{
    "presets": [
        [
            "@babel/preset-env"
        ]
    ],
    "plugins": [
        [
            "@babel/plugin-transform-runtime",
            {
                "corejs": {
                    "version": 3,
                    "proposals": true
                }
            }
        ]
    ]
}
```

原文: 
[99% 开发者没弄明白的 babel 知识](https://mp.weixin.qq.com/s/sJMydobsSxzxj2SECwcr_A)
