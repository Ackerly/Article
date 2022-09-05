# 使用CSS text-combine-upright实现字符的横向合并
## 从垂直排版说起
垂直排版段落中的数字都是90度旋转排列的，这样的排列并不利于阅读，希望可以横着排列，可以使用text-orientation属性，例如，在容器元素上设置：
``` 
p {
    text-orientation: upright;
    writing-mode: vertical-lr;
}
```
上面的数字虽然正立了，但是阅读起来还是有些吃力，尤其2021，阅读速度还不如默认的90度旋转排列的效果呢。
## text-combine-upright简介
text-orientation属性可以让垂直排版中的局部区域的文字水平排版，同时多个水平排版字符占据的宽度和一个正常的字符一样。  
text-combine-upright属性支持下面这些属性值：
``` 
/* 关键字值 */
text-combine-upright: none;
text-combine-upright: all;

/* 数字值 */
text-combine-upright: digits;     /* 按照2个数字算 */
text-combine-upright: digits 4;   
```
none: 初始值。不连续横排。  
all:  试图水平排版框内所有连续字符，使它们占用框垂直线内单个字符的空间。  
digits <integer>?:  多少个连续数字认为是横着显示。默认是2，范围不能在2-4之外，否则认为是不合法。也就是最多只能让一个标签内4个字符水平排列。  
Chrome 88以及Firefox 85以及Safari都不支持这个高级的语法。  
只能使用text-combine-upright:all进行水平排版，已知有如下HTML和CSS：
``` 
<p class="upright">测试文字排版<span>2021</span>年<span>2</span>月<span>18</span>日我<span>36</span>岁</p>

.upright {
    writing-mode: vertical-lr;
}
.upright span {
    text-combine-upright: all;
    /* forSafari */
    -webkit-text-combine: horizontal;
    text-combine-upright: digits 2;
}
```
### 和普通水平排版的区别
重置span元素的排版方式也是可以让数字横向排列，例如：
``` 
.upright {
    writing-mode: vertical-lr;
}
.upright span {
    writing-mode: initial;
}
```
## IE和Safari的兼容
IE浏览器使用的是-ms-text-combine-horizontal这个属性，语法如下所示：
``` 
-ms-text-combine-horizontal: none | all | [ digits <integer> ? ]
```
和text-combine-upright属性一致。  
Safari浏览器使用的是-webkit-text-combine属性，仅支持none和horizontal这两个属性值。
``` 
.target {
    -ms-text-combine-horizontal: all;
    -webkit-text-combine: horizontal;
    text-combine-upright: all;
}
```
IE11浏览器单个数字的时候，字形变得宽大厚重了，这个是和其他现代浏览器不一样的表现。
## 水平排版中的应用
需要垂直排版，然后目标应该是数字或者字符长度不超过4个词组。以布局的角度思考这个属性，可以实现任意2~4个字符按照1个字符宽度渲染排版的效果，比如'ces'现在是3字符宽度，使用text-combine-upright可以让其最多占据一个字符宽度。

原文: 
[使用CSS text-combine-upright实现字符的横向合并](https://www.zhangxinxu.com/wordpress/2021/02/css-text-combine-upright/)
