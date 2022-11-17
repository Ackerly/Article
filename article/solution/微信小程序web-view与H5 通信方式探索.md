# 微信小程序web-view与H5 通信方式探索
**需求**  
微信小程序 H5 混合开发就是 在一个小程序中，采用部分小程序原生页面，部分通过Webview内嵌 H5 页面¹，二者配合实现完整业务逻辑的方案。

## 为什么需要混合开发
微信小程序 H5 混合开发就是 在一个小程序中，采用部分小程序原生页面，部分通过Webview内嵌 H5 页面¹，二者配合实现完整业务逻辑的方案。 

## 为什么需要混合开发
- 原生无法满足（例如某团队维护SDK 只提供了WEB端jsSDK，且不维护小程序SDK）
- H5可以同时适用多端（适用范围更广）
- H5可以弥补小程序部分欠缺
- 微信生态有部分限制（包大小，设计规范等）

## 小程序WebView基本用法
- 定义：微信小程序组件 Web-view 定义：承载网页的容器

用法  
``` 
<web-view class="web-holder" src="{{url}}" bindload="bindload" binderror="binderror" bindmessage="bindGetMsg"></web-view>
```
web-view网页中可使用JSSDK 1.3.2提供的接口返回小程序页面。支持的接口有：  

| 接口名                         | 说明                                             | 最低版本  |
|-----------------------------|------------------------------------------------|-------|
| wx.miniProgram.navigateTo   | 参数与小程序接口一致                                     | 1.6.4 |
| wx.miniProgram.navigateBack | 参数与小程序接口一致                                     | 1.6.4 |
| wx.miniProgram.switchTab    | 参数与小程序接口一致                                     | 1.6.5 |
| wx.miniProgram.reLaunch     | 参数与小程序接口一致                                     | 1.6.5 |
| wx.miniProgram.redirectTo   | 参数与小程序接口一致                                     | 1.6.5 |
| wx.miniProgram.postMessage  | 向小程序发送消息，会在特定时机（小程序后退、组件销毁、分享）触发组件的 message 事件 | 1.7.1 |
| wx.miniProgram.getEnv       | 获取当前环境                                         | 1.7.1 |

- 解决方式-小程序后台配置合法域名
- 由于处于开发阶段，本地H5项目 ip 为127.0.0.1，需要配置一下，临时打开

## Bug & Tip
1. tip：网页内 iframe 的域名也需要配置到域名白名单
2. tip：开发者工具上，可以在 web-view 组件上通过右键 - 调试，打开 web-view 组件的调试
3. tip：每个页面只能有一个 web-view，web-view 会自动铺满整个页面，并覆盖其他组件
4. tip：web-view 网页与小程序之间不支持除 JSSDK 提供的接口之外的通信
5. tip：在 iOS 中，若存在 JSSDK 接口调用无响应的情况，可在 web-view 的 src 后面加个#wechat_redirect解决
6. tip：避免在链接中带有中文字符，在 iOS 中会有打开白屏的问题，建议加一下 encodeURIComponent

## 小程序与H5 通信方式
### 小程序->H5  通过 URL 拼接参数  
``` 
http://127.0.0.1:8080/test?key=123
```
### H5->小程序  
wx.miniProgram.postMessage api  
**实现方式：**  
- 引入js SDK
    ```
     <script type="text/javascript" src="https://res.wx.qq.com/open/js/jweixin-1.6.0.js"></script>
    ```
- vue 项目需要安装依赖
    ```
     npm install weixin-webview-jssdk
    ```
- 小程序绑定方法
    ```
     <web-view  bindmessage="bindGetMsg"></web-view>

    bindGetMsg:function(res){
        console.log('从h5页面获取到的信息----->',res)
    }
    ```
- h5端 调用wx.miniProgram.postMessage
    ```
      import wx from "weixin-webview-jssdk";
      wx.miniProgram.postMessage({ data: { foo: {} } });
    ```

优点: 接入成本低
缺点: 向小程序发送消息，会在特定时机（小程序后退、组件销毁、分享）触发组件的 message 事件，只能这些特定时机，基本宣布postMessage没用！因为这些时机很苛刻，不符合我们要求。反人类设计  

