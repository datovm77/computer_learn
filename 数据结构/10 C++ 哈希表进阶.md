# C++ 哈希表完全教程（中）—— 阶段四：动手实践

---

> 📌 **编译标准说明**：本文多数代码在 C++17 可用；涉及 `contains()`、`std::ranges::sort`、`operator== = default` 的示例需要 C++20（或更高）。

## 4.1 认识 `std::unordered_map`：接口、迭代器、桶操作

### 基础接口速览

```cpp
#include <unordered_map>
#include <string>
#include <iostream>

int main() {
    // ===== 构造 =====
    std::unordered_map<std::string, int> scores;
    std::unordered_map<std::string, int> ages{{"Alice", 25}, {"Bob", 30}};

    // ===== 插入 =====
    scores["Alice"] = 95;                     // operator[]：不存在→插入，存在→覆盖
    scores.insert({"Bob", 87});               // insert：不存在→插入，存在→不覆盖
    scores.insert_or_assign("Alice", 98);     // C++17：不存在→插入，存在→覆盖
    scores.emplace("Charlie", 72);            // 原地构造
    scores.try_emplace("Charlie", 0);         // C++17：key 已存在则什么都不做（连 value 都不构造）

    // ===== 访问 =====
    std::cout << scores["Alice"] << "\n";     // 98
    std::cout << scores.at("Bob") << "\n";    // 87（不存在则抛 std::out_of_range）

    // ===== 查找 =====
    if (scores.contains("Charlie"))           // C++20
        std::cout << "找到 Charlie\n";

    auto it = scores.find("Eve");             // 返回迭代器
    if (it != scores.end())
        std::cout << it->second << "\n";
    else
        std::cout << "Eve 不存在\n";

    // ===== 删除 =====
    scores.erase("Bob");                      // 按 key
    if (auto it2 = scores.find("Charlie"); it2 != scores.end())
        scores.erase(it2);                     // 按迭代器（先判空更安全）

    // ===== 遍历（无序！）=====
    for (const auto& [name, score] : scores)  // C++17 结构化绑定
        std::cout << name << ": " << score << "\n";
}
```

### `operator[]` 的经典陷阱

```cpp
std::unordered_map<std::string, int> m;

// ❌ 危险！key "xyz" 不存在时，operator[] 会自动插入 {"xyz", 0}
if (m["xyz"] == 0) { /* m 中凭空多了一个元素！ */ }

// ✅ 安全做法
if (m.find("xyz") == m.end()) { /* 不存在 */ }
if (!m.contains("xyz"))        { /* C++20 */ }
if (m.count("xyz") == 0)      { /* 通用 */ }
```

### 桶操作——窥探内部结构

```cpp
std::unordered_map<int, std::string> m;
for (int i = 0; i < 20; ++i) m[i] = "val";

std::cout << "bucket_count = " << m.bucket_count() << "\n";
std::cout << "load_factor  = " << m.load_factor() << "\n";

// 遍历每个桶
for (std::size_t i = 0; i < m.bucket_count(); ++i) {
    std::cout << "桶 " << i << " (" << m.bucket_size(i) << " 个元素): ";
    for (auto it = m.begin(i); it != m.end(i); ++it)
        std::cout << "[" << it->first << "] ";
    std::cout << "\n";
}

// 查询某个 key 属于哪个桶
std::cout << "key=7 在桶 " << m.bucket(7) << "\n";
```

### 迭代器特性

| 特性     | 说明                                                        |
| -------- | ----------------------------------------------------------- |
| 类型     | **前向迭代器**，只支持 `++`                                 |
| 顺序     | **无序**，不保证任何顺序                                    |
| 失效规则 | 插入导致 rehash → **所有迭代器失效**；删除 → 仅被删元素失效 |

---

## 4.2 自定义 `hash<T>`：给自己的结构体写哈希函数

`unordered_map<K,V>` 要求 K 具备**哈希函数**和 **`operator==`**。标准库已为 `int`、`string` 等提供。自定义类型需手动实现。

