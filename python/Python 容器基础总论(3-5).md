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

> 📌 **版本与输出说明（建议先读）**：
> 1. 本文默认 Python 3.10+（至少 3.9+，因为示例中用到了 `dict` 合并运算符 `|`）。
> 2. `set`/`frozenset` 的打印顺序不保证固定，示例中的元素顺序仅用于说明内容，不作为判题标准。
> 3. `dict` 在 Python 3.7+ 保证保持插入顺序；3.6 仅为 CPython 实现细节，不建议依赖。

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

# set —— 检查元素是否在集合中（通常更快，平均 O(1)）
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
> - `set` 和 `dict`：平均 O(1)（哈希表）；最坏在严重冲突时可退化到 O(n)
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
    
#print(*[10, 20, 30], sep=" ") 
#  *号负责把列表拆开，sep=" " 负责在两个元素之间插入空格

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

for index, item in enumerate([10, 20, 30]):
    if index == 0:
        print(item, end="")      # 第一个元素，直接打印 10
    else:
        print(f" {item}", end="") # 后续元素，打印 " 20" 和 " 30"
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

# 第6章 元组入门：定义、特点与典型应用场景

---

## 6.1 什么是元组？

**元组（tuple）** 是 Python 中四大内置容器类型之一，也是仅次于列表的第二种**序列（sequence）**类型。

你可以把它理解为一个**"上了锁的列表"**——它和列表一样按顺序排列元素、支持索引和切片，但一旦创建就**不能修改**。就像一封密封的信，你可以看里面的内容，但无法增删改任何东西。

从语言分类上说，元组属于**不可变序列（immutable sequence）**。这一定义直接决定了：

- ✅ 元组支持所有**读取类**操作：索引、切片、遍历、成员测试
- ❌ 元组**不支持**任何**修改类**操作：不能 append、insert、remove、pop，也不能通过索引赋值

```python
# 元组的基本形态
numbers = (10, 20, 30, 40, 50)
names = ("Alice", "Bob", "Charlie")
mixed = (1, "hello", 3.14, True, None)   # 可以混合任意类型
empty = ()                                # 空元组
single = (42,)                            # 单元素元组 ← 注意逗号！
```

### 6.1.1 元组的核心特征

|    特征     |                     说明                     |
| :---------: | :------------------------------------------: |
|  **有序**   | 元素按创建时的顺序排列，有确定的位置（索引） |
| **不可变**  |   创建后不能增删改元素，也不能重新排列顺序   |
| **可重复**  |             同一个值可以出现多次             |
|  **异构**   |            可以存储不同类型的元素            |
| **可哈希**★ |    若所有元素本身可哈希，则元组整体可哈希    |

> ★ **可哈希的意义**：可哈希意味着元组可以作为**字典的键**和**集合的元素**，这是列表做不到的。后面我们会反复看到这一点的重要性。

### 6.1.2 元组的语法细节

#### 1. 基本创建

```python
# 使用圆括号
t1 = (1, 2, 3)

# 实际上，逗号才是关键，圆括号在多数情况下可以省略
t2 = 1, 2, 3           # 合法！与 (1, 2, 3) 等价
print(type(t2))         # <class 'tuple'>
print(t2)               # (1, 2, 3)
```

#### 2. 单元素元组的陷阱（最常见的新手错误之一）

```python
# ❌ 错误写法——这不是元组，只是一个带括号的整数！
not_a_tuple = (42)
print(type(not_a_tuple))   # <class 'int'>
print(not_a_tuple)         # 42

# ✅ 正确写法——必须有逗号
is_a_tuple = (42,)
print(type(is_a_tuple))    # <class 'tuple'>
print(is_a_tuple)          # (42,)

# ✅ 省略括号也行，但可读性差
also_a_tuple = 42,
print(type(also_a_tuple))  # <class 'tuple'>
```

> 🧠 **为什么需要逗号？**  
> 因为 Python 把 `(42)` 理解为"数学括号中的表达式 42"，而不是"包含 42 的容器"。逗号 `,` 才是标记"这是一个元组"的语法要素。圆括号只是用来分组的。

#### 3. 多行书写

```python
# 当元素较多时，推荐多行书写
student = (
    "Alice",
    20,
    "Computer Science",
    3.8,       # 末尾逗号推荐保留
)
```

#### 4. 用 `tuple()` 构造器创建

```python
t = tuple()               # 空元组，等价于 ()
t = tuple([1, 2, 3])      # 从列表转换：(1, 2, 3)
t = tuple("hello")        # 从字符串转换：('h', 'e', 'l', 'l', 'o')
t = tuple(range(5))       # 从 range 转换：(0, 1, 2, 3, 4)

# ⚠️ 元组没有推导式，圆括号里的推导式是生成器表达式
gen = (x ** 2 for x in range(5))
print(type(gen))           # <class 'generator'>

# 需要元组时，把生成器传给 tuple()
t = tuple(x ** 2 for x in range(5))
print(t)                   # (0, 1, 4, 9, 16)
```

### 6.1.3 元组的"不可变"到底意味着什么？

这是理解元组最关键的概念，必须精准把握。

**不可变 = 元组对象本身的元素引用不能改变。**

- 不能增加元素（没有 `append`、`insert`、`extend`）
- 不能删除元素（没有 `remove`、`pop`、`clear`）
- 不能替换元素（`t[0] = 99` 会报错）

```python
t = (1, 2, 3)

# ❌ 以下操作全部报错
# t[0] = 99         # TypeError: 'tuple' object does not support item assignment
# t.append(4)       # AttributeError: 'tuple' object has no attribute 'append'
# del t[0]          # TypeError: 'tuple' object doesn't support item deletion
```

**但是！如果元组中包含可变对象（比如列表），那个可变对象的内容是可以被修改的：**

```python
t = (1, [2, 3], "hello")

# 修改元组中的列表内容
t[1].append(4)
print(t)           # (1, [2, 3, 4], 'hello')  ← 列表内容变了

# 但不能替换元组的元素引用
# t[1] = [5, 6]    # ❌ TypeError
```

用一张概念图来理解：

```
元组 t ──→ tuple 对象
            [0] ──→ int: 1          (不可变，不能换指向)
            [1] ──→ list: [2, 3]    (不能换指向，但 list 本身可变)
            [2] ──→ str: "hello"    (不可变，不能换指向)
```

元组保证的是**"引用不可变"**（不能让 `[1]` 指向别的对象），不保证**"深层不可变"**（如果被引用的对象本身是可变的，它的内容仍然可以变）。

> ⚠️ **哈希的影响**：如果元组中包含了不可哈希的元素（如列表），那么整个元组也变得不可哈希：
>
> ```python
> hash((1, 2, 3))           # ✅ 正常返回哈希值
> # hash((1, [2, 3]))       # ❌ TypeError: unhashable type: 'list'
> ```

---

## 6.2 元组与列表的对比定位

既然元组和列表如此相似（同为序列），为什么 Python 要设计两种类型？

### 6.2.1 功能对比

|     特性     |    列表 `list`    |      元组 `tuple`       |
| :----------: | :---------------: | :---------------------: |
|    可变性    | ✅ 可变（mutable） |  ❌ 不可变（immutable）  |
|  索引、切片  |         ✅         |            ✅            |
|  增删改操作  |         ✅         |            ❌            |
|    可哈希    |         ❌         | ✅（当所有元素可哈希时） |
|  可做字典键  |         ❌         | ✅（当所有元素可哈希时） |
| 可做集合元素 |         ❌         | ✅（当所有元素可哈希时） |
|    推导式    | ✅ `[... for ...]` |            ❌            |
|   内存占用   |  较多（有预留）   |   较少（无预留空间）    |
|   创建速度   |       较慢        |          较快           |

### 6.2.2 设计哲学的差异

这才是最核心的区别——**语义不同**：

- **列表**表达**"同类元素的集合"**——强调的是数量不定的"一堆东西"
  - 例如：`[90, 85, 78, 92]`（一组成绩）、`["Alice", "Bob", "Charlie"]`（一组人名）
  - 元素之间是**同质**的、**平等**的

- **元组**表达**"一条记录 / 一个组合"**——强调的是固定结构中各个字段
  - 例如：`("Alice", 20, "CS")`（一条学生记录：姓名、年龄、专业）
  - 各元素之间是**异质**的、**有各自含义**的，位置决定语义

```python
# 列表的典型用法——同类元素的可变集合
scores = [90, 85, 78, 92]           # 一组成绩，随时可能增减
students = ["Alice", "Bob", "Charlie"]  # 一组学生

# 元组的典型用法——固定结构的记录
point = (3.0, 4.0)                  # 一个坐标点：(x, y)
student = ("Alice", 20, "CS")       # 一条记录：(姓名, 年龄, 专业)
rgb = (255, 128, 0)                 # 一个颜色：(R, G, B)
```

> 💡 **经验法则**：
>
> - 如果你会对元素进行增删改，用**列表**
> - 如果数据创建后就不应该被修改（坐标、配置项、数据库查询结果），用**元组**
> - 如果你需要把数据用作字典键或集合元素，用**元组**

### 6.2.3 性能差异

```python
import sys

lst = [1, 2, 3, 4, 5]
tup = (1, 2, 3, 4, 5)

print(sys.getsizeof(lst))   # 例如 96 字节
print(sys.getsizeof(tup))   # 例如 80 字节 ← 更小

# 创建速度对比（概念性说明，不必运行验证）
# 元组的创建速度通常快于列表，因为：
# 1. 不需要预分配空间
# 2. CPython 对小元组有缓存池（free list）优化
```

> 🧠 **CPython 的元组缓存池**：当一个小元组（通常长度 ≤ 20）被销毁时，CPython 不会立即释放内存，而是放入一个缓存池。下次创建同样大小的元组时，直接从缓存池取，避免了系统级别的内存分配调用。这使得元组的创建和销毁在高频场景下非常高效。

---

## 6.3 元组的典型应用场景

### 场景1：函数返回多个值

这是元组最经典、最频繁的使用场景。Python 函数可以"返回多个值"，实际上返回的是一个元组：

```python
def divide(a, b):
    """返回商和余数"""
    return a // b, a % b    # 实际返回一个元组

result = divide(17, 5)
print(result)               # (3, 2)
print(type(result))         # <class 'tuple'>

# 解包接收（最推荐的写法）
quotient, remainder = divide(17, 5)
print(f"17 ÷ 5 = {quotient} 余 {remainder}")   # 17 ÷ 5 = 3 余 2
```

Python 标准库中有大量函数返回元组：

```python
# divmod() —— 内置函数，返回 (商, 余数)
q, r = divmod(17, 5)
print(q, r)    # 3 2

# str.partition() —— 返回 (前部, 分隔符, 后部)
before, sep, after = "hello-world".partition("-")
print(before, sep, after)   # hello - world

# enumerate() —— 产出 (索引, 值) 元组
for i, ch in enumerate("abc"):
    print(i, ch)
```

### 场景2：作为字典的键

列表不可哈希，不能作为字典的键；元组（当元素全部可哈希时）可以：

