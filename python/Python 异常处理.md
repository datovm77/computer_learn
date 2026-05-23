# Python 异常处理完全教程

> 本篇系统讲解 Python 的**异常处理机制**：从语法形式（`try-except-else-finally`、`raise`、`assert`、`with`）到背后的设计思路（异常对象、异常链、传播机制、工程化异常体系）。读完本篇，你应该能写出"出错时优雅退出、不崩溃、不丢信息"的健壮代码。

## 学习路线

| 阶段   | 学什么                                            | 解决什么问题                                     |
| ------ | ------------------------------------------------- | ------------------------------------------------ |
| 第一章 | 异常基本概念、`try-except` 基础、异常对象         | 看懂报错、能用 `try-except` 拦住已知错误         |
| 第二章 | `else`、`finally`、捕获顺序、`Exception` 基类     | 写出结构完整、不会误吞错误的异常处理代码         |
| 第三章 | `raise`、自定义异常、异常传播、异常链             | 主动报错、设计自己的异常体系                     |
| 第四章 | `traceback`、`assert`、常见异常分类、`logging`    | 调试、断言、把错误打进日志                       |
| 第五章 | 文件异常处理、`with` 上下文管理器、工程化异常设计 | 在真实工程中合理使用异常                         |
| 第六章 | 综合练习：5 个完整小项目                          | 把所学串起来，模拟真实开发场景                   |
| 附录   | 异常组（3.11+）、反模式速查、调试技巧             | 进阶补充与避坑指南                               |

---

## 第一章：异常的基本概念与捕获

### 1.1 什么是异常

在程序运行过程中，如果发生了**预期之外的事件**——比如除以零、访问不存在的下标、打开不存在的文件——Python 不会让程序"默默出错"，而是会创建一个**异常对象**，中断当前正在执行的代码，并把这个异常向外抛出。

> **语法错误通常不是本教程重点。** `SyntaxError` 也属于 Python 异常体系，但通常发生在解析/编译阶段；本章重点讨论代码语法没问题、运行时遇到无法处理情形时产生的异常。

来看一个最直接的例子：

```python
print("开始")
result = 10 / 0          # 这里会抛出 ZeroDivisionError
print("结束")            # 这一行不会执行
```

类似输出：

```
开始
Traceback (most recent call last):
  File "demo.py", line 2, in <module>
    result = 10 / 0
ZeroDivisionError: division by zero
```

注意三点：

1. `print("结束")` **没有执行**——异常会中断后续代码。
2. Python 打印了一段叫做 **Traceback** 的报错信息，告诉你出错的位置和原因。
3. 程序最终以**非零退出码**结束，操作系统会把它视为"失败"。

### 1.2 常见异常类型

Python 内置了大量异常类型，每种都对应一类典型错误。下面这张表是日常开发中遇到频率最高的几种：

| 异常类型              | 触发场景                                   | 示例                                |
| --------------------- | ------------------------------------------ | ----------------------------------- |
| `ZeroDivisionError`   | 除以零                                     | `10 / 0`                            |
| `TypeError`           | 类型不匹配                                 | `"abc" + 1`                         |
| `ValueError`          | 类型对，但值不合理                         | `int("abc")`                        |
| `NameError`           | 引用了不存在的变量名                       | `print(xyz)` 而 `xyz` 没定义        |
| `IndexError`          | 序列下标越界                               | `[1, 2, 3][5]`                      |
| `KeyError`            | 字典中没有这个键                           | `{"a": 1}["b"]`                     |
| `AttributeError`      | 对象没有这个属性                           | `"abc".foo()`                       |
| `FileNotFoundError`   | 文件不存在                                 | `open("not_exist.txt")`             |
| `PermissionError`     | 没有读写权限                               | 读取系统受保护的文件                |
| `UnicodeDecodeError`  | 编码不匹配                                 | 用 UTF-8 读 GBK 文件                |
| `ImportError`         | 模块导入失败                               | `from math import not_exist_name`   |
| `ModuleNotFoundError` | `ImportError` 的子类，模块根本不存在       | `import xyz`                        |
| `RuntimeError`        | 运行时通用错误，没有更合适分类时使用       | 框架内部状态错误                    |
| `StopIteration`       | 迭代器已经耗尽                             | `next(iter([]))`                    |

> **学习建议：** 不需要硬背。先记住前 8 个高频项，遇到其他异常时查文档或 `help(异常名)` 即可。

### 1.3 try-except：捕获异常

异常处理的核心语法是 `try-except`：

```python
try:
    # 可能出错的代码
    ...
except 异常类型:
    # 出错后要做什么
    ...
```

把 1.1 节那段代码改成"出错也不崩"：

```python
print("开始")
try:
    result = 10 / 0
except ZeroDivisionError:
    print("除数不能为零，已忽略本次计算")
print("结束")
```

类似输出：

```
开始
除数不能为零，已忽略本次计算
结束
```

**关键点：**

- 程序没有崩溃，三行 `print` 都按顺序执行了。
- `except` 后面写的是**异常类型**——只有这个类型（或它的子类）的异常会被捕获。

#### 用户输入校验的典型场景

```python
text = input("请输入一个整数：")
try:
    n = int(text)
    print(f"你输入的是 {n}，平方为 {n * n}")
except ValueError:
    print(f"输入 '{text}' 不是合法整数")
```

如果用户输入 `abc`，`int("abc")` 会抛 `ValueError`，但程序不会崩，而是给出友好提示。

### 1.4 捕获多个异常

一段代码可能抛出**不止一种**异常。Python 提供了多种写法。

#### 写法一：多个 except 分支

```python
text = input("请输入一个整数：")
try:
    n = int(text)
    result = 100 / n
    print(f"100 / {n} = {result}")
except ValueError:
    print("输入不是合法整数")
except ZeroDivisionError:
    print("整数不能为 0")
```

异常匹配是**自上而下**的：哪个 `except` 第一个匹配上，就执行哪个分支，其余分支跳过。

#### 写法二：一个 except 捕获多种异常（元组）

如果对几种异常处理方式相同，可以把它们合并成一个元组：

```python
try:
    n = int(input("请输入一个整数："))
    result = 100 / n
except (ValueError, ZeroDivisionError):
    print("输入有问题：要么不是整数，要么是 0")
```

> **注意：** 为了兼容 Python 3.13 及以下，多个异常类型统一写成**元组** `(A, B)`。Python 3.14+ 在不使用 `as` 绑定异常对象时才允许省略括号。

---

### 1.5 异常对象：`except 异常类型 as e`

异常本身是一个**对象**。如果你想知道"具体出了什么错"，可以用 `as` 把异常对象绑定到一个变量：

```python
try:
    n = int("abc")
except ValueError as e:
    print(f"出错了：{e}")
    print(f"异常类型：{type(e).__name__}")
```

输出：

```
出错了：invalid literal for int() with base 10: 'abc'
异常类型：ValueError
```

**`e` 上常用的属性：**

| 表达式               | 含义                                       |
| -------------------- | ------------------------------------------ |
| `str(e)`             | 异常的描述信息                             |
| `type(e).__name__`   | 异常类型名（字符串）                       |
| `e.args`             | 创建异常时传入的所有参数，元组形式         |
| `e.__traceback__`    | 异常的 traceback 对象（高级用途）          |

```python
try:
    raise ValueError("年龄必须 >= 0", -3)
except ValueError as e:
    print(e.args)          # ('年龄必须 >= 0', -3)
    print(e.args[0])       # 年龄必须 >= 0
    print(e.args[1])       # -3
```

---

## 第二章：完整的异常处理结构

### 2.1 else 与 finally

`try-except` 还可以搭配 `else` 和 `finally`，组成完整的四段式结构：

```python
try:
    # 可能出错的代码
    ...
except 异常类型:
    # 出错时执行
    ...
else:
    # 没有出错时才执行
    ...
finally:
    # 无论是否出错都执行
    ...
```

每一段的作用都很明确：

| 关键字     | 触发时机                       | 典型用途                       |
| ---------- | ------------------------------ | ------------------------------ |
| `try`      | 总是执行                       | 放可能出错的核心代码           |
| `except`   | `try` 中抛出匹配异常时         | 处理错误、给出友好提示         |
| `else`     | `try` 正常执行完毕且没有异常、`return`、`break`、`continue` 时 | 放"成功之后"的逻辑             |
| `finally`  | 正常控制流下无论是否出错都会执行 | 释放资源（关文件、关连接等）   |

