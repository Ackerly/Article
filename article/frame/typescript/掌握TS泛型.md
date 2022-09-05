# 掌握TS泛型
## 为什么需要泛型
> 软件工程中，我们不仅要创建一致的定义良好的 API，同时也要考虑可重用性。 组件不仅能够支持当前的数据类型，同时也能支持未来的数据类型，这在创建大型系统时为你提供了十分灵活的功能。
> 在像 C# 和 Java 这样的语言中，可以使用泛型来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件。

举例：定义一个 print 函数，这个函数的功能是把传入的参数打印出来，再返回这个参数，传入参数的类型是 string，函数返回类型为 string。  
``` 
function print(arg:string):string {
    console.log(arg)
    return arg
}
```
现在需求变了，我还需要打印 number 类型，怎么办？可以使用联合类型来改造：  
``` 
function print(arg:string | number):string | number {
    console.log(arg)
    return arg
}
```
现在需求又变了，我还需要打印 string 数组、number 数组，甚至任何类型，怎么办？ 有个笨方法，支持多少类型就写多少联合类型。或者把参数类型改成 any。  
在 TS中还是尽量不要写 any,而且这也不是我们想要的结果，只能说传入的值是 any 类型，输出的值是 any 类型，传入和返回并不是统一的。这么写甚至还会出现bug  
``` 
function print(arg:any):any {
    console.log(arg)
    return arg
}
const res:string = print(123) 
```
定义 string 类型来接收 print 函数的返回值，返回的是个 number 类型，TS 并不会报错提示我们。 这个时候，泛型就出现了，它可以轻松解决输入输出要一致的问题。  
## 泛型基本使用
**处理函数参数**  
泛型的语法是 <> 里写类型参数，一般可以用 T 来表示。  
``` 
function print<T>(arg:T):T {
    console.log(arg)
    return arg
}
```
这样，就做到了输入和输出的类型统一，且可以输入输出任何类型。如果类型不统一，就会报错  
泛型中的 T 就像一个占位符、或者说一个变量，在使用的时候可以把定义的类型像参数一样传入，它可以原封不动地输出。  
> 泛型的写法对前端工程师来说是有些古怪，比如 <> T ，但记住就好，只要一看到 <>，就知道这是泛型。

使用的时候可以有两种方式指定类型:
- 定义要使用的类型
- TS 类型推断，自动推导出类型

