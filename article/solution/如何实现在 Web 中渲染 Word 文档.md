# 如何实现在 Web 中渲染 Word 文档
## 方案选择
具体如何做到？已知的方案有如下几种：  
1. 使用微软的 Office Online 服务，8 年前我曾经搭过一个，还需要搭 AD 服务，挺麻烦的
    - 优点：实现简单，效果有保证，毕竟是微软自己出的
    - 缺点：除了部署复杂，还需要客户有微软的 Volume Licensing，并不是所有客户都有，价格也不便宜
2. 使用国内的 WPS、永中等厂商的产品
    - 优点：实现简单，效果不比微软的差，还是信创的
    - 缺点：价格也不便宜
3. 使用工具将 Word 转成 HTML 或 PDF 格式
    - 优点：实现相对简单，如果使用开源的产品还无需付费
    - 缺点：需要配合后端服务进行格式转换，需要实现对结果的存储等功能，如果较慢还需要加队列等机制，整合起来较为麻烦
4. 直接前端解析 Word 文件并渲染
    - 优点：无需后端服务，对接成本低，还能支持基于前端变量的动态渲染
    - 缺点：现有开源实现效果一般，需要自己实现；难以支持老的 doc 格式

最终我们选择了前端渲染方案，因为接入成本低  
接下来的问题是前端使用什么技术实现，目前主要有两种：
- 使用 HTML+CSS 渲染，优点是实现成本低，缺点是某些功能及定位方式难以实现，导致复杂布局的文档还原度低
- 使用 Canvas 渲染，优点是渲染效果可以无限接近桌面端，是国内 WPS 等商业产品使用的方案，但实现成本较高，比如表格布局得从零实现

大部分文档并没有使用复杂的布局，基本都是常见的表格、图片、列表等功能，偶尔有几个形状及文本框，而且Office 365 的在线版本就是用 HTML 渲染的，因此使用 HTML 预计可以满足我们的基本需求，最终就选择了这个方案  

## HTML+CSS 渲染细节
源项目 docxjs，它已经实现了不少功能，包括图片、列表、表格等，但我们没直接使用，主要是以下方面的考虑：  
- 这个项目基本只有一个人提交，很不稳定，如果有不满足的需求提 Pull requests 可能要等很久
- 需要整合 amis 上下文做动态渲染功能，这部分代码合入官方也不合适
- 整个项目大概 5k 行代码，从零开始写一遍成本也不高

## 文件解析
首先第一步是解析文档，Word 文档主要有两种格式 doc 和 docx。  
其中 doc 是老的二进制格式，早年这个文件格式没有说明文档，WPS 之前是自己逆向二进制分析出来的，后来微软放出了个文档，基本上就是各种 C 结构体解析，文档有 210 页，完整实现一遍成本很高  
尝试了找一下开源项目：
- 在 npm 仓库里没找到，大概没人用 JavaScript 实现过，无果
- 在 Java 语言中有个 POI 项目里有相关实现，但我们没法在前端使用 Java 代码，而且这个项目代码有 5w 行，参考它转成 JavaScript 实现也很花时间
- 在 C++ 语言里的 OpenOffice 和 ONLYOFFICE 都有相关实现，理论上可以将它们的相关代码抠出来，独立实现个 doc 转 docx 工具，然后编译成 WebAssembly 给前端用，但简单尝试后发现依赖很多，工作量也不小

由于前端解析成本太高，暂时放弃了 doc 格式支持，让用户自己转成 docx 格式。  
docx 格式的解析就容易多了，它就是个 zip 文件，里面的文档全都是 XML 格式，JavaScript 可以轻松解析。  
zip 可以使用 JSZip 解压，但它只有异步接口，导致所有对接地方都得换成异步，用起来比较麻烦，所以找了个更轻量级的项目 fflate，它体积小还支持同步。  
那 XML 怎么解析呢？最简单就是用浏览器自带的 DOMParser，但一开始我没用，因为找个 XML 转 JSON 的工具，这样只需要一次转换，后面的节点变量全都是 js 对象原生操作，速度也肯定比调用 XML 的 DOM 函数快得多。  
然而XML 并不能很好转成 JSON,比如类似下面的 xml
``` 
<w:body>
     <w:p></w:p>
     <w:tbl></w:tbl>
 </w:body>
```
使用fast-xml-parser 的库转成 JSON,最终要转成什么 json 格式呢？最简单的就是
``` 
 {
   "body": {
     "p": {},
     "tbl": {}
   }
 }
```
但这么无法支持多个 p 和 tpl，因此必须改成数组形式，比如  
``` 
 {
   "body": [
     {
       "p": {}
     },
     {
       "tbl": {}
     }
   ]
 }
```
另一方面 w:body 的属性也得找个地方放，所以最后其实发现最好的格式是类似 dom 的结构，变成  
``` 
{
   "tag": "body",
   "attributes": {},
   "children": []
 }
```
单 fast-xml-parser 并不支持，它输出的格式在只适合简单 xml 场景，复杂 xml 反而更复杂了，对输出结果定义类型非常麻烦，所以最后改成了使用原生 DOMParser，最后发现性能其实并不差。  
## 文档格式转换
实现过程大概就是这几步：
- 看 Open Open XML 规范
- 使用桌面 Word 编写简单示例，查看最终 XML
- 解析 XML 转成内部数据结构，这么做的目的是后续操作都是强类型，保证了质量
- 最后递归这个数据结构，将其转成 HTML+CSS 格式

