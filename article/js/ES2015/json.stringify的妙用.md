#  JSON.stringify妙用
## 基本用法
JSON.stringify()可以吧一个JavaScript对象序列化为一个JSON字符串
``` 
let json1 = {
  title: "Json.stringify",
  author: [
    "浪里行舟"
  ],
  year: 2021
};
let jsonText = JSON.stringify(json1);
```
序列化之后
``` 
"{"title":"Json.stringify","author":["浪里行舟"],"year":2021}"
```
序列化JavaScript对象时，所有函数和原型成员都会有意地在结果中省略，并且undefined的任何属性都会被忽略。  
JSON.stringify()有三个参数，其中两个可选参数。这两个可选仓鼠可以用于指定其他序列化JavaScript对象的方式。第二个参数是过滤器，可以是数组或函数，第三个参数用于缩进结果JSON字符串的选项。
## JSON过滤器
如果第二个参数是一个数组，那么JSON.stringify()返回的结果只会包含该数组中列出的对象属性
``` 
let json1 = {
  title: "Json.stringify",
  author: [
    "浪里行舟"
  ],
  year: 2021,
  like: 'frontend',
  weixin: 'frontJS'
};
let jsonText = JSON.stringify(json1, ['weixin']);
```
序列化之后
``` 
"{"weixin":"frontJS"}"
```
如果序列化是一个函数，提供的函数接受两个参数，属性名（key）和属性值（value）。可以根据这个key决定对应属性执行什么操作，这个key始终是字符创，只是在值不属于某个键/值对时会是空字符串。
``` 
const students = [
  {
    name: 'james',
    score: 100,
  }, {
    name: 'jordon',
    score: 60,
  }, {
    name: 'kobe',
    score: 90,
  }
];

function replacer (key, value) {
  if (key === 'score') {
    if (value === 100) {
      return 'S';
    } else if (value >= 90) {
      return 'A';
    } else if (value >= 70) {
      return 'B';
    } else if (value >= 50) {
      return 'C';
    } else {
      return 'E';
    }
  }
  return value;
}
console.log(JSON.stringify(students, replacer, 4))
```
序列化后
``` 
[
    {
        "name": "james",
        "score": "S"
    },
    {
        "name": "jordon",
        "score": "C"
    },
    {
        "name": "kobe",
        "score": "A"
    }
]
```
## 字符串缩进
JSON.stringify()方法的第三个参数控制缩进和空格。参数是数值时，表示每一级缩进的空格数
``` 
let json1 = {
  title: "Json.stringify",
  author: [
    "浪里行舟"
  ],
  year: 2021
};
let jsonText = JSON.stringify(json1, null, 4);
```
序列化之后
``` 
{
    "title": "Json.stringify",
    "author": [
        "浪里行舟"
    ],
    "year": 2021
}
```
## toJSON()方法--自定义JSON序列化
``` 
let json1 = {
  title: "Json.stringify",
  author: [
    "浪里行舟"
  ],
  year: 2021,
  like: 'frontend',
  weixin: 'frontJS',
  toJSON: function () {
    return this.author
  }
};
console.log(JSON.stringify(json1)); // ["浪里行舟"]
```
**注意:箭头函数不能用来定义toJSON方法**
## 使用场景
### 判断数组是否包含某对象，或者判断对象是否相等
``` 
//判断数组是否包含某对象
let data = [
    {name:'浪里行舟'},
    {name:'前端工匠'},
    {name:'前端开发'},
    ],
    val = {name:'浪里行舟'};
console.log(JSON.stringify(data).indexOf(JSON.stringify(val)) !== -1);//true
```
### 判断对象是否相等
``` 
// 判断对象是否相等
let obj1 = {
    a: 1,
    b: 2
  }
  let obj2 = {
    a: 1,
    b: 2,
  }
console.log(JSON.stringify(obj1) === JSON.stringify(obj2)) // true
```
如果调整键的顺序，就会判断出错
``` 
// 调整对象键的位置后
let obj1 = {
    a: 1,
    b: 2
  }
  let obj2 = {
    b: 2,
    a: 1,
  }
console.log(JSON.stringify(obj1) === JSON.stringify(obj2)) // false
```
### 使用localStorage/sessionStorage
localStorage/sessionStorage默认只能存储字符串，使用json.stringify将对象转为字符串，获取本地缓存再使用json.parse转为对象
``` 
// 存数据
function setLocalStorage(key,val) {
    window.localStorage.setItem(key, JSON.stringify(val));
};
// 取数据
function getLocalStorage(key) {
    let val = JSON.parse(window.localStorage.getItem(key));
    return val;
};
// 测试
setLocalStorage('Test',['前端工匠','浪里行舟']);
console.log(getLocalStorage('Test'));
```
### 实现对象深拷贝
``` 
let arr1 = [1, 3, {
    username: ' kobe'
}];
let arr2 = JSON.parse(JSON.stringify(arr1));
arr2[2].username = 'duncan'; 
console.log(arr1, arr2)
```
这种方法虽然可以实现数组或对象深拷贝,但不能处理函数和正则，因为这两者基于JSON.stringify和JSON.parse处理后，得到的正则就不再是正则（变为空对象），得到的函数就不再是函数（变为null）了。
## 使用注意事项
1. 被转换值中有 NaN 和 Infinity
``` 
let myObj = {
  name: "浪里行舟",
  age: Infinity,
  money: NaN,
};
console.log(JSON.stringify(myObj));
// {"name":"浪里行舟","age":null,"money":null}

JSON.stringify([NaN, Infinity])
// [null,null]
```
2.被转换值中有 undefined、任意的函数以及 symbol 值
- 数组的undefined、任意的函数以及symbol值在序列化的过程中会被转换成 null
``` 
JSON.stringify([undefined, function () { }, Symbol("")]);
// '[null,null,null]'
```
- 非数组的undefined、任意的函数以及symbol值在序列化的过程中会被忽略
``` 
JSON.stringify({ x: undefined, y: function () { }, z: Symbol("") });
// '{}'
```
3. 循环引用
``` 
let bar = {
  a: {
    c: foo
  }
};
let foo = {
  b: bar
};

JSON.stringify(foo)
```
序列化会报错
```
 // 错误信息
 Uncaught ReferenceError: foo is not defined
     at <anonymous>:3:8
```
4.含有不可枚举的属性值时
不可枚举的属性默认会被忽略
``` 
let personObj = Object.create(null, {
  name: { value: "浪里行舟", enumerable: false },
  year: { value: "2021", enumerable: true },
})

console.log(JSON.stringify(personObj)) // {"year":"2021"}
```

参考:
[你会用JSON.stringify()?
](https://github.com/ljianshu/Blog/issues/97)
