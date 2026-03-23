# C++ STL 课程缺失内容 — 补充学习清单

---

## 一、缺失容器

### 1. `priority_queue`（优先队列）

- **基本概念**

  - 什么是优先队列，与普通 `queue` 的区别

  - 大顶堆（`std::less`，默认）与小顶堆（`std::greater`）

- **构造函数**

  - 默认构造（大顶堆）

  - 指定比较器构造小顶堆：`priority_queue<int, vector<int>, greater<int>>`

  - 用已有容器（如 `vector`）初始化 `priority_queue`

- **常用接口**

  - `push()` — 插入元素

  - `pop()` — 弹出堆顶元素

  - `top()` — 访问堆顶元素（不弹出）

  - `size()` — 元素个数

  - `empty()` — 是否为空

- **自定义数据类型的使用**

  - 存放自定义类/结构体

  - 编写自定义比较器（仿函数或 lambda）

  - 重载 `operator<` 实现自定义优先级

---

### 2. `unordered_map`（无序映射 / 哈希表）

- **基本概念**

  - 底层数据结构：哈希表

  - 与 `map` 的核心区别：无序、平均 O(1) 查找 vs 有序、O(log n) 查找

- **构造和赋值**

  - 默认构造

  - 初始化列表构造：`unordered_map<string, int> m = {{"a", 1}, {"b", 2}}`

  - 拷贝构造与赋值运算符

- **插入和删除**

  - `insert()` — 插入键值对

  - `emplace()` — 原地构造插入

  - `erase()` — 按 key 删除、按迭代器删除

  - `clear()` — 清空

  - `operator[]` — 通过下标访问/插入

- **查找和统计**

  - `find()` — 查找 key，返回迭代器

  - `count()` — 统计 key 出现次数（0 或 1）

  - `contains()` — C++20 判断 key 是否存在

- **大小操作**

  - `size()` — 元素个数

  - `empty()` — 是否为空

- **哈希相关**

  - 桶（bucket）的概念

  - `bucket_count()` — 桶的数量

  - `load_factor()` — 负载因子

  - 自定义类型作为 key 时需要提供自定义哈希函数

---

### 3. `unordered_set`（无序集合）

- **基本概念**

  - 底层数据结构：哈希表

  - 与 `set` 的核心区别：无序、平均 O(1) 查找 vs 有序、O(log n) 查找

  - 元素唯一性

- **构造和赋值**

  - 默认构造

  - 初始化列表构造

  - 拷贝构造与赋值运算符

- **插入和删除**

  - `insert()` — 插入元素

  - `emplace()` — 原地构造插入

  - `erase()` — 按值删除、按迭代器删除

  - `clear()` — 清空

- **查找和统计**

  - `find()` — 查找元素，返回迭代器

  - `count()` — 统计元素出现次数（0 或 1）

  - `contains()` — C++20

- **大小操作**

  - `size()`, `empty()`

- `unordered_multiset` 简要了解

  - 与 `unordered_set` 的区别：允许重复元素

---

### 4. `std::array`（固定大小数组）

- **基本概念**

  - 编译期确定大小的数组容器

  - 与 C 风格数组的区别（不会退化为指针、可获取大小、支持赋值）

- **构造**

  - `array<int, 5> arr = {1, 2, 3, 4, 5}`

  - 聚合初始化

- **常用接口**

  - `at()` — 带越界检查的元素访问

  - `operator[]` — 不带越界检查的元素访问

  - `front()`, `back()` — 首尾元素

  - `size()` — 大小

  - `empty()` — 是否为空

  - `fill()` — 全部填充为某个值

  - `swap()` — 交换两个 `array`

  - `data()` — 获取底层数组指针

---

### 5. `forward_list`（单向链表）

- **基本概念**

  - 与 `list`（双向链表）的区别：内存更小、只能单向遍历

  - 只支持前向迭代器

- **常用接口**

  - `push_front()`, `pop_front()` — 头部插入/删除

  - `insert_after()` — 在指定位置之后插入

  - `erase_after()` — 删除指定位置之后的元素

  - `before_begin()` — 返回首元素之前的迭代器

  - `sort()`, `merge()`, `reverse()`, `unique()` — 成员函数版本

