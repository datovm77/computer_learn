# 第二十阶段：常用遍历算法

> **前置知识**：你需要了解 STL 容器（如 `vector`、`list`）、迭代器的基本用法，以及函数指针 / 仿函数 / Lambda 表达式的基本概念。

---

## 引言：什么是 STL 算法？

在开始学习具体算法之前，我们先建立一个核心认知：

**STL 的设计哲学是"容器与算法分离"**。容器负责数据的存储，算法负责数据的操作，而**迭代器**则是连接二者的桥梁。

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  容器     │ ──→ │  迭代器   │ ──→ │  算法     │
│ (数据)    │     │ (桥梁)    │     │ (操作)    │
└──────────┘     └──────────┘     └──────────┘
```

STL 算法定义在 `<algorithm>` 头文件中（部分在 `<numeric>` 中），它们**不依赖于特定容器**，只要你给它合法的迭代器范围 `[first, last)`，就能工作。

所有 STL 算法操作的范围都是**左闭右开区间** `[first, last)`，即包含 `first` 指向的元素，不包含 `last` 指向的元素。

---

# 77. 常用遍历算法 - `for_each`

## 77.1 为什么需要 `for_each`？

在没有 `for_each` 之前，我们遍历容器的写法是：

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // 1. 传统 for 循环
    for (int i = 0; i < v.size(); i++) {
        cout << v[i] << " ";
    }
    cout << endl;

    // 2. 迭代器遍历
    for (vector<int>::iterator it = v.begin(); it != v.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;

    // 3. 范围 for（C++11）
    for (int x : v) {
        cout << x << " ";
    }
    cout << endl;

    return 0;
}
```

这些写法都可以，但 `for_each` 提供了一种**更函数式、更灵活**的遍历方式，它允许你把"对每个元素做什么"这件事**抽象成一个可调用对象**，从而实现：

- ✅ 代码更简洁、意图更明确
- ✅ 操作可以被复用（定义一次，到处传递）
- ✅ 可以利用仿函数携带状态
- ✅ 作为"算法思维"的入门，为学习 `transform`、`find` 等打基础

---

## 77.2 函数原型

```cpp
#include <algorithm>

template<class InputIterator, class Function>
Function for_each(InputIterator first, InputIterator last, Function fn);
```

| 参数       | 含义                                      |
| ---------- | ----------------------------------------- |
| `first`    | 起始迭代器（包含）                        |
| `last`     | 结束迭代器（不包含）                      |
| `fn`       | 可调用对象，接收一个元素作为参数          |
| **返回值** | 返回传入的可调用对象 `fn`（移动后的副本） |

**核心逻辑等价于：**

```cpp
// for_each 内部大致等价于这段代码：
for (; first != last; ++first) {
    fn(*first);
}
return fn;
```

---

## 77.3 三种调用方式

### 方式一：普通函数

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// 定义一个普通函数
void printElement(int val) {
    cout << val << " ";
}

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // 传入函数名（函数名本身就是函数指针）
    for_each(v.begin(), v.end(), printElement);
    // 输出: 10 20 30 40 50

    cout << endl;
    return 0;
}
```

> 📌 **要点**：传入的是 `printElement` 而不是 `printElement()`。前者是**函数指针**，后者是**函数调用**（会报错）。

---

### 方式二：仿函数（函数对象）

仿函数是一个**重载了 `operator()` 的类的对象**，它的优势是**可以携带状态**。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// 定义一个仿函数类
class PrintWithPrefix {
public:
    string prefix;

    PrintWithPrefix(const string& p) : prefix(p) {}

    // 重载 ()
    void operator()(int val) {
        cout << prefix << val << " ";
    }
};

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // 传入仿函数对象（携带了 "Value=" 这个状态）
    for_each(v.begin(), v.end(), PrintWithPrefix("Value="));
    // 输出: Value=10 Value=20 Value=30 Value=40 Value=50

    cout << endl;
    return 0;
}
```

> 📌 **仿函数 vs 普通函数**：仿函数能携带额外的数据（如前缀字符串），普通函数做不到（除非用全局变量，但那样不优雅）。

---

### 方式三：Lambda 表达式（推荐 ⭐）

