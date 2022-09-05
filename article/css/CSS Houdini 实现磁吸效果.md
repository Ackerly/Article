# CSS Houdini 实现磁吸效果
**CSS Paint API**  
Houdini 提供了 Paint API，基于这个 API 提供的 2D 渲染  
context (CanvasRenderingContext2D 的子集)，开发者就可以通过编写 JS 代码来为元素提供 background-image/border-image/mask-image 等  
要创建这样的渲染上下文，需要先创建一个 PaintWorklet，和 Worker 类似，Worklet 可以让绘制的代码脱离 JS 主线程去执行  
首先创建一个最基础的 Worklet 类，这个类需要提供 paint，浏览器在合适的时候调用这个方法，并传入 ctx (类似 Canvas 上下文) 以及 size 元素的尺寸信息。调用 ctx 的绘制方法，就可以绘制任何你希望绘制的内容了，以下面的自定义背景色绘制器为例：  
``` 
 class CustomBackgroundPainter {
     paint(ctx, size) {
       const color = '#2ecc71';
       const margin = 30
       // 设置填充色
       ctx.fillStyle = color;
       // 画一个矩形覆盖整个元素
       ctx.rect(margin, margin, size.width - margin * 2, size.height - margin * 2);
       // 颜色填充
       ctx.fill();
     }
   }

 // 注册绘制器
 registerPaint('custom-background', CustomBackgroundPainter);
```
接着还需要加载 Painter  
``` 
CSS.paintWorklet.addModule('./custom-background.js')
```
除了以文件 url 的形式加载，也可以直接使用 base64 的模式：  
``` 
CSS.paintWorklet.addModule(`data:application/javascript;charset=utf8,${encodeURIComponent(`
   class CustomBackgroundPainter {
     paint(ctx, size) {
       const color = '#2ecc71';
       const margin = 30
       // 设置填充色
       ctx.fillStyle = color;
       // 画一个矩形覆盖整个元素
       ctx.rect(margin, margin, size.width - margin * 2, size.height - margin * 2);
       // 颜色填充
       ctx.fill();
     }
   }

   // Register our class under a specific name
   registerPaint('custom-background', CustomBackgroundPainter); `)}`)
```
完成之后，在需要使用自定义 Painter 的元素 CSS 中应用即可。  
``` 
background-image: paint(custom-background);
```
通过 Painter API 实现了一个支持 background margin 的效果  
``` 
class CustomBackgroundPainter {
     // 申明依赖
     static get inputProperties() { return ['--backgroundColor', '--backgroundMargin']; }

     paint(ctx, size, props) {
       // 从 CSS Variables 中获取
       const color = props.get('--backgroundColor');
       const margin = props.get('--backgroundMargin');

       // 设置填充色
       ctx.fillStyle = color;
       // 画一个矩形覆盖整个元素
       ctx.rect(margin, margin, size.width - margin * 2, size.height - margin * 2);
       // 颜色填充
       ctx.fill();
     }
 }
```
声明静态方法 inputProperties，这个方法返回需要监听的 CSS Variables 列表，在这些变量发生变更的时候，浏览器就会重新调用 paint 方法进行绘制  
paint 方法的第三个参数会返回一个 StylePropertyMap，调用 get 方法就可以读取到最新的值。  
现在就可以通过 CSS Variables 自定义每个元素 custom-background 的 margin 和 color 了。  
``` 
--backgroundColor: #ff4c4b;
 --backgroundMargin: 60;
 background-image: paint(custom-background);
```
**磁吸效果**  
背景需要感知鼠标的位置，我们可以给元素绑定鼠标事件，将鼠标位置通过 CSS Variables 的形式传给 Worklet  
``` 
element.addEventListener('mouseenter', function (e) {
   this.style.setProperty('--mouse-x', e.clientX);
   this.style.setProperty('--mouse-y', e.clientY);
 })
 element.addEventListener('mousemove', function (e) {
   this.style.setProperty('--mouse-x', e.clientX);
   this.style.setProperty('--mouse-y', e.clientY);
 })
```
除此之外可能还需要配置每个点的颜色，大小，间隔以及影响半径，全都定义好，在 paint 方法中读取出来。  
``` 
static get inputProperties() { return ['--mouse-x', '--mouse-y', '--magnet-color', '--magnet-size', '--magnet-gap', '--magnet-radius']; }
 const mouseX = parseInt(props.get('--mouse-x'))
 const mouseY = parseInt(props.get('--mouse-y'))
 const color = props.get('--magnet-color')
 const lineWidth = parseInt(props.get('--magnet-size'))
 const gap = parseInt(props.get('--magnet-gap'))
 const radius = parseInt(props.get('--magnet-radius'))
```
接着就可以计算每个点和鼠标位置的距离 distance，通过这个距离推导出它的宽度（距离约小宽度越大）和角度，超出受力范围的点就不受影响。  
```  
class MagnetMatrixPaintWorklet {
     //...

     paint(ctx, size, props) {
       //...
       ctx.lineCap = "round";
       for (let i = 0; i * gap < size.width; i++) {
         for (let j = 0; j * gap < size.height; j++) {
           const posX = i * gap           const posY = j * gap           const distance = Math.sqrt(Math.pow(posX - mouseX, 2) + Math.pow(posY - mouseY, 2))
           const width = distance < radius ? (1 - distance / radius * distance / radius) * gap * 0.4 : 0
           const startPosX = posX - width * 0.5
           const endPosX = posX + width * 0.5
           const rotate = Math.atan2(mouseY - posY, mouseX - posX)

           ctx.save()
           ctx.beginPath();
           ctx.translate(posX, posY);
           ctx.rotate(rotate);
           ctx.translate(-posX, -posY);
           ctx.moveTo(startPosX, posY);
           ctx.strokeStyle = color
           ctx.lineWidth = lineWidth;
           ctx.lineCap = "round";
           ctx.lineTo(endPosX, posY);
           ctx.stroke()
           ctx.closePath()
           ctx.restore()
         }
       }
     }
 }
```
距离越近力越大，而且按照现象来看不是线性的，我用 (1 - distance /radius * distance /radius) 来模拟这个效果。  
已知绘制线的中点，长度和旋转角度，要绘制这条线可以先 translate 到中心点的位置，进行 rotate 之后，再将 translate revert 回去，相当于整个画板以中心点为圆心旋转。  
``` 
ctx.translate(posX, posY);
 ctx.rotate(rotate);
 ctx.translate(-posX, -posY);
```


原文: 
[CSS Houdini 实现磁吸效果](https://mp.weixin.qq.com/s/g8tj4XkQg3NSMunlHE4IJQ)
