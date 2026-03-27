# Python 核心语法 —— 函数基础与进阶 · 类型注解 完整教程

---

## 第58讲：函数基础 —— 函数定义

### 一、为什么需要函数？

在你学习 Python 的过程中，你一定写过这样的代码：

```python
# 计算 1+2+3+...+100
total = 0
for i in range(1, 101):
    total += i
print(f"1到100的和为: {total}")

# 又需要计算 1+2+3+...+50
total = 0
for i in range(1, 51):
    total += i
print(f"1到50的和为: {total}")

# 又需要计算 1+2+3+...+200
total = 0
for i in range(1, 201):
    total += i
print(f"1到200的和为: {total}")
```

你会发现，**相同的逻辑被重复书写了三次**。这带来了几个严重的问题：

1. **代码冗余**：相同的代码反复出现，浪费空间。
2. **维护困难**：如果逻辑需要修改（例如改成求平方和），你需要修改每一处。
3. **可读性差**：长篇累牍的代码让人难以快速理解程序结构。

**函数（Function）** 就是为了解决这些问题而存在的。函数允许你把一段逻辑"打包"起来，给它起一个名字，以后每次需要执行这段逻辑时，只需要"调用"这个名字即可。

### 二、函数的基本概念

> **函数**是一段**组织好的、可重复使用的代码块**，用来实现**单一的、或相关联的**功能。

你可以将函数类比为现实生活中的**自动售货机**：
- 你投入硬币（传入参数）
- 售货机内部执行一系列操作（函数体）
- 吐出一瓶饮料（返回值）

你不需要知道售货机内部是怎么运转的，你只需要知道：投入什么，得到什么。这就是编程中常说的**封装**和**抽象**。

### 三、Python 中函数的分类

| 分类             | 说明                              | 示例                                    |
| ---------------- | --------------------------------- | --------------------------------------- |
| **内置函数**     | Python 解释器自带的函数，开箱即用 | `print()`, `len()`, `type()`, `input()` |
| **标准库函数**   | 需要 `import` 导入后使用          | `math.sqrt()`, `random.randint()`       |
| **第三方库函数** | 需要 `pip install` 安装后使用     | `requests.get()`, `numpy.array()`       |
| **自定义函数**   | 程序员根据需求自己编写的函数      | 本讲的重点                              |

前三类你已经用过很多了。本讲开始，我们系统学习**自定义函数**。

### 四、函数定义的语法

```python
def 函数名(参数列表):
    """函数说明文档（可选）"""
    函数体（一条或多条语句）
    return 返回值  # 可选
```

让我们**逐一拆解**每个组成部分：

#### 4.1 `def` 关键字

`def` 是 **define（定义）** 的缩写，它告诉 Python 解释器："我要定义一个函数了"。

```python
def say_hello():
    print("Hello, World!")
```

#### 4.2 函数名

函数名的命名规则与变量名完全一致：

| 规则                       | 合法示例                              | 非法示例                      |
| -------------------------- | ------------------------------------- | ----------------------------- |
| 只能包含字母、数字、下划线 | `calculate_sum`                       | `calculate-sum`（连字符不行） |
| 不能以数字开头             | `area2d`                              | `2d_area`                     |
| 不能使用 Python 关键字     | `my_list`                             | `for`（关键字不能作为标识符） |
| 不建议与内置名同名         | `user_list`                           | `list`（会遮蔽内置类型名）    |
| 区分大小写                 | `myFunc` 和 `myfunc` 是两个不同的函数 | —                             |

**Python 函数命名约定（PEP 8 规范）：**

```python
# ✅ 推荐：使用 snake_case（小写字母 + 下划线）
def calculate_area():
    pass

def get_user_name():
    pass

def is_valid_email():    # 返回布尔值的函数，常用 is_ 开头
    pass

def has_permission():    # 同上，常用 has_ 开头
    pass

# ❌ 不推荐（但不会报错）
def CalculateArea():     # 这是类名的风格（PascalCase），不适合函数
    pass

def calculateArea():     # 这是 Java/JavaScript 风格（camelCase）
    pass
```

> **重要提示**：不要用内置函数名作为自己的函数名！

```python
# ❌ 极其危险的做法
def print(msg):
    pass  # 你覆盖了内置的 print 函数！

# 在此之后，原本的 print() 将无法使用
# print("hello")  # 不会输出任何内容
```

#### 4.3 参数列表

参数列表放在函数名后面的**圆括号**中。参数是函数的"入口"，用于从外部接收数据。

```python
# 无参数的函数
def greet():
    print("你好！")

# 一个参数的函数
def greet_someone(name):
    print(f"你好，{name}！")

# 多个参数的函数
def add(a, b):
    print(f"{a} + {b} = {a + b}")
```

> 注意：即使函数没有参数，**圆括号也不能省略**！

#### 4.4 冒号 `:`

函数定义行的末尾必须有冒号，这和 `if`、`for`、`while` 等语句一致——冒号表示"下面是一个代码块"。

#### 4.5 函数体

函数体是函数被调用时**实际执行的代码**。函数体必须**缩进**（Python 推荐 4 个空格）。

```python
def greet():
    # 以下两行都是函数体
    print("你好！")
    print("欢迎学习 Python！")
```

如果函数暂时没有实现，可以用 `pass` 占位：

```python
def todo_function():
    pass  # 占位符，表示函数体暂未实现
```

> `pass` 是 Python 中的"空操作"语句，什么也不做，但语法上满足了"函数体不能为空"的要求。也可以使用 `...`（Ellipsis 字面量）代替 `pass`——两者在此场景下效果相同，`...` 在类型桩文件（`.pyi`）中更为常见。

### 五、函数的调用

**定义函数并不会执行函数体中的代码**。你必须**调用**函数，才能让它执行。

```python
# 定义函数
def say_hello():
    print("Hello!")

# 调用函数
say_hello()
```

调用语法：`函数名(参数)`

```python
def greet(name):
    print(f"你好，{name}！")

greet("小明")   # 输出: 你好，小明！
greet("小红")   # 输出: 你好，小红！
```

### 六、定义与调用的顺序

**在 Python 中，函数必须先定义，再调用。**

```python
# ✅ 正确：先定义，后调用
def say_hello():
    print("Hello!")

say_hello()
```

```python
# ❌ 错误：先调用，后定义
say_hello()  # NameError: name 'say_hello' is not defined

def say_hello():
    print("Hello!")
```

> **原理**：Python 是**逐行解释执行**的语言。当执行到 `say_hello()` 时，此时 `def say_hello()` 还没被执行，所以 `say_hello` 这个名字还不存在于当前命名空间中。

但有一个常见的误解需要澄清：

```python
# ✅ 这是合法的！
def a():
    b()   # 在 a 的定义中引用了 b，但此时 b 还没定义
          # 这没关系，因为 b() 不会立即执行

def b():
    print("我是 b")

a()  # 调用 a 时，b 已经被定义了，所以 b() 能正常执行
```

关键区别在于：`def a()` 这一行只是**定义了函数 a**，函数体中的 `b()` 此时**并不会执行**。只有当 `a()` 被实际调用时，Python 才会去执行 `b()`，而此时 `b` 已经被定义，所以不会报错。

### 七、一个完整示例

让我们回到开头的问题，用函数来重构代码：

```python
def sum_range(n):
    """计算从 1 到 n 的累加和"""
    total = 0
    for i in range(1, n + 1):
        total += i
    print(f"1到{n}的和为: {total}")

# 现在只需要调用函数即可
sum_range(100)   # 输出: 1到100的和为: 5050
sum_range(50)    # 输出: 1到50的和为: 1275
sum_range(200)   # 输出: 1到200的和为: 20100
```

代码从 15 行缩减到了 6 行（不算空行），而且逻辑更加清晰。

### 八、本讲小结

| 要点       | 说明                                |
| ---------- | ----------------------------------- |
| 函数是什么 | 组织好的、可重复使用的代码块        |
| 定义语法   | `def 函数名(参数列表):`             |
| 命名规范   | 使用 snake_case，不要覆盖内置函数名 |
| 调用语法   | `函数名(参数)`                      |
| 定义顺序   | 调用之前必须已定义                  |
| 空函数     | 可用 `pass` 或 `...` 占位           |

---

## 第59讲：函数基础 —— 函数参数与返回值

### 一、函数的参数（Parameter vs Argument）

在深入学习之前，我们先厘清两个经常被混用的术语：

| 术语                | 英文      | 含义                   | 出现位置     |
| ------------------- | --------- | ---------------------- | ------------ |
| **形参** (形式参数) | Parameter | 函数定义时声明的变量名 | `def` 语句中 |
| **实参** (实际参数) | Argument  | 函数调用时传入的具体值 | 调用语句中   |

```python
def greet(name):     # name 是形参（Parameter）
    print(f"你好，{name}！")

greet("小明")         # "小明" 是实参（Argument）
```

> 简单记忆：**形参是"坑位"，实参是"填坑的具体东西"**。

### 二、参数的基本使用