### 方法一：特化 `std::hash`（最正统）

```cpp
#include <functional>
#include <unordered_map>
#include <string>
#include <type_traits>

struct Student {
    std::string name;
    int id;
    bool operator==(const Student&) const = default; // C++20 默认比较
};

// 通用哈希组合工具（来自 Boost）
inline void hash_combine(std::size_t& seed, std::size_t value) {
    seed ^= value + 0x9e3779b9 + (seed << 6) + (seed >> 2);
    // 0x9e3779b9 = 黄金比例 × 2^32，用于"搅乱"比特分布
}

namespace std {
template<>
struct hash<Student> {
    std::size_t operator()(const Student& s) const noexcept {
        std::size_t seed = std::hash<std::string>{}(s.name);
        hash_combine(seed, std::hash<int>{}(s.id));
        return seed;
    }
};
} // namespace std

// 直接使用，不需要额外模板参数
std::unordered_map<Student, double> gpa;
```

### 方法二：自定义仿函数（不侵入 std）

```cpp
struct StudentHash {
    std::size_t operator()(const Student& s) const noexcept {
        auto h1 = std::hash<std::string>{}(s.name);
        auto h2 = std::hash<int>{}(s.id);
        return h1 ^ (h2 << 1);
    }
};

// 传入第三个模板参数
std::unordered_map<Student, double, StudentHash> gpa;
```

### 进阶：可变参数哈希组合（C++17）

```cpp
template<typename... Args>
std::size_t hash_values(const Args&... args) {
    std::size_t seed = 0;
    (hash_combine(seed, std::hash<std::decay_t<Args>>{}(args)), ...);
    // ↑ fold expression：对每个参数依次调用 hash_combine
    return seed;
}

// 用法极简洁
namespace std {
template<>
struct hash<Student> {
    std::size_t operator()(const Student& s) const noexcept {
        return ::hash_values(s.name, s.id);
    }
};
} // namespace std
```

---

## 4.3 从零手写哈希表模板类

使用**拉链法**，完整实现 `put`/`get`/`erase`/`operator[]`/`rehash`：

