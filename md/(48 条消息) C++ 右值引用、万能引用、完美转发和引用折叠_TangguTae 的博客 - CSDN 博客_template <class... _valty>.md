> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_43164548/article/details/126033687)

**目录**

[什么是左值，什么是右值？](#t0)

[右值引用](#t1)

[万能引用](#t2)

[引用折叠](#t3)

[完美转发](#t4)

什么是左值，什么是右值？
------------

（接下来我们将左值称为 lvalue，右值成为 rvalue）

左值通常指的是变量，或者说是可以放到等号左边的表达式。右值通常是**常量、表达式或者函数返回值**（临时对象）。

更加精简的说法是：

如果**可以对一个表达式取地址**，那这个表达式就是 lvalue。

其他情况下，这个表达式就是一个 rvalue。

**举例子：**

常见的左值的情况

```
int x=2,y=3;//x,y就为左值
++x;--y;//表达式为左值
const int& z = x;//z也为左值
```

右值又可以分为两种情况：**纯右值**和**将亡值**

**纯右值**：基本类型（int，char 等）的常量或者临时对象。

```
100,true//常量
x++,x+1//表达式
```

**将亡值**：自定义类型的临时对象 和 **函数返回对象类型的右值引用**。

```
int&& fuc1(void)
{
    return 100;
}
 
string fuc2(void)
{
    string str = "hello";
    return str;
}
int main()
{
    fuc1();//函数返回值为右值引用
    fuc2();//函数返回值为临时对象
}
 
//对对象类型右值引用的转换
void fuc3()
{
    static_cast<int&&>(100);
    std::move(100);
}
```

> **表达式的左右值与类型无关！**
> 
> 这里有两个概念，一个是值类别，一个是值类型。
> 
> 值类别：对应着左值和右值的概念
> 
> 值类型：数据类型。
> 
> 例如：
> 
> int x = 100;
> 
> const int& y = x;
> 
> int&& z = 100;
> 
> 此时 x 是的值类别为左值，值类型为 int 类型。
> 
> 同理，y 的值类别为左值，而值类型为 const int&
> 
> 注意：z 的值类别也是左值，值类型为 int&&。

[右值引用](https://so.csdn.net/so/search?q=%E5%8F%B3%E5%80%BC%E5%BC%95%E7%94%A8&spm=1001.2101.3001.7020)
----------------------------------------------------------------------------------------------------

正常的左值引用是无法引用右值的（**常左值引用可以**），所以需要右值引用去引用右值。

```
int main()
{
    int x = 1,y = 2;
 
    //左值引用
    int a = 0;
    int& b = a;
 
    //左值引用不能引用右值，const左值引用可以，因为临时变量具有常性
    const int& e = 10;
    const int& f = x + y;
 
    //右值引用
    int&& c = 10;
    int&& d = x + y;
 
    //右值引用不能引用左值，但是可以引用move后的左值
    int&& m = move(a);
    return 0;
}
```

万能引用
----

万能意思就和表面的意思一样，即**既可以引用左值，也可以引用右值**。

万能引用的形式也是 “&&”，所以说如何区分一个引用是右值引用还是万能引用？

总共有两种情况会出现万能引用

**1、函数模板参数**

template<typename T>

void fuc(T**&&** param); // 此时为万能引用

**2、auto 声明**

auto**&&** var2 = var1;//var2 是一个万能引用

这两种存在共同的特点：**都存在类型的推导**。其实这两者可以归为一类，auto 声明的变量的类型推导规则本质上和模板是一样的，所以使用 auto 的时候你也可能得到一个万能引用。

例如：

```
template<typename T>
void fuc1(T&& param); //此时为万能引用
 
void fuc2(int&& param);//没有类型推导
 
 
int main()
{
	int x = 100;
	fuc1(100);
	fuc1(x);
	return 0;
}
```

因为**通用引用是引用，所以必须初始化**。通用引用的初始化决定了它表示的是右值引用还是左值引用。fuc1(100) 中 100 是右值，说明通用引用被一个右值初始化。fuc1(x) 中 x 为左值，说明通用引用被一个左值初始化。

有几种情况不要弄混淆

**1、引用必须得精确！**

**即一定得是 T&& 的形式**

template<typename T>  
void fuc(std::vector<T>**&&** param);

上面这个例子不具备 T&& param 的格式，而是 vector<T>&& param 的格式，所以只是一个普通的右值引用。

```
template<typename T>
void fuc(vector<T>&& param); //此时不是万能引用
int main
{
    vector<int> v;//左值
	fuc(v);//error...
	fuc(vector<int>());//vector<int>()临时变量，右值
    
    return 0;
}
```

![](https://img-blog.csdnimg.cn/f97651190ff64f5abf6b02c791291337.png)

**注意**：const T&& 也会使得通用引用失效。

template<typename T>  
void fuc(const T**&&** param);// 此时为右值引用

**2、模板内部的函数参数为 T&& 的形式，注意分辨清楚！**

利用 vector 中的 push_back 和 emplace_back 为例来说明这个情况。

```
template <class T, class Allocator = allocator<T> >
class vector {
public:
	...
		void push_back(T&& x);       // fully specified parameter type ⇒ no type deduction;
	...                          // && ≡ rvalue reference
};
```

注意这种情况，虽然 T&& 是模板参数，但是此时这个参数是右值引用，原因是 vector<T> 这个类就已经知道 T 是什么类型了，此时对于 void push_back(T&& x); 中的 T 没必要再推导了，缺少万能应用推导的那一个环节。也就是说 **push_back 依赖于 vector 的实例化**，**而实例化的类型就完全决定了 push_back 的函数声明**。可以在看一下 void push_back(T&& x); 在类外面是如何定义的。

```
template <class T>
void vector<T>::push_back(T&& x);//依赖于vector<T>类
```

vector 模板实例化的过程：

以 vector<string> vs; 为例

```
template <class string, class Allocator = allocator<string> >
class vector {
public:
	...
		void push_back(string&& x);       // && ≡ rvalue reference
	...                          
};
```

push_back 并没有用到类型推导，直接取决于 vector 的实例化。

所以这也是为什么 push_back 有两个版本的原因。

![](https://img-blog.csdnimg.cn/c45a6cec893241adb6df6979f408db1f.png)

与此相反，vector 中另一个插入的函数 emplace_back 就可以实现类型的推导。

```
template <class T, class Allocator = allocator<T> >
class vector {
public:
    ...
    template <class... _Valty>
    void emplace_back(_Valty&&... _val); // deduced parameter types ⇒ type deduction;
    ...                                // && ≡ universal references
};
```

类型 **_Valty** 是独立于模板参数 T 的，所以每次调用 emplace_back 时，_val 就需要推导一次，这就是它比较巧妙的地方。

```
template<class... Args>
void std::vector<string>::emplace_back(Args&&... args);
```

看一下这两个的原码（msvc）

```
template<class... _Valty>
		decltype(auto) emplace_back(_Valty&&... _Val)
		{	// insert by perfectly forwarding into element at end, provide strong guarantee
		if (_Has_unused_capacity())
			{
			return (_Emplace_back_with_unused_capacity(_STD forward<_Valty>(_Val)...));
			}
 
		_Ty& _Result = *_Emplace_reallocate(this->_Mylast(), _STD forward<_Valty>(_Val)...);
#if _HAS_CXX17
		return (_Result);
#else /* ^^^ _HAS_CXX17 ^^^ // vvv !_HAS_CXX17 vvv */
		(void)_Result;
#endif /* _HAS_CXX17 */
		}
 
	void push_back(const _Ty& _Val)
		{	// insert element at end, provide strong guarantee
		emplace_back(_Val);
		}
 
	void push_back(_Ty&& _Val)
		{	// insert by moving into element at end, provide strong guarantee
		emplace_back(_STD move(_Val));
		}
```

可以看到在 msvc 下的 push_back 是基于 emplace_back 实现的。在这里，有两个比较重要的地方，一个是 **std::move** 和 **std::forward** 两个函数，放在后面的**完美转发**再讲。

引用折叠
----

在这里，其实是涉及到类型的推导，也就是去推模板参数中的 T 具体是什么类型，而且是针对通用引用场景下的类型的推导。

在通用引用下类型推倒的机制很简单：**当实参是左值时，T 的类型为左值引用，当传右值时，T 被推导为非引用类型。**

例如：

```
template<typename T>
void fuc(T&& param); //此时为万能引用
 
string str = "hello";
 
fuc(str);//T推导为 string&
fuc(string());//T推导为string
```

![](https://img-blog.csdnimg.cn/618d25aa50e6453a9d7ca768976acb53.png)

![](https://img-blog.csdnimg.cn/9396246de09b4fa8ac5c63ceebc40bfc.png)

str 为左值，所以 T 的类型会被推到为 string&，那么 param 的类型就为 string**& &&**，我们知道在 C++ 里面引用的引用是非法的，例如 int x = 100;  auto**& &**y = x;

![](https://img-blog.csdnimg.cn/d46a4ee767df4058afc6e38a6d286347.png)

所以 string**& &&** 这该如何解释呢？

**引用折叠！**

程序员是不能自己声明引用的引用，但是编译器可以在特定的上下文中生成，模板的实例化就是其中之一（另外还有三种情况），编译器生成的引用的引用就会触发引用折叠。

**引用折叠的情况**

由于引用有两种类型：左值引用 & 和右值引用 &&，所以两两组合总共四种情况，分别是 lvalue reference to lvalue reference,  lvalue reference to rvalue reference,  rvalue reference to lvalue reference, 以及 rvalue reference to rvalue reference。

**引用折叠的规则：**

如果两个引用中有任何一个是左值引用 lvalue reference ，那么最终的结果一定是左值引用，否则就为右值引用。

所以上面的 string& && 最终得到 param 的类型为 string&。

```
template<typename T>
void fuc(T&& param); //此时为万能引用
 
int main()
{
	int x = 100;
	int& lx = x;
	int&& rx = 100;
 
	fuc(x);//T被推导为int&
	fuc(lx);//T被推导为int&
	fuc(rx);//T被推导为int&
 
	return 0;
}
```

有些人会在上面的例子存在疑惑，这里的右值引用传进去为什么还是 int&，其实在文章开头的时候就已经说过了，要区分**值类别**与**值类型**。虽然 x，lx，rx 的值类型都不相同，分别为 int，int&，int&&，但是他们的值类别是相同的，都是 lvalue。根据上面的规则左值对应的就是类型的引用。

除此之外，还有三种情况可能导致引用折叠

**auto&&**

auto 本身类型推导与模板类型推导基本相同

```
string str = "hello";
auto&& lstr = str;//等价为 string& && lstr = str;
auto&& rstr = string();//等价为 string&& lstr = string();
```

**typedef**

如下例子可以说明 typedef 场景也可能存在引用折叠的情况。 

```
template<typename T>
class myclass
{
	typedef T&& RvalueRfeType;
};
int main()
{
    myclass<int&> mc;
}
```

myclass<int&> mc; 此时将 int& 带入模板可以得到

typedef **int& &&** RvalueRfeType; 可以推出 RvalueRfeType 为一个左值引用 int&，所以并不是想象当中的右值引用，需要注意。

**decltype**

这里先留个坑，等到下次详细讲 auto 与 decltype 时在回过来填坑。

完美转发
----

先看个例子

```
void fuc(int& param)//左值引用
{
	cout << "lvalue reference" << endl;
}
 
void fuc(int&& param)//右值引用
{
	cout << "rvalue reference" << endl;
}
 
template<typename T>
void PerfectFoward(T&& param)//通用引用
{
	fuc(param);
};
 
int main()
{
	PerfectFoward(100);//传一个右值
 
	return 0;
}
```

输出结果：lvalue reference

![](https://img-blog.csdnimg.cn/e01c3968cf0944e3a6c65b15bef8ea2c.png)

传进来是个右值，为什么最终调用的却是 fuc 函数参数为左值引用 的版本呢？

捋一捋过程：

100 是右值，模板类型的推导 T 为 int，然后 param 的类型就是 int&&，到这没问题，但是，忽略掉了此时的 param 为左值，那调用函数当然是去调用形参为左值引用版本的函数。

所以说，在传参的过程中，**右值引用在第二次传递参数的过程中，右值属性会发生丢失**，导致调用的都是左值引用的函数。

为了解决这个问题，引入**完美转发**。

```
template<typename T>
void PerfectFoward(T&& param)
{
	fuc(forward<T>(param));//forward完美转发
};
```

![](https://img-blog.csdnimg.cn/06ae901927694f92a03265ad334f98ff.png)

**forward 的实现原理**

底层源码：

```
template <typename T>
T&& forward(typename std::remove_reference<T>::type& param)
{
    return static_cast<T&&>(param);
}
```

在这里面有一个东西 **remove_reference**，这个就比较有意思了。他的作用是移除 T 中的引用部分。**也就是将 T & 和 T&& 变成 T**。

```
template<typename _Tp>
struct remove_reference
{ typedef _Tp   type; };
 
template<typename _Tp>
struct remove_reference<_Tp&>
{ typedef _Tp   type; };
 
template<typename _Tp>
struct remove_reference<_Tp&&>
{ typedef _Tp   type; };
```

这里其实也是模板推导的一些知识（不懂得可以去看一下 effective modern c++ 里面的条款 1），正因为如此才可以把 T 里面的引用给去除掉。

（C++ 真的是语法让人又爱又恨，不得感叹有意思，但是就很麻烦）

在头文件 <type_traits> 中有各种各样的模板完成这种需求的转换工作，把这些叫做模板元编程（TMP），抱歉我不是一个牛叉的 C++ 程序员。我要走的路还很长。

我们回过头来再来看看 forward 的实现。

假设我们现在给一个左值，int x = 100。

```
template<typename T>
void PerfectFoward(T&& param)
{
	fuc(forward<T>(param));//forward完美转发
};
int x = 100;
PerfectFoward(x);
```

此时 T 的类型为 int&，并将 int& 传递给 forward。再来看 forward 这里

```
int& && forward(typename std::remove_reference<int&>::type& param)
{
    return static_cast<int& &&>(param);
}
```

remove_reference<int&>::type 会去除掉 int& 中的 & 部分，所以返回值为 int，那么引用折叠完之后最终得到的是：

```
int& forward(int& param)
{
    return static_cast<int&>(param);
}
```

static_cast 返回值为一个左值引用，此时 static_cast 啥也没做，**保留了输入参数的左值属性**。

当传递一个右值的时候

```
template<typename T>
void PerfectFoward(T&& param)
{
	fuc(forward<T>(param));//forward完美转发
};
PerfectFoward(100);
```

 此时模板参数 T 被推导出为 int，那么 forward 这里

```
int&& forward(int& param)
{
    return static_cast<int&&>(param);
}
```

此时 static_cast 的返回值为 int&&，在这篇文章开头讲过，**函数返回值的右值引用是一个右值**，转换成功，将一个左值 param 转换为一个右值，**保留了最开始输入参数的右值属性**。

与 forward 对应的是 **move 函数**

move 函数就很直接，直接把任何类别的对象都转化为右值。

move 的源码

```
template<typename _Tp>  constexpr typename std::remove_reference<_Tp>::type&&  move(_Tp&& __t) noexcept  
{ 
    return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); 
}
```

可以看到，不管怎样 static_cast 返回值都为右值。

注意 move 是一个掠夺的机制，所以需要小心，针对右值是没什么问题的，但是针对左值时就需要小心，effective modern c++ 中建议，**对右值使用 move，对通用引用使用 forward**，根据前面所描述的，想必你们也想到了在通用引用中用 move 可能会存在什么问题。

讲了这么多，那么右值引用有什么用？

我认为**移动构造**和**移动赋值**

```
class String
{
public:
	String(const char* str = " ")
	{
		_str = new char[strlen(str) + 1];
		strcpy(_str, str);
	}
	String(const String& s)//左值版本
	{
		cout << "String(const String& s)-深拷贝" << endl;
		_str = new char[strlen(s._str) + 1];
		strcpy(_str, s._str);
	}
	~String()
	{
		delete[] _str;
	}
private:
	char* _str;
};
String fuc(const char* str)
{
	String tmp(str);
	return tmp;//返回的是临时对象
}
int main()
{
	String s1("hello world");
	String s2(s1);//参数是左值
	String s3(fuc("临时对象-右值"));//临时对象，参数是右值-将亡值，用完就析构了
	return 0;
}
```

![](https://img-blog.csdnimg.cn/7ac60e0aa4f14a988c7568dbaa9c059c.png) 

对于这个右值，而且是将亡值，没有必要进行深拷贝，用完直接析构，所以考虑**移动拷贝**。

```
String(String&& s)//右值，将亡值
        :_str(nullptr)//把空值交换，进行析构就没问题，随机值析构很危险
    {
        cout << "String(const String&& s)-移动拷贝" << endl;
        swap(_str, s._str);
    }
```

直接将空间进行交换，这样效率高。

![](https://img-blog.csdnimg.cn/8f4e255b131c483da83df264a9f44ff0.png)

**参考的文章**

[现代 C++ 之万能引用、完美转发、引用折叠 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/99524127 "现代C++之万能引用、完美转发、引用折叠 - 知乎 (zhihu.com)")

《effective modern c++》