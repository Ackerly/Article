# html2canvas遇到的问题
## html2canvas生成海报后清晰度问题
原因:  
通过background-image引入了图片资源 导致转化为canvas时候就会不清晰
解决方式:  
采用了image定位+层级的方法，直接通过img标签将图片引入，并通过定位将图片准确定位到视图中，通过层级展示为具体的背景效果
继续优化:  
html2canvas最后得到的是一个base64图片，而canvas生成base64图片那显然是用了canvas.toDataUrl,这个api最高的质量只能保证到0.92，可以将页面中的图片元素质量提高从而达到，生成的image也随之提高的效果。并且在生成前将待生成的html预先设置为2倍宽高，从而使得页面整体放大2倍，因为生成后的是按原来宽高决定的，所以生成后可以通过transform scale属性去手动的放缩我们的图片，从而达到清晰效果。  
## html2canvas无法生成页面里面带canvas元素的html
通过canvas的toDataUrl的API去将canvas生成图片并渲染在html文档里，为了避免视觉差，将这一步放在了html2canvas前去进行渲染，等生成之后原封不动的将canvas标签还原
## html2canvas对某些css属性不支持问题
**高斯模糊**  
直接用filter:blur() 去实现模糊会导致背景模糊但是文字没有模糊，使用backdrop-filter：blur()属性处理

参考:
[使用html2canvas遇到的哪些坑](https://juejin.cn/post/7047745596366520351?utm_source=gold_browser_extension)
