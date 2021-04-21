# Vue-Router实现原理

## 后端路由
基本流程
1. 浏览器发出请求
2. 服务器监听到80端口有请求过来，并解析url路劲
3. 根据路由配置，返回相应信息
4. 浏览器根据数据包的Content-Type来决定如何解析数据

**后端路由优点是有利于SEO且安全性较高，缺点是代码耦合度高，加大服务器压力，且http请求受限于网络环境，影响用户体验**
  
## 前端路由
前端单页应用（SPA）的兴起，前端页面完全变成组件化，不同的页面就是不同的组件，页面的切换就是组件的切换，页面切换的时候不需要再通过http请求，直接通过JS解析url地址，找到对应的组件进行渲染

**前端的优点就是组件切换不需要发送http请求，切换跳转快，用户体验好，缺点就是没有合理的利用缓存且不利于SEO**

### hash模式
通过window.location.hash获取当前url的hash，hash模式通过hashchange方法可以监听url中hash的变化  
**hash模式的特点是兼容性更好，并且hash变化会在浏览器的history增加一条记录，可以实现浏览器的前进和后退**  
**缺点是多了#**

### history模式
history模式是另一种路由模式，基于HTML5的history对象，通过location.pathname获取url路由地址，通过pushState和replaceState方法修改url地址，结合popState监听url中的路由变化

**history模式特点实现更方便，可读性强**  
**缺点是用户属性或直接输入地址会向服务器发送一个请求，所以history模式需要服务端进行支持，将路由重定向到根路由**

### vue-router工作流程
1. url改变
2. 触发事件监听
3. 改变vue-router中的current变量
4. 监视current变量的监视者
5. 获取新的组件
6. render

### Vue router实现
vue-router通过Vue.use方法注入进Vue实例，使用的时候需要全局用到vue-router的router-view和router-link组件，以及this.$router/$route这样的实例对象

参考文章：  
[vue-router原理及其核心功能实现](https://juejin.cn/post/6844904169954869262)  
[前端路由简介以及vue-router实现原理](https://zhuanlan.zhihu.com/p/37730038)
