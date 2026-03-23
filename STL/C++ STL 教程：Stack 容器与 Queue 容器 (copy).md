# C++ STL 教程：Stack 容器与 Queue 容器

------

## 第一章：Stack 容器 — 基本概念

------

### 1.1 什么是栈（Stack）？

**栈（Stack）** 是一种 **后进先出**（LIFO，Last In First Out）的数据结构。

你可以把栈想象成一摞盘子：

- 你只能在**最顶部**放盘子（压栈 / 入栈）
- 你只能从**最顶部**取盘子（弹栈 / 出栈）
- 你不能直接抽取中间或底部的盘子

```
    ┌───────┐
    │   3   │  ← 栈顶（top）：最后放入，最先取出
    ├───────┤
    │   2   │
    ├───────┤
    │   1   │  ← 栈底（bottom）：最先放入，最后取出
    └───────┘
```

**核心特征：**

> **只有栈顶元素才能被外界访问到，因此栈不允许有遍历行为。**

这是栈最重要的性质——你 **不能** 像 vector、list 那样用迭代器去遍历栈中的所有元素。栈只暴露一个"窗口"，就是栈顶。

------

### 1.2 生活中的栈模型

| 生活场景                 | 对应栈操作                     |
| ------------------------ | ------------------------------ |
| 一摞盘子，只能从顶部取放 | 入栈 / 出栈                    |
| 浏览器的"后退"按钮       | 每访问一个页面压栈，后退时弹栈 |
| 撤销操作（Ctrl+Z）       | 每次操作压栈，撤销时弹栈       |
| 弹夹中的子弹             | 最后压入的子弹最先射出         |

------

### 1.3 栈的入栈与出栈过程图解

假设我们依次将 `1`、`2`、`3` 压入栈中，再依次弹出：

**入栈过程（push）：**

```
步骤1: push(1)       步骤2: push(2)       步骤3: push(3)
​
  ┌───┐                ┌───┐                ┌───┐
  │ 1 │  ← top         │ 2 │  ← top         │ 3 │  ← top
  └───┘                ├───┤                ├───┤
                       │ 1 │                │ 2 │
                       └───┘                ├───┤
                                            │ 1 │
                                            └───┘
```

**出栈过程（pop）：**

```
步骤4: pop()          步骤5: pop()          步骤6: pop()
弹出 3                 弹出 2                 弹出 1
​
  ┌───┐                ┌───┐
  │ 2 │  ← top         │ 1 │  ← top         （空栈）
  ├───┤                └───┘
  │ 1 │
  └───┘
```

> **注意：** 出栈顺序为 `3 → 2 → 1`，与入栈顺序 `1 → 2 → 3` 恰好相反。这就是 LIFO。

------

### 1.4 C++ STL 中的 stack

在 C++ 标准模板库（STL）中，`stack` 是一个 **容器适配器（Container Adaptor）**。

**什么是容器适配器？**

容器适配器并不是一个全新的底层数据结构，而是对已有容器（如 `deque`、`vector`、`list`）进行 **包装和限制**，使其表现出特定的行为。

`stack` 默认使用 `deque`（双端队列）作为底层容器，但你也可以指定其他容器：

```cpp name=stack_underlying.cpp
#include <stack>
#include <vector>
#include <list>
​
// 默认底层容器为 deque
std::stack<int> s1;
​
// 指定底层容器为 vector
std::stack<int, std::vector<int>> s2;
​
// 指定底层容器为 list
std::stack<int, std::list<int>> s3;
```

> 初学阶段只需使用默认的 `std::stack<int> s;` 即可，无需关心底层容器选择。

------

### 1.5 使用 stack 需要包含的头文件

```cpp name=stack_header.cpp
#include <stack>
```

------

### 1.6 stack 不允许遍历！

这一点必须反复强调：

```
✗ stack 没有迭代器（iterator）
✗ stack 不能用 for 循环遍历
✗ stack 不能用范围 for（range-based for）
✗ stack 不能随机访问（不能用下标 []）
✗ stack 只能访问栈顶元素
```

