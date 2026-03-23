# C++ STL 内建函数对象（Built-in Function Objects）

---

## 前置知识回顾

在学习内建函数对象之前，你需要理解一个核心概念——**仿函数（Functor）**。

仿函数就是重载了 `operator()` 的类/结构体，它的对象可以像函数一样被调用：

```cpp
struct MyAdd {
    int operator()(int a, int b) {
        return a + b;
    }
};

MyAdd myAdd;
cout << myAdd(10, 20) << endl; // 输出 30，看起来像在调用函数
```

STL 提供了一批**现成的仿函数**，称为**内建函数对象**，省去你自己手写的麻烦。

使用它们需要引入头文件：

```cpp
#include <functional>
```

建议至少使用 C++14 及以上标准（如 `-std=c++14` / `-std=c++17` / `-std=c++23`），这样文中的 `greater<>()` 等写法都可直接使用。

内建函数对象分为三大类：

| 类别       | 用途                   | 示例                                       |
| ---------- | ---------------------- | ------------------------------------------ |
| 算术仿函数 | 加减乘除、取模、取反   | `plus`, `minus`, `multiplies`...           |
| 关系仿函数 | 大于、小于、等于等比较 | `greater`, `less`, `equal_to`...           |
| 逻辑仿函数 | 与、或、非             | `logical_and`, `logical_or`, `logical_not` |

---

## 一、算术仿函数（Arithmetic Functors）

算术仿函数用于执行基本的数学运算。

### 1.1 完整清单

| 仿函数          | 功能 | 参数数量             | 等价表达式 |
| --------------- | ---- | -------------------- | ---------- |
| `plus<T>`       | 加法 | 二元（两个参数）     | `a + b`    |
| `minus<T>`      | 减法 | 二元                 | `a - b`    |
| `multiplies<T>` | 乘法 | 二元                 | `a * b`    |
| `divides<T>`    | 除法 | 二元                 | `a / b`    |
| `modulus<T>`    | 取模 | 二元                 | `a % b`    |
| `negate<T>`     | 取反 | **一元**（一个参数） | `-a`       |

> 注意：前五个都是**二元仿函数**（接收两个参数），只有 `negate` 是**一元仿函数**（接收一个参数）。
>
> 再提醒两个初学者高频坑：
> - `divides<int>()(a, b)` 是**整数除法**（会截断小数部分）
> - 除数不能为 0，否则是未定义行为
> - `modulus<T>` 通常用于整数类型；对浮点数不能直接使用 `%`

### 1.2 基础用法

```cpp
#include <iostream>
#include <functional>
using namespace std;

int main() {
    // --- 二元算术仿函数 ---
    plus<int> myPlus;
    cout << "10 + 20 = " << myPlus(10, 20) << endl;       // 30

    minus<int> myMinus;
    cout << "10 - 20 = " << myMinus(10, 20) << endl;      // -10

    multiplies<int> myMul;
    cout << "10 * 20 = " << myMul(10, 20) << endl;        // 200

    divides<int> myDiv;
    cout << "20 / 10 = " << myDiv(20, 10) << endl;        // 2

    modulus<int> myMod;
    cout << "10 % 3  = " << myMod(10, 3) << endl;         // 1

    // --- 一元算术仿函数 ---
    negate<int> myNeg;
    cout << "negate(50)  = " << myNeg(50) << endl;         // -50
    cout << "negate(-50) = " << myNeg(-50) << endl;        // 50

    return 0;
}
```

输出：
```
10 + 20 = 30
10 - 20 = -10
10 * 20 = 200
20 / 10 = 2
10 % 3  = 1
negate(50)  = -50
negate(-50) = 50
```

### 1.3 你可能会问：直接写 `+` 不就行了，为什么要用 `plus<int>`？

单独使用确实没必要。它的真正价值在于**作为参数传递给 STL 算法**。

**核心场景：`transform` 算法**

`transform` 可以将两个容器的元素逐一做运算，把结果存到第三个容器里：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>   // transform
#include <functional>  // plus
using namespace std;

