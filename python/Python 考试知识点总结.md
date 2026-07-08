# Python 考试知识点总结

> 覆盖：基础语法 → 容器 → 字符串/正则 → 函数 → 类型注解 → 面向对象 → 异常处理 → 文件操作 → 并发编程 → 网络编程 → NumPy → Pandas → map/reduce → 刷题技巧

---

## 一、Python 基础语法

### 1.1 变量与赋值

- Python 是**动态类型语言**，变量无须声明类型
- 变量本质是"标签"（引用），重新赋值即指向新对象
- 多变量赋值：`a, b, c = 1, 2, 3`（元组解包）；`x = y = z = 0`（同值）
- 交换变量：`a, b = b, a`，右边先构成元组再解包

### 1.2 数字类型

| 类型 | 要点 |
|------|------|
| `int` | 无大小限制；支持 `0b`/`0o`/`0x` 表示；可用 `1_000_000` 分隔 |
| `float` | 精度问题 `0.1+0.2 != 0.3`；用 `math.isclose()` 比较；精确计算用 `Decimal` |
| `complex` | 如 `3+4j` |

**算术运算符**：`+ - * / // % **` ；`/` 结果总是 float；`//` 向下取整（`-17//5 == -4`）

**类型转换**：`int(3.7)` 截断为 3（非四舍五入）；`round(2.5)` 银行家舍入 = 2

### 1.3 字符串基础

- **不可变**类型，不能通过索引修改
- 索引：正向从 0，反向从 -1；越界抛 `IndexError`
- 切片：`s[start:stop:step]`，左闭右开，不越界；`s[::-1]` 反转

**常用方法**：

| 方法 | 功能 |
|------|------|
| `strip()` / `lstrip()` / `rstrip()` | 去除空白/指定字符 |
| `upper()` / `lower()` / `title()` / `capitalize()` | 大小写转换 |
| `find(sub)` / `index(sub)` | 查找（find 返回 -1，index 抛异常） |
| `count(sub)` | 统计出现次数 |
| `replace(old, new, count)` | 替换（不修改原串） |
| `split(sep, maxsplit)` / `join(iter)` | 分割/拼接 |
| `startswith()` / `endswith()` | 前后缀判断 |
| `isdigit()` / `isalpha()` / `isalnum()` / `isspace()` | 字符类型判断 |

**重要区别**：`split()`（无参）按任意空白分割且忽略连续空白；`split(" ")` 按单个空格分割

### 1.4 字符串格式化

| 方式 | 推荐度 | 示例 |
|------|--------|------|
| f-string（3.6+） | **首选** | `f"{name}今年{age}岁"` |
| `.format()` | 备选 | `"{}今年{}岁".format(name, age)` |
| `%` 格式化 | 不推荐 | `"%s今年%d岁" % (name, age)` |

**f-string 格式控制**：`:.2f` 保留两位、`:.1%` 百分比、`:,` 千位分隔、`:^10` 居中、`:03d` 补零

### 1.5 布尔类型与真值判断

- `and` / `or` / `not` 有短路求值
- **Falsy 值**：`False`、`0`、`0.0`、`""`、`[]`、`()`、`{}`、`set()`、`None`
- `None` 判断永远用 `is None` / `is not None`

### 1.6 类型检查

- `type(x)` — 不考虑继承；`isinstance(x, Type)` — 考虑继承（**推荐**）
- `isinstance(True, int)` 为 `True`（bool 是 int 子类）

### 1.7 可变与不可变对象

| 不可变 | 可变 |
|--------|------|
| int, float, str, tuple, bool, None, frozenset, bytes | list, dict, set, bytearray |

- `is` 比较身份（内存地址），`==` 比较值（调用 `__eq__`）
- 可变对象 `y = x` 共享引用（别名陷阱）
- 函数参数传对象引用：内部修改可变对象会影响外部

### 1.8 输入输出

```python
# 输出
print(a, b, sep=" ", end="\n")

# 输入（返回值始终是字符串）
name = input("请输入：")
x, y = map(int, input().split())  # 一次读入多个值
```

### 1.9 条件控制

```python
if 条件:
    ...
elif 条件:
    ...
else:
    ...

# 三元表达式
x = a if 条件 else b

# 链式比较
if 1 < x < 10:  # 等价于 1 < x and x < 10

# match/case（3.10+）
match value:
    case 1:
        ...
    case _:  # default
        ...
```

### 1.10 循环控制

```python
# for 遍历可迭代对象
for item in iterable:
    ...

# range
range(stop)           # 0 ~ stop-1
range(start, stop, step)  # step 可为负数

# 循环控制
break     # 跳出整个循环
continue  # 跳过本次迭代
pass      # 占位符

# 常用搭配
enumerate(iter, start=0)   # 同时获取索引和值
zip(*iters)                # 并行遍历，以最短为准；3.10+ strict=True
for...else                 # 循环未被 break 中断时执行 else
```

---

## 二、数据容器

