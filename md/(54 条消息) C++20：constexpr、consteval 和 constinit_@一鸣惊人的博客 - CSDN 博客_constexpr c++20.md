> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/liuguang841118/article/details/127754252)

缘由
==

C++ 之父 Bjarne Stroustrup 和 Gabriel Dos Reis 于 2003 年向 C++ 标准委员会提出了一种用于编译期求值的更好的机制。他们的目标：

*   让编译时计算达到类型安全
*   通过将计算移至编译时，提升运行效率
*   支持嵌入式编程
*   直接支持元编程（而非模板元编程）
*   让编译时编程和 “普通编程” 类似

2003 年提议实现：允许在常量表达式中使用以 constexpr 为前缀的函数，还允许常量表达式使用简单的用户自定义类型，即字面值常量，这些字面值常量是一种所有运算都是 constexpr 的类型。

C++ 标准
======

C++ 标准委员会从 C++11 开始支持 Bjarne Stroustrup 和 Gabriel Dos Reis 的提议：

*   C++11 引入 constexpr 关键字，此时 constexpr 可修饰变量亦可函数，但是要求函数必须是纯函数；
*   C++14 标准移除了 C++11 的大部分限制，允许 constexpr 函数使用局部变量，同时实现对其他函数的支持；
*   C++17 将 constexpr 关键字引⼊到 if 语句，允许 lambda 声明为 constexpr；
*   C++20 允许将字面值类型用做模板参数。

到 C++20 对 constexpr 的完整支持，constexpr 标准化跨越了 13 年，由 4 个标准实现。C++20 也是最接近最初语言设计模板的版本，但是 constexpr 函数的设计其实也不够严谨，所以 C++20 又引入了 consteval；由于 constexpr 仅能实现编译时常量求值，为了解决编译时非常量求值问题，C++20 又引入了 constinit 关键字。

> **具备下述条件的函数，我们称之为纯函数**
> 
> *   函数无法访问非本地对象
> *   函数不能对调用者的环境产生副作用

C++11
=====

C++11 中的 constexpr 可修饰变量亦可修饰函数。但是无论是修饰变量还是函数，都要求其必须能在编译期常量求值。

constexpr 变量
------------

C++11 标准规定，满足下述条件的变量，可声明为 constexpr 变量：

*   变量的类型必须是字面类型
*   变量必须立即被初始化
*   变量的初始化包括所有隐式转换、构造函数调用等的全表达式必须是常量表达式
*   变量的类型不能是类类型或类类型的数组

constexpr 表达式
-------------

constexpr 表达式是指值不会改变并且在编译过程就可求值的表达式。声明为 constexpr 的变量一定 const 变量，而且必须用常量表达式初始化。

例如：

```
constexpr int hoursOfDay = 24; // 24 是常量表达式
constexpr int minutesOfHour = 60; // 60 是常量表达式
constexpr int minutesOfDay =  hoursOfDay * minutesOfHour; // hoursOfDay和minutesOfHour都是常量表达式，所以hoursOfDay * minutesOfHour也是常量表达式。
constexpr int seconds = secondsOfDay(); // secondsOfDay必须是常量函数，否则此声明非法。
```

constexpr 与自定义类型
----------------

对应自定义类型，不能用 constexpr 直接修饰类型，例如：

```
constexpr struct Rectangle  // constexpr 修饰自定义类型无效
{
	int length;
	int width;
};
```

如果要定一个结构体 / 或类常量对象，可采用这样的实现模式，例如：

```
struct Rectangle
{
	int length;
	int width;
};

constexpr Rectangle rect{1, 2};
constexpr int length = rect.length;
constexpr int width = rect.width;
```

constexpr 与指针
-------------

如果一个指针声明为 constexpr，那么限定符 constexpr 仅对指针有效，与指针所指对象无关。例如：

```
#include <iostream>
 
int main()
{
    int i = 1;
    int j = 2;
    std::cout << "i=" << i << std::endl;
    constexpr int* p = &i;
    *p = 8;
    std::cout << "i=" << i << std::endl;
    //p = &j; // 由于p有constexpr限定，导致此行代码编译失败
    return 0;
}
```

代码执行结果

```
i=1
i=8
```

constexpr 函数
------------

C++11 标准规定，满足下述条件的函数，可声明为 constexpr 函数：