#### 2.1 无参数

```python
def print_line():
    print("-" * 40)

print_line()  # 输出: ----------------------------------------
```

#### 2.2 单个参数

```python
def print_line(char):
    print(char * 40)

print_line("-")  # ----------------------------------------
print_line("=")  # ========================================
print_line("*")  # ****************************************
```

#### 2.3 多个参数

```python
def print_line(char, length):
    print(char * length)

print_line("-", 20)  # --------------------
print_line("=", 30)  # ==============================
```

多个参数之间用**逗号 `,`** 分隔。调用时，实参的顺序必须与形参的顺序严格对应（这叫做**位置参数**，后面会详细讲）。

### 三、返回值（Return Value）

#### 3.1 什么是返回值？

之前我们的函数只是执行操作（如 `print()`），但没有把结果"传"回来。在实际开发中，函数更常见的用途是**计算出一个结果并返回给调用者**。

```python
# 没有返回值——只打印，不传回
def add_v1(a, b):
    print(a + b)

# 有返回值——把结果传回给调用者
def add_v2(a, b):
    return a + b
```

两者的核心区别：

```python
result1 = add_v1(3, 5)   # 打印 8，但 result1 是 None
result2 = add_v2(3, 5)   # 不打印任何东西，但 result2 等于 8

print(result1)  # None
print(result2)  # 8
```

> **`return` 关键字的作用：**
> 1. 将 `return` 后面的值作为函数的"执行结果"传递给调用者。
> 2. **立即结束**函数的执行（`return` 之后的代码不会被执行）。

#### 3.2 `return` 的立即终止特性

```python
def check_age(age):
    if age < 0:
        return "年龄不能为负数"   # 遇到 return，函数立即结束
    if age < 18:
        return "未成年"
    return "成年"               # 如果前面都没 return，才执行到这里

print(check_age(-1))   # 年龄不能为负数
print(check_age(10))   # 未成年
print(check_age(25))   # 成年
```

#### 3.3 没有 `return` 的函数

如果函数没有 `return` 语句（或者 `return` 后面没有值），Python 会**隐式返回 `None`**。

```python
def say_hello():
    print("Hello!")

result = say_hello()  # 打印 Hello!
print(result)         # None
print(type(result))   # <class 'NoneType'>
```

```python
def say_hello():
    print("Hello!")
    return           # 显式 return，但不带值，效果等同于 return None

result = say_hello()
print(result)  # None
```

#### 3.4 返回多个值

Python 的函数可以一次返回多个值！这在很多其他语言中需要借助结构体或对象，而 Python 直接利用**元组（tuple）**实现。

```python
def divide(a, b):
    """返回商和余数"""
    quotient = a // b
    remainder = a % b
    return quotient, remainder   # 实际上是返回一个元组 (quotient, remainder)

# 方式1：用一个变量接收，得到一个元组
result = divide(17, 5)
print(result)        # (3, 2)
print(type(result))  # <class 'tuple'>

# 方式2：用多个变量接收（解包赋值），最常用
q, r = divide(17, 5)
print(f"商: {q}, 余数: {r}")  # 商: 3, 余数: 2
```

> **重点理解**：`return a, b` 等价于 `return (a, b)`，Python 会自动将逗号分隔的值打包成一个元组。

更复杂的例子：

```python
def get_min_max_avg(numbers):
    """返回列表的最小值、最大值和平均值"""
    min_val = min(numbers)
    max_val = max(numbers)
    avg_val = sum(numbers) / len(numbers)
    return min_val, max_val, avg_val

data = [23, 45, 12, 67, 34, 89, 56]
minimum, maximum, average = get_min_max_avg(data)
print(f"最小值: {minimum}")   # 最小值: 12
print(f"最大值: {maximum}")   # 最大值: 89
print(f"平均值: {average:.2f}")  # 平均值: 46.57
```

#### 3.5 返回值可以是任意类型

```python
def create_user(name, age):
    """返回一个字典"""
    return {"name": name, "age": age}

user = create_user("小明", 20)
print(user)  # {'name': '小明', 'age': 20}

def get_even_numbers(n):
    """返回一个列表"""
    return [i for i in range(1, n + 1) if i % 2 == 0]

evens = get_even_numbers(10)
print(evens)  # [2, 4, 6, 8, 10]

def is_adult(age):
    """返回一个布尔值"""
    return age >= 18

print(is_adult(20))  # True
print(is_adult(15))  # False
```

### 四、参数与返回值的组合

根据有无参数和有无返回值，函数可以分为四种组合：

| 类型         | 有参数？ | 有返回值？ | 使用场景                           |
| ------------ | -------- | ---------- | ---------------------------------- |
| 无参无返回值 | ❌        | ❌          | 执行固定操作，如打印分隔线         |
| 有参无返回值 | ✅        | ❌          | 根据输入执行操作，如打印个性化问候 |
| 无参有返回值 | ❌        | ✅          | 获取状态信息，如获取当前时间       |
| 有参有返回值 | ✅        | ✅          | 最常用，如计算两数之和             |

```python
# 类型1：无参无返回值
def print_separator():
    print("=" * 40)

# 类型2：有参无返回值
def greet(name):
    print(f"你好，{name}！")

# 类型3：无参有返回值
import time
def get_current_time():
    return time.strftime("%Y-%m-%d %H:%M:%S")

# 类型4：有参有返回值（最常用）
def calculate_circle_area(radius):
    import math
    return math.pi * radius ** 2
```

### 五、实践练习

**练习1**：编写一个函数，接收三个数，返回其中的最大值。

```python
def find_max(a, b, c):
    """返回三个数中的最大值"""
    max_val = a
    if b > max_val:
        max_val = b
    if c > max_val:
        max_val = c
    return max_val

# 测试
print(find_max(3, 7, 5))    # 7
print(find_max(10, 2, 8))   # 10
print(find_max(1, 1, 1))    # 1
```

**练习2**：编写一个函数，接收一个字符串，返回字符串中大写字母、小写字母和数字的个数。

```python
def count_chars(s):
    """统计字符串中大小写字母和数字的个数"""
    upper_count = 0
    lower_count = 0
    digit_count = 0
    for char in s:
        if char.isupper():
            upper_count += 1
        elif char.islower():
            lower_count += 1
        elif char.isdigit():
            digit_count += 1
    return upper_count, lower_count, digit_count

# 测试
u, l, d = count_chars("Hello World 123")
print(f"大写: {u}, 小写: {l}, 数字: {d}")
# 输出: 大写: 2, 小写: 8, 数字: 3
```

### 六、本讲小结

| 要点            | 说明                                       |
| --------------- | ------------------------------------------ |
| 形参 vs 实参    | 形参是定义时的变量名，实参是调用时的具体值 |
| `return` 的作用 | ①返回值给调用者 ②立即终止函数              |
| 无 `return`     | 隐式返回 `None`                            |
| 返回多个值      | 利用元组打包，调用方可解包接收             |
| 返回值类型      | 可以是任意 Python 对象                     |

---

## 第60讲：函数基础 —— 函数说明文档

### 一、为什么需要函数说明文档？

想象你在一个团队中，同事写了这样一个函数：

```python
def process(d, k, v, m=False):
    if m:
        if k in d:
            d[k].append(v)
        else:
            d[k] = [v]
    else:
        d[k] = v
    return d
```

你能看懂它在做什么吗？参数 `d`, `k`, `v`, `m` 分别是什么？返回什么？

现在看加了文档的版本：

```python
def process(data_dict, key, value, multi=False):
    """向字典中添加键值对。

    如果 multi 为 True，则将 value 追加到 key 对应的列表中
    （如果 key 不存在则创建新列表）。
    如果 multi 为 False，则直接覆盖 key 对应的值。

    Args:
        data_dict: 目标字典。
        key: 要设置的键。
        value: 要设置的值。
        multi: 是否启用多值模式，默认 False。

    Returns:
        修改后的字典。
    """
    if multi:
        if key in data_dict:
            data_dict[key].append(value)
        else:
            data_dict[key] = [value]
    else:
        data_dict[key] = value
    return data_dict
```

是不是清晰多了？这就是**函数说明文档（Docstring）** 的价值。

### 二、什么是 Docstring？

**Docstring（文档字符串）** 是紧跟在函数定义（`def` 行）之后的**第一条字符串字面量**，用三引号包裹。

```python
def my_function():
    """这就是一个 docstring"""
    pass
```

Python 解释器会将这个字符串**自动赋值给该函数的 `__doc__` 属性**。

```python
def my_function():
    """这是我的函数"""
    pass

print(my_function.__doc__)  # 输出: 这是我的函数
```

### 三、Docstring 的基本格式

#### 3.1 单行 Docstring

适用于非常简单的函数：

```python
def square(n):
    """返回 n 的平方。"""
    return n ** 2
```

规范：
- 使用三重双引号 `"""`（也可以用单引号 `'''`，但 PEP 257 推荐双引号）。
- 首字母大写（英文时），以句号结尾。
- 开头和结束的三引号**在同一行**。

