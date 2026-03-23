# 常用查找算法

> 头文件：`<algorithm>`
> 所有查找算法都定义在 `std::` 命名空间中。

---

## 79. 常用查找算法 — `find`

### 79.1 功能说明

`find` 用于在指定范围内查找**第一个等于给定值**的元素。

它是最基础、最常用的线性查找算法之一。只要容器中的元素支持比较“是否等于目标值”，就可以使用 `find`。

### 79.2 函数原型

```cpp
template <class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T& value);
```

| 参数       | 含义                                                  |
| ---------- | ----------------------------------------------------- |
| `first`    | 查找范围起始迭代器                                    |
| `last`     | 查找范围结束迭代器（不包含）                          |
| `value`    | 目标值                                                |
| **返回值** | 指向第一个等于 `value` 的元素；若未找到，返回 `last` |

> `find` 依赖的是“相等性判断”。默认情况下，它会使用 `==` 来比较元素和目标值。

### 79.3 底层逻辑（伪代码）

```cpp
template <class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T& value) {
    for (; first != last; ++first) {
        if (*first == value) {
            return first;
        }
    }
    return last;
}
```

**时间复杂度**：O(n)，最坏情况下需要遍历整个范围。

### 79.4 基础用法

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    auto it = find(v.begin(), v.end(), 30);

    if (it != v.end()) {
        cout << "找到元素: " << *it << endl;                    // 30
        cout << "位置索引: " << (it - v.begin()) << endl;       // 2
    } else {
        cout << "没有找到 30" << endl;
    }

    return 0;
}
```

### 79.5 未找到时的处理

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 7, 9};

    auto it = find(v.begin(), v.end(), 4);

    if (it == v.end()) {
        cout << "4 不存在" << endl;
    }

    return 0;
}
```

> **重要**：当 `it == end()` 时，不能解引用 `*it`，否则就是未定义行为。

### 79.6 对自定义类型使用

对于自定义类型，`find` 默认要求该类型支持 `operator==`。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;

    Person(string n, int a) : name(n), age(a) {}

    bool operator==(const Person& other) const {
        return name == other.name && age == other.age;
    }
};

int main() {
    vector<Person> people = {
        {"Alice", 20},
        {"Bob", 17},
        {"Charlie", 25}
    };

    Person target("Bob", 17);
    auto it = find(people.begin(), people.end(), target);

    if (it != people.end()) {
        cout << "找到: " << it->name << "，年龄: " << it->age << endl;
    } else {
        cout << "未找到目标对象" << endl;
    }

    return 0;
}
```

> **教学提醒**：`operator==` 应该表达“对象是否真正相等”。如果只是想按某个成员查找，不建议随意改写“相等”的含义，更适合使用 `find_if`。

### 79.7 `find` 的典型特点

| 特点         | 说明                                      |
| ------------ | ----------------------------------------- |
| 查找方式     | 线性查找                                  |
| 匹配条件     | 固定为 `==`                               |
| 是否需要有序 | 不需要                                    |
| 返回结果     | 迭代器                                    |
| 适用场景     | 无序容器中查找某个具体值，或简单存在性判断 |

---

## 80. 常用查找算法 — `find_if`

### 80.1 为什么需要 `find_if`？

在上一阶段我们学过 `find`，它用来查找**等于某个值**的元素。但很多时候，我们的查找条件并不是简单的"等于"：

- 找第一个**大于 100** 的元素
- 找第一个**名字长度超过 5** 的字符串
- 找第一个**年龄 < 18** 的 `Person` 对象

这些场景下，`find` 无能为力，因为它只能判断 `==`。`find_if` 就是为此而生的——它允许你**自定义查找条件**。

### 80.2 函数原型

```cpp
template <class InputIterator, class UnaryPredicate>
InputIterator find_if(InputIterator first, InputIterator last, UnaryPredicate pred);
```

| 参数       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| `first`    | 查找范围的起始迭代器                                         |
| `last`     | 查找范围的结束迭代器（不包含）                               |
| `pred`     | 一元谓词（接受一个元素，返回 `bool`）                        |
| **返回值** | 指向**第一个**使 `pred` 返回 `true` 的元素的迭代器；若未找到，返回 `last` |

> **一元谓词（Unary Predicate）**：只接受一个参数并返回 `bool` 的可调用对象。可以是普通函数、函数对象（仿函数）或 Lambda 表达式。

### 80.3 底层逻辑（伪代码）

`find_if` 的内部实现非常直观——逐个遍历，对每个元素调用谓词：

```cpp
template <class InputIterator, class UnaryPredicate>
InputIterator find_if(InputIterator first, InputIterator last, UnaryPredicate pred) {
    for (; first != last; ++first) {
        if (pred(*first)) {  // 对解引用后的元素调用谓词
            return first;    // 找到了，立刻返回
        }
    }
    return last;  // 没找到，返回 last
}
```

**时间复杂度**：O(n)，最坏情况遍历整个范围。

### 80.4 使用方式一：普通函数作为谓词

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// 谓词函数：判断是否为偶数

int main() {
    vector<int> v = {1, 3, 5, 8, 7, 9};

    auto it = find_if(v.begin(), v.end(), isEven);

    if (it != v.end()) {
        cout << "找到第一个偶数: " << *it << endl;           // 输出: 8
        cout << "位置索引: " << (it - v.begin()) << endl;    // 输出: 3
    } else {
        cout << "没有偶数" << endl;
    }

    return 0;
}
```

### 80.5 使用方式二：Lambda 表达式（推荐）

Lambda 是最灵活、最常用的方式，尤其适合一次性使用的简单条件：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {10, 25, 30, 45, 50};

    // 查找第一个大于 30 的元素
    auto it = find_if(v.begin(), v.end(), [](int val) {
        return val > 30;
    });

    if (it != v.end()) {
        cout << "第一个大于30的元素: " << *it << endl;  // 输出: 45
    }

    return 0;
}
```

**Lambda 的优势**：
- 就地定义，不需要额外声明函数
- 可以通过**捕获列表**引用外部变量

```cpp
int threshold = 30;
auto it = find_if(v.begin(), v.end(), [threshold](int val) {
    return val > threshold;  // 使用外部变量 threshold
});
```

### 80.6 使用方式三：仿函数（函数对象）

仿函数适合需要**携带状态**或**复用条件**的场景：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// 仿函数：大于指定阈值
class GreaterThan {
public:
    int threshold;
    GreaterThan(int t) : threshold(t) {}

    bool operator()(int val) const {
        return val > threshold;
    }
};

int main() {
    vector<int> v = {10, 25, 30, 45, 50};

    auto it = find_if(v.begin(), v.end(), GreaterThan(30));

    if (it != v.end()) {
        cout << "第一个大于30的元素: " << *it << endl;  // 输出: 45
    }

    return 0;
}
```