*   函数必须为非虚函数
*   函数体不能包含 try 和 goto 语句
*   函数的返回值和入参必须都是字面类型
*   函数体只能包含:  
    – 空语句（仅分号）  
    – static_assert 声明  
    – 类或枚举的 typedef 声明及别名声明  
    – using 声明  
    – using 指令  
    – 如果函数不是构造函数，函数体仅能存在一条 return 语句
*   构造函数可声明为 constexpr(而且该类必须无虚基类)，析构函数不可声明为 constexpr  
    – 函数体不能 = delete  
    – 每个子对象都必须初始化，而且子对象都必须存在 constexpr 构造函数

对于 constexpr 函数模板和类模板的 constexpr 函数成员，必须至少有一个特化满足上述要求。

*   constexpr 非构造函数最多只能包含一行 return 语句，例如 constexpr 斐波那契数列。

```
// 此斐波那契数列实现的复杂度等同于迭代的方法，基本上为O(n)。
constexpr long int fibonacci(int n) 
{ 
	return (n <= 1)? n : fibonacci(n-1) + fibonacci(n-2); //只能包含一个retrun语句
}
```

*   constexpr 只能引用全局字面值常量 (编译阶段即可确定取值的变量)，例如：

```
constexpr int DAYS_OF_YEAR = 365;
constexpr int daysOfYear() 
{
	return DAYS_OF_YEAR;
}
```

*   constexpr 只能调用其他 constexpr 函数，不能调用非 constexpr 函数。例如：

```
constexpr int hoursOfYear()
{
	return 24 * daysOfYear();
}
```

*   constexpr 可以用于模板类和函数模板，而且可将非 constexpr 模板的显式专用化声明为 constexpr，例如：

```
// 例 1： 函数模板
// Compile-time computation of array length
template<typename T, int N>
constexpr int length(const T(&)[N])
{
    return N;
}

const int nums2[length(nums) * 2] { 1, 2, 3, 4, 5, 6, 7, 8 };
```

```
// 例 2： 将非 constexpr 模板的显式专用化声明为 constexpr
#include <cstdint>

template<uint64_t N>
struct Fibonacci
{
　　static constexpr uint64_t value = Fib<N - 1>::value + Fib<N - 2>::value
};

template<>
struct Fibonacci<1>
{
    static constexpr uint64_t value = 1;
};

template<>
struct Fibonacci<2>
{
    static constexpr uint64_t value = 1;
};

int main()
{
    auto value = Fibonacci<26>::value;
    return 0;
}
```

C++14
=====

C++11 标准中，constexpr 修饰函数除了可以包含 using 指令、typedef 语句以及 static_assert 断⾔ 外，只能包含⼀条 return 语句。而 C++14 标准允许 constexpr 函数使用局部变量，同时实现对其他函数的支持，所以 constexpr 修饰的函数可包含 if/switch 等条件语句，也可包含 for 循环。

虽然 C++14 放开了很多限制，但是依然存在部分 C++11 存在的严格限制：

*   函数体内不能有 goto 和 try 块，以及任何 static 和局部线程变量；
*   在函数中只能调用其他 constexpr 函数；
*   该函数也不能有任何运行时才会有的行为，比如抛出异常、使用 new 或 delete 操作等等；
*   constexpr 函数依然必须是纯函数

所以在 C++14 中，如果把斐波那契函数可以改成普通函数一样，它的可读性会大大提升。

```
constexpr unsigned fibonacci(unsigned i) 
{
 	switch (i) 
   	{
    case 0: 
    	return 0;
    	break;
    case 1: 
    	return 1;
    	break;
    default: 
    	return fibonacci(i - 1) + fibonacci(i - 2);
    	break;
   }
}
```

常量模板
----

常量模板是变量模板的一种特别形式，常量模板的表示常量的类型是模板，但是常量的值是 const 属性。我们可以利用 constexpr 实现常量模板。

```
template<typename T = long double>
constexpr T pi = T{3.1415926535897932385};
```

C++17
=====

C++17 对 C++14 标准增加了 2 个扩展：（一）将 constexpr 这个关键字引⼊到 if 语句；（二）将 constexpr 与 Lambda Expression 结合。

constexpr 与 if
--------------

C++17 基于 C++14，将 constexpr 这个关键字引⼊到 if 语句，允许在代码中声明常量表达式的判断条件。我们称 constexpr if 这种结合方式叫静态 if，静态 if 格式为：

```
if constexpr(cond)
     statement1; // Discarded if cond is false
else
     statement2; // Discarded if cond is true
```