如果你需要遍历，说明你应该用 `vector`、`deque` 等其他容器，而不是 `stack`。

------

------

## 第二章：Stack 容器 — 常用接口

------

### 2.1 接口总览

| 函数         | 功能说明            | 时间复杂度 |
| ------------ | ------------------- | ---------- |
| `push(elem)` | 向栈顶添加元素 elem | O(1)       |
| `pop()`      | 从栈顶移除一个元素  | O(1)       |
| `top()`      | 返回栈顶元素的引用  | O(1)       |
| `empty()`    | 判断栈是否为空      | O(1)       |
| `size()`     | 返回栈中元素个数    | O(1)       |

此外，stack 还支持：

| 操作                      | 功能说明                    |
| ------------------------- | --------------------------- |
| `stack<T> s;`             | 默认构造函数，创建空栈      |
| `stack(const stack &stk)` | 拷贝构造函数                |
| `operator=`               | 赋值运算符重载              |
| `emplace(args...)`        | 原地构造元素并压栈（C++11） |
| `swap(stack &stk)`        | 交换两个栈的内容            |

------

### 2.2 构造函数

```cpp name=stack_constructor.cpp
#include <iostream>
#include <stack>
using namespace std;
​
int main() {
​
    // 1. 默认构造：创建一个空栈
    stack<int> s1;
​
    // 2. 先压入一些元素
    s1.push(10);
    s1.push(20);
    s1.push(30);
​
    // 3. 拷贝构造：用已有栈创建新栈
    stack<int> s2(s1);  // s2 是 s1 的一份完整拷贝
​
    // 4. 赋值操作
    stack<int> s3;
    s3 = s1;  // s3 的内容与 s1 完全一致
​
    cout << "s1 栈顶: " << s1.top() << endl;  // 30
    cout << "s2 栈顶: " << s2.top() << endl;  // 30
    cout << "s3 栈顶: " << s3.top() << endl;  // 30
​
    return 0;
}
```

**输出：**

```
s1 栈顶: 30
s2 栈顶: 30
s3 栈顶: 30
```

------

### 2.3 push() — 入栈

`push()` 将一个元素添加到栈顶。

```cpp name=stack_push.cpp
#include <iostream>
#include <stack>
using namespace std;
​
int main() {
​
    stack<int> s;
​
    // 依次压入 10, 20, 30
    s.push(10);
    s.push(20);
    s.push(30);
​
    // 此时栈的状态：
    //   ┌────┐
    //   │ 30 │  ← top
    //   ├────┤
    //   │ 20 │
    //   ├────┤
    //   │ 10 │
    //   └────┘
​
    cout << "栈顶元素: " << s.top() << endl;   // 30
    cout << "栈的大小: " << s.size() << endl;   // 3
​
    return 0;
}
```

------

### 2.4 pop() — 出栈

`pop()` 移除栈顶元素。

> **注意：** `pop()` **没有返回值**（返回 `void`）。如果你需要获取栈顶元素的值，必须在 `pop()` 之前先调用 `top()`。

```cpp name=stack_pop.cpp
#include <iostream>
#include <stack>
using namespace std;
​
int main() {
​
    stack<int> s;
    s.push(10);
    s.push(20);
    s.push(30);
​
    cout << "弹出前栈顶: " << s.top() << endl;  // 30
​
    s.pop();  // 弹出 30
​
    cout << "弹出后栈顶: " << s.top() << endl;  // 20
    cout << "栈的大小: " << s.size() << endl;    // 2
​
    return 0;
}
```

**⚠️ 常见错误：对空栈调用 pop() 或 top()**

```cpp name=stack_error.cpp
stack<int> s;
// s.pop();   // ❌ 未定义行为！空栈不能 pop
// s.top();   // ❌ 未定义行为！空栈不能 top
​
// ✅ 正确做法：先判断是否为空
if (!s.empty()) {
    cout << s.top() << endl;
    s.pop();
}
```

------

### 2.5 top() — 访问栈顶元素

`top()` 返回栈顶元素的**引用**，意味着你不仅可以读取，还可以修改它。

