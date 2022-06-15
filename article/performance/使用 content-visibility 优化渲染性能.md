# 使用 content-visibility 优化渲染性能
**何为 content-visibility**  
content-visibility：属性控制一个元素是否渲染其内容，它允许用户代理（浏览器）潜在地省略大量布局和渲染工作，直到需要它为止  
有几个常见的取值  
``` 
content-visibility: visible;
content-visibility: hidden;
content-visibility: auto;
```
- content-visibility: visible：默认值，没有任何效果，相当于没有添加 content-visibility，元素的渲染与往常一致。
- content-visibility: hidden：与 display: none 类似，用户代理将跳过其内容的渲染。（这里需要注意的是，跳过的是内容的渲染）
- content-visibility: auto：如果该元素不在屏幕上，并且与用户无关，则不会渲染其后代元素。

参考:  
[使用 content-visibility 优化渲染性能](https://mp.weixin.qq.com/s/webv8u3M43Jy3BVCxwEQFg)
