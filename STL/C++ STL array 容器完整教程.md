# 📚 C++ STL std::array 容器完整教程

---

## 一、基本概念

### 1.1 什么是 std::array？

`std::array` 是 C++11 引入的一个**固定大小**的顺序容器，定义在头文件 `<array>` 中。它本质上是对 **C 风格数组的封装**，但提供了 STL 容器的标准接口（如迭代器、`size()`、`at()` 等），让你可以像使用其他 STL 容器一样使用它。

```
#include <array>

std::array<int, 5> arr = {1, 2, 3, 4, 5};
//         ^^^  ^
//        类型  大小（必须是编译期常量）
```

> **关键特征：大小在编译期确定，运行时不可改变。**

---

### 1.2 与 C 风格数组的对比

这是理解 `std::array` 价值的核心，我们来做一个全方位对比：

| 特性           | C 风格数组 int a[5]    | std::array<int, 5>                |
| -------------- | ---------------------- | --------------------------------- |
| 大小信息       | 退化为指针后丢失大小   | 始终可通过 .size() 获取           |
| 边界检查       | ❌ 无，越界是未定义行为 | ✅ at() 提供边界检查               |
| 赋值/拷贝      | ❌ 不能直接赋值 a = b   | ✅ 支持 = 赋值和拷贝构造           |
| 比较操作       | ❌ 不能直接比较         | ✅ 支持 ==, !=, < 等               |
| 作为函数参数   | 退化为指针，丢失大小   | 完整传递，类型安全                |
| 作为函数返回值 | ❌ 不能直接返回数组     | ✅ 可以直接返回                    |
| 迭代器支持     | 需要手动用指针         | ✅ begin(), end() 等               |
| STL 算法兼容   | 需额外处理             | ✅ 完美兼容                        |
| 性能           | 栈上分配，零开销       | 栈上分配，零开销（与 C 数组一致） |
| 内存布局       | 连续                   | 连续（与 C 数组完全一致）         |

#### 用代码感受差异：

```
#include <iostream>
#include <array>

// ===== C 风格数组的痛点 =====

void printCArray(int arr[], int size) {
    // 必须额外传递大小参数！
    // arr 已经退化为 int* 指针，丢失了大小信息
    for (int i = 0; i < size; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;
}

// ===== std::array 的优雅 =====

void printStdArray(const std::array<int, 5>& arr) {
    // 不需要额外传大小！类型本身就包含大小信息
    for (const auto& elem : arr) {  // 支持范围 for 循环
        std::cout << elem << " ";
    }
    std::cout << std::endl;
}

int main() {
    // --- C 风格数组 ---
    int cArr1[5] = {1, 2, 3, 4, 5};
    int cArr2[5] = {1, 2, 3, 4, 5};
    // cArr1 = cArr2;                 // ❌ 编译错误！不能直接赋值
    // if (cArr1 == cArr2) {}         // ⚠️ 比较的是指针地址，不是内容！

    // --- std::array ---
    std::array<int, 5> stdArr1 = {1, 2, 3, 4, 5};
    std::array<int, 5> stdArr2 = {1, 2, 3, 4, 5};
    stdArr1 = stdArr2;                // ✅ 完美工作！逐元素拷贝
    if (stdArr1 == stdArr2) {         // ✅ 逐元素比较内容
        std::cout << "两个数组相等" << std::endl;
    }

    return 0;
}
```

---

## 二、构造与初始化

### 2.1 各种构造方式

```
#include <array>
#include <iostream>

int main() {
    // 1. 默认构造（元素未初始化，值不确定）
    std::array<int, 5> arr1;
    // ⚠️ 对于基本类型，元素值是未确定的（和 C 数组行为一致）

    // 2. 聚合初始化（推荐）
    std::array<int, 5> arr2 = {1, 2, 3, 4, 5};

    // 3. 列表初始化
    std::array<int, 5> arr3{10, 20, 30, 40, 50};

    // 4. 部分初始化（剩余元素零初始化）
    std::array<int, 5> arr4 = {1, 2};
    // arr4 = {1, 2, 0, 0, 0}

    // 5. 全部零初始化
    std::array<int, 5> arr5 = {};   // 全部为 0
    std::array<int, 5> arr6{};      // 全部为 0

    // 6. 拷贝构造
    std::array<int, 5> arr7 = arr2; // 拷贝 arr2 的所有元素
    std::array<int, 5> arr8(arr2);  // 同上

    // 7. C++17 类模板参数推导（CTAD） —— 更现代的写法
    std::array arr9 = {1, 2, 3, 4, 5}; // 自动推导为 std::array<int, 5>

    // 8. C++20 std::to_array（从 C 数组转换）
    int cArr[] = {1, 2, 3};
    auto arr10 = std::to_array(cArr); // std::array<int, 3>

    return 0;
}
```

