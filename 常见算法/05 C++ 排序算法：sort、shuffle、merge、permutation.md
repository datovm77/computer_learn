# 常用排序算法

> **适用读者**：STL 初学者 · C++11 及以上 · 已了解迭代器基本概念

---

## 写在前面：排序算法在 STL 中的位置

C++ STL 的算法库 `<algorithm>` 提供了一整套经过高度优化的排序相关工具。与手写排序相比，STL 排序算法：

- **性能更强**：`std::sort` 内部采用 Introsort（快排 + 堆排 + 插排混合），极端情况下不会退化
- **接口统一**：全部基于迭代器区间 `[first, last)` 操作，对各类容器通用
- **语义清晰**：每个函数名字就说明了它要干什么

在学习具体函数之前，先牢记一个约定：

```
[first, last)  ←  左闭右开区间
```

`first` 指向第一个元素，`last` 指向**最后一个元素的下一个位置**，这是 STL 的统一惯例。

---

## 85 · `std::sort` — 通用排序

### 1. 最基础的用法

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {5, 3, 8, 1, 9, 2, 7, 4, 6};

    // 默认升序（从小到大）
    sort(v.begin(), v.end());

    for (int x : v) cout << x << " ";
    // 输出：1 2 3 4 5 6 7 8 9
    cout << endl;
    return 0;
}
```

**函数签名**：

```cpp
// 版本一：使用默认 operator< 升序排列
template< class RandomIt >
void sort( RandomIt first, RandomIt last );

// 版本二：使用自定义比较函数
template< class RandomIt, class Compare >
void sort( RandomIt first, RandomIt last, Compare comp );
```

> ⚠️ **重要限制**：`sort` 要求**随机访问迭代器（RandomAccessIterator）**。`vector`、`deque`、`array`、原生数组可以用；`list`、`set`、`map` **不能用**（`list` 有自己的 `list::sort()`）。

---

### 2. 对数组排序

`sort` 同样可以直接操作原生数组，传入指针即可：

```cpp
#include <algorithm>
#include <iostream>
using namespace std;

int main() {
    int arr[] = {4, 2, 7, 1, 5};
    int n = sizeof(arr) / sizeof(arr[0]);

    sort(arr, arr + n);  // arr 是首地址，arr+n 是尾后地址

    for (int i = 0; i < n; i++) cout << arr[i] << " ";
    // 输出：1 2 4 5 7
    return 0;
}
```

---

### 3. 降序排序 — 使用 `greater<T>`

STL 提供了现成的**函数对象（functor）**`greater<T>`，用于反转比较：

```cpp
#include <algorithm>
#include <vector>
#include <functional>  // greater 在这里
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {5, 3, 8, 1, 9};

    // greater<int>() 相当于 a > b，即降序
    sort(v.begin(), v.end(), greater<int>());

    for (int x : v) cout << x << " ";
    // 输出：9 8 5 3 1
    return 0;
}
```

> **💡 C++14 起**，可以写 `greater<>()` 省略模板参数（推导版本），更简洁。

---

### 4. 自定义比较函数

这是 `sort` 最强大的地方！只要能告诉它"谁排在前面"，它就能排出你想要的顺序。

**比较函数的规则**：`comp(a, b)` 返回 `true` 表示 `a` 应该排在 `b` **前面（更小）**。

#### 示例：按绝对值升序排序

```cpp
#include <algorithm>
#include <vector>
#include <cmath>
#include <iostream>
using namespace std;

// 方式一：普通函数
bool cmpByAbs(int a, int b) {
    return abs(a) < abs(b);
}

int main() {
    vector<int> v = {-5, 3, -1, 4, -2};

    sort(v.begin(), v.end(), cmpByAbs);

    for (int x : v) cout << x << " ";
    // 输出：-1 -2 3 4 -5
    return 0;
}
```

#### 示例：Lambda 表达式（最常用写法）

```cpp
#include <algorithm>
#include <vector>
#include <cmath>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {-5, 3, -1, 4, -2};

    // Lambda 直接写在 sort 调用里，简洁而直观
    sort(v.begin(), v.end(), [](int a, int b) {
        return abs(a) < abs(b);
    });

    for (int x : v) cout << x << " ";
    return 0;
}
```

---

### 5. 对结构体排序 — 最实用场景

实际项目中，我们经常需要对结构体按某个字段排序：

```cpp
#include <algorithm>
#include <vector>
#include <string>
#include <iostream>
using namespace std;

struct Student {
    string name;
    int score;
    int age;
};