Lambda 是 C++11 引入的**匿名函数**，在现代 C++ 中是最推荐的写法。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // Lambda 表达式
    for_each(v.begin(), v.end(), [](int val) {
        cout << val << " ";
    });
    // 输出: 10 20 30 40 50

    cout << endl;
    return 0;
}
```

**Lambda 也可以捕获外部变量：**

```cpp
int main() {
    vector<int> v = {10, 20, 30, 40, 50};
    int sum = 0;

    // [&sum] 表示以引用方式捕获外部变量 sum
    for_each(v.begin(), v.end(), [&sum](int val) {
        sum += val;
    });

    cout << "总和 = " << sum << endl;  // 输出: 总和 = 150
    return 0;
}
```

---

## 77.4 利用返回值：仿函数的状态提取

`for_each` 的一个特殊能力是——**它会返回传入的可调用对象**。当你传入的是仿函数时，可以通过返回值获取仿函数在遍历过程中积累的状态。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

class Accumulator {
public:
    int sum = 0;
    int count = 0;

    void operator()(int val) {
        sum += val;
        count++;
    }
};

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // for_each 返回在内部经过遍历后的仿函数对象
    Accumulator result = for_each(v.begin(), v.end(), Accumulator());

    cout << "总和: " << result.sum << endl;      // 150
    cout << "元素个数: " << result.count << endl;  // 5
    cout << "平均值: " << (double)result.sum / result.count << endl;  // 30

    return 0;
}
```

> ⚠️ **注意**：`for_each` 内部会拷贝这个仿函数，遍历完毕后把**拷贝后修改过的副本**返回给你。所以你必须接收返回值才能拿到最终状态。如果只是 `for_each(v.begin(), v.end(), acc);` 而不接收返回值，那 `acc` 本身可能没有变化（因为传入的是副本）。

---

## 77.5 `for_each` 修改元素

如果你想在遍历过程中**修改容器中的元素**，只需要把参数声明为**引用**：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5};

    // 注意参数是 int& 引用类型
    for_each(v.begin(), v.end(), [](int& val) {
        val *= 2;  // 每个元素乘以 2
    });

    // 验证结果
    for_each(v.begin(), v.end(), [](int val) {
        cout << val << " ";
    });
    // 输出: 2 4 6 8 10

    cout << endl;
    return 0;
}
```

> 📌 如果参数是 `int val`（值传递），修改的只是副本，原容器不会改变。
> 📌 如果参数是 `int& val`（引用传递），修改的是容器中的原始元素。

---

## 77.6 `for_each` 的适用范围

`for_each` 不仅限于 `vector`，它可以用于**任何提供迭代器的容器**：

```cpp
#include <iostream>
#include <list>
#include <set>
#include <map>
#include <algorithm>
using namespace std;

int main() {
    // 用于 list
    list<string> names = {"Alice", "Bob", "Charlie"};
    for_each(names.begin(), names.end(), [](const string& name) {
        cout << "Hello, " << name << "!" << endl;
    });

    cout << "---" << endl;

    // 用于 set
    set<int> s = {5, 3, 1, 4, 2};
    for_each(s.begin(), s.end(), [](int val) {
        cout << val << " ";
    });
    cout << endl;  // 输出: 1 2 3 4 5（set 自动排序）

    cout << "---" << endl;
	
    // 用于 map（每个元素是 pair<const Key, Value>）
    map<string, int> scores = {{"Math", 90}, {"English", 85}, {"Science", 92}};
    for_each(scores.begin(), scores.end(), [](const pair<const string, int>& p) {
        cout << p.first << ": " << p.second << endl;
    });

    return 0;
}
```

输出：
```
Hello, Alice!
Hello, Bob!
Hello, Charlie!
---
1 2 3 4 5
---
English: 85
Math: 90
Science: 92
```

---

## 77.7 `for_each` vs 范围 for 的对比

| 特性             | `for_each`                   | 范围 `for`           |
| ---------------- | ---------------------------- | -------------------- |
| 引入版本         | C++98                        | C++11                |
| 能否指定子范围   | ✅ 可以（指定任意迭代器范围） | ❌ 只能遍历整个容器   |
| 能否复用操作     | ✅ 传入函数/仿函数可复用      | ❌ 操作内联在循环体中 |
| 能否获取累积状态 | ✅ 通过仿函数返回值           | 需要外部变量         |
| 代码简洁度       | 一行搞定                     | 更直观，更像传统循环 |
| 能否 `break`     | ❌ 不能中途退出               | ✅ 可以 `break`       |

**重要差异示例——遍历子范围：**

```cpp
vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// for_each 可以只遍历前5个元素
for_each(v.begin(), v.begin() + 5, [](int val) {
    cout << val << " ";
});
// 输出: 1 2 3 4 5

// 范围 for 做不到（必须借助其他方式截取）
```

---

## 77.8 C++17 并行 `for_each`

C++17 引入了执行策略（Execution Policy），可以让 `for_each` 并行执行：

```cpp
#include <algorithm>
#include <execution>  // C++17
#include <vector>

