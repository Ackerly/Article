# vue2和vue3的slot
## 2.x语法
### vm.$slots模板用法
``` 
// fahter
<template>
    <Book>
      <span>默认值</span>
      <span slot="header">header</span>
      <span slot="content">content1</span>
      <span slot="content">content2</span>
    </Book>
</template>

// child
<template>
  <div>
    <slot>default 1</slot>
    <slot name="header">default 2</slot>
    <slot name="content">default 3</slot>
  </div>
</template>
```
- slot默认
- slot name具名slot，这样可以指定多个slot，并任意排列位置，实现布局需求
### vm.$slots渲染函数中用法
``` 
render(h) {
    return h(Book,{},[
      h('span',{},'默认值'),
      h('span',{slot:'header'},'header'),
      h('span',{slot:'content'},'content1'),
      h('span',{slot:'content'},'content2')
    ]) 
  }
```
- 参数1：Book组件
- 参数2：对第一个参数 {html tag | component} 进行参数设置
- 参数3：children VNodes {String | Array}可以字符串，有多个使用数组的形式

实现自定义默认值
``` 
export default {
  render(h) {
    return h('div',{},[
      this.$slots.default || 'default1',
      this.$slots.header  || 'default2',
      this.$slots.content  || 'default3'
    ])
  }
}
```
### vm.$scopedSlots模板用法
与 $slots 的区别是，$scopedSlots 可以从组件内部向父级作用域传递数据。
``` 
<template>
  <div>
    <Child>
      <template v-slot:default="slotProps">
        {{ slotProps.user.name }}
      </template>
    </Child>
  </div>
</template>

<template>
  <div>
    <slot :user="user"></slot>
  </div>
</template>
```
### vm.$scopedSlots 渲染函数中用法
- $scopedSlots.default 是一个函数可以向父组件传递数据
- 父组件中通过 scopedSlots 来接收子组件传递的数据
``` 
  // fahter
  render(h) {
    return h(Book,{
            scopedSlots: {
              default:function(props){
                return [ 
                        props.user.name,
                        <div>text</div>,
                        'sdfsdf'
                      ]
              }
            } 
          })
  }

   // Book.vue
    render(h) {
      return h('div',{},[
        this.$scopedSlots.default({
          user:this.user 
        })
      ])
    }
```
## 3.x语法
``` 
<script lang='ts'>
  import { h } from 'vue';
  import Child from './Child.vue'
  export default {
    components:{
      Child
    },
    render() {
      return h(Child,{},
        { 
          header:(props:{year:number})=>{
            return h('div',`this year is ${props.year}`)
          },
          title:()=> h('h1', {
            innerHTML:'标题',
            onClick(){
              console.log('点击了标题')
            },
            style:{
              color:'#f66'
            }
          })
        }
      )
    }
  }
</script>

// child
<script lang='ts'>
  import { h } from 'vue';
  export default {
    render() {
      return h('div',[this.$slots.title(),this.$slots.header({year:2021})])
    }
  }
</script>
```

## h渲染函数
**2.x语法**  
``` 
{
  class: ['button', 'is-outlined'],
  style: { color: '#34495E' },
  attrs: { id: 'submit' },
  domProps: { innerHTML: '' },
  on: { click: submitForm },
  key: 'submit-button'
}
```
**3.x语法**  
```
{
  class: ['button', 'is-outlined'],
  style: { color: '#34495E' },
  id: 'submit',
  innerHTML: '',
  onClick: submitForm,
  key: 'submit-button'
}
```

参考:
[[vue3 vs vue2] slot 用法详解](https://juejin.cn/post/6931286420040450061?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
