# C++ STL vector 容器 —— 从入门到精通

---

## 一、vector 是什么？

`vector` 是 C++ STL（标准模板库）中最常用的**序列式容器**，可以理解为一个**动态数组**。

与普通数组的对比：

| 特性     | 普通数组 int arr[10] | vector<int> v                 |
| -------- | -------------------- | ----------------------------- |
| 大小     | 编译期固定           | 运行时动态调整                |
| 内存管理 | 手动（或栈上自动）   | 自动管理（堆上）              |
| 越界检查 | 无                   | 提供 .at() 可检查             |
| 增删元素 | 不支持               | 支持（尾部高效）              |
| 传参     | 退化为指针           | 值传递/引用传递，保留大小信息 |

**头文件**：`#include <vector>`

**底层原理**：`vector` 在堆上维护一块**连续的内存空间**。当元素数量超过当前容量时，会自动**重新分配**一块更大的内存（通常是当前容量的 1.5 倍或 2 倍，取决于编译器实现），将旧元素拷贝/移动过去，然后释放旧内存。

---

## 二、构造函数（创建 vector 的多种方式）

> **阅读约定（给初学者）**：本文有两类代码示例——
>
> - **完整可运行示例**：包含 `#include` 与 `main()`，可直接编译运行；
> - **API 片段示例**：为突出语法而省略了部分上下文（如头文件、`main()`）。练习时请补齐最少头文件，例如 `#include <vector>`、`#include <iostream>`、`#include <algorithm>`。

### 2.1 默认构造 —— 空容器

```
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v;  // 创建一个空的 int 型 vector
    cout << "size: " << v.size() << endl;       // 0
    cout << "capacity: " << v.capacity() << endl; // 0（还未分配内存）
    return 0;
}
```

> 此时 `v` 里没有任何元素，但可以随时通过 `push_back` 等操作添加。

### 2.2 指定数量与初始值

```
// 创建包含 10 个元素的 vector，每个元素初始化为 0（int 的默认值）
vector<int> v1(10);

// 创建包含 10 个元素的 vector，每个元素初始化为 42
vector<int> v2(10, 42);

// 创建包含 5 个字符串，每个都是 "hello"
vector<string> v3(5, "hello");
```

**注意**：`vector<int> v1(10)` 中的 10 个元素都会被**值初始化**（int 类型为 0），这不是"随机值"。

### 2.3 通过初始化列表构造（C++11）

```
vector<int> v = {1, 2, 3, 4, 5};
// 或者
vector<int> v2{10, 20, 30};
```

⚠️ **易错点**：注意区分 `()` 和 `{}`

```
vector<int> v1(5, 1);   // 5 个元素，每个值为 1 → {1, 1, 1, 1, 1}
vector<int> v2{5, 1};   // 2 个元素，值分别为 5 和 1 → {5, 1}
```

当使用 `{}` 时，编译器**优先匹配** `initializer_list` 构造函数。

### 2.4 拷贝构造

```
vector<int> v1 = {1, 2, 3};
vector<int> v2(v1);       // 深拷贝，v2 是 v1 的完整副本
vector<int> v3 = v1;      // 与上面等价
```

修改 `v2` 不会影响 `v1`，因为是**深拷贝**。

### 2.5 移动构造（C++11）

```
vector<int> v1 = {1, 2, 3, 4, 5};
vector<int> v2(std::move(v1));  // 窃取 v1 的资源

// 此时 v1 处于"有效但未指定"的状态，通常为空
cout << v1.size() << endl;  // 0
cout << v2.size() << endl;  // 5
```

移动构造的性能优势巨大——它**不拷贝元素**，只是转移内部指针，时间复杂度 O(1)。

### 2.6 用迭代器范围构造

```
vector<int> v1 = {10, 20, 30, 40, 50};

// 用 v1 的一部分构造 v2（取索引 1~3 的元素）
vector<int> v2(v1.begin() + 1, v1.begin() + 4);
// v2 = {20, 30, 40}

// 也可以从普通数组构造
int arr[] = {100, 200, 300};
vector<int> v3(arr, arr + 3);   // v3 = {100, 200, 300}

// 从其他容器构造
#include <set>
set<int> s = {5, 3, 1, 4, 2};
vector<int> v4(s.begin(), s.end());  // v4 = {1, 2, 3, 4, 5}（set 有序）
```

