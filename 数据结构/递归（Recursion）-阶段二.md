# 阶段二：递归（Recursion）——完整教程

> **目标**：把递归和栈的关系彻底看明白。后面链表、树、分治、DFS 都靠它。

---

# 第一章 递归的工作原理

## 1.1 什么是递归？

**递归（Recursion）** 是指一个函数 **直接或间接地调用自身** 的编程技巧。

用最通俗的话说：

> 一个函数在执行过程中，遇到了"调用自己"的语句，于是它暂停当前的工作，重新进入自身去执行，等那次执行完毕后，再回来继续完成剩余工作。

**生活类比**：想象你站在两面相对的镜子之间——你看到镜子里有你，镜子里的你又看到镜子里的你……这就是"自我引用"。但递归不能无限下去，必须有一个 **终止条件（Base Case）**，否则就像无限镜像一样永远不会停。

## 1.2 递归的两个必备要素

| 要素                           | 说明                             | 如果缺失会怎样？                    |
| ------------------------------ | -------------------------------- | ----------------------------------- |
| **基线条件（Base Case）**      | 不再递归调用自身的终止条件       | 无限递归 → 栈溢出（Stack Overflow） |
| **递归条件（Recursive Case）** | 函数调用自身，但问题规模必须缩小 | 无法趋近基线条件 → 同样无限递归     |

## 1.3 第一个递归示例：跟踪执行过程

```cpp
#include <iostream>
using namespace std;

void fun(int n) {
    if (n > 0) {           // 递归条件
        cout << n << " ";   // 打印当前 n 的值
        fun(n - 1);         // 递归调用：问题规模缩小（n 变为 n-1）
    }
    // 当 n <= 0 时，什么都不做，直接返回 —— 这就是隐含的基线条件
}

int main() {
    fun(3);
    return 0;
}
```

**输出**：`3 2 1`

### 逐步跟踪（Tracing）

这是学递归最关键的技能——**手动跟踪调用过程**。我们来一步步分析 `fun(3)` 的执行：

```
main() 调用 fun(3)
│
├─ n=3 > 0？是 → 打印 3 → 调用 fun(2)
│   │
│   ├─ n=2 > 0？是 → 打印 2 → 调用 fun(1)
│   │   │
│   │   ├─ n=1 > 0？是 → 打印 1 → 调用 fun(0)
│   │   │   │
│   │   │   └─ n=0 > 0？否 → 直接返回
│   │   │
│   │   └─ fun(0) 返回后，fun(1) 也执行完毕，返回
│   │
│   └─ fun(1) 返回后，fun(2) 也执行完毕，返回
│
└─ fun(2) 返回后，fun(3) 也执行完毕，返回到 main()
```

**关键观察**：
- 每次调用 `fun(n-1)` 时，当前函数 **暂停** 在调用点，等待子调用完成。
- 子调用完成后，控制权 **返回** 到暂停的位置，继续执行剩余代码。
- 本例中 `fun(n-1)` 之后没有其他语句，所以返回后就直接结束了。

### 变换一下：打印语句放在递归调用之后

```cpp
void fun2(int n) {
    if (n > 0) {
        fun2(n - 1);        // 先递归
        cout << n << " ";   // 后打印
    }
}
```

调用 `fun2(3)` 的输出是什么？

```
main() 调用 fun2(3)
│
├─ n=3 > 0？是 → 调用 fun2(2)     [打印 3 被推迟]
│   │
│   ├─ n=2 > 0？是 → 调用 fun2(1)  [打印 2 被推迟]
│   │   │
│   │   ├─ n=1 > 0？是 → 调用 fun2(0) [打印 1 被推迟]
│   │   │   │
│   │   │   └─ n=0 > 0？否 → 返回
│   │   │
│   │   └─ fun2(0) 返回 → 现在打印 1
│   │
│   └─ fun2(1) 返回 → 现在打印 2
│
└─ fun2(2) 返回 → 现在打印 3
```

**输出**：`1 2 3`

> **核心领悟**：打印语句在递归调用 **之前** 还是 **之后**，会产生完全不同的效果。前者在"递（going down）"的阶段执行，后者在"归（coming back）"的阶段执行。

---

## 1.4 递归的两个阶段

每一次递归调用，都包含两个阶段：

```
            调用阶段（Calling Phase / Ascending）
            ─────────────────────────────────→
fun(3) → fun(2) → fun(1) → fun(0)
            ←─────────────────────────────────
            返回阶段（Returning Phase / Descending）
```

| 阶段               | 特点                             |
| ------------------ | -------------------------------- |
| **调用阶段（递）** | 从大问题走向小问题，函数依次入栈 |
| **返回阶段（归）** | 从基线条件往回走，函数依次出栈   |

在调用阶段之前执行的操作 → **在"递"的时候完成**
在递归调用之后执行的操作 → **在"归"的时候完成**

---

# 第二章 递归的泛化（Generalization）

## 2.1 什么叫"泛化"？

刚才的例子 `fun(3)` 里 `n=3` 是写死的。**泛化**就是把递归写成通用公式，使之对任意输入都成立。

### 示例分析

```cpp
void fun(int n) {
    if (n > 0) {
        cout << n << " ";
        fun(n - 1);
    }
}
```

对于任意正整数 $n$，这个函数会输出：

$$n, \; n-1, \; n-2, \; \ldots, \; 2, \; 1$$

我们可以用数学方式描述它：

$$
\text{fun}(n) = 
\begin{cases} 
\text{什么都不做} & \text{if } n \le 0 \\
\text{打印 } n, \;\text{然后调用 fun}(n-1) & \text{if } n > 0
\end{cases}
$$

这就是递归函数的 **递推定义（Recurrence Definition）**。

## 2.2 从特殊到一般

当你设计递归函数时，步骤是：

1. **找到基线条件**：最小/最简单的输入是什么？该返回什么？
2. **假设子问题已解决**：假设 `fun(n-1)` 已经正确地完成了它的工作。
3. **用子问题构造当前问题的解**：当前 `fun(n)` 要在 `fun(n-1)` 的基础上做什么？

> 🧠 **递归思维的精髓**：不要试图在脑子里展开所有调用层次！只考虑"当前层做什么"和"子问题做什么"即可。

---

# 第三章 递归如何使用栈（Stack）

## 3.1 函数调用与栈帧

当一个函数被调用时，系统会在 **调用栈（Call Stack）** 上分配一块内存区域，称为 **栈帧（Stack Frame）** 或 **激活记录（Activation Record）**。

每个栈帧存储：
- 函数的 **参数**
- 函数的 **局部变量**
- **返回地址**（调用完成后回到哪里继续执行）

```
┌─────────────────────┐ ← 栈顶（Stack Top）
│  fun(0) 的栈帧       │   n=0, 返回地址=fun(1)中的某行
├─────────────────────┤
│  fun(1) 的栈帧       │   n=1, 返回地址=fun(2)中的某行
├─────────────────────┤
│  fun(2) 的栈帧       │   n=2, 返回地址=fun(3)中的某行
├─────────────────────┤
│  fun(3) 的栈帧       │   n=3, 返回地址=main()中的某行
├─────────────────────┤
│  main() 的栈帧       │
└─────────────────────┘ ← 栈底（Stack Bottom）
```

## 3.2 调用过程详解

以 `fun(3)` 为例，我们详细展示栈的变化：

**第 1 步：main() 调用 fun(3)**

```
栈状态：
┌──────────┐
│ fun(3)   │  n = 3
│ main()   │
└──────────┘
```
`fun(3)` 开始执行：$n=3>0$，打印 `3`，然后调用 `fun(2)`。

**第 2 步：fun(3) 调用 fun(2)**

```
栈状态：
┌──────────┐
│ fun(2)   │  n = 2
│ fun(3)   │  n = 3  [暂停中，等待 fun(2) 返回]
│ main()   │
└──────────┘
```
`fun(2)` 开始执行：$n=2>0$，打印 `2`，然后调用 `fun(1)`。

**第 3 步：fun(2) 调用 fun(1)**

