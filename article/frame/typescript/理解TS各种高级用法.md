# 理解TS各种高级用法
## 接口泛型位置 
``` 
// 定义一个泛型接口 IPerson表示一个类，它返回的实例对象取决于使用接口时传入的泛型T
interface IPerson<T> {
  // 因为我们还没有讲到unknown 所以暂时这里使用any 代替
  new(...args: unknown[]): T;
}

function getInstance<T>(Clazz: IPerson<T>) {
  return new Clazz();
}

// use it
class Person {}

// TS推断出函数返回值是person实例类型
const person = getInstance(Person);
```
定义接口 IPerson 时，这个接口定义了一个泛型参数 T 表示返回的实例类型。当使用时，需要在使用接口时声明该 T 类型，比如IPerson<T>。  
另外一个例子：  
``` 
// 声明一个接口IPerson代表函数
interface IPerson {
  // 此时注意泛型是在函数中参数 而非在IPerson接口中
  <T>(a: T): T;
}

// 函数接受泛型
const getPersonValue: IPerson = <T>(a: T): T => {
  return a;
};

// 相当于getPersonValue<number>(2)
getPersonValue(2)
```
泛型接口中泛型的位置是代表完全不同的含义：  
**当泛型出现在接口内部时，比如第二个例子中的 IPerson接口代表一个函数，接口本身并不具备任何泛型定义。而接口代表的函数则会接受一个泛型定义。换句话说接口本身不需要泛型，而在实现使用接口代表的函数类型时需要声明该函数接受一个泛型参数。**  
当我们希望实现一个数组的 forEach 方法时，尝试使用泛型来实现：  
``` 
// 定义callback遍历方法 两种方式 应该采用哪一种？
type Callback = <T>(item: T) => void
// 第二种声明方式
type Callback<T> = (item: T) => void;

const forEach = <T>(arr: T[], callback: Callback) => {
  for (let i = 0; i < arr.length - 1; i++) {
    callback(arr[i])
  }
};

forEach(['1', 2, 3, '4'], (item) => {});
```
forEach 中的 callback 类型定义应该采用第几种呢？答案是第二种方式type Callback<T> = (item: T) => void;。  
TS 是一种静态类型检测，并不会执行你的代码。  
``` 
// item的类型取决于调用函数时传入的类型参数
type Callback = <T>(item: T) => void;

const forEach = <T>(arr: T[], callback: Callback) => {
  for (let i = 0; i < arr.length - 1; i++) {
    // 这里调用callback时，ts并不会执行我们的代码。
    // 换句话说：它并不清楚arr[i]是什么类型
    callback(arr[i]);
  }
};

// 所以这里我们并不清楚 callback 定义中的T是什么类型，自然它的类型还是T
forEach(['1', 2, 3, '4'], <T>(item: T) => {});
```
采用第二种声明方式，在 forEach 内部的 callback 函数调用时，才会清楚函数传入的参数类型。  
``` 
// item 的类型取决于使用类型时传入的泛型参数
type Callback<T> = (item: T) => void;

// 在声明阶段就已经确定了 callback 接口中的泛型参数为外部传入的
const forEach = <T>(arr: T[], callback: Callback<T>) => {
  for (let i = 0; i < arr.length - 1; i++) {
    callback(arr[i]);
  }
};

// 自然，我们在调用forEach时显式声明泛型参数为 string | number 类型
// 所以根据forEach的函数类型定义时，
// 自然 callback 的 item 也会在定义时被推导为 T 也就是所谓的 string | number 类型
forEach<string | number>(['1', 2, 3, '4'], (item) => {});
```
在泛型接口中泛型的声明位置不同所产生的效果是完全不同的。  
## 泛型约束
``` 
// 定义方法获取传入参数的length属性
function getLength<T>(arg: T) {
  // throw error: arr上不存在length属性
  return arg.length;
}
```
这里，我们定义了一个 getLength 方法，希望函数获取传入参数的 length 属性。  
因为传入的参数是不固定的，有可能是 string 、 array 、 arguments 对象甚至一些我们自己定义的 { name:"19Qingfeng", length: 100 }，所以我们为函数增加泛型来为函数增加更加灵活的类型定义。  
随之而来的问题来了，那么此时我们在函数内部访问了 arg.length 属性。但是此时，arg 所代表的泛型可以是任意类型。  
比如我们可以传入一个 boolean ，那么此时函数中的泛型 T 代表 boolean 类型，访问 boolean.length ? 这显然是一个 bug  
那么如果解决这个问题呢，当然就提到了所谓的泛型约束 extends 关键字。  
如何使用:  
``` 
interface IHasLength {
  length: number;
}

// 利用 extends 关键字在声明泛型时约束泛型需要满足的条件
function getLength<T extends IHasLength>(arg: T) {
  // throw error: arr上不存在length属性
  return arg.length;
}

getLength([1, 2, 3]); // correct
getLength('123'); // correct
getLength({ name: '19Qingfeng', length: 100 }); // correct
// error 当传入true时，TS会进行自动类型推导 相当于 getLength<boolean>(true)
// 显然 boolean 类型上并不存在拥有 length 属性的约束，所以TS会提示语法错误
getLength(true); 
```
## keyof 关键字
keyof 关键字代表它接受一个对象类型作为参数，并返回该对象所有 key 值组成的联合类型。  
``` 
interface IProps {
  name: string;
  age: number;
  sex: string;
}

// Keys 类型为 'name' | 'age' | 'sex' 组成的联合类型
type Keys = keyof IProps
```
需要额外注意的一点是当 keyof any 时候我们会得到什么类型呢？  
any 可以代表任何类型。那么任何类型的 key 都可能为 string 、 number 或者 symbol 。所以自然 keyof any 为 string | number | symbol 的联合类型。  
实现一个函数,该函数希望接受两个参数，第一个参数为一个对象object，第二个参数为该对象的 key 。函数内部通过传入的 object 以及对应的 key 返回 object[key] 。    
``` 
// 函数接受两个泛型参数
// T 代表object的类型，同时T需要满足约束是一个对象
// K 代表第二个参数K的类型，同时K需要满足约束keyof T （keyof T 代表object中所有key组成的联合类型）
// 自然，我们在函数内部访问obj[key]就不会提示错误了
function getValueFromKey<T extends object, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
```
## is 关键字
is 关键字其实更多用在函数的返回值上，用来表示对于函数返回值的类型保护。
``` 
// 函数的返回值类型中 通过类型谓词 is 来保护返回值的类型
function isNumber(arg: any): arg is number {
  return typeof arg === 'number'
}

function getTypeByVal(val:any) {
  if (isNumber(val)) {
    // 此时由于isNumber函数返回值根据类型谓词的保护
    // 所以可以断定如果 isNumber 返回true 那么传入的参数 val 一定是 number 类型
    val.toFixed()
  }
}
```
## 分发 
``` 
type isString<T> = T extends string ? true : false;

// a 的类型为 true
let a: isString<'a'>

// b 的类型为 false
let b: isString<1>;
```
isString 类型内部通过 extends 关键字结合 ? 和 : 实现了所谓的 Conditional Types （条件类型）判断。  
这里的 T extends string 更像是一种判断泛型 T 是否满足 string 的判断，和之前所讲的泛型约束完全不是同一个意思  
使用 isString 时，你可以为它传入任意类型作为泛型参数的实现。但是 isString 类型内部会对于传入的泛型类型进行判断，如果 T 满足 string 的约束条件，那么返回类型 true，反过来则是 false 。  
需要注意的是条件类型 a extends b ? c : d 仅仅支持在 type 关键字中使用  
``` 
type GetSomeType<T extends string | number> = T extends string ? 'a' : 'b';

let someTypeOne: GetSomeType<string> // someTypeone 类型为 'a'

let someTypeTwo: GetSomeType<number> // someTypeone 类型为 'b'

let someTypeThree: GetSomeType<string | number>; // what ? 
```
这里我们定义了一个 GetSomeType 的类型，它接受一个泛型参数 T 。这个泛型参数 T 在传入时需要满足为 string 和 number 的联合类型的约束。  
someTypeThree 定义时的类型 GetSomeType<'string' | 1> 我们传入的泛型参数为联合类型时 'string' | 1 时，它会的到什么类型  
按照正常逻辑来思考。'string' | 1 一定是不满足 T extends string，因为一个 'string' | 1 的联合类型一定是无法和 string 类型做兼容的。  
按照之前的逻辑来梳理，理所应当 someTypeThree 的类型应该是 'b', 实际上推导成为了 'a' | 'b' 组成的联合类型。
在使用 GetSomeType 你可以传入n个类型组成的联合类型作为泛型参数，同理它会进行进入 GetSomeType 类型中进行 n 次分发判断，这就是分发
什么情况下会产生分发呢？满足分发需要一定的条件：  
- 分发一定是需要产生在 extends 产生的类型条件判断中，并且是前置类型
- 其次，分发一定是要满足联合类型，只有联合类型才会产生分发
- 分发一定要满足所谓的裸类型中才会产生效果

