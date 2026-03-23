# C++ STL 入门教程

---

## 一、STL 的基本概念

### 1.1 什么是 STL？

**STL（Standard Template Library，标准模板库）** 是 C++ 标准库中最重要的组成部分之一。它提供了一套 **通用的、基于模板的** 数据结构和算法，让程序员不必重复造轮子。

> 💡 STL 的核心思想：**将数据结构和算法分离**，通过迭代器将两者连接起来。

### 1.2 STL 的六大组件

STL 由以下六大组件构成，它们相互协作，构成了一个完整的体系：

| 组件       | 英文名    | 作用                                    | 示例                    |
| ---------- | --------- | --------------------------------------- | ----------------------- |
| 容器       | Container | 存放数据的数据结构                      | vector、list、map       |
| 算法       | Algorithm | 操作数据的函数                          | sort()、find()、count() |
| 迭代器     | Iterator  | 连接容器与算法的桥梁                    | vector<int>::iterator   |
| 仿函数     | Functor   | 行为类似函数的对象（重载了 operator()） | greater<int>            |
| 适配器     | Adapter   | 修饰容器、仿函数、迭代器的接口          | stack、queue            |
| 空间配置器 | Allocator | 负责内存的分配与释放                    | allocator<T>            |

#### 六大组件的关系图

```
算法(Algorithm)
       ↑
       | 通过迭代器访问数据
       |
  迭代器(Iterator) ←——→ 容器(Container) ←—— 空间配置器(Allocator)
       |                      ↑
       |                      | 适配器(Adapter) 包装
       ↓                      |
  仿函数(Functor) ——→ 为算法提供策略
```

### 1.3 容器的分类

STL 中的容器分为三大类：

#### ① 序列容器（Sequence Containers）

元素按照 **插入顺序** 排列，支持顺序访问。

| 容器         | 底层结构              | 特点                                |
| ------------ | --------------------- | ----------------------------------- |
| vector       | 动态数组              | 尾部插入/删除快，随机访问快         |
| deque        | 双端队列              | 头尾插入/删除快                     |
| list         | 双向链表              | 任意位置插入/删除快，不支持随机访问 |
| forward_list | 单向链表（C++11）     | 比 list 更省内存                    |
| array        | 固定大小数组（C++11） | 编译期确定大小，不可动态扩容        |

#### ② 关联容器（Associative Containers）

元素按照 **键值** 自动排序（底层通常是红黑树）。

| 容器     | 特点             |
| -------- | ---------------- |
| set      | 不允许重复元素   |
| multiset | 允许重复元素     |
| map      | 键值对，键不重复 |
| multimap | 键值对，键可重复 |

#### ③ 无序关联容器（Unordered Associative Containers，C++11）

底层为 **哈希表**，查找效率接近 O(1)。

| 容器               | 特点                   |
| ------------------ | ---------------------- |
| unordered_set      | 不允许重复，无序       |
| unordered_multiset | 允许重复，无序         |
| unordered_map      | 键值对，键不重复，无序 |
| unordered_multimap | 键值对，键可重复，无序 |

### 1.4 迭代器的分类

迭代器是 STL 的灵魂，它让算法不需要知道容器的内部结构就能遍历元素。

| 迭代器类型     | 功能               | 支持的容器           |
| -------------- | ------------------ | -------------------- |
| 输入迭代器     | 只读，单向移动     | istream_iterator     |
| 输出迭代器     | 只写，单向移动     | ostream_iterator     |
| 前向迭代器     | 读写，单向移动     | forward_list         |
| 双向迭代器     | 读写，双向移动     | list、set、map       |
| 随机访问迭代器 | 读写，可跳跃式访问 | vector、deque、array |

> 💡 **记忆方法**：功能从上到下递增。随机访问迭代器功能最强，能做其他迭代器能做的一切。

**常用的迭代器操作：**

```cpp
it++       // 向后移动一个位置
it--       // 向前移动一个位置（双向及以上）
*it        // 解引用，获取当前指向的元素
it + n     // 向后跳跃 n 个位置（仅随机访问迭代器）
it1 == it2 // 判断两个迭代器是否指向同一位置
```

### 1.5 STL 原理补充：复杂度与迭代器失效（易错点）

