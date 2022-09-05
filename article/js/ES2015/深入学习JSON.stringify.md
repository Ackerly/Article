# 深入学习JSON.stringify
**场景**  
一个动态表单搜集页面，用户选择或者填写了信息之后（各字段非必填情况下也可以直接提交），接着前端把数据发送给后端，结束  
**错误原因**  
> 非必填情况下，signInfo字段中经过JSON.stringify后的字符串对象缺少value key，导致后端parse之后无法正确读取value值，进而报接口系统异常，用户无法进行下一步动作  
```  
// 默认情况下数据是这样的
let signInfo = [
  {
    fieldId: 539,
    value: undefined
  },
  {
    fieldId: 540,
    value: undefined
  },
  {
    fieldId: 546,
    value: undefined
  },
]
// 经过JSON.stringify之后的数据,少了value key,导致后端无法读取value值进行报错
// 具体原因是`undefined`、`任意的函数`以及`symbol值`，出现在`非数组对象`的属性值中时在序列化过程中会被忽略
console.log(JSON.stringify(signInfo))
// '[{"fieldId":539},{"fieldId":540},{"fieldId":546}]'
```
**解决方案**  
1.新开一个对象处理  
``` 
let signInfo = [
  {
    fieldId: 539,
    value: undefined
  },
  {
    fieldId: 540,
    value: undefined
  },
  {
    fieldId: 546,
    value: undefined
  },
]

let newSignInfo = signInfo.map((it) => {
  const value = typeof it.value === 'undefined' ? '' : it.value
  return {
    ...it,
    value
  }
})

console.log(JSON.stringify(newSignInfo))
// '[{"fieldId":539,"value":""},{"fieldId":540,"value":""},{"fieldId":546,"value":""}]'
```
2. 利用JSON.stringify第二个参数，直接处理
``` 
let signInfo = [
  {
    fieldId: 539,
    value: undefined
  },
  {
    fieldId: 540,
    value: undefined
  },
  {
    fieldId: 546,
    value: undefined
  },
]

// 判断到value为undefined，返回空字符串即可
JSON.stringify(signInfo, (key, value) => typeof value === 'undefined' ? '' : value)
// '[{"fieldId":539,"value":""},{"fieldId":540,"value":""},{"fieldId":546,"value":""}]'
```
## 基本使用  
参数
- value  
将要序列化成 一个 JSON 字符串的值
- replacer 可选  
    1. 如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；
    2. 如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中；
    3. 如果该参数为 null 或者未提供，则对象所有的属性都会被序列化
- space 可选
    1. 指定缩进用的空白字符串，用于美化输出（pretty-print）
    2. 如果参数是个数字，它代表有多少的空格；上限为10
    3.该值若小于1，则意味着没有空格；
    4. 如果该参数为字符串（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格；
    5. 如果该参数没有提供（或者为 null），将没有空格。

**异常**  
- 在循环引用时会抛出异常TypeError ("cyclic object value")（循环对象值）
- 尝试去转换 BigInt 类型的值会抛出TypeError ("BigInt value can't be serialized in JSON")（BigInt值不能JSON序列化）

**注意**  
- JSON.stringify可以转换对象或者值（平常用的更多的是转换对象）
- 可以指定replacer为函数选择性的地替换
- 可以指定replacer为数组，可转换指定的属性
``` 
// 1. 转换对象
console.log(JSON.stringify({ name: '前端胖头鱼', sex: 'boy' })) // '{"name":"前端胖头鱼","sex":"boy"}'

// 2. 转换普通值
console.log(JSON.stringify('前端胖头鱼')) // "前端胖头鱼"
console.log(JSON.stringify(1)) // "1"
console.log(JSON.stringify(true)) // "true"
console.log(JSON.stringify(null)) // "null"

// 3. 指定replacer函数
console.log(JSON.stringify({ name: '前端胖头鱼', sex: 'boy', age: 100 }, (key, value) => {
  return typeof value === 'number' ? undefined : value
}))
// '{"name":"前端胖头鱼","sex":"boy"}'

// 4. 指定数组
console.log(JSON.stringify({ name: '前端胖头鱼', sex: 'boy', age: 100 }, [ 'name' ]))
// '{"name":"前端胖头鱼"}'

// 5. 指定space(美化输出)
console.log(JSON.stringify({ name: '前端胖头鱼', sex: 'boy', age: 100 }))
// '{"name":"前端胖头鱼","sex":"boy","age":100}'
console.log(JSON.stringify({ name: '前端胖头鱼', sex: 'boy', age: 100 }, null , 2))
/*
{
  "name": "前端胖头鱼",
  "sex": "boy",
  "age": 100
}
*/
```
## 9大特性  
### 特性一
1. undefined、任意的函数以及symbol值，出现在非数组对象的属性值中时在序列化过程中会被忽略
2. undefined、任意的函数以及symbol值出现在数组中时会被转换成 null
3. undefined、任意的函数以及symbol值被单独转换时，会返回 undefined

