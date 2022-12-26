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

并行开发中动态内存管理十分复杂、危险（内存泄漏），这里推荐使用C++11的智能指针（自带GC）

### Flow Graph

将函数执行制作成Graph，实现消息驱动的并行（有些类似FrameGraph）

![FlowGraph](Image/FlowGraph.jpeg)

[Code](https://github.com/Reuben-Sun/TBB--Programing-Sample/tree/main/FlowGraph)

```c++
void fig1_10(const std::vector<ImagePtr>& image_vector){
    const double tint_array[] = {0.75, 0, 0};

    tbb::flow::graph g;
    int i = 0;
    //注意，source_node已经失效
    tbb::flow::input_node<ImagePtr> src(g,
        [&i, &image_vector](tbb::flow_control &fc) -> ImagePtr {
            if(i < image_vector.size()){
                return image_vector[i++];
            }
            else{
                fc.stop();
                return {};
            }
    });
    tbb::flow::function_node<ImagePtr, ImagePtr> gamma(g,
        tbb::flow::unlimited, [] (ImagePtr img) -> ImagePtr{
                return applyGamma(img, 1.4);
        }
    );
    tbb::flow::function_node<ImagePtr, ImagePtr> tint(g,
        tbb::flow::unlimited, [tint_array] (ImagePtr img) -> ImagePtr{
                return applyTint(img, tint_array);
        }
    );
    tbb::flow::function_node<ImagePtr> write(g,
         tbb::flow::unlimited, [] (ImagePtr img){
                writeImage(img);
        }
    );

    tbb::flow::make_edge(src, gamma);
    tbb::flow::make_edge(gamma, tint);
    tbb::flow::make_edge(tint, write);
    src.activate();
    g.wait_for_all();
}
```

最后该程序会以**流水线**的形式执行，当第一张图片完成gamma矫正，去执行tint着色时，第二张图片开始执行gamma矫正（每张图片间互不影响）

相比于TimeStudy的串型执行，流水线执行效率会更快

## 资料

[Pro TBB](https://github.com/Apress/pro-TBB)

[API Document](https://spec.oneapi.io/versions/latest/elements/oneTBB/source/nested-index.html)
