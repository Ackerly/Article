# never 和 unknown 的优雅之道
## TypeScript 中的 top type、bottom type 
在类型系统设计中，有两种特别的类型：  
- Top type：被称为通用父类型，也就是能够包含所有值的类型
- Bottom type：代表没有值的类型，它也被称为零或空类型，是所有类型的子类型

### unknown 和 any
**unknown —— 代表万物**  
阅读代码时，很少看到 unknown 类型的出现。这并不意味着它不重要，相反，它是安全版本的 any 类型。  
它和 any 的区别很简单，参考下面的例子  
``` 
function format1(value: any) {
     value.toFixed(2); // 不飘红，想干什么干什么，very dangerous
 }

 function format2(value: unknown) {
     value.toFixed(2); // 代码会飘红，阻止你这么做

     // 你需要收窄类型范围，例如：

     // 1、类型断言 —— 不飘红，但执行时可能错误
     (value as Number).toFixed(2);

     // 2、类型守卫 —— 不飘红，且确保正常执行
     if (typeof value === 'number') {
         // 推断出类型: number
         value.toFixed(2);
     }

     // 3、类型断言函数，抛出错误 —— 不飘红，且确保正常执行
     assertIsNumber(value);
     value.toFixed(2);
 }


 /** 类型断言函数，抛出错误 */
 function assertIsNumber(arg: unknown): asserts arg is Number {
     if (!(arg instanceof Number)) {
         thrownewTypeError('Not a Number: ' + arg);
     }
 }
```
使用 any 好比鬼屋探险，代码执行的时候处处见鬼。而 unknown 结合类型守卫等方式，可以确保上游数据结构不确定时，也能让代码正常执行。  

**any —— 一丝不挂**  
用到 any，就意味着放弃类型检查了，因为它不是用来描述具体类型的。  
在使用它之前，需要想两件事：
- 能否使用更具体的类型
- 能否使用 unknown 代替

现有的一些类型设计用到了 any，其实不够准确。这里举两个例子：  
_String()_  
String() 能够接受任何参数，转化为字符串。  
结合上文介绍的 unknown 类型，其实这里的参数也可以设计成 unknown，但内部实现就需要多设计些类型守卫了。但 unknown 类型是后面才出现的，所以一开始的设计还是采用了 any，也就是我们现在看到的：  
``` 
/**
  * typescript/lib/lib.es5.d.ts
  */
 interface StringConstructor {
     new(value?: any): String;
     (value?: any): string;
     readonly prototype: String;
     fromCharCode(...codes: number[]): string;
 }
```
_JSON.parse()_  
``` 
exportfunction deleteCommentFromComments<T>(comments: GenericsComment<T>[], comment: GenericsComment<T>) {
   // 深拷贝
   const list: GenericsComment<T>[] = JSON.parse(JSON.stringify(comments));

   // 找到对应的评论下标
   const targetIndex = list.findIndex((item) => {
     if (item.comment_id === comment.comment_id) {
       returntrue;
     }
     returnfalse;
   });

   if (targetIndex !== -1) {
     // 剔除对应的评论
     list.splice(targetIndex, 1);
   }

   return list;
 }
```
JSON.parse () 的输出是随着输入动态改变的（甚至有可能抛出 Error），它的函数签名被设计成了  
``` 
interface JSON {
     parse(text: string, reviver?: (this: any, key: string, value: any) =>any): any;
     ...
 }
```
不过原因和上面一样，JSON.parse() 的函数签名被添加到 TypeScript 系统之前，unknown 类型还没出现，否则它的返回类型应该是 unknown  
_never_  
never 类型表示的是空类型，也就是值永不存在的类型  
值会永不存在的两种情况：  
- 如果一个函数执行时抛出了异常，那么这个函数永远不存在返回值（因为抛出异常会直接中断程序运行，这使得程序运行不到返回值那一步，即具有不可达的终点，也就永不存在返回了）
- 函数中执行无限循环的代码（死循环），使得程序永远无法运行到函数返回值那一步，永不存在返回

