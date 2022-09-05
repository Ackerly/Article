# 3 个即将推出的 CSS 特性
## @container
@container 是一个容器查询方法，正如它的名字一样，它是用来支持根据当前元素所在容器的大小来进行动态修改添加样式的，这跟 @media 基于视口大小是不一样的。  
``` 
// html
<body>
  <aside class="sidebar">
    <div class="card">
      <h4>侧边栏</h4>
      <p>
        To the world you may be one person, but to one person you may be the world.
      </p>
    </div>
  </aside>
  <main class="content">
    <div class="card">
      <h4>主内容</h4>
      <p>
        To the world you may be one person, but to one person you may be the world.
      </p>
    </div>
  </main>
</body>
// CSS
body {
  display: flex;
  color: white;
}

h4 {
  color: black;
}

.sidebar {
  width: 30%;
}

.content {
  width: 70%;
  background: #f0f5f9;  /* 给个底色，与侧边栏区分 */
}

.card {
  background: lightpink;
  box-shadow: 3px 10px 20px rgba(0, 0, 0, 0.2);
  border-radius: 8px;
}
```
主内容这块儿空间很富余，便想改变一下标题和内容文字的布局，此时就可以用上 @container 了，直接让主内容在当前容器宽度大于 400px 时变成横向布局  
``` 
@container (min-width: 400px) {
  .content .card {
    display: flex;
  }
}
```
## object-view-box
object-view-box 属性就类似于 SVG 中的 viewBox 属性。它允许您使用一行 CSS 来平移、缩放、裁剪 图像
``` 
.crop {
  object-view-box: inset(10% 50% 35% 5%);
}
```
## animation-timeline
它允许我们基于容器滚动的进度来对动画进行处理，简而言之就是页面滚动了百分之多少，动画就执行百分之多少。而且动画也能根据页面倒着滚动而倒着播放  
``` 
.shoes {
  animation-name: Rotate;
  animation-duration: 1s; 
  animation-timeline: scrollTimeline;
}

@scroll-timeline scrollTimeline {
  source: selector('#container');
  orientation: "vertical";
}

@keyframes Rotate {
  from {
    transform: translate(-200px, -200px) rotate(0deg);
  }

  to {
    transform: translate(100vw, 100vh) rotate(720deg);
  }
}

```

原文: 
[令人期待的 3 个即将推出的 CSS 特性](https://mp.weixin.qq.com/s/phOZIr8edzWzJgTJfeyE9A)
