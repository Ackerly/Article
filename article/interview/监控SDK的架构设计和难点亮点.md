# 说说前端监控平台/监控SDK的架构设计和难点亮点
## 整体 架构设计
在应用层SDK上报的数据，在接入层经过 削峰限流 和 数据加工 后，将原始日志存储于 ES 中，再经过 数据清洗 、数据聚合 后，将 issue（聚合的数据） 持久化存储 于 MySQL ，最后提供 RESTful API 提供给监控平台调用；  
- 削峰限流是为了避免 激增的大数据量、恶意用户访问等高并发数据导致的服务崩溃；
- 数据加工是为了将 IP、运营商、归属地等各种二次加工数据，封装进上报数据里；
- 数据清洗是为了经由白名单、黑名单过滤等的业务需要，还有避免已关闭的应用数据继续入库；
- 数据聚合是为了将相同信息的数据进行抽象聚合成 issue，以便查询和追踪；

## SDK 架构设计
为支持多平台、可拓展、可插拔的特点，整体SDK的架构设计是 内核+插件 的插件式设计；  
每个 SDK 首先继承于平台无关的 Core 层代码。然后在自身SDK中，初始化内核实例和插件；

### SDK 如何设计成多平台支持 
在前端监控的领域里，可能不仅仅只是监控一个 web环境 下的数据，包括 Nodejs、微信小程序、Electron 等各种其余的环境都是有监控的业务需求在的；  
那么就要思考一个点，一个 SDK 项目，既然功能全，又要支持多平台，那么怎么设计这个 SDK 可以让它既支持多平台，但是在启用某个平台的时候不会引入无用的代码呢  
最简单的办法：将每个平台单独放一个仓库，单独维护 ；但是这种办法的问题也很严重：人力资源浪费严重；会导致一些重复的代码很多；维护非常困难；  
而较好一点的解决方案：以通过插件化对代码进行组织  
- 用 Core 来管理 SDK 内与平台无关的一些代码，比如一些公共方法(生成mid方法、格式化)；
- 然后每个平台单独一个 SDK；去继承 core 的类；SDK 内自己管理SDK特有的核心方法逻辑，比如上报、参数初始化；
- 最后就是 Plugins 插件，每个SDK都是由 内核+插件 组成的，将所有的插件功能，比如性能监控、错误监控都抽离成插件；

**进行 SDK 的设计有很多好处：**  
- 每个平台分开打包，每个包的体积会大大缩小；
- 代码的逻辑更加清晰自恰

最后打包上线时，通过修改 build 的脚本，对 packages 文件夹下的每个平台都单独打一个包，并且分开上传到 npm 平台  
### SDK 如何方便的进行业务拓展和定制  
业务功能总是会不断迭代的，SDK 也一样，所以在设计SDK的时候就要考虑它的一个拓展性  
SDK 内部的一个架构设计 ———— 内核+插件 的设计：  
- 内核里是SDK内的公共逻辑或者基础逻辑；比如数据格式化和数据上报是底下插件都要用到的公共逻辑；而配置初始化是SDK运行的一个基础逻辑；
- 插件里是SDK的上层拓展业务，比如说监听js错误、监听promise错误，每一个小功能都是一个插件；
- 内核和插件一起组成了 SDK实例 Instance，最后暴露给客户端使用；

