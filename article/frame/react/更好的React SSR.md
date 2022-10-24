# 更好的 React SSR
**Server Side Rendering**  
SSR（Server Side Rendering）: 在服务端直接运行相关框架和代码，渲染出 HTML，发送给客户端后再进行水合，以响应用户的交互。这样做的好处是大大降低了传统 SPA 应用（也就是 CSR）的 “白屏” 时间，拥有更好的性能和用户体验，同时天然支持 SEO，弥补了 SPA 应用最大的短板。  
**性能指标**  
在 2020 年的时候，Google 正式推出了新的 Web Vitals。它的三大核心指标分别是：  
- LCP：最大内容绘制，测量加载性能。
- FID：首次输入延迟，测量交互性。
- CLS：累积布局偏移，测量视觉稳定性

和以往的 FCP、FMP、TTI、FCI 等性能指标相比，新的 Core Web Vitals 显然更加 “人性化”。它最大限度的从用户的视角出发，来测量性能。  
基于新标准来衡量页面的性能，一些传统方案的短板就暴露无遗了。举个例子，比如页面的加载性能，以往使用 DOMContentLoaded（DOM 内容加载完毕）、First Meaningful Paint（首次有效绘制）等，其实都不能有效的测量。常规的 SPA 应用，会第一时间响应一个几乎空白的 HTML，DOM 加载完毕可能才刚开始请求数据，首次绘制的可能也仅仅是一个 Loading:  
``` 
<div id="root">
   <img src="https://example.com/loading.gif" />
 </div>
```

> web vitals 其实也还存在一些不太完善的地方。拿 LCP 来说，可能某些情况下，最大的元素并不是页面的核心内容。比如背景图片，在页面渲染完毕后，背景图可能只露出很少的部分，但是它面积很大，会被判定为 LCP 元素

**SSR 的优势**  
SSR 相比传统的 SPA 方案，性能上最大的优势便是 LCP 了。  
对于 SSR，整个页面是直接响应给客户端的，客户端可以直接渲染 HTML，第一时间请求并渲染图片。而 SPA 呢？要等到 HTML 加载完毕、CSS & JS 解析执行、请求接口并收到响应、视频相关的 HTML 结构渲染完成、图片加载 & 渲染完毕，才算做加载成功。  
所以从 LCP 元素的渲染路径上看，SSR 明显会比传统 SPA 更精简、更高效。  
无论是 SSR 还是 CSR，都是常规的 fetch 请求，主要的区别在于网络环境，理论上 SSR 的请求是服务器内部的（之间的），必须要优于 CSR 的公网请求，否则 SSR 的优化效果就会大大降低。另外，SSR 还需要注意控制请求的超时时间，做好降级，否则用户看到的会是一个比 CSR “更白的白屏”。  

> CLS 这个指标是测量视觉稳定性的，更多的是和 HTML、CSS 布局相关。举个例子：<img /> 要有明确的宽高，否则等图片加载完成后撑开容器，必然产生布局位移。除了一些极特殊的场景外，大部分情况下，布局的时候仔细点，开发完成后多体验几次，基本能避免 CLS 问题

LCP 其实可以理解为首屏渲染速度的一个可度量指标，这是 SSR 最明显的优势之一。其他的还有 SEO、心智上的前后端统一等  

