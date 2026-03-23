# 📘 迭代器专题（补充）

> 原课程全程缺少对迭代器的系统讲解，放在这里是因为你刚接触了 `vector` 的迭代器用法，趁热打铁建立系统认知，后面学所有容器和算法都会更顺畅。

---

## 一、迭代器是什么？——从“指针的泛化”说起

在学习 `vector` 时，你已经见过这样的代码：

```cpp
std::vector<int> v = {10, 20, 30};
for (std::vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
    std::cout << *it << " ";
}
```

你用 `*it` 取值、用 `++it` 前进、用 `!=` 比较——这些操作和**指针**一模一样。这不是巧合。

**迭代器（Iterator）本质上就是"行为像指针的对象"。** 它把"遍历容器"这件事从容器本身抽离出来，形成了一个统一的访问接口。正因如此：

- `std::sort()` 不需要知道你传的是 `vector` 还是 `deque`，它只需要"能随机跳跃的迭代器"。

- `std::find()` 不需要知道你传的是 `list` 还是 `set`，它只需要"能往前走一步的迭代器"。

这就是 STL 的核心设计哲学：**算法不直接操作容器，算法通过迭代器间接操作容器。** 迭代器是连接"容器"与"算法"的桥梁。

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  容器     │ ──▶ │  迭代器   │ ──▶ │  算法     │
│ Container │     │ Iterator │     │Algorithm │
└──────────┘     └──────────┘     └──────────┘
```

> 💡 **一句话记忆**：指针是最原始的迭代器；迭代器是被"升华"的指针。

---

## 二、迭代器的五种分类

并非所有迭代器都拥有相同的能力。你可以把迭代器理解为一条**四级主链**加一条**并列的输出分支**：

```
输入迭代器（Input）        ──  只读，单次前进
   ↓
前向迭代器（Forward）      ──  读写，可多次遍历
   ↓
双向迭代器（Bidirectional） ──  可前进，也可后退
   ↓
随机访问迭代器（Random Access）── 可跳跃到任意位置

输出迭代器（Output）       ──  只写，单次前进（并列分支，不在主链上）
```

下面逐一拆解。

---

### 2.1 输入迭代器（Input Iterator）

**能力**：只能**读取**元素，只能**单向前进**（`++`），且只能遍历**一次**。

**支持的操作**：

| 操作                    | 含义                           |
| ----------------------- | ------------------------------ |
| *it                     | 读取当前元素（只读）           |
| ++it / it++             | 前进到下一个元素               |
| it1 == it2 / it1 != it2 | 比较两个迭代器是否指向同一位置 |

**典型代表**：`std::istream_iterator` —— 从输入流逐个读取数据。

```cpp
#include <iostream>
#include <iterator>
#include <vector>
#include <algorithm>

