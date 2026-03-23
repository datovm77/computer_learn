# 第十四阶段：`unordered_set` / `unordered_map` 容器（补充）

> 🎯 **学习目标**：在掌握了有序容器 `set`/`map` 之后，立即学习它们的无序版本，通过对比加深理解。掌握哈希表的底层原理、无序容器的使用方法，以及何时该选择有序 vs 无序容器。

> 📌 **编译标准说明**：本文大部分示例在 C++17 可用；涉及 `contains()`、`operator== = default`（结构体默认比较）等示例需要 C++20（或更高）编译。

---

# 一、`unordered_set`

---

## 1.1 基本概念

### 1.1.1 什么是 `unordered_set`？

`unordered_set` 是 C++11 引入的一种**无序关联容器**，它存储的元素是**唯一的**（不允许重复），但**不保证任何特定的排列顺序**。

```cpp
#include <unordered_set>  // 必须包含此头文件

std::unordered_set<int> us = {3, 1, 4, 1, 5};
// 存储结果可能是：{5, 4, 3, 1}（1 被去重，顺序不确定）
```

### 1.1.2 底层数据结构：哈希表（Hash Table）

要真正理解 `unordered_set`，必须先理解它的底层——**哈希表**。

#### 哈希表的工作原理

哈希表的核心思想是：**通过一个哈希函数（Hash Function），将元素直接映射到一个"桶（Bucket）"中，从而实现近乎 O(1) 的查找速度。**

```
假设我们有一个 unordered_set，底层有 8 个桶（bucket）

哈希函数：hash(key) % 8

插入元素 3：hash(3) % 8 = 3  → 放入桶 3
插入元素 10：hash(10) % 8 = 2 → 放入桶 2
插入元素 11：hash(11) % 8 = 3 → 放入桶 3（与 3 冲突！）

桶的示意图：
┌─────┬─────┬─────┬──────────┬─────┬─────┬─────┬─────┐
│  0  │  1  │  2  │    3     │  4  │  5  │  6  │  7  │
│     │     │ 10  │  3 → 11  │     │     │     │     │
└─────┴─────┴─────┴──────────┴─────┴─────┴─────┴─────┘
                    ↑ 哈希冲突：用链表串起来
```

当两个元素被哈希到同一个桶时，就发生了**哈希冲突（Hash Collision）**。STL 中的 `unordered_set` 使用**链地址法（Separate Chaining）**来解决冲突——同一个桶中的元素用链表串联。

#### 时间复杂度

| 操作 | 平均复杂度 | 最坏复杂度 |
| ---- | ---------- | ---------- |
| 插入 | **O(1)**   | O(n)       |
| 查找 | **O(1)**   | O(n)       |
| 删除 | **O(1)**   | O(n)       |

> ⚠️ 最坏情况发生在所有元素都被哈希到同一个桶中（退化成链表），但实际使用中几乎不会遇到。

### 1.1.3 `unordered_set` 与 `set` 的核心区别

| 特性           | `set`                    | `unordered_set`                    |
| -------------- | ------------------------ | ---------------------------------- |
| **底层结构**   | 红黑树（平衡二叉搜索树） | 哈希表                             |
| **元素有序性** | ✅ 自动排序（默认升序）   | ❌ 无序                             |
| **查找复杂度** | O(log n)                 | **O(1)**（平均）                   |
| **插入复杂度** | O(log n)                 | **O(1)**（平均）                   |
| **删除复杂度** | O(log n)                 | **O(1)**（平均）                   |
| **元素要求**   | 需要支持 `<` 比较运算符  | 需要支持**哈希函数**和 `==` 运算符 |
| **内存开销**   | 较小                     | 较大（需要维护桶数组）             |
| **遍历顺序**   | 有序遍历                 | 无序遍历                           |
| **适用场景**   | 需要有序访问、范围查询   | 只关心"存在性"查询，追求速度       |

#### 选择建议

```
需要元素有序？ ──── 是 ──→ 用 set
       │
       否
       │
需要范围查询？ ──── 是 ──→ 用 set（lower_bound / upper_bound）
       │
       否
       │
只关心查找速度？── 是 ──→ 用 unordered_set ✓
```

**简单记忆：如果你不需要排序，`unordered_set` 几乎总是更快。**

---

## 1.2 构造和赋值

### 1.2.1 默认构造

```cpp
#include <iostream>
#include <unordered_set>
#include <string>
using namespace std;

int main() {
    // 1. 默认构造：创建一个空的 unordered_set
    unordered_set<int> us1;
    cout << "us1 大小: " << us1.size() << endl;  // 0

    // 2. 默认构造 string 类型
    unordered_set<string> us2;
    
    return 0;
}
```

### 1.2.2 初始化列表构造（最常用）

```cpp
#include <iostream>
#include <unordered_set>
#include <string>
using namespace std;

int main() {
    // 初始化列表构造
    unordered_set<int> us1 = {3, 1, 4, 1, 5, 9, 2, 6};
    // 注意：重复的 1 会被自动去重
    
    cout << "us1 中的元素: ";
    for (int x : us1) {
        cout << x << " ";
    }
    cout << endl;
    // 输出顺序不确定！可能是：6 2 9 5 4 1 3（每次运行可能不同）
    
    // 对比 set 的输出（一定是有序的）：1 2 3 4 5 6 9
    
    // string 类型
    unordered_set<string> fruits = {"apple", "banana", "cherry", "apple"};
    // "apple" 被去重
    cout << "水果种类数: " << fruits.size() << endl;  // 3
    
    return 0;
}
```

### 1.2.3 拷贝构造和移动构造