int main() {
    vector<Student> students = {
        {"Alice", 90, 20},
        {"Bob",   85, 22},
        {"Carol", 90, 19},
        {"Dave",  78, 21}
    };

    // 按分数降序，分数相同则按年龄升序（多关键字排序）
    sort(students.begin(), students.end(), [](const Student& a, const Student& b) {
        if (a.score != b.score)
            return a.score > b.score;  // 分数不同：高分在前
        return a.age < b.age;          // 分数相同：年龄小的在前
    });

    for (auto& s : students) {
        cout << s.name << "\t" << s.score << "\t" << s.age << "\n";
    }
    // 输出：
    // Carol   90   19
    // Alice   90   20
    // Bob     85   22
    // Dave    78   21
    return 0;
}
```

---

### 6. 重载 `operator<` — 让结构体"天然可排序"

除了传 Lambda，也可以为结构体重载 `<` 运算符，这样直接调用 `sort(v.begin(), v.end())` 就行：

```cpp
struct Point {
    int x, y;

    // 重载 < ：先比 x，x 相同则比 y
    bool operator<(const Point& other) const {
        if (x != other.x) return x < other.x;
        return y < other.y;
    }
};

// 之后直接：
// sort(points.begin(), points.end());  // 自动用 operator<
```

---

### 7. `sort` 的时间复杂度

| 情况 | 复杂度                                         |
| ---- | ---------------------------------------------- |
| 平均 | O(N log N)                                     |
| 最坏 | O(N log N)（Introsort 保证，不会退化到 O(N²)） |
| 空间 | O(log N)（递归栈）                             |

> ⚡ **性能与安全提示**：
> 1. **移动语义**：现代 C++ 中 `sort` 会优先使用移动操作（Move）交换元素。确保自定义类型实现了高效的**移动构造函数**和**移动赋值运算符**。
> 2. **异常安全性**：若比较函数或元素的拷贝/移动操作抛出异常，区间内的元素依然有效，但其**顺序变为未指定状态**；也就是说，元素不会凭空消失，但你不能再假定它们保持原顺序或部分有序。

> ⚠️ **比较函数必须满足严格弱序（strict weak ordering）**：
> 1. **不自反**：`comp(a, a)` 必须为 `false`
> 2. **非对称**：若 `comp(a, b)` 为 `true`，则 `comp(b, a)` 必为 `false`
> 3. **传递性**：若 `comp(a, b)` 且 `comp(b, c)`，则 `comp(a, c)`
> 4. **不可比的传递性**：若 a 与 b 不可比（即互不小于），b 与 c 不可比，则 a 与 c 必然不可比。
>
> 违背这些规则（例如用了 `<=`），程序行为是**未定义的（UB）**，极易因迭代器越界而崩溃！

**常见错误示例**（`<=` 导致 UB）：

```cpp
// ❌ 错误！<= 不满足严格弱序
sort(v.begin(), v.end(), [](int a, int b){ return a <= b; });

// ✅ 正确！只用 <
sort(v.begin(), v.end(), [](int a, int b){ return a < b; });
```

---

### 8. 检查是否已排好序

在处理排序问题或调试时，STL 提供了两个实用的检查函数（都在 `<algorithm>` 中）：

- **`std::is_sorted(first, last)`**：返回 `bool`，检查整个区间是否已按升序（或自定义比较器）排好。
- **`std::is_sorted_until(first, last)`**：返回一个迭代器，指向**最长已排序前缀**的末尾（即第一个破坏顺序的元素位置）。若完全有序，返回 `last`。

```cpp
vector<int> v = {1, 2, 4, 3, 5};
bool ok = is_sorted(v.begin(), v.end());  // false
auto bad_pos = is_sorted_until(v.begin(), v.end()); 
// *bad_pos 当前指向 3
```

---

### 9. C++20 引入的 `std::ranges::sort`（前瞻扩展）

C++20 提供了基于 Ranges 的算法，彻底省去了反复写 `.begin()` 和 `.end()` 的麻烦，并原生支持**投影（Projections）**：

```cpp
// 传统 C++17 写法：
sort(students.begin(), students.end(), [](const Student& a, const Student& b){ return a.age < b.age; });

