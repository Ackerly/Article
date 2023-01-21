# 为iframe正名,你可能不需要微前端
## 优缺点分析

| 对比项         | iframe方案                                                                                               | 微前端方案                                                                       |
|-------------|--------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| 时间成本(旧项目改造) | 低,且时间成本与数量没有绝对关系,一般只需要对所有对跳转做统一拦截处理、同时保证URL同步更新能够正常前进后退即可                                              | 高,所有接口都要改成支持跨、绝对路径，所有页面都需要二次改造,如果旧页面服务端也包含大量业务逻辑，那改造成本指数上涨，且会随着页面数量都增多改造成本一直上升 |
| 时间成本(全新项目)  | 对于全新项目而言，两种方案时间成本应该相差不大                                                                                |                                                                             |
| 上下文隔离       | 浏览器原生提供的硬隔离方案，隔离效果好，几乎没有缺点                                                                             | 多多少少存在一些兼容性问题，而且会随着接入页面复杂度的增加而增加，需要投入一部分精力在兼容问题的处理                          |
| 测试回归成本      | 如果是基于旧项目改造，iframe几乎无新增回归成本，旧页面功能正常，新页面一般问题都不大，通常只需要针对URL跳转相关重点验证即可                                     | 需要所有页面全部重新回归                                                                |
| 加载速度        | 相比微前端会慢一些，但主要还是看页面自身但加载速度，如果页面原本就是JS和CSS但简单直出，那么iframe和微前端并不会有很大差别，且浏览器自带缓存功能，非第一次加载静态资源会默认缓存          | 比iframe快一些，非第一次加载速度会更快                                                      |
| 交互体验        | 由于DOM完全分离，部分交互但实现可能会受到限制，最典型但就是弹窗问题，如果iframe区域比较小，想实现一个全屏居中还带遮罩但弹窗非常困难，但如果iframe占据了大部分屏幕的时候这个问题可能就不是问题 | 没有iframe的DOM隔离限制，和普通网页开发无异                                                  |
| 前进后退,URL同步  | 默认页面跳转URL不会同步，且前进后退需要特殊处理                                                                              | 无类似问题 |

**iframe适合的场景**  
由于iframe的一些限制，部分场景并不适合用iframe，比如像下面这种iframe只占据页面中间部分区域，由于父页面已经有一个滚动条了，为了避免出现双滚动条，只能动态计算iframe的内容高度赋值给iframe，使得iframe高度完全撑满，但这样带来的问题是弹窗很难处理，如果居中的话一般弹窗都相对的是iframe内容高度而不是屏幕高度，从而导致弹窗可能看不见，如果固定弹窗top又会导致弹窗跟随页面滚动，而且稍有不慎iframe内容高度计算有一点点偏差就会出现双滚动条。  
所以：
- 如果页面本身比较简单，是一个没有弹窗、浮层、高度也是固定的纯信息展示页的话，用iframe一般没什么问题
- 如果页面是包含弹窗、信息提示、或者高度不是固定的话，需要看iframe是否占据了全部的内容区域，如果是像下图这种经典的导航+菜单+内容结构、并且整个内容区域都是iframe，那么可以放心大胆地尝试iframe，否则，需要慎重考虑方案选型

## 实战：A系统接入B系统
满足“iframe占据全部内容区域”条件的场景，iframe的几个缺点都比较好解决。下面通过一个实际案例来详细介绍将一个线上在运行的系统接入到另外一个系统的全过程。以阿里巴巴国际站旗下一站式全球收款平台，下称A系统）接入生意贷（下称B系统）为例，已知：
- ACP和生意贷都是MPA页面
- ACP系统在此之前没有接入其他系统的先例，生意贷是第一个
- 生意贷作为被接入系统，本次需要接入的一共有20多个页面，且服务端包含大量业务逻辑以及跳转控制，有些页面想看看长什么样子都非常困难，需要在Node层mock大量接口
- 接入时需要做功能删减，部分接口入参需要调整
- 生意贷除了接入到ACP系统中，之前还接入过AMES系统，本次接入需要兼容这部分历史逻辑

