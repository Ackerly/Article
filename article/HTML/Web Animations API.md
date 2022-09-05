# Web Animations API
JavaScript 中有一个用于动画的原生 API，称为 Web Animations API，简称为 WAAPI。  
目前浏览器的支持的能力是有限的，WAAPI 有一个全面和强大的 polyfill 工具，使得它可以在生产环境下使用。polyfill 是 Web 动画 API 的 JavaScript 实现，它在本机不支持它的浏览器中提供 Web 动画功能。当一个可用时，polyfill 回退到本机实现。  
## WAAPI基础
WAAPI 的基本语法：  
```
var element = document.querySelector('.animate-me');
element.animate(keyframes, 1000);
```
animate 方法接受两个参数：关键帧和持续时间。与 jQuery 相比，它不仅具有内置于浏览器的优点，而且性能更高。  
第一个参数，关键帧，应该是一个对象数组。每个对象都是我们动画中的关键帧。这是一个简单的例子：  
```
var keyframes = [
  { opacity: 0 },
  { opacity: 1 }
];
```
第二个参数，持续时间，是我们希望动画持续多长时间。在上面的例子中，它是 1000 毫秒。看一个有趣的例子:
```
// CSS关键帧
0% {
  transform: translateY(-1000px) scaleY(2.5) scaleX(.2);
  transform-origin: 50% 0;
  filter: blur(40px);
  opacity: 0;
}
100% {
  transform: translateY(0) scaleY(1) scaleX(1);
  transform-origin: 50% 50%;
  filter: blur(0);
  opacity: 1;
}
```
以下是包含一些示例值的可用选项的完整列表：  
```
var options = {
  iterations: Infinity,
  iterationStart: 0,
  delay: 0,
  endDelay: 0,
  direction: 'alternate',
  duration: 700,
  fill: 'forwards',
  easing: 'ease-out',
}
element.animate(keyframes, options);
```
## 缓动
缓动是任何动画中最重要的元素之一。WAAPI 为我们提供了两种不同的方式来设置缓动动画——在我们的关键帧数组中或在我们的选项对象中。  
在 CSS 中，如果您应用了，animation-timing-function: ease-in-out 缓入缓出动画。整个动画从开头到结尾以 ease-in-out 的速度来播放动画。  
```
div {
  animation-timing-function: ease-in-out;
}
```
缓动应用于关键帧之间，而不是整个动画，这可以对动画的感觉进行细粒度的控制。WAAPI 提供了这种能力。  
```
var keyframes = [
  { opacity: 0, easing: 'ease-in' }, 
  { opacity: 0.5, easing: 'ease-out' }, 
  { opacity: 1 }
]
```
CSS 和 WAAPI 中，不应该为最后一帧传递缓动值，因为这将不起作用。  
 CSS 动画和 WAAPI 之间的另一个区别：CSS 的默认值是 ease，而 WAAPI 的默认值是 linear。  
 WAAPI 提供了与 CSS 动画相同的性能改进，可以实现流畅的动画，意味着我们 will-change 可以避免使用
 ## 动画对象
 .animate() 方法不仅为我们的元素设置动画，它还返回一个动画对象。这为我们提供了各种功能。例如暂停动画：
``` 
myAnimation.pause() 
```
也可以通过更改 animation-play-state 属性来使用 CSS 动画实现类似的效果
``` 
element.style.animationPlayState = "paused"
```
使用 WAAPI，可以简单地使用 myAnimation.play() 从头开始播放动画，如果它之前已经完成，或者如果我们暂停它，则从中间迭代播放它。
甚至可以完全轻松地更改动画的速度:  
``` 
myAnimation.playbackRate = 2; // speed it up
myAnimation.playbackRate = .4; // use a number less than one to slow it down
```
## 获取动画
``` 
element.getAnimations()
```
此方法将为返回 WAAPI 定义的任何动画以及任何 CSS transitions 或者 animations 动画
即使一个 DOM 元素只应用了一个动画，getAnimations() 也将始终返回一个数组:  
``` 
var h2 = document.querySelector("h2");
var myCSSAnimation = h2.getAnimations()[0];
```
## 事件 promise
WAAPI 中使用 onfinish，与 animationend 或 transitionend 相同：
``` 
myAnimation.onfinish = function() {
  element.remove();
}
```
WAAPI 供了使用事件和 promise 两种选择。 finished 我们的动画对象的属性将返回一个在动画结束时解析的 promise。以下是使用 promise 的上述示例：
``` 
myAnimation.finished.then(() =>
  element.remove())
```
只有在页面上的所有动画都完成后函数才会运行:  
``` 
Promise.all(document.getAnimations().map(animation => 
  animation.finished)).then(function() {           
    // do something cool 
  })
```
原文:  
[什么是Web Animations API？](https://juejin.cn/post/7065093728737689614?utm_source=gold_browser_extension)