// 现代 C++20 Ranges 写法：
// 第三个参数 &Student::age 是投影函数，它自动取出每个元素的 age 来进行比较
std::ranges::sort(students, {}, &Student::age); 
```

---

## 86 · `std::random_shuffle` / `std::shuffle` — 随机打乱

### 1. 背景与区别

STL 提供两个打乱函数：

| 函数             | 头文件        | 状态             | 说明                         |
| ---------------- | ------------- | ---------------- | ---------------------------- |
| `random_shuffle` | `<algorithm>` | **C++14弃用，C++17移除** | 使用 C 的 `rand()`，随机性差 |
| `shuffle`        | `<algorithm>` | ✅ 推荐           | 使用现代随机引擎，质量更高   |

---

### 2. `random_shuffle`（了解即可，了解它的问题）

```cpp
#include <algorithm>
#include <vector>
#include <cstdlib>  // srand, rand
#include <ctime>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};

    srand(time(nullptr));              // 先设置随机种子
    random_shuffle(v.begin(), v.end()); // 打乱

    for (int x : v) cout << x << " ";
    return 0;
}
```

**为什么被废弃/移除？**
- 内部依赖全局状态 `rand()`，线程不安全
- `rand()` 的随机质量很差，分布不均匀
- 在 **C++14 标准中被标记为废弃（deprecated）**，而在 **C++17 标准中被彻底移除（removed）**。在现代编译器中轻则收到警告，重则直接编译报错提示找不到该函数！

---

### 3. `std::shuffle`（现代写法，强烈推荐）

```cpp
#include <algorithm>
#include <vector>
#include <random>   // 现代随机库
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8};

    // 创建随机引擎（Mersenne Twister，质量极高）
    // random_device{} 获取真随机种子（来自硬件或系统熵）
    mt19937 rng(random_device{}());

    // shuffle 需要传入随机引擎
    shuffle(v.begin(), v.end(), rng);

    for (int x : v) cout << x << " ";
    return 0;
}
```

**关键组成部分：**

```
mt19937 rng(random_device{}());
   │           │
   │           └─ random_device：真随机种子（硬件熵）
   └─ mt19937：Mersenne Twister 伪随机数生成器
              周期长达 2^19937-1，远超实际需要
```

---

### 4. 固定种子 — 让随机结果可复现（用于调试）

```cpp
mt19937 rng(42);  // 固定种子 42
shuffle(v.begin(), v.end(), rng);
// 每次运行结果相同，方便调试
```

---

### 5. 实际应用：洗牌算法

```cpp
#include <algorithm>
#include <vector>
#include <random>
#include <numeric>  // iota
#include <iostream>
using namespace std;

// 模拟一副扑克牌（1~52）
int main() {
    vector<int> deck(52);
    iota(deck.begin(), deck.end(), 1);  // 填充 1, 2, ..., 52

    mt19937 rng(random_device{}());
    shuffle(deck.begin(), deck.end(), rng);

    cout << "前5张牌: ";
    for (int i = 0; i < 5; i++) cout << deck[i] << " ";
    return 0;
}
```

> `iota` 来自 `<numeric>`，用于填充连续序列，非常好用。

---

### 6. 时间复杂度

| 函数             | 复杂度 |
| ---------------- | ------ |
| `random_shuffle` | O(N)   |
| `shuffle`        | O(N)   |

---

## 87 · `std::merge` — 合并两个有序序列

### 1. 基本概念

`merge` 就是**归并排序中的"归并"步骤**：将两个**已经有序**的序列合并成一个有序序列。

```
序列 A：1  3  5  7
序列 B：2  4  6  8
合并后：1  2  3  4  5  6  7  8
```

**函数签名**：

```cpp
template< class InputIt1, class InputIt2, class OutputIt >
OutputIt merge( InputIt1 first1, InputIt1 last1,
                InputIt2 first2, InputIt2 last2,
                OutputIt d_first );

// 带自定义比较器版本
template< class InputIt1, class InputIt2, class OutputIt, class Compare >
OutputIt merge( InputIt1 first1, InputIt1 last1,
                InputIt2 first2, InputIt2 last2,
                OutputIt d_first, Compare comp );
```

> **返回值**：指向**输出结果末尾的下一个位置**的迭代器。

---

### 2. 基础示例

```cpp
#include <algorithm>
#include <iterator>   // distance
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> a = {1, 3, 5, 7, 9};    // 必须已排序！
    vector<int> b = {2, 4, 6, 8, 10};   // 必须已排序！

    vector<int> result(a.size() + b.size());  // 预留足够空间

    // 合并 a 和 b，结果写入 result
    merge(a.begin(), a.end(),
          b.begin(), b.end(),
          result.begin());

    for (int x : result) cout << x << " ";
    // 输出：1 2 3 4 5 6 7 8 9 10
    return 0;
}
```

---

### 3. 使用 `back_inserter` — 不用预分配大小

上面的写法需要手动计算 `result` 的大小。用 `back_inserter` 可以让结果自动追加：

```cpp
#include <algorithm>
#include <vector>
#include <iterator>  // back_inserter
#include <iostream>
using namespace std;

int main() {
    vector<int> a = {1, 3, 5};
    vector<int> b = {2, 4, 6, 7, 8};

    vector<int> result;
    // back_inserter 会自动调用 result.push_back()
    merge(a.begin(), a.end(),
          b.begin(), b.end(),
          back_inserter(result));

    for (int x : result) cout << x << " ";
    // 输出：1 2 3 4 5 6 7 8
    return 0;
}
```

---

### 4. 自定义比较器

两个序列必须按**同样的规则**排好序，merge 才能正确合并：

```cpp
#include <algorithm>
#include <vector>
#include <iterator>
#include <functional>
#include <iostream>
using namespace std;

