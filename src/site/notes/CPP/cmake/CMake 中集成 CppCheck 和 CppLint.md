---
{"dg-publish":true,"tags":["cpp"],"permalink":"/CPP/cmake/CMake 中集成 CppCheck 和 CppLint/","dgPassFrontmatter":true}
---



#### 引言
CppCheck 是一款强大的静态代码分析工具，它可以帮助开发者发现 C/C++ 代码中的问题。你可以通过[这个链接](https://cppcheck.sourceforge.io/)下载并安装 CppCheck。我强烈推荐你在项目中使用这个工具，它对于提高代码质量非常有帮助。

#### CppCheck 的集成使用方法

如果你在使用 CMake 来构建项目，那么你会很高兴知道 CppCheck 可以直接与 CMake 集成。这种集成支持使用 Makefiles 或 Ninja 生成器。通过集成，CppCheck 可以在构建过程中运行，对你的 C/C++ 文件进行分析。要使用它，你需要设置 `CMAKE_CXX_CPPCHECK` 变量。以下是一个如何在 CMake 中寻找并设置 CppCheck 的示例：

```cmake
find_program(CPPCHECK cppcheck)

if(CPPCHECK)
    set(CMAKE_CXX_CPPCHECK "cppcheck")
endif()
```

这段代码首先尝试找到 cppcheck 程序。如果找到了，`CMAKE_CXX_CPPCHECK` 变量就会被设置为 `"cppcheck"`，这样 CMake 就会在每次构建时自动运行 CppCheck。

#### CppLint 的集成方法

CppLint 是另一款静态代码分析工具，它的使用方式与 CppCheck 类似，也可以通过设置 CMake 变量来集成。以下是集成 CppLint 的步骤：

```cmake
find_program(CPPLINT cpplint)

if(CPPLINT)
    set(CMAKE_CXX_CPPLINT "cpplint")
endif()
```

这段代码尝试找到 cpplint 程序，并在找到后通过设置 `CMAKE_CXX_CPPLINT` 变量来启用其集成。