```python
# 用坐标元组作为键，存储地图数据
grid = {}
grid[(0, 0)] = "起点"
grid[(3, 4)] = "宝藏"
grid[(7, 2)] = "陷阱"

print(grid[(3, 4)])    # 宝藏

# 实际应用：稀疏矩阵、图的邻接表、缓存等
# 用 (行, 列) 元组作为键存储非零元素
sparse_matrix = {
    (0, 1): 5,
    (2, 3): 8,
    (4, 0): 3,
}

# 获取某位置的值，不存在则返回 0
value = sparse_matrix.get((2, 3), 0)
print(value)   # 8
```

### 场景3：数据解包（Unpacking）

元组支持非常灵活的解包操作，这是 Python 独有的优雅特性：

```python
# 基本解包
name, age, major = ("Alice", 20, "CS")
print(name)    # Alice
print(age)     # 20

# 交换变量（Python 最优雅的写法）
a, b = 1, 2
a, b = b, a        # 右边先创建元组 (2, 1)，然后解包赋值
print(a, b)         # 2 1

# 星号解包（Python 3+）
first, *rest = (1, 2, 3, 4, 5)
print(first)    # 1
print(rest)     # [2, 3, 4, 5]  ← 注意：rest 是列表

first, *middle, last = (1, 2, 3, 4, 5)
print(first)    # 1
print(middle)   # [2, 3, 4]
print(last)     # 5

# 忽略不需要的值
_, _, third = (10, 20, 30)
print(third)    # 30

name, _, _ = ("Alice", 20, "CS")
print(name)    # Alice
```

> 💡 `_`（下划线）在解包中是约定俗成的"丢弃变量"，表示"我不关心这个值"。

### 场景4：不可变的配置数据

当你有一组不应该被修改的常量数据时，用元组比列表更安全：

```python
# 数据库连接配置
DB_CONFIG = ("localhost", 5432, "mydb", "admin")

# RGB 颜色常量
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)

# 方向常量
DIRECTIONS = ("North", "South", "East", "West")

# 如果有人不小心试图修改，Python 会立即报错
# RED[0] = 128    # ❌ TypeError —— 这就是元组的"安全保护"
```

### 场景5：作为集合的元素

```python
# 记录一组坐标点（需要去重）
visited_points = set()
visited_points.add((0, 0))
visited_points.add((1, 2))
visited_points.add((0, 0))        # 重复，不会添加
print(visited_points)              # {(0, 0), (1, 2)}

# 如果用列表就不行
# visited_points.add([0, 0])      # ❌ TypeError: unhashable type: 'list'
```

---

## 6.4 元组的内存模型

### 6.4.1 元组与列表的内存差异

```python
import sys

# 同样的数据，元组比列表更紧凑
lst = [1, 2, 3, 4, 5]
tup = (1, 2, 3, 4, 5)

print(sys.getsizeof(lst))   # 例如 96 字节
print(sys.getsizeof(tup))   # 例如 80 字节
```

原因：

1. **列表需要额外空间**：列表底层维护一个**可变长的引用数组**，会**预分配**多余的空间以便 `append` 时不必每次都扩容。
2. **元组没有预分配**：元组是不可变的，创建时就确定了大小，不需要留备用空间。

```
列表内存布局:
┌──────────┬───────┬─────────────────────────────────┐
│ ob_refcnt│ob_size│ ob_item (引用数组,可能有空余位) │
└──────────┴───────┴─────────────────────────────────┘

元组内存布局:
┌──────────┬───────┬─────────────────┐
│ ob_refcnt│ob_size│ ob_item (刚好)  │
└──────────┴───────┴─────────────────┘
```

### 6.4.2 元组的缓存机制

CPython（Python 的标准实现）对元组有一项重要的优化——**小元组缓存池（free list）**：

```python
# 空元组是单例——全局只有一个
a = ()
b = ()
print(a is b)    # True ← 同一个对象

# 小元组可能被缓存复用
t1 = (1, 2, 3)
t_id = id(t1)
del t1               # 删除 t1，元组进入缓存池
t2 = (4, 5, 6)       # 创建新元组，可能复用同一块内存
# id(t2) 可能等于 t_id
```

> 💡 **实际意义**：在高频创建和销毁小元组的场景（如循环中大量使用 `enumerate()`、`zip()` 等），这种缓存机制可以显著减少内存分配开销。

### 6.4.3 不可变的连锁效应

元组的不可变性带来了几个重要的连锁效应：

```python
# 1. 别名是安全的
a = (1, 2, 3)
b = a              # b 和 a 指向同一个元组
# 因为元组不可变，所以不存在"一方修改影响另一方"的问题
# 这与列表的别名陷阱形成鲜明对比

# 2. 不需要浅拷贝
# 对元组来说，t[:] 和 tuple(t) 通常直接返回原对象
t = (1, 2, 3)
t2 = t[:]
print(t is t2)     # True ← CPython 优化：不可变对象不需要复制

# 对比列表
lst = [1, 2, 3]
lst2 = lst[:]
print(lst is lst2) # False ← 列表必须创建新对象

# 3. 并发读取更安全
# 元组不可变，多个线程并发读取同一个元组通常不需要额外加锁；
# 但这不等于程序整体“线程安全”，共享可变状态仍需同步机制。
```

---

## 6.5 本章小结

|    要点    |                      内容                       |
| :--------: | :---------------------------------------------: |
| 元组是什么 |    有序、不可变、可重复、异构的固定长度序列     |
|  创建方式  |   `()` 字面量（逗号是关键）、`tuple()` 构造器   |
|  核心特点  |      不可变 → 可哈希 → 可做字典键/集合元素      |
|  适用场景  | 函数多返回值、记录/坐标、字典键、解包、配置常量 |
| 与列表区别 | 语义不同：列表="一组同类数据"，元组="一条记录"  |

---

# 第7章 元组的基本操作：索引、切片、解包与只读查询

---

## 7.1 索引（Indexing）

元组的索引规则与列表完全一致，因为它们同属序列类型。

### 7.1.1 正向索引

```python
colors = ("red", "green", "blue", "yellow", "purple")
#  索引:    0       1        2        3         4

print(colors[0])     # red
print(colors[2])     # blue
print(colors[4])     # purple
# print(colors[5])   # ❌ IndexError: tuple index out of range
```

### 7.1.2 负向索引

```python
colors = ("red", "green", "blue", "yellow", "purple")
#  正索引:   0       1        2        3         4
#  负索引:  -5      -4       -3       -2        -1

print(colors[-1])    # purple（最后一个）
print(colors[-3])    # blue（倒数第三个）
print(colors[-5])    # red（倒数第五个 = 第一个）
```

### 7.1.3 索引只读——不能通过索引修改

这是元组与列表的**核心差异**：

```python
colors = ("red", "green", "blue")

# ❌ 不能修改
# colors[0] = "pink"    # TypeError: 'tuple' object does not support item assignment

# 如果需要"修改"，只能创建一个新元组
new_colors = ("pink",) + colors[1:]
print(new_colors)    # ('pink', 'green', 'blue')
```

---

## 7.2 切片（Slicing）

元组的切片语法与列表完全一致，**但切片返回的是新元组（不是列表！）**。

### 7.2.1 基本切片

```python
nums = (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

print(nums[2:5])      # (2, 3, 4)      ← 返回元组
print(nums[:4])       # (0, 1, 2, 3)
print(nums[6:])       # (6, 7, 8, 9)
print(nums[:])        # (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
print(nums[::2])      # (0, 2, 4, 6, 8)
```

### 7.2.2 负向切片与反转

```python
nums = (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

print(nums[-3:])       # (7, 8, 9)
print(nums[:-2])       # (0, 1, 2, 3, 4, 5, 6, 7)
print(nums[::-1])      # (9, 8, 7, 6, 5, 4, 3, 2, 1, 0)  ← 反转元组
print(nums[-1:-4:-1])  # (9, 8, 7)
```

### 7.2.3 切片的特性

```python
nums = (1, 2, 3)

# 1. 切片不会引发 IndexError
print(nums[0:100])    # (1, 2, 3)
print(nums[100:200])  # ()

# 2. 切片返回新元组
sliced = nums[1:3]
print(sliced)         # (2, 3)
print(type(sliced))   # <class 'tuple'>

# 3. ⚠️ 元组不支持切片赋值（因为不可变）
# nums[1:3] = (20, 30)   # ❌ TypeError: 'tuple' object does not support item assignment
```

---

## 7.3 解包（Unpacking）——元组的核心操作

解包是元组最强大、最具特色的操作，在日常编程中使用频率极高。

### 7.3.1 基本解包

```python
# 变量数量必须与元组元素数量匹配
point = (3.0, 4.0)
x, y = point
print(f"x = {x}, y = {y}")    # x = 3.0, y = 4.0

# 三元素解包
name, age, major = ("Alice", 20, "Computer Science")
print(f"{name} 今年 {age} 岁，专业是 {major}")

# 不匹配会报错
# a, b = (1, 2, 3)    # ❌ ValueError: too many values to unpack
# a, b, c = (1, 2)    # ❌ ValueError: not enough values to unpack
```

### 7.3.2 星号解包（Extended Unpacking）

Python 3 引入的星号解包可以处理长度不定的序列：

```python
# 取首尾，中间全部收集
first, *middle, last = (1, 2, 3, 4, 5)
print(first)     # 1
print(middle)    # [2, 3, 4]    ← 注意：*middle 总是列表
print(last)      # 5

# 只取第一个，其余全部收集
head, *tail = (10, 20, 30, 40)
print(head)      # 10
print(tail)      # [20, 30, 40]

# 只取最后一个
*init, last = (10, 20, 30, 40)
print(init)      # [10, 20, 30]
print(last)      # 40

# 边界情况
a, *b = (1,)
print(a)         # 1
print(b)         # []    ← 空列表

a, *b = (1, 2)
print(a)         # 1
print(b)         # [2]   ← 单元素列表
```

> ⚠️ **限制**：一个解包表达式中最多只能有**一个**星号变量。
>
> ```python
> # *a, *b = (1, 2, 3, 4)   # ❌ SyntaxError: multiple starred expressions
> ```

### 7.3.3 嵌套解包

```python
# 嵌套元组可以嵌套解包
data = ("Alice", (90, 85, 78))
name, (score1, score2, score3) = data
print(f"{name}: {score1}, {score2}, {score3}")
# Alice: 90, 85, 78

# 更复杂的嵌套
record = ("Bob", ("CS", 3), [90, 88])
name, (dept, year), scores = record
print(f"{name}, {dept}系{year}年级, 成绩: {scores}")
# Bob, CS系3年级, 成绩: [90, 88]
```

### 7.3.4 解包在循环中的应用

```python
# 遍历元组列表时解包
students = [
    ("Alice", 90),
    ("Bob", 85),
    ("Charlie", 78),
]

for name, score in students:
    status = "及格" if score >= 60 else "不及格"
    print(f"{name}: {score}分 ({status})")

# 配合 enumerate
for i, (name, score) in enumerate(students, start=1):
    print(f"第{i}名: {name} - {score}分")

# 配合 dict.items()
config = {"host": "localhost", "port": 5432, "db": "mydb"}
for key, value in config.items():
    print(f"{key} = {value}")
```

### 7.3.5 交换变量的原理

