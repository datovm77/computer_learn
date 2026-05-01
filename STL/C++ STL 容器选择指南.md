# 第十七阶段：容器选择指南（总回顾）

> 恭喜你！走到这一步，意味着你已经学完了 C++ STL 中所有主要的容器。本阶段不会引入新的容器，而是对你学过的所有容器做一次**系统性的总回顾**，帮你在头脑中建立一张完整的"容器地图"，让你在未来面对任何场景时，都能**快速、准确**地选出最合适的容器。

---

## 一、容器全景分类

在开始对比之前，先把所有容器按**底层数据结构**分成五个家族：

### 1.1 数组家族（连续内存）

|        容器        |      底层结构      |   头文件   | 一句话描述                          |
| :----------------: | :----------------: | :--------: | :---------------------------------- |
| `std::array<T, N>` | 固定大小的原生数组 | `<array>`  | 编译期确定大小，不可扩容的轻量封装  |
|  `std::vector<T>`  |   动态增长的数组   | `<vector>` | 最常用的容器，尾部增长，连续存储    |
|  `std::deque<T>`   |   分段连续缓冲区   | `<deque>`  | 双端都能高效插入/删除的"伪连续"数组 |

**共同特征**：支持**随机访问**（`operator[]`），元素在内存中（大致）连续排列，对 CPU 缓存非常友好。

### 1.2 链表家族（节点存储）

|          容器          | 底层结构 |      头文件      | 一句话描述                        |
| :--------------------: | :------: | :--------------: | :-------------------------------- |
|     `std::list<T>`     | 双向链表 |     `<list>`     | 任意位置 O(1) 插入/删除，双向遍历 |
| `std::forward_list<T>` | 单向链表 | `<forward_list>` | 最节省内存的链表，只能向前遍历    |

**共同特征**：**不支持随机访问**，每个元素是一个独立的堆节点（有指针开销），但在**任意已知位置**插入/删除是 O(1)。

### 1.3 有序关联家族（红黑树）

|         容器          | 底层结构 | 头文件  | 一句话描述                 |
| :-------------------: | :------: | :-----: | :------------------------- |
|     `std::set<T>`     |  红黑树  | `<set>` | 不重复、自动排序的键集合   |
|  `std::multiset<T>`   |  红黑树  | `<set>` | 允许重复、自动排序的键集合 |
|   `std::map<K, V>`    |  红黑树  | `<map>` | 不重复键的有序键值对       |
| `std::multimap<K, V>` |  红黑树  | `<map>` | 允许重复键的有序键值对     |

**共同特征**：元素始终**有序**，所有操作（插入、删除、查找）都是 **O(log n)**，需要键类型支持 `operator<`（或自定义比较器）。

### 1.4 无序关联家族（哈希表）

|              容器               | 底层结构 |      头文件       | 一句话描述             |
| :-----------------------------: | :------: | :---------------: | :--------------------- |
|     `std::unordered_set<T>`     |  哈希表  | `<unordered_set>` | 不重复、无序的键集合   |
|  `std::unordered_multiset<T>`   |  哈希表  | `<unordered_set>` | 允许重复、无序的键集合 |
|   `std::unordered_map<K, V>`    |  哈希表  | `<unordered_map>` | 不重复键的无序键值对   |
| `std::unordered_multimap<K, V>` |  哈希表  | `<unordered_map>` | 允许重复键的无序键值对 |

**共同特征**：元素**无序**，平均操作是 **O(1)**，最坏 O(n)；需要键类型提供 `std::hash` 特化和 `operator==`。

### 1.5 代码示例阅读约定（初学者必看）

为保证版面简洁，下面部分示例会省略 `std::` 前缀和 `#include`。如果你要直接复制到编译器，请先套用这个最小模板：

```cpp
#include <array>
#include <vector>
#include <deque>
#include <list>
#include <forward_list>
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>
#include <stack>
#include <queue>
#include <string>
#include <numeric>
#include <algorithm>
#include <functional>
#include <iostream>

using namespace std;
```

### 1.6 容器适配器家族

|          适配器          | 默认底层容器 |  头文件   | 一句话描述                   |
| :----------------------: | :----------: | :-------: | :--------------------------- |
|     `std::stack<T>`      |   `deque`    | `<stack>` | 后进先出（LIFO）             |
|     `std::queue<T>`      |   `deque`    | `<queue>` | 先进先出（FIFO）             |
| `std::priority_queue<T>` |   `vector`   | `<queue>` | 最大（或最小）元素始终在顶部 |

**共同特征**：它们**不是独立容器**，而是对已有容器的**功能限制封装**。你不能遍历它们，只能按特定顺序访问元素。

---

## 二、核心操作时间复杂度大对比

这是本阶段最重要的一张表。建议你**保存下来反复查阅**，直到形成肌肉记忆。

### 2.1 综合对比表

> 约定：n = 容器中元素个数。"摊销"表示大部分操作是该复杂度，偶尔一次较大开销（如 `vector` 扩容）。