### 2.1 列表 `list`（有序、可变、可重复）

**创建**：`[]`、`list()`、列表推导式

**增删改查**：

| 操作 | 方法 | 说明 |
|------|------|------|
| 末尾加 | `append(x)` | 加一个元素 |
| 插入 | `insert(i, x)` | 在位置 i 插入 |
| 扩展 | `extend(iter)` | 逐个追加（vs `append` 加整个对象） |
| 拼接 | `+` / `*` | 返回新列表 |
| 按值删 | `remove(v)` | 删第一个匹配 |
| 按索引删 | `pop(i)` | 删并返回，默认最后一个 |
| 删元素 | `del lst[i]` | 语句 |
| 清空 | `clear()` | |
| 查找 | `index(v)` / `count(v)` / `in` | |
| 排序 | `sort(key=, reverse=)` | 原地；`sorted()` 返回新列表 |
| 反转 | `reverse()` | 原地；`reversed()` 返回迭代器 |

**列表推导式**：
```python
[x**2 for x in range(10) if x % 2 == 0]          # 带过滤
["偶数" if x%2==0 else "奇数" for x in range(5)]   # if-else 放前面
[x for row in matrix for x in row]                # 展平嵌套
```

**陷阱**：`[[0]*3]*3` 三行共享同一对象 → 用 `[[0]*3 for _ in range(3)]`

### 2.2 元组 `tuple`（有序、不可变、可重复）

- 创建：`(1,2,3)` 或直接 `1,2,3`（逗号是关键）；单元素须加逗号 `(42,)`
- 不可增删改，但内部可变对象可被修改；含可变元素的元组不可哈希
- 解包：`a, b, c = t`；`first, *middle, last`；`_` 表示丢弃
- 只有 `index()` 和 `count()` 两个方法
- 命名元组：`collections.namedtuple` 或 `typing.NamedTuple`

### 2.3 字典 `dict`（键值对，3.7+ 有序）

**创建**：`{k: v}`、`dict(k=v)`、`dict.fromkeys()`、字典推导式

**键的要求**：必须可哈希（hashable）；键唯一，后覆盖前

| 操作 | 方法 | 说明 |
|------|------|------|
| 增/改 | `d[key] = value` | |
| 安全读 | `d.get(key, default)` | 不存在返回 default |
| 设默认 | `d.setdefault(key, default)` | 不存在则设置 |
| 删 | `del d[key]` / `d.pop(key)` / `d.pop(key, default)` | |
| 检查 | `key in d` | O(1) |
| 遍历 | `d.keys()` / `d.values()` / `d.items()` | |
| 合并 | `d1 \| d2`（3.9+） / `{**d1, **d2}` | |

**陷阱**：`dict.fromkeys(keys, [])` 所有键共享同一个列表

### 2.4 集合 `set`（无序、可变、唯一）

- 创建：`{1,2,3}` 或 `set()`；**`{}` 是空字典**
- 元素必须可哈希；`True==1==1.0` 视为同一元素
- 操作：`add()`、`remove()`（报错）、`discard()`（不报错）、`pop()`（随机）

**集合运算**：`|` 并集、`&` 交集、`-` 差集、`^` 对称差、`<=` 子集、`>=` 超集

**典型应用**：去重 `list(set(lst))`；保序去重 `list(dict.fromkeys(lst))`；O(1) 成员测试

**`frozenset`**：不可变版本，可哈希，可作字典键

### 2.5 时间复杂度速查

| 操作 | list | tuple | dict | set |
|------|------|-------|------|-----|
| 索引访问 | O(1) | O(1) | — | — |
| 成员检测 `in` | O(n) | O(n) | O(1) | O(1) |
| 末尾追加 | O(1) | — | — | — |
| 头部插入 | O(n) | — | — | — |

### 2.6 collections 模块

| 类型 | 用途 |
|------|------|
| `deque` | 双端队列，两端 O(1)；`maxlen` 滑动窗口；`rotate(n)` |
| `defaultdict` | 访问不存在键自动创建默认值；慎用：仅查看也会创建 |
| `Counter` | 计数器；`most_common(n)`；支持 `+ - & \|` 运算 |
| `OrderedDict` | 额外功能：`move_to_end()`、`popitem()` |
| `ChainMap` | 多字典逐层查找 |
| `namedtuple` | 有字段名的元组 |

### 2.7 其他工具模块

| 模块 | 关键函数 | 用途 |
|------|---------|------|
| `heapq` | `heappush/pop`、`heapify`、`nlargest/nsmallest` | 最小堆 |
| `bisect` | `bisect_left/right`、`insort_left/right` | 二分查找/插入 |
| `itertools` | `count/cycle/repeat`、`chain`、`product/permutations/combinations`、`groupby`、`accumulate` | 迭代器工具 |

### 2.8 五大问题范式