```cpp name=stack_top.cpp
#include <iostream>
#include <stack>
using namespace std;
​
int main() {
​
    stack<int> s;
    s.push(10);
    s.push(20);
​
    // 读取栈顶
    cout << "栈顶: " << s.top() << endl;  // 20
​
    // 修改栈顶（因为返回的是引用）
    s.top() = 99;
    cout << "修改后栈顶: " << s.top() << endl;  // 99
​
    return 0;
}
```

------

### 2.6 empty() 和 size()

```cpp name=stack_empty_size.cpp
#include <iostream>
#include <stack>
using namespace std;
​
int main() {
​
    stack<int> s;
​
    // 空栈检测
    if (s.empty()) {
        cout << "栈为空" << endl;  // ✔ 输出
    }
    cout << "大小: " << s.size() << endl;  // 0
​
    s.push(100);
    s.push(200);
​
    if (!s.empty()) {
        cout << "栈不为空" << endl;  // ✔ 输出
    }
    cout << "大小: " << s.size() << endl;  // 2
​
    return 0;
}
```

------

### 2.7 ⭐ 如何"遍历"栈中所有元素

虽然栈不支持迭代器遍历，但我们可以用 **循环弹出** 的方式依次访问并输出所有元素。

> **注意：这种方式会清空栈！** 如果需要保留原始数据，请先拷贝一份。

```cpp name=stack_traverse.cpp
#include <iostream>
#include <stack>
using namespace std;
​
int main() {
​
    stack<int> s;
    s.push(10);
    s.push(20);
    s.push(30);
    s.push(40);
​
    // 方式：循环弹出，直到栈为空
    cout << "栈中元素（从栈顶到栈底）: ";
    while (!s.empty()) {
        cout << s.top() << " ";  // 先访问栈顶
        s.pop();                  // 再弹出
    }
    cout << endl;
    // 输出: 40 30 20 10
​
    cout << "遍历后栈的大小: " << s.size() << endl;  // 0，栈已被清空
​
    return 0;
}
```

**如果不想破坏原栈，可以先拷贝：**

```cpp name=stack_traverse_safe.cpp
#include <iostream>
#include <stack>
using namespace std;
​
void printStack(stack<int> s) {  // 注意：参数是【值传递】，不影响原栈
    while (!s.empty()) {
        cout << s.top() << " ";
        s.pop();
    }
    cout << endl;
}
​
int main() {
​
    stack<int> s;
    s.push(10);
    s.push(20);
    s.push(30);
​
    printStack(s);  // 输出: 30 20 10
​
    // 原栈没有被破坏
    cout << "原栈大小: " << s.size() << endl;  // 3
    cout << "原栈顶: " << s.top() << endl;     // 30
​
    return 0;
}
```

> **技巧：** 把 `stack` 以**值传递**的方式传入函数，函数内部操作的是副本，原栈不受影响。

------

### 2.8 emplace() — 原地构造（C++11）

`emplace()` 直接在栈顶**原地构造**元素，避免先构造再拷贝的开销，对于复杂对象效率更高。

```cpp name=stack_emplace.cpp
#include <iostream>
#include <stack>
#include <string>
using namespace std;
​
class Person {
public:
    string name;
    int age;
    Person(string n, int a) : name(n), age(a) {
        cout << "构造: " << name << endl;
    }
};
​
int main() {
​
    stack<Person> s;
​
    // push 方式：先构造临时对象，再拷贝/移动到栈中
    s.push(Person("张三", 25));
​
    // emplace 方式：直接在栈中原地构造，更高效
    s.emplace("李四", 30);
​
    cout << "栈顶: " << s.top().name << ", " << s.top().age << endl;
    // 输出: 栈顶: 李四, 30
​
    return 0;
}
```

------

### 2.9 swap() — 交换两个栈

