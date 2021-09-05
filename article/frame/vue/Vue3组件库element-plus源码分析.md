# Vue3组件库Element-plus源码分析
## Typescript相关
element-plus 引入了 typescript, 除了配置对应的 eslint 校验规则、插件，定义 tsconfig.json 之外，打包 es-module 格式组件库的时候的时候使用到了一些 rollup 插件。
- @rollup/plugin-node-resolve
- rollup-plugin-terser
- rollup-plugin-typescript2
- rollup-plugin-vue
``` 
// build/rollup.config.bundle.js
import { nodeResolve } from '@rollup/plugin-node-resolve'
import { terser } from 'rollup-plugin-terser'
import typescript from 'rollup-plugin-typescript2'
const vue = require('rollup-plugin-vue')
export default [{
  // ... 省略前面部分内容 
  plugins: [
   terser(),
   nodeResolve(),
   vue({
     target: 'browser',
     css: false,
     exposeFilename: false,
   }),
   typescript({
     tsconfigOverride: {
       'include': [
         'packages/**/*',
         'typings/vue-shim.d.ts',
       ],
       'exclude': [
         'node_modules',
         'packages/**/__tests__/*',
       ],
     },
   }),
 ],
}]
```
@rollup/plugin-node-resolve 打包依赖的 npm 包  
rollup-plugin-terser 压缩代码  
rollup-plugin-vue 打包 vue 文件, css 样式交给了后续会提到的 gulp 来处理。  
rollup-plugin-typescript2 是用了编译 typescript 的, 配置中排除了 node-modules 和测试相关文件， include 除了包含组件实现，还包含了 typings/vue-shim.d.ts 文件。  
插件中使用到的 typings/vue-shim.d.ts 类型声明文件( 以 .d.ts 结尾的文件会被自动解析)，定义了一些全局的类型声明，可以直接在 ts 或者 vue 文件中使用这些类型约束变量。还使用扩展模板对 import XX from XX.vue 的引入变量给出类型提示。
``` 
// typings/vue-shim.d.ts
declare module '*.vue' {
  import { defineComponent } from 'vue'
  const component: ReturnType<typeof defineComponent>
  export default component
}
declare type Nullable<T> = T | null;
declare type CustomizedHTMLElement<T> = HTMLElement & T
declare type Indexable<T> = {
  [key: string]: T
}
declare type Hash<T> = Indexable<T>
declare type TimeoutHandle = ReturnType<typeof global.setTimeout>
declare type ComponentSize = 'large' | 'medium' | 'small' | 'mini'
```
除了 d.ts 文件之外，element-plus 中对于 props 的类型声明使用了 vue3 的 propType。以 下面的 Alert 为例， 使用了 PropType 的 props 类型会执行符合我们自定义的规则的构造函数，然后结合 typescript 做类型校验。其他非 props 中的类型声明则是使用了 interface 。
``` 
import { PropType } from 'vue'
export default defineComponent({
  name: 'ElAlert',
  props: {
    type: {
      type: String as PropType<'success' | 'info' | 'error' | 'warning'>,
      default: 'info',
    }
  }
})
```
## Composition API
除了使用新的 Composition API 来改写组件之外，element-plus 中 packages/hooks 目录下抽取了几个可复用的 hooks 文件。  
以 autocomplete, input 等控件使用到的 use-attrs 为例, 主要做的事情是继承绑定的属性和事件，类似于 $attrs 和 $listener 功能，但是做了一些筛选，去掉了一些不需要继承的属性和事件绑定。
``` 
watchEffect(() => {
    const res = entries(instance.attrs)
      .reduce((acm, [key, val]) => {
        if (
          !allExcludeKeys.includes(key) &&
          !(excludeListeners && LISTENER_PREFIX.test(key))
        ) {
          acm[key] = val
        }
        return acm
      }, {})
    attrs.value = res
  })
```
Vue3 中仍然保留了 mixin，我们可以在特定组件或者是全局使用 mixin 来复用逻辑，同时也引入了 hooks 来改善 mixin 存在的一些问题
1. 渲染上下文中公开的属性的来源不清楚。 例如，当使用多个 mixin 读取组件的模板时，可能很难确定从哪个 mixin 注入了特定的属性。
2. 命名空间冲突。 Mixins 可能会在属性和方法名称上发生冲突

Hooks带来的好处
1. 暴露给模板的属性具有明确的来源，因为它们是从 Hook 函数返回的值。
2. Hook 函数返回的值可以任意命名，因此不会发生名称空间冲突。

