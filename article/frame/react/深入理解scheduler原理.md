# 深入理解scheduler原理
react可以运行在node环境中和浏览器环境中，所以在不同环境下实现requesHostCallback等函数的时候采用了不同的方式，其中在node环境下采用setTimeout来实现任务的及时调用，浏览器环境下则使用MessageChannel。react为什么放弃了requesIdleCallback和setTimeout而采用MessageChannel来实现?  
1. 由于requestIdleCallback依赖于显示器的刷新频率，使用时需要看vsync cycle（指硬件设备的频率）
2. MessageChannel方式也会有问题，会加剧和浏览器其它任务的竞争
3. 为了尽可能每帧多执行任务，采用了5ms间隔的消息event发起调度，也就是这里真正有必要使用postmessage来传递消息
4. 浏览器在后台运行时postmessage和requestAnimationFrame、setTimeout的具体差异还不清楚,但是需要确认

react团队做这一改动可能是react团队更希望控制调度的频率，根据任务的优先级不同，提高任务的处理速度，放弃本身对于浏览器帧的依赖。优化react的性能   

**调度的实现**  
调度中心比较重要的函数在SchedulerHostConfig.default.js中  
该js文件一共导出了8个函数  
```
export let requestHostCallback;//请求及时回调 
export let cancelHostCallback;
export let requestHostTimeout;
export let cancelHostTimeout;
export let shouldYieldToHost;
export let requestPaint;
export let getCurrentTime;
export let forceFrameRate;
```
**调度相关**  
- requestHostCallback
- cancelHostCallbac
- requestHostTimeout
- requestHostTimeout

