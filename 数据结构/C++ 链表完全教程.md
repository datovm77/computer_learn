# C++ 链表完全教程

> 从零基础到手写 `std::list`，为学习 STL `list` 容器打下坚实基础。

---

## 第一阶段：链表基础

---

### 1. 链表概述

#### 1.1 什么是链表？

**链表（Linked List）** 是一种**线性数据结构**，它通过**指针**将一组零散的内存块（称为**节点**）串联起来，形成一个逻辑上连续的序列。

与数组不同，链表的每个元素**不要求在内存中连续存放**。每个节点除了存储数据本身，还要额外存储一个（或多个）指针，指向下一个（或上一个）节点。

形象地说：

- **数组**像一排连续的储物柜，每个柜子紧挨着，通过编号直接找到。
- **链表**像一条寻宝链条，每个宝箱里有一条线索，告诉你下一个宝箱在哪里。

```
数组（连续内存）:
┌───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │
└───┴───┴───┴───┴───┘
  ↑ 起始地址，后续元素地址 = 起始 + index × sizeof(元素)

链表（离散内存）:
┌───┬───┐    ┌───┬───┐    ┌───┬────────┐
│ 1 │  ─┼───>│ 2 │  ─┼───>│ 3 │ nullptr│
└───┴───┘    └───┴───┘    └───┴────────┘
 data next    data next    data  next
```

#### 1.2 链表与数组的全方位对比

| 特性          | 数组 array / vector                   | 链表 linked list             |
| ------------- | ------------------------------------- | ---------------------------- |
| 内存布局      | 连续                                  | 离散                         |
| 随机访问      | ✅ O(1)，支持下标 []                   | ❌ O(n)，必须从头遍历         |
| 头部插入/删除 | ❌ O(n)，需要移动后续元素              | ✅ O(1)，修改指针即可         |
| 中间插入/删除 | ❌ O(n)                                | ✅ O(1)（已知位置时）         |
| 尾部插入      | ✅ 均摊 O(1)（vector）                 | O(1)（维护尾指针时）         |
| 查找元素      | O(n)（无序） / O(log n)（有序，二分） | O(n)，只能顺序查找           |
| 内存开销      | 仅存数据，紧凑                        | 每个节点额外存储指针，开销大 |
| 缓存友好性    | ✅ 极好，连续内存利于 CPU 缓存预取     | ❌ 差，节点散布在内存各处     |
| 内存分配      | 一次性分配大块内存                    | 每个节点单独分配             |
| 扩容方式      | 需要重新分配+拷贝（vector 翻倍策略）  | 天然支持动态增长             |

#### 1.3 链表的优缺点分析

**优点：**

- **动态大小**：无需预先确定大小，也不需要像 `vector` 那样在扩容时进行昂贵的内存重新分配和拷贝。
- **插入/删除高效**：在已知位置的情况下，插入和删除操作只需修改指针，时间复杂度 O(1)。
- **容量管理更弹性**：不需要像 `vector` 一样预留 capacity；但每个节点本身和分配器元数据仍有额外开销。

**缺点：**

- **不支持随机访问**：访问第 i 个元素必须从头遍历，时间复杂度 O(n)。
- **额外的内存开销**：每个节点都需要额外存储指针（单链表 1 个指针，双向链表 2 个指针）。
- **缓存不友好**：节点在堆上离散分配，容易导致 CPU 缓存缺失（cache miss），实际性能可能不如理论分析。

#### 1.4 链表的应用场景

- **频繁在中间/头部做插入和删除**的场景（如文本编辑器的行缓冲区、LRU 缓存淘汰策略）。
- **不确定数据规模**、但不需要随机访问的场景。
- **操作系统内核**中大量使用链表管理进程、文件描述符、内存页等。
- **STL 容器** `std::list` 和 `std::forward_list` 的底层实现。
- **哈希表的链地址法**（每个桶挂一条链表解决冲突）。

---

### 2. 节点（Node）的定义

#### 2.1 数据域与指针域

一个链表节点由两部分组成：

```
┌──────────────────────┐
│   数据域 (data)       │  ← 存储实际的数据
├──────────────────────┤
│   指针域 (next)       │  ← 指向下一个节点的指针
└──────────────────────┘
```

- **数据域（data field）**：存储节点承载的数据，可以是 `int`、`string` 或任意类型。
- **指针域（pointer field）**：存储下一个节点的地址。当它为 `nullptr` 时，表示链表的结束。

#### 2.2 用 struct 定义节点

在 C++ 中，使用 `struct` 来定义链表节点是最自然的做法：

```
// 单链表节点定义
struct ListNode {
    int data;           // 数据域
    ListNode* next;     // 指针域：指向下一个节点

    // 构造函数：方便创建节点时直接赋值
    ListNode(int val) : data(val), next(nullptr) {}
};
```

**为什么要写构造函数？**

如果不写构造函数，每次创建节点都需要手动初始化：

```
// 没有构造函数，繁琐且容易忘记初始化 next
ListNode* node = new ListNode;
node->data = 10;
node->next = nullptr;  // 忘记这一步就是野指针！

// 有构造函数，一行搞定
ListNode* node = new ListNode(10);  // data=10, next=nullptr
```

> **⚠️ 关键点**：未初始化指针属于**野指针（wild pointer）**；**悬垂指针（dangling pointer）** 是“曾经有效但所指对象已被释放”的指针。两者一旦解引用都会触发未定义行为。

#### 2.3 节点的内存分配（new / delete）

链表节点在**堆（heap）**上动态分配，使用 `new` 分配、`delete` 释放：

```
// 分配一个新节点
ListNode* p = new ListNode(42);  
// new 做了两件事：
//   1. 在堆上分配 sizeof(ListNode) 字节的内存
//   2. 调用构造函数 ListNode(42)，初始化 data=42, next=nullptr

std::cout << p->data << std::endl;  // 42
std::cout << p->next << std::endl;  // 0 (nullptr)

// 使用完毕后，必须释放内存
delete p;
p = nullptr;  // 好习惯：释放后将指针置空，避免悬垂指针
```

**内存泄漏的危险：**

```
void bad_example() {
    ListNode* p = new ListNode(1);
    ListNode* q = new ListNode(2);
    p->next = q;
    
    // 函数结束时，p 和 q 这两个局部变量被销毁
    // 但堆上的两个 ListNode 对象并没有被 delete！
    // 这就是内存泄漏！
}

void good_example() {
    ListNode* p = new ListNode(1);
    ListNode* q = new ListNode(2);
    p->next = q;
    
    // 正确释放：从头开始逐个删除
    ListNode* curr = p;
    while (curr != nullptr) {
        ListNode* temp = curr->next;  // 先保存下一个节点
        delete curr;                  // 再删除当前节点
        curr = temp;                  // 移动到下一个
    }
}
```

> **📌 规则**：在这类“手写裸指针链表”里，每一个 `new` 最终都要被 `delete` 回收，否则就是内存泄漏。

---

## 第二阶段：单链表（Singly Linked List）

---

### 3. 单链表的创建与遍历

#### 3.1 头指针（head pointer）的概念

**头指针**是一个指向链表第一个节点的指针变量。通过头指针，我们可以访问整条链表。

```
ListNode* head = nullptr;  // 空链表：头指针为 nullptr
```

```
head ──> nullptr     （空链表）

head ──> [10|──>] ──> [20|──>] ──> [30|nullptr]
          第1个节点    第2个节点     第3个节点（尾节点）
```

> **注意**：头指针本身**不是**节点，它只是一个存储地址的指针变量。

#### 3.2 头插法建表

**头插法**：每次将新节点插入到链表**头部**。特点是**后输入的数据在前面**（类似栈的 LIFO 行为）。

```
// 头插法建立链表
// 输入：1 -> 2 -> 3
// 结果：3 -> 2 -> 1
ListNode* createListByHead(const std::vector<int>& arr) {
    ListNode* head = nullptr;

    for (int val : arr) {
        ListNode* newNode = new ListNode(val);
        newNode->next = head;  // 新节点的 next 指向原来的头
        head = newNode;        // 更新头指针，指向新节点
    }

    return head;
}
```

**逐步图解（输入 1, 2, 3）：**

```
初始状态：  head -> nullptr

插入 1：    head -> [1|nullptr]

插入 2：    head -> [2|──>] -> [1|nullptr]
            新节点插在最前面

插入 3：    head -> [3|──>] -> [2|──>] -> [1|nullptr]
            最后插入的在最前面
```

