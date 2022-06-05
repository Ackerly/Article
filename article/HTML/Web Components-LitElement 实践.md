# Web Components-LitElement 实践
## LitElement介绍
**基本内容**  
Lit 的核心是一个组件基类，它提供响应式、scoped 样式和一个小巧、快速且富有表现力的声明性模板系统，且支持 TypeScript 类型声明。Lit 在开发过程中不需要编译或构建，几乎可以在无工具的情况下使用。  
HTMLElement 是浏览器内置的类，LitElement 基类则是 HTMLElement 的子类，因此 Lit 组件继承了所有标准 HTMLElement 属性和方法。更具体来说，LitElement 继承自 ReactiveElement，后者实现了响应式属性，而后者又继承自 HTMLElement。  
**定义一个组件**  
Lit 组件作为 Custom Element 的实现，并在浏览器中注册。  
原生的写法主要是继承 HTMLElement 类并重写它的方法。而 LitElement 框架则是基于 HTMLElement 类二次封装了 LitElement 类，它将很多的写法通过一些语法糖的封装变得更简单了，极大地简化了这些代码。开发者只需继承 LitElement 类开发自己的组件然后通过浏览器原生方法 customElements.define 注册即可。  
``` 
export class LitButton extends LitElement { /* ... */  }
customElements.define('lit-button', LitButton);
```
当定义一个 Lit 组件时，就是定义了一个自定义 HTML 元素。因此，可以像使用任何内置元素一样使用新元素。  
``` 
<lit-button type="primary"></lit-button>
```
**渲染**  
组件具有 render 方法，该方法被调用以渲染组件的内容。  
虽然 Lit 模板看起来像字符串插值，但 Lit 解析并创建一次静态 HTML，然后只更新表达式中需要更改的值。  
``` 
export class LitButton extends LitElement {
 /* ... */
 
 render() {
    // 使用模板字符串，可以包含表达式
    return html`
      <div><slot name="btnText"></slot></div>
    `;
  }
}
```
通常，组件的 render() 方法返回单个 TemplateResult 对象（与 html 标记函数返回的类型相同）。  
> TemplateResult对象：是 lit-html 接收模板字符串并经过它的 html 标记函数处理得到的一个纯值对象

它可以返回 Lit 可以渲染的任何内容，包括：  
- primitive 原始类型值，如字符串、数字或布尔值。
- 由 html 函数创建的 TemplateResult 对象。
- DOM 节点。
- 任何受支持类型的数组或可迭代对象。

**响应式 properties**  
DOM 中 property 与 attribute 的区别：  
- attribute 是 HTML 标签上的特性，可以理解为标签属性，它的值只能够是 String 类型，并且会自动添加同名 DOM 属性作为 property 的初始值
- property 是 DOM 中的属性，是 JavaScript 里的对象，有同名 attribiute 标签属性的 property 属性值的改变也并不会同步引起 attribute 标签属性值的改变

Lit 组件接收标签属性 attribute 并将其状态存储为 JavaScript 的 class 字段属性或 properties。响应式 properties 是可以在更改时触发响应式更新周期、重新渲染组件以及可选地读取或重新写入 attribute 的属性。每一个 properties 属性都可以配置它的选项对象。  
``` 
export class LitButton extends LitElement {
 // 在静态属性类字段中声明属性，Lit 会处理为响应式属性
  static properties = {
    type: {
      type: String,
      reflect: true,
      /*...其他选项属性...*/
    },
    other: {
      type: Object
    }
  };
  
  /* ... */
}
```
它的选项对象可以具有以下属性：  
- attribute：表示是否与 property 关联，或者 attribute 关联属性的自定义名称。默认值：true，表示 property 会与标签属性 attribute 进行关联。如果设置为 false，则下面的 converter 转换器、reflect 反射和 type 类型选项将被忽略。主要用来将 attribute 与 property 建立关联。
- type：在将 String 类型的 attribute 转换为 property 时，Lit 的默认属性转换器会将 String 类型解析为给定的类型。将 property 反映到 attribute 时反之亦然。如果设置了 converter 转换器，则将此字段传递给转换器。如果未指定类型，则默认转换器将其视为 String 类型。
- converter：用于在 attribute 和 property 之间转换的自定义转换器。如果未指定，则使用默认属性转换器。主要用来决定 attribute 与 property 确定建立关联后如何进行数据转换，毕竟 attribute 只能是 String 类型而 property 却是可以自定义的类型，默认属性转换器则是依据 property 配置的 type 选项进行目标类型的转换。上例中表示接受的 other 属性的 attribute 后会序列化为目标 Object 类型。  
- hasChanged：每当设置属性时调用的函数以确定属性是否已更改，并应触发更新。如果未指定，LitElement 将使用严格的不等式检查 (newValue !== oldValue) 来确定属性值是否已更改
- reflect：property 属性值是否反映回关联的 attribute 属性。默认值：false，即 property 的改变不会主动引起 attribute 的改变。上例中表示接收 type 组件属性 properties 的改动会同步到对应 attribute 标签属性上。  
- state：设置为 true 以将 property 属性声明为内部 state。内部 state 的改变也会触发更新，就像响应式属性 property，但 Lit 不会为其生成 attribute 属性，用户不应从组件外部访问它。这些属性应标记为 private 或 protected。还建议使用前导下划线 (_) 之类的约定来标识  JavaScript 用户的 private 或 protected 属性。可以为 state 内部状态指定的唯一选项是 hasChanged 函数。

