# 第3章 容器通用操作基础：创建、访问、遍历、判断、转换与比较

---

## 3.1 引言

在第1、2章中，我们了解了 Python 四大容器的分类和核心概念。本章将聚焦于一个实用问题：**哪些操作是所有容器（或大多数容器）共通的？**

Python 的设计哲学之一是**"鸭子类型"（Duck Typing）**——"如果它走路像鸭子、叫声像鸭子，那它就是鸭子"。这意味着不同容器只要支持相同的接口，就可以用相同的方式来操作。

掌握这些通用操作后，你在学习每种具体容器时会事半功倍。

不过也要注意：**“通用”不等于“完全一致”**。例如：

- `list`、`tuple` 属于**序列（sequence）**，强调位置和顺序；
- `dict` 属于**映射（mapping）**，强调“键 -> 值”的对应关系；
- `set` 强调**成员资格与去重**，不提供索引。

所以本章的学习重点是：既要看到它们的共同点，也要分清它们在语义上的根本差异。

---

## 3.2 创建容器

### 3.2.1 字面量语法创建

每种容器都有自己的字面量（literal）语法，这是最常用的创建方式：

```python
# list —— 方括号
my_list = [1, 2, 3, 4, 5]
empty_list = []

# tuple —— 圆括号
my_tuple = (1, 2, 3, 4, 5)
empty_tuple = ()
single_tuple = (42,)      # ⚠️ 注意逗号！没有逗号就只是一个带括号的表达式

# dict —— 花括号 + 冒号
my_dict = {"name": "Alice", "age": 20}
empty_dict = {}

# set —— 花括号（无冒号）
my_set = {1, 2, 3, 4, 5}
# ⚠️ 空集合不能用 {}，那是空字典！
empty_set = set()          # 必须用 set() 构造器
```

> ⚠️ **两个重要陷阱**：
> 1. 单元素元组必须有逗号：`(42,)` 是元组，`(42)` 只是整数 42 加了括号。
> 2. `{}` 是空字典，不是空集合。创建空集合必须用 `set()`。

```python
# 验证陷阱
print(type((42,)))   # <class 'tuple'>
print(type((42)))    # <class 'int'>    ← 不是元组！

print(type({}))      # <class 'dict'>   ← 不是集合！
print(type(set()))   # <class 'set'>
```

### 3.2.2 构造器创建

每种容器都可以通过其同名构造器函数来创建，通常接受一个可迭代对象作为参数：

```python
# 从其他可迭代对象创建
list("hello")            # ['h', 'e', 'l', 'l', 'o']
tuple("hello")           # ('h', 'e', 'l', 'l', 'o')
set("hello")             # {'h', 'e', 'l', 'o'}  ← 去重了
dict([("a", 1), ("b", 2)])  # {'a': 1, 'b': 2}
dict(name="Alice", age=20)  # {'name': 'Alice', 'age': 20}

# 从 range 创建
list(range(5))           # [0, 1, 2, 3, 4]
tuple(range(1, 6))       # (1, 2, 3, 4, 5)
set(range(0, 10, 2))     # {0, 2, 4, 6, 8}

# 容器之间互转
list({1, 2, 3})          # [1, 2, 3]（顺序不保证）
set([1, 2, 2, 3])        # {1, 2, 3}
tuple([10, 20, 30])      # (10, 20, 30)
```

> 💡 **补充说明**：
> 1. `dict()` 的参数要求能解释成“键值对”，或者使用关键字参数。
> 2. `set` 的元素、`dict` 的键必须是**可哈希（hashable）**对象，因此列表不能直接作为集合元素或字典键。

```python
# { [1, 2], [3, 4] }      # ❌ TypeError: list 是可变对象，不可哈希
# { [1, 2]: "value" }     # ❌ TypeError

ok_set = {(1, 2), (3, 4)}     # 元组通常可哈希，可作为集合元素
ok_dict = {(1, 2): "point"}   # 元组也可作为字典键
```

### 3.2.3 推导式创建（Comprehension）

`list`、`dict`、`set` 都支持推导式，这是 Python 中非常优雅的创建方式：

```python
# 列表推导式
squares = [x ** 2 for x in range(1, 6)]
print(squares)   # [1, 4, 9, 16, 25]

# 集合推导式
even_squares = {x ** 2 for x in range(1, 6) if x % 2 == 0}
print(even_squares)   # {4, 16}

# 字典推导式
char_index = {ch: i for i, ch in enumerate("abcde")}
print(char_index)   # {'a': 0, 'b': 1, 'c': 2, 'd': 3, 'e': 4}

# ⚠️ 元组没有推导式！圆括号里的推导式是生成器表达式
gen = (x ** 2 for x in range(5))
print(type(gen))   # <class 'generator'>  ← 不是元组！
# 如果需要元组推导，可以：
t = tuple(x ** 2 for x in range(5))
print(t)   # (0, 1, 4, 9, 16)
```

