# CSS文字动画
## 文字与阴影  
将字体与字体阴影 text-shadow，相结合，阴影的不同运用方式，产生不一样的效果。  
**长阴影文字效果**  
通过多层次，颜色逐渐变化（透明）的阴影变化，可以生成长阴影：  
``` 
div {
  text-shadow: 0px 0px #992400, 1px 1px rgba(152, 36, 1, 0.98), 2px 2px rgba(151, 37, 2, 0.96), 3px 3px rgba(151, 37, 2, 0.94), 4px 4px rgba(150, 37, 3, 0.92), 5px 5px rgba(149, 38, 4, 0.9), 6px 6px rgba(148, 38, 5, 0.88), 7px 7px rgba(148, 39, 5, 0.86), 8px 8px rgba(147, 39, 6, 0.84), 9px 9px rgba(146, 39, 7, 0.82), 10px 10px rgba(145, 40, 8, 0.8), 11px 11px rgba(145, 40, 8, 0.78), 12px 12px rgba(144, 41, 9, 0.76), 13px 13px rgba(143, 41, 10, 0.74), 14px 14px rgba(142, 41, 11, 0.72), 15px 15px rgba(142, 42, 11, 0.7), 16px 16px rgba(141, 42, 12, 0.68), 17px 17px rgba(140, 43, 13, 0.66), 18px 18px rgba(139, 43, 14, 0.64), 19px 19px rgba(138, 43, 15, 0.62), 20px 20px rgba(138, 44, 15, 0.6), 21px 21px rgba(137, 44, 16, 0.58), 22px 22px rgba(136, 45, 17, 0.56), 23px 23px rgba(135, 45, 18, 0.54), 24px 24px rgba(135, 45, 18, 0.52), 25px 25px rgba(134, 46, 19, 0.5), 26px 26px rgba(133, 46, 20, 0.48), 27px 27px rgba(132, 47, 21, 0.46), 28px 28px rgba(132, 47, 21, 0.44), 29px 29px rgba(131, 48, 22, 0.42), 30px 30px rgba(130, 48, 23, 0.4), 31px 31px rgba(129, 48, 24, 0.38), 32px 32px rgba(129, 49, 24, 0.36), 33px 33px rgba(128, 49, 25, 0.34), 34px 34px rgba(127, 50, 26, 0.32), 35px 35px rgba(126, 50, 27, 0.3), 36px 36px rgba(125, 50, 28, 0.28), 37px 37px rgba(125, 51, 28, 0.26), 38px 38px rgba(124, 51, 29, 0.24), 39px 39px rgba(123, 52, 30, 0.22), 40px 40px rgba(122, 52, 31, 0.2), 41px 41px rgba(122, 52, 31, 0.18), 42px 42px rgba(121, 53, 32, 0.16), 43px 43px rgba(120, 53, 33, 0.14), 44px 44px rgba(119, 54, 34, 0.12), 45px 45px rgba(119, 54, 34, 0.1), 46px 46px rgba(118, 54, 35, 0.08), 47px 47px rgba(117, 55, 36, 0.06), 48px 48px rgba(116, 55, 37, 0.04), 49px 49px rgba(116, 56, 37, 0.02), 50px 50px rgba(115, 56, 38, 0);
}
```
多重阴影以及每重的颜色很难一个一个手动去写，在写长阴影的时候通常需要借助 SASS、LESS 去帮助节省时间：  
``` 
@function makelongrightshadow($color) {
    $val: 0px 0px $color;

    @for $i from 1 through 50 {
        $color: fade-out(desaturate($color, 1%), .02);
        $val: #{$val}, #{$i}px #{$i}px #{$color};
    }

    @return $val;
}
div {
    text-shadow: makeLongShadow(hsl(14, 100%, 30%));
}
```
**立体阴影文字效果**  
多层阴影，但是颜色变化没那么强烈，能够塑造一种立体的效果。
``` 
div {
  text-shadow: 0 -1px 0 #ffffff, 0 1px 0 #2e2e2e, 0 2px 0 #2c2c2c, 0 3px 0 #2a2a2a, 0 4px 0 #282828, 0 5px 0 #262626, 0 6px 0 #242424, 0 7px 0 #222222, 0 8px 0 #202020, 0 9px 0 #1e1e1e, 0 10px 0 #1c1c1c, 0 11px 0 #1a1a1a, 0 12px 0 #181818, 0 13px 0 #161616, 0 14px 0 #141414, 0 15px 0 #121212);
}
```
**内嵌阴影文字效果**  
合理的阴影颜色和背景底色搭配，搭配，可以实现类似内嵌效果的阴影
``` 
div {
  color: #202020;
  background-color: #2d2d2d;
  letter-spacing: .1em;
  text-shadow: -1px -1px 1px #111111, 2px 2px 1px #363636;
}
```
**氖光效果**  
只需要设置 3~n 层阴影效果，每一层的模糊半径（文字阴影的第三个参数）间隔较大，并且每一层的阴影颜色相同即可。  
``` 
p {
    color: #fff;
    text-shadow: 
        0 0 10px #0ebeff,
        0 0 20px #0ebeff,
        0 0 50px #0ebeff,
        0 0 100px #0ebeff,
        0 0 200px #0ebeff
}
```
通常运用 Neon 效果时，背景底色都是偏黑色。  
合理运用 Neon 效果，就可以制作非常多有意思的动效。譬如作用于鼠标 hover 上去的效果：  
``` 
p {
    transition: .2s;
 
    &:hover {
        text-shadow: 0 0 10px #0ebeff, 0 0 20px #0ebeff, 0 0 50px #0ebeff, 0 0 100px #0ebeff, 0 0 200px #0ebeff;
    }
}
```
可以选取适当合适的字体，配合动画效果，实现一种渐进的亮灯效果：  
``` 
// html
<p>
  <span id="n">n</span>
  <span id="e">e</span>
  <span id="o">o</span>
  <span id="n2">n</span>
</p>

// css
p:hover span {
  animation: flicker 1s linear forwards;
}
p:hover #e {
  animation-delay: .2s;
}
p:hover #o {
  animation-delay: .5s;
}
p:hover #n2 {
  animation-delay: .6s;
}

@keyframes flicker {
  0% {
    color: #333;
  }
  5%, 15%, 25%, 30%, 100% {
    color: #fff;
    text-shadow: 
      0px 0px 5px var(--color),
      0px 0px 10px var(--color),
      0px 0px 20px var(--color),
      0px 0px 50px var(--color);
      
  }
  10%, 20% {
    color: #333;
    text-shadow: none;
  }
}
```
## 文字与背景
**background-clip 与文字**  
background-clip， 其作用就是设置元素的背景（背景图片或颜色）的填充规则。  
与 box-sizing 的取值非常类似，通常而言，它有 3 个取值，border-box，padding-box，content-box，后面规范新增了一个 background-clip。时至今日，部分浏览器仍需要添加前缀 webkit 进行使用 -webkit-background-clip。  
使用了这个属性的意思是，以区块内的文字作为裁剪区域向外裁剪，文字的背景即为区块的背景，文字之外的区域都将被裁剪掉。  
由于文字设置了颜色，挡住了 div 块的背景，如果将文字设置为透明呢？原本 div 的背景就显现出来了，而文字以外的区域全部被裁剪了，这就是 background-clip:text 的作用  
**利用 background-clip 实现渐变文字**  
利用这个属性，可以轻松的实现渐变色的文字
``` 
{
    background: linear-gradient(45deg, #009688, yellowgreen, pink, #03a9f4, #9c27b0, #8bc34a);
    background-clip: text;
}
```
background-position 或者 filter: hue-rotate()，让渐变动起来：
``` 
{
    background: linear-gradient(45deg, #009688, yellowgreen, pink, #03a9f4, #9c27b0, #8bc34a);
    background-clip: text;
    animation: huerotate 5s infinite;
}

@keyframes huerotate {
    100% {
        filter: hue-rotate(360deg);
    }
}
```
**利用 background-clip 给文字增加高光动画**  
利用 background-clip，可以轻松的给文字增加高光动画。
``` 
// html
<p data-text="Lorem ipsum dolor"> Lorem ipsum dolor </p>

// CSS
p {
    position: relative;
    color: transparent;
    background-color: #E8A95B;
    background-clip: text;
}
p::after {
    content: attr(data-text);
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: linear-gradient(120deg, transparent 0%, transparent 6rem, white 11rem, transparent 11.15rem, transparent 15rem, rgba(255, 255, 255, 0.3) 20rem, transparent 25rem, transparent 27rem, rgba(255, 255, 255, 0.6) 32rem, white 33rem, rgba(255, 255, 255, 0.3) 33.15rem, transparent 38rem, transparent 40rem, rgba(255, 255, 255, 0.3) 45rem, transparent 50rem, transparent 100%);
    background-clip: text;
    background-size: 150% 100%;
    background-repeat: no-repeat;
    animation: shine 5s infinite linear;
}
@keyframes shine {
	0% {
		background-position: 50% 0;
	}
	100% {
		background-position: -190% 0;
	}
}
```
**mask与文字**  
添加了 mask 属性的元素，其内容会与 mask 表示的渐变的 transparent 的重叠部分，并且重叠部分将会变得透明  
利用 mask，可以实现各种文字的出场特效：  
``` 
// html
<div>
    <p>Hello MASK</p>
</div>

// CSS
div {
    mask: radial-gradient(circle at 50% 0%, #000, transparent 30%);
    animation: scale 6s infinite;
}
@keyframes scale {
    0% {
        mask-size: 100% 100%;
    }
    60%,
    100% {
        mask-size: 150% 800%;
    }
}
```
## 文字与混合模式(mix-blend-mode)及滤镜(filter)
**给文字添加边框，生成镂空文字**  
利用 -webkit-text-stroke，给文字快速的添加边框，利用这个，可以快速生成镂空型的文字：  
``` 
p {
    -webkit-text-stroke: 3px #373750;
}
```
用到的属性 -webkit-text-stroke 带了 webkit 前缀，存在一定的兼容性问题。  
所以，在更早的时候，我们还会使用 text-shadow，生成镂空文字。  
因为使用的是阴影，所以有很明显的虚化的感觉，存在一定的瑕疵。  
还有一种非常绕的方法，利用混合模式加上滤镜，也能生成镂空文字。  
``` 
p {
    position: relative;
    color: #fff;

    &::after {
        content: 'Magic Text';
        position: absolute;
        left: 0;
        top: 0;
        color: #fff;
        mix-blend-mode: difference;
        filter: blur(1px);
    }
}
```
利用 filter: blur(1px) 生成了一个比原字体稍微大一点点的字体覆盖在原字体之上，再利用 mix-blend-mode: difference 消除掉了同色的部分，只留下了利用模糊滤镜多出来的那一部分。  
> mix-blend-mode: difference: 差值模式（Difference），作用是查看每个通道中的颜色信息，比较底色和绘图色，用较亮的像素点的像素值减去较暗的像素点的像素值。与白色混合将使底色反相；与黑色混合则不产生变化。  