int main() {
    vector<int> v(1000000, 1);

    // 并行执行（需要编译器和标准库支持）
    for_each(execution::par, v.begin(), v.end(), [](int& val) {
        val *= 2;
    });

    return 0;
}
```

> ⚠️ 使用并行策略时，你的可调用对象必须是**线程安全**的，不能有数据竞争。

---

## 77.9 小结

```
for_each 核心要点：
┌──────────────────────────────────────────┐
│ 1. 头文件: #include <algorithm>           │
│ 2. 三种调用方式:                           │
│    ├── 普通函数                            │
│    ├── 仿函数（可携带状态）                  │
│    └── Lambda（最推荐 ⭐）                 │
│ 3. 引用参数可修改原始元素                    │
│ 4. 返回值可提取仿函数状态                    │
│ 5. 可指定任意迭代器子范围                    │
│ 6. 不能 break，不能跳过元素                 │
└──────────────────────────────────────────┘
```

---

# 78. 常用遍历算法 - `transform`

## 78.1 `transform` 是做什么的？

如果说 `for_each` 是"遍历并执行某个操作"，那么 `transform` 就是"**遍历并将操作的结果写入另一个地方**"。

核心区别：

|                  | `for_each`                   | `transform`                            |
| ---------------- | ---------------------------- | -------------------------------------- |
| 目的             | 对每个元素执行操作（副作用） | 对每个元素执行**转换**，并写入目标位置 |
| 是否需要输出容器 | 不需要                       | **需要**（必须提供输出迭代器）         |
| 典型用途         | 打印、累加、修改原容器       | 类型转换、数学变换、数据映射           |

---

## 78.2 函数原型

`transform` 有**两个重载版本**：

### 版本一：一元变换（Unary）

```cpp
template<class InputIterator, class OutputIterator, class UnaryOperation>
OutputIterator transform(InputIterator first1, InputIterator last1,
                         OutputIterator result,
                         UnaryOperation op);
```

**逻辑等价代码：**

```cpp
while (first1 != last1) {
    *result = op(*first1);  // 把变换结果写入输出位置
    ++result;
    ++first1;
}
return result;
```

### 版本二：二元变换（Binary）

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator, class BinaryOperation>
OutputIterator transform(InputIterator1 first1, InputIterator1 last1,
                         InputIterator2 first2,
                         OutputIterator result,
                         BinaryOperation op);
```

**逻辑等价代码：**

```cpp
while (first1 != last1) {
    *result = op(*first1, *first2);  // 取两个源的元素进行运算
    ++result;
    ++first1;
    ++first2;
}
return result;
```

---

## 78.3 一元变换：基础示例

### 示例 1：将元素乘以 2，结果存入新容器

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> src = {1, 2, 3, 4, 5};
    vector<int> dst(src.size());  // ⚠️ 必须预先分配空间！

    transform(src.begin(), src.end(), dst.begin(), [](int val) {
        return val * 2;
    });

    for (int x : dst) {
        cout << x << " ";
    }
    // 输出: 2 4 6 8 10

    cout << endl;
    return 0;
}
```

> ⚠️ **关键注意**：目标容器 `dst` 必须**预先分配足够的空间**！`transform` 不会自动扩容，它只是往 `result` 迭代器指向的位置写入数据。如果空间不够，就是**未定义行为**（危险！）。

### 三种预分配空间的方式

```cpp
// 方式 1：构造时指定大小
vector<int> dst(src.size());

// 方式 2：使用 resize
vector<int> dst;
dst.resize(src.size());

// 方式 3：使用 back_inserter（不需要预分配，自动 push_back）
vector<int> dst;
transform(src.begin(), src.end(), back_inserter(dst), [](int val) {
    return val * 2;
});
```

> 💡 `back_inserter` 定义在 `<iterator>` 头文件中，它会在每次赋值时调用容器的 `push_back`，因此**不需要预分配空间**。这是初学者最安全的方式。

---

### 示例 2：将字符串全部转为大写

```cpp
#include <iostream>
#include <string>
#include <algorithm>
#include <cctype>
using namespace std;

int main() {
    string src = "Hello, World!";
    string dst(src.size(), ' ');  // 预分配空间

    // transform(src.begin(), src.end(), dst.begin(),
          //[](unsigned char c) { return toupper(c); });
    transform(src.begin(), src.end(), dst.begin(), ::toupper);

    cout << dst << endl;  // 输出: HELLO, WORLD!
    return 0;
}
```

> 📌 `::toupper` 是 C 标准库中的函数（前面的 `::` 表示全局命名空间），它将小写字母转为大写。`transform` 对字符串的每个字符调用 `toupper`，结果写入 `dst`。

---

### 示例 3：原地变换（source 和 destination 相同）

`transform` 允许**输出迭代器指向源容器本身**，实现"原地修改"：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, -2, 3, -4, 5};

    // 将所有元素取绝对值，结果写回自身
    transform(v.begin(), v.end(), v.begin(), [](int val) {
        return val < 0 ? -val : val;
    });

    for (int x : v) {
        cout << x << " ";
    }
    // 输出: 1 2 3 4 5

    cout << endl;
    return 0;
}
```

