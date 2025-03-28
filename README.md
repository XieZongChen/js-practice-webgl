# 练习 WebGL

## 笔记

## GLSL 着色器

- 数据类型
  - `vec2`：2 维向量容器，可以存储 2 个浮点数
  - `vec4`：4 维向量容器，可以存储 4 个浮点数
- 运算符：向量的对应位置进行运算，得到一个新的向量
  - `vec * 浮点数`： vec2(x, y) \* 2.0 = vec(x \* 2.0, y \* 2.0)
  - `vec2 * vec2`：vec2(x1, y1) \* vec2(x2, y2) = vec2(x1 \* x2, y1 \* y2)
  - 加减乘除规则基本一致。但是要注意一点，如果参与运算的是两个 vec 向量，那么这两个 vec 的维数必须相同
- 修饰符
  - `attribute`：属性修饰符
  - `uniform`：全局变量修饰符
  - `varying`：顶点着色器传递给片元着色器的属性修饰符，此修饰的变量具备从顶点着色器向片元着色器传递插值数据的能力。在 OpenGL ES 3.0+ 或 WebGL 2.0 版本之后，此修饰被 `out` 和 `in` 替代
- `precision`：精度设置限定符，使用此限定符设置完精度后，之后所有该数据类型都将沿用该精度，除非单独设置
  - `highp`：高精度
  - `mediump`：中等精度
  - `lowp`：低精度
- 内置变量
  - `gl_Position`：用来设置顶点坐标
  - `gl_FragColor`：用来设置像素颜色
  - `gl_PointSize`：用来设置顶点大小
- 屏幕坐标系到设备坐标系的转换
  - 屏幕坐标系左上角为原点，X 轴坐标向右为正，Y 轴坐标向下为正
  - 屏幕坐标系坐标范围：
    - X 轴：[0, canvas.width]
    - Y 轴：[0, canvas.height]
  - 设备坐标系以屏幕中心为原点，X 轴坐标向右为正，Y 轴向上为正
  - 设备坐标系坐标范围是
    - X 轴：[-1, 1]
    - Y 轴：[-1, 1]

## 常用 WebGL API

- shader：着色器对象
  - `gl.createShader`：创建着色器。
  - `gl.shaderSource`：指定着色器源码。
  - `gl.compileShader`：编译着色器。
- program：着色器程序
  - `gl.createProgram`：创建着色器程序。
  - `gl.attachShader`：链接着色器对象。
  - `gl.linkProgram`：链接着色器程序。
  - `gl.useProgram`：使用着色器程序。
- attribute：着色器属性
  - `gl.getAttribLocation`：获取顶点着色器中的属性位置。
  - `gl.enableVertexAttribArray`：启用着色器属性。
  - `gl.vertexAttribPointer`：设置着色器属性读取 buffer 的方式。
  - `gl.vertexAttrib2f`：给着色器属性赋值，值为两个浮点数。
  - `gl.vertexAttrib3f`：给着色器属性赋值，值为三个浮点数。
- uniform：着色器全局属性
  - `gl.getUniformLocation`：获取全局变量位置。
  - `gl.uniform4f`：给全局变量赋值 4 个浮点数。
  - `gl.uniform1i`：给全局变量赋值 1 个整数。
- buffer：缓冲区
  - `gl.createBuffer`：创建缓冲区对象。
  - `gl.bindBuffer`：将缓冲区对象设置为当前缓冲。
  - `gl.bufferData`：向当前缓冲对象复制数据。
- clear：清屏
  - `gl.clearColor`：设置清除屏幕的背景色。
  - `gl.clear`：清除屏幕。
- draw：绘制
  - `gl.drawArrays`：数组绘制方式。
  - `gl.drawElements`：索引绘制方式。
- 图元
  - `gl.POINTS`：点。
  - `gl.LINE`：基本线段。
  - `gl.LINE_STRIP`：连续线段。
  - `gl.LINE_LOOP`：闭合线段。
  - `gl.TRIANGLES`：基本三角形。
  - `gl.TRIANGLE_STRIP`：三角带。
  - `gl.TRIANGLE_FAN`：三角扇。
- 纹理
  - `gl.createTexture`：创建纹理对象。
  - `gl.activeTexture`：激活纹理单元。
  - `gl.bindTexture`：绑定纹理对象到当前纹理。
  - `gl.texImage2D`：将图片数据传递给 GPU。
  - `gl.texParameterf`：设置图片放大缩小时的过滤算法。

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
   - 统一生命周期：所有顶点相关属性共享同一 Buffer 对象，避免多 Buffer 的创建/销毁带来的资源管理复杂度。
   - 兼容 WebGL 2.0 优化：支持 VAO（Vertex Array Object） 将 Buffer 配置与着色器属性绑定固化，进一步提升多模型切换效率。
5. 典型应用场景
   - 复杂模型渲染：需同时处理顶点、法线、UV 等属性的模型。
   - 动态数据更新：如骨骼动画中顶点位置与权重的实时更新，通过单 Buffer 的 gl.bufferSubData 局部更新更高效。
   - 流式渲染：在实时生成顶点数据（如地形、流体模拟）时，单 Buffer 的双缓冲（Ping-Pong）机制可避免卡顿。

> 注意事项：
>
> - 数据类型对齐：需确保步长（stride）与硬件内存对齐要求匹配（如 4 字节对齐）。
> - 更新策略：若部分数据频繁变动，可通过 gl.bufferSubData 局部更新，避免整体重传。
> - 数据规模：若单 Buffer 数据量过大，可能导致内存溢出等问题。
> - 分块（Chunking）：将大型场景拆分为多个 Buffer 区块，按需加载（如地形网格）。
> - LOD（Level of Detail）：根据距离动态切换不同精度的 Buffer 数据，降低远处模型的顶点数