```cpp
#include <iostream>
#include <unordered_set>
#include <string>
using namespace std;

int main() {
    unordered_set<int> us1 = {10, 20, 30};
    
    // 拷贝构造
    unordered_set<int> us2(us1);       // 深拷贝
    unordered_set<int> us3 = us1;      // 等价写法
    
    // 移动构造（C++11）—— us4 "偷走" us1 的内存，us1 变为空
    unordered_set<int> us4(std::move(us1));
    cout << "us1 大小（被移动后）: " << us1.size() << endl;  // 0
    cout << "us4 大小: " << us4.size() << endl;              // 3
    
    return 0;
}
```

### 1.2.4 迭代器范围构造

```cpp
#include <iostream>
#include <unordered_set>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 2, 1, 4, 5};
    
    // 从 vector 的迭代器范围构造 unordered_set（自动去重）
    unordered_set<int> us(v.begin(), v.end());
    
    cout << "去重后元素个数: " << us.size() << endl;  // 5
    for (int x : us) {
        cout << x << " ";
    }
    cout << endl;
    
    return 0;
}
```

> 💡 **技巧**：利用迭代器范围构造，可以轻松实现**对 vector 去重**。

### 1.2.5 赋值操作

```cpp
#include <iostream>
#include <unordered_set>
#include <utility>
using namespace std;

int main() {
    unordered_set<int> us1 = {1, 2, 3};
    unordered_set<int> us2;
    
    // 拷贝赋值
    us2 = us1;
    
    // 初始化列表赋值
    us2 = {10, 20, 30};
    
    // 移动赋值
    unordered_set<int> us3;
    us3 = std::move(us2);  // us2 变为空
    
    return 0;
}
```

---

## 1.3 插入和删除

### 1.3.1 `insert()` —— 插入元素

```cpp
#include <iostream>
#include <unordered_set>
#include <vector>
using namespace std;

int main() {
    unordered_set<int> us;
    
    // ===== 方式一：插入单个元素 =====
    // insert() 返回 pair<iterator, bool>
    //   - first：指向插入元素（或已存在的相同元素）的迭代器
    //   - second：表示是否插入成功（true = 新插入，false = 已存在）
    
    auto [it1, success1] = us.insert(10);   // C++17 结构化绑定
    cout << *it1 << ", 插入成功: " << boolalpha << success1 << endl;
    // 输出：10, 插入成功: true（boolalpha令“1”变为true）
    
    auto [it2, success2] = us.insert(10);   // 再次插入 10
    cout << *it2 << ", 插入成功: " << boolalpha << success2 << endl;
    // 输出：10, 插入成功: false（因为 10 已经存在）
    
    // 如果不使用 C++17，可以这样写：
    pair<unordered_set<int>::iterator, bool> result = us.insert(20);
    if (result.second) {
        cout << "20 插入成功" << endl;
    }
    
    // ===== 方式二：插入初始化列表 =====
    us.insert({30, 40, 50, 30});  // 30 重复，只会插入一个
    
    // ===== 方式三：插入迭代器范围 =====
    vector<int> v = {60, 70};
    us.insert(v.begin(), v.end());
    
    // 输出所有元素
    for (int x : us) {
        cout << x << " ";
    }
    cout << endl;
    
    return 0;
}
```

### 1.3.2 `emplace()` —— 就地构造插入

`emplace()` 和 `insert()` 功能类似，但它**直接在容器内部构造元素**，避免了临时对象的创建和拷贝，**效率更高**。

```cpp
#include <iostream>
#include <unordered_set>
#include <string>
using namespace std;

int main() {
    unordered_set<string> us;
    
    // insert 方式：先创建临时 string，再拷贝/移动到容器中
    us.insert(string("hello"));
    
    // emplace 方式：直接在容器内部构造 string，更高效 ✓
    us.emplace("world");  // 直接传构造参数
    
    // emplace 也返回 pair<iterator, bool>
    auto [it, success] = us.emplace("hello");  // 已存在
    cout << "emplace hello 成功: " << boolalpha << success << endl;  // false
    
    // 对于像 int 这样的基本类型，insert 和 emplace 区别不大
    unordered_set<int> nums;
    nums.emplace(42);  // OK
    nums.insert(42);   // 也 OK，效率差异可忽略
    
    return 0;
}
```

> 💡 **最佳实践**：对于复杂类型（如 `string`），优先使用 `emplace()`；对于基本类型（如 `int`），两者都可以。

### 1.3.3 `erase()` —— 删除元素

`erase()` 有三种重载形式：

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

int main() {
    unordered_set<int> us = {10, 20, 30, 40, 50};
    
    // ===== 方式一：按值删除 =====
    // 返回删除的元素个数（0 或 1，因为 unordered_set 中元素唯一）
    size_t count = us.erase(30);
    cout << "删除了 " << count << " 个元素" << endl;  // 1
    
    count = us.erase(99);  // 不存在的元素
    cout << "删除了 " << count << " 个元素" << endl;  // 0（不会报错）
    
    // ===== 方式二：按迭代器删除 =====
    auto it = us.find(20);  // 先找到元素
    if (it != us.end()) {
        us.erase(it);  // 删除迭代器指向的元素
        cout << "成功删除 20" << endl;
    }
    
    // ===== 方式三：删除迭代器范围 =====
    // 注意：由于 unordered_set 是无序的，范围删除的实用性不如 set
    // us.erase(us.begin(), us.end());  // 等同于 clear()
    
    for (int x : us) {
        cout << x << " ";
    }
    cout << endl;
    
    return 0;
}
```

### 1.3.4 `clear()` —— 清空容器

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

int main() {
    unordered_set<int> us = {1, 2, 3, 4, 5};
    cout << "清空前大小: " << us.size() << endl;  // 5
    
    us.clear();  // 删除所有元素
    cout << "清空后大小: " << us.size() << endl;  // 0
    cout << "是否为空: " << boolalpha << us.empty() << endl;  // true
    
    return 0;
}
```

