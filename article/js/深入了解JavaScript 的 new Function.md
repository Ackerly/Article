# 深入了解JavaScript的 new Function
**语法**  
```
let func = new Function ([arg1, arg2, …argN], functionBody);
```
最后一个参数必须是函数体，其余参数作为传递给函数体的参数。  
例如：  
```
let sum = new Function('a', 'b', 'return a + b');
console.log(sum(1, 2));
```
平时开发 JavaScript 或者 Node.js 的时候，没有理由使用 new Function 构造函数，因为不需要直接使用函数或者 () => {} 箭头函数。  

## 不可替代的角色  
**无效的 JSON 对象字符串合法化**  
有以下字符串：  
```
let str = `{ "id": 103, name: 'yh', 'date': '2022–07–06' }`;
```
其中的字符串不符合JSON格式（键值需要双引号），使用JSON.parse()解析会报错  
有没有什么办法可以把这个字符串对象转换成可以解析的JSON呢？很多人会想到正则匹配然后替换，或者使用eval等渣属性进行处理。没必要这么麻烦  
```
console.log(JSON.stringify(new Function('return ' + str)()));
```
使用返回语法，你可以轻松地将任意字符串转换为其他 JavaScript数据类型。  

**模板字符串作为模板**  
比如Vue、React等现在都有自己的模板语法，比如{}语法。  
如果想直接使用 ES6 自己的语法作为模板语言，就必须使用 new Function 的能力，比如下面的 HTML：  
```
<template id="template">
 ${data.map(function (obj, index) {
return `<p>Article: ${obj.article}</p>
<p>Author: ${obj.author}</p>
`;
 }).join('')}
</template>
```
可以扩展字符串并定义一个名为 interpolate 的字符串方法来将 ES6 语法字符串转换为可执行的 ES6 代码：  
```
String.prototype.interpolate = function (params) {
const names = Object.keys(params);
const vals = Object.values(params);return new Function(…names,`return \`${this}\`;`)(…vals);
};
```
只要有对应的数据，就可以根据<template>模板获取最终编译好的HTML字符串，例如：  
```
const html = template.innerHTML.interpolate({
data: [{
article: 'Article title one',
author: 'y'
 }, {
article: 'Article title two',
author: 'h'
 }]
});
console.log(html);
```
无需任何第三方模板渲染引擎，就能使用复杂语法下的模板渲染效果  

**闭包和上下文**  
new Function 的 body 参数中变量的上下文是全局的，不是私有的，没有所谓的闭包。  
例如，下面新函数代码中的值与主函数中的值无关：  
```
function getFunc() {
let value = 'yh';
let func = new Function('console.log(value)');
return func;
}
getFunc()(); // error: value is not defined
```
如果是常规函数语法，没有问题：  
```
function getFunc() {
let value = 'yh';
let func = function () {
console.log(value)
  };
return func;
}
getFunc()(); // print 'yh'
```
**其他**  
与 new Function 语法类似的是新的RegExp，它可以使用字符串作为正则表达式的内容，特别适合动态匹配，或者增加代码混淆（一些混淆工具可以对字符串进行混淆）。  
例如，要匹配以动态值开头的属性值，可以使用以下用法：  
```
let reg = new RegExp('^' + value, 'g');
```


参考:  
[你需要深入了解一下 JavaScript 的 new Function](https://mp.weixin.qq.com/s/DDRE5GrplB-ZlBGQ083JxA)
