# 12个CSS技巧
## 使用 Shape-outside 在浮动图像周围弯曲文本
``` 
.any-shape {
  width: 300px;
  float: left;
  shape-outside: circle(50%);
}
```
##  魔法组合
``` 
* {
padding: 0;
margin: 0;
max-width: 100%;
overflow-x: hidden;
position: relative;
display: block;
}
```
有时“display:block”没有用，但在大多数情况下，你会将 <a> 和 <span> 视为与其他块一样的块。所以，在大多数情况下，它实际上会帮助你
## ::首字母
``` 
p.intro:first-letter {
  font-size: 100px;
  display: block;
  float: left;
  line-height: .5;
  margin: 15px 15px 10px 0 ;
}
```
## 动画属性四大核心属性
- 缩放 - transform:scale(2)
- 旋转 - transform:rotate(180deg)
- 位置 – transform:translateX(50rem)
- 不透明度 - opacity: 0.5
## 圆锥梯度
使用 conic-gradient,设置的颜色过渡围绕中心点旋转
``` 
.piechart {
  background: conic-gradient(rgb(255, 132, 45) 0% 25%, rgb(166, 195, 209) 25% 56%, #ffb50d  56% 100%);
  border-radius: 50%;
  width: 300px;
  height: 300px;
}
```
## 更改文本选择颜色
``` 
::selection {
     background-color: #f3b70f;
 }
```
## 投影
``` 
.img-wrapper img{
          width: 100% ;
          height: 100% ;
          object-fit: cover ;
          filter: drop-shadow(30px 10px 4px #757575);
 }

```
## 使用放置项居中 Div
``` 
main{
     width: 100% ;
      height: 80vh ;
      display: grid ;
      place-items: center center;
}
```
## 使用 Flexbox 居中 Div
``` 
<div class="center h-48">
  <div></div>
</div>

.center {
  display: flex;
  align-items: center;
  justify-content: center;
}

.center div {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  background: #b8b7cd;
}

```

原文:
[12 个救命的 CSS 技巧](https://juejin.cn/post/7024372412632268813)