```
栈状态：
┌──────────┐
│ fun(1)   │  n = 1
│ fun(2)   │  n = 2  [暂停中]
│ fun(3)   │  n = 3  [暂停中]
│ main()   │
└──────────┘
```
`fun(1)` 开始执行：$n=1>0$，打印 `1`，然后调用 `fun(0)`。

**第 4 步：fun(1) 调用 fun(0)**

```
栈状态：
┌──────────┐
│ fun(0)   │  n = 0
│ fun(1)   │  n = 1  [暂停中]
│ fun(2)   │  n = 2  [暂停中]
│ fun(3)   │  n = 3  [暂停中]
│ main()   │
└──────────┘
```
`fun(0)` 开始执行：$n=0$，不满足 $n>0$，所以 **不递归**，直接返回。

**第 5 步：fun(0) 返回 → fun(1) 恢复**

```
栈状态：
┌──────────┐
│ fun(1)   │  n = 1  [恢复执行，但 fun(n-1) 之后已无其他语句，返回]
│ fun(2)   │  n = 2
│ fun(3)   │  n = 3
│ main()   │
└──────────┘
```

**第 6 步：fun(1) 返回 → fun(2) 恢复 → 返回**

```
栈状态：
┌──────────┐
│ fun(2)   │  n = 2  [恢复，返回]
│ fun(3)   │  n = 3
│ main()   │
└──────────┘
```

**第 7 步：fun(2) 返回 → fun(3) 恢复 → 返回**

```
栈状态：
┌──────────┐
│ fun(3)   │  n = 3  [恢复，返回]
│ main()   │
└──────────┘
```

**第 8 步：fun(3) 返回 → main() 继续**

```
栈状态：
┌──────────┐
│ main()   │
└──────────┘
```

## 3.3 空间消耗分析

对于 `fun(n)`，最多同时存在 $n+1$ 个栈帧（包括 `fun(0)` 和 `main()`）。

$$\text{空间复杂度} = O(n)$$

> ⚠️ **栈溢出（Stack Overflow）**：如果递归深度太大（比如 `fun(1000000)`），栈空间可能不够用，程序会崩溃。这就是为什么递归必须有基线条件，且问题规模必须缩小。

## 3.4 递归与栈的等价性

**一个深刻的事实**：任何递归都可以用显式栈（stack 数据结构）+ 循环来模拟。

```cpp
// 递归版
void fun(int n) {
    if (n > 0) {
        cout << n << " ";
        fun(n - 1);
    }
}

// 等价的迭代版（用栈模拟）
#include <stack>
void fun_iterative(int n) {
    stack<int> s;
    while (n > 0) {
        cout << n << " ";
        s.push(n);      // 模拟"入栈"
        n = n - 1;
    }
    // 此处栈中依次弹出（模拟返回阶段），本例中返回阶段无操作，故忽略
}
```

这个等价性告诉我们：**递归本质上就是利用系统调用栈来保存中间状态**。理解这一点，对后面的树遍历、DFS 至关重要。

---

# 第四章 递归的时间复杂度——递推关系

## 4.1 如何分析递归函数的时间复杂度？

对于递归函数，我们不能像循环那样直接数"迭代次数"。我们需要建立 **递推关系（Recurrence Relation）**，然后求解。

### 示例 1

```cpp
void fun(int n) {       // T(n) 表示 fun(n) 的时间
    if (n > 0) {         // O(1) —— 比较
        cout << n << " "; // O(1) —— 打印
        fun(n - 1);       // T(n-1)
    }
}
```

建立递推关系：

$$
T(n) = 
\begin{cases} 
1 & \text{if } n = 0 \\
T(n-1) + 1 & \text{if } n > 0
\end{cases}
$$

（这里的 $1$ 代表常数时间操作；为简洁起见，我们用 $1$ 替代 $O(1)$。）

### 求解方法：逐步代入法（Substitution / Back Substitution）

$$
T(n) = T(n-1) + 1
$$

用 $T(n-1) = T(n-2) + 1$ 代入：

$$
T(n) = [T(n-2) + 1] + 1 = T(n-2) + 2
$$

再代入 $T(n-2) = T(n-3) + 1$：

$$
T(n) = T(n-3) + 3
$$

观察规律，代入 $k$ 次后：

$$
T(n) = T(n-k) + k
$$

当 $n - k = 0$，即 $k = n$ 时：

$$
T(n) = T(0) + n = 1 + n
$$

$$\boxed{T(n) = n + 1 = O(n)}$$

### 示例 2

```cpp
void fun(int n) {
    if (n > 0) {
        for (int i = 0; i < n; i++) {   // O(n) 的循环
            cout << n << " ";
        }
        fun(n - 1);                      // T(n-1)
    }
}
```

递推关系：

$$
T(n) = T(n-1) + n
$$

求解：

$$
T(n) = T(n-1) + n = T(n-2) + (n-1) + n = T(n-3) + (n-2) + (n-1) + n
$$

代入 $k$ 次后当 $n - k = 0$：

$$
T(n) = 1 + 2 + 3 + \cdots + n = \frac{n(n+1)}{2}
$$

$$\boxed{T(n) = O(n^2)}$$

### 示例 3：每次减半

```cpp
void fun(int n) {
    if (n > 1) {
        cout << n << " ";
        fun(n / 2);
    }
}
```

递推关系：

$$
T(n) = T(n/2) + 1
$$

代入：$T(n) = T(n/4) + 2 = T(n/8) + 3 = \cdots = T(n/2^k) + k$

当 $n/2^k = 1$，即 $k = \log_2 n$ 时：

$$
T(n) = T(1) + \log_2 n = 1 + \log_2 n
$$

$$\boxed{T(n) = O(\log n)}$$

## 4.2 常见递推关系速查表

| 递推关系                 | 时间复杂度    | 典型场景         |
| ------------------------ | ------------- | ---------------- |
| $T(n) = T(n-1) + 1$      | $O(n)$        | 线性递归         |
| $T(n) = T(n-1) + n$      | $O(n^2)$      | 含循环的线性递归 |
| $T(n) = T(n-1) + \log n$ | $O(n \log n)$ | —                |
| $T(n) = T(n/2) + 1$      | $O(\log n)$   | 二分搜索         |
| $T(n) = T(n/2) + n$      | $O(n)$        | —                |
| $T(n) = 2T(n/2) + n$     | $O(n \log n)$ | 归并排序         |
| $T(n) = 2T(n-1) + 1$     | $O(2^n)$      | 树递归           |

---

# 第五章 递归代码实现基础

## 5.1 完整代码模板

```cpp
#include <iostream>
using namespace std;

// 递归函数：打印从 n 到 1 的数
void printDescending(int n) {
    if (n > 0) {              // 递归条件
        cout << n << " ";      // 在"递"的阶段执行
        printDescending(n - 1); // 递归调用
    }
    // 隐含基线条件：n <= 0 时不做任何事
}

// 递归函数：打印从 1 到 n 的数
void printAscending(int n) {
    if (n > 0) {               // 递归条件
        printAscending(n - 1);  // 先递归
        cout << n << " ";       // 在"归"的阶段执行
    }
}

int main() {
    cout << "降序: ";
    printDescending(5);  // 输出: 5 4 3 2 1
    cout << endl;

    cout << "升序: ";
    printAscending(5);   // 输出: 1 2 3 4 5
    cout << endl;

    return 0;
}
```

## 5.2 有返回值的递归

```cpp
// 递归求 1+2+...+n
int sum(int n) {
    if (n == 0)          // 基线条件
        return 0;
    return n + sum(n - 1); // 递归条件
}

int main() {
    cout << sum(5) << endl;  // 输出: 15
    return 0;
}
```

跟踪求解过程：

```
sum(5) = 5 + sum(4)
             = 4 + sum(3)
                   = 3 + sum(2)
                         = 2 + sum(1)
                               = 1 + sum(0)
                                     = 0     ← 基线条件
                               = 1 + 0 = 1
                         = 2 + 1 = 3
                   = 3 + 3 = 6
             = 4 + 6 = 10
       = 5 + 10 = 15
```

---

# 第六章 递归中的静态变量与全局变量

## 6.1 局部变量 vs 静态变量

在递归中，**局部变量** 和 **静态/全局变量** 的行为截然不同：