省略选项对象或指定一个空的选项对象等效于为所有选项指定默认值。  
Lit 为每个响应式属性生成一个 getter/setter 对。当响应式属性发生变化时，组件会安排更新。Lit 也会自动应用 super 类声明的属性选项。除非需要更改选项，否则不需要重新声明该属性。  
**样式**  
组件模板被渲染到它的 shadow root。添加到组件的样式会自动作用于 shadow root，并且只会影响组件 shadow root 中的元素  
Shadow DOM 为样式提供了强大的封装。如果 Lit 没有使用 Shadow DOM，则必须非常小心不要意外地为组件之外的元素设置样式，无论是组件的父组件还是子组件。这可能涉及编写冗长而繁琐的类名。通过使用 Shadow DOM，Lit 确保编写的任何选择器仅适用于 Lit 组件的 shadow root 中的元素。  
可以使用标记的模板 css 函数在静态 styles 类字段中定义 scoped 样式。  
``` 
export class LitButton extends LitElement {
 // 使用纯 CSS 为组件定义 scoped 样式
  static styles = css`
    .lit-button {
      display: inline-block;
      padding: 4px 20px;
      font-size: 14px;
      line-height: 1.5715;
      font-weight: 400;
      border: 1px solid #1890ff;
      border-radius: 2px;
      background-color: #1890ff;
      color: #fff;
      box-shadow: 0 2px #00000004;
      cursor: pointer;
    }
  `;
  
  /* ... */
}
```
同样应用了 lit-button 样式，但样式只对 shodow root 中的部分起作用  
静态 styles 类字段的值可以是：  
- 单个标记的模板文字
    ```
     static styles = css`...`;
    ```
- 一组标记的模板文字。
    ```
    static styles = [ css`...`, css`...`];
    ```