```python
# Python 的变量交换写法
a, b = 1, 2
a, b = b, a      # 等价于：temp = (b, a); a, b = temp
print(a, b)       # 2 1

# 三变量轮换
a, b, c = 1, 2, 3
a, b, c = b, c, a
print(a, b, c)    # 2 3 1
```

> 💡 **原理解析**：右边的表达式 `b, a` 先求值，生成临时元组 `(2, 1)` ，然后再将这个元组解包赋值给左边的 `a, b`。因此不需要临时变量。

---

## 7.4 遍历元组

元组的遍历方式与列表完全一致：

```python
colors = ("red", "green", "blue", "yellow")

# 方式1：直接遍历元素
for color in colors:
    print(color, end=" ")
# red green blue yellow

print()

# 方式2：带索引遍历
for i, color in enumerate(colors):
    print(f"[{i}] {color}")

# 方式3：反向遍历
for color in reversed(colors):
    print(color, end=" ")
# yellow blue green red

print()

# 方式4：遍历切片
for color in colors[1:3]:
    print(color, end=" ")
# green blue
```

---

## 7.5 查询操作

元组作为不可变序列，只支持**只读的查询操作**。元组只有两个方法：`index()` 和 `count()`。

```python
t = (10, 20, 30, 20, 40, 20)

# ========== 成员测试 ==========
print(20 in t)         # True
print(99 in t)         # False
print(99 not in t)     # True

# ========== index()：查找元素首次出现的位置 ==========
print(t.index(20))         # 1（第一个 20 的索引）
print(t.index(20, 2))      # 3（从索引 2 开始找）
print(t.index(20, 4))      # 5（从索引 4 开始找）
# t.index(99)              # ❌ ValueError: tuple.index(x): x not in tuple

# ========== count()：统计元素出现次数 ==========
print(t.count(20))    # 3
print(t.count(99))    # 0

# ========== len()：获取长度 ==========
print(len(t))         # 6

# ========== min() / max() / sum() ==========
nums = (10, 5, 30, 15)
print(min(nums))      # 5
print(max(nums))      # 30
print(sum(nums))      # 60

# ========== sorted()：排序（返回列表，不是元组！） ==========
print(sorted(nums))           # [5, 10, 15, 30]  ← 注意是列表
print(tuple(sorted(nums)))    # (5, 10, 15, 30)  ← 转回元组
```

---

## 7.6 元组的拼接与重复

虽然元组不可变，但可以通过 `+` 和 `*` 创建**新元组**：

```python
# + 拼接
t1 = (1, 2, 3)
t2 = (4, 5, 6)
t3 = t1 + t2
print(t3)          # (1, 2, 3, 4, 5, 6)
print(t1)          # (1, 2, 3)  ← 原元组不变

# * 重复
t4 = (0,) * 5
print(t4)          # (0, 0, 0, 0, 0)

t5 = ("ab",) * 3
print(t5)          # ('ab', 'ab', 'ab')

# += 的真相
t = (1, 2)
old_id = id(t)
t += (3, 4)          # 实际上创建了一个新元组，然后让 t 指向它
print(t)             # (1, 2, 3, 4)
print(id(t) == old_id)   # False ← 不是原地修改，是新对象
```

> ⚠️ **性能注意**：元组的 `+` 操作每次都创建新元组并复制所有元素，时间复杂度 O(n)。如果需要频繁拼接，应该先用列表 `append()`，最后再转成元组。

---

## 7.7 命名元组（NamedTuple）——元组的进化形态

当元组用作"记录"时，用位置索引来访问字段很容易出错。Python 提供了 `namedtuple` 来给每个位置起名字：

### 7.7.1 使用 `collections.namedtuple`（经典写法）

```python
from collections import namedtuple

# 定义一个命名元组类型
Point = namedtuple("Point", ["x", "y"])

# 创建实例
p = Point(3.0, 4.0)
print(p)         # Point(x=3.0, y=4.0)
print(p.x)       # 3.0  ← 用名字访问
print(p.y)       # 4.0
print(p[0])      # 3.0  ← 仍然支持索引

# 仍然是元组
print(isinstance(p, tuple))   # True
print(len(p))                 # 2

# 仍然不可变
# p.x = 5.0    # ❌ AttributeError
```

### 7.7.2 使用 `typing.NamedTuple`（现代写法，推荐）

```python
from typing import NamedTuple

class Student(NamedTuple):
    name: str
    age: int
    major: str
    gpa: float = 0.0    # 可以设置默认值

s = Student("Alice", 20, "CS", 3.8)
print(s)             # Student(name='Alice', age=20, major='CS', gpa=3.8)
print(s.name)        # Alice
print(s.major)       # CS

# 使用默认值
s2 = Student("Bob", 21, "Math")
print(s2.gpa)        # 0.0

# 解包依然可用
name, age, major, gpa = s
print(f"{name}, {age}岁, {major}专业, GPA={gpa}")
```

### 7.7.3 `_replace()` ——创建修改后的副本

```python
from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float

p1 = Point(3.0, 4.0)
p2 = p1._replace(x=5.0)     # 创建新对象，原对象不变
print(p1)    # Point(x=3.0, y=4.0)
print(p2)    # Point(x=5.0, y=4.0)
```

> 💡 **何时用命名元组？**
>
> - 当你需要一个轻量级的"只读记录"，但又不想定义完整的 class 时
> - 当位置索引容易出错时（第3个字段到底是年龄还是专业？）
> - 当你需要元组的不可变性和可哈希性，同时希望代码更可读时

---

## 7.8 本章操作速查表

|   操作    |      方法/语法      |        功能        |   返回值   |
| :-------: | :-----------------: | :----------------: | :--------: |
|   索引    |       `t[i]`        |  访问第 i 个元素   |   元素值   |
|   切片    |     `t[a:b:c]`      |      取子序列      |   新元组   |
|   解包    |     `a, b = t`      |   分别赋值给变量   |     —      |
| 星号解包  |     `a, *b = t`     | 收集剩余元素为列表 |     —      |
|   查找    |    `t.index(x)`     |    首次出现位置    |   索引值   |
|   计数    |    `t.count(x)`     |      出现次数      |    整数    |
| 成员测试  |      `x in t`       |      是否存在      |   布尔值   |
|   长度    |      `len(t)`       |      元素数量      |    整数    |
|   拼接    |      `t1 + t2`      |      合并元组      |   新元组   |
|   重复    |       `t * n`       |     重复 n 次      |   新元组   |
|   排序    |     `sorted(t)`     |    排序后的元素    | 新**列表** |
|   反转    |    `reversed(t)`    |     反向迭代器     |   迭代器   |
| 最大/最小 | `max(t)` / `min(t)` |        极值        |   元素值   |
|   求和    |      `sum(t)`       |   数值元组的总和   |    数值    |

> 💡 **记忆诀窍**：元组的方法只有 **2 个**——`index()` 和 `count()`。所有其他操作要么是通用函数（`len()`、`sorted()`、`min()`），要么是运算符（`+`、`*`、`in`）。

---

# 第8章 集合入门：定义、特点与典型应用场景

---

## 8.1 什么是集合？

**集合（set）** 是 Python 四大内置容器中最"数学化"的一种。它直接对应数学中的**集合**概念——一组**无序的、不重复的**元素。

你可以把它想象成一个**抽签箱**——你往里面放球，相同号码的球只会保留一个，而且你从中取球时没有固定的顺序。

从语言分类上说，集合是**可变的、无序的、元素唯一的**容器。

```python
# 集合的基本形态
numbers = {1, 2, 3, 4, 5}
names = {"Alice", "Bob", "Charlie"}
mixed = {1, "hello", 3.14, True}     # 可以混合（但需要可哈希）
empty = set()                         # ⚠️ 空集合只能这样创建！

# ⚠️ 典型陷阱
not_a_set = {}
print(type(not_a_set))   # <class 'dict'>  ← 这是空字典，不是空集合！
```

### 8.1.1 集合的核心特征

|       特征       |                    说明                    |
| :--------------: | :----------------------------------------: |
|     **无序**     |     元素没有固定位置，不支持索引、切片     |
|   **元素唯一**   |        自动去重，同一个值只出现一次        |
|     **可变**     |      可以添加、删除元素（`set` 类型）      |
| **元素须可哈希** | 元素必须是不可变类型（int, str, tuple 等） |
|     **动态**     |              大小可以随时变化              |

### 8.1.2 集合的语法细节

#### 1. 创建集合的各种方式

```python
# 方式1：花括号字面量
s1 = {1, 2, 3}

# 方式2：set() 构造器
s2 = set()                  # 空集合
s3 = set([1, 2, 3, 2, 1])   # 从列表创建，自动去重 → {1, 2, 3}
s4 = set("hello")           # 从字符串创建 → {'h', 'e', 'l', 'o'}
s5 = set(range(5))          # 从 range 创建 → {0, 1, 2, 3, 4}

# 方式3：集合推导式
s6 = {x ** 2 for x in range(1, 6)}
print(s6)    # {1, 4, 9, 16, 25}

s7 = {ch.lower() for ch in "Hello World" if ch.isalpha()}
print(s7)    # {'h', 'e', 'l', 'o', 'w', 'r', 'd'}
```

#### 2. 元素的限制——必须可哈希

```python
# ✅ 可哈希类型可以作为集合元素
s = {1, 3.14, "hello", True, (1, 2)}
print(s)

# ❌ 不可哈希类型不能作为集合元素
# {[1, 2]}           # TypeError: unhashable type: 'list'
# {{1, 2}}           # TypeError: unhashable type: 'set'
# {{"a": 1}}         # TypeError: unhashable type: 'dict'
```

> 🧠 **为什么要可哈希？**  
> 集合底层使用**哈希表（hash table）**实现，每个元素的哈希值用于确定存储位置。可变对象如果修改后哈希值改变，就会破坏哈希表的内部结构。因此 Python 强制要求集合元素必须是不可变的（可哈希的）。

#### 3. 关于 `True` 和 `1` 的特殊行为

```python
s = {1, True, 1.0}
print(s)    # {1}  ← 只剩一个元素！

# 原因：在 Python 中，1 == True == 1.0 都为 True，
# 且 hash(1) == hash(True) == hash(1.0)
# 集合认为它们是"同一个元素"

s2 = {0, False, 0.0}
print(s2)   # {0}  ← 同理，0 == False == 0.0
```

### 8.1.3 集合的"无序"到底意味着什么？

```python
# 1. 不能通过索引访问
s = {10, 20, 30}
# print(s[0])     # ❌ TypeError: 'set' object is not subscriptable

# 2. 每次打印/遍历的顺序可能不同（实际上在同一次运行中通常一致，但不保证）
print({3, 1, 2})    # 打印顺序不确定

# 3. 不能切片
# s[0:2]           # ❌ TypeError

# 4. 两个包含相同元素的集合永远相等，无论插入顺序
s1 = {1, 2, 3}
s2 = {3, 2, 1}
print(s1 == s2)    # True
```

---

## 8.2 集合与其他容器的对比定位

