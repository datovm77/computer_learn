# Python `map` 与 `reduce` 完全教程

---

## 一、准备知识

在学习 `map` 和 `reduce` 之前，你必须先理解两个核心概念：**函数是一等公民** 和 **`lambda` 表达式**。它们是后续所有内容的基石。

### 1.1 函数是一等公民（First-Class Citizen）

在 Python 中，**函数和整数、字符串一样，都是"对象"**。这意味着函数可以：

- **赋值给变量**
- **作为参数传递给另一个函数**
- **作为另一个函数的返回值**
- **存储在列表、字典等数据结构里**

我们把拥有这些特性的函数称为"一等公民"。

#### 示例 1：把函数赋值给变量

```python
def greet(name):
    return f"你好，{name}！"

# 把函数赋值给变量 say_hello（注意：没有加括号！）
say_hello = greet

# 现在 say_hello 和 greet 指向同一个函数对象
print(say_hello("小明"))   # 输出：你好，小明！
print(greet("小明"))       # 输出：你好，小明！

# 验证：它们确实是同一个对象
print(say_hello is greet)  # 输出：True
```

> **关键区分**：`greet` 是函数对象本身，`greet("小明")` 是调用函数后得到的返回值。赋值时**不加括号**，是在传递函数本身。

#### 示例 2：函数作为参数传递

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def calculate(func, x, y):
    """接收一个函数和两个数字，用这个函数处理它们"""
    result = func(x, y)      # 调用传进来的函数
    return result

# 把 add 函数传进去
print(calculate(add, 10, 3))       # 输出：13

# 把 subtract 函数传进去
print(calculate(subtract, 10, 3))  # 输出：7
```

**理解要点**：`calculate` 并不知道也不关心传进来的是什么函数，它只知道"调用这个函数，传入 x 和 y"。这就是**高阶函数**的思想——接受函数作为参数的函数。

`map` 和 `reduce` 正是典型的高阶函数。

#### 示例 3：函数作为返回值

```python
def make_multiplier(factor):
    """返回一个新的函数，这个新函数会把参数乘以 factor"""
    def multiplier(x):
        return x * factor
    return multiplier   # 返回函数对象本身

double = make_multiplier(2)   # double 现在是一个函数
triple = make_multiplier(3)   # triple 现在也是一个函数

print(double(5))   # 输出：10
print(triple(5))   # 输出：15
```

#### 示例 4：函数存储在数据结构中

```python
def square(x):
    return x ** 2

def cube(x):
    return x ** 3

# 把函数放进列表
operations = [square, cube]

for func in operations:
    print(func(4))
# 输出：
# 16
# 64
```

### 1.2 `lambda` 表达式（匿名函数）

有时候你只需要一个简单的、一次性的小函数，为它单独写一个 `def` 显得很繁琐。这时可以用 `lambda` 创建一个**匿名函数**（没有名字的函数）。

#### 语法

```text
lambda 参数1, 参数2, ... : 表达式
```

- `lambda` 是关键字
- 冒号左边是参数（可以有多个，用逗号分隔）
- 冒号右边是**一个表达式**（不是语句！），这个表达式的值就是返回值
- **不能写 `return`**，表达式的结果会自动返回
- **不能写多行逻辑**（如 `if-else` 语句块、`for` 循环等）

#### 示例 1：最基础的 lambda

```python
# 普通函数写法
def add(a, b):
    return a + b

# 等价的 lambda 写法
add_lambda = lambda a, b: a + b

print(add(3, 5))         # 输出：8
print(add_lambda(3, 5))  # 输出：8
```

#### 示例 2：无参数的 lambda

```python
say_hi = lambda: "你好！"
print(say_hi())  # 输出：你好！
```

#### 示例 3：带条件表达式的 lambda

虽然 `lambda` 不能写 `if-else` 语句块，但可以使用 Python 的**条件表达式**（三元运算符）：

```python
# 判断奇偶
check = lambda x: "偶数" if x % 2 == 0 else "奇数"

print(check(4))  # 输出：偶数
print(check(7))  # 输出：奇数
```

> 条件表达式的语法：`值1 if 条件 else 值2`，这是一个**表达式**，所以可以在 `lambda` 中使用。

#### 示例 4：lambda 最常见的用途——作为参数传递

```python
# 对一组元组按第二个元素排序
students = [("小明", 88), ("小红", 95), ("小刚", 72)]

# key 参数需要一个函数，告诉 sorted 按什么排序
students_sorted = sorted(students, key=lambda student: student[1])

print(students_sorted)
# 输出：[('小刚', 72), ('小明', 88), ('小红', 95)]
```

#### lambda 的局限性

```python
# ❌ 不能有多行逻辑
# lambda x: 
#     if x > 0:
#         return x
#     else:
#         return -x

# ✅ 如果逻辑复杂，请老老实实写 def
def process(x):
    if x > 0:
        return x * 2
    elif x == 0:
        return 0
    else:
        return x * -1