很多 STL 代码“看起来能跑”，但性能和稳定性问题往往出在这两件事上：

- **复杂度意识**：`vector` 随机访问是 O(1)，但中间插入/删除通常是 O(n)。
- **迭代器失效**：容器结构变化后，旧迭代器/引用/指针可能失效。

高频规则（先记住这 4 条）：

- `vector` 扩容后，原有迭代器、引用、指针通常全部失效。
- `vector::erase` 会让被删位置及其后的迭代器失效。
- `reserve()` 改变的是 **capacity**，不改变 **size**；`resize()` 会改变 **size**。
- 迭代器尽量用 `auto`，只读遍历优先 `const auto&`，减少类型噪音与拷贝。

> 💡 编译建议：本文示例默认按 `-std=c++17`；涉及 C++20 的代码会单独标注。

---

## 二、vector 存放内置数据类型

### 2.1 vector 简介

`vector` 是 STL 中最常用的容器，可以理解为一个 **可以动态扩容的数组**。

**核心特性：**

- 内存连续，支持 **随机访问**（通过下标 `[]` 或 `.at()`）

- **尾部** 插入/删除效率高（`push_back` / `pop_back`）

- **中间/头部** 插入/删除效率低（需要移动大量元素）

- 自动管理内存，当容量不够时会 **自动扩容**（通常是当前容量的 1.5 倍或 2 倍）

**需要包含的头文件：**

```cpp
#include <vector>
```

### 2.2 基本用法：创建与添加元素

```cpp
#include <vector>
#include <iostream>

int main() {
    // ========== 1. 创建 vector（从基础到进阶） ==========
    std::vector<int> v1;                 // 空 vector
    std::vector<int> v2(5);              // 5 个元素，值初始化为 0
    std::vector<int> v3(5, 42);          // 5 个 42
    std::vector<int> v4{10, 20, 30};     // 列表初始化（推荐）
    std::vector<int> v5(v4);             // 拷贝构造
    std::vector<int> v6(v4.begin(), v4.end()); // 区间构造
    std::vector v7{1, 2, 3};             // C++17: 类模板实参推导（CTAD）

    // ========== 2. 添加元素 ==========
    v1.reserve(8); // 预留容量，减少扩容次数
    for (int x : {10, 20, 30, 40}) {
        v1.push_back(x);
    }

    // emplace_back 对内置类型收益不大，对自定义类型更明显
    v1.emplace_back(50);

    std::cout << "v1 size = " << v1.size() << '\n';
    return 0;
}
```

> 💡 **`push_back` vs `emplace_back`**：
>
> 
>
> - `push_back(val)` 先构造一个临时对象，再拷贝/移动到容器中
>
> - `emplace_back(args...)` 直接在容器末尾 **原地构造**，效率更高
>
> - 对于内置类型（如 `int`）差别不大，但对于自定义类型差别明显

### 2.3 遍历 vector 的五种方式

```cpp
#include <iostream>
#include <vector>
#include <algorithm> // std::for_each
#include <ranges>    // std::ranges::for_each (C++20)

int main() {
    std::vector<int> v{10, 20, 30, 40, 50};

    // ========== 方式一：下标（先理解顺序访问） ==========
    std::cout << "方式一（下标）：";
    for (std::size_t i = 0; i < v.size(); ++i) {
        std::cout << v[i] << ' ';
    }
    std::cout << '\n';

    // ========== 方式二：auto 迭代器 ==========
    std::cout << "方式二（auto 迭代器）：";
    for (auto it = v.begin(); it != v.end(); ++it) {
        std::cout << *it << ' ';
    }
    std::cout << '\n';

    // ========== 方式三：范围 for（最推荐） ==========
    std::cout << "方式三（范围 for）：";
    for (const int val : v) {
        std::cout << val << ' ';
    }
    std::cout << '\n';

    // 如果需要修改元素，用引用：
    // for (int& val : v) { val *= 2; }

    // ========== 方式四：std::for_each ==========
    std::cout << "方式四（std::for_each）：";
    std::for_each(v.begin(), v.end(), [](int val) {
        std::cout << val << ' ';
    });
    std::cout << '\n';

    // ========== 方式五：std::ranges::for_each（C++20） ==========
    std::cout << "方式五（ranges::for_each）：";
    std::ranges::for_each(v, [](int val) {
        std::cout << val << ' ';
    });
    std::cout << '\n';

    return 0;
}
```

