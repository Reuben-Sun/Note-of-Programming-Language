# Standard C++

基于C++11（都什么年代了，还在整C++11？八股害人啊）

## 一：C++11的“新”特性

### nullptr

一个新关键字，用于表示指针指向no value，可以被自动转化为各种指针类型，但是不会被转化为整数

*比NULL好，因为NULL的本质就是整数0*

### auto

通过auto声明变量、对象，可以自动推到其类型，在处理表达式时有奇效

```c++
auto l = [](int x) -> {...};
```

### 一致性初始化

一致性初始化（Uniform Initiallization），简单来说就是可以用大括号来做初始化

```c++
int v[] {1, 2, 3};
vector<int> v2 {1, 2, 3};
complex<double> c{4.0, 3.0};
```

但是这个操作不支持**窄化（narrowing）**，即

```c++
int x = 5.3;	//x == 5
int y {5.3};	//Error
```

### 新的for循环

```c++
for(auto& item: lists){...}

for(int i : {1, 2, 3, 4}){...}
```

### 转移语意

转移语意（move semantic），用于避免非必要的拷贝和临时对象

比如一个函数定义为

```c++
void fun(const T& v){...}
```

当我们调用时

```c++
T t;
fun(t);		//1
fun(t+1);	//2
```

我们发现，第一种调用时，直接传了引用，没有拷贝和临时变量，而第二种，实际上是

```c++
T temp = t+1;		//T temp(t+1)或者 T temp.T(t+1)
fun(temp);
```

总之是调用了拷贝构造函数和生成了临时变量，这不是我们想要的，于是引入了左右值和转移语意的概念，然后就诞生了一种新的调用方式：

```c++
fun(std::move(t+1));
```

`std::move`的作用是将其参数`t+1`变成一个**右值（rvalue reference）**，是一个`T&&`的类型

一旦一个类型被“标记”为右值，就意味着这是一个临时对象，你可以随意使用它的内容和资源

然后我们可以优化一下这个函数的定义

```c++
class T{
public:
  T (const T& lvalue);	//通过左值拷贝构造（根传统C++一样）
  T (T&& rvalue);		//通过右值move构造
}
void fun(T&& v){...}
```

右值被move以后，就变成有效但不确定的状态

### 字符串字面量

#### Raw String Literal

以R开头的字符串，可以换行，不需要转义，用于正则表示式有奇效

```c++
R"(\\n)";	//等于 "\\\\n"
```

#### Encoded String Literal

用于国际化

### noexcept

让函数无法抛出异常，遇到未定义事件会直接`abort`

noexcept后面可以跟一个bool条件，为true时就不抛异常

```c++
void fun() noexcept;
void fun2(T& x, T& y) noexcept(noexcept(x.swap(y))){
    x.swap(y);
    //这个函数意味着，只要xy交换不抛异常，那么下面这些函数体做任何事都不会抛异常
}
```

### constexpr

用于让表达式核定与编译期，能助力TMP编程

```c++
constexpr int square(int x){
    return x*x;
}
float a[square(9)];
```

```c++
//以前的写法
template<unsigned n>
struct Factorial
{
    enum { value = n * Factorial<n-1>::value };
};
//C++11写法
template<unsigned n>
struct Factorial
{
    constexpr static auto value{ n * Factorial<n - 1>::value };
};
```



### 新的模板

#### 不定个数的参数

```c++
void f(int i){
    cout << i << endl;
}
template <typename T , typename... Types>
void f(const T& first, const Types&... args){
    cout << first << endl;
    f(args...);
}
int main() {
    f(1, 2, 3);
    return 0;
}
```

#### 模板别名

```c++
template <typename T>
using Vec = std::vector<T, MyAlloc<T>>;
Vec<int> coll;	//等价于std::vector<int, MyAlloc<int>> coll;
```

### Lambda

允许函数的定义式被用作一个参数、local对象

#### 定义与调用

```c++
[]{
    cout << "hello" << endl;
};	//这是一个lambda表达式

[]{
    cout << "hello" << endl;
}();	//定义并直接调用表达式

auto l = []{
    cout << "hello" << endl;
};
l();	//调用表达式
```

#### 含参

```c++
auto l2 = [](const std::string& s){
    cout << s << endl;
};
l("hello");
```

#### 返回值

```c++
[]() -> int {
    return 42;	
}
```

#### 外部作用域

分值传递和引用传递两种，值传递不能进行修改

```c++
int x = 0;
int y = 42;
auto l = [x, &y] {
    cout << x << "+" << y << endl;
    ++y;
}
l();	//调用，注意xy不是参数，不需要写在括号里
```

#### mutable

这个关键词是const的反义词，意思是可变的，于是让值传递也可变

```c++
int x = 0;
auto l = [x]() mutable {
    cout << x << endl;
    x++;
}
l();
```

### decltype

自动推导表达式的类型，大号typeof

```c++
const int &i = 1;
int a = 2;
decltype(i) b = 2; // b是const int&
```

#### 推断返回类型

