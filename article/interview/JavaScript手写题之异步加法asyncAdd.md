# JavaScript 手写题之异步加法 asyncAdd  
```
// 异步加法
function asyncAdd(a,b,cb){
  setTimeout(() => {
    cb(null, a + b)
  }, Math.random() * 1000)
}

async function total(){
  const res1 = await sum(1,2,3,4,5,6,4)
  const res2 = await sum(1,2,3,4,5,6,4)
  return [res1, res2]
}

total()

// 实现下 sum 函数。注意不能使用加法，在 sum 中借助 asyncAdd 完成加法。尽可能的优化这个方法的时间。
function sum(){

}
```
- 只能修改 sum 部分的内容，sum 可接收任意长度的参数
- sum 中只能通过 asyncAdd 实现加法计算
- sum 中需要处理异步逻辑，需要使用 Promise
- 需要优化 sum 方法的计算时间

**隐藏的考察点 — setTimeout & cb**  
```
// 异步加法
function asyncAdd(a, b, cb){
  setTimeout(() => {
    cb(null, a + b)
  }, Math.random() * 1000)
}
```
最明显的就是 setTimeout 和 cb 了，其实这不难理解因为在 asyncAdd 中使用了 setTimeout 只能通过回调函数 cb 将本次计算结果返回出去，那其中的第一个参数 null 代表什么呢？其实可以认为它是一个错误信息对象，如果你比较了解 node 的话，就会知道在 node 中的异步处理的回调函数通常第一个参数就是错误对象，用于传递给外部在发生错误时自定义后续执行逻辑等。  
cb 函数会接收 错误对象 和 计算结果 作为参数传递给外部。  
**隐藏的考察点 — async & await**  
```
async function total(){
  const res1 = await sum(1,2,3,4,5,6,4)
  const res2 = await sum(1,2,3,4,5,6,4)
  return [res1, res2]
}
```
sum方法的 返回值 肯定是一个 promise 类型的，因为最前面明显的使用了 await sum(...) 的形式  
另外 total 函数返回值也必然是一个 promise 类型，因为整个 total 函数被定义为了一个 async 异步函数  
sum 需要返回 promise 类型的值，即 sum 一定会使用到 promise，并且从 sum(1,2,3,4,5,6,4) 可知 sum 可接收任意长度的参数  

**具体实现**  
实现思路如下：  
- 考虑到外部参数长度不固定，使用剩余运算符接收所有传入的参数
- 考虑到 asyncAdd 中的异步操作，将其封装为 Promise 的实现，即 caculate 函数
- 考虑到 asyncAdd 实际只能一次接收两个数字进行计算，使用循环的形式将多个参数分别传入
- 考虑到通过循环处理异步操作的顺序问题，使用 async/await 来保证正确的执行顺序，且 async 函数的返回值正好符合 sum 是 Promise 类型的要求

具体代码如下：  
```
// 通过 ES6 的剩余运算符（...） 接收外部传入长度不固定的参数
async function sum(...nums: number[]) {
    // 封装 Promise 
    function caculate(num1: number, num2: number) {
        return new Promise((resolve, reject) => {
            // 调用 asyncAdd 实现加法
            asyncAdd(num1, num2, (err: any, rs: number) => {
                // 处理错误逻辑
                if (err) {
                    reject(err);
                    return;
                }
                // 向外部传递对应的计算结果
                resolve(rs);
            });
        })
    }

    let res: any = 0;
    
    // 通过遍历将参数一个个进行计算
    for (const n of nums) {
        // 为了避免异步执行顺序问题，使用 await 等待执行结果 
        res = await caculate(res, n);
    }

    return res;
}
```
**进行优化**  
抽离内层函数  
- caculate 函数可抽离到 sum 函数外层
- asyncAdd 函数的回调函数没必要抽离，因为它依赖的参数和外部方法太多

```
function caculate(num1: number, num2: number) {
    return new Promise((resolve, reject) => {
        asyncAdd(num1, num2, (err: any, rs: number) => {
            if (err) {
                reject(err);
                return;
            }
            resolve(rs);
        });
    })
}

async function sum(...nums: number[]) {
    let res: any = 0;

    for (const n of nums) {
        res = await caculate(res, n);
    }

    return res;
}
```
**缓存计算结果**  
total 方法，其中 sum 调用了两次，而且参数还是一模一样的，目的就是提示你在第二次计算相同内容时结果直接 从缓存中获取，而不是在通过异步计算。  
```
async function total(){
  const res1 = await sum(1,2,3,4,5,6,4)
  const res2 = await sum(1,2,3,4,5,6,4)
  return [res1, res2]
}
```
一个简单的缓存方案的实现，具体实现如下：  
```
async function sum(...nums: number[]) {
    let res: any = 0;
    const key = nums.join('+');

    if (!isUndefined(cash[key])) return cash[key];

    for (const n of nums) {
        res = await caculate(res, n);
    }

    cash[key] = res;

    return res;
}

function caculate(num1: number, num2: number) {
    return new Promise((resolve, reject) => {
        asyncAdd(num1, num2, (err: any, rs: number) => {
            if (err) {
                reject(err);
                return;
            }
            resolve(rs);
        });
    })
}
```


原文:  
[一道简单又有意思的 JavaScript 手写题 — 异步加法 asyncAdd](https://mp.weixin.qq.com/s/5MklZgOo7wy-8f7MtxcgCA)