# JavaScript中 4 种创建枚举方式
## 基于普通对象的枚举
枚举是一种数据结构，它定义了一组有限的命名常量。每个常量可以通过其名称访问。  
定义一个T-shirt的size为：Small、Medium、Large。  
在JavaScript中创建枚举的一种简单的方式，是使用普通JavaScript对象  
``` 
const Sizes = {
  Small: 'small',
  Medium: 'medium',
  Large: 'large',
}
const mySize = Sizes.Medium
console.log(mySize === Sizes.Medium) // logs true
```
Sizes是一个基于普通JavaScript对象的枚举，它有3个命名的常量： Sizes.Small、 Sizes.Medium 、Sizes.Large。  
Sizes 同时也是一个字符串枚举，命名的常量的值是字符串： 'small', 'medium' 、'large'。  
要访问命名的常量值，请使用属性访问器。例如: Sizes.Medium的值是'medium'。  
枚举的可读性更强，更明确，并消除了对魔法字符串或数字的使用。  
**优点和缺点**  
这种使用普通对象的枚举方法非常简单明了，只要定义一个带有键和值的对象，就可以创建一个枚举  
在大型项目中，有人可能会无意间修改枚举对象，这将影响应用程序的运行时。由于枚举对象是可变的，因此开发人员无法将其设置为不可变。这是普通对象枚举的主要缺点。  
``` 
const Sizes = {
  Small: 'small',
  Medium: 'medium',
  Large: 'large',
}
const size1 = Sizes.Medium
const size2 = Sizes.Medium = 'foo' // Changed!
console.log(size1 === Sizes.Medium) // logs false
```
在上述代码中Sizes.Medium 枚举值被意外的修改了，size1在初始化时为Sizes.Medium, 不再和Sizes.Medium 相等！普通对象的实现无法防止这种意外更改。  
## 枚举值类型
除了字符串类型外，枚举的值也可以是number类型：
``` 
const Sizes = {
  Small: 0,
  Medium: 1,
  Large: 2
}
const mySize = Sizes.Medium
console.log(mySize === Sizes.Medium) // logs true
```
上例中的Sizes枚举是一个数字枚举，因为其值是数字： 0、1、2。  
``` 
const Sizes = {
  Small: Symbol('small'),
  Medium: Symbol('medium'),
  Large: Symbol('large')
}
const mySize = Sizes.Medium
console.log(mySize === Sizes.Medium) // logs true
```
使用符号的好处是，每一个符号都是唯一的，这意味着您必须始终使用枚举本身来比较枚举：  
``` 
const Sizes = {
  Small: Symbol('small'),
  Medium: Symbol('medium'),
  Large: Symbol('large')
}
const mySize = Sizes.Medium
console.log(mySize === Sizes.Medium)     // logs true
console.log(mySize === Symbol('medium')) // logs false
```
使用符号枚举的缺点是 JSON.stringify() 将符号序列化为 null、undefined，或者跳过包含符号值的属性，这将导致在需要将枚举转换为字符串形式的情况下产生问题。因此，符号枚举的使用可能需要在其他位置进行修改以支持 JSON.stringify()。  
``` 
const Sizes = {
  Small: Symbol('small'),
  Medium: Symbol('medium'),
  Large: Symbol('large')
}
const str1 = JSON.stringify(Sizes.Small)
console.log(str1) // logs undefined
const str2 = JSON.stringify([Sizes.Small])
console.log(str2) // logs '[null]'
const str3 = JSON.stringify({ size: Sizes.Small })
console.log(str3) // logs '{}'
```
如果你可以自由选择枚举值的类型，就选择字符串吧。字符串比数字和符号更容易进行调试。  
## 基于Object.freeze()的枚举
可以保护枚举对象免受修改的一种好方法是将其冻结。当对象被冻结时，您无法修改或添加新属性到该对象中。换句话说，该对象变为只读。  
在JavaScript中，Object.freeze() 函数冻结一个对象。让我们冻结Sizes枚举：  
``` 
const Sizes = Object.freeze({
  Small: 'small',
  Medium: 'medium',
  Large: 'large',
})
const mySize = Sizes.Medium
console.log(mySize === Sizes.Medium) // logs true
```
const Sizes = Object.freeze({ ... }) 创建一个被冻结的对象. 即使被冻结，也可以自由的访问这些枚举值：const mySize = Sizes.Medium  

**优点和缺点**  
如果一个枚举属性被意外的修改，JavaScript会抛出一个错误（在严格模式下）  
``` 
const Sizes = Object.freeze({
  Small: 'Small',
  Medium: 'Medium',
  Large: 'Large',
})
const size1 = Sizes.Medium
const size2 = Sizes.Medium = 'foo' // throws TypeError
```
语句 const size2 = Sizes.Medium = 'foo' 对 Sizes.Medium 属性进行了意外的赋值操作。因为Sizes 是一个被冻结的对象，JavaScript（在严格模式下）抛出一个错误：  
``` 
TypeError: Cannot assign to read only property 'Medium' of object <Object>
```
冻结的对象枚举被保护起来，这可以防止无意中修改枚举对象。  
还有一个问题。如果无意间拼写错了枚举常量，结果会变成undefined:  
``` 
const Sizes = Object.freeze({
  Small: 'small',
  Medium: 'medium',
  Large: 'large',
})
console.log(Sizes.Med1um) // logs undefined
```
在上面的示例中Sizes.Med1um表达式（Med1um是Medium的错误拼写）的结果将会是undefined。  