| 特性         | 局部变量                     | 静态变量 / 全局变量    |
| ------------ | ---------------------------- | ---------------------- |
| 存储位置     | 栈帧中（每次调用有独立副本） | 静态区（全局只有一份） |
| 生命周期     | 函数调用结束即销毁           | 程序整个运行期间       |
| 每次递归调用 | **独立的一份**               | **所有调用共享同一份** |

### 示例对比

```cpp
#include <iostream>
using namespace std;

// 使用局部变量
int fun_local(int n) {
    if (n > 0) {
        return fun_local(n - 1) + n;  // 每次 n 是独立的
    }
    return 0;
}

// 使用静态变量
int fun_static(int n) {
    static int x = 0;   // 只初始化一次！以后调用不会重新初始化
    if (n > 0) {
        x++;             // 所有递归层共享同一个 x
        return fun_static(n - 1) + x;
    }
    return 0;
}

int main() {
    cout << fun_local(5) << endl;   // 输出: 15 (= 5+4+3+2+1)
    cout << fun_static(5) << endl;  // 输出: ?（需要仔细分析！）
    return 0;
}
```

### 详细跟踪 `fun_static(5)`

```
x 初始化为 0（static，只执行一次）

调用 fun_static(5): n=5 > 0, x++ → x=1, 调用 fun_static(4)
  调用 fun_static(4): n=4 > 0, x++ → x=2, 调用 fun_static(3)
    调用 fun_static(3): n=3 > 0, x++ → x=3, 调用 fun_static(2)
      调用 fun_static(2): n=2 > 0, x++ → x=4, 调用 fun_static(1)
        调用 fun_static(1): n=1 > 0, x++ → x=5, 调用 fun_static(0)
          调用 fun_static(0): n=0, 返回 0
        返回 fun_static(0) + x = 0 + 5 = 5
      返回 fun_static(1) + x = 5 + 5 = 10
    返回 fun_static(2) + x = 10 + 5 = 15
  返回 fun_static(3) + x = 15 + 5 = 20
返回 fun_static(4) + x = 20 + 5 = 25
```

**输出：25**

> ⚡ **关键**：注意在"归"的阶段，`x` 的值已经变成了 5，而不是每层各自对应的值。因为 `x` 是共享的！这与局部变量的结果（15）完全不同。

## 6.2 全局变量的效果

```cpp
int x = 0;  // 全局变量

int fun_global(int n) {
    if (n > 0) {
        x++;
        return fun_global(n - 1) + x;
    }
    return 0;
}
```

全局变量的行为与 `static` 局部变量完全相同——所有递归调用共享同一个 `x`，结果也是 25。

## 6.3 练习（对应 P31）

**题目**：分析以下代码的输出：

```cpp
#include <iostream>
using namespace std;

int fun(int n) {
    static int x = 0;
    if (n > 0) {
        x++;
        return fun(n - 1) + x;
    }
    return 0;
}

int main() {
    int a = fun(5);
    cout << a << endl;  // 问题：输出什么？

    // 注意：再次调用 fun(5)，static x 不会重新初始化！
    int b = fun(5);
    cout << b << endl;  // 问题：输出什么？
    return 0;
}
```

### 参考答案与解析

第一次调用 `fun(5)`：如前分析，$x$ 最终变为 5，返回 25。输出 **25**。

第二次调用 `fun(5)`：此时 $x$ 从 5 开始（`static` 变量不会重新初始化）！
- $x$ 依次变为 6, 7, 8, 9, 10
- 在"归"的阶段，每次加的都是 $x=10$
- 返回值：$0 + 10 + 10 + 10 + 10 + 10 = 50$

输出 **50**。

**教训**：`static` 变量带有"记忆力"，第二次调用时状态延续。在递归中使用 `static` 变量要特别小心。

---

# 第七章 尾递归（Tail Recursion）

## 7.1 定义

> **尾递归（Tail Recursion）**：如果递归调用是函数中执行的 **最后一个操作**（即递归调用之后没有任何其他语句需要执行），则称为尾递归。

```cpp
// 尾递归 ✅
void tailFun(int n) {
    if (n > 0) {
        cout << n << " ";   // 先执行操作
        tailFun(n - 1);     // 最后才递归调用 —— 之后没有任何其他操作
    }
}
```

```cpp
// 非尾递归 ❌
void nonTailFun(int n) {
    if (n > 0) {
        nonTailFun(n - 1);  // 递归调用
        cout << n << " ";   // 递归调用之后还有操作！
    }
}
```

## 7.2 尾递归的重要性

**尾递归通常更容易改写为循环（迭代）**。另外，部分编译器在特定优化配置下可能进行 **尾调用优化（Tail Call Optimization, TCO）**，但在 C++ 中这并不是语言层面的强保证。

- ✅ 手工改写为循环后，空间复杂度可从 $O(n)$ 降为 $O(1)$
- ✅ 若编译器确实执行了 TCO，则递归调用可复用栈帧，显著降低栈增长
- ⚠️ 在调试构建、未开启优化或代码形态不满足条件时，TCO 可能不会发生

### 尾递归 → 循环的转换

```cpp
// 尾递归版
void tailFun(int n) {
    if (n > 0) {
        cout << n << " ";
        tailFun(n - 1);
    }
}

// 等价的循环版
void loopFun(int n) {
    while (n > 0) {
        cout << n << " ";
        n = n - 1;
    }
}
```

转换规则很简单：
1. `if` 变成 `while`
2. 递归调用中的参数变化变成循环体末尾的变量更新
3. 删掉递归调用

## 7.3 效率对比

|            | 尾递归（无 TCO） | 循环   |
| ---------- | ---------------- | ------ |
| 时间复杂度 | $O(n)$           | $O(n)$ |
| 空间复杂度 | $O(n)$（调用栈） | $O(1)$ |

> 💡 **建议**：如果你写的递归是尾递归，那最好直接改写成循环，因为效率更高。

> 💡 **严谨补充**：对于教学与工程实践，优先把“可改写的尾递归”主动写成循环，不要把“是否有 TCO”当作可依赖前提。

---

# 第八章 头递归（Head Recursion）

## 8.1 定义

> **头递归（Head Recursion）**：递归调用是函数中执行的 **第一个操作**（在递归调用之前，没有任何执行语句）。所有的处理都在递归调用之后。

```cpp
// 头递归
void headFun(int n) {
    if (n > 0) {
        headFun(n - 1);     // 第一个操作就是递归
        cout << n << " ";   // 归来时才处理
    }
}
```

调用 `headFun(3)` 的输出：`1 2 3`

## 8.2 头递归转循环——没那么直观

头递归不能像尾递归那样简单地替换成循环。需要一些思考：

```cpp
// 头递归版：输出 1 2 3 ... n
void headFun(int n) {
    if (n > 0) {
        headFun(n - 1);
        cout << n << " ";
    }
}

// 等价的循环版
void loopHeadFun(int n) {
    int i = 1;
    while (i <= n) {
        cout << i << " ";
        i++;
    }
}
```

注意：转换时需要理解递归的逻辑含义（此处是"先到底，再从 1 开始打印"），然后重新设计循环。

## 8.3 头递归 vs 尾递归

| 特性           | 尾递归         | 头递归           |
| -------------- | -------------- | ---------------- |
| 递归调用位置   | 函数末尾       | 函数开头         |
| 处理工作发生在 | 调用阶段（递） | 返回阶段（归）   |
| 转为循环       | 简单直接       | 需要重新思考逻辑 |
| 编译器优化     | 容易 TCO       | 不容易直接 TCO   |

---

# 第九章 树递归（Tree Recursion）

## 9.1 定义

> **树递归（Tree Recursion）**：一个函数在一次调用中 **递归调用自身两次或更多次**。

```cpp
void treeFun(int n) {
    if (n > 0) {
        cout << n << " ";
        treeFun(n - 1);   // 第一次递归调用
        treeFun(n - 1);   // 第二次递归调用
    }
}
```

## 9.2 跟踪树递归

调用 `treeFun(3)` 的完整调用树：

