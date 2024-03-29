# Singleton(单例模式)
Singleton（单例模式）属于创建型模式，提供一种对象获取方式，保证在一定范围内是唯一的。
单例模式在前端体会的不明显原因有:  
1. 前端代码本身在单机运行，创建的任何变量都是天然分布式的，不需要担心影响另一个用户
2. 后端代码是一对多的，分辨出哪些资源是请求间共享的，哪些是请求内独有的很重要

单例是隐含了一个范围的，指的是在某个范围内单例，比如在一个上下文中，还是一个房间中，还是一个进程，一个线程中单例，不同场景范围会不同。  
## 举例
**多人游戏的共享物品**  
在每局游戏中使用的公共物品在当前房间中是唯一的，但在游戏房间间却不是唯一的，所以这些公共物品肯定有不同的类去描述，那每局游戏中怎么拿公共物品，可以保证拿到的是当前局内唯一的？
**Redux 数据流**  
前端的 Redux 数据流本身就是单例模式，在一个应用中，数据是唯一的，但可以有不同的 UI 使用这份唯一的数据，甚至把一个表格组件展示在两个不同地方，比如全屏模式，但数据依然是一份，我们没有必要为了全屏展示表格，就让它再发一次取数请求，完全可以和原来的表格共享一份数据。  
**数据库连接池**  
每个 SQL 查询都依赖数据库连接池，如果每次查询都建立一次数据库连接池，则建立连接的速度会远远慢于 SQL 查询速度，因此你会怎么设计数据库连接池的获取方法
## 解释
对于多人游戏的共享物品，比如一口锅，要保证在一局游戏内唯一，就要提供一种方法访问到唯一实例。  
Redux 数据流的 connect 装饰器就是全局访问点的一种设计  
数据库连接池可以提前初始化好，并通过固定 API 提供这个唯一实例  

![image](./../../assets/images/design%20patterns/Singleton.png)  

``` 
class Ball {
  private _instance = undefined

  // 构造函数申明为 private，就可以阻止 new Ball() 行为
  private constructor() {}

  public static getInstance = () => {
    if (this._instance === undefined) {
      this._instance = new Ball()
    }

    return this._instance
  }
}

// 使用
const ball = Ball.getInstance()
```
实际上单例模式还有几种模式
- 饿汉式  
初始化时就生成一份实例，这样调用时直接就能获取
- 懒汉式  
按需实例化，即调用的时候再实例化  
按需不一定是什么好事，如果 New 的成本很高还按需实例化，可能把系统异常的风险留到随机的触发时机，导致难以排查 BUG，另外也会影响第一次实例化时的系统耗时。  

对 JAVA 来说，单例还需要考虑并发性，有 双重检测、静态内部类、枚举 等办法解决
## 缺点
单例模式的问题有：  
- 对面向对象不太友好。对封装、继承、多态支持不够友好。
- 不利于梳理类之间的依赖关系。毕竟单例是直接调用的，而不是在构造函数申明的，所以要梳理关系要看完每一行代码才能确定。
- 可拓展性不好。万一要支持多例就比较难拓展，比如全局数据流可能因为微前端方案改成多实例、数据库连接池为了分治 SQL改成多实例，都是有可能的，在系统设计之初就要考虑到未来是否还会保持单例。
- 可测试性不好，因为单例是全局共享的，无法保证测试用例间的隔离。
- 无法使用构造函数传参。

原文: 
[Singleton（单例模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/171.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Singleton%20%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