### 2.7 小结表格

| 构造方式   | 示例                           | 说明            |
| ---------- | ------------------------------ | --------------- |
| 默认构造   | vector<int> v;                 | 空容器          |
| n 个相同值 | vector<int> v(5, 10);          | 5个10           |
| 初始化列表 | vector<int> v{1,2,3};          | C++11           |
| 拷贝构造   | vector<int> v2(v1);            | 深拷贝          |
| 移动构造   | vector<int> v2(std::move(v1)); | 窃取资源        |
| 迭代器范围 | vector<int> v(it1, it2);       | 区间 [it1, it2) |

---

## 三、赋值操作

### 3.1 operator= 赋值运算符

```
vector<int> v1 = {1, 2, 3};
vector<int> v2;

v2 = v1;  // 深拷贝赋值
v2 = {10, 20, 30};  // 用初始化列表赋值（C++11）

// 移动赋值
vector<int> v3;
v3 = std::move(v1);  // v1 的资源被转移给 v3
```

### 3.2 assign() 成员函数

`assign()` 有三种重载形式，它会**完全替换**当前内容：

```
vector<int> v;

// 形式一：n 个相同值
v.assign(5, 100);
// v = {100, 100, 100, 100, 100}

// 形式二：迭代器范围
vector<int> src = {10, 20, 30};
v.assign(src.begin(), src.end());
// v = {10, 20, 30}

// 形式三：初始化列表（C++11）
v.assign({7, 8, 9});
// v = {7, 8, 9}
```

### 3.3 assign vs operator= 的区别

```
vector<int> v1 = {1, 2, 3, 4, 5};
vector<int> v2 = {10, 20};

// operator= 是整体赋值
v2 = v1;  // v2 变为 {1, 2, 3, 4, 5}

// assign 可以赋部分内容
vector<int> v3;
v3.assign(v1.begin(), v1.begin() + 3);  // v3 = {1, 2, 3}

// assign 可以赋 n 个相同值
v3.assign(4, 0);  // v3 = {0, 0, 0, 0}
```

**核心区别**：`assign()` 提供了更灵活的赋值方式（按范围、按数量），而 `operator=` 只能整体赋值。

### 3.4 swap() 交换

```
vector<int> v1 = {1, 2, 3};
vector<int> v2 = {10, 20};

v1.swap(v2);
// 现在 v1 = {10, 20}，v2 = {1, 2, 3}

// 也可以用 std::swap
std::swap(v1, v2);  // 再次交换回来
```

`swap()` 的效率极高——它只交换内部的三个指针（开始、结束、容量末尾），**不复制任何元素**，时间复杂度 O(1)。

---

## 四、容量和大小

这是理解 `vector` 内存管理的**核心概念**。

### 4.1 size() vs capacity() —— 最重要的区别

```
内存布局示意图：

|  已使用的元素  |  已分配但未使用的空间  |
|<---- size --->|                        |
|<------------ capacity --------------->|
```

- **`size()`**：当前实际存储的元素个数
- **`capacity()`**：当前已分配的内存能容纳的最大元素个数（不重新分配的前提下）

```
vector<int> v;
cout << "size: " << v.size() << endl;         // 0
cout << "capacity: " << v.capacity() << endl; // 0

v.push_back(1);
cout << "size: " << v.size() << endl;         // 1
cout << "capacity: " << v.capacity() << endl; // 1（或更大，取决于实现）

v.push_back(2);
cout << "size: " << v.size() << endl;         // 2
cout << "capacity: " << v.capacity() << endl; // 2（或更大）

v.push_back(3);
cout << "size: " << v.size() << endl;         // 3
cout << "capacity: " << v.capacity() << endl; // 4（扩容了！通常翻倍）
```

### 4.2 观察扩容过程

```
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v;
    int prev_cap = 0;

    for (int i = 0; i < 100; ++i) {
        v.push_back(i);
        if (v.capacity() != prev_cap) {
            cout << "size=" << v.size() 
                 << "  capacity=" << v.capacity() << endl;
            prev_cap = v.capacity();
        }
    }
    return 0;
}
```

