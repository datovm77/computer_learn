# 第十一阶段：`forward_list` 容器（补充）

> **前置知识**：你已经学完了 `list`（双向链表）。本阶段我们学习 `forward_list`（单向链表），并与 `list` 做全面对比。

---

## 一、`forward_list` — 基本概念

### 1.1 什么是 `forward_list`？

`forward_list` 是 C++11 引入的一个**单向链表**容器，定义在头文件 `<forward_list>` 中。

回忆一下数据结构：

```
【双向链表 list】
  nullptr ⇄ [A] ⇄ [B] ⇄ [C] ⇄ nullptr
               每个节点有 prev 和 next 两个指针

【单向链表 forward_list】
  [A] → [B] → [C] → nullptr
               每个节点只有 next 一个指针
```

因为每个节点只保存"指向下一个节点"的指针，所以：
- ✅ 内存开销更小（每个节点少存一个指针）
- ✅ 插入/删除操作更轻量
- ❌ 只能**单向**遍历（从前往后），不能反向遍历
- ❌ 没有 `size()` 成员函数（为了保持零开销原则）
- ❌ 没有 `push_back()`、`pop_back()`（因为找不到尾节点，除非从头遍历到尾）

### 1.2 `forward_list` 与 `list` 的核心区别对比

| 特性             | `list`（双向链表）                 | `forward_list`（单向链表）                |
| ---------------- | ---------------------------------- | ----------------------------------------- |
| 头文件           | `<list>`                           | `<forward_list>`                          |
| 引入标准         | C++98                              | **C++11**                                 |
| 节点指针数       | 2个（`prev` + `next`）             | 1个（仅 `next`）                          |
| 内存开销         | 较大                               | **较小**                                  |
| 遍历方向         | 双向（前向 + 反向）                | **仅前向**                                |
| 迭代器类型       | 双向迭代器 `BidirectionalIterator` | **前向迭代器 `ForwardIterator`**          |
| `size()`         | ✅ 有（O(1)）                       | ❌ **没有**（需用 `std::distance` 手动算） |
| `push_back()`    | ✅ 有                               | ❌ **没有**                                |
| `pop_back()`     | ✅ 有                               | ❌ **没有**                                |
| `push_front()`   | ✅ 有                               | ✅ 有                                      |
| `pop_front()`    | ✅ 有                               | ✅ 有                                      |
| `insert()`       | 在迭代器**之前**插入               | ❌ 没有 `insert()`，只有 `insert_after()`  |
| `erase()`        | 删除迭代器**指向**的元素           | ❌ 没有 `erase()`，只有 `erase_after()`    |
| `before_begin()` | ❌ 没有                             | ✅ **有**（返回头节点之前的位置）          |
| `back()`         | ✅ 有                               | ❌ **没有**                                |
| `reverse()`      | ✅ 有                               | ✅ 有                                      |
| `sort()`         | ✅ 有                               | ✅ 有                                      |
| 适用场景         | 需要双向遍历、频繁在任意位置增删   | **极致节省内存**、只需前向遍历            |

### 1.3 为什么要有 `forward_list`？

你可能会想：既然 `list` 功能更全，为什么还要 `forward_list`？

> **设计哲学：零开销原则（Zero-overhead Principle）**
>
> C++ 委员会设计 `forward_list` 的目标是：**提供一个与手写 C 语言单向链表性能完全一致的标准容器，不多花一分一毫的额外开销。**

具体体现：
1. **没有 `size()` 成员函数** — 如果维护一个 `size` 计数器，每次插入/删除都要更新，这对于不需要知道大小的场景是额外开销。
2. **没有 `push_back()`** — 单向链表要在尾部插入，必须从头遍历到尾，O(n) 复杂度。标准库不提供这个函数，防止用户误以为它是 O(1)。
3. **只有 `insert_after()` 和 `erase_after()`** — 单向链表无法高效地操作"前一个节点"，所以所有操作都是基于"在某节点**之后**"进行的。

### 1.4 迭代器类型详解

```
迭代器能力层级（从弱到强）：

  InputIterator（输入迭代器）
       ↓
  ForwardIterator（前向迭代器）     ← forward_list 在这里
       ↓
  BidirectionalIterator（双向迭代器） ← list 在这里
       ↓
  RandomAccessIterator（随机访问迭代器） ← vector / deque 在这里
```

`forward_list` 的迭代器是**前向迭代器（ForwardIterator）**：
- ✅ 可以 `++it`（向前移动）
- ❌ 不可以 `--it`（不能后退！编译报错）
- ❌ 不可以 `it + n`（不能随机跳跃）
- ✅ 可以 `*it`（解引用，读写元素）
- ✅ 可以 `it1 == it2` / `it1 != it2`（比较）

