# Vue3组件之封装一个更好用的url组件
## 定义共用函数（改进版）
接收父组件的v-model的属性值，并且提交数据的代码抽象出来放在独立的js文件里面，这样各种组件就都可以拿来用了。
``` 
// controlManage.js
import { ref, watch } from 'vue'

/**
* 控件的赋值、提交的统一管理函数
** 属性：
* ** props： 组件的属性，获取modelValue，和meta
* ** context： 上下文获取emit，提交数据
** 返回：
* ** value：绑定到组件的值
* ** mySubmit：向父组件提交的事件
*/
const controlManage = (props, context) => {
  // 用于绑定控件的值。
  const value = ref(props.meta.defaultValue)

  // 获取父组件设置的属性
  const _value = props.modelValue

  // 设置控件值。如果有属性值（修改状态）则把属性值设置给控件值。
  if (!(_value === '' || _value === 0 || _value === null)) {
    value.value = _value
  }

  // 监听 modelValue 属性，给 value 赋值
  watch(() => props.modelValue, (v1, v2) => {
    // console.log('controlManage监听属性变化', v1)
    value.value = v1
  })

  // 向父组件提交事件
  const mySubmit = (val) => {
    context.emit('update:modelValue', val)
    context.emit('input', val)
  }

  return {
    /**
    * 用于绑定控件的值。
    ** 添加状态可以获取默认值。
    ** 修改状态可以设置 modelValue 值
    ** 监听 modelValue 属性，给value 赋值
    */
    value,
    /**
    * 向父组件提交事件
    ** 可以直接绑定到组件的事件，
    ** 也可以套个娃。
    */
    mySubmit
  }
}

export default controlManage
```
### 流程
- 定义value用于绑定控件值
- 添加状态：把默认值设置给value
- 修改状态：把v-model设置给value
- 监听父组件的v-model，有变化就设置给value
- input事件或者其他事件的时候，提交给父组件

## 定义url管理类
``` 
/**
 * 处理url的管理类
 * * 功能：
 * ** 提交拼接后的完整的url
 * ** 提供绑定控件值、事件
 * ** 修改时自动拆分属性
 * * 参数：
 * ** value： control类的value
 * ** mySubmit： control类的mySubmit，直接就提交了
 */
const urlManage = (value, mySubmit) => {
  // 把url分成三份处理
  const url = reactive({
    http: 'Https://',
    com: '.com',
    value: ''
  })

  // 域名后缀
  const comList = [
    { value: '.com' },
    { value: '.cn' },
    { value: '.net' },
    { value: '.com.cn' },
    { value: '.net.cn' },
    { value: '.org.cn' },
    { value: '.org' },
    { value: '.top' },
    { value: '.vip' },
    { value: '.中国' },
    { value: '.企业' },
    { value: '.公司' },
    { value: '.网络' }
  ]

  // 拆分属性，给url赋值
  watch(() => value.value, (v1, v2) => {
    const arrUrlAll = v1.toLowerCase().split('://')
    console.log('===============================================')
    console.log('父组件:', v1)
    // 判断 ://
    if (arrUrlAll.length === 1) { // 没有http://，直接算作url.value
      url.value = arrUrlAll[0]
    } else if (arrUrlAll.length === 2) {
      url.http = arrUrlAll[0] + '://'
      // 有http://，用 . 拆分后面的
      const arrUrl = arrUrlAll[1].split('.')
      const len = arrUrl.length
      let endPosition = 0
      switch (len) {
        case 1: // 只有一个，直接算作url.value
          url.value = arrUrl[0]
          break
        case 2: // 有两个，一个是url.value，一个是com
          url.value = arrUrl[0]
          url.com = '.' + arrUrl[1]
          break
        default: // 有两个以上，判断两个后缀的情况
          if (arrUrl[len - 1] === 'cn' && (arrUrl[len - 2] === 'com' || arrUrl[len - 2] === 'net' || arrUrl[len - 2] === 'org')) {
            endPosition = len - 2
            url.com = '.' + arrUrl[endPosition] + '.cn'
          } else {
            endPosition = len - 1
            url.com = '.' + arrUrl[endPosition]
          }
          url.value = arrUrl[0]
          for (let i = 1; i < endPosition; i++) {
            url.value += '.' + arrUrl[i]
          }
      }
    }
  })

  // com的查询事件
  const querySearch = (str, cb) => {
    const results = str
      ? comList.filter((item) =>
        item.value.indexOf(str.toLowerCase()) === 0)
      : comList

    // 调用 callback 返回建议列表的数据
    cb(results)
  }

  // url的三个change事件
  const urlSubmit = () => {
    mySubmit(url.http + url.value + url.com)
  }

  return {
    url,
    querySearch,
    urlSubmit
  }
}
```
因为要把url拆分成三份，于是就定义了一个对象。
- 域名的后缀  
- 添加状态
- 修改状态  
需要把完整的url拆分开分别赋值。 
## 定义url组件
引用两个js文件，进行拼接
``` 
<template>
  <el-input
    :placeholder="meta.placeholder"
    :maxlength="meta.maxlength"
    v-model="url.value"
    @input="urlSubmit"
  >
    <template #prepend><!--前面的选项-->
      <el-select style="width: 90px;"
        v-model="url.http"
        @change="urlSubmit"
        placeholder="请选择">
          <el-option label="Http://" value="Http://"></el-option>
          <el-option label="Https://" value="Https://"></el-option>
      </el-select>
    </template>
    <template #append><!--后面的域名后缀-->
      <el-autocomplete style="width: 100px;"
        class="inline-input"
        v-model="url.com"
        :fetch-suggestions="querySearch"
        placeholder="请输入内容"
        @select="urlSubmit"
        @change="urlSubmit"
      ></el-autocomplete>
    </template>
  </el-input>
</template>
```
``` 
export default {
  name: 'nf-el-from-url',
  props: {
    modelValue: String,
    meta: metaInput
  },
  emits: ['input', 'change', 'blur', 'focus', 'clear'],
  setup (props, context) {
    const { value, mySubmit } = controlManage(props, context)

    // const { url, querySearch, urlSubmit } = urlManage(value, mySubmit)

    return {
      ...urlManage(value, mySubmit)
      // querySearch, // com的筛选的函数
      // url, // url 相关的值
      // urlSubmit // 触发事件
    }
  }
}
```
- 模板  
基于element-plus提供的组件修改。
- return  
在setup里面先把函数引用进来，获取内部对象，然后在返回。 那么如果没有其他操作的话，可以直接在return里面使用解构的方式直接返给模板。

参考:
[Vue3组件（五）封装一个更好用的url组件](https://juejin.cn/post/6930378495587516423?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