假设新增一个页面 /fin/base.html?entry=xxx 作为A系统承接B系统的地址，A系统有类似如下代码：  
``` 
class App extends React.Component {
    state = {
        currentEntry: decodeURIComponent(iutil.getParam('entry') || '') || '',
    };
    render() {
        return <div>
            <iframe id="microFrontIframe" src={this.state.currentEntry}/>
        </div>;
    }
}
```
### 隐藏原系统导航菜单
因为是接入到另外一个系统，所以需要将原系统的菜单和导航等都通过一个类似“hideLayout”的参数去隐藏  
**前进后退处理**  
需要特别注意的是，iframe页面内部的跳转虽然不会让浏览器地址栏发生变化，但是却会产生一个看不见的“history记录”，也就是点击前进或后退按钮（history.forward()或history.back()）可以让iframe页面也前进后退，但是地址栏无任何变化。  
所以准确来说前进后退无需做任何处理，要做的就是让浏览器地址栏同步更新即可。  

> 如果要禁用浏览器的上述默认行为，一般只能在iframe跳转时通知父页面更新整个<iframe />DOM节点

### URL的同步更新  
让URL同步更新需要处理2个问题，一个是什么时候去触发更新的动作，一个是URL更新的规律，即父页面的URL地址（A系统）与iframe的URL地址（B系统）映射关系的维护。  
保证URL同步更新功能正常需要满足这3种情况：  
- case1: 页面刷新，iframe能够加载正确页面；
- case2: 页面跳转，浏览器地址栏能够正确更新
- case3: 点击浏览器的前进或后退，地址栏和iframe都能够同步变化

**什么时候更新URL地址**  
首先想到的肯定是在iframe加载完发送一个通知给父页面，父页面通过history.replaceState去更新URL  
> 为什么不是history.pushState呢？浏览器默认会产生一条历史记录，只需要更新地址即可，如果用pushState会产生2条记录。

B系统：
``` 
<script>
var postMessage = function(type, data) {
    if (window.parent !== window) {
        window.parent.postMessage({
            type: type,
            data: data,
        }, '*');
    }
}
// 为了让URL地址尽早地更新，这段代码需要尽可能前置，例如可以直接放在document.head中
postMessage('afterHistoryChange', { url: location.href });
</script>
```
A系统：  
``` 
window.addEventListener('message', e => {
    const { data, type } = e.data || {};
    if (type === 'afterHistoryChange' && data?.url) {
        // 这里先采用一个兜底的URL承接任意地址
        const entry = `/fin/base.html?entry=${encodeURIComponent(data.url)}`;
        // 地址不一样才需要更新
        if (location.pathname + location.search !== entry) {
            window.history.replaceState(null, '', entry);
        }
    }
});
```
**优化URL的更新速度**  
按照上面的方法实现后可以发现，URL虽然可以更新但是速度有点慢，点击跳转后一般需要等待7-800毫秒地址栏才会更新，有点美中不足。可以把地址栏的更新在“跳转后”基础之上再加一个“跳转前”。为此必须有一个全局的beforeRedirect钩子，先不考虑它的具体实现：  
B系统：
``` 
function beforeRedirect(href) {
    postMessage('beforeHistoryChange', { url: href });
}
```
A系统：  
``` 
window.addEventListener('message', e => {
    const { data, type } = e.data || {};
    if ((type === 'beforeHistoryChange' || type === 'afterHistoryChange') && data?.url) {
        // 这里先采用一个兜底的URL承接任意地址
        const entry = `/fin/base.html?entry=${encodeURIComponent(data.url)}`;
        // 地址不一样才需要更新
        if (location.pathname + location.search !== entry) {
            window.history.replaceState(null, '', entry);
        }
    }
});
```
加上上述代码之后，点击iframe中的跳转链接，URL会实时更新，浏览器的前进后退功能也正常。  
> 为什么需要同时保留跳转前和跳转后呢？因为如果只保留跳转前，只能满足前面的case1和case2，case3无法满足，也就是点击后退按钮只有iframe会后退，URL地址不会更新