**输出：**

```
方式一（下标）：10 20 30 40 50
方式二（auto 迭代器）：10 20 30 40 50
方式三（范围 for）：10 20 30 40 50
方式四（std::for_each）：10 20 30 40 50
方式五（ranges::for_each）：10 20 30 40 50
```

### 2.4 迭代器详解：begin(), end(), rbegin(), rend()

```
          begin()                    end()
            ↓                         ↓
          +----+----+----+----+----+
          | 10 | 20 | 30 | 40 | 50 | (不存在的位置)
          +----+----+----+----+----+
       ↑                         ↑                   
     rend()                    rbegin()            
（rend 指向第一个元素之前）   （rbegin 指向最后一个元素）
```

- `begin()` → 指向 **第一个元素**

- `end()` → 指向 **最后一个元素的下一个位置**（一个"哨兵"位置）

- `rbegin()` → 反向迭代器，指向 **最后一个元素**

- `rend()` → 反向迭代器，指向 **第一个元素之前的位置**

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // 正向遍历
    cout << "正向：";
    for (auto it = v.begin(); it != v.end(); ++it) {
        cout << *it << " ";
    }
    cout << endl;

    // 反向遍历
    cout << "反向：";
    for (auto rit = v.rbegin(); rit != v.rend(); ++rit) {
        cout << *rit << " ";
    }
    cout << endl;

    // const 迭代器（只读，不能修改元素）
    cout << "只读：";
    for (auto cit = v.cbegin(); cit != v.cend(); ++cit) {
        // *cit = 100;  // 编译错误！const 迭代器不允许修改
        cout << *cit << " ";
    }
    cout << endl;

    return 0;
}
```

**输出：**

```
正向：10 20 30 40 50
反向：50 40 30 20 10
只读：10 20 30 40 50
```

### 2.5 常用成员函数一览

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // ========== 容量相关 ==========
    cout << "size     = " << v.size()     << endl;  // 元素个数：5
    cout << "capacity = " << v.capacity() << endl;  // 当前容量（≥ size）
    cout << "empty    = " << v.empty()    << endl;  // 是否为空：0 (false)

    v.reserve(100);  // 预留容量为 100（避免频繁扩容）
    cout << "reserve 后 capacity = " << v.capacity() << endl; // 100

    v.shrink_to_fit(); // 释放多余内存，使 capacity 尽量等于 size
    cout << "shrink 后 capacity  = " << v.capacity() << endl;

    // ========== 元素访问 ==========
    cout << "v[0]     = " << v[0]        << endl;  // 下标访问（不检查越界）
    cout << "v.at(0)  = " << v.at(0)     << endl;  // at 访问（越界抛异常）
    cout << "v.front()= " << v.front()   << endl;  // 第一个元素
    cout << "v.back() = " << v.back()    << endl;  // 最后一个元素
    cout << "v.data() = " << v.data()    << endl;  // 底层数组指针

    // ========== 修改相关 ==========
    v.push_back(60);                         // 尾部添加
    v.pop_back();                            // 尾部删除
    v.insert(v.begin() + 1, 15);             // 在第 2 个位置插入 15
    v.erase(v.begin() + 1);                  // 删除第 2 个位置的元素
    v.erase(v.begin(), v.begin() + 2);       // 删除 [begin, begin+2) 范围
    v.clear();                               // 清空所有元素（size 变 0，capacity 不变）

    // ========== 赋值与交换 ==========
    vector<int> a = {1, 2, 3};
    vector<int> b = {4, 5, 6};

    a.swap(b);  // 交换 a 和 b 的内容

    // 实用技巧：利用 swap 收缩内存
    vector<int> large(10000, 1);
    large.resize(3);  // size = 3，但 capacity 仍然是 10000
    cout << "收缩前 capacity = " << large.capacity() << endl;

    vector<int>(large).swap(large);  // 用匿名对象交换，释放多余内存
    cout << "收缩后 capacity = " << large.capacity() << endl;

    return 0;
}
```

