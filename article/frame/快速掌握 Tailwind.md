# 快速掌握 Tailwind
平时写 css 是这样的：
``` 
<div class="aaa"></div>

.aaa {
    font-size: 16px;
    border: 1px solid #000;
    padding: 4px;
}
```
在 html 里指定 class，然后在 css 里定义这个 class 的样式。也就是 class 里包含多个样式

而原子化 css 是这样的写法：
``` 
<div class="text-base p-1 border border-black border-solid"></div>

.text-base {
    font-size: 16px;
}
.p-1 {
    padding: 4px;
}
.border {
    border-width: 1px;
}
.border-black {
    border-color: black;
}
.border-solid {
    border-style: solid;
}
```
定义一些细粒度的 class，叫做原子 class，然后在 html 里直接引入这些原子化的 class。  
这个原子化 css 的概念还是很好理解的，但它到底有啥好处呢? 它解决了什么问题？  
通过 crerate-react-app 创建一个 react 项目：
``` 
npx create-react-app tailwind-test
```
然后进入 tailwind-test 目录，执行
``` 
npm install -D tailwindcss
npx tailwindcss init
```
tailwind 实际上是一个 postcss 插件，因为 cra 内部已经做了 postcss 集成 tailwind 插件的配置  
在入口 css 里加上这三行代码：
``` 
@tailwind base;
@tailwind components;
@tailwind utilities;
```
这三行分别是引入 tailwind 的基础样式、组件样式、工具样式的。之后就可以在组件里用 tailwind 提供的 class 了：  
``` 
import './App.css';

function App() {
  return (
    <div className='text-base p-1 border border-black border-solid'>guang</div>
  );
}

export default App;
```
p-1 是 padding:0.25rem，你也可以在配置文件里修改它的值  
.text-base 是 font-size、line-height 两个样式，这种通过数组配置  
也就是说所有 tailwind 提供的所有内置原子 class 都可以配置。  
但这些都是全局的更改，有的时候你想临时设置一些值，可以用 [] 语法。  
比如 text-[14px]，它就会生成 font-size:14px 的样式  
平时经常指定 hover 时的样式，在 tailwind 里怎么指定呢？
``` 
<div class='hover:text-[30px]'></div>
```
写响应式的页面的时候，我们要指定什么宽度的时候用什么样式，这个用 tailwind 怎么写呢？
``` 
<div class='md:bg-blue-500'></div>
```
光这些就很方便了, 之前要这么写：
``` 
<div class="aaa"></div>

.aaa {
    background: red;
    font-size: 16px;
}

.aaa:hover {
    font-size: 30px;
}

@media(min-width:768px) {
    .aaa {
        background: blue;
    }
}
```
省去了很多样板代码，还省掉了 class 的命名。并且这些 class 都可以通过配置来统一修改。  
多个 class 里都包含了类似的样式，但你需要写多次，而如果用了原子 class，就只需要定义一次就好了。  
css 没有模块作用域，所以可能你在这里加了一个样式，结果别的地方样式错乱了。而用原子 class 就没这种问题，因为样式是只是作用在某个 html 标签的。  
之前你要在 css、js 文件里反复跳来跳去的，查找某个 class 的样式是啥，现在不用这么跳了，直接在 html 里写原子样式，它不香么？  
而且 tailwindcss 就前面提到的那么几个语法，没啥学习成本，很容易看懂才对。  
但是还要每次去查文档哪些 class 对应什么样式呀,这个可以用 tailwind css 提供的 vscode 插件来解决  

**难以调试**  
在 chrome devtools 里可以直接看到有啥样式，而且样式之间基本没有交叉，很容易调试  
之前那种写法容易多个 class 的样式相互覆盖，还要确定优先级和顺序，那个更难调试才对  
类型太长了而且重复多次,这种问题可以用 @layer @apply 指令来扩展：
前面讲过 @tailwind 是引入不同的样式的，而 @layer 就是在某一层样式做修改和扩充，里面可以用 @apply 应用其他样式。  

**内置 class 不能满足我的需求**  
上面那个 @layer 和 @apply 就能扩展内置原子 class,但如果你想跨项目复用，那可以开发个 tailwind 插件  
``` 
const plugin = require('tailwindcss/plugin');

module.exports = plugin(function({ addUtilities }) {
    addUtilities({
        '.guang': {
            background: 'blue',
            color: 'yellow'
        },
        '.guangguang': {
            'font-size': '70px'
        }
    })
})
```
在 tailwind.config.js 里引入,这样就可以用这个新加的原子 class 了  
**ailwind 的 class 名和我已有的 class 冲突了咋办**  
这个可以通过加 prefix 解决,不过这样所有的原子 class 都得加 prefix 了  
tailwind 可以单独跑，也可以作为 postcss 插件来跑。这是因为如果单独跑的话，它也会跑起 postcss，然后应用 tailwind 的插件  
tailwind 本质上就是个 postcss 插件,postcss 是一个 css 编译器，它是 parse、transform、generate 的流程.  
而 postcss 就是通过 AST 来拿到 @tailwind、@layer、@apply 这些它扩展的指令，分别作相应的处理，也就是对 AST 的增删改查。  
它是怎么扫描到 js、html 中的 className 的呢？它有个 extractor 的东西，用来通过正则匹配文本中的 class，之后添加到 AST 中，最终生成代码。  
所以说，tailwind 就是基于 postcss 的 AST 实现的 css 代码生成工具，并且做了通过 extractor 提取 js、html 中 class 的功能。  
tailwind 还有种叫 JIT 的编译方式，这个原理也容易理解，本来是全部引入原子 css 然后过滤掉没有用到的，而 JIT 的话就是根据提取到的 class 来动态引入原子 css，更高效一点。


原文:  
[快速掌握 Tailwind：最流行的原子化 CSS 框架](https://mp.weixin.qq.com/s/Uur6erFJ9t9lxv5iK5KCOw)