#### 3.2 多行 Docstring

适用于较复杂的函数：

```python
def calculate_bmi(weight, height):
    """根据体重和身高计算 BMI 指数。

    BMI（Body Mass Index）是国际通用的衡量人体胖瘦程度的标准。
    计算公式为：体重(kg) / 身高(m)的平方。

    Args:
        weight: 体重，单位为千克(kg)。
        height: 身高，单位为米(m)。

    Returns:
        float: BMI 指数值。

    Raises:
        ValueError: 如果身高或体重为非正数。
    """
    if weight <= 0 or height <= 0:
        raise ValueError("身高和体重必须为正数")
    return weight / (height ** 2)
```

多行 Docstring 的结构：

```
"""第一行：简短的功能概述（摘要行）。
                                         ← 一个空行
详细的描述段落，可以是多行。说明函数的行为、
算法、注意事项等。

Args:                                    ← 参数说明段
    参数1: 说明。
    参数2: 说明。

Returns:                                 ← 返回值说明段
    返回值的描述。

Raises:                                  ← 可能抛出的异常
    异常类型: 说明。
"""
```

### 四、主流 Docstring 风格

Python 社区有三种主流的 Docstring 书写风格：

#### 4.1 Google 风格（推荐初学者使用）

```python
def fetch_data(url, timeout=30):
    """从指定 URL 获取数据。

    发送 HTTP GET 请求并返回响应内容。
    支持自定义超时时间。

    Args:
        url: 目标 URL 地址。
        timeout: 请求超时时间（秒），默认 30 秒。

    Returns:
        str: 响应的文本内容。

    Raises:
        ConnectionError: 无法连接到目标服务器。
        TimeoutError: 请求超时。
    """
    pass
```

#### 4.2 NumPy/SciPy 风格

```python
def fetch_data(url, timeout=30):
    """从指定 URL 获取数据。

    发送 HTTP GET 请求并返回响应内容。

    Parameters
    ----------
    url : str
        目标 URL 地址。
    timeout : int, optional
        请求超时时间（秒），默认 30 秒。

    Returns
    -------
    str
        响应的文本内容。

    Raises
    ------
    ConnectionError
        无法连接到目标服务器。
    """
    pass
```

#### 4.3 reStructuredText 风格（Sphinx 默认）

```python
def fetch_data(url, timeout=30):
    """从指定 URL 获取数据。

    发送 HTTP GET 请求并返回响应内容。

    :param url: 目标 URL 地址。
    :type url: str
    :param timeout: 请求超时时间（秒），默认 30 秒。
    :type timeout: int
    :returns: 响应的文本内容。
    :rtype: str
    :raises ConnectionError: 无法连接到目标服务器。
    """
    pass
```

> **建议**：初学者选择 **Google 风格**，它简洁直观。科学计算领域更常用 NumPy 风格。大型项目如果使用 Sphinx 文档工具，可能偏好 reStructuredText 风格。

### 五、如何查看 Docstring

#### 5.1 使用 `help()` 函数

```python
def add(a, b):
    """返回两个数的和。"""
    return a + b

help(add)
```

输出：
```
Help on function add in module __main__:

add(a, b)
    返回两个数的和。
```

`help()` 不仅能查看自定义函数的文档，也能查看内置函数和标准库函数的文档：

```python
help(print)
help(len)
help(list.append)
```

#### 5.2 使用 `__doc__` 属性

```python
print(add.__doc__)  # 返回两个数的和。
```

#### 5.3 在 IDE 中查看

在 PyCharm、VS Code 等 IDE 中，将鼠标悬停在函数名上（或按 `Ctrl+Q` / `Ctrl+Shift+Space`），IDE 会自动显示该函数的 Docstring。这是编写 Docstring 最直接的回报——**你和你的团队成员在编码时会受益匪浅**。

### 六、Docstring 编写的最佳实践

1. **每个公共函数都应该写 Docstring**。
2. **摘要行应该是一句完整的话**，简明扼要地描述函数做什么。
3. **描述"做什么"，而不是"怎么做"**。用户关心的是接口，不是实现细节。
4. **记录所有参数和返回值**。
5. **记录可能抛出的异常**。
6. **在一个项目中保持风格统一**。

### 七、本讲小结

| 要点             | 说明                                           |
| ---------------- | ---------------------------------------------- |
| Docstring 是什么 | 函数定义后的第一条字符串，用于函数的说明文档   |
| 访问方式         | `help(函数名)` 或 `函数名.__doc__`             |
| 主流风格         | Google 风格、NumPy 风格、reStructuredText 风格 |
| 最佳实践         | 每个公共函数都应有 Docstring，统一风格         |

---

## 第61讲：函数基础 —— 函数嵌套调用

### 一、什么是函数嵌套调用？

**函数嵌套调用**是指：在一个函数的内部，调用了另一个函数。

```python
def func_a():
    print("A 开始")
    func_b()          # 在 func_a 中调用 func_b
    print("A 结束")

def func_b():
    print("  B 开始")
    print("  B 结束")

func_a()
```

输出：
```
A 开始
  B 开始
  B 结束
A 结束
```

### 二、执行流程详解

让我们用一个更详细的例子来理解执行流程：

```python
def step1():
    print("1. 准备食材")

def step2():
    print("2. 开始烹饪")
    step2_1()           # 嵌套调用
    step2_2()           # 嵌套调用
    print("2. 烹饪完成")

def step2_1():
    print("   2.1 切菜")

def step2_2():
    print("   2.2 炒菜")

def step3():
    print("3. 装盘上菜")

def cook():
    """烹饪的完整流程"""
    step1()
    step2()
    step3()

cook()
```

输出：
```
1. 准备食材
2. 开始烹饪
   2.1 切菜
   2.2 炒菜
2. 烹饪完成
3. 装盘上菜
```

**执行流程图（调用栈）**：

```
cook()
├── step1() → 打印 "1. 准备食材" → 返回
├── step2() 
│   ├── 打印 "2. 开始烹饪"
│   ├── step2_1() → 打印 "   2.1 切菜" → 返回
│   ├── step2_2() → 打印 "   2.2 炒菜" → 返回
│   └── 打印 "2. 烹饪完成" → 返回
└── step3() → 打印 "3. 装盘上菜" → 返回
```

> **核心原则**：当一个函数调用另一个函数时，当前函数**暂停执行**，等待被调用的函数执行完毕并返回后，才继续执行当前函数的下一条语句。

### 三、嵌套调用中的返回值传递

嵌套调用中最常见也最重要的模式是**使用一个函数的返回值作为另一个函数的输入**。

```python
def add(a, b):
    return a + b

def square(n):
    return n ** 2

# 嵌套调用：先计算 add(3, 4) = 7，然后计算 square(7) = 49
result = square(add(3, 4))
print(result)  # 49
```

执行顺序：
1. Python 要调用 `square(add(3, 4))`，但需要先知道参数的值。
2. 计算 `add(3, 4)` → 返回 `7`。
3. 现在调用 `square(7)` → 返回 `49`。
4. `result = 49`。

### 四、实践：用函数嵌套构建复杂功能

**案例：打印一个带边框的问候语**

```python
def print_border(width, char="*"):
    """打印一行边框"""
    print(char * width)

def print_centered(text, width):
    """打印居中文本（两侧带 * 号）"""
    content = text.center(width - 2)  # 减去两侧的 * 号
    print(f"*{content}*")

def print_greeting_card(name):
    """打印一个问候卡片"""
    greeting = f"你好, {name}!"
    width = len(greeting) + 8   # 两侧各留 4 个字符的边距

    # 利用嵌套调用组合功能
    print_border(width)
    print_centered("", width)               # 空行
    print_centered(greeting, width)          # 问候语
    print_centered("欢迎学习 Python!", width) # 欢迎语
    print_centered("", width)               # 空行
    print_border(width)

print_greeting_card("小明")
```

输出（大致效果）：
```
********************
*                  *
*   你好, 小明!    *
* 欢迎学习 Python! *
*                  *
********************
```

在这个例子中，`print_greeting_card` 函数内部嵌套调用了 `print_border` 和 `print_centered` 两个函数，每个函数各司其职，组合起来完成了一个复杂的输出任务。

### 五、函数嵌套调用的好处

1. **分解复杂问题**：将大任务拆成多个小任务，每个函数只做一件事。
2. **代码复用**：小函数可以被不同的大函数复用。
3. **易于测试**：可以单独测试每个小函数。
4. **提高可读性**：函数名本身就是一种注释。

### 六、本讲小结

| 要点       | 说明                                       |
| ---------- | ------------------------------------------ |
| 嵌套调用   | 在一个函数内部调用另一个函数               |
| 执行流程   | 当前函数暂停 → 被调用函数执行 → 返回后继续 |
| 返回值传递 | 一个函数的返回值可作为另一个函数的参数     |
| 设计原则   | 每个函数只做一件事，通过嵌套调用组合功能   |

---

## 第62讲：函数基础 —— 综合案例