### 3.2.4 创建方式总结

|   创建方式   |      list       |    tuple     |       dict       |       set       |
| :----------: | :-------------: | :----------: | :--------------: | :-------------: |
|    字面量    |    `[1, 2]`     |   `(1, 2)`   |    `{"a": 1}`    |    `{1, 2}`     |
| 空容器字面量 |      `[]`       |     `()`     |       `{}`       | ❌（用 `set()`） |
|    构造器    |   `list(...)`   | `tuple(...)` |   `dict(...)`    |   `set(...)`    |
|    推导式    | `[... for ...]` |      ❌       | `{k: v for ...}` | `{... for ...}` |

---

## 3.3 获取容器信息

### 3.3.1 len()——获取元素数量

`len()` 是所有容器都支持的操作，返回容器中元素的数量：

```python
print(len([1, 2, 3]))            # 3
print(len((1, 2, 3)))            # 3
print(len({"a": 1, "b": 2}))    # 2（键值对的数量）
print(len({1, 2, 3}))           # 3
print(len("hello"))             # 5（字符数量）
print(len([]))                  # 0（空容器）
```

### 3.3.2 成员测试：in 和 not in

`in` 运算符用于判断元素是否存在于容器中，是所有容器都支持的核心操作：

```python
# list —— 检查值是否在列表中
print(3 in [1, 2, 3, 4])          # True
print(5 in [1, 2, 3, 4])          # False

# tuple —— 同 list
print("b" in ("a", "b", "c"))     # True

# set —— 检查元素是否在集合中（最快！）
print(3 in {1, 2, 3, 4})          # True

# dict —— ⚠️ 检查的是"键"，不是"值"
d = {"name": "Alice", "age": 20}
print("name" in d)                # True   ← 检查键
print("Alice" in d)               # False  ← "Alice" 是值，不是键
print("Alice" in d.values())      # True   ← 这样才能检查值

# not in —— 取反
print(5 not in [1, 2, 3])         # True
```

> 💡 **性能提示**：`in` 操作在不同容器上的时间复杂度不同：
> - `set` 和 `dict`：O(1)——常数时间，极快（基于哈希表）
> - `list` 和 `tuple`：O(n)——线性时间，需要逐个比较

```python
# 性能差异演示（概念性，不必运行验证）
large_list = list(range(1_000_000))
large_set = set(range(1_000_000))

# 999_999 in large_list   # 慢——需要从头到尾扫描
# 999_999 in large_set    # 快——直接哈希定位
```

### 3.3.3 bool() 与空容器判断

在 Python 中，空容器被视为"假值"（Falsy），非空容器为"真值"（Truthy）：

```python
# 空容器 → False
print(bool([]))        # False
print(bool(()))        # False
print(bool({}))        # False
print(bool(set()))     # False

# 非空容器 → True
print(bool([1]))       # True
print(bool((0,)))      # True  ← 注意：包含一个 0 的元组仍然是 True
print(bool({"a": 1}))  # True

# 利用这一特性进行简洁判断
data = []
if not data:
    print("列表为空")      # 推荐写法
# 等价但不推荐：
if len(data) == 0:
    print("列表为空")      # 功能相同，但不够 Pythonic

# 实际应用
def process_items(items):
    if not items:
        print("没有要处理的内容")
        return
    for item in items:
        print(f"处理: {item}")
```

---

## 3.4 遍历容器

### 3.4.1 基本 for 循环遍历

所有容器都支持 `for` 循环遍历：

```python
# list
for item in [10, 20, 30]:
    print(item, end=" ")       # 10 20 30

print()

# tuple
for item in ("a", "b", "c"):
    print(item, end=" ")       # a b c

print()

# set（顺序不保证）
for item in {3, 1, 2}:
    print(item, end=" ")       # 顺序不确定

print()

# dict —— 默认遍历键
d = {"name": "Alice", "age": 20, "city": "Beijing"}
for key in d:
    print(key, end=" ")        # name age city

print()

# dict —— 遍历值
for value in d.values():
    print(value, end=" ")      # Alice 20 Beijing

print()

# dict —— 遍历键值对
for key, value in d.items():
    print(f"{key}={value}", end=" ")   # name=Alice age=20 city=Beijing
```

