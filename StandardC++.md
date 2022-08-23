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

用于让表达式核定与编译期，感觉和内联和宏编译思想一致？

```c++
constexpr int square(int x){
    return x*x;
}
float a[square(9)];
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
  - 为了解决引用计数法的缺点，提供weak_ptr等辅助类
- unique_ptr：独占式拥有
  - 同一时间只能有一个unique_ptr指向某个资源，可以进行拥有权的移交