#### 3.3 尾插法建表

**尾插法**：每次将新节点插入到链表**尾部**。特点是**先输入的数据在前面**，保持原始顺序。

```
// 尾插法建立链表
// 输入：1 -> 2 -> 3
// 结果：1 -> 2 -> 3
ListNode* createListByTail(const std::vector<int>& arr) {
    ListNode* head = nullptr;
    ListNode* tail = nullptr;  // 尾指针，始终指向最后一个节点

    for (int val : arr) {
        ListNode* newNode = new ListNode(val);

        if (head == nullptr) {
            // 链表为空，新节点既是头又是尾
            head = newNode;
            tail = newNode;
        } else {
            // 将新节点挂到尾部
            tail->next = newNode;
            tail = newNode;  // 更新尾指针
        }
    }

    return head;
}
```

**逐步图解（输入 1, 2, 3）：**

```
初始状态：  head -> nullptr, tail -> nullptr

插入 1：    head -> [1|nullptr]
            tail ──────↑

插入 2：    head -> [1|──>] -> [2|nullptr]
            tail ────────────────↑

插入 3：    head -> [1|──>] -> [2|──>] -> [3|nullptr]
            tail ──────────────────────────↑
```

#### 3.4 遍历与打印链表

遍历链表的核心思想：从头指针开始，沿着 `next` 指针逐个访问节点，直到遇到 `nullptr`。

```
void printList(ListNode* head) {
    ListNode* curr = head;  // 用一个临时指针遍历，不要修改 head！
    while (curr != nullptr) {
        std::cout << curr->data;
        if (curr->next != nullptr) {
            std::cout << " -> ";
        }
        curr = curr->next;  // 移动到下一个节点
    }
    std::cout << std::endl;
}

// 使用示例
ListNode* list = createListByTail({1, 2, 3, 4, 5});
printList(list);  // 输出：1 -> 2 -> 3 -> 4 -> 5
```

> **⚠️ 初学者常见错误**：用 `head` 直接遍历，遍历完后 `head` 变成了 `nullptr`，链表"丢了"！一定要用一个**临时指针** `curr`。

---

### 4. 单链表的基本操作

#### 4.1 按位置查找

查找链表中第 `index` 个节点（0-indexed）。

```
// 返回第 index 个节点的指针（从 0 开始计数）
// 找不到返回 nullptr
ListNode* getNodeAt(ListNode* head, int index) {
    if (index < 0) return nullptr;

    ListNode* curr = head;
    for (int i = 0; i < index && curr != nullptr; ++i) {
        curr = curr->next;
    }
    return curr;  // 如果 index 越界，curr 已经是 nullptr
}

// 使用示例
// 链表：10 -> 20 -> 30 -> 40
ListNode* node = getNodeAt(head, 2);  // 返回指向 30 的指针
if (node) {
    std::cout << node->data << std::endl;  // 30
}
```

#### 4.2 按值查找

查找第一个值为 `target` 的节点。

```
// 按值查找，返回第一个匹配的节点指针
ListNode* findNode(ListNode* head, int target) {
    ListNode* curr = head;
    while (curr != nullptr) {
        if (curr->data == target) {
            return curr;  // 找到了
        }
        curr = curr->next;
    }
    return nullptr;  // 没找到
}

// 也可以返回位置索引
int findIndex(ListNode* head, int target) {
    ListNode* curr = head;
    int index = 0;
    while (curr != nullptr) {
        if (curr->data == target) {
            return index;
        }
        curr = curr->next;
        ++index;
    }
    return -1;  // 没找到
}
```

#### 4.3 指定位置插入节点

在链表的第 `index` 个位置**之前**插入新节点：

```
// 在第 index 个位置插入新节点（0-indexed）
// 返回新的头指针（因为头部插入时头指针会改变）
ListNode* insertAt(ListNode* head, int index, int val) {
    ListNode* newNode = new ListNode(val);

    // 特殊情况：在头部插入（index == 0）
    if (index == 0) {
        newNode->next = head;
        return newNode;  // 新节点成为新的头
    }

    // 找到第 index-1 个节点（即插入位置的前驱节点）
    ListNode* prev = getNodeAt(head, index - 1);
    if (prev == nullptr) {
        // index 越界，插入失败
        delete newNode;
        std::cout << "插入位置无效！" << std::endl;
        return head;
    }

    // 执行插入：
    //   prev -> [某节点]
    //   变为：prev -> [newNode] -> [某节点]
    newNode->next = prev->next;
    prev->next = newNode;

    return head;
}
```

**图解插入过程（在位置 2 插入 25）：**

```
原链表：   head -> [10] -> [20] -> [30] -> [40] -> nullptr
                            ↑ prev（index-1 = 1 号节点）

步骤1：newNode->next = prev->next
           [25] ──> [30]

步骤2：prev->next = newNode
           [20] ──> [25]

结果：     head -> [10] -> [20] -> [25] -> [30] -> [40] -> nullptr
```

> **⚠️ 关键**：步骤 1 和步骤 2 的顺序**不能颠倒**！如果先执行 `prev->next = newNode`，那原来 `prev->next` 指向的节点就丢失了。

#### 4.4 删除指定节点

删除链表中第 `index` 个节点：

```
// 删除第 index 个节点（0-indexed）
// 返回新的头指针
ListNode* deleteAt(ListNode* head, int index) {
    if (head == nullptr) return nullptr;

    // 特殊情况：删除头节点
    if (index == 0) {
        ListNode* newHead = head->next;
        delete head;
        return newHead;
    }

    // 找到第 index-1 个节点（前驱节点）
    ListNode* prev = getNodeAt(head, index - 1);
    if (prev == nullptr || prev->next == nullptr) {
        std::cout << "删除位置无效！" << std::endl;
        return head;
    }

    // 执行删除
    ListNode* toDelete = prev->next;
    prev->next = toDelete->next;  // 跳过被删除的节点
    delete toDelete;              // 释放内存

    return head;
}
```

**图解删除过程（删除位置 2 的节点）：**

```
原链表：   head -> [10] -> [20] -> [30] -> [40] -> nullptr
                            ↑ prev    ↑ toDelete

步骤1：prev->next = toDelete->next
           [20] ──────────────────> [40]

步骤2：delete toDelete
           [30] 被释放

结果：     head -> [10] -> [20] -> [40] -> nullptr
```

#### 4.5 按值删除节点

删除链表中第一个值为 `target` 的节点：

```
ListNode* deleteByValue(ListNode* head, int target) {
    if (head == nullptr) return nullptr;

    // 特殊情况：头节点就是目标
    if (head->data == target) {
        ListNode* newHead = head->next;
        delete head;
        return newHead;
    }

    // 查找目标节点的前驱
    ListNode* prev = head;
    while (prev->next != nullptr && prev->next->data != target) {
        prev = prev->next;
    }

    if (prev->next == nullptr) {
        std::cout << "未找到值为 " << target << " 的节点" << std::endl;
        return head;
    }

    ListNode* toDelete = prev->next;
    prev->next = toDelete->next;
    delete toDelete;

    return head;
}
```

#### 4.6 求链表长度

```
int getLength(ListNode* head) {
    int count = 0;
    ListNode* curr = head;
    while (curr != nullptr) {
        ++count;
        curr = curr->next;
    }
    return count;
}
```

---

### 5. 带头节点 vs 不带头节点

#### 5.1 什么是哨兵节点（Dummy Head）？

**哨兵节点**（也叫虚拟头节点、Dummy Head）是一个**不存储有效数据**的额外节点，放在链表的最前面。它的 `next` 指向真正的第一个数据节点。

```
不带头节点：
head -> [10] -> [20] -> [30] -> nullptr
         ↑ 第一个数据节点

带头节点（哨兵节点）：
head -> [dummy] -> [10] -> [20] -> [30] -> nullptr
         ↑ 哨兵节点    ↑ 第一个数据节点
         不存储有效数据
```

#### 5.2 为什么需要哨兵节点？

**核心动机：消除对"头节点"的特殊处理。**

回顾前面的插入和删除操作，我们每次都需要单独处理 `index == 0` 的情况（即操作头节点时），因为头节点没有前驱节点。

**不带头节点的插入——需要特殊处理头部：**

```
ListNode* insertAt(ListNode* head, int index, int val) {
    ListNode* newNode = new ListNode(val);
    
    if (index == 0) {           // ← 特殊处理！
        newNode->next = head;
        return newNode;         // ← 头指针改变了！
    }
    
    ListNode* prev = getNodeAt(head, index - 1);
    newNode->next = prev->next;
    prev->next = newNode;
    return head;
}
```

