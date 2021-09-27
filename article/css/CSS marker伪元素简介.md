# CSS ::marker伪元素简介
## 了解::marker伪元素
::marker是CSS中新出的一种伪元素，用来匹配列表项中的“标记盒子”，并可以设置标记盒子里面的内容以及与字符显示相关的UI。可以匹配任意设置了display:list-item的元素或伪元素，例如<li>元素就可以直接使用::marker伪元素改变项目符号颜色、字号字体、甚至内容。
``` 
<ol>
    <li>有序列表</li>
    <li>作者张鑫旭</li>
    <li>看看序号的颜色？</li>
</ol>
::maker {
    color: deepskyblue;
    font-weight: bold;
}
```
### 普通元素应用::marker
如果是普通的HTML标签元素，例如<div>元素想要使用::marker伪元素，可以设置display为list-item，代码示意：
``` 
<div class="marker">summary元素有自己的marker伪元素</div>
div.marker {
  display: list-item;
  margin-left: 1em;
  padding-left: 5px;
}
div.marker::marker {
  content: '▶';
}
```
实时渲染效果如下（左侧应该是个三角尖头，如果浏览器不支持会是一个圆点，如果什么都没有，您访问的是盗版）
其中:
- content:'▶'不是必须的，默认就会创建符号‘·’作为项目符号
- margin-left:1em也不是必须的，可以设置list-style-position:inside让项目符号字符的位置在标签内。
- 标记字符可以是任意字符，数量也不限

注意，Safari浏览器目前还不支持content自定义标记符号，仅支持list-style-type属性设置标记符号，是时候祭出这张十几年的老图了。  
## 只支持部分CSS
和::first-letter伪元素、::first-line伪元素类似，::marker伪元素仅支持部分的CSS属性，具体如下：  
- white-space属性；
- text-shadow属性（仅Chrome支持），其他text相关属性并不支持；
- letter-spacing和word-spacing属性（仅Chrome支持）；
- color属性；
- text-combine-upright、unicode-bidi和direction属性，这几个属性与文字排版方位相关；
- content属性，Safari目前不支持
- 所有动画和过渡相关的CSS属性，也就是animation和transition属性；

::marker伪元素支持的CSS属性里面支持动画的CSS属性并不多，也就是color属性能用.  
Firefox浏览器虽然很早就支持了::marker伪元素，但是::marker支持动画是80这个版本才开始支持的
``` 
.marker {
  display: list-item;
}
.marker::marker {
  transition: color .2s;
  content: '▶';
}
.marker:hover::marker {
  color: deepskyblue;
}
```

## ::before/::after中使用::marker  
::before::marker和::after::marker选择器都是合法的，只需要::before和::after是列表项，也就是display计算值是list-item