### 1.5 基本使用：创建与初始化

```cpp name=forward_list_basic.cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {

    // ===== 1. 默认构造：创建空的 forward_list =====
    forward_list<int> fl1;
    // fl1: （空）

    // ===== 2. 初始化列表构造 =====
    forward_list<int> fl2 = {10, 20, 30, 40, 50};
    // fl2: 10 → 20 → 30 → 40 → 50

    // ===== 3. 指定个数和值 =====
    forward_list<int> fl3(5, 100);
    // fl3: 100 → 100 → 100 → 100 → 100

    // ===== 4. 拷贝构造 =====
    forward_list<int> fl4(fl2);
    // fl4: 10 → 20 → 30 → 40 → 50（fl2 的副本）

    // ===== 5. 迭代器范围构造 =====
    forward_list<int> fl5(fl2.begin(), fl2.end());
    // fl5: 10 → 20 → 30 → 40 → 50

    // ===== 6. 移动构造（C++11）=====
    forward_list<int> fl6(move(fl3));
    // fl6: 100 → 100 → 100 → 100 → 100
    // fl3: （被移动后为空）

    // ===== 遍历输出 =====
    cout << "fl2: ";
    for (int val : fl2) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: fl2: 10 20 30 40 50

    // ===== 获取元素个数（没有 size()！）=====
    // fl2.size();  // ❌ 编译错误！forward_list 没有 size()

    // 正确做法：用 std::distance
    #include <iterator>  // 实际应放在文件顶部
    auto count = distance(fl2.begin(), fl2.end());
    cout << "fl2 的元素个数: " << count << endl;
    // 输出: fl2 的元素个数: 5

    // ===== empty() 判空 =====
    cout << "fl1 是否为空: " << fl1.empty() << endl;  // 1 (true)
    cout << "fl2 是否为空: " << fl2.empty() << endl;  // 0 (false)

    // ===== front() 访问第一个元素 =====
    cout << "fl2 的第一个元素: " << fl2.front() << endl;
    // 输出: 10
    // 注意：没有 back()！因为单向链表无法高效访问最后一个元素

    return 0;
}
```

**编译运行**（需要 C++11 或更高）：
```bash name=compile.sh
g++ -std=c++11 forward_list_basic.cpp -o forward_list_basic
./forward_list_basic
```

---

## 二、`forward_list` — 常用接口详解

### 2.1 理解 `forward_list` 操作的核心思想

在学习具体接口之前，必须先理解一个关键概念：

> **单向链表的所有修改操作，都是基于"在某节点之后"进行的。**

为什么？画图就明白了：

```
要删除节点 B：
  [A] → [B] → [C]
  
  你需要让 A 的 next 指向 C：
  [A] ───────→ [C]      （B 被跳过，即被删除）
  
  所以你需要知道 B 的"前一个节点" A。
  但单向链表中，给定 B，你无法高效找到 A（没有 prev 指针）！
  
  解决方案：API 设计为"给定 A，删除 A 之后的节点"
  即 erase_after(指向A的迭代器) → 删除 B
```

同理，插入也是一样的道理。这就是为什么 `forward_list` 只有 `insert_after()` 和 `erase_after()`，而没有 `insert()` 和 `erase()`。

### 2.2 `before_begin()` — 头节点之前的"哨兵位置"

这是 `forward_list` 独有的、非常重要的接口。

```
forward_list: [10] → [20] → [30] → nullptr

迭代器位置：

 before_begin()    begin()                     end()
        ↓             ↓                          ↓
    [ 哨兵位 ]   →   [ 10 ]   →   [ 20 ]   →   [ 30 ]   →   nullptr
```

- `before_begin()` 返回一个指向**第一个元素之前**的迭代器
- 这个位置是"虚拟的"，不能对它解引用（`*it` 是未定义行为）
- **它的唯一用途**：配合 `insert_after()` 和 `erase_after()` 来操作**第一个元素**

```cpp name=before_begin_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {

    forward_list<int> fl = {10, 20, 30};
    // fl: [10] → [20] → [30]

    // before_begin() 返回的迭代器指向"第一个元素之前"
    auto it_before = fl.before_begin();

    // 不能解引用 before_begin()！
    // cout << *it_before;  // ❌ 未定义行为！

    // 但可以用 insert_after 在它之后插入（即在最前面插入）
    fl.insert_after(it_before, 5);
    // fl: [5] → [10] → [20] → [30]

    // 也可以用 erase_after 删除它之后的元素（即删除第一个元素）
    fl.erase_after(fl.before_begin());
    // fl: [10] → [20] → [30]  （5 被删除了）

    for (int val : fl) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: 10 20 30

    return 0;
}
```

