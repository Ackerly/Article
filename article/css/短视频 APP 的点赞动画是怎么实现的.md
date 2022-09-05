# 短视频APP的点赞动画是怎么实现的
**实现不同表情的不断上升**  
如果使用纯 CSS 实现这一整套动画的话。首先需要实现一段无限循环的，大量不同的表情不断向上漂浮的动画。  
这个整体还是比较容易实现的，核心原理就是同一个动画，设置不同的 transition-duration，transition-dalay，和一定范围内的旋转角度。  
首先要实现多个表情，一个 DOM 标签放入一个随机的表情  
可以手动一个一个的添加：  
``` 
<ul class="g-wrap">
    <li>😀</li>
    <li>❤️</li>
    <li>👏</li>
    // ... 随机设置不同的表情符号，共 50 个
    <li>...</li>
</ul>
```
也可以利用 SASS 的循环函数及随机函数，利用伪元素的 content 去随机生成不同表情。像是这样：  
``` 
<ul class="g-wrap">
    <li></li>
    <li></li>
    <li></li>
    // ... 共50个空标签
</ul>

// CSS
$expression: "😀", "🤣", "❤️", "😻", "👏", "🤘", "🤡", "🤩", "👍🏼", "🐮", "🎈", "💕", "💓", "💚";
.g-wrap {
    position: relative;
    width: 50px;
    height: 50px;
}
@for $i from 1 to 51 {
    li:nth-child(#{$i}) {
        position: absolute;
        top: 0;
        left: 0;
        width: 50px;
        height: 50px;
        
        &::before {
            content: nth($expression, random(length($expression)));
            position: absolute;
            font-size: 50px;
        }
    }
}
```
> 因为透明度为 1 的缘故，只能看到最上面的几个表情，实际上这里叠加了 50 个不同的表情

这里的核心就是 content: nth($expression, random(length($expression)))，我们利用了 SASS 的 random 和 length 和 nth 等方法，随机的将 $expression 列表中的表情，添加给了不同的 li 的 before 伪元素的 content 内  
接下来，需要让它们动起来。  
添加一个无限的 transform: translate() 动画:  
``` 
@for $i from 1 to 51 {
    li:nth-child(#{$i}) {
        animation: move 3000ms infinite linear;
    }
}
@keyframes move {
    100% {
        transform: translate(0, -250px);
    }
}
```
由于 50 个元素都叠加在一起，所以需要将动画区分开来，给它们添加随机的动画时长，并且，赋予不同的负 transition-delay 值：  
``` 
@for $i from 1 to 51 {
    li:nth-child(#{$i}) {
        animation: move #{random() * 2500 + 1500}ms infinite #{random() * 4000 / -1000}s linear;
    }
}
@keyframes move {
    100% {
        transform: translate(0, -250px);
    }
}
```
这里有一点点的跳跃，需要理解 move #{random() * 2500 + 1500}ms infinite #{random() * 4000 / -1000}s linear 这里大段代码：  
1. #{random() * 2500 + 1500}ms 生成 1500ms ~ 4000ms 之间的随机数，表示动画的持续时长
2. #{random() * 4000 / -1000}s 生成 -4000ms ~ 0s 之间的随机数，表示负的动画延迟量，这样做的目的是为了让动画提前进行

