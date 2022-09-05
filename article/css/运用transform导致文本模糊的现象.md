# 运用transform导致文本模糊现象
在页面中，经常会出现这样的问题，一块区域内的文本或者边框，在展示的时候，变得特别的模糊
**何时触发?**  
1. 当文本元素的某个祖先容器存在 transform: translate() 或者 transform: scale() 等 transform 操作时，容易出现这种问题(只是必要条件，不是充分条件)
2. 元素作用了 transform: translate() 或者 transform: scale() 后的计算值产生了非整数，
3. 文本内容是否模糊还与屏幕有关，高清屏（dpr > 2）下不容易触发，更多发生在普通屏幕下（dpr = 1）
4. 并非所有浏览器都是这个表现，基本发生在 chromium 内核。

触发的 CSS 代码如下：
```
.container {
    position: absolute;
    width: 1104px; 
    height: 475px;
    top: 50%;
    transform: translateY(-50%);
    // ...
}
```
由于元素的高度为 475px，translateY(-50%) 等于 237.5px，非整数，才导致了内部的字体模糊。并非所有产生的非整数都会导致了内部的字体模糊。  
**为何发生这种现象**  
普遍的认为是因为：  
由于浏览器将图层拆分到 GPU 以进行 3D 转换，而非整数的像素偏移，使得 Chrome 在字体渲染的时候，不是那么的精确。
**如何解决？**  
1. 社区里给出的一种方案：
   给元素设置 -webt-font-smoothing: antialiased,font-smooth CSS 属性用来控制字体渲染时的平滑效果，该特性是非标准的，我们应该尽量不要在生产环境中使用它。并且在我的实测中，这个方法不太奏效。
2. 保证运用了 transform: translate() 或者 transform: scale() 的元素的高宽为偶数
3. 弃用 transform

原文: 
[疑难杂症：运用 transform 导致文本模糊的现象探究](https://juejin.cn/post/7066986698575446030?utm_source=gold_browser_extension)