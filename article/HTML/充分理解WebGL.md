# 充分理解WebGL
WebGL 的技术栈和传统的 Web 前端技术有极大的差别。相对而言，传统 Web 前端使用的 API 比较高级，不存在太多需要理解的底层原理和概念，而 WebGL 的核心是 OpenGL，它是 OpenGL 在 Web 上的实现。OpenGL 是通过操作 GPU 来完成图形绘制渲染的，因此它的 API 相对比较底层，使用起来较为繁琐，这使得一些习惯于前端开发的工程师很难适应，所以就会觉得学习门槛较高。  
要理解和学会 WebGL，并没有那么困难，我们只需要理解一下 GPU，了解它与 CPU 的不同点，然后再理解运行 GPU 代码的语言——glsl，了解着色器的基本概念和用法，就可以轻松理解 WebGL 的本质原理，然后再花一点时间和耐心，慢慢学习 WebGL 的 API，就可以掌握 WebGL 这门技术了。  
CPU 就像是一个管道，数据（图中的箱子）从左侧输入，在 CPU 中完成处理，然后从右侧输出。CPU 是由多个这样的管道构成的，每个管道我们叫做一个 CPU 内核，如果你打开你电脑的操作系统，查看本机信息，你可能会看到类似这样的信息：2.6 GHz 六核Intel Core i7，这里的六核，你可以理解成有 6 个这样的管道，因此可以同时处理 6 个任务。  
CPU 的工作能力，与管道本身的处理速度（频率）和管道的数量（内核数）有关系，频率越高，那么运算处理单一任务的速度就越快，内核数越多，那么能同时并行处理的任务数就越多。  
现代计算机的 CPU 运算能力很强，但是它也有局限性，对于某些场景，它并不擅长，比如图形渲染。  
计算机图像是由像素构成，所谓像素，可以简单理解为最终呈现在显示设备上的一个 1x1 的颜色小方块。  
现在的显示设备非常先进，可以用非常多的像素小方块来精确构图。前端的 CSS 中的px单位，就是像素单位，一张800px长、600px宽的图片，逻辑上是由600*800，也就是 48 万个像素点构成的。如果要对这张图片的像素进行计算，用 CPU 来运算的话，单核 CPU 需要处理 48 万个微小任务  
不是说 CPU 不能完成这样的处理，每一个像素的计算可能是非常简单的（只是处理一下颜色），但是数量太多，对 CPU 这样的结构也会造成负担。因此，在这个时候，另外一种高并发结构，也就是 GPU 就登场了。  
与 CPU 不同，GPU 可以看成是由数量非常多的微小管道构成的结构，每一个管道恰好可以处理“一粒沙子”，这样，如果对于一张 600 像素 x800 像素的图片，有 48 万个管道组成的 GPU，就可以同时处理这 48 万个像素点了！事实上，GPU 几乎就是这样做的。  
WebGL 利用 JavaScript 来准备数据，将数据通过共享数据结构（ArrayBuffer) 传递给 GPU，由 GPU 进行着色处理，然后再将处理后的数据输出到帧缓冲区，最后再渲染出来。  
这其中的关键是头两个步骤，也就是准备数据和着色处理，其中准备数据一般是通过 JavaScript 的类型数组（TypedArray），着色处理是通过 WebGL Program 执行一种特殊的 glsl 语言来实现。在 WebGL 中，着色阶段通常分成两步，分别是顶点着色和片段着色。  
WebGL 的 API 比较底层，所以操作起来比较繁琐，但是也有许多 JS 库，帮我们封装了基本的操作  
**两个着色器**  
``` 
const canvas = document.querySelector('canvas');
const renderer = new GlRenderer(canvas, {webgl2: true});

const fragment = `#version 300 es
precision highp float;
out vec4 FragColor;
void main() {
  FragColor = vec4(1, 0, 0, 1);
}
`;
const program = renderer.compileSync(fragment);
renderer.useProgram(program);
renderer.render();
```
JS的部分很好理解，但是其中有一段fragment变量中的字符串，是需要我们关注的部分，没错，它就是两个着色器之一的片段着色器代码。  
这段代码是用 glsl 语言写的，但是并不难理解，第一句 #version 300 es 声明这个着色器是 webgl 2.0 版本的着色器，浏览器目前同时支持 webgl 1.0 和 webgl 2.0 两个版本，它们有一些差异，但差异不是很大。  
第二句precision highp float;表示设置浮点数精度为高精度，这个可以暂时忽略  
第三句out vec4 FragColor 声明FragColor是输出变量，它的类型是vec4，是一个四维向量，用来表示一个 RGBA 颜色值，它与 CSS 的颜色区别是，CSS 的 RGB 值是 0 到 255，Alpha 值是 0 到 1，但是在着色器里面，RGBA 的值都是从 0 到 1。  
void main是主函数，vec4(1, 0, 0, 1) 表示将 FragColor 设置为红色。  
GPU 是并行计算的，也就是说，这段着色器代码，是每个像素都并行执行（严格来说是根据图元装配的结果来执行对应区域的像素，但在这里我们先略过这一点），因此，整个画布中，所有的像素点，你可以认为都执行了一遍上面的着色器代码，而且是同时、并行执行的，由于是无差别执行的设置颜色为红色，因此整块画布都成为了红色的。  
可以修改一下上面例子的代码，看一下给每个像素设置不同颜色的情况：  
``` 
const canvas = document.querySelector('canvas');
const renderer = new GlRenderer(canvas, {webgl2: true});

const fragment = `#version 300 es
precision highp float;
out vec4 FragColor;
uniform vec2 resolution;
void main() {
  vec2 st = gl_FragCoord.xy / resolution;
  FragColor = vec4(st, 0, 1);
}
`;
const program = renderer.compileSync(fragment);
renderer.useProgram(program);
renderer.uniforms.resolution = [canvas.width, canvas.height];
renderer.render();
```
修改了上面的代码，在着色器中增加了一个uniform vec2 resolution，这是声明了一个 resolution 变量，它的类型是二维向量，我们通过renderer.uniforms.resolution将画布的宽高传入。  
gl_FragCoord.xy是一个内置变量，它表示当前渲染的像素在画布内的坐标，左下角是[0,0]，右上角是[width,height]，所以gl_FragCoord.xy / resolution可以将坐标值“归一”（即将值限制到 0~1 区间，这是一种在写着色器的时候经常使用的数学技巧）。  
接下来我们就来了解另一个着色器：顶点着色器。  
``` 
const canvas = document.querySelector('canvas');
const renderer = new GlRenderer(canvas, {webgl2: true});

const fragment = `#version 300 es
precision highp float;
out vec4 FragColor;
void main() {
  FragColor = vec4(1, 0, 0, 1);
}
`;
const program = renderer.compileSync(fragment);
renderer.useProgram(program);
renderer.setMeshData([
  {
    positions: [[0, 1, 0], [-1, -1, 0], [1, -1, 0]],
    cells: [[0, 1, 2]],
  },
]);
renderer.render();
```
我们没有添加顶点着色器，只是给 renderer 设置了一些数据，在数据里我们指定了三个顶点，它们的三维坐标分别是[0, 1, 0], [-1, -1, 0], [1, -1, 0]，这里我们没有用到 z 轴，所以 z 保持 0，x 和 y 在 WebGL 中，默认范围是从-1 到 1，所以 WebGL 的平面坐标系原点在中心，左下角是[-1,-1]，右下角是[1, -1]，右上角是[1, 1]，左上角是[-1,1]。我们设置了 position 这三个顶点之后，绘制在画面上的就变成了一个三角形。  
没有指定顶点着色器，是因为 gl-renderer 有默认的顶点着色器，代码如下：  
``` 
#version 300 es
precision highp float;
precision highp int;

in vec3 a_vertexPosition;

void main() {
  gl_PointSize = 1.0;
  gl_Position = vec4(a_vertexPosition, 1);
}
```
也可以指定顶点着色器，继续修改代码：  
``` 
const canvas = document.querySelector('canvas');
const renderer = new GlRenderer(canvas, {webgl2: true});

const vertex = `#version 300 es
precision highp float;
precision highp int;