### 案例 1：银行账户余额计算器

```python
def create_account(owner, balance=0):
    """创建一个银行账户信息"""
    return {"owner": owner, "balance": balance}

def deposit(account, amount):
    """存款"""
    if amount <= 0:
        print("存款金额必须为正数！")
        return account["balance"]
    account["balance"] += amount
    print(f"成功存入 {amount} 元，当前余额: {account['balance']} 元")
    return account["balance"]

def withdraw(account, amount):
    """取款"""
    if amount <= 0:
        print("取款金额必须为正数！")
        return account["balance"]
    if amount > account["balance"]:
        print(f"余额不足！当前余额: {account['balance']} 元，取款金额: {amount} 元")
        return account["balance"]
    account["balance"] -= amount
    print(f"成功取出 {amount} 元，当前余额: {account['balance']} 元")
    return account["balance"]

def check_balance(account):
    """查询余额"""
    print(f"账户 [{account['owner']}] 当前余额: {account['balance']} 元")
    return account["balance"]

def print_separator():
    """打印分隔线"""
    print("-" * 40)

# ========== 主程序 ==========
my_account = create_account("小明", 1000)

print_separator()
check_balance(my_account)

print_separator()
deposit(my_account, 500)

print_separator()
withdraw(my_account, 300)

print_separator()
withdraw(my_account, 2000)   # 余额不足

print_separator()
check_balance(my_account)
```

输出：
```
----------------------------------------
账户 [小明] 当前余额: 1000 元
----------------------------------------
成功存入 500 元，当前余额: 1500 元
----------------------------------------
成功取出 300 元，当前余额: 1200 元
----------------------------------------
余额不足！当前余额: 1200 元，取款金额: 2000 元
----------------------------------------
账户 [小明] 当前余额: 1200 元
```

**案例要点**：
- 每个函数只负责**一个**操作：创建、存款、取款、查询。
- 函数之间通过**参数**和**返回值**传递数据。
- 使用**数据验证**确保输入合法。

### 案例 2：成绩管理系统

```python
def input_scores():
    """输入多个学生成绩，返回成绩列表"""
    scores = []
    print("请输入学生成绩（输入 -1 结束）：")
    while True:
        raw = input("成绩: ").strip()
        try:
            score = float(raw)
        except ValueError:
            print("输入无效，请输入数字（如 88、76.5）或 -1 结束！")
            continue

        if score == -1:
            break
        if 0 <= score <= 100:
            scores.append(score)
        else:
            print("成绩无效，请输入 0 ~ 100 之间的数！")
    return scores

def calculate_stats(scores):
    """计算成绩的统计信息"""
    if not scores:
        return 0, 0, 0, 0
    total = sum(scores)
    average = total / len(scores)
    highest = max(scores)
    lowest = min(scores)
    return total, average, highest, lowest

def count_grade_distribution(scores):
    """统计各等级的人数"""
    excellent = 0   # 90-100
    good = 0        # 80-89
    medium = 0      # 70-79
    passing = 0     # 60-69
    failing = 0     # 0-59

    for score in scores:
        if score >= 90:
            excellent += 1
        elif score >= 80:
            good += 1
        elif score >= 70:
            medium += 1
        elif score >= 60:
            passing += 1
        else:
            failing += 1

    return excellent, good, medium, passing, failing

def display_report(scores, stats, grade_dist):
    """显示成绩报告"""
    total, avg, highest, lowest = stats
    excellent, good, medium, passing, failing = grade_dist

    print("\n" + "=" * 40)
    print("         成 绩 报 告")
    print("=" * 40)
    print(f"学生人数: {len(scores)}")
    print(f"总    分: {total}")
    print(f"平 均 分: {avg:.2f}")
    print(f"最 高 分: {highest}")
    print(f"最 低 分: {lowest}")
    print("-" * 40)
    print("等级分布:")
    print(f"  优秀 (90-100): {excellent} 人")
    print(f"  良好 (80-89) : {good} 人")
    print(f"  中等 (70-79) : {medium} 人")
    print(f"  及格 (60-69) : {passing} 人")
    print(f"  不及格 (0-59): {failing} 人")
    print("=" * 40)

def main():
    """主函数：串联所有功能"""
    scores = input_scores()

    if not scores:
        print("没有输入任何成绩！")
        return

    stats = calculate_stats(scores)
    grade_dist = count_grade_distribution(scores)
    display_report(scores, stats, grade_dist)

# 运行主程序
if __name__ == "__main__":
    main()
```

**案例要点**：
- 使用 `main()` 函数作为程序的入口点（Python 的常见惯例）。
- 函数之间形成清晰的**流水线**：输入 → 计算 → 输出。
- 每个函数职责单一，便于**独立测试和修改**。

### 案例 3：温度转换工具

```python
def celsius_to_fahrenheit(celsius):
    """摄氏度转华氏度：F = C × 9/5 + 32"""
    return celsius * 9 / 5 + 32

def fahrenheit_to_celsius(fahrenheit):
    """华氏度转摄氏度：C = (F - 32) × 5/9"""
    return (fahrenheit - 32) * 5 / 9

def celsius_to_kelvin(celsius):
    """摄氏度转开尔文：K = C + 273.15"""
    return celsius + 273.15

def kelvin_to_celsius(kelvin):
    """开尔文转摄氏度：C = K - 273.15"""
    return kelvin - 273.15

def format_temperature(value, unit):
    """格式化温度输出"""
    return f"{value:.2f}°{unit}"

def convert_temperature(value, from_unit, to_unit):
    """通用温度转换函数（嵌套调用）"""
    # 第一步：统一转成摄氏度
    if from_unit == "C":
        celsius = value
    elif from_unit == "F":
        celsius = fahrenheit_to_celsius(value)
    elif from_unit == "K":
        celsius = kelvin_to_celsius(value)
    else:
        return "未知的温度单位"

    # 第二步：从摄氏度转成目标单位
    if to_unit == "C":
        result = celsius
    elif to_unit == "F":
        result = celsius_to_fahrenheit(celsius)
    elif to_unit == "K":
        result = celsius_to_kelvin(celsius)
    else:
        return "未知的温度单位"

    return f"{format_temperature(value, from_unit)} = {format_temperature(result, to_unit)}"

# 测试
print(convert_temperature(100, "C", "F"))   # 100.00°C = 212.00°F
print(convert_temperature(72, "F", "C"))    # 72.00°F = 22.22°C
print(convert_temperature(300, "K", "C"))   # 300.00°K = 26.85°C
print(convert_temperature(0, "C", "K"))     # 0.00°C = 273.15°K
```

**案例要点**：
- `convert_temperature` 利用"中间量转换"策略——先转成摄氏度，再转成目标单位。
- 通过嵌套调用多个小函数来实现复杂逻辑，代码简洁且易扩展。

---

## 第63讲：函数进阶 —— 变量作用域

### 一、什么是作用域？

**作用域（Scope）** 是指变量在程序中**可被访问的范围**。

简单来说：不是所有变量在程序的任何地方都能使用，变量只在它被创建的"区域"内有效。

### 二、Python 的四种作用域（LEGB 规则）

Python 中有四种作用域，Python 查找变量时按照 **L → E → G → B** 的顺序逐层查找：

| 作用域           | 英文      | 含义                         | 示例                     |
| ---------------- | --------- | ---------------------------- | ------------------------ |
| **L** - 局部     | Local     | 函数内部定义的变量           | 函数内的变量             |
| **E** - 外层函数 | Enclosing | 外层（嵌套）函数的局部作用域 | 闭包中外层函数的变量     |
| **G** - 全局     | Global    | 模块（文件）级别的变量       | 写在所有函数外面的变量   |
| **B** - 内置     | Built-in  | Python 解释器内置的名称      | `print`, `len`, `int` 等 |

> 当你使用一个变量名时，Python 会按照 **L → E → G → B** 的顺序逐层查找，找到第一个匹配的就使用，如果都找不到就抛出 `NameError`。

### 三、局部变量（Local Variable）

**在函数内部定义的变量**就是局部变量。局部变量**只在该函数内部有效**。

```python
def my_function():
    x = 10          # x 是局部变量
    print(f"函数内部: x = {x}")

my_function()       # 输出: 函数内部: x = 10
# print(x)          # ❌ NameError: name 'x' is not defined
```

> **关键**：函数执行结束后，局部变量就被销毁了，在函数外部无法访问。

#### 局部变量的生命周期

```python
def demo():
    a = 100          # a 被创建
    print(a)         # a 被使用
    # 函数结束，a 被销毁

demo()
demo()   # 再次调用时，a 是一个全新的变量，与上次无关
```

每次调用函数时，局部变量都会**重新创建**，函数返回后**自动销毁**。这意味着多次调用同一个函数，局部变量之间**互不影响**。

### 四、全局变量（Global Variable）

**在函数外部定义的变量**就是全局变量。全局变量在整个模块（文件）中都可以访问。

