The user wants a detailed tutorial on the five `<algorithm>` functions: `std::for_each`, `std::copy`, `std::find`, `std::sort`, and `std::swap`. I'll write a comprehensive tutorial from shallow to deep, in Chinese.

# `<algorithm>` 核心五算法 完全教程

---

## 前置概念：算法如何"看"容器？

在学具体函数之前，先理解一个核心模型：

```
容器 (vector/list/...)  →  迭代器 (begin/end)  →  算法 (sort/find/...)
     存储数据                  提供统一接口              操作数据
```

**算法只认迭代器，不认容器本身。** 这就是为什么一个 `std::sort` 能同时作用于 `vector`、`array`、`deque`——只要你给它一对合适的迭代器就行。

所有算法都需要包含头文件：

```cpp
#include <algorithm>
```

---

## 一、`std::for_each` — 遍历并操作每个元素

### 🌱 基础：它是什么？

`for_each` 就是一个"对每个元素执行某件事"的工具，等价于手写的 for 循环，但更简洁、更符合语义。

**函数签名：**

```cpp
template<class InputIt, class UnaryFunction>
UnaryFunction for_each(InputIt first, InputIt last, UnaryFunction f);
```

参数说明：

- `first`、`last`：**左闭右开**区间 `[first, last)`，标准 STL 惯例

- `f`：对每个元素执行的**可调用对象**（函数、函数指针、Lambda、仿函数均可）

### 🌿 第一层：用普通函数

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

void print(int x) {
    std::cout << x << " ";
}

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::for_each(v.begin(), v.end(), print);
    // 输出：1 2 3 4 5
}
```

### 🌿 第二层：用 Lambda（最常用）

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// 遍历打印
std::for_each(v.begin(), v.end(), [](<int x>) {
    std::cout << x << " ";
});

// 遍历修改（注意参数要加 & 引用）
std::for_each(v.begin(), v.end(), [](int& x) {
    x *= 2;
});
// v 现在是 {2, 4, 6, 8, 10}
```

> ⚠️ **关键区别**：
>
> 
>
> - `[](int x)`  — 按值传入，**只能读，不能修改原容器**
>
> - `[](int& x)` — 按引用传入，**可以修改原容器中的值**

### 🌳 第三层：Lambda 捕获外部变量

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
int sum = 0;

// [&sum] 表示"捕获外部变量 sum 的引用"
std::for_each(v.begin(), v.end(), [&sum](<int x>) {
    sum += x;
});

std::cout << "总和：" << sum << std::endl; // 输出：总和：15
```

**捕获方式速查：**

| 捕获语法 | 含义                                    |
| -------- | --------------------------------------- |
| []       | 不捕获任何外部变量                      |
| [x]      | 按值捕获变量 x（Lambda 内是副本）       |
| [&x]     | 按引用捕获变量 x（Lambda 内可修改原值） |
| [=]      | 按值捕获所有外部变量                    |
| [&]      | 按引用捕获所有外部变量                  |
| [=, &x]  | 其他按值，x 按引用                      |

### 🌳 第四层：用于自定义对象

```cpp
struct Student {
    std::string name;
    int score;
};

std::vector<Student> students = {
    {"Alice", 92},
    {"Bob",   78},
    {"Carol", 85}
};

// 打印所有不及格学生
std::for_each(students.begin(), students.end(), [](const Student& s) {
    if (s.score < 80) {
        std::cout << s.name << " 需要补考！\n";
    }
});
// 输出：Bob 需要补考！
```

### 💡 `for_each` vs 范围 for 循环

```cpp
// 两者等价
std::for_each(v.begin(), v.end(), [](int x) { std::cout << x; });
for (int x : v) { std::cout << x; }

// for_each 的优势：可以指定范围（只遍历一部分）
std::for_each(v.begin() + 1, v.end() - 1, [](int x) {
    std::cout << x; // 跳过首尾，只遍历中间部分
});
```

---

## 二、`std::copy` — 复制元素到另一个位置

### 🌱 基础：它是什么？

`copy` 将一段区间的元素**逐个复制**到另一个位置。

**函数签名：**

```cpp
template<class InputIt, class OutputIt>
OutputIt copy(InputIt first, InputIt last, OutputIt d_first);
```

参数说明：

- `first`、`last`：**源区间** `[first, last)`

- `d_first`：**目标位置**的起始迭代器（目标空间必须已经足够大！）

- 返回值：指向目标区间最后一个被复制元素的**下一个位置**的迭代器

### 🌿 第一层：从一个 vector 复制到另一个

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> src = {1, 2, 3, 4, 5};
    std::vector<int> dst(5); // ⚠️ 必须预先分配好空间！

    std::copy(src.begin(), src.end(), dst.begin());

    for (int x : dst) std::cout << x << " ";
    // 输出：1 2 3 4 5
}
```

