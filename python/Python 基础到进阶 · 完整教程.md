# 🐍 Python 基础到进阶 · 完整教程

> 本教程面向有一定编程基础的 Python 学习者，适合快速复习与查漏补缺。内容由浅入深，力求细致全面。

---

# 第一章 · 输入输出与注释

---

## 1.1 `print()` 基本用法

`print()` 是 Python 中最常用的输出函数，功能远比"打印一句话"丰富。

### 输出字符串、数字、多个值

```python
# 输出字符串
print("Hello, Python!")       # Hello, Python!

# 输出数字
print(42)                     # 42
print(3.14)                   # 3.14

# 输出多个值（默认用空格分隔）
print("姓名:", "张三", "年龄:", 25)
# 姓名: 张三 年龄: 25
```

> **要点**：`print()` 可以接收任意数量的参数，内部会对非字符串类型自动调用 `str()` 转换后再输出。

### `sep` 和 `end` 参数

```python
# sep：控制多个值之间的分隔符（默认是空格 " "）
print("2026", "03", "18", sep="-")
# 2026-03-18

print("A", "B", "C", sep=" | ")
# A | B | C

# end：控制输出末尾的字符（默认是换行符 "\n"）
print("加载中", end="...")
print("完成！")
# 加载中...完成！

# 实际应用：在同一行打印进度
for i in range(5):
    print(i, end=" ")
# 0 1 2 3 4
```

### 三种字符串格式化方式

Python 提供了三种格式化字符串的方式，它们的演进关系是：`%` → `.format()` → `f-string`。

#### ① `%` 格式化（C 风格，最古老）

```python
name = "张三"
age = 25
score = 95.5

print("我叫%s，今年%d岁" % (name, age))
# 我叫张三，今年25岁

print("成绩：%.2f" % score)
# 成绩：95.50

# 常用占位符：
# %s → 字符串    %d → 整数    %f → 浮点数
# %x → 十六进制  %o → 八进制  %e → 科学计数法
# %.2f → 保留2位小数
# %10s → 右对齐，宽度10
# %-10s → 左对齐，宽度10
```

> **注意**：`%` 格式化是 Python 2 时代的遗留写法。新代码中**不推荐使用**，但你在阅读旧代码时一定会遇到，所以必须认识。

#### ② `.format()` 方法（Python 2.6+ 引入）

```python
name = "李四"
age = 30

# 按位置
print("我叫{}，今年{}岁".format(name, age))
# 我叫李四，今年30岁

# 按索引
print("{0}今年{1}岁，{0}是程序员".format(name, age))
# 李四今年30岁，李四是程序员

# 按名称
print("{name}的成绩是{score}分".format(name="王五", score=88))
# 王五的成绩是88分

# 格式控制
print("{:.2f}".format(3.14159))         # 3.14（保留2位小数）
print("{:>10}".format("hello"))          #      hello（右对齐，宽度10）
print("{:<10}|".format("hello"))         # hello     |（左对齐，宽度10）
print("{:^10}".format("hello"))          #   hello   （居中，宽度10）
print("{:0>8}".format(42))              # 00000042（用0填充）
print("{:,}".format(1000000))           # 1,000,000（千位分隔符）
```

#### ③ f-string（Python 3.6+，**强烈推荐**）

```python
name = "赵六"
age = 28
score = 92.567

# 基本用法：在字符串前加 f，大括号内放表达式
print(f"我叫{name}，今年{age}岁")
# 我叫赵六，今年28岁

# 大括号里可以放任意合法的 Python 表达式
print(f"明年{age + 1}岁")              # 明年29岁
print(f"名字大写：{name.upper()}")       # 名字大写：赵六（中文没变化，英文会变大写）

# 格式控制（语法：{表达式:格式说明符}）
print(f"成绩：{score:.2f}")              # 成绩：92.57
print(f"十六进制：{255:#x}")             # 十六进制：0xff
print(f"百分比：{0.856:.1%}")            # 百分比：85.6%
print(f"{'居中':^10}")                   # 居中的宽度10

# 多行 f-string
info = (
    f"姓名：{name}\n"
    f"年龄：{age}\n"
    f"成绩：{score:.1f}"
)
print(info)

# Python 3.12+ 新特性：f-string 内可嵌套引号
d = {"key": 100}
print(f"结果是 {d['key']}")      # 3.6+ 通用写法
# 说明：Python 3.12+ 对 f-string 表达式里的引号嵌套更宽松。
```

> **总结对比**：
> | 方式        | 版本要求 | 可读性 | 性能     | 推荐度       |
> | ----------- | -------- | ------ | -------- | ------------ |
> | `%`         | 所有版本 | ⭐⭐     | 中等     | ❌ 旧代码兼容 |
> | `.format()` | 2.6+     | ⭐⭐⭐    | 中等     | ✅ 备选       |
> | f-string    | 3.6+     | ⭐⭐⭐⭐⭐  | 通常更快 | ✅✅ **首选**  |

### ✅ 小练习（含答案）

**题目**：已知 `name="小明"`、`score=87.456`、`rate=0.923`，用 **f-string** 输出：
- `姓名：小明`
- `成绩：87.5`
- `完成率：92.3%`

**参考答案**：

```python
name = "小明"
score = 87.456
rate = 0.923

print(f"姓名：{name}")
print(f"成绩：{score:.1f}")
print(f"完成率：{rate:.1%}")
```

---

## 1.2 `input()` 获取用户输入

```python
# 基本用法：括号里的字符串是提示信息
name = input("请输入你的名字：")
print(f"你好，{name}！")
```

### 返回值始终是字符串

这是新手最容易踩的坑之一：

```python
age = input("请输入年龄：")  # 用户输入 25
print(type(age))              # <class 'str'> —— 是字符串 "25"，不是整数 25！

# 如果你直接做运算，会出问题：
# print(age + 1)  # ❌ TypeError: can only concatenate str to str
print(age + "1")   # "251" —— 字符串拼接！
```

### 类型转换

```python
# 转整数
age = int(input("请输入年龄："))
print(f"明年你{age + 1}岁")

# 转浮点数
height = float(input("请输入身高(米)："))
print(f"身高：{height}米")

# 安全的做法：先接收再转换，或配合异常处理
user_input = input("请输入一个数字：")
try:
    num = int(user_input)
    print(f"你输入的数字是 {num}")
except ValueError:
    print(f"'{user_input}' 不是有效的整数！")
```

### 一次读入多个值

```python
# 配合 split() 分割
a, b = input("输入两个数(空格分隔)：").split()
# 此时 a, b 仍是字符串

# 用 map() 批量转换类型
x, y = map(int, input("输入两个整数：").split())
print(f"两数之和：{x + y}")
```

> **补充说明**：
> 1. `input().split()` 默认按任意空白字符分割，连续多个空格也能正确处理。
> 2. 若输入的字段数量和左侧变量数量不一致（如只输入一个值，或输入了三个值），会抛出 `ValueError`。
> 3. 面向真实用户输入时，通常应配合 `try/except` 做合法性校验。

### ✅ 小练习（含答案）

**题目**：读入用户输入的一行 `a b`（两个整数），输出它们的乘积；如果输入不合法，输出“输入错误”。

**参考答案**：

```python
raw = input("请输入两个整数（空格分隔）：")
try:
    a, b = map(int, raw.split())
    print(a * b)
except ValueError:
    print("输入错误")
```

---

## 1.3 注释

### 单行注释 `#`

```python
# 这是一行注释
x = 42  # 行尾注释：给变量赋值

# 快捷键（多数IDE）：Ctrl + / 或 Cmd + /
```

### 多行注释 `""" """`

严格来说，Python **没有**专门的“多行注释语法”。三引号字符串（`"""` 或 `'''`）本质上仍然是**字符串字面量**；如果它既没有赋值给变量，也不位于函数/类/模块开头作为 docstring，执行后通常会被立刻丢弃，因此常被**约定俗成**地当作多行注释：

```python
"""
这是一段多行注释
可以写很多行
不会被执行
"""

'''
用单引号的三引号也可以
'''
```

> **进阶提示**：三引号放在函数/类/模块的第一行时，会成为 **docstring**（文档字符串），可以通过 `help()` 或 `.__doc__` 访问：
>
> ```python
> def add(a, b):
>     """返回两个数的和。
> 
>     参数:
>         a: 第一个数
>         b: 第二个数
>     """
>     return a + b
> 
> print(add.__doc__)
> help(add)
> ```

---

# 第二章 · 变量与数据类型

---

## 2.1 变量赋值

### 动态类型，无需声明

Python 是**动态类型语言**——变量不需要声明类型，赋什么值就是什么类型：

```python
x = 10          # x 是 int
x = "hello"     # 现在 x 变成了 str
x = [1, 2, 3]   # 现在 x 又变成了 list
```

> 变量名本质上只是一个"标签"（引用），它贴在对象上。重新赋值就是把标签撕下来贴到新对象上。

### 命名规则

```python
# ✅ 合法的变量名
my_var = 1
_private = 2
__dunder__ = 3
camelCase = 4
MAX_SIZE = 100     # 全大写：约定表示常量

# ❌ 非法的变量名
# 2name = 1        # 不能以数字开头
# my-var = 1       # 不能用连字符
# class = 1        # 不能用关键字（class, if, for 等）
```

### 多变量赋值

```python
# 同时给多个变量赋不同值（本质是元组解包）
a, b, c = 1, 2, 3
print(a, b, c)    # 1 2 3

# 同时给多个变量赋相同值
x = y = z = 0
print(x, y, z)    # 0 0 0
```

### 交换变量（Python 特色技巧）

```python
a, b = 10, 20
a, b = b, a        # 一行搞定交换！
print(a, b)         # 20 10

# 在其他语言中通常需要临时变量：
# temp = a; a = b; b = temp
```

> **原理**：右边的 `b, a` 先组成一个元组 `(20, 10)`，然后再解包赋给左边的 `a, b`。

---

## 2.2 数字类型

### `int`（整数）

```python
a = 42
b = -100
c = 0

# Python 的整数没有大小限制（不像 C/Java 有 int32 的限制）
big = 99999999999999999999999999999
print(big + 1)     # 100000000000000000000000000000

# 不同进制的表示
binary = 0b1010     # 二进制 → 10
octal = 0o17        # 八进制 → 15
hexadecimal = 0xFF  # 十六进制 → 255

# 大数字可以用下划线分隔，提高可读性
population = 1_400_000_000
print(population)   # 1400000000
```

### `float`（浮点数）

