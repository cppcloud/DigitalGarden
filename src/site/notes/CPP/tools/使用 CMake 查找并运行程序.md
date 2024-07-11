---
{"dg-publish":true,"tags":["cpp"],"permalink":"/CPP/tools/使用 CMake 查找并运行程序/","dgPassFrontmatter":true,"noteIcon":"","created":"2024-07-12T03:30:29.065+08:00","updated":"2024-07-12T03:34:02.384+08:00"}
---



## 查找程序

在 CMake 中，您可以使用 `find_program()` 命令查找任何程序，该命令适用于 Windows 上的可执行文件以及其他平台上的可执行文件，即使它们的扩展名不同。有关 `find_program()` 的更多信息，请参阅[官方文档](https://cmake.org/cmake/help/latest/command/find_program.html)。

通常，您需要在 CMake 文件中编写如下内容：

```cmake
find_program(PROGRAM_VAR python)


if(NOT PROGRAM_VAR)
    message(FATAL_ERROR "Python not found!")
endif()

add_custom_target(MyTarget
    COMMAND ${PROGRAM_VAR} example.py
)
```

或者，如果在不同平台上 Python 可能有不同的名称，可以这样使用：

```cmake
find_program(PROGRAM_VAR NAMES python3 python)

if(NOT PROGRAM_VAR)
    message(FATAL_ERROR "Python not found!")
endif()

add_custom_target(MyTarget
    COMMAND ${PROGRAM_VAR} example.py
)
```

## 解释

1. **find_program ()**：这个命令用于查找可执行文件。参数 `PROGRAM_VAR` 是存储找到的可执行文件路径的变量名。可以提供一个或多个程序名称。

2. **检查结果**：`if(NOT PROGRAM_VAR)` 用于检查是否找到了指定的程序。如果没有找到，CMake 将显示致命错误信息并停止配置过程。

3. **add_custom_target ()**：这个命令用于创建一个自定义目标。`COMMAND ${PROGRAM_VAR} example.py` 表示当构建这个目标时，将运行找到的程序并传递 `example.py` 作为参数。

# 将运行 Python 脚本作为 MyExecutable 的构建依赖

```cpp
Add_dependencies (MyExecutable RunPythonScript)
```
