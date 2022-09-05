# 如何防止他人恶意调试你的web程序
防止调试的方法主要通过不断debugger的方法来疯狂输出断点，让控制台打开后程序就无法正常执行。  
## 方法一
``` 
(() => {
    function block() {
        setInterval(() => {
            debugger;
        }, 50);
    }
    try {
        block();
    } catch (err) {}
})();
```
**如何绕过**  
禁止断点，在 Chrome 控制台的 Source Tab 页点击 Deactivate breakpoints 按钮或者按下 Ctrl + f8。
## 方法二
对对应的代码行,通过添加logpoint为 false，然后按回车后刷新网页，成功跳过无限 debugger。  
另外一种方法：  
通过add script ignore list来添加需要忽略执行代码行或文件。
**对于第一个方法**  
将setInterval(() => {debugger;}, 50);写在一行中,通过添加logpoint为 false,也没用,仍然是疯狂 debugger,,通过左下角的代码格式化,来格式一下setInterval(() => {debugger;}, 50);将它变成多行的,也是没用的,仍然会在刷新后重新弹 debugger。
**对于第二个方法**
``` 
(() => {
    function block() {
        setInterval(() => {
            Function("debugger")();
        }, 50);
    }
    try {
        block();
    } catch (err) {}
})();
```
通过将debugger改写成Function("debugger")();的形式,来应对;Function 构造器生成的 debugger 会在每一次执行时开启一个临时 js 文件,
## 强化方法
加密混淆
``` 
eval(function(c,g,a,b,d,e){d=String;if(!"".replace(/^/,String)){for(;a--;)e[a]=b[a]||a;b=[function(f){return e[f]}];d=function(){return"\\w+"};a=1}for(;a--;)b[a]&&(c=c.replace(new RegExp("\\b"+d(a)+"\\b","g"),b[a]));return c}('(()=>{1 0(){2(()=>{3("4")()},5)}6{0()}7(8){}})();',9,9,"block function setInterval Function debugger 50 try catch err".split(" "),0,{}));
```
将Function("debugger").call()改成(function(){return false;})["constructor"]("debugger")["call"]();
并且,添加条件,当窗口外部宽高,和内部宽高的差值大于一定的值,把 body 里的内容全部清空掉.  
像 toG 的项目或者是一些为了保护自己的版权又或者是一些比较敏感的项目,出于安全的考虑在部署到生产环境后最好是不让别人调试的,当然,前端所做的也就那么一些,需要前后端一起配合,便可以很好的对项目或者数据进行私密的保护.
``` 
(() => {
    function block() {
        if (
            window.outerHeight - window.innerHeight > 200 ||
            window.outerWidth - window.innerWidth > 200
        ) {
            document.body.innerHTML =
                "检测到非法调试,请关闭后刷新重试!";
        }
        setInterval(() => {
            (function () {
                return false;
            }
                ["constructor"]("debugger")
                ["call"]());
        }, 50);
    }
    try {
        block();
    } catch (err) {}
})();
```

原文: 
[如何防止他人恶意调试你的web程序](https://juejin.cn/post/7000784414858805256)

