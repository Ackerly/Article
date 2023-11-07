# 对于空数组every()方法返回true
对于空数组，不管回调函数是什么，every () 都返回 true，因为根本不会调用该回调函数。看一下例子：  
``` 
function isNumber(value) {
     return typeof value === "number";
 }

 [1].every(isNumber);            // true
 ["1"].every(isNumber);          // false
 [1, 2, 3].every(isNumber);      // true
 [1, "2", 3].every(isNumber);    // false
 [].every(isNumber);             // true
```
在此示例的每种情况下，均调用 every () 来检查数组中的每一项是否为数字。前四个调用相当简单，每个都会产生预期的结果。考虑如下的例子：  
``` 
[].every(() => true);           // true
[].every(() => false);          // true
```
对于 every()，返回 true 或 false 的回调都具有相同的结果。发生这种情况的唯一原因是调用回调函数没有被调用，并且 every () 的默认返回值为 true。但是，当没有值可以用来运行回调函数时，为什么空数组对 every () 返回 true 呢？  

**实现 every() 方法**  
定义了一个 Array.Prototype.every () 算法，该算法大致可以翻译成这段 JavaScript 代码：   
``` 
Array.prototype.every = function(callbackfn, thisArg) {
   const O = this;
   const len = O.length;
   if (typeof callbackfn !== "function") {
       throw new TypeError("Callback isn't callable");
   }
   let k = 0;
   while (k < len) {
       const Pk = String(k);
       const kPresent = O.hasOwnProperty(Pk);
       if (kPresent) {
           const kValue = O[Pk];
           const testResult = Boolean(callbackfn.call(thisArg, kValue, k, O));
           if (testResult === false) {
               return false;
           }
       }
       k = k + 1;
   }
   return true;
 };
```
every () 假定结果为 true，并且只有在回调函数对数组中的任何一项返回 false 时才返回 false。如果数组中没有元素，那么就没有机会执行回调函数，因此方法就没有办法返回 false。  
**数学和 JavaScript 中的全称量词**  
MDN 提供了为什么 every () 对于空数组返回 true 的答案：  
> every 和数学中的全称量词 "任意（∀）" 类似。特别的，对于空数组，它只返回 true。这种情况属于无条件正确，因为空集的所有元素都符合给定的条件。）

无条件正确是一个数学概念，它意味着如果一个给定的条件 (称为先行条件) 不能被满足 (也就是说，给定的条件是不真实的) ，那么某些东西就是真的。要将其应用到 JavaScript 中，那就是 every () 对于空数组返回 true，因为没有办法调用回调函数 。回调代表了要测试的条件，如果由于数组中没有值而无法执行，那么 every () 必须返回 true。  
全称量词是数学中一个更大的主题的一部分，这个主题被称为 “全称量化”，它允许你对数据集进行推理。考虑到 JavaScript 数组对于执行数学计算的重要性，特别是对于类型化数组，内置支持这种操作是有意义的。要知道，every() 并不是唯一的例子。  

**数学和 JavaScript 中的存在量词**  
JavaScript 的 some() 方法实现了存在量词。“存在” 量词指出，对于任何空集，结果都是 false。因此，some () 方法对于空数组返回 false，并且也不执行回调函数。下面是一些例子：  
``` 
function isNumber(value) {
     return typeof value === "number";
 }
 [1].some(isNumber);            // true
 ["1"].some(isNumber);          // false
 [1, 2, 3].some(isNumber);      // true
 [1, "2", 3].some(isNumber);    // true
 [].some(isNumber);             // false
 [].some(() => true);           // false
 [].some(() => false);          // false
```

**其他语言对量词的实现**  
JavaScript 并不是唯一的一种为集合或可迭代对象实现了量词相关方法的编程语言：  
Python: all () 函数实现全称量词，而 any () 函数实现了存在量词。  
Rust: Iterator: : all () 函数实现了全称量词，而 any () 函数实现了存在量词。  
因此，JavaScript 与 every () 和 some () 都有着良好的合作关系。  

**意味着全称量词的 every()**  
需要了 every() 所具有的全程量词的性质以避免错误。简而言之，如果可能为空的数组使用了 every ()，则应该事先添加一个显式检查。例如，如果您有一个依赖于数字数组的操作，并且该操作将以空数组失败，那么您应该在使用 every () 之前检查该数组是否为空：  
``` 
function doSomethingWithNumbers(numbers) {
     // first check the length
     if (numbers.length === 0) {
         throw new TypeError("Numbers array is empty; this method requires at least one number.");
     }
     // now check with every()
     if (numbers.every(isNumber)) {
         operationRequiringNonEmptyArray(numbers);
     }
 }

```





参考:  
[JavaScript 令人惊讶的一点：对于空数组every()方法返回true](https://mp.weixin.qq.com/s/j1qyT2_KYFNRp3M896MWjA)
