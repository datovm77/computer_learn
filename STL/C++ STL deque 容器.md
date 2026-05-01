# C++ STL deque 容器 完整教程

---

## 一、deque 概述

### 1.1 什么是 deque？

`deque`（**d**ouble-**e**nded **que**ue，双端队列）是 C++ STL 中的一种**序列式容器**。它允许在容器的**头部和尾部**都能高效地进行插入和删除操作，时间复杂度为 **O(1)**。

```
#include <deque>
```

### 1.2 deque 与 vector 的对比

| 特性          | vector                     | deque                                      |
| ------------- | -------------------------- | ------------------------------------------ |
| 内存结构      | 一段连续内存               | 分段连续内存（由多个固定大小的缓冲区组成） |
| 头部插入/删除 | O(n)，非常慢               | O(1)，非常快                               |
| 尾部插入/删除 | O(1)（均摊）               | O(1)                                       |
| 随机访问      | O(1)，极快                 | O(1)，但比 vector 略慢                     |
| 迭代器稳定性  | 插入可能导致所有迭代器失效 | 中间插入/删除使所有迭代器失效              |
| 内存连续性    | 完全连续                   | 逻辑连续，物理分段                         |
| capacity()    | ✅ 有                       | ❌ 没有                                     |

### 1.3 deque 的内部结构（理解核心）

`deque` 并**不是**一整块连续内存，而是由一个**中控器（map）**管理的多段缓冲区：

```
中控器 (map)
┌───────┐
│  ptr  │ ──→ [ 缓冲区 0: elem elem elem elem ]
├───────┤
│  ptr  │ ──→ [ 缓冲区 1: elem elem elem elem ]
├───────┤
│  ptr  │ ──→ [ 缓冲区 2: elem elem elem elem ]
├───────┤
│  ptr  │ ──→ [ 缓冲区 3: elem elem elem elem ]
└───────┘
```

- 中控器是一个小型的**指针数组**，每个指针指向一段固定大小的**缓冲区（buffer）**。
- 当头部或尾部需要扩展时，只需分配一个**新的缓冲区**并在中控器中注册其指针，无需搬移已有数据。
- 这就是 `deque` 能在头尾都做到 O(1) 插入/删除的根本原因。

> **核心理解**：`deque` 对外表现为连续存储（支持 `[]` 随机访问），但内部是分段管理的。迭代器内部维护了"当前缓冲区指针"和"当前元素在缓冲区中的位置"等信息，跨缓冲区时自动跳转，因此比 `vector` 的迭代器开销略大。

---

## 二（38）、deque 容器 - 构造函数

`deque` 提供了多种构造方式，与 `vector` 非常相似。

### 2.1 默认构造

```
deque<int> d;
// 创建一个空的 deque，元素类型为 int
```

这是最基础的用法，创建一个不包含任何元素的空 `deque`。

### 2.2 填充构造（n 个相同元素）

```
deque<int> d(10, 42);
// 创建包含 10 个元素、每个元素值为 42 的 deque
```

- 第一个参数 `10`：元素个数
- 第二个参数 `42`：每个元素的初始值

如果省略第二个参数：

```
deque<int> d(5);
// 创建包含 5 个元素、每个元素为 0（int 的值初始化）的 deque
```

### 2.3 区间构造（迭代器范围）

```
deque<int> d1 = {10, 20, 30, 40, 50};
deque<int> d2(d1.begin() + 1, d1.begin() + 4);
// d2 = {20, 30, 40}
```

区间是**左闭右开** `[first, last)`，即包含 `first` 指向的元素，不包含 `last` 指向的元素。

也可以从**其他容器**或**数组**构造：

```
int arr[] = {1, 2, 3, 4, 5};
deque<int> d(arr, arr + 5);       // 从 C 风格数组构造
// d = {1, 2, 3, 4, 5}

vector<int> v = {10, 20, 30};
deque<int> d2(v.begin(), v.end()); // 从 vector 构造
// d2 = {10, 20, 30}
```

### 2.4 拷贝构造

```
deque<int> d1 = {1, 2, 3};
deque<int> d2(d1);   // 拷贝构造，d2 是 d1 的一份完整副本
```

拷贝构造执行的是**深拷贝**，修改 `d2` 不会影响 `d1`。