```python
count = 0   # 全局变量

def show_count():
    print(f"count = {count}")   # 函数内可以读取全局变量

show_count()   # 输出: count = 0
print(count)   # 也可以在函数外直接使用
```

#### 函数内部修改全局变量

```python
count = 0

def increment():
    count = count + 1   # ❌ UnboundLocalError!
    print(count)

increment()
```

**为什么会报错？**

当 Python 编译函数体时，发现 `count = count + 1` 中有一个 **对 `count` 的赋值操作** (`count = ...`)。Python 因此认为 `count` 是一个**局部变量**。但在执行 `count + 1` 时，这个局部变量 `count` 还没有被赋值（它正在被赋值的过程中），所以引发了 `UnboundLocalError`。

> **规则总结**：
> - 函数内**读取**全局变量：直接使用即可。
> - 函数内**修改**全局变量：必须使用 `global` 关键字声明。

#### `global` 关键字

```python
count = 0

def increment():
    global count      # 声明：我要使用的是全局变量 count，而非创建局部变量
    count = count + 1
    print(f"函数内: count = {count}")

increment()   # 函数内: count = 1
increment()   # 函数内: count = 2
increment()   # 函数内: count = 3
print(f"函数外: count = {count}")  # 函数外: count = 3
```

#### `global` 的注意事项

```python
# 可以用 global 在函数中创建全局变量
def create_global():
    global new_var
    new_var = "我是在函数中创建的全局变量"

create_global()
print(new_var)  # 我是在函数中创建的全局变量
```

> **⚠️ 工程建议**：**尽量避免使用 `global`！** 全局变量使得函数的行为依赖于外部状态，增加了调试难度和出错风险。推荐通过参数传入、通过返回值传出数据。

### 五、Enclosing 作用域与 `nonlocal` 关键字

当你有**嵌套函数**（函数中定义函数）时，内层函数可以访问外层函数的局部变量，这就是 **Enclosing（闭包）作用域**。

```python
def outer():
    x = 10        # outer 的局部变量

    def inner():
        print(x)  # inner 可以读取 outer 的 x（Enclosing 作用域）

    inner()

outer()  # 输出: 10
```

如果要在内层函数中**修改**外层函数的变量，需要使用 `nonlocal` 关键字：

```python
def outer():
    count = 0

    def inner():
        nonlocal count   # 声明：我要修改的是 outer 的 count
        count += 1
        print(f"inner: count = {count}")

    inner()   # inner: count = 1
    inner()   # inner: count = 2
    print(f"outer: count = {count}")  # outer: count = 2

outer()
```

> `nonlocal` 与 `global` 的区别：
> - `global`：声明变量属于**模块级**（最外层）全局作用域。
> - `nonlocal`：声明变量属于**外层函数**的局部作用域（但不是全局的）。

### 六、LEGB 规则综合示例

```python
x = "global"          # Global 作用域

def outer():
    x = "enclosing"   # Enclosing 作用域

    def inner():
        x = "local"   # Local 作用域
        print(f"inner 中: x = {x}")

    inner()
    print(f"outer 中: x = {x}")

outer()
print(f"全局中: x = {x}")
```

输出：
```
inner 中: x = local
outer 中: x = enclosing
全局中: x = global
```

每一层都有自己的 `x`，互不干扰。如果把某一层的 `x = ...` 赋值去掉，它就会去上一层查找。

```python
x = "global"

def outer():
    # 这里没有定义 x

    def inner():
        # 这里也没有定义 x
        print(f"inner 中: x = {x}")  # L 没有 → E 没有 → G 找到了

    inner()

outer()  # inner 中: x = global
```

### 七、变量遮蔽（Shadowing）

当局部变量与全局变量同名时，局部变量会**遮蔽（shadow）**全局变量：

```python
name = "全局小明"

def greet():
    name = "局部小红"   # 遮蔽了全局的 name
    print(f"函数内: {name}")

greet()            # 函数内: 局部小红
print(f"函数外: {name}")  # 函数外: 全局小明（全局的 name 不受影响）
```

### 八、本讲小结

| 要点       | 说明                                                |
| ---------- | --------------------------------------------------- |
| LEGB 规则  | 变量查找顺序：Local → Enclosing → Global → Built-in |
| 局部变量   | 函数内定义，函数结束即销毁                          |
| 全局变量   | 函数外定义，整个模块可见                            |
| `global`   | 在函数内声明使用/修改全局变量                       |
| `nonlocal` | 在内层函数中声明使用/修改外层函数的变量             |
| 变量遮蔽   | 局部变量会遮蔽同名的外层/全局变量                   |
| 工程建议   | 尽量少用 `global`，通过参数和返回值传递数据         |

---

## 第64讲：函数进阶 —— 传参方式

### 一、位置参数（Positional Arguments）

最基本的传参方式——**按位置顺序一一匹配**。

```python
def introduce(name, age, city):
    print(f"我叫{name}，{age}岁，来自{city}")

introduce("小明", 20, "北京")  # 按位置：name="小明", age=20, city="北京"
introduce("小红", 18, "上海")  # 按位置：name="小红", age=18, city="上海"
```

**注意**：实参的数量必须与形参一致，否则报错。

```python
introduce("小明", 20)          # ❌ TypeError: 缺少参数 'city'
introduce("小明", 20, "北京", "学生")  # ❌ TypeError: 多了一个参数
```

### 二、关键字参数（Keyword Arguments）

通过 `参数名=值` 的形式传参，可以**不按顺序**。

```python
def introduce(name, age, city):
    print(f"我叫{name}，{age}岁，来自{city}")

# 使用关键字参数，顺序可以随意
introduce(age=20, city="北京", name="小明")
# 输出: 我叫小明，20岁，来自北京
```

#### 位置参数与关键字参数混用

可以混用，但**位置参数必须在前**：

```python
introduce("小明", city="北京", age=20)  # ✅ 合法

# introduce(name="小明", 20, "北京")    # ❌ SyntaxError: 位置参数在关键字参数后面
```

> **规则：位置参数必须出现在关键字参数之前。**

### 三、Python 的参数传递机制（重要！）

Python 的参数传递本质是**对象引用的传递**（有时称为"传对象引用"或"传共享"）。

理解这个问题的关键在于分清 Python 对象的**可变性**：

| 不可变类型（Immutable） | 可变类型（Mutable） |
| ----------------------- | ------------------- |
| `int`, `float`, `bool`  | `list`              |
| `str`, `tuple`          | `dict`              |
| `frozenset`             | `set`               |

#### 3.1 传入不可变类型

```python
def modify_number(n):
    print(f"修改前: n = {n}, id = {id(n)}")
    n = n + 1                   # n 被重新绑定到新对象
    print(f"修改后: n = {n}, id = {id(n)}")

x = 10
print(f"调用前: x = {x}, id = {id(x)}")
modify_number(x)
print(f"调用后: x = {x}, id = {id(x)}")  # x 不变！
```

输出：
```
调用前: x = 10, id = 140...160
修改前: n = 10, id = 140...160    ← n 和 x 指向同一个对象
修改后: n = 11, id = 140...192    ← n 指向了新对象，x 不受影响
调用后: x = 10, id = 140...160    ← x 仍然是 10
```

**原理解析**：整数是不可变对象。`n = n + 1` 不是"修改了 n 指向的对象"，而是"让 n 指向一个新的整数对象 11"。外部的 `x` 仍然指向原来的 `10`。

#### 3.2 传入可变类型

```python
def modify_list(lst):
    print(f"修改前: lst = {lst}, id = {id(lst)}")
    lst.append(4)               # 在原对象上修改（原地操作）
    print(f"修改后: lst = {lst}, id = {id(lst)}")

my_list = [1, 2, 3]
print(f"调用前: my_list = {my_list}, id = {id(my_list)}")
modify_list(my_list)
print(f"调用后: my_list = {my_list}, id = {id(my_list)}")
```

输出：
```
调用前: my_list = [1, 2, 3], id = 2000...
修改前: lst = [1, 2, 3], id = 2000...    ← lst 和 my_list 指向同一个列表对象
修改后: lst = [1, 2, 3, 4], id = 2000... ← 在原对象上追加，id 没变
调用后: my_list = [1, 2, 3, 4], id = 2000... ← 外部也看到了变化！
```

**原理**：列表是可变对象，`lst.append(4)` 是在**原对象上**修改。`lst` 和 `my_list` 指向同一个对象，所以外部也能看到变化。

#### 3.3 可变类型的"重新绑定"陷阱

```python
def try_replace_list(lst):
    lst = [100, 200, 300]   # 重新绑定！lst 指向了一个全新的列表
    print(f"函数内: {lst}")

my_list = [1, 2, 3]
try_replace_list(my_list)
print(f"函数外: {my_list}")  # [1, 2, 3]  ← 没变！
```

> **核心总结**：
> - `lst.append()` / `lst[0] = ...` → **修改原对象** → 外部受影响
> - `lst = 新列表` → **重新绑定变量** → 外部不受影响

#### 3.4 如果不想让函数修改原列表怎么办？