```cpp
#include <vector>
#include <list>
#include <functional>
#include <stdexcept>
#include <utility>
#include <cmath>
#include <iostream>
#include <cassert>

template<typename Key, typename Value,
         typename Hash = std::hash<Key>,
         typename KeyEqual = std::equal_to<Key>>
class HashMap {
    using Entry  = std::pair<Key, Value>;
    using Bucket = std::list<Entry>;

    std::vector<Bucket> buckets_;
    std::size_t size_ = 0;
    float max_lf_ = 1.0f;
    Hash hasher_;
    KeyEqual equal_;

    std::size_t idx(const Key& key) const { return hasher_(key) % buckets_.size(); }

    auto find_entry(Bucket& b, const Key& key) {
        for (auto it = b.begin(); it != b.end(); ++it)
            if (equal_(it->first, key)) return it;
        return b.end();
    }
    auto find_entry(const Bucket& b, const Key& key) const {
        for (auto it = b.cbegin(); it != b.cend(); ++it)
            if (equal_(it->first, key)) return it;
        return b.cend();
    }

    void maybe_rehash() {
        if (static_cast<float>(size_) / buckets_.size() > max_lf_)
            rehash(buckets_.size() * 2);
    }

public:
    explicit HashMap(std::size_t n = 16) : buckets_(n == 0 ? 1 : n) {}

    // ---- 插入/更新 ----
    void put(const Key& key, Value value) {
        maybe_rehash();
        auto& b = buckets_[idx(key)];
        if (auto it = find_entry(b, key); it != b.end())
            it->second = std::move(value);   // 更新
        else { b.emplace_back(key, std::move(value)); ++size_; }
    }

    // ---- 查找 ----
    Value*       get(const Key& key)       { auto& b=buckets_[idx(key)]; auto it=find_entry(b,key); return it!=b.end()?&it->second:nullptr; }
    const Value* get(const Key& key) const { auto& b=buckets_[idx(key)]; auto it=find_entry(b,key); return it!=b.cend()?&it->second:nullptr; }

    Value& at(const Key& key) {
        if (auto* p = get(key)) return *p;
        throw std::out_of_range("HashMap::at");
    }

    // ---- operator[] ----
    Value& operator[](const Key& key) {
        maybe_rehash();
        auto& b = buckets_[idx(key)];
        if (auto it = find_entry(b, key); it != b.end()) return it->second;
        b.emplace_back(key, Value{});
        ++size_;
        return b.back().second;
    }

    // ---- 删除 ----
    bool erase(const Key& key) {
        auto& b = buckets_[idx(key)];
        if (auto it = find_entry(b, key); it != b.end()) { b.erase(it); --size_; return true; }
        return false;
    }

    bool contains(const Key& key) const { return get(key) != nullptr; }

    // ---- Rehash ----
    void rehash(std::size_t new_n) {
        if (new_n <= buckets_.size()) return;
        std::vector<Bucket> nb(new_n);
        for (auto& b : buckets_)
            for (auto& e : b)
                nb[hasher_(e.first) % new_n].push_back(std::move(e));  // 移动语义！
        buckets_ = std::move(nb);
    }

    void reserve(std::size_t count) {
        auto need = static_cast<std::size_t>(std::ceil(count / max_lf_));
        if (need > buckets_.size()) rehash(need);
    }

    // ---- 属性 ----
    std::size_t size()         const { return size_; }
    std::size_t bucket_count() const { return buckets_.size(); }
    float load_factor()        const { return static_cast<float>(size_) / buckets_.size(); }

    void dump() const {
        for (std::size_t i = 0; i < buckets_.size(); ++i) {
            if (buckets_[i].empty()) continue;
            std::cout << "桶" << i << ": ";
            for (const auto& [k,v] : buckets_[i]) std::cout << "[" << k << "=" << v << "] ";
            std::cout << "\n";
        }
        std::cout << "size=" << size_ << " buckets=" << buckets_.size()
                  << " load=" << load_factor() << "\n\n";
    }
};
```

**测试**：
```cpp
int main() {
    HashMap<std::string, int> m;
    m.put("Alice", 95); m.put("Bob", 87); m.put("Charlie", 72);
    m.dump();

    m["Dave"] = 88;            // operator[]
    m.put("Alice", 99);       // 更新
    m.erase("Bob");           // 删除
    std::cout << "Alice=" << m.at("Alice") << " contains Bob? " << m.contains("Bob") << "\n";

    // 大量插入触发自动扩容
    HashMap<int, int> big;
    for (int i = 0; i < 100; ++i) big.put(i, i * 10);
    std::cout << "size=" << big.size() << " buckets=" << big.bucket_count() << "\n";
    for (int i = 0; i < 100; ++i) assert(big.contains(i) && *big.get(i) == i * 10);
    std::cout << "100 个元素全部验证通过 ✓\n";
}
```

---

## 4.4 处理边界与内存安全

### RAII 的威力

我们的 `HashMap` 用 `std::vector<std::list<...>>` 存储数据——**不需要手写析构函数**！`vector` 和 `list` 的析构自动释放所有内存。

如果用裸指针（`Node** buckets = new Node*[cap]{}`），则必须完整实现 **Rule of Five**：

| 必须实现 | 作用                               |
| -------- | ---------------------------------- |
| 析构函数 | 释放所有 `Node` 和 `buckets_` 数组 |
| 拷贝构造 | 深拷贝所有链表节点                 |
| 拷贝赋值 | copy-and-swap 惯用法               |
| 移动构造 | 窃取指针，源对象置空               |
| 移动赋值 | 通过 swap 实现                     |

> **原则**：实际开发中**优先用 RAII 容器**，裸指针版本留给学习和面试。

### 移动语义在 rehash 中的关键作用

