# 深入浅出音视频与 WebRTC
## 常见的音视频网络通信协议
### 普通直播协议  
这类直播对实时性要求不那么高，使用CDN进行内容分发，会有几秒甚至十几秒的延时，主要关注画面质量、音视频是卡顿等问题，一般选用 RTMP 和 HLS 协议  
**基本概念**  
_RTMP_  
RTMP （Real Time Messaging Protocol），即“实时消息传输协议”， 它实际上并不能做到真正的实时，一般情况最少都会有几秒到几十秒的延迟，是 Adobe 公司开发的音视频数据传输的实时消息传送协议，RTMP 协议基于 TCP，包括 RTMP 基本协议及 RTMPT/RTMPS/RTMPE 等多种变种，RTMP 是目前主流的流媒体传输协议之一，对CDN支持良好，实现难度较低，是大多数直播平台的选择，不过RTMP有一个最大的不足 —— 不支持浏览器，且苹果 ios 不支持，Adobe 已停止对其更新  
_HLS_  
HLS （Http Live Streaming）是由苹果公司定义的基于 HTTP 的流媒体实时传输协议，被广泛的应用于视频点播和直播领域，HLS 规范规定播放器至少下载一个 ts 切片才能播放，所以 HLS 理论上至少会有一个切片的延迟  
HLS 在移动端兼容性比较好，ios就不用说了，Android现在也基本都支持 HLS 协议了，pc端如果要使用可以使用 hls.js 适配器  
> HLS 的原理是将整个流分为多个小的文件来下载，每次只下载若干个，服务器端会将最新的直播数据生成新的小文件，当客户端获取直播时，它通过获取最新的视频文件片段来播放，从而保证用户在任何时候连接进来时都会看到较新的内容，实现近似直播的体验；HLS 的延迟一般会高于普通的流媒体直播协议，传输内容包括两部分：一部分 M3U8 是索引文件，另一部分是 TS 文件，用来存储音视频的媒体信息

_RTMP 和 HLS 如何选择_  
- 流媒体推流，一般使用 RTMP 协议
- 移动端的网页播放器最好使用 HLS 协议，RTMP 不支持浏览器
- iOS 要使用 HLS 协议，因为不支持 RTMP 协议
- 点播系统最好使用 HLS 协议，因为点播没有实时互动需求，延迟大一些是可以接受的，并且可以在浏览器上直接观看

**普通直播基本架构**  
由直播 客户端 、 信令 服务器和 CDN 网络这三部分组成  
直播 客户端主要包括音视频数据的采集、编码、推流、拉流、解码与播放功能，但实际上这些功能并不是在同一个客户端中实现的，为什么呢？因为作为主播来说，他不需要看到观众的视频或听到观众的声音，而作为观众来讲，他们与主播之间是通过文字进行交流的，不需要向主播分享自己的音视频信息  
对于主播客户端来说，它可以设备的摄像头、麦克风采集数据，然后对采集到的音视频数据进行编码，最后将编码后的音视频数据推送给 CDN  
对于观众客户端来说，它首先需要获取到主播房间的流媒体地址，观众进入房间后从 CDN 拉取音视频数据，并对获取到的音视频数据进行解码，最后进行音视频的渲染与播放  
信令服务器，主要用于接收信令，并根据信令处理一些和业务相关的逻辑，如创建房间、加入房间、离开房间、文字聊天等  
CDN 网络，主要用于媒体数据的分发，传给它的媒体数据可以很快传送给各地的用户  

### 实时直播协议
WebRTC（Web Real-Time Communication），即“网页即时通信”，WebRTC 是一个支持浏览器进行实时语音、视频对话的开源协议，目前主流浏览器都支持WebRTC，即便在网络信号一般的情况下也具备较好的稳定性，WebRTC 可以实现点对点通信，通信双方延时低，使用户无需下载安装任何插件就可以进行实时通信  
在WebRTC发布之前，开发实时音视频交互应用的成本很高，需要考虑的技术问题很多，如音视频的编解码问题，数据传输问题，延时、丢包、抖动、回音的处理和消除等，如果要兼容浏览器端的实时音视频通信，还需要额外安装插件， WebRTC 大大降低了音视频开发的门槛，开发者只需要调用 WebRTC API 即可快速构建出音视频应用  