### 80.7 对自定义类型使用 `find_if`

这是实际开发中最常见的场景。当容器存储的是自定义对象时，通常需要根据对象的**某个成员**来查找：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;
    Person(string n, int a) : name(n), age(a) {}
};

int main() {
    vector<Person> people = {
        {"Alice", 20},
        {"Bob", 17},
        {"Charlie", 25},
        {"David", 15}
    };

    // 查找第一个未成年人（age < 18）
    auto it = find_if(people.begin(), people.end(), [](const Person& p) {
        return p.age < 18;
    });

    if (it != people.end()) {
        cout << "第一个未成年人: " << it->name
             << ", 年龄: " << it->age << endl;
        // 输出: 第一个未成年人: Bob, 年龄: 17
    }

    // 按名字查找
    string target = "Charlie";
    auto it2 = find_if(people.begin(), people.end(), [&target](const Person& p) {
        return p.name == target;
    });

    if (it2 != people.end()) {
        cout << "找到 " << it2->name << ", 年龄: " << it2->age << endl;
        // 输出: 找到 Charlie, 年龄: 25
    }

    return 0;
}
```

### 80.8 `find_if` 与 `find_if_not`

C++11 引入了 `find_if_not`，它查找**第一个使谓词返回 `false`** 的元素，相当于取反：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {2, 4, 6, 7, 8, 10};

    // find_if_not：找第一个"不是偶数"的元素 → 即第一个奇数
    auto it = find_if_not(v.begin(), v.end(), [](int val) {
        return val % 2 == 0;
    });

    if (it != v.end()) {
        cout << "第一个非偶数: " << *it << endl;  // 输出: 7
    }

    return 0;
}
```

**`find_if_not` 的等价写法**：

```cpp
// 这两行完全等价：
find_if_not(v.begin(), v.end(), pred);
find_if(v.begin(), v.end(), [&pred](const auto& x) { return !pred(x); });
```

### 80.9 实战技巧：查找所有满足条件的元素

`find_if` 只返回第一个匹配，如果要找**所有**匹配的元素，需要循环调用：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 4, 7, 2, 8, 3, 9, 6};

    cout << "所有大于5的元素: ";

    auto it = v.begin();
    while ((it = find_if(it, v.end(), [](int val) { return val > 5; })) != v.end()) {
        cout << *it << " ";
        ++it;  // 移动到下一个位置，避免死循环！
    }
    cout << endl;
    // 输出: 7 8 9 6

    return 0;
}
```

> **注意**：每次找到后必须 `++it`，否则会反复找到同一个元素，死循环！

### 80.10 `find` vs `find_if` 总结

| 对比项   | `find`               | `find_if`              |
| -------- | -------------------- | ---------------------- |
| 查找条件 | 固定的 `==` 比较     | 自定义谓词（任意条件） |
| 参数     | 传一个**值**         | 传一个**可调用对象**   |
| 灵活性   | 低                   | 高                     |
| 典型场景 | 查找等于某个值的元素 | 查找满足复杂条件的元素 |

---

## 81. 常用查找算法 — `adjacent_find`

### 81.1 什么是 `adjacent_find`？

`adjacent_find` 用于查找容器中**第一对相邻的重复元素**。所谓"相邻"，就是在容器中紧挨着的两个元素。

更准确地说：它查找第一个位置 `i`，使得 `*i == *(i+1)`。

### 81.2 函数原型

```cpp
// 版本一：默认使用 == 比较
template <class ForwardIterator>
ForwardIterator adjacent_find(ForwardIterator first, ForwardIterator last);

// 版本二：自定义比较条件
template <class ForwardIterator, class BinaryPredicate>
ForwardIterator adjacent_find(ForwardIterator first, ForwardIterator last,
                              BinaryPredicate pred);
```

| 参数            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| `first`, `last` | 查找范围 `[first, last)`                                     |
| `pred`（可选）  | 二元谓词，接受两个相邻元素，返回 `bool`                      |
| **返回值**      | 指向**第一对**满足条件的相邻元素中的**前一个**的迭代器；若未找到，返回 `last` |

### 81.3 底层逻辑（伪代码）

```cpp
template <class ForwardIterator>
ForwardIterator adjacent_find(ForwardIterator first, ForwardIterator last) {
    if (first == last) return last;

    ForwardIterator next = first;
    ++next;

    while (next != last) {
        if (*first == *next) {  // 比较相邻的两个元素
            return first;       // 返回前一个
        }
        ++first;
        ++next;
    }
    return last;
}
```

**时间复杂度**：O(n)。

### 81.4 基础用法：查找相邻重复元素

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 3, 4, 5, 5, 5, 6};

    auto it = adjacent_find(v.begin(), v.end());

    if (it != v.end()) {
        cout << "第一对相邻重复元素: " << *it
             << "（位置索引: " << (it - v.begin()) << "）" << endl;
        // 输出: 第一对相邻重复元素: 3（位置索引: 2）
    } else {
        cout << "没有相邻重复元素" << endl;
    }

    return 0;
}
```

注意：返回的迭代器指向这一对中的**第一个**元素，所以 `*it` 和 `*(it+1)` 都等于 3。

### 81.5 无相邻重复的情况

```cpp
vector<int> v = {1, 2, 3, 4, 5};  // 没有相邻重复

auto it = adjacent_find(v.begin(), v.end());
// it == v.end()，没有找到
```

> **易混淆点**：`{1, 3, 1, 3}` 中虽然有重复元素 1 和 3，但它们不是**相邻**的，所以 `adjacent_find` 返回 `end()`。

### 81.6 查找所有相邻重复对

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 1, 2, 3, 3, 3, 4, 5, 5};

    cout << "所有相邻重复对:" << endl;

    auto it = v.begin();
    while ((it = adjacent_find(it, v.end())) != v.end()) {
        cout << "  位置 " << (it - v.begin())
             << ": " << *it << " == " << *(it + 1) << endl;
        ++it;  // 移动到下一个位置继续查找
    }

    return 0;
}
```

输出：
```
所有相邻重复对:
  位置 0: 1 == 1
  位置 3: 3 == 3
  位置 4: 3 == 3
  位置 7: 5 == 5