裸类型：
``` 
// 此时的T并不是一个单独的”裸类型“T 而是 [T]
type GetSomeType<T extends string | number | [string]> = [T] extends string[]
  ? 'a'
  : 'b';

// 即使我们修改了对应的类型判断，仍然不会产生所谓的分发效果。因为[T]并不是一个裸类型
// 只会产生一次判断  [string] | number extends string[]  ? 'a' : 'b'
// someTypeThree 仍然只有 'b' 类型 ，如果进行了分发的话那么应该是 'a' | 'b'
let someTypeThree: GetSomeType<[string] | number>;
```
在 TypeScript 内部拥有一个高级内置类型 Exclude 意为排除，它的用法如下：  
``` 
type TypeA = string | number | boolean | symbol;

// ExcludeSymbolType 类型为 string | number | boolean，排除了symbol类型
type ExcludeSymbolType = Exclude<TypeA, symbol>;
```
如何实现一个 Exclude 内置类型  
``` 
type TypeA = string | number | boolean | symbol;

type MyExclude<T, K> = T extends K ? never : T;

// ExcludeSymbolType 类型为 string | number | boolean，排除了symbol类型
type ExcludeSymbolType = MyExclude<TypeA, symbol | boolean>;
```
MyExclude 类型接受两个泛型参数，因为 T extends K ? never : T 中 T 满足裸类型并且在 extends 关键字前。  
传入的 TypeA 为联合类型，那么满足分发的所有条件。则会产生分发效果，也就是说会将联合类型 TypeA 中所有的单个类型依次进入 T extends K ? never : T; 去判断。  
当满足条件时，也就是 T extends symbol | boolean 时，此时会得到 never 。  
而如果不满足 T extends symbol | boolean 则会被记录，最终返回不满足 T extends symbol | boolean 的所有类型组成的联合类型，也就是所谓的 string | number 。  
## 循环
TypeScript 中同样存在对于类型的循环语法(Mapping Type)，可以通过 in 关键字配合联合类型来对于类型进行迭代。  
``` 
interface IProps {
  name: string;
  age: number;
  highSchool: string;
  university: string;
}

// IPropsKey类型为
// type IPropsKey = {
//  name: boolean;
//  age: boolean;
//  highSchool: boolean;
//  university: boolean;
//  }
type IPropsKey = { [K in keyof IProps]: boolean };
```
keyof IProps 我们在之前提到过它会返回 IProps 所有 key 组成的联合类型，也就是 'name' | 'age' | 'highSchool' | 'university' 。  
in 关键字的作用类似于 for 循环，它会循环 keyof IProps 这个联合类型中的每一项类型，同时在每一次循环中将对应的类型赋值给 K  
``` 
interface IInfo {
  name: string;
  age: number;
}

type MyPartial<T> = { [K in keyof T]?: T[K] };

type OptionalInfo = MyPartial<IInfo>;
```
通过 in 循环传入的 IInfo，同时构造了一个新的类型它的 key 和 IInfo 是一模一样的。  
无论是 TS 内置的 Partial 还是我们刚刚自己实现的 Partial ，它仅仅对象中一层的转化并不能递归处理。  
对于内层嵌套类型是无法深度进入类型中进行标记的  
实现深度可选:
``` 
interface IInfo {
  name: string;
  age: number;
  school: {
    middleSchool: string;
    highSchool: string;
    university: string;
  };
}

// 其实实现很简单，首先我们在构造新的类型value时
// 利用 extends 条件判断新的类型value是否为 object 
// 如果是 -> 那么我仍然利用 deepPartial<T[K]> 进行包裹递归可选处理
// 如果不是 -> 普通类型直接返回即可
type deepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? deepPartial<T[K]> : T[K];
};

type OptionalInfo = deepPartial<IInfo>;

let value: OptionalInfo = {
  name: '1',
  school: {
    middleSchool:'xian'
  },
};
```
## 逆变
通俗来说也就是多的可以赋值给少的，思考这样一段代码：  
``` 
let fn1!: (a: string, b: number) => void;
let fn2!: (a: string, b: number, c: boolean) => void;

fn1 = fn2; // TS Error: 不能将fn2的类型赋值给fn1
复制代码
```
将 fn2 赋值给 fn1 ，刚刚才提到类型兼容性的原因 TS 允许不同类型进行互相赋值（只需要父/子集关系），那么明明 fn2 的参数包括了所有的 fn1 为什么会报错？  
换一个角度来理解这个问题：  
针对于 fn1 声明时，函数类型需要接受两个参数，换句话说调用 fn1 时我需要支持两个参数的传入分别是 a:string和b:number。  
同理 fn2 函数定义时，定义了三个参数那么调用 fn2 时自然也需要传入三个参数。  
将 fn2 赋值给 fn1 ，如果赋值成功了，当调用 fn1 时，其实相当于调用 fn2 。  
但是，由于 fn1 的函数类型定义仅仅支持两个参数 a:string和b:number 即可。但是由于我们执行了 fn1 = fn2。  
调用 fn1 时，实际相当于调用了 fn2 函数。但是类型定义上来说 fn1 满足两个参数传入即可，而 fn2 是实打实的需要传入 3 个参数。  
那么此时，如果执行了 fn1 = fn2 当调用 fn1 时明显参数个数会不匹配（由于类型定义不一致）会缺少一个第三个参数，显然这是不安全的，自然也不是被 TS 允许的。  
反过来  
``` 
let fn1!: (a: string, b: number) => void;
let fn2!: (a: string, b: number, c: boolean) => void;

fn2 = fn1; // 正确，被允许
```
将 fn1 赋值给 fn2 。fn2 的类型定义需要支持三个参数的传入，但实际 fn2 内部指针已经被修改称为 fn1 的指针。  
fn1 在执行时仅仅需要两个参数 a: string, b: number，显然 fn2 的类型定义中是满足这个条件的（当然它还多传递了第三个参数 c:boolean，在 JS 中对于函数而言调用时的参数个数大于定义时的参数个数是被允许的）。  
上述函数的参数类型赋值就被称为逆变，参数少（父）的可以赋给参数多（子）的那一个。看起来和类型兼容性（多的可以赋给少的）相反，但是通过调用的角度来考虑的话恰恰满足多的可以赋给少的兼容性原则。  
另外一个复杂点的例子  
```` 
class Parent {}

