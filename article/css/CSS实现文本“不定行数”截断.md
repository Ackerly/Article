# CSS 实现文本"不定行数"截断
效果：  
- 整个容器高度是固定的，标题和内容总共 3 行
- 标题最多2行，超出省略
- 内容由剩余空间决定，如果标题占2行，则内容超出1行省略；如果标题占1行，则内容超出2行省略

## 标题超出省略
``` 
<div class="section">
    <h3 class="title">LGD 在 TI10 放猛犸，RNG 在 S7 放加里奥最后都输了，哪个更让你愤怒失望？</h3>
    <p class="excerpt">猛犸是对面的绝中绝，大家都知道，并且之前扳回两局已经证明了，当lgd选择ban掉猛犸，或者自己抢掉猛犸时，对面完全不是对手。</p>
</div>
```
标题的规则是超出2行省略，用 -webkit-line-clamp 实现
``` 
.title{
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
}
```
## 内容自适应行数
整个高度是固定的，如果让内容部分自适应剩余空间
``` 
.section{
    display: flex;
    overflow: hidden;
    height: 72px;/*定一个高度*/
    flex-direction: column;
}
.excerpt{
  	flex: 1; /*自适应剩余空间*/
  	overflow: hidden;
}
```
## 不定行超出省略
**右下角环绕效果**  
添加一个伪元素，设置右浮动
``` 
.excerpt::before{
    content: '...';
    float: right;
}
```
省略号目前是在右上角的，现在需要挪到右下角来
``` 
.excerpt::before{
    content: '...';
    float: right;
  	height: 100%;
    display: flex;
    align-items: flex-end;
}
```
省略号虽然到了右下角，但是上面的部分也被挤走了，没有达到环绕的效果。用 shapes 布局了，利用 shape-outside:inset()就可以了，表示以自身为边界，然后上、右、下、左四个方向分别向内缩进的距离。
``` 
.excerpt::before{
    /*其他样式**/
  	shape-outside: inset(calc(100% - 1.5em) 0 0);
}
```
**自动隐藏省略号**  
用一个足够大的色块盖住省略号，设置绝对定位后，色块是跟随内容文本的，所以在文字较多时，色块也跟随文本挤下去了
``` 
.excerpt::after{
    content: '';
    position: absolute;
    width: 999vh;
    height: 999vh;
    background: #fff;
}
```
个别极端情况可能会遮挡不住,可以随便找个东西补上，比如 box-shadow，往左下角偏移一点就可以了
``` 
.excerpt::after{
    content: '';
    position: absolute;
    width: 999vh;
    height: 999vh;
    background: #fff;
    box-shadow: -2em 2em #fff; /*左下的阴影*/
}
```

参考:
[CSS 实现文本"不定行数"截断](https://juejin.cn/post/7022876094608982030)
