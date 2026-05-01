# 第十六阶段：迭代器失效专题（补充）

> 📌 **前言**：你已经学完了所有主要容器的插入与删除操作。现在，是时候系统性地理解一个**极其重要且极易出错**的话题——**迭代器失效（Iterator Invalidation）**。这个话题是 C++ 面试的高频考点，更是实际开发中 Bug 的重灾区。本章将逐容器、逐场景地剖析失效规则，并在最后给出**安全删除元素的标准写法**。

---

## 一、什么是迭代器失效？

### 1.1 迭代器的本质

在深入失效规则之前，我们先回顾迭代器的本质。迭代器本质上是**对容器内部数据位置的一种引用/封装**。不同容器的迭代器实现方式截然不同：

| 容器                              | 迭代器本质                                              |
| --------------------------------- | ------------------------------------------------------- |
| `vector`                          | 封装了**指向连续内存的原始指针**                        |
| `deque`                           | 封装了指向**分段连续内存**（缓冲区）的指针 + 控制块信息 |
| `list`                            | 封装了指向**链表节点**的指针                            |
| `set` / `map`                     | 封装了指向**红黑树节点**的指针                          |
| `unordered_set` / `unordered_map` | 封装了指向**哈希桶中节点**的指针                        |

### 1.2 失效的含义

所谓"迭代器失效"，是指**迭代器所指向的内存位置不再有效**，或者**迭代器所指向的元素已经不是你期望的那个元素**。

对失效的迭代器进行解引用（`*it`）、递增（`++it`）、比较（`it != end()`）等操作，都是**未定义行为（Undefined Behavior）**，可能导致：
- 程序崩溃（段错误）
- 读取到垃圾数据
- 程序"看起来正常"但实际结果错误（最危险！）

### 1.3 引发失效的两大操作

迭代器失效几乎总是由以下两类操作引发：

1. **插入元素**（`insert`、`push_back`、`emplace` 等）
2. **删除元素**（`erase`、`pop_back`、`clear` 等）

不同容器由于底层数据结构不同，失效规则也完全不同。下面我们逐一剖析。

---

## 二、`vector` 迭代器失效

`vector` 是迭代器失效问题**最严重**的容器，因为它的底层是**一段连续内存**。

### 2.1 底层内存模型回顾

```
vector 内部状态：
┌──────────────────────────────────────────────────┐
│  [0] [1] [2] [3] [4]  ·  ·  ·                   │
│   ↑                 ↑              ↑             │
│ begin()           size()      capacity()         │
└──────────────────────────────────────────────────┘
        已使用的元素          预留的空闲空间
```

`vector` 有两个关键概念：

- **`size()`**：当前实际存储的元素个数
- **`capacity()`**：当前已分配内存能容纳的元素个数

当 `size() == capacity()` 时再插入元素，就会触发**扩容（reallocation）**。

### 2.2 场景一：扩容导致**全部迭代器失效**

这是最猛烈的一种失效。

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3};
    // 假设此时 capacity() == 3（刚好满了）

    auto it = v.begin() + 1;  // 指向元素 2
    cout << "扩容前: *it = " << *it << endl;  // 输出 2

    v.push_back(4);  // size 超过 capacity，触发扩容！

    // ❌ 此时 it 已经失效！
    // 旧内存已被释放，it 仍指向旧内存
    // cout << *it;  // 未定义行为！可能崩溃或输出垃圾值

    return 0;
}
```

**扩容时发生了什么？**

```
扩容前：
旧内存：[1] [2] [3]
              ↑
             it（指向旧内存中的位置）

扩容过程：
1. 分配一块更大的新内存（通常是原来的 1.5 倍或 2 倍）
2. 将旧元素逐个移动/拷贝到新内存
3. 释放旧内存 ← it 指向的内存被释放了！

扩容后：
旧内存：[已释放] ← it 还指着这里（悬垂指针！）
新内存：[1] [2] [3] [4]
```

> ⚠️ **关键结论**：`vector` 扩容后，**所有**迭代器、指针、引用**全部失效**，无一幸免。

### 2.3 场景二：不扩容时，插入点之后的迭代器失效

如果 `capacity()` 足够大，不需要扩容，那情况会好一些，但仍有部分迭代器失效：

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5};
    v.reserve(10);  // 预留足够空间，确保不扩容

    auto it1 = v.begin();      // 指向 1
    auto it2 = v.begin() + 2;  // 指向 3
    auto it4 = v.begin() + 4;  // 指向 5

    // 在位置 2（即元素 3 之前）插入 99
    v.insert(v.begin() + 2, 99);
    // 内存变化：[1] [2] [99] [3] [4] [5]

    // ✅ it1 仍然有效，指向 1（在插入点之前）
    cout << *it1 << endl;  // 输出 1

    // ❌ it2 原本指向位置 2，但位置 2 现在是 99
    //    它的值已经"变了"——虽然指针本身没有"悬垂"，
    //    但它指向的元素已经不是预期的了
    // 标准规定：插入点及之后的迭代器全部失效

    // ❌ it4 同理，也已失效

    return 0;
}
```

**图解不扩容时的插入：**

