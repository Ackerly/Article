# JS 实现网络测速
名词解释：
- ping：给目标 IP 地址发送一个 ICMP 报文，再要求对方返回一个大小相同的数据包来确定两台网络机器是否能正常通信以及有多少时延。
- jitter：抖动，用来描述网络的波动情况。比如每秒测量一次 ping 值，5s 后取五次测量结果的最大最小值求差，可以看出网络的波动情况，差值越小代表网络越稳定
- bandwidth：带宽，用来描述理论上单位时间内网络传输数据的最高速率，它只是一个理论上的最大值。通常我们所说的 1 兆带宽就是 1Mb/s = 1000Kb/s = (1000/8)KB/s = 125KB/s。带宽越大自然越好，但是受用户计算机性能、网络设备质量、资源使用情况、网络高峰期、网站服务能力、线路衰耗，信号衰减等多因素的影响，它并不能直接反映当前的网络环境
- throughput：吞吐量，用来描述单位时间内网络传输数据的实际速率。受多方面影响，吞吐量一般都小于真正的带宽值
- 上传速度与下载速度：上传速度就是从本地上传一个文件的速度，相反，下载速度就是从网络上下载一个文件的速度，使用迅雷等下载软件的时候看到的那个数值就是下载速度，通常下载速度会大于上传速度。由于下载场景较多，我们更关心下载速度数值。

在网络检测中，没有单独哪一个指标可以说明问题，应尽可能多的结合各种指标来全面评估网络情况。接下来介绍几种测速方法。  

## Network Information API
浏览器为我们提供了网络相关的 API ，NetworkInformation 提供了设备与网络进行通信的信息和链接类型变更时的有关事件，它是通过 Navigator 的 connection 属性进行访问的。connection 对象有一个 downlink 属性，返回以 Mb/s 为单位的有效带宽，MDN 上官方解释说该值是基于最近监测的保持活跃连接的应用层吞吐量，因此吞吐量的查询并不是实时的，如果距离上一个 http 请求间隔较长，这个数值并不准确。因此 downlink 值只具备有限的参考意义，且该功能还在实验中，不同浏览器兼容性也较差，因此不推荐使用这种方式来检测网络情况。
补充一下，connection 对象还有一个 type 属性和 onChange 方法，type 属性返回的是当前设备联网的类型，枚举值有如下几种：  
- bluetooth
- cellular
- ethernet
- none
- wifi
- wimax
- other
- unknown

当联网类型 type 发生改变时，会触发 change 事件，通过 onChange 回调函数能做一些事情：比如我们在播放视频时，从 wifi 环境切换为使用流量时，可以暂停视频并提示用户选择是否用流量继续播放。  

## 测量 ping 和 jitter 值
由于 JS 无法真正原生地测量 ping 值，因此需要提供一种替代方案来模拟。为了尽可能准确地得到 ping 值，可以通过请求一个尽量小的资源来模拟发送 ICMP 报文，记录发起请求到收到返回值的时间差。请求的内容可以是网站的 favicon.ico ，一个空文件，甚至是一个空接口（注意需要配置跨域）。但是这些方案都依赖于图片资源、文件以及接口的稳定性，如果服务挂掉的话，得到的 ping 值是有问题的。接下来通过多次测量 ping 值就可以计算代表网络波动情况的 jitter 值了。代码如下：  
``` 
const Dashboard = React.memo(() => {
  const [ping, setPing] = useState<number>(0);
  const [count, setCount] = useState<number>(0);
  const [pingList, setPingList] = useState<number[]>([]);
  const [jitter, setJitter] = useState<number>(0);

  useEffect(() => {
    const timer = setInterval(() => {
      const img = new Image();
      const startTime = new Date().getTime();
      // 此处选择加载 github 的 favicon，大小为2.2kB
      img.src = `https://github.com/favicon.ico?d=${startTime}`;
      img.onload = () => {
        const endTime = new Date().getTime();
        const delta = endTime - startTime;
        if ((count + 1) % 5 === 0) {
          const maxPing = Math.max(delta, ...pingList);
          const minPing = Math.min(delta, ...pingList);
          setJitter(maxPing - minPing);
          setPingList([]);
        } else {
          setPingList(lastList => [...lastList, delta]);
        }
        setCount(count + 1);
        setPing(delta);
      };
      img.onerror = err => {
        console.log('error', err);
      };
    }, 3000);
    return () => clearInterval(timer);
  }, [count, pingList]);

  return (
    <PageContainer className={styles.dashboard}>
      <div className="text-center">
        <h1>欢迎使用 仓储管理系统</h1>
        <h1>PING: {ping}ms</h1>
        <h1>抖动: {jitter}ms</h1>
      </div>
    </PageContainer>
  );
});
```
以上代码是采用测量加载 github f
avicon 的时间来模拟 ping 值的，图片大小为 2.2kB，可以获得更准确的 ping 值。注意在 img.src 的 url 最后拼接上一个时间戳，保证每次都会重新发起请求，而不是使用第一次加载的图片缓存。动图效果如下，3 秒测量一次 ping 值，拿到最近 5 次 ping 值后计算一次抖动值  

## 测量下载速度
下载速度测量与上述 ping 值测量原理相同，只不过需要将下载的对象换成一个更大的资源，通过计算单位时间内下载资源的大小来测量下载速度。像下载软件迅雷，我们常看到的那个数字就是下载速度。另外迅雷还有一个很好的功能可以选择全速下载模式和不影响正常上网的模式，因为下载时可能会挤占带宽影响用户正常浏览网页。其中的原理就是迅雷在下载的时候在不停做 ping，如果发现 ping 的延迟增加，就限制下载速度，如果 ping 还高，就继续降到 ping 回归期望值。  

## 总结
如果用户感到访问的网站反应过慢，有可能是各种原因导致的，大致可以遵循以下流程进行简单排查： 
1. 打开百度等常用网站，查看是否仍然存在网速慢的情况。如果其他网站访问正常，可以确定是当前站点的问题，需要继续排查：
    - 可能是 DNS 解析问题，可以在终端输入 nslookup + 当前网站域名来检查
    - 如果 DNS 解析正常，那么有可能是网站访问量过高等原因，具体情况还需排查
2. 如果其他网站速度也比较慢，可以检查是否在下载文件，如果在下载文件也是会占用带宽的，可以选择下载软件的限制带宽功能来确保正常上网网速
3. 如果确实网络有问题，那就需要运营商维修人员来排查了，有可能是各种原因：用户计算机性能、网络设备质量、网络高峰期、线路衰耗、信号衰减等等。

注意事项：
1. 不要在页面加载的初始阶段就去测速，否则会影响 LCP 时间，建议等组件 mounted 后再测速
2. 根据业务场景合理设计测速方案，比如根据测速时机的不同分为两种方案：
    - 实时检测：设置时间间隔来实时检测网络状况，优点是当网络出现异常时可以提前告警，缺点是会浪费网络请求；
    - 人为触发检测：用户察觉网络出现异常时，再手动触发检测优点是节省网络资源，缺点是缺乏预警性；
3. 若使用加载图片的方式测量，图片 url 应拼接时间戳，防止请求时直接使用缓存。

原文:  
[JS 实现网络测速](https://mp.weixin.qq.com/s/3DxzntvcNt08XU8MNjmBeA)