| 问题 | 方案 |
|------|------|
| **排序** | `sorted()` / `list.sort()`；`key=` 多级排序；Timsort 稳定 |
| **查找** | `in` 运算符；生成器+`next()` 找第一个；高频用 dict 索引 |
| **统计** | `len/sum/min/max`；`Counter` 计数；`sum(1 for x in data if cond)` 条件计数 |
| **去重** | 不保序：`set()`；保序：`dict.fromkeys()` |
| **分组** | `defaultdict(list)` 最通用；`itertools.groupby` 需先排序 |

---

## 三、字符串与正则表达式

### 3.1 转义字符与原始字符串

| 转义 | 含义 |
|------|------|
| `\n` | 换行 |
| `\t` | Tab |
| `\\` | 反斜杠 |
| `\'` `\"` | 引号 |

原始字符串 `r"..."` 不转义反斜杠（正则中必须用）。限制：不能以单反斜杠结尾。

### 3.2 编码

- `ord(c)` → Unicode 码点；`chr(n)` → 字符
- `s.encode("utf-8")` → bytes；`b.decode("utf-8")` → str
- Python 3 源码默认 UTF-8；中文在 UTF-8 中占 3 字节
- `len(s)` 对 str 计字符数，对 bytes 计字节数

### 3.3 正则表达式（`re` 模块）

**元字符**：`.` 任意字符、`\d` 数字、`\w` 单词字符、`\s` 空白（大写取反）

**量词**：`*` 0+、`+` 1+、`?` 0/1、`{n}` 恰好 n、`{n,m}` n~m
默认贪婪，加 `?` 变非贪婪（如 `.*?`）

**其他**：`[abc]` 字符集、`[^abc]` 取反、`()` 捕获分组、`(?:)` 非捕获、`\1` 反向引用、`^` 开头、`$` 结尾、`|` 或

**常用函数**：

| 函数 | 用途 |
|------|------|
| `re.search(pattern, text)` | 搜索第一个匹配，返回 Match 或 None |
| `re.findall(pattern, text)` | 返回所有匹配列表；有分组返回元组列表 |
| `re.sub(pattern, repl, text)` | 替换；repl 可为函数 |
| `re.match(pattern, text)` | 只从开头匹配 |
| `re.fullmatch(pattern, text)` | 整个字符串完全匹配 |
| `re.split(pattern, text)` | 按模式分割 |
| `re.compile(pattern, flags)` | 预编译；flags 如 `re.IGNORECASE` |

---

## 四、函数

### 4.1 定义与调用

```python
def func(pos_only, /, normal, *, kw_only, **kwargs):
    """文档字符串（Google风格：Args/Returns/Raises）"""
    return 返回值  # 无 return 隐式返回 None
```

- `snake_case` 命名；先定义后调用
- 多返回值 `return a, b` 本质是返回元组

### 4.2 参数类型

| 参数类型 | 语法 | 说明 |
|----------|------|------|
| 位置参数 | `def f(a, b)` | 按顺序匹配 |
| 默认参数 | `def f(a, b=10)` | 必须在非默认参数之后 |
| `*args` | 打包为**元组** | 接收多余位置参数 |
| `**kwargs` | 打包为**字典** | 接收多余关键字参数 |
| 仅关键字 | `def f(*, a)` | `*` 之后的参数 |
| 仅位置 | `def f(a, /)` | `/` 之前的参数（3.8+） |

**参数解包**：`func(*list)` 和 `func(**dict)`

### 4.3 可变默认参数陷阱

```python
# ❌ 所有调用共享同一个列表
def f(a, lst=[]):
    ...

# ✅ 正确做法
def f(a, lst=None):
    if lst is None:
        lst = []
```

### 4.4 作用域（LEGB 规则）

Local → Enclosing → Global → Built-in

- `global`：函数内修改全局变量
- `nonlocal`：内层函数修改外层（非全局）变量
- 尽量不用 `global`，通过参数和返回值传递

### 4.5 参数传递机制

传对象引用：不可变对象"修改"=重新绑定（不影响外部）；可变对象原地修改会影响外部；重新赋值不影响外部。

### 4.6 Lambda 表达式

```python
lambda 参数: 表达式  # 只能一个表达式，不能多语句
```

适用场景：`sorted(key=...)`、`map()`、`filter()`。PEP 8 不建议 `f = lambda ...` 赋值。

### 4.7 递归

- 必须有 base case
- 默认递归深度约 1000 层
- 用 `@functools.lru_cache` 记忆化优化

### 4.8 装饰器

```python
def decorator(func):
    @functools.wraps(func)  # 保留原函数元信息
    def wrapper(*args, **kwargs):
        # 前置逻辑
        result = func(*args, **kwargs)
        # 后置逻辑
        return result
    return wrapper
```

`@decorator` 等价于 `func = decorator(func)`。内置：`@staticmethod`、`@classmethod`、`@property`

### 4.9 生成器

用 `yield` 代替 `return`，按需产出值，省内存。`next(gen)` 驱动，结束抛 `StopIteration`。

### 4.10 map / filter / reduce