``` 
// 异常
 function err(msg: string): never { // OK
   throw new Error(msg);
 }

 // 死循环
 function loopForever(): never { // OK
   while (true) {};
 }
```
_唯一的 bottom type_  
never 是 typescript 的唯一一个 bottom type，它能够表示任何类型的子类型，所以能够赋值给任何类型：  
``` 
let err: never;
 let num: number = 4;

 num = err; // OK
```
可以使用集合来理解 never，unknown 是全集，never 是最小单元（空集），任意类型都包含了 never。  
**null/undefined 和 never**  
null 和 undefined 好像也可以表示任何类型的子类型，为啥不是 bottom type。非也，never 特殊就特殊在，除了自身以外，没有任何类型是它的子类型，或者说可以赋值给它。它才是人下人（狗头），可以用下面的例子对比看看：  
``` 
// null 和 undefined，可以被 never 赋值
 declare const n: never;

 let a: null = n; // 正确
 let b: undefined = n; // 正确

 // never 是 bottom type，除了自己以外没有任何类型可以赋值给它
 let ne: never;

 ne = null; // 错误
 ne = undefined; // 错误

 declare const an: any;
 ne = an; // 错误，any 也不可以

 declareconst nev: never;
 ne = nev; // 正确，只有 never 可以赋值给 never
```
_为什么说 any 不是严格的 bottom type_  
除了 never 自身，没有任何类型能赋值给 never。any 是否满足这个特性呢？显然不能，举个很简单的例子：  
``` 
const a = 'anything';

 const b: any = a; // 能够赋值
 const c: never = a; // 报错，不能赋值
```
在一个类型系统中，bottom type 是独一无二的，它唯一地描述了函数无返回的情况。所以，有了 never 之后，any 这种脱离了类型检查的异端肯定称不上是 bottom type。  

**never 的妙用**  
never 有以下的使用场景：  
- Unreachable code 检查：标记不可达代码，获得编译提示
- 类型运算：作为类型运算中的最小因子
- Exhaustive Check：为复合类型创造编译提示
- ...

关于 never 的用途，知乎上有个很好的讨论。不可否认的是，never 这个东西很奇妙，从集合论的角度，它是一个空集合，因此它可以通过空集合的一些特性，为我们的类型运算工作带来很大便利。接下来来具体讲讲各个使用场景：  
_Unreachable code 检查_  
一个萌新写出了下面这行代码：  
``` 
process.exit(0);
 console.log("hello world") // Unreachable code detected.ts(7027)
```
当然这时候如果你使用了 ts，它会给你一个编译器提示：  
``` 
Error: Unreachable code detected.ts(7027)
```
因为 process.exit() 返回类型被定义为了 never，在它之后的自然就是「unreachable code」了。  
``` 
function listen(): never {
   while(true){
     let conn = server.accept();
   }
 }

 listen();
 console.log("!!!"); // Unreachable code detected.ts(7027)
```
手动标记函数返回值为 never 类型，来帮助编译器识别「unreachable code」，并帮助我们收窄（narrow）类型。下面是一个没标记的例子：  
``` 
function throwError() {
   throw new Error();
 }

 function firstChar(msg: string | undefined) {
   if (msg === undefined)
     throwError();
   let chr = msg.charAt(1) // Object is possibly 'undefined'.
 }
```
由于编译器不知道 throwError 是一个无返回的函数，所以 throwError() 之后的代码被认为在任意情况下都是可达的，让编译器误会 msg 的类型是 string | undefined。  
这时候如果标记上了 never 类型，那么 msg 的类型将会在空检查之后收窄为 string：  
``` 
function throwError(): never {
   throw new Error();
 }

 function firstChar(msg: string | undefined) {
   if (msg === undefined)
     throwError();
   let chr = msg.charAt(1) // ✅
 }
```
**类型运算**   
_最小因子_  
上文提到 never 可以理解为一个空集，那么它将满足下面的运算规则：  
``` 
T | never => T
T & never =>never
```
never 是类型运算的最小因子。这些规则帮助我们简化了一些琐碎的类型运算，举个例子，像 Promise.race 合并的多个 Promise，有时是无法确切知道时序和返回结果的。现在我们使用一个 Promise.race 来将一个有网络请求返回值的 Promise 和另一个在给定时间之内就会被 reject 的 Promise 合并起来。  
``` 
asyncfunction fetchNameWithTimeout(userId: string): Promise<string> {
   const data = await Promise.race([
     fetchData(userId),
     timeout(3000)
   ])
   return data.userName;
 }
```
下面是一个 timeout 函数的实现，如果超过指定时间，将会抛出一个 Error。由于它是无返回的，所以返回结果定义为了 Promise<never>：  
``` 
function timeout(ms: number): Promise<never> {
   return new Promise((_, reject) => {
     setTimeout(() => reject(newError("Timeout!")), ms)
   })
 }
```
接下来编译器会去推断 Promise.race 的返回值，因为 race 会取最先完成的那个 Promise 的结果，所以在上面这个例子里，它的函数签名类似这样：  
``` 
function race<A, B>(inputs: [Promise<A>, Promise<B>]): Promise<A | B>
```
代入 fetchData 和 timeout 进来，A 则是 { userName: string }，而 B 则是 never。因此，函数输出的 promise 返回值类型为 { userName: string } | never。又因为 never 是最小因子，可以消去。故返回值可简化为 { userName: string }，这正是我们希望的。  
那如果在这里使用了 any 或者 unknown，结果又会怎样呢？  
``` 
// 使用 any
 function timeout(ms: number): Promise<any> {
   ......
 }
 // { userName: string } | any => any，失去了类型检查
 asyncfunction fetchNameWithTimeout(userId: string): Promise<string> {
   ......
   return data.userName; //  data 被推断为 any
 }
```
any 很好理解，虽然能正常通过，但相当于没有类型检查了  
``` 
// 使用 unknown
 function timeout(ms: number): Promise<unknown> {
   ......
 }
 // { userName: string } | unknown => unknown，类型被模糊
 asyncfunction fetchNameWithTimeout(userId: string): Promise<string> {
   ......
   return data.userName; // ❌ data 被推断为 unknown
 }
```
unknown 则是模糊了类型，需要我们手动去收窄类型。  
严格使用 never 来描述「unreachable code」时，编译器便能够帮助我们准确地收窄类型，做到代码即文档。  

