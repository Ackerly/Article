# 关于box-size面试题
**CSS 盒子模型**  
CSS 盒子组成由 4 个区域组成，从内到外依次为:
- Content box：内容盒子，用于显示内容（innerHTML），默认通过 width 和 height 控制宽高。但如果box-sizing（盒模型的意思）属性设置为非 content-box 值，运用的规则会发生改变。
- Padding box：内边距盒子，通过 padding 属性可以设置内边距大小。
- Border box：边框盒子，通过 border 属性可以设置边框大小及样式。
- Margin box：外边距盒子，通过 margin 属性设置外边距大小

需要注意的是，margin 不计入盒子的实际大小。比如盒子的背景色不会覆盖到 margin 的范围。  
**标准盒模型**  
对于现代浏览器来说，元素默认应用标准盒模型。  
标准盒模型中，width 和 height 属性用于设置 Content box 盒子。  
```
.box {
  width: 10px;
  height: 10px;
  border: 1px solid #000;
  padding: 2px;
  margin: 2px;
  background-color: orange;
}
.content-box {
  box-sizing: content-box;
}

<div class="box content-box"></div>
```
content 的宽度为 10px。padding 为 2px，这个 padding 是 padding-left、padding-right、padding-bottom、padding-left 的简写属性。盒子的宽需要将 padding-left 和 padding-right 都计算在内。左右两个 border 条。margin 不计算在盒模型中。  
所以对于盒模型来说，宽度就是 16px（10 + 2 * 2 + 1 * 2），高度同理，也是 16px。  
要找到的橙色块的宽高，其实就是 Padding box 的宽高，这个块并不包括黑色的 border 边框线。所以我们的第一个橙色块宽高为 14px。  
如果给 border 颜色设置为透明，比如 border: 1px solid rgb(0, 0, 0, 0)，觉得橙色块宽高为多少？  
答案是 16px。背景色会先填充整个盒子，然后再在其上添加 border。如果 border 变成透明了，就会将它原本覆盖的部分橙色区域显现出来。  
**怪异盒模型**
怪异盒模型，也叫 IE 模型。IE 浏览器的早期版本没有遵循 CSS 标准，width 和 height 是用来设置 Border box 的宽高，而不是 Content box 的宽高，导致不同浏览器的表现不同，毫无疑问是个浏览器 bug。  
后来 CSS3 引入了 box-sizing，让开发者可以选择使用哪种盒模型，提供更好的灵活性。  
怪异盒模型中，width 和 height 属性用于设置 Border box 盒子。即我们直接给元素对应的盒子设置了宽高，再通过 padding 和 border，才能计算出 Content box。  
```
.box {
  width: 10px;
  height: 10px;
  border: 1px solid #000;
  padding: 2px;
  margin: 2px;
  background-color: orange;
}
.border-box {
  box-sizing: border-box;
}
<div class="box border-box">
```
盒模型宽为 10px，减去 border 的 2px（左右两条 1px 的边框线），计算出来的就是 Border box 盒子的宽度 8px。高度计算同理。  


参考:
[一道关于 box-sizing 的字节面试题](https://juejin.cn/post/7082774002099290149)