| 操作               | `array` |  `vector`  |  `deque`   |  `list`  | `forward_list` | `set`/`map` | `unordered_set`/`map` |
| :----------------- | :-----: | :--------: | :--------: | :------: | :------------: | :---------: | :-------------------: |
| **随机访问 `[i]`** | ✅ O(1)  |   ✅ O(1)   |   ✅ O(1)   |  ❌ N/A   |     ❌ N/A      |    ❌ N/A    |         ❌ N/A         |
| **头部插入**       |  ❌ N/A  |   ⚠️ O(n)   | ✅ O(1)摊销 |  ✅ O(1)  |     ✅ O(1)     |      —      |           —           |
| **尾部插入**       |  ❌ N/A  | ✅ O(1)摊销 | ✅ O(1)摊销 |  ✅ O(1)  |    ⚠️ O(n)*     |      —      |           —           |
| **中间插入**       |  ❌ N/A  |   ⚠️ O(n)   |   ⚠️ O(n)   | ✅ O(1)** |    ✅ O(1)**    |      —      |           —           |
| **头部删除**       |  ❌ N/A  |   ⚠️ O(n)   | ✅ O(1)摊销 |  ✅ O(1)  |     ✅ O(1)     |      —      |           —           |
| **尾部删除**       |  ❌ N/A  |   ✅ O(1)   | ✅ O(1)摊销 |  ✅ O(1)  |    ⚠️ O(n)*     |      —      |           —           |
| **中间删除**       |  ❌ N/A  |   ⚠️ O(n)   |   ⚠️ O(n)   | ✅ O(1)** |    ✅ O(1)**    |      —      |           —           |
| **按值查找**       |  O(n)   |    O(n)    |    O(n)    |   O(n)   |      O(n)      |  O(log n)   |       O(1)平均        |
| **排序遍历**       | 需排序  |   需排序   |   需排序   |  需排序  |     需排序     | ✅ 天然有序  |        ❌ 无序         |

> `forward_list` 是单向链表，没有 `back()` 和 `push_back()`，尾部操作需要先遍历到最后。
>
> 链表的"中间插入 O(1)"有前提：**你必须已经持有指向插入位置的迭代器**。如果你需要先"找到位置"，那查找本身是 O(n)。
>
> 关联容器（`set/map/unordered_*`）没有“按位置中间插入”语义，只有按键插入/删除。

### 2.2 逐容器详解

#### 📦 `std::array<T, N>`

```
内存布局：[0][1][2][3][4]   ← 固定 N 个槽位，对象内连续
```

| 操作                   |   复杂度   | 说明               |
| :--------------------- | :--------: | :----------------- |
| `at(i)` / `operator[]` |    O(1)    | 直接计算偏移       |
| `front()` / `back()`   |    O(1)    | 首尾直接访问       |
| `size()`               |    O(1)    | 编译期常量         |
| 插入/删除              | **不支持** | 大小固定，不可增长 |

**核心特点**：
- 大小在**编译期确定**，不能增减。
- 存储在**栈**上（如果作为局部变量），访问速度极快。
- 是对原生 C 数组 `T arr[N]` 的安全封装，提供 `.size()`、`.at()`、迭代器等。
- **适用场景**：大小已知且不变的数据，比如一周 7 天、RGB 三个通道等。

#### 📦 `std::vector<T>`

```
内存布局：[0][1][2][3][4][_][_][_]
                              ↑ size=5, capacity=8
```

| 操作                    |    复杂度     | 说明                                      |
| :---------------------- | :-----------: | :---------------------------------------- |
| `operator[]`            |     O(1)      | 连续内存直接偏移                          |
| `push_back()`           | **O(1) 摊销** | 容量不足时扩容（通常翻倍），单次扩容 O(n) |
| `pop_back()`            |     O(1)      | 直接截断                                  |
| `insert(pos, val)`      |     O(n)      | 需要移动 pos 后面所有元素                 |
| `erase(pos)`            |     O(n)      | 需要前移 pos 后面所有元素                 |
| `find`（`<algorithm>`） |     O(n)      | 线性搜索                                  |

**核心特点**：
- **最常用的容器**，没有之一。当你不知道该用什么的时候，先试 `vector`。
- 连续内存 → 缓存命中率高 → 遍历极快。
- 缺点：头部和中间插入/删除代价大。
- `reserve()` 可以预分配容量，避免多次扩容。

**扩容机制详解**：

```cpp
vector<int> v;
// 假设初始 capacity = 0
v.push_back(1);  // capacity: 0 → 1，分配内存，拷贝 0 个元素
v.push_back(2);  // capacity: 1 → 2，分配新内存，拷贝 1 个元素
v.push_back(3);  // capacity: 2 → 4，分配新内存，拷贝 2 个元素
v.push_back(4);  // capacity 够用，直接放入
v.push_back(5);  // capacity: 4 → 8，分配新内存，拷贝 4 个元素
```

虽然单次扩容是 O(n)，但按倍增策略，**总的拷贝次数**不超过 2n，所以每次 `push_back` 的**摊销**复杂度是 O(1)。

#### 📦 `std::deque<T>`

