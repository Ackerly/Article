# JavaScript中的原型链
## JavaScript的诞生
JavaScript是一门脚本语言，是为了操作网页的，如果只将其作为简易的脚本语言，其实不需要有"继承"机制。但是Javascript里面都是对象，必须有一种机制，可以将对象之间关联起来。所以Brendan Eich最后还是设计了"继承"。  
但是他不打算引入"类"（class）的概念，因为一旦有了"类"，Javascript就是一种完整的面向对象编程语言了，这好像有点太正式了，Brendan Eich考虑到C++和Java语言都使用new命令生成实例。   
C++的写法是：
``` 
ClassName *object = new ClassName();
```
Java的写法是：
``` 
ClassName object = new ClassName();
```

因此，他把new命令引入了Javascript，用来从类（JavaScript中叫原型对象）生成一个实例对象。但是Javascript没有"类"，怎么来表示类（原型对象）呢？  
这时，他想到C++和Java使用new命令时，都会调用"类"的构造函数（constructor）。他就做了一个简化的设计，在Javascript语言中，new命令后面跟的不是类，而是构造函数。  
举例来说，现在有一个叫做Person的构造函数，表示人对象的原型（可以理解成java中的类）。  
``` 
function Person(name){
   this.name = name;
}
```
对这个构造函数使用new，就会生成一个人对象的实例  
``` 
var pA = new Person('老王');

alert(pA.name); // 老王
```
构造函数中的this关键字，它就代表了新创建的实例对象  

## prototype属性的由来
对于面向对象编程语言比如java或者c++来说，用构造函数生成实例对象是无法共享属性和方法，都有其独立的内存区域，互不影响  
比如，在Person对象的构造函数中，设置一个实例对象的共有属性race  
``` 
function Person(name){

    this.name = name;

     this.race = '汉族';

}
```
然后，生成两个实例对象：  
``` 
var pA = new Person('老王');
var pB = new Person('老张');
```
这两个对象的race属性是独立的，修改其中一个，不会影响到另一个  
``` 
pA.species = '苗族';
alert(pB.species); // 显示"汉族"，不受pA的影响
```
按前面new运算符所述，每一个实例对象，都有自己的属性和方法的副本。但此时如果我们想在同类但不同对象间共享数据（继承）怎么办呢，解决此问题的方法就是prototype  
考虑到共享数据的问题，Brendan Eich决定为JavaScript的构造函数设置一个prototype属性。  
这个属性包含一个对象（以下简称"prototype对象"），所有实例对象需要共享的属性和方法，都放在这个对象里面；那些不需要共享的属性和方法，就放在构造函数里面。这里prototype对象有点像C++基类  
实例对象一旦创建，将自动引用prototype对象的属性和方法。也就是说，实例对象的属性和方法，分成两种，一种是本地的，另一种是引用的  
还是以Person构造函数为例，现在用prototype属性进行改写：  
``` 
function Person(name){

  this.name = name;

}

Person.prototype = { race : '汉族' };


var pA = new Person('老王');

var pB = new Person('老张');


alert(pA.race); // 汉族

alert(pB.race); // 汉族
```
race属性放在prototype对象里，是两个实例对象共享的。只要修改了prototype对象，就会同时影响到两个实例对象  
``` 
Person.prototype.race = '苗族';
alert(pA.race); // 苗族
alert(pB.race); // 苗族
```
由于所有的实例对象共享同一个prototype对象，那么从外界看起来，而实例对象则好像"继承"了prototype对象一样  
## 重写prototype属性、方法
用过java、c++类语言的都知道，既然有继承，必然有重写。举例说明JavaScript的重写  
``` 
Person.prototype.hairColor = 'black';
Person.prototype.eat = function(){
    console.log('Person eat')
}
console.log(pA)
console.log(pB)
```
此时我们打印pA、pB，他们有了属性hairColor和eat方法；实例动态的获得了Person构造函数之后添加的属性、方法，这就是原型意义所在！可以动态获取，这样可以节省内存。  
如果pA将头发染成了黄色，那么hairColor会是什么呢？  
``` 
pA.hairColor = 'yellow'；
console.log(pA)
console.log(pB)
```
pA的hairColor = 'yellow'， 而pB的hairColor = 'black'；实例对象重写原型上继承的属性、方法，相当于 “属性覆盖、属性屏蔽” ，这一操作不会改变原型上的属性、方法，自然也不会改变由统一构造函数创建的其他实例，只有修改原型对象上的属性、方法，才能改变其他实例通过原型链获得的属性、方法。  
## 继承与原型链
JavaScript 中一切皆对象。每个实例对象都有一个私有属性，称之为 proto，指向它的构造函数的原型对象（prototype）。该原型对象也有一个自己的原型对象（proto），层层向上直到一个对象的原型对象为 null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。  
首先得记住并理解几个概念  
1. 属性__proto__是一个对象，它有两个属性，constructor和__proto__
2. 原型对象prototype有一个默认的constructor属性，用于记录实例是由哪个构造函数创建
3. 除了Object的原型对象（Object.prototype）的__proto__指向null，其他内置函数的原型对象和自定义构造函数的原型对象的 __proto__都指向Object.prototype