```python
map(func, iter)       # 变换，返回迭代器
filter(func, iter)    # 过滤；filter(None, data) 过滤假值
reduce(func, iter, initial)  # 归约（from functools）
```

**选择指南**：有现成函数 → `map`；简单变换/过滤 → 推导式；复杂累积 → `reduce`；数据量大 → 惰性求值

---

## 五、类型注解

### 5.1 基本语法

```python
# 变量
name: str = "小明"
age: int = 20

# 函数
def add(a: int, b: int) -> int:
    return a + b

# 容器（3.9+）
scores: list[int] = [90, 85]
age_map: dict[str, int] = {"张三": 18}
point: tuple[int, int] = (3, 5)
tags: set[str] = {"Python"}

# 特殊类型
x: int | str = 10          # 联合类型（3.10+）
y: str | None = None       # Optional（3.10+）
```

### 5.2 关键规则

- 类型注解**不影响运行时**，是给 IDE 和 mypy 看的
- `bool` 是 `int` 子类，类型窄化时先检查 `bool` 再 `int`
- 类内部引用自身用字符串 `-> "ClassName"`
- 泛型：`T = TypeVar("T")` → `def first(items: list[T]) -> T`

---

## 六、面向对象编程

### 6.1 类与对象

```python
class Student:                      # 大驼峰命名
    school = "HDU"                  # 类属性（所有实例共享）

    def __init__(self, name, age):  # 构造方法，创建时自动调用
        self.name = name            # 实例属性
        self.age = age

    def study(self):                # 实例方法，self 自动传入
        print(f"{self.name}在学习")

    def __str__(self):              # print() 时自动调用
        return f"Student({self.name}, {self.age})"
```

### 6.2 封装

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner          # 公开
        self.__balance = balance    # 私有（__开头）

    def get_balance(self):          # getter
        return self.__balance

    def set_balance(self, amount):  # setter（可加验证逻辑）
        if amount >= 0:
            self.__balance = amount
```

- `__name` → 名称改写为 `_ClassName__name`（伪私有，不真安全）
- `_name` → 约定保护（"请别用"）
- `name` → 公开

### 6.3 魔术方法

| 方法 | 触发 | 用途 |
|------|------|------|
| `__init__` | 创建对象 | 初始化 |
| `__str__` | `print()` / `str()` | 可读字符串（**必须 return**） |
| `__repr__` | `repr()` / 交互环境 | 官方表示 |
| `__lt__` | `<` | 定义后 `sorted()` 可直接用 |
| `__le__` | `<=` | |
| `__eq__` | `==` | 默认比较内存地址 |
| `__add__` | `+` | 运算符重载 |
| `__len__` | `len()` | |

### 6.4 继承

```python
class Dog(Animal):                  # 单继承
    def __init__(self, name, breed):
        super().__init__(name)      # 必须调用父类构造！
        self.breed = breed

    def speak(self):                # 复写（Override）
        super().speak()             # 先调父类
        print("汪汪汪！")
```

**核心规则**：
- 子类 `__init__` 忘记 `super().__init__()` → 父类属性未初始化 → AttributeError
- 子类**不能**直接访问父类 `__私有` 成员
- `super()` 按 **MRO** 顺序查找下一个类（不是简单=父类）
- `object` 是所有类的最终祖先
- `issubclass(子, 父)` / `isinstance(obj, 类)` 判断关系

### 6.5 多继承与 MRO

```python
class D(B, C):  # 靠前的父类优先
    pass

D.__mro__  # D → B → C → A → object
```

**菱形继承**中 `super()` 按 MRO 链式调用，保证每个类只初始化一次。

### 6.6 多态

```python
def make_sound(animal: Animal):
    animal.speak()      # 同一调用，不同对象不同行为

make_sound(Dog())       # 汪汪汪！
make_sound(Cat())       # 喵喵喵！
```

**抽象类**：
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):    # 子类必须实现
        pass
```

**鸭子类型**：不强制继承关系，只要对象有对应方法就能用。"如果它走起来像鸭子，叫起来像鸭子，那它就是鸭子。"

### 6.7 dataclass（3.7+）

```python
from dataclasses import dataclass, field

@dataclass
class Student:
    name: str
    age: int
    scores: list = field(default_factory=list)  # 可变默认值必须用 default_factory
```

自动生成 `__init__`、`__repr__`、`__eq__`。

---

## 七、异常处理

### 7.1 基本结构

```python
try:
    # 可能出错的代码
except ValueError as e:
    # 处理 ValueError
except (TypeError, KeyError) as e:
    # 合并处理多种
else:
    # 没出错才执行
finally:
    # 无论如何都执行（return 之前）
```

### 7.2 核心规则

- **子类在前，父类在后**（匹配按 isinstance 规则）
- 裸 `except:` 捕获 BaseException（包括 Ctrl+C），**应避免**
- `else` 放"成功后才执行的代码"，缩小 try 范围
- `finally` 释放资源（现代代码优先用 `with`）

