---
{"dg-publish":true,"tags":["cpp"],"permalink":"/CPP/C++迭代器失效/","dgPassFrontmatter":true}
---


### C++17 关联容器迭代器和引用失效情况（CPP 17 - n 4659）

#### 插入操作
##### 顺序容器

- **vector**:
  - 如果新大小超过旧容量，`insert`、`emplace_back`、`emplace`、`push_back` 函数会导致重新分配。重新分配会使所有引用、指针和迭代器失效。如果没有发生重新分配，则插入点之前的所有迭代器和引用保持有效。【26.3.11.5/1】
  - 对于 `reserve` 函数，如果新大小超过旧容量，重新分配会使所有引用、指针和迭代器失效。调用 `reserve` 后，直到插入操作使大小超过容量值之前，不会发生重新分配。【26.3.11.3/6】

- **deque**:
  - 在 `deque` 中间插入会使所有迭代器和引用失效。在 `deque` 末尾插入会使所有迭代器失效，但不会影响对 `deque` 元素的引用的有效性。【26.3.8.4/1】

- **list**:
  - 插入、`emplace_front`、`emplace_back`、`emplace`、`push_front`、`push_back` 操作不会影响迭代器和引用的有效性。如果抛出异常，没有任何效果。【26.3.10.4/1】

- **forward_list**:
  - 所有 `insert_after` 的重载不会影响迭代器和引用的有效性。【26.3.9.5/1】

- **array**:
  - 作为规则，数组的迭代器在数组的整个生命周期内都不会失效。然而，在交换期间，迭代器将继续指向相同的数组元素，因此会改变其值。

##### 关联容器

- **所有关联容器**：
  - `insert` 和 `emplace` 成员不会影响迭代器和引用的有效性。【26.2.6/9】

##### 无序关联容器

- **所有无序关联容器**：
  - 重新散列会使迭代器失效，改变元素间的顺序，并改变元素出现的桶，但不会使指针或引用失效。【26.2.7/9】
  - `insert` 和 `emplace` 成员不会影响对容器元素的引用的有效性，但可能会使所有迭代器失效。【26.2.7/14】
  - 如果 \( (N+n) \leq z \times B \)，其中 \( N \) 是插入操作前的元素数，\( n \) 是插入的元素数，\( B \) 是容器的桶数，\( z \) 是容器的最大负载因子，则 `insert` 和 `emplace` 成员不会影响迭代器的有效性。【26.2.7/15】
  - 在合并操作（例如，`a.merge(a2)`）的情况下，指向转移元素的迭代器和所有指向 `a` 的迭代器将失效，但指向仍在 `a2` 中的元素的迭代器将保持有效。（表 91 — 无序关联容器要求）

##### 容器适配器

- **stack**：继承自基础容器
- **queue**：继承自基础容器
- **priority_queue**：继承自基础容器

#### 删除操作
##### 顺序容器

- **vector**:
  - `erase` 和 `pop_back` 函数使擦除点或擦除点之后的迭代器和引用失效。【26.3.11.5/3】

- **deque**:
  - 删除 `deque` 的最后一个元素的 `erase` 操作只使末尾迭代器和所有指向被删除元素的迭代器和引用失效。删除 `deque` 的第一个元素但不是最后一个元素的 `erase` 操作只使指向被删除元素的迭代器和引用失效。删除既不是第一个也不是最后一个元素的 `erase` 操作使末尾迭代器和所有指向 `deque` 元素的迭代器和引用失效。【26.3.8.4/4】

- **list**:
  - 仅使指向被删除元素的迭代器和引用失效。这适用于 `erase`、`pop_front`、`pop_back`、`clear` 函数。【26.3.10.4/3】
  - `remove` 和 `remove_if` 成员函数：删除由满足条件 `*i == value` 或 `pred(*i) != false` 的 `list` 迭代器 `i` 指向的所有元素。仅使指向被删除元素的迭代器和引用失效。【26.3.10.5/15】
  - `unique` 成员函数：删除范围 `[first + 1, last)` 中由 `i` 指向的每组连续相等元素中的所有元素，只保留第一个元素（无参版本）或 `pred(*i, *(i - 1))` 为真时（带谓词版本）。仅使指向被删除元素的迭代器和引用失效。【26.3.10.5/19】

