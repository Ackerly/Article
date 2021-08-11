# vue3配置jest环境踩坑
## jest只能用26+版本，不能用27+版本
npm install jest --save-dev安装后进行配置，出现Cannot destructure property 'config' of 'undefined'，
解决方法使用26版本
``` 
package.json的jest相关包配置
 "@types/jest": "^26.0.23",
 "babel-jest": "^26.3.0",
 "jest": "^26.6.3",
 "ts-jest": "^26.0.0",
```
## Cannot call text on an empty DOMWrapper错误
编写dialog组件测试用例执行失败
``` 
const TESTSTR = 'risk everywhere risk everywhere risk everywhere';
describe('Dialog vue',() => {
  test('dialog should have content when content has been given', async () => {
 const wrapper = mount(Dialog,{
   props:{
     content: TESTSTR,
     modelValue: true
   }
 });
 await nextTick();
 expect(wrapper.find('.modal-body').text()).toEqual(TESTSTR);
  });
});
```
原因：dialog组件报错,teleport的问题，元素都搬走了，DOMWrapper变成empty了，应该要加一个配置属性能让teleport失效
``` 
// 错误代码
<template>
 <teleport to="body">
 </teleport>
</template>
```
``` 
// 修正代码
<template>
 <teleport to="body" :disabled="!appendToBody">
 </teleport>
</template>
```