**带头节点的插入——所有位置统一处理：**

```
// dummyHead 是哨兵节点，永远不会被删除
// 返回是否插入成功：index 必须在 [0, length] 内
bool insertAt(ListNode* dummyHead, int index, int val) {
    if (index < 0) return false;

    // 统一逻辑：找到第 index 个位置的前驱节点
    // 当 index == 0 时，前驱就是 dummyHead 本身
    ListNode* prev = dummyHead;
    for (int i = 0; i < index; ++i) {
        if (prev->next == nullptr) return false;  // 越界
        prev = prev->next;
    }

    ListNode* newNode = new ListNode(val);
    newNode->next = prev->next;
    prev->next = newNode;
    return true;
    // 注意：不需要返回新的头指针，因为 dummyHead 始终是入口
}
```

#### 5.3 两种方式的对比

| 特性          | 不带头节点                   | 带头节点（哨兵）           |
| ------------- | ---------------------------- | -------------------------- |
| 插入/删除头部 | 需要特殊处理，头指针可能改变 | 统一逻辑，无需特殊处理     |
| 函数返回值    | 通常需要返回新的头指针       | 不需要，dummyHead 固定     |
| 空链表表示    | head == nullptr              | dummyHead->next == nullptr |
| 代码简洁度    | 略复杂（多个 if 分支）       | 更简洁、更统一             |
| 额外内存      | 无                           | 多一个节点的空间           |
| STL list 采用 | —                            | ✅ std::list 使用哨兵节点   |

> **📌 建议**：在实际项目和面试中，带哨兵节点的写法通常更受青睐，因为代码更简洁、更不容易出 bug。

**完整的带哨兵节点示例：**

```
// 创建带哨兵的链表
ListNode* createWithDummy(const std::vector<int>& arr) {
    ListNode* dummy = new ListNode(0);  // 哨兵节点，data 值无意义
    ListNode* tail = dummy;

    for (int val : arr) {
        tail->next = new ListNode(val);
        tail = tail->next;
    }

    return dummy;  // 返回哨兵节点
    // 第一个数据节点是 dummy->next
}

// 遍历时跳过哨兵
void printList(ListNode* dummy) {
    ListNode* curr = dummy->next;  // 从第一个数据节点开始
    while (curr != nullptr) {
        std::cout << curr->data << " ";
        curr = curr->next;
    }
    std::cout << std::endl;
}

// 统一的删除操作
// 返回是否删除成功：index 必须在 [0, length-1] 内
bool deleteAt(ListNode* dummy, int index) {
    if (index < 0) return false;

    ListNode* prev = dummy;
    for (int i = 0; i < index; ++i) {
        if (prev->next == nullptr) return false;  // 越界
        prev = prev->next;
    }
    if (prev->next == nullptr) return false;      // index == length，越界

    ListNode* toDelete = prev->next;
    prev->next = toDelete->next;
    delete toDelete;
    return true;
}
```

---

### 6. 单链表的进阶操作

#### 6.1 链表反转

**将链表 1->2->3->4->5 反转为 5->4->3->2->1。**

这是面试高频题，有两种经典方法：

**方法一：迭代法（推荐）**

核心思想：遍历链表，将每个节点的 `next` 指针反转，指向前一个节点。

```
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;

    while (curr != nullptr) {
        ListNode* nextTemp = curr->next;  // 先保存下一个节点
        curr->next = prev;               // 反转指针方向
        prev = curr;                     // prev 向前移动
        curr = nextTemp;                 // curr 向前移动
    }

    return prev;  // prev 现在是新的头节点
}
```

**逐步图解：**

```
初始：  nullptr   1 -> 2 -> 3 -> nullptr
         prev   curr

第1步：  nullptr <- 1    2 -> 3 -> nullptr
                  prev  curr

第2步：  nullptr <- 1 <- 2    3 -> nullptr
                        prev  curr

第3步：  nullptr <- 1 <- 2 <- 3
                             prev  curr(nullptr)

结束：返回 prev，即 3（新的头节点）
结果：  3 -> 2 -> 1 -> nullptr
```

**方法二：递归法**

```
ListNode* reverseList(ListNode* head) {
    // 基线条件：空链表或只有一个节点
    if (head == nullptr || head->next == nullptr) {
        return head;
    }

    // 递归反转 head->next 及之后的部分
    ListNode* newHead = reverseList(head->next);

    // 现在 head->next 是反转后子链表的尾节点
    // 让它指回 head
    head->next->next = head;
    head->next = nullptr;  // head 变成新的尾节点

    return newHead;
}
```

**递归展开图解（链表 1->2->3）：**

```
reverseList(1)
  ├── reverseList(2)
  │     ├── reverseList(3) → 返回 3（基线条件）
  │     │
  │     ├── head=2, head->next=3
  │     │   3->next = 2    →  3 -> 2
  │     │   2->next = nullptr
  │     └── 返回 newHead=3
  │
  ├── head=1, head->next=2
  │   2->next = 1    →  3 -> 2 -> 1
  │   1->next = nullptr
  └── 返回 newHead=3

最终结果：3 -> 2 -> 1 -> nullptr
```

#### 6.2 链表排序

**方法一：插入排序 —— O(n²)，简单直观**

```
ListNode* insertionSortList(ListNode* head) {
    if (head == nullptr || head->next == nullptr) return head;

    ListNode* dummy = new ListNode(0);  // 使用哨兵节点
    ListNode* curr = head;

    while (curr != nullptr) {
        ListNode* nextTemp = curr->next;  // 保存下一个待处理节点

        // 在已排序部分中找到 curr 应该插入的位置
        ListNode* prev = dummy;
        while (prev->next != nullptr && prev->next->data < curr->data) {
            prev = prev->next;
        }

        // 将 curr 插入到 prev 之后
        curr->next = prev->next;
        prev->next = curr;

        curr = nextTemp;  // 处理下一个节点
    }

    ListNode* sortedHead = dummy->next;
    delete dummy;
    return sortedHead;
}
```

**方法二：归并排序 —— O(n log n)，链表排序的最优解**

归并排序非常适合链表，因为“合并”步骤只需要改指针，不需要像数组那样搬移元素。若采用递归写法，额外空间主要来自调用栈（O(log n)）。

```
// 辅助函数：找到链表的中间节点（快慢指针法）
ListNode* getMiddle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head->next;  // 注意：fast 从 head->next 开始

    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }

    return slow;  // slow 指向中间节点（偏左）
}

// 辅助函数：合并两个有序链表
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;

    while (l1 != nullptr && l2 != nullptr) {
        if (l1->data <= l2->data) {
            tail->next = l1;
            l1 = l1->next;
        } else {
            tail->next = l2;
            l2 = l2->next;
        }
        tail = tail->next;
    }

    tail->next = (l1 != nullptr) ? l1 : l2;
    return dummy.next;
}

// 归并排序主函数
ListNode* mergeSort(ListNode* head) {
    // 基线条件：空链表或只有一个节点
    if (head == nullptr || head->next == nullptr) {
        return head;
    }

    // 1. 找到中间节点，断开链表
    ListNode* mid = getMiddle(head);
    ListNode* rightHalf = mid->next;
    mid->next = nullptr;  // 断开！

    // 2. 递归排序左右两半
    ListNode* left = mergeSort(head);
    ListNode* right = mergeSort(rightHalf);

    // 3. 合并两个有序链表
    return mergeTwoLists(left, right);
}
```

> **📌 知识点**：`std::list::sort()` 在主流实现中通常采用归并排序（稳定，且适配链表的指针拼接）。标准主要约束的是复杂度和稳定性。

#### 6.3 检测链表是否有环（快慢指针 / Floyd 判圈算法）

**什么是环？**

正常链表的尾节点 `next` 指向 `nullptr`。如果某个节点的 `next` 指向了前面的某个节点，就形成了一个环，遍历永远不会结束。

```
正常链表：  1 -> 2 -> 3 -> 4 -> nullptr

有环链表：  1 -> 2 -> 3 -> 4
                 ↑              |
                 └──────────────┘
                 4 的 next 指向 2，形成环
```

**快慢指针法：**

- **慢指针（slow）**：每次走 1 步
- **快指针（fast）**：每次走 2 步
- 如果有环，快指针最终会追上慢指针（就像操场跑步，跑得快的人会"套圈"）
- 如果无环，快指针会先到达 `nullptr`