```python
pi = 3.14159
e = 2.718
small = 1.5e-10     # 科学计数法：1.5 × 10⁻¹⁰

# ⚠️ 浮点数精度问题（所有语言都有）
print(0.1 + 0.2)           # 0.30000000000000004
print(0.1 + 0.2 == 0.3)    # False！

# 正确的浮点数比较方式
import math
print(math.isclose(0.1 + 0.2, 0.3))  # True

# 需要精确计算（如货币），使用 decimal 模块
from decimal import Decimal
print(Decimal("0.1") + Decimal("0.2"))  # 0.3
```

### `complex`（复数）

```python
z = 3 + 4j          # j 是虚数单位
print(z.real)        # 3.0（实部）
print(z.imag)        # 4.0（虚部）
print(abs(z))        # 5.0（模）
```

### 算术运算符

```python
a, b = 17, 5

print(a + b)    # 22    加法
print(a - b)    # 12    减法
print(a * b)    # 85    乘法
print(a / b)    # 3.4   除法（结果总是 float）
print(a // b)   # 3     整除（向下取整）
print(a % b)    # 2     取余
print(a ** b)   # 1419857  幂运算：17⁵

# ⚠️ 整除的"向下取整"规则
print(-17 // 5)   # -4（不是 -3！向负无穷取整）
print(-17 % 5)    # 3 （满足 a == (a // b) * b + (a % b)）

# 增强赋值运算符
x = 10
x += 3    # x = x + 3 → 13
x -= 2    # x = x - 2 → 11
x *= 4    # x = x * 4 → 44
x //= 5   # x = x // 5 → 8
x **= 2   # x = x ** 2 → 64
```

### 类型转换

```python
# int → float
print(float(10))       # 10.0

# float → int（截断小数部分，不是四舍五入）
print(int(3.7))        # 3
print(int(-3.7))       # -3

# 字符串 → 数字
print(int("42"))       # 42
print(float("3.14"))   # 3.14

# 数字 → 字符串
print(str(100))        # "100"

# 四舍五入
print(round(3.14159, 2))   # 3.14
print(round(2.5))          # 2（银行家舍入法：.5 时取偶数）
print(round(3.5))          # 4
```

---

## 2.3 字符串 `str`

字符串是 Python 中最常用的数据类型之一。

### 创建字符串

```python
s1 = 'hello'           # 单引号
s2 = "hello"            # 双引号（功能完全相同）
s3 = """多行
字符串"""                # 三引号支持换行
s4 = r"C:\new\text"    # 原始字符串（r 前缀），反斜杠不转义
```

### 索引与切片

```python
s = "Hello, Python!"
#    0123456789...

# 索引（0 开始，支持负索引）
print(s[0])      # H
print(s[-1])     # !
print(s[7])      # P

# 切片 s[start:stop:step]
# start 包含，stop 不包含
print(s[0:5])     # Hello
print(s[7:13])    # Python
print(s[:5])      # Hello（省略 start 表示从头开始）
print(s[7:])      # Python!（省略 stop 表示到末尾）
print(s[::2])     # Hlo yhn（每隔一个取一个）
print(s[::-1])    # !nohtyP ,olleH（反转字符串！）

# 切片不会越界，多了自动截断
print(s[0:100])   # Hello, Python!（不报错）
```

### 常用方法

```python
s = "  Hello, World!  "

# 去除首尾空白字符
print(s.strip())        # "Hello, World!"
print(s.lstrip())       # "Hello, World!  "（只去左边）
print(s.rstrip())       # "  Hello, World!"（只去右边）

# 大小写转换
print("hello".upper())      # "HELLO"
print("HELLO".lower())      # "hello"
print("hello world".title()) # "Hello World"
print("hello world".capitalize())  # "Hello world"

# 查找与替换
s = "Hello, World!"
print(s.find("World"))      # 7（返回起始索引，找不到返回 -1）
print(s.index("World"))     # 7（找不到会抛 ValueError）
print(s.count("l"))         # 3（出现次数）
print(s.replace("World", "Python"))  # "Hello, Python!"

# 判断方法
print("123".isdigit())      # True（全是数字）
print("abc".isalpha())      # True（全是字母）
print("abc123".isalnum())   # True（字母或数字）
print("  ".isspace())       # True（全是空白）
print("Hello".startswith("He"))  # True
print("Hello".endswith("lo"))    # True

# 分割与拼接
csv = "苹果,香蕉,橘子"
fruits = csv.split(",")         # ["苹果", "香蕉", "橘子"]
print(" / ".join(fruits))       # "苹果 / 香蕉 / 橘子"

# split 的 maxsplit 参数
log = "2026-03-18 22:00:00 ERROR 程序崩溃了"
parts = log.split(" ", maxsplit=2)
print(parts)  # ["2026-03-18", "22:00:00", "ERROR 程序崩溃了"]

# 对齐
print("hello".center(20, "-"))   # -------hello--------
print("hello".ljust(20, "."))    # hello...............
print("hello".rjust(20, "."))    # ...............hello
print("42".zfill(8))             # 00000042
```

### 字符串是不可变类型

```python
s = "Hello"
# s[0] = "h"  # ❌ TypeError: 'str' object does not support item assignment

# 要"修改"字符串，只能创建一个新字符串
s = "h" + s[1:]   # "hello"
```

---

## 2.4 布尔类型 `bool`

```python
a = True
b = False
print(type(a))   # <class 'bool'>
```

### 逻辑运算

```python
print(True and False)    # False
print(True or False)     # True
print(not True)          # False

# 短路求值
# and：如果第一个为 False，直接返回第一个值（不再计算第二个）
# or：如果第一个为 True，直接返回第一个值
print(0 and "hello")     # 0（0 是 Falsy，短路返回 0）
print(1 and "hello")     # "hello"（1 是 Truthy，返回第二个）
print("" or "default")   # "default"（常用于设置默认值）
print("hi" or "default") # "hi"
```

### Falsy 值（被视为 `False` 的值）

```python
# 以下值在布尔上下文中等价于 False：
falsy_values = [
    False,      # 布尔值 False
    0,          # 整数零
    0.0,        # 浮点零
    0j,         # 复数零
    "",          # 空字符串
    [],         # 空列表
    (),         # 空元组
    {},         # 空字典
    set(),      # 空集合
    None,       # None
]

for val in falsy_values:
    print(f"{str(val):>10} → bool: {bool(val)}")

# 除了上面这些，其他所有值都是 Truthy
print(bool(1))          # True
print(bool("hello"))    # True
print(bool([0]))        # True（列表不为空，即使里面是 0）
```

### 实际用法

```python
# 用在条件判断中
lst = [1, 2, 3]
if lst:           # 等价于 if len(lst) > 0:
    print("列表不为空")

name = ""
if not name:      # 此处 name 是字符串时，表示“空字符串”
    print("名字为空")
```

### `None` 与真值判断

`None` 表示“没有值”或“尚未设置”，它的类型是 `NoneType`。在 Python 中，`None` 既非常常见，也很容易和 `0`、`""`、`False` 混淆。

```python
value = None

print(value is None)   # True
print(bool(value))     # False

result = 0
if not result:
    print("result 在布尔上下文中是假值")

# 但“假值”不等于“None”
print(result is None)  # False
```

> **教学建议**：
> - 只要是判断“是否没有值”，优先写 `is None` / `is not None`。
> - 只要是判断“是否为空、是否为 0、是否为空容器”，才使用 `if not x` 这类真值判断。

---

## 2.5 类型检查

```python
x = 42
print(type(x))             # <class 'int'>
print(type(x).__name__)    # "int"

# isinstance()：推荐的类型判断方式（支持继承关系）
print(isinstance(42, int))         # True
print(isinstance(42, (int, float)))  # True（可以传元组检查多种类型）
print(isinstance(True, int))       # True！bool 是 int 的子类

# type() vs isinstance() 的区别
# type() 不考虑继承关系，isinstance() 考虑
class Animal: pass
class Dog(Animal): pass

d = Dog()
print(type(d) == Animal)        # False（严格匹配类型）
print(isinstance(d, Animal))    # True（Dog 是 Animal 的子类）
```

---

## 2.6 可变对象与不可变对象（重要补充）

这是 Python 初学者最容易“代码能写、概念没吃透”的主题之一。很多看似奇怪的问题，比如“函数默认参数为什么会串数据”“为什么改列表会影响别处”，本质上都和**对象是否可变**有关。

### 常见可变类型与不可变类型

```python
# 常见不可变类型
immutable_examples = (
    42,          # int
    3.14,        # float
    "hello",     # str
    (1, 2, 3),   # tuple
    True,        # bool
    None,        # NoneType
)

# 常见可变类型
mutable_examples = [
    [1, 2, 3],           # list
    {"name": "张三"},    # dict
    {1, 2, 3},           # set
]
```

### 修改对象 vs 重新绑定变量

```python
# 重新绑定：变量指向了新对象
x = 10
print(id(x))
x = x + 1
print(id(x))     # 变了，因为 int 不可变

# 原地修改：对象本身被改了
lst = [1, 2]
alias = lst
lst.append(3)

print(lst)       # [1, 2, 3]
print(alias)     # [1, 2, 3] —— alias 也“看到”了变化
print(lst is alias)   # True
```

> **记忆方法**：
> - 不可变对象：改不了，只能“创建新对象再让变量指向它”。
> - 可变对象：对象本身能被原地修改，因此多个变量若引用同一对象，可能互相影响。
>
> 这也是为什么教程前面一再强调“不要把可变对象写成默认参数”。

---

# 第三章 · 条件控制 · if / elif / else

---

## 3.1 基本语法

```python
score = 85

if score >= 90:
    print("优秀")
elif score >= 80:
    print("良好")      # ← 会执行这个
elif score >= 60:
    print("及格")
else:
    print("不及格")
```

> **核心规则**：Python 用**缩进**（通常 4 个空格）来定义代码块，而不是花括号 `{}`。这不是风格建议，而是**语法要求**。

```python
# ⚠️ 缩进错误会直接报 IndentationError
if True:
print("hello")    # ❌ IndentationError

# 嵌套 if
age = 20
has_id = True

if age >= 18:
    if has_id:
        print("可以进入")
    else:
        print("请出示身份证")
else:
    print("未成年不可进入")
```

### ✅ 小练习（含答案）

**题目**：输入一个分数 `score`，输出评级：`A(>=90)`、`B(>=80)`、`C(>=60)`、`D(<60)`。

**参考答案**：

```python
score = 83

if score >= 90:
    print("A")
elif score >= 80:
    print("B")
elif score >= 60:
    print("C")
else:
    print("D")
```

---

## 3.2 比较与成员运算符

### 比较运算符

