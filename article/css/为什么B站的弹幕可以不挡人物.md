# 为什么B站的弹幕可以不挡人物
只需要-webkit-mask-image属性就可以实现
例如：  
``` 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
  <style>
    .video {
      width: 668px;
      height: 376px;
      position: relative;
      -webkit-mask-image: url("mask.svg");
      -webkit-mask-size: 668px 376px;
    }
    .bullet {
      position: absolute;
      font-size: 20px;
    }
  </style>
</head>
<body>
<div class="video">
  <div class="bullet" style="left: 100px; top: 0;">元芳，你怎么看</div>
  <div class="bullet" style="left: 200px; top: 20px;">你难道就是传说中的奶灵</div>
  <div class="bullet" style="left: 300px; top: 40px;">你好，我是胖灵</div>
  <div class="bullet" style="left: 400px; top: 60px;">这是第一集，还没有舔灵</div>
</div>
</body>
</html>
```
mask-image的其他属性
``` 
/* Keyword value */
mask-image: none;

/* <mask-source> value */
mask-image: url(masks.svg#mask1);

/* <image> values */
mask-image: linear-gradient(rgba(0, 0, 0, 1.0), transparent);
mask-image: image(url(mask.png), skyblue);

/* Multiple values */
mask-image: image(url(mask.png), skyblue), linear-gradient(rgba(0, 0, 0, 1.0), transparent);

/* Global values */
mask-image: inherit;
mask-image: initial;
mask-image: unset;
```

原文:  
[为什么B站的弹幕可以不挡人物](https://juejin.cn/post/7141012605535010823)
