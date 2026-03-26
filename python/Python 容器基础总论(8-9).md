# 第八编 Python容器与C++ STL的对照理解

---

# 第32章 Python容器与C++ STL的设计哲学比较

---

## 32.1 引言：为什么要做对照？

如果你已经学过C++的STL（Standard Template Library），那么在学习Python容器时，最大的陷阱不是"不会用"，而是**用C++的思维去写Python代码**。两种语言的容器体系虽然解决的问题相似（存储和管理数据），但它们背后的**设计哲学截然不同**。

本章的目标是帮助你理解这种根本性的差异，这样你在写Python时才能真正"用Python的方式思考"，而不是"用Python语法写C++逻辑"。

> **严谨说明**：本教程默认以**现代 Python 3（主要参考 Python 3.9+ 语法，涉及顺序保证等语义时以 Python 3.7+ 语言规范为准）**和**CPython 实现**为背景。凡是涉及对象内存占用、引用计数、容器底层布局等内容，都应理解为"以 CPython 为主的工程事实"，而不是所有 Python 实现都完全一致的强制规范。

---

## 32.2 C++ STL的设计哲学："零成本抽象"与"泛型编程"

### 32.2.1 核心理念

C++ STL由Alexander Stepanov设计，其核心哲学可以概括为：

1. **零成本抽象（Zero-Cost Abstraction）**：你不为你不用的东西付费。抽象不应带来运行时开销。
2. **泛型编程（Generic Programming）**：算法与数据结构分离——容器负责存数据，算法负责处理数据，迭代器是连接两者的桥梁。
3. **静态类型（Static Typing）**：一切在编译期确定，容器中元素类型固定。
4. **性能第一（Performance First）**：设计决策以运行时性能为最高优先级。

### 32.2.2 三大支柱：容器、算法、迭代器

```
┌──────────┐     ┌──────────┐    ┌──────────┐
│  容器     │◄── │  迭代器   │──► │  算法     │
│ Container │    │ Iterator │    │ Algorithm │
│           │    │          │    │           │
│ vector    │    │ begin()  │    │ sort()    │
│ map       │    │ end()    │    │ find()    │
│ set       │    │ ++       │    │ count()   │
│ ...       │    │ *        │    │ ...       │
└──────────┘     └──────────┘    └──────────┘
```

这种设计的精髓在于**正交分离**：
- `M`个容器 + `N`个算法 = `M × N`种组合
- 只需要`M + N`个实现（而不是`M × N`个）

```cpp
// C++：容器与算法通过迭代器分离
#include <vector>
#include <list>
#include <algorithm>

std::vector<int> v = {3, 1, 4, 1, 5};
std::list<int> l = {3, 1, 4, 1, 5};

// 同一个 find 算法，作用于不同容器
auto it1 = std::find(v.begin(), v.end(), 4);  // 在vector中查找
auto it2 = std::find(l.begin(), l.end(), 4);  // 在list中查找

// 同一个 sort 算法（注意：list不支持随机访问迭代器，不能用std::sort）
std::sort(v.begin(), v.end());
```

### 32.2.3 C++的"你选择、你负责"

C++给你极大的控制权，但同时要求你理解：
- 每种容器的内存布局
- 迭代器的分类（输入/输出/前向/双向/随机访问）
- 操作的时间复杂度
- 迭代器失效规则

```cpp
// C++：你必须知道 vector 的 erase 会导致迭代器失效
std::vector<int> v = {1, 2, 3, 4, 5};
for (auto it = v.begin(); it != v.end(); ) {
    if (*it % 2 == 0) {
        it = v.erase(it);  // erase返回下一个有效迭代器
    } else {
        ++it;
    }
}
// 如果写成 v.erase(it); ++it; 就是未定义行为！
```

---

## 32.3 Python容器的设计哲学："实用主义"与"鸭子类型"

### 32.3.1 核心理念

Python的设计哲学由Guido van Rossum确立，体现在《The Zen of Python》中：

1. **可读性至上（Readability Counts）**：代码写出来是给人看的，顺便给机器执行。
2. **实用主义（Practicality Beats Purity）**：不追求理论上的完美，追求实际解决问题。
3. **鸭子类型（Duck Typing）**："如果它走起来像鸭子，叫起来像鸭子，那它就是鸭子。"不关心类型，关心行为。
4. **内置电池（Batteries Included）**：常用功能直接内置，开箱即用。
5. **人生苦短（Life is Short）**：减少样板代码，让开发者专注于解决问题。

### 32.3.2 "一切皆对象"的统一模型

在Python中，**容器自带方法**，算法直接嵌入容器或作为内置函数/语法糖提供：

```python
# Python：容器自身就"知道"如何完成常见操作
numbers = [3, 1, 4, 1, 5]

# 排序——直接调用方法
numbers.sort()

# 查找——直接用 in 运算符
if 4 in numbers:
    print("找到了")

# 去重——直接转换类型
unique_numbers = list(set(numbers))

# 统计——直接调用方法
count = numbers.count(1)

# 过滤——列表推导式
evens = [x for x in numbers if x % 2 == 0]
```

注意这里没有出现"迭代器"的显式概念。Python的迭代器是**隐含的、自动的**——`for`循环和`in`运算符在底层使用迭代协议，但你通常不需要直接操作它。

### 32.3.3 Python的"我来管、你别操心"

Python帮你处理了许多C++中需要手动管理的事情：

```python
# Python：通常不需要像 C++ 那样显式处理"迭代器失效规则"
numbers = [1, 2, 3, 4, 5]

# 方式一：列表推导式（创建新列表）
odds = [x for x in numbers if x % 2 != 0]

# 方式二：filter 函数
odds = list(filter(lambda x: x % 2 != 0, numbers))

# Python不会像C++那样因为迭代器失效触发未定义行为，
# 但这不意味着可以一边遍历一边随意修改容器；
# "遍历时修改容器"仍可能造成逻辑错误或直接抛出异常（见第38章）
```

---

## 32.4 核心差异对照表

| 设计维度     | C++ STL                    | Python                                       |
| ------------ | -------------------------- | -------------------------------------------- |
| **类型系统** | 静态类型，编译期确定       | 动态类型，运行期确定                         |
| **容器元素** | 同质（同一类型）           | 异质（可混合类型）                           |
| **内存管理** | 手动/RAII，精确控制        | 自动内存管理（CPython主要为引用计数+循环GC） |
| **算法位置** | 独立于容器，通过迭代器连接 | 内嵌于容器方法或内置函数                     |
| **迭代器**   | 显式、分层（5个类别）      | 隐式、统一（鸭子类型）                       |
| **性能哲学** | 零成本抽象，极致性能       | 开发效率优先，性能够用就好                   |
| **抽象层次** | 低层抽象，暴露底层细节     | 高层抽象，隐藏底层细节                       |
| **错误处理** | 未定义行为（越界等）       | 抛出异常（IndexError等）                     |
| **扩展方式** | 模板特化、继承             | 鸭子类型、协议                               |

---

## 32.5 类型系统的深层影响

### 32.5.1 C++的静态类型容器

```cpp
// C++：vector<int> 只能存int，编译器在编译期就检查
std::vector<int> numbers = {1, 2, 3};
// numbers.push_back("hello");  // 编译错误！类型不匹配

// 如果需要存不同类型，必须用 std::variant 或 std::any
#include <variant>
std::vector<std::variant<int, std::string>> mixed;
mixed.push_back(42);
mixed.push_back(std::string("hello"));
// 访问时需要 std::get 或 std::visit，非常繁琐
```

### 32.5.2 Python的动态类型容器

```python
# Python：列表可以存任意类型的混合
mixed = [1, "hello", 3.14, True, [1, 2], {"key": "value"}]

# 这在 Python 中语法上完全合法
# 但在工程代码里通常仍建议保持元素语义一致，必要时配合类型提示；
# 否则很多错误只能在运行时暴露
def process_numbers(lst):
    return sum(lst)

# 如果 lst 里混入了字符串，运行时才会报 TypeError
# process_numbers([1, 2, "three"])  # TypeError!
```

### 32.5.3 Python的类型提示：可选的静态检查

Python 3.5+引入了类型提示（Type Hints），这是一个重要的"桥梁"概念：

```python
# Python 3.9+ 推荐直接使用内置类型作为泛型（PEP 585）
# 如果是低于 3.9 的版本，需要 from typing import List, Dict

# 类型提示：声明意图，但不强制执行
def process_scores(scores: list[int]) -> float:
    """计算平均分"""
    return sum(scores) / len(scores)

# 运行时不会因为类型提示而报错
# 但 mypy 等静态检查工具可以在运行前发现类型问题
students: dict[str, list[int]] = {
    "Alice": [90, 85, 92],
    "Bob": [78, 88, 95],
}
```

> **给C++程序员的提示**：Python的类型提示类似于C++的模板参数，但它是**可选的**、**不强制的**。就像给代码加注释一样，它帮助理解代码意图，但Python解释器不会根据它做类型检查。你可以把它理解为一种"文档化的类型约束"。

---

## 32.6 迭代器模型的差异

### 32.6.1 C++的迭代器层次体系

C++ STL定义了5种迭代器，形成一个层次结构：

```
InputIterator    OutputIterator
       \              /
     ForwardIterator
           |
     BidirectionalIterator
           |
     RandomAccessIterator
```

不同容器提供不同级别的迭代器，不同算法要求不同级别的迭代器：

```cpp
// 随机访问迭代器：vector, deque, array
std::vector<int> v = {3, 1, 4};
auto it = v.begin();
it += 2;          // 随机跳转 O(1) ✓
std::sort(v.begin(), v.end());  // sort需要随机访问迭代器 ✓

// 双向迭代器：list, set, map
std::list<int> l = {3, 1, 4};
auto it2 = l.begin();
// it2 += 2;     // 编译错误！list只提供双向迭代器
++it2; ++it2;    // 必须逐步移动
// std::sort(l.begin(), l.end());  // 编译错误！sort需要随机访问迭代器
l.sort();        // list有自己的sort成员函数
```

### 32.6.2 Python的迭代协议

Python没有这种层次体系。更准确地说，Python把"可迭代对象"和"迭代器对象"区分开来：

```python
# 这个类既是"可迭代对象"，也是"迭代器对象"
# 作为迭代器，它需要实现 __iter__ 和 __next__
class CountDown:
    """一个简单的自定义可迭代对象"""
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self  # 返回迭代器对象（自身）
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration  # 迭代结束
        self.current -= 1
        return self.current + 1

# 用法——和内置容器完全一致
for n in CountDown(5):
    print(n)  # 5, 4, 3, 2, 1

# list()、sum()、min() 等所有接受可迭代对象的函数都能用
print(list(CountDown(5)))    # [5, 4, 3, 2, 1]
print(sum(CountDown(5)))     # 15
```

> **核心区别**：C++的迭代器是按"能力"分类的（能不能随机跳转？能不能向后移动？），Python的迭代器是按"协议"统一的（迭代器对象通常实现`__iter__`和`__next__`）。Python不关心你"怎么实现的"，只关心你"能不能迭代"。

> **补充严谨点**：上面的说法针对"迭代器对象"本身。对一般"可迭代对象"来说，常见做法是实现`__iter__()`并返回一个迭代器；此外，Python还兼容一部分旧式序列协议对象。

---

## 32.7 错误处理哲学的差异

### 32.7.1 C++："相信程序员"——未定义行为

```cpp
// C++：越界访问 vector 是未定义行为（UB）
std::vector<int> v = {1, 2, 3};
int x = v[10];   // 未定义行为！不会报错，但结果不可预测
                  // 可能是垃圾值、可能崩溃、可能看起来正常

// 安全访问需要主动使用 at()
try {
    int y = v.at(10);  // 抛出 std::out_of_range 异常
} catch (const std::out_of_range& e) {
    std::cerr << e.what() << std::endl;
}
```

### 32.7.2 Python："请求宽恕而非许可"——异常优先

```python
# Python：越界访问一定会抛出异常
numbers = [1, 2, 3]

# 越界——直接报错
try:
    x = numbers[10]
except IndexError as e:
    print(f"索引越界: {e}")  # IndexError: list index out of range

# 字典访问不存在的键——也直接报错
d = {"a": 1, "b": 2}
try:
    value = d["c"]
except KeyError as e:
    print(f"键不存在: {e}")  # KeyError: 'c'

# Python 风格是 EAFP（Easier to Ask Forgiveness than Permission）
# "先做了再说，出错了再处理"
# 而不是 LBYL（Look Before You Leap）
# "先检查再做"

# EAFP 风格（Python 推荐）
try:
    value = d["c"]
except KeyError:
    value = "default"

# 或更简洁地
value = d.get("c", "default")  # Python容器自带安全访问方法
```

---

## 32.8 "抽象层次"的根本差异

这是理解两种语言设计哲学的**最关键的一点**：

### C++：暴露底层，让你做选择

```cpp
// C++ 要求你理解底层才能做出正确选择
// 例：存储学生成绩

// 选择1：vector——连续内存，缓存友好，随机访问O(1)
std::vector<int> scores_v;

// 选择2：list——链表，插入删除O(1)，但遍历缓存不友好
std::list<int> scores_l;

// 选择3：deque——双端队列，两端操作O(1)
std::deque<int> scores_d;

// 你必须根据访问模式选择最合适的容器
// 选错了不会报错，但性能可能差很多倍
```

### Python：隐藏底层，给你最佳默认选择