传入列表的**副本**：

```python
def safe_modify(lst):
    lst.append(4)
    return lst

my_list = [1, 2, 3]
new_list = safe_modify(my_list.copy())   # 传入副本
print(my_list)   # [1, 2, 3]  ← 原列表未受影响
print(new_list)  # [1, 2, 3, 4]
```

### 四、本讲小结

| 要点       | 说明                                     |
| ---------- | ---------------------------------------- |
| 位置参数   | 按顺序一一对应                           |
| 关键字参数 | `参数名=值`，可不按顺序                  |
| 混用规则   | 位置参数必须在关键字参数前面             |
| 传参本质   | 传递的是对象引用（不是值的拷贝）         |
| 不可变类型 | 函数内修改不影响外部（重新绑定到新对象） |
| 可变类型   | 原地修改影响外部；重新赋值不影响外部     |

---

## 第65讲：函数进阶 —— 默认参数

### 一、什么是默认参数？

**默认参数**允许你在定义函数时为形参指定一个默认值。调用时如果不传该参数，就使用默认值。

```python
def greet(name, greeting="你好"):
    print(f"{greeting}，{name}！")

greet("小明")           # 使用默认值 → 你好，小明！
greet("小红", "早上好")  # 传入自定义值 → 早上好，小红！
```

### 二、语法规则

**默认参数必须在非默认参数后面**：

```python
# ✅ 正确
def func(a, b, c=10):
    pass

def func(a, b=5, c=10):
    pass

# ❌ 错误：非默认参数跟在默认参数后面
# def func(a, b=5, c):  # SyntaxError
#     pass
```

### 三、实用示例

```python
def create_user(name, age, city="未知", role="普通用户"):
    return {
        "name": name,
        "age": age,
        "city": city,
        "role": role
    }

user1 = create_user("小明", 20)
print(user1)  # {'name': '小明', 'age': 20, 'city': '未知', 'role': '普通用户'}

user2 = create_user("小红", 25, city="北京")
print(user2)  # {'name': '小红', 'age': 25, 'city': '北京', 'role': '普通用户'}

user3 = create_user("管理员", 30, role="admin")
print(user3)  # {'name': '管理员', 'age': 30, 'city': '未知', 'role': 'admin'}
```

### 四、⚠️ 可变默认参数的经典陷阱

这是 Python 中**最著名的坑之一**，必须牢记！

```python
# ❌ 危险代码
def append_to(item, target=[]):
    target.append(item)
    return target

print(append_to(1))   # [1]        ← 看起来正常
print(append_to(2))   # [1, 2]     ← 等等？为什么不是 [2]？
print(append_to(3))   # [1, 2, 3]  ← 列表在不断增长！
```

**原因**：默认参数的值只在**函数定义时**计算**一次**。`target=[]` 这个空列表在 `def` 执行时就被创建了，之后每次调用如果不传 `target`，用的都是**同一个列表对象**。

**正确做法**：使用 `None` 作为哨兵值。

```python
# ✅ 正确做法
def append_to(item, target=None):
    if target is None:
        target = []        # 每次调用都创建新列表
    target.append(item)
    return target

print(append_to(1))   # [1]
print(append_to(2))   # [2]   ← 正确！每次都是新列表
print(append_to(3))   # [3]
```

> **铁律：永远不要使用可变对象（`list`、`dict`、`set`）作为默认参数值！用 `None` 代替。**

### 五、本讲小结

| 要点     | 说明                                   |
| -------- | -------------------------------------- |
| 默认参数 | `def func(a, b=默认值)`                |
| 位置规则 | 默认参数必须在非默认参数之后           |
| 经典陷阱 | 不要用可变对象做默认值，用 `None` 替代 |

---

## 第66讲：函数进阶 —— 不定长参数

### 一、为什么需要不定长参数？

有时候函数需要接收**不确定数量**的参数。例如内置函数 `print()` 可以接收任意多个值：

```python
print(1)
print(1, 2, 3)
print("a", "b", "c", "d", "e")
```

Python 提供了两种不定长参数：`*args` 和 `**kwargs`。

### 二、`*args` —— 接收任意数量的位置参数

**在形参前加一个 `*`**，该参数会把**多余的位置参数**收集为一个**元组（tuple）**。

```python
def add(*args):
    print(f"args = {args}")
    print(f"type = {type(args)}")
    return sum(args)

print(add(1, 2))           # args = (1, 2)        → 3
print(add(1, 2, 3, 4, 5))  # args = (1, 2, 3, 4, 5) → 15
print(add())               # args = ()             → 0
```

> `args` 只是一个约定俗成的名字，你可以用任何名字，关键是前面的 `*` 号。但强烈建议遵循约定使用 `args`。

#### 与普通参数混用

```python
def greet(greeting, *names):
    for name in names:
        print(f"{greeting}，{name}！")

greet("你好", "小明", "小红", "小刚")
# 你好，小明！
# 你好，小红！
# 你好，小刚！
```

> `*args` 之前的参数用位置匹配，剩余的所有位置参数收入 `args`。

### 三、`**kwargs` —— 接收任意数量的关键字参数

**在形参前加两个 `**`**，该参数会把**多余的关键字参数**收集为一个**字典（dict）**。

```python
def print_info(**kwargs):
    print(f"kwargs = {kwargs}")
    print(f"type = {type(kwargs)}")
    for key, value in kwargs.items():
        print(f"  {key}: {value}")

print_info(name="小明", age=20, city="北京")
```

输出：
```
kwargs = {'name': '小明', 'age': 20, 'city': '北京'}
type = <class 'dict'>
  name: 小明
  age: 20
  city: 北京
```

### 四、`*args` 和 `**kwargs` 混用

```python
def universal(*args, **kwargs):
    print(f"位置参数: {args}")
    print(f"关键字参数: {kwargs}")

universal(1, 2, 3, name="小明", age=20)
# 位置参数: (1, 2, 3)
# 关键字参数: {'name': '小明', 'age': 20}
```

**参数顺序规则**（从左到右）：

```python
def func(普通参数, *args, 默认参数, **kwargs):
    pass

# 完整示例
def example(a, b, *args, c=10, **kwargs):
    print(f"a={a}, b={b}")
    print(f"args={args}")
    print(f"c={c}")
    print(f"kwargs={kwargs}")

example(1, 2, 3, 4, c=20, x=100, y=200)
# a=1, b=2
# args=(3, 4)
# c=20
# kwargs={'x': 100, 'y': 200}
```

### 五、参数解包（Unpacking）

用 `*` 和 `**` 不仅能收集参数，还能**展开**序列/字典作为参数传入。

```python
def add(a, b, c):
    return a + b + c

# 用 * 解包列表/元组为位置参数
numbers = [1, 2, 3]
print(add(*numbers))   # 等价于 add(1, 2, 3) → 6

# 用 ** 解包字典为关键字参数
params = {"a": 10, "b": 20, "c": 30}
print(add(**params))   # 等价于 add(a=10, b=20, c=30) → 60
```

### 六、仅限关键字参数（Keyword-only Arguments）

放在 `*args` 之后（或单独的 `*` 之后）的参数，调用时**必须**使用关键字方式传入：

```python
def safe_divide(a, b, *, on_error=0):
    """on_error 必须用关键字传入"""
    if b == 0:
        return on_error
    return a / b

print(safe_divide(10, 3))                # 3.333...
print(safe_divide(10, 0))                # 0（默认值）
print(safe_divide(10, 0, on_error=-1))   # -1
# print(safe_divide(10, 0, -1))  # ❌ TypeError
```

### 七、仅限位置参数（Python 3.8+）

在参数列表中使用 `/`，`/` 之前的参数**只能按位置**传入：

```python
def greet(name, /, greeting="你好"):
    print(f"{greeting}，{name}！")

greet("小明")              # ✅
greet("小明", greeting="早上好")  # ✅
# greet(name="小明")       # ❌ TypeError: 'name' 是仅限位置参数
```

### 八、完整参数顺序总结

```python
def func(pos_only, /, normal, *, kw_only, **kwargs):
    pass

#        仅位置  /  普通   *  仅关键字  **kwargs
```

| 位置            | 说明                         |
| --------------- | ---------------------------- |
| `/` 之前        | 仅限位置参数                 |
| `/` 与 `*` 之间 | 普通参数（位置或关键字都行） |
| `*` 之后        | 仅限关键字参数               |
| `**kwargs`      | 收集所有额外的关键字参数     |

---

## 第67讲：函数进阶 —— 参数类型（函数作为参数）

### 一、Python 中函数是"一等公民"

在 Python 中，**函数本身也是一个对象**，和整数、字符串、列表没有本质区别。

```python
def say_hello():
    print("Hello!")

print(type(say_hello))   # <class 'function'>
print(id(say_hello))     # 一个内存地址

# 可以赋值给变量
greet = say_hello
greet()   # Hello!  ← 通过新变量名调用
```