```python
print(5 == 5)     # True
print(5 != 3)     # True
print(5 > 3)      # True
print(5 < 3)      # False
print(5 >= 5)     # True
print(5 <= 4)     # False

# Python 支持链式比较（其他语言通常不支持）
x = 5
print(1 < x < 10)         # True（等价于 1 < x and x < 10）
print(1 < x < 3)          # False
print(0 <= x <= 100)      # True
```

### 成员运算符 `in` / `not in`

```python
# 检查元素是否在容器中
print("a" in "apple")           # True
print(3 in [1, 2, 3])           # True
print("name" in {"name": "张三"})  # True（检查的是 key）
print(4 not in [1, 2, 3])       # True
```

### 身份运算符 `is` / `is not`

```python
# is 检查两个变量是否指向同一个对象（内存地址相同）
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)     # True（值相等）
print(a is b)     # False（不是同一个对象）
print(a is c)     # True（c 和 a 指向同一个列表）

# 最常见的用法：判断 None
x = None
if x is None:       # ✅ 推荐用 is 判断 None
    print("x 是 None")
# if x == None:      # ❌ 不推荐，虽然通常也能用
```

> **记忆口诀**：`==` 比较**值**，`is` 比较**身份（地址）**。判断 `None` 时**永远用 `is`**。

---

## 3.3 三元表达式

```python
age = 20

# 传统写法
if age >= 18:
    status = "成年"
else:
    status = "未成年"

# 三元表达式（一行搞定）
status = "成年" if age >= 18 else "未成年"
print(status)  # 成年

# 可以嵌套（但不推荐过度嵌套，影响可读性）
score = 85
grade = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 60 else "D"
print(grade)  # B
```

---

## 3.4 match / case（Python 3.10+，结构模式匹配）

这是 Python 3.10 引入的新特性，类似其他语言的 `switch/case`，但功能**强大得多**。

> **版本提醒**：如果你所在的教学机房、服务器或科研环境仍在使用 Python 3.9 及以下版本，这一节的代码将无法运行。

```python
# 基本用法
command = "start"

match command:
    case "start":
        print("启动程序")
    case "stop":
        print("停止程序")
    case "restart":
        print("重启程序")
    case _:                # _ 是通配符，等价于 default
        print(f"未知命令: {command}")
```

### 模式匹配的威力

```python
# 匹配结构（这是 match/case 最强大的地方）
point = (3, 0)

match point:
    case (0, 0):
        print("原点")
    case (x, 0):
        print(f"在 X 轴上，x = {x}")
    case (0, y):
        print(f"在 Y 轴上，y = {y}")
    case (x, y):
        print(f"点 ({x}, {y})")

# 带条件守卫（guard）
match point:
    case (x, y) if x == y:
        print(f"在对角线上：({x}, {y})")
    case (x, y):
        print(f"普通点：({x}, {y})")

# 匹配字典
user = {"name": "张三", "role": "admin"}

match user:
    case {"role": "admin", "name": name}:
        print(f"管理员：{name}")
    case {"role": "user", "name": name}:
        print(f"普通用户：{name}")

# 匹配类型
def process(value):
    match value:
        case int(n):
            print(f"整数：{n}")
        case str(s):
            print(f"字符串：{s}")
        case list(items):
            print(f"列表，长度 {len(items)}")
        case _:
            print("暂不支持的类型")
```

> **使用建议**：
> - 当分支是“按值匹配”或“按结构拆解”时，`match/case` 往往比一串 `if/elif` 更清晰。
> - 如果只是做简单区间判断（如成绩分档），`if/elif` 通常仍然更直接。

### ✅ 小练习（含答案）

**题目**：变量 `status` 可能是 `"ok"`、`"warn"`、`"error"`。用 `match/case` 输出对应中文状态。

**参考答案**：

```python
status = "warn"

match status:
    case "ok":
        print("正常")
    case "warn":
        print("警告")
    case "error":
        print("错误")
    case _:
        print("未知状态")
```

---

# 第四章 · 循环控制 · for / while

---

## 4.1 `for` 循环

Python 的 `for` 循环专门用于**遍历可迭代对象**（列表、字符串、字典、range 等）。

```python
# 遍历列表
fruits = ["苹果", "香蕉", "橘子"]
for fruit in fruits:
    print(fruit)

# 遍历字符串（逐个字符）
for ch in "Python":
    print(ch, end=" ")
# P y t h o n

# 遍历字典
person = {"name": "张三", "age": 25, "city": "深圳"}
for key in person:              # 默认遍历 key
    print(f"{key}: {person[key]}")

for key, value in person.items():   # 同时获取 key 和 value
    print(f"{key}: {value}")
```

### `range()` 函数

```python
# range(stop)：从 0 到 stop-1
for i in range(5):
    print(i, end=" ")    # 0 1 2 3 4

# range(start, stop)
for i in range(2, 6):
    print(i, end=" ")    # 2 3 4 5

# range(start, stop, step)
for i in range(0, 10, 2):
    print(i, end=" ")    # 0 2 4 6 8

# step 可以是负数（倒序）
for i in range(5, 0, -1):
    print(i, end=" ")    # 5 4 3 2 1

# range 是懒加载的，不会真正创建列表
# 即使 range(1_000_000_000) 也几乎不占内存
```

### ✅ 小练习（含答案）

**题目**：用 `for + range` 计算 `1~100` 的偶数和。

**参考答案**：

```python
total = 0
for i in range(2, 101, 2):
    total += i

print(total)  # 2550
```

---

## 4.2 `while` 循环

```python
# 基本用法
count = 0
while count < 5:
    print(count, end=" ")
    count += 1
# 0 1 2 3 4

# 经典场景：不知道循环次数
password = ""
while password != "123456":
    password = input("请输入密码：")
print("登录成功！")

# ⚠️ 避免死循环
# while True:
#     pass        # 这会永远执行下去，CPU 100%

# 正确的无限循环写法（配合 break）
while True:
    cmd = input("输入命令（quit 退出）：")
    if cmd == "quit":
        break
    print(f"执行命令：{cmd}")
```

---

## 4.3 循环控制关键字

### `break`：跳出整个循环

```python
for i in range(10):
    if i == 5:
        break           # 遇到 5 就结束整个循环
    print(i, end=" ")
# 0 1 2 3 4

# break 只跳出最内层的循环
for i in range(3):
    for j in range(3):
        if j == 2:
            break        # 只跳出内层 for j
        print(f"({i},{j})", end=" ")
    print()
# (0,0) (0,1)
# (1,0) (1,1)
# (2,0) (2,1)
```

### `continue`：跳过本次，进入下一次

```python
for i in range(10):
    if i % 2 == 0:
        continue         # 跳过偶数
    print(i, end=" ")
# 1 3 5 7 9
```

### `pass`：占位符

```python
# pass 什么都不做，用于语法上需要语句但逻辑上暂不实现的地方
for i in range(10):
    pass                  # TODO: 后续实现

def future_function():
    pass                  # 占位，防止语法错误

class EmptyClass:
    pass
```

### ✅ 小练习（含答案）

**题目**：给定 `nums=[3, 4, 7, 9, 12, 15]`，跳过偶数，遇到第一个大于 `10` 的奇数就停止，并打印它。

**参考答案**：

```python
nums = [3, 4, 7, 9, 12, 15]

for n in nums:
    if n % 2 == 0:
        continue
    if n > 10:
        print(n)  # 15
        break
```

---

## 4.4 常用搭配

### `enumerate()`：同时获取索引和值

```python
fruits = ["苹果", "香蕉", "橘子"]

# 不用 enumerate 的写法（笨拙）
for i in range(len(fruits)):
    print(f"{i}: {fruits[i]}")

# 用 enumerate 的写法（优雅）
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# 指定起始索引
for i, fruit in enumerate(fruits, start=1):
    print(f"第{i}种: {fruit}")
# 第1种: 苹果
# 第2种: 香蕉
# 第3种: 橘子
```

### `zip()`：并行遍历多个序列

```python
names = ["张三", "李四", "王五"]
scores = [90, 85, 78]
grades = ["A", "B", "C"]

# 打包遍历
for name, score in zip(names, scores):
    print(f"{name}: {score}分")

# 三个序列一起遍历
for name, score, grade in zip(names, scores, grades):
    print(f"{name}: {score}分 ({grade})")

# ⚠️ zip 以最短的序列为准
a = [1, 2, 3, 4, 5]
b = ["a", "b", "c"]
print(list(zip(a, b)))  # [(1, 'a'), (2, 'b'), (3, 'c')]

# 想以最长的为准，用 itertools.zip_longest
from itertools import zip_longest
print(list(zip_longest(a, b, fillvalue="?")))
# [(1, 'a'), (2, 'b'), (3, 'c'), (4, '?'), (5, '?')]
```

### `for...else` / `while...else`

这是 Python 独有的语法，很多人觉得困惑。**`else` 块在循环正常结束（没有被 `break` 中断）时执行**。

```python
# 场景：查找一个元素
numbers = [1, 3, 5, 7, 9]
target = 4

for num in numbers:
    if num == target:
        print(f"找到了 {target}")
        break
else:
    # 循环正常结束（没有 break），说明没找到
    print(f"没找到 {target}")
# 没找到 4

# 场景：判断质数
n = 17
for i in range(2, int(n**0.5) + 1):
    if n % i == 0:
        print(f"{n} 不是质数，可以被 {i} 整除")
        break
else:
    print(f"{n} 是质数")
# 17 是质数
```

> **理解技巧**：把 `else` 理解为 "no break" —— 如果循环中**没有执行 break**，就执行 else 块。

---

# 第五章 · 数据容器

---

## 5.1 列表 `list`（有序、可变）

列表是 Python 中最常用、最灵活的容器。

### 创建列表

```python
# 直接创建
a = [1, 2, 3, 4, 5]
b = ["hello", 3.14, True, None, [1, 2]]   # 可以存不同类型

# 空列表
c = []
d = list()

# 从其他类型转换
e = list("hello")       # ['h', 'e', 'l', 'l', 'o']
f = list(range(5))      # [0, 1, 2, 3, 4]

# 列表复制的坑
g = [1, 2, 3]
h = g           # ❌ 这只是引用复制！修改 h 也会改 g
h[0] = 99
print(g)        # [99, 2, 3]  ← g 也变了！

# 正确的浅拷贝
h = g.copy()    # 或 h = g[:] 或 h = list(g)
```

### 增加元素