## 音视频设备检测
### 设备的基本原理
**音频设备**  
音频输入设备的主要工作是采集音频数据，而采集音频数据的本质就是模数转换（A/D），即将模似信号转换成数字信号，采集到的数据再经过量化、编码，最终形成数字信号，这就是音频设备所要完成的工作  
**视频设备**  
视频设备，与音频输入设备很类似，视频设备的模数转换（A/D）模块即光学传感器， 将光转换成数字信号，即 RGB（Red、Green、Blue）数据，获得 RGB 数据后，还要通过 DSP（Digital Signal Processer）进行优化处理，如自动增强、色彩饱和等都属于这一阶段要做的事情，通过 DSP 优化处理后获得 RGB 图像，然后进行压缩、传输，而编码器一般使用的输入格式为 YUV，所以在摄像头内部还有一个专门的模块用于将 RGB 图像转为 YUV 格式的图像  
YUV 也是一种色彩编码方法，它将亮度信息（Y）与色彩信息（UV）分离，即使没有 UV 信息一样可以显示完整的图像，只不过是黑白的，这样的设计很好地解决了彩色电视机与黑白电视的兼容问题（这也是 YUV 设计的初衷）相对于 RGB 颜色空间，YUV 的目的是为了编码、传输的方便，减少带宽占用和信息出错，人眼的视觉特点是对亮度更敏感，对位置、色彩相对来说不敏感，在视频编码系统中为了降低带宽，可以保存更多的亮度信息，保存较少的色差信息  

### 获取音视频设备列表  
**设备的基本原理**  
_音频设备_  
音频输入设备的主要工作是采集音频数据，而采集音频数据的本质就是模数转换（A/D），即将模似信号转换成数字信号，采集到的数据再经过量化、编码，最终形成数字信号，这就是音频设备所要完成的工作  

_视频设备_  
视频设备，与音频输入设备很类似，视频设备的模数转换（A/D）模块即光学传感器， 将光转换成数字信号，即 RGB（Red、Green、Blue）数据，获得 RGB 数据后，还要通过 DSP（Digital Signal Processer）进行优化处理，如自动增强、色彩饱和等都属于这一阶段要做的事情，通过 DSP 优化处理后获得 RGB 图像，然后进行压缩、传输，而编码器一般使用的输入格式为 YUV，所以在摄像头内部还有一个专门的模块用于将 RGB 图像转为 YUV 格式的图像  
那什么是 YUV 呢？  
UV 也是一种色彩编码方法，它将亮度信息（Y）与色彩信息（UV）分离，即使没有 UV 信息一样可以显示完整的图像，只不过是黑白的，这样的设计很好地解决了彩色电视机与黑白电视的兼容问题（这也是 YUV 设计的初衷）相对于 RGB 颜色空间，YUV 的目的是为了编码、传输的方便，减少带宽占用和信息出错，人眼的视觉特点是对亮度更敏感，对位置、色彩相对来说不敏感，在视频编码系统中为了降低带宽，可以保存更多的亮度信息，保存较少的色差信息  