### 2.5 移动构造（C++11）

```
deque<int> d1 = {1, 2, 3, 4, 5};
deque<int> d2(std::move(d1));
// d2 拥有了 d1 的所有资源
// d1 此时处于"有效但未指定"的状态（通常为空）
```

移动构造通过"窃取"源对象的内部资源来避免深拷贝，效率极高，时间复杂度 O(1)。

### 2.6 初始化列表构造（C++11）

```
deque<int> d = {1, 2, 3, 4, 5};
// 或
deque<int> d{1, 2, 3, 4, 5};
```

这是最常用、最直观的方式之一。

### 2.7 综合示例

```
#include <iostream>
#include <deque>
using namespace std;

void printDeque(const deque<int>& d) {
    for (const auto& elem : d) {
        cout << elem << " ";
    }
    cout << endl;
}

int main() {
    // 1. 默认构造
    deque<int> d1;
    cout << "d1 (默认构造): ";
    printDeque(d1);  // (空)

    // 2. 填充构造
    deque<int> d2(5, 100);
    cout << "d2 (5个100): ";
    printDeque(d2);  // 100 100 100 100 100

    // 3. 区间构造
    deque<int> d3(d2.begin(), d2.begin() + 3);
    cout << "d3 (d2的前3个): ";
    printDeque(d3);  // 100 100 100

    // 4. 拷贝构造
    deque<int> d4(d3);
    cout << "d4 (拷贝d3): ";
    printDeque(d4);  // 100 100 100

    // 5. 初始化列表构造
    deque<int> d5 = {10, 20, 30, 40, 50};
    cout << "d5 (初始化列表): ";
    printDeque(d5);  // 10 20 30 40 50

    // 6. 移动构造
    deque<int> d6(std::move(d5));
    cout << "d6 (移动自d5): ";
    printDeque(d6);  // 10 20 30 40 50
    cout << "d5 (被移动后): ";
    printDeque(d5);  // (通常为空)

    return 0;
}
```

---

## 三（39）、deque 容器 - 赋值操作

### 3.1 operator=（赋值运算符）

```
deque<int> d1 = {1, 2, 3};
deque<int> d2;

d2 = d1;  // 拷贝赋值，d2 变为 {1, 2, 3}
```

如果 `d2` 之前有数据，赋值后会被**完全替换**。

移动赋值：

```
deque<int> d3;
d3 = std::move(d1);  // 移动赋值，d3 窃取 d1 的资源
```

初始化列表赋值：

```
deque<int> d4;
d4 = {100, 200, 300};  // 使用初始化列表赋值
```

### 3.2 assign() 函数

`assign()` 提供了更灵活的赋值方式，它会**清空当前内容**并重新赋值。

#### 3.2.1 区间版本

```
deque<int> d1 = {1, 2, 3, 4, 5};
deque<int> d2;

d2.assign(d1.begin() + 1, d1.end());  // d2 = {2, 3, 4, 5}
```

#### 3.2.2 填充版本

```
deque<int> d;
d.assign(6, 99);  // d = {99, 99, 99, 99, 99, 99}
```

#### 3.2.3 初始化列表版本（C++11）

```
deque<int> d;
d.assign({10, 20, 30, 40});  // d = {10, 20, 30, 40}
```

### 3.3 swap() 函数

`swap()` 用于**交换**两个 `deque` 的内容。由于交换的是内部指针/管理结构，而非逐元素拷贝，效率为 **O(1)**。

```
deque<int> d1 = {1, 2, 3};
deque<int> d2 = {100, 200};

d1.swap(d2);
// d1 = {100, 200}
// d2 = {1, 2, 3}
```

也可以使用非成员函数版本：

```
std::swap(d1, d2);  // 效果相同
```

### 3.4 综合示例