#### 完整示例

```python
def safe_divide(a, b):
    print(f"开始计算 {a} / {b}")
    try:
        result = a / b
    except ZeroDivisionError:
        print("  [except] 除数为 0，无法计算")
        return None
    else:
        print(f"  [else] 计算成功，结果为 {result}")
        return result
    finally:
        print("  [finally] 收尾工作（清理资源等）")


safe_divide(10, 2)
safe_divide(10, 0)
```

输出：

```
开始计算 10 / 2
  [else] 计算成功，结果为 5.0
  [finally] 收尾工作（清理资源等）

开始计算 10 / 0
  [except] 除数为 0，无法计算
  [finally] 收尾工作（清理资源等）
```

> **细节：** 即使 `except` 或 `else` 里有 `return`，`finally` 仍然会在函数真正返回之前执行。这是 `finally` 的**核心保证**。这里说的"总会"指正常解释器控制流；进程被强杀、调用 `os._exit()` 或解释器崩溃时不保证。也不要在 `finally` 中写 `return`、`break`、`continue`，否则可能覆盖原本的异常或返回值。

#### else 存在的意义

为什么不直接把"成功之后的逻辑"写在 `try` 里？看下面对比：

```python
# 写法 A：把成功逻辑写在 try 里
try:
    n = int(input())
    print(f"你输入了 {n}")        # ← 万一这里抛出别的异常，会被下面误捕获
except ValueError:
    print("输入非法")

# 写法 B：用 else
try:
    n = int(input())
except ValueError:
    print("输入非法")
else:
    print(f"你输入了 {n}")        # ← 这里抛异常不会被上面的 except 捕获
```

**写法 B 的好处：** `except` 只负责拦截 `int()` 自己产生的异常，"成功之后"的代码出错时不会被它误吞。这让异常处理的"作用范围"更精准。

#### finally 与资源释放

`finally` 最常见的场景是**释放资源**——文件句柄、数据库连接、网络套接字等。

```python
f = open("data.txt", encoding="utf-8")
try:
    content = f.read()
    process(content)             # 这里可能出异常
finally:
    f.close()                    # 无论是否出异常都会关闭
```

> **注：** 现代 Python 中，文件操作推荐用 `with` 语句（详见 5.2 节）。`with` 会自动调用 `close()`，相当于内置了 `finally`。

---

### 2.2 异常捕获顺序：子类在前，父类在后

Python 异常是**类继承体系**——具体异常都继承自更通用的异常。比如：

```
BaseException
 └── Exception
      ├── ArithmeticError
      │    ├── ZeroDivisionError
      │    └── OverflowError
      ├── LookupError
      │    ├── IndexError
      │    └── KeyError
      ├── ValueError
      ├── TypeError
      ├── OSError
      │    ├── FileNotFoundError
      │    └── PermissionError
      └── ...
```

`except` 匹配时遵循 `isinstance` 规则：**子类异常也会被父类的 except 捕获**。

#### 错误示范：父类写在前面

```python
try:
    [1, 2, 3][10]
except Exception as e:                    # ← 父类先写，会先匹配
    print(f"捕获了 Exception：{e}")
except IndexError as e:                   # ← 永远走不到这里！
    print(f"捕获了 IndexError：{e}")
```

输出：

```
捕获了 Exception：list index out of range
```

`IndexError` 是 `Exception` 的子类，第一个 `except Exception` 已经把它捕获了，第二个分支永远不会被命中。

#### 正确做法：子类在前

```python
try:
    [1, 2, 3][10]
except IndexError as e:                   # ← 先匹配具体的子类
    print(f"下标越界：{e}")
except Exception as e:                    # ← 再兜底匹配其他所有异常
    print(f"其他错误：{e}")
```

> **记忆口诀：** **从具体到通用，从特殊到一般。** 写多个 `except` 时，越具体的类型越靠前。

---

### 2.3 Exception 与 BaseException

Python 的异常顶层结构是这样的：

```
BaseException                  ← 所有异常的总基类
 ├── SystemExit                 ← sys.exit() 抛出
 ├── KeyboardInterrupt          ← 用户按 Ctrl+C 抛出
 ├── GeneratorExit              ← 生成器被关闭抛出
 ├── BaseExceptionGroup         ← 异常组的基类（Python 3.11+）
 └── Exception                  ← "普通"异常的基类
      ├── ZeroDivisionError
      ├── ValueError
      ├── ExceptionGroup        ← 同时也是 BaseExceptionGroup 的子类
      ├── ...
```

**关键区分：**

- **`Exception`** 是大多数业务异常的基类——绝大多数你需要捕获的异常都是它的子类。
- **`BaseException`** 是更顶层的基类，它的直接子类 `SystemExit`、`KeyboardInterrupt`、`GeneratorExit` 不属于普通错误，而是**控制流信号**。
- **`ExceptionGroup`** 用于打包多个异常（见附录 A），它也属于 `Exception`，所以 `except Exception` 可以捕获未被 `except*` 拆分处理的异常组。

#### 为什么通常不要捕获 BaseException？

```python
# ❌ 危险写法
try:
    do_something()
except BaseException:           # 把 Ctrl+C 也吞掉了
    pass
```

这种写法会让 `Ctrl+C` 无效——用户想强制结束程序时，按 Ctrl+C 会被这里捕获，程序继续运行，让人非常困扰。

```python
# ✅ 推荐写法
try:
    do_something()
except Exception:               # 只捕获"普通异常"，不影响 Ctrl+C 退出
    print("出错了")
```

> **规则：** 用 `except Exception:` 兜底就足够了，几乎从来不需要写 `except BaseException:`。除非你确实知道自己在做什么（比如清理资源后再重新抛出）。3
>
> .0
>
> +-
>
> +0...00.
>
> ****9**-///**

#### 裸 except 也要避免

```python
# ❌ 危险：等价于 except BaseException
try:
    ...
except:
    ...
```

不带任何异常类型的 `except:` 会捕获**所有**异常，包括 `KeyboardInterrupt`。**任何时候都不要这么写。** 至少写 `except Exception:`。

---

## 第三章：主动抛出与传播

### 3.1 raise：主动抛出异常

到目前为止，所有异常都是 Python 自己抛出的。但很多时候，**你希望在代码不合预期时主动报错**——这就需要 `raise`。

#### 基本用法

```python
def set_age(age):
    if age < 0:
        raise ValueError(f"年龄必须 >= 0，收到 {age}")
    if age > 150:
        raise ValueError(f"年龄太大，收到 {age}")
    print(f"年龄设置成功：{age}")


set_age(25)         # 年龄设置成功：25
set_age(-3)         # ValueError: 年龄必须 >= 0，收到 -3
```

#### `raise` 语句的形式

```text
raise 异常类(参数)        # 最常用：创建一个异常对象并抛出
raise 异常类              # 等价于 raise 异常类()，无参数
raise 异常实例            # 抛出已经创建好的异常对象
```

不带参数的裸 `raise` 只能写在正在处理异常的 `except` 块里；否则会抛出 `RuntimeError`。

#### 在 except 中重新抛出

有时你想"先记一笔再继续抛"，可以用不带参数的 `raise`：

```python
def parse_number(text):
    try:
        return int(text)
    except ValueError:
        print(f"[日志] 无法解析的输入：{text!r}")
        raise                    # ← 重新抛出当前异常，不丢失原始信息
```

调用 `parse_number("abc")` 会先打印日志，再把 `ValueError` 继续向上抛。

> **使用场景：** 你想在中间层"截获-记录-放行"——既不吞掉错误，又留下了痕迹。

---

### 3.2 自定义异常

内置异常往往不够精准。比如你写一个银行转账模块，转账失败时抛 `ValueError` 就太笼统了——调用方很难分辨"是金额不合法"还是"是余额不足"。这时应该**定义自己的异常类**。

#### 最简单的自定义异常

```python
class InsufficientBalanceError(Exception):
    """余额不足时抛出"""
    pass


def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientBalanceError(f"余额 {balance} 不足以支付 {amount}")
    return balance - amount


try:
    withdraw(100, 200)
except InsufficientBalanceError as e:
    print(f"取款失败：{e}")
```