```

#### 小结：什么时候用 lambda？

| 场景                                | 用 `def`   | 用 `lambda` |
| ----------------------------------- | ---------- | ----------- |
| 逻辑复杂（多行）                    | ✅          | ❌           |
| 需要复用、多次调用                  | ✅          | ❌           |
| 一次性的简单转换                    | 可以但啰嗦 | ✅           |
| 作为 `map`/`sorted`/`filter` 的参数 | 可以       | ✅ 更简洁    |

---

## 二、`map` 函数

### 2.1 什么是 `map`？

`map` 是 Python 的内置函数。它的作用一句话概括：

> **把一个函数依次作用到序列的每一个元素上，返回一个新的迭代器。**

你可以把 `map` 想象成一条流水线：每个元素进来，经过函数"加工"，出来一个新元素。

```
输入序列:   [1, 2, 3, 4, 5]
    ↓         ↓  ↓  ↓  ↓  ↓
函数 f(x):  f(1) f(2) f(3) f(4) f(5)
    ↓         ↓  ↓  ↓  ↓  ↓ 
输出:       [f(1), f(2), f(3), f(4), f(5)]
```

### 2.2 基本语法

```text
map(function, iterable)
```

- `function`：一个函数（可以是 `def` 定义的，也可以是 `lambda`）
- `iterable`：一个可迭代对象（列表、元组、字符串、range 等）
- **返回值**：一个 `map` 对象（迭代器），不是列表！

### 2.3 基础用法

#### 示例 1：将列表中的每个数平方

```python
numbers = [1, 2, 3, 4, 5]

def square(x):
    return x ** 2

result = map(square, numbers)

print(result)        # 输出：<map object at 0x...>（这是一个迭代器对象）
print(list(result))  # 输出：[1, 4, 9, 16, 25]（转成列表才能看到所有元素）
```

> **重要**：`map` 返回的是**迭代器**（惰性求值），不是列表。要查看所有结果，通常需要用 `list()` 转换。

#### 示例 2：将字符串列表转为整数列表

这是一个**超级常用**的场景：

```python
str_numbers = ["10", "20", "30", "40"]

# 用 map 把每个字符串转成整数
int_numbers = list(map(int, str_numbers))

print(int_numbers)  # 输出：[10, 20, 30, 40]
```

这里 `int` 本身就是一个函数（Python 的内置类型转换函数），直接传给 `map` 即可。

#### 示例 3：将每个单词首字母大写

```python
words = ["hello", "world", "python"]

capitalized = list(map(str.capitalize, words))

print(capitalized)  # 输出：['Hello', 'World', 'Python']
```

`str.capitalize` 是字符串的方法，本质上也是一个函数。

### 2.4 `map` 与 `lambda` 结合

大多数情况下，`map` 搭配 `lambda` 使用更加简洁：

```python
numbers = [1, 2, 3, 4, 5]

# 用 def 定义函数
def square(x):
    return x ** 2
result1 = list(map(square, numbers))

# 用 lambda 更简洁
result2 = list(map(lambda x: x ** 2, numbers))

print(result1)  # [1, 4, 9, 16, 25]
print(result2)  # [1, 4, 9, 16, 25]
```

更多例子：

```python
# 每个元素乘以 3
print(list(map(lambda x: x * 3, [1, 2, 3])))
# 输出：[3, 6, 9]

# 每个字符串加上问候语
names = ["小明", "小红", "小刚"]
print(list(map(lambda name: f"你好，{name}！", names)))
# 输出：['你好，小明！', '你好，小红！', '你好，小刚！']

# 每个数取绝对值
print(list(map(lambda x: abs(x), [-3, -1, 0, 2, 5])))
# 输出：[3, 1, 0, 2, 5]

# 上面这个其实可以直接传 abs（因为 abs 本身就是函数）
print(list(map(abs, [-3, -1, 0, 2, 5])))
# 输出：[3, 1, 0, 2, 5]
```

### 2.5 多序列用法

`map` 可以接收**多个可迭代对象**，此时函数也必须接收对应数量的参数：

```text
map(function, iterable1, iterable2, ...)
```

函数会同时从每个可迭代对象中取一个元素，组合起来传入函数。

#### 示例 1：两个列表对应元素相加

```python
a = [1, 2, 3]
b = [10, 20, 30]

result = list(map(lambda x, y: x + y, a, b))
print(result)  # 输出：[11, 22, 33]
```

执行过程：
```
第1次：lambda(1, 10)  → 11
第2次：lambda(2, 20)  → 22
第3次：lambda(3, 30)  → 33
```

#### 示例 2：三个列表的计算

```python
a = [1, 2, 3]
b = [4, 5, 6]
c = [7, 8, 9]

# 三个列表对应位置元素相乘
result = list(map(lambda x, y, z: x * y * z, a, b, c))
print(result)  # 输出：[28, 80, 162]
```

#### 示例 3：长度不等时怎么办？

当多个序列长度不同时，`map` **以最短的为准**，多出来的元素会被忽略：

```python
a = [1, 2, 3, 4, 5]
b = [10, 20, 30]

result = list(map(lambda x, y: x + y, a, b))
print(result)  # 输出：[11, 22, 33]
# a 中的 4 和 5 被忽略了
```

> **注意**：在 Python 2 中，`map` 对于长度不等的序列会用 `None` 填充，但 Python 3 已经改为截断。如果你需要填充行为，可以使用 `itertools.zip_longest`。

#### 示例 4：用 map + 多序列替代复杂操作

```python
# 计算多个学生的加权总分
names = ["小明", "小红", "小刚"]
math_scores = [90, 85, 78]
english_scores = [88, 92, 80]