```python
# Python：在绝大多数场景下，只需要 list
scores = [90, 85, 92, 78, 88]

# 不需要考虑"连续内存还是链表"
# 不需要考虑"缓存友好性"
# list 在绝大多数日常开发场景下都足够好

# 如果真的需要特殊的数据结构，Python 提供了高级容器
from collections import deque
# deque 在两端操作时有更好的性能
scores_deque = deque([90, 85, 92, 78, 88])
scores_deque.appendleft(95)  # O(1)
```

---

## 32.9 本章小结

| 维度           | C++ STL                  | Python                           |
| -------------- | ------------------------ | -------------------------------- |
| **设计目标**   | 极致性能 + 泛型抽象      | 开发效率 + 实用性                |
| **核心口号**   | "不为不使用的东西付费"   | "人生苦短，我用Python"           |
| **容器与算法** | 分离（通过迭代器桥接）   | 融合（方法 + 内置函数 + 语法糖） |
| **类型系统**   | 静态、同质容器           | 动态、异质容器                   |
| **迭代器**     | 显式分层（5个级别）      | 隐式统一（鸭子类型协议）         |
| **错误哲学**   | 信任程序员（UB）         | 安全优先（异常）                 |
| **学习曲线**   | 陡——需要理解大量底层概念 | 缓——直觉式API设计                |

> **一句话总结**：C++ STL是给"需要极致性能的专家"设计的工具箱，Python容器是给"需要快速解决问题的所有人"设计的工具箱。两者没有优劣之分，只有适用场景之分。

---

# 第33章 list、tuple、dict、set与vector、pair、map、set的对应关系

---

## 33.1 引言：对应关系不是"等号"

在开始对照之前，必须先强调一个关键认知：

> **Python的容器和C++的容器之间是"近似对应"，而非"等价替换"。**

它们在功能上有重叠，但在实现、行为、性能特征上可能有本质区别。本章会逐一对照，既讲"相似之处"帮你建立快速映射，也讲"关键差异"帮你避免踩坑。

---

## 33.2 list ↔ vector：最核心的序列容器

### 33.2.1 功能对照

| 特性         | Python `list`            | C++ `std::vector`      |
| ------------ | ------------------------ | ---------------------- |
| **底层结构** | 动态数组（存指针的数组） | 动态数组（存值的数组） |
| **元素类型** | 异质（可混合类型）       | 同质（固定类型）       |
| **随机访问** | O(1)                     | O(1)                   |
| **末尾追加** | O(1) 均摊                | O(1) 均摊              |
| **头部插入** | O(n)                     | O(n)                   |
| **中间插入** | O(n)                     | O(n)                   |
| **自动扩容** | 是                       | 是                     |

### 33.2.2 基本操作对照

```python
# ==================== Python list ====================
# 创建
py_list = [1, 2, 3, 4, 5]
empty_list = []
repeated = [0] * 10           # [0, 0, 0, ..., 0]  10个0

# 访问
first = py_list[0]             # 首元素
last = py_list[-1]             # 末尾元素（负索引！C++没有）
sub = py_list[1:3]             # 切片 [2, 3]（C++没有原生切片）

# 修改
py_list.append(6)              # 末尾追加
py_list.insert(0, 0)           # 指定位置插入
py_list.extend([7, 8, 9])     # 批量追加
py_list.pop()                  # 弹出末尾
py_list.pop(0)                 # 弹出指定位置
py_list.remove(3)              # 按值删除（第一个匹配项）

# 查找
idx = py_list.index(4)         # 查找元素索引
exists = 4 in py_list          # 成员检测

# 排序 (底层为 Timsort 算法，天然稳定排序)
py_list.sort()                 # 原地排序
py_list.sort(reverse=True)     # 降序
py_list.sort(key=abs)          # 按自定义键排序
sorted_copy = sorted(py_list)  # 返回新列表（不修改原列表）

# 大小
length = len(py_list)
```

```cpp
// ==================== C++ vector ====================
#include <vector>
#include <algorithm>

// 创建
std::vector<int> cpp_vec = {1, 2, 3, 4, 5};
std::vector<int> empty_vec;
std::vector<int> repeated(10, 0);  // 10个0

// 访问
int first = cpp_vec[0];            // 首元素（或 .front()）
int last = cpp_vec.back();         // 末尾元素
// 没有负索引！没有原生切片！

// 修改
cpp_vec.push_back(6);              // 末尾追加
cpp_vec.insert(cpp_vec.begin(), 0);// 指定位置插入
// 批量追加
std::vector<int> more = {7, 8, 9};
cpp_vec.insert(cpp_vec.end(), more.begin(), more.end());
cpp_vec.pop_back();                // 弹出末尾（注意：不返回值！）
cpp_vec.erase(cpp_vec.begin());    // 删除指定位置
// 按值删除——传统做法需要 erase-remove 惯用法：
cpp_vec.erase(std::remove(cpp_vec.begin(), cpp_vec.end(), 3), cpp_vec.end());
// 注：C++20 引入了更加简洁的 std::erase，更接近 Python 的直观理念：
// std::erase(cpp_vec, 3);

// 查找——需要用算法库
auto it = std::find(cpp_vec.begin(), cpp_vec.end(), 4);
bool exists = it != cpp_vec.end();

// 排序——需要用算法库 (底层为 Introsort 算法，不稳定排序)
std::sort(cpp_vec.begin(), cpp_vec.end());
// 注：如果需要稳定排序，必须显式调用 std::stable_sort
std::sort(cpp_vec.begin(), cpp_vec.end(), std::greater<int>());  // 降序
std::sort(cpp_vec.begin(), cpp_vec.end(), [](int a, int b) {     // 自定义
    return std::abs(a) < std::abs(b);
});
// 没有直接的"返回新排序容器"的方式（需要先复制再排序）

// 大小
size_t length = cpp_vec.size();
```

### 33.2.3 关键差异深入：底层存储

这是最容易被忽视的差异：

```
C++ vector<int>:
┌─────┬─────┬─────┬─────┬─────┐
│  1  │  2  │  3  │  4  │  5  │   值直接存储在连续内存中
└─────┴─────┴─────┴─────┴─────┘   → 缓存友好，访问极快

Python list:
┌─────┬─────┬─────┬─────┬─────┐
│ ptr │ ptr │ ptr │ ptr │ ptr │   存储的是指向对象的指针
└──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘
   │     │     │     │     │
   ▼     ▼     ▼     ▼     ▼
 ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
 │ 1 │ │ 2 │ │ 3 │ │ 4 │ │ 5 │  每个元素是堆上的独立对象
 └───┘ └───┘ └───┘ └───┘ └───┘   → 有间接寻址开销
```

**这意味着什么？**
- Python的`list`即使存整数，每个整数也是一个完整的Python对象（以64位 CPython 为例，一个小整数对象常见约几十字节，常被近似写成约28字节）
- C++ `vector<int>`中每个`int`只占4字节
- 存储100万个整数时，Python对象模型通常会比C++原生整型占用明显更多内存；若只做数量级估算，常会看到"Python数十MB vs C++数MB"这种对比

> **实际影响**：如果你在Python中需要存储大量同类型数值数据，通常应优先考虑`numpy.ndarray`而不是`list`，因为`ndarray`采用紧凑的同类型连续存储，更接近C/C++数组或`vector`处理数值数据时的内存模型。

### 33.2.4 Python独有的强大特性

```python
# 1. 负索引——从末尾开始计数
lst = [10, 20, 30, 40, 50]
print(lst[-1])    # 50（最后一个）
print(lst[-2])    # 40（倒数第二个）

# 2. 切片——获取子序列
print(lst[1:3])   # [20, 30]   从索引1到2
print(lst[:3])    # [10, 20, 30] 前3个
print(lst[2:])    # [30, 40, 50] 从索引2到末尾
print(lst[::2])   # [10, 30, 50] 每隔一个取一个
print(lst[::-1])  # [50, 40, 30, 20, 10] 反转！

# 3. 列表推导式——创建新列表的优雅方式
squares = [x**2 for x in range(10)]            # [0, 1, 4, 9, ...]
evens = [x for x in range(20) if x % 2 == 0]  # [0, 2, 4, 6, ...]
matrix = [[0]*3 for _ in range(3)]             # 3x3零矩阵

# 4. 解包赋值
first, *middle, last = [1, 2, 3, 4, 5]
# first = 1, middle = [2, 3, 4], last = 5

# 5. 成员检测（语法层面）
if 30 in lst:
    print("找到了")  # 'in' 是 Python 关键字，而非函数调用
```

在C++中要实现上述功能，要么需要手写循环，要么需要引入第三方库，要么根本无法做到如此简洁。

---

## 33.3 tuple ↔ pair/tuple：不可变的有序组合

### 33.3.1 功能对照

| 特性         | Python `tuple`         | C++ `std::pair`     | C++ `std::tuple`     |
| ------------ | ---------------------- | ------------------- | -------------------- |
| **元素数量** | 任意个                 | 固定2个             | 任意个（编译期确定） |
| **元素类型** | 异质                   | 异质（但要声明）    | 异质（但要声明）     |
| **可变性**   | 不可变                 | 可变                | 可变                 |
| **可哈希**   | 是（如果元素都可哈希） | **否**（无默认哈希函数） | **否**（无默认哈希函数） |
| **访问方式** | 索引 `t[0]`            | `.first`, `.second` | `std::get<0>(t)`     |

### 33.3.2 基本操作对照

```python
# ==================== Python tuple ====================
# 创建
t1 = (1, 2, 3)
t2 = (1,)                  # 单元素元组，逗号不能省！
t3 = 1, 2, 3               # 括号可以省略
t4 = tuple([1, 2, 3])      # 从列表转化

# 访问
first = t1[0]
last = t1[-1]
sub = t1[1:3]              # 切片，返回新tuple

# 解包
x, y, z = t1
a, *rest = t1              # a=1, rest=[2, 3]

# 作为字典的键（因为不可变且可哈希）
location_data = {
    (35.6762, 139.6503): "Tokyo",
    (48.8566, 2.3522): "Paris",
}

# 命名元组——给元组的每个位置命名
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(3, 4)
print(p.x, p.y)    # 3 4（按名称访问）
print(p[0], p[1])  # 3 4（按索引访问也行）
```

```cpp
// ==================== C++ pair / tuple ====================
#include <tuple>
#include <utility>

// pair：固定2个元素
std::pair<int, std::string> p = {1, "hello"};
int first = p.first;
std::string second = p.second;

// tuple：任意个元素（但编译期固定）
std::tuple<int, double, std::string> t = {1, 3.14, "hello"};
int a = std::get<0>(t);
double b = std::get<1>(t);
std::string c = std::get<2>(t);

// C++17 结构化绑定（类似Python解包）
auto [x, y, z] = t;  // x=1, y=3.14, z="hello"

// pair 常用场景：map的元素
std::map<std::string, int> m = {{"a", 1}, {"b", 2}};
for (const auto& [key, value] : m) {  // 结构化绑定
    // key 和 value 对应 pair 的 first 和 second
}

// 注意：C++ pair/tuple 是可变的！
p.first = 42;  // 合法
// Python tuple 是不可变的！
```

### 33.3.3 关键差异：可变性

这是最重要的差异——**Python的tuple不可变，C++的pair/tuple可变**：

```python
# Python：tuple 不可变
t = (1, 2, 3)
# t[0] = 10  # TypeError: 'tuple' object does not support item assignment

# 不可变的好处：
# 1. 可以作为 dict 的键和 set 的元素
my_dict = {(1, 2): "坐标A", (3, 4): "坐标B"}
my_set = {(1, 2), (3, 4)}

# 2. 只读共享更省心——不可变对象不会在原地被改写
#    但这不等于"并发编程自动安全"；线程安全仍取决于运行时、共享方式和复合操作
# 3. 表达"这组数据不应被修改"的语义
# 4. 只有当 tuple 内部元素也都可哈希时，它才能作为 dict 键或 set 元素
```

```cpp
// C++：pair/tuple 是可变的，且标准库默认【不支持哈希】！
// 试图用 std::pair 作为 unordered_map 的键会导致编译报错，需手动提供类型特化的哈希函数。
std::pair<int, int> p = {1, 2};
p.first = 10;   // 完全合法

// C++无法像Python那样轻松表达"不可变"的语义
// 最接近的方式是 const
const std::pair<int, int> cp = {1, 2};
// cp.first = 10;  // 编译错误
```

### 33.3.4 Python独有的优势：多返回值的优雅语法

```python
# Python：函数返回多个值——本质上是返回tuple
def divide(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder  # 返回 tuple

q, r = divide(17, 5)  # 自动解包
print(f"17 ÷ 5 = {q} 余 {r}")  # 17 ÷ 5 = 3 余 2

# C++ 实现相同功能需要更多样板代码
```

```cpp
// C++：返回多个值——需要显式使用 pair 或 tuple
#include <tuple>

std::pair<int, int> divide(int a, int b) {
    return {a / b, a % b};
}

auto [q, r] = divide(17, 5);  // C++17结构化绑定
// 或之前的写法：
// auto result = divide(17, 5);
// int q = result.first;
// int r = result.second;
```

---

## 33.4 dict ↔ map/unordered_map：键值映射

### 33.4.1 功能对照

