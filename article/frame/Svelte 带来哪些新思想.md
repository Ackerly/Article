# Svelte 带来哪些新思想
## Svelte 的优势
Svelte 翻译成中文就是“苗条”的意思，侧面表明它打包出来的包非常小.Svelte 主要优势有以下几点。  
**编译器**  
Svelte 是一种全新的构建用户界面的方法。传统框架如 React 和 Vue 在浏览器中需要做大量的工作，而 Svelte 将这些工作放到构建应用程序的编译阶段来处理。  
Svelte 组件需要在 .svelte 后缀的文件中编写，Svelte 会将编写好的代码翻编译 JS 和 CSS 代码。  
**打包体积更小**  
Svelte 在打包会将引用到的代码打包起来，而没引用过的代码将会被过滤掉，打包时不会加入进来  
在经过 gzip 压缩后生成的包大小，Svelte 打包出来的体积甩开 Vue、React 和 Angular 几条街。
这是因为经过 Svelte 编译的代码，仅保留引用到的部分。  
**不使用 Virtual DOM**  
Virtual DOM 就是 虚拟 DOM，是用 JS 对象描述 DOM 节点的数据，由 React 团队推广出来的。  
在 React 中实现数据驱动视图大概流程是这样的：  
``` 
数据发生变化 -> 通过diff算法判断要更新哪些节点 -> 找到要更新的节点 -> 更新真实DOM
```
Vue 的数据更新原理其实也差不多，只是实现方式和使用语法会有所不同。  
diff 算法 会根据数据更新前和更新后生成的虚拟 DOM 进行对比，只有两个版本的虚拟 DOM 存在差异时，才会更新对应的真实 DOM  
使用虚拟 DOM 对比的方式会比直接对比真实 DOM 的效率高  
而且真实 DOM 身上挂载的属性和方法非常多，使用虚拟 DOM 的方式去描述 DOM 节点树会显得更轻便  
每次数据发生变化时都要先创建一个虚拟 DOM，并使用 diff 算法 将新虚拟 DOM 与旧虚拟 DOM 进行比对，这个步骤会消耗一点性能和需要一点执行时间。  
而 Svelte 在未使用虚拟 DOM 的情况下实现了响应式设计  
Svelte 会监听顶层组件所有变量，一旦某个变量发生变化，就更新使用过该变量的组件。这就仅仅只需更新受影响的那部分 DOM 元素，而不需要整个组件更新。  
**更自然的响应式**  
现在流行的前端框架基本都使用 数据驱动视图 这个概念，像 Vue 和 React 这些框架，都有响应式数据的概念。  
Vue 和 React 在数据响应方面还是有点“不那么自然”，简单举几个例子：  
- 在 React 中，如果需要更新数据并在视图中响应，需要使用 setState 方法更新数据。
- 在 Vue2 中，响应式数据要放在 data 里，在 methods 中使用 this.xxx 来更新数据。
- 在 Vue3 的 Composition API 语法中，需要使用 ref 或者 reactive 等方法包裹数据，使用 xxx.value 等方式修改数据。

上面这几种情况，感觉多少都添加了点东西才能实现响应式数据功能  
在 Svelte 的理念中，响应式应该给开发者一种无感体验，比如在 Excel 中，当我规定 C1 单元格的值是 A1 + B1 的和，设置好规则后，用户只需要修改 A1 和 B1 即可，C1 会自动响应，而不需再做其他操作。  

``` 
<h1>{name}</h1>

<script>
  let name = '雷猴'

  setTimeout(() => {
    name = '鲨鱼辣椒'
  }, 1000)
</script>
```
上面的代码中，1 秒后修改 name 的值，并更新视图。  
从代码就能看出，在使用 Svelte 开发项目时，开发者一般无需使用额外的方法就能做到和 Vue、React 的响应式效果。  
**性能强**  
对比里多个热门框架的性能。从 Svelte 的性能测试结果可以看出，Svelte 是相当优秀的  
**内存优化**  
Svelte 对内存的管理做到非常极致，占用的内存也是非常小，这对于配置不高的设备来说是件好事  
**更关注无障碍体验**  
在使用 Svelte 开发时会 自动对无障碍访问方面的体验进行检测，比如 img 元素没有添加 alt 属性，Svelte 会向你发出一条警告。无障碍体验对特殊人事来说是很有帮助的,比如当你在 img 标签中设置好 alt 属性值，使用有声浏览器会把 alt 的内容读出来  

