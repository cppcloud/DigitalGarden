---
{"dg-publish":true,"tags":["cpp"],"permalink":"/CPP/cmake/CMake构建后的资源复制/","dgPassFrontmatter":true}
---



在开发项目中，经常需要在构建后将某些资源文件复制到特定目录。虽然有专门的工具可以完成这个任务，但实际上，通过 CMake 中的简单指令也能轻松实现。

你只需添加如下代码到你的 CMakeLists. Txt 文件中：

```cmake
add_custom_command(TARGET Exe POST_BUILD
    COMMAND "${CMAKE_COMMAND}" -E copy_directory "${CMAKE_CURRENT_LIST_DIR}/data" "${CMAKE_CURRENT_BINARY_DIR}/data")
```

这段代码的作用是，在构建 Exe 目标之后，使用 CMake 命令将当前列表目录中的 `data` 文件夹复制到当前二进制目录中的 `data` 文件夹。

### 详细解释

- **add_custom_command**: 这个命令用于向构建过程添加自定义命令。在这里，我们使用它来定义一个在构建后执行的命令。
- **TARGET Exe POST_BUILD**: 这部分指定了自定义命令将作用于 `Exe` 目标，并且在构建之后执行。
- **COMMAND**: 这里定义了实际执行的命令。`${CMAKE_COMMAND}` 是 CMake 自身的可执行文件，通过它可以执行各种 CMake 命令。
- **-E copy_directory**: 这是 CMake 的一个内置命令，用于复制目录。它将复制 `"${CMAKE_CURRENT_LIST_DIR}/data"` 目录到 `"${CMAKE_CURRENT_BINARY_DIR}/data"` 目录。

通过这种方式，你可以确保在每次构建之后，指定的目录都会被正确复制，从而避免了手动复制的麻烦。