```
#include <iostream>
#include <deque>
using namespace std;

void printDeque(const string& name, const deque<int>& d) {
    cout << name << ": ";
    for (const auto& elem : d) {
        cout << elem << " ";
    }
    cout << "(size=" << d.size() << ")" << endl;
}

int main() {
    deque<int> d1 = {1, 2, 3, 4, 5};

    // operator= 赋值
    deque<int> d2;
    d2 = d1;
    printDeque("d2 (operator=)", d2);  // 1 2 3 4 5

    // assign 区间赋值
    deque<int> d3;
    d3.assign(d1.begin(), d1.begin() + 3);
    printDeque("d3 (assign区间)", d3);  // 1 2 3

    // assign 填充赋值
    deque<int> d4;
    d4.assign(4, 88);
    printDeque("d4 (assign填充)", d4);  // 88 88 88 88

    // assign 初始化列表
    deque<int> d5;
    d5.assign({10, 20, 30});
    printDeque("d5 (assign列表)", d5);  // 10 20 30

    // swap 交换
    cout << "\n--- swap前 ---" << endl;
    printDeque("d4", d4);  // 88 88 88 88
    printDeque("d5", d5);  // 10 20 30
    d4.swap(d5);
    cout << "--- swap后 ---" << endl;
    printDeque("d4", d4);  // 10 20 30
    printDeque("d5", d5);  // 88 88 88 88

    return 0;
}
```

---

## 四（40）、deque 容器 - 大小操作

### 4.1 size()

返回 `deque` 中当前**元素的个数**。

```
deque<int> d = {1, 2, 3, 4, 5};
cout << d.size();  // 5
```

### 4.2 max_size()

返回 `deque` 理论上**能容纳的最大元素数量**。这个值取决于系统和实现，通常是一个天文数字，实际开发中几乎不会用到。

```
deque<int> d;
cout << d.max_size();  // 例如：4611686018427387903（视平台而定）
```

### 4.3 empty()

判断 `deque` 是否为空。返回 `true` 表示为空，`false` 表示非空。

```
deque<int> d;
if (d.empty()) {
    cout << "deque 为空" << endl;
}

d.push_back(42);
if (!d.empty()) {
    cout << "deque 非空, size = " << d.size() << endl;
}
```

> **最佳实践**：判断容器是否为空时，优先使用 `empty()` 而非 `size() == 0`。`empty()` 保证 O(1) 复杂度，而 `size()` 在某些容器中（如 `std::list` 在 C++11 之前）可能是 O(n)。虽然 `deque` 的 `size()` 也是 O(1)，但养成使用 `empty()` 的习惯是好的。

### 4.4 resize()

重新调整 `deque` 的大小。

#### 4.4.1 扩大容器

```
deque<int> d = {1, 2, 3};
d.resize(6);       // 扩大到 6 个元素，新增元素用 0 填充
// d = {1, 2, 3, 0, 0, 0}

d.resize(8, 99);   // 扩大到 8 个元素，新增元素用 99 填充
// d = {1, 2, 3, 0, 0, 0, 99, 99}
```

#### 4.4.2 缩小容器

```
deque<int> d = {1, 2, 3, 4, 5};
d.resize(3);       // 缩小到 3 个元素，多余的被删除
// d = {1, 2, 3}
```

> **注意**：`resize()` 缩小时，被删除的是**尾部**的元素。

### 4.5 shrink_to_fit()（C++11）

请求释放多余的内存。这是一个**非强制**的请求，实现可以忽略它。

```
deque<int> d(1000, 1);
d.resize(10);
d.shrink_to_fit();  // 请求释放多余内存
```

> **注意**：与 `vector` 不同，`deque` **没有** `capacity()` 和 `reserve()` 函数。这是因为 `deque` 的分段内存管理方式决定了"预留容量"这个概念对它意义不大。

### 4.6 综合示例

```
#include <iostream>
#include <deque>
using namespace std;

void printDeque(const string& name, const deque<int>& d) {
    cout << name << " (size=" << d.size() << "): ";
    for (const auto& elem : d) {
        cout << elem << " ";
    }
    cout << endl;
}

int main() {
    deque<int> d = {10, 20, 30, 40, 50};
    printDeque("初始", d);          // size=5: 10 20 30 40 50

    // empty()
    cout << "是否为空: " << boolalpha << d.empty() << endl;  // false

    // resize 扩大
    d.resize(8);
    printDeque("resize(8)", d);     // size=8: 10 20 30 40 50 0 0 0

    d.resize(10, 666);
    printDeque("resize(10,666)", d); // size=10: 10 20 30 40 50 0 0 0 666 666

    // resize 缩小
    d.resize(4);
    printDeque("resize(4)", d);     // size=4: 10 20 30 40

    return 0;
}
```