```
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;

    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;           // 慢指针走 1 步
        fast = fast->next->next;     // 快指针走 2 步

        if (slow == fast) {
            return true;  // 相遇了，说明有环
        }
    }

    return false;  // fast 到达 nullptr，说明无环
}
```

**进阶：找到环的入口节点**

```
ListNode* detectCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;

    // 第一阶段：判断是否有环
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;

        if (slow == fast) {
            // 第二阶段：找环的入口
            // 数学证明：从 head 和相遇点同时出发，每次走 1 步，
            // 再次相遇的地方就是环的入口
            ListNode* p1 = head;
            ListNode* p2 = slow;
            while (p1 != p2) {
                p1 = p1->next;
                p2 = p2->next;
            }
            return p1;  // 环的入口
        }
    }

    return nullptr;  // 无环
}
```

**数学证明要点：**
设从 head 到环入口距离为 `a`，环入口到相遇点距离为 `b`，环的长度为 `c`。

- 慢指针走了 `a + b` 步
- 快指针走了 `a + b + n*c` 步（在环里多转了 n 圈）
- 快指针速度是慢指针的 2 倍：`2(a+b) = a + b + n*c` → `a + b = n*c` → `a = n*c - b = (n-1)*c + (c-b)`
- 所以从 head 走 `a` 步 = 从相遇点走 `(n-1)` 整圈 + `c-b` 步，两者恰好在环入口相遇。

#### 6.4 合并两个有序链表

```
// 将两个升序链表合并为一个新的升序链表
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);       // 栈上创建哨兵节点
    ListNode* tail = &dummy; // tail 指向已合并部分的末尾

    while (l1 != nullptr && l2 != nullptr) {
        if (l1->data <= l2->data) {
            tail->next = l1;
            l1 = l1->next;
        } else {
            tail->next = l2;
            l2 = l2->next;
        }
        tail = tail->next;
    }

    // 将剩余部分直接接上
    tail->next = (l1 != nullptr) ? l1 : l2;

    return dummy.next;
}

// 使用示例
// l1: 1 -> 3 -> 5
// l2: 2 -> 4 -> 6
// 合并后: 1 -> 2 -> 3 -> 4 -> 5 -> 6
```

> **📌 注意**：这里的 `dummy` 是在**栈上**创建的（`ListNode dummy(0)` 而不是 `new ListNode(0)`），函数结束时自动销毁，不需要手动 `delete`，也不会导致内存泄漏。这是一个实用的小技巧。

---

## 第三阶段：双向链表（Doubly Linked List）⭐ 重点

> **这是最核心的阶段！** `std::list` 的底层就是**带哨兵头节点的双向循环链表**。

---

### 7. 双向链表的结构与创建

#### 7.1 prev 与 next 双指针

单链表的节点只有一个 `next` 指针，只能**单方向遍历**。如果要访问前一个节点，必须从头重新遍历，非常低效。

**双向链表**为每个节点增加了一个 `prev` 指针，指向前一个节点，实现了**双方向遍历**。

```
单链表：   [10] ──> [20] ──> [30] ──> nullptr
           只能从左往右

双向链表：  nullptr <── [10] ⟺ [20] ⟺ [30] ──> nullptr
            可以双向移动
```

#### 7.2 节点定义

```
struct DListNode {
    int data;
    DListNode* prev;  // 指向前一个节点
    DListNode* next;  // 指向后一个节点

    DListNode(int val) : data(val), prev(nullptr), next(nullptr) {}
};
```

#### 7.3 双向链表的创建

```
// 尾插法创建双向链表
DListNode* createDoublyList(const std::vector<int>& arr) {
    if (arr.empty()) return nullptr;

    DListNode* head = new DListNode(arr[0]);
    DListNode* tail = head;

    for (size_t i = 1; i < arr.size(); ++i) {
        DListNode* newNode = new DListNode(arr[i]);
        tail->next = newNode;   // 当前尾节点的 next 指向新节点
        newNode->prev = tail;   // 新节点的 prev 指向当前尾节点
        tail = newNode;         // 更新尾指针
    }

    return head;
}
```

**内存结构图：**

```
创建 {10, 20, 30}：

   nullptr                                       nullptr
     ↑                                              ↑
  ┌──┴──┬────┬────┐    ┌────┬────┬────┐    ┌────┬────┬──┴──┐
  │prev │ 10 │next├───>│prev│ 20 │next├───>│prev│ 30 │next │
  │     │    │    │<───┤    │    │    │<───┤    │    │     │
  └─────┴────┴────┘    └────┴────┴────┘    └────┴────┴─────┘
    head                                          tail
```

---

### 8. 双向链表的基本操作

#### 8.1 双向遍历

双向链表可以从头到尾（正向）或从尾到头（反向）遍历。

```
// 正向遍历
void printForward(DListNode* head) {
    DListNode* curr = head;
    std::cout << "正向: ";
    while (curr != nullptr) {
        std::cout << curr->data << " ";
        curr = curr->next;
    }
    std::cout << std::endl;
}

// 反向遍历（需要先找到尾节点）
void printBackward(DListNode* head) {
    if (head == nullptr) return;

    // 先走到尾节点
    DListNode* tail = head;
    while (tail->next != nullptr) {
        tail = tail->next;
    }

    // 从尾节点往前遍历
    std::cout << "反向: ";
    DListNode* curr = tail;
    while (curr != nullptr) {
        std::cout << curr->data << " ";
        curr = curr->prev;
    }
    std::cout << std::endl;
}

// 示例输出：
// 正向: 10 20 30 40 50
// 反向: 50 40 30 20 10
```

#### 8.2 插入节点

双向链表的插入需要维护 **4 根指针**（比单链表多了 `prev` 的维护）。

**在指定节点 `pos` 之后插入新节点：**

```
// 在节点 pos 之后插入新节点
void insertAfter(DListNode* pos, int val) {
    if (pos == nullptr) return;

    DListNode* newNode = new DListNode(val);
    DListNode* nextNode = pos->next;  // 保存 pos 的后继

    // 建立 pos 和 newNode 之间的连接
    pos->next = newNode;
    newNode->prev = pos;

    // 建立 newNode 和 nextNode 之间的连接
    newNode->next = nextNode;
    if (nextNode != nullptr) {
        nextNode->prev = newNode;
    }
}
```

**图解（在节点 20 之后插入 25）：**

```
原链表：  [10] ⟺ [20] ⟺ [30]
                   ↑ pos

步骤1：保存 nextNode = pos->next（即 30）
步骤2：pos->next = newNode      →  [20] ──> [25]
步骤3：newNode->prev = pos      →  [20] <── [25]
步骤4：newNode->next = nextNode →  [25] ──> [30]
步骤5：nextNode->prev = newNode →  [25] <── [30]

结果：  [10] ⟺ [20] ⟺ [25] ⟺ [30]
```

**在指定节点 `pos` 之前插入新节点：**

```
// 在节点 pos 之前插入新节点
// 返回新的头指针（因为可能在头部之前插入）
DListNode* insertBefore(DListNode* head, DListNode* pos, int val) {
    if (pos == nullptr) return head;

    DListNode* newNode = new DListNode(val);
    DListNode* prevNode = pos->prev;

    // 建立 newNode 和 pos 之间的连接
    newNode->next = pos;
    pos->prev = newNode;

    // 建立 prevNode 和 newNode 之间的连接
    newNode->prev = prevNode;
    if (prevNode != nullptr) {
        prevNode->next = newNode;
    } else {
        // prevNode 为空意味着 pos 是头节点
        // newNode 成为新的头节点
        head = newNode;
    }

    return head;
}
```

#### 8.3 删除节点

```
// 删除指定节点 pos
// 返回新的头指针
DListNode* deleteNode(DListNode* head, DListNode* pos) {
    if (pos == nullptr) return head;

    DListNode* prevNode = pos->prev;
    DListNode* nextNode = pos->next;

    // 处理与前驱的连接
    if (prevNode != nullptr) {
        prevNode->next = nextNode;
    } else {
        // pos 是头节点，更新头指针
        head = nextNode;
    }

    // 处理与后继的连接
    if (nextNode != nullptr) {
        nextNode->prev = prevNode;
    }

    delete pos;
    return head;
}
```

> **双向链表删除的优势**：单链表删除需要找到前驱节点（O(n)），而双向链表可以直接通过 `pos->prev` 获取前驱（O(1)），所以双向链表的删除操作在已知节点指针的情况下是 O(1) 的。