### 1.3.5 插入删除操作一览表

| 操作                 | 说明           | 返回值                 |
| -------------------- | -------------- | ---------------------- |
| `insert(val)`        | 插入值         | `pair<iterator, bool>` |
| `insert({...})`      | 插入初始化列表 | `void`                 |
| `emplace(args...)`   | 就地构造插入   | `pair<iterator, bool>` |
| `erase(val)`         | 按值删除       | 删除个数（`size_t`）   |
| `erase(it)`          | 按迭代器删除   | 下一个元素的迭代器     |
| `erase(first, last)` | 删除范围       | 下一个元素的迭代器     |
| `clear()`            | 清空所有元素   | `void`                 |

---

## 1.4 查找和统计

### 1.4.1 `find()` —— 查找元素

```cpp
#include <iostream>
#include <unordered_set>
#include <string>
using namespace std;

int main() {
    unordered_set<string> colors = {"red", "green", "blue", "yellow"};
    
    // find() 返回迭代器
    //   - 找到：返回指向该元素的迭代器
    //   - 未找到：返回 end()
    
    auto it = colors.find("green");
    if (it != colors.end()) {
        cout << "找到了: " << *it << endl;  // 找到了: green
    } else {
        cout << "没找到" << endl;
    }
    
    // 查找不存在的元素
    if (colors.find("purple") == colors.end()) {
        cout << "purple 不在集合中" << endl;
    }
    
    return 0;
}
```

### 1.4.2 `count()` —— 统计元素个数

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

int main() {
    unordered_set<int> us = {1, 2, 3, 4, 5};
    
    // count() 返回元素出现的次数
    // 对于 unordered_set，结果只能是 0 或 1
    cout << "3 出现次数: " << us.count(3) << endl;  // 1
    cout << "9 出现次数: " << us.count(9) << endl;  // 0
    
    // 常用 count() 来判断元素是否存在（等价于 find() != end()）
    if (us.count(3)) {  // 非零即为 true
        cout << "3 存在" << endl;
    }
    
    return 0;
}
```

### 1.4.3 `contains()` —— 判断元素是否存在（C++20）

C++20 新增了 `contains()` 方法，它比 `count()` 和 `find()` 更加**语义清晰**。

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

int main() {
    unordered_set<int> us = {1, 2, 3, 4, 5};
    
    // C++20 contains()：直接返回 bool
    if (us.contains(3)) {
        cout << "3 存在于集合中" << endl;  // ✓
    }
    
    if (!us.contains(99)) {
        cout << "99 不存在于集合中" << endl;  // ✓
    }
    
    return 0;
}
// 编译时需要 C++20 标准：g++ -std=c++20 xxx.cpp
```

### 1.4.4 三种查找方式的对比

```cpp
unordered_set<int> us = {1, 2, 3};

// 方式一：find()（C++11，所有版本可用）
if (us.find(2) != us.end()) { /* 存在 */ }

// 方式二：count()（C++11，所有版本可用）
if (us.count(2)) { /* 存在 */ }

// 方式三：contains()（C++20，最推荐 ✓）
if (us.contains(2)) { /* 存在 */ }
```

| 方法         | 版本要求  | 返回值   | 语义清晰度                  | 推荐度         |
| ------------ | --------- | -------- | --------------------------- | -------------- |
| `find()`     | C++11     | 迭代器   | 一般（需要与 end() 比较）   | 需要迭代器时用 |
| `count()`    | C++11     | `size_t` | 一般（利用隐式转换为 bool） | 兼容旧代码时用 |
| `contains()` | **C++20** | `bool`   | **最好**                    | **首选** ✓     |

---

## 1.5 大小操作

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

int main() {
    unordered_set<int> us;
    
    // empty()：判断是否为空
    cout << "是否为空: " << boolalpha << us.empty() << endl;  // true
    
    us = {10, 20, 30, 40, 50};
    
    // size()：返回元素个数
    cout << "元素个数: " << us.size() << endl;  // 5
    
    // max_size()：返回理论上的最大容量（受系统内存限制）
    cout << "最大容量: " << us.max_size() << endl;  // 一个很大的数
    
    // empty()
    cout << "是否为空: " << boolalpha << us.empty() << endl;  // false
    
    return 0;
}
```

---

## 1.6 `unordered_multiset` 简要了解

`unordered_multiset` 是 `unordered_set` 的**允许重复**版本，定义在同一个头文件 `<unordered_set>` 中。

### 1.6.1 与 `unordered_set` 的区别

| 特性              | `unordered_set`        | `unordered_multiset`   |
| ----------------- | ---------------------- | ---------------------- |
| 元素唯一性        | ✅ 唯一                 | ❌ 允许重复             |
| `count()` 返回值  | 0 或 1                 | 0 到 n                 |
| `insert()` 返回值 | `pair<iterator, bool>` | `iterator`（总是成功） |

### 1.6.2 基本使用

```cpp
#include <iostream>
#include <unordered_set>  // 同一个头文件！
using namespace std;