int main() {
    std::cout << "请输入若干整数（以非数字结束）：" << std::endl;

    // istream_iterator<int>{std::cin} 是输入迭代器
    // 每次 ++ 就从 cin 读取下一个 int
    std::istream_iterator<int> input_begin(std::cin);  // 绑定到 cin
    std::istream_iterator<int> input_end;               // 默认构造 = 流结束哨兵

    std::vector<int> v(input_begin, input_end);  // 用迭代器范围构造 vector

    std::cout << "你输入了：";
    for (int x : v) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
}
```

> **为什么只能遍历一次？** 想象你从键盘读取数据——读过的字符就"消耗"了，你无法倒回去重新读。这就是"单次遍历"的含义。

---

### 2.2 输出迭代器（Output Iterator）

**能力**：只能**写入**元素，只能**单向前进**（`++`），只能遍历**一次**。

**支持的操作**：

| 操作        | 含义                     |
| ----------- | ------------------------ |
| *it = value | 向当前位置写入值（只写） |
| ++it / it++ | 前进到下一个写入位置     |

注意：输出迭代器**不支持读取**（`*it` 只能出现在赋值号左边），且通常是单遍语义，算法层面**不要求支持比较**（`==` / `!=`）。

**典型代表**：`std::ostream_iterator` —— 向输出流逐个写入数据。

```cpp
#include <iostream>
#include <iterator>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};

    // ostream_iterator<int>{std::cout, ", "} 是输出迭代器
    // 每次赋值就把值输出到 cout，并追加 ", "
    std::copy(v.begin(), v.end(),
              std::ostream_iterator<int>(std::cout, ", "));
    // 输出：10, 20, 30, 40, 50,
    std::cout << std::endl;
}
```

> 💡 **输入 vs 输出迭代器的记忆方法**：
>
> 
>
> - **输入**迭代器 = 数据**流入**程序（从外部读进来）
>
> - **输出**迭代器 = 数据**流出**程序（从程序写出去）

---

### 2.3 前向迭代器（Forward Iterator）

**能力**：可**读写**元素，只能**单向前进**（`++`），但可以**多次遍历**同一区间。

**支持的操作**：输入迭代器的所有操作 + 可以多次遍历（保存迭代器副本后，副本仍有效）。

```cpp
// 前向迭代器允许这样做：
auto saved = it;   // 保存当前位置
++it;               // 前进
// saved 仍然有效，可以回头访问之前保存的位置
std::cout << *saved << std::endl;
```

**典型代表**：`std::forward_list::iterator`、`std::unordered_set::iterator`、`std::unordered_map::iterator`。

**为什么 `forward_list` 只有前向迭代器？** 因为 `forward_list` 是单向链表——每个节点只存"下一个节点"的指针，根本不知道"上一个节点"在哪，自然无法后退。

```cpp
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> fl = {1, 2, 3, 4, 5};

    // forward_list 的迭代器只能 ++，不能 --
    for (auto it = fl.begin(); it != fl.end(); ++it) {
        std::cout << *it << " ";
    }
    // 输出：1 2 3 4 5
    std::cout << std::endl;

    // ❌ 编译错误：forward_list 的迭代器不支持 --
    // auto it = fl.end();
    // --it;
}
```

---

### 2.4 双向迭代器（Bidirectional Iterator）

**能力**：在前向迭代器的基础上，增加了**后退**（`--`）的能力。

**支持的操作**：前向迭代器的所有操作 + `--it` / `it--`。

**典型代表**：`std::list::iterator`、`std::set::iterator`、`std::map::iterator`。

```cpp
#include <list>
#include <iostream>

