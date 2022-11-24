# ES6 之 Promise 实践应用与控制反转
**Promise 含义及基本介绍**  
Promise 也是一个类或构造函数，是 JS 原生提供的，和我们自定义的类一样，通过对它进行实例化后，来完成预期的异步任务处理。  
Promise 接受异步任务并立即执行，然后在任务完成后，将状态标注成最终结果（成功或失败）  
Promise 有三种状态：初始化时，刚开始执行主体任务，这时它的初始状态时  pending（进行中） ；等到任务执行完成，这时根据成功或失败，分别对应状态 fulfilled（成功）和  rejected（失败） ，这时的状态就固定不能被改变了，即  Promise 状态是不可逆的。  
**基本用法**  
Promise 就是一个类，所以使用时，我们照常 new 一个实例即可。  
``` 
const myPromise = new Promise((resolve, reject) => {
  // 这里是 Promise 主体，执行异步任务
  ajax('xxx', () => {
     resolve('成功了'); // 或 reject('失败了')
  })
})
```
上面创建好 Promise 实例后，里面的主体会立即执行，比如，如果是发送请求，则会立即把请求发出去，如果是定时器，则会立即启动计时。至于请求什么时候返回，我们就在返回成功的地方，通过 resolve() 将状态标注为成功即可，同时 resolve(data) 可以附带着返回数据。然后在 then() 里面进行回调处理。

原文:  
[前端 ES6 之 Promise 实践应用与控制反转](https://mp.weixin.qq.com/s/pxZz6YJQWvomlm0dhoS_Uw)

