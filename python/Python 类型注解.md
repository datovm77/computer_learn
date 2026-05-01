# Python 类型注解详解

> **学习目标**
> 1. 理解类型注解的作用和意义
> 2. 掌握变量的类型注解
> 3. 掌握函数和方法的类型注解
> 4. 掌握 Union 联合类型注解

---

## 一、什么是类型注解？

### 1.1 Python 是动态类型语言

Python 是动态类型语言，变量不需要声明类型：

```python
x = 10        # x 是 int
x = "hello"   # x 变成了 str
x = [1, 2, 3] # x 变成了 list
```

这种灵活性是优点，但在大型项目中会带来问题：
- 不知道函数期望什么类型的参数
- 不知道函数返回什么类型的值
- IDE 无法提供准确的代码补全
- 类型相关的 bug 只能在运行时发现

### 1.2 类型注解是什么？

**类型注解**（Type Annotation）是 Python 3.5+ 引入的语法，允许我们为变量和函数添加类型信息。

```python
# 不使用类型注解
def greet(name):
    return f"Hello, {name}!"

# 使用类型注解
def greet(name: str) -> str:
    return f"Hello, {name}!"
```

**重要：** 类型注解**不会影响程序运行**！Python 解释器会忽略它们。它们的作用是：
- 让 IDE 提供更好的代码补全和错误检查
- 让代码更易读，明确函数的输入输出
- 配合静态类型检查工具（如 mypy）在运行前发现类型错误

### 1.3 启用类型检查

类型注解本身不会报错，需要使用静态类型检查工具：

```bash
# 安装 mypy
pip install mypy

# 检查类型
mypy your_script.py
```

#### mypy 实际使用示例

下面通过一个完整例子展示 mypy 如何帮你抓错：

```python
# example.py
def add(a: int, b: int) -> int:
    return a + b

result = add(1, 2)      # 正确：int + int
result = add("hi", 2)   # 错误！str 不能传给 int 参数
```

在终端运行：

```bash
mypy example.py
# 输出:
# example.py:5: error: Argument 1 to "add" has incompatible type "str"; expected "int"
```

mypy 只检查、不运行。它会在你运行程序之前就告诉你哪里类型不对。

#### 新手常见错误

**错误 1：以为类型注解会让程序报错**

```python
name: int = "hello"  # 程序照常运行，不会报错！
# 类型注解只是"标签"，Python 解释器完全忽略它
# 只有 mypy 等工具才会检查
```

**错误 2：忘记导入需要的类型**

```python
# 错误写法（Python 3.8 及以下，函数参数/返回值中会报 TypeError）
numbers: list[int] = [1, 2, 3]  # 注意：变量注解中 3.8 不报错，但函数签名中会报错

# 正确写法（Python 3.8 及以下）
from typing import List
numbers: List[int] = [1, 2, 3]

# Python 3.9+ 可以直接用 list[int]
```

**错误 3：把类型注解当类型转换**

```python
age: int = "25"  # 这不会把 "25" 转成 25！
# age 的值仍然是字符串 "25"
# 类型注解 ≠ 类型转换，它只是告诉工具"我期望这里是 int"
```

#### 小练习

1. 安装 mypy：`pip install mypy`
2. 创建一个文件，写一个类型错误的函数，用 mypy 检查它
3. 尝试给一个变量同时赋不同类型的值，观察 mypy 的输出

---

## 二、变量的类型注解

### 2.1 基本语法

```python
# 语法: 变量名: 类型 = 值

name: str = "张三"
age: int = 25
height: float = 1.75
is_student: bool = True
```

也可以不赋值，只声明类型：

```python
name: str
age: int

# 后续赋值
name = "张三"
age = 25
```

### 2.2 常见基本类型

```python
# 整数
count: int = 42

# 浮点数
price: float = 9.99

# 字符串
message: str = "Hello"

# 布尔值
is_valid: bool = True

# 字节
data: bytes = b"hello"
```

#### 通俗理解

你可以把类型注解想象成给盒子贴标签：

- 你有一个盒子（变量），标签上写着"里面装整数"（`int`）
- 如果你往里面放了一个字符串，标签和内容就不匹配了
- mypy 就是那个检查标签和内容是否匹配的人

#### 新手常见错误

**错误 1：int 和 float 混淆**

```python
# Python 中 int 和 float 是不同类型
price: float = 10       # 可以！int 可以赋给 float（PEP 484 规定的数值类型兼容）
count: int = 3.14       # mypy 会报错！float 不能赋给 int
count: int = int(3.14)  # 正确：显式转换
```

**错误 2：bool 和 int 混淆**

```python
# Python 中 bool 是 int 的子类型（True == 1, False == 0）
flag: bool = 1      # mypy 不会报错（但不推荐，应该用 True/False）
flag: bool = True   # 推荐写法
num: int = True     # mypy 不会报错（bool 是 int 的子类型）
```

**错误 3：None 赋给非 Optional 类型**

```python
name: str = None       # mypy 会报错！str 不允许 None
name: str | None = None  # 正确：用 Optional 或 str | None（Python 3.10+）
# 旧写法：from typing import Optional; name: Optional[str] = None
# Optional 会在 2.5 节详细讲解
```

#### 小练习

判断以下哪些写法 mypy 会报错：

```python
a: int = 42
b: float = 42
c: int = 42.0
d: str = 42
e: bool = 0
f: int = True
```

**参考答案：**

- `a: int = 42` — 正确
- `b: float = 42` — 正确（int 可以赋给 float）
- `c: int = 42.0` — **报错**（float 不能赋给 int）
- `d: str = 42` — **报错**（int 不能赋给 str）
- `e: bool = 0` — 正确（int 可以赋给 bool，但不推荐）
- `f: int = True` — 正确（bool 是 int 子类型，但不推荐）


---

### 2.3 容器类型注解

#### 列表（List）

```python
from typing import List

# 方式一：使用 typing.List（Python 3.8 及以下）
numbers: List[int] = [1, 2, 3, 4, 5]
names: List[str] = ["Alice", "Bob", "Charlie"]

# 方式二：直接使用 list（Python 3.9+）
numbers: list[int] = [1, 2, 3, 4, 5]
names: list[str] = ["Alice", "Bob", "Charlie"]
```

#### 元组（Tuple）

```python
from typing import Tuple

# 固定长度的元组，每个位置有固定类型
point: Tuple[int, int] = (10, 20)
person: Tuple[str, int, float] = ("张三", 25, 1.75)

# Python 3.9+
point: tuple[int, int] = (10, 20)
```

#### 字典（Dict）

```python
from typing import Dict

# 键是 str，值是 int
scores: Dict[str, int] = {"数学": 90, "英语": 85}

# Python 3.9+
scores: dict[str, int] = {"数学": 90, "英语": 85}

# 复杂的字典类型
user_info: dict[str, str | int] = {
    "name": "张三",
    "age": 25
}
```

#### 集合（Set）

```python
from typing import Set

unique_ids: Set[int] = {1, 2, 3, 4, 5}

# Python 3.9+
unique_ids: set[int] = {1, 2, 3, 4, 5}
```

#### 通俗理解

容器类型注解就像描述一个"有规格的收纳盒"：