int main() {
    std::list<int> lst = {10, 20, 30, 40, 50};

    // 正向遍历
    std::cout << "正向：";
    for (auto it = lst.begin(); it != lst.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 反向遍历 —— 双向迭代器可以 --
    std::cout << "反向：";
    auto it = lst.end();
    while (it != lst.begin()) {
        --it;  // 先 --，因为 end() 指向的是"尾后位置"
        std::cout << *it << " ";
    }
    std::cout << std::endl;
    // 输出：
    // 正向：10 20 30 40 50
    // 反向：50 40 30 20 10
}
```

> **为什么 `list`（双向链表）不提供随机访问？** 因为链表节点在内存中不连续，想跳到第 N 个元素，必须从头一步一步走过去，做不到 O(1) 的随机跳跃。

---

### 2.5 随机访问迭代器（Random Access Iterator）

**能力**：在双向迭代器的基础上，增加了**随机跳跃**和**比较大小**的能力——这使得它与原生指针的行为几乎完全一致。

**支持的操作**：双向迭代器的所有操作 + 以下操作：

| 操作                 | 含义                     |
| -------------------- | ------------------------ |
| it + n / it - n      | 向前/向后跳跃 n 个位置   |
| it += n / it -= n    | 同上（就地修改）         |
| it[n]                | 等价于 *(it + n)         |
| it1 - it2            | 计算两个迭代器之间的距离 |
| it1 < it2、<=、>、>= | 比较两个迭代器的前后顺序 |

**典型代表**：`std::vector::iterator`、`std::deque::iterator`、`std::array::iterator`，以及**原生指针**。

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};

    auto it = v.begin();

    // 随机跳跃
    std::cout << *(it + 2) << std::endl;   // 30（跳到第3个元素）
    std::cout << it[3] << std::endl;       // 40（下标访问）

    // 计算距离
    auto it2 = v.end();
    std::cout << (it2 - it) << std::endl;  // 5（两个迭代器之间有5个元素）

    // 大小比较
    if (it < it2) {
        std::cout << "begin 在 end 前面" << std::endl;
    }
}
```

> 💡 **为什么 `std::sort()` 要求随机访问迭代器？** 因为快速排序、堆排序等高效排序算法需要在序列中来回跳跃（比如取中间元素 `it + (last - first) / 2`）。只能 `++` / `--` 一步一步走的迭代器，根本喂不饱这些算法。

---

### 2.6 五种迭代器的层级关系总结

```
                 ┌────────────────────┐
                 │  输入迭代器 (Input)  │  只读 + 单次 + 单向
                 └────────┬───────────┘
                          │ 能力递进（非类继承）
                 ┌────────▼───────────┐
                 │ 前向迭代器 (Forward) │  读写 + 多次 + 单向
                 └────────┬───────────┘
                          │
              ┌───────────▼────────────┐
              │双向迭代器 (Bidirectional)│  读写 + 多次 + 双向
              └───────────┬────────────┘
                          │
           ┌──────────────▼─────────────┐
           │随机访问迭代器 (Random Access)│  读写 + 多次 + 随机跳跃
           └────────────────────────────┘
```

输出迭代器（Output）比较特殊，它与输入迭代器是**并列关系**，不参与上面的能力层级链。可以把它理解为一条独立的"只写"分支。

> 🧠 **核心原则**：算法声明自己需要"最低等级"的迭代器类型。你传入的迭代器只要**等于或高于**这个等级，就能正常工作。例如 `std::find()` 只需要输入迭代器，所以你传 `vector` 的随机访问迭代器、`list` 的双向迭代器，统统没问题——因为它们都"至少"是输入迭代器。

---

## 三、各容器对应的迭代器类型

这是一张你今后会反复查阅的表。**记住容器的迭代器类型，就能立刻判断它能配合哪些算法使用。**

| 容器                             | 迭代器类型 | 能用 std::sort() 吗？                    |
| -------------------------------- | ---------- | ---------------------------------------- |
| std::array                       | 随机访问   | ✅ 能                                     |
| std::vector                      | 随机访问   | ✅ 能                                     |
| std::deque                       | 随机访问   | ✅ 能                                     |
| std::string                      | 随机访问   | ✅ 能                                     |
| std::list                        | 双向       | ❌ 不能（但 list 自带 .sort() 成员函数）  |
| std::forward_list                | 前向       | ❌ 不能（但自带 .sort() 成员函数）        |
| std::set / std::multiset         | 双向       | ❌ 不能（元素本已有序，也不允许修改）     |
| std::map / std::multimap         | 双向       | ❌ 不能（键值对本已有序，也不允许修改键） |
| std::unordered_set / std::unordered_multiset | 前向 | ❌ 不能 |
| std::unordered_map / std::unordered_multimap | 前向 | ❌ 不能 |
| std::stack                       | 无迭代器   | ❌ 不能（不可遍历）                       |
| std::queue / std::priority_queue | 无迭代器   | ❌ 不能（不可遍历）                       |

> 💡 **规律总结**：
>
> 
>
> - **连续内存**的容器（`array`、`vector`、`deque`、`string`）→ 随机访问迭代器
>
> - **双向链表**（`list`）和**有序关联容器**（`set`、`map`）→ 双向迭代器
>
> - **单向链表**（`forward_list`）和**无序关联容器**（`unordered_*`）→ 前向迭代器
>
> - **容器适配器**（`stack`、`queue`、`priority_queue`）→ 没有迭代器

---

## 四、反向迭代器：`rbegin()` 和 `rend()`

### 4.1 为什么需要反向迭代器？

有时候你需要**从后往前**遍历容器。虽然对于双向迭代器/随机访问迭代器，你可以用 `--` 来后退，但代码会很别扭：

```cpp
// 手动反向遍历 —— 不直观
auto it = v.end();
while (it != v.begin()) {
    --it;
    std::cout << *it << " ";
}
```

问题在于：`end()` 指向的是"尾后"（past-the-end），不能直接 `*it`；你必须先 `--it` 才能解引用。这个逻辑不对称，容易出错。

反向迭代器就是为了解决这个问题——**它让你用正常的 `++` 语法，实现从后往前的遍历**。

### 4.2 基本用法

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};

    // rbegin() 指向最后一个元素，rend() 指向第一个元素之前的位置
    std::cout << "反向遍历：";
    for (auto rit = v.rbegin(); rit != v.rend(); ++rit) {
        //                                       ^^^^ 注意：是 ++，不是 --
        std::cout << *rit << " ";
    }
    std::cout << std::endl;
    // 输出：反向遍历：50 40 30 20 10
}
```

### 4.3 正向迭代器 vs 反向迭代器的位置关系

理解反向迭代器的关键是搞清楚它和正向迭代器之间的**位置映射关系**。

```
元素:     [10]  [20]  [30]  [40]  [50]
                                         