**美化URL地址**  
简单的使用/fin/base.html?entry=xxx这样的通用地址虽然能用，但是不太美观，而且很容易被人看出来是iframe实现的，比较没有诚意，所以如果被接入系统的页面数量在可枚举范围内，建议给每个地址维护一个新的短地址。  
首先，新增一个SPA页面/fin/*.html，和前面的/fin/base.html指向同一个页面，然后维护一个URL地址的映射，类似这样：  
``` 
// A系统地址到B系统地址映射
const entryMap = {
    '/fin/home.html': 'https://fs.alibaba.com/xxx/home.htm?hideLayout=1',
    '/fin/apply.html': 'https://fs.alibaba.com/xxx/apply?hideLayout=1',
    '/fin/failed.html': 'https://fs.aibaba.com/xxx/failed?hideLayout=1',
    // 省略
};
const iframeMap = {}; // 同时再维护一个子页面 -> 父页面URL映射
for (const entry in entryMap) {
    iframeMap[entryMap[entry].split('?')[0]] = entry;
}
class App extends React.Component {
    state = {
        currentEntry: decodeURIComponent(iutil.getParam('entry') || '') || entryMap[location.pathname] || '',
    };
    render() {
        return <div>
            <iframe id="microFrontIframe" src={this.state.currentEntry}/>
        </div>;
    }
}
```
同时完善一下更新URL地址部分：  
``` 
// base.html继续用作兜底
let entry = `/fin/base.html?entry=${encodeURIComponent(data.url)}`;
const [path, search] = data.url.split('?');
if (iframeMap[path]) {
    entry = `${iframeMap[path]}?${search || ''}`;
}
// 地址不一样才需要更新
if (location.pathname + location.search !== entry) {
    window.history.replaceState(null, '', entry);
}
```
### 全局跳转拦截
为什么一定要做全局跳转拦截呢？一个因为需要把hideLayout参数一直透传下去，否则就会点着点着突然出现双菜单  
另一个是有些页面在被嵌入前是当前页面打开的，但是被嵌入后不能继续在当前iframe打开，比如支付宝付款这种第三方页面，这类页面一定要做特殊处理让它跳出去而不是当前页面打开。  
URL跳转可以分为服务端跳转和浏览器跳转，浏览器跳转又包括A标签跳转、location.href跳转、window.open跳转、historyAPI跳转等；  
根据是否新标签打开又可以分为以下4种场景：
1. 继续当前iframe打开，需要隐藏原系统的所有layout；
2. 当前父页面打开第三方页面，不需要任何layout；
3. 新开标签打开第三方页面（如支付宝页面），不需要做特殊处理
4. 新开标签打开宿主页面，需要把原系统layout替换成新layout

为此，先定义好一个beforeRedirect方法，由于新标签打开有target="_blank"和window.open等方式，父页面打开有target="_parent"和window.parent.location.href等方式，为了更好的统一封装，把特殊情况的跳转统一在beforeRedirect处理好，并约定只有有返回值的情况才需要后续继续处理跳转：  
``` 
// 维护一个需要做特殊处理的第三方页面列表
const thirdPageList = [
    'https://service.alibaba.com/',
    'https://sale.alibaba.com/xxx/',
    'https://alipay.com/xxx/',
    // ...
];
/**
 * 封装统一的跳转拦截钩子，处理参数透传和一些特殊情况
 * @param {*} href 要跳转的地址，允许传入相对路径
 * @param {*} isNewTab 是否要新标签打开
 * @param {*} isParentOpen 是否要在父页面打开
 * @returns 返回处理好的跳转地址，如果没有返回值则表示不需要继续处理跳转
 */