int main() {
    unordered_multiset<int> ums = {1, 2, 2, 3, 3, 3};
    
    cout << "元素个数: " << ums.size() << endl;  // 6（不去重）
    
    // count() 可以返回大于 1 的值
    cout << "3 出现次数: " << ums.count(3) << endl;  // 3
    cout << "2 出现次数: " << ums.count(2) << endl;  // 2
    
    // insert() 总是成功，返回迭代器（不是 pair）
    auto it = ums.insert(2);  // 再插入一个 2
    cout << "2 现在出现次数: " << ums.count(2) << endl;  // 3
    
    // erase(val) 会删除所有相同的值
    ums.erase(3);  // 删除所有的 3
    cout << "删除后3的个数: " << ums.count(3) << endl;  // 0
    
    // 如果只想删除一个，用迭代器版本的 erase
    auto it2 = ums.find(2);  // 找到第一个 2
    if (it2 != ums.end()) {
        ums.erase(it2);  // 只删除一个 2
    }
    cout << "删除一个后2的个数: " << ums.count(2) << endl;  // 2
    
    return 0;
}
```

> 💡 **使用场景**：当你需要统计元素频次，同时需要 O(1) 查找，但又想保留重复元素时，可以用 `unordered_multiset`。不过实际开发中，更常见的做法是用 `unordered_map<key, int>` 来统计频次。

---

# 二、`unordered_map`

---

## 2.1 基本概念

### 2.1.1 什么是 `unordered_map`？

`unordered_map` 是 C++11 引入的**无序关联容器**，存储的是 **键值对（key-value pair）**，**键（key）是唯一的**，但元素**没有特定的排列顺序**。

```cpp
#include <unordered_map>  // 必须包含此头文件
#include <string>

std::unordered_map<std::string, int> ages = {
    {"Alice", 25},
    {"Bob", 30},
    {"Charlie", 35}
};
// 遍历顺序不确定，和插入顺序无关
```

### 2.1.2 底层数据结构：哈希表

`unordered_map` 的底层同样是**哈希表**，但它哈希的是 **key**，value 跟随 key 一起存储。

```
假设 unordered_map 有 4 个桶

插入 {"Alice", 25}：hash("Alice") % 4 = 1  → 桶 1
插入 {"Bob", 30}：  hash("Bob") % 4 = 3    → 桶 3
插入 {"Eve", 22}：  hash("Eve") % 4 = 1    → 桶 1（冲突！）

桶的示意图：
┌─────┬──────────────────────────┬─────┬──────────┐
│  0  │           1              │  2  │    3     │
│     │ {"Alice",25}→{"Eve",22}  │     │{"Bob",30}│
└─────┴──────────────────────────┴─────┴──────────┘
```

### 2.1.3 `unordered_map` 与 `map` 的核心区别

| 特性             | `map`            | `unordered_map`                 |
| ---------------- | ---------------- | ------------------------------- |
| **底层结构**     | 红黑树           | 哈希表                          |
| **元素有序性**   | ✅ 按 key 排序    | ❌ 无序                          |
| **查找复杂度**   | O(log n)         | **O(1)**（平均）                |
| **插入复杂度**   | O(log n)         | **O(1)**（平均）                |
| **删除复杂度**   | O(log n)         | **O(1)**（平均）                |
| **key 要求**     | 需要 `operator<` | 需要**哈希函数**和 `operator==` |
| **内存开销**     | 较小             | 较大                            |
| **遍历顺序**     | key 有序         | 无序                            |
| **`operator[]`** | ✅ 支持           | ✅ 支持                          |
| **适用场景**     | 需要 key 有序    | 只关心 key-value 映射，追求速度 |

#### 选择建议

```
需要按 key 排序遍历？ ──── 是 ──→ 用 map
       │
       否
       │
需要范围查询？ ────── 是 ──→ 用 map（lower_bound / upper_bound）
       │
       否
       │
只需要 key-value 映射？── 是 ──→ 用 unordered_map ✓（更快）
```

> 💡 **实际开发中，`unordered_map` 的使用频率远高于 `map`**，因为大部分场景下我们只关心"通过 key 找 value"，不需要排序。

---

## 2.2 构造和赋值

### 2.2.1 默认构造

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
#include <vector>
using namespace std;

int main() {
    // 默认构造：创建空的 unordered_map
    unordered_map<string, int> um1;
    cout << "um1 大小: " << um1.size() << endl;  // 0
    
    // key 和 value 可以是任意类型
    unordered_map<int, string> um2;        // int → string
    unordered_map<string, vector<int>> um3; // string → vector<int>
    
    return 0;
}
```

### 2.2.2 初始化列表构造（最常用）

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    // 使用初始化列表构造
    unordered_map<string, int> scores = {
        {"math", 95},
        {"english", 88},
        {"physics", 92},
        {"math", 100}   // key 重复！后面的会被忽略，math 仍然是 95
    };
    
    cout << "科目数: " << scores.size() << endl;  // 3（math 没有重复插入）
    
    // 遍历（顺序不确定）
    for (const auto& [key, value] : scores) {  // C++17 结构化绑定
        cout << key << ": " << value << endl;
    }
    
    return 0;
}
```

> ⚠️ **注意**：初始化列表中如果有重复的 key，只有**第一个**会被保留，后续重复的 key 会被忽略。这和 `operator[]` 的行为不同（`operator[]` 会覆盖）。

### 2.2.3 拷贝构造、移动构造和赋值

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um1 = {{"a", 1}, {"b", 2}};
    
    // 拷贝构造
    unordered_map<string, int> um2(um1);
    unordered_map<string, int> um3 = um1;
    
    // 移动构造
    unordered_map<string, int> um4(std::move(um1));
    // um1 现在为空
    
    // 赋值操作
    unordered_map<string, int> um5;
    um5 = um2;                        // 拷贝赋值
    um5 = {{"x", 10}, {"y", 20}};    // 初始化列表赋值
    um5 = std::move(um3);            // 移动赋值
    
    return 0;
}
```