**Svelte 的不足**  
1. Svelte 对 IE 是非常不友好的，但我并不把这放在眼里。如果想兼容 IE 我还是推荐使用 jQuery。
2. Svelte 的生态不够丰富。由于是“新宠”，生态方面肯定是不如 Vue 和 React 的。

## 与 Svelte 相关的库  
**Sapper**  
Sapper 是构建在 Svelte 上的框架，Sapper 提供了页面路由、布局模板、SSR 等功能  
**Svelte Native**  
Svelte Native 是建立在 NativeScript之上的产物，可以开发安卓和 iOS 应用，是一个跨端技术，有点类似于 React Native 和 Weex 之类的东西。  
**svelte-gl**  
svelte-gl 还没正式发布，但这是个很有趣的工具，它和 three.js[38] 类似，专门做 3D 应用的

## 创建项目  
创建或使用开发环境有以下几种方式：  
1. REPL
2. Rollup 版
3. Webpack 版
4. Parcel 版
5. Vite 版

**REPL**  
REPL 是 Svelte 提供的一个线上环境，打开 Svelte 官网可以看到顶部导航栏上面有个 REPL的选项。点击该选项就可以跳转到 Svelte 线上开发环境了  
REPL 是 read(读取)、evaluate(执行)、print(打印) 和 loop(循环) 这几个单词的缩写。  
REPL 还提供了多组件开发，按左上角的 +号 可以创建新组件。组件的内容稍后会说到。  
界面右侧，顶部有 3 个选项：
- Result: 运行结果。
- JS output: Svelte 编译后的 JS 代码。
- CSS output: Svelte 编译后的 CSS 代码。

在 REPL 界面右上角还有一个下载按钮。当你在线上环境写好代码，可以点击下载按钮把项目保存到本地，下载的文件是一个 zip，需要自己手动解压。  
使用以下命令初始化项目并运行  
``` 
# 1、初始化项目
npm install

# 2、运行项目
npm run dev

# 3、在浏览器访问 http://localhost:5000
```
**Rollup 版**  
``` 
# 1、下载模板
npx degit sveltejs/template 项目名称

# 2、安装依赖
npm install

# 3、运行项目
npm run dev

# 4、在浏览器访问 http://localhost:8080
```
**Webpack 版**  
``` 
# 1、下载模板
npx degit sveltejs/template-webpack 项目名称

# 2、安装依赖
npm install

# 3、运行项目
npm run dev

# 4、在浏览器访问 http://localhost:8080/
```
**Parcel 版**  
不推荐使用 该方法创建项目，因为 Svelte 并没有提供使用 Parcel 打包工具的模板。但 GitHub 上有第三方的解决方案  
``` 
# 1、进入 `packages/svelte-3-example` 目录

# 2、安装依赖
npm install

# 3、运行项目
npm run start

# 4、在浏览器访问 http://localhost:1234/
```
**Vite 版**  
``` 
# 1、下载模板的命令
npm init vite@latest

# 2、输入项目名

# 3、选择 Svelte 模板（我没选ts）

# 4、进入项目并安装依赖
npm install

# 5、运行项目
npm run dev

# 6、在浏览器访问 http://127.0.0.1:5173/
```
## 起步
index.html 、src/main.js 和 src/App.svelte 这三个是最主要的文件。  
index.html 是项目运行的入口文件，它里面引用了 src/main.js 文件。  
src/main.js 里引入了 src/App.svelte 组件，并使用以下代码将 src/App.svelte 的内容渲染到 #app 元素里。  
``` 
const app = new App({
  target: document.getElementById('app')
})
```
target 指明目标元素  
**清空全局样式**  
如果使用 Rollup 版 创建项目，不需要做这一步。  
使用 Vite 创建的 Svelte 项目中，找到 src/app.css 文件，并把里面的内容清空掉  

**改造 src/App.svelte**  
将 src/App.svelte 文件改成以下内容  
``` 
<script>
  let name = '雷猴'

  function handleClick() {
    name = '鲨鱼辣椒'
  }
</script>

<div>Hello {name}</div>
<button on:click={handleClick}>改名</button>
```
上面的代码其实和 Vue 有点像  
- 变量和方法都写在 <script> 标签里。
- 在 HTML 中使用 {} 可以绑定变量和方法。
- 通过 on:click 可以绑定点击事件

只需写以上代码，Svelte 就会自动帮我们做数据响应的操作。一旦数据发生改变，视图也会自动改变。  