基于静态 if，获取变量数值的模板函数举例：

```
template <typename T>
auto getValue(T t) 
{
    if constexpr (std::is_pointer_v<T>)
    {
        return *t;
    }
    else
    {
        return t;
    }
}
```

constexpr 与 Lambda Expression
-----------------------------

C++17 标准允许 lambda expression 在编译期求值，但是 constexpr lambda expression 也准寻下述 C++17 标准：

*   Lambda Expression 捕获的参数必须是字面值类型（Literal Type）

```
int y = 32;
    auto answer = [y]() constexpr
    {
        int x = 10;
        return y + x;
    };

    constexpr int increment(int n)
    {
        return [n] { return n + 1; }();
    }
```

*   如果 lambda 结果满足 constexpr 函数的要求，则 lambda 是隐式的 constexpr；

```
auto answer = [](int n)
    {
        return 32 + n;
    };

    constexpr int response = answer(10);
```

*   如果 lambda 是隐式或显式的 constexpr，并且将其转换为函数指针，则生成的函数也是 constexpr：

```
auto increment = [](int n)
    {
        return n + 1;
    };

    constexpr int(*inc)(int) = increment;
```

C++20
=====

C++20 标准兼容 C++11,C++14,C++17 标准，一方面对 constexpr 扩展；另一方面引入 consteval 和 constinit 解决 constexpr 存在的缺陷，使之臻于完美。

C++20 标准的扩展。第一个是非类型模板参数的约束释放；第二个是编译时内存分配；第三个是编译时多态，即 constexpr 虚拟函数的引入；第四个 constexpr 允许 try-catch；第五个 constexpr 中改变联合体的活跃成员。

虽然 C++20 增加了多项扩展，但是 C++20 constexpr 关键字依然存在两个缺陷，第一个缺陷是无法强制函数在编译器求值，第二个缺陷是无法解决编译时非常量求值问题。

非类型模板参数
-------

C++20 之前，模板的非类型模板参数仅支持简单的数值类型，但是 C++20 对此做出了重大调整。

*   在简单数值类型的基础上，增加对 float 类型的支持。

```
template <size_t N>
constexpr int f() {...}

template <float val>
constexpr float g() {...}
```

*   允许使用 auto 由编译期进行类型推导的非类型参数

```
template <auto ...>
struct ArgList
{
};

ArgList<'C', 0, 2L, nullptr> argList;
```

*   STL 对非类型字符串模板参数的支持。 C++20 之前，你不能将字符串用作非类型的模板参数，那现在我们可以使用 stl 中的 basic_fixed_string 解决这个问题，例如：

```
template<std::basic_fixed_string T>
class StringTemplate 
{
    static constexpr char const* name = T;
public:
    void hello() const 
    {
    	return name;
    }
};

int main() 
{
    StringTemplate<"Hello!"> stringTemplate;
    stringTemplate.hello();
}
```

一个可保障非类型字符串模板参数正常工作的 basic_fixed_string 定义：

```
#include <iostream>

template<unsigned N>
struct basic_fixed_string
{
    char m_str[N + 1]{};
    
    constexpr basic_fixed_string(char const* s)
    {
        for (unsigned i = 0; i != N; ++i)
        {
            m_str[i] = s[i];
        }
    }
    constexpr operator char const*() const
    {
        return m_str;
    }

    // C++20 三目运算符
    auto operator<=>(const basic_fixed_string&) const = default;
};

// CTAD 自定义类模板推送指引 C++17 开始支持
template<unsigned N> basic_fixed_string(char const (&)[N]) -> basic_fixed_string<N - 1>;

template<basic_fixed_string name>
class StringTemplate
{
public:
    auto hello() const { return name; }
};

int main()
{
    basic_fixed_string fixedString("Hello!!!!");
    std::cout << fixedString << std::endl;
    
    StringTemplate<"Hello!"> foo;
    std::cout << foo.hello() << std::endl;
}
```

编译时内存分配
-------

C++20 编译时内存分配，constexpr 函数可以进行有限制的动态内存分配和和使用 std::vector/std::string。C++20 之前只有 std::array 可以在编译期使用。当然这依然是有限制的使用：

*   constexpr 函数中不能使用 std::unique_ptr / std::shared_ptr
*   动态内存的生命周期必须在 constexpr 函数的上下文中，即不能返回动态内存分配的指针
*   不能返回 std::vector / std::string 对象。

