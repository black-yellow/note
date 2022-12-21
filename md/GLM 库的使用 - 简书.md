> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/7bbe387109ea)

简介
--

OpenGL 没有内建矩阵运算方法，常用的第三方库为 [GLM](https://links.jianshu.com/go?to=https%3A%2F%2Fglm.g-truc.net%2F)。GLM 是 OpenGL Mathematics 的缩写。作为一个 header only 库，GLM 只要包括了相应的头文件就可以使用它提供的类和函数。GLM 是 C++ 语言编写的，故不适用于 C 语言工程。

OpenGL Mathematics (GLM) 是基于 OpenGL 着色语言（GLSL）规范的图形软件的头文件 C ++ 数学库。提供的类和函数使用与 GLSL 相同的命名约定和功能设计和实现，因此任何知道 GLSL 的人都可以在 C++ 中使用 GLM。这个项目不限于 GLSL 的功能。基于 GLSL 扩展约定的扩展系统提供扩展能力：矩阵变换，四元数，数据打包，随机数，噪声等等。这个库与 OpenGL 完美地工作，但它也确保与其他第三方库和 SDK 的互操作性。它是软件渲染（光线追踪 / 光栅化），图像处理，物理模拟和任何需要简单方便的数学库的开发上下文的良好候选。

GLM 是用 C ++ 98 编写的，但是当编译器支持时可以利用 C ++ 11。它是一个没有依赖的平台独立库，它正式支持以下编译器：

*   苹果 Clang 6.0 及更高版本
*   GCC 4.7 及以上
*   英特尔 C ++ Composer XE 2013 及更高版本
*   LLVM 3.4 及更高版本
*   Visual C ++ 2013 及更高版本
*   CUDA 7.0 及更高版本（实验版）
*   任何 C ++ 11 编译器

头文件
---

*   GLM 对于矩阵数据类型的定义位于 glm/glm.hpp 头文件中。
*   生成变换矩阵的函数位于 glm/gtc/matrix_transform.hpp 头文件中。
*   生成投影矩阵的函数位于 glm/ext/matrix_clip_space.hpp 头文件中。
*   将数组转换成矩阵的函数位于头文件 glm/gtc/type_ptr.hpp 中。

GLM 常用数据类型
----------

*   vec2 二维向量
*   vec3 三维向量
*   vec4 四维向量
*   mat2 二阶矩阵
*   mat3 三阶矩阵
*   mat4 四阶矩阵

GLM 常用函数
--------

*   glm::radians()  
    角度制转弧度制，可应用于 glm::rotate() 中。
*   glm::translate()  
    返回一个平移矩阵，第一个参数是目标矩阵，第二个参数是平移的方向向量。
*   glm::rotate()  
    返回一个将点绕某个轴逆时针旋转一定弧度的旋转矩阵，第一个参数是目标矩阵，第二个参数是弧度，第三个参数是旋转轴。
*   glm::scale()  
    返回一个缩放矩阵，第一个参数是目标矩阵，第二个参数是在各坐标轴上的缩放系数。
*   glm::ortho(float left, float right, float bottom, float top, float zNear, float zFar);  
    正交投影矩阵。前四个参数分别是视口的左、右、上、下坐标。第五和第六个参数则定义了近平面和远平面的距离。
*   glm::perspective(float fovy, float aspect, float zNear, float zFar);  
    透视投影矩阵。第一个参数为视锥上下面之间的夹角，第二个参数为视口宽高比，第三、四个参数分别为近平面和远平面的深度。
*   glm::value_ptr()  
    传入一个矩阵，返回一个数组，从左到右按列优先。
*   glm::normalize(vector)  
    向量的单位化（保持其方向不变，将其长度化为 1）.  
    可以 normalize 一个运算式赋值，直接 normalize(a) 但不赋值，不能 normlize 单个向量，然后赋值给其他向量:

```
// 可以这样使用：
glm::vec3 a = { 3.0, 2.0, 3.0 };
glm::vec3 b = { 1.0, 0.0, 3.0 };
glm::vec3 c = glm::normalize(a*b);
// 或者：
glm::normalize(a); // 直接单位化，这里直接修改了参数a
// 但不能这么使用：
glm::vec3 b=glm::normalize(c);
```

*   glm::cross(vector1,vector2)  
    向量的叉乘。
*   glm::dot(vector1,vector2)  
    向量的点乘。
*   未完待续

GLM 矩阵的默认构造
-----------

GLM 库从 0.9.9 版本起，默认会将矩阵类型初始化为一个零矩阵（所有元素均为 0），而不是单位矩阵。如果使用 0.9.9 及以上的版本，需要在声明矩阵时传入参数 1，例如 glm::mat4 mat(1.0f)。

转载
--

[https://blog.csdn.net/qq_40565033/article/details/107716760](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fqq_40565033%2Farticle%2Fdetails%2F107716760)  
[https://blog.csdn.net/JuniorChestnut/article/details/106054005](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2FJuniorChestnut%2Farticle%2Fdetails%2F106054005)