# CSS backdrop-filter实现毛玻璃
## backdrop-filter
CSS中有一个属性叫filter,这两个属性在语法层面是相同的。filter 是将效果用于元素自身，而 backdrop-filter 则是将效果用于元素的背景。
> /* 关键词值 */  
  backdrop-filter: none;  
  /* 指向 SVG 滤镜的 URL */  
  backdrop-filter: url(commonfilters.svg#filter);  
  /* <filter-function> 滤镜函数值 */  
  backdrop-filter: blur(2px);  
  backdrop-filter: brightness(60%);  
  backdrop-filter: contrast(40%);  
  backdrop-filter: drop-shadow(4px 4px 10px blue);  
  backdrop-filter: grayscale(30%);  
  backdrop-filter: hue-rotate(120deg);  
  backdrop-filter: invert(70%);  
  backdrop-filter: opacity(20%);  
  backdrop-filter: sepia(90%);  
  backdrop-filter: saturate(80%);  
  /* 多重滤镜 */  
  backdrop-filter: url(filters.svg#filter) blur(4px) saturate(150%);  
  /* 全局值 */  
  backdrop-filter: inherit;  
  backdrop-filter: initial;  
  backdrop-filter: unset;  

## 实现
``` 
// html
<div class="container">
  <div class="card">Life doesn't have to be perfect， but to be wonderful</div>
</div>
```
``` 
// 容器样式
.container{
         position: relative;
         width: 1000px;
         height: 800px;
         background: url(./bk.jpg);
         background-repeat: no-repeat;
   }
```
``` 
// 卡片样式
使用 blur： backdrop-filter: blur(10px);
.card{
     position: absolute;
     top: 50%;
     left: 50%;
     height:50px;
     line-height: 50px;
     backdrop-filter: blur(10px);
     transform: translate(-50% ,-50%);
     padding: 0 20px;
 }
```
## 图片背景
效果：图片在上一层，下层是一个模糊的背景。  
```
<div class="bkImg">
  <img src="" class="Img"/>
</div>
```
``` 
 .bkImg {
        background-color: #222;
        align-items: center;
        display: flex;
        justify-content: center;
        height: 100vh;
        width: 100%;
        position: relative;
        background-image: url();

        background-position: center;
        background-repeat: no-repeat;
        background-size: cover;
      }

      .Img {
        position: relative;
        max-height: 90%;
        max-width: 90%;
        z-index: 1;
      }
    .bkImg::after {
      backdrop-filter: blur(50px);
      content: "";
      display: block;
      height: 100%;
      left: 0;
      position: absolute;
      top: 0;
      width: 100%;
      z-index: 0;
    }
```
## 添加日出动画
``` 
// 日出背景
<div class="card">
</div>
 .card {
      background-color: #f5f5f7;
      padding-top:150px;
      height: 300px;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 64px 94px;
      position: relative;
    }
```
``` 
// 太阳
<div class="card">
  <div class="sun"></div>
  <div class="bk"></div>
</div>

.sun {
  position: absolute;
  top: 150px;
  width: 130px;
  height: 130px;
  background-color: #ff6260;
  border-radius: 50%;
  animation: sunRise 5S;
}

.bk {
  position: sticky;
  bottom: 0;
  width: 100%;
  background-color: green;
  padding-top: 64px;
  padding-bottom: 8px;
  height: 130px;
  background-color: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  margin-top: 32px;
  backdrop-filter: blur(20px);
}
```
``` 
// 动画
@keyframes sunRise{
  0%{
      top:150px
  }
  100%{
    top:0px;
  }
}
```

参考:  
[backdrop-filter,让你的网站熠熠生”毛’](https://juejin.cn/post/7015608045895942180)