styles 也支持在样式中使用表达式、使用语句、继承父类样式、共享样式、使用 unicode  escapes 以及在模板 template 中使用样式等功能。Lit 也提供了两个指令，classMap 和 styleMap，可以方便地在 HTML 模板中条件式的应用 class 和 style  
``` 
import {LitElement, html, css} from 'lit';
import {classMap} from 'lit/directives/class-map.js';
import {styleMap} from 'lit/directives/style-map.js';

export class LitButton extends LitElement {
  static properties = {
    classes: {},
    styles: {},
  };
  static styles = css`
   .lit-button {
      display: inline-block;
      padding: 4px 20px;
      font-size: 14px;
      line-height: 1.5715;
      font-weight: 400;
      border: 1px solid #1890ff;
      border-radius: 2px;
      background-color: #1890ff;
      color: #fff;
      box-shadow: 0 2px #00000004;
      cursor: pointer;
    }
    .someclass {
      color: #000;
    }
    .anotherclass {
      font-size: 16px;
    }
  `;

  constructor() {
    super();
    this.classes = {'lit-button': true, someclass: true, anotherclass: true};
    this.styles = {fontFamily: 'Roboto'};
  }
  render() {
    return html`
      <div class=${classMap(this.classes)} style=${styleMap(this.styles)}>
        <slot name="btnText"></slot>
      </div>
    `;
  }
}
customElements.define('lit-button', LitButton);
```
**生命周期**  
Lit 组件可以继承原生的自定义元素生命周期方法。但如果需要使用自定义元素生命周期方法，确保调用 super 类的生命周期，以保证父子组件生命周期的一致。  
标准的自定义组件生命周期  
- constructor()：创建元素时调用。适用于执行必须在第一次更新之前完成的一次性初始化任务
- connectedCallback()：在将组件添加到文档的 DOM 时调用。适用于仅在元素连接到文档时才发生的任务。其中最常见的是将事件侦听器添加到元素节点。
- disconnectedCallback()：当组件从文档的 DOM 中移除时调用，用于移除对元素的引用。比如移除添加到元素节点的事件侦听器
- attributeChangedCallback()：当元素的 observedAttributes 之一更改时调用
- adoptedCallback()：当组件移动到新文档时调用
``` 
 connectedCallback() {
  super.connectedCallback()
  addEventListener('keydown', this._handleKeydown);
}

disconnectedCallback() {
  super.disconnectedCallback()
  window.removeEventListener('keydown', this._handleKeydown);
}
```
除了标准的自定义元素生命周期之外，Lit 组件还实现了响应式更新周期。Lit 异步执行更新，因此属性更改是批处理的，如果在请求更新后但在更新开始之前发生了更多属性更改，则所有更改都将在同一个更新中进行。当响应式 prpperties 属性发生变化或显式调用 requestUpdate() 方法时，将触发响应更新周期，它会将更改呈现给 DOM。
**响应式更新周期**  
第一阶段：触发更新
- haschanged()：在设置响应式属性时隐式调用。默认情况下 hasChanged() 会进行严格的相等性检查，如果返回 true，则会安排更新。
- requestUpdate()：调用 requestUpdate() 来安排显式更新。如果需要在与属性无关的内容发生更改时更新和呈现元素，将很有用。
``` 
connectedCallback() {
  super.connectedCallback();
  this._timerInterval = setInterval(() => this.requestUpdate(), 1000);
}

disconnectedCallback() {
super.disconnectedCallback();
clearInterval(this._timerInterval);
}
```

第二阶段：执行更新  
- shouldUpdate()：调用以确定是否需要更新周期
- willUpdate()：在 update() 之前调用以计算更新期间所需的值。
- update()：调用以更新组件的 DOM
- render()：由 update() 调用，并应实现返回用于渲染组件 DOM 的可渲染结果（例如 TemplateResult）

第三阶段：完成更新
- firstUpdated()：在组件的 DOM 第一次更新后调用，紧接在调用 updated() 之前
- updated()：每当组件的更新完成并且元素的 DOM 已更新和呈现时调用
- updateComplete()：updateComplete Promise 在元素完成更新时更新为 resolved 状态

其他：  
- performUpdate()：调用 performUpdate() 以立即处理挂起的更新。这通常不需要，但在需要同步更新的极少数情况下可以这样做
- hasUpdated()：如果组件至少更新过一次，则 hasUpdated 属性返回 true。仅当组件尚未更新时，才可以在任何生命周期方法中使用 hasUpdated 来执行工作
- getUpdateComplete()：在执行 updateComplete 之前等待其他条件执行完成