**MSVC（Visual Studio）典型输出**（1.5 倍扩容）：

```
size=1  capacity=1
size=2  capacity=2
size=3  capacity=3
size=4  capacity=4
size=5  capacity=6
size=7  capacity=9
size=10  capacity=13
size=14  capacity=19
size=20  capacity=28
size=29  capacity=42
size=43  capacity=63
size=64  capacity=94
size=95  capacity=141
```

**GCC 典型输出**（2 倍扩容）：

```
size=1  capacity=1
size=2  capacity=2
size=3  capacity=4
size=5  capacity=8
size=9  capacity=16
size=17  capacity=32
size=33  capacity=64
size=65  capacity=128
```

> 💡 **为什么不是每次只增加一个？** 因为扩容涉及内存分配和元素拷贝，代价高昂。按倍数增长可以保证 `push_back` 的**均摊时间复杂度**为 O(1)。

### 4.3 empty()

```
vector<int> v;
if (v.empty()) {
    cout << "vector 是空的" << endl;
}

v.push_back(42);
if (!v.empty()) {
    cout << "vector 不为空, size = " << v.size() << endl;
}
```

> 推荐用 `empty()` 而不是 `size() == 0` 来判空，语义更清晰。

### 4.4 resize() —— 改变元素个数

`resize()` 会改变 `size()`，可能也会改变 `capacity()`。

```
vector<int> v = {1, 2, 3, 4, 5};

// 缩小：多余的元素被销毁
v.resize(3);
// v = {1, 2, 3}，size=3
// 注意：capacity 通常不会缩小！

// 扩大：新增元素用默认值（int 为 0）填充
v.resize(6);
// v = {1, 2, 3, 0, 0, 0}，size=6

// 扩大并指定填充值
v.resize(9, -1);
// v = {1, 2, 3, 0, 0, 0, -1, -1, -1}，size=9
```

**`resize` 的行为总结**：

| 调用            | 新 size < 旧 size | 新 size > 旧 size   |
| --------------- | ----------------- | ------------------- |
| 效果            | 截断尾部元素      | 用默认值/指定值填充 |
| size() 变化     | 变小              | 变大                |
| capacity() 变化 | 不变              | 可能增大            |

### 4.5 max_size()

```
vector<int> v;
cout << v.max_size() << endl;
// 64 位系统可能输出：4611686018427387903（约 4.6 × 10^18）
// 这是理论最大值，实际受内存限制
```

### 4.6 shrink_to_fit() —— 释放多余内存（C++11）

```
vector<int> v(1000);
cout << "capacity: " << v.capacity() << endl;  // 1000

v.resize(10);
cout << "size: " << v.size() << endl;          // 10
cout << "capacity: " << v.capacity() << endl;  // 1000（还占着大内存！）

v.shrink_to_fit();
cout << "capacity: " << v.capacity() << endl;  // 通常会接近 10（实现可选择忽略请求）
```

> ⚠️ `shrink_to_fit()` 是一个**非绑定请求**（non-binding request），标准允许实现忽略它。但主流编译器通常会响应。

---

## 五、插入和删除

### 5.1 尾部操作 —— 最常用

```
vector<int> v;

// push_back：在尾部添加元素
v.push_back(10);
v.push_back(20);
v.push_back(30);
// v = {10, 20, 30}

// pop_back：移除最后一个元素
v.pop_back();
// v = {10, 20}
```

> 💡 `push_back` 和 `pop_back` 时间复杂度均为 **O(1)**（均摊），这是 vector 最高效的操作。

### 5.2 emplace_back() —— push_back 的高效替代（C++11）

```
struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {
        cout << "构造 Point(" << x << ", " << y << ")" << endl;
    }
};

vector<Point> points;

// push_back 方式：先构造临时对象，再拷贝/移动到 vector 中
points.push_back(Point(1, 2));

// emplace_back 方式：直接在 vector 内部原地构造，避免拷贝
points.emplace_back(3, 4);  // 直接传构造函数参数
```

**区别**：