> 💡 **顺序说明**：
> - `list`、`tuple` 的遍历顺序就是其元素顺序。
> - `dict` 在 **Python 3.7 及之后的语言规范中保证保持插入顺序**，因此遍历通常按插入顺序进行。
> - `set` **不保证固定顺序**，不要依赖其打印或遍历次序。

### 3.4.2 带索引的遍历：enumerate()

当你需要同时获得元素及其位置时，使用 `enumerate()`：

```python
fruits = ["苹果", "香蕉", "橘子", "葡萄"]

# ❌ 不推荐的写法
for i in range(len(fruits)):
    print(f"{i}: {fruits[i]}")

# ✅ 推荐的写法
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# 输出:
# 0: 苹果
# 1: 香蕉
# 2: 橘子
# 3: 葡萄

# 指定起始编号
for i, fruit in enumerate(fruits, start=1):
    print(f"第{i}种: {fruit}")
# 输出:
# 第1种: 苹果
# 第2种: 香蕉
# 第3种: 橘子
# 第4种: 葡萄
```

### 3.4.3 并行遍历：zip()

当你需要同时遍历多个容器时，使用 `zip()`：

```python
names = ["Alice", "Bob", "Charlie"]
scores = [90, 85, 78]
grades = ["A", "B", "C+"]

for name, score, grade in zip(names, scores, grades):
    print(f"{name}: {score}分, 等级{grade}")
# 输出:
# Alice: 90分, 等级A
# Bob: 85分, 等级B
# Charlie: 78分, 等级C+

# zip 的长度取决于最短的那个
short = [1, 2]
long = [10, 20, 30, 40]
print(list(zip(short, long)))   # [(1, 10), (2, 20)] ← 多余的被截断
```

> ⚠️ `zip()` 返回的是一个**迭代器对象**，不是立即生成的列表。很多时候我们直接在 `for` 循环中使用它；如果要查看完整内容，可再套 `list(...)`。
>
> 在 Python 3.10+ 中，如果你希望长度不一致时直接报错，可以写：
>
> ```python
> # zip(names, scores, strict=True)
> ```
>
> 这样更适合对数据完整性要求高的场景。

### 3.4.4 遍历方式总结

|          遍历方式          |     适用场景     |               示例               |
| :------------------------: | :--------------: | :------------------------------: |
|    `for x in container`    |    只需要元素    |        `for n in [1,2,3]`        |
| `for i, x in enumerate(c)` |  需要元素和索引  |   `for i, n in enumerate(lst)`   |
| `for a, b in zip(c1, c2)`  | 并行遍历多个容器 | `for n, s in zip(names, scores)` |
|  `for k, v in d.items()`   |  遍历字典键值对  |  `for k, v in my_dict.items()`   |

---

## 3.5 容器之间的转换

### 3.5.1 转换规则总结

```python
# ========== 转为 list ==========
list((1, 2, 3))              # [1, 2, 3]        ← 从 tuple
list({3, 1, 2})              # [1, 2, 3]        ← 从 set（顺序不保证）
list({"a": 1, "b": 2})       # ['a', 'b']       ← 从 dict（只取键）
list("hello")                # ['h', 'e', 'l', 'l', 'o']
list(range(5))               # [0, 1, 2, 3, 4]

# ========== 转为 tuple ==========
tuple([1, 2, 3])             # (1, 2, 3)
tuple({3, 1, 2})             # (1, 2, 3)        ← 顺序不保证
tuple("abc")                 # ('a', 'b', 'c')

# ========== 转为 set ==========
set([1, 2, 2, 3])            # {1, 2, 3}        ← 自动去重
set((1, 2, 2, 3))            # {1, 2, 3}
set("hello")                 # {'h', 'e', 'l', 'o'}  ← 去重
set({"a": 1, "b": 2})        # {'a', 'b'}       ← 只取键

# ========== 转为 dict ==========
# 需要键值对的可迭代对象
dict([("a", 1), ("b", 2)])   # {'a': 1, 'b': 2}
dict(zip("abc", [1, 2, 3]))  # {'a': 1, 'b': 2, 'c': 3}
# dict([1, 2, 3])            # ❌ TypeError —— 列表元素不是键值对
```

如果你希望转换后保留明确顺序，应主动排序，而不要依赖 `set` 的遍历结果：

```python
numbers = [3, 1, 2, 1]
unique_sorted = sorted(set(numbers))
print(unique_sorted)   # [1, 2, 3]
```

### 3.5.2 转换时的信息增减

转换过程中，数据可能**增加、减少或改变形式**：

