# Web 端实时防挡脸弹幕
## 主流防挡脸弹幕实现原理
**点播**  
- up 上传视频
- 服务器后台计算提取视频画面中的人像区域，转换成 svg 存储
- 客户端播放视频的同时，从服务器下载 svg 与弹幕合成，人像区域不显示弹幕

**直播**  
- 主播推流时，实时（主播设备）从画面提取人像区域，转换成 svg
- 将 svg 数据合并到视频流中（SEI），推流至服务器
- 客户端播放视频同时，从视频流中（SEI）解析出 svg
- 将 svg 与弹幕合成，人像区域不显示弹幕

## 本文实现方案
客户端播放视频同时，实时从画面提取人像区域信息，将人像区域信息导出成图片与弹幕合成，人像区域不显示弹幕。  
**实现原理**  
- 采用机器学习开源库从视频画面实时提取人像轮廓，如 Body Segmentation
- 将人像轮廓转导出为图片，设置弹幕层的 mask-image

## 对比传统（直播 SEI 实时）方案
**优点：**  
- 易于实现；只需要 Video 标签一个参数，无需多端协同配合
- 无网络带宽消耗

**缺点：**  
- 理论性能极限劣于传统方案；相当于性能资源换网络资源

**面临的问题**
“JS 性能太辣鸡”，不适合执行 CPU 密集型任务。由官方 demo 变成工程实践，最大的挑战就是 —— 性能。  

## 实践调优过程
**选择机器学习模型**  
BodyPix：精确度太差，面部偏窄，有很明显的弹幕与人物面部边缘重叠现象  
BlazePose：精确度优秀，且提供了肢体点位信息，但性能较差  
```
[
   {
     score: 0.8,
     keypoints: [
       {x: 230, y: 220, score: 0.9, score: 0.99, name: "nose"},
       {x: 212, y: 190, score: 0.8, score: 0.91, name: "left_eye"},
       ...
     ],
     keypoints3D: [
       {x: 0.65, y: 0.11, z: 0.05, score: 0.99, name: "nose"},
       ...
     ],
     segmentation: {
       maskValueToLabel: (maskValue: number) => { return 'person' },
       mask: {
         toCanvasImageSource(): ...
         toImageData(): ...
         toTensor(): ...
         getUnderlyingType(): ...
       }
     }
   }
 ]
```
MediaPipe SelfieSegmentation：精确度优秀（跟 BlazePose 模型效果一致），CPU 占用相对 BlazePose 模型降低 15% 左右，性能取胜，但返回数据中不提供肢体点位信息  
返回数据结构示例  
```
{
   maskValueToLabel: (maskValue: number) => { return 'person' },
   mask: {
     toCanvasImageSource(): ...
     toImageData(): ...
     toTensor(): ...
     getUnderlyingType(): ...
   }
 }
```
**初版实现**  
参考 MediaPipe SelfieSegmentation 模型 官方实现，未做优化的情况下 CPU 占用 70% 左右  
```
const canvas = document.createElement('canvas')
 canvas.width = videoEl.videoWidth
 canvas.height = videoEl.videoHeight
 async function detect (): Promise<void> {
   const segmentation = await segmenter.segmentPeople(videoEl)
   const foregroundColor = { r: 0, g: 0, b: 0, a: 0 }
   const backgroundColor = { r: 0, g: 0, b: 0, a: 255 }

   const mask = await toBinaryMask(segmentation, foregroundColor, backgroundColor)

   await drawMask(canvas, canvas, mask, 1, 9)
   // 导出Mask图片，需要的是轮廓，图片质量设为最低
   handler(canvas.toDataURL('image/png', 0))

   window.setTimeout(detect, 33)
 }

 detect().catch(console.error)
```
**降低提取频率，平衡 性能 - 体验**  
一般视频 30FPS，尝试弹幕遮罩（后称 Mask）刷新频率降为 15FPS，体验上还能接受  
```
window.setTimeout(detect, 66) // 33 => 66
```

