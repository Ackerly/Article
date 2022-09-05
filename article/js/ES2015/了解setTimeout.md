# 了解setTimeout
## setTimeout的创建和执行
消息队列是用来存储宏任务的，并且主线程会按照顺序取出队列里的任务依次执行，所以为了保证setTimeout能够在规定的时间内执行，setTimeout创建的任务不会被添加到消息队列里，与此同时，浏览器还维护了另外一个队列叫做延迟消息队列，该队列就是用来存放延迟任务，setTimeout创建的任务会被存放于此，同时它会被记住创建时间，延迟执行时间。
``` 
setTimeout(function showName() { console.log('showName') }, 1000)
setTimeout(function showName() { console.log('showName1') }, 1000) 
console.log('martincai')
```
执行顺序:  
1. 消息队列中取出宏任务进行执行(首次任务直接执行)
2. .执行setTimeout，此时会创建一个延迟任务，延迟任务的回调函数是showName，发起时间是当前的时间，延迟时间是第二个参数1000ms，然后该延迟任务会被推入到延迟任务队列
3. 执行console.log('martincai')代码
4. 从延迟队列里去筛选所有已过期任务(当前时间 >= 发起时间 + 延迟时间)，然后依次执行

可以看到showName和showName1同时执行，原因是两个延迟任务都已经过期  
循环源码：  
``` 
void MainTherad(){
  for(;;){
    // 执行消息队列中的任务
    Task task = task_queue.takeTask();
    ProcessTask(task);
    
    // 执行延迟队列中的任务
    ProcessDelayTask()
 
    if(!keep_running) // 如果设置了退出标志，那么直接退出线程循环
        break; 
  }
}
```

**删除延时任务**  
clearTimeout是windows下提供的原生方法，用于删除特定的setTimeout的延迟任务，在定义setTimeout的时候返回值就是当前任务的唯一id值，clearTimeout就会拿着id在延迟消息队列里查找对应的任务，将其踢出队列即可

## setTimeout的几个注意点
1. setTimeout会受到消息队列里的宏任务的执行时间影响，上面我们可以看到延迟消息队列的任务会在消息队列的弹出的当前任务执行完之后再执行，所以当前任务的执行时间会阻碍到setTimeout的延迟任务的执行时间

``` 
  function showName() {
    setTimeout(function show() {
      console.log('show')
    }, 0)
    for (let i = 0; i <= 5000; i++) {}
  }
  showName()
```
这里去执行一遍可以发现setTimeout并不是在0ms左右执行，中间会有明显的延迟，因为setTimeout在执行的时候首先会将任务放入到延迟消息队列里，等到showName执行完之后，才会去延迟队列里去查找已过期的任务，这里setTimeout任务会被showName耽误

2. setTimeout嵌套下会有4ms的延迟  

Chrome会把嵌套5层以上的setTimeout后当作阻塞方法，在第6次调用setTimeout的时候会自动将延时器更改为至少4ms的延迟时间

3. 未激活的页面的setTimeout更改为至少1000ms  

当前tab页面不在active状态的时候，setTimeout的延迟至少会被更改1000ms，这样做是为了减少性能消耗和电量消耗  

4. 延迟时间有最大值  

目前Chrome、Firefox等主流浏览器都是用32bit去存储延时时间，所以最大值是2的31次方 - 1
``` 
  setTimeout(() => {
    console.log(1)
  }, 2 ** 31)
```
以上代码会立即执行  

原文: 
[中高级前端不一定了解的setTimeout ｜ 网易实践小总结](https://juejin.cn/post/7032091028609990692)