### 7.3 raise 与异常链

```python
raise ValueError("消息")
raise                   # 裸 raise 重新抛出（仅 except 内）
raise NewError("...") from e     # 保留原因链（推荐）
raise NewError("...") from None  # 隐藏原异常
```

### 7.4 自定义异常

```python
class InsufficientBalanceError(Exception):  # 继承 Exception
    def __init__(self, balance, amount):
        self.balance = balance
        super().__init__(f"余额{balance}不足支付{amount}")
```

### 7.5 常见内置异常

| 异常 | 触发 |
|------|------|
| `ZeroDivisionError` | 除以零 |
| `TypeError` | 类型不匹配 |
| `ValueError` | 类型对但值不合理 |
| `NameError` | 未定义变量 |
| `IndexError` | 列表下标越界 |
| `KeyError` | 字典键不存在 |
| `AttributeError` | 对象无此属性 |
| `FileNotFoundError` | 文件不存在 |

### 7.6 assert

```python
assert 条件, "消息"     # 条件为 False 抛 AssertionError
```

用于**内部不变量检查**和测试，不用做用户输入校验（`python -O` 关闭 assert）。

### 7.7 异常处理原则

- 校验用户输入用 `raise`；校验内部逻辑用 `assert`
- 包装底层异常用 `from e`
- 不静默吞错，至少 `logging.exception()`
- EAFP（直接干，出错再说）比 LBYL（先检查再操作）更 Pythonic

---

## 八、文件操作与编码

### 8.1 基本操作

```python
# 最佳实践
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()           # 读全部
    # f.readline()               # 读一行
    # f.readlines()              # 读所有行到列表
    # for line in f:             # 逐行迭代（大文件推荐）

with open("out.txt", "w", encoding="utf-8") as f:
    f.write("内容\n")
    # f.writelines(lines)        # 不会自动加换行
```

### 8.2 打开模式

| 模式 | 含义 | 文件不存在 | 文件存在 |
|------|------|-----------|----------|
| `r` | 只读 | 报错 | 从头读 |
| `w` | 覆盖写 | 创建 | 清空 |
| `a` | 追加 | 创建 | 末尾写 |
| `x` | 排他创建 | 创建 | 报错 |
| `r+` | 读写 | 报错 | 可读写 |
| `rb/wb/ab` | 二进制模式 | 同上 | 同上 |

### 8.3 路径操作

**pathlib（推荐）**：
```python
from pathlib import Path
p = Path("project") / "data" / "input.txt"
p.name        # 文件名（含扩展名）
p.stem        # 文件名（不含扩展名）
p.suffix      # 扩展名
p.parent      # 父目录
p.exists() / p.is_file() / p.is_dir()
p.read_text(encoding="utf-8") / p.write_text("内容")
p.mkdir(parents=True, exist_ok=True)
p.glob("*.py")       # 不递归匹配
p.rglob("*.py")      # 递归匹配（杀手级功能）
```

**os.path（传统）**：`os.path.join()`、`os.path.exists()`、`os.listdir()`、`os.makedirs()`

### 8.4 bytes 与编码

```python
# str ↔ bytes
"你好".encode("utf-8")     # → b'\xe4\xbd\xa0\xe5\xa5\xbd' (6字节)
b'\xe4\xbd\xa0'.decode("utf-8")  # → '你'

# 查看编码
ord("中")   # 20013
chr(20013)  # '中'
```

- UTF-8 变长编码：英文 1 字节，中文 3 字节
- 始终显式指定 `encoding="utf-8"`

### 8.5 大文件处理

```python
CHUNK_SIZE = 4096
with open("large.bin", "rb") as f_in, open("copy.bin", "wb") as f_out:
    while chunk := f_in.read(CHUNK_SIZE):  # 分块读写
        f_out.write(chunk)
```

### 8.6 shutil 常用操作

```python
shutil.copy(src, dst)       # 复制文件
shutil.copy2(src, dst)      # 复制+元数据
shutil.copytree(src, dst)   # 递归复制目录
shutil.rmtree(path)         # 递归删除（危险）
shutil.move(src, dst)       # 移动
```

---

## 九、并发编程

### 9.1 核心概念

| 概念 | 说明 |
|------|------|
| 进程 | OS 资源分配单位，独立内存 |
| 线程 | CPU 调度单位，共享内存 |
| GIL | CPython 全局锁：同一时刻仅一个线程执行 Python 字节码 |
| 并发 | 任务交替执行（单核） |
| 并行 | 任务同时执行（多核） |

### 9.2 多线程（threading）

```python
import threading

# 创建线程
t = threading.Thread(target=func, args=(arg1,), daemon=True)
t.start()
t.join(timeout=10)

# 互斥锁
lock = threading.Lock()
with lock:
    # 临界区
    ...

# 其他同步工具
sem = threading.Semaphore(5)    # 信号量（限流）
event = threading.Event()       # 事件通知
cond = threading.Condition()    # 条件变量
```

