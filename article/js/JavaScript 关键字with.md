# JavaScript 关键字with
## 用法
使用 with 关键字来打印消息到 console：  
``` 
with (console) {
  log('I dont need the "console." part anymore!');
}
```
with 还可以用来把数组合并为字符串：  
``` 
with (console) {
  with (['a', 'b', 'c']) {
    log(join('')); // 在 console 中输出 'abc'
  }
}
```
**with做了什么？**  
> JavaScript 查找某个未使用命名空间的变量时，会通过作用域链来查找，作用域链是跟执行代码的 context 或者包含这个变量的函数有关。“with” 语句将某个对象添加到作用域链的顶部，如果在 statement 中有某个未使用命名空间的变量，跟作用域链中的某个属性同名，则这个变量将指向这个属性值。如果沒有同名的属性，则将拋出 ReferenceError 异常

当代码中存在标识符时（就像前文中代码段里的 log 或者 join ），JavaScript 会查找 作用域链 上的对象，如果其中存在一个对象的属性名和代码中的标识符 一致，JavaScript 就会使用该对象的属性值  
with 关键字允许你将任何对象注入到 作用域链 顶部。举个例子，你应该更好理解：  
``` 
with ({ myProperty: 'Hello world!' }) {
  console.log(myProperty); // 打印 "Hello world!"
}
```

## 不要使用
在大多数情况下，其实我们用临时变量就能实现同样的效果了，而且 解构赋值 语法能帮我们更方便的使用临时变量。  
with 还有很多致命的问题，MDN 列出了其中一些：
**严格模式下被禁用**  
不能在严格模式下使用 with。由于 ES module 和 class 会自动启用严格模式，所以这基本上打消了 with 语法在现代开发中使用的可能性。  
**变量遮蔽（Variable shadowing）**  
将两个数求平均，然后将结果四舍五入：  
``` 
function getAverage(min, max) {
  with (Math) {
    return round((min + max) / 2);
  }
}

getAverage(1, 5);
```
最终返回是 NaN 。为什么？因为 Math.min() 和 Math.max() 遮蔽  了函数接收的参数，所以我们其实是在将两个函数相加，其结果当然就是 NaN 了。  
如果你用了 with , 那你就不得不小心翼翼的选择标识符的命名了。你必须检查你往 with 里传了什么东西，确认其中的属性不会导致 高层作用域的 变量遮蔽 。  
使用 with 还可能引发安全漏洞。如果 传入 with 的对象 被 攻击者添加一些属性，那么就可能会遮蔽你的标识符，并会通过各种你意想不到的方式影响你程序的行为。  
从一个未经过验证的  JSON HTTP 请求体 中解析得到 JS对象 后，直接传给 with，这么做是极其危险的行为。  

**性能**  
把东西添加到 作用域链，会降低代码的运行速度，因为这增加了解析标识符到实际值时所需要搜索的对象数量。  



原文:  
[被遗忘的 JavaScript 关键字with](https://mp.weixin.qq.com/s/98f-6DMFiuVOMoAXwynItw)