```python
# list → set：可能丢失信息（重复元素、顺序）
original = [3, 1, 2, 1, 3]
converted = set(original)          # {1, 2, 3} ← 丢失了重复和顺序

# dict → list：丢失值信息（只保留键）
d = {"a": 1, "b": 2}
keys_only = list(d)                # ['a', 'b'] ← 值 1, 2 丢了
# 保留完整信息的方式：
items = list(d.items())            # [('a', 1), ('b', 2)]
```

---

## 3.6 容器的比较操作

### 3.6.1 序列的比较：逐元素字典序

`list` 和 `tuple` 支持 `<`、`<=`、`>`、`>=`、`==`、`!=` 比较。比较规则是**逐元素、按字典序**（和 C++ STL 一致）：

```python
# 逐元素比较
print([1, 2, 3] == [1, 2, 3])    # True
print([1, 2, 3] < [1, 2, 4])     # True  ← 前两个相同，第三个 3 < 4
print([1, 2, 3] < [1, 3])        # True  ← 第二个 2 < 3
print([1, 2] < [1, 2, 0])        # True  ← 前两个相同，但左边更短

# 元组同理
print((1, 2) < (1, 3))           # True
print((1,) < (1, 0))             # True

# 不同类型不能比较（Python 3）
# [1, 2] < (1, 2)                # ❌ TypeError
```

> ⚠️ **进一步说明**：序列不仅要求“容器类型兼容”，还要求参与比较的**对应元素本身也能比较**。
>
> ```python
> # [1, "2"] < [1, 3]   # ❌ TypeError: 'str' 和 'int' 不能做大小比较
> ```

也就是说，“列表支持大小比较”不代表“任意两个列表都能比较”。

### 3.6.2 集合的比较：子集/超集关系

集合的比较运算符含义与序列不同，表示的是**子集/超集关系**：

```python
a = {1, 2, 3}
b = {1, 2, 3, 4, 5}

print(a < b)     # True  ← a 是 b 的真子集
print(a <= b)    # True  ← a 是 b 的子集
print(b > a)     # True  ← b 是 a 的真超集
print(a == b)    # False

print({1, 2} <= {1, 2})    # True  ← 子集（包含相等的情况）
print({1, 2} < {1, 2})     # False ← 真子集（不包含相等）
```

### 3.6.3 字典的比较

字典只支持 `==` 和 `!=`，不支持大小比较：

```python
print({"a": 1, "b": 2} == {"b": 2, "a": 1})   # True ← 与插入顺序无关
print({"a": 1} != {"a": 2})                     # True

# {"a": 1} < {"b": 2}   # ❌ TypeError: '<' not supported between instances of 'dict'
```

---

## 3.7 本章小结

|    通用操作     |    用途    |   list    |   tuple   |     dict     |     set     |
| :-------------: | :--------: | :-------: | :-------: | :----------: | :---------: |
|     `len()`     |  元素数量  |     ✅     |     ✅     |      ✅       |      ✅      |
| `in` / `not in` |  成员测试  |  ✅(O(n))  |  ✅(O(n))  | ✅(O(1),查键) |   ✅(O(1))   |
|   `for` 遍历    |  逐个访问  |     ✅     |     ✅     |      ✅       |      ✅      |
|    `bool()`     | 空容器判断 |     ✅     |     ✅     |      ✅       |      ✅      |
|   构造器转换    |  类型互转  |     ✅     |     ✅     |      ✅       |      ✅      |
|   `==` / `!=`   |  等值比较  |     ✅     |     ✅     |      ✅       |      ✅      |
|  `<` `>` 比较   | 大小/包含  | ✅(字典序) | ✅(字典序) |      ❌       | ✅(子集关系) |

---

# 第4章 列表入门：定义、特点与典型应用场景

---

## 4.1 什么是列表？

**列表（list）** 是 Python 中最基础、最常用、最灵活的容器类型。

你可以把它理解为一个**可以自由伸缩、随时修改的有序数据序列**。它就像一个智能书架——你可以随时往上面放书、取书、调换书的顺序，书架还能自动扩容。

从语言分类上说，列表属于**可变序列（mutable sequence）**。这一定义很重要，因为它直接决定了列表支持：

- 通过索引访问元素；
- 通过切片获得子序列；
- 原地修改、插入、删除；
- 按顺序遍历和比较。

```python
# 列表的基本形态
numbers = [10, 20, 30, 40, 50]
names = ["Alice", "Bob", "Charlie"]
mixed = [1, "hello", 3.14, True, None, [1, 2]]  # 可以混合任意类型
empty = []  # 空列表
```

### 4.1.1 列表的核心特征