这几个函数的代码量非常少，它们的作用就是用来通知消息请求调用或者注册异步任务等待调用。  
**ScheduleCallback 注册任务**    
```
function unstable_scheduleCallback(priorityLevel, callback, options) {
  var currentTime = getCurrentTime();
  // 确定当前时间 startTime 和延迟更新时间 timeout
  var startTime;
  if (typeof options === 'object' && options !== null) {
    var delay = options.delay;
    if (typeof delay === 'number' && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }
  // 根据优先级不同timeout不同，最终导致任务的过期时间不同，而任务的过期时间是用来排序的唯一条件
  // 所以我们可以理解优先级最高的任务，过期时间越短，任务执行的靠前
  var timeout;
  switch (priorityLevel) {
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      break;
    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      break;
    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      break;
    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
           break;
    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      break;
  }

  var expirationTime = startTime + timeout;

  var newTask = {
    id: taskIdCounter++,
    // 任务本体
    callback,
    // 任务优先级
    priorityLevel,
    // 任务开始的时间，表示任务何时才能执行
    startTime,
    // 任务的过期时间
    expirationTime,
    // 在小顶堆队列中排序的依据
    sortIndex: -1,
  };
  if (enableProfiling) {
    newTask.isQueued = false;
  }
  // 如果是延迟任务则将 newTask 放入延迟调度队列（timerQueue）并执行 requestHostTimeout
  // 如果是正常任务则将 newTask 放入正常调度队列（taskQueue）并执行 requestHostCallback

  if (startTime > currentTime) {
    // This is a delayed task.
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);
    if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
      // All tasks are delayed, and this is the task with the earliest delay.
      if (isHostTimeoutScheduled) {
        // Cancel an existing timeout.
        cancelHostTimeout();
      } else {
        isHostTimeoutScheduled = true;
      }
      // Schedule a timeout.
      // 会把handleTimeout放到setTimeout里，在startTime - currentTime时间之后执行
      // 待会再调度
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else {
    newTask.sortIndex = expirationTime;
    // taskQueue是最小堆，而堆内又是根据sortIndex（也就是expirationTime）进行排序的。
    // 可以保证优先级最高（expirationTime最小）的任务排在前面被优先处理。
    push(taskQueue, newTask);
    if (enableProfiling) {
      markTaskStart(newTask, currentTime);
      newTask.isQueued = true;
    }
    // Schedule a host callback, if needed. If we're already performing work,
    // wait until the next time we yield.
    // 调度一个主线程回调，如果已经执行了一个任务，等到下一次交还执行权的时候再执行回调。
    // 立即调度
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    }
  }

  return newTask;
}   
```
**requestHostCallback 调度任务**  
开始调度任务，scheduleHostCallback这个变量被赋值成为了flushWork
```
const channel = new MessageChannel();
const port = channel.port2;
// 收到消息之后调用performWorkUntilDeadline来处理
channel.port1.onmessage = performWorkUntilDeadline;
requestHostCallback = function(callback) {
    scheduledHostCallback = callback;
    if (!isMessageLoopRunning) {
      isMessageLoopRunning = true;
      port.postMessage(null);
    }
  };
```
**performWorkUntilDeadline**  
这个函数主要的逻辑设置deadline为当前时间加上5ms 对应前言提到的5ms，同时开始消费任务并判断是否还有新的任务以决定后续的逻辑
```
const performWorkUntilDeadline = () => {
    if (scheduledHostCallback !== null) {
      const currentTime = getCurrentTime();
      // Yield after `yieldInterval` ms, regardless of where we are in the vsync
      // cycle. This means there's always time remaining at the beginning of
      // the message event.
      // yieldInterval 5ms
      deadline = currentTime + yieldInterval;
      const hasTimeRemaining = true;
      try {
        // scheduledHostCallback 由requestHostCallback 赋值为flushWork
        const hasMoreWork = scheduledHostCallback(
          hasTimeRemaining,
          currentTime,
        );
        if (!hasMoreWork) {
          isMessageLoopRunning = false;
          scheduledHostCallback = null;
        } else {
          // If there's more work, schedule the next message event at the end
          // of the preceding one.
          port.postMessage(null);
        }
      } catch (error) {
        // If a scheduler task throws, exit the current browser task so the
        // error can be observed.
        port.postMessage(null);
        throw error;
      }
    } else {
      isMessageLoopRunning = false;
    }
    // Yielding to the browser will give it a chance to paint, so we can
    // reset this.
    needsPaint = false;
  };
```
**flushWork 消费任务**  
消费任务的主要逻辑是在workLoop这个循环中实现的  
```
function flushWork(hasTimeRemaining, initialTime) {
  // 1. 做好全局标记, 表示现在已经进入调度阶段
  isHostCallbackScheduled = false;
  isPerformingWork = true;
  const previousPriorityLevel = currentPriorityLevel;
  try {
    // 2. 循环消费队列
    return workLoop(hasTimeRemaining, initialTime);
  } finally {
    // 3. 还原标记
    currentTask = null;
    currentPriorityLevel = previousPriorityLevel;
    isPerformingWork = false;
  }}
```
**workLoop 任务调度循环**  
```
function workLoop(hasTimeRemaining, initialTime) {
  let currentTime = initialTime;
  advanceTimers(currentTime);
  // 获取taskQueue中最紧急的任务
  currentTask = peek(taskQueue);
  while (
    currentTask !== null &&
    !(enableSchedulerDebugging && isSchedulerPaused)
  ) {
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      // This currentTask hasn't expired, and we've reached the deadline.
      // 当前任务没有过期，但是已经到了时间片的末尾，需要中断循环
      break;
    }
    const callback = currentTask.callback;
    if (typeof callback === 'function') {
      currentTask.callback = null;
      currentPriorityLevel = currentTask.priorityLevel;
      const didUserCallbackTimeout = currentTask.expirationTime <= currentTime;
      markTaskRun(currentTask, currentTime);
      const continuationCallback = callback(didUserCallbackTimeout);
      currentTime = getCurrentTime();
      if (typeof continuationCallback === 'function') {
        // 检查callback的执行结果返回的是不是函数，如果返回的是函数，则将这个函数作为当前任务新的回调。
        // concurrent模式下，callback是performConcurrentWorkOnRoot，其内部根据当前调度的任务
        // 是否相同，来决定是否返回自身，如果相同，则说明还有任务没做完，返回自身，其作为新的callback
        // 被放到当前的task上。while循环完成一次之后，检查shouldYieldToHost，如果需要让出执行权，
        // 则中断循环，走到下方，判断currentTask不为null，返回true，说明还有任务，回到performWorkUntilDeadline
        // 中，判断还有任务，继续port.postMessage(null)，调用监听函数performWorkUntilDeadline，
        // 继续执行任务
        currentTask.callback = continuationCallback;
        markTaskYield(currentTask, currentTime);
      } else {
        if (enableProfiling) {
          markTaskCompleted(currentTask, currentTime);
          currentTask.isQueued = false;
        }
        if (currentTask === peek(taskQueue)) {
          pop(taskQueue);
        }
      }
      advanceTimers(currentTime);
    } else {
      pop(taskQueue);
    }
    currentTask = peek(taskQueue);
  }
  // Return whether there's additional work
  // return 的结果会作为 performWorkUntilDeadline 中hasMoreWork的依据
  // 高优先级任务完成后，currentTask.callback为null，任务从taskQueue中删除，此时队列中还有低优先级任务，
  // currentTask = peek(taskQueue)  currentTask不为空，说明还有任务，继续postMessage执行workLoop，但它被取消过，导致currentTask.callback为null
  // 所以会被删除，此时的taskQueue为空，低优先级的任务重新调度，加入taskQueue
  if (currentTask !== null) {
    return true;
  } else {
    const firstTimer = peek(timerQueue);
    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }
    return false;
  }
}
```
workLoop本身是一个大循环，这个循环非常重要。此时实现了时间切片和fiber树的可中断渲染。首先task本身采用最小堆根据sortIndex也即expirationTime。并通过peek方法从taskQueue中取出来最紧急的任务。  
每次while循环的退出就是一个时间切片，详细看下while循环退出的条件，可以看到一共有两种方式可以退出  
1. 队列被清空：这种情况就是正常下情况。见49行从taskQueue队列中获取下一个最紧急的任务来执行，如果这个任务为null，则表示此任务队列被清空。退出workLoop循环
2. 任务执行超时：在执行任务的过程中由于任务本身过于复杂在执行task.callback之前就会判断是否超时（shouldYieldToHost）。如果超时也需要退出循环交给performWorkUntilDeadline发起下一次调度，与此同时浏览器可以有空闲执行别的任务。因为本身MessageChannel监听事件是一个异步任务，故可以理解在浏览器执行完别的任务后会继续执行performWorkUntilDeadline。