int main() {
    // 两个序列都是降序
    vector<int> a = {9, 7, 5, 3, 1};
    vector<int> b = {10, 8, 6, 4, 2};

    vector<int> result;
    // 合并时也使用降序比较器
    merge(a.begin(), a.end(),
          b.begin(), b.end(),
          back_inserter(result),
          greater<int>());

    for (int x : result) cout << x << " ";
    // 输出：10 9 8 7 6 5 4 3 2 1
    return 0;
}
```

---

### 5. ⚠️ 重要警告：输入必须已排好序

```cpp
vector<int> a = {3, 1, 5};  // ❌ 未排序！
vector<int> b = {2, 4, 6};  // ✅ 已排序

vector<int> result;
merge(a.begin(), a.end(), b.begin(), b.end(), back_inserter(result));
// 结果是错的！merge 假设两个序列都已有序
```

**解决方案**：先 `sort`，再 `merge`。

---

### 6. `inplace_merge` — 原地合并

如果两段有序序列**在同一个容器内相邻**，可以用 `inplace_merge` 就地合并。

> ⚠️ **迭代器要求**：`inplace_merge` 至少要求**双向迭代器（BidirectionalIterator）**，因此 `list`、`deque`、`vector` 都可以；但两段区间必须已经各自按同一规则排好序。

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    // v 的前半段和后半段各自有序
    vector<int> v = {1, 3, 5, 2, 4, 6};
    //               [前半段]  [后半段]

    auto mid = v.begin() + 3;  // 中间位置

    inplace_merge(v.begin(), mid, v.end());

    for (int x : v) cout << x << " ";
    // 输出：1 2 3 4 5 6
    return 0;
}
```

---

### 7. 时间复杂度

| 函数            | 时间            | 空间                             |
| --------------- | --------------- | -------------------------------- |
| `merge`         | O(N+M)          | O(N+M)（需要额外输出空间）       |
| `inplace_merge` | **O(N)**（有足够内存时）<br>**O(N log N)**（内存不足时） | O(log N) 或 O(N)（有无额外内存） |

其中 N 和 M 分别是两个序列的长度。

---

## 88 · `std::reverse` — 翻转序列

### 1. 基础用法

`reverse` 是最简单直接的算法之一：将区间内的元素原地翻转。

> ⚠️ **迭代器要求**：`reverse` 要求操作的容器支持**双向迭代器（BidirectionalIterator）**。`vector`、`list`、`deque` 等均可，但不支持单向迭代器如 `forward_list`。


```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5};

    reverse(v.begin(), v.end());

    for (int x : v) cout << x << " ";
    // 输出：5 4 3 2 1
    return 0;
}
```

---

### 2. 翻转字符串

```cpp
#include <algorithm>
#include <string>
#include <iostream>
using namespace std;

int main() {
    string s = "Hello, World!";

    reverse(s.begin(), s.end());

    cout << s << "\n";  // 输出：!dlroW ,olleH
    return 0;
}
```

---

### 3. 翻转部分区间

`reverse` 可以只翻转序列中的**一段**，只需调整迭代器范围：

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5, 6, 7};

    // 只翻转索引 2 到 4（即第 3、4、5 个元素）
    reverse(v.begin() + 2, v.begin() + 5);

    for (int x : v) cout << x << " ";
    // 原来：1 2 [3 4 5] 6 7
    // 翻转：1 2 [5 4 3] 6 7
    return 0;
}
```

---

### 4. `reverse_copy` — 翻转并复制到新容器

若不想修改原容器，可用 `reverse_copy`：

```cpp
#include <algorithm>
#include <vector>
#include <iterator>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5};
    vector<int> result;

    reverse_copy(v.begin(), v.end(), back_inserter(result));

    cout << "原序列: "; for (int x : v)      cout << x << " "; cout << "\n";
    cout << "新序列: "; for (int x : result) cout << x << " "; cout << "\n";
    // 原序列: 1 2 3 4 5
    // 新序列: 5 4 3 2 1
    return 0;
}
```

---

### 5. 实际应用：利用 reverse 实现"旋转"

一个经典技巧：将数组向右旋转 k 位，等价于三次翻转：

```cpp
// 将 v 向右旋转 k 位
// 例如 {1,2,3,4,5}，k=2 → {4,5,1,2,3}
void rotateRight(vector<int>& v, int k) {
    int n = v.size();
    if (n == 0) return;  // 空容器直接返回，避免 k %= n 除以 0
    k %= n;  // 处理 k >= n 的情况
    reverse(v.begin(),         v.end());       // 翻转整体
    reverse(v.begin(),         v.begin() + k); // 翻转前 k 个
    reverse(v.begin() + k,     v.end());       // 翻转后 n-k 个
}
```

---

### 6. 时间复杂度

| 函数           | 时间 | 空间         |
| -------------- | ---- | ------------ |
| `reverse`      | O(N) | O(1)（原地） |
| `reverse_copy` | O(N) | O(N)         |

---

## `[补]` 排序算法进阶

---

## `[补]` `std::stable_sort` — 稳定排序

### 1. 什么是"稳定性"？

设有两个元素 `a` 和 `b`，若 `a == b`（按排序关键字相等），排序后它们**在原序列中的相对顺序保持不变**，则称这个排序是**稳定的（stable）**。

**为什么重要？** 来看一个例子：

```
原序列（按姓名字母序存储）：
  Alice  90
  Bob    85
  Carol  90