---

## 二、缺失算法

### 1. 二分查找相关算法

- **`lower_bound(first, last, value)`**

  - 在有序范围中查找第一个 **≥** value 的元素位置

  - 返回迭代器

  - 自定义比较器版本

- **`upper_bound(first, last, value)`**

  - 在有序范围中查找第一个 **>** value 的元素位置

  - 返回迭代器

  - 自定义比较器版本

- **`equal_range(first, last, value)`**

  - 返回一个 `pair<iterator, iterator>`，表示等于 value 的元素范围

  - 等价于 `{lower_bound(), upper_bound()}`

---

### 2. 排列算法

- **`next_permutation(first, last)`**

  - 将序列变为下一个字典序排列

  - 如果已是最后一个排列，则变为最小排列并返回 `false`

  - 自定义比较器版本

- **`prev_permutation(first, last)`**

  - 将序列变为上一个字典序排列

  - 如果已是最小排列，则变为最大排列并返回 `false`

---

### 3. 最值算法

- **`min_element(first, last)`**

  - 返回指向范围内最小元素的迭代器

  - 自定义比较器版本

- **`max_element(first, last)`**

  - 返回指向范围内最大元素的迭代器

  - 自定义比较器版本

- **`minmax_element(first, last)`**（C++11）

  - 同时返回最小和最大元素的迭代器对 `pair<iterator, iterator>`

---

### 4. 去重与移除算法

- **`unique(first, last)`**

  - 去除**相邻**重复元素，将不重复元素移到前面

  - 返回新逻辑末尾的迭代器

  - 常配合 `erase()` 使用（erase-unique 惯用法）

  - 使用前通常需要先 `sort()`

- **`remove(first, last, value)`**

  - 将不等于 value 的元素移到前面

  - 返回新逻辑末尾的迭代器

  - 不会真正删除元素，需配合 `erase()` 使用（**erase-remove 惯用法**）

- **`remove_if(first, last, pred)`**

  - 与 `remove` 相同，但使用谓词判断哪些元素应被移除

---

### 5. 部分排序与选择算法

- **`stable_sort(first, last)`**

  - 稳定排序：相等元素保持原有相对顺序

  - 自定义比较器版本

- **`partial_sort(first, middle, last)`**

  - 只排序前 `middle - first` 个最小元素

  - 适用于只需要前 N 小/大的场景

- **`nth_element(first, nth, last)`**

  - 重排序列，使第 nth 个位置上放的是排序后应该在此位置的元素

  - nth 之前的元素都 ≤ 它，之后的元素都 ≥ 它

  - 平均时间复杂度 O(n)

---

### 6. 分区算法

- **`partition(first, last, pred)`**

  - 将满足谓词的元素移动到前面，不满足的移动到后面

  - 不保证相对顺序

  - 返回分区点的迭代器

- **`stable_partition(first, last, pred)`**

  - 与 `partition` 相同，但保持各组内元素的相对顺序

---

### 7. 旋转算法

- **`rotate(first, middle, last)`**

  - 将 `[first, middle)` 和 `[middle, last)` 两段进行旋转

  - 旋转后 `middle` 指向的元素变成新的首元素

---

## 三、迭代器专题

### 1. 迭代器的分类

- **输入迭代器（Input Iterator）**

  - 只读、单次遍历、只能前进

  - 如 `istream_iterator`

- **输出迭代器（Output Iterator）**

  - 只写、单次遍历、只能前进

  - 如 `ostream_iterator`

- **前向迭代器（Forward Iterator）**

  - 可读可写、可多次遍历、只能前进

  - 如 `forward_list` 的迭代器

- **双向迭代器（Bidirectional Iterator）**

  - 可读可写、可前进可后退

  - 如 `list`、`set`、`map` 的迭代器

- **随机访问迭代器（Random Access Iterator）**

  - 可读可写、支持跳跃访问（`it + n`、`it - n`、`it[n]`、比较大小）

  - 如 `vector`、`deque`、`array` 的迭代器

### 2. 各容器对应的迭代器类型

- `vector` → 随机访问迭代器

- `deque` → 随机访问迭代器

