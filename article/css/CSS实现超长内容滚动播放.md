# CSS实现超长内容滚动播放
## 功能实现
**基础布局**  
构造一个基本样式  
- box 作为任意需要包含滚动组件 div 的 class，宽度、边框等个性化的 style 都应该在 box 上设置
- scroll-wrap 作为滚动容器的通用 class
- scroll-item 作为滚动内容的通用 class

``` 
// index.html <!DOCTYPE html>
 <html lang="en">
 <head>
     <meta charset="UTF-8">
     <meta http-equiv="X-UA-Compatible" content="IE=edge">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Document</title>
     <link rel="stylesheet" href="./style.css">
 </head>
 <body>
     <div class="box">
         <div class="scroll-wrap">
             <div class="scroll-item">
                 我是开头，我是中间我是中间我是中间，我是结尾             </div>
         </div>
     </div>
 </body>
 </html>
```
``` 
box {
     max-width: 15em;
     border: 1px solid;
 }

 .scroll-wrap {
     overflow: hidden;
     white-space: nowrap;
 }
```

**动效分析**  
只要实现内容的的两种状态，再加上 animation 让这两种状态来回切换。如果这两种状态的实现都使用的相同的属性，那么切换过程中自然就会加上动画效果  
两种状态：
- 「我是开头」靠左显示
- 「我是结尾」靠右显示

先写一个来回切换状态的 animation 动画  
``` 
.scroll-item {
     animation: scroll linear 4s alternate infinite;
 }

 @keyframes scroll {
     0% {
         background: red;
     }

     100% {
         background: yellow;
     }
 }
```
animation 的属性  
- animation-name：所使用的 @keyframes 的名称 scroll
- animation-timing-function：动画的速度曲线，linear 表示匀速
- animation-duration：规定完成动画所花费的时间为 4s
- animation-direction：是否应该轮流反向播放动画，alternate 表示在偶数次会反向播放
- animation-iteration-count：定义动画的播放次数，infinite 表示无限次
- animation 定义好了，然后开始实现最关键的部分，头尾靠边显示的两种状态

**头尾位置**  
单纯的实现其实很简单，直接用 position: absolute，再分别设置 left，right 即可  
``` 
.scroll-wrap {
     overflow: hidden;
     white-space: nowrap;
     position: relative;
     height: 1.4em;

 }

 .scroll-item {
     animation: scroll linear 4s alternate infinite;
     position: absolute;
 }

 @keyframes scroll {
     0% {
         left: 0;
     }

     100% {
         right:0;
     }
 }
```
由于 left、right 是不同的属性，所以状态改变过程中并没有动画效果 ,所以接下来我们的目标就是：如何用 left 实现 right: 0
思考 position: absolute 是如何实现居中的。就会比较容易联想到，可以通过 transform 属性来实现我们想要的效果   
``` 
@keyframes scroll {
     0% {
         left: 0;
         transform: translateX(0);
     }

     100% {
         left: 100%;
         transform: translateX(-100%);
     }
 }
```
再给 keyframes 加上两个动画节点，让文案出现头尾的停留  
``` 
@keyframes scroll {
     0% {
         left: 0;
         transform: translateX(0);
     }

     10% {
         left: 0;
         transform: translateX(0);
     }

       90% {
         left: 100%;
         transform: translateX(-100%);
     }

     100% {
         left: 100%;
         transform: translateX(-100%);
     }
 }
```

**代码优化**  
由于用了 position: absolute，导致需要额外给父元素指定高度。理想的状态应该是  
- 内容高度自然撑开
- box 宽度大于内容时，不会发生滚动
- box 宽度由内容撑开

究其原因，在于宽高都无法自适应，而这主要是由 position: absolute 造成的。所以换一个其他方案来实现两种状态。  
``` 
// 个性化部分 ⬇️
 .box {
     width: 15em;
     border: 1px solid #ddd;
 }

 // 通用代码部分 ⬇️
 .scroll-wrap {
     max-width: 100%;
     display: inline-block;
     vertical-align: top;
     overflow: hidden;
     white-space: nowrap;
 }

 .scroll-item {
     animation: scroll linear 4s alternate infinite;
     float: left;
 }

 @keyframes scroll {
     0% {
         margin-left: 0;
         transform: translateX(0);
     }

     10% {
         margin-left: 0;
         transform: translateX(0);
     }

     90% {
         margin-left: 100%;
         transform: translateX(-100%);
     }

     100% {
         margin-left: 100%;
         transform: translateX(-100%);
     }
 }
```
- max-width: 100%：保证自身宽度不超过 box
- display: inline-block：目的是让 scroll-wrap 宽度自适应，可以被子元素撑开
- vertical-align: top：设置完上面的属性之后，inline 会带上 1.x 的行高导致高度过大。设置 top 可以消除
- float: left：为了让后续的 margin-left，transform 符合预期需要设置 float
- margin-left：等效于上一个方案中的 left

原文: 
[利用CSS实现超长内容滚动播放](https://mp.weixin.qq.com/s/BAlijJ-t5PKetbU53Ge-yg)