#### 8.4 双向链表 vs 单链表操作对比

| 操作             | 单链表             | 双向链表                  |
| ---------------- | ------------------ | ------------------------- |
| 正向遍历         | ✅ O(n)             | ✅ O(n)                    |
| 反向遍历         | ❌ 不支持           | ✅ O(n)                    |
| 在给定节点后插入 | ✅ O(1)             | ✅ O(1)，但需要多维护 prev |
| 在给定节点前插入 | ❌ O(n)，需要找前驱 | ✅ O(1)，有 prev 指针      |
| 删除给定节点     | ❌ O(n)，需要找前驱 | ✅ O(1)，有 prev 指针      |
| 每个节点额外空间 | 1 个指针           | 2 个指针                  |

---

### 9. 双向循环链表

#### 9.1 循环链表的概念

普通链表的尾节点指向 `nullptr`，而**循环链表**的尾节点重新指回头节点，形成一个闭环。

```
普通双向链表：
nullptr <── [10] ⟺ [20] ⟺ [30] ──> nullptr

双向循环链表：
     ┌────────────────────────────────┐
     ↓                                |
    [10] ⟺ [20] ⟺ [30]              |
     ↑                  |             |
     └──────────────────┘             |
                                      ↓
   （尾节点的 next 指向头，头节点的 prev 指向尾）
```

#### 9.2 带哨兵头节点的双向循环链表

这就是 **`std::list` 的底层结构**！

- 有一个**不存储有效数据**的哨兵节点
- 哨兵的 `next` 指向第一个数据节点
- 哨兵的 `prev` 指向最后一个数据节点
- 最后一个数据节点的 `next` 指向哨兵
- 第一个数据节点的 `prev` 指向哨兵
- **空链表**时：哨兵的 `next` 和 `prev` 都指向自己

```
空链表：
    ┌───────┐
    ↓       |
  [sentinel]─┘   （next 和 prev 都指向自己）
    ↑       |
    └───────┘

含有 3 个元素的链表：
    ┌──────────────────────────────────────────┐
    ↓                                          |
  [sentinel] ⟺ [10] ⟺ [20] ⟺ [30]           |
    ↑                              |           |
    └──────────────────────────────┘           |
                                               ↓
  sentinel->next = 10（第一个数据节点）
  sentinel->prev = 30（最后一个数据节点）
  30->next = sentinel
  10->prev = sentinel
```

**实现代码：**

```
struct DListNode {
    int data;
    DListNode* prev;
    DListNode* next;

    DListNode(int val = 0) : data(val), prev(this), next(this) {}
    // 默认构造：next 和 prev 都指向自己（用于哨兵节点初始化）
};

class CircularDoublyList {
private:
    DListNode* sentinel;  // 哨兵节点
    int size_;

public:
    // 构造函数：创建空链表
    CircularDoublyList() : size_(0) {
        sentinel = new DListNode();
        // sentinel->next = sentinel, sentinel->prev = sentinel
        // 构造函数中已经默认设置好了
    }

    // 判断是否为空
    bool empty() const {
        return sentinel->next == sentinel;
    }

    // 获取元素个数
    int size() const { return size_; }

    // 在 pos 节点之前插入新节点（核心操作）
    void insertBefore(DListNode* pos, int val) {
        DListNode* newNode = new DListNode(val);
        DListNode* prevNode = pos->prev;

        // 建立 prevNode <-> newNode 的连接
        prevNode->next = newNode;
        newNode->prev = prevNode;

        // 建立 newNode <-> pos 的连接
        newNode->next = pos;
        pos->prev = newNode;

        ++size_;
    }

    // 头部插入
    void pushFront(int val) {
        insertBefore(sentinel->next, val);
    }

    // 尾部插入
    void pushBack(int val) {
        insertBefore(sentinel, val);
        // sentinel 的前面就是链表的尾部！
        // 在 sentinel 之前插入 = 在最后一个数据节点之后插入
    }

    // 删除指定节点
    void erase(DListNode* pos) {
        if (pos == sentinel) return;  // 不能删除哨兵节点！

        pos->prev->next = pos->next;
        pos->next->prev = pos->prev;
        delete pos;
        --size_;
    }

    // 头部删除
    void popFront() {
        if (!empty()) erase(sentinel->next);
    }

    // 尾部删除
    void popBack() {
        if (!empty()) erase(sentinel->prev);
    }

    // 正向遍历
    void printForward() const {
        DListNode* curr = sentinel->next;
        std::cout << "正向: ";
        while (curr != sentinel) {  // 回到 sentinel 就结束
            std::cout << curr->data << " ";
            curr = curr->next;
        }
        std::cout << "(size=" << size_ << ")" << std::endl;
    }

    // 反向遍历
    void printBackward() const {
        DListNode* curr = sentinel->prev;
        std::cout << "反向: ";
        while (curr != sentinel) {
            std::cout << curr->data << " ";
            curr = curr->prev;
        }
        std::cout << "(size=" << size_ << ")" << std::endl;
    }

    // 获取头部数据
    int front() const { return sentinel->next->data; }

    // 获取尾部数据
    int back() const { return sentinel->prev->data; }
    // 注意：front()/back() 仅可在 !empty() 时调用

    // 析构函数：释放所有节点
    ~CircularDoublyList() {
        DListNode* curr = sentinel->next;
        while (curr != sentinel) {
            DListNode* temp = curr->next;
            delete curr;
            curr = temp;
        }
        delete sentinel;
    }
};
```

**使用示例：**

```
int main() {
    CircularDoublyList list;

    list.pushBack(10);
    list.pushBack(20);
    list.pushBack(30);
    list.pushFront(5);

    list.printForward();   // 正向: 5 10 20 30 (size=4)
    list.printBackward();  // 反向: 30 20 10 5 (size=4)

    list.popFront();       // 删除 5
    list.popBack();        // 删除 30

    list.printForward();   // 正向: 10 20 (size=2)

    std::cout << "front: " << list.front() << std::endl;  // 10
    std::cout << "back: " << list.back() << std::endl;    // 20

    return 0;
}
```

> **💡 精妙之处**：注意 `pushBack` 是通过 `insertBefore(sentinel, val)` 实现的！在环形结构中，sentinel 的前面就是最后一个节点，所以"在 sentinel 之前插入"等价于"在链表末尾追加"。这个设计极其优雅，也是 `std::list` 的核心思想之一。

---

## 第四阶段：封装与模板化

---

### 10. 用类封装链表

#### 10.1 完整的类封装

前面我们已经初步用类封装了循环双向链表。现在我们把它进一步完善，加入**拷贝构造**、**赋值运算符**和**析构函数**（所谓的"三大件" / Rule of Three）。

```
#include <iostream>
#include <initializer_list>
#include <utility>

class LinkedList {
private:
    struct Node {
        int data;
        Node* prev;
        Node* next;
        Node(int val = 0) : data(val), prev(this), next(this) {}
    };

    Node* sentinel;
    int size_;

    // 私有辅助：在 pos 之前插入
    void insertBefore(Node* pos, int val) {
        Node* newNode = new Node(val);
        Node* prevNode = pos->prev;

        prevNode->next = newNode;
        newNode->prev = prevNode;
        newNode->next = pos;
        pos->prev = newNode;

        ++size_;
    }

    // 私有辅助：删除节点
    void eraseNode(Node* pos) {
        pos->prev->next = pos->next;
        pos->next->prev = pos->prev;
        delete pos;
        --size_;
    }

    // 私有辅助：清空所有数据节点（保留 sentinel）
    void clearNodes() {
        Node* curr = sentinel->next;
        while (curr != sentinel) {
            Node* temp = curr->next;
            delete curr;
            curr = temp;
        }
        sentinel->next = sentinel;
        sentinel->prev = sentinel;
        size_ = 0;
    }

public:
    // =================== 构造函数 ===================

    // 默认构造：空链表
    LinkedList() : size_(0) {
        sentinel = new Node();
    }

    // 初始化列表构造
    LinkedList(std::initializer_list<int> init) : LinkedList() {
        for (int val : init) {
            pushBack(val);
        }
    }

    // =================== 拷贝构造函数 ===================
    // 深拷贝：创建新的节点，不共享内存
    LinkedList(const LinkedList& other) : LinkedList() {
        Node* curr = other.sentinel->next;
        while (curr != other.sentinel) {
            pushBack(curr->data);
            curr = curr->next;
        }
    }

    // =================== 赋值运算符重载 ===================
    // 使用 Copy-and-Swap 惯用法 —— 异常安全且简洁
    LinkedList& operator=(LinkedList other) {
        // 注意：参数是值传递，已经完成了拷贝
        swap(other);
        return *this;
        // other（拷贝的旧数据）在这里被析构
    }

    // 交换函数
    void swap(LinkedList& other) {
        std::swap(sentinel, other.sentinel);
        std::swap(size_, other.size_);
    }

    // =================== 析构函数 ===================
    ~LinkedList() {
        clearNodes();
        delete sentinel;
    }

    // =================== 基本操作 ===================

    void pushFront(int val) { insertBefore(sentinel->next, val); }
    void pushBack(int val) { insertBefore(sentinel, val); }

    void popFront() {
        if (!empty()) eraseNode(sentinel->next);
    }

    void popBack() {
        if (!empty()) eraseNode(sentinel->prev);
    }

    int front() const { return sentinel->next->data; }
    int back() const { return sentinel->prev->data; }
    // 注意：front()/back() 仅可在 !empty() 时调用
    bool empty() const { return sentinel->next == sentinel; }
    int size() const { return size_; }

    void clear() { clearNodes(); }

    // =================== 输出 ===================
    void print() const {
        Node* curr = sentinel->next;
        std::cout << "[";
        while (curr != sentinel) {
            std::cout << curr->data;
            if (curr->next != sentinel) std::cout << ", ";
            curr = curr->next;
        }
        std::cout << "]" << std::endl;
    }
};
```

