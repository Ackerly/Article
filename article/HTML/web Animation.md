# Web Animation
前端动画的实现有三种:  
1. CSS动画
2. (requestAnimation/setTimeout/setInterval + 属性改变) 动画
3. Canvas动画

**落球**  
``` 
const ballEl = document.querySelector(".ball");
ballEl.addEventListener("click", function () {
            let fallAni = ballEl.animate({
                transform: ['translate(0, 0)', 'translate(20px, 8px)', 'translate(50px, 200px)']
            }, {
                easing: "cubic-bezier(.68,.08,.89,-0.05)",
                duration: 2000,
                fill: "forwards"
            })
});
```
**直播的世界消息或者弹幕**  
我们需要消息先运动到屏幕中间，消息最少需要在停留2秒，如果消息过长，消息还需要 匀速滚动 ，之后再滑出屏幕。  
难点就在于，暂停阶段，消息滚动的时间并不是确定的，需要计算。 这个时候，纯CSS3的动画，难度就有些高了，采用 Web Animations API，天然的和JS亲和，那就简单多了。
``` 
async function startAnimate() {
    // 滑入
    const totalWidth = stageWidth + DANMU_WITH;
    const centerX = stageWidth * 0.5 - DANMU_WITH * 0.5;
    const kfsIn = {
        transform: [`translateX(${totalWidth}px)`, `translateX(${centerX}px)`]
    }
    await danmuEl.animate(kfsIn, {
        duration: 2000,
        fill: 'forwards',
        easing: 'ease-out'
    }).finished;

    // 暂停部分
    const contentEl = danmuEl.querySelector(".danmu-content");
    const itemWidth = contentEl.getBoundingClientRect().width;
    const gapWidth = Math.max(0, itemWidth - DANMU_WITH);
    const duration = Math.max(0, Math.floor(gapWidth / 200) * 1000);

    const translateX = duration > 0 ? gapWidth : 0;
    const kfsTxt = {
        transform: [`translateX(0px)`, `translateX(-${gapWidth}px)`]
    };
    await contentEl.animate(kfsTxt, {
        duration,
        delay: 2000,
        fill: 'forwards',
        easing: 'linear',
    }).finished;

    // 滑出
    const kfsOut = {
        transform: [`translateX(${centerX}px)`, `translateX(-${DANMU_WITH}px)`]
    };
    await danmuEl.animate(kfsOut, {
        duration: 2000,
        fill: "forwards",
        easing: 'ease-in'
    }).finished;

    if (danmuEl) {
        stageEl.removeChild(danmuEl);
    }
    isAnimating = false
}
```

## web Animations API 两个核心的对象
1. KeyframeEffect 描述动画属性
2. Animation 控制播放

**KeyframeEffect**  
描述动画属性的集合，调用keyframes及 Animation Effect Timing Properties。 然后可以使用 Animation 构造函数进行播放。  
三种构建方式：  
``` 
new KeyframeEffect(target, keyframes);
new KeyframeEffect(target, keyframes, options)
new KeyframeEffect(source)
```
可以显示的去创建 KeyframeEffect, 然后交付给Animation去播放。 但是通常不需要这么做， 有更加简单的API， 
``` 
const box1ItemEl = document.querySelector(".box1");
const box2ItemEl = document.querySelector(".box2");

const kyEffect = new KeyframeEffect(null, {
    transform: ['translateX(0)', 'translateX(200px)']
},
{ duration: 3000, fill: 'forwards' })

const ky1 = new KeyframeEffect(kyEffect);
ky1.target = box1ItemEl;

const ky2 = new KeyframeEffect(kyEffect);
ky2.target = box2ItemEl;

new Animation(ky1).play();
new Animation(ky2).play();
```
**Animation**  
提供播放控制、动画节点或源的时间轴。 可以接受使用 KeyframeEffect 构造函数创建的对象作为参数。
``` 
const box1ItemEl = document.querySelector(".box1");

const kyEffect = new KeyframeEffect(box1ItemEl, {
    transform: ['translateX(0)', 'translateX(200px)']
},
{ duration: 3000, fill: 'forwards' })

const ani1 = new Animation(kyEffect);
ani1.play();
```
常用的方法：  
- cancel() 取消
- finish() 完成
- pause() 暂停
- play() 播放
- reverse() 逆转播放方向