```cpp name=stack_swap.cpp
#include <iostream>
#include <stack>
using namespace std;
​
int main() {
​
    stack<int> s1, s2;
    s1.push(1);
    s1.push(2);
​
    s2.push(100);
    s2.push(200);
    s2.push(300);
​
    cout << "交换前:" << endl;
    cout << "s1 大小: " << s1.size() << ", 栈顶: " << s1.top() << endl;  // 2, 2
    cout << "s2 大小: " << s2.size() << ", 栈顶: " << s2.top() << endl;  // 3, 300
​
    s1.swap(s2);  // 交换
​
    cout << "交换后:" << endl;
    cout << "s1 大小: " << s1.size() << ", 栈顶: " << s1.top() << endl;  // 3, 300
    cout << "s2 大小: " << s2.size() << ", 栈顶: " << s2.top() << endl;  // 2, 2
​
    return 0;
}
```

------

### 2.10 综合案例：用栈判断括号是否匹配

这是栈最经典的应用场景之一：

```cpp name=stack_bracket_matching.cpp
#include <iostream>
#include <stack>
#include <string>
using namespace std;
​
bool isValid(const string& s) {
    stack<char> stk;
​
    for (char c : s) {
        // 遇到左括号，压入对应的右括号
        if (c == '(')  stk.push(')');
        else if (c == '[')  stk.push(']');
        else if (c == '{')  stk.push('}');
        else {
            // 遇到右括号
            // 如果栈为空，或者栈顶不匹配，返回 false
            if (stk.empty() || stk.top() != c) {
                return false;
            }
            stk.pop();  // 匹配成功，弹出
        }
    }
​
    // 最终栈为空说明全部匹配
    return stk.empty();
}
​
int main() {
​
    cout << isValid("()[]{}") << endl;   // 1 (true)
    cout << isValid("([{}])") << endl;   // 1 (true)
    cout << isValid("(]") << endl;       // 0 (false)
    cout << isValid("([)]") << endl;     // 0 (false)
    cout << isValid("{[]}") << endl;     // 1 (true)
​
    return 0;
}
```

------

------

## 第三章：Queue 容器 — 基本概念

------

### 3.1 什么是队列（Queue）？

**队列（Queue）** 是一种 **先进先出**（FIFO，First In First Out）的数据结构。

你可以把队列想象成排队买东西：

- 新来的人从**队尾**排队（入队）
- 最先排队的人从**队头**离开（出队）
- 不能插队！

```
  入队 (push)                              出队 (pop)
    ──→   ┌───┬───┬───┬───┬───┐   ──→
          │ 5 │ 4 │ 3 │ 2 │ 1 │
    ──→   └───┴───┴───┴───┴───┘   ──→
          队尾                队头
         (back)             (front)
```

**核心特征：**

> **队列只允许从队尾入队，从队头出队。只能访问队头和队尾元素，不允许遍历。**

------

### 3.2 Stack 与 Queue 的对比

| 特性         | Stack（栈）      | Queue（队列）                  |
| ------------ | ---------------- | ------------------------------ |
| 原则         | 后进先出（LIFO） | 先进先出（FIFO）               |
| 入口         | 栈顶（一端）     | 队尾（一端）                   |
| 出口         | 栈顶（同一端）   | 队头（另一端）                 |
| 可访问元素   | 只有栈顶 `top()` | 队头 `front()` + 队尾 `back()` |
| 是否有迭代器 | ✗ 没有           | ✗ 没有                         |
| 是否可遍历   | ✗ 不可以         | ✗ 不可以                       |

------

### 3.3 生活中的队列模型

| 生活场景       | 对应队列操作                   |
| -------------- | ------------------------------ |
| 排队买票       | 先到先得，从队尾加入，队头离开 |
| 打印机任务队列 | 先提交的打印任务先执行         |
| 消息队列       | 先发的消息先被处理             |
| 客服电话等待   | 先打电话的客户先接通           |

------

### 3.4 队列的入队与出队过程图解

假设我们依次将 `1`、`2`、`3` 入队，再依次出队：

**入队过程（push）：**

```
步骤1: push(1)
​
   队头 → ┌───┐ ← 队尾
         │ 1 │
         └───┘
​
步骤2: push(2)
​
  队头 → ┌───┬───┐ ← 队尾
         │ 1 │ 2 │
         └───┴───┘
​
步骤3: push(3)
​
  队头 → ┌───┬───┬───┐ ← 队尾
         │ 1 │ 2 │ 3 │
         └───┴───┴───┘
```

