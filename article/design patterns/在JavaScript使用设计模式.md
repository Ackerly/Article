# 在JavaScript中使用设计模式
设计模式是可重用的、高级的、面对对象的解决方案，可以使用这些解决方案来解决软件设计中日常问题的类似实例。  
在JavaScript中，设计模式在创建应用时可能是有用的。设计模式在开发社区是公认的直接解决方案。  
本文将讨论使用设计模式的好处，介绍JavaScript中的一些常用设计模式。  

## 使用设计模式的好处
**无需从零开始**
我们可以不费劲的在应用程序中快速实现经验证的设计模式。它们由有经验的开发者优化过。  
**可复用的**  
我们可以多次使用相同的设计模式来解决类似的问题。它们不仅仅是解决一个问题的办法，我们可以实施它们来解决许多问题。  
**减少代码体积**  
设计模式在适当应用优化，降低项目总体复杂行并提升了性能。  
**无需重构代码**  
如果我们遵循标准的设计模式，那么代码对其他开发人员来说是浅显易懂对。这使得每个人在同一页面上进行解释和讨论变得更加容易。  

## 设计模式对类型
我们可以在JavaScript应用程序中使用各种设计模式。让我们来看看三组重要对设计模式。  
**创建型设计模式**  
通过处理对象创建过程来解决问题。示例：原型模式、单例模式、构造函数模式。  
**结构型设计模式**  
通过确定实现对象之间关系对方法来解决问题。示例：适配器模式、代理模式  
**行为型设计模式**  
通过处理对象之间对通信来解决问题。示例：观察者模式、模板模式  

## JavaScript中对设计模式
JavaScript中流行对设计模式  

**构造函数模式**  
构造函数模式属于创建型设计模式。它允许我们用特定对方法和属性初始化新对象。在创建同一个模板对多个实例时，可以使用构造函数模式。这些实例可能不同，但他猛可以共享它们但方法。  
下面代码展示构造函数设计模式的一个简单示例  
``` 
// Creating constructor
function Tree(name, height, scientificname) {
  this.name = name;
  this.height = height;
  this.scientificname = scientificname;
  // Declaring a method
  this.getData = function() {
    return this.name + ' has a height of ' + this.height;
  };
}
// Creating new instance
const Coconut = new Tree('coconut', '30m', 'cocos nucifera');
console.log(Coconut.getData()); // coconut has a height of 30m
```
这里是树的构造函数。它包含三个属性和一个方法，分别命名为name、height、scientificname和getData。之后，我们从它示例话一个新对象椰子。这个对象包含构造函数的属性和方法。我们可以通过对象传递相关数据，调用函数得到高度为30m的椰子的输出。  

**工厂模式**  
工厂模式是另外一个创建型设计模式，但构造函数接受各种参数以返回所需但对象。这种模式对于需要处理具有相似特征不同对象组的情况非常理想。
例如，以食品制造厂为例。如果你需要蛋糕他们就给你蛋糕，如果你想要面包，他们就给你面包。这就是这个模式的基本思想。让我们看看如何在代码库中实现这个  
``` 
function foodFactory() {
  this.createFood = function(foodType) {
    let food;
switch(foodType) {
      case 'cake':
        food = new Cake();
        break;
      default:
        food = new Bread();
        break;
    }
return food;
  }
}
const Cake = function() {
  this.givesCustomer = () => {
    console.log('This is a cake!');
  }
}
const factory = new foodFactory();
const cake = factory.createFood('cake');
cake.givesCustomer();
```
我们有一个交foodFactory的工厂。我们有createFood函数，它接受一个刹那火速foodType。我们可以根据这个参数实例化对象。例如，如果我们往createFood函数传递一个cake参数，我们会收到相应的输出，所以，这个设计模式帮我们示例话具有相同特征的不同对象。  