| 特性         | Python `dict`        | C++ `std::map`      | C++ `std::unordered_map` |
| ------------ | -------------------- | ------------------- | ------------------------ |
| **底层结构** | 哈希表               | 红黑树              | 哈希表                   |
| **有序性**   | 插入顺序（3.7+保证） | 键的排序顺序        | 无序                     |
| **查找**     | O(1) 平均            | O(log n)            | O(1) 平均                |
| **插入**     | O(1) 平均            | O(log n)            | O(1) 平均                |
| **键的要求** | 可哈希（`__hash__`） | 可比较（`<`运算符） | 可哈希（`std::hash`）    |

### 33.4.2 基本操作对照

```python
# ==================== Python dict ====================
# 创建
py_dict = {"name": "Alice", "age": 25, "city": "Beijing"}
empty_dict = {}
from_pairs = dict([("a", 1), ("b", 2)])
from_keys = dict.fromkeys(["x", "y", "z"], 0)  # {"x": 0, "y": 0, "z": 0}

# 注意：如果默认值是可变对象，它会被所有键共享
shared = dict.fromkeys(["u", "v"], [])
shared["u"].append(1)
print(shared)  # {'u': [1], 'v': [1]}

# 访问
name = py_dict["name"]              # "Alice"（键不存在会抛 KeyError）
age = py_dict.get("age", 0)         # 25（键不存在返回默认值，不抛异常）
email = py_dict.get("email", "N/A") # "N/A"

# 修改
py_dict["age"] = 26                 # 修改已有键
py_dict["email"] = "alice@example.com"  # 添加新键
py_dict.update({"age": 27, "phone": "123"})  # 批量更新

# 删除
del py_dict["phone"]               # 删除指定键
removed = py_dict.pop("email")     # 删除并返回值
py_dict.setdefault("score", 100)   # 键不存在时设置默认值

# 遍历——这是Python dict最优雅的部分
for key in py_dict:                 # 遍历键
    print(key)

for key, value in py_dict.items(): # 遍历键值对
    print(f"{key}: {value}")

for value in py_dict.values():     # 遍历值
    print(value)

# 字典推导式
squares = {x: x**2 for x in range(1, 6)}  # {1:1, 2:4, 3:9, 4:16, 5:25}

# 成员检测
if "name" in py_dict:              # 检测键存在 O(1)
    print("name exists")
```

```cpp
// ==================== C++ map ====================
#include <map>
#include <unordered_map>

// 创建
std::map<std::string, int> cpp_map = {
    {"Alice", 90},
    {"Bob", 85},
    {"Charlie", 92}
};

// 访问
int score1 = cpp_map["Alice"];      // 90（注意：键不存在会插入默认值！）
int score2 = cpp_map.at("Bob");     // 85（键不存在抛 std::out_of_range）
auto it = cpp_map.find("David");
if (it != cpp_map.end()) {
    int score = it->second;         // 类似 Python 的 get()
}

// 修改
cpp_map["Alice"] = 95;              // 修改
cpp_map["David"] = 88;             // 添加（或修改）
cpp_map.insert({"Eve", 91});       // 插入（键存在则不插入）
// insert_or_assign (C++17)：插入或覆盖
cpp_map.insert_or_assign("Eve", 93);

// 删除
cpp_map.erase("David");

// 遍历
for (const auto& [key, value] : cpp_map) {  // C++17
    std::cout << key << ": " << value << "\n";
}
// 注意：map 遍历顺序是按键排序的！与 Python dict 的插入顺序不同

// 成员检测
if (cpp_map.count("Alice") > 0) {   // 或 cpp_map.contains("Alice") (C++20)
    std::cout << "Alice exists\n";
}
```

### 33.4.3 关键差异深入

**差异1：`[]` 运算符对不存在的键的行为**

```python
# Python：键不存在 → 抛异常
d = {"a": 1}
# value = d["b"]  # KeyError: 'b'
```

```cpp
// C++：键不存在 → 静默插入默认值！！
std::map<std::string, int> m = {{"a", 1}};
int value = m["b"];  // 不会报错！插入 {"b", 0} 并返回 0
// ← 这是C++新手最常见的bug之一
```

> **⚠️ 这是从C++迁移到Python时必须注意的核心差异！** Python的`dict`不会静默创建键值对，而是直接报错。反过来，如果你习惯了Python的行为，回到C++时也要小心`map`的`[]`运算符。

**差异2：有序性的含义不同**

```python
# Python 3.7+：dict 保持插入顺序
d = {}
d["banana"] = 3
d["apple"] = 1
d["cherry"] = 2
print(list(d.keys()))  # ['banana', 'apple', 'cherry'] -- 插入顺序
```

```cpp
// C++ map：按键的排序顺序
std::map<std::string, int> m;
m["banana"] = 3;
m["apple"] = 1;
m["cherry"] = 2;
for (auto& [k, v] : m) {
    std::cout << k << " ";
}
// 输出：apple banana cherry -- 字典序！不是插入顺序！
```

**差异3：Python的`defaultdict`——C++ `map[]`行为的显式版**

```python
from collections import defaultdict

# defaultdict：键不存在时自动创建默认值
word_count = defaultdict(int)  # 默认值为 int() → 0
for word in ["apple", "banana", "apple", "cherry", "banana", "apple"]:
    word_count[word] += 1
# word_count = {'apple': 3, 'banana': 2, 'cherry': 1}

# 这和 C++ map 的 [] 行为很像！
# 但 Python 用 defaultdict 显式声明了这种行为
# 普通 dict 不会这样做——更安全
```

---

## 33.5 set ↔ set/unordered_set：集合

### 33.5.1 功能对照

| 特性           | Python `set` | C++ `std::set` | C++ `std::unordered_set` |
| -------------- | ------------ | -------------- | ------------------------ |
| **底层结构**   | 哈希表       | 红黑树         | 哈希表                   |
| **有序性**     | 无序         | 排序           | 无序                     |
| **元素唯一性** | 是           | 是             | 是                       |
| **查找**       | O(1) 平均    | O(log n)       | O(1) 平均                |
| **元素要求**   | 可哈希       | 可比较         | 可哈希                   |

### 33.5.2 基本操作对照

```python
# ==================== Python set ====================
# 创建
py_set = {1, 2, 3, 4, 5}
empty_set = set()          # 注意：{} 是空字典，不是空集合！
from_list = set([1, 2, 2, 3, 3, 3])  # {1, 2, 3}（自动去重）

# 添加 / 删除
py_set.add(6)              # 添加元素
py_set.discard(3)          # 删除元素（不存在不报错）
py_set.remove(4)           # 删除元素（不存在抛 KeyError）
popped = py_set.pop()      # 随机弹出一个元素

# 成员检测——set 最核心的优势
if 2 in py_set:            # O(1)！比 list 的 O(n) 快得多
    print("存在")

# 集合运算——Python set 的杀手级特性
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

union = a | b              # 并集 {1, 2, 3, 4, 5, 6, 7, 8}
intersection = a & b       # 交集 {4, 5}
difference = a - b         # 差集 {1, 2, 3}
sym_diff = a ^ b           # 对称差集 {1, 2, 3, 6, 7, 8}

# 子集 / 超集 检测
{1, 2}.issubset(a)         # True：{1,2} 是 a 的子集
a.issuperset({1, 2})       # True：a 是 {1,2} 的超集
{1, 2} <= a                # True（运算符形式）
a >= {1, 2}                # True（运算符形式）

# 集合推导式
squares_set = {x**2 for x in range(-5, 6)}  # {0, 1, 4, 9, 16, 25}
```

```cpp
// ==================== C++ set ====================
#include <set>
#include <algorithm>

// 创建
std::set<int> cpp_set = {1, 2, 3, 4, 5};

// 添加 / 删除
cpp_set.insert(6);
cpp_set.erase(3);

// 成员检测
if (cpp_set.count(2) > 0) {  // 或 cpp_set.contains(2) (C++20)
    std::cout << "存在" << std::endl;
}

// 集合运算——必须用算法库 + 输出迭代器
std::set<int> a = {1, 2, 3, 4, 5};
std::set<int> b = {4, 5, 6, 7, 8};

// 并集
std::set<int> union_set;
std::set_union(a.begin(), a.end(), b.begin(), b.end(),
               std::inserter(union_set, union_set.begin()));

// 交集
std::set<int> inter_set;
std::set_intersection(a.begin(), a.end(), b.begin(), b.end(),
                      std::inserter(inter_set, inter_set.begin()));

// 差集
std::set<int> diff_set;
std::set_difference(a.begin(), a.end(), b.begin(), b.end(),
                    std::inserter(diff_set, diff_set.begin()));

// 对称差集
std::set<int> sym_diff_set;
std::set_symmetric_difference(a.begin(), a.end(), b.begin(), b.end(),
                              std::inserter(sym_diff_set, sym_diff_set.begin()));

// 子集检测
bool is_subset = std::includes(a.begin(), a.end(),
                               std::set<int>{1, 2}.begin(),
                               std::set<int>{1, 2}.end());
```

### 33.5.3 对比体会：Python的运算符优势

对比上面的代码量就能体会到 Python 在集合运算上的优雅程度：

```python
# Python：一目了然
result = (a | b) - (a & b)  # 等价于 a ^ b
```

```cpp
// C++：需要中间变量和大量样板代码
std::set<int> temp_union, temp_inter, result;
std::set_union(a.begin(), a.end(), b.begin(), b.end(),
               std::inserter(temp_union, temp_union.begin()));
std::set_intersection(a.begin(), a.end(), b.begin(), b.end(),
                      std::inserter(temp_inter, temp_inter.begin()));
std::set_difference(temp_union.begin(), temp_union.end(),
                    temp_inter.begin(), temp_inter.end(),
                    std::inserter(result, result.begin()));
```

---

## 33.6 frozenset：不可变集合（C++中无严格等价物）

```python
# Python frozenset：不可变集合
fs = frozenset([1, 2, 3])
# fs.add(4)  # AttributeError！不可变

# 最重要的用途：可以作为 dict 的键或放入另一个 set
tag_groups = {
    frozenset({"python", "web"}): "后端课程",
    frozenset({"html", "css"}): "前端课程",
}
```

```cpp
// C++ 标准库没有与 frozenset 严格等价的内置容器
// const std::set 只能表达"当前对象不可修改"，
// 但它不具备 frozenset 那种可哈希、可作为另一个集合元素的语义
const std::set<int> fs = {1, 2, 3};
// fs.insert(4);  // 编译错误
```

---

## 33.7 其他对应关系速查表

| Python                    | C++               | 说明                                                          |
| ------------------------- | ----------------- | ------------------------------------------------------------- |
| `list`                    | `vector`          | 最常用的序列容器                                              |
| `tuple`                   | `pair` / `tuple`  | 异质元素组合                                                  |
| `dict`                    | `unordered_map`   | 键值映射（哈希表实现最像）                                    |
| `dict`（保插入顺序）      | -                 | 保留插入顺序，但不按键排序                                    |
| `set`                     | `unordered_set`   | 无序唯一集合                                                  |
| `set`（排序后）           | `set`             | 有序唯一集合                                                  |
| `frozenset`               | -                 | 不可变且可哈希的集合，无严格等价物                            |
| `collections.deque`       | `deque`           | 双端队列                                                      |
| `collections.OrderedDict` | -                 | 保序字典；3.7+普通dict通常够用，但它仍支持`move_to_end`等特性 |
| `collections.defaultdict` | `map`的`[]`       | 默认值字典                                                    |
| `collections.Counter`     | `map<T, int>`     | 计数器                                                        |
| `heapq`（模块）           | `priority_queue`  | 堆 / 优先队列                                                 |
| `array.array`             | `vector<T>`       | 同质类型紧凑数组                                              |
| `bytes` / `bytearray`     | `vector<uint8_t>` | 字节序列                                                      |
| `str`                     | `string`          | 字符串（Python中不可变）                                      |
| -                         | `list`            | Python无链表内置容器                                          |
| -                         | `forward_list`    | Python无单向链表内置容器                                      |
| -                         | `array<T,N>`      | Python无固定大小数组                                          |
| -                         | `stack`           | Python用`list`当栈                                            |
| -                         | `queue`           | Python用`collections.deque`                                   |
| -                         | `multimap`        | Python用`dict[key] = list`                                    |
| -                         | `multiset`        | Python用`Counter`                                             |

---

## 33.8 本章小结：建立思维映射

当你从C++转向Python时，可以用以下"快速映射表"作为出发点：

```
      C++                    Python
  ──────────             ──────────
  vector<T>      →       list
  pair<A,B>      →       tuple (2元素)
  tuple<A,B,C>   →       tuple (多元素)
  map<K,V>       →       dict (仅保插入顺序，不按键排序)
  unordered_map  →       dict (性能特征最像)
  set<T>         →       set  (注意: Python是哈希实现)
  unordered_set  →       set  (性能特征最像)
```

但请始终记住：**这些只是近似映射，不是等号。** 每种Python容器都有自己独特的"Python味"——负索引、切片、推导式、运算符重载等等。真正掌握Python容器，不是"翻译C++代码"，而是学会"用Python的方式思考"。

---

# 第34章 Python"重容器、轻算法"与STL"容器+算法分离"的差异

---

## 34.1 引言：两种组织代码的哲学

如果用一句话概括C++ STL和Python容器体系最本质的区别，那就是：

> **C++ STL = 容器 + 算法 + 迭代器（三者分离）**
> **Python = 容器自带算法 + 内置函数 + 语法糖（三者融合）**

这不仅仅是API设计的差异，而是**组织代码的哲学差异**——它影响着你如何思考问题、如何分解任务、如何组合工具。