|         场景         |    推荐容器    |         原因         |
| :------------------: | :------------: | :------------------: |
|       需要去重       |    `set` ✅     |    自动保证唯一性    |
| 快速判断元素是否存在 |    `set` ✅     |    O(1) 成员测试     |
|  集合运算（交并差）  |    `set` ✅     | 原生支持数学集合操作 |
|   需要保持插入顺序   |     `list`     |    集合不保证顺序    |
|   需要索引访问元素   | `list`/`tuple` |    集合不支持索引    |
|     需要键值映射     |     `dict`     | 集合只存值不存键值对 |
|     需要重复元素     | `list`/`tuple` |     集合自动去重     |

> 💡 **实用原则**：当你的核心需求是**"某个值在不在？"**或者**"把重复的去掉"**时，首选 `set`。

---

## 8.3 集合的典型应用场景

### 场景1：去重

这是集合最经典的用途：

```python
# 列表去重
scores = [90, 85, 90, 78, 85, 92, 78]
unique_scores = list(set(scores))
print(unique_scores)    # [90, 85, 78, 92]（顺序不保证）

# 如果需要保持原有顺序去重（Python 3.7+）
# 利用 dict.fromkeys() 保持插入顺序
unique_ordered = list(dict.fromkeys(scores))
print(unique_ordered)   # [90, 85, 78, 92]  ← 保持原始出现顺序

# 字符串去重
word = "abracadabra"
unique_chars = set(word)
print(unique_chars)     # {'a', 'b', 'r', 'c', 'd'}
print(f"共有 {len(unique_chars)} 个不同字符")
```

### 场景2：快速成员测试

```python
# 假设有一个大型的"有效用户"名单
valid_users = {"alice", "bob", "charlie", "david", "eve"}  # 集合

username = "charlie"
if username in valid_users:     # O(1) 查找
    print(f"欢迎, {username}!")
else:
    print("用户不存在。")

# 如果用列表，username in valid_list 是 O(n) 的
# 当名单有上百万条时，性能差距是巨大的
```

### 场景3：集合运算（数学操作）

这是集合最强大的特色功能，与数学中的集合运算完全对应：

```python
# 选了不同课程的学生
math_students = {"Alice", "Bob", "Charlie", "David"}
cs_students = {"Charlie", "David", "Eve", "Frank"}

# 交集：同时选了两门课的学生
both = math_students & cs_students
print(both)    # {'Charlie', 'David'}

# 并集：选了至少一门课的学生
either = math_students | cs_students
print(either)  # {'Alice', 'Bob', 'Charlie', 'David', 'Eve', 'Frank'}

# 差集：只选了数学课的学生
math_only = math_students - cs_students
print(math_only)   # {'Alice', 'Bob'}

# 对称差集：只选了其中一门课的学生
exclusive = math_students ^ cs_students
print(exclusive)   # {'Alice', 'Bob', 'Eve', 'Frank'}
```

### 场景4：数据清洗与过滤

```python
# 从两个数据源找出共有的 ID
source_a = {101, 102, 103, 104, 105}
source_b = {103, 104, 105, 106, 107}

# 两边都有的
common = source_a & source_b
print(f"共有 ID: {common}")              # {103, 104, 105}

# 只在 A 中有的（需要补充到 B 中）
missing_in_b = source_a - source_b
print(f"B 中缺失: {missing_in_b}")       # {101, 102}

# 只在一边出现的（需要人工核查的）
to_review = source_a ^ source_b
print(f"需要核查: {to_review}")           # {101, 102, 106, 107}
```

### 场景5：词汇分析

```python
text1 = "the quick brown fox jumps over the lazy dog"
text2 = "the fast brown cat jumps under the lazy dog"

words1 = set(text1.split())
words2 = set(text2.split())

print(f"文本1的词汇: {words1}")
print(f"文本2的词汇: {words2}")
print(f"共有词汇: {words1 & words2}")
print(f"文本1独有: {words1 - words2}")
print(f"文本2独有: {words2 - words1}")
```

---

## 8.4 集合的内存模型

### 8.4.1 哈希表结构

集合底层使用**哈希表（hash table）**实现，这决定了它的性能特征：

```
集合 {3, 7, 11, 5} 的内存布局（概念图）：

哈希表（数组）:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│     │     │     │  3  │     │  5  │     │  7  │ ...
│  空 │  空 │  空 │     │  空  │     │ 空  │     │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  [0]   [1]   [2]   [3]   [4]   [5]   [6]   [7]

每个元素的存储位置由 hash(element) % 表大小 决定
```

关键性能特征：

|     操作      | 平均时间复杂度 | 最坏时间复杂度 |
| :-----------: | :------------: | :------------: |
|   `x in s`    |      O(1)      |      O(n)      |
|  `s.add(x)`   |      O(1)      |      O(n)      |
| `s.remove(x)` |      O(1)      |      O(n)      |
|   `len(s)`    |      O(1)      |      O(1)      |

> 💡 最坏情况 O(n) 只在极端哈希冲突时出现（在正常使用中几乎不会遇到）。

### 8.4.2 集合的内存开销

由于哈希表需要维护一定的"空余率"（负载因子），集合的内存占用比列表大：

```python
import sys

lst = list(range(100))
s = set(range(100))

print(sys.getsizeof(lst))   # 例如 856 字节
print(sys.getsizeof(s))     # 例如 8408 字节 ← 远大于列表

# 这是用空间换时间的典型案例
```

> ⚠️ **设计权衡**：集合用更多的内存，换来了 O(1) 的查找速度。在你需要频繁做成员测试但数据量不太大时，这个交换是非常划算的。

---

## 8.5 `frozenset`——不可变集合

Python 还提供了集合的不可变版本——`frozenset`：

```python
# 创建冻结集合
fs = frozenset([1, 2, 3, 4, 5])
print(fs)          # frozenset({1, 2, 3, 4, 5})
print(type(fs))    # <class 'frozenset'>

# 支持所有查询和集合运算
print(3 in fs)                 # True
print(fs & frozenset({3, 4, 5, 6}))   # frozenset({3, 4, 5})

# ❌ 不支持修改
# fs.add(6)        # AttributeError
# fs.remove(1)     # AttributeError

# ✅ 因为不可变，所以可哈希 → 可做字典键、集合元素
s = {frozenset({1, 2}), frozenset({3, 4})}
print(s)   # {frozenset({1, 2}), frozenset({3, 4})}

d = {frozenset({1, 2}): "pair_a"}
print(d)   # {frozenset({1, 2}): 'pair_a'}
```

|     特性     | `set` | `frozenset` |
| :----------: | :---: | :---------: |
|     可变     |   ✅   |      ❌      |
|    可哈希    |   ❌   |      ✅      |
|  可做字典键  |   ❌   |      ✅      |
| 可做集合元素 |   ❌   |      ✅      |
|   集合运算   |   ✅   |      ✅      |
|    推导式    |   ✅   |      ❌      |

> 💡 `frozenset` 之于 `set`，就如 `tuple` 之于 `list`——不可变版本。

---

## 8.6 本章小结

|    要点    |                    内容                     |
| :--------: | :-----------------------------------------: |
| 集合是什么 |  无序、可变、元素唯一、元素须可哈希的容器   |
|  创建方式  | `{1, 2}` 字面量、`set()` 构造器、集合推导式 |
|  核心特点  |    自动去重、O(1) 成员测试、原生集合运算    |
|  适用场景  |    去重、快速查找、交并差运算、数据清洗     |
|  不可变版  |      `frozenset`——可做字典键/集合元素       |

---

# 第9章 集合的基本操作：增删、集合运算与实用方法

---

## 9.1 增（添加元素）

### 9.1.1 `add()` ——添加单个元素

```python
s = {1, 2, 3}

s.add(4)
print(s)    # {1, 2, 3, 4}

# 添加已存在的元素——静默忽略，不报错
s.add(3)
print(s)    # {1, 2, 3, 4}  ← 没有变化

# 元素必须可哈希
# s.add([5, 6])   # ❌ TypeError: unhashable type: 'list'
s.add((5, 6))     # ✅ 元组可以添加
print(s)          # {1, 2, 3, 4, (5, 6)}
```

### 9.1.2 `update()` ——批量添加多个元素

```python
s = {1, 2, 3}

# 从列表添加
s.update([4, 5, 6])
print(s)    # {1, 2, 3, 4, 5, 6}

# 从多个可迭代对象添加
s.update([7, 8], (9, 10), {11, 12})
print(s)    # {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12}

# 从字符串添加（每个字符是一个元素）
s2 = set()
s2.update("hello")
print(s2)   # {'h', 'e', 'l', 'o'}
```

> 💡 `update()` 相当于集合的 `extend()`——一次性往里"倒入"另一个可迭代对象的所有元素。

---

## 9.2 删（删除元素）

### 9.2.1 四种删除方式对比

```python
s = {1, 2, 3, 4, 5}

# ===== remove()：删除指定元素，不存在则报错 =====
s.remove(3)
print(s)           # {1, 2, 4, 5}
# s.remove(99)     # ❌ KeyError: 99

# ===== discard()：删除指定元素，不存在也不报错 =====
s.discard(2)
print(s)           # {1, 4, 5}
s.discard(99)      # 什么都不发生，不报错
print(s)           # {1, 4, 5}

# ===== pop()：随机弹出一个元素并返回 =====
item = s.pop()
print(f"弹出了: {item}")   # 弹出了: 1（不确定是哪个）
print(s)                    # 剩余元素
# set().pop()              # ❌ KeyError: 'pop from an empty set'

# ===== clear()：清空整个集合 =====
s.clear()
print(s)           # set()
```

### 9.2.2 删除方式选择指南

|     方法     |     行为     | 元素不存在时 |   返回值   |    适用场景    |
| :----------: | :----------: | :----------: | :--------: | :------------: |
| `remove(x)`  |    删除 x    |  ❌ KeyError  |   `None`   | 确定元素存在时 |
| `discard(x)` |    删除 x    |   静默忽略   |   `None`   | 不确定是否存在 |
|   `pop()`    | 随机弹出一个 |  ❌ KeyError  | 被弹出元素 |  逐个消耗集合  |
|  `clear()`   |   清空全部   |      —       |   `None`   |    完全清空    |

```python
# 实际开发中的选择
s = {1, 2, 3, 4, 5}

# 推荐：不确定是否存在时用 discard
user_input = 99
s.discard(user_input)    # 安全，即使不存在也不出错

# 或者先检查再 remove
if user_input in s:
    s.remove(user_input)
```

---

## 9.3 集合运算——集合的核心能力

集合运算是 `set` 最独特、最强大的功能。Python 同时提供了**运算符语法**和**方法语法**。

### 9.3.1 四大基本运算

```python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7, 8}
```

#### 1. 交集（Intersection）：两边都有的

```python
# 运算符语法
print(A & B)                    # {4, 5}
# 方法语法
print(A.intersection(B))       # {4, 5}

# 多集合交集
C = {3, 4, 5, 6}
print(A & B & C)                # {4, 5}
print(A.intersection(B, C))    # {4, 5}
```

#### 2. 并集（Union）：至少在一边出现的

```python
print(A | B)                    # {1, 2, 3, 4, 5, 6, 7, 8}
print(A.union(B))              # {1, 2, 3, 4, 5, 6, 7, 8}

# 多集合并集
print(A | B | C)               # {1, 2, 3, 4, 5, 6, 7, 8}
```

#### 3. 差集（Difference）：只在左边有的

