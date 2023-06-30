# 搭建自动化 Web 页面性能检测系统
## 技术选型
服务端框架选择的是 Nestjs，Web 页面选择的是 Vite + React。由于所在团队当前的研发环境已经全面接入自研的 devops 系统，易测收录的页面也是对接 devops 进行检测的。  

**整体架构设计**
易测的检测服务基于 Lighthouse + Puppeteer 实现。  

**实现方案**
易测的检测流程是：根据子产品的版本获取到待检测的地址、对应的登录地址、用户名和密码，然后通过 Puppeteer 先跳转到对应的登录页面，接着由 Puppeteer 输入用户名、密码、验证码，待登录完成后跳转至待检测的页面，再进行页面性能检测。如果登录后还在登录页，表示登录失败，则获取错误提示并抛出到日志。为了检测方便，检测的均为开发环境且将登录的验证码校验关闭。  

## Lighthouse
易测通过 Node 模块引入 Lighthouse，不需要登录的页面检测可以直接使用 Lighthouse，基础用法：  
```
const lighthouse = require('lighthouse');
const runResult = await lighthouse(url, lhOptions, lhConfig);
```

**options**   
lhOptions 的主要参数有：  
```
 {
   port: PORT, // chrome 运行的端口
   logLevel: 'error',
   output: 'html', // 以 html 文件的方式输出报告
   onlyCategories: ['performance'], // 仅采集 performance 数据
   disableStorageReset: true, // 禁止在运行前清除浏览器缓存和其他存储 API
 }
```

**config**  
lhConfig 的主要参数有：  
```
{
   extends: 'lighthouse:default', // 继承默认配置
   settings: {
     onlyCategories: ['performance'],
     // onlyAudits: ['first-contentful-paint'],
     formFactor: 'desktop',
     throttling: {
       rttMs: 0, // 网络延迟，单位 ms
       throughputKbps: 10 * 1024,
       cpuSlowdownMultiplier: 1,
       requestLatencyMs: 0, // 0 means unset
       downloadThroughputKbps: 0,
       uploadThroughputKbps: 0,
     },
     screenEmulation: {
       mobile: false,
       width: 1440,
       height: 960,
       deviceScaleFactor: 1,
       disabled: false,
     },
     skipAudits: ['uses-http2'], // 跳过的检查
     emulatedUserAgent: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4695.0 Safari/537.36 
 Chrome-Lighthouse',
   },
 }
```
settings 属于 Lighthouse 的运行时配置，主要是用来模拟网络和设备的信息，以及使用到哪些审查器。如果检测的页面有 web 端和 h5 端之分，也是在 settings 进行配置。  

## Puppeteer
需要登录后才能访问的页面涉及到登录、点击等操作，我们需要借助 Puppeteer 来模拟点击。基础用法：




原文:  
[搭建自动化 Web 页面性能检测系统 —— 实现篇](https://mp.weixin.qq.com/s/sfbQUFzhJzYWGdfLh0yIzA)