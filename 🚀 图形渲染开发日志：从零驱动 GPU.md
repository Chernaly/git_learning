# 🚀 图形渲染开发日志：从零驱动 GPU

## 📅 日期：2026-02-06
## 💻 环境：HP OMEN 10 (Windows 11) | Visual Studio 2022
## 🛠 核心工具链：CMake + GLFW + GLAD

---

## 1. 环境配置：打通 CPU 与 GPU 的“桥梁”
图形渲染的第一步是配置环境。在 C++ 中，这涉及到库的编译与链接：
- **GLFW**：处理窗口创建与用户输入。
- **GLAD**：OpenGL 的函数加载器。
- **编译原理**：通过 `CMakeLists.txt` 将 `.cpp` 源码、`.c` 文件（GLAD）与外部库文件（GLFW）链接成可执行文件。

---

## 2. 核心概念：坐标系与管线 (Pipeline)
作为一个 CV（计算机视觉）学习者，图形学的坐标系非常特别：

### 📍 坐标系对比
| 特性           | 计算机视觉 (OpenCV) | 图形学 (OpenGL)                 |
| :------------- | :------------------ | :------------------------------ |
| **原点 (0,0)** | 左上角              | 屏幕中心                        |
| **Y 轴方向**   | 向下为正            | 向上为正                        |
| **数值范围**   | 像素 (0 ~ 1080)     | 标准化设备坐标 NDC (-1.0 ~ 1.0) |



### 🏗 渲染管线 (Pipeline)
数据从 C++ 传递到显示器的过程就像工厂流水线：
1. **顶点数据**：C++ 定义坐标数组。
2. **顶点着色器 (Vertex Shader)**：处理空间位置。
3. **光栅化**：将数学上的三角形转换成像素点阵。
4. **片段着色器 (Fragment Shader)**：决定每个像素的颜色。

---

## 3. 内存管理：VBO 与 VAO
为了提高性能，数据不能每一帧都在 CPU 和 GPU 之间搬运。
- **VBO (Vertex Buffer Object)**：GPU 显存里的“仓库”，一次性存入所有顶点。
- **VAO (Vertex Array Object)**：数据的“解析说明书”，告诉 GPU 仓库里的二进制数据哪部分是坐标，哪部分是颜色。

---

## 4. 实时信号控制：Uniform 变量
我通过 `glUniform4f` 实现了 CPU 对 GPU 变量的实时控制。

### 📡 信号处理的视角
利用 `sin` 函数控制颜色，本质上是**时域信号对颜色通道的调制**：
$$Color(t) = \frac{\sin(\omega t)}{2} + 0.5$$
- **直流偏移 (DC Offset)**：$+0.5$ 保证颜色值永远在 $[0, 1]$ 范围内。
- **振幅 (Amplitude)**：$0.5$ 决定了颜色变化的深度。
- **频率 (Frequency)**：通过调整 `timeValue` 的系数可以改变颜色闪烁的速度。



---

## 5. 核心代码复盘 (动态彩色三角形)
```cpp
// 在渲染循环中动态修改颜色
float timeValue = glfwGetTime();
float greenValue = (sin(timeValue) / 2.0f) + 0.5f; // 信号调制

int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
// 将计算出的信号值传给片段着色器的 Uniform 变量
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.5f, 1.0f);