``` 
// 1. 对象中存在这三种值会被忽略
console.log(JSON.stringify({
  name: '前端胖头鱼',
  sex: 'boy',
  // 函数会被忽略
  showName () {
    console.log('前端胖头鱼')
  },
  // undefined会被忽略
  age: undefined,
  // Symbol会被忽略
  symbolName: Symbol('前端胖头鱼')
}))
// '{"name":"前端胖头鱼","sex":"boy"}'

// 2. 数组中存在着三种值会被转化为null
console.log(JSON.stringify([
  '前端胖头鱼',
  'boy',
  // 函数会被转化为null
  function showName () {
    console.log('前端胖头鱼')
  },
  //undefined会被转化为null
  undefined,
  //Symbol会被转化为null
  Symbol('前端胖头鱼')
]))
// '["前端胖头鱼","boy",null,null,null]'

// 3.单独转换会返回undefined
console.log(JSON.stringify(
  function showName () {
    console.log('前端胖头鱼')
  }
)) // undefined
console.log(JSON.stringify(undefined)) // undefined
console.log(JSON.stringify(Symbol('前端胖头鱼'))) // undefined
```
### 特性二  
1. 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
``` 
console.log(JSON.stringify([new Number(1), new String("前端胖头鱼"), new Boolean(false)]))
// '[1,"前端胖头鱼",false]'
```
### 特性三  
所有以symbol为属性键的属性都会被完全忽略掉，即便 replacer 参数中强制指定包含了它们。  
``` 
console.log(JSON.stringify({
  [Symbol('前端胖头鱼')]: '前端胖头鱼'}
)) 
// '{}'
console.log(JSON.stringify({
  [ Symbol('前端胖头鱼') ]: '前端胖头鱼',
}, (key, value) => {
  if (typeof key === 'symbol') {
    return value
  }
}))
// undefined
```
### 特性四
NaN 和 Infinity 格式的数值及 null 都会被当做 null。
``` 
console.log(JSON.stringify({
  age: NaN,
  age2: Infinity,
  name: null
}))
// '{"age":null,"age2":null,"name":null}'
```
### 特性五
转换值如果有 toJSON() 方法，该方法定义什么值将被序列化。
``` 
const toJSONObj = {
  name: '前端胖头鱼',
  toJSON () {
    return 'JSON.stringify'
  }
}

console.log(JSON.stringify(toJSONObj))
// "JSON.stringify"
```
### 特性六
Date 日期调用了 toJSON() 将其转换为了 string 字符串（同Date.toISOString()），因此会被当做字符串处理
``` 
const d = new Date()

console.log(d.toJSON()) // 2021-10-05T14:01:23.932Z
console.log(JSON.stringify(d)) // "2021-10-05T14:01:23.932Z"
```
### 特性七  
对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。  
``` 
let cyclicObj = {
  name: '前端胖头鱼',
}

cyclicObj.obj = cyclicObj

console.log(JSON.stringify(cyclicObj))
// Converting circular structure to JSON
```
### 特性八
其他类型的对象，包括 Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性  
``` 
let enumerableObj = {}

Object.defineProperties(enumerableObj, {
  name: {
    value: '前端胖头鱼',
    enumerable: true
  },
  sex: {
    value: 'boy',
    enumerable: false
  },
})

console.log(JSON.stringify(enumerableObj))
// '{"name":"前端胖头鱼"}'
```
### 特性九  
``` 
const alsoHuge = BigInt(9007199254740991)

console.log(JSON.stringify(alsoHuge))
// TypeError: Do not know how to serialize a BigInt
```
## 手写一个JSON.stringify
``` 
// 1. 测试一下基本输出
console.log(jsonstringify(undefined)) // undefined 
console.log(jsonstringify(() => { })) // undefined
console.log(jsonstringify(Symbol('前端胖头鱼'))) // undefined
console.log(jsonstringify((NaN))) // null
console.log(jsonstringify((Infinity))) // null
console.log(jsonstringify((null))) // null
console.log(jsonstringify({
  name: '前端胖头鱼',
  toJSON() {
    return {
      name: '前端胖头鱼2',
      sex: 'boy'
    }
  }
}))
// {"name":"前端胖头鱼2","sex":"boy"}

// 2. 和原生的JSON.stringify转换进行比较
console.log(jsonstringify(null) === JSON.stringify(null));
// true
console.log(jsonstringify(undefined) === JSON.stringify(undefined));
// true
console.log(jsonstringify(false) === JSON.stringify(false));
// true
console.log(jsonstringify(NaN) === JSON.stringify(NaN));
// true
console.log(jsonstringify(Infinity) === JSON.stringify(Infinity));
// true
let str = "前端胖头鱼";
console.log(jsonstringify(str) === JSON.stringify(str));
// true
let reg = new RegExp("\w");
console.log(jsonstringify(reg) === JSON.stringify(reg));
// true
let date = new Date();
console.log(jsonstringify(date) === JSON.stringify(date));
// true
let sym = Symbol('前端胖头鱼');
console.log(jsonstringify(sym) === JSON.stringify(sym));
// true
let array = [1, 2, 3];
console.log(jsonstringify(array) === JSON.stringify(array));
// true
let obj = {
  name: '前端胖头鱼',
  age: 18,
  attr: ['coding', 123],
  date: new Date(),
  uni: Symbol(2),
  sayHi: function () {
    console.log("hello world")
  },
  info: {
    age: 16,
    intro: {
      money: undefined,
      job: null
    }
  },
  pakingObj: {
    boolean: new Boolean(false),
    string: new String('前端胖头鱼'),
    number: new Number(1),
  }
}
console.log(jsonstringify(obj) === JSON.stringify(obj)) 
// true
console.log((jsonstringify(obj)))
// {"name":"前端胖头鱼","age":18,"attr":["coding",123],"date":"2021-10-06T14:59:58.306Z","info":{"age":16,"intro":{"job":null}},"pakingObj":{"boolean":false,"string":"前端胖头鱼","number":1}}
console.log(JSON.stringify(obj))
// {"name":"前端胖头鱼","age":18,"attr":["coding",123],"date":"2021-10-06T14:59:58.306Z","info":{"age":16,"intro":{"job":null}},"pakingObj":{"boolean":false,"string":"前端胖头鱼","number":1}}

// 3. 测试可遍历对象
let enumerableObj = {}

Object.defineProperties(enumerableObj, {
  name: {
    value: '前端胖头鱼',
    enumerable: true
  },
  sex: {
    value: 'boy',
    enumerable: false
  },
})

console.log(jsonstringify(enumerableObj))
// {"name":"前端胖头鱼"}

// 4. 测试循环引用和Bigint

let obj1 = { a: 'aa' }
let obj2 = { name: '前端胖头鱼', a: obj1, b: obj1 }
obj2.obj = obj2

console.log(jsonstringify(obj2))
// TypeError: Converting circular structure to JSON
console.log(jsonStringify(BigInt(1)))
// TypeError: Do not know how to serialize a BigInt
```

原文:  
[就因为JSON.stringify，我的年终奖差点打水漂了](https://juejin.cn/post/7017588385615200270)
