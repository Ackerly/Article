# 两行CSS让页面提升了近7倍渲染性能
**content-visibility**  
> content-visibility是CSS新增的属性，主要用来提高页面渲染性能，它可以控制一个元素是否渲染其内容，并且允许浏览器跳过这些元素的布局与渲染。

- visible：默认值，没有效果。元素的内容被正常布局和呈现
- hidden：元素跳过它的内容。跳过的内容不能被用户代理功能访问，例如在页面中查找、标签顺序导航等，也不能被选择或聚焦。这类似于给内容设置display: none
- auto：该元素打开布局包含、样式包含和绘制包含。如果该元素与用户不相关，它也会跳过其内容。与 hidden 不同，跳过的内容必须仍可正常用于用户代理功能，例如在页面中查找、tab 顺序导航等，并且必须正常可聚焦和可选择。

**content-visibility: hidden手动管理可见性**  
content-visibility: hidden的效果与display: none类似，但其实两者还是有比较大的区别的：  
- content-visibility: hidden 只是隐藏了子元素，自身不会被隐藏
- content-visibility: hidden 隐藏内容的渲染状态会被缓存，所以当它被移除或者设为可见时，浏览器不会重新渲染，而是会应用缓存，所以对于需要频繁切换显示隐藏的元素，这个属性能够极大地提高渲染性能。

添加了content-visibility: hidden元素的子元素确实是没有渲染，但它自身是会渲染的！  

**content-visibility: auto 跳过渲染工作**  
页面上虽然会有很多元素，但是它们会同时呈现在用户眼前吗，很显然是不会的，用户每次能够真实看到就只有设备可见区那些内容，对于非可见区的内容只要页面不发生滚动，用户就永远看不到。虽然用户看不到，但浏览器却会实实在在的去渲染，以至于浪费大量的性能。所以我们得想办法让浏览器不渲染非可视区的内容就能够达到提高页面渲染性能的效果。  
虚拟列表原理其实就跟这个类似，在首屏加载时，只加载可视区的内容，当页面发生滚动时，动态通过计算获得可视区的内容，并将非可视区的内容进行删除，这样就能够大大提高长列表的渲染性能。  
但这个需要配合JS才能实现，现在我们可以使用CSS中content-visibility: auto，它可以用来跳过屏幕外的内容渲染，对于这种有大量离屏内容的长列表，可以大大减少页面渲染时间。  
将上面的例子稍微改改：  
``` 
<template>
  <div class="card_item">
    <div class="card_inner">
      <img :src="book.bookCover" class="book_cover" />
      <div class="card_item_right">
        <div class="book_title">{{ `${book.bookName}${index + 1}` }}</div>
        <div class="book_author">{{ book.catlog }}</div>
        <div class="book_tags">
          <div class="book_tag" v-for="(item, index) in book.tags" :key="index">
            {{ item }}
          </div>
        </div>
        <div class="book_desc">
          {{ book.desc }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { toRefs } from "vue";

const props = defineProps<{
  book: any;
  index: any;
}>();
const { book, index } = toRefs(props);
</script>

<style lang="less" scoped>
.card_item {
  margin: 20px auto;
  content-visibility: auto;
}
  / *
  ...
  */
</style>
```
首先是没有添加content-visibility: auto的效果，无论这些元素是否在可视区，都会被渲染  
为了性能考虑，我们为每一个列表项加上：  
``` 
.card_item {
  content-visibility: auto;
}
```
没在可视区的元素就没有被渲染，这可比上面那种全部元素都渲染好太多了，但是如果浏览器不渲染页面内的一些元素，滚动将是一场噩梦，因为无法正确计算页面高度。这是因为，content-visibility会将分配给它的元素的高度（height）视为0，浏览器在渲染之前会将这个元素的高度变为0，从而使我们的页面高度和滚动变得混乱。  
页面上的滚动条会出现抖动现象，这是因为可视区外的元素只有出现在了可视区才会被渲染，这就回导致前后页面高度会发生变化，从而出现滚动条的诡异抖动现象，这是虚拟列表基本都会存在的问题。  
_注意：当元素接近视口时，浏览器不再添加size容器并开始绘制和命中测试元素的内容。这使得渲染工作能够及时完成以供用户查看。_  
为什么从第十个才开始不渲染子元素，因为它需要一个缓冲区以便浏览器能够在页面发生滚动时及时渲染呈现在用户眼前。  
size其实是一种 CSS 属性的潜在值contain，它指的是元素上的大小限制确保元素的框可以在不需要检查其后代的情况下进行布局。这意味着如果我们只需要元素的大小，我们可以跳过后代的布局。  
**contain-intrinsic-size 救场**  
页面在滚动过程中滚动条一直抖动，这是一个不能接受的体验问题，为了更好地实现content-visibility，浏览器需要应用 size containment 以确保内容的渲染结果不会以任何方式影响元素的大小。这意味着该元素将像空的一样布局。如果元素没有在常规块布局中指定的高度，那么它将是 0 高度。  
这个时候我们可以使用contain-intrinsic-size来指定的元素自然大小，确保我们未渲染子元素的 div 仍然占据空间，同时也保留延迟渲染的好处。  

