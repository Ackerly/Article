# CSS长度单位
**px**  
相对于屏幕分辨率而不是视窗大小，通常为1个点或1/72英寸；  
**em**  
 em 参照的并不是父元素的 font-size，而是参照的当前元素的 font-size;
然而当前元素没有设置 font-size 的时候，当前元素的 font-size 会继承自父元素的 font-size 值，所以会造成一种 em 是参照了父元素的 font-size 的错觉。  
``` 
// html
<div class="parent">
    <div class="son"></div>
</div>

// css
.parent {
    font-size: 20px;
}

.son {
    // 当前 font-size 没有设置则会继承祖先元素的 font-size
    // 则 2em = 2 * 20px = 40px
    text-indent: 2em;
}

```
**rem**  
rem 作用于"非根元素(除 html 标签外的其它标签)"时，计算值是参照于"根元素"字体大小；rem 作用于"根元素"字体大小时，相对于其出初始字体大小。  
rem 最常见的用法就是用在移动端适配当中。
``` 
html {
    // Chrome 中 html 初始字号大小是 16px, 所以这里 2rem = 32px
    font-size: 2rem;
}

// 或者  
html {
    font-size: 100px;
}

p {
    // 100px * 0.5 = 50px
    font-size: 0.5rem;
    // 100px * 0.75 = 75px
    line-height: 0.75rem;
}
```
**百分比**  
以百分比为长度单位的可以分为三类：  
1. 在 font-size 属性中使用百分比，其中的 font-size 的值会是其父元素的 font-size 值乘以百分比。
2. 要计算 height, top 及 bottom 中的百分值，是通过包含块的 height 的值。如果包含块的 height 值会根据它的内容变化，而且包含块的 position 属性的值被赋予 relative 或 static ，那么，这些值的计算值为 auto。
3. 要计算 width, left, right, padding, margin 这些属性由包含块的 width 属性的值来计算它的百分值。

``` 
<div class="parent">
    <div class="son"></div>
</div>

.parent {
    width: 100px;
    height: 88px;
    font-size: 20px;
    background: #eee;
}

.son {
    // 相对于父元素的 font-size, 即 20px * 200% = 40px
    font-size: 200%;
    // 相对于父元素的 width, 即 100px * 30% = 30px
    width: 30%;
    // 相对于父元素的 height, 即 88px * 100% = 88px
    height: 100%;
    // 相对于父元素的 width, 即 100px * 5% = 5px
    padding: 5%;
    // 相对于父元素的 width, 即 100px * 6% = 6px
    margin: 6%;

    position: relative;
    // 相对于父元素的 width, 即 100px * 5% = 5px
    left: 5%;
    // 相对于父元素的 width, 即 100px * 5% = 5px
    right: 5%;
    // 相对于父元素的 height, 即 88px * 10% = 8.8px
    top: 10%;
    // 相对于父元素的 height, 即 88px * 10% = 8.8px
    bottom: 10%;
}
```
**viewport**  
viewport 的意思是视口，我们可以浅层次的理解为浏览器中页面的可见区域；现在也是比较主流的移动端适配方案  

|  单位   | 描述  |
|  ----  | ----  |
| vw  | viewport width，视窗宽度，1vw = 视窗宽度的1% |
| vh  | viewport height，视窗高度，1vh = 视窗高度的1% |
| vmin  | vw 和 vh 中较小的那个 |
| vmax  | vw 和 vh 中较大的那个 |

**不常用的单位**  
|  单位   | 描述  |
|  ----  | ----  |
| ch  | 相对于0尺寸 |
| in  | inch, 表英寸 |
| cm  | centimeter, 表厘米 |
| mm  | millimeter, 表毫米 |
| pt  | 1/72英寸 |
| pc  | 12点活字，或1/12点 |
| ex  | 相对于小写字母"x"的高度 |
| gd  | 一般用在东亚字体排版上，这个与英文并无关系  |

原文:  
[CSS 长度单位中，你不知道的细节](https://juejin.cn/post/7032247874931032100)
