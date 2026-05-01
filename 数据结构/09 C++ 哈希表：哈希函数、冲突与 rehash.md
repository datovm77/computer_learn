# C++ 哈希表完全教程：从直觉到实战

---

## 阶段一：建立直觉

### 1.1 为什么 `std::vector` 线性查找不够用？

假设你有一个通讯录，存了 10 万个联系人的姓名和电话号码：

```cpp
#include <vector>
#include <string>
#include <iostream>

struct Contact {
    std::string name;
    std::string phone;
};

int main() {
    std::vector<Contact> contacts;
    // 假设已填充 100,000 条记录...
    
    // 查找 "张三" 的电话号码
    std::string target = "张三";
    for (const auto& c : contacts) {   // 线性查找 O(n)
        if (c.name == target) {
            std::cout << c.phone << "\n";
            break;
        }
    }
}
```

**问题在哪里？**

| 操作         | `std::vector` 线性查找      | 理想目标            |
| ------------ | --------------------------- | ------------------- |
| 查找一个元素 | **O(n)** — 最坏遍历整个容器 | **O(1)** — 一步到位 |
| 查找 m 次    | **O(m × n)**                | **O(m)**            |

当 `n = 100,000`、`m = 100,000` 时：
- 线性查找：**100 亿次** 比较 → 慢到不可接受
- O(1) 查找：**10 万次** 操作 → 瞬间完成

> **核心矛盾**：`vector` 是连续内存，擅长按**下标**访问（`v[42]` 是 O(1)），但按**值**查找只能逐个比较。
>
> **关键洞察**：如果我们能把 "张三" 这个名字**直接变成一个数组下标**，就能像 `v[42]` 一样一步到位！

这就是**哈希表（Hash Table）**的核心思想。

---

### 1.2 哈希函数：把任意 key 映射成数组下标

**哈希函数（Hash Function）** 是一个函数 `h(key) → index`，它把任意类型的键转换成一个整数下标。

#### 最简单的例子：整数取余

```cpp
// 假设我们有一个大小为 10 的数组（桶数组）
constexpr int BUCKET_COUNT = 10;

int hash_func(int key) {
    return key % BUCKET_COUNT;  // 取余，结果在 [0, 9]
}
```

运算过程：

```
hash_func(42)  = 42 % 10  = 2   → 放入桶 2
hash_func(17)  = 17 % 10  = 7   → 放入桶 7
hash_func(102) = 102 % 10 = 2   → 也是桶 2！（冲突！后面会讲）
hash_func(0)   = 0 % 10   = 0   → 放入桶 0
```

#### 字符串的哈希函数

对于字符串，常用的方法是把每个字符的 ASCII 值加权求和：

```cpp
// 经典的多项式滚动哈希
std::size_t string_hash(const std::string& s) {
    std::size_t hash = 0;
    for (char c : s) {
        hash = hash * 31 + static_cast<unsigned char>(c);  
        // 31 是常用的质数乘子
    }
    return hash;
}

// 使用时：
// index = string_hash("张三") % BUCKET_COUNT;
```

**为什么选 31？**
- 31 是质数，能让哈希值分布更均匀（减少冲突）
- `x * 31` 可以被编译器优化为 `(x << 5) - x`，速度快
- Java 的 `String.hashCode()` 也用的 31

#### 好的哈希函数应满足的性质

| 性质       | 含义                                     |
| ---------- | ---------------------------------------- |
| **确定性** | 同一个 key 永远返回同一个值              |
| **均匀性** | 不同的 key 尽可能均匀地散布到各个桶      |
| **高效性** | 计算速度要快（否则就失去了 O(1) 的意义） |

---

### 1.3 用原生数组模拟你的第一个桶

让我们从最简单的场景开始：key 是 `int`，value 是 `std::string`。

```cpp
#include <iostream>
#include <string>

constexpr int BUCKET_COUNT = 10;

// 每个桶存一个键值对
struct Bucket {
    int key;
    std::string value;
    bool occupied = false;  // 该桶是否有数据
};

Bucket table[BUCKET_COUNT];  // 桶数组

// 插入
void put(int key, const std::string& value) {
    int idx = key % BUCKET_COUNT;
    table[idx].key = key;
    table[idx].value = value;
    table[idx].occupied = true;
}

// 查找
std::string get(int key) {
    int idx = key % BUCKET_COUNT;
    if (table[idx].occupied && table[idx].key == key) {
        return table[idx].value;
    }
    return "(未找到)";
}

int main() {
    put(42, "Alice");
    put(17, "Bob");
    put(5,  "Charlie");

    std::cout << "key=42 → " << get(42) << "\n";  // Alice
    std::cout << "key=17 → " << get(17) << "\n";  // Bob
    std::cout << "key=5  → " << get(5)  << "\n";  // Charlie
    std::cout << "key=99 → " << get(99) << "\n";  // (未找到)

    // 但是！如果插入 key=52 呢？
    put(52, "Dave");
    std::cout << "key=42 → " << get(42) << "\n";  // 被覆盖了！输出 Dave 而不是 Alice
    // 因为 42 % 10 == 52 % 10 == 2 → 哈希冲突！
}
```

**这个"玩具版"暴露了致命问题**：
1. 不能处理**哈希冲突**（两个不同的 key 映射到同一个桶）
2. 桶的数量是固定的，装满了怎么办？
3. 不支持删除

这些问题，正是阶段二和阶段三要解决的。

---

## 阶段二：核心机制

### 2.1 哈希冲突是什么，为什么不可避免

**哈希冲突（Hash Collision）** ：两个不同的 key 经过哈希函数后得到相同的下标。

```
h("Alice") % 10 = 3
h("Frank") % 10 = 3   ← 冲突！
```

#### 为什么不可避免？——鸽巢原理

> **鸽巢原理（Pigeonhole Principle）**：如果有 n 只鸽子要飞进 m 个鸽巢，且 n > m，那么至少有一个鸽巢里有超过一只鸽子。