---

## 2.3 插入和删除

### 2.3.1 `insert()` —— 插入键值对

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um;
    
    // ===== 方式一：插入 pair =====
    um.insert(pair<string, int>("Alice", 25));
    
    // 更简洁的写法：
    um.insert({"Bob", 30});                    // 推荐 ✓
    um.insert(make_pair("Charlie", 35));       // 也可以
    
    // insert() 返回 pair<iterator, bool>
    auto [it, success] = um.insert({"Alice", 99});  // Alice 已存在
    cout << "插入 Alice 成功: " << boolalpha << success << endl;  // false
    cout << "Alice 的值仍是: " << it->second << endl;             // 25（不会被覆盖！）
    
    // ===== 方式二：插入初始化列表 =====
    um.insert({
        {"Dave", 28},
        {"Eve", 22}
    });
    
    // 遍历
    for (const auto& [name, age] : um) {
        cout << name << ": " << age << endl;
    }
    
    return 0;
}
```

> ⚠️ **重要**：`insert()` 在 key 已存在时**不会覆盖**原有的 value！这是初学者常犯的错误。

### 2.3.2 `emplace()` —— 就地构造插入

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um;
    
    // emplace 直接传入 key 和 value 的构造参数
    um.emplace("Alice", 25);     // 更高效（避免临时 pair 的创建）
    um.emplace("Bob", 30);
    
    // 同样返回 pair<iterator, bool>
    auto [it, ok] = um.emplace("Alice", 99);
    cout << "emplace Alice: " << boolalpha << ok << endl;  // false（已存在）
    
    return 0;
}
```

### 2.3.3 `operator[]` —— 下标访问（最常用！）

`operator[]` 是 `unordered_map` 最强大也最需要注意的操作：

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um;
    
    // ===== 用途一：插入新元素 =====
    um["Alice"] = 25;   // key 不存在 → 创建新键值对 {"Alice", 25}
    um["Bob"] = 30;
    
    // ===== 用途二：修改已有元素 =====
    um["Alice"] = 26;   // key 已存在 → 覆盖！Alice 变为 26
    cout << "Alice: " << um["Alice"] << endl;  // 26
    
    // ===== 用途三：读取元素 =====
    cout << "Bob: " << um["Bob"] << endl;  // 30
    
    // ===== ⚠️ 危险：读取不存在的 key =====
    cout << "Charlie: " << um["Charlie"] << endl;  // 0
    // 注意！！上面这行代码做了两件事：
    // 1. 因为 "Charlie" 不存在，所以自动创建 {"Charlie", 0}（int 的默认值）
    // 2. 返回该默认值 0
    
    cout << "当前大小: " << um.size() << endl;  // 3（Charlie 被偷偷插入了！）
    
    return 0;
}
```

> ⚠️ **这是初学者最容易踩的坑！** `operator[]` 访问不存在的 key 时，**会自动插入**一个新的键值对（value 为该类型的默认值）。如果你只想查询而不想意外插入，请使用 `find()` 或 `count()`。

#### `operator[]` 与 `insert()` 的核心区别

```cpp
unordered_map<string, int> um = {{"Alice", 25}};

// insert()：key 存在时 → 不覆盖
um.insert({"Alice", 99});
cout << um["Alice"] << endl;  // 25（未改变）

// operator[]：key 存在时 → 覆盖
um["Alice"] = 99;
cout << um["Alice"] << endl;  // 99（被覆盖了）
```

| 操作         | key 不存在时 | key 已存在时           |
| ------------ | ------------ | ---------------------- |
| `insert()`   | 插入新元素   | **不覆盖**，返回 false |
| `operator[]` | 插入默认值   | **覆盖**               |
| `emplace()`  | 插入新元素   | **不覆盖**，返回 false |

#### `operator[]` 的经典应用：统计频次

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
#include <vector>
using namespace std;

int main() {
    vector<string> words = {"apple", "banana", "apple", "cherry", "banana", "apple"};
    
    // 统计每个单词出现的次数
    unordered_map<string, int> freq;
    for (const string& word : words) {
        freq[word]++;
        // 第一次遇到某个 word 时：
        //   freq[word] 自动创建并初始化为 0，然后 ++ 变为 1
        // 之后再遇到同一个 word：
        //   freq[word] 在已有值基础上 ++
    }
    
    for (const auto& [word, count] : freq) {
        cout << word << ": " << count << "次" << endl;
    }
    // 可能的输出：
    // cherry: 1次
    // banana: 2次
    // apple: 3次
    
    return 0;
}
```

> 💡 这是 `unordered_map` 最经典的用法之一，利用 `operator[]` 自动初始化的特性，一行代码搞定频次统计。

### 2.3.4 `erase()` —— 删除元素

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um = {
        {"Alice", 25}, {"Bob", 30}, {"Charlie", 35}, {"Dave", 28}
    };
    
    // 方式一：按 key 删除
    size_t count = um.erase("Charlie");
    cout << "删除了 " << count << " 个元素" << endl;  // 1
    
    um.erase("NotExist");  // 不存在的 key，不会报错，返回 0
    
    // 方式二：按迭代器删除
    auto it = um.find("Bob");
    if (it != um.end()) {
        um.erase(it);
    }
    
    // 方式三：清空所有元素
    // um.clear();
    
    for (const auto& [name, age] : um) {
        cout << name << ": " << age << endl;
    }
    
    return 0;
}
```

---

## 2.4 查找和统计

### 2.4.1 `find()` —— 查找元素

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um = {
        {"Alice", 25}, {"Bob", 30}, {"Charlie", 35}
    };
    
    // find() 返回迭代器，指向 pair<const Key, Value>
    auto it = um.find("Bob");
    if (it != um.end()) {
        cout << "找到 Bob，年龄: " << it->second << endl;  // 30
        
        // 也可以通过迭代器修改 value（但不能修改 key）
        it->second = 31;  // 修改 Bob 的年龄
    }
    
    // 查找不存在的 key
    if (um.find("Zara") == um.end()) {
        cout << "Zara 不在 map 中" << endl;
    }
    
    return 0;
}
```