```
插入前：[1] [2] [3] [4] [5] · · · · ·
        ↑       ↑               ↑
       it1     it2             it4

在 begin()+2 处插入 99：
[1] [2] [99] [3] [4] [5] · · · ·
 ↑        ↑                  ↑
it1      it2                it4
         （值变了！）       （值变了！）

it1 在插入点之前 → ✅ 有效
it2 在插入点处   → ❌ 失效（指向的值从 3 变成了 99）
it4 在插入点之后 → ❌ 失效（指向的值从 5 变成了 4）
```

### 2.4 场景三：删除元素导致删除点之后的迭代器失效

删除不会导致扩容，但会导致元素前移：

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    auto it1 = v.begin();      // 指向 10
    auto it2 = v.begin() + 2;  // 指向 30
    auto it3 = v.begin() + 3;  // 指向 40
    auto it4 = v.begin() + 4;  // 指向 50

    // 删除位置 2 的元素（即 30）
    v.erase(v.begin() + 2);
    // 内存变化：[10] [20] [40] [50]

    // ✅ it1 仍然有效（在删除点之前）
    cout << *it1 << endl;  // 输出 10

    // ❌ it2 失效（原本指向 30，现在位置 2 是 40）
    // ❌ it3 失效（原本指向 40，现在位置 3 是 50）
    // ❌ it4 失效（原本指向 50，现在该位置已越界！）

    return 0;
}
```

**图解删除：**

```
删除前：[10] [20] [30] [40] [50]
          ↑         ↑    ↑    ↑
         it1       it2  it3  it4

删除 begin()+2（即 30）后：
[10] [20] [40] [50]
  ↑         ↑    ↑    ↑
 it1       it2  it3  it4（越界！）

it1 在删除点之前 → ✅ 有效
it2 位于删除点   → ❌ 失效
it3 在删除点之后 → ❌ 失效
it4 在删除点之后 → ❌ 失效（更危险，越界访问！）
```

### 2.5 `vector` 迭代器失效规则总结

| 操作                                    | 是否扩容 | 失效范围                            |
| --------------------------------------- | -------- | ----------------------------------- |
| `push_back` / `emplace_back` / `insert` | 触发扩容 | **所有**迭代器、指针、引用失效      |
| `push_back` / `emplace_back`            | 不扩容   | 只有 `end()` 失效                   |
| `insert(pos, ...)`                      | 不扩容   | `pos` 及之后的所有迭代器失效        |
| `erase(pos)`                            | —        | `pos` 及之后的所有迭代器失效        |
| `pop_back()`                            | —        | 只有末尾元素的迭代器和 `end()` 失效 |
| `clear()`                               | —        | **所有**迭代器失效                  |

> 💡 **黄金法则**：对 `vector` 做了插入/删除后，不要再使用之前保存的迭代器。要么重新获取，要么用 `erase()` 的返回值。

---

## 三、`deque` 迭代器失效

### 3.1 底层内存模型回顾

`deque`（双端队列）的底层是一个**分段连续**的结构：一个中控数组（`map`）指向多个固定大小的缓冲区（`buffer`）。

```
中控数组 map：
┌───┬───┬───┬───┬───┐
│ · │ · │ · │ · │ · │
└─┬─┴─┬─┴─┬─┴─┬─┴─┬─┘
  │   │   │   │   │
  ↓   ↓   ↓   ↓   ↓
 buf0 buf1 buf2 buf3 buf4
 [··] [··] [··] [··] [··]
```

这种结构意味着：
- 在首尾操作时，可能只需要新增一个缓冲区
- 但**中控数组本身可能需要重新分配**

### 3.2 场景一：在首部或尾部插入

```cpp
#include <iostream>
#include <deque>
using namespace std;

int main() {
    deque<int> dq = {1, 2, 3, 4, 5};

    auto it = dq.begin() + 2;  // 指向 3

    // 在末尾插入
    dq.push_back(6);

    // ⚠️ 标准规定：在首部或尾部插入时
    //    所有 **迭代器** 失效，但 **引用和指针** 不失效！
    // 这意味着：
    //    int& ref = dq[2];  // 如果之前保存了引用，仍然有效
    //    auto it = ...;     // 但迭代器失效了

    return 0;
}
```

**为什么迭代器会失效但引用不会？**

> 这与 `deque` 迭代器的内部实现有关。`deque` 的迭代器不仅仅是一个简单的指针，它内部包含了**对中控数组的引用、当前缓冲区的首尾指针**等"导航信息"。当在首尾插入导致新增缓冲区时，中控数组可能重新分配，这些导航信息就失效了。
>
> 但元素本身的内存（在各个缓冲区中）没有被移动，所以**指向元素的引用和指针仍然有效**。

### 3.3 场景二：在中间位置插入或删除

```cpp
#include <deque>
using namespace std;

int main() {
    deque<int> dq = {1, 2, 3, 4, 5};

    auto it1 = dq.begin();      // 指向 1
    auto it2 = dq.begin() + 2;  // 指向 3
    int& ref = dq[4];           // 引用元素 5

    // 在中间插入
    dq.insert(dq.begin() + 2, 99);
    // 结果：{1, 2, 99, 3, 4, 5}

    // ❌ 所有迭代器全部失效（it1, it2 都失效）
    // ❌ 所有引用和指针也全部失效（ref 也失效！）
    // 这比首尾插入更严重！

    return 0;
}
```

**为什么中间操作更严重？**

在中间插入/删除时，`deque` 需要移动元素。它会选择移动**较少的一侧**（前半部分或后半部分）来提高效率。但无论移动哪一侧，都会导致元素在缓冲区中的位置发生变化，所以**引用和指针也一并失效**。

### 3.4 场景三：删除首部或尾部元素

```cpp
#include <deque>
using namespace std;

