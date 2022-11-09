# 配个 json，表单就出来了？FormRender 初探
FormRender有什么优点?  
1. 支持 Ant Design 和 Fusion Design 主流的视觉主题
2. 使用 JSON Schema 标准协议描述表单配置，并搭配丰富类型且可扩展的组件
3. 支持 1 排 N、支持对象无限嵌套、自定义正则校验、自定义样式组件、列表拖拽等特性
4. 已在飞猪、亚博科技、安全智能、淘宝、新零售行业工作台、人工智能实验室、天猫等多场景使用，可支持复杂场景使用
5. 维护上有专人支持

遇到这种灵活多变的表单配置化需求，一个输入框和select都甚至需要封装成一个组件，写完一个组件，就要在components增加一个文件夹，要是用了这个插件，一些基础的组件可以直接用，要是不满足你的需求的话，也可以自定义组件  
插件的缺点:  
插件诞生也就两三年，可能会有一些bug，这是难免的 文档不是很全  


## 设计方案
协议层定义协议（schema）配置，展示层控制协议的渲染，工具层提供上下游的进一步支持。在此之上，FR 遵循如下的 api 设计：  
1. 基于 JSON schema 的协议规范。JSON schema 作为 JSON 数据校验表述的国际标准，主要用于表单数据的服务端校验。已经接入JSON Schema标准的团队可以几乎无缝接入 FR
2. 极简的组件 api:  FormRender 使用上类似一个单独的可控的 input：
   ``` 
    // 可控的input
    <input value={value} onChange={onChange} />
    // form-render, 只多了schema，用于描述 Form 长什么样
    <FormRender formData={value} onChange={onChange} schema={schema} />
   ```
   这样的设计下，FR 只负责管理和改动表单数据/时时校验，而将具体如何使用表单数据和校验信息乃至提交的方式全权交给了使用者自由书写。
3. 支持复杂联动:  schema 的大部分属性都支持函数表达式，实现了灵活但强大的联动效果。
    ```
     "showMore": {
        "title": "显示更多",
        "type": "boolean"
        },
        "input1": {
        "title": "输入框1",
        "type": "string",
        "hidden": "{{rootValue.showMore === false}}" // 当showMore值为false时，隐藏
    }。
    ```
4. 支持个性化扩展： 当出现现有表单元素无法满足需求的场景，FR 使用自定义组件的方式，让用户自由扩展 FR 的组件库  

> 自定义组件就是普通的React组件，唯一的 要求是要有 value/onChange 这两个 props，用于双向绑定值。所以如果现成的组件已经默认使用了  value/onChange,就可以直接拿来用。

```
// 写自定义组件
const MyInput = ({ value, onChange }) => {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
};
// 传入自定义组件
<FormRender {...props} widgets={{ myInput: MyInput }} />;
```
协议上只需指明"widget": "MyInput"，即可使用对应的组件来渲染：  
```
text: {
  title: "你好",
  type: "string",
  "widget": "myInput"
}
```
基于 JSON schema 的协议确保了 FR 的规范性，可控组件的模型确保了外层 api 的简洁和解耦，联动&自定义组件的 api 确保了对大量复杂的场景的很好支持。  

## 使用体验 & 流程
**安装：**  
FormRender 依赖 ant design，单独使用不要忘记同时安装 antd  
```
npm i form-render --save
```
最简demo
```
import React from 'react';
import { Button } from 'antd';
import FormRender, { useForm } from 'form-render';

const schema = {
  type: 'object',
  properties: {
    input1: {
      title: '简单输入框',
      type: 'string',
      required: true,
    },
    select1: {
      title: '单选',
      type: 'string',
      enum: ['a', 'b', 'c'],
      enumNames: ['早', '中', '晚'],
    },
  },
};

const Demo = () => {
  const form = useForm();
  const onFinish = (formData, errors) => {
    console.log('formData:', formData, 'errors', errors);
  };
  return (
    <div>
      <FormRender form={form} schema={schema} onFinish={onFinish} />
      <Button type="primary" onClick={form.submit}>
        提交
      </Button>
    </div>
  );
};

export default Demo;
```
从 demo 中我们不难发现 FormRender 的一些设计：  
- 以 schema 来描述表单展示，提交方式与 antd v4 的方式类似
- schema 以国际标准的 JSON schema 为基础，同时能够方便使用任何 antd 的 props
- 通过 bind 字段，我们允许数据的双向绑定，数据展示和真实提交的数据可以根据开发需求不同（例如从服务端接口拿到不规则数据时，也能直接使用）
- 使用 {{...}} 书写表达式来完成简单的联动，值得一提的是，这里表达式支持所有 js 语法。FR 还提供自定义组件、dependencies 声明、watch 等工具用于更加复杂的定制
- 可以通过 displayType，labelWidth 等字段轻易修改展示

通过使用 schema 编辑器，生成一个表单的流程非常简单
1. 使用 在线 schema 编辑器 搭建表单，导出 schema
2. 将 schema 作为props传入<FormRender {...schema} />组件 在线（demo中替换 schema.json 文件内容即可）

在封装这个自定义组件时遇到的一个小坑：  
组件只挂载了一次，导致给自定义组件传的recycleLinkCode，刚开始拿到的初始值是空，但是组件已经挂载完了，等到接口返回recycleLinkCode的值时，传过去的数据没有实时更新。  
官方给出的理由是： formData的更新才会触发表单的重新渲染，但是recycleLinkCode不是表单值，所以不能用form.setValues和form.setValueByPath来修改表单值  
解决办法：  
1. 接口拿到数据之后再去加载组件
2. 触发组件重新渲染，可以试试这样写：useMemo(()=>,[recycleLinkCode])

原文:  
[配个 json，表单就出来了？FormRender 初探](https://mp.weixin.qq.com/s/bEaL-TYR0aNMwWZWYgXnWA)
