# async await错误处理
JavaScript 是一个单线程的语言，假如不加 try ...catch ，会导致直接报错无法继续执行。当然不意味着你代码中一定要用 try...catch 包住，使用 try...catch 意味着你知道这个位置代码很可能出现报错，所以你使用了 try...catch 进行捕获处理，并让程序继续执行。
 await-to-js处理函数 
``` 
/**
 * @param { Promise } promise
 * @param { Object= } errorExt - Additional Information you can pass to the err object
 * @return { Promise }
 */
export function to<T, U = Error> (
  promise: Promise<T>,
  errorExt?: object
): Promise<[U, undefined] | [null, T]> {
  return promise
    .then<[null, T]>((data: T) => [null, data]) // 执行成功，返回数组第一项为 null。第二个是结果。
    .catch<[U, undefined]>((err: U) => {
      if (errorExt) {
        Object.assign(err, errorExt);
      }

      return [err, undefined]; // 执行失败，返回数组第一项为错误信息，第二项为 undefined
    });
}

export default to;
```
await 是在等待一个 Promise 的返回值，正常情况下，await 命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。所以只需要利用 Promise 的特性，分别在 promise.then 和 promise.catch 中返回不同的数组，其中 fulfilled 的时候返回数组第一项为 null，第二个是结果。rejected 的时候，返回数组第一项为错误信息，第二项为 undefined。使用的时候，判断第一项是否为空，即可知道是否有错误  
具体使用
``` 
import to from 'await-to-js';
// If you use CommonJS (i.e NodeJS environment), it should be:
// const to = require('await-to-js').default;

async function asyncTaskWithCb(cb) {
     let err, user, savedTask, notification;

     [ err, user ] = await to(UserModel.findById(1));
     if(!user) return cb('No user found');

     [ err, savedTask ] = await to(TaskModel({userId: user.id, name: 'Demo Task'}));
     if(err) return cb('Error occurred while saving task');

    if(user.notificationsEnabled) {
       [ err ] = await to(NotificationService.sendNotification(user.id, 'Task Created'));
       if(err) return cb('Error while sending notification');
    }

    if(savedTask.assignedUser.id !== user.id) {
       [ err, notification ] = await to(NotificationService.sendNotification(savedTask.assignedUser.id, 'Task was created for you'));
       if(err) return cb('Error while sending notification');
    }

    cb(null, savedTask);
```

参考：  
[async await 更优雅的错误处理](https://juejin.cn/post/7011299888465969166)
