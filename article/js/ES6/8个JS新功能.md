# 8个JS新功能
## .at()
例如:  
``` 
lat arr=[1,2,3,4,5]
```
想获取数组中的第二位
``` 
arr[1] //2
```
最后一位的话，可能就是
``` 
arr[arr.length-1]
```
有了.at()方法就很简单了，.at()支持正索引和负索引
``` 
arr.at(-1)  //5
arr.at(-2)  //4
```
## Object.hasOwn(object, property)
Object.hasOwn(object, property)主要是用来替代Object.prototype.hasOwnProperty()。  
想要判断对象是否具有指定的对象，主要写法如下：  
``` 
if (Object.prototype.hasOwnProperty.call(object, "fn")) {
  console.log('有')
}
```
Object.hasOwn的写法:  
``` 
if (Object.hasOwn(object, "fn")) {
  console.log("有")
}
```
## 类的私有方法和getter/setter
使用#来定义私有属性/方法
``` 
class Person{
    name = '小芳';
    #age = 16;
    consoleAge(){
       console.log(this.#age)
    }
}
const person = new Person();
console.log(person.name); //小芳
console.log(button.#value); //报错
button.#value = false;//报错
```
## 检查私有属性和方法
提供in来检查私有属性和方法是否存在
``` 
class C {
  #brand;

  #method() {}

  get #getter() {}

  static isC(obj) {
    return #brand in obj && #method in obj && #getter in obj;
  }
}
```
## Top-level await（顶层await）
支持在没有async的情况下使用await
``` 
// 动态引入依赖
const strings = await import(`/i18n/${navigator.language}`);
```
允许模块使用运行时值来确定依赖关系。这对于开发/生产拆分、国际化、环境拆分等非常有用。
``` 
// 资源初始化
const connection = await dbConnector();
```
允许模块表示资源，并在模块永远无法使用的情况下产生错误。
``` 
// 加载依赖
let jQuery;
try {
  jQuery = await import('https://cdn-a.com/jQuery');
} catch {
  jQuery = await import('https://cdn-b.com/jQuery');
}
```
## 正则匹配索引
提供了一个新的/d，用来获取每个匹配的开始位置和结束位置信息
```
const str = 'The question is TO BE, or not to be, that is to be.';
const regex = /to be/gd;

const matches = [...str.matchAll(regex)];
matches[0];
```
## new Error()抛出异常的具体原因
将错误与原因相关联，，向具有属性的Error() 构造函数添加一个附加选项参数cause，其值将作为属性分配给错误实例  
``` 
async function doJob() {
  const rawResource = await fetch('//domain/resource-a')
    .catch(err => {
      throw new Error('Download raw resource failed', { cause: err });
    });
  const jobResult = doComputationalHeavyJob(rawResource);
  await fetch('//domain/upload', { method: 'POST', body: jobResult })
    .catch(err => {
      throw new Error('Upload job result failed', { cause: err });
    });
}

try {
  await doJob();
} catch (e) {
  console.log(e);
  console.log('Caused by', e.cause);
}
// Error: Upload job result failed
// Caused by TypeError: Failed to fetch
```
## 类static初始化块
对静态字段和静态私有字段的提供了一种在 ClassDefinitionEvaluation 期间执行类静态端的每个字段初始化的机制-static blocks.例如官方提供的例子：    
在没有static blocks之前，想给静态变量初始化（非直接赋值，可能是表达式赋值）的话，可能是放在外部实现：
``` 
// without static blocks:
class C {
  static x = ...;
  static y;
  static z;
}

try {
  const obj = doSomethingWith(C.x);
  C.y = obj.y
  C.z = obj.z;
}
catch {
  C.y = ...;
  C.z = ...;
}
```
有了static block的情况下：可以直接在static blocks中初始化变量：
``` 
class C {
  static x = ...;
  static y;
  static z;
  static {
    try {
      const obj = doSomethingWith(this.x);
      this.y = obj.y;
      this.z = obj.z;
    }
    catch {
      this.y = ...;
      this.z = ...;
    }
  }
```
原文: 
[这8个JS 新功能，你应该去尝试一下](https://juejin.cn/post/7054009669206933534?utm_source=gold_browser_extension)