- `array` → 随机访问迭代器

- `list` → 双向迭代器

- `forward_list` → 前向迭代器

- `set` / `map` → 双向迭代器

- `unordered_set` / `unordered_map` → 前向迭代器

### 3. 反向迭代器

- `rbegin()`, `rend()` — 反向遍历

- 与正向迭代器的关系、转换方式

### 4. const 迭代器

- `cbegin()`, `cend()` — C++11 起推荐使用

- `crbegin()`, `crend()` — 反向 const 迭代器

### 5. 迭代器失效问题

- `vector` 扩容导致所有迭代器失效

- `vector` / `deque` 插入或删除元素后，被操作位置之后的迭代器失效

- `list` 删除元素后，只有被删除元素的迭代器失效

- `map` / `set` 删除元素后，只有被删除元素的迭代器失效

- `unordered_map` / `unordered_set` rehash 导致所有迭代器失效

- 安全删除元素的正确写法：使用 `erase()` 的返回值

### 6. 迭代器辅助函数

- `std::advance(it, n)` — 将迭代器前进 n 步

- `std::distance(first, last)` — 计算两个迭代器之间的距离

- `std::next(it, n)` — 返回前进 n 步后的迭代器（C++11）

- `std::prev(it, n)` — 返回后退 n 步后的迭代器（C++11）

---

## 四、实用工具补充

### 1. `std::tuple`（元组）

- **构造**

  - `tuple<int, string, double> t(1, "hello", 3.14)`

  - `make_tuple()` 创建元组

- **元素访问**

  - `std::get<0>(t)` — 按索引访问

  - `std::get<int>(t)` — 按类型访问（C++14，类型必须唯一）

- **解包**

  - `std::tie(a, b, c) = t` — 将元组解包到变量

  - `std::ignore` — 忽略某个元素

  - 结构化绑定：`auto [x, y, z] = t`（C++17）

- **其他**

  - `tuple_size<>` — 获取元组元素个数

  - `tuple_element<>` — 获取某个位置的类型

### 2. `std::bitset`（位集合）

- **构造**

  - 从整数构造：`bitset<8> b(42)`

  - 从字符串构造：`bitset<8> b("10101010")`

- **常用操作**

  - `set()`, `reset()`, `flip()` — 置位/复位/翻转

  - `test(pos)` — 测试某一位

  - `count()` — 统计 1 的个数

  - `any()`, `none()`, `all()` — 判断是否有 1 / 全是 0 / 全是 1

  - `to_ulong()`, `to_string()` — 转换

  - 位运算符：`&`, `|`, `^`, `~`, `<<`, `>>`

---

## 五、容器选择指南

### 1. 各容器操作的时间复杂度对比

- 随机访问：`vector` O(1)、`deque` O(1)、`list` O(n)

- 头部插入删除：`vector` O(n)、`deque` O(1)、`list` O(1)

- 尾部插入删除：`vector` 均摊 O(1)、`deque` O(1)、`list` O(1)

- 中间插入删除：`vector` O(n)、`list` O(1)（已知位置）

- 查找：`set`/`map` O(log n)、`unordered_set`/`unordered_map` 平均 O(1)

### 2. 如何根据场景选择容器

- 需要频繁随机访问 → `vector`

- 需要频繁头尾插入 → `deque`

- 需要频繁在任意位置插入删除 → `list`

- 需要快速查找且不关心顺序 → `unordered_set` / `unordered_map`

- 需要有序存储且快速查找 → `set` / `map`

- 需要按优先级取元素 → `priority_queue`

- 需要固定大小数组 → `array`

# 调整后的完整学习路线

下面我把你原课程的编号保留，**新增补充内容用 `[补]` 标记**，插入在最合理的学习位置。

---

## 第一阶段：STL 入门

**20** STL初识 - vector存放内置数据类型
**21** STL初识 - vector存放自定义数据类型
**22** STL初识 - 容器嵌套容器

### `[补] 迭代器专题`

> 原课程全程缺少对迭代器的系统讲解，放在这里是因为你刚接触了 `vector` 的迭代器用法，趁热打铁建立系统认知，后面学所有容器和算法都会更顺畅。