_条件类型中使用_  
经常在条件类型中见到 never，它被用于表示 else 的情况  
``` 
type Arguments<T> = T extends (...args: infer A) => any ? A : never
type Return<T> = T extends (...args: any[]) => infer R ? R : never
```
上述推导函数参数和返回值的两个条件类型，即使传入的 T 是非函数类型，也能够得到编译器的提示：  
``` 
// Error: Type '3' is not assignable to type 'never'
 const x: Return<"not a function type"> = 3;
```
在收窄联合类型时，never 也巧妙地发挥了它作为最小因子的作用。比如说下面这个从 T 中排除 null 和 undefined 的例子：  
``` 
type NullOrUndefined = null | undefined
 type NonNullable<T> = T extends NullOrUndefined ? never : T

 // 运算过程
 type NonNullable<string | null>
   // 联合类型被分解成多个分支单独运算
   => (string extends NullOrUndefined ? never : string) | (nullextends  NullOrUndefined ? never : null)
   // 多个分支得到结果，再次联合
   => string | never
   // never 在联合类型运算中被消解
   => string
```
_Exhaustive Check_  
联合类型、代数数据类型等复合类型，可以结合 switch 语句来进行类型收窄：  
``` 
interface Foo {
   type: 'foo'
 }

 interface Bar {
   type: 'bar'
 }

 type All = Foo | Bar;

 function handleValue(val: All) {
   switch (val.type) {
     case'foo':
       // val 此时是 Foo
       break;
     case'bar':
       // val 此时是 Bar
       break;
     default:
       // val 此时是 never
       const exhaustiveCheck: never = val;
       break;
   }
 }
```
后面有人修改了 All 类型，它会发现产生了一个编译错误：  
``` 
type All = Foo | Bar | Baz;

 function handleValue(val: All) {
   switch (val.type) {
     case'foo':
       // val 此时是 Foo
       break;
     case'bar':
       // val 此时是 Bar
       break;
     default:
       // val 此时是 Baz
       // Type 'Baz' is not assignable to type 'never'.(2322)
       const exhaustiveCheck: never = val;
       break;
   }
 }
```
在 default branch 里面 val 会被收窄为 Baz，导致无法赋值给 never，产生一个编译错误。开发者能够意识到 handleValue 里面需要加上针对 Baz 的处理逻辑。通过这个办法，可以确保 handleValue 总是穷尽 (exhaust) 了 All 所有可能的类型。

原文:  
[never 和 unknown 的优雅之道](https://mp.weixin.qq.com/s/P7YwM3IwCmXZ6x8NdnsFHQ)