> 💡 **`cbefore_begin()`** 是 `before_begin()` 的 `const` 版本，返回 `const_iterator`，不允许通过迭代器修改元素。

### 2.3 `push_front()` — 在头部插入元素

```cpp name=push_front_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {

    forward_list<int> fl;

    // push_front：在链表头部插入
    fl.push_front(30);   // fl: [30]
    fl.push_front(20);   // fl: [20] → [30]
    fl.push_front(10);   // fl: [10] → [20] → [30]

    // 注意：插入顺序和最终顺序是"反的"！
    // 最后插入的在最前面

    for (int val : fl) {
        cout << val << " ";
    }
    cout << endl;
    // 输出: 10 20 30

    // emplace_front：原地构造（效率更高，避免拷贝）
    fl.emplace_front(5);
    // fl: [5] → [10] → [20] → [30]

    cout << "front: " << fl.front() << endl;
    // 输出: front: 5

    return 0;
}
```

### 2.4 `pop_front()` — 删除头部元素

```cpp name=pop_front_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {

    forward_list<int> fl = {10, 20, 30, 40};
    // fl: [10] → [20] → [30] → [40]

    cout << "删除前 front: " << fl.front() << endl;  // 10

    fl.pop_front();
    // fl: [20] → [30] → [40]

    cout << "删除后 front: " << fl.front() << endl;  // 20

    fl.pop_front();
    // fl: [30] → [40]

    fl.pop_front();
    // fl: [40]

    fl.pop_front();
    // fl: （空）

    cout << "是否为空: " << fl.empty() << endl;  // 1 (true)

    // fl.pop_front();  // ❌ 对空链表调用 pop_front() 是未定义行为！

    return 0;
}
```

### 2.5 `insert_after()` — 在指定位置**之后**插入

这是 `forward_list` 最核心的插入操作，有多个重载版本：

```cpp name=insert_after_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

// 辅助函数：打印 forward_list
void printList(const string& label, const forward_list<int>& fl) {
    cout << label << ": ";
    for (int val : fl) {
        cout << val << " ";
    }
    cout << endl;
}

int main() {

    // ===== 重载1：在指定位置之后插入单个元素 =====
    {
        forward_list<int> fl = {10, 20, 30};
        // fl: [10] → [20] → [30]

        auto it = fl.begin();  // 指向 10
        fl.insert_after(it, 15);
        // 在 10 之后插入 15
        // fl: [10] → [15] → [20] → [30]
        printList("重载1", fl);
    }

    // ===== 重载2：在指定位置之后插入 n 个相同值 =====
    {
        forward_list<int> fl = {10, 20, 30};
        auto it = fl.begin();  // 指向 10
        fl.insert_after(it, 3, 99);
        // 在 10 之后插入 3 个 99
        // fl: [10] → [99] → [99] → [99] → [20] → [30]
        printList("重载2", fl);
    }

    // ===== 重载3：在指定位置之后插入初始化列表 =====
    {
        forward_list<int> fl = {10, 20, 30};
        auto it = fl.begin();  // 指向 10
        fl.insert_after(it, {11, 12, 13});
        // 在 10 之后插入 11, 12, 13
        // fl: [10] → [11] → [12] → [13] → [20] → [30]
        printList("重载3", fl);
    }

    // ===== 重载4：在指定位置之后插入迭代器范围 =====
    {
        forward_list<int> fl = {10, 20, 30};
        forward_list<int> other = {100, 200, 300};
        auto it = fl.begin();  // 指向 10
        fl.insert_after(it, other.begin(), other.end());
        // fl: [10] → [100] → [200] → [300] → [20] → [30]
        printList("重载4", fl);
    }

    // ===== 在最前面插入（利用 before_begin()）=====
    {
        forward_list<int> fl = {10, 20, 30};
        fl.insert_after(fl.before_begin(), 5);
        // fl: [5] → [10] → [20] → [30]
        printList("头部插入", fl);
        // 效果等同于 push_front(5)
    }

    // ===== emplace_after()：原地构造版本 =====
    {
        forward_list<int> fl = {10, 20, 30};
        auto it = fl.begin();  // 指向 10
        fl.emplace_after(it, 15);
        // fl: [10] → [15] → [20] → [30]
        printList("emplace_after", fl);
    }

    return 0;
}
```

**输出**：
```
重载1: 10 15 20 30
重载2: 10 99 99 99 20 30
重载3: 10 11 12 13 20 30
重载4: 10 100 200 300 20 30
头部插入: 5 10 20 30
emplace_after: 10 15 20 30
```

### 2.6 `erase_after()` — 删除指定位置**之后**的元素