## SSR 的问题
**FID 性能**  
SSR 依然存在一些性能问题，相比 SPA，SSR 在 FID 的表现上，可能会更糟糕。  
从用户的角度想，页面很快呈现在了他的面前，那接下来肯定是操作了，上下左右的划一划，点击几个 Button 或者 Tab，都是再正常不过的交互。而此时 JS 代码可能还在解析运行中，React 正在水化（re-hydrated）页面中，来不及或者根本无法响应用户。  
SPA 虽然在 LCP 上具有天然的劣势，有更长的白屏时间，但是在页面初始化时的 JS 解析执行、请求接口、渲染等操作都是比较 “分散” 的，而且在不同的异步任务中，反而一定程度上避免了 “Long Task” 的产生。SSR 则正好相反，在页面初始阶段需要密集的执行大量代码。  
更坏的情况是，hydrate 异常降级为普通 render。这个还是比较容易发生的，有 class 顺序不一致导致的，还有 HTML 被意外压缩（没用的空格和换行）导致的。这种异常降级在 production 环境通常是静默的，表面上看没有影响用户的使用，但是对性能的损害却极大。  
FID 的问题在简单的页面上不会太明显，但是在企业应用开发中，总是会有埋点、反垃圾、反作弊、异常监控、设备指纹、加解密等操作，会接入一些性能一言难尽的二方 & 三方 SDK。而且它们中有相当一部分需要在页面初始化的时候执行。这类型的代码对用户也是无感知的，但对性能的影响同样非常大。  
优化 FID 是非常重要的。不少开发者可能日常对交互性能关注较少（除非卡成 PPT），但这个观念需要转变了。Lighthouse 8 将 TBT（实验室版的 FID）的性能分占比，提升到了 30%，CLS 也有大幅度提升。以前优化的大都是速度，对布局稳定性和交互流畅度的重视不够。
**强依赖接口性能**  
并不是把代码在服务端 renderToString 就是所谓的服务端渲染,真正的服务端渲染，一定是包含必要数据的，渲染出的结果也基本能等同于用户看到的最终呈现（至少是首屏）  
所以数据是非常重要的，SSR 的效果会依赖服务端接口的性能。如果服务端接口迟迟不能响应，并且前端应用没有相应的超时和降级机制，那么用户看到的只比白屏更白。  
数据的实时性要求很高，因为其他种种原因而无法缓存，平均响应时间大于 200ms 的接口。这种情况最好直接使用 SSG，再加上 “骨架屏” 之类优雅点的 Loading。  
**无意义的 hydrate 和冗余的代码**  
在日常开发中，总是会遇到一些静态，或者大部分区域是静态的页面。这些页面或区域，其实只需要服务端生成好 HTML，客户端渲染出来就可以了。hydrate 是完全没必要的，甚至 JS 都没必要加载。徒增负担。  
除了客户端运行时的不必要负担，还增大了整个页面的体积，因为 JSX 里的内容会重复出现在 HTML 和 JS 里。  
实践中 JSX 的冗余其实还好，更可怕的是数据，最可怕是 “不做处理，有用没用都返回” 的数据。这些数据通常是内联在 HTML 中，保证客户端 hydrate 的时候可以第一时间拿到。SSR 数据占 HTML 体积 50% 以上也是常事。  
**复杂度**  
应用方面，需要确保稳定性，不管是守护进程还是容器伸缩，都要做好必要的保障。同时要能主动或者自动的降级，前文提到过，服务端渲染时要控制接口超时时间，没有超时降级机制，性能的损害反而是小事。更糟糕的情况是接口长时间无响应，导致页面也长时间无法响应，请求堆积在 node 层，直到崩溃，直到 502。  
编码方面，需要一开始就做好规划，考虑清楚代码运行在服务端和客户端的各种情况，明确哪部分必须要 SSR，哪部分可以在客户端再执行，怎么设计才能达到最好的性能。SSR 的部分还要兼容降级的场景。此外，还需要设法兼容无法在 server 端执行的类库。  
这两方面的复杂度都不难克服，框架层面做好稳定性保障，编码方面只要完全明白了原理，也不会增加太多的开发成本和心智负担。  

**实例对比**  
一个活动项目，它本身是 SSR 的，单独拉了一个分支改造为 CSR，部署了两套一样的环境  
其他一些信息：  
- 测试页面是活动项目，首屏没有动态数据，SSR 不涉及数据请求。
- 页面的埋点、异常监控、性能监控、指纹、加解密（JS 文件会大挺多）、eruda 等都保留着。
- 构建、部署流程调整起来比较麻烦，这次用的都是开发的配置，production 下的优化全没有。
- 主要技术栈就是 React 全家桶 + CSS Modules。
- 使用 Edge 的 devtools performance 测试，和 Chrome 差别不大。Edge 的本地化做的不错，而且我没有装任何扩展插件，不会产生干扰。