按分数降序排序后：
  stable_sort：Alice 90 → Carol 90 → Bob 85  ✅ 原来 Alice 在 Carol 前，排完后依然如此
  sort：        Carol 90 → Alice 90 → Bob 85  ❓ 不保证（可能乱序）
```

---

### 2. 基础用法

```cpp
#include <algorithm>
#include <vector>
#include <string>
#include <iostream>
using namespace std;

struct Item {
    string name;
    int priority;
};

int main() {
    vector<Item> items = {
        {"task_A", 2},
        {"task_B", 1},
        {"task_C", 2},  // ← 和 task_A 同优先级，但在后面
        {"task_D", 3}
    };

    // 按优先级升序排，同优先级的保持原来的相对顺序
    stable_sort(items.begin(), items.end(), [](const Item& a, const Item& b) {
        return a.priority < b.priority;
    });

    for (auto& it : items) {
        cout << it.name << "(" << it.priority << ") ";
    }
    // 输出：task_B(1) task_A(2) task_C(2) task_D(3)
    //                    ↑           ↑
    //               A 仍然在 C 前面！这就是稳定性
    return 0;
}
```

---

### 3. `sort` vs `stable_sort`

| 对比项     | `sort`     | `stable_sort`                        |
| ---------- | ---------- | ------------------------------------ |
| 常见实现思路 | Introsort  | 稳定归并类实现（常见为归并排序变体） |
| 稳定性     | 不保证     | 保证                                 |
| 时间复杂度 | O(N log N) | **O(N log N)**（有足够额外内存时）<br>**O(N log² N)**（无足够内存时） |
| 空间复杂度 | O(log N)   | 通常需要 O(N) 额外空间               |

> **实践建议**：绝大多数情况用 `sort` 就够了。只有当你需要**明确保证相等元素顺序不变**时，才使用 `stable_sort`。不同标准库实现细节可能不同，但"稳定"这一语义保证不变。

---

### 4. 多关键字排序的稳定技巧

`stable_sort` 的稳定性可以用来实现"分阶段多关键字排序"：

```cpp
// 目标：先按分数降序，分数相同则按年龄升序
// 技巧：先按次要关键字（年龄升序）排一次，再按主要关键字（分数降序）稳定排一次

stable_sort(v.begin(), v.end(), [](const auto& a, const auto& b){ return a.age < b.age; });
stable_sort(v.begin(), v.end(), [](const auto& a, const auto& b){ return a.score > b.score; });
// 第二次排序稳定，所以分数相同时，第一次排好的年龄顺序会被保留
```

---

## `[补]` `std::partial_sort` — 部分排序

### 1. 使用场景

**问题**：给你 100 万条数据，只需要找出**前 10 名**（最小的 10 个），全排序太浪费。

`partial_sort` 的语义：**保证 `[first, middle)` 区间内是全局最小的那些元素，并且已排好序**；`[middle, last)` 区间的内容顺序不确定。

```cpp
partial_sort(first, middle, last);
//           ↑      ↑       ↑
//        起点   中间点    终点
// 排完后：[first, middle) 是最小的 (middle-first) 个元素，已升序排好
//         [middle, last)  是剩余元素，顺序不确定
```

> 💡 **内部原理**：通常使用**堆排序（Heap Sort）**实现。将 `[first, middle)` 组织成一个最大堆，然后遍历剩余元素，若比堆顶小则替换堆顶并维护堆，最后对堆进行整体排序。

---

### 2. 基础示例

```cpp
#include <algorithm>
#include <vector>
#include <functional>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {5, 3, 8, 1, 9, 2, 7, 4, 6};

    // 只需要最小的前 3 个，并排好序
    partial_sort(v.begin(), v.begin() + 3, v.end());

    cout << "前3个最小值（已排序）: ";
    for (int i = 0; i < 3; i++) cout << v[i] << " ";
    cout << "\n";
    // 输出：1 2 3

    cout << "剩余元素（顺序不保证）: ";
    for (int i = 3; i < (int)v.size(); i++) cout << v[i] << " ";
    cout << "\n";
    // 可能输出：8 9 5 4 7 6（不确定）

    return 0;
}
```

---

### 3. 找前 N 大的元素

配合 `greater<>` 就可以找前 N 大的：

```cpp
#include <algorithm>
#include <vector>
#include <functional>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {5, 3, 8, 1, 9, 2, 7, 4, 6};

    // 找前 3 大，降序排列
    partial_sort(v.begin(), v.begin() + 3, v.end(), greater<int>());

    cout << "前3个最大值: ";
    for (int i = 0; i < 3; i++) cout << v[i] << " ";
    // 输出：9 8 7
    return 0;
}
```

---

### 4. `partial_sort_copy` — 结果写到另一个容器

```cpp
#include <algorithm>
#include <iterator>   // distance
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> src = {5, 3, 8, 1, 9, 2, 7, 4, 6};
    vector<int> top3(3);  // 存放结果

    // 原容器不被修改，结果写入 top3
    // 它会返回一个迭代器，指向目标范围内最后写入元素的下一个位置
    auto it = partial_sort_copy(src.begin(), src.end(),
                                top3.begin(), top3.end());
    cout << "实际写入了 " << distance(top3.begin(), it) << " 个元素\n";

    cout << "原容器不变: "; for (int x : src)  cout << x << " "; cout << "\n";
    cout << "前3小: ";      for (int x : top3) cout << x << " "; cout << "\n";
    // 原容器不变: 5 3 8 1 9 2 7 4 6
    // 前3小: 1 2 3
    return 0;
}
```

当目标区间更短时，`partial_sort_copy` 只会写满目标区间；当目标区间更长时，返回值可以告诉你**实际写入了多少个元素**。

---

### 5. 时间复杂度

| 函数           | 复杂度                                 |
| -------------- | -------------------------------------- |
| `sort`         | O(N log N)                             |
| `partial_sort` | **O(N log M)**，M 是需要排好的元素数量 |

当 M 远小于 N 时，`partial_sort` 比全排序快得多！例如 N=1000000，M=10，节省极大。

---

## `[补]` `std::nth_element` — 快速选择第 N 小

### 1. 核心思想

`nth_element` 解决的是**"第 N 小的元素是什么"**这个问题，无需完全排序。

排完后的保证：
- `v[nth]` 的值就是全局排好序后第 `nth` 位的元素
- `v[nth]` 左边的所有元素 ≤ `v[nth]`
- `v[nth]` 右边的所有元素 ≥ `v[nth]`
- 左边和右边各自**内部顺序不确定**

```
[nth] 左边：都 ≤ v[nth]    [nth]    右边：都 ≥ v[nth]
  乱序                   ← 这个值确定 →   乱序