```python
lst = [1, 2, 3]

# append：在末尾添加一个元素
lst.append(4)           # [1, 2, 3, 4]

# insert：在指定位置插入
lst.insert(0, 0)        # [0, 1, 2, 3, 4]
lst.insert(2, 99)       # [0, 1, 99, 2, 3, 4]

# extend：把另一个可迭代对象的元素逐个添加到末尾
lst.extend([5, 6, 7])   # [0, 1, 99, 2, 3, 4, 5, 6, 7]

# ⚠️ append vs extend 的区别
a = [1, 2]
a.append([3, 4])        # [1, 2, [3, 4]]  ← 把整个列表作为一个元素
b = [1, 2]
b.extend([3, 4])        # [1, 2, 3, 4]    ← 把元素逐个追加

# + 运算符：拼接（返回新列表，不修改原列表）
c = [1, 2] + [3, 4]     # [1, 2, 3, 4]

# * 运算符：重复
d = [0] * 5              # [0, 0, 0, 0, 0]

# ⚠️ * 运算符的坑（嵌套可变对象）
matrix = [[0] * 3] * 3   # ❌ 3 个子列表是同一个对象！
matrix[0][0] = 1
print(matrix)             # [[1, 0, 0], [1, 0, 0], [1, 0, 0]]

# 正确的写法
matrix = [[0] * 3 for _ in range(3)]  # ✅ 每个子列表独立
```

### 删除元素

```python
lst = [10, 20, 30, 20, 40]

# remove(value)：删除第一个匹配的值
lst.remove(20)          # [10, 30, 20, 40]

# pop(index)：删除指定索引并返回该元素
val = lst.pop(1)        # val = 30, lst = [10, 20, 40]
val = lst.pop()         # 不传参数：删除最后一个

# del：删除指定索引或切片
del lst[0]              # 删除第一个
del lst[1:3]            # 删除索引 1 到 2

# clear()：清空列表
lst.clear()             # []
```

### 查找与排序

```python
lst = [3, 1, 4, 1, 5, 9, 2, 6]

# 查找
print(lst.index(4))       # 2（返回第一次出现的索引）
print(lst.count(1))       # 2（出现次数）
print(4 in lst)           # True

# 排序（原地排序，修改原列表）
lst.sort()                # [1, 1, 2, 3, 4, 5, 6, 9]
lst.sort(reverse=True)    # [9, 6, 5, 4, 3, 2, 1, 1]

# sorted()：返回新列表，不修改原列表
original = [3, 1, 4, 1, 5]
new = sorted(original)            # new = [1, 1, 3, 4, 5]
print(original)                    # [3, 1, 4, 1, 5] 原列表不变

# 按自定义规则排序
words = ["banana", "apple", "cherry", "date"]
words.sort(key=len)              # 按长度排序：['date', 'apple', 'banana', 'cherry']
words.sort(key=str.lower)          # 按小写字母排序

# 反转
lst = [1, 2, 3]
lst.reverse()                      # [3, 2, 1]
print(list(reversed(lst)))         # [1, 2, 3]（reversed 返回迭代器）
```

### 列表推导式（List Comprehension）

这是 Python 最优雅的特性之一。

```python
# 基本语法：[表达式 for 变量 in 可迭代对象]
squares = [x**2 for x in range(6)]
# [0, 1, 4, 9, 16, 25]

# 带条件过滤：[表达式 for 变量 in 可迭代对象 if 条件]
evens = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]

# 带转换
names = ["alice", "bob", "charlie"]
upper_names = [name.upper() for name in names]
# ['ALICE', 'BOB', 'CHARLIE']

# 嵌套循环
pairs = [(x, y) for x in range(3) for y in range(3)]
# [(0,0), (0,1), (0,2), (1,0), (1,1), (1,2), (2,0), (2,1), (2,2)]

# 带 if...else（注意位置不同！）
labels = ["偶数" if x % 2 == 0 else "奇数" for x in range(5)]
# ['偶数', '奇数', '偶数', '奇数', '偶数']

# ⚠️ 位置规则：
# if 放后面 → 过滤  ：[x for x in range(10) if x > 5]
# if 放前面 → 必须有 else ：[x if x > 5 else 0 for x in range(10)]

# 矩阵转置
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
transposed = [[row[i] for row in matrix] for i in range(3)]
# [[1, 4, 7], [2, 5, 8], [3, 6, 9]]
```

#### Python 列表之列表推导式

当列表里的元素本身还是列表时，我们通常把它看成"二维列表"或"嵌套列表"。这时列表推导式依然非常好用，只是要特别注意 `for` 的顺序。

```python
# 1）生成二维列表：3 行 4 列
grid = [[0 for _ in range(4)] for _ in range(3)]
print(grid)
# [[0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]

# 2）对二维列表中的每个元素做变换
matrix = [[1, 2, 3], [4, 5, 6]]
doubled = [[x * 2 for x in row] for row in matrix]
print(doubled)
# [[2, 4, 6], [8, 10, 12]]

# 3）展平二维列表：把多行合并成一行
matrix = [[1, 2], [3, 4], [5, 6]]
flat = [x for row in matrix for x in row]
print(flat)
# [1, 2, 3, 4, 5, 6]

# 它等价于下面的普通写法：
flat2 = []
for row in matrix:
    for x in row:
        flat2.append(x)

# 4）展平时顺便过滤
matrix = [[1, -2, 3], [-4, 5], [0, 6]]
positives = [x for row in matrix for x in row if x > 0]
print(positives)
# [1, 3, 5, 6]

# 5）保留二维结构，只筛选每一行中符合条件的元素
matrix = [[1, 2, 3, 4], [5, 6], [7, 8, 9]]
evens_by_row = [[x for x in row if x % 2 == 0] for row in matrix]
print(evens_by_row)
# [[2, 4], [6], [8]]
```

> **记忆顺序**：`[x for row in matrix for x in row]` 可以读成"先拿到每一行 `row`，再从这一行里取每个元素 `x`"。

#### 二维列表初始化的常见坑

下面这两种写法看起来很像，但含义完全不同：

```python
# ❌ 不推荐：3 行其实引用的是同一个列表对象
bad = [[0] * 4] * 3
bad[0][1] = 99
print(bad)
# [[0, 99, 0, 0], [0, 99, 0, 0], [0, 99, 0, 0]]

# ✅ 推荐：每一行都是独立的新列表
good = [[0] * 4 for _ in range(3)]
good[0][1] = 99
print(good)
# [[0, 99, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]
```

> **原因**：`[[0] * 4] * 3` 是把同一个内部列表重复引用了 3 次；而 `for` 推导式会在每次循环时重新创建一行。

### ✅ 小练习（含答案）

**题目**：把二维列表 `matrix=[[1,2,3],[4,5,6],[7,8,9]]` 展平成一维列表，并且只保留偶数。

**参考答案**：

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
result = [x for row in matrix for x in row if x % 2 == 0]
print(result)  # [2, 4, 6, 8]
```

---

## 5.2 元组 `tuple`（有序、不可变）

```python
# 创建
t1 = (1, 2, 3)
t2 = 1, 2, 3        # 括号可以省略
t3 = tuple([1, 2, 3])

# ⚠️ 创建只有一个元素的元组，必须加逗号
single = (42,)       # 这是元组
not_tuple = (42)     # 这只是整数 42！
print(type(single))  # <class 'tuple'>
print(type(not_tuple))  # <class 'int'>

# 空元组
empty = ()
empty = tuple()
```

### 元组是不可变的

```python
t = (1, 2, 3)
# t[0] = 99   # ❌ TypeError: 'tuple' object does not support item assignment

# 但如果元组里有可变对象（如列表），那个可变对象本身还是可以改的
t = (1, [2, 3], 4)
t[1].append(99)
print(t)       # (1, [2, 3, 99], 4) ← 列表被修改了！
```

### 解包

```python
# 基本解包
a, b, c = (1, 2, 3)
print(a, b, c)    # 1 2 3

# 星号解包（Python 3+）
first, *middle, last = [1, 2, 3, 4, 5]
print(first)    # 1
print(middle)   # [2, 3, 4]
print(last)     # 5

# 忽略不需要的值
_, b, _ = (1, 2, 3)  # 只要第二个值

# 函数返回多个值（本质就是返回元组）
def min_max(lst):
    return min(lst), max(lst)

lo, hi = min_max([3, 1, 7, 2])
print(lo, hi)   # 1 7
```

### 元组 vs 列表

| 特性     | 元组 `tuple`         | 列表 `list`      |
| -------- | -------------------- | ---------------- |
| 可变性   | ❌ 不可变             | ✅ 可变           |
| 语法     | `(1, 2, 3)`          | `[1, 2, 3]`      |
| 性能     | 更快，更省内存       | 稍慢             |
| 可哈希   | ✅ 可作字典的 key     | ❌ 不可以         |
| 使用场景 | 固定数据、函数返回值 | 需要增删改的场景 |

---

## 5.3 字典 `dict`（键值对，从 Python 3.7+ 保证有序）

```python
# 创建
d = {"name": "张三", "age": 25, "city": "深圳"}
d = dict(name="张三", age=25, city="深圳")  # 关键字参数方式
d = dict([("name", "张三"), ("age", 25)])    # 从键值对列表

# 空字典
d = {}
d = dict()
```

### 增 / 查 / 改 / 删

```python
d = {"name": "张三", "age": 25}

# 增 / 改（key 不存在则新增，存在则覆盖）
d["city"] = "深圳"       # 新增
d["age"] = 26             # 修改

# 查
print(d["name"])           # "张三"
# print(d["email"])        # ❌ KeyError！key 不存在

# ✅ 安全取值：.get(key, 默认值)
print(d.get("email", "未设置"))   # "未设置"（key 不存在返回默认值）
print(d.get("name", "未设置"))    # "张三"（key 存在返回对应值）

# setdefault：key 不存在时设置默认值并返回
d.setdefault("email", "none@test.com")
print(d["email"])          # "none@test.com"

# 删
del d["email"]             # 删除指定 key
val = d.pop("city")        # 删除并返回值：val = "深圳"
val = d.pop("xxx", None)   # key 不存在返回默认值 None，而不是报错
d.clear()                  # 清空字典

# 检查 key 是否存在
print("name" in d)         # True
print("email" not in d)    # True
```

### 遍历字典

```python
d = {"name": "张三", "age": 25, "city": "深圳"}

# 遍历 key
for key in d:
    print(key, end=" ")          # name age city
for key in d.keys():             # 同上
    print(key, end=" ")

# 遍历 value
for val in d.values():
    print(val, end=" ")          # 张三 25 深圳

# 遍历 key-value（最常用）
for key, val in d.items():
    print(f"{key}: {val}")
