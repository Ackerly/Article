# 浏览器的五种Observer
## IntersectionObserver
IntersectionObserver 可以监听一个元素和可视区域相交部分的比例，然后在可视比例达到某个阈值的时候触发回调。
``` 
// html
<div id="box1">BOX111</div>
<div id="box2">BOX222</div>

// CSS
#box1,#box2 {
    width: 100px;
    height: 100px;
    background: blue;
    color: #fff;

    position: relative;
}
#box1 {
    top: 500px;
}
#box2 {
    top: 800px;
}

// JS
const intersectionObserver = new IntersectionObserver(
    function (entries) {
        console.log('info:');
        entries.forEach(item => {
            console.log(item.target, item.intersectionRatio)
        })
    }, {
    threshold: [0.5, 1]
});

intersectionObserver.observe( document.querySelector('#box1'));
intersectionObserver.observe( document.querySelector('#box2'));
```
创建一个 IntersectionObserver 对象，监听 box1 和 box2 两个元素，当可见比例达到 0.5 和 1 的时候触发回调。  
元素 box1 和 box2 在可视范围达到一半（0.5）和全部（1）的时候分别触发了回调。  
在做一些数据采集的时候，希望知道某个元素是否是可见的，什么时候可见的，就可以用这个 api 来监听，还有做图片的懒加载的时候，可以当可视比例达到某个比例再触发加载。  
## MutationObserver
可以监听对元素的属性的修改、对它的子节点的增删改.  
``` 
// html
<div id="box"><button>光</button></div>
// CSS
 #box {
    width: 100px;
    height: 100px;
    background: blue;

    position: relative;
}
// JS
setTimeout(() => {
    box.style.background = 'red';
},2000);

setTimeout(() => {
    const dom = document.createElement('button');
    dom.textContent = '东东东';
    box.appendChild(dom);
},3000);

setTimeout(() => {
   document.querySelectorAll('button')[0].remove();
},5000);
```
2s 的时候修改背景颜色为红色，3s 的时候添加一个 button 的子元素，5s 的时候删除第一个 button。然后监听它的变化：  
``` 
const mutationObserver = new MutationObserver((mutationsList) => {
    console.log(mutationsList)
});

mutationObserver.observe(box, {
    attributes: true,
    childList: true
});
```
三次变化的时候都监听到了并打印了一些信息：
1. 第一次改变的是 attributes，属性是 style
2. 第二次改变的是 childList，添加了一个节点
3. 第三次也是改变的 childList，删除了一个节点

这个可以用来做什么呢？比如文章水印被人通过 devtools 去掉了，那么就可以通过 MutationObserver 监听这个变化，然后重新加上，让水印去不掉。
## ResizeObserver
元素可以用 ResizeObserver 监听大小的改变，当 width、height 被修改时会触发回调。  
``` 
// html
<div id="box"></div>
// CSS
#box {
    width: 100px;
    height: 100px;
    background: blue;
}
// JS
const box = document.querySelector('#box');

setTimeout(() => {
    box.style.width = '200px';
}, 3000);
// ResizeObserver 监听它的变化
const resizeObserver = new ResizeObserver(entries => {
    console.log('当前大小', entries)
});
resizeObserver.observe(box);
```
大小变化被监听到了,可以拿到元素和它的位置、尺寸。
## PerformanceObserver
PerformanceObserver 用于监听记录 performance 数据的行为，一旦记录了就会触发回调，就可以在回调里把这些数据上报。  
比如 performance 可以用 mark 方法记录某个时间点：
``` 
performance.mark('registered-observer');
```
用 measure 方法记录某个时间段：  
``` 
performance.measure('button clicked', 'from', 'to');
```
后两个个参数是时间点，不传代表从开始到现在。  
用 PerformanceObserver 监听它们：  
``` 
<html>
<body>
  <button onclick="measureClick()">Measure</button>

  <img src="https://p9-passport.byteacctimg.com/img/user-avatar/4e9e751e2b32fb8afbbf559a296ccbf2~300x300.image" />

  <script>
    const performanceObserver = new PerformanceObserver(list => {
      list.getEntries().forEach(entry => {
        console.log(entry);// 上报
      })
    });
    performanceObserver.observe({entryTypes: ['resource', 'mark', 'measure']});

    performance.mark('registered-observer');

    function measureClick() {
      performance.measure('button clicked');
    }
  </script>
</body>
</html>
```
创建 PerformanceObserver 对象，监听 mark（时间点）、measure（时间段）、resource（资源加载耗时） 这三种记录时间的行为。  
用 mark 记录了某个时间点，点击 button 的时候用 measure 记录了某个时间段的数据，还加载了一个图片。当这些记录行为发生的时候，希望能触发回调，在里面可以上报。  
mark 的时间点记录、资源加载的耗时、点击按钮的 measure 时间段记录都监听到了。
## ReportingObserver
当浏览器运行到过时（deprecation）的 api 的时候，会在控制台打印一个过时的报告。浏览器还会在一些情况下对网页行为做一些干预（intervention），比如会把占用 cpu 太多的广告的 iframe 删掉。会在网络比较慢的时候把图片替换为占位图片，点击才会加载。这些干预都是浏览器做的，会在控制台打印一个报告。  
这些干预或者过时的 api 并不是报错，所以不能用错误监听的方式来拿到，但这些情况对网页 app 来说可能也是很重要的：  
比如我这个网页就是为了展示广告的，但浏览器一干预给我把广告删掉了，我却不知道。如果我知道的话或许可以优化下 iframe。  
比如我这个网页的图片很重要，结果浏览器一干预给我换成占位图了，我却不知道。如果我知道的话可能会优化下图片大小。  
所以浏览器提供了 ReportingObserver 的 api 用来监听这些报告的打印，我们可以拿到这些报告然后上传。  
``` 
const reportingObserver = new ReportingObserver((reports, observer) => {
    for (const report of reports) {
        console.log(report.body);//上报
    }
}, {types: ['intervention', 'deprecation']});

reportingObserver.observe();
```

原文:  
[浏览器的 5 种 Observer，你用过几种？](https://juejin.cn/post/7064557881492209678?utm_source=gold_browser_extension)