# 从零开始搞监控系统 —— SDK
前端监控地基本目的：了解当前项目实际使用的情况，有哪些异常，在追踪到后，对其进行分析，并提供合适的解决方案。  
前端监控地终极目标：1 分钟感知、5 分钟定位、10 分钟恢复。目前是初版，离该目标还比较遥远。  
SDK（采用ES5语法）取名为 shin.js，其作用就是将数据通过 JavaScript 采集起来，统一发送到后台，采集的方式包括监听或劫持原始方法，获取需要上报的数据，并通过 gif 传递数据。    
## 异常捕获
异常包括运行时错误、Promise错误、框架错误等。  
**error事件**  
为 window 注册 error 事件，捕获全局错误，过滤掉与业务无关的错误，例如“Script error.”、JSBridge告警等，还需统一资源载入和运行时错误的数据格式。  
``` 
// 定义的错误类型码
var ERROR_RUNTIME = "runtime";
var ERROR_SCRIPT = "script";
var ERROR_STYLE = "style";
var ERROR_IMAGE = "image";
var ERROR_AUDIO = "audio";
var ERROR_VIDEO = "video";
var ERROR_PROMISE = "promise";
var ERROR_VUE = "vue";
var ERROR_REACT = "react";
var LOAD_ERROR_TYPE = {
  SCRIPT: ERROR_SCRIPT,
  LINK: ERROR_STYLE,
  IMG: ERROR_IMAGE,
  AUDIO: ERROR_AUDIO,
  VIDEO: ERROR_VIDEO
};
/**
 * 监控脚本运行时的异常
 */
window.addEventListener(
  "error",
  function (event) {
    var errorTarget = event.target;
    // 过滤掉与业务无关的错误
    if (event.message === "Script error." || !event.filename) {
      return;
    }
    if (
      errorTarget !== window &&
      errorTarget.nodeName &&
      LOAD_ERROR_TYPE[errorTarget.nodeName.toUpperCase()]
    ) {
      handleError(formatLoadError(errorTarget));
    } else {
      handleError(
        formatRuntimerError(
          event.message,
          event.filename,
          event.lineno,
          event.colno,
          event.error
        )
      );
    }
  },
  true //捕获
);
/**
 * 生成 laod 错误日志
 * 需要加载资源的元素
 */
function formatLoadError(errorTarget) {
  return {
    type: LOAD_ERROR_TYPE[errorTarget.nodeName.toUpperCase()],
    desc: errorTarget.baseURI + "@" + (errorTarget.src || errorTarget.href),
    stack: "no stack"
  };
}
```
原本以为脚本、样式和图像在无响应时，也会触发 error 事件，但事与愿违，并不是这样。  
performance.getEntriesByType("resource") 读取到资源列表（由PerformanceResourceTiming组成），然后循环列表。  
当数据项的decodedBodySize属性为0时，就可判断无法读取这个资源；或者没有该属性，可认为当前资源缓存在浏览器中。  
这种判断的条件不够全，也不够精确，后面就用比较简单粗暴的方式来做判断依据，那就是 duration 大于20秒，就认为请求超时了。  
在日志中会将各个阶段的时间参数都保存，便于后期的校验。  
``` 
/**
   * 监控资源异常，即无法响应的资源
   */
   window.addEventListener(
    "load",
    function () {
      // 罗列资源列表，PerformanceResourceTiming类型
      var resources = performance.getEntriesByType("resource");
      // 映射initiatorType和错误类型
      var hashError = {
        script: ERROR_SCRIPT,
        link: ERROR_STYLE,
        // img: ERROR_IMAGE
      };
      resources && resources.forEach(function(value) {
        var type = hashError[value.initiatorType];
        /**
         * 非监控资源、响应时间在20秒内、监控资源是ma.gif或shin.js，则结束当前循环
         */
        if(!type ||                                   //非监控资源
          value.duration < 20000  ||                  //20秒内
          value.name.indexOf("ma.gif") >= 0 ||
          value.name.indexOf("shin.js") >= 0) {
          return;
        }
        // 若是CSS文件，则过滤脚本文件
        if(type === ERROR_STYLE && 
          value.name.indexOf(".js") >= 0) {
          return;
        }
        handleError({
          type: type,
          desc: handleNumber(value.toJSON()),
        });
      });
    },
    false
  );
```
主要是为了监控脚本文本的响应，因为有时候会由于脚本没响应而导致页面空白，直接影响到业务，业务人员也不可能一直盯着页面的，为了避免这种情况，就需要实时监控资源的响应状态  
**unhandledrejection事件**  
为 window 注册 unhandledrejection 事件，捕获未处理的 Promise 错误，当 Promise 被 reject 且没有 reject 处理器时触发。  
``` 
window.addEventListener(
  "unhandledrejection",
  function (event) {
    //处理响应数据，只抽取重要信息
    var response = event.reason.response;
    //若无响应，则不监控
    if (!response) {
      return;
    }
    var desc = response.request.ajax;
    desc.status = event.reason.status || response.status;
    handleError({
      type: ERROR_PROMISE,
      desc: desc
    });
  },
  true
);
```
Promise 常用于异步通信，例如axios库，当响应异常通信时，就能借助该事件将其捕获，得到的结果如下  
``` 
{
  "type": "promise",
  "desc": {
    "response": {
      "data": "Error occured while trying to proxy to: localhost:8000/monitor/performance/statistic",
      "status": 504,
      "statusText": "Gateway Timeout",
      "headers": {
        "connection": "keep-alive",
        "date": "Wed, 24 Mar 2021 07:53:25 GMT",
        "transfer-encoding": "chunked",
        "x-powered-by": "Express"
      },
      "config": {
        "transformRequest": {},
        "transformResponse": {},
        "timeout": 0,
        "xsrfCookieName": "XSRF-TOKEN",
        "xsrfHeaderName": "X-XSRF-TOKEN",
        "maxContentLength": -1,
        "headers": {
          "Accept": "application/json, text/plain, */*",
        },
        "method": "get",
        "url": "/api/monitor/performance/statistic"
      },
      "request": {
        "ajax": {
          "type": "GET",
          "url": "/api/monitor/performance/statistic",
          "status": 504,
          "endBytes": 0,
          "interval": "13.15ms",
          "network": {
            "bandwidth": 0,
            "type": "4G"
          },
          "response": "Error occured while trying to proxy to: localhost:8000/monitor/performance/statistic"
        }
      }
    },
    "status": 504
  },
  "stack": "Error: Gateway Timeout
    at handleError (http://localhost:8000/umi.js:18813:15)"
}
```
这样就能分析出 500、502、504 等响应码所占通信的比例，当高于日常数量时，就得引起注意，查看是否在哪块逻辑出现了问题。  
上面的结构中包含响应信息，这是需要对 Error 做些额外扩展的，如下所示。
``` 
import fetch from 'axios';
function handleError(errorObj) {
  const { response } = errorObj;
  if (!response) {
    const error = new Error('你的网络有点问题');
    error.response = errorObj;
    error.status = 504;
    throw error;
  }
  const error = new Error(response.statusText);
  error.response = response;
  error.status = response.status;
  throw error;
}
export default function request(url, options) {
  return fetch(url, options)
    .catch(handleError)
    .then((response) => {
      return { data: response.data };
    });
}
```
有一套项目依赖的是 jQuery 库，因此要监控此处的异常通信，需要做点改造。  
所有的通信都会请求一个通用函数，那么只要修改此函数的逻辑，就能覆盖到项目中的所有页面。  
需要声明一个 xhr() 函数，在函数中初始化 XMLHttpRequest 对象，从而才能监控它的实例。并且在 error 方法中需要手动触发 unhandledrejection 事件。  
``` 
$.ajax({
  url,
  method,
  data,
  success: (res) => {
    success(res);
  },
  xhr: function () {
    this.current = new XMLHttpRequest();
    return this.current;
  },
  error: function (res) {
    error(res);
    Promise.reject({
      status: res.status,
      response: {
        request: {
          ajax: this.current.ajax
        }
      }
    }).catch((error) => {
      throw error;
    });
  }
});
```
**框架错误**  
React 需要在项目中创建一个 ErrorBoundary 类，捕获错误。  
``` 
import React from 'react';
export default class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  componentDidCatch(error, info) {
    this.setState({ hasError: true });
    // 将component中的报错发送到后台
    shin && shin.reactError(error, info);
  }
  render() {
    if (this.state.hasError) {
      return null
      // 也可以在出错的component处展示出错信息
      // return <h1>出错了!</h1>;
    }
    return this.props.children;
  }
}
```
其中 reactError() 方法在组装错误信息  
``` 
/**
 * 处理 React 错误（对外）
 */
shin.reactError = function (err, info) {
  handleError({
    type: ERROR_REACT,
    desc: err.toString(),
    stack: info.componentStack
  });
};
```
要对 Vue 进行错误捕获，那么就得重写 Vue.config.errorHandler()，其参数就是 Vue 对象。  
``` 
/**
 * Vue.js 错误劫持（对外）
 */
shin.vueError = function (vue) {
  var _vueConfigErrorHandler = vue.config.errorHandler;
  vue.config.errorHandler = function (err, vm, info) {
    handleError({
      type: ERROR_VUE,
      desc: err.toString(), 　　//描述
      stack: err.stack 　　　　　//堆栈
    });
    // 控制台打印错误
    if (
      typeof console !== "undefined" &&
      typeof console.error !== "undefined"
    ) {
      console.error(err);
    }
    // 执行原始的错误处理程序
    if (typeof _vueConfigErrorHandler === "function") {
      _vueConfigErrorHandler.call(err, vm, info);
    }
  };
};
```
如果 Vue 是被模块化引入的，那么就得在模块的某个位置调用该方法，因为此时 Vue 不会绑定到 window 中，即不是全局变量。  
**难点**  
虽然把错误都搜集起来了，但是现代化的前端开发，都会做一次代码合并压缩混淆，也就是说，无法定位错误的真正位置。  
选择了 devtool: "hidden-source-map"，生成完成的原始代码，并在脚本中隐藏Source Map路径。  
在生成映射文件后，就需要让运维配合，编写一个脚本（在发完代码后触发），将这些文件按年月日小时分钟的格式命名（例如 202103041826.js.map），并迁移到指定目录中，用于后期的映射。  
之所以没有到秒是因为没必要，在执行发代码的操作时，发布按钮会被锁定，其他人无法再发。  
映射的逻辑是用 Node.js 实现的，会在后文中详细讲解。注意，必须要有列号，才能完成代码还原。  
## 行为搜集
将行为分成：用户行为、浏览器行为、控制台打印行为。监控这些主要是为了在排查错误时，能还原用户当时的各个动作，从而能更好的找出问题出错的原因。  
**用户行为**  
监听了点击事件，并且只会对 button 和 a 元素上注册的点击事件做监控。  
``` 
/**
 * 全局监听事件
 */
var eventHandle = function (eventType, detect) {
  return function (e) {
    if (!detect(e)) {
      return;
    }
    handleAction(ACTION_EVENT, {
      type: eventType,
      desc: e.target.outerHTML
    });
  };
};
// 监听点击事件
window.addEventListener(
  "click",
  eventHandle("click", function (e) {
    var nodeName = e.target.nodeName.toLowerCase();
    // 白名单
    if (nodeName !== "a" && nodeName !== "button") {
      return false;
    }
    // 过滤 a 元素
    if (nodeName === "a") {
      var href = e.target.getAttribute("href");
      if (
        !href ||
        href !== "#" ||
        href.toLowerCase() !== "javascript:void(0)"
      ) {
        return false;
      }
    }
    return true;
  }),
  false
);
```
**浏览器行为**  
监控异步通信，重写 XMLHttpRequest 对象，并通过 Navigator.connection 读取当前的网络环境，例如4G、3G等。  
``` 
var _XMLHttpRequest = window.XMLHttpRequest; 　　//保存原生的XMLHttpRequest
//覆盖XMLHttpRequest
window.XMLHttpRequest = function (flags) {
  var req;
  req = new _XMLHttpRequest(flags); 　　　　　　　//调用原生的XMLHttpRequest
  monitorXHR(req); 　　　　//埋入我们的“间谍”
  return req;
};
var monitorXHR = function (req) {
  req.ajax = {};
  req.addEventListener(
    "readystatechange",
    function () {
      if (this.readyState == 4) {
        var end = shin.now(); 　　　　　　　　　//结束时间
        req.ajax.status = req.status; 　　　　//状态码
        if ((req.status >= 200 && req.status < 300) || req.status == 304) {
          //请求成功
          req.ajax.endBytes = _kb(req.responseText.length * 2) + "KB"; 　　//KB
        } else {
          //请求失败
          req.ajax.endBytes = 0;
        }
        req.ajax.interval = _rounded(end - start, 2) + "ms"; 　　//单位毫秒
        req.ajax.network = shin.network();
        //只记录300个字符以内的响应
        req.responseText.length <= 300 &&
          (req.ajax.response = req.responseText);
        handleAction(ACTION_AJAX, req.ajax);
      }
    },
    false
  );

  // “间谍”又对open方法埋入了间谍
  var _open = req.open;
  req.open = function (type, url, async) {
    req.ajax.type = type; 　　　//埋点
    req.ajax.url = url; 　　　　//埋点
    return _open.apply(req, arguments);
  };

  var _send = req.send;
  var start; 　　　　//请求开始时间
  req.send = function (data) {
    start = shin.now(); 　　　　 //埋点
    if (data) {
      req.ajax.startBytes = _kb(JSON.stringify(data).length * 2) + "KB";
      req.ajax.data = data; 　　//传递的参数
    }
    return _send.apply(req, arguments);
  };
};
/**
 * 计算KB值
 */
function _kb(bytes) {
  return _rounded(bytes / 1024, 2); 　　//四舍五入2位小数
}
/**
 * 四舍五入
 */
function _rounded(number, decimal) {
  return parseFloat(number.toFixed(decimal));
}
/**
 * 网络状态
 */
shin.network = function () {
  var connection =
    window.navigator.connection ||
    window.navigator.mozConnection ||
    window.navigator.webkitConnection;
  var effectiveType = connection && connection.effectiveType;
  if (effectiveType) {
    return { bandwidth: 0, type: effectiveType.toUpperCase() };
  }
  var types = "Unknown Ethernet WIFI 2G 3G 4G".split(" ");
  var info = { bandwidth: 0, type: "" };
  if (connection && connection.type) {
    info.type = types[connection.type];
  }
  return info;
};
```
浏览器的行为还包括跳转，当前非常流行 SPA，所以在记录跳转地址时，只需监听 onpopstate 事件即可，其中上一页地址也会被记录。  
``` 
/**
 * 全局监听跳转
 */
var _onPopState = window.onpopstate;
window.onpopstate = function (args) {
  var href = location.href;
  handleAction(ACTION_REDIRECT, {
    refer: shin.refer,
    current: href
  });
  shin.refer = href;
  _onPopState && _onPopState.apply(this, args);
};
```
**控制台打印行为**  
重写 console 中的方法，对 log() 做了处理。在实际使用中发现了两个问题。  
第一个是在项目调试阶段，将数据打印在控制台时，显示的文件和行数都是 SDK 的名称和位置，无法得知真正的位置，很是别扭。
并且在 SDK 的某些位置调用 console.log() 会形成死循环。后面就加了个 isDebug 开关，在调试时就关闭监控，省心。 
``` 
function injectConsole(isDebug) {
  !isDebug &&
    ["log"].forEach(function (level) {
      var _oldConsole = console[level];
      console[level] = function () {
        var params = [].slice.call(arguments); 　　// 参数转换成数组
        _oldConsole.apply(this, params); 　　　　　 // 执行原先的 console 方法
        var seen = [];
        handleAction(ACTION_PRINT, {
          level: level,
          // 避免循环引用
          desc: JSON.stringify(params, function (key, value) {
            if (typeof value === "object" && value !== null) {
              if (seen.indexOf(value) >= 0) {
                return;
              }
              seen.push(value);
            }
            return value;
          })
        });
      };
    });
}
```
第二个就是某些要打印的变量包含循环引用，这样在调用 JSON.stringify() 时就会报错。  
## 其他
**环境信息**  
通过解析请求中的 UA 信息，可以得到操作系统、浏览器名称版本、CPU等信息。  
``` 
{
  "ua": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36",
  "browser": {
    "name": "Chrome",
    "version": "89.0.4389.82",
    "major": "89"
  },
  "engine": {
    "name": "Blink",
    "version": "89.0.4389.82"
  },
  "os": {
    "name": "Mac OS",
    "version": "10.14.6"
  },
  "device": {},
  "cpu": {}
}
```
**上报**  
上报选择了 Gif 的方式，即把参数拼接到一张 Gif 地址后，传送到后台。  
``` 
/**
 * 组装监控变量
 */
function _paramify(obj) {
  obj.token = shin.param.token;
  obj.subdir = shin.param.subdir;
  obj.identity = getIdentity();
  return encodeURIComponent(JSON.stringify(obj));
}
/**
 * 推送监控信息
 */
shin.send = function (data) {
  var ts = new Date().getTime().toString();
  var img = new Image(0, 0);
  img.src = shin.param.src + "?m=" + _paramify(data) + "&ts=" + ts;
};
```
这种方式有几个优势：  
- 兼容性高，所有的浏览器都支持。
- 不存在跨域问题。
- 不会携带当前域名中的 cookie。
- 不会阻塞页面加载。
- 相比于其他类型的图片格式（BMP、PNG等），能节约更多的网络资源。

不过这种方式也有一个问题，那就是采用 GET 的请求后，浏览器会限制 URL 的长度，也就是不能携带太多的数据，否则会报 431 错误。  
在正式使用中发现，由于做了字符判断，因此有时候会缺失查询列表的信息，而这些信息都很关键，对排查起到决定性作用，因为后面就改成了普通的POST提交，这样就不会有数据量的限制了。但存储量一下子就暴增，从原先每个月100G增加到半个月250G，保存6个月的话就要存储3T的数据，经济成本上也增加了不少  
**身份标识**  
每次进入页面都会生成一个唯一的标识，存储在 sessionStorage 中。在查询日志时，可通过该标识过滤出此用户的上下文日志，消除与他不相干的日志。  
``` 
function getIdentity() {
  var key = "shin-monitor-identity";
  //页面级的缓存而非全站缓存
  var identity = sessionStorage.getItem(key);
  if (!identity) {
    //生成标识
    identity = Number(
      Math.random().toString().substr(3, 3) + Date.now()
    ).toString(36);
    sessionStorage.setItem(key, identity);
  }
  return identity;
}
```


参考:  
[从零开始搞监控系统 —— SDK](https://mp.weixin.qq.com/s/y6oZuej94mR6KkkI3bAQag)
