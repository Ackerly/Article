# 前端动画必知必会：React 和 Vue 都在用的 FLIP 思想实战
## FLIP
**First**  
即将做动画的元素的初始状态（比如位置、透明度等等）  
**Last**  
即将做动画的元素的最终状态  
**Invert**  
假设我们图片的初始位置是 左: 0, 上：0，元素动画后的最终位置是 左：100, 上100，那么很明显这个元素是向右下角运动了 100px。  
但是，此时我们不按照常规思维去先计算它的最终位置，然后再命令元素从 0, 0 运动到 100, 100，而是先让元素自己移动过去（比如在 Vue 中用数据来驱动，在数组前面追加几个图片，之前的图片就自己移动到下面去了）。  
DOM 元素属性的改变（比如 left、right、 transform 等等），会被集中起来延迟到浏览器的下一帧统一渲染，所以我们可以得到一个这样的中间时间点：DOM 状态（位置信息）改变了，而浏览器还没渲染。  
有了这个前置条件，我们就可以保证先让 Vue 去操作 DOM 变更，此时浏览器还未渲染，我们已经能得到 DOM 状态变更后的位置了。  
假设我们的图片是一行两个排列，图片数组初始化的状态是 [img1, img2，此时我们往数组头部追加两个元素 [img3, img4, img1, img2]，那么 img1 和 img2 就自然而然的被挤到下一行去了。  
假设 img1 的初始位置是 0, 0，被数据驱动导致的 DOM 改变挤下去后的位置是 100, 100，那么此时浏览器还没有渲染，我们可以在这个时间点把 img1.style.transform = translate(-100px, -100px)，让它 先 Invert 倒置回位移前的位置。
**Play**
倒置了以后，想要让它做动画就很简单了，再让它回到 0, 0 的位置即可，本文会采用最新的 Web Animation API 来实现最后的 Play。
## 实现
``` 
.wrap {
  display: flex;
  flex-wrap: wrap;
}

.img {
  width: 25%;
}

<div v-else class="wrap">
  <div class="img-wrap" v-for="src in imgs" :key="src">
    <img ref="imgs" class="img" :src="src" />
  </div>
</div>
```
实现追加图片的方法 add
``` 
async add() {
  const newData = this.getSister()
  await preload(newData)
}
```
随机的取出几张图片作为待放入数组的元素，利用 new Image 预加载这些图片，防止渲染一堆空白图片到屏幕上。  
然后定义一个计算一组 DOM 元素位置的函数 getRects，利用 getBoundingClientRect 可以获得最新的位置信息，这个方法在接下来获取图片元素旧位置和新位置时都要使用。  
``` 
function getRects(doms) {
  return doms.map((dom) => {
    const rect = dom.getBoundingClientRect()
    const { left, top } = rect
    return { left, top }
  })
}

// 当前已有的图片
const prevImgs = this.$refs.imgs.slice()
const prevPositions = getRects(prevImgs)
```
记录完图片的旧位置后，就可以向数组里追加新的图片了
``` 
this.imgs = newData.concat(this.imgs)
```
Vue 是异步渲染的，也就是改变了这个 imgs 数组后不会立刻发生 DOM 的变动，此时我们要用到 nextTick 这个 API，这个 API 把你传入的回调函数放进了 microTask 队列,microTask队列的执行一定发生在浏览器重新渲染前。  
先调用了 this.imgs = newData.concat(this.imgs) 这段代码，触发了 Vue 的响应式依赖更新，此时 Vue 内部会把本次 DOM 更新的渲染函数先放到 microTask队列中，此时的队列是[changeDOM]。  
调用了 nextTick(callback) 后，这个callback函数也会被追加到队列中，此时的队列是 [changeDOM, callback]。  
由于我们之前保存了图片元素节点的数组 prevImgs，所以在 nextTick 里调用同样的 getRect 方法获取到的就是旧图片的最新位置了。  
``` 
async add() {
  // 最新 DOM 状态
  this.$nextTick(() => {
    // 再调用同样的方法获取最新的元素位置
    const currentPositions = getRects(prevImgs)
  })
},
```
此时我们已经拥有了 Invert 步骤的关键信息，新位置和旧位置，那么接下来就很简单了，把图片数组循环做一个倒置后 Play的动画即可。
``` 
prevImgs.forEach((imgRef, imgIndex) => {
  const currentPosition = currentPositions[imgIndex]
  const prevPosition = prevPositions[imgIndex]

  // 倒置后的位置，虽然图片移动到最新位置了，但你先给我回去，等着我来让你做动画。
  const invert = {
    left: prevPosition.left - currentPosition.left,
    top: prevPosition.top - currentPosition.top,
  }

  const keyframes = [
    // 初始位置是倒置后的位置
    {
      transform: `translate(${invert.left}px, ${invert.top}px)`,
    },
    // 图片更新后本来应该在的位置
    { transform: "translate(0)" },
  ]

  const options = {
    duration: 300,
    easing: "cubic-bezier(0,0,0.32,1)",
  }

  // 开始运动！
  const animation = imgRef.animate(keyframes, options)
})
```
## 乱序
动画的逻辑：  
保存旧位置 -> 改变数据驱动视图更新 -> 获得新位置 -> 利用 FLIP 做动画   
其实外部只需要传入一个 update 方法告诉我们如何去更新图片数组，就可以把这个逻辑完全抽象到一个函数里去。
``` 
scheduleAnimation(update) {
  // 获取旧图片的位置
  const prevImgs = this.$refs.imgs.slice()
  const prevSrcRectMap = createSrcRectMap(prevImgs)
  // 更新数据
  update()
  // DOM更新后
  this.$nextTick(() => {
    const currentSrcRectMap = createSrcRectMap(prevImgs)
    Object.keys(prevSrcRectMap).forEach((src) => {
      const currentRect = currentSrcRectMap[src]
      const prevRect = prevSrcRectMap[src]

      const invert = {
        left: prevRect.left - currentRect.left,
        top: prevRect.top - currentRect.top,
      }

      const keyframes = [
        {
          transform: `translate(${invert.left}px, ${invert.top}px)`,
        },
        { transform: "" },
      ]
      const options = {
        duration: 300,
        easing: "cubic-bezier(0,0,0.32,1)",
      }

      const animation = currentRect.img.animate(keyframes, options)
    })
  })
}
```
追加图片和乱序的函数
``` 
// 追加图片
async add() {
  const newData = this.getSister()
  await preload(newData)
  this.scheduleAnimation(() => {
    this.imgs = newData.concat(this.imgs)
  })
},
// 乱序图片
shuffle() {
  this.scheduleAnimation(() => {
    this.imgs = shuffle(this.imgs)
  })
}
```
## Web Animation
利用 Web Animation API 可以让我们用 JavaScript 更加直观的描述我们需要元素去做的动画，想象一下这个需求如果用 CSS 来做，我们大概会这样去完成这个需求：
``` 
const currentImgStyle = currentRect.img.style
currentImgStyle.transform = `translate(${invert.left}px, ${invert.top}px)`
currentImgStyle.transitionDuration = "0s"

this._reflow = document.body.offsetHeight

currentRect.img.classList.add("move")

currentImgStyle.transform = currentRect.img.style.transitionDuration = ""

currentRect.img.addEventListener("transitionend", () => {
  currentRect.img.classList.remove("move")
})
```
这也是 Vue 内部 transition-group 组件实现 FLIP 动画的大致思路，Vue 应该是为了兼容性和代码体积等一些方面的权衡，还是选择用比较原生的方式去实现 FLIP 动画,这段代码让不好的点在于：
1. 需要通过 class 的增加和删除来和 CSS 来进行交互，整体流程不太符合直觉
2. 需要监听动画完成事件，并且做一些清理操作，容易遗漏。
3. 需要利用 document.body.offsetHeight 这样的方式触发 强制同步布局，比较 hack 的知识点。
4. 需要利用 this._reflow = document.body.offsetHeight 这样的方式向元素实例上增加一个没有意义的属性，防止被 Rollup 等打包工具 tree-shaking 误删。

利用 Web Animation API 的代码则变得非常符合直觉和易于维护：  
``` 
const keyframes = [
  {
    transform: `translate(${invert.left}px, ${invert.top}px)`,
  },
  { transform: "" },
]
const options = {
  duration: 300,
  easing: "cubic-bezier(0,0,0.32,1)",
}

const animation = currentRect.img.animate(keyframes, options)
```