---

## 34.2 C++ STL的"算法独立"设计

### 34.2.1 为什么要分离？

C++ STL的设计者Stepanov认为：

- **算法是数学概念**，它不应该被绑定到特定的数据结构上
- `sort`就是排序——不管数据存在`vector`还是`deque`里，排序的逻辑是一样的
- 所以算法应该独立于容器，通过**迭代器**作为"适配层"连接两者

### 34.2.2 经典示例：用独立算法处理不同容器

```cpp
#include <vector>
#include <deque>
#include <array>
#include <algorithm>
#include <numeric>
#include <iostream>

int main() {
    std::vector<int> v = {5, 3, 1, 4, 2};
    std::deque<int> d = {5, 3, 1, 4, 2};
    std::array<int, 5> a = {5, 3, 1, 4, 2};

    // 同一个 sort 算法，作用于三种不同容器
    std::sort(v.begin(), v.end());
    std::sort(d.begin(), d.end());
    std::sort(a.begin(), a.end());

    // 同一个 accumulate 算法，计算三种容器的总和
    int sum_v = std::accumulate(v.begin(), v.end(), 0);
    int sum_d = std::accumulate(d.begin(), d.end(), 0);
    int sum_a = std::accumulate(a.begin(), a.end(), 0);

    // 同一个 count_if 算法，统计偶数
    int even_v = std::count_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
    int even_d = std::count_if(d.begin(), d.end(), [](int x) { return x % 2 == 0; });

    // 这就是"正交分离"的威力：M个容器 × N个算法 = M×N种组合
    // 但我们只写了 M + N 个实现
}
```

### 34.2.3 STL算法分类一览

C++ STL有大约80+个独立算法，分为以下几大类：

| 类别           | 典型算法                                             | 说明         |
| -------------- | ---------------------------------------------------- | ------------ |
| 非修改序列操作 | `find`, `count`, `for_each`, `any_of`, `all_of`      | 只读遍历     |
| 修改序列操作   | `copy`, `fill`, `replace`, `remove`, `unique`        | 修改元素     |
| 排序           | `sort`, `stable_sort`, `partial_sort`, `nth_element` | 排序相关     |
| 二分查找       | `binary_search`, `lower_bound`, `upper_bound`        | 有序序列查找 |
| 集合操作       | `set_union`, `set_intersection`, `set_difference`    | 集合运算     |
| 堆操作         | `make_heap`, `push_heap`, `pop_heap`                 | 堆维护       |
| 数值操作       | `accumulate`, `inner_product`, `partial_sum`         | 数值计算     |
| 排列           | `next_permutation`, `prev_permutation`               | 全排列       |

每个算法都是**独立函数**，接收**迭代器范围**作为参数。

---

## 34.3 Python的"容器即算法"设计

### 34.3.1 三层工具体系

Python不是没有"算法"，而是把算法分散在了三个层面：

```
                Python 的算法工具层次
    ┌─────────────────────────────────────────┐
    │ 第1层：容器方法（最常用）                   │
    │ list.sort(), dict.get(), set.union()    │
    ├─────────────────────────────────────────┤
    │ 第2层：内置函数（通用工具）                  │
    │ sorted(), len(), sum(), min(), max()    │
    │ map(), filter(), zip(), enumerate()     │
    ├─────────────────────────────────────────┤
    │ 第3层：语法糖（Python独有的优雅）            │
    │ 列表推导式、字典推导式、解包、切片           │
    │ in 运算符、for...in、| & - ^             │
    └─────────────────────────────────────────┘
```

### 34.3.2 第1层：容器方法

Python的容器"知道"自己能做什么操作，并把这些操作作为自己的方法：

```python
# list 自带的方法
numbers = [5, 3, 1, 4, 2]
numbers.sort()           # 排序（原地）
numbers.reverse()        # 反转（原地）
numbers.append(6)        # 追加
numbers.extend([7, 8])   # 批量追加
numbers.insert(0, 0)     # 插入
numbers.remove(3)        # 按值删除
numbers.pop()            # 弹出末尾
numbers.pop(0)           # 弹出指定位置
numbers.index(4)         # 查找索引
numbers.count(1)         # 计数
numbers.copy()           # 浅拷贝
numbers.clear()          # 清空

# dict 自带的方法
d = {"a": 1, "b": 2}
d.get("a", 0)            # 安全获取
d.setdefault("c", 3)     # 设置默认值
d.update({"d": 4})       # 批量更新
d.pop("a")               # 删除并返回
d.keys()                 # 键视图
d.values()               # 值视图
d.items()                # 键值对视图
# 注意：这些视图对象会随着原字典变化而动态反映最新内容，并不是独立拷贝的列表

# set 自带的方法
s = {1, 2, 3}
s.add(4)                 # 添加
s.discard(2)             # 安全删除
s.union({5, 6})          # 并集
s.intersection({2, 3})   # 交集
s.difference({1})        # 差集
s.issubset({1, 2, 3, 4}) # 子集检测
```

### 34.3.3 第2层：内置函数

Python提供的内置函数可以作用于**任意可迭代对象**（不仅仅是某种特定容器），这其实就是Python版的"通用算法"：

```python
# 这些函数接受任意可迭代对象——类似STL算法接受迭代器范围
data = [5, 3, 1, 4, 2]

# 聚合函数
total = sum(data)         # 求和 → 15（类似 std::accumulate）
maximum = max(data)       # 最大值 → 5（类似 std::max_element）
minimum = min(data)       # 最小值 → 1（类似 std::min_element）
length = len(data)        # 元素个数 → 5（类似 .size()）
has_any = any(x > 3 for x in data)  # 任一满足 → True（类似 std::any_of）
all_pos = all(x > 0 for x in data)  # 全部满足 → True（类似 std::all_of）

# 变换函数
sorted_data = sorted(data)              # 排序（返回新列表）（类似 sort + copy）
reversed_data = list(reversed(data))    # 反转
mapped = list(map(str, data))           # 映射 → ['5','3','1','4','2']（类似 std::transform）
filtered = list(filter(lambda x: x > 2, data))  # 过滤 → [5, 3, 4]（类似 copy_if）

# 组合函数
pairs = list(zip([1, 2, 3], ['a', 'b', 'c']))  # 配对 → [(1,'a'), (2,'b'), (3,'c')]
indexed = list(enumerate(data))                # 带索引枚举 → [(0,5),(1,3),(2,1),(3,4),(4,2)]
```

> **关键观察**：Python的内置函数接受"可迭代对象"，C++ STL算法接受"迭代器对"。前者更简洁，后者更灵活（可以指定子范围）。

### 第3层：语法糖——Python的杀手锏

语法糖是Python最独特的"算法层"，它让数据处理直接嵌入语法：

```python
data = [5, 3, 1, 4, 2]

# 列表推导式 ≈ transform + filter 的组合
squares = [x**2 for x in data]                     # [25, 9, 1, 16, 4]
even_squares = [x**2 for x in data if x % 2 == 0]  # [16, 4]

# 字典推导式
score_map = {name: score for name, score in
             zip(["Alice","Bob","Carol"], [90,85,92])}

# 集合推导式
unique_lengths = {len(word) for word in ["hello", "world", "hi", "hey"]}

# in 运算符——成员检测直接写在条件中
if 4 in data:
    print("找到了")

# 切片——获取子序列
first_three = data[:3]
every_other = data[::2]

# 解包赋值
first, *rest = data   # first=5, rest=[3,1,4,2]
```

## 34.4 同一个任务的对比：感受差异

### 任务1：统计列表中正数的平方和

```cpp
// C++ STL——组合独立算法
#include <vector>
#include <algorithm>
#include <numeric>

std::vector<int> data = {-3, 1, -4, 1, 5, -9, 2, 6};

// 方法1：先过滤再变换再累加（需要中间容器）
std::vector<int> positives;
std::copy_if(data.begin(), data.end(), std::back_inserter(positives),
             [](int x) { return x > 0; });

std::vector<int> squares;
std::transform(positives.begin(), positives.end(),
               std::back_inserter(squares),
               [](int x) { return x * x; });

int result = std::accumulate(squares.begin(), squares.end(), 0);

// 方法2：用 transform_reduce (C++17) 更紧凑
int result2 = std::transform_reduce(
    data.begin(), data.end(), 0, std::plus{},
    [](int x) { return x > 0 ? x * x : 0; });
```

```python
# Python——一行搞定
data = [-3, 1, -4, 1, 5, -9, 2, 6]

result = sum(x**2 for x in data if x > 0)  # 67
```

### 任务2：找出字符串列表中长度最大的字符串

```cpp
// C++
std::vector<std::string> words = {"apple", "hi", "banana", "kiwi"};
auto it = std::max_element(words.begin(), words.end(),
    [](const std::string& a, const std::string& b) {
        return a.size() < b.size();
    });
std::string longest = *it;  // "banana"
```

```python
# Python
words = ["apple", "hi", "banana", "kiwi"]
longest = max(words, key=len)  # "banana"
```

### 任务3：将字典按值降序排列

```cpp
// C++：map按值排序需要转换到vector<pair>再用sort
std::map<std::string, int> scores = {
    {"Alice", 90}, {"Bob", 78}, {"Carol", 95}
};
std::vector<std::pair<std::string, int>> vec(scores.begin(), scores.end());
std::sort(vec.begin(), vec.end(),
    [](const auto& a, const auto& b) {
        return a.second > b.second;
    });
```

```python
# Python
scores = {"Alice": 90, "Bob": 78, "Carol": 95}
sorted_scores = sorted(scores.items(), key=lambda x: x[1], reverse=True)
# [('Carol', 95), ('Alice', 90), ('Bob', 78)]
```

### 任务4：去重并保持原始顺序

```cpp
// C++：这是一个经典难题
// std::unique 要求已排序，会破坏顺序
// 需要手动用set辅助去重
std::vector<int> data = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3};
std::vector<int> result;
std::unordered_set<int> seen;
for (int x : data) {
    if (seen.insert(x).second) {  // insert返回pair<iter, bool>
        result.push_back(x);
    }
}
```

```python
# Python：利用 dict 保持插入顺序的特性（3.7+）
data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
result = list(dict.fromkeys(data))  # [3, 1, 4, 5, 9, 2, 6]
```

### 任务5：按条件分组

```cpp
// C++：需要手动构建map<key, vector<value>>
#include <map>
#include <vector>
#include <string>

std::vector<std::string> words = {"apple", "ant", "banana", "bat", "cherry"};
std::map<char, std::vector<std::string>> groups;
for (const auto& w : words) {
    groups[w[0]].push_back(w);
}
```

```python
# Python
from collections import defaultdict

words = ["apple", "ant", "banana", "bat", "cherry"]
groups = defaultdict(list)
for w in words:
    groups[w[0]].append(w)

# 或用 itertools.groupby（但需要先排序）
from itertools import groupby
words_sorted = sorted(words, key=lambda w: w[0])
groups2 = {k: list(v) for k, v in groupby(words_sorted, key=lambda w: w[0])}
```

## 34.5 "重容器"意味着什么？

Python的"重容器"体现在：容器本身承担了大量"算法"的工作。

### 容器自身功能的丰富程度对比

```python
# Python list 自带 11 个方法（不含魔法方法）
# append, clear, copy, count, extend, index, insert, pop, remove, reverse, sort

# Python dict 自带 11 个方法
# clear, copy, get, items, keys, pop, popitem, setdefault, update, values, fromkeys

# Python set 自带 17 个方法
# add, clear, copy, difference, difference_update, discard, intersection,
# intersection_update, isdisjoint, issubset, issuperset, pop, remove,
# symmetric_difference, symmetric_difference_update, union, update
```

```cpp
// C++ vector 自带的方法主要是"存储管理"类的
// push_back, pop_back, insert, erase, clear, resize, reserve,
// size, empty, front, back, at, [], begin, end
// 没有 sort、find、count、remove——这些都在 <algorithm> 里

// C++ set 自带的方法也是"存储管理"类的
// insert, erase, find, count, contains(C++20), clear, size
// 没有 union、intersection、difference——这些都在 <algorithm> 里
```

### 设计哲学的根源

**C++的理由**：如果给vector加sort方法，就要给deque也加sort方法，给array也加……这违反了DRY（Don't Repeat Yourself）原则。算法独立出来，一次实现，处处可用。

**Python的理由**：开发者不需要记住"sort在哪个模块里"——它就在列表上。最常用的操作应该最容易找到。而且Python只有一个list，不需要担心M个容器的重复实现问题。

## 34.6 "轻算法"不等于"没算法"

Python并非没有独立的算法模块，只是使用场景较少：

```python
# itertools——迭代器工具（最接近STL算法的模块）
import itertools

# 排列组合
list(itertools.permutations([1,2,3]))        # 全排列
list(itertools.combinations([1,2,3,4], 2))   # 组合
list(itertools.product([1,2], ['a','b']))     # 笛卡尔积

# 分组
for key, group in itertools.groupby("AAABBBCC"):
    print(key, list(group))   # A [A,A,A], B [B,B,B], C [C,C]

# 链式迭代
list(itertools.chain([1,2], [3,4], [5,6]))   # [1,2,3,4,5,6]

# 累积
list(itertools.accumulate([1,2,3,4,5]))       # [1,3,6,10,15]

# functools——函数工具
import functools
functools.reduce(lambda a, b: a * b, [1,2,3,4,5])  # 120（连乘）

# operator——运算符函数（类似C++ <functional>）
import operator
functools.reduce(operator.mul, [1,2,3,4,5])   # 120
sorted(students, key=operator.itemgetter(1))  # 按第2个字段排序
```