```cpp name=erase_after_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

void printList(const string& label, const forward_list<int>& fl) {
    cout << label << ": ";
    for (int val : fl) {
        cout << val << " ";
    }
    cout << endl;
}

int main() {

    // ===== 重载1：删除指定位置之后的单个元素 =====
    {
        forward_list<int> fl = {10, 20, 30, 40, 50};
        // fl: [10] → [20] → [30] → [40] → [50]

        auto it = fl.begin();  // 指向 10
        fl.erase_after(it);
        // 删除 10 之后的元素（即 20）
        // fl: [10] → [30] → [40] → [50]
        printList("删除单个", fl);
    }

    // ===== 重载2：删除指定范围 (first, last) 之间的所有元素 =====
    // 注意：是开区间！删除 first 之后、last 之前的所有元素
    {
        forward_list<int> fl = {10, 20, 30, 40, 50};
        // fl: [10] → [20] → [30] → [40] → [50]

        auto first = fl.begin();      // 指向 10
        auto last = fl.end();         // 指向末尾（nullptr）

        fl.erase_after(first, last);
        // 删除 10 之后、end() 之前的所有元素（即 20, 30, 40, 50）
        // fl: [10]
        printList("删除范围", fl);
    }

    // ===== 删除第一个元素（利用 before_begin()）=====
    {
        forward_list<int> fl = {10, 20, 30};
        fl.erase_after(fl.before_begin());
        // 删除 before_begin() 之后的元素（即第一个元素 10）
        // fl: [20] → [30]
        printList("删除首元素", fl);
        // 效果等同于 pop_front()
    }

    return 0;
}
```

**输出**：
```
删除单个: 10 30 40 50
删除范围: 10
删除首元素: 20 30
```

### 2.7 综合实战：在 `forward_list` 中查找并操作

由于 `forward_list` 的特殊性（只能操作"之后"），实际使用中常常需要同时维护**当前迭代器**和**前一个迭代器**。

```cpp name=find_and_operate.cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {

    forward_list<int> fl = {10, 20, 30, 40, 50};

    // 任务：找到值为 30 的元素，在它之后插入 35

    for (auto it = fl.begin(); it != fl.end(); ++it) {
        if (*it == 30) {
            fl.insert_after(it, 35);
            break;
        }
    }
    // fl: 10 → 20 → 30 → 35 → 40 → 50

    cout << "插入35后: ";
    for (int val : fl) cout << val << " ";
    cout << endl;

    // 任务：删除值为 30 的元素
    // 需要找到 30 的前一个节点！

    auto prev = fl.before_begin();  // 从 before_begin 开始
    auto curr = fl.begin();

    while (curr != fl.end()) {
        if (*curr == 30) {
            fl.erase_after(prev);  // 删除 prev 之后的元素（即 curr 指向的 30）
            break;
        }
        prev = curr;
        ++curr;
    }
    // fl: 10 → 20 → 35 → 40 → 50

    cout << "删除30后: ";
    for (int val : fl) cout << val << " ";
    cout << endl;

    return 0;
}
```

**输出**：
```
插入35后: 10 20 30 35 40 50
删除30后: 10 20 35 40 50
```

> 💡 **核心技巧**：在 `forward_list` 中删除指定值的元素时，需要维护一个 `prev` 迭代器（从 `before_begin()` 开始），始终指向当前元素的前一个位置。这是 `forward_list` 编程中最常见的模式。

### 2.8 其他常用接口速查

```cpp name=other_interfaces.cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {

    forward_list<int> fl = {10, 20, 30};

    // ===== assign()：重新赋值 =====
    fl.assign({100, 200, 300});
    // fl: 100 → 200 → 300

    fl.assign(4, 0);
    // fl: 0 → 0 → 0 → 0

    // ===== resize()：调整大小 =====
    fl.assign({10, 20, 30});

    fl.resize(5);       // 扩充到 5 个，新增元素为默认值 0
    // fl: 10 → 20 → 30 → 0 → 0

    fl.resize(2);       // 缩减到 2 个，多余元素被删除
    // fl: 10 → 20

    // ===== clear()：清空 =====
    fl.clear();
    cout << "clear 后是否为空: " << fl.empty() << endl;  // 1

    // ===== swap()：交换两个 forward_list =====
    forward_list<int> a = {1, 2, 3};
    forward_list<int> b = {4, 5, 6};
    a.swap(b);
    // a: 4 → 5 → 6
    // b: 1 → 2 → 3

    cout << "a: ";
    for (int val : a) cout << val << " ";
    cout << endl;

    cout << "b: ";
    for (int val : b) cout << val << " ";
    cout << endl;

    return 0;
}
```

---

## 三、`forward_list` — 成员函数详解

`forward_list` 和 `list` 一样，提供了一组**链表专属的算法成员函数**。这些函数利用链表"修改指针即可重新排列"的特性，比通用的 `<algorithm>` 中的同名函数**效率更高**。