它们是怎么工作的呢？以concurrent模式下performConcurrentWorkOnRoot举例：  
```
function performConcurrentWorkOnRoot(root) {
  //省略无关代码
  const originalCallbackNode = root.callbackNode;
  // 省略无关代码
  ensureRootIsScheduled(root, now());
  if (root.callbackNode === originalCallbackNode) {
    // The task node scheduled for this root is the same one that's
    // currently executed. Need to return a continuation.
    return performConcurrentWorkOnRoot.bind(null, root);
  }
  return null;
}
```
在callbackNode === originalCallBackNode的时候会返回performConcurrentWorkOnRoot本身，也即workLoop中19~36行中的continuationCallback。那么可以大概猜测callbackNode 值在ensureRootIsScheduled函数中被修改了  
**ensureRootIsScheduled**  
```
function ensureRootIsScheduled(root: FiberRoot, currentTime: number) {
  const existingCallbackNode = root.callbackNode;

  // Check if any lanes are being starved by other work. If so, mark them as
  // expired so we know to work on those next.
  markStarvedLanesAsExpired(root, currentTime);

  // Determine the next lanes to work on, and their priority.
  const nextLanes = getNextLanes(
    root,
    root === workInProgressRoot ? workInProgressRootRenderLanes : NoLanes,
  );
  // This returns the priority level computed during the `getNextLanes` call.
  const newCallbackPriority = returnNextLanesPriority();
  // 在fiber树构建、更新完成后。nextLanes会赋值为NoLanes 此时会将callbackNode赋值为null, 表示此任务执行结束
  if (nextLanes === NoLanes) {
    // Special case: There's nothing to work on.
    if (existingCallbackNode !== null) {
      cancelCallback(existingCallbackNode);
      root.callbackNode = null;
      root.callbackPriority = NoLanePriority;
    }
    return;
  }
  // 节流防抖
  // Check if there's an existing task. We may be able to reuse it.
  if (existingCallbackNode !== null) {
    const existingCallbackPriority = root.callbackPriority;
    if (existingCallbackPriority === newCallbackPriority) {
      // The priority hasn't changed. We can reuse the existing task. Exit.
      return;
    }
    // The priority changed. Cancel the existing callback. We'll schedule a new
    // one below.
    cancelCallback(existingCallbackNode);
  }

  // Schedule a new callback.
  let newCallbackNode;
  if (newCallbackPriority === SyncLanePriority) {
    // Special case: Sync React callbacks are scheduled on a special
    // internal queue
    // 开始调度返回newCallbackNode，也即scheduler中的task.
    newCallbackNode = scheduleSyncCallback(
      performSyncWorkOnRoot.bind(null, root),
    );
  } else if (newCallbackPriority === SyncBatchedLanePriority) {
    newCallbackNode = scheduleCallback(
      ImmediateSchedulerPriority,
      performSyncWorkOnRoot.bind(null, root),
    );
  } else {
    const schedulerPriorityLevel = lanePriorityToSchedulerPriority(
      newCallbackPriority,
    );
    newCallbackNode = scheduleCallback(
      schedulerPriorityLevel,
      performConcurrentWorkOnRoot.bind(null, root),
    );
  }
  // 更新标记
  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}
```
看到了中断渲染原理是如何做的，以及注册调度任务部分、节流防抖部分的代码  

