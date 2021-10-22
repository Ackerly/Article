# ES6的Map
JS中的对象以键值对的形式存放数据，它的键可以是字符串类型或者symbol类型。ES新的数据结构Map的key可以是任何数据类型。
## 基本语法
初始化 Map 直接使用一个二维数组
``` 
const newMap = new Map([['name', 'Tom'], ['age', 20], ['root', 'china']]);
```
Map 构造函数接受数组作为参数，实际上执行的是下面的算法：
``` 
const items = [
  ['name', 'Tom'],
  ['age', 20],
  ['root', 'china']
];

const map = new Map();

items.forEach(
  ([key, value]) => map.set(key, value)
);
```
**注意事项**
如果对同一个键多次赋值，后面的值将覆盖前面的值
``` 
const map = new Map();

map
.set(1, 'aaa')
.set(1, 'bbb');

map.get(1) // "bbb"
```
只有对同一个对象的引用，Map 结构才将其视为同一个键
``` 
const map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined  
```
Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。
## Map常用的方法和属性
### size属性
size 属性返回 Map 结构的成员总数。
``` 
const newMap = new Map([['name', 'Tom'], ['age', 20], ['root', 'china']]);
newMap.size;  // 3
```
### set
set 方法设置键名key 键的值为 value，然后返回整个 Map 结构。如果 key 已经有值，则键值会被更新，否则就新生成该键。set方法返回的是当前的 Map对象，因此可以采用链式写法
``` 
let newMap = new Map().set(1, 'a').set(2, 'b').set(3, 'c');
// Map(3) {1 => 'a', 2 => 'b', 3 => 'c'}
```
### get
get 方法读取 key 对应键的值，如果找不到key，返回undefined。
``` 
let newMap = new Map().set(1, 'a').set(2, 'b').set(3, 'c');
newMap.get(1);  // 'a'
newMap.get(4);  // undefined
```
### has
某个键是否在当前 Map 对象之中，返回一个Boolean值
``` 
let newMap = new Map().set(1, 'a').set(2, 'b').set(3, 'c');
newMap.has(1);  // true
newMap.has(4);  // false
```
### delete
删除某个键，返回true。如果删除失败，返回false。
``` 
let newMap = new Map().set(1, 'a').set(2, 'b').set(3, 'c');
newMap.delete(1); // true
newMap.has(1);  // false
newMap.delete(1); // false
```
### clear
清除所有成员，无返回值。
``` 
let newMap = new Map().set(1, 'a').set(2, 'b').set(3, 'c');
newMap.clear();

newMap.size;  // 0
```

参考:
[我竟然有点忘记了 ES6 的Map](https://juejin.cn/post/7020669015819288590)
