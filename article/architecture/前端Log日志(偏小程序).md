# 前端Log日志(小程序)
小程序侧提供了两种小程序 Log 日志接口：
- LogManager ( 普通日志 )
- RealtimeLogManager ( 实时日志 )

两者的区别是:  
- LogManager 可以让用户有种心安的感觉，因为 LogManager 是用户手动反馈的问题
- RealtimeLogManager 则对开发者更友好，可以在用户不知情的情况下收集到问题信息，并在用户无感的情况下对问题进行修复。

## LogManager
小程序提供的 Log 日志接口，通过 wx.getLogManager() 获取实例。  
注意：  
- 最多保存5M的日志内容，超过5M后，旧的日志内容会被删除
- 对于 小程序 ，用户可以通过使用 button 组件的 open-type="feedback" 来上传打印的日志
- 对于 小游戏 ，用户可以通过使用 wx.createFeedbackButton 来创建上传打印的日志的按钮
- 开发者可以通过小程序管理后台左侧菜单 反馈管理 页面查看相关打印日志

**创建LogManager实例**  
wx.getLogManager() 获取日志实例，括号中可以传参 { level: 0 | 1 } 来决定是否写入 Page 的生命周期函数， wx 命名空间下的函数日志。  
**用户反馈上传LogManager记录的日志**  
当日志记录后， 用户可以在小程序的 profile 页面，单击 反馈与投诉 ，在点击 功能异常 进行日志上传。  
**开发者处理用户反馈和用户沟通**
开发者可以在小程序后台 管理 -> 用户反馈 -> 功能异常 查看用户反馈的信息。  
开发者可以在 功能 -> 客服 下绑定客服微信，绑定后可以在 48小时 内通过微信和反馈用户沟通。 
>沟通需要用户反馈时勾选：允许开发者在 48 小时内通过客服消息联系我。

## RealtimeLogManager
小程序提供的 实时Log 日志接口，通过 wx.getRealtimeLogManager() 获取实例。  
- wx.getRealtimeLogManager() 基础库 2.7.1 开始支持
- 官方给出实时日志每条的容量上限是 5kb
- 官方对每条日志的定义：在一个页面 onShow -> onHide 之间，会聚合成一条日志上报
- 开发者可从小程序管理后台： 开发 -> 运维中心 -> 实时日志 进入小程序端日志查询页面

> 为了定位问题方便，日志是按页面划分的，某一个页面，在onShow到onHide（切换到其它页面、右上角圆点退到后台）之间打的日志，会聚合成一条日志上报，并且在小程序管理后台上可以根据页面路径搜索出该条日志

**创建RealtimeLogManager实例**  
可以通过 wx.getRealtimeLogManager() 获取实时日志实例  
**使用RealtimeLogManager实例**  
``` 
const logger = wx.getRealtimeLogManager()
logger.debug({str: 'hello world'}, 'debug log', 100, [1, 2, 3])
logger.info({str: 'hello world'}, 'info log', 100, [1, 2, 3])
logger.error({str: 'hello world'}, 'error log', 100, [1, 2, 3])
logger.warn({str: 'hello world'}, 'warn log', 100, [1, 2, 3])
```
**查看实时日志**  
与普通日志不同的是，实时日志不再需要用户反馈，可以直接通过以下方式查看实例。  
1. 登录小程序后台
2. 通过路径 开发 -> 开发管理 -> 运维中心 -> 实时日志 查看实时日志

## 如何搭建小程序Log日志系统
### 封装小程序 Log 方法
**版本问题**  
需要一个常量用以定义版本号，以便于我们定位出问题的代码版本  
``` 
const VERSION = "1.0.0"
const logger = wx.getLogManager({ level: 0 })

logger.log(VERSION, info)
```
**打印格式**  
通过 [version] file | content 的统一格式来更快的定位内容
``` 
const VERSION = "1.0.0"
const logger = wx.getLogManager({ level: 0 })

logger.log(`[${VERSION}] ${file} | `, ...args)
```
**兼容性**  
考虑用户小程序版本不足以支持 getLogManager 、 getRealtimeLogManager 的情况
``` 
const VERSION = "0.0.18";

const canIUseLogManage = wx.canIUse("getLogManager");
const logger = canIUseLogManage ? wx.getLogManager({ level: 0 }) : null;
const realtimeLogger = wx.getRealtimeLogManager ? wx.getRealtimeLogManager() : null;

export function RUN(file, ...args) {
  console.log(`[${VERSION}]`, file, " | ", ...args);
  if (canIUseLogManage) {
    logger.log(`[${VERSION}]`, file, " | ", ...args);
  }

  realtimeLogger && realtimeLogger.info(`[${VERSION}]`, file, " | ", ...args);
}
```
**类型**  
需要兼容 debug 、 log 、 error 类型的 log日志   
``` 
export function RUN(file, ...args) {
    ...
}

export function DEBUG(file, ...args) {
    ...
}

export function ERROR(file, ...args) {
    ...
}

export function getLogger(fileName) {
  return {
    DEBUG: function (...args) {
      DEBUG(fileName, ...args)
    },
    RUN: function (...args) {
      RUN(fileName, ...args)
    },
    ERROR: function (...args) {
      ERROR(fileName, ...args)
    }
  }
}
```
### 全局注入Log
小程序提供了 behaviors 参数，用以让多个页面拥有相同的数据字段和方法。
> 需要注意的是， page 级别的 behaviors 在 2.9.2 之后开始支持

通过封装一个通用的 behaviors ，然后在需要 log 的页面进行引入即可
``` 
import * as Log from "./log-test";

export default Behavior({
  definitionFilter(defFields) {
    console.log(defFields);
    Object.keys(defFields.methods || {}).forEach(methodName => {
      const originMethod = defFields.methods[methodName];
      defFields.methods[methodName] = function (ev, ...args) {
        if (ev && ev.target && ev.currentTarget && ev.currentTarget.dataset) {
          Log.RUN(defFields.data.PAGE_NAME, `${methodName} invoke, event dataset = `, ev.currentTarget.dataset, "params = ", ...args);
        } else {
          Log.RUN(defFields.data.PAGE_NAME, `${methodName} invoke, params = `, ev, ...args);
        }
        originMethod.call(this, ev, ...args)
      }
    })
  }
})
```

原文: 
[前端架构之路：前端 Log 日志（偏小程序）](https://juejin.cn/post/7053835525785911333)