int main() {
    deque<int> dq = {1, 2, 3, 4, 5};

    auto it_mid = dq.begin() + 2;  // 指向 3

    dq.pop_front();  // 删除首部的 1
    // ✅ 除被删除元素外，其余迭代器通常仍可用
    // ✅ 指向未被删除元素的引用和指针仍然有效

    dq.pop_back();   // 删除尾部的 5
    // ⚠️ 同理；另外 pop_back 后 end() 迭代器会失效
}
```

> 注意：`deque` 在首尾删除时，通常是**仅被删元素相关迭代器失效**；其中 `pop_back` 还会使 `end()` 失效。为了避免实现细节差异带来的误判，工程上仍建议修改后重新获取关键迭代器。

### 3.5 `deque` 迭代器失效规则总结

| 操作                 | 迭代器             | 引用和指针         |
| -------------------- | ------------------ | ------------------ |
| 在**首部或尾部**插入 | ❌ **全部失效**     | ✅ 不失效           |
| 在**中间**插入       | ❌ **全部失效**     | ❌ **全部失效**     |
| 删除**首部元素**     | 除被删元素外不失效 | 除被删元素外不失效 |
| 删除**尾部元素**     | 除被删元素外不失效 | 除被删元素外不失效 |
| 删除**中间元素**     | ❌ **全部失效**     | ❌ **全部失效**     |

> 💡 **实用建议**：`deque` 的迭代器失效规则非常复杂，实际编程中**保守起见，对 `deque` 做任何修改操作后，都重新获取迭代器**。

---

## 四、`list` / `forward_list` 迭代器失效

### 4.1 底层内存模型回顾

`list` 是**双向链表**，`forward_list` 是**单向链表**。每个元素都是一个独立的**堆上节点**，节点之间通过指针相连。

```
list 的内存布局：

  ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐
  │ prev │←────│ prev │←────│ prev │←────│ prev │
  │ data │     │ data │     │ data │     │ data │
  │ next │────→│ next │────→│ next │────→│ next │
  └──────┘     └──────┘     └──────┘     └──────┘
  节点独立分配，互不相邻
```

关键特性：**插入或删除一个节点，只是修改相邻节点的指针，其他节点的内存位置完全不受影响。**

### 4.2 插入操作：完全不影响已有迭代器

```cpp
#include <iostream>
#include <list>
#include <iterator>
using namespace std;

int main() {
    list<int> lst = {1, 2, 3, 4, 5};

    auto it1 = lst.begin();           // 指向 1
    auto it2 = next(lst.begin(), 2);  // 指向 3
    auto it3 = prev(lst.end());       // 指向 5

    // 在 it2（指向 3）之前插入 99
    lst.insert(it2, 99);
    // 链表变为：1 -> 2 -> 99 -> 3 -> 4 -> 5

    // ✅ it1 仍然有效，指向 1
    cout << *it1 << endl;  // 1

    // ✅ it2 仍然有效，仍然指向 3（节点没有移动！）
    cout << *it2 << endl;  // 3

    // ✅ it3 仍然有效，仍然指向 5
    cout << *it3 << endl;  // 5

    // 所有迭代器都完好无损！
    return 0;
}
```

**为什么插入不会导致失效？**

```
插入前：
[1] ←→ [2] ←→ [3] ←→ [4] ←→ [5]
 ↑             ↑                ↑
it1           it2              it3

插入 99 在 it2 之前：
[1] ←→ [2] ←→ [99] ←→ [3] ←→ [4] ←→ [5]
 ↑                      ↑                ↑
it1                    it2              it3

新节点 [99] 被单独分配在堆上，只是修改了 [2] 和 [3] 的指针。
it2 指向的节点 [3] 的内存地址没有任何变化！
```

### 4.3 删除操作：只有被删除节点的迭代器失效

```cpp
#include <iostream>
#include <list>
#include <iterator>
using namespace std;

int main() {
    list<int> lst = {1, 2, 3, 4, 5};

    auto it1 = lst.begin();           // 指向 1
    auto it2 = next(lst.begin(), 2);  // 指向 3
    auto it3 = next(lst.begin(), 3);  // 指向 4
    auto it4 = prev(lst.end());       // 指向 5

    // 删除 it2 指向的节点（值为 3）
    lst.erase(it2);
    // 链表变为：1 -> 2 -> 4 -> 5

    // ✅ it1 有效，指向 1
    cout << *it1 << endl;  // 1

    // ❌ it2 已失效！节点已被释放（delete）
    // cout << *it2;  // 未定义行为！

    // ✅ it3 有效，指向 4（节点没受影响）
    cout << *it3 << endl;  // 4

    // ✅ it4 有效，指向 5
    cout << *it4 << endl;  // 5

    return 0;
}
```

**图解删除：**

```
删除前：
[1] ←→ [2] ←→ [3] ←→ [4] ←→ [5]
 ↑             ↑      ↑      ↑
it1           it2    it3    it4

删除 it2 指向的 [3]：
[1] ←→ [2] ←→ [4] ←→ [5]       [3] 被 delete
 ↑             ↑       ↑         ↑