**线程安全队列**（生产者-消费者模式）：
```python
from queue import Queue
q = Queue(maxsize=10)
q.put(item)     # 满时阻塞
item = q.get()  # 空时阻塞
q.task_done()   # 标记完成（配合 q.join()）
```

### 9.3 线程池

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(func, arg) for arg in args]
    for future in as_completed(futures):  # 按完成顺序处理
        result = future.result(timeout=10)
```

### 9.4 多进程（multiprocessing）

```python
import multiprocessing

p = multiprocessing.Process(target=func, args=(arg,))
p.start()
p.join()

# 进程池
with multiprocessing.Pool(processes=4) as pool:
    results = pool.map(func, iterable)       # 阻塞，按序
    results = pool.map_async(func, iterable) # 异步
    results = pool.starmap(func, [(a,b), ...])  # 多参数
    results = pool.imap(func, iterable)      # 惰性迭代器
```

**重要**：多进程代码必须放在 `if __name__ == '__main__':` 内

### 9.5 进程间通信（IPC）

| 方式 | 特点 |
|------|------|
| `Queue` | 进程安全队列，支持多对多 |
| `Pipe` | 点对点管道，更快 |
| `Value/Array` | 共享内存，需加锁 |
| `Manager` | 代理模式，最慢但最方便 |

### 9.6 选择决策

| 场景 | 方案 |
|------|------|
| CPU 密集型 | 多进程（multiprocessing.Pool） |
| I/O 密集型 | 多线程 / 线程池 |
| 大量并发 | asyncio（异步编程） |

---

## 十、网络编程

### 10.1 Socket 基础

```python
import socket

# TCP 客户端
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((host, port))
sock.sendall(data.encode('utf-8'))
data = sock.recv(4096)
sock.close()

# TCP 服务端
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sock.bind((host, port))
sock.listen(5)
client, addr = sock.accept()  # 阻塞等待
```

### 10.2 TCP vs UDP

| 特性 | TCP | UDP |
|------|-----|-----|
| 连接 | 面向连接 | 无连接 |
| 可靠性 | 可靠有序 | 不可靠 |
| 边界 | 字节流（粘包） | 数据报（天然边界） |
| 场景 | 文件/HTTP | 视频/游戏/DNS |

### 10.3 粘包问题

TCP 字节流无消息边界，需设计长度前缀协议：4 字节长度 + 数据体

```python
import struct
struct.pack('!I', len(data))    # 打包长度（网络字节序）
struct.unpack('!I', header)[0]  # 解包长度
```

### 10.4 IO 多路复用

```python
import selectors
sel = selectors.DefaultSelector()
sel.register(sock, selectors.EVENT_READ, data=callback)
events = sel.select(timeout)  # 返回可读/可写的 socket 列表
```

`selectors` 自动选择平台最优实现（Linux=epoll, macOS=kqueue），支持上万并发。

### 10.5 日志系统

```python
import logging
logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)
logger.exception("消息")  # 带完整堆栈
```

---

## 十一、NumPy

```python
import numpy as np
```

### 11.1 数组创建

| 函数 | 说明 |
|------|------|
| `np.array(list)` | 从列表创建，可指定 `dtype` |
| `np.zeros(shape)` / `np.ones(shape)` | 全0/全1 |
| `np.full(shape, val)` | 填充指定值 |
| `np.eye(N)` | 单位矩阵 |
| `np.arange(start, stop, step)` | 类似 range（不含 stop） |
| `np.linspace(start, stop, num)` | 等间隔（包含 stop） |

### 11.2 数组属性

| 属性 | 说明 |
|------|------|
| `ndim` | 维度数 |
| `shape` | 形状元组 |
| `size` | 元素总数 |
| `dtype` | 数据类型 |
| `.T` | 转置 |

### 11.3 形状变换

```python
arr.reshape(3, -1)              # -1 自动推导
arr.flatten()                   # 始终返回副本
arr.ravel()                     # 尽量返回视图（更快）
arr[:, np.newaxis]              # 增加维度
arr.squeeze()                   # 删除长度为1的维度
```

### 11.4 拼接与分割

```python
np.concatenate([a, b], axis=0)  # 沿指定轴拼接
np.vstack([a, b]) / np.hstack([a, b])  # 垂直/水平堆叠
np.vsplit(arr, n) / np.hsplit(arr, n)  # 分割
```

### 11.5 运算与广播

**注意**：`*` 是逐元素乘法，`@` / `np.dot()` 才是矩阵乘法

**广播规则**：从后往前对齐维度，大小相同或一个为 1 则兼容；大小为 1 的维度被拉伸。

**聚合函数**：`np.sum/mean/std/var/max/min` + `axis=`。
- `axis=0`：沿行操作（对每列聚合，减少的是第 0 轴）
- `axis=1`：沿列操作（对每行聚合）
- 记忆：axis=你要"消失"的维度

**通用函数**：`np.sqrt/exp/log/sin/cos/round/floor/ceil/abs`

### 11.6 索引与切片

| 类型 | 返回 | 特点 |
|------|------|------|
| 基本切片 `arr[2:5]` | **视图** | 修改影响原数组 |
| 布尔索引 `arr[arr>5]` | **副本** | 安全 |
| 花式索引 `arr[[0,2,4]]` | **副本** | 安全 |

布尔条件必须用 `& | ~` 和括号：`arr[(arr>3) & (arr<8)]`

### 11.7 随机数

```python
rng = np.random.default_rng(seed=42)  # 新版推荐
rng.random(5)
rng.integers(0, 10, size=5)
rng.normal(0, 1, size=100)
```

### 11.8 常见陷阱

- 浮点数比较用 `np.isclose()` 而非 `==`
- `np.append` 很慢，应预分配数组
- `(3,)` 是一维，`(1,3)` 是二维
- 整体相等用 `np.array_equal()`，`==` 返回逐元素布尔数组

---

## 十二、Pandas

```python
import pandas as pd
```

### 12.1 Series（一维）

```python
s = pd.Series([10, 20, 30], index=['a', 'b', 'c'])
s['a']         # 按标签
s.iloc[0]      # 按位置
s['a':'c']     # 标签切片：**包含末尾**
s[0:2]         # 位置切片：不包含末尾
```

### 12.2 DataFrame（二维）

```python
df = pd.DataFrame({'姓名': ['张三','李四'], '年龄': [25, 30]})
```

**核心属性**：`shape`、`columns`、`index`、`dtypes`、`head(n)`、`tail(n)`、`info()`、`describe()`

### 12.3 loc vs iloc（考试重点）

| | loc | iloc |
|---|---|---|
| 含义 | 按**标签** | 按**整数位置** |
| 切片 | **包含**末尾 | **不包含**末尾 |

```python
df.loc[0, '姓名']         # 标签访问
df.iloc[0, 0]             # 位置访问
df.loc[0:2, '姓名':'年龄'] # 标签切片（包含末尾）
df.iloc[0:2, 0:2]         # 位置切片（不包含末尾）
```

### 12.4 数据读取

```python
pd.read_csv('data.csv', encoding='utf-8')
pd.read_excel('data.xlsx', sheet_name='Sheet1')
df.to_csv('out.csv', index=False)     # 几乎总需要 index=False
df.to_excel('out.xlsx', index=False)
```

### 12.5 数据操作

```python
# 选取
df['列名']                # 单列 → Series
df[['列1', '列2']]        # 多列 → DataFrame

