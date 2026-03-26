# 第六编 四大容器的横向比较与通用方法

---

# 第22章 list、tuple、dict、set的联系、差异与选型原则

## 22.1 为什么需要横向比较？

在前面的学习中，我们已经分别认识了 Python 的四大内置容器：`list`（列表）、`tuple`（元组）、`dict`（字典）和 `set`（集合）。每种容器都有自己的语法、方法和适用场景。但在实际编程中，面对一个具体问题，你往往需要做出选择：**用哪种容器来存储和组织数据？**

这个选择并不是随意的。选错容器，轻则代码冗余、可读性差，重则性能低下、逻辑出错。因此，我们需要把四种容器放在一起，从多个维度进行系统比较，建立**选型思维**。

---

## 22.2 四大容器的本质定位

在开始详细比较之前，先建立一个直觉认知——每种容器的"角色定位"是什么：

| 容器    | 一句话定位               | 现实类比                        |
| ------- | ------------------------ | ------------------------------- |
| `list`  | 有序的、可变的元素序列   | 一份可以随时修改的待办事项清单  |
| `tuple` | 有序的、不可变的元素序列 | 一张已打印出的成绩单，不能涂改  |
| `dict`  | 键值对的映射集合         | 一本字典/电话簿，通过名字查号码 |
| `set`   | 无序的、不重复的元素集合 | 一个抽签箱，每个号码只出现一次  |

---

## 22.3 维度一：有序性（Ordering）

### 22.3.1 什么是"有序"？

"有序"指的是：**容器的遍历顺序有稳定规则；对于序列类型，还可以按位置（索引）访问元素**。

```python
# list 是有序的
fruits = ["apple", "banana", "cherry"]
print(fruits[0])  # apple —— 可以通过索引访问
print(fruits[2])  # cherry

# tuple 也是有序的
coordinates = (3, 5, 7)
print(coordinates[1])  # 5

# dict 在 Python 3.7+ 保证插入顺序
person = {"name": "Alice", "age": 25, "city": "Beijing"}
for key in person:
    print(key, end=" ")  # name age city —— 按插入顺序

# set 是无序的
colors = {"red", "green", "blue"}
print(colors)  # 输出顺序不确定，可能是 {'blue', 'red', 'green'}
# colors[0]  # TypeError! set 不支持索引
```

### 22.3.2 有序性对比表

| 特性         | list | tuple | dict             | set |
| ------------ | ---- | ----- | ---------------- | --- |
| 保持插入顺序 | ✅    | ✅     | ✅（Python 3.7+） | ❌   |
| 支持索引访问 | ✅    | ✅     | ❌（通过键访问）  | ❌   |
| 支持切片操作 | ✅    | ✅     | ❌                | ❌   |

### 22.3.3 关于 dict 的有序性

这里需要特别说明：`dict` 的"有序"和 `list` 的"有序"含义不同。

- `list`/`tuple` 的有序 = **可以用整数索引**访问元素，如 `lst[0]`、`lst[-1]`、`lst[1:3]`。
- `dict` 的有序 = **遍历时按照插入的先后顺序**输出键值对，但**不能用整数索引**访问。

```python
d = {"a": 1, "b": 2, "c": 3}
# d[0]  # KeyError! 这里的 0 会被当作键，不是索引
d["a"]  # 1 —— 必须用键来访问

# 如果你真的想按顺序取第 n 个元素：
keys = list(d.keys())
print(keys[0])  # 'a' —— 先转成 list 再用索引
```

---

## 22.4 维度二：可变性（Mutability）

### 22.4.1 可变 vs 不可变

"可变"意味着**创建之后还可以修改（增、删、改元素）**。"不可变"意味着**一旦创建，内容就固定了**。

```python
# list 是可变的
lst = [1, 2, 3]
lst[0] = 10        # 修改元素 ✅
lst.append(4)      # 添加元素 ✅
lst.remove(2)      # 删除元素 ✅
print(lst)          # [10, 3, 4]

# tuple 是不可变的
tup = (1, 2, 3)
# tup[0] = 10      # TypeError: 'tuple' object does not support item assignment
# tup.append(4)    # AttributeError: 'tuple' object has no attribute 'append'
print(tup)          # (1, 2, 3) —— 永远不变

# dict 是可变的
d = {"a": 1}
d["b"] = 2          # 添加键值对 ✅
d["a"] = 100        # 修改值 ✅
del d["b"]          # 删除键值对 ✅
print(d)            # {'a': 100}

# set 是可变的
s = {1, 2, 3}
s.add(4)            # 添加元素 ✅
s.discard(2)        # 删除元素 ✅
print(s)            # {1, 3, 4}
```

### 22.4.2 不可变的意义

为什么需要不可变容器？初学者常常觉得"能改不是更方便吗？"其实不可变有很多好处：

**1. 安全性**：不可变对象不会被意外修改

```python
# 假设你把一个配置传给函数
config = ("localhost", 8080, "production")

def connect(cfg):
    # 即使函数内部想修改 cfg，也做不到
    # cfg[0] = "hacker-server"  # TypeError!
    host, port, mode = cfg
    print(f"Connecting to {host}:{port} in {mode} mode")

connect(config)
# 因为 tuple 不能被原地改写，所以 connect() 无法偷偷修改这个配置对象
```

**2. 可哈希（Hashable）**：**哈希值稳定的对象**可以作为 `dict` 的键或 `set` 的元素。初学阶段可以先记成：**常见的不可变对象通常可哈希，但要以实际类型规则为准**。

```python
# tuple 可以做 dict 的键
locations = {
    (35.68, 139.69): "Tokyo",
    (39.90, 116.40): "Beijing",
    (48.85, 2.35): "Paris",
}
print(locations[(39.90, 116.40)])  # Beijing

# list 不能做 dict 的键
# bad = {[35.68, 139.69]: "Tokyo"}  # TypeError: unhashable type: 'list'

# tuple 可以做 set 的元素
point_set = {(1, 2), (3, 4), (1, 2)}
print(point_set)  # {(1, 2), (3, 4)} —— 自动去重
```

**3. 性能**：不可变对象的哈希值可以缓存，访问更快；Python 内部对 tuple 还有创建优化。

### 22.4.3 可变性对比表

| 特性                              | list  | tuple               | dict | set       |
| --------------------------------- | ----- | ------------------- | ---- | --------- |
| 可变                              | ✅     | ❌                   | ✅    | ✅         |
| 可哈希（可做 dict 键 / set 元素） | ❌     | ✅（元素也需可哈希） | ❌    | ❌         |
| 有不可变对应版本                  | tuple | —                   | —    | frozenset |

> **补充**：`frozenset` 是 `set` 的不可变版本，就像 `tuple` 是 `list` 的不可变版本一样。

```python
fs = frozenset([1, 2, 3])
# fs.add(4)  # AttributeError! 不可变
print(fs)  # frozenset({1, 2, 3})

# frozenset 可以作为 dict 的键
d = {frozenset({1, 2}): "pair", frozenset({3}): "single"}
```

---

## 22.5 维度三：元素唯一性（Uniqueness）

### 22.5.1 哪些容器保证唯一性？

```python
# list 允许重复
lst = [1, 2, 2, 3, 3, 3]
print(lst)  # [1, 2, 2, 3, 3, 3] —— 重复元素全部保留

# tuple 允许重复
tup = (1, 2, 2, 3, 3, 3)
print(tup)  # (1, 2, 2, 3, 3, 3)

# set 自动去重
s = {1, 2, 2, 3, 3, 3}
print(s)  # {1, 2, 3} —— 重复的自动合并

# dict 的键是唯一的（但值可以重复）
d = {"a": 1, "b": 2, "a": 3}  # 键 "a" 重复了
print(d)  # {'a': 3, 'b': 2} —— 后面的覆盖前面的
```

### 22.5.2 唯一性对比

| 容器  | 元素是否唯一     | 说明                     |
| ----- | ---------------- | ------------------------ |
| list  | 不要求唯一       | 可存储任意数量的重复元素 |
| tuple | 不要求唯一       | 同上                     |
| dict  | 键唯一，值不唯一 | 重复键会被覆盖           |
| set   | 元素唯一         | 天然去重                 |

---

## 22.6 维度四：内部实现与时间复杂度

### 22.6.1 底层数据结构

理解底层实现有助于理解性能表现：

| 容器  | 底层实现                 | 核心底层特征与内部机制         |
| ----- | ------------------------ | ------------------------------ |
| list  | **动态对象指针数组**       | 连续一块内存存储指针，支持索引 O(1) 访问。伴有类似 C++ `vector` 的 capacity 预分配扩容机制 |
| tuple | **固定大小的对象指针数组** | 内存固定不扩容，占用更低。CPython 还具有缓存优化（Free List）加速小元组生成 |
| dict  | **优化后的哈希表**         | Py3.6+ 分离了“稀疏哈希索引表”和“键值对密集数组”，不仅降低了 20%~30% 的内存消耗，还副带维持了 O(1) 平均查找并实现了“插入有序” |
| set   | **纯稀疏哈希表**           | 结构传统，只存键。由于没有引入 dict 的双表紧凑结构，不保证有序，但它的存在测试是所有核心容器中性能最极致的（哈希探测） |

### 22.6.2 常见操作的时间复杂度

| 操作             | list      | tuple    | dict           | set            |
| ---------------- | --------- | -------- | -------------- | -------------- |
| 按索引/键访问    | O(1)      | O(1)     | 平均 O(1)      | —              |
| 查找元素（`in`） | **O(n)**  | **O(n)** | **平均 O(1)**  | **平均 O(1)**  |
| 末尾添加         | O(1) 均摊 | —        | 平均 O(1) 均摊 | 平均 O(1) 均摊 |
| 头部插入         | **O(n)**  | —        | —              | —              |
| 删除元素         | O(n)      | —        | 平均 O(1)      | 平均 O(1)      |
| 遍历             | O(n)      | O(n)     | O(n)           | O(n)           |

> **复杂度备注**：`dict`/`set` 的 O(1) 是**平均复杂度**。在哈希冲突严重等极端情况下，单次操作可能退化到 O(n)。教学和工程实践中通常按“平均 O(1)”来选型。

> **关键结论**：如果你频繁执行 `in` 判断（检查某个元素是否存在），使用 `set` 或 `dict` 通常比 `list` 快得多。

```python
import time

# 创建大数据
big_list = list(range(1_000_000))
big_set = set(range(1_000_000))

# 在 list 中查找（慢）
start = time.perf_counter()
999_999 in big_list
print(f"list 查找耗时: {time.perf_counter() - start:.6f}s")

# 在 set 中查找（快）
start = time.perf_counter()
999_999 in big_set
print(f"set  查找耗时: {time.perf_counter() - start:.6f}s")

# 典型输出：
# list 查找耗时: 0.012345s
# set  查找耗时: 0.000001s
# 在该规模下通常明显更快；具体倍数取决于数据分布、Python 版本与运行环境
```

---

## 22.7 维度五：对元素类型的要求

### 22.7.1 各容器对元素的限制

```python
# list 和 tuple：几乎无限制，元素可以是任意类型
lst = [1, "hello", 3.14, [1, 2], {"a": 1}, None]  # 全部合法

# dict 的键必须是可哈希的（更准确地说：哈希值必须稳定）
d = {
    1: "int key",           # ✅ int 可哈希
    "hello": "str key",     # ✅ str 可哈希
    (1, 2): "tuple key",    # ✅ tuple 可哈希（元素也可哈希时）
    # [1, 2]: "list key",   # ❌ TypeError: unhashable type: 'list'
    # {1, 2}: "set key",    # ❌ TypeError: unhashable type: 'set'
}

# dict 的值没有限制
d2 = {"data": [1, 2, 3], "nested": {"a": 1}}  # ✅

# set 的元素必须是可哈希的（更准确地说：哈希值必须稳定）
s = {1, "hello", (1, 2)}   # ✅
# s2 = {[1, 2]}            # ❌ TypeError: unhashable type: 'list'
# s3 = {{"a": 1}}          # ❌ TypeError: unhashable type: 'dict'
```

### 22.7.2 "可哈希"到底是什么意思？

简单理解：**可哈希**指对象能够计算出稳定的哈希值，并且在参与相等比较时保持一致。  
在 Python 内置类型里，你可以先把它记成：**可变容器通常不可哈希；很多不可变对象通常可哈希**。但两者**不是完全等价**。

Python 中常见的可哈希类型包括：
- 所有不可变的基本类型：`int`、`float`、`str`、`bool`、`None`
- `tuple`（前提是其内部元素也都是可哈希的）
- `frozenset`
- 用户自定义的对象（默认可哈希，除非你重写了 `__eq__` 但没重写 `__hash__`）

常见的不可哈希类型：
- `list`、`dict`、`set`（都是可变容器）

```python
# 验证可哈希性
print(hash(42))          # 可以 → int 可哈希
print(hash("hello"))     # 可以 → str 可哈希
print(hash((1, 2, 3)))   # 可以 → tuple 可哈希
# print(hash([1, 2, 3])) # TypeError → list 不可哈希

# 注意：tuple 内含不可哈希元素时，tuple 自身也不可哈希
# print(hash(([1, 2], 3)))  # TypeError: unhashable type: 'list'

# frozenset 也是可哈希的（前提是其中元素本身可哈希）
print(hash(frozenset({1, 2, 3})))
```

---

## 22.8 维度六：创建方式汇总

```python
# ========== list ==========
list1 = [1, 2, 3]                  # 字面量
list2 = list()                      # 空列表
list3 = list("hello")              # 从可迭代对象创建: ['h', 'e', 'l', 'l', 'o']
list4 = list(range(5))             # [0, 1, 2, 3, 4]
list5 = [x**2 for x in range(5)]  # 列表推导式: [0, 1, 4, 9, 16]

# ========== tuple ==========
tup1 = (1, 2, 3)                   # 字面量
tup2 = tuple()                     # 空元组
tup3 = (42,)                       # 单元素元组（注意逗号！）
tup4 = tuple("hello")             # 从可迭代对象创建: ('h', 'e', 'l', 'l', 'o')
tup5 = tuple(x**2 for x in range(5))  # 从生成器: (0, 1, 4, 9, 16)

# ========== dict ==========
dict1 = {"a": 1, "b": 2}           # 字面量
dict2 = dict()                     # 空字典
dict3 = dict(a=1, b=2)            # 关键字参数
dict4 = dict([("a", 1), ("b", 2)])# 从键值对列表
dict5 = dict(zip("abc", [1, 2, 3]))  # 从 zip 对象
dict6 = {x: x**2 for x in range(5)}  # 字典推导式: {0:0, 1:1, 2:4, 3:9, 4:16}
dict7 = dict.fromkeys("abc", 0)    # {'a': 0, 'b': 0, 'c': 0}

# 注意：如果默认值是可变对象，它会被所有键共享
dict8 = dict.fromkeys(["x", "y"], [])
dict8["x"].append(1)
print(dict8)  # {'x': [1], 'y': [1]}

# ========== set ==========
set1 = {1, 2, 3}                   # 字面量
set2 = set()                       # 空集合（注意：{} 是空 dict，不是空 set！）
set3 = set("hello")                # 从可迭代对象: {'h', 'e', 'l', 'o'}
set4 = set(range(5))               # {0, 1, 2, 3, 4}
set5 = {x**2 for x in range(5)}   # 集合推导式: {0, 1, 4, 9, 16}
```

> **补充提醒**：`dict.fromkeys(keys, value)` 中的 `value` 并不会为每个键各复制一份；如果它是列表、字典、集合这类可变对象，所有键会共享同一个默认值对象。

> **常见陷阱**：`{}` 创建的是空字典，不是空集合！空集合必须用 `set()` 创建。

```python
empty_dict = {}
empty_set = set()
print(type(empty_dict))  # <class 'dict'>
print(type(empty_set))   # <class 'set'>
```

---

## 22.9 全方位对比总表