### 3.1 `sort()` — 排序

```cpp name=sort_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {

    forward_list<int> fl = {50, 20, 40, 10, 30};

    // ===== 默认升序排序 =====
    fl.sort();
    cout << "升序: ";
    for (int val : fl) cout << val << " ";
    cout << endl;
    // 输出: 升序: 10 20 30 40 50

    // ===== 自定义排序规则（降序）=====
    fl.sort(greater<int>());
    cout << "降序: ";
    for (int val : fl) cout << val << " ";
    cout << endl;
    // 输出: 降序: 50 40 30 20 10

    // ===== 使用 lambda 自定义排序 =====
    // 按绝对值排序
    forward_list<int> fl2 = {-5, 3, -1, 4, -2};
    fl2.sort([](int a, int b) {
        return abs(a) < abs(b);  // 按绝对值升序
    });
    cout << "按绝对值: ";
    for (int val : fl2) cout << val << " ";
    cout << endl;
    // 输出: 按绝对值: -1 -2 3 4 -5

    return 0;
}
```

> ⚠️ **重要**：不能对 `forward_list` 使用 `std::sort()`（来自 `<algorithm>`），因为 `std::sort()` 要求**随机访问迭代器**，而 `forward_list` 只有前向迭代器。必须使用成员函数 `fl.sort()`。

```cpp name=sort_error.cpp
// ❌ 编译错误！
// #include <algorithm>
// std::sort(fl.begin(), fl.end());  // forward_list 不支持！

// ✅ 正确做法
fl.sort();  // 使用成员函数
```

### 3.2 `merge()` — 合并两个已排序的链表

`merge()` 将另一个**已排序**的 `forward_list` 合并到当前链表中，合并后仍然保持有序。合并完成后，另一个链表变为空。

```cpp name=merge_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

void printList(const string& label, const forward_list<int>& fl) {
    cout << label << ": ";
    for (int val : fl) cout << val << " ";
    cout << endl;
}

int main() {

    // ===== 基本合并（两个链表必须先排好序！）=====
    {
        forward_list<int> fl1 = {10, 30, 50};    // 已排序
        forward_list<int> fl2 = {20, 40, 60};    // 已排序

        fl1.merge(fl2);
        // fl1: 10 → 20 → 30 → 40 → 50 → 60（有序合并）
        // fl2: （空，元素被转移到 fl1 中了）

        printList("合并后 fl1", fl1);
        printList("合并后 fl2", fl2);
    }

    // ===== 自定义比较规则的合并（降序）=====
    {
        forward_list<int> fl1 = {50, 30, 10};    // 降序
        forward_list<int> fl2 = {60, 40, 20};    // 降序

        fl1.merge(fl2, greater<int>());
        // fl1: 60 → 50 → 40 → 30 → 20 → 10（降序合并）

        printList("降序合并后 fl1", fl1);
    }

    // ===== 注意：如果链表未排序就 merge，结果是未定义的！=====
    {
        forward_list<int> fl1 = {30, 10, 50};  // ❌ 未排序
        forward_list<int> fl2 = {40, 20, 60};  // ❌ 未排序

        // fl1.merge(fl2);  // 行为未定义！结果不可预测！
        // 正确做法：先排序再合并
        fl1.sort();
        fl2.sort();
        fl1.merge(fl2);
        printList("先排序再合并", fl1);
    }

    return 0;
}
```

**输出**：
```
合并后 fl1: 10 20 30 40 50 60
合并后 fl2: 
降序合并后 fl1: 60 50 40 30 20 10
先排序再合并: 10 20 30 40 50 60
```

> 💡 **`merge()` 的本质**：它不是复制元素，而是**转移节点指针**。所以 `fl2` 合并后变为空，且整个过程是 O(n) 的。

### 3.3 `reverse()` — 反转链表

```cpp name=reverse_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

int main() {

    forward_list<int> fl = {10, 20, 30, 40, 50};
    // fl: [10] → [20] → [30] → [40] → [50]

    cout << "反转前: ";
    for (int val : fl) cout << val << " ";
    cout << endl;
    // 输出: 反转前: 10 20 30 40 50

    fl.reverse();
    // fl: [50] → [40] → [30] → [20] → [10]

    cout << "反转后: ";
    for (int val : fl) cout << val << " ";
    cout << endl;
    // 输出: 反转后: 50 40 30 20 10

    return 0;
}
```

`reverse()` 通过重新链接节点的指针来反转链表，**不会移动或复制元素本身**，时间复杂度 O(n)。

### 3.4 `unique()` — 去除连续重复元素

`unique()` 删除**相邻的重复元素**，只保留一个。