# 增删改
df['新列'] = 表达式
df.drop(columns=['列名'])
df.rename(columns={'旧': '新'})  # 推荐赋值而非 inplace=True

# 排序
df.sort_values('列', ascending=False)
df.sort_values(['列1', '列2'], ascending=[True, False])
```

### 12.6 数据筛选

```python
# 必须用 & | ~ 和括号
df[(df['年龄'] > 25) & (df['工资'] > 10000)]

# 其他
df[df['城市'].isin(['北京', '上海'])]
df[df['姓名'].str.contains('三')]
df[df['年龄'].between(25, 30)]
df.query('年龄 > 25 and 工资 > 10000')   # query 更简洁
```

### 12.7 数据清洗

```python
# 缺失值
df.isnull().sum()                     # 检查（第一步）
df.dropna(subset=['工资'])            # 删除
df.fillna({'年龄': df['年龄'].mean()})  # 填充
df.ffill() / df.bfill()               # 前后填充

# 重复值
df.duplicated().sum()
df.drop_duplicates(subset=['姓名'])

# 类型转换
df['列'].astype(int)
pd.to_datetime(df['日期'])
pd.to_numeric(s, errors='coerce')     # 安全转换
```

### 12.8 分组聚合（groupby）

Split → Apply → Combine

```python
df.groupby('部门')['工资'].mean()
df.groupby(['部门', '城市'])['工资'].agg(['mean', 'sum', 'max'])
df.groupby('部门').agg({'工资': 'mean', '年龄': 'max'})

