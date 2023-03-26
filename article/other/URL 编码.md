# URL 编码
Web 项目中经常会遇到处理 URL 中 Query 的情况  
- 项目中发现会用到 qs、query-string、URLSearchParams、甚至 querystring 几种不同的库，其到底差异在哪里，该用哪个
- 在 query 中 key=a&key=b 这种情况 key 取值是什么？和 key[]=a&key[]=b 有区别嘛
- 在 query 中会有结构如 %HH 的数据，为什么是这样形式的？我们为什么要使用 encodeURIComponent 进行编码？和过时的 escape 又有何区别？
- Content-type 中 x-www-form-urlencoded 的取值，是怎么一回事

## URL QueryString
通常的理解就是 URL 中问号（?）后面的部分，其设计最初是用做 HTML form 表单提交时的传参  

**基本结构**  
query 的基本结构包含了如下标准：
- Query String 由一组键值对(field-value)组成
- 每组键值对的 field 和 value 用=分割
- 每组数据用&分割
- 允许多个value被关联到同一个field上，但 field 如何取值，其实并无明确的处理标准。
  例如：field=a&field=b时，field 的值应该是 a、b、['a', 'b']、'a, b' 并无任何权威解释。

补充个冷知识：除了使用&分割每对数据外，W3C 曾在 1999 年建议所有 Web 服务器同时支持分号;分割符,但在 2014 年以来，就只建议使用 & 作为分隔符了。  
通常这类情况会按照数组的方式处理，即 field 值为 ['a', 'b']，但这仅是不同的框架的决定了如何实现而已。  

**数据编码**  
由于某些字符集（如中文）和在 URL 中有特殊含义的字符（如 空格、%、&、=、?、# 等）无法直接在 Query String 中使用，因此使用了一种叫做「百分号编码 Percent-encoding的方式先将这类特殊字符进行编码后，再进行传输。  
其基本结构就是 % + 2 个 16 进制数字（一个 Byte 的内容），范围 %00 - %FF。  
具体规则如下：
- 对保留字符进行编码
- 如下非保留字符不进行编码，包含：[A-Z]、[a-z]、[0-9]、- 、_、.、~；
- %百分号编码为%25
- 空格编码为+或%20
- 其余字符数据使用某种编码方式转换为字节流，再用百分号编码%HH方式表示。
  这里需要注意的是，由于早期规范中未明确应使用何种编码，所以会导致如果不明确说明使用何种编码，数据的解析会有歧义。因此在 2005 年发布的 RFC 3986 建议是先转成 UTF-8 编码，再对每个字节进行%HH的编码

注意，如果使用 from 表单 action 方式时，具体编码会根据 meta 头的 charset 的选择。  
上述只是标准，实践中 JavaScript 内置了使用 UTF-8 编码的 encodeURI/encodeURIComponent 函数，大大简化的编码过程。  

## 编码实践
### qs
官方介绍很简单：一个增加了安全性的 Query String 解析和序列化的函数库  
**.parse(string, [options])**  
1. 对于简单 query，可以进行常规的转换，同时会对 field 和 value 进行 decode 解码  
``` 
qs.parse('a=c&b%201=d%26e');

// { a: 'c', 'b 1': 'd&e' }
```
qs 不会忽略头部的 ?，需要自行去掉，否则会当做 field 的一部分，例如：qs.parse('?a=b')会解析为 { '?a': 'b' }。  

2. 支持 query 中的嵌套对象  
``` 
qs.parse('foo[bar]=baz');

// { foo: { bar: 'baz' } }
```
默认子元素最多嵌套 5 层，需要通过 parse(string, [options]) 的 opinion.depth 来修改。  
``` 
// defalut
qs.parse('a[b][c][d][e][f][g][h][i]=j');

// {a: {b: {c: {d: {e: {f: {'[g][h][i]': 'j'}}}}}}}

// set depth
qs.parse('a[b][c][d][e][f][g][h][i]=j', { depth: 1 });
// { a: { b: { '[c][d][e][f][g][h][i]': 'j' } } }
```
3. 支持自定义除&以外的分隔符  
``` 
var delimited = qs.parse('a=b;c=d', { delimiter: ';' });
// { a: 'b', c: 'd' }
```
这点符合 W3C 对;支持的建议，但大部分情况应该不会用到。  
4. 支持各种 array 的解析，虽然官方文档写了 [] 作为数组标识，但实际上不使用[]依然可以解析**  
``` 
var withArray = qs.parse('a[]=b&a[]=c');
// { a: ['b', 'c'] }

var withArray = qs.parse('a=b&a=c');
// { a: ['b', 'c'] }
```
同时也支持为数组指定索引顺序。  
``` 
var withIndexes = qs.parse('a[1]=c&a[0]=b');
// { a: ['b', 'c'] };
```
并行支持 allowSparse 获取抽稀形式的数组  
``` 
var sparseArray = qs.parse('a[1]=2&a[3]=5', { allowSparse: true });
// { a: [, '2', , '5'] };
```
默认指定的 index 最大值为 20，如果超过最大值，则按照 object 形式解析。使用 arrayLimit控制最大值。  
``` 
var withMaxIndex = qs.parse('a[100]=b');
// { a: { '100': 'b' } }

var withArrayLimit = qs.parse('a[1]=b', { arrayLimit: 0 });
// { a: { '1': 'b' } }
```
**.stringify(object, [options])**  
qs 默认会对 field 和 value 都进行编码，同时会使用[]作为数据的标识（且默认对[]进行百分号编码），需指定 encodeValuesOnly: true才仅对 value 编码。  
``` 
// defalut
qs.stringify({key: ['a', 'b']});
// key%5B0%5D=a&key%5B1%5D=b

//
qs.stringify({key: ['a', 'b']}, { encodeValuesOnly: true });
// key[0]=a&key[1]=b
```
去掉[]标识，可使用 { indices: false }  
``` 
qs.stringify({key: ['a', 'b']}, { indices: false });
// key=a&key=b
```
**支持配置 charset**  
默认使用 UTF-8，内置了 ISO-8859-1 模式，也可以支持 encoder 扩展。

