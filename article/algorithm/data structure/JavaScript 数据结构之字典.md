# JavaScript 数据结构 — 字典
**什么是字典**  
字典中，存储的方式是键值对，也就是 key->value 的形式，字典只要求 key 必须唯一，value 则没有限制。  
这里 key 的作用是唯一标识，用来查询对应的 value 值。也就是说可以通过唯一的 key 映射到对应的 value。所以字典也称作映射，符号表或关联数组。  
在计算机世界中，字典经常用来标识对象的引用地址。比如在 JavaScript 当中的引用类型数据，变量名会指向数据的引用，这是一对映射关系。变量名不能重复，但是不同的变量名可以指向同一块引用  

与 Set 类似，JavaScript ES6 中同样包含了一个 Map 类，即我们所说的字典。  
**创建字典类**  
参照 ES6 Map 类的实现，自己动手实现一个 Dictionary 类。  
```
class Dictionary {
  constructor() {
    this.table = {}
  }
}
```
与前面的其他数据结构实现类似，在一个对象 table 中存储所有字典的元素。保存形式为：table[key] = {key, value}。  
在字典中，通常是用字符串作为键名（key），数据值可以是任意类型。但是 JavaScript 并不是强类型的语言，无法保证传入的键名一定是字符串。所以我们需要将键名做一次字符串的转化。  
写一个默认的转换字符串函数：  
```
function keyToString(item) {
  if(item === null) {
    return 'NULL'
  }
  if(item === undefined) {
    return 'UNDEFINED'
  }
  if(item instanceof String) {
    return `${item}`
  }
  return item.toString()
}
```
除此之外，还有必要将键值对的数据格式封装成一个单独的类。因为key 是不固定的，然而在后面的方法中要频繁使用 key，此时你不知道键名具体是什么。所以封装一个 ValuePair 类会让后续的使用更方便：
```
class ValuePair {
  constructor(key, value) {
    this.key = key;
    this.value = value;
  }
}
```
接下来在类中声明一些必要的方法如下：  
- set：向字典中添加新元素
- remove：以键名为参数，移除字典中对应的键值
- hasKey：检测某个键名是否存在于字典中，存在则返回 true
- get：用键名查找对应的键值并返回
- clear：清空字典
- size：返回字典所包含键的数量
- isEmpty：在 size 等于零时返回 true
- keys：返回字典中所有键名组成的数组
- values：返回字典中所有键值组成的数组
- keyValues：返回所有键值对
- forEach：迭代所有的键值对

**hasKey 方法**  
作用是检测一个键是否在字典中。因为这个方法会在添加和删除元素时使用，所以先实现：  
```
hasKey(key) {
  return this.table[keyToString(key)] != null
}
```
对传入的键进行字符串转换，然后判断键值是不是 null 或者 undefined  
**set 方法**  
```
set(key, value) {
  if(key != null && value != null) {
    let table_key = keyToString(key)
    this.table[table_key] = new ValuePair(key, value)
    return true
  }
  return false
}
```
**remove 方法**  
```
remove(key) {
  if(this.hasKey(key)) {
    delete this.table[keyToString(key)]
    return true
  }
  return false
}
```
**get 方法**  
```
get(key) {
  if(this.hasKey(key)) {
    let table_key = keyToString(key)
    return this.table[table_key].value
  }
  return undefined
}
```
**keys, values, keyValues 方法**  
```
keyValues() {
  return Object.values(this.table)
}
keys() {
  return this.keyValues().map(valuePair=> valuePair.key)
}
values() {
  return this.keyValues().map(valuePair=> valuePair.value)
}
```
首先 keyValues 方法会以数组的形式返回字典的所有键值，返回结果是一个 ValuePair 实例的数组。然后在这个函数的基础上，再分别获取对应的 key 数组和 value 数组。  

**forEach 方法**  
forEach 方法与数组的 forEach 方法功能一致，就是迭代所有元素  
```
forEach(callFn) {
  let valuePairs = this.keyValues()
  for(let i = 0; i < valuePairs.length; i++) {
    let result = callFn(valuePairs[i].key, valuePairs[i].value)
    if(result === false) break;
  }
}
```
首先传一个回调函数作为参数，然后遍历字典的长度，并在循环里调用这个回调函数。这里的一个设计是，如果在回调函数内返回 false，则会中断循环。  

**clear, size, isEmpty 方法**  
```
size() {
  return Object.keys(this.table).length;
}
isEmpty() {
  return this.size() === 0
}
clear() {
  this.table = {}
}
```

**使用字典**  
```
var dict = new Dictionary();
dict.set("name", "赛罗");
dict.set("color", "红蓝");
dict.set("skill", "头标");
```
添加了三个键值对，看一下基本方法的返回结果：  
```
console.log(dict.keys()); // ['name', 'color', 'skill']
console.log(dict.values()); // ['赛罗', '红蓝', '头标']
console.log(dict.size()); // 3

console.log(dict.hasKey("color")); // true
console.log(dict.get("color")); // 红蓝

console.log(dict.hasKey("like")); // false
console.log(dict.get("like")); // undefined
```
再来一波遍历：  
```
dict.forEach((key, value) => {
  console.log(key, value);
  if (key === "color") return false;
});
// 打印结果：
// name 赛罗
// color 红蓝
```
可见循环遍历是没有问题的，而且当函数执行返回 false 时，则会终止遍历，因此第三个键值对没有打印出来，结果达标。  
最后再看一下删除：  
```
// 删除键值对
console.log(dict.remove("color")); // true
console.log(dict.remove("like")); // false
console.log(dict.remove("skill")); // true
console.log(dict.keyValues());  // [ValuePair]
console.log(dict.hasKey("color")); false
console.log(dict.size()); 1
// 清空字典
dict.clear();
console.log(dict.keyValues()); // []
console.log(dict.isEmpty()); // true
```

原文:  
[怒肝 JavaScript 数据结构 — 字典篇](https://mp.weixin.qq.com/s/_B1V1njmnfU4buS_PS1uLg)