```

### 81.7 自定义比较条件

第二个重载版本允许你自定义"什么算相邻匹配"。这极大地扩展了 `adjacent_find` 的应用范围：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 2, 8, 5, 7};

    // 查找第一对"后一个元素 > 前一个元素的2倍"的相邻对
    auto it = adjacent_find(v.begin(), v.end(), [](int a, int b) {
        return b > 2 * a;
    });

    if (it != v.end()) {
        cout << "找到: " << *it << " 和 " << *(it + 1) << endl;
        // 找到: 1 和 3
    }

    return 0;
}
```

### 81.8 对自定义类型使用

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;
    Person(string n, int a) : name(n), age(a) {}

    // 重载 == 运算符（供默认版本使用）
    bool operator==(const Person& other) const {
        return age == other.age;
    }
};

int main() {
    vector<Person> people = {
        {"Alice", 20}, {"Bob", 20}, {"Charlie", 25}, {"David", 25}
    };

    // 默认版本：查找相邻的同龄人
    auto it = adjacent_find(people.begin(), people.end());

    if (it != people.end()) {
        cout << it->name << " 和 " << (it + 1)->name
             << " 同龄，都是 " << it->age << " 岁" << endl;
        // 输出: Alice 和 Bob 同龄，都是 20 岁
    }

    // 自定义条件：查找相邻的且名字首字母相同的人
    auto it2 = adjacent_find(people.begin(), people.end(),
        [](const Person& a, const Person& b) {
            return a.name[0] == b.name[0];
        });

    if (it2 != people.end()) {
        cout << it2->name << " 和 " << (it2 + 1)->name
             << " 名字首字母相同" << endl;
        // 未找到（A, B, C, D 都不同）
    }

    return 0;
}
```

### 81.9 应用场景总结

| 场景                 | 说明                                            |
| -------------------- | ----------------------------------------------- |
| 检测有序序列中的重复 | 排序后用 `adjacent_find` 可快速判断有没有重复值 |
| 检测信号数据的突变   | 自定义谓词检测相邻数据点差异过大                |
| 文本处理             | 查找连续重复字符（如 `"aabbcc"` 中的 `aa`）     |
| 数据校验             | 检查序列的单调性是否被打破                      |

---

## 82. 常用查找算法 — `binary_search`

### 82.1 前置知识：二分查找思想

二分查找是最经典的搜索算法之一。对于一个**已排序**的序列，每次取中间元素与目标比较：

- 如果中间元素 == 目标：找到了！
- 如果中间元素 < 目标：目标在右半部分
- 如果中间元素 > 目标：目标在左半部分

这样每次都能排除一半的元素，时间复杂度为 **O(log n)**，比线性查找的 O(n) 快得多。

### 82.2 ⚠️ 核心前提：序列必须有序！

> **这是使用 `binary_search` 最重要的前提条件。如果序列无序，结果是未定义的（不是报错，是给你一个错误的结果）！**

### 82.3 函数原型

```cpp
// 版本一：默认使用 < 比较（要求升序排列）
template <class ForwardIterator, class T>
bool binary_search(ForwardIterator first, ForwardIterator last, const T& value);

// 版本二：自定义比较器
template <class ForwardIterator, class T, class Compare>
bool binary_search(ForwardIterator first, ForwardIterator last, const T& value,
                   Compare comp);
```

| 参数            | 含义                                        |
| --------------- | ------------------------------------------- |
| `first`, `last` | 查找范围 `[first, last)`，**必须有序**      |
| `value`         | 要查找的目标值                              |
| `comp`（可选）  | 自定义比较器，必须与排序时使用的比较器一致  |
| **返回值**      | `bool`——存在返回 `true`，不存在返回 `false` |

> **关键限制**：`binary_search` **只能告诉你"有没有"，不能告诉你"在哪里"**。如果需要知道位置，请使用后面要讲的 `lower_bound`。

### 82.4 基础用法

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 7, 9, 11, 13};
    // 已经是升序排列 ✓

    bool found5  = binary_search(v.begin(), v.end(), 5);
    bool found6  = binary_search(v.begin(), v.end(), 6);

    cout << "5 存在吗？ " << (found5 ? "是" : "否") << endl;   // 是
    cout << "6 存在吗？ " << (found6 ? "是" : "否") << endl;   // 否

    return 0;
}
```

### 82.5 对未排序序列使用会怎样？

```cpp
vector<int> v = {5, 1, 3, 9, 7};  // 未排序！

bool result = binary_search(v.begin(), v.end(), 3);
// result 可能是 true，也可能是 false！
// 结果不可预测，属于未定义行为
```

**正确做法**：先排序，再查找。

```cpp
vector<int> v = {5, 1, 3, 9, 7};
sort(v.begin(), v.end());  // 先排序！
bool result = binary_search(v.begin(), v.end(), 3);  // 现在结果正确了
```

### 82.6 降序排序 + 自定义比较器

如果序列是**降序**排列的，必须提供与排序一致的比较器：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>  // greater<>
using namespace std;

int main() {
    vector<int> v = {13, 11, 9, 7, 5, 3, 1};  // 降序

    // 错误！默认 binary_search 假设升序
    // bool wrong = binary_search(v.begin(), v.end(), 7);

    // 正确：提供 greater<int>()，与降序排列一致
    bool found = binary_search(v.begin(), v.end(), 7, greater<int>());
    cout << "7 存在吗？ " << (found ? "是" : "否") << endl;  // 是

    return 0;
}
```

> **规则**：排序用什么比较器，`binary_search` 就用什么比较器！

### 82.7 对自定义类型使用

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;
    Person(string n, int a) : name(n), age(a) {}
};

int main() {
    vector<Person> people = {
        {"Alice", 15}, {"Bob", 20}, {"Charlie", 25}, {"David", 30}
    };
    // 按 age 升序排列 ✓

    // 比较器仍然描述的是“谁排在前面”
    auto cmp_full = [](const Person& a, const Person& b) {
        return a.age < b.age;
    };

    // 构造一个具有同年龄的“目标对象”
    Person target("", 25);
    bool found = binary_search(people.begin(), people.end(), target, cmp_full);

    cout << "年龄25的人存在吗？ " << (found ? "是" : "否") << endl;  // 是

    return 0;
}
```

这里有一个很容易误解的点：

- 当前示例中，序列元素类型和查找值类型都为 `Person`，所以比较器写成 `(const Person&, const Person&)` 就够了
- 如果你想直接拿 `25` 这样的 `int` 去查 `vector<Person>`，那就不能直接照搬这个比较器；更稳妥的做法通常是使用 `lower_bound` 做“按键查找”，或先构造一个同类目标对象

