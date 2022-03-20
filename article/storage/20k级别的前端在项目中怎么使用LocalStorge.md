# 20k级别的前端在项目中怎么使用LocalStorge.md
localStorage、sessionStorage的用途：  
1. 登录完成后token的存储
2. 用户部分信息的存储，比如昵称、头像、简介
3. 一些项目通用参数的存储，例如某个id、某个参数params
4. 项目状态管理的持久化，例如vuex的持久化、redux的持久化
5. 项目整体的切换状态存储，例如主题颜色、icon风格、语言标识

## 普通使用
1. 基础变量
```
// 当我们存基本变量时
localStorage.setItem('基本变量', '这是一个基本变量')
// 当我们取值时
localStorage.getItem('基本变量')
// 当我们删除时
localStorage.removeItem('基本变量')
```
2. 引用变量
```
// 当我们存引用变量时
localStorage.setItem('引用变量', JSON.stringify(data))
// 当我们取值时
const data = JSON.parse(localStorage.getItem('引用变量'))
// 当我们删除时
localStorage.removeItem('引用变量')
```
3. 清空
```
localStorage.clear()
```

## 暴露的问题
**命名过于简单**  
1. 比如我们存用户信息会使用user作为 key 来存储
2. 存储主题的时候用theme 作为 key 来存储
3. 存储令牌时使用token作为 key 来存储

同源的两个项目，它们的localStorage是互通的，两个项目都需要往localStorage中存储一个 key 为name的值，那么这就会造成两个项目的name互相顶替的现象，也就是互相污染现象  
**时效性**  
localStorage、sessionStorage这两个的生命周期分别是  
- localStorage：除非手动清除，否则一直存在
- sessionStorage：生命结束于当前标签页的关闭或浏览器的关闭

普通的使用时没什么问题的，但是给某些指定缓存加上特定的时效性，是非常重要的！比如某一天：  
后端：”兄弟，你一登录我就把token给你“
前端：”好呀，那你应该会顺便判断token过期没吧？“
后端：”不行哦，放在你前端判断过期呗“
前端：”行吧。。。。。“
这时候，因为需要在前端判断过期，所以咱们就得给token设置一个时效性，或者是1天，或者是7天  

**隐秘性**  
存在localStorage、sessionStorage中，在开发过程中，确实有利于咱们的开发，想看的时候也是一目了然，点击Application就可以看到。  
但是，一旦产品上线了，用户也是可以看到缓存中的东西的，而咱们肯定是会想：有些东西可以让用户看到，但是有些东西我不想让用户看到
在做状态管理持久化时，需要把数据先存在localStorage中，这个时候就很有必要对缓存进行加密了。  
## 解决方案
**命名规范**  
项目名 + 当前环境 + 项目版本 + 缓存key  
**expire定时**  
设置缓存key时，将value包装成一个对象，对象中有相应的时效时段，当下一次想获取缓存值时，判断有无超时，不超时就获取value，超时就删除这个缓存  
**crypto加密**  
使用crypto-js进行对数据的加密，使用这个库里的encrypt、decrypyt进行加密、解密  






参考:
[想知道一个20k级别前端在项目中是怎么使用LocalStorage的吗?](https://juejin.cn/post/7033749571939336228)