> **但关键区别**：C++中你**必须**使用`<algorithm>`才能完成大部分数据处理；Python中你**大部分时候不需要**`itertools`，列表推导式和内置函数已经够用。

## 34.7 本章总结：两种风格的思维导图

```
C++ STL 思维模式:
  "我有数据在容器里 → 我需要什么算法 → 传入迭代器范围"

  vector<int> v = {...};
  auto it = std::find(v.begin(), v.end(), target);
  std::sort(v.begin(), v.end());
  int sum = std::accumulate(v.begin(), v.end(), 0);

Python 思维模式:
  "我有数据在容器里 → 容器/内置函数/推导式能直接做吗？"

  v = [...]
  target in v
  v.sort()
  sum(v)
```

| 维度     | C++ STL            | Python                   |
| -------- | ------------------ | ------------------------ |
| 算法位置 | 独立于容器的函数   | 容器方法+内置函数+语法糖 |
| 算法组合 | 迭代器范围链接     | 推导式/生成器管道        |
| 代码量   | 较多（显式）       | 较少（隐式）             |
| 灵活性   | 极高（子范围操作） | 够用（全量操作为主）     |
| 学习曲线 | 需记住大量算法     | 记住容器方法即可上手     |

---

# 第35章 从STL思维迁移到Python思维：写法、抽象层次与性能取舍

## 35.1 迁移的核心原则

从C++ STL迁移到Python，最重要的不是学语法，而是**转变思维方式**。核心原则：

> **在Python中，用最自然、最可读的方式表达意图；只在真正需要时才考虑性能。**

## 35.2 思维转变一：从"循环"到"表达式"

### C++习惯：用for循环处理一切

```cpp
// C++：统计正数个数
std::vector<int> data = {-3, 1, -4, 1, 5, -9, 2, 6};
int count = 0;
for (int x : data) {
    if (x > 0) count++;
}
```

### Python新手常犯的错误：直译C++

```python
# ❌ Python新手写法（翻译C++）
data = [-3, 1, -4, 1, 5, -9, 2, 6]
count = 0
for x in data:
    if x > 0:
        count += 1
```

### Python正确姿势：用表达式

```python
# ✅ Python风格写法
data = [-3, 1, -4, 1, 5, -9, 2, 6]
count = sum(1 for x in data if x > 0)      # 方法1：生成器表达式
count = len([x for x in data if x > 0])    # 方法2：列表推导式+len
```

### 更多"循环→表达式"的迁移示例

```python
# ---- 示例1：创建映射字典 ----
# ❌ C++思维
result = {}
for item in items:
    result[item.name] = item.value

# ✅ Python思维
result = {item.name: item.value for item in items}

# ---- 示例2：字符串拼接 ----
# ❌ C++思维
s = ""
for word in words:
    s += word + " "

# ✅ Python思维
s = " ".join(words)

# ---- 示例3：展平二维列表 ----
# ❌ C++思维
flat = []
for row in matrix:
    for item in row:
        flat.append(item)

# ✅ Python思维
flat = [item for row in matrix for item in row]

# ---- 示例4：条件赋值 ----
# ❌ C++思维
if condition:
    value = a
else:
    value = b

# ✅ Python思维
value = a if condition else b
```

## 35.3 思维转变二：从"迭代器操作"到"容器变换"

C++习惯把数据处理看成"在迭代器范围上施加算法"；Python习惯把数据处理看成"容器整体变换"。

### 过滤后转换

```cpp
// C++：迭代器范围思维
std::vector<int> data = {1, 2, 3, 4, 5, 6, 7, 8};
std::vector<int> result;
for (auto it = data.begin(); it != data.end(); ++it) {
    if (*it % 2 == 0) {
        result.push_back(*it * *it);
    }
}
// 或用 copy_if + transform（更STL但更冗长）
```

```python
# Python：容器变换思维
data = [1, 2, 3, 4, 5, 6, 7, 8]
result = [x**2 for x in data if x % 2 == 0]  # [4, 16, 36, 64]
```

### 多步变换管道

```cpp
// C++：需要中间变量或复杂的迭代器适配
// 取绝对值 → 去重 → 排序 → 取前3
std::vector<int> data = {-3, 1, -4, 1, 5, -9, 2, 6};
std::vector<int> abs_data;
std::transform(data.begin(), data.end(), std::back_inserter(abs_data),
               [](int x) { return std::abs(x); });
std::sort(abs_data.begin(), abs_data.end());
abs_data.erase(std::unique(abs_data.begin(), abs_data.end()), abs_data.end());
abs_data.resize(std::min((size_t)3, abs_data.size()));
```

```python
# Python：链式调用 / 管道思维
data = [-3, 1, -4, 1, 5, -9, 2, 6]
result = sorted(set(abs(x) for x in data))[:3]  # [1, 2, 3]
```

## 35.4 思维转变三：从"关注底层实现"到"关注意图表达"

### C++：你需要知道底层才能写对代码

```cpp
// 要判断vector是否包含某元素，你得知道 find 返回迭代器
if (std::find(v.begin(), v.end(), target) != v.end()) { ... }

// 要删除满足条件的元素，你得知道 erase-remove 惯用法
v.erase(std::remove_if(v.begin(), v.end(), pred), v.end());

// 要统计map中值大于阈值的键数量，你得组合多个工具
int cnt = std::count_if(m.begin(), m.end(),
    [&](const auto& p) { return p.second > threshold; });
```

### Python：直接表达意图

```python
# 判断是否包含——自然语言级
if target in v: ...

# 过滤——直接表达"我要什么"
v = [x for x in v if not pred(x)]

# 统计——直接表达"数什么"
cnt = sum(1 for val in m.values() if val > threshold)
```

## 35.5 思维转变四：从"手动管理"到"利用语言特性"

### 交换两个变量

```cpp
// C++
int a = 1, b = 2;
std::swap(a, b);   // 或用临时变量
```

```python
# Python
a, b = 1, 2
a, b = b, a   # 一行搞定，无需临时变量
```

### 同时遍历索引和值

```cpp
// C++
for (size_t i = 0; i < v.size(); ++i) {
    std::cout << i << ": " << v[i] << "\n";
}
```

```python
# Python
for i, val in enumerate(v):
    print(f"{i}: {val}")
```

### 同时遍历两个容器

```cpp
// C++：手动同步索引
for (size_t i = 0; i < std::min(v1.size(), v2.size()); ++i) {
    process(v1[i], v2[i]);
}
```

```python
# Python
for a, b in zip(v1, v2):
    process(a, b)
```

### 字典的合并

```cpp
// C++：手动遍历插入
for (const auto& [k, v] : map2) {
    map1.insert_or_assign(k, v);
}
```

```python
# Python 3.9+
merged = map1 | map2       # 创建新字典

# Python 3.5+
merged = {**map1, **map2}  # 解包合并
```

## 35.6 性能取舍：Python的代价与应对

### 35.6.1 Python慢在哪里？

Python容器操作通常比C++慢10-100倍，原因：

1. **解释执行**：Python逐行解释，C++编译为机器码
2. **动态类型**：每次操作都要检查类型
3. **间接存储**：list存指针而非值，缓存不友好
4. **GIL**：全局解释器锁限制多线程并行

### 35.6.2 Python的应对策略

**策略1：选择正确的容器实现O(1)而非O(n)**

```python
# 陷阱：list的in操作是O(n)
big_list = list(range(1000000))
if 999999 in big_list: pass      # 扫描整个列表！

# 正解：用set获得O(1)查找
big_set = set(range(1000000))
if 999999 in big_set: pass       # 哈希查找，瞬间完成
```

**策略2：利用内置函数（C实现）而非手写循环**

```python
import time

data = list(range(1000000))

# ❌ 手写循环（纯Python字节码执行）
# 注：工程性能测试推荐使用 time.perf_counter()，比 time.time() 更精确且不受系统时钟调整干扰
start = time.perf_counter()
total = 0
for x in data:
    total += x
print(f"手写循环: {time.perf_counter() - start:.4f}s")

# ✅ 内置函数（C语言实现，底层直接操作）
start = time.perf_counter()
total = sum(data)
print(f"内置sum:  {time.perf_counter() - start:.4f}s")

# sum() 通常快5-10倍，因为它是用C写的
```

**策略3：大规模数值计算用NumPy**

```python
import numpy as np

# 纯Python
py_list = list(range(1000000))
py_result = [x**2 for x in py_list]

# NumPy（底层C实现，向量化运算）
np_array = np.arange(1000000)
np_result = np_array ** 2   # 快50-100倍
```

**策略4：真正的性能瓶颈用C扩展或Cython**

```python
# 如果算法的核心循环是性能瓶颈，可以：
# 1. 用 C 扩展模块（ctypes / cffi）
# 2. 用 Cython 编译 Python 代码为 C
# 3. 用 numba 的 @jit 装饰器即时编译
# 4. 用 PyPy 解释器（JIT编译的Python实现）
```

### 35.6.3 性能优先级金字塔

```
            ┌─────────────┐
            │ C扩展/Cython │ ← 最后的手段
            ├─────────────┤
            │  NumPy等库   │ ← 数值计算
            ├─────────────┤
            │ 内置函数/推导式│ ← 利用C实现
            ├─────────────┤
            │ 正确的容器选择 │ ← 算法复杂度
            ├─────────────┤
            │ 正确的算法设计 │ ← 最重要！
            └─────────────┘
```

> **核心法则**：在Python中，**选对算法和数据结构**永远是最重要的优化——一个O(n)的Python代码通常比O(n²)的C++代码更快。

## 35.7 常见迁移陷阱与正解速查表

| C++习惯                        | Python陷阱                          | Python正解                                      |
| ------------------------------ | ----------------------------------- | ----------------------------------------------- |
| `for(int i=0;i<n;i++)`         | `for i in range(n)` 然后用 `lst[i]` | 直接`for x in lst`或`for i,x in enumerate(lst)` |
| `v.push_back(x)` 批量          | 在循环里`.append()`                 | 用列表推导式一次性构建                          |
| `map[key]`检查存在             | 先`if key in d`再`d[key]`           | 用`d.get(key, default)`                         |
| `vec.size() == 0`              | `len(lst) == 0`                     | 直接`if not lst:`（空容器为falsy）              |
| 手动swap                       | 引入临时变量                        | `a, b = b, a`                                   |
| `std::sort + 自定义comparator` | 写完整的比较函数                    | 用`key=`参数                                    |
| 手动构建`map<K, vector<V>>`    | 手动`if key not in d: d[key]=[]`    | 用`defaultdict(list)`                           |
| 手动迭代器对                   | 手写索引控制                        | 用`zip`,`enumerate`,`itertools`                 |

## 35.8 本章总结：迁移口诀

1. **能用推导式就不用循环**——推导式更快、更Pythonic
2. **能用内置函数就不手写**——`sum`、`max`、`sorted`等都是C实现
3. **能用容器方法就不用外部工具**——`dict.get()` > 自己`try/except`
4. **先选对数据结构**——`set`的`in`比`list`快100倍不止
5. **写人能看懂的代码**——Python先追求可读性，性能问题用profile验证后再优化

> **一句话总结**：从STL迁移到Python，最大的思维转变是——**停止像编译器一样思考，开始像人一样思考**。Python代码读起来应该接近自然语言。

---

# 第九编 容器的工程实践与综合训练

# 第36章 面向工程实践的容器选择：性能、可读性与维护成本平衡

## 36.1 工程实践中的容器选择不是"做算法题"

在学习阶段，我们关心"用什么容器能解这道题"；在工程实践中，选择容器时要同时权衡**三个维度**：

```
                性能 (Performance)
                 /\
                /  \
               /    \
              /  选择  \
             /   容器    \
            /____________\
  可读性                    维护成本
(Readability)          (Maintainability)
```

> **工程箴言**：最好的容器选择不是"最快的"，而是"在可接受的性能下，最容易理解和维护的"。

## 36.2 性能维度：操作复杂度速查

### 36.2.1 四大核心容器的时间复杂度对照表

| 操作                | `list`     | `tuple`  | `dict`   | `set`    |
| ------------------- | ---------- | -------- | -------- | -------- |
| 索引访问 `x[i]`     | O(1)       | O(1)     | —        | —        |
| 键/值访问 `x[k]`    | —          | —        | O(1)均摊 | —        |
| 成员检测 `in`       | **O(n)**   | **O(n)** | **平均/O(1)** | **平均/O(1)** |
| 末尾追加 `append`   | O(1)均摊   | —        | —        | —        |
| 任意位置插入        | O(n)       | —        | —        | —        |
| 添加元素 `add`      | —          | —        | O(1)均摊 | O(1)均摊 |
| 按索引删除 `pop(i)` | O(n)       | —        | —        | —        |
| 末尾删除 `pop()`    | O(1)       | —        | —        | —        |
| 按键删除            | —          | —        | O(1)均摊 | O(1)均摊 |
| 排序 `sort`         | O(n log n)(Timsort稳定) | —        | —        | —        |
| 遍历                | O(n)       | O(n)     | O(n)     | O(n)     |
| 长度 `len`          | O(1)       | O(1)     | O(1)     | O(1)     |

### 36.2.2 最常见的性能陷阱