> ⚠️ **注意 `[]` 和 `.at()` 的区别**：
>
> 
>
> - `v[i]`：不检查越界，越界时行为未定义（可能崩溃，也可能得到垃圾值）
>
> - `v.at(i)`：检查越界，越界时抛出 `std::out_of_range` 异常

#### 2.5.1 高频易错点（建议先掌握）

- `reserve(n)` 只扩容量，不会产生 `n` 个元素；访问 `v[n - 1]` 依然可能越界。
- `resize(n)` 会真正改变元素个数，新增元素会被值初始化。
- 在 `for` 循环里 `erase` 元素时，不要直接 `++it`，要使用 `it = v.erase(it)`。
- 若后续还要继续插入，先 `reserve` 可以显著减少“扩容导致迭代器失效”问题。

---

## 三、vector 存放自定义数据类型

### 3.1 存放自定义类对象

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;

    Person(const string& name, int age) : name(name), age(age) {}

    // 为了方便输出，可以重载 << 运算符
    friend ostream& operator<<(ostream& os, const Person& p) {
        os << "{" << p.name << ", " << p.age << "}";
        return os;
    }
};

int main() {
    // ========== 1. 存放对象本身 ==========
    vector<Person> v;

    // push_back 方式（先构造临时对象，再拷贝/移动进去）
    v.push_back(Person("张三", 25));
    v.push_back(Person("李四", 30));
    v.push_back(Person("王五", 28));

    // emplace_back 方式（直接在容器内原地构造，更高效）
    v.emplace_back("赵六", 35);
    v.emplace_back("钱七", 22);

    // 遍历
    cout << "=== 存放对象 ===" << endl;
    for (const auto& p : v) {
        cout << p << endl;
    }

    return 0;
}
```

**输出：**

```
=== 存放对象 ===
{张三, 25}
{李四, 30}
{王五, 28}
{赵六, 35}
{钱七, 22}
```

### 3.2 存放对象指针

当对象很大、需要多态或希望稳定地址时，存放指针更合适。现代 C++ 中优先使用智能指针。

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <memory>  // 智能指针

class Person {
public:
    std::string name;
    int age;

    Person(const std::string& name, int age) : name(name), age(age) {}

    friend std::ostream& operator<<(std::ostream& os, const Person& p) {
        os << "{" << p.name << ", " << p.age << "}";
        return os;
    }
};

int main() {
    // ========== 推荐：unique_ptr（唯一所有权） ==========
    std::cout << "=== unique_ptr ===\n";
    std::vector<std::unique_ptr<Person>> v1;
    v1.push_back(std::make_unique<Person>("赵六", 35));
    v1.push_back(std::make_unique<Person>("钱七", 22));
    for (const auto& p : v1) {
        std::cout << *p << '\n';
    }

    // ========== 需要共享所有权时：shared_ptr ==========
    std::cout << "\n=== shared_ptr ===\n";
    std::vector<std::shared_ptr<Person>> v2;
    auto p = std::make_shared<Person>("孙八", 40);
    v2.push_back(p);
    v2.push_back(p); // 两个位置共享同一对象
    std::cout << "use_count = " << p.use_count() << '\n';

    return 0;
}
```

**输出：**

```
=== unique_ptr ===
{赵六, 35}
{钱七, 22}

=== shared_ptr ===
use_count = 3
```

> ⚠️ 除非是与旧接口兼容，否则尽量避免在示例中直接写 `new`/`delete`。

> 💡 **存对象 vs 存指针，如何选择？**
>
>
> | 场景                           | 推荐方式                               |
> | ------------------------------ | -------------------------------------- |
> | 对象较小，拷贝开销低           | 直接存对象                             |
> | 对象较大，拷贝开销高           | 存智能指针（unique_ptr 或 shared_ptr） |
> | 需要多态（基类指针指向派生类） | 必须存指针                             |
> | 不想操心内存管理               | 直接存对象 或 智能指针                 |

### 3.3 自定义类型与 STL 算法配合

当你需要对自定义类型使用 `sort`、`find` 等算法时，需要提供 **比较规则**：

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

class Person {
public:
    string name;
    int age;

    Person(const string& name, int age) : name(name), age(age) {}

    // 重载 == 运算符（find 需要）
    bool operator==(const Person& other) const {
        return name == other.name && age == other.age;
    }