// Son继承了Parent 并且比parent多了一个实例属性 name
class Son extends Parent {
  public name: string = '19Qingfeng';
}

// GrandSon继承了Son 在Son的基础上额外多了一个age属性
class Grandson extends Son {
  public age: number = 3;
}

// 分别创建父子实例
const son = new Son();

function someThing(cb: (param: Son) => any) {
  // do some someThing
  // 注意：这里调用函数的时候传入的实参是Son
  cb(Son);
}

someThing((param: Grandson) => param); // error
someThing((param: Parent) => param); // correct
````
这里定义了三个类，他们之间的关系分别是 Parent 是基类，Son 继承 Parent ，Grandson 继承 Son 。   
同时我们定义了一个函数，它接受一个 cb 回调参数作为参数，我们定义了这个回调函数的类型为接受一个 param 为 Son 实例类型的参数，此时我们不关心它的返回值给一个 any 即可。  
函数的参数的方式被称为逆变，所以当我们调用 someThing 时传递的 callback 需要赋给定义 something 函数中的 cb 。  
换句话说类型 (param: Grandson) => param 需要赋给 cb: (param: Son) => any，这显然是不被允许的。  
因为逆变的效果函数的参数只允许“从少的赋值给多的”，显然 Grandson 相较于 Son 来说多了一个 name 属性少，所以这是不被允许的。  
第二个someThing((param: Parent) => param);相当于函数参数重将 Parent 赋给 Son 将少的赋给多的满足逆变，所以是正确的。  
在定义 someThing 函数时，声明了这个函数接受一个 cb 的函数。这个函数接受一个类型为 Son 的参数。  
someThing 内部cb 函数声明时需要满足 Son 的参数，它会在 cb 函数调用时传入一个 Son 参数的实参。  
当传入 someThing((param: Parent) => param) 时，相当于在 something 函数内部调用 (param: Parent) => param 时会根据 someThing 中callback的定义传入一个 Son 。  
函数真实调用时期望得到是 Parent，但是实际得到了 Son 。Son 是 Parent 的子类涵盖所有 Parent 的公共属性方法，自然也是满足条件的。  
当我们使用someThing((param: Grandson) => param); ，由于 something 定义 cb 的类型传入 Son，但是真实调用 someThing 时，我们确需要一个 Grandson 类型参数的函数，这显然是不符合的。
## 协变
``` 
let fn1!: (a: string, b: number) => string;
let fn2!: (a: string, b: number) => string | number | boolean;

fn2 = fn1; // correct 
fn1 = fn2 // error: 不可以将 string|number|boolean 赋给 string 类型
```
函数类型赋值兼容时函数的返回值就是典型的协变场景，我们可以看到 fn1 函数返回值类型规定为 string，fn2 返回值类型规定为 string | number | boolean  
显然 string | number | boolean 是无法分配给 string 类型的，但是 string 类型是满足 string | number | boolean 其中之一，所以自然可以赋值给 string | number | boolean 组成的联合类型。  
将 fn1 赋给 fn2 ，fn1 要求返回值是 string ，而真实调用的 fn1=fn2 相当于调用了 fn2 自然 string | number | boolean 无法满足string类型的要求，所以 TS 会认为这是错误的。
## 待推断类型
infer 代表待推断类型，它的必须和 extends 条件约束类型一起使用。  
它表示我们可以在条件类型中推断一些暂时无法确定的类型，比如这样：  
``` 
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```
定义了一个 Flatten 类型，它接受一个传入的泛型 Type ，我们在类型定义内部对于传入的泛型 Type 进行了条件约束：  
- 如果 Type 满足 Array<infer Item>，那么此时返回 Item 类型。
- Type 不满足 Array<infer Item>类型，那么此时返回 Type 类型

