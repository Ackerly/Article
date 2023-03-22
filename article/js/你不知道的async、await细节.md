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

> 结论：async函数在抛出返回值时，会根据返回值类型开启不同数目的微任务
> return结果值：非thenable、非promise（不等待）
> return结果值：thenable（等待 1个then的时间）
> return结果值：promise（等待 2个then的时间）

``` 
async function testA () {
    return 1;
}

testA().then(() => console.log(1));
Promise.resolve()
    .then(() => console.log(2))
    .then(() => console.log(3));

// (不等待)最终结果: 1 2 3
```
例子2:
``` 
async function testB () {
    return {
        then (cb) {
            cb();
        }
    };
}

testB().then(() => console.log(1));
Promise.resolve()
    .then(() => console.log(2))
    .then(() => console.log(3));
```
例子3：
``` 
async function testC () {
    return new Promise((resolve, reject) => {
        resolve()
    })
}

testC().then(() => console.log(1));
Promise.resolve()
    .then(() => console.log(2))
    .then(() => console.log(3));
    
// (等待两个then)最终结果👉: 2 3 1




async function testC () {
    return new Promise((resolve, reject) => {
        resolve()
    })
} 

testC().then(() => console.log(1));
Promise.resolve()
    .then(() => console.log(2))
    .then(() => console.log(3))
    .then(() => console.log(4))

// (等待两个then)最终结果👉: 2 3 1 4
```

经典面试题  
``` 
async function async1 () {
    console.log('1')
    await async2()
    console.log('AAA')
}

async function async2 () {
    console.log('3')
    return new Promise((resolve, reject) => {
        resolve()
        console.log('4')
    })
}

console.log('5')

setTimeout(() => {
    console.log('6')
}, 0);

async1()

new Promise((resolve) => {
    console.log('7')
    resolve()
}).then(() => {
    console.log('8')
}).then(() => {
    console.log('9')
}).then(() => {
    console.log('10')
})
console.log('11')

// 最终结果👉: 5 1 3 4 7 11 8 9 AAA 10 6
```
步骤拆分👇：
1. 先执行同步代码，输出5
2. 执行setTimeout，是放入宏任务异步队列中
3. 接着执行async1函数，输出1
4. 执行async2函数，输出3
5. Promise构造器中代码属于同步代码，输出4(async2函数的返回值是Promise，等待2个then后放行，所以AAA暂时无法输出  )
6. async1函数暂时结束，继续往下走，输出7
7. 同步代码，输出11
8. 执行第一个then，输出8
9. 执行第二个then，输出9
10. 终于等到了两个then执行完毕，执行async1函数里面剩下的，输出AAA
11. 再执行最后一个微任务then，输出10
12. 执行最后的宏任务setTimeout，输出6

## await 右值类型区别
**非 thenable**  
例子1：
``` 
async function test () {
    console.log(1);
    await 1;
    console.log(2);
}

test();
console.log(3);
// 最终结果: 1 3 2
```
例子2：  
``` 
function func () {
    console.log(2);
}

async function test () {
    console.log(1);
    await func();
    console.log(3);
}

test();
console.log(4);

// 最终结果: 1 2 4 3
```
例子3：  
``` 
async function test () {
    console.log(1);
    await 123
    console.log(2);
}

test();
console.log(3);

Promise.resolve()
    .then(() => console.log(4))
    .then(() => console.log(5))
    .then(() => console.log(6))
    .then(() => console.log(7));

// 最终结果: 1 3 2 4 5 6 7
```
> await后面接非 thenable 类型，会立即向微任务队列添加一个微任务then，但不需等待

**thenable类型**  
``` 
async function test () {
    console.log(1);
    await {
        then (cb) {
            cb();
        },
    };
    console.log(2);
}

test();
console.log(3);

Promise.resolve()
    .then(() => console.log(4))
    .then(() => console.log(5))
    .then(() => console.log(6))
    .then(() => console.log(7));

// 最终结果👉: 1 3 4 2 5 6 7
```
> await 后面接 thenable 类型，需要等待一个 then 的时间之后执行

**Promise类型**  
``` 
async function test () {
    console.log(1);
    await new Promise((resolve, reject) => {
        resolve()
    })
    console.log(2);
}

test();
console.log(3);

Promise.resolve()
    .then(() => console.log(4))
    .then(() => console.log(5))
    .then(() => console.log(6))
    .then(() => console.log(7));

// 最终结果👉: 1 3 2 4 5 6 7
```



原文:  
[你不知道的 async、await 魔鬼细节](https://mp.weixin.qq.com/s/Gr7ajRYYazdm5YQvZS6gXg)
