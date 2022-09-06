# 高级 Promise 模式：Promise缓存
## 一个例子：缓存异步请求结果
一个简单的 API 客户端：  
```
const getUserById = async (userId: string): Promise<User> => {
   const user = await request.get(`https://users-service/${userId}`);
   return user;
 };
```
但是，如果要关注性能，该怎么办？users-service 解析用户详细信息可能很慢，也许经常使用相同的用户 ID 集来调用此方法。  

**简单的解决方案**  
```
 const usersCache = new Map<string, User>();

 const getUserById = async (userId: string): Promise<User> => {
   if (!usersCache.has(userId)) {
     const user = await request.get(`https://users-service/${userId}`);
     usersCache.set(userId, user);
   }

   return usersCache.get(userId);
 };
```
**并发场景**  
上面的代码，它将在以下情况下进行重复的网络调用：  
```
await Promise.all([
   getUserById('user1'),
   getUserById('user1')
 ]);
```
问题在于直到第一个调用解决后，我们才分配缓存。但是，如何在获得结果之前填充缓存？  
如果我们缓存结果的 Promise 而不是结果本身，该怎么办？代码如下：  
```
 const userPromisesCache = new Map<string, Promise<User>>();

 const getUserById = (userId: string): Promise<User> => {
   if (!userPromisesCache.has(userId)) {
     const userPromise = request.get(`https://users-service/v1/${userId}`);
     userPromisesCache.set(userId, userPromise);
   }

   return userPromisesCache.get(userId)!;
 };
```
非常相似，但是我们没有 await 发出网络请求，而是将其 Promise 放入缓存中，然后将其返回给调用方。  
> 注意，我们不需要声明我们的方法 async ，因为它不再调用 await 。我们的方法签名虽然没有改变我们仍然返回一个 promise ，但是我们是同步进行的。  

这样可以解决并发条件，无论时间如何，当我们对进行多次调用时，只会触发一个网络请求 getUserById('user1')。这是因为所有后续调用者都收到与第一个相同的 Promise 单例。  

**Promise 缓存**  
从另一个角度看，最后一个缓存实现实际上只是在记忆 getUserById！给定我们已经看到的输入后，我们只返回存储的结果（恰好是一个 Promise）。  

借助 lodash，可以将最后一个解决方案简化为：  
```
import _ from 'lodash';

 const getUserById = _.memoize(async (userId: string): Promise<User> => {
   const user = await request.get(`https://users-service/${userId}`);
   return user;
 });
```
采用了原始的无缓存实现，并放入了 _.memoize 包装器，非常简洁  

**错误处理**  
API 客户端，应考虑操作可能失败的可能性。如果内存实现已缓存了被拒绝的 Promise ，则所有将来的调用都将以同样的失败 Promise 被拒绝！  
```
import memoize from 'memoizee';

 const getUserById = memoize(async (userId: string): Promise<User> => {
   const user = await request.get(`https://users-service/${userId}`);
   return user;
 }, { promise: true});
```

原文:  
[高级 Promise 模式：Promise缓存](https://mp.weixin.qq.com/s/WLvqOIZAjEnek2vb1CAFOw)