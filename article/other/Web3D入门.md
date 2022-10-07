# Web3D入门
## 概念篇：计算机图形相关知识
OpenGL 是API 、是规范。GPU 硬件厂商需要满足统一OpenGL规范。而 OpenGL ES (Open Graphics Library for Embedded Systems) 是 OpenGL 子集，专门针对手机等嵌入式设备而设计的。  
WebGL 是在 OpenGL ES 基础上建立的在 浏览器 跑起来的图形学标准，同理是浏览器厂商规范 ≈ 让JS 操作接口。光有规范是不够的，还要程序告诉 GPU 如何进行渲染。  
代码阉割版：有类C语言的着色器语言、有我们熟悉JS语言、有矩阵相乘等
```
 <canvas id="canvas"></canvas>
<script type="shader-source" id="vertexShader">
        precision mediump float; // 接收 JavaScript 传递过来的点的坐标（X, Y, Z）
        attribute vec3 a_Position;
        attribute vec4 a_Color; // 接收顶点颜色
        varying vec4 v_Color;
        uniform mat4 u_Matrix;
        void main(){
            gl_Position = u_Matrix * vec4(a_Position, 1);
            v_Color = a_Color; // 将顶点颜色插值处理传递给片元着色器
            gl_PointSize = 5.0;
        }
</script>
<script type="shader-source" id="fragmentShader">
        precision mediump float;
        varying vec4 v_Color;
        void main(){
            gl_FragColor = v_Color;  // 点的最终颜色。
        }
</script>
<script> //获取canvaslet 
        canvas = getCanvas('#canvas');
        //使用该着色器程序
        let program = createSimpleProgramFromScript(gl, 'vertexShader', 'fragmentShader');
        gl.useProgram(program);
        let positions = [
            -0.5, -0.5, 0.5, 1, 0, 0, 1,
            0.5, -0.5, 0.5, 1, 0, 0, 1,
            0.5, 0.5, 0.5, 1, 0, 0, 1,
            -0.5, 0.5, 0.5, 1, 0, 0, 1, .....
        ]
        // ----- 省略一些代码 -----
        function render() {
            //先绕 Y 轴旋转矩阵。
            matrix.rotationY(deg2radians(yAngle), dstMatrix);
            //再绕 X 轴旋转
            matrix.multiply(dstMatrix, matrix.rotationX(deg2radians(xAngle), tmpMatrix), dstMatrix);
            //模型投影矩阵。
            matrix.multiply(projectionMatrix, dstMatrix, dstMatrix)
            // ----- 省略一些代码 -----
            requestAnimationFrame(render);
        }      
    </script>
```

**总结**  
我们需要一个规范/接口告诉驱动如何 和 GPU 通信，这个规范/接口是 OpenGL，发展至今嵌入式设备崛起，OpenGL ES 也应运而生，WebGL 是基于 OpenGL ES 可以让其在浏览器上通过Javascript 调用的规范/接口，但WebGL门槛不低，要和GPU通信，就需要了解计算机图形学知识，那肯定也需要用到着色器，所以 Three.js 封装好成为三维引擎，也不用知道那么多底层知识，就可以创建 Web 3D。  

### 理解 GPU 设计模型
CPU像一个大的管道，处理每一项任务，任务重就占用在管道等时间长，虽然CPU运算能力很强，在处理图像是有局限的  
例如一张图片100px*100px就有10w个像素点构成，要对每个像素计算，如此设计模式对CPU压力会比较大，故有 GPU 结构去处理该场景  
GPU对每个像素进行计算，而且是相同的运算，这样并行计算的效率会更高  
GPU 计算能力不如 CPU，但是 GPU 人多力量大 (管子多，且管子只处理已知简单任务)

### 了解 图形渲染管线
任何用3D空间中表示的事物，在屏幕中都是2D像素数组，而WebGL/OpenGL 大部分工作也是把3D坐标转换为2D像素。这个过程叫做 图形渲染管线 Render Pipeline。  
那些个GPU 模型里的管子，有一堆原始图形数据 经过 一个管子后，最终输出至屏幕中的过程就是 图形渲染管线。  
图形渲染管线 Render Pipeline 被划分为几个阶段，跟咱们的ByteCycle 流水线一样，每个阶段会把上个阶段的输出作为输入，也可以理解是函数式编程 pipeline 模式。也就是说，每个阶段都有专门的函数 / 小程序去处理，函数 / 小程序 ≈ 着色器 (Shader) 。  
CPU 和 GPU 是通力合作的关系来渲染图像  
渲染管线抽象流程  

