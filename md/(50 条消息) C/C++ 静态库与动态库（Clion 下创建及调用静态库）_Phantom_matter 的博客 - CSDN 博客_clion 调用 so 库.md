> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Phantom_matter/article/details/121082906)

### 文章目录

*   [静态库](#_6)
*   [动态库](#_13)
*   [Clion 创建及调用静态库](#Clion_30)
*   *   [创建一个 C 静态库](#C_31)
    *   [对于生成的工程构建项目](#_33)
    *   [这两个文件会在调用的时候用到](#_35)
    *   [新建一个 C 工程，创建两个文件夹（在工程里），将刚才的两个文件放进去：](#C_37)
    *   [对 cmakelists.txt 进行修改](#cmakeliststxt__39)
    *   [运行结果](#_59)

库是现有的，已经写好的，可以复用的代码。

本质上来说库是一种可执行代码的二进制形式，可以被操作系统载入内存执行。库有两种：静态库（.a、.lib）和动态库（.so、.dll）。

所谓静态、动态是指链接。回顾一下，将一个程序编译成可执行程序的步骤：

![](https://img-blog.csdnimg.cn/88b0051bd9fd4fa6a479a796e141a4e4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGhhbnRvbV9tYXR0ZXI=,size_13,color_FFFFFF,t_70,g_se,x_16#pic_center)

静态库
===

之所以成为【静态库】，是因为在链接阶段，会将汇编生成的目标文件. o 与引用到的库一起链接打包到可执行文件中。因此对应的链接方式称为静态链接。  
l 静态库对函数库的链接是放在编译时期完成的。

l 程序在运行时与函数库再无瓜葛，移植方便。

l 浪费空间和资源，因为所有相关的目标文件与牵涉到的函数库被链接合成一个可执行文件。

动态库
===

为什么需要动态库，其实也是静态库的特点导致。

l 空间浪费是静态库的一个问题。  
![](https://img-blog.csdnimg.cn/3dee8f5670f94f2882666c9e7f431327.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGhhbnRvbV9tYXR0ZXI=,size_11,color_FFFFFF,t_70,g_se,x_16#pic_center)  
l 另一个问题是静态库对程序的更新、部署和发布页会带来麻烦。如果静态库 liba.lib 更新了，所以使用它的应用程序都需要重新编译、发布给用户（对于玩家来说，可能是一个很小的改动，却导致整个程序重新下载，全量更新）。

动态库在程序编译时并不会被连接到目标代码中，而是在程序运行是才被载入。不同的应用程序如果调用相同的库，那么在内存里只需要有一份该共享库的实例，规避了空间浪费问题。动态库在程序运行是才被载入，也解决了静态库对程序的更新、部署和发布页会带来麻烦。用户只需要更新动态库即可，增量更新。  
动态库特点总结：

l 动态库把对一些库函数的链接载入推迟到程序运行的时期。

l 可以实现进程之间的资源共享。（因此动态库也称为共享库）

l 将一些程序升级变得简单。

l 甚至可以真正做到链接载入完全由程序员在程序代码中控制（显示调用）。

[Clion](https://so.csdn.net/so/search?q=Clion&spm=1001.2101.3001.7020) 创建及调用静态库
===============================================================================

创建一个 C 静态库
----------

![](https://img-blog.csdnimg.cn/5fc0d6bb87ab46ba9f561ce30a247a97.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_10,text_Q1NETiBAUGhhbnRvbV9tYXR0ZXI=,size_10,color_FFFFFF,t_70,g_se,x_16#pic_center)

对于生成的工程构建项目
-----------

![](https://img-blog.csdnimg.cn/8618c89ea46442dba31603690cbb5fa3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGhhbnRvbV9tYXR0ZXI=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

这两个文件会在调用的时候用到
--------------

![](https://img-blog.csdnimg.cn/a08b5221bf074efab0d1d280cde2f137.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGhhbnRvbV9tYXR0ZXI=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

新建一个 C 工程，创建两个文件夹（在工程里），将刚才的两个文件放进去：
------------------------------------

![](https://img-blog.csdnimg.cn/edd0728a5de648bb8c46be656db3a375.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGhhbnRvbV9tYXR0ZXI=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

对 cmakelists.txt 进行修改
---------------------

```
cmake_minimum_required(VERSION 3.17)
project(untitled4 C)

set(CMAKE_C_STANDARD 11)
# 指定lib目录
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 指定头文件搜索路径
include_directories(${PROJECT_SOURCE_DIR}/include)


add_executable(${PROJECT_NAME} main.c)
# 将库链接到项目中
target_link_libraries(${PROJECT_NAME} libuntitled3.a)
```

![](https://img-blog.csdnimg.cn/034e86530d004a6bb1a31b45af824c86.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGhhbnRvbV9tYXR0ZXI=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)  
然后重新加载 **cmakelists.txt** 就行了。

运行结果
----

![](https://img-blog.csdnimg.cn/5714949645f049a5a505b74884ce4dc4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAUGhhbnRvbV9tYXR0ZXI=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)