# Sorting in JavaScript
JavaScript 数组有一个内置的 sort 方法，它基本上可以完成所期望的：  
``` 
[1, 4, 2, 3].sort(); // [1, 2, 3, 4]
 ["d", "a", "b", "c"].sort(); // ['a', 'b', 'c', 'd']
```
例如，数字排序就像它们是字符串一样，只要所有数字的长度相同就没问题，但对于不同长度的数字，就会出现意想不到的结果。  
``` 
[2, 1, 10].sort(); // [1, 10, 2] Not what we wanted!
```
字符串的默认排序通常是可以的，但在非 ASCII 字符的情况下可能会出现意外，例如带有重音的字母：  
``` 
["ä", "c", "b"].sort(); // ['b', 'c', 'ä' ] Maybe not what we wanted?
```
排序是在原数组进行的。这意味着原始数组被修改。请确保对一个数组进行排序时，该修改不会对其他地方产生任何副作用。  
**基本排序比较器**   
将回调作为参数传递给 sort 方法。对于数字，可以用减法来得到正确的数字排序：  
``` 
[2, 1, 10].sort((a, b) => a - b); // [1, 2, 10] That's better!
```
通常 JavaScript 不知道你希望你的数组如何排序。你可以将比较回调看作是对 "给定数组中的任何两个项，它们应该如何按照你想要的顺序相互比较？" 的回答。从你的回调中返回的值必须是一个数字，其中。  
- 如果 a 应该在 b 之前，那么返回一个负值
- 如果 a 在 b 之后，则返回一个正值
- 如果 a 和 b 的顺序相同，则返回 0。

对于字符串，可以使用 String.prototype.localeCompare  
``` 
["ä", "b", "c"].sort((a, b) => a.localeCompare(b)); // ['ä', 'b', 'c']
```
此外，你可以传递一个特定的区域设置，因为某些区域的排序方式与其他地区不同：  
``` 
// German
 ["ä", "b"].sort((a, b) => a.localeCompare(b, "de")); // ['ä', 'b', 'c']

 // Swedish
 ["ä", "b"].sort((a, b) => a.localeCompare(b, "sv")); // ['b', 'c', 'ä']
```
**按属性对对象进行排序**  
可以对数字和字符串进行排序...... 但是对象呢？对于对象数组，我们通常需要使用对象的某些属性作为比较的一部分。例如，如果我们有一些图书的数组，我们可能想按出版日期或标题来排序  
``` 
const sortedByDate = books.sort((a, b) => a.published - b.published);
 const sortedByTitle = books.sort((a, b) => a.title.localeCompare(b.title));
```
实际上并排进行这两种排序会导致第二种排序覆盖第一种排序，因为排序是按照前面提到的那样进行的。  
可以通过复制数组来解决这个问题  
``` 
const sortedByDate = [...books].sort((a, b) => a.published - b.published);
 const sortedByTitle = [...books].sort((a, b) => a.title.localeCompare(b.title));
```

