# Web Worker 项目实践
Web Workers 宏观语义上包含了三种不同的 Worker：DedicatedWorker(专有worker)、 SharedWorker(共享Worker)、 ServiceWorker  
## 引入 Web Worker  
1、兼容性如何？  
2、使用场景在哪？  

Web Workers 是 2009 年的提案，2012 年各大浏览器已经基本支持，11 年过去了，现在使用已经完全没有问题啦  
问题 2，主要考虑了以下 3 点：  
1. Worker API 的局限性：同源限制、无 DOM 对象、异步通信，因此适合不涉及 DOM 操作的任务
2. Worker 的使用成本：创建时间 + 数据传输时间；考虑到可以预创建，可以忽略创建时间，只考虑数据传输成本，这里可参考 19 年的一个测试 Is postMessage slow，简要结论是比较乐观的，大部分设备和数据情况下速度不是瓶颈  
3. 任务特点：需要是可并行的多任务，为了充分利用多核能力，可并行的任务数越接近 CPU 数量，收益会越高。多线程场景的收益计算，可以参考 Amdahl 公式，其中 F 是初始化所需比例，N 是可并行数  

## Worker 实践
一个问题出现了：为什么一个兼容性良好，能够发挥并发能力的技术（听起来很有诱惑力），到现在还没有大规模使用呢？  
一是暂无匹配度完美的使用场景，因此引入被搁置了；二是 worker api 设计得太难用，参考很多 demo 看，限制多配置还麻烦，让人望而却步。  

**Worker 到底有多难用**  
``` 
// index.js
const worker = new Worker('./worker.js')
worker.onmessage = function (messageEvent) {
  console.log(messageEvent)
}

// worker.js
importScripts('constant.js')
function a() {
  console.log('test')
}
```
其中问题有：  
1. postMessage 传递消息的方式不适合现代编程模式，当出现多个事件时就涉及分拆解析和解决耦合问题，因此需要改造
2. 新建 worker 需要单独文件，因此项目内需要处理打包拆分逻辑，独立出 worker 文件
3. worker 内可支持定义函数，可通过importScript 方式引入依赖文件，但是都独立于主线程文件，依赖和函数的复用都需要改造
4. 多线程环境必然涉及同步运行多个 worker，多 worker 的启动、复用和管理都需要自行处理

**类库调研**  
常见的几款 worker 类库，其中我们可能会关注的关键能力有：
1. 通信是否有包装成更好用的方式，比如 promise 化或者 rpc 化
2. 是否可以动态创建函数——可以增加 worker 灵活性
3. 是否包含多 worker 的管理能力，也就是线程池
4. 考虑 node 的使用场景，是否可以跨端运行

**有类库加持的 worker 现状**  
通过使用 workerpool，我们可以在主线程文件内新建 worker；它自动处理多 worker 的管理；可以执行 worker 内定义好的函数 a；可以动态创建一个函数并传入参数，让 worker 来执行。  
``` 
// index.js
import workerpool from 'workerpool'
const pool = workerpool.pool('./worker.js')
// 执行一个 worker 内定义好的函数
pool.exec('a', [1, 2]).then((res) => {
  console.log(res)
})
// 执行一个自定义函数
pool
  .exec(
    (x, y) => {
      return x + y
    }, // 自定义函数体
    [1, 2], // 自定义函数参数
  )
  .then((res) => {
    console.log(res)
  })

// worker.js
importScripts('constant.js')
function a() {
  console.log('test')
}
```
**向着舒适无感的 worker 编写前进**    
期望的目标是：  
1. 足够灵活：可以随意编写函数，今天我想计算1+1，明天我想计算1+2，这些都可以动态编写，最好它可以直接写在主线程我自己的文件里，不需要我跑到 worker 文件里去改写；  
2. 足够灵活：可以随意编写函数，今天我想计算1+1，明天我想计算1+2，这些都可以动态编写，最好它可以直接写在主线程我自己的文件里，不需要我跑到 worker 文件里去改写

workerpool 具备了动态创建函数的能力，第一点已经可以实现；而第二点关于依赖的管理，则需要自行搭建，接下来介绍搭建步骤。  
1. 抽取依赖，管理编译和更新：
   新增一个依赖管理文件worker-depts.js，可按照路径作为 key 名构建一个聚合依赖对象，然后在 worker 文件内引入这份依赖
    ``` 
    // worker-depts.js
    import * as _ from 'lodash-es'
    import * as math from '../math'
    
    const workerDepts = {
    _,
    'util/math': math,
    }
    
    export default workerDepts
   // worker.js
    import workerDepts from '../util/worker/worker-depts'
    ```
2. 定义公共调用函数，引入所打包的依赖并串联流程
   worker 内定义一个公共调用函数，注入 worker-depts 依赖，并注册在 workerpool 的方法内
    ```
    // worker.js
    import workerDepts from '../util/worker/worker-depts'
    
    function runWithDepts(fn: any, ...args: any) {
    var f = new Function('return (' + fn + ').apply(null, arguments);')
    return f.apply(f, [workerDepts].concat(args))
    }
    
    workerpool.worker({
    runWithDepts,
    })
    ```
   主线程文件内定义相应的调用方法，入参是自定义函数体和该函数的参数列表
    ``` 
    // index.js
    import workerpool from 'workerpool'
    export async function workerDraw(fn, ...args) {
      const pool = workerpool.pool('./worker.js')
      return pool.exec('runWithDepts', [String(fn)].concat(args))
    }
    ```
   完成以上步骤，就可以在项目任意需要调用 worker 的位置，像下面这样，自定义函数内容，引用所需依赖（已注入在函数第一个参数），进行使用了。  
   这里引用了一个项目内的公共函数 fibonacci，也引用了一个 lodash 的 map 方法，都可以在depts 对象上取到  
    ```
    // 项目内需使用worker时
    const res = await workerDraw(
    (depts, m, n) => {
    const { map } = depts['_']
    const { fibonacci } = depts['util/math']
    return map([m, n], (num) => fibonacci(num))
    },
    input1,
    input2,
    )
    ```
3. 优化语法支持  
   没有语法支持的依赖管理是很难用的，通过对 workerDraw 进行 ts 语法包装，可以实现在使用时的依赖提示：  
    ``` 
    import workerpool from 'workerpool'
    import type TDepts from './worker-depts'
    
    export async function workerDraw<T extends any[], R>(fn: (depts: typeof TDepts, ...args: T) => Promise<R> | R, ...args: T) {
    const pool = workerpool.pool('./worker.js')
    return pool.exec('runWithDepts', [String(fn)].concat(args))
    }
    ```
4. 其他问题  
   新增了 worker 以后，出现了 window和 worker 两种运行环境，如果你恰好和我一样需要兼容 node 端运行，那么运行环境就是三种，原本我们通常判断 window 环境使用的也许是 typeof window === 'object'这样，现在不够用了，这里可以改为 globalThis 对象，它是三套环境内都存在的一个对象，通过判断globalThis.constructor.name的值，值分别是'Window' / 'DedicatedWorker'/ 'Object'，从而实现环境的区分

原文:  
[2023 年的 Web Worker 项目实践](https://mp.weixin.qq.com/s/y5ADlfVINr17nbJ3r-y9xQ)
