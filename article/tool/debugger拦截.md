# debugger拦截
debugger 指令，一般用于调试，在如浏览器调试执行环境中，可以在 JavaScript 代码中产生中断。  
如果想要拦截 debugger，是不容易的，常用的函数替代、proxy 方法均对它无效，如：  
``` 
window.debugger = (function() {
     var origDebug = console.debugger;
     return function() {
     // do something before debugger statement execution
     origDebug.apply(console, arguments);
     // do something after debugger statement execution
     };
 })();
```
或：  
``` 
var handler = {
   get: function(target, prop, receiver) {
     if (prop === 'debugger') {
       throw new Error("Debugger statement not allowed!");
     }
     return Reflect.get(target, prop, receiver);
   }
 };
 var obj = new Proxy({}, handler);
```
以上两方法，都无法对 debugger 生效。
而 debugger 有多种写法，如：
1. debugger
2. Function("debugger").call()
3. eval("debugger")
4. setInterval(function(){debugger;},1000)
5. [].constructor.constructor('debugger')()

最原始的 debugger，想要拦截这一个单词，确实是似乎不可行，但它在现实中的使用频率是不高的，更多的是后面几种用法。  
这是因为，debugger 更多的被人们用于反调试，比如用 JShaman 对 JavaScript 代码进行混淆加密后，就可以被加入多种不同的 debugger 指令用于反调试。
**Function("debugger").call()**  
拦截示例：  
``` 
Function_backup = Function;
 Function = function(a){
     if (a =='debugger'){
         console.log("拦截了debugger，中断不会发生1")
         return Function_backup("console.log()")
     }else{
         return Function_backup(a)
     }
 }
 Function("debugger").call();
```
**eval("debugger")**  
拦截示例：  
``` 
eval_backup = eval;
 eval = function(a){
 if(a=='debugger'){
 console.log("拦截了debugger，中断不会发生0")
         return ''
     }else{
         return eval_backup(a)
     }
 }
 eval("debugger");
```
**setInterval(function(){debugger;},1000)**  
拦截示例：
``` 
var setInterval_backup = setInterval
 setInterval = function(a,b){
     if(a.toString().indexOf('debugger') != -1){
         console.log("拦截了debugger，中断不会发生2")
         return null;
     }
     setInterval_backup(a, b)
 }
 setInterval(function(){
     debugger;
 },1000);
```
**[].constructor.constructor('debugger')()**  
拦截示例：  
``` 
var constructor_backup = [].constructor.constructor;
 [].constructor.constructor = function(a){
     if(a=="debugger"){
         console.log("拦截了debugger，中断不会发生3");
     }else{
         constructor_backup(a);
     }
 }
 try {
     [].constructor.constructor('debugger')();
 } catch (error) {
     console.error("Anti debugger");
 }
```

原文:  
[JavaScript奇技淫巧：debugger拦截](https://mp.weixin.qq.com/s/P2rzXCilBboM7DvAQ9gVLg)