# 总分 = 数学 * 0.6 + 英语 * 0.4
results = list(map(
    lambda name, math, eng: f"{name}: {math * 0.6 + eng * 0.4:.1f}",
    names, math_scores, english_scores
))

for r in results:
    print(r)
# 输出：
# 小明: 89.2
# 小红: 87.8
# 小刚: 78.8
```

### 2.6 `map` 返回的迭代器特性

这一点非常重要，很多初学者会踩坑：

```python
numbers = [1, 2, 3]
result = map(lambda x: x ** 2, numbers)

# 第一次遍历——正常
print(list(result))  # 输出：[1, 4, 9]

# 第二次遍历——空了！
print(list(result))  # 输出：[]
```

**原因**：`map` 返回的是**迭代器**，迭代器只能遍历一次。遍历完之后就"耗尽"了。如果需要多次使用结果，请先转成列表保存。

```python
# 正确做法：先转成列表
result = list(map(lambda x: x ** 2, [1, 2, 3]))
print(result)  # [1, 4, 9]
print(result)  # [1, 4, 9]  ← 列表可以多次访问
```

### 2.7 `map` 与列表推导式的对比

几乎所有 `map` 能做的事，列表推导式也能做，而且往往更加 Pythonic：

```python
numbers = [1, 2, 3, 4, 5]

# map 写法
result1 = list(map(lambda x: x ** 2, numbers))

# 列表推导式写法
result2 = [x ** 2 for x in numbers]

# 两者结果完全一样
print(result1)  # [1, 4, 9, 16, 25]
print(result2)  # [1, 4, 9, 16, 25]
```

> **Pythonic 建议**：如果只是简单的转换，列表推导式更清晰。但如果你已经有一个**现成的函数**（如 `int`、`str.upper`），使用 `map` 反而更简洁。

```python
# 有现成函数时，map 更简洁
list(map(int, ["1", "2", "3"]))    # ✅ 简洁
[int(x) for x in ["1", "2", "3"]]  # 也可以，但稍显啰嗦

# 需要写 lambda 时，列表推导式往往更好读
list(map(lambda x: x ** 2 + 1, range(5)))  # 可读性一般
[x ** 2 + 1 for x in range(5)]              # ✅ 更清晰
```

---

## 三、`reduce` 函数

### 3.1 什么是 `reduce`？

`reduce` 的作用一句话概括：

> **把一个序列"归约"成一个单一的值。**

它从左到右逐步把序列中的元素用函数"合并"在一起。

```
序列:     [a, b, c, d, e]

步骤 1:   f(a, b)     → result1
步骤 2:   f(result1, c) → result2
步骤 3:   f(result2, d) → result3
步骤 4:   f(result3, e) → 最终结果
```

**形象比喻**：想象你在做雪球——先用前两片雪花捏成小球，再把第三片雪花粘上去，继续滚，直到所有雪花都加入，最终得到一个大雪球。

### 3.2 导入

> **重要**：`reduce` 不是内置函数！它在 `functools` 模块中，使用前必须导入。

```python
from functools import reduce
```

在 Python 2 中 `reduce` 是内置函数，但 Python 3 把它移进了 `functools`，因为 Guido van Rossum（Python 之父）认为人们经常滥用它、导致代码可读性差。

### 3.3 基本语法

```text
reduce(function, iterable[, initializer])
```

- `function`：一个接收**两个参数**的函数
- `iterable`：可迭代对象
- `initializer`（可选）：初始值

### 3.4 基础用法

#### 示例 1：求一个列表所有元素的和

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

total = reduce(lambda x, y: x + y, numbers)
print(total)  # 输出：15
```

执行过程（逐步展开）：

```
初始：取出前两个元素 1, 2
步骤 1：lambda(1, 2)  = 3         ← 累积值变为 3
步骤 2：lambda(3, 3)  = 6         ← 累积值变为 6（3 + 下一个元素 3）
步骤 3：lambda(6, 4)  = 10        ← 累积值变为 10
步骤 4：lambda(10, 5) = 15        ← 最终结果
```

> 当然，求和直接用内置的 `sum()` 更好。这里只是用来演示 `reduce` 的工作原理。

#### 示例 2：求阶乘

```python
from functools import reduce

n = 5
factorial = reduce(lambda x, y: x * y, range(1, n + 1))
print(factorial)  # 输出：120（即 1 * 2 * 3 * 4 * 5）
```

执行过程：
```
range(1, 6) → [1, 2, 3, 4, 5]

步骤 1：1 * 2 = 2
步骤 2：2 * 3 = 6
步骤 3：6 * 4 = 24
步骤 4：24 * 5 = 120
```

#### 示例 3：找最大值

```python
from functools import reduce

numbers = [34, 12, 89, 23, 56]

max_val = reduce(lambda x, y: x if x > y else y, numbers)
print(max_val)  # 输出：89
```

执行过程：
```
步骤 1：max(34, 12) = 34
步骤 2：max(34, 89) = 89
步骤 3：max(89, 23) = 89
步骤 4：max(89, 56) = 89
```

### 3.5 初始值的作用

`reduce` 可以接收第三个参数作为**初始值**（`initializer`）。

#### 不带初始值 vs 带初始值