it1           it3     it4       it2（悬垂！）

只有 it2 失效，其他迭代器完全不受影响。
```

### 4.4 `forward_list` 的规则

`forward_list`（单向链表）的规则与 `list` **完全一致**：

- 插入操作：**不会**导致任何迭代器失效
- 删除操作：**只有被删除节点的迭代器**失效

唯一的区别在 API 上（`forward_list` 使用 `insert_after` / `erase_after` 而不是 `insert` / `erase`）。

### 4.5 `list` / `forward_list` 迭代器失效规则总结

| 操作             | 失效范围                       |
| ---------------- | ------------------------------ |
| 任意位置**插入** | ✅ **无迭代器失效**             |
| **删除**某个元素 | ❌ **仅被删除元素的迭代器**失效 |
| `clear()`        | ❌ **所有**迭代器失效           |

> 🎯 **这就是链表的优势所在**：如果你的场景需要频繁插入删除，且需要持有大量迭代器，`list` 是最安全的选择。

---

## 五、`set` / `map` 迭代器失效

### 5.1 底层数据结构回顾

`set`、`multiset`、`map`、`multimap` 的底层都是**红黑树**（一种自平衡二叉搜索树）。每个元素存储在独立的**树节点**中。

```
红黑树示意图（set<int> = {1, 2, 3, 4, 5, 6, 7}）：

           4(B)
         /     \
       2(R)    6(R)
      / \      / \
    1(B) 3(B) 5(B) 7(B)

每个数字是一个独立的堆上节点
```

### 5.2 迭代器失效规则：与 `list` 完全相同

红黑树的节点和链表的节点一样，都是**独立分配在堆上**的。插入或删除一个节点，只是修改树的指针结构（加上旋转和重新着色），**其他节点的内存位置完全不变**。

```cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {10, 20, 30, 40, 50};

    auto it20 = s.find(20);  // 指向 20
    auto it40 = s.find(40);  // 指向 40

    // 插入新元素
    s.insert(25);
    s.insert(35);

    // ✅ it20 和 it40 仍然有效！
    cout << *it20 << endl;  // 20
    cout << *it40 << endl;  // 40

    // 删除 30
    s.erase(30);

    // ✅ it20 和 it40 仍然有效！
    cout << *it20 << endl;  // 20
    cout << *it40 << endl;  // 40

    // 但如果删除 it20 指向的元素...
    s.erase(it20);
    // ❌ it20 已失效！
    // ✅ it40 仍然有效

    return 0;
}
```

### 5.3 `map` 同理

```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

int main() {
    map<string, int> m = {
        {"apple", 1}, {"banana", 2}, {"cherry", 3}
    };

    auto it = m.find("banana");  // 指向 {"banana", 2}

    // 插入新元素
    m["date"] = 4;
    m["elderberry"] = 5;

    // ✅ it 仍然有效
    cout << it->first << ": " << it->second << endl;  // banana: 2

    // 删除其他元素
    m.erase("apple");
    m.erase("cherry");

    // ✅ it 仍然有效
    cout << it->first << ": " << it->second << endl;  // banana: 2

    return 0;
}
```

### 5.4 红黑树旋转不影响迭代器的原理

你可能会好奇：红黑树在插入/删除后会进行**旋转**操作来保持平衡，旋转不会影响迭代器吗？

```
旋转前：                    旋转后（右旋）：
      Y                         X
     / \                       / \
    X   C        →            A   Y
   / \                           / \
  A   B                         B   C
```

答案是：**不会**。旋转只是修改了节点之间的**父子指针关系**，节点本身的内存位置完全没有变化。迭代器指向的是节点的内存地址，地址没变，迭代器自然不会失效。

### 5.5 `set` / `map` 迭代器失效规则总结

| 操作             | 失效范围                       |
| ---------------- | ------------------------------ |
| 任意位置**插入** | ✅ **无迭代器失效**             |
| **删除**某个元素 | ❌ **仅被删除元素的迭代器**失效 |
| `clear()`        | ❌ **所有**迭代器失效           |

> 这与 `list` 的规则完全一致，原因也一样：基于节点的容器，每个元素独立分配。

---

## 六、`unordered_set` / `unordered_map` 迭代器失效

### 6.1 底层数据结构回顾

`unordered_set` 和 `unordered_map` 底层是**哈希表**。典型实现是**拉链法（separate chaining）**：一个桶数组，每个桶挂一个链表。

```
桶数组 (bucket array)：
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 0  │ 1  │ 2  │ 3  │ 4  │ 5  │ 6  │ 7  │
└──┬─┴──┬─┴────┴──┬─┴────┴──┬─┴────┴────┘
   │    │         │         │
   ↓    ↓         ↓         ↓
  [A]  [B]       [D]       [F]
   │              │
   ↓              ↓
  [C]            [E]
