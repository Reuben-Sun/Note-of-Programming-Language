# 高性能C++

在摩尔定律逐渐失效的今日，CPU主频和单核性能提升越来越不明显，为了得到跟高的性能，我们走向了并行计算的道路

## 概念

### 并发和并行

- 并发（Concurrent）：happening during the same time span，处理器在两件事间快速切换，宏观上看，就像是同时发生的（下图AB）

- 并行（Parallel）：happening at the same time，多个处理器一起做事（下图CD）

![并发和并行](Image/并发和并行.png)

## TBB

TBB（Threading Building Blocks）是一个非常流行的C++并行编程（parallel programming）方案

- 使用Task而非Thread
  - Thread与硬件相关，直接编写难以跨平台，而且线程间通信过于频繁，过于繁琐
  - TBB使用Task编程，运行时会将程序映射到硬件上（该方法对嵌套并行的支持很不错）

- TBB实现了可组合性（Composability）
  - 简单来说就是支持for循环

- 方便移植（portable）

### Mac上安装使用TBB

1. 安装tbb

```sh
$brew install tbb
```

2. 链接tbb

```cmake
#寻找tbb包
find_package(TBB REQUIRED)
...
#链接
target_link_libraries(TimeStudy TBB::tbb)
```

3. 引用头文件

```c++
#include<tbb/tbb.h>
```

### Hello TBB

一个简单的TBB示例，并行输出两行字符串

```c++
#include <iostream>
#include <tbb/tbb.h>

int main() {
    tbb::parallel_invoke(
            [](){std::cout << "TBB!" << std::endl;},
            [](){std::cout << "Hello" << std::endl;}
            );
    return 0;
}
```

我们这里使用lambda表达式创建了匿名（anonymous）函数，可以大幅简化TBB编写

### 时刻查询

[Code](https://github.com/Reuben-Sun/TBB--Programing-Sample/tree/main/TimeStudy)

```c++
//t0时刻
tbb::tick_count t0 = tbb::tick_count::now();	
...
//当前时刻-t0时刻=经过了多长时间段（并转化为秒）
std::cout << "Time: " << (tbb::tick_count::now() - t0).seconds() << " seconds" << std::endl;
```



## 资料

[Pro TBB](https://github.com/Apress/pro-TBB)

