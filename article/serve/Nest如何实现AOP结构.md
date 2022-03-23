# Nest如何实现AOP结构
Nest.js 是一个 Node.js 的后端框架，它对 express 等 http 平台做了一层封装，解决了架构问题。它提供了 express 没有的 MVC、IOC、AOP 等架构特性，使得代码更容易维护、扩展。  
**MVC、IOC**  
MVC 是 Model View Controller 的简写。MVC 架构下，请求会先发送给 Controller，由它调度 Model 层的 Service 来完成业务逻辑，然后返回对应的 View。  
Nest.js 提供了 @Controller 装饰器用来声明 Controller，用 @Injectable 装饰器来声明Service。
通过 @Controller、@Injectable 装饰器声明的 class 会被 Nest.js 扫描，创建对应的对象并加到一个容器里，这些所有的对象会根据构造器里声明的依赖自动注入，也就是 DI（dependency inject），这种思想叫做 IOC（Inverse Of Control）  
IOC 架构的好处是不需要手动创建对象和根据依赖关系传入不同对象的构造器中，一切都是自动扫描并创建、注入的。  
Nest.js 提供了 AOP （Aspect Oriented Programming）的能力，也就是面向切面编程的能力  
## AOP  
一个请求过来，可能会经过 Controller（控制器）、Service（服务）、Repository（数据库访问） 的逻辑，如果想在这个调用链路里加入一些通用逻辑该怎么加呢？比如日志记录、权限控制、异常处理等。  
容易想到的是直接改造 Controller 层代码，加入这段逻辑。这样可以，但是不优雅，因为这些通用的逻辑侵入到了业务逻辑里面。  
是不是可以在调用 Controller 之前和之后加入一个执行通用逻辑的阶段呢？这样的横向扩展点就叫做切面，这种透明的加入一些切面逻辑的编程方式就叫做 AOP （面向切面编程）。  
AOP 的好处是可以把一些通用逻辑分离到切面中，保持业务逻辑的存粹性，这样切面逻辑可以复用，还可以动态的增删  
Express 的中间件的洋葱模型也是一种 AOP 的实现，因为你可以透明的在外面包一层，加入一些逻辑，内层感知不到。  
Nest.js 实现 AOP 的方式更多，一共有五种，包括 Middleware、Guard、Pipe、Inteceptor、ExceptionFilter  
**Middleware**  
Nest.js 基于 Express 自然也可以使用中间件，但是做了进一步的细分，分为了全局中间件和路由中间件：  
全局中间件就是 Express 的那种中间件，在请求之前和之后加入一些处理逻辑，路由中间件则是针对某个路由来说的，范围更小一些  
**Guard**  
Guard 是路由守卫的意思，可以用于在调用某个 Controller 之前判断权限，返回 true 或者 flase 来决定是否放行  
创建 Guard 的方式是这样的：  
```
@Injectable
export class RolesGuard implements CanActivate {
    canActivate(
        context:ExecutionContext,
    ): boolean | Promise<boolean> | Observable<boolean> {
        return true;
    }
}
```
Guard 要实现 CanActivate 接口，实现 canActive 方法，可以从 context 拿到请求的信息，然后做一些权限验证等处理之后返回 true 或者 false。  
通过 @Injectable 装饰器加到 IOC 容器中，然后就可以在某个 Controller 启用了  
Controller 本身不需要做啥修改，却透明的加上了权限判断的逻辑，这就是 AOP 架构的好处。  
就像 Middleware 支持全局级别和路由级别一样，Guard 也可以全局启用  
Guard 可以抽离路由的访问控制逻辑，但是不能对请求、响应做修改，这种逻辑可以使用 Interceptor  
**Interceptor**  
Interceptor 是拦截器的意思，可以在目标 Controller 方法前后加入一些逻辑  
nterceptor 要实现 NestInterceptor 接口，实现 intercept 方法，调用 next.handle() 就会调用目标 Controller，可以在之前和之后加入一些处理逻辑。  
Controller 之前之后的处理逻辑可能是异步的。Nest.js 里通过 rxjs 来组织它们，所以可以使用 rxjs 的各种 operator。  
Interceptor 支持每个路由单独启用，只作用于某个 controller，也同样支持全局启用，作用于全部 controller  
除了路由的权限控制、目标 Controller 之前之后的处理这些都是通用逻辑外，对参数的处理也是一个通用的逻辑，所以 Nest.js 也抽出了对应的切面，也就是 Pipe  
**Pipe**  
Pipe 是管道的意思，用来对参数做一些验证和转换  
Pipe 要实现 PipeTransform 接口，实现 transform 方法，里面可以对传入的参数值 value 做参数验证，比如格式、类型是否正确，不正确就抛出异常。也可以做转换，返回转换后的值。  
内置的有 8 个 Pipe，从名字就能看出它们的意思：
- ValidationPipe
- ParseIntPipe
- ParseBoolPipe
- ParseArrayPipe
- ParseUUIDPipe
- DefaultValuePipe
- ParseEnumPipe
- ParseFloatPipe

不管是 Pipe、Guard、Interceptor 还是最终调用的 Controller，过程中都可以抛出一些异常，如何对某种异常做出某种响应呢？  
这种异常到响应的映射也是一种通用逻辑，Nest.js 提供了 ExceptionFilter 来支持  
**ExceptionFilter**  
ExceptionFilter 可以对抛出的异常做处理，返回对应的响应  
首先要实现 ExceptionFilter 接口，实现 catch 方法，就可以拦截异常了，但是要拦截什么异常还需要用 @Catch 装饰器来声明，拦截了异常之后，可以异常对应的响应，给用户更友好的提示。  
不是所有的异常都会处理，只有继承 HttpException 的异常才会被 ExceptionFilter 处理，Nest.js 内置了很多 HttpException 的子类：  
- BadRequestException
- UnauthorizedException
- NotFoundException
- ForbiddenException
- NotAcceptableException
- RequestTimeoutException
- ConflictException
- GoneException
- PayloadTooLargeException
- UnsupportedMediaTypeException
- UnprocessableException
- InternalServerErrorException
- NotImplementedException
- BadGatewayException
- ServiceUnavailableException
- GatewayTimeoutException

当然，也可以自己扩展：  
```
export class ForbiddenException extends HttpException {
    constructor() {
        super('Forbidden', HttpStatus.FORBIDDEN)
    }
}
```
Nest.js 通过这样的方式实现了异常到响应的对应关系，代码里只要抛出不同的 HttpException，就会返回对应的响应，很方便.  
同样，ExceptionFilter 也可以选择全局生效或者某个路由生效  
**几种 AOP 机制的顺序**  
Middleware、Guard、Pipe、Interceptor、ExceptionFilter 都可以透明的添加某种处理逻辑到某个路由或者全部路由，这就是 AOP 的好处  
但是它们之间的顺序关系是什么呢？调用关系这个得看源码了。  
 进入这个路由的时候，会先调用 Guard，判断是否有权限等，如果没有权限，这里就抛异常了,抛出的 HttpException 会被 ExceptionFilter 处理  
 如果有权限，就会调用到拦截器，拦截器组织了一个链条，一个个的调用，最后会调用的 controller 的方法  
 调用 controller 方法之前，会使用 pipe 对参数做处理  
 会对每个参数做转换：  
 ExceptionFilter 的调用时机很容易想到，就是在响应之前对异常做一次处理。  
而 Middleware 是 express 中的概念，Nest.js 只是继承了下，那个是在最外层被调用。

参考：  
[Nest.js 是如何实现 AOP 架构的？](https://mp.weixin.qq.com/s/S-wlEkk6rjv8ID9L6RydPQ)