- 可能的 key 的数量：**无穷多**（比如所有可能的字符串）
- 桶的数量：**有限的**（比如 10、1000、100万）
- 所以无论哈希函数设计得多好，冲突在数学上**不可避免**

#### 实际影响

即使桶的数量很大，冲突也比你直觉中来得更早。根据**生日悖论**：
- 一个有 365 个桶的哈希表，只要放入 **23 个元素**，就有 > 50% 的概率出现冲突
- 放入 **70 个元素**时，冲突概率 > 99.9%

所以，解决冲突不是"可选的优化"，而是**必须实现的核心机制**。

---

### 2.2 拉链法：用 `std::list` 挂在桶上

**拉链法（Separate Chaining）** 是最经典的冲突解决方案：每个桶不止存一个元素，而是**挂一条链表**。

```
桶数组:
[0] → ∅
[1] → (11, "Alice") → (21, "Eve") → ∅
[2] → (42, "Bob") → (52, "Dave") → (102, "Frank") → ∅
[3] → (33, "Charlie") → ∅
...
[9] → ∅
```

#### 完整实现

```cpp
#include <iostream>
#include <string>
#include <list>
#include <utility>  // std::pair

constexpr int BUCKET_COUNT = 10;

// 每个桶是一条链表，链表里存 <key, value> 对
std::list<std::pair<int, std::string>> table[BUCKET_COUNT];

int hash_func(int key) {
    // 注意处理负数：C++ 的 % 对负数结果可能为负
    return ((key % BUCKET_COUNT) + BUCKET_COUNT) % BUCKET_COUNT;
}

// 插入（如果 key 已存在则更新）
void put(int key, const std::string& value) {
    int idx = hash_func(key);
    // 遍历链表，看 key 是否已经存在
    for (auto& [k, v] : table[idx]) {
        if (k == key) {
            v = value;  // key 已存在，更新 value
            return;
        }
    }
    // key 不存在，追加到链表尾部
    table[idx].emplace_back(key, value);
}

// 查找
std::string* get(int key) {
    int idx = hash_func(key);
    for (auto& [k, v] : table[idx]) {
        if (k == key) {
            return &v;  // 返回指向 value 的指针
        }
    }
    return nullptr;  // 未找到
}

// 删除
bool erase(int key) {
    int idx = hash_func(key);
    auto& chain = table[idx];
    for (auto it = chain.begin(); it != chain.end(); ++it) {
        if (it->first == key) {
            chain.erase(it);
            return true;
        }
    }
    return false;
}

int main() {
    put(42,  "Alice");
    put(52,  "Dave");   // 42 和 52 都映射到桶 2，形成链表
    put(102, "Frank");  // 也映射到桶 2

    if (auto* val = get(42))
        std::cout << "42 → " << *val << "\n";   // Alice ✓
    if (auto* val = get(52))
        std::cout << "52 → " << *val << "\n";   // Dave ✓
    if (auto* val = get(102))
        std::cout << "102 → " << *val << "\n";  // Frank ✓

    erase(52);
    std::cout << "删除 52 后: " << (get(52) ? *get(52) : "未找到") << "\n";
}
```

#### 时间复杂度分析

| 场景                         | 查找/插入/删除                    |
| ---------------------------- | --------------------------------- |
| 理想情况（均匀分布）         | O(1 + n/m)，其中 n=元素数，m=桶数 |
| 最坏情况（所有元素在同一桶） | O(n) — 退化为链表                 |

关键参数 `n/m` 就是**装载因子（Load Factor）** ，阶段三会详细讲。

#### 拉链法的优点

1. **实现简单**：桶里挂链表，逻辑清晰
2. **删除容易**：链表删除一个节点是 O(1)
3. **不怕装满**：链表可以无限延长（只是性能下降）

#### 拉链法的缺点

1. **额外内存**：每个节点需要额外的指针（链表的 `next`）
2. **缓存不友好**：链表节点散布在内存各处，CPU 缓存命中率低

---

### 2.3 开放地址法：线性探测 vs 二次探测

**开放地址法（Open Addressing）** 的思路完全不同：所有元素都直接存在桶数组里，如果发生冲突，就按照某种**探测序列**去找下一个空桶。

#### 2.3.1 线性探测（Linear Probing）

冲突时，依次检查下一个桶：`(h(key) + 1)`, `(h(key) + 2)`, `(h(key) + 3)`, ...

