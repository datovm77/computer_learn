# C++ `set` 容器完全教程

---

## 目录

- [第一章 set容器 — 构造和赋值](#第一章-set容器--构造和赋值)
- [第二章 set容器 — 大小和交换](#第二章-set容器--大小和交换)
- [第三章 set容器 — 插入和删除](#第三章-set容器--插入和删除)
- [第四章 set容器 — 查找和统计](#第四章-set容器--查找和统计)
- [第五章 pair 对组的创建与使用](#第五章-pair-对组的创建与使用)
- [第六章 set和multiset的区别](#第六章-set和multiset的区别)
- [第七章 综合实战与进阶](#第七章-综合实战与进阶)

---

## 第一章 set容器 — 构造和赋值

### 1.1 什么是 set？

在学习 `set` 之前，我们先用生活中的例子来理解它：

> 想象你有一个**签到表**，每个人只能签到一次，而且名字会自动按字母顺序排列。这就是 `set`。

`set` 是 C++ STL（标准模板库）中的一种**关联式容器**，它具有以下核心特性：

| 特性                   | 说明                                                       |
| ---------------------- | ---------------------------------------------------------- |
| **元素唯一**           | 不允许有重复的元素，插入重复值会被自动忽略                 |
| **自动排序**           | 插入元素后，容器内部会自动按照排序规则排列（默认升序）     |
| **底层结构**           | 基于**红黑树**（一种自平衡二叉搜索树）实现                 |
| **不可直接修改元素值** | 元素一旦插入，不能通过迭代器直接修改（因为会破坏排序结构） |

### 1.2 使用 set 前的准备

要使用 `set`，需要包含头文件：

```cpp name=set_header.cpp
#include <set>      // 必须包含此头文件
#include <iostream>
using namespace std;
```

### 1.3 set 的构造方式

`set` 提供了多种构造方式，下面逐一讲解：

#### 方式一：默认构造（最常用）

创建一个空的 set 容器。

```cpp name=set_construct_default.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    // 默认构造：创建一个空的 set，元素类型为 int
    set<int> s1;

    cout << "s1 的大小: " << s1.size() << endl;  // 输出: 0

    return 0;
}
```

**说明**：`set<int>` 表示创建一个存储 `int` 类型元素的集合，当然你也可以存储其他类型：

```cpp name=set_types.cpp
set<int> s1;           // 存储整数
set<float> s2;         // 存储浮点数
set<double> s3;        // 存储双精度浮点数
set<char> s4;          // 存储字符
set<string> s5;        // 存储字符串（需要 #include <string>）
```

#### 方式二：初始化列表构造（C++11 及以上）

使用花括号 `{}` 直接在创建时指定元素。

```cpp name=set_construct_initlist.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    // 使用初始化列表构造
    set<int> s1 = {30, 10, 50, 20, 40};

    // 遍历输出 —— 注意：输出是自动排好序的！
    for (int val : s1) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: 10 20 30 40 50（自动升序排列）

    // 尝试插入重复值
    set<int> s2 = {10, 20, 10, 30, 20, 40};
    for (int val : s2) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: 10 20 30 40（重复值被自动去掉了！）

    return 0;
}
```

> 🔑 **核心要点**：即使你传入重复的值，`set` 也只会保留一份，并且自动排序。

#### 方式三：拷贝构造

用一个已有的 `set` 来创建一个新的 `set`，内容完全相同。

```cpp name=set_construct_copy.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s1 = {10, 20, 30, 40, 50};

    // 拷贝构造：s2 是 s1 的一份完整拷贝
    set<int> s2(s1);

    cout << "s1: ";
    for (int val : s1) cout << val << " ";
    cout << endl;  // 输出: 10 20 30 40 50

    cout << "s2: ";
    for (int val : s2) cout << val << " ";
    cout << endl;  // 输出: 10 20 30 40 50

    // s1 和 s2 是独立的，修改一个不影响另一个
    s1.insert(60);
    cout << "s1 插入60后: ";
    for (int val : s1) cout << val << " ";
    cout << endl;  // 输出: 10 20 30 40 50 60

    cout << "s2 不受影响: ";
    for (int val : s2) cout << val << " ";
    cout << endl;  // 输出: 10 20 30 40 50

    return 0;
}
```

#### 方式四：迭代器区间构造

用另一个容器的某个**区间**来构造 `set`。

```cpp name=set_construct_range.cpp
#include <iostream>
#include <set>
#include <vector>
using namespace std;

int main() {
    // 可以从 vector 的区间来构造 set
    vector<int> vec = {50, 30, 10, 40, 20, 30, 10};

    // 使用 vec 的全部区间 [begin, end) 来构造 set
    set<int> s1(vec.begin(), vec.end());

    cout << "s1: ";
    for (int val : s1) cout << val << " ";
    cout << endl;
    // 输出: 10 20 30 40 50（去重 + 排序）

    // 也可以从另一个 set 的部分区间构造
    set<int> s2 = {10, 20, 30, 40, 50, 60, 70};
    auto it_begin = s2.find(20);   // 指向 20
    auto it_end   = s2.find(60);   // 指向 60

    // 区间 [20, 60)，即包含 20, 30, 40, 50，不包含 60
    set<int> s3(it_begin, it_end);

    cout << "s3: ";
    for (int val : s3) cout << val << " ";
    cout << endl;
    // 输出: 20 30 40 50

    return 0;
}
```

### 1.4 set 的赋值操作

#### 方式一：`=` 赋值运算符（最常用）

```cpp name=set_assign_operator.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s1 = {10, 20, 30};
    set<int> s2;

    // 使用 = 赋值
    s2 = s1;

    cout << "s1: ";
    for (int val : s1) cout << val << " ";
    cout << endl;  // 输出: 10 20 30

    cout << "s2: ";
    for (int val : s2) cout << val << " ";
    cout << endl;  // 输出: 10 20 30

    // 也可以直接用初始化列表赋值
    set<int> s3;
    s3 = {100, 200, 300};

    cout << "s3: ";
    for (int val : s3) cout << val << " ";
    cout << endl;  // 输出: 100 200 300

    return 0;
}
```

### 1.5 构造和赋值总结

| 构造/赋值方式  | 语法                    | 说明                  |
| -------------- | ----------------------- | --------------------- |
| 默认构造       | `set<T> s;`             | 创建空集合            |
| 初始化列表构造 | `set<T> s = {a, b, c};` | 用花括号直接初始化    |
| 拷贝构造       | `set<T> s2(s1);`        | 拷贝另一个 set        |
| 区间构造       | `set<T> s(beg, end);`   | 用迭代器区间构造      |
| `=` 赋值       | `s2 = s1;`              | 将 s1 的内容赋值给 s2 |

---

## 第二章 set容器 — 大小和交换

### 2.1 获取 set 的大小

#### `size()` — 获取元素个数

```cpp name=set_size.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50};

    cout << "s 的大小: " << s.size() << endl;  // 输出: 5

    // 插入重复值不会增加大小
    s.insert(30);
    cout << "插入重复值30后的大小: " << s.size() << endl;  // 输出: 还是 5

    // 插入新值才会增加大小
    s.insert(60);
    cout << "插入新值60后的大小: " << s.size() << endl;  // 输出: 6

    return 0;
}
```

#### `empty()` — 判断是否为空

```cpp name=set_empty.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s1;
    set<int> s2 = {10, 20};

    if (s1.empty()) {
        cout << "s1 是空的" << endl;   // ✅ 会输出
    }

    if (s2.empty()) {
        cout << "s2 是空的" << endl;
    } else {
        cout << "s2 不是空的，大小为: " << s2.size() << endl;  // ✅ 输出: 大小为 2
    }

    return 0;
}
```

> 💡 **注意**：`set` **没有** `resize()` 和 `capacity()` 函数。因为 `set` 基于红黑树实现，元素个数完全取决于实际插入了多少不重复的元素，不存在"预分配空间"的概念。

### 2.2 交换两个 set — `swap()`

`swap()` 可以交换两个 `set` 容器的所有内容，且效率极高（内部只是交换指针，时间复杂度 O(1)）。

```cpp name=set_swap.cpp
#include <iostream>
#include <set>
using namespace std;

// 封装一个打印函数，方便复用
void printSet(const set<int>& s, const string& name) {
    cout << name << ": ";
    for (int val : s) {
        cout << val << " ";
    }
    cout << " (大小: " << s.size() << ")" << endl;
}

int main() {
    set<int> s1 = {10, 20, 30};
    set<int> s2 = {100, 200, 300, 400, 500};

    cout << "===== 交换前 =====" << endl;
    printSet(s1, "s1");  // s1: 10 20 30  (大小: 3)
    printSet(s2, "s2");  // s2: 100 200 300 400 500  (大小: 5)

    // 交换 s1 和 s2
    s1.swap(s2);

    cout << "===== 交换后 =====" << endl;
    printSet(s1, "s1");  // s1: 100 200 300 400 500  (大小: 5)
    printSet(s2, "s2");  // s2: 10 20 30  (大小: 3)

    return 0;
}
```

### 2.3 大小和交换总结

| 函数          | 说明                 | 返回值类型 |
| ------------- | -------------------- | ---------- |
| `s.size()`    | 返回容器中元素的个数 | `size_t`   |
| `s.empty()`   | 判断容器是否为空     | `bool`     |
| `s1.swap(s2)` | 交换两个容器的内容   | `void`     |

> ⚠️ **set 不支持的大小操作**：`resize()`、`capacity()`、`reserve()`

---

## 第三章 set容器 — 插入和删除

### 3.1 插入操作 — `insert()`

#### 基本插入

```cpp name=set_insert_basic.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s;

    // 逐个插入元素
    s.insert(30);
    s.insert(10);
    s.insert(50);
    s.insert(20);
    s.insert(40);

    // 遍历：自动排序
    for (int val : s) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: 10 20 30 40 50

    return 0;
}
```

#### 🔑 insert 的返回值（重要！）

`set::insert()` 的返回值是一个 **`pair<iterator, bool>`** 类型，这非常关键：

- `pair.first`  → 指向插入位置（或已存在元素位置）的**迭代器**
- `pair.second` → 一个 `bool` 值，`true` 表示插入成功，`false` 表示元素已存在、插入失败

```cpp name=set_insert_return.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s;

    // 第一次插入 10 —— 成功
    pair<set<int>::iterator, bool> result1 = s.insert(10);
    if (result1.second) {
        cout << "插入 10 成功！" << endl;        // ✅ 输出
    } else {
        cout << "插入 10 失败，已存在！" << endl;
    }

    // 第二次插入 10 —— 失败（已存在）
    pair<set<int>::iterator, bool> result2 = s.insert(10);
    if (result2.second) {
        cout << "插入 10 成功！" << endl;
    } else {
        cout << "插入 10 失败，已存在！" << endl;  // ✅ 输出
    }

    // 使用 auto 简化写法（推荐）
    auto result3 = s.insert(20);
    cout << "插入 20 " << (result3.second ? "成功" : "失败") << endl;
    cout << "插入位置的值: " << *(result3.first) << endl;  // 输出: 20

    return 0;
}
```

#### 区间插入

```cpp name=set_insert_range.cpp
#include <iostream>
#include <set>
#include <vector>
using namespace std;

int main() {
    set<int> s = {10, 20, 30};
    vector<int> vec = {40, 50, 20, 60};  // 注意: 20 已存在于 s 中

    // 将 vec 中的元素批量插入 s
    s.insert(vec.begin(), vec.end());

    for (int val : s) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: 10 20 30 40 50 60（20 不会重复插入）

    // 也可以用初始化列表批量插入
    s.insert({70, 80, 90});
    for (int val : s) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: 10 20 30 40 50 60 70 80 90

    return 0;
}
```

### 3.2 删除操作

`set` 提供了多种删除方式：

#### 方式一：`erase(value)` — 按值删除

```cpp name=set_erase_value.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50};

    // 删除值为 30 的元素
    size_t count = s.erase(30);
    // 返回值: 删除的元素个数（set中为 0 或 1）

    cout << "删除了 " << count << " 个元素" << endl;  // 输出: 1

    // 尝试删除不存在的值
    count = s.erase(100);
    cout << "删除了 " << count << " 个元素" << endl;  // 输出: 0（没找到，不报错）

    for (int val : s) cout << val << " ";
    cout << endl;
    // 输出: 10 20 40 50

    return 0;
}
```

#### 方式二：`erase(iterator)` — 按迭代器删除

```cpp name=set_erase_iterator.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50};

    // 删除第一个元素
    s.erase(s.begin());
    for (int val : s) cout << val << " ";
    cout << endl;
    // 输出: 20 30 40 50

    // 删除 "最后一个" 元素
    // 注意: s.end() 指向的是末尾的"下一个"位置，所以需要 --
    auto it = s.end();
    --it;           // 现在 it 指向最后一个元素
    s.erase(it);
    for (int val : s) cout << val << " ";
    cout << endl;
    // 输出: 20 30 40

    return 0;
}
```

#### 方式三：`erase(beg, end)` — 按区间删除

```cpp name=set_erase_range.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50, 60, 70};

    // 删除从第二个元素开始，到倒数第二个元素（不包含）的范围
    auto beg = s.begin();
    ++beg;               // 指向 20
    auto end = s.end();
    --end;               // 指向 70

    s.erase(beg, end);   // 删除 [20, 70)，即删除 20,30,40,50,60

    for (int val : s) cout << val << " ";
    cout << endl;
    // 输出: 10 70

    return 0;
}
```

#### 方式四：`clear()` — 清空所有元素

```cpp name=set_clear.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50};

    cout << "清空前大小: " << s.size() << endl;  // 输出: 5

    s.clear();

    cout << "清空后大小: " << s.size() << endl;   // 输出: 0
    cout << "是否为空: " << (s.empty() ? "是" : "否") << endl;  // 输出: 是

    return 0;
}
```

### 3.3 `emplace()` — 原地构造插入（C++11）

`emplace()` 和 `insert()` 类似，但对于复杂对象可以避免额外的拷贝或移动操作，效率更高。

```cpp name=set_emplace.cpp
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main() {
    set<string> s;

    // insert 和 emplace 对于简单类型差别不大
    s.insert("Hello");
    s.emplace("World");

    for (const string& val : s) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: Hello World（字符串按字典序排列）

    // emplace 的返回值与 insert 相同：pair<iterator, bool>
    auto result = s.emplace("Hello");
    cout << "重复 emplace: " << (result.second ? "成功" : "失败") << endl;
    // 输出: 失败

    return 0;
}
```

### 3.4 插入和删除总结

| 操作           | 语法                  | 说明                           |
| -------------- | --------------------- | ------------------------------ |
| 插入单个值     | `s.insert(value)`     | 返回 `pair<iterator, bool>`    |
| 区间插入       | `s.insert(beg, end)`  | 将 `[beg, end)` 区间的元素插入 |
| 初始化列表插入 | `s.insert({a, b, c})` | 批量插入                       |
| 原地构造插入   | `s.emplace(args...)`  | 高效插入，避免拷贝             |
| 按值删除       | `s.erase(value)`      | 返回删除的元素个数（0或1）     |
| 按迭代器删除   | `s.erase(it)`         | 删除迭代器指向的元素           |
| 按区间删除     | `s.erase(beg, end)`   | 删除 `[beg, end)` 区间         |
| 清空           | `s.clear()`           | 删除所有元素                   |

---

## 第四章 set容器 — 查找和统计

### 4.1 查找 — `find()`

`find()` 用于查找某个值是否存在于 `set` 中。

- **找到** → 返回指向该元素的迭代器
- **未找到** → 返回 `s.end()`

```cpp name=set_find.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50};

    // 查找 30
    set<int>::iterator it = s.find(30);

    if (it != s.end()) {
        cout << "找到了: " << *it << endl;      // ✅ 输出: 找到了: 30
    } else {
        cout << "未找到" << endl;
    }

    // 查找 99（不存在）
    auto it2 = s.find(99);  // 推荐使用 auto 简化

    if (it2 != s.end()) {
        cout << "找到了: " << *it2 << endl;
    } else {
        cout << "未找到 99" << endl;            // ✅ 输出
    }

    return 0;
}
```

> 💡 **时间复杂度**：`find()` 的时间复杂度为 **O(log n)**，因为底层是红黑树。这比 `vector` 的线性查找 O(n) 要快得多！

### 4.2 统计 — `count()`

`count()` 统计某个值在 `set` 中出现的次数。

- 对于 `set`：结果只可能是 **0**（不存在）或 **1**（存在）
- 对于 `multiset`：结果可以是任意非负整数

```cpp name=set_count.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50};

    // 统计 30 出现的次数
    cout << "30 出现次数: " << s.count(30) << endl;  // 输出: 1

    // 统计 99 出现的次数
    cout << "99 出现次数: " << s.count(99) << endl;  // 输出: 0

    // 因此 count() 也可以用来判断元素是否存在
    if (s.count(20)) {
        cout << "20 存在于集合中" << endl;   // ✅ 输出
    }

    if (!s.count(100)) {
        cout << "100 不在集合中" << endl;    // ✅ 输出
    }

    return 0;
}
```

### 4.3 进阶查找 — `lower_bound()` / `upper_bound()` / `equal_range()`

这三个函数在需要查找**范围**时非常有用。

#### `lower_bound(value)` — 返回 **≥ value** 的第一个元素的迭代器

#### （查找数值 `val` 出现的**第一个位置**。）

#### `upper_bound(value)` — 返回 **> value** 的第一个元素的迭代器

#### （查找数值 `val` 出现的**最后一个位置的下一个位置**。）



```cpp name=set_bound.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50, 60};

    // lower_bound(25): 第一个 >= 25 的元素
    auto it1 = s.lower_bound(25);
    if (it1 != s.end()) {
        cout << "lower_bound(25) = " << *it1 << endl;  // 输出: 30
    }

    // lower_bound(30): 第一个 >= 30 的元素
    auto it2 = s.lower_bound(30);
    if (it2 != s.end()) {
        cout << "lower_bound(30) = " << *it2 << endl;  // 输出: 30（包含等于）
    }

    // upper_bound(30): 第一个 > 30 的元素
    auto it3 = s.upper_bound(30);
    if (it3 != s.end()) {
        cout << "upper_bound(30) = " << *it3 << endl;  // 输出: 40（不包含等于）
    }

    // 利用 lower_bound 和 upper_bound 获取一个范围
    // 例如：获取 [20, 50] 之间的所有元素
    auto low  = s.lower_bound(20);  // 指向 20（>= 20 的第一个）
    auto high = s.upper_bound(50);  // 指向 60（> 50 的第一个）

    cout << "[20, 50] 范围内的元素: ";
    for (auto it = low; it != high; ++it) {
        cout << *it << " ";
    }
    cout << endl;
    // 输出: 20 30 40 50

    return 0;
}
```

#### `equal_range(value)` — 同时返回 lower_bound 和 upper_bound

返回一个 `pair<iterator, iterator>`，表示等于 value 的元素范围 `[lower_bound, upper_bound)`。

```cpp name=set_equal_range.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50};

    // 对于 set，equal_range 返回的范围最多包含一个元素
    auto range = s.equal_range(30);

    cout << "lower_bound: " << *(range.first) << endl;   // 输出: 30
    cout << "upper_bound: " << *(range.second) << endl;   // 输出: 40

    // 遍历范围
    cout << "equal_range(30) 中的元素: ";
    for (auto it = range.first; it != range.second; ++it) {
        cout << *it << " ";
    }
    cout << endl;
    // 输出: 30（只有一个元素，因为 set 不允许重复）

    return 0;
}
```

### 4.4 查找和统计总结

| 函数                 | 说明                              | 返回值                       | 时间复杂度 |
| -------------------- | --------------------------------- | ---------------------------- | ---------- |
| `find(value)`        | 查找某个值                        | 迭代器（未找到返回 `end()`） | O(log n)   |
| `count(value)`       | 统计某个值的个数                  | `size_t`（0 或 1）           | O(log n)   |
| `lower_bound(value)` | 第一个 ≥ value 的迭代器           | 迭代器                       | O(log n)   |
| `upper_bound(value)` | 第一个 > value 的迭代器           | 迭代器                       | O(log n)   |
| `equal_range(value)` | 返回 `[lower_bound, upper_bound)` | `pair<iterator, iterator>`   | O(log n)   |

---

## 第五章 pair 对组的创建与使用

在前面的章节中，我们已经多次遇到了 `pair`：`insert()` 的返回值是 `pair<iterator, bool>`，`equal_range()` 返回 `pair<iterator, iterator>`。本章专门讲解 `pair` 的创建和使用方式。

### 5.1 什么是 pair？

`pair` 是 C++ STL 提供的一个**模板类**，用于将**两个值**组合成一个单元。它定义在 `<utility>` 头文件中（`<set>`、`<map>` 等头文件会自动包含它）。

```cpp
// pair 的基本结构
template <class T1, class T2>
struct pair {
    T1 first;    // 第一个值
    T2 second;   // 第二个值
};
```

你可以把 `pair` 理解为一个**只有两个成员的简单结构体**，通过 `.first` 和 `.second` 来访问。

### 5.2 pair 的创建方式

#### 方式一：默认构造

```cpp name=pair_default.cpp
#include <iostream>
#include <utility>  // pair 所在头文件
using namespace std;

int main() {
    // 默认构造：两个值都会被默认初始化
    pair<string, int> p1;
    // string 默认为空串 ""，int 默认为 0

    cout << "p1: (" << p1.first << ", " << p1.second << ")" << endl;
    // 输出: p1: (, 0)

    return 0;
}
```

#### 方式二：有参构造（最常用）

```cpp name=pair_constructor.cpp
#include <iostream>
using namespace std;

int main() {
    // 直接传入两个值来构造
    pair<string, int> p1("张三", 20);

    cout << "姓名: " << p1.first << endl;   // 输出: 张三
    cout << "年龄: " << p1.second << endl;  // 输出: 20

    return 0;
}
```

#### 方式三：使用 `make_pair()` 函数（推荐）

`make_pair()` 可以自动推导类型，写起来更简洁。

```cpp name=pair_make_pair.cpp
#include <iostream>
using namespace std;

int main() {
    // make_pair 会自动推导模板参数类型
    auto p1 = make_pair("李四", 25);

    cout << "姓名: " << p1.first << endl;   // 输出: 李四
    cout << "年龄: " << p1.second << endl;  // 输出: 25

    // 也可以显式指定类型
    pair<string, int> p2 = make_pair("王五", 30);
    cout << "姓名: " << p2.first << endl;   // 输出: 王五
    cout << "年龄: " << p2.second << endl;  // 输出: 30

    return 0;
}
```

#### 方式四：花括号初始化（C++11）

```cpp name=pair_brace_init.cpp
#include <iostream>
using namespace std;

int main() {
    // C++11 花括号初始化
    pair<string, int> p1 = {"赵六", 22};

    cout << "姓名: " << p1.first << endl;   // 输出: 赵六
    cout << "年龄: " << p1.second << endl;  // 输出: 22

    return 0;
}
```

#### 方式五：拷贝构造

```cpp name=pair_copy.cpp
#include <iostream>
using namespace std;

int main() {
    pair<string, int> p1("孙七", 18);

    // 拷贝构造
    pair<string, int> p2(p1);
    // 或者：pair<string, int> p2 = p1;

    cout << "p2: (" << p2.first << ", " << p2.second << ")" << endl;
    // 输出: p2: (孙七, 18)

    return 0;
}
```

### 5.3 pair 在 set 中的实际应用

```cpp name=pair_with_set.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s;

    // insert 返回 pair<iterator, bool>
    pair<set<int>::iterator, bool> ret;

    ret = s.insert(10);
    if (ret.second) {
        cout << "插入 " << *(ret.first) << " 成功" << endl;  // ✅
    }

    ret = s.insert(10);
    if (!ret.second) {
        cout << "插入 10 失败，元素已存在" << endl;  // ✅
    }

    // 也可以用 auto 简化
    auto [it, success] = s.insert(20);  // C++17 结构化绑定
    cout << "插入 20 " << (success ? "成功" : "失败") << endl;  // 成功

    return 0;
}
```

### 5.4 pair 创建方式总结

| 创建方式       | 语法                             | 说明                    |
| -------------- | -------------------------------- | ----------------------- |
| 默认构造       | `pair<T1, T2> p;`               | 值被默认初始化          |
| 有参构造       | `pair<T1, T2> p(v1, v2);`       | 直接传入两个值          |
| `make_pair()`  | `auto p = make_pair(v1, v2);`   | 自动推导类型，推荐使用  |
| 花括号初始化   | `pair<T1, T2> p = {v1, v2};`    | C++11，简洁             |
| 拷贝构造       | `pair<T1, T2> p2(p1);`          | 复制已有 pair           |
| 结构化绑定     | `auto [a, b] = make_pair(v1, v2);` | C++17，直接拆解为两个变量 |

> 🔑 **记住**：`pair` 的两个成员通过 `.first` 和 `.second` 访问，这在使用 `set::insert()` 和 `equal_range()` 的返回值时非常关键。

---

## 第六章 set 和 multiset 的区别

### 6.1 核心区别一览

| 特性                | `set`                  | `multiset`                       |
| ------------------- | ---------------------- | -------------------------------- |
| 头文件              | `#include <set>`       | `#include <set>`（同一个头文件） |
| 元素唯一性          | **不允许**重复元素     | **允许**重复元素                 |
| `insert()` 返回值   | `pair<iterator, bool>` | `iterator`（只返回迭代器）       |
| `count(value)` 结果 | 0 或 1                 | 0 到 n（任意非负整数）           |
| `erase(value)` 效果 | 最多删除 1 个          | 删除**所有**等于 value 的元素    |
| 自动排序            | ✅ 是                   | ✅ 是                             |
| 底层结构            | 红黑树                 | 红黑树                           |

### 6.2 详细对比演示

#### 对比1：插入行为的区别

```cpp name=set_vs_multiset_insert.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    // ========== set：不允许重复 ==========
    cout << "===== set =====" << endl;
    set<int> s;

    auto ret1 = s.insert(10);
    cout << "第1次插入10: " << (ret1.second ? "成功" : "失败")
         << ", 值=" << *(ret1.first) << endl;
    // 输出: 第1次插入10: 成功, 值=10

    auto ret2 = s.insert(10);
    cout << "第2次插入10: " << (ret2.second ? "成功" : "失败")
         << ", 值=" << *(ret2.first) << endl;
    // 输出: 第2次插入10: 失败, 值=10

    cout << "set 大小: " << s.size() << endl;  // 输出: 1

    // ========== multiset：允许重复 ==========
    cout << "\n===== multiset =====" << endl;
    multiset<int> ms;

    // multiset 的 insert 返回值只是一个 iterator（没有 bool）
    auto it1 = ms.insert(10);
    cout << "第1次插入10: 值=" << *it1 << endl;
    // 输出: 第1次插入10: 值=10

    auto it2 = ms.insert(10);
    cout << "第2次插入10: 值=" << *it2 << endl;
    // 输出: 第2次插入10: 值=10（再次插入成功！）

    auto it3 = ms.insert(10);
    cout << "第3次插入10: 值=" << *it3 << endl;
    // 输出: 第3次插入10: 值=10（第三次也成功！）

    cout << "multiset 大小: " << ms.size() << endl;  // 输出: 3

    // 遍历 multiset
    cout << "multiset 内容: ";
    for (int val : ms) cout << val << " ";
    cout << endl;
    // 输出: 10 10 10

    return 0;
}
```

> 🔑 **关键区别**：
> - `set::insert()` 返回 `pair<iterator, bool>` —— `bool` 告诉你是否成功
> - `multiset::insert()` 返回 `iterator` —— 永远成功，无需 `bool`

#### 对比2：count() 的区别

```cpp name=set_vs_multiset_count.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    // set 中 count 只能是 0 或 1
    set<int> s = {10, 20, 30};
    cout << "set 中 10 的个数: " << s.count(10) << endl;   // 输出: 1
    cout << "set 中 99 的个数: " << s.count(99) << endl;   // 输出: 0

    // multiset 中 count 可以是任意值
    multiset<int> ms = {10, 10, 10, 20, 20, 30};
    cout << "multiset 中 10 的个数: " << ms.count(10) << endl;  // 输出: 3
    cout << "multiset 中 20 的个数: " << ms.count(20) << endl;  // 输出: 2
    cout << "multiset 中 30 的个数: " << ms.count(30) << endl;  // 输出: 1
    cout << "multiset 中 99 的个数: " << ms.count(99) << endl;  // 输出: 0

    return 0;
}
```

#### 对比3：erase() 的区别

```cpp name=set_vs_multiset_erase.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    multiset<int> ms = {10, 10, 10, 20, 20, 30};

    cout << "删除前: ";
    for (int val : ms) cout << val << " ";
    cout << endl;
    // 输出: 10 10 10 20 20 30

    // erase(value) 会删除 **所有** 等于该值的元素
    size_t count = ms.erase(10);
    cout << "erase(10) 删除了 " << count << " 个元素" << endl;  // 输出: 3

    cout << "删除后: ";
    for (int val : ms) cout << val << " ";
    cout << endl;
    // 输出: 20 20 30

    // 如果只想删除一个，用 find() + erase(iterator)
    multiset<int> ms2 = {10, 10, 10, 20, 20, 30};
    auto it = ms2.find(10);   // 找到第一个 10
    if (it != ms2.end()) {
        ms2.erase(it);        // 只删除这一个
    }

    cout << "只删一个10后: ";
    for (int val : ms2) cout << val << " ";
    cout << endl;
    // 输出: 10 10 20 20 30（只少了一个 10）

    return 0;
}
```

#### 对比4：equal_range() 在 multiset 中更有意义

```cpp name=multiset_equal_range.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    multiset<int> ms = {10, 20, 20, 20, 30, 40, 50};

    auto range = ms.equal_range(20);

    cout << "值为 20 的所有元素: ";
    for (auto it = range.first; it != range.second; ++it) {
        cout << *it << " ";
    }
    cout << endl;
    // 输出: 20 20 20

    return 0;
}
```

### 6.3 如何选择 set 还是 multiset？

```
需要存储的元素可能有重复值吗？
     |
     ├── 否 → 使用 set（自动去重，insert 有成功/失败反馈）
     |
     └── 是 → 使用 multiset（允许重复，适合统计频次等场景）
```

**使用场景举例**：

| 场景                           | 推荐容器               |
| ------------------------------ | ---------------------- |
| 学生选课名单（每人只选一次）   | `set`                  |
| 考试成绩排名（可能有相同分数） | `multiset`             |
| 去重后排序输出                 | `set`                  |
| 统计单词出现频次               | `multiset`（或 `map`） |
| 维护一组不重复的 ID            | `set`                  |

---

## 第七章 综合实战与进阶

### 7.1 自定义排序规则

默认情况下，`set` 按**升序**排列。如果需要**降序**或自定义排序，可以使用**仿函数（functor）**。

```cpp name=set_custom_sort.cpp
#include <iostream>
#include <set>
using namespace std;

// 方法一：定义仿函数（函数对象）
class MyCompare {
public:
    // 重载 () 运算符
    bool operator()(const int &a, const int &b) const {
        return a > b;  // 降序排列
    }
};

int main() {
    // ===== 默认升序 =====
    set<int> s1 = {30, 10, 50, 20, 40};
    cout << "默认(升序): ";
    for (int val : s1) cout << val << " ";
    cout << endl;
    // 输出: 10 20 30 40 50

    // ===== 使用仿函数实现降序 =====
    // 注意：排序规则必须在声明 set 时指定，作为第二个模板参数
    set<int, MyCompare> s2;
    s2.insert(30);
    s2.insert(10);
    s2.insert(50);
    s2.insert(20);
    s2.insert(40);

    cout << "自定义(降序): ";
    for (int val : s2) cout << val << " ";
    cout << endl;
    // 输出: 50 40 30 20 10

    // ===== 使用 STL 内置的 greater<> =====
    set<int, greater<int>> s3 = {30, 10, 50, 20, 40};
    cout << "greater(降序): ";
    for (int val : s3) cout << val << " ";
    cout << endl;
    // 输出: 50 40 30 20 10

    return 0;
}
```

### 7.2 set 存储自定义对象

当 `set` 存储自定义类型时，**必须**提供排序规则，否则编译报错。

```cpp name=set_custom_class.cpp
#include <iostream>
#include <set>
#include <string>
using namespace std;

class Student {
public:
    string name;
    int age;
    int score;

    Student(string n, int a, int s) : name(n), age(a), score(s) {}
};

// 仿函数：按分数降序排列，分数相同按年龄升序
class CompareStudent {
public:
    bool operator()(const Student& a, const Student& b) const {
        if (a.score != b.score) {
            return a.score > b.score;  // 分数高的排前面
        }
        return a.age < b.age;          // 分数相同，年龄小的排前面
    }
};

int main() {
    set<Student, CompareStudent> students;

    students.insert(Student("张三", 20, 85));
    students.insert(Student("李四", 19, 92));
    students.insert(Student("王五", 21, 78));
    students.insert(Student("赵六", 20, 92));
    students.insert(Student("孙七", 18, 85));

    cout << "学生成绩排名：" << endl;
    cout << "----------------------------" << endl;
    for (const auto& stu : students) {
        cout << "姓名: " << stu.name
             << "  年龄: " << stu.age
             << "  分数: " << stu.score << endl;
    }

    /*
    输出：
    姓名: 李四  年龄: 19  分数: 92
    姓名: 赵六  年龄: 20  分数: 92
    姓名: 孙七  年龄: 18  分数: 85
    姓名: 张三  年龄: 20  分数: 85
    姓名: 王五  年龄: 21  分数: 78
    */

    return 0;
}
```

### 7.3 set 的遍历方式汇总

```cpp name=set_traversal.cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {50, 30, 10, 40, 20};

    // 方式1：范围 for 循环（最简洁，推荐）
    cout << "方式1(范围for): ";
    for (int val : s) {
        cout << val << " ";
    }
    cout << endl;

    // 方式2：迭代器遍历
    cout << "方式2(迭代器): ";
    for (set<int>::iterator it = s.begin(); it != s.end(); ++it) {
        cout << *it << " ";
    }
    cout << endl;

    // 方式3：auto 简化迭代器
    cout << "方式3(auto): ";
    for (auto it = s.begin(); it != s.end(); ++it) {
        cout << *it << " ";
    }
    cout << endl;

    // 方式4：反向遍历（降序输出）
    cout << "方式4(反向): ";
    for (auto rit = s.rbegin(); rit != s.rend(); ++rit) {
        cout << *rit << " ";
    }
    cout << endl;
    // 输出: 50 40 30 20 10

    return 0;
}
```

### 7.4 实战案例：利用 set 进行数据去重排序

```cpp name=set_practical_example.cpp
#include <iostream>
#include <set>
#include <vector>
#include <string>
using namespace std;

int main() {
    // 场景：统计一段文本中出现了哪些不重复的单词，并按字典序输出
    vector<string> words = {
        "apple", "banana", "cherry", "apple", "date",
        "banana", "elderberry", "fig", "cherry", "apple"
    };

    // 方法：将所有单词插入 set，自动去重 + 排序
    set<string> uniqueWords(words.begin(), words.end());

    cout << "原始单词数量: " << words.size() << endl;        // 10
    cout << "去重后单词数量: " << uniqueWords.size() << endl;  // 7

    cout << "\n按字典序排列的不重复单词:" << endl;
    int index = 1;
    for (const string& word : uniqueWords) {
        cout << "  " << index++ << ". " << word << endl;
    }
    /*
    输出:
      1. apple
      2. banana
      3. cherry
      4. date
      5. elderberry
      6. fig
    */

    // 检查某个单词是否出现过
    string target = "grape";
    if (uniqueWords.count(target)) {
        cout << "\n\"" << target << "\" 出现过" << endl;
    } else {
        cout << "\n\"" << target << "\" 没有出现过" << endl;  // ✅
    }

    return 0;
}
```

### 7.5 set 常用操作时间复杂度速查表

| 操作            | 时间复杂度 | 说明             |
| --------------- | ---------- | ---------------- |
| `insert()`      | O(log n)   | 红黑树插入       |
| `erase()`       | O(log n)   | 红黑树删除       |
| `find()`        | O(log n)   | 红黑树查找       |
| `count()`       | O(log n)   | 等价于 find      |
| `lower_bound()` | O(log n)   | 二分查找         |
| `upper_bound()` | O(log n)   | 二分查找         |
| `size()`        | O(1)       | 直接返回         |
| `empty()`       | O(1)       | 直接返回         |
| `clear()`       | O(n)       | 需要释放所有节点 |
| 遍历全部元素    | O(n)       | 中序遍历         |

### 7.6 set vs 其他容器对比

| 特性     | `set`    | `vector`      | `list`        | `map`       |
| -------- | -------- | ------------- | ------------- | ----------- |
| 有序     | ✅ 自动   | ❌ 需手动sort  | ❌ 需手动sort  | ✅ 按key自动 |
| 去重     | ✅ 自动   | ❌ 需手动      | ❌ 需手动      | ✅ key自动   |
| 随机访问 | ❌        | ✅ `[]`        | ❌             | ❌           |
| 查找效率 | O(log n) | O(n)          | O(n)          | O(log n)    |
| 插入效率 | O(log n) | O(n) 中间插入 | O(1) 已知位置 | O(log n)    |
| 存储方式 | 键值     | 值            | 值            | 键值对      |

---

## 附录：本教程 API 速查手册

```
┌─────────────────────────────────────────────────────────────────┐
│                      set<T> 常用 API 速查                        │
├──────────────────┬──────────────────────────────────────────────┤
│     构造         │  set<T> s;                                    │
│                  │  set<T> s = {a, b, c};                       │
│                  │  set<T> s2(s1);                              │
│                  │  set<T> s(beg, end);                         │
├──────────────────┼──────────────────────────────────────────────┤
│     赋值         │  s2 = s1;                                     │
├──────────────────┼──────────────────────────────────────────────┤
│     大小         │  s.size()   s.empty()                         │
├──────────────────┼──────────────────────────────────────────────┤
│     交换         │  s1.swap(s2)                                  │
├──────────────────┼──────────────────────────────────────────────┤
│     插入         │  s.insert(value)                              │
│                  │  s.insert(beg, end)                           │
│                  │  s.emplace(args...)                           │
├──────────────────┼────────────────────────────────────────────── ┤
│     删除         │  s.erase(value)                                │
│                  │  s.erase(it)                                   │
│                  │  s.erase(beg, end)                             │
│                  │  s.clear()                                     │
├──────────────────┼──────────────────────────────────────────────┤
│     查找         │  s.find(value)                                 │
│                  │  s.count(value)                                │
│                  │  s.lower_bound(value)                          │
│                  │  s.upper_bound(value)                          │
│                  │  s.equal_range(value)                          │
├──────────────────┼──────────────────────────────────────────────┤
│     迭代器       │  s.begin()  s.end()                            │
│                  │  s.rbegin() s.rend()                           │
├──────────────────┼──────────────────────────────────────────────┤
│   自定义排序      │  set<T, MyCompare> s;                         │
│                  │  set<T, greater<T>> s;                         │
└──────────────────┴──────────────────────────────────────────────┘
```

---

> 📝 **学习建议**：
> 1. **先掌握基础**：构造、插入、删除、查找 — 这是日常编程中最高频使用的操作
> 2. **理解返回值**：`insert()` 返回 `pair<iterator, bool>` 是面试和笔试的高频考点
> 3. **区分 set 和 multiset**：明确何时需要去重、何时允许重复
> 4. **动手实践**：每个代码示例都自己编译运行一遍，尝试修改参数观察结果
> 5. **进阶方向**：掌握自定义排序后，可以继续学习 `map`/`unordered_set`/`unordered_map`