- **forward_list**:
  - `erase_after` 仅使指向被删除元素的迭代器和引用失效。【26.3.9.5/1】
  - `remove` 和 `remove_if` 成员函数：删除由满足条件 `*i == value`（对于 `remove()`）或 `pred(*i)` 为真（对于 `remove_if()`）的 `list` 迭代器 `i` 指向的所有元素。仅使指向被删除元素的迭代器和引用失效。【26.3.9.6/12】
  - `unique` 成员函数：删除范围 `[first + 1, last)` 中由 `i` 指向的每组连续相等元素中的所有元素，只保留第一个元素（无参版本）或 `pred(*i, *(i - 1))` 为真时（带谓词版本）。仅使指向被删除元素的迭代器和引用失效。【26.3.9.6/16】

- **所有顺序容器**：
  - `clear` 使指向容器元素的所有引用、指针和迭代器失效，并可能使末尾迭代器失效（表 87 — 顺序容器要求）。但对于 `forward_list`，`clear` 不会使末尾迭代器失效。【26.3.9.5/32】
  - `assign` 使指向容器元素的所有引用、指针和迭代器失效。对于 `vector` 和 `deque`，还会使末尾迭代器失效。（表 87 — 顺序容器要求）

##### 关联容器

- **所有关联容器**：
  - `erase` 成员仅使指向被删除元素的迭代器和引用失效。【26.2.6/9】
  - `extract` 成员仅使指向被移除元素的迭代器失效；指向被移除元素的指针和引用保持有效。【26.2.6/10】

##### 容器适配器

- **stack**：继承自基础容器
- **queue**：继承自基础容器
- **priority_queue**：继承自基础容器

#### 与迭代器失效相关的一般容器要求

- 除非另有明确说明（显式或通过定义某个函数在其他函数的基础上），调用容器成员函数或将容器作为参数传递给库函数不会使迭代器失效，也不会改变容器内对象的值。【26.2.1/12】
- 没有任何 `swap()` 函数会使指向正在交换的容器元素的任何引用、指针或迭代器失效。【26.2.1/(11.6)】

##### 示例

- **transform 算法**：
  - `op` 和 `binary_op` 函数不应使迭代器或子范围失效，也不应修改范围中的元素。【28.6.4/1】

- **accumulate 算法**：
  - 在范围 `[first, last)` 中，`binary_op` 不应修改元素或使迭代器或子范围失效。【29.8.2/1】

- **reduce 算法**：
  - `binary_op` 不应使迭代器或子范围失效，也不应修改范围 `[first, last)` 中的元素。【29.8.3/5】



C++17 标准明确规定了在不同容器和操作下，迭代器和引用的失效情况。对插入、删除和提取操作的一般规则：

- **关联容器**：
  - 插入和置入操作不会使迭代器和引用失效。
  - 删除和提取操作仅会使被删除或移除元素的迭代器和引用失效。

- **无序关联容器**：
  - 插入和置入操作不会使引用失效，但可能会使所有迭代器失效。如果 \( (N+n) \leq z \times B \)，则插入和置入操作不会使迭代器失效。
  - 删除和提取操作仅会使被删除或移除元素的迭代器失效，引用仍然有效。
  - 重新散列会使迭代器失效，但不会使指针或引用失效。

### 代码示例

#### 插入和置入操作示例