**利用混合模式，生成渐变色镂空文字**  
可以利用混合模式 mix-blend-mode: multiply 生成渐变色的文字。  
> mix-blend-mode: multiply: 正片叠底（multiply），将两个颜色的像素值相乘，然后除以255得到的结果就是最终色的像素值。通常执行正片叠底模式后的颜色比原来两种颜色都深。任何颜色和黑色正片叠底得到的仍然是黑色，任何颜色和白色执行正片叠底则保持原来的颜色不变，而与其他颜色执行此模式会产生暗室中以此种颜色照明的效果。  

``` 
p {
    position: relative;
    -webkit-text-stroke: 3px #9a9acc;

    &::before{
		content: ' ';
		width: 100%;
		height: 100%;
		position: absolute;
		left: 0;
		top: 0;
		background-image: linear-gradient(45deg, #ff269b, #2ab5f5, #ffbf00);
		mix-blend-mode: multiply;
	}
}
```
mix-blend-mode: multiply 发挥的作用和 mask 非常的类似，其实是生成了一幅渐变图案，但是只有在文字轮廓内，渐变颜色才会显现  
**利用混合模式，生成光影效果文字**  
利用剩余的一个 ::after 伪类，再添加一个 mix-blend-mode: color-dodge 混合模式，给文字加上最后的点缀，实现美妙的光影效果。  
> mix-blend-mode: color-dodge: 颜色减淡模式（Color Dodge），查看每个通道的颜色信息，通过降低“对比度”使底色的颜色变亮来反映绘图色，和黑色混合没变化。

