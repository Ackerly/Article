# JS 中的正则
## 一般在哪里用得到正则  
**1.RegExp.prototype.test()**  
> test() 方法执行一个检索，用来查看正则表达式与指定的字符串是否匹配。返回 true 或 false。

``` 
function test(str: string): boolean;
```
若正则对象带了全局标志符号时，test() 的执行会改变正则表达式的 lastIndex 属性。连续执行 test() 方法，后续的执行将会从 lastIndex 处开始匹配字符串。  
``` 
var regex = /foo/g;
var str = 'foo bar foo bar';
console.log(regex.lastIndex); // 0  初始值为 0
regex.test(str); // true
console.log(regex.lastIndex); // 3
regex.test(str); // true
console.log(regex.lastIndex); // 11
regex.test(str); // false
console.log(regex.lastIndex); // 0  匹配为 false 后将 lastIndex 重置为 0
```
**2.RegExp.prototype.exec()**  
> 在一个指定字符串中执行一个搜索匹配。返回一个结果数组或 null

正常情况下，如果匹配成功，则返回一个数组，数组的第 0 项是匹配的所有字符串，第 1 项及以后的表示括号中的分组捕获，index 表示匹配到的索引  
``` 
var regex = /b(a)r/;
var str = 'foo bar foo bar';
regex.exec(str); // [ 'bar', 'a', index: 4, input: 'foo bar foo bar', groups: undefined ]
```
如果正则对象带了全局标志符号时，exec 的执行也可以被多次执行，只不过该正则对象的 lastIndex 会随着改变。当匹配失败后，exec() 方法返回 null，并将 lastIndex 重置为 0  
``` 
var regex = /bar/g;
var str = 'foo bar foo bar';
console.log(regex.lastIndex); // 0
regex.exec(str); // [ 'bar', index: 4, input: 'foo bar foo bar', groups: undefined ]
console.log(regex.lastIndex); // 7
regex.exec(str); // [ 'bar', index: 12, input: 'foo bar foo bar', groups: undefined ]
console.log(regex.lastIndex); // 15
regex.exec(str); // null
console.log(regex.lastIndex); // 0
```
**3. String.prototype.search()**  
> search() 方法执行正则表达式和 String 对象之间的一个搜索匹配

该方法传入一个正则表达式对象（若为非正则表达式会隐式地转换成正则表达式对象new RegExp(regexp)），返回正则表达式在字符串中首次匹配项的索引；否则，返回 -1。
``` 
function search(reg: RegExp): number;
```
**4. String.prototype.match()**  
> 检索返回一个字符串匹配正则表达式的结果。

如果未使用 g 标志，则仅返回第一个完整匹配及其相关的捕获组（Array）。在这种情况下，返回的项目将具有如下所述的其他属性  
``` 
var regex = /bar/;
var str = 'foo bar foo bar';
regex.exec(str); // [ 'bar', 'a', index: 4, input: 'foo bar foo bar', groups: undefined ]
```
如果使用 g ，则将返回与完整正则表达式匹配的所有结果，但不会返回捕获组。  
``` 
var regex = /bar/g;
var str = 'foo bar foo bar';
regex.exec(str); // [ 'bar', 'bar' ]
```
**5. String.prototype.replace**  
> 返回一个由替换值（replacement）替换部分或所有的模式（pattern）匹配项后的新字符串。模式可以是一个字符串或者一个正则表达式，替换值可以是一个字符串或者一个每次匹配都要调用的回调函数。如果 pattern 是字符串，则仅替换第一个匹配项。

## 实际场景
**命名方式的转换**  
假如存在这样一个对象 origin， 需要实现一个函数 format，将 origin 的下划线键名转换为 target 的驼峰式键名：  
``` 
// 待转换的对象
var origin = {
  first_name: 'a',
  last_name: 'b',
  say_hi: function () {
    console.log('hi');
  },
  dream_to_be: 'teacher',
  best_friends: [
    {
      first_name: 'c',
      last_name: 'd',
      favorite_sport: 'basketball',
    },
    {
      first_name: 'e',
      last_name: 'f',
      say_hi: function () {
        console.log('hello');
      },
    },
  ],
};

// 期待转换后的对象
var target = {
  firstName: 'a',
  lastName: 'b',
  sayHi: function () {
    console.log('hi');
  },
  dreamToBe: 'teacher',
  bestFriends: [
    {
      firstName: 'c',
      lastName: 'd',
      favoriteSport: 'basketball',
    },
    {
      firstName: 'e',
      lastName: 'f',
      sayHi: function () {
        console.log('hello');
      },
    },
  ],
};

// 需要实现的转化函数
function format(origin) {}
```
实现思路：遍历源对象的 key，用正则将其命名风格转换过来，如果该 key 对应的 value 是个对象或数组则递归地遍历它。  
``` 
const formatKey = (key) => {
  return key.replace(/_(\w)/g, ($0, $1) => {
    return $1.toUpperCase();
  });
};
const isObject = (obj) => typeof obj === 'object' && obj !== null;
const format = (origin) => {
  if (isObject(origin)) {
    return Object.keys(origin).reduce((target, key) => {
      target[formatKey(key)] = format(origin[key]);
      return target;
    }, {});
  } else if (Array.isArray(origin)) {
    return origin.map(format);
  } else {
    return origin;
  }
};
```
**金额千分位分割**  
给定一个较长字符串表示的金额，返回一个千分位分割的字符串。举例：  
``` 
//  待转化的字符串
var originStr = '123456789.00';

// 期待转换后的字符串
var targetStr = '123,456,789.00';

// 需要实现的转化函数
function format(originStr) {}
```
写这个正则需要了解位置匹配(?=p) 和 (?!p)。(?=p)，其中 p 是一个子模式，即 p 前面的位置，或者说，该位置后面的字符要匹配 p。  
比如 (?=l)，表示 "l" 字符前面的位置，例如：  
``` 
var result = 'hello'.replace(/(?=l)/g, '#');
console.log(result);
// => "he#l#lo"
```
而 (?!p) 就是 (?=p) 的反面意思，比如：  
``` 
var result = 'hello'.replace(/(?!l)/g, '#');
console.log(result);
// => "#h#ell#o#"
```
具体实现：  
``` 
function format(originStr) {
  var regex = /(?!^)(?=(\d{3})+\.)/g;
  return originStr.replace(regex, ',');
}
```
**模板字符串函数**  
实现类似 ES6 中模板字符串功能的函数，将字符串中特定分割的字符替换为对象中对应的值。  
``` 
//  输入的字符串和对象
var originStr = 'Hello, ${name}, your grade is ${grade}.';
var obj = {
  name: 'Micah',
  grade: 90,
};

// 期待输出的字符串
var targetStr = 'Hello, Micah, your grade is 90.';

// 需要实现的转化函数
function format(originStr, obj) {}
```
用正则中捕获组的技巧提取位于${和}的变量名，再将其与对象中的值替换：  
``` 
function format(originStr, obj) {
  return originStr.replace(/\$\{(\w*)\}/g, (_, key) => {
    return obj[key];
  });
}
```

原文:  
[JS 中的正则](https://mp.weixin.qq.com/s/6QKe1zH-rG33SZUvJ9mhwQ)