正向:  begin                          end
         ↓                             ↓
位置:     0     1     2     3     4    (尾后)

反向:                                rbegin    rend
                                       ↓        ↓
逻辑:                                 50       (首前)
```

| 正向                 | 反向                    |
| -------------------- | ----------------------- |
| begin() → 第一个元素 | rbegin() → 最后一个元素 |
| end() → 尾后位置     | rend() → 首前位置       |
| ++it → 向右移动      | ++rit → 向左移动        |

### 4.4 反向迭代器与算法配合

反向迭代器通常可以传给接受对应**迭代器类别**的算法。这是因为反向迭代器在 `++` 时内部做了 `--`，对算法来说它依然表现为"向前移动"。

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};

    // 用反向迭代器找到最后一个值为 1 的元素
    auto rit = std::find(v.rbegin(), v.rend(), 1);
    if (rit != v.rend()) {
        // 计算它在 vector 中的正向下标
        // rit.base() 返回对应的正向迭代器（指向反向迭代器的下一个位置）
        auto dist = std::distance(v.begin(), rit.base()) - 1;
        std::cout << "最后一个1的位置（下标）：" << dist << std::endl;
        // 输出：最后一个1的位置（下标）：3
    }
}
```

### 4.5 `base()`：反向迭代器 → 正向迭代器

每个反向迭代器都有一个 `base()` 成员函数，可以将它转换回对应的正向迭代器。**但要注意偏移关系**：

```
元素:   [10]  [20]  [30]  [40]  [50]
位置:    0     1     2     3     4     (尾后)

若 rit 指向 [30]（即 *rit == 30）
则 rit.base() 指向 [40]（即 *(rit.base()) == 40）
```

**规则**：`rit.base()` 始终指向 `rit` 所指元素的**右边一个位置**（正向方向的下一个）。

这个设计确保了 `[rbegin, rend)` 和 `[begin, end)` 覆盖的是同一区间。

### 4.6 哪些容器支持反向迭代器？

**只有提供双向迭代器或随机访问迭代器的容器才有反向迭代器**，因为反向遍历需要 `--` 能力。

| 有反向迭代器                 | 无反向迭代器                                 |
| ---------------------------- | -------------------------------------------- |
| vector, deque, array, string | forward_list                                 |
| list                         | forward_list、unordered_set、unordered_map 及其 multi 版本 |
| set, map, multiset, multimap | stack, queue, priority_queue                 |

---

## 五、const 迭代器：`cbegin()` / `cend()` / `crbegin()` / `crend()`

### 5.1 为什么需要 const 迭代器？

普通迭代器（`iterator`）允许你通过 `*it = value` **修改**容器中的元素。但有时候你只想**只读地遍历**容器——比如在一个接收 `const` 引用参数的函数中：

```cpp
void print(const std::vector<int>& v) {
    // v 是 const 的，所以 v.begin() 返回的是 const_iterator
    // 编译器不允许你通过 const_iterator 修改元素
    for (auto it = v.begin(); it != v.end(); ++it) {
        std::cout << *it << " ";  // ✅ 只读访问，OK
        // *it = 100;             // ❌ 编译错误！不能通过 const_iterator 修改
    }
}
```