需要拓展业务，只需要在内核的基础上，不断的往上叠加 Monitor 插件的数量就可以了；  
至于说定制化，插件里的功能，都是使用与否不影响整个SDK运行的，所以可以自由的让用户对插件里的功能进行定制化，决定哪个监控功能启用、哪个监控功能不启用等等....
举个代码例子：  
``` 
// 服务于 Web 的SDK，继承了 Core 上的与平台无关方法;
class WebSdk extends Core {
  // 性能监控实例，实例里每个插件实现一个性能监控功能；
  public performanceInstance: WebVitals;

  // 行为监控实例，实例里每个插件实现一个行为监控功能；
  public userInstance: UserVitals;

  // 错误监控实例，实例里每个插件实现一个错误监控功能；
  public errorInstance: ErrorVitals;

  // 上报实例，这里面封装上报方法
  public transportInstance: TransportInstance;

  // 数据格式化实例
  public builderInstance: BuilderInstance;

  // 维度实例，用以初始化 uid、sid等信息
  public dimensionInstance: DimensionInstance;

  // 参数初始化实例
  public configInstance: ConfigInstance;

  private options: initOptions;

  constructor(options: initOptions) {
    super();
    this.configInstance = new ConfigInstance(this, options);
    // 各种初始化......
  }
}

export default WebSdk;
```
在初始化每个插件的时候，都将 this 传入进去，那么每个插件里面都可以访问内核里的方法；
### SDK 在拓展新业务的时候，如何保证原有业务的正确性
内核+插件 设计下，开发新业务对原功能的影响基本上可以忽略不计，但是难免有意外，所以在 SDK 项目的层面上，需要有 单元测试 的来保证业务的稳定性；  
可以引入单元测试，并对 每一个插件，每一个内核方法，都单独编写测试用例，在覆盖率达标的情况下，只要每次代码上传都测试通过，就可以保证原有业务的一个稳定性；  
### SDK 如何实现异常隔离以及上报
引入监控系统的原因之一就是为了避免页面产生错误，而如果因为监控SDK报错，导致整个应用主业务流程被中断，这是不能够接受的；  
实际上，我们无法保证我们的 SDK 不出现错误，那么假如万一SDK本身报错了，我们就需要它不会去影响主业务流程的运行；最简单粗暴的方法就是把整个 SDK 都用 try catch 包裹起来，那么这样子即使出现了错误，也会被拦截在我们的 catch 里面；  
这样简单粗暴的包裹，会带来哪些问题：  
- 只能获取到一个报错的信息，但是我们无法得知报错的位置、插件；
- 没有将其上报，我们无法感知到 SDK 产生了错误
- 没法获取 SDK 出错的一个环境数据

我们需要一个相对优雅的一个异常隔离+上报机制，回想我们上文的架构：内核+插件的形式；我们对每一个插件模块，都单独的用trycatch包裹起来，然后当抛出错误的时候，进行数据的封装、上报；  
样子，就完成了一个异常隔离机制：  
- 它实现了：当SDK产生异常时不会影响主业务的流程；
- 当SDK产生异常时进行数据的封装、上报；
- 出现异常后，中止 SDK 的运行，并移除所有的监听；

### SDK 如何实现服务端时间的校对 
通过 JS 调用 new Date() 获取的时间，是我们的机器时间；也就是说：这个时间是一个随时都有可能不准确的时间；  
既然时间是不准确的，假如有一个对时间精准度要求比较敏感的功能：比如说 API全链路监控；最后整体绘制出来的全链路图直接客户端的访问时间点变成了未来的时间点，直接时间穿梭那可不行；  
应头 上有一个字段 Date；它的值是服务端发送资源时的服务器时间，我们可以在初始化SDK的时候，发送一个简单的请求给上报服务器，获取返回的 Date 值后计算 Diff差值 存在本地；  
这样子就可以提供一个 公共API，来提供一个时间校对的服务，让本地的时间 比较趋近于 服务端的真实时间；（只是比较趋近的原因是：还会有一个单程传输耗时的误差）  
``` 
let diff = 0;
export const diffTime = (date: string) => {
  const serverDate = new Date(date);
  const inDiff = Date.now() - serverDate.getTime();
  if (diff === 0 || diff > inDiff) {
    diff = inDiff;
  }
};

export const getTime = () => {
  return new Date(Date.now() - diff);
};
```
> 这里还可以做的更精确一点，我们可以让后端服务在返回的时候，带上 API 请求在后端服务执行完毕所消耗的时间 server-timing，放在响应头里；我们取到数据后，将 ttfb 耗时 减去返回的 server-timing 再除以 2；就是单程传输的耗时；那这样我们上文的计算中差的 单程传输耗时的误差 就可以补上了；

**SDK 如何实现会话级别的错误上报去重**  
- 在用户的一次会话中，如果产生了同一个错误，那么将这同一个错误上报多次是没有意义的
- 在用户的不同会话中，如果产生了同一个错误，那么将不同会话中产生的错误进行上报是有意义的

为什么有上面的结论呢？理由很简单:  
- 在用户的同一次会话中，如果点击一个按钮出现了错误，那么再次点击同一个按钮，必定会出现同一个错误，而这出现的多次错误，影响的是同一个用户、同一次访问；所以将其全部上报是没有意义的；
- 而在同一个用户的不同会话中，如果出现了同一个错误，那么这不同会话里的错误进行上报就显得有意义了