    // 重载 < 运算符（sort 默认使用）
    bool operator<(const Person& other) const {
        return age < other.age; // 按年龄升序
    }

    friend ostream& operator<<(ostream& os, const Person& p) {
        os << "{" << p.name << ", " << p.age << "}";
        return os;
    }
};

int main() {
    vector<Person> v = {
        {"张三", 25},
        {"李四", 30},
        {"王五", 28},
        {"赵六", 22},
        {"钱七", 35}
    };

    // ========== 排序（使用 operator<） ==========
    sort(v.begin(), v.end());
    cout << "按年龄升序：" << endl;
    for (const auto& p : v) {
        cout << "  " << p << endl;
    }

    // ========== 自定义排序规则（使用 lambda） ==========
    sort(v.begin(), v.end(), [](const Person& a, const Person& b) {
        return a.age > b.age;  // 按年龄降序
    });
    cout << "\n按年龄降序：" << endl;
    for (const auto& p : v) {
        cout << "  " << p << endl;
    }

    // ========== 查找（使用 operator==） ==========
    auto it = find(v.begin(), v.end(), Person("王五", 28));
    if (it != v.end()) {
        cout << "\n找到了：" << *it << endl;
    }

    return 0;
}
```

**输出：**

```
按年龄升序：
  {赵六, 22}
  {张三, 25}
  {王五, 28}
  {李四, 30}
  {钱七, 35}

按年龄降序：
  {钱七, 35}
  {李四, 30}
  {王五, 28}
  {张三, 25}
  {赵六, 22}

找到了：{王五, 28}
```

---

## 四、容器嵌套容器

### 4.1 什么是容器嵌套？

容器嵌套就是 **容器中套容器**，类似于二维数组的概念。最常见的是 `vector<vector<int>>`——一个 vector，其中每个元素又是一个 vector。

```
外层 vector:
+--------------------------------------------+
| vector<int> | vector<int> | vector<int> | ...
+--------------------------------------------+
      ↓              ↓             ↓
  {1, 2, 3}     {4, 5, 6}    {7, 8, 9}
```

### 4.2 基本用法

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // ========== 创建二维 vector ==========
    vector<vector<int>> matrix;

    // 创建 3 个内层 vector
    vector<int> row1 = {1, 2, 3};
    vector<int> row2 = {4, 5, 6};
    vector<int> row3 = {7, 8, 9};

    // 添加到外层 vector
    matrix.push_back(row1);
    matrix.push_back(row2);
    matrix.push_back(row3);

    // ========== 遍历方式一：范围 for（推荐） ==========
    cout << "=== 范围 for ===" << endl;
    for (const auto& row : matrix) {
        for (int val : row) {
            cout << val << " ";
        }
        cout << endl;
    }

    // ========== 遍历方式二：迭代器 ==========
    cout << "\n=== 迭代器 ===" << endl;
    for (auto it = matrix.begin(); it != matrix.end(); ++it) {
        // *it 的类型是 vector<int>&
        for (auto jt = it->begin(); jt != it->end(); ++jt) {
            cout << *jt << " ";
        }
        cout << endl;
    }

    // ========== 遍历方式三：下标 ==========
    cout << "\n=== 下标 ===" << endl;
    for (size_t i = 0; i < matrix.size(); ++i) {
        for (size_t j = 0; j < matrix[i].size(); ++j) {
            cout << matrix[i][j] << " ";
        }
        cout << endl;
    }

    return 0;
}
```

**输出：**

```
=== 范围 for ===
1 2 3
4 5 6
7 8 9

=== 迭代器 ===
1 2 3
4 5 6
7 8 9

=== 下标 ===
1 2 3
4 5 6
7 8 9
```