- `list[int]` = 一个只能放整数的收纳盒
- `dict[str, int]` = 一个柜子，每个抽屉有名字（str），里面放整数
- `tuple[str, int]` = 一个固定格子的托盘，第一格放字符串，第二格放整数
- `set[int]` = 一个不允许重复的整数集合

#### 新手常见错误

**错误 1：列表里混放不同类型**

```python
# 如果你声明了 list[int]，mypy 会检查每个元素
numbers: list[int] = [1, 2, "three"]  # mypy 报错！"three" 不是 int

# 如果确实需要混合类型，用 Union
from typing import Union
mixed: list[Union[int, str]] = [1, "two", 3]  # OK
mixed: list[int | str] = [1, "two", 3]         # Python 3.10+ 更简洁
```

**错误 2：字典的键类型不对**

```python
# 字典键必须是声明的类型
scores: dict[str, int] = {"math": 90}
scores[123] = 100   # mypy 报错！键应该是 str，不是 int
scores["eng"] = 85  # OK
```

**错误 3：元组长度或类型不匹配**

```python
from typing import Tuple

# 元组注解规定了每个位置的类型和长度
point: Tuple[int, int] = (1, 2)         # OK
point: Tuple[int, int] = (1, 2, 3)      # mypy 报错！多了个元素
point: Tuple[int, int] = (1, "hello")   # mypy 报错！第二个应该是 int

# 如果元组长度不定但类型相同，用 ... 语法
numbers: Tuple[int, ...] = (1,)          # OK
numbers: Tuple[int, ...] = (1, 2, 3)    # OK
```

**错误 4：混淆 list 和 tuple 的语义**

```python
# list：同类型、可变长度  → 装一堆同种东西
# tuple：固定结构、固定长度 → 装一组有固定含义的东西

# 好的用法
names: list[str] = ["Alice", "Bob"]           # 一堆名字
person: tuple[str, int] = ("Alice", 25)       # 一个（姓名, 年龄）对

# 不推荐的用法
person: list[str | int] = ["Alice", 25]       # 丢失了结构信息
```

#### 小练习

1. 声明一个字典，键是学生名字（str），值是他们的三门课成绩列表（list[float]）
2. 声明一个元组，表示一个二维坐标点（float, float）
3. 声明一个列表，里面可以放 int 或 str

**参考答案：**

```python
# 1. 学生成绩字典
grades: dict[str, list[float]] = {
    "Alice": [90.5, 85.0, 92.0],
    "Bob": [78.0, 82.5, 88.0]
}

# 2. 二维坐标
point: tuple[float, float] = (3.14, 2.71)

# 3. 混合类型列表
mixed: list[int | str] = [1, "two", 3, "four"]
```


---

### 2.4 嵌套容器类型

```python
from typing import List, Dict

# 列表的列表（二维列表）
matrix: List[List[int]] = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

# 字典，值是列表
grade_book: Dict[str, List[int]] = {
    "Alice": [90, 85, 92],
    "Bob": [78, 82, 88]
}

# Python 3.9+
matrix: list[list[int]] = [[1, 2], [3, 4]]
grade_book: dict[str, list[int]] = {"Alice": [90, 85]}
```

#### 通俗理解

嵌套容器就像"套娃"：

- `list[list[int]]` = 一个大盒子里装着小盒子，每个小盒子里装整数（比如矩阵）
- `dict[str, list[int]]` = 一个柜子，每个抽屉里放着一个整数列表（比如学生成绩表）

嵌套多少层都可以，但层数越多，代码越难读。一般超过两层就该考虑用类（class）来组织数据了。

#### 新手常见错误

**错误：嵌套类型写错层级**

```python
# 错误：这不是"字典列表"，而是"列表的字典"
data: dict[str, list] = {"a": [1, 2]}  # list 没写元素类型！

# 正确：明确每一层的类型
data: dict[str, list[int]] = {"a": [1, 2]}

# 如果想要"字典的列表"
data: list[dict[str, int]] = [{"score": 90}, {"score": 85}]
```

#### 小练习

声明以下类型（不看答案先自己写）：

1. 一个三维矩阵（list 嵌套三层，元素是 float）
2. 一个通讯录：键是姓名（str），值是电话号码列表（list[str]）
3. 一个表格：每行是一个字典，字典的键是列名（str），值是任意类型

**参考答案：**

```python
from typing import Any

# 1. 三维矩阵
matrix_3d: list[list[list[float]]] = [[[1.0, 2.0], [3.0, 4.0]], [[5.0, 6.0], [7.0, 8.0]]]

# 2. 通讯录
contacts: dict[str, list[str]] = {
    "Alice": ["13800001111", "13900002222"],
    "Bob": ["13700003333"]
}

# 3. 表格（如果列值类型不确定，用 Any）
table: list[dict[str, Any]] = [
    {"name": "Alice", "age": 25, "scores": [90, 85]},
    {"name": "Bob", "age": 30, "scores": [88, 92]}
]
```


---

### 2.5 可选类型（Optional）

当变量可能是某个类型，也可能是 `None` 时：

```python
from typing import Optional

# name 可能是 str，也可能是 None
name: Optional[str] = None

# 等价于
name: str | None = None  # Python 3.10+

# 赋值
name = "张三"  # OK
name = None    # OK
```

**示例：**

```python
from typing import Optional


def find_user(user_id: int) -> Optional[str]:
    """查找用户，可能找到也可能找不到"""
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)  # 找到返回名字，找不到返回 None


result = find_user(1)    # result 的类型是 Optional[str]
result = find_user(999)  # result 是 None
```

#### 通俗理解

`Optional[str]` 就是说："这个值可能是字符串，也可能是空的（None）"。

现实类比：你去超市买牛奶，结果可能买到（`str`），也可能卖完了（`None`）。`Optional[str]` 就是"买牛奶的结果"。

#### 新手常见错误

**错误 1：忘记处理 None 的情况**

```python
from typing import Optional

def find_user(user_id: int) -> Optional[str]:
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)

# 危险！直接用返回值，没检查 None
name = find_user(999)
print(name.upper())  # 运行时报错：AttributeError: 'NoneType' has no attribute 'upper'

# 正确做法：先检查
name = find_user(999)
if name is not None:
    print(name.upper())  # 安全
else:
    print("用户不存在")
```

**错误 2：Optional 和 Union 的等价关系不理解**

```python
from typing import Optional, Union

# 以下三种写法完全等价
x: Optional[str]        # 推荐：简洁明了
x: Union[str, None]     # 也行，但更啰嗦
x: str | None           # Python 3.10+，最简洁

# Optional[int] 就是 Union[int, None]，不是"可选参数"的意思！
# 它说的是"值可能是 int，也可能是 None"
```

**错误 3：把 Optional 当成"有默认值"**

```python
# Optional 不是"参数可选"的意思！
def greet(name: Optional[str]) -> str:   # name 仍然必填！只是可以传 None
    return f"Hello, {name}"

greet()           # 报错！缺少参数
greet(None)       # OK，输出 "Hello, None"
greet("Alice")    # OK，输出 "Hello, Alice"

# 如果想让参数可选（有默认值），直接给默认值
def greet(name: str = "World") -> str:   # name 可以不传，默认 "World"
    return f"Hello, {name}"

greet()           # OK，输出 "Hello, World"
```

#### 小练习

1. 写一个函数 `safe_divide(a: float, b: float) -> Optional[float]`，除数为 0 时返回 None
2. 调用该函数后，安全地打印结果（处理 None 的情况）
3. 用 mypy 检查你写的代码