```
内存布局（分段连续）：
  中控数组: [ptr0][ptr1][ptr2][ptr3]
               ↓     ↓     ↓     ↓
             [块0] [块1] [块2] [块3]
             ┌───┐ ┌───┐ ┌───┐ ┌───┐
             │a│b│ │c│d│ │e│f│ │g│h│
             └───┘ └───┘ └───┘ └───┘
```

| 操作               |  复杂度   | 说明                        |
| :----------------- | :-------: | :-------------------------- |
| `operator[]`       |   O(1)    | 通过中控数组 + 块内偏移定位 |
| `push_front()`     | O(1) 摊销 | 在块 0 的前面插入           |
| `push_back()`      | O(1) 摊销 | 在最后一个块的后面插入      |
| `insert(pos, val)` |   O(n)    | 需要移动元素                |
| `erase(pos)`       |   O(n)    | 需要移动元素                |

**核心特点**：

- **双端**高效操作，比 `vector` 多了高效的头部操作。
- **非完全连续**：所以指针算术不能跨块使用（虽然迭代器封装了这个细节）。
- 缓存性能略逊于 `vector`（内存不完全连续）。
- 是 `stack` 和 `queue` 的**默认底层容器**。

#### 📦 `std::list<T>`

```
内存布局（双向链表）：
  ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
  │  A  │⇄   │  B  │⇄  │  C  │⇄   │  D  │
  └─────┘    └─────┘    └─────┘    └─────┘
  每个节点：[prev指针 | 数据 | next指针]
```

| 操作                           |   复杂度   | 说明                                     |
| :----------------------------- | :--------: | :--------------------------------------- |
| `push_front()` / `push_back()` |    O(1)    | 修改指针即可                             |
| `insert(iter, val)`            |    O(1)    | 已有迭代器时，修改指针                   |
| `erase(iter)`                  |    O(1)    | 已有迭代器时，修改指针                   |
| `splice()`                     |    O(1)    | 独有操作！将另一个 list 的节点"嫁接"过来 |
| `sort()`                       | O(n log n) | 自带的归并排序（因为不能用 `std::sort`） |
| `operator[]`                   | **不支持** | 没有随机访问                             |
| 按值查找                       |    O(n)    | 线性遍历                                 |

**核心特点**：
- 任意位置插入/删除 O(1)（前提是有迭代器）。
- 插入与 `splice` 不会使其他元素迭代器失效；`erase` 只会使被删元素的迭代器失效。
- 每个节点有两个指针的额外开销 → 内存使用更多。
- 节点分散在堆上 → **缓存不友好**。
- 独有的 `splice()` 操作非常高效。

#### 📦 `std::forward_list<T>`

```
内存布局（单向链表）：
  ┌─────┐ → ┌─────┐ → ┌─────┐ → ┌─────┐ → null
  │  A  │   │  B  │   │  C  │   │  D  │
  └─────┘   └─────┘   └─────┘   └─────┘
  每个节点：[数据 | next指针]
```

| 操作                      |   复杂度   | 说明                           |
| :------------------------ | :--------: | :----------------------------- |
| `push_front()`            |    O(1)    | 头部插入                       |
| `insert_after(iter, val)` |    O(1)    | 在某节点**之后**插入           |
| `erase_after(iter)`       |    O(1)    | 删除某节点**之后**的节点       |
| `push_back()`             | **不支持** | 没有尾指针，需要 O(n) 找到尾部 |
| `size()`                  | **不提供** | 为了节省空间，不维护元素计数   |
| `operator[]`              | **不支持** | 没有随机访问                   |

**核心特点**：
- 比 `list` **节省一半指针开销**（每节点一个指针 vs 两个）。
- 所有操作都是 `*_after` 形式 → 只能操作"某节点的后继"。
- 连 `size()` 都没有！要么用 `std::distance(fl.begin(), fl.end())` 计数（但这是 O(n)）。
- **适用场景**：极端追求内存效率的链表场景。

#### 📦 `std::set<T>` / `std::map<K,V>`（及 multi 版本）

```
内存布局（红黑树，简化示意）：
           [  50  ]          ← 根节点
          /        \
      [ 30 ]      [ 70 ]
      /    \      /    \
   [20]  [40]   [60]  [80]
```

| 操作                              |  复杂度  | 说明                      |
| :-------------------------------- | :------: | :------------------------ |
| `insert()`                        | O(log n) | 沿树查找位置 + 红黑树旋转 |
| `erase()`                         | O(log n) | 摘除节点 + 红黑树旋转     |
| `find()` / `count()`              | O(log n) | 沿树二分查找              |
| `lower_bound()` / `upper_bound()` | O(log n) | 有序 → 支持范围查询       |
| `operator[]`（仅 map）            | O(log n) | 查找或插入                |
| 遍历（begin → end）               |   O(n)   | 中序遍历 → **有序输出**   |

**核心特点**：
- 所有操作稳定 O(log n)，**没有最坏退化**。
- 元素自动有序 → 天然支持范围查询（`lower_bound`、`upper_bound`）。
- 每个节点有左右子指针 + 颜色标志 → 内存开销较大。
- `set` vs `multiset`：是否允许重复键。
- `map` vs `multimap`：是否允许重复键。`map` 有 `operator[]`，`multimap` 没有。