```

---

### 2. 基础示例：找中位数

```cpp
#include <algorithm>
#include <iterator>   // distance
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {5, 3, 8, 1, 9, 2, 7, 4, 6};
    int n = v.size();

    // 找中位数（第 n/2 小，0-indexed）
    auto mid = v.begin() + n / 2;
    nth_element(v.begin(), mid, v.end());

    cout << "中位数是: " << *mid << "\n";
    // 输出：5（9个元素中第5小的）

    return 0;
}
```

---

### 3. 找第 K 小的元素

```cpp
#include <algorithm>
#include <functional>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {5, 3, 8, 1, 9, 2, 7, 4, 6};

    int k = 3;  // 找第 3 小（0-indexed 是第 2 位）

    nth_element(v.begin(), v.begin() + k - 1, v.end());

    cout << "第" << k << "小的元素: " << v[k - 1] << "\n";
    // 输出：3

    return 0;
}
```

---

### 4. `nth_element` vs `partial_sort` vs `sort`

| 函数                                 | 时间          | 结果保证                    |
| ------------------------------------ | ------------- | --------------------------- |
| `sort`                               | O(N log N)    | 全部有序                    |
| `partial_sort(first, first+M, last)` | O(N log M)    | 前 M 个有序，后面无序       |
| `nth_element(first, nth, last)`      | **O(N)** | 只有 nth 位置正确，两侧无序 |

`nth_element` 基于**快速选择算法（QuickSelect）及其改良版（Introselect）**。在 C++11 中平均为 O(N)（最坏 O(N²)），但自 **C++20 标准起，强制要求其最坏复杂度也为 O(N)**。它是寻找第 K 大/小值的最快选择。

> ⚠️ **边界提醒**：`nth` 必须落在 `[first, last)` 区间内。比如想找前 `k` 小中的"分界点"，应传 `v.begin() + k - 1` 或先确认你要的是第几个元素，而不是随手写成 `v.begin() + k`。

---

### 5. 综合实战：找前 K 小的所有元素（不需排序）

```cpp
#include <algorithm>
#include <functional>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {5, 3, 8, 1, 9, 2, 7, 4, 6};

    int k = 4;
    nth_element(v.begin(), v.begin() + k, v.end());

    // 前 k 个就是最小的 k 个，但内部顺序不确定
    cout << "最小的" << k << "个元素（顺序不定）: ";
    for (int i = 0; i < k; i++) cout << v[i] << " ";
    cout << "\n";
    // 可能输出：3 1 2 4（这4个是最小的，但顺序随机）

    return 0;
}
```

---

## `[补]` 排列算法

---

## `[补]` `std::next_permutation` — 生成下一个字典序排列

### 1. 什么是字典序？

字典序就是"词典中单词的排列顺序"，逐位比较：

```
"abc" < "abd"  （第3位 c < d）
"abc" < "ac"   （第2位 b < c）
"ba"  < "bb"   （第2位 a < b）
```

对于序列 `[1, 2, 3]`，所有排列按字典序为：

```
1 2 3  →  第1个（最小字典序）
1 3 2  →  第2个
2 1 3  →  第3个
2 3 1  →  第4个
3 1 2  →  第5个
3 2 1  →  第6个（最大字典序）
```

`next_permutation` 的作用：给定当前排列，**原地修改**为下一个字典序排列。

---

### 2. 基础用法

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3};

    cout << "第1个排列: ";
    for (int x : v) cout << x << " ";
    cout << "\n";

    next_permutation(v.begin(), v.end());
    cout << "第2个排列: ";
    for (int x : v) cout << x << " ";
    cout << "\n";

    next_permutation(v.begin(), v.end());
    cout << "第3个排列: ";
    for (int x : v) cout << x << " ";
    cout << "\n";

    // 输出：
    // 第1个排列: 1 2 3
    // 第2个排列: 1 3 2
    // 第3个排列: 2 1 3
    return 0;
}
```