``` 
// 对每一个错误详情，生成一串编码
export const getErrorUid = (input: string) => {
  return window.btoa(unescape(encodeURIComponent(input)));
};
```
所以说我们传入的参数，是 错误信息、错误行号、错误列号、错误文件等可能的关键信息的一个集合，这样保证了产生在同一个地方的错误，生成的 错误mid 都是相等的；这样子，我们才能在错误上报的入口函数里，做上报去重；  
``` 
// 封装错误的上报入口，上报前，判断错误是否已经发生过
errorSendHandler = (data: ExceptionMetrics) => {
  // 统一加上 用户行为追踪 和 页面基本信息
  const submitParams = {
    ...data,
    breadcrumbs: this.engineInstance.userInstance.breadcrumbs.get(),
    pageInformation: this.engineInstance.userInstance.metrics.get('page-information'),
  } as ExceptionMetrics;
  // 判断同一个错误在本次页面访问中是否已经发生过;
  const hasSubmitStatus = this.submitErrorUids.includes(submitParams.errorUid);
  // 检查一下错误在本次页面访问中，是否已经产生过
  if (hasSubmitStatus) return;
  this.submitErrorUids.push(submitParams.errorUid);
  // 记录后清除 breadcrumbs
  this.engineInstance.userInstance.breadcrumbs.clear();
  // 一般来说，有报错就立刻上报;
  this.engineInstance.transportInstance.kernelTransportHandler(
    this.engineInstance.transportInstance.formatTransportData(transportCategory.ERROR, submitParams),
  );
};
```
### SDK 采用什么样的上报策略
SDK的数据上报可不是随随便便就上报上去了，里面有涉及到数据上报的方式取舍以及上报时机的选择等等，还有一些可以让数据上报更加优雅的优化点；  
日志上报并不是应用的主要功能逻辑，日志上报行为不应该影响业务逻辑，不应该占用业务计算资源；  
目前通用的几个上报方式：  
- 信标（Beacon API）
- Ajax（XMLHttpRequest 和 fetch）
- Image（GIF、PNG）

首先 Beacon API是一个较新的 API:  
- 它可以将数据以 POST 方法将少量数据发送到服务端
- 它保证页面卸载之前启动信标请求
- 并允许运行完成且不会阻塞请求或阻塞处理用户交互事件的任务

Image 上报方式：可以以向服务端请求图片资源的形式，像服务端传输少量数据，这种方式不会造成跨域；

**上报方式**  
采用 sendBeacon + xmlHttpRequest 降级上报的方式，当浏览器不支持 sendBeacon 或者 传输的数据量超过了 sendBeacon 的限制，我们就降级采用 xmlHttpRequest 进行上报数据；  
优先选用 Beacon API 的理由是它可以保证页面卸载之前启动信标请求，是一种数据可靠，传输异步并且不会影响下一页面的加载 的传输方式。  
而降级使用 XMLHttpRequest 的原因是， Beacon API 现在并不是所有的浏览器都完全支持，我们需要一个保险方案兜底，并且 sendbeacon 不能传输大数据量的信息，这个时候还是得回到 Ajax 来；  
为什么不用 Image 呀？那跨域怎么办呀？原因也很简单：  
- Image 是以GET方式请求图片资源的方式，将上报数据附在 URL 上携带到服务端，而URL地址的长度是有一定限制的。规范对 URL 长度并没有要求，但是浏览器、服务器、代理服务器都对 URL 长度有要求。有的浏览器要求URL中path部分不超过 2048，这就导致有些请求会发送不完全
- 至于跨域问题，作为接受数据上报的服务端，允许跨域是理所应当的；