```cpp
void rehash(std::size_t new_n) {
    std::vector<Bucket> nb(new_n);
    for (auto& b : buckets_)
        for (auto& e : b)
            nb[hasher_(e.first) % new_n].push_back(std::move(e));
            //                                      ^^^^^^^^^
            // std::move 避免拷贝！如果 Value 是 string/vector，性能差异巨大
    buckets_ = std::move(nb);  // 移动赋值，O(1)
}
```

### 异常安全

`std::list::emplace_back` 具有**强异常保证**：成功或回滚。我们的 `++size_` 在 `emplace_back` 之后执行，所以如果抛异常，`size_` 不会错误增加。这就是 RAII 容器自动带来的安全性。

---

# C++ 哈希表完全教程（下）—— 阶段五：算法实战

---

## 5.1 模式一：频率统计——字母异位词分组

> **LeetCode 49**：给定字符串数组，将字母异位词分组。
> 输入：`["eat","tea","tan","ate","nat","bat"]`
> 输出：`[["eat","tea","ate"], ["tan","nat"], ["bat"]]`

**核心思路**：异位词排序后完全相同 → 排序后的字符串作为哈希表的 key。

### 解法一：排序作 key — O(n · k log k)

```cpp
#include <vector>
#include <string>
#include <array>
#include <unordered_map>
#include <algorithm>

std::vector<std::vector<std::string>> groupAnagrams(std::vector<std::string>& strs) {
    std::unordered_map<std::string, std::vector<std::string>> groups;

    for (const auto& s : strs) {
        std::string key = s;
        std::ranges::sort(key);        // 排序 → "eat" "tea" "ate" 都变成 "aet"
        groups[key].push_back(s);      // 同 key 的归为一组
    }

    std::vector<std::vector<std::string>> result;
    result.reserve(groups.size());
    for (auto& [_, group] : groups)
        result.push_back(std::move(group));  // 移动语义，避免拷贝
    return result;
}
```

**逐行剖析**：

| 步骤             | 操作                          | 时间                       |
| ---------------- | ----------------------------- | -------------------------- |
| 拷贝字符串并排序 | `sort(key)`                   | O(k log k)，k 为字符串长度 |
| 哈希表插入/查找  | `groups[key]` 中 `operator[]` | 均摊 O(k)（哈希计算）      |
| 所有字符串处理   | 循环 n 次                     | 总计 O(n · k log k)        |

### 解法二：字符频率作 key — O(n · k)

排序消耗 O(k log k)。如果用**字符计数**，可以降到 O(k)：

```cpp
std::vector<std::vector<std::string>> groupAnagrams(std::vector<std::string>& strs) {
    std::unordered_map<std::string, std::vector<std::string>> groups;

    for (const auto& s : strs) {
        // 统计 26 个字母的频率
        std::array<int, 26> freq{};
        for (char c : s) ++freq[c - 'a'];

        // 拼接成唯一的字符串 key: "1#0#0#0#1#...#1#0#..."
        std::string key;
        key.reserve(26 * 3);  // 预分配，减少动态扩容
        for (int f : freq) {
            key += std::to_string(f);
            key += '#';  // 分隔符！防止 "12"+"3" 和 "1"+"23" 混淆
        }

        groups[key].push_back(s);
    }

    std::vector<std::vector<std::string>> result;
    result.reserve(groups.size());
    for (auto& [_, group] : groups)
        result.push_back(std::move(group));
    return result;
}
```

> **为什么需要 `#` 分隔符？** 
> 假设 a 出现 12 次、b 出现 3 次 → `"123"`。但 a 出现 1 次、b 出现 23 次也是 `"123"`。加分隔符：`"12#3#"` vs `"1#23#"`，完美区分。

---

## 5.2 模式二：快速查找——两数之和全变形

> **LeetCode 1**：给定数组 `nums` 和目标值 `target`，找出和为 `target` 的两个元素的下标。

### 核心思想

暴力法对每个 `nums[i]`，遍历之后的元素找 `target - nums[i]`，O(n²)。