```python
print(A - B)                    # {1, 2, 3}   ← 在 A 中但不在 B 中
print(B - A)                    # {6, 7, 8}   ← 在 B 中但不在 A 中
print(A.difference(B))         # {1, 2, 3}

# 注意：差集不满足交换律！ A - B ≠ B - A
```

#### 4. 对称差集（Symmetric Difference）：只在一边有的

```python
print(A ^ B)                           # {1, 2, 3, 6, 7, 8}
print(A.symmetric_difference(B))      # {1, 2, 3, 6, 7, 8}

# 等价于 (A | B) - (A & B)
# 也等价于 (A - B) | (B - A)
```

### 9.3.2 运算符 vs 方法的区别

```python
A = {1, 2, 3}

# 运算符要求两边都是 set
# A & [3, 4, 5]          # ❌ TypeError

# 方法可以接受任意可迭代对象
A.intersection([3, 4, 5])   # ✅ {3}
A.union(range(4, 8))         # ✅ {1, 2, 3, 4, 5, 6, 7}
A.difference((2, 3, 4))     # ✅ {1}
```

> 💡 **经验法则**：当两边都是 `set` 时用运算符（更简洁）；当另一方是列表、元组等其他可迭代对象时用方法。

### 9.3.3 原地运算（就地修改）

上面的运算都返回**新集合**，原集合不变。如果要**原地修改**，使用对应的赋值运算符或方法：

```python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7, 8}

# ===== 原地交集 =====
s = A.copy()
s &= B                          # 等价于 s.intersection_update(B)
print(s)                         # {4, 5}

# ===== 原地并集 =====
s = A.copy()
s |= B                          # 等价于 s.update(B)
print(s)                         # {1, 2, 3, 4, 5, 6, 7, 8}

# ===== 原地差集 =====
s = A.copy()
s -= B                          # 等价于 s.difference_update(B)
print(s)                         # {1, 2, 3}

# ===== 原地对称差集 =====
s = A.copy()
s ^= B                          # 等价于 s.symmetric_difference_update(B)
print(s)                         # {1, 2, 3, 6, 7, 8}
```

### 9.3.4 集合运算速查表

|   运算   | 运算符 |           方法            |                原地版本                 |
| :------: | :----: | :-----------------------: | :-------------------------------------: |
|   交集   |  `&`   |     `.intersection()`     |     `&=` / `.intersection_update()`     |
|   并集   |  `\|`  |        `.union()`         |           `\|=` / `.update()`           |
|   差集   |  `-`   |      `.difference()`      |      `-=` / `.difference_update()`      |
| 对称差集 |  `^`   | `.symmetric_difference()` | `^=` / `.symmetric_difference_update()` |

---

## 9.4 子集与超集判断

```python
A = {1, 2, 3}
B = {1, 2, 3, 4, 5}
C = {1, 2, 3}

# ===== 子集判断 =====
print(A <= B)              # True   ← A 是 B 的子集
print(A.issubset(B))       # True
print(A <= C)              # True   ← 相等也算子集
print(A < C)               # False  ← 真子集（不能相等）
print(A < B)               # True   ← A 是 B 的真子集

# ===== 超集判断 =====
print(B >= A)              # True   ← B 是 A 的超集
print(B.issuperset(A))     # True
print(B > A)               # True   ← B 是 A 的真超集

# ===== 不相交判断 =====
X = {1, 2}
Y = {3, 4}
Z = {2, 3}
print(X.isdisjoint(Y))    # True   ← 完全没有交集
print(X.isdisjoint(Z))    # False  ← 有共同元素 2
```

---

## 9.5 遍历集合

```python
colors = {"red", "green", "blue", "yellow"}

# 直接遍历（顺序不保证）
for color in colors:
    print(color, end=" ")
print()

# 排序后遍历（如果需要固定顺序）
for color in sorted(colors):
    print(color, end=" ")
# blue green red yellow
print()

# 带编号遍历
for i, color in enumerate(sorted(colors), start=1):
    print(f"{i}. {color}")
```

> ⚠️ 集合不支持 `reversed()` ，因为它是无序的。如果需要反向输出，先 `sorted()` 再 `reversed()`。

---

## 9.6 本章操作速查表

|     操作     |     方法/语法     |       功能        |  返回值  |
| :----------: | :---------------: | :---------------: | :------: |
|   添加单个   |    `s.add(x)`     |   添加一个元素    |  `None`  |
|   批量添加   | `s.update(iter)`  |   添加多个元素    |  `None`  |
| 删除（严格） |   `s.remove(x)`   | 删除，不存在报错  |  `None`  |
| 删除（宽松） |  `s.discard(x)`   | 删除，不存在忽略  |  `None`  |
|   随机弹出   |     `s.pop()`     |   弹出一个元素    | 被弹元素 |
|     清空     |    `s.clear()`    |     删除所有      |  `None`  |
|     交集     |      `A & B`      |     共有元素      |  新集合  |
|     并集     |     `A \| B`      |     所有元素      |  新集合  |
|     差集     |      `A - B`      |   A 有 B 没有的   |  新集合  |
|   对称差集   |      `A ^ B`      |   只在一边有的    |  新集合  |
|   子集判断   |     `A <= B`      | A 是否为 B 的子集 |  布尔值  |
|   超集判断   |     `A >= B`      | A 是否为 B 的超集 |  布尔值  |
|  不相交判断  | `A.isdisjoint(B)` |   是否没有交集    |  布尔值  |
|   成员测试   |     `x in s`      |     是否存在      |  布尔值  |
|     长度     |     `len(s)`      |     元素数量      |   整数   |
|     复制     |    `s.copy()`     |      浅拷贝       |  新集合  |

---

# 第10章 字典入门：定义、特点与典型应用场景

---

## 10.1 什么是字典？

**字典（dict）** 是 Python 中最重要的数据结构之一，也是 Python 语言内部使用最广泛的容器（模块、类、实例的属性都用字典存储）。

字典存储的是**键值对（key-value pair）**。你可以把它想象成一本真正的"字典"——通过**词条（键）**快速查到**释义（值）**，而不需要从头翻到尾。

从语言分类上说，字典是**可变的映射（mutable mapping）**，它不属于序列，而是属于独立的映射类型。

```python
# 字典的基本形态
student = {"name": "Alice", "age": 20, "major": "CS"}
scores = {"math": 90, "english": 85, "physics": 78}
empty = {}    # 空字典

# 键值对的组成
# 键（key）：必须是可哈希的（不可变类型）
# 值（value）：可以是任意类型
```

### 10.1.1 字典的核心特征

|       特征       |                   说明                   |
| :--------------: | :--------------------------------------: |
|  **键值对存储**  |       每个元素由一个键和一个值组成       |
|    **键唯一**    |        同一个字典中不能有重复的键        |
|   **值可重复**   |          不同键可以对应相同的值          |
|     **可变**     |             可以增删改键值对             |
|  **键须可哈希**  | 键必须是不可变类型（str, int, tuple 等） |
| **保持插入顺序** | Python 3.7+ 保证字典按插入顺序维护键值对 |
|  **O(1) 查找**   |          通过键查找值的速度极快          |

### 10.1.2 字典的语法细节

#### 1. 创建字典的多种方式

```python
# 方式1：花括号 + 冒号
d1 = {"name": "Alice", "age": 20}

# 方式2：dict() 构造器 + 关键字参数
d2 = dict(name="Alice", age=20)          # 键自动变成字符串
print(d2)    # {'name': 'Alice', 'age': 20}

# 方式3：dict() + 键值对列表
d3 = dict([("name", "Alice"), ("age", 20)])

# 方式4：dict() + zip
keys = ["name", "age", "major"]
values = ["Alice", 20, "CS"]
d4 = dict(zip(keys, values))
print(d4)    # {'name': 'Alice', 'age': 20, 'major': 'CS'}

# 方式5：dict.fromkeys() —— 用相同值初始化多个键
d5 = dict.fromkeys(["a", "b", "c"], 0)
print(d5)    # {'a': 0, 'b': 0, 'c': 0}

d6 = dict.fromkeys("abc")     # 默认值为 None
print(d6)    # {'a': None, 'b': None, 'c': None}

# ⚠️ 谨慎：可变默认值会被所有键共享同一个对象
d_shared = dict.fromkeys(["x", "y"], [])
d_shared["x"].append(1)
print(d_shared)  # {'x': [1], 'y': [1]}

# ✅ 更推荐：为每个键单独创建新对象
d_independent = {k: [] for k in ["x", "y"]}
d_independent["x"].append(1)
print(d_independent)  # {'x': [1], 'y': []}

# 方式6：字典推导式
d7 = {x: x ** 2 for x in range(1, 6)}
print(d7)    # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

#### 2. 键的限制

```python
# ✅ 可哈希类型可以做键
d = {
    "name": "Alice",       # str
    42: "the answer",      # int
    3.14: "pi",            # float
    True: "yes",           # bool
    (1, 2): "point",       # tuple（元素全部可哈希时）
}

# ❌ 不可哈希类型不能做键
# {[1, 2]: "bad"}          # TypeError: unhashable type: 'list'
# {{1}: "bad"}             # TypeError: unhashable type: 'set'

# ⚠️ True 和 1 是同一个键！
d = {True: "yes", 1: "one"}
print(d)    # {True: 'one'}  ← 第二次赋值覆盖了第一次，但保留了首次的键
```

#### 3. 值可以是任意类型

```python
complex_dict = {
    "string": "hello",
    "number": 42,
    "list": [1, 2, 3],
    "dict": {"nested": True},
    "set": {1, 2, 3},
    "none": None,
    "function": print,        # 甚至可以存函数
}
```

---

## 10.2 字典与其他容器的对比定位

|        场景         | 推荐容器 |          原因          |
| :-----------------: | :------: | :--------------------: |
| 通过"名称/标签"查找 | `dict` ✅ |  键值映射，O(1) 查找   |
|   存储结构化记录    | `dict` ✅ |  字段名作键，清晰明了  |
|    配置/选项管理    | `dict` ✅ |      键名即选项名      |
|      统计/计数      | `dict` ✅ |      值作为计数器      |
|  需要有序、可索引   |  `list`  | 字典的"顺序"不用于索引 |
|      需要去重       |  `set`   |  字典用于去重过于笨重  |
|    数据固定不变     | `tuple`  |      字典是可变的      |

---

## 10.3 字典的典型应用场景

### 场景1：存储结构化记录

```python
# 一条学生记录
student = {
    "name": "Alice",
    "age": 20,
    "major": "Computer Science",
    "gpa": 3.8,
    "courses": ["Math", "Physics", "Programming"],
}

print(f"{student['name']} 的 GPA 是 {student['gpa']}")
print(f"选修课程: {', '.join(student['courses'])}")
```

### 场景2：统计/计数

```python
# 统计字符出现次数
text = "hello world"
counter = {}
for ch in text:
    counter[ch] = counter.get(ch, 0) + 1
print(counter)
# {'h': 1, 'e': 1, 'l': 3, 'o': 2, ' ': 1, 'w': 1, 'r': 1, 'd': 1}

