# 你不知道的async、await细节
``` 
async function async1 () {
    await new Promise((resolve, reject) => {
        resolve()
    })
    console.log('A')
}

async1()

new Promise((resolve) => {
    console.log('B')
    resolve()
}).then(() => {
    console.log('C')
}).then(() => {
    console.log('D')
})

// 最终结果: B A C D
```
例子2
``` 
async function async1 () {
    await new Promise((resolve, reject) => {
        resolve()
    })
    console.log('A')
}

async1()

new Promise((resolve) => {
    console.log('B')
    resolve()
}).then(() => {
    console.log('C')
}).then(() => {
    console.log('D')
})

// 最终结果: B A C D
```
**async 函数返回值**  
async 函数处理返回值的问题，它会像 Promise.prototype.then 一样，会对返回值的类型进行辨识。  
_根据返回值的类型，引起 js引擎 对返回值处理方式的不同_  


原文:  
[你不知道的 async、await 魔鬼细节](https://mp.weixin.qq.com/s/Gr7ajRYYazdm5YQvZS6gXg)