```cpp name=unique_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

void printList(const string& label, const forward_list<int>& fl) {
    cout << label << ": ";
    for (int val : fl) cout << val << " ";
    cout << endl;
}

int main() {

    // ===== 基本用法：去除相邻重复 =====
    {
        forward_list<int> fl = {10, 10, 20, 20, 20, 30, 10, 10};
        //                       ^^     ^^^^^^^^^         ^^
        //                      重复      重复           重复

        fl.unique();
        // 结果: 10 → 20 → 30 → 10
        // 注意：最后的 10 没有被去掉！因为它和前面的 10 不相邻

        printList("unique后", fl);
    }

    // ===== 如果想去除所有重复（不仅仅是相邻的），先排序！=====
    {
        forward_list<int> fl = {10, 10, 20, 20, 20, 30, 10, 10};

        fl.sort();     // 排序后: 10 → 10 → 10 → 10 → 20 → 20 → 20 → 30
        fl.unique();   // unique后: 10 → 20 → 30

        printList("sort+unique后", fl);
    }

    // ===== 自定义去重规则 =====
    {
        forward_list<int> fl = {10, 11, 20, 21, 22, 30};

        // 自定义规则：如果两个相邻元素的差值小于 5，则视为"重复"
        fl.unique([](int a, int b) {
            return abs(a - b) < 5;
        });
        // 10 和 11 差值为 1 < 5 → 删除 11
        // 10 和 20 差值为 10 ≥ 5 → 保留 20
        // 20 和 21 差值为 1 < 5 → 删除 21
        // 20 和 22 差值为 2 < 5 → 删除 22
        // 20 和 30 差值为 10 ≥ 5 → 保留 30
        // 结果: 10 → 20 → 30

        printList("自定义unique", fl);
    }

    return 0;
}
```

**输出**：
```
unique后: 10 20 30 10
sort+unique后: 10 20 30
自定义unique: 10 20 30
```

### 3.5 `remove()` 和 `remove_if()` — 按值/条件删除

这两个函数虽然不在你的提纲中，但它们是 `forward_list` 非常常用的成员函数，且与 `unique()` 等函数关系密切，这里一并讲解。

```cpp name=remove_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

void printList(const string& label, const forward_list<int>& fl) {
    cout << label << ": ";
    for (int val : fl) cout << val << " ";
    cout << endl;
}

int main() {

    // ===== remove()：删除所有等于指定值的元素 =====
    {
        forward_list<int> fl = {10, 20, 30, 20, 40, 20};

        fl.remove(20);  // 删除所有值为 20 的元素
        // fl: 10 → 30 → 40

        printList("remove(20)", fl);
    }

    // ===== remove_if()：删除满足条件的所有元素 =====
    {
        forward_list<int> fl = {10, 25, 30, 15, 40, 5};

        fl.remove_if([](int val) {
            return val < 20;  // 删除所有小于 20 的元素
        });
        // fl: 25 → 30 → 40

        printList("remove_if(<20)", fl);
    }

    return 0;
}
```

**输出**：
```
remove(20): 10 30 40
remove_if(<20): 25 30 40
```

> 💡 `remove()` 和 `remove_if()` 比手动用 `erase_after()` 逐个删除方便得多，是 `forward_list` 编程中的常用利器。

### 3.6 `splice_after()` — 转移节点（进阶）

`splice_after()` 可以将另一个链表的部分或全部节点**转移**到当前链表中，不涉及元素拷贝，效率很高。

```cpp name=splice_after_demo.cpp
#include <iostream>
#include <forward_list>
using namespace std;

void printList(const string& label, const forward_list<int>& fl) {
    cout << label << ": ";
    for (int val : fl) cout << val << " ";
    cout << endl;
}

int main() {

    // ===== 重载1：将整个链表转移到指定位置之后 =====
    {
        forward_list<int> fl1 = {1, 2, 3};
        forward_list<int> fl2 = {10, 20, 30};

        auto it = fl1.begin();  // 指向 1
        fl1.splice_after(it, fl2);
        // 将 fl2 的所有元素转移到 fl1 中 1 之后
        // fl1: 1 → 10 → 20 → 30 → 2 → 3
        // fl2: （空）

        printList("fl1", fl1);
        printList("fl2", fl2);
    }

    // ===== 重载2：只转移另一个链表中某位置之后的单个元素 =====
    {
        forward_list<int> fl1 = {1, 2, 3};
        forward_list<int> fl2 = {10, 20, 30};

        fl1.splice_after(fl1.begin(), fl2, fl2.begin());
        // 将 fl2 中 begin()（指向10）之后的元素（即20）转移到 fl1 的 begin()（指向1）之后
        // fl1: 1 → 20 → 2 → 3
        // fl2: 10 → 30

        printList("fl1", fl1);
        printList("fl2", fl2);
    }

    // ===== 重载3：转移另一个链表中指定范围的元素 =====
    {
        forward_list<int> fl1 = {1, 2, 3};
        forward_list<int> fl2 = {10, 20, 30, 40, 50};

        auto first = fl2.begin();        // 指向 10
        auto last  = fl2.end();          // end

        // 第一个参数也可以用 next 来精确定位
        auto pos = fl2.begin();
        advance(pos, 2);                 // 指向 30

        fl1.splice_after(fl1.begin(), fl2, first, pos);
        // 将 fl2 中 (first, pos) 即 (10, 30) 开区间内的元素转移
        // 即转移 20
        // fl1: 1 → 20 → 2 → 3
        // fl2: 10 → 30 → 40 → 50

        printList("fl1", fl1);
        printList("fl2", fl2);
    }

    return 0;
}
```