> ⚠️ **常见错误：目标容器未分配空间**
>
>
> ```cpp
> std::vector<int> dst; // 空的！
> std::copy(src.begin(), src.end(), dst.begin()); // ❌ 未定义行为！
> ```

### 🌿 第二层：用 `back_inserter` 自动追加

不想手动分配空间？用 `std::back_inserter`（来自 `<iterator>`）：

```cpp
#include <iterator> // back_inserter

std::vector<int> src = {1, 2, 3, 4, 5};
std::vector<int> dst; // 空的也没关系

std::copy(src.begin(), src.end(), std::back_inserter(dst));
// dst 自动变为 {1, 2, 3, 4, 5}
```

`back_inserter(dst)` 会在每次赋值时自动调用 `dst.push_back()`，不需要预分配空间。

### 🌿 第三层：只复制一部分（指定范围）

```cpp
std::vector<int> v = {10, 20, 30, 40, 50};
std::vector<int> out(3);

// 只复制第2到第4个元素（索引1~3）
std::copy(v.begin() + 1, v.begin() + 4, out.begin());
// out = {20, 30, 40}
```

### 🌳 第四层：配合 `ostream_iterator` 直接输出到控制台

这是 `copy` 的高阶用法——无需循环，直接打印：

```cpp
#include <iterator>  // ostream_iterator

std::vector<int> v = {1, 2, 3, 4, 5};

// 直接把容器内容"复制"到标准输出，以空格分隔
std::copy(v.begin(), v.end(),
          std::ostream_iterator<int>(std::cout, " "));
// 输出：1 2 3 4 5
```

`ostream_iterator<int>(std::cout, " ")` 是一种特殊的**输出迭代器**，每次"赋值"都会向 `cout` 输出一个元素。

### 🌳 第五层：在类的深拷贝中使用

这是你项目中实际使用 `copy` 的场景——替代手写的 for 循环来实现**深拷贝**：

```cpp
// ❌ 手动循环（繁琐）
for (int i = 0; i < other.m_size; ++i) {
    m_pAddress[i] = other.m_pAddress[i];
}

// ✅ 使用 std::copy（简洁、安全）
std::copy(other.m_pAddress, other.m_pAddress + other.m_size, m_pAddress);
```

> 💡 `copy` 接受**裸指针**作为迭代器——在 C++ 中，指针本身就是一种随机访问迭代器。

### `copy` 家族扩展

| 函数               | 功能                                 |
| ------------------ | ------------------------------------ |
| std::copy          | 正向复制                             |
| std::copy_if       | 有条件地复制（满足谓词才复制）       |
| std::copy_n        | 复制前 n 个元素                      |
| std::copy_backward | 从末尾向前复制（防止区间重叠时覆盖） |

```cpp
// copy_if：只复制偶数
std::copy_if(v.begin(), v.end(), std::back_inserter(out), [](int x) {
    return x % 2 == 0;
});
```

---

## 三、`std::find` — 线性查找元素

### 🌱 基础：它是什么？

`find` 在一段范围内**从头到尾线性查找**第一个等于目标值的元素，返回指向它的迭代器。

**函数签名：**

```cpp
template<class InputIt, class T>
InputIt find(InputIt first, InputIt last, const T& value);
```

- 找到：返回**指向该元素的迭代器**

- 未找到：返回 `last`（即 `end()`）

> ⚠️ **必须判断返回值！** 使用前一定要检查是否等于 `end()`，否则解引用会导致未定义行为。

### 🌿 第一层：在 vector 中查找整数

```cpp
std::vector<int> v = {10, 20, 30, 40, 50};

auto it = std::find(v.begin(), v.end(), 30);

if (it != v.end()) {
    std::cout << "找到了：" << *it << std::endl;       // 找到了：30
    std::cout << "位置索引：" << std::distance(v.begin(), it); // 2
} else {
    std::cout << "未找到" << std::endl;
}
```

### 🌿 第二层：查找后进行操作

找到之后，迭代器不只是用来读值，还可以**直接操作该元素**：

```cpp
std::vector<int> v = {10, 20, 30, 40, 50};

auto it = std::find(v.begin(), v.end(), 30);
if (it != v.end()) {
    *it = 999;          // 修改该元素
    v.erase(it);        // 或者删除该元素
    v.insert(it, 888);  // 或者在该位置前插入
}
```