``` 
Object.prototype.__proto__ === null;
Array.prototype.__proto__ === Object.prototype;
```
## 创建对象的方法
**使用语法结构创建**  
``` 
var p = {name: "老三"};

// p 这个对象继承了 Object.prototype 上面的所有属性
// Object.prototype 的原型为 null
// 此对象原型链如下：
// p ---> Object.prototype ---> null

var arr = [1,2,3,4,5];
// 数组都继承于 Array.prototype
// 原型链如下：
// arr ---> Array.prototype ---> Object.prototype ---> null
```
**使用构造器创建**  
在 JavaScript 中，构造器其实就是一个普通的函数。当使用 new 操作符 来作用这个函数时，它就可以被称为构造方法（构造函数）。  
``` 
function Animal(name,age) {
  this.name = name;
  this.age = age;
}

Animal.prototype.eat = function(){
    console.log('Animal eat')
};

var a = new Animal("cat",1);
// a 是生成的对象，他的自身属性有 'name' 和 'age'。
// 在 a 被实例化时，a.__proto__ 指向了 Animal.prototype。
```
**使用 Object.create 创建**  
ECMAScript 5 中引入了一个新方法：Object.create()。可以调用这个方法来创建一个新对象。新对象的原型就是调用 create 方法时传入的第一个参数：
``` 
var a = {name: "老三"};
// a ---> Object.prototype ---> null
var b = Object.create(p);
// b ---> p ---> Object.prototype ---> null
console.log(b.name); // 老三 (继承而来)

var c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); // undefined，因为 d 没有继承 Object.prototype
```
**class关键字**  
ECMAScript6 引入了一套新的关键字用来实现 class。使用java、swift等面向对象的语言的开发人员会对这些结构感到熟悉，但它们是不同的。JavaScript 仍然基于原型。这些新的关键字包括 class, constructor，static，extends 和 super。  
``` 
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Person extends Animal {
  constructor(name,sex) {
    super(name);
    this.sex = sex;
    this.arrs = [1,2,3,4];
  }
  
}

var pC = new Person("老王","男");
```
注意：类的本质还是一个函数，类就是构造函数的另一种写法，JavaScript 中并没有一个真正的 class 原始类型， class 、extends 仅仅只是对原型对象运用语法糖，这样便于程序员理解。  
``` 
function Person(){}
console.log(typeof Person); //function

class Person extends Animal {
  constructor(name,sex) {
    super(name);
    this.sex = sex;
  }
  get mysex() {
    return this.sex;
  }
  set mysex(sex) {
    this.sex = sex;
  }
}

console.log(typeof Person); //function
```


原文:  
[JavaScript中的原型链](https://mp.weixin.qq.com/s/SoCVCPq1UraAES_iIarC4Q)