**哈希表优化**：一边遍历、一边存。对于当前元素 `nums[i]`，查找 `target - nums[i]` 是否已经在哈希表中。

```cpp
#include <vector>
#include <unordered_map>

std::vector<int> twoSum(std::vector<int>& nums, int target) {
    std::unordered_map<int, int> seen;  // value → index

    for (int i = 0; i < static_cast<int>(nums.size()); ++i) {
        int complement = target - nums[i];
        if (auto it = seen.find(complement); it != seen.end()) {
            return {it->second, i};  // 找到了！
        }
        seen[nums[i]] = i;  // 存入当前元素
    }
    return {};  // 无解
}
```

**复杂度**：O(n) 时间，O(n) 空间。一次遍历搞定。

### 变形一：两数之和 — 返回所有配对

```cpp
// 找出所有和为 target 的不重复配对（值配对，非下标）
#include <vector>
#include <unordered_map>
#include <set>
#include <algorithm>

std::vector<std::pair<int,int>> twoSumAll(std::vector<int>& nums, int target) {
    std::unordered_map<int, int> freq;  // 元素 → 出现次数
    for (int x : nums) ++freq[x];

    std::set<std::pair<int,int>> result;  // 去重
    for (auto& [val, cnt] : freq) {
        int comp = target - val;
        if (comp == val) {
            if (cnt >= 2) result.insert({val, val});
        } else if (freq.find(comp) != freq.end()) {
            auto p = std::minmax(val, comp);
            result.insert(p);
        }
    }
    return {result.begin(), result.end()};
}
```

### 变形二：三数之和（LeetCode 15）

```cpp
// 找出所有 a+b+c=0 的三元组（不重复）
#include <vector>
#include <algorithm>

std::vector<std::vector<int>> threeSum(std::vector<int>& nums) {
    std::ranges::sort(nums);
    std::vector<std::vector<int>> result;
    int n = static_cast<int>(nums.size());

    for (int i = 0; i < n - 2; ++i) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;  // 跳过重复
        int lo = i + 1, hi = n - 1;
        while (lo < hi) {
            int sum = nums[i] + nums[lo] + nums[hi];
            if (sum < 0)      ++lo;
            else if (sum > 0) --hi;
            else {
                result.push_back({nums[i], nums[lo], nums[hi]});
                while (lo < hi && nums[lo] == nums[lo + 1]) ++lo;  // 跳重复
                while (lo < hi && nums[hi] == nums[hi - 1]) --hi;
                ++lo; --hi;
            }
        }
    }
    return result;
}
```

> 三数之和虽然经典搭配排序+双指针，但**核心仍然是"快速查找补数"**的思想。也可以用哈希表做，但排序+双指针在去重方面更方便。

---

## 5.3 模式三：滑动窗口 + 哈希——最长无重复子串

> **LeetCode 3**：给定字符串 `s`，找出不含重复字符的最长子串长度。
> 输入：`"abcabcbb"` → 输出：`3`（`"abc"`）

### 方法一：`unordered_map` 记录字符最后出现位置

```cpp
#include <string>
#include <unordered_map>
#include <algorithm>

int lengthOfLongestSubstring(const std::string& s) {
    std::unordered_map<char, int> last_pos;  // 字符 → 最后出现的下标
    int max_len = 0;
    int left = 0;  // 窗口左边界

    for (int right = 0; right < static_cast<int>(s.size()); ++right) {
        char c = s[right];
        // 如果 c 在窗口内出现过，收缩左边界
        if (auto it = last_pos.find(c); it != last_pos.end() && it->second >= left) {
            left = it->second + 1;  // 跳到重复字符的下一个位置
        }
        last_pos[c] = right;  // 更新最后出现位置
        max_len = std::max(max_len, right - left + 1);
    }
    return max_len;
}
```

**图解过程**（`s = "abcabcbb"`）：

