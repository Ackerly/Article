# 斐波那契数列
**斐波那契数列**  
斐波那契数列是一个由 0、1、1、2、3、5、8、13、21、34 等数组成的序列  
序列前两位固定值是 0, 1，从第三位开始，每个数值都是前两位数相加之和，以此不断累加  
比如数值 3 由 1+2 得到，数值 5 由 2+3 得到。根据这个规则可以推断，在 n 位置的斐波那契数，是 n-2 位置的数值加上 n-1 位置的数值  
循环实现斐波那契数列：  
``` 
function fibonacci(n) {
  if(n === 0) return 0;
  if(n <= 2) return 1;
  
  let value = 0;
  let n_2 = 1
  let n_1 = 1
  for(let i = 3; i <= n; i++) {
    value = n_2 + n_1
    n_2 = n_1
    n_1 = value
  }
  return value
}
```
上述代码中，n_2 表示 n-2 位置的值，n_1 表示 n-1 位置的值。从位置 3 处开始，在循环体中计算 n_2 + n_1 的值，最后赋予 value，并且将 n_2 和 n_1 的值后移一位，以供下一次循环使用。  
这样通过循环 + 三个变量，实现了斐波那契数列  
**递归实现斐波那契数列**  
前面我们推断出，在 n 位置的斐波那契数，是 n-2 位置的数值加上 n-1 位置的数值，所以表达式就是：  
``` 
f(n) = f(n-2) + f(n-1)
```
表达式如上，终止条件呢就是 n <= 2 时返回固定值，因为数列前三个值是固定的。所以最终的递归函数如下：  
``` 
function fibonacci(n) {
  if(n <= 0) return 0;
  if(n <= 2) return 1;
  return fibonacci(n-2) + fibonacci(n-1)
}
```
**记忆化斐波那契数**  
记忆化的含义就是将前面计算的值缓存下来，根据这些已有值计算出新值。新值再缓存下来，当后面需要这些一层层缓存下来的值时，可以直接拿来使用  
递归重复执行函数会降低性能，可以通过缓存值，也就是记忆化来优化逻辑  
记忆化的 fibonacci 函数：  
``` 
function fibonacciMemoization(n) { 
  const memo = [0, 1];  
  const fibonacci = (n) => { 
    if (memo[n] != null) return memo[n];
    return memo[n] = fibonacci(n - 1) + fibonacci(n - 2);
  };
  return fibonacci(n)
}
```
这个函数使用我们非常熟悉的闭包，将计算过的值缓存在了 memo 这个数组中。这样的话只要计算过的值都会被复用，减少了多余的函数调用。

参考:  
[怒肝 JavaScript 数据结构 — 斐波那契数列](https://mp.weixin.qq.com/s/kQt_KudoRDgRcBqmHclmRw)
