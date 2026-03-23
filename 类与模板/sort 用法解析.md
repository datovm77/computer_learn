# `std::sort` 深度解析

## 一、最基础的用法

`std::sort` 定义在 `<algorithm>` 头文件中，最简单的用法就是对一个数组或容器排序：

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {5, 3, 1, 4, 2};
    std::sort(v.begin(), v.end());
    // 结果：1 2 3 4 5（默认升序）
}
```

对原始数组也同样适用：

```cpp
int arr[] = {5, 3, 1, 4, 2};
std::sort(arr, arr + 5);
```

------

## 二、函数签名理解

```cpp
// 版本1：使用默认的 < 运算符比较
template< class RandomIt >
void sort( RandomIt first, RandomIt last );

// 版本2：使用自定义比较器
template< class RandomIt, class Compare >
void sort( RandomIt first, RandomIt last, Compare comp );
```

**关键点：** `sort` 要求的是 **随机访问迭代器（RandomAccessIterator）**，这意味着：

| 容器      | 能否用 sort | 原因                               |
| --------- | ----------- | ---------------------------------- |
| `vector`  | ✅           | 随机访问迭代器                     |
| `deque`   | ✅           | 随机访问迭代器                     |
| `array`   | ✅           | 随机访问迭代器                     |
| `list`    | ❌           | 只有双向迭代器，需用 `list.sort()` |
| `set/map` | ❌           | 已自动排序，不支持                 |

------

## 三、自定义比较器（comp）

这是 `sort` 最灵活的地方。`comp` 是一个**严格弱序（Strict Weak Ordering）**函数，规则：`comp(a, b)` 返回 `true` 表示 `a` 应排在 `b` **前面**。

### 3.1 用函数指针

```cpp
bool descending(int a, int b) {
    return a > b; // 降序
}
std::sort(v.begin(), v.end(), descending);
```

### 3.2 用 Lambda（最常用）

```cpp
// 降序
std::sort(v.begin(), v.end(), [](int a, int b) {
    return a > b;
});

// 对字符串按长度排序
std::vector<std::string> words = {"banana", "fig", "apple", "kiwi"};
std::sort(words.begin(), words.end(), [](const std::string& a, const std::string& b) {
    return a.size() < b.size();
});
// 结果：fig kiwi apple banana
```

### 3.3 对结构体排序

```cpp
struct Student {
    std::string name;
    int score;
};

std::vector<Student> students = {{"Alice", 90}, {"Bob", 85}, {"Charlie", 92}};

// 按分数降序，分数相同按名字升序
std::sort(students.begin(), students.end(), [](const Student& a, const Student& b) {
    if (a.score != b.score) return a.score > b.score;
    return a.name < b.name;
});
```

### 3.4 重载 `operator<`

```cpp
struct Student {
    std::string name;
    int score;
    bool operator<(const Student& other) const {
        return score > other.score; // 直接定义排序逻辑
    }
};

std::sort(students.begin(), students.end()); // 无需传比较器
```



**❌ 经典错误：** 用 `<=` 而不是 `<`：

```cpp
// 错误！违反严格弱序，导致未定义行为
std::sort(v.begin(), v.end(), [](int a, int b) {
    return a <= b; // 当 a==b 时，comp(a,b) 和 comp(b,a) 都为 true，违反非对称性
});
```

------

## 五、底层算法：内省排序（Introsort）

`std::sort` 在标准库中通常实现为 **Introsort（内省排序）**，它是三种算法的混合体：

```
递归深度 < 2*log(n)  →  快速排序（QuickSort）
递归深度超限         →  堆排序（HeapSort）
子数组大小 < 阈值    →  插入排序（InsertionSort，通常阈值为16）
```

这样设计的原因：

- **快速排序**：平均 O(n log n)，缓存友好，实践中最快
- **堆排序**：保证最坏情况 O(n log n)，防止快排退化到 O(n²)
- **插入排序**：小数组上常数系数极小，比快排更快

所以 `std::sort` 的复杂度是：

- 平均：**O(n log n)**
- 最坏：**O(n log n)**（这比纯快排要好）

------

## 六、相关函数对比

| 函数                | 特点                         | 复杂度      |
| ------------------- | ---------------------------- | ----------- |
| `std::sort`         | 不稳定排序                   | O(n log n)  |
| `std::stable_sort`  | 稳定排序，相等元素保持原顺序 | O(n log² n) |
| `std::partial_sort` | 只排序前 k 个元素            | O(n log k)  |
| `std::nth_element`  | 只保证第 n 个位置元素正确    | O(n) 平均   |

```cpp
// stable_sort：相等元素原顺序不变
std::stable_sort(v.begin(), v.end());

// partial_sort：只需要最小的3个，且排好序
std::partial_sort(v.begin(), v.begin() + 3, v.end());

// nth_element：第2个位置放正确元素，两侧无序
std::nth_element(v.begin(), v.begin() + 2, v.end());
```

------

## 七、实战技巧

```cpp
// 技巧1：配合 std::greater 降序（比 lambda 简洁）
#include <functional>
std::sort(v.begin(), v.end(), std::greater<int>());

// 技巧2：只排序部分范围
std::sort(v.begin() + 2, v.begin() + 7); // 只排序下标 [2, 7)

// 技巧3：排序后用 lower_bound 二分查找
std::sort(v.begin(), v.end());
auto it = std::lower_bound(v.begin(), v.end(), 3);
```

------

## 总结脉络

```
基础使用（begin/end）
    ↓
自定义比较器（lambda / 函数 / operator<）
    ↓
严格弱序规则（避免 UB）
    ↓
底层 Introsort 原理（快排 + 堆排 + 插排）
    ↓
选择合适的变体（stable_sort / partial_sort / nth_element）
```

有哪个部分想深入展开，比如 Introsort 的实现细节，或者某种使用场景，可以继续聊！