#### 📦 `std::unordered_set<T>` / `std::unordered_map<K,V>`（及 multi 版本）

```
内存布局（哈希表，拉链法）：
  桶数组 (bucket array):
  [0] → nullptr
  [1] → { "cat", hash=1 } → { "dog", hash=1 } → nullptr
  [2] → { "fish", hash=2 } → nullptr
  [3] → nullptr
  [4] → { "bird", hash=4 } → nullptr
  ...
```

| 操作                   |     平均复杂度      | 最坏复杂度 | 说明               |
| :--------------------- | :-----------------: | :--------: | :----------------- |
| `insert()`             |        O(1)         |    O(n)    | 哈希冲突严重时退化 |
| `erase()`              |        O(1)         |    O(n)    | 同上               |
| `find()` / `count()`   |        O(1)         |    O(n)    | 同上               |
| `operator[]`（仅 map） |        O(1)         |    O(n)    | 同上               |
| 遍历                   | O(n + bucket_count) |     —      | 需要扫描所有桶     |

**核心特点**：
- 平均 O(1) 是**杀手级**优势，在大数据量下远快于红黑树。
- 但**不保证**元素有序 → 不支持范围查询。
- 哈希函数质量至关重要。内置类型（`int`, `string` 等）已有良好的默认哈希。
- 自定义类型需要提供 `std::hash` 特化和 `operator==`。
- 存在 rehash 操作（类似 vector 扩容），偶尔造成 O(n) 开销。

### 2.3 适配器复杂度

适配器的复杂度取决于底层容器，但因为它们限制了接口，所以核心操作都很高效：

| 适配器           | 核心操作                     |  复杂度  |  底层容器默认  |
| :--------------- | :--------------------------- | :------: | :------------: |
| `stack`          | `push()`, `pop()`, `top()`   |   O(1)   |    `deque`     |
| `queue`          | `push()`, `pop()`, `front()` |   O(1)   |    `deque`     |
| `priority_queue` | `push()`, `pop()`            | O(log n) | `vector`（堆） |
| `priority_queue` | `top()`                      |   O(1)   | `vector`（堆） |

---

## 三、内存布局与缓存性能

时间复杂度告诉你算法意义上的代价，但在现代 CPU 上，**缓存**的影响同样巨大。一个 O(n) 的连续内存遍历，可能比 O(n) 的链表遍历快 **10 倍以上**。

### 3.1 缓存友好度排名

```
缓存友好（最快）                             缓存不友好（较慢）
     ←──────────────────────────────────────────→
  array ≈ vector  >  deque  >>  set/map ≈ list ≈ unordered_*
  │               │         │
  │ 完全连续      │ 分段连续 │ 节点分散在堆上
```

**为什么这很重要？**

```cpp
// 同样是遍历 100 万个 int
vector<int> vec(1'000'000);    // 连续的 4MB 内存块
list<int>   lst;               // 100 万个独立节点，散布在堆上
for (int i = 0; i < 1'000'000; ++i) lst.push_back(i);

// 遍历 vec：CPU 预取器能准确预测下一个数据地址，几乎每次都命中缓存
// 遍历 lst：每个节点地址随机，几乎每次都缓存未命中 (cache miss)
// 实际测试：vec 的遍历速度通常是 lst 的 5~20 倍！
```

### 3.2 各容器内存开销估算

以存储 n 个 `int`（4 字节）为例：

| 容器                 |             每元素额外开销              | 总内存（约）  | 说明                             |
| :------------------- | :-------------------------------------: | :-----------: | :------------------------------- |
| `array<int, n>`      |                    0                    |      4n       | 纯数据，无额外开销               |
| `vector<int>`        |            ≈0（有少量预留）             |    4n ~ 8n    | capacity 可能大于 size           |
| `deque<int>`         |            少量（中控指针）             | ≈ 4n + 小常数 | 分段存储有少量管理开销           |
| `list<int>`          | **2 个指针**（16 字节/节点，64 位系统） |     ≈ 20n     | 数据 4 + prev 8 + next 8         |
| `forward_list<int>`  |       **1 个指针**（8 字节/节点）       |     ≈ 12n     | 数据 4 + next 8                  |
| `set<int>`           |  **3 个指针 + 颜色**（≈32 字节/节点）   |     ≈ 36n     | left/right/parent + color + data |
| `unordered_set<int>` |       **至少 1 个指针** + 桶数组        |  ≈ 12n + 桶   | 依赖实现                         |

> 💡 **启示**：如果你在存储大量小型数据（如 `int`、`char`），`list` 和 `set` 的内存开销可能是 `vector` 的 **5~9 倍**。
>
> ⚠️ 上表是用于建立“量级直觉”的估算值，不同标准库实现、对齐策略和分配器会有明显差异。

---

## 四、容器选择决策指南

这是本阶段的**精华**。当你面对一个具体场景时，按照下面的**决策流程**来选择容器：

### 4.1 决策流程图