|  顶点着色器   | 3D坐标 转为 另一种3D坐标(后面会解释 从局部到世界坐标系)，并对顶点属性进行处理  |
|  ----  | ----  |
| 图元的装配  | 将顶点着色器输出的点作为输入，并绘制成图元形状 |
| 几何着色器  | 将 图元形状 构造成新的图元 或 其他形状 |
| 光栅化过程  | 把 图元 映射为 最终屏幕上相应的 像素 |
| 片段着色器  | 计算一个像素的最终颜色。例如一个立方体在灯光照射下会有阴影，这里也会将其处理 |
| 测试与混合  | 例如 有3D遮挡场景 或 物体是透明，在这个过程中就需要判断是否在该帧被丢弃 |

简单了解下GLSL 语言，类似C语言，以下是顶点着色器的例子：  
```
#version 300 es  #声明了着色器版本号 300 代表是 3.0 之后版本
in vec4 aPos ;
# in = 输入变量 浮点型向量vec4 变量名称 aPos
# eg: aPos = {1.0, 1.0, 1.0, 1.0}
void main()
{
    gl_Position = aPos ;
    # 顶点着色器的内置输出变量
}
```
**总结**  
基于浏览器，通过Javascript 来实现编程技术，能在 2D屏幕 上看到 3D效果  
- 基础能力：数学、物理 ....
- 能力支持：基于 GPU 图形渲染管线架构设计，在 Web端 通过 WebGL (-> OpenGL ES -> OpenGL) 和 着色器 (着色器GLSL 语言实现)，实现驱动能力。(任何语言实现都是以硬件为基础)

### 3D空间点 to 屏幕 2D二维点
一个复杂场景中，物体都需要软件建模，建模好后再将其放置到该场景中。  
当对每个物体建模的时候，物体本身是有自己的独立坐标系局部坐标系 Local Space，但物体放到场景中就有不同放置位置，所有物体共享同一个坐标系，叫世界坐标系 World Space  
在世界坐标系场景下，是从正面某个位置去观察物体，如果视角变化至沿着Z轴负方向看呢？又是另外一个画面，叫做视觉坐标系 View Space。
裁剪坐标系 Clip Space / DNC: 归一化处理，和 需要判断哪个片段需要展示在屏幕内  
屏幕坐标系 Screen Space：根据裁剪坐标系计算，再转换为屏幕坐标。最后将数据传到光栅器  

局部空间->世界空间，涉及 矩阵的平移、缩放、旋转  
缩放：代表多少倍，缩放S1、S2、S3 倍数  
世界空间 -> 视觉空间，构建 线性变换矩阵  
任何方位观察到的物体都是不同的，从A 位置 变换至 B 位置，只要知道 变换前后的基向量，就能知道 运动至哪里，方法通过 矩阵相乘  
矩阵向量乘积: 变换后的基向量 * 未变化前的位置 (x, y) = 基向量变换后新(x', y')  

视觉空间 -> 裁剪空间 -> 屏幕空间  
- 将 3D 点 表示到 2D 点， 投影 -> 点积 (实际上会更复杂些) ....
- 再将能视觉展示的空间展示，不能展示的被剪裁掉
- 剪裁后点位，会归一化处理，保证交付给发光二极管


## 实践篇：用 Three.js 入个门
### 3D 建模概念必备
如果你是个大导演，有一天你想请 安琪拉大宝贝儿 来北京 献歌一曲，

- 要有地点 Scene 场景 ，选择 人民大会堂作为 舞台吧；
- 要有灯光 Light 灯光, 才能让观众看到 安琪拉大宝贝儿 唱歌；
- 关于 安琪拉大宝贝 作为 模型，来之前要保养一下，皮肤看起来吹弹可破 材质 Material；
- XXX 大品牌疯狂赞助，并要求她穿上新一季 服饰 和 配上妆发 贴图与纹理 Texture;
- 一切准备就绪后，N个机器 Camera 相机 360 度无死角的拍摄，她唱 XXX歌曲。
你刚在脑海里构建出来的画面 ≈ 渲染器 Render