**参考答案：**

```python
from typing import Optional

def safe_divide(a: float, b: float) -> Optional[float]:
    """安全除法，除数为 0 返回 None"""
    if b == 0:
        return None
    return a / b

# 安全使用
result = safe_divide(10, 3)
if result is not None:
    print(f"结果: {result:.2f}")   # 结果: 3.33
else:
    print("除数不能为 0")

result = safe_divide(10, 0)
if result is not None:
    print(f"结果: {result:.2f}")
else:
    print("除数不能为 0")  # 除数不能为 0
```


---

### 2.6 Any 类型

`Any` 表示任意类型，关闭类型检查：

```python
from typing import Any

# 可以赋值任何类型
data: Any = 42
data = "hello"
data = [1, 2, 3]

# 用于不确定类型的情况
def process(data: Any) -> None:
    print(data)
```

**注意：** 尽量避免使用 `Any`，它会绕过类型检查，失去类型注解的意义。

#### 通俗理解

`Any` 就是"我不在乎类型"的标签。它相当于关闭了类型检查。

类比：其他类型注解是"这个盒子里只能放苹果"，`Any` 是"随便放什么都行"。

#### 什么时候可以谨慎使用 Any

```python
from typing import Any

# 场景 1：对接第三方库，库没有类型注解
def call_legacy_api(data: Any) -> Any:
    # 旧 API 没有类型信息，暂时用 Any
    return legacy_process(data)

# 场景 2：调试阶段，先让代码跑起来
def debug_print(value: Any) -> None:
    print(type(value), value)
```

#### 新手常见错误：过度使用 Any

```python
# 差：到处用 Any，等于没写注解
def process(data: Any) -> Any:
    return data  # 完全没有类型信息

# 好：尽可能给出具体类型
def process(data: list[int]) -> int:
    return sum(data)
```

**原则：先写具体类型，实在不行才用 `Any`。**

---

### 2.7 类型别名

当类型注解太长时，可以创建别名：

```python
from typing import Dict, List, Tuple

# 创建类型别名
Vector = List[float]
Matrix = List[List[float]]
UserDict = Dict[str, Dict[str, str | int]]

# 使用别名
def scale_vector(vector: Vector, scalar: float) -> Vector:
    return [x * scalar for x in vector]

def create_matrix(rows: int, cols: int) -> Matrix:
    return [[0.0] * cols for _ in range(rows)]

# Python 3.12+ 使用 type 语句
type Vector = list[float]
type Matrix = list[list[float]]
```

#### 通俗理解

类型别名就是给复杂类型起个简短的名字。就像"张三丰"是本名，"张真人"是别名——指的都是同一个人。

```python
# 没有别名：每次都要写一长串
def process(data: dict[str, list[tuple[int, float]]]) -> None: ...

# 有别名：简洁多了
UserData = dict[str, list[tuple[int, float]]]
def process(data: UserData) -> None: ...
```

---

### 附录：typing 模块常用类型导入速查表

| 类型 | 导入方式 | Python 版本要求 | 说明 |
|------|---------|----------------|------|
| `List` | `from typing import List` | 3.5+ | 列表类型（3.9+ 可直接用 `list`） |
| `Dict` | `from typing import Dict` | 3.5+ | 字典类型（3.9+ 可直接用 `dict`） |
| `Tuple` | `from typing import Tuple` | 3.5+ | 元组类型（3.9+ 可直接用 `tuple`） |
| `Set` | `from typing import Set` | 3.5+ | 集合类型（3.9+ 可直接用 `set`） |
| `Optional` | `from typing import Optional` | 3.5+ | 可选类型（等价于 `Union[X, None]`） |
| `Union` | `from typing import Union` | 3.5+ | 联合类型（3.10+ 可用 `X \| Y`） |
| `Any` | `from typing import Any` | 3.5+ | 任意类型 |
| `Callable` | `from typing import Callable` | 3.5+ | 可调用类型（3.9+ 可直接用 `collections.abc.Callable` 做类型注解） |
| `TypeVar` | `from typing import TypeVar` | 3.5+ | 类型变量（泛型用） |
| `Literal` | `from typing import Literal` | 3.8+ | 字面量类型 |
| `Final` | `from typing import Final` | 3.8+ | 常量标记 |
| `TypeGuard` | `from typing import TypeGuard` | 3.10+ | 类型守卫 |
| `overload` | `from typing import overload` | 3.5+ | 函数重载装饰器 |
| `TypeAlias` | `from typing import TypeAlias` | 3.10+ | 类型别名标记 |

#### 一次性导入多个常用类型

```python
# 推荐写法：按需导入，用到什么导入什么
from typing import Optional, Union, List, Dict

# 也可以一次导入多个
from typing import (
    Any,
    Callable,
    Dict,
    Final,
    List,
    Literal,
    Optional,
    Tuple,
    TypeVar,
    Union,
)
```

#### Python 3.9+ vs 3.8 及以下的写法对比

```python
# Python 3.8 及以下：必须从 typing 导入
from typing import List, Dict, Tuple, Set
names: List[str] = ["Alice"]
scores: Dict[str, int] = {"math": 90}
point: Tuple[int, int] = (1, 2)
ids: Set[int] = {1, 2, 3}

# Python 3.9+：可以直接用内置类型
names: list[str] = ["Alice"]
scores: dict[str, int] = {"math": 90}
point: tuple[int, int] = (1, 2)
ids: set[int] = {1, 2, 3}

# Python 3.10+：Union 和 Optional 也可以简化
from typing import Union, Optional
value: Union[int, str] = 42   # 旧写法
value: int | str = 42          # 新写法

name: Optional[str] = None    # 旧写法
name: str | None = None        # 新写法
```

---

### 2.8 字面量类型（Literal）

限制变量只能是特定的值：

```python
from typing import Literal

# direction 只能是这四个值之一
direction: Literal["north", "south", "east", "west"] = "north"

# level 只能是 1, 2, 3
level: Literal[1, 2, 3] = 2

# 用于函数参数
def set_mode(mode: Literal["read", "write", "append"]) -> None:
    print(f"Mode set to: {mode}")

set_mode("read")   # OK
set_mode("delete") # 类型检查报错！
```

### 2.9 Final 类型

表示变量不应该被重新赋值（常量）：

```python
from typing import Final

# MAX_SIZE 是常量，不应被修改
MAX_SIZE: Final = 100
MAX_SIZE = 200  # mypy 会报错

# 类中的常量
class Config:
    API_VERSION: Final[str] = "v1"
    TIMEOUT: Final[int] = 30
```

#### 通俗理解

`Final` 就是"只读标签"——告诉所有人（和工具）这个值不应该被改。

```python
MAX_RETRY: Final = 3
# 之后如果写了 MAX_RETRY = 5，mypy 会警告你
```

`Literal` 就是"只能选这几个值"——像单选题。

```python
direction: Literal["up", "down", "left", "right"] = "up"
# direction = "diagonal"  → mypy 报错，不在选项里
```

#### 新手常见错误

**错误 1：把 Final 当 const 用**

```python
from typing import Final

MAX_SIZE: Final = 100
MAX_SIZE = 200  # mypy 警告，但 Python 运行时不会报错！
# Final 只是告诉工具"别改"，Python 本身没有真正的常量
```