```
                        treeFun(3)
                       /          \
                 treeFun(2)      treeFun(2)
                /        \       /        \
          treeFun(1)  treeFun(1)  treeFun(1)  treeFun(1)
           /    \      /    \      /    \      /    \
        tf(0) tf(0) tf(0) tf(0) tf(0) tf(0) tf(0) tf(0)
```

**输出顺序**（按照深度优先、先左后右）：

```
3 2 1 1 2 1 1
```

让我仔细跟踪：

```
treeFun(3): 打印 3
├── treeFun(2): 打印 2
│   ├── treeFun(1): 打印 1
│   │   ├── treeFun(0): 返回
│   │   └── treeFun(0): 返回
│   └── treeFun(1): 打印 1
│       ├── treeFun(0): 返回
│       └── treeFun(0): 返回
└── treeFun(2): 打印 2
    ├── treeFun(1): 打印 1
    │   ├── treeFun(0): 返回
    │   └── treeFun(0): 返回
    └── treeFun(1): 打印 1
        ├── treeFun(0): 返回
        └── treeFun(0): 返回
```

**输出**：`3 2 1 1 2 1 1`

## 9.3 时间复杂度

递推关系：

$$T(n) = 2T(n-1) + 1$$

求解（代入法）：

$$
T(n) = 2T(n-1) + 1
$$
$$
= 2[2T(n-2) + 1] + 1 = 4T(n-2) + 2 + 1
$$
$$
= 4[2T(n-3) + 1] + 3 = 8T(n-3) + 4 + 2 + 1
$$
$$
= 2^k T(n-k) + (2^k - 1)
$$

当 $n - k = 0$，$k = n$：

$$
T(n) = 2^n \cdot T(0) + 2^n - 1 = 2^n \cdot 1 + 2^n - 1 = 2^{n+1} - 1
$$

$$\boxed{T(n) = O(2^n)}$$

## 9.4 空间复杂度

虽然总调用次数是 $O(2^n)$，但 **同时存在于栈中的帧数** 最多为 $n+1$ 层（从根到叶的最长路径），所以：

$$\boxed{\text{空间复杂度} = O(n)}$$

> 🎯 **重要**：树递归是指数级时间复杂度的根源。后面的斐波那契朴素递归就是典型的树递归。

## 9.5 练习（对应 P35）

**题目**：以下函数调用 `treeFun(4)` 的输出是什么？总共调用了多少次？

```cpp
void treeFun(int n) {
    if (n > 0) {
        cout << n << " ";
        treeFun(n - 1);
        treeFun(n - 1);
    }
}
```

### 参考答案与解析

输出：`4 3 2 1 1 2 1 1 3 2 1 1 2 1 1`

总调用次数（包括基线条件的调用）：$2^{4+1} - 1 = 31$ 次。
其中有效打印（`n > 0` 的调用）：$2^4 - 1 = 15$ 次。

---

# 第十章 间接递归（Indirect Recursion）

## 10.1 定义

> **间接递归（Indirect Recursion）** / **互递归（Mutual Recursion）**：函数 A 调用函数 B，函数 B 又调用函数 A。形成一个 **循环调用链**。

```
funA() → funB() → funA() → funB() → ...
```

也可以是多个函数形成的链：

```
funA() → funB() → funC() → funA() → ...
```

## 10.2 示例

```cpp
#include <iostream>
using namespace std;

void funB(int n);  // 前向声明（因为 funA 中要调用 funB）

void funA(int n) {
    if (n > 0) {
        cout << n << " ";
        funB(n - 1);       // A 调用 B
    }
}

void funB(int n) {
    if (n > 1) {
        cout << n << " ";
        funA(n / 2);       // B 调用 A
    }
}

int main() {
    funA(20);
    return 0;
}
```

### 跟踪

```
funA(20): 20 > 0 → 打印 20 → 调用 funB(19)
  funB(19): 19 > 1 → 打印 19 → 调用 funA(9)
    funA(9): 9 > 0 → 打印 9 → 调用 funB(8)
      funB(8): 8 > 1 → 打印 8 → 调用 funA(4)
        funA(4): 4 > 0 → 打印 4 → 调用 funB(3)
          funB(3): 3 > 1 → 打印 3 → 调用 funA(1)
            funA(1): 1 > 0 → 打印 1 → 调用 funB(0)
              funB(0): 0 > 1？否 → 返回
```

**输出**：`20 19 9 8 4 3 1`

## 10.3 注意事项

1. **前向声明**：C++ 要求在使用函数之前声明它。互递归中至少需要一个前向声明。
2. **终止条件**：两个函数中都需要基线条件，确保递归会终止。
3. **应用场景**：奇偶判断、某些语法分析器、状态机等。

### 奇偶判断的间接递归示例

```cpp
bool isOdd(int n);  // 前向声明

bool isEven(int n) {
    if (n == 0) return true;
    return isOdd(n - 1);
}

bool isOdd(int n) {
    if (n == 0) return false;
    return isEven(n - 1);
}

// isEven(4) → isOdd(3) → isEven(2) → isOdd(1) → isEven(0) → true
```

## 10.4 练习（对应 P37）

**题目**：跟踪以下间接递归的执行，写出输出：

```cpp
void B(int n);

void A(int n) {
    if (n > 0) {
        cout << n << " ";
        B(n - 1);
    }
}

void B(int n) {
    if (n > 0) {
        cout << n << " ";
        A(n - 1);
    }
}

int main() {
    A(5);
    return 0;
}
```

### 参考答案与解析

```
A(5): 打印 5 → B(4)
  B(4): 打印 4 → A(3)
    A(3): 打印 3 → B(2)
      B(2): 打印 2 → A(1)
        A(1): 打印 1 → B(0)
          B(0): 0 > 0？否 → 返回
```

**输出**：`5 4 3 2 1`

这个间接递归等价于一个简单的从 $n$ 到 1 的递减打印。

---

# 第十一章 嵌套递归（Nested Recursion）

## 11.1 定义

> **嵌套递归（Nested Recursion）**：递归调用的 **参数** 本身也是一个递归调用。即递归调用"嵌套在"参数位置。

```cpp
int nested(int n) {
    if (n > 100)
        return n - 10;
    else
        return nested(nested(n + 11));  // 参数里套了一层递归！
}
```

这就是著名的 **McCarthy 91 函数**。

## 11.2 跟踪嵌套递归

让我们跟踪 `nested(95)`：

```
nested(95): 95 ≤ 100 → nested(nested(106))
  nested(106): 106 > 100 → 返回 96
nested(96): 96 ≤ 100 → nested(nested(107))
  nested(107): 107 > 100 → 返回 97
nested(97): 97 ≤ 100 → nested(nested(108))
  nested(108): 108 > 100 → 返回 98
nested(98): 98 ≤ 100 → nested(nested(109))
  nested(109): 109 > 100 → 返回 99
nested(99): 99 ≤ 100 → nested(nested(110))
  nested(110): 110 > 100 → 返回 100
nested(100): 100 ≤ 100 → nested(nested(111))
  nested(111): 111 > 100 → 返回 101
nested(101): 101 > 100 → 返回 91
```

**最终结果**：`nested(95) = 91`

> 🎯 **McCarthy 91 函数的性质**：对于所有 $n \le 100$，该函数都返回 91。对于 $n > 100$，返回 $n - 10$。

$$
M(n) = 
\begin{cases} 
n - 10 & \text{if } n > 100 \\
91 & \text{if } n \le 100
\end{cases}
$$

## 11.3 嵌套递归的跟踪技巧

跟踪嵌套递归时：
1. **从内层开始**：先求解参数中的递归调用
2. **用结果替换**：把内层结果代入外层
3. **重复**：直到达到基线条件

## 11.4 练习（对应 P39）

**题目**：计算 `nested(98)` 的返回值。

### 参考答案与解析

按照上面的分析模式：

```
nested(98) → nested(nested(109))
  nested(109) → 99
nested(99) → nested(nested(110))
  nested(110) → 100
nested(100) → nested(nested(111))
  nested(111) → 101
nested(101) → 91
```

**返回值**：91

（对于所有 $n \le 100$，都返回 91。）

---

# 第十二章 递归经典应用（一）：自然数之和