## 基础模板语法
**插值**  
``` 
<script>
  let name = '雷猴'
</script>

<div>{name}</div>
```
**表达式**  
``` 
<script>
  let name = '雷猴'

  function sayHi() {
    return `${name} 世界！`
  }

  let a = 1
  let b = 2

  let state = false
</script>

<div>{sayHi()}</div>

<div>{a} + {b} = {a + b}</div>

<div>{state ? '雷猴' : '鲨鱼辣椒'}</div>
```
**属性绑定**  
``` 
<script>
  let name = '雷猴'
</script>

<div title={name}>Hello</div>
```
**渲染 HTML 标签 @html**  
只是使用插值的方法渲染带有 HTML 标签的内容，Svelte 会自动转义 < 、> 之类的标签  
``` 
<script>
  let h1El = '<h1 style="color: pink;">雷猴</h1>'
</script>

<div>{h1El}</div>
```
Vue 中有 v-html 方法，它可以将 HTML 标签渲染出来。在 Svelte 中也有这个方法，在插值前面使用 @html 标记一下即可,但此方法有可能遭受 XSS 攻击。    
``` 
<script>
  let h1El = '<h1 style="color: pink;">雷猴</h1>'
</script>

<div>{@html h1El}</div>
```
## 样式绑定  
**行内样式 style**  
``` 
<script>
  let color = 'red'

  setTimeout(() => {
    color = 'blue'
  }, 1000)
</script>

<div style="color: {color}">雷猴</div>
```
**绑定 class**  
``` 
<script>
  let foo = true

  setTimeout(() => {
    foo = false
  }, 1000)
</script>

<div class:active={foo}>雷猴</div>

<style>
  .active {
    color: red;
  }
</style>
```
在 HTML 里可以使用 class:xxx 动态设置要激活的类。这里的 xxx 是对应的类名。   
语法是 class:xxx={state} ，当 state 为 true 时，这个样式就会被激活使用。  

## 条件渲染 #if 
**基础条件判断**  
``` 
<script>
  let state = true

  setTimeout(() => {
    state = false
  }, 1000)
</script>

{#if state}
  <div>雷猴</div>
{/if}
```
**两种条件**  
``` 
<script>
  let state = true

  setTimeout(() => {
    state = false
  }, 1000)
</script>

{#if state}
  <div>雷猴</div>
{:else}
  <div>鲨鱼辣椒</div>
{/if}
```
**多种条件**  
``` 
<script>
  let count = 1

  setInterval(() => {
    count++
  }, 1000)
</script>

{#if count === 1}
  <div>雷猴</div>
{:else if count === 2}
  <div>鲨鱼辣椒</div>
{:else}
  <div>蟑螂恶霸</div>
{/if}
```
## 列表渲染 #each  
**遍历数组**
``` 
<script>
  let list = ['a', 'b', 'c', 'd', 'e', 'f']
</script>

<ul>
  {#each list as item}
   <li>{item}</li>
  {/each}
</ul>
```
**遍历数组（带下标）**  
``` 
<script>
  let list = ['a', 'b', 'c', 'd', 'e', 'f']
</script>

<ul>
  {#each list as item, index}
   <li>{index} -- {item}</li>
  {/each}
</ul>
```
注意：as 后面首先跟着元素，然后才是下标。而且元素和下标不需要用括号括起来。  
**如果元素是对象，可以解构**  
``` 
<script>
  let list = [
    {name: '雷猴'},
    {name: '鲨鱼辣椒'}
  ]
</script>

<ul>
  {#each list as {name}}
   <li>{name}</li>
  {/each}
</ul>
```
**默认内容**  
如果源数据没有内容，是空数组的情况下，还可以组合 {:else} 一起使用。
```  
<script>
  let list = []
</script>

<div>
  {#each list as {name}}
   <div>{name}</div>
  {:else}
   <div>暂无数据</div>
  {/each}
</div>
```
## 事件绑定 on:event
使用 on: 指令监听 DOM 事件，on: 后面跟随事件类型,
``` 
<script>
  function sayHi() {
    console.log('雷猴')
  }
</script>

<button on:click={sayHi}>打招呼</button>
```
绑定其他事件（比如 change 等）也是同样的道理。
## 事件修饰符
你只希望某些事件只执行一次，或者取消默认行为，或者阻止冒泡等，可以使用事件修饰符。  
``` 
<script>
  function sayHi() {
    console.log('雷猴')
  }
</script>

<button on:click|once={sayHi}>打招呼</button>
```
除了 once 之外，还有以下这些修饰符可以用：   
- preventDefault ：禁止默认事件。在程序运行之前调用 event.preventDefault()
- stopPropagation ：调用 event.stopPropagation(), 防止事件到达下一个标签
- passive ：改善了 touch/wheel 事件的滚动表现（Svelte 会在合适的地方自动加上它）
- capture：表示在 capture阶段而不是bubbling触发其程序
- once ：程序运行一次后删除自身