在上面的代码中，因为 `v` 本身是 `const` 的，`v.begin()` 自动返回 `const_iterator`，编译器替你做了保护。

但如果 `v` 不是 `const` 的呢？

```cpp
std::vector<int> v = {1, 2, 3};
auto it = v.begin();  // 这里 it 是 iterator，不是 const_iterator
*it = 100;            // 合法！你可能不小心就改了
```

如果你想在**非 const 容器**上也获得只读保护，就需要**显式使用 `cbegin()` / `cend()`**。

### 5.2 四个 const 迭代器函数

| 函数      | 返回类型               | 方向 | 含义                     |
| --------- | ---------------------- | ---- | ------------------------ |
| cbegin()  | const_iterator         | 正向 | 指向第一个元素（只读）   |
| cend()    | const_iterator         | 正向 | 指向尾后位置（只读）     |
| crbegin() | const_reverse_iterator | 反向 | 指向最后一个元素（只读） |
| crend()   | const_reverse_iterator | 反向 | 指向首前位置（只读）     |

### 5.3 完整示例

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};

    // 1. 正向 const 遍历
    std::cout << "正向只读：";
    for (auto it = v.cbegin(); it != v.cend(); ++it) {
        std::cout << *it << " ";
        // *it = 0;  // ❌ 编译错误：const_iterator 不允许修改
    }
    std::cout << std::endl;

    // 2. 反向 const 遍历
    std::cout << "反向只读：";
    for (auto it = v.crbegin(); it != v.crend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;
    // 输出：
    // 正向只读：10 20 30 40 50
    // 反向只读：50 40 30 20 10
}
```

### 5.4 `auto` 与 const 迭代器的"陷阱"

```cpp
std::vector<int> v = {1, 2, 3};

auto it1 = v.begin();    // 推导为 iterator        → 可读可写
auto it2 = v.cbegin();   // 推导为 const_iterator  → 只读

const auto it3 = v.begin(); // ⚠️ 这是 const iterator，不是 const_iterator！
```

**注意区分两个概念**：

| 写法           | 含义                    | 迭代器本身能移动吗？    | 能修改指向的元素吗？       |
| -------------- | ----------------------- | ----------------------- | -------------------------- |
| const_iterator | 指向 const 元素的迭代器 | ✅ 能（++it）            | ❌ 不能（*it = x 编译失败） |
| const iterator | 迭代器本身是 const 的   | ❌ 不能（++it 编译失败） | ✅ 能（*it = x 合法）       |

这就好比指针：

```cpp
const int* p1 = &x;   // 指向 const int 的指针 → 不能改 *p，能改 p
int* const p2 = &x;   // const 指针 → 能改 *p，不能改 p
```

`const_iterator` 对应 `const int*`，`const iterator` 对应 `int* const`。

### 5.5 最佳实践

> 📌 **当你只需要读、不需要写的时候，优先使用 `cbegin()` / `cend()`。** 这是现代 C++ 的推荐做法——把"只读意图"显式表达在代码中，编译器会帮你守住底线。

---

## 六、迭代器辅助函数

标准库在 `<iterator>` 头文件中提供了四个常用的迭代器辅助函数。它们的存在是为了**屏蔽不同迭代器类型的能力差异**，让你写出通用的代码。

### 6.1 `std::advance(it, n)` —— 移动迭代器

**功能**：将迭代器 `it` 移动 `n` 个位置（`n` 为正则前进，为负则后退）。**注意：它直接修改 `it`，没有返回值。**

**为什么不直接用 `it + n`？** 因为 `it + n` 只有随机访问迭代器才支持！对于双向迭代器或前向迭代器，你只能一步一步 `++` / `--`。`std::advance()` 帮你做了这件事，并且**对随机访问迭代器自动优化为 `it += n`**（O(1)），对其他迭代器则逐步移动（O(n)）。

```cpp
#include <list>
#include <iterator>
#include <iostream>
#include <algorithm>