一句话描述就是我们利用 infer 声明了一个数组类型，数组中值的类型我们并不清楚所以使用 infer 来进行推断数组中的值。  
比如：  
类型Flatten传入一个 string 类型，显然传入的 string 并不满足数组的约束。自然直接返回传入的 string 类型。  
Array<infer Item>代表的进行条件判断时要求前者（Type）必须是一个数组，但是数组中的类型我并不清楚（或者说可以是任意）  
使用 infer 关键字表示待推断的类型， infer 后紧跟着类型变量 Item 表示的就是待推断的数组元素类型。  
类型定义时并不能立即确定某些类型，而是在使用类型时来根据条件来推断对应的类型。之后，因为数组中的元素可能为 string 也可能为 number，自然在使用类型时 infer Item 会将待推断的 Item 推断为 string | number 联合类型。  
TS 中存在一个内置类型 Parameters ，它接受传入一个函数类型作为泛型参数并且会返回这个函数所有的参数类型组成的元祖。  
``` 
// 定义函数类型
interface IFn {
  (age: number, name: string): void;
}

// type FnParameters = [age: number, name: string]
type FnParameters = Parameters<IFn>;

let a: FnParameters = [25, '19Qingfeng'];
```
它的内部实现恰恰是利用 infer 来实现的
``` 
type MyParameters<T extends (...args: any) => any> = T extends (
  ...args: infer R
) => any
  ? R
  : never;
```
定义的 MyParameters 类型中接受一个泛型 T 当传入 T 时需要满足它为函数类型的约束。  
在 MyParameters 内部对于 传入的泛型参数进行了条件判断，如果满足条件也就是 T extends ( ...args: infer R ) => any，需要注意的是条件判断中函数的参数并不是在类型定义时就确认的，函数的参数需要根据传入的泛型来确认后赋给变量 R 所以使用了 infer R 来表示待推断的函数参数类型。  
此时会返回满足条件的函数推断参数组成的数组也就是 ...args 的类型 R ，否则则返回 never 。  
## unknown & any
TypeScript 中同样存在一个高级类型 unknown ，它可以代表任意类型的值，这一点和 any 是非常类型的。  
声明为 any 之后会跳过任何类型检查，比如这样：  
``` 
let myName: any;

myName = 1

// 这明显是一个bug
myName()
```
unknown 和 any 代表的含义完全是不一样的，虽然 unknown 可以和 any 一样代表任意类型的值，但是这并不代表它可以绕过 TS 的类型检查。  
``` 
let myName: unknown;

myName = 1

// ts error: unknown 无法被调用，这被认为是不安全的
myName()

// 使用typeof保护myName类型为function
if (typeof myName === 'function') {
  // 此时myName的类型从unknown变为function
  // 可以正常调用
  myName()
}
```
unknown 就代表一些并不会绕过类型检查但又暂时无法确定值的类型，我们在一些无法确定函数参数（返回值）类型中 unknown 使用的场景非常多。比如：  
``` 
// 在不确定函数参数的类型时
// 将函数的参数声明为unknown类型而非any
// TS同样会对于unknown进行类型检测，而any就不会
function resultValueBySome(val:unknown) {
  if (typeof val === 'string') {
    // 此时 val 是string类型
    // do someThing
  } else if (typeof val === 'number') {
    // 此时 val 是number类型
    // do someThing
  }
  // ...
}
``` 
unknown类型可以接收任意类型的值，但并不支持将unknown赋值给其他类型。  
any类型同样支持接收任意类型的值，同时赋值给其他任意类型（除了never）。  
any 和 unknown 都代表任意类型，但是 unknown 只能接收任意类型的值，而 any 除了可以接收任意类型的值，也可以赋值给任意类型（除了 never）。  
比如下面这样：  
``` 
let a!: any;
let b!: unknown;

// 任何类型值都可以赋给any、unknown
a = 1;
b = 1;

// callback函数接受一个类型为number的参数 
function callback(val: number): void {}

// 调用callback传入aaa（any）类型 correct
callback(a);

// 调用callback传入b（unknown）类型给 val（number）类型 error
// ts Error: 类型“unknown”的参数不能赋给类型“number”的参数
callback(b);
```

原文:
[如何进阶TypeScript功底？一文带你理解TS中各种高级语法](https://mp.weixin.qq.com/s/VUUoUkQNt_3g6YOWJtTDDg)