Animation 事件监听
1. event 方式

因其继承于EventTarget，所有依旧有两种形式
``` 
animation.onfinish = function() {
  element.remove();
}

animation.addEventListener("finish", function() {
  element.remove();
}
```
2. Promise形式
``` 
animation.finished.then(() =>
  element.remove()
)
```
常用事件回调
- oncancel 取消
- onfinish 完成
- onremove 删除

## 便捷的 Element.animate
任何 Element都具备该方法， 其语法：
> animate(keyframes, options)

其参数和 new KeyframeEffect(target, keyframes, options)的后两个参数基本一样， 返回的是一个Animation对象。  
**第一个参数 keyframes**  
keyframes有两种形式，一种是数组形式，一种是对象形式。  
- 数组形式

一组对象(关键帧) ，由要迭代的属性和值组成。  
关键帧的偏移可以通过提供一个offset来指定 ，值必须是在 [0.0, 1.0] 这个区间内，且须升序排列。简单理解就是进度的百分比的小数值。  
``` 
element.animate([ { opacity: 1 },
                  { opacity: 0.1, offset: 0.7 },
                  { opacity: 0 } ],
                2000);
```
并非所有的关键帧都需要设置offset。 没有指定offset的关键帧将与相邻的关键帧均匀间隔  

- 对象形式  

一个包含key-value键值的对象需要包含动画的属性和要循环变化的值数组。
``` 
element.animate({
  opacity: [ 0, 0.9, 1 ],
  offset: [ 0, 0.8 ], // [ 0, 0.8, 1 ] 的简写
  easing: [ 'ease-in', 'ease-out' ],
}, 2000);
```

**第二个参数 options**  
和new KeyframeEffect(target, keyframes, options)的第三个参数基本一致，但是多了一个可选属性，就是id，用来标记动画，也方便 在Element.getAnimations结果中精确的查找。  

|  参数名   | 含义  |
|  ----  | ----  |
| delay  | 延迟动画开始的毫秒数。默认为0。 |
| direction  | 动画运动方向 |
| duration  | 动画每次迭代完成所需的毫秒数。默认为0 |
| easing  | 动画曲线函数，可以自定义 |
| endDelay  | 动画结束后要延迟的毫秒数。这主要用于基于另一个动画的结束时间对动画进行排序。默认为0。 |
| fill  | 动画结束后属性值的状态 |
| iterationStart  | 描述动画应该在迭代的什么时候开始。0.5表示在第一次迭代中途开始，使用这个值集，一个有两次迭代的动画将在第三次迭代中途结束。默认为0.0 |
| iterations  | 动画应该重复的次数。默认值为1，也可以取一个值 Infinity，使其在元素存在期间重复。 |
| composite  | 动画和其他单独的动画之间组合。 这是个高级特性，默认是replace，就是替换提起的动画。 |
| iterationComposite  | 动画的属性值变化如何在每次动画迭代时累积或相互覆盖 |

## Element.getAnimations  
通过Element.animate或者创建Animation给Element添加很多动画，通过这个方法可以获得所有Animation的实例。  
批量暂停动画：
``` 
box1ItemEl.getAnimations()
    .forEach(el=> el.pause()) // 暂停全部动画
```

## 优势
- 相对css动画更加灵活
- 相对requestAnimation/setTimeout/setInterval 动画，性能更好，代码更简洁
- 天然支持Promise

## 对比 CSS Animation
|  Web Animation API   | CSS  |
|  ----  | ----  |
| delay  | animation-delay |
| duration  | animation-duration |
| iterations  | animation-iteration-count |
| direction  | animation-direction |
| easing  | animation-timing-function |
| fill  | animation-fill-mode |

**参数设置值上的区别**  
1. duration 参数只支持毫秒
2. 迭代次数无限使用的是 JS的Infinity，不是字符串 "infinite"
3. 默认动画的贝塞尔是linear，而不是css的ease

原文:  
[复杂web动画，不慌，选择 web Animations API](https://juejin.cn/post/7023903552946995214)