还是不够随机，我们再通过随机添加一个较小的旋转角度，让整体的效果更加的随机：  
``` 
@for $i from 1 to 51 {
    li:nth-child(#{$i}) {
        transform: rotate(#{random() * 80 - 40}deg);
        animation: move #{random() * 2500 + 1500}ms infinite #{random() * 4000 / -1000}s linear;
    }
}
@keyframes move {
    100% {
        transform: rotate(0) translate(0, -250px);
    }
}
```
transform: rotate(#{random() * 80 - 40}deg) 的作用就是随机生成 -40deg ~ 40deg 的随机数，产生一个随机的角度  

**利用 transition 化腐朽为神奇**  
明明是点赞一次产生一个表情，为什么需要一次生成这么多不断运动的表情效果呢？  
由于 CSS 没法直接正面做到点击一次，生成一个表情，所以我们需要换一种思路实现。  
如果这些表情一直都是在运动的，只不过不点击的时候，它们的透明度都为 0，我们要做的，就是当我们点击的时候，让它们从 opacity: 0 变到 opacity: 1  
要实现这一点，我们需要巧妙的用到 transition  
以一个表情为例子  
1. 默认它的透明度为 opacity: 0.1
2. 点击的时候，它的透明度瞬间变成 opacity: 1
3. 然后，通过 transition-delay 让 opacity: 1 的状态保持一段时间后
4. 逐渐再消失，变回 opacity: 0.1

看上去有点复杂，代码会更容易理解：  
``` 
li {
    opacity: .1;
    transition: 1.5s opacity 0.8s;
}
li:active {
    opacity: 1;
    transition: .1s opacity;
}
```
巧妙地利用 transition 在正常状态和 active 状态下的变化，实现了巧妙的点击效果  
把初始的 opacity: 0.1 改成 opacity: 0  
结合一下上面两个动画：  
1. 将所有的表情，默认的透明度改为 0.1
2. 被点击的时候，透明度变成 1
3. 透明度在 1  维持一段时间，逐渐消失

``` 
@for $i from 1 to 51{
    li:nth-child(#{$i}) {
        position: absolute;
        top: 0;
        left: 0;
        width: 50px;
        height: 50px;
        transform: rotate(#{random() * 80 - 40}deg);
        animation: move #{random() * 2500 + 1500}ms infinite #{random() * 4000 / -1000}s linear;
        opacity: .1;
        transition: 1.5s opacity .8s;
        
        &::before {
            content: nth($expression, random(length($expression)));
            position: absolute;
        }
    }
    
    li:active {
        opacity: 1;
        transition: .1s opacity;
    }
}

@keyframes move {
    100% {
        transform: rotate(0) translate(0, -250px);
    }
}
```
通过一个点击按钮引导用户点击，并且给与一个点击反馈，每次点击的时候，点赞按钮放大 1.1 倍，同时，我们把默认表情的透明度从 opacity: 0.1 彻底改为 opacity: 0。  
整个动画的完整的核心代码：  
``` 
<ul class="g-wrap">
    <li></li>
    <li></li>
    <li></li>
    // ... 共50个空标签
</ul>

$expression: "😀", "🤣", "❤️", "😻", "👏", "🤘", "🤡", "🤩", "👍🏼", "🐮", "🎈", "💕", "💓", "💚";
.g-wrap {
    position: relative;
    width: 50px;
    height: 50px;
    &::before {
        content: "👍🏼";
        position: absolute;
        width: 50px;
        height: 50px;
        transition: 0.1s;
    }
    &:active::before {
        transform: scale(1.1);
    }
}

@for $i from 1 to 51 {
    li:nth-child(#{$i}) {
        position: absolute;
        width: 50px;
        height: 50px;
        transform: rotate(#{random() * 80 - 40}deg);
        animation: move #{random() * 2500 + 1500}ms infinite #{random() * 4000 / -1000}s cubic-bezier(.46,.53,.51,.62);
        opacity: 0;
        transition: 1.5s opacity .8s;
        &::before {
            content: nth($expression, random(length($expression)));
            position: absolute;
        }
    }
    li:active {
        transition: .1s opacity;
        opacity: 1!important;
    }
}
@keyframes move {
    100% {
        transform: rotate(0) translate(0, -250px);
    }
}
```
需要注意的是：  
1. 点赞的按钮，通过了父元素 .g-wrap 的伪元素实现，这样的好处是，子元素 li 的 :active 点击事件，是可以冒泡传给父元素的，这样每次子元素被点击，我们都可以放大一次点赞按钮，用于实现点击反馈。
2. 稍微修改一下缓动函数，让整体效果更为均衡合理

**瑕疵**  
1. 就是如果当点击的速率过快，是无法实现一个点击，产生一个表情的    
   这是由于 CSS 方案的本质是通过点击一个透明表情，让它变成不透明。而点击过快的话，会导致两次或者多次点击，点在了同一个元素上，这样，就无法实现一个点击，产生一个表情。所以上面代码中修改缓动 cubic-bezier(.46,.53,.51,.62) 的目的也是在于，让元素动画前期运动更快，这样可以有利于适配更快的点击速率
2. 不仅仅是点击按钮，点击按钮上方也能出现效果  
   由于本质是个障眼法，所以点击按钮上方，只要是元素运动路径的地方，也是会有元素显形的。这个硬要解决也可以，通过再叠加一层透明元素在按钮上方，通过层级关系屏蔽掉点击事件。
3. 表情的随机只是伪随机
   利用 SASS 随机的方案在经过编译后是不会产生随机效果的。所以，这里只能是伪随机，基于 DOM 的个数，当 DOM 数越多，整体而言，随机的效果越好。基本上 50 个 DOM 是比较足够的。
4. CSS 版本的点赞效果是单机版  
   无法多用户联动，可能是影响能不能实际使用最为关键的因素。


原文: 
[短视频 APP 的点赞动画是怎么实现的](https://mp.weixin.qq.com/s/JJYXFcocCeguYg7egRxhYw)