**传入复杂数据类型**  
对于复杂数据的处理，为什么会存在这个问题，根本原因还是因为 attribute 标签属性值只能是 String 类型，其他类型需要进行序列化。在 LitElement 中，只需要在父组件模板的属性值前使用(.)操作符，这样子组件内部 properties 就可以正确序列化为目标类型。  
``` 
/**
 * 父组件-复杂数据类型
 */
 import { html, LitElement } from 'lit';
 import './person';

 class LitComplex extends LitElement {

  constructor() {
    super();
    this.person= {'name':'person'};
    this.friends = [{'name':'one'},{'name':'two'}];
  }

   render() {
     return html`
     <div>复杂数据类型</div>
     <lit-person .person=${this.person} .friends=${this.friends}></lit-person>
     `
   }
 }

 customElements.define('lit-complex', LitComplex);

 export default LitComplex;
```
``` 
/**
 * 基础组件
 */
 import { html, LitElement } from 'lit';

 class LitPerson extends LitElement {
   static properties = {
     person: {
       type: Object
     },
     friends: {
       type: Array,
     },
     date: {
       type: Date,
     }
   }

   firstUpdated() {
     console.log(this.person instanceof Object, this.friends instanceof Array, this.date instanceof Date); 
     // true true true
   }

   render() {
     return html`
     <div>${this.person.name}有${this.friends.length}个朋友</div>
     `
   }
 }

 customElements.define('lit-person', LitPerson);

 export default LitPerson;
```
这样可以支持各种类型数据的传递使用。  
**数据的双向绑定**  
``` 
/**
 * 数据绑定- father
 */
 import { html, LitElement } from 'lit';
 import './lit-input';

class LitInputFather extends LitElement {
  static properties = {
    data: {
      type: String
    }
  }

  constructor() {
    super();
    this.data = 'default';
  }

  render() {
    return html`
    <lit-input value=${this.data}></lit-input>
    `;
  }
}

customElements.define('lit-input-father', LitInputFather);

 export default LitInputFather;
```
``` 
/**
 * 数据绑定
 */
 import { html, LitElement } from 'lit';

 class LitInput extends LitElement {
   static properties = {
     value: {
       type: String,
       reflect: true
     }
   }

   change = (e) => {
     this.value = e.target.value;
   }

   render() {
     return html`
     <div>输入：<input value=${this.value} @input=${this.change}/></div>
     `
   }
 }

 customElements.define('lit-input', LitInput);

 export default LitInput;
```
这里子组件接收了父组件的 value 属性，默认值设为了 'default'，在子组件内通过监听输入事件更新了 value 值，因为 value 属性配置了 reflect 为 true，即可将属性值的改变反映回关联的 attribute 属性。  
如图：input 组件默认值为 'default'并在紧接着输入'123'后，组件的标签属性 value 同时发生了变化  
这时在父组件通过获取子组件的 attribute 即可获得子组件同步改动的值。以此实现数据的双向绑定，但 LitElement 本身是单向的数据流。  
**指令使用**  
指令是可以通过自定义表达式呈现方式来扩展 Lit 的函数。Lit 包含许多内置指令，可帮助满足各种渲染需求：以组件缓存为例  
在更改模板而不是丢弃 DOM 时缓存渲染的 DOM。在大型模板之间频繁切换时，可以使用此指令优化渲染性能  
``` 
/**
 * cache 内置指令使用
 */
 import {LitElement, html} from 'lit';
 import {cache} from 'lit/directives/cache.js';

 class LitCache extends LitElement {
  static properties = {
    show: false,
    data: {},
  };

  constructor() {
    super();
    this.data = {
      detail: 'detail',
      sumary: 'sumary'
    };
  }

  detailView = (data) => html`<div>${data.detail}</div>`;

  summaryView = (data) => html`<div>${data.sumary}</div>`

  changeTab = () => {
    this.show = !this.show;
  }

  render() {
    return html`${cache(this.show
      ? this.detailView(this.data)
      : this.summaryView(this.data)
    )}
    <button @click=${this.changeTab}>切换</button>
    `;
  }
}
customElements.define('lit-cache', LitCache);
```
这个例子在模板中使用了语句表达式，再通过 click 事件切换组件时展示不同的模板内容；引入了 cache 指令函数，实现了 DOM 的缓存  
LitElement 内置了大量的指令函数可以使用  

LitElement 在 Web Components 开发方面有着很多比原生的优势，它具有以下特点：  
- 简单：在 Web Components 标准之上构建，Lit 添加了响应式、声明性模板和一些周到的功能，减少了模板文件
- 快速：更新速度很快，因为 Lit 会跟踪 UI 的动态部分，并且只在底层状态发生变化时更新那些部分——无需重建整个虚拟树并将其与 DOM 的当前状态进行比较
- 轻便：Lit 的压缩后大小约为 5 KB，有助于保持较小的包大小并缩短加载时间
- 高扩展性：lit-html 基于标记的 template，它结合了 ES6 中的模板字符串语法，使得它无需预编译、预处理，就能获得浏览器原生支持，并且扩展能力强
- 兼容良好：对浏览器兼容性非常好，对主流浏览器都能有非常好的支持

参考:  
[Web Components-LitElement 实践](https://mp.weixin.qq.com/s/7lmx1ifHfhf3rpnwZEPc_g)