| 维度       | list     | tuple                | dict             | set       |
| ---------- | -------- | -------------------- | ---------------- | --------- |
| 有序性     | 有序     | 有序                 | 插入有序（3.7+） | 无序      |
| 可变性     | 可变     | 不可变               | 可变             | 可变      |
| 元素唯一性 | 不要求   | 不要求               | 键唯一           | 唯一      |
| 索引访问   | ✅        | ✅                    | 按键访问         | ❌         |
| 切片       | ✅        | ✅                    | ❌                | ❌         |
| 查找效率   | O(n)     | O(n)                 | 平均 O(1)        | 平均 O(1) |
| 元素要求   | 任意     | 任意                 | 键可哈希         | 可哈希    |
| 底层结构   | 动态数组 | 连续存储的不可变序列 | 哈希表           | 哈希表    |
| 推导式     | ✅        | ❌（用生成器）        | ✅                | ✅         |
| 不可变版本 | tuple    | —                    | —                | frozenset |
| 字面量语法 | `[]`     | `()`                 | `{k:v}`          | `{v}`     |

---

## 22.10 选型原则：什么场景用什么容器？

### 22.10.1 选型决策树

面对一个数据存储需求，按以下顺序思考：

```
1. 数据是"键→值"的映射关系吗？
   ├── 是 → 用 dict
   └── 否 → 继续

2. 需要去重 / 做集合运算（交集、并集）吗？
   ├── 是 → 用 set
   └── 否 → 继续

3. 数据创建后是否需要修改？
   ├── 不需要修改 → 用 tuple
   └── 需要修改 → 用 list
```

### 22.10.2 典型场景推荐

| 使用场景                     | 推荐容器              | 原因                     |
| ---------------------------- | --------------------- | ------------------------ |
| 存储一组有序数据，需要增删改 | `list`                | 可变 + 有序 + 支持索引   |
| 函数返回多个值               | `tuple`               | 不可变 + 轻量，语义清晰  |
| 不可修改的配置/常量          | `tuple`               | 不可变，防止意外修改     |
| 存储键值对映射（如用户信息） | `dict`                | 键值访问，平均 O(1) 查找 |
| 统计词频/计数                | `dict`                | 键 → 出现次数的映射      |
| 去重                         | `set`                 | 天然唯一性               |
| 判断元素是否存在（大数据量） | `set`                 | 平均 O(1) 查找           |
| 集合运算（交集、并集、差集） | `set`                 | 内置集合运算操作符       |
| 作为 dict 的键               | `tuple` / `frozenset` | 必须可哈希               |
| 存储数据库查询结果           | `list` of `dict`      | 每行一条记录             |
| 存储坐标点                   | `tuple`               | `(x, y)` 语义清晰        |
| 记录历史操作 / 日志          | `list`                | 按时间顺序追加           |

### 22.10.3 实战选型练习

```python
# 场景1：存储学生成绩，需要按名字查分数
# 分析：名字 → 分数 = 键值映射 → dict
scores = {"Alice": 95, "Bob": 87, "Charlie": 92}

# 场景2：一组不重复的标签
# 分析：不重复 → set
tags = {"python", "tutorial", "beginner"}

# 场景3：RGB颜色值
# 分析：固定3个值，不需要修改 → tuple
color = (255, 128, 0)

# 场景4：待办事项列表，会不断添加和删除
# 分析：需要增删改 + 有序 → list
todos = ["写报告", "买菜", "锻炼"]

# 场景5：判断某个用户ID是否在黑名单中（百万级数据）
# 分析：只需要判断是否存在 + 大数据量 → set
def load_blacklist_ids():
    # 教学示例中用一个小样本模拟“从文件/数据库加载”
    return [1001, 1002, 1003, 2001]

blacklist = set(load_blacklist_ids())  # 用 set 存储，成员测试平均 O(1)

# 场景6：存储棋盘上已占用的格子坐标
# 分析：坐标是(行,列)对 + 需要快速判断是否占用 + 不重复 → set of tuple
occupied = {(0, 0), (1, 2), (3, 4)}
if (2, 2) not in occupied:
    print("这个格子可以下棋")
```

---

## 22.11 容器之间的相互转换

四大容器之间可以互相转换，这在实际编程中非常常用：

```python
# === 从 list 转换 ===
lst = [1, 2, 2, 3, 3, 3]
print(tuple(lst))   # (1, 2, 2, 3, 3, 3) → 保留顺序和重复
print(set(lst))     # {1, 2, 3}           → 去重，失去顺序
# list → dict 需要元素是二元组
pairs = [("a", 1), ("b", 2)]
print(dict(pairs))  # {'a': 1, 'b': 2}

# === 从 tuple 转换 ===
tup = (4, 5, 5, 6)
print(list(tup))    # [4, 5, 5, 6]
print(set(tup))     # {4, 5, 6}

# === 从 set 转换 ===
s = {10, 20, 30}
print(list(s))      # [10, 20, 30]（顺序不确定）
print(tuple(s))     # (10, 20, 30)（顺序不确定）
print(sorted(s))    # [10, 20, 30]（排序后的 list）

# === 从 dict 转换 ===
d = {"x": 1, "y": 2, "z": 3}
print(list(d))         # ['x', 'y', 'z']       → 默认取键
print(list(d.values()))# [1, 2, 3]             → 取值
print(list(d.items())) # [('x',1),('y',2),('z',3)] → 取键值对
print(set(d))          # {'x', 'y', 'z'}       → 键的集合

# === 经典用法：list 去重并保持顺序 ===
data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
# 方法1：用 set 去重（不保持顺序）
unique_unordered = list(set(data))
print(unique_unordered)  # 顺序不确定

# 方法2：用 dict.fromkeys 去重（保持首次出现的顺序）
unique_ordered = list(dict.fromkeys(data))
print(unique_ordered)  # [3, 1, 4, 5, 9, 2, 6] —— 保持原始顺序！
```

---

## 22.12 本章小结

1. **四大容器各有定位**：list 是万能工具，tuple 是只读保护，dict 是键值映射，set 是去重和集合运算。
2. 选型的核心维度是：**有序性、可变性、唯一性、查找效率**。
3. **选型决策顺序**：先问是否需要映射（dict），再问是否需要去重（set），最后看是否需要修改（list vs tuple）。
4. 理解底层实现（数组 vs 哈希表）有助于正确预估性能。
5. 容器之间可以互相转换，要注意转换可能带来的信息变化（如去重、失序）。

---

# 第23章 容器相关内置函数：len、sum、min、max、sorted、reversed、enumerate、zip、all、any

## 23.1 为什么要学容器相关的内置函数？

Python 提供了大量内置函数（built-in functions），其中有一批函数专门用于操作容器。这些函数的共同特点是：**接受可迭代对象（iterable）作为参数**，这意味着它们可以同时作用于 `list`、`tuple`、`dict`、`set`，甚至生成器和字符串。

掌握这些函数，可以大幅减少你"手写循环"的次数，让代码更加简洁和 Pythonic。

---

## 23.2 len()——获取容器的长度

### 23.2.1 基本用法

`len()` 返回容器中元素的个数。

```python
print(len([1, 2, 3]))           # 3
print(len((4, 5, 6)))           # 3
print(len({"a": 1, "b": 2}))   # 2 —— dict 返回键的个数
print(len({10, 20, 30}))        # 3
print(len("hello"))             # 5 —— 字符串也可以
print(len(range(100)))          # 100 —— range 对象也可以
```

### 23.2.2 注意事项

```python
# 对内置序列与映射（list/tuple/dict/set/str/range 等），len() 通常是 O(1)
# 它们内部会维护长度信息，所以 len() 不需要遍历整个容器
big_list = list(range(10_000_000))
print(len(big_list))  # 瞬间返回 10000000

# 嵌套容器只计算最外层
nested = [[1, 2], [3, 4], [5, 6]]
print(len(nested))  # 3（3个子列表），不是6

# 空容器
print(len([]))   # 0
print(len({}))   # 0
print(len(set()))# 0

my_list = []

# 利用 len() 判断容器是否为空（但更 Pythonic 的写法见下方）
if len(my_list) == 0:
    print("列表为空")

# 更 Pythonic 的写法：直接将容器用于布尔判断
# 空容器为 False（"falsy"），非空容器为 True（"truthy"）
if not my_list:
    print("列表为空")
```

---

## 23.3 sum()——求和

### 23.3.1 基本用法

```python
print(sum([1, 2, 3, 4, 5]))      # 15
print(sum((10, 20, 30)))           # 60
print(sum({100, 200, 300}))        # 600
print(sum(range(1, 101)))          # 5050 (1+2+...+100)
```

### 23.3.2 start 参数

`sum()` 有一个可选的 `start` 参数，表示初始值（默认为 0）。

```python
# sum(iterable, start=0) 的计算过程：start + iterable[0] + iterable[1] + ...
print(sum([1, 2, 3], 10))         # 10 + 1 + 2 + 3 = 16
print(sum([1, 2, 3], 100))        # 100 + 1 + 2 + 3 = 106

# 实用场景：合并多个列表
lists = [[1, 2], [3, 4], [5, 6]]
print(sum(lists, []))  # [] + [1,2] + [3,4] + [5,6] = [1, 2, 3, 4, 5, 6]
# 注意：这种方式效率不高（每次 + 都会创建新列表），大量数据时推荐用：
import itertools
flat = list(itertools.chain.from_iterable(lists))  # 更高效
```

### 23.3.3 sum() 的限制

```python
# sum() 不能用于字符串拼接
# sum(["hello", " ", "world"], "")  # TypeError: sum() can't sum strings
# 正确做法：
print("".join(["hello", " ", "world"]))  # "hello world"

# sum() 只能处理数值类型（或实现了 __add__ 的对象）
# sum(["a", "b"])  # TypeError

# 对 dict 使用 sum() 会对键求和（如果键是数值）
print(sum({1: "a", 2: "b", 3: "c"}))  # 1 + 2 + 3 = 6

# 对 dict 的值求和
d = {"math": 90, "english": 85, "physics": 92}
print(sum(d.values()))  # 267
```

---

## 23.4 min() 和 max()——最小值与最大值

### 23.4.1 基本用法

```python
# 对容器求最值
print(min([3, 1, 4, 1, 5, 9]))  # 1
print(max([3, 1, 4, 1, 5, 9]))  # 9

print(min((10, 20, 5)))          # 5
print(max({100, 200, 50}))       # 200

# 也可以直接传多个参数（不传容器）
print(min(3, 1, 4))              # 1
print(max(3, 1, 4))              # 4

# 对字符串求最值（按字典序/Unicode码点比较）
print(min("hello"))              # 'e'
print(max("hello"))              # 'o'
print(min("apple", "banana", "cherry"))  # 'apple'

# 对 dict 求最值：默认作用于键
d = {"c": 3, "a": 1, "b": 2}
print(min(d))         # 'a'（键的最小值）
print(max(d))         # 'c'（键的最大值）
print(min(d.values()))# 1（值的最小值）
print(max(d.values()))# 3（值的最大值）
```

### 23.4.2 key 参数：自定义比较规则

`key` 参数是 `min()` 和 `max()` 最强大的功能。它接受一个函数，该函数作用于每个元素，返回用于比较的值。

```python
# 找到绝对值最小的数
numbers = [-10, 3, -5, 8, -1]
print(min(numbers, key=abs))  # -1（abs(-1)=1 是最小的绝对值）
print(max(numbers, key=abs))  # -10（abs(-10)=10 是最大的绝对值）

# 找到最长的字符串
words = ["apple", "hi", "watermelon", "cat"]
print(max(words, key=len))    # 'watermelon'
print(min(words, key=len))    # 'hi'

# 在字典列表中找到年龄最大的人
people = [
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": 25},
    {"name": "Charlie", "age": 35},
]
oldest = max(people, key=lambda p: p["age"])
print(oldest)  # {'name': 'Charlie', 'age': 35}

youngest = min(people, key=lambda p: p["age"])
print(youngest)  # {'name': 'Bob', 'age': 25}

# 在 dict 中找到值最大的键
scores = {"Alice": 95, "Bob": 87, "Charlie": 92}
best_student = max(scores, key=scores.get)
print(best_student)  # 'Alice'
# 解释：max 遍历 scores 的键，对每个键调用 scores.get(key) 获取值，
#        以值作为比较依据，返回值最大的那个键
```

### 23.4.3 default 参数：处理空容器

```python
# 对空容器求 min/max 会报错
# min([])  # ValueError: min() arg is an empty sequence

# 使用 default 参数避免报错
print(min([], default=0))       # 0
print(max([], default="无数据"))  # '无数据'

# 实际应用：安全地取最值
def safe_max(data):
    return max(data, default=None)

print(safe_max([1, 2, 3]))  # 3
print(safe_max([]))          # None
```

---

## 23.5 sorted()——排序

### 23.5.1 基本用法

`sorted()` 返回一个**新的已排序列表**（不修改原容器）。

```python
# 对 list 排序
nums = [3, 1, 4, 1, 5, 9, 2, 6]
result = sorted(nums)
print(result)   # [1, 1, 2, 3, 4, 5, 6, 9]
print(nums)     # [3, 1, 4, 1, 5, 9, 2, 6] —— 原列表不变！

# 对 tuple 排序（返回的仍然是 list）
print(sorted((5, 3, 8, 1)))   # [1, 3, 5, 8]

# 对 set 排序（结果是 list，赋予了顺序）
print(sorted({30, 10, 20}))   # [10, 20, 30]

# 对 str 排序（每个字符作为元素）
print(sorted("python"))       # ['h', 'n', 'o', 'p', 't', 'y']

# 对 dict 排序（默认按键排序）
d = {"banana": 2, "apple": 5, "cherry": 1}
print(sorted(d))              # ['apple', 'banana', 'cherry']
```

### 23.5.2 reverse 参数：降序排序

```python
nums = [3, 1, 4, 1, 5, 9]
print(sorted(nums, reverse=True))  # [9, 5, 4, 3, 1, 1]
```

### 23.5.3 key 参数：自定义排序规则

```python
# 按字符串长度排序
words = ["banana", "apple", "cherry", "date"]
print(sorted(words, key=len))
# ['date', 'apple', 'banana', 'cherry']

# 按绝对值排序
nums = [-5, 3, -2, 8, -1]
print(sorted(nums, key=abs))
# [-1, -2, 3, -5, 8]

# 对字典列表排序
students = [
    {"name": "Alice", "grade": 88},
    {"name": "Bob", "grade": 95},
    {"name": "Charlie", "grade": 72},
]
# 按成绩升序
by_grade = sorted(students, key=lambda s: s["grade"])
for s in by_grade:
    print(f"{s['name']}: {s['grade']}")
# Charlie: 72
# Alice: 88
# Bob: 95

# 按成绩降序
by_grade_desc = sorted(students, key=lambda s: s["grade"], reverse=True)

# 多级排序：先按年级升序，相同年级按名字字母序
students2 = [
    {"name": "Charlie", "grade": 88},
    {"name": "Alice", "grade": 88},
    {"name": "Bob", "grade": 95},
]
result = sorted(students2, key=lambda s: (s["grade"], s["name"]))
# 先按 grade 排，grade 相同的按 name 排
for s in result:
    print(f"{s['name']}: {s['grade']}")
# Alice: 88
# Charlie: 88
# Bob: 95
```

### 23.5.4 sorted() vs list.sort()

| 特性     | `sorted()`     | `list.sort()`      |
| -------- | -------------- | ------------------ |
| 作用对象 | 任何可迭代对象 | 仅 list            |
| 返回值   | **新列表**     | `None`（原地修改） |
| 原容器   | 不变           | 被修改             |
| 内存     | 需要额外空间   | 原地操作，省内存   |

```python
# sorted() —— 创建新列表
original = [3, 1, 2]
new_list = sorted(original)
print(original)  # [3, 1, 2] 不变
print(new_list)  # [1, 2, 3] 新列表

# list.sort() —— 原地修改
original = [3, 1, 2]
result = original.sort()  # 原地排序
print(original)  # [1, 2, 3] 已被修改
print(result)    # None —— sort() 返回 None！
```