## 12.1 问题定义

计算 $1 + 2 + 3 + \cdots + n$，即：

$$S(n) = \sum_{i=1}^{n} i$$

## 12.2 递推关系

$$
S(n) = 
\begin{cases} 
0 & \text{if } n = 0 \\
S(n-1) + n & \text{if } n > 0
\end{cases}
$$

**理解**：要算前 $n$ 个数的和，先算前 $n-1$ 个数的和，再加上 $n$。

## 12.3 代码实现

```cpp
#include <iostream>
using namespace std;

// 递归版
int sumRecursive(int n) {
    if (n == 0)
        return 0;              // 基线条件
    return sumRecursive(n - 1) + n;  // 递归条件
}

// 迭代版（对比）
int sumIterative(int n) {
    int total = 0;
    for (int i = 1; i <= n; i++)
        total += i;
    return total;
}

// 公式法（对比）
int sumFormula(int n) {
    return n * (n + 1) / 2;
}

int main() {
    int n = 10;
    cout << "递归: " << sumRecursive(n) << endl;  // 55
    cout << "迭代: " << sumIterative(n) << endl;  // 55
    cout << "公式: " << sumFormula(n) << endl;    // 55
    return 0;
}
```

## 12.4 复杂度分析

| 方法 | 时间复杂度 | 空间复杂度       |
| ---- | ---------- | ---------------- |
| 递归 | $O(n)$     | $O(n)$（调用栈） |
| 迭代 | $O(n)$     | $O(1)$           |
| 公式 | $O(1)$     | $O(1)$           |

## 12.5 练习（对应 P41）

**题目**：用递归实现计算 $1^2 + 2^2 + 3^2 + \cdots + n^2$（前 $n$ 个自然数的平方和）。

### 参考答案与解析

$$
S(n) = 
\begin{cases} 
0 & \text{if } n = 0 \\
S(n-1) + n^2 & \text{if } n > 0
\end{cases}
$$

```cpp
int sumOfSquares(int n) {
    if (n == 0)
        return 0;
    return sumOfSquares(n - 1) + n * n;
}
```

验证：`sumOfSquares(3)` = $1 + 4 + 9 = 14$  ✅

公式法：$\frac{n(n+1)(2n+1)}{6}$，当 $n=3$：$\frac{3 \times 4 \times 7}{6} = 14$ ✅

---

# 第十三章 递归经典应用（二）：阶乘

## 13.1 问题定义

$$n! = 1 \times 2 \times 3 \times \cdots \times n$$

$$
n! = 
\begin{cases} 
1 & \text{if } n = 0 \\
n \times (n-1)! & \text{if } n > 0
\end{cases}
$$

阶乘是递归最经典的入门示例，因为它的数学定义本身就是递归的。

## 13.2 代码实现

```cpp
#include <iostream>
using namespace std;

// 递归版
long long factorialRecursive(int n) {
    if (n <= 0)                          // 基线条件：0! = 1
        return 1;
    return n * factorialRecursive(n - 1); // n! = n * (n-1)!
}

// 迭代版
long long factorialIterative(int n) {
    long long result = 1;
    for (int i = 2; i <= n; i++)
        result *= i;
    return result;
}

int main() {
    for (int i = 0; i <= 10; i++) {
        cout << i << "! = " << factorialRecursive(i) << endl;
    }
    return 0;
}
```

**输出**：
```
0! = 1
1! = 1
2! = 2
3! = 6
4! = 24
5! = 120
6! = 720
7! = 5040
8! = 40320
9! = 362880
10! = 3628800
```

## 13.3 跟踪 `factorial(5)`

```
factorial(5) = 5 * factorial(4)
                   = 4 * factorial(3)
                         = 3 * factorial(2)
                               = 2 * factorial(1)
                                     = 1 * factorial(0)
                                           = 1         ← 基线条件
                                     = 1 * 1 = 1
                               = 2 * 1 = 2
                         = 3 * 2 = 6
                   = 4 * 6 = 24
             = 5 * 24 = 120
```

## 13.4 栈帧图示

```
调用阶段（入栈）：                    返回阶段（出栈）：

┌──────────────────┐                  返回 1
│ factorial(0)     │ → return 1  ─────────────→
├──────────────────┤                  返回 1*1=1
│ factorial(1): 1* │ ← ──────────────────────→
├──────────────────┤                  返回 2*1=2
│ factorial(2): 2* │ ← ──────────────────────→
├──────────────────┤                  返回 3*2=6
│ factorial(3): 3* │ ← ──────────────────────→
├──────────────────┤                  返回 4*6=24
│ factorial(4): 4* │ ← ──────────────────────→
├──────────────────┤                  返回 5*24=120
│ factorial(5): 5* │ ← ──────────────────────→
└──────────────────┘
```

## 13.5 复杂度

$$T(n) = T(n-1) + 1 = O(n)$$
$$S(n) = O(n) \quad \text{（栈深度）}$$

## 13.6 练习（对应 P43）

**题目**：跟踪 `factorial(7)` 的递归调用，写出调用栈的最大深度以及最终结果。

### 参考答案与解析

调用栈最大深度：8 层（`factorial(7)` 到 `factorial(0)` 共 8 个栈帧，不含 `main()`）。

最终结果：$7! = 5040$

计算过程：$7 \times 6 \times 5 \times 4 \times 3 \times 2 \times 1 = 5040$

---

# 第十四章 递归经典应用（三）：幂运算

## 14.1 问题定义

计算 $m^n$（$m$ 的 $n$ 次方），其中 $n \ge 0$。

## 14.2 方法一：朴素递归

$$
m^n = 
\begin{cases} 
1 & \text{if } n = 0 \\
m \times m^{n-1} & \text{if } n > 0
\end{cases}
$$

```cpp
// 时间 O(n)，空间 O(n)
int power(int m, int n) {
    if (n == 0)
        return 1;
    return m * power(m, n - 1);
}
```

**跟踪** `power(2, 5)`：

```
power(2,5) = 2 * power(2,4)
               = 2 * power(2,3)
                   = 2 * power(2,2)
                       = 2 * power(2,1)
                           = 2 * power(2,0)
                               = 1
                           = 2 * 1 = 2
                       = 2 * 2 = 4
                   = 2 * 4 = 8
               = 2 * 8 = 16
           = 2 * 16 = 32
```

$2^5 = 32$ ✅

## 14.3 方法二：快速幂（分治法优化）⭐

利用以下数学性质：

$$
m^n = 
\begin{cases} 
1 & \text{if } n = 0 \\
(m^{n/2})^2 & \text{if } n > 0 \text{ 且 } n \text{ 是偶数} \\
m \times (m^{(n-1)/2})^2 & \text{if } n > 0 \text{ 且 } n \text{ 是奇数}
\end{cases}
$$

```cpp
// 时间 O(log n)，空间 O(log n)
long long fastPower(int m, int n) {
    if (n == 0)
        return 1;
    
    if (n % 2 == 0) {
        long long half = fastPower(m, n / 2);
        return half * half;            // 偶数：平方
    } else {
        long long half = fastPower(m, (n - 1) / 2);
        return m * half * half;        // 奇数：m × 平方
    }
}
```

### 跟踪 `fastPower(2, 10)`

```
fastPower(2, 10): 10 是偶数
  half = fastPower(2, 5): 5 是奇数
    half = fastPower(2, 2): 2 是偶数
      half = fastPower(2, 1): 1 是奇数
        half = fastPower(2, 0) = 1
      return 2 * 1 * 1 = 2
    return 2 * 2 = 4
  return 2 * 4 * 4 = 32
return 32 * 32 = 1024
```

$2^{10} = 1024$ ✅

共进入 **5 次函数调用**（含 `n=0` 的基线调用），若只统计非基线递归层则为 **4 层**。相比朴素方法（10 层）明显更少。

## 14.4 复杂度对比

| 方法     | 时间复杂度  | 空间复杂度  | 乘法次数            |
| -------- | ----------- | ----------- | ------------------- |
| 朴素递归 | $O(n)$      | $O(n)$      | $n$ 次              |
| 快速幂   | $O(\log n)$ | $O(\log n)$ | $\sim 2\log_2 n$ 次 |