**时间切片原理**  
消费任务队列的过程中, 可以消费1~n个 task, 甚至清空整个 queue. 但是在每一次具体执行task.callback之前都要进行超时检测, 如果超时可以立即退出循环并等待下一次调用  
**可中断渲染原理**  
在时间切片的基础之上, 如果单个callback执行的时间过长。就需要task.callback在执行的时候自己判断下是否超时，所以concurrent模式下，fiber树每构建完一个单元都会判断是否超时。如果超时则退出循环并返回回调，等待下次调用，完成之前没有完成的fiber树构建。  
```
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

workLoop中还有3个相对重要的函数没分析
**advanceTimers & handleTimeout**  
```
function advanceTimers(currentTime) {
  // Check for tasks that are no longer delayed and add them to the queue.
  // 检查过期任务队列中不应再被推迟的，放到taskQueue中
  let timer = peek(timerQueue);
  while (timer !== null) {
    if (timer.callback === null) {
      // Timer was cancelled.
      pop(timerQueue);
    } else if (timer.startTime <= currentTime) {
      // Timer fired. Transfer to the task queue.
      pop(timerQueue);
      timer.sortIndex = timer.expirationTime;
      push(taskQueue, timer);
      if (enableProfiling) {
        markTaskStart(timer, currentTime);
        timer.isQueued = true;
      }
    } else {
      // Remaining timers are pending.
      return;
    }
    timer = peek(timerQueue);
  }
}

function handleTimeout(currentTime) {
  // 这个函数的作用是检查timerQueue中的任务，如果有快过期的任务，将它
  // 放到taskQueue中，执行掉
  // 如果没有快过期的，并且taskQueue中没有任务，那就取出timerQueue中的
  // 第一个任务，等它的任务快过期了，执行掉它
  isHostTimeoutScheduled = false;
  // 检查过期任务队列中不应再被推迟的，放到taskQueue中
  advanceTimers(currentTime);

  if (!isHostCallbackScheduled) {
    if (peek(taskQueue) !== null) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    } else {
      const firstTimer = peek(timerQueue);
      if (firstTimer !== null) {
        requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
      }
    }
  }
}
```
**shouldYieldToHost**  
```
shouldYieldToHost = function() {
      const currentTime = getCurrentTime();
      if (currentTime >= deadline) {
        // There's no time left. We may want to yield control of the main
        // thread, so the browser can perform high priority tasks. The main ones
        // are painting and user input. If there's a pending paint or a pending
        // input, then we should yield. But if there's neither, then we can
        // yield less often while remaining responsive. We'll eventually yield
        // regardless, since there could be a pending paint that wasn't
        // accompanied by a call to `requestPaint`, or other main thread tasks
        // like network events.
        if (needsPaint || scheduling.isInputPending()) {
          // There is either a pending paint or a pending input.
          return true;
        }
        // There's no pending input. Only yield if we've reached the max
        // yield interval.
        return currentTime >= maxYieldInterval;
      } else {
        // There's still time left in the frame.
        return false;
      }
    };
```
参考:
[深入理解 scheduler 原理](https://mp.weixin.qq.com/s/fI5iLpYC8Pp6G0NSmKqCOQ)