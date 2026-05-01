# C++ STL priority_queue 容器（补充）

> 原课程讲完了 `stack` 和 `queue`，却漏掉了同属容器适配器的 `priority_queue`，紧接着补上最自然。

---

## 目录

1. [priority_queue - 基本概念（大顶堆 / 小顶堆）](#1-priority_queue---基本概念大顶堆--小顶堆)
2. [priority_queue - 构造函数](#2-priority_queue---构造函数)
3. [priority_queue - 常用接口](#3-priority_queue---常用接口)
4. [priority_queue - 存放自定义数据类型](#4-priority_queue---存放自定义数据类型)

---

## 1. priority_queue - 基本概念（大顶堆 / 小顶堆）

### 1.1 什么是 priority_queue？

`priority_queue`（优先级队列）是 C++ STL 提供的第三个**容器适配器**（另外两个是 `stack` 和 `queue`）。

回忆一下：

| 容器适配器           | 特点                                 |
| -------------------- | ------------------------------------ |
| `stack`              | 后进先出（LIFO）                     |
| `queue`              | 先进先出（FIFO）                     |
| **`priority_queue`** | **每次出队的都是"优先级最高"的元素** |

普通队列是"排队"——谁先来谁先走；**优先级队列**是"看病挂号"——不管谁先来，病情最严重的（优先级最高的）先看。

> **一句话理解：priority_queue 保证你每次取到的永远是当前容器里"最大"（或"最小"）的那个元素。**

使用前需要包含头文件：

```cpp name=include_header.cpp
#include <queue>   // priority_queue 和 queue 共用同一个头文件
```

### 1.2 底层原理——堆（Heap）

`priority_queue` 的底层默认使用 `vector` 作为存储容器，并在其上维护一个**二叉堆（Binary Heap）** 结构。

**什么是堆？**

堆是一棵**完全二叉树**，满足以下性质之一：

- **大顶堆（Max-Heap）**：每个节点的值 **≥** 其子节点的值 → 根节点（堆顶）是最大值
- **小顶堆（Min-Heap）**：每个节点的值 **≤** 其子节点的值 → 根节点（堆顶）是最小值

用图来理解：

```
        大顶堆                    小顶堆
          90                        10
         /  \                      /  \
        70    80                  30    20
       / \   /                   / \   /
      50 60 40                  70  50 80
```

堆的特点：
- 插入元素：O(log n) —— 新元素放到末尾，然后"上浮"到正确位置
- 删除堆顶：O(log n) —— 堆顶移除，末尾元素补到堆顶，然后"下沉"到正确位置
- 获取堆顶：O(1) —— 直接访问

> **重要记忆：C++ 的 `priority_queue` 默认是大顶堆，即 `top()` 返回的是最大元素。**

### 1.3 与 stack / queue 的对比

| 特性         | `stack`              | `queue`                                | `priority_queue`     |
| ------------ | -------------------- | -------------------------------------- | -------------------- |
| 访问方式     | 只能访问栈顶 `top()` | 只能访问队头 `front()` 和队尾 `back()` | 只能访问堆顶 `top()` |
| 出入规则     | LIFO（后进先出）     | FIFO（先进先出）                       | 优先级最高的先出     |
| 底层默认容器 | `deque`              | `deque`                                | `vector`             |
| 是否可遍历   | ❌ 不可以             | ❌ 不可以                               | ❌ 不可以             |
| 是否有迭代器 | ❌ 没有               | ❌ 没有                                 | ❌ 没有               |

> **注意：三者都是容器适配器，都没有迭代器，都不支持遍历！**

### 1.4 priority_queue 的模板声明

```cpp name=template_declaration.cpp
template <
    class T,                                    // 元素类型
    class Container = vector<T>,                // 底层容器，默认 vector
    class Compare   = less<typename Container::value_type>  // 比较器，默认 less（大顶堆）
>
class priority_queue;
```

三个模板参数的含义：

| 参数        | 含义                           | 默认值              |
| ----------- | ------------------------------ | ------------------- |
| `T`         | 存储的元素类型                 | （必须指定）        |
| `Container` | 底层容器类型                   | `vector<T>`         |
| `Compare`   | 比较器（决定大顶堆还是小顶堆） | `less<T>`（大顶堆） |

> **关键理解：**
> - `less<T>` → **大顶堆**（默认） → `top()` 是最大值
> - `greater<T>` → **小顶堆** → `top()` 是最小值
>
> 初学者常常搞反：`less` 对应"大顶"，`greater` 对应"小顶"。
> 助记方法：`less` 的意思是"父节点 < 子节点时需要调整"，所以最终父节点 ≥ 子节点 → 大顶堆。

---

## 2. priority_queue - 构造函数

### 2.1 默认构造（大顶堆）

最简单的用法，创建一个空的大顶堆：

```cpp name=default_constructor.cpp
#include <iostream>
#include <queue>
using namespace std;

int main() {
    // 默认构造：大顶堆，底层用 vector，比较器用 less
    priority_queue<int> pq;

    pq.push(30);
    pq.push(10);
    pq.push(50);
    pq.push(20);
    pq.push(40);

    cout << "大顶堆，依次取出：" << endl;
    while (!pq.empty()) {
        cout << pq.top() << " ";   // 每次取出当前最大值
        pq.pop();
    }
    cout << endl;

    return 0;
}
```

**输出：**
```
大顶堆，依次取出：
50 40 30 20 10
```

> 可以看到，不管我们以什么顺序 push，取出时总是从大到小。

### 2.2 指定比较器构造小顶堆

要创建小顶堆，需要**显式指定全部三个模板参数**：

```cpp name=min_heap_constructor.cpp
#include <iostream>
#include <queue>
#include <vector>
#include <functional>   // greater<T> 在这个头文件中（实际上 <queue> 通常已间接包含）
using namespace std;

int main() {
    // 小顶堆：需要写全三个模板参数
    //   第一个：元素类型 int
    //   第二个：底层容器 vector<int>
    //   第三个：比较器 greater<int>
    priority_queue<int, vector<int>, greater<int>> pq;

    pq.push(30);
    pq.push(10);
    pq.push(50);
    pq.push(20);
    pq.push(40);

    cout << "小顶堆，依次取出：" << endl;
    while (!pq.empty()) {
        cout << pq.top() << " ";   // 每次取出当前最小值
        pq.pop();
    }
    cout << endl;

    return 0;
}
```

**输出：**
```
小顶堆，依次取出：
10 20 30 40 50
```

> **常见错误提醒：** 不能只写 `priority_queue<int, greater<int>>`，因为第二个模板参数是底层容器类型，不是比较器。必须把 `vector<int>` 也写上。

### 2.3 用已有容器（迭代器范围）初始化

可以用一个已有的数组或容器来初始化 priority_queue：

```cpp name=range_constructor.cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

int main() {
    // ========== 方法一：用数组初始化 ==========
    int arr[] = {30, 10, 50, 20, 40};

    // 传入首尾迭代器（指针）
    priority_queue<int> pq1(arr, arr + 5);

    cout << "用数组初始化的大顶堆：";
    while (!pq1.empty()) {
        cout << pq1.top() << " ";
        pq1.pop();
    }
    cout << endl;

    // ========== 方法二：用 vector 初始化 ==========
    vector<int> v = {35, 15, 55, 25, 45};

    priority_queue<int> pq2(v.begin(), v.end());

    cout << "用 vector 初始化的大顶堆：";
    while (!pq2.empty()) {
        cout << pq2.top() << " ";
        pq2.pop();
    }
    cout << endl;

    // ========== 方法三：用 vector 初始化小顶堆 ==========
    vector<int> v2 = {35, 15, 55, 25, 45};

    // 注意：迭代器范围放在构造函数参数中
    priority_queue<int, vector<int>, greater<int>> pq3(v2.begin(), v2.end());

    cout << "用 vector 初始化的小顶堆：";
    while (!pq3.empty()) {
        cout << pq3.top() << " ";
        pq3.pop();
    }
    cout << endl;

    return 0;
}
```

**输出：**
```
用数组初始化的大顶堆：50 40 30 20 10
用 vector 初始化的大顶堆：55 45 35 25 15
用 vector 初始化的小顶堆：15 25 35 45 55
```

### 2.4 构造函数汇总表

| 构造方式           | 代码示例                                                                 | 说明               |
| ------------------ | ------------------------------------------------------------------------ | ------------------ |
| 默认构造（大顶堆） | `priority_queue<int> pq;`                                                | 空的大顶堆         |
| 小顶堆             | `priority_queue<int, vector<int>, greater<int>> pq;`                     | 空的小顶堆         |
| 用迭代器范围初始化 | `priority_queue<int> pq(v.begin(), v.end());`                            | 用已有数据建堆     |
| 用迭代器 + 小顶堆  | `priority_queue<int, vector<int>, greater<int>> pq(v.begin(), v.end());` | 用已有数据建小顶堆 |

---

## 3. priority_queue - 常用接口

### 3.1 接口一览表

| 函数               | 功能                             | 时间复杂度 |
| ------------------ | -------------------------------- | ---------- |
| `push(elem)`       | 将元素 elem 插入堆中             | O(log n)   |
| `pop()`            | 移除堆顶元素                     | O(log n)   |
| `top()`            | 返回堆顶元素（只读引用）         | O(1)       |
| `size()`           | 返回堆中元素个数                 | O(1)       |
| `empty()`          | 判断堆是否为空                   | O(1)       |
| `emplace(args...)` | 原地构造并插入元素（C++11）      | O(log n)   |
| `swap(other)`      | 与另一个 priority_queue 交换内容 | O(1)       |

> **注意：priority_queue 没有 `front()`、`back()`、`clear()` 函数，也没有迭代器！**

### 3.2 完整示例

```cpp name=common_interfaces.cpp
#include <iostream>
#include <queue>
using namespace std;

int main() {
    priority_queue<int> pq;

    // ==================== push() ====================
    // 向堆中插入元素
    pq.push(30);
    pq.push(10);
    pq.push(50);
    pq.push(20);
    pq.push(40);
    cout << "push 了 5 个元素：30, 10, 50, 20, 40" << endl;

    // ==================== size() ====================
    // 返回元素个数
    cout << "当前元素个数：" << pq.size() << endl;   // 5

    // ==================== empty() ====================
    // 判断是否为空
    if (!pq.empty()) {
        cout << "优先队列不为空" << endl;
    }

    // ==================== top() ====================
    // 访问堆顶元素（大顶堆 → 最大值）
    cout << "堆顶元素（最大值）：" << pq.top() << endl;   // 50

    // ==================== pop() ====================
    // 移除堆顶元素
    cout << endl << "依次 pop 并输出：" << endl;
    while (!pq.empty()) {
        cout << "  top = " << pq.top()
             << "  (size = " << pq.size() << ")" << endl;
        pq.pop();   // 移除堆顶
    }
	
    cout << endl;
    cout << "全部 pop 后，empty() = "
         << (pq.empty() ? "true" : "false") << endl;

    return 0;
}
```

**输出：**
```
push 了 5 个元素：30, 10, 50, 20, 40
当前元素个数：5
优先队列不为空
堆顶元素（最大值）：50

依次 pop 并输出：
  top = 50  (size = 5)
  top = 40  (size = 4)
  top = 30  (size = 3)
  top = 20  (size = 2)
  top = 10  (size = 1)

全部 pop 后，empty() = true
```

### 3.3 emplace() 与 push() 的区别

```cpp name=emplace_demo.cpp
#include <iostream>
#include <queue>
#include <string>
using namespace std;

int main() {
    priority_queue<string> pq;

    // push：先构造临时对象，再拷贝/移动到堆中
    string s = "Hello";
    pq.push(s);           // 拷贝
    pq.push("World");     // 隐式构造临时 string，然后移动

    // emplace：直接在堆内部原地构造，避免额外拷贝/移动
    // 参数直接转发给 string 的构造函数
    pq.emplace("Zebra");      // 等价于 pq.push(string("Zebra"))，但更高效
    pq.emplace(5, 'A');        // 构造 string("AAAAA")

    cout << "依次取出（按字典序从大到小）：" << endl;
    while (!pq.empty()) {
        cout << "  " << pq.top() << endl;
        pq.pop();
    }

    return 0;
}
```

**输出：**
```
依次取出（按字典序从大到小）：
  Zebra
  World
  Hello
  AAAAA
```

> **建议：** 对于简单类型（int、double），`push` 和 `emplace` 差别不大；对于复杂对象（如 `string`、自定义类），优先使用 `emplace` 以获得更好的性能。

### 3.4 swap() 交换

```cpp name=swap_demo.cpp
#include <iostream>
#include <queue>
using namespace std;

void printPQ(string name, priority_queue<int> pq) {
    cout << name << ": ";
    while (!pq.empty()) {
        cout << pq.top() << " ";
        pq.pop();
    }
    cout << endl;
}

int main() {
    priority_queue<int> pq1;
    pq1.push(10);
    pq1.push(30);
    pq1.push(20);

    priority_queue<int> pq2;
    pq2.push(100);
    pq2.push(200);

    cout << "===== 交换前 =====" << endl;
    printPQ("pq1", pq1);
    printPQ("pq2", pq2);

    pq1.swap(pq2);   // 交换两个优先队列的全部内容

    cout << "===== 交换后 =====" << endl;
    printPQ("pq1", pq1);
    printPQ("pq2", pq2);

    return 0;
}
```

**输出：**
```
===== 交换前 =====
pq1: 30 20 10
pq2: 200 100
===== 交换后 =====
pq1: 200 100
pq2: 30 20 10
```

### 3.5 常见"坑"与注意事项

**坑 1：对空堆调用 `top()` 或 `pop()` 是未定义行为**

```cpp name=pitfall_empty.cpp
priority_queue<int> pq;
// pq.top();    // ❌ 未定义行为！程序可能崩溃
// pq.pop();    // ❌ 未定义行为！

// 正确做法：先判断
if (!pq.empty()) {
    cout << pq.top() << endl;
    pq.pop();
}
```

**坑 2：priority_queue 没有 `clear()` 函数**

如果需要清空，通常用以下方法：

```cpp name=pitfall_clear.cpp
priority_queue<int> pq;
pq.push(10);
pq.push(20);
pq.push(30);

// 方法一：逐个 pop
while (!pq.empty()) {
    pq.pop();
}

// 方法二：用空的 priority_queue 交换（更简洁高效）
priority_queue<int>().swap(pq);

// 方法三：直接重新赋值（C++11）
pq = priority_queue<int>();
```

**坑 3：不能遍历 priority_queue**

```cpp name=pitfall_iterate.cpp
priority_queue<int> pq;
pq.push(10);
pq.push(20);

// ❌ 以下写法全部错误：
// for (auto it = pq.begin(); it != pq.end(); it++) {}  // 没有迭代器
// for (auto x : pq) {}                                  // 不支持范围 for

// ✅ 想"遍历"只能一边 top 一边 pop（会破坏原数据）
// 如果需要保留原数据，先拷贝一份
priority_queue<int> temp = pq;   // 拷贝
while (!temp.empty()) {
    cout << temp.top() << " ";
    temp.pop();
}
```

---

## 4. priority_queue - 存放自定义数据类型

这是 priority_queue 最重要也最常考的部分。当我们存放自定义类型（如结构体、类）时，必须告诉编译器**如何比较两个对象的优先级**。

有两种主流方式：
1. **在类内部重载 `operator<`**
2. **自定义比较器（仿函数 / lambda）**

### 4.1 背景：为什么需要自定义比较？

```cpp name=custom_type_error.cpp
#include <queue>
using namespace std;

struct Student {
    string name;
    int score;
};

int main() {
    priority_queue<Student> pq;
    pq.push({"Alice", 90});
    pq.push({"Bob", 85});
    // ❌ 编译错误！编译器不知道两个 Student 谁"大"谁"小"
    return 0;
}
```

编译器报错大意：`no match for 'operator<'`。因为默认比较器 `less<Student>` 需要 `Student` 支持 `<` 运算符。

### 4.2 方法一：重载 `operator<`（推荐用于大顶堆）

在结构体/类内部定义 `<` 运算符：

```cpp name=overload_operator.cpp
#include <iostream>
#include <queue>
#include <string>
using namespace std;

struct Student {
    string name;
    int score;

    // 重载 < 运算符
    // 注意：priority_queue 默认是大顶堆（用 less）
    // less 内部比较 a < b，当返回 true 时，a 的优先级低于 b
    // 所以 score 小的 < score 大的 → score 大的在堆顶 → 按成绩从高到低
    bool operator<(const Student& other) const {
        return this->score < other.score;
    }
};

int main() {
    priority_queue<Student> pq;

    pq.push({"Alice", 90});
    pq.push({"Bob", 85});
    pq.push({"Charlie", 95});
    pq.push({"Diana", 88});
    pq.push({"Eve", 92});

    cout << "按成绩从高到低出队（大顶堆）：" << endl;
    while (!pq.empty()) {
        cout << "  " << pq.top().name
             << " - " << pq.top().score << "分" << endl;
        pq.pop();
    }

    return 0;
}
```

**输出：**
```
按成绩从高到低出队（大顶堆）：
  Charlie - 95分
  Eve - 92分
  Alice - 90分
  Diana - 88分
  Bob - 85分
```

> **技巧：** 如果想让成绩低的先出队（小顶堆效果），只需把 `<` 的逻辑反过来：
> ```cpp
> bool operator<(const Student& other) const {
>  return this->score > other.score;  // 注意这里是 >
> }
> ```
> 这样 score 大的反而"更小"，会沉到堆底，score 小的在堆顶。

### 4.3 方法二：自定义比较器——仿函数（Functor）

仿函数（函数对象）是一个**重载了 `operator()` 的类/结构体**。这是最通用、最灵活的方式：

```cpp name=functor_comparator.cpp
#include <iostream>
#include <queue>
#include <string>
#include <vector>
using namespace std;

struct Student {
    string name;
    int score;
    int age;
};

// ============ 仿函数：按成绩从高到低（大顶堆） ============
struct CmpByScoreDesc {
    // 返回 true 表示 a 的优先级 < b 的优先级（a 排在 b 后面）
    bool operator()(const Student& a, const Student& b) const {
        return a.score < b.score;   // score 小的优先级低 → score 大的在堆顶
    }
};

// ============ 仿函数：按成绩从低到高（小顶堆） ============
struct CmpByScoreAsc {
    bool operator()(const Student& a, const Student& b) const {
        return a.score > b.score;   // score 大的优先级低 → score 小的在堆顶
    }
};

// ============ 仿函数：多关键字排序 ============
// 先按成绩从高到低；成绩相同按年龄从小到大
struct CmpMultiKey {
    bool operator()(const Student& a, const Student& b) const {
        if (a.score != b.score) {
            return a.score < b.score;    // 成绩高的优先
        }
        return a.age > b.age;            // 成绩相同，年龄小的优先
    }
};

int main() {
    vector<Student> students = {
        {"Alice",   90, 20},
        {"Bob",     85, 22},
        {"Charlie", 95, 21},
        {"Diana",   90, 19},
        {"Eve",     92, 23}
    };

    // ========== 按成绩从高到低 ==========
    cout << "===== 按成绩从高到低（大顶堆） =====" << endl;
    priority_queue<Student, vector<Student>, CmpByScoreDesc> pq1;
    for (const auto& s : students) {
        pq1.push(s);
    }
    while (!pq1.empty()) {
        auto& s = pq1.top();
        cout << "  " << s.name << " 成绩:" << s.score << " 年龄:" << s.age << endl;
        pq1.pop();
    }

    // ========== 按成绩从低到高 ==========
    cout << "\n===== 按成绩从低到高（小顶堆） =====" << endl;
    priority_queue<Student, vector<Student>, CmpByScoreAsc> pq2;
    for (const auto& s : students) {
        pq2.push(s);
    }
    while (!pq2.empty()) {
        auto& s = pq2.top();
        cout << "  " << s.name << " 成绩:" << s.score << " 年龄:" << s.age << endl;
        pq2.pop();
    }

    // ========== 多关键字排序 ==========
    cout << "\n===== 多关键字：成绩降序，同分按年龄升序 =====" << endl;
    priority_queue<Student, vector<Student>, CmpMultiKey> pq3;
    for (const auto& s : students) {
        pq3.push(s);
    }
    while (!pq3.empty()) {
        auto& s = pq3.top();
        cout << "  " << s.name << " 成绩:" << s.score << " 年龄:" << s.age << endl;
        pq3.pop();
    }

    return 0;
}
```

**输出：**
```
===== 按成绩从高到低（大顶堆） =====
  Charlie 成绩:95 年龄:21
  Eve 成绩:92 年龄:23
  Alice 成绩:90 年龄:20
  Diana 成绩:90 年龄:19
  Bob 成绩:85 年龄:22

===== 按成绩从低到高（小顶堆） =====
  Bob 成绩:85 年龄:22
  Alice 成绩:90 年龄:20
  Diana 成绩:90 年龄:19
  Eve 成绩:92 年龄:23
  Charlie 成绩:95 年龄:21

===== 多关键字：成绩降序，同分按年龄升序 =====
  Charlie 成绩:95 年龄:21
  Eve 成绩:92 年龄:23
  Diana 成绩:90 年龄:19
  Alice 成绩:90 年龄:20
  Bob 成绩:85 年龄:22
```

> 注意最后一组中，Diana（90分, 19岁）排在 Alice（90分, 20岁）前面，因为同分时年龄小的优先。

### 4.4 方法三：使用 Lambda 表达式（C++11）

C++11 起可以用 lambda 表达式代替仿函数，写起来更简洁：

```cpp name=lambda_comparator.cpp
#include <iostream>
#include <queue>
#include <string>
#include <vector>
#include <functional>
using namespace std;

struct Student {
    string name;
    int score;
};

int main() {
    // 定义 lambda 比较器
    auto cmp = [](const Student& a, const Student& b) {
        return a.score < b.score;   // 大顶堆：score 大的在堆顶
    };

    // 注意：lambda 的类型是编译器生成的唯一类型
    // 所以必须用 decltype(cmp) 来获取类型，并把 cmp 传入构造函数
    priority_queue<Student, vector<Student>, decltype(cmp)> pq(cmp);

    pq.push({"Alice", 90});
    pq.push({"Bob", 85});
    pq.push({"Charlie", 95});

    cout << "使用 lambda 比较器：" << endl;
    while (!pq.empty()) {
        cout << "  " << pq.top().name
             << " - " << pq.top().score << "分" << endl;
        pq.pop();
    }

    return 0;
}
```

**输出：**
```
使用 lambda 比较器：
  Charlie - 95分
  Alice - 90分
  Bob - 85分
```

> **注意事项：**
> - lambda 的类型通过 `decltype(cmp)` 获取
> - 构造 priority_queue 时，必须将 lambda 对象 `cmp` 作为构造函数参数传入
> - 如果忘记传入 `cmp`，某些编译器会报错（因为 lambda 类型没有默认构造函数）

### 4.5 方法四：使用 `std::function` 包装（C++11）

如果不想用 `decltype`，也可以用 `std::function` 做类型擦除：

```cpp name=std_function_comparator.cpp
#include <iostream>
#include <queue>
#include <string>
#include <functional>
using namespace std;

struct Student {
    string name;
    int score;
};

int main() {
    // 用 std::function 指定比较器类型
    using Cmp = function<bool(const Student&, const Student&)>;

    priority_queue<Student, vector<Student>, Cmp> pq(
        [](const Student& a, const Student& b) {
            return a.score < b.score;
        }
    );

    pq.push({"Alice", 90});
    pq.push({"Bob", 85});
    pq.push({"Charlie", 95});

    while (!pq.empty()) {
        cout << pq.top().name << " - " << pq.top().score << endl;
        pq.pop();
    }

    return 0;
}
```

> 这种方式代码更直观，但 `std::function` 有微小的性能开销（类型擦除的代价），一般场景下可以忽略不计。

### 4.6 四种方法对比总结

| 方法             | 适用场景                     | 优点                               | 缺点                                         |
| ---------------- | ---------------------------- | ---------------------------------- | -------------------------------------------- |
| 重载 `operator<` | 只需大顶堆，且排序规则固定   | 最简洁，声明时不用写比较器         | 只能定义一种排序规则；改成小顶堆需要反转逻辑 |
| 仿函数（推荐 ✅） | 需要多种排序规则，或团队协作 | 可复用，可定义多个仿函数；命名清晰 | 代码略多                                     |
| Lambda           | 排序规则只用一次，追求简洁   | 写法简洁，就地定义                 | 需要 `decltype`；不可复用                    |
| `std::function`  | 比较器需要运行时确定         | 灵活，类型统一                     | 有微小性能开销                               |

### 4.7 综合实战：任务调度器

下面是一个贴近真实场景的完整示例——模拟一个简单的任务调度器：

```cpp name=task_scheduler.cpp
#include <iostream>
#include <queue>
#include <string>
#include <vector>
using namespace std;

struct Task {
    string name;        // 任务名称
    int priority;       // 优先级（数字越大越紧急）
    int createdTime;    // 创建时间（模拟时间戳）

    // 构造函数
    Task(string n, int p, int t) : name(n), priority(p), createdTime(t) {}
};

// 仿函数：优先级高的先执行；优先级相同，创建时间早的先执行
struct TaskComparator {
    bool operator()(const Task& a, const Task& b) const {
        if (a.priority != b.priority) {
            return a.priority < b.priority;     // 优先级小的排后面
        }
        return a.createdTime > b.createdTime;   // 同优先级，创建晚的排后面
    }
};

int main() {
    priority_queue<Task, vector<Task>, TaskComparator> scheduler;

    // 模拟提交任务
    scheduler.emplace("编译代码",      3, 100);
    scheduler.emplace("修复崩溃Bug",   5, 95);
    scheduler.emplace("写单元测试",    2, 80);
    scheduler.emplace("代码审查",      3, 90);
    scheduler.emplace("紧急上线",      5, 98);
    scheduler.emplace("优化性能",      4, 110);
    scheduler.emplace("更新文档",      1, 70);

    cout << "====== 任务调度顺序 ======" << endl;
    cout << "（优先级高的先执行，同级按创建时间先后）" << endl;
    cout << endl;

    int order = 1;
    while (!scheduler.empty()) {
        const Task& t = scheduler.top();
        cout << "第" << order++ << "个执行：["
             << "优先级:" << t.priority
             << " 时间:" << t.createdTime
             << "] " << t.name << endl;
        scheduler.pop();
    }

    return 0;
}
```

**输出：**
```
====== 任务调度顺序 ======
（优先级高的先执行，同级按创建时间先后）

第1个执行：[优先级:5 时间:95] 修复崩溃Bug
第2个执行：[优先级:5 时间:98] 紧急上线
第3个执行：[优先级:4 时间:110] 优化性能
第4个执行：[优先级:3 时间:90] 代码审查
第5个执行：[优先级:3 时间:100] 编译代码
第6个执行：[优先级:2 时间:80] 写单元测试
第7个执行：[优先级:1 时间:70] 更新文档
```

> 可以看到：
> - "修复崩溃Bug"和"紧急上线"都是优先级5，但"修复崩溃Bug"创建时间更早（95 < 98），所以先执行
> - "代码审查"和"编译代码"都是优先级3，"代码审查"创建更早（90 < 100），先执行

---

## 附录：priority_queue 知识速查卡

```
┌─────────────────────────────────────────────────────────────────┐
│                    priority_queue 速查卡                         │
├─────────────────────────────────────────────────────────────────┤
│  头文件：  #include <queue>                                      │
│  本质：    容器适配器，底层默认 vector + 二叉堆                      │
│  默认：    大顶堆（less<T>），top() 返回最大值                      │
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 大顶堆：priority_queue<int> pq;                             │ │
│  │ 小顶堆：priority_queue<int, vector<int>, greater<int>> pq;  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  常用接口：                                                       │
│    pq.push(x)    插入元素          O(log n)                      │
│    pq.emplace(x) 原地构造插入      O(log n)                       │
│    pq.pop()      移除堆顶          O(log n)                      │
│    pq.top()      访问堆顶          O(1)                          │
│    pq.size()     元素个数          O(1)                          │
│    pq.empty()    是否为空          O(1)                          │
│    pq.swap(pq2)  交换内容          O(1)                          │
│                                                                  │
│  自定义类型比较（选其一）：                                         │
│    ① 重载 operator<（大顶堆最简单）                                │
│    ② 仿函数 struct Cmp { bool operator()(...) }（最通用 ✅）      │
│    ③ lambda + decltype（简洁）                                    │
│    ④ std::function（灵活）                                        │
│                                                                  │
│  ⚠️ 注意事项：                                                    │
│    • 无迭代器，不可遍历                                            │
│    • 无 clear()，用 swap 空对象清空                                │
│    • 空堆调 top()/pop() 是未定义行为                               │
│    • less → 大顶堆，greater → 小顶堆（别搞反！）                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 附录：比较器逻辑记忆口诀

初学者最容易搞混的就是**比较器返回 `true` 到底意味着什么**。这里给出一个万能口诀：

> **"返回 true 的那个参数，优先级更低，排在更后面。"**

| 比较器写法      | 含义                      | 效果                               |
| --------------- | ------------------------- | ---------------------------------- |
| `return a < b;` | a 比 b 小时，a 优先级更低 | b 排前面 → **大的在堆顶** → 大顶堆 |
| `return a > b;` | a 比 b 大时，a 优先级更低 | b 排前面 → **小的在堆顶** → 小顶堆 |

记住这个口诀，无论多复杂的自定义比较器，都不会写错方向。

---

以上就是 `priority_queue` 容器的完整教程。从基本概念到底层原理，从简单的内置类型到复杂的自定义类型比较，再到实战级的任务调度器案例，覆盖了日常开发和考试中可能遇到的所有场景。建议将每个示例代码都亲手敲一遍、运行一遍，加深理解！