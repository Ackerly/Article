# 自动化生成Vue组件文档
## 概述
当一个多人开发的Vue项目经过长期维护之后往往会沉淀出很多的公共组件，经常会出现一个人 开发了一个组件而其他维护者或新接手的人却不知道这个组件是做什么的、该怎么用，还必须得再去翻看源码，或者压根就没注意到这个组件 的存在导致重复开发。这个时候就非常需要维护对应的组件文档来保障不同开发者之间良好的协作关系了。  
传统手动维护文档带来的问题:
- 效率低，组件开发完了回头还要写文档，
- 效率低，写文档是个费时费力的体力活，好不容易抽时间把组件开发完了回头还要写文档，
- 不智能，组件更新迭代的同时，需要手动将变更同步到文档中，消耗时间还容易遗漏。

理想中的文档维护方式：  
- 工作量小，能够结合Vue组件自动获取相关信息，减少从头开始写文档的工作量。
- 信息准确，组件的关键信息与组件内容一致，不出错
- 智能同步，Vue组件迭代升级时，文档内容可以自动的同步更新，无需人工校验信息是否一致。
## 社区解决方案
**业务梳理**  
Vue官方提供了Vue-press可以用于快速搭建Vue项目文档， 而且也已经有了可以自动从Vue组件中提取信息的库了。 
已有的第三方库并不能完全满足需求，主要存在以下两个问题:
- 信息不全面，一些重要内容无法获取例如不能处理v-model，不能解析属性的修饰符sync，不能获取methods中函数入参的详细信息等。
- value属性与input事件可以合起来构成一个v-model属性，但是这个信息在生成的文档中没有体现出来，要文档读者自行理解判断。而且生成的文档中没有展示是否支持sync。  

有较多的自定义标识，而且标识的命名过于个性化，对原有的代码侵入还是比较大的。为了标记注释，需要在原有的 业务代码中额外添加"@vuese" "@arg"等标识，使得业务代码多出了一些业务无关内容。

## 技术方案
### Vue文件解析
官方开发了Vue-template-compiler库专门用于Vue解析，Vue-template-compiler提供了一个parseComponent方法可以对原始的Vue文件进行处理。
``` 
import { parseComponent } from 'Vue-template-compiler'
const result = parseComponent(VueFileContent, [options])
```
处理后的结果如下，其中template和script分别对应Vue文件中的template和script的文本内容。
``` 
export interface SFCDescriptor {
  template: SFCBlock | undefined;
  script: SFCBlock | undefined;
  styles: SFCBlock[];
  customBlocks: SFCBlock[];
}
```
得到script后，我们可以用babel把js编译成js的AST（抽象语法树），这个AST是一个普通的js对象，可以通过js进行遍历和读取 有了Ast之后我们就可以从中获取到我们想到详细的组件信息了。
``` 
import { parse } from '@babel/parser';
const jsAst = parse(script, [options]);
```
compile将template编译成AST
``` 
export interface CompiledResult {
  ast: ASTElement,
  render: string,
  staticRenderFns: Array<string>,
  errors: Array<string>
}
```
### 信息提取
信息可以分为两种：
- 一种是可以直接从Vue组件中获取，例如props、events等。
- 另一种是需要额外约定格式的，例如：组件的说明注释，props的属性说明等，这部分可以放到注释里，通过对注释进行解析获取。  

@babel/traverse库是babel官方提供的专门用于遍历js AST的。使用方式如下:
``` 
import traverse from '@babel/traverse'
traverse(jsAst, options);
```
通过在options中配置对应内容的回调函数，可以获得想要的ast节点。  
**可直接获取的信息**  
可以直接获取的信息有：
- 组件属性props
- 提供外部调用的方法methods
- 事件events
- 插槽slots