> 💡 **`find()` vs `operator[]`**：如果你只想查（不想插入），必须用 `find()`。用 `operator[]` 查不存在的 key 会导致意外插入！

### 2.4.2 `count()` —— 统计 key 是否存在

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um = {{"a", 1}, {"b", 2}, {"c", 3}};
    
    // 对于 unordered_map，count() 只返回 0 或 1
    if (um.count("b")) {
        cout << "key 'b' 存在" << endl;
    }
    
    if (!um.count("z")) {
        cout << "key 'z' 不存在" << endl;
    }
    
    return 0;
}
```

### 2.4.3 `contains()` —— C++20 判断 key 是否存在

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um = {{"x", 10}, {"y", 20}};
    
    // C++20：最清晰的判断方式
    if (um.contains("x")) {
        cout << "x 存在" << endl;
    }
    
    if (!um.contains("z")) {
        cout << "z 不存在" << endl;
    }
    
    return 0;
}
// 编译：g++ -std=c++20 xxx.cpp
```

### 2.4.4 安全访问：`at()` 方法

除了 `operator[]`，还有一个值得了解的访问方法——`at()`：

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
#include <stdexcept>
using namespace std;

int main() {
    unordered_map<string, int> um = {{"Alice", 25}, {"Bob", 30}};
    
    // at()：如果 key 存在，返回对应的 value 引用
    cout << um.at("Alice") << endl;  // 25
    
    // at()：如果 key 不存在，抛出 std::out_of_range 异常
    try {
        cout << um.at("Zara") << endl;  // 抛异常！
    } catch (const out_of_range& e) {
        cout << "异常: " << e.what() << endl;
    }
    
    // 对比：
    // operator[] → key 不存在时自动插入（危险）
    // at()       → key 不存在时抛异常（安全但需要处理异常）
    // find()     → key 不存在时返回 end()（最灵活）
    
    return 0;
}
```

---

## 2.5 大小操作

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um;
    
    // empty()
    cout << "是否为空: " << boolalpha << um.empty() << endl;  // true
    
    um = {{"a", 1}, {"b", 2}, {"c", 3}};
    
    // size()
    cout << "键值对数量: " << um.size() << endl;  // 3
    
    // max_size()
    cout << "最大容量: " << um.max_size() << endl;
    
    // empty()
    cout << "是否为空: " << boolalpha << um.empty() << endl;  // false
    
    return 0;
}
```

---

## 2.6 哈希相关：桶、负载因子与自定义哈希

### 2.6.1 桶（Bucket）的概念

哈希表内部维护了一个**桶数组**，每个桶可以存储零个或多个元素。STL 提供了一系列方法来观察桶的状态。

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> um = {
        {"apple", 1}, {"banana", 2}, {"cherry", 3},
        {"date", 4}, {"elderberry", 5}, {"fig", 6}
    };
    
    // bucket_count()：当前桶的总数
    cout << "桶的总数: " << um.bucket_count() << endl;
    // 输出例如：7（由实现决定，通常是质数或2的幂）
    
    // bucket_size(n)：第 n 个桶中有多少个元素
    for (size_t i = 0; i < um.bucket_count(); ++i) {
        cout << "桶 " << i << " 中有 " << um.bucket_size(i) << " 个元素";
        if (um.bucket_size(i) > 0) {
            cout << " → ";
            // 遍历某个桶中的元素
            for (auto it = um.begin(i); it != um.end(i); ++it) {
                cout << "[" << it->first << ":" << it->second << "] ";
            }
        }
        cout << endl;
    }
    
    // bucket(key)：查询某个 key 在哪个桶中
    cout << "'apple' 在桶 " << um.bucket("apple") << " 中" << endl;
    cout << "'banana' 在桶 " << um.bucket("banana") << " 中" << endl;
    
    return 0;
}
```

### 2.6.2 负载因子（Load Factor）

**负载因子 = 元素个数 / 桶的数量**

负载因子越大，哈希冲突的概率越高，查找性能越差。当负载因子超过阈值时，哈希表会自动**扩容（rehash）**。

```cpp
#include <iostream>
#include <unordered_map>
using namespace std;