> 📌 这是完全合法的用法。当 `result == first1` 时，每个元素被读取、变换，然后写回原位。

---

## 78.4 二元变换：两个容器合并运算

二元版本的 `transform` 从**两个源范围**各取一个元素，执行二元操作，将结果写入目标：

### 示例 1：两个向量对应元素相加

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> a = {1, 2, 3, 4, 5};
    vector<int> b = {10, 20, 30, 40, 50};
    vector<int> result(a.size());

    // 二元变换：a[i] + b[i] -> result[i]
    transform(a.begin(), a.end(), b.begin(), result.begin(),
              [](int x, int y) {
                  return x + y;
              });

    for (int x : result) {
        cout << x << " ";
    }
    // 输出: 11 22 33 44 55

    cout << endl;
    return 0;
}
```

> ⚠️ **注意**：二元变换只指定了第一个范围的 `[first1, last1)`，第二个范围只提供了起始迭代器 `first2`。因此你必须确保**第二个范围的元素数量 ≥ 第一个范围的元素数量**，否则是未定义行为。

### 示例 2：使用标准库函数对象

C++ 标准库在 `<functional>` 中提供了一系列预定义的函数对象：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>  // plus, minus, multiplies 等
using namespace std;

int main() {
    vector<int> a = {10, 20, 30};
    vector<int> b = {1, 2, 3};
    vector<int> result(a.size());

    // 使用 std::multiplies<int>()：a[i] * b[i]
    transform(a.begin(), a.end(), b.begin(), result.begin(),
              multiplies<int>());

    for (int x : result) {
        cout << x << " ";
    }
    // 输出: 10 40 90

    cout << endl;
    return 0;
}
```

**常用标准库函数对象：**

| 函数对象          | 操作    | 示例       |
| ----------------- | ------- | ---------- |
| `plus<T>()`       | `a + b` | 两元素相加 |
| `minus<T>()`      | `a - b` | 两元素相减 |
| `multiplies<T>()` | `a * b` | 两元素相乘 |
| `divides<T>()`    | `a / b` | 两元素相除 |
| `modulus<T>()`    | `a % b` | 取模       |
| `negate<T>()`     | `-a`    | 一元取反   |

---

## 78.5 `transform` 的类型转换能力

`transform` 的一个强大特性是**输入和输出可以是不同类型**：

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

int main() {
    vector<int> numbers = {1, 2, 3, 4, 5};
    vector<string> strings;

    // int -> string 的类型转换
    transform(numbers.begin(), numbers.end(), back_inserter(strings),
              [](int val) -> string {
                  return "No." + to_string(val);
              });

    for (const auto& s : strings) {
        cout << s << " ";
    }
    // 输出: No.1 No.2 No.3 No.4 No.5

    cout << endl;
    return 0;
}
```

> 📌 Lambda 的返回类型 `-> string` 可以省略（编译器能自动推导），但写上更清晰。

---

## 78.6 `transform` 实战：成绩处理系统

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <iterator>
using namespace std;

struct Student {
    string name;
    int score;
};

int main() {
    vector<Student> students = {
        {"Alice", 85}, {"Bob", 92}, {"Charlie", 78},
        {"Diana", 96}, {"Eve", 63}
    };

    // 1. 提取所有分数
    vector<int> scores;
    transform(students.begin(), students.end(), back_inserter(scores),
              [](const Student& s) { return s.score; });

    cout << "所有分数: ";
    for (int s : scores) cout << s << " ";
    cout << endl;  // 85 92 78 96 63

    // 2. 将分数转换为等级
    vector<string> grades;
    transform(scores.begin(), scores.end(), back_inserter(grades),
              [](int score) -> string {
                  if (score >= 90) return "A";
                  if (score >= 80) return "B";
                  if (score >= 70) return "C";
                  if (score >= 60) return "D";
                  return "F";
              });

    cout << "等级: ";
    for (const auto& g : grades) cout << g << " ";
    cout << endl;  // B A C A D

    // 3. 分数加分（每人加 5 分，封顶 100）
    transform(scores.begin(), scores.end(), scores.begin(),
              [](int score) { return min(score + 5, 100); });

    cout << "加分后: ";
    for (int s : scores) cout << s << " ";
    cout << endl;  // 90 97 83 100 68

    return 0;
}
```

---