```
开始
  │
  ├─ Q1: 你需要键值对（key-value）吗？
  │     │
  │     ├─ 是 ──→ Q2: 需要有序吗？
  │     │         │
  │     │         ├─ 是 ──→ 允许重复键？
  │     │         │         ├─ 是 → multimap
  │     │         │         └─ 否 → map
  │     │         │
  │     │         └─ 否 ──→ 允许重复键？
  │     │                   ├─ 是 → unordered_multimap
  │     │                   └─ 否 → unordered_map ★最常用
  │     │
  │     └─ 否 ──→ Q3: 只需要判断"存在性"（集合语义）？
  │               │
  │               ├─ 是 ──→ 需要有序吗？
  │               │         ├─ 是 ──→ 允许重复？
  │               │         │         ├─ 是 → multiset
  │               │         │         └─ 否 → set
  │               │         │
  │               │         └─ 否 ──→ 允许重复？
  │               │                   ├─ 是 → unordered_multiset
  │               │                   └─ 否 → unordered_set
  │               │
  │               └─ 否 ──→ Q4: 你需要什么样的访问模式？
  │                         │
  │                         ├─ 大小编译期固定 → array
  │                         │
  │                         ├─ 只在尾部增删 → vector ★最常用
  │                         │
  │                         ├─ 头尾都要增删 → deque
  │                         │
  │                         ├─ 频繁在中间插删 → list
  │                         │  （且已有迭代器）
  │                         │
  │                         ├─ 极致内存节省 → forward_list
  │                         │     的链表
  │                         │
  │                         ├─ LIFO 行为 → stack
  │                         ├─ FIFO 行为 → queue
  │                         └─ 需要快速取最大/最小 → priority_queue
```

### 4.2 十大经典场景与推荐容器

#### 场景 1：存一组数据，可能增长，需要随机访问

```
✅ 推荐：vector
❌ 避免：list（无随机访问）、set（不按下标访问）
```

```cpp
vector<int> scores;
scores.push_back(95);
scores.push_back(87);
scores.push_back(92);
cout << scores[1];  // 87，O(1) 随机访问
```

**为什么选 `vector`**：连续内存、支持下标、尾部操作高效，是最通用的序列容器。

---

#### 场景 2：需要频繁在两端插入/删除

```
✅ 推荐：deque
❌ 避免：vector（头部插入 O(n)）
```

```cpp
deque<int> dq;
dq.push_front(1);   // O(1)
dq.push_back(2);    // O(1)
dq.pop_front();     // O(1)
dq.pop_back();      // O(1)
```

**为什么选 `deque`**：首尾都是 O(1)，还支持随机访问，`vector` 做不到高效的头部操作。

---

#### 场景 3：大量在中间插入/删除，且已有迭代器定位

```
✅ 推荐：list
❌ 避免：vector/deque（中间操作 O(n) 移动元素）
```

```cpp
list<int> lst = {1, 2, 3, 4, 5};
auto it = lst.begin();
advance(it, 2);      // 定位到第 3 个元素（这步是 O(n)）
lst.insert(it, 99);  // 在 3 前面插入 99 → {1, 2, 99, 3, 4, 5}
// insert 本身是 O(1)，只修改指针
```

**注意**：`list` 的优势只在**已持有迭代器**的情况下才明显。如果每次都要从头查找位置，那总代价还是 O(n)，此时 `vector` 因为缓存友好可能反而更快。

---

#### 场景 4：需要快速判断某元素是否存在

```
✅ 推荐：unordered_set（平均 O(1)）
✅ 备选：set（稳定 O(log n)，元素有序）
❌ 避免：vector + find（O(n)）
```

```cpp
unordered_set<string> dict = {"apple", "banana", "cherry"};
if (dict.count("banana")) {        // O(1) 平均
    cout << "找到了！" << endl;
}
```

---

#### 场景 5：需要键值映射，按键快速查找

```
✅ 推荐：unordered_map（平均 O(1) 查找）
✅ 备选：map（O(log n)，但有序）
```

```cpp
unordered_map<string, int> word_count;
word_count["hello"]++;      // O(1) 平均
word_count["world"]++;
cout << word_count["hello"]; // 1
```

---

#### 场景 6：需要有序遍历 + 范围查询

```
✅ 推荐：set 或 map
❌ 避免：unordered_* （无序，不支持范围查询）
```

```cpp
set<int> s = {50, 30, 70, 20, 40, 60, 80};

// 查询 [25, 65) 范围内的元素
auto lo = s.lower_bound(25);  // 指向 30
auto hi = s.lower_bound(65);  // 指向 70
for (auto it = lo; it != hi; ++it) {
    cout << *it << " ";       // 30 40 50 60
}
```

**这是有序容器的独家能力！** `unordered_set` 做不到。

---

#### 场景 7：LIFO 行为（后进先出）

```
✅ 推荐：stack
```

```cpp
stack<int> stk;
stk.push(1);
stk.push(2);
stk.push(3);
cout << stk.top();  // 3（最后压入的先出）
stk.pop();
cout << stk.top();  // 2
```

**典型应用**：表达式求值、括号匹配、DFS 回溯、撤销操作。

---

#### 场景 8：FIFO 行为（先进先出）

```
✅ 推荐：queue
```