```

### 字典推导式

```python
# 基本语法
squares = {x: x**2 for x in range(6)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# 带条件过滤
even_sq = {x: x**2 for x in range(10) if x % 2 == 0}
# {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}

# 交换 key 和 value
original = {"a": 1, "b": 2, "c": 3}
swapped = {v: k for k, v in original.items()}
# {1: 'a', 2: 'b', 3: 'c'}

# 从两个列表构建字典
keys = ["name", "age", "city"]
values = ["张三", 25, "深圳"]
d = dict(zip(keys, values))
# {"name": "张三", "age": 25, "city": "深圳"}
```

### 合并字典

```python
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}

# Python 3.9+ 使用 | 运算符（推荐）
merged = d1 | d2           # {"a": 1, "b": 3, "c": 4}（d2 覆盖 d1 的重复 key）

# Python 3.5+ 使用 ** 解包
merged = {**d1, **d2}      # {"a": 1, "b": 3, "c": 4}

# update() 方法（会修改原字典）
d1.update(d2)               # d1 变成 {"a": 1, "b": 3, "c": 4}
```

### ✅ 小练习（含答案）

**题目**：统计字符串 `"banana"` 每个字符出现次数，输出一个字典。

**参考答案**：

```python
text = "banana"
freq = {}

for ch in text:
    freq[ch] = freq.get(ch, 0) + 1

print(freq)  # {'b': 1, 'a': 3, 'n': 2}
```

---

## 5.4 集合 `set`（无序、不重复）

```python
# 创建
s = {1, 2, 3, 4, 5}
s = set([1, 2, 2, 3, 3, 4])   # 自动去重：{1, 2, 3, 4}

# ⚠️ 创建空集合必须用 set()，不能用 {}
empty_set = set()       # ✅ 空集合
empty_dict = {}         # ❌ 这是空字典！
```

### 增删操作

```python
s = {1, 2, 3}

s.add(4)                # {1, 2, 3, 4}
s.add(3)                # {1, 2, 3, 4}（已存在则忽略）

s.remove(2)             # {1, 3, 4}（元素不存在会报 KeyError）
s.discard(99)           # 不报错（元素不存在也没事）

val = s.pop()           # 删除并返回任意一个元素（不保证顺序）
s.clear()               # 清空
```

### 集合运算

```python
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

# 并集（所有元素）
print(a | b)            # {1, 2, 3, 4, 5, 6, 7, 8}
print(a.union(b))       # 同上

# 交集（共有的元素）
print(a & b)            # {4, 5}
print(a.intersection(b))

# 差集（在 a 中但不在 b 中）
print(a - b)            # {1, 2, 3}
print(a.difference(b))

# 对称差集（不同时属于两者的元素）
print(a ^ b)            # {1, 2, 3, 6, 7, 8}
print(a.symmetric_difference(b))

# 子集 / 超集
print({1, 2} <= {1, 2, 3})    # True（是子集）
print({1, 2, 3} >= {1, 2})    # True（是超集）
```

### 集合的典型应用

```python
# 去重
lst = [1, 2, 2, 3, 3, 3, 4]
unique = list(set(lst))       # [1, 2, 3, 4]（注意顺序可能变化）

# 保持原顺序的去重（Python 3.7+ dict 有序）
unique_ordered = list(dict.fromkeys(lst))  # [1, 2, 3, 4]

# 快速判断成员关系（set 的 in 操作是 O(1)，远快于 list 的 O(n)）
allowed = {"admin", "editor", "viewer"}
role = "admin"
if role in allowed:
    print("权限通过")
```

### 集合推导式

```python
s = {x**2 for x in range(10) if x % 2 == 0}
# {0, 4, 16, 36, 64}
```

### ✅ 小练习（含答案）

**题目**：给定两个集合 `a={1,2,3,4}`、`b={3,4,5}`，输出交集、并集、差集 `a-b`。

**参考答案**：

```python
a = {1, 2, 3, 4}
b = {3, 4, 5}

print(a & b)  # {3, 4}
print(a | b)  # {1, 2, 3, 4, 5}
print(a - b)  # {1, 2}
```

---

## 5.5 嵌套容器

实际开发中，容器经常互相嵌套。

```python
# 列表嵌套字典（最常见：表示"一组记录"）
students = [
    {"name": "张三", "score": 90},
    {"name": "李四", "score": 85},
    {"name": "王五", "score": 92},
]

# 遍历
for stu in students:
    print(f"{stu['name']}: {stu['score']}分")

# 按成绩排序
students.sort(key=lambda s: s["score"], reverse=True)

# 字典嵌套字典
config = {
    "database": {
        "host": "localhost",
        "port": 3306,
        "name": "mydb",
    },
    "server": {
        "host": "0.0.0.0",
        "port": 8080,
    }
}
print(config["database"]["host"])   # localhost

# 列表嵌套列表（矩阵）
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]
print(matrix[1][2])   # 6
```

---

# 第六章 · 函数

---

## 6.1 函数定义与调用

```python
# 基本定义
def greet(name):
    """向指定的人问好。"""   # docstring
    print(f"你好，{name}！")

greet("张三")    # 你好，张三！

# 有返回值的函数
def add(a, b):
    return a + b

result = add(3, 5)
print(result)   # 8

