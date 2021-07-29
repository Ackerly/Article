# CSS斜线实现
## CSS3旋转缩放
使用伪元素画出一条直线，围绕div中心旋转45deg，再缩放一下就可以得到.  
HTML
``` 
<div></div>
```
CSS
``` 
HTML SCSSResult Skip Results Iframe
EDIT ON
div{
  position:relative;
  margin:50px auto;
  width:100px;
  height:100px;
  box-sizing:border-box;
  border:1px solid #333;  
  // background-color:#333;
  line-height:120px;
  text-indent:5px;
}

div::before{
  content:"";
  position:absolute;
  left:0;
  top:0;
  width:100%;
  height:50px;
  box-sizing:border-box;
  border-bottom:1px solid deeppink;
  transform-origin:bottom center;
  // transform:rotateZ(45deg) scale(1.414);
  animation:slash 5s infinite ease;
}

@keyframes slash{
  0%{
    transform:rotateZ(0deg) scale(1);
  }
  30%{
    transform:rotateZ(45deg) scale(1);
  }
  60%{
    transform:rotateZ(45deg) scale(1.414);
  }
  100%{
    transform:rotateZ(45deg) scale(1.414);
  }
}
```
## 线性渐变
使用背景的线性渐变，选定线性渐变的方向为45deg，依次设置为transparent -> deeppink -> deeppink ->transparent。
``` 
div{
  background:
    linear-gradient(45deg,transparent 49.5%, deeppink49.5%, deeppink50.5%,transparent 50.5%);
}
```
## 伪元素 + 三角形
利用CSS border实现三角形，画出带下不一样的三角形，通过叠加一起的方式，实现一条斜线
``` 
body{
  background:#eee;
}
div{
  position:relative;
  margin:50px auto;
  width:100px;
  height:100px;
  box-sizing:border-box;
  border:1px solid #333;  
  background:#fff;
  line-height:120px;
  text-indent:5px;
}

div::before{
  content:"";
  position:absolute;
  left:0;
  bottom:0;
  width:0;
  height:0;
  border:49px solid transparent;
  border-left:49px solid deeppink;
  border-bottom:49px solid deeppink;
  animation:slash 6s infinite ease;
}

div::after{
  content:"";
  position:absolute;
  left:0;
  bottom:0;
  width:0;
  height:0;
  border:48px solid transparent;
  border-left:48px solid #fff;
  border-bottom:48px solid #fff;
  animation:slash2 6s infinite ease;
}

@keyframes slash{
  0%{
    transform:translate(-50px);
  }
  30%{
    transform:translate(0px);
  }
  100%{
    transform:translate(0px);
  }
}
@keyframes slash2{
  0%{
    transform:translate(-100px);
  }
  30%{
    transform:translate(-100px);
  }
  60%{
    transform:translate(0px);
  }
  100%{
    transform:translate(0px);
  }
}
```
## clip-path
使用clip-path的polygon(0 0, 0 100px, 100px 100px, 0 0)画出一个区域，加上两个伪元素画出两个三角形
``` 
body{
  background:#eee;
}
div{
  position:relative;
  margin:50px auto;
  width:100px;
  height:100px;
  box-sizing:border-box;
  // border:1px solid deeppink;  
  background-color:deeppink;
  line-height:120px;
  text-indent:5px;
}

div::before{
  content:"";
  position:absolute;
  left:0px;
  top:0;
  right:0;
  bottom:0;
  -webkit-clip-path: polygon(0 0, 0 100px, 100px 100px, 0 0);
  background:#fff;
  border:1px solid #333;
  transform:translateX(-120px);
  animation:move 5s infinite linear;
}

div::after{
  content:"";
  position:absolute;
  left:0;
  top:0;
  right:0;
  bottom:0;
  -webkit-clip-path: polygon(100px 99px, 100px 0, 1px 0, 100px 99px);
  background:#fff;
  border:1px solid #333;
  transform:translateX(120px);
  animation:move 5s infinite linear;
}

@keyframes move{
  40%{
    transform:translateX(0px);
  }
  100%{
    transform:translateX(0px);
  }
}
```

参考：  
[巧妙的实现 CSS 斜线](https://www.cnblogs.com/coco1s/p/6026009.html)