- `push_back(Point(1,2))`：构造临时 Point → 移动到 vector → 析构临时 Point
- `emplace_back(3, 4)`：直接在 vector 内存上调用 `Point(3, 4)` 构造函数

> **现代 C++ 建议**：优先使用 `emplace_back()`，尤其是对于构造开销大的对象。

### 5.3 insert() —— 在指定位置插入

```
vector<int> v = {1, 2, 3};

// 在第 2 个位置（索引 1）前插入 100
auto it = v.insert(v.begin() + 1, 100);
// v = {1, 100, 2, 3}
// it 指向新插入的 100

*it = 200;           // 修改刚插入的元素
v.erase(it + 1);     // 删除它后面的元素
// v = {1, 200, 3}

// 在开头插入 3 个 0
v.insert(v.begin(), 3, 0);
// v = {0, 0, 0, 1, 200, 3}

// 插入另一个容器的范围
vector<int> extra = {88, 99};
v.insert(v.end(), extra.begin(), extra.end());
// v = {0, 0, 0, 1, 200, 3, 88, 99}

// 插入初始化列表（C++11）
v.insert(v.begin() + 2, {11, 22});
// v = {0, 0, 11, 22, 0, 1, 200, 3, 88, 99}
```

> ⚠️ **性能警告**：在 vector 的**头部或中间**插入元素需要移动后面所有元素，时间复杂度 **O(n)**。如果需要频繁在头部插入，考虑使用 `deque`。

### 5.4 emplace() —— insert 的原地构造版本（C++11）

```
vector<pair<int, string>> v;
v.emplace_back(1, "one");
v.emplace_back(3, "three");

// 在位置 1 原地构造 pair
v.emplace(v.begin() + 1, 2, "two");
// v = {{1,"one"}, {2,"two"}, {3,"three"}}
```

### 5.5 erase() —— 删除元素

```
vector<int> v = {10, 20, 30, 40, 50};

// 删除单个元素（第 3 个，索引 2）
v.erase(v.begin() + 2);
// v = {10, 20, 40, 50}

// 删除范围 [begin+1, begin+3)，即索引 1 和 2
v.erase(v.begin() + 1, v.begin() + 3);
// v = {10, 50}
```

`erase` 返回指向被删除元素**之后**元素的迭代器，这在遍历删除时非常重要：

```
// ✅ 正确的遍历删除方式
vector<int> v = {1, 2, 3, 2, 4, 2, 5};
for (auto it = v.begin(); it != v.end(); ) {
    if (*it == 2) {
        it = v.erase(it);  // erase 返回下一个有效迭代器
    } else {
        ++it;
    }
}
// v = {1, 3, 4, 5}
```

```
// ❌ 错误的遍历删除方式（迭代器失效！）
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it == 2) {
        v.erase(it);  // erase 后 it 已失效，再 ++it 是未定义行为！
    }
}
```

### 5.6 更高效的删除——erase-remove 惯用法

如果你需要从 vector 中删除所有满足条件的元素，上面的逐个 `erase` 效率很低（O(n²)）。推荐使用 `erase-remove` 惯用法：

```
#include <algorithm>

vector<int> v = {1, 2, 3, 2, 4, 2, 5};

// 删除所有值为 2 的元素
v.erase(std::remove(v.begin(), v.end(), 2), v.end());
// v = {1, 3, 4, 5}

// C++20 更简洁：
std::erase(v, 2);  // 等价于上面的 erase-remove

// 删除所有偶数（C++20）
std::erase_if(v, [](int x) { return x % 2 == 0; });
```

**`std::remove` 的工作原理**：

```
原始：{1, 2, 3, 2, 4, 2, 5}

remove(v.begin(), v.end(), 2) 后：
{1, 3, 4, 5, ?, ?, ?}
              ↑ remove 返回的迭代器

然后 erase 从这个迭代器到 end() 的范围
最终：{1, 3, 4, 5}
```

`remove` 不真正删除元素，只是把不需要删除的元素**往前移**，返回新的逻辑尾部。

### 5.7 clear() —— 清空所有元素

```
vector<int> v = {1, 2, 3, 4, 5};
cout << "size: " << v.size() << endl;         // 5
cout << "capacity: " << v.capacity() << endl; // 5

v.clear();
cout << "size: " << v.size() << endl;         // 0
cout << "capacity: " << v.capacity() << endl; // 5（内存没释放！）
```