**规则：**

- 自定义业务异常通常继承 `Exception`（或它的某个子类）；从语法上，可被 `raise` 的对象必须属于 `BaseException` 体系，但业务异常不要直接继承 `BaseException`。
- 类名通常以 `Error` 或 `Exception` 结尾，便于阅读。
- 类体可以只写 `pass`——继承自 `Exception` 的基础行为就够用了。

#### 带额外数据的自定义异常

异常不只是"消息字符串"，它可以携带任何你需要的上下文信息：

```python
class InsufficientBalanceError(Exception):
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(f"余额 {balance} 不足以支付 {amount}")


try:
    raise InsufficientBalanceError(balance=100, amount=200)
except InsufficientBalanceError as e:
    print(e)                       # 余额 100 不足以支付 200
    print(e.balance, e.amount)     # 100 200
    print(e.amount - e.balance)    # 100  ← 还差多少
```

调用方就能拿到结构化数据，而不是只能 `str(e)` 然后正则提取。

#### 设计你自己的异常体系

如果一个模块抛出多种相关错误，可以建一棵小小的继承树：

```python
class BankError(Exception):
    """银行相关错误的基类"""
    pass


class InsufficientBalanceError(BankError):
    pass


class AccountFrozenError(BankError):
    pass


class InvalidAmountError(BankError):
    pass
```

这样调用方就有两种粒度可选：

```python
# 粒度细：分别处理
try:
    withdraw(...)
except InsufficientBalanceError:
    ...
except AccountFrozenError:
    ...

# 粒度粗：一把抓
try:
    withdraw(...)
except BankError as e:
    print(f"银行业务出错：{e}")
```

---

### 3.3 异常传播机制

异常被抛出后，如果**当前函数没有捕获**，它会向**调用栈**上层一层层传播，直到被某层 `try-except` 接住，或者一直传到顶层导致程序终止。

```python
def level3():
    print("进入 level3")
    raise ValueError("level3 出错")          # 在最深处抛出
    print("离开 level3")                    # 不会执行


def level2():
    print("进入 level2")
    level3()
    print("离开 level2")                    # 不会执行


def level1():
    print("进入 level1")
    try:
        level2()
    except ValueError as e:
        print(f"在 level1 捕获：{e}")
    print("离开 level1")


level1()
```

输出：

```
进入 level1
进入 level2
进入 level3
在 level1 捕获：level3 出错
离开 level1
```

执行流程：

1. `level1` 调用 `level2`，`level2` 调用 `level3`。
2. `level3` 抛出 `ValueError` → 没人接，向上传到 `level2`。
3. `level2` 也没接，继续向上传到 `level1`。
4. `level1` 的 `try-except` 接住了。
5. `try` 之后的代码继续正常执行。

> **设计启示：** 不必在每个函数里都写 `try-except`。让异常自然传播到**能真正处理它的层级**——通常是离用户最近的入口函数、或者业务边界（比如 Web 框架的请求处理器）。

#### 没人接住会怎样？

```python
def main():
    raise RuntimeError("致命错误")

main()
```

异常一路传到最顶层，没有任何 `except` 接住——Python 会打印完整的 traceback，然后以非零状态码退出程序。

---

### 3.4 异常链：`raise ... from ...`

很多时候，一个异常是由另一个异常**引起的**。你希望抛新异常时**不丢失原始原因**，可以用 `raise ... from ...`：

```python
class ConfigError(Exception):
    pass


def load_config(path):
    try:
        with open(path, encoding="utf-8") as f:
            content = f.read()
    except FileNotFoundError as e:
        raise ConfigError(f"配置文件读取失败：{path}") from e
    return parse(content)


load_config("missing.ini")
```

类似输出：

```
Traceback (most recent call last):
  File "demo.py", line ..., in load_config
    with open(path, encoding="utf-8") as f:
FileNotFoundError: [Errno 2] No such file or directory: 'missing.ini'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "demo.py", line ..., in <module>
    load_config("missing.ini")
  File "demo.py", line ..., in load_config
    raise ConfigError(f"配置文件读取失败：{path}") from e
ConfigError: 配置文件读取失败：missing.ini
```

**两段 Traceback 都被保留：** 上层是原始的 `FileNotFoundError`，下层是新抛的 `ConfigError`，中间用"The above exception was the direct cause of the following exception"连起来。

#### 三种写法对比

```python
# 写法 1：raise ... from ...     显式标注因果关系
raise ConfigError("加载失败") from e

# 写法 2：raise NewError(...)     在 except 里抛新异常，Python 会自动保留隐式链
raise ConfigError("加载失败")

# 写法 3：raise ... from None     主动隐藏原始异常
raise ConfigError("加载失败") from None
```

| 写法           | 表现                                                          |
| -------------- | ------------------------------------------------------------- |
| `from e`       | 明确说"这个新异常是 e 引起的"——推荐用于业务层包装底层异常     |
| 不加 from      | Python 自动用 "During handling of..." 把原异常作为上下文带上 |
| `from None`    | 抑制链，只显示新异常——适合你完全不想暴露底层细节的场景       |

> **实用建议：** 在自己的业务层抛包装异常时，**默认使用 `from e`**，调试时能看到完整因果链；只有当原始异常会泄露敏感信息或干扰理解时，才用 `from None`。

---

## 第四章：调试、断言与日志

### 4.1 traceback：看懂报错信息

Traceback（追溯信息）是 Python 报错的"案发现场重建报告"。学会读它能让你的调试效率翻倍。

#### 一个典型的 traceback

```python
def divide(a, b):
    return a / b

def calculate():
    return divide(10, 0)

calculate()
```

类似报错：

```
Traceback (most recent call last):
  File "demo.py", line 7, in <module>
    calculate()
  File "demo.py", line 5, in calculate
    return divide(10, 0)
    ~~~~~~^^^^^^^
  File "demo.py", line 2, in divide
    return a / b
           ~~^~~
ZeroDivisionError: division by zero
```

**逐行解读：**

1. `Traceback (most recent call last):`——固定开头，提示"最近的调用在最下面"。
2. 每个 `File ... line ... in ...` 代表调用栈上的一帧（frame）。
3. **从上到下**：调用从哪里开始（`<module>` 顶层）→ 调用了 `calculate` → `calculate` 调用了 `divide` → 在 `divide` 内部出错。
4. **最下面一行**是异常类型和消息：`ZeroDivisionError: division by zero`。
5. Python 3.11+ 还会用 `~~^~~` 这种波浪线标记**精确到列**出错的位置。

> **看 traceback 的口诀：** 从下往上看类型，从上往下看路径。

#### 用 traceback 模块手动打印

有时候你捕获了异常但还想看完整的栈信息：

```python
import traceback

try:
    1 / 0
except ZeroDivisionError:
    traceback.print_exc()        # 打印当前异常的完整 traceback
```

或者把 traceback 存成字符串，方便写日志：

```python
import traceback

try:
    1 / 0
except ZeroDivisionError:
    info = traceback.format_exc()
    print("--- 错误详情 ---")
    print(info)
```

---

### 4.2 assert：条件断言

`assert` 是一种**简洁的运行时检查**，用来确认"程序内部状态是否符合预期"。

#### 基本语法

```text
assert 条件, 出错信息
```

- 条件为真：什么都不做，继续往下执行。
- 条件为假：抛出 `AssertionError`，可选信息会作为异常的消息。

```python
def average(numbers):
    assert len(numbers) > 0, "列表不能为空"
    return sum(numbers) / len(numbers)


average([1, 2, 3])        # 2.0
average([])               # AssertionError: 列表不能为空
```

这个例子只适合内部函数：调用方本应保证传入非空列表。对外 API 校验空输入时，应使用 `if not numbers: raise ValueError(...)`。

#### `assert` 不是用来做用户输入校验的

```python
# ❌ 不要这样用
def set_age(age):
    assert age >= 0, "年龄必须 >= 0"
    ...
```

**两个原因：**

