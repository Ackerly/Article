# CSS防止按钮重复点击
## CSS 实现思路分析
pointer-events对点击事件进行限制，也就是禁用点击事件
animation每次点击后需要自动禁用300ms  
伪类:active触发点击行为  
这种场景可以理解成是对 CSS 动画的控制，比如有一个动画控制按钮从禁用->可点击的变化，每次点击时让这个动画重新执行一遍，在执行的过程中，一直处于禁用状态，就达到了“节流”的效果了  

## CSS 动画的精准控制
假设有一个按钮，绑定了一个点击事件  
``` 
<button onclick="console.log('保存')">保存</button>
```
定义一个关于pointer-events的动画，就叫做 throttle  
``` 
@keyframes throttle {
  from {
    pointer-events: none;
  }
  to {
    pointer-events: all;
  }
}
```
将这个动画绑定在按钮上，将动画设置成了2s  
``` 
button{
  animation: throttle 2s step-end forwards;
}
```
这里动画的缓动函数设置成了阶梯曲线，step-end，它可以很方便的控制pointer-events的变化时间点  
pointer-events在0~2秒内的值都是none，一旦到达2秒，就立刻变成了all，由于是forwards，会一直保持all的状态  
最后，在点击时重新执行一遍动画，只需要在按下时设置动画为none就行了  

实现如下:  
``` 
button:active{
  animation: none;
}

@keyframes throttle {
  from {
    color: red;
    pointer-events: none;
  }
  to {
    color: green;
    pointer-events: all;
  }
}
```
现在如果文字是red，表示是禁用态，只有是green，才表示可以被点击  
如果需要改限制时间，直接改动画时间就行了  
``` 
button{
  animation: throttle 2s step-end forwards;
}
button:active{
  animation: none;
}
@keyframes throttle {
  from {
    pointer-events: none;
  }
  to {
    pointer-events: all;
  }
}
```
## CSS 实现的其他思路
通过:active去触发transition变化，然后通过监听transition回调去动态设置按钮的禁用状态，实现如下  
定义一个无关紧要的过渡属性，比如opacity  
``` 
button{
  opacity: .99;
  transition: opacity 2s;
}
button:not(:disabled):active{
  opacity: 1;
  transition: 0s;
}
```
然后监听transition的起始回调   
``` 
// 过渡开始
document.addEventListener('transitionstart', function(ev){
  ev.target.disabled = true
})
// 过渡结束
document.addEventListener('transitionend', function(ev){
  ev.target.disabled = false
})
```
这样做的最大好处是，这部分禁用的逻辑是完全和业务逻辑是解耦的，可以在任意时候，任意场合下无缝接入，也不受框架和环境影响  


原文:  
[还在用 JS 做节流吗？CSS 也可以防止按钮重复点击](https://mp.weixin.qq.com/s/9_53TqxExIfvsxSFfOnEsA)