---

## 23.6 reversed()——反转

### 23.6.1 基本用法

`reversed()` 返回一个**反转迭代器**（不是列表），需要用 `list()` 或循环来消费。

```python
# 反转 list
nums = [1, 2, 3, 4, 5]
rev = reversed(nums)
print(rev)           # <list_reverseiterator object at 0x...> → 是迭代器！
print(list(rev))     # [5, 4, 3, 2, 1]
print(nums)          # [1, 2, 3, 4, 5] → 原列表不变

# 反转 tuple
print(tuple(reversed((1, 2, 3))))  # (3, 2, 1)

# 反转 str
print("".join(reversed("hello")))  # "olleh"

# 反转 range
print(list(reversed(range(5))))    # [4, 3, 2, 1, 0]

# 直接用于 for 循环（最常见用法）
for i in reversed(range(5)):
    print(i, end=" ")  # 4 3 2 1 0
print()
```

### 23.6.2 reversed() 的限制

```python
# reversed() 只能用于序列类型或实现了 __reversed__ 的对象
# 不能用于 set 和 dict（虽然 dict 在 Python 3.8+ 支持了 reversed）

# set 不支持 reversed
# reversed({1, 2, 3})  # TypeError: argument to reversed() must be a sequence

# dict 在 Python 3.8+ 支持 reversed
d = {"a": 1, "b": 2, "c": 3}
print(list(reversed(d)))  # ['c', 'b', 'a']  (Python 3.8+)
```

### 23.6.3 reversed() vs 切片反转

```python
lst = [1, 2, 3, 4, 5]

# 切片反转：创建新列表，占用内存
rev1 = lst[::-1]         # [5, 4, 3, 2, 1]

# reversed()：返回迭代器，惰性求值
rev2 = reversed(lst)     # 迭代器，用到时才取值

# 如果只是遍历，reversed() 更省内存
for x in reversed(lst):  # 不需要创建新列表
    print(x)
```

---

## 23.7 enumerate()——带索引遍历

### 23.7.1 问题引入

遍历容器时，如果你需要同时获得**元素的索引和值**，该怎么办？

```python
# 笨办法：手动维护索引变量
names = ["Alice", "Bob", "Charlie"]
i = 0
for name in names:
    print(f"第{i}个人: {name}")
    i += 1

# 也不太好的办法：用 range(len(...))
for i in range(len(names)):
    print(f"第{i}个人: {names[i]}")
```

这两种写法都很"不 Pythonic"。Python 的优雅写法是用 `enumerate()`。

### 23.7.2 基本用法

```python
names = ["Alice", "Bob", "Charlie"]

# enumerate() 返回 (索引, 元素) 的迭代器
for index, name in enumerate(names):
    print(f"第{index}个人: {name}")
# 第0个人: Alice
# 第1个人: Bob
# 第2个人: Charlie
```

### 23.7.3 start 参数：指定起始编号

```python
# 默认从 0 开始编号
for i, name in enumerate(["Alice", "Bob", "Charlie"]):
    print(f"{i}: {name}")
# 0: Alice
# 1: Bob
# 2: Charlie

# 从 1 开始编号
for i, name in enumerate(["Alice", "Bob", "Charlie"], start=1):
    print(f"第{i}名: {name}")
# 第1名: Alice
# 第2名: Bob
# 第3名: Charlie

# 从 100 开始编号
for i, name in enumerate(["Alice", "Bob"], start=100):
    print(f"编号{i}: {name}")
# 编号100: Alice
# 编号101: Bob
```

### 23.7.4 enumerate() 作用于不同容器

```python
# 对字符串
for i, ch in enumerate("hello"):
    print(f"位置{i}: '{ch}'")

# 对 tuple
for i, val in enumerate((10, 20, 30)):
    print(f"[{i}] = {val}")

# 对 dict（遍历的是键）
d = {"a": 1, "b": 2, "c": 3}
for i, key in enumerate(d):
    print(f"第{i}个键: {key}")

# 对 dict 的键值对
for i, (key, val) in enumerate(d.items()):
    print(f"[{i}] {key} -> {val}")
# [0] a -> 1
# [1] b -> 2
# [2] c -> 3
```

### 23.7.5 实际应用场景

```python
# 场景1：在列表中查找满足条件的元素的索引
scores = [88, 95, 72, 90, 65, 98]
excellent_indices = [i for i, s in enumerate(scores) if s >= 90]
print(excellent_indices)  # [1, 3, 5]

# 场景2：给输出加上行号
lines = ["第一行内容", "第二行内容", "第三行内容"]
for line_no, line in enumerate(lines, start=1):
    print(f"{line_no:3d} | {line}")
#   1 | 第一行内容
#   2 | 第二行内容
#   3 | 第三行内容

# 场景3：构建索引字典（元素 → 位置列表）
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
index_map = {}
for i, word in enumerate(words):
    index_map.setdefault(word, []).append(i)
print(index_map)
# {'apple': [0, 2, 5], 'banana': [1, 4], 'cherry': [3]}
```

---

## 23.8 zip()——并行遍历多个容器

### 23.8.1 基本用法

`zip()` 将多个可迭代对象"拉链式"地组合在一起，返回一个由元组组成的迭代器。

```python
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]

# zip 将两个列表按位置配对
for name, age in zip(names, ages):
    print(f"{name} is {age} years old")
# Alice is 25 years old
# Bob is 30 years old
# Charlie is 35 years old

# 查看 zip 的输出
print(list(zip(names, ages)))
# [('Alice', 25), ('Bob', 30), ('Charlie', 35)]
```

### 23.8.2 zip 多个可迭代对象

```python
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
cities = ["Beijing", "Shanghai", "Guangzhou"]

for name, age, city in zip(names, ages, cities):
    print(f"{name}, {age}岁, 来自{city}")
# Alice, 25岁, 来自Beijing
# Bob, 30岁, 来自Shanghai
# Charlie, 35岁, 来自Guangzhou
```

### 23.8.3 长度不等时的行为

```python
# 默认行为：以最短的为准（截断）
a = [1, 2, 3, 4, 5]
b = ["a", "b", "c"]
print(list(zip(a, b)))  # [(1, 'a'), (2, 'b'), (3, 'c')]  → 多余的4,5被丢弃

# 如果不想丢弃，使用 itertools.zip_longest
from itertools import zip_longest
print(list(zip_longest(a, b, fillvalue="?")))
# [(1, 'a'), (2, 'b'), (3, 'c'), (4, '?'), (5, '?')]

# Python 3.10+ 提供 strict 参数：长度不一致时报错
# list(zip(a, b, strict=True))  # ValueError: zip() has arguments with different lengths
```

### 23.8.4 zip() 的经典用途

```python
# 用途1：构建字典
keys = ["name", "age", "city"]
values = ["Alice", 25, "Beijing"]
person = dict(zip(keys, values))
print(person)  # {'name': 'Alice', 'age': 25, 'city': 'Beijing'}

# 用途2：矩阵转置
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]
transposed = list(zip(*matrix))  # * 解包操作
print(transposed)
# [(1, 4, 7), (2, 5, 8), (3, 6, 9)]
# 即每一列变成了一行

# 如果需要列表而非元组：
transposed_lists = [list(row) for row in zip(*matrix)]
print(transposed_lists)  # [[1, 4, 7], [2, 5, 8], [3, 6, 9]]

# 用途3：并行比较两个序列
old_prices = [10, 20, 30, 40]
new_prices = [12, 18, 35, 40]
for i, (old, new) in enumerate(zip(old_prices, new_prices)):
    change = new - old
    symbol = "↑" if change > 0 else ("↓" if change < 0 else "→")
    print(f"商品{i}: {old} → {new} ({symbol}{abs(change)})")
# 商品0: 10 → 12 (↑2)
# 商品1: 20 → 18 (↓2)
# 商品2: 30 → 35 (↑5)
# 商品3: 40 → 40 (→0)

# 用途4：解压（zip 的逆操作）
pairs = [("Alice", 25), ("Bob", 30), ("Charlie", 35)]
names, ages = zip(*pairs)
print(names)  # ('Alice', 'Bob', 'Charlie')
print(ages)   # (25, 30, 35)
```

---

## 23.9 all() 和 any()——全称判断与存在判断

### 23.9.1 all()：是否所有元素都为真？

```python
# 基本用法
print(all([True, True, True]))    # True
print(all([True, False, True]))   # False
print(all([1, 2, 3]))             # True（非零数值为 True）
print(all([1, 0, 3]))             # False（0 为 False）
print(all([]))                    # True（空容器返回 True！这是重要的边界情况）

# 实际用途：检查所有元素是否满足条件
scores = [85, 92, 78, 90, 88]
all_passed = all(s >= 60 for s in scores)
print(all_passed)  # True —— 所有人都及格了

scores2 = [85, 92, 55, 90, 88]
all_passed2 = all(s >= 60 for s in scores2)
print(all_passed2)  # False —— 有人不及格

# 检查所有字符串是否非空
strings = ["hello", "world", "python"]
print(all(strings))  # True（非空字符串为 True）

strings2 = ["hello", "", "python"]
print(all(strings2))  # False（空字符串为 False）

# 检查列表中所有元素是否为某种类型
data = [1, 2, 3, 4, 5]
print(all(isinstance(x, int) for x in data))  # True
```

### 23.9.2 any()：是否至少有一个元素为真？

```python
# 基本用法
print(any([False, False, True]))   # True（有一个 True）
print(any([False, False, False]))  # False（全是 False）
print(any([0, 0, 1]))              # True（1 为 True）
print(any([0, 0, 0]))              # False
print(any([]))                     # False（空容器返回 False）

# 实际用途：检查是否存在满足条件的元素
scores = [45, 55, 38, 62, 50]
has_passed = any(s >= 60 for s in scores)
print(has_passed)  # True —— 至少有一个人及格了

# 检查列表中是否有负数
numbers = [1, 2, -3, 4, 5]
has_negative = any(n < 0 for n in numbers)
print(has_negative)  # True

# 检查文件名列表中是否有 Python 文件
files = ["readme.md", "data.csv", "script.py", "notes.txt"]
has_python = any(f.endswith(".py") for f in files)
print(has_python)  # True
```

### 23.9.3 all() 和 any() 与逻辑运算的关系

```python
# all() 等价于连续的 and
# all([a, b, c]) ≡ a and b and c

# any() 等价于连续的 or
# any([a, b, c]) ≡ a or b or c

# 德摩根定律在这里同样适用：
# not all(x) ≡ any(not x)   "不是所有都满足" ≡ "存在一个不满足"
# not any(x) ≡ all(not x)   "不存在一个满足" ≡ "所有都不满足"

# 示例：
nums = [2, 4, 6, 8]
print(all(n % 2 == 0 for n in nums))       # True: 全偶数
print(not any(n % 2 != 0 for n in nums))   # True: 不存在奇数（等价说法）
```

### 23.9.4 短路特性

和 `and`/`or` 一样，`all()` 和 `any()` 具有**短路（short-circuit）**特性：

```python
# all() 遇到第一个 False 就停止
def check(x):
    print(f"  检查 {x}")
    return x > 0

print(all(check(x) for x in [3, 5, -1, 7, 9]))
# 输出：
#   检查 3
#   检查 5
#   检查 -1    ← 发现 False，立刻停止，不再检查 7 和 9
# False

# any() 遇到第一个 True 就停止
print(any(check(x) for x in [-3, -5, 1, -7, -9]))
# 输出：
#   检查 -3
#   检查 -5
#   检查 1     ← 发现 True，立刻停止
# True
```

---

## 23.10 内置函数通用性总结

| 函数          | 作用于 list | tuple | dict      | set | str | 返回类型 |
| ------------- | ----------- | ----- | --------- | --- | --- | -------- |
| `len()`       | ✅           | ✅     | ✅         | ✅   | ✅   | int      |
| `sum()`       | ✅           | ✅     | ✅（对键） | ✅   | ❌   | 数值     |
| `min()`       | ✅           | ✅     | ✅（对键） | ✅   | ✅   | 元素类型 |
| `max()`       | ✅           | ✅     | ✅（对键） | ✅   | ✅   | 元素类型 |
| `sorted()`    | ✅           | ✅     | ✅（对键） | ✅   | ✅   | list     |
| `reversed()`  | ✅           | ✅     | ✅(3.8+)   | ❌   | ✅   | 迭代器   |
| `enumerate()` | ✅           | ✅     | ✅（对键） | ✅   | ✅   | 迭代器   |
| `zip()`       | ✅           | ✅     | ✅         | ✅   | ✅   | 迭代器   |
| `all()`       | ✅           | ✅     | ✅         | ✅   | ✅   | bool     |
| `any()`       | ✅           | ✅     | ✅         | ✅   | ✅   | bool     |

---

## 23.11 本章小结

1. Python 的内置函数统一作用于**可迭代对象**，不局限于某一种容器。
2. 对常见内置容器，`len()` 通常是 O(1)；`sum()`/`min()`/`max()` 通常是 O(n)。
3. `sorted()` 返回新列表，`reversed()` 返回迭代器——都不修改原数据。
4. `enumerate()` 让你告别手动维护索引计数器。
5. `zip()` 是并行遍历多个容器的利器，也是构建字典和矩阵转置的常用手段。
6. `all()`/`any()` 是对容器进行全称/存在性判断的简洁工具，且具有短路特性。

---

# 第24章 容器中的排序、查找、统计、去重与分组问题范式

## 24.1 什么是"问题范式"？

在日常编程中，你会反复遇到一些**模式高度相似**的数据处理任务。我们不必每次从零开始思考，而是可以识别出这些**问题范式（Pattern）**，掌握它们的**标准解法**。

本章覆盖五大问题范式：
1. 排序
2. 查找
3. 统计
4. 去重
5. 分组

---

## 24.2 排序范式

### 24.2.1 基础排序

```python
# ✅ sorted()：不修改原数据，返回新列表
data = [5, 2, 8, 1, 9]
print(sorted(data))               # [1, 2, 5, 8, 9]
print(sorted(data, reverse=True)) # [9, 8, 5, 2, 1]

# ✅ list.sort()：原地修改
data.sort()
print(data)  # [1, 2, 5, 8, 9]
```

### 24.2.2 自定义排序：key 函数

```python
# 按字符串长度排序
words = ["watermelon", "hi", "apple", "go"]
print(sorted(words, key=len))        # ['hi', 'go', 'apple', 'watermelon']

# 按最后一个字符排序
print(sorted(words, key=lambda w: w[-1]))  # ['apple', 'hi', 'watermelon', 'go']

# 不区分大小写排序
names = ["alice", "Bob", "CHARLIE", "david"]
print(sorted(names, key=str.lower))  # ['alice', 'Bob', 'CHARLIE', 'david']

# 按字典的某个值排序
records = [
    {"name": "Alice", "score": 88},
    {"name": "Bob", "score": 95},
    {"name": "Charlie", "score": 72},
]
print(sorted(records, key=lambda r: r["score"]))
# [{'name': 'Charlie', ...}, {'name': 'Alice', ...}, {'name': 'Bob', ...}]
```

### 24.2.3 多级排序（多关键字排序）

```python
students = [
    ("Charlie", 88, 20),
    ("Alice", 88, 22),
    ("Bob", 95, 21),
    ("David", 72, 20),
]

# 先按成绩降序，成绩相同按年龄升序
result = sorted(students, key=lambda s: (-s[1], s[2]))
for name, score, age in result:
    print(f"{name}: 成绩={score}, 年龄={age}")
# Bob: 成绩=95, 年龄=21
# Charlie: 成绩=88, 年龄=20
# Alice: 成绩=88, 年龄=22
# David: 成绩=72, 年龄=20

# 注意：负号技巧只适用于数值类型
# 对于字符串，多级排序需求可以用 functools.cmp_to_key 或分两次排序
```

### 24.2.4 稳定排序与分两次排序技巧