> 如果想彻底释放内存，用 `shrink_to_fit()` 或 swap 技巧（下文会讲）。

---

## 六、数据存取

### 6.1 operator[] 下标访问

```
vector<int> v = {10, 20, 30, 40, 50};

cout << v[0] << endl;   // 10
cout << v[2] << endl;   // 30
v[3] = 999;             // 修改第 4 个元素
cout << v[3] << endl;   // 999
```

> ⚠️ **不会检查越界**！访问超出范围是**未定义行为**。

```
cout << v[100] << endl;  // 未定义行为！可能崩溃、输出垃圾值、或看起来"正常"
```

### 6.2 at() 带边界检查的访问

```
vector<int> v = {10, 20, 30};

cout << v.at(1) << endl;  // 20

try {
    cout << v.at(100) << endl;  // 抛出 std::out_of_range 异常
} catch (const std::out_of_range& e) {
    cout << "越界错误: " << e.what() << endl;
}
```

`at()` 比 `[]` 慢一点（多了范围检查），但更**安全**。

**选择建议**：

- 性能关键的代码，且你**确定**索引合法 → 用 `[]`
- 索引可能不合法，或调试阶段 → 用 `at()`

### 6.3 front() 和 back()

```
vector<int> v = {10, 20, 30, 40, 50};

cout << v.front() << endl;  // 10（第一个元素）
cout << v.back() << endl;   // 50（最后一个元素）

// 它们返回的是引用，可以修改
v.front() = 100;
v.back() = 500;
// v = {100, 20, 30, 40, 500}
```

> ⚠️ 对空 vector 调用 `front()` 或 `back()` 是**未定义行为**。

### 6.4 data() —— 获取底层数组指针

```
vector<int> v = {1, 2, 3, 4, 5};

int* p = v.data();
cout << p[0] << endl;   // 1
cout << p[2] << endl;   // 3

// 常用于与 C 风格 API 交互
// 例如：some_c_function(v.data(), v.size());
```

### 6.5 迭代器访问

```
vector<int> v = {10, 20, 30, 40, 50};

// 正向迭代器
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << " ";
}
// 输出：10 20 30 40 50

// 反向迭代器
for (auto rit = v.rbegin(); rit != v.rend(); ++rit) {
    cout << *rit << " ";
}
// 输出：50 40 30 20 10

// const 迭代器（只读）（C++11 cbegin/cend）
for (auto cit = v.cbegin(); cit != v.cend(); ++cit) {
    // *cit = 100;  // 编译错误！不能通过 const 迭代器修改
    cout << *cit << " ";
}
```

### 6.6 范围 for 循环（C++11，最推荐）

```
vector<int> v = {10, 20, 30, 40, 50};

// 只读遍历
for (const auto& val : v) {
    cout << val << " ";
}

// 可修改遍历
for (auto& val : v) {
    val *= 2;
}
// v = {20, 40, 60, 80, 100}
```

> 💡 遍历时使用 `const auto&` 可以避免不必要的拷贝，是**最佳实践**。

---

## 七、互换容器（swap 的妙用）

### 7.1 基本交换

```
vector<int> v1 = {1, 2, 3};
vector<int> v2 = {10, 20, 30, 40, 50};

cout << "交换前：" << endl;
cout << "v1 size=" << v1.size() << " capacity=" << v1.capacity() << endl;
cout << "v2 size=" << v2.size() << " capacity=" << v2.capacity() << endl;

v1.swap(v2);

cout << "交换后：" << endl;
cout << "v1 size=" << v1.size() << " capacity=" << v1.capacity() << endl;
cout << "v2 size=" << v2.size() << " capacity=" << v2.capacity() << endl;
```

输出：

```
交换前：
v1 size=3 capacity=3
v2 size=5 capacity=5
交换后：
v1 size=5 capacity=5
v2 size=3 capacity=3
```

> `swap` 交换的是**所有内部状态**（size、capacity、元素），效率 O(1)。

### 7.2 经典用法：用 swap 收缩内存（swap trick）

