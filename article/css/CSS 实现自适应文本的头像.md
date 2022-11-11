# CSS 实现自适应文本的头像
## CSS 容器尺寸单位
容器查询语句: 
- cqw 容器查询宽度（Container Query Width）占比。1cqw 等于容器宽度的 1%。假设容器宽度是 1000px，则此时 1cqw 对应的计算值就是 10px
- cqh 容器查询高度（Container Query Height）占比。1cqh 等于容器高度的 1%
- cqi 表示容器查询内联方向尺寸（Container Query Inline-Size）占比。这个是逻辑属性单位，默认情况下等同于 cqw
- cqb 容器查询块级方向尺寸（Container Query Block-Size）占比。同上，默认情况下等同于 cqh
- cqmin 容器查询较小尺寸的（Container Query Min）占比。取 cqw 和 cqh 中较小的一个
- cqmax 表示容器查询较大尺寸的（Container Query Min）占比。取 cqw 和 cqh 中较大的一个

举个例子  
``` 
<div class="con">
   <p class="text">大家好，欢迎关注前端侦探</p>
 </div>
```
在不声明容器类型的情况下，cqw 等同于 vw，也就是相当于把整个页面当成容器，这里希望将这个 div 作为参考对象，需要提前声明 container-type，如下  
``` 
.con {
     container-type: inline-size;
 }
 .text{
   font-size: 10cqw;
 }
```
## 自适应文本头像
有一个比较简单的思路就是，文字越多，占据的宽度越多，然后根据前面的原理，让文字大小随着宽度的变化而变化，只不过这里是成反比，宽度越宽，字号越小。  
假设 HTML 是这样的  
``` 
<div class="avator">
     <span>侦探</span>
 </div>
 
 // CSS
 .avator{
   display: flex;
   align-items: center;
   justify-content: center;
   width: 40px;
   height: 40px;
   border-radius: 8px;
   background: bisque;
   color: rgb(250, 84, 28);
   white-space: nowrap;
 }
```
现在问题来了，目前外层宽度是固定的，好像没办法根据文字占据宽度就行容器查询，怎么办呢？  
创建一份一模一样的文本，让外层容器（A）宽度由内部文本决定，然后将容器盒子（B）的宽度设置成和（A）一样，这样不就完成了容器查询吗？  
可以将 HTML 改造成这样  
``` 
<div class="avator">
   <div class="avator-inner" alt="侦探"><!--外层容器A-->
     <div class="avator-container"><!--容器盒子B-->
       <span>侦探</span>
     </div>
   </div>
 </div>
```
外层容器 A 的文本可以通过伪元素生成  
``` 
.avator-inner::before{
   content: attr(alt);
   font-size: 40px;
 }
```
这时外层容器 A ，也就是.avator-inner 的尺寸完全由伪元素::before 撑开,然后，将容器盒子 B，也就是.avator-container 设置成和外层容器 A 一样，可以采用绝对定位的方式  
``` 
.avator-container {
   position: absolute;
   inset: 0;
   container-type: inline-size;
   text-align: center;
   display: flex;
   justify-content: center;
   align-items: center;
 }
```
容器盒子就可以跟随伪元素所占大小自动变化了，然后给内部文字设置一个合适的大小，由于是成反比，所以可以采取相减的方式，如下  
``` 
.avator-container span {
   font-size: calc( 24px - 10cqw );
 }
```
再换一下文本，比如 4 个字的,文本越多，内部文字大小越小，正好是我们需要的效果  
把伪元素生成的文本隐藏起来，就可以得到文章开头的效果了  
完整代码:  
``` 
.avator{
   display: flex;
   align-items: center;
   justify-content: center;
   width: 40px;
   height: 40px;
   border-radius: 8px;
   background: bisque;
   color: rgb(250, 84, 28);
   white-space: nowrap;
 }
 .avator-inner{
   position: relative;
 }
 .avator-inner::before{
   content: attr(alt);
   visibility: hidden;
   font-size: 40px;
 }
 .avator-container {
   position: absolute;
   inset: 0;
   container-type: inline-size;
   text-align: center;
   display: flex;
   justify-content: center;
   align-items: center;
 }
 .avator-container span {
   font-size: calc( 24px - 10cqw );
   overflow: hidden;
   max-width: 40px;
   text-overflow: ellipsis;
 }
```
## 容器查询的一些局限性
1. 容器查询只对容器的子元素有效，对本身无效，这样导致结构有些冗余  
   在上面的例子中，div 容器中还包含了一个 p 元素，文字大小是设置在 p 元素上，看似有些多余，能不能直接设置在 div 上呢？这样就可以省一层标签了。答案是不可以！  
   可以看到，文字大小依赖于页面视图宽度了。所以，如果直接设置在 div 上，那么此时 cqw 参考的容器就不再是本身了，而是继续向上查找，直到最外层，也就是说 cqw 查找的对象是最近的父级容器元素，并不包含自身，这个需要多多注意      
2. 再者，容器查询盒子尺寸本身不能由内部元素所决定
   上面的例子中为啥要创建一份相同的文本呢，原因就是这个，比如在容器盒子本身不设置宽度的情况下，正常的 inline-block 元素宽度应该是有内部文本决定的，但是设置 container-type 之后就不行了，完全没有宽度了，这也就是为啥前面要通过绝对定位的方式直接设置宽度了  
   不过这个原因也容易理解：假设这个成立，如果子元素字号发生变化，导致容器宽度发生变化，容器宽度发生变化又会导致字号发生变化，这样就死循环了，所以不允许这种情况  
3. 容器查询尺寸对应的具体的尺寸，是一种尺寸单位，这样导致有很多属性无法应用，比如 scale  
   在上面的例子中，文字大小是通过 font-size 改变的，其实最好的方式是 scale，因为浏览器有最小字号的限制，而 scale 就没这个限制了，但是 cqw 这种单位无法用在 scale 之上

原文:  
[CSS 实现自适应文本的头像](https://mp.weixin.qq.com/s/3Kt2BqiAa6v8GMSnHo1HBQ)
