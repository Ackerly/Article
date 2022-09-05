# 利用噪声构建美妙的 CSS 图形
假设我们实现一个 10x10 的格子,利用一些随机效果，给它随机添加不同的颜色,随机填充了每一个格子的颜色，看着有那么点意思，但是这只是一幅杂乱无章的图形，并没有什么艺术感。  
这里的随机属于完全随机，属于一种白噪声。  
**什么是白噪声？**  
噪声（Noise）实际上就是一个随机数生成器  
噪声的基础是随机数，譬如我们给上述的图形每一个格子添加了一个随机颜色，得到的就是一幅杂乱无章的图形块，没有太多美感可言  
> 白噪声或白杂讯，是一种功率谱密度[1]为常数的随机信号。换句话说，此信号[2]在各个频段上的功率谱密度是一样的，由于白光是由各种频率（颜色）的单色光混合而成，因而此信号的这种具有平坦功率谱[3]的性质被称作是“白色的”，此信号也因此被称作白噪声

观察现实生活中的自然噪声，它们不会长成上面的样子。例如木头的纹理、山脉的起伏，它们的形状是趋于分形状（fractal）的，即包含了不同程度的细节，这些随机的成分并不是完全独立的，它们之间有一定的关联。和显然，白噪声没有做到这一点。  
**柏林噪声**  
Perlin 噪声 ( Perlin noise ) 指由 Ken Perlin 发明的自然噪声生成算法  
利用柏林噪声随机效果产生的图形，彼此之间并非毫无关联，它们之间的变化是连续的，彼此之间并没有发生跳变。这种随机效果，类似于自然界中的随机效果，譬如上面说的，木头纹理、山脉起伏的变化。  
噪声实际上就是一个随机数生成器。而这里：  
1. 白噪声的问题在于，它实在太过于随机，毫无规律可言
2. 而柏林噪声基于随机，并在此基础上利用缓动曲线进行平滑插值，使得最终得到噪声效果更加趋于自然