> **复杂度边界说明**：`dict`和`set`基于哈希表，因此成员检测、插入、删除通常讨论的是**平均复杂度或均摊复杂度**。在极端哈希冲突等退化情形下，最坏复杂度可能上升；教材中若简写成"O(1)",应默认理解为"平均情况下的 O(1)"。

**陷阱1：在list中做成员检测**

```python
# ❌ 严重性能问题——list的 in 是O(n)
whitelist = [
    "user_001", "user_002", "user_003",
    # ... 假设有10万个用户
]

def is_allowed(user_id):
    return user_id in whitelist  # 每次调用O(n)！

# 如果每秒调用1000次，10万×1000 = 1亿次比较/秒

# ✅ 正解——set的 in 是O(1)
whitelist = {
    "user_001", "user_002", "user_003",
    # ... 同样10万个用户
}

def is_allowed(user_id):
    return user_id in whitelist  # O(1)，立即返回
```

**陷阱2：频繁在list头部插入**

```python
# ❌ list头部插入是O(n)——每次都要移动所有元素
log_entries = []
for entry in stream:
    log_entries.insert(0, entry)  # 随着列表增长越来越慢

# ✅ 正解1——用deque，头部插入O(1)
from collections import deque
log_entries = deque()
for entry in stream:
    log_entries.appendleft(entry)  # O(1)

# ✅ 正解2——末尾追加后反转
log_entries = []
for entry in stream:
    log_entries.append(entry)
log_entries.reverse()              # 整体O(n)只执行一次
```

**陷阱3：用list存储需要频繁去重的数据**

```python
# ❌ 手动去重O(n²)
unique = []
for item in data:
    if item not in unique:    # O(n) × n次 = O(n²)
        unique.append(item)

# ✅ 如果不需要保序——直接用set
unique = set(data)             # O(n)

# ✅ 如果需要保序——dict.fromkeys
unique = list(dict.fromkeys(data))  # O(n)
```

### 36.2.3 容器性能选择决策树

```
需要存储数据
  ├── 需要键值映射？
  │     ├── 是 → dict
  │     │     ├── 需要默认值？ → defaultdict
  │     │     ├── 需要有序？(插入序) → dict(3.7+自动)
  │     │     └── 需要计数？ → Counter
  │     └── 否 → 继续
  ├── 需要唯一性？
  │     ├── 是 → set
  │     │     └── 需要不可变？ → frozenset
  │     └── 否 → 继续
  ├── 需要有序序列？
  │     ├── 数据不变？ → tuple
  │     ├── 需要两端操作？ → deque
  │     ├── 需要带优先级？ → heapq
  │     └── 通用序列 → list
  └── 其他特殊需求
        ├── 大规模数值 → numpy.ndarray
        └── 稀疏数据 → dict模拟
```

## 36.3 可读性维度：让代码表达意图

### 36.3.1 容器选择本身就是文档

```python
# 用 tuple 而非 list，表达"这组数据不应被修改"
DIRECTIONS = ("north", "south", "east", "west")  # 意图：常量
# vs
directions = ["north", "south", "east", "west"]  # 意图不明确

# 用 set 而非 list，表达"元素唯一、我关心成员检测"
VALID_EXTENSIONS = {".py", ".js", ".ts", ".go"}  # 意图：查找集
# vs
valid_extensions = [".py", ".js", ".ts", ".go"]  # 可能还关心顺序？

# 用 dict 而非两个list，表达"键值关联"
scores = {"Alice": 90, "Bob": 85}  # 意图：映射
# vs
names = ["Alice", "Bob"]            # 需要人工保持同步
grades = [90, 85]                    # 改一个忘改另一个就出bug
```

### 36.3.2 命名元组 vs 字典 vs 普通元组

```python
# 场景：存储一个学生的信息 (姓名, 年龄, 成绩)

# ❌ 普通tuple：不知道每个位置是什么
student = ("Alice", 20, 95)
print(student[2])  # 95 —— 2是什么意思？

# ⚠️ dict：有键名，但可被随意修改、容易写错键
student = {"name": "Alice", "age": 20, "score": 95}
student["scroe"] = 100  # 拼写错误！不会报错，只是多了一个键！

# ✅ namedtuple：有字段名 + 不可变 + 轻量
from collections import namedtuple
Student = namedtuple("Student", ["name", "age", "score"])
s = Student("Alice", 20, 95)
print(s.name, s.score)  # Alice 95——清晰！
# s.score = 100  → AttributeError（不可变，安全）

# ✅ dataclass（Python 3.7+）：最现代的方式
from dataclasses import dataclass

# 注：Python 3.10+ 可用 slots=True 取消常规实例 __dict__，
# 通常能减少内存占用，并可能带来一定属性访问收益；
# 但它并不等同于 tuple，灵活性与适用场景也不同
@dataclass(slots=True)
class Student:
    name: str
    age: int
    score: float

s = Student("Alice", 20, 95)
print(s.name, s.score)  # 清晰 + 可变 + 有类型提示 + 自带__repr__ + 极致内存优化
```

### 36.3.3 选对推导式类型

```python
data = [1, 2, 3, 4, 5, 6, 7, 8]

# 需要列表（有序、可索引） → 列表推导式
squares = [x**2 for x in data if x > 3]

# 需要集合（去重） → 集合推导式
unique_squares = {x**2 for x in data}

# 需要字典（映射） → 字典推导式
square_map = {x: x**2 for x in data}

# 只需要遍历一次（节省内存） → 生成器表达式
total = sum(x**2 for x in data)  # 不创建中间list
```

## 36.4 维护成本维度：半年后还能看懂吗？

### 36.4.1 避免嵌套过深的容器

```python
# ❌ 维护噩梦：三层嵌套字典
data = {
    "2024": {
        "Q1": {
            "sales": [100, 200, 300],
            "costs": [50, 100, 150],
        },
        "Q2": {
            "sales": [400, 500, 600],
            "costs": [200, 250, 300],
        }
    }
}
# 访问代码：data["2024"]["Q1"]["sales"][0] —— 脆弱且难读

# ✅ 用扁平结构 + 复合键
data = {
    ("2024", "Q1", "sales"): [100, 200, 300],
    ("2024", "Q1", "costs"): [50, 100, 150],
    ("2024", "Q2", "sales"): [400, 500, 600],
    ("2024", "Q2", "costs"): [200, 250, 300],
}

# ✅ 或用dataclass建模
@dataclass
class QuarterData:
    sales: list[int]
    costs: list[int]

@dataclass
class YearData:
    quarters: dict[str, QuarterData]
```

### 36.4.2 给复杂容器加类型提示

```python
from typing import Dict, List, Tuple, Optional

# ❌ 没有类型提示，谁也不知道这是什么
def process(data):
    result = {}
    for k, v in data.items():
        result[k] = sum(v) / len(v)
    return result

# ✅ 有类型提示，一目了然
def process(data: Dict[str, List[float]]) -> Dict[str, float]:
    """计算每个类别的平均值"""
    return {k: sum(v) / len(v) for k, v in data.items()}
```

## 36.5 实际场景的最佳容器选择

| 场景        | 推荐容器                             | 理由               |
| ----------- | ------------------------------------ | ------------------ |
| 配置项      | `dict`或`dataclass`                  | 键值映射，结构清晰 |
| 常量列表    | `tuple`                              | 不可变，语义明确   |
| 去重ID集合  | `set`                                | O(1)查找，自动去重 |
| 日志队列    | `deque(maxlen=N)`                    | 自动丢弃旧数据     |
| 频率统计    | `Counter`                            | 专为计数设计       |
| 缓存/LRU    | `OrderedDict`或`functools.lru_cache` | 有序+可淘汰        |
| 分组聚合    | `defaultdict(list)`                  | 自动创建空列表     |
| 有序唯一值  | `dict.fromkeys()`结果                | 保序+去重          |
| 优先队列    | `heapq`                              | 堆操作O(log n)     |
| 双端操作    | `deque`                              | 两端O(1)           |
| 大规模数值  | `numpy.ndarray`                      | 向量化运算         |
| API响应数据 | `dataclass`或`TypedDict`             | 有结构、有类型     |

## 36.6 本章总结

> **容器选择三步法**：
> 1. 先选对**数据结构**（性能——这决定了算法复杂度）
> 2. 再选对**容器类型**（可读性——让代码自文档化）
> 3. 最后考虑**封装层次**（维护性——用dataclass/namedtuple替代裸dict/tuple）

---

# 第37章 综合案例：数据清洗、词频统计、分组聚合与结构转换

## 37.1 案例一：数据清洗

### 场景描述

从CSV文件读取的用户数据有许多质量问题：空值、重复、格式不一致。我们需要清洗为规范格式。

```python
# 原始数据（模拟从CSV读取的结果）
raw_data = [
    {"name": "Alice",   "email": "Alice@Example.COM",   "age": "25"},
    {"name": "bob",     "email": "BOB@example.com",     "age": "30"},
    {"name": " Charlie","email": "charlie@example.com",  "age": ""},
    {"name": "Alice",   "email": "alice@example.com",    "age": "25"},   # 重复
    {"name": "",        "email": "",                      "age": "22"},   # 无效
    {"name": "David",   "email": "david@example.com",    "age": "abc"},  # 非法年龄
    {"name": "Eve",     "email": "eve@example.com",      "age": "28"},
    {"name": "  bob ",  "email": "bob@example.com",      "age": "30"},   # 重复(带空格)
]
```

### 清洗流程

```python
def clean_record(record: dict) -> dict | None:
    """清洗单条记录，返回清洗后的记录或None（表示无效）"""
    # 1. 去除字段两端空白
    name = record.get("name", "").strip()
    email = record.get("email", "").strip().lower()

    # 2. 验证必填字段
    if not name or not email:
        return None   # 无效记录

    # 3. 统一名字格式（首字母大写）
    name = name.title()

    # 4. 解析年龄（安全转换）
    raw_age = record.get("age", "").strip()
    try:
        age = int(raw_age) if raw_age else None
    except ValueError:
        age = None  # 非法值设为None

    return {"name": name, "email": email, "age": age}


def clean_dataset(raw_data: list[dict]) -> list[dict]:
    """清洗整个数据集"""
    # 步骤1：逐条清洗，过滤掉无效记录
    cleaned = [
        record for raw in raw_data
        if (record := clean_record(raw)) is not None  # 海象运算符(3.8+)
    ]

    # 步骤2：按email去重（保留首次出现的）
    seen_emails = set()
    unique = []
    for record in cleaned:
        if record["email"] not in seen_emails:
            seen_emails.add(record["email"])
            unique.append(record)

    return unique


# 执行清洗
result = clean_dataset(raw_data)
for r in result:
    print(r)

# 输出：
# {'name': 'Alice',   'email': 'alice@example.com',   'age': 25}
# {'name': 'Bob',     'email': 'bob@example.com',     'age': 30}
# {'name': 'Charlie', 'email': 'charlie@example.com', 'age': None}
# {'name': 'David',   'email': 'david@example.com',   'age': None}
# {'name': 'Eve',     'email': 'eve@example.com',     'age': 28}
```

### 容器知识点回顾

- **dict**：存储每条记录的字段
- **set**：`seen_emails`用于O(1)去重检测
- **list**：推导式过滤无效记录
- **海象运算符`:=`**：在推导式中同时赋值和过滤

## 37.2 案例二：词频统计

### 场景描述

统计一段英文文本中每个单词的出现频率，输出Top 10。

```python
text = """
Python is a programming language. Python is widely used in web development,
data science, artificial intelligence, and automation. Python is known for
its simple syntax. Many developers love Python because Python makes coding
fun and productive. Learning Python is a great investment for your career.
"""

from collections import Counter
import re

def word_frequency(text: str, top_n: int = 10) -> list[tuple[str, int]]:
    """统计词频并返回Top N"""
    # 步骤1：文本预处理
    # 转小写 → 提取单词（去除标点）
    words = re.findall(r'[a-z]+', text.lower())

    # 步骤2：用Counter统计
    counter = Counter(words)

    # 步骤3：返回最常见的Top N
    return counter.most_common(top_n)


# 执行
for word, count in word_frequency(text):
    print(f"  {word:<15} {count}")
```

### 如果不用Counter，手动实现

```python
def word_frequency_manual(text: str, top_n: int = 10):
    words = re.findall(r'[a-z]+', text.lower())

    # 方法1：defaultdict
    from collections import defaultdict
    freq = defaultdict(int)
    for word in words:
        freq[word] += 1

    # 方法2：普通dict + get
    freq2 = {}
    for word in words:
        freq2[word] = freq2.get(word, 0) + 1

    # 排序：按频率降序
    return sorted(freq.items(), key=lambda x: x[1], reverse=True)[:top_n]
```

### 进阶：统计二元词组（bigram）频率

```python
def bigram_frequency(text: str, top_n: int = 10):
    words = re.findall(r'[a-z]+', text.lower())

    # 用 zip 生成相邻词对——这是Python的优雅之处
    bigrams = zip(words, words[1:])  # (w0,w1), (w1,w2), (w2,w3), ...

    # 用 tuple 作为 Counter 的键（tuple可哈希）
    counter = Counter(bigrams)
    return counter.most_common(top_n)

for bigram, count in bigram_frequency(text):
    print(f"  {str(bigram):<30} {count}")
```

### 容器知识点回顾

- **Counter**：专用计数容器，内置`most_common()`
- **defaultdict(int)**：手动实现Counter的基础
- **tuple**：作为Counter/dict的键（存储bigram）
- **zip**：优雅地生成相邻词对
- **sorted + key**：自定义排序