## 78.7 `for_each` vs `transform` 的选择指南

```
你是否需要产生一个新的结果序列？
│
├── 是 → 用 transform ✅
│         （将每个元素转换后写入目标位置）
│
└── 否 → 你是否只是想"对每个元素做点什么"？
          │
          ├── 是 → 用 for_each ✅
          │        （打印、累加、修改原容器等副作用操作）
          │
          └── 否 → 考虑其他算法
```

**深入理解两者的语义差异：**

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// for_each：强调"执行动作"（副作用）
for_each(v.begin(), v.end(), [](int& val) {
    val *= 2;  // 直接修改原容器
});

// transform：强调"产生结果"（转换映射）
vector<int> result(v.size());
transform(v.begin(), v.end(), result.begin(), [](int val) {
    return val * 2;  // 产生新值写入新位置
});
```

虽然两者都能实现"乘以 2"，但语义不同：
- `for_each` 说的是"对每个元素**做**一件事"
- `transform` 说的是"把每个元素**变成**另一个东西"

---

## 78.8 小结

```
transform 核心要点：
┌──────────────────────────────────────────────────┐
│ 1. 头文件: #include <algorithm>                  │
│ 2. 两个版本:                                      │
│    ├── 一元变换: transform(f1, l1, result, op)    │
│    └── 二元变换: transform(f1, l1, f2, result, op)│
│ 3. 目标容器必须有足够空间（或用 back_inserter）   │
│ 4. 支持原地变换（result == first1）               │
│ 5. 输入输出可以是不同类型                         │
│ 6. 语义：转换映射，产生新的结果序列               │
└──────────────────────────────────────────────────┘
```

---

# 79. 常用遍历算法 - `find`

## 79.1 `find` 家族概述

`find` 是 STL 中最基本的**查找算法**。严格来说，它属于"查找算法"而不是"遍历算法"，但因为它的底层实现就是**线性遍历**，所以放在这里一起讲。

STL 提供了一个 `find` 家族：

| 算法            | 功能                                  | 头文件        |
| --------------- | ------------------------------------- | ------------- |
| `find`          | 查找等于某个值的元素                  | `<algorithm>` |
| `find_if`       | 查找满足某个条件的元素                | `<algorithm>` |
| `find_if_not`   | 查找**不满足**某个条件的元素（C++11） | `<algorithm>` |
| `find_first_of` | 查找第一个属于某集合的元素            | `<algorithm>` |
| `adjacent_find` | 查找第一对相邻重复元素                | `<algorithm>` |

我们逐一详解。

---

## 79.2 `find`：按值查找

### 函数原型

```cpp
template<class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T& value);
```

| 参数            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| `first`, `last` | 查找范围 `[first, last)`                                     |
| `value`         | 要查找的值                                                   |
| **返回值**      | 指向**第一个**等于 `value` 的元素的迭代器；如果没找到，返回 `last` |

**内部逻辑等价于：**

```cpp
while (first != last) {
    if (*first == value)  // 使用 == 运算符比较
        return first;
    ++first;
}
return last;  // 没找到
```

### 基础示例

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // 查找值为 30 的元素
    auto it = find(v.begin(), v.end(), 30);

    if (it != v.end()) {
        cout << "找到了: " << *it << endl;          // 输出: 找到了: 30
        cout << "位置索引: " << (it - v.begin()) << endl;  // 输出: 位置索引: 2
    } else {
        cout << "没找到" << endl;
    }

    // 查找一个不存在的值
    auto it2 = find(v.begin(), v.end(), 99);
    if (it2 == v.end()) {
        cout << "99 不存在于容器中" << endl;  // 会执行这里
    }

    return 0;
}
```

> 📌 **关键模式**：`find` 返回迭代器。**永远要检查返回值是否等于 `end()`**，这是 STL 的惯用法，表示"没找到"。

### 注意：`it - v.begin()` 求索引

```cpp
auto it = find(v.begin(), v.end(), 30);
int index = it - v.begin();  // 只适用于随机访问迭代器（如 vector、deque）
```

对于 `list`、`set` 等非随机访问迭代器的容器，用 `distance`：

```cpp
int index = distance(v.begin(), it);  // 适用于所有迭代器
```

---

## 79.3 `find` 查找自定义类型

当你对自定义类型使用 `find` 时，该类型**必须重载 `operator==`**：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

class Person {
public:
    string name;
    int age;

    Person(const string& n, int a) : name(n), age(a) {}

    // 必须重载 == 运算符，find 才能工作
    bool operator==(const Person& other) const {
        return name == other.name && age == other.age;
    }
};