- `[补]` 迭代器的五种分类（输入 / 输出 / 前向 / 双向 / 随机访问）

- `[补]` 各容器对应的迭代器类型

- `[补]` 反向迭代器：`rbegin()`, `rend()`

- `[补]` const 迭代器：`cbegin()`, `cend()`, `crbegin()`, `crend()`

- `[补]` 迭代器辅助函数：`std::advance()`, `std::distance()`, `std::next()`, `std::prev()`

> ⚠️ 迭代器失效问题先不讲，等学完各容器的插入删除后再回来集中学习。

---

## 第二阶段：string 容器

**23** string容器 - 构造函数
**24** string容器 - 赋值操作
**25** string容器 - 字符串拼接
**26** string容器 - 字符串查找和替换
**27** string容器 - 字符串比较
**28** string容器 - 字符存取
**29** string容器 - 字符串插入和删除
**30** string容器 - 子串获取

---

## 第三阶段：vector 容器

**31** vector容器 - 构造函数
**32** vector容器 - 赋值操作
**33** vector容器 - 容量和大小
**34** vector容器 - 插入和删除
**35** vector容器 - 数据存取
**36** vector容器 - 互换容器
**37** vector容器 - 预留空间

---

## 第四阶段：array 容器（补充）

> `array` 和 `vector` 都是连续内存的序列容器，放在 `vector` 之后对比学习效果最好。

- `[补]` array容器 - 基本概念与构造（与 C 风格数组的区别）

- `[补]` array容器 - 常用接口：`at()`, `operator[]`, `front()`, `back()`, `size()`, `fill()`, `swap()`, `data()`

---

## 第五阶段：deque 容器

**38** deque容器 - 构造函数
**39** deque容器 - 赋值操作
**40** deque容器 - 大小操作
**41** deque容器 - 插入和删除
**42** deque容器 - 数据存取
**43** deque容器 - 排序操作

---

## 第六阶段：案例实践 1

**44** STL案例1 - 评委打分

---

## 第七阶段：stack 容器

**45** stack容器 - 基本概念
**46** stack容器 - 常用接口

---

## 第八阶段：queue 容器

**47** queue容器 - 基本概念
**48** queue容器 - 常用接口

---

## 第九阶段：priority_queue 容器（补充）

> 原课程讲完了 `stack` 和 `queue`，却漏掉了同属容器适配器的 `priority_queue`，紧接着补上最自然。

- `[补]` priority_queue - 基本概念（大顶堆 / 小顶堆）

- `[补]` priority_queue - 构造函数（默认构造、指定比较器构造小顶堆、用已有容器初始化）

- `[补]` priority_queue - 常用接口：`push()`, `pop()`, `top()`, `size()`, `empty()`

- `[补]` priority_queue - 存放自定义数据类型（自定义比较器 / 重载 `operator<`）

---

## 第十阶段：list 容器

**49** list容器 - 基本概念
**50** list容器 - 构造函数
**51** list容器 - 赋值和交换
**52** list容器 - 大小操作
**53** list容器 - 插入和删除
**54** list容器 - 数据存取
**55** list容器 - 反转和排序
**56** list容器 - 排序案例

---

## 第十一阶段：forward_list 容器（补充）

> 刚学完 `list`（双向链表），紧接着学 `forward_list`（单向链表）做对比。

- `[补]` forward_list - 基本概念（与 `list` 的区别、只支持前向迭代器）

- `[补]` forward_list - 常用接口：`push_front()`, `pop_front()`, `insert_after()`, `erase_after()`, `before_begin()`

- `[补]` forward_list - 成员函数：`sort()`, `merge()`, `reverse()`, `unique()`

---

## 第十二阶段：set 容器

**57** set容器 - 构造和赋值
**58** set容器 - 大小和交换
**59** set容器 - 插入和删除
**60** set容器 - 查找和统计
**61** set容器 - set和multiset区别
**62** pair使用 - pair对组的创建
**63** set容器 - 内置类型指定排序规则
**64** set容器 - 自定义数据类型指定排序规则

---

## 第十三阶段：map 容器