> **严格要求**：二分查找家族依赖的是“与排序一致的严格弱序关系”，不是随便写一个能比较大小的 Lambda 就可以。

### 82.8 `binary_search` vs `find` 对比

| 对比项     | `find`             | `binary_search`                                     |
| ---------- | ------------------ | --------------------------------------------------- |
| 前提条件   | 无                 | **序列必须有序**                                    |
| 时间复杂度 | O(n)               | O(log n)                                            |
| 返回值     | 迭代器（位置信息） | `bool`（仅存在性）                                  |
| 适用场景   | 小规模、无序容器   | 大规模、已排序容器                                  |
| 适用容器   | 所有容器           | 支持随机访问最佳，也可用于其他容器（但退化为 O(n)） |

### 82.9 性能注意事项

从**比较次数**看，`binary_search` 是 O(log n)。但如果迭代器不是随机访问迭代器，定位“中点”本身也要走很多步。

`binary_search` 在**随机访问迭代器**（如 `vector`、`deque`、数组）上最能发挥优势。对于**双向迭代器**（如 `list`），虽然语法上可以使用，但推进到中点需要线性移动，整体常数和实际性能都很差，往往不如直接线性查找。

> 对于 `set`/`map`，应该使用它们自带的 `.find()` 或 `.count()` 成员函数，复杂度为 O(log n)。

---

## 83. 常用查找算法 — `count`

### 83.1 功能说明

`count` 统计指定范围内**等于某个值**的元素个数。

### 83.2 函数原型

```cpp
template <class InputIterator, class T>
typename iterator_traits<InputIterator>::difference_type
count(InputIterator first, InputIterator last, const T& value);
```

简单理解：

```cpp
// 返回 [first, last) 中等于 value 的元素个数
int count(迭代器 first, 迭代器 last, 值 value);
```

返回值类型通常是 `ptrdiff_t`（一种有符号整数），但你可以用 `int` 或 `auto` 接收。

### 83.3 底层逻辑

```cpp
template <class InputIterator, class T>
auto count(InputIterator first, InputIterator last, const T& value) {
    int cnt = 0;
    for (; first != last; ++first) {
        if (*first == value) {
            ++cnt;
        }
    }
    return cnt;
}
```

**时间复杂度**：O(n)。

### 83.4 基础用法

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 3, 7, 3, 9};

    int n = count(v.begin(), v.end(), 3);
    cout << "3 出现了 " << n << " 次" << endl;  // 输出: 3 出现了 3 次

    int n2 = count(v.begin(), v.end(), 100);
    cout << "100 出现了 " << n2 << " 次" << endl;  // 输出: 100 出现了 0 次

    return 0;
}
```

### 83.5 统计字符出现次数

`count` 也可以作用于字符串（`string` 内部就是一个字符容器）：

```cpp
#include <iostream>
#include <string>
#include <algorithm>
using namespace std;

int main() {
    string s = "hello world";

    int n = count(s.begin(), s.end(), 'l');
    cout << "'l' 出现了 " << n << " 次" << endl;  // 输出: 3

    int n2 = count(s.begin(), s.end(), ' ');
    cout << "空格出现了 " << n2 << " 次" << endl;  // 输出: 1

    return 0;
}
```

### 83.6 对自定义类型使用

自定义类型需要**重载 `==` 运算符**：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;
    Person(string n, int a) : name(n), age(a) {}

    // 重载 == ：按年龄判断
    bool operator==(const Person& other) const {
        return age == other.age;
    }
};

int main() {
    vector<Person> people = {
        {"Alice", 20}, {"Bob", 25}, {"Charlie", 20}, {"David", 30}, {"Eve", 20}
    };

    Person target("", 20);  // 只关心年龄
    int n = count(people.begin(), people.end(), target);

    cout << "年龄为20的人有 " << n << " 个" << endl;  // 输出: 3

    return 0;
}
```

### 83.7 `count` 的快捷判断技巧

由于 `count` 返回整数，可以直接用在布尔上下文中：

```cpp
if (count(v.begin(), v.end(), 42)) {
    cout << "42 存在！" << endl;
}
// 0 为 false，非 0 为 true
```

但如果只想判断是否存在，用 `find` 更高效（找到一个就停止），而 `count` 会遍历完整个容器。

### 83.8 关联容器的 `.count()` 成员函数

`set`、`map`、`multiset`、`multimap` 以及它们的 `unordered_` 版本都有自己的 `.count()` 成员函数。对于这些关联容器，优先使用成员函数而不是通用算法 `count`，因为成员函数会利用容器自身的数据结构：

```cpp
set<int> s = {1, 3, 5, 7};
cout << s.count(3) << endl;  // 1（set 中要么 0 要么 1）

multiset<int> ms = {1, 3, 3, 3, 5};
cout << ms.count(3) << endl;  // 3
```

> **准则**：如果容器有自己的查找/统计成员函数，优先使用成员函数；树形关联容器通常是 O(log n)，`unordered_` 容器平均为 O(1)。

---

## 84. 常用查找算法 — `count_if`

### 84.1 功能说明

`count_if` 是 `count` 的"带条件"版本：统计满足**自定义条件**的元素个数。它与 `find_if` 的关系，就像 `count` 与 `find` 的关系。

### 84.2 函数原型

```cpp
template <class InputIterator, class UnaryPredicate>
typename iterator_traits<InputIterator>::difference_type
count_if(InputIterator first, InputIterator last, UnaryPredicate pred);
```

| 参数            | 含义                             |
| --------------- | -------------------------------- |
| `first`, `last` | 统计范围 `[first, last)`         |
| `pred`          | 一元谓词                         |
| **返回值**      | 使 `pred` 返回 `true` 的元素个数 |

### 84.3 底层逻辑

```cpp
template <class InputIterator, class UnaryPredicate>
auto count_if(InputIterator first, InputIterator last, UnaryPredicate pred) {
    int cnt = 0;
    for (; first != last; ++first) {
        if (pred(*first)) {
            ++cnt;
        }
    }
    return cnt;
}
```

