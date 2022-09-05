# 为什么Proxy一定要配合Reflect使用
Proxy代理，它内置了一系列”陷阱“用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）  
Reflect反射，它提供拦截 JavaScript 操作的方法。这些方法与 Proxy的方法相同。  
通过 Proxy 创建对于原始对象的代理对象，从而在代理对象中使用 Reflect 达到对于 JavaScript 原始操作的拦截。  

**单独使用 Proxy**  
```
const obj = {
  name: 'wang.haoyu',
};

const proxy = new Proxy(obj, {
  // get陷阱中target表示原对象 key表示访问的属性名
  get(target, key) {
    console.log('劫持你的数据访问' + key);
    return target[key]
  },
});
```
通过 Proxy 创建了一个基于 obj 对象的代理，同时在 Proxy 中声明了一个 get 陷阱。当访问 proxy.name 时实际触发了对应的 get 陷阱，它会执行 get 陷阱中的逻辑，同时会执行对应陷阱中的逻辑，最终返回对应的 target[key] 也就是所谓的 wang.haoyu .   
**Proxy 中的 receiver**  
```
const obj = {
  name: 'wang.haoyu',
};

const proxy = new Proxy(obj, {
  // get陷阱中target表示原对象 key表示访问的属性名
  get(target, key, receiver) {
    console.log(receiver === proxy);
    return target[key];
  },
});

// log: true
proxy.name;
```
在 Proxy 实例对象的 get 陷阱上接收了 receiver 这个参数,同时，我们在陷阱内部打印 console.log(receiver === proxy); 它会打印出 true ，表示这里 receiver 的确是和代理对象相等的  
另外一个例子:  
```
const parent = {
  get value() {
    return '19Qingfeng';
  },
};

const proxy = new Proxy(parent, {
  // get陷阱中target表示原对象 key表示访问的属性名
  get(target, key, receiver) {
    console.log(receiver === proxy);
    return target[key];
  },
});

const obj = {
  name: 'wang.haoyu',
};

// 设置obj继承与parent的代理对象proxy
Object.setPrototypeOf(obj, proxy);
```
同样在 proxy 对象的 get 陷阱上打印了 console.log(receiver === proxy); 结果却是 false 。  
```
...
const proxy = new Proxy(parent, {
  // get陷阱中target表示原对象 key表示访问的属性名
  get(target, key, receiver) {
+   console.log(receiver === proxy) // log:false
-   console.log(receiver === obj) // log:true
    return target[key];
  },
});
...
```
get 陷阱中的 receiver 存在的意义就是为了正确的在陷阱中传递上下文。  
 Proxy 中 get 陷阱的 receiver 不仅仅代表的是 Proxy 代理对象本身，同时也许他会代表继承 Proxy 的那个对象  
 其实本质上来说它还是为了确保陷阱函数中调用者的正确的上下文访问，比如这里的 receiver 指向的是 obj 。  
 ```
 const parent = {
  get value() {
    return '19Qingfeng';
  },
};

const handler = {
  get(target, key, receiver) {
    console.log(this === handler); // log: true
    console.log(receiver === obj); // log: true
    return target[key];
  },
};

const proxy = new Proxy(parent, handler);

const obj = {
  name: 'wang.haoyu',
};

// 设置obj继承与parent的代理对象proxy
Object.setPrototypeOf(obj, proxy);

// log: false
obj.value
```
**Reflect 中的 receiver**  
```
const parent = {
  name: '19Qingfeng',
  get value() {
    return this.name;
  },
};

const handler = {
  get(target, key, receiver) {
    return Reflect.get(target, key);
    // 这里相当于 return target[key]
  },
};

const proxy = new Proxy(parent, handler);

const obj = {
  name: 'wang.haoyu',
};

// 设置obj继承与parent的代理对象proxy
Object.setPrototypeOf(obj, proxy);

// log: false
console.log(obj.value);
```
代码分析：  
- 调用 obj.value 时，由于 obj 本身不存在 value 属性
- 继承的 proxy 对象中存在 value 的属性访问操作符，所以会发生屏蔽效果
- 此时会触发 proxy 上的 get value() 属性访问操作
- 同时由于访问了 proxy 上的 value 属性访问器，所以此时会触发 get 陷阱。
- 进入陷阱时，target 为源对象也就是 parent ，key 为 value 
- 陷阱中返回 Reflect.get(target,key) 相当于 target[key]
- 不知不觉中 this 指向在 get 陷阱中被偷偷修改掉了！！
- 原本调用方的 obj 在陷阱中被修改成为了对应的 target 也就是 parent 。
- 自然而然打印出了对应的 parent[value] 也就是 19Qingfeng 。

这显然不是我们期望的结果，当访问 obj.value 时，希望应该正确输出对应的自身上的 name 属性也就是所谓的 obj.value => wang.haoyu 。    
Relfect 中 get 陷阱的 receiver 就大显神通了
```
const parent = {
  name: '19Qingfeng',
  get value() {
    return this.name;
  },
};

const handler = {
  get(target, key, receiver) {
-   return Reflect.get(target, key);
+   return Reflect.get(target, key, receiver);
  },
};

const proxy = new Proxy(parent, handler);

const obj = {
  name: 'wang.haoyu',
};

// 设置obj继承与parent的代理对象proxy
Object.setPrototypeOf(obj, proxy);

// log: wang.haoyu
console.log(obj.value);
```
上述代码原理其实非常简单：  
-  Proxy 中 get 陷阱的 receiver 不仅仅会表示代理对象本身同时也还有可能表示继承于代理对象的对象，具体需要区别与调用方。这里显然它是指向继承与代理对象的 obj 。
-  Reflect 中 get 陷阱中第三个参数传递了 Proxy 中的 receiver 也就是 obj 作为形参，它会修改调用时的 this 指向。

> 可以简单的将 Reflect.get(target, key, receiver) 理解成为 target[key].call(receiver)


原文: 
[为什么Proxy一定要配合Reflect使用？](https://mp.weixin.qq.com/s/A1uRq0XwhZPRIZetrEFM0g)