简单封装一下：  
``` 
export enum transportCategory {
  // PV访问数据
  PV = 'pv',
  // 性能数据
  PERF = 'perf',
  // api 请求数据
  API = 'api',
  // 报错数据
  ERROR = 'error',
  // 自定义行为
  CUS = 'custom',
}

export interface DimensionStructure {
  // 用户id，存储于cookie
  uid: string;
  // 会话id，存储于cookiestorage
  sid: string;
  // 应用id，使用方传入
  pid: string;
  // 应用版本号
  release: string;
  // 应用环境
  environment: string;
}
export interface TransportStructure {
  // 上报类别
  category: transportCategory;
  // 上报的维度信息
  dimension: DimensionStructure;
  // 上报对象(正文)
  context?: Object;
  // 上报对象数组
  contexts?: Array<Object>;
  // 捕获的sdk版本信息，版本号等...
  sdk: Object;
}

export default class TransportInstance {
  private engineInstance: EngineInstance;

  public kernelTransportHandler: Function;

  private options: TransportParams;

  constructor(engineInstance: EngineInstance, options: TransportParams) {
    this.engineInstance = engineInstance;
    this.options = options;
    this.kernelTransportHandler = this.initTransportHandler();
  }

  // 格式化数据,传入部分为 category 和 context \ contexts
  formatTransportData = (category: transportCategory, data: Object | Array<Object>): TransportStructure => {
    const transportStructure = {
      category,
      dimension: this.engineInstance.dimensionInstance.getDimension(),
      sdk: getSdkVersion(),
    } as TransportStructure;
    if (data instanceof Array) {
      transportStructure.contexts = data;
    } else {
          transportStructure.context = data;
    }
    return transportStructure;
  };

  // 初始化上报方法
  initTransportHandler = () => {
    return typeof navigator.sendBeacon === 'function' ? this.beaconTransport() : this.xmlTransport();
  };

  // beacon 形式上报
  beaconTransport = (): Function => {
    const handler = (data: TransportStructure) => {
      const status = window.navigator.sendBeacon(this.options.transportUrl, JSON.stringify(data));
      // 如果数据量过大，则本次大数据量用 XMLHttpRequest 上报
      if (!status) this.xmlTransport().apply(this, data);
    };
    return handler;
  };

  // XMLHttpRequest 形式上报
  xmlTransport = (): Function => {
    const handler = (data: TransportStructure) => {
      const xhr = new (window as any).oXMLHttpRequest();
      xhr.open('POST', this.options.transportUrl, true);
      xhr.send(JSON.stringify(data));
    };
    return handler;
  };
}
```
**上报时机**  
- PV、错误、用户自定义行为 都是触发后立即就进行上报；
- 性能数据 需要等待页面加载完成、数据采集完毕后进行上报；
- API请求数据 会进行本地暂存，在数据量达到10条(自拟)时触发一次上报，并且在页面可见性变化、以及页面关闭之前进行上报；
- 如果你还要上报 点击行为 等其余的数据，跟 API请求数据 一样的上报时机；

**上报优化**  
- 启用 HTTP2，在 HTTP1 中，每次日志上报请求头都携带了大量的重复数据导致性能浪费。HTTP2头部压缩 采用Huffman Code压缩请求头，能有效减少请求头的大小；
- 服务端可以返回 204 状态码，省去响应体；
- 使用 `requestIdleCallback`[7] ，这是一个较新的 API，它可以插入一个函数，这个函数将在浏览器空闲时期被调用。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如果不支持的话，就使用 settimeout

### 平台数据如何进行 削峰限流 
有某一个时间点，突然间流量爆炸，无数的数据向服务器访问过来，这时如果没有一个削峰限流的策略，很可能会导致机器Down掉  
所以有必要去做一个削峰限流，从概率学的角度上讲，在大数据量的基础上我们对于整体数据做一个百分比的截断，并不会影响整体的一个数据比例。  
**简单方案-随机丢弃策略进行限流**  
让用户传入一个采样率
``` 
if(Math.random()<0.5) return;
```
但是这个方案不是一个很优雅的解决办法  
- 大流量项目限制了 50% 的流量，它的流量仍然多；
- 小流量项目限制了 50% 的流量，那就没有流量了

**优化方案-流量整型**  
流量整形的方法很多，最常见的就是三种：  
- 计数器算法：计数器算法就是单位时间内入库数量固定，后面的数据全部丢弃；缺点是无法应对恶意用户；
- 漏桶算法：漏桶算法就是系统以固有的速率处理请求，当请求太多超过了桶的容量时，请求就会被丢弃；缺点是漏桶算法对于骤增的流量来说缺乏效率；
- 令牌桶算法：令牌桶算法就是系统会以恒定的速度往固定容量的桶里放入令牌，当请求需要被处理时就会从桶里取一个令牌，当没有令牌可取的时候就会据拒绝服务；