```cpp
queue<int> q;
q.push(1);
q.push(2);
q.push(3);
cout << q.front();  // 1（最先入队的先出）
q.pop();
cout << q.front();  // 2
```

**典型应用**：BFS 广度优先搜索、任务调度、消息队列。

---

#### 场景 9：需要快速获取最大/最小元素

```
✅ 推荐：priority_queue
❌ 避免：每次 sort 整个 vector（O(n log n)）
```

```cpp
// 最大堆（默认）
priority_queue<int> max_pq;
max_pq.push(30);
max_pq.push(50);
max_pq.push(10);
cout << max_pq.top();  // 50

// 最小堆
priority_queue<int, vector<int>, greater<int>> min_pq;
min_pq.push(30);
min_pq.push(50);
min_pq.push(10);
cout << min_pq.top();  // 10
```

**典型应用**：Dijkstra 最短路、合并 K 个有序链表、数据流中的中位数、Top-K 问题。

---

#### 场景 10：大小固定，编译期已知

```
✅ 推荐：array
❌ 避免：vector（动态分配的额外开销不必要）
```

```cpp
array<int, 7> week_temps = {28, 30, 27, 31, 29, 26, 28};
int avg = accumulate(week_temps.begin(), week_temps.end(), 0) / week_temps.size();
```

---

## 五、常见误区与深度辨析

### 5.1 误区一："`list` 的中间插入比 `vector` 快"

> 这句话**不完全正确**。

真相分两层：

1. **算法复杂度上**：`list` 的 `insert(iter, val)` 确实是 O(1)（修改指针），而 `vector` 的 `insert` 是 O(n)（移动元素）。
2. **实际运行时间上**：对于小规模数据（n < 几千），`vector` 的连续内存带来的缓存优势往往能**碾压** `list` 的O(1) 理论优势。

```
经验法则：
- 小到中等规模数据下，`vector` 往往因为缓存友好而更快（即使有一部分中间插入）
- 数据规模很大且“已定位迭代器下的中间插删”占主要开销时，`list` 才可能体现优势
- 阈值高度依赖编译器、数据类型、CPU 和分配器；不确定时先用 `vector`，再用 profiling 验证
```

### 5.2 误区二："`unordered_map` 总是比 `map` 快"

> 这句话**通常正确，但有例外**。

| 情况                          |       `unordered_map`       |      `map`       |     谁更好      |
| :---------------------------- | :-------------------------: | :--------------: | :-------------: |
| n 很大，纯查找/插入           |         ✅ O(1) 平均         |     O(log n)     | `unordered_map` |
| n 很小（< 100）               | 哈希计算 + 桶结构的常数开销 |     简单比较     |   可能差不多    |
| 需要有序遍历                  |          ❌ 不支持           |    ✅ 天然有序    |      `map`      |
| 需要 `lower_bound` 等范围查询 |          ❌ 不支持           |    ✅ O(log n)    |      `map`      |
| 哈希冲突严重                  |        ⚠️ 退化到 O(n)        |  稳定 O(log n)   |      `map`      |
| 键类型没有好的哈希函数        |         需要自定义          | 只需 `operator<` |  `map` 更方便   |

### 5.3 误区三："适配器只能用默认容器"

> **错！** 适配器可以指定底层容器。

```cpp
// stack 默认用 deque，但你可以换成 vector
stack<int, vector<int>> stk;

// queue 默认用 deque，但你可以换成 list
queue<int, list<int>> q;

// priority_queue 默认用 vector（必须支持随机访问）
// 不能换成 list！因为堆操作需要随机访问
priority_queue<int, vector<int>, greater<int>> min_pq;
```

### 5.4 误区四："`deque` 就是两个 `vector` 拼在一起"

> **不是**。`deque` 的实现比这复杂得多。

`deque` 的底层是一个**中控数组**（map，注意不是 `std::map`），每个指针指向一个**固定大小的缓冲区**。当需要在前面或后面扩展时，分配新的缓冲区并更新中控数组。

这意味着：
- `deque` 的元素**不保证完全连续**（跨块时不连续）。
- 不能像 `vector` 那样用 `data()` 获取原始指针来做 C 风格操作。
- 但 `deque` 的迭代器**封装了**跨块逻辑，用起来和 `vector` 几乎一样。

### 5.5 误区五："`forward_list` 就是比 `list` 好因为更省内存"

> **省内存是真的，但牺牲了便利性。**

`forward_list` 的"残缺"清单：
- ❌ 没有 `push_back()`
- ❌ 没有 `size()`  
- ❌ 没有 `back()`
- ❌ 不能反向遍历
- ❌ 所有操作都是 `*_after` 形式（只能操作后继，不能操作前驱）

除非你的数据量大到每个节点省 8 字节（一个指针）真的很关键，否则直接用 `list`。

---

## 六、`set`/`map` vs `unordered_set`/`unordered_map` 深度对比

这是面试和实战中最常见的选择题，值得深入分析。

### 6.1 全面对比表