# 返回多个值（本质是返回元组）
def divide(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder

q, r = divide(17, 5)
print(f"商: {q}, 余: {r}")   # 商: 3, 余: 2

# 没有 return 或 return 不带值，函数返回 None
def say_hello():
    print("hello")

result = say_hello()
print(result)           # None
```

---

## 6.2 参数类型

### 位置参数和默认参数

```python
# 位置参数：调用时必须按顺序传入
def power(base, exp):
    return base ** exp

print(power(2, 10))     # 1024

# 默认参数：有默认值，调用时可以省略
def power(base, exp=2):
    return base ** exp

print(power(3))         # 9（exp 默认为 2）
print(power(3, 3))      # 27（覆盖默认值）

# ⚠️ 默认参数的坑：不要用可变对象作默认值！
def bad_append(item, lst=[]):   # ❌ 默认列表被共享
    lst.append(item)
    return lst

print(bad_append(1))    # [1]
print(bad_append(2))    # [1, 2] —— 不是 [2]！默认列表被修改了

# 正确做法
def good_append(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

### `*args`（可变位置参数）

```python
def add_all(*args):
    """接收任意数量的位置参数，打包成元组。"""
    print(type(args))     # <class 'tuple'>
    return sum(args)

print(add_all(1, 2, 3))          # 6
print(add_all(1, 2, 3, 4, 5))    # 15

# 可以和普通参数混合
def greet(greeting, *names):
    for name in names:
        print(f"{greeting}, {name}!")

greet("你好", "张三", "李四", "王五")
```

### `**kwargs`（可变关键字参数）

```python
def print_info(**kwargs):
    """接收任意数量的关键字参数，打包成字典。"""
    print(type(kwargs))    # <class 'dict'>
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="张三", age=25, city="深圳")
# name: 张三
# age: 25
# city: 深圳
```

### 参数顺序规则

```python
# 参数定义顺序（常见形式）通常是：
# 普通参数（可带默认值）→ *args → 仅关键字参数（可带默认值）→ **kwargs
# 另外，还可以用 / 声明“仅位置参数”（见下一节示例）。
def func(a, b=0, *args, c=10, d, **kwargs):
    print(f"a={a}, b={b}, args={args}, c={c}, d={d}, kwargs={kwargs}")

func(1, 2, 3, 4, d=5, extra="hello")
# a=1, b=2, args=(3, 4), c=10, d=5, kwargs={'extra': 'hello'}
```

### 仅关键字参数（`*` 之后的参数）

```python
# * 之后的参数必须通过关键字传递
def make_user(name, *, age, city):
    return {"name": name, "age": age, "city": city}

# make_user("张三", 25, "深圳")   # ❌ TypeError
make_user("张三", age=25, city="深圳")  # ✅

# 仅位置参数（Python 3.8+）：/ 之前的参数必须按位置传递
def greet(name, /, greeting="你好"):
    print(f"{greeting}, {name}")

greet("张三")                # ✅
# greet(name="张三")          # ❌ TypeError
```

### 解包传参

```python
def add(a, b, c):
    return a + b + c

# 用 * 解包列表/元组传给位置参数
nums = [1, 2, 3]
print(add(*nums))      # 6

# 用 ** 解包字典传给关键字参数
params = {"a": 10, "b": 20, "c": 30}
print(add(**params))    # 60

# 组合使用
def func(x, y, z, w):
    print(x, y, z, w)

args = (1, 2)
kwargs = {"z": 3, "w": 4}
func(*args, **kwargs)   # 1 2 3 4
```

### ✅ 小练习（含答案）

**题目**：写函数 `build_user(name, age=18, **kwargs)`，返回合并后的用户字典；调用时额外传入 `city` 和 `role`。

**参考答案**：

```python
def build_user(name, age=18, **kwargs):
    user = {"name": name, "age": age}
    user.update(kwargs)
    return user

u = build_user("张三", city="深圳", role="student")
print(u)  # {'name': '张三', 'age': 18, 'city': '深圳', 'role': 'student'}
```

---

## 6.3 作用域

### 局部变量 vs 全局变量

```python
x = 100          # 全局变量

def foo():
    x = 10       # 局部变量（和全局的 x 是两个独立变量）
    print(x)     # 10

foo()
print(x)         # 100（全局变量没被改）
```

### `global` 关键字

```python
count = 0

def increment():
    global count      # 声明：我要修改全局变量 count
    count += 1

increment()
increment()
print(count)          # 2

# ⚠️ 如果不加 global，直接写 count += 1 会报 UnboundLocalError
# 因为 Python 看到赋值就会把 count 当局部变量，但它还没初始化
```

### `nonlocal` 关键字（嵌套函数）

```python
def outer():
    x = 10

    def inner():
        nonlocal x    # 声明：我要修改外层函数（非全局）的 x
        x += 1
        print(f"inner: x = {x}")

    inner()
    print(f"outer: x = {x}")

outer()
# inner: x = 11
# outer: x = 11
```

> **作用域查找规则 LEGB**（由内到外）：
> - **L**ocal → **E**nclosing（外层函数） → **G**lobal → **B**uilt-in

---

## 6.4 匿名函数 `lambda`

```python
# lambda 参数: 表达式
# 等价于只有一行 return 的函数

double = lambda x: x * 2
print(double(5))     # 10

add = lambda a, b: a + b
print(add(3, 4))     # 7

# 最常见的用法：作为 key 函数传给 sorted / max / min
students = [("张三", 90), ("李四", 78), ("王五", 95)]

# 按成绩排序
students.sort(key=lambda s: s[1])
print(students)  # [('李四', 78), ('张三', 90), ('王五', 95)]

# 按成绩降序
students.sort(key=lambda s: s[1], reverse=True)

# 配合 map：对每个元素应用函数
nums = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, nums))
# [1, 4, 9, 16, 25]

# 配合 filter：过滤元素
evens = list(filter(lambda x: x % 2 == 0, nums))
# [2, 4]
```

> **提示**：lambda 适合短小的一次性函数。如果逻辑复杂，还是用 `def` 定义正式函数，可读性更好。

---

## 6.5 递归函数

```python
# 阶乘：n! = n × (n-1) × ... × 1
def factorial(n):
    if n <= 1:         # 终止条件（base case）
        return 1
    return n * factorial(n - 1)   # 递归调用

print(factorial(5))    # 120

# 斐波那契数列（经典但低效的写法）
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(10))   # 55

# ⚠️ 上面的 fib 是指数级复杂度，改用记忆化
from functools import lru_cache

@lru_cache(maxsize=None)
def fib_fast(n):
    if n <= 1:
        return n
    return fib_fast(n - 1) + fib_fast(n - 2)

print(fib_fast(100))  # 354224848179261915075（瞬间完成）

# Python 默认递归深度限制 1000，可以修改（但不推荐太大）
import sys
print(sys.getrecursionlimit())    # 1000
# sys.setrecursionlimit(10000)    # 修改上限
```

---

## 6.6 类型注解（Type Hints）基础

类型注解不会改变 Python 的运行行为，但能显著提升代码可读性、IDE 补全与静态检查质量。

```python
from typing import Iterable

def add(a: int, b: int) -> int:
    return a + b

def average(values: Iterable[float]) -> float:
    values = list(values)
    if not values:
        raise ValueError("values 不能为空")
    return sum(values) / len(values)

name: str = "张三"
scores: list[int] = [90, 85, 92]

print(add(3, 5))                 # 8
print(average(scores))           # 89.0
```

> **说明**：类型注解本身不会在运行时自动拦截错误（`add("1", 2)` 仍可能运行到出错处）；它主要服务于静态分析工具（如 Pylance、mypy）和团队协作。

### 常见写法补充

```python
def greet(name: str | None = None) -> str:
    if name is None:
        return "Hello, World!"
    return f"Hello, {name}!"

def first(items: list[int]) -> int | None:
    if items:
        return items[0]
    return None
```

> 在 Python 3.10+ 中，联合类型通常直接写成 `A | B`；旧版本也可写作 `Optional[str]` 或 `Union[int, None]`。

---

# 第七章 · 文件操作

---

## 7.1 打开与关闭文件

```python
# 基本语法
f = open("test.txt", "r", encoding="utf-8")
content = f.read()
f.close()       # ⚠️ 必须手动关闭，否则可能占用资源

# ✅ 推荐写法：with 语句（自动关闭，即使出错也会关）
with open("test.txt", "r", encoding="utf-8") as f:
    content = f.read()
# 离开 with 块后，f 自动关闭
```

### 文件打开模式

| 模式            | 说明                         | 文件不存在时           |
| --------------- | ---------------------------- | ---------------------- |
| `"r"`           | 只读（默认）                 | 报错 FileNotFoundError |
| `"w"`           | 覆盖写入                     | 自动创建               |
| `"a"`           | 追加写入                     | 自动创建               |
| `"x"`           | 排他创建（文件已存在则报错） | 自动创建               |
| `"r+"`          | 读写（文件必须存在）         | 报错                   |
| `"w+"`          | 读写（覆盖）                 | 自动创建               |
| `"rb"` / `"wb"` | 二进制读 / 写                | 同上                   |

### ✅ 小练习（含答案）

**题目**：把 `"你好\nPython"` 写入 `demo.txt`，再读出并打印。

**参考答案**：

```python
with open("demo.txt", "w", encoding="utf-8") as f:
    f.write("你好\nPython")

with open("demo.txt", "r", encoding="utf-8") as f:
    print(f.read())
```

---

## 7.2 读文件

```python
with open("data.txt", "r", encoding="utf-8") as f:
    # 方式1：一次读取全部内容
    content = f.read()            # 返回整个字符串

    # 方式2：读一行
    f.seek(0)                     # 指针回到开头
    first_line = f.readline()     # 读一行（含 \n）

    # 方式3：读所有行到列表
    f.seek(0)
    lines = f.readlines()         # ["第一行\n", "第二行\n", ...]

    # 方式4：逐行遍历（内存友好，推荐用于大文件）
    f.seek(0)
    for line in f:
        print(line.strip())       # strip() 去掉末尾 \n
```

---

## 7.3 写文件

```python
# 覆盖写入
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("第一行\n")
    f.write("第二行\n")

# 追加写入
with open("output.txt", "a", encoding="utf-8") as f:
    f.write("追加的内容\n")

# 写入多行
lines = ["aaa\n", "bbb\n", "ccc\n"]
with open("output.txt", "w", encoding="utf-8") as f:
    f.writelines(lines)     # ⚠️ writelines 不会自动加换行符
```

---

## 7.4 文件与路径操作

### `pathlib` 模块（**现代推荐方式**）

```python
from pathlib import Path

# 创建路径对象
p = Path("./data") / "file.txt"    # 用 / 拼接路径
print(p)                            # data/file.txt（或在 Windows 上显示为 data\file.txt）

# 常用属性
print(p.name)        # file.txt（文件名）
print(p.stem)        # file（不含扩展名）
print(p.suffix)      # .txt（扩展名）
print(p.parent)      # data（父目录）
print(p.is_file())   # True/False
print(p.is_dir())    # True/False
print(p.exists())    # True/False

# 创建目录
Path("./output/sub").mkdir(parents=True, exist_ok=True)

# 读写文件（pathlib 提供的便捷方法）
p = Path("test.txt")
p.write_text("hello world", encoding="utf-8")
content = p.read_text(encoding="utf-8")

# 遍历目录
for f in Path(".").iterdir():       # 当前目录下所有文件/目录
    print(f)

for f in Path(".").glob("*.txt"):   # 匹配模式
    print(f)

for f in Path(".").rglob("*.py"):   # 递归搜索所有子目录
    print(f)
```

### `os` 模块（传统方式，了解即可）

```python
import os

print(os.path.exists("test.txt"))       # 判断路径是否存在
print(os.path.isfile("test.txt"))       # 是否是文件
print(os.path.isdir("./data"))          # 是否是目录
print(os.listdir("."))                  # 列出目录内容
os.makedirs("./output/sub", exist_ok=True)  # 递归创建目录
os.path.join("data", "file.txt")        # 拼接路径："data/file.txt"
```

---

## 7.5 异常处理（配合文件操作）

```python
try:
    with open("不存在的文件.txt", "r", encoding="utf-8") as f:
        content = f.read()
except FileNotFoundError as e:
    print(f"文件不存在：{e}")
except PermissionError:
    print("没有读取权限")
finally:
    print("无论成功与否都会执行这里")
```

---

## 7.6 编码、换行与科研数据文件（补充）

面向教学、科研与跨平台协作时，文件读写除了“能打开”之外，更要关注**编码一致性**与**换行一致性**。中文文本若依赖系统默认编码，在不同机器上很容易出现乱码；因此实践中建议始终显式写出 `encoding="utf-8"`。

```python
from pathlib import Path

text_path = Path("notes.txt")

# 始终显式指定编码，避免“我的电脑没问题，别人打开却乱码”
text_path.write_text("第一行\n第二行\n", encoding="utf-8")
content = text_path.read_text(encoding="utf-8")

# 需要精确控制换行时，可以指定 newline
with open("report.txt", "w", encoding="utf-8", newline="\n") as f:
    f.write("标题\n")
    f.write("内容\n")

# 二进制文件（图片、PDF、Excel 等）不要指定 encoding
with open("figure.png", "rb") as f:
    raw = f.read(16)
```

> **实践建议**：
> 1. 文本文件默认优先使用 UTF-8。
> 2. 二进制文件使用 `rb` / `wb`，不要混用文本模式。
> 3. 若与 Excel、CSV、日志系统交互，先确认对方要求的编码与分隔符，再决定是否继续使用 UTF-8。
> 4. 某些 Windows 下的 Excel 对 UTF-8 CSV 兼容性一般，必要时可单独评估 `utf-8-sig`，但教学示例默认仍以 UTF-8 为主。

---

# 第八章 · 异常处理

---

## 8.1 基本结构

```python
try:
    num = int(input("输入一个数字："))
    result = 100 / num
except ValueError:
    print("输入的不是有效数字")
except ZeroDivisionError:
    print("不能除以零")
else:
    # 没有任何异常时执行
    print(f"结果是：{result}")
finally:
    # 无论如何都执行（常用于释放资源）
    print("计算结束")
```

> **执行流程**：
> - 有异常 → `try` → 对应的 `except` → `finally`
> - 无异常 → `try` → `else` → `finally`

### ✅ 小练习（含答案）

**题目**：实现 `safe_div(a, b)`，正常时返回结果；除数为 0 返回 `None` 并打印错误信息；最后总是打印“结束”。

**参考答案**：

```python
def safe_div(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("除数不能为 0")
        return None
    finally:
        print("结束")

print(safe_div(10, 2))  # 5.0
print(safe_div(10, 0))  # None
```

---

## 8.2 捕获多种异常

```python
# 方式1：多个 except 分别处理
try:
    x = int("abc")
except ValueError:
    print("值错误")
except TypeError:
    print("类型错误")

# 方式2：合并处理
try:
    x = int("abc")
except (ValueError, TypeError) as e:
    print(f"出错了：{e}")

# 方式3：捕获所有异常（慎用！会掩盖 bug）
try:
    x = 1 / 0
except Exception as e:
    print(f"捕获到异常：{type(e).__name__}: {e}")
    # 捕获到异常：ZeroDivisionError: division by zero
```

> **补充说明**：
> - `except Exception:` 比“裸 `except:`”更安全，因为它不会吞掉 `KeyboardInterrupt`、`SystemExit` 等系统级异常。
> - 教学代码中应尽量捕获“你预期会发生的异常”，不要无差别兜底。

---

## 8.3 主动抛出异常

```python
def set_age(age):
    if not isinstance(age, int):
        raise TypeError("年龄必须是整数")
    if age < 0 or age > 150:
        raise ValueError(f"年龄 {age} 不合理")
    print(f"设置年龄为 {age}")

try:
    set_age(-5)
except ValueError as e:
    print(e)   # 年龄 -5 不合理
```

---

## 8.4 自定义异常类

```python
class InsufficientBalanceError(Exception):
    """余额不足异常"""
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(f"余额不足：当前 {balance}，需要 {amount}")

class BankAccount:
    def __init__(self, balance=0):
        self.balance = balance

    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientBalanceError(self.balance, amount)
        self.balance -= amount
        return self.balance

account = BankAccount(100)
try:
    account.withdraw(200)
except InsufficientBalanceError as e:
    print(e)                # 余额不足：当前 100，需要 200
    print(e.balance)        # 100
    print(e.amount)         # 200
```

---

## 8.5 异常链、重新抛出与断言（重要补充）

大型项目中，异常不只是“报错了就打印一下”，还承担着传递上下文、定位根因的职责。

### 异常链：`raise ... from ...`

```python
def parse_age(text):
    try:
        return int(text)
    except ValueError as e:
        raise ValueError(f"无法把 {text!r} 解析为年龄") from e

try:
    parse_age("十八")
except ValueError as e:
    print(e)
```

> `from e` 的作用是保留原始异常链。调试时，你既能看到“表面错误”，也能看到“最初是哪里失败的”。

### 重新抛出当前异常

```python
try:
    1 / 0
except ZeroDivisionError:
    print("记录日志后继续向上抛出")
    raise
```

### `assert` 适合检查“程序员假设”

```python
def divide(a, b):
    assert b != 0, "b 不应为 0"
    return a / b
```

> **注意**：
> - `assert` 主要用于开发和调试阶段验证内部假设。
> - 面向用户输入、文件内容、网络数据等外部不可信数据时，应优先使用 `if ...: raise ...` 做显式校验，而不是依赖 `assert`。

---

# 第九章 · 高级特性与常用内置函数

---

## 9.1 推导式（Comprehension）

```python
# 列表推导式（前面讲过，这里补充更多用法）
flat = [x for row in [[1,2],[3,4],[5,6]] for x in row]
# [1, 2, 3, 4, 5, 6]（展平嵌套列表）

# 字典推导式
word = "hello"
freq = {ch: word.count(ch) for ch in set(word)}
# {'h': 1, 'e': 1, 'l': 2, 'o': 1}

# 集合推导式
s = {len(w) for w in ["apple", "banana", "cherry", "date"]}
# {4, 5, 6}（所有单词长度的集合）

# 生成器表达式（惰性求值，省内存）
gen = (x**2 for x in range(1_000_000))
print(type(gen))       # <class 'generator'>
print(next(gen))       # 0
print(next(gen))       # 1
print(sum(gen))        # 计算剩余元素平方和（不会占用大量内存）

# 对比内存占用
import sys
lst = [x**2 for x in range(10000)]     # 列表：立即计算所有值
gen = (x**2 for x in range(10000))     # 生成器：按需计算
print(sys.getsizeof(lst))              # ~87616 字节
print(sys.getsizeof(gen))              # ~200 字节（恒定很小）
```

---

## 9.2 常用内置函数

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]

# map：对每个元素应用函数，返回迭代器
result = list(map(str, nums))           # ['3','1','4','1','5','9','2','6']
result = list(map(lambda x: x*2, nums)) # [6, 2, 8, 2, 10, 18, 4, 12]

# filter：过滤元素，返回迭代器
result = list(filter(lambda x: x > 3, nums))  # [4, 5, 9, 6]

# zip：打包多个可迭代对象
names = ["张三", "李四"]
scores = [90, 85]
print(list(zip(names, scores)))   # [('张三', 90), ('李四', 85)]

# sorted：返回排序后的新列表
print(sorted(nums))                          # [1, 1, 2, 3, 4, 5, 6, 9]
print(sorted(nums, reverse=True))            # [9, 6, 5, 4, 3, 2, 1, 1]
print(sorted(nums, key=lambda x: -x))       # 同上

# 数学类
print(sum(nums))           # 31
print(min(nums))           # 1
print(max(nums))           # 9
print(abs(-42))            # 42
print(round(3.14159, 2))   # 3.14

# 逻辑判断
print(any([False, False, True]))   # True（有一个为真就返回 True）
print(all([True, True, False]))    # False（全部为真才返回 True）
print(any([]))                     # False
print(all([]))                     # True（空序列 all 返回 True）

# 类型转换
print(list("abc"))          # ['a', 'b', 'c']
print(tuple([1, 2, 3]))    # (1, 2, 3)
print(set([1, 1, 2]))      # {1, 2}
print(dict([("a",1)]))     # {'a': 1}
```

---

## 9.3 装饰器 `@decorator`

装饰器本质上是一个**接收函数并返回新函数**的函数。

```python
# 理解基础：函数可以作为参数传递
def shout(text):
    return text.upper()

def whisper(text):
    return text.lower()

def greet(func):
    print(func("Hello, World"))

greet(shout)     # HELLO, WORLD
greet(whisper)   # hello, world
```

### 编写装饰器

```python
import time
from functools import wraps

def timer(func):
    """计时装饰器"""
    @wraps(func)   # 保留原函数的名称和文档
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} 执行耗时：{elapsed:.4f}秒")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "完成"

