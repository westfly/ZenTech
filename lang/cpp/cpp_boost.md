boost 学习指北
---

## 编译boost库

下载最新的boost，解压到一定的目录。
因为boost依赖于b2编译，需要执行 bootstrap.sh 生成b2工具。

https://stackoverflow.com/questions/5346454/building-boost-with-different-gcc-version 可以解决非系统GCC编译问题。

## 时间处理


## 宏预处理
如下内容主要参考了
[Boost 库 C/C++ Preprocessor 子集](http://sns.hwcrazy.com/boost_1_41_0/libs/preprocessor/doc/index.html)

### BOOST_PP_CAT
BOOST_PP_CAT 包含于boost/preprocessor/cat.hpp 中。
BOOST_PP_CAT(a,b) 用来连接两个标识符
```cpp
#include <boost/preprocessor/cat.hpp>
BOOST_PP_CAT(x, BOOST_PP_CAT(y, z)) // 展开为 xyz
```
### BOOST_PP_IF
BOOST_PP_IF 包含于 boost/preprocessor/control/if.hpp

BOOST_PP_IF(cond, t, f) 的用法为
cond
确定选择结果的条件。有效值为 0 到 BOOST_PP_LIMIT_MAG.
t
cond 为非零时的展开结果。
f
cond 为 0 时的展开结果。

```cpp
#include <boost/preprocessor/control/if.hpp>

BOOST_PP_IF(10, a, b) // 展开为 a
BOOST_PP_IF(0, a, b) // 展开为 b
```

### BOOST_PP_TUPLE_ELEM
BOOST_PP_TUPLE_ELEM 包含于boost/preprocessor/tuple/elem.hpp 
BOOST_PP_TUPLE_ELEM(size, i, tuple)  
从 tuple 中取出i的下标元素。
```cpp
#include <boost/preprocessor/tuple/elem.hpp>
#define TUPLE (a, b, c, d)
BOOST_PP_TUPLE_ELEM(4, 0, TUPLE) // 展开为 a
BOOST_PP_TUPLE_ELEM(4, 3, TUPLE) // 展开为 d
```

### BOOST_PP_SEQ_FOR_EACH
BOOST_PP_SEQ_FOR_EACH 包含于 boost/preprocessor/seq/for_each.hpp 中。
BOOST_PP_SEQ_FOR_EACH(func, data, seq) ，其中：
func 为自己定义的一个宏
data 为一个常量
seq 为一个序列
这个宏会将序列中的参数依次按照指定的宏 func 展开。
```cpp

#include <boost/preprocessor/seq/for_each.hpp>
 
#define SEQ (w)(x)(y)(z)
 
#define MACRO(r, data, elem) elem::GetInstance()
// expands to w::GetInstance() x::GetInstance() y::GetInstance() z::GetInstance()
BOOST_PP_SEQ_FOR_EACH(MACRO, _, SEQ) 
```
可以用g++ -E -P src/boost_macro.cpp 进行验证

### BOOST_PP_SEQ_FOR_EACH_I
BOOST_PP_SEQ_FOR_EACH_I 跟 BOOST_PP_SEQ_FOR_EACH 基本是一样的。
就是对MACRO的参数要求变为四个。
即 macro(r, data, i, elem) ，将SEQ展开的时候，会将i（index）传递给macro，index从0开始。
```cpp
#include <boost/preprocessor/cat.hpp>
#include <boost/preprocessor/seq/for_each_i.hpp>
#define SEQ (a)(b)(c)(d)
#define MACRO(r, data, i, elem) BOOST_PP_CAT(elem, BOOST_PP_CAT(data, i))
BOOST_PP_SEQ_FOR_EACH_I(MACRO, _, SEQ) // 展开为 a_0 b_1 c_2 d_3

```

gen 
## boost-log