**获取音视频设备列表**  
_MediaDevices.enumerateDevices()_  
此方法返回一个可用的媒体输入和输出设备的列表，例如麦克风，摄像机，耳机设备等  
``` 
navigator.mediaDevices.enumerateDevices().then(function(deviceInfos) {
  deviceInfos.forEach(function(deviceInfo) {
    console.log(deviceInfo);
  });
})
```
出于安全原因，除非用户已被授予访问媒体设备的权限（要想授予权限需要使用 HTTPS 请求），否则 label 字段始终为空
**设备检测方法**  
- 返回信息 deviceInfo 中的 kind 字段可以区分出设备是音频设备还是视频设备，同时音频设备能区分出是输入设备和输出设备，我们平时使用的耳机它是一个音频设备，但它同时兼有音频输入设备和音频输出设备的功能
- 对于音频设备和视频设备会设置各自的默认设备， 还是以耳机这个音频设备为例，将耳机插入电脑后，耳机就变成了音频的默认设备，将耳机拔出后，默认设备又切换成了系统的音频设备
- 在获取到所有的设备列表后，如果我们不指定某个具体设备，采集音视频数据时，就会从设备列表中的默认设备上采集数据，如果能从指定的设备上采集到音视频数据，那说明这个设备就是有效的设备，这样我们就可以对音视频设备进行一项一项检测
- 通过调用 getUserMedia 方法 （下面音视频采集的时候会讲到） 进行设备检测
  - 视频设备检测：调用 getUserMedia API 采集视频数据并将其展示出来，如果用户能看到自己的视频，说明视频设备是有效的，否则，设备无效
  - 音频设备检测：调用 getUserMedia API 采集音频数据，由于音频数据不能直接展示，所以需要使用 JavaScript 将其处理后展示到页面上，这样当用户看到音频数值的变化后，说明音频设备也是有效的

## 音视频采集
### 基本概念
- 帧率  
  帧率表示1秒钟视频内图像的数量，一般帧率达到 10～12fps 人眼就会觉得是连贯的，帧率越高，代表着每秒钟处理的图像数量越高，因此流量会越大，对设备的性能要求也越高，所以在直播系统中一般不会设置太高的帧率，高的帧率可以得到更流畅、更逼真的动画，一般来说 30fps 就是可以接受的，但是将性能提升至 60fps 则可以明显提升交互感和逼真感，但是一般来说超过 75fps 一般就不容易察觉到有明显的流畅度提升了
- 轨（Track）
  WebRTC 中的“轨”借鉴了多媒体的概念，两条轨永远不会相交，“轨”在多媒体中表达的就是每条轨数据都是独立的，不会与其他轨相交，如 MP4 中的音频轨、视频轨，它们在 MP4 文件中是被分别存储的

### 音视频采集接口
mediaDevices.getUserMedia  
``` 
const mediaStreamContrains = {
    video: true,
    audio: true
};

const promise = navigator.mediaDevices.getUserMedia(mediaStreamContrains).then(
    gotLocalMediaStream
)

const $video = document.querySelector('video');
 
function gotLocalMediaStream(mediaStream){
    $video.srcObject = mediaStream;
}
 
function handleLocalMediaStreamError(error){
    console.log('getUserMedia 接口调用出错: ', error);
}
```
srcObject:属性设定或返回一个对象，这个对象提供了一个与 HTMLMediaElement 关联的媒体源，这个对象通常是 MediaStream，根据规范也可以是 MediaSource, Blob 或者 File，但对于 MediaSource, Blob 和File类型目前浏览器的兼容性不太好，所以对于这几种类型可以通过 URL.createObjectURL() 创建 URL，并将其赋值给 HTMLMediaElement.src  
MediaStreamConstraints 参数，可以指定MediaStream中包含哪些类型的媒体轨（音频轨、视频轨），并且可为这些媒体轨设置一些限制  
``` 
const mediaStreamContrains = {
    video: {
       frameRate: {min: 15}, // 视频的帧率最小 15 帧每秒
       width: {min: 320, ideal: 640}, // 宽度最小是 320，理想的宽度是 640
       height: {min: 480, ideal: 720}，// 高度最小是 480，最理想高度是 720
       facingMode: 'user'， // 优先使用前置摄像头
       deviceId: '' // 指定使用哪个设备
    },
    audio: {
       echoCancellation: true, // 对音频开启回音消除功能
       noiseSuppression: true // 对音频开启降噪功能
    }
}
```
### 浏览器实现自拍  
视频是由一幅幅帧图像和一组音频构成的，所以拍照的过程其实是从连续播放的视频流（一幅幅画面）中抽取正在显示的那张画面，上面我们讲过可以通过 getUserMedia 获取到视频流，那如何从视频流中获取到正在显示的图片呢？  
这里就要用到 canvas 的 drawImage  
``` 
const ctx = document.querySelector('canvas');
// 需要拍照时执行此代码，完成拍照
ctx.getContext('2d').drawImage($video, 0, 0);

function downLoad(url){
    const $a = document.createElement("a");
    $a.download = 'photo';
    $a.href = url;
    document.body.appendChild($a);
    $a.click();
    $a.remove();
}

// 调用 download 函数进行图片下载
downLoad(ctx.toDataURL("image/jpeg"));
```
drawImage 的第一个参数支持 HTMLVideoElement 类型，所以可以直接将 $video 作为第一个参数传入，这样就通过 canvas 获取到照片了  
然后通过 a 标签的 download 将照片下载下来保存到本地  
- 通过 canvas 的 toDataURL 方法获得图片的 URL 地址
- 利用 a 标签的 downLoad 属性来实现图片的下载