|     特征     |                   说明                   |
| :----------: | :--------------------------------------: |
|   **有序**   | 元素按插入顺序排列，有确定的位置（索引） |
|   **可变**   |      创建后可以增删改元素，id 不变       |
|  **可重复**  |           同一个值可以出现多次           |
|   **异构**   |          可以存储不同类型的元素          |
| **动态长度** |      长度可以随时变化，自动管理内存      |

### 4.1.2 列表的语法细节

```python
# 基本创建
a = [1, 2, 3]

# 末尾可以有逗号（trailing comma），不影响结果
b = [1, 2, 3,]     # 完全等价于 [1, 2, 3]

# 多行书写（当元素很多时推荐）
config = [
    "option_a",
    "option_b",
    "option_c",     # 末尾逗号在多行写法中推荐保留，方便后续添加
]

# 嵌套列表
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

# 用 list() 构造器创建
c = list()            # 空列表，等价于 []
d = list("hello")     # ['h', 'e', 'l', 'l', 'o']
e = list(range(5))    # [0, 1, 2, 3, 4]
```

---

## 4.2 列表与其他容器的对比定位

列表是"万能选手"，但并非所有场景都是最优选择：

|            场景            | 推荐容器 |           原因           |
| :------------------------: | :------: | :----------------------: |
|       需要频繁增删改       | `list` ✅ |      可变，操作灵活      |
|        数据固定不变        | `tuple`  | 不可变，更安全、更省内存 |
|      需要通过名称查找      |  `dict`  |   键值映射，O(1) 查找    |
|          需要去重          |  `set`   |         自动唯一         |
| 需要有序、可修改的通用序列 | `list` ✅ |       最通用的选择       |

> 💡 **实用原则**：**当你不确定用什么容器时，先用列表**。等到你发现列表不够好用（比如需要去重、需要快速查找、需要不可变保证）时，再换成更合适的容器。

---

## 4.3 列表的典型应用场景

### 场景1：收集和积累数据

```python
# 收集用户输入
inputs = []
for _ in range(3):
    val = input("请输入一个数字: ")
    inputs.append(int(val))
print(f"你输入了: {inputs}")
print(f"总和: {sum(inputs)}")
```

### 场景2：维护有序序列

```python
# 任务清单
tasks = ["写代码", "看文档", "测试"]
tasks.append("部署")                   # 添加任务
tasks.insert(0, "开早会")              # 在开头插入
completed = tasks.pop(1)               # 完成第2个任务
print(f"完成了: {completed}")
print(f"剩余: {tasks}")
```

### 场景3：存储表格/矩阵数据

```python
# 3x3 矩阵
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

# 访问第2行第3列
print(matrix[1][2])   # 6

# 遍历所有元素
for row in matrix:
    for val in row:
        print(val, end=" ")
    print()
```

### 场景4：作为栈（Stack）使用

```python
# 列表天然支持栈操作（后进先出）
stack = []
stack.append("A")    # 入栈
stack.append("B")
stack.append("C")
print(stack)         # ['A', 'B', 'C']

top = stack.pop()    # 出栈（弹出最后一个）
print(top)           # C
print(stack)         # ['A', 'B']
```

### 场景5：数据转换与过滤

```python
# 将所有成绩转换为等级
scores = [95, 82, 67, 43, 88, 71]
grades = []
for s in scores:
    if s >= 90:
        grades.append("A")
    elif s >= 80:
        grades.append("B")
    elif s >= 60:
        grades.append("C")
    else:
        grades.append("D")

print(list(zip(scores, grades)))
# [(95, 'A'), (82, 'B'), (67, 'C'), (43, 'D'), (88, 'B'), (71, 'C')]

# 用列表推导式更简洁
passing = [s for s in scores if s >= 60]
print(passing)   # [95, 82, 67, 88, 71]
```

---

## 4.4 列表的内存模型

理解列表在内存中的存储方式有助于你理解后续的很多行为：

```python
a = [10, "hello", [1, 2]]
```

在内存中，列表对象存储的是**一组引用（指针）**，而不是值本身：

```
变量 a ──→ list 对象（id=xxx）
              [0] ──→ int 对象: 10
              [1] ──→ str 对象: "hello"
              [2] ──→ list 对象: [1, 2]
                         [0] ──→ int: 1
                         [1] ──→ int: 2
```

关键点：
1. **列表存储的是引用**，不是值的副本。
2. 这就是为什么列表可以包含不同类型的元素——因为它只存引用，不关心引用指向什么类型。
3. 赋值 `b = a` 只是让 `b` 也指向同一个列表对象，不会复制内容。

```python
a = [1, 2, 3]
b = a              # b 和 a 指向同一个列表
b.append(4)
print(a)           # [1, 2, 3, 4] ← a 也受影响了！
print(a is b)      # True
```