```

### 6.2 关键概念：负载因子和 Rehash

- **负载因子（load factor）** = `size() / bucket_count()`
- **最大负载因子**：默认为 1.0（可通过 `max_load_factor()` 修改）
- 当插入元素导致 `load_factor() > max_load_factor()` 时，触发 **rehash**

**Rehash 的过程：**

1. 分配一个**更大的桶数组**
2. 将所有元素按照新的桶数量**重新计算哈希值并重新分配到新桶中**
3. 释放旧的桶数组

这意味着元素所在的链表节点可能被移动到不同的桶中（在某些实现中，节点本身被重新分配）。

### 6.3 场景一：插入可能触发 Rehash

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

int main() {
    unordered_set<int> us = {1, 2, 3};

    cout << "bucket_count: " << us.bucket_count() << endl;
    cout << "load_factor: " << us.load_factor() << endl;
    cout << "max_load_factor: " << us.max_load_factor() << endl;

    auto it = us.find(2);  // 指向元素 2

    // 大量插入，很可能触发 rehash
    for (int i = 10; i < 100; ++i) {
        us.insert(i);
    }

    // ❌ 如果发生了 rehash，it 已经失效！
    // cout << *it;  // 未定义行为

    return 0;
}
```

> ⚠️ **关键规则**：如果插入操作触发了 rehash，**所有**迭代器失效。如果**没有**触发 rehash，则迭代器**不失效**。

### 6.4 场景二：删除操作

删除操作不会触发 rehash（桶数组不会收缩），因此：

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

int main() {
    unordered_set<int> us = {1, 2, 3, 4, 5};

    auto it2 = us.find(2);
    auto it4 = us.find(4);

    // 删除元素 3
    us.erase(3);

    // ✅ it2 仍然有效
    cout << *it2 << endl;  // 2

    // ✅ it4 仍然有效
    cout << *it4 << endl;  // 4

    // 删除 it2 指向的元素
    us.erase(it2);
    // ❌ it2 已失效
    // ✅ it4 仍然有效

    return 0;
}
```

### 6.5 如何判断插入是否会触发 Rehash？

```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

int main() {
    unordered_set<int> us;

    // 方法 1：提前预留空间，避免 rehash
    us.reserve(1000);  // 预先分配足够的桶

    auto it = us.insert(42).first;

    // 后续插入不超过 1000 个元素，就不会 rehash
    for (int i = 0; i < 500; ++i) {
        us.insert(i);
    }
    // ✅ it 仍然有效（因为没有 rehash）
    cout << *it << endl;  // 42

    // 方法 2：在插入前检查
    size_t old_bucket_count = us.bucket_count();
    us.insert(9999);
    if (us.bucket_count() != old_bucket_count) {
        cout << "发生了 rehash！之前的迭代器全部失效！" << endl;
    }

    return 0;
}
```

### 6.6 `unordered_set` / `unordered_map` 迭代器失效规则总结

| 操作                      | 失效范围                                   |
| ------------------------- | ------------------------------------------ |
| **插入**（未触发 rehash） | ✅ **无迭代器失效**                         |
| **插入**（触发 rehash）   | ❌ **所有**迭代器失效                       |
| **删除**某个元素          | ❌ **仅被删除元素的迭代器**失效             |
| `clear()`                 | ❌ **所有**迭代器失效                       |
| `rehash()` / `reserve()`  | 可能导致 rehash → ❌ **所有**迭代器可能失效 |

---

## 七、安全删除元素的正确写法：使用 `erase()` 的返回值

这是本章**最核心的实践技能**。在遍历容器的同时删除满足条件的元素，是迭代器失效的**第一大陷阱**。

### 7.1 错误写法（经典 Bug）

```cpp
// ❌ 错误！经典的迭代器失效 Bug！
vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};

for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it % 2 == 0) {
        v.erase(it);  // erase 后 it 已经失效！
        // 然后循环还要 ++it，对失效迭代器++是未定义行为！
    }
}
```

**问题分析：**
1. `v.erase(it)` 删除了 `it` 指向的元素
2. 对于 `vector`，`it` 及之后的迭代器全部失效
3. 但循环的 `++it` 还要对这个已经失效的迭代器做递增——**未定义行为！**
4. 即使"碰巧不崩溃"，也会**跳过元素**——因为删除后后面的元素前移了一位，`++it` 会跳过紧跟在删除位置后面的那个元素

### 7.2 `erase()` 的返回值

C++ 标准为所有序列容器和关联容器的 `erase()` 方法定义了返回值：

> **`erase()` 返回一个迭代器，指向被删除元素之后的那个元素。**

这个返回值就是**安全删除的关键**。

```cpp
auto it = container.erase(some_it);
// it 现在指向被删除元素的"下一个"元素
// 对于 vector：后面的元素前移了，it 自动指向新的正确位置
// 对于 list：it 指向被删节点的后继节点
// 对于 set/map：it 指向中序遍历的下一个节点
```

### 7.3 正确写法一：通用方法（适用于所有容器）

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};

    // ✅ 正确写法：使用 erase 的返回值
    for (auto it = v.begin(); it != v.end(); /* 注意：这里不写 ++it */) {
        if (*it % 2 == 0) {
            it = v.erase(it);  // erase 返回下一个有效迭代器
            // 不需要 ++it，因为 erase 返回的已经是"下一个"了
        } else {
            ++it;  // 只在不删除时才手动前进
        }
    }

    // 输出：1 3 5 7
    for (int x : v) cout << x << " ";
    cout << endl;

    return 0;
}
```

**执行过程详细跟踪：**