# 恢复索引
df.groupby('部门')['工资'].mean().reset_index()
```

### 12.9 数据合并

```python
pd.concat([df1, df2], ignore_index=True)  # 纵向拼接
pd.merge(left, right, on='key', how='left')  # SQL JOIN
# how: inner(默认), left, right, outer
```

### 12.10 时间序列

```python
df['日期'].dt.year / .dt.month / .dt.day / .dt.dayofweek
df.set_index('日期').resample('ME').sum()  # 按月重采样
# 频率：'D'日, 'W'周, 'ME'月末, 'QE'季度末, 'YE'年末
```

### 12.11 完整分析流程

1. `pd.read_csv()` → 2. `df.head()/info()/describe()` → 3. `df.isnull().sum()` → 4. `fillna()/dropna()` → 5. `drop_duplicates()` → 6. `astype()/to_datetime()` → 7. 新增计算列 → 8. 筛选 → 9. `groupby().agg()` → 10. `to_csv()`

---

## 十三、面向对象速查表

| 语法 | 含义 |
|------|------|
| `class A:` | 定义类 |
| `self` | 指向当前对象 |
| `__init__(self)` | 构造方法 |
| `__str__(self)` | `print(obj)` 触发 |
| `__lt__(self, other)` | `<` 比较 |
| `__eq__(self, other)` | `==` 比较 |
| `__name` | 私有成员（名称改写） |
| `class B(A):` | B 继承 A |
| `super().__init__()` | 调用父类构造 |
| `super().method()` | 调用父类方法 |
| `issubclass(B, A)` | B 是否是 A 子类 |
| `isinstance(obj, A)` | obj 是否是 A 实例 |
| `类.__mro__` | 方法解析顺序 |
| `ABC` + `@abstractmethod` | 抽象基类 |
| `@dataclass` | 自动生成 `__init__/__repr__/__eq__` |

---

## 十四、常见陷阱速查（20条）

1. **`input()` 返回字符串** — 运算前需类型转换
2. **浮点数精度** — `0.1+0.2 != 0.3`，用 `math.isclose()` 或 `Decimal`
3. **`{}` 是空字典** — 空集合必须 `set()`
4. **单元素元组需要逗号** — `(42,)` 而非 `(42)`
5. **可变默认参数** — `def f(lst=[])` 共享同一列表，用 `None` 哨兵
6. **二维列表 `*` 陷阱** — `[[0]*3]*3` 三行共享，用推导式
7. **浅拷贝只复制外层** — 嵌套结构用 `copy.deepcopy()`
8. **遍历时修改容器** — 先收集再操作或构建新容器
9. **`dict.fromkeys()` 值共享** — 可变默认值被所有键共享
10. **`True == 1 == 1.0`** — 集合/字典视作同一元素
11. **`is` vs `==`** — is 比较身份，`==` 比较值；None 用 `is`
12. **`append` vs `extend`** — 前者加整个对象，后者逐个追加
13. **`groupby` 需先排序** — `itertools.groupby` 只合并连续相同键
14. **子类 `__init__` 忘记 `super().__init__()`** — 父类属性未初始化
15. **`str.isidentifier()` 不是 C 标识符** — 接受大量 Unicode 字符
16. **map 迭代器只能遍历一次** — 提前 `list()` 转换
17. **reduce 空序列无初始值** — 抛 TypeError，始终提供第三个参数
18. **NumPy `*` 是逐元素乘** — 矩阵乘法用 `@` 或 `np.dot()`
19. **Pandas `loc` 切片包含末尾** — `iloc` 不包含
20. **CPU 密集用多线程无效** — GIL 限制，改用多进程

---

## 十五、HDU 刷题技巧速查

### 15.1 OJ 输入模式

```python
# EOF 模式
import sys
for line in sys.stdin:
    data = line.split()

# 先给组数 T
T = int(input())
for _ in range(T):
    ...

# n == 0 终止
while True:
    n = int(input())
    if n == 0:
        break
```

### 15.2 Python 内置函数速查

| 函数 | 用途 | 典型题 |
|------|------|--------|
| `math.hypot(dx, dy)` | 斜边，避免溢出 | 2001 |
| `math.isqrt(n)` | 精确整数平方根 | 2012, 2053 |
| `math.gcd(a, b)` | 最大公约数 | 2028 |
| `math.comb(n, k)` | 组合数 | 2032 |
| `pow(a, b, mod)` | 快速幂取模 | 2035 |
| `int(s, base)` | 任意进制转整数 | 2057 |
| `bin(n)[2:]` / `hex(n)[2:].upper()` | 转进制字符串 | 2051, 2057 |
| `divmod(a, b)` | 同时返回商和余数 | 2021, 2031, 2033 |
| `sorted(lst, key=abs, reverse=True)` | 按绝对值排序 | 2020 |
| `bisect.insort(lst, x)` | 有序插入 | 2019 |

### 15.3 关键算法模式

| 模式 | 核心思路 | 题号 |
|------|---------|------|
| 素数判定 | `math.isqrt()` + 跳偶数 | 2012 |
| 闰年 | `(y%4==0 and y%100!=0) or y%400==0` | 2005 |
| 回文 | `s == s[::-1]` | 2029 |
| 完全平方数 | 因子个数为奇数 ⇔ N 是完全平方数 | 2053 |
| 矩形交集 | `max(lefts)`, `min(rights)`，宽/高 ≤0 则无交集 | 2056 |
| 贪心找零 | `divmod` 从大到小 | 2021 |
| 杨辉三角 | `tri[i][j] = tri[i-1][j-1] + tri[i-1][j]` | 2032 |
| 逆向递推 | 蟠桃：`总 = (剩+1)*2` | 2013 |
| 字符串标准化 | 去前导零、尾部零、处理正负号 | 2054 |
