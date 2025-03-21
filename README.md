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