几个关键信息：  
- LCP 时长：SSR 120ms 左右，CSR 570ms 左右。即使不包含首屏数据的优化，LCP 上的优势还是很明显的  
- LCP 元素的渲染路径：SSR 在渲染前没有大段的 JS Task，而 CSR 需要等 JS 执行完。与我们前面的分析一致  
- Long Task：SSR 最长 199.71ms，而且比较集中；CSR 最长 188.96ms，分散成了 3 段。更糟糕的是 SSR 的长任务就在 LCP 元素渲染完成之后，非常容易阻塞用户的操作

>  web vitals 在实验室环境（Lighthouse）下测量并不是很准确，波动比较大，LCP 这类型的指标和实际网络情况有关。对于更加人性化的性能指标，我们应该更加关注用户侧的真实性能数据采集，综合分析一个时间段内的数据。

**可能的解决方案**  
接口性能差的问题，如果很普遍，那就没必要用 SSR 了，SSG 是更好的选择；个别情况的话，可以考虑 SSR + SSG 的混合模式，这个并不复杂，SSG 可以看成是 “构建时” 把每个页面跑一遍 SSR，并将 HTML 存下来（预渲染）。目前，如果业务、交互、设计等方面允许的话，在开发时会更倾向于 SSG，彻底干掉服务端数据预取这不稳定的一环，也更便于 CDN 缓存，命中率更高。  
**Suspense SSR Architecture**  
功能层面，引入了两个重要的能力：  
- Streaming HTML：服务端流式 HTML。服务端尽早响应 HTML，后续，其他的 HTML 片段附带 script 一起流式传输给客户端，script 执行将 HTML 插入到正确的位置。
- Selective Hydration：在代码完全下载之前，尽早开始 hydrate。优先 hydrate 用户交互的部分

API 层面主要有这么几个变化：  
- hydrateRoot：新的 react-dom/client API，替换旧的 hydrate
- <Suspense>：不是新 API，但是在 React 18 中，通过 <Suspense> 将页面拆分为小的、独立的单元，这些单元可以自主的 hydrate
- React.lazy：也不是新 API，但是在 v18 里，支持了 SSR
- renderToPipeableStream：实现 Streaming HTML 的核心