## 14.5 练习（对应 P45）

**题目**：
1. 用朴素递归和快速幂分别计算 $3^4$，跟踪每个递归调用。
2. 计算 $2^{20}$ 时，朴素递归需要多少次乘法？快速幂呢？

### 参考答案与解析

**1. 计算 $3^4$**

朴素递归：
```
power(3,4) = 3 * power(3,3)
           = 3 * 3 * power(3,2)
           = 3 * 3 * 3 * power(3,1)
           = 3 * 3 * 3 * 3 * power(3,0)
           = 3 * 3 * 3 * 3 * 1
           = 81
```
4 次乘法，4 次递归调用。

快速幂：
```
fastPower(3, 4): 偶数
  half = fastPower(3, 2): 偶数
    half = fastPower(3, 1): 奇数
      half = fastPower(3, 0) = 1
    return 3 * 1 * 1 = 3
  return 3 * 3 = 9
return 9 * 9 = 81
```
4 次乘法（不过算上平方操作），3 次递归调用。

**2. $2^{20}$**

- 朴素递归：20 次乘法
- 快速幂：$\lceil \log_2 20 \rceil \approx 5$ 次递归调用，大约 $2 \times 5 = 10$ 次乘法（最坏情况）

实际上远少于 20 次。

---

# 第十五章 递归经典应用（四）：斐波那契数列与记忆化

## 15.1 斐波那契数列

$$
F(n) = 
\begin{cases} 
0 & \text{if } n = 0 \\
1 & \text{if } n = 1 \\
F(n-1) + F(n-2) & \text{if } n \ge 2
\end{cases}
$$

前几项：$0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, \ldots$

## 15.2 朴素递归实现

```cpp
int fib(int n) {
    if (n <= 1)
        return n;
    return fib(n - 1) + fib(n - 2);
}
```

### 调用树（以 `fib(5)` 为例）

```
                          fib(5)
                       /         \
                  fib(4)          fib(3)
                /      \         /      \
           fib(3)    fib(2)   fib(2)   fib(1)
          /    \     /    \   /    \      |
      fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)  1
      /   \    |      |      |      |      |
  fib(1) fib(0) 1     1      0      1      0
    |      |
    1      0
```

结果：$F(5) = 5$ ✅

### 问题：大量重复计算！

从调用树可以看到：
- `fib(3)` 被计算了 **2 次**
- `fib(2)` 被计算了 **3 次**
- `fib(1)` 被计算了 **5 次**
- `fib(0)` 被计算了 **3 次**

总调用次数随 $n$ **指数增长**！

## 15.3 复杂度分析

朴素斐波那契递归的递推关系：

$$T(n) = T(n-1) + T(n-2) + 1$$

这是一个类似斐波那契数列本身的递推关系，其解为：

$$T(n) \approx O(\phi^n) \approx O(1.618^n) \approx O(2^n)$$

其中 $\phi = \frac{1+\sqrt{5}}{2} \approx 1.618$（黄金比例）。

| $n$ | 调用次数    | 大约                   |
| --- | ----------- | ---------------------- |
| 10  | 177         |                        |
| 20  | 21,891      |                        |
| 30  | 2,692,537   |                        |
| 40  | 331,160,281 | 3.3 亿                 |
| 50  | ~40 billion | 400 亿（程序会卡住！） |

> 🔥 **朴素递归计算 `fib(50)` 在大多数计算机上需要几分钟甚至更久！**

## 15.4 记忆化（Memoization）⭐

**核心思想**：用一个数组（或哈希表）存储已经计算过的结果。下次需要时直接查表，避免重复计算。

```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<long long> memo;  // 记忆化数组

long long fibMemo(int n) {
    if (n <= 1)
        return n;
    
    // 如果之前已经计算过，直接返回
    if (memo[n] != -1)
        return memo[n];
    
    // 否则计算并存储
    memo[n] = fibMemo(n - 1) + fibMemo(n - 2);
    return memo[n];
}

int main() {
    int n = 50;
    memo.assign(n + 1, -1);  // 初始化为 -1，表示尚未计算
    
    cout << "F(" << n << ") = " << fibMemo(n) << endl;
    // 输出: F(50) = 12586269025（瞬间完成！）
    
    return 0;
}
```

### 记忆化后的调用树

```
                          fib(5)
                       /         \
                  fib(4)          fib(3) ← 直接查表返回！
                /      \
           fib(3)    fib(2) ← 直接查表返回！
          /    \
      fib(2) fib(1)
      /   \
  fib(1) fib(0)
```

大量分支被"剪掉"了！

### 记忆化的复杂度

|                  | 时间复杂度 | 空间复杂度             |
| ---------------- | ---------- | ---------------------- |
| 朴素递归         | $O(2^n)$   | $O(n)$（栈深度）       |
| 记忆化递归       | $O(n)$     | $O(n)$（栈 + 数组）    |
| 迭代（自底向上） | $O(n)$     | $O(1)$（只需两个变量） |

## 15.5 自底向上的迭代实现

```cpp
long long fibIterative(int n) {
    if (n <= 1) return n;
    
    long long prev2 = 0;  // F(0)
    long long prev1 = 1;  // F(1)
    long long curr;
    
    for (int i = 2; i <= n; i++) {
        curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    
    return curr;
}
```

这是最优方案——$O(n)$ 时间，$O(1)$ 空间。

## 15.6 三种方法的完整对比代码

```cpp
#include <iostream>
#include <vector>
#include <chrono>
using namespace std;

// 方法 1：朴素递归
long long fibNaive(int n) {
    if (n <= 1) return n;
    return fibNaive(n - 1) + fibNaive(n - 2);
}

// 方法 2：记忆化递归
vector<long long> memo;
long long fibMemo(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    memo[n] = fibMemo(n - 1) + fibMemo(n - 2);
    return memo[n];
}

// 方法 3：迭代
long long fibIter(int n) {
    if (n <= 1) return n;
    long long a = 0, b = 1, c;
    for (int i = 2; i <= n; i++) {
        c = a + b;
        a = b;
        b = c;
    }
    return c;
}

int main() {
    int n = 40;
    
    // 测试朴素递归
    auto start = chrono::high_resolution_clock::now();
    cout << "朴素: F(" << n << ") = " << fibNaive(n);
    auto end = chrono::high_resolution_clock::now();
    cout << "  耗时: " 
         << chrono::duration_cast<chrono::milliseconds>(end - start).count()
         << " ms" << endl;
    
    // 测试记忆化递归
    memo.assign(n + 1, -1);
    start = chrono::high_resolution_clock::now();
    cout << "记忆化: F(" << n << ") = " << fibMemo(n);
    end = chrono::high_resolution_clock::now();
    cout << "  耗时: "
         << chrono::duration_cast<chrono::microseconds>(end - start).count()
         << " μs" << endl;
    
    // 测试迭代
    start = chrono::high_resolution_clock::now();
    cout << "迭代: F(" << n << ") = " << fibIter(n);
    end = chrono::high_resolution_clock::now();
    cout << "  耗时: "
         << chrono::duration_cast<chrono::microseconds>(end - start).count()
         << " μs" << endl;
    
    return 0;
}
```

**典型输出（n=40）**：
```
朴素: F(40) = 102334155  耗时: 523 ms
记忆化: F(40) = 102334155  耗时: 2 μs
迭代: F(40) = 102334155  耗时: 0 μs
```

> 差距是 **数万倍** 到 **数十万倍**！这就是算法优化的力量。

## 15.7 练习（对应 P52）

**题目**：
1. 朴素递归计算 `fib(6)` 时，`fib(2)` 被计算了多少次？画出完整调用树验证。
2. 用记忆化递归计算 `fib(6)` 时，总共进行了多少次递归调用？

### 参考答案与解析

**1.** 完整调用树（`fib(6)`）：

```
                                fib(6)
                           /              \
                      fib(5)               fib(4)
                    /       \             /       \
               fib(4)      fib(3)     fib(3)    fib(2)
              /     \      /    \     /    \     /    \
          fib(3) fib(2) fib(2) fib(1) fib(2) fib(1) fib(1) fib(0)
          /  \   / \    / \          / \
      f(2) f(1) f(1) f(0) f(1) f(0)  f(1) f(0)
      / \
  f(1) f(0)
```