```python
from functools import reduce

numbers = [1, 2, 3, 4]

# 不带初始值：从第一个元素开始
result1 = reduce(lambda x, y: x + y, numbers)
# 步骤：1+2=3, 3+3=6, 6+4=10
print(result1)  # 输出：10

# 带初始值 100：从 100 开始累加
result2 = reduce(lambda x, y: x + y, numbers, 100)
# 步骤：100+1=101, 101+2=103, 103+3=106, 106+4=110
print(result2)  # 输出：110
```

#### 初始值的重要场景 1：空序列的安全处理

```python
from functools import reduce

empty_list = []

# ❌ 无初始值 + 空序列 → 报错！
# reduce(lambda x, y: x + y, empty_list)
# TypeError: reduce() of empty iterable with no initial value

# ✅ 有初始值 + 空序列 → 返回初始值
result = reduce(lambda x, y: x + y, empty_list, 0)
print(result)  # 输出：0
```

> **最佳实践**：如果序列可能为空，**一定要提供初始值**！

#### 初始值的重要场景 2：累积到不同类型

初始值还能让累积结果的类型与序列元素的类型不同：

```python
from functools import reduce

words = ["Hello", " ", "World", "!"]

# 把字符串列表拼接成一个字符串
sentence = reduce(lambda acc, word: acc + word, words, "")
print(sentence)  # 输出：Hello World!
```

```python
from functools import reduce

# 统计每个字符出现的次数
text = "hello"

def count_chars(acc, char):
    acc[char] = acc.get(char, 0) + 1
    return acc

result = reduce(count_chars, text, {})  # 初始值是空字典
print(result)  # 输出：{'h': 1, 'e': 1, 'l': 2, 'o': 1}
```

在上面的例子中，序列的元素是字符串类型，但累积结果是字典类型——这就是初始值为 `{}` 的作用。

### 3.6 更多典型案例

#### 案例 1：展平嵌套列表

```python
from functools import reduce

nested = [[1, 2], [3, 4], [5, 6]]

flat = reduce(lambda acc, sublist: acc + sublist, nested)
print(flat)  # 输出：[1, 2, 3, 4, 5, 6]
```

执行过程：
```
步骤 1：[1, 2] + [3, 4] = [1, 2, 3, 4]
步骤 2：[1, 2, 3, 4] + [5, 6] = [1, 2, 3, 4, 5, 6]
```

> **注意**：这种方式效率不高（每次 `+` 都创建了新列表），实际中遇到大量数据应该用 `itertools.chain.from_iterable` 或列表推导式。

#### 案例 2：数字列表拼成一个整数

```python
from functools import reduce

digits = [1, 3, 5, 7, 9]

number = reduce(lambda acc, d: acc * 10 + d, digits)
print(number)  # 输出：13579
```

执行过程：
```
步骤 1：1 * 10 + 3 = 13
步骤 2：13 * 10 + 5 = 135
步骤 3：135 * 10 + 7 = 1357
步骤 4：1357 * 10 + 9 = 13579
```

#### 案例 3：安全地访问嵌套字典

```python
from functools import reduce

data = {
    "user": {
        "profile": {
            "name": "小明",
            "age": 25
        }
    }
}

def safe_get(d, key):
    """安全取键：只在字典上取值，缺失时返回 None"""
    return d.get(key) if isinstance(d, dict) else None

# 按路径取值（存在的路径）
keys1 = ["user", "profile", "name"]
result1 = reduce(safe_get, keys1, data)
print(result1)  # 输出：小明

# 按路径取值（不存在的路径）
keys2 = ["user", "profile", "email"]
result2 = reduce(safe_get, keys2, data)
print(result2)  # 输出：None（不会抛 KeyError）
```

执行过程：
```
初始值：data（整个字典）
步骤 1：safe_get(data, "user") → {"profile": {"name": "小明", "age": 25}}
步骤 2：safe_get(..., "profile") → {"name": "小明", "age": 25}
步骤 3：safe_get(..., "name") → "小明"
若某一步键不存在，则返回 None，并在后续继续保持 None，不会抛异常
```

#### 案例 4：函数组合（管道）

```python
from functools import reduce

def add_one(x):
    return x + 1

def double(x):
    return x * 2

def square(x):
    return x ** 2

# 按顺序应用一系列函数
pipeline = [add_one, double, square]

result = reduce(lambda acc, func: func(acc), pipeline, 3)
print(result)  # 输出：64

# 过程：3 → add_one → 4 → double → 8 → square → 64
```

---

## 四、对比与选择

### 4.1 `map` vs `reduce` 的核心区别

| 特性             | `map`                     | `reduce`                       |
| ---------------- | ------------------------- | ------------------------------ |
| **输入**         | 一个/多个序列             | 一个序列                       |
| **输出**         | 一个新序列（迭代器）      | 一个单一值                     |
| **操作**         | 逐元素变换（一对一）      | 逐步累积（多对一）             |
| **函数参数个数** | 与序列数相同（通常 1 个） | 固定 2 个（累积值 + 当前元素） |
| **是否内置**     | ✅ 内置                    | ❌ 需从 `functools` 导入        |
| **返回类型**     | 迭代器                    | 取决于函数                     |

