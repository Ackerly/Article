# Prototype(原型模式)
Prototype（原型模式）属于创建型模式，既不是工厂也不是直接 New，而是以拷贝的方式创建对象。
## 举例
**做钥匙**  
为了房屋安全，要尽量做到一把钥匙只能开一扇门，每把钥匙结构都多多少少不一样，却又很相似，做钥匙的人按照你给的钥匙一模一样做一个新的，这属于什么模式呢  
**两种状态表**  
当网站做不停机维护时，假设维护内容是给每个高级会员账户多打 100 元现金，现在需要改数据库表。已知：  
1. 数据库表有几千万条数据，其中高级会员有几千位，为了方便调用已经缓存在中间层了，且数据库对应 ID 更新后对应缓存也会更新。
2. 几千条数据修改语句执行完需要几分钟，这几分钟内无法接受用户数据不同步的问题。

一种常见的做法是，我们生成一份高级会员列表的拷贝，代替数据库缓存的结果，数据库只要读到对应会员 ID 就从拷贝列表中获取，数据表新增一列状态标志，操作完后这个拷贝移除，更新高级会员缓存。  
但是如何生成高级会员列表拷贝呢？如果直接从几千万条用户数据中重新查询，会有较高的数据库查询成本。
**模版组件**  
通用搭建系统中，我们可以将某个拖拽到页面的区块设置为 “模版”，这个模版可以作为一个新组件被重新拖拽到任意位置，实例化任意次。实际上，这是一种分段式复制粘贴，你会如何实现这个功能呢？  
## 解释
就是基于已有对象进行复制即可，效率比 New 一个，或者工厂模式都要高。  
原型实例，就是被选为拷贝模版的那个对象，比如做钥匙例子中，你给老板的样板钥匙；两种状态表中的已有缓存高级会员列表；模版组件中选中的那个组件。然后，通过拷贝这些原型创建你想要的对象即可。
如果每把钥匙都遵循 Prototype 接口，提供了 clone() 方法以复制自己，那就可以快速复制任意一把钥匙。钥匙工厂可无法解决每把钥匙不一样的问题，我们要的就是和某个钥匙一模一样的副本，复制一份钥匙最简单。  
高级会员状态表例子中，查询数据库的成本是高昂的，但如果仅仅复制已经查询好的列表，时间可以忽略不计，因此最经济的方案是直接复制，而不是通过工厂模式重新连接数据库并执行查询。  
模版组件更是如此，我们根本没有定义那么多组件实例的基类，只要每个组件提供一个 clone() 函数，就可以立即复制任意组件实例，这无疑是最经济实惠的方案。  
原型模式的拷贝建议用深拷贝，毕竟新对象最好不要影响到旧对象，但是在深拷贝性能问题较大的情况下，可以考虑深浅拷贝结合，也就是将在新对象中，不会修改的数据使用浅拷贝，可能被修改的数据使用深拷贝。  

![image](./../../assets/images/design%20patterns/Prototype.png)  

Client 是发出指令的客户端，Prototype 是一个接口，描述了一个对象如何克隆自身，比如必须拥有 clone() 方法，而 ConcretePrototype 就是克隆具体的实现，不同对象有不同的实现来拷贝自身。
``` 
class Component implements Prototype {
  /**
   * 组件名
   */
  private name: string
  /**
   * 组件版本
   */
  private version: string

  /**
   * 拷贝自身
   */
  public clone = () => {
    // 构造函数省略了，大概就是传递 name 和 version
    return new Component(this.name, this.version)
  }
}
```
实现了 Prototype 接口的 Component 必须实现 clone 方法，这样任意组件在执行复制时，就可以直接调用 clone 函数，而不用关心每个组件不同的实现方式了。  
一般来说，如果要二次修改生成的对象，不建议给 clone 函数加参数，因为这样会导致接口的不一致。 可以为对象实例提供一些 set 函数进行二次修改。另外，clone 函数要考虑性能，就像前面说过的，可以考虑深浅拷贝结合的方式，同时要注意当对象存在引用关系甚至循环引用时，甚至不一定能实现拷贝函数。  
## 缺点
- 每个类都要实现 clone 方法，对类的实现是有一定入侵的，要修改已有类时，违背了开闭原则。
- 当类又调用了其他对象时，如果要实现深拷贝，需要对应对象也实现 clone 方法，整体链路可能会特别长，实现起来比较麻烦。
原文: 
[Prototype（原型模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/170.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Prototype%20%E5%8E%9F%E5%9E%8B%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