数一数 `fib(2)` 出现的次数：**5 次**。

总调用次数：25 次。

**2.** 记忆化版本中，每个 `fib(k)`（$k = 0, 1, 2, \ldots, 6$）只被真正计算一次。总共 7 次有效计算，加上直接查表返回的那些调用，总递归调用次数约为 **11 次**（远少于 25 次）。

---

# 第十六章 汉诺塔问题（Tower of Hanoi）

## 16.1 问题描述

有三根柱子 **A**（源）、**B**（辅助）、**C**（目标），和 $n$ 个大小不同的圆盘，初始时全部按从小到大的顺序叠放在柱子 A 上。

**规则**：
1. 每次只能移动一个圆盘
2. 只能从柱子顶部取盘，放到另一根柱子顶部
3. **大盘不能放在小盘上面**

**目标**：把所有圆盘从 A 移到 C。

```
初始状态：                    目标状态：

    |          |      |           |       |        |
   [1]         |      |           |       |       [1]
  [ 2 ]        |      |           |       |      [ 2 ]
 [  3  ]       |      |           |       |     [  3  ]
════A═════ ════B═════ ════C═════   ════A═════ ════B═════ ════C═════
```

## 16.2 递归思路

这是递归思维的绝佳范例！

**目标**：把 $n$ 个盘从 A 移到 C（借助 B）

**递归分解**：

1. **子问题 1**：先把上面 $n-1$ 个盘从 A 移到 B（借助 C）
2. **移动 1 次**：把最大的盘（第 $n$ 个）从 A 直接移到 C
3. **子问题 2**：再把那 $n-1$ 个盘从 B 移到 C（借助 A）

```
步骤 1：移动 n-1 个盘从 A → B

    |          |      |
    |         [1]     |
    |        [ 2 ]    |
 [  3  ]      |       |
════A═════ ════B═════ ════C═════

步骤 2：移动第 n 个盘从 A → C

    |          |      |
    |         [1]     |
    |        [ 2 ]    |
    |          |    [  3  ]
════A═════ ════B═════ ════C═════

步骤 3：移动 n-1 个盘从 B → C

    |          |      |
    |          |     [1]
    |          |    [ 2 ]
    |          |   [  3  ]
════A═════ ════B═════ ════C═════
```

**基线条件**：当 $n = 0$ 时，没有盘要移动，直接返回。

## 16.3 代码实现

```cpp
#include <iostream>
using namespace std;

// n：盘子数量，from：源柱，to：目标柱，aux：辅助柱
void hanoi(int n, char from, char to, char aux) {
    if (n == 0) return;  // 基线条件：没有盘子，什么都不做
    
    // 步骤 1：将 n-1 个盘从 from 移到 aux（借助 to）
    hanoi(n - 1, from, aux, to);
    
    // 步骤 2：将第 n 个盘从 from 移到 to
    cout << "将盘 " << n << " 从 " << from << " 移到 " << to << endl;
    
    // 步骤 3：将 n-1 个盘从 aux 移到 to（借助 from）
    hanoi(n - 1, aux, to, from);
}

int main() {
    int n = 3;
    cout << "汉诺塔（" << n << " 个盘）的解法：" << endl;
    hanoi(n, 'A', 'C', 'B');
    return 0;
}
```

**输出**：
```
汉诺塔（3 个盘）的解法：
将盘 1 从 A 移到 C
将盘 2 从 A 移到 B
将盘 1 从 C 移到 B
将盘 3 从 A 移到 C
将盘 1 从 B 移到 A
将盘 2 从 B 移到 C
将盘 1 从 A 移到 C
```

共 7 步。

## 16.4 完整跟踪（n=3）

```
hanoi(3, A, C, B)
│
├─ hanoi(2, A, B, C)                    ← 步骤1：将2个盘从A移到B
│   │
│   ├─ hanoi(1, A, C, B)               ← 将1个盘从A移到C
│   │   │
│   │   ├─ hanoi(0, A, B, C) → 返回
│   │   ├─ 打印："将盘 1 从 A 移到 C"   ★
│   │   └─ hanoi(0, B, C, A) → 返回
│   │
│   ├─ 打印："将盘 2 从 A 移到 B"        ★
│   │
│   └─ hanoi(1, C, B, A)               ← 将1个盘从C移到B
│       │
│       ├─ hanoi(0, C, A, B) → 返回
│       ├─ 打印："将盘 1 从 C 移到 B"   ★
│       └─ hanoi(0, A, B, C) → 返回
│
├─ 打印："将盘 3 从 A 移到 C"            ★ 步骤2：最大盘直接移动
│
└─ hanoi(2, B, C, A)                    ← 步骤3：将2个盘从B移到C
    │
    ├─ hanoi(1, B, A, C)
    │   │
    │   ├─ hanoi(0, B, C, A) → 返回
    │   ├─ 打印："将盘 1 从 B 移到 A"   ★
    │   └─ hanoi(0, C, A, B) → 返回
    │
    ├─ 打印："将盘 2 从 B 移到 C"        ★
    │
    └─ hanoi(1, A, C, B)
        │
        ├─ hanoi(0, A, B, C) → 返回
        ├─ 打印："将盘 1 从 A 移到 C"   ★
        └─ hanoi(0, B, C, A) → 返回
```

按 ★ 顺序排列得到前面的 7 步输出。

## 16.5 移动次数分析

设 $T(n)$ 为移动 $n$ 个盘所需的最少步数：

$$
T(n) = 
\begin{cases} 
0 & \text{if } n = 0 \\
2T(n-1) + 1 & \text{if } n > 0
\end{cases}
$$

（两次递归调用 $T(n-1)$，加上一次移动最大盘。）

求解：

$$
T(n) = 2T(n-1) + 1
$$
$$
= 2[2T(n-2) + 1] + 1 = 4T(n-2) + 3
$$
$$
= 4[2T(n-3) + 1] + 3 = 8T(n-3) + 7
$$
$$
= 2^k T(n-k) + (2^k - 1)
$$

当 $k = n$：

$$
T(n) = 2^n \cdot T(0) + 2^n - 1 = 0 + 2^n - 1
$$

$$\boxed{T(n) = 2^n - 1}$$

| $n$ | 移动次数 $2^n - 1$   |
| --- | -------------------- |
| 1   | 1                    |
| 2   | 3                    |
| 3   | 7                    |
| 4   | 15                   |
| 10  | 1,023                |
| 20  | 1,048,575            |
| 64  | $1.8 \times 10^{19}$ |

> 传说中的汉诺塔寺庙有 64 个金盘。如果每秒移动一个盘子，需要约 **5849 亿年** 才能完成——远超宇宙年龄（138 亿年）。

## 16.6 时间与空间复杂度

| 指标                 | 值       |
| -------------------- | -------- |
| 时间复杂度           | $O(2^n)$ |
| 空间复杂度（栈深度） | $O(n)$   |

## 16.7 练习（对应 P56）

**题目**：
1. 写出 4 个盘的汉诺塔解法的所有步骤（或运行程序获得）。
2. 验证步骤数确实是 $2^4 - 1 = 15$。
3. 思考：如果要将所有盘从 A 移到 B（而不是 C），需要修改代码的哪里？

### 参考答案与解析

**1 & 2.** 运行 `hanoi(4, 'A', 'C', 'B')` 的输出：

```
将盘 1 从 A 移到 B
将盘 2 从 A 移到 C
将盘 1 从 B 移到 C
将盘 3 从 A 移到 B
将盘 1 从 C 移到 A
将盘 2 从 C 移到 B
将盘 1 从 A 移到 B
将盘 4 从 A 移到 C
将盘 1 从 B 移到 C
将盘 2 从 B 移到 A
将盘 1 从 C 移到 A
将盘 3 从 B 移到 C
将盘 1 从 A 移到 B
将盘 2 从 A 移到 C
将盘 1 从 B 移到 C
```

共 15 步 = $2^4 - 1$ ✅

**3.** 只需在 `main()` 中修改调用：