```cpp
#include <iostream>
#include <string>
#include <optional>

constexpr int BUCKET_COUNT = 16;  // 建议用 2 的幂

enum class State { EMPTY, OCCUPIED, DELETED };

struct Bucket {
    int key{};
    std::string value{};
    State state = State::EMPTY;
};

Bucket table[BUCKET_COUNT];

int hash_func(int key) {
    return ((key % BUCKET_COUNT) + BUCKET_COUNT) % BUCKET_COUNT;
}

// 插入
bool put(int key, const std::string& value) {
    int idx = hash_func(key);
    int first_deleted = -1;  // 记录第一个 DELETED 位置

    for (int i = 0; i < BUCKET_COUNT; ++i) {
        int probe = (idx + i) % BUCKET_COUNT;

        if (table[probe].state == State::EMPTY) {
            // 找到空桶
            int target = (first_deleted != -1) ? first_deleted : probe;
            table[target].key = key;
            table[target].value = value;
            table[target].state = State::OCCUPIED;
            return true;
        }
        if (table[probe].state == State::DELETED && first_deleted == -1) {
            first_deleted = probe;  // 记住第一个被删除的位置，可以复用
        }
        if (table[probe].state == State::OCCUPIED && table[probe].key == key) {
            table[probe].value = value;  // key 已存在，更新
            return true;
        }
    }
    return false;  // 表满了
}

// 查找
std::string* get(int key) {
    int idx = hash_func(key);
    for (int i = 0; i < BUCKET_COUNT; ++i) {
        int probe = (idx + i) % BUCKET_COUNT;

        if (table[probe].state == State::EMPTY) {
            return nullptr;  // 遇到 EMPTY，说明 key 不存在
        }
        if (table[probe].state == State::OCCUPIED && table[probe].key == key) {
            return &table[probe].value;
        }
        // 如果是 DELETED，继续探测（不能停！）
    }
    return nullptr;
}

// 删除（懒删除：标记为 DELETED）
bool erase(int key) {
    int idx = hash_func(key);
    for (int i = 0; i < BUCKET_COUNT; ++i) {
        int probe = (idx + i) % BUCKET_COUNT;

        if (table[probe].state == State::EMPTY) {
            return false;  // key 不存在
        }
        if (table[probe].state == State::OCCUPIED && table[probe].key == key) {
            table[probe].state = State::DELETED;  // 不能设为 EMPTY！
            return true;
        }
    }
    return false;
}

int main() {
    put(2,  "Alice");
    put(18, "Bob");     // 18 % 16 = 2，冲突！探测到桶 3
    put(34, "Charlie"); // 34 % 16 = 2，冲突！探测到桶 4

    std::cout << "2  → " << *get(2) << "\n";   // Alice
    std::cout << "18 → " << *get(18) << "\n";  // Bob
    std::cout << "34 → " << *get(34) << "\n";  // Charlie

    // 删除 18
    erase(18);
    // 34 还能找到吗？
    std::cout << "34 → " << *get(34) << "\n";  // Charlie ✓
    // 因为桶 3 被标记为 DELETED 而不是 EMPTY，探测不会中断
}
```

**为什么删除时不能直接设为 EMPTY？**

这是线性探测中最容易犯的错误：

```
桶 2: (2, "Alice")   — OCCUPIED
桶 3: (18, "Bob")    — OCCUPIED  
桶 4: (34, "Charlie") — OCCUPIED

如果删除 key=18 时把桶 3 设为 EMPTY:
桶 2: (2, "Alice")   — OCCUPIED
桶 3:                 — EMPTY     ← 中断了！
桶 4: (34, "Charlie") — OCCUPIED

查找 key=34: h(34)=2 → 桶2(不是) → 桶3(EMPTY) → 结论：不存在 ✗ 错误！
```

所以必须用 `DELETED` 状态（也叫**墓碑标记 Tombstone**），告诉探测链"这里曾经有人，继续往后找"。

#### 2.3.2 二次探测（Quadratic Probing）

线性探测有一个严重问题——**聚集效应（Primary Clustering）**：

```
冲突的元素倾向于连续堆积：
[_][_][A][B][C][D][E][_][_][_]
              ↑ 新元素也会落入这个连续区域，加剧拥堵
```

二次探测用**平方序列**代替步长 1：`(h(key) + 1²)`, `(h(key) + 2²)`, `(h(key) + 3²)`, ...

```cpp
// 线性探测的探测序列：
// probe = (idx + i) % BUCKET_COUNT;     // i = 0, 1, 2, 3, 4, ...

// 二次探测的探测序列：
// probe = (idx + i * i) % BUCKET_COUNT; // i = 0, 1, 4, 9, 16, ...

// 更常用的变体（保证能遍历所有桶，前提是 BUCKET_COUNT 是 2 的幂）：
// probe = (idx + (i + i*i) / 2) % BUCKET_COUNT;  
// 即步长为 0, 1, 3, 6, 10, ... (三角数序列)

int quadratic_probe(int idx, int i) {
    // 常用变体：交替探测
    return (idx + i * i) % BUCKET_COUNT;
}
```

**注意**：如果用简单的 `i²`，且 `BUCKET_COUNT` 不是质数或 2 的幂，可能无法遍历所有桶。推荐：
- `BUCKET_COUNT` 为**质数**时，装载因子 ≤ 0.5 可保证找到空桶
- `BUCKET_COUNT` 为 **2 的幂**时，用 `(i + i²) / 2` 三角数序列

#### 2.3.3 双重哈希（Double Hashing）简介

另一种高级探测方式，用**第二个哈希函数**决定步长：

```cpp
// h1(key) = key % m
// h2(key) = 1 + (key % (m - 1))   // 确保步长 ≥ 1
// probe_i = (h1(key) + i * h2(key)) % m

int double_hash_probe(int key, int i) {
    int h1 = key % BUCKET_COUNT;
    int h2 = 1 + (key % (BUCKET_COUNT - 1));
    return (h1 + i * h2) % BUCKET_COUNT;
}
```

不同 key 的步长不同，所以几乎不会形成聚集。

---

### 2.4 拉链法 vs 开放地址法：性能与内存的取舍

| 维度           | 拉链法 (Separate Chaining)    | 开放地址法 (Open Addressing) |
| -------------- | ----------------------------- | ---------------------------- |
| **内存布局**   | 桶数组 + 链表节点（散落各处） | 所有数据在同一个数组中       |
| **缓存友好性** | ❌ 差（链表指针跳转）          | ✅ 好（数组连续内存）         |
| **删除操作**   | ✅ 简单（链表删除）            | ⚠️ 需要墓碑标记               |
| **装载因子**   | 可以 > 1（链表无限长）        | **必须 < 1**（否则表满）     |
| **额外内存**   | 每个节点需要指针开销          | 可能有大量空桶浪费           |
| **实现复杂度** | 较简单                        | 较复杂（探测 + 墓碑）        |
| **最坏性能**   | O(n)（长链）                  | O(n)（长聚集）               |
| **适用场景**   | 频繁删除、装载因子不确定      | 数据量可预估、追求缓存性能   |

**标准库的选择**：
- **C++ `std::unordered_map`**：使用**拉链法**
- **Python `dict`**：使用**开放地址法**（紧凑字典）
- **Go `map`**：使用**拉链法**（桶内用小数组）

---

## 阶段三：动态特性

