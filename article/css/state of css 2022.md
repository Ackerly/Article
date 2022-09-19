# State of CSS 2022
## 2022 已经支持的特性
**@layer**  
解决业务代码的 !important 问题。为什么业务代码需要用 !important 解决问题？因为 css 优先级由文件申明顺序有关，而现在大量业务使用动态插入 css 的方案，插入的时机与 js 文件加载与执行时间有关，这就导致了样式优先级不固定。  
@layer 允许业务定义样式优先级，层越靠后优先级越高，比如下面的例子，override 定义的样式优先级比 framework 高：  
```
@layer framework, override;

@layer override {
  .title {
    color: white;
  }
}

@layer framework {
  .title {
    color: red;
  }
}
```

**subgrid**  
subgrid 解决 grid 嵌套 grid 时，子网格的位置、轨迹线不能完全对齐到父网格的问题。只要在子网格样式做如下定义：  
```
.sub-grid {
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}
```
**@container**  
@container 使元素可以响应某个特定容器大小。在 @container 出来之前，只能用 @media 响应设备整体的大小，而 @container 可以将相应局限在某个 DOM 容器内：  
```
// 将 .container 容器的 container-name 设置为 abc
.container {
  container-name: abc;
}

// 根据 abc 容器大小做出响应
@container abc (max-width: 200px) {
  .text {
    font-size: 14px;
  }
}
```
一个使用场景是：元素在不同的 .container 元素内的样式可以是不同的，取决于当前所在 .container 的样式  
**hwb**  
支持 hwb（hue, whiteness, blackness） 定义颜色：  
```
.text {
  color: hwb(30deg 0% 20%);
}
```
三个参数分别表示：角度 [0-360]，混入白色比例、混入黑色比例。角度对应在色盘不同角度的位置，每个角度都有属于自己的颜色值，这个函数非常适合直观的从色盘某个位置取色  

**lch, oklch, lab, oklab, display-p3 等**  
lch(lightness, chroma, hue)，即亮度、饱和度和色相，语法如：  
```
.text {
  color: lch(50%, 100, 100deg);
}
```
chroma(饱和度) 指颜色的鲜艳程度，越高色彩越鲜艳，越低色彩越暗淡。hue(色相) 指色板对应角度的颜色值  
oklch(lightness, chroma, hue, alpha)，即亮度、饱和度、色相和透明度  
```
.text {
  color: oklch(59.69% 0.156 49.77 / 0.5);
}
```
**color-mix()**  
css 语法支持的 mix，类似 sass 的 mix() 函数功能：  
```
.text {
  color: color-mix(in srgb, #34c9eb 10%, white);
}
```
第一个参数是颜色类型，比如 hwb、lch、lab、srgb 等，第二个参数就是要基准颜色与百分比，第三个参数是要混入的颜色  

**color-contrast()**  
让浏览器自动在不同背景下调整可访问颜色。换句话说，就是背景过深时自动用白色文字，背景过浅时自动用黑色文字：  
```
.text {
  color: color-contrast(black);
}
```
**相对颜色语法**  
可以根据语法类型，基于某个语法将某个值进行一定程度的变化：  
```
.text {
  color: hsl(from var(--color) h s calc(l - 20%));
}
```
如上面的例子，将 --color 这个变量在 hsl 颜色模式下，将其 l(lightness) 亮度降低 20%。  

**渐变色 namespace**  
渐变色也支持申明 namespace 了，比如：  
```
.text {
  background-image: linear-gradient(to right in hsl, black, white);
}
```
这是为了解决一种叫 灰色死区 的问题，即渐变色如果在色盘穿过了饱和度为 0 的区域，中间就会出现一段灰色，而指定命名空间比如 hsl 后就可以绕过灰色死区。  
这是为了解决一种叫 灰色死区 的问题，即渐变色如果在色盘穿过了饱和度为 0 的区域，中间就会出现一段灰色，而指定命名空间比如 hsl 后就可以绕过灰色死区。  
**accent-color**  
accent-color 主要对单选、多选、进度条这些原生输入组件生效，控制的是它们的主题色：  
```
body {
  accent-color: red;
}
```
比如这样设置之后，单选与多选的背景色会随之变化，进度条表示进度的横向条与圆心颜色也会随之变化。  

**inert**  
inert 是一个 attribute，可以让拥有该属性的 dom 与其子元素无法被访问，即无法被点击、选中、也无法通过快捷键选中：  
```
<div inert>...</div>
```

**COLRv1 Fonts**  
COLRv1 Fonts 是一种自定义颜色与样式的矢量字体方案，浏览器支持了这个功能，用法如下：  
```
@import url(https://fonts.googleapis.com/css2?family=Bungee+Spice);

@font-palette-values --colorized {
  font-family: "Bungee Spice";
  base-palette: 0;
  override-colors: 0 hotpink, 1 cyan, 2 white;
}

.spicy {
  font-family: "Bungee Spice";
  font-palette: --colorized;
}
```
上面的例子我们引入了矢量图字体文件，并通过 @font-palette-values --colorized 自定义了一个叫做 colorized 的调色板，这个调色板通过 base-palette: 0 定义了其继承第一个内置的调色板  
使用上除了 font-family 外，还需要定义 font-palette 指定使用哪个调色板，比如上面定义的 --colorized  
**视口单位**  
除了 vh、vw 等，还提供了 dvh、lvh、svh，主要是在移动设备下的区别：
- dvh: dynamic vh, 表示内容高度，会自动忽略浏览器导航栏高度。
- lvh: large vh, 最大高度，包含浏览器导航栏高度。
- svh: small vh, 最小高度，不包含浏览器导航栏高度。

