cpp 新增的标准库
----------
## 1 容器库

### 1.1 array ###
std::array 是std::vector的一个弱化。在初始化时就必须指定大小，且不会动态增长；其分配的空间是栈上的数组，而不是堆上。


```cpp
std: : array<int, 4> arr= {1, 2, 3, 4};
constexpr int len = 4;
std: : array<int, len> arr = {1, 2, 3, 4};

void foo( int *p, int len) {
	return;
}

foo(&arr[0] , arr. size() )); // 非法
foo(arr. data(), arr. size());
std::sort(arr.begin() , arr.end());

```

### 1.2 std::forward_list ###
std::forward_list 相当于std::list来说，其实现的为单链表，相当与没进标准的slist。
不提供size()方法。

其使用方法跟list一致，transform之类的方法可能比较受限。详细用法参考 [如何使用std::forward_list 单链表](https://elloop.github.io/c++/2015-12-25/learning-using-stl-7-std-forward-list)

```cpp
#include <forward_list>
#include <iostream>
using namespace std;
int main()
{
    forward_list<int> fl = {1, 2, 3, 4, 5};
    for (const auto &elem : fl) {
        cout << elem;
		}
    return 0;
}
```
### 1.3 无序容器 ###
如下两组无序容器
std::unordered_map|std::unordered_multimap 和 std::unordered_set|std::unordered_multiset 
只是将C++03未进入标准的hash_map等容器标准化了而已。

用法和原有的std::map | std::multimap | std::set | set::multiset 基本类似。

hash_map的底层是hash存储，用开链法解决冲突。
map 和set底层结构是rbtree，插入和查找的效率不同
### 1.4 std::variant
std::variant是类型安全的联合体，是一个加强版的 union，variant支持更加复杂的数据类型，例如map，string等等。

std::variant 是 C++17 中，一个新加入标准函式库的 template 容器；他的概念基本上是和 union（参考）一样，是一个可以用来储存多种型别的容器。

比如说：
std::variant<int, double> v;

就代表 v 这个变数，可以用来储存 int 或 double 的变量，variant 内部自己会去记录相关的信息。
而和 union 不同的地方，variant 也是 type-safe 的，再加上有许多函式可以搭配使用，所以在使用上应该算是相对安全；另外也由于他是标准函式库的 template class，在使用时不需要另外去宣告一个新的型别。

### 1.4 std::optional

[std::optional](https://zh.cppreference.com/w/cpp/utility/optional) 是一个工具类，但是也可以看做是一个容器，管理一个可选的容纳值，即可以存在也可以不存在的值。在C++17中引进。

一种常见的 optional 使用情况是一个可能失败的函数的返回值，主要有如下方法
```cpp
has_value()   // 检查对象是否有值
value()       // 返回对象的值，值不存在时则抛出 std::bad_optional_access 异常
value_or()    // 值存在时返回值，不存在时返回默认值
```
具体的代码如下
```cpp
auto create_string(bool b) {
    return b ? std::optional<std::string>{"Godzilla"} :
		std::nullopt;
}
auto name = create_string(false);
// 支持bool operator() operator -> operator * 三种操作
if (name) {
	printf("%s\t%u\n", name->c_str(), (*name).size())
}
```
### std::any
std::any是一个类型安全的可以保存任何值的容器

##  std::function ##
C++11 std::function 是一种通用、多态的函数封装。
### std::string_view

c++17中的string_view 和 boost类似，string_view可以理解成原始字符串一个只读引用。 string_view 本身没有申请额外的内存来存储原始字符串的data，仅仅保存了原始字符串地址和长度等信息。 在很多情况下，我们只是临时处理字符串，本不需要对原始字符串的一份拷贝。

使用string_view可以减少不必要的内存拷贝，可以提高程序性能。相比使用字符串指针，string_view做了更好的封装。

需要注意的是，string_view 由于没有原始字符串的所有权，使用string_view 一定要注意原始字符串的生命周期。

当原始的字符串已经销毁，则不能再调用string_view。

### 1.5 std::tuple ###
元组是从pytnon（存疑）中引入的，用于存放不同类型的数据容器，是std::pair的增强版，需要在编译器确定类型。

在C++中是不支持多个返回值的，往往需要用于用户定义一个struct结构体。std::tuple提供了这样的能力。

基本操作有三种
std::make_tuple // 封装元组
std::tie // 解包
std::get<> // 获取其中某些元素
```cpp
auto get_student( int id) {
	std::make_tuple(3. 8, ' A' , "rayan");
}
double gpa;
char grade;
std::string name;
// 通过tie拆包
std::tie(gpa, grade, name) = get_student( 1) ;
std::tie(gpa, std::ignore, name) = get_student(0) ;
// 通过get解元素
auto student = get_student(0) ;
gpa = std::get<0>(student);
grade = std::get<1>(student);
name = std::get<2>(student);
// c++14提供根据type类获取，但tuple中的元素类型需要不一样，否则出发编译错误
grade = std::get<decltype(grade)>(student);

```

两个tuple可以通过std::tuple_cat进行合并

```cpp
auto tuple_list = std::tuple_cat(get_student(0), get_student(1));
// 获取tuple的大小
template <typename T>
auto tuple_len( T &tpl) {
	return std: : tuple_size<T>: : value;
}
// tuple的遍历 std::get<0>中的0必须是编译期确定
// 所以为了遍历必须有boost::varint的支持，具体代码如下
#include <boost/variant. hpp>
template <size_t n, typename. . . T>
boost::variant<T...> iterative_tuple_index( size_t i, const std::tuple<T... >& tpl) {
if ( i == n)
	return std::get<n>( tpl) ;
else if ( n == sizeof...(T) - 1)
	throw std::out_of_range( " 越界. ") ;
else
	return iterative_tuple_index<( n < sizeof...( T) - 1 ? n+1 : 0) >( i, tpl) ;
}
template <typename...T>
boost::variant<T...> tuple_index( size_t i, const std::tuple<T...>& tpl) {
	return iterative_tuple_index<0>( i, tpl) ;
}
// 迭代
for( int i = 0; i ! = tuple_len( new_tuple) ; ++i)
// 运行期索引
std::cout << tuple_index( i, new_tuple) << std::endl;

```


## 2 资源管理
### 2.1 引用计数与智能指针 ###
RAII 资源获取即初始化技术
C++11 引入了智能指针的概念，使用了引用计数的概念，让程序员不需关心手动释放内存。
基本想法是对于动态分配的对象，进行引用计数，每当增加一次对同一个对象的引用，那么引用对象的引用计数就会增加一次，每删除一次引用，引用计数就会减一， 当一个对象的引用计数减为零时， 就自动删除指向的堆内存。

这些指针都包含在<memory>中
std::shared_ptr|std::unique_ptr|std::weak_ptr.

### 2.1 shared_ptr
```cpp
auto pointer = std::shared_ptr<int>(new int(10));
auto pointer = std::make_shared<int>(10); //推荐用法
pointer.reset(); // 减少一次内存引用
pointer.get_count();//查看引用次数
```
enable_shared_from_this 的原型为
```cpp
template< class T > class enable_shared_from_this;
```

主要是为了[保活](https://blog.csdn.net/caoshangpa/article/details/79392878)

在异步调用中，存在一个保活机制，异步函数执行的时间点我们是无法确定的，然而异步函数可能会使用到异步调用之前就存在的变量。为了保证该变量在异步函数执期间一直有效，我们可以传递一个指向自身的share_ptr给异步函数，这样在异步函数执行期间share_ptr所管理的对象就不会析构，所使用的变量也会一直有效了（保活）。
### 2.2 unique_ptr
std::unique_ptr 有些库的实现为std::scoped_ptr，用于替代有问题的auto_ptr，表示独享的智能指针，通过禁止拷贝来禁止与其它智能指针共享同一个对象。
```cpp
// From C++14
std::unique_ptr<int> pointer = std::make_unique<int>(10);
template<typename T, typename ... Args>
std::unique_ptr<T> make_unique( Args&& ... args ) {
return std::unique_ptr<T>( new T( std::forward<Args>(args)...)) ;
}
```
std::unique_ptr 还可以[自定义释放函数](https://stackoverflow.com/questions/26360916/using-custom-deleter-with-unique-ptr)，替代默认的 delete 函数，可以用于模拟golang中的defer语义

```cpp
#include <stdio.h>
#include <memory>
int main() {
		std::unique_ptr<FILE, decltype(&fclose)> fp(fopen("test.txt", "r"), &fclose);
    if(fp) {
        char buf[4096];
        while(fgets(buf, sizeof(buf), fp.get())) {
            printf("%s", buf);
        }
    }
}
```
对于某些释放函数可以直接用decltype，但需要注意签名需要匹配，如上 decltype(fclose) 是编译不过的。


### 2.3 weak_ptr
std::weak_ptr 是为了解决两个对象A，B的成员变量相互持有对方的shared_ptr智能指针。
std::weak_ptr 没有 * 运算符和 -> 运算符， 所以不能够对资源进行操作，它的唯一作用就是用 于检查 std::shared_ptr 是否存在， expired()方法在资源未被释放时， 会返回 true, 否则返回false。

TODO 补充示例
## 3. 随机生成引擎（Random Engine）
stateful generator that generates random value within predefined min and max. Not truely random -- pseudorandom

### 3.1 default_random_engine

```cpp
#include <random>
std::default_random_engine eng; //

cout << "Min: " << eng.min() << endl; 
cout << "Max: " << eng.max() << endl;
cout << "Max: " << eng() << endl;
std::stringstream state;
state << eng;  // Save the state
state >> eng;  // Restore the state
// with seed
unsigned seed = std::chrono::steady_clock::now().time_since_epoch().count();
std::default_random_engine e3(seed);

vector<int> d =  {1,2,3,4,5,6,7,8,9};
std::shuffle(d.begin(), d.end(), e3);
```

### 3.2 uniform_int_distribution

```cpp
std::uniform_int_distribution<int> distr(0,100);  // range: [0~100]
std::default_random_engine eng;
cout << distr(e) << endl;
std::uniform_real_distribution<double> distrReal(0,100); // Range: [0~100)
std::poisson_distribution<int> distrP(1.0);  //  mean (double) 
std::normal_distribution<double> distrN(10.0, 3.0);  // mean and standard deviation
```
## 正则表达式 ##
正则表达式是一种用于在字符串中匹配模式的微型语言。


##  时间相关 ###
相关主要在chrono库，需要#include<chrono>，其所有实现均在std::chrono namespace下。
chrono是一个模版库，使用简单，功能强大，只需要理解三个概念：duration、time_point、clock
### Durations
std::chrono::duration 表示一段时间。
```cpp
template <class Rep, class Period = ratio<1> > class duration;
template <intmax_t N, intmax_t D = 1> class ratio;
ratio<3600, 1>         hours
ratio<60, 1>           minutes
ratio<1, 1>            seconds
ratio<1, 1000>         microseconds
ratio<1, 1000000>      microseconds
ratio<1, 1000000000>   nanosecons
```
由于各种duration表示不同，chrono库提供了duration_cast类型转换函数。
```cpp
template <class ToDuration, class Rep, class Period>
constexpr ToDuration duration_cast (const duration<Rep,Period>& dtn);
```
如下是一些示例

```cpp
auto start = std::chrono::high_resolution_clock::now();
//...do something
auto end = std::chrono::high_resolution_clock::now();
typedef std::chrono::microseconds us;
auto diff = std::chrono::duration_cast<us>(end - start);
cout<< diff.count()<<endl;

```
### time_point
std::chrono::time_point 表示一个具体时间。
```cpp
template <class Clock, class Duration = typename Clock::duration>  class time_point;
```

### Clock
std::chrono::system_clock 它表示当前的系统时钟，系统中运行的所有进程使用now()得到的时间是一致的。
每一个clock类中都有确定的time_point, duration, Rep, Period类型。
操作有：
now() 当前时间 time_point
to_time_t()   time_point 转换成time_t秒
from_time_t() 从time_t转换成time_point
典型的应用是计算时间日期：
```cpp
// system_clock example
#include <iostream>
#include <ctime>
#include <ratio>
#include <chrono>
 
int main ()
{
  using std::chrono::system_clock;
 
  std::chrono::duration<int,std::ratio<60*60*24> > one_day (1);
 
  system_clock::time_point today = system_clock::now();
  system_clock::time_point tomorrow = today + one_day;
 
  std::time_t tt;
 
  tt = system_clock::to_time_t ( today );
  std::cout << "today is: " << ctime(&tt);
 
  tt = system_clock::to_time_t ( tomorrow );
  std::cout << "tomorrow will be: " << ctime(&tt);
 
  return 0;
}
```

## 参考文档
http://www.umich.edu/~eecs381/handouts/handouts.html