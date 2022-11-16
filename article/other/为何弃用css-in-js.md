# 为何弃用css-in-js
**css-in-js 的优缺点**  
优点：
1. 无全局样式冲突。就像 js 文件天然支持模块化的好处一样，原生 css 因为没有模块化能力，天然容易导致全局样式污染，如果不是特意用 BEM 方式命名，想要避免冲突就只能借助 css-in-js 了。（css-modules 也一样能做到）
2. 与 js 代码合在一起。天然融合进 js 代码方便模块化管理，使 css 可以与某个局部模块绑定。（css-modules 也一样能做到，只是必须单独拆一个样式文件）
3. 能将 js 变量应用到样式上。虽然 css 变量也能解决这个问题，但不如 css-in-js 那么直观，inline-style 也能解决这个问题，但会产生大量重复的局部样式，且这个优势 css-modules 做不到

缺点：
1. css-in-js 运行时解析的实现版本增加了运行时性能压力，尤其在 React18 调度机制模式下，存在无法解决的性能问题（运行时插入样式会导致 React 渲染暂停，浏览器解析一遍样式，渲染再继续，然后浏览器又解析一遍样式）。
2. 增加了包体积。相比原生或者 css-modules 方案来说，增加了运行时框架代码 8kb 左右
3. 让 ReactDevTools 结构变得复杂，因为 css-in-js 会包裹额外的 React 组件层用来实现样式插入

css-in-js 还有三点深度使用后才能察觉的坑：  
1. 多个不同（甚至是相同）版本的 css-in-js 库同时加载时可能导致错误。笔者用 styled-components 就遇到了类似问题，甚至语法会产生不兼容的情况，虽然这些问题都可以被解决，但花费的额外时间需要计算一样，相比 css-in-js 得到的收益是否值得
2. 样式插入优先级无法自定义，这就导致产生样式覆盖时，业务对样式覆盖的优先级无法产生稳定的预期。class 优先级由 header 定义顺序决定，而非 className 的字符顺序决定，而 header 定义顺序又由资源加载与 css-in-js 插入执行时机决定，导致业务几乎不可能有稳定的样式覆盖顺序。这里产生的问题就是业务代码不断增多的 !impprtant 定义
3. 不同 React 版本的 SSR，css-in-js 需要适配不同的实现，这对框架作者不太友好。

**无解的性能问题**  
第一条缺点提到的运行时解析，是 css-in-js 方案永远跨不过去的困境，即便对于编译时 css-in-js 方案来说，也免不了在渲染时做额外的逻辑执行拖慢渲染速度：  
```
 function App() {
  return <div css={{ color: "red" }} />; // 就是这种代码导致了性能问题
}
```
原因是当 React 重渲染组件时，需要重新解析样式定义，并序列化 className，当渲染非常频繁时会导致明显的性能瓶颈，而解决方法是把样式定义抽出来，但这样就损失了第三个优点，即无法读取 js 变量了：  
```
 const myCss = css({
  backgroundColor: "blue",
  width: 100,
  height: 100,
});
```
不得不的说 React 的渲染机制实在是太有问题了，如果换成 SolidJS 这个问题就好办了，因为运行时的样式代码仅会运行一次，组件重渲染也不会导致这段解析代码被重复执行，此时 css-in-js 在样式变化时再做一次精确样式更新，性能问题就可以被解决了  

**换成 css modules**  
css-modules 同时支持优点一和二，而优点三可以通过一些特定语法糖绕过：通过 :import :export 伪类做 css 变量的导入导出，用 webpack-loader 实现 js 中引用 css 变量，用 css variable 实现 css 引用 js 变量。  
所以当性能问题是绕不过去的话题，而 css-modules 在性能最优的情况下，有一些曲线方案可以同时支持 css-in-js 的优点，也就能理解为什么作者要弃用 css-in-js 了。  

**包体积真的变大了吗**  
原文谈到的 css-in-js 增加了 8~16kb 其实是在强行堆缺点了，除非你的项目只有一行 css 定义。如果我们只考虑传输时的包体积与 HTML 中样式定义数量，而忽略运行时产生的性能负担，那么 css-in-js 在大型项目无疑是最优的  
原因就是 css-in-js 样式是按需插入的，没有渲染的组件就不会插入样式。甚至渲染了的组件也不一定会插入样式，因为 css-in-js 可以对包含相同样式定义的场景做 className 合并，类似于 webpack 打包时，可以把不同模块公共代码抽到一个 chunk 里  
**编译时 css-in-js 方案是出路吗**  
理论上是出路，但限制了 css-in-js 的灵活性。从 vanilla-extract 等编译时 css-in-js 框架来看，确实解决了运行时 css-in-js 性能问题，但带来了更多语法限制，比如必须预先定义样式再使用：   
```
import { style } from '@vanilla-extract/css'

const myStyle = style({
  display: 'flex',
  paddingTop: '3px'
})

const App = () => <div className={myStyle}/>
```
编译时 css-in-js 想要做到通用性，只能提供一个 className，这样就不受任何框架和环境的限制了，但这样也限制了声明语法的灵活性，显然不可以用内连方式定义样式。  
这种编译时的方案本质上和 css-modules 是一样的，背后都是定义了一些静态样式名，只是说这些样式问题以 .sass 定义还是 .ts 定义，如果用 .ts 定义，配合编译工具可以使代码原生 import 的更加舒服。  
所以使用了编译时 css-in-js 方案，本质上还是抛弃了运行时 css-in-js，投向了变种的 css-modules 阵营。


原文:  
[精读《我们为何弃用 css-in-js》](https://mp.weixin.qq.com/s/SQZvAguGydMnFDBqf4U9uA)