# Python 3.1+ 提供了专用工具
from collections import Counter
counter = Counter(text)
print(counter)             # Counter({'l': 3, 'o': 2, ...})
print(counter.most_common(3))  # [('l', 3), ('o', 2), ('h', 1)]
```

### 场景3：配置管理

```python
config = {
    "host": "localhost",
    "port": 5432,
    "database": "mydb",
    "debug": True,
    "max_connections": 100,
}

# 使用配置
print(f"连接到 {config['host']}:{config['port']}/{config['database']}")
if config["debug"]:
    print("调试模式已开启")
```

### 场景4：映射/转换表

```python
# 成绩等级映射
def get_grade(score: int) -> str:
    grade_map = {
        range(90, 101): "A",
        range(80, 90): "B",
        range(70, 80): "C",
        range(60, 70): "D",
    }
    for score_range, grade in grade_map.items():
        if score in score_range:
            return grade
    return "F"

# 更简单的映射
day_names = {
    1: "星期一", 2: "星期二", 3: "星期三",
    4: "星期四", 5: "星期五", 6: "星期六", 7: "星期日",
}
print(day_names[3])    # 星期三
```

### 场景5：分组数据

```python
# 按首字母分组
words = ["apple", "banana", "avocado", "blueberry", "cherry", "apricot"]
groups = {}
for word in words:
    key = word[0]
    if key not in groups:
        groups[key] = []
    groups[key].append(word)

print(groups)
# {'a': ['apple', 'avocado', 'apricot'], 'b': ['banana', 'blueberry'], 'c': ['cherry']}

# 更优雅的写法：用 setdefault
groups2 = {}
for word in words:
    groups2.setdefault(word[0], []).append(word)
```

---

## 10.4 字典的内存模型

### 10.4.1 哈希表结构

字典和集合一样基于**哈希表**，但每个槽位存储的是**键值对**而不只是键：

```
字典 {"name": "Alice", "age": 20} 的概念布局：

哈希表:
┌──────┬──────┬──────┬──────────────────────┬──────┬─────────────────┐
│  空  │  空  │  空  │ key="name",val="Alice"│  空  │ key="age",val=20│
└──────┴──────┴──────┴──────────────────────┴──────┴─────────────────┘
  [0]    [1]    [2]          [3]               [4]         [5]
```

### 10.4.2 Python 3.6+ 的紧凑字典

从 CPython 3.6 开始（Python 3.7 正式成为语言规范），字典使用了一种更紧凑的实现：

```
旧版字典：稀疏哈希表，浪费大量空间

新版字典（3.6+）：
  indices（稀疏索引数组）:  [_, _, 0, _, 1, _, _, _]
  entries（紧凑条目数组）:  [(hash_a, "name", "Alice"),
                             (hash_b, "age", 20)]
```

好处：

1. **内存更省**：紧凑存储减少了约 20-25% 的内存占用
2. **保持插入顺序**：遍历 entries 数组就是插入顺序
3. **遍历更快**：紧凑数组的缓存命中率更高

---

## 10.5 本章小结

|    要点    |                         内容                         |
| :--------: | :--------------------------------------------------: |
| 字典是什么 |        可变的键值对映射容器，键唯一且须可哈希        |
|  创建方式  | `{}` 字面量、`dict()` 构造器、字典推导式、`fromkeys` |
|  核心特点  |     O(1) 键查找、保持插入顺序（3.7+）、值无限制      |
|  适用场景  |    结构化记录、统计计数、配置管理、映射转换、分组    |

---

# 第11章 字典的基本操作：增删改查与常用方法

---

## 11.1 访问值（查）

### 11.1.1 方括号访问 `d[key]`

最直接的访问方式，但键不存在时会报错：

```python
student = {"name": "Alice", "age": 20, "major": "CS"}

print(student["name"])     # Alice
print(student["age"])      # 20
# print(student["gpa"])    # ❌ KeyError: 'gpa'
```

### 11.1.2 `get()` 方法——安全访问

`get()` 是日常开发中**最推荐**的访问方式，键不存在时返回默认值而不是报错：

```python
student = {"name": "Alice", "age": 20, "major": "CS"}

# get(key) —— 键不存在返回 None
print(student.get("name"))      # Alice
print(student.get("gpa"))       # None（不报错）

# get(key, default) —— 键不存在返回自定义默认值
print(student.get("gpa", 0.0))       # 0.0
print(student.get("name", "未知"))    # Alice ← 键存在时返回真实值，忽略默认值
```

> 💡 **选择建议**：
>
> - 确定键一定存在时，用 `d[key]`（更简洁，且键不存在时报错有助于发现 bug）
> - 键可能不存在时，用 `d.get(key, default)`（更安全）

### 11.1.3 `keys()`、`values()`、`items()` ——视图对象

这三个方法返回的是**视图对象（view object）**，而不是列表。视图对象是字典的"活窗口"——字典变化时，视图会自动反映：

```python
d = {"name": "Alice", "age": 20, "major": "CS"}

# ===== keys() —— 所有键的视图 =====
print(d.keys())      # dict_keys(['name', 'age', 'major'])

# ===== values() —— 所有值的视图 =====
print(d.values())    # dict_values(['Alice', 20, 'CS'])

# ===== items() —— 所有键值对的视图 =====
print(d.items())     # dict_items([('name', 'Alice'), ('age', 20), ('major', 'CS')])

# 视图是动态的——字典变化会反映到视图中
keys_view = d.keys()
print("age" in keys_view)    # True
d["gpa"] = 3.8
print(keys_view)             # dict_keys(['name', 'age', 'major', 'gpa']) ← 自动更新

# 视图可以用于遍历、成员测试、集合运算
# 转换为列表
keys_list = list(d.keys())
print(keys_list)    # ['name', 'age', 'major', 'gpa']
```

> 🧠 **视图对象 vs 列表**：视图对象不复制数据，因此内存开销极小。只有在需要索引或切片时才需要转为列表。

### 11.1.4 遍历字典（最常用的操作之一）

```python
config = {"host": "localhost", "port": 5432, "debug": True}

# 方式1：遍历键（默认行为）
for key in config:
    print(f"{key} = {config[key]}")

# 方式2：遍历键值对（最推荐——更清晰高效）
for key, value in config.items():
    print(f"{key} = {value}")

# 方式3：只遍历值
for value in config.values():
    print(value)

# 方式4：带编号遍历
for i, (key, value) in enumerate(config.items(), start=1):
    print(f"{i}. {key}: {value}")
```

输出（方式4）：

```
1. host: localhost
2. port: 5432
3. debug: True
```

---

## 11.2 增与改（添加 / 修改键值对）

在字典中，"增"和"改"使用的是同一种语法——赋值。键不存在就是添加，键已存在就是修改。

### 11.2.1 方括号赋值

```python
d = {"name": "Alice", "age": 20}

# 修改已有键的值
d["age"] = 21
print(d)    # {'name': 'Alice', 'age': 21}

# 添加新键值对
d["major"] = "CS"
print(d)    # {'name': 'Alice', 'age': 21, 'major': 'CS'}
```

### 11.2.2 `update()` ——批量添加/更新

```python
d = {"name": "Alice", "age": 20}

# 用另一个字典更新
d.update({"age": 21, "major": "CS", "gpa": 3.8})
print(d)    # {'name': 'Alice', 'age': 21, 'major': 'CS', 'gpa': 3.8}

# 用关键字参数更新
d.update(age=22, city="Beijing")
print(d)    # {'name': 'Alice', 'age': 22, 'major': 'CS', 'gpa': 3.8, 'city': 'Beijing'}

# 用键值对列表更新
d.update([("phone", "12345"), ("email", "alice@example.com")])
```

> 💡 `update()` 的规则：键已存在就覆盖，键不存在就添加——与 `d[key] = value` 逻辑一致。

### 11.2.3 `setdefault()` ——"不存在才添加"

`setdefault()` 是一个非常实用但常被忽视的方法。它的逻辑是：**键存在就返回已有值，键不存在就设置默认值并返回**。

```python
d = {"name": "Alice", "age": 20}

# 键存在：返回已有值，不做修改
result = d.setdefault("name", "Bob")
print(result)    # Alice ← 返回已有值
print(d)         # {'name': 'Alice', 'age': 20} ← 没有变化

# 键不存在：设置默认值并返回
result = d.setdefault("major", "CS")
print(result)    # CS ← 返回新设置的值
print(d)         # {'name': 'Alice', 'age': 20, 'major': 'CS'}

# 实际应用：分组数据
words = ["apple", "banana", "avocado", "blueberry", "cherry"]
groups = {}
for word in words:
    groups.setdefault(word[0], []).append(word)
print(groups)
# {'a': ['apple', 'avocado'], 'b': ['banana', 'blueberry'], 'c': ['cherry']}
```

> 🧠 `setdefault()` 的等价逻辑：
>
> ```python
> # d.setdefault(key, default) 等价于：
> if key not in d:
>  d[key] = default
> return d[key]
> ```

### 11.2.4 `|` 合并运算符（Python 3.9+）

Python 3.9 引入了字典的合并运算符，语法非常优雅：

```python
defaults = {"theme": "light", "font_size": 14, "language": "zh"}
user_prefs = {"font_size": 18, "language": "en"}

# | 合并（返回新字典，右边优先）
merged = defaults | user_prefs
print(merged)    # {'theme': 'light', 'font_size': 18, 'language': 'en'}

# |= 原地合并
defaults |= user_prefs
print(defaults)  # {'theme': 'light', 'font_size': 18, 'language': 'en'}
```

> 💡 `|` 运算符相当于 `{**d1, **d2}` 的语法糖，右边字典的值会覆盖左边的同名键。

---

## 11.3 删（删除键值对）

### 11.3.1 四种删除方式

```python
d = {"name": "Alice", "age": 20, "major": "CS", "gpa": 3.8}

# ===== del d[key]：删除指定键值对，不存在报错 =====
del d["gpa"]
print(d)              # {'name': 'Alice', 'age': 20, 'major': 'CS'}
# del d["xyz"]        # ❌ KeyError: 'xyz'

# ===== pop(key)：删除并返回值，不存在报错 =====
age = d.pop("age")
print(age)            # 20
print(d)              # {'name': 'Alice', 'major': 'CS'}

# pop(key, default)：不存在时返回默认值，不报错
result = d.pop("gpa", None)
print(result)         # None

# ===== popitem()：删除并返回最后插入的键值对（Python 3.7+） =====
d = {"a": 1, "b": 2, "c": 3}
last = d.popitem()
print(last)           # ('c', 3)  ← 最后插入的
print(d)              # {'a': 1, 'b': 2}
# {}.popitem()        # ❌ KeyError: 'popitem(): dictionary is empty'

# ===== clear()：清空字典 =====
d.clear()
print(d)              # {}
```

### 11.3.2 删除方式选择指南

|      方法       |     行为     | 键不存在时 |     返回值     |     适用场景     |
| :-------------: | :----------: | :--------: | :------------: | :--------------: |
|  `del d[key]`   |  删除键值对  | ❌ KeyError |       —        |   确定键存在时   |
|  `d.pop(key)`   | 删除并返回值 | ❌ KeyError |       值       | 需要使用被删的值 |
| `d.pop(key, d)` | 删除并返回值 | 返回默认值 |   值或默认值   |  不确定是否存在  |
|  `d.popitem()`  | 弹出最后一对 | ❌ KeyError | `(key, value)` |   逐个消耗字典   |
|   `d.clear()`   |   清空全部   |     —      |     `None`     |     完全清空     |

---

## 11.4 成员测试与查找

```python
d = {"name": "Alice", "age": 20, "major": "CS"}