int main() {
    unordered_map<int, int> um;
    
    // 插入之前
    cout << "=== 初始状态 ===" << endl;
    cout << "桶数: " << um.bucket_count() << endl;
    cout << "元素数: " << um.size() << endl;
    cout << "负载因子: " << um.load_factor() << endl;
    cout << "最大负载因子: " << um.max_load_factor() << endl;  // 默认 1.0
    
    // 插入 100 个元素
    for (int i = 0; i < 100; ++i) {
        um[i] = i * 10;
    }
    
    cout << "\n=== 插入 100 个元素后 ===" << endl;
    cout << "桶数: " << um.bucket_count() << endl;
    // 桶数会自动增长，以保持负载因子不超过 max_load_factor
    cout << "元素数: " << um.size() << endl;           // 100
    cout << "负载因子: " << um.load_factor() << endl;  // ≤ 1.0
    
    // 手动设置最大负载因子
    um.max_load_factor(0.5);  // 设置为 0.5（更严格），后续插入更容易触发 rehash
    cout << "\n=== 设置 max_load_factor = 0.5 后 ===" << endl;
    cout << "桶数: " << um.bucket_count() << endl;     // 可能暂时不变（实现相关）
    cout << "负载因子: " << um.load_factor() << endl;
    
    // 手动 rehash：预分配桶数
    um.rehash(500);  // 至少分配 500 个桶
    cout << "\n=== rehash(500) 后 ===" << endl;
    cout << "桶数: " << um.bucket_count() << endl;     // ≥ 500
    
    // reserve：预分配能容纳指定元素数量的空间（更实用）
    um.reserve(1000);  // 预分配能容纳 1000 个元素的桶
    cout << "\n=== reserve(1000) 后 ===" << endl;
    cout << "桶数: " << um.bucket_count() << endl;
    
    return 0;
}
```

#### 相关方法一览

| 方法                 | 说明                                      |
| -------------------- | ----------------------------------------- |
| `bucket_count()`     | 当前桶的总数                              |
| `max_bucket_count()` | 最大桶数                                  |
| `bucket_size(n)`     | 第 n 个桶中的元素数                       |
| `bucket(key)`        | key 所在的桶编号                          |
| `load_factor()`      | 当前负载因子（`size() / bucket_count()`） |
| `max_load_factor()`  | 获取/设置最大负载因子（默认 1.0）         |
| `rehash(n)`          | 重新分配至少 n 个桶                       |
| `reserve(n)`         | 预分配能容纳 n 个元素的桶                 |

> 💡 **性能提示**：如果你预先知道要插入多少个元素，使用 `reserve()` 预分配空间，可以避免多次 rehash，显著提高性能。

### 2.6.3 自定义类型的哈希函数（进阶重点！）

STL 默认只为基本类型（`int`, `double`, `string` 等）提供了哈希函数。如果你想把**自定义类型**作为 `unordered_map` 或 `unordered_set` 的 key，你需要**自己定义哈希函数**和 **`operator==`**。

#### 为什么需要两个东西？

1. **哈希函数**：决定元素放到哪个桶中
2. **`operator==`**：当两个元素哈希到同一个桶时，用来判断它们是否真的相同

#### 方式一：特化 `std::hash`（推荐）

```cpp
#include <iostream>
#include <unordered_set>
#include <unordered_map>
#include <string>
using namespace std;

// 自定义类型
struct Student {
    string name;
    int id;
    
    // 必须提供 operator==
    bool operator==(const Student& other) const {
        return name == other.name && id == other.id;
    }
};

// 方式一：特化 std::hash
// 在 std 命名空间中为 Student 提供 hash 特化
namespace std {
    template<>
    struct hash<Student> {
        size_t operator()(const Student& s) const {
            // 组合多个字段的哈希值
            size_t h1 = hash<string>{}(s.name);
            size_t h2 = hash<int>{}(s.id);
            // 常用的哈希组合方式：异或 + 位移
            return h1 ^ (h2 << 1);
        }
    };
}

int main() {
    // 现在可以直接使用了！
    unordered_set<Student> students;
    students.insert({"Alice", 1001});
    students.insert({"Bob", 1002});
    students.insert({"Alice", 1001});  // 重复，不会插入
    
    cout << "学生数: " << students.size() << endl;  // 2
    
    unordered_map<Student, double> grades;
    grades[{"Alice", 1001}] = 95.5;
    grades[{"Bob", 1002}] = 88.0;
    
    return 0;
}
```

#### 方式二：自定义哈希函数对象（不污染 std 命名空间）

```cpp
#include <iostream>
#include <unordered_set>
#include <unordered_map>
#include <string>
using namespace std;