```
right=0: c='a', left=0, 窗口="a",   len=1
right=1: c='b', left=0, 窗口="ab",  len=2
right=2: c='c', left=0, 窗口="abc", len=3 ← 目前最长
right=3: c='a', a 在 pos 0(>=left), left→1, 窗口="bca", len=3
right=4: c='b', b 在 pos 1(>=left), left→2, 窗口="cab", len=3  
right=5: c='c', c 在 pos 2(>=left), left→3, 窗口="abc", len=3
right=6: c='b', b 在 pos 4(>=left), left→5, 窗口="cb",  len=2
right=7: c='b', b 在 pos 6(>=left), left→7, 窗口="b",   len=1
结果: 3
```

### 方法二：用数组代替哈希表（ASCII 优化）

当字符集有限（如 ASCII 128 个字符），数组比哈希表更快：

```cpp
#include <array>
#include <string>
#include <algorithm>

int lengthOfLongestSubstring(const std::string& s) {
    std::array<int, 128> last_pos;
    last_pos.fill(-1);  // -1 表示未出现
    int max_len = 0, left = 0;

    for (int right = 0; right < static_cast<int>(s.size()); ++right) {
        unsigned char c = static_cast<unsigned char>(s[right]);
        if (last_pos[c] >= left)
            left = last_pos[c] + 1;
        last_pos[c] = right;
        max_len = std::max(max_len, right - left + 1);
    }
    return max_len;
}
```

> **经验法则**：当 key 的范围很小（如 26 个字母、128 个 ASCII），用**定长数组**替代哈希表，速度更快且无冲突。

---

## 5.4 综合挑战：手写 LRU 缓存

> **LeetCode 146**：设计 LRU（Least Recently Used）缓存，支持 `get(key)` 和 `put(key, value)`，两者均为 O(1)。当容量满时，淘汰最久未使用的元素。

### 数据结构设计

| 需求                   | 选择            | 原因                            |
| ---------------------- | --------------- | ------------------------------- |
| O(1) 查找              | `unordered_map` | 哈希表恒定时间查找              |
| O(1) 删除 + 移动到最近 | **双向链表**    | 已知节点位置时，O(1) 删除和插入 |

**关键设计**：`unordered_map` 的 value 存**链表迭代器**，实现 O(1) 从哈希表定位到链表节点。

```
双向链表（头=最新，尾=最久）：
  head ↔ [A] ↔ [B] ↔ [C] ↔ [D] ↔ tail
                                    ↑ 容量满时淘汰这个

unordered_map:
  "A" → 指向链表节点 [A]
  "B" → 指向链表节点 [B]
  "C" → 指向链表节点 [C]
  "D" → 指向链表节点 [D]
```

### 完整实现

```cpp
#include <unordered_map>
#include <list>
#include <iostream>

class LRUCache {
    int capacity_;

    // 链表节点：<key, value>，头部=最近使用，尾部=最久未使用
    std::list<std::pair<int, int>> order_;

    // 哈希表：key → 链表迭代器（指向对应节点）
    std::unordered_map<int, std::list<std::pair<int,int>>::iterator> map_;

    // 将节点移到链表头部（标记为"最近使用"）
    void touch(std::list<std::pair<int,int>>::iterator it) {
        // splice 不会使迭代器失效！这是 std::list 的关键特性
        order_.splice(order_.begin(), order_, it);
    }

public:
    explicit LRUCache(int capacity) : capacity_(capacity) {}

    int get(int key) {
        auto it = map_.find(key);
        if (it == map_.end()) return -1;  // 未找到

        touch(it->second);                // 移到最近
        return it->second->second;        // 返回 value
    }

    void put(int key, int value) {
        if (capacity_ <= 0) return;

        if (auto it = map_.find(key); it != map_.end()) {
            // key 已存在：更新 value，移到最近
            it->second->second = value;
            touch(it->second);
            return;
        }

        // key 不存在：检查容量
        if (static_cast<int>(map_.size()) >= capacity_) {
            // 淘汰最久未使用的（链表尾部）
            auto& victim = order_.back();
            map_.erase(victim.first);    // 从哈希表删除
            order_.pop_back();           // 从链表删除
        }

        // 插入新节点到链表头部
        order_.emplace_front(key, value);
        map_[key] = order_.begin();      // 哈希表记录迭代器
    }

    // 调试用：打印当前缓存状态
    void dump() const {
        std::cout << "LRU [";
        for (auto it = order_.begin(); it != order_.end(); ++it) {
            if (it != order_.begin()) std::cout << " → ";
            std::cout << it->first << ":" << it->second;
        }
        std::cout << "] (size=" << map_.size() << "/" << capacity_ << ")\n";
    }
};
```

