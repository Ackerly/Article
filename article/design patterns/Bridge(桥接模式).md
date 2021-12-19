# Bridge(桥接模式)
Bridge（桥接模式）属于结构型模式，是一种解决继承后灵活拓展的方案。将抽象部分与它的实现部分分离，使它们可以独立地变化。
## 举例
**汽车生产线改造为新能源生产线**  
汽油车与新能源汽车的生产流程有很大相似之处，那么汽油车生产线能否快速改造为新能源汽车生产线呢？  
汽油车生产线没有将内部实现解耦，只把生产汽油车的各部分独立了出来，对新能源车生产线是没什么用处的，但如果汽油车生产线提供了更底层的能力，比如加装轮胎，加装方向盘，那么这些步骤是可以同时被汽油车与新能源车所共享的。  
**窗口（Window）类的派生**  
假设存在一个 Window 窗口类，其底层实现在不同操作系统是不一样的，假设对于操作系统 A 与 B，分别有 AWindow 与 BWindow 继承自 Window，现在要做一个新功能 ManageWindow（管理器窗口），就要针对操作系统 A 与 B 分别生成 AManageWindow 与 BManageWindow，这样显然不容易拓展。  
无论我们新增支持 C 操作系统，还是新增支持一个 IconWindow，类的数量都会成倍提升，因为所做的 AMangeWindow 与 BMangeWindow 同时存在两个即以上的独立维度，这使得增加维度时，代码变得很冗余。  
**适配多个搭建平台的物料**  
做前端搭建平台时，经常出现一些物料（组件）因为固化了某个搭建平台的 API，因此无法迁移到另一个搭建平台，如果要迁移，就需要为不同的平台写不同的组件，而这些组件中大部分 UI 逻辑都是一样的，这使得产生大量代码冗余，如果再兼容一个新搭建平台，或者为已有的 10 个搭建平台再创建一个新组件，工作量都是写一个组件的好几倍。

## 实践
桥接模式中，抽象指的是一种接口（Abstraction），实现指的也是一种接口（Implementor），其中 Implementor 并不是直接实现了 Abstraction 定义的接口，而是提供更底层的方法，使 Abstraction 可以基于它们封装出自己的接口实现。  
这样一来，Abstraction 的接口可以随意变化，只要 Implementor 提供的功能全面，Implementor 可以不变；相应的，Implementor 的实现也可以随意变化，只要提供的底层函数不变，就不影响 Abstraction 对其的使用。  

![image](./../../assets/images/design%20patterns/Bridge.png)  

- Abstraction：定义抽象类的接口。
- RefinedAbstraction：扩充 Abstraction。
- Implementor：定义实现类的接口，该接口可以与 Abstraction 接口不一致。
- ConcreteImplementor：实现 Implementor 接口并定义它的具体实现。

抽象部分就是 Abstraction，实现部分就是 Implementor，在这个结构图中，它们是分离的，可以各自独立变化的，桥接模式，就是指 imp 这个桥，通过 Implementor 实现 Abstraction 接口，就算是桥接上了，这种组合的桥接相比普通的类实现更灵活，更具有拓展性。  

``` 
class Window {
  private windowImp: WindowImp

  public drawBox() {
    // 通过画线生成 box
    this.windowImp.drawLine(0, 1)
    this.windowImp.drawLine(1, 1)
    this.windowImp.drawLine(1, 0)
    this.windowImp.drawLine(0, 0)
  }
}

// 拓展 window 就非常容易
class SuperWindow extends Window {
  public drawIcon {
    // 通过自定义画线
    this.windowImp.drawLine(0, 5)
    this.windowImp.drawLine(3, 9)
  }
}
```
Window 的能力是 drawBox，那继承 Window 容易拓展 drawIcon 吗？默认是不行的，因为 Window 并没有提供这个能力.划线是一种基础能力，不应该与 Window 代码耦合，因此我们将基础能力放到 windowImp 中，这样 drawIcon 也可以利用其基础能力画线了。  
## 缺点
不要过度抽象，桥接模式是为了让类的职责更单一，维护更便捷，但如果只是个小型项目，桥接模式会增加架构设计的复杂度，而且不正确的模块拆分，把本来关联的逻辑强制解耦，在未来会导致更大的问题。  
桥接模式也有简单与复杂模式之分，只有一种实现的场景就不要用抽象工厂做过度封装了。

参考：  
[Bridge（桥接模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/173.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Bridge%20%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