1. **`assert` 可以被关闭。** Python 解释器以 `-O`（optimize）选项启动时，所有 `assert` 都会被跳过——你的校验就失效了。
2. **`assert` 抛 `AssertionError`，语义不准确。** 用户输入非法属于业务错误，应该用 `ValueError` 等更具体的异常。

#### `assert` 的正确使用场景

- **内部不变量检查**：开发阶段确认"这种情况不可能发生"。
- **测试代码**：单元测试里大量使用。
- **临时调试**：写一个 `assert x > 0` 来排查某个变量为什么变成了负数。

```python
def binary_search(sorted_list, target):
    # 内部不变量：列表必须有序
    assert all(sorted_list[i] <= sorted_list[i + 1]
               for i in range(len(sorted_list) - 1)), "列表不是升序的"
    ...
```

> **口诀：** **校验用户给的，用异常；校验自己写的，用 assert。**

---

### 4.3 常见异常分类速查

下面把第一章那张表按"作用类别"重新整理，方便记忆：

#### 类型相关

| 异常         | 触发场景                          |
| ------------ | --------------------------------- |
| `TypeError`  | 类型不匹配，比如 `"a" + 1`        |
| `ValueError` | 类型对但值不合理，如 `int("abc")` |

#### 名称/属性相关

| 异常             | 触发场景                          |
| ---------------- | --------------------------------- |
| `NameError`      | 变量名未定义                      |
| `UnboundLocalError` | 局部变量在赋值前被使用            |
| `AttributeError` | 对象没有该属性                    |

#### 容器查找相关

| 异常         | 触发场景                       |
| ------------ | ------------------------------ |
| `IndexError` | 序列下标越界（列表、元组、字符串） |
| `KeyError`   | 字典中没有该键                 |
| `LookupError` | `IndexError` 和 `KeyError` 的共同父类 |

#### 文件/系统相关

| 异常                 | 触发场景                       |
| -------------------- | ------------------------------ |
| `FileNotFoundError`  | 文件不存在                     |
| `PermissionError`    | 没有读写权限                   |
| `IsADirectoryError`  | 试图把目录当文件操作           |
| `OSError`            | 上述异常的共同父类，系统级错误 |

#### 数值相关

| 异常                | 触发场景                       |
| ------------------- | ------------------------------ |
| `ZeroDivisionError` | 除以零                         |
| `OverflowError`     | 数值结果过大无法表示           |
| `ArithmeticError`   | 算术运算异常的共同父类         |

#### 编码相关

| 异常                 | 触发场景                       |
| -------------------- | ------------------------------ |
| `UnicodeDecodeError` | 字节解码为字符串失败           |
| `UnicodeEncodeError` | 字符串编码为字节失败           |
| `UnicodeError`       | 上述两者的共同父类             |

> **小窍门：** 当你不确定该写哪个异常时，先让代码在不捕获、不包装的情况下运行一次，看 traceback 里实际抛出的异常类型——通常那就是最合适的类型。

---

### 4.4 logging 与异常日志

在生产环境中，光打印异常是不够的——你需要把错误**记录到日志文件**，便于事后排查。Python 内置了 `logging` 模块，专门用来做这件事。

#### 最简单的日志配置

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
)

logging.info("程序启动")
logging.warning("配置项缺失，使用默认值")
logging.error("数据库连接失败")
```

#### `logging.exception()`：异常日志的最佳工具

在 `except` 块中调用 `logging.exception()`，它会自动把**完整 traceback**写进日志：

```python
import logging

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s [%(levelname)s] %(message)s",
                    datefmt="%Y-%m-%d %H:%M:%S")


def divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        logging.exception("计算 %s / %s 时除以零", a, b)
        return None


divide(10, 0)
```

类似输出：

```
2026-05-14 10:30:45 [ERROR] 计算 10 / 0 时除以零
Traceback (most recent call last):
  File "demo.py", line 9, in divide
    return a / b
ZeroDivisionError: division by zero
```

`logging.exception()` 等价于 `logging.error(..., exc_info=True)`——会自动附带当前异常的 traceback。**只应在 `except` 块内调用**；在 `except` 外调用没有实际异常可附加，通常只会记录 `NoneType: None`。

#### 三种异常记录方式对比

```python
import logging

try:
    1 / 0
except ZeroDivisionError as e:
    # 方式 1：print —— 只能调试，不能进日志系统
    print(f"出错了：{e}")

    # 方式 2：logging.error —— 只有消息，没有 traceback
    logging.error(f"出错了：{e}")

    # 方式 3：logging.exception —— 消息 + 完整 traceback（推荐）
    logging.exception("出错了")
```

> **生产建议：** 对未预期、无法完全处理、或者准备忽略但后续可能需要排查的异常，至少要记录日志。**永远不要写 `except: pass` 把异常完全吞掉**——这等于自己埋雷。

---

## 第五章：文件、上下文与工程化设计

### 5.1 文件异常处理

文件操作是异常最容易出现的地方。常见的几种错误如下：

```python
try:
    with open("data.txt", encoding="utf-8") as f:
        content = f.read()
        process(content)
except FileNotFoundError:
    print("文件不存在")
except PermissionError:
    print("没有读取权限")
except IsADirectoryError:
    print("这是一个目录，不是文件")
except UnicodeDecodeError:
    print("编码不匹配，请检查文件编码")
except OSError as e:
    # 其他系统级 IO 错误：磁盘满、坏块、网络挂载断开等
    print(f"系统错误：{e}")
```

**关注点：**

- 文件相关异常**几乎都是 `OSError` 的子类**——如果不想分情况处理，可以只写一个 `except OSError`。
- `UnicodeDecodeError` 是 `ValueError` 的子类，不属于 `OSError`，要单独处理。
- 这里的 `except` 顺序是"子类在前、父类在后"，符合 2.2 节的规则。

#### EAFP vs LBYL

Python 社区有两种处理"可能失败"的风格：

```python
# LBYL（Look Before You Leap）：先检查再操作
import os
if os.path.exists(path):
    with open(path) as f:
        content = f.read()
else:
    print("文件不存在")

# EAFP（Easier to Ask Forgiveness than Permission）：直接干，出错再说
try:
    with open(path) as f:
        content = f.read()
except FileNotFoundError:
    print("文件不存在")
```

**Python 社区更推荐 EAFP**——理由是：

1. **避免竞态条件**：在多线程或多进程环境下，`os.path.exists()` 和 `open()` 之间文件可能被删掉，LBYL 反而不安全。
2. **代码更简洁**：异常处理本身就是"主路径 + 失败处理"的清晰结构。

> 但这不绝对。比如要确认目录存在再创建子目录，先 `os.makedirs(path, exist_ok=True)` 直接干比先 `exists()` 更省事——只要 API 自身支持就用 API 自身的"幂等参数"。

---

### 5.2 with 与上下文管理器

第二章末尾提到，文件操作推荐用 `with`——它能保证文件最终一定被关闭。这是**上下文管理器**（context manager）的应用之一。

#### with 是异常安全的

```python
# 不用 with，需要手动 try-finally
f = open("data.txt", encoding="utf-8")
try:
    content = f.read()
    process(content)               # 即使这里抛异常
finally:
    f.close()                      # 文件也一定会关闭

# 用 with，自动等价于上面
with open("data.txt", encoding="utf-8") as f:
    content = f.read()
    process(content)
# 离开 with 块时自动调用 f.close()，无论是否出异常
```

`with` 块的退出机制相当于内置了 `finally`——这是它的**核心保证**。

#### 自己写一个上下文管理器

任何实现了 `__enter__` 和 `__exit__` 的对象都可以用在 `with` 语句中。

```python
class Timer:
    """测量代码块执行时间的上下文管理器"""

    def __enter__(self):
        import time
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        self.elapsed = time.perf_counter() - self.start
        print(f"耗时：{self.elapsed:.4f} 秒")
        # 返回 False（或 None）：异常会继续向外传播
        # 返回 True：异常会被"吞掉"（一般不要这么做）
        return False


with Timer():
    total = sum(i * i for i in range(1_000_000))