Word 和 HTML 都属于文档，它们直接大部分样式属性是相通的，比如下面是其中的一部分转换代码示例：  
``` 
case 'w:b':
     // http://webapp.docx4java.org/OnlineDemo/ecma376/WordML/b.html
     style['font-weight'] = getValBoolean(child) ? 'bold' : 'normal';
     break;

 case 'w:i':
     // http://webapp.docx4java.org/OnlineDemo/ecma376/WordML/i.html
     style['font-style'] = getValBoolean(child) ? 'italic' : 'normal';
     break;

 case 'w:caps':
     // http://webapp.docx4java.org/OnlineDemo/ecma376/WordML/caps.html
     style['text-transform'] = getValBoolean(child) ? 'uppercase' : 'normal';
     break;
```
不过有部分样式不好转换，比如边框样式在 Word 中有 193 种，而 CSS 中只有 8 种，这是目前方案和 Canvas 相比无法实现的功能之一。  
如何减少 typo？Word 规范中有大量属性，开发过程中很容易不小心拷错了，很多属性还是简写让人误以为拼错了，比如列表是 Lst 而不是 List，所以最好能找到所有属性的 TypeScript 类型定义，这样能极大避免 typo 问题，但官方并没有提供这个，所有开源项目都是自己在项目里又定义了一遍，非常费时，有没有什么方法简化呢？  
在规范里找了一圈后发现有个 xsd 文件，这是 XML 的 Schema 定义，因此可以通过解析这些 xsd 文件，自动生成 TypeScript 定义，降低了开发成本。  
另外这里有个小细节，就是对于枚举类型使用哪种 TypeScript 定义，可以是以下两种：  
``` 
enum ST_Em {
     None = 'none',
     Dot = 'dot',
     Comma = 'comma',
     Circle = 'circle',
     UnderDot = 'underDot'
 }

 type ST_Em = 'none' | 'dot' | 'comma' | 'circle' | 'underDot';
```
从功能上看两种是一样的，第一种看上去更专业点，但选择了第二种，为什么？  
因为第一种编译后会多个对象，而第二种编译后没有内容，而 Word 里的类型定义有几千个，光这些类型定义编译出来就占了不少体积，也没必要。  
下面介绍实现过程中几个不太一样的细节点：  
- 流式布局及版式布局
- 表格
- 形状
- 公式
- 动态字段

**流式布局及版式布局**  
虽然 Word 也属于文档，但它更偏向版式布局，每个 Word 文档都必须指定高宽，这使得在渲染时有些功能和 Web 不一致。  
比如分页符，在插入这个符号后，后面的内容会从新页面开始，甚至这个分页符还可以指定单数页还是偶数页，但这个功能在 Web 流式渲染里没有意义，因为流式布局没有分页。  
为了能更好还原桌面端 Web 效果，加了个单独配置项，可以使用分页方式渲染,这样就能得到比较接近的效果了。  

**表格的实现**  
接下来是表格和列表的实现，Word 中单元格合并写法和 HTML 中不太一致，需要做一下转换。  
实现了表格合并之后就能渲染大量代表的文档了，虽然最终效果由于字体和行间距导致有些差异，但布局是正确的  
不过使用 HTML 渲染也有缺点，尤其是绝对定位难以解决，比如图片在单元格里使用 relative 定位是不生效的。  