**使用示例：**

```
int main() {
    // 初始化列表构造
    LinkedList list1 = {1, 2, 3, 4, 5};
    list1.print();  // [1, 2, 3, 4, 5]

    // 拷贝构造
    LinkedList list2(list1);
    list2.pushBack(6);
    list2.print();  // [1, 2, 3, 4, 5, 6]
    list1.print();  // [1, 2, 3, 4, 5]  ← 深拷贝，互不影响

    // 赋值
    LinkedList list3;
    list3 = list1;
    list3.print();  // [1, 2, 3, 4, 5]

    return 0;
}
```

#### 10.2 Copy-and-Swap 惯用法详解

```
LinkedList& operator=(LinkedList other) {  // 值传递！
    swap(other);
    return *this;
}
```

这个写法看起来简单，但极其精妙：

1. **参数是值传递**（`LinkedList other` 而非 `const LinkedList& other`），在调用时就已经通过拷贝构造函数完成了深拷贝。
2. 然后用 `swap` 将当前对象的内容和拷贝的新内容交换。
3. 函数结束时，`other`（现在持有旧数据）的析构函数自动释放旧内存。

**优点：**

- **异常安全**：如果拷贝构造过程中抛出异常，原对象不受影响。
- **自赋值安全**：`list = list` 也能正确工作。
- **代码简洁**：不需要手动检查自赋值、手动释放旧内存。

---

### 11. 模板化链表

#### 11.1 使用 template<typename T> 支持泛型

前面的 `LinkedList` 只能存 `int`，如果想存 `double`、`string` 或自定义类型，难道要写多个版本？当然不用——C++ 的**模板**就是为此而生。

```
template<typename T>
class LinkedList {
private:
    struct Node {
        T data;             // 数据域变为泛型 T
        Node* prev;
        Node* next;
        Node(const T& val = T()) : data(val), prev(this), next(this) {}
    };

    Node* sentinel;
    int size_;

    void insertBefore(Node* pos, const T& val) {
        Node* newNode = new Node(val);
        Node* prevNode = pos->prev;
        prevNode->next = newNode;
        newNode->prev = prevNode;
        newNode->next = pos;
        pos->prev = newNode;
        ++size_;
    }

    void eraseNode(Node* pos) {
        pos->prev->next = pos->next;
        pos->next->prev = pos->prev;
        delete pos;
        --size_;
    }

    void clearNodes() {
        Node* curr = sentinel->next;
        while (curr != sentinel) {
            Node* temp = curr->next;
            delete curr;
            curr = temp;
        }
        sentinel->next = sentinel;
        sentinel->prev = sentinel;
        size_ = 0;
    }

public:
    LinkedList() : size_(0) { sentinel = new Node(); }

    LinkedList(std::initializer_list<T> init) : LinkedList() {
        for (const T& val : init) pushBack(val);
    }

    LinkedList(const LinkedList& other) : LinkedList() {
        Node* curr = other.sentinel->next;
        while (curr != other.sentinel) {
            pushBack(curr->data);
            curr = curr->next;
        }
    }

    LinkedList& operator=(LinkedList other) {
        swap(other);
        return *this;
    }

    void swap(LinkedList& other) {
        std::swap(sentinel, other.sentinel);
        std::swap(size_, other.size_);
    }

    ~LinkedList() { clearNodes(); delete sentinel; }

    void pushFront(const T& val) { insertBefore(sentinel->next, val); }
    void pushBack(const T& val)  { insertBefore(sentinel, val); }
    void popFront() { if (!empty()) eraseNode(sentinel->next); }
    void popBack()  { if (!empty()) eraseNode(sentinel->prev); }

    T& front() { return sentinel->next->data; }
    T& back()  { return sentinel->prev->data; }
    const T& front() const { return sentinel->next->data; }
    const T& back()  const { return sentinel->prev->data; }
    // 注意：front()/back() 仅可在 !empty() 时调用

    bool empty() const { return sentinel->next == sentinel; }
    int size() const { return size_; }
    void clear() { clearNodes(); }
};
```

> **⚠️ 限制说明**：这个简化模板把哨兵节点也建模为 `Node<T>`，并使用 `Node(const T& val = T())` 初始化哨兵，因此要求 `T` 可默认构造。工业级 `std::list` 会把“哨兵节点”和“数据节点”分离来避免这个限制。

#### 11.2 模板类的注意事项

**关键规则：模板类的声明和实现通常都放在头文件中（`.h` / `.hpp`）。**

原因是模板在**编译期实例化**。编译器看到 `LinkedList<int>` 时，需要能看到完整的实现代码才能生成对应的代码。如果实现放在 `.cpp` 里，其他翻译单元看不到实现，就会报链接错误。

```
// ✅ 正确做法：全部放在头文件中
// LinkedList.hpp
template<typename T>
class LinkedList {
    // 声明 + 实现全部在这里
};

// ❌ 错误做法：声明和实现分离
// LinkedList.h   → 只有声明
// LinkedList.cpp → 实现（其他 .cpp 文件 include 头文件后找不到实现）
```

**使用示例：**

```
#include "LinkedList.hpp"
#include <string>

int main() {
    // 存 int
    LinkedList<int> intList = {1, 2, 3, 4, 5};

    // 存 double
    LinkedList<double> doubleList = {1.1, 2.2, 3.3};

    // 存 string
    LinkedList<std::string> strList = {"Hello", "World", "C++"};
    strList.pushBack("Template");

    return 0;
}
```

---

## 第五阶段：迭代器模式

---

### 12. 自定义迭代器

#### 12.1 什么是迭代器？为什么需要迭代器？

**迭代器（Iterator）** 是一种**设计模式**，它提供了一种**统一的方式**来遍历容器中的元素，而不需要暴露容器的内部实现细节。

**没有迭代器的问题：**

```
// 遍历数组 —— 用下标
for (int i = 0; i < arr.size(); ++i)
    std::cout << arr[i];

// 遍历链表 —— 用指针
for (Node* p = head; p != nullptr; p = p->next)
    std::cout << p->data;

// 每种容器的遍历方式都不一样！
// 如果写一个通用算法（如排序、查找），就需要为每种容器写一个版本
```

**有了迭代器：**

```
// 不管是数组、链表、树还是哈希表，都用同一种方式遍历：
for (auto it = container.begin(); it != container.end(); ++it)
    std::cout << *it;

// 甚至可以用范围 for（底层就是迭代器）
for (auto& item : container)
    std::cout << item;
```

**迭代器的本质**：它像一个"智能指针"，包装了对容器内部节点的访问。通过重载 `*`、`->`、`++`、`--`、`==`、`!=` 等运算符，让它用起来和指针一样自然。

#### 12.2 为链表实现迭代器类

我们在前面的模板化 `LinkedList` 类中加入迭代器的支持：