## 音视频录制
### 基本概念
**ArrayBuffer**  
ArrayBuffer 对象表示通用的、固定长度的二进制数据缓冲区，可以使用它存储图片、视频等内容，但ArrayBuffer 对象不能直接进行访问，ArrayBuffer 只是描述有这样一块空间可以用来存放二进制数据，但在计算机的内存中并没有真正地为其分配空间，只有当具体类型化后，它才真正地存在于内存中  
``` 
let buffer = new ArrayBuffer(16); // 创建一个长度为 16 的 buffer
let view = new Uint32Array(buffer);
```
**ArrayBufferView**  
是Int32Array、Uint8Array、DataView等类型的总称，这些类型都是使用 ArrayBuffer 类实现的，因此才统称他们为 ArrayBufferView  
**Blob**  
Binary Large Object）是 JavaScript 的大型二进制对象类型，WebRTC 最终就是使用它将录制好的音视频流保存成多媒体文件的，而它的底层是由上面所讲的 ArrayBuffer 对象的封装类实现的，即 Int8Array、Uint8Array 等类型  
### 音频录制接口
``` 
const mediaRecorder = new MediaRecorder(stream[, options]);
```
stream参数是将要录制的流，它可以是来自于使用 navigator.mediaDevices.getUserMedia 创建的流或者来自于 audio，video 以及 canvas DOM 元素  
MediaRecorder.ondataavailable事件可用于获取录制的媒体资源 (在事件的 data 属性中会提供一个可用的 Blob 对象)  
录制的流程如下：
- 使用 getUserMedia 接口获取视频流数据
- 使用 MediaRecorder 接口进行录制（视频流数据来源上一步获取的数据）
- 使用 MediaRecorder 的 ondataavailable 事件获取录制的 buffer 数据
- 将 buffer 数据转成 Blob 类型，然后使用 createObjectURL 生成可访问的视频地址
- 利用 a 标签的 download 属性进行视频下载

