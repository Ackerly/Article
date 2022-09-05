# Command(命令模式)
将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作。  
## 举例
**点菜是命令模式**  
为什么顾客会找服务员点菜，而不是直接冲到后厨盯着厨师做菜？因为做菜比较慢，肯定会出现排队的现象，而且有些菜可能是一起做效率更高，所以将点菜和做菜分离比较容易控制整体效率。  
点菜就是一个个请求，点菜员记录的菜单就是将请求生成的对象，点菜员不需要关心怎么做菜、谁来做，他只要把菜单传到后厨即可，由后厨统一调度。  
**大型软件系统的操作菜单**  
大型软件操作系统都有一个特点，即软件非常复杂，菜单按钮非常多。但由于菜单按钮本身并没有业务逻辑，所以通过菜单按钮点击后触发的业务行为不适合由菜单按钮完成，此时可利用命令模式生成一个或一系列指令，由软件系统的实现部分来真正执行。  
**浏览器请求排队**  
浏览器的请求不仅会排队，还会取消、重试，因此是个典型的命令模式场景。如果不能将 window.fetch 序列化为一个个指令放入到队列中，是无法实现请求排队、取消、重试的。  
## 解释
一个请求指的是来自客户端的一个操作，比如菜单按钮点击。重点在点击后并不直接实现，而是将请求封装为一个对象，可以理解为从直接实现：  
``` 
function onClick() {
  // ... balabala 实现逻辑
}
```
改为生成一个对象，序列化这个请求：  
``` 
function onClick() {
  concreteCommand.push({
    // ... 描述这个请求
  })
  // 执行所有命令队列
  concreteCommand.executeAll()
}
```
看上去繁琐了一些，但得到了后面所说的好处：“从而可用不同的请求对客户进行参数化”，也就是可以对任何请求进行参数化存储，可以在任意时刻调用。这相当于掌握了执行时机，可以在任意时刻调用，以实现排队或记录日志，如果再记录下反向操作信息，就可以实现撤销重做了。  

![image](./../../assets/images/design%20patterns/Command.png)  

``` 
const command1 = new Command('test1')
const command2 = new Command('test2')

const invoker = new Invoker()
invoker.push(command1)
invoker.push(command2)
invoker.execute()
```
Invoker 内部用一个队列维护，执行的时候其实是 for 循环执行了每个 command.execute():  
``` 
class Invoker {
  push(command) {
    // 队列里推入命令
    this.commands.push(command)
  }

  execute() {
    this.commands.forEach(command => command.execute())
    // 清空 this.commands
  }
}
```
## 缺点
命令模式需要注意序列化大小，一般分为：  
1. 仅记录操作。
2. 记录全量快照。
3. 全量快照共享内存。

记录操作是较为精细的管理方式，并且可以延伸出协同编辑功能。记录快照要注意尽量共享内存，防止快照过大，而且协同编辑场景因为快照无法做冲突处理，所以快照模式在协同编辑场景无法应用。  
要识别没必要使用命令模式的场景，对于没有撤销重做的前端大部分场景来说，都无需改为命令模式。  

原文: 
[Command（命令模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/180.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Command%20%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