### 4.4.1 列表的动态扩容机制（Over-allocation）

作为希望深入理解底层运行原理的学者，理解列表的动态扩容非常重要。当列表元素数量超过现有内存容量时，Python 的 `list` 会在内存中申请一块**更大的新数组**，并将原有的对象指针逐个复制过去。

```python
import sys

# 观察列表容量如何动态增长
lst = []
print(sys.getsizeof(lst))  # 基础大小，例如 56 字节（不同系统和Python版本可能不同）

lst.append(1)
print(sys.getsizeof(lst))  # 增长，例如 88 字节（说明内部预分配了备用空间）

lst.append(2)
print(sys.getsizeof(lst))  # 依然是 88 字节，直接使用备用空间

lst.append(3)
```

这种“预分配空间”（Over-allocation）的设计，避免了每一次 `append` 都触发昂贵的内存重新分配操作。这使得 `append` 动作的**均摊时间复杂度能够保持在 O(1)**。该机制与 C++ 中 `std::vector` 遇到 `size == capacity` 时的扩容策略如出一辙。

### 4.4.2 别名、浅拷贝与深层对象

这也是初学者最容易出错的地方之一：**“切片复制了外层列表，但没有复制内部嵌套对象。”**

```python
original = [[1, 2], [3, 4]]
copied = original[:]          # 或 list(original)、original.copy()

print(original is copied)         # False  ← 外层列表不是同一个对象
print(original[0] is copied[0])   # True   ← 但内部子列表仍是同一个对象

copied[0].append(99)
print(original)   # [[1, 2, 99], [3, 4]]  ← 原列表内部也被改了
```

结论：

1. `b = a` 是**别名**，完全没有复制。
2. `a[:]`、`list(a)`、`a.copy()` 是**浅拷贝**，只复制最外层容器。
3. 如果列表里还有列表、字典等可变对象，浅拷贝仍可能共享内部数据。

后续学习中如果涉及“完全独立的嵌套复制”，通常需要 `copy.deepcopy()`。

---

## 4.5 本章小结

|    要点    |                      内容                      |
| :--------: | :--------------------------------------------: |
| 列表是什么 |     有序、可变、可重复、异构的动态长度序列     |
|  创建方式  |      `[]` 字面量、`list()` 构造器、推导式      |
|  适用场景  | 收集数据、维护有序序列、栈、矩阵、数据过滤转换 |
|  内存模型  |         存储引用（指针），赋值创建别名         |
|  默认首选  |         不确定用什么容器时，先选 list          |

---

# 第5章 列表的基本操作：索引、切片、遍历与增删改查

---

## 5.1 索引（Indexing）

### 5.1.1 正向索引

列表的索引从 **0** 开始，这是 Python（以及大多数编程语言）的统一约定：

```python
fruits = ["苹果", "香蕉", "橘子", "葡萄", "西瓜"]
#  索引:    0       1       2       3       4

print(fruits[0])    # 苹果（第1个元素）
print(fruits[1])    # 香蕉（第2个元素）
print(fruits[4])    # 西瓜（最后一个元素）
# print(fruits[5])  # ❌ IndexError: list index out of range
```

### 5.1.2 负向索引

Python 独特而实用的特性——**负索引**，从末尾往前数：

```python
fruits = ["苹果", "香蕉", "橘子", "葡萄", "西瓜"]
#  正索引:  0       1       2       3       4
#  负索引: -5      -4      -3      -2      -1

print(fruits[-1])    # 西瓜（最后一个）
print(fruits[-2])    # 葡萄（倒数第二个）
print(fruits[-5])    # 苹果（倒数第五个 = 第一个）
```

> 💡 **负索引的本质**：`fruits[-n]` 等价于 `fruits[len(fruits) - n]`。

### 5.1.3 通过索引修改元素

列表是可变的，所以可以通过索引直接修改元素：

```python
colors = ["red", "green", "blue"]
colors[1] = "yellow"
print(colors)   # ['red', 'yellow', 'blue']

colors[-1] = "purple"
print(colors)   # ['red', 'yellow', 'purple']
```

---

## 5.2 切片（Slicing）

切片是 Python 序列操作中最强大、最优雅的特性之一。

### 5.2.1 基本语法：`list[start:stop:step]`