### 4.3 直接初始化二维 vector

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // ========== 方式一：初始化列表（C++11） ==========
    vector<vector<int>> m1 = {
        {1,  2,  3,  4},
        {5,  6,  7,  8},
        {9, 10, 11, 12}
    };

    // ========== 方式二：创建 M × N 的全零矩阵 ==========
    int rows = 3, cols = 4;
    vector<vector<int>> m2(rows, vector<int>(cols, 0));
    //                     ↑行数       ↑列数   ↑初始值

    // 修改某个元素
    m2[1][2] = 42;

    cout << "m1:" << endl;
    for (const auto& row : m1) {
        for (int val : row) {
            cout << val << "\t";
        }
        cout << endl;
    }

    cout << "\nm2:" << endl;
    for (const auto& row : m2) {
        for (int val : row) {
            cout << val << "\t";
        }
        cout << endl;
    }

    return 0;
}
```

**输出：**

```
m1:
1	2	3	4
5	6	7	8
9	10	11	12

m2:
0	0	0	0
0	0	42	0
0	0	0	0
```

### 4.4 嵌套自定义类型

容器嵌套不仅限于内置类型，也可以嵌套自定义类型的容器：

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

class Student {
public:
    string name;
    int score;

    Student(const string& name, int score) : name(name), score(score) {}

    friend ostream& operator<<(ostream& os, const Student& s) {
        os << s.name << "(" << s.score << "分)";
        return os;
    }
};

int main() {
    // 模拟：3 个班级，每个班级有若干学生
    vector<vector<Student>> school;

    // 一班
    school.push_back({
        {"张三", 90},
        {"李四", 85},
        {"王五", 92}
    });

    // 二班
    school.push_back({
        {"赵六", 88},
        {"钱七", 76}
    });

    // 三班
    school.push_back({
        {"孙八", 95},
        {"周九", 82},
        {"吴十", 79},
        {"郑一", 91}
    });

    // 遍历所有班级和学生
    for (size_t i = 0; i < school.size(); ++i) {
        cout << "=== 第 " << (i + 1) << " 班 ===" << endl;
        for (const auto& student : school[i]) {
            cout << "  " << student << endl;
        }
    }

    return 0;
}
```

**输出：**

```
=== 第 1 班 ===
  张三(90分)
  李四(85分)
  王五(92分)
=== 第 2 班 ===
  赵六(88分)
  钱七(76分)
=== 第 3 班 ===
  孙八(95分)
  周九(82分)
  吴十(79分)
  郑一(91分)
```

### 4.5 更复杂的嵌套：三层嵌套

虽然实际开发中很少用到，但了解一下三层嵌套是如何工作的：

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // vector<vector<vector<int>>> → 三维数组
    vector<vector<vector<int>>> cube = {
        {   // 第一层
            {1, 2, 3},
            {4, 5, 6}
        },
        {   // 第二层
            {7, 8, 9},
            {10, 11, 12}
        }
    };

    for (size_t i = 0; i < cube.size(); ++i) {
        cout << "=== 层 " << i << " ===" << endl;
        for (size_t j = 0; j < cube[i].size(); ++j) {
            cout << "  行 " << j << ": ";
            for (size_t k = 0; k < cube[i][j].size(); ++k) {
                cout << cube[i][j][k] << " ";
            }
            cout << endl;
        }
    }

    return 0;
}
```

**输出：**

```
=== 层 0 ===
  行 0: 1 2 3
  行 1: 4 5 6
=== 层 1 ===
  行 0: 7 8 9
  行 1: 10 11 12
```

> ⚠️ **嵌套层数过多时代码可读性会急剧下降**。超过两层嵌套时，建议用结构体或类来封装，让代码更清晰。

---

## 五、总结

| 知识点            | 核心要点                                                     |
| ----------------- | ------------------------------------------------------------ |
| STL 概念          | 六大组件：容器、算法、迭代器、仿函数、适配器、空间配置器     |
| vector 内置类型   | push_back / emplace_back 添加、5 种遍历方式、size / capacity / reserve |
| vector 自定义类型 | 存对象 vs 存指针、emplace_back 更高效、需要重载运算符配合算法 |
| 容器嵌套          | vector<vector<T>> 作为二维结构、双重循环遍历、支持多种初始化方式 |

### 下一步学习建议

1. **vector 的其他操作**：`resize`、`assign`、`insert` 的各种重载

2. **其他序列容器**：`deque`、`list` 的用法和与 `vector` 的对比

3. **关联容器**：`map`、`set` 的用法

4. **STL 算法**：`sort`、`find`、`count`、`accumulate`、`transform` 等

5. **仿函数**：自定义比较规则、`greater<>`、`less<>` 等