**65** map容器 - 构造和赋值
**66** map容器 - 大小和交换
**67** map容器 - 插入和删除
**68** map容器 - 查找和统计
**69** map容器 - 排序

---

## 第十四阶段：unordered_set / unordered_map 容器（补充）

> 刚学完有序的 `set`/`map`，立刻学无序版本做对比，理解最深刻。

- `[补]` unordered_set - 基本概念（底层哈希表、与 `set` 的区别）

- `[补]` unordered_set - 构造和赋值（默认构造、初始化列表构造）

- `[补]` unordered_set - 插入和删除：`insert()`, `emplace()`, `erase()`, `clear()`

- `[补]` unordered_set - 查找和统计：`find()`, `count()`, `contains()`（C++20）

- `[补]` unordered_set - 大小操作：`size()`, `empty()`

- `[补]` unordered_multiset 简要了解

---

- `[补]` unordered_map - 基本概念（底层哈希表、与 `map` 的区别）

- `[补]` unordered_map - 构造和赋值（默认构造、初始化列表构造）

- `[补]` unordered_map - 插入和删除：`insert()`, `emplace()`, `erase()`, `operator[]`

- `[补]` unordered_map - 查找和统计：`find()`, `count()`, `contains()`（C++20）

- `[补]` unordered_map - 大小操作：`size()`, `empty()`

- `[补]` unordered_map - 哈希相关：桶的概念、`bucket_count()`, `load_factor()`、自定义类型的哈希函数

---

## 第十五阶段：实用工具补充

> 所有容器学完了，补上两个常用工具类型。

- `[补]` std::tuple - 构造、`make_tuple()`、`std::get<>()`、`std::tie()` 解包、结构化绑定（C++17）

- `[补]` std::bitset - 构造（整数/字符串）、`set()`, `reset()`, `flip()`, `test()`, `count()`, `any()`, `none()`, 位运算

---

## 第十六阶段：迭代器失效专题（补充）

> 所有容器的插入删除都学完了，现在是集中学习迭代器失效的最佳时机。

- `[补]` vector 迭代器失效（扩容导致全部失效、插入/删除导致后续失效）

- `[补]` deque 迭代器失效（首尾插入可能导致全部失效、中间插入全部失效）

- `[补]` list / forward_list 迭代器失效（只有被删除节点的迭代器失效）

- `[补]` set / map 迭代器失效（只有被删除节点的迭代器失效）

- `[补]` unordered_set / unordered_map 迭代器失效（rehash 导致全部失效）

- `[补]` 安全删除元素的正确写法：使用 `erase()` 的返回值

---

## 第十七阶段：容器选择指南（补充）

> 所有容器都学完了，做一次总回顾。

- `[补]` 各容器操作的时间复杂度对比（随机访问、头尾插入、中间插入、查找）

- `[补]` 如何根据场景选择合适的容器

---

## 第十八阶段：案例实践 2

**70** STL案例2 - 员工分组

---

## 第十九阶段：函数对象与谓词

**71** 函数对象 - 函数对象基本使用
**72** 谓词 - 一元谓词
**73** 谓词 - 二元谓词
**74** 内建函数对象 - 算术仿函数
**75** 内建函数对象 - 关系仿函数
**76** 内建函数对象 - 逻辑仿函数

---

## 第二十阶段：常用遍历算法

**77** 常用遍历算法 - for_each
**78** 常用遍历算法 - transform
**79** 常用遍历算法 - find

---

## 第二十一阶段：常用查找算法

**80** 常用查找算法 - find_if
**81** 常用查找算法 - adjacent_find
**82** 常用查找算法 - binary_search
**83** 常用查找算法 - count
**84** 常用查找算法 - count_if

### `[补] 二分查找进阶`

> `binary_search` 只能判断"存不存在"，而 `lower_bound` 和 `upper_bound` 能告诉你"在哪里"，紧跟在 `binary_search` 后面学是最佳时机。

- `[补]` lower_bound(first, last, value) — 查找第一个 ≥ value 的位置

- `[补]` upper_bound(first, last, value) — 查找第一个 > value 的位置

- `[补]` equal_range(first, last, value) — 返回等于 value 的范围