function beforeRedirect(href, isNewTab) {
    if (!href) {
        return;
    }
    // 传过来的href可能是相对路径，为了做统一判断需要转成绝对路径
    if (href.indexOf('http') !== 0) {
        var a = document.createElement('a');
        a.href = href;
        href = a.href;
    }
    // 如果命中白名单
    if (thirdPageList.some(item => href.indexOf(item) === 0)) {
        if (isNewTab) {
            // _rawOpen参见后面 window.open 拦截
            window._rawOpen(href);
        } else {
            // 第三方页面如果不是新标签打开就一定是父页面打开
            window.parent.location.href = href;
        }
        return;
    }
    // 需要从当前URL继续往下透传的参数
    var params = ['hideLayout', 'tracelog'];
    for (var i = 0; i < params.length; i++) {
        var value = getParam(params[i], location.href);
        if (value) {
            href = setParam(params[i], value, href);
        }
    }
    if (isNewTab) {
        let entry = `/fin/base.html?entry=${encodeURIComponent(href)}`;
        const [path, search] = href.split('?');
        if (iframeMap[path]) {
            entry = `${iframeMap[path]}?${search || ''}`;
        }
        href = `https://payment.alibaba.com${entry}`;
        window._rawOpen(href);
        return;
    }
    // 如果是以iframe方式嵌入，向父页面发送通知
    postMessage('beforeHistoryChange', { url: href });
    return href;
}
```
### 服务端跳转拦截
服务端主要是对301或302重定向跳转进行拦截，以Egg为例，只要重写 ctx.redirect 方法即可  
**A标签跳转拦截**  
``` 
document.addEventListener('click', function (e) {
    var target = e.target || {};
    // A标签可能包含子元素，点击目标可能不是A标签本身，这里只简单判断2层
    if (target.tagName === 'A' || (target.parentNode && target.parentNode.tagName === 'A')) {
        target = target.tagName === 'A' ? target : target.parentNode;
        var href = target.href;
        // 不处理没有配置href或者指向JS代码的A标签
        if (!href || href.indexOf('javascript') === 0) {
            return;
        }
        var newHref = beforeRedirect(href, target.target === '_blank');
        // 没有返回值一般是已经处理了跳转，需要禁用当前A标签的跳转
        if (!newHref) {
            target.target = '_self';
            target.href = 'javascript:;';
        } else if (newHref !== href) {
            target.href = newHref;
        }
    }
}, true);
```
**location.href拦截**  
location.href拦截至今是一个困扰前端界的难题，这里只能采用一个折中的方法：
``` 
// 由于 location.href 无法重写，只能实现一个 location2.href = ''
if (Object.defineProperty) {
    window.location2 = {};
    Object.defineProperty(window.location2, 'href', {
        get: function() {
            return location.href;
        },
        set: function(href) {
            var newHref = beforeRedirect(href);
            if (newHref) {
                location.href = newHref;
            }
        },
    });
}
```
不仅实现了location.href的写，location.href的读也一起实现了，所以可以放心大胆的进行全局替换。找到对应前端工程，首先全局搜索window.location.href，批量替换成(window.location2 || window.location).href，然后再全局搜索location.href，批量替换成(window.location2 || window.location).href  
**window.open拦截**  
``` 
var tempOpenName = '_rawOpen';
if (!window[tempOpenName]) {
    window[tempOpenName] = window.open;
    window.open = function(url, name, features) {
        url = beforeRedirect(url, true);
        if (url) {
            window[tempOpenName](url, name, features);
        }
    }
}
```
history.pushState拦截  
``` 
var tempName = '_rawPushState';
if (!window.history[tempName]) {
    window.history[tempName] = window.history.pushState;
    window.history.pushState = function(state, title, url) {
        url = beforeRedirect(url);
        if (url) {
            window.history[tempName](state, title, url);
        }
    }
}
```
**history.replaceState拦截**  
``` 
var tempName = '_rawReplaceState';
if (!window.history[tempName]) {
    window.history[tempName] = window.history.replaceState;
    window.history.replaceState = function(state, title, url) {
        url = beforeRedirect(url);
        if (url) {
            window.history[tempName](state, title, url);
        }
    }
}
```
### 全局loading处理
完成上述步骤后，基本上已经看不出来是iframe了，但是跳转的时候中间有短暂的白屏会有一点顿挫感，体验不算很流畅，这时候可以给iframe加一个全局的loading，开始跳转前显示，页面加载完再隐藏：  
B系统：  
``` 
document.addEventListener('DOMContentLoaded', function (e) {
    postMessage('iframeDOMContentLoaded', { url: location.href });
});
```
A系统：  
``` 
window.addEventListener('message', (e) => {
    const { data, type } = e.data || {};
    // iframe 加载完毕
    if (type === 'iframeDOMContentLoaded') {
        this.setState({loading: false});
    }
    if (type === 'beforeHistoryChange') {
        // 此时页面并没有立即跳转，需要再稍微等待一下再显示loading
        setTimeout(() => this.setState({loading: true}), 100);
    }
});
```
除此之外还需要利用iframe自带的onload加一个兜底，防止iframe页面没有上报 iframeDOMContentLoaded 事件导致loading不消失：  
``` 
// iframe自带的onload做兜底
iframeOnLoad = () => {
    this.setState({loading: false});
}
render() {
    return <div>
        <Loading visible={this.state.loading} tip="正在加载..." inline={false}>
            <iframe id="microFrontIframe" src={this.state.currentEntry} onLoad={this.iframeOnLoad}/>
        </Loading>
    </div>;
}
```
还需要注意，当新标签页打开页面时并不需要显示loading，需要注意区分  

### 弹窗居中问题
当前场景下弹窗个人觉得并不需要处理，因为菜单的宽度有限，不仔细看的话甚至都没注意到弹窗没有居中  
如果非要处理的话也不麻烦，覆盖一下原来页面弹窗的样式，当包含hideLayout参数时，让弹窗的位置分别向左移动menuWidth/2、向上移动navbarHeight/2即可（遮罩位置不能动、也动不了）  



原文:  
[为iframe正名，你可能并不需要微前端](https://juejin.cn/post/7185070739064619068)