**形状的实现**  
Office 里的形状定义可以在 presetShapeDefinitions.xml 中找到，比如右侧箭头是如下配置（去掉了不相关的 ahLst 和 cxnLst）  
``` 
 <rightArrow>
     <avLst xmlns="http://schemas.openxmlformats.org/drawingml/2006/main">
       <gd name="adj1" fmla="val 50000" />
       <gd name="adj2" fmla="val 50000" />
     </avLst>
     <gdLst xmlns="http://schemas.openxmlformats.org/drawingml/2006/main">
       <gd name="maxAdj2" fmla="*/ 100000 w ss" />
       <gd name="a1" fmla="pin 0 adj1 100000" />
       <gd name="a2" fmla="pin 0 adj2 maxAdj2" />
       <gd name="dx1" fmla="*/ ss a2 100000" />
       <gd name="x1" fmla="+- r 0 dx1" />
       <gd name="dy1" fmla="*/ h a1 200000" />
       <gd name="y1" fmla="+- vc 0 dy1" />
       <gd name="y2" fmla="+- vc dy1 0" />
       <gd name="dx2" fmla="*/ y1 dx1 hd2" />
       <gd name="x2" fmla="+- x1 dx2 0" />
     </gdLst>
     <rect l="l" t="y1" r="x2" b="y2" xmlns="http://schemas.openxmlformats.org/drawingml/2006/main" />
     <pathLst xmlns="http://schemas.openxmlformats.org/drawingml/2006/main">
       <path>
         <moveTo>
           <pt x="l" y="y1" />
         </moveTo>
         <lnTo>
           <pt x="x1" y="y1" />
         </lnTo>
         <lnTo>
           <pt x="x1" y="t" />
         </lnTo>
         <lnTo>
           <pt x="r" y="vc" />
         </lnTo>
         <lnTo>
           <pt x="x1" y="b" />
         </lnTo>
         <lnTo>
           <pt x="x1" y="y2" />
         </lnTo>
         <lnTo>
           <pt x="l" y="y2" />
         </lnTo>
         <close />
       </path>
     </pathLst>
   </rightArrow>
```
其中 avLst 和 gdLst 都是用来定义变量，有个公式 fmla 字段来对各个变量进行计算来得到新变量，而 pathLst 是类似 SVG 的 path 命令。  
fmla 公式的解析和执行比较简单，因为它只有一个操作，比如 <gd name="x1" fmla="+- r 0 dx1" /> 等价于 x1 = r + 0 - dx1。  
后面的 path 大部分都有对应的 SVG path 命令，比如 moveTo 就是 M，lnTo 就是 L，然而有个指令很麻烦，就是 arcTo，它的参数和 SVG 的 A 命令不一样。  
arcTo 使用的参数是：
- hR，椭圆半高度
- wR，椭圆半长度
- stAng，起始角度
- swAng，旋转角度

而 SVG 中 A 命令参数是
- rx/ry，中心点的 x 和 y 坐标
- x-axis-rotation，旋转角度
- large-arc-flag，是否是大弧，0 代表否
- sweep-flag，绘制方向，0 代表逆时针
- x/y，绘制终点的坐标

所以问题就变成了如何根据 arcTo 参数计算出对应 A 命令的参数，w3c 文档里给出了基于中心点计算不同角度下的椭圆曲线坐标位置的公式:将这个公式右边的中心点坐标挪到左边，就能根据当前点位置计算中心点坐标  
然后再根据前面的公式和中心点坐标来计算终点坐标，而 large-arc-flag 和 sweep-flag 的参数都可以通过 swAng 算出，这样 SVG 命令所需的参数就凑齐了。  

**公式的实现**  
Office 中的公式使用自定义的 Office Math 格式，而在 Web 中原生支持的是 MathML 格式，因此需要做一下格式转换。  
安装 Windows 版本 Word 后可以在 C:\Program Files\Microsoft Office\root\Office16 目录下找到一个 OMML2MML.XSL 文件，因此就只需要使用浏览器自带的 XSLTProcessor 类做一下 XSL 转换，就能将 Office Math 转成 MathML 了，实现起来非常简单，因此顺带做了。  

**动态字段的实现**  
动态字段是指可以在 Word 中设置变量，最终渲染的时候根据前端上下文变量替换成对应的值。  
简单文本替换就不说了，这里面最麻烦的是图片替换，不能仅在渲染时替换，为了支持下载，还的将图片插入到 docx 文件里，然后更新 relation 来进行替换，尤其是对于循环图片的情况，需要插入多张图片。  
这是目前这个方案独有的功能，它可以让某个 Word 文档作为页面的一部分渲染，这样就能将部分页面开发工作交给非技术人员，真正做到只需要会用 Word 就能开发页面。

原文:  
[如何实现在 Web 中渲染 Word 文档](https://mp.weixin.qq.com/s/1X8tp7zdX-MVrwJPihA45g)