``` 
p {
    position: relative;
    -webkit-text-stroke: 3px #7272a5;

    &::before {
		content: ' ';
		background-image: linear-gradient(45deg, #ff269b, #2ab5f5, #ffbf00);
		mix-blend-mode: multiply;
    }

    &::after {
        content: "";
        position: absolute;
        background: radial-gradient(circle, #fff, #000 50%);
        background-size: 25% 25%;
        mix-blend-mode: color-dodge;
        animation: mix 8s linear infinite;
    }
}

@keyframes mix {
    to {
        transform: translate(50%, 50%);
    }
}
```
**利用混合模式实现文字与底色反色的效果**  
利用 mix-blend-mode: difference 差值模式，实现一种文字与底色反色的 Title 效果。  
> mix-blend-mode: difference: 差值模式（Difference），作用是查看每个通道中的颜色信息，比较底色和绘图色，用较亮的像素点的像素值减去较暗的像素点的像素值。与白色混合将使底色反相；与黑色混合则不产生变化。

现一个黑白相间的背景，文本的颜色为白色，配合上差值模式，即可实现黑底上的文字为白色，白底上的文字为黑色的效果。
``` 
p  {
    background: repeating-radial-gradient(circle at 200% 200%, #000 0, #000 150px, #fff 150px, #fff 300px);

    &::before {
        content: "LOREM IPSUM";
        color: #fff;
        mix-blend-mode: difference;
    }
}
```
**利用混合模式实现动态类抖音风格 Glitch 效果**  
关键点
- 利用 mix-blend-mode: lighten 混合模式实现两段文字结构重叠部分为白色
- 利用元素位移完成错位移动动画，形成视觉上的冲击效果