int main() {
    vector<Person> people = {
        Person("Alice", 25),
        Person("Bob", 30),
        Person("Charlie", 35)
    };

    Person target("Bob", 30);
    auto it = find(people.begin(), people.end(), target);

    if (it != people.end()) {
        cout << "找到: " << it->name << ", 年龄: " << it->age << endl;
        // 输出: 找到: Bob, 年龄: 30
    }

    return 0;
}
```

> ⚠️ 如果你不重载 `operator==`，编译器会报错，因为 `find` 内部使用 `==` 来比较元素。

---

## 79.4 `find_if`：按条件查找

`find_if` 不是按值比较，而是按**谓词（条件函数）**查找。

### 函数原型

```cpp
template<class InputIterator, class Predicate>
InputIterator find_if(InputIterator first, InputIterator last, Predicate pred);
```

**内部逻辑：**

```cpp
while (first != last) {
    if (pred(*first))  // 调用谓词，如果返回 true 就找到了
        return first;
    ++first;
}
return last;
```

### 示例 1：查找第一个偶数

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 4, 7, 8};

    auto it = find_if(v.begin(), v.end(), [](int val) {
        return val % 2 == 0;  // 条件：是偶数
    });

    if (it != v.end()) {
        cout << "第一个偶数: " << *it << endl;  // 输出: 第一个偶数: 4
        cout << "索引: " << (it - v.begin()) << endl;  // 输出: 索引: 3
    }

    return 0;
}
```

### 示例 2：在自定义类型中按条件查找

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Student {
    string name;
    int score;
};

int main() {
    vector<Student> students = {
        {"Alice", 78}, {"Bob", 92}, {"Charlie", 85}, {"Diana", 96}
    };

    // 查找第一个分数 >= 90 的学生
    auto it = find_if(students.begin(), students.end(),
                      [](const Student& s) {
                          return s.score >= 90;
                      });

    if (it != students.end()) {
        cout << it->name << " 的分数是 " << it->score << endl;
        // 输出: Bob 的分数是 92
    }

    return 0;
}
```

> 📌 `find_if` 的优势在于它**不需要重载 `operator==`**，你可以用 Lambda 定义任意复杂的查找条件。

---

## 79.5 `find_if_not`：查找第一个不满足条件的元素

`find_if_not`（C++11）是 `find_if` 的反义词：

```cpp
template<class InputIterator, class Predicate>
InputIterator find_if_not(InputIterator first, InputIterator last, Predicate pred);
```

**内部逻辑：**

```cpp
while (first != last) {
    if (!pred(*first))  // 注意这里是取反！
        return first;
    ++first;
}
return last;
```

### 示例：找出第一个不及格的成绩

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> scores = {85, 92, 78, 55, 90};

    // 找到第一个"不及格"的（即不满足 >= 60 的）
    auto it = find_if_not(scores.begin(), scores.end(), [](int s) {
        return s >= 60;  // 条件：及格
    });

    if (it != scores.end()) {
        cout << "第一个不及格的分数: " << *it << endl;  // 输出: 55
    }

    return 0;
}
```

> 💡 **`find_if_not` vs `find_if`**：
> - `find_if(pred)` = 找第一个让 `pred` 返回 `true` 的
> - `find_if_not(pred)` = 找第一个让 `pred` 返回 `false` 的
> - 两者等价：`find_if_not(f, l, pred)` 等同于 `find_if(f, l, [&](auto& x){ return !pred(x); })`

---

## 79.6 `adjacent_find`：查找相邻重复元素

### 函数原型

```cpp
// 版本 1：查找相邻的两个相等元素
template<class ForwardIterator>
ForwardIterator adjacent_find(ForwardIterator first, ForwardIterator last);

// 版本 2：查找满足条件的相邻元素对
template<class ForwardIterator, class BinaryPredicate>
ForwardIterator adjacent_find(ForwardIterator first, ForwardIterator last,
                              BinaryPredicate pred);
```

### 示例 1：查找相邻重复元素

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 3, 4, 4, 5};

    auto it = adjacent_find(v.begin(), v.end());

    if (it != v.end()) {
        cout << "第一对相邻重复元素: " << *it << endl;  // 输出: 3
        cout << "索引: " << (it - v.begin()) << endl;    // 输出: 2
    }

    return 0;
}
```

> 📌 返回的迭代器指向**第一对相邻重复元素中的第一个**。

### 示例 2：用谓词查找相邻关系

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 2, 8, 5, 7};

    // 查找第一对相邻元素中，前一个 > 后一个的情况（即降序相邻对）
    auto it = adjacent_find(v.begin(), v.end(), [](int a, int b) {
        return a > b;  // 前一个大于后一个
    });

    if (it != v.end()) {
        cout << "第一个降序对: " << *it << " 和 " << *(it + 1) << endl;
        // 输出: 第一个降序对: 3 和 2
    }

    return 0;
}
```