**错误 2：Literal 值拼写错误**

```python
from typing import Literal

def set_color(color: Literal["red", "green", "blue"]) -> None:
    print(color)

set_color("Red")   # mypy 报错！大小写必须完全匹配
set_color("red")   # OK
```

---

### 附录：Python 3.9 以下版本兼容写法总结

如果你的项目需要兼容 Python 3.8 或更早版本，注意以下区别：

| 功能 | Python 3.9+ | Python 3.8 及以下 |
|------|------------|------------------|
| 列表类型 | `list[int]` | `List[int]`（需 `from typing import List`） |
| 字典类型 | `dict[str, int]` | `Dict[str, int]`（需 `from typing import Dict`） |
| 元组类型 | `tuple[int, str]` | `Tuple[int, str]`（需 `from typing import Tuple`） |
| 集合类型 | `set[int]` | `Set[int]`（需 `from typing import Set`） |
| 联合类型 | `int \| str`（3.10+） | `Union[int, str]`（需 `from typing import Union`） |
| 可选类型 | `str \| None`（3.10+） | `Optional[str]`（需 `from typing import Optional`） |
| 字面量 | `Literal["a", "b"]` | `Literal["a", "b"]`（需 `from typing import Literal`，3.8+） |
| Final | `Final = 100` | `Final = 100`（需 `from typing import Final`，3.8+） |
| 类型别名语句 | `type X = int`（3.12+） | `X = int`（普通赋值，无 `type` 语句） |

#### 兼容多版本的写法技巧

```python
import sys
from typing import Union, Optional, List, Dict

# 方法 1：统一用 typing 模块（兼容 3.5+）
def greet(names: List[str]) -> Dict[str, str]:
    return {name: f"Hello, {name}" for name in names}

# 方法 2：用 TYPE_CHECKING 守卫（运行时不执行导入检查）
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from typing import List, Dict

# 方法 3：使用 __future__ 注解（Python 3.7+）
from __future__ import annotations  # 让所有注解变成字符串，延迟求值
# 加了这行后，3.7+ 也可以用 list[int] 语法
# 但注意：这只影响注解，不影响运行时的 isinstance 检查
```

#### `from __future__ import annotations` 详解

```python
# Python 3.7/3.8 想用 list[int] 语法？
# 在文件最开头加这行：
from __future__ import annotations

# 之后就可以用新语法了
def get_names() -> list[str]:       # 3.7+ 都可以
    return ["Alice", "Bob"]

def get_scores() -> dict[str, int]: # 3.7+ 都可以
    return {"math": 90}
```

**注意：** `from __future__ import annotations` 让所有注解变成惰性字符串，不影响运行。

---

### 变量类型注解 — 阶段小练习

**练习 A：** 为以下变量添加合适的类型注解：

```python
# 1. 学生姓名
student_name = "张三"

# 2. 所有课程成绩
all_scores = [85, 90, 78, 92, 88]

# 3. 学生信息字典
student_info = {"name": "张三", "age": 20, "grades": [85, 90]}

# 4. 可能为空的搜索结果
search_result = None

# 5. HTTP 状态码（只能是 200, 404, 500）
status_code = 200
```

**参考答案：**

```python
from typing import Optional, Literal, Any

# 1.
student_name: str = "张三"

# 2.
all_scores: list[int] = [85, 90, 78, 92, 88]

# 3.
student_info: dict[str, str | int | list[int]] = {
    "name": "张三", "age": 20, "grades": [85, 90]
}
# 或者用 Any（如果结构很复杂）
student_info: dict[str, Any] = {"name": "张三", "age": 20, "grades": [85, 90]}

# 4.
search_result: Optional[str] = None  # 搜索结果可能是字符串，也可能没有

# 5.
status_code: Literal[200, 404, 500] = 200
```


**练习 B：** 写出以下类型的声明（用 Python 3.9+ 语法）：

1. 一个字符串到浮点数字典
2. 一个包含 (x, y, z) 三维坐标的元组
3. 一个可能为空的整数列表

**参考答案：**

```python
from typing import Optional

# 1.
prices: dict[str, float] = {"apple": 3.5, "banana": 2.0}

# 2.
point_3d: tuple[float, float, float] = (1.0, 2.0, 3.0)

# 3.
numbers: Optional[list[int]] = None
# 等价写法
numbers: list[int] | None = None  # Python 3.10+
```


---

## 三、函数和方法类型注解

### 3.1 基本语法

```python
# 语法: def 函数名(参数: 类型) -> 返回类型:

def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"Hello, {name}!"

def is_even(n: int) -> bool:
    return n % 2 == 0
```

#### 通俗理解

函数类型注解就像餐厅菜单：

- 参数注解 = "这道菜需要什么食材"（`name: str` = 需要一个字符串）
- 返回注解 = "这道菜端出来是什么"（`-> str` = 返回一个字符串）

```python
# 菜单式阅读：
def greet(name: str) -> str:
#         ↑ 需要一个 str    ↑ 做出一个 str
    return f"Hello, {name}!"
```

#### 新手常见错误

**错误 1：忘记返回值注解**

```python
# 差：没有返回值注解，调用者不知道返回什么
def get_name():
    return "Alice"

# 好：明确标注返回类型
def get_name() -> str:
    return "Alice"

# 特殊情况：没有返回值用 None
def print_hello() -> None:
    print("hello")
```

**错误 2：参数默认值类型和注解不匹配**

```python
# 错误：默认值是 None，但注解是 str
def greet(name: str = None) -> str:   # mypy 报错！
    return f"Hello, {name}"

# 正确：用 Optional 或给合适的默认值
def greet(name: Optional[str] = None) -> str:
    if name is None:
        return "Hello, World"
    return f"Hello, {name}"

# 或者
def greet(name: str = "World") -> str:
    return f"Hello, {name}"
```

---

### 3.2 各种参数类型的注解

#### 位置参数

```python
def move(x: int, y: int) -> None:
    print(f"Moving to ({x}, {y})")
```

#### 默认值参数

```python
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}!"
```

#### 可变位置参数（*args）

```python
def sum_all(*args: int) -> int:
    return sum(args)

# 等价于 args 是一个 tuple[int, ...]
sum_all(1, 2, 3, 4, 5)  # 15
```

> **新手提示：** `*args: int` 的意思是"接收任意多个 int 参数"，不是"接收一个 int"。
> 在函数内部，`args` 的类型实际上是 `tuple[int, ...]`（一个整数元组）。

#### 可变关键字参数（**kwargs）

```python
def print_info(**kwargs: str) -> None:
    for key, value in kwargs.items():
        print(f"{key}: {value}")

# 等价于 kwargs 是一个 dict[str, str]
print_info(name="Alice", city="Beijing")
```

> **新手提示：** `**kwargs: str` 的意思是"接收任意多个关键字参数，值都是 str"。
> 注意这里注解的是**值**的类型，键永远是 str。在函数内部，`kwargs` 的类型是 `dict[str, str]`。

#### 混合使用

```python
def create_user(
    name: str,
    age: int,
    *hobbies: str,
    is_active: bool = True,
    **extra: str
) -> dict[str, str | int | bool | tuple[str, ...]]:
    return {
        "name": name,
        "age": age,
        "hobbies": hobbies,
        "is_active": is_active,
        "extra": extra
    }
```