```python
from functools import reduce

numbers = [1, 2, 3, 4]

# map：每个元素翻倍 → 新序列
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)  # [2, 4, 6, 8]

# reduce：所有元素求和 → 单一值
total = reduce(lambda x, y: x + y, numbers)
print(total)    # 10
```

### 4.2 `map` vs 列表推导式

```python
numbers = [1, 2, 3, 4, 5]

# ----- 简单变换 -----
# map
list(map(lambda x: x ** 2, numbers))
# 列表推导式
[x ** 2 for x in numbers]           # ← 更清晰

# ----- 已有函数 -----
# map
list(map(int, ["1", "2", "3"]))     # ← 更简洁
# 列表推导式
[int(x) for x in ["1", "2", "3"]]

# ----- 需要过滤 -----
# map 不能过滤，需要配合 filter
list(map(lambda x: x**2, filter(lambda x: x > 2, numbers)))
# 列表推导式可以直接过滤
[x ** 2 for x in numbers if x > 2]  # ← 更清晰

# ----- 多重循环 -----
# map 很难做到
# 列表推导式很自然
[(x, y) for x in [1,2] for y in [3,4]]  # [(1,3),(1,4),(2,3),(2,4)]
```

### 4.3 `reduce` vs 内置函数 vs 循环

很多 `reduce` 的常见用法都有更好的替代：

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# 求和
reduce(lambda x, y: x + y, numbers)   # 可以，但...
sum(numbers)                            # ← 更好！

# 求最大值
reduce(lambda x, y: x if x > y else y, numbers)  # 可以，但...
max(numbers)                                       # ← 更好！

# 求最小值
reduce(lambda x, y: x if x < y else y, numbers)  # 可以，但...
min(numbers)                                       # ← 更好！

# 字符串拼接
reduce(lambda x, y: x + y, ["a", "b", "c"])  # 可以，但...
"".join(["a", "b", "c"])                       # ← 更好！

# 判断所有元素为真
reduce(lambda x, y: x and y, [True, True, False])  # 可以，但...
all([True, True, False])                             # ← 更好！
```

> **原则**：如果有对应的内置函数，优先用内置函数。`reduce` 适合那些**没有对应内置函数**的累积操作。

### 4.4 选择指南速查

```
需要对每个元素做变换？
├── 是 → 有现成函数吗（如 int、str.upper）？
│       ├── 是 → 用 map ✅
│       └── 否 → 用列表推导式 ✅
│
需要把序列归约成一个值？
├── 是 → 有内置函数吗（sum、max、min、any、all）？
│       ├── 是 → 用内置函数 ✅
│       └── 否 → 用 reduce ✅（或者用 for 循环）
│
需要过滤元素？
├── 是 → 用列表推导式（带 if）✅
│       （或 filter + map 组合）
│
需要同时变换 + 过滤？
└── 用列表推导式 ✅
```

---

## 五、组合进阶：`filter` + `map` + `reduce` 数据处理管道

### 5.1 `filter` 快速回顾

在构建管道之前，先快速了解 `filter`：

```text
filter(function, iterable)
```

- `function`：一个返回 `True` 或 `False` 的函数
- 保留返回 `True` 的元素，丢弃返回 `False` 的元素
- 返回迭代器

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 筛选偶数
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8, 10]

# 筛选非空字符串
words = ["hello", "", "world", "", "python"]
non_empty = list(filter(None, words))  # None 表示使用元素本身的真值
print(non_empty)  # ['hello', 'world', 'python']
```

### 5.2 管道思维：filter → map → reduce

数据处理往往是一个流水线：

```
原始数据
  ↓
filter（筛选）→ 留下符合条件的数据
  ↓
map（变换）→ 对每条数据做加工
  ↓
reduce（归约）→ 汇总成最终结果
```

#### 示例 1：计算及格学生的平均分

```python
from functools import reduce

scores = [45, 78, 92, 33, 88, 67, 54, 95, 71, 39]

# 步骤 1：filter — 筛选出及格的分数（>= 60）
passing = filter(lambda s: s >= 60, scores)

# 步骤 2：map — 统一转成 float，便于后续统计或格式化
passing_list = list(map(float, passing))

# 步骤 3：reduce — 求和
total = reduce(lambda acc, x: acc + x, passing_list, 0)

# 计算平均分
average = total / len(passing_list) if passing_list else 0
print(f"及格人数：{len(passing_list)}")   # 及格人数：6
print(f"及格平均分：{average:.1f}")       # 及格平均分：81.8
```

#### 示例 2：电商订单处理

```python
from functools import reduce

orders = [
    {"product": "手机",   "price": 2999, "quantity": 1, "status": "已完成"},
    {"product": "耳机",   "price": 199,  "quantity": 2, "status": "已完成"},
    {"product": "键盘",   "price": 399,  "quantity": 1, "status": "已取消"},
    {"product": "显示器", "price": 1599, "quantity": 1, "status": "已完成"},
    {"product": "鼠标",   "price": 89,   "quantity": 3, "status": "已完成"},
]

# 步骤 1：filter — 只要已完成的订单
completed = filter(lambda o: o["status"] == "已完成", orders)

# 步骤 2：map — 计算每笔订单的总金额
amounts = map(lambda o: o["price"] * o["quantity"], completed)

# 步骤 3：reduce — 求总金额
total_revenue = reduce(lambda acc, x: acc + x, amounts, 0)

print(f"总营收：¥{total_revenue}")  # 总营收：¥5263
```

