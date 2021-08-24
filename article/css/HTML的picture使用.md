# HTML <picture>元素使用
## 概述
HTML <picture>元素还是挺实用的，往往和<source>元素（可以多个）、<img>元素（最多一个）一同使用。渲染的时候，浏览器优先使用<source>元素，<img>元素兜底。需要在不同场景显示不同图片的时候，<picture>元素就特别的实用。
``` 
// 宽度小于640像素的时候，显示的是正方向图片，如果浏览器的宽度大于640px，会显示长方形图片
<picture>
  <source srcset="rect.png" media="(min-width: 640px)">
  <img src="square.png">
</picture>
```
``` 
// 不同宽度、不同屏幕密度显示不同图片
<picture>
  <source srcset="128px.jpg, 256px.jpg 2x, 512px.jpg 3x">
  <img src="128px.jpg">
</picture>
```
``` 
不同浏览器显示不同后缀图片
<picture>
  <source srcset="zxx.avif" type="image/avif">
  <source srcset="zxx.webp" type="image/webp">
  <img src="zxx.jpg">
</picture>
```
## 缺点
1. <picture>元素使用的最大问题是，无论什么场景，我们都需要准备多份的素材。
2.不同宽度显示不同图片不如@media查询一句搞定的，实时渲染响应，维护方便，代码干净
3.不同设备密度显示不同图片不如CSS Medias Query，无需像picture一样进行嵌套
4.不同浏览器显示不同格式的图片通常交给云服务厂商处理，通常COS服务支持通过配置让Chrome等浏览器显示webp，IE、Safari等浏览器显示jpg的。
5.
## 适合场景
<picture>元素并不太适合那种长期迭代维护，以功能开发为主，项目复杂度比较高的项目。更适合类似集团官网、运营活动这种独立的简单的几个页面的场景。可以专门对这些页面使用<picture>元素进行优化，容易出结果，容易获得好的数据，也能体现前端专业性。  
而且这些项目本身不复杂，就几个页面，正好不需要那种工程化，自动化的东西，此时，手工对HTML、图像资源进行处理反而合适，投入产出比高。
