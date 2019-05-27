cpp 学习笔记
------
根据
[聊聊C++11](https://www.bilibili.com/video/av7701532) 的内容整理而来
新的特性
## 1. initializer_list ##

```
// c++03
vector<int> a;
int array [] = {1, 2, 3}
for(int i = 0; i < 3; ++i) {
    a.push_back(arry[i]);
}

// c++11
#include <initializer_list>
vector<int> v = {1, 2, 3}
```
在初始化时可以避免重复写代码。


## 2. Uniform Initialziation
cpp在语言的层面为类对象的初始化与普通数组以及POD的初始化方法提供了统一的桥梁，极大的改善了写模板。

搜索函数时有最高的优先级，用户可以自定义

```cpp
#include <initializer_list>
class Dog {
    Dog(const initializer_list<int>& vec);
    Dog(int age);
};
```

## 3. auto ##
auto 在很早以前就已经进入了 C++， 但是他始终作为一个存储类型 的指示符存在， 与 register 并存。 在传统 C++ 中 ， 如果一个变量没有声明为 register变量， 将自动被视为一个 auto 变量。 而随着register 被弃用 ， 对 auto 的语义变更也就非常自然了 。

可以用如下语法糖进行遍历容器

```cpp
vector<int> v = {1, 2, 3}
for(auto it = v.begin(); it != v.end(); ++it) {
}
```

auto避免了需要写冗长类型的麻烦，用法vector迭代器的肯定知道我在说没什么。
后面还有auto的方法会加以补充。


## 4. foreach ##
foreach 严格来说还是一种语法糖，从其它语言（python）中借鉴而来，具体如下

```cpp
for(int el : v) {
    // copy
}
for(auto & el : v) {
    ++el
}
// c++17 TODO##
for(auto key, value : map) {
	
}
```
## 5. nullptr ##

nullptr的为了解决如下链接时冲突的问题，在C++11之后的代码中，严格来说，没有NULL。nullptr会提供更安全的检查。

```cpp
void for(int i);
void for(char* ptr);
foo(NULL) //ambiguity
for(nullptr) // c++11
```
## 6. enum class ##

解决各个enum变量可比较（被拉平为整形），可能带来的风险，为了代码更安全。

```cpp
enum class Apple  {Green, Red};
enum class Orange {Green, Red};
auto a = Apple::Green;
auto o = Orange::Green;
if (a == o) // compiler fails, no defining opertor == 
```
但是依然如下强转的方式输出
```cpp
#include <iostream>
template<typename T>
std::ostream& operator<<(typename std::enable_if<std::is_enum<T>::value, std::ostream>::type& stream, const T& e)
{
    return stream << static_cast<typename 
    std::underlying_type<T>::type>(e);
}
```

## 7. static assert ##
static_assert 提供编译期的断言，对平台的移植方面会有较强的作用

```cpp
static_assert(sizeof(int) == 4);
```

## 8. delegation Constructor ##

即构造函数可以相互调用，类似于Java中的Super语义。

```cpp
class Dog {
 public:
    Dog() {}
    Dog(int a) : Dog() {
        other();
    }
};
```

## 9. override (for vritual funciton) ##
避免 inavertently（不经意间的）在子类中创建新的函数
用于给编译器一个参考
```cpp
class Dog {
 public:
    virtual void Bark(int a);       //
};
class RedDog : public Dog {
    virtual void Bark(float a) override; // no funciton to override
    virtual void Bark(double a ); // create new funciton
};
```

## 10. final ##
既可用在函数，和可用在类上，从Java中借鉴而来，表示该类不可再被继承或函数不能被重载。
```cpp
class Dog final { //no class derived from Dog
    virtual void Bark() final; // no class can override
};
```
## 11. Compiler Generate Default Constructor ##

```cpp
class Dog {
    Dog() == default; // compiler generator code
};
Dog toy;
```

声明一个Empty类，编译器会生成什么？

在C++03中，默认下会生成(也有教程上说，只有在有必要时会生成，如何验证？)

```cpp
class Empty {
    Empty();
    Empty(const Empty& rhs);
    Empty& operator=(const Empty &rhs)
    ~Empty();
};
```
C++11之后添加了Move语义，所以多了如下[移动构造函数](https://zh.cppreference.com/w/cpp/language/move_constructor)。
```cpp
class Empty {
    Empty(Empty&& rhs);
    Empty& operator=(Empty&& rhs)
}
```

## 12. delete Constructor ##

```cpp
class Dog {
    Dog(int a) {};
    Dog(double) = delete; // 禁止从double隐式转为int
    Dog& opertor=(const Dog&) = delete; //禁止赋值
};
```
## 13. constexpr ##

在编译期间可以计算的常量表达式

```cpp
constexpr int cubed(int x) {return x*x*x}
int y = cubed(191); // computed at compile time
```
从 C++14 开始， constexptr 函数可以在内 部使用 局部变量、 循环和分支等简单语句，例如下面的代码在 C++11 的标准下是不能够通过编译
```cpp
constexpr int fibonacci( const int n) {
	if( n == 1) return 1;
	if( n == 2) return 1;
	return fibonacci( n- 1) +fibonacci( n- 2) ;
}
```

## 14. New String Literals ##
提供更好的国际化（UTF-8）支持。

```cpp
char*       a = u8"string" // UTF-8 string
char16_t*   b = u"string"
char32_t*   c = U"string"
char*       d = R"string \\" // raw string
```

## 15. lamda function ##

Lambda 表达式是 C++11 中最重要的新特性之一。本质上来说，通过“语法糖”的方式提供了一种匿名函数的特性。

从函数式编程中借鉴而来，熟悉stl的人应该记得std::mem_fn等函数多么的trick。
基本语法为
```
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {
// 函数体
}
```
捕获列表，其实可以理解为参数的一种类型。默认情况下，lambda表达式内部不能使用函数体外部的变量，捕获列表起到传递外部数据的作用。C/C++中参数涉及到值传递、引用（左值|右值）传递，如下分情况讨论

**1. 值捕获**

与参数传值类似，值捕获的前期是变量可以拷贝，不同之处则在于，被捕获的变量在 lambda 表达式被创建时拷贝，而非调用时才拷贝。这意味着，如果在调用前值被修改，lambda中保存的依然是修改前的值。
```cpp
void learn_lambda_func_1() {
    int value_1 = 1;
    auto copy_value_1 = [value_1] {
        return value_1;
    };
    value_1 = 100;
    // stored_value_1 == 1 被绑定为之前的值
    auto stored_value_1 = copy_value_1();
}
```

**2. 引用捕获**
与引用传参类似，引用捕获保存的是引用，相当于指针，值会发生变化。

```cpp
void learn_lambda_func_2() {
    int value_2 = 1;
    auto copy_value_2 = [&value_2] {
        return value_2;
    };
    value_2 = 100;
    // stored_value_2 == 100 相当于拷贝为指针(地址)
    auto stored_value_2 = copy_value_2();
}
```
**3. 隐式捕获**
在捕获列表中写一个 & 或 = 向编译器声明采用引用捕获或者值捕获，以避免手工撰写。
如下是四种最为常见的捕获

* [] 空捕获列表
* [name1, name2, ...] 捕获一系列变量
* [&] 引用捕获, 让编译器自行推导捕获列表
* [=] 值捕获, 让编译器执行推导应用列表

**注意**
值和引用捕获在C++11中只能捕获左值，在C++14可以捕获右值（如何理解呢？）

一般的代码举例

```cpp

auto f = [](int x, int y) {return x + y;}
f(3,4)
map(f, vector);
```
## 16. 自定字符常量 ##

常量有如下四种
1. 整形 4l 4UL
2. 浮点 4.5f
3. 字符 'z'
4. 字符串 "string"

为常量提供符号重载，通过例子看，作用不大（希望被打脸）。

```cpp
constexpr double operator"" _cm (double x) {return x*10;}
constexpr double operator"" _mm(double x) {return x;}

double height = 3.4_cm;
cout<<3.4_cm/3_mm<<endl;

```
## 17. shared_ptr ##

shared_ptr是智能指针的一种，用于资源管理。


```cpp
class Dog {
    Dog(std::string&& name);
};

shared_ptr<Dog> p = make_shared<Dog>("name");
shared_ptr<Dog> p2 = make_shared<Dog>("name",
                    [](Dog* p) {
                        cout<<"custom deleting..."<<endl;
                        delete p;
                    });
p1=p2; //
p1=nullptr; // 对象释放
p1.reset(); // 对象释放
Dog* d = p1.get(); //获取裸指针


```
## 18. decltype ##
**decltype** 是为了解决auto只能对变量进行类型推导。用法跟**sizeof** 很类似，在模板推导中有大量的应用。

```cpp
template<typename R, typename T, typename U>
R add(T x, U y) {
    return x+y
}
//C++11中的尾递归，用auto来锚定解析
template<typename T, typename U>
auto add(T x, U y) -> decltype(x+y) {
    return x+y;
}
// C++14 更加简单和自然
template<typename T, typename U>
auto add(T x, U y) {
    return x+y
}
```
## 19. using  ##

### 19.1 [类型别名模板]

模板是用来产生类型的，但在传统中typedef 为类型定义一个新的名称

```cpp
template<typename T, typename U>
class SuckType;
typedef SuckType<std::vector, std::string> NewType; // 不合法
using NewType=SuckType<std::vector, std::string>; // 合法

typedef int (*sql_fn)(std::string *);
using sql_fn = int(*)(std::string *); //更加直观

```
### 19.2 继承构造

```cpp
class Base {
public:
    int value1;
    int value2;
    Base() {
        value1 = 1;
    }
    Base(int value) : Base() {//构造函数
    value2 = 2;
    }
};
class Subclass : public Base {
public:
    using Base::Base; // 继承构造
};
```
## 20 函数对象 ##

###20.1 std::funciton

C++11 std::function 是一种通用、多态的函数封装， 它的实例可以对任何可以调用的目标实体进行存储、复制和调用操作， 它也是对 C++中 现有的可调用实体的一种类型安全的包裹（ 相对来说， 函数指针的调用不是类型安全的） ， 换句话说， 就是函数的容器 。 
```cpp
int foo( int para) {
return para;
}
std::function<int( int) > func = foo;
std::cout << func( 10) << std::endl;

```
###20.2 std::bind

std::bind 是用来绑定函数调用的参数的，它解决的需求是我们有时候可
能并不一定能够一次性获得调用某个函数的全部参数，通过这个函数，我们可以将部分调用参数提前绑定到函数身上成为一个新的对象

```cpp
int foo( int a, int b, int c) {
	return a+b+c;
}
auto bindFoo = std::bind( foo, std::placeholders::_1, 1, 2) ;
std::cout << bindFoo(1)<< std::endl;
```
需要注意的是在异步编程时，可能需要注意bind变量的生命周期。



##21. 模板增强 ##

###21.1 外部模板
###21.2 默认模板参数
避免每次指定参数
```
template<typename T = int, typename U = int>
auto add( T x, U y) - > decltype( x+y) {
	return x+y
}
```
### 21.3. 变长模板参数 ##
在 C++11 之前，
无论是类模板还是函数模板，都只能按其指定的样子，接受一组固定数量的模板参
数；而 C++11 加入了新的表示方法，允许任意个数、任意类别的模板参数，同时
也不需要再定义时将参数的个数固定。
```cpp
template<typename... Ts>
class Magic;
template<typename Require, typename... Args>
class RequireMagic;
```




此处的暂时只要求看懂代码就行，后面会补充解包模板参数的方法等示例



##右值引用

纯右值(prvalue, pure rvalue)， 纯粹的右值， 要么是纯粹的字面量， 例 如 10 ,true ； 要么是求值结果相当于字面量或匿名临时对象， 例 如 1+2 。 非引 用 返回
的临时变量、 运算表达式产生的临时变量、 原始字面量、Lambda 表达式都属 于纯右值。
将亡值(xvalue, expiring value)， 是 C++11为了引入右值引用 而提出的概念（ 因此在传统 C++中 ， 纯右值和右值是统一个概念） ， 也就是即将被销毁、 却能够被移动的值。


```
std::vector<int> foo( ) {
std::vector<int> temp = {1, 2, 3, 4};
return temp;
}
std::vector<int> v = foo( ) 
```
v 是左值、 foo( ) 返回的值就是右值（ 也是纯右值）


## 左值和右值
什么是左值，什么是右值？
能在左边的表达式，即有存储空间，能够被修改，有名字，被取地址。
```
T   t1;         // 类型 T
T&  t2 = t1;    // T&  表示 T 的左值引用类型，t2 是左值引用类型的变量，它引用的是一个左值
T&& t3 = T();   // T&& 表示 T 的右值引用类型，t3 是右值引用类型的变量，它引用的是一个右值

T&  t4 = T();   // err: 左值引用 不能绑定一个 右值
const T& t7 = T();  // 常量引用类型是“万能”的引用类型
```
如上的策略都可以用来快速定位是左值，但也不是绝对的，需要具体具体分析。

引用折叠规则
```
所有的右值引用叠加到右值引用上仍然还是一个右值引用。（T&& && 变成 T&&）
所有的其他引用类型之间的叠加都将变成左值引用。 （T& &, T& &&, T&& & 都变成 T&）
对常量引用规则一致
```




int a = 5; // 左值
[右值](https://github.com/imhuay/Algorithm_Interview_Notes-Chinese/blob/master/C-%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80/Cpp-C-%E5%B7%A6%E5%80%BC%E4%B8%8E%E5%8F%B3%E5%80%BC.md)


## Move语义 ##
move 简单来说是为了解决性能拷贝的性能损耗而产生的。

std::move() 的主要作用是将一个左值转为 xvalue（右值）, 其实现本质上是一个 static_cast<T>。


```

```
## 完美转发 ##

std::forward() 主要用于实现完美转发，其作用是将一个类型为（左值/右值）引用的左值，转化为它的类型所对应的值类型（左值/右值）

## 并行编程 ##
[std::promise](https://en.cppreference.com/w/cpp/thread/promise) 对象可以保存某一类型 T 的值，该值可被[std::future](https://en.cppreference.com/w/cpp/thread/future)对象读取（可能在另外一个线程中），因此 promise 也提供了一种线程同步的手段。

TODO 需要检查一下精度问题





### std::future ###
[future](https://en.cppreference.com/w/cpp/thread/future)
程序不太符合预期

```cpp
#include <future>
#include <iostream>

bool is_prime(int x)
{
  for (int i=0; i<x; i++)
  {
    if (x % i == 0)
      return false;
  }
  return true;
}

int main()
{
  std::future<bool> fut = std::async(is_prime, 700020007);
  std::cout << "please wait";
  std::chrono::milliseconds span(100);
  while (fut.wait_for(span) != std::future_status::ready)
    std::cout << ".";
  std::cout << std::endl;
  bool ret = fut.get();
  std::cout << "final result: " << ret << std::endl;
  return 0;
}
```
### std::promise ###

```cpp
std::promise<int> prom;
std::future<int> fut = prom.get_future();
```
