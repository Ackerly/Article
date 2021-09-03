# CSS禁止元素拖拽
## 用户行为三个用户属性
- user-select属性可以设置是否允许用户选择页面中的图文内容
- user-modify属性可以设置是否允许输入框输入内容，以及是否只允许输入纯文本
- user-drag属性可以设置是否允许页面元素拖拽。

## user-drag禁止拖拽
页面中的图文元素设置-webkit-user-drag:none，则该元素就无法拖拽了。
``` 
<img src="by zhangxinxu.jpg">
<img src="by zhangxinxu.jpg" class="user-drag">
.user-drag {
    -webkit-user-drag: none;
}
```
### 兼容性
Firefox浏览器尚未支持，Chrome浏览器不支持-webkit-user-drag: element声明，移动端不能使用

## HTML draggable属性
使用CSS设置元素能够拖拽的优点在于控制方便，适合用在只需要兼容Chrome、Safari浏览器的桌面端PC网页产品中。
如果HTML有操作权限，且最终效果需要较好的兼容性，则还是需要使用传统的draggable属性实现，通过设置true还是false设置元素能够拖拽。
``` 
<!-- 元素可以被拖动 -->
<img src="zxx.jpg" draggable="true">

<!-- 元素不可以被拖动 -->
<img src="zxx.jpg" draggable="false">
```
draggable属性常常和原生的drag & drop事件配合使用，可以实现任意元素的拖拽效果。  
HTML draggable属性的兼容性是相当的好，移动端全部支持，以及IE10+浏览器也都支持，基本都可以使用的HTML属性：
参考:  
[如何使用CSS禁止元素拖拽](https://www.zhangxinxu.com/wordpress/2021/05/css-user-drag/)