Python 的 `sorted()` 和 `list.sort()` 使用的是 **Timsort**，它是**稳定排序**。"稳定"意味着：**相等的元素在排序后保持原来的相对顺序**。

利用稳定性，我们可以**分两次排序**来实现多级排序：

```python
students = [
    ("Charlie", 88),
    ("Alice", 88),
    ("Bob", 95),
]

# 技巧：先按次要关键字排，再按主要关键字排
# 第一步：按名字排序（次要关键字）
step1 = sorted(students, key=lambda s: s[0])
# [('Alice', 88), ('Bob', 95), ('Charlie', 88)]

# 第二步：按成绩排序（主要关键字）
step2 = sorted(step1, key=lambda s: s[1])
# [('Alice', 88), ('Charlie', 88), ('Bob', 95)]
# 注意：成绩都是 88 的 Alice 和 Charlie 保持了第一步排序后的顺序！
```

### 24.2.5 对 dict 排序

```python
scores = {"Alice": 88, "Bob": 95, "Charlie": 72, "David": 90}

# 按键排序
by_key = dict(sorted(scores.items()))
print(by_key)  # {'Alice': 88, 'Bob': 95, 'Charlie': 72, 'David': 90}

# 按值排序
by_value = dict(sorted(scores.items(), key=lambda item: item[1]))
print(by_value)  # {'Charlie': 72, 'Alice': 88, 'David': 90, 'Bob': 95}

# 按值降序
by_value_desc = dict(sorted(scores.items(), key=lambda item: item[1], reverse=True))
print(by_value_desc)  # {'Bob': 95, 'David': 90, 'Alice': 88, 'Charlie': 72}
```

---

## 24.3 查找范式

### 24.3.1 基础查找：in 运算符

```python
# 判断元素是否存在
fruits = ["apple", "banana", "cherry"]
print("banana" in fruits)     # True
print("grape" in fruits)      # False
print("grape" not in fruits)  # True

# 在 dict 中，in 检查的是键
d = {"a": 1, "b": 2}
print("a" in d)       # True
print(1 in d)          # False（1 不是键）
print(1 in d.values()) # True（1 是值）
```

### 24.3.2 查找索引位置

```python
# list.index()：查找第一次出现的索引
nums = [10, 20, 30, 20, 40]
print(nums.index(20))     # 1（第一次出现的位置）
# nums.index(99)          # ValueError: 99 is not in list

# 安全查找：先检查存在性
target = 99
if target in nums:
    idx = nums.index(target)
else:
    idx = -1
print(idx)  # -1

# 或者用异常处理
try:
    idx = nums.index(99)
except ValueError:
    idx = -1
```

### 24.3.3 条件查找：找到满足条件的元素

```python
data = [3, 7, 2, 9, 4, 8, 1, 6]

# 找到第一个大于 5 的元素
first_gt5 = next((x for x in data if x > 5), None)
print(first_gt5)  # 7

# 找到所有大于 5 的元素
all_gt5 = [x for x in data if x > 5]
print(all_gt5)  # [7, 9, 8, 6]

# 找到第一个大于 5 的元素的索引
first_idx = next((i for i, x in enumerate(data) if x > 5), None)
print(first_idx)  # 1

# 找到所有满足条件的元素的索引
all_indices = [i for i, x in enumerate(data) if x > 5]
print(all_indices)  # [1, 3, 5, 7]
```

### 24.3.4 在字典列表中查找

```python
users = [
    {"id": 1, "name": "Alice", "active": True},
    {"id": 2, "name": "Bob", "active": False},
    {"id": 3, "name": "Charlie", "active": True},
]

# 按 id 查找用户
def find_user_by_id(users, user_id):
    return next((u for u in users if u["id"] == user_id), None)

user = find_user_by_id(users, 2)
print(user)  # {'id': 2, 'name': 'Bob', 'active': False}

user = find_user_by_id(users, 99)
print(user)  # None

# 如果频繁按 id 查找，应该构建 dict 索引
user_by_id = {u["id"]: u for u in users}
print(user_by_id[2])  # {'id': 2, 'name': 'Bob', 'active': False}
# 平均 O(1) 查找，比遍历列表的 O(n) 更高效
```

### 24.3.5 二分查找（有序数据）

```python
import bisect

# 对已排序的列表进行高效查找
sorted_data = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]

# 查找插入位置
pos = bisect.bisect_left(sorted_data, 23)
print(pos)  # 5

# 判断元素是否存在（利用插入位置）
def binary_search(sorted_list, target):
    pos = bisect.bisect_left(sorted_list, target)
    return pos < len(sorted_list) and sorted_list[pos] == target

print(binary_search(sorted_data, 23))  # True
print(binary_search(sorted_data, 24))  # False
```

---

## 24.4 统计范式

### 24.4.1 基础统计

```python
data = [85, 92, 78, 90, 65, 98, 72, 88]

# 个数
print(len(data))           # 8

# 总和
print(sum(data))           # 668

# 平均值
print(sum(data) / len(data))  # 83.5

# 最大值 / 最小值
print(max(data))           # 98
print(min(data))           # 65

# 极差
print(max(data) - min(data))  # 33
```

### 24.4.2 元素计数：count()

```python
# list.count()
colors = ["red", "blue", "red", "green", "blue", "red"]
print(colors.count("red"))    # 3
print(colors.count("blue"))   # 2
print(colors.count("yellow")) # 0

# str.count()
text = "hello world hello python"
print(text.count("hello"))  # 2
```

### 24.4.3 频率统计：手动实现 vs Counter

```python
# 方法1：手动用 dict 统计
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]

freq = {}
for word in words:
    freq[word] = freq.get(word, 0) + 1
print(freq)  # {'apple': 3, 'banana': 2, 'cherry': 1}

# 方法2（等价）：用 setdefault
freq2 = {}
for word in words:
    freq2.setdefault(word, 0)
    freq2[word] += 1

# 方法3（推荐）：用 collections.Counter
from collections import Counter
freq3 = Counter(words)
print(freq3)          # Counter({'apple': 3, 'banana': 2, 'cherry': 1})
print(freq3["apple"]) # 3

# Counter 的好处
print(freq3.most_common(2))  # [('apple', 3), ('banana', 2)] —— 最常见的2个
print(freq3.most_common())   # 全部按频率降序排列
```

### 24.4.4 条件计数

```python
data = [3, 7, 2, 9, 4, 8, 1, 6]

# 统计大于 5 的元素个数
count_gt5 = sum(1 for x in data if x > 5)
print(count_gt5)  # 4

# 等价写法
count_gt5_v2 = len([x for x in data if x > 5])
print(count_gt5_v2)  # 4

# 更 Pythonic：利用 True=1, False=0
count_gt5_v3 = sum(x > 5 for x in data)
print(count_gt5_v3)  # 4
```

---

## 24.5 去重范式

### 24.5.1 不保留顺序的去重

```python
data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]

# 方法1：用 set
unique = list(set(data))
print(unique)  # 顺序不确定，如 [1, 2, 3, 4, 5, 6, 9]
```

### 24.5.2 保留顺序的去重

```python
data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]

# 方法1：dict.fromkeys()（推荐，简洁且在 Python 3.7+ 保证顺序）
unique = list(dict.fromkeys(data))
print(unique)  # [3, 1, 4, 5, 9, 2, 6]

# 方法2：手动去重（更灵活）
seen = set()
unique2 = []
for x in data:
    if x not in seen:
        seen.add(x)
        unique2.append(x)
print(unique2)  # [3, 1, 4, 5, 9, 2, 6]
```

### 24.5.3 按条件去重（自定义唯一性标准）

```python
# 按某个字段去重
records = [
    {"name": "Alice", "dept": "IT"},
    {"name": "Bob", "dept": "HR"},
    {"name": "Charlie", "dept": "IT"},
    {"name": "David", "dept": "HR"},
    {"name": "Eve", "dept": "Finance"},
]

# 每个部门只保留第一个人
seen_depts = set()
unique_by_dept = []
for r in records:
    if r["dept"] not in seen_depts:
        seen_depts.add(r["dept"])
        unique_by_dept.append(r)

print(unique_by_dept)
# [{'name': 'Alice', 'dept': 'IT'}, 
#  {'name': 'Bob', 'dept': 'HR'}, 
#  {'name': 'Eve', 'dept': 'Finance'}]
```

### 24.5.4 对不可哈希元素的去重

```python
# 如果元素是列表（不可哈希），不能直接放入 set
data = [[1, 2], [3, 4], [1, 2], [5, 6], [3, 4]]

# 方法1：转成 tuple 后去重
unique = [list(t) for t in dict.fromkeys(tuple(x) for x in data)]
print(unique)  # [[1, 2], [3, 4], [5, 6]]

# 方法2：手动去重
seen = []
unique2 = []
for x in data:
    if x not in seen:
        seen.append(x)  # seen 也是 list，in 操作为 O(n)
        unique2.append(x)
print(unique2)  # [[1, 2], [3, 4], [5, 6]]
```

---

## 24.6 分组范式

### 24.6.1 按条件分组

```python
# 将数据按某个条件分成若干组

# 场景：将数字按奇偶分组
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 方法1：多次列表推导
odds = [x for x in numbers if x % 2 != 0]
evens = [x for x in numbers if x % 2 == 0]
print(f"奇数: {odds}")   # [1, 3, 5, 7, 9]
print(f"偶数: {evens}")  # [2, 4, 6, 8, 10]

# 方法2：用 dict 分组（更通用）
groups = {}
for x in numbers:
    key = "odd" if x % 2 != 0 else "even"
    groups.setdefault(key, []).append(x)
print(groups)  # {'odd': [1, 3, 5, 7, 9], 'even': [2, 4, 6, 8, 10]}

# 方法3：用 defaultdict（更简洁）
from collections import defaultdict
groups2 = defaultdict(list)
for x in numbers:
    key = "odd" if x % 2 != 0 else "even"
    groups2[key].append(x)
print(dict(groups2))  # {'odd': [1, 3, 5, 7, 9], 'even': [2, 4, 6, 8, 10]}
```

### 24.6.2 按字段分组

这是实际工程中最常见的分组场景——将一组记录按某个字段的值分组。

```python
students = [
    {"name": "Alice", "class": "A", "score": 90},
    {"name": "Bob", "class": "B", "score": 85},
    {"name": "Charlie", "class": "A", "score": 78},
    {"name": "David", "class": "B", "score": 92},
    {"name": "Eve", "class": "A", "score": 88},
    {"name": "Frank", "class": "C", "score": 75},
]

# 按班级分组
from collections import defaultdict

by_class = defaultdict(list)
for s in students:
    by_class[s["class"]].append(s)

# 查看结果
for cls, members in sorted(by_class.items()):
    names = [m["name"] for m in members]
    print(f"班级 {cls}: {names}")
# 班级 A: ['Alice', 'Charlie', 'Eve']
# 班级 B: ['Bob', 'David']
# 班级 C: ['Frank']
```

### 24.6.3 分组后聚合

分组往往不是目的，分组后进行**聚合计算**（求和、求均值、取最大值等）才是真正需求。

```python
from collections import defaultdict

students = [
    {"name": "Alice", "class": "A", "score": 90},
    {"name": "Bob", "class": "B", "score": 85},
    {"name": "Charlie", "class": "A", "score": 78},
    {"name": "David", "class": "B", "score": 92},
    {"name": "Eve", "class": "A", "score": 88},
    {"name": "Frank", "class": "C", "score": 75},
]

# 先分组
by_class = defaultdict(list)
for s in students:
    by_class[s["class"]].append(s["score"])

# 再聚合
for cls, scores in sorted(by_class.items()):
    print(f"班级 {cls}: "
          f"人数={len(scores)}, "
          f"平均分={sum(scores)/len(scores):.1f}, "
          f"最高分={max(scores)}, "
          f"最低分={min(scores)}")
# 班级 A: 人数=3, 平均分=85.3, 最高分=90, 最低分=78
# 班级 B: 人数=2, 平均分=88.5, 最高分=92, 最低分=85
# 班级 C: 人数=1, 平均分=75.0, 最高分=75, 最低分=75
```

### 24.6.4 用 itertools.groupby 分组

`itertools.groupby` 是另一种分组工具，但要特别注意：它本质上是对**连续相邻且键相同的元素**分组。  
如果你希望“所有键相同的元素，不管原来隔多远，都被归到同一组”，那就必须先按分组键排序。

```python
from itertools import groupby

students = [
    {"name": "Alice", "class": "A", "score": 90},
    {"name": "Bob", "class": "B", "score": 85},
    {"name": "Charlie", "class": "A", "score": 78},
    {"name": "David", "class": "B", "score": 92},
    {"name": "Eve", "class": "A", "score": 88},
]

# 第一步：如果想把同班同学真正合并到同一组，就先排序
students_sorted = sorted(students, key=lambda s: s["class"])

# 第二步：用 groupby 分组
for cls, group in groupby(students_sorted, key=lambda s: s["class"]):
    members = list(group)  # group 是迭代器，必须转成 list 才能多次使用
    names = [m["name"] for m in members]
    print(f"班级 {cls}: {names}")
# 班级 A: ['Alice', 'Charlie', 'Eve']
# 班级 B: ['Bob', 'David']
```

> **重要提醒**：如果不先排序，`groupby` 只会把**连续的相同键**分为一组，不连续的会被拆成多组，导致结果错误。这是初学者最常犯的错误！

```python
from itertools import groupby

# 错误示范：不排序直接 groupby
data = [1, 1, 2, 2, 1, 1]
for key, group in groupby(data):
    print(f"{key}: {list(group)}")
# 1: [1, 1]    ← 前面的两个1
# 2: [2, 2]
# 1: [1, 1]    ← 后面的两个1，被当成单独一组了！
```

### 24.6.5 分组范式选择指南

| 场景           | 推荐方案                 | 说明                     |
| -------------- | ------------------------ | ------------------------ |
| 简单二分组     | 两个列表推导             | 如奇偶分组               |
| 多值分组       | `defaultdict(list)`      | 最通用、最常用           |
| 需要排序的分组 | `itertools.groupby`      | 适合流式处理已排序数据   |
| 分组后聚合     | `defaultdict` + 手动聚合 | 分两步：先攒数据，再计算 |

---

## 24.7 综合范式组合

实际问题往往需要组合多个范式。以下是一个综合案例：

```python
"""
需求：分析销售数据
1. 按产品分组
2. 每组去重（去掉重复订单号）
3. 统计每个产品的总销售额和订单数
4. 按总销售额降序排列
"""

from collections import defaultdict

orders = [
    {"order_id": 1001, "product": "手机", "amount": 3999},
    {"order_id": 1002, "product": "电脑", "amount": 6999},
    {"order_id": 1003, "product": "手机", "amount": 4599},
    {"order_id": 1001, "product": "手机", "amount": 3999},  # 重复订单
    {"order_id": 1004, "product": "耳机", "amount": 299},
    {"order_id": 1005, "product": "电脑", "amount": 7999},
    {"order_id": 1002, "product": "电脑", "amount": 6999},  # 重复订单
    {"order_id": 1006, "product": "耳机", "amount": 599},
]

# 第一步：去重（按 order_id）
seen_ids = set()
unique_orders = []
for order in orders:
    if order["order_id"] not in seen_ids:
        seen_ids.add(order["order_id"])
        unique_orders.append(order)

# 第二步：按产品分组
by_product = defaultdict(list)
for order in unique_orders:
    by_product[order["product"]].append(order["amount"])

# 第三步：聚合统计
stats = []
for product, amounts in by_product.items():
    stats.append({
        "product": product,
        "order_count": len(amounts),
        "total": sum(amounts),
        "avg": sum(amounts) / len(amounts),
    })

# 第四步：按总销售额降序排序
stats.sort(key=lambda s: s["total"], reverse=True)

# 输出结果
print(f"{'产品':<8} {'订单数':<8} {'总额':<10} {'均价':<10}")
print("-" * 36)
for s in stats:
    print(f"{s['product']:<8} {s['order_count']:<8} "
          f"¥{s['total']:<9} ¥{s['avg']:<9.0f}")

# 输出：
# 产品     订单数    总额        均价
# ------------------------------------
# 电脑     2        ¥14998     ¥7499
# 手机     2        ¥8598      ¥4299
# 耳机     2        ¥898       ¥449
```

