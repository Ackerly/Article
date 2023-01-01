# 为什么“false == []”和“false == ![]”都返回true？
**为什么“false == []”和“false == ![]”都返回true**  
遇到一个布尔值和一个对象进行比较时，会将这两个值转换为数字进行最后的比较。  
``` 

// 1. Convert false to a number to get 0
// 2. Convert [] to a number to get 0
// 3. "0 == 0" Returns true
console.log(false == []) // true
// 1. The result of executing "![]" is false
// 2. false == false Returns true
console.log(false == ![]) // true
```
**为什么“[] == ![]”返回true？**  
为什么空数组如此特别？  
``` 
// 1. The result of executing "![]" is false
// 2. Next, compare "[] == false"
// 3. Convert [] to a number to get 0
// 4. Convert false to a number to get 0
// 5. "0 == 0" Returns true

console.log([] == ![])
```
**关于奇怪的“try catch”**  
getName执行返回的是fatfish，还是我medium  
``` 
const getName = () => {
  try {
    return 'fatfish'
  } finally {
    return 'medium'
  }
}
getName() // ?
```
答案是“medium”  
因为在“try….catch….finally”语句中，finally子句无论是否抛出异常都会被执行。另外，如果抛出异常，即使没有catch子句处理异常，finally子句中的语句也会被执行。  
**关于箭头功能**  
``` 
const fn = () => 'fatfish'
console.log(fn()) // fatfish
```
这段代码会输出什么
``` 
onst fn = () => {}

console.log(fn()) // ?
```
‘{}’是最终结果吗？未定义的是最后的赢家。因为‘{}’是fn函数的一个包含块，所以它等价于下面的代码。  
``` 
const fn = () = {
}

console.log(fn()) // understand
```
**为什么 JSON.stringify('fatfish') ! ==‘fatfish’？**  
``` 
const name1 = JSON.stringify('fatfish')
const name2 = 'fatfish'

console.log(name1 === name2) // ?
```
为什么name1不等于name2？  
``` 
const name1 = JSON.stringify('fatfish') // => '"fatfish"'
const name2 = 'fatfish'

console.log(name1 === name2) // '"fatfish"' === 'fatfish'  => false
```


原文:  
[面试官：为什么“false == []”和“false == ![]”都返回true？](https://mp.weixin.qq.com/s/g_5p-YSIzDtMSjGm5-metA)