**出队过程（pop）：**

```
步骤4: pop()   → 弹出 1
​
   队头 → ┌───┬───┐ ← 队尾
         │ 2 │ 3 │
         └───┴───┘
​
步骤5: pop()   → 弹出 2
​
   队头 → ┌───┐ ← 队尾
         │ 3 │
         └───┘
​
步骤6: pop()   → 弹出 3
​
         （空队列）
```

> **注意：** 出队顺序为 `1 → 2 → 3`，与入队顺序 `1 → 2 → 3` **完全相同**。这就是 FIFO。

------

### 3.5 C++ STL 中的 queue

与 `stack` 类似，`queue` 也是一个 **容器适配器（Container Adaptor）**。

- 默认底层容器：`deque`
- 也可以使用 `list` 作为底层容器
- **不能** 使用 `vector`（因为 vector 不支持高效的头部删除操作 `pop_front()`）

```cpp name=queue_underlying.cpp
#include <queue>
#include <list>
​
// 默认底层容器为 deque
std::queue<int> q1;
​
// 指定底层容器为 list
std::queue<int, std::list<int>> q2;
​
// ❌ 不能使用 vector，因为 vector 没有 pop_front()
// std::queue<int, std::vector<int>> q3;  // 编译错误！
```

------

### 3.6 使用 queue 需要包含的头文件

```cpp name=queue_header.cpp
#include <queue>
```

------

### 3.7 queue 同样不允许遍历！

```
✗ queue 没有迭代器（iterator）
✗ queue 不能用 for 循环遍历
✗ queue 不能用范围 for（range-based for）
✗ queue 不能随机访问（不能用下标 []）
✓ queue 只能访问队头元素 front() 和队尾元素 back()
```

------

------

## 第四章：Queue 容器 — 常用接口

------

### 4.1 接口总览

| 函数         | 功能说明            | 时间复杂度 |
| ------------ | ------------------- | ---------- |
| `push(elem)` | 向队尾添加元素 elem | O(1)       |
| `pop()`      | 从队头移除一个元素  | O(1)       |
| `front()`    | 返回队头元素的引用  | O(1)       |
| `back()`     | 返回队尾元素的引用  | O(1)       |
| `empty()`    | 判断队列是否为空    | O(1)       |
| `size()`     | 返回队列中元素个数  | O(1)       |

此外：

| 操作                      | 功能说明                    |
| ------------------------- | --------------------------- |
| `queue<T> q;`             | 默认构造函数，创建空队列    |
| `queue(const queue &que)` | 拷贝构造函数                |
| `operator=`               | 赋值运算符重载              |
| `emplace(args...)`        | 原地构造元素并入队（C++11） |
| `swap(queue &que)`        | 交换两个队列的内容          |

------

### 4.2 构造函数

```cpp name=queue_constructor.cpp
#include <iostream>
#include <queue>
using namespace std;
​
int main() {
​
    // 1. 默认构造
    queue<int> q1;
​
    q1.push(10);
    q1.push(20);
    q1.push(30);
​
    // 2. 拷贝构造
    queue<int> q2(q1);
​
    // 3. 赋值操作
    queue<int> q3;
    q3 = q1;
​
    cout << "q1 队头: " << q1.front() << ", 队尾: " << q1.back() << endl;  // 10, 30
    cout << "q2 队头: " << q2.front() << ", 队尾: " << q2.back() << endl;  // 10, 30
    cout << "q3 队头: " << q3.front() << ", 队尾: " << q3.back() << endl;  // 10, 30
​
    return 0;
}
```

------

### 4.3 push() — 入队

`push()` 将一个元素添加到队尾。

```cpp name=queue_push.cpp
#include <iostream>
#include <queue>
using namespace std;
​
int main() {
​
    queue<int> q;
​
    q.push(10);   // 队列: [10]          front=10, back=10
    q.push(20);   // 队列: [10, 20]      front=10, back=20
    q.push(30);   // 队列: [10, 20, 30]  front=10, back=30
​
    cout << "队头: " << q.front() << endl;  // 10
    cout << "队尾: " << q.back() << endl;   // 30
    cout << "大小: " << q.size() << endl;   // 3
​
    return 0;
}
```

