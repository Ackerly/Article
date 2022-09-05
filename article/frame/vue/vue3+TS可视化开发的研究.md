# vue3+TS可视化开发的研究  
## 可视化开发需要解决什么问题  
1. 组件可拖拽，不论是html5的拖拽还是自定义的位置拖拽都是核心
2. 在如今的前端框架下，该如何解决组件动态渲染的问题
3. 上线之后的产品，不可能弄一个基础库上去吧。该如何解决页面加载困难的问题
## 关于拖拽
**两种实现拖拽的方案**  
1. html5拖拽draggable
2. 自定义元素的style的top，left等值

**选择html拖拽的原因**  
1. 拖拽更好看
2. 能检测拖拽到哪个元素下面

**核心实现**  
- draggable="true"，定位拖拽元素
- ondrop拖拽到哪里
- ondragstart开始拖拽

``` 
// HTML部分  
<template v-for="(item, index) in list">
  <div
    :key="index"
    @dragover.prevent
    @drop.prevent="drop($event, index)"
    draggable="true"
    @dragstart="dragstart($event, index)"
    class="ceshi"
  >
    {{ item }}
  </div>
</template>
```
``` 
setup(props, ctx) {
    const list = ref([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]).value
    const setArr = (): void => {
      list[0] = 1111
    }
    // 开始移动
    const dragstart = (ev: Event, index: number): void => {
      ev.dataTransfer.setData('text', index)
    }
    // 放到何处
    const drop = (ev: Event, index: number): void => {
      const startIndex: number = Number(ev.dataTransfer.getData('text'))
      const startLi: number = list[startIndex]
      const endLi: number = list[index]
      list.splice(index, 1, startLi) // 放过去
      list.splice(startIndex, 1, endLi) // 反过来
    }

    return {
      drop,
      dragstart,
      list,
      setArr,
    }
}
```
## 拖拽过程中的坑
js的原生事件怎么声明类型，声明了object之后是无法定义下一步操作的，其实ts有一个内置类型Event  
但是event还不够，我在使用的时候，如下面这段代码会报错
``` 
ev.dataTransfer.setData('text', index)
```
需要使用到在src目录下定义gobal.d.ts进行重新定义
``` 
declare interface Event {
  dataTransfer: {
    setData(...args): void
    getData(...args)
  }
}
```
## vue3框架ts的坑
使用composition-api你没有办法使用this。可以使用命周期写法，但是vue3的this是vue单独实例化出来放在全局的。为了兼容老代码，我们要使用
``` 
app.config.globalProperties
```
vue3在ts模式下有点问题，上面代码进行定义的时候，定义没有问题。但是最终在使用的时候，this.变量是ts报error了。解决方式是
``` 
// 解决ts下全局变量定义问题
declare module '@vue/runtime-core' {
  interface ComponentCustomProperties {
    http: number
  }
}
```
## 关于动态组件
解决方式就是
``` 
<component :is="DynamicComponents"></component>
```
is传入的格式需要是你导入的组件的变量名称  
``` 
setup(props, ctx) {
  console.log({ ...ctx })
  // 动态组件测试
  const dom = ref(-1).value
  console.log(dom)

  const DynamicComponents = ref('DynamicComponents')
  const setDynamicComponents = (): void => {
    DynamicComponents.value = 'DhtTest'
    console.log(DynamicComponents.value)
  }

  return {
    setDynamicComponents,
    DynamicComponents,
  }
}
```
## 远程加载组件
vue3的按需加载是利用了webpack的code-spliting，然后其实每次我们跳转一个新页面的时候是执行了一段本身script就是远程从服务器拿到数据然后加载的。  


原文:  
[vue3+Ts可视化开发的研究，实战拖拽基础，组件动态生成，远程加载组件](https://juejin.cn/post/6860290630435012621?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