### 实践代码走一波
STEP1: 创建舞台 和 相机，并渲染至页面上  
```
import * as THREE from 'three'

class ThreeDemo {
  constructor () {
    this.width = window.innerWidth
    this.height = window.innerHeight
    this.aspectRatio = this.width / this.height

    // 创建场景
    this.scene = null
    // 创建相机
    this.camera = null
    // 创建灯光
    this.light = null
    // 创建模型
    this.model = null
    // 创建材质
    this.material = null
    // 创建纹理
    this.texture = null
    // 创建渲染
    this.renderer = null
  }

  init () {
    this.createScene() // 创建舞台 和 相机
    this.createRenderer() // 创建渲染
    document.body.appendChild(this.renderer.domElement) // 渲染至页面上
    
    const render = () => {
      this.renderer.render(this.scene, this.camera) // 渲染场景
      requestAnimationFrame(render)
    }
    render()
    
    this.axesHelper()
  }

  createScene () {
    // ====== 搭建个舞台 ======
    this.scene = new THREE.Scene() 
    this.scene.fog = new THREE.Fog(0x090918, 1, 600)
    
    // ====== 搭建相机 (模拟人视角去看景象) PerspectiveCamera = 透视相机 ======
    this.camera = new THREE.PerspectiveCamera(
      75, // 视角
      this.aspectRatio, // 纵横比
      0.1, // nearPlane 近平面
      2000 // farPlane 远平面
    )
    // 设置相机位置
    this.camera.position.set(10, 10, 10) // x, y, z
    // 更新摄像头宽高比例
    this.camera.aspect = this.aspectRatio
    // 更新摄像头的矩阵
    this.camera.updateProjectionMatrix()

    // 将相机放到舞台上
    this.scene.add(this.camera)
  }

  createRenderer () {
    this.renderer = new THREE.WebGLRenderer({ antialias: true })
    this.renderer.outputEncoding = THREE.sRGBEncoding
    // 设置渲染器宽高
    this.renderer.setSize(this.width, this.height)
    this.renderer.setClearColor(this.scene.fog.color)

    // 屏幕变化 更新渲染 (相机视角变化 和 渲染器变化)
    window.addEventListener('resize', () => {
      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()
      this.renderer.setSize(window.innerWidth, window.innerHeight)
    })
  }
  
  // 辅助坐标系
  axesHelper () {
    const axesHelper = new THREE.AxesHelper(5)
    this.scene.add(axesHelper)
  }
}

const instance = new ThreeDemo()
instance.init()
```
STEP2: 加模型 和 灯光  
```
// 加入环境光
// 环境光会均匀的照亮场景中的所有物体
this.light = new THREE.AmbientLight(0x404040) // soft white light
this.scene.add(this.light)

// 场景中添加球
const geometry = new THREE.BoxGeometry(2, 2, 2)
const geometry_material = new THREE.MeshStandardMaterial({ color: 0xaafabb })
instance.scene.add(new THREE.Mesh(geometry, geometry_material))
```
无光照 (无环境光加入)  
这里不仅加入环境光，还加入了平行光，即平行光是沿着特定方向发射的光  
```
createLight () {
    // 环境光会均匀的照亮场景中的所有物体
    this.light = new THREE.AmbientLight(0x404040) // soft white light
    this.scene.add(this.light)

    // 平行光是沿着特定方向发射的光
    this.directionalLight = new THREE.DirectionalLight( 0xffffff, 0.6 )
    this.directionalLight.position.set(0, 5, 5)
    
    this.scene.add(this.directionalLight)
  }
```
STEP3: 贴膜 (材质和纹理)  
```
// 场景中添加立方体
const geometry = new THREE.BoxGeometry(2, 2, 2)
const geometry_material = new THREE.MeshStandardMaterial({ 
  map: textureLoader.load('../public/textures/RoofTilesTerracotta004/RoofTilesTerracotta004_COL_1K.jpg'),
  aoMap: textureLoader.load('../public/textures/RoofTilesTerracotta004/RoofTilesTerracotta004_AO_1K.jpg'),
  alphaMap: textureLoader.load('../public/textures/RoofTilesTerracotta004/RoofTilesTerracotta004_AO_1K.jpg'),
  normalMap: textureLoader.load('../public/textures/RoofTilesTerracotta004/RoofTilesTerracotta004_NRM_1K.png'),
  transparent: true,
  roughness: 0,
})
const model = new THREE.Mesh(geometry, geometry_material)
instance.scene.add(model)
```
小结:  
1. 先将 Scene 场景、Light 灯光、Camera 相机 设置好，并将其通过渲染器 Render 渲染至页面上
2. 确定好 模型 穿上 材质 Material 和 贴图 Texture 后，并设定好该模型位置，再添加至场景Scene 场景 中，即可得到3D物体啦。

原文:  
[Web 3D 从入门到跑路](https://mp.weixin.qq.com/s/TI6phB5E4_TZUzac7vZHEQ)