执行过程：
```
filter 后：手机、耳机、显示器、鼠标（键盘被过滤）
map 后：[2999, 398, 1599, 267]
reduce 后：2999 + 398 + 1599 + 267 = 5263
# 实际输出：总营收：¥5263
```

#### 示例 3：文本数据清洗与统计

```python
from functools import reduce

raw_data = [
    "  Hello World  ",
    "",
    "  PYTHON programming  ",
    "   ",
    "data SCIENCE  ",
    "",
    "  Machine Learning  ",
]

# 步骤 1：map — 去除首尾空白并转为小写
cleaned = map(lambda s: s.strip().lower(), raw_data)

# 步骤 2：filter — 去除空字符串
non_empty = filter(lambda s: s != "", cleaned)

# 步骤 3：reduce — 统计总单词数
total_words = reduce(
    lambda acc, s: acc + len(s.split()),
    non_empty,
    0
)

print(f"总单词数：{total_words}")  # 总单词数：8
```

### 5.3 用函数封装管道

可以把管道封装成一个通用的函数：

```python
from functools import reduce

def pipeline(data, *functions):
    """
    将 data 依次通过一系列函数处理。
    每个函数接收上一步的输出作为输入。
    """
    return reduce(lambda acc, func: func(acc), functions, data)

# 定义各处理步骤
def remove_negatives(lst):
    return list(filter(lambda x: x >= 0, lst))

def double_all(lst):
    return list(map(lambda x: x * 2, lst))

def sum_all(lst):
    return sum(lst)

# 使用管道
data = [3, -1, 4, -1, 5, 9, -2, 6]
result = pipeline(data, remove_negatives, double_all, sum_all)
print(result)  # 输出：54

# 过程：
# [3, -1, 4, -1, 5, 9, -2, 6]
# → remove_negatives → [3, 4, 5, 9, 6]
# → double_all → [6, 8, 10, 18, 12]
# → sum_all → 54
```

### 5.4 对比：管道风格 vs 列表推导式风格

```python
from functools import reduce

numbers = range(1, 21)  # 1 到 20

# ===== 管道风格 =====
result_pipe = reduce(
    lambda acc, x: acc + x,
    map(
        lambda x: x ** 2,
        filter(lambda x: x % 2 == 0, numbers)
    ),
    0
)
# 读起来需要从内到外读：filter → map → reduce

# ===== 列表推导式 + sum =====
result_comp = sum(x ** 2 for x in numbers if x % 2 == 0)

print(result_pipe)  # 输出：1540
print(result_comp)  # 输出：1540
```

在这个例子中，列表推导式版本明显更易读。但是当处理步骤更多、更复杂时（尤其是每个步骤是预定义的函数），管道风格会更清晰。

---

## 六、实战 + 常见坑

### 6.1 实战练习

#### 练习 1：罗马数字转整数

```python
from functools import reduce

def roman_to_int(s):
    """将罗马数字字符串转换为整数"""
    roman = {'I': 1, 'V': 5, 'X': 10, 'L': 50,
             'C': 100, 'D': 500, 'M': 1000}
    
    # 步骤 1：map — 把每个罗马字符转成对应的整数
    values = list(map(lambda c: roman[c], s))
    
    # 步骤 2：处理"小值在大值左边要减去"的规则
    # 例如 IV = -1 + 5 = 4，IX = -1 + 10 = 9
    adjusted = list(map(
        lambda i: -values[i] if i < len(values) - 1 and values[i] < values[i + 1] else values[i],
        range(len(values))
    ))
    
    # 步骤 3：reduce — 求和
    return reduce(lambda acc, x: acc + x, adjusted, 0)

print(roman_to_int("III"))     # 输出：3
print(roman_to_int("IV"))      # 输出：4
print(roman_to_int("IX"))      # 输出：9
print(roman_to_int("XLII"))    # 输出：42
print(roman_to_int("MCMXCIV")) # 输出：1994
```

#### 练习 2：简易的 CSV 数据分析

```python
from functools import reduce

# 模拟 CSV 数据（第一行是标题）
csv_data = """姓名,语文,数学,英语
小明,85,92,78
小红,90,88,95
小刚,72,65,80
小芳,95,98,92
小李,60,55,70"""

def analyze_csv(csv_text):
    lines = csv_text.strip().split("\n")
    
    # 步骤 1：分离标题和数据
    header = lines[0].split(",")
    data_lines = lines[1:]
    
    # 步骤 2：map — 把每行转成字典
    records = list(map(
        lambda line: dict(zip(header, line.split(","))),
        data_lines
    ))
    # records 现在是：[{"姓名": "小明", "语文": "85", ...}, ...]
    
    # 步骤 3：map — 计算每人总分
    with_total = list(map(
        lambda r: {
            **r,  # 保留原有字段
            "总分": sum(map(int, [r["语文"], r["数学"], r["英语"]]))
        },
        records
    ))
    
    # 步骤 4：filter — 筛选总分 >= 240 的优秀学生
    excellent = list(filter(lambda r: r["总分"] >= 240, with_total))
    
    # 步骤 5：reduce — 计算优秀学生的平均总分
    avg_total = reduce(
        lambda acc, r: acc + r["总分"],
        excellent,
        0
    ) / len(excellent) if excellent else 0
    
    return {
        "all_students": with_total,
        "excellent_students": excellent,
        "excellent_average": avg_total,
    }

result = analyze_csv(csv_data)

print("=== 所有学生成绩 ===")
for s in result["all_students"]:
    print(f"  {s['姓名']}：总分 {s['总分']}")

print(f"\n=== 优秀学生（总分≥240） ===")
for s in result["excellent_students"]:
    print(f"  {s['姓名']}：总分 {s['总分']}")

print(f"\n优秀学生平均总分：{result['excellent_average']:.1f}")
```