### 🌿 第三层：在自定义对象中查找

`find` 默认使用 `==` 运算符比较，所以自定义类型需要**重载 `operator==`**：

```cpp
struct Point {
    int x, y;
    // 必须重载 == 才能用 find
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

std::vector<Point> points = {{1, 2}, {3, 4}, {5, 6}};
auto it = std::find(points.begin(), points.end(), Point{3, 4});

if (it != points.end()) {
    std::cout << "找到点：(" << it->x << ", " << it->y << ")\n";
}
```

### 🌳 第四层：`find_if` — 按条件查找（更灵活）

当你不是要查找**具体值**，而是要查找**满足某种条件**的元素时，用 `find_if`：

```cpp
std::vector<int> v = {1, 3, 7, 4, 9, 2};

// 找第一个偶数
auto it = std::find_if(v.begin(), v.end(), [](int x) {
    return x % 2 == 0;
});

if (it != v.end()) {
    std::cout << "第一个偶数：" << *it << std::endl; // 4
}
```

```cpp
// 找第一个 score 大于 90 的学生
auto it = std::find_if(students.begin(), students.end(),
    [](const Student& s) {
        return s.score > 90;
    });
```

### 🌳 第五层：配合反向迭代器从末尾查找

在你的迭代器专题笔记中出现过的用法——从后向前查找：

```cpp
std::vector<int> v = {1, 2, 3, 1, 2, 3};

// 正向：找第一个 1（索引 0）
auto fwd = std::find(v.begin(), v.end(), 1);

// 反向：找最后一个 1（索引 3）
auto rev = std::find(v.rbegin(), v.rend(), 1);

// 将反向迭代器转换为正向迭代器（.base() 指向其后一位）
if (rev != v.rend()) {
    auto pos = std::prev(rev.base()); // 转换后指向该元素本身
    std::cout << "最后一个1的索引：" << std::distance(v.begin(), pos); // 3
}
```

### `find` 家族速查

| 函数               | 功能                               |
| ------------------ | ---------------------------------- |
| std::find          | 查找等于某值的第一个元素           |
| std::find_if       | 查找满足谓词的第一个元素           |
| std::find_if_not   | 查找不满足谓词的第一个元素         |
| std::find_end      | 查找子序列最后一次出现的位置       |
| std::find_first_of | 查找集合中任意元素第一次出现的位置 |

---

## 四、`std::sort` — 排序

### 🌱 基础：它是什么？

`sort` 对一段区间进行**原地排序**，默认**升序**（从小到大）。

**函数签名：**

```cpp
template<class RandomIt>
void sort(RandomIt first, RandomIt last);

template<class RandomIt, class Compare>
void sort(RandomIt first, RandomIt last, Compare comp);
```

> ‼️ **重要限制**：`sort` 要求**随机访问迭代器**。
>
> 
>
> - ✅ `vector`、`array`、`deque` — 可用
>
> - ❌ `list`、`forward_list` — 不可用（只能用它们自己的 `.sort()` 成员函数）

### 🌿 第一层：基本升序排序

```cpp
std::vector<int> v = {5, 3, 1, 4, 2};
std::sort(v.begin(), v.end());
// v = {1, 2, 3, 4, 5}
```

### 🌿 第二层：改变排序方向

```cpp
#include <functional> // std::greater

std::vector<int> v = {5, 3, 1, 4, 2};

// 降序：使用标准库提供的比较器
std::sort(v.begin(), v.end(), std::greater<int>());
// v = {5, 4, 3, 2, 1}

// 等价写法（用 Lambda）
std::sort(v.begin(), v.end(), [](int a, int b) {
    return a > b; // 返回 true 表示 a 应排在 b 前面
});
```

### 🌿 第三层：对部分范围排序

```cpp
std::vector<int> v = {50, 3, 1, 4, 2, 99};

// 只排序中间部分，首尾元素保持不动
std::sort(v.begin() + 1, v.end() - 1);
// v = {50, 1, 2, 3, 4, 99}
```

### 🌳 第四层：自定义对象排序

这是最常见的实际需求——按照结构体某个字段排序：