---

## 四、综合实战与总结

### 4.1 综合案例：学生成绩管理

```cpp name=student_score.cpp
#include <iostream>
#include <forward_list>
#include <string>
using namespace std;

struct Student {
    string name;
    int score;

    // 方便输出
    friend ostream& operator<<(ostream& os, const Student& s) {
        os << "{" << s.name << ", " << s.score << "}";
        return os;
    }
};

void printStudents(const string& label, const forward_list<Student>& fl) {
    cout << label << ": ";
    for (const auto& s : fl) {
        cout << s << " ";
    }
    cout << endl;
}

int main() {

    // 1. 创建学生链表
    forward_list<Student> students = {
        {"Alice", 85},
        {"Bob", 72},
        {"Charlie", 90},
        {"Diana", 72},
        {"Eve", 95}
    };
    printStudents("初始", students);

    // 2. 在头部添加新学生
    students.emplace_front("Frank", 88);
    printStudents("添加Frank", students);

    // 3. 按分数排序（降序）
    students.sort([](const Student& a, const Student& b) {
        return a.score > b.score;  // 降序
    });
    printStudents("按分数降序", students);

    // 4. 去除相邻分数相同的学生（只保留第一个）
    students.unique([](const Student& a, const Student& b) {
        return a.score == b.score;
    });
    printStudents("去重后", students);

    // 5. 删除所有分数低于 80 的学生
    students.remove_if([](const Student& s) {
        return s.score < 80;
    });
    printStudents("删除<80分", students);

    // 6. 反转链表
    students.reverse();
    printStudents("反转后", students);

    return 0;
}
```

**输出**：
```
初始: {Alice, 85} {Bob, 72} {Charlie, 90} {Diana, 72} {Eve, 95} 
添加Frank: {Frank, 88} {Alice, 85} {Bob, 72} {Charlie, 90} {Diana, 72} {Eve, 95} 
按分数降序: {Eve, 95} {Charlie, 90} {Frank, 88} {Alice, 85} {Bob, 72} {Diana, 72} 
去重后: {Eve, 95} {Charlie, 90} {Frank, 88} {Alice, 85} {Bob, 72} 
删除<80分: {Eve, 95} {Charlie, 90} {Frank, 88} {Alice, 85} 
反转后: {Alice, 85} {Frank, 88} {Charlie, 90} {Eve, 95} 
```

### 4.2 `forward_list` 全部接口速查表

