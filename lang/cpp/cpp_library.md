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

### 1.5 std::optional

[std::optional](https://zh.cppreference.com/w/cpp/utility/optional) 是从C++17开始，引进一个工具类。
可以看做是一个容器，管理一个可选的容纳值，即可以存在，也可以不存在的值。

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
### 1.6 std::any
std::any是一个类型安全的可以保存任何值的容器

##  1.7 std::function ##
C++11 std::function 是一种通用、多态的函数封装。
### 1.8 std::string_view

C++17中的string_view 和 boost类似，string_view可以理解成原始字符串一个只读引用，近似于google开源代码中StringPiece类。
string_view 本身没有申请额外的内存来存储原始字符串的data，仅仅保存了原始字符串地址和长度等信息。 在很多情况下，我们只是临时处理字符串，本不需要对原始字符串的一份拷贝。

使用string_view可以减少不必要的内存拷贝，可以提高程序性能。相比使用字符串指针，string_view做了更好的封装。

需要注意的是，string_view 由于没有原始字符串的所有权，使用string_view 一定要注意原始字符串的生命周期。

当原始的字符串已经销毁，则不能再调用string_view。



### 1.9 std::tuple ###
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
## 4正则表达式 ##
正则表达式是一种用于在字符串中匹配模式的微型语言。


##  5. 时间相关 ###
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

## 6 多线程 ##

C++ 中关于并发多线程的部分，主要包含 \<thread>、\<mutex>、\<atomic>、\<condition_varible>、\<future>[五个部分](https://www.jianshu.com/p/81f0b071b3e0)。
* \<atomic>：该头文主要声明了两个类, std::atomic 和 std::atomic_flag，另外还声明了一套 C 风格的原子类型和与 C 兼容的原子操作的函数。
* \<thread>：该头文件主要声明了 std::thread 类，另外 std::this_thread 命名空间也在该头文件中。
* \<mutex>：该头文件主要声明了与互斥量(mutex)相关的类，包括 std::mutex 系列类，std::lock_guard, std::unique_lock, 以及其他的类型和函数。
* \<condition_variable>：该头文件主要声明了与条件变量相关的类，包括 std::condition_variable 和 std::condition_variable_any。
* \<future>：该头文件主要声明了 std::promise, std::package_task 两个 Provider 类，以及 std::future 和 std::shared_future 两个 Future 类，另外还有一些与之相关的类型和函数，std::async() 函数就声明在此头文件中。


### 6.1 atomic

#### 内存模型

[如何理解 C++11 的六种 memory order？](https://www.zhihu.com/question/24301047)



[atomic](https://zh.cppreference.com/w/cpp/atomic/atomic)
从原理上说，就是包装了GCC的 语义簇


```

```

### 6.2 thread

[thread](https://zh.cppreference.com/w/cpp/thread/thread) 构造函数为 callback f，后面的不定参数是f的参数，具体如下
```
template< class Function, class... Args > 
explicit thread( Function&& f, Args&&... args );
```

* thread 不可复制，但是可以move
* callback 可以是函数，类成员函数，跟bind的语义很类似。
* 到线程函数的参数被移动或按值复制，若需要传递引用参数给线程函数，则必须包装它（例如用 std::ref 或 std::cref ）

如下是一些示例

```cpp
void f2(int& n) {
    for (int i = 0; i < 5; ++i) {
        std::cout << "Thread 2 executing\n";
        ++n;
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}
int main() {
  int n = 5;
  std::thread t2(f2, std::ref(n));
  t2.join();
}
```

也可以定义线程数组，初始化的时候需要用到 move-assign 函数
```cpp
void f2(int& n) {
  std::cout <<std::this_thread::get_id() <<"\t" <<n <<"\n";
}
void thread_array_init() {
  std::thread task_array[4];
  for(auto i = 0; i < 4; ++i) {
    task_array[i] = std::thread(f2, std::ref(i));
  }
  for(auto i = 0; i < 4; ++i) {
    task_array[i].join();
  } 
}
void thread_vector_init() {
  std::vector<std::thread> tasks;
  for(auto i = 0; i < 4; ++i) {
    tasks.push_back(std::thread(f2, std::ref(i)));
  }
  for (auto& th : tasks){
    th.join();
  }
}
```

在linux下编译的时候，需要动态链接pthread，否则会报未实现的错误。


从原理上说，基本是将pthread_x 的接口按照C++的方式包装了一层，在C++11以前也有很多的类似thread类可以参考，标准只是更加规范化了而已。

### 6.3 lock

如下讨论的lock和timed_lock在linux下是基于pthread_x的封装。
[mutex](https://zh.cppreference.com/w/cpp/thread/mutex) 对应于 pthread_mutex_unlock pthread_mutex_lock

[recursive_mutex](https://zh.cppreference.com/w/cpp/thread/recursive_mutex)  对应于 pthread_mutex_lock，但初始化属性PTHREAD_MUTEX_RECURSIVE。

[shared_mutex](https://zh.cppreference.com/w/cpp/thread/shared_mutex) 对应于 pthread_rwlock 系列函数，表示读写锁。







#### 6.3.1 mutex

[mutex](https://zh.cppreference.com/w/cpp/thread/mutex) 提供了对共享数据免受从多个线程同时访问的同步原语。

通常不直接使用 std::mutex ，而通过 std::unique_lock 、 std::lock_guard 或 std::scoped_lock (C++17 起) 更加安全的方式管理其生命周期。
```cpp
std::mutex g_pages_mutex;
{ 
  std::lock_guard<std::mutex> guard(g_pages_mutex);
  ...
}
```

#### 6.3.2 recursive_mutex

[recursive_mutex](https://zh.cppreference.com/w/cpp/thread/recursive_mutex) 递归互斥锁是一个可锁的对象，就像互斥一样，但是允许同一个线程获得对互斥对象的多个级别的所有权，是为了解决 一个线程中可能在执行中需要再次获得锁，可能会导致死锁的问题。

根据这个问题[stdmutex-vs-stdrecursive-mutex-as-class-member](https://stackoverflow.com/questions/14498892/stdmutex-vs-stdrecursive-mutex-as-class-member) 下的回复，如果出现该情况，需要重新设计你的类。

#### 6.3.3 shared_mutex

[shared_mutex](https://zh.cppreference.com/w/cpp/thread/shared_mutex) 实际上是读写锁的封装，跟mutex一样，一般不直接使用它，通过 shared_lock 和  unique_lock 管理其周期。

```cpp
class ThreadSafeCounter {
 public:
  ThreadSafeCounter() = default;
 
  // 多个线程/读者能同时读计数器的值。
  unsigned int get() const {
    std::shared_lock<std::shared_mutex> lock(mutex_);
    return value_;
  }
 
  // 只有一个线程/写者能增加/写线程的值。
  void increment() {
    std::unique_lock<std::shared_mutex> lock(mutex_);
    value_++;
  }
 
  // 只有一个线程/写者能重置/写线程的值。
  void reset() {
    std::unique_lock<std::shared_mutex> lock(mutex_);
    value_ = 0;
  }
 private:
  mutable std::shared_mutex mutex_;
  unsigned int value_ = 0;
};
```

### 6.4 timed_lock

在lock的基础上，其有对应带超时版本的锁。

[timed_mutex](https://zh.cppreference.com/w/cpp/thread/timed_mutex) 锁比 [mutex](https://zh.cppreference.com/w/cpp/thread/mutex) 锁多了两个成员函数try_lock_for 和 try_lock_until。
区别在于try_lock_for 传递一个时间间隔，try_lock_until 传递一个未来的时间点。
```cpp
std::timed_mutex mtx;
void fireworks () {
  // waiting to get a lock: each thread prints "-" every 200ms:
  while (!mtx.try_lock_for(std::chrono::milliseconds(2))) {
    std::cout << "-";
  }
  // got a lock! - wait for 1s, then this thread prints "*"
  std::this_thread::sleep_for(std::chrono::milliseconds(100));
  std::cout << "*\n";
  mtx.unlock();
}

```

[shared_timed_mutex ](https://zh.cppreference.com/w/cpp/thread/shared_timed_mutex) 与 [recursive_timed_mutex](https://zh.cppreference.com/w/cpp/thread/recursive_timed_mutex) 类似，不再赘述。

### 6.5 lock-guard

在上面mutex的例子中已经介绍了用于管理锁声明周期的Guard，利用到了C++的 RAII，主要有如下类型

[lock_guard](https://zh.cppreference.com/w/cpp/thread/lock_guard) 实现了 [BasicLockable](https://zh.cppreference.com/w/cpp/named_req/BasicLockable)，即lock & unlock接口，除了构造函数外没有其他member function，使用起来比较简单，初始化的时候必须bind一个mutex。

[unique_lock](https://zh.cppreference.com/w/cpp/thread/unique_lock) 除了lock_guard的功能外，提供了更多的member_function，如延迟锁定、锁定的有时限尝试、递归锁定、所有权转移和与条件变量一同使用，需要付出更多的时间、性能成本。
与lock_guard的区别[参考](https://stackoverflow.com/questions/20516773/stdunique-lockstdmutex-or-stdlock-guardstdmutex)

[scoped_lock](https://zh.cppreference.com/w/cpp/thread/scoped_lock) 接收多个mutex对象，解决获取多个对象的问题，等同于多个std::lock_guard 语句，能够有效的避免因为释放顺序而导致的死锁问题。

```cpp
std::mutex mtx;
void print_block (int n, char c) {
    // unique_lock有多组构造函数, 
    // 这里std::defer_lock不设置锁状态
    std::unique_lock<std::mutex> my_lock (mtx, std::defer_lock);
    //尝试加锁, 如果加锁成功则执行
    //(适合定时执行一个job的场景, 
    // 一个线程执行就可以, 可以用更新时间戳辅助)
    if(my_lock.try_lock()){
    }
}
```
scoped_lock的示例
```cpp
std::mutex mtx1;
std::mutex mtx2;
{
  std::scoped_lock<std::mutex> lock(mtx1, mtx2);
  /* 等价于
  {
    std::lock_guard<std::mutex> lk1(mtx1, std::adopt_lock);
    std::lock_guard<std::mutex> lk2(mtx2, std::adopt_lock);
  }
  */
}

```

### 6.6 future | promise

[std::promise](https://en.cppreference.com/w/cpp/thread/promise) 对象可以保存某一类型 T 的值，该值可被 [std::future](https://en.cppreference.com/w/cpp/thread/future) 对象读取（可能在另外一个线程中），因此 promise 也提供了一种线程同步的手段。


```cpp
std::promise<int> prom;
std::future<int> fut = prom.get_future();
```

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



## 参考文档
http://www.umich.edu/~eecs381/handouts/handouts.html
