> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/369203981)

写在前面

完美转发是一个比较简单，却又比较复杂的东西。

简单之处在于理解**动机**：C++ 为什么需要完美转发？

复杂之处在于理解**原理**：完美转发基于万能引用，引用折叠以及 std::forward 模板函数。

本文将会结合 GCC 源码，详细解读完美转发的动机和原理。

### 动机：C++ 为什么需要完美转发？

我们从一个简单的例子出发。 假设有这么一种情况，用户一般使用 testForward 函数，testForward 什么也不做，只是简单的转调用到 print 函数。

```
template<typename T>
void print(T & t){
    std::cout << "Lvalue ref" << std::endl;
}

template<typename T>
void print(T && t){
    std::cout << "Rvalue ref" << std::endl;
}

template<typename T>
void testForward(T && v){ 
    print(v);//v此时已经是个左值了,永远调用左值版本的print
    print(std::forward<T>(v)); //本文的重点
    print(std::move(v)); //永远调用右值版本的print

    std::cout << "======================" << std::endl;
}

int main(int argc, char * argv[])
{
    int x = 1;
    testForward(x); //实参为左值
    testForward(std::move(x)); //实参为右值
}
```

上面的程序的运行结果：

```
Lvalue ref
Lvalue ref
Rvalue ref
======================
Lvalue ref
Rvalue ref
Rvalue ref
======================
```

用户希望`testForward(x);`最终调用的是左值版本的 print，而`testForward(std::move(x));`最终调用的是右值版本的 print。

**可惜的是，在 testForward 中，虽然参数 v 是右值类型的，但此时 v 在内存中已经有了位置，所以 v 其实是个左值！（请仔细阅读这段话，保证你理解了）**

所以，`print(v)`永远调用左值版本的 print，与用户的本意不符。`print(std::move(v));`永远调用右值版本的 print，与用户的本意也不符。只有`print(std::forward<T>(v));`才符合用户的本意，这就是本文的主题。

不难发现，本质问题在于，左值右值在函数调用时，都转化成了左值，使得函数转调用时无法判断左值和右值。

在 STL 中，随处可见这种问题。比如 C++11 引入的`emplace_back`，它接受左值也接受右值作为参数，接着，它转调用了空间配置器的 construct 函数，而 construct 又转调用了`placement new`，`placement new`根据参数是左值还是右值，决定调用拷贝构造函数还是移动构造函数。

这里要保证从`emplace_back`到`placement new`，参数的左值和右值属性保持不变。这其实不是一件简单的事情。

### 前置知识 引用折叠 万能引用

C++ Primer 里面写的比较容易理解，在 P608（我的是第 5 版）。

也可以参考我之前的文章：

### 原理：完美转发

std::forward 不是独自运作的，在我的理解里，完美转发 = std::forward + 万能引用 + 引用折叠。三者合一才能实现完美转发的效果。

std::forward 的正确运作的前提，是引用折叠机制，为 T && 类型的万能引用中的模板参数 T 赋了一个恰到好处的值。我们用 T 去指明 std::forward 的模板参数，从而使得 std::forward 返回的是正确的类型。（看不懂？没关系，先看下面的例子和代码）

当然，我们还是先回到一开始的例子。

### testForward(x)

回到上面的例子。先考虑`testForward(x);`这一行代码。

### step 1 实例化 testForward

根据万能引用的实例化规则，实例化的 testForward 大概长这样：

```
T = int &
void testForward(int & && v){
    print(std::forward<T>(v));
}
```

又根据引用折叠，上面的等价于下面的代码：

```
T = int &
void testForward(int & v){
    print(std::forward<int &>(v));
}
```

如果你正确的理解了引用折叠，那么这一步是很好理解的。

### step 2 实例化 std::forward

> 注：C++ Primer：forward 必须通过显式模板实参来调用，不能依赖函数模板参数推导。  

接下来我们来看下`std::forward`在 libstdc++ 中的实现：

```
68   /**
69    *  @brief  Forward an lvalue.
70    *  @return The parameter cast to the specified type.
71    *
72    *  This function is used to implement "perfect forwarding".
73    */
74   template<typename _Tp>
75     constexpr _Tp&&
76     forward(typename std::remove_reference<_Tp>::type& __t) noexcept
77     { return static_cast<_Tp&&>(__t); }
```

由于 Step1 中我们调用`std::forward<int &>`，所以此处我们代入`T = int &`，我们有：

```
constexpr int & && //折叠
forward(typename std::remove_reference<int &>::type& __t) noexcept //remove_reference的作用与名字一致，不过多解释
 { return static_cast<int & &&>(__t); } //折叠
```

这里又发生了 2 次引用折叠，所以上面的代码等价于：

```
constexpr int & //折叠
forward(int & __t) noexcept //remove_reference的作用与名字一致，不过多解释
 { return static_cast<int &>(__t); } //折叠
```

所以最终`std::forward<int &>(v)`的作用就是将参数强制转型成`int &`，而`int &`为左值。所以，调用左值版本的 print。

### testForward(std::move(x))

接下来，考虑`testForward(std::move(x))`的情况。

### step 1 实例化 testForward

`testForward(std::move(x))`也就是`testForward(static_cast<int &&>(x))`。根据万能引用的实例化规则，实例化的 testForward 大概长这样：

```
T = int 
void testForward(int && v){
    print(std::forward<int>(v));
}
```

万能引用绑定到右值上时，不会发生引用折叠，所以这里没有引用折叠。

### step 2 实例化 std::forward

> 注：C++ Primer：forward 必须通过显式模板实参来调用，不能依赖函数模板参数推导。  
> 这里用到的 std::forward 的代码和上面的一样，故略去。  
> 由于 Step1 中我们调用`std::forward<int>`，所以此处我们代入`T = int`，我们有：  

```
constexpr int && 
forward(typename std::remove_reference<int>::type& __t) noexcept //remove_reference的作用与名字一致，不过多解释
 { return static_cast<int &&>(__t); }
```

这里又发生了 2 次引用折叠，所以上面的代码等价于：

```
constexpr int &&
forward(int & __t) noexcept //remove_reference的作用与名字一致，不过多解释
 { return static_cast<int &&>(__t); }
```

所以最终`std::forward<int>(v)`的作用就是将参数强制转型成`int &&`，而`int &&`为右值。所以，调用右值版本的 print。

### 写在后面

在 GCC 源码中，std::forward 还有第二个版本：[link](https://zhuanlan.zhihu.com/p/369203981/move.h%20Source%20File)，分析的方法与本文一致，这里就不讲了。。

右值的概念其实很微妙，一旦某个右值，有了名字，也就在内存中有了位置，它就变成了 1 个左值。但它又是一个很有用的概念，**允许程序员更加细粒度的处理对象拷贝时的内存分配问题，提高了对临时对象和不需要的对象的利用率**，极大提高程序的效率。当然，也会引入更多的 bug。不过，这就是 C++ 的哲学，什么都允许你做，但出了问题，可别赖 C++ 这门语言。

完美转发基于万能引用，引用折叠以及 std::forward 模板函数。据我所知，STL 出现 std::forward，一定出现万能引用。其实这也很好理解，完美转发机制，是为了将左值和右值统一处理，节约代码量，而只有万能引用会出现同时接受左值和右值的情况，所以完美转发只存在于万能引用中。