### 测试与运行过程

```cpp
int main() {
    LRUCache cache(3);  // 容量为 3

    cache.put(1, 100);  cache.dump();  // [1:100]
    cache.put(2, 200);  cache.dump();  // [2:200 → 1:100]
    cache.put(3, 300);  cache.dump();  // [3:300 → 2:200 → 1:100]

    // 访问 key=1，它被移到最近
    std::cout << "get(1) = " << cache.get(1) << "\n";
    cache.dump();                      // [1:100 → 3:300 → 2:200]

    // 插入 key=4，容量满！淘汰最久未使用的 key=2
    cache.put(4, 400);
    cache.dump();                      // [4:400 → 1:100 → 3:300]
    std::cout << "get(2) = " << cache.get(2) << "\n";  // -1（已被淘汰）

    // 更新 key=3
    cache.put(3, 333);
    cache.dump();                      // [3:333 → 4:400 → 1:100]

    // 插入 key=5，淘汰 key=1
    cache.put(5, 500);
    cache.dump();                      // [5:500 → 3:333 → 4:400]
    std::cout << "get(1) = " << cache.get(1) << "\n";  // -1
    std::cout << "get(4) = " << cache.get(4) << "\n";  // 400
    cache.dump();                      // [4:400 → 5:500 → 3:333]
}
```

### 复杂度分析

| 操作  | 时间            | 说明                               |
| ----- | --------------- | ---------------------------------- |
| `get` | **O(1)**        | 哈希查找 O(1) + `splice` O(1)      |
| `put` | **O(1)**        | 哈希查找/插入 O(1) + 链表操作 O(1) |
| 空间  | **O(capacity)** | 哈希表 + 链表各存 capacity 个节点  |

### `std::list::splice` 的妙用

`splice` 是 LRU 实现的灵魂操作。它把一个节点从链表的当前位置**移动**到另一个位置，**不涉及内存分配和释放**，且**不会使迭代器失效**：

```cpp
// 把 order_ 中的 it 节点移动到 order_.begin() 前面
order_.splice(order_.begin(), order_, it);
// 参数: (目标位置, 源链表, 源节点)
// 因为源和目标是同一个链表，所以是"链表内移动"
```

---

## 全教程总结

| 阶段             | 核心收获                                         |
| ---------------- | ------------------------------------------------ |
| **一、建立直觉** | 哈希 = 把 key 变成下标，实现 O(1) 查找           |
| **二、核心机制** | 冲突不可避免；拉链法 vs 开放地址法各有取舍       |
| **三、动态特性** | 装载因子控制性能；rehash 代价高但均摊 O(1)       |
| **四、动手实践** | `unordered_map` 全接口；自定义 hash；RAII 保安全 |
| **五、算法实战** | 频率统计、快速查找、滑动窗口、LRU 缓存           |

### 哈希表关键复杂度速查

| 操作 | 平均                | 最坏                    |
| ---- | ------------------- | ----------------------- |
| 插入 | O(1) 均摊           | O(n)（全冲突或 rehash） |
| 查找 | O(1)                | O(n)                    |
| 删除 | O(1)                | O(n)                    |
| 遍历 | O(n + bucket_count) | —                       |

> **最后的建议**：理解原理后，日常开发直接用 `std::unordered_map` / `std::unordered_set`。手写哈希表的价值在于面试和深入理解底层机制。当 key 的范围很小时（如字符、小整数），优先考虑**数组**代替哈希表，更快且无冲突。