int main() {
    vector<int> v1 = {10, 20, 30, 40};
    vector<int> v2 = {1,  2,  3,  4};
    vector<int> vResult(v1.size()); // 提前分配空间！

    // 将 v1 和 v2 对应元素相加，结果存入 vResult
    transform(v1.begin(), v1.end(), v2.begin(), vResult.begin(), plus<int>());
    //                                                           ^^^^^^^^^^^
    //                                              这里传入的是一个匿名对象（临时对象）

    for (int val : vResult) {
        cout << val << " ";
    }
    // 输出: 11 22 33 44

    return 0;
}
```

> 安全前提：二元 `transform` 里，`v2` 至少要有 `v1.size()` 个元素，`vResult` 也要有足够空间。

逐步解释这行代码：

```
transform(v1.begin(), v1.end(), v2.begin(), vResult.begin(), plus<int>());
         |__________|________| |_________| |______________| |__________|
          v1的遍历范围         v2的起点    结果存放起点     对每对元素执行的操作
```

`transform` 内部大致做了这件事：
```
vResult[0] = plus<int>()(v1[0], v2[0]);  // 10 + 1 = 11
vResult[1] = plus<int>()(v1[1], v2[1]);  // 20 + 2 = 22
vResult[2] = plus<int>()(v1[2], v2[2]);  // 30 + 3 = 33
vResult[3] = plus<int>()(v1[3], v2[3]);  // 40 + 4 = 44
```

如果不用 `plus<int>()`，你就得自己写一个函数或 lambda：

```cpp
// 用 lambda 替代——能实现，但没有 plus<int>() 简洁
transform(v1.begin(), v1.end(), v2.begin(), vResult.begin(),
          [](int a, int b) { return a + b; });
```

**`negate` 配合一元 `transform`：**

```cpp
vector<int> v = {10, -20, 30, -40};
vector<int> vResult(v.size());

// 一元 transform：对 v 中每个元素取反
transform(v.begin(), v.end(), vResult.begin(), negate<int>());

for (int val : vResult) {
    cout << val << " ";
}
// 输出: -10 20 -30 40
```

### 1.4 小结

```
算术仿函数 = STL 帮你预定义好的 "运算函数对象"
主要用途   = 作为参数传递给 STL 算法（transform 等）
记忆要点   = 5个二元 + 1个一元(negate)
```

---

## 二、关系仿函数（Relational Functors）

关系仿函数用于**比较两个值**，返回 `bool`。

### 2.1 完整清单

| 仿函数             | 功能     | 等价表达式 |
| ------------------ | -------- | ---------- |
| `equal_to<T>`      | 等于     | `a == b`   |
| `not_equal_to<T>`  | 不等于   | `a != b`   |
| `greater<T>`       | 大于     | `a > b`    |
| `greater_equal<T>` | 大于等于 | `a >= b`   |
| `less<T>`          | 小于     | `a < b`    |
| `less_equal<T>`    | 小于等于 | `a <= b`   |

全部都是**二元仿函数**，返回值为 `bool`。

### 2.2 基础用法

```cpp
#include <iostream>
#include <functional>
using namespace std;

int main() {
    equal_to<int> eq;
    cout << "10 == 10 ? " << eq(10, 10) << endl;  // 1 (true)
    cout << "10 == 20 ? " << eq(10, 20) << endl;  // 0 (false)

    greater<int> gt;
    cout << "10 > 20  ? " << gt(10, 20) << endl;  // 0
    cout << "20 > 10  ? " << gt(20, 10) << endl;  // 1

    less<int> lt;
    cout << "10 < 20  ? " << lt(10, 20) << endl;  // 1
    cout << "20 < 10  ? " << lt(20, 10) << endl;  // 0

    return 0;
}
```

### 2.3 最高频用法：`sort` 降序排列

这是关系仿函数最经典、最常见的使用场景。

**问题**：`sort` 默认是升序排列（从小到大），如果我想降序怎么办？

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

int main() {
    vector<int> v = {30, 10, 50, 20, 40};

    // 默认升序（内部使用的是 less<int>）
    sort(v.begin(), v.end());
    for (int val : v) cout << val << " ";
    cout << endl;
    // 输出: 10 20 30 40 50

    // 降序：传入 greater<int>()
    sort(v.begin(), v.end(), greater<int>());
    for (int val : v) cout << val << " ";
    cout << endl;
    // 输出: 50 40 30 20 10

    return 0;
}
```

**原理解析**：`sort` 的第三个参数是一个"比较规则"：

```
sort(begin, end, 比较规则);
```