例如，constexpr helloWorld 的定义就是 C++20 标准不允许的。：

```
constexpr auto helloWorld()
{
    std::string s1{"hello "};
    std::string s2{"world"};
    std::string s3 = s1 + s2;
    return s3;
}
```

但是仅函数内部使用 std::vector/std::string，new/delete，则这又是 constexpr 函数所允许的。举例如下：

```
// 例1：基于new/delete采用constexpr函数求和
constexpr int sum(int n)
{
    auto p = new int[n];

    std::iota(p, p + n, 1);

    auto t = std::accumulate(p, p + n, 0);
    
    delete[] p;
    return t;
}

static_assert(sum(10) == 55);
```

```
// 例2：基于vector采用constexpr函数求和
constexpr int sum(int n) 
{
    std::vector<int> v(10);

    std::iota(v.begin(), v.end(), 1);
    auto t = std::accumulate(v.begin(), v.end(), 0);

    return t;
}

static_assert(sum(10) == 55);
```

编译时多态
-----

constexpr 可以在编译时内存分配，那编译时多态有了应用条件。与此同时在编译期使用 dynamic_cast 和 typeid 也是可行的。

```
struct Box
{
    double width{0.0};
    double height{0.0};
    double length{0.0};
};

struct Product
{
    constexpr virtual ~Product() = default;
    constexpr virtual Box getBox() const noexcept = 0;
};

struct Notebook : public Product
{
    constexpr ~Notebook() noexcept {};
    constexpr Box getBox() const noexcept override
    {
        return {.width = 30.0, .height = 2.0, .length = 30.0};
    }
};

struct Flower : public Product
{
    constexpr Box getBox() const noexcept override
    {
        return {.width = 10.0, .height = 20.0, .length = 10.0};
    }
};

constexpr bool canFit(const Product &prod, const Box &minBox)
{
    const auto box = prod.getBox();
    return box.width < minBox.width && box.height < minBox.height && box.length < minBox.length;
}

int main()
{
    constexpr Notebook nb;
    constexpr Box minBox{100.0, 100.0, 100.0};
    static_assert(canFit(nb, minBox));
}
```

允许 try-catch
------------

C++20 constexpr 函数虽然允许 try-catch，但是这依然是有限制的，开发者不能在函数中 throw 异常。

允许变更 union 活跃成员
---------------

允许变更 union 活跃成员是 C++20 标准 [P1330R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1330r0.pdf) 提案，此提案的核心目标是允许在 constexpr 函数中重新指定 union 的当前有效成员。

```
union Foo 
{
	int i;
	float f;
};
constexpr int use() 
{
	Foo foo{};
	foo.i = 3;
	foo.f = 1.2f; // C++20之前不合法，C++20标准合法
	return 1;
}

static_assert(use());
```

consteval
---------

constexpr 修饰函数仅表示支持在编译期求值（是否真的在编译期求值，不确定），但是在有些时候我们要求必须在编译期求值。这就是 consteval 引入的价值。consteval 修饰函数，要求函数必须在编译期求值。

```
consteval int min(std::initializer_list<int> array)
{
    int low = std::numeric_limits<int>::max();

    for (auto& i : array)
    {
        if (i < low)
        {
            low = i;
        }
    }

    return low;
}

static_assert(min({ 1, 2, 3 }) == 1);
```

constinit
---------

constinit 修饰变量，保证变量在编译期初始化。目标是为了解决 Static Initialization Order Fiasco 文档，即相互影响的静态存储周期的变量之间，由于动态初始化的不确定性而导致的问题。

*   constinit 不能和 consteval/constexpr 同时使用
*   constinit 修饰引入对象，此时 constinit 与 constexpr 等价
*   只能使用 constexpr 或 consteval 函数初始化 constinit 变量

```
constexpr int sum(int n)
{
    auto p = new int[n];

    std::iota(p, p + n, 1);

    auto t = std::accumulate(p, p + n, 0);
    
    delete[] p;
    return t;
}

consteval int min(std::initializer_list<int> array)
{
    int low = std::numeric_limits<int>::max();

    for (auto& i : array)
    {
        if (i < low)
        {
            low = i;
        }
    }

    return low;
}

constinit auto g_min = min({ 1, 2 });
constinit auto g_sum = sum(2);

int main()
{
    static_assert(min({ 1, 2, 3 }) == 1);
	return 0;
}
```