------

### 4.4 pop() — 出队

`pop()` 移除队头元素。

> **注意：** 与 stack 一样，`pop()` **没有返回值**（返回 `void`）。需要先用 `front()` 获取队头的值。

```cpp name=queue_pop.cpp
#include <iostream>
#include <queue>
using namespace std;
​
int main() {
​
    queue<int> q;
    q.push(10);
    q.push(20);
    q.push(30);
​
    cout << "出队前 → 队头: " << q.front() << endl;  // 10
​
    q.pop();  // 弹出 10
​
    cout << "出队后 → 队头: " << q.front() << endl;  // 20
    cout << "大小: " << q.size() << endl;             // 2
​
    return 0;
}
```

**⚠️ 同样要注意空队列的安全检查：**

```cpp name=queue_safety.cpp
queue<int> q;
// q.pop();     // ❌ 未定义行为！
// q.front();   // ❌ 未定义行为！
// q.back();    // ❌ 未定义行为！
​
// ✅ 正确做法
if (!q.empty()) {
    cout << q.front() << endl;
    q.pop();
}
```

------

### 4.5 front() 和 back() — 访问队头和队尾

这是 queue 与 stack 的一个重要区别：**queue 可以访问两端的元素**（队头和队尾），而 stack 只能访问栈顶。

```cpp name=queue_front_back.cpp
#include <iostream>
#include <queue>
using namespace std;
​
int main() {
​
    queue<int> q;
    q.push(100);
    q.push(200);
    q.push(300);
​
    // 访问队头和队尾
    cout << "队头 (front): " << q.front() << endl;  // 100（最先入队的）
    cout << "队尾 (back):  " << q.back() << endl;   // 300（最后入队的）
​
    // 修改队头和队尾（返回的是引用）
    q.front() = 999;
    q.back()  = 888;
​
    cout << "修改后队头: " << q.front() << endl;  // 999
    cout << "修改后队尾: " << q.back() << endl;   // 888
​
    return 0;
}
```

------

### 4.6 empty() 和 size()

```cpp name=queue_empty_size.cpp
#include <iostream>
#include <queue>
using namespace std;
​
int main() {
​
    queue<int> q;
​
    cout << "是否为空: " << (q.empty() ? "是" : "否") << endl;  // 是
    cout << "大小: " << q.size() << endl;                        // 0
​
    q.push(1);
    q.push(2);
    q.push(3);
​
    cout << "是否为空: " << (q.empty() ? "是" : "否") << endl;  // 否
    cout << "大小: " << q.size() << endl;                        // 3
​
    return 0;
}
```

------

### 4.7 ⭐ 如何"遍历"队列中所有元素

与栈类似，需要循环弹出。注意会清空队列。

```cpp name=queue_traverse.cpp
#include <iostream>
#include <queue>
using namespace std;
​
void printQueue(queue<int> q) {  // 值传递，不影响原队列
    while (!q.empty()) {
        cout << q.front() << " ";  // 访问队头
        q.pop();                    // 弹出队头
    }
    cout << endl;
}
​
int main() {
​
    queue<int> q;
    q.push(10);
    q.push(20);
    q.push(30);
    q.push(40);
​
    cout << "队列元素（从队头到队尾）: ";
    printQueue(q);
    // 输出: 10 20 30 40
​
    // 原队列未被破坏
    cout << "原队列大小: " << q.size() << endl;  // 4
​
    return 0;
}
```

------

### 4.8 emplace() — 原地构造（C++11）