### 3.3 返回类型注解

#### 无返回值

```python
def print_message(msg: str) -> None:
    print(msg)
```

#### 返回单一类型

```python
def get_length(s: str) -> int:
    return len(s)
```

#### 返回容器类型

```python
from typing import List, Dict, Tuple

def get_names() -> List[str]:
    return ["Alice", "Bob", "Charlie"]

def get_scores() -> Dict[str, int]:
    return {"math": 90, "english": 85}

def get_position() -> Tuple[int, int]:
    return (10, 20)
```

#### 返回可选类型

```python
from typing import Optional

def find_item(item_id: int) -> Optional[str]:
    items = {1: "Apple", 2: "Banana"}
    return items.get(item_id)  # 找到返回 str，找不到返回 None
```

### 3.4 回调函数类型注解

当函数参数是另一个函数时：

```python
from typing import Callable

# 接受一个函数作为参数
def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

def add(x: int, y: int) -> int:
    return x + y

def multiply(x: int, y: int) -> int:
    return x * y

result1 = apply(add, 3, 4)       # 7
result2 = apply(multiply, 3, 4)  # 12
```

**`Callable` 语法说明：**
- `Callable[[参数类型], 返回类型]`
- `Callable[[int, int], int]` 表示接受两个 int 参数，返回 int 的函数

**通俗理解：**
- `Callable` 就是"可以被调用的东西"——函数、方法、lambda 都算
- `Callable[[int, int], int]` 可以读作"接受两个 int，吐出一个 int 的机器"
- 方括号里第一部分 `[int, int]` 是参数类型列表，第二部分 `int` 是返回类型

#### 更复杂的回调类型

```python
from typing import Callable, List

# 排序的 key 函数
def sort_by_key(items: List[dict], key_func: Callable[[dict], int]) -> List[dict]:
    return sorted(items, key=key_func)

users = [
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": 25},
    {"name": "Charlie", "age": 35}
]

sorted_users = sort_by_key(users, lambda u: u["age"])
```

### 3.5 方法的类型注解

#### 实例方法

```python
class Calculator:
    def __init__(self, initial_value: float = 0):
        self.result: float = initial_value

    def add(self, value: float) -> "Calculator":
        self.result += value
        return self  # 返回自身，支持链式调用

    def subtract(self, value: float) -> "Calculator":
        self.result -= value
        return self

    def get_result(self) -> float:
        return self.result


# 链式调用
calc = Calculator(10)
result = calc.add(5).subtract(3).get_result()  # 12.0
```

**注意：** `"Calculator"` 用引号包裹是因为类还没有完全定义完，这叫**前向引用**。

#### 类方法

```python
class Date:
    def __init__(self, year: int, month: int, day: int):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, date_str: str) -> "Date":
        year, month, day = map(int, date_str.split("-"))
        return cls(year, month, day)

    @classmethod
    def today(cls) -> "Date":
        from datetime import datetime
        now = datetime.now()
        return cls(now.year, now.month, now.day)
```

#### 静态方法

```python
class MathUtils:
    @staticmethod
    def is_prime(n: int) -> bool:
        if n < 2:
            return False
        for i in range(2, int(n ** 0.5) + 1):
            if n % i == 0:
                return False
        return True

    @staticmethod
    def factorial(n: int) -> int:
        if n <= 1:
            return 1
        return n * MathUtils.factorial(n - 1)
```

### 3.6 泛型函数

当函数可以处理多种类型时：

```python
from typing import TypeVar, List

T = TypeVar("T")  # 声明一个类型变量

def first(items: List[T]) -> T:
    """返回列表的第一个元素"""
    return items[0]

# 类型会自动推断
result1 = first([1, 2, 3])       # result1 的类型是 int
result2 = first(["a", "b", "c"]) # result2 的类型是 str
```

#### 多个类型变量

```python
from typing import TypeVar, Tuple

T = TypeVar("T")
U = TypeVar("U")

def pair(first: T, second: U) -> Tuple[T, U]:
    return (first, second)

result = pair("hello", 42)  # 类型是 Tuple[str, int]
```

### 3.7 重载（Overload）

有时候一个函数根据参数类型不同，返回类型也不同：

```python
from typing import overload, Union

@overload
def parse(value: str) -> str: ...

@overload
def parse(value: int) -> int: ...

def parse(value: Union[str, int]) -> Union[str, int]:
    if isinstance(value, str):
        return value.strip()
    return value * 2
```

**注意：** `@overload` 装饰器只是给类型检查工具看的，实际执行的是没有装饰器的版本。

#### mypy 检查函数类型注解的示例

```python
# func_check.py
from typing import Optional

def divide(a: float, b: float) -> float:
    return a / b

def find(name: str) -> Optional[str]:
    items = {"alice": "Alice", "bob": "Bob"}
    return items.get(name)

# mypy 检查
result1 = divide(10, 3)         # OK：float / float -> float
result2 = divide("10", 3)       # mypy 报错：str 不能传给 float 参数

name = find("alice")
print(name.upper())             # mypy 报错：name 可能是 None
if name is not None:
    print(name.upper())         # OK：已排除 None
```

```bash
$ mypy func_check.py
func_check.py:10: error: Argument 1 to "divide" has incompatible type "str"; expected "float"
func_check.py:13: error: Item "None" of "Optional[str]" has no attribute "upper"
Found 2 errors in 1 file
```

#### 函数注解常见错误汇总

| 错误 | 说明 |
|------|------|
| 参数类型写错 | `def f(x: int)` 但传了 `str` |
| 返回值类型写错 | 标注 `-> int` 但实际返回了 `str` |
| 忘记处理 Optional 返回 | `find()` 返回 `Optional[str]` 但直接 `.upper()` |
| `*args` 类型写错 | `*args: int` 表示每个参数都是 int，不是"一个 int" |
| `**kwargs` 类型写错 | `**kwargs: str` 表示每个值都是 str |
| `-> None` 忘记写 | 没有返回值的函数应该标注 `-> None` |

---

### 函数类型注解 — 阶段小练习

**练习 A：** 为以下函数添加完整类型注解：

```python
def calculate_average(numbers):
    if not numbers:
        return 0
    return sum(numbers) / len(numbers)

def find_max(items, key=None):
    if key:
        return max(items, key=key)
    return max(items)

def process_string(text, uppercase=False, repeat=1):
    result = text
    if uppercase:
        result = result.upper()
    return result * repeat
```

**参考答案：**

```python
from typing import TypeVar, Callable, Optional, overload

T = TypeVar("T")

def calculate_average(numbers: list[int | float]) -> float:
    """计算平均值，空列表返回 0"""
    if not numbers:
        return 0.0
    return sum(numbers) / len(numbers)

def find_max(items: list[T], key: Optional[Callable[[T], int]] = None) -> T:
    """找到最大值，可选 key 函数"""
    if key:
        return max(items, key=key)
    return max(items)

def process_string(text: str, uppercase: bool = False, repeat: int = 1) -> str:
    """处理字符串：可选转大写、可选重复"""
    result = text
    if uppercase:
        result = result.upper()
    return result * repeat
```


**练习 B：** 写一个泛型函数 `last(items: list[T]) -> T`，返回列表最后一个元素，并用 mypy 验证类型推断。

**参考答案：**