---

## 24.8 本章小结

| 范式 | 核心工具                           | 要点                                     |
| ---- | ---------------------------------- | ---------------------------------------- |
| 排序 | `sorted()`, `key`, 多级排序        | Timsort 是稳定排序，善用 key 和元组比较  |
| 查找 | `in`, `index()`, `next()` + 生成器 | 大数据量查找用 set/dict 建索引           |
| 统计 | `len()`, `sum()`, `Counter`        | `Counter.most_common()` 是词频统计利器   |
| 去重 | `set()`, `dict.fromkeys()`, 手动   | 需要保序时用 `dict.fromkeys()`           |
| 分组 | `defaultdict(list)`, `groupby`     | `groupby` 要先排序，`defaultdict` 最通用 |

---

# 第25章 容器中的嵌套结构与复合数据建模方法

## 25.1 什么是嵌套结构？

**嵌套结构**是指容器中的元素本身又是容器。这是 Python 中处理复杂数据的基本方式。

```python
# 列表嵌套列表（矩阵）
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

# 字典嵌套字典（多层配置）
config = {
    "database": {
        "host": "localhost",
        "port": 5432,
        "credentials": {
            "username": "admin",
            "password": "secret"
        }
    },
    "server": {
        "host": "0.0.0.0",
        "port": 8080,
    }
}

# 列表嵌套字典（记录集合，类似数据库表）
users = [
    {"id": 1, "name": "Alice", "tags": ["python", "data"]},
    {"id": 2, "name": "Bob", "tags": ["java", "web"]},
    {"id": 3, "name": "Charlie", "tags": ["python", "ml"]},
]

# 字典嵌套列表（分组数据）
class_students = {
    "A班": ["Alice", "Charlie", "Eve"],
    "B班": ["Bob", "David"],
    "C班": ["Frank"],
}
```

---

## 25.2 嵌套结构的访问与操作

### 25.2.1 多层索引访问

```python
# 列表嵌套列表
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print(matrix[0])       # [1, 2, 3] → 第一行
print(matrix[0][1])    # 2 → 第一行第二列
print(matrix[2][2])    # 9 → 第三行第三列

# 字典嵌套字典
config = {
    "database": {"host": "localhost", "port": 5432},
    "server": {"host": "0.0.0.0", "port": 8080},
}
print(config["database"]["host"])  # "localhost"
print(config["server"]["port"])    # 8080

# 字典列表
users = [
    {"name": "Alice", "scores": [90, 85, 92]},
    {"name": "Bob", "scores": [78, 88, 95]},
]
print(users[0]["name"])          # "Alice"
print(users[1]["scores"][2])     # 95 → Bob 的第三个成绩
```

### 25.2.2 安全访问：避免 KeyError 和 IndexError

嵌套越深，访问出错的风险越高。

```python
config = {"database": {"host": "localhost"}}

# 危险写法：如果某个层级不存在就会报错
# print(config["server"]["port"])  # KeyError: 'server'

# 安全写法1：逐层检查
if "server" in config and "port" in config["server"]:
    port = config["server"]["port"]
else:
    port = 8080  # 默认值

# 安全写法2：用 dict.get() 链式调用
port = config.get("server", {}).get("port", 8080)
print(port)  # 8080

# 安全写法3：try-except
try:
    port = config["server"]["port"]
except KeyError:
    port = 8080
```

```python
# 安全写法4：封装一个通用的深层取值函数
def deep_get(data, keys, default=None):
    """沿着 keys 路径逐层深入取值，任何一层找不到就返回 default"""
    sentinel = object()
    for key in keys:
        if isinstance(data, dict):
            data = data.get(key, sentinel)
            if data is sentinel:
                return default
        elif isinstance(data, (list, tuple)) and isinstance(key, int):
            if -len(data) <= key < len(data):
                data = data[key]
            else:
                return default
        else:
            return default
    return data

config = {
    "database": {
        "credentials": {"username": "admin", "password": "secret"}
    }
}
print(deep_get(config, ["database", "credentials", "username"]))  # "admin"
print(deep_get(config, ["database", "credentials", "token"]))     # None
print(deep_get(config, ["server", "port"], default=8080))          # 8080
```

### 25.2.3 嵌套结构的修改

```python
# 修改嵌套列表
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
matrix[1][1] = 99
print(matrix)  # [[1, 2, 3], [4, 99, 6], [7, 8, 9]]

# 修改嵌套字典
config = {"database": {"host": "localhost", "port": 5432}}
config["database"]["port"] = 3306
print(config)  # {'database': {'host': 'localhost', 'port': 3306}}

# 添加新的嵌套层级
config["cache"] = {"type": "redis", "ttl": 300}
print(config)
# {'database': {'host': 'localhost', 'port': 3306},
#  'cache': {'type': 'redis', 'ttl': 300}}

# 向嵌套列表追加元素
users = [
    {"name": "Alice", "tags": ["python"]},
    {"name": "Bob", "tags": ["java"]},
]
users[0]["tags"].append("data")
print(users[0])  # {'name': 'Alice', 'tags': ['python', 'data']}
```

---

## 25.3 嵌套结构的遍历

### 25.3.1 遍历二维列表（矩阵）

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

# 方式1：嵌套 for 循环
for row in matrix:
    for val in row:
        print(val, end=" ")
    print()
# 1 2 3
# 4 5 6
# 7 8 9

# 方式2：带索引遍历
for i, row in enumerate(matrix):
    for j, val in enumerate(row):
        print(f"matrix[{i}][{j}] = {val}")