**串联修饰符**  
修饰符还可以串联起来使用，比如 on:click|once|capture={...}  
但需要注意，有些特殊的标签使用修饰符会出现“意想不到”的结果，比如 <a> 标签
``` 
<script>
  function toLearn() {
    console.log('还在思考要不要学Canvas')
  }
</script>

<a
  href="https://juejin.cn/post/7116784455561248775"
  on:click|once|preventDefault={toLearn}
>去学习Canvas ？</a>
```
本来是想给 <a> 标签绑定一个点击事件，第一次点击时在控制台输出一句话，并且禁止 <a> 标签的默认事件。所以使用了 once 和 preventDefault 修饰符。  
但实际上并非如此。上面的代码意思是 once 设定了只执行一次 toLearn 事件，并且只有一次 preventDefault 是有效的。  
只有点击时就不触发 toLearn 了，而且 preventDefault 也会失效。所以再次点击时，<a> 元素就会触发自身的跳转功能。  

## 数据绑定 bind
语法：  
``` 
bind:property={variable}
```
**input 单行输入框**
``` 
<script>
  let msg = 'hello'

  function print() {
    console.log(msg)
  }
</script>

<input type="text" value={msg} />
<button on:click={print}>打印</button>
```
如果只是使用 value={msg} 的写法，input 默认值是 hello ，当输入框的值发生改变时，并没有把内容反应回 msg 变量里。此时就需要使用 bind 了
``` 
<input type="text" bind:value={msg} />
```
**textarea 多行文本框**  
``` 
<script>
  let msg = 'hello'
</script>

<textarea type="text" bind:value={msg} />
<p>{msg}</p>
```
**input range 范围选择**  
``` 
<script>
  let val = 3
</script>

<input type="range" bind:value={val} min=0 max=10 />
<p>{val}</p>
```
**radio 单选**  
单选框通常是成组出现的，所以要绑定一个特殊的值 bind:grout={variable}  
``` 
<script>
  let selected = '2'
</script>

<input type="radio" bind:group={selected} value="1" />
<input type="radio" bind:group={selected} value="2" />
<input type="radio" bind:group={selected} value="3" />
<p>{selected}</p>
```
**checkbox 复选框**  
``` 
<script>
  let roles = []
</script>

<input type="checkbox" bind:group={roles} value="雷猴" />
<input type="checkbox" bind:group={roles} value="鲨鱼辣椒" />
<input type="checkbox" bind:group={roles} value="蟑螂恶霸" />
<input type="checkbox" bind:group={roles} value="蝎子莱莱" />

<p>{roles}</p>
```
**select 选择器**  
``` 
<script>
  let selected = 'a'
</script>

<select bind:value={selected}>
 <option value='a'>a</option>
 <option value='b'>b</option>
 <option value='c'>c</option>
</select>

<span>{selected}</span>
```
**select multiple 选择器**  
``` 
<script>
  let selected = []
</script>

<select multiple bind:value={selected}>
 <option value="雷猴">雷猴</option>
 <option value="鲨鱼辣椒">鲨鱼辣椒</option>
 <option value="蟑螂恶霸">蟑螂恶霸</option>
 <option value="蝎子莱莱">蝎子莱莱</option>
</select>

<span>{selected}</span>
```
**简写形式**  
如果 bind 绑定的属性和在 JS 里声明的变量名相同，那可以直接绑定  
``` 
<script>
  let value = 'hello'
</script>

<input type="text" bind:value />

<p>{value}</p>
```
bind:value 绑定的属性是 value ，而在 JS 中声明的变量名也叫 value ，此时就可以使用简写的方式  

## $: 声明反应性
> 通过使用$: JS label 语法作为前缀。可以让任何位于 top-level 的语句（即不在块或函数内部）具有反应性。每当它们依赖的值发生更改时，它们都会在 component 更新之前立即运行

``` 
<script>
  let count = 0;
  $: doubled = count * 2;

  function handleClick() {
    count += 1;
  }
</script>

<button on:click={handleClick}>
  点击加1
</button>

<p>{count} 翻倍后 {doubled}</p>
```
使用 $: 声明的 double 会自动根据 count 的值改变而改变。  
如果将以上代码中 $: 改成 let 或者 var 声明 count ，那么 count 将失去响应性。  