输出：
```
=== 所有学生成绩 ===
  小明：总分 255
  小红：总分 273
  小刚：总分 217
  小芳：总分 285
  小李：总分 185

=== 优秀学生（总分≥240） ===
  小明：总分 255
  小红：总分 273
  小芳：总分 285

优秀学生平均总分：271.0
```

#### 练习 3：简易密码强度检测器

```python
from functools import reduce

def check_password_strength(password):
    """用 map + reduce 检测密码强度"""
    
    # 定义一系列检查规则，每个规则是一个 (描述, 检查函数, 分值) 的元组
    rules = [
        ("长度 >= 8",        lambda p: len(p) >= 8,                   2),
        ("包含大写字母",      lambda p: any(c.isupper() for c in p),   2),
        ("包含小写字母",      lambda p: any(c.islower() for c in p),   2),
        ("包含数字",          lambda p: any(c.isdigit() for c in p),   2),
        ("包含特殊字符",      lambda p: any(c in "!@#$%^&*" for c in p), 2),
    ]
    
    # map：对每条规则，检查是否通过，返回 (描述, 是否通过, 分值)
    check_results = list(map(
        lambda rule: (rule[0], rule[1](password), rule[2]),
        rules
    ))
    
    # reduce：累加通过的规则的分值
    total_score = reduce(
        lambda acc, result: acc + result[2] if result[1] else acc,
        check_results,
        0
    )
    
    # 输出详细报告
    print(f"密码：{'*' * len(password)}（长度 {len(password)}）")
    for desc, passed, score in check_results:
        status = "✅" if passed else "❌"
        print(f"  {status} {desc} (+{score}分)" if passed else f"  {status} {desc}")
    
    # 判断强度等级
    levels = {range(0, 4): "弱", range(4, 7): "中", range(7, 11): "强"}
    strength = next(v for k, v in levels.items() if total_score in k)
    print(f"  总分：{total_score}/10 —— 强度：{strength}")

check_password_strength("abc")
print()
check_password_strength("Hello123")
print()
check_password_strength("P@ssw0rd!")
```

### 6.2 常见坑与注意事项

#### 坑 1：`map` 返回的迭代器只能遍历一次

```python
result = map(lambda x: x * 2, [1, 2, 3])

# 第一次——有数据
print(list(result))  # [2, 4, 6]

# 第二次——空了！
print(list(result))  # []

# ✅ 解决方案：提前转成列表
result = list(map(lambda x: x * 2, [1, 2, 3]))
```

#### 坑 2：`reduce` 空序列不给初始值会报错

```python
from functools import reduce

# ❌ 报错
# reduce(lambda x, y: x + y, [])

# ✅ 提供初始值
reduce(lambda x, y: x + y, [], 0)  # 返回 0
```

#### 坑 3：`map` 中的 lambda 闭包陷阱

```python
# ❌ 经典坑
funcs = []
for i in range(5):
    funcs.append(lambda: i)  # 所有 lambda 都引用同一个变量 i

print([f() for f in funcs])  # [4, 4, 4, 4, 4]  ← 全是 4！

# ✅ 解决方案：用默认参数捕获当前值
funcs = []
for i in range(5):
    funcs.append(lambda i=i: i)  # 用默认参数绑定当前的 i 值

print([f() for f in funcs])  # [0, 1, 2, 3, 4]  ← 正确

# ✅ 或者继续用 map + 默认参数捕获
funcs = list(map(lambda i: (lambda i=i: i), range(5)))
print([f() for f in funcs])  # [0, 1, 2, 3, 4]
```

> 这个坑不仅限于 `map`，是 Python 闭包的通用问题。遇到在循环中创建 lambda/函数时，要格外小心。

#### 坑 4：`reduce` 的参数顺序

```python
from functools import reduce

# lambda 的两个参数有明确含义：
# 第一个参数：累积值（accumulator）
# 第二个参数：当前元素（current）

# ❌ 搞反了会出问题
items = [{"price": 10}, {"price": 20}, {"price": 30}]

# 这里 acc 是累积的数字，item 是字典
total = reduce(lambda acc, item: acc + item["price"], items, 0)
print(total)  # 60

# 如果写反了：
# reduce(lambda item, acc: item["price"] + acc, items, 0)
# 第一次调用时 item=0（初始值），acc={"price": 10}
# 0["price"] → TypeError!
```

#### 坑 5：不要滥用 `map` 产生副作用