```
初始：[1, 2, 3, 4, 5, 6, 7, 8]
      it↑

*it=1, 不是偶数 → ++it
[1, 2, 3, 4, 5, 6, 7, 8]
   it↑

*it=2, 是偶数 → it = erase(it)
[1, 3, 4, 5, 6, 7, 8]    ← 2被删除，后面元素前移
   it↑                    ← erase返回的it指向原来2的位置，现在是3

*it=3, 不是偶数 → ++it
[1, 3, 4, 5, 6, 7, 8]
      it↑

*it=4, 是偶数 → it = erase(it)
[1, 3, 5, 6, 7, 8]
      it↑                ← 指向5

*it=5, 不是偶数 → ++it
*it=6, 是偶数 → it = erase(it)
[1, 3, 5, 7, 8]
         it↑             ← 指向7（原来8前面的7）
... 以此类推

最终：[1, 3, 5, 7]
```

### 7.4 各容器的安全删除示例

#### `list` 的安全删除

```cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> lst = {1, 2, 3, 4, 5, 6, 7, 8};

    // ✅ 写法完全一致
    for (auto it = lst.begin(); it != lst.end(); ) {
        if (*it % 2 == 0) {
            it = lst.erase(it);
        } else {
            ++it;
        }
    }

    for (int x : lst) cout << x << " ";  // 1 3 5 7
    cout << endl;

    return 0;
}
```

#### `set` 的安全删除

```cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    set<int> s = {1, 2, 3, 4, 5, 6, 7, 8};

    // ✅ 同样的模式
    for (auto it = s.begin(); it != s.end(); ) {
        if (*it % 2 == 0) {
            it = s.erase(it);  // C++11 起，set::erase 返回迭代器
        } else {
            ++it;
        }
    }

    for (int x : s) cout << x << " ";  // 1 3 5 7
    cout << endl;

    return 0;
}
```

> 📝 **历史注意**：在 C++11 之前，关联容器（`set`/`map`）的 `erase()` 返回 `void`！那时候需要用另一种写法（见 7.5）。C++11 之后统一改为返回下一个迭代器，所以现在可以用统一写法了。

#### `map` 的安全删除

```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

int main() {
    map<string, int> scores = {
        {"Alice", 85}, {"Bob", 42}, {"Charlie", 91},
        {"David", 38}, {"Eve", 76}
    };

    // 删除分数低于 50 的条目
    for (auto it = scores.begin(); it != scores.end(); ) {
        if (it->second < 50) {
            it = scores.erase(it);
        } else {
            ++it;
        }
    }

    for (const auto& [name, score] : scores) {
        cout << name << ": " << score << endl;
    }
    // 输出：Alice: 85, Charlie: 91, Eve: 76

    return 0;
}
```

#### `unordered_map` 的安全删除

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main() {
    unordered_map<string, int> inventory = {
        {"apple", 0}, {"banana", 5}, {"cherry", 0},
        {"date", 12}, {"elderberry", 0}
    };

    // 删除库存为 0 的商品
    for (auto it = inventory.begin(); it != inventory.end(); ) {
        if (it->second == 0) {
            it = inventory.erase(it);
        } else {
            ++it;
        }
    }

    for (const auto& [item, count] : inventory) {
        cout << item << ": " << count << endl;
    }

    return 0;
}
```

### 7.5 C++11 之前关联容器的替代写法（`erase` 返回 `void`）

虽然现代 C++ 已不需要这种写法，但你可能会在老代码中见到，了解一下：

```cpp
// C++11 之前的关联容器安全删除（历史写法）
set<int> s = {1, 2, 3, 4, 5, 6};

for (auto it = s.begin(); it != s.end(); ) {
    if (*it % 2 == 0) {
        s.erase(it++);  // 后置++：先保存当前位置，再递增，然后删除旧位置
        // 注意：这对 set/map 有效，因为删除不影响其他节点
        // 但对 vector 无效！因为 vector erase 后 it 后面的迭代器全部失效
    } else {
        ++it;
    }
}
```

**`s.erase(it++)` 的执行顺序**：
1. `it++`：后置递增，返回 `it` 的**旧值**（一个临时拷贝），同时 `it` 自身已经指向下一个元素
2. `s.erase(旧值)`：删除旧位置指向的节点
3. 因为 `set` 删除只影响被删节点，`it`（已经指向下一个节点）仍然有效

> ⚠️ 再次强调：`erase(it++)` 这个技巧**只对基于节点的容器**（`list`、`set`、`map` 等）安全，对 `vector` 和 `deque` **不安全**！因为它们的 `erase` 会使删除点之后的迭代器全部失效。

### 7.6 C++20 的终极方案：`std::erase_if`

C++20 引入了 `std::erase_if`——一个非成员函数，为**所有标准容器**提供了统一的"按条件删除"接口：

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <set>
#include <map>
#include <unordered_set>
#include <string>
using namespace std;

int main() {
    // ✅ vector
    vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};
    erase_if(v, [](int x) { return x % 2 == 0; });
    // v = {1, 3, 5, 7}

    // ✅ list
    list<int> lst = {1, 2, 3, 4, 5, 6, 7, 8};
    erase_if(lst, [](int x) { return x % 2 == 0; });

    // ✅ set
    set<int> s = {1, 2, 3, 4, 5, 6, 7, 8};
    erase_if(s, [](int x) { return x % 2 == 0; });

    // ✅ map（注意：谓词参数是 pair）
    map<string, int> m = {{"a", 1}, {"b", 2}, {"c", 3}};
    erase_if(m, [](const pair<const string, int>& p) {
        return p.second % 2 == 0;
    });

    // ✅ unordered_set
    unordered_set<int> us = {1, 2, 3, 4, 5};
    erase_if(us, [](int x) { return x < 3; });

    return 0;
}
```