在 C++11 的 `shrink_to_fit()` 出现之前，这是收缩 vector 内存的标准做法：

```
vector<int> v;
for (int i = 0; i < 10000; ++i) {
    v.push_back(i);
}
cout << "size=" << v.size() << " capacity=" << v.capacity() << endl;
// size=10000 capacity=16384（某些实现）

v.resize(10);
cout << "resize 后: size=" << v.size() << " capacity=" << v.capacity() << endl;
// size=10 capacity=16384（内存没释放！）

// 🔑 swap 技巧：用当前内容构造一个临时 vector，再与自身交换
vector<int>(v).swap(v);
cout << "swap 后: size=" << v.size() << " capacity=" << v.capacity() << endl;
// size=10 capacity=10（内存收缩了！）
```

**原理分析**：

1. `vector<int>(v)` — 用 `v` 的内容创建一个临时 vector，其 capacity 刚好等于 size（10）
2. `.swap(v)` — 将临时 vector 与 `v` 交换
3. 语句结束后，临时 vector（此时持有旧的大内存）被析构，旧内存释放

### 7.3 用 swap 彻底清空并释放内存

```
vector<int> v(10000, 42);
cout << "capacity: " << v.capacity() << endl;  // 10000

// clear() 不释放内存
v.clear();
cout << "clear 后 capacity: " << v.capacity() << endl;  // 10000

// 用 swap 彻底释放
vector<int>().swap(v);
cout << "swap 后 capacity: " << v.capacity() << endl;  // 0
```

> **现代 C++ 推荐**：直接使用 `v.clear(); v.shrink_to_fit();` 代替 swap 技巧，语义更清晰。但了解 swap 技巧对理解原理很有帮助。

---

## 八、预留空间（reserve）

### 8.1 为什么需要预留空间？

回顾之前的扩容过程——每次 capacity 不足时，vector 需要：

1. 分配新的更大内存
2. 将所有旧元素拷贝/移动到新内存
3. 销毁旧元素
4. 释放旧内存

如果你**事先知道**大概需要多少元素，用 `reserve()` 提前分配好内存，就能**完全避免扩容开销**。

### 8.2 基本用法

```
vector<int> v;
v.reserve(1000);  // 预留至少 1000 个元素的空间

cout << "size: " << v.size() << endl;         // 0（没有元素！）
cout << "capacity: " << v.capacity() << endl; // >= 1000
```

> ⚠️ **`reserve` 只改变 `capacity`，不改变 `size`！** 预留空间后，元素并没有被创建，不能用 `[]` 访问。

```
vector<int> v;
v.reserve(100);
// v[0] = 42;  // 未定义行为！size 仍为 0
v.push_back(42);  // ✅ 正确方式
```

### 8.3 reserve vs resize 对比

|               | reserve(n)         | resize(n)        |
| ------------- | ------------------ | ---------------- |
| 改变 size     | ❌ 不改变           | ✅ 改变为 n       |
| 改变 capacity | ✅ 至少为 n         | ✅ 可能增大       |
| 创建元素      | ❌ 不创建           | ✅ 创建（或销毁） |
| 可用 [] 访问  | ❌ 只能访问已有元素 | ✅ 所有 n 个元素  |
| 典型用途      | 性能优化           | 设定容器大小     |

### 8.4 性能对比实验

```
#include <iostream>
#include <vector>
#include <chrono>
using namespace std;
using namespace chrono;

int main() {
    const int N = 1000000;

    // 不预留空间
    {
        auto start = high_resolution_clock::now();
        vector<int> v;
        for (int i = 0; i < N; ++i) {
            v.push_back(i);
        }
        auto end = high_resolution_clock::now();
        auto dur = duration_cast<microseconds>(end - start).count();
        cout << "不预留: " << dur << " 微秒, "
             << "扩容后 capacity=" << v.capacity() << endl;
    }

    // 预留空间
    {
        auto start = high_resolution_clock::now();
        vector<int> v;
        v.reserve(N);  // 提前预留
        for (int i = 0; i < N; ++i) {
            v.push_back(i);
        }
        auto end = high_resolution_clock::now();
        auto dur = duration_cast<microseconds>(end - start).count();
        cout << "有预留: " << dur << " 微秒, "
             << "capacity=" << v.capacity() << endl;
    }

    return 0;
}
```