```cpp
struct Student {
    std::string name;
    int score;
};

std::vector<Student> students = {
    {"Alice", 92},
    {"Bob",   78},
    {"Carol", 85}
};

// 按分数升序
std::sort(students.begin(), students.end(),
    [](const Student& a, const Student& b) {
        return a.score < b.score;
    });
// Bob(78) → Carol(85) → Alice(92)

// 按名字字母顺序
std::sort(students.begin(), students.end(),
    [](const Student& a, const Student& b) {
        return a.name < b.name;
    });
```

或者在类内重载 `operator<`，让 `sort` 默认使用：

```cpp
struct Student {
    std::string name;
    int score;

    // 重载 < 后，sort 不需要额外的比较器
    bool operator<(const Student& other) const {
        return score < other.score; // 按分数升序
    }
};

std::sort(students.begin(), students.end()); // 直接使用
```

### 🌳 第五层：理解比较器规则（严格弱序）

`sort` 的比较函数必须满足**严格弱序**（Strict Weak Ordering）：

```
comp(a, a) == false          // 不可反射（元素与自身比较必须返回 false）
comp(a, b) → !comp(b, a)     // 非对称性
comp(a, b) && comp(b, c) → comp(a, c)  // 传递性
```

```cpp
// ✅ 正确：严格小于
[](int a, int b) { return a < b; }

// ❌ 错误：使用 <= 违反严格弱序，可能导致程序崩溃！
[](int a, int b) { return a <= b; }
```

### `sort` 家族速查

| 函数              | 功能                                    | 复杂度      |
| ----------------- | --------------------------------------- | ----------- |
| std::sort         | 完整排序                                | O(n log n)  |
| std::stable_sort  | 稳定排序（相等元素保持原相对顺序）      | O(n log² n) |
| std::partial_sort | 只排好前 k 个元素                       | O(n log k)  |
| std::nth_element  | 保证第 n 个元素是正确的，其余不保证有序 | O(n)        |

```cpp
// stable_sort：相同分数的学生，保持原来的顺序
std::stable_sort(students.begin(), students.end(),
    [](const Student& a, const Student& b) {
        return a.score < b.score;
    });
```

---

## 五、`std::swap` — 交换两个值

### 🌱 基础：它是什么？

`swap` 交换两个变量（或对象）的值。这听起来简单，但在现代 C++ 中，它有**深远的设计意义**。

**函数签名：**

```cpp
template<class T>
void swap(T& a, T& b);
```

### 🌿 第一层：交换基础类型

```cpp
int a = 10, b = 20;
std::swap(a, b);
std::cout << a << " " << b; // 20 10
```

相比手动用临时变量交换，`std::swap` 更简洁：

```cpp
// 手动（繁琐）
int tmp = a;
a = b;
b = tmp;

// std::swap（简洁）
std::swap(a, b);
```

### 🌿 第二层：交换容器（非常高效！）

`std::swap` 对标准库容器做了**特化优化**——交换两个 `vector` 时，并不是逐个元素复制，而是只交换内部指针（O(1) 常数时间）：

```cpp
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2 = {100, 200};

std::swap(v1, v2);
// v1 = {100, 200}，v2 = {1, 2, 3}
// 这是 O(1) 操作！不是 O(n)！
```

> 💡 `vector` 也有自己的成员函数 `.swap()`，两者等价：`v1.swap(v2)`

### 🌳 第三层：在类中正确使用 swap（copy-and-swap 惯用法）

这是你项目中实际用到 `swap` 的场景——在移动赋值运算符中：

```cpp
class MyArray {
    int* m_data;
    int  m_size;

public:
    // 移动赋值运算符中使用 swap
    MyArray& operator=(MyArray&& other) noexcept {
        using std::swap;          // ADL（参数依赖查找）
        swap(m_data, other.m_data);
        swap(m_size, other.m_size);
        return *this;
        // other 析构时，原来 this 的资源被释放
    }
};
```

**为什么写 `using std::swap;` 而不直接 `std::swap()`？**

这是 C++ 的 **ADL（Argument-Dependent Lookup，参数依赖查找）** 机制：

```cpp
// ❌ 直接用 std::swap：只会找 std 命名空间里的版本
std::swap(a, b);

// ✅ using std::swap + 不带命名空间调用
// 编译器会优先查找该类型自定义的 swap，找不到才用 std::swap
using std::swap;
swap(a, b); // 如果类型有自定义 swap，用自定义的（更高效）；否则用 std 的
```

### 🌳 第四层：实现 copy-and-swap 惯用法（完整示例）

这是 C++ 中实现异常安全的拷贝赋值运算符的**最佳实践**：