```python
nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# [start:stop] —— 从 start 到 stop-1（左闭右开）
print(nums[2:5])      # [2, 3, 4]
print(nums[0:3])      # [0, 1, 2]

# 省略 start —— 从头开始
print(nums[:4])       # [0, 1, 2, 3]

# 省略 stop —— 到末尾结束
print(nums[6:])       # [6, 7, 8, 9]

# 都省略 —— 复制整个列表
print(nums[:])        # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# [start:stop:step] —— 带步长
print(nums[0:10:2])   # [0, 2, 4, 6, 8]  ← 每隔一个取一个
print(nums[1:10:2])   # [1, 3, 5, 7, 9]  ← 所有奇数位置
print(nums[::3])      # [0, 3, 6, 9]     ← 每隔两个取一个
# print(nums[::0])     # ❌ ValueError: step 不能为 0
```

### 5.2.2 负向切片

```python
nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# 最后三个
print(nums[-3:])      # [7, 8, 9]

# 除了最后两个
print(nums[:-2])      # [0, 1, 2, 3, 4, 5, 6, 7]

# 反转列表
print(nums[::-1])     # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]

# 从倒数第1个到倒数第4个（反向）
print(nums[-1:-4:-1]) # [9, 8, 7]
```

> 💡 当 `step` 为负数时，切片方向会反过来，此时通常要满足 `start` 在 `stop` 的右侧，否则结果可能为空列表。

### 5.2.3 切片的重要特性

**1. 切片不会引发 IndexError**：即使索引超出范围，切片也会优雅地处理。

```python
nums = [1, 2, 3]
print(nums[0:100])    # [1, 2, 3]  ← 不报错，只取到末尾
print(nums[100:200])  # []         ← 空列表，不报错
print(nums[-100:2])   # [1, 2]    ← 负索引超出也不报错
```

**2. 切片返回新列表**：原列表不受影响。

```python
original = [1, 2, 3, 4, 5]
sliced = original[1:4]
sliced[0] = 99
print(sliced)     # [99, 3, 4]   ← 修改了切片
print(original)   # [1, 2, 3, 4, 5]  ← 原列表不受影响
```

**3. 切片赋值（可以改变列表长度）**：

```python
lst = [0, 1, 2, 3, 4, 5]

# 替换中间部分（等长替换）
lst[2:4] = [20, 30]
print(lst)   # [0, 1, 20, 30, 4, 5]

# 替换中间部分（插入更多元素）
lst[2:4] = [200, 300, 400, 500]
print(lst)   # [0, 1, 200, 300, 400, 500, 4, 5]

# 删除中间部分
lst[2:6] = []
print(lst)   # [0, 1, 4, 5]

# 在特定位置插入（不删除任何元素）
lst[1:1] = [10, 11, 12]
print(lst)   # [0, 10, 11, 12, 1, 4, 5]
```

如果切片带有步长，赋值规则会更严格：

```python
lst = [0, 1, 2, 3, 4, 5]
lst[::2] = [10, 20, 30]   # 合法：左边选中了 3 个位置
print(lst)                # [10, 1, 20, 3, 30, 5]

# lst[::2] = [10, 20]     # ❌ ValueError：元素个数不匹配
```

这是因为“扩展切片赋值”要求左右两边选中的元素数量一致。

---

## 5.3 遍历列表

```python
fruits = ["苹果", "香蕉", "橘子"]

# 方式1：直接遍历元素（最常用）
for fruit in fruits:
    print(fruit)

# 方式2：带索引遍历（用 enumerate）
for i, fruit in enumerate(fruits):
    print(f"[{i}] {fruit}")

# 方式3：反向遍历
for fruit in reversed(fruits):
    print(fruit)       # 橘子、香蕉、苹果

# 方式4：遍历切片
for fruit in fruits[1:]:
    print(fruit)       # 香蕉、橘子（跳过第一个）
```

> ⚠️ **遍历时尽量不要同时修改同一个列表**。例如一边 `for x in lst` 一边 `remove()`，很容易漏掉元素或产生难以理解的结果。
>
> 更稳妥的做法通常是：
>
> - 遍历副本：`for x in lst[:]`
> - 或使用列表推导式重新生成新列表
>
> ```python
> nums = [1, 2, 3, 4, 5, 6]
> nums = [x for x in nums if x % 2 == 0]
> print(nums)   # [2, 4, 6]
> ```

---

## 5.4 增（添加元素）

```python
lst = [1, 2, 3]

# append：在末尾添加一个元素
lst.append(4)             # [1, 2, 3, 4]

# insert：在指定位置插入一个元素
lst.insert(0, 0)          # [0, 1, 2, 3, 4]  ← 在索引0处插入
lst.insert(2, 1.5)        # [0, 1, 1.5, 2, 3, 4]

# extend：在末尾添加多个元素（展开可迭代对象）
lst.extend([5, 6, 7])     # [0, 1, 1.5, 2, 3, 4, 5, 6, 7]

# + 拼接：创建新列表（不修改原列表）
new_lst = lst + [8, 9]
print(lst)                # 原列表不变
print(new_lst)            # [..., 8, 9]

# * 重复
zeros = [0] * 5           # [0, 0, 0, 0, 0]
```