``` 
<video autoplay playsinline controls id="video-show"></video>
<video id="video-replay"></video>
<button id="record">开始录制</button>
<button id="stop">停止录制</button>
<button id="recplay">录制播放</button>
<button id="download">录制视频下载</button>
```
``` 
let buffer;
const $videoshow = document.getElementById('video-show');
const promise = navigator.mediaDevices.getUserMedia({
  video: true
}).then(
  stream => {
  console.log('stream', stream);
  window.stream = stream;
  $videoshow.srcObject = stream;
})

function startRecord(){     
  buffer = [];     
  // 设置录制下来的多媒体格式 
  const options = {
    mimeType: 'video/webm;codecs=vp8'
  }

  // 判断浏览器是否支持录制
  if(!MediaRecorder.isTypeSupported(options.mimeType)){
    console.error(`${options.mimeType} is not supported!`);
    return;
  }

  try{
    // 创建录制对象
    mediaRecorder = new MediaRecorder(window.stream, options);
    console.log('mediaRecorder', mediaRecorder);
  }catch(e){
    console.error('Failed to create MediaRecorder:', e);
    return;
  }

  // 当有音视频数据来了之后触发该事件
  mediaRecorder.ondataavailable = handleDataAvailable;
  // 开始录制
  mediaRecorder.start(2000); // 若设置了 timeslice 这个毫秒值，那么录制的数据会按照设定的值分割成一个个单独的区块
}

// 当该函数被触发后，将数据压入到 blob 中
function handleDataAvailable(e){
  console.log('e', e.data);
  if(e && e.data && e.data.size > 0){
    buffer.push(e.data);
  }
}

document.getElementById('record').onclick = () => {
  startRecord();
};

document.getElementById('stop').onclick = () => {
  mediaRecorder.stop();
  console.log("recorder stopped, data available");
};

// 回放录制文件
const $video = document.getElementById('video-replay');
document.getElementById('recplay').onclick = () => {
  const blob = new Blob(buffer, {type: 'video/webm'});
  $video.src = window.URL.createObjectURL(blob);
  $video.srcObject = null;
  $video.controls = true;
  $video.play();
};

// 下载录制文件
document.getElementById('download').onclick = () => {
  const blob = new Blob(buffer, {type: 'video/webm'});
  const url = window.URL.createObjectURL(blob);
  const a = document.createElement('a');

  a.href = url;
  a.style.display = 'none';
  a.download = 'video.webm';
  a.click();
}; 
```
## 创建连接
数据采集完成，接下来就要开始建立连接，然后进行数据通信了  
要实现一套 1 对 1 的通话系统，通常我们的思路会是在每一端创建一个 socket，然后通过该 socket 与对端相连，当 socket 连接成功之后，就可以通过 socket 向对端发送数据或者接收对端的数据了，WebRTC 中提供了 RTCPeerConnection 类，其工作原理和 socket 基本一样，不过它的功能更强大，实现也更为复杂，下面就来讲讲 WebRTC 中的 RTCPeerConnection  
### RTCPeerConnection
在音视频通信中，每一方只需要有一个 RTCPeerConnection 对象，用它来接收或发送音视频数据，然而在真实的场景中，为了实现端与端之间的通话，还需要利用信令服务器交换一些信息，比如交换双方的 IP 和 port 地址，这样通信的双方才能彼此建立连接  
WebRTC 规范对 WebRTC 要实现的功能、API 等相关信息做了大量的约束，比如规范中定义了如何采集音视频数据、如何录制以及如何传输等，甚至更细的，还定义了都有哪些 API，以及这些 API 的作用是什么，但这些约束只针对于客户端，并没有对服务端做任何限制，这就导致了我们在使用 WebRTC 的时候，必须自己去实现 信令 服务  
**RTCPeerConnection 如何工作呢?**  
1. 获取本地音视频流  
   为连接的每个端创建一个 RTCPeerConnection 对象，并且给 RTCPeerConnection 对象添加一个本地流，该流是从 getUserMedia 获取的
    ```
     // 调用 getUserMedia API 获取音视频流
    navigator.mediaDevices.getUserMedia(mediaStreamConstraints).
    then(gotLocalMediaStream)
    
    function gotLocalMediaStream(mediaStream) {
    window.stream = mediaStream;
    }
    
    // 创建 RTCPeerConnection 对象
    let localPeerConnection = new RTCPeerConnection();
    
    // 将音视频流添加到 RTCPeerConnection 对象中
    localPeerConnection.addStream(stream);
    ```