分析一下：  
- 计数器能够削峰，限制最大并发数以保证服务高可用
- 令牌桶实现流量均匀入库，保证下游服务健康

最终选择了 计数器 + 令牌桶 的方案
- 首先从外部来的流量是我们无法预估的，假设如上图我们有三个 服务器Pod ，如果总流量来的非常大，那么这时我们通过计数器算法，给它设置一个很大的最大值；这个最大值只防小人不防君子，可能 99% 的项目都不会触发
- 这样经过总流量的计数器削峰后，再到中心化的令牌桶限流：通过 redis 来实现，我们先做一个定时器每分钟都去令牌桶里写令牌，然后单机的流量每个进来后，都去 redis 里取令牌，有令牌就处理入库；没有令牌就把流量抛弃
- 这样子我们就实现了一个 单机的削峰 + 中心化的限流，两者一结合，就是解决了小流量应用限流后没流量的问题，以及控制了入库的数量均匀且稳定

### 平台数据为什么需要 数据加工
数据上报之后，因为 IP地址 是在服务端获取的嘛，所以服务端就需要有一个服务，去统一给请求数据中家加上 IP地址 以及 IP地址 解析后的归属地、运营商等信息；  
根据业务需要，还可以加上服务端服务的版本号 等其余信息，方便后续做追踪；

### 平台数据为什么需要 数据清洗、聚合 
- 数据清洗是为了白名单、黑名单过滤等的业务需要，还有避免已关闭的应用数据继续入库；
- 数据聚合是为了将相同信息的数据进行抽象聚合成 issue，以便查询和追踪；

假设后续我们需要在数据库查询：某一条错误，产生了几次，影响了几个人，错误率是多少，这样子可以不用再去 ES 中捞日志，而是在 MySQL 中直接查询即可  
还可以将抽象聚合出来的 issue ，关联于公司的 缺陷平台（类bug管理平台） ，实现 issue追踪 、 直接自动贴bug到负责人头上 等业务功能  
### 平台数据如何进行 多维度追踪
首先会对每一个用户（user），会去生成一个 用户id（uid ；并对每一次会话（session），生成一个 会话id（sid）；  
uid 和 sid 都是28位的随机ID，sid 和 uid 都在初始化时生成，不同的是，因为 uid 的生命周期只在一次会话之中（关闭页签之前），所以 sid 我们存放在 sessionStorage 中，而 uid 我们存放在 cookie 里，过期时间设置六个月  
每次SDK初始化时，都先去 cookie 和 sessionStorage 里取 uid 和 sid，如果取不到就重新生成一份；并且在每次数据上报时，都将这些 id 附带上去  
如果有需要，还可以再搞一个登录id，由使用方传入，专门存放登录成功后的登录态ID；  
这样一系列搞完之后，收集了很多的行为数据，包括PV访问、路由跳转、http请求、click事件、自定义事件、甚至第三章的错误数据等等；这些种种零零散散的数据就可以被串联起来，得到新的分析价值；  
### 代码错误如何进行 源码映射
通过解析错误堆栈，得到了错误的文件、行列号等信息，可以通过对 sourcemap 可以对源码进行映射，定位错误源码的位置
生产环境是不可以将 sourcemap 文件发布上线的，可以通过手动上传到监控平台的形式去进行错误的分析定位；  
### 如何设计监控告警的维度
- 宏观告警 更加关注的是：一段时间区间内，新增异常的数量、比率是否超过了阈值；如果超过了那就进行告警；
- 微观告警 更加关注的是：是否有新增的、且未解决的异常；

只要出现的新异常，它的 uid 是当前已激活的异常中全新的一个；那么就进行告警，通知大群、通知负责人、在缺陷平台上新建 bug 指派给负责人  
**监控告警如何指派给代码提交者**  
当发现新 bug 产生时，可以将这个 bug 指派给负责人；这里其实还可以做的更细致一点，可以做一个 处理人自动分配 的机制；  
处理人自动分配，分配给谁呢？捕获错误时上报了错误的位置，也就是源码所在；那么只需要找到最近一次提交这行代码的人就可以了；  
出错行author 的原理其实就是 Git Blame  


原文: 
[腾讯三面：说说前端监控平台/监控SDK的架构设计和难点亮点？](https://mp.weixin.qq.com/s/dUoKdzJDKdK1vDgkw4LQ_A)
