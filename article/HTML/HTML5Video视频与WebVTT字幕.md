# HTML5Video视频与WebVTT字幕
## 概念
HTML5 Video视频支持外挂字幕，后缀名是.vtt，称为WebVTT格式，专门的web字幕格式。使用很简单，用一个<track>元素即可。
例如：
``` 
<video id="video">
    <source src="example.mp4" type="video/mp4">
    <track src="example.vtt" default>
</video>
```
.vtt文件内容：
``` 
WEBVTT

00:00:00.001 --> 00:00:01.000
字幕1

00:00:01.001 --> 00:00:03.500
字幕2

00:00:03.501 --> 00:00:07.000
字幕3

00:00:07.501 --> 00:00:10.000
字幕4
```
只要src属性地址OK，同时又default属性，字幕就会生效。.vtt文件就是文本文件，先声明WebVTT，然后就是视频时间范围，下一行就是字幕内容，时间可以精确到毫秒，但通常0.5秒足矣。  
.vtt文件的MIME type是text/vtt。在Chrome和FireFox浏览器下，.vtt字幕是可以无障碍加载显示的，但对于IE10+浏览器，虽然也支持.vtt字幕，但是去需要定义MIME type，否则会无视WebVTT格式。比较简单的方式就是在字幕所在文件夹下面添加.htaccess文件，里面写上AddType text/vtt .vtt。  
通常我们保存在电脑的外挂字幕都不是VTT而是srt格式等，需要用在web中，可以使用工具转一下就好。

## HTML5Video视频与track元素
track完整写法：  
```
<track src="example.vtt" kind="subtitles" label="中文字幕" srclang="zh" default>
```
### kind属性
用来表明文字轨迹是干嘛用的，默认值是subtitles，如果没有添加kind属性，kind会被认为subtitles；如果有kind属性但是不合法会被认为是metadata。合法值包括：
- subtitles
就是平常看电影看动漫时候下面出现的字幕，一般是翻译，或采访时候口音不清的字幕显示。有时还会标注一些说明，例如西安市人物姓名身份，当前场景或者标注之前语言的梗在哪里等。
- captions
这里captions专指隐藏时字幕，隐藏字幕是电视节目或影碟有特殊情况或者需要的观众而准备的字幕。例如观众听力有障碍，或者需要无音条件下观赏节目。此时字幕可以使用一些解释性语言来描述节目内容  
subtitles和captions几乎看不到任何区别，有区别的应该是在语义上，或者字幕性质上。subtitles主要就是对人说话进行翻译或确认；而captions不仅需要人对话的内容提示，紧张的背景音乐，或者汽车吱吱作响的刹车声都需要在字幕中描述出来。这样，即使用户静音也能知道视频里到底在玩些什么。我想，如果是经常看国外影视作品的小伙伴肯定会有类似的字幕体验，有的就对话字幕，有的事无巨细，就是subtitles和captions的区别
- control
可以显示CC标示按钮，可以切换字幕，可以关闭字幕
- descriptions
对视频内容的文本描述，可以让盲人用户知道这个视频描述内容，如果设置kind为descriptions，VTT内容不会在屏幕上出现。当视频地址不可见的场合也有类似的作用。
- chapters
用户流量媒体资源时候出现的章节标题
- metadata
元信息。用户不可见，给脚本用的。如自定义字幕效果
- label
点击CC按钮出现的文字
- srclang
VTT文本信息使用的语言，例如中文zh、英文en
- default
默认会显示的字幕，default之鞥呢出现在一个track上
## HTML5 Video视频字幕的样式控制
CSS有专门的伪元素::cue可以控制的样式
可以控制的CSS样式包括
- color
- opacity
- visibility
- text-decoration及相关属性
- text-shadow
- background及相关属性
- outline及相关属性
- font及相关属性，包括line-height
- white-space
- text-combine-upright
- ruby-position
除此之外WebVTT还支持HTML标签进行样式控制，常见有声音v标签，颜色c标签，加粗b标签，倾斜i标签，下划线u标签，还有ruby和lang标签等。
列入：
```
::cue(v[voice=hanmeimei]) {
   color: red;
}
::cue(v[voice=lilei]) {
   color: green;
}
```
可以标签控制样式
``` 
video::cue(i) {
    color: blue;
}
```
类名控制样式
``` 
video::cue(.red) {
    color: red;
}
```

原文:  
[HTML5 Video视频与WebVTT字幕](https://www.zhangxinxu.com/wordpress/2018/03/html5-video-webvtt-subtitle/)
