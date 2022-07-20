# 妙用 CSS 构建花式透视背景效果
在页面的的滚动过程中，顶栏的背景不是白色的，也不是毛玻璃效果，而是能够将背景颗粒化：准确而言，是一种基于颗粒化的毛玻璃效果，元素首先是被颗粒化，其次，元素的边缘也是在一定程度上被虚化了。那么，我们该如何实现这个效果呢？  
**需求拆解**  
上述效果看似神奇，其实原理也非常简单。主要就是颗粒化的背景 background加上backdrop-filter: blur() 即可  
background 实现这样一个背景：  
``` 
<div></div>  
div {
    background: radial-gradient(transparent, #000 20px);
    background-size: 40px 40px;
}
```
实现从透明到黑色的径向渐变效果
将 background: radial-gradient(transparent, #000 20px) 中的黑色替换成白色  
每个径向渐变的圆设置的比较大，把它调整回正常大小：  
``` 
div {
    background: radial-gradient(transparent, rgba(255, 255, 255, 1) 2px);
    background-size: 4px 4px;
}
```
这样，就成功的将背景颗粒化  
此时透出的背景看上去非常生硬，也不美观，还需要 backdrop-filter: blur()  
``` 
div {
    background: radial-gradient(transparent, rgba(255, 255, 255, 1) 2px);
    background-size: 4px 4px;
    backdrop-filter: blur(10px);
}
```
这里需要注意的是，background-size 的大小控制，和不同的 backdrop-filter: blur(10px) 值，都会影响效果

参考:  
[妙用 CSS 构建花式透视背景效果](https://mp.weixin.qq.com/s/eHwFbX_C-NjV3XRNORspew)