## 37.3 案例三：分组聚合

### 场景描述

有一组订单数据，需要按不同维度分组后进行聚合计算。

```python
from collections import defaultdict
from dataclasses import dataclass
from typing import Dict, List

@dataclass
class Order:
    order_id: str
    customer: str
    product: str
    category: str
    amount: float
    quantity: int

orders = [
    Order("001", "Alice",   "Laptop",    "Electronics", 999.99,  1),
    Order("002", "Bob",     "Phone",     "Electronics", 699.99,  2),
    Order("003", "Alice",   "Book",      "Education",   29.99,   3),
    Order("004", "Charlie", "Tablet",    "Electronics", 499.99,  1),
    Order("005", "Bob",     "Course",    "Education",   199.99,  1),
    Order("006", "Alice",   "Keyboard",  "Electronics", 79.99,   2),
    Order("007", "Charlie", "Book",      "Education",   39.99,   5),
    Order("008", "Bob",     "Mouse",     "Electronics", 29.99,   3),
    Order("009", "Alice",   "Course",    "Education",   149.99,  1),
    Order("010", "Charlie", "Monitor",   "Electronics", 349.99,  1),
]
```

### 操作1：按客户分组，计算总消费

```python
def total_by_customer(orders: list[Order]) -> dict[str, float]:
    """按客户分组，计算每人的总消费金额"""
    totals = defaultdict(float)
    for order in orders:
        totals[order.customer] += order.amount * order.quantity
    return dict(sorted(totals.items(), key=lambda x: x[1], reverse=True))

result = total_by_customer(orders)
for customer, total in result.items():
    print(f"  {customer}: ¥{total:.2f}")
# Alice:   ¥1,429.94
# Charlie: ¥1,049.93
# Bob:     ¥1,589.91
```

### 操作2：按类别分组，计算每类的订单列表

```python
def group_by_category(orders: list[Order]) -> dict[str, list[Order]]:
    """按品类分组"""
    groups = defaultdict(list)
    for order in orders:
        groups[order.category].append(order)
    return dict(groups)

for cat, cat_orders in group_by_category(orders).items():
    print(f"\n  [{cat}] ({len(cat_orders)}笔订单)")
    for o in cat_orders:
        print(f"    {o.order_id}: {o.product} - ¥{o.amount}")
```

### 操作3：多维聚合——客户×类别的消费矩阵

```python
def spending_matrix(orders: list[Order]) -> dict[str, dict[str, float]]:
    """生成 客户×类别 的消费矩阵"""
    matrix = defaultdict(lambda: defaultdict(float))
    for order in orders:
        matrix[order.customer][order.category] += order.amount * order.quantity
    return {k: dict(v) for k, v in matrix.items()}

matrix = spending_matrix(orders)

# 漂亮打印
categories = sorted({o.category for o in orders})
header = f"{'客户':<10}" + "".join(f"{c:<15}" for c in categories)
print(header)
print("-" * len(header))
for customer, spending in sorted(matrix.items()):
    row = f"{customer:<10}"
    for cat in categories:
        amount = spending.get(cat, 0)
        row += f"¥{amount:<14.2f}"
    print(row)
```

### 操作4：用itertools.groupby实现分组

```python
from itertools import groupby

def group_by_category_v2(orders: list[Order]):
    """用 groupby 分组（注意：必须先排序！）"""
    # groupby 要求数据按分组键排序
    sorted_orders = sorted(orders, key=lambda o: o.category)

    groups = {}
    for key, group_iter in groupby(sorted_orders, key=lambda o: o.category):
        groups[key] = list(group_iter)
    return groups
```

> **⚠️ groupby的陷阱**：`itertools.groupby`只对**相邻的相同元素**分组，数据必须先按分组键排序。如果忘了排序，结果会出错且不报任何异常。在大多数场景下，`defaultdict(list)`更安全。

### 容器知识点回顾

- **defaultdict(float/list/lambda)**：三种典型用法
- **嵌套defaultdict**：多维聚合
- **set推导式**：`{o.category for o in orders}`提取唯一值
- **sorted + key**：自定义排序
- **dataclass**：结构化数据的最佳实践

## 37.4 案例四：结构转换

### 场景：将嵌套数据在不同格式间转换

```python
# 原始格式：列表套字典（行式存储，类似数据库行）
students_rows = [
    {"name": "Alice",   "math": 90, "english": 85, "science": 92},
    {"name": "Bob",     "math": 78, "english": 88, "science": 75},
    {"name": "Carol",   "math": 95, "english": 70, "science": 98},
    {"name": "David",   "math": 60, "english": 92, "science": 65},
]
```

### 转换1：行式 → 列式（转置）

```python
def rows_to_columns(rows: list[dict]) -> dict[str, list]:
    """行式存储 → 列式存储"""
    if not rows:
        return {}
    columns = {key: [] for key in rows[0]}
    for row in rows:
        for key, value in row.items():
            columns[key].append(value)
    return columns

columns = rows_to_columns(students_rows)
# {
#   "name":    ["Alice", "Bob", "Carol", "David"],
#   "math":    [90, 78, 95, 60],
#   "english": [85, 88, 70, 92],
#   "science": [92, 75, 98, 65],
# }

# 列式存储的好处：方便按学科做统计
for subject in ["math", "english", "science"]:
    scores = columns[subject]
    avg = sum(scores) / len(scores)
    print(f"  {subject}: 平均={avg:.1f}, 最高={max(scores)}, 最低={min(scores)}")
```

### 转换2：列式 → 行式（逆转置）

```python
def columns_to_rows(columns: dict[str, list]) -> list[dict]:
    """列式存储 → 行式存储"""
    keys = list(columns.keys())
    n = len(next(iter(columns.values())))
    return [{key: columns[key][i] for key in keys} for i in range(n)]
```

### 转换3：列表 → 字典索引（构建查找表）

```python
# 把列表转为以某个字段为键的字典——构建"索引"
# 这是工程中极其常见的操作

students_by_name = {s["name"]: s for s in students_rows}
# 查找特定学生变成了O(1)
alice = students_by_name["Alice"]
print(alice)  # {'name': 'Alice', 'math': 90, 'english': 85, 'science': 92}
```

### 转换4：扁平化 ↔ 嵌套化

```python
# 嵌套 → 扁平
nested = {
    "user": {
        "name": "Alice",
        "address": {
            "city": "Beijing",
            "zip": "100000"
        }
    },
    "score": 95
}

def flatten_dict(d: dict, prefix: str = "", sep: str = ".") -> dict:
    """将嵌套字典扁平化"""
    result = {}
    for key, value in d.items():
        new_key = f"{prefix}{sep}{key}" if prefix else key
        if isinstance(value, dict):
            result.update(flatten_dict(value, new_key, sep))
        else:
            result[new_key] = value
    return result

flat = flatten_dict(nested)
# {'user.name': 'Alice', 'user.address.city': 'Beijing',
#  'user.address.zip': '100000', 'score': 95}


# 扁平 → 嵌套
def unflatten_dict(d: dict, sep: str = ".") -> dict:
    """将扁平字典还原为嵌套结构"""
    result = {}
    for key, value in d.items():
        parts = key.split(sep)
        current = result
        for part in parts[:-1]:
            current = current.setdefault(part, {})
        current[parts[-1]] = value
    return result

restored = unflatten_dict(flat)
# 完美还原为原始嵌套结构
```

### 容器知识点回顾

- **字典推导式**：结构转换的核心工具
- **setdefault**：安全地构建嵌套字典
- **isinstance**：类型检测（递归展平时判断是否是dict）
- **dict.fromkeys / 推导式**：构建索引/查找表

---

# 第38章 容器常见误区与高频问题：从会用语法到会用思想

## 38.1 误区一：可变默认参数

**这是Python最经典的陷阱之一**，不了解它的人迟早会被坑。

```python
# ❌ 致命错误：用可变对象作为函数默认参数
def add_item(item, lst=[]):
    lst.append(item)
    return lst

print(add_item("a"))   # ['a']        ← 看起来正常
print(add_item("b"))   # ['a', 'b']   ← 等等……为什么有'a'？！
print(add_item("c"))   # ['a', 'b', 'c']  ← 越来越离谱！
```

**原因**：默认参数只在函数**定义时**求值一次，所有调用共享同一个列表对象。

```python
# ✅ 正确写法：用 None 作为哨兵值
def add_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst

print(add_item("a"))   # ['a']
print(add_item("b"))   # ['b']   ← 每次都是新列表
```

> **规则**：永远不要用`list`、`dict`、`set`等可变对象作为函数的默认参数。

## 38.2 误区二：浅拷贝 vs 深拷贝

```python
# 浅拷贝：只复制最外层，内层仍然共享引用
original = [[1, 2, 3], [4, 5, 6]]
shallow = original.copy()      # 或 list(original) 或 original[:]

shallow[0][0] = 999
print(original[0][0])  # 999！原始数据也被修改了！

# 深拷贝：递归复制所有层
import copy
original = [[1, 2, 3], [4, 5, 6]]
deep = copy.deepcopy(original)

deep[0][0] = 999
print(original[0][0])  # 1——原始数据不受影响
```

**判断规则**：
- 容器只有一层（元素全是不可变的）→ 浅拷贝就够了
- 容器有嵌套（元素包含list/dict/set等可变对象）→ 必须用深拷贝

```python
# 浅拷贝够用的情况
simple = [1, 2, 3, "hello"]    # 元素全是不可变的
safe_copy = simple.copy()      # 浅拷贝足矣

# 必须深拷贝的情况
nested = {"a": [1, 2], "b": {"x": 10}}
safe_copy = copy.deepcopy(nested)
```

## 38.3 误区三：在遍历时修改容器

```python
# ❌ 遍历 list 时删除元素——跳过元素
numbers = [1, 2, 3, 4, 5, 6]
for x in numbers:
    if x % 2 == 0:
        numbers.remove(x)
print(numbers)  # [1, 3, 5, 6]  ← 6没被删除！

# 原因：删除元素后索引偏移，3被跳过检查到了5，6被跳过了
```

```python
# ✅ 正解1：创建新列表（最推荐）
numbers = [1, 2, 3, 4, 5, 6]
numbers = [x for x in numbers if x % 2 != 0]

# ✅ 正解2：倒序遍历删除（如果必须原地修改）
numbers = [1, 2, 3, 4, 5, 6]
for i in range(len(numbers) - 1, -1, -1):
    if numbers[i] % 2 == 0:
        numbers.pop(i)

# ✅ 正解3：遍历拷贝，修改原件
numbers = [1, 2, 3, 4, 5, 6]
for x in numbers[:]:       # numbers[:] 是浅拷贝
    if x % 2 == 0:
        numbers.remove(x)
```

```python
# ❌ 遍历 dict 时修改——RuntimeError
d = {"a": 1, "b": 2, "c": 3}
for key in d:
    if d[key] < 2:
        del d[key]   # RuntimeError: dictionary changed size during iteration

# ✅ 正解：先收集要删除的键，再删除
keys_to_delete = [k for k, v in d.items() if v < 2]
for k in keys_to_delete:
    del d[k]

# ✅ 或直接构建新字典
d = {k: v for k, v in d.items() if v >= 2}
```

## 38.4 误区四：`==` vs `is`

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)    # True  —— 值相等
print(a is b)    # False —— 不是同一个对象！

# 常见陷阱：用 is 比较值
# ❌
if my_list is []:    # 几乎永远是 False（创建了新的空列表对象）
    pass

# ✅
if my_list == []:    # 比较值
    pass

# ✅✅ 最Pythonic的写法
if not my_list:      # 空容器为falsy
    pass
```

**唯一应该用`is`的场景**：和`None`比较。

```python
# ✅ 和 None 比较用 is
if x is None:
    pass
if x is not None:
    pass
```

## 38.5 误区五：字典/集合的键必须可哈希

```python
# ❌ list不可哈希，不能当dict的键或set的元素
# {[1,2]: "hello"}       # TypeError: unhashable type: 'list'
# {[1,2], [3,4]}         # TypeError: unhashable type: 'list'

# ✅ 转成 tuple（不可变，可哈希）
{(1,2): "hello"}         # 合法
{(1,2), (3,4)}           # 合法

# ⚠️ 但tuple里包含list也不行！
# {([1,2], 3): "hello"}  # TypeError——tuple里有list，整体不可哈希

# 可哈希的类型：int, float, str, bool, tuple(元素可哈希), frozenset, None
# 不可哈希的类型：list, dict, set, bytearray
```

## 38.6 误区六：`*` 重复创建的陷阱

```python
# ❌ 二维列表的经典陷阱
matrix = [[0] * 3] * 3
print(matrix)       # [[0, 0, 0], [0, 0, 0], [0, 0, 0]] ← 看起来正常

matrix[0][0] = 1
print(matrix)       # [[1, 0, 0], [1, 0, 0], [1, 0, 0]] ← 三行都变了！

# 原因：[[0]*3] * 3 创建了3个指向同一个内层列表的引用
# [0]*3 → 创建一个 [0,0,0]
# [上面的引用] * 3 → 三行都指向同一个 [0,0,0]

# ✅ 正确创建二维列表
matrix = [[0] * 3 for _ in range(3)]  # 每次循环创建新列表
matrix[0][0] = 1
print(matrix)       # [[1, 0, 0], [0, 0, 0], [0, 0, 0]] ← 正确！
```

## 38.7 误区七：混淆排序的两种方式

```python
data = [3, 1, 4, 1, 5]

