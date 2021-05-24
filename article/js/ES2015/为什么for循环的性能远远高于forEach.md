# 为什么for循环的性能远远高于forEach
- for没有任何额外的函数调用栈和上下文
- forEach函数签名实际上市array.forEach(function(currentValue, index, arr), this.value)
性能上for>forEach>map，因为map会返回一个长数组，forEach不会，多以forEach大于map
## 为什么Array#Map方法在Chrome比Node慢10倍
实际是和环境的V8版本相关，等价于chrome版本还有Node版本
```
forEach不是普通的for循环语法糖，还要考虑参数上下文需要在执行的时候考虑进来
let arrs = new Array(100000);
console.time('for');
for (let i = 0; i < arrs.length; i++) {
};
console.timeEnd('for');
console.time('forEach');
arrs.forEach((arr) => {
});
console.timeEnd('forEach');

for: 2.263ms
forEach: 0.254ms
```
在10万这个级别，forEach的性能是for的十倍
```
for:2.263ms
forEach:0.254ms
```
在100万这个量级下，forEach性能和for的一致
```
for: 2.844ms
forEach: 2.652ms
```
在1000万以上的量级，forEach性能远远低于for的性能
```
for:8.422ms
forEach:30.328ms
```


参考：
[为什么普通 for 循环的性能远远高于 forEach 的性能](https://www.kancloud.cn/freya001/interview/1235136)
