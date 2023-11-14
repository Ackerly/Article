# 使用Intl segmenter API
## Polypane 如何利用分词 API
在 Polypane 中选择文本并右键打开菜单时，会显示选中文本中的字符、单词和表情符号数，这就是利用了分词 API。  
可以快速检查推文字符数是否正确，或者博客文章词数是否达标。  

**配置segmenter API**  
坏消息是，该 API 仅在 Safari 14.1+ 和 Chromium 87+ 可用，Firefox 还不支持。有个 polyfill 但似乎非常笨重。  
使用时，先创建一个分词实例，需要指定语言和选项。  
``` 
 const segmenter = new Intl.Segmenter('en', { granularity: 'word' });
```
选项对象有两个可选属性:
- localeMatcher 可选 'lookup' 或 'best fit'(默认)。'lookup' 使用特定匹配算法 (BCP 47),'best fit' 使用更宽松算法尽可能匹配可用语言。
- granularity 可选 'grapheme'(默认)、'word' 或'sentence', 决定返回的段落类型。默认 'grapheme' 返回可见字符。

调用 segmenter.segment 传入要分割的文本，它返回一个迭代器，可以用 Array.from 转换为数组:  
``` 
const segments = segmenter.segment('This has four words!');
 Array.from(segments).map((segment) => segment.segment);
 // ['This', ' ', 'has', ' ', 'four', ' ', 'words', '!']
```
迭代器每个项目是一个对象，根据粒度它包含不同的值，至少有:
- segment 段落文本。
- index 段落在原文中的索引，如起始位置。
- input 原文。

单词粒度还包含:
- isWordLike 布尔值，如果是单词则为 true

要获取单词，需过滤数组:  
``` 
const segments = segmenter.segment('This has four words!');
 Array.from(segments)
   .filter((segment) => segment.isWordLike)
   .map((segment) => segment.segment);
 // ['This', 'has', 'four', 'words']
```
不仅可以分割单词像上例，还可以分割句子，它能识别句号是结束句子还是数字的千分位分隔符:  
``` 
const segmenter = new Intl.Segmenter('en', { granularity: 'sentence' });
 const segments = segmenter.segment('This is a sentence with the value 100.000 in it. This is another sentence.');

 Array.from(segments).map((segment) => segment.segment);
 // [
 //    'This is a sentence with the value 100.000 in it.',
 //    'This is another sentence.'
 // ]
```
没有 isSentenceLike 和 isGraphemeLike 属性，可以直接统计项目数。  

## 支持其他语言
API 的实力在于，它不仅根据空格或句号来分割，而是为每种语言使用特定算法。这意味着它可以分割没有空格的语言单词，如日语:  
``` 
const segmenter = new Intl.Segmenter('ja', { granularity: 'word' });
 const segments = segmenter.segment('これは日本語のテキストです');

 Array.from(segments).map((segment) => segment.segment);
 // ['これ', 'は', '日本語', 'の', 'テキスト', 'です']
```
与字符串拆分或正则表达式相比，这更准确，正确处理非拉丁字符和标点，代码更少，无需额外检查和测试。浏览器内置大量语言支持，你不必担心这点。  

## 深入了解 Intl API
Intl API 提供大量创建自然语言构造的实用工具，如列表、日期、货币、数字等。总体来说使用简单，但它使用了你可能不熟悉的精确术语 (如使用 “字形” 而不是 “字符”, 后者太模糊), 刚开始可能有点难搞。如果从未使用过可以查看 MDN 上的 Intl 介绍。  
Polypane 的下个版本会有跨语言的文本测量选项，并显示选中文本的句子数。  


原文:  
[使用Intl segmenter API](https://mp.weixin.qq.com/s/ryMweb0phD93tiQymukrFg)