### 3.1 装载因子：`size / bucket_count` 超限怎么办

**装载因子（Load Factor）**：
```
α = 已存元素数量（size）/ 桶的数量（bucket_count）
```

装载因子越大，冲突越严重，性能越差：

| 装载因子 α | 拉链法平均查找 | 线性探测平均查找 |
| ---------- | -------------- | ---------------- |
| 0.25       | 1.25 次        | ~1.17 次         |
| 0.50       | 1.50 次        | ~1.50 次         |
| 0.75       | 1.75 次        | ~2.50 次         |
| 0.90       | 1.90 次        | ~5.50 次         |
| 1.00       | 2.00 次        | ∞（表满）        |

**C++ `std::unordered_map` 的默认策略**：
```cpp
#include <unordered_map>
#include <iostream>

int main() {
    std::unordered_map<int, int> m;
    
    std::cout << "最大装载因子: " << m.max_load_factor() << "\n";  // 默认 1.0
    
    // 你可以调整它：
    m.max_load_factor(0.75f);  // 设为 0.75，更积极地扩容
}
```

当 `size / bucket_count > max_load_factor()` 时，自动触发**扩容**。

---

### 3.2 扩容与 Rehashing：重新分配所有元素

扩容的过程（Rehashing）：

```
扩容前：bucket_count = 4
桶 0: [(8, "A")]
桶 1: [(1, "B"), (5, "C")]
桶 2: [(2, "D")]
桶 3: [(3, "E"), (7, "F")]

触发扩容！（示意中）bucket_count 从 4 增长到 8

扩容后：每个元素用 key % 8 重新定位
桶 0: [(8, "A")]        ← 8 % 8 = 0
桶 1: [(1, "B")]        ← 1 % 8 = 1
桶 2: [(2, "D")]        ← 2 % 8 = 2  
桶 3: [(3, "E")]        ← 3 % 8 = 3
桶 4: []
桶 5: [(5, "C")]        ← 5 % 8 = 5（从桶 1 搬出来了！）
桶 6: []
桶 7: [(7, "F")]        ← 7 % 8 = 7（从桶 3 搬出来了！）
```

**关键要点**（不同标准库实现的扩容细节可能不同，下面是原理示意）：
1. **不是简单追加桶**，而是创建一个全新的、更大的桶数组
2. **每个元素都要重新计算哈希值**并放入新位置——这就是 "Re-hashing"
3. Rehashing 的时间复杂度是 **O(n)**

让我们验证一下 C++ 标准库的行为：

```cpp
#include <unordered_map>
#include <iostream>

int main() {
    std::unordered_map<int, std::string> m;
    m.max_load_factor(1.0f);

    for (int i = 0; i < 20; ++i) {
        m[i] = "val_" + std::to_string(i);
        std::cout << "size=" << m.size()
                  << "  bucket_count=" << m.bucket_count()
                  << "  load_factor=" << m.load_factor()
                  << "\n";
    }
    // 你会看到 bucket_count 在某些时刻突然增长（具体增长策略由实现决定），例如：
    // size=1  bucket_count=1  load_factor=1
    // size=2  bucket_count=2  load_factor=1    ← 扩容！
    // size=3  bucket_count=4  load_factor=0.75 ← 扩容！
    // ...
}
```

#### 手动控制

```cpp
std::unordered_map<int, int> m;

// 预分配：如果你知道大约要存 1000 个元素
m.reserve(1000);
// reserve 会让 bucket_count = ceil(1000 / max_load_factor())
// 这样插入 1000 个元素期间不会触发 rehashing

// 手动 rehash
m.rehash(2048);  
// 将 bucket_count 设为至少 2048
```

---

### 3.3 均摊分析：为什么整体还是 O(1)

单次 rehashing 是 O(n)，看起来很吓人。但通过**均摊分析（Amortized Analysis）**，可以证明每次插入的平均代价仍然是 O(1)。

#### 直觉解释

假设每次扩容翻倍（桶数 × 2），从空表开始插入 n 个元素：

```
插入第 1 个元素:  无需扩容,  代价 1
插入第 2 个元素:  扩容！搬 1 个元素, 代价 1 + 1 = 2
插入第 3 个元素:  扩容！搬 2 个元素, 代价 1 + 2 = 3
插入第 4 个元素:  无需扩容,  代价 1
插入第 5 个元素:  扩容！搬 4 个元素, 代价 1 + 4 = 5
插入第 6~8:       无需扩容,  每次代价 1
插入第 9 个元素:  扩容！搬 8 个元素, 代价 1 + 8 = 9
...
```

在“按倍数扩容”的简化模型中，扩容会发生在接近 1, 2, 4, 8, 16, ... 这些节点。

总代价：
```
n 次普通插入: n × 1 = n
所有 rehashing: 1 + 2 + 4 + 8 + ... + n/2 = n - 1 ≈ n  (等比数列求和)
总计: n + n = 2n
每次插入的均摊代价: 2n / n = 2 = O(1)
```

#### 更严谨的理解：银行家法

每次普通插入时，你不仅支付 1 的代价，还"存入" 2 个代币到银行。当 rehashing 发生时，用存款支付搬迁代价。你会发现存款总是够的，所以每次插入收取 3 个代币（常数）就能覆盖所有开销。

> **结论**：哈希表的插入、查找、删除，在**均摊**意义下都是 **O(1)** 的。

---

## 阶段四：动手实践

### 4.1 认识 `std::unordered_map`：接口、迭代器、桶操作

#### 基础接口