int main() {
    std::list<int> lst = {10, 20, 30, 40, 50};

    auto it = lst.begin();   // 指向 10
    std::advance(it, 3);     // 前进 3 步 → 指向 40
    std::cout << *it << std::endl;  // 输出：40

    std::advance(it, -2);    // 后退 2 步 → 指向 20
    std::cout << *it << std::endl;  // 输出：20

    // 对比：如果用 list 的迭代器写 it + 3
    // auto it2 = lst.begin() + 3;  // ❌ 编译错误！list 是双向迭代器，不支持 +
}
```

| 迭代器类型 | advance 的时间复杂度 |
| ---------- | -------------------- |
| 随机访问   | O(1)                 |
| 双向       | O(n)                 |
| 前向       | O(n)，且 n 必须 ≥ 0  |

> ⚠️ **对前向迭代器使用负数 `n` 是未定义行为**。这很合理——前向迭代器根本不能后退。

---

### 6.2 `std::distance(first, last)` —— 计算距离

**功能**：返回从 `first` 到 `last` 的元素个数（即需要多少次 `++first` 才能到达 `last`）。

**为什么不直接用 `last - first`？** 同理——减法运算只有随机访问迭代器才支持。

```cpp
#include <list>
#include <set>
#include <iterator>
#include <algorithm>
#include <iostream>

int main() {
    // 示例1：用于 list
    std::list<int> lst = {10, 20, 30, 40, 50};
    auto it1 = lst.begin();
    auto it2 = lst.end();
    std::cout << "list 的元素个数：" << std::distance(it1, it2) << std::endl;
    // 输出：5

    // 示例2：找到某个元素后，计算它的"下标"
    auto found = std::find(lst.begin(), lst.end(), 30);
    if (found != lst.end()) {
        std::cout << "30 是第 " << std::distance(lst.begin(), found) << " 个元素（从0开始）"
                  << std::endl;
        // 输出：30 是第 2 个元素（从0开始）
    }

    // 示例3：用于 set（双向迭代器）
    std::set<int> s = {5, 3, 1, 4, 2};
    std::cout << "set 的元素个数：" << std::distance(s.begin(), s.end()) << std::endl;
    // 输出：5
}
```

| 迭代器类型  | distance 的时间复杂度         |
| ----------- | ----------------------------- |
| 随机访问    | O(1)（内部直接 last - first） |
| 双向 / 前向 | O(n)（内部逐步 ++）           |

> ⚠️ **`first` 必须能通过 `++` 到达 `last`**。如果 `first` 在 `last` 后面（对于非随机访问迭代器），行为是**未定义**的。

---

### 6.3 `std::next(it, n = 1)` —— 返回前进后的迭代器（C++11）

**功能**：返回 `it` 前进 `n` 步后的**新迭代器**，**不修改原始迭代器**。`n` 默认为 1。

**与 `advance` 的关键区别**：

|                | std::advance(it, n) | std::next(it, n)                       |
| -------------- | ------------------- | -------------------------------------- |
| 修改原迭代器？ | ✅ 是（就地修改）    | ❌ 否（返回新的）                       |
| 返回值         | void                | 新的迭代器                             |
| 适用场景       | 需要移动迭代器本身  | 需要"看一眼后面的元素"同时保留当前位置 |

```cpp
#include <list>
#include <iterator>
#include <iostream>