## 异步渲染 #await
语法：  
``` 
{#await expression}
...
{:then name}
...
{:catch name}
...
{/await}
```
以 #await 开始，以 /await 结束。  
:then 代表成功结果，:catch 代表失败结果。  
expression 是判断体，要求返回一个 Promise  

``` 
<script>
  const api = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('请求成功，数据是xxxxx')
    }, 1000)
  })
</script>

{#await api}
  <span>Loading...</span>
{:then response}
  <span>{response}</span>
{:catch error}
  <span>{error}</span>
{/await}
```
如果将上面的 resolve 改成 reject 就会走 :catch 分支。  
## 基础组件
在 Svelte 中，创建组件只需要创建一个 .svelte 为后缀的文件即可。通过 import 引入子组件。
比如，在 src 目录下有 App.svelte 和 Phone.svelte 两个组件。App.svelte 是父级，想要引入 Phone.svelte 并在 HTML 中使用。  
App.svelte  
``` 
<script>
  import Phone from './Phone.svelte'
</script>

<div>子组件 Phone 的内容：</div>
<Phone />
```
Phone.svelte  
``` 
<div>电话：13266668888</div>
```
## 组件通讯
**父传子**  
手机号希望从 App.svelte 组件往 Phone.svelte 里传。可以在 Phone.svelte 中声明一个变量，并公开该变量。App.svelte 就可以使用对应的属性把值传入。  
App.svelte  
``` 
<script>
  import Phone from './Phone.svelte'
</script>

<div>子组件 Phone 的内容：</div>
<Phone number="88888888" />
```
Phone.svelte  
``` 
<script>
  export let number = '13266668888'
</script>

<div>电话：{number}</div>
```
如果此时 App.svelte 组件没有传值进来，Phone.svelte 就会使用默认值。  
**子传父**  
如果想在子组件中修改父组件的内容，需要把修改的方法定义在父组件中，并把该方法传给子组件调用. 同时需要在子组件中引入 createEventDispatcher 方法。  
App.svelte
``` 
script>
  import Phone from './Phone.svelte'
  function print(data) {
    console.log(`手机号: ${data.detail}`)
  }
</script>

<div>子组件 Phone 的内容：</div>
<Phone on:printPhone={print} />
```
Phone.svelte  
``` 
<script>
  import { createEventDispatcher } from 'svelte'
  const dispatch = createEventDispatcher()

  function printPhone() {
    dispatch('printPhone', '13288888888')
  }
</script>

<button on:click={printPhone}>输出手机号</button>
```
父组件接受参数是一个对象，子组件传过来的值都会放在 detail 属性里。  

## 插槽 slot
App.svelte  
``` 
<script>
  import Phone from './Phone.svelte'
</script>

<div>子组件 Phone 的内容：</div>
<Phone>
  <div>电话：</div>
  <div>13288889999</div>
</Phone>
```
Phone.svelte  
``` 
<style>
 .box {
  width: 100px;
  border: 1px solid #aaa;
  border-radius: 8px;
  box-shadow: 2px 2px 8px rgba(0,0,0,0.1);
  padding: 1em;
  margin: 1em 0;
 }
</style>

<div class="box">
 <slot>默认值</slot>
</div>
```
## 生命周期
Svelte 中主要有以下几个生命周期：
- onMount: 组件挂载时调用。
- onDestroy: 组件销毁时执行。
- beforeUpdate: 在数据更新前执行。
- afterUpdate: 在数据更新完成后执行。
- tick: DOM 元素更新完成后执行。

``` 
<script>
  import { onMount } from 'svelte'
  let title = 'Hello world'

  onMount(() => {
    console.log('onMount')
    setTimeout(() => title = '雷猴', 1000)
  })
</script>

<h1>{title}</h1>
```
在组件加载完 1 秒后，改变 title 的值。  
onDestroy、beforeUpdate 和 afterUpdate 都和 onMount 的用法差不多，只是执行的时间条件不同。你可以自己创建个项目试试看。  
tick 是比较特殊的，tick 和 Vue 的 nextTick 差不多。  
在 Svelte 中，tick 的使用语法如下：  
``` 
import { tick } from 'svelte'

await tick()
// 其他操作
```

原文: 
[前端新宠 Svelte 带来哪些新思想？赶紧学起来！](https://mp.weixin.qq.com/s/5o7qiDC_BGIq6n0FWHvClw)