# ===== in / not in —— 检查键是否存在 =====
print("name" in d)         # True
print("gpa" in d)          # False
print("gpa" not in d)      # True

# ⚠️ in 检查的是键，不是值！
print("Alice" in d)        # False ← "Alice" 是值，不是键
print("Alice" in d.values())  # True ← 这样才能检查值

# ===== 检查键值对是否存在 =====
print(("name", "Alice") in d.items())     # True
print(("name", "Bob") in d.items())       # False
```

> 💡 **性能**：`key in d` 和 `key in d.keys()` 都是 O(1)；`value in d.values()` 是 O(n)。

---

## 11.5 字典推导式（Dictionary Comprehension）

字典推导式是创建和转换字典的最优雅方式：

### 11.5.1 基本语法

```python
# 基本形式：{key_expr: value_expr for variable in iterable}

# 平方映射
squares = {x: x ** 2 for x in range(1, 6)}
print(squares)    # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# 带条件筛选
even_squares = {x: x ** 2 for x in range(1, 11) if x % 2 == 0}
print(even_squares)    # {2: 4, 4: 16, 6: 36, 8: 64, 10: 100}
```

### 11.5.2 实用示例

```python
# 1. 键值互换
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(inverted)    # {1: 'a', 2: 'b', 3: 'c'}
# ⚠️ 注意：如果原字典有重复值，互换后会丢失条目

# 2. 筛选字典
config = {"host": "localhost", "port": 5432, "debug": True, "verbose": False}
# 只保留值为 True 的布尔配置
active = {k: v for k, v in config.items() if v is True}
print(active)    # {'debug': True}

# 3. 转换值
prices_usd = {"apple": 1.5, "banana": 0.8, "orange": 2.0}
rate = 7.2
prices_cny = {fruit: round(price * rate, 2) for fruit, price in prices_usd.items()}
print(prices_cny)    # {'apple': 10.8, 'banana': 5.76, 'orange': 14.4}

# 4. 从两个列表构建字典
names = ["Alice", "Bob", "Charlie"]
ages = [20, 22, 21]
roster = {name: age for name, age in zip(names, ages)}
print(roster)    # {'Alice': 20, 'Bob': 22, 'Charlie': 21}
```

---

## 11.6 字典的嵌套

字典的值可以是任何类型，包括其他字典——这使得嵌套字典成为表示复杂结构化数据的常用方式：

### 11.6.1 嵌套字典的访问

```python
# 学校数据
school = {
    "CS": {
        "Alice": {"age": 20, "gpa": 3.8},
        "Bob": {"age": 21, "gpa": 3.5},
    },
    "Math": {
        "Charlie": {"age": 22, "gpa": 3.9},
    },
}

# 访问嵌套数据——逐层用 []
print(school["CS"]["Alice"]["gpa"])      # 3.8

# 安全访问（避免 KeyError）
dept = school.get("Physics")             # None（不报错）
# 多层安全访问
gpa = school.get("CS", {}).get("Alice", {}).get("gpa", 0.0)
print(gpa)    # 3.8
```

### 11.6.2 嵌套字典的修改

```python
school = {
    "CS": {
        "Alice": {"age": 20, "gpa": 3.8},
    },
}

# 修改嵌套值
school["CS"]["Alice"]["gpa"] = 3.9

# 添加嵌套记录
school["CS"]["David"] = {"age": 19, "gpa": 3.6}

# 添加新部门
school["Physics"] = {
    "Eve": {"age": 21, "gpa": 3.7},
}

print(school)
```

### 11.6.3 嵌套字典的遍历

```python
school = {
    "CS": {"Alice": 3.8, "Bob": 3.5},
    "Math": {"Charlie": 3.9, "David": 3.2},
}

# 双层遍历
for dept, students in school.items():
    print(f"\n--- {dept} 系 ---")
    for name, gpa in students.items():
        print(f"  {name}: GPA = {gpa}")
```

输出：

```
--- CS 系 ---
  Alice: GPA = 3.8
  Bob: GPA = 3.5

--- Math 系 ---
  Charlie: GPA = 3.9
  David: GPA = 3.2
```

---

## 11.7 字典的拷贝

### 11.7.1 浅拷贝

```python
original = {"name": "Alice", "scores": [90, 85, 78]}

# 三种浅拷贝方式
copy1 = original.copy()
copy2 = dict(original)
copy3 = {**original}       # 解包语法

# 浅拷贝的特性：外层独立，内层共享
copy1["name"] = "Bob"
print(original["name"])    # Alice ← 外层不受影响

copy1["scores"].append(92)
print(original["scores"])  # [90, 85, 78, 92] ← 内层还是共享的！
```

### 11.7.2 深拷贝

```python
import copy

original = {"name": "Alice", "scores": [90, 85, 78]}
deep = copy.deepcopy(original)

deep["scores"].append(92)
print(original["scores"])  # [90, 85, 78] ← 完全独立！
print(deep["scores"])      # [90, 85, 78, 92]
```

> 💡 **何时用深拷贝？**  
> 当字典的值中包含可变对象（列表、字典、集合等），并且你需要一个完全独立的副本时。

---

## 11.8 实用进阶方法

### 11.8.1 `collections.defaultdict`

`defaultdict` 在访问不存在的键时，自动调用一个工厂函数创建默认值：

```python
from collections import defaultdict

# 默认值为 0 的计数器
counter = defaultdict(int)     # int() 返回 0
for ch in "hello world":
    counter[ch] += 1           # 键不存在时自动创建值为 0，然后 +1
print(dict(counter))           # {'h': 1, 'e': 1, 'l': 3, ...}

# 默认值为空列表的分组器
groups = defaultdict(list)     # list() 返回 []
words = ["apple", "banana", "avocado", "blueberry", "cherry"]
for word in words:
    groups[word[0]].append(word)
print(dict(groups))
# {'a': ['apple', 'avocado'], 'b': ['banana', 'blueberry'], 'c': ['cherry']}

# 默认值为空集合
unique_tags = defaultdict(set)
data = [("user1", "python"), ("user1", "java"), ("user2", "python"), ("user1", "python")]
for user, tag in data:
    unique_tags[user].add(tag)
print(dict(unique_tags))
# {'user1': {'python', 'java'}, 'user2': {'python'}}
```

> 💡 `defaultdict` 在做计数、分组、多值映射等任务时，能大幅简化代码——它消除了"先检查键是否存在"的样板逻辑。

### 11.8.2 `collections.Counter`

`Counter` 是专门用于计数的字典子类：

```python
from collections import Counter

# 从可迭代对象创建
c = Counter("abracadabra")
print(c)    # Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

# 最常见的 N 个元素
print(c.most_common(3))   # [('a', 5), ('b', 2), ('r', 2)]

# Counter 之间可以做算术运算
c1 = Counter(a=3, b=1)
c2 = Counter(a=1, b=2, c=3)
print(c1 + c2)    # Counter({'a': 4, 'c': 3, 'b': 3})
print(c1 - c2)    # Counter({'a': 2})  ← 只保留正数
print(c1 & c2)    # Counter({'a': 1, 'b': 1})  ← 取最小值
print(c1 | c2)    # Counter({'a': 3, 'c': 3, 'b': 2})  ← 取最大值
```

---

## 11.9 本章操作速查表

> 说明：本表中出现的 \|（如 d1 \| d2）只是为了在 Markdown 表格里正确显示竖线，属于排版转义；Python 真实语法是 d1 | d2（不写反斜杠）。

|     操作     |       方法/语法        |          功能          |    返回值    |
| :----------: | :--------------------: | :--------------------: | :----------: |
|    读取值    |        `d[key]`        |    取值，不存在报错    |      值      |
|   安全读取   |   `d.get(key, def)`    |  取值，不存在返回默认  |   值或默认   |
|  添加/修改   |     `d[key] = val`     | 键存在覆盖，不存在新增 |      —       |
|   批量更新   |    `d.update(...)`     |    合并字典/键值对     |    `None`    |
|   条件添加   | `d.setdefault(k, def)` |      不存在才设置      |      值      |
| 合并（3.9+） |       `d1 \| d2`       |   创建合并后的新字典   |    新字典    |
| 删除（严格） |      `del d[key]`      |    删除，不存在报错    |      —       |
|  删除并返回  |      `d.pop(key)`      |      删除并返回值      |      值      |
|   安全弹出   |   `d.pop(key, def)`    |    不存在返回默认值    |   值或默认   |
| 弹出最后一对 |     `d.popitem()`      |  删除最后插入的键值对  | `(key, val)` |
|     清空     |      `d.clear()`       |     删除所有键值对     |    `None`    |
|    获取键    |       `d.keys()`       |      所有键的视图      |   视图对象   |
|    获取值    |      `d.values()`      |      所有值的视图      |   视图对象   |
|  获取键值对  |      `d.items()`       |    所有键值对的视图    |   视图对象   |
|   成员测试   |       `key in d`       |       键是否存在       |    布尔值    |
|     长度     |        `len(d)`        |       键值对数量       |     整数     |
|     拷贝     |       `d.copy()`       |         浅拷贝         |    新字典    |

---

# 第12章 三大容器综合对比与选型指南

---

## 12.1 核心特性对比总览

|       特性       |    元组 `tuple`     |   集合 `set`    |     字典 `dict`      |
| :--------------: | :-----------------: | :-------------: | :------------------: |
|     **类型**     |     不可变序列      |  可变无序容器   |     可变映射容器     |
|     **有序**     |   ✅ 保持插入顺序    |  ❌ 无固定顺序   | ✅ 保持插入顺序(3.7+) |
|     **可变**     |          ❌          |        ✅        |          ✅           |
|    **可索引**    |      ✅ `t[0]`       |        ❌        | ✅ `d["key"]`（按键） |
|    **可切片**    |     ✅ `t[1:3]`      |        ❌        |          ❌           |
|   **元素唯一**   |          ❌          |        ✅        |   键唯一，值可重复   |
|    **可哈希**    | ✅（元素全可哈希时） |        ❌        |          ❌           |
|  **可做字典键**  | ✅（元素全可哈希时） |        ❌        |          ❌           |
| **可做集合元素** | ✅（元素全可哈希时） |        ❌        |          ❌           |
|    **推导式**    |          ❌          | ✅ `{x for ...}` |  ✅ `{k: v for ...}`  |
| **`in` 复杂度**  |        O(n)         |      O(1)       |    O(1)（检查键）    |
|   **存储内容**   |         值          |    值(唯一)     |        键值对        |

---

## 12.2 操作能力对比

### 12.2.1 增（添加元素）

|   操作   |         元组          |       集合       |        字典        |
| :------: | :-------------------: | :--------------: | :----------------: |
| 添加单个 |      ❌（不可变）      |    `s.add(x)`    |   `d[key] = val`   |
| 批量添加 |      ❌（不可变）      | `s.update(iter)` | `d.update(other)`  |
| 拼接新建 | ✅ `t1 + t2`（新对象） |    `s1 \| s2`    | `d1 \| d2`（3.9+） |

### 12.2.2 删（删除元素）

|    操作    |    元组     |      集合      |     字典     |
| :--------: | :---------: | :------------: | :----------: |
|  按值删除  | ❌（不可变） | `s.remove(x)`  |      —       |
| 安全按值删 | ❌（不可变） | `s.discard(x)` |      —       |
|  按键删除  |      —      |       —        | `del d[key]` |
| 弹出并返回 | ❌（不可变） |   `s.pop()`    | `d.pop(key)` |
|    清空    | ❌（不可变） |  `s.clear()`   | `d.clear()`  |

### 12.2.3 改（修改元素）

|     操作      |    元组     |    集合     |       字典        |
| :-----------: | :---------: | :---------: | :---------------: |
| 按索引/键修改 | ❌（不可变） | ❌（无索引） | ✅ `d[key] = val`  |
|   批量修改    | ❌（不可变） |      ❌      | ✅ `d.update(...)` |

### 12.2.4 查（查找元素）

|   操作   |      元组       |      集合       |          字典          |
| :------: | :-------------: | :-------------: | :--------------------: |
| 成员测试 | ✅ `x in t` O(n) | ✅ `x in s` O(1) |   ✅ `key in d` O(1)    |
| 查找位置 | ✅ `t.index(x)`  |    ❌（无序）    |           —            |
| 统计次数 | ✅ `t.count(x)`  |    —（唯一）    |           —            |
|  获取值  |    ✅ `t[i]`     |        ❌        | ✅ `d[key]` / `d.get()` |
| 获取长度 |   ✅ `len(t)`    |   ✅ `len(s)`    |       ✅ `len(d)`       |

---

## 12.3 选型决策流程

当你面对一个具体问题，不确定应该用哪种容器时，可以按照以下流程来判断：

```
你的数据需求是什么？
│
├─ 需要"键→值"的映射关系吗？
│   └─ ✅ 是 ──→ 用 dict
│       ├─ 需要可变吗？ → dict
│       └─ 不需要可变？ → MappingProxyType (只读视图)
│
├─ 需要去重 / 集合运算（交并差）吗？
│   └─ ✅ 是 ──→ 用 set
│       ├─ 需要可变吗？ → set
│       └─ 不需要可变？ → frozenset
│
├─ 数据创建后需要修改吗？
│   ├─ ✅ 需要修改 ──→ 用 list
│   └─ ❌ 不需要修改 ──→ 用 tuple
│       └─ 需要做字典键/集合元素？ → tuple
│
└─ 不确定？──→ 先用 list，后续按需改
```

### 12.3.1 各容器的"最佳场景"

|  容器   | 最适合的场景 |               典型代码               |
| :-----: | :----------: | :----------------------------------: |
| `tuple` | 函数多返回值 |            `return x, y`             |
| `tuple` |  坐标/记录   |         `point = (3.0, 4.0)`         |
| `tuple` |    字典键    |         `d[(x, y)] = value`          |
|  `set`  |     去重     |      `unique = list(set(data))`      |
|  `set`  |   快速查找   |         `if x in valid_set`          |
|  `set`  |   集合运算   |       `common = set_a & set_b`       |
| `dict`  |  结构化数据  |    `student = {"name": "Alice"}`     |
| `dict`  |  计数/统计   | `counter[x] = counter.get(x, 0) + 1` |
| `dict`  |  分组/索引   |      `groups[key].append(item)`      |
| `dict`  |   配置管理   |      `config = {"debug": True}`      |
| `dict`  | 缓存/备忘录  |        `cache[args] = result`        |

---

## 12.4 常见误用与陷阱

### 陷阱1：该用元组时用了列表

```python
# ❌ 不好的做法：函数坐标用列表
def get_position():
    return [3.0, 4.0]    # 调用者可能意外修改返回值

