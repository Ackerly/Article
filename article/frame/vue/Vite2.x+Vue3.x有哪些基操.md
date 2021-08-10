# Vite2.x + Vue3.x有哪些常用的基操
## 注册组件
开发项目多数情况下使用组件库快速开发搭建我们的页面，为减小体积都采用按需加载的方式
``` 
import { createApp } from "vue";
import App from "./App.vue";

const app = createApp(App);

// 引入组件
import {
  Progress,
  Picker,
  Popup,
  ....
} from "vant";

// app.use() 注册
app.use(Progress)
   .use(Picker)
   .use(Picker)
   ...
   .mount("#app");
```
在使用组件越大越多的情况下会导致main.ts越来越大，基于模块化开发，最好单独封装到一个配置文件
``` 
import {
  Progress,
  Picker,
  Popup,
  ....
} from "vant";

const components = {
  Progress,
  Picker,
  Popup,
  ....
}

// .use 内部会自动寻找 install 方法进行调用注册挂载
const componentsInstallHandler = {
    install(app) {
        Object.keys(components).forEach(key => app.use(components[key]))
    }
}

export default componentsInstallHandler
```
main.ts中
``` 
// ...省略 createApp 引入

// 导入
import vantComponentsInstall from "./config/vant.config";

app
  .use(vantComponentsInstall)
  .mount("#app");
```
## 组件全局注册
自己编写一个<Header />组件想要全局使用，可以调用createApp返回的component函数进行全局注册。
``` 
import Header from '@/components/Header/Index.vue'

app.component('Header', Header)
```
## 过滤器
在3.x中，过滤器已删除，不在支持，可以使用方法调用或者计算属性代替
``` 
<template>
  <div>
      {{computedFunc('computed')}}
  </div>
</template>
<script lang="ts">
import { computed, defineComponent} from "vue";
export default defineComponent({
  setup() {
      // 使用 闭包 接受传递参数
      const computedFunc = () => {
          return val => val
      }
      return {
          computedFunc
      }
   }	
})
</script>
```
**全局过滤器**
如果多个组件中渲染数据都需要解密后才能正确显示，不可能每一个组件都写computed，可以通过全局属性使用
``` 
// main.ts
const app = createApp(App)

app.config.globalProperties.$filters = {
  decrypt(value) {
    return mdDecrypt(value)
},

<template>
  <p>{{ $filters.decrypt(encryptVal) }}</p>
</template>
```
## 自定义UI框架主题色
``` 
// vant定义主题色
import { defineConfig } from 'vite'

export default defineConfig({
  css: {
    preprocessorOptions: {
      less: {
        javascriptEnabled: true,
        // 需要自定义样式变量名称
        modifyVars: {           
          'blue': '#00C6B8',
          'gray-7': '#333333',
          'gray-4': '#00C6B8',
           ...
        },
      },
    },
  },
})
```
## 服务代理server
``` 
import { defineConfig } from 'vite'

export default defineConfig({
  server: {
    port: 3006, // 设置端口，默认 3000
    open: true, // 设置启动时，自动打开浏览器
    cors: true, // 允许跨域
    host:'10.x.x.x', // 本地 IP 访问，如何移动端手机调试就需要填写本地 IP 了
    proxy: {
      '/api': {
        target: 'http://10.x.x.x.x',
        changeOrigin: true,
        secure: false,
        rewrite: path => path.replace('/api/', '/')
      }
      // 配置多个继续添加
      '/api2': {
        ...
      }
    },
  },
})
```
## 配置别名
``` 
import vue from '@vitejs/plugin-vue'
import { defineConfig } from 'vite'

export default defineConfig({  
  resolve: {      
    alias: [
      {
        find: "@",
        replacement: resolve(__dirname, "src"),
      },
      {
        find: "@ASS",
        replacement: resolve(__dirname, "src/assets"),
      },
    ]
  },
    
  server: {
	....
  },
})
```
## 移动端rem自适应
使用 amfe-flexible 和 postcss-pxtorem 实现移动端适配  
**amfe-flexible**  
amfe-flexible 是配置克伸缩布局方案，主要将 1rem设为viewWidth/10。  
**postcss-pxtorem**  
postcss-pxtorem 是 postcss的插件，用于将像素单位生成 rem 单位。  
**postcss-pxtorem 配置**  
配置 postcss-pxtorem 方式有三种，可以在 vite.config.ts、.postcssrc.ts、postcss.copnfig.ts 其中之一配置即可。
``` 
// vite.config.ts 中配置
import { defineConfig } from 'vite'

export default defineConfig({
  css: {
    ... ,
    loaderOptions: {
      postcss: {
        plugins: [
          require('postcss-pxtorem')({
            rootValue: 37.5, // 根据设计稿宽度除以10进行设置，这边假设设计稿为375，即rootValue设为37.5,我这里设置 
            propList: ['*'] // // propList是设置需要转换的属性，这边*为所有都进行转换。
          })
        ]
      }
    }
  },
})
```
``` 
//  .postcssrc.ts 或者 postcss.copnfig.ts 配置
module.exports = {
  plugins: {
    autoprefixer: {
      overrideBrowserslist: [
        'Android 4.1',
        'iOS 7.1',
        'Chrome > 31',
        'ff > 31',
        'ie >= 8',
        'last 10 versions', // 所有主流浏览器最近10版本用
      ],
    },
    postcss-pxtorem: {
      rootValue: 37.5,
      propList: ['*'], 
    },
  },
}
```
## 环境变量
``` 
//  package.json 配置
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test:build": "vite build --mode test", 
  },
```
``` 
// URL统一管理 config.ts
export const DEV_BASE_URL = "https://xxx"
export const TEST_BASE_URL = "https://xxx"
export const PRO_BASE_URL = "https://xxx"
```
``` 
// axiospeizhi baseURL
import { DEV_BASE_URL, PRO_BASE_URL, TEST_BASE_URL, TIMEOUT } from "./config";

const envMap = {
  development: DEV_BASE_URL,
  production: PRO_BASE_URL,
  test: TEST_BASE_URL,
};

const base_URL = envMap[import.meta.env.MODE];

// axios
const instance = axios.create({
  baseURL: base_URL,
  timeout: 30000,
});
```
## 去除console.log
配置 configureWebpack去除
``` 
import { defineConfig, loadEnv, ConfigEnv, UserConfigExport } from "vite";

export default ({ command, mode }): UserConfigExport => {
  return defineConfig({
    //...其他配置
    build: {
      terserOptions: {
        compress: {
          drop_console: mode === "production",
        },
      },
    },
  });
};
```
配置文件中我们无法通过 import.meta.env 获取到运行环境变量，因为它是暴露再客户端源码中的，可以通过返回一个函数的方式，在函数接收的 mode 参数中获取到环境变量。
## hooks使用
``` 
import { reactive, toRefs } from "vue";

export default const useVerificationCode = () => {
    const state = reactive({
        isGetCode: false,
        codeText: '获取验证码',
        countDown: 60,
        phone: ''
    })
    // 点击后倒计时
    const countTime = () => {
        state.isGetCode = true
        let timer
        timer = setInterval(
            (function setIntervalFunc() {
                if (countDown !== 0) {
                    state.codeText = `重新发送${countDown.value--}`
                } else {
                    state.isGetCode = false
                    state.codeText = '获取验证码'
                    state.countDown = 60
                    clearInterval(timer)
                }
                return setIntervalFunc
            })(),
            1000
        )
    }
    // 获取验证码请求
    const onGetCode = async () => {
        const { success, message } = await getPhoneCode(encrypt(state.phone))
        if (success) {
            countTime()
            Toast.success(message)
        }
    }
    return {
        ...toRefs(state)
        onGetCode,
    }
}
```
拆分逻辑为单文件
``` 
// useVerificationCode.js
import { reactive, toRefs } from "vue";

export default const useVerificationCode = () => {
    const state = reactive({
        isGetCode: false,
        codeText: '获取验证码',
        countDown: 60,
        phone: ''
    })
    // 点击后倒计时
    const countTime = () => {
        state.isGetCode = true
        let timer
        timer = setInterval(
            (function setIntervalFunc() {
                if (countDown !== 0) {
                    state.codeText = `重新发送${countDown.value--}`
                } else {
                    state.isGetCode = false
                    state.codeText = '获取验证码'
                    state.countDown = 60
                    clearInterval(timer)
                }
                return setIntervalFunc
            })(),
            1000
        )
    }
    // 获取验证码请求
    const onGetCode = async () => {
        const { success, message } = await getPhoneCode(encrypt(state.phone))
        if (success) {
            countTime()
            Toast.success(message)
        }
    }
    return {
        ...toRefs(state)
        onGetCode,
    }
}

作者：ihoneys
链接：https://juejin.cn/post/6991441703979171871
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
优化后
``` 
// example.vue
<template>
  <button @click="onGetCode">{{ codeText }}</button>
</template>
<script lang="ts">
import { defineComponent } from "vue";
// 引入 useVerificationCode.js
import useVerificationCode from "./useVerificationCode.js";
export default defineComponent({
  setup(props, ctx) {
      // 获取验证码
     const { onGetCode, isGetCode, codeText, countDown, phone } = useVerificationCode()
     
     // ...
     // ...
    return {
      onGetCode, 
      isGetCode, 
      codeText, 
      countDown, 
      phone
    }
  }
})
</script>

```
参考:
[总结 Vite2.x + Vue3.x 有哪些常用的基操｜ 8月更文挑战](https://juejin.cn/post/6991441703979171871?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
