> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.5axxw.com](https://www.5axxw.com/wiki/content/a8okwr)

> GL/GLES/EGL/GLX/WGLLoader-Generator 基于官方规范。

GL/GLES/EGL/GLX/WGLLoader-Generator 基于官方规范。

使用 webservice 生成您需要的文件！

```
#include <glad/glad.h>

int main(int argc, char **argv)
{
    // .. setup the context

    if(!gladLoadGL()) {
        printf("Something went wrong!\n");
        exit(-1);
    }
    printf("OpenGL %d.%d\n", GLVersion.major, GLVersion.minor);

    // .. render here ..
}
```

Examples:

*   simple.c
*   hellowindow2.cpp 使用 GLFW：

Usage
-----

如果你不想安装的话，你可以使用 webservice

否则，请通过 pip 安装 glad：

```
# Windows
pip install glad

# Linux
pip install --user glad
# Linux global (root)
pip install glad

glad --help
```

要从 Github 安装最新版本：

```
pip install --upgrade git+https://github.com/dav1dde/glad.git#egg=glad
```

或者直接启动 glad（在克隆存储库之后）：

```
python -m glad --help
```

通过 vcpkg 安装和建造 GLADE

您可以使用 vcpkg dependency manager 下载并安装 GLADE：

```
```
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh
./vcpkg integrate install
vcpkg install glad
```
```

vcpkg 中的 glad 端口由 Microsoft 团队成员和社区贡献者保持最新。如果版本已过期，请在 vcpkg 存储库上创建问题或请求请求。

可能的命令行选项：

```
usage: glad [-h] [--profile {core,compatibility}] --out-path OUT
                 [--api API] --generator {c,d,volt}
                 [--extensions EXTENSIONS] [--spec {gl,egl,glx,wgl}]
                 [--no-loader]

Uses the official Khronos-XML specs to generate a GL/GLES/EGL/GLX/WGL Loader
made for your needs. Glad currently supports the languages C, D and Volt.

optional arguments:
  -h, --help            show this help message and exit
  --profile {core,compatibility}
                        OpenGL profile (defaults to compatibility)
  --out-path OUT        Output path for loader
  --api API             API type/version pairs, like "gl=3.2,gles=", no
                        version means latest
  --generator {c,c-debug,d,volt}
                        Language to generate the binding for
  --extensions EXTENSIONS
                        Path to extensions file or comma separated list of
                        extensions, if missing all extensions are included
  --spec {gl,egl,glx,wgl}
                        Name of the spec
  --reproducible        Makes the build reproducible by not fetching 
                        the latest specification from Khronos
  --no-loader
  --omit-khrplatform    Omits inclusion of the khrplatform.h file which is
                        often unnecessary. Only has an effect if used
                        together with c generators.
  --local-files         Forces every file directly into the output directory.
                        No src or include subdirectories are generated. Only
                        has an effect if used together with c generators.
```

要为具有两个扩展名的 C 生成加载程序，可以如下所示：

```
python main.py --generator=c --extensions=GL_EXT_framebuffer_multisample,GL_EXT_texture_filter_anisotropic --out-path=GL
```

`--out-path`和`--generator`是必需的！如果缺少`--extensions`选项，glad 将添加对规范中找到的所有扩展的支持。

当将 glad 集成到构建系统中时，强烈建议使用`--reproducible`选项，这样可以防止 Khronos 对规范进行不兼容的更改时生成失败。

Generators
----------

### C/C++

```
struct gladGLversionStruct {
    int major;
    int minor;
};

extern struct gladGLversionStruct GLVersion;

typedef void* (* GLADloadproc)(const char *name);

/*
* Load OpenGL using the internal loader.
* Returns the true/1 if loading succeeded.
*
*/
int gladLoadGL(void);

/*
* Load OpenGL using an external loader like SDL_GL_GetProcAddress.
*
* Substitute GL with the API you generated
*
*/
int gladLoadGLLoader(GLADloadproc);
```

`glad.h`完全取代`gl.h`或`gl3.h`只包含`glad.h`。

```
if(!gladLoadGL()) { exit(-1); }
    printf("OpenGL Version %d.%d loaded", GLVersion.major, GLVersion.minor);
    
    if(GLAD_GL_EXT_framebuffer_multisample) {
        /* GL_EXT_framebuffer_multisample is supported */ 
    }
    
    if(GLAD_GL_VERSION_3_0) {
        /* We support at least OpenGL version 3 */
    }
```

在 non-Windows 平台上，glad 需要`libdl`，请确保与它链接（`-ldl`用于 gcc）！

注意，有两种扩展 / 版本符号，例如`GL_VERSION_3_0`和`GLAD_VERSION_3_0`。后者是一个运行时布尔值（表示为整数），而第一个（没有前缀`GLAD_`）是 compiletime-constant，表明这个头支持这个版本（官方的头也定义了这些符号）。运行时布尔值只有在成功调用`gladLoadGL`或`gladLoadGLLoader`之后才有效。

### C/C++ Debug

C-Debug 生成器通过以下两个函数扩展 API：

```
// this symbol only exists if generated with the c-debug generator
#define GLAD_DEBUG
typedef void (* GLADcallback)(const char *name, void *funcptr, int len_args, ...);

/*
* Sets a callback which will be called before every function call
* to a function loaded by glad.
*
*/
GLAPI void glad_set_pre_callback(GLADcallback cb);

/*
* Sets a callback which will be called after every function call
* to a function loaded by glad.
*
*/
GLAPI void glad_set_post_callback(GLADcallback cb);
```

在回调前缀为`glGetError`的函数中调用`glad_`，例如，默认的 post 回调如下所示：

```
void _post_call_callback_default(const char *name, void *funcptr, int len_args, ...) {
    GLenum error_code;
    error_code = glad_glGetError();

    if (error_code != GL_NO_ERROR) {
        fprintf(stderr, "ERROR %d in %s\n", error_code, name);
    }
}
```

您还可以为每个调用提交自己的实现，方法是用前缀`glad_debug_`的函数名覆盖函数指针。

E、 可以使用`glad_debug_glClear = glad_glClear`禁用 glClear 的回调，其中`glad_glClear`是 glad 加载的函数指针。

`glClear`宏被定义为`#define glClear glad_debug_glClear`，`glad_debug_glClear`是用一个默认实现初始化的，它调用两个回调函数和实函数，在本例中是`glad_glClear`。

### D

Import`glad.gl`对于 OpenGL 函数 / 扩展，Import`glad.loader`导入初始化 glad 和加载 OpenGL 函数所需的函数。

```
enforce(gladLoadGL()); // optionally you can pass a loader to this function
    writefln("OpenGL Version %d.%d loaded", GLVersion.major, GLVersion.minor);
    
    if(GL_EXT_framebuffer_multisample) { 
        /* GL_EXT_framebuffer_multisample is supported */ 
    }
    
    if(GL_VERSION_3_0) {
        /* We support at least OpenGL version 3 */
    }
```

在 non-Windows 平台上，glad 需要`libdl`，请确保与它链接（`L-ldl`用于 dmd）！

FAQ
---

### 我如何构建 glad 或如何集成 glad？

使用 glad 最简单的方法是通过 webservice。

或者，glad 与：

*   `CMake`
*   Conan  
    ![](https://www.5axxw.com/images_oss/zv/00cdb8d2-26c5-4cbb-af8e-c0e4b4127b89.svg)

感谢所有的帮助和支持维护这些！

### 包括 windows.h42

因为 0.1.30:glad 不再包含`windows.h`。

在 0.1.30 之前：在包含`glad.h`之前定义`APIENTRY`解决了这个问题：

```
#ifdef _WIN32
    #define APIENTRY __stdcall
#endif

#include <glad/glad.h>
```

但是对于那些定义了`_WIN32`但不使用`__stdcall`的平台，请确保您有正确的`APIENTRY`定义

### glad 生成代码的许可证是什么？

#101 #253

glad 生成的代码本身是公共域、WTFPL 或 CC0 中的任何一个，生成代码的源文件都在 Khronos 的各种许可下。

*   EGL：见 egl.xml
*   GL:Apache Version2.0 版
*   GLX:Apache Version2.0
*   工作日志：Apache Version2.0
*   Vulkan:Apache Version2.0，生成的代码除外

现在 Apache License 可能适用于生成的代码（不是律师），但请参见下面的澄清注释。

Glad 还添加了 Khronos 的头文件，这些文件头中有单独的许可证。

Contribute
----------

贡献很容易！发现虫子了？给我留言或者请求拉！添加新的生成器后端？发出请求！

特别感谢所有贡献和将要贡献的人！也要感谢那些在我根本想不出解决办法的时候帮我解决问题的人。