# css实现跳动的声波
## HTML
``` 
<div class="music">
    <div class="item one"></div> 
    <div class="item two"></div>
    <div class="item three"></div>
    <div class="item four"></div> 
    <div class="item five"></div> 
    <div class="item six"></div> 
    <div class="item seven"></div>        
</div>
```
## CSS
``` 
.item {
    position: absolute;
    width: 8px;
    border-radius: 6px;
    background-color: #1f94ea;
 
}  
```
通过 transform：translateX 属性 排列声波位置，让其进行间隔20px，按照 item * 20 间距来进行计算，高度根据喜好随意设置了，看起来不那么规律一些
``` 
.music .one {
    height: 18px;
    transform: translateX(-60px);
}
.music .two {
    height: 53px;
    transform: translateX(-40px);
}
.music .three {
    height: 36px;
    transform: translateX(-20px);
}
.music .four {
    height: 70px;
    transform: translateX(0);
}

.music .five {
    height: 30px;
    transform: translateX(20px);
}

.music .six {
    height: 40px;
    transform: translateX(40px);
}
.music .seven {
    height: 50px;
    transform: translateX(60px);
}
```
使用animation 属性, 在其 100% 的时候，设置高度为10px，并增加一定的对比度
``` 
@keyframes radius-animation {
    100% {
        height: 10px;
        border-radius: 50%;
        filter: contrast(2);
    }
}
```
给 item 设置animation 属性，动画时间根据想要波动的速度来进行设置，动画的速度这里是用 ease
``` 
        .music .one {
            animation: radius-animation .3s ease;
        }
        .music .two {
            animation: radius-animation .6s ease;
        }
        .music .three {
            animation: radius-animation .57s ease;
        }
        .music .four {
            animation: radius-animation .52s ease;
        }
        .music .five {
            animation: radius-animation .4s ease;
        }
        .music .six {
            animation: radius-animation .3s ease;
        }
        .music .seven {
            animation: radius-animation .7s ease;
        }

```

原文:  
[css实现跳动的声波](https://juejin.cn/post/7025129069603880991)