```cpp
class MyArray {
    int* m_data;
    int  m_size;
    int  m_capacity;

public:
    // 构造
    MyArray(int cap) : m_size(0), m_capacity(cap), m_data(new int[cap]) {}

    // 拷贝构造
    MyArray(const MyArray& other)
        : m_size(other.m_size),
          m_capacity(other.m_capacity),
          m_data(new int[other.m_capacity])
    {
        std::copy(other.m_data, other.m_data + other.m_size, m_data);
    }

    // 析构
    ~MyArray() { delete[] m_data; }

    // ✅ copy-and-swap 惯用法的拷贝赋值
    // 1. 参数按值传入，触发拷贝构造（强异常安全保证！）
    // 2. 用 swap 把新对象的资源和 this 交换
    // 3. other（临时对象）析构，释放 this 原来的旧资源
    MyArray& operator=(MyArray other) {   // 注意：按值传入！
        using std::swap;
        swap(m_data,     other.m_data);
        swap(m_size,     other.m_size);
        swap(m_capacity, other.m_capacity);
        return *this;
    }
};
```

**这个模式为什么好？**

```
MyArray a = ...;
MyArray b = ...;
a = b;
```

执行步骤：

1. `b` 按值传入 `operator=`，触发**拷贝构造** → 得到临时 `other`

2. 若拷贝构造抛异常，`this`（即 `a`）**完全没有被修改**（强异常安全）

3. 若成功，`swap` 交换资源（不会抛异常）

4. `other` 析构，自动释放 `a` 原来的资源

---

## 六、综合实战示例

把五个算法组合在一起，处理一个学生成绩管理的场景：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>
#include <string>

struct Student {
    std::string name;
    int score;
    bool operator==(const Student& o) const { return name == o.name; }
    bool operator<(const Student& o) const  { return score < o.score; }
};

int main() {
    std::vector<Student> students = {
        {"Alice", 92}, {"Bob", 78}, {"Carol", 85},
        {"Dave", 55},  {"Eve", 90}
    };

    // 1. for_each：统计不及格人数
    int fail_count = 0;
    std::for_each(students.begin(), students.end(), [&fail_count](<const Student& s>) {
        if (s.score < 60) ++fail_count;
    });
    std::cout << "不及格人数：" << fail_count << "\n"; // 1

    // 2. find_if：找到第一个不及格的学生
    auto fail_it = std::find_if(students.begin(), students.end(),
        [](const Student& s) { return s.score < 60; });
    if (fail_it != students.end())
        std::cout << "第一个不及格：" << fail_it->name << "\n"; // Dave

    // 3. sort：按分数升序排列
    std::sort(students.begin(), students.end()); // 使用 operator<

    // 4. copy：打印排名后的结果
    std::cout << "排名（低→高）：";
    std::for_each(students.begin(), students.end(), [](const Student& s) {
        std::cout << s.name << "(" << s.score << ") ";
    });
    std::cout << "\n";
    // Dave(55) Bob(78) Carol(85) Eve(90) Alice(92)

    // 5. swap：交换第一名和最后一名的位置
    std::swap(students.front(), students.back());
    std::cout << "交换后：" << students.front().name
              << " 和 " << students.back().name << "\n";
    // Alice 和 Dave

    // 6. copy + ostream_iterator：输出最终成绩单到控制台
    std::vector<int> scores;
    std::for_each(students.begin(), students.end(),
        [&scores](<const Student& s>) { scores.push_back(s.score); });
    std::copy(scores.begin(), scores.end(),
              std::ostream_iterator<int>(std::cout, " "));
    // 92 78 85 90 55
}
```

---

## 总结对比表

| 算法              | 核心作用                  | 必要条件          | 返回值                       |
| ----------------- | ------------------------- | ----------------- | ---------------------------- |
| for_each(b, e, f) | 对每个元素执行 f          | 输入迭代器        | f 本身                       |
| copy(b, e, d)     | 将 [b,e) 复制到 d         | 输入+输出迭代器   | 目标区间末尾的下一个         |
| find(b, e, val)   | 查找第一个等于 val 的元素 | 输入迭代器        | 指向找到元素的迭代器（或 e） |
| sort(b, e, cmp?)  | 对 [b,e) 排序             | 随机访问迭代器    | void                         |
| swap(a, b)        | 交换 a 和 b               | 可移动/可拷贝类型 | void                         |

> 🎯 **记住这条核心设计原则**：STL 算法通过**迭代器**解耦了"数据存储"和"数据操作"——算法不关心你用的是什么容器，只要给它合适的迭代器，它就能正确工作。这就是 C++ 泛型编程的精髓所在。