## Teleport的使用
element-plus 对几个挂载类组件使用了 vue3 的新特性 Teleport，这个新特性可以帮我们把其包裹的元素移动到我们指定的节点下。  
Teleport 提供了一种干净的方法，允许我们控制在 DOM 中哪个父节点下呈现 HTML，而不必求助于全局状态或将其拆分为两个组件。  
Dialog，Drawer，以及使用了 Popper 的 Tooltip 和 Popover 都新增了一个 append-to-body 属性。以 Dialog 为例：  
appendToBody 为 false， Teleport 会被 disabled, DOM 还是在当前位置渲染，当为 true 时， dialog 中的内容放到了 body 下面。
``` 
<template>
  <teleport to="body" :disabled="!appendToBody">
    <transition
      name="dialog-fade"
      @after-enter="afterEnter"
      @after-leave="afterLeave"
    >
        ...
     </transition>
  </teleport>
</tamplate>
```
在原来的 element-ui 中，Tooltip  和 Popover  也是直接放在了 body 中，原来是通过 vue-popper.js 来使用 document.body.appendChild 来添加元素到 body 下的，element-plus 使用 Teleport 来实现相关逻辑。
## 全局 API - 实例 API
Vue 2.x中Vue.component 方法绑定全局组件，Vue.use 绑定全局自定义指令，Vue.prototype 绑定全局变量和全局方法
``` 
const install = function(Vue, opts = {}) {
  locale.use(opts.locale);
  locale.i18n(opts.i18n);
  
  // Vue.component 方法绑定全局组件
  components.forEach(component => {
    Vue.component(component.name, component);
  });
  
  // Vue.use 绑定全局自定义指令
  Vue.use(InfiniteScroll);
  Vue.use(Loading.directive);
 
  // Vue.prototype 绑定全局变量和全局方法
  Vue.prototype.$ELEMENT = {
    size: opts.size || '',
    zIndex: opts.zIndex || 2000
  };
  Vue.prototype.$loading = Loading.service;
  Vue.prototype.$msgbox = MessageBox;
  Vue.prototype.$alert = MessageBox.alert;
  Vue.prototype.$confirm = MessageBox.confirm;
  Vue.prototype.$prompt = MessageBox.prompt;
  Vue.prototype.$notify = Notification;
  Vue.prototype.$message = Message;
};
```
 vue 3.0 中，任何全局改变 Vue 行为的 API 现在都会移动到应用实例上，也就是 createApp 产生的 app
 ``` 
 import type { App } from 'vue'
 const plugins = [
   ElInfiniteScroll,
   ElLoading,
   ElMessage,
   ElMessageBox,
   ElNotification,
 ]
 const install = (app: App, opt: InstallOptions): void => {
   const option = Object.assign(defaultInstallOpt, opt)
   use(option.locale)
   app.config.globalProperties.$ELEMENT = option    // 全局设置默认的size属性和z-index属性
   // 全局注册所有除了plugins之外的组件
   components.forEach(component => {
     app.component(component.name, component)
   })
   plugins.forEach(plugin => {
     app.use(plugin as any)
   })
 }
 ```
 消息类组件添加 $ 全局方法在 element-plus 中被移动到了 index.ts 里面， 几个消息通知类型的组件都放到了 plugins，使用 app.use 会调用对应组件 index.ts 中的 install 方法
 ``` 
 (Message as any).install = (app: App): void => {
   app.config.globalProperties.$message = Message
 }
 ```
 ## 国际化
 packages 下有一个 locale 文件夹，控制语言切换 packages/locale/index.ts 中抛出了 2 个方法，方法 t 和方法 use, t 控制 vue 文件中文本的翻译替换，use 方法修改全局语言
 ``` 
 // packages/locale/index.ts
 export const t = (path:string, option?): string => {
   let value
   const array = path.split('.')
   let current = lang
   for (let i = 0, j = array.length; i < j; i++) {
     const property = array[i]
     value = current[property]
     if (i === j - 1) return template(value, option)
     if (!value) return ''
     current = value
   }
   return ''
 }
 ```
 在vue文件中引入locale中的t方法
 ``` 
 import { t } from '@element-plus/locale'
 ```
 然后就可以在 template 使用多语言 key 值了，例如：label="t('el.datepicker.nextMonth')" ，t 方法会帮你找到对应的语言文件中的对应值。  
 抛出的 use 方法可以设置全局语言种类，也修改 day.js 的语言配置。 element-plus 中引入了 day.js 替换原来的 moment.js 来做时间的格式化和时区信息等的处理。  
 ``` 
 export const use = (l): void => {
   lang = l || lang
   if (lang.name) {
     dayjs.locale(lang.name)
   }
 }
 ```
 ## Website打包
 website, 也就是文档网站，提供各个控件的使用示例。  
 引入了 packages/element-plus/index.ts 文件，然后就可以在 md 中使用 packages 中的各个组件了，组件逻辑修改也可以立即生效。  
 和 element-ui 一致，element-plus 的 website dev 起服务和打包都是用的 webpack，使用到了 vue-loader 来处理 vue 文件，使用 babel-loader 处理 js/ts 文件，样式文件和字体图标分别使用了对应的 css-loader， url-loader 等。
 文档展示主要的 md 文件，使用了 website/md-loader/index.js 自己实现的 md-loader, 分别从 md 中提取出 <template> 和 <script> 内容 ，将 md 中的组件示例转化成了 vue 的字符串，然后再通过 vue-loader 来处理。
 ``` 
 rules: [
   {
     test: /\.vue$/,
     use: 'vue-loader',
   },
   {
     test: /\.(ts|js)x?$/,
     exclude: /node_modules/,
     loader: 'babel-loader',
   },
   {
     test: /\.md$/,
     use: [
       {
         loader: 'vue-loader',
         options: {
           compilerOptions: {
             preserveWhitespace: false,
           },
         },
       },
       {
         loader: path.resolve(__dirname, './md-loader/index.js'),
       },
     ],
   },
   {
     test: /\.(svg|otf|ttf|woff2?|eot|gif|png|jpe?g)(\?\S*)?$/,
     loader: 'url-loader',
     // todo: 这种写法有待调整
     query: {
       limit: 10000,
       name: path.posix.join('static', '[name].[hash:7].[ext]'),
     },
   },
 ]
 ```