### 84.4 基础用法

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 统计偶数个数
    int evenCount = count_if(v.begin(), v.end(), [](int val) {
        return val % 2 == 0;
    });
    cout << "偶数个数: " << evenCount << endl;  // 输出: 5

    // 统计大于5的元素个数
    int gt5Count = count_if(v.begin(), v.end(), [](int val) {
        return val > 5;
    });
    cout << "大于5的个数: " << gt5Count << endl;  // 输出: 5

    return 0;
}
```

### 84.5 对自定义类型使用

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

class Person {
public:
    string name;
    int age;
    Person(string n, int a) : name(n), age(a) {}
};

int main() {
    vector<Person> people = {
        {"Alice", 20}, {"Bob", 17}, {"Charlie", 25},
        {"David", 15}, {"Eve", 30}
    };

    // 统计成年人（age >= 18）
    int adultCount = count_if(people.begin(), people.end(),
        [](const Person& p) { return p.age >= 18; });

    cout << "成年人数: " << adultCount << endl;  // 输出: 3

    // 统计名字长度大于 4 的人
    int longNameCount = count_if(people.begin(), people.end(),
        [](const Person& p) { return p.name.length() > 4; });

    cout << "名字超过4个字母的人数: " << longNameCount << endl;  // 输出: 2（Alice, Charlie）

    return 0;
}
```

### 84.6 使用仿函数

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

class InRange {
    int low, high;
public:
    InRange(int l, int h) : low(l), high(h) {}

    bool operator()(int val) const {
        return val >= low && val <= high;
    }
};

int main() {
    vector<int> v = {5, 12, 3, 8, 21, 15, 7, 9, 18, 2};

    // 统计在 [5, 15] 范围内的元素个数
    int n = count_if(v.begin(), v.end(), InRange(5, 15));
    cout << "在 [5, 15] 范围内的元素个数: " << n << endl;  // 输出: 6

    return 0;
}
```

### 84.7 `count` vs `count_if` 总结

| 对比项     | `count`       | `count_if`       |
| ---------- | ------------- | ---------------- |
| 条件       | 固定的 `==`   | 自定义谓词       |
| 参数       | 传一个值      | 传一个可调用对象 |
| 适用场景   | 精确计数      | 按条件计数       |
| 自定义类型 | 需要重载 `==` | 不需要重载运算符 |

---

## `[补]` 二分查找进阶

> 前面我们学了 `binary_search`，它只能判断"有没有"。在实际开发中，我们更常需要知道"**在哪里**"。STL 提供了三个更强大的二分查找函数来解决这个问题。

### 📍 前提回顾

以下三个函数与 `binary_search` 一样，**要求序列必须有序**（默认升序，或使用与排序一致的比较器）。

---

### `[补]` `lower_bound` — 查找第一个 ≥ value 的位置

#### 函数原型

```cpp
template <class ForwardIterator, class T>
ForwardIterator lower_bound(ForwardIterator first, ForwardIterator last, const T& value);

template <class ForwardIterator, class T, class Compare>
ForwardIterator lower_bound(ForwardIterator first, ForwardIterator last, const T& value,
                            Compare comp);
```

| 参数            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| `first`, `last` | 有序范围 `[first, last)`                                     |
| `value`         | 要查找的目标值                                               |
| `comp`          | 自定义比较器                                                 |
| **返回值**      | 指向第一个 **≥ value** 的元素的迭代器；若不存在，返回 `last` |

#### 核心语义

`lower_bound` 回答的问题是：**如果我要把 `value` 插入到有序序列中，应该插在哪里（保持有序）？**

更正式地说：它返回第一个"不小于" `value` 的位置。

> 对默认升序比较来说，"不小于"就是 `>=`；如果使用自定义比较器，更严谨的表述应是：返回第一个**不在 `value` 前面**的位置。

#### 图解理解

```
有序序列:  1  3  5  5  5  7  9
                ↑
           lower_bound(5) → 指向第一个 5（索引 2）

如果查找 4:
有序序列:  1  3  5  5  5  7  9
                ↑
           lower_bound(4) → 指向第一个 ≥ 4 的元素，即 5（索引 2）

如果查找 6:
有序序列:  1  3  5  5  5  7  9
                         ↑
                       lower_bound(6) → 指向 7（索引 5）

如果查找 10:
有序序列:  1  3  5  5  5  7  9
                              ↑
                        lower_bound(10) → 指向 end()
```

#### 示例代码

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 5, 5, 7, 9};

    // 查找第一个 >= 5 的位置
    auto it = lower_bound(v.begin(), v.end(), 5);
    cout << "lower_bound(5) 的索引: " << (it - v.begin()) << endl;  // 2
    cout << "值: " << *it << endl;  // 5

    // 查找第一个 >= 4 的位置
    auto it2 = lower_bound(v.begin(), v.end(), 4);
    cout << "lower_bound(4) 的索引: " << (it2 - v.begin()) << endl;  // 2（第一个5）

    // 查找第一个 >= 6 的位置
    auto it3 = lower_bound(v.begin(), v.end(), 6);
    cout << "lower_bound(6) 的索引: " << (it3 - v.begin()) << endl;  // 5（值为7）

    // 超出范围
    auto it4 = lower_bound(v.begin(), v.end(), 100);
    if (it4 == v.end()) {
        cout << "100: 不存在 >= 100 的元素" << endl;
    }

    return 0;
}
```

#### 用 `lower_bound` 判断元素是否存在

```cpp
auto it = lower_bound(v.begin(), v.end(), value);
if (it != v.end() && *it == value) {
    // 找到了！it 指向该元素
    cout << "找到: " << *it << endl;
} else {
    // 没找到
    cout << "不存在" << endl;
}
```

> 可以把它理解为 `binary_search` 的典型实现思路：先找 `lower_bound`，再检查该位置是否等于目标值。

---

### `[补]` `upper_bound` — 查找第一个 > value 的位置

#### 函数原型

```cpp
template <class ForwardIterator, class T>
ForwardIterator upper_bound(ForwardIterator first, ForwardIterator last, const T& value);

template <class ForwardIterator, class T, class Compare>
ForwardIterator upper_bound(ForwardIterator first, ForwardIterator last, const T& value,
                            Compare comp);
```

| 参数       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| **返回值** | 指向第一个 **> value** 的元素的迭代器；若不存在，返回 `last` |

#### 核心语义

- `lower_bound`：第一个 **≥** value 的位置
- `upper_bound`：第一个 **>** value 的位置

两者的区别仅在于**是否包含等于的情况**。

#### 图解理解

```
有序序列:  1  3  5  5  5  7  9
                         ↑
                    upper_bound(5) → 指向第一个 > 5 的元素，即 7（索引 5）

对比 lower_bound:
                ↑
           lower_bound(5) → 指向第一个 >= 5 的元素，即第一个 5（索引 2）
```