```cpp
#include <iostream>
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>

void set_example() {
    std::set<int> myset = {1, 2, 3, 4, 5};
    auto it = myset.find(3);
    auto ref = *it;

    // 插入操作
    myset.insert(6);

    std::cout << "set 插入操作后:\n";
    std::cout << "it: " << *it << ", ref: " << ref << "\n"; // 迭代器和引用仍然有效
}

void map_example() {
    std::map<int, int> mymap = {{1, 10}, {2, 20}, {3, 30}};
    auto it = mymap.find(2);
    auto ref = it->second;

    // 插入操作
    mymap.insert({4, 40});

    std::cout << "map 插入操作后:\n";
    std::cout << "it: (" << it->first << ", " << it->second << "), ref: " << ref << "\n"; // 迭代器和引用仍然有效
}

void unordered_set_example() {
    std::unordered_set<int> myuset = {1, 2, 3, 4, 5};
    auto it = myuset.find(3);
    auto ref = *it;

    // 插入操作
    myuset.insert(6);

    std::cout << "unordered_set 插入操作后:\n";
    std::cout << "ref: " << ref << "\n"; // 引用仍然有效，迭代器可能失效
}

void unordered_map_example() {
    std::unordered_map<int, int> myumap = {{1, 10}, {2, 20}, {3, 30}};
    auto it = myumap.find(2);
    auto ref = it->second;

    // 插入操作
    myumap.insert({4, 40});

    std::cout << "unordered_map 插入操作后:\n";
    std::cout << "ref: " << ref << "\n"; // 引用仍然有效，迭代器可能失效
}

int main() {
    set_example();
    map_example();
    unordered_set_example();
    unordered_map_example();
    return 0;
}
```

#### 删除和提取操作示例

```cpp
#include <iostream>
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>

void set_example() {
    std::set<int> myset = {1, 2, 3, 4, 5};
    auto it = myset.find(3);
    auto ref = *it;

    // 删除操作
    myset.erase(it);

    std::cout << "set 删除操作后:\n";
    // 迭代器 it 和引用 ref 已失效，访问它们会导致未定义行为
    // std::cout << "it: " << *it << ", ref: " << ref << "\n";
}

void map_example() {
    std::map<int, int> mymap = {{1, 10}, {2, 20}, {3, 30}};
    auto it = mymap.find(2);
    auto ref = it->second;

    // 删除操作
    mymap.erase(it);

    std::cout << "map 删除操作后:\n";
    // 迭代器 it 和引用 ref 已失效，访问它们会导致未定义行为
    // std::cout << "it: (" << it->first << ", " << it->second << "), ref: " << ref << "\n";
}

void unordered_set_example() {
    std::unordered_set<int> myuset = {1, 2, 3, 4, 5};
    auto it = myuset.find(3);
    auto ref = *it;

    // 删除操作
    myuset.erase(it);

    std::cout << "unordered_set 删除操作后:\n";
    // 迭代器 it 已失效，引用 ref 仍然有效
    // std::cout << "it: " << *it << ", ref: " << ref << "\n";
}

void unordered_map_example() {
    std::unordered_map<int, int> myumap = {{1, 10}, {2, 20}, {3, 30}};
    auto it = myumap.find(2);
    auto ref = it->second;

    // 删除操作
    myumap.erase(it);

    std::cout << "unordered_map 删除操作后:\n";
    // 迭代器 it 已失效，引用 ref 仍然有效
    // std::cout << "it: (" << it->first << ", " << it->second << "), ref: " << ref << "\n";
}

int main() {
    set_example();
    map_example();
    unordered_set_example();
    unordered_map_example();
    return 0;
}
```

### 总结
- **插入和置入**：关联容器和无序关联容器的插入和置入操作不会使引用失效。关联容器的插入和置入操作不会使迭代器失效，而无序关联容器可能会使迭代器失效。
- **删除和提取**：删除和提取操作仅会使被删除或移除元素的迭代器和引用失效。无序关联容器中的引用仍然有效。

## 迭代器失效总结图

![Pasted image 20240710142700.png](/img/user/CPP/assert/Pasted%20image%2020240710142700.png)