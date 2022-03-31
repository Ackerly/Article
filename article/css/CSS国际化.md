# CSS国际化
**与语言相关的样式**  
你有没有想过Chrome怎么会问你是否想翻译网页的内容?这是因为HTML元素上的lang属性。  
lang属性是一个非常重要的属性，因为它识别WEB上文本内容的语言，并且该信息在许多地方都被使用。前面提到的Chrome内置翻译，针对特定语言内容的搜索引擎，以及屏幕阅读器。  
与语言相关的样式化的关键在于在页面标记中适当使用lang属性。lang属性将ISO 639语言代码识别为值。
更新:Tobias Bengfort指出lang属性使用了一个名为BCP 47的IETF规范，该规范很大程度上基于ISO 639标准。
在大多数情况下，您将使用像zh这样的两个字母的代码来表示中文，但是中文(包括阿拉伯语等其他语言)被认为是一种宏语言，它由许多具有更具体的主语言子标记的语言组成。  
一般的指导是html元素必须总是有一个lang属性集，然后由所有其他元素继承。  
在同一页上看到不同语言的内容并不少见。在本例中，使用span或div包装该内容，并将正确的lang属性应用于包装元素。
```
<p>The fourth animal in the Chinese Zodiac is Rabbit (<span lang="zh">兔子</span>).</p>
```
**:lang()伪类选择器**  
这个伪类选择器可以识别内容的语言，即使该语言是在元素之外声明的。  
行包含两种语言的标记，如下所示：  
```
<p>
  We use <em>italics</em> to emphasise words in English, <span lang="zh">但是中文则是用<em>着重号</em></span>.
</p>
```
可采用以下样式：
```
em:lang(zh) {
  font-style: normal;
  text-emphasis: dot;
}
```
用斜体字强调英语中的单词，但是中文则是用着重号.
lang属性不是应用于em元素，而是应用于其父元素。伪类仍然有效。如果我们使用更常见的属性选择器，例如[lang=“zh]，则该属性必须位于em元素上才能生效。  
**使用属性选择器**  

参考：
[CSS国际化](https://mp.weixin.qq.com/s/LCgdM02tXHMOFiQ7wC0H7Q)