---

## 五（41）、deque 容器 - 插入和删除

这是 `deque` 最强大、与 `vector` 差异最大的部分。

### 5.1 头部操作（deque 的独门绝技）

#### push_front()

```
deque<int> d = {2, 3, 4};
d.push_front(1);   // d = {1, 2, 3, 4}
d.push_front(0);   // d = {0, 1, 2, 3, 4}
```

在头部插入元素，时间复杂度 **O(1)**。这是 `deque` 相比 `vector` 的最大优势。

#### emplace_front()（C++11）

```
deque<string> d;
d.emplace_front("Hello");  // 直接在头部原地构造
```

与 `push_front()` 的区别在于：`emplace_front()` 接收构造函数的参数，在容器内**原地构造**对象，避免了临时对象的创建和拷贝/移动。

```
struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {
        cout << "构造 Point(" << x << ", " << y << ")" << endl;
    }
};

deque<Point> d;
d.push_front(Point(1, 2));    // 先构造临时对象，再移动进容器
d.emplace_front(3, 4);        // 直接在容器内构造，效率更高
```

#### pop_front()

```
deque<int> d = {1, 2, 3, 4, 5};
d.pop_front();   // d = {2, 3, 4, 5}
```

删除头部元素，时间复杂度 **O(1)**。

> ⚠️ 对空容器调用 `pop_front()` 是**未定义行为**！调用前请确认 `!d.empty()`。

### 5.2 尾部操作

#### push_back()

```
deque<int> d = {1, 2, 3};
d.push_back(4);  // d = {1, 2, 3, 4}
```

#### emplace_back()（C++11）

```
deque<int> d;
d.emplace_back(42);  // 原地构造
```

#### pop_back()

```
deque<int> d = {1, 2, 3, 4};
d.pop_back();  // d = {1, 2, 3}
```

> ⚠️ 同样，对空容器调用 `pop_back()` 也是**未定义行为**！

### 5.3 任意位置插入 insert()

`insert()` 返回一个迭代器，指向**第一个新插入的元素**。

#### 5.3.1 插入单个元素

```
deque<int> d = {1, 2, 3};
auto it = d.insert(d.begin() + 1, 99);
// d = {1, 99, 2, 3}
// it 指向 99
```

#### 5.3.2 插入 n 个相同元素

```
deque<int> d = {1, 2, 3};
d.insert(d.begin() + 1, 3, 88);
// d = {1, 88, 88, 88, 2, 3}
```

#### 5.3.3 插入迭代器区间

```
deque<int> d1 = {1, 2, 3};
deque<int> d2 = {100, 200, 300};

d1.insert(d1.begin() + 1, d2.begin(), d2.end());
// d1 = {1, 100, 200, 300, 2, 3}
```

#### 5.3.4 插入初始化列表（C++11）

```
deque<int> d = {1, 2, 3};
d.insert(d.begin() + 1, {10, 20, 30});
// d = {1, 10, 20, 30, 2, 3}
```

#### 5.3.5 原地构造插入 emplace()（C++11）

```
deque<pair<int, string>> d;
d.emplace(d.begin(), 1, "hello");
// 直接在头部原地构造 pair<int, string>{1, "hello"}
```

> **性能注意**：在 `deque` 的**中间位置**进行插入，时间复杂度为 **O(n)**，因为需要移动元素。但 `deque` 的实现有一个优化：它会选择移动元素**较少的一端**。例如，如果插入位置更靠近头部，则移动头部的元素；如果更靠近尾部，则移动尾部的元素。

### 5.4 删除操作

#### erase() 删除单个元素

```
deque<int> d = {1, 2, 3, 4, 5};
auto it = d.erase(d.begin() + 2);  // 删除第 3 个元素 (值为 3)
// d = {1, 2, 4, 5}
// it 指向删除后该位置的元素，即 4
```

#### erase() 删除区间

```
deque<int> d = {1, 2, 3, 4, 5};
d.erase(d.begin() + 1, d.begin() + 4);  // 删除索引 1~3 的元素
// d = {1, 5}
```

#### clear() 清空所有元素

```
deque<int> d = {1, 2, 3, 4, 5};
d.clear();
cout << d.size();  // 0
cout << d.empty(); // true
```