**典型输出**：

```
不预留: 12345 微秒, 扩容后 capacity=1048576
有预留: 4567 微秒, capacity=1000000
```

> 预留空间后，性能提升显著：没有扩容的开销，而且内存恰好够用，没有浪费。

### 8.5 统计扩容次数

```
#include <iostream>
#include <vector>
using namespace std;

int main() {
    const int N = 100000;
    int resize_count = 0;

    // 不预留
    {
        vector<int> v;
        int prev_cap = 0;
        for (int i = 0; i < N; ++i) {
            v.push_back(i);
            if (v.capacity() != prev_cap) {
                ++resize_count;
                prev_cap = v.capacity();
            }
        }
        cout << "不预留: 扩容 " << resize_count << " 次" << endl;
    }

    // 预留
    {
        resize_count = 0;
        vector<int> v;
        v.reserve(N);
        int prev_cap = v.capacity();
        for (int i = 0; i < N; ++i) {
            v.push_back(i);
            if (v.capacity() != prev_cap) {
                ++resize_count;
                prev_cap = v.capacity();
            }
        }
        cout << "有预留: 扩容 " << resize_count << " 次" << endl;
    }

    return 0;
}
```

输出：

```
不预留: 扩容 30 次
有预留: 扩容 0 次
```

---

## 九、迭代器失效问题（进阶重点⚠️）

这是使用 vector 时最容易出错的地方，面试也经常问。

### 9.1 什么是迭代器失效？

当 vector 执行某些操作后，之前获取的迭代器、指针、引用可能变得**无效**（指向已释放或错误的内存）。

### 9.2 哪些操作导致失效？

| 操作                     | 迭代器/指针/引用失效范围                     |
| ------------------------ | -------------------------------------------- |
| push_back / emplace_back | 如果触发扩容：全部失效；否则仅 end() 失效    |
| insert / emplace         | 如果触发扩容：全部失效；否则插入点及之后失效 |
| erase                    | 删除点及之后失效                             |
| resize                   | 如果扩大且触发扩容：全部失效                 |
| reserve                  | 如果改变了 capacity：全部失效                |
| clear                    | 全部失效                                     |
| swap                     | 所有迭代器仍有效，但指向的容器变了           |

### 9.3 经典错误示例

```
// ❌ 危险！迭代器可能失效
vector<int> v = {1, 2, 3};
auto it = v.begin();
v.push_back(4);  // 可能触发扩容，it 失效！
cout << *it << endl;  // 未定义行为！

// ✅ 正确做法
v.push_back(4);
it = v.begin();  // 重新获取迭代器
cout << *it << endl;
```

```
// ❌ 在遍历中 push_back
vector<int> v = {1, 2, 3};
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it == 2) {
        v.push_back(20);  // 扩容后 it 和 end() 全部失效！
    }
}
```

---

## 十、vector<bool>——特殊的存在

`vector<bool>` 是标准库的一个**特化版本**，它将每个 `bool` 压缩为 1 bit 存储，节省内存但带来很多问题：

```
vector<bool> vb = {true, false, true};

// ⚠️ operator[] 返回的不是 bool&，而是一个代理对象
auto val = vb[0];  // val 的类型不是 bool，而是 vector<bool>::reference

// 这会导致一些反直觉的行为
bool* p = &vb[0];  // ❌ 编译错误！不能取地址
```

> **建议**：如果你不需要 bit 压缩，可用 `vector<unsigned char>`（或 `vector<std::uint8_t>`）代替 `vector<bool>`。如果确实需要按位存储，使用 `std::bitset`（编译期固定大小）或专门的动态位图结构。

---

## 十一、实用技巧汇总

### 11.1 二维 vector

```
// 创建 3 行 4 列的二维 vector，初始值为 0
int rows = 3, cols = 4;
vector<vector<int>> matrix(rows, vector<int>(cols, 0));

// 访问
matrix[1][2] = 42;

// 遍历
for (const auto& row : matrix) {
    for (int val : row) {
        cout << val << " ";
    }
    cout << endl;
}
```

