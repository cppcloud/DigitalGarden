---
{"dg-publish":true,"tags":["cpp"],"permalink":"/CPP/使用 scope_guard 进行资源管理/","dgPassFrontmatter":true}
---


在现代 C++ 编程中，资源管理是一个非常重要的主题。正确管理资源可以避免资源泄漏，确保程序的稳定性和性能。
### `scope_guard` 类定义

```cpp
#ifndef SCOPE_GUARD_H
#define SCOPE_GUARD_H

#include <functional>

namespace helpers {
    class scope_guard final {
    public:
        using Callback = std::function<void()>;

        scope_guard(const Callback &fn) : fn_{fn} {}

        ~scope_guard() { fn_(); }

    private:
        scope_guard() = delete;
        scope_guard(scope_guard &&) = delete;
        scope_guard &operator=(scope_guard &&) = delete;
        scope_guard(const scope_guard &) = delete;
        scope_guard &operator=(const scope_guard &) = delete;

        const Callback &fn_;
    };
} // namespace helpers

#endif
```

`scope_guard` 是一个简单的作用域保护实现：在 `scope_guard` 对象离开作用域时执行一个回调函数。这个类可以帮助我们在资源管理中确保资源在适当的时候被释放。

### 使用案例

通过一个文件处理函数来说明 `scope_guard` 的实际使用。在这个例子中，我们将尝试打开一个文件，处理文件内容，并在文件处理完毕后确保文件被正确关闭。

```cpp
#include "scope_guard.h"
#include <iostream>
#include <fstream>
#include <string>

int processFile(const std::string &filePath) {
    std::ifstream file(filePath);

    if (!file.is_open()) {
        std::cerr << "Failed to open file: " << filePath << std::endl;
        return -1; // 返回错误码
    }

    helpers::scope_guard closeFileGuard([&file]() {
        std::cout << "Closing file." << std::endl;
        file.close();
    });

    std::cout << "File opened successfully: " << filePath << std::endl;

    std::string line;
    while (std::getline(file, line)) {
        // 逐行处理文件内容
        std::cout << "Processing line: " << line << std::endl;
        if (line == "error") {
            std::cerr << "Error encountered, exiting early." << std::endl;
            return -2; // 提前返回错误码
        }
    }

    std::cout << "File processed successfully." << std::endl;
    return 0; // 成功处理文件，返回0
}

int main() {
    const std::string filePath = "example.txt";
    int result = processFile(filePath);

    if (result == 0) {
        std::cout << "File processed without errors." << std::endl;
    } else {
        std::cerr << "File processing encountered errors, code: " << result << std::endl;
    }

    return result;
}
```

### 解释

在这个例子中：

1. **文件打开失败处理**：
    - `processFile` 函数尝试打开一个文件。如果文件打开失败，输出错误信息并返回 `-1` 作为错误码。

2. **创建 `scope_guard` 对象**：
    - 创建 `scope_guard` 对象 `closeFileGuard`，它持有一个 lambda 函数，该函数在 `scope_guard` 对象销毁时执行，确保文件在退出作用域时被关闭。

3. **逐行处理文件内容**：
    - 读取文件并逐行处理。如果遇到某种错误（例如，行内容为 "error"），输出错误信息并返回 `-2` 作为错误码。由于 `scope_guard` 的存在，即使在提前返回的情况下也能确保文件被正确关闭。

4. **成功处理文件**：
    - 如果文件正常处理完成，输出处理成功的信息并返回 `0` 作为成功码。