---

## 79.7 `find_first_of`：查找任意一个匹配元素

`find_first_of` 在第一个范围中查找**第一个出现在第二个范围中的元素**。

### 函数原型

```cpp
template<class ForwardIterator1, class ForwardIterator2>
ForwardIterator1 find_first_of(ForwardIterator1 first1, ForwardIterator1 last1,
                               ForwardIterator2 first2, ForwardIterator2 last2);
```

### 示例

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> data = {1, 5, 3, 7, 9, 2, 8};
    vector<int> targets = {2, 7, 10};  // 我们要找 data 中第一个属于 targets 的

    auto it = find_first_of(data.begin(), data.end(),
                            targets.begin(), targets.end());

    if (it != data.end()) {
        cout << "data 中第一个属于 targets 的元素: " << *it << endl;
        // 输出: 7（因为 7 在 data 中出现的位置比 2 更靠前）
        cout << "索引: " << (it - data.begin()) << endl;  // 输出: 3
    }

    return 0;
}
```

> 📌 这个算法是 O(n × m) 的复杂度，其中 n 是第一个范围的大小，m 是第二个范围的大小。

---

## 79.8 `find` 在不同容器中的表现

### 重要：关联容器请用成员函数 `find`！

```cpp
#include <iostream>
#include <set>
#include <map>
#include <algorithm>
using namespace std;

int main() {
    set<int> s = {1, 2, 3, 4, 5};

    // ❌ 不推荐：使用 std::find（算法版本）→ O(n) 线性查找
    auto it1 = find(s.begin(), s.end(), 3);

    // ✅ 推荐：使用 set 的成员函数 find → O(log n) 对数查找
    auto it2 = s.find(3);

    // 对于 unordered_set / unordered_map 更是如此
    // 成员函数 find 的复杂度是 O(1) 平均，而 std::find 是 O(n)

    // map 也一样
    map<string, int> m = {{"a", 1}, {"b", 2}};

    // ❌ 不推荐
    auto it3 = find_if(m.begin(), m.end(), [](const pair<const string, int>& p) {
        return p.first == "b";
    });

    // ✅ 推荐
    auto it4 = m.find("b");

    return 0;
}
```

**各容器查找方式对比：**

| 容器               | `std::find`（算法） | 成员函数 `find` | 推荐方式    |
| ------------------ | ------------------- | --------------- | ----------- |
| `vector`           | O(n) ✅              | 无              | `std::find` |
| `deque`            | O(n) ✅              | 无              | `std::find` |
| `list`             | O(n) ✅              | 无              | `std::find` |
| `set` / `multiset` | O(n) ❌              | **O(log n) ✅**  | 成员函数    |
| `map` / `multimap` | O(n) ❌              | **O(log n) ✅**  | 成员函数    |
| `unordered_set`    | O(n) ❌              | **O(1) 平均 ✅** | 成员函数    |
| `unordered_map`    | O(n) ❌              | **O(1) 平均 ✅** | 成员函数    |

> ⚠️ **这是一个常见的性能陷阱！** 对于关联容器和无序容器，一定要使用其**成员函数** `find`，而不是全局的 `std::find` 算法。

---

## 79.9 `find` 家族综合实战

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

struct Employee {
    string name;
    string department;
    int salary;
};

int main() {
    vector<Employee> employees = {
        {"Alice",   "Engineering", 8000},
        {"Bob",     "Marketing",   6000},
        {"Charlie", "Engineering", 9500},
        {"Diana",   "HR",          7000},
        {"Eve",     "Engineering", 8500},
        {"Frank",   "Marketing",   6000}
    };

    // 1. find_if：查找第一个工程部的员工
    auto it1 = find_if(employees.begin(), employees.end(),
                       [](const Employee& e) {
                           return e.department == "Engineering";
                       });
    if (it1 != employees.end()) {
        cout << "第一个工程部员工: " << it1->name << endl;
        // 输出: Alice
    }

    // 2. find_if：查找薪资最高超过 9000 的员工
    auto it2 = find_if(employees.begin(), employees.end(),
                       [](const Employee& e) {
                           return e.salary > 9000;
                       });
    if (it2 != employees.end()) {
        cout << "第一个薪资超过 9000 的: " << it2->name
             << "，薪资: " << it2->salary << endl;
        // 输出: Charlie，薪资: 9500
    }

    // 3. find_if_not：查找第一个不在工程部的员工
    auto it3 = find_if_not(employees.begin(), employees.end(),
                           [](const Employee& e) {
                               return e.department == "Engineering";
                           });
    if (it3 != employees.end()) {
        cout << "第一个非工程部: " << it3->name
             << "（" << it3->department << "）" << endl;
        // 输出: Bob（Marketing）
    }

    // 4. adjacent_find：查找相邻的同薪资员工
    auto it4 = adjacent_find(employees.begin(), employees.end(),
                             [](const Employee& a, const Employee& b) {
                                 return a.salary == b.salary;
                             });
    if (it4 != employees.end()) {
        auto next = it4 + 1;
        cout << "相邻同薪资: " << it4->name << " 和 " << next->name
             << "，薪资都是 " << it4->salary << endl;
        // 这取决于数据顺序，本例不一定能找到
    }

    // 5. 查找所有工程部员工（find_if 的循环版）
    cout << "\n所有工程部员工:" << endl;
    auto it = employees.begin();
    while ((it = find_if(it, employees.end(),
                         [](const Employee& e) {
                             return e.department == "Engineering";
                         })) != employees.end()) {
        cout << "  " << it->name << "，薪资: " << it->salary << endl;
        ++it;  // 移到下一个位置继续找
    }
    // 输出:
    //   Alice，薪资: 8000
    //   Charlie，薪资: 9500
    //   Eve，薪资: 8500

    return 0;
}
```

