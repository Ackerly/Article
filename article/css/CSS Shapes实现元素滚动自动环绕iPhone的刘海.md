# CSS Shapes实现元素滚动自动环绕iPhone的刘海
## CSS3 Shapes实现元素滚动自动环绕iPhone X头部刘海效果
CSS Shapes中有个CSS属性名为shape-outside，可以让内联元素以不规则的形状进行外部排列，其语法如下
``` 
/* 关键字值 */
shape-outside: none;
shape-outside: margin-box;
shape-outside: content-box;
shape-outside: border-box;
shape-outside: padding-box;

/* 函数值 */
shape-outside: circle();
shape-outside: ellipse();
shape-outside: inset(10px 10px 10px 10px);
shape-outside: polygon(10px 10px, 20px 20px, 30px 30px);

/* <url>值 */
shape-outside: url(image.png);

/* 渐变值 */
shape-outside: linear-gradient(45deg, rgba(255, 255, 255, 0) 150px, red 150px);
```
shape-outside属性要想生效，本身需要是浮动float元素  
使用shape-outside:polygon()，通过点坐标勾勒出和齐刘海形状相似的多边形形状，CSS代码为：  
``` 
.shape {
  float: left;
  shape-outside: polygon(0 0, 0 150px, 16px 154px, 30px 166px, 30px 314px, 16px 326px, 0 330px, 0 0);
}
```
后面没有设置BFC（块状格式化上下文）的列表元素就会自动环绕这个形状排列，也就是自动避开了齐刘海区域。搞个假的iPhone X的齐刘海图片覆盖在区域上就可以了。  
**如何让滚动的时候，列表元素动态的跟着环绕呢**  
由于shape-outside所在的元素是浮动元素，因此，必定会跟着容器一起滚动，需要的效果是所绘制的这个刘海区域需要是固定的，可以借助JavaScript处理。
监听容器的滚动事件，让shape-outside绘制的区域实时偏移滚动的大小。此时肉眼看上去的效果就是shape-outside区域永远固定在了滚动容器clientHeight的中间。 
``` 
box.addEventListener('scroll', function () {
  var scrollTop = box.scrollTop;
  // 滚动偏移应用在shape-outside上
  shape.style.shapeOutside = 'polygon(0 0, 0 '+ (150 + scrollTop) +'px, 16px '+ (154 + scrollTop) +'px, 30px '+ (166 + scrollTop) +'px, 30px '+ (314 + scrollTop) +'px, 16px '+ (326 + scrollTop) +'px, 0 '+ (330 + scrollTop) +'px, 0 0)';
});
``` 
## 其他方法
如果技术选型是更看重简单易懂，而不是资源消耗与占用，还可以使用shape-outside:url(image.png)语法实现类似的效果，其中'image.png'就是用来被环绕的图片，环绕与否是基于计算alpha通道决定，用句简单的话描述，就是沿着图片非透明区域环绕。  
由于使用url()的形状计算是基于图片元素，和inset(), circle(), ellipse()或者polygon()这些基础形状方法的计算性质不一样，因此，可以直接使用垂直方向的margin进行偏移。这要比polygon()这样实时计算坐标位置要好理解的多。  
``` 
.shape {
  float: left;
  shape-outside: url(liu-outside.png);
  margin-top: 150px;
}

box.addEventListener('scroll', function () {
  var scrollTop = box.scrollTop;
  // 滚动偏移应用在margin-top上
  shape.style.marginTop = (150 + scrollTop) + 'px';
});
```
可以看到，当滚动容器的时候，改变的就一个marginTop值就好了；而上面的 shape-outside:polygon()实现需要同时改变多个坐标值。  
**有个细节说明**  
作为环绕区域的图片和前面显示的那个刘海图片不是一张图片，因为刘海区域需要和后面的文字有一段的间隙，因此，url(liu-outside.png)中的这张'liu-outside.png'图片是有特别的实色填充处理的  
## 兼容性  
CSS Shapes的兼容性为Chrome浏览器和Safari浏览器（包括iOS）都是支持的，也就意味是可以在iPhone上使用的，完美。只是需要注意的是在iOS10.2及其之前的版本，CSS Shapes的使用还是需要加webkit私有前缀的，但据说iPhone X至少默认iOS 11，而刘海头交互效果就是针对iPhone X处理的，因此webkit私有前缀不加也没关系。


参考:  
[借助CSS Shapes实现元素滚动自动环绕iPhone X的刘海](https://juejin.cn/post/6844903496290926605?utm_source=gold_browser_extension)
