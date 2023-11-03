# 支持原生数组分组了
## 以前的数组分组
假设有一个由表示人员的对象组成的数组，需要按照年龄进行分组。可以使用 forEach 循环来实现，代码如下：  
``` 
const people = [
   { name: "Alice", age: 28 },
   { name: "Bob", age: 30 },
   { name: "Eve", age: 28 },
 ];

 const peopleByAge = {};

 people.forEach((person) => {
   const age = person.age;
   if (!peopleByAge[age]) {
     peopleByAge[age] = [];
   }
   peopleByAge[age].push(person);
 });

 console.log(peopleByAge);
```
输出结果如下：  
``` 
{
   "28": [{"name":"Alice","age":28}, {"name":"Eve","age":28}],
   "30": [{"name":"Bob","age":30}]
 }
```
也可以使用 reduce 方法：  
``` 
const peopleByAge = people.reduce((acc, person) => {
   const age = person.age;
   if (!acc[age]) {
     acc[age] = [];
   }
   acc[age].push(person);
   return acc;
 }, {});
```
无论哪种方式，代码都略显繁琐。每次都要检查对象，看分组键是否存在，如果不存在，则创建一个空数组，并将项目添加到该数组中。  
## 使用 Object.groupBy 分组
通过以下方式来使用新的 Object.groupBy 方法：  
``` 
 const peopleByAge = Object.groupBy(people, (person) => person.age);
```
使用 Object.groupBy 方法返回一个没有原型（即没有继承任何属性和方法）的对象。这意味着该对象不会继承 Object.prototype 上的任何属性或方法，例如 hasOwnProperty 或 toString 等。虽然这样做可以避免意外覆盖 Object.prototype 上的属性，但也意味着不能使用一些与对象相关的方法。  
``` 
onst peopleByAge = Object.groupBy(people, (person) => person.age);
 console.log(peopleByAge.hasOwnProperty("28"));
 // TypeError: peopleByAge.hasOwnProperty is not a function
```
在调用 Object.groupBy 时，传递给它的回调函数应该返回一个字符串或 Symbol 类型的值。如果回调函数返回其他类型的值，它将被强制转换为字符串。  
在这个例子中，回调函数返回的是一个数字类型的 age 属性值，但由于 Object.groupBy 方法要求键必须是字符串或 Symbol 类型，所以该数字会被强制转换为字符串类型。  
``` 
console.log(peopleByAge[28]);
 // => [{"name":"Alice","age":28}, {"name":"Eve","age":28}]
 console.log(peopleByAge["28"]);
 // => [{"name":"Alice","age":28}, {"name":"Eve","age":28}]
```

## 使用 Map.groupBy 分组
Map.groupBy 和 Object.groupBy 几乎做的是相同的事情，只是返回的结果类型不同。Map.groupBy 返回一个 Map 对象，而不是像 Object.groupBy 返回一个普通的对象。  
``` 
const ceo = { name: "Jamie", age: 40, reportsTo: null };
 const manager = { name: "Alice", age: 28, reportsTo: ceo };

 const people = [
   ceo
   manager,
   { name: "Bob", age: 30, reportsTo: manager },
   { name: "Eve", age: 28, reportsTo: ceo },
 ];

 const peopleByManager = Map.groupBy(people, (person) => person.reportsTo);
```
如果想通过对象来从这个 Map 中获取数据，那么要求这些对象具有相同的身份或引用。这是因为 Map 在比较键时使用的是严格相等（===），只有两个对象具有相同的引用，才能被认为是相同的键。  
``` 
peopleByManager.get(ceo);
 // => [{ name: "Alice", age: 28, reportsTo: ceo }, { name: "Eve", age: 28, reportsTo: ceo }]
 peopleByManager.get({ name: "Jamie", age: 40, reportsTo: null });
 // => undefined
```
在上面的例子中，如果尝试使用与 ceo 对象类似的对象作为键去访问 Map 中的项，由于这个对象与之前存储在 Map 中的 ceo 对象不是同一个对象，所以无法检索到对应的值。  

## 浏览器支持
这两个 groupBy 方法是 proposal-array-grouping 提案提出的，该提案目前处于第 3 阶段，预计会在 2024 年成为正式标准。  
9 月 12 日，Chrome 117 发布，该版本支持了这两个方法。Firefox Nightly 在一个标志后已经实现了这两个方法。Safari 已经以不同的名称实现了这些方法。由于这些方法在 Chrome 中可用，这意味着它们已经在 V8 中被实现，所以下一次 V8 更新时它们将在 Node 中可用。  

原文:  
[JavaScript 支持原生数组分组了](https://mp.weixin.qq.com/s/Ylc_oHdWEhqGIMlW2R5g2g)