具体的实现方式  
``` 
// This code implements the algorithm I describe in a corresponding SIGGRAPH 2002 paper.
// JAVA REFERENCE IMPLEMENTATION OF IMPROVED NOISE - COPYRIGHT 2002 KEN PERLIN.

public final class ImprovedNoise {
   static public double noise(double x, double y, double z) {
      int X = (int)Math.floor(x) & 255,                  // FIND UNIT CUBE THAT
          Y = (int)Math.floor(y) & 255,                  // CONTAINS POINT.
          Z = (int)Math.floor(z) & 255;
      x -= Math.floor(x);                                // FIND RELATIVE X,Y,Z
      y -= Math.floor(y);                                // OF POINT IN CUBE.
      z -= Math.floor(z);
      double u = fade(x),                                // COMPUTE FADE CURVES
             v = fade(y),                                // FOR EACH OF X,Y,Z.
             w = fade(z);
      int A = p[X  ]+Y, AA = p[A]+Z, AB = p[A+1]+Z,      // HASH COORDINATES OF
          B = p[X+1]+Y, BA = p[B]+Z, BB = p[B+1]+Z;      // THE 8 CUBE CORNERS,

      return lerp(w, lerp(v, lerp(u, grad(p[AA  ], x  , y  , z   ),  // AND ADD
                                     grad(p[BA  ], x-1, y  , z   )), // BLENDED
                             lerp(u, grad(p[AB  ], x  , y-1, z   ),  // RESULTS
                                     grad(p[BB  ], x-1, y-1, z   ))),// FROM  8
                     lerp(v, lerp(u, grad(p[AA+1], x  , y  , z-1 ),  // CORNERS
                                     grad(p[BA+1], x-1, y  , z-1 )), // OF CUBE
                             lerp(u, grad(p[AB+1], x  , y-1, z-1 ),
                                     grad(p[BB+1], x-1, y-1, z-1 ))));
   }
   static double fade(double t) { return t * t * t * (t * (t * 6 - 15) + 10); }
   static double lerp(double t, double a, double b) { return a + t * (b - a); }
   static double grad(int hash, double x, double y, double z) {
      int h = hash & 15;                      // CONVERT LO 4 BITS OF HASH CODE
      double u = h<8 ? x : y,                 // INTO 12 GRADIENT DIRECTIONS.
             v = h<4 ? y : h==12||h==14 ? x : z;
      return ((h&1) == 0 ? u : -u) + ((h&2) == 0 ? v : -v);
   }
   static final int p[] = new int[512], permutation[] = { 151,160,137,91,90,15,
   131,13,201,95,96,53,194,233,7,225,140,36,103,30,69,142,8,99,37,240,21,10,23,
   190, 6,148,247,120,234,75,0,26,197,62,94,252,219,203,117,35,11,32,57,177,33,
   88,237,149,56,87,174,20,125,136,171,168, 68,175,74,165,71,134,139,48,27,166,
   77,146,158,231,83,111,229,122,60,211,133,230,220,105,92,41,55,46,245,40,244,
   102,143,54, 65,25,63,161, 1,216,80,73,209,76,132,187,208, 89,18,169,200,196,
   135,130,116,188,159,86,164,100,109,198,173,186, 3,64,52,217,226,250,124,123,
   5,202,38,147,118,126,255,82,85,212,207,206,59,227,47,16,58,17,182,189,28,42,
   223,183,170,213,119,248,152, 2,44,154,163, 70,221,153,101,155,167, 43,172,9,
   129,22,39,253, 19,98,108,110,79,113,224,232,178,185, 112,104,218,246,97,228,
   251,34,242,193,238,210,144,12,191,179,162,241, 81,51,145,235,249,14,239,107,
   49,192,214, 31,181,199,106,157,184, 84,204,176,115,121,50,45,127, 4,150,254,
   138,236,205,93,222,114,67,29,24,72,243,141,128,195,78,66,215,61,156,180
   };
   static { for (int i=0; i < 256 ; i++) p[256+i] = p[i] = permutation[i]; }
}
```
**利用 CSS-doodle，在 CSS 中利用柏林噪声**  
CSS-doodle 它是一个基于 Web-Component 的库。允许我们快速的创建基于 CSS Grid 布局的页面，并且提供各种便捷的指令及函数（随机、循环等等），让我们能通过一套规则，得到不同 CSS 效果。  
简单解释下：  
1. css-doodle 是基于 Web-Component 封装的，基本所有的代码都写在 <css-doodle> 标签内，当然也可以写一些原生 CSS/JavaScript 辅助
2. 使用 grid="10x10" 即可生成一个 10x10 的 Grid 网格，再配合 @size: 50vmin，表示生成一个宽高大小为 50vmin 的 10x10 Grid 网格布局，其中 gap: 1px 表示 Gird 网格布局的 gap
3. 整个代码的核心部分即是 background: hsl(@rn(255, 1, 2), @rn(10%, 90%), @rn(10%, 90%))，这里即表示对每个 grid item 赋予背景色，其中 @rn()，就是最核心的部分，利用了柏林噪声算法，有规律的将背景色 map 到每一个 grid 上

@rn() 的实现使用的就是柏林噪声的实现。同时，函数相当于是类似 p5.js 里面的 noise 函数同时做了 map，map 到前面函数参数设定的 from 到 to 范围内。  
@rn() 柏林噪声随机会根据 Grid 网格，Map 到每一个网格上，使之相邻的 Grid item 之间的值，存在一定的关联。这就使得，我们利用它创造出来的图形，会具备一定的规律。  

