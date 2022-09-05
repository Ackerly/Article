# 16道TypeScript题目  
## 实现 Pick
从类型 T 中选择出属性 K，构造成一个新的类型。  
例如：  
``` 
interface Todo {
  title: string
  description: string
  completed: boolean
}
  
type TodoPreview = MyPick<Todo, 'title' | 'completed'>
  
const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
}
```
解答
``` 
type MyPick<T, K extends keyof T = keyof T> = {
  [P in K]: T[P]
}
```
思路  
**keyof取出key**  
MyPick<Todo, 'title' | 'completed'> 尖括号的右侧是'title' | 'completed'，它们是Todo类型的两个key。所以，需要用keyof T取出。keyof T 等于 'title' | 'completed' | 'description'。  
**extends对泛型进行约束**  
'title' | 'completed' 是 'title' | 'completed' | 'description'的子集（keyof T）。所以 'title' | 'completed' 等于 K extends keyof T。  
**in遍历枚举类型**  
[P in K]的意思是将K中的key遍历出来，赋予P。T[P]的意思是将T中对应的key === P的值取出来。所以，[P in K]: T[P]的含义就很明白了  
**为什么要加 = keyof T**  
如果以MyPick<T>中形式调用时，T如果没有默认值，会报错。所以，需要加上 = keyof T。

## Readonly
不要使用内置的Readonly<T>，自己实现一个。  
该 Readonly 会接收一个 泛型参数，并返回一个完全一样的类型，只是所有属性都会被 readonly 所修饰。  
也就是不可以再对该对象的属性赋值。  
例如：  
``` 
interface Todo {
  title: string
  description: string
}

const todo: MyReadonly<Todo> = {
  title: "Hey",
  description: "foobar"
}

todo.title = "Hello" // Error: cannot reassign a readonly property
todo.description = "barFoo" // Error: cannot reassign a readonly property
```
**解答**
``` 
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P]
}
```
使用readonly关键字声明属性是只读属性。使用keyof T取出泛型T中的所有key，再用in遍历。  
## 元组转换为对象
传入一个元组类型，将这个元组类型转换为对象类型，这个对象类型的键/值都是从元组中遍历出来。  
例如：
``` 
const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const

type result = TupleToObject<typeof tuple> // expected { tesla: 'tesla', 'model 3': 'model 3', 'model X': 'model X', 'model Y': 'model Y'}
```
**解答**  
``` 
type TupleToObject<T extends readonly any[]> = {
  [P in T[number]]: P
}
```
**思路**  
```
type Flatten<T> = T extends any[] ? T[number] : T;
 
// Extracts out the element type.
type Str = Flatten<string[]>;
     
`type Str = string`
 
// Leaves the type alone.
type Num = Flatten<number>;
     
`type Num = number`
```
泛型T是数组，数组以number为索引，所以T[number]对应数组中的每个值。使用in遍历T[number]，也即遍历泛型数组T中的每个值  
## 第一个元素First<T>
实现一个通用First<T>，它接受一个数组T并返回它的第一个元素的类型。  
例如：  
``` 
type arr1 = ['a', 'b', 'c']
type arr2 = [3, 2, 1]

type head1 = First<arr1> // expected to be 'a'
type head2 = First<arr2> // expected to be 3
```
**解法一**  
``` 
type First<T extends any[]> = T['length'] extends 0 ? never : T[0];
type First<T extends any[]> = T extends [] ? never : T[0]
```
判断数组长度，如果长度不是0，则代表0位有值，可以取0位。  
**解法二**  
``` 
type First<T extends any[]> = T[0] extends T[number] ? T[0] : never;
```
通过T[number]代表数组单个项的值，推断T[0] extends T[number]。 如果属于，则代表长度大于0，有值  
**解法三**  
``` 
type First<T extends any[]> = T extends [infer F] ? F : never
```
通过infer指代数组第一个项，如果T extends [infer F]推断是true，则得到第一个项type为F  
**解法四**  
``` 
type First<T> = T extends [infer P, ...infer Rest] ? P : never
```
使用扩展运算符，配合infer，可以得到第一个项的type  
## 获取元组长度
创建一个通用的Length，接受一个readonly的数组，返回这个数组的长度。  
例如：  
``` 
type tesla = ['tesla', 'model 3', 'model X', 'model Y']
type spaceX = ['FALCON 9', 'FALCON HEAVY', 'DRAGON', 'STARSHIP', 'HUMAN SPACEFLIGHT']

type teslaLength = Length<tesla> // expected 4
type spaceXLength = Length<spaceX> // expected 5
```
**解答**  
``` 
type Length<T extends readonly any[]> = T['length'];
```
数组的长度，用T['length']可以推导出来。加readonly的原因是，如果数组是用const声明，则必须是readonly  
## 实现 Exclude
实现内置的Exclude <T, U>类型，但不能直接使用它本身。  
**解答**  
``` 
type MyExclude<T, U> = T extends U ? never : T;
```
使用extends条件推断，如果为false，则是独立存在于T的集合  
## Awaited
假如我们有一个 Promise 对象，这个 Promise 对象会返回一个类型。在 TS 中，我们用 Promise<T> 中的 T 来描述这个 Promise 返回的类型。请你实现一个类型，可以获取这个类型。  
**解答**  
``` 
type MyAwaited<T extends Promise<any>> = T extends Promise<infer U> ?  U extends Promise<any> 
                                          ? MyAwaited<U> 
                                          : U 
                                        : never;
```
先用条件推断，泛型T是否是Promise返回，并用infer U指代返回值。U有两种情况：  
- 普通返回值类型
- Promise类型