### 二、把函数作为参数传入另一个函数

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def calculate(func, a, b):
    """接收一个函数和两个数，用该函数计算结果"""
    result = func(a, b)
    print(f"计算结果: {result}")
    return result

calculate(add, 10, 5)       # 计算结果: 15
calculate(subtract, 10, 5)  # 计算结果: 5
```

> 注意：传入的是 `add` 而不是 `add()`。加括号是**调用**函数，不加括号是**传递**函数对象本身。

### 三、实用案例：自定义排序

`sorted()` 函数的 `key` 参数就是"函数作为参数"的经典应用：

```python
students = [
    {"name": "小明", "score": 85},
    {"name": "小红", "score": 92},
    {"name": "小刚", "score": 78},
]

def get_score(student):
    return student["score"]

# 按成绩排序（key 接收一个函数）
sorted_students = sorted(students, key=get_score)
for s in sorted_students:
    print(f"{s['name']}: {s['score']}")
# 小刚: 78
# 小明: 85
# 小红: 92
```

### 四、实用案例：通用数据过滤器

```python
def is_even(n):
    return n % 2 == 0

def is_positive(n):
    return n > 0

def my_filter(func, data):
    """通用过滤器：保留 func 返回 True 的元素"""
    return [item for item in data if func(item)]

numbers = [-3, -1, 0, 1, 2, 4, 7, -5, 8]

print(my_filter(is_even, numbers))      # [0, 2, 4, 8]
print(my_filter(is_positive, numbers))  # [1, 2, 4, 7, 8]
```

> 这里 `my_filter` 的逻辑是固定的（遍历+判断），但**判断规则**由外部传入的函数决定。这就是**高阶函数**——接受函数作为参数或返回函数的函数。

---

## 第68讲：函数进阶 —— 匿名函数（Lambda 表达式）

### 一、什么是 Lambda？

**Lambda 表达式**是一种创建**小型匿名函数**的简写方式。

```python
# 普通函数
def add(a, b):
    return a + b

# 等价的 Lambda
add = lambda a, b: a + b

print(add(3, 5))  # 8
```

### 二、语法

```python
lambda 参数列表: 表达式
```

| 组成部分 | 说明                                       |
| -------- | ------------------------------------------ |
| `lambda` | 关键字，声明这是一个匿名函数               |
| 参数列表 | 和普通函数一样，多个参数用逗号分隔         |
| `:`      | 分隔参数和表达式                           |
| 表达式   | **只能是单个表达式**，其计算结果即为返回值 |

**重要限制**：Lambda 只能包含**一个表达式**，不能包含多条语句、赋值、循环等。

```python
# ✅ 合法
square = lambda x: x ** 2
maximum = lambda a, b: a if a > b else b

# ❌ 不合法（不能有多条语句）
# bad = lambda x: print(x); return x
```

### 三、Lambda 的典型使用场景

Lambda 最适合作为**高阶函数的参数**，避免为简单逻辑定义命名函数。

#### 3.1 配合 `sorted()` 使用

```python
students = [("小明", 85), ("小红", 92), ("小刚", 78)]

# 按成绩排序
result = sorted(students, key=lambda s: s[1])
print(result)  # [('小刚', 78), ('小明', 85), ('小红', 92)]

# 按名字排序
result = sorted(students, key=lambda s: s[0])
```

#### 3.2 配合 `map()` 使用

`map(func, iterable)` 将函数应用于可迭代对象的每个元素：

```python
numbers = [1, 2, 3, 4, 5]

squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# 等价的列表推导式（通常更推荐）
squared = [x ** 2 for x in numbers]
```

#### 3.3 配合 `filter()` 使用

`filter(func, iterable)` 过滤出函数返回 `True` 的元素：

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)   # [2, 4, 6, 8, 10]

# 等价的列表推导式
evens = [x for x in numbers if x % 2 == 0]
```

#### 3.4 配合 `max()` / `min()` 使用

```python
products = [
    {"name": "手机", "price": 3999},
    {"name": "电脑", "price": 7999},
    {"name": "耳机", "price": 299},
]

most_expensive = max(products, key=lambda p: p["price"])
print(most_expensive)  # {'name': '电脑', 'price': 7999}

cheapest = min(products, key=lambda p: p["price"])
print(cheapest)  # {'name': '耳机', 'price': 299}
```

### 四、Lambda vs 命名函数，何时用哪个？

| 场景                                       | 推荐       |
| ------------------------------------------ | ---------- |
| 逻辑简单，只用一次（如 `sorted` 的 `key`） | Lambda ✅   |
| 逻辑复杂，需要多行代码                     | 命名函数 ✅ |
| 需要在多处复用                             | 命名函数 ✅ |
| 需要写 Docstring 说明                      | 命名函数 ✅ |

> **PEP 8 建议**：不要用 `f = lambda x: ...` 把 lambda 赋值给变量——如果你需要一个有名字的函数，直接用 `def` 定义。Lambda 的价值在于**匿名性和内联使用**。

---

## 第69讲：函数进阶 —— 案例1（递归）

### 一、什么是递归？

**递归（Recursion）** 是指函数**在函数体中调用自身**。

```python
def countdown(n):
    if n <= 0:
        print("发射！")
        return           # 基准情况（Base Case）：终止递归
    print(n)
    countdown(n - 1)     # 递归调用（Recursive Case）

countdown(5)
```

输出：
```
5
4
3
2
1
发射！
```

### 二、递归的两个核心要素

| 要素                           | 说明                             | 缺少会怎样？                    |
| ------------------------------ | -------------------------------- | ------------------------------- |
| **基准情况**（Base Case）      | 终止递归的条件                   | 无限递归，最终 `RecursionError` |
| **递推关系**（Recursive Case） | 问题规模逐步缩小，向基准情况靠近 | 同上                            |

### 三、经典案例：阶乘

数学定义：`n! = n × (n-1) × ... × 2 × 1`，其中 `0! = 1`。

递推关系：`n! = n × (n-1)!`

```python
def factorial(n):
    """计算 n 的阶乘（递归版）"""
    # 基准情况
    if n == 0 or n == 1:
        return 1
    # 递推关系
    return n * factorial(n - 1)

print(factorial(5))   # 120
print(factorial(10))  # 3628800
```

**执行过程**（以 `factorial(5)` 为例）：

```
factorial(5)
= 5 * factorial(4)
= 5 * 4 * factorial(3)
= 5 * 4 * 3 * factorial(2)
= 5 * 4 * 3 * 2 * factorial(1)
= 5 * 4 * 3 * 2 * 1
= 120
```

### 四、经典案例：斐波那契数列

`F(0)=0, F(1)=1, F(n)=F(n-1)+F(n-2)`

```python
def fibonacci(n):
    """返回第 n 个斐波那契数"""
    if n <= 0:
        return 0
    if n == 1:
        return 1
    return fibonacci(n - 1) + fibonacci(n - 2)

for i in range(10):
    print(fibonacci(i), end=" ")
# 0 1 1 2 3 5 8 13 21 34
```

> **⚠️ 性能警告**：朴素递归的斐波那契时间复杂度为 O(2ⁿ)，因为大量重复计算。可以用**记忆化**优化：

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci_fast(n):
    if n <= 0:
        return 0
    if n == 1:
        return 1
    return fibonacci_fast(n - 1) + fibonacci_fast(n - 2)

print(fibonacci_fast(50))  # 12586269025（瞬间完成）
```

### 五、递归的优缺点

| 优点                   | 缺点                                  |
| ---------------------- | ------------------------------------- |
| 代码简洁，贴近数学定义 | 性能开销大（函数调用栈）              |
| 适合树结构、分治问题   | 默认递归深度限制（Python 约 1000 层） |
| 逻辑清晰易懂           | 不当使用容易栈溢出                    |

```python
import sys
print(sys.getrecursionlimit())  # 默认 1000
# sys.setrecursionlimit(5000)  # 可修改，但不推荐设太大
```

---

## 第70讲：函数进阶 —— 案例2

### 案例：多功能字符串处理工具箱

综合运用本章所有知识——默认参数、不定长参数、函数作为参数、Lambda。

```python
def apply_to_all(func, *items):
    """将 func 应用到所有 items 上，返回结果列表"""
    return [func(item) for item in items]

def compose(*functions):
    """函数组合：将多个函数串联执行（从右到左）"""
    def composed(x):
        result = x
        for func in reversed(functions):
            result = func(result)
        return result
    return composed

def create_formatter(prefix="", suffix="", transform=None):
    """创建一个字符串格式化器（工厂函数）"""
    def formatter(text):
        result = text
        if transform:
            result = transform(result)
        return f"{prefix}{result}{suffix}"
    return formatter

# ===== 使用示例 =====

# 1. apply_to_all: 批量处理
results = apply_to_all(str.upper, "hello", "world", "python")
print(results)  # ['HELLO', 'WORLD', 'PYTHON']

results = apply_to_all(lambda s: s[::-1], "hello", "world")
print(results)  # ['olleh', 'dlrow']