```cpp name=queue_emplace.cpp
#include <iostream>
#include <queue>
#include <string>
using namespace std;
​
class Person {
public:
    string name;
    int age;
    Person(string n, int a) : name(n), age(a) {}
};
​
int main() {
​
    queue<Person> q;
​
    // push 方式
    q.push(Person("张三", 20));
​
    // emplace 方式：直接原地构造，更高效
    q.emplace("李四", 25);
    q.emplace("王五", 30);
​
    // 查看队头和队尾
    cout << "队头: " << q.front().name << ", " << q.front().age << endl;  // 张三, 20
    cout << "队尾: " << q.back().name << ", " << q.back().age << endl;    // 王五, 30
​
    return 0;
}
```

------

### 4.9 swap() — 交换两个队列

```cpp name=queue_swap.cpp
#include <iostream>
#include <queue>
using namespace std;
​
int main() {
​
    queue<int> q1, q2;
    q1.push(1);
    q1.push(2);
​
    q2.push(100);
    q2.push(200);
    q2.push(300);
​
    cout << "交换前:" << endl;
    cout << "q1 大小: " << q1.size() << ", 队头: " << q1.front() << endl;  // 2, 1
    cout << "q2 大小: " << q2.size() << ", 队头: " << q2.front() << endl;  // 3, 100
​
    q1.swap(q2);
​
    cout << "交换后:" << endl;
    cout << "q1 大小: " << q1.size() << ", 队头: " << q1.front() << endl;  // 3, 100
    cout << "q2 大小: " << q2.size() << ", 队头: " << q2.front() << endl;  // 2, 1
​
    return 0;
}
```

------

### 4.10 综合案例：模拟排队系统

```cpp name=queue_simulation.cpp
#include <iostream>
#include <queue>
#include <string>
using namespace std;
​
class Customer {
public:
    string name;
    int serviceTime;  // 需要的服务时间（分钟）
​
    Customer(string n, int t) : name(n), serviceTime(t) {}
};
​
int main() {
​
    queue<Customer> waitingLine;
​
    // 顾客依次排队
    waitingLine.emplace("张三", 5);
    waitingLine.emplace("李四", 3);
    waitingLine.emplace("王五", 8);
    waitingLine.emplace("赵六", 2);
​
    cout << "=== 银行窗口服务模拟 ===" << endl;
    cout << "当前排队人数: " << waitingLine.size() << endl;
    cout << endl;
​
    int totalTime = 0;
    int customerCount = 0;
​
    while (!waitingLine.empty()) {
        // 获取队头的顾客
        Customer current = waitingLine.front();
        waitingLine.pop();
​
        customerCount++;
        totalTime += current.serviceTime;
​
        cout << "第 " << customerCount << " 位顾客: " << current.name
             << "，服务时间: " << current.serviceTime << " 分钟"
             << "，剩余等待人数: " << waitingLine.size() << endl;
    }
​
    cout << endl;
    cout << "=== 服务结束 ===" << endl;
    cout << "共服务 " << customerCount << " 位顾客" << endl;
    cout << "总耗时: " << totalTime << " 分钟" << endl;
​
    return 0;
}
```

**输出：**

```
=== 银行窗口服务模拟 ===
当前排队人数: 4
​
第 1 位顾客: 张三，服务时间: 5 分钟，剩余等待人数: 3
第 2 位顾客: 李四，服务时间: 3 分钟，剩余等待人数: 2
第 3 位顾客: 王五，服务时间: 8 分钟，剩余等待人数: 1
第 4 位顾客: 赵六，服务时间: 2 分钟，剩余等待人数: 0
​
=== 服务结束 ===
共服务 4 位顾客
总耗时: 18 分钟
```

------

------

## 第五章：Stack 与 Queue 接口对比速查表

------

| 操作         | `stack<T>`              | `queue<T>`                         |
| ------------ | ----------------------- | ---------------------------------- |
| 头文件       | `#include <stack>`      | `#include <queue>`                 |
| 数据原则     | LIFO（后进先出）        | FIFO（先进先出）                   |
| 添加元素     | `push(elem)` — 压入栈顶 | `push(elem)` — 加入队尾            |
| 移除元素     | `pop()` — 弹出栈顶      | `pop()` — 弹出队头                 |
| 访问元素     | `top()` — 栈顶          | `front()` — 队头 / `back()` — 队尾 |
| 判空         | `empty()`               | `empty()`                          |
| 大小         | `size()`                | `size()`                           |
| 原地构造     | `emplace(args...)`      | `emplace(args...)`                 |
| 交换         | `swap(stk)`             | `swap(que)`                        |
| 有无迭代器   | ✗ 没有                  | ✗ 没有                             |
| 能否遍历     | ✗ 不能                  | ✗ 不能                             |
| 默认底层容器 | `deque`                 | `deque`                            |