Suspense SSR Architecture 的具体功能点，页面结构大体如下：  
``` 
const UserInfo = React.lazy(() => import('./UserInfo.js'))
 const Explore = React.lazy(() => import('./Explore.js'))

 // ...

 const App = (
   <Layout>
     {/* 视频播放器 */}
     <VideoPlayer />

     {/* 作者信息 */}
     <Suspense fallback={<UserInfoSkeleton />}>
       <UserInfo />
     </Suspense>

     {/* 更多推荐视频 */}
     <Suspense fallback={<ExploreSkeleton />}>
       <Explore />
     </Suspense>
   </Layout>
 )
```
**Streaming HTML**  
SSR 强依赖于接口的性能。假如 <Explore /> 要花很长时间来 fetch data，此时就必须做出取舍，要么忍受由此带来的性能损耗，客户端收到 HTML 的时间变得更长；要么放弃 SSR，<Explore /> 使用纯客户端渲染。  
实践中的经验是，首先判断是否是页面的核心内容，或者 LCP，如果是，优先使用 SSR 渲染；其次，判断内容是否会出现在首屏，例如 <UserInfo>，非核心但是也很重要，属于首屏很关键的一部分，也优先考虑 SSR；第三，相关接口的性能，如果还算勉强合格，偶尔有波动，那么就控制好 fetch timeout，密切观察性能指标，通过数据分析，决定后续优化方向；最后，如果接口就是很慢，几乎难以优化，那么就直接放弃，打磨好交互、视觉，保证用户体验，SSG、Skeleton、Spinner 等各种手段能用则用，具体情况具体分析。  
React 18 发布后，情况变得不一样，我们可以 “既要、又要”，页面核心的部分不变，直接 SSR 渲染。而其他部分，嵌套在 <Suspense> 中，立即响应 fallback，组件渲染完成之后，流式下发到客户端，插入对应的位置。  
上面的 “短视频作品” 页面，在客户端呈现的过程大致如下图，直接渲染出 <VideoPlayer /> 和两个定制 Skeleton，而后视接口响应的速度，分别补齐 <UserInfo /> 和 <Explore />：  
Streaming HTML 很好的解决了服务端渲染页面时，对接口响应速度的依赖。加载比较慢的组件，不会影响到较快的部分  
**Selective Hydration**  
SSR 的 hydrate 需要一次性处理完整个页面，这个过程可能耗时比较长，极容易产生 Long Task，阻塞客户端对用户操作的响应。之前处理此类问题的方案是把一部分组件通过 Code Splitting 拆分出去，在客户端异步的 CSR，说白了就是放弃 SSR。  
在 React 18 中，React.lazy 支持了服务端渲染，可组合使用 React.lazy 和 <Suspense>。就像上面示例的 <UserInfo /> 和 <Explore />，它们的 JS 加载和 hydrate 都是独立的，互不影响。Selective Hydration 机制打破了之前一次性水合的限制，我们可以根据需求灵活的控制，同时享受 SSR、Code Splitting 和独立的 hydrate，三倍的快乐。  
Selective Hydration 真正的大杀器，其实是基于用户交互的优先级调度。  
在 React 18 里，<Suspense> 内执行 hydrate 时，会有极小的间隙来响应用户事件  
<UserInfo /> 和 <Explore /> 都还未 hydrate，无法响应用户的交互。用户点击 <Explore /> 组件，React 会认为它是更重要更紧急的部分，在 click 事件的捕获阶段，同步完成 hydrate，然后响应用户的点击。点击这种离散事件是在不同阶段处理的，其他连续事件，比如 mouseover，处理逻辑是不一样的，具体细节可以看这里。  
<Suspense> 还有一个 “隐性” 的好处，相信应该有人在使用 React 18 SSR 的时候，看到过这样的异常：  
> There was an error while hydrating. Because the error happened outside of a Suspense boundary, the entire root will switch to client rendering.

它其实还起到了 hydrate 的 error boundaries 的作用，万一发生了异常，可以避免整页重新渲染。  
**Progressive Hydration**  
Progressive Hydration（渐进式水合）其实提出的时间要比 React 18 更早，社区里不少人很早就意识到了前文提到的 SSR 的问题，于是提出了 Progressive Hydration 的想法。它的名字就很好的阐述了它的本质：“不要一次性 hydrate，要按需 & 依次 & 渐进式的执行”。  
Suspense SSR Architecture 其实就是 Progressive Hydration 的一种很好的实现，而且是框架级的支持。较早的时候，社区已经有一些在 React 之上的变通方案了。比如在 Google I/O '19 上，Chrome 团队就演示了一个 Demo。它的核心代码如下：  
``` 
import React from 'react'
 import ReactDOM from 'react-dom'

 function interopDefault(mod) {
   return (mod && mod.default) || mod
 }

 export function ServerHydrator({ load, ...props }) {
   const Child = interopDefault(load())
   return (
     <section>
       <Child {...props} />
     </section>
   )
 }

 export class Hydrator extends React.Component {
   shouldComponentUpdate() {
     return false
   }

   componentDidMount() {
     new IntersectionObserver(async ([entry], obs) => {
       if (!entry.isIntersecting) return
       obs.unobserve(this.root)

       const { load, ...props } = this.props
       const Child = interopDefault(await load())
       ReactDOM.hydrate(<Child {...props} />, this.root)
     }).observe(this.root)
   }

   render() {
     return (
       <section
         ref={(c) => (this.root = c)}
         dangerouslySetInnerHTML={{ __html: '' }}
         suppressHydrationWarning
       />
     )
   }
 }
```
这段代码过于 hack，在服务端使用 ServerHydrator 正常渲染，在客户端变成了 Hydrator，同时利用 dangerouslySetInnerHTML 维持 HTML 结构不变，最后使用 IntersectionObserver，在合适的时机执行 ReactDOM.hydrate。  
Progressive Hydration 本质上只是一种思想，它的具体实现方案可能有很多，但核心就是 “可控的 hydrate”。要么完全可控，像上面使用 IntersectionObserver 一样；要么完全不用管，框架做到完美，就是 Suspense SSR Architecture 的方向。  
**Islands Architecture**  
Fresh 将构成页面的各种组件，分为 route 和 island 两类，约定存放在 routes/ 和 islands/ 两个目录。Fresh 处理这两类组件的方式完全不同：  
- Route Component：仅在服务端执行，直接响应 SSR 渲染出的 HTML 给客户端，在客户端不会加载和执行任何 JS 代码，更不用说 hydrate 了
- Island Component：不仅在服务端执行，它的 JS 也会在客户端加载，并且 hydrate，所以 Island 可以响应用户的交互