C++20 还为 `vector` 和其他序列容器提供了 `std::erase`（按值删除）：

```cpp
vector<int> v = {1, 2, 3, 2, 4, 2, 5};
erase(v, 2);  // 删除所有值为 2 的元素
// v = {1, 3, 4, 5}
```

> 🚀 **推荐**：如果你使用 C++20 或更高版本，**优先使用 `std::erase_if`**，它在内部已经处理好了迭代器失效问题，代码更简洁、更安全。

---

## 八、全容器迭代器失效规则大汇总

这是一张你应该**牢记于心**的表格：

### 8.1 插入操作

| 容器                    | 插入位置         | 迭代器失效     | 引用/指针失效  |
| ----------------------- | ---------------- | -------------- | -------------- |
| `vector`                | 任意（触发扩容） | ❌ **全部**     | ❌ **全部**     |
| `vector`                | 任意（不扩容）   | ❌ 插入点及之后 | ❌ 插入点及之后 |
| `deque`                 | 首/尾            | ❌ **全部**     | ✅ 不失效       |
| `deque`                 | 中间             | ❌ **全部**     | ❌ **全部**     |
| `list` / `forward_list` | 任意             | ✅ **不失效**   | ✅ **不失效**   |
| `set` / `map`           | —                | ✅ **不失效**   | ✅ **不失效**   |
| `unordered_*`           | 无 rehash        | ✅ **不失效**   | ✅ **不失效**   |
| `unordered_*`           | 有 rehash        | ❌ **全部**     | ❌ **全部**     |

### 8.2 删除操作

| 容器                    | 删除位置 | 迭代器失效     | 引用/指针失效  |
| ----------------------- | -------- | -------------- | -------------- |
| `vector`                | 任意     | ❌ 删除点及之后 | ❌ 删除点及之后 |
| `deque`                 | 首/尾    | 仅被删元素     | 仅被删元素     |
| `deque`                 | 中间     | ❌ **全部**     | ❌ **全部**     |
| `list` / `forward_list` | 任意     | 仅被删元素     | 仅被删元素     |
| `set` / `map`           | 任意     | 仅被删元素     | 仅被删元素     |
| `unordered_*`           | 任意     | 仅被删元素     | 仅被删元素     |

### 8.3 记忆口诀

> **"连续内存怕扩容，分段结构怕中间，节点容器最安全，哈希要防 rehash 关。"**

- **`vector`**（连续内存）：扩容 = 全军覆没；不扩容 = 插入/删除点之后完蛋
- **`deque`**（分段连续）：首尾还好，中间全完
- **`list` / `set` / `map`**（独立节点）：只有被删的那个节点失效
- **`unordered_*`**（哈希表）：rehash = 全军覆没；不 rehash = 同节点容器

---

## 九、综合实战练习

### 9.1 练习一：从 `vector` 中删除所有负数

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {3, -1, 4, -1, 5, -9, 2, -6, 5};

    // 正确写法
    for (auto it = v.begin(); it != v.end(); ) {
        if (*it < 0) {
            it = v.erase(it);
        } else {
            ++it;
        }
    }

    for (int x : v) cout << x << " ";
    // 输出：3 4 5 2 5
    cout << endl;

    return 0;
}
```

### 9.2 练习二：从 `map` 中删除值为空字符串的键值对

```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

int main() {
    map<string, string> config = {
        {"host", "localhost"},
        {"port", "8080"},
        {"username", ""},
        {"password", ""},
        {"database", "mydb"}
    };

    for (auto it = config.begin(); it != config.end(); ) {
        if (it->second.empty()) {
            it = config.erase(it);
        } else {
            ++it;
        }
    }

    for (const auto& [key, value] : config) {
        cout << key << " = " << value << endl;
    }
    // 输出：
    // database = mydb
    // host = localhost
    // port = 8080

    return 0;
}
```

### 9.3 练习三：在遍历 `vector` 时满足条件就插入（错误 vs 正确）

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // 需求：在每个偶数后面插入一个 0

    // ❌ 错误写法
    // vector<int> v = {1, 2, 3, 4, 5};
    // for (auto it = v.begin(); it != v.end(); ++it) {
    //     if (*it % 2 == 0) {
    //         v.insert(it + 1, 0);  // 插入后 it 失效！
    //     }
    // }

    // ✅ 正确写法：使用 insert 的返回值
    vector<int> v = {1, 2, 3, 4, 5};
    for (auto it = v.begin(); it != v.end(); ++it) {
        if (*it % 2 == 0) {
            it = v.insert(it + 1, 0);  // insert 返回指向新插入元素的迭代器
            // it 现在指向刚插入的 0，循环末尾 ++it 会跳过它，继续下一个
        }
    }

    for (int x : v) cout << x << " ";
    // 输出：1 2 0 3 4 0 5
    cout << endl;

    return 0;
}
```

### 9.4 练习四：在 `unordered_map` 中统计并删除出现次数小于 2 的元素