# 2. compose: 函数组合
shout = compose(str.upper, str.strip)
print(shout("  hello  "))  # "HELLO" (先 strip，再 upper)

# 3. create_formatter: 工厂函数
html_bold = create_formatter("<b>", "</b>")
print(html_bold("重要内容"))  # <b>重要内容</b>

upper_bracket = create_formatter("[", "]", transform=str.upper)
print(upper_bracket("warning"))  # [WARNING]

# 综合运用
formatters = [
    create_formatter(">>> "),
    create_formatter(transform=str.title),
    create_formatter("「", "」"),
]
text = "hello python"
for fmt in formatters:
    print(fmt(text))
# >>> hello python
# Hello Python
# 「hello python」
```

**案例要点**：
- `apply_to_all`：`*args` + 函数作为参数
- `compose`：不定长参数 + 闭包（函数中定义函数并返回）
- `create_formatter`：默认参数 + 函数作为参数 + 闭包（工厂模式）

---

## 第71讲：类型注解 —— 介绍

### 一、什么是类型注解？

Python 是**动态类型语言**——变量的类型在运行时确定，不需要提前声明。但从 **Python 3.5** 起，Python 引入了**类型注解（Type Hints）**，允许你为变量、函数参数和返回值**标注类型**。

```python
# 没有类型注解
name = "小明"
age = 20

# 有类型注解
name: str = "小明"
age: int = 20
```

### 二、类型注解不是强制的

**关键理解**：类型注解**不会影响代码运行**。Python 解释器**不会**在运行时检查类型注解。

```python
age: int = "我不是整数"   # 不会报错！Python 不做运行时类型检查
print(age)  # 我不是整数
```

类型注解的价值在于：

| 好处             | 说明                                   |
| ---------------- | -------------------------------------- |
| **代码可读性**   | 一眼看出变量/参数应该是什么类型        |
| **IDE 智能提示** | IDE 能据此提供更好的自动补全和错误提示 |
| **静态检查**     | 工具如 `mypy` 可在运行前发现类型错误   |
| **文档作用**     | 类型注解本身就是一种文档               |

### 三、基本变量类型注解

```python
# 基本类型
name: str = "小明"
age: int = 20
height: float = 1.75
is_student: bool = True

# 容器类型（Python 3.9+，可直接用内置类型）
scores: list[int] = [90, 85, 78]
student: dict[str, int] = {"小明": 90, "小红": 85}
unique_ids: set[int] = {1, 2, 3}
point: tuple[int, int] = (10, 20)

# Python 3.8 及更早版本，需要从 typing 导入
from typing import List, Dict, Set, Tuple
scores: List[int] = [90, 85, 78]
student: Dict[str, int] = {"小明": 90}
```

> **推荐**：如果你使用 Python 3.9+，直接用 `list[int]`、`dict[str, int]` 等内置类型即可，无需导入 `typing`。

### 四、特殊类型

```python
from typing import Optional, Union

# Optional：表示"可能是指定类型，也可能是 None"
# Optional[int] 等价于 int | None (Python 3.10+)
def find_user(user_id: int) -> Optional[str]:
    if user_id == 1:
        return "小明"
    return None

# Union：表示"可以是多种类型之一"
# Union[int, str] 等价于 int | str (Python 3.10+)
def process(data: Union[int, str]) -> str:
    return str(data)

# Python 3.10+ 使用 | 语法更简洁
def process(data: int | str) -> str:
    return str(data)

# Any：表示任意类型（不做类型约束）
from typing import Any
def debug_print(value: Any) -> None:
    print(f"[DEBUG] {value}")
```

---

## 第72讲：类型注解 —— 函数的类型注解

### 一、语法

```python
def 函数名(参数1: 类型1, 参数2: 类型2) -> 返回类型:
    函数体
```

`->` 后面标注返回值类型。

```python
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"你好，{name}！"

def is_even(n: int) -> bool:
    return n % 2 == 0
```

### 二、返回 None 的函数

如果函数没有返回值，标注为 `-> None`：

```python
def print_info(name: str, age: int) -> None:
    print(f"{name}, {age}岁")
```

### 三、复杂类型的函数注解

```python
def get_scores() -> list[int]:
    return [90, 85, 78]

def create_user(name: str, age: int) -> dict[str, str | int]:
    return {"name": name, "age": age}

def find_min_max(numbers: list[int]) -> tuple[int, int]:
    return min(numbers), max(numbers)
```

### 四、函数类型作为参数的注解

使用 `Callable` 来标注"函数类型"的参数：

> 兼容性提示：Python 3.9+ 推荐 `from collections.abc import Callable`；若使用 Python 3.8 及更早版本，请改为 `from typing import Callable`。

```python
from collections.abc import Callable

def apply_operation(
    func: Callable[[int, int], int],
    a: int,
    b: int
) -> int:
    """
    func 的类型注解 Callable[[int, int], int] 表示：
    - 接收两个 int 参数
    - 返回一个 int
    """
    return func(a, b)

def add(a: int, b: int) -> int:
    return a + b

result = apply_operation(add, 3, 5)  # 8
```

### 五、带默认参数和不定长参数的注解

```python
def greet(name: str, greeting: str = "你好") -> str:
    return f"{greeting}，{name}！"

def total(*args: int) -> int:
    """*args 中的每个元素都是 int"""
    return sum(args)

def print_info(**kwargs: str) -> None:
    """**kwargs 中的每个值都是 str"""
    for key, value in kwargs.items():
        print(f"{key}: {value}")
```

### 六、综合示例

```python
from collections.abc import Callable

def create_validator(
    min_val: float = 0,
    max_val: float = 100
) -> Callable[[float], bool]:
    """创建一个范围验证器（返回值是一个函数）"""
    def validator(value: float) -> bool:
        return min_val <= value <= max_val
    return validator

def process_scores(
    scores: list[float],
    validator: Callable[[float], bool],
    transform: Callable[[float], float] | None = None
) -> list[float]:
    """处理成绩：验证 + 可选转换"""
    valid_scores: list[float] = []
    for score in scores:
        if validator(score):
            if transform:
                valid_scores.append(transform(score))
            else:
                valid_scores.append(score)
    return valid_scores

# 使用
score_validator = create_validator(0, 100)
raw_scores: list[float] = [95.5, -10, 88.0, 150, 72.3]

# 过滤无效分数
valid = process_scores(raw_scores, score_validator)
print(valid)  # [95.5, 88.0, 72.3]

# 过滤 + 归一化到 [0, 1]
normalized = process_scores(
    raw_scores,
    score_validator,
    transform=lambda x: x / 100
)
print(normalized)  # [0.955, 0.88, 0.723]
```

### 七、使用 mypy 进行静态类型检查

```bash
pip install mypy
mypy your_script.py
```

```python
# example.py
def add(a: int, b: int) -> int:
    return a + b

result = add("hello", "world")  # mypy 会报错：参数类型不匹配
```

```
$ mypy example.py
example.py:4: error: Argument 1 to "add" has incompatible type "str"; expected "int"
```

### 八、本讲小结

| 要点         | 说明                                              |
| ------------ | ------------------------------------------------- |
| 参数注解     | `参数名: 类型`                                    |
| 返回值注解   | `-> 类型`                                         |
| 运行时不强制 | Python 不做运行时类型检查                         |
| 静态检查     | 使用 `mypy` 等工具在运行前发现错误                |
| `Callable`   | 标注"函数类型"的参数                              |
| `Optional`   | 标注"可能是 None"                                 |
| 版本推荐     | Python 3.9+ 直接用 `list[int]`；3.10+ 用 `X \| Y` |

---

## 全章总结

```
函数体系知识图谱
├── 函数基础
│   ├── 函数定义 (def, 命名规范)
│   ├── 参数与返回值 (形参/实参, return, 多返回值)
│   ├── 函数说明文档 (Docstring, Google/NumPy/rST 风格)
│   ├── 函数嵌套调用 (调用栈, 返回值传递)
│   └── 综合案例 (银行系统, 成绩管理, 温度转换)
├── 函数进阶
│   ├── 变量作用域 (LEGB 规则, global, nonlocal)
│   ├── 传参方式 (位置参数, 关键字参数, 对象引用传递)
│   ├── 默认参数 (可变默认参数陷阱!)
│   ├── 不定长参数 (*args, **kwargs, 解包)
│   ├── 函数作为参数 (一等公民, 高阶函数)
│   ├── Lambda 表达式 (匿名函数, sorted/map/filter)
│   ├── 递归 (基准情况, 递推关系, 阶乘, 斐波那契)
│   └── 综合案例 (函数组合, 工厂函数)
└── 类型注解
    ├── 介绍 (动态类型 vs 类型注解, 不强制)
    └── 函数类型注解 (参数注解, 返回值注解, Callable, mypy)
```

以上就是 Python 函数基础、函数进阶和类型注解的完整教程。从最基本的 `def` 定义，到参数传递机制的底层原理，再到 Lambda、递归、类型注解等高级主题，希望这份教程能帮助你系统掌握 Python 函数的方方面面！