| 对比维度         | `set` / `map`                                 | `unordered_set` / `unordered_map` |
| :--------------- | :-------------------------------------------- | :-------------------------------- |
| **底层**         | 红黑树                                        | 哈希表                            |
| **插入**         | O(log n)                                      | O(1) 平均 / O(n) 最坏             |
| **删除**         | O(log n)                                      | O(1) 平均 / O(n) 最坏             |
| **查找**         | O(log n)                                      | O(1) 平均 / O(n) 最坏             |
| **有序性**       | ✅ 自动排序                                    | ❌ 无序                            |
| **范围查询**     | ✅ `lower_bound`, `upper_bound`, `equal_range` | ❌ 不支持                          |
| **键的要求**     | 需要 `operator<`（或比较器）                  | 需要 `std::hash` + `operator==`   |
| **迭代器稳定性** | 插入/删除不影响其他迭代器                     | rehash 会使**所有**迭代器失效     |
| **最坏情况**     | O(log n) **保证**                             | O(n)（哈希冲突）                  |
| **内存**         | 每节点 ~32 字节（64 位系统）                  | 依赖负载因子和桶数                |
| **适合场景**     | 需要有序、范围查询、稳定性能                  | 纯查找/插入、追求最大速度         |

### 6.2 选择建议的简化规则

```
默认可优先考虑 unordered 版本（平均更快），除非你需要以下任一功能：
  1. 有序遍历
  2. lower_bound / upper_bound 范围查询
  3. 保证最坏 O(log n)
  4. 键类型没有好用的哈希函数
```

---

## 七、实战场景综合分析

下面用几个综合场景，展示完整的**选择思考过程**：

### 场景 A：高频交易系统的订单簿

> 需求：按价格排序维护买单和卖单，支持快速插入新订单、按价格范围查询、删除已成交的订单。

```
分析：
✅ 需要按价格排序 → 有序容器
✅ 需要范围查询（某个价格区间的所有订单） → lower_bound/upper_bound
✅ 同一价格可能有多个订单 → 允许重复键
✅ 频繁插入/删除 → O(log n)

→ 选择：multimap<double, Order>
```

```cpp
multimap<double, Order> buy_orders;   // 买单，按价格升序
multimap<double, Order> sell_orders;  // 卖单，按价格升序

// 查询价格 >= 100 的所有买单
auto it = buy_orders.lower_bound(100.0);
for (; it != buy_orders.end(); ++it) {
    // 处理订单
}
```

### 场景 B：社交网络的好友推荐

> 需求：给用户 A 推荐"朋友的朋友"。需要快速查询某用户的好友列表，判断两人是否已经是好友。

```
分析：
✅ 需要存储"用户 → 好友集合"的映射 → map 类
✅ 需要快速判断存在性（是否已是好友） → set 类
✅ 不需要有序，追求速度 → unordered

→ 选择：unordered_map<UserId, unordered_set<UserId>>
```

```cpp
unordered_map<int, unordered_set<int>> friends;

// 添加双向好友关系
friends[userA].insert(userB);
friends[userB].insert(userA);

// 推荐"朋友的朋友"
unordered_set<int> recommendations;
for (int f : friends[userA]) {              // A 的每个好友 f
    for (int ff : friends[f]) {             // f 的每个好友 ff
        if (ff != userA && !friends[userA].count(ff)) {
            recommendations.insert(ff);     // ff 不是 A 且不是 A 的现有好友
        }
    }
}
```

### 场景 C：文本编辑器的撤销/重做

> 需求：记录用户操作历史，支持撤销（回到上一步）和重做（前进一步）。

```
分析：
✅ 撤销 = 后进先出 → LIFO 语义
✅ 重做 = 另一个栈
✅ 只需要访问栈顶

→ 选择：两个 stack<Operation>
```

```cpp
stack<Operation> undo_stack;
stack<Operation> redo_stack;

void do_operation(Operation op) {
    apply(op);
    undo_stack.push(op);
    // 新操作后清空 redo 栈
    redo_stack = stack<Operation>();
}

void undo() {
    if (undo_stack.empty()) return;
    auto op = undo_stack.top();
    undo_stack.pop();
    reverse_apply(op);
    redo_stack.push(op);
}

void redo() {
    if (redo_stack.empty()) return;
    auto op = redo_stack.top();
    redo_stack.pop();
    apply(op);
    undo_stack.push(op);
}
```

### 场景 D：合并 K 个有序数组

> 需求：有 K 个已排序的数组，合并成一个有序数组。

```
分析：
✅ 需要每次从 K 个数组的"当前最小值"中选最小
✅ 选出后更新该数组的"当前指针"
✅ 每次取最小 → 最小堆

→ 选择：priority_queue（最小堆）
```

```cpp
struct Element {
    int val;
    int array_idx;
    int element_idx;
    bool operator>(const Element& other) const { return val > other.val; }
};

vector<int> merge_k_sorted(vector<vector<int>>& arrays) {
    priority_queue<Element, vector<Element>, greater<Element>> min_heap;
    
    // 初始化：每个数组的第一个元素入堆
    for (int i = 0; i < arrays.size(); ++i) {
        if (!arrays[i].empty()) {
            min_heap.push({arrays[i][0], i, 0});
        }
    }
    
    vector<int> result;
    while (!min_heap.empty()) {
        auto [val, arr_i, elem_i] = min_heap.top();
        min_heap.pop();
        result.push_back(val);
        
        // 该数组的下一个元素入堆
        if (elem_i + 1 < arrays[arr_i].size()) {
            min_heap.push({arrays[arr_i][elem_i + 1], arr_i, elem_i + 1});
        }
    }
    return result;
}
```