```cpp
#include <unordered_map>
#include <string>
#include <iostream>

int main() {
    // ========== 构造 ==========
    std::unordered_map<std::string, int> scores;                 // 空构造
    std::unordered_map<std::string, int> ages{{"Alice", 25}, {"Bob", 30}};  // 初始化列表

    // ========== 插入 ==========
    scores["Alice"] = 95;                         // operator[]：不存在则插入，存在则覆盖
    scores["Bob"] = 87;
    scores.insert({"Charlie", 72});               // insert：不存在则插入，存在则**不覆盖**
    scores.insert_or_assign("Alice", 98);         // C++17：不存在则插入，存在则覆盖
    scores.emplace("Dave", 88);                   // 原地构造
    scores.try_emplace("Dave", 0);                // C++17：若 key 已存在，不做任何事（包括不构造 value）

    // ========== 访问 ==========
    std::cout << scores["Alice"] << "\n";         // 98（operator[] 若 key 不存在会插入默认值！危险！）
    std::cout << scores.at("Bob") << "\n";        // 87（at() 若 key 不存在抛 std::out_of_range）

    // ========== 查找 ==========
    if (scores.contains("Charlie")) {             // C++20
        std::cout << "Charlie 的分数: " << scores["Charlie"] << "\n";
    }

    auto it = scores.find("Eve");                 // 返回迭代器
    if (it != scores.end()) {
        std::cout << it->second << "\n";
    } else {
        std::cout << "Eve 不存在\n";
    }

    std::cout << "Bob 出现次数: " << scores.count("Bob") << "\n";  // 0 或 1

    // ========== 删除 ==========
    scores.erase("Bob");                          // 按 key 删除
    if (auto it2 = scores.find("Dave"); it2 != scores.end()) {
        scores.erase(it2);                         // 按迭代器删除（先判空更安全）
    }

    // ========== 大小信息 ==========
    std::cout << "元素数量: " << scores.size() << "\n";
    std::cout << "是否为空: " << scores.empty() << "\n";

    // ========== 遍历 ==========
    for (const auto& [name, score] : scores) {    // C++17 结构化绑定
        std::cout << name << ": " << score << "\n";
    }
    // 注意：遍历顺序是不确定的！（unordered）
}
```

#### `operator[]` 的陷阱

```cpp
std::unordered_map<std::string, int> m;

// 危险！key "xyz" 不存在时，operator[] 会自动插入 {"xyz", 0}
if (m["xyz"] == 0) {  // 副作用：m 中多了一个元素！
    // ...
}

// 安全做法：
if (m.count("xyz") == 0) { /* 不存在 */ }
if (!m.contains("xyz"))  { /* C++20, 不存在 */ }
if (m.find("xyz") == m.end()) { /* 不存在 */ }
```

#### 桶操作——窥探内部结构

```cpp
#include <unordered_map>
#include <iostream>

int main() {
    std::unordered_map<int, std::string> m;
    for (int i = 0; i < 20; ++i)
        m[i] = "val";

    // 桶的数量
    std::cout << "bucket_count = " << m.bucket_count() << "\n";

    // 查看每个桶里有几个元素
    for (std::size_t i = 0; i < m.bucket_count(); ++i) {
        std::cout << "桶 " << i << " 有 " << m.bucket_size(i) << " 个元素";
        if (m.bucket_size(i) > 0) {
            std::cout << ": ";
            // 遍历某一个桶内的所有元素
            for (auto it = m.begin(i); it != m.end(i); ++it) {
                std::cout << "[" << it->first << "=" << it->second << "] ";
            }
        }
        std::cout << "\n";
    }

    // 查看某个 key 在第几个桶
    std::cout << "key=7 在桶 " << m.bucket(7) << "\n";

    // 装载因子
    std::cout << "load_factor = " << m.load_factor() << "\n";
    std::cout << "max_load_factor = " << m.max_load_factor() << "\n";
}
```

#### 迭代器特性

| 特性       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 迭代器类型 | **前向迭代器**（Forward Iterator），只能 `++`                |
| 遍历顺序   | **无序**，不保证任何顺序                                     |
| 迭代器失效 | 插入导致 rehashing 时所有迭代器失效；删除只令被删元素的迭代器失效 |

---

### 4.2 自定义 `hash<T>`：给自己的结构体写哈希函数

`std::unordered_map<K, V>` 要求 key 类型 `K` 满足两个条件：
1. 有对应的 **哈希函数**（能计算 `std::hash<K>{}(key)`）
2. 有 **相等比较**（`operator==`）

标准库已经为 `int`, `double`, `std::string`, 指针等常见类型提供了 `std::hash` 特化。但对于**自定义类型**，你需要自己写。

#### 方法一：特化 `std::hash`（最正统）

```cpp
#include <unordered_map>
#include <iostream>
#include <string>
#include <functional>  // std::hash

struct Student {
    std::string name;
    int id;

    // 必须提供 operator==
    bool operator==(const Student& other) const {
        return name == other.name && id == other.id;
    }
};

// 在 std 命名空间里特化 hash
namespace std {
template<>
struct hash<Student> {
    std::size_t operator()(const Student& s) const noexcept {
        // 分别计算各字段的哈希，然后组合
        std::size_t h1 = std::hash<std::string>{}(s.name);
        std::size_t h2 = std::hash<int>{}(s.id);

        // 经典的哈希组合方式（来自 Boost）
        // seed ^= hash(val) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
        std::size_t seed = h1;
        seed ^= h2 + 0x9e3779b9 + (seed << 6) + (seed >> 2);
        return seed;
    }
};
} // namespace std

int main() {
    std::unordered_map<Student, double> gpa;
    gpa[{"Alice", 1001}] = 3.8;
    gpa[{"Bob", 1002}]   = 3.5;

    Student query{"Alice", 1001};
    std::cout << query.name << " 的 GPA: " << gpa[query] << "\n";
}
```

##### `0x9e3779b9` 是什么？

这是黄金比例 `φ = (√5 - 1) / 2` 乘以 `2³²` 的整数部分。用它来"搅乱"哈希值，可以让组合后的结果分布更均匀。

#### 方法二：传入自定义哈希仿函数