> 📌 **第 5 个例子非常重要！** `find_if` 只返回第一个匹配的，如果你要找**所有**匹配的，需要在循环中不断调用 `find_if`，每次从上一次找到的位置的下一个开始。

---

## 79.10 `find` 家族的复杂度

| 算法            | 时间复杂度 | 说明                       |
| --------------- | ---------- | -------------------------- |
| `find`          | O(n)       | 线性扫描                   |
| `find_if`       | O(n)       | 线性扫描                   |
| `find_if_not`   | O(n)       | 线性扫描                   |
| `adjacent_find` | O(n)       | 最多比较 n-1 对            |
| `find_first_of` | O(n × m)   | n 为第一范围，m 为第二范围 |

> 如果你需要更高效的查找，考虑：
> - 有序容器 → `lower_bound` / `upper_bound`（O(log n)）
> - 无序容器 → 成员函数 `find`（O(1) 平均）
> - 有序数组 → `binary_search`（O(log n)）

---

## 79.11 小结

```
find 家族核心要点：
┌──────────────────────────────────────────────────────────┐
│ 1. find(f, l, val)      → 按值查找（需要 operator==）    │
│ 2. find_if(f, l, pred)  → 按条件查找（最灵活 ⭐）        │
│ 3. find_if_not(f, l, pred) → 找第一个不满足条件的        │
│ 4. adjacent_find(f, l)  → 找相邻重复元素                 │
│ 5. find_first_of(...)   → 找第一个属于某集合的元素        │
│                                                          │
│ 通用注意事项:                                             │
│ ├── 返回迭代器，始终检查 != end()                         │
│ ├── 关联/无序容器用成员函数 find（性能！）                │
│ ├── 要找"所有"匹配的，需要循环调用 find_if                │
│ └── 这些都是线性 O(n) 算法                               │
└──────────────────────────────────────────────────────────┘
```

---

# 综合对比与总结

## 三大算法一览表

| 特性                 | `for_each`           | `transform`        | `find` / `find_if`   |
| -------------------- | -------------------- | ------------------ | -------------------- |
| **目的**             | 遍历并执行操作       | 遍历并转换映射     | 遍历并查找           |
| **是否需要输出容器** | ❌                    | ✅                  | ❌                    |
| **返回值**           | 可调用对象本身       | 输出范围的尾迭代器 | 找到的元素的迭代器   |
| **能否修改原容器**   | ✅（引用参数）        | ✅（原地变换）      | ❌（只读查找）        |
| **典型用途**         | 打印、累加、原地修改 | 类型转换、数学变换 | 条件搜索、存在性检查 |
| **复杂度**           | O(n)                 | O(n)               | O(n)（找到即停）     |

## 选择指南

```
你想对容器中的元素做什么？
│
├── "我想对每个元素做一个操作（打印/修改/累加）"
│   └── for_each ✅
│
├── "我想把每个元素转换成另一种形式"
│   └── transform ✅
│
├── "我想找到第一个满足条件的元素"
│   └── find / find_if ✅
│
└── "我想知道这个值在不在容器里"
    ├── 无序/关联容器 → 成员函数 find / count ✅
    └── 序列容器 → std::find + 检查 != end() ✅
```

---

希望这份教程能帮助你建立对 STL 遍历算法的全面认知！这三个算法是 STL 算法库中最基础也是最常用的，掌握它们之后，你会发现学习后续的 `sort`、`accumulate`、`count`、`copy` 等算法会变得非常自然。