# 33个实用JavaScript一行代码
## 日期处理
**检查日期是否有效**  
```
const isDateValid = (...val) => !Number.isNaN(new Date(...val).valueOf());

isDateValid("December 17, 1995 03:24:00");  // true
```
**计算两个日期之间的间隔**  
```
const dayDif = (date1, date2) => Math.ceil(Math.abs(date1.getTime() - date2.getTime()) / 86400000)
    
dayDif(new Date("2021-11-3"), new Date("2022-2-1"))  // 90
```
**查找日期位于一年中的第几天**
```
const dayOfYear = (date) => Math.floor((date - new Date(date.getFullYear(), 0, 0)) / 1000 / 60 / 60 / 24);

dayOfYear(new Date());   // 307
```
**时间格式化**  
```
const timeFromDate = date => date.toTimeString().slice(0, 8);
    
timeFromDate(new Date(2021, 11, 2, 12, 30, 0));  // 12:30:00
timeFromDate(new Date());  // 返回当前时间 09:00:00
```
## 字符串处理
**字符串首字母大写**  
```
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1)

capitalize("hello world")  // Hello world
```
**翻转字符**  
```
const reverse = str => str.split('').reverse().join('');

reverse('hello world');   // 'dlrow olleh'
```
**随机字符串**  
```
const randomString = () => Math.random().toString(36).slice(2);

randomString();
```
**截断字符串**  
```
const truncateString = (string, length) => string.length < length ? string : `${string.slice(0, length - 3)}...`;

truncateString('Hi, I should be truncated because I am too loooong!', 36)   // 'Hi, I should be truncated because...'
```
**去除字符串中的HTML**
```
const stripHtml = html => (new DOMParser().parseFromString(html, 'text/html')).body.textContent || '';
```
## 数组处理
**从数组中移除重复项**  
```
const removeDuplicates = (arr) => [...new Set(arr)];

console.log(removeDuplicates([1, 2, 2, 3, 3, 4, 4, 5, 5, 6]));
```
**判断数组是否为空**  
```
const isNotEmpty = arr => Array.isArray(arr) && arr.length > 0;

isNotEmpty([1, 2, 3]);  // true
```
**合并两个数组**  
```
const merge = (a, b) => a.concat(b);

const merge = (a, b) => [...a, ...b];
```
## 数字操作
**判断一个数是奇数还是偶数**  
```
const isEven = num => num % 2 === 0;

isEven(996); 
```
**获得一组数的平均值**  
```
const average = (...args) => args.reduce((a, b) => a + b) / args.length;

average(1, 2, 3, 4, 5);   // 3
```
**获取两个整数之间的随机整数**  
```
const random = (min, max) => Math.floor(Math.random() * (max - min + 1) + min);

random(1, 50);
````
**指定位数四舍五入**  
```
const round = (n, d) => Number(Math.round(n + "e" + d) + "e-" + d)

round(1.005, 2) //1.01
round(1.555, 2) //1.56
```
## 颜色操作
**将RGB转化为十六机制**
```
const rgbToHex = (r, g, b) => "#" + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1);

rgbToHex(255, 255, 255);  // '#ffffff'
```
**获取随机十六进制颜色**  
```
const randomHex = () => `#${Math.floor(Math.random() * 0xffffff).toString(16).padEnd(6, "0")}`;

randomHex();
```
## 浏览器操作
**复制内容到剪切板**  
```
const copyToClipboard = (text) => navigator.clipboard.writeText(text);

copyToClipboard("Hello World");
```
**清除所有cookie**  
```
const clearCookies = document.cookie.split(';').forEach(cookie => document.cookie = cookie.replace(/^ +/, '').replace(/=.*/, `=;expires=${new Date(0).toUTCString()};path=/`));
```
**获取选中的文本**  
```
const getSelectedText = () => window.getSelection().toString();

getSelectedText();
```
**检测是否是黑暗模式**  
```
const isDarkMode = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches
```
**滚动到页面顶部**  
```
const goToTop = () => window.scrollTo(0, 0);

goToTop();
```
**判断当前标签页是否激活**  
```
const isTabInView = () => !document.hidden; 
```
**判断当前是否是苹果设备**
```
const isAppleDevice = () => /Mac|iPod|iPhone|iPad/.test(navigator.platform);

isAppleDevice();
```
**是否滚动到页面底部**  
```
const scrolledToBottom = () => document.documentElement.clientHeight + window.scrollY >= document.documentElement.scrollHeight;
```
**重定向到一个URL**  
```
const redirect = url => location.href = url

redirect("https://www.google.com/")
```
**打开浏览器打印框**  
```
const showPrintDialog = () => window.print()
```
## 其他操作
**随机布尔值**  
```
const randomBoolean = () => Math.random() >= 0.5;

randomBoolean();
```
**变量交换**  
```
[foo, bar] = [bar, foo];
```
**获取变量的类型**  
```
const trueTypeOf = (obj) => Object.prototype.toString.call(obj).slice(8, -1).toLowerCase();

trueTypeOf('');     // string
trueTypeOf(0);      // number
trueTypeOf();       // undefined
trueTypeOf(null);   // null
trueTypeOf({});     // object
trueTypeOf([]);     // array
trueTypeOf(0);      // number
trueTypeOf(() => {});  // function
```
**华氏度和摄氏度之间的转化**  
```
const celsiusToFahrenheit = (celsius) => celsius * 9/5 + 32;
const fahrenheitToCelsius = (fahrenheit) => (fahrenheit - 32) * 5/9;

celsiusToFahrenheit(15);    // 59
celsiusToFahrenheit(0);     // 32
celsiusToFahrenheit(-20);   // -4
fahrenheitToCelsius(59);    // 15
fahrenheitToCelsius(32);    // 0
```
**检测对象是否为空**  
```
const isEmpty = obj => Reflect.ownKeys(obj).length === 0 && obj.constructor === Object;
```





原文:  
[33个非常实用的JavaScript一行代码，建议收藏！](https://juejin.cn/post/7025771605422768159?utm_source=gold_browser_extension)