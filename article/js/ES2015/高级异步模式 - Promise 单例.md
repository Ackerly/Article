# ç
## 一次性懒惰初始化
“一次性懒惰初始化” 是一个很麻烦的操作，但实际上使用场景很普遍。例如，它通常适用于数据库客户端（Sequelize，Mongoose，TypeORM 等），或基于这些客户端的封装。  
懒惰的一次性初始化意味着数据库客户端在执行任何查询之前会根据需要初始化自身，并且只会执行一次  

**初始**  
初始化意味着使用数据库服务器进行身份验证，从连接池中获取连接或执行查询之前必须完成的所有操作  

**懒惰**  
这意味着客户端将在执行第一个查询的时候自动连接。调用者不需要显式连接数据库客户端，因为客户端封装了连接状态。  

**一次性**  
一次性意味着初始化仅发生一次。这很重要，因为例如过多的初始化可能会增加延迟或耗尽连接池。

### 简单的解决方案
公开一个 getRecord() 方法，该方法在内部调用 .connect() 执行初始化的私有方法：  
```
class DbClient {
   private isConnected: boolean;

   constructor() {
     this.isConnected = false;
   }

   private async connect() {
     if (this.isConnected) {
       return;
     }

     await connectToDatabase(); // stub
     this.isConnected = true;
   }

   public async getRecord(recordId: string) {
     await this.connect();
     return getRecordFromDatabase(recordId); // stub
   }
 }
```
乍一看，这看起来还不错。如果客户端还没连接，它将自动连接。这意味着使用者可以简单地执行查询而无需关心连接状态：  
```
 const db = new DbClient()
 const record = await db.getRecord('record1');
```
**条件竞争**   
有一个并发查询的场景：  
```
 const db = new DbClient();
 const [record1, record2] = await Promise.all([
   db.getRecord('record1'),
   db.getRecord('record2'),
 ]);
```
这可能会导致我们的数据库客户端连接两次！我们违反了 “一次性” 要求！  
问题是这样的：因为我们的数据库客户端的 .connect() 方法是异步的，所以在 .getRecord() 执行第二个调用时不太可能已经完成。this.isConnected 依然是 false。  
这似乎看起来没什么大不了的。但是，这个问题曾经真实发生在我负责的一个系统上，它造成了资源泄漏，最终导致服务器瘫痪  

**单例 Promise**  
可以引入一个额外的 isConnectionInProgress 布尔值，用于记录第一个 .connect() 调用的 Promise 的引用。然后，我们可以保证在执行任何将来的查询之前，该 Promise 已得到解决：  
``` 
class DbClient {
   private connectionPromise: Promise<void> | null;

   constructor() {
     this.connectionPromise = null;
   }

   private async connect() {
     if (!this.connectionPromise) {
       this.connectionPromise = connectToDatabase(); // stub
     }

     return this.connectionPromise;
   }

   public async getRecord(recordId: string) {
     await this.connect();
     return getRecordFromDatabase(recordId); // stub
   }
 }
```
由于变量 this.connectionPromise 是同步分配的，因此 .getRecord() 可以确保重复调用始终重用相同的 Promise 。这意味着第二个 .getRecord() 调用将等到第一个调用 .connect() 解决后再继续  
**一个实验**  
如何在 connectionPromise 不等待的情况下使用它，以及如何调用 await this.connectionPromise 解决已解决的问题？  
之所以可行，是因为仍可以等待已解决的 Promise 。（这实际上是 await Promise.resolve() 工作方式，因为它 Promise.resolve() 返回了已解决的 Promise。）  
浏览器的 JS 控制台中运行该实验:  
```
 const sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms));
 const myPromise = sleep(5000); // Note we don't `await` yet.

 console.time('first await');
 await myPromise;
 console.timeEnd('first await');

 console.time('second await');
 await myPromise;
 console.timeEnd('second await');
```
它输出：  
```
 first await: 5002ms - timer ended
 second await: 0ms - timer ended
```
实验表明：
- 可以多次等待同样的 Promise 。
- 可以等待已经解决的 Promise ，并且将立即解决

原文: 
[高级异步模式 - Promise 单例](https://mp.weixin.qq.com/s/ESgIwnPAS83CHRTwWpGj_A)
