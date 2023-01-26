# 爆ant design的定位组件！floating-ui
## 定位组件的实现的复杂度在哪
拿最简单的情况来看，如何把红框部分渲染到按钮”更多“的下方呢？  
可以计算更多按钮的getBoundingRect()，返回  
``` 
const reference = {
  top: xx, // 按钮距离浏览器顶部的距离
  left: xx， // 按钮距离浏览器左边的距离
  width: xx， // 按钮的宽：没有padding
  height：xx，// 按钮的高：没有padding
  ...等等其他属性
}
```
所以红框部分左上角的坐标就轻易的计算出来了,上面的数据在reference对象上，所以借助reference的定位，计算红框部分的下拉框的定位是在哪  
``` 
{
    position: 'absolute',
    top: reference.top + window.pageYOffset // 竖直方向滚动距离 + reference.height
    left: reference.left + window.pageXOffset // 横向滚动距离 
}
```
为啥是上面这么计算呢，假如没有滚动条滚动，那么红框部分的绝对定位的top，是不是等于按钮的距离浏览器顶部的高度 + 本身的高度  
然后，如果滚动条滚动了的话，是不是要在上面top的基础上加上这段距离，就是红框部分在文档流绝对定位的top。  
到此为止，就是最基本的定位组件的逻辑了  
**复杂度1**  
拿下拉框来说，下拉框一般是在下面，但是我可以定位到上边吧？左边，右边也没啥吧，再过分点，右上，左下？定位组件要处理对吧  
**复杂度2**  
假设，定位在下面，想向左偏移8px，向下偏移3px咋办，是不是应该有暴露一个口子  

**复杂度3**  
假设定位在下面，那么一直滚，马上就要滚动到看不见下拉框了,此时我想让定位在上面，能不能自动帮我处理？  

**复杂度4**  
是不是还有可能超出浏览器视口了，想要自动处理  

**复杂度5**  
此时我定位了一次，但是有可能滚动容器不是window，是另一个div，这个计算咋办？还有，是不是我滚动的时候，我要监听滚动事件，还要监听浏览器resize事件，因为我定位的值可能会变？为啥呢，我们上面复杂度3是不是自动帮我们在滚动的时候调整位置，所以你不监听滚动事件你咋知道要调整位置了？  

## 国内组件库怎么实现这个功能
### floating-ui为啥代码质量比ant高
它是以中间件的形式去处理的，思路是什么呢？它假设最开始有一个 computePosition函数，我们假设上面提到的复杂度都没有，也就是不考虑的前提下，我们怎么计算定位组件的坐标，也就是我们最前面的图里说的，红色框部分绝对定位的的的top值和left值：  
API如下：  
``` 
computePosition(要挂载的dom节点，下拉框组件，参数...)
```
然后刚才提到的复杂度，它分别用中间件的形式去处理，比如复杂度2，是想定位之后还有点偏移量，floating-ui咋做的呢   
``` 
import {computePosition, offset} from '@floating-ui/dom';

// referenceEl： 要挂载的dom节点
// floatingEl：下拉框组件（或者说想要挂载到上面referenceEl的dom元素）
computePosition(referenceEl, floatingEl, {
  middleware: [offset(10)],
});
```
如上，offset就是一个中间件，offset(10)，就是向左偏移10px  
如果想处理复杂度3呢，用另一个中间件  
``` 
import {computePosition, flip} from '@floating-ui/dom';
 
computePosition(referenceEl, floatingEl, {
  middleware: [flip()],
});
```
其实所有这些复杂度的解决方案，在floating-ui里都是以中间件的形式去处理的，还可以传多个中间件解决多个问题  

### 中间件的形式好在哪
可以自定义很多中间件了，也就是你的组件不仅仅提供了很多功能，解决了很多常用的问题，你还允许用户写代码去拓展  

**代码中间件原理**  
floating-ui的computePosition API是怎么实现的，它是floating-ui的核心方法，是串联所有中间件的基础。  
``` 
let {x, y} = 求出红色框里的下拉框绝对定位的x坐标和y坐标

// 记录原始placement
  let statefulPlacement = placement;
  // 所有中间件导出的值都挂载到下面的对象上
  let middlewareData: MiddlewareData = {};
  
  // 数据经过middleware的处理
  // middleware是一个数组，存放所有中间件，就是我们上面说的处理每一个复杂度的对象
  for (let i = 0; i < middleware.length; i++) {
   // name是中间件的名字，fn是处理复杂度的逻辑
    const {name, fn} = middleware[i];
   
   
   // 通过把最前面计算的x，y经过fn的处理，得到了新的x,y的值
   // data是指返回的数据，想让后面的中间件也能访问到的数据
    /**
     * 每个middleware需要返回
     * x 新的x坐标
     * y 新的y坐标
     * data 
     * reset
     */
    const {
      x: nextX,
      y: nextY,
      data,
      reset,
    } = await fn({
      /**
       * 每个middleware收到的参数
       * x 目前的x坐标
       * y 目前的y坐标
       * initialPlacement 最初传入的placement
       * placement 
       * middlewareData middleware返回的额外数据
       */
      x,
      y,
      initialPlacement: placement,
      placement: statefulPlacement,
      strategy,
      middlewareData,
      rects,
      platform,
      elements: {reference, floating},
    });

    x = nextX ?? x;
    y = nextY ?? y;

    // 每次处理后的数据想要让后面的中间件访问，就需要挂载到middlewareData对象
    // 这个对象非常好啊，用name隔离了作用域
    middlewareData = {
      ...middlewareData,
      [name]: {
        ...middlewareData[name],
        ...data,
      },
    };
```
最后return出被处理完的x,y坐标，或者自动帮我们监听滚动事件和resize事件，然后拿着x,y就可以赋在css的绝对定位的top和left上，实现了定位。  
每次处理后的数据想要让后面的中间件访问，就需要挂载到middlewareData对象，这个对象非常好啊，用name隔离了作用域，这就是比koa这个框架处理的高明之处，koa里的ctx对象就像一个垃圾桶，什么属性都往上面挂载，挂载太多了，你也不知道是哪个中间件挂载的  
### 中间件如何写
``` 
export const offset = (value: Options = 0): Middleware => ({
  name: 'offset', // 中间件名字
  options: value,  // 传给中间件的值
  async fn(middlewareArguments) { // 中间件处理函数
    const {x, y} = middlewareArguments;
    const diffCoords = await convertValueToCoords(middlewareArguments, value);

    return {
      x: x + diffCoords.x,
      y: y + diffCoords.y,
      data: diffCoords,
    };
  },
});
```
所以如果市面上的组件库的每个组件都是这个形式暴露给用户，就是提供插件式的自定义的中间件，那么整个组件库的拓展性可以说碾压市面上国内所有的react的组件库



原文:  
[不吹牛，完爆ant design的定位组件！floating-ui来也](https://juejin.cn/post/7171054283591254029)