```cpp
struct StudentHash {
    std::size_t operator()(const Student& s) const noexcept {
        auto h1 = std::hash<std::string>{}(s.name);
        auto h2 = std::hash<int>{}(s.id);
        return h1 ^ (h2 << 1);  // 简单组合，适合字段少的情况
    }
};

// 同时也可以自定义相等比较
struct StudentEqual {
    bool operator()(const Student& a, const Student& b) const {
        return a.name == b.name && a.id == b.id;
    }
};

// 使用时传入模板参数
std::unordered_map<Student, double, StudentHash, StudentEqual> gpa;
```

#### 方法三：用 Lambda（C++20）

```cpp
auto hasher = [](const Student& s) {
    return std::hash<std::string>{}(s.name) ^ (std::hash<int>{}(s.id) << 1);
};

// C++20 允许 lambda 作为模板参数（需要无捕获的 lambda）
std::unordered_map<Student, double, decltype(hasher)> gpa(16, hasher);
```

#### 通用哈希组合工具

在实际项目中，推荐写一个可复用的 `hash_combine`：

```cpp
// 通用哈希值组合函数
inline void hash_combine(std::size_t& seed, std::size_t value) {
    seed ^= value + 0x9e3779b9 + (seed << 6) + (seed >> 2);
}

// 可变参数模板版本：一次性组合多个字段
template<typename... Args>
std::size_t hash_values(const Args&... args) {
    std::size_t seed = 0;
    (hash_combine(seed, std::hash<std::decay_t<Args>>{}(args)), ...);
    // fold expression (C++17)
    return seed;
}

// 使用示例
namespace std {
template<>
struct hash<Student> {
    std::size_t operator()(const Student& s) const noexcept {
        return ::hash_values(s.name, s.id);  // 简洁！
    }
};
} // namespace std
```

---

### 4.3 从零手写哈希表模板类：`put` / `get` / `resize`

这是本教程的重头戏——我们用**拉链法**从零实现一个完整的、泛型的哈希表。

```cpp
#include <vector>
#include <list>
#include <functional>   // std::hash, std::equal_to
#include <type_traits>  // std::decay_t
#include <cmath>        // std::ceil
#include <stdexcept>    // std::out_of_range
#include <utility>      // std::pair, std::move
#include <iostream>

template<
    typename Key,
    typename Value,
    typename Hash = std::hash<Key>,
    typename KeyEqual = std::equal_to<Key>
>
class HashMap {
public:
    using Entry = std::pair<Key, Value>;
    using Bucket = std::list<Entry>;

private:
    std::vector<Bucket> buckets_;   // 桶数组
    std::size_t size_ = 0;         // 当前元素数量
    float max_load_factor_ = 1.0f;
    Hash hasher_;
    KeyEqual equal_;

    static constexpr std::size_t INITIAL_BUCKET_COUNT = 16;

    // 计算 key 应该放入哪个桶
    std::size_t bucket_index(const Key& key) const {
        return hasher_(key) % buckets_.size();
    }

    // 在指定桶中查找 key，返回链表中的迭代器
    typename Bucket::iterator find_in_bucket(Bucket& bucket, const Key& key) {
        for (auto it = bucket.begin(); it != bucket.end(); ++it) {
            if (equal_(it->first, key)) {
                return it;
            }
        }
        return bucket.end();
    }

    typename Bucket::const_iterator find_in_bucket(const Bucket& bucket, const Key& key) const {
        for (auto it = bucket.cbegin(); it != bucket.cend(); ++it) {
            if (equal_(it->first, key)) {
                return it;
            }
        }
        return bucket.cend();
    }

    // 检查是否需要扩容，如果需要就 rehash
    void check_and_rehash() {
        float lf = static_cast<float>(size_) / static_cast<float>(buckets_.size());
        if (lf > max_load_factor_) {
            rehash(buckets_.size() * 2);
        }
    }

public:
    // ========== 构造函数 ==========
    explicit HashMap(std::size_t initial_buckets = INITIAL_BUCKET_COUNT)
        : buckets_(initial_buckets) {}

    // ========== 插入 / 更新 ==========
    void put(const Key& key, const Value& value) {
        check_and_rehash();
        auto& bucket = buckets_[bucket_index(key)];
        auto it = find_in_bucket(bucket, key);
        if (it != bucket.end()) {
            it->second = value;  // key 已存在，更新 value
        } else {
            bucket.emplace_back(key, value);
            ++size_;
        }
    }

    // 移动语义版本
    void put(const Key& key, Value&& value) {
        check_and_rehash();
        auto& bucket = buckets_[bucket_index(key)];
        auto it = find_in_bucket(bucket, key);
        if (it != bucket.end()) {
            it->second = std::move(value);
        } else {
            bucket.emplace_back(key, std::move(value));
            ++size_;
        }
    }

    // ========== 查找 ==========
    Value* get(const Key& key) {
        auto& bucket = buckets_[bucket_index(key)];
        auto it = find_in_bucket(bucket, key);
        return (it != bucket.end()) ? &(it->second) : nullptr;
    }

    const Value* get(const Key& key) const {
        const auto& bucket = buckets_[bucket_index(key)];
        auto it = find_in_bucket(bucket, key);
        return (it != bucket.end()) ? &(it->second) : nullptr;
    }

    // at() 版本，找不到抛异常
    Value& at(const Key& key) {
        auto* ptr = get(key);
        if (!ptr) throw std::out_of_range("HashMap::at - key not found");
        return *ptr;
    }

    // ========== operator[] ==========
    Value& operator[](const Key& key) {
        check_and_rehash();
        auto& bucket = buckets_[bucket_index(key)];
        auto it = find_in_bucket(bucket, key);
        if (it != bucket.end()) {
            return it->second;
        }
        // key 不存在，插入默认值
        bucket.emplace_back(key, Value{});
        ++size_;
        return bucket.back().second;
    }

    // ========== 删除 ==========
    bool erase(const Key& key) {
        auto& bucket = buckets_[bucket_index(key)];
        auto it = find_in_bucket(bucket, key);
        if (it != bucket.end()) {
            bucket.erase(it);
            --size_;
            return true;
        }
        return false;
    }

    // ========== 包含检查 ==========
    bool contains(const Key& key) const {
        return get(key) != nullptr;
    }

    // ========== Rehash ==========
    void rehash(std::size_t new_bucket_count) {
        if (new_bucket_count <= buckets_.size()) return;

        std::vector<Bucket> new_buckets(new_bucket_count);

        // 把所有旧元素搬到新桶
        for (auto& bucket : buckets_) {
            for (auto& entry : bucket) {
                std::size_t new_idx = hasher_(entry.first) % new_bucket_count;
                new_buckets[new_idx].push_back(std::move(entry));
            }
        }

        buckets_ = std::move(new_buckets);  // 移动赋值，高效！
    }

    // ========== 预分配 ==========
    void reserve(std::size_t count) {
        auto needed = static_cast<std::size_t>(
            std::ceil(static_cast<float>(count) / max_load_factor_)
        );
        if (needed > buckets_.size()) {
            rehash(needed);
        }
    }

    // ========== 属性 ==========
    std::size_t size() const { return size_; }
    bool empty() const { return size_ == 0; }
    std::size_t bucket_count() const { return buckets_.size(); }
    float load_factor() const {
        return static_cast<float>(size_) / static_cast<float>(buckets_.size());
    }
    float max_load_factor() const { return max_load_factor_; }
    void max_load_factor(float mlf) { max_load_factor_ = mlf; }

    // ========== 调试输出 ==========
    void dump() const {
        for (std::size_t i = 0; i < buckets_.size(); ++i) {
            std::cout << "桶 " << i << ": ";
            for (const auto& [k, v] : buckets_[i]) {
                std::cout << "[" << k << "=" << v << "] → ";
            }
            std::cout << "∅\n";
        }
        std::cout << "size=" << size_ << " buckets=" << buckets_.size()
                  << " load=" << load_factor() << "\n";
    }
};

// ========== 测试 ==========
int main() {
    HashMap<std::string, int> scores;

    scores.put("Alice", 95);
    scores.put("Bob", 87);
    scores.put("Charlie", 72);
    scores.put("Dave", 88);
    scores.put("Eve", 91);

    std::cout << "=== 初始状态 ===\n";
    scores.dump();

    // 查找
    if (auto* val = scores.get("Alice")) {
        std::cout << "\nAlice 的成绩: " << *val << "\n";
    }

    // operator[]
    scores["Frank"] = 76;
    std::cout << "\n添加 Frank 后, size = " << scores.size() << "\n";

    // 更新
    scores.put("Alice", 99);
    std::cout << "Alice 更新后: " << scores.at("Alice") << "\n";

    // 删除
    scores.erase("Bob");
    std::cout << "删除 Bob 后, contains Bob? " << scores.contains("Bob") << "\n";

    // 触发扩容：插入大量元素
    std::cout << "\n=== 插入大量元素测试 ===\n";
    HashMap<int, int> m;
    for (int i = 0; i < 100; ++i) {
        m.put(i, i * 10);
    }
    std::cout << "size=" << m.size()
              << " bucket_count=" << m.bucket_count()
              << " load_factor=" << m.load_factor() << "\n";
    // 验证所有元素都能找到
    for (int i = 0; i < 100; ++i) {
        if (!m.contains(i) || *m.get(i) != i * 10) {
            std::cout << "错误: key=" << i << " 丢失或值不正确\n";
        }
    }
    std::cout << "所有 100 个元素验证通过 ✓\n";
}
```

