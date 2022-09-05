# Decorator(装饰器模式)
Decorator（装饰器模式）属于结构型模式，是一种拓展对象额外功能的设计模式，别名 wrapper。动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator 模式相比生成子类更为灵活。  
## 举例
**相框**  
照片 + 相框 = 带相框的照片，这背后就是一种装饰器模式：照片具有看的功能，相框具有装饰功能，在你看照片的基础上，还能看到精心设计的相框，增加了美感，同时相框还可以增加照片的保存时间与安全性。  
相框与照片是一种组合关系，任何照片都可以放到相框中，而不是每个照片生成一个特定的相框，显然，组合的方式更加灵活。  
**带有缓存的文件读写**  
假设我们有一个类 FileIO 用来读写文件，但是没有缓存能力，此时是新建一个 CachedFileIO 子类好，还是创建一个 CachedIO?  
看上去好像 CachedFileIO 用起来更方便，而 CachedIO 的用法是 new CachedIO(new FileIO()) 稍微麻烦一些，但如果我们增加一个网络读写类 NetworkIO，一个数据库读写类 DBIO 呢？  
继承的方式会使子类数量极速膨胀，而组合的方式则非常灵活，生成一个支持缓存的网络读写器，只需要 new CachedIO(new NetworkIO()) 即可，这就是组合灵活的地方。
为了实现这个能力，CachedIO 需要与 FileIO、CachedFileIO、CachedIO 继承自同一个类，具备相同的接口。  
**搭建平台的组件 wrapper**  
装饰器模式别名也叫 wrapper，wrapper 也经常在前端搭建场景中遇到，当搭建平台加载一个组件时，希望拓展其基础能力，一般会使用 wrapper 层对组件进行嵌套，wrapper 层就是在不改变 API 的基础上，对第三方组件进行增强
## 解释
不同于继承，组合可以在运行时进行，所以称之为 “动态添加”，这里的 “额外职责” 泛指一切功能，比如在按钮点击时进行一些 log 日志的打印，在绘制 text 文本框时，额外绘制一个滚动条和边框等等。  
“就增加功能来说，Decorator 模式相比生成子类更为灵活” 这句话的含义是，组合比继承更灵活，当可拓展的功能很多时，继承方案会产生大量的子类，而组合可以提前写好处理函数，在需要时动态构造，显然是更灵活的。

![image](./../../assets/images/design%20patterns/Decorator.png)  

ConcreteComponent 指的是需要被装饰的组件，可以看到，装饰器 Decorator 与他都继承同一个类，这样能保证 API 的一致，才保证无论装饰多少层，始终符合 Component 类型。  
装饰器如果有多种，就要将 Decorator 申明为抽象类，ConcreteDecoratorA、ConcreteDecoratorB 分别实现它们，如果只有一种装饰器，可以退化到 Decorator 自身就是一种实现
``` 
class Component {
  // 具有点击事件
  public onClick = () => {}
}

class Decorator extends Component {
  private _component

  constructor(component) {
    this._component = component
  }

  public onClick = () => {
    log('打点')
    this._component.onClick()
  }
}

const component = new Component()
// 一个普通的点击
component.onClick()

const wrapperComponent = new Decorator(component)
// 一个具有打点功能的点击
wrapperComponent.onClick()
```
通过组合，我们得到了一个能力更强的组件，而实现的方式就是利用构造函数保存组件实例，并在复写函数时，增加一些增强实现。  
## 缺点
装饰器的问题也是组合的问题，过多的组合会导致：  
- 组合过程的复杂，要生成过多的对象。
- 包装器层次增多，会增加调试成本，我们比较难追溯到一个 bug 是在哪一层包装导致的。

原文: 
[Decorator（装饰器模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/175.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Decorator%20%E8%A3%85%E9%A5%B0%E5%99%A8%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
