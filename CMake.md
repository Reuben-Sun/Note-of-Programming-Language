# CMake

*来自小鹏老师的CMake课堂*

### C++项目结构

一个大项目（Project）内嵌多个子项目（SubProject）

一个子项目内有src、include、CMakeLists.txt，其中有一个子项目中有main.cpp

<img src="Image/C++项目结构.png" alt="C++项目结构" style="zoom:50%;" />

最外面的CMakeLists.txt，负责连接所有子项目：

```cmake
cmake_minimum_required(VERSION 3.20)
project(Project)
set(CMAKE_CXX_STANDARD 14)
add_subdirectory(subProject1)
add_subdirectory(subProject2)
```

subProject1（main.cpp所在的子项目）下面的CMakeLists.txt：

```cmake
file(GLOB_RECURSE srcs CONFIGURE_DEPENDS src/*.cpp include/*.h)
add_executable(subProject1 ${srcs})
target_include_directories(subProject1 PUBLIC include)
target_link_libraries(subProject1 PUBLIC subProject2)
```

subProject2下面的CMakeLists.txt：

```cmake
file(GLOB_RECURSE srcs CONFIGURE_DEPENDS src/*.cpp include/*.h)
add_library(subProject2 STATIC ${srcs})
target_include_directories(subProject2 PUBLIC include)
```

### 第三方库

查找名为OpenCV的包，如果没找到就报错

```cmake
find_package(OpenCV REQUIRED)
```

该函数的本质就是去寻找一个`包名-config.cmake`文件