- `[补]` 以上三者的自定义比较器版本

### `[补] 最值算法`

- `[补]` min_element(first, last) — 查找最小元素

- `[补]` max_element(first, last) — 查找最大元素

- `[补]` minmax_element(first, last) — 同时查找最小和最大元素（C++11）

---

## 第二十二阶段：常用排序算法

**85** 常用排序算法 - sort
**86** 常用排序算法 - random_shuffle
**87** 常用排序算法 - merge
**88** 常用排序算法 - reverse

### `[补] 排序算法进阶`

> 刚学完 `sort`，趁热补上其他排序变体。

- `[补]` stable_sort(first, last) — 稳定排序（相等元素保持原有顺序）

- `[补]` partial_sort(first, middle, last) — 只排出前 N 小的元素

- `[补]` nth_element(first, nth, last) — 找第 n 小的元素（快速选择）

### `[补] 排列算法`

- `[补]` next_permutation(first, last) — 生成下一个字典序排列

- `[补]` prev_permutation(first, last) — 生成上一个字典序排列

---

## 第二十三阶段：常用拷贝和替换算法

**89** 常用拷贝和替换算法 - copy
**90** 常用拷贝和替换算法 - replace
**91** 常用拷贝和替换算法 - replace_if
**92** 常用拷贝和替换算法 - swap

### `[补] 去重与移除算法`

> 跟 `replace` / `replace_if` 是同一类"修改序列"的算法，放在一起学。

- `[补]` remove(first, last, value) — 移除等于 value 的元素（逻辑移除）

- `[补]` remove_if(first, last, pred) — 按谓词移除元素

- `[补]` **erase-remove 惯用法**：`v.erase(remove(v.begin(), v.end(), val), v.end())`

- `[补]` unique(first, last) — 去除相邻重复元素（需先排序）

- `[补]` **erase-unique 惯用法**：`v.erase(unique(v.begin(), v.end()), v.end())`

### `[补] 分区与旋转算法`

- `[补]` partition(first, last, pred) — 按条件将元素分成两组

- `[补]` stable_partition(first, last, pred) — 稳定分区（保持组内相对顺序）

- `[补]` rotate(first, middle, last) — 旋转序列

---

## 第二十四阶段：常用算术生成算法

**93** 常用算术生成算法 - accumulate
**94** 常用算术生成算法 - fill

---

## 第二十五阶段：常用集合算法

**95** 常用集合算法 - set_intersection
**96** 常用集合算法 - set_union
**97** 常用集合算法 - set_difference

---

## 总览

整个路线的调整逻辑：

1. **迭代器专题**插在 STL 初识之后 — 先建立全局认知

2. **`array`** 插在 `vector` 之后 — 同类对比

3. **`priority_queue`** 插在 `queue` 之后 — 同属容器适配器，不能断开

4. **`forward_list`** 插在 `list` 之后 — 链表对比

5. **`unordered_set/map`** 插在 `set/map` 之后 — 有序 vs 无序对比

6. **`tuple` / `bitset`** 插在所有容器之后 — 工具补充

7. **迭代器失效** 插在所有容器学完之后 — 需要对所有容器都有认知才能理解

8. **容器选择指南** 作为容器部分的收尾总结

9. **缺失算法**分别插在对应的原课程算法类别之后 — `lower_bound` 跟在 `binary_search` 后面，`stable_sort` 跟在 `sort` 后面，`remove` 跟在 `replace` 后面，逻辑连贯

### 🔴 优先级最高（影响后续理解）

1. **vector 的扩容机制** - 为什么 1.5 倍或 2 倍？内存分配策略？
2. **迭代器失效场景** - 必须写代码验证，不要只看书
3. **哈希表原理** - `unordered_map` 的 bucket、load_factor、rehash
4. **仿函数 vs Lambda** - 捕获列表、mutable 关键字

### 🟡 值得深挖的细节

- `priority_queue` 的底层堆调整过程（`push_heap`/`pop_heap`）
- `list` vs `forward_list` 的内存占用差异（节点结构）
- `deque` 的分段连续存储 vs `vector` 的完全连续存储
- 各种算法的复杂度保证（C++ 标准有明确规定）