```cpp
hanoi(n, 'A', 'B', 'C');  // 从 A 到 B，用 C 做辅助
```

函数本身完全不需要改！这就是递归的优美之处——参数的变化自动处理了所有细节。

---

# 第十七章 总结与知识图谱

## 17.1 递归类型总览

```
                          递归
                           │
              ┌────────────┼────────────┐
              │            │            │
           直接递归     间接递归     嵌套递归
              │        (A→B→A)    (参数中递归)
    ┌─────────┼─────────┐
    │         │         │
  尾递归    头递归    树递归
 (末尾)   (开头)   (多次调用)
```

## 17.2 各类递归的特征速查

| 类型         | 定义               | 典型时间复杂度 | 可否轻松转为循环 |
| ------------ | ------------------ | -------------- | ---------------- |
| **尾递归**   | 递归调用在最后     | $O(n)$         | ✅ 非常简单       |
| **头递归**   | 递归调用在最前     | $O(n)$         | ⚠️ 需重新设计     |
| **树递归**   | 一次调用中多次递归 | $O(2^n)$ 常见  | ❌ 不容易         |
| **间接递归** | A 调 B，B 调 A     | 取决于具体问题 | ⚠️ 需分析         |
| **嵌套递归** | 参数本身是递归调用 | 取决于具体问题 | ❌ 很难           |

## 17.3 递归设计三步法

1. **定义子问题**：$f(n)$ 能否用 $f(n-1)$、$f(n/2)$ 等更小规模的问题表示？
2. **确定基线条件**：最小输入是什么？直接返回什么？
3. **信任递归**：假设子问题已经正确解决，只关注"当前层该做什么"。

## 17.4 递归 vs 迭代

| 维度       | 递归                    | 迭代               |
| ---------- | ----------------------- | ------------------ |
| 代码简洁度 | 通常更简洁              | 可能更冗长         |
| 可读性     | 对数学递推友好          | 对线性逻辑友好     |
| 性能       | 有调用开销              | 通常更快           |
| 空间       | $O(n)$ 或更多（调用栈） | $O(1)$ 居多        |
| 栈溢出风险 | 有                      | 无                 |
| 适用场景   | 树/图遍历、分治、回溯   | 简单循环、动态规划 |

## 17.5 核心要点回顾

> 📌 **递归的本质是利用系统调用栈保存中间状态。**
>
> 📌 **每个递归问题都有两个阶段："递"和"归"。**
>
> 📌 **记忆化是消除递归重复计算的关键技巧。**
>
> 📌 **尾递归应转为循环；树递归要警惕指数爆炸。**
>
> 📌 **理解递归，是学好链表、二叉树、排序、DFS/回溯的基石。**

## 17.6 面向学术读者的严谨性补充

1. **复杂度口径先声明**：
    - 是统计“函数调用次数”，还是统计“基本操作次数”，需要在小节开头明确。
    - 例如树递归中，$T(n)=2T(n-1)+1$ 统计的是“调用次数”（含基线调用）。

2. **实现边界要写清楚**：
    - 阶乘、幂运算、斐波那契在整数范围上都可能溢出。
    - `long long` 也有上限，示例应注明“仅用于教学规模输入”。

3. **工程行为与语言标准区分**：
    - TCO 属于“编译器优化行为”，不是“C++ 语言保证”。
    - 文档中应避免“必然优化”“不会栈溢出”这类绝对表述。

4. **可复现实验建议**：
    - 对性能对比章节，建议补充“编译参数、硬件、是否开优化（如 `-O2`）”。
    - 否则不同平台的耗时数字只能作为示意，不应当作严格结论。

---

# 附录：完整练习代码汇总

### 完整练习代码

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ========== 1. 基本递归：打印 ==========
void printDesc(int n) {
    if (n > 0) { cout << n << " "; printDesc(n-1); }
}

void printAsc(int n) {
    if (n > 0) { printAsc(n-1); cout << n << " "; }
}

// ========== 2. 自然数之和 ==========
int sumN(int n) {
    if (n == 0) return 0;
    return sumN(n-1) + n;
}

// ========== 3. 阶乘 ==========
long long factorial(int n) {
    if (n <= 0) return 1;
    return n * factorial(n-1);
}

// ========== 4. 幂运算 ==========
long long power(int m, int n) {
    if (n == 0) return 1;
    return m * power(m, n-1);
}

long long fastPower(int m, int n) {
    if (n == 0) return 1;
    if (n % 2 == 0) {
        long long h = fastPower(m, n/2);
        return h * h;
    } else {
        long long h = fastPower(m, (n-1)/2);
        return m * h * h;
    }
}

// ========== 5. 斐波那契（三种方法） ==========
long long fibNaive(int n) {
    if (n <= 1) return n;
    return fibNaive(n-1) + fibNaive(n-2);
}

vector<long long> memo;
long long fibMemo(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    memo[n] = fibMemo(n-1) + fibMemo(n-2);
    return memo[n];
}

long long fibIter(int n) {
    if (n <= 1) return n;
    long long a = 0, b = 1, c;
    for (int i = 2; i <= n; i++) { c = a + b; a = b; b = c; }
    return c;
}

// ========== 6. 汉诺塔 ==========
void hanoi(int n, char from, char to, char aux) {
    if (n == 0) return;
    hanoi(n-1, from, aux, to);
    cout << "盘 " << n << ": " << from << " -> " << to << endl;
    hanoi(n-1, aux, to, from);
}

// ========== 7. 尾递归示例 ==========
void tailExample(int n) {
    if (n > 0) { cout << n << " "; tailExample(n-1); }
}

// ========== 8. 头递归示例 ==========
void headExample(int n) {
    if (n > 0) { headExample(n-1); cout << n << " "; }
}

// ========== 9. 树递归示例 ==========
void treeExample(int n) {
    if (n > 0) {
        cout << n << " ";
        treeExample(n-1);
        treeExample(n-1);
    }
}

// ========== 10. 间接递归示例 ==========
void indB(int n);
void indA(int n) {
    if (n > 0) { cout << n << " "; indB(n-1); }
}
void indB(int n) {
    if (n > 1) { cout << n << " "; indA(n/2); }
}

// ========== 11. 嵌套递归（McCarthy 91） ==========
int mccarthy91(int n) {
    if (n > 100) return n - 10;
    return mccarthy91(mccarthy91(n + 11));
}

// ========== 主函数：测试所有功能 ==========
int main() {
    cout << "=== 降序打印 ===" << endl;
    printDesc(5); cout << endl;
    
    cout << "=== 升序打印 ===" << endl;
    printAsc(5); cout << endl;
    
    cout << "=== 自然数之和 ===" << endl;
    cout << "sum(10) = " << sumN(10) << endl;
    
    cout << "=== 阶乘 ===" << endl;
    cout << "5! = " << factorial(5) << endl;
    
    cout << "=== 幂运算 ===" << endl;
    cout << "2^10 = " << power(2, 10) << endl;
    cout << "2^10 (fast) = " << fastPower(2, 10) << endl;
    
    cout << "=== 斐波那契 ===" << endl;
    cout << "fib(10) naive = " << fibNaive(10) << endl;
    memo.assign(51, -1);
    cout << "fib(50) memo = " << fibMemo(50) << endl;
    cout << "fib(50) iter = " << fibIter(50) << endl;
    
    cout << "=== 汉诺塔 (3 盘) ===" << endl;
    hanoi(3, 'A', 'C', 'B');
    
    cout << "=== 树递归 ===" << endl;
    treeExample(3); cout << endl;
    
    cout << "=== 间接递归 ===" << endl;
    indA(20); cout << endl;
    
    cout << "=== McCarthy 91 ===" << endl;
    for (int i = 85; i <= 105; i++)
        cout << "M(" << i << ") = " << mccarthy91(i) << "  ";
    cout << endl;
    
    return 0;
}
```

---

> **下一阶段预告**：掌握递归后，你将具备理解 **链表的递归操作**、**二叉树遍历（先序/中序/后序）**、**归并排序**、**快速排序**、**DFS 搜索** 的核心基础。递归是数据结构的"内功心法"，后续所有高级话题都建立在它之上。