**按属性对对象进行排序**  
可以对数字和字符串进行排序...... 但是对象呢？对于对象数组，我们通常需要使用对象的某些属性作为比较的一部分。例如，如果我们有一些图书的数组，我们可能想按出版日期或标题来排序。  
``` 
const sortedByDate = books.sort((a, b) => a.published - b.published);
 const sortedByTitle = books.sort((a, b) => a.title.localeCompare(b.title));
```
实际上并排进行这两种排序会导致第二种排序覆盖第一种排序，因为排序是按照前面提到的那样进行的。  
可以通过复制数组来解决这个问题  
``` 
const sortedByDate = [...books].sort((a, b) => a.published - b.published);
 const sortedByTitle = [...books].sort((a, b) => a.title.localeCompare(b.title));
```
**按升序或降序排序**  
在升序或降序之间进行切换，就像在我们的比较中颠倒 a 和 b 的顺序一样简单。举例来说。  
``` 
[2, 1, 10].sort((a, b) => a - b); // [1, 2, 10] ascending order
 [2, 1, 10].sort((a, b) => b - a); // [10, 2, 1] descending order

 books.sort((a, b) => a.published - b.published); // ascending by publication date
 books.sort((a, b) => b.published - a.published); // descending by publication date
```
**按升序或降序排序**  
在升序或降序之间进行切换，就像在我们的比较中颠倒 a 和 b 的顺序一样简单。举例来说。  
``` 
[2, 1, 10].sort((a, b) => a - b); // [1, 2, 10] ascending order
 [2, 1, 10].sort((a, b) => b - a); // [10, 2, 1] descending order

 books.sort((a, b) => a.published - b.published); // ascending by publication date
 books.sort((a, b) => b.published - a.published); // descending by publication date
```
**按多个属性进行排序**  
通常情况下，只按一个属性进行排序是不够的。我们可能需要为那些本来会被 "平等" 排序的项提供一个分界点。例如，考虑到我们的图书收藏，我们想按作者排序，但同一作者的多本书怎么办？我认为使用标题作为分界线是明智的。这样，如果你要找某位作者的某本书，你可以先按作者然后按书名轻松找到。  
一种方法是依靠 ||（OR）运算符来做到这一点。  
``` 
const sortedBooks = books.sort((a, b) => {
   return a.author.localeCompare(b.author) || a.title.localeCompare(b.title);
 });
```
这通过首先检查作者的名字来实现。如果它们不同，那么这个值将被立即使用。但是如果他们是相同的，那么这个值将是 0，由于 0 是 false 的，那么表达式的右半部分将被使用。所以总的来说，书籍将按作者排序，但在一个特定的作者中，书籍将按标题排序。  
这种模式对于单一的比较来说是完美的，甚至可以扩展到支持更多的比较。  
``` 
 // Compare by author first, then by title, and finally by edition number
 const sortedBooks = books.sort((a, b) => {
   return (
     a.author.localeCompare(b.author) ||
     a.title.localeCompare(b.title) ||
     a.edition - b.edition
   );
 });
```
把它抽象成一个可重复使用的工具，它可以接受一个比较器的集合，一个接一个地使用它们，直到找到一个非零的结果。  
``` 
const multiSort =
   <Item>(...comparators: Array<(a: Item, b: Item) => number>) =>
   (a: Item, b: Item) => {
     // Try each comparator in turn
     for (let comparator of comparators) {
       // Get its result
       const comparatorResult = comparator(a, b);
       // Return that result only if it is non-zero
       if (comparatorResult !== 0) return comparatorResult;
     }
     // All comparators returned zero, so these items cannot be distinguished
     return 0;
   };
```
使用  
``` 
const sortedBooks = books.sort(
   multiSort(
     (a, b) => a.title.localeCompare(b.title),
     (a, b) => a.published - b.published,
   )
 );
```
这使得重新排列或添加和删除比较能变得快速而容易。  
**出错的方法**  
``` 
// Sort users by name, but put all nameless users at the end.
 users.sort((a, b) => {
   // If a doesn't have a name...
   if (!a.name) return 1; // ...then a should go after b
   // If b doesn't have a name...
   if (!b.name) return -1; // ...then b should go after a
   // Otherwise compare by name
   return a.name.localeCompare(b.name);
 });
```
当 a 和 b 都没有名字时会发生什么？那么这个比较器声明 a 应该在 b 之前，但这是不正确的，它们应该被视为相等。一般来说，你应该避免检查其中一个值而不是另一个值的情况。从形式上讲，这个比较方法破坏了总序的反对称属性。这是用一种华丽的方式说 "它是坏的"。  
总排序的数学概念是定义一个排序是否一致的东西。它由四条规则组成。  
- a <= a (反身性)
- 如果 a<=b，b<=c，那么 a<=c（传递性的）
- 如果 a<=b，b<=a，那么 a=b（反对称性）。
- a <= b 或 b <= a (强连接)

这是一种定义集合中每一对之间关系的奇特方式。在 JavaScript 中处理排序时，我们已经免费得到了规则 1 和规则 4，只要你总是从比较器中返回任何东西。但是规则 2 和规则 3 可能会被违反。  
违反传递规则有点困难，但在极少数情况下会发生。举个例子，你可以考虑在 “石头、剪刀、布” 中选择 “最好” 的招式。你可以试着通过直接比较哪个值胜过其他值来做到这一点:  
``` 
const sortByWinner = (a, b) => {
   // If a beats b, then a should go first
   if (a.beats(b)) return -1;
   // If b beats a, then b should go first
   if (b.beats(a)) return 1;
   // Neither beats the other, so these two are equal
   return 0;
 };
```
尽管这段代码中没有明显的错误，但它还是会导致排序不一致。不幸的是，没有办法解决这个问题。从根本上说，我们试图排序的属性是不可传递的，这与其说是代码中的错误，不如说是我们在尝试排序时所做的基本假设。它不是我们可以用来排序的属性。  

**如何正确处理**  
上面的例子是如何通过优先处理一个属性而出错的，但是我们如何正确地做到这一点呢？这是一个相当棘手的问题，因为 a 或 b 可能没有名字，所以我们需要优雅地处理只有在它们都有名字的情况下才用字符串进行比较。如果其中一个没有名字，那么我们就需要小心地考虑 a 和 b，并仅仅根据它们是否有名字来进行比较。  
``` 
// Sort users by name, but nameless users should go at the end
 users.sort((a, b) => {
   // Both users have a name, so compare directly
   if (a.name && b.name) return a.name.localeCompare(b.name);

   // Otherwise we sort by having a name or not
   return !!b.name - !!a.name;
   // b goes first because we want names first, non-names second
 });
```
一般来说，为了持续编写正确的比较器，我们应该尝试遵循这些一般规则。  
- 始终平等对待 a 和 b
- 对数字使用减法
- 对字符串使用 localeCompare
- 在排序前检查是否可以修改数组
- 在少数情况下，一个属性与排序的概念在逻辑上是不相容的

原文:  
[Sorting in JavaScript](https://mp.weixin.qq.com/s/MRTFmVOc27k3W6x8uWJ0AQ)