### 2.2 构造注意事项

```
// ❌ 大小不能为 0（某些编译器允许但不推荐，行为未定义）
// std::array<int, 0> empty; // 不推荐

// ❌ 大小必须是编译期常量
// int n = 5;
// std::array<int, n> arr; // ❌ 编译错误（除非 n 是 constexpr）

constexpr int N = 5;
std::array<int, N> arr; // ✅ OK

// ❌ 不同大小的 array 是不同类型！
std::array<int, 3> a3;
std::array<int, 5> a5;
// a3 = a5; // ❌ 编译错误！类型不匹配
```

---

## 三、常用接口详解

### 3.1 元素访问

#### operator[] —— 下标访问（不检查边界）

```
std::array<int, 5> arr = {10, 20, 30, 40, 50};

// 读取
std::cout << arr[0] << std::endl; // 10
std::cout << arr[4] << std::endl; // 50

// 修改
arr[2] = 999;

// ⚠️ 越界访问是未定义行为（和 C 数组一样，不做检查）
// std::cout << arr[10]; // 未定义行为！不会抛异常，可能读到垃圾值或崩溃
```

#### at() —— 安全访问（有边界检查）

```
std::array<int, 5> arr = {10, 20, 30, 40, 50};

std::cout << arr.at(0) << std::endl; // 10
std::cout << arr.at(4) << std::endl; // 50

// ✅ 越界访问会抛出 std::out_of_range 异常
try {
    std::cout << arr.at(10) << std::endl;
} catch (const std::out_of_range& e) {
    std::cerr << "越界了！错误信息: " << e.what() << std::endl;
}
```

> **何时用哪个？**
>
> 性能优先、确定不会越界 → 用 `operator[]`
> 安全优先、索引来源不可控（如用户输入） → 用 `at()`

#### front() 和 back() —— 首尾元素

```
std::array<int, 5> arr = {10, 20, 30, 40, 50};

std::cout << arr.front() << std::endl; // 10 （等价于 arr[0]）
std::cout << arr.back()  << std::endl; // 50 （等价于 arr[arr.size() - 1]）

// 也可以用来修改
arr.front() = 100;
arr.back()  = 500;
// arr = {100, 20, 30, 40, 500}

// ⚠️ 对空的 array（size 为 0）调用 front()/back() 是未定义行为
```

#### data() —— 获取底层数组指针

```
std::array<int, 5> arr = {10, 20, 30, 40, 50};

int* ptr = arr.data(); // 返回指向第一个元素的指针

// 可以像 C 指针一样使用
std::cout << *ptr       << std::endl; // 10
std::cout << *(ptr + 2) << std::endl; // 30

// 典型用途：与 C 接口互操作
// 假设有一个 C 函数：void processData(int* data, int size);
// processData(arr.data(), arr.size()); // 完美兼容！
```

> **`data()` 的核心价值**：让 `std::array` 可以无缝对接需要原始指针的 C 风格 API。

---

### 3.2 容量相关

#### size() —— 返回元素个数

```
std::array<int, 5> arr = {1, 2, 3, 4, 5};

std::cout << arr.size() << std::endl; // 5（始终为 5，编译时确定）

// size() 返回的是 size_t 类型（无符号整数）
// 在循环中使用：
for (std::size_t i = 0; i < arr.size(); ++i) {
    std::cout << arr[i] << " ";
}
```

#### empty() —— 是否为空

```
std::array<int, 5> arr = {1, 2, 3, 4, 5};
std::cout << std::boolalpha;
std::cout << arr.empty() << std::endl; // false（大小为 5，不为空）

std::array<int, 0> emptyArr; // 注意：大小为 0 的 array
std::cout << emptyArr.empty() << std::endl; // true
```