**返回值**：
- 若成功找到下一个排列，返回 `true`
- 若当前已是最大字典序（如 `3 2 1`），将其重置为最小字典序（`1 2 3`）并返回 `false`

---

### 3. 枚举所有排列（最常用模式）

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4};

    // 关键：必须先排序！从最小字典序开始
    sort(v.begin(), v.end());

    int count = 0;
    do {
        count++;
        for (int x : v) cout << x << " ";
        cout << "\n";
    } while (next_permutation(v.begin(), v.end()));
    // do-while 会在 next_permutation 返回 false（回到最小排列）时停止

    cout << "共 " << count << " 个排列\n";  // 共 24 个排列
    return 0;
}
```

> **⚠️ 必须先 `sort` 确保从最小字典序开始，否则只会生成从当前排列开始到最大排列的子集！**

---

### 4. 对字符串使用

`next_permutation` 对 `string` 同样适用：

```cpp
#include <algorithm>
#include <string>
#include <iostream>
using namespace std;

int main() {
    string s = "abc";
    sort(s.begin(), s.end());

    do {
        cout << s << "\n";
    } while (next_permutation(s.begin(), s.end()));
    // 输出：abc, acb, bac, bca, cab, cba
    return 0;
}
```

---

### 5. 含重复元素的排列

`next_permutation` 自动处理重复元素，**只生成不重复的排列**：

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 1, 2};
    sort(v.begin(), v.end());

    do {
        for (int x : v) cout << x << " ";
        cout << "\n";
    } while (next_permutation(v.begin(), v.end()));

    // 输出：
    // 1 1 2
    // 1 2 1
    // 2 1 1
    // 共 3 个（而非 3! = 6 个，自动去重）
    return 0;
}
```

---

### 6. `next_permutation` 的内部原理

理解原理有助于你灵活使用。算法步骤（以 `2 4 3 1` 为例）：

```
序列：2  4  3  1

第一步：从右往左找第一个"下降位置" i，即 v[i] < v[i+1]
        2  [4]  3  1  ← 4 > 3，不是；[2] < 4，找到 i=0，v[i]=2

第二步：从右往左找第一个大于 v[i] 的位置 j
        2  4  [3]  1  ← 3 > 2，找到 j=2，v[j]=3

第三步：交换 v[i] 和 v[j]
        3  4  2  1  ← 交换后

第四步：将 i+1 到末尾的部分翻转（使其变成最小）
        3  [1  2  4]  ← 翻转后

结果：3  1  2  4（这就是 2 4 3 1 的下一个排列）
```

---

### 7. 时间复杂度

每次调用 `next_permutation`：O(N)（最坏情况翻转整个序列）

枚举所有 N! 个排列：O(N × N!)

---

## `[补]` `std::prev_permutation` — 生成上一个字典序排列

### 1. 基础用法

