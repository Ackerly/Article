# 在 forEach 里跳出循环吗
正如 MDN 所述：
> 除非抛出异常，否则无法停止或中断 forEach() 循环。如果你需要这样的行为，那么 forEach() 方法就不是正确的工具。

如果你事先知道可能需要从循环中跳出，就不应该使用 forEach()。相反，你应该使用 for...of，这是最完美的解决方案。  

## 使用 for...of 代替
可以使用 break 关键字轻松地跳出循环  
``` 
const arr = [1, 2, 3, 4],
  doubled = [];

for (const num of arr) {
  if (num === 3) break;
  doubled.push(num * 2);
}

console.log(doubled);
```

## 使用 every() 或 some() 代替
在 JavaScript 中，every() 和 some() 是两个类似于 forEach() 的函数，不同之处在于：  
- every()：如果每个回调函数都返回 true，则返回 true
- some()：如果至少有一个回调函数返回 true，则返回 true

幸运的是，可以利用这些特点来跳出循环，具体做法如下：  
- every()：在你想要跳出循环的元素处返回 false
- some()：在你

可以利用这些特点来跳出循环，具体做法如下：  
- every()：在你想要跳出循环的元素处返回 false
- some()：在你

使用 every()：  
``` 
const arr = [1, 2, 3, 4],
  doubled = [];

arr.every(num => {
  if (num === 3) return false;
  doubled.push(num * 2);
  return true;
})

console.log(doubled);
```
使用 some()：  
``` 
const arr = [1, 2, 3, 4],
  doubled = [];

arr.some(num => {
  if (num === 3) return true;
  doubled.push(num * 2);
  return false;
})

console.log(doubled);
```

## 使用辅助变量
仍然使用 forEach() 函数，但同时使用一个控制流程的变量  
``` 
const arr = [1, 2, 3, 4],
  doubled = [];

let shouldBreak = false;

arr.forEach(num => {
  if (num === 3) shouldBreak = true;
  if (shouldBreak === false) doubled.push(num * 2);
})

console.log(doubled);
```



原文:  
[小朋友，你会在 forEach 里跳出循环吗？](https://mp.weixin.qq.com/s/vkIomNgvX-FSPm1yv8XJpw)