- 不传第三个参数 → 默认用 `less<>()`（对 `int` 可理解为 `less<int>()`），即 `a < b` 时把 a 放前面 → **升序**
- 传入 `greater<int>()` → `a > b` 时把 a 放前面 → **降序**

### 2.4 更多实战场景

**场景一：在有序容器中指定排序方式**

`set` 和 `map` 默认使用 `less<T>` 升序排列，你可以用 `greater<T>` 改为降序：

```cpp
#include <iostream>
#include <set>
#include <functional>
using namespace std;

int main() {
    // 默认升序 set
    set<int> s1 = {30, 10, 50, 20, 40};
    for (int val : s1) cout << val << " ";
    cout << endl;
    // 输出: 10 20 30 40 50

    // 降序 set：模板参数中传入 greater<int>
    set<int, greater<int>> s2 = {30, 10, 50, 20, 40};
    for (int val : s2) cout << val << " ";
    cout << endl;
    // 输出: 50 40 30 20 10

    return 0;
}
```

> 注意语法区别：
> - `sort` 中传的是**对象**：`greater<int>()`（带括号，构造一个临时对象）
> - `set` 模板中传的是**类型**：`greater<int>`（不带括号，是个类型参数）

**场景二：`count_if` 配合 `bind` 统计满足条件的元素**

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // 写法1（推荐给初学者）：lambda，可读性最好
    int count1 = count_if(v.begin(), v.end(), [](int x) { return x > 25; });
    cout << "大于25的元素个数(lambda): " << count1 << endl; // 3（30, 40, 50）

    // 写法2：bind（能用，但一般不如 lambda 直观）
    // bind2nd 已废弃(C++17移除)，现代写法用 bind 或 lambda
    int count2 = count_if(v.begin(), v.end(),
                          bind(greater<int>(), placeholders::_1, 25));
    //                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    //                    对每个元素 x，执行 greater<int>()(x, 25)，即 x > 25
    cout << "大于25的元素个数(bind): " << count2 << endl;   // 3

    return 0;
}
```

### 2.5 小结

```
关系仿函数  = 返回 bool 的比较函数对象
最常见用途  = sort 降序 → sort(v.begin(), v.end(), greater<int>())
              set/map 降序 → set<int, greater<int>>
记忆要点    = greater 降序，less 升序（默认）
```

---

## 三、逻辑仿函数（Logical Functors）

逻辑仿函数用于执行布尔逻辑运算。**实际开发中使用频率最低**，但了解它们是完整的。

### 3.1 完整清单

| 仿函数           | 功能   | 参数数量 | 等价表达式 |
| ---------------- | ------ | -------- | ---------- |
| `logical_and<T>` | 逻辑与 | 二元     | `a && b`   |
| `logical_or<T>`  | 逻辑或 | 二元     | `a \|\| b` |
| `logical_not<T>` | 逻辑非 | **一元** | `!a`       |

> 与算术仿函数类似：两个二元 + 一个一元。

### 3.2 基础用法

```cpp
#include <iostream>
#include <functional>
using namespace std;

int main() {
    logical_and<bool> myAnd;
    cout << "true  && true  = " << myAnd(true, true)   << endl; // 1
    cout << "true  && false = " << myAnd(true, false)  << endl; // 0
    cout << "false && false = " << myAnd(false, false) << endl; // 0

    logical_or<bool> myOr;
    cout << "true  || false = " << myOr(true, false)   << endl; // 1
    cout << "false || false = " << myOr(false, false)  << endl; // 0

    logical_not<bool> myNot;
    cout << "!true  = " << myNot(true)  << endl; // 0
    cout << "!false = " << myNot(false) << endl; // 1

    return 0;
}
```

### 3.3 实战场景：`transform` + `logical_not` 取反容器

将一个 `bool` 容器中的所有值取反：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

int main() {
    vector<bool> v = {true, false, true, true, false};
    vector<bool> vResult(v.size());

    // 对 v 中每个元素执行 logical_not，结果存入 vResult
    transform(v.begin(), v.end(), vResult.begin(), logical_not<bool>());

    cout << "原始: ";
    for (bool b : v)       cout << b << " ";
    cout << endl;

    cout << "取反: ";
    for (bool b : vResult) cout << b << " ";
    cout << endl;

    return 0;
}
```