#### 示例代码

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 5, 5, 7, 9};

    auto lo = lower_bound(v.begin(), v.end(), 5);  // 第一个 >= 5
    auto hi = upper_bound(v.begin(), v.end(), 5);  // 第一个 > 5

    cout << "lower_bound(5) 索引: " << (lo - v.begin()) << endl;  // 2
    cout << "upper_bound(5) 索引: " << (hi - v.begin()) << endl;  // 5

    // 等于 5 的元素个数 = upper_bound - lower_bound
    cout << "5 出现了 " << (hi - lo) << " 次" << endl;  // 3

    return 0;
}
```

> **核心公式**：`upper_bound(value) - lower_bound(value)` = **value 出现的次数**

#### `lower_bound` 与 `upper_bound` 的关系图

```
有序序列:  1  3  [5  5  5]  7  9
                 ↑         ↑
            lower_bound  upper_bound
                  (5)      (5)

[lower_bound, upper_bound) 就是所有等于 value 的元素范围
```

---

### `[补]` `equal_range` — 返回等于 value 的范围

#### 函数原型

```cpp
template <class ForwardIterator, class T>
pair<ForwardIterator, ForwardIterator>
equal_range(ForwardIterator first, ForwardIterator last, const T& value);

template <class ForwardIterator, class T, class Compare>
pair<ForwardIterator, ForwardIterator>
equal_range(ForwardIterator first, ForwardIterator last, const T& value,
            Compare comp);
```

| 参数       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| **返回值** | `pair<迭代器, 迭代器>`，等价于 `{lower_bound(value), upper_bound(value)}` |

#### 核心语义

`equal_range` 一次性返回“等值区间”的左右边界。你可以把它理解为：在一个接口里同时拿到 `lower_bound` 和 `upper_bound` 的结果。

返回的是一个 `pair`：
- `pair.first` = `lower_bound(value)`，即第一个 ≥ value 的位置
- `pair.second` = `upper_bound(value)`，即第一个 > value 的位置

`[pair.first, pair.second)` 就是所有等于 `value` 的元素的范围。

#### 示例代码

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {1, 3, 5, 5, 5, 7, 9};

    auto [lo, hi] = equal_range(v.begin(), v.end(), 5);  // C++17 结构化绑定

    cout << "equal_range(5):" << endl;
    cout << "  起始索引: " << (lo - v.begin()) << endl;   // 2
    cout << "  结束索引: " << (hi - v.begin()) << endl;   // 5
    cout << "  出现次数: " << (hi - lo) << endl;          // 3

    // 遍历所有等于 5 的元素
    cout << "  所有 5: ";
    for (auto it = lo; it != hi; ++it) {
        cout << *it << " ";
    }
    cout << endl;  // 输出: 5 5 5

    // 查找不存在的值
    auto [lo2, hi2] = equal_range(v.begin(), v.end(), 4);
    cout << "equal_range(4): 出现 " << (hi2 - lo2) << " 次" << endl;  // 0
    cout << "  插入位置索引: " << (lo2 - v.begin()) << endl;           // 2

    return 0;
}
```

#### C++11 写法（不使用结构化绑定）

```cpp
pair<vector<int>::iterator, vector<int>::iterator> result = equal_range(v.begin(), v.end(), 5);
// 或用 auto
auto result = equal_range(v.begin(), v.end(), 5);
cout << "起始: " << (result.first - v.begin()) << endl;
cout << "结束: " << (result.second - v.begin()) << endl;
```

---

### `[补]` 自定义比较器版本

以上三个函数都有接受自定义比较器的重载版本。使用规则：

> **排序时用什么比较器，查找时就用什么比较器！**

#### 降序序列

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

int main() {
    vector<int> v = {9, 7, 5, 5, 5, 3, 1};  // 降序

    // 使用 greater<int>()，与降序一致
    auto lo = lower_bound(v.begin(), v.end(), 5, greater<int>());
    auto hi = upper_bound(v.begin(), v.end(), 5, greater<int>());

    cout << "lower_bound(5) 索引: " << (lo - v.begin()) << endl;  // 2
    cout << "upper_bound(5) 索引: " << (hi - v.begin()) << endl;  // 5
    cout << "5 出现了 " << (hi - lo) << " 次" << endl;            // 3

    auto [elo, ehi] = equal_range(v.begin(), v.end(), 5, greater<int>());
    cout << "equal_range 也一样: " << (ehi - elo) << " 次" << endl;  // 3

    return 0;
}
```

> **注意语义变化**：
> - 升序时 `lower_bound` = 第一个 **≥** value
> - 降序（使用 `greater`）时 `lower_bound` = 第一个 **≤** value（因为"不在 value 前面"的含义变了）

更严谨地说，`lower_bound` 的真正语义始终是：

- 返回第一个使 `comp(element, value)` 为 `false` 的位置
- 在默认升序比较下，这等价于“第一个 `>= value`”
- 在 `greater<>` 的降序比较下，这等价于“第一个 `<= value`”

#### 对自定义类型使用

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

struct Student {
    string name;
    int score;
};

int main() {
    vector<Student> students = {
        {"Alice", 60}, {"Bob", 75}, {"Charlie", 85},
        {"David", 85}, {"Eve", 90}, {"Frank", 95}
    };
    // 按 score 升序排列 ✓

    auto cmp = [](const Student& a, const Student& b) {
        return a.score < b.score;
    };

    Student target{"", 85};

    auto lo = lower_bound(students.begin(), students.end(), target, cmp);
    auto hi = upper_bound(students.begin(), students.end(), target, cmp);

    cout << "分数为 85 的学生:" << endl;
    for (auto it = lo; it != hi; ++it) {
        cout << "  " << it->name << " (分数: " << it->score << ")" << endl;
    }
    // 输出:
    //   Charlie (分数: 85)
    //   David (分数: 85)

    cout << "共 " << (hi - lo) << " 人" << endl;

    return 0;
}
```

---

### `[补]` 四个二分查找函数总结

| 函数            | 返回值                 | 语义              | 用途                      |
| --------------- | ---------------------- | ----------------- | ------------------------- |
| `binary_search` | `bool`                 | 是否存在          | 仅判断存在性              |
| `lower_bound`   | 迭代器                 | 第一个 ≥ value    | 精确定位、插入点          |
| `upper_bound`   | 迭代器                 | 第一个 > value    | 配合 lower_bound 确定范围 |
| `equal_range`   | `pair<迭代器, 迭代器>` | 等于 value 的范围 | 一次性获取完整范围        |

**时间复杂度**：比较次数通常为对数级；若使用随机访问迭代器，可获得最好的实际性能。对 `list` 这类非随机访问迭代器，迭代器移动成本会明显更高。

**选择指南**：
- 只需要"有没有"→ `binary_search`
- 需要"在哪里"→ `lower_bound`
- 需要"有几个"或"所有等于它的元素"→ `equal_range`（或 `upper_bound - lower_bound`）
- 需要"插入位置"→ `lower_bound`

