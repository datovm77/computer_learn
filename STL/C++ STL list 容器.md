# C++ STL `list` 容器完全教程

---

## 目录

1. [list 容器 - 基本概念](#1-list-容器---基本概念)
2. [list 容器 - 构造函数](#2-list-容器---构造函数)
3. [list 容器 - 赋值和交换](#3-list-容器---赋值和交换)
4. [list 容器 - 大小操作](#4-list-容器---大小操作)
5. [list 容器 - 插入和删除](#5-list-容器---插入和删除)
6. [list 容器 - 数据存取](#6-list-容器---数据存取)
7. [list 容器 - 反转和排序](#7-list-容器---反转和排序)
8. [list 容器 - 排序案例（综合实战）](#8-list-容器---排序案例综合实战)

---

## 1. list 容器 - 基本概念

### 1.1 什么是 list？

`list` 是 C++ STL（标准模板库）中的**序列式容器**，其底层通常实现为**双向链表**（很多实现会使用循环哨兵结点）。

要使用 `list`，需要包含头文件：

```cpp name=include_example.cpp
#include <list>
```

### 1.2 链表基础回顾

在理解 `list` 之前，我们先回顾一下链表的基本知识。

**链表**是一种物理存储单元上**非连续、非顺序**的存储结构。数据元素之间的逻辑顺序是通过链表中的**指针链接**实现的。

链表由一系列**结点（Node）**组成，每个结点包括：
- **数据域**：存储实际的数据元素
- **指针域**：存储指向下一个（或上一个）结点的指针

链表的种类：

| 类型             | 结构特点                                       |
| ---------------- | ---------------------------------------------- |
| **单向链表**     | 每个结点有一个指针，指向下一个结点             |
| **双向链表**     | 每个结点有两个指针，分别指向前一个和后一个结点 |
| **循环链表**     | 尾结点的指针指回头结点，形成环                 |
| **双向循环链表** | 双向链表 + 循环链表的结合体                    |

**可以把 `list` 理解为双向链表容器（常见实现带循环哨兵）。**

其结点结构在概念上等价于：

```cpp name=list_node_concept.cpp
// 概念性展示，非STL源码
template<typename T>
struct ListNode {
    T data;           // 数据域
    ListNode* prev;   // 指向前驱结点的指针
    ListNode* next;   // 指向后继结点的指针
};
```

内存示意图：

```
       ┌─────────────────────────────────────────────────────┐
       │                                                     │
       ▼                                                     │
  ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
  │  prev   │◄────│  prev   │◄────│  prev   │◄────│  prev   │
  │ data: 10│     │ data: 20│     │ data: 30│     │ data: 40│
  │  next   │────►│  next   │────►│  next   │────►│  next   │
  └─────────┘     └─────────┘     └─────────┘     └─────────┘
       │                                                    ▲
      └─────────────────────────────────────────────────────┘
```

### 1.3 list 与 vector、deque 的对比

为什么已经有了 `vector` 和 `deque`，还需要 `list`？它们各有优劣：

| 特性          | `vector`                  | `deque`                   | `list`                         |
| ------------- | ------------------------- | ------------------------- | ------------------------------ |
| 底层结构      | 动态数组（连续内存）      | 分段连续数组              | 双向链表（常见实现带循环哨兵） |
| 随机访问      | ✅ 支持，O(1)              | ✅ 支持，O(1)              | ❌ 不支持                       |
| 头部插入/删除 | ❌ O(n)                    | ✅ O(1)                    | ✅ O(1)                         |
| 尾部插入/删除 | ✅ O(1) 均摊               | ✅ O(1)                    | ✅ O(1)                         |
| 中间插入/删除 | ❌ O(n)                    | ❌ O(n)                    | ✅ O(1)（已定位时）             |
| 迭代器失效    | 插入/删除可能导致全部失效 | 插入/删除可能导致全部失效 | 仅被删除元素的迭代器失效       |
| 内存利用      | 连续，缓存友好            | 分段连续                  | 不连续，每个结点有额外指针开销 |

### 1.4 list 的优点

1. **在任意位置插入和删除元素非常高效**：时间复杂度为 O(1)（前提是已经通过迭代器定位到了该位置）
2. **动态内存管理**：不需要预先分配大块连续内存，每次插入仅分配一个结点的空间
3. **迭代器稳定性极好**：插入操作不会导致其他迭代器失效；删除操作只会使被删除元素的迭代器失效

### 1.5 list 的缺点

1. **不支持随机访问**：不能用下标 `[]` 或 `at()` 访问元素，只能通过迭代器从头（或尾）逐个遍历
2. **内存开销较大**：每个结点除了存储数据外，还需要额外存储两个指针（prev 和 next）
3. **缓存不友好**：结点在内存中不连续，CPU 缓存命中率低，遍历速度不如 `vector`

### 1.6 list 的迭代器

这是一个非常重要的概念。

`list` 的迭代器是**双向迭代器（Bidirectional Iterator）**，不是随机访问迭代器。

这意味着：

```cpp name=iterator_operations.cpp
list<int>::iterator it;

// ✅ 支持的操作
it++;       // 前进一步
it--;       // 后退一步
++it;       // 前进一步（前置版本，效率更高）
--it;       // 后退一步（前置版本，效率更高）
*it;        // 解引用，获取当前元素
it == it2;  // 比较
it != it2;  // 比较

// ❌ 不支持的操作（以下代码会编译报错！）
it + 5;     // 不能跳跃前进
it - 3;     // 不能跳跃后退
it[2];      // 不能下标访问
it < it2;   // 不能用 <, >, <=, >= 比较
```

> **核心记忆点**：`list` 的迭代器只能「一步一步走」，不能「跳着走」。

### 1.7 第一个 list 程序

```cpp name=first_list.cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    // 创建一个存放 int 类型的 list
    list<int> myList;

    // 向尾部添加元素
    myList.push_back(10);
    myList.push_back(20);
    myList.push_back(30);

    // 向头部添加元素
    myList.push_front(5);

    // 使用迭代器遍历并输出
    cout << "list 中的元素: ";
    for (list<int>::iterator it = myList.begin(); it != myList.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
    // 输出: list 中的元素: 5 10 20 30

    return 0;
}
```

---

## 2. list 容器 - 构造函数

`list` 提供了多种构造方式，满足不同场景的初始化需求。

### 2.1 构造函数一览

| 构造方式       | 函数原型                          | 说明                             |
| -------------- | --------------------------------- | -------------------------------- |
| 默认构造       | `list<T> lst;`                    | 创建空的 list                    |
| 填充构造       | `list<T> lst(n, elem);`           | 用 n 个 elem 元素初始化          |
| 范围构造       | `list<T> lst(beg, end);`          | 用 `[beg, end)` 区间的元素初始化 |
| 拷贝构造       | `list<T> lst(const list& other);` | 拷贝另一个 list                  |
| 初始化列表构造 | `list<T> lst{e1, e2, e3, ...};`   | 使用初始化列表（C++11）          |

### 2.2 逐一详解与示例

#### 2.2.1 默认构造函数

创建一个空的 `list` 容器，不含任何元素。

```cpp name=default_constructor.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (list<int>::const_iterator it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    // 默认构造：创建空 list
    list<int> L1;
    cout << "L1 的大小: " << L1.size() << endl;   // 输出: 0
    cout << "L1 是否为空: " << L1.empty() << endl; // 输出: 1 (true)

    return 0;
}
```

#### 2.2.2 填充构造函数

创建一个包含 n 个相同元素的 `list`。

```cpp name=fill_constructor.cpp
int main() {
    // 填充构造：10个值为100的元素
    list<int> L2(10, 100);

    cout << "L2: ";
    printList(L2);
    // 输出: L2: 100 100 100 100 100 100 100 100 100 100

    // 只指定个数，不指定值 —— 元素会被值初始化（int 默认为0）
    list<int> L2b(5);
    cout << "L2b: ";
    printList(L2b);
    // 输出: L2b: 0 0 0 0 0

    return 0;
}
```

#### 2.2.3 范围构造函数

用另一个容器（或数组）的某个区间来初始化 `list`。

```cpp name=range_constructor.cpp
int main() {
    // 先创建一个 list
    list<int> L2(10, 100);

    // 范围构造：用 L2 的 [begin, end) 区间构造 L3
    list<int> L3(L2.begin(), L2.end());
    cout << "L3: ";
    printList(L3);
    // 输出: L3: 100 100 100 100 100 100 100 100 100 100

    // 也可以从数组构造
    int arr[] = {1, 2, 3, 4, 5};
    list<int> L3b(arr, arr + 5);
    cout << "L3b: ";
    printList(L3b);
    // 输出: L3b: 1 2 3 4 5

    // 甚至可以从 vector 构造
    // vector<int> v = {10, 20, 30};
    // list<int> L3c(v.begin(), v.end());

    return 0;
}
```

#### 2.2.4 拷贝构造函数

用一个已有的 `list` 完整拷贝出一个新的 `list`（深拷贝）。

```cpp name=copy_constructor.cpp
int main() {
    list<int> L3 = {10, 20, 30, 40, 50};

    // 拷贝构造
    list<int> L4(L3);
    cout << "L4: ";
    printList(L4);
    // 输出: L4: 10 20 30 40 50

    // 修改 L4 不会影响 L3（深拷贝）
    L4.push_back(60);
    cout << "修改后 L4: ";
    printList(L4);  // 10 20 30 40 50 60
    cout << "L3 不受影响: ";
    printList(L3);  // 10 20 30 40 50

    return 0;
}
```

#### 2.2.5 初始化列表构造（C++11）

使用花括号 `{}` 直接列出元素，这是最简洁直观的方式。

```cpp name=initializer_list_constructor.cpp
int main() {
    // 初始化列表构造 (C++11)
    list<int> L5 = {10, 20, 30, 40, 50};

    // 也可以写成
    list<int> L5b{10, 20, 30, 40, 50};

    cout << "L5: ";
    printList(L5);
    // 输出: L5: 10 20 30 40 50

    // 对于 string 类型
    list<string> names = {"Alice", "Bob", "Charlie"};

    return 0;
}
```

### 2.3 完整示例

```cpp name=constructor_complete.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (list<int>::const_iterator it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    // 1. 默认构造
    list<int> L1;

    // 2. 填充构造
    list<int> L2(10, 100);

    // 3. 范围构造
    list<int> L3(L2.begin(), L2.end());

    // 4. 拷贝构造
    list<int> L4(L3);

    // 5. 初始化列表构造
    list<int> L5 = {1, 2, 3, 4, 5};

    cout << "L1 (默认构造):      ";  printList(L1);
    cout << "L2 (填充构造):      ";  printList(L2);
    cout << "L3 (范围构造):      ";  printList(L3);
    cout << "L4 (拷贝构造):      ";  printList(L4);
    cout << "L5 (初始化列表构造): ";  printList(L5);

    return 0;
}
```

运行结果：

```
L1 (默认构造):      
L2 (填充构造):      100 100 100 100 100 100 100 100 100 100 
L3 (范围构造):      100 100 100 100 100 100 100 100 100 100 
L4 (拷贝构造):      100 100 100 100 100 100 100 100 100 100 
L5 (初始化列表构造): 1 2 3 4 5 
```

---

## 3. list 容器 - 赋值和交换

### 3.1 赋值操作一览

| 操作                  | 函数原型                              | 说明                      |
| --------------------- | ------------------------------------- | ------------------------- |
| `operator=`           | `list& operator=(const list& other);` | 重载赋值运算符            |
| `assign` (填充)       | `void assign(int n, T elem);`         | 用 n 个 elem 赋值         |
| `assign` (范围)       | `void assign(beg, end);`              | 用 `[beg, end)` 区间赋值  |
| `assign` (初始化列表) | `void assign(initializer_list<T>);`   | 用初始化列表赋值（C++11） |

### 3.2 详解与示例

#### 3.2.1 operator= 赋值

```cpp name=assignment_operator.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (list<int>::const_iterator it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    list<int> L1 = {10, 20, 30, 40, 50};

    // 使用 = 运算符赋值
    list<int> L2;
    L2 = L1;

    cout << "L1: "; printList(L1);  // 10 20 30 40 50
    cout << "L2: "; printList(L2);  // 10 20 30 40 50

    // L2 是 L1 的深拷贝
    L2.push_back(60);
    cout << "修改后 L2: "; printList(L2);  // 10 20 30 40 50 60
    cout << "L1 不变:   "; printList(L1);  // 10 20 30 40 50

    return 0;
}
```

#### 3.2.2 assign 赋值

`assign` 会**清空原有内容**，然后用新的值重新填充。

```cpp name=assign_example.cpp
int main() {
    list<int> L1 = {10, 20, 30};

    // assign - 范围赋值
    list<int> L2;
    L2.assign(L1.begin(), L1.end());
    cout << "L2 (范围赋值): ";
    printList(L2);  // 10 20 30

    // assign - 填充赋值
    list<int> L3;
    L3.assign(5, 100);
    cout << "L3 (填充赋值): ";
    printList(L3);  // 100 100 100 100 100

    // assign 会清空原有元素
    list<int> L4 = {1, 2, 3, 4, 5, 6, 7};
    cout << "L4 赋值前: "; printList(L4);  // 1 2 3 4 5 6 7
    L4.assign(3, 999);
    cout << "L4 赋值后: "; printList(L4);  // 999 999 999

    // assign - 初始化列表赋值 (C++11)
    list<int> L5;
    L5.assign({100, 200, 300});
    cout << "L5 (初始化列表赋值): ";
    printList(L5);  // 100 200 300

    return 0;
}
```

### 3.3 交换操作 - swap

`swap` 用于交换两个 `list` 容器的内容。

**核心要点**：`swap` 操作的时间复杂度为 **O(1)**，它只是交换了两个 list 内部的指针，并不会逐元素拷贝。

```cpp name=swap_example.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    list<int> L1 = {10, 20, 30};
    list<int> L2 = {100, 200, 300, 400, 500};

    cout << "===== 交换前 =====" << endl;
    cout << "L1: "; printList(L1);  // 10 20 30
    cout << "L2: "; printList(L2);  // 100 200 300 400 500
    cout << "L1 大小: " << L1.size() << endl;  // 3
    cout << "L2 大小: " << L2.size() << endl;  // 5

    // 交换
    L1.swap(L2);

    cout << "\n===== 交换后 =====" << endl;
    cout << "L1: "; printList(L1);  // 100 200 300 400 500
    cout << "L2: "; printList(L2);  // 10 20 30
    cout << "L1 大小: " << L1.size() << endl;  // 5
    cout << "L2 大小: " << L2.size() << endl;  // 3

    return 0;
}
```

> **小提示**：也可以使用 `std::swap(L1, L2);` 达到同样的效果。

---

## 4. list 容器 - 大小操作

### 4.1 大小操作一览

| 函数       | 原型                               | 说明                                |
| ---------- | ---------------------------------- | ----------------------------------- |
| `size()`   | `size_type size() const;`          | 返回容器中元素的个数                |
| `empty()`  | `bool empty() const;`              | 判断容器是否为空                    |
| `resize()` | `void resize(size_type n);`        | 重新指定容器长度                    |
| `resize()` | `void resize(size_type n, T val);` | 重新指定长度，多出的部分用 val 填充 |

> **注意**：`list` **没有** `capacity()` 方法！因为链表不需要预先分配连续内存，不存在「容量」的概念。

### 4.2 size() 和 empty()

```cpp name=size_empty.cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> L1;
    cout << "L1 是否为空: " << (L1.empty() ? "是" : "否") << endl;  // 是
    cout << "L1 的大小:   " << L1.size() << endl;                    // 0

    L1.push_back(10);
    L1.push_back(20);
    L1.push_back(30);

    cout << "L1 是否为空: " << (L1.empty() ? "是" : "否") << endl;  // 否
    cout << "L1 的大小:   " << L1.size() << endl;                    // 3

    return 0;
}
```

> **最佳实践**：判断容器是否为空时，优先使用 `empty()` 而非 `size() == 0`。`empty()` 对所有容器保证是 O(1) 时间复杂度。

### 4.3 resize() 重新指定大小

`resize` 可以改变 `list` 的元素个数。有两种情况：

**情况一：新大小 > 原大小** → 多出的位置用默认值（或指定值）填充

**情况二：新大小 < 原大小** → 多余的尾部元素被删除

```cpp name=resize_example.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    list<int> L1 = {1, 2, 3, 4, 5};
    cout << "初始 L1: ";
    printList(L1);  // 1 2 3 4 5
    cout << "大小: " << L1.size() << endl;  // 5

    // --- 情况一：变大，默认填充 0 ---
    L1.resize(8);
    cout << "\nresize(8) 后: ";
    printList(L1);  // 1 2 3 4 5 0 0 0
    cout << "大小: " << L1.size() << endl;  // 8

    // --- 情况一变体：变大，指定填充值 ---
    list<int> L2 = {1, 2, 3, 4, 5};
    L2.resize(8, 999);
    cout << "\nresize(8, 999) 后: ";
    printList(L2);  // 1 2 3 4 5 999 999 999
    cout << "大小: " << L2.size() << endl;  // 8

    // --- 情况二：变小，多余元素被删除 ---
    list<int> L3 = {1, 2, 3, 4, 5};
    L3.resize(3);
    cout << "\nresize(3) 后: ";
    printList(L3);  // 1 2 3
    cout << "大小: " << L3.size() << endl;  // 3

    return 0;
}
```

### 4.4 关于 max_size()

`list` 还有一个 `max_size()` 成员函数，返回容器理论上能容纳的最大元素数。这个值取决于系统和编译器，实际中很少使用。

```cpp name=max_size_example.cpp
list<int> L;
cout << "max_size: " << L.max_size() << endl;
// 输出类似: max_size: 768614336404564650 （因平台而异）
```

---

## 5. list 容器 - 插入和删除

插入和删除是 `list` 最核心、最强大的功能。这正是链表结构的优势所在。

### 5.1 接口一览

#### 两端操作

| 函数               | 说明               |
| ------------------ | ------------------ |
| `push_back(elem)`  | 在尾部添加一个元素 |
| `pop_back()`       | 删除最后一个元素   |
| `push_front(elem)` | 在头部添加一个元素 |
| `pop_front()`      | 删除第一个元素     |

#### 指定位置操作

| 函数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `insert(pos, elem)`     | 在迭代器 pos 指向的位置**之前**插入 elem，返回新元素的迭代器 |
| `insert(pos, n, elem)`  | 在迭代器 pos 指向的位置之前插入 n 个 elem                    |
| `insert(pos, beg, end)` | 在迭代器 pos 指向的位置之前插入 `[beg, end)` 区间的元素      |
| `erase(pos)`            | 删除迭代器 pos 指向的元素，返回下一个元素的迭代器            |
| `erase(beg, end)`       | 删除 `[beg, end)` 区间的所有元素，返回下一个元素的迭代器     |
| `remove(elem)`          | 删除容器中所有与 elem 值匹配的元素                           |
| `clear()`               | 清空所有元素                                                 |

### 5.2 两端操作详解

```cpp name=push_pop.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    list<int> L;

    // push_back: 尾部插入
    L.push_back(10);
    L.push_back(20);
    L.push_back(30);
    cout << "尾部插入后: ";
    printList(L);  // 10 20 30

    // push_front: 头部插入
    L.push_front(5);
    L.push_front(1);
    cout << "头部插入后: ";
    printList(L);  // 1 5 10 20 30

    // pop_back: 尾部删除
    L.pop_back();
    cout << "尾部删除后: ";
    printList(L);  // 1 5 10 20

    // pop_front: 头部删除
    L.pop_front();
    cout << "头部删除后: ";
    printList(L);  // 5 10 20

    return 0;
}
```

### 5.3 insert 详解

```cpp name=insert_example.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    list<int> L = {10, 20, 30, 40, 50};
    cout << "初始: ";
    printList(L);  // 10 20 30 40 50

    // --- insert(pos, elem) ---
    // 在第二个元素（20）之前插入 15
    list<int>::iterator it = L.begin();
    it++;  // 现在 it 指向 20
    L.insert(it, 15);
    cout << "insert(it, 15): ";
    printList(L);  // 10 15 20 30 40 50
    // 注意：it 仍然指向 20，没有失效！这就是 list 的优势

    // --- insert(pos, n, elem) ---
    // 在 20 之前插入 3 个 99
    L.insert(it, 3, 99);
    cout << "insert(it, 3, 99): ";
    printList(L);  // 10 15 99 99 99 20 30 40 50

    // --- insert(pos, beg, end) ---
    // 在开头插入另一个容器的内容
    list<int> L2 = {1000, 2000};
    L.insert(L.begin(), L2.begin(), L2.end());
    cout << "insert(begin, L2): ";
    printList(L);  // 1000 2000 10 15 99 99 99 20 30 40 50

    return 0;
}
```

### 5.4 erase 详解

```cpp name=erase_example.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    list<int> L = {10, 20, 30, 40, 50, 60, 70};
    cout << "初始: ";
    printList(L);  // 10 20 30 40 50 60 70

    // --- erase(pos) ---
    // 删除第一个元素
    L.erase(L.begin());
    cout << "erase(begin): ";
    printList(L);  // 20 30 40 50 60 70

    // --- erase(beg, end) ---
    // 删除前3个元素
    auto it_beg = L.begin();
    auto it_end = L.begin();
    advance(it_end, 3);  // advance 让迭代器前进3步，因为 list 不支持 it + 3
    L.erase(it_beg, it_end);
    cout << "erase(前3个): ";
    printList(L);  // 50 60 70

    return 0;
}
```

> **重要**：`erase` 返回的是被删除元素的**下一个**元素的迭代器。在循环中删除元素时，一定要正确接收返回值，否则迭代器会失效！

#### 循环中安全删除的正确写法

```cpp name=safe_erase_in_loop.cpp
int main() {
    list<int> L = {1, 2, 3, 2, 4, 2, 5};
    cout << "原始: ";
    printList(L);  // 1 2 3 2 4 2 5

    // 删除所有值为 2 的元素
    for (auto it = L.begin(); it != L.end(); /* 不在这里 it++ */) {
        if (*it == 2) {
            it = L.erase(it);  // erase 返回下一个元素的迭代器
        } else {
            it++;  // 只有不删除时才手动前进
        }
    }

    cout << "删除2后: ";
    printList(L);  // 1 3 4 5

    return 0;
}
```

### 5.5 remove 详解

`remove` 是 `list` 的成员函数，它会**删除容器中所有等于指定值的元素**。

```cpp name=remove_example.cpp
int main() {
    list<int> L = {1, 2, 3, 2, 4, 2, 5};
    cout << "原始: ";
    printList(L);  // 1 2 3 2 4 2 5

    // 删除所有值为 2 的元素（一行搞定，比手动循环 erase 简洁）
    L.remove(2);
    cout << "remove(2) 后: ";
    printList(L);  // 1 3 4 5

    return 0;
}
```

> **对比**：`erase` 是按位置删除，`remove` 是按值删除。

### 5.6 clear 清空

```cpp name=clear_example.cpp
int main() {
    list<int> L = {1, 2, 3, 4, 5};
    cout << "清空前大小: " << L.size() << endl;  // 5

    L.clear();
    cout << "清空后大小: " << L.size() << endl;  // 0
    cout << "是否为空: " << L.empty() << endl;    // 1

    return 0;
}
```

### 5.7 emplace 系列（C++11，进阶）

`emplace` 系列函数是 `insert`/`push` 的高效替代，它们**原地构造**元素，避免了临时对象的拷贝或移动。

| 函数                     | 说明                          |
| ------------------------ | ----------------------------- |
| `emplace_back(args...)`  | 在尾部原地构造元素            |
| `emplace_front(args...)` | 在头部原地构造元素            |
| `emplace(pos, args...)`  | 在迭代器 pos 之前原地构造元素 |

```cpp name=emplace_example.cpp
#include <iostream>
#include <list>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;

    Person(string n, int a) : name(n), age(a) {
        cout << "  [构造] " << name << endl;
    }

    Person(const Person& p) : name(p.name), age(p.age) {
        cout << "  [拷贝构造] " << name << endl;
    }
};

int main() {
    list<Person> L;

    cout << "--- push_back ---" << endl;
    L.push_back(Person("Alice", 25));
    // 输出：[构造] Alice
    //       [拷贝构造] Alice  ← 多了一次拷贝！

    cout << "\n--- emplace_back ---" << endl;
    L.emplace_back("Bob", 30);
    // 输出：[构造] Bob  ← 直接在容器内部构造，无拷贝！

    return 0;
}
```

> **总结**：当存储的是自定义类时，优先使用 `emplace` 系列，性能更好。

---

## 6. list 容器 - 数据存取

### 6.1 核心概念

**`list` 不支持随机访问！** 这是它与 `vector`/`deque` 最大的区别之一。

```cpp name=no_random_access.cpp
list<int> L = {10, 20, 30};

// ❌ 以下代码会编译报错！
// cout << L[0] << endl;    // 错误：list 没有 operator[]
// cout << L.at(1) << endl; // 错误：list 没有 at()
```

### 6.2 可用的数据存取接口

| 函数      | 说明                   |
| --------- | ---------------------- |
| `front()` | 返回第一个元素的引用   |
| `back()`  | 返回最后一个元素的引用 |

就这两个！非常简洁，因为 `list` 只能直接访问头部和尾部。

### 6.3 front() 和 back()

```cpp name=front_back.cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> L = {10, 20, 30, 40, 50};

    // 获取第一个元素
    cout << "第一个元素 (front): " << L.front() << endl;  // 10

    // 获取最后一个元素
    cout << "最后一个元素 (back): " << L.back() << endl;   // 50

    // front() 和 back() 返回的是引用，可以用来修改元素
    L.front() = 999;
    L.back() = 888;

    cout << "修改后第一个元素: " << L.front() << endl;   // 999
    cout << "修改后最后一个元素: " << L.back() << endl;   // 888

    return 0;
}
```

### 6.4 通过迭代器访问中间元素

既然不能随机访问，要访问中间的元素就只能用迭代器"走过去"。

```cpp name=access_middle.cpp
#include <iostream>
#include <list>
#include <iterator>  // for std::advance
using namespace std;

int main() {
    list<int> L = {10, 20, 30, 40, 50};

    // 方法1：手动移动迭代器
    list<int>::iterator it = L.begin();
    it++;  // 指向第2个元素
    it++;  // 指向第3个元素
    cout << "第3个元素: " << *it << endl;  // 30

    // 方法2：使用 std::advance（推荐）
    list<int>::iterator it2 = L.begin();
    advance(it2, 3);  // 前进3步，指向第4个元素
    cout << "第4个元素: " << *it2 << endl;  // 40

    // 方法3：使用 std::next（C++11，推荐，不修改原迭代器）
    auto it3 = next(L.begin(), 2);  // 从 begin 前进2步
    cout << "第3个元素: " << *it3 << endl;  // 30

    // std::prev 则是后退
    auto it4 = prev(L.end(), 1);  // 从 end 后退1步，指向最后一个元素
    cout << "最后一个元素: " << *it4 << endl;  // 50

    return 0;
}
```

### 6.5 为什么 list 不支持随机访问？

底层原因分析：

- `vector` 的元素存储在**连续内存**中。第 i 个元素的地址 = 起始地址 + i × 元素大小。所以 `O(1)` 就能定位。
- `list` 的结点**散落在内存各处**，没有公式可以直接算出第 i 个结点的地址。只能从头结点开始，顺着指针一个一个数过去，时间复杂度为 `O(n)`。

```
vector 的内存布局（连续）：
┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │   → 直接算地址，O(1) 访问
└────┴────┴────┴────┴────┘

list 的内存布局（分散）：
地址0x100: [30] ──→ 地址0x500
地址0x300: [10] ──→ 地址0x800
地址0x500: [40] ──→ 地址0x900
地址0x800: [20] ──→ 地址0x100    → 只能顺着指针找，O(n) 访问
地址0x900: [50] ──→ ...
```

### 6.6 迭代器的验证实验

```cpp name=iterator_experiment.cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> L = {10, 20, 30, 40, 50};

    // 验证：list 的迭代器不能 +、- 运算
    list<int>::iterator it = L.begin();

    // it = it + 1;     // ❌ 编译错误！
    // it = it + 3;     // ❌ 编译错误！
    // it += 2;         // ❌ 编译错误！

    // 只能 ++ 和 --
    it++;               // ✅ 可以
    ++it;               // ✅ 可以
    it--;               // ✅ 可以
    --it;               // ✅ 可以

    cout << "*it = " << *it << endl;

    return 0;
}
```

---

## 7. list 容器 - 反转和排序

### 7.1 反转 - reverse()

`reverse()` 是 `list` 的**成员函数**，将链表中所有元素的顺序反转。

```cpp name=reverse_example.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    list<int> L = {10, 20, 30, 40, 50};

    cout << "反转前: ";
    printList(L);  // 10 20 30 40 50

    L.reverse();

    cout << "反转后: ";
    printList(L);  // 50 40 30 20 10

    return 0;
}
```

### 7.2 排序 - sort()

#### 7.2.1 为什么 list 不能用 std::sort？

这是一个**非常重要**的知识点。

`<algorithm>` 中的全局 `std::sort()` 要求**随机访问迭代器**，而 `list` 只有**双向迭代器**，所以**不能使用 `std::sort()`**！

```cpp name=cannot_use_std_sort.cpp
#include <algorithm>
#include <list>

int main() {
    list<int> L = {30, 10, 50, 20, 40};

    // ❌ 编译错误！std::sort 需要随机访问迭代器
    // sort(L.begin(), L.end());

    // ✅ 正确方式：使用 list 自带的 sort 成员函数
    L.sort();

    return 0;
}
```

#### 7.2.2 list::sort() 默认排序（升序）

```cpp name=sort_ascending.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

int main() {
    list<int> L = {30, 10, 50, 20, 40};

    cout << "排序前: ";
    printList(L);  // 30 10 50 20 40

    L.sort();  // 默认升序排列

    cout << "排序后: ";
    printList(L);  // 10 20 30 40 50

    return 0;
}
```

#### 7.2.3 自定义排序规则（降序）

`list::sort()` 可以接受一个**二元谓词（比较函数）**，用于自定义排序规则。

```cpp name=sort_descending.cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& L) {
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}

// 方法1：普通函数作为比较器
bool myCompare(int v1, int v2) {
    // 降序：前一个数 > 后一个数
    return v1 > v2;
}

int main() {
    list<int> L = {30, 10, 50, 20, 40};

    // 使用自定义比较函数 —— 降序
    L.sort(myCompare);
    cout << "降序排列: ";
    printList(L);  // 50 40 30 20 10

    // 方法2：使用 lambda 表达式（C++11）
    L.sort([](int a, int b) {
        return a < b;  // 升序
    });
    cout << "升序排列: ";
    printList(L);  // 10 20 30 40 50

    // 方法3：使用标准库仿函数
    L.sort(greater<int>());  // 降序
    cout << "降序排列: ";
    printList(L);  // 50 40 30 20 10

    return 0;
}
```

### 7.3 sort 的规则解读

理解比较函数的规则至关重要：

```cpp name=sort_rule_explanation.cpp
// 比较函数的含义：
// 如果返回 true，表示第一个参数应该排在第二个参数前面

bool ascending(int a, int b) {
    return a < b;
    // a < b 为 true → a 排在 b 前面 → 小的在前 → 升序
}

bool descending(int a, int b) {
    return a > b;
    // a > b 为 true → a 排在 b 前面 → 大的在前 → 降序
}
```

### 7.4 反向迭代器遍历

除了 `reverse()`，我们还可以使用**反向迭代器**来倒序遍历 list，而不改变原数据：

```cpp name=reverse_iterator.cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> L = {10, 20, 30, 40, 50};

    // 正向遍历
    cout << "正向: ";
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;  // 10 20 30 40 50

    // 反向遍历（使用反向迭代器）
    cout << "反向: ";
    for (auto rit = L.rbegin(); rit != L.rend(); rit++) {
        cout << *rit << " ";
    }
    cout << endl;  // 50 40 30 20 10

    // 原 list 并没有被修改
    cout << "原 list 的 front: " << L.front() << endl;  // 10

    return 0;
}
```

---

## 8. list 容器 - 排序案例（综合实战）

### 8.1 案例需求

**题目**：某公司招聘，需要对候选人按照一定的规则进行排序。

候选人有以下属性：
- **姓名（string）**
- **年龄（int）**
- **身高（int）**

**排序规则**：
1. 按**年龄**升序排列
2. 年龄相同时，按**身高**降序排列

### 8.2 定义 Person 类

```cpp name=sort_case_person.cpp
#include <iostream>
#include <list>
#include <string>
using namespace std;

// 定义 Person 类
class Person {
public:
    string m_Name;   // 姓名
    int m_Age;       // 年龄
    int m_Height;    // 身高

    Person(string name, int age, int height)
        : m_Name(name), m_Age(age), m_Height(height) {}
};
```

### 8.3 编写打印函数

```cpp name=sort_case_print.cpp
// 打印 list<Person>
void printPersonList(const list<Person>& L) {
    cout << "--------------------------------------" << endl;
    cout << "姓名\t\t年龄\t身高" << endl;
    cout << "--------------------------------------" << endl;
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << it->m_Name << "\t\t"
             << it->m_Age << "\t"
             << it->m_Height << endl;
    }
    cout << "--------------------------------------" << endl;
}
```

### 8.4 编写自定义排序规则

```cpp name=sort_case_comparator.cpp
// 自定义排序规则
// 规则：年龄升序；年龄相同时，身高降序
bool comparePerson(const Person& p1, const Person& p2) {
    if (p1.m_Age != p2.m_Age) {
        // 年龄不同：按年龄升序
        return p1.m_Age < p2.m_Age;
    } else {
        // 年龄相同：按身高降序
        return p1.m_Height > p2.m_Height;
    }
}
```

### 8.5 完整代码

```cpp name=sort_case_complete.cpp
#include <iostream>
#include <list>
#include <string>
using namespace std;

// ========================
// Person 类定义
// ========================
class Person {
public:
    string m_Name;
    int m_Age;
    int m_Height;

    Person(string name, int age, int height)
        : m_Name(name), m_Age(age), m_Height(height) {}
};

// ========================
// 打印函数
// ========================
void printPersonList(const list<Person>& L) {
    cout << "-------------------------------------------" << endl;
    cout << "姓名\t\t年龄\t身高(cm)" << endl;
    cout << "-------------------------------------------" << endl;
    for (auto it = L.begin(); it != L.end(); it++) {
        cout << it->m_Name << "\t\t"
             << it->m_Age << "\t"
             << it->m_Height << endl;
    }
    cout << "-------------------------------------------" << endl;
}

// ========================
// 自定义排序规则
// 年龄升序；年龄相同时，身高降序
// ========================
bool comparePerson(const Person& p1, const Person& p2) {
    if (p1.m_Age != p2.m_Age) {
        return p1.m_Age < p2.m_Age;     // 年龄升序
    } else {
        return p1.m_Height > p2.m_Height; // 身高降序
    }
}

// ========================
// 主函数
// ========================
int main() {
    // 创建候选人列表
    list<Person> personList;

    // 添加候选人数据
    personList.emplace_back("刘备",   35, 175);
    personList.emplace_back("曹操",   45, 180);
    personList.emplace_back("孙权",   30, 170);
    personList.emplace_back("赵云",   25, 190);
    personList.emplace_back("张飞",   35, 185);
    personList.emplace_back("关羽",   35, 200);
    personList.emplace_back("貂蝉",   20, 165);
    personList.emplace_back("吕布",   30, 195);
    personList.emplace_back("周瑜",   25, 178);
    personList.emplace_back("诸葛亮", 25, 182);

    // 排序前
    cout << "\n========== 排序前 ==========" << endl;
    printPersonList(personList);

    // 执行排序
    personList.sort(comparePerson);

    // 排序后
    cout << "\n========== 排序后 ==========" << endl;
    cout << "（规则：年龄升序，年龄相同按身高降序）\n" << endl;
    printPersonList(personList);

    return 0;
}
```

### 8.6 运行结果

```
========== 排序前 ==========
-------------------------------------------
姓名		年龄	身高(cm)
-------------------------------------------
刘备		35	175
曹操		45	180
孙权		30	170
赵云		25	190
张飞		35	185
关羽		35	200
貂蝉		20	165
吕布		30	195
周瑜		25	178
诸葛亮		25	182
-------------------------------------------

========== 排序后 ==========
（规则：年龄升序，年龄相同按身高降序）

-------------------------------------------
姓名		年龄	身高(cm)
-------------------------------------------
貂蝉		20	165
赵云		25	190
诸葛亮		25	182
周瑜		25	178
吕布		30	195
孙权		30	170
关羽		35	200
张飞		35	185
刘备		35	175
曹操		45	180
-------------------------------------------
```

分析排序结果：

| 分组  | 年龄 | 组内排列（按身高降序）              |
| ----- | ---- | ----------------------------------- |
| 第1组 | 20岁 | 貂蝉(165)                           |
| 第2组 | 25岁 | 赵云(190) → 诸葛亮(182) → 周瑜(178) |
| 第3组 | 30岁 | 吕布(195) → 孙权(170)               |
| 第4组 | 35岁 | 关羽(200) → 张飞(185) → 刘备(175)   |
| 第5组 | 45岁 | 曹操(180)                           |

✅ 完全符合排序规则！

### 8.7 使用 Lambda 表达式简化（C++11）

如果你更喜欢简洁的写法，可以用 lambda 表达式替代独立的比较函数：

```cpp name=sort_case_lambda.cpp
int main() {
    list<Person> personList;
    // ... 添加数据 ...

    // 使用 lambda 表达式排序
    personList.sort([](const Person& p1, const Person& p2) -> bool {
        if (p1.m_Age != p2.m_Age) {
            return p1.m_Age < p2.m_Age;
        }
        return p1.m_Height > p2.m_Height;
    });

    printPersonList(personList);
    return 0;
}
```

### 8.8 扩展：三级排序

如果需求变为：年龄升序 → 身高降序 → 姓名按字典序，只需扩展比较函数：

```cpp name=three_level_sort.cpp
bool comparePerson3(const Person& p1, const Person& p2) {
    if (p1.m_Age != p2.m_Age) {
        return p1.m_Age < p2.m_Age;         // 第一优先级：年龄升序
    }
    if (p1.m_Height != p2.m_Height) {
        return p1.m_Height > p2.m_Height;    // 第二优先级：身高降序
    }
    return p1.m_Name < p2.m_Name;            // 第三优先级：姓名字典序升序
}
```

---

## 附录：list 常用成员函数速查表

| 分类     | 函数                | 说明         | 时间复杂度     |
| -------- | ------------------- | ------------ | -------------- |
| **构造** | `list()`            | 默认构造     | O(1)           |
|          | `list(n, val)`      | 填充构造     | O(n)           |
|          | `list(beg, end)`    | 范围构造     | O(n)           |
|          | `list(other)`       | 拷贝构造     | O(n)           |
| **赋值** | `operator=`         | 赋值运算符   | O(n)           |
|          | `assign(n, val)`    | 填充赋值     | O(n)           |
|          | `assign(beg, end)`  | 范围赋值     | O(n)           |
|          | `swap(other)`       | 交换内容     | **O(1)**       |
| **大小** | `size()`            | 元素个数     | O(1)           |
|          | `empty()`           | 是否为空     | O(1)           |
|          | `resize(n)`         | 调整大小     | O(n)           |
| **插入** | `push_back(elem)`   | 尾部插入     | O(1)           |
|          | `push_front(elem)`  | 头部插入     | O(1)           |
|          | `insert(pos, elem)` | 指定位置插入 | O(1)（已定位） |
| **删除** | `pop_back()`        | 尾部删除     | O(1)           |
|          | `pop_front()`       | 头部删除     | O(1)           |
|          | `erase(pos)`        | 删除指定位置 | O(1)（已定位） |
|          | `clear()`           | 清空容器     | O(n)           |
| **其他** | `reverse()`         | 反转         | O(n)           |
|          | `sort()`            | 排序         | O(n log n)     |