### 简洁专注 query-string
**parse(string, [options])**  
_基本的解析和 qs 一样，会对 field 和 value 进行 decode_  
不过，头部的?和#的部分将被忽略，因此可以直接将 location.search 和 location.hash 传入。  
``` 
queryString.parse('a=c&b%201=d%26e');

// { a: 'c', 'b 1': 'd&e' }
```
_不支持嵌套，官方建议可以使用 JSON 序列化的方式传值_  
_query 中 array 的解析，默认不支持[]形式，需要指定 { arrayFormat: 'bracket' }开启。_  
``` 
queryString.parse('key=a&key=b');
// { key: ['a', 'b'] };

queryString.parse('key[]=a&key[]=b');
// { 'key[]': ['a', 'b'] };

queryString.parse('key[]=a&key[]=b', { arrayFormat: 'bracket' });
// { key: ['a', 'b'] };
```
当然 query-string 也支持索引的方式标记的数组，{arrayFormat: 'index'}  
``` 
queryString.parse('foo[0]=1&foo[1]=2&foo[3]=3', {arrayFormat: 'index'});
{foo: ['1', '2', '3']}
```
**.stringify(object, [options])**  
点介绍 array 类型的编码，默认不使用[]标识  
``` 
queryString.stringify({key: ['a', 'b']});
// key=a&key=b
```
需要[]的话，使用 {arrayFormat: 'bracket'}开启，默认[]也不会被 encode  
``` 
queryString.stringify({key: ['a', 'b']}, {arrayFormat: 'bracket'});
// key[]=a&key[]=b
```

### 历史产物 querystring
NodeJS 14.x 中明确标记为 Legacy，官方推荐 URLSearchaParms 代替。功能类似 query-string，不支持嵌套对象的解析

### 血统纯正 URL / URLSearchParams
URL 和 URLSearchParams 是 URL API 规范 中的两个标准的接口。其提供了访问、操作 URL 的 API。
其中，URL 定义了像域名、主机和 IP 地址等概念，URLSearchParams 定义了一些常用的方法来处理 Query String。  

**URLSearchParams**  
两种方式创建 URLSearchParams 对象，URLSearchParams构造函数会忽略 search 中的?  
``` 
// 1. 通过 URL
const url = new URL('https://abc.com/path/v1?key=a&key=b%26c');
const search1 = url.searchParams;

// 2. 直接构造
const search2 = new URLSearchParams(location.search);
```
_.get(name)_  
该方法获取的值会被自动 decode，如果 name 不存在返回 null，如果 value 不存在返回空字符串  
``` 
const search = new URLSearchParams('key=b%26c&key2');
search.get('key'); // b&c
search.get('key2'); // ''
search.get('key3'); // null
```
_.getAll(name)_  
需要特别注意，如果有多个相同的 name，get() 只能获取第一个值。获取全部需要使用 getAll()，该函数返回数组（即便只有一个 value）  
``` 
const search = new URLSearchParams('key=a&key=b');
search.get('key'); // a
search.getAll('key'); // ['a', 'b']
```
_.set(name, string) / .append(name, string)_  
向 URLSearchParams 中添加数据，set() 会覆盖原有值。如果需要添加重复的 name，需要使用 append()  
set() 和 append() 仅支持 string 类型的 value。同时 field 和 value 都会被 encode，无需额外处理  
``` 
const search = new URLSearchParams();
search.append('key', 'a');
search.append('key', 'b');

search.toString(); // key=a&key=b
```
_.keys()_  
返回一个 IterableIterator迭代器，可以使用for...of遍历。需要注意，重复的 key 会出现多次  
``` 
const search = new URLSearchParams('key=a&key=b');
for (const key of search.keys()) {
  console.log(key);
}
// key
// key
```
_.toString()_  
获取的 Query String，会被自动 encode 处理。空格转成+。对于重复 field，使用了 field=v1&field=v2 的方式。  
``` 
const search = new URLSearchParams();
search.set('key', '?&=')
search.set('key2', 'a b');
search.toString(); // key=%3F%26%3D&&key2=a+
```

## 总结对比
qs 和 query-string / URLSearchParams 最大的差异在于对于多层嵌套对象（Nested object）的支持与否  
- qs被设计用于解析x-www-form-urlencoded数据，拥有强大的序列化能力，可以处理复杂的类 JSON数据。
- query-string 和 URLSearchParams 则使用简单的序列化算法，适合常规的 Web 端数据传输，处理平面数据结构
- 对于平面数据，以上效果是一样的

当使用复杂的 JSON 数据结构时，我们通常会使用JSON.stringify() 方法先将数据进行序列化（也称字符串化），将复杂数据转换成基本的字符串数据后，再进行传输
- 外如果有特殊编码需求，除qs外都仅支持 UTF-8 的编码

因此通常情况下：
- 在 Web 项目中解析 GET 形式的 query，使用 URLSearchParams 就足够了（可代替query-string）；
- 而在 NodeJS 项目中，除了解析 GET query 外，还要解析 POST body 中的数据，因此使用 qs 可以获得更好的兼容性。同时不少框架也依然使用了 querystring 这个原生 API。

原文:  
[关于编码的那些事 - URL 编码](https://mp.weixin.qq.com/s/QQEq-_84SxcLzAbRHSuYVw)