```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
#include <string>
using namespace std;

int main() {
    vector<string> words = {
        "hello", "world", "hello", "cpp", "world", "hello", "stl"
    };

    // 统计词频
    unordered_map<string, int> freq;
    for (const auto& w : words) {
        freq[w]++;
    }

    // 删除出现次数小于 2 的词
    for (auto it = freq.begin(); it != freq.end(); ) {
        if (it->second < 2) {
            it = freq.erase(it);
        } else {
            ++it;
        }
    }

    for (const auto& [word, count] : freq) {
        cout << word << ": " << count << endl;
    }
    // 输出（顺序不定）：
    // hello: 3
    // world: 2

    return 0;
}
```

---

## 十、常见面试题与陷阱

### 10.1 面试题："请说出 vector 迭代器失效的所有情况"

**标准答案：**
1. `push_back` / `insert` 等插入操作，若触发扩容 → **所有**迭代器失效
2. `push_back` 不触发扩容 → 只有 `end()` 失效
3. `insert(pos, ...)` 不触发扩容 → `pos` 及之后的迭代器失效
4. `erase(pos)` → `pos` 及之后的迭代器失效
5. `pop_back()` → 末尾元素的迭代器和 `end()` 失效

### 10.2 面试题："如何安全地在遍历容器时删除元素？"

**标准答案：**
- 使用 `erase()` 的返回值获取下一个有效迭代器
- C++20 可以使用 `std::erase_if` 一行解决
- 对于 `vector`，也可以先用 `std::remove_if` + `erase`（erase-remove idiom）

### 10.3 补充：Erase-Remove 惯用法（`vector` 专用高效方案）

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};

    // erase-remove idiom
    v.erase(
        remove_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; }),
        v.end()
    );

    for (int x : v) cout << x << " ";  // 1 3 5 7
    cout << endl;

    return 0;
}
```

**为什么 erase-remove 比逐个 erase 高效？**

逐个 `erase` 的时间复杂度是 O(n²)：每次删除都要移动后续元素。

而 `remove_if` 的时间复杂度是 O(n)：它将不需要删除的元素"前移"，最后用一次 `erase` 批量删除尾部。

```
原始：   [1, 2, 3, 4, 5, 6, 7, 8]

remove_if 之后（删除偶数）：
         [1, 3, 5, 7, ?, ?, ?, ?]
                      ↑
                  remove_if 返回的迭代器
                  （从这里到 end() 都是"垃圾"）

erase 从返回的迭代器到 end()：
         [1, 3, 5, 7]
```

---

## 十一、总结与最佳实践

### 11.1 防止迭代器失效的黄金法则

1. **修改容器后，不要再使用旧迭代器**
2. **使用 `erase()` 的返回值**来获取下一个有效迭代器
3. **预分配容量**（`vector::reserve`、`unordered_*::reserve`）可以减少扩容/rehash 导致的失效
4. **如果需要在遍历时修改容器，使用正确的循环模式**（`for(auto it = ...; it != ...; )` 手动控制递增）
5. **如果可以使用 C++20，优先使用 `std::erase_if`**

### 11.2 选择容器的参考

| 需求                             | 推荐容器      | 原因                  |
| -------------------------------- | ------------- | --------------------- |
| 频繁随机访问                     | `vector`      | O(1) 随机访问         |
| 频繁在中间插入/删除 + 持有迭代器 | `list`        | 迭代器几乎不失效      |
| 需要有序 + 频繁查找              | `set` / `map` | O(log n) + 迭代器稳定 |
| 需要最快查找 + 不关心迭代器      | `unordered_*` | O(1) 查找             |
| 频繁在两端操作                   | `deque`       | O(1) 首尾操作         |

### 11.3 调试建议

如果你怀疑代码中有迭代器失效的 Bug，可以尝试以下方法：

1. **使用 Debug 模式编译**：MSVC 的 Debug 模式会在迭代器失效后自动检测并报错（`_ITERATOR_DEBUG_LEVEL=2`）
2. **使用 ASan（AddressSanitizer）**：`-fsanitize=address` 可以检测悬垂指针访问
3. **代码审查**：检查所有在循环中调用 `insert` / `erase` / `push_back` 的地方，确认迭代器是否被正确更新
4. **最小化迭代器持有时间**：用完即弃，不要长期保存迭代器（尤其是 `vector` 和 `unordered_*` 的）

---

### 11.4 一句话速查表（贴在屏幕旁边）

| 容器          | 插入失效                   | 删除失效               | 安全删除写法         |
| ------------- | -------------------------- | ---------------------- | -------------------- |
| `vector`      | 扩容→全部；不扩容→插入点后 | 删除点及之后           | `it = v.erase(it)`   |
| `deque`       | 首尾→迭代器全部；中间→全部 | 首尾→仅被删；中间→全部 | `it = dq.erase(it)`  |
| `list`        | ✅ 无失效                   | 仅被删节点             | `it = lst.erase(it)` |
| `set`/`map`   | ✅ 无失效                   | 仅被删节点             | `it = s.erase(it)`   |
| `unordered_*` | rehash→全部；否则无        | 仅被删节点             | `it = us.erase(it)`  |

> 🎯 **核心口诀**：**"连续怕扩容，分段怕中间，节点最稳定，哈希防 rehash。删除用返回值，万事都平安。"**

---

以上就是迭代器失效专题的全部内容。这个主题的核心就两句话：

1. **理解不同容器的底层结构**，你就能推导出失效规则，不用死记硬背
2. **遍历时删除，永远用 `it = container.erase(it)` 的模式**——这是最安全的万能写法