```

类似输出：

```
耗时：0.xxxx 秒
```

`__exit__` 的三个参数：

| 参数       | 含义                                                |
| ---------- | --------------------------------------------------- |
| `exc_type` | 异常的类（无异常时为 `None`）                       |
| `exc_val`  | 异常的实例                                          |
| `exc_tb`   | traceback 对象                                      |

如果 `with` 块内没有抛异常，三个参数都是 `None`。

#### 用 `contextlib.contextmanager` 简化

写完整的 `__enter__` / `__exit__` 有点啰嗦。`contextlib.contextmanager` 装饰器可以用**生成器**写一个上下文管理器：

```python
from contextlib import contextmanager
import time


@contextmanager
def timer():
    start = time.perf_counter()
    try:
        yield                              # 这里"暂停"，让 with 块的代码运行
    finally:
        elapsed = time.perf_counter() - start
        print(f"耗时：{elapsed:.4f} 秒")


with timer():
    total = sum(i * i for i in range(1_000_000))
```

`yield` 前的代码相当于 `__enter__`，`yield` 后的代码相当于 `__exit__`。用 `try-finally` 包裹 `yield`，能确保即使 `with` 块内抛异常，清理代码也会执行。

---

### 5.3 工程化异常设计

到这里，所有语法点都讲完了。最后聊一聊**如何在真实项目中合理使用异常**——这部分没有标准答案，只有一些被广泛接受的设计原则。

#### 原则 1：明确异常的"作用范围"

异常处理的代码应该写在**能真正处理它的层级**，不要写在最里面。

```python
# ❌ 反例：在最底层每个函数都 try-except
def read_file(path):
    try:
        with open(path) as f:
            return f.read()
    except Exception as e:
        print(f"读取失败：{e}")
        return None


def parse_config(path):
    content = read_file(path)
    if content is None:                # ← 调用方需要检查 None
        return None
    try:
        return json.loads(content)
    except Exception as e:
        print(f"解析失败：{e}")
        return None


def main():
    config = parse_config("config.json")
    if config is None:                 # ← 调用方又得检查一次
        ...
```

每一层都把异常变成 `None`，调用方反而要在每一层都做 `is None` 判断，代码冗余且容易漏掉一处。

```python
# ✅ 正例：让异常自然传播，在入口统一处理
def read_file(path):
    with open(path) as f:
        return f.read()


def parse_config(path):
    return json.loads(read_file(path))


def main():
    try:
        config = parse_config("config.json")
        run(config)
    except FileNotFoundError:
        print("找不到配置文件")
        sys.exit(1)
    except json.JSONDecodeError as e:
        print(f"配置文件格式错误：{e}")
        sys.exit(1)
```

**原则：让异常向上传播，只在最适合处理的地方接住它。** 这通常是程序入口、Web 请求处理器、消息队列消费者这种"业务边界"。

#### 原则 2：用自定义异常包装底层异常

对外暴露的接口，不应该让调用方接触到一堆 `FileNotFoundError`、`json.JSONDecodeError` 这类"实现细节异常"。**用自定义异常包一层**：

```python
class ConfigError(Exception):
    """配置加载失败"""
    pass


