# 使用 JavaScript 实现数字的国际化
以可读的格式显示数字有多种形式，从可视化图表到简单地添加标点符号。但是，这些标点符号在国际化的基础上是不同的。一些国家用 , 表示十进制，而另一些国家用.。  
基本类型 Number 有一个 toLocaleString 方法来为你做基本的格式化:  
``` 
const price = 16601.91;

 // Basic decimal format, no providing locale
 // Uses locale provided by browser since none defined
 price.toLocaleString(); // "16,601.91"

 // Provide a specific locale
 price.toLocaleString('de-DE'); // "16.601,91"

 // Formatting currency is possible
 price.toLocaleString('de-DE', {
   style: 'currency',
   currency: 'EUR'
 }); // "16.601,91 €"

 // You can also use Intl.NumberFormat for formatting
 new Intl.NumberFormat('en-US', {
   style: 'currency',
   currency: 'GBP'
 }).format(price); // £16,601.91
```

原文:  
[如何使用 JavaScript 实现数字的国际化](https://mp.weixin.qq.com/s/O3Nxtc57qH3GMX5covCgaA)
