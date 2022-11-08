# IntersectionObserver 实现虚拟列表
前端开发中经常会遇到大数据量列表展示的性能问题，即大数据量一次性展示时前端渲染大量 Dom，触发渲染性能问题，造成初始加载白屏，交互卡顿等。解决这类问题的方案也有很多，使用虚拟列表展示是一个比较常见的解决方案。  
## 传统列表
在未使用虚拟列表之前，传统列表很难处理大量数据的渲染问题，常出现以下情况：  
- 列表数据渲染时间长甚至出现白屏
- 列表交互卡顿

为了解决该类问题，可以选用虚拟列表来承载大量数据的渲染，增强用户体验，IntersectionObserver API 作为浏览器原生的 API，可以做到“观察”所需元素是否需要在页面上显示，以此来对大量数据的渲染进行优化。  

## 传统的实现方案
传统方法一般是监听 scroll, 在回调方案中 手动计算偏移量然后计算定位，由于 scroll 事件密集发生，计算量很大，容易造成性能问题。另外如果行行高不固定（实际业务中往往需要这样）, 那计算将会更加复杂。  
所有的这些计算都是为了判断一个 dom 是否在可视范围内，如果存在一个方法可以方便地让我们知道这点，那实现虚拟列表方案将大大简化。幸运的是目前大部分浏览器已经提供了这个api——IntersectionObserver  

## IntersectionObserver介绍
IntersectionObserver 接口 (从属于 Intersection Observer API) 提供了一种异步观察目标元素与其祖先元素或顶级文档视窗 (viewport) 交叉状态的方法。祖先元素与视窗 (viewport) 被称为根 (root)。  
当一个 IntersectionObserver 对象被创建时，其被配置为监听根中一段给定比例的可见区域。一旦 IntersectionObserver 被创建，则无法更改其配置，所以一个给定的观察者对象只能用来监听可见区域的特定变化值；然而，你可以在同一个观察者对象中配置监听多个目标元素。  
示例:  
``` 
var intersectionObserver = new IntersectionObserver(function(entries) {
  // If intersectionRatio is 0, the target is out of view
  // and we do not need to do anything.
  if (entries[0].intersectionRatio <= 0) return;

  loadItems(10);
  console.log('Loaded new items');
});
// start observing
intersectionObserver.observe(document.querySelector('.scrollerFooter'));
```
可以看到用法很简单：  
1. 首先new IntersectionObserver 构造函数，这个函数接受两个参数：callback 是可见性变化时的回调函数，option 是配置对象（该参数可选）, 然后就得到一个观察器实例
2. 调用实例的 observe 方法对目标 dom 元素进行监听
3. 在回调函数 callback 中拿到 entries， entries是一个数组，里面每个成员都是一个IntersectionObserverEntry对象，监听了几个元素， entries 就包含了几个成员。IntersectionObserverEntry对象描述了目标对象与容器之前的相交信息。其中 intersectionRatio 目标元素的可见比例，即intersectionRect占boundingClientRect的比例，完全可见时为 1，完全不可见时小于等于 0。在这里我们取 entries[0].intersectionRatio 来判断目标元素是否在视野中, 大于 0 代表在视野中，小于 0 表示已移出视野。  

## 使用 IntersectionObserver 实现虚拟列表方案
**基本思路**  
1. 实例化配置一个观察器，在这里除了传入回调函数外我们还会传入配置项：config = { root: document.querySelector('.main'), } 这样我们就设置了 class 为 main 的 dom 元素为容器
2. 监听列表的每一行元素
3. 在回调函数中拿到每一个行元素的 intersectionRatio，一次判断是否在可是区域内。如果进入视野则给这一行附上实际的数据进行渲染，如果移出视野则将这一行的数据置为空。此外为了定位准确，我们在元素移出视野时给一个实际渲染时的高度。

简化示例代码如下：  
``` 
<div class="main">
        <div v-for="(row, index0) in uiPeriodList" :key="index0">
          <div class="period" :style="periodStyle">
            <div
              v-for="(column, index1) in row.columnList"
              :key="column.id"
            
            >
                /* 详细展示元素 */
            </div>
          </div>
        </div>
</div>
```
``` 
update() {
    let rolwList = document.querySelectorAll('.period')
    let _this = this
    let config = {
       root: document.querySelector('.main'),
      }
      
     let intersectionObserver = new IntersectionObserver(function(entries) {
        entries.forEach((row)=> {
          if (row.intersectionRatio <= 0) {
        if (!_this.isFirst) {
         row.target.style.height = `${row.target.clientHeight}px`
        }
    
        _this.uiPeriodList[index].coordList = []
       } else {
        row.target.className = 'period'
        row.target.style.height = ''
        _this.uiPeriodList[index].columnList = _this.periodList[index].columnList // 附上实际元素
       }
        })
      }, config)
    if (this.isFirst) {
     rowList.forEach((row, index) => {
      intersectionObserver.observe(row)
     })
     this.isFirst = false
    }
}
```
**没有效果**  
实际测试却发现并没有达到预想效果，初始加载时仍然非常缓慢，出现长时间白屏。这是为什么呢？打印发现，初始时每一行的元素都进入了视野中，触发了附上实际数据的动作从而引发渲染。怀疑是初始加载元素时没有实际内容，导致大量的行元素没有高度而一下子直接进入了视野区，进而触发大数据量渲染。为了解决这个问题，我们在初始时给行元素设置一个非常大的行高，使得在视野中只存在一行，然后对这一行附上实际数据，去除行高样式，使行的高度由实际内容决定。这样可以使各个行依次进入视野，逐个渲染直到实际的高度的行元素撑满视野  
``` 
created() {
     this.periodStyle = {
    'grid-template-columns': `52px repeat(${this.headList.length}, 1fr)`,
    height: '2000px',
   }
}
```
``` 
let intersectionObserver = new IntersectionObserver(function(entries) {
       entries.forEach((row)=>{
        if (row.intersectionRatio <= 0) {
        if (!_this.isFirst) {
         row.target.style.height = `${row.target.clientHeight}px`
        }
    
        _this.uiPeriodList[index].coordList = []
       } else {
        row.target.style.height = ''
        _this.uiPeriodList[index].columnList = _this.periodList[index].columnList // 附上实际元素
       }
       })
       
      }, config)
```
**快速下拉出现空白行**  
解决了上面的问题，虚拟列表方案已基本实现，但还有瑕疵。当快速滚动列表时有可能出现空白区域，原因是监听回调是异步触发，不随着目标元素的滚动而触发，这样性能消耗很低，但也会导致回调函数没有执行，导致出现在视野中的元素但没有附上实际数据。
自然地想到增加冗余量来解决这个问题，在行元素还没出现在视野当中时就附上实际数据进行渲染。查看发现在初始化 IntersectionObserver 可以传入配置项 rootMargin,  rootMargin 定义根元素的 margin，用来扩展或缩小 rootBounds 这个矩形的大小，从而影响 intersectionRect 交叉区域的大小。这样就变相地达成在视野单位外就进行数据实际渲染的目的  
``` 
let config = {
     root: document.querySelector('.main'),
     rootMargin: '100px 0px',
    }

```


原文:  
[IntersectionObserver 实现虚拟列表初探](https://mp.weixin.qq.com/s/cf9EOPKyXVhtxbm9fsPpLA)