### 5.5 综合示例

```
#include <iostream>
#include <deque>
using namespace std;

void printDeque(const string& label, const deque<int>& d) {
    cout << label << ": ";
    for (const auto& elem : d) {
        cout << elem << " ";
    }
    cout << "(size=" << d.size() << ")" << endl;
}

int main() {
    deque<int> d;

    // === 头部和尾部操作 ===
    d.push_back(3);
    d.push_back(4);
    d.push_front(2);
    d.push_front(1);
    printDeque("push 后", d);  // 1 2 3 4

    d.pop_front();
    printDeque("pop_front 后", d);  // 2 3 4

    d.pop_back();
    printDeque("pop_back 后", d);   // 2 3

    // === 中间插入 ===
    d.insert(d.begin() + 1, 99);
    printDeque("insert(pos1, 99)", d);  // 2 99 3

    d.insert(d.end(), 3, 0);
    printDeque("insert(end, 3个0)", d); // 2 99 3 0 0 0

    d.insert(d.begin(), {-2, -1});
    printDeque("insert(begin, 列表)", d); // -2 -1 2 99 3 0 0 0

    // === 删除操作 ===
    d.erase(d.begin() + 3);  // 删除 99
    printDeque("erase(pos3)", d);  // -2 -1 2 3 0 0 0

    d.erase(d.begin() + 4, d.end());  // 删除后面的 0
    printDeque("erase(区间)", d);  // -2 -1 2 3

    // === 清空 ===
    d.clear();
    printDeque("clear 后", d);  // (空)

    return 0;
}
```

### 5.6 迭代器失效问题（⚠️ 重要）

`deque` 的迭代器失效规则比 `vector` **复杂**，必须牢记：

| 操作                  | 迭代器失效情况                       |
| --------------------- | ------------------------------------ |
| push_back()           | 所有迭代器失效，但引用和指针不受影响 |
| push_front()          | 所有迭代器失效，但引用和指针不受影响 |
| pop_back()            | 尾部迭代器 + end() 失效              |
| pop_front()           | 首部迭代器失效                       |
| 中间 insert()/erase() | 所有迭代器、引用和指针全部失效       |

> **安全法则**：在任何修改 `deque` 的操作后，**不要再使用之前保存的迭代器**，除非你非常明确地知道它们仍然有效。

---

## 六（42）、deque 容器 - 数据存取

### 6.1 operator[]（下标访问）

```
deque<int> d = {10, 20, 30, 40, 50};
cout << d[0];   // 10
cout << d[2];   // 30
d[2] = 300;     // 修改第 3 个元素
cout << d[2];   // 300
```

- **不做越界检查**，越界访问是**未定义行为**。
- 时间复杂度 O(1)，但由于 `deque` 的分段内存结构，实际上需要先定位到正确的缓冲区，再在缓冲区内定位，因此比 `vector` 的 `[]` 略慢。

### 6.2 at()（带越界检查的访问）

```
deque<int> d = {10, 20, 30};
cout << d.at(1);   // 20

try {
    cout << d.at(100);  // 越界！
} catch (const out_of_range& e) {
    cout << "越界异常: " << e.what() << endl;
}
```

- 越界时抛出 `std::out_of_range` 异常。
- 比 `[]` 稍慢（因为有检查），但更安全。

> **选择建议**：
>
> 确定索引合法时用 `d[i]`（性能优先）
> 索引来自外部输入或不确定时用 `d.at(i)`（安全优先）

### 6.3 front() 和 back()

```
deque<int> d = {10, 20, 30, 40, 50};

cout << d.front();  // 10（第一个元素）
cout << d.back();   // 50（最后一个元素）

d.front() = 100;    // 修改第一个元素
d.back()  = 500;    // 修改最后一个元素
// d = {100, 20, 30, 40, 500}
```

- `front()` 和 `back()` 返回的是**引用**，可以直接修改。
- 对空容器调用是**未定义行为**。

### 6.4 迭代器访问

`deque` 提供了完整的迭代器支持：