#### max_size() —— 最大容量

```
std::array<int, 5> arr;
std::cout << arr.max_size() << std::endl; // 5
// 对于 std::array，max_size() 始终等于 size()，因为大小固定
```

---

### 3.3 修改操作

#### fill() —— 用指定值填充所有元素

```
std::array<int, 5> arr;

arr.fill(0);    // arr = {0, 0, 0, 0, 0}
arr.fill(42);   // arr = {42, 42, 42, 42, 42}
arr.fill(-1);   // arr = {-1, -1, -1, -1, -1}

// 比手动循环赋值更简洁、语义更清晰
// 等价于：
// for (auto& elem : arr) elem = 42;
```

#### swap() —— 交换两个数组

```
std::array<int, 5> arr1 = {1, 2, 3, 4, 5};
std::array<int, 5> arr2 = {10, 20, 30, 40, 50};

arr1.swap(arr2);
// 现在 arr1 = {10, 20, 30, 40, 50}
// 现在 arr2 = {1, 2, 3, 4, 5}

// 也可以用 std::swap
std::swap(arr1, arr2); // 再交换回来
```

> ⚠️ **注意**：`std::array` 的 `swap()` 复杂度是 **O(N)**（逐元素交换），而 `std::vector` 的 `swap()` 是 **O(1)**（只交换内部指针）。这是因为 `array` 直接存储在栈上，没有堆指针可以交换。

---

### 3.4 迭代器

```
std::array<int, 5> arr = {10, 20, 30, 40, 50};

// 正向迭代器
for (auto it = arr.begin(); it != arr.end(); ++it) {
    std::cout << *it << " "; // 10 20 30 40 50
}

// 反向迭代器
for (auto rit = arr.rbegin(); rit != arr.rend(); ++rit) {
    std::cout << *rit << " "; // 50 40 30 20 10
}

// const 迭代器（只读）
for (auto cit = arr.cbegin(); cit != arr.cend(); ++cit) {
    // *cit = 100; // ❌ 编译错误，不能通过 const 迭代器修改
    std::cout << *cit << " ";
}

// 最推荐的方式：范围 for 循环
for (const auto& elem : arr) {
    std::cout << elem << " ";
}
```

---

## 四、与 STL 算法配合

`std::array` 完美兼容所有 STL 算法，这是它相比 C 数组的巨大优势：

```
#include <array>
#include <algorithm>
#include <numeric>
#include <iostream>

int main() {
    std::array<int, 6> arr = {5, 3, 1, 4, 2, 6};

    // 排序
    std::sort(arr.begin(), arr.end());
    // arr = {1, 2, 3, 4, 5, 6}

    // 反转
    std::reverse(arr.begin(), arr.end());
    // arr = {6, 5, 4, 3, 2, 1}

    // 查找
    auto it = std::find(arr.begin(), arr.end(), 4);
    if (it != arr.end()) {
        std::cout << "找到 4，位置索引: "
                  << std::distance(arr.begin(), it) << std::endl;
    }

    // 求和
    int sum = std::accumulate(arr.begin(), arr.end(), 0);
    std::cout << "总和: " << sum << std::endl; // 21

    // 最大/最小元素
    auto [minIt, maxIt] = std::minmax_element(arr.begin(), arr.end());
    std::cout << "最小值: " << *minIt << ", 最大值: " << *maxIt << std::endl;

    // 统计
    int count = std::count_if(arr.begin(), arr.end(),
                              [](int x) { return x > 3; });
    std::cout << "大于3的元素个数: " << count << std::endl;

    // 替换
    std::replace(arr.begin(), arr.end(), 3, 99);
    // 将所有值为 3 的元素替换为 99

    // 填充（STL 算法版本）
    std::fill(arr.begin(), arr.end(), 0);
    // 等价于 arr.fill(0)

    return 0;
}
```

---

## 五、进阶用法

### 5.1 多维数组