---

## `[补]` 最值算法

> 头文件：`<algorithm>`
> 这三个算法用于查找序列中的最小值、最大值，或同时查找两者。

---

### `[补]` `min_element` — 查找最小元素

#### 函数原型

```cpp
template <class ForwardIterator>
ForwardIterator min_element(ForwardIterator first, ForwardIterator last);

template <class ForwardIterator, class Compare>
ForwardIterator min_element(ForwardIterator first, ForwardIterator last, Compare comp);
```

| 参数            | 含义                                                    |
| --------------- | ------------------------------------------------------- |
| `first`, `last` | 查找范围 `[first, last)`                                |
| `comp`          | 自定义比较器（可选）                                    |
| **返回值**      | 指向范围中**最小**元素的迭代器；若范围为空，返回 `last` |

> 若有多个最小值，返回**第一个**。

#### 底层逻辑

```cpp
template <class ForwardIterator>
ForwardIterator min_element(ForwardIterator first, ForwardIterator last) {
    if (first == last) return last;

    ForwardIterator smallest = first;
    ++first;
    for (; first != last; ++first) {
        if (*first < *smallest) {
            smallest = first;
        }
    }
    return smallest;
}
```

**时间复杂度**：O(n)，恰好 n-1 次比较。

#### 基础用法

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {5, 2, 8, 1, 9, 3};

    auto it = min_element(v.begin(), v.end());

    cout << "最小值: " << *it << endl;                    // 1
    cout << "位置索引: " << (it - v.begin()) << endl;     // 3

    return 0;
}
```

#### 自定义比较器

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

struct Person {
    string name;
    int age;
};

int main() {
    vector<Person> people = {
        {"Alice", 30}, {"Bob", 17}, {"Charlie", 25}
    };

    // 找年龄最小的人
    auto it = min_element(people.begin(), people.end(),
        [](const Person& a, const Person& b) {
            return a.age < b.age;
        });

    cout << "年龄最小的人: " << it->name
         << " (" << it->age << " 岁)" << endl;
    // 输出: 年龄最小的人: Bob (17 岁)

    return 0;
}
```

#### 注意：`min_element` 返回迭代器，不是值

```cpp
// 如果只需要值，解引用即可
int minVal = *min_element(v.begin(), v.end());

// 如果需要索引
int minIdx = min_element(v.begin(), v.end()) - v.begin();

// 如果需要值和索引
auto it = min_element(v.begin(), v.end());
int minVal = *it;
int minIdx = it - v.begin();
```

---

### `[补]` `max_element` — 查找最大元素

#### 函数原型

```cpp
template <class ForwardIterator>
ForwardIterator max_element(ForwardIterator first, ForwardIterator last);

template <class ForwardIterator, class Compare>
ForwardIterator max_element(ForwardIterator first, ForwardIterator last, Compare comp);
```

用法与 `min_element` 完全对称，只是查找的是**最大**元素。

#### 示例

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {5, 2, 8, 1, 9, 3};

    auto it = max_element(v.begin(), v.end());

    cout << "最大值: " << *it << endl;                 // 9
    cout << "位置索引: " << (it - v.begin()) << endl;  // 4

    return 0;
}
```

#### 综合示例：同时使用 `min_element` 和 `max_element`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

int main() {
    vector<int> scores = {78, 92, 65, 88, 95, 70, 83};

    auto minIt = min_element(scores.begin(), scores.end());
    auto maxIt = max_element(scores.begin(), scores.end());

    cout << "最低分: " << *minIt << " (位置: " << (minIt - scores.begin()) << ")" << endl;
    cout << "最高分: " << *maxIt << " (位置: " << (maxIt - scores.begin()) << ")" << endl;
    cout << "分差: " << (*maxIt - *minIt) << endl;

    // 输出:
    // 最低分: 65 (位置: 2)
    // 最高分: 95 (位置: 4)
    // 分差: 30

    return 0;
}
```

> 思考：上面的代码对序列遍历了**两次**。有没有办法只遍历一次就同时得到最小和最大值？答案就是下面的 `minmax_element`。

---

### `[补]` `minmax_element` — 同时查找最小和最大元素（C++11）

#### 为什么需要 `minmax_element`？

如果你需要同时获取最小值和最大值，分别调用 `min_element` 和 `max_element` 需要遍历序列**两次**（2n 次比较）。`minmax_element` 只需遍历**一次**，最多 `3⌊n/2⌋` 次比较即可完成，效率更高。

#### 函数原型

```cpp
template <class ForwardIterator>
pair<ForwardIterator, ForwardIterator>
minmax_element(ForwardIterator first, ForwardIterator last);

template <class ForwardIterator, class Compare>
pair<ForwardIterator, ForwardIterator>
minmax_element(ForwardIterator first, ForwardIterator last, Compare comp);
```

| 参数       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| **返回值** | `pair<迭代器, 迭代器>` — `first` 指向最小元素，`second` 指向最大元素 |

> 若有多个最小值，返回**第一个**最小值；若有多个最大值，返回**最后一个**最大值。

#### 基础用法

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<int> v = {5, 2, 8, 1, 9, 3};

    auto [minIt, maxIt] = minmax_element(v.begin(), v.end());  // C++17

    cout << "最小值: " << *minIt << " (索引: " << (minIt - v.begin()) << ")" << endl;
    cout << "最大值: " << *maxIt << " (索引: " << (maxIt - v.begin()) << ")" << endl;

    // 输出:
    // 最小值: 1 (索引: 3)
    // 最大值: 9 (索引: 4)

    return 0;
}
```

#### C++11 写法

```cpp
auto result = minmax_element(v.begin(), v.end());
cout << "最小值: " << *(result.first) << endl;
cout << "最大值: " << *(result.second) << endl;
```

#### 自定义比较器

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

struct Student {
    string name;
    int score;
};

int main() {
    vector<Student> students = {
        {"Alice", 88}, {"Bob", 72}, {"Charlie", 95},
        {"David", 60}, {"Eve", 95}
    };

    auto [worst, best] = minmax_element(students.begin(), students.end(),
        [](const Student& a, const Student& b) {
            return a.score < b.score;
        });

    cout << "最低分: " << worst->name << " (" << worst->score << "分)" << endl;
    cout << "最高分: " << best->name << " (" << best->score << "分)" << endl;

    // 输出:
    // 最低分: David (60分)
    // 最高分: Eve (95分)
    // （有两个 95 分，返回最后一个最大值，即 Eve）

    return 0;
}
```

