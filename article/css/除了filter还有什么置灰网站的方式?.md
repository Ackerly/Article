# 除了filter还有什么置灰网站的方式?
通常而言，全站置灰是非常简单的事情，仅仅需要使用一行 CSS，就能实现全站置灰的方式。  
像是这样，我们仅仅需要给 HTML 添加一个统一的滤镜即可：  
``` 
html {
    filter: grayscale(.95);
    -webkit-filter: grayscale(.95);
}
```
又或者，使用 SVG 滤镜，也可以快速实现网站的置灰：  
``` 
<div>
// ...
</div>

<svg xmlns="https://www.w3.org/2000/svg">
  <filter id="grayscale">
    <feColorMatrix type="matrix" values="0.3333 0.3333 0.3333 0 0 0.3333 0.3333 0.3333 0 0 0.3333 0.3333 0.3333 0 0 0 0 0 1 0"/>
    </filter>
</svg>

html {
    filter: url(#grayscale);
}
```
大部分时候，这样都可以解决大部分问题。不过，也有一些例外。譬如，如果我们仅仅需要置灰网站的首屏，而当用户开始滚动页面的时候，非首屏部分不需要置灰  

## 使用 backdrop-filter 实现滤镜遮罩
**filter VS backdrop-filter**  
- filter：该属性将模糊或颜色偏移等图形效果应用于元素。
- backdrop-filter：该属性可以让你为一个元素后面区域添加图形效果（如模糊或颜色偏移）。它适用于元素背后的所有元素，为了看到效果，必须使元素或其背景至少部分透明。

注意两者之间的差异，filter 是作用于元素本身，而 backdrop-filter 是作用于元素背后的区域所覆盖的所有元素。而它们所支持的滤镜种类是一模一样的。  
backdrop-filter 最为常见的使用方式是用其实现毛玻璃效果。  

filter 和 backdrop-filter 使用上最明显的差异在于：
- filter 作用于当前元素，并且它的后代元素也会继承这个属性
- backdrop-filter 作用于元素背后的所有元素

仔细区分理解，一个是当前元素和它的后代元素，一个是元素背后的所有元素。  

## 使用 backdrop-filter 实现首屏置灰遮罩
借助 backdrop-filter 实现首屏的置灰遮罩效果：  
``` 
html {
    position: relative;
    width: 100%;
    height: 100%;
    overflow: scroll;
}
html::before {
    content: "";
    position: absolute;
    inset: 0;
    backdrop-filter: grayscale(95%);
    z-index: 10;
}
```

## 借助 pointer-events: none 保证页面交互
页面存在大量交互效果的，如果叠加了一层遮罩效果在其上，那这层遮罩下方的所有交互事件都将失效，譬如 hover、click 等。  
可以通过给这层遮罩添加上 pointer-events: none，让这层遮罩不阻挡事件的点击交互。  
代码如下：  
``` 
html::before {
    content: "";
    position: absolute;
    inset: 0;
    backdrop-filter: grayscale(95%);
    z-index: 10;
  + pointer-events: none;
}
```
> 注意:Firefox 的最新版本为 109，但是在 Firefox 103 之前，都是不支持 backdrop-filter 的。

## 借助混合模式实现网站置灰
除了 filter 和 backdrop-filter 外，CSS 中另外一个能对颜色进行一些干预及操作的属性就是 mix-blend-mode 和 background-blend-mode 了，翻译过来就是混合模式。  
backdrop-filter 的替代方案是使用 mix-blend-mode。  
``` 
html {
    position: relative;
    width: 100%;
    height: 100%;
    overflow: scroll;
    background: #fff;
}
html::before {
    content: "";
    position: absolute;
    inset: 0;
    background: rgba(0, 0, 0, 1);
    mix-blend-mode: color;
    pointer-events: none;
    z-index: 10;
}
```
叠加了一层额外的元素在整个页面的首屏，并且把它的背景色设置成了黑色 background: rgba(0, 0, 0, 1)，正常而言，我们的网站应该是一片黑色  
神奇的地方在于，通过混合模式的叠加，也能够实现网站元素的置灰。
实测：  
``` 
{
  mix-blend-mode: hue;            // 色相
  mix-blend-mode: saturation;     // 饱和度
  mix-blend-mode: color;          // 颜色
}
```
上述 3 个混合模式，叠加黑色背景，都是可以实现内容的置灰的。  
上述方法，需要给 HTML 设置一个白色的背景色，同时，不要忘记了给遮罩层添加一个 pointer-events: none。

原文:  
[除了 filter 还有什么置灰网站的方式？](https://mp.weixin.qq.com/s/pwXyZ-MAemaBhlPC6KM0hA)
