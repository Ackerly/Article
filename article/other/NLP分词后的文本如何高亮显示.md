# NLP分词后的文本如何高亮显示
**文本高亮显示**  
首先文本是段落结构，存在多个段落，其次是每段文本中，存在多句文本；例如：‘武汉大学坐落于武汉市珞珈山’  
所以后端返回的数据格式形式：
```
const data = [{
  "paraId": 0,
  "paraText": "武汉大学坐落于武汉市珞珈山",
  "paraEntity": [{
     "category": "名词",
     "labelText": "武汉大学",
    "startIndex": 0,
    "endIndex": 4,
    "color": 'rgba(240,215,12,.5)'
  }, {
     "category": "地名",
     "labelText": "武汉",
    "startIndex": 7,
    "endIndex": 9,
    "color": '#00baff'
  }]
}];
```
**前端如何在文本中高亮显示呢**  
替换法：
> 将后端返回的数据格式提取形成一个数组，里面的元素以对象的形式存储，然后循环遍历，将文本中需要高亮的词替换

存在的问题很明显，无法针对相同的词汇给出不同的词性,比如：*武汉*大学坐落于*武汉*市珞珈山  
这里识别前面武汉大学是一个名词，武汉是一个地名，通过上述方法，会将所有武汉标记为地名或者其它词性;  
拆分法：  
将文本中每一个字遍历，替换成em标签；拆分形成的em标签文本，我们可以按每个字去显示不同的颜色，也就是词性；
data属性：  
> data-* 全局属性 是一类被称为自定义数据属性的属性，它赋予我们在所有 HTML 元素上嵌入自定义数据属性的能力，并可以通过脚本在 HTML 与 DOM 表现之间进行专有数据的交换。  

使用js中的 getAttribute 方法这样我们可以通过后台给定的起始位置和长度，定位到相应的文本  
```
/* 切割和拼成em */
// paranum是段落数 index是这句话中第几个字
createElement(textStr, tagName = 'em', k) {
  let NewTextStr = '';
  for (let i = 0; i < textStr.length; i += 1) {
    NewTextStr += `<${tagName} data-paranum="${k}" data-index="${i}">${textStr[i]}</${tagName}>`;
  }
  return NewTextStr;
},
```
由于文本中会存在特殊文号，比如书名号‘《》’， 括号‘（）’，所以需要对此进行转义  
```
/* 转义 */
escapeStr(str) {
  let regStr = '';
  const specialArry = ['(', ')', '[', ']', '\\', '+', '*', '?', '.', '|'];
  for (let k = 0; k < str.length; k += 1) {
    if (specialArry.indexOf(str[k]) > -1) {
      regStr += `\\${str[k]}`;
    } else {
      regStr += str[k];
    }
  }
  return regStr;
},
```
**总结**  
1. 先将段落文本每一个字切割合并成em标签，并加上data属性（段落以及字位置）；
2. 循环遍历数据，最小单位为每个字;
3. 在循环中拼接非高亮的字符串以及高亮的字符串
    ```
    notHightStr = <em data-paranum="${k}" data-index="${i + e.paraEntity[m].startIndex}">${e.paraEntity[m].labelText[i]}</em>`;
    hightStr = `<em data-paranum="${k}" data-index="${i + e.paraEntity[m].startIndex}" style="background: ${ e.paraEntity[m].color};">${e.paraEntity[m].labelText[i]}</em>`;
    ```
4. 替换
    ```
    let emStr = createElement(fileText, 'em', k);
    emStr = emStr.replace(notHightStr, hightStr);
    ```
5. 生成拼接后的高亮html
    ```
    
    ```
原文:  
[NLP分词后的文本如何在页面高亮显示](https://juejin.cn/post/7068545836271009823?utm_source=gold_browser_extension)