#### 空范围处理

```cpp
vector<int> empty_v;
auto [lo, hi] = minmax_element(empty_v.begin(), empty_v.end());
// lo == hi == empty_v.end()
// 解引用 end() 是未定义行为！所以必须先检查
if (lo != empty_v.end()) {
    cout << *lo << " " << *hi << endl;
}
```

---

### `[补]` 最值算法 vs 其他方式对比

```cpp
vector<int> v = {5, 2, 8, 1, 9, 3};
```

| 方式                          | 代码                          | 返回值                 | 比较次数      |
| ----------------------------- | ----------------------------- | ---------------------- | ------------- |
| `min_element` + `max_element` | 分别调用两次                  | 两个迭代器             | 2(n-1)        |
| `minmax_element`              | 一次调用                      | `pair<迭代器, 迭代器>` | ≤ 3⌊n/2⌋      |
| 排序后取首尾                  | `sort` + `v.front()/v.back()` | 值                     | O(n log n)    |
| 手写循环                      | 遍历一次维护两个变量          | 值                     | n-1 到 2(n-1) |

> **结论**：只需要最值之一 → 用 `min_element` 或 `max_element`；两个都需要 → 用 `minmax_element`。

---

### `[补]` 最值算法实战：归一化数据

一个典型的应用场景——将一组数据归一化到 `[0, 1]` 范围：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main() {
    vector<double> data = {12.5, 3.7, 25.0, 8.1, 19.3};

    auto [minIt, maxIt] = minmax_element(data.begin(), data.end());
    double minVal = *minIt;
    double maxVal = *maxIt;
    double range = maxVal - minVal;

    cout << "原始数据 → 归一化:" << endl;
    for (double x : data) {
        double normalized = (x - minVal) / range;
        cout << "  " << x << " → " << normalized << endl;
    }

    return 0;
}
```

输出：
```
原始数据 → 归一化:
  12.5 → 0.413146
  3.7 → 0
  25 → 1
  8.1 → 0.206573
  19.3 → 0.732394
```

---

### `[补]` 补充：`std::min` / `std::max` / `std::minmax`（值版本）

与 `min_element` / `max_element` 不同，STL 还提供了直接比较**两个值**（或初始化列表）的版本：

```cpp
#include <algorithm>
#include <iostream>
using namespace std;

int main() {
    // 比较两个值
    cout << min(3, 7) << endl;   // 3
    cout << max(3, 7) << endl;   // 7

    // C++11：比较初始化列表
    cout << min({5, 2, 8, 1}) << endl;  // 1
    cout << max({5, 2, 8, 1}) << endl;  // 8

    // C++11：minmax 返回 pair
    auto [lo, hi] = minmax({5, 2, 8, 1});
    cout << lo << " " << hi << endl;  // 1 8

    // 自定义比较
    string a = "hello", b = "hi";
    cout << min(a, b, [](const string& x, const string& y) {
        return x.length() < y.length();
    }) << endl;  // "hi"（更短的）

    return 0;
}
```

| 函数                                                    | 操作对象   | 返回值                   |
| ------------------------------------------------------- | ---------- | ------------------------ |
| `min(a, b)` / `max(a, b)`                               | 两个值     | 值的引用                 |
| `min({...})` / `max({...})`                             | 初始化列表 | 值                       |
| `min_element(first, last)` / `max_element(first, last)` | 迭代器范围 | **迭代器**（有位置信息） |

---

## 第二十一阶段总结：查找算法全家福

| 算法             | 功能                   | 前提     | 时间复杂度 | 返回值                 |
| ---------------- | ---------------------- | -------- | ---------- | ---------------------- |
| `find`           | 按值查找第一个匹配     | 无       | O(n)       | 迭代器                 |
| `find_if`        | 按条件查找第一个匹配   | 无       | O(n)       | 迭代器                 |
| `find_if_not`    | 按条件查找第一个不匹配 | 无       | O(n)       | 迭代器                 |
| `adjacent_find`  | 查找第一对相邻匹配     | 无       | O(n)       | 迭代器                 |
| `count`          | 按值统计个数           | 无       | O(n)       | 整数                   |
| `count_if`       | 按条件统计个数         | 无       | O(n)       | 整数                   |
| `binary_search`  | 判断是否存在           | **有序** | O(log n)   | `bool`                 |
| `lower_bound`    | 第一个 ≥ value 的位置  | **有序** | O(log n)   | 迭代器                 |
| `upper_bound`    | 第一个 > value 的位置  | **有序** | O(log n)   | 迭代器                 |
| `equal_range`    | 等于 value 的完整范围  | **有序** | O(log n)   | `pair<迭代器, 迭代器>` |
| `min_element`    | 查找最小元素           | 无       | O(n)       | 迭代器                 |
| `max_element`    | 查找最大元素           | 无       | O(n)       | 迭代器                 |
| `minmax_element` | 同时查找最小和最大     | 无       | O(n)       | `pair<迭代器, 迭代器>` |

> 对 `binary_search`、`lower_bound`、`upper_bound`、`equal_range` 而言，表中的 O(log n) 主要指比较次数；若容器不支持随机访问，实际迭代器移动成本会更高。

### 选择决策树

```
需要查找元素？
├── 序列已排序吗？
│   ├── 是 → 只判断存在性？
│   │        ├── 是 → binary_search
│   │        └── 否 → 需要位置/范围？
│   │                 ├── 位置 → lower_bound
│   │                 └── 范围 → equal_range
│   └── 否 → 查找条件？
│            ├── 精确匹配 → find
│            ├── 自定义条件 → find_if
│            └── 相邻匹配 → adjacent_find
│
需要统计个数？
├── 精确匹配 → count（关联容器用 .count()）
└── 自定义条件 → count_if
│
需要最值？
├── 只需最小 → min_element
├── 只需最大 → max_element
└── 两个都要 → minmax_element
```

---

以上就是**第二十一阶段：常用查找算法**的完整教程，涵盖了 STL 中所有常用的查找相关算法。掌握这些算法后，你应该能应对绝大多数的元素查找和统计需求。核心要点回顾：

1. **`find_if` 是最灵活的线性查找**——任意条件，Lambda 最方便
2. **二分查找家族必须有序**——`binary_search` 判断存在，`lower_bound`/`upper_bound` 定位，`equal_range` 获取范围
3. **排序用什么比较器，查找就用什么比较器**
4. **关联容器优先使用自带的成员函数**（`.find()`、`.count()`）
5. **同时需要最大最小值时，用 `minmax_element` 效率最高**