struct Point {
    int x, y;
    
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

// 定义一个哈希函数对象（仿函数）
struct PointHash {
    size_t operator()(const Point& p) const {
        size_t h1 = hash<int>{}(p.x);
        size_t h2 = hash<int>{}(p.y);
        return h1 ^ (h2 << 1);
    }
};

int main() {
    // 将哈希函数对象作为第二个（unordered_set）或第三个（unordered_map）模板参数
    unordered_set<Point, PointHash> points;
    points.insert({1, 2});
    points.insert({3, 4});
    points.insert({1, 2});  // 重复
    
    cout << "点的数量: " << points.size() << endl;  // 2
    
    unordered_map<Point, string, PointHash> labels;
    labels[{1, 2}] = "原点附近";
    labels[{3, 4}] = "第一象限";
    
    cout << "(1,2) 的标签: " << labels[{1, 2}] << endl;
    
    return 0;
}
```

#### 方式三：使用 Lambda 表达式（C++20 简化写法）

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

struct Point {
    int x, y;
    bool operator==(const Point&) const = default;  // C++20 自动生成
};

int main() {
    // C++20：lambda 作为哈希函数
    auto hash_fn = [](const Point& p) {
        return hash<int>{}(p.x) ^ (hash<int>{}(p.y) << 1);
    };
    
    // 需要传入桶数和哈希函数对象
    unordered_set<Point, decltype(hash_fn)> points(16, hash_fn);
    points.insert({1, 2});
    points.insert({3, 4});
    
    cout << "点的数量: " << points.size() << endl;  // 2
    
    return 0;
}
```

#### 好用的哈希函数组合技巧

当自定义类型有多个字段时，一个好的哈希函数应该将所有字段都纳入计算。以下是常用的组合方法：

```cpp
// 方法一：简单异或 + 位移（上面已经用过）
size_t hash_val = h1 ^ (h2 << 1);

// 方法二：Boost 风格的 hash_combine（更好的分布）
// 这是业界广泛使用的方案
template <typename T>
void hash_combine(size_t& seed, const T& val) {
    seed ^= hash<T>{}(val) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
}

// 使用示例
struct Student {
    string name;
    int id;
    int grade;
    
    bool operator==(const Student&) const = default;  // C++20
};

struct StudentHash {
    size_t operator()(const Student& s) const {
        size_t seed = 0;
        hash_combine(seed, s.name);
        hash_combine(seed, s.id);
        hash_combine(seed, s.grade);
        return seed;
    }
};
```

> 💡 **`0x9e3779b9`** 这个"魔数"是黄金分割比的 32 位近似值，能让哈希值分布更均匀，减少冲突。

---

# 三、综合对比与总结

## 3.1 四种无序容器一览

| 容器                 | 头文件            | key 唯一 | 有 value |
| -------------------- | ----------------- | -------- | -------- |
| `unordered_set`      | `<unordered_set>` | ✅        | ❌        |
| `unordered_multiset` | `<unordered_set>` | ❌        | ❌        |
| `unordered_map`      | `<unordered_map>` | ✅        | ✅        |
| `unordered_multimap` | `<unordered_map>` | ❌        | ✅        |

## 3.2 有序 vs 无序完整对比

| 维度         | 有序（`set`/`map`）           | 无序（`unordered_*`） |
| ------------ | ----------------------------- | --------------------- |
| 底层         | 红黑树                        | 哈希表                |
| 增删查复杂度 | O(log n)                      | **O(1)** 平均         |
| 有序遍历     | ✅                             | ❌                     |
| key 要求     | `operator<`                   | `hash` + `operator==` |
| 范围查询     | ✅ `lower_bound`/`upper_bound` | ❌ 不支持              |
| 内存         | 较小                          | 较大                  |

## 3.3 场景选择速查

| 场景              | 推荐            | 原因                           |
| ----------------- | --------------- | ------------------------------ |
| 统计频次          | `unordered_map` | O(1) + `operator[]` 自动初始化 |
| 判断是否存在      | `unordered_set` | O(1) 查找                      |
| 需要有序/范围查询 | `map` / `set`   | 排序 + bound 查询              |
| 两数之和等配对题  | `unordered_map` | 快速查找互补元素               |
| 去重              | `unordered_set` | 自动去重且快                   |

## 3.4 常见陷阱汇总

> ⚠️ **陷阱 1**：`operator[]` 访问不存在的 key 会**自动插入默认值**
```cpp
unordered_map<string, int> um;
cout << um["ghost"];  // 输出 0，但同时偷偷插入了 {"ghost", 0}！
// 安全做法：用 find() 或 count() 或 contains() 先判断
```

> ⚠️ **陷阱 2**：`insert()` 在 key 已存在时**不覆盖**
```cpp
um.insert({"key", 1});
um.insert({"key", 2});  // 值仍是 1！
um["key"] = 2;          // 这样才会覆盖为 2
```

> ⚠️ **陷阱 3**：无序容器**不能用 `lower_bound` / `upper_bound`**
```cpp
// ❌ 编译错误！unordered_set 无此方法
// us.lower_bound(10);
// 需要范围查询，请改用 set / map
```

> ⚠️ **陷阱 4**：自定义类型做 key 时，**必须同时提供哈希函数和 `operator==`**
```cpp
// 缺少任何一个都会编译错误
```

## 3.5 快速参考表

### `unordered_set` 常用 API

| 操作 | 方法                                   | 返回值               |
| ---- | -------------------------------------- | -------------------- |
| 插入 | `insert(val)` / `emplace(args)`        | `pair<iter, bool>`   |
| 删除 | `erase(val)` / `erase(it)` / `clear()` | 个数 / 迭代器 / void |
| 查找 | `find(val)`                            | 迭代器               |
| 存在 | `count(val)` / `contains(val)` (C++20) | 0/1 / bool           |
| 大小 | `size()` / `empty()`                   | `size_t` / bool      |

### `unordered_map` 常用 API

| 操作      | 方法                                   | 返回值                              |
| --------- | -------------------------------------- | ----------------------------------- |
| 插入/修改 | `operator[](key) = val`                | value 引用                          |
| 仅插入    | `insert({k,v})` / `emplace(k,v)`       | `pair<iter, bool>`                  |
| 删除      | `erase(key)` / `erase(it)` / `clear()` | 个数 / 迭代器 / void                |
| 查找      | `find(key)`                            | 迭代器（`it->first`, `it->second`） |
| 安全访问  | `at(key)`                              | value  **引用** （不存在则抛异常）  |
| 存在      | `count(key)` / `contains(key)` (C++20) | 0/1 / bool                          |
| 大小      | `size()` / `empty()`                   | `size_t` / bool                     |
| 桶信息    | `bucket_count()` / `load_factor()`     | `size_t` / float                    |
| 预分配    | `reserve(n)` / `rehash(n)`             | void                                |

---

到这里，你已经具备了在实际题目中使用 `unordered_set` / `unordered_map` 的核心能力：

- 能根据“是否需要有序”在有序容器与无序容器之间做选择。
- 能正确使用插入、查找、删除与遍历 API，并避开 `operator[]` 的常见坑。
- 能理解负载因子与 `reserve()` 对性能的影响。
- 能为自定义类型补齐 `operator==` 和哈希函数。

建议在下一阶段配合典型题目（频次统计、去重、两数之和、分组）进行专项练习，加深理解。