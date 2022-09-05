# JS 检测网络带宽
主要是分为以下几种:  
1. 通过请求一个和服务端同域的文件，例如图片等，在前端开始请求和收到响应两个时间点分别通过 Date.now 标记 start 和 end，因为 Date.now 得出的是 1970 年 1 月 1 日 (UTC) 到当前时间经过的毫秒数，所以我们通过 end - start 求出时间差（ms），然后通过计算：
``` 
 文件大小（KB） * 1000 /( end -start )
```
而请求文件又有两种方法：通过 img 加载或者 AJAX 加载:  
- 通过创建 img 对象，设置 onload 监听回调，然后指定 src, 一旦指定 src, 图片资源就会加载，完成时 onload 回调就会调用，我们可以根据时机分别标记 start 和 end。
- 通过 AJAX 进行请求，即创建 XHR 对象，在 onreadystatechange 回调里，判断当 readystate = 4 时候加载完成，根据时机分别标记 start 和 end。

2. 通过一些 H5 的先进 API 去实现，例如这里我们可以使用的是 window.navigator.connection.downlink 去查询，但是正如你所知道的是，这类 API 都是一副德性，即老生常谈的兼容性问题，所以我们一般都是作为一种预备的手段，通过能力检测，能用就用它，不能用就通过别的方法。而且需要注意 downlink 的单位是 mbps, 转化成 KB/S 的公式是
``` 
navigator.connection.downlink * 1024 / 8;. 
```
乘 1024 可以理解，为什么后面要除 8 呢？这是因为 mbps 里的 b 指的是 bit (比特)，KB/s 里面的 B 指的是 Byte（字节），1 字节 (b)=8 比特 (bit)，所以需要除个 8  
3. 一般来说，通过请求文件测算网速，单次可能会有误差，所以我们可以请求多次并计算均值。

**前端判断网速的方法及其优缺点**  
img 加载测速：借助 img 对象加载测算网速。
- 优点：没有跨域带来的问题。
- 缺点：
  - 要自己测文件大小并提供参数 fileSize,
  - 文件必须为图片
  - 文件大小不能灵活控制

Ajax 测速：通过 Ajax 测算网速
- 优点：
  - 不用提供文件大小参数，因为可以从 response 首部获得
  - 测试的文件不一定要是图片，且数据量能灵活控制。
- 缺点：跨域问题

downlink 测速：通过 navigator.connection.downlink 读取网速  
- 优点：不需要任何参数。
- 缺点：
  - 兼容性很有问题，
  - 带宽查询不是实时的，具有分钟级别的时间间隔

**方法一**  
第一种思路是 加载一张图片，通过的加载时长和图片的大小来计算出网络带宽  
``` 
function measureBW(fn) {
     var startTime, endTime, fileSize;
     var xhr = new XMLHttpRequest();
     xhr.onreadystatechange = () => {
         if(xhr.readyState === 2){
             startTime = Date.now();
         }
         if (xhr.readyState === 4 && xhr.status === 200) {
             endTime = Date.now();
             fileSize = xhr.responseText.length;
             var speed = fileSize  / ((endTime - startTime)/1000) / 1024;
             fn && fn(Math.floor(speed))
         }
     }

     xhr.open("GET", "https://upload.wikimedia.org/wikipedia/commons/5/51/Google.png", true);
     xhr.send();
 }

 measureBW((speed)=>{
     console.log(speed + " KB/sec");  //215 KB/sec
 })
```
**方法二**  
但是考虑到 http 请求需要建立连接，以及等待响应，这些过程也会消耗一些时间，所以以上的方法可能不会准确的检测出网络带宽  
可以同时发出多次请求，来减少 http 请求建立连接，等待响应的影响，参考如下代码：  
``` 
function measureBW(fn,time) {
     time = time || 1;
     var startTime, endTime, fileSize;
     var count = time ;
     var _this = this;
     function measureBWSimple () {
         var xhr = new XMLHttpRequest();
         xhr.onreadystatechange = () => {
             if (xhr.readyState === 4 && xhr.status === 200) {
                 if(!fileSize){
                     fileSize = xhr.responseText.length;
                 }
                 count --;
                 if(count<=0){
                     endTime = Date.now();
                     var speed = fileSize * time  / ((endTime - startTime)/1000) / 1024;
                     fn && fn(Math.floor(speed));
                 }
             }
         }
         xhr.open("GET", "https://upload.wikimedia.org/wikipedia/commons/5/51/Google.png", true);
         xhr.send();
     }
     startTime = Date.now();
     for(var x = time;x>0;x--){
         measureBWSimple()
     }
 }

 measureBW((speed)=>{
     console.log(speed + " KB/sec");  //913 KB/sec
 },10)
```
> 前两种还要额外设置 http 请求头来禁止使用本地缓存（开发测试下可以在控制台 Network 面板下点击禁用缓存），要不然图片加载一次后就不会在去服务器加载，自然也测不出网络的带宽.

**方法三**  
在 Chrome65+ 的版本中，添加了一些原生的方法可以检测有关设备正在使用的连接与网络进行通信的信息。  
可以检测到网络带宽：  
``` 
function measureBW () {
     return navigator.connection.downlink;
 }
 measureBW() ;
```
navigator.connection.downlink 会返回以（兆比特 / 秒）为单位的有效带宽估计值 (参考 MDN), 这和我们常用的（KB/sec）有所差别，所以我们需要再做一下单位换算，参考如下代码：  
``` 
 function measureBW () {
     return navigator.connection.downlink * 1024 /8;   //单位为KB/sec
 }
 measureBW() ;
```
通过 navigator.connection 上的 change 事件来监听网络带宽的变化：  
``` 
navigator.connection.addEventListener('change', measureBW());
```

原文: 
[JS 检测网络带宽](https://mp.weixin.qq.com/s/XfdeRmHM-GxzSZK0VWjFmA)