```
template<typename T>
class LinkedList {
private:
    struct Node {
        T data;
        Node* prev;
        Node* next;
        Node(const T& val = T()) : data(val), prev(this), next(this) {}
    };

    Node* sentinel;
    int size_;

    // ... 省略之前的私有方法 ...

public:
    // =================== 迭代器类 ===================
    class iterator {
    private:
        Node* ptr;  // 指向当前节点

        // 让 LinkedList 可以访问 iterator 的私有成员
        friend class LinkedList;

    public:
        // 构造函数
        iterator(Node* p = nullptr) : ptr(p) {}

        // 解引用运算符：获取当前节点的数据
        T& operator*() { return ptr->data; }

        // 箭头运算符：当 T 是结构体/类时，访问其成员
        T* operator->() { return &(ptr->data); }

        // 前置 ++：移动到下一个节点
        iterator& operator++() {
            ptr = ptr->next;
            return *this;
        }

        // 后置 ++
        iterator operator++(int) {
            iterator tmp = *this;
            ptr = ptr->next;
            return tmp;
        }

        // 前置 --：移动到上一个节点
        iterator& operator--() {
            ptr = ptr->prev;
            return *this;
        }

        // 后置 --
        iterator operator--(int) {
            iterator tmp = *this;
            ptr = ptr->prev;
            return tmp;
        }

        // 比较运算符
        bool operator==(const iterator& other) const {
            return ptr == other.ptr;
        }

        bool operator!=(const iterator& other) const {
            return ptr != other.ptr;
        }
    };

    // =================== const 迭代器类 ===================
    class const_iterator {
    private:
        const Node* ptr;
        friend class LinkedList;

    public:
        const_iterator(const Node* p = nullptr) : ptr(p) {}

        // 从普通迭代器隐式转换
        const_iterator(const iterator& it) : ptr(it.ptr) {}

        const T& operator*() const { return ptr->data; }
        const T* operator->() const { return &(ptr->data); }

        const_iterator& operator++() { ptr = ptr->next; return *this; }
        const_iterator operator++(int) {
            const_iterator tmp = *this;
            ptr = ptr->next;
            return tmp;
        }
        const_iterator& operator--() { ptr = ptr->prev; return *this; }
        const_iterator operator--(int) {
            const_iterator tmp = *this;
            ptr = ptr->prev;
            return tmp;
        }

        bool operator==(const const_iterator& other) const { return ptr == other.ptr; }
        bool operator!=(const const_iterator& other) const { return ptr != other.ptr; }
    };

    // =================== begin() / end() ===================

    iterator begin() { return iterator(sentinel->next); }
    iterator end()   { return iterator(sentinel); }  // end 指向哨兵节点

    const_iterator begin() const { return const_iterator(sentinel->next); }
    const_iterator end()   const { return const_iterator(sentinel); }

    const_iterator cbegin() const { return const_iterator(sentinel->next); }
    const_iterator cend()   const { return const_iterator(sentinel); }

    // =================== 用迭代器实现 insert / erase ===================

    // 在 pos 之前插入，返回指向新元素的迭代器
    iterator insert(iterator pos, const T& val) {
        Node* newNode = new Node(val);
        Node* posNode = pos.ptr;
        Node* prevNode = posNode->prev;

        prevNode->next = newNode;
        newNode->prev = prevNode;
        newNode->next = posNode;
        posNode->prev = newNode;

        ++size_;
        return iterator(newNode);
    }

    // 删除 pos 指向的节点，返回指向下一个元素的迭代器
    iterator erase(iterator pos) {
        Node* posNode = pos.ptr;
        if (posNode == sentinel) return end();  // 约定：不删除 end()
        Node* nextNode = posNode->next;

        posNode->prev->next = nextNode;
        nextNode->prev = posNode->prev;

        delete posNode;
        --size_;
        return iterator(nextNode);
    }

    // ... 保留之前的构造函数、析构函数等 ...
};
```

#### 12.3 begin() 和 end() 的设计哲学

```
链表：   [sentinel] ⟺ [10] ⟺ [20] ⟺ [30] ⟺ [sentinel]
                        ↑                        ↑
                     begin()                   end()

有效范围：[begin, end)  —— 左闭右开区间！
```

- `begin()` 返回指向**第一个数据节点**的迭代器
- `end()` 返回指向**哨兵节点**的迭代器（即"越过最后一个元素的位置"）
- 遍历条件：`it != end()`，当迭代器到达哨兵节点时，说明遍历完毕

这种**左闭右开**的设计是 STL 的核心约定，所有容器都遵循。

**使用示例：**

```
int main() {
    LinkedList<int> list = {10, 20, 30, 40, 50};

    // 迭代器遍历
    for (auto it = list.begin(); it != list.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;  // 10 20 30 40 50

    // 范围 for（编译器自动翻译为迭代器版本）
    for (int& val : list) {
        val *= 2;  // 修改每个元素
    }
    for (int val : list) {
        std::cout << val << " ";
    }
    std::cout << std::endl;  // 20 40 60 80 100

    // 用迭代器插入
    auto it = list.begin();
    ++it;  // 指向第 2 个元素（40）
    list.insert(it, 35);  // 在 40 前面插入 35
    // 链表变为：20 35 40 60 80 100

    // 用迭代器删除
    it = list.begin();
    ++it;  // 指向 35
    it = list.erase(it);  // 删除 35，it 现在指向 40
    // 链表变为：20 40 60 80 100

    // 存储结构体
    struct Student {
        std::string name;
        int score;
    };

    LinkedList<Student> students;
    students.pushBack({"Alice", 95});
    students.pushBack({"Bob", 87});

    for (auto it = students.begin(); it != students.end(); ++it) {
        std::cout << it->name << ": " << it->score << std::endl;
        // 箭头运算符 -> 直接访问 Student 的成员
    }

    return 0;
}
```

> **📌 重要**：`erase` 之后，原来的迭代器就**失效**了（因为它指向的节点已被删除）。所以 `erase` 返回下一个有效迭代器，使用时一定要接住返回值：`it = list.erase(it)`。

---

## 第六阶段：过渡到 STL std::list

---

### 13. STL list 的使用

现在你已经亲手实现了链表、迭代器，理解了底层原理。接下来过渡到 `std::list` 会非常顺畅——因为**你写的就是 `std::list` 的简化版！**

```
#include <list>
#include <iostream>
#include <algorithm>
#include <iterator>   // std::advance
#include <functional> // std::greater
```

#### 13.1 构造函数

```
// 1. 默认构造：空链表
std::list<int> l1;

// 2. 指定个数和初始值
std::list<int> l2(5, 100);         // {100, 100, 100, 100, 100}

// 3. 初始化列表
std::list<int> l3 = {1, 2, 3, 4, 5};

// 4. 拷贝构造
std::list<int> l4(l3);             // l4 是 l3 的深拷贝

// 5. 迭代器范围构造
std::list<int> l5(l3.begin(), l3.end());

// 6. 移动构造（C++11）
std::list<int> l6(std::move(l5));  // l5 变为空，l6 获得数据
```

#### 13.2 赋值操作

```
std::list<int> l1 = {1, 2, 3};
std::list<int> l2;

// operator=
l2 = l1;

// assign
l2.assign(5, 10);                   // {10, 10, 10, 10, 10}
l2.assign({7, 8, 9});              // {7, 8, 9}
l2.assign(l1.begin(), l1.end());   // {1, 2, 3}
```

#### 13.3 大小操作

```
std::list<int> l = {1, 2, 3};
l.size();     // 3
l.empty();    // false
l.resize(5);       // {1, 2, 3, 0, 0}  不足补 0
l.resize(5, 99);   // 不足补 99
l.resize(2);       // {1, 2}  多余截断
// 注意：std::list 没有 capacity()，因为链表不需要预分配
```

#### 13.4 插入与删除

```
std::list<int> l = {10, 20, 30};

// ===== 头尾操作 =====
l.push_back(40);       // {10, 20, 30, 40}
l.push_front(5);       // {5, 10, 20, 30, 40}
l.pop_back();          // {5, 10, 20, 30}
l.pop_front();         // {10, 20, 30}

l.emplace_back(40);    // 原地构造，比 push_back 更高效（对复杂对象而言）
l.emplace_front(5);    // 同理

// ===== 迭代器插入 =====
auto it = l.begin();
std::advance(it, 2);   // 移动迭代器到第 3 个位置
l.insert(it, 25);      // 在 it 之前插入 25
l.insert(it, 3, 99);   // 在 it 之前插入 3 个 99
l.insert(it, {61, 62});// 在 it 之前插入 {61, 62}

l.emplace(it, 50);     // 原地构造并插入

// ===== 删除 =====
it = l.begin();
it = l.erase(it);      // 删除 it 指向的元素，返回下一个迭代器
l.erase(l.begin(), l.end());  // 删除范围内所有元素（等效于 clear）

l.remove(10);          // 删除所有值为 10 的元素
l.remove_if([](int x) { return x > 50; });  // 删除所有大于 50 的元素

l.clear();             // 清空
```