```
deque<int> d = {1, 2, 3, 4, 5};

// 正向迭代器
for (auto it = d.begin(); it != d.end(); ++it) {
    cout << *it << " ";
}
// 输出: 1 2 3 4 5

// 反向迭代器
for (auto rit = d.rbegin(); rit != d.rend(); ++rit) {
    cout << *rit << " ";
}
// 输出: 5 4 3 2 1

// const 迭代器 (C++11 cbegin/cend)
for (auto cit = d.cbegin(); cit != d.cend(); ++cit) {
    // *cit = 100;  // ❌ 编译错误，不能通过 const 迭代器修改
    cout << *cit << " ";
}
```

迭代器类型说明：

| 迭代器              | 类型             | 说明                       |
| ------------------- | ---------------- | -------------------------- |
| begin() / end()     | 正向可修改迭代器 | 从头到尾遍历               |
| rbegin() / rend()   | 反向可修改迭代器 | 从尾到头遍历               |
| cbegin() / cend()   | 正向只读迭代器   | 不能修改元素               |
| crbegin() / crend() | 反向只读迭代器   | 从尾到头遍历，不能修改元素 |

> `deque` 的迭代器是**随机访问迭代器（Random Access Iterator）**，支持 `+`、`-`、`<`、`>` 等运算，与 `vector` 的迭代器类别相同。

### 6.5 基于范围的 for 循环（推荐，C++11）

```
deque<int> d = {1, 2, 3, 4, 5};

// 只读遍历
for (const auto& elem : d) {
    cout << elem << " ";
}

// 可修改遍历
for (auto& elem : d) {
    elem *= 2;
}
// d = {2, 4, 6, 8, 10}
```

### 6.6 综合示例

```
#include <iostream>
#include <deque>
#include <stdexcept>
using namespace std;

int main() {
    deque<int> d = {10, 20, 30, 40, 50};

    // [] 和 at()
    cout << "d[0] = " << d[0] << endl;       // 10
    cout << "d.at(4) = " << d.at(4) << endl;  // 50

    // at() 越界保护
    try {
        d.at(10);
    } catch (const out_of_range& e) {
        cout << "捕获异常: " << e.what() << endl;
    }

    // front() 和 back()
    cout << "front = " << d.front() << endl;  // 10
    cout << "back  = " << d.back() << endl;   // 50

    // 利用 front/back 修改
    d.front() = 999;
    d.back()  = 111;
    cout << "修改后: ";
    for (const auto& elem : d) {
        cout << elem << " ";
    }
    cout << endl;
    // 999 20 30 40 111

    // 反向遍历
    cout << "反向: ";
    for (auto rit = d.rbegin(); rit != d.rend(); ++rit) {
        cout << *rit << " ";
    }
    cout << endl;
    // 111 40 30 20 999

    return 0;
}
```

---

## 七（43）、deque 容器 - 排序操作

### 7.1 deque 没有自带排序成员函数

与 `std::list` 不同，`deque` **没有** `sort()` 成员函数。但因为 `deque` 的迭代器是**随机访问迭代器**，所以可以直接使用 `<algorithm>` 中的通用排序算法。

```
#include <algorithm>
```

### 7.2 std::sort() 默认升序排序

```
deque<int> d = {50, 20, 40, 10, 30};
sort(d.begin(), d.end());
// d = {10, 20, 30, 40, 50}
```

底层使用的是**混合排序算法**（通常是 IntroSort = 快速排序 + 堆排序 + 插入排序），平均时间复杂度 **O(n log n)**。

### 7.3 自定义排序规则

#### 7.3.1 使用函数指针

```
bool descending(int a, int b) {
    return a > b;  // 降序
}

deque<int> d = {50, 20, 40, 10, 30};
sort(d.begin(), d.end(), descending);
// d = {50, 40, 30, 20, 10}
```

#### 7.3.2 使用 Lambda 表达式（推荐，C++11）

```
deque<int> d = {50, 20, 40, 10, 30};
sort(d.begin(), d.end(), [](int a, int b) {
    return a > b;  // 降序
});
// d = {50, 40, 30, 20, 10}
```

#### 7.3.3 使用标准库仿函数

```
#include <functional>

deque<int> d = {50, 20, 40, 10, 30};
sort(d.begin(), d.end(), greater<int>());  // 降序
// d = {50, 40, 30, 20, 10}

sort(d.begin(), d.end(), less<int>());     // 升序（默认行为）
// d = {10, 20, 30, 40, 50}
```

