# web前端性能分析的工具
## 前端性能监控
前端性能监控包括两种方式：合成监控（Synthetic Monitoring，SYN）、真实用户监控（Real User Monitoring，RUM）。
### 合成监控
合成监控就是在一个模拟场景里，去提交一个需要做性能审计的页面，通过一系列的工具、规则去运行你的页面，提取一些性能指标，得出一个审计报告。
一般可能出现在开发和测试的过程中，例如结合流水线跑性能报告、定位性能问题时本地跑的一些简单任务分析等。  
该方式的优点:  
- 可采集更丰富的数据指标，例如结合 Chrome Debugging Protocol (opens new window)获取到的数据
- 较成熟的解决方案和工具，实现成本低
- 不影响真实用户的性能体验

### 真实用户监控
真实用户监控，就是用户在我们的页面上访问，访问之后就会产生各种各样的性能指标。我们在用户访问结束的时候，把这些性能指标上传到我们的日志服务器上，进行数据的提取清洗加工，最后在我们的监控平台上进行展示的一个过程。常见的一些性能监控包括加载耗时、DOM 渲染耗时、接口耗时统计等， 
在力所能及的地方进行打点、计算、采集、上报，该过程常常需要借助 Performance Timeline API。将需要的数据发送到服务端，然后再对这些数据进行处理，最终通过可视化等方式进行监控。真实用户监控往往需要结合业务本身的前后端架构设计来建设，其优点是：
- 完全还原真实场景，减去模拟成本
- 数据样本足够抹平个体的差异
- 采集数据可用于更多场景的分析和优化

对比合成监控，真实用户监控在有些场景下无法拿到更多的性能分析数据（例如具体哪里 CPU 占用、内存占用高），因此更多情况下作为优化效果来参考。这些情况下，具体的分析和定位可能还是得依赖合成监控。  
但真实用户监控也有自身的优势，例如 TCP、DNS 连接耗时过高，在各种环境下的一些运行耗时问题，合成监控是很难发现的。  

## 性能分析自动化
### 使用 Lighthouse
它提供了脚本的方式使用。因此可以通过自动化任务跑脚本的方式，使用 Lighthouse 跑分析报告，通过对比以往的数据来进行功能变更、性能优化等场景的性能回归。  
 Lighthouse 的优势在于开发成本低，只需要按照官方提供的配置 (opens new window)来调整、获取自己需要的一些数据，就可以快速接入较全面的 Lighthouse 拥有的性能分析能力。  
 由于 Lighthouse 同样基于 CDP(Chrome DevTools Protocol)，因此除了实现成本降低了，CDP 缺失的一些能力它也一样会缺失。
### Chrome DevTools Protocol
Chrome DevTools Protocol (opens new window)允许第三方对基于 Chrome 的 Web 应用程序进行检测、检查、调试、分析等。有了这个协议，我们就可以自己开发工具获取 Chrome 的数据了  
**Chrome DevTools 协议**  
Chrome DevTools 协议基于 WebSocket，利用 WebSocket 建立连接 DevTools 和浏览器内核的快速数据通道。  
我们使用的 Chrome DevTools 其实也是一个 Web 应用。我们使用 DevTools 的时候，浏览器内核 Chromium 本身会作为一个服务端，我们看到的浏览器调试工具界面，通过 Websocket 和 Chromium 进行通信。建立过程如下：  
1. DevTools 将作为 Web 应用程序，Chromium 作为服务端提供连接。
2. 通过 HTTP 提取 HTML、JavaScript 和 CSS 文件。
3. 资源加载后，DevTools 会建立与浏览器的 Websocket 连接，并开始交换 JSON 消息。

通过 DevTools 从 Windows、Mac 或 Linux 计算机远程调试 Android 设备上的实时内容时，使用的也是该协议。当 Chromium 以一个--remote-debugging-port=0标志启动时，它将启动 Chrome DevTools 协议服务器。  
**Chrome DevTools 协议域划分**  
Chrome DevTools协议具有与浏览器的许多不同部分（例如页面、Service Worker 和扩展程序）进行交互的 API。该协议把不同的操作划分为了不同的域（domain），每个域负责不同的功能模块。比如DOM、Debugger、Network、Console和Performance等，可以理解为 DevTools 中的不同功能模块。  
使用该协议可以：  
- 获取 JS 的 Runtime 数据，常用的如window.performance和window.chrome.loadTimes()等
- 获取Network及Performance数据，进行自动性能分析
- 使用 Puppeteer (opens new window)的 CDPSession (opens new window)，与浏览器的协议通信会变得更加简单

**与性能相关的域**  
1. Performance。 从Performance域中Performance.getMetrics()可以拿到获取运行时性能指标包括：
    - Timestamp: 采取度量样本的时间戳
    - Documents: 页面中的文档数
    - Frames: 页面中的帧数
    - JSEventListeners: 页面中的事件数
    - Nodes: 页面中的 DOM 节点数
    - LayoutCount: 全部或部分页面布局的总数
    - RecalcStyleCount: 页面样式重新计算的总数
    - LayoutDuration: 所有页面布局的合并持续时间
    - RecalcStyleDuration: 所有页面样式重新计算的总持续时间
    - ScriptDuration: JavaScript 执行的持续时间
    - TaskDuration: 浏览器执行的所有任务的合并持续时间
    - JSHeapUsedSize: 使用的 JavaScript 栈大小
    - JSHeapTotalSize: JavaScript 栈总大小

2. Tracing。 Tracing域可获取页面加载的 DevTools 性能跟踪。可以使用Tracing.start和Tracing.stop创建可在 Chrome DevTools 或时间轴查看器中打开的跟踪文件。  
3. Runtime。 Runtime域通过远程评估和镜像对象暴露 JavaScript 的运行时。可以通过Runtime.getHeapUsage获取 JavaScript 栈的使用情况，通过Runtime.evaluate计算全局对象的表达式，通过Runtime.queryObjects迭代 JavaScript 栈并查找具有给定原型的所有对象（可用于计算原型链中某处具有相同原型的所有对象，衡量 JavaScript 内存泄漏）。

### 自动化性能分析
通过使用 Chrome DevTools 协议，我们可以获取 DevTools 提供的很多数据，包括网络数据、性能数据、运行时数据。
对于如何使用该协议，已经有很多大神针对这个协议封装出不同语言的库，包括 Node.js、Python、Java等，可以根据需要在 awesome-chrome-devtools (opens new window)这个项目中找到。
原文:  
[补齐Web前端性能分析的工具盲点](https://godbasin.github.io/front-end-playground/front-end-basic/deep-learning/front-end-performance-analyze.html#performance-%E9%9D%A2%E6%9D%BF)
