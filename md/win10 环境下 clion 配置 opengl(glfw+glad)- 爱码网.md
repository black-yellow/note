> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.likecs.com](https://www.likecs.com/show-203849102.html)

> 先下载 glfw 和 gladGLFW:https://www.glfw.org/download.htmlGlad:https://glad.dav1d.de/ glad:glfw: 根据 clion 使用的编译器版本选择 32 位或者 64 位，我按网上的教程下了 32 位的，但忘了 clion 用的是 TDM-GCC-64, 结果一堆错误，花了不少时间才搞好。

先下载 glfw 和 glad

GLFW:[https://www.glfw.org/download.html](https://www.glfw.org/download.html)

Glad:[https://glad.dav1d.de/](https://glad.dav1d.de/)

glad:

![](https://www.likecs.com/default/index/img?u=L2RlZmF1bHQvaW5kZXgvaW1nP3U9YUhSMGNITTZMeTl3YVdGdWMyaGxiaTVqYjIwdmFXMWhaMlZ6THpnek9TODBZV1JtTkRJMk1UVmhZek0yWldVNFl6QmlOR0l4TVdFMk1Ea3hNV1kxWmk1d2JtYz0=)

![](https://www.likecs.com/default/index/img?u=L2RlZmF1bHQvaW5kZXgvaW1nP3U9YUhSMGNITTZMeTl3YVdGdWMyaGxiaTVqYjIwdmFXMWhaMlZ6THpVMk5pODROVFkyWkdZM05tSXpabUZqWldZME5HTmlOR0UzTUdNeFptRXdabUpoWlM1d2JtYz0=)

**glfw: 根据 clion 使用的编译器版本选择 32 位或者 64 位，我按网上的教程下了 32 位的，但忘了 clion 用的是 TDM-GCC-64, 结果一堆错误，花了不少时间才搞好。**

![](https://www.likecs.com/default/index/img?u=L2RlZmF1bHQvaW5kZXgvaW1nP3U9YUhSMGNITTZMeTl3YVdGdWMyaGxiaTVqYjIwdmFXMWhaMlZ6THpJek9DODJZV0l5T1dFNVpESTFOelV4WTJKbU5XVTBOREpoTnpBeU1HTXdZMkZqTmk1d2JtYz0=)

两个库都下好后解压，glad 解压后得到两个文件夹：include 和 src, 这两个文件夹里所有的文件都需要

glfw 只需要 include 文件夹和 lib-mingw(32 位的文件夹，64 位下是 lib-mingw-w64) 下的 glfw3.dll

新建一个项目，在工程根目录下新建 3 个文件夹，include,lib,src。把 glad 和 glfw 的 include 文件夹下所有文件夹都复制到刚建的 include 文件夹下 (glad 的 include 下的 glad 和 KHR 文件夹，glfw 的 include 下的 GLFW 文件夹), 把 glad 下的 src 下的 glad.c 复制到刚建的 src 文件夹下，把 glfw 的 glfw3.dll 文件复制到刚建的 lib 文件夹下。最后一步，别忘了把 glfw3.dll 复制到程序运行目录，clion 编译好的文件默认会放到项目根目录下的 cmake-build-debug 下，所以要把 glfw3.dll 也复制到这个文件夹下，不然运行不了。

工程目录结构：

![](https://www.likecs.com/default/index/img?u=L2RlZmF1bHQvaW5kZXgvaW1nP3U9YUhSMGNITTZMeTl3YVdGdWMyaGxiaTVqYjIwdmFXMWhaMlZ6THpNM05TODJOVE0wWVRZM1pqRXhOamxsWlRFNU16TmhNbVUxWldRM1pqUTFNVE5tWmk1d2JtYz0=)

修改 CMakeLists.txt:

![](https://www.likecs.com/default/index/img?u=L2RlZmF1bHQvaW5kZXgvaW1nP3U9YUhSMGNITTZMeTl3YVdGdWMyaGxiaTVqYjIwdmFXMWhaMlZ6THpZNE1pOWpZekppT0RCaFlUWTBaalJpWVdReU1XTXlNVGhqT0dZeVlqSmtNVEJqTWk1d2JtYz0=)

```
指定头文件所在的文件夹路径：
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/include)
```

指定库文件所在文件夹路径：

link_directories(${PROJECT_SOURCE_DIR}/lib)

链接：

```
target_link_libraries(opengldemo4 -lopengl32 ${PROJECT_SOURCE_DIR}/lib/glfw3.dll )
```

其中  ${PROJECT_SOURCE_DIR} 是指工程目录的根目录。这也是为什么上面说在工程根目录下新建 include,lib,src 的原因。如果不要工程根目录下，而在其它位置，那就要用全路径指定清楚，不然编译器找不到文件，报一堆错误。

另外不要忘了在 add_executable 中把 src 下的 glad.c 加上，一起编译。

最后写个 demo 测试一下。

![](https://www.likecs.com/default/index/img?u=L2RlZmF1bHQvaW5kZXgvaW1nP3U9YUhSMGNITTZMeTl3YVdGdWMyaGxiaTVqYjIwdmFXMWhaMlZ6THprMk1pOW1NR0ZoTUdabU9UVXlPV05oWVdVM04yRmpZV0ptTnpaa1ptWXhNVGxtTWk1d2JtYz0=)

![](https://www.likecs.com/default/index/img?u=L2RlZmF1bHQvaW5kZXgvaW1nP3U9YUhSMGNITTZMeTl3YVdGdWMyaGxiaTVqYjIwdmFXMWhaMlZ6THpVeEx6ZzBZemxqTkRjNVlXRmpaV1ptWVdZeU56Y3hZVFkyWVRsak1Ua3hNelZpTG5CdVp3PT0=)

代码参考：[https://learnopengl-cn.github.io/](https://learnopengl-cn.github.io/)

[cmakefile 参考: https://www.cnblogs.com/jhy16193335/p/10773377.html，非常感谢这位博主，网上找了很多教程都不行。](https://www.cnblogs.com/jhy16193335/p/10773377.html)

来源网络，如有侵犯到您的权益请联系 zengyin969@gmail.com 进行下架处理