in vec3 a_vertexPosition;

void main() {
  gl_PointSize = 1.0;
  gl_Position = vec4(0.5 * a_vertexPosition, 1);
}`;

const fragment = `#version 300 es
precision highp float;
out vec4 FragColor;
void main() {
  FragColor = vec4(1, 0, 0, 1);
}
`;
const program = renderer.compileSync(fragment, vertex);
renderer.useProgram(program);
renderer.setMeshData([
  {
    positions: [[0, 1, 0], [-1, -1, 0], [1, -1, 0]],
    cells: [[0, 1, 2]],
  },
]);
renderer.render();
```
上面的代码我们指定了顶点着色器，在它里面我们把a_vertexPosition，也就是我们传入的顶点坐标给乘以了 0.5，所以最终绘制出来的三角形周长就是原来的 1/2。  
**为什么是三角形**  
因为三角形是 WebGL 的基本图元，WebGL 支持点、线、三角形等基本图元。下面的代码更换了图元。  
``` 
const canvas = document.querySelector('canvas');
const renderer = new GlRenderer(canvas, {webgl2: true});

const vertex = `#version 300 es
precision highp float;
precision highp int;

in vec3 a_vertexPosition;

void main() {
  gl_PointSize = 1.0;
  gl_Position = vec4(0.5 * a_vertexPosition, 1);
}`;

const fragment = `#version 300 es
precision highp float;
out vec4 FragColor;
void main() {
  FragColor = vec4(1, 0, 0, 1);
}
`;
const program = renderer.compileSync(fragment, vertex);
renderer.useProgram(program);
renderer.setMeshData([
  {
    mode: 'LINE_STRIP',
    positions: [[0, 1, 0], [-1, -1, 0], [1, -1, 0]],
    cells: [[0, 1, 2, 0]],
  },
]);
renderer.render();
```

参考:
[充分理解WebGL](https://mp.weixin.qq.com/s/f5utQ9wS90zhfbXs3NJ3Eg)
