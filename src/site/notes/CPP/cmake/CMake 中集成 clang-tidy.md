---
{"dg-publish":true,"tags":["tools"],"permalink":"/CPP/cmake/CMake 中集成 clang-tidy/","dgPassFrontmatter":true}
---

Clang-tidy 是来自 LLVM 项目的 C++ 静态分析工具，在现代化代码库或检查代码是否符合既定指南和最佳实践时特别有用。

通常，您需要使用与编译时使用的相同选项和编译器标志来运行 `clang-tidy` 。手动执行此操作可能会很麻烦。

解决此问题的一种方法是使用编译数据库。然而，我发现这本身就相当复杂。

如果您使用 CMake，则有一个更简单的替代方案：您可以直接使用 CMake 中的 `clang-tidy`

您需要执行两个步骤：

+ 查找并设置 `clang-tidy` 命令

+ 告诉 CMake 使用 `clang-tidy`

第一步，您只需使用 CMake 的 `find_program()` 并将相应的 CMake 变量设置为您要使用的命令和选项：

```cmake
find_program(CLANG_TIDY_EXE NAMES "clang-tidy")
set(CLANG_TIDY_COMMAND "${CLANG_TIDY_EXE}" "-checks=-*,modernize-*")
```

在上面，我禁用所有默认检查 ( `-*` )，仅启用提倡使用现代 C++ 语言结构的检查 ( `modernize-*` )。

对于第二步，您通过为构建目标设置 `CXX_CLANG_TIDY` 属性来告诉 CMake 使用 `clang-tidy` ：

```cmake
set_target_properties(target PROPERTIES CXX_CLANG_TIDY "${CLANG_TIDY_COMMAND}")
```

就这样！像往常一样运行您的构建， `clang-tidy` 将报告现代化建议作为警告。

## 安装常用的编译工具链

```bash 
sudo apt install clang-tidy cmake ninja-build gcc g++
```
### 使用模板

```cmake
cmake_minimum_required(VERSION 3.20)
project(Example)

file(GLOB_RECURSE ExampleSource CONFIGURE_DEPENDS
	"${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/*.h"
)

# search for clang-tidy
find_program(CLANG_TIDY_EXE NAMES "clang-tidy" REQUIRED)

# setup clang-tidy command from executable + options
set(CLANG_TIDY_COMMAND "${CLANG_TIDY_EXE}" "-checks=-*,modernize-*")

# add target using generated source file
add_executable(Example ${ExampleSource})

# set CXX_CLANG_TIDY property after defining the target
set_target_properties(Example PROPERTIES CXX_CLANG_TIDY "${CLANG_TIDY_COMMAND}")
```

## 测试文件

```cpp
#include <stdio.h>

int main(int, char**){

    printf("Hello, from Example!\n");

}
```

### 测试结果

```cpp
[0/2] Re-checking globbed directories...
[1/2] Building CXX object CMakeFiles/Example.dir/src/main.cpp.o
/home/duang/WorkSpace/src/main.cpp:1:10: warning: inclusion of deprecated C++ header 'stdio.h'; consider using 'cstdio' instead [modernize-deprecated-headers]
    1 | #include <stdio.h>
      |          ^~~~~~~~~
      |          <cstdio>
/home/duang/WorkSpace/src/main.cpp:3:5: warning: use a trailing return type for this function [modernize-use-trailing-return-type]
    3 | int main(int, char**){
      | ~~~ ^                
      | auto                  -> int
[2/2] Linking CXX executable Example
```

### 源码链接

[GitHub - cppcloud/CmakeUseClangTidySample: This repository provides an integrated setup for using clang-tidy with CMake to ensure high-quality, clean code in C++ projects. It includes configuration files and scripts to facilitate the automatic application of clang-tidy.](https://github.com/cppcloud/CmakeUseClangTidySample)