```
// 二维数组：3 行 4 列
std::array<std::array<int, 4>, 3> matrix = {{
    {1, 2, 3, 4},
    {5, 6, 7, 8},
    {9, 10, 11, 12}
}};

// 访问元素
std::cout << matrix[1][2] << std::endl; // 7

// 遍历
for (const auto& row : matrix) {
    for (const auto& elem : row) {
        std::cout << elem << "\t";
    }
    std::cout << std::endl;
}
```

### 5.2 作为函数参数和返回值

```
#include <array>
#include <iostream>

// ✅ 作为参数（推荐传 const 引用）
void printArray(const std::array<int, 5>& arr) {
    for (const auto& elem : arr) {
        std::cout << elem << " ";
    }
    std::cout << std::endl;
}

// ✅ 作为返回值（C 数组做不到这一点！）
std::array<int, 5> createArray() {
    return {1, 2, 3, 4, 5};
}

// ✅ 使用模板让函数支持任意大小
template <std::size_t N>
void printAnyArray(const std::array<int, N>& arr) {
    std::cout << "Array of size " << N << ": ";
    for (const auto& elem : arr) {
        std::cout << elem << " ";
    }
    std::cout << std::endl;
}

int main() {
    auto arr = createArray();
    printArray(arr);

    std::array<int, 3> small = {10, 20, 30};
    std::array<int, 7> big = {1, 2, 3, 4, 5, 6, 7};

    printAnyArray(small); // Array of size 3: 10 20 30
    printAnyArray(big);   // Array of size 7: 1 2 3 4 5 6 7

    return 0;
}
```

### 5.3 constexpr 编译期计算

`std::array` 可以用于编译期计算，这是它的一个强大特性：

```
#include <array>

// 编译期创建并操作数组
constexpr std::array<int, 5> getSquares() {
    std::array<int, 5> arr{};
    for (int i = 0; i < 5; ++i) {
        arr[i] = (i + 1) * (i + 1);
    }
    return arr; // {1, 4, 9, 16, 25}
}

// 编译期查表
constexpr auto squares = getSquares();
static_assert(squares[0] == 1);
static_assert(squares[4] == 25);
```

### 5.4 结构化绑定（C++17）

```
std::array<int, 3> arr = {10, 20, 30};

// 使用结构化绑定解包
auto [a, b, c] = arr;
std::cout << a << ", " << b << ", " << c << std::endl;
// 输出: 10, 20, 30
```

---

## 六、std::array vs std::vector 选择指南

| 场景                          | 推荐        | 原因                          |
| ----------------------------- | ----------- | ----------------------------- |
| 大小编译期已知且不变          | std::array  | 零开销，栈分配，最快          |
| 大小运行时确定或需动态变化    | std::vector | array 无法动态调整大小        |
| 嵌入式/实时系统               | std::array  | 无堆分配，行为可预测          |
| 大数据量（数万元素以上）      | std::vector | 栈空间有限（通常几 MB）       |
| 需要 push_back / erase 等操作 | std::vector | array 不支持增删元素          |
| 与 C API 互操作               | 都可以      | 都能通过 .data() 获取原始指针 |
| 需要编译期计算                | std::array  | 完美支持 constexpr            |

---

## 七、完整综合示例

