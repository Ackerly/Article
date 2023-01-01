# SpriteJS：图形库造轮子
**原始需求：和渲染无关**  
D3 关心的是数据的组织，它并不关心数据最终渲染的结果，但是，D3 的数据组织形式是基于树状结构的，因为它天然契合树状结构的渲染形式。正因为如此，所以一般来说，D3 的官方例子都是用 DOM 或 SVG 渲染，这是因为基于 DOM 树的渲染和 D3 的树状数据组织形式是绝配。  

## 设计一个图形系统的“骨架”
**坐标系的选择**  
在图形系统的设计中，首先要确定默认坐标系。理论上讲，任何一种直角坐标系，甚至非直角坐标系（比如极坐标）都可以作为默认坐标系，在欧式几何中，这些坐标系都可以自由转换。不过，考虑与 DOM 的一致性，采用浏览器默认的坐标系是一个极好的选择。  
对于 WebGL 渲染来说，需要将顶点坐标转换成 WebGL 坐标，在这里，采用根据 canvas 的坐标动态设置 projectionMatrix 即可：  
``` 
updateResolution() {
  const {width, height} = this.canvas;
  const m1 = [ // translation
    1, 0, 0,
    0, 1, 0,
    -width / 2, -height / 2, 1,
  ];
  const m2 = [ // scale
    2 / width, 0, 0,
    0, -2 / height, 0,
    0, 0, 1,
  ];
  const m3 = mat3(m2) * mat3(m1);
  this.projectionMatrix = m3;
  if(this[_glRenderer]) {
    this[_glRenderer].gl.viewport(0, 0, width, height);
  }
}
```
``` 
attribute vec3 a_vertexPosition;
attribute vec3 a_vertexTextureCoord;
varying vec3 vTextureCoord;
uniform mat3 viewMatrix;
uniform mat3 projectionMatrix;
void main() {
  gl_PointSize = 1.0;
  vec3 pos = projectionMatrix * viewMatrix * vec3(a_vertexPosition.xy, 1.0);
  gl_Position = vec4(pos.xy, 1.0, 1.0);
  vTextureCoord = a_vertexTextureCoord;
}
```
**图层、树形结构与元素类型**  
SpriteJS 用 Scene 表示场景，一个 Layer 表示一个图层，在这里，我的设计是一个 Layer 对应一个画布，即默认每个 Layer 都是独立的 Canvas 元素。这么做有优点也有缺点，是一种设计上的取舍。  
优点是，每个 Layer 彼此独立，Layer 间不必考虑绘制次序，可以充分利用 WebWorker 这样的多线程来并行绘制，而且逻辑上比较简单，如果需要在多层响应事件，只需要注意事件处理的次序。缺点是如果分多层绘制，有可能产生较多 Canvas 对象实例，比较耗内存。

原文:  
[SpriteJS：图形库造轮子的那些事儿](https://mp.weixin.qq.com/s/-L7BVHxP1HiS_3NS3WGUtQ)
