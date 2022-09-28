# transform 支持单独赋值改变
利用 transform 配合绝对定位的 top、left 进行任意元素的水平垂直居中，像是这样：  
```
<div></div>  

div {
    width: 200px;
    height: 200px;
    background: #000;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```
想对这个元素进行一个缩放的动画，该怎么做呢？会是这样：  
```
div {
    width: 200px;
    height: 200px;
    background: #000;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
@keyframes scale {
    0% {
        transform: translate(-50%, -50%) scale(1);
    }
    100% {
        transform: translate(-50%, -50%) scale(1.2);
    }
}
```
上述的 @keyframes 代码中，想改变的其实只有 scale() 的值，但是基于现有的 CSS 机制，必须把前面控制定位的 translate() 一并写上。  

**transform 拆开书写**  
为了解决这个痛点，规范支持了将 transform 分开书写的方式  
对于这一句，transform: translate(-50%, -50%) scale(1)，我们可以分别拆开成 translate 和 scale  
```
div {
    width: 200px;
    height: 200px;
    background: #000;
    position: absolute;
    top: 50%;
    left: 50%;
    // transform: translate(-50%, -50%); 删掉这句
    translate: -50% -50%;
    scale: 1;
    animation: scale 1s infinite linear alternate;
    
}

@keyframes scale {
    0% {
        scale: 1;
    }
    100% {
        scale: 1.2
    }
}
```
以通过 translate 和 scale 分开控制它们，这样就能在 @keyframes 中，只进行 scale 的动画了  
根据规范 -- W3 - individual-transforms，于 transform 而言，可以将它整个拆解为：  
1. translate
2. rotate
3. scale

譬如 translate 接收 3 个参数，第 2、3 个参数如果不存在，则为零（例如：translate: 50% 50% 0）。  
下面两组代码是等效的：  
```
{
    transform: translate(-50%, -50%) rotateZ(30deg) scale(1.2);
}
{
    translate: -50% -50%;
    rotate: Z 30deg;
    scale: 1.2;
}
```
**顺序对 transform 的影响**  
使用的是 transform，将它们全部写在一起，像是这样：  
```
div {
     transform: rotateY(90deg) translateZ(400px) scale(1.2);
}
```
此时，元素的 transform 变换遵循的是从左向右进行变换。也就是先旋转 rotate，再 translateZ()，最后缩放 scale()  
如果分开来写：  
```
div {
    rotate: Y 90deg;
    translate: 0 0 400px;
    scale: 1.2;
}
```
代码属性的顺序是 rotate --> translate --> scale，但是实际上，无论书写顺序如何，解析的时候都会按照首先 translate，然后 rotate，最后 scale 的顺序进行！
实际使用的时候一定要注意，矩阵变换的顺序会影响最终的效果。如果对顺序有严格的要求，还是需要合在一起写  

原文:  
[解放生产力！transform 支持单独赋值改变](https://mp.weixin.qq.com/s/t5DdI192Tr4ASzJ9INpdWg)