---

### 4.4 处理边界与内存安全：RAII、移动语义、拷贝控制

我们上面的 `HashMap` 实现已经具备了基本的内存安全，因为我们用的是 `std::vector<std::list<...>>`——这些容器本身就是 RAII 的。但让我们深入理解为什么，以及在更底层的实现中需要注意什么。

#### 4.4.1 RAII 与自动内存管理

**RAII（Resource Acquisition Is Initialization）** 的核心思想：

```cpp
// 好：RAII 风格（我们的实现）
class HashMap {
    std::vector<Bucket> buckets_;  
    // vector 的析构函数自动释放内存
    // list 的析构函数自动释放所有节点
    // 不需要手写析构函数！
};

// 坏：如果你用了裸指针...
class BadHashMap {
    Bucket* buckets_;  // 裸指针
    int capacity_;

    BadHashMap(int cap) : buckets_(new Bucket[cap]), capacity_(cap) {}
    ~BadHashMap() { delete[] buckets_; }  // 必须手动释放！
    // 还得处理拷贝构造、拷贝赋值、移动构造、移动赋值...
};
```

#### 4.4.2 拷贝控制：五则（Rule of Five）

如果你的类管理资源（如裸指针），你需要正确实现"五大金刚"：

```cpp
template<typename Key, typename Value>
class RawHashMap {
    struct Node {
        Key key;
        Value value;
        Node* next;
    };

    Node** buckets_;         // 指针数组
    std::size_t capacity_;
    std::size_t size_;

public:
    // 1. 构造函数
    explicit RawHashMap(std::size_t cap = 16)
        : buckets_(new Node*[cap]{}),  // {} 零初始化，所有指针为 nullptr
          capacity_(cap), size_(0) {}

    // 2. 析构函数
    ~RawHashMap() {
        clear();
        delete[] buckets_;
    }

    // 3. 拷贝构造函数（深拷贝）
    RawHashMap(const RawHashMap& other)
        : buckets_(new Node*[other.capacity_]{}),
          capacity_(other.capacity_), size_(0)
    {
        for (std::size_t i = 0; i < capacity_; ++i) {
            Node* curr = other.buckets_[i];
            while (curr) {
                put(curr->key, curr->value);  // 逐个插入
                curr = curr->next;
            }
        }
    }

    // 4. 拷贝赋值运算符（使用 copy-and-swap 惯用法）
    RawHashMap& operator=(RawHashMap other) {  // 注意：按值传参，触发拷贝
        swap(*this, other);
        return *this;
    }

    // 5. 移动构造函数
    RawHashMap(RawHashMap&& other) noexcept
        : buckets_(other.buckets_),
          capacity_(other.capacity_),
          size_(other.size_)
    {
        other.buckets_ = nullptr;
        other.capacity_ = 0;
        other.size_ = 0;
    }

    // swap 函数
    friend void swap(RawHashMap& a, RawHashMap& b) noexcept {
        using std::swap;
        swap(a.buckets_, b.buckets_);
        swap(a.capacity_, b.capacity_);
        swap(a.size_, b.size_);
    }

    // 清空所有元素
    void clear() {
        for (std::size_t i = 0; i < capacity_; ++i) {
            Node* curr = buckets_[i];
            while (curr) {
                Node* next = curr->next;
                delete curr;
                curr = next;
            }
            buckets_[i] = nullptr;
        }
        size_ = 0;
    }

    // put 和 get 实现省略...
    void put(const Key& key, const Value& value) { /* ... */ }
};
```