**:has()**  
可以用来表示具有某些子元素的父元素：  
```
.parent:has(.child) {
}
```
表示选中那些有用 .child 子节点的 .parent 节点。  

## 即将支持的特性
**@scope**  
可以让某个作用域内样式与外界隔绝，不会继承外部样式：  
```
@scope (.card) {
  header {
    color: var(--text);
  }
}
```
如上定义后，.card 内 header 元素样式就被确定了，不会受到外界影响。如果我们用 .card { header {} } 来定义样式，全局的 header {} 样式定义依然可能覆盖它。  

**样式嵌套**  
@nest 提案时 css 内置支持了嵌套，就像 postcss 做的一样：  
```
.parent {
  &:hover {
    // ...
  }
}
```

**prefers-reduced-data**  
@media 新增了 prefers-reduced-data，描述用户对资源占用的期望，比如 prefers-reduced-data: reduce 表示期望仅占用很少的网络带宽，那我们可以在这个情况下隐藏所有图片和视频：  
```
@media (prefers-reduced-data: reduce) {
  picture,
  video {
    display: none;
  }
}
```
 也可以针对 reduce 情况降低图片质量，至于要压缩多少效果取决于业务

 **自定义 media 名称**  
允许给 @media 自定义名称了，如下定义了很多自定义 @media：  
```
@custom-media --OSdark (prefers-color-scheme: dark);
@custom-media --OSlight (prefers-color-scheme: light);

@custom-media --pointer (hover) and (pointer: coarse);
@custom-media --mouse (hover) and (pointer: fine);

@custom-media --xxs-and-above (width >= 240px);
@custom-media --xxs-and-below (width <= 240px);
```
可以按照自定义名称使用了：  
```
@media (--OSdark) {
  :root {
    …
  }
}
```
**media 范围描述支持表达式**  
只能用 @media (min-width: 320px) 描述宽度不小于某个值，现在可以用 @media (width >= 320px) 代替了  

**@property**  
@property 允许拓展 css 变量，描述一些修饰符:  
```
@property --x {
  syntax: "<length>";
  initial-value: 0px;
  inherits: false;
}
```
上面的例子把变量 x 定义为长度类型，所以如果错误的赋值了字符串类型，将会沿用其 initial-value  

**scroll-start**  
scroll-start 允许定义某个容器从某个位置开始滚动  
```
.snap-scroll-y {
  scroll-start-y: var(--nav-height);
}
```
**:snapped**  
:snapped 这个伪类可以选中当前滚动容器中正在被响应的元素：  
```
.card:snapped {
  --shadow-distance: 30px;
}
```
这个特性有点像 IOS select 控件，上下滑动后就像个左轮枪一样转动元素，最后停留在某个元素上，这个元素就处于 :snapped 状态。同时 JS 也支持了 snapchanging 与 snapchanged 两种事件类型  

**:toggle()**  
只有一些内置 html 元素拥有 :checked 状态，:toggle 提案是用它把这个状态拓展到每一个自定义元素：  
```
button {
  toggle-trigger: lightswitch;
}

button::before {
  content: "🌚 ";
}
html:toggle(lightswitch) button::before {
  content: "🌝 ";
}
```  
上面的例子把 button 定义为 lightswitch 的触发器，且定义当 lightswitch 触发或不触发时的 css 样式，这样就可以实现点击按钮后，黑脸与黄脸的切换。  

**anchor()**  
anchor() 可以将没有父子级关系的元素建立相对位置关系，更详细的用法可以看 CSS Anchored Positioning  

**selectmenu**  
selectmenu 允许将任何元素添加为 select 的 option：  
```
<selectmenu>
  <option>Option 1</option>
  <option>Option 2</option>
  <option>Option 3</option>
</selectmenu>
```
还支持更复杂的语法，比如对下拉内容分组：  
```
<selectmenu class="my-custom-select">
  <div slot="button">
    <span class="label">Choose a plant</span>
    <span behavior="selected-value" slot="selected-value"></span>
    <button behavior="button"></button>
  </div>
  <div slot="listbox">
    <div popup behavior="listbox">
      <div class="section">
        <h3>Flowers</h3>
        <option>Rose</option>
        <option>Lily</option>
        <option>Orchid</option>
        <option>Tulip</option>
      </div>
      <div class="section">
        <h3>Trees</h3>
        <option>Weeping willow</option>
        <option>Dragon tree</option>
        <option>Giant sequoia</option>
      </div>
    </div>
  </div>
</selectmenu>
```

原文:  
[精读《State of CSS 2022》](https://mp.weixin.qq.com/s/N33CBhVRwETgbtr3oSW-TA)