int main() {
    std::list<int> lst = {10, 20, 30, 40, 50};

    auto it = lst.begin();                      // 指向 10
    auto it3 = std::next(it, 3);                // it3 指向 40，it 仍然指向 10
    std::cout << "*it = " << *it << std::endl;  // 输出：*it = 10（未被修改）
    std::cout << "*it3 = " << *it3 << std::endl;// 输出：*it3 = 40

    // 常见用法：获取倒数第二个元素
    auto second_last = std::next(lst.end(), -2);// 等价于 prev(lst.end(), 2)
    // 注意：n 为负时要求至少是双向迭代器；前向迭代器不能这样用
    // 但更推荐用 std::prev()，语义更清晰（见下面）

    // 常见用法：在算法中需要"从第二个元素开始"
    int sum_from_second = 0;
    std::for_each(std::next(lst.begin()), lst.end(),
                  [&sum_from_second](int n) { sum_from_second += n; });
}
```

---

### 6.4 `std::prev(it, n = 1)` —— 返回后退后的迭代器（C++11）

**功能**：返回 `it` 后退 `n` 步后的**新迭代器**，**不修改原始迭代器**。`n` 默认为 1。

**要求**：至少是**双向迭代器**（因为需要 `--` 能力）。

```cpp
#include <vector>
#include <iterator>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};

    // 获取最后一个元素
    auto last = std::prev(v.end());
    std::cout << "最后一个元素：" << *last << std::endl;  // 输出：50

    // 获取倒数第三个元素
    auto third_from_end = std::prev(v.end(), 3);
    std::cout << "倒数第三个元素：" << *third_from_end << std::endl;  // 输出：30

    // 实际用途：排除首尾元素进行处理
    // std::sort(std::next(v.begin()), std::prev(v.end()));
    // → 只对 v[1] 到 v[size-2] 排序，保留首尾元素不动
}
```

---

### 6.5 四个辅助函数的对比总结

| 函数                       | 功能      | 修改原迭代器？ | 返回值         | 最低迭代器要求                |
| -------------------------- | --------- | -------------- | -------------- | ----------------------------- |
| std::advance(it, n)        | 移动 n 步 | ✅ 是           | void           | 前向（n ≥ 0）/ 双向（n 可负） |
| std::distance(first, last) | 计算距离  | ❌ 否           | 距离值（整数） | 前向                          |
| std::next(it, n)           | 前进 n 步 | ❌ 否           | 新迭代器       | 前向                          |
| std::prev(it, n)           | 后退 n 步 | ❌ 否           | 新迭代器       | 双向                          |

> 💡 **经验法则**：
>
> 
>
> - 需要**就地移动**迭代器 → 用 `std::advance()`
>
> - 需要**保留原位**、只是看一眼其他位置 → 用 `std::next()` / `std::prev()`
>
> - 需要**算两个迭代器之间隔了多远** → 用 `std::distance()`

---

## 七、综合实战：把今天学的所有知识串起来

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <string>
#include <iterator>  // advance, distance, next, prev
#include <algorithm> // find, copy

int main() {
    // ============================
    // 1. vector —— 随机访问迭代器
    // ============================
    std::vector<int> v = {5, 3, 8, 1, 9, 2, 7, 4, 6};

    // (a) 正向遍历
    std::cout << "=== vector 正向 ===" << std::endl;
    for (auto it = v.cbegin(); it != v.cend(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // (b) 反向遍历
    std::cout << "=== vector 反向 ===" << std::endl;
    for (auto rit = v.crbegin(); rit != v.crend(); ++rit) {
        std::cout << *rit << " ";
    }
    std::cout << std::endl;

    // (c) 随机访问：直接跳到第5个元素
    auto it_v = v.begin();
    std::cout << "第5个元素（下标4）：" << it_v[4] << std::endl;
    std::cout << "第5个元素（用next）：" << *std::next(it_v, 4) << std::endl;

    // ============================
    // 2. list —— 双向迭代器
    // ============================
    std::list<std::string> words = {"Hello", "Modern", "C++", "Iterator", "World"};

    // (a) 找到 "C++" 的位置，计算它是第几个元素
    auto found = std::find(words.cbegin(), words.cend(), "C++");
    if (found != words.cend()) {
        auto pos = std::distance(words.cbegin(), found);
        std::cout << "\"C++\" 是第 " << pos << " 个元素（从0开始）" << std::endl;
    }

    // (b) 用 advance 跳到第4个元素
    auto it_list = words.begin();
    std::advance(it_list, 3);
    std::cout << "第4个单词：" << *it_list << std::endl;  // Iterator

    // (c) 用 prev 获取最后一个元素
    auto last_word = std::prev(words.end());
    std::cout << "最后一个单词：" << *last_word << std::endl;  // World

    // (d) 反向遍历 list
    std::cout << "=== list 反向 ===" << std::endl;
    for (auto rit = words.crbegin(); rit != words.crend(); ++rit) {
        std::cout << *rit << " ";
    }
    std::cout << std::endl;

    // ============================
    // 3. 用输出迭代器优雅地输出
    // ============================
    std::cout << "=== ostream_iterator 输出 ===" << std::endl;
    std::copy(v.cbegin(), v.cend(),
              std::ostream_iterator<int>(std::cout, " "));
    std::cout << std::endl;

    return 0;
}
```

