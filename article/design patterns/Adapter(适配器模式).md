# Adapter（适配器模式）
Adapter（适配器模式）属于结构型模式，别名 wrapper，结构性模式关注的是如何组合类与对象，以获得更大的结构。  
将一个类的接口转换成客户希望的另一个接口。Adapter 模式使得原本由于接口不兼容而不能在一起工作的那些类可以一起工作。

## 举例
**接口转换器**  
插座的种类很多，我们都用过许多适配器，将不同的插头进行转换，可以在不替换插座的情况下正常使用。  
USB 接口转换也同样精彩，有将 TypeC 接口转换为 TypeA 的，也有将 TypeA 接口转换为 TypeC 的，支持双向转换。  
接口转换器就是在生活中使用到的适配器模式，因为厂商并没有生产一个新的插座，我们也没有因为接口不适配而换一个手机，一切只需要一个接口转换器即可  
**数据库 ORM**  
ORM 屏蔽了 SQL 这一层，带来的好处是不需要理解不同 SQL 语法之间的区别，对于通用功能，ORM 会根据不同的平台，比如 Postgresql、Mysql 进行 SQL 的转换。对 ORM 来说，屏蔽不同平台的差异，就是利用适配器模式做到的。 
**API Deprecated**  
当一个广泛使用的库进行了含有 break change 的升级时，往往要留给开发者足够的时间去升级，而不能升级后就直接挂掉，因此被废弃的 API 要标记为 deprecated，而这种被废弃标记的 API 的实际实现，往往是使用新的 API 替代，这种场景正是使用了适配器模式，将新的 API 适配到旧的 API，实现 API Deprecated。
## 解释
上面三个例子都满足下面两个条件：  
1. API 不兼容：因为接口的不同；数据库 SQL 语法的不同；框架 API 的不同。
2. 能力已支持：插座都拥有充电或读取能力；不同的 SQL 都拥有查询数据库能力；新 API 覆盖了旧 API 的能力。

Adapter 适配器，把 Adeptee 适配成 Target。  
Adaptee 被适配的内容，比如不兼容的接口。  
Target 适配为的内容，比如需要用的接口。  

继承：  

![image](./../../assets/images/design%20patterns/Adapter%20extend.png)  

适配器继承 Adaptee 并实现 Target，适用场景是 Adaptee 与 Target 结构类似的情况，因为这样只需要实现部分差异化即可。  

组合：  

![image](./../../assets/images/design%20patterns/Adapter%20combination.png)  

继承：
``` 
interface ITarget {
  // 标准方式是 hello
  hello: () => void
}

class Adaptee {
  // 要被适配的类方法叫 sayHello
  sayHello() {
    console.log('hello')
  }
}

// 适配器继承 Adaptee 并实现 ITarget
class Adapter extends Adaptee implements ITarget {
  hello() {
    // 用 sayHello 对接到 hello
    super.sayHello()
  }
}
```
组合：  
``` 
interface ITarget {
  // 标准方式是 hello
  hello: () => void
}

class Adaptee {
  // 要被适配的类方法叫 sayHello
  sayHello() {
    console.log('hello')
  }
}

// 适配器继承 Adaptee 并实现 ITarget
class Adapter implements ITarget {
  private adaptee: Adaptee 

  constructor(adaptee: Adaptee) {
    this.adaptee = adaptee
  }

  hello() {
    // 用 adaptee.sayHello 对接到 hello
    this.adaptee.sayHello()
  }
}
```
## 缺点
使用适配器模式本身就可能是个问题，因为一个好的系统内部不应该做任何侨界，模型应该保持一致性。只有在如下情况才考虑使用适配器模式：  
1. 新老系统接替，改造成本非常高。
2. 三方包适配。
3. 新旧 API 兼容。
4. 统一多个类的接口。一般可以结合工厂方法使用

原文: 
[Adapter（适配器模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/172.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Adapter%20%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
