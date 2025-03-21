# 练习 WebGL

## 笔记

## GLSL

- `gl_Position`：内置变量，用来设置顶点坐标。
- `gl_PointSize`：内置变量，用来设置顶点大小。
- `vec2`：2 维向量容器，可以存储 2 个浮点数。
- `gl_FragColor`：内置变量，用来设置像素颜色。
- `vec4`：4 维向量容器，可以存储 4 个浮点数。
- `precision`：精度设置限定符，使用此限定符设置完精度后，之后所有该数据类型都将沿用该精度，除非单独设置。
- `varying`：变量声明，此声明的变量具备从顶点着色器向片元着色器传递插值数据的能力。在 OpenGL ES 3.0+ 或 WebGL 2.0 版本之后，此声明被 `out` 和 `in` 替代。
- 运算符：向量的对应位置进行运算，得到一个新的向量。
  - `vec * 浮点数`： vec2(x, y) \* 2.0 = vec(x \* 2.0, y \* 2.0)。
  - `vec2 * vec2`：vec2(x1, y1) \* vec2(x2, y2) = vec2(x1 \* x2, y1 \* y2)。
  - 加减乘除规则基本一致。但是要注意一点，如果参与运算的是两个 vec 向量，那么这两个 vec 的维数必须相同。

## WebGL

- drawArrays: 用指定的图元进行绘制
- gl.POINTS: 将绘制图元类型设置成点图元
- 三角形图元分类
  - gl.TRIANGLES：基本三角形
  - gl.TRIANGLE_STRIP：三角带
  - gl.TRIANGLE_FAN：三角扇
- 线段图元分类
  - LINES：基本线段
  - LINE_STRIP：带状线段
  - LINE_LOOP：环状（闭环）线段
- 连接着色器程序
  - `createShader`：创建着色器对象
  - `shaderSource`：提供着色器源码
  - `compileShader`：编译着色器对象
  - `createProgram`：创建着色器程序
  - `attachShader`：绑定着色器对象
  - `linkProgram`：链接着色器程序
  - `useProgram`：启用着色器程序
- 给着色器传递数据
  - `getAttribLocation`：找到着色器中的 attribute 变量地址
  - `getUniformLocation`：找到着色器中的 uniform 变量地址
  - `vertexAttrib2f`：给 attribute 变量传递两个浮点数
  - `uniform4f`：给 uniform 变量传递四个浮点数。
- 使用缓冲区传递数据
  - `createBuffer`：创建 buffer
  - `bindBuffer`：绑定某个缓冲区对象为当前缓冲区
  - `bufferData`：往缓冲区中复制数据
  - `enableVertexAttribArray`：启用顶点属性
  - `vertexAttribPointer`：设置顶点属性从缓冲区中读取数据的方式

### 单 Buffer 存储多个信息数据的优势

将多个信息（如顶点位置、颜色、纹理坐标等）集中到一个 Buffer 中传递，主要通过 **结构化存储数据** 实现高效渲染，具体优势如下:

1. 减少 API 调用开销
   - 传统方式：每个顶点属性（如位置、颜色）需单独绑定 Buffer 并调用 gl.vertexAttribPointer ，导致在数据存入缓冲区时需要多次状态切换状态机（bindBuffer）。
   - 集中存储：通过 单 Buffer 多属性 的布局（使用 stride 和 offset 参数），仅需一次 bindBuffer 和 vertexAttribPointer 调用，显著降低 WebGL 状态机切换的 CPU 开销。
2. 提升 GPU 数据吞吐效率
   - 内存连续性：集中存储的数据在显存中连续排列，减少 GPU 访问碎片化数据时的缓存未命中问题。
   - 批量传输：通过 gl.bufferData 一次性写入全部数据（如顶点 + 颜色 + UV），避免多次数据传输的等待时间。
3. 优化复杂模型渲染性能
   - 合批（Batching）：同一 Buffer 中的多属性数据可支持单次 gl.drawArrays 调用绘制复杂模型，避免因分批次绘制产生的性能瓶颈。
   - 实例化（Instancing）：结合 gl.vertexAttribDivisor ，可复用 Buffer 中的结构化数据高效渲染大量重复对象（如粒子系统）。
4. 简化数据管理
   统一生命周期：所有顶点相关属性共享同一 Buffer 对象，避免多 Buffer 的创建/销毁带来的资源管理复杂度。
   兼容 WebGL 2.0 优化：支持 VAO（Vertex Array Object） 将 Buffer 配置与着色器属性绑定固化，进一步提升多模型切换效率。
5. 典型应用场景
   动态数据更新：如骨骼动画中顶点位置与权重的实时更新，通过单 Buffer 的 gl.bufferSubData 局部更新更高效。
   流式渲染：在实时生成顶点数据（如地形、流体模拟）时，单 Buffer 的双缓冲（Ping-Pong）机制可避免卡顿。
   数据布局示例（伪代码）
   // 单个 Buffer 存储位置（vec3） + 颜色（vec3） + UV（vec2）
   const data = new Float32Array([
   // 顶点 1: x, y, z, r, g, b, u, v
   0, 0, 0, 1, 0, 0, 0.5, 0.5,
   // 顶点 2: ...
   ]);

// 配置属性
gl.vertexAttribPointer(positionLoc, 3, gl.FLOAT, false, 8*4, 0); // 步长 8*4，偏移 0
gl.vertexAttribPointer(colorLoc, 3, gl.FLOAT, false, 8*4, 3*4); // 偏移 3*4
gl.vertexAttribPointer(uvLoc, 2, gl.FLOAT, false, 8*4, 6*4); // 偏移 6*4
注意事项
数据类型对齐：需确保步长（stride）与硬件内存对齐要求匹配（如 4 字节对齐）。
移动端优化：部分低端 GPU 对多属性 Buffer 的访问效率敏感，需实测性能差异。
通过集中存储数据，WebGL 应用可在复杂场景中实现更高的帧率和更流畅的交互体验。