2. 交换媒体描述信息
   获得音视频流后，就可以开始与对端进行媒体协商了（媒体协商就是看看你的设备都支持哪些编解码器，我的设备是否也支持？如果我的设备也支持，那么咱们双方就算协商成功了），这个过程需要通过信令服务器完成  
   现在假设 A 和 B 需要通讯
   - A 通过 createOffer方法启动创建一个 SDP offer，即得到 A 的本地会话描述
   - A 通过 setLocalDescription ****方法保存本地会话描述
   - A 通过信令服务器发送信令给 B
    ``` 
   localPeerConnection.createOffer([options])
    .then((description) => {
    // 将 offer 保存到本地
    localPeerConnection.setLocalDescription(description)
    .then(() => {
    setLocalDescriptionSuccess(localPeerConnection);
    });
    })
   ```
    - B 接收到带有 A offer 的信令，调用 setRemoteDescription，设置远程会话描述
    - B 通过 createAnswer 方法将本地会话描述成功回调
    - B 调用 setLocalDescription 设置他自己的本地局部描述回调函数中保存本地会话描述
    - B 通过信令服务器发送信令给 A
    ```  
    // B 设置远程会话描述
    remotePeerConnection.setRemoteDescription(description)
    .then(() => {
    setRemoteDescriptionSuccess(remotePeerConnection);
    });
    
    remotePeerConnection.createAnswer()
    .then((description)=> {
    // B 保存本地会话描述
    remotePeerConnection.setLocalDescription(description)
    .then(() => {
    setLocalDescriptionSuccess(remotePeerConnection);
    });
    });
   ```
    - A 通过 setRemoteDescription 将 B 的应答 answer 保存为远程会话描述
    ``` 
    // A 保存 B 的 应答 answer 为远程会话描述
    localPeerConnection.setRemoteDescription(description)
    .then(() => {
    setRemoteDescriptionSuccess(localPeerConnection);
    });
   ```
3. 端与端建立连接
   - 当 A 调用 setLocalDescription 函数成功后，会触发 icecandidate 事件（在建立通讯之前，我们需要获得双方的网络信息，例如 IP、端口等，candidate 便是用于保存这些东西的）
   ```  
   localPeerConnection.onicecandidate= function(event) {
    // 获取到触发 icecandidate 事件的 RTCPeerConnection 对象
    const peerConnection = event.target;
    // 获取到具体的 candidate
    const iceCandidate = event.candidate;
    // 将 candidate 包装成需要的格式，然后通过信令服务器发送给B
    }
   ```
   - B 接收到信令服务器传递过来的 A 的关于 candidate 的信息，把消息包装成 RTCIceCandidate 对象，然后调用 addIceCandidate 保存起来
   ``` 
   // 创建 RTCIceCandidate 对象 
    const newIceCandidate = new RTCIceCandidate(iceCandidate);
    remotePeerConnection.addIceCandidate(newIceCandidate);
   ```
   每当获得一个新的 Candidate 后，就会通过信令服务器交换给对端，对端再调用 RTCPeerConnection 对象的 addIceCandidate() 方法将收到的 Candidate 保存起来，然后按照 Candidate 的优先级进行连通性检测，如果 Candidate 连通性检测完成，那么端与端之间就建立了连接，这时媒体数据就可以通过这个连接进行传输了  

## 音视频编解码
视频是连续的图像序列，由连续的帧构成，一帧即为一幅图像，由于人眼的视觉暂留效应，当帧序列以一定的速率播放时，我们看到的就是动作连续的视频，由于连续的帧之间相似性极高，为便于储存传输，我们需要对原始的视频进行编码压缩，以去除空间、时间维度的冗余  
视频编解码是采用算法将视频数据的冗余信息去除，对图像进行压缩、存储及传输， 再将视频进行解码及格式转换， 追求在可用的计算资源内，尽可能高的视频重建质量和尽可能高的压缩比，以达到带宽和存储容量要求的视频处理技术  
视频流传输中最为重要的编解码标准有H.26X系列（H.261、H.263、H.264），MPEG系列，Apple公司的 QuickTime 等  

## 显示远端媒体流
通过 RTCPeerConnection 对象 A 与 B 双方建立连接后，本地的多媒体数据经过编码以后就可以被传送到远端了，远端收到了媒体数据解码后，怎么显示出来呢，下面以 video 为例，看看怎么让 RTCPeerConnection 获得的媒体数据与 video 标签结合起来  
当远端有数据流到来的时候，浏览器会回调 onaddstream 函数，在回调函数中将得到的 stream 赋值给 video 标签的 srcObject 对象，这样 video 就与 RTCPeerConnection 进行了绑定，video 就能从 RTCPeerConnection 获取到视频数据，并最终将其显示出来了  
``` 
localPeerConnection.onaddstream = function(event) {
  $remoteVideo.srcObject = event.stream;
}
```

原文:  
[深入浅出音视频与 WebRTC](https://mp.weixin.qq.com/s/yVnEnA1IhVde1OiNlGKN-w)