#### 13.5 数据访问

```
std::list<int> l = {10, 20, 30};
l.front();  // 10（首元素引用）
l.back();   // 30（尾元素引用）

// ⚠️ list 不支持 [] 和 at()，因为链表不支持随机访问！
// l[0];    // 编译错误！
// l.at(0); // 编译错误！
```

#### 13.6 遍历

```
std::list<int> l = {1, 2, 3, 4, 5};

// 1. 迭代器遍历
for (auto it = l.begin(); it != l.end(); ++it) {
    std::cout << *it << " ";
}

// 2. 范围 for
for (int val : l) {
    std::cout << val << " ";
}

// 3. 反向遍历
for (auto rit = l.rbegin(); rit != l.rend(); ++rit) {
    std::cout << *rit << " ";
}
// 输出：5 4 3 2 1
```

#### 13.7 list 特有的算法操作

`std::list` 提供了一些**成员函数版本**的算法，比通用 `<algorithm>` 版本更高效（因为只需修改指针，不需要移动元素）。

```
// ===== sort() =====
// list 不能用 std::sort()（因为不支持随机访问迭代器）
// 必须用成员函数 sort()
std::list<int> l = {5, 3, 1, 4, 2};
l.sort();                         // 升序：{1, 2, 3, 4, 5}
l.sort(std::greater<int>());      // 降序：{5, 4, 3, 2, 1}

// ===== reverse() =====
l.reverse();  // 反转链表

// ===== unique() =====
// 删除连续的重复元素（通常先 sort 再 unique）
std::list<int> l2 = {1, 1, 2, 2, 3, 3, 3};
l2.unique();  // {1, 2, 3}

// ===== merge() =====
// 将另一个有序链表合并进来（两个链表都必须有序）
std::list<int> a = {1, 3, 5};
std::list<int> b = {2, 4, 6};
a.merge(b);   // a = {1, 2, 3, 4, 5, 6}，b 变成空链表
// 底层只是修改指针，O(n)，不需要复制元素

// ===== splice() =====
// 将另一个链表的元素"移接"到当前链表（O(1)！只修改指针）
std::list<int> x = {1, 2, 3};
std::list<int> y = {10, 20, 30};

auto pos = x.begin();
++pos;  // 指向 2

// 把 y 的所有元素移接到 x 中 pos 之前
x.splice(pos, y);
// x = {1, 10, 20, 30, 2, 3}
// y = {}（变空了，元素被移走了，不是复制！）

// splice 也可以只移动单个元素或一段范围
std::list<int> m = {100, 200, 300};
x.splice(x.begin(), m, m.begin());
// 只把 m 的第一个元素 100 移接到 x 头部
// x = {100, 1, 10, 20, 30, 2, 3}
// m = {200, 300}
```

> **📌 `splice` 是 `list` 最强大的操作之一**：整表移接和单节点移接通常是 O(1)（只改指针，不搬元素）；区间移接通常是 O(k)（k 为移接区间长度）。

---

### 14. STL list 源码简析（选学）

> 这一节帮助你理解 `std::list` 的底层实现，和你手写的版本做对照。

#### 14.1 节点结构

在 GCC 的 libstdc++ 实现中，`list` 的节点大致如下：

```
// 基类：只存储前后指针（不含数据）
struct _List_node_base {
    _List_node_base* _M_next;
    _List_node_base* _M_prev;
};

// 派生类：添加数据域
template<typename _Tp>
struct _List_node : public _List_node_base {
    _Tp _M_data;  // 实际存储的数据
};
```

**为什么要分两层？**

哨兵节点不需要存储数据！哨兵节点只需要 `prev` 和 `next` 指针。所以 `std::list` 内部的哨兵节点类型是 `_List_node_base`（没有数据域），而数据节点类型是 `_List_node<T>`（有数据域）。这样哨兵节点不会浪费一个 `T` 的空间。

```
你手写的版本：  Node sentinel（data 字段浪费了空间）
std::list：    _List_node_base sentinel（没有 data 字段，更节省）
               _List_node<T> 数据节点（有 data 字段）
```

#### 14.2 迭代器实现

```
template<typename _Tp>
struct _List_iterator {
    typedef _Tp  value_type;
    typedef _Tp& reference;
    typedef _Tp* pointer;

    _List_node_base* _M_node;  // 内部存储的是基类指针

    reference operator*() const {
        // 向下转型为 _List_node<_Tp>*，然后访问 _M_data
        return static_cast<_List_node<_Tp>*>(_M_node)->_M_data;
    }

    pointer operator->() const {
        return &(operator*());
    }

    _List_iterator& operator++() {
        _M_node = _M_node->_M_next;
        return *this;
    }

    _List_iterator& operator--() {
        _M_node = _M_node->_M_prev;
        return *this;
    }

    bool operator==(const _List_iterator& other) const {
        return _M_node == other._M_node;
    }
    // ...
};
```

> 你会发现，这和你手写的迭代器几乎一模一样！唯一的区别是 STL 版本使用了基类指针 + `static_cast` 的技巧。

#### 14.3 内存分配器（Allocator）

```
template<typename _Tp, typename _Alloc = std::allocator<_Tp>>
class list {
    // ...
};
```

`std::list` 的第二个模板参数是**分配器（Allocator）**，默认是 `std::allocator<T>`。

- 分配器负责**内存的申请和释放**，替代了直接使用 `new` / `delete`。
- 默认的 `std::allocator` 底层调用的还是 `operator new` / `operator delete`。
- 分配器的好处是**可替换**：你可以传入自定义的分配器，比如内存池分配器，以优化大量小对象分配的性能（链表节点恰好就是大量小对象）。
- 这个参数对初学者来说可以忽略，使用默认即可。

---

## 总结：从手写链表到 std::list 的对照

| 你手写的                                        | std::list 对应的                        |
| ----------------------------------------------- | --------------------------------------- |
| struct Node { T data; Node* prev; Node* next; } | _List_node_base + _List_node<T>         |
| Node* sentinel                                  | _List_node_base _M_node（对象内嵌成员） |
| class iterator { Node* ptr; }                   | _List_iterator<T>                       |
| begin() → iterator(sentinel->next)              | 完全一致                                |
| end() → iterator(sentinel)                      | 完全一致                                |
| insertBefore(pos, val)                          | list::insert(pos, val)                  |
| eraseNode(pos)                                  | list::erase(pos)                        |
| pushBack(val) → insertBefore(sentinel)          | list::push_back(val)                    |
| new Node(val)                                   | 使用 Allocator::allocate() + 构造       |

> **🎯 关键认知**：`std::list` 本质上就是你在第三~第五阶段手写的那个**带哨兵头节点的双向循环链表 + 迭代器类**。STL 只是在此基础上加入了分配器、更完善的类型特征（type traits）、异常安全等工业级特性。

---

## 附录：链表常见面试题速查

| 题目              | 核心技巧                     | 复杂度     |
| ----------------- | ---------------------------- | ---------- |
| 反转链表          | 三指针迭代 / 递归            | O(n)       |
| 检测环            | 快慢指针                     | O(n)       |
| 找环入口          | 快慢指针 + 数学推导          | O(n)       |
| 合并两个有序链表  | 哨兵节点 + 双指针            | O(n+m)     |
| 找倒数第 k 个节点 | 快慢指针（快先走 k 步）      | O(n)       |
| 链表排序          | 归并排序                     | O(n log n) |
| 判断回文链表      | 找中点 + 反转后半段 + 比较   | O(n)       |
| 相交链表找交点    | 双指针各走一遍对方路径       | O(n+m)     |
| 删除重复节点      | 有序则顺序去重；无序用哈希表 | O(n)       |

---

以上就是全部内容！从基础的节点定义到手写带迭代器的模板链表，再到 `std::list` 的源码解析，这条路线走下来，你对链表和 `std::list` 的理解应该相当扎实了。接下来学习 `std::list` 的具体 API 时，你不会只是"背接口"，而是真正理解每一个操作在底层做了什么。🚀