---

## 八、速查秘诀：一句话选容器

当你需要快速做出选择时，记住这些**一句话规则**：

| 如果你需要…                     | 用…                               |
| :------------------------------ | :-------------------------------- |
| **默认通用容器**                | `vector`                          |
| 大小编译期确定                  | `array`                           |
| 双端高效操作 + 随机访问         | `deque`                           |
| 频繁在中间插入/删除（有迭代器） | `list`                            |
| 自动排序 + 范围查询             | `set` / `map`                     |
| 最快的查找速度                  | `unordered_set` / `unordered_map` |
| LIFO（后进先出）                | `stack`                           |
| FIFO（先进先出）                | `queue`                           |
| 快速取最大/最小                 | `priority_queue`                  |
| 计数某元素出现次数              | `unordered_map<T, int>`           |
| 有序 + 允许重复                 | `multiset` / `multimap`           |
| 链表 + 极致省内存               | `forward_list`                    |

---

## 九、终极对比总结表

最后，把所有容器按**你最关心的维度**做一次总汇：

| 容器             | 随机访问 | 有序  | 唯一键 | 头插  |   尾插   | 中间插 |   查找   | 迭代器类型 |
| :--------------- | :------: | :---: | :----: | :---: | :------: | :----: | :------: | :--------: |
| `array`          |  ✅ O(1)  |   —   |   —    |   ❌   |    ❌     |   ❌    |   O(n)   |    随机    |
| `vector`         |  ✅ O(1)  |   —   |   —    | O(n)  |  O(1)*   |  O(n)  |   O(n)   |    随机    |
| `deque`          |  ✅ O(1)  |   —   |   —    | O(1)* |  O(1)*   |  O(n)  |   O(n)   |    随机    |
| `list`           |    ❌     |   —   |   —    | O(1)  |   O(1)   | O(1)†  |   O(n)   |    双向    |
| `forward_list`   |    ❌     |   —   |   —    | O(1)  |    ❌     | O(1)†  |   O(n)   |    前向    |
| `set`            |    ❌     |   ✅   |   ✅    |   —   |    —     |   —    | O(log n) |    双向    |
| `multiset`       |    ❌     |   ✅   |   ❌    |   —   |    —     |   —    | O(log n) |    双向    |
| `map`            |    ❌     |   ✅   |   ✅    |   —   |    —     |   —    | O(log n) |    双向    |
| `multimap`       |    ❌     |   ✅   |   ❌    |   —   |    —     |   —    | O(log n) |    双向    |
| `unordered_set`  |    ❌     |   ❌   |   ✅    |   —   |    —     |   —    |  O(1)‡   |    前向    |
| `unordered_map`  |    ❌     |   ❌   |   ✅    |   —   |    —     |   —    |  O(1)‡   |    前向    |
| `stack`          |    ❌     |   —   |   —    |   —   |   O(1)   |   ❌    |    ❌     |     无     |
| `queue`          |    ❌     |   —   |   —    |   —   |   O(1)   |   ❌    |    ❌     |     无     |
| `priority_queue` |    ❌     | 部分§ |   —    |   —   | O(log n) |   ❌    |    ❌     |     无     |

> **\*** = 摊销 O(1)
> **†** = 需要已有迭代器
> **‡** = 平均，最坏 O(n)
> **§** = `priority_queue` 只保证堆顶是最大/最小值，不是完全有序

---

## 十、总结：三条黄金法则

经过这一阶段的学习，请把以下三条法则刻在脑海里：

### 法则一：**默认用 `vector`**

> 除非你有明确的理由不用它。`vector` 的连续内存、随机访问和缓存友好性使它在大多数场景下是最优选择。"先 `vector`，后优化"是 C++ 社区的共识。

### 法则二：**需要查找用 `unordered`，需要排序用 `ordered`**

> 如果你的核心操作是"某个键在不在"或"某个键对应什么值"，首选 `unordered_set` / `unordered_map`。如果你还需要"按键排序遍历"或"查找最近的键"，那选 `set` / `map`。

### 法则三：**不要过早优化容器选择**

> 写代码时先选最直觉的容器（通常是 `vector` 或 `unordered_map`），让程序**先跑起来**。然后用 profiling 工具找到真正的性能瓶颈。很多时候你以为的瓶颈根本不在容器选择上——算法的选择比容器的选择重要 10 倍。

---

🎉 **恭喜你完成了 STL 容器的全部学习！** 从 `array` 到 `priority_queue`，从连续内存到红黑树再到哈希表，你已经掌握了 C++ STL 容器家族的全貌。未来你还会学到 STL 的**算法库**（`<algorithm>`）和**迭代器适配器**，它们和容器配合使用，能让你的代码更加简洁高效。