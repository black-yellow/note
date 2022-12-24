> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_43331089/article/details/124340554)

C++ [单例](https://so.csdn.net/so/search?q=%E5%8D%95%E4%BE%8B&spm=1001.2101.3001.7020)模式
======================================================================================

一 、单例模式
-------

单例可能是最常用的简单的一种设计模式，实现方法多样，根据不同的需求有不同的写法; 同时单例也有其局限性，因此有很多人是反对使用单例的。本文对 C++ 单例的常见写法进行了一个总结, 包括**懒汉式**、**线程安全**、单例模板等； 按照从简单到复杂，最终回归简单的的方式循序渐进地介绍，并且对各种实现方法的局限进行了简单的阐述，大量用到了 C++ 11 的特性如**智能指针**, **magic static**，**线程锁**; 从头到尾理解下来，对于学习和巩固 C++ 语言特性还是很有帮助的。

### 1、什么是单例

单例 Singleton 是设计模式的一种，其特点是只提供唯一一个类的实例, 具有全局变量的特点，在任何位置都可以通过接口获取到那个唯一实例; 具体运用场景如：

*   设备管理器，系统中可能有多个设备，但是只有一个设备管理器，用于管理设备驱动;
    
*   数据池，用来缓存数据的数据结构，需要在一处写，多处读取或者多处写，多处读取;
    

### 2、 什么时候使用单例模式

你需要系统中只有唯一一个实例存在的类的全局变量的时候才使用单例。

### 3、C++ 单例模式的实现

**基础要点**

*   全局只有一个实例：`static` 特性，同时禁止用户自己声明并定义实例（把构造函数设为 private）
    
*   线程安全
    
*   禁止赋值和拷贝
    
*   用户通过接口获取实例：使用 static 类成员函数
    

五种实现方式
------

### （1）有缺陷的懒汉式

懒汉式 (Lazy-Initialization) 的方法是直到使用时才实例化对象，也就说直到调用 geInstance() 方法的时候才 new 一个单例的对象。好处是如果被调用就不会占用内存。

```
/*
 * C++设计模式之单例模式
 *
 * version1:
 * with problems below:
 * 1. thread is not safe
 * 2. memory leak
 * */
 
#include <iostream>
 
using namespace std;
 
class Singleton{
private:
    static Singleton* m_pInstance;
 
private:
    Singleton(){
        cout << "constructor called!" << endl;
    }
 
    Singleton(Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
 
public:
    ~Singleton(){
        cout << "destructor called!" << endl;
    }
 
    static Singleton* getInstance() {
        if (m_pInstance == nullptr) {
            m_pInstance = new Singleton;
        }
 
        return m_pInstance;
    }
};
 
Singleton* Singleton::m_pInstance = nullptr;
 
int main() {
    Singleton* instance1 = Singleton::getInstance();
    Singleton* instance2 = Singleton::getInstance();
 
    return 0;
}
```

输出结果：

```
constructor called!
```

可以看到，获取了两次类的实例，却只有一次类的构造函数被调用，表明只生成了唯一实例，这是个最基础版本的单例实现，他有哪些问题呢？

1.  **线程安全的问题**, 当多线程获取单例时有可能引发竞态条件：第一个线程在 if 中判断 m_pInstance 是空的，于是开始实例化单例; 同时第 2 个线程也尝试获取单例，这个时候判断 m_pInstance 还是空的，于是也开始实例化单例; 这样就会实例化出两个对象, 这就是线程安全问题的由来; **解决办法**: 加锁

*   **内存泄漏**. 注意到类中只负责 new 出对象，却没有负责 delete 对象，因此只有构造函数被调用，析构函数却没有被调用; 因此会导致内存泄漏。**解决办法**： 使用共享指针;

因此，这里提供一个改进的，线程安全的、使用[智能指针](https://so.csdn.net/so/search?q=%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88&spm=1001.2101.3001.7020)的实现;

### （2）线程安全、内存安全的懒汉式单例 （智能指针，锁）

```
#include <iostream>
#include <memory> // shared_ptr
#include <mutex>  // mutex
 
// version 2:
// with problems below fixed:
// 1. thread is safe now
// 2. memory doesn't leak
 
class Singleton{
public:
    //typedef std::shared_ptr<Singleton> Ptr;
    using Ptr = std::shared_ptr<Singleton>;
 
    ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
 
    Singleton(Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
 
    static Ptr getInstance(){
        // 双重锁
        if(m_pInstance==nullptr){
            std::lock_guard<std::mutex> lk(m_mutex);
 
            if(m_pInstance == nullptr){
                m_pInstance = std::shared_ptr<Singleton>(new Singleton);
            }
        }
 
        return m_pInstance;
    }
 
private:
    Singleton() {
        std::cout << "constructor called!" << std::endl;
    }
 
private:
    static Ptr m_pInstance;
    static std::mutex m_mutex;
};
 
// initialization static variables out of class
Singleton::Ptr Singleton::m_pInstance = nullptr;
std::mutex Singleton::m_mutex;
 
int main(){
    Singleton::Ptr instance1 = Singleton::getInstance();
    Singleton::Ptr instance2 = Singleton::getInstance();
 
    return 0;
}
```

输出结果

```
constructor called! destructor called!
```

`shared_ptr`和`mutex`都是 C++11 的标准，以上这种方法的优点是

基于 shared_ptr, 用了 C++ 比较倡导的 `RAII`思想，用对象管理资源, 当 shared_ptr 析构的时候，new 出来的对象也会被 delete 掉。以此避免内存泄漏。

加了锁，使用互斥量来达到线程安全。这里使用了两个 if 判断语句的技术称为双检锁；好处是，只有判断指针为空的时候才加锁，避免每次调用 get_instance 的方法都加锁，锁的开销毕竟还是有点大的。

不足之处在于： 使用智能指针会要求用户也得使用智能指针，非必要不应该提出这种约束; 使用锁也有开销; 同时代码量也增多了，实现上我们希望越简单越好。

还有更加严重的问题，在某些平台（与编译器和指令集架构有关），双检锁会失效！

因此这里还有第三种的基于 `Magic Staic`的方法达到线程安全

### （3）最推荐的懒汉式单例 ([magic](https://so.csdn.net/so/search?q=magic&spm=1001.2101.3001.7020) static )——局部静态变量

```
class Singleton
{
public:
    ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
 
    Singleton(const Singleton&)=delete;
    Singleton& operator=(const Singleton&)=delete;
 
    static Singleton& getInstance(){
        static Singleton instance;
        return instance;
    }
 
private:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }
};
 
int main(int argc, char *argv[])
{
    Singleton& instance_1 = Singleton::getInstance();
    Singleton& instance_2 = Singleton::getInstance();
 
    return 0;
}
```

这种方法又叫做 `Meyers' SingletonMeyer's`的单例， 是著名的写出`《Effective C++》`系列书籍的作者 Meyers 提出的。所用到的特性是在 C++11 标准中的 Magic Static 特性：

> If control enters the declaration concurrently while the variable is being initialized, the concurrent execution shall wait for completion of the initialization.
> 
> > 如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束。

**这样保证了并发线程在获取静态局部变量的时候一定是初始化过的，所以具有线程安全性。**

C++ 静态变量的生存期 是从声明到程序结束，这也是一种懒汉式。

这是**最推荐**的一种单例实现方式：**通过局部静态变量的特性保证了线程安全 , 不需要使用共享指针，代码简洁；注意在使用的时候需要声明单例的引用 Single& 才能获取对象。**

当然，C++11 后面出现`std::call_once` 就是解决这个问题，但是没有`magic static` 方便，所以推荐`magic static`的使用方法 。

另外网上有人的实现返回指针而不是返回引用

```
static Singleton* get_instance(){
    static Singleton instance;
    return &instance;
}
```

这样做并不好，理由主要是无法避免用户使用 delete instance 导致对象被提前销毁。还是建议大家使用返回引用的方式。

### （4）单例模式模板类的实现

```
#include <iostream>
 
template<typename T>
class Singleton{
public:
    static T& getInstance(){
        static T instance;
        return instance;
    }
 
    virtual ~Singleton(){
        std::cout<<"destructor called!"<<std::endl;
    }
 
    Singleton(const Singleton&)=delete;
    Singleton& operator =(const Singleton&)=delete;
 
protected:
    Singleton(){
        std::cout<<"constructor called!"<<std::endl;
    }
};
 
/********************************************/
// Example:
// 1.friend class declaration is requiered!
// 2.constructor should be private
 
class DerivedSingle : public Singleton<DerivedSingle>{
    // !!!! attention!!!
    // needs to be friend in order to
    // access the private constructor/destructor
    friend class Singleton<DerivedSingle>;
 
public:
    DerivedSingle(const DerivedSingle&)=delete;
    DerivedSingle& operator =(const DerivedSingle&)= delete;
 
private:
    DerivedSingle()=default;
};
 
int main(int argc, char* argv[]){
    DerivedSingle& instance1 = DerivedSingle::getInstance();
    DerivedSingle& instance2 = DerivedSingle::getInstance();
    return 0;
}
```

输出结果

```
constructor called! destructor called!
```

以上实现一个单例的模板基类，使用方法如例子所示意，子类需要**将自己作为模板参数 T** 传递给 Singleton 模板; 同时需要将基类声明为友元，这样才能调用子类的**私有构造函数**。

基类模板的实现要点是：

*   构造函数需要是 `protected`，这样**子类才能继承**；
    
*   使用了[奇异递归模板模式](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern) CRTP(Curiously recurring template pattern)
    
*   get instance 方法和 2.2.3 的 static local 方法一个原理。
    
*   在这里基类的析构函数可以不需要 virtual ，因为子类在应用中只会用 Derived 类型，**保证了析构时和构造时的类型一致**
    

### (5）不需要在子类声明友元的实现方法

在 [stackoverflow](https://codereview.stackexchange.com/questions/173929/modern-c-singleton-template) 上， 有大神给出了不需要在子类中声明友元的方法，在这里一并放出; 精髓在于使用一个代理类 token，子类构造函数需要传递 token 类才能构造，但是把 token 保护其起来， 然后子类的构造函数就可以是公有的了，这个子类只有 Derived(token) 的这样的构造函数，这样用户就无法自己定义一个类的实例了，起到控制其唯一性的作用。代码如下。

```
#include <iostream>
 
template<typename T>
class Singleton{
public:
    static T& getInstance() noexcept(std::is_nothrow_constructible<T>::value){
        static T instance{token()};
        return instance;
    }
 
    virtual ~Singleton() =default;
 
    Singleton(const Singleton&)=delete;
    Singleton& operator =(const Singleton&)=delete;
 
protected:
    struct token{}; // helper class
    Singleton() noexcept=default;
};
 
 
/********************************************/
// Example:
// constructor should be public because protected `token` control the access
 
class DerivedSingle : public Singleton<DerivedSingle>{
public:
    DerivedSingle(token){
        std::cout<<"destructor called!"<<std::endl;
    }
 
    ~DerivedSingle(){
        std::cout<<"constructor called!"<<std::endl;
    }
 
    DerivedSingle(const DerivedSingle&)=delete;
    DerivedSingle& operator =(const DerivedSingle&)= delete;
};
 
int main(int argc, char* argv[]){
    DerivedSingle& instance1 = DerivedSingle::getInstance();
    DerivedSingle& instance2 = DerivedSingle::getInstance();
    return 0;
}
```

总结
--

以上是单利模式的 5 种设计方案，建议使用第 3 种，如果项目大，需要单例复用，可以使用第 4 种。