### 11.2 排序

```
#include <algorithm>
#include <cstdlib>

vector<int> v = {5, 2, 8, 1, 9, 3};

// 升序排序
sort(v.begin(), v.end());
// v = {1, 2, 3, 5, 8, 9}

// 降序排序
sort(v.begin(), v.end(), greater<int>());
// v = {9, 8, 5, 3, 2, 1}

// 自定义排序
sort(v.begin(), v.end(), [](int a, int b) {
    return std::abs(a - 5) < std::abs(b - 5);  // 按与 5 的距离排序
});
```

### 11.3 去重

```
vector<int> v = {3, 1, 2, 1, 3, 2, 4};

sort(v.begin(), v.end());           // 先排序：{1, 1, 2, 2, 3, 3, 4}
auto last = unique(v.begin(), v.end());  // 去重：{1, 2, 3, 4, ?, ?, ?}
v.erase(last, v.end());            // 删除尾部无效元素
// v = {1, 2, 3, 4}
```

### 11.4 查找

```
vector<int> v = {10, 20, 30, 40, 50};

// 线性查找
auto it = find(v.begin(), v.end(), 30);
if (it != v.end()) {
    cout << "找到了，索引: " << distance(v.begin(), it) << endl;  // 索引: 2
}

// 二分查找（需要已排序）
sort(v.begin(), v.end());
bool found = binary_search(v.begin(), v.end(), 30);  // true

// lower_bound：第一个 >= 目标值的位置
auto lb = lower_bound(v.begin(), v.end(), 25);
cout << *lb << endl;  // 30
```

### 11.5 作为函数参数

```
// 以下写法均可直接编译，重点是参数形式的差异

// ❌ 值传递：拷贝整个 vector，开销大
void func1(vector<int> v) {}

// ✅ 常引用：只读，不拷贝
void func2(const vector<int>& v) {}

// ✅ 引用：可修改，不拷贝
void func3(vector<int>& v) {}

// ✅ 移动语义：转移所有权
void func4(vector<int>&& v) {}
```

> **黄金法则**：只读传 `const vector<T>&`，需修改传 `vector<T>&`，不需要原来的了用 `std::move`。

---

## 十二、总结：vector 常用操作速查表

| 操作          | 函数                         | 时间复杂度 |
| ------------- | ---------------------------- | ---------- |
| 尾部添加      | push_back() / emplace_back() | O(1) 均摊  |
| 尾部删除      | pop_back()                   | O(1)       |
| 随机访问      | operator[] / at()            | O(1)       |
| 头部/中间插入 | insert()                     | O(n)       |
| 头部/中间删除 | erase()                      | O(n)       |
| 获取大小      | size()                       | O(1)       |
| 获取容量      | capacity()                   | O(1)       |
| 判空          | empty()                      | O(1)       |
| 清空          | clear()                      | O(n)       |
| 预留空间      | reserve()                    | O(n) 最坏  |
| 调整大小      | resize()                     | O(n) 最坏  |
| 交换          | swap()                       | O(1)       |
| 首/尾元素     | front() / back()             | O(1)       |
| 底层指针      | data()                       | O(1)       |

---

## 十三、何时选择 vector？

| 场景                    | 推荐容器           |
| ----------------------- | ------------------ |
| 需要随机访问 + 尾部增删 | ✅ vector           |
| 需要频繁头部插入/删除   | ❌ 用 deque         |
| 需要频繁中间插入/删除   | ❌ 用 list          |
| 元素不重复 + 快速查找   | ❌ 用 unordered_set |
| 键值对映射              | ❌ 用 unordered_map |
| 大小固定不变            | ❌ 用 std::array    |

> **经验法则**：如果不确定用什么容器，**先用 vector**。由于其连续内存布局带来的缓存友好性，vector 在大多数场景下的实际性能都优于理论上更合适的容器。

---

以上就是 `vector` 的完整教程，涵盖了从基础构造到高级用法的各个方面。建议你按照章节顺序逐一练习，特别是**迭代器失效**和**reserve/resize 的区别**这两个容易出错的点，多写代码加深理解。如果哪个部分需要进一步展开或有疑问，随时提问！