如果U是Promise类型，则需要递归检查。对应的代码是：  
``` 
U extends Promise<any> ? MyAwaited<U> : U 
```
如果是普通返回值类型，则直接返回U  
**为什么要加extends Promise<any>**  
MyAwaited<T extends Promise<any>>的含义，是为了避免用户传入非Promise function。
如果用户违反规则，TypeScript会按报错处理  

## IF 
实现一个 IF 类型，它接收一个条件类型 C ，一个判断为真时的返回类型 T ，以及一个判断为假时的返回类型 F。 C 只能是 true 或者 false， T 和 F 可以是任意类型。  
举例:  
``` 
type A = If<true, 'a', 'b'>  // expected to be 'a'
type B = If<false, 'a', 'b'> // expected to be 'b'
```
**解答**  
``` 
type If<C extends boolean, T, F> = C extends false ? F : T;
```
题意要求，C必须是boolean类型，所以对非boolean类型的传值，应该按报错处理。使用extends进行条件推断，即可根据判断决定返回的值  
## Concat
在类型系统里实现 JavaScript 内置的 Array.concat 方法，这个类型接受两个参数，返回的新数组类型应该按照输入参数从左到右的顺序合并为一个新的数组。  
举例  
``` 
type Result = Concat<[1], [2]> // expected to be [1, 2]
```
**解答**  
``` 
type Concat<T extends any[], U extends any[] > = [...T, ...U];
```
先用extends any[]限制泛型T和U是数组类型。接着，就可以使用扩展运算符进行扩展数组。  

## Includes
在类型系统里实现 JavaScript 的 Array.includes 方法，这个类型接受两个参数，返回的类型要么是 true 要么是 false  
举例来说  
``` 
type isPillarMen = Includes<['Kars', 'Esidisi', 'Wamuu', 'Santana'], 'Dio'> 
```
**解答**  
``` 
type Equal<X, Y> = (<T>() => T extends X ? 1 : 2) extends (<T>() => T extends Y ? 1 : 2) ? true : false;

type Includes<T extends readonly any[], U> = T extends [infer K, ...infer R] ? 
                                                Equal<U, K> extends true ? 
                                                  true
                                                  :
                                                  Includes<R, U>
                                                : false
```
解题的重点有两个：  
- 判断两个类型相等，需要实现Equal
- 逐个拆解类型数组T，将单个项取出来比较

## Equal
<T>() => T extends X ? 1 : 2 和 (<T>() => T extends Y ? 1 : 2)： 取出比较参数的类型。  
如果X,Y不相等，则第1个表达式取到的数字，和第2个表达式取到的数字不一样。原因是T只能是一种类型。  
**逐个取出数组中的值**  
T extends [infer K, ...infer R]用于提取数组中的值，并使用Equal<U, K>进行比较。
如果不相等，则用Includes<R, U>递归处理  

## Push
在类型系统里实现通用的 Array.push  
举例  
``` 
type Result = Push<[1, 2], '3'> 
```
**解答**  
``` 
type Push<T extends any[], U> = [...T, U];
```
使用extends any []限制泛型T为数组类型，然后，使用扩展运算符展开T,进行合并。  
## Pop
实现一个通用Pop<T>，它接受一个数组T并返回一个没有最后一个元素的数组  
例如  
``` 
type arr1 = ['a', 'b', 'c', 'd']
type arr2 = [3, 2, 1]

type re1 = Pop<arr1> // expected to be ['a', 'b', 'c']
type re2 = Pop<arr2> // expected to be [3, 2]
```
**解答**  
``` 
type Pop<T extends any[]> = T extends [...infer K, infer U] ? K : never;
```
使用infer进行指代，配合解构
## Unshift
实现类型版本的 Array.unshift  
举例  
``` 
type Result = Unshift<[1, 2], 0> // [0, 1, 2,]
```
**解答**  
``` 
type Unshift<T extends any[], U> = [U, ...T];
```
使用extends any []限制泛型T为数组类型，然后，使用扩展运算符展开T,进行合并。  
## 实现内置的 Parameters<T> 类型
实现内置的 Parameters<T> 类型，而不是直接使用它  
解答  
``` 
type MyParameters<T extends (...args: any[]) => any> = T extends (...args: infer U) => any ? U : never;
```
使用infer U指代参数列表，就可以正确推导出类型  
## 获取函数返回类型
不使用 ReturnType 实现 TypeScript 的 ReturnType<T> 泛型。
例如：
``` 
const fn = (v: boolean) => {
  if (v)
    return 1
  else
    return 2
}

type a = MyReturnType<typeof fn> // 应推导出 "1 | 2"
```
**解答**  
``` 
type MyReturnType<T> = T extends (...args: any[]) => infer U ? U : never;
```
使用infer U指代返回值类型即可。  
## 计算字符串的长度
计算字符串的长度，类似于 String#length  
**解答**  
``` 
type LengthOfString<S extends string, T extends any[] = []> = S extends `${infer L}${infer R}` ? 
                                                                LengthOfString<R, [...T, L]> 
                                                                : 
                                                                T['length'];
```
解题的关键点有两个：  
- 增加参数T，默认是空数组，用于存放读取的字符，方便使用数组的length属性，得到长度
- 使用递归，逐个拆解字符串

``` 
S extends `${infer L}${infer R}`
```
对字符串进行拆解  
如果当前的字符串已拆解完，则读取存放数组T的长度。  
如果没有拆解完，则递归调用LengthOfString<R, [...T, L]> ，并将取出的字符，放入T中。  
原文: 
[你以为学会TypeScript了？先看看这16道题能做对多少先](https://juejin.cn/post/7110232056826691591)