> ⚠️ **append vs extend 的区别**：
> ```python
> a = [1, 2, 3]
> a.append([4, 5])      # [1, 2, 3, [4, 5]]  ← 把列表作为一个整体添加
> 
> b = [1, 2, 3]
> b.extend([4, 5])      # [1, 2, 3, 4, 5]    ← 把列表中的元素逐个添加
> ```

> ⚠️ **列表乘法的经典陷阱**：
> ```python
> grid = [[0] * 3] * 2
> grid[0][0] = 99
> print(grid)   # [[99, 0, 0], [99, 0, 0]]
> ```
>
> 原因是两行其实引用了**同一个内部列表对象**。正确写法是：
>
> ```python
> grid = [[0] * 3 for _ in range(2)]
> ```

---

## 5.5 删（删除元素）

```python
lst = ["a", "b", "c", "d", "e", "b"]

# remove：按值删除（删除第一个匹配项）
lst.remove("b")
print(lst)              # ['a', 'c', 'd', 'e', 'b'] ← 只删了第一个 "b"
# lst.remove("z")       # ❌ ValueError: list.remove(x): x not in list

# pop：按索引删除并返回被删除的元素
item = lst.pop(2)       # 删除索引 2
print(item)             # 'd'
print(lst)              # ['a', 'c', 'e', 'b']

last = lst.pop()        # 不指定索引，默认删除最后一个
print(last)             # 'b'
print(lst)              # ['a', 'c', 'e']
# [].pop()              # ❌ IndexError: 空列表不能 pop

# del：按索引或切片删除
del lst[0]
print(lst)              # ['c', 'e']

lst = [1, 2, 3, 4, 5]
del lst[1:4]            # 删除切片
print(lst)              # [1, 5]

# clear：清空列表
lst.clear()
print(lst)              # []
```

---

## 5.6 改（修改元素）

```python
lst = [10, 20, 30, 40, 50]

# 按索引修改
lst[0] = 99
print(lst)          # [99, 20, 30, 40, 50]

# 按切片批量修改
lst[1:3] = [200, 300]
print(lst)          # [99, 200, 300, 40, 50]

# 切片赋值可以改变长度
lst[1:3] = [2, 3, 4, 5, 6]
print(lst)          # [99, 2, 3, 4, 5, 6, 40, 50]
```

---

## 5.7 查（查找元素）

```python
lst = [10, 20, 30, 20, 40]

# in：判断是否存在
print(20 in lst)       # True
print(99 in lst)       # False

# index：查找元素的索引（返回第一个匹配项）
print(lst.index(20))       # 1（第一个 20 的位置）
print(lst.index(20, 2))    # 3（从索引 2 开始找）
# lst.index(99)            # ❌ ValueError

# count：统计元素出现次数
print(lst.count(20))   # 2
print(lst.count(99))   # 0
```

> 💡 `index()`、`count()`、`in` 在列表上的时间复杂度通常都是 `O(n)`。如果你的核心需求是“高频判断某元素是否存在”，往往应考虑 `set` 或 `dict`。

---

## 5.8 本章操作速查表

|    操作    |     方法/语法      |      功能       |  返回值  |
| :--------: | :----------------: | :-------------: | :------: |
|    索引    |      `lst[i]`      | 访问第 i 个元素 |  元素值  |
|    切片    |    `lst[a:b:c]`    |    取子序列     |  新列表  |
|    添加    |  `lst.append(x)`   |  末尾添加一个   |  `None`  |
|    插入    | `lst.insert(i, x)` |  指定位置插入   |  `None`  |
|    扩展    | `lst.extend(iter)` |  末尾添加多个   |  `None`  |
|  删除(值)  |  `lst.remove(x)`   |  删第一个匹配   |  `None`  |
| 删除(索引) |    `lst.pop(i)`    |    删并返回     | 被删元素 |
| 删除(切片) |   `del lst[a:b]`   |    批量删除     |    —     |
|    清空    |   `lst.clear()`    |    删除所有     |  `None`  |
|    修改    |    `lst[i] = x`    |    替换元素     |    —     |
|    查找    |   `lst.index(x)`   |  首次出现位置   |  索引值  |
|    计数    |   `lst.count(x)`   |    出现次数     |   整数   |
|  成员测试  |     `x in lst`     |    是否存在     |  布尔值  |
|    长度    |     `len(lst)`     |    元素数量     |   整数   |

---

