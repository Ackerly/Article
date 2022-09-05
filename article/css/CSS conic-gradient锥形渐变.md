# CSS conic-gradient锥形渐变简介
``` 
.example {
    width: 300px; height: 300px;
    background: conic-gradient(from 45deg, white, black, white);
}
```
## 语法
``` 
conic-gradient( [ from <angle> ]? [ at <position> ]?, <angular-color-stop-list> )
```
锥形渐变组成:  
1. 起始角度
2. 中心位置
3. 角渐变断点

其中起始角度和中心位置都是可以省略的，因此，最简单的锥形渐变用法：
``` 
.example {
    width: 300px; height: 150px;
    background-image: conic-gradient(white, deepskyblue);
}
```
渐变起始角度改成45度，中心点位置移动到相对元素左上角25%的位置
``` 
conic-gradient(from 45deg at 25% 25%, white, deepskyblue);
```
**锥形渐变的颜色断点**  
锥形渐变的颜色断点数据类型是<angular-color-stop-list>，是角颜色断点列表，有别于线性渐变和径向渐变的断点列表，使用的是角度值，而非长度值。  
``` 
conic-gradient(white, deepskyblue 45deg, white);
```
deepskyblue 45deg这里使用的是'45deg'是个角度值，于是仰望鑫空间，可以明显看到2点钟方向的颜色最深。  
角渐变断点中设置的角度值是一个相对角度值，最终渲染的角度值是和起始角度累加的值，例如：
``` 
conic-gradient(from 45deg, white, deepskyblue 45deg, white);
```
deepskyblue实际渲染的坐标角度是90deg（45deg + 45deg）,有此可见，锥形渐变中颜色断点角度值和百分比值没有什么区别，两者可以互相转换，一个完整的旋转总共360度，因此，45deg等同于12.5%，因此，下面两段CSS效果是一模一样的：  
``` 
/* 下面两段语句效果一样 */
conic-gradient(white, deepskyblue 45deg, white);
conic-gradient(white, deepskyblue 12.5%, white);
```
如果是渐变转换点，角度值和百分比值也是也互相转换的，例如下面的两条语句都是合法的：
``` 
/* 合法 */
conic-gradient(white, 12.5%, deepskyblue);
/* 合法 */
conic-gradient(white, 45deg, deepskyblue);
```
## 应用
实现饼图效果
``` 
.pie {
    width: 150px; height: 150px;
    border-radius: 50%;
    background: conic-gradient(yellowgreen 40%, gold 0deg 75%, deepskyblue 0deg);   
}
```
'gold 0deg 75%'这里就是渐变范围语法（IE浏览器不支持）。  
这里设置的数值应该是40%，或者144deg，而不是0deg，不过渐变断点有个特性，如果后面的渐变断点位置值比前面的渐变断点位置值小的时候，后面的渐变断点的位置值会按照前面较大的渐变断点位置值渲染。  
使用锥形渐变实现的基于色相和饱和度的取色盘，
``` 
.hs-wheel {
    width: 150px; height: 150px;
    border-radius: 50%;
    background: radial-gradient(closest-side, gray, transparent),
        conic-gradient(red, magenta, blue, aqua, lime, yellow, red);
}
```
实现棋盘效果（PNG透明背景的灰白网格效果）最简单的方法，  
```
.checkerboard {
    width: 300px; height: 160px;
    background: conic-gradient(#eee 25%, white 0deg 50%, #eee 0deg 75%, white 0deg) 0 / 20px 20px;
}
```
渐变实现很实用的loading效果  
``` 
.loading {
    width: 100px; height: 100px;
    border-radius: 50%;
    background: conic-gradient(deepskyblue, 30%, white);
    --mask: radial-gradient(closest-side, transparent 75%, black 76%);
    -webkit-mask-image: var(--mask);
    mask-image: var(--mask);
    animation: spin 1s linear infinite reverse;
}
@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
```
## 兼容性
锥形渐变是CSS Images Module Level 4规范中新定义的一种渐变，所以兼容性要比线性渐变和径向渐变差很多，目前Chrome和Safari浏览器均已支持，Firefox即将正式支持。

原文: 
[CSS conic-gradient锥形渐变简介](https://www.zhangxinxu.com/wordpress/2020/04/css-conic-gradient/)