1、2都可以利用traverse在js AST上直接遍历名称为props和methods的对象节点获取。  
事件的获取稍微麻烦一点，可以通过查找$emit函数来定位到事件的位置，而$emit函数可以在traverse中监听MemberExpress(复杂类型节点)， 然后通过节点上的属性名是否是'$emit'判断是否是事件。如果是事件，那么在$emit父级中读取arguments字段， arguments的第一个元素就是事件名称，后面的元素为事件传参。  
``` 
this.$emit('event', arg);
traverse(jsAst, {
 MemberExpression(Node) {
  // 判断是不是event
  if (Node.node.property.name === '$emit') {
  // 第一个元素是事件名称
    const eventName = Node.parent.arguments[0];
  }
 }
});
```
在成功获取到Events后，那么结合Events和props，就可以进一步的判断出props中的两个特殊属性：  
是否存在v-model：查找props中是否存在value属性并且Events中是否存在input事件来确定。  
props的某个属性是否支持sync：判断Events的时间名中是否存在有update开头的事件，并且事件名称与属性名相同。  
插槽slots的信息保存在上文的template的AST中，递归遍历template AST找到名为slots的节点，进而还可以在节点上查找到name。  
**需要约定的信息**
可以约定的内容有以下几条：
- 组件名称
- 组件的整体介绍
- props、Events、methods、slots文字说明
- Methods标记和入参的详细说明。这些内容都可以放在注释中进行维护，之所以放在注释中进行维护是因为注释可以很容易从上文提到的js AST以及template AST中获取到， 解析Vue组件信息的同时就可以把这部分针对性的说明一起解析到。

Methods标记和入参的详细说明。这些内容都可以放在注释中进行维护，之所以放在注释中进行维护是因为注释可以很容易从上文提到的js AST以及template AST中获取到， 在我们解析Vue组件信息的同时就可以把这部分针对性的说明一起解析到。  
``` 
// 头部注释
export default {} // 尾部注释
```
解析结果
``` 
const exportNode = {
  type: "ExportDefaultDeclaration",
  leadingComments: [{
    type: 'CommentLine',
    value: '头部注释'
  }],
  trailingComments: [{
    type: 'CommentLine',
    value: '尾部注释'
  }]
}
```
在同一个位置上，根据注释格式的不同又分为单行注释(CommentLine)和块级注释(CommentBlock)，两种注释的区别会反应在注释节点的type字段中：
``` 
/**
 * 块级注释
 */
// 单行注释
export default {}
```
解析结果
``` 
const exportNode = {
  type: "ExportDefaultDeclaration",
  leadingComments: [
    {
      type: 'CommentBlock',
      value: '块级注释'
    },
    {
      type: 'CommentLine',
      value: '单行注释'
    }
  ]
}
```
从上面的解析结果我们也可以看到，注释节点是挂载在被注释的export节点里面的  
template中注释节点与其他节点一样是作为dom节点存在的， 在遍历节点的时候通过判断isComment字段的值是否为true来确定是否是注释节点。而被注释的内容的位置在兄弟节点的后一位：
``` 
<!--template的注释-->
<slot>被注释的节点</slot>
```
解析结果
``` 
const templateAst = [
  {
    isComment: true,
    text: "template的注释",
    type: 3
  },
  {
    tag: "slot",
    type: 1
  }
]
```
通过在methods的方法的注释中约定一个标记@public来区分是私有方法还是公共方法，如果更细节一点的话， 还可以参考另一个专门用于解析js注释的库js-doc的格式，对方法的入参进行更进一步的说明，丰富文档的内容。  
只需要在获取到注释内容之后对文本进行切割读取即可，例如：
``` 
export default {
  methods: {
    /**
     * @public
     * @param {boolean} value 入参说明
     */
    show(value) {}
  }
}
```
当然了为了避免对代码侵入过多，我们还是需要尽量少的添加额外的标识。而入参说明采用了与js-doc相同的格式，主要还是因为这套方案 使用比较普遍，而且代码编辑器都自动支持方便编辑。


参考:
[解放生产力，自动化生成Vue组件文档](https://mp.weixin.qq.com/s/-LcNL_CVZWYCeVg9Xb6Rvg)
