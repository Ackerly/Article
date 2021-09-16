# 深入TypeScript中的子类型、逆变、协变
## 子类型
``` 
interface Animal {
  age: number
}

interface Dog extends Animal {
  bark(): void
}
```
Animal 是 Dog 的父类，Dog是Animal的子类型，子类型的属性比父类型更多，更具体。
- 类型系统中，属性更多的类型是子类型
- 集合论中，属性更少的集合是子集

## 可赋值性 assignable
assignable 是类型系统中很重要的一个概念，当你把一个变量赋值给另一个变量时，就要检查这两个变量的类型之间是否可以相互赋值。
``` 
let animal: Animal
let dog: Dog

animal = dog // ok
dog = animal // error! animal 实例上缺少属性 'bark'
```
animal 是一个「更宽泛」的类型，它的属性比较少，所以更「具体」的子类型是可以赋值给它的，adog 上拥有 animal 所拥有的一切类型，赋值给 animal 是不会出现类型安全问题的。  
反之，如果 dog = animal，那么后续使用者会期望 dog 上拥有 bark 属性，当他调用了 dog.bark() 就会引发运行时的崩溃。
从可赋值性角度来说，子类型是可以赋值给父类型的，也就是 父类型变量 = 子类型变量 是安全的，因为子类型上涵盖了父类型所拥有的的一切属性。
## 在函数中运用
``` 
function f(val: { a: number; b: number })
let val1 = { a: 1 }
let val2 = { a: 1, b: 2, c: 3 }
```
调用 f(val1) 是会报错的，因为缺少属性 b，而函数 f 中很可能去访问 b 属性并且做一些操作，比如 b.substr()，这就会导致崩溃。  
其实就相当于把函数定义中的形参 val 赋值成了 val1， 把父类型的变量赋值给子类型的变量，这是危险的。  
反之，调用 f(val2) 没有任何问题，因为 val2 的类型是 val类型的子类型，它拥有更多的属性，函数有可能使用的一切属性它都有。
声明 dispatch 类型：
``` 
interface Action {
  type: string
}

declare function dispatch<T extends Action>(action: T)
```
## 在联合类型中运用
'a' | 'b' | 'c' 是 'a' | 'b' 的父类型。因为前者比后者更「宽泛」，后者比前者更「具体」。
``` 
type Parent = 'a' | 'b' | 'c'
type Son = 'a' | 'b'

let parent: Parent
let son: Son

parent = son // ok
son = parent // error! parent 有可能是 'c'
```
这里 son 是可以安全的赋值给 parent 的，因为 son 的所有可能性都被 parent 涵盖了。而反之则不行，parent 太宽泛了，它有可能是 'c'，这是 Son 类型 hold 不住的。  
## 逆变和协变
>协变与逆变(covariance and contravariance)是在计算机科学中，描述具有父/子型别关系的多个型别通过型别构造器、构造出的多个复杂型别之间是否有父/子型别关系的用语。
``` 
let animals: Animal[]
let dogs: Dog[]

animals = dogs

animals[0].age // ok
```
转变成数组之后，对于父类型的变量，我们依然只会去 Dog 类型中一定有的那些属性。那么，对于 type MakeArray<T> = T[] 这个类型构造器来说，它就是 协变（Covariance） 的。  
**协变**  
``` 
let visitAnimal = (animal: Animal) => void;
let visitDog = (dog: Dog) => void;
```
visitAnimal = visitDog其实是不安全的，比如：
``` 
let visitAnimal = (animal: Animal) => {
  animal.age
}

let visitDog = (dog: Dog) => {
  dog.age
  dog.bark()
}
```
由于 visitDog 的参数期望的是一个更具体的带有 bark 属性的子类型，所以如果 visitAnimal = visitDog 后，我们可能会用一个不带 bark 属性的普通的 animal 类型来传给 visitDog。
这会造成运行时错误，animal.bark 根本不存在，去调用这个方法会引发崩溃。但是反过来，visitDog = visitAnimal 却是完全可行的。因为后续调用方会传入一个比 animal 属性更具体的 dog，函数体内部的一切访问都是安全的。  
在对 Animal 和 Dog 类型分别调用如下的类型构造器之后
``` 
type MakeFunction<T> = (arg: T) => void
```
父子类型关系逆转了，这就是 逆变（Contravariance）。

参考：  
[深入 TypeScript 中的子类型、逆变、协变，进阶 Vue3 源码前必须搞懂的](https://juejin.cn/post/6855517117778198542?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