> 补充：`vector<bool>` 在 STL 中是一个特殊化实现，不是“普通 `vector<T>`”的完全同构版本。这里用它是因为示例短小；若你在调试中觉得行为不直观，也可以先用 `vector<int>`（0/1）来练习算法流程。

输出：
```
原始: 1 0 1 1 0
取反: 0 1 0 0 1
```

### 3.4 `logical_and` 和 `logical_or` 配合 `transform`

将两个 `bool` 容器逐元素做逻辑运算：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

int main() {
    vector<bool> v1 = {true,  true,  false, false};
    vector<bool> v2 = {true,  false, true,  false};
    vector<bool> vAnd(4), vOr(4);

    // 逐元素 &&
    transform(v1.begin(), v1.end(), v2.begin(), vAnd.begin(), logical_and<bool>());

    // 逐元素 ||
    transform(v1.begin(), v1.end(), v2.begin(), vOr.begin(), logical_or<bool>());

    cout << "v1:  ";  for (bool b : v1)   cout << b << " ";  cout << endl;
    cout << "v2:  ";  for (bool b : v2)   cout << b << " ";  cout << endl;
    cout << "AND: ";  for (bool b : vAnd) cout << b << " ";  cout << endl;
    cout << "OR:  ";  for (bool b : vOr)  cout << b << " ";  cout << endl;

    return 0;
}
```

输出：
```
v1:  1 1 0 0
v2:  1 0 1 0
AND: 1 0 0 0
OR:  1 1 1 0
```

这其实就是一个**真值表**的直接验证。

### 3.5 小结

```
逻辑仿函数  = 对 bool 值做 与/或/非 运算的函数对象
使用频率    = 三类中最低，实际开发中不太常用
记忆要点    = logical_not 是一元，其余二元
典型场景    = transform 中对 bool 容器做批量逻辑运算
```

---

## 四、总结对比与记忆技巧

### 4.1 完整总览表

| 类别     | 仿函数                                                       | 参数数量 | 返回类型 |
| -------- | ------------------------------------------------------------ | -------- | -------- |
| **算术** | `plus` `minus` `multiplies` `divides` `modulus`              | 二元     | `T`      |
| **算术** | `negate`                                                     | 一元     | `T`      |
| **关系** | `equal_to` `not_equal_to` `greater` `greater_equal` `less` `less_equal` | 二元     | `bool`   |
| **逻辑** | `logical_and` `logical_or`                                   | 二元     | `bool`   |
| **逻辑** | `logical_not`                                                | 一元     | `bool`   |

### 4.2 一元 vs 二元速记

整整 15 个内建函数对象中，只有 **2 个是一元**的：
- `negate<T>` — 算术取反
- `logical_not<T>` — 逻辑取反

它们都是"取反"操作，很好记。其余全是二元。

### 4.3 使用口诀

```
算术仿函数配 transform，两组容器做运算；
关系仿函数配 sort 用，greater 降序最常见；
逻辑仿函数用得少，bool 容器批量翻。
```

### 4.4 匿名对象 vs 命名对象

两种写法都行，通常用匿名对象更简洁：

```cpp
// 命名对象
greater<int> gt;
sort(v.begin(), v.end(), gt);

// 匿名对象（推荐，更简洁）
sort(v.begin(), v.end(), greater<int>());
//                       ^^^^^^^^^^^^^^^
//                       greater<int>() 构造了一个临时的匿名对象
```

### 4.5 C++14 的简化：透明运算符

C++14 起，可以省略模板参数，写成 `greater<>()`：

```cpp
sort(v.begin(), v.end(), greater<>());   // C++14 起可用
sort(v.begin(), v.end(), greater<int>()); // C++11 写法
```

`greater<>()` 内部使用了模板自动推导，适用于任何可比较类型，更加通用。初学阶段用哪种都可以。

---

### 最终忠告

作为 STL 新手，先牢牢记住这两个高频点就够了：

```cpp
sort(v.begin(), v.end(), greater<int>()); // 降序排序
transform(v1.begin(), v1.end(), v2.begin(), out.begin(), plus<int>()); // 批量运算
```

其余的内建函数对象，知道它们存在即可，用到时翻看这份笔记就够了。随着你使用 STL 算法（`transform`、`count_if`、`find_if` 等）越来越多，它们的价值自然会体现出来。