## 组件库和样式打包
 element-plus 的打包命令有这么一长串，其中 yarn build:lib 和 yarn build:lib-full 是用到了 webpack 打 umd 格式的全量包。其余的则是分别使用到了 rollup 和 gulp。
 ``` 
 "build": "yarn bootstrap && yarn clean:lib && yarn build:esm-bundle && yarn build:lib && yarn build:lib-full && yarn build:esm && yarn build:utils && yarn build:locale && yarn build:locale-umd && yarn build:theme"
 ```
### 使用 rollup 打包组件 bundle
 rollup 相关的逻辑在 build/rollup.config.bundle.js 文件中
 入口为 export 所有组件的 /packages/element-plus/index.ts , 采用 es-module 规范最终打包到 lib/index.esm.js 中。由于打包时使用了 Typescript 插件，最后生成的文件除了全量的 index.esm.js，还有每个组件单独的 lib 文件。
 ``` 
 // build/rollup.config.bundle.js
 export default [
   {
     input: path.resolve(__dirname, '../packages/element-plus/index.ts'),
     output: {
       format: 'es',    // 打包格式为 es，可选cjs(commonJS) ,umd 等
       file: 'lib/index.esm.js',
     },
     external(id) {
       return /^vue/.test(id)
         || deps.some(k => new RegExp('^' + k).test(id))
     },
   }
 ]
 ```
 ### 使用 gulp 打包样式文件和字体图标
 和 element-ui 一样，样式文件和字体图标的打包使用的是 packages/theme-chalk/gulpfile.js, 把每个 scss 文件打包成单独的 css, 其中包含了通用的 base 样式，还有每个组件的样式文件。
 ``` 
 // packages/theme-chalk/gulpfile.js
 function compile() {
   return src('./src/*.scss')
     .pipe(sass.sync())
     .pipe(autoprefixer({ cascade: false }))
     .pipe(cssmin())
     .pipe(rename(function (path) {
       if(!noElPrefixFile.test(path.basename)) {
         path.basename = `el-${path.basename}`
       }
     }))
     .pipe(dest('./lib'))
 }
 function copyfont() {
   return src('./src/fonts/**')
     .pipe(cssmin())
     .pipe(dest('./lib/fonts'))
 }
 ```
 再通过 npm script 中一些文件拷贝和删除操作，打包之后的样式和字体图标文件最终会放到 lib/theme-chalk 目录下。
 ``` 
 cp-cli packages/theme-chalk/lib lib/theme-chalk && rimraf packages/theme-chalk/lib
 ```
 ## 引入 lerna
 element-plus 采用了 lerna 进行包管理，lerna 可以负责 element-plus 版本和组件版本管理，还可以将每个组件单独发布成 npm 包（不过 element-plus 目前 npm 上只有全量包, 单个组件的皮肤包和多语言文件现在也是放在了一个文件夹下而不是每个组件当中）。
 每个组件都有这样一个 package.json 文件
 ``` 
 {
   "name": "@element-plus/message",
   "version": "0.0.0",
   "main": "dist/index.js",
   "license": "MIT",
   "peerDependencies": {
     "vue": "^3.0.0"
   },
   "devDependencies": {
     "@vue/test-utils": "^2.0.0-beta.3"
   }
 }
 ```
 使用了 workspaces 匹配 packages 目录，依赖会统一放在根目录下的 node-modules，而不是每个组件下都有，这样相同的依赖可以复用，目录结构也更加清晰。
 ``` 
   // package.json
   "workspaces": [
     "packages/*"
   ]
 ```
 lement-plus 的 script 中还提供了一个 shell 脚本用于开发新组件的时候生成基础文件, 使用 npm run gen 可以在 packages 下生成一个基础的组件文件夹。
 ``` 
 "gen": "bash ./scripts/gc.sh"
 ```
 
参考:
[Vue 3 组件库：element-plus 源码分析](https://juejin.cn/post/6914598983205847053?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
