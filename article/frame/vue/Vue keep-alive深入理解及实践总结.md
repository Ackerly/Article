# Vue keep-alive深入理解及实践总结  
## 什么是keep-alive
keepalive 是 Vue 内置的一个组件，可以使被包含的组件保留状态，或避免重新渲染 。也就是所谓的组件缓存。  
<keep-alive>是Vue的内置组件，能在组件切换过程中将状态保留在内存中，防止重复渲染DOM。  
> <keep-alive> 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 <transition> 相似，<keep-alive> 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。  
prop:  
- include: 字符串或正则表达式。只有匹配的组件会被缓存。
- exclude: 字符串或正则表达式。任何匹配的组件都不会被缓存。

## keep-alive的声明周期执行
- 页面第一次进入，钩子的触发顺序  
created-> mounted-> activated，
退出时触发 deactivated 当再次进入（前进或者后退）时，只触发 activated
- 事件挂载的方法等，只执行一次的放在 mounted 中；组件每次进去执行的方法放在 activated 中

## 基本用法
``` 
<!--被keepalive包含的组件会被缓存-->
<keep-alive>
    <component><component />
</keep-alive>
```
被keepalive包含的组件不会被再次初始化，也就意味着不会重走生命周期函数。但是有时候是希望我们缓存的组件可以能够再次进行渲染，这时 Vue 为我们解决了这个问题 被包含在 keep-alive 中创建的组件，会多出两个生命周期的钩子: activated 与 deactivated：
- activated 当 keepalive 包含的组件再次渲染的时候触发
- deactivated 当 keepalive 包含的组件销毁的时候触发

keepalive是一个抽象的组件，缓存的组件不会被 mounted,为此提供activated和deactivated钩子函数
## 参数理解
keepalive 可以接收3个属性做为参数进行匹配对应的组件进行缓存:
- include 包含的组件(可以为字符串，数组，以及正则表达式,只有匹配的组件会被缓存)
- exclude 排除的组件(以为字符串，数组，以及正则表达式,任何匹配的组件都不会被缓存)
- max 缓存组件的最大值(类型为字符或者数字,可以控制缓存组件的个数)

注：当使用正则表达式或者数组时，一定要使用 v-bind
``` 
<!-- 将（只）缓存组件name为a或者b的组件, 结合动态组件使用 -->
<keep-alive include="a,b">
  <component></component>
</keep-alive>

<!-- 组件name为c的组件不缓存(可以保留它的状态或避免重新渲染) -->
<keep-alive exclude="c"> 
  <component></component>
</keep-alive>

<!-- 使用正则表达式，需使用v-bind -->
<keep-alive :include="/a|b/">
  <component :is="view"></component>
</keep-alive>

<!-- 动态判断 -->
<keep-alive :include="includedComponents">
  <router-view></router-view>
</keep-alive>

<!-- 如果同时使用include,exclude,那么exclude优先于include， 下面的例子只缓存a组件 -->
<keep-alive include="a,b" exclude="b"> 
  <component></component>
</keep-alive>

<!-- 如果缓存的组件超过了max设定的值5，那么将删除第一个缓存的组件 -->
<keep-alive exclude="c" max="5"> 
  <component></component>
</keep-alive>
```

## 遇见vue-router结合router使用，缓存部分页面
所有路径下的视图组件都会被缓存
``` 
<keep-alive>
    <router-view>
        <!-- 所有路径匹配到的视图组件都会被缓存！ -->
    </router-view>
</keep-alive>
```
如果只想要router-view里面的某个组件被缓存
- 使用 include/exclude
- 使用 meta 属性

1. 用 include (exclude例子类似)
> 缺点：需要知道组件的 name，项目复杂的时候不是很好的选择

``` 
<keep-alive include="a">
    <router-view>
        <!-- 只有路径匹配到的 include 为 a 组件会被缓存 -->
    </router-view>
</keep-alive>
```
2. 使用 meta 属性
> 优点：不需要例举出需要被缓存组件名称

使用$route.meta的keepAlive属性：
``` 
<keep-alive>
    <router-view v-if="$route.meta.keepAlive"></router-view>
</keep-alive>
<router-view v-if="!$route.meta.keepAlive"></router-view>
```
需要在router中设置router的元信息meta：
``` 
//...router.js
export default new Router({
  routes: [
    {
      path: '/',
      name: 'Hello',
      component: Hello,
      meta: {
        keepAlive: false // 不需要缓存
      }
    },
    {
      path: '/page1',
      name: 'Page1',
      component: Page1,
      meta: {
        keepAlive: true // 需要被缓存
      }
    }
  ]
})
```
**使用router。meta拓展**
假设这里有 3 个路由： A、B、C。
- 需求：
    - 默认显示 A
    - B 跳到 A，A 不刷新
    - C 跳到 A，A 刷新
 - 实现方式
    - 在 A 路由里面设置 meta 属性
 ```
{
        path: '/',
        name: 'A',
        component: A,
        meta: {
            keepAlive: true // 需要被缓存
        }
}
```
- 在 B 组件里面设置 beforeRouteLeave：
``` 
export default {
        data() {
            return {};
        },
        methods: {},
        beforeRouteLeave(to, from, next) {
             // 设置下一个路由的 meta
            to.meta.keepAlive = true;  // 让 A 缓存，即不刷新
            next();
        }
};
```
- 在 C 组件里面设置 beforeRouteLeave：
``` 
export default {
        data() {
            return {};
        },
        methods: {},
        beforeRouteLeave(to, from, next) {
            // 设置下一个路由的 meta
            to.meta.keepAlive = false; // 让 A 不缓存，即刷新
            next();
        }
};

```
## 注意
1. keep-alive 先匹配被包含组件的 name 字段，如果 name 不可用，则匹配当前组件 components 配置中的注册名称。
2. keep-alive 不会在函数式组件中正常工作，因为它们没有缓存实例。
3. 当匹配条件同时在 include 与 exclude 存在时，以 exclude 优先级最高(当前vue 2.4.2 version)。比如：包含于排除同时匹配到了组件A，那组件A不会被缓存。
4. 包含在 keep-alive 中，但符合 exclude ，不会调用 activated 和 deactivated。

原文: 
[Vue keep-alive深入理解及实践总结](https://juejin.cn/post/6844903919273918477)