> **建议**：在实际开发中，**优先使用 `std::vector`、`std::list` 等 RAII 容器**，避免手动管理内存。手写裸指针版本主要用于学习和面试。

#### 4.4.3 移动语义在哈希表中的应用

`rehash` 中使用 `std::move` 可以避免不必要的拷贝：

```cpp
void rehash(std::size_t new_bucket_count) {
    std::vector<Bucket> new_buckets(new_bucket_count);
    
    for (auto& bucket : buckets_) {
        for (auto& entry : bucket) {
            std::size_t idx = hasher_(entry.first) % new_bucket_count;
            // ↓ 使用 std::move 避免拷贝 entry  
            new_buckets[idx].push_back(std::move(entry));
        }
    }
    
    // ↓ 使用 std::move 把新桶数组移动给 buckets_
    buckets_ = std::move(new_buckets);
    // 旧的 new_buckets（现在是空壳）在函数结束时自动析构
}
```

**移动 vs 拷贝的性能差异**：
- 如果 Value 是 `std::string`，拷贝需要分配新内存 + 复制字符；移动只需要转移指针（O(1)）
- 如果 Value 是 `std::vector<int>`，差异更大

#### 4.4.4 异常安全

考虑这个场景：`put` 在链表 `emplace_back` 时抛出异常（比如内存不足）：

```cpp
void put(const Key& key, const Value& value) {
    check_and_rehash();
    auto& bucket = buckets_[bucket_index(key)];
    auto it = find_in_bucket(bucket, key);
    if (it != bucket.end()) {
        it->second = value;  // 可能抛异常
    } else {
        bucket.emplace_back(key, value);  // 可能抛异常
        ++size_;  // 只有在 emplace_back 成功后才 ++size_
        // 如果 emplace_back 抛异常，size_ 不会增加 → 正确！
    }
}
```

因为 `std::list::emplace_back` 具有**强异常保证**（要么成功，要么回滚），我们的 `put` 操作也自动获得了这个安全性。这就是**RAII 容器的威力**。

---

## 阶段五：算法实战

### 5.1 模式一：频率统计——字母异位词分组

> **题目（LeetCode 49）**：给定一个字符串数组，将字母异位词（anagrams）组合在一起。
>
> 输入：`["eat","tea","tan","ate","nat","bat"]`
> 输出：`[["eat","tea","ate"],["tan","nat"],["bat"]]`

**核心思想**：两个字符串是异位词 ⟺ 排序后相同。用排序后的字符串作为 key，分组。

```cpp
#include <vector>
#include <string>
#include <array>
#include <unordered_map>
#include <algorithm>
#include <iostream>

std::vector<std::vector<std::string>> groupAnagrams(std::vector<std::string>& strs) {
    std::unordered_map<std::string, std::vector<std::string>> groups;

    for (const auto& s : strs) {
        std::string key = s;
        std::ranges::sort(key);  // 排序后的字符串作为 key
        groups[key].push_back(s);
    }

    std::vector<std::vector<std::string>> result;
    result.reserve(groups.size());
    for (auto& [_, group] : groups) {
        result.push_back(std::move(group));
    }
    return result;
}
```

**复杂度分析**：
- 排序每个字符串：O(k log k)，k 是字符串长度
- 总共 n 个字符串：**O(n · k log k)**
- 哈希表查找和插入：均摊 O(1)

#### 进阶：不排序的 O(n·k) 解法

用字符频率作为 key，避免排序：

```cpp
std::vector<std::vector<std::string>> groupAnagrams(std::vector<std::string>& strs) {
    // 用 26 个字母的频率字符串作为 key
    // 例如 "eat" → "1#0#0#0#1#0#...#1#0#0#..."（a出现1次, e出现1次, t出现1次）
    std::unordered_map<std::string, std::vector<std::string>> groups;

    for (const auto& s : strs) {
        std::array<int, 26> freq{};
        for (char c : s) {
            ++freq[c - 'a'];
        }
        // 构造唯一的 key 字符串
        std::string key;
        for (int i = 0; i < 26; ++i) {
            key += std::to_string(freq[i]);
            key += '#';  // 分隔符，防止 "12" + "3" 和 "1" + "23"
        }
        groups[key].push_back(s);
    }

    std::vector<std::vector<std::string>> result;
    result.reserve(groups.size());
    for (auto& [_, group] : groups) {
        result.push_back(std::move(group));
    }
    return result;
}
```

**复杂度分析**：
- 构造频率签名：每个字符串 O(k)
- 总复杂度：**O(n · k)**
- 额外空间：哈希表与分组结果，共 O(n · k)

> 说明：该写法默认字符串字符集为小写字母 `'a'`~`'z'`。如果包含大写、中文或其他字符，需要改成更通用的计数策略。