``` 
print<string>('hello')  // 定义 T 为 string

print('hello')  // TS 类型推断，自动推导类型为 string
```
type 和 interface都可以定义函数类型，也用泛型来写一下，type 这么写：  
``` 
type Print = <T>(arg: T) => T
const printFn:Print = function print(arg) {
    console.log(arg)
    return arg
}
```
interface 这么写：  
``` 
function print<T>(arg:T):T {
    console.log(arg)
    return arg
}

interface Iprint<T> {
    (arg: T): T
}

const myPrinnt: Iprint<number> = print
```
**默认参数**  
给泛型加默认参数，可以这么写：  
``` 
function print<T>(arg:T):T {
    console.log(arg)
    return arg
}

interface Iprint<T = number> {
    (arg: T): T
}

const myPrinnt: Iprint = print
```
这样默认就是 number 类型了  
**处理多个函数参数**  
现在有这么一个函数，传入一个只有两项的元组，交换元组的第 0 项和第 1 项，返回这个元组  
``` 
function swap(tuple) {
    return [tuple[1], tuple[0]]
}
```
这么写就丧失了类型，用泛型来改造一下，用 T 代表第 0 项的类型，用 U 代表第 1 项的类型。  
``` 
function swap<T, U>(tuple: [T, U]): [U, T]{
    return [tuple[1], tuple[0]]
}
```
这样就可以实现了元组第 0 项和第 1 项类型的控制。传入的参数里，第 0 项为 string 类型，第 1 项为 number 类型。在交换函数的返回值里，第 0 项为 number 类型，第 1 项为 string 类型。第 0 项上全是 number 的方法。第 1 项上全是 string 的方法。  
**函数副作用操作**  
泛型不仅可以很方便地约束函数的参数类型，还可以用在函数执行副作用操作的时候。比如有一个通用的异步请求方法，想根据不同的 url 请求返回不同类型的数据。  
``` 
function request(url:string) {
    return fetch(url).then(res => res.json())
}
request('user/info').then(res =>{
    console.log(res)
})
```
这时候的返回结果 res 就是一个 any 类型，但我们希望调用 API 都清晰的知道返回类型是什么数据结构，就可以这么做：
```
interface UserInfo {
    name: string
    age: number
}

function request<T>(url:string): Promise<T> {
    return fetch(url).then(res => res.json())
}

request<UserInfo>('user/info').then(res =>{
    console.log(res)
})
```
这样就能很舒服地拿到接口返回的数据类型
## 约束泛型
假设现在有这么一个函数，打印传入参数的长度
``` 
function printLength<T>(arg: T): T {
    console.log(arg.length)
    return arg
}
```
因为不确定 T 是否有 length 属性，会报错。那么现在想约束这个泛型，一定要有 length 属性，怎么办？可以和 interface 结合，来约束类型。  
``` 
interface ILength {
    length: number
}

function printLength<T extends ILength>(arg: T): T {
    console.log(arg.length)
    return arg
}
```
<T extends ILength>让这个泛型继承接口 ILength，这样就能约束泛型。  
定义的变量一定要有 length 属性，比如下面的 str、arr 和 obj，才可以通过 TS 编译。  
``` 
const str = printLength('lin')
const arr = printLength([1,2,3])
const obj = printLength({ length: 10 })
```
例子也再次印证了 interface 的 duck typing。只要你有 length 属性，都符合约束，那就不管你是 str，arr 还是obj，都没问题。当然，我们定义一个不包含 length 属性的变量，比如数字，就会报错  
## 泛型的一些应用  
使用泛型，可以在定义函数、接口或类的时候，不预先指定具体类型，而是在使用的时候再指定类型。  
**泛型约束类**  
定义一个栈，有入栈和出栈两个方法，如果想入栈和出栈的元素类型统一，就可以这么写：  
``` 
class Stack<T> {
    private data: T[] = []
    push(item:T) {
        return this.data.push(item)
    }
    pop():T | undefined {
        return this.data.pop()
    }
}
```
在定义实例的时候写类型，比如，入栈和出栈都要是 number 类型，就这么写：  
``` 
const s1 = new Stack<number>()
```
入栈一个字符串就会报错  
这是非常灵活的，如果需求变了，入栈和出栈都要是 string 类型，在定义实例的时候改一下就好了：  
``` 
const s1 = new Stack<string>()
```
这样，入栈一个数字就会报错  
特别注意的是，泛型无法约束类的静态成员，给 pop 方法定义 static 关键字，就报错了。  
**泛型约束接口**  
使用泛型，也可以对 interface 进行改造，让 interface 更灵活  
``` 
interface IKeyValue<T, U> {
    key: T
    value: U
}

const k1:IKeyValue<number, string> = { key: 18, value: 'lin'}
const k2:IKeyValue<string, number> = { key: 'lin', value: 18}
```
**泛型定义数组**  
定义一个数组，之前是这么写的：  
``` 
const arr: number[] = [1,2,3]
```
现在这么写也可以：  
``` 
const arr: Array<number> = [1,2,3]
```
## 实战，泛型约束后端接口参数类型
来看一个泛型非常有助于项目开发的用法，约束后端接口参数类型。
``` 
import axios from 'axios'

interface API {
    '/book/detail': {
        id: number,
    },
    '/book/comment': {
        id: number
        comment: string
    }
    ...
}


function request<T extends keyof API>(url: T, obj: API[T]) {
    return axios.post(url, obj)
}

request('/book/comment', {
    id: 1,
    comment: '非常棒！'
})
```
这样在调用接口的时候就会有提醒，比如：路径写错了，参数类型传错了，参数传少了




原文:  
[](https://juejin.cn/post/7064351631072526350?utm_source=gold_browser_extension)