`prev_permutation` 与 `next_permutation` 完全对称：给定当前排列，修改为**上一个字典序**排列。

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {3, 2, 1};  // 最大字典序

    cout << "当前: ";
    for (int x : v) cout << x << " ";
    cout << "\n";

    prev_permutation(v.begin(), v.end());
    cout << "上一个: ";
    for (int x : v) cout << x << " ";
    cout << "\n";

    // 当前: 3 2 1
    // 上一个: 3 1 2
    return 0;
}
```

**返回值**：
- 若成功找到上一个排列，返回 `true`
- 若当前已是最小字典序（如 `1 2 3`），将其重置为最大字典序（`3 2 1`）并返回 `false`

---

### 2. 从最大字典序往前枚举

```cpp
#include <algorithm>
#include <functional>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3};

    // 先变成最大字典序作为起始点
    sort(v.begin(), v.end(), greater<int>());  // v = {3, 2, 1}

    int count = 0;
    do {
        count++;
        for (int x : v) cout << x << " ";
        cout << "\n";
    } while (prev_permutation(v.begin(), v.end()));

    cout << "共 " << count << " 个排列\n";
    return 0;
}
```

---

### 3. `next_permutation` vs `prev_permutation` 对比总结

| 项目       | `next_permutation`                     | `prev_permutation`                     |
| ---------- | -------------------------------------- | -------------------------------------- |
| 方向       | 当前 → 字典序更大的下一个              | 当前 → 字典序更小的上一个              |
| 初始状态   | 从最小字典序（升序）开始               | 从最大字典序（降序）开始               |
| 到达边界时 | 重置为最小字典序，返回 `false`         | 重置为最大字典序，返回 `false`         |
| 结合循环   | `do { } while(next_permutation(...));` | `do { } while(prev_permutation(...));` |

---

## 综合总结

### 函数速查表

| 函数               | 头文件        | 要求           | 时间复杂度  | 核心用途             |
| ------------------ | ------------- | -------------- | ----------- | -------------------- |
| `sort`             | `<algorithm>` | 随机访问迭代器 | O(N log N)  | 通用排序             |
| `stable_sort`      | `<algorithm>` | 随机访问迭代器 | O(N log N) / O(N log² N) | 保持相等元素相对顺序 |
| `partial_sort`     | `<algorithm>` | 随机访问迭代器 | O(N log M)  | 只排前 M 小/大       |
| `nth_element`      | `<algorithm>` | 随机访问迭代器 | O(N) 平均   | 找第 N 小，快速选择  |
| `shuffle`          | `<algorithm>` | 随机访问迭代器 | O(N)        | 随机打乱             |
| `merge`            | `<algorithm>` | 输入迭代器     | O(N+M)      | 合并两个有序序列     |
| `inplace_merge`    | `<algorithm>` | 双向迭代器     | O(N) / O(N log N) | 原地合并相邻有序段   |
| `reverse`          | `<algorithm>` | 双向迭代器     | O(N)        | 原地翻转             |
| `reverse_copy`     | `<algorithm>` | 双向迭代器     | O(N)        | 翻转并写入新位置     |
| `next_permutation` | `<algorithm>` | 双向迭代器     | O(N)        | 生成下一字典序排列   |
| `prev_permutation` | `<algorithm>` | 双向迭代器     | O(N)        | 生成上一字典序排列   |

---

### 选择哪个排序函数？

```
你的需求是什么？
│
├─ 全部排好序？
│   ├─ 需要保证相等元素顺序不变？  → stable_sort
│   └─ 不需要保证                  → sort（更快）
│
├─ 只需要前 K 个排好序？           → partial_sort
│
├─ 只需要知道第 K 小是什么？       → nth_element（最快）
│
├─ 合并两个已有序的序列？          → merge / inplace_merge
│
├─ 翻转序列？                      → reverse
│
├─ 随机打乱？                      → shuffle（配合 mt19937）
│
└─ 枚举所有排列？
    ├─ 从小到大方向？              → 先 sort，再 next_permutation 循环
    └─ 从大到小方向？              → 先降序 sort，再 prev_permutation 循环
```

---

### 一个综合示例：竞赛题常用组合

```cpp
#include <algorithm>
#include <vector>
#include <random>
#include <numeric>   // iota
#include <iostream>
using namespace std;

int main() {
    // ① 生成随机测试数据
    vector<int> v(10);
    iota(v.begin(), v.end(), 1);  // {1,2,...,10}
    mt19937 rng(42);
    shuffle(v.begin(), v.end(), rng);  // 打乱

    cout << "打乱后: ";
    for (int x : v) cout << x << " ";
    cout << "\n";

    // ② 找中位数（不用完整排序）
    auto midIt = v.begin() + v.size() / 2;
    nth_element(v.begin(), midIt, v.end());
    cout << "中位数: " << *midIt << "\n";

    // ③ 找最小的 3 个（不用完整排序）
    partial_sort(v.begin(), v.begin() + 3, v.end());
    cout << "最小3个: " << v[0] << " " << v[1] << " " << v[2] << "\n";

    // ④ 全部排好序，然后枚举所有排列（仅演示用小数组）
    vector<int> small = {1, 2, 3};
    sort(small.begin(), small.end());
    cout << "所有排列:\n";
    do {
        for (int x : small) cout << x << " ";
        cout << "\n";
    } while (next_permutation(small.begin(), small.end()));

    return 0;
}
```

---

> **学习建议**：
> 1. 先把 `sort` + Lambda 写熟，这是日常最常用的组合
> 2. 记住 `nth_element` 是 O(N) 的，竞赛中找第 K 小优先用它
> 3. `next_permutation` 枚举排列是算法题中的固定套路，务必熟练
> 4. `stable_sort` 用的场景比较专一，理解"稳定性"概念比记函数名更重要
