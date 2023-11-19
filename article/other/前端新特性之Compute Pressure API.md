# 前端新特性：Compute Pressure API
如果你在一个具有视频会议功能的网站上跟别人开会，会议里的人都开着摄像头。那么你的设备的 CPU 压力就会很大，因为它需要处理很多视频流。如果网站不能根据 CPU 的压力动态调整远端视频流的分辨率，那么这个网站就会出现使用卡顿，甚至出现网站崩溃的现象。  
Compute Pressure个 API 还没有被 W3C 标准化，目前还处于草案阶段，所以 MDN 上也没有相关的文档。  

## 前置条件
- 使用 Chrome 浏览器
- Chrome 浏览器版本 >= 115

如果满足前置条件，那么 demo API Status 的值是 enabled。  
首先我们点击 Start 按钮，此时 Pressure 的状态是 nominal。  
增加 1 个 worker 并开启模拟，此时 Pressure 的状态是 Fair。  
> fair 表示 CPU 的压力正常，但是有一些任务正在运行

增加到 6 个 worker 的时候，Pressure 的状态开始变为 Serious  
> serious 表示 CPU 的压力严重，但是仍然可以正常工作

增加到 8 个 worker 的时候，Pressure 的状态变为 Critical  
> critical 表示 CPU 的压力非常严重，无法正常工作

**use-Cases**
WebRTC 是 Web Real-Time Communication 的缩写。Real-Time 的意思是实时，而 Compute Pressure API 的能力也是实时反映 CPU 的压力  
这个 API 在 WebRTC 应用的场景可以用来：  
- 动态调整视频流的质量和数量
- 选择性的开启或关闭虚拟背景，滤镜，降噪等功能,依次填入表单中的信息，然后点击 Register 按钮.

## 如何使用 Compute Pressure API
首先满足前置条件,然后注册 Compute Pressure 的 origin trial,
注意这里的 Web Origin 需要填写 https 或 http,注册完成后，你会得到一个 token,然后在你的网站中添加以下 meta 标签  
``` 
<meta http-equiv="origin-trial" content="copy from your token" />
```
这样你就可以在你的网站中使用 Compute Pressure API 了。为了避免 token 过期，你可以添加以下防御性代码：  
``` 
if ("PressureObserver" in globalThis) {
   // Use PressureObserver interface
 }
```
这里用了 globalThis，因为 PressureObserver 可以在 worker 使用。 实际上，PressureObserver 支持在以下 contexts 使用：  
- DedicatedWorker
- SharedWorker
- Window

创建一个 PressureObserver 实例：  
``` 
const observer = new PressureObserver(
   (changes) => {
     /* ... */
   },
   {
     sampleRate: 0.5,
   }
 );
```
你也可以不指定 sampleRate，默认值是 1。  
sampleRate 的单位是 Hz，表示每秒采样的次数。  
举个例子如果 sampleRate 是 0.5，那么就是每 2 秒采样一次。  
这里有一个坑点，如果你的 sampleRate 设置为 2 但系统最多只能提供 1 Hz 的采样频率，那么最终 PressureObserver 的采样频率就是 1 Hz。  
demo 中看到那个 emoji 表情随着 worker 数量的增加而发生变化，前提是 PressureObserver 的 observer 了 cpu：  
``` 
observer.observe("cpu");
```
PressureObserver 还可以观察 thermals（散热）  
但是现在 Chrome 只支持 cpu，可以用 PressureObserver 的静态方法 supportedSources 判断：  
``` 
console.log(PressureObserver.supportedSources());
```
取消 observer 也很简单：  
``` 
observer.unobserve("cpu");
```
如果你也可以使用：  
``` 
 observer.disconnect();
```
取消所有的 observer，但现在 Chrome 还不支持 thermals，这个方法的便利性还没有体现出来。   
实际上我测试下来 PressureObserver 的 callback 函数的参数虽然是一个数组，但是数组的长度始终是 1。  
``` 
const observer = new PressureObserver((changes) => {
   // changes.length 始终为 1
 });
```
虽然数据结构有点奇怪，但这并不妨碍我们使用这个 API
``` 
const observer = new PressureObserver((changes) => {
   switch (changes[0].state) {
     case "nominal": {
     }
     case "fair": {
     }
     case "serious": {
     }
     case "critical": {
     }
     default: {
     }
   }
 });
```
前介绍了 PressureObserver 的 unobserver 与 disconnect 方法，那么细心的你可能注意到了，如果调用这些方法的时候 cpu 的压力发生了变化怎么办？  
PressureObserver 的 takeRecords 方法考虑到了这个问题：在 disconnect 前通过调用这个方法获取到 cpu 压力的变化。


原文:  
[探索前端新特性：Compute Pressure API](https://mp.weixin.qq.com/s/AR5a2HPyclhNX2_wazMxeg)
