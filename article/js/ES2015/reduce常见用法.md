# reduce常见用法
**语法**  
``` 
let value = arr.reduce(function(previousValue, item, index, array) {
  // ...
}, [initial]);
```
参数：  
- previousValue: 上一个函数调用的结果，第一次等于 initial（如果提供了 initial 的话）  
- item : 当前的数组元素
- index :当前索引
- arr : 数组本身

## 应用
**经典累加器**   
``` 
const a = [1, 2, 8, 7, 4];

const sum = a.reduce((pre, cur) => {
 const res = pre + cur;
 return res;
}, 0);

console.log(sum); // 22
```
**二维数组转一维**  
``` 
const arr2 = [
 [1, 2],
 [3, 4],
 [5, 6],
].reduce((acc, cur) => {
 return acc.concat(cur);
}, []);

console.log(arr2); //
```
**多维数组扁平**  
``` 
const arr3 = [
 [1, 2],
 [3, 4],
 [5, [7, [9, 10], 8], 6],
];
const flatten = arr =>
 arr.reduce(
  (pre, cur) => pre.concat(Array.isArray(cur) ? flatten(cur) : cur),
  []
 );
console.log(flatten(arr3)); // [ 1, 2, 3, 4, 5, 7, 9, 10, 8, 6 ]
```
**数组分块**  
根据传入限制大小，对数组进行分块。小于限制长度时就往里添加，否则直接将其加入res  
``` 
const chunk = (arr, size) => {
 return arr.reduce(
  (res, cur) => (
   res[res.length - 1].length < size
    ? res[res.length - 1].push(cur)
    : res.push([cur]),
   res
  ),
  [[]]
 );
};
const arr4 = [1, 2, 3, 4, 5, 6];
console.log(chunk(arr4, 3)); // [ [ 1, 2, 3 ], [ 4, 5, 6 ] ]
```
**字符统计**  
``` 
const countChar = text => {
 text = text.split("");
 return text.reduce((record, c) => {
  record[c] = (record[c] || 0) + 1;
  return record;
 }, {});
};
const text = "划水水摸鱼鱼";
console.log(countChar(text)); // { '划': 1, '水': 2, '摸': 1, '鱼': 2 }
```

原文: 
[比你想象中更强大的 reduce 以及对敲码的思考](https://mp.weixin.qq.com/s/qMNRMU3yqFoCzuauTvspqw)