# 方式3：展平（将二维变一维）
flat = [val for row in matrix for val in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# 注意推导式的阅读顺序：
# [val for row in matrix for val in row]
# 等价于：
# result = []
# for row in matrix:        ← 外层循环在前
#     for val in row:        ← 内层循环在后
#         result.append(val)
```

### 25.3.2 遍历字典列表

```python
employees = [
    {"name": "Alice", "dept": "IT", "salary": 8000},
    {"name": "Bob", "dept": "HR", "salary": 7000},
    {"name": "Charlie", "dept": "IT", "salary": 9000},
]

# 遍历并格式化输出
for emp in employees:
    print(f"{emp['name']} | 部门: {emp['dept']} | 薪资: ¥{emp['salary']}")

# 提取某一列
names = [emp["name"] for emp in employees]
print(names)  # ['Alice', 'Bob', 'Charlie']

# 条件筛选
it_staff = [emp for emp in employees if emp["dept"] == "IT"]
print(it_staff)
# [{'name': 'Alice', 'dept': 'IT', 'salary': 8000},
#  {'name': 'Charlie', 'dept': 'IT', 'salary': 9000}]
```

### 25.3.3 遍历嵌套字典

```python
school = {
    "一年级": {
        "A班": ["Alice", "Bob"],
        "B班": ["Charlie", "David"],
    },
    "二年级": {
        "A班": ["Eve", "Frank"],
        "B班": ["Grace"],
    },
}

# 三层遍历
for grade, classes in school.items():
    print(f"\n{grade}:")
    for cls_name, students in classes.items():
        print(f"  {cls_name}: {', '.join(students)}")

# 输出：
# 一年级:
#   A班: Alice, Bob
#   B班: Charlie, David
#
# 二年级:
#   A班: Eve, Frank
#   B班: Grace
```

### 25.3.4 递归遍历：处理任意深度的嵌套

当嵌套层级不确定时，需要用递归来遍历：

```python
def flatten(data):
    """递归展平任意深度的嵌套列表"""
    result = []
    for item in data:
        if isinstance(item, (list, tuple)):
            result.extend(flatten(item))  # 递归处理子容器
        else:
            result.append(item)
    return result

nested = [1, [2, 3], [4, [5, 6, [7, 8]]], 9]
print(flatten(nested))  # [1, 2, 3, 4, 5, 6, 7, 8, 9]


def print_nested_dict(d, indent=0):
    """递归打印嵌套字典，带缩进"""
    for key, value in d.items():
        if isinstance(value, dict):
            print(" " * indent + f"{key}:")
            print_nested_dict(value, indent + 2)
        else:
            print(" " * indent + f"{key}: {value}")

config = {
    "app": {
        "name": "MyApp",
        "version": "1.0",
        "database": {
            "host": "localhost",
            "port": 5432,
            "pool": {
                "min": 5,
                "max": 20,
            }
        }
    }
}

print_nested_dict(config)
# app:
#   name: MyApp
#   version: 1.0
#   database:
#     host: localhost
#     port: 5432
#     pool:
#       min: 5
#       max: 20
```

---

## 25.4 常见复合数据建模模式

### 25.4.1 模式一：表格数据 —— list of dict

这是最常见的数据建模模式，类似于数据库表或 CSV 文件。每个字典代表一行记录。

```python
# 学生成绩表
students = [
    {"id": 1, "name": "Alice", "math": 90, "english": 85},
    {"id": 2, "name": "Bob", "math": 78, "english": 92},
    {"id": 3, "name": "Charlie", "math": 95, "english": 88},
]

# 优点：
# 1. 直观，每条记录结构清晰
# 2. 字段名自文档化（不需要记住"第几列是什么"）
# 3. 容易扩展字段
# 4. 可以直接用 json 模块序列化

# 常见操作：
# 筛选
honor = [s for s in students if s["math"] >= 90]

# 排序
by_math = sorted(students, key=lambda s: s["math"], reverse=True)

# 提取某一列
names = [s["name"] for s in students]

# 添加新记录
students.append({"id": 4, "name": "David", "math": 88, "english": 90})
```

### 25.4.2 模式二：索引/映射表 —— dict of dict

当你需要按某个键快速查找记录时。

```python
# 用户信息表（按 ID 索引）
users = {
    1001: {"name": "Alice", "email": "alice@example.com", "role": "admin"},
    1002: {"name": "Bob", "email": "bob@example.com", "role": "user"},
    1003: {"name": "Charlie", "email": "charlie@example.com", "role": "user"},
}

# O(1) 按 ID 查找
print(users[1001]["name"])  # "Alice"

# 从 list of dict 构建索引
students = [
    {"id": 1, "name": "Alice", "score": 90},
    {"id": 2, "name": "Bob", "score": 85},
]
# 构建 id → 学生记录 的索引
student_by_id = {s["id"]: s for s in students}
print(student_by_id[1])  # {'id': 1, 'name': 'Alice', 'score': 90}

# 构建 name → score 的映射
score_by_name = {s["name"]: s["score"] for s in students}
print(score_by_name["Bob"])  # 85
```

### 25.4.3 模式三：邻接表/图结构 —— dict of list

用于表示图（Graph）、树（Tree）、以及各种关系网络。

```python
# 社交网络：每个人的好友列表
friends = {
    "Alice": ["Bob", "Charlie", "David"],
    "Bob": ["Alice", "Eve"],
    "Charlie": ["Alice", "Frank"],
    "David": ["Alice"],
    "Eve": ["Bob"],
    "Frank": ["Charlie"],
}

# 查找 Alice 的好友
print(friends["Alice"])  # ['Bob', 'Charlie', 'David']

# 查找 Alice 和 Bob 的共同好友
common = set(friends["Alice"]) & set(friends["Bob"])
print(common)  # set()

# 更正确的共同好友（排除查询者自身）
alice_friends = set(friends["Alice"])
bob_friends = set(friends["Bob"])
common = alice_friends & bob_friends
common -= {"Alice", "Bob"}
print(common)  # set()（该示例数据下二者没有共同好友）

# 城市之间的路线图（带权重的邻接表）
routes = {
    "北京": [("上海", 1200), ("广州", 2000)],
    "上海": [("北京", 1200), ("杭州", 200)],
    "广州": [("北京", 2000), ("深圳", 150)],
    "杭州": [("上海", 200)],
    "深圳": [("广州", 150)],
}

# 查看从北京出发的路线
for dest, distance in routes["北京"]:
    print(f"北京 → {dest}: {distance}km")
# 北京 → 上海: 1200km
# 北京 → 广州: 2000km
```

### 25.4.4 模式四：分组容器 —— dict of list（按类别聚合）

与模式三形式相同，但语义不同——用于分类存储。

```python
# 按文件扩展名分组
from collections import defaultdict

files = [
    "readme.md", "main.py", "utils.py", "style.css",
    "index.html", "config.json", "test.py", "notes.md"
]

by_ext = defaultdict(list)
for f in files:
    ext = f.rsplit(".", 1)[-1]  # 取扩展名
    by_ext[ext].append(f)

print(dict(by_ext))
# {'md': ['readme.md', 'notes.md'],
#  'py': ['main.py', 'utils.py', 'test.py'],
#  'css': ['style.css'],
#  'html': ['index.html'],
#  'json': ['config.json']}
```

### 25.4.5 模式五：多级索引 —— dict of dict of list

适用于需要按多个维度组织数据的场景。

```python
# 学校 → 年级 → 学生列表
from collections import defaultdict

# 用 lambda 创建嵌套 defaultdict
school_data = defaultdict(lambda: defaultdict(list))

# 添加数据
records = [
    ("一中", "高一", "Alice"),
    ("一中", "高一", "Bob"),
    ("一中", "高二", "Charlie"),
    ("二中", "高一", "David"),
    ("二中", "高一", "Eve"),
    ("二中", "高二", "Frank"),
]

for school, grade, student in records:
    school_data[school][grade].append(student)

# 查看结构
for school, grades in school_data.items():
    print(f"\n{school}:")
    for grade, students in grades.items():
        print(f"  {grade}: {students}")

# 一中:
#   高一: ['Alice', 'Bob']
#   高二: ['Charlie']
#
# 二中:
#   高一: ['David', 'Eve']
#   高二: ['Frank']
```

### 25.4.6 模式六：树形结构 —— dict 嵌套 dict

```python
# 文件系统目录树
file_tree = {
    "project": {
        "src": {
            "main.py": None,     # None 表示叶子节点（文件）
            "utils.py": None,
            "models": {
                "user.py": None,
                "post.py": None,
            }
        },
        "tests": {
            "test_main.py": None,
            "test_utils.py": None,
        },
        "README.md": None,
    }
}

# 递归打印目录树
def print_tree(tree, prefix=""):
    items = list(tree.items())
    for i, (name, subtree) in enumerate(items):
        # 判断是最后一个元素吗？
        is_last = (i == len(items) - 1)
        connector = "└── " if is_last else "├── "
        print(f"{prefix}{connector}{name}")
        if subtree is not None:  # 如果是目录，递归进入
            extension = "    " if is_last else "│   "
            print_tree(subtree, prefix + extension)

print_tree(file_tree)
# └── project
#     ├── src
#     │   ├── main.py
#     │   ├── utils.py
#     │   └── models
#     │       ├── user.py
#     │       └── post.py
#     ├── tests
#     │   ├── test_main.py
#     │   └── test_utils.py
#     └── README.md
```

---

## 25.5 嵌套结构的陷阱

### 25.5.1 陷阱一：浅拷贝与深拷贝

这是嵌套结构中**最危险的陷阱**。

```python
# 浅拷贝：只拷贝最外层，内层仍然是引用共享
original = [[1, 2], [3, 4]]
shallow = original.copy()  # 或 list(original)、original[:]

shallow[0][0] = 99  # 修改内层
print(original)  # [[99, 2], [3, 4]] ← 原始数据也被修改了！
print(shallow)   # [[99, 2], [3, 4]]

# 原因：shallow[0] 和 original[0] 指向同一个列表对象

# 深拷贝：递归拷贝所有层级
import copy
original2 = [[1, 2], [3, 4]]
deep = copy.deepcopy(original2)

deep[0][0] = 99
print(original2)  # [[1, 2], [3, 4]] ← 原始数据保持不变！
print(deep)       # [[99, 2], [3, 4]]
```

```python
# 另一个经典陷阱：用 * 重复创建嵌套列表
# 错误写法：
matrix = [[0] * 3] * 3
print(matrix)  # [[0, 0, 0], [0, 0, 0], [0, 0, 0]] 看起来没问题

matrix[0][0] = 1
print(matrix)  # [[1, 0, 0], [1, 0, 0], [1, 0, 0]] ← 全部行都被修改了！
# 原因：* 3 创建的是同一个内层列表的 3 个引用

# 正确写法：用列表推导式
matrix = [[0] * 3 for _ in range(3)]
matrix[0][0] = 1
print(matrix)  # [[1, 0, 0], [0, 0, 0], [0, 0, 0]] ✅ 只修改了第一行
```

### 25.5.2 陷阱二：可变默认参数

```python
# 错误写法：默认参数使用可变容器
def add_item(item, lst=[]):  # ← 危险！
    lst.append(item)
    return lst

print(add_item("a"))  # ['a'] ← 看起来没问题
print(add_item("b"))  # ['a', 'b'] ← 问题出现了！默认的 lst 被累积修改

# 原因：默认参数 [] 只在函数定义时创建一次，后续调用共享同一个列表对象

# 正确写法：使用 None 作为占位符
def add_item_v2(item, lst=None):
    if lst is None:
        lst = []  # 每次调用都创建新列表
    lst.append(item)
    return lst

print(add_item_v2("a"))  # ['a']
print(add_item_v2("b"))  # ['b'] ✅ 正确！
```

### 25.5.3 陷阱三：遍历时修改嵌套结构

```python
# 在遍历字典时删除键 → 直接报错
d = {"a": 1, "b": 2, "c": 3}
# for key in d:
#     if d[key] < 2:
#         del d[key]  # RuntimeError: dictionary changed size during iteration

# 正确做法：先收集要操作的键，再修改
keys_to_delete = [k for k, v in d.items() if v < 2]
for k in keys_to_delete:
    del d[k]
print(d)  # {'b': 2, 'c': 3}

# 或者直接构建新字典
d2 = {"a": 1, "b": 2, "c": 3}
d2 = {k: v for k, v in d2.items() if v >= 2}
print(d2)  # {'b': 2, 'c': 3}
```

---

## 25.6 命名元组（NamedTuple）：比字典更轻量的记录

当你的数据结构是固定的字段集合时，`namedtuple` 是比 dict 更好的选择。

```python
from collections import namedtuple

# 定义一个"学生"类型
Student = namedtuple("Student", ["name", "age", "score"])

# 创建实例
s1 = Student("Alice", 20, 90)
s2 = Student(name="Bob", age=21, score=85)

# 访问字段（像属性一样）
print(s1.name)   # "Alice"
print(s1.score)  # 90

# 也支持索引（兼容 tuple 操作）
print(s1[0])     # "Alice"
print(s1[2])     # 90

# 可以解包
name, age, score = s1
print(f"{name} is {age} years old with score {score}")

# 不可变（和 tuple 一样）
# s1.name = "Charlie"  # AttributeError: can't set attribute

# 在列表中使用
students = [
    Student("Alice", 20, 90),
    Student("Bob", 21, 85),
    Student("Charlie", 22, 92),
]

# 排序
by_score = sorted(students, key=lambda s: s.score, reverse=True)
for s in by_score:
    print(f"{s.name}: {s.score}")
# Charlie: 92
# Alice: 90
# Bob: 85
```

**Python 3.6+ 更推荐使用 `typing.NamedTuple`**：

```python
from typing import NamedTuple

class Student(NamedTuple):
    name: str
    age: int
    score: float

s = Student("Alice", 20, 90.5)
print(s.name)   # "Alice"
print(s.score)  # 90.5
```

---

## 25.7 dataclass：可变的结构化数据

如果你需要**可变的**结构化数据（可以修改字段值），Python 3.7+ 提供了 `@dataclass`：

```python
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    age: int
    score: float

# 创建实例
s = Student("Alice", 20, 90.5)
print(s)          # Student(name='Alice', age=20, score=90.5) → 自动生成 __repr__
print(s.name)     # "Alice"

# 可以修改！
s.score = 95.0
print(s.score)    # 95.0

# 自动生成 __eq__
s2 = Student("Alice", 20, 95.0)
print(s == s2)    # True（所有字段相等）

# 在列表中使用
students = [
    Student("Alice", 20, 90),
    Student("Bob", 21, 85),
    Student("Charlie", 22, 92),
]
by_score = sorted(students, key=lambda s: s.score, reverse=True)
```

```python
# dataclass 支持默认值和高级特性
from dataclasses import dataclass, field

@dataclass
class Student:
    name: str
    age: int
    score: float = 0.0                    # 默认值
    tags: list = field(default_factory=list)  # 可变默认值用 field

s = Student("Alice", 20)
print(s)  # Student(name='Alice', age=20, score=0.0, tags=[])
s.tags.append("honor")
print(s)  # Student(name='Alice', age=20, score=0.0, tags=['honor'])
```

---

## 25.8 如何选择数据建模方式？

| 方案         | 适用场景           | 优点                       | 缺点                     |
| ------------ | ------------------ | -------------------------- | ------------------------ |
| `dict`       | 动态字段、快速原型 | 灵活、无需定义类           | 无类型提示、容易拼错键名 |
| `namedtuple` | 固定字段、只读记录 | 轻量、不可变、可哈希       | 不能修改字段值           |
| `@dataclass` | 固定字段、需要修改 | 可变、有类型提示、自动方法 | 需要定义类（略重）       |
| `tuple`      | 少量固定位置数据   | 极轻量                     | 无字段名，可读性差       |

```python
# 对比同一数据的不同表示方式：

# 方式1：tuple（可读性差）
point = (3.5, 7.2)     # 哪个是 x？哪个是 y？

# 方式2：dict（灵活但无约束）
point = {"x": 3.5, "y": 7.2}  # 清晰，但可能在某处写成 "X" 或 "z"

# 方式3：namedtuple（不可变，轻量）
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
point = Point(3.5, 7.2)  # point.x, point.y 清晰可读

# 方式4：dataclass（可变，功能丰富）
from dataclasses import dataclass
@dataclass
class Point:
    x: float
    y: float
point = Point(3.5, 7.2)  # 可以 point.x = 10 修改
```

---

## 25.9 本章小结

1. **嵌套是表达复杂数据的基础手段**，常见形式有 list of dict、dict of list、dict of dict 等。
2. **安全访问**嵌套数据需要注意 KeyError 和 IndexError，善用 `dict.get()` 和异常处理。
3. 掌握**递归遍历**技巧，才能应对任意深度的嵌套结构。
4. 最大陷阱是**浅拷贝 vs 深拷贝**——嵌套结构中 `copy()` 不等于完全复制！
5. 对于固定结构的数据，优先使用 `namedtuple` 或 `@dataclass` 代替裸 dict/tuple，获得更好的可读性和安全性。

---

# 第七编 Python中的高级容器与类STL工具

---

# 第26章 Python标准库中的高级容器生态：从基础容器到扩展工具

## 26.1 基础容器的不足

前几章我们深入学习了 `list`、`tuple`、`dict`、`set` 四大基础容器。它们覆盖了绝大多数数据存储需求，但在某些特定场景下，它们的效率或易用性并不理想：

| 场景                             | 基础容器          | 痛点                                     |
| -------------------------------- | ----------------- | ---------------------------------------- |
| 频繁在列表头部插入/删除          | `list`            | 头部操作是 O(n)，性能差                  |
| 字典中访问不存在的键时需要默认值 | `dict`            | 每次都要写 `if key not in d`，很啰嗦     |
| 频率统计                         | `dict`            | 需要手动写计数逻辑                       |
| 取前 K 大/前 K 小                | `list` + `sorted` | 排序整个列表是 O(n log n)，但只需要 K 个 |
| 在有序列表中高效插入并保持有序   | `list`            | 手动查找插入点 + insert 是 O(n)          |
| 生成排列组合                     | 手动循环          | 代码复杂，容易出错                       |

Python 标准库提供了一系列**高级容器和工具模块**来精准解决这些问题。

---

## 26.2 高级容器工具全景图

```
Python 标准库中的容器与工具
├── collections 模块  ← 高级容器
│   ├── deque            → 双端队列（高效首尾操作）
│   ├── defaultdict      → 带默认值的字典
│   ├── Counter          → 计数器字典
│   ├── OrderedDict      → 有序字典（3.7 前的替代品；现在有额外功能）
│   ├── ChainMap         → 字典链（多层字典合并查找）
│   └── namedtuple       → 命名元组（已在上章介绍）
│
├── heapq 模块  ← 堆/优先队列
│   ├── heappush / heappop → 维护最小堆
│   ├── nlargest / nsmallest → Top-K 问题
│   └── heapify / merge     → 堆化与归并
│
├── bisect 模块  ← 二分查找与有序插入
│   ├── bisect_left / bisect_right → 查找插入点
│   └── insort_left / insort_right → 有序插入
│
├── itertools 模块  ← 迭代器工具
│   ├── count / cycle / repeat     → 无限迭代器
│   ├── chain / zip_longest        → 迭代器拼接
│   ├── product / permutations / combinations → 组合枚举
│   ├── groupby                    → 连续分组
│   ├── islice / takewhile / dropwhile → 迭代器切片
│   └── accumulate / starmap       → 累积与映射
│
└── functools 模块（部分）  ← 函数式工具
    └── reduce → 累积归约
```

接下来的第27-31章将逐个深入讲解这些模块。

---

## 26.3 这些工具之间的协作关系

这些模块不是孤立的，它们经常协作使用：

```python
# 示例：用多个模块协作解决一个问题
# 问题：在一组日志记录中，找出出现频率最高的 5 个错误类型

from collections import Counter
from itertools import chain

# 假设有多个日志文件的错误列表
errors_day1 = ["TimeoutError", "ValueError", "TimeoutError", "KeyError"]
errors_day2 = ["ValueError", "ValueError", "IOError", "TimeoutError"]
errors_day3 = ["KeyError", "KeyError", "ValueError", "MemoryError"]

# itertools.chain：将多个列表连接为一个迭代器
all_errors = chain(errors_day1, errors_day2, errors_day3)

# collections.Counter：统计频率
error_freq = Counter(all_errors)

# Counter.most_common：取前 N 个
print("最常见的错误类型：")
for error, count in error_freq.most_common(3):
    print(f"  {error}: {count}次")
# 最常见的错误类型：
#   ValueError: 4次
#   TimeoutError: 3次
#   KeyError: 3次
```

---

## 26.4 本章小结

本章是第七编的导航章节。我们了解了：
1. 基础容器在**特定场景下的局限性**。
2. Python 标准库提供的**四大高级工具模块**：`collections`、`heapq`、`bisect`、`itertools`。
3. 这些工具之间的**协作关系**。

接下来我们将逐章深入每个模块。

---

# 第27章 collections模块：deque、defaultdict、Counter、OrderedDict、ChainMap

## 27.1 collections 模块概述

`collections` 是 Python 标准库中最实用的模块之一。它提供了几种**基础容器的增强版本**，每个都针对特定场景优化。

```python
from collections import deque, defaultdict, Counter, OrderedDict, ChainMap
```

---

## 27.2 deque：双端队列

### 27.2.1 为什么需要 deque？

`list` 在**末尾**进行增删操作效率很高（O(1)），但在**头部**操作效率很低（O(n)，因为要移动所有元素）。

```python
# list 头部操作的低效
import time

lst = list(range(100_000))
start = time.perf_counter()
for _ in range(1000):
    lst.insert(0, 0)  # 头部插入
print(f"list 头部插入 1000 次: {time.perf_counter() - start:.3f}s")

# deque 头部操作的高效
from collections import deque
dq = deque(range(100_000))
start = time.perf_counter()
for _ in range(1000):
    dq.appendleft(0)  # 头部插入
print(f"deque 头部插入 1000 次: {time.perf_counter() - start:.6f}s")

# 典型结果：
# list  头部插入 1000 次: 0.850s
# deque 头部插入 1000 次: 0.000100s （快了几千倍）
```

### 27.2.2 deque 的基本操作

```python
from collections import deque

# 创建
dq = deque()                     # 空 deque
dq = deque([1, 2, 3, 4, 5])     # 从 list 创建
dq = deque("hello")             # 从字符串创建: deque(['h', 'e', 'l', 'l', 'o'])
dq = deque(range(5))             # 从 range 创建

# ===== 两端添加 =====
dq = deque([1, 2, 3])
dq.append(4)         # 右端添加: deque([1, 2, 3, 4])
dq.appendleft(0)     # 左端添加: deque([0, 1, 2, 3, 4])

# ===== 两端删除 =====
dq = deque([1, 2, 3, 4, 5])
right = dq.pop()       # 右端弹出: 5, deque变为 [1, 2, 3, 4]
left = dq.popleft()    # 左端弹出: 1, deque变为 [2, 3, 4]

# ===== 批量添加 =====
dq = deque([1, 2, 3])
dq.extend([4, 5, 6])       # 右端批量: deque([1, 2, 3, 4, 5, 6])
dq.extendleft([0, -1, -2]) # 左端批量: deque([-2, -1, 0, 1, 2, 3, 4, 5, 6])
# 注意 extendleft 的顺序：逐个从左端插入，所以结果是反序的！

# ===== 旋转 =====
dq = deque([1, 2, 3, 4, 5])
dq.rotate(2)    # 右旋2步: deque([4, 5, 1, 2, 3])   (尾部的2个移到头部)
dq.rotate(-2)   # 左旋2步: deque([1, 2, 3, 4, 5])   (恢复原状)

# ===== 其他方法 =====
dq = deque([1, 2, 3, 2, 1])
print(dq.count(2))       # 2（出现次数）
print(dq.index(3))       # 2（第一次出现的位置）
dq.remove(2)              # 删除第一个出现的 2: deque([1, 3, 2, 1])
dq.reverse()              # 原地反转: deque([1, 2, 3, 1])
dq.clear()                # 清空
```

### 27.2.3 maxlen：限制最大长度的 deque

这是 deque 的一个独特特性——当 deque 已满时，从一端添加元素会自动从另一端丢弃元素。

```python
from collections import deque

# 创建一个最多容纳 5 个元素的 deque
dq = deque(maxlen=5)
for i in range(8):
    dq.append(i)
    print(f"追加 {i}: {list(dq)}")

# 追加 0: [0]
# 追加 1: [0, 1]
# 追加 2: [0, 1, 2]
# 追加 3: [0, 1, 2, 3]
# 追加 4: [0, 1, 2, 3, 4]   ← 满了
# 追加 5: [1, 2, 3, 4, 5]   ← 左端的 0 被自动丢弃
# 追加 6: [2, 3, 4, 5, 6]   ← 左端的 1 被自动丢弃
# 追加 7: [3, 4, 5, 6, 7]   ← 左端的 2 被自动丢弃
```

### 27.2.4 deque 的典型应用

```python
from collections import deque

# 应用1：滑动窗口（保留最近 N 个数据点）
def moving_average(data, window_size):
    """计算滑动平均值"""
    window = deque(maxlen=window_size)
    results = []
    for val in data:
        window.append(val)
        avg = sum(window) / len(window)
        results.append(avg)
    return results

prices = [100, 102, 104, 103, 105, 107, 106, 108]
print(moving_average(prices, 3))
# [100.0, 101.0, 102.0, 103.0, 104.0, 105.0, 106.0, 107.0]

# 应用2：BFS（广度优先搜索）
def bfs(graph, start):
    """广度优先遍历图"""
    visited = set()
    queue = deque([start])  # 用 deque 作为队列（左进右出 或 右进左出）
    order = []

    while queue:
        node = queue.popleft()  # O(1) 从左端取出
        if node not in visited:
            visited.add(node)
            order.append(node)
            queue.extend(graph.get(node, []))  # 将邻居加入右端

    return order

graph = {
    "A": ["B", "C"],
    "B": ["D", "E"],
    "C": ["F"],
    "D": [], "E": [], "F": [],
}
print(bfs(graph, "A"))  # ['A', 'B', 'C', 'D', 'E', 'F']

# 应用3：保留最近 N 条日志
recent_logs = deque(maxlen=100)  # 只保留最近 100 条

def log(message):
    from datetime import datetime
    entry = f"[{datetime.now():%H:%M:%S}] {message}"
    recent_logs.append(entry)
    print(entry)

log("服务启动")
log("收到请求")
# recent_logs 永远不会超过 100 条
```

---

## 27.3 defaultdict：带默认值的字典

### 27.3.1 问题引入

使用普通 `dict` 时，访问不存在的键会引发 `KeyError`：

```python
d = {}
# d["missing"]  # KeyError: 'missing'

# 典型的"先检查再赋值"模式
word_counts = {}
for word in ["apple", "banana", "apple", "cherry", "banana", "apple"]:
    if word not in word_counts:
        word_counts[word] = 0
    word_counts[word] += 1

# 或者用 get
for word in ["apple", "banana", "apple"]:
    word_counts[word] = word_counts.get(word, 0) + 1

# 或者用 setdefault
groups = {}
data = [("a", 1), ("b", 2), ("a", 3)]
for key, val in data:
    groups.setdefault(key, []).append(val)
```

这些写法都正确，但不够简洁。`defaultdict` 一劳永逸地解决了这个问题。

### 27.3.2 defaultdict 的基本用法

```python
from collections import defaultdict

# 创建 defaultdict 时，传入一个"工厂函数"
# 当访问不存在的键时，自动用这个工厂函数创建默认值

# int 工厂 → 默认值为 0
word_counts = defaultdict(int)
for word in ["apple", "banana", "apple", "cherry", "banana", "apple"]:
    word_counts[word] += 1  # 不存在的键自动初始化为 0
print(dict(word_counts))  # {'apple': 3, 'banana': 2, 'cherry': 1}

# list 工厂 → 默认值为 []
groups = defaultdict(list)
data = [("a", 1), ("b", 2), ("a", 3), ("b", 4), ("c", 5)]
for key, val in data:
    groups[key].append(val)  # 不存在的键自动初始化为空列表
print(dict(groups))  # {'a': [1, 3], 'b': [2, 4], 'c': [5]}

# set 工厂 → 默认值为 set()
tag_users = defaultdict(set)
data = [("python", "Alice"), ("java", "Bob"), ("python", "Charlie"), ("python", "Alice")]
for tag, user in data:
    tag_users[tag].add(user)
print(dict(tag_users))  # {'python': {'Alice', 'Charlie'}, 'java': {'Bob'}}
```

### 27.3.3 自定义工厂函数

```python
from collections import defaultdict

# 使用 lambda 自定义默认值
scores = defaultdict(lambda: -1)  # 默认值为 -1
scores["Alice"] = 90
print(scores["Alice"])    # 90
print(scores["Unknown"])  # -1（自动创建）

# 嵌套 defaultdict
nested = defaultdict(lambda: defaultdict(int))
nested["Alice"]["math"] = 90
nested["Alice"]["english"] = 85
nested["Bob"]["math"] = 78
print(nested["Alice"]["math"])     # 90
print(nested["Charlie"]["physics"])# 0（两层都自动创建）
print(dict(nested))
# {'Alice': defaultdict(<class 'int'>, {'math': 90, 'english': 85}),
#  'Bob': defaultdict(<class 'int'>, {'math': 78}),
#  'Charlie': defaultdict(<class 'int'>, {'physics': 0})}
```

### 27.3.4 defaultdict 的注意事项

```python
from collections import defaultdict

dd = defaultdict(int)

# 注意：仅仅"查看"一个不存在的键，也会自动创建并写入！
print(dd["x"])  # 0
print(dict(dd))  # {'x': 0}  ← 自动创建了键 'x'

# 如果只是想查看而不想创建，应该用 get()
dd2 = defaultdict(int)
print(dd2.get("y", 0))  # 0
print(dict(dd2))        # {}  ← 没有自动创建

# in 运算符不会创建键
print("z" in dd2)       # False
print(dict(dd2))        # {}  ← 没有自动创建
```

---

## 27.4 Counter：计数器

### 27.4.1 基本用法

`Counter` 是 `dict` 的子类，专门用于计数。

```python
from collections import Counter

# 从可迭代对象创建
c1 = Counter("abracadabra")
print(c1)  # Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

c2 = Counter(["apple", "banana", "apple", "cherry", "banana", "apple"])
print(c2)  # Counter({'apple': 3, 'banana': 2, 'cherry': 1})

# 从字典创建
c3 = Counter({"红": 3, "蓝": 2, "绿": 1})
print(c3)  # Counter({'红': 3, '蓝': 2, '绿': 1})

# 关键字参数创建
c4 = Counter(cats=4, dogs=8)
print(c4)  # Counter({'dogs': 8, 'cats': 4})
```

### 27.4.2 Counter 的核心方法

```python
from collections import Counter

c = Counter("abracadabra")

# most_common(n)：返回出现次数前 n 多的元素
print(c.most_common(3))    # [('a', 5), ('b', 2), ('r', 2)]
print(c.most_common())     # 全部，按计数降序

# elements()：按计数展开为迭代器
print(list(c.elements()))
# ['a', 'a', 'a', 'a', 'a', 'b', 'b', 'r', 'r', 'c', 'd']

# total()（Python 3.10+）：所有计数之和
# print(c.total())  # 11

# 访问不存在的键返回 0（不报错！）
print(c["z"])  # 0（不会产生 KeyError）

# 更新计数
c.update("aaa")   # 追加计数
print(c["a"])       # 8（原来5 + 新增3）

c2 = Counter("xyz")
c.update(c2)        # 合并另一个 Counter
print(c["x"])       # 1

# 减去计数
c.subtract("aa")
print(c["a"])       # 6（8 - 2）
```

### 27.4.3 Counter 的算术运算

```python
from collections import Counter

c1 = Counter(a=3, b=2, c=1)
c2 = Counter(a=1, b=2, c=3)

# 加法
print(c1 + c2)  # Counter({'a': 4, 'b': 4, 'c': 4})

# 减法（只保留正数计数）
print(c1 - c2)  # Counter({'a': 2})  ← b和c的差值为0或负数，被丢弃

# 交集（取较小值）
print(c1 & c2)  # Counter({'b': 2, 'a': 1, 'c': 1})

# 并集（取较大值）
print(c1 | c2)  # Counter({'a': 3, 'c': 3, 'b': 2})
```

### 27.4.4 Counter 的实际应用

```python
from collections import Counter

# 应用1：词频统计
text = "the quick brown fox jumps over the lazy dog the fox"
words = text.split()
freq = Counter(words)
print(freq.most_common(3))
# [('the', 3), ('fox', 2), ('quick', 1)]

# 应用2：找到两个列表的公共元素（带频率）
list1 = [1, 2, 2, 3, 3, 3]
list2 = [2, 2, 2, 3, 3, 4]
common = Counter(list1) & Counter(list2)
print(list(common.elements()))  # [2, 2, 3, 3]（取每个元素共有的最小次数）

# 应用3：验证两个字符串是否为字母异位词（Anagram）
def is_anagram(s1, s2):
    return Counter(s1) == Counter(s2)

print(is_anagram("listen", "silent"))  # True
print(is_anagram("hello", "world"))    # False

# 应用4：投票统计
votes = ["Alice", "Bob", "Alice", "Charlie", "Bob", "Alice", "Bob", "Bob"]
result = Counter(votes)
winner = result.most_common(1)[0]
print(f"获胜者: {winner[0]}（{winner[1]}票）")  # 获胜者: Bob（4票）
```

---

## 27.5 OrderedDict：有序字典

### 27.5.1 为什么还需要 OrderedDict？

从 Python 3.7 开始，普通 `dict` 已经保证插入顺序。那 `OrderedDict` 还有存在的价值吗？

答案：**有**。`OrderedDict` 提供了一些普通 dict 没有的功能：

```python
from collections import OrderedDict

# 功能1：move_to_end()
od = OrderedDict([("a", 1), ("b", 2), ("c", 3)])
od.move_to_end("a")           # 移到末尾
print(list(od.keys()))          # ['b', 'c', 'a']

od.move_to_end("c", last=False)# 移到开头
print(list(od.keys()))          # ['c', 'b', 'a']

# 功能2：popitem(last=True/False)
od = OrderedDict([("a", 1), ("b", 2), ("c", 3)])
print(od.popitem(last=True))   # ('c', 3) ← 弹出最后一个
print(od.popitem(last=False))  # ('a', 1) ← 弹出第一个
# 普通 dict 的 popitem() 只有 last=True 的行为

# 功能3：比较时考虑顺序
od1 = OrderedDict([("a", 1), ("b", 2)])
od2 = OrderedDict([("b", 2), ("a", 1)])
print(od1 == od2)  # False ← 顺序不同，视为不等

d1 = {"a": 1, "b": 2}
d2 = {"b": 2, "a": 1}
print(d1 == d2)  # True ← 普通 dict 比较时忽略顺序
```

### 27.5.2 OrderedDict 实现 LRU 缓存

`OrderedDict` 最经典的应用是实现 **LRU（Least Recently Used）缓存**：

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = OrderedDict()

    def get(self, key):
        if key in self.cache:
            self.cache.move_to_end(key)  # 访问过的移到末尾（标记为"最近使用"）
            return self.cache[key]
        return -1

    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)  # 淘汰最久没使用的（开头的那个）

# 使用
lru = LRUCache(3)
lru.put("a", 1)
lru.put("b", 2)
lru.put("c", 3)
print(lru.get("a"))  # 1（a 被标记为最近使用）
lru.put("d", 4)      # 容量超了，淘汰最久没用的 "b"
print(lru.get("b"))   # -1（已被淘汰）
print(list(lru.cache.keys()))  # ['c', 'a', 'd']
```

---

## 27.6 ChainMap：字典链

### 27.6.1 什么是 ChainMap？

`ChainMap` 将多个字典组合在一起，查找时**按顺序逐层搜索**——找到第一个匹配的就返回。

```python
from collections import ChainMap

defaults = {"color": "red", "size": "medium", "font": "Arial"}
user_settings = {"color": "blue", "font": "Helvetica"}
session_settings = {"color": "green"}

# 创建 ChainMap：越靠前的优先级越高
config = ChainMap(session_settings, user_settings, defaults)

print(config["color"])  # "green"     ← session_settings 中找到
print(config["font"])   # "Helvetica" ← 跳过 session（没有），在 user 中找到
print(config["size"])   # "medium"    ← 跳过 session 和 user，在 defaults 中找到

# ChainMap 不会创建新字典，只是维护引用链
# 修改 user_settings 会影响 ChainMap
user_settings["size"] = "large"
print(config["size"])  # "large"
```

### 27.6.2 ChainMap 的操作

```python
from collections import ChainMap

# 写入操作只影响第一个字典
config = ChainMap({"a": 1}, {"b": 2}, {"c": 3})
config["d"] = 4
print(config.maps[0])  # {'a': 1, 'd': 4} ← 新键被加到第一个字典

# new_child()：创建新的子层
config2 = config.new_child({"e": 5})
print(config2["e"])  # 5
print(config2["a"])  # 1

# parents：访问除第一层以外的部分
print(config2.parents)  # ChainMap({'a': 1, 'd': 4}, {'b': 2}, {'c': 3})
```

### 27.6.3 ChainMap 的实际应用

```python
import os
from collections import ChainMap

# 应用1：多层配置（命令行 > 环境变量 > 配置文件 > 默认值）
cli_args = {"debug": True}
env_vars = {"PORT": "8080"}
file_config = {"PORT": "3000", "HOST": "localhost"}
defaults_config = {"PORT": "80", "HOST": "0.0.0.0", "debug": False}

config = ChainMap(cli_args, env_vars, file_config, defaults_config)
print(config["debug"])  # True（来自命令行）
print(config["PORT"])   # "8080"（来自环境变量，覆盖了配置文件的3000）
print(config["HOST"])   # "localhost"（来自配置文件，覆盖了默认的0.0.0.0）

# 应用2：变量作用域模拟（像 Python 的局部/全局/内置作用域）
builtin_scope = {"print": print, "len": len}
global_scope = {"x": 10, "y": 20}
local_scope = {"x": 100, "z": 30}

scope = ChainMap(local_scope, global_scope, builtin_scope)
print(scope["x"])  # 100（局部优先）
print(scope["y"])  # 20（从全局找到）
```

---

## 27.7 本章小结

| 容器          | 核心特点               | 典型应用               |
| ------------- | ---------------------- | ---------------------- |
| `deque`       | 两端 O(1) 操作、maxlen | 队列、BFS、滑动窗口    |
| `defaultdict` | 自动初始化缺失键       | 分组、计数、嵌套字典   |
| `Counter`     | 专业计数器、算术运算   | 词频统计、投票、异位词 |
| `OrderedDict` | 顺序敏感、move_to_end  | LRU 缓存               |
| `ChainMap`    | 多层字典链式查找       | 多级配置、作用域模拟   |

---

# 第28章 heapq模块：堆结构、优先队列与Top-K问题

## 28.1 什么是堆（Heap）？

**堆**是一种特殊的完全二叉树结构，Python 中的 `heapq` 模块实现的是**最小堆（Min-Heap）**：

- **性质**：父节点的值 ≤ 子节点的值
- **结果**：堆顶（第一个元素）永远是最小值
- **存储**：使用普通的 `list` 存储，不需要额外的数据结构（隐含二叉树逻辑）

> 💡 **对比 C++ STL**：C++ 中的 `std::priority_queue` 默认是一个**最大堆（Max-Heap）**，而 Python 的 `heapq` 模块按照最小堆算法设计（且只有最小堆）。如果不加留意，跨语言转换很容易在这里栽跟头。（隐含二叉树逻辑）

> 💡 **对比 C++ STL**：C++ 中的 `std::priority_queue` 默认是一个**最大堆（Max-Heap）**，而 Python 的 `heapq` 模块按照最小堆算法设计（且只有最小堆）。如果不加留意，跨语言转换很容易在这里栽跟头。

```
        1           ← 最小值在顶部
       / \
      3   2
     / \ / \
    7  4 5  6
    
对应的列表: [1, 3, 2, 7, 4, 5, 6]
# 对于索引 i 的元素:
# 父节点: (i-1) // 2
# 左子节点: 2*i + 1
# 右子节点: 2*i + 2
```

### 28.1.1 堆的核心操作时间复杂度

| 操作         | 时间复杂度 | 说明             |
| ------------ | ---------- | ---------------- |
| 查看最小值   | O(1)       | 直接取 `heap[0]` |
| 插入元素     | O(log n)   | `heappush`       |
| 弹出最小值   | O(log n)   | `heappop`        |
| 堆化整个列表 | O(n)       | `heapify`        |

---

## 28.2 heapq 基本操作

```python
import heapq

# ===== heappush：向堆中添加元素 =====
heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 3)
heapq.heappush(heap, 7)
heapq.heappush(heap, 1)
print(heap)       # [1, 3, 7, 5] → 注意：不是完全排序的，只保证堆性质
print(heap[0])    # 1 → 最小值始终在索引 0

# ===== heappop：弹出最小值 =====
smallest = heapq.heappop(heap)
print(smallest)   # 1
print(heap)       # [3, 5, 7]

# 连续 pop 可以实现排序（即堆排序）
heap = [5, 3, 7, 1, 9, 2]
heapq.heapify(heap)
sorted_result = []
while heap:
    sorted_result.append(heapq.heappop(heap))
print(sorted_result)  # [1, 2, 3, 5, 7, 9]

# ===== heapify：将列表原地转为堆 =====
data = [5, 3, 7, 1, 9, 2]
heapq.heapify(data)  # O(n) 原地堆化
print(data)     # [1, 3, 2, 5, 9, 7]（堆结构）
print(data[0])  # 1（最小值）

# ===== heapreplace：弹出最小值并插入新元素（一步操作） =====
heap = [1, 3, 5, 7]
old = heapq.heapreplace(heap, 4)
print(old)    # 1（被弹出的最小值）
print(heap)   # [3, 4, 5, 7]（新元素 4 已插入并维持堆性质）

# ===== heappushpop：先插入再弹出最小值（比分两步快） =====
heap = [2, 4, 6]
old = heapq.heappushpop(heap, 1)
print(old)    # 1（插入1后，1是最小的，立刻被弹出）
print(heap)   # [2, 4, 6]

old = heapq.heappushpop(heap, 3)
print(old)    # 2（插入3后，2是最小的，被弹出）
print(heap)   # [3, 4, 6]
```

---

## 28.3 Top-K 问题

### 28.3.1 nlargest 和 nsmallest

```python
import heapq

data = [45, 12, 67, 34, 89, 23, 56, 78, 90, 11]

# 取前 3 大
print(heapq.nlargest(3, data))   # [90, 89, 78]

# 取前 3 小
print(heapq.nsmallest(3, data))  # [11, 12, 23]

# 支持 key 参数
students = [
    {"name": "Alice", "score": 90},
    {"name": "Bob", "score": 75},
    {"name": "Charlie", "score": 95},
    {"name": "David", "score": 82},
]
top2 = heapq.nlargest(2, students, key=lambda s: s["score"])
print(top2)
# [{'name': 'Charlie', 'score': 95}, {'name': 'Alice', 'score': 90}]
```

### 28.3.2 性能选择指南

| 场景                 | 推荐方法                   | 复杂度     |
| -------------------- | -------------------------- | ---------- |
| 只要最小/最大的 1 个 | `min()` / `max()`          | O(n)       |
| 要前 K 个（K 较小）  | `heapq.nlargest/nsmallest` | O(n log K) |
| 要全部排序           | `sorted()`                 | O(n log n) |

---

## 28.4 实现最大堆

Python 的 `heapq` 只支持最小堆。实现最大堆的技巧是**取反**：

```python
import heapq

# 最大堆：插入时取反，弹出时再取反
max_heap = []
for val in [3, 1, 4, 1, 5, 9]:
    heapq.heappush(max_heap, -val)

# 弹出最大值
largest = -heapq.heappop(max_heap)
print(largest)  # 9

# 查看当前最大值
print(-max_heap[0])  # 5
```

---

## 28.5 优先队列

```python
import heapq

# 优先队列：(优先级, 数据) 的堆
task_queue = []
heapq.heappush(task_queue, (3, "低优先级任务"))
heapq.heappush(task_queue, (1, "紧急任务"))
heapq.heappush(task_queue, (2, "普通任务"))

while task_queue:
    priority, task = heapq.heappop(task_queue)
    print(f"[优先级{priority}] 执行: {task}")
# [优先级1] 执行: 紧急任务
# [优先级2] 执行: 普通任务
# [优先级3] 执行: 低优先级任务
```

> **注意**：当优先级相同时，Python 会比较元组的第二个元素。如果第二个元素不可比较（如 dict），会报错。解决方法是插入一个递增计数器：

```python
import heapq
from itertools import count

counter = count()  # 全局递增计数器
task_queue = []

heapq.heappush(task_queue, (1, next(counter), "任务A"))
heapq.heappush(task_queue, (1, next(counter), "任务B"))  # 相同优先级
# 即使优先级相同，counter 不同，不会比较到字符串
```

---

## 28.6 merge：归并多个有序序列

```python
import heapq

a = [1, 4, 7, 10]
b = [2, 5, 8, 11]
c = [3, 6, 9, 12]

merged = list(heapq.merge(a, b, c))
print(merged)  # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
# 前提：每个输入序列本身必须是有序的
```

---

## 28.7 本章小结

| 函数                 | 用途         | 复杂度     |
| -------------------- | ------------ | ---------- |
| `heappush/heappop`   | 堆的基本增删 | O(log n)   |
| `heapify`            | 列表转堆     | O(n)       |
| `nlargest/nsmallest` | Top-K 问题   | O(n log K) |
| `merge`              | 归并有序序列 | O(n) 惰性  |

---

# 第29章 bisect模块：有序序列的二分查找与插入维护

## 29.1 bisect 模块概述

`bisect` 模块提供了在**已排序列表**中进行高效二分查找和插入的工具。

```python
import bisect
```

---

## 29.2 核心函数

### 29.2.1 bisect_left 和 bisect_right

```python
import bisect

sorted_list = [1, 3, 3, 5, 7, 9]

# bisect_left：找到元素应插入的最左位置
print(bisect.bisect_left(sorted_list, 3))   # 1（在第一个3之前）
print(bisect.bisect_left(sorted_list, 4))   # 3（在5之前）
print(bisect.bisect_left(sorted_list, 0))   # 0（在开头）
print(bisect.bisect_left(sorted_list, 10))  # 6（在末尾）

# bisect_right（等价于 bisect）：找到应插入的最右位置
print(bisect.bisect_right(sorted_list, 3))  # 3（在最后一个3之后）
print(bisect.bisect(sorted_list, 3))        # 3（bisect 是 bisect_right 的别名）
```

### 29.2.2 insort_left 和 insort_right

```python
import bisect

# insort：二分查找 + 插入，保持列表有序
lst = [1, 3, 5, 7, 9]
bisect.insort(lst, 4)
print(lst)  # [1, 3, 4, 5, 7, 9]

bisect.insort(lst, 5)
print(lst)  # [1, 3, 4, 5, 5, 7, 9]（insort_right 默认在重复元素右边插入）

# insort_left 会插在重复元素的左边
lst2 = [1, 3, 5, 7, 9]
bisect.insort_left(lst2, 5)
print(lst2)  # [1, 3, 5, 5, 7, 9]
```

---

## 29.3 实际应用

### 29.3.1 用二分查找判断元素是否存在

```python
import bisect

def binary_search(sorted_list, target):
    i = bisect.bisect_left(sorted_list, target)
    return i < len(sorted_list) and sorted_list[i] == target

data = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
print(binary_search(data, 23))   # True
print(binary_search(data, 24))   # False
```

### 29.3.2 成绩等级映射

```python
import bisect

def grade(score):
    breakpoints = [60, 70, 80, 90]  # 分数线
    grades = "FDCBA"                 # 对应等级
    i = bisect.bisect(breakpoints, score)
    return grades[i]

print(grade(55))   # F
print(grade(65))   # D
print(grade(75))   # C
print(grade(85))   # B
print(grade(95))   # A
```

### 29.3.3 维护动态有序列表

```python
import bisect

# 维护一个始终有序的列表
class SortedList:
    def __init__(self):
        self._data = []

    def add(self, val):
        bisect.insort(self._data, val)

    def __repr__(self):
        return str(self._data)

    def __contains__(self, val):
        i = bisect.bisect_left(self._data, val)
        return i < len(self._data) and self._data[i] == val

sl = SortedList()
for x in [5, 2, 8, 1, 9, 3]:
    sl.add(x)
print(sl)           # [1, 2, 3, 5, 8, 9]
print(3 in sl)      # True
print(4 in sl)      # False
```

---

## 29.4 本章小结

| 函数                | 用途           | 复杂度                            |
| ------------------- | -------------- | --------------------------------- |
| `bisect_left/right` | 找插入位置     | O(log n)                          |
| `insort_left/right` | 插入并保持有序 | O(n)（查找 O(log n) + 插入 O(n)） |

---

# 第30章 itertools模块：组合枚举、迭代构造与惰性计算思想

## 30.1 什么是 itertools？

`itertools` 提供了一系列使用极其高效 C 代码实现的**迭代器构建工具**。所有函数都返回**迭代器**（惰性求值，Lazy Evaluation），它不会一次把全部数据装入内存，而是每次调用 `next()` 时才产生下一个数据。在处理海量组合或序列时，保证了时间和空间的极高效率。

---

## 30.2 无限迭代器

```python
from itertools import count, cycle, repeat

# count(start, step)：无限计数器
for i in count(10, 3):
    if i > 25:
        break
    print(i, end=" ")  # 10 13 16 19 22 25
print()

# cycle(iterable)：无限循环可迭代对象
colors = cycle(["红", "绿", "蓝"])
for i in range(7):
    print(next(colors), end=" ")  # 红 绿 蓝 红 绿 蓝 红
print()

# repeat(elem, n)：重复元素 n 次（不传 n 则无限）
print(list(repeat("hello", 3)))  # ['hello', 'hello', 'hello']
```

---

## 30.3 组合拼接类

```python
from itertools import chain, zip_longest, islice

# chain：串联多个可迭代对象
a = [1, 2, 3]
b = [4, 5]
c = [6, 7, 8, 9]
print(list(chain(a, b, c)))  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# chain.from_iterable：展平嵌套列表
nested = [[1, 2], [3, 4], [5, 6]]
print(list(chain.from_iterable(nested)))  # [1, 2, 3, 4, 5, 6]

# zip_longest：以最长者为准的 zip
print(list(zip_longest([1, 2, 3], "ab", fillvalue="?")))
# [(1, 'a'), (2, 'b'), (3, '?')]

# islice：对迭代器切片（不支持负索引）
print(list(islice(range(100), 5, 15, 3)))  # [5, 8, 11, 14]
```

---

## 30.4 排列组合类

```python
from itertools import product, permutations, combinations, combinations_with_replacement

# product：笛卡尔积
print(list(product("AB", "12")))
# [('A','1'), ('A','2'), ('B','1'), ('B','2')]

# 等价于多层 for 循环
print(list(product(range(2), repeat=3)))
# [(0,0,0),(0,0,1),(0,1,0),(0,1,1),(1,0,0),(1,0,1),(1,1,0),(1,1,1)]

# permutations：排列（顺序有关）
print(list(permutations("ABC", 2)))
# [('A','B'),('A','C'),('B','A'),('B','C'),('C','A'),('C','B')]

# combinations：组合（顺序无关）
print(list(combinations("ABC", 2)))
# [('A','B'), ('A','C'), ('B','C')]

# combinations_with_replacement：允许重复的组合
print(list(combinations_with_replacement("AB", 2)))
# [('A','A'), ('A','B'), ('B','B')]
```

---

## 30.5 过滤与分组类

```python
from itertools import takewhile, dropwhile, filterfalse, groupby

data = [1, 3, 5, 2, 4, 6, 7]

# takewhile：从头开始取，直到条件不满足
print(list(takewhile(lambda x: x < 5, data)))  # [1, 3]

# dropwhile：从头跳过，直到条件不满足
print(list(dropwhile(lambda x: x < 5, data)))  # [5, 2, 4, 6, 7]

# filterfalse：保留使函数返回 False 的元素（filter 的反面）
print(list(filterfalse(lambda x: x % 2, data)))  # [2, 4, 6]

# groupby：它按“连续相同键”分组；若想把同键元素完全聚合，通常要先排序
data2 = sorted(["a", "ab", "abc", "b", "bc", "c"], key=len)
for key, group in groupby(data2, key=len):
    print(f"长度{key}: {list(group)}")
# 长度1: ['a', 'b', 'c']
# 长度2: ['ab', 'bc']
# 长度3: ['abc']
```

---

## 30.6 累积类

```python
from itertools import accumulate
import operator

# accumulate：累积计算
nums = [1, 2, 3, 4, 5]
print(list(accumulate(nums)))              # [1, 3, 6, 10, 15] 前缀和
print(list(accumulate(nums, operator.mul)))# [1, 2, 6, 24, 120] 前缀积
print(list(accumulate(nums, max)))         # [1, 2, 3, 4, 5] 前缀最大值

# starmap：解包参数并映射
from itertools import starmap
pairs = [(2, 5), (3, 2), (10, 3)]
print(list(starmap(pow, pairs)))  # [32, 9, 1000] → pow(2,5), pow(3,2), pow(10,3)
```

---

## 30.7 本章小结

| 函数类别 | 代表函数                                  | 核心用途           |
| -------- | ----------------------------------------- | ------------------ |
| 无限迭代 | `count`, `cycle`, `repeat`                | 生成无限序列       |
| 拼接切片 | `chain`, `islice`, `zip_longest`          | 合并/截取迭代器    |
| 排列组合 | `product`, `permutations`, `combinations` | 枚举所有可能       |
| 过滤分组 | `takewhile`, `dropwhile`, `groupby`       | 条件筛选与分组     |
| 累积映射 | `accumulate`, `starmap`                   | 前缀计算、参数展开 |

---

# 第31章 容器与函数式工具协作：map、filter、reduce与推导式的关系

## 31.1 map()：对每个元素施加变换

```python
# map(func, iterable) → 迭代器
nums = [1, 2, 3, 4, 5]

# 每个元素平方
squared = list(map(lambda x: x**2, nums))
print(squared)  # [1, 4, 9, 16, 25]

# 等价的列表推导式
squared_v2 = [x**2 for x in nums]

# map 可以传多个可迭代对象
a = [1, 2, 3]
b = [10, 20, 30]
print(list(map(lambda x, y: x + y, a, b)))  # [11, 22, 33]
```

## 31.2 filter()：按条件过滤

```python
nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 保留偶数
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)  # [2, 4, 6, 8, 10]

# 等价推导式
evens_v2 = [x for x in nums if x % 2 == 0]

# filter(None, iterable)：过滤掉所有"假值"（0、空串、None等）
data = [0, 1, "", "hello", None, [], [1, 2], False, True]
print(list(filter(None, data)))  # [1, 'hello', [1, 2], True]
```

## 31.3 functools.reduce()：累积归约

```python
from functools import reduce

nums = [1, 2, 3, 4, 5]

# 累积求和: ((((1+2)+3)+4)+5) = 15
total = reduce(lambda acc, x: acc + x, nums)
print(total)  # 15

# 累积求积: 1*2*3*4*5 = 120
product = reduce(lambda acc, x: acc * x, nums)
print(product)  # 120

# 带初始值
total_with_init = reduce(lambda acc, x: acc + x, nums, 100)
print(total_with_init)  # 115

# 实际应用：展平嵌套列表
nested = [[1, 2], [3, 4], [5, 6]]
flat = reduce(lambda acc, x: acc + x, nested, [])
print(flat)  # [1, 2, 3, 4, 5, 6]
```

## 31.4 推导式 vs 函数式工具

| 操作      | 函数式写法             | 推导式写法                  | 推荐                              |
| --------- | ---------------------- | --------------------------- | --------------------------------- |
| 变换      | `map(f, x)`            | `[f(i) for i in x]`         | **推导式**（更 Pythonic）         |
| 过滤      | `filter(f, x)`         | `[i for i in x if f(i)]`    | **推导式**                        |
| 变换+过滤 | `map(f, filter(g, x))` | `[f(i) for i in x if g(i)]` | **推导式**（更简洁）              |
| 归约      | `reduce(f, x)`         | 无直接等价                  | `reduce`（或用 `sum` 等专用函数） |

```python
# Python 社区的共识是：推导式更 Pythonic
# Guido van Rossum（Python 之父）本人也更推荐推导式

# 但 map/filter 在以下场景仍然有价值：
# 1. 已有现成函数时，不需要写 lambda
names = ["alice", "bob", "charlie"]
print(list(map(str.upper, names)))  # ['ALICE', 'BOB', 'CHARLIE']
# 比推导式更简洁：不需要写 [s.upper() for s in names]

# 2. 链式处理时，函数式风格更清晰
from itertools import chain
result = sorted(
    filter(lambda x: x > 0,
        map(int, ["3", "-1", "5", "0", "-2", "4"]))
)
print(result)  # [3, 4, 5]
```

---

## 31.5 本章小结

1. `map`/`filter` 返回惰性迭代器，推导式返回完整列表/集合/字典。
2. **Python 社区惯例**：简单变换/过滤优先用推导式。
3. `reduce` 在 Python 3 中移入 `functools`，说明它不如 `sum`/`max` 等内置函数常用。
4. 选择原则：**可读性第一**，推导式和函数式只是风格差异，不存在绝对优劣。

---

