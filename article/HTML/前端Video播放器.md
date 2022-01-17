# 前端Video播放器
## 传统的播放模式
大多数 Web 开发者对 <video> 都不会陌生，在以下 HTML 片段中，声明了一个 <video> 元素并设置相关的属性，然后通过 <source> 标签设置视频源和视频格式：
``` 
<video id="mse" autoplay=true playsinline controls="controls">
   <source src="https://h5player.bytedance.com/video/mp4/xgplayer-demo-720p.mp4" type="video/mp4">
   你的浏览器不支持Video标签
</video>
```
通过 Chrome 开发者工具，可以知道当播放 「xgplayer-demo-720p.mp4」 视频文件时，发了 3 个 HTTP 请求：  
头两个 HTTP 请求响应的状态码是 「206」,第一个 HTTP 请求头有一个 range: bytes=0- 首部信息，该信息用于检测服务端是否支持 Range 请求。如果在响应中存在 Accept-Ranges 首部（并且它的值不为 “none”），那么表示该服务器支持范围请求。  
Accept-Ranges: bytes 表示界定范围的单位是 bytes 。 Content-Length 是有效信息，它提供了要下载的视频的完整大小。  
### 从服务器请求特定的范围
假如服务器支持范围请求的话，你可以使用 Range 首部来生成该类请求。该首部指示服务器应该返回文件的哪一或哪几部分。  
**单一范围**  
 对于使用 REST Client 发起的 「单一范围请求」，服务器端会返回状态码为 「206 Partial Content」 的响应。而响应头中的 「Content-Length」 首部现在用来表示先前请求范围的大小（而不是整个文件的大小）。「Content-Range」 响应首部则表示这一部分内容在整个资源中所处的位置。  
**多重范围**  
Range 头部也支持一次请求文档的多个部分。请求范围用一个逗号分隔开。  
``` 
$ curl http://www.example.com -i -H "Range: bytes=0-50, 100-150"
```
因为是请求文档的多个部分，所以每个部分都会拥有独立的 「Content-Type」 和 「Content-Range」 信息，并且使用 boundary 参数对响应体进行划分。  
**条件式范围请求**  
当重新开始请求更多资源片段的时候，必须确保自从上一个片段被接收之后该资源没有进行过修改。  
「If-Range」 请求首部可以用来生成条件式范围请求：假如条件满足的话，条件请求就会生效，服务器会返回状态码为 206 Partial 的响应，以及相应的消息主体。假如条件未能得到满足，那么就会返回状态码为 「200 OK」 的响应，同时返回整个资源。该首部可以与 「Last-Modified」 验证器或者 「ETag」 一起使用，但是二者不能同时使用。  
**范围请求的响应**  
与范围请求相关的有三种状态:  
- 

参考:  
[玩转前端 Video 播放器 ](https://juejin.cn/post/6850037275579121671?utm_source=gold_browser_extension)