## 基于代理的枚举
代理枚举是一种有趣的实现方式，也是我最喜欢的一种方式。一个代理对象是一个特殊的对象，它包装另一个对象并修改对原始对象的操作的行为。代理不会改变原始对象的结构。  
枚举代理拦截枚举对象的读取和写入操作，并且：  
- 当访问不存在的枚举值时抛出错误
- 当更改枚举对象属性时抛出错误

下面是一个工厂函数的实现，该函数接受一个普通的枚举对象，并返回一个代理对象：  
``` 
// enum.js
export function Enum(baseEnum) {
  return new Proxy(baseEnum, {
    get(target, name) {
      if (!baseEnum.hasOwnProperty(name)) {
        throw new Error(`"${name}" value does not exist in the enum`)
      }
      return baseEnum[name]
    },
    set(target, name, value) {
      throw new Error('Cannot add a new value to the enum')
    }
  })
}
```
代理的get()方法拦截读操作，并在属性名称不存在时抛出错误。set()方法拦截写操作并抛出错误。它旨在保护枚举对象免受写操作的影响。  
将Sizes对象枚举包装到代理中：
``` 
import { Enum } from './enum'
const Sizes = Enum({
  Small: 'small',
  Medium: 'medium',
  Large: 'large',
})
const mySize = Sizes.Medium
console.log(mySize === Sizes.Medium) // logs true
```
代理枚举的使用方式和普通对象枚举完全相同  
**优点和缺点**  
使用基于代理的枚举，可以获得一种既灵活又安全的枚举实现方式。代理枚举不仅可以捕获枚举属性的更改，还可以防止访问不存在的枚举常量。  
``` 
import { Enum } from './enum'
const Sizes = Enum({
  Small: 'small',
  Medium: 'medium',
  Large: 'large',
})
const size1 = Sizes.Med1um         // throws Error: non-existing constant
const size2 = Sizes.Medium = 'foo' // throws Error: changing the enum
```
因为枚举中没有定义名为'Med1um'的常量，所以 Sizes.Med1um 会引发错误。由于枚举属性已被更改，因此 Sizes.Medium = 'foo' 会引发错误。  
代理枚举的缺点是始终需要导入 Enum 工厂函数并将枚举对象包装在其中。与使用其他实现方式相比，代理枚举可能会带来一些性能损失。代理枚举涉及使用 JavaScript 的代理特性，这可能会使枚举对象的访问速度稍慢一些。因此，在实现枚举时，应该考虑的代码的性能需求。  
## 基于类的枚举
使用 JavaScript 类创建枚举, 基于类的枚举包含一组 静态字段，每个静态字段都代表一个命名的枚举常量。每个枚举常量的值本身就是该类的一个实例。  
使用 Sizes 类实现sizes枚举：  
``` 
class Sizes {
  static Small = new Sizes('small')
  static Medium = new Sizes('medium')
  static Large = new Sizes('large')
  #value
  constructor(value) {
    this.#value = value
  }
  toString() {
    return this.#value
  }
}
const mySize = Sizes.Small
console.log(mySize === Sizes.Small)  // logs true
console.log(mySize instanceof Sizes) // logs true
```
Sizes 是代表枚举的类。枚举常量是类上的静态字段，例如 static Small = new Season('small')。 Sizes 类的每个实例都有一个私有字段 #value，它代表枚举的原始值。由于 Sizes 类的构造函数是私有的，所以不能创建新的枚举实例。这就确保了枚举值的不可变性。  
基于类的枚举的一个很好的优点是，在运行时能够使用instanceof操作来确定值是否为枚举。例如，mySize instanceof Sizes评估为true，因为mySize是一个枚举值。  
在基于类的枚举中，比较是基于实例的（相对于普通、冻结或代理枚举而言，基于实例的比较更为原始）：  
``` 
class Sizes {
  static Small = new Sizes('small')
  static Medium = new Sizes('medium')
  static Large = new Sizes('large')
  #value
  constructor(value) {
    this.#value = value
  }
  toString() {
    return this.#value
  }
}
const mySize = Sizes.Small
console.log(mySize === new Sizes('small')) // logs false
```
mySize（它的值为 Sizes.Small）并不等于 new Sizes('small')。即使 Sizes.Small 和 new Sizes('small') 拥有相同的 #value值，但是它们是不同的对象实例。  
**优点和缺点**  
基于类的枚举无法防止覆盖或访问不存在的枚举命名常量。  
``` 
class Sizes {
  static Small = new Season('small')
  static Medium = new Season('medium')
  static Large = new Season('large')
  #value
  constructor(value) {
    this.#value = value
  }
  toString() {
    return this.#value
  }
}
const size1 = Sizes.medium         // a non-existing enum value can be accessed
const size2 = Sizes.Medium = 'foo' // enum value can be overwritten accidentally
```


原文:  
[在JavaScript中 4 种创建枚举方式](https://mp.weixin.qq.com/s/_5SYPbTWRclkUkyucZaJ8w)