------

## 第六章：常见问题与易错点总结

------

### ❓ Q1: 为什么 pop() 不返回被弹出的元素？

**A：** 这是 C++ STL 的设计哲学——**将"查询"和"命令"分离**（Command-Query Separation）。

- `top()` / `front()` 负责查询
- `pop()` 负责删除

这样做的好处是在异常安全性方面更好。如果 `pop()` 返回值，当拷贝构造抛出异常时，元素已经被移除但值没有成功返回，就会丢失数据。

**正确的使用模式：**

```cpp name=pop_pattern.cpp
// ✅ 先获取，再弹出
int val = s.top();  // 或 q.front()
s.pop();            // 或 q.pop()
```

------

### ❓ Q2: 对空栈/空队列调用 top()/front()/back()/pop() 会怎样？

**A：** 这是**未定义行为（Undefined Behavior）**！程序可能崩溃、输出垃圾值、或者"看起来正常"但实际已经出错。

**永远在操作前检查 empty()：**

```cpp name=safe_pattern.cpp
if (!s.empty()) {
    // 安全操作 stack
    cout << s.top();
    s.pop();
}
​
if (!q.empty()) {
    // 安全操作 queue
    cout << q.front();
    q.pop();
}
```

------

### ❓ Q3: stack 和 queue 的效率如何？

**A：** 所有核心操作（push、pop、top、front、back、empty、size）都是 **O(1) 常数时间复杂度**，非常高效。

------

### ❓ Q4: 什么时候用 stack，什么时候用 queue？

| 场景                 | 选择      |
| -------------------- | --------- |
| 需要"后处理先完成"   | **stack** |
| 撤销/回退操作        | **stack** |
| 深度优先搜索（DFS）  | **stack** |
| 表达式求值、括号匹配 | **stack** |
| 需要"先来先服务"     | **queue** |
| 广度优先搜索（BFS）  | **queue** |
| 任务调度、消息队列   | **queue** |
| 按顺序处理请求       | **queue** |

------

### ❓ Q5: 如果我需要两端都能操作怎么办？

**A：** 使用 `deque`（双端队列），它支持两端的高效插入和删除，并且支持迭代器遍历和随机访问。

------

## 附录：完整知识脑图

```
STL 容器适配器
├── stack（栈）
│   ├── 特征：LIFO（后进先出）
│   ├── 头文件：<stack>
│   ├── 默认底层：deque
│   ├── 核心操作
│   │   ├── push()    → 入栈
│   │   ├── pop()     → 出栈（无返回值）
│   │   ├── top()     → 访问栈顶（返回引用）
│   │   ├── empty()   → 判空
│   │   └── size()    → 大小
│   ├── 不支持：迭代器、遍历、随机访问
│   └── 应用：括号匹配、DFS、撤销操作、表达式求值
│
└── queue（队列）
    ├── 特征：FIFO（先进先出）
    ├── 头文件：<queue>
    ├── 默认底层：deque
    ├── 核心操作
    │   ├── push()    → 入队（队尾）
    │   ├── pop()     → 出队（队头，无返回值）
    │   ├── front()   → 访问队头（返回引用）
    │   ├── back()    → 访问队尾（返回引用）
    │   ├── empty()   → 判空
    │   └── size()    → 大小
    ├── 不支持：迭代器、遍历、随机访问
    └── 应用：BFS、任务调度、排队系统、消息队列
```

------

> **学习建议：** stack 和 queue 的接口非常少且简单，关键是理解它们的 **设计思想**（LIFO vs FIFO）以及 **适用场景**。建议动手敲一遍每个示例代码，然后尝试用 stack 实现括号匹配，用 queue 实现 BFS，加深理解。