```
#include <iostream>
#include <array>
#include <algorithm>
#include <numeric>
#include <stdexcept>

// 打印任意大小的 int 数组
template <std::size_t N>
void printArray(const std::string& label, const std::array<int, N>& arr) {
    std::cout << label << ": { ";
    for (std::size_t i = 0; i < arr.size(); ++i) {
        std::cout << arr[i];
        if (i < arr.size() - 1) std::cout << ", ";
    }
    std::cout << " }" << std::endl;
}

int main() {
    // ===================== 构造与初始化 =====================
    std::array<int, 6> scores = {85, 92, 78, 95, 88, 73};
    printArray("原始成绩", scores);

    // ===================== 元素访问 =====================
    std::cout << "\n--- 元素访问 ---" << std::endl;
    std::cout << "用 operator[] 访问第一个: " << scores[0] << std::endl;
    std::cout << "用 at() 安全访问第三个: "   << scores.at(2) << std::endl;
    std::cout << "front() 最前: "             << scores.front() << std::endl;
    std::cout << "back()  最后: "             << scores.back() << std::endl;
    std::cout << "data() 指针值: "            << scores.data() << std::endl;

    // at() 的安全性演示
    try {
        scores.at(100); // 越界
    } catch (const std::out_of_range& e) {
        std::cout << "✅ at() 捕获越界: " << e.what() << std::endl;
    }

    // ===================== 容量 =====================
    std::cout << "\n--- 容量信息 ---" << std::endl;
    std::cout << "size(): "     << scores.size() << std::endl;
    std::cout << "max_size(): " << scores.max_size() << std::endl;
    std::cout << "empty(): "    << std::boolalpha << scores.empty() << std::endl;

    // ===================== 修改操作 =====================
    std::cout << "\n--- 修改操作 ---" << std::endl;

    // fill
    std::array<int, 6> zeros;
    zeros.fill(0);
    printArray("fill(0) 后", zeros);

    // swap
    std::array<int, 6> backup = {99, 98, 97, 96, 95, 94};
    printArray("交换前 scores", scores);
    printArray("交换前 backup", backup);
    scores.swap(backup);
    printArray("交换后 scores", scores);
    printArray("交换后 backup", backup);

    // 换回来继续用
    scores.swap(backup);

    // ===================== STL 算法 =====================
    std::cout << "\n--- STL 算法 ---" << std::endl;

    // 排序
    std::sort(scores.begin(), scores.end());
    printArray("排序后", scores);

    // 反转
    std::reverse(scores.begin(), scores.end());
    printArray("反转后（降序）", scores);

    // 求和与平均分
    int sum = std::accumulate(scores.begin(), scores.end(), 0);
    double avg = static_cast<double>(sum) / scores.size();
    std::cout << "总分: " << sum << ", 平均分: " << avg << std::endl;

    // 最高分和最低分
    auto [minIt, maxIt] = std::minmax_element(scores.begin(), scores.end());
    std::cout << "最低分: " << *minIt << ", 最高分: " << *maxIt << std::endl;

    // 及格人数（>=60）
    int passCount = std::count_if(scores.begin(), scores.end(),
                                  [](int s) { return s >= 60; });
    std::cout << "及格人数: " << passCount << "/" << scores.size() << std::endl;

    // ===================== 比较操作 =====================
    std::cout << "\n--- 比较操作 ---" << std::endl;
    std::array<int, 3> a = {1, 2, 3};
    std::array<int, 3> b = {1, 2, 3};
    std::array<int, 3> c = {1, 2, 4};
    std::cout << "a == b: " << (a == b) << std::endl; // true
    std::cout << "a == c: " << (a == c) << std::endl; // false
    std::cout << "a < c:  " << (a < c) << std::endl;  // true（字典序比较）

    return 0;
}
```

---

## 八、总结

### 📌 核心要点速记

1. **`std::array` = 更安全、更强大的 C 数组**，零运行时开销
2. **大小是类型的一部分**：`std::array<int, 3>` 和 `std::array<int, 5>` 是**不同类型**
3. 支持**拷贝、赋值、比较**（C 数组做不到）
4. `at()` 提供**边界检查**，`operator[]` 不检查但更快
5. `data()` 返回原始指针，用于**与 C 接口对接**
6. `fill()` 快速填充，`swap()` 交换内容（O(N) 复杂度）
7. 完美兼容**迭代器**和所有 **STL 算法**
8. 适合**大小固定、编译期已知**的场景；需要动态大小则用 `vector`

### 📌 接口速查表

| 接口            | 功能                 | 时间复杂度 |
| --------------- | -------------------- | ---------- |
| operator[i]     | 下标访问（不检查）   | O(1)       |
| at(i)           | 安全访问（检查越界） | O(1)       |
| front()         | 第一个元素           | O(1)       |
| back()          | 最后一个元素         | O(1)       |
| data()          | 底层数组指针         | O(1)       |
| size()          | 元素个数             | O(1)       |
| empty()         | 是否为空             | O(1)       |
| max_size()      | 最大容量（= size）   | O(1)       |
| fill(val)       | 用 val 填充所有元素  | O(N)       |
| swap(other)     | 交换两个数组         | O(N)       |
| begin()/end()   | 正向迭代器           | O(1)       |
| rbegin()/rend() | 反向迭代器           | O(1)       |

---

