# ES6使用
## 取值
从对象obj中取值
``` 
const obj = {
    a:1,
    b:2,
    c:3,
    d:4,
    e:5,
}
```
bad
``` 
const a = obj.a;
const b = obj.b;
const c = obj.c;
const d = obj.d;
const e = obj.e;
const f = obj.a + obj.d;
const g = obj.c + obj.e;
```
good
``` 
const {a,b,c,d,e} = obj;
const f = a + d;
const g = c + e;
```
如果想创建的变量名和对象的属性名不一致，可以这么写：
``` 
const {a:a1} = obj;
console.log(a1);// 1
```
注意：解构的对象不能为undefined、null。否则会报错
## 数据合并
bad
``` 
const a = [1,2,3];
const b = [1,5,6];
const c = a.concat(b);//[1,2,3,1,5,6]

const obj1 = {
  a:1,
}
const obj2 = {
  b:1,
}
const obj = Object.assign({}, obj1, obj2);//{a:1,b:1}
```
good
``` 
const a = [1,2,3];
const b = [1,5,6];
const c = [...new Set([...a,...b])];//[1,2,3,5,6]

const obj1 = {
  a:1,
}
const obj2 = {
  b:1,
}
const obj = {...obj1,...obj2};//{a:1,b:1}
```
## 字符串拼接
bad
``` 
const name = '小明';
const score = 59;
let result = '';
if(score > 60){
  result = `${name}的考试成绩及格`; 
}else{
  result = `${name}的考试成绩不及格`; 
}
```
good
``` 
const name = '小明';
const score = 59;
const result = `${name}${score > 60?'的考试成绩及格':'的考试成绩不及格'}`;
```
## if判断条件的优化
bad
``` 
if(
    type == 1 ||
    type == 2 ||
    type == 3 ||
    type == 4 ||
){
   //...
}
```
good
``` 
const condition = [1,2,3,4];

if( condition.includes(type) ){
   //...
}
```
## 列表搜索
bad
``` 
const a = [1,2,3,4,5];
const result = a.filter( 
  item =>{
    return item === 3
  }
)
```
good
``` 
const a = [1,2,3,4,5];
const result = a.find( 
  item =>{
    return item === 3
  }
)
```
find方法中找到符合条件的项，就不会继续遍历数组。
## 扁平化数组的优化
bad
``` 
const deps = {
'采购部':[1,2,3],
'人事部':[5,8,12],
'行政部':[5,14,79],
'运输部':[3,64,105],
}
let member = [];
for (let item in deps){
    const value = deps[item];
    if(Array.isArray(value)){
        member = [...member,...value]
    }
}
member = [...new Set(member)]
```
good
``` 
const deps = {
    '采购部':[1,2,3],
    '人事部':[5,8,12],
    '行政部':[5,14,79],
    '运输部':[3,64,105],
}
let member = Object.values(deps).flat(Infinity);
```
使用Infinity作为flat的参数，使得无需知道被扁平化的数组的维度。(flat方法不支持IE浏览器。)
## 获取对象属性值
bad
``` 
const name = obj && obj.name;
```
good
``` 
const name = obj?.name;
```
## 添加对象属性
bad
``` 
let obj = {};
let index = 1;
let key = `topic${index}`;
obj[key] = '话题内容';
```
good
``` 
let obj = {};
let index = 1;
obj[`topic${index}`] = '话题内容';
```
## 输入非空的判断
bad
``` 
if(value !== null && value !== undefined && value !== ''){
    //...
}
```
good
``` 
if(value??'' !== ''){
  //...
}
```
## 异步函数
bad
``` 
const fn1 = () =>{
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(1);
    }, 300);
  });
}
const fn2 = () =>{
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(2);
    }, 600);
  });
}
const fn = () =>{
   fn1().then(res1 =>{
      console.log(res1);// 1
      fn2().then(res2 =>{
        console.log(res2)
      })
   })
}
```
good
``` 
const fn = async () =>{
  const res1 = await fn1();
  const res2 = await fn2();
  console.log(res1);// 1
  console.log(res2);// 2
}
```
做并发请求时，还是要用到Promise.all()
``` 
const fn = () =>{
   Promise.all([fn1(),fn2()]).then(res =>{
       console.log(res);// [1,2]
   }) 
}
```
如果并发请求时，只要其中一个异步函数处理完成，就返回结果，要用到Promise.race()

参考:  
[你会用ES6，那倒是用啊！](https://juejin.cn/post/7016520448204603423)