### 7.4 对自定义类型排序

```
#include <iostream>
#include <deque>
#include <algorithm>
#include <string>
using namespace std;

struct Student {
    string name;
    int age;
    double score;

    Student(const string& n, int a, double s)
        : name(n), age(a), score(s) {}
};

int main() {
    deque<Student> students = {
        {"Alice",   20, 92.5},
        {"Bob",     19, 88.0},
        {"Charlie", 21, 95.5},
        {"Diana",   20, 92.5},
        {"Eve",     19, 90.0}
    };

    // 按成绩降序，成绩相同按年龄升序
    sort(students.begin(), students.end(),
        [](const Student& a, const Student& b) {
            if (a.score != b.score) {
                return a.score > b.score;  // 成绩高的在前
            }
            return a.age < b.age;          // 成绩相同，年龄小的在前
        }
    );

    cout << "排序结果:" << endl;
    for (const auto& s : students) {
        cout << s.name << " | 年龄: " << s.age
             << " | 成绩: " << s.score << endl;
    }

    return 0;
}
```

输出：

```
排序结果:
Charlie | 年龄: 21 | 成绩: 95.5
Alice | 年龄: 20 | 成绩: 92.5
Diana | 年龄: 20 | 成绩: 92.5
Eve | 年龄: 19 | 成绩: 90
Bob | 年龄: 19 | 成绩: 88
```

### 7.5 其他常用排序相关算法

这些算法在 `<algorithm>` 中，均可用于 `deque`：

#### std::stable_sort() —— 稳定排序

```
deque<int> d = {50, 20, 40, 10, 30};
stable_sort(d.begin(), d.end());
```

`stable_sort` 保证**相等元素的相对顺序不变**。上面自定义类型排序的例子中，如果希望成绩相同的学生保持原始输入顺序，应该使用 `stable_sort`。

#### std::partial_sort() —— 部分排序

只排序出前 N 个最小（或最大）的元素：

```
deque<int> d = {50, 20, 40, 10, 30};
partial_sort(d.begin(), d.begin() + 3, d.end());
// 前 3 个位置放置最小的 3 个元素（已排序）
// d 的前三个元素为: 10 20 30，后面的元素顺序不确定
```

#### std::nth_element() —— 第 N 小元素

找到第 N 小的元素，并将其放到正确的位置：

```
deque<int> d = {50, 20, 40, 10, 30};
nth_element(d.begin(), d.begin() + 2, d.end());
// d[2] 一定是第 3 小的元素（30）
// d[2] 左边的所有元素 <= 30, 右边的所有元素 >= 30
// 但左右两侧各自内部不保证有序
```

#### std::reverse() —— 反转

```
deque<int> d = {1, 2, 3, 4, 5};
reverse(d.begin(), d.end());
// d = {5, 4, 3, 2, 1}
```

#### std::random_shuffle() / std::shuffle() —— 随机打乱

```
#include <random>

deque<int> d = {1, 2, 3, 4, 5};

// C++11 推荐方式
std::random_device rd;
std::mt19937 g(rd());
shuffle(d.begin(), d.end(), g);
// d 的元素被随机打乱
```

> `random_shuffle()` 在 C++14 中被弃用，C++17 中被移除。请使用 `shuffle()` + 随机引擎。

### 7.6 std::sort vs std::list::sort

| 特性     | std::sort(d.begin(), d.end())                 | list::sort()                               |
| -------- | --------------------------------------------- | ------------------------------------------ |
| 适用容器 | 需要随机访问迭代器（deque、vector）           | 仅适用于 list / forward_list               |
| 算法     | IntroSort, O(n log n)                         | 归并排序, O(n log n)                       |
| 稳定性   | 不稳定                                        | 稳定                                       |
| 原因     | list 的迭代器是双向迭代器，无法使用 std::sort | 成员函数可以利用链表特性进行高效的指针操作 |

### 7.7 综合示例