**模块设计模式**  
模块设计模式属于结构性设计模式。这个设计模式类似封装。它允许你在不影响其他组件的情况下更改和更新特定的代码段。它是自包含的，这个模块有助于保持代码的干净和分离。  
由于JavaScript没有像public或private这样的访问说明符，可以在必要时使用这个模块模式来实现封装。我们使用闭包和立即调用的函数表达式来实现它。  
我们可以使用这种设计模式来模拟封装，并从全局作用域隐藏某些组件。  
基于这种设计模式狗年实际代码的示例：  
``` 
function foodContainter () {
    const container = \[\];
    
    function addFood (name) {
      container.push(name);
    }
    
    function getAllFood() {
      return container;
    }
    
    function removeFood(name) {
      const index = container.indexOf(name);
      if(index < 1) {
        throw new Error('This food is unavailable');
      }
      container.splice(index, 1)
    }
    
    return {
      add: addFood,
      get: getAllFood,
      remove: removeFood
    }
}
    
const container = FoodContainter();
container.add('Cake');
container.add('Bread');
    
console.log(container.get()) // Gives both cake and bread
container.remove('Bread') // Removes bread
```
在大妈的开头，有一个数组容器来包含我们输入的所有元素。每次调用addFood()方法时，都会执行一个push操作，一以便像数组中输入一个新元素。然后getAllFood函数返回数组中所有可用得到元素。removeFood()方法检查输入的项在数组中是否可用，如果存在则将其删除。否则它抛出一个错误：此食物不可用。  
实现之后，我们可以创建新对象并使用前面列出的三个方法。我们在foodContainer中所做的任何更改都不会影响任何其他组件。  

**单例模式**  
单例模式限制对象示例话一次以上。如果类中不存在实例，则从此类中示例化一个实例。  
我们可以通过使用一个方法创建一个类来实现这一点，该方法仅在同一类中没有其他现有实例时才创建一个实例。如果有一个当前对象，它将引用一个已经创建的对象，而不是一个新对象。  
单例模式有助于减少空间污染，它还最大限度减少对全局变量对需求。我们可以使用这种模式从一个地方协调系统范围对操作。  
例如：一个数据库连接池，用于管理对整个应用程序共享对资源对访问：  
``` 
var firstSingleton = (function () {
    var instance;
function createInstance() {
        var object = new Object("First instance");
        return object;
    }
return {
        getInstance: function () {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    };
})();
function execute() {
var firstinstance = firstSingleton.getInstance();
var secondinstance = firstSingleton.getInstance();
console.log("Is it the same instance? " + (instance1 === instance2));
}
```
createInstance()方法创建一个新对象。getInstance()函数充当单例对把关 人，检查对象对可用型，以创建一个新对象或引用一个已经存在对对象  
getInstance()方法对一部分表示另一种设计模式，即延迟加载。这个新对设计模式检查现有实例对可用性，如果实例不存在，则创建一个实例。只有在需要时才创建实例，有助于节省内存和CPU  
在execute()方法中，创建两个实例并比较它们，以检查这两个实例是否相同。我们将得到输出"Is it the same instance? true"，因为任何其他实例创建都会返回对第一个实例本身对引用。  

**原型模式**  
原型模式将现有对象属性克隆为新对象。这种设计模式对基本概念是原型继承。它是关于在实例化新对象时创建一个充当原型对对象，我们正使用这种对象，创建这种JavaScript工具  
JavaScript不支持类对概念。因此我们可以使用这个原型模式在JavaScript应用中实现继承。  
以公共汽车为例:  
``` 
const bus = {
  wheels: 4,
  start() {
    return 'started';
  },
  stop() {
    return 'stopped';
  },
};
// create object
const myBus = Object.create(bus, { model: { value: 'Single deck' } });
console.log(myBus.\_\_proto\_\_ === bus); // true
```
我们创建了一个名为bus对原型，它有一个属性和两个名为wheels、start()和stopped()的方法。我们从它创建一个新对象myBus并给这个对象一个额外的属性model。但新对象包含了原型中但所有属性和方法  
在最后一行代码中，我们可以证明创建的对象myBus的原型与原始原型bus是相同的  



参考:  
[Using Design Patterns in JavaScript —The Ultimate Guide](https://www.syncfusion.com/blogs/post/using-design-patterns-in-javascript-the-ultimate-guide.aspx)