这两类组件，便是 Islands Architecture 的核心。在这个架构中，Routes 就像是海，它就放在那里，看得见，但无法在上面活动（交互）；而 Islands 就像是岛屿，看得见也摸得着，可以在上面产生交互。Islands 之间是相互独立的，一个崩溃不会影响另一个，同时它们的 hydrate 过程也是独立的，hydrate 完成后即可立即响应用户的交互（只要此时没被别的任务阻塞）。  
Route 和 Island 这两种组件，本质上都是 Preact 组件。Island 比 Route 更 “正常” 一些，和常规的参与 SSR 的组件没太大区别。而 Route，更像是被 Fresh 当做模板引擎使用了，JSX + Props => HTML，写了 useEffect 也不会执行。任何需要在客户端执行 JS 的区块，都必须抽成一个 Island Component 独立出去。  
官方的一个很简单的示例:  
``` 
// routes/index.tsx

 import Counter from '../islands/Counter.tsx'

 export default function Home() {
   return (
     <div>
       <p>
         Welcome to Fresh. Try to update this message in the ./routes/index.tsx file, and
         refresh.
       </p>
       <Counter start={3} />
     </div>
   )
 }
 
 // islands/Counter.tsx

 import { useState } from 'preact/hooks'
 import { IS_BROWSER } from '$fresh/runtime.ts'

 interface CounterProps {
   start: number
 }

 export default function Counter(props: CounterProps) {
   const [count, setCount] = useState(props.start)
   return (
     <div>
       <p>{count}</p>
       {/* 这个 disabled 可以说很细节了 */}
       <button onClick={() => setCount(count - 1)} disabled={!IS_BROWSER}>
         -1
       </button>
       <button onClick={() => setCount(count + 1)} disabled={!IS_BROWSER}>
         +1
       </button>
     </div>
   )
 }
```
客户端加载的 JS 代码很少：
- main.js 和 chunk-TDJO6WAF.js 主要是 Fresh 的 runtime 代码和 preact
- island-counter.js 就是的 Island 组件

后续如果添加更多的可交互组件，也会类似 island-counter.js 一样，独立为一个个互不影响的 island  
Fresh 内部强依赖 Preact，通过 Preact 将所有组件渲染为 HTML，给 Islands 打好标记。同时 JS 的依赖收集根据约定的目录控制好范围。在客户端，使用少量运行时和 Preact，完成 hydrate  
``` 
<!-- 这是 SSR 生成的 HTML 片段 -->

 <div>
   <p>Welcome to `fresh`. Try updating this message in the ./routes/index.tsx file, and refresh.</p>
   <!--frsh-counter:0-->
   <div>
     <p>3</p>
     <button disabled>-1</button>
     <button disabled>+1</button>
   </div>
   </!--frsh-counter:0-->
 </div>
```
除了 Islands Architecture 之外，Fresh 的 Just-in-time rendering + Edge Runtime 设计，也是对 “强动态化页面” 很好的优化，虽然有一定的门槛。但这种做法，在当下看，可能有点极端了，有点过于 “next-gen” 了SSG 之类的功能，还有很有必要的。

原文:  
[更好的 React SSR](https://mp.weixin.qq.com/s/3AVzctr8LwaFK9R_-x-uaA)