**重写 toBinaryMask**  
分析源码，结合打印 segmentation 的信息，发现 segmentation.mask.toCanvasImageSource 可获取原始 ImageBitmap 对象，即是模型提取出来的信息。  
尝试自行实现将 ImageBitmap 转换成 Mask 的能力，替换开源库提供的默认实现  
实现原理
```
async function detect (): Promise<void> {
   const segmentation = await segmenter.segmentPeople(videoEl)

   context.clearRect(0, 0, canvas.width, canvas.height)
   // 1. 将`ImageBitmap`绘制到 Canvas 上
   context.drawImage(
     // 经验证 即使出现多人，也只有一个 segmentation
     await segmentation[0].mask.toCanvasImageSource(),
     0, 0,
     canvas.width, canvas.height
   )
   // 2. 设置混合模式
   context.globalCompositeOperation = 'source-out'
   // 3. 反向填充黑色
   context.fillRect(0, 0, canvas.width, canvas.height)
   // 导出Mask图片，需要的是轮廓，图片质量设为最低
   handler(canvas.toDataURL('image/png', 0))

   window.setTimeout(detect, 66)
 }
```
第 2、3 步相当于给人像区域外的内容填充黑色（反向填充 ImageBitmap），是为了配合 css（mask-image）， 不然只有当弹幕飘到人像区域才可见（与目标效果正好相反）。  
globalCompositeOperation MDN：此时，CPU 占用 33% 左右  

**多线程优化**  
只剩下 toDataURL 这个耗时操作了，本以为 toDataURL 是浏览器内部实现，无法再进行优化了。  
虽没有替换实现，但可使用 OffscreenCanvas + Worker，将耗时任务转移到 Worker 中去， 避免占用主线程，就不会影响用户体验了。  
并且 ImageBitmap 实现了 Transferable 接口，可被转移所有权，跨 Worker 传递也没有性能损耗。  
```
// 前文 detect 的反向填充 ImageBitmap 也可以转移到 Worker 中
 // 用 OffscreenCanvas 实现， 此处略过

 const reader = new FileReaderSync()
 // OffscreenCanvas 不支持 toDataURL，使用 convertToBlob 代替
 offsecreenCvsEl.convertToBlob({
   type: 'image/png',
   quality: 0
 }).then((blob) => {
   const dataURL = reader.readAsDataURL(blob)
   self.postMessage({
     msgType: 'mask',
     val: dataURL
   })
 }).catch(console.error)
```
可以看到两个耗时的操作消失了,此时，CPU 占用 15% 左右  

**降低分辨率**  
Demo 足够简单很容易推测到是这行代码导致的，发现 imgStr 大概 100kb 左右（视频分辨率 1280x720）。  
```
danmakuContainer.style.webkitMaskImage = `url(${imgStr})
```
通过 canvas 缩小图片尺寸（360P 甚至更低），再进行推理。  
优化后，导出的 imgStr 大概 12kb，重新计算样式耗时约 0.5ms。此时，CPU 占用 5% 左右  

**启动条件优化**  
虽然提取 Mask 整个过程的 CPU 占用已优化到可喜程度。  
当在画面没人的时候，或没有弹幕时候，可以停止计算，实现 0 CPU 占用。  
无弹幕判断比较简单（比如 10s 内收超过两条弹幕则启动计算），也不在该 SDK 实现范围，略过  

**判定画面是否有人**  
第一步中为了高性能，选择的模型只有 ImageBitmap，并没有提供肢体点位信息，所以只能使用 getImageData 返回的像素点值来判断画面是否有人。  
画面无人时，CPU 占用接近 0%  

**发布构建优化**  
依赖包的提交较大，构建出的 bundle 体积：684.75 KiB /gzip: 125.83 KiB,所以，可以进行异步加载 SDK，提升页面加载性能。  
- 分别打包一个 loader，一个主体
- 由业务方 import loader，首次启用时异步加载主体





参考:  
[Web 端实时防挡脸弹幕（基于机器学习）](https://mp.weixin.qq.com/s/WCfCIck8HmsnzRRJ9OljgA)