源码的实现  
``` 
rn({ x, y, context, position, grid, extra, shuffle }) {
      let counter = 'noise-2d' + position;
      let [ni, nx, ny, nm, NX, NY] = last(extra) || [];
      let isSeqContext = (ni && nm);
      return (...args) => {
        let {from = 0, to = from, frequency = 1, amplitude = 1} = get_named_arguments(args, [
          'from', 'to', 'frequency', 'amplitude'
        ]);

        if (args.length == 1) {
          [from, to] = [0, from];
        }
        if (!context[counter]) {
          context[counter] = new Perlin(shuffle);
        }
        frequency = clamp(frequency, 0, Infinity);
        amplitude = clamp(amplitude, 0, Infinity);
        let transform = [from, to].every(is_letter) ? by_charcode : by_unit;
        let t = isSeqContext
          ? context[counter].noise((nx - 1)/NX * frequency, (ny - 1)/NY * frequency, 0)
          : context[counter].noise((x - 1)/grid.x * frequency, (y - 1)/grid.y * frequency, 0);
        let fn = transform((from, to) => map2d(t * amplitude, from, to, amplitude));
        let value = fn(from, to);
        return push_stack(context, 'last_rand', value);
      };
    },
```
语法大概是 @rn(from, to, frequency, amplitude)，其中 from、to 表示随机范围，而 frequency 表示噪声的频率，amplitude 表示噪声的振幅。这两个参数可以理解为控制随机效果的频率和幅度。  
其中 new Perlin(shuffle) 即运用到了柏林噪声算法。  
**Show Time**  
可以再添加上随机的 scale()、以及 skew()。代码是这样的： 
``` 
<css-doodle grid="20">
    :doodle {
        grid-gap: 1px;
        width: 600px; height: 600px;
    }
    background: hsl(@r(360), 80%, 80%);
    transform: 
        scale(@r(1.1, .3, 3)) 
        skew(@r(-45deg, 45deg, 3));
</css-doodle>

html,
body {
    width: 100%;
    height: 100%;
    background-color: #000;
}
```
上述代码表示的是一个 20x20 的 Grid 网格，每个 Grid item 都设置了完全随机的背景色、scale() 以及 skew()。当然，这里我们用的是 @r()而不是 @rn()，每个格子的每个属性的随机，没有任何的关联
在上述代码的基础上，将普通的完全随机，改为柏林噪声随机 @rn()：
```
<css-doodle grid="20">
    :doodle {
        grid-gap: 1px;
        width: 600px; height: 600px;
    }
    background: hsl(@rn(360), 80%, 80%);
    transform: 
        scale(@rn(1.1, .3, 3)) 
        skew(@rn(-45deg, 45deg, 3));
</css-doodle>
```
此时，就能得到完全不一样的效果,这是由于每个 Grid item 的随机效果，都基于它们在 Grid 布局中的位置，彼此存在关联，这就是柏林噪声随机的效果。  
再添加上 hue-rotate 动画：  
``` 
html,
body {
    width: 100%;
    height: 100%;
    background-color: #000;
    animation: change 10s linear infinite;
}
@keyframes change {
    10% {
        filter: hue-rotate(360deg);
    }
}

```

可以把柏林噪声随机应用在各种属性上，我们可以放飞想象，去尝试各种不一样的搭配。下面这个， 就是把柏林噪声运用在点阵定位上：  
``` 
<css-doodle grid="30x30">
    :doodle {
        @size: 90vmin;
        perspective: 10px;
    }
    position: absolute;
    top: 0;
    left: 0;
    width: 2px;
    height: 2px;
    border-radius: 50%;
    top: @rn(1%, 100%, 1.5);
    left: @rn(1%, 100%, 1.5);
    transform: scale(@rn(.1, 5, 2));
    background: hsl(@rn(1, 255, 3), @rn(10%, 90%), @rn(10%, 90%));
</css-doodle>
```  
亦或者配合运用在 transform: rotate() 上：  
``` 
<css-doodle grid="20x5">
    @place-cell: center;
    @size: calc(@i * 1.5%);
    :doodle {
        width: 60vmin; 
        height: 60vmin;
    }
    z-index: calc(999 - @i);
    border-radius: 50%;
    border: 1px @p(dashed, solid, double) hsl(@rn(255), 70%, @rn(60, 90%));
    border-bottom-color: transparent;
    border-left-color: transparent;
    transform: 
        rotate(@rn(-720deg, 720deg))
        scale(@rn(.8, 1.2, 3));
</css-doodle>

```

原文: 
[利用噪声构建美妙的 CSS 图形](https://mp.weixin.qq.com/s/l80muJPaGgbLyeljZ5mK2g)