| 接口                                | 说明                   | 时间复杂度   |
| ----------------------------------- | ---------------------- | ------------ |
| **构造/赋值**                       |                        |              |
| `forward_list<T> fl`                | 默认构造               | O(1)         |
| `forward_list<T> fl(n, val)`        | n 个 val               | O(n)         |
| `forward_list<T> fl{a,b,c}`         | 初始化列表             | O(n)         |
| `fl.assign(...)`                    | 重新赋值               | O(n)         |
| **容量**                            |                        |              |
| `fl.empty()`                        | 是否为空               | O(1)         |
| `fl.max_size()`                     | 最大容量               | O(1)         |
| ~~`fl.size()`~~                     | ❌ **不存在**           | —            |
| **访问**                            |                        |              |
| `fl.front()`                        | 访问第一个元素         | O(1)         |
| ~~`fl.back()`~~                     | ❌ **不存在**           | —            |
| **迭代器**                          |                        |              |
| `fl.begin()` / `fl.end()`           | 前向迭代器             | O(1)         |
| `fl.before_begin()`                 | 首元素之前（哨兵）     | O(1)         |
| `fl.cbegin()` / `fl.cend()`         | const 版本             | O(1)         |
| `fl.cbefore_begin()`                | const 哨兵             | O(1)         |
| **插入**                            |                        |              |
| `fl.push_front(val)`                | 头部插入               | O(1)         |
| `fl.emplace_front(args...)`         | 头部原地构造           | O(1)         |
| `fl.insert_after(pos, val)`         | pos 之后插入           | O(1)         |
| `fl.insert_after(pos, n, val)`      | pos 之后插入 n 个      | O(n)         |
| `fl.insert_after(pos, first, last)` | pos 之后插入范围       | O(m)         |
| `fl.insert_after(pos, {a,b,c})`     | pos 之后插入列表       | O(m)         |
| `fl.emplace_after(pos, args...)`    | pos 之后原地构造       | O(1)         |
| **删除**                            |                        |              |
| `fl.pop_front()`                    | 删除头部               | O(1)         |
| `fl.erase_after(pos)`               | 删除 pos 之后的元素    | O(1)         |
| `fl.erase_after(first, last)`       | 删除范围 (first, last) | O(m)         |
| `fl.remove(val)`                    | 删除所有等于 val 的    | O(n)         |
| `fl.remove_if(pred)`                | 删除满足条件的         | O(n)         |
| `fl.clear()`                        | 清空                   | O(n)         |
| **链表操作**                        |                        |              |
| `fl.sort()`                         | 排序（默认升序）       | O(n log n)   |
| `fl.sort(comp)`                     | 自定义排序             | O(n log n)   |
| `fl.merge(other)`                   | 合并已排序链表         | O(n + m)     |
| `fl.reverse()`                      | 反转                   | O(n)         |
| `fl.unique()`                       | 去除相邻重复           | O(n)         |
| `fl.unique(pred)`                   | 自定义去重             | O(n)         |
| `fl.splice_after(...)`              | 节点转移               | O(1) 或 O(m) |
| **其他**                            |                        |              |
| `fl.swap(other)`                    | 交换                   | O(1)         |
| `fl.resize(n)`                      | 调整大小               | O(n)         |

### 4.3 何时选择 `forward_list`？

| 场景                                                  | 推荐容器                    |
| ----------------------------------------------------- | --------------------------- |
| 需要频繁在任意位置插入/删除，且需要双向遍历           | `list`                      |
| 需要频繁在头部插入/删除，只需前向遍历，**极致省内存** | **`forward_list`**          |
| 需要随机访问（`[]` 运算符）                           | `vector` / `deque`          |
| 需要尾部高效插入/删除                                 | `vector` / `deque` / `list` |
| 替代 C 语言手写单向链表                               | **`forward_list`**          |

> **实话实说**：在实际 C++ 工程中，`forward_list` 的使用频率远低于 `vector`、`list`，甚至低于 `deque`。它的主要适用场景是：**嵌入式系统或对内存极度敏感的环境**，以及**替代 C 语言手写单向链表**的场景。但学习它能帮你加深对链表数据结构和 C++ 迭代器体系的理解。

### 4.4 常见易错点总结

```
❌ 错误1：试图调用 size()
   fl.size();  // 编译错误！forward_list 没有 size()
   ✅ 正确：std::distance(fl.begin(), fl.end());  // O(n)

❌ 错误2：试图调用 push_back()
   fl.push_back(100);  // 编译错误！
   ✅ 正确：fl.push_front(100);  // 只能在头部插入

❌ 错误3：对 before_begin() 解引用
   auto it = fl.before_begin();
   cout << *it;  // 未定义行为！
   ✅ 正确：before_begin() 只用于 insert_after / erase_after

❌ 错误4：使用 std::sort()
   std::sort(fl.begin(), fl.end());  // 编译错误！
   ✅ 正确：fl.sort();  // 使用成员函数

❌ 错误5：未排序就 merge
   fl1.merge(fl2);  // 如果 fl1 或 fl2 未排序，结果未定义！
   ✅ 正确：先 sort()，再 merge()

❌ 错误6：混淆 insert 和 insert_after
   fl.insert(it, val);  // 编译错误！forward_list 没有 insert()
   ✅ 正确：fl.insert_after(it, val);

❌ 错误7：混淆 erase 和 erase_after
   fl.erase(it);  // 编译错误！forward_list 没有 erase()
   ✅ 正确：fl.erase_after(prev_it);  // 需要前一个位置的迭代器

❌ 错误8：用 -- 后退迭代器
   auto it = fl.begin();
   --it;  // 编译错误！前向迭代器不支持 --
   ✅ 正确：forward_list 只能向前遍历（++it）
```

---

> **阶段小结**：`forward_list` 是 C++ 标准库中唯一的单向链表容器。它以牺牲部分便利性（没有 `size()`、`push_back()`、`back()` 等）为代价，换取了**最小的内存开销**和**与手写 C 单向链表完全一致的性能**。掌握它的关键在于理解"**所有操作都基于'之后'**"这一核心思想，以及熟练使用 `before_begin()` 这个哨兵迭代器。