```
#include <iostream>
#include <deque>
#include <algorithm>
#include <functional>
using namespace std;

void printDeque(const string& label, const deque<int>& d) {
    cout << label << ": ";
    for (const auto& elem : d) {
        cout << elem << " ";
    }
    cout << endl;
}

int main() {
    deque<int> d = {5, 3, 1, 4, 2, 8, 7, 6, 9, 0};
    printDeque("原始数据", d);

    // 默认升序排序
    sort(d.begin(), d.end());
    printDeque("升序排序", d);  // 0 1 2 3 4 5 6 7 8 9

    // 降序排序
    sort(d.begin(), d.end(), greater<int>());
    printDeque("降序排序", d);  // 9 8 7 6 5 4 3 2 1 0

    // 反转
    reverse(d.begin(), d.end());
    printDeque("反转后", d);    // 0 1 2 3 4 5 6 7 8 9

    // 部分排序：找出最大的3个
    deque<int> d2 = {50, 20, 40, 10, 30, 80, 60, 70};
    partial_sort(d2.begin(), d2.begin() + 3, d2.end(), greater<int>());
    cout << "最大的3个: " << d2[0] << " " << d2[1] << " " << d2[2] << endl;
    // 80 70 60

    return 0;
}
```

---

## 八、总结与最佳实践

### 8.1 何时使用 deque？

| 场景                   | 推荐容器                                             |
| ---------------------- | ---------------------------------------------------- |
| 只在尾部频繁增删       | vector                                               |
| 在头部和尾部都频繁增删 | ✅ deque                                              |
| 需要最快的随机访问     | vector                                               |
| 需要在中间频繁增删     | list                                                 |
| 实现队列 (FIFO)        | ✅ deque（std::queue 的默认底层容器）                 |
| 实现栈 (LIFO)          | deque 或 vector（std::stack 的默认底层容器是 deque） |

### 8.2 deque API 速查表

| 分类 | 函数                           | 说明               |
| ---- | ------------------------------ | ------------------ |
| 构造 | deque<T> d                     | 默认构造           |
|      | deque<T> d(n, val)             | n 个 val           |
|      | deque<T> d(first, last)        | 迭代器区间         |
|      | deque<T> d(other)              | 拷贝构造           |
|      | deque<T> d{1,2,3}              | 初始化列表         |
| 赋值 | d1 = d2                        | 赋值运算符         |
|      | d.assign(n, val)               | 填充赋值           |
|      | d.assign(first, last)          | 区间赋值           |
|      | d.swap(d2)                     | 交换               |
| 大小 | d.size()                       | 元素个数           |
|      | d.empty()                      | 是否为空           |
|      | d.resize(n)                    | 调整大小           |
| 插入 | d.push_front(val)              | 头部插入           |
|      | d.push_back(val)               | 尾部插入           |
|      | d.emplace_front(args...)       | 头部原地构造       |
|      | d.emplace_back(args...)        | 尾部原地构造       |
|      | d.insert(pos, val)             | 指定位置插入       |
|      | d.emplace(pos, args...)        | 指定位置原地构造   |
| 删除 | d.pop_front()                  | 删除头部           |
|      | d.pop_back()                   | 删除尾部           |
|      | d.erase(pos)                   | 删除指定位置       |
|      | d.erase(first, last)           | 删除区间           |
|      | d.clear()                      | 清空               |
| 访问 | d[i]                           | 下标访问（无检查） |
|      | d.at(i)                        | 下标访问（有检查） |
|      | d.front()                      | 首元素             |
|      | d.back()                       | 尾元素             |
| 排序 | sort(d.begin(), d.end())       | 升序排序           |
|      | sort(d.begin(), d.end(), comp) | 自定义排序         |

### 8.3 注意事项清单

1. ❌ `deque` **没有** `capacity()` 和 `reserve()`
2. ⚠️ `deque` 的随机访问比 `vector` **略慢**（分段内存的代价）
3. ⚠️ 中间位置的插入/删除仍然是 **O(n)**
4. ⚠️ **迭代器失效规则**比 `vector` 更复杂，修改后不要复用旧迭代器
5. ✅ 头部操作 O(1) 是 `deque` 最大的竞争优势
6. ✅ `std::queue` 和 `std::stack` 的**默认底层容器**就是 `deque`

---

以上就是 `deque` 容器的完整教程。从构造到排序，每个知识点都配有详细解释和可运行的代码示例。建议你**逐节阅读**，将每个示例都上机运行一遍，观察输出结果，加深理解。如果有任何部分需要进一步讲解，随时告诉我！