```python
from typing import TypeVar

T = TypeVar("T")

def last(items: list[T]) -> T:
    """返回列表最后一个元素"""
    if not items:
        raise ValueError("列表不能为空")
    return items[-1]

# 类型推断
num = last([1, 2, 3])       # num 的类型是 int
name = last(["a", "b"])     # name 的类型是 str
```


---

## 四、Union 联合类型注解

### 4.1 什么是 Union？

`Union` 表示变量可以是多种类型中的一种。

```python
from typing import Union

# value 可以是 int 或 str
value: Union[int, str] = 42
value = "hello"  # 也可以
```

#### 通俗理解

`Union[int, str]` 就像一个"二选一"的标签：这个盒子里可以放整数 **或者** 字符串，但不能放列表。

- `int` → 只能放整数
- `str` → 只能放字符串
- `Union[int, str]` → 整数或字符串都行
- `int | str` → 同上，Python 3.10+ 更简洁的写法

### 4.2 基本用法

```python
from typing import Union

def double(x: Union[int, float]) -> Union[int, float]:
    return x * 2

# 可以传入 int 或 float
result1 = double(5)      # 10 (int)
result2 = double(3.14)   # 6.28 (float)
```

### 4.3 Python 3.10+ 新语法

从 Python 3.10 开始，可以使用 `|` 代替 `Union`：

```python
# 旧写法
from typing import Union
def process(value: Union[int, str]) -> Union[int, str]:
    pass

# 新写法（Python 3.10+）
def process(value: int | str) -> int | str:
    pass
```

**两种写法完全等价，推荐使用新语法。**

### 4.4 Union 与 Optional 的关系

`Optional[X]` 实际上就是 `Union[X, None]`：

```python
from typing import Optional, Union

# 这两种写法完全等价
def find(name: str) -> Optional[str]:
    pass

def find(name: str) -> Union[str, None]:
    pass

# Python 3.10+
def find(name: str) -> str | None:
    pass
```

### 4.5 Union 的常见使用场景

#### 场景一：函数接受多种类型

```python
def format_id(user_id: int | str) -> str:
    """格式化用户ID，支持数字和字符串"""
    if isinstance(user_id, int):
        return f"ID-{user_id:06d}"
    return f"ID-{user_id}"


print(format_id(42))        # ID-000042
print(format_id("abc-123")) # ID-abc-123
```

#### 场景二：函数返回不同类型

```python
def divide(a: float, b: float) -> float | str:
    """除法，除数为0时返回错误信息"""
    if b == 0:
        return "Error: division by zero"
    return a / b


result = divide(10, 3)   # float
result = divide(10, 0)   # str
```

#### 场景三：容器中的混合类型

```python
from typing import List

# 列表中可以包含 int 和 str
mixed: List[int | str] = [1, "two", 3, "four"]

# 字典的值可以是多种类型
data: dict[str, int | str | list[int]] = {
    "name": "Alice",
    "age": 25,
    "scores": [90, 85, 92]
}
```

### 4.6 Union 的类型窄化

当使用 Union 类型时，需要先判断具体类型：

```python
def process(value: int | str | list) -> str:
    # 使用 isinstance 窄化类型
    if isinstance(value, int):
        # 这里 value 被窄化为 int
        return f"整数: {value * 2}"
    elif isinstance(value, str):
        # 这里 value 被窄化为 str
        return f"字符串: {value.upper()}"
    else:
        # 这里 value 被窄化为 list
        return f"列表: 长度 {len(value)}"


print(process(42))           # 整数: 84
print(process("hello"))      # 字符串: HELLO
print(process([1, 2, 3]))    # 列表: 长度 3
```

### 4.7 复杂的 Union 类型

```python
from typing import Union, List, Dict, Tuple

# 复杂的联合类型
JsonValue = Union[
    str,
    int,
    float,
    bool,
    None,
    List["JsonValue"],
    Dict[str, "JsonValue"]
]

def parse_json(data: JsonValue) -> str:
    if isinstance(data, str):
        return f"string: {data}"
    elif isinstance(data, (int, float)):
        return f"number: {data}"
    elif isinstance(data, bool):
        return f"boolean: {data}"
    elif data is None:
        return "null"
    elif isinstance(data, list):
        return f"array: [{', '.join(parse_json(item) for item in data)}]"
    elif isinstance(data, dict):
        items = [f"{k}: {parse_json(v)}" for k, v in data.items()]
        return f"object: {{{', '.join(items)}}}"


# 测试
print(parse_json("hello"))
print(parse_json(42))
print(parse_json([1, "two", None]))
print(parse_json({"name": "Alice", "age": 25}))
```

### 4.8 Union 与类型守卫

```python
from typing import Union

def is_string(value: Union[str, int]) -> bool:
    return isinstance(value, str)


def process(value: str | int) -> str:
    # 使用类型守卫函数
    if is_string(value):
        # 但这里 value 不会被窄化，因为 is_string 不是类型守卫
        # 需要用 TypeGuard（Python 3.10+）
        pass

    if isinstance(value, str):
        # 这里 value 会被窄化为 str
        return value.upper()
    return str(value)
```

**注意：** 普通函数不会让 mypy 窄化类型。要让自定义函数触发窄化，需要用 `TypeGuard`，见下方章节。

#### 使用 TypeGuard（Python 3.10+）

```python
from typing import TypeGuard

def is_string_list(val: list[object]) -> TypeGuard[list[str]]:
    """类型守卫：判断是否是字符串列表"""
    return all(isinstance(x, str) for x in val)


def process(items: list[object]) -> None:
    if is_string_list(items):
        # 这里 items 被窄化为 list[str]
        print(", ".join(items))
    else:
        print("Not a string list")


process(["a", "b", "c"])  # a, b, c
process([1, 2, 3])         # Not a string list
```

### 4.9 Union 与 isinstance 配合的最佳实践

当函数接收 Union 类型时，几乎总是需要用 `isinstance` 来区分具体类型再做不同处理。这是使用 Union 的核心模式。

#### 模式一：简单的类型分支

```python
def format_value(value: int | float | str) -> str:
    """根据类型格式化值"""
    if isinstance(value, int):
        # 这里 mypy 知道 value 是 int
        return f"整数: {value:,}"           # 可以用 int 特有的格式化
    elif isinstance(value, float):
        # 这里 mypy 知道 value 是 float
        return f"浮点数: {value:.2f}"       # 可以用 float 特有的格式化
    else:
        # 这里 mypy 知道 value 是 str（排除了 int 和 float）
        return f"字符串: {value.upper()}"   # 可以用 str 特有的方法
```

#### 模式二：处理 Optional 返回值

```python
def get_user_name(user_id: int) -> str | None:
    """查找用户名，可能返回 None"""
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)

# 正确模式：先检查再使用
name = get_user_name(1)
if name is not None:
    # mypy 知道这里 name 是 str，可以安全调用 str 方法
    print(name.upper())
else:
    print("用户不存在")

# 错误模式：直接使用
name = get_user_name(999)
print(name.upper())  # mypy 报错：name 可能是 None
```

#### 模式三：处理复杂 Union 类型（用 match，Python 3.10+）

```python
def describe(value: int | str | list[int] | None) -> str:
    """用 match 语句处理多种类型"""
    match value:
        case int():
            return f"整数 {value}"
        case str():
            return f"字符串 '{value}'"
        case list():
            return f"列表，长度 {len(value)}"
        case None:
            return "空值"
```

#### 模式四：避免 Union 过多（考虑拆分）

```python
# 差：Union 太多，函数职责不清
def process(data: int | str | list | dict | None) -> str:
    # 五种类型都要处理，逻辑复杂
    ...

# 好：拆分成多个函数，每个只处理一种类型
def process_int(data: int) -> str:
    return f"int: {data}"

def process_str(data: str) -> str:
    return f"str: {data}"

def process_list(data: list) -> str:
    return f"list: {len(data)} items"

# 或者用一个分发函数
def process(data: int | str | list) -> str:
    if isinstance(data, int):
        return process_int(data)
    elif isinstance(data, str):
        return process_str(data)
    elif isinstance(data, list):
        return process_list(data)
    else:
        raise TypeError(f"不支持的类型: {type(data)}")
```

#### isinstance 检查顺序的注意事项

```python
# 注意：bool 是 int 的子类型！
def check(value: int | bool | str) -> str:
    if isinstance(value, bool):       # 必须先检查 bool
        return f"布尔: {value}"
    elif isinstance(value, int):      # 再检查 int
        return f"整数: {value}"
    else:
        return f"字符串: {value}"

# 如果先检查 int，True/False 会被当成 int 处理
def check_wrong(value: int | bool | str) -> str:
    if isinstance(value, int):        # True 也会进这里！
        return f"整数: {value}"       # check_wrong(True) → "整数: 1"
    elif isinstance(value, bool):     # 永远走不到
        return f"布尔: {value}"
    else:
        return f"字符串: {value}"
```

#### 新手常见错误

**错误 1：用 type() 而不是 isinstance()**

```python
def process(value: int | str) -> str:
    if type(value) == int:     # 不推荐：不支持子类型
        return str(value)
    return value.upper()

# 推荐用 isinstance，它支持继承关系
def process(value: int | str) -> str:
    if isinstance(value, int):
        return str(value)
    return value.upper()
```

**错误 2：窄化后又赋值**

```python
def process(value: int | str) -> str:
    if isinstance(value, str):
        result = value.upper()   # 这里 value 是 str
    # result 的类型已经是 str 了
    # 但 value 在这个作用域仍然是 int | str
    # 如果在 if 外面用 value，mypy 不会窄化
    return str(value)  # mypy 不确定 value 是 int 还是 str
```

---

### Union 类型 — 阶段小练习

**练习 A：** 写一个函数 `safe_get(data: dict, key: str, default: T) -> T | str`，从字典获取值，键不存在时返回默认值，值类型不对时返回错误信息。

**练习 B：** 写一个函数处理 API 响应，响应可能是 `dict`（成功）、`str`（错误消息）或 `None`（网络超时），对每种情况返回不同的处理结果。

**参考答案：**

```python
from typing import TypeVar

T = TypeVar("T")

def safe_get(data: dict, key: str, default: T) -> T | str:
    """安全获取字典值"""
    if key not in data:
        return default
    value = data[key]
    if not isinstance(value, type(default)):
        return f"类型错误: 期望 {type(default).__name__}, 得到 {type(value).__name__}"
    return value

# 测试
data = {"name": "Alice", "age": 25}
print(safe_get(data, "name", "default"))    # Alice
print(safe_get(data, "missing", "default")) # default
print(safe_get(data, "age", "default"))     # 类型错误: 期望 str, 得到 int


def handle_response(response: dict | str | None) -> str:
    """处理 API 响应"""
    if response is None:
        return "网络超时，请重试"
    elif isinstance(response, str):
        return f"服务器错误: {response}"
    else:
        # response 被窄化为 dict
        if "error" in response:
            return f"业务错误: {response['error']}"
        return f"成功: {response.get('data', '无数据')}"

# 测试
print(handle_response(None))                          # 网络超时，请重试
print(handle_response("500 Internal Server Error"))   # 服务器错误: 500 Internal Server Error
print(handle_response({"error": "用户不存在"}))         # 业务错误: 用户不存在
print(handle_response({"data": "hello"}))              # 成功: hello
```


---

## 五、综合示例

### 5.1 带完整类型注解的学生成绩系统

```python
from typing import Optional


class Student:
    """学生类"""

    def __init__(self, student_id: int, name: str) -> None:
        self.student_id: int = student_id
        self.name: str = name
        self.scores: dict[str, float] = {}

    def add_score(self, subject: str, score: float) -> None:
        """添加成绩"""
        if not 0 <= score <= 100:
            raise ValueError(f"成绩必须在 0-100 之间，当前值: {score}")
        self.scores[subject] = score

    def get_score(self, subject: str) -> Optional[float]:
        """获取某科成绩，可能不存在"""
        return self.scores.get(subject)

    def get_average(self) -> float:
        """计算平均分"""
        if not self.scores:
            return 0.0
        return sum(self.scores.values()) / len(self.scores)

    def get_highest(self) -> tuple[str, float] | None:
        """获取最高分科目"""
        if not self.scores:
            return None
        subject = max(self.scores, key=lambda s: self.scores[s])
        return (subject, self.scores[subject])

    def __str__(self) -> str:
        return f"Student(id={self.student_id}, name={self.name})"


class GradeBook:
    """成绩册"""

    def __init__(self) -> None:
        self.students: dict[int, Student] = {}

    def add_student(self, student: Student) -> None:
        """添加学生"""
        self.students[student.student_id] = student

    def get_student(self, student_id: int) -> Optional[Student]:
        """查找学生"""
        return self.students.get(student_id)

    def get_top_students(self, n: int = 3) -> list[Student]:
        """获取平均分最高的 n 名学生"""
        sorted_students = sorted(
            self.students.values(),
            key=lambda s: s.get_average(),
            reverse=True
        )
        return sorted_students[:n]

    def get_subject_average(self, subject: str) -> float | None:
        """计算某科目的全班平均分"""
        scores = [
            s.get_score(subject)
            for s in self.students.values()
            if s.get_score(subject) is not None
        ]
        if not scores:
            return None
        return sum(scores) / len(scores)

    def summary(self) -> dict[str, int | float]:
        """获取成绩册摘要"""
        all_scores = []
        for student in self.students.values():
            all_scores.extend(student.scores.values())

        return {
            "student_count": len(self.students),
            "total_scores": len(all_scores),
            "average": sum(all_scores) / len(all_scores) if all_scores else 0.0,
            "highest": max(all_scores) if all_scores else 0.0,
            "lowest": min(all_scores) if all_scores else 0.0
        }


# 使用示例
def main() -> None:
    # 创建成绩册
    book = GradeBook()

    # 添加学生
    students_data = [
        (1, "Alice", {"数学": 95, "英语": 88, "物理": 92}),
        (2, "Bob", {"数学": 78, "英语": 85, "物理": 90}),
        (3, "Charlie", {"数学": 92, "英语": 90, "物理": 88}),
        (4, "Diana", {"数学": 88, "英语": 92, "物理": 95}),
    ]

    for sid, name, scores in students_data:
        student = Student(sid, name)
        for subject, score in scores.items():
            student.add_score(subject, score)
        book.add_student(student)

    # 查找学生
    student = book.get_student(1)
    if student:
        print(f"学生: {student.name}")
        print(f"平均分: {student.get_average():.2f}")
        highest = student.get_highest()
        if highest:
            print(f"最高分: {highest[0]} - {highest[1]}")

    # 获取前三名
    print("\n前三名:")
    for student in book.get_top_students(3):
        print(f"  {student.name}: {student.get_average():.2f}")

    # 全班数学平均分
    math_avg = book.get_subject_average("数学")
    if math_avg:
        print(f"\n数学全班平均分: {math_avg:.2f}")

    # 成绩册摘要
    print("\n摘要:")
    summary = book.summary()
    for key, value in summary.items():
        print(f"  {key}: {value}")


if __name__ == "__main__":
    main()
```

### 5.2 带类型注解的数据处理管道

```python
from typing import TypeVar, Callable, List, Optional
from functools import reduce

T = TypeVar("T")
U = TypeVar("U")


def pipe(initial: T, *functions: Callable[[T], T]) -> T:
    """管道函数：将多个函数串联执行"""
    return reduce(lambda val, func: func(val), functions, initial)


def apply_to_list(items: List[T], func: Callable[[T], U]) -> List[U]:
    """对列表中的每个元素应用函数"""
    return [func(item) for item in items]


def filter_list(items: List[T], predicate: Callable[[T], bool]) -> List[T]:
    """过滤列表"""
    return [item for item in items if predicate(item)]


def find_first(items: List[T], predicate: Callable[[T], bool]) -> Optional[T]:
    """找到第一个满足条件的元素"""
    for item in items:
        if predicate(item):
            return item
    return None


# 使用示例
def main() -> None:
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    # 管道：过滤偶数 -> 平方 -> 求和
    result = pipe(
        numbers,
        lambda nums: filter_list(nums, lambda x: x % 2 == 0),  # [2, 4, 6, 8, 10]
        lambda nums: apply_to_list(nums, lambda x: x ** 2),    # [4, 16, 36, 64, 100]
        lambda nums: sum(nums)                                 # 220
    )
    print(f"结果: {result}")  # 220

    # 查找第一个大于 5 的平方数
    squares = apply_to_list(numbers, lambda x: x ** 2)
    first_large = find_first(squares, lambda x: x > 50)
    print(f"第一个大于 50 的平方数: {first_large}")  # 64


if __name__ == "__main__":
    main()
```

---

## 六、本章小结

### 变量类型注解
- 基本语法：`变量名: 类型 = 值`
- 容器类型：`list[int]`、`dict[str, int]`、`tuple[int, str]`
- 可选类型：`Optional[str]` 或 `str | None`
- 类型别名：简化复杂类型

### 函数类型注解
- 参数注解：`def func(x: int, y: str) -> None`
- 回调函数：`Callable[[int, str], bool]`
- 泛型函数：使用 `TypeVar`
- 重载：使用 `@overload`

### Union 联合类型
- `Union[int, str]` 或 `int | str`（Python 3.10+）
- `Optional[X]` 等价于 `Union[X, None]`
- 使用 `isinstance` 进行类型窄化
- TypeGuard 用于自定义类型守卫

---

## 七、练习题

### 练习 1：添加类型注解
为以下函数添加完整的类型注解：

```python
def process_data(data, threshold=0.5, verbose=False):
    results = []
    for item in data:
        if item["score"] > threshold:
            results.append(item["name"])
            if verbose:
                print(f"Selected: {item['name']}")
    return results
```

### 练习 2：实现泛型栈
使用泛型实现一个栈（Stack）类，支持 `push`、`pop`、`peek`、`is_empty` 方法。

### 练习 3：类型安全的配置解析器
实现一个配置解析器，支持从字典中获取不同类型的值，并使用 Union 类型处理可能的类型错误。

### 练习 4：Optional 处理链

实现以下函数链，要求每一步都安全处理 None：

```python
def parse_int(s: str) -> Optional[int]:
    """将字符串转为整数，失败返回 None"""
    ...

def double(n: int) -> int:
    """将数字翻倍"""
    ...

def format_result(n: int) -> str:
    """格式化结果"""
    ...
```

要求：写一个函数 `safe_pipeline(s: str) -> str`，将三个函数串联起来，任何一步失败都返回错误提示。

### 练习 5：泛型缓存类

实现一个泛型缓存类 `Cache[K, V]`，支持：
- `put(key: K, value: V) -> None`：存入缓存
- `get(key: K) -> Optional[V]`：获取缓存，不存在返回 None
- `has(key: K) -> bool`：判断键是否存在
- `remove(key: K) -> bool`：删除缓存，返回是否成功

### 练习 6：类型安全的事件分发器

实现一个事件分发器，支持注册不同类型事件的处理函数：

```python
from typing import Callable, Union

class EventEmitter:
    def on(self, event: str, handler: Callable[..., None]) -> None: ...
    def emit(self, event: str, *args, **kwargs) -> None: ...
```

要求：使用类型注解确保类型安全，并用 mypy 验证。

### 练习 7：mypy 实战

1. 安装 mypy：`pip install mypy`
2. 写一个包含 3 个类型错误的 Python 文件
3. 运行 `mypy your_file.py`，阅读错误信息
4. 修复所有错误，让 mypy 输出 `Success: no issues found`

---

## 八、进阶阅读

- [Python 官方 typing 文档](https://docs.python.org/3/library/typing.html)
- [mypy 官方文档](https://mypy.readthedocs.io/)
- [PEP 484 - Type Hints](https://peps.python.org/pep-0484/)
- [PEP 585 - Type Hinting Generics In Standard Collections](https://peps.python.org/pep-0585/)（3.9+ 内置类型泛型）
- [PEP 604 - Allow writing union types as X | Y](https://peps.python.org/pep-0604/)（3.10+ Union 语法）

---

## 九、速查卡片

### 什么时候用什么类型？

| 场景 | 类型写法 | 示例 |
|------|---------|------|
| 确定是某个类型 | 直接写类型 | `x: int = 42` |
| 可能是 None | `Optional[X]` 或 `X \| None` | `name: str \| None = None` |
| 多种类型之一 | `Union[X, Y]` 或 `X \| Y` | `value: int \| str` |
| 任意类型 | `Any`（尽量避免） | `data: Any` |
| 列表 | `list[X]` | `names: list[str]` |
| 字典 | `dict[K, V]` | `scores: dict[str, int]` |
| 元组（固定结构） | `tuple[X, Y]` | `point: tuple[int, int]` |
| 元组（变长同类型） | `tuple[X, ...]` | `numbers: tuple[int, ...]` |
| 函数 | `Callable[[参数], 返回]` | `func: Callable[[int], str]` |
| 只能是特定值 | `Literal[...]` | `mode: Literal["r", "w"]` |
| 常量 | `Final` | `MAX: Final = 100` |
| 泛型 | `TypeVar` | `T = TypeVar("T")` |

### 从 typing 导入还是直接用内置类型？

| Python 版本 | 推荐写法 |
|------------|---------|
| 3.5-3.8 | `from typing import List, Dict, Tuple, Set, Union, Optional` |
| 3.9 | 内置 `list, dict, tuple, set` + `from typing import Union, Optional` |
| 3.10+ | 内置 `list, dict, tuple, set` + `X \| Y` 语法（不再需要 Union/Optional） |
| 3.12+ | 以上 + `type X = Y` 类型别名语句 |