result = slow_function()   # slow_function 执行耗时：1.0012秒
```

> `@timer` 等价于 `slow_function = timer(slow_function)`

### 带参数的装饰器

```python
def repeat(n):
    """让函数重复执行 n 次"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

say_hello()
# Hello!
# Hello!
# Hello!
```

### 内置装饰器速览

```python
class MyClass:
    class_var = "类变量"

    @staticmethod
    def static_method():
        """静态方法：不需要 self 或 cls，就是个普通函数放在类里"""
        print("我是静态方法")

    @classmethod
    def class_method(cls):
        """类方法：第一个参数是 cls（类本身）"""
        print(f"类变量：{cls.class_var}")

    @property
    def info(self):
        """属性方法：像访问属性一样调用方法"""
        return "一些信息"

obj = MyClass()
MyClass.static_method()    # 我是静态方法
MyClass.class_method()     # 类变量：类变量
print(obj.info)            # 一些信息（注意没有括号！）
```

---

## 9.4 生成器与 `yield`

生成器是一种特殊的迭代器，用 `yield` 代替 `return`，**按需产出值**，极其省内存。

```python
def count_up(n):
    """从 1 数到 n"""
    i = 1
    while i <= n:
        yield i       # 暂停函数，返回 i，下次调用从这里继续
        i += 1

# 使用方式和普通迭代器一样
for num in count_up(5):
    print(num, end=" ")    # 1 2 3 4 5

# 手动驱动
gen = count_up(3)
print(next(gen))    # 1
print(next(gen))    # 2
print(next(gen))    # 3
# print(next(gen))  # ❌ StopIteration

# 实际应用：读取大文件的每一行
def read_large_file(path):
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            yield line.strip()

# 无限序列（不可能用列表存储）
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci()
for _ in range(10):
    print(next(fib), end=" ")   # 0 1 1 2 3 5 8 13 21 34
```

---

## 9.5 上下文管理器

`with` 语句背后依赖于上下文管理器协议：`__enter__` 和 `__exit__`。

```python
# 自定义上下文管理器（类方式）
class Timer:
    def __enter__(self):
        import time
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        self.elapsed = time.perf_counter() - self.start
        print(f"耗时：{self.elapsed:.4f}秒")
        return False   # False 表示不吞掉异常

with Timer():
    total = sum(range(1_000_000))
# 耗时：0.0312秒

# 更简洁的方式：用 contextlib
from contextlib import contextmanager

@contextmanager
def timer():
    import time
    start = time.perf_counter()
    try:
        yield                         # yield 之前 = __enter__
    finally:
        elapsed = time.perf_counter() - start
        print(f"耗时：{elapsed:.4f}秒")  # yield 之后 = __exit__

with timer():
    total = sum(range(1_000_000))
```

---

# 第十章 · 模块与包

---

## 10.1 导入模块

```python
# 整体导入
import os
print(os.getcwd())

# 导入特定内容
from math import sqrt, pi
print(sqrt(16))    # 4.0
print(pi)          # 3.141592653589793

# 别名
import pathlib as pl
from datetime import datetime as dt

print(pl.Path.cwd())
print(dt.now())

# 导入所有（不推荐，容易命名冲突）
# from math import *
```

---

## 10.2 常用标准库速览

### `os` / `sys`

```python
import os, sys
print(os.getcwd())                # 当前工作目录
if "__file__" in globals():
    print(os.path.abspath(__file__))  # 当前文件绝对路径（脚本环境）
else:
    print("当前在交互环境中，__file__ 不存在")
print(sys.argv)                   # 命令行参数列表
print(sys.version)                # Python 版本
print(sys.platform)               # 操作系统平台
```

### `math` / `random`

```python
import math, random

print(math.ceil(3.2))      # 4（向上取整）
print(math.floor(3.8))     # 3（向下取整）
print(math.gcd(12, 8))     # 4（最大公约数）
print(math.log(100, 10))   # 2.0

print(random.randint(1, 10))        # 1~10 随机整数
print(random.choice(["a","b","c"])) # 随机选一个
print(random.random())              # 0~1 随机浮点数

lst = [1, 2, 3, 4, 5]
random.shuffle(lst)                 # 原地打乱
print(random.sample(lst, 3))        # 随机取 3 个（不修改原列表）
```

### `datetime`

```python
from datetime import datetime, timedelta

now = datetime.now()
print(now)                              # 2026-03-18 22:18:00.123456
print(now.strftime("%Y-%m-%d %H:%M"))   # 2026-03-18 22:18

# 字符串 → datetime
dt = datetime.strptime("2026-03-18", "%Y-%m-%d")

# 时间运算
tomorrow = now + timedelta(days=1)
diff = datetime(2026, 12, 31) - now
print(diff.days)    # 剩余天数
```

### `json`

```python
import json

data = {"name": "张三", "age": 25, "scores": [90, 85, 92]}

# Python 对象 → JSON 字符串
json_str = json.dumps(data, ensure_ascii=False, indent=2)
print(json_str)

# JSON 字符串 → Python 对象
obj = json.loads(json_str)
print(obj["name"])

# 读写 JSON 文件
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

with open("data.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)
```

### `re`（正则表达式）

```python
import re

text = "电话: 138-1234-5678，邮箱: test@example.com"

# 查找
phone = re.search(r"\d{3}-\d{4}-\d{4}", text)
print(phone.group())     # 138-1234-5678

# 查找所有
numbers = re.findall(r"\d+", text)
print(numbers)           # ['138', '1234', '5678']

# 替换
clean = re.sub(r"\d", "*", text)
print(clean)             # 电话: ***-****-****，邮箱: test@example.com

# 分割
parts = re.split(r"[,，]", text)
```

### `collections`

```python
from collections import Counter, defaultdict, deque

# Counter：计数器
c = Counter("abracadabra")
print(c)                      # Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
print(c.most_common(2))       # [('a', 5), ('b', 2)]

# defaultdict：带默认值的字典
dd = defaultdict(list)
dd["fruits"].append("苹果")   # 不用先检查 key 是否存在
dd["fruits"].append("香蕉")
print(dd)                      # defaultdict(<class 'list'>, {'fruits': ['苹果', '香蕉']})
print(dict(dd))                # {'fruits': ['苹果', '香蕉']}

# deque：双端队列（两端增删都是 O(1)）
dq = deque([1, 2, 3])
dq.appendleft(0)               # deque([0, 1, 2, 3])
dq.append(4)                   # deque([0, 1, 2, 3, 4])
dq.popleft()                   # 0
dq.rotate(2)                   # 向右旋转 2 步
```

---

## 10.3 自定义模块与包

### 模块：一个 `.py` 文件就是一个模块

```python
# myutils.py
def greet(name):
    return f"Hello, {name}!"

PI = 3.14159

# 在另一个文件中使用
# import myutils
# print(myutils.greet("张三"))
# from myutils import greet, PI
```

### 包：通常是含 `__init__.py` 的目录

```
mypackage/
├── __init__.py      # 可以为空，传统项目里通常保留它
├── math_utils.py
└── string_utils.py
```

```python
# 使用
from mypackage import math_utils
from mypackage.string_utils import some_function
```

> **补充说明**：自 Python 3.3 起，也存在**命名空间包（namespace package）**，某些场景下目录里可以没有 `__init__.py`。但对初学者、教学项目和团队协作来说，保留 `__init__.py` 通常更直观，也更少踩坑。

### `if __name__ == "__main__":`

```python
# mymodule.py
def main():
    print("作为主程序运行")

if __name__ == "__main__":
    # 只有直接运行这个文件时才执行
    # 被 import 时不执行
    main()
```

> **原理**：当直接运行 `python mymodule.py` 时，`__name__` 的值是 `"__main__"`；当被 import 时，`__name__` 是模块名（如 `"mymodule"`）。

---

## 10.4 虚拟环境与第三方包管理（补充）

对教学和团队项目而言，强烈建议“每个项目一个虚拟环境”，避免依赖冲突。

```bash
# 1) 创建虚拟环境（在项目根目录执行）
python -m venv .venv

# 2) 激活虚拟环境
# Windows PowerShell:
.venv\Scripts\Activate.ps1
# Windows CMD:
# .venv\Scripts\activate.bat
# macOS / Linux:
# source .venv/bin/activate

# 3) 安装依赖
python -m pip install requests

# 4) 冻结依赖
python -m pip freeze > requirements.txt

# 5) 还原依赖
python -m pip install -r requirements.txt
```

> **实践建议**：优先使用 `python -m pip ...` 而非直接 `pip ...`，可避免多 Python 环境下装错解释器。

---

# 第十一章 · 面向对象基础

---

## 11.1 类与对象

```python
class Student:
    """学生类"""

    # 类属性（所有实例共享）
    school = "深圳技术大学"

    def __init__(self, name, age):
        """构造方法：创建实例时自动调用"""
        # 实例属性（每个实例独有）
        self.name = name
        self.age = age
        self.scores = []

    def add_score(self, score):
        """实例方法：第一个参数永远是 self"""
        self.scores.append(score)

    def average(self):
        if not self.scores:
            return 0
        return sum(self.scores) / len(self.scores)

# 创建实例
stu1 = Student("张三", 20)
stu2 = Student("李四", 21)

stu1.add_score(90)
stu1.add_score(85)
print(f"{stu1.name}，平均分：{stu1.average()}")   # 张三，平均分：87.5
print(f"学校：{Student.school}")                   # 深圳技术大学
```

### ✅ 小练习（含答案）

**题目**：基于 `Student` 类再创建一个学生对象，添加 3 门成绩并输出平均分。

**参考答案**：

```python
stu3 = Student("王五", 19)
stu3.add_score(88)
stu3.add_score(91)
stu3.add_score(79)

print(stu3.name, round(stu3.average(), 2))  # 王五 86.0
```

---

## 11.2 属性与方法

```python
class MyClass:
    class_var = 0          # 类属性

    def __init__(self, x):
        self.x = x         # 实例属性

    def instance_method(self):
        """实例方法：通过 self 访问实例"""
        return f"实例方法，x = {self.x}"

    @classmethod
    def class_method(cls):
        """类方法：通过 cls 访问类"""
        cls.class_var += 1
        return f"类方法，class_var = {cls.class_var}"

    @staticmethod
    def static_method(a, b):
        """静态方法：和类/实例无关，就是个普通函数"""
        return a + b

obj = MyClass(42)
print(obj.instance_method())           # 实例方法，x = 42
print(MyClass.class_method())          # 类方法，class_var = 1
print(MyClass.static_method(3, 5))     # 8
```

---

## 11.3 继承与多态

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."

class Dog(Animal):
    def speak(self):           # 方法重写（Override）
        return f"{self.name}: 汪汪！"

class Cat(Animal):
    def speak(self):
        return f"{self.name}: 喵喵！"

# 多态：不同类型的对象调用同名方法，表现不同
animals = [Dog("旺财"), Cat("咪咪"), Dog("大黄")]
for animal in animals:
    print(animal.speak())
# 旺财: 汪汪！
# 咪咪: 喵喵！
# 大黄: 汪汪！

# super()：调用父类方法
class Puppy(Dog):
    def __init__(self, name, toy):
        super().__init__(name)    # 调用父类 Dog 的 __init__
        self.toy = toy

    def speak(self):
        base = super().speak()    # 调用父类 Dog 的 speak
        return f"{base} (最爱玩{self.toy})"

p = Puppy("小白", "球")
print(p.speak())   # 小白: 汪汪！ (最爱玩球)

# 检查继承关系
print(isinstance(p, Puppy))    # True
print(isinstance(p, Dog))      # True
print(isinstance(p, Animal))   # True
print(issubclass(Dog, Animal)) # True
```

---

## 11.4 常用魔法方法（Magic Methods / Dunder Methods）

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        """print() 和 str() 调用"""
        return f"Vector({self.x}, {self.y})"

    def __repr__(self):
        """交互式环境和调试时显示"""
        return f"Vector({self.x!r}, {self.y!r})"

    def __len__(self):
        """len() 调用（这里返回维度数）"""
        return 2

    def __eq__(self, other):
        """== 比较"""
        if not isinstance(other, Vector):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __lt__(self, other):
        """< 比较（定义后 sorted 可以直接排序）"""
        if not isinstance(other, Vector):
            return NotImplemented
        return (self.x**2 + self.y**2) < (other.x**2 + other.y**2)

    def __add__(self, other):
        """+ 运算符"""
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)

    def __getitem__(self, index):
        """支持索引访问 v[0], v[1]"""
        if index == 0: return self.x
        if index == 1: return self.y
        raise IndexError("Vector 只有 2 个分量")

v1 = Vector(1, 2)
v2 = Vector(3, 4)

print(v1)              # Vector(1, 2)
print(v1 + v2)         # Vector(4, 6)
print(v1 == Vector(1, 2))  # True
print(v1 < v2)         # True
print(len(v1))         # 2
print(v1[0], v1[1])    # 1 2

# 排序也能用了
vectors = [Vector(3,4), Vector(1,1), Vector(2,3)]
print(sorted(vectors))   # [Vector(1, 1), Vector(2, 3), Vector(3, 4)]
```

> 更多魔法方法：`__hash__`（哈希）、`__contains__`（`in` 运算符）、`__iter__`（迭代）、`__call__`（使实例可调用）等。

---

## 11.5 `dataclass`（现代 Python 常用写法）

如果一个类的主要职责是“保存数据”，手写 `__init__`、`__repr__`、`__eq__` 往往比较重复。这时可以使用标准库 `dataclasses` 简化样板代码。

```python
from dataclasses import dataclass, field

@dataclass
class Student:
    name: str
    age: int
    scores: list[float] = field(default_factory=list)

    def average(self) -> float:
        if not self.scores:
            return 0.0
        return sum(self.scores) / len(self.scores)

stu = Student("张三", 20, [90, 85, 92])
print(stu)               # Student(name='张三', age=20, scores=[90, 85, 92])
print(stu.average())     # 89.0
```

> **为什么这里要用 `field(default_factory=list)`，而不是直接写 `scores=[]`？**
> 因为它和函数默认参数一样，会带来“多个实例共享同一个可变对象”的问题。`default_factory` 会为每个实例创建新的列表。

---

## 11.6 封装与 `@property`（教学中常被遗漏）

面向对象不只是“把函数写进类里”，还包括**封装**: 对外暴露稳定接口，对内约束数据合法性。`@property` 是 Python 中最常见也最实用的封装工具之一。

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score    # 实际会走到下面的 setter

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not 0 <= value <= 100:
            raise ValueError("score 必须在 0~100 之间")
        self._score = value

stu = Student("张三", 95)
print(stu.score)      # 95

stu.score = 88
# stu.score = 120     # ❌ ValueError
```

> **为什么不直接公开属性？**
> 因为一旦这个值需要校验、转换、延迟计算或兼容旧接口时，`@property` 可以在不改变调用方式（`obj.score`）的前提下平滑演进。

---

# 📋 全教程总结

| 章节               | 核心要点                                                                                          |
| ------------------ | ------------------------------------------------------------------------------------------------- |
| **一、I/O与注释**  | `print()` 的 `sep/end` 参数；f-string **首选**；`input()` 返回 `str`                              |
| **二、变量与类型** | 动态类型；`a,b = b,a` 交换；浮点精度问题；Falsy 值列表；`None` 与真值判断；理解可变/不可变对象    |
| **三、条件控制**   | 缩进即代码块；`is` 比身份、`==` 比值；`match/case` 模式匹配                                       |
| **四、循环控制**   | `enumerate` + `zip` 是利器；`for...else` 表示"没 break"                                           |
| **五、数据容器**   | list 可变、tuple 不可变、dict 有序键值对、set 去重；推导式                                        |
| **六、函数**       | `*args/**kwargs`；默认参数勿用可变对象；LEGB 作用域；lambda                                       |
| **七、文件操作**   | `with open() as f` 是标准写法；文本文件显式指定 `encoding="utf-8"`；`pathlib` 优先于 `os.path`    |
| **八、异常处理**   | `try/except/else/finally`；自定义异常继承 `Exception`；会用异常链保留上下文                       |
| **九、高级特性**   | 生成器省内存；装饰器 = 函数套函数；上下文管理器                                                   |
| **十、模块与包**   | `__init__.py` 定义包；`if __name__ == "__main__"` 防误执行；掌握 venv + pip 基础流程              |
| **十一、OOP**      | `__init__` + `self`；继承 + `super()`；魔法方法定制行为；`dataclass` 简化数据类；`@property` 封装 |