# .sort() —— 原地排序，返回 None
result = data.sort()
print(result)   # None！不是排序后的列表！
print(data)     # [1, 1, 3, 4, 5]——data本身被修改了

# sorted() —— 返回新列表，原列表不变
data = [3, 1, 4, 1, 5]
result = sorted(data)
print(result)   # [1, 1, 3, 4, 5]——新列表
print(data)     # [3, 1, 4, 1, 5]——原列表不变
```

**选择规则**：
- 不再需要原始顺序 → `.sort()`（省内存）
- 需要保留原列表 → `sorted()`
- 对非list可迭代对象排序 → 只能用`sorted()`（tuple/set/dict没有`.sort()`方法）

## 38.8 误区八：`+=` 对不同容器的行为差异

```python
# list：+= 是原地扩展（等价于 extend）
a = [1, 2]
b = a           # b 和 a 指向同一个对象
a += [3, 4]     # 原地修改！
print(b)        # [1, 2, 3, 4] ← b 也变了！

# tuple：+= 是创建新对象（因为 tuple 不可变）
a = (1, 2)
b = a           # b 和 a 指向同一个对象
a += (3, 4)     # 创建新 tuple！a 指向新对象
print(b)        # (1, 2) ← b 没变，仍指向旧对象
print(a)        # (1, 2, 3, 4)
```

## 38.9 高频面试/实战问题速答

| 问题                          | 答案                                  |
| ----------------------------- | ------------------------------------- |
| list和tuple的核心区别？       | 可变性。list可变，tuple不可变         |
| dict的键有什么要求？          | 必须是可哈希的（不可变类型）          |
| 怎么判断容器是否为空？        | `if not container:`（不要`len()==0`） |
| list的in操作复杂度？          | O(n)。需要O(1)用set                   |
| dict是有序的吗？              | 3.7+保证插入顺序                      |
| 怎样安全地访问dict？          | `d.get(key, default)`                 |
| 如何合并两个dict？            | `{**d1, **d2}`或`d1 \| d2`(3.9+)      |
| 列表推导式和map哪个更Python？ | 列表推导式更推荐                      |
| 什么时候用tuple不用list？     | 数据是常量/不应被修改/需要做dict键    |
| defaultdict和普通dict区别？   | 访问不存在的键时自动创建默认值        |

---

# 第39章 Python容器体系总结：从基础容器到高级工具的知识闭环

## 39.1 知识全景图

```
                    Python 容器体系全景图
    ┌───────────────────────────────────────────────┐
    │              内置基础容器（4种）                  │
    │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────────┐  │
    │  │ list │  │tuple │  │ dict │  │   set    │  │
    │  │可变序列│  │不可变 │  │键值映射│  │唯一值集合│  │
    │  │有序   │  │序列   │  │有序   │  │无序     │  │
    │  └──────┘  └──────┘  └──────┘  └──────────┘  │
    ├───────────────────────────────────────────────┤
    │            内置辅助类型                         │
    │  str(不可变序列)  bytes/bytearray  frozenset    │
    │  range(惰性序列)  memoryview                    │
    ├───────────────────────────────────────────────┤
    │          collections 高级容器                   │
    │  deque    defaultdict    Counter    OrderedDict│
    │  namedtuple    ChainMap                        │
    ├───────────────────────────────────────────────┤
    │          功能模块                               │
    │  heapq(堆)   bisect(二分)   array(紧凑数组)     │
    │  itertools(迭代器工具)   functools(函数工具)     │
    ├───────────────────────────────────────────────┤
    │          第三方生态                             │
    │  numpy.ndarray    pandas.DataFrame/Series      │
    │  sortedcontainers.SortedList/SortedDict        │
    └───────────────────────────────────────────────┘
```

## 39.2 四大基础容器的本质特征回顾

| 容器    | 可变 | 有序 | 唯一 | 可哈希 | 核心场景                 |
| ------- | ---- | ---- | ---- | ------ | ------------------------ |
| `list`  | ✅    | ✅    | ❌    | ❌      | 通用有序集合             |
| `tuple` | ❌    | ✅    | ❌    | ✅*     | 不可变记录、dict键       |
| `dict`  | ✅    | ✅†   | ✅键  | ❌      | 键值映射、查找表         |
| `set`   | ✅    | ❌    | ✅    | ❌      | 去重、成员检测、集合运算 |

> *tuple可哈希的前提是所有元素都可哈希。†dict从3.7起保证插入顺序。

## 39.3 collections模块：高级容器速查

### deque——双端队列

```python
from collections import deque

# deque擅长两端操作；尤其是左端 appendleft / popleft 为 O(1)
# list 的尾端 append / pop 也很高效，但左端操作代价高
dq = deque([1, 2, 3])
dq.appendleft(0)      # [0, 1, 2, 3]
dq.pop()               # [0, 1, 2]
dq.popleft()           # [1, 2]

# 定长队列——自动丢弃旧数据
recent = deque(maxlen=3)
for i in range(5):
    recent.append(i)
print(recent)           # deque([2, 3, 4])——只保留最近3个

# 旋转
dq = deque([1, 2, 3, 4, 5])
dq.rotate(2)            # [4, 5, 1, 2, 3]——右旋2位
dq.rotate(-2)           # [1, 2, 3, 4, 5]——左旋2位
```

### defaultdict——默认值字典

```python
from collections import defaultdict

# 分组：defaultdict(list)
groups = defaultdict(list)
for name, dept in [("Alice","IT"), ("Bob","HR"), ("Carol","IT")]:
    groups[dept].append(name)
# {'IT': ['Alice', 'Carol'], 'HR': ['Bob']}

# 计数：defaultdict(int)
counter = defaultdict(int)
for char in "hello world":
    counter[char] += 1

# 集合收集：defaultdict(set)
tags = defaultdict(set)
for item, tag in [("p1","A"), ("p1","B"), ("p2","A")]:
    tags[item].add(tag)
# {'p1': {'A', 'B'}, 'p2': {'A'}}
```

### Counter——计数器

```python
from collections import Counter

# 基本计数
c = Counter("abracadabra")
# Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

c.most_common(3)    # [('a',5), ('b',2), ('r',2)]

# 计数器运算
c1 = Counter(a=3, b=1)
c2 = Counter(a=1, b=2)
c1 + c2              # Counter({'a': 4, 'b': 3})
c1 - c2              # Counter({'a': 2})——只保留正值
c1 & c2              # Counter({'a': 1, 'b': 1})——交集(取min)
c1 | c2              # Counter({'a': 3, 'b': 2})——并集(取max)
```

### namedtuple——命名元组

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)
print(p.x, p.y)      # 3 4——按名称访问
print(p[0], p[1])     # 3 4——保留索引访问
x, y = p              # 支持解包

# 不可变 → 可以放入set或作为dict键
points = {Point(0,0), Point(1,1), Point(0,0)}  # 自动去重
```

## 39.4 功能模块速查

### heapq——堆/优先队列

```python
import heapq

data = [5, 3, 1, 4, 2]
heapq.heapify(data)           # 原地建堆 O(n)
heapq.heappush(data, 0)       # 插入 O(log n)
smallest = heapq.heappop(data)# 弹出最小值 O(log n)
top3 = heapq.nsmallest(3, data)  # 最小的3个
bot3 = heapq.nlargest(3, data)   # 最大的3个
```

### bisect——二分查找与有序插入

```python
import bisect

sorted_list = [10, 20, 30, 40, 50]
pos = bisect.bisect_left(sorted_list, 25)   # 2（应插入的位置）
bisect.insort(sorted_list, 25)               # 插入并保持有序
# [10, 20, 25, 30, 40, 50]
```

### itertools——迭代器工具精选

```python
import itertools

# 排列和组合
list(itertools.permutations([1,2,3], 2))      # 排列 P(3,2)
list(itertools.combinations([1,2,3,4], 2))    # 组合 C(4,2)
list(itertools.product("AB", "12"))            # 笛卡尔积

# 链式
list(itertools.chain([1,2], [3,4], [5,6]))    # [1,2,3,4,5,6]

# 累积
list(itertools.accumulate([1,2,3,4,5]))        # [1,3,6,10,15]

# 分组（注意：需要先排序！）
data = sorted("AABBBCC")
for k, g in itertools.groupby(data):
    print(k, list(g))

# 无限迭代器
itertools.count(10, 2)     # 10, 12, 14, 16, ...
itertools.cycle("ABC")     # A, B, C, A, B, C, ...
itertools.repeat(42, 5)    # 42, 42, 42, 42, 42
```

## 39.5 容器选择最终决策流程

当你面对一个新的数据存储需求时，按以下顺序思考：

```
1. 数据是键值对吗？
   ├── 是 → dict（或 defaultdict / Counter）
   └── 否 → 继续

2. 需要唯一性吗？
   ├── 是 → set（或 frozenset）
   └── 否 → 继续

3. 数据需要修改吗？
   ├── 不需要 → tuple（或 namedtuple）
   └── 需要 → 继续

4. 需要两端操作吗？
   ├── 是 → deque
   └── 否 → list

5. 特殊需求？
   ├── 优先队列 → heapq
   ├── 大规模数值 → numpy.ndarray
   ├── 有序维护 + 查找 → bisect + list
   └── 表格数据 → pandas.DataFrame
```

## 39.6 Python容器设计的统一哲学

回顾整个容器体系，可以提炼出Python容器设计的几个统一原则：

### 原则1：协议优于继承

Python容器不依赖单一的继承树来使用，而是更强调**协议**（Protocol）：
- 迭代器协议：迭代器对象通常实现`__iter__`和`__next__`
- 可迭代对象协议：对象实现`__iter__`并返回迭代器，或提供兼容的旧式序列访问
- 序列、映射、成员检测等行为由一组约定方法共同体现，`for`、`in`、`len()`等语法会按协议协同工作

```python
# 迭代器对象实现 __iter__ 和 __next__ 后，可被 for 循环逐个取值
# 实现了 __len__ 的对象通常都能用 len()
# in 会优先尝试 __contains__，否则回退到迭代或旧式索引协议
# 这就是"鸭子类型"——不看类型看行为
```

### 原则2：内置优于外部

最常用的操作直接内置在语法中：
- `in` → 成员检测
- `for...in` → 遍历
- `[...]` → 推导式
- `[a:b]` → 切片
- `| & - ^` → 集合运算

### 原则3：简单场景用简单工具

```
简单场景 ──────────────────────────── 复杂场景
list/dict/set/tuple → collections → 第三方库 → 自定义类
```

## 39.7 全书核心知识点清单

| 编号 | 知识点           | 掌握标准                                 |
| ---- | ---------------- | ---------------------------------------- |
| 1    | list的CRUD操作   | 能熟练使用append/insert/pop/remove/切片  |
| 2    | 列表推导式       | 能写出带条件过滤和嵌套的推导式           |
| 3    | tuple的不可变性  | 理解为什么tuple能做dict键而list不能      |
| 4    | dict的核心操作   | 熟练使用get/setdefault/update/items      |
| 5    | set的集合运算    | 能用`\| & - ^`完成并交差运算             |
| 6    | 容器的时间复杂度 | 知道list的in是O(n)，set/dict的in是O(1)   |
| 7    | 浅拷贝 vs 深拷贝 | 知道何时用copy何时用deepcopy             |
| 8    | 可变默认参数陷阱 | 永远用None作为可变参数的默认值           |
| 9    | 推导式的四种形式 | 列表/字典/集合推导式 + 生成器表达式      |
| 10   | collections模块  | 会用deque/defaultdict/Counter/namedtuple |
| 11   | 容器选择策略     | 能根据场景选择最合适的容器类型           |
| 12   | 遍历时不修改     | 知道为什么不能在遍历时修改容器           |
| 13   | 结构转换         | 能在行式/列式/字典索引/扁平嵌套间转换    |
| 14   | 性能优化思路     | 选对容器 > 用内置函数 > 用C扩展库        |

## 39.8 结语

Python容器体系的设计体现了Python的核心哲学：**实用主义**。它没有追求学术上的"完美分离"（像C++ STL那样），而是把最常用的功能以最直觉的方式呈现给开发者。

学完这套容器体系，你应该能够：

1. **面对任何数据存储需求**，快速选出最合适的容器类型
2. **写出Pythonic的代码**，充分利用推导式、内置函数和语法糖
3. **避开常见陷阱**，不被可变默认参数、浅拷贝、遍历修改等问题困扰
4. **在C++和Python之间自如切换**，理解两种语言容器设计的本质差异
5. **面对工程场景**，在性能、可读性和维护成本之间找到平衡

> **最后的忠告**：容器是工具，不是目的。掌握容器的最终目的是**更高效地解决问题**。当你不再纠结"该用什么容器"而是自然而然地做出正确选择时，你就真正掌握了Python容器体系。

---

至此，**第八编（第32-35章）**和**第九编（第36-39章）**全部完成。整体内容涵盖了：

- ✅ Python与C++ STL的设计哲学比较
- ✅ 四大容器的精确对应关系与关键差异
- ✅ "重容器轻算法"vs"容器+算法分离"的深入分析
- ✅ STL思维→Python思维的迁移指南与性能取舍
- ✅ 工程实践中的容器选择方法论
- ✅ 四个完整综合案例（数据清洗、词频统计、分组聚合、结构转换）
- ✅ 八大常见误区与高频问题
- ✅ 容器体系的知识闭环总结
