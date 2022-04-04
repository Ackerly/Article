# 基于 TypeScript 理解程序设计的 SOLID 原则
SOLID 由 罗伯特·C·马丁 在 21 世纪早期引入，指代了面向对象编程和面向对象设计的五个基本原则， SOLID 其实是以下五个单词的缩写：  
- Single Responsibility Principle：单一职责原则
- Open Closed Principle：开闭原则
- Liskov Substitution Principle：里氏替换原则
- Interface Segregation Principle：接口隔离原则
- Dependency Inversion Principle：依赖倒置原则

**单一职责原则（SRP）**  
> 核心思想：类的职责应该单一，不要承担过多的职责。

为 Book 创建了一个类，但是类中却承担了多个职责，比如把书保存为一个文件：  
``` 
class Book {
  public title: string;
  public author: string;
  public description: string;
  public pages: number;

  // constructor and other methods

  public saveToFile(): void {
    // some fs.write method to save book to file
  }
}
```
遵循单一职责原则，我们应该创建两个类，分别负责不同的事情：  
``` 
class Book {
  public title: string;
  public author: string;
  public description: string;
  public pages: number;

  // constructor and other methods
}

class Persistence {
  public saveToFile(book: Book): void {
    // some fs.write method to save book to file
  }
}
```
> 好处：降低类的复杂度、提高可读性、可维护性、扩展性、最大限度的减少潜在的副作用

**开闭原则（OCP）**  
> 核心思想：类应该对扩展开放，但对修改关闭。简单理解就是当别人要修改软件功能的时候，不能让他修改我们原有代码，尽量让他在原有的基础上做扩展。

单独封装了一个 AreaCalculator 类来负责计算 Rectangle 和 Circle 类的面积。想象一下，如果我们后续要再添加一个形状，我们要创建一个新的类，同时我们也要去修改 AreaCalculator 来计算新类的面积，这违反了开闭原则。  
``` 
class Rectangle {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }
}

class Circle {
  public radius: number;

  constructor(radius: number) {
    this.radius = radius;
  }
}

class AreaCalculator {
  public calculateRectangleArea(rectangle: Rectangle): number {
    return rectangle.width * rectangle.height;
  }

  public calculateCircleArea(circle: Circle): number {
    return Math.PI * (circle.radius * circle.radius);
  }
}
```
只需要添加一个名为 Shape 的接口，每个形状类（矩形、圆形等）都可以通过实现它来依赖该接口。通过这种方式，我们可以将 AreaCalculator 类简化为一个带有参数的函数，每当我们创建一个新的形状类，都必须实现这个函数，这样就不需要修改原有的类了：  
``` 
interface Shape {
  calculateArea(): number;
}

class Rectangle implements Shape {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  public calculateArea(): number {
    return this.width * this.height;
  }
}

class Circle implements Shape {
  public radius: number;

  constructor(radius: number) {
    this.radius = radius;
  }

  public calculateArea(): number {
    return Math.PI * (this.radius * this.radius);
  }
}

class AreaCalculator {
  public calculateArea(shape: Shape): number {
    return shape.calculateArea();
  }
}
```
**里氏替换原则（LSP）**  
> 在使用基类的的地方可以任意使用其子类，能保证子类完美替换基类。简单理解就是所有父类能出现的地方，子类就可以出现，并且替换了也不会出现任何错误。

子类的所有相同方法，都必须遵循父类的约定，否则当父类替换为子类时就会出错。  
下面这段代码，Square 类扩展了 Rectangle 类。但是这个扩展没有任何意义，因为我们通过覆盖宽度和高度属性来改变了原有的逻辑。  
``` 
class Rectangle {
  public width: number;
  public height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  public calculateArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  public _width: number;
  public _height: number;

  constructor(width: number, height: number) {
    super(width, height);

    this._width = width;
    this._height = height;
  }
}
```
遵循里氏替换原则，我们不需要覆盖基类的属性，而是直接删除掉 Square 类并，将它的逻辑带到 Rectangle 类，而且也不改变其用途。
> 好处：增强程序的健壮性，即使增加了子类，原有的子类还可以继续运行。

**接口隔离原则（ISP）**  
> 核心思想：类间的依赖关系应该建立在最小的接口上。简单理解就是接口的内容一定要尽可能地小，能有多小就多小。我们要为各个类建立专用的接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用。  

下面的代码有一个名为 Troll 的类，它实现了一个名为 Character 的接口，但是 Troll 既不会游泳也不会说话，所以它似乎不太适合实现我们的接口：  
``` 
interface Character {
  shoot(): void;
  swim(): void;
  talk(): void;
  dance(): void;
}

class Troll implements Character {
  public shoot(): void {
    // some method
  }
  
  public swim(): void {
    // a troll can't swim
  }

  public talk(): void {
    // a troll can't talk
  }

  public dance(): void {
    // some method
  }
}
```
遵循接口隔离原则，删除 Character 接口并将它的功能拆分为四个接口，然后 Troll 类只需要依赖于实际需要的这些接口。  
``` 
interface Talker {
  talk(): void;
}

interface Shooter {
  shoot(): void;
}

interface Swimmer {
  swim(): void;
}

interface Dancer {
  dance(): void;
}

class Troll implements Shooter, Dancer {
  public shoot(): void {
    // some method
  }

  public dance(): void {
    // some method
  }
}
```
**依赖倒置原则（DIP）**  
> 依赖一个抽象的服务接口，而不是去依赖一个具体的服务执行者，从依赖具体实现转向到依赖抽象接口，倒置过来

这段代码的SoftwareProject 类，它初始化了 FrontendDeveloper 和 BackendDeveloper 类：
``` 
class FrontendDeveloper {
  public writeHtmlCode(): void {
    // some method
  }
}

class BackendDeveloper {
  public writeTypeScriptCode(): void {
    // some method
  }
}

class SoftwareProject {
  public frontendDeveloper: FrontendDeveloper;
  public backendDeveloper: BackendDeveloper;

  constructor() {
    this.frontendDeveloper = new FrontendDeveloper();
    this.backendDeveloper = new BackendDeveloper();
  }

  public createProject(): void {
    this.frontendDeveloper.writeHtmlCode();
    this.backendDeveloper.writeTypeScriptCode();
  }
}
```
遵循依赖倒置原则，我们创建一个 Developer 接口，由于 FrontendDeveloper 和 BackendDeveloper 是相似的类，它们都依赖于 Developer 接口。  
不需要在 SoftwareProject 类中以单一方式初始化 FrontendDeveloper 和 BackendDeveloper，而是将它们作为一个列表来遍历它们，分别调用每个 develop() 方法。  
``` 
interface Developer {
  develop(): void;
}

class FrontendDeveloper implements Developer {
  public develop(): void {
    this.writeHtmlCode();
  }

  private writeHtmlCode(): void {
    // some method
  }
}

class BackendDeveloper implements Developer {
  public develop(): void {
    this.writeTypeScriptCode();
  }

  private writeTypeScriptCode(): void {
    // some method
  }
}

class SoftwareProject {
  public developers: Developer[];

  public createProject(): void {
    this.developers.forEach((developer: Developer) => {
      developer.develop();
    });
  }
}
```
> 实现模块间的松耦合，更利于多模块并行开发。

参考:
[基于 TypeScript 理解程序设计的 SOLID 原则](https://mp.weixin.qq.com/s/1hKy1abZXvKcAcKtNPipuw)