可以将返回类型的声明放在参数列之后

```c++
template <typename T1, typename T2>
auto add(T1 x, T2 y) -> decltype(x+y){
    return x + y;
}
```

### 带领域的枚举

## 二：一般概念

### 可被调用的对象

Callable Object，一种对象，可以通过某些方式调用其某些函数，它可以是

- 函数
- 指向成员函数的指针
- 函数对象
- lambda表达式

## 三：通用工具

### pair

本质是一个struct

```c++
int add(pair<int, int> p){
    return p.first + p.second;
}

int main() {
    pair<int, int> p(42, 10);
    cout << add(p);
    return 0;
}
```

### tuple

大号pair，可以有多个值

```c++
int f(tuple<int, int, int> t){
    return get<0>(t) + get<1>(t) + get<2>(t);
}
int main() {
    tuple<int, int, int> t(3, 5, 7);
    cout << f(t);
    return 0;
}
```

### 智能指针

智能指针智能在，它能知道自己是不是指向某物的最后一个指针

- shared_ptr：共享式拥有
  - 多个指针可以指向一个资源，通过引用计数法GC
  - 为了解决引用计数法的缺点（比如循环引用），提供weak_ptr等辅助类
- unique_ptr：独占式拥有
  - 同一时间只能有一个unique_ptr指向某个资源，可以进行拥有权的移交

### 极值

Numeric Limit

用于得到当前平台下，一些数值类型的长度（大小）

### Trait

是一种技术方案，用来为同一类数据提供统一的操作函数，核心就是使用另外的模版类`type_traits`存储不同数据类型的`type`，这样就可以兼容各种数据类型

### 外覆器

#### Reference Wrapper

允许函数模板可以操作引用，不需要写特化版本

具体有两个函数

- ref：隐式转化为`T&`
- cref：隐式转化为`const T &`

```c++
template <typename T>
void fun(T v);

int x;
fun(std::ref(x));	//此时T为int&

int x;
fun(std::cref(x));	//此时T为const int&
```

#### Function Type Wrapper

允许将可调用对象当作最高级对象（first-class object）

```c++
vector<function<void(int, int)>> tasks;	//一个存储多个可调用对象的vector
tasks.push_back(func);	//void func(int x, int y);
tasks.push_back([] (int x, int y) {...});	//添加一个lambda表达式

for(function<void(int, int)> f : tasks){
    f(36, 36);	//遍历所有的可调用对象，并调用
}
```

### 辅助函数

- min
- max
- swap
- operator
  - `==`
  - `!=`
  - `>`
  - `<`
  - `>=`
  - `<=`

### 编译期分数运算

```c++
ratio<5, 5> one;
cout << one.num << "/" << one.den << endl;	// 1/1

ratio<5, 3> two;
cout << two.num << "/" << two.den << endl;	// 5/3
```

## 四：STL

STL是C++标准库的核心，是一个泛型（generic）程序库，由三部分组成

- 容器（Container）
- 迭代器（Iterator）
- 算法（Algorithm）

### 容器

#### 有序容器

顺序与插入顺序有关，与元素值无关，常常通过array、linked list实现

- array
- vector
- deque
- list
- forward_list

#### 关联式容器

在内部进行排序的集合，位置取决于value，常常通过二叉树实现

- set
- multiset（mult的意思是元素可以重复）
- map
- multimap（mult的意思是key可以重复）

#### 无序容器

元素位置无关紧要，重要的是元素是否在容器内，常常通过哈希表实现

- unordered_set
- unordered_multiset
- unordered_map
- unordered_multmap

#### 其他容器

- string
- 寻常的数组（一种type，而非class）

### 迭代器

迭代器是一个可以遍历STL容器全部、部分元素的对象

#### 操作

- `*`：取元素
- `++`：迭代器前进至下一个元素
  - 注意，`++i`比`i++`效率高一点点，因为后者要创建临时对象
- `==`、`!=`：判断两个迭代器是否指向同一个位置
- `=`：赋值

### 种类

|                 | R/W       | 读写次数         | 跳转   | 举例                                 |
| --------------- | --------- | ---------------- | ------ | ------------------------------------ |
| 输入/输出迭代器 | 只读/只写 | 能且仅能读写一次 | i++    | istream_iterators、ostream_iterators |
| 前向/双向迭代器 | 读写      | 能读写多次       | i++    | STL的set、map                        |
| 随机访问迭代器  | 读写      | 能读写多次       | i += n | vector、deque、string、array         |

从上到下，迭代器能力越来越强（他们间有继承关系，前向 is a 输入）

### 算法

大多为非成员函数，思想是泛型编程（而不是OOP）

### 函数对象

一个行为像函数的对象，思想是泛型编程

```c++
class X{
public:
    int operator() (int a, int b) const;
};
...
X fo;
fo(arg1, arg2);	//等同于fo.operator()(arg1, arg2);
```

- 函数对象是一个带状态的函数
- 函数对象有自己的类型
- 函数对象速度通常比普通函数快（编译期间有更好的优化）