_语法_  
此属性是以下 CSS 属性的简写：
- contain-intrinsic-width
- contain-intrinsic-height

``` 
/* Keyword values */
contain-intrinsic-width: none;

/* <length> values */
contain-intrinsic-size: 1000px;
contain-intrinsic-size: 10rem;

/* width | height */
contain-intrinsic-size: 1000px 1.5em;

/* auto <length> */
contain-intrinsic-size: auto 300px;

/* auto width | auto height */
contain-intrinsic-size: auto 300px auto 4rem;
```
> contain-intrinsic-size 可以为元素指定以下一个或两个值。如果指定了两个值，则第一个值适用于宽度，第二个值适用于高度。如果指定单个值，则它适用于宽度和高度

_实现_  
只需要给添加了content-visibility: auto的元素添加上contain-intrinsic-size就能够解决滚动条抖动的问题，当然，这个高度约接近真实渲染的高度，效果会越好，如果实在无法知道准确的高度，也可以给一个大概的值，也会使滚动条的问题相对减少。  
``` 
.card_item {
  content-visibility: auto;
  contain-intrinsic-size: 200px;
}
```
之前没添加contain-intrinsic-size属性时，可视区外的元素高度都是0，现在这些元素高度都是我们设置的contain-intrinsic-size的值，这样的话整个页面的高度就是不会发生变化（或者说变化很小），从而页面滚动条也不会出现抖动问题（或者说抖动减少）

**性能对比**  
实际对比：  
- 首先是没有content-visibility的页面渲染
- 然后是有content-visibility的页面渲染

上面是用1000个列表元素进行测试的，有content-visibility的页面渲染花费时间大概是37ms，而没有content-visibility的页面渲染花费时间大概是269ms，提升了足足有7倍之多  
对于列表元素更多的页面，content-visibility带来的渲染性能提升会更加明显。  

**思考**  
_能否减小页面的内存占用_  
通过chrome浏览器 设置 --> 更多工具 --> 任务管理器 查看页面占用内存大小,它并不会减少页面占用内存大小，这些元素是真实存在于DOM树中的，并且我们也可以通过JS访问到
_是否会影响脚本的加载行为_  
在添加了content-visibility: auto的元素内去加载脚本，并且此时的元素处于一个不可见的状态，那么此时元素内的脚本能够正常加载呢  
``` 
<div class="visibility_item">
        <div class="inner">
            测试脚本
            <img src="../../../../images/22-11/content-s1.png" alt="">
            <script src="./2.js"></script>
        </div>
        
</div>
```
使用了content-visibility: auto的元素影响的只是子元素的渲染，对于内部静态资源的加载还是正常进行。
但需要注意的是脚本的执行时机，如果要获取DOM元素的话，此时的脚本只能获取到它加载位置之前的DOM元素，而与它自身DOM有没有渲染无关！  

_可访问性_  
content-visibility: auto是屏幕外的内容在文档对象模型中仍然可用，因此在可访问性树中（与visibility: hidden不同）。这意味着我们可以在页面上搜索并导航到该内容，而无需等待它加载或牺牲渲染性能。  
这个功能特性是在chrome 90 中更新的，在 chrome 85-89 中，屏幕外的子元素content-visibility: auto被标记为不可见。  

_兼容性_  
content-visibility是chrome85新增的特性，所以兼容性还不是很高，但它是一个非常实用的CSS属性，由于跳过了渲染，如果我们大部分内容都在屏幕外，利用该content-visibility属性可以使初始用户加载速度更快。相信兼容性的问题在不久的将来会得到解决


原文:  
[两行CSS让页面提升了近7倍渲染性能](https://juejin.cn/post/7168629736838463525)