# ✅ 正确做法：用元组表示固定结构的记录
def get_position():
    return (3.0, 4.0)    # 不可变，更安全
```

### 陷阱2：用列表做成员测试

```python
# ❌ 性能差：对大列表做 in 测试
valid_ids = list(range(100_000))    # 示例：10 万个 id
if user_id in valid_ids:           # O(n)，慢
    pass

# ✅ 正确做法：转为集合
valid_ids = set(range(100_000))     # set
if user_id in valid_ids:           # O(1)，快
    pass
```

### 陷阱3：用列表做去重

```python
# ❌ 手动去重：费力且效率低
unique = []
for x in data:
    if x not in unique:    # O(n) × n 次 = O(n²)
        unique.append(x)

# ✅ 正确做法：用集合
unique = list(set(data))   # O(n)
# 如果要保持顺序：
unique = list(dict.fromkeys(data))   # O(n)
```

### 陷阱4：空集合用 `{}`

```python
# ❌ 错误
empty_set = {}
print(type(empty_set))    # <class 'dict'>  ← 是字典！

# ✅ 正确
empty_set = set()
print(type(empty_set))    # <class 'set'>
```

### 陷阱5：单元素元组忘加逗号

```python
# ❌ 错误
t = (42)
print(type(t))    # <class 'int'>

# ✅ 正确
t = (42,)
print(type(t))    # <class 'tuple'>
```

### 陷阱6：遍历字典时修改字典

```python
d = {"a": 1, "b": 2, "c": 3}

# ❌ 遍历中直接修改——RuntimeError
# for key in d:
#     if d[key] < 3:
#         del d[key]    # RuntimeError: dictionary changed size during iteration

# ✅ 正确做法1：遍历副本
for key in list(d.keys()):
    if d[key] < 3:
        del d[key]
print(d)    # {'c': 3}

# ✅ 正确做法2：用字典推导式创建新字典
d = {"a": 1, "b": 2, "c": 3}
d = {k: v for k, v in d.items() if v >= 3}
print(d)    # {'c': 3}
```

### 陷阱7：可变对象作为字典键

```python
# ❌ 列表不能做键
# d = {[1, 2]: "bad"}    # TypeError: unhashable type: 'list'

# ✅ 转成元组做键
d = {(1, 2): "good"}

# ⚠️ 包含列表的元组也不行
# d = {(1, [2, 3]): "bad"}   # TypeError: unhashable type: 'list'
```

---

## 12.5 性能特征综合对比

|      操作      | `tuple`  |      `list`      |       `set`        |       `dict`       |
| :------------: | :------: | :--------------: | :----------------: | :----------------: |
|    创建速度    | 通常较快 |     通常较快     |      通常较慢      |      通常较慢      |
|    内存占用    | 通常更省 | 中等（动态扩容） | 通常较高（哈希表） | 通常较高（哈希表） |
|  `x in` 查找   |   O(n)   |       O(n)       |        O(1)        |        O(1)        |
| 索引访问 `[i]` |   O(1)   |       O(1)       |       不支持       |      O(1)按键      |
|    遍历速度    |    快    |        快        |        较快        |        较快        |
|    添加元素    |  不支持  |      O(1)*       |       O(1)*        |       O(1)*        |
|    删除元素    |  不支持  |       O(n)       |       O(1)*        |       O(1)*        |

> *均摊时间复杂度；哈希容器在极端冲突下最坏可退化到 O(n)
>
> ⚠️ 上表为工程实践中的常见趋势，具体表现会受 Python 版本、数据分布、哈希冲突和运行环境影响。

---

## 12.6 容器之间的转换关系图

```
       list()               tuple()
  tuple ←────────→ list ←────────→ tuple
    │                │                │
    │ set()          │ set()          │ set()
    ↓                ↓                ↓
  set  ←───────── set ──────────→  set
    │                                │
    │ frozenset()                    │ frozenset()
    ↓                                ↓
frozenset                        frozenset

dict 的转换比较特殊：
  list(d)        → 键列表
  list(d.items()) → 键值对列表 [(k,v), ...]
  dict(pairs)    → 从键值对列表创建字典
  set(d)         → 键集合
```

```python
# 转换示意
d = {"a": 1, "b": 2, "c": 3}

# dict → 其他
list(d)           # ['a', 'b', 'c']        ← 只取键
tuple(d)          # ('a', 'b', 'c')        ← 只取键
set(d)            # {'a', 'b', 'c'}        ← 只取键
list(d.values())  # [1, 2, 3]             ← 取值
list(d.items())   # [('a', 1), ('b', 2), ('c', 3)]  ← 取键值对

# 其他 → dict（需要键值对结构）
dict([("a", 1), ("b", 2)])     # ✅
dict(zip(["a", "b"], [1, 2]))  # ✅
# dict([1, 2, 3])              # ❌ 无法解释为键值对
```

---

## 12.7 不可变版本对照表

Python 为部分可变容器提供了对应的不可变版本：

|  可变版本   | 不可变版本  |       关系描述        |
| :---------: | :---------: | :-------------------: |
|   `list`    |   `tuple`   | 有序序列的可变/不可变 |
|    `set`    | `frozenset` |   集合的可变/不可变   |
|   `dict`    |     —★      | 没有内置的不可变字典  |
| `bytearray` |   `bytes`   | 字节序列的可变/不可变 |

> ★ 标准库中 `types.MappingProxyType` 可以创建字典的**只读视图**，但它本身不可哈希，不能作为字典键或集合元素。

```python
from types import MappingProxyType

d = {"name": "Alice", "age": 20}
proxy = MappingProxyType(d)

print(proxy["name"])        # Alice
# proxy["name"] = "Bob"    # ❌ TypeError: 不支持修改

# 但原字典修改会反映到 proxy 中
d["age"] = 21
print(proxy["age"])         # 21 ← proxy 是视图，不是副本
```

---

## 12.8 本章总结：一句话选型指南

| 如果你需要...                      |         用这个         |
| :--------------------------------- | :--------------------: |
| 函数返回多个值                     |        `tuple`         |
| 一组不可修改的数据（坐标、配置）   |        `tuple`         |
| 把复合数据用作字典键或集合元素     |        `tuple`         |
| 去掉重复元素                       |         `set`          |
| 快速判断"某个值是否存在"           |         `set`          |
| 对数据做交集、并集、差集运算       |         `set`          |
| 通过名称/标签查找对应的数据        |         `dict`         |
| 存储结构化记录（姓名、年龄、成绩） |         `dict`         |
| 统计元素出现次数                   |   `dict` / `Counter`   |
| 按条件分组数据                     | `dict` / `defaultdict` |

> 💡 **终极口诀**：
>
> - **不变选元组，去重选集合，映射选字典，其余选列表。**

---

以上就是**元组、集合、字典**三大容器的完整教程，涵盖：

- **第6章**：元组入门——定义、不可变性、与列表的对比、典型场景、内存模型
- **第7章**：元组操作——索引、切片、解包（核心操作）、只读查询、命名元组
- **第8章**：集合入门——定义、唯一性、哈希要求、典型场景、frozenset
- **第9章**：集合操作——增删、四大集合运算（交并差对称差）、子集超集判断
- **第10章**：字典入门——定义、键值对、典型场景、内存模型
- **第11章**：字典操作——增删改查、视图对象、推导式、嵌套、defaultdict/Counter
- **第12章**：综合对比——特性对比、选型决策、常见陷阱、性能特征、转换关系