def load_config(path):
    try:
        with open(path, encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError as e:
        raise ConfigError(f"配置文件不存在：{path}") from e
    except json.JSONDecodeError as e:
        raise ConfigError(f"配置文件格式错误：{path}") from e
```

调用方只需要 `except ConfigError`，**屏蔽了底层实现**：将来你换成 YAML、TOML，调用方代码完全不用动。同时用 `from e` 保留了原因链，调试时能看到根因。

#### 原则 3：异常类型要"够用就好"

不要为每一种轻微的错误都创建一个异常类——那是过度设计。一个判断标准是：

> 如果调用方需要对两种错误**做不同处理**，那它们应该是不同的异常类。如果调用方总是用同样的方式处理，就用同一个类，错误消息里区分即可。

#### 原则 4：异常用于"异常情况"，不要用作流程控制

```python
# ❌ 反例：用异常实现"找到就返回"
def find_user(users, name):
    try:
        return next(u for u in users if u.name == name)
    except StopIteration:
        return None
```

虽然能工作，但这里的"没找到"是正常结果，用普通返回值更清晰；不要为了常规分支刻意制造异常路径。EAFP 是直接尝试可能失败的操作，并在失败路径处理异常；反模式是把普通业务分支刻意设计成异常路径。

```python
# ✅ 正例：用普通流程
def find_user(users, name):
    for u in users:
        if u.name == name:
            return u
    return None
```

**异常是为"不应该发生但发生了"的情况准备的**，不要把它当成 `goto`。

#### 原则 5：别让 `except` 静默吞错

```python
# ❌ 最危险的反模式
try:
    risky_operation()
except Exception:
    pass                            # 错误被完全埋掉
```

将来出问题时，你根本不知道哪里、什么时候、出了什么错。**至少要记录日志**：

```python
try:
    risky_operation()
except Exception:
    logging.exception("risky_operation 失败，已忽略")
```

如果你**真的**想忽略某个错误，也要明确写出**为什么**：

```python
try:
    os.remove(temp_file)
except FileNotFoundError:
    pass    # 文件已经被别人删了，正好达到目的，无需处理
```

#### 原则总结

| 原则                       | 一句话                                       |
| -------------------------- | -------------------------------------------- |
| 异常作用范围               | 让异常传播到能处理它的层，不要每层都 try     |
| 包装底层异常               | 对外用自己的异常类型，用 `from` 保留原因     |
| 异常体系够用就好           | 调用方需要区分处理时才分新类                 |
| 异常 ≠ 流程控制            | 不是"错"的情况就不要用异常                   |
| 不要静默吞错               | 至少 `logging.exception()` 记录              |

---

## 第六章：综合练习

前面五章把语法和原则都讲完了。这一章用五个完整的小项目，把所学的知识串起来。

### 练习 1：健壮的整数计算器

需求：循环让用户输入两个数和一个运算符，给出结果；非法输入要给出准确提示，但不能崩溃；用户按 Ctrl+C 时优雅退出。

```python
import logging
import sys

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s [%(levelname)s] %(message)s")


class CalcError(Exception):
    """计算器相关错误"""


def parse_int(text, name):
    try:
        return int(text)
    except ValueError as e:
        raise CalcError(f"{name} 不是合法整数：{text!r}") from e


def compute(a, b, op):
    if op == "+":
        return a + b
    if op == "-":
        return a - b
    if op == "*":
        return a * b
    if op == "/":
        if b == 0:
            raise CalcError("除数不能为 0")
        return a / b
    raise CalcError(f"不支持的运算符：{op!r}（仅支持 + - * /）")


def main():
    print("整数计算器（输入 q 退出）")
    while True:
        try:
            a = input("第一个数：")
            if a.lower() == "q":
                break
            b = input("第二个数：")
            op = input("运算符（+ - * /）：")

            x = parse_int(a, "第一个数")
            y = parse_int(b, "第二个数")
            result = compute(x, y, op)
            print(f"  结果：{x} {op} {y} = {result}")

        except CalcError as e:
            # 业务错误：友好提示，继续循环
            print(f"  [错误] {e}")
        except KeyboardInterrupt:
            # Ctrl+C：优雅退出
            print("\n已退出。")
            sys.exit(0)
        except EOFError:
            # 标准输入被关闭：按正常退出处理
            print("\n输入结束，已退出。")
            break
        except Exception:
            # 其他未预期错误：记日志后退出循环，避免反复失败
            logging.exception("未预期的错误")
            break


if __name__ == "__main__":
    main()
```

**设计要点：**

1. **分层异常**：`parse_int` 抛 `CalcError`，`compute` 也抛 `CalcError`，调用方只需要 `except CalcError` 一处。
2. **`from e` 保留原因链**：调试时可以看到底层的 `ValueError`。
3. **三层 `except` 分工**：`CalcError` 处理业务错误，`KeyboardInterrupt` 单独做优雅退出；它不属于 `Exception`，不会被最后的兜底分支捕获。
4. **不静默吞错**：兜底分支用 `logging.exception()` 记录完整 traceback。

---

### 练习 2：带重试的网络请求模拟器

需求：模拟一个不稳定的远程服务（随机失败）；调用方最多尝试 3 次，每次失败后重试前要等待递增的时间；最终都失败时抛出包装异常。

```python
import logging
import random
import time

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s [%(levelname)s] %(message)s")


class NetworkError(Exception):
    """网络相关错误的基类"""


class TimeoutError_(NetworkError):
    """请求超时"""


class ServerError(NetworkError):
    """服务端错误"""


class RetryExhaustedError(NetworkError):
    """尝试次数耗尽"""


def unstable_request(url):
    """模拟一个不稳定的请求：60% 成功，20% 超时，20% 服务端错误"""
    r = random.random()
    if r < 0.6:
        return f"OK: {url} 的响应内容"
    elif r < 0.8:
        raise TimeoutError_(f"请求 {url} 超时")
    else:
        raise ServerError(f"{url} 返回 500")


def fetch_with_retry(url, max_attempts=3, base_delay=0.5):
    """带指数退避重试的请求"""
    last_exc = None
    for attempt in range(1, max_attempts + 1):
        try:
            logging.info("第 %d 次尝试请求 %s", attempt, url)
            return unstable_request(url)
        except NetworkError as e:
            last_exc = e
            logging.warning("第 %d 次失败：%s", attempt, e)
            if attempt < max_attempts:
                delay = base_delay * (2 ** (attempt - 1))
                logging.info("等待 %.2f 秒后重试", delay)
                time.sleep(delay)

    raise RetryExhaustedError(
        f"尝试 {max_attempts} 次后仍然失败：{url}"
    ) from last_exc


if __name__ == "__main__":
    random.seed(42)
    try:
        content = fetch_with_retry("https://example.com/api")
        print(f"成功：{content}")
    except RetryExhaustedError as e:
        logging.exception("最终失败")
```

**设计要点：**

1. **异常体系**：`NetworkError` 是基类，下面分 `TimeoutError_`、`ServerError`、`RetryExhaustedError`——调用方既可以分别处理，也可以 `except NetworkError` 一把抓。
2. **指数退避**：失败后重试前的等待时间翻倍（默认 3 次尝试会等待 0.5s → 1s），避免对故障服务造成压力。
3. **`from last_exc`**：最终抛出 `RetryExhaustedError` 时，用 `from` 保留**最后一次**失败的原始异常作为根因。

> **命名冲突提醒：** Python 内置 `TimeoutError`（注意末尾没下划线），它是 `OSError` 的子类。这里为了演示自定义异常体系，用了带下划线的 `TimeoutError_` 避免冲突。真实项目中如果你的"超时"语义就是系统级的，直接继承内置 `TimeoutError` 即可。

---

### 练习 3：批量文件处理与错误汇总

需求：对一个目录下所有文本文件做处理，单个文件失败时不能影响其他文件；最终汇总成功/失败数量并打印错误列表。

```python
import logging
from pathlib import Path

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s [%(levelname)s] %(message)s")


def count_words(path):
    """统计一个文本文件的词数"""
    with open(path, encoding="utf-8") as f:
        return len(f.read().split())


def batch_process(folder):
    folder = Path(folder)
    if not folder.is_dir():
        raise NotADirectoryError(f"{folder} 不是有效目录")

    results = []
    errors = []

    for file in folder.rglob("*.txt"):
        try:
            n = count_words(file)
        except UnicodeDecodeError:
            errors.append((file, "编码不是 UTF-8"))
            logging.exception("跳过 %s：编码错误", file)
        except PermissionError:
            errors.append((file, "没有读取权限"))
            logging.exception("跳过 %s：权限错误", file)
        except Exception as e:
            errors.append((file, f"未知错误：{e}"))
            logging.exception("跳过 %s：未知错误", file)
        else:
            results.append((file, n))

    # 汇总报告
    print(f"\n{'=' * 50}")
    print(f"  批量处理报告")
    print(f"{'=' * 50}")
    print(f"  成功：{len(results)} 个文件")
    print(f"  失败：{len(errors)} 个文件")

    if results:
        total = sum(n for _, n in results)
        print(f"  总词数：{total}")

    if errors:
        print(f"\n  失败详情：")
        for path, reason in errors:
            print(f"    - {path}：{reason}")


if __name__ == "__main__":
    sample = Path("text_demo")
    sample.mkdir(exist_ok=True)
    (sample / "ok.txt").write_text("hello world\npython exception", encoding="utf-8")
    (sample / "gbk.txt").write_bytes("这是 GBK 编码".encode("gbk"))
    (sample / "empty.txt").write_text("", encoding="utf-8")

    batch_process(sample)
```

**设计要点：**

1. **不让单个失败拖垮整体**：每个文件用独立的 `try-except` 包裹，捕获后**继续循环**。
2. **结构化错误收集**：`errors` 列表记录 `(文件, 原因)` 元组，便于最终汇总和后续处理。
3. **使用 `else` 分支**：把"成功后的统计"放在 `else` 而不是 `try` 里，避免误吞统计代码自己的异常。
4. **日志 + 报告**：日志给开发者看 traceback，控制台报告给用户看简要信息。

---

### 练习 4：用上下文管理器保证事务原子性

需求：模拟一个简易的转账操作。如果转账过程中出错，必须回滚到操作前的状态（不能扣了 A 的钱却没加到 B 上）。用上下文管理器实现这个"事务"语义。

```python
from contextlib import contextmanager
import copy


class TransferError(Exception):
    """转账失败"""


@contextmanager
def transaction(accounts):
    """事务上下文：失败时自动回滚"""
    snapshot = copy.deepcopy(accounts)        # 进入时存档
    try:
        yield accounts                        # 进入 with 块
    except BaseException:
        accounts.clear()                      # 失败：恢复存档
        accounts.update(snapshot)
        raise                                 # 清理后必须继续抛出
    # 没异常：什么都不做，新状态保留


def transfer(accounts, src, dst, amount, *, fail_after_debit=False):
    if amount <= 0:
        raise TransferError(f"金额必须 > 0，收到 {amount}")
    if src not in accounts:
        raise TransferError(f"源账户不存在：{src}")
    if dst not in accounts:
        raise TransferError(f"目标账户不存在：{dst}")
    if accounts[src] < amount:
        raise TransferError(
            f"{src} 余额 {accounts[src]} 不足以支付 {amount}"
        )

    accounts[src] -= amount
    # 假设这里需要做一些跨网络的远程操作
    # 如果中间失败，src 已经被扣款但 dst 还没加上
    if fail_after_debit:
        raise TransferError("远程入账服务失败")
    accounts[dst] += amount


def demo():
    accounts = {"Alice": 100, "Bob": 50}

    print(f"初始：{accounts}")

    # 成功的转账
    with transaction(accounts):
        transfer(accounts, "Alice", "Bob", 30)
    print(f"转账成功后：{accounts}")

    # 失败的转账：扣款后远程入账失败，应当回滚
    try:
        with transaction(accounts):
            transfer(accounts, "Alice", "Bob", 20, fail_after_debit=True)
    except TransferError as e:
        print(f"转账失败：{e}")
    print(f"失败回滚后：{accounts}")


if __name__ == "__main__":
    demo()
```

输出：

```
初始：{'Alice': 100, 'Bob': 50}
转账成功后：{'Alice': 70, 'Bob': 80}
转账失败：远程入账服务失败
失败回滚后：{'Alice': 70, 'Bob': 80}
```

**设计要点：**

1. **`@contextmanager` 的标准结构**：进入时做准备，`yield` 让 `with` 块运行，用 `try-except` 包裹 `yield` 处理失败。
2. **`raise` 重新抛出**：上下文管理器只负责回滚，不吞错——让调用方还能 `except TransferError` 处理业务。
3. **少数可捕获 `BaseException` 的场景**：这里为了回滚才捕获 `BaseException`，但清理后必须立刻重新抛出，不能吞掉 `KeyboardInterrupt`、`SystemExit` 这类控制流信号。
4. **拷贝再恢复的模式**：适合内存中的小数据结构。真实数据库事务用的是 WAL/日志机制，但**思想完全一样**：失败时回到操作前的状态。

---

### 练习 5：自定义异常体系 + CLI 工具演示版

把前面所有要点综合起来，做一个简易的"配置文件检查器"：读取 JSON 配置，验证必需字段，错误发生时保留异常链，并在需要时显示底层原因或写入日志。

```python
import json
import logging
import sys
from pathlib import Path

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s [%(levelname)s] %(message)s")


# --- 异常体系 ---

class ConfigError(Exception):
    """配置相关错误的基类"""


class ConfigNotFoundError(ConfigError):
    """配置文件不存在"""


class ConfigParseError(ConfigError):
    """配置文件无法解析"""


class ConfigValidationError(ConfigError):
    """配置内容不符合要求"""

    def __init__(self, field, reason):
        self.field = field
        self.reason = reason
        super().__init__(f"字段 '{field}' 验证失败：{reason}")


# --- 业务逻辑 ---

REQUIRED_FIELDS = {
    "name": str,
    "port": int,
    "debug": bool,
}


def load_config(path):
    """加载并验证配置文件"""
    p = Path(path)

    # 第一步：读取
    try:
        text = p.read_text(encoding="utf-8")
    except FileNotFoundError as e:
        raise ConfigNotFoundError(f"配置文件不存在：{path}") from e
    except OSError as e:
        raise ConfigError(f"无法读取配置文件 {path}") from e

    # 第二步：解析
    try:
        data = json.loads(text)
    except json.JSONDecodeError as e:
        raise ConfigParseError(f"配置文件 JSON 解析失败：{path}") from e

    # 第三步：验证
    if not isinstance(data, dict):
        raise ConfigValidationError("(根)", "配置必须是一个 JSON 对象")

    for field, expected_type in REQUIRED_FIELDS.items():
        if field not in data:
            raise ConfigValidationError(field, "字段缺失")
        # 用精确类型检查，避免 bool 被 isinstance(..., int) 当作 int。
        if type(data[field]) is not expected_type:
            raise ConfigValidationError(
                field,
                f"类型应为 {expected_type.__name__}，"
                f"实际为 {type(data[field]).__name__}"
            )

    # 业务校验
    if not (1 <= data["port"] <= 65535):
        raise ConfigValidationError("port", f"必须在 1~65535 之间，收到 {data['port']}")

    return data


# --- 入口 ---

def main():
    if len(sys.argv) != 2:
        print("用法：python config_check.py <config.json>")
        sys.exit(2)

    path = sys.argv[1]

    try:
        config = load_config(path)
    except ConfigNotFoundError as e:
        print(f"[错误] {e}")
        sys.exit(1)
    except ConfigParseError as e:
        print(f"[错误] {e}")
        print(f"  原因：{e.__cause__}")
        sys.exit(1)
    except ConfigValidationError as e:
        print(f"[错误] {e}")
        print(f"  问题字段：{e.field}")
        sys.exit(1)
    except ConfigError:
        logging.exception("加载配置失败")
        sys.exit(1)

    print(f"✅ 配置加载成功：{config}")


if __name__ == "__main__":
    # 真实 CLI 脚本中通常只需要 main()。
    # 下面为了教程演示，临时创建样例并多次调用 main()。
    Path("good.json").write_text(
        json.dumps({"name": "my-app", "port": 8080, "debug": True}),
        encoding="utf-8",
    )
    Path("bad_json.json").write_text("{this is not json", encoding="utf-8")
    Path("bad_field.json").write_text(
        json.dumps({"name": "x", "port": 99999, "debug": True}),
        encoding="utf-8",
    )

    for name in ["good.json", "missing.json", "bad_json.json", "bad_field.json"]:
        print(f"\n--- 测试 {name} ---")
        sys.argv = ["config_check.py", name]
        try:
            main()
        except SystemExit as e:
            print(f"  退出码：{e.code}")
```

**设计要点：**

1. **三层异常体系**：`ConfigError`（基类）→ 三个具体子类，调用方可以根据需要选择粒度。
2. **读取和解析时用 `from e`**：原始的 `FileNotFoundError`、`JSONDecodeError` 都被保留为 `__cause__`，调试时能看到根因。
3. **`ConfigValidationError` 携带结构化字段**：不止有消息字符串，还有 `field` 属性，调用方可以直接拿去做 UI 高亮。
4. **入口的退出码**：`sys.exit(1)` 表示失败、`sys.exit(2)` 表示用法错误，符合 Unix 惯例。
5. **`except` 的层级和顺序**：从最具体到最通用，最后一层 `except ConfigError` 用 `logging.exception()` 兜底。

---

## 附录 A：Python 3.11+ 的异常组（ExceptionGroup）

到目前为止，所有的 `raise` 一次只抛**一个**异常。但有些场景天然会产生**多个并发异常**——比如批量处理时多个任务同时失败、`asyncio.TaskGroup` 中多个协程同时报错。

Python 3.11 引入了 **`ExceptionGroup`**，用来"打包"多个异常一起抛出，并配套引入 **`except*`** 语法来处理它们。

### 抛出异常组

```python
errors = [
    ValueError("第 1 个文件解析失败"),
    PermissionError("第 2 个文件没权限"),
    ValueError("第 3 个文件格式错误"),
]

raise ExceptionGroup("批量处理失败", errors)
```

Traceback 会以类似下面的形式显示所有打包的异常：

```
  + Exception Group Traceback (most recent call last):
  | ExceptionGroup: 批量处理失败 (3 sub-exceptions)
  +-+---------------- 1 ----------------
    | ValueError: 第 1 个文件解析失败
    +---------------- 2 ----------------
    | PermissionError: 第 2 个文件没权限
    +---------------- 3 ----------------
    | ValueError: 第 3 个文件格式错误
    +------------------------------------
```

### 用 `except*` 按类型分别处理

```python
def process_batch():
    errors = [
        ValueError("解析错误 A"),
        PermissionError("权限错误"),
        ValueError("解析错误 B"),
    ]
    raise ExceptionGroup("批量失败", errors)


try:
    process_batch()
except* ValueError as eg:
    print(f"捕获到 {len(eg.exceptions)} 个 ValueError")
    for e in eg.exceptions:
        print(f"  - {e}")
except* PermissionError as eg:
    print(f"捕获到 {len(eg.exceptions)} 个 PermissionError")
    for e in eg.exceptions:
        print(f"  - {e}")
```

输出：

```
捕获到 2 个 ValueError
  - 解析错误 A
  - 解析错误 B
捕获到 1 个 PermissionError
  - 权限错误
```

**`except*` 与 `except` 的关键区别：**

| 特性             | `except`                       | `except*`                              |
| ---------------- | ------------------------------ | -------------------------------------- |
| 适用于           | 普通异常                       | 主要用于 `ExceptionGroup`；匹配普通异常时会以单元素异常组形式处理 |
| 匹配后是否继续   | 只匹配第一个 `except` 分支     | 多个 `except*` 分支可能在一次 `try` 中执行；每个分支只处理前面未处理的匹配子组 |
| 捕获到的变量是   | 异常本身                       | 包含同类异常的 `ExceptionGroup` 子集   |

> **新手建议：** 日常代码用不到异常组——它主要服务于 `asyncio.TaskGroup` 等并发场景。先把前面 5 章的基础打牢，遇到并发再回来看这一节。

---

## 附录 B：常见反模式速查

下面列出 8 个**最常见、最危险**的异常处理反模式。每条都给出错例、原因和修正方式。

### 反模式 1：裸 except

```python
# ❌
try:
    do_something()
except:
    pass
```

**问题：** 捕获了 `BaseException`，包括 `KeyboardInterrupt`、`SystemExit`——Ctrl+C 失效。

**修正：** 至少写 `except Exception`，最好写具体类型。

---

### 反模式 2：静默吞错

```python
# ❌
try:
    risky()
except Exception:
    pass
```

**问题：** 错误被完全隐藏，将来排查问题时毫无线索。

**修正：** 至少 `logging.exception("操作失败")`，让错误留下痕迹。

---

### 反模式 3：捕获范围过大

```python
# ❌
try:
    x = int(text)
    result = expensive_calc(x)         # 这一行的错误也被吞了
    save_to_db(result)                  # 这一行的错误也被吞了
except ValueError:
    print("解析失败")
```

**问题：** `expensive_calc` 或 `save_to_db` 内部如果抛了 `ValueError`，会被误判为"解析失败"。

**修正：** 只把可能抛 `ValueError` 的那一行放进 `try`：

```python
# ✅
def handle(text):
    try:
        x = int(text)
    except ValueError:
        print("解析失败")
        return
    result = expensive_calc(x)
    save_to_db(result)
```

---

### 反模式 4：异常用作流程控制

```python
# ❌
try:
    value = my_dict[key]
except KeyError:
    value = "default"
```

**问题：** 这个场景有更直接的默认值 API，用异常会让意图不如 `dict.get()` 清楚。

**修正：** 用 `dict.get()`：

```python
# ✅
value = my_dict.get(key, "default")
```

> **例外：** 如果"键不存在"在你的业务中确实是异常情况（不应该发生），那 `KeyError` 是合理的。判断标准：**这件事是否应该被监控告警？**

---

### 反模式 5：把所有错误转成 None

```python
# ❌
def get_user(user_id):
    try:
        return db.query(user_id)
    except Exception:
        return None
```

**问题：** 调用方不知道是"查不到"还是"数据库挂了"——这两种情况应该有完全不同的处理方式。

**修正：** 让"查不到"返回 `None`，让"数据库挂了"抛异常：

```python
# ✅
def get_user(user_id):
    return db.query(user_id)            # 不存在时 query 返回 None，挂了时抛异常
```

---

### 反模式 6：没有显式保留原因链

```python
# ❌
try:
    config = load(path)
except FileNotFoundError:
    raise ConfigError("配置加载失败")    # 原异常只会作为隐式上下文保留
```

**问题：** Python 3 会把原异常作为 `__context__` 隐式保留，但没有明确标注"新异常由原异常直接导致"，因果关系不如 `from e` 清楚。

**修正：** 用 `from`：

```python
# ✅
try:
    config = load(path)
except FileNotFoundError as e:
    raise ConfigError("配置加载失败") from e
```

---

### 反模式 7：在 finally 中抛异常

```python
# ❌
try:
    do_something()
finally:
    cleanup()                           # 如果这里抛异常，会覆盖原始异常
```

**问题：** `finally` 中如果抛新异常，**会替换** `try` 中的原始异常（Python 3 中原异常会被保留为 `__context__`，但顶层显示的还是新异常）。

**修正：** `finally` 里的清理代码也用 `try-except` 保护：

```python
# ✅
try:
    do_something()
finally:
    try:
        cleanup()
    except Exception:
        logging.exception("清理失败")
```

---

### 反模式 8：用 assert 做生产环境的校验

```python
# ❌
def handle_request(user_input):
    assert user_input is not None
    assert len(user_input) > 0
    ...
```

**问题：** `python -O` 启动时所有 `assert` 都被忽略，校验失效，恶意输入可能直接进入业务逻辑。

**修正：** 用 `raise`：

```python
# ✅
def handle_request(user_input):
    if user_input is None:
        raise ValueError("user_input 不能为 None")
    if not user_input:
        raise ValueError("user_input 不能为空")
    ...
```

---

## 附录 C：异常调试技巧

实际开发中，遇到异常时不一定能"一眼看穿"。下面是一些实用的调试招式。

### 技巧 1：进入异常发生时的现场

Python 内置了 `pdb`（Python Debugger），出异常时可以直接跳进交互模式：

```bash
python -m pdb your_script.py
```

或者在代码里手动设断点（Python 3.7+）：

```python
def divide(a, b):
    breakpoint()                # 程序运行到这里会暂停，进入 pdb
    return a / b
```

**`pdb` 常用命令：**

| 命令          | 含义                       |
| ------------- | -------------------------- |
| `n` (next)    | 执行下一行                 |
| `s` (step)    | 进入函数内部               |
| `c` (continue)| 继续运行到下一个断点       |
| `p 变量名`     | 打印变量值                 |
| `l` (list)    | 显示当前位置附近的代码     |
| `q` (quit)    | 退出调试                   |

### 技巧 2：用 `traceback.print_exc()` 打印当前异常

在 `except` 里随时打印完整 traceback：

```python
import traceback

try:
    risky()
except Exception:
    traceback.print_exc()        # 输出到 stderr
    # 或者
    info = traceback.format_exc()  # 返回字符串，方便存入日志
    save_to_log(info)
```

### 技巧 3：用 `sys.exc_info()` 获取异常信息

```python
import sys

try:
    risky()
except Exception:
    exc_type, exc_value, exc_tb = sys.exc_info()
    print(f"类型：{exc_type.__name__}")
    print(f"消息：{exc_value}")
```

### 技巧 4：自定义顶层异常钩子

让未捕获的异常自动发到日志/监控系统：

```python
import sys
import logging

logging.basicConfig(level=logging.INFO)


def global_excepthook(exc_type, exc_value, exc_tb):
    if issubclass(exc_type, KeyboardInterrupt):
        # Ctrl+C 不当作错误处理
        sys.__excepthook__(exc_type, exc_value, exc_tb)
        return
    logging.critical("未捕获的异常",
                     exc_info=(exc_type, exc_value, exc_tb))


sys.excepthook = global_excepthook


# 测试：抛一个不捕获的异常
raise RuntimeError("演示用未捕获异常")
```

这样，即使你忘了写 `try-except`，崩溃也会被记录到日志，便于事后追查。生产服务推荐配合 Sentry、Rollbar 等错误监控平台使用。

### 技巧 5：异常对象的 `__cause__` 和 `__context__`

```python
try:
    try:
        1 / 0
    except ZeroDivisionError as e:
        raise ValueError("外层错误") from e
except ValueError as e:
    print(f"当前异常：{e}")
    print(f"__cause__（显式 from）：{e.__cause__}")
    print(f"__context__（隐式上下文）：{e.__context__}")
```

输出：

```
当前异常：外层错误
__cause__（显式 from）：division by zero
__context__（隐式上下文）：division by zero
```

| 属性             | 含义                                                |
| ---------------- | --------------------------------------------------- |
| `__cause__`      | 用 `raise ... from X` 时设置的"显式原因"，否则为 None |
| `__context__`    | 在 `except` 块中重新抛异常时自动设置的"隐式上下文"  |
| `__suppress_context__` | 用 `from None` 时为 True，traceback 隐藏 `__context__` |

调试复杂异常链时，沿着 `__cause__` 一路向下追，就能找到最深的根因。

---

## 总结与进阶方向

### 本教程覆盖的核心知识

| 章节                   | 核心技能                                                                     |
| ---------------------- | ---------------------------------------------------------------------------- |
| 异常基础               | 常见异常类型、`try-except`、`except ... as e`                                |
| 完整结构               | `else`、`finally`、捕获顺序、`Exception` vs `BaseException`                  |
| 主动抛出               | `raise`、自定义异常、传播机制、`raise ... from ...` 异常链                   |
| 调试与日志             | `traceback` 阅读、`assert` 断言、`logging.exception()`                       |
| 工程化                 | 文件异常、`with` 与上下文管理器、异常体系设计原则                            |
| 综合练习               | 计算器、重试器、批处理、事务、CLI 配置检查器                                 |
| 附录                   | 异常组（3.11+）、反模式速查、调试技巧                                        |

### 关键最佳实践回顾

1. **`except` 要写明类型**——优先 `except Exception`，避免裸 `except:` 或 `except BaseException`。
2. **子类在前，父类在后**——多个 `except` 时严格遵守。
3. **`finally` 用来释放资源**——文件、连接、锁；现代代码更常用 `with` 替代。
4. **`with` 是异常安全的**——只要资源支持上下文管理器，就用 `with`。
5. **`raise ... from ...` 保留原因链**——包装底层异常时不要丢信息。
6. **`assert` 不是用户校验工具**——它可能被 `-O` 关闭，校验输入用 `raise ValueError`。
7. **`logging.exception()` 记录 traceback**——生产环境永远不要只 `print`。
8. **不要静默吞错**——至少记录日志，宁可不处理也不要伪装成成功。

### 进阶方向

掌握本教程后，推荐继续学习：

- **`contextlib` 模块**：`contextmanager`、`suppress`、`ExitStack` 等高级上下文工具
- **`warnings` 模块**：区别于异常的"警告"机制
- **`sys.excepthook`**：定制顶层异常处理器，把崩溃信息发到监控系统
- **`logging` 进阶**：`Handler`、`Formatter`、`Filter`、按级别和模块分发日志
- **异常与并发**：`threading`、`asyncio`、`concurrent.futures` 里的异常传播规则
- **测试中的异常**：`pytest.raises`、`unittest.TestCase.assertRaises`