**预期输出**：

```
=== vector 正向 ===
5 3 8 1 9 2 7 4 6
=== vector 反向 ===
6 4 7 2 9 1 8 3 5
第5个元素（下标4）：9
第5个元素（用next）：9
"C++" 是第 2 个元素（从0开始）
第4个单词：Iterator
最后一个单词：World
=== list 反向 ===
World Iterator C++ Modern Hello
=== ostream_iterator 输出 ===
5 3 8 1 9 2 7 4 6
```

---

## 八、本节知识图谱

```
迭代器专题
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
      五种分类            方向变体           辅助函数
          │                  │                  │
   ┌──────┼──────┐     ┌────┼────┐      ┌──────┼──────┐
   │      │      │     │         │      │      │      │
 Input  Forward  │   rbegin   cbegin  advance  next  distance
 Output Bidir    │   rend     cend    (就地)   prev  (计算)
      RandomAccess    crbegin        (返回新)
                       crend
```

---

## 九、常见误区提醒

### ❌ 误区 1：对前向迭代器使用 `--` 或负数 `advance`

```cpp
std::forward_list<int> fl = {1, 2, 3};
auto it = fl.begin();
std::advance(it, 1);
std::advance(it, -1);  // ❌ 未定义行为！前向迭代器不能后退
--it;                   // ❌ 编译错误！
```

### ❌ 误区 2：对非随机访问迭代器使用 `+`、`-`、`<`

```cpp
std::list<int> lst = {1, 2, 3};
auto it = lst.begin() + 2;     // ❌ 编译错误！list 迭代器不支持 +
auto d = lst.end() - lst.begin();  // ❌ 编译错误！不支持 -
if (lst.begin() < lst.end())   // ❌ 编译错误！不支持 <
```

正确做法：用辅助函数。

```cpp
auto it = lst.begin();
std::advance(it, 2);                            // ✅
auto d = std::distance(lst.begin(), lst.end());  // ✅
```

### ❌ 误区 3：混淆 `const_iterator` 和 `const iterator`

```cpp
std::vector<int> v = {1, 2, 3};

std::vector<int>::const_iterator cit = v.begin();
// *cit = 10;  // ❌ 不能修改元素
++cit;          // ✅ 能移动

const std::vector<int>::iterator it = v.begin();
*it = 10;       // ✅ 能修改元素
// ++it;        // ❌ 不能移动，因为 it 本身是 const
```

---

## 十、写在最后

迭代器是 STL 的**中枢神经**。整个 STL 的设计可以浓缩为一句话：

> **容器管存储，算法管计算，迭代器管连接。**

现在你已经掌握了：

1. **五种迭代器的分类与层级**——知道每种迭代器能做什么、不能做什么

2. **各容器对应的迭代器类型**——知道一个容器能配合哪些算法使用

3. **反向迭代器**——可以优雅地从后往前遍历

4. **const 迭代器**——"只读意图"的显式表达

5. **四个辅助函数**——让你的代码在所有迭代器类型上都能通用

接下来学习其他容器（`deque`、`list`、`set`、`map` 等）时，你会自然而然地把它们和今天学的迭代器类型对应起来，形成系统性的理解。

> ⚠️ **迭代器失效问题**（即在插入/删除元素后，哪些迭代器会变成"野指针"）是另一个重要主题，等学完各容器的插入删除操作后再集中讲解——那时候你有了足够的"案发现场"，理解起来会更加透彻。