**方式二：url 携带信息navigateTo、reLaunch、redirectTo**  
实现方式：  
```
wx.miniProgram.navigateTo({
        url: '../h5/loading-page',
      })

wx.miniProgram.navigateTo({
        url: '../h5/loading-page?type=aaa',
      })
```
缺点：url 数据量有限，且需要打开界面  

**方式三：内存共享**  
无法实现，原因 wx.setStorage 与localStorage 隔离  
```
localStorage.setItem('h5key','value')
wx.setStorageSync('wx-key', 'value')
```

## 长连-Websocket
- Websocket 简介：WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议
- 建立在 TCP 协议之上，服务器端的实现比较容易
- 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器
- 数据格式比较轻量，性能开销小，通信高效
- 可以发送文本，也可以发送二进制数据
- 没有同源限制，客户端可以与任意服务器通信
- 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL

优点：可以实现实时通信  
缺点：成本高，服务器压力大等；放弃此方式。  

**总结**  
- 微信并不鼓励在小程序中大范围嵌入 H5，为了避免开发者把小程序变成“浏览器”，微信对小程序与内嵌 H5 的通讯做了诸多限制
- 尽量使用单一方式实现，比如纯小程序原生，将h5功能移至小程序原生
- 原生页面与 H5 之间通过 URL 进行通信
- 不要尝试越过wx 限制
- 不得不用混合开发时，尽量做好优化，引入骨架屏等优化方式提高用户体验感
- 以上三种方式均未很好实现web-view 与H5双向通信

## 优化-骨架屏
骨架屏是页面的一个空白版本，通常会在页面完全渲染之前，通过一些灰色的区块大致勾勒出轮廓，待数据加载完成后，再替换成真实的内容。骨架屏在内容还没有出现之前的页面骨架填充，以免留白。  

### 小程序骨架屏引入方式
- 小程序骨架屏引入方式

使用方法：  
1. 生成骨架屏页面index.skeleton.wxml
    ```
    <template name="skeleton">
    <view class="sk-container">
        <view class="container">
        <view class="userinfo">
            <view class="userinfo-avatar">
            <open-data type="userAvatarUrl"></open-data>
            </view>
            <open-data type="userNickName"></open-data>
        </view>
        <view class="usermotto">
            <text class="user-motto sk-transparent sk-text-14-2857-765 sk-text">Hello World</text>
        </view>
        </view>
    </view>
    </template>
    ```
2. 生成骨架屏样式index.skeleton.wxss
    ```
    .sk-transparent {
    color: transparent !important;
    }
    .sk-text-14-2857-765 {
    background-image: linear-gradient(transparent 14.2857%, #EEEEEE 0%, #EEEEEE 85.7143%, transparent 0%) !important;
    background-size: 100% 44.8000rpx;
    position: relative !important;
    }
    .sk-text {
    background-origin: content-box !important;
    background-clip: content-box !important;
    background-color: transparent !important;
    color: transparent !important;
    background-repeat: repeat-y !important;
    }
    .sk-container {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background-color: transparent;
    }
    ```
3. 在 /pages/index/index.wxml 引入模板
   ```
   <import src="index.skeleton.wxml"/>
    <template is="skeleton" wx:if="{{loading}}" />
   ```
4. 在 /pages/index/index.wxss 中引入样式
   ```
   @import "./index.skeleton.wxss";
   ```

## H5骨架屏引入方式
Page Skeleton是一款 webpack 插件，该插件的目的是根据你项目中不同的路由页面生成相应的骨架屏页面，并将骨架屏页面通过 webpack 打包到对应的静态路由页面中。  
- 支持多种加载动画
- 针对移动端 web 页面
- 支持多路由
- 可定制化，可以通过配置项对骨架块形状颜色进行配置，同时也可以在预览页面直接修改骨架页面源码
- 几乎可以零配置使用

原文:  
[微信小程序web-view与H5 通信方式探索](https://mp.weixin.qq.com/s/qYrDTuEag_AKp44B43lqAw)