也不是一定要使用混合模式去使得融合部分为白色，可以仅仅是使用这个配色效果，基于上面效果的另外一个版本，没有使用混合模式  
关键点：  
- 利用了伪元素生成了文字的两个副本
- 视觉效果由位移、遮罩、混合模式完成
- 配色借鉴了抖音 LOGO 的风格

仅仅使用配色没有使用混合模式的好处在于，对于每一个文字的副本，有了更大的移动距离和可以处理的空间   
**使用滤镜生成文字融合效果**  
利用了模糊滤镜叠加对比度滤镜产生的融合效果。  
单独将两个滤镜拿出来，它们的作用分别是：  
1. filter: blur()： 给图像设置高斯模糊效果  
2. filter: contrast()： 调整图像的对比度

**文字与SVG**
利用 SVG 中几个和边框、线条相关的属性，来实现文字的线条动画  
- stroke-width：类比 css 中的 border-width，给 svg 图形设定边框宽度
- stroke：类比 css 中的 border-color，给 svg 图形设定边框颜色
- stroke-linejoin | stroke-linecap：设定线段连接处的样式
- stroke-dasharray：值是一组数组，没数量上限，每个数字交替表示划线与间隔的宽度；
- stroke-dashoffset：则是虚线的偏移量

**线条文字动画**  
用 stroke-* 相关属性，实现一个简单的线条文字动画  
``` 
<svg viewBox="0 0 400 200">
	<text x="0" y="70%"> Lorem </text>
</svg>	

svg text {
	animation: stroke 5s infinite alternate;
	letter-spacing: 10px;
	font-size: 150px;
}
@keyframes stroke {
	0% {
		fill: rgba(72, 138, 20, 0);
		stroke: rgba(54, 95, 160, 1);
		stroke-dashoffset: 25%;
		stroke-dasharray: 0 50%;
		stroke-width: 1;
	}
	70% {
		fill: rgba(72, 138, 20, 0);
		stroke: rgba(54, 95, 160, 1);
		stroke-width: 3;
	}
	90%,
	100% {
		fill: rgba(72, 138, 204, 1);
		stroke: rgba(54, 95, 160, 0);
		stroke-dashoffset: -25%;
		stroke-dasharray: 50% 0;
		stroke-width: 0;
	}
}
```
动画的核心就是，利用动态变化文字的 stroke-dasharray 和 stroke-dashoffset 形成视觉上的线条变换，动画的最后再给文字上色。

参考：  
[奇思妙想 CSS 文字动画](https://github.com/chokcoco/iCSS/issues/101)