```python
# ❌ 不要这样做
results = []
map(lambda x: results.append(x ** 2), [1, 2, 3])
print(results)  # [] ← 空的！因为 map 是惰性的，没有人消费它

# ❌ 即使强制执行了，这也不是 map 的正确用法
list(map(lambda x: results.append(x ** 2), [1, 2, 3]))
print(results)  # [1, 4, 9] ← 虽然"有效"了，但这是反模式

# ✅ 正确做法：用 for 循环
results = []
for x in [1, 2, 3]:
    results.append(x ** 2)

# ✅ 或者列表推导式
results = [x ** 2 for x in [1, 2, 3]]
```

> **原则**：`map` 用于**纯函数变换**（把 A 变成 B），不要用它来执行有副作用的操作（如修改外部变量、打印、写文件等）。

### 6.3 可读性与 Pythonic 建议

#### 建议 1：简单逻辑优先用列表推导式

```python
# ❌ 过度使用 map + lambda
list(map(lambda x: x ** 2, filter(lambda x: x > 3, range(10))))

# ✅ 列表推导式更清晰
[x ** 2 for x in range(10) if x > 3]
```

#### 建议 2：有现成函数时优先用 `map`

```python
numbers = [1, 2, 3, 4, 5]

# ✅ map + 现成函数 = 最简洁
list(map(str, [1, 2, 3]))       # ['1', '2', '3']
list(map(int, ['1', '2', '3'])) # [1, 2, 3]
list(map(len, ['hi', 'hello']))  # [2, 5]
```

#### 建议 3：有内置替代时不要用 `reduce`

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# ❌ 不需要 reduce
reduce(lambda x, y: x + y, numbers)   # → sum(numbers)
reduce(lambda x, y: x * y, numbers)   # → math.prod(numbers)  (Python 3.8+)
reduce(lambda x, y: x if x > y else y, numbers)  # → max(numbers)

# ✅ reduce 的好场景：没有内置替代的复杂累积
# 比如上面的字符频率统计、嵌套字典取值、函数管道等
```

#### 建议 4：复杂的 lambda 应该改用 `def`

```python
from functools import reduce

items = [
    {"category": "书籍", "name": "算法导论"},
    {"category": "电子产品", "name": "机械键盘"},
    {"category": "书籍", "name": "流畅的 Python"},
]

# ❌ 难以阅读的 lambda
result = reduce(
    lambda acc, x: {**acc, x["category"]: acc.get(x["category"], []) + [x["name"]]},
    items,
    {}
)

# ✅ 改用 def，加上注释
def group_by_category(acc, item):
    """将商品按分类分组"""
    category = item["category"]
    acc.setdefault(category, []).append(item["name"])
    return acc

result = reduce(group_by_category, items, {})
```

#### 建议 5：考虑生成器表达式的惰性优势

```python
# map 是惰性的（不会立即计算所有元素）
# 生成器表达式也是惰性的
# 列表推导式则会立即生成完整列表

# 处理大量数据时，惰性计算更节省内存
import sys

# 列表推导式：立刻在内存中创建 100 万个元素
big_list = [x ** 2 for x in range(1_000_000)]
print(sys.getsizeof(big_list))  # 仅列表容器本身大小（不含元素对象）

# map：惰性，几乎不占额外内存
big_map = map(lambda x: x ** 2, range(1_000_000))
print(sys.getsizeof(big_map))   # map 对象本身大小（与 Python 实现有关）

# 生成器表达式：同样惰性
big_gen = (x ** 2 for x in range(1_000_000))
print(sys.getsizeof(big_gen))   # 生成器对象本身大小（与 Python 实现有关）
```

> 补充说明：`sys.getsizeof()` 更适合比较**对象本身**的开销，不代表完整数据的总内存占用；这里的重点是说明“惰性对象不会立刻生成全部数据”。

### 6.4 `reduce` 边界行为补充（易错但常被忽略）

```python
from functools import reduce

# 1) 只有一个元素且没有初始值：直接返回该元素
print(reduce(lambda x, y: x + y, [42]))  # 42

# 2) 空序列 + 有初始值：返回初始值（不会调用函数）
print(reduce(lambda x, y: x + y, [], 100))  # 100

# 3) 空序列 + 无初始值：TypeError
# reduce(lambda x, y: x + y, [])
```

> 记忆口诀：**空序列必须给初始值；单元素序列可不写初始值。**

---

## 总结速查表

| 工具                   | 用途            | 输入 | 输出             | 典型搭配                       |
| ---------------------- | --------------- | ---- | ---------------- | ------------------------------ |
| `map(f, seq)`          | 逐元素变换      | 序列 | 迭代器（新序列） | `list()` 转换                  |
| `filter(f, seq)`       | 筛选元素        | 序列 | 迭代器（子集）   | `list()` 转换                  |
| `reduce(f, seq, init)` | 累积归约        | 序列 | 单个值           | `from functools import reduce` |
| 列表推导式             | 变换 + 筛选     | 序列 | 列表             | 替代简单 `map` + `filter`      |
| 生成器表达式           | 惰性变换 + 筛选 | 序列 | 生成器           | 大数据量时节省内存             |

**黄金法则**：
1. 有现成函数 → `map`
2. 简单变换/过滤 → 列表推导式
3. 复杂累积且无内置替代 → `reduce`
4. lambda 太复杂 → 改用 `def`
5. 数据量大 → 使用惰性求值（`map` 或生成器表达式）

---
