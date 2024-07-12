---
{"dg-publish":true,"tags":["cpp"],"permalink":"/CPP/cmake/CMake中使用CCache加速编译/","dgPassFrontmatter":true}
---


CCache 是一个广泛用于加速 C 和 C++ 程序编译的工具。它的核心功能是缓存之前的编译结果，并在再次编译时重用这些结果，从而显著减少重复编译所需的时间。当你使用相同的源代码和编译器选项进行编译时，CCache 会检查其缓存，如果发现已缓存的编译结果，就会直接使用这些结果，避免了再次进行完整编译的时间消耗。

CCache 实际上并不是一个编译器，而是一个编译器启动器，意味着它会代理传统的编译器命令。使用 CCache 时，你只需要将原本的编译命令传给 CCache，CCache 再调用实际的编译器完成编译任务。

在 CMake 中，为了使用 CCache，通常会在 CMakeLists. Txt 文件中设置 `CMAKE_CXX_COMPILER_LAUNCHER` 变量。这个变量用于指定编译器启动器。下面是一个典型的配置例子：

```cmake
find_program(CCACHE ccache)
if(CCACHE)
    message(STATUS "Using CCache as a compiler launcher")
    set(CMAKE_CXX_COMPILER_LAUNCHER "${CCACHE}")
endif()
```

这段代码首先尝试找到 CCache 程序。如果找到了，就会打印一条状态消息并设置 `CMAKE_CXX_COMPILER_LAUNCHER` 为 `ccache`，这样所有的编译指令都会通过 CCache 来进行，利用缓存机制来加快编译速度。
