# Python 文件操作完全教程

> 本篇重点讲**文件读写、二进制处理和目录访问**。如果你的目标是系统理解乱码、UTF-8、GBK、换行符等问题，可以把本篇与《Python 文件操作与文本编码》配合阅读。

## 学习路线

| 阶段 | 学什么 | 解决什么问题 |
| ---- | ------ | ------------ |
| 第一章 | 文本文件读写 | 读取 `.txt`、写入结果、逐行处理文本 |
| 第二章 | 二进制文件读写 | 复制图片、音频、压缩包等非文本文件 |
| 第三章 | 目录访问 | 查找文件、遍历文件夹、处理路径 |
| 第四章 | 综合练习 | 把文件读写、目录扫描和安全操作组合起来 |

---

## 第一章：文本文件的读写基础

### 1.1 文件与路径的基本概念

在编程世界中，**文件**是存储在磁盘上的数据集合。你电脑里的每一张图片、每一首歌、每一个文档，本质上都是文件。Python 作为一门强大的编程语言，提供了非常简洁的方式来操作这些文件。

#### 什么是文件？

从操作系统的角度看，文件可以分为两大类：

| 类型           | 说明                                           | 常见格式                                |
| -------------- | ---------------------------------------------- | --------------------------------------- |
| **文本文件**   | 内容由人类可读的字符组成，可以用记事本直接打开 | `.txt`、`.py`、`.csv`、`.json`、`.html` |
| **二进制文件** | 内容由原始字节组成，用记事本打开会显示乱码     | `.jpg`、`.mp3`、`.exe`、`.zip`、`.pdf`  |

本章我们先聚焦于**文本文件**的操作。

#### 什么是路径？

路径（Path）就是文件在计算机中的"地址"。就像你告诉快递员"XX市XX区XX路XX号"一样，路径告诉 Python "去哪里找这个文件"。

```
Windows 路径示例：C:\Users\zhangsan\Desktop\hello.txt
Mac/Linux 路径示例：/home/zhangsan/Desktop/hello.txt
```

> **重要提示：** Windows 路径使用反斜杠 `\`，但在 Python 字符串中，`\` 是转义字符的前缀（例如 `\n` 表示换行）。因此在 Python 中书写 Windows 路径时，你有三种方式避免冲突：

```python
# 方式一：使用原始字符串（推荐）
path = r"C:\Users\zhangsan\Desktop\hello.txt"

# 方式二：使用双反斜杠
path = "C:\\Users\\zhangsan\\Desktop\\hello.txt"

# 方式三：使用正斜杠（Python 在 Windows 上也能识别）
path = "C:/Users/zhangsan/Desktop/hello.txt"
```

三种方式效果完全一样，推荐使用**方式一**或**方式三**，写起来最简洁。

---

### 1.2 用 `open()` 打开文件

`open()` 是 Python 内置的文件操作核心函数，用来打开一个文件并返回一个**文件对象**。后续的读写操作都通过这个文件对象来完成。

#### 基本语法

```python
file_object = open(file, mode='r', encoding=None)
```

| 参数       | 说明                                           |
| ---------- | ---------------------------------------------- |
| `file`     | 文件路径（字符串），可以是相对路径或绝对路径   |
| `mode`     | 打开模式，默认为 `'r'`（只读）                 |
| `encoding` | 编码方式，读写文本文件时建议显式指定 `'utf-8'` |

#### 常用打开模式（文本模式）

| 模式   | 含义           | 文件不存在时               | 文件存在时             |
| ------ | -------------- | -------------------------- | ---------------------- |
| `'r'`  | 只读（Read）   | ❌ 报错 `FileNotFoundError` | 从头读取               |
| `'w'`  | 只写（Write）  | ✅ 自动创建                 | ⚠️ **清空**原内容后写入 |
| `'a'`  | 追加（Append） | ✅ 自动创建                 | 在末尾追加写入         |
| `'r+'` | 读写           | ❌ 报错                     | 可读可写               |

#### 第一个例子：打开并读取文件

假设你的**当前工作目录**下有一个 `hello.txt` 文件，内容是：

```
你好，世界！
Hello, Python!
文件操作入门。
```

现在用 Python 来读取它。这里的 `"hello.txt"` 是相对路径，表示从程序运行时的当前工作目录中查找这个文件：

```python
# 打开文件
f = open("hello.txt", mode="r", encoding="utf-8")

# 读取全部内容
content = f.read()

# 打印内容
print(content)

# 关闭文件（非常重要！）
f.close()
```

输出：

```
你好，世界！
Hello, Python!
文件操作入门。
```

#### 为什么必须关闭文件？

打开文件时，操作系统会为你分配资源（文件句柄）。如果你只开不关，就像你借了图书馆的书却从不还——资源会被占用，数量有限的文件句柄可能耗尽，导致程序出错。

**核心原则：打开文件后，必须确保关闭。** 稍后我们会学到 `with` 语句，它能自动帮你关闭文件。

#### 编码问题详解

**编码（Encoding）** 是字符与字节之间的映射规则。同一个汉字"你"，在不同编码下对应不同的字节序列：

| 编码  | "你" 对应的字节          |
| ----- | ------------------------ |
| UTF-8 | `\xe4\xbd\xa0`（3 字节） |
| GBK   | `\xc4\xe3`（2 字节）     |

如果文件用 GBK 保存，但你用 UTF-8 去读，就会出现**乱码**或报错：

```python
# 错误示范：编码不匹配
f = open("gbk_file.txt", encoding="utf-8")  # 文件实际是 GBK 编码
content = f.read()  # 可能抛出 UnicodeDecodeError
```

**解决方案：**

```python
# 正确做法：指定正确的编码
f = open("gbk_file.txt", encoding="gbk")
content = f.read()
print(content)
f.close()
```

> **建议：** 现代项目中，统一使用 `UTF-8` 编码是最佳实践。创建新文件时，始终指定 `encoding="utf-8"`。

---

### 1.3 三种读取方法详解

Python 提供了三种读取文件内容的方法，各有适用场景。

#### 1.3.1 `read()` —— 一次性读取全部内容

```python
f = open("hello.txt", encoding="utf-8")
content = f.read()       # 读取全部内容，返回一个字符串
print(content)
print(type(content))     # <class 'str'>
f.close()
```

`read()` 还可以指定读取的**字符数**：

```python
f = open("hello.txt", encoding="utf-8")
part = f.read(5)         # 只读取前 5 个字符
print(part)              # 输出：你好，世界
print(f.read(10))        # 继续从第 6 个字符开始，读取 10 个字符
f.close()
```

> **注意：** 每次调用 `read()` 后，文件内部的"光标"（称为**文件指针**）会向后移动。第二次调用会从上次停下的位置继续读取，不会回到开头。

**适用场景：** 文件较小（几 KB 到几 MB）时，一次性读入内存处理非常方便。

**⚠️ 风险：** 如果文件特别大（例如几 GB 的日志文件），`read()` 会一次性把所有内容加载到内存，可能导致内存溢出（MemoryError）。

#### 1.3.2 `readline()` —— 逐行读取

每调用一次 `readline()`，读取文件中的**一行**内容（包含行尾的换行符 `\n`）：

```python
f = open("hello.txt", encoding="utf-8")

line1 = f.readline()
print(repr(line1))   # '你好，世界！\n'  ← 注意末尾有 \n

line2 = f.readline()
print(repr(line2))   # 'Hello, Python!\n'

line3 = f.readline()
print(repr(line3))   # '文件操作入门。\n'

line4 = f.readline()
print(repr(line4))   # ''  ← 空字符串，说明已经读到文件末尾

f.close()
```

> **知识点：** `repr()` 函数会显示字符串的"原始面貌"，包括 `\n` 等不可见字符。调试时非常有用。

**去掉行尾换行符：**

```python
with open("hello.txt", encoding="utf-8") as f:
    line = f.readline().strip()       # strip() 去除首尾空白字符（包括 \n）

with open("hello.txt", encoding="utf-8") as f:
    line = f.readline().rstrip("\n")  # rstrip() 只去除右侧的换行符
```

**用循环逐行读取整个文件：**

```python
f = open("hello.txt", encoding="utf-8")

while True:
    line = f.readline()
    if not line:        # 读到空字符串说明到了末尾
        break
    print(line.strip())

f.close()
```

**适用场景：** 需要逐行处理的大文件，每次只在内存中保留一行。

#### 1.3.3 `readlines()` —— 一次读取所有行，返回列表

```python
f = open("hello.txt", encoding="utf-8")
lines = f.readlines()    # 返回一个列表，每个元素是一行内容
print(lines)
# ['你好，世界！\n', 'Hello, Python!\n', '文件操作入门。\n']
print(type(lines))       # <class 'list'>
f.close()
```

可以配合循环处理每一行：

```python
f = open("hello.txt", encoding="utf-8")
lines = f.readlines()

for i, line in enumerate(lines, start=1):
    print(f"第{i}行: {line.strip()}")

f.close()
```

输出：

```
第1行: 你好，世界！
第2行: Hello, Python!
第3行: 文件操作入门。
```

#### 三种方法对比总结

| 方法          | 返回类型      | 内存占用       | 适用场景             |
| ------------- | ------------- | -------------- | -------------------- |
| `read()`      | `str`         | 高（全部加载） | 小文件，需要整体处理 |
| `readline()`  | `str`（一行） | 低（一行）     | 大文件，逐行处理     |
| `readlines()` | `list[str]`   | 高（全部加载） | 需要按行索引访问     |

---

### 1.4 `with open()` —— 自动关闭文件（最佳实践）

手动调用 `close()` 有一个风险：如果在 `open()` 和 `close()` 之间的代码出错了，`close()` 就不会执行，文件资源就泄露了。

```python
# 危险的写法
f = open("hello.txt", encoding="utf-8")
content = f.read()
result = 1 / 0           # 这里抛出异常 ZeroDivisionError
f.close()                 # ← 这行永远不会执行！文件没有被关闭！
```

Python 提供了 `with` 语句（上下文管理器）来优雅地解决这个问题：

```python
with open("hello.txt", encoding="utf-8") as f:
    content = f.read()
    print(content)
# 离开 with 代码块后，文件自动关闭，无论是否发生异常
```

**`with` 语句的工作原理：**

1. 执行 `open()`，将返回的文件对象赋给 `f`
2. 执行 `with` 缩进块中的代码
3. 无论代码是正常结束还是抛出异常，都会**自动调用 `f.close()`**

> **从现在起，请始终使用 `with open()` 的写法。** 这是 Python 社区的标准做法，也是面试和工作中的基本要求。

#### 同时打开多个文件

```python
# 同时读取两个文件
with open("file1.txt", encoding="utf-8") as f1, \
     open("file2.txt", encoding="utf-8") as f2:
    content1 = f1.read()
    content2 = f2.read()
    print(content1)
    print(content2)
```

Python 3.10+ 还支持用括号包裹，更清晰：

```python
with (
    open("file1.txt", encoding="utf-8") as f1,
    open("file2.txt", encoding="utf-8") as f2,
):
    content1 = f1.read()
    content2 = f2.read()
```

---

### 1.5 遍历文件的最佳方式：直接迭代文件对象

文件对象本身是**可迭代的**，你可以直接用 `for` 循环逐行遍历，这是处理大文件最推荐的方式：

```python
with open("hello.txt", encoding="utf-8") as f:
    for line in f:                   # 每次迭代自动读取一行
        print(line.strip())
```

这种方式的优势：

- **内存友好：** 每次只加载一行到内存，通常更适合处理大文件
- **代码简洁：** 比 `while + readline()` 更优雅
- **自动停止：** 读到文件末尾时循环自动结束

---

### 1.6 文件写入基础

读文件和写文件通常是一起出现的。掌握读取方法之后，还需要知道如何创建练习文件、保存处理结果。

#### 使用 `'w'` 模式写入（覆盖写入）

```python
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("第一行内容\n")
    f.write("第二行内容\n")
    f.write("第三行内容\n")
```

> **⚠️ 注意：** `'w'` 模式会**清空**文件原有内容！如果文件不存在则自动创建。

#### 使用 `'a'` 模式追加写入

```python
with open("output.txt", "a", encoding="utf-8") as f:
    f.write("追加的新内容\n")
```

`'a'` 模式会在文件末尾追加，不会清空原有内容。

#### 使用 `writelines()` 写入多行

```python
lines = ["苹果\n", "香蕉\n", "橘子\n"]

with open("fruits.txt", "w", encoding="utf-8") as f:
    f.writelines(lines)    # 将列表中的字符串依次写入
```

> **注意：** `writelines()` 不会自动添加换行符，你需要在每个字符串末尾自己加 `\n`。

---

### 1.7 文件指针与 `seek()`

文件对象内部维护着一个**指针**（cursor），记录当前读写的位置。

```python
with open("hello.txt", encoding="utf-8") as f:
    print(f.tell())      # 0  ← 刚打开时，指针在文件开头位置
    
    f.read(3)            # 读取 3 个字符
    print(f.tell())      # 返回当前位置标记，通常会反映底层字节位置
    
    f.seek(0)            # 将指针移回文件开头
    print(f.tell())      # 0
    
    content = f.read()   # 从头重新读取全部内容
    print(content)
```

| 方法                     | 说明                                           |
| ------------------------ | ---------------------------------------------- |
| `f.tell()`               | 返回当前位置标记，可传给 `seek()` 用来回到该处 |
| `f.seek(0)`              | 将指针移到文件开头                             |
| `f.seek(offset, whence)` | 移动指针到指定位置（高级用法）                 |

> **提示：** 在二进制模式下，`tell()` / `seek()` 使用字节位置；在文本模式下，`tell()` 返回的是一个可交给 `seek()` 的位置标记，不应简单理解为“已读字符数”。文本模式中最常用、最安全的操作是 `seek(0)` 回到文件开头。

---

### 1.8 异常处理与文件操作

文件操作是最容易出错的场景之一：文件可能不存在、没有读取权限、磁盘满了……你应该学会用 `try-except` 来捕获这些错误。

```python
try:
    with open("不存在的文件.txt", encoding="utf-8") as f:
        content = f.read()
except FileNotFoundError:
    print("错误：文件不存在，请检查路径是否正确")
except PermissionError:
    print("错误：没有权限读取该文件")
except UnicodeDecodeError:
    print("错误：文件编码不匹配，请尝试其他编码")
except Exception as e:
    print(f"发生了未知错误：{e}")
```

常见的文件相关异常：

| 异常类型             | 触发场景                     |
| -------------------- | ---------------------------- |
| `FileNotFoundError`  | 文件路径不存在               |
| `PermissionError`    | 没有读写权限                 |
| `UnicodeDecodeError` | 编码不匹配                   |
| `IsADirectoryError`  | 试图把目录当文件打开         |
| `OSError`            | 磁盘满、路径非法等系统级错误 |

---

### 1.9 实战练习

#### 练习 1：读取文件并统计行数

```python
def count_lines(filepath):
    """统计文件行数"""
    total = 0
    non_empty = 0

    with open(filepath, encoding="utf-8") as f:
        for line in f:
            total += 1
            if line.strip():      # 非空行
                non_empty += 1
    
    print(f"文件：{filepath}")
    print(f"总行数：{total}")
    print(f"非空行数：{non_empty}")
    print(f"空行数：{total - non_empty}")


# 先创建测试文件
with open("test_lines.txt", "w", encoding="utf-8") as f:
    f.write("第一行\n")
    f.write("\n")                # 空行
    f.write("第三行\n")
    f.write("   \n")            # 只有空格的行
    f.write("第五行\n")

# 统计
count_lines("test_lines.txt")
```

输出：

```
文件：test_lines.txt
总行数：5
非空行数：3
空行数：2
```

#### 练习 2：在文件中查找关键词

```python
def search_keyword(filepath, keyword):
    """在文件中查找包含关键词的行"""
    results = []
    
    with open(filepath, encoding="utf-8") as f:
        for line_num, line in enumerate(f, start=1):
            if keyword in line:
                results.append((line_num, line.strip()))
    
    if results:
        print(f"在 '{filepath}' 中找到 {len(results)} 处 '{keyword}'：")
        for num, content in results:
            print(f"  第{num}行: {content}")
    else:
        print(f"在 '{filepath}' 中未找到 '{keyword}'")
    
    return results


# 创建测试文件
with open("article.txt", "w", encoding="utf-8") as f:
    f.write("Python 是一门优雅的编程语言。\n")
    f.write("它广泛应用于数据分析和人工智能。\n")
    f.write("学好 Python 可以提高工作效率。\n")
    f.write("Java 也是一门流行的编程语言。\n")
    f.write("但 Python 的语法更加简洁。\n")

# 搜索
search_keyword("article.txt", "Python")
```

输出：

```
在 'article.txt' 中找到 3 处 'Python'：
  第1行: Python 是一门优雅的编程语言。
  第3行: 学好 Python 可以提高工作效率。
  第5行: 但 Python 的语法更加简洁。
```

#### 练习 3：统计文件中非空白字符的出现频率

```python
from collections import Counter

def char_frequency(filepath, top_n=10):
    """统计文件中字符出现频率（忽略空白字符）"""
    with open(filepath, encoding="utf-8") as f:
        text = f.read()
    
    # 过滤掉空白字符
    chars = [c for c in text if not c.isspace()]
    counter = Counter(chars)
    
    print(f"文件总字符数（不含空白）：{len(chars)}")
    print(f"不同字符数：{len(counter)}")
    print(f"\n出现频率最高的 {top_n} 个字符：")
    for char, count in counter.most_common(top_n):
        print(f"  '{char}' → {count} 次")

# 使用上面创建的 article.txt
char_frequency("article.txt")
```

---

## 第二章：二进制文件的读写

### 2.1 为什么需要二进制模式？

上一章中，我们使用 `open("file.txt", "r")` 读取文件，这是**文本模式**。Python 在文本模式下会自动进行编码/解码（字节 ↔ 字符串），并自动处理换行符差异（Windows 的 `\r\n` 会被统一转换为 `\n`）。

但如果你试图用文本模式打开一张图片：

```python
# ❌ 错误用法
with open("photo.jpg", "r", encoding="utf-8") as f:
    data = f.read()
# UnicodeDecodeError: 图片的字节不是合法的 UTF-8 字符序列
```

图片、音频、视频等文件的内容不是"字符"，而是**原始字节数据**。你不能把它们当文本去"翻译"，而应该原封不动地按字节来读写——这就是**二进制模式**的用途。

### 2.2 二进制模式的打开方式

在模式字符串中加上 `b` 即进入二进制模式：

- **`'rb'`**（Read Binary）—— 二进制只读
- **`'wb'`**（Write Binary）—— 二进制只写（清空原内容）
- **`'ab'`**（Append Binary）—— 二进制追加
- **`'rb+'`** —— 二进制读写

另外还有 `wb+`、`ab+`、`xb` 等模式。其中 `xb` 表示**独占创建**：文件已存在时会报错，适合用来避免误覆盖。

> **重要区别：** 二进制模式下，`open()` 函数**不需要也不能指定 `encoding` 参数**。因为二进制模式直接操作字节，不涉及字符编码。

```python
# 文本模式：读写的是 str（字符串）
with open("hello.txt", "r", encoding="utf-8") as f:
    text = f.read()
    print(type(text))    # <class 'str'>

# 二进制模式：读写的是 bytes（字节序列）
with open("hello.txt", "rb") as f:
    raw = f.read()
    print(type(raw))     # <class 'bytes'>
    print(raw)           # b'\xe4\xbd\xa0...'  bytes 的转义表示
    print(raw.hex())     # e4bda0...  真正的十六进制字符串
```

### 2.3 `bytes` 数据类型详解

`bytes` 是 Python 中表示**字节序列**的不可变类型，是二进制文件操作的基础。

#### 创建 bytes 对象

```python
# 方式一：使用 b 前缀（只能包含 ASCII 字符）
data = b"Hello"
print(data)          # b'Hello'
print(type(data))    # <class 'bytes'>
print(len(data))     # 5（5 个字节）

# 方式二：将字符串编码为 bytes
text = "你好"
data = text.encode("utf-8")
print(data)          # b'\xe4\xbd\xa0\xe5\xa5\xbd'
print(len(data))     # 6（UTF-8 下每个中文字符 3 个字节）

# 方式三：从整数列表创建
data = bytes([72, 101, 108, 108, 111])
print(data)          # b'Hello'  （72=H, 101=e, 108=l, ...）

# 方式四：创建指定长度的零字节
data = bytes(10)
print(data)          # b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
```

#### bytes 与 str 的转换

```python
# str → bytes：编码（encode）
text = "Python 文件操作"
data = text.encode("utf-8")
print(data)   # b'Python \xe6\x96\x87\xe4\xbb\xb6\xe6\x93\x8d\xe4\xbd\x9c'

# bytes → str：解码（decode）
restored = data.decode("utf-8")
print(restored)   # Python 文件操作
```

> **记忆技巧：** `str` 是人类看的（字符），`bytes` 是机器看的（字节）。`encode` = 人→机，`decode` = 机→人。

#### bytes 的索引和切片

```python
data = b"Hello, World!"

# 索引返回整数（该字节的数值，0~255）
print(data[0])      # 72  （字母 H 的 ASCII 码）
print(data[7])      # 87  （字母 W 的 ASCII 码）

# 切片返回 bytes
print(data[0:5])    # b'Hello'
print(data[-6:])    # b'World!'
```

#### `bytearray`：可变的字节序列

`bytes` 是不可变的，如果你需要修改字节内容，可以使用 `bytearray`：

```python
data = bytearray(b"Hello")
data[0] = 74           # 把 H(72) 改成 J(74)
print(data)            # bytearray(b'Jello')

data.append(33)        # 追加一个字节 !(33)
print(data)            # bytearray(b'Jello!')

data.extend(b" World")
print(data)            # bytearray(b'Jello! World')
```

### 2.4 二进制文件的读取

#### 读取整个文件

```python
with open("photo.jpg", "rb") as f:
    data = f.read()

print(type(data))        # <class 'bytes'>
print(len(data))         # 文件大小（字节数）
print(data[:10])         # 查看前 10 个字节
```

#### 分块读取（处理大文件的关键技术）

大文件不能一次性加载到内存，需要分块读取：

```python
CHUNK_SIZE = 4096  # 每次读取 4KB（4096 字节）

with open("large_video.mp4", "rb") as f:
    while True:
        chunk = f.read(CHUNK_SIZE)
        if not chunk:          # 读到空 bytes 表示文件结束
            break
        # 对 chunk 进行处理...
        print(f"读取了 {len(chunk)} 字节")
```

更 Pythonic 的写法——使用**海象运算符** `:=`（Python 3.8+）。这是语法糖，不会也不影响理解分块读取：

```python
CHUNK_SIZE = 4096

with open("large_video.mp4", "rb") as f:
    while chunk := f.read(CHUNK_SIZE):   # 赋值的同时判断是否为空
        print(f"读取了 {len(chunk)} 字节")
```

#### 查看文件头部来识别文件类型

不同类型的文件通常以特定的"魔术字节"（Magic Bytes）开头，这可以用来对常见文件做初步识别：

```python
def identify_file_type(filepath):
    """通过文件头部的魔术字节识别文件类型"""
    # 各种文件类型的魔术字节签名
    signatures = {
        b'\x89PNG':      'PNG 图片',
        b'\xff\xd8\xff':  'JPEG 图片',
        b'GIF87a':        'GIF 图片（87a）',
        b'GIF89a':        'GIF 图片（89a）',
        b'PK':            'ZIP 家族文件（可能是 zip/docx/xlsx 等）',
        b'%PDF':          'PDF 文档',
        b'ID3':           'MP3 音频（ID3 标签）',
        b'\xff\xfb':      'MP3 音频',
    }
    
    with open(filepath, "rb") as f:
        header = f.read(8)   # 读取前 8 个字节就够了
    
    for magic, file_type in signatures.items():
        if header.startswith(magic):
            return file_type
    
    return "未知类型"


# 使用示例
print(identify_file_type("photo.jpg"))    # JPEG 图片
print(identify_file_type("doc.pdf"))      # PDF 文档
```

> **注意：** 魔术字节只能辅助判断文件类型，不保证 100% 准确。很多格式还有更复杂的版本差异和容器结构。

### 2.5 二进制文件的写入

```python
# 写入原始字节
with open("output.bin", "wb") as f:
    f.write(b'\x00\x01\x02\x03\x04')
    f.write(bytes([255, 254, 253]))
    f.write("你好".encode("utf-8"))
```

---

### 2.6 实战：文件复制

文件复制是二进制操作最经典的应用场景——不管是图片、音频还是视频，用二进制模式都能原封不动地复制。

#### 版本一：一次性复制（简单，但不适合大文件）

```python
import os

def copy_file_simple(src, dst):
    """一次性读入内存再写出，适合小文件"""
    if os.path.abspath(src) == os.path.abspath(dst):
        raise ValueError("目标文件不能和源文件相同")

    with open(src, "rb") as f_in:
        data = f_in.read()          # 全部读入内存

    with open(dst, "wb") as f_out:
        f_out.write(data)           # 全部写出

    print(f"复制完成：{src} → {dst}（{len(data)} 字节）")


# 创建测试文件，再复制
with open("sample.bin", "wb") as f:
    f.write(b"demo data" * 1000)

copy_file_simple("sample.bin", "sample_copy.bin")
```

**优点：** 代码简单，逻辑清晰。
**缺点：** 如果文件有 2GB，就会占用 2GB 内存。对于大文件完全不可行。

#### 版本二：分块复制（推荐，适用于任意大小的文件）

```python
import os

def copy_file(src, dst, chunk_size=4096):
    """分块读写，内存友好，适用于任意大小的文件
    
    Args:
        src: 源文件路径
        dst: 目标文件路径
        chunk_size: 每次读取的字节数，默认 4KB
    """
    if os.path.abspath(src) == os.path.abspath(dst):
        raise ValueError("目标文件不能和源文件相同")

    bytes_copied = 0

    with open(src, "rb") as f_in, open(dst, "wb") as f_out:
        while True:
            chunk = f_in.read(chunk_size)
            if not chunk:              # 空 bytes 表示读到文件末尾
                break
            f_out.write(chunk)
            bytes_copied += len(chunk)

    # 格式化文件大小显示
    if bytes_copied < 1024:
        size_str = f"{bytes_copied} B"
    elif bytes_copied < 1024 * 1024:
        size_str = f"{bytes_copied / 1024:.1f} KB"
    else:
        size_str = f"{bytes_copied / (1024 * 1024):.1f} MB"

    print(f"复制完成：{src} → {dst}（{size_str}）")


# 创建测试文件，再复制
with open("sample.bin", "wb") as f:
    f.write(b"demo data" * 1000)

copy_file("sample.bin", "sample_copy.bin")
```

**核心思想：** 每次只读 4KB 到内存，写完就释放，再读下一块。无论文件多大，内存占用始终只有 4KB 左右。

> **`chunk_size` 怎么选？** 一般设置为 4096（4KB）、8192（8KB）或 65536（64KB）。太小则读写次数多、速度慢；太大则占用内存多。4096 是操作系统磁盘的常见块大小，是一个经典默认值。实际应用中 64KB ~ 1MB 也很常用。

#### 版本三：使用海象运算符简化（Python 3.8+）

```python
import os

def copy_file_walrus(src, dst, chunk_size=4096):
    """使用海象运算符 := 的简洁写法"""
    if os.path.abspath(src) == os.path.abspath(dst):
        raise ValueError("目标文件不能和源文件相同")

    with open(src, "rb") as f_in, open(dst, "wb") as f_out:
        while chunk := f_in.read(chunk_size):   # 赋值并判断，非空则继续
            f_out.write(chunk)

    print(f"复制完成：{src} → {dst}")


# 创建测试文件，再复制
with open("sample.bin", "wb") as f:
    f.write(b"demo data" * 1000)

copy_file_walrus("sample.bin", "sample_copy.bin")
```

海象运算符 `:=` 的作用是"在表达式中赋值"——它把 `f_in.read(chunk_size)` 的结果赋给 `chunk`，同时整个表达式的值就是 `chunk`。当 `chunk` 为空 `b''`（即到达文件末尾），`while` 判定为 `False`，循环结束。不会这种写法也没关系，上一版 `while True` 更适合打基础。

#### 版本四：使用标准库 `shutil`（生产环境推荐）

Python 标准库 `shutil` 已经封装好了高效的文件复制函数，生产环境中直接使用即可：

```python
import shutil

# 复制文件内容
shutil.copy("photo.jpg", "photo_backup.jpg")

# 复制文件内容 + 元数据（修改时间、权限等）
shutil.copy2("photo.jpg", "photo_backup.jpg")

# 复制整个目录（递归复制）
shutil.copytree("my_folder", "my_folder_backup")
```

> **教学建议：** 自己实现分块复制是为了理解原理。实际项目中，优先使用 `shutil`，它经过充分优化和测试。

### 2.7 实战：带进度条的文件复制

结合 `os.path.getsize()` 获取文件总大小，我们可以实现一个带进度显示的复制工具：

```python
import os

def copy_with_progress(src, dst, chunk_size=65536):
    """带进度百分比的文件复制"""
    if os.path.abspath(src) == os.path.abspath(dst):
        raise ValueError("目标文件不能和源文件相同")

    total_size = os.path.getsize(src)       # 获取源文件总字节数
    bytes_copied = 0

    if total_size == 0:
        with open(dst, "wb"):
            pass
        print("复制完成！源文件为空，共 0 字节")
        return

    with open(src, "rb") as f_in, open(dst, "wb") as f_out:
        while chunk := f_in.read(chunk_size):
            f_out.write(chunk)
            bytes_copied += len(chunk)

            # 计算并打印进度
            percent = bytes_copied / total_size * 100
            bar_len = 40
            filled = int(bar_len * bytes_copied / total_size)
            bar = "█" * filled + "░" * (bar_len - filled)
            print(f"\r[{bar}] {percent:.1f}%", end="", flush=True)

    print(f"\n复制完成！共 {bytes_copied:,} 字节")


# 创建测试文件，再复制
with open("large_file.bin", "wb") as f:
    f.write(b"demo data" * 20000)

copy_with_progress("large_file.bin", "large_file_backup.bin")
```

运行效果（动态更新）：

```
[████████████████████████░░░░░░░░░░░░░░░░] 60.3%
```

**知识点解析：**

- `\r` 是回车符，让光标回到行首，实现"覆盖打印"效果，看起来就像进度条在原地更新
- `end=""` 阻止 `print` 在末尾自动换行
- `flush=True` 强制立即输出（默认情况下输出可能会被缓存）
- 如果终端不能正确显示 `█` / `░`，可以把它们换成 `#` / `-`

### 2.8 实战练习

#### 练习 1：文件备份工具

```python
import os
from datetime import datetime

def backup_file(filepath):
    """创建带日期时间戳的文件备份
    
    例如：report.docx → report_20260507_102345.docx
    """
    if not os.path.isfile(filepath):
        print(f"错误：'{filepath}' 不存在或不是普通文件")
        return None

    # 拆分文件名和扩展名
    name, ext = os.path.splitext(filepath)
    # 生成时间戳
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S_%f")
    # 拼接备份文件名
    backup_path = f"{name}_{timestamp}{ext}"

    # 分块复制
    with open(filepath, "rb") as f_in, open(backup_path, "xb") as f_out:
        while chunk := f_in.read(65536):
            f_out.write(chunk)

    size = os.path.getsize(backup_path)
    print(f"备份成功：{filepath} → {backup_path}（{size:,} 字节）")
    return backup_path


# 创建测试文件并备份
with open("important_document.docx", "wb") as f:
    f.write(b"important content")

backup_file("important_document.docx")
# 输出：备份成功：important_document.docx → important_document_20260507_102345_123456.docx（25,600 字节）
```

#### 练习 2：比较两个文件是否完全相同

```python
import os

def files_are_identical(file1, file2, chunk_size=65536):
    """逐块比较两个文件是否完全相同"""
    # 先比较文件大小，大小不同肯定不相同
    if os.path.getsize(file1) != os.path.getsize(file2):
        return False

    with open(file1, "rb") as f1, open(file2, "rb") as f2:
        while True:
            chunk1 = f1.read(chunk_size)
            chunk2 = f2.read(chunk_size)
            if chunk1 != chunk2:
                return False
            if not chunk1:      # 两个文件都读完了，且此前所有块都一致
                return True


# 创建两个测试文件并验证
with open("file_a.bin", "wb") as f:
    f.write(b"Python file demo")

with open("file_b.bin", "wb") as f:
    f.write(b"Python file demo")

if files_are_identical("file_a.bin", "file_b.bin"):
    print("✅ 两个文件完全一致，复制正确！")
else:
    print("❌ 文件内容不同！")
```

---

## 第三章：目录访问

### 3.1 相对路径与绝对路径

这是文件操作中最基础也最容易混淆的概念，必须彻底搞清楚。

#### 绝对路径

从磁盘根目录开始的**完整路径**，唯一确定一个文件的位置：

```python
# Windows 绝对路径：以盘符开头
r"C:\Users\zhangsan\Desktop\project\data.txt"

# Mac/Linux 绝对路径：以 / 开头
"/home/zhangsan/Desktop/project/data.txt"
```

特点：无论你的程序在哪里运行，绝对路径指向的文件位置都不变。

#### 相对路径

相对于**当前工作目录（Current Working Directory, CWD）** 的路径：

```python
import os
print(os.getcwd())   # 查看当前工作目录，例如：C:\Users\zhangsan\Desktop\project
```

假设当前工作目录是 `C:\Users\zhangsan\Desktop\project`，那么：

```python
# 相对路径                    → 实际指向的绝对路径
"data.txt"                   # → C:\...\project\data.txt （当前目录下）
"data/input.txt"             # → C:\...\project\data\input.txt （子目录下）
"../other/file.txt"          # → C:\...\Desktop\other\file.txt （上级目录的同级目录）
"../../file.txt"             # → C:\...\Users\zhangsan\file.txt （上两级目录）
```

路径中的特殊符号：

- **`.`**（一个点）—— 当前目录
- **`..`**（两个点）—— 上级目录（父目录）

#### 何时用哪种？

- **脚本内部引用项目文件** → 不要直接依赖当前工作目录，建议先确定一个稳定的基准目录，再从它拼接路径
- **用户输入或配置文件** → 要明确约定路径相对于哪个基准目录，必要时统一转成绝对路径
- **跨盘符引用**（如从 C 盘访问 D 盘的文件）→ 必须用绝对路径

例如，让路径相对于当前脚本所在目录，而不是相对于启动程序时的当前工作目录：

```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent
data_file = BASE_DIR / "data" / "input.txt"
```

---

### 3.2 `os` 模块：目录操作的瑞士军刀

`os` 模块是 Python 标准库中操作文件系统的核心模块。

#### 3.2.1 路径相关操作：`os.path`

```python
import os

path = os.path.join("project", "data", "report.txt")

# 获取文件名
os.path.basename(path)        # 'report.txt'

# 获取目录部分
os.path.dirname(path)         # 'project/data'（Linux/Mac）或 'project\\data'（Windows）

# 拆分目录和文件名
os.path.split(path)           # ('project/data', 'report.txt') 或 ('project\\data', 'report.txt')

# 拆分文件名和扩展名
os.path.splitext(path)        # ('project/data/report', '.txt') 或 ('project\\data\\report', '.txt')

# 拼接路径（推荐！自动处理分隔符）
os.path.join("data", "2024", "report.txt")  # 'data\\2024\\report.txt'（Windows）

# 判断路径是否存在
os.path.exists(path)          # True 或 False

# 判断是文件还是目录
os.path.isfile(path)          # True：是文件
os.path.isdir(path)           # False：不是目录

# 获取文件大小（字节）
os.path.getsize(path)         # 例如 1024

# 获取绝对路径
os.path.abspath("data.txt")   # 将相对路径转为绝对路径
```

> **重点：** 用 `os.path.join()` 拼接路径，不要手动拼字符串！它会自动处理不同操作系统的路径分隔符（Windows 用 `\`，Linux/Mac 用 `/`）。

```python
# ❌ 手动拼接：在 Linux 上会出问题
path = "data" + "\\" + "file.txt"

# ✅ 用 os.path.join：跨平台兼容
path = os.path.join("data", "file.txt")
```

#### 3.2.2 目录的创建与删除

```python
import os

# 创建单层目录
os.mkdir("new_folder")              # 父目录必须存在，否则报错

# 创建多层目录（推荐）
os.makedirs("a/b/c", exist_ok=True)
# exist_ok=True：目录已存在时不报错（非常实用！）
# 会自动创建所有不存在的中间目录：a → a/b → a/b/c

# 删除空目录
os.rmdir("new_folder")             # 目录必须为空，否则报错

# 递归删除空目录
os.removedirs("a/b/c")             # 从最内层开始，逐层删除空目录

# 删除文件
os.remove("temp.txt")              # 删除单个文件

# 重命名文件或目录
os.rename("old_name.txt", "new_name.txt")
```

> **⚠️ 删除类 API 风险提示：** `os.remove()`、`os.rmdir()`、`os.removedirs()`、`Path.unlink()`、`Path.rmdir()` 和 `shutil.rmtree()` 通常都不会把文件放进回收站，而是直接删除。执行前最好先打印目标路径，确认路径正确；递归删除目录时尤其要谨慎。

如果要删除非空目录（包含文件和子目录），需要用 `shutil.rmtree()`：

```python
import shutil

shutil.rmtree("my_folder")    # 递归删除目录及其所有内容，⚠️ 极其危险
```

#### 3.2.3 获取当前工作目录与切换目录

```python
import os

# 获取当前工作目录
cwd = os.getcwd()
print(cwd)   # 例如：C:\Users\zhangsan\Desktop\project

# 切换工作目录（不推荐在程序中使用，影响全局状态）
os.chdir(r"C:\Users\zhangsan\Documents")
print(os.getcwd())   # C:\Users\zhangsan\Documents
```

> **建议：** 尽量不要在程序中调用 `os.chdir()`。它会改变整个程序的当前目录，可能导致其他相对路径失效。推荐用绝对路径或 `os.path.join()` 来构造路径。

---

### 3.3 遍历目录

#### 3.3.1 `os.listdir()` —— 列出目录内容

```python
import os

# 列出当前目录下所有文件和子目录的名称
items = os.listdir(".")     # "." 表示当前目录
print(items)
# ['data', 'main.py', 'readme.txt', 'output']  ← 只有名称，不含路径

# 列出指定目录
items = os.listdir(r"C:\Users\zhangsan\Desktop")
```

**注意事项：**

- 返回的只是**名称列表**（不含路径前缀），你需要自己用 `os.path.join()` 拼接完整路径
- 不会递归进入子目录，只列出当前一层
- 不区分文件和目录，需要用 `os.path.isfile()` / `os.path.isdir()` 判断

**常见用法——筛选特定类型的文件：**

```python
import os

folder = "."

# 列出所有 .py 文件
py_files = [f for f in os.listdir(folder)
            if f.endswith(".py") and os.path.isfile(os.path.join(folder, f))]

print("Python 文件：", py_files)

# 列出所有子目录
subdirs = [d for d in os.listdir(folder)
           if os.path.isdir(os.path.join(folder, d))]

print("子目录：", subdirs)
```

#### 3.3.2 `os.walk()` —— 递归遍历目录树（重点！）

`os.walk()` 是处理目录结构最强大的工具，它会**递归地**遍历指定目录下的所有子目录。

```python
import os

for dirpath, dirnames, filenames in os.walk("my_project"):
    print(f"当前目录：{dirpath}")
    print(f"  子目录：{dirnames}")
    print(f"  文件：{filenames}")
    print()
```

假设 `my_project` 的结构是：

```
my_project/
├── main.py
├── config.json
├── data/
│   ├── input.txt
│   └── output.txt
└── utils/
    ├── helper.py
    └── tools.py
```

输出：

```
当前目录：my_project
  子目录：['data', 'utils']
  文件：['main.py', 'config.json']

当前目录：my_project/data（Windows 上可能显示为 my_project\data）
  子目录：[]
  文件：['input.txt', 'output.txt']

当前目录：my_project/utils（Windows 上可能显示为 my_project\utils）
  子目录：[]
  文件：['helper.py', 'tools.py']
```

`os.walk()` 每次迭代返回一个三元组 `(dirpath, dirnames, filenames)`：

- **`dirpath`**：当前正在遍历的目录路径（字符串）
- **`dirnames`**：该目录下的子目录名列表
- **`filenames`**：该目录下的文件名列表

**实用示例——统计某目录下所有文件的总大小：**

```python
import os

def get_folder_size(folder):
    """递归计算文件夹总大小"""
    total = 0
    for dirpath, dirnames, filenames in os.walk(folder):
        for filename in filenames:
            filepath = os.path.join(dirpath, filename)
            total += os.path.getsize(filepath)
    return total


size = get_folder_size("my_project")
print(f"总大小：{size / 1024 / 1024:.2f} MB")
```

**实用示例——查找所有指定扩展名的文件：**

```python
import os

def find_files(folder, extension):
    """递归查找所有指定扩展名的文件"""
    results = []
    for dirpath, dirnames, filenames in os.walk(folder):
        for filename in filenames:
            if filename.endswith(extension):
                full_path = os.path.join(dirpath, filename)
                results.append(full_path)
    return results


# 查找所有 Python 文件
py_files = find_files("my_project", ".py")
for f in py_files:
    print(f)
# my_project/main.py（Linux/Mac）或 my_project\main.py（Windows）
# my_project/utils/helper.py（Linux/Mac）或 my_project\utils\helper.py（Windows）
# my_project/utils/tools.py（Linux/Mac）或 my_project\utils\tools.py（Windows）
```

---

### 3.4 `pathlib` —— 现代化的路径处理（Python 3.4+）

`os.path` 用字符串来表示路径，操作起来需要调用很多函数。Python 3.4 引入了 `pathlib` 模块，用**面向对象**的方式处理路径，代码更直观、更优雅。

#### 创建 Path 对象

```python
from pathlib import Path

# 创建 Path 对象
p = Path("data/input.txt")
print(p)             # data\input.txt（Windows）或 data/input.txt（Linux/Mac）
print(type(p))       # Windows 上是 WindowsPath，Linux/Mac 上是 PosixPath
```

#### 路径拼接：用 `/` 运算符

这是 `pathlib` 最优雅的特性——用 `/` 来拼接路径：

```python
from pathlib import Path

base = Path("project")
data_dir = base / "data"            # project/data
file_path = data_dir / "input.txt"  # project/data/input.txt

print(file_path)   # project/data/input.txt（Linux/Mac）或 project\data\input.txt（Windows）
```

对比一下 `os.path.join()`：

```python
# os.path 风格（老方法）
import os
path = os.path.join("project", "data", "input.txt")

# pathlib 风格（新方法，更直观）
from pathlib import Path
path = Path("project") / "data" / "input.txt"
```

#### 常用属性和方法

```python
from pathlib import Path

p = Path("project/data/report.txt")

# --- 路径的各个组成部分 ---
p.name          # 'report.txt'         文件名（含扩展名）
p.stem          # 'report'             文件名（不含扩展名）
p.suffix        # '.txt'               扩展名
p.parent        # project/data         父目录
p.parts         # ('project', 'data', 'report.txt')  各部分

# --- 状态判断 ---
p.exists()      # True/False   路径是否存在
p.is_file()     # True/False   是否是文件
p.is_dir()      # True/False   是否是目录

# --- 路径转换 ---
p.resolve()     # 转为绝对路径（并解析符号链接）
# p.absolute() 也能得到绝对路径，但不做 resolve() 那样的规范化处理，初学时优先用 resolve()

# --- 文件信息 ---
p.stat()        # 返回文件状态信息（大小、修改时间等）
p.stat().st_size    # 文件大小（字节）
```

#### 目录操作

```python
from pathlib import Path

folder = Path("my_new_folder/sub1/sub2")

# 创建多层目录（相当于 os.makedirs）
folder.mkdir(parents=True, exist_ok=True)
# parents=True：自动创建不存在的父目录
# exist_ok=True：目录已存在时不报错

# 删除空目录
folder.rmdir()

# 删除文件
Path("temp.txt").unlink(missing_ok=True)
# missing_ok=True：文件不存在时不报错（Python 3.8+）
```

#### 遍历目录

```python
from pathlib import Path

project = Path("my_project")

# iterdir()：列出当前目录内容（不递归），相当于 os.listdir()
for item in project.iterdir():
    kind = "📁" if item.is_dir() else "📄"
    print(f"  {kind} {item.name}")

# glob()：按模式匹配文件（不递归）
for py_file in project.glob("*.py"):
    print(py_file)
# my_project/main.py（Linux/Mac）或 my_project\main.py（Windows）

# rglob()：按模式递归匹配（r = recursive），这是 pathlib 的杀手级功能
for py_file in project.rglob("*.py"):
    print(py_file)
# my_project/main.py（Linux/Mac）或 my_project\main.py（Windows）
# my_project/utils/helper.py（Linux/Mac）或 my_project\utils\helper.py（Windows）
# my_project/utils/tools.py（Linux/Mac）或 my_project\utils\tools.py（Windows）
```

> **`rglob("*.py")` vs `os.walk()` + 手动筛选：** 一行代码顶五六行，这就是 `pathlib` 的威力。

#### `pathlib` 读写文件的快捷方法

`Path` 对象还内置了读写方法，小文件操作时非常方便：

```python
from pathlib import Path

p = Path("hello.txt")

# 写入文本（自动创建文件，覆盖写入）
p.write_text("你好，世界！\nHello, Python!", encoding="utf-8")

# 读取文本
content = p.read_text(encoding="utf-8")
print(content)

# 写入二进制数据
Path("data.bin").write_bytes(b'\x00\x01\x02\x03')

# 读取二进制数据
raw = Path("data.bin").read_bytes()
print(raw)   # b'\x00\x01\x02\x03'
```

> **注意：** 这些快捷方法会自动打开和关闭文件（内部使用了 `with`），但它们**一次性读写全部内容**，不适合大文件。大文件还是要用 `with open()` 分块处理。

#### `os.path` vs `pathlib` 对照表

| 操作     | `os` / `os.path`                | `pathlib`                              |
| -------- | ------------------------------- | -------------------------------------- |
| 拼接路径 | `os.path.join(a, b)`            | `Path(a) / b`                          |
| 文件名   | `os.path.basename(p)`           | `p.name`                               |
| 扩展名   | `os.path.splitext(p)[1]`        | `p.suffix`                             |
| 父目录   | `os.path.dirname(p)`            | `p.parent`                             |
| 是否存在 | `os.path.exists(p)`             | `p.exists()`                           |
| 创建目录 | `os.makedirs(p, exist_ok=True)` | `p.mkdir(parents=True, exist_ok=True)` |
| 遍历目录 | `os.listdir(p)`                 | `p.iterdir()`                          |
| 递归匹配 | `os.walk()` + 手动筛选          | `p.rglob("*.ext")`                     |
| 读取文本 | `open(p).read()`                | `p.read_text()`                        |

> **建议：** 新代码推荐使用 `pathlib`，它更直观、更 Pythonic。但 `os` 模块仍然广泛使用，两者都要掌握。

---

### 3.5 实战练习

#### 练习 1：批量查找文件

```python
from pathlib import Path

def find_files_by_extension(folder, *extensions):
    """在指定目录下递归查找所有匹配扩展名的文件
    
    Args:
        folder: 要搜索的目录路径
        *extensions: 一个或多个扩展名，如 ".py", ".txt"
    
    Returns:
        按扩展名分组的字典
    """
    folder = Path(folder)
    if not folder.is_dir():
        print(f"错误：'{folder}' 不是有效目录")
        return {}

    normalized_exts = [
        ext.lower() if ext.startswith(".") else f".{ext.lower()}"
        for ext in extensions
    ]

    results = {ext: [] for ext in normalized_exts}
    for file in folder.rglob("*"):
        if file.is_file():
            ext = file.suffix.lower()
            if ext in results:
                results[ext].append(file)

    for ext in results:
        results[ext].sort()

    # 打印结果
    for ext, files in results.items():
        print(f"\n{'=' * 50}")
        print(f"  {ext} 文件（共 {len(files)} 个）")
        print(f"{'=' * 50}")
        for f in files:
            size = f.stat().st_size
            print(f"  {f}  ({size:,} 字节)")

    return results


# 查找当前项目下所有 Python 和 JSON 文件
find_files_by_extension(".", ".py", ".json")
```

#### 练习 2：统计目录中各类文件的数量和大小

```python
from pathlib import Path
from collections import defaultdict

def analyze_folder(folder):
    """分析文件夹中各类文件的数量和总大小"""
    folder = Path(folder)
    if not folder.is_dir():
        print(f"错误：'{folder}' 不是有效目录")
        return {}

    stats = defaultdict(lambda: {"count": 0, "size": 0})

    for file in folder.rglob("*"):
        if file.is_file():
            ext = file.suffix.lower() if file.suffix else "(无扩展名)"
            stats[ext]["count"] += 1
            stats[ext]["size"] += file.stat().st_size

    # 按文件数量降序排列
    sorted_stats = sorted(stats.items(), key=lambda x: x[1]["count"], reverse=True)

    def fmt_size(n):
        if n < 1024:
            return f"{n} B"
        elif n < 1024 ** 2:
            return f"{n / 1024:.1f} KB"
        elif n < 1024 ** 3:
            return f"{n / 1024**2:.1f} MB"
        else:
            return f"{n / 1024**3:.2f} GB"

    print(f"目录分析：{folder.resolve()}")
    print(f"{'扩展名':<12} {'数量':>6} {'总大小':>12}")
    print("-" * 34)
    for ext, info in sorted_stats:
        print(f"{ext:<12} {info['count']:>6} {fmt_size(info['size']):>12}")

    return stats


analyze_folder(".")
```

---

## 第四章：综合练习

### 练习 1：文本文件统计工具

```python
from pathlib import Path
from collections import Counter

def analyze_text_file(filepath):
    """全面分析一个文本文件的统计信息"""
    path = Path(filepath)
    if not path.is_file():
        print(f"错误：'{filepath}' 不存在或不是文件")
        return

    text = path.read_text(encoding="utf-8")
    lines = text.splitlines()           # 按行拆分（不保留换行符）

    # 基本统计
    total_chars = len(text)
    total_lines = len(lines)
    non_empty_lines = sum(1 for line in lines if line.strip())
    words = text.split()                # 简单按空白分词
    total_words = len(words)

    # 字符频率（前 10）
    char_counter = Counter(c for c in text if not c.isspace())
    top_chars = char_counter.most_common(10)

    # 最长行和最短非空行
    non_empty = [line for line in lines if line.strip()]
    longest = max(non_empty, key=len) if non_empty else ""
    shortest = min(non_empty, key=len) if non_empty else ""

    # 输出报告
    print(f"{'=' * 50}")
    print(f"  文件分析报告：{path.name}")
    print(f"{'=' * 50}")
    print(f"  文件大小：{path.stat().st_size:,} 字节")
    print(f"  总字符数：{total_chars:,}")
    print(f"  总行数：{total_lines}")
    print(f"  非空行数：{non_empty_lines}")
    print(f"  空行数：{total_lines - non_empty_lines}")
    print(f"  词数（按空白分割）：{total_words}")
    print(f"  最长行（{len(longest)}字符）：{longest[:60]}{'...' if len(longest) > 60 else ''}")
    print(f"  最短非空行（{len(shortest)}字符）：{shortest}")
    print(f"\n  出现频率最高的 10 个字符：")
    for char, count in top_chars:
        print(f"    '{char}' → {count} 次")


# 创建测试文件
test_content = """Python 是一门优雅的编程语言。
它由 Guido van Rossum 于 1991 年创造。

Python 的设计哲学强调代码可读性。
使用 Python，你可以用更少的代码完成更多的工作。

Python 广泛应用于：
- Web 开发
- 数据科学
- 人工智能
- 自动化脚本
"""

Path("test_article.txt").write_text(test_content, encoding="utf-8")
analyze_text_file("test_article.txt")
```

---

### 练习 2：二进制文件复制工具（带校验）

```python
import hashlib
from pathlib import Path

def copy_and_verify(src, dst, chunk_size=65536):
    """复制文件并用 SHA-256 校验完整性"""
    src_path = Path(src)
    dst_path = Path(dst)

    if not src_path.is_file():
        print(f"错误：源文件 '{src}' 不存在或不是普通文件")
        return False

    if src_path.resolve() == dst_path.resolve():
        print("错误：目标文件不能和源文件相同")
        return False

    # 复制文件，同时计算源文件的哈希值
    src_hash = hashlib.sha256()
    total = 0

    with open(src_path, "rb") as f_in, open(dst_path, "wb") as f_out:
        while chunk := f_in.read(chunk_size):
            f_out.write(chunk)
            src_hash.update(chunk)     # 边读边算哈希
            total += len(chunk)

    # 计算目标文件的哈希值
    dst_hash = hashlib.sha256()
    with open(dst_path, "rb") as f:
        while chunk := f.read(chunk_size):
            dst_hash.update(chunk)

    # 校验
    src_digest = src_hash.hexdigest()
    dst_digest = dst_hash.hexdigest()
    is_ok = (src_digest == dst_digest)

    print(f"源文件 SHA-256：{src_digest}")
    print(f"目标 SHA-256：  {dst_digest}")
    print(f"文件大小：      {total:,} 字节")
    print(f"校验结果：      {'✅ 一致' if is_ok else '❌ 不一致！'}")
    return is_ok


# 创建一个测试用二进制文件，让示例可以直接运行
Path("sample.bin").write_bytes(b"\x00\x01\x02Python\xff\xfe")
copy_and_verify("sample.bin", "sample_backup.bin")
```

---

### 练习 3：目录文本扫描工具

```python
from pathlib import Path

def scan_text_files(folder, extensions=(".txt", ".md")):
    """扫描目录中的文本文件，统计总行数和总词数"""
    root = Path(folder)
    if not root.is_dir():
        print(f"错误：'{folder}' 不是有效目录")
        return []

    extensions = {ext.lower() for ext in extensions}
    reports = []

    for file in root.rglob("*"):
        if not file.is_file() or file.suffix.lower() not in extensions:
            continue

        try:
            text = file.read_text(encoding="utf-8")
        except UnicodeDecodeError:
            print(f"跳过编码不匹配的文件：{file}")
            continue

        lines = text.splitlines()
        words = text.split()
        reports.append({
            "path": file,
            "lines": len(lines),
            "words": len(words),
            "size": file.stat().st_size,
        })

    reports.sort(key=lambda item: item["words"], reverse=True)

    print(f"\n{'=' * 60}")
    print(f"  文本扫描报告：{root.resolve()}")
    print(f"{'=' * 60}")
    print(f"  {'文件':<25} {'行数':>6} {'词数':>6} {'大小':>10}")
    print(f"  {'-' * 54}")
    for item in reports:
        print(f"  {item['path'].name:<25} {item['lines']:>6} {item['words']:>6} {item['size']:>8} B")

    total_lines = sum(item["lines"] for item in reports)
    total_words = sum(item["words"] for item in reports)
    print(f"  {'-' * 54}")
    print(f"  {'合计':<25} {total_lines:>6} {total_words:>6}")
    return reports


# 创建测试目录和文本文件
sample_dir = Path("text_samples")
sample_dir.mkdir(exist_ok=True)
(sample_dir / "notes.txt").write_text("Python 文件操作\n逐行读取更省内存\n", encoding="utf-8")
(sample_dir / "todo.md").write_text("# TODO\n- 学习 pathlib\n- 练习文件扫描\n", encoding="utf-8")

scan_text_files(sample_dir)
```

---

### 练习 4：批量文件整理工具

这是一个实用的自动化工具——将一个杂乱的文件夹中的文件**按扩展名自动分类**到子目录中。

```python
import shutil
from pathlib import Path

def organize_files(source_folder, dry_run=True):
    """按扩展名将文件分类整理到子目录中
    
    Args:
        source_folder: 要整理的目录
        dry_run: True=预览模式（不实际移动），False=实际执行
    """
    # 分类规则：扩展名 → 目标文件夹名
    category_map = {
        "图片": {".jpg", ".jpeg", ".png", ".gif", ".bmp", ".webp", ".svg"},
        "文档": {".txt", ".pdf", ".doc", ".docx", ".xls", ".xlsx", ".ppt", ".pptx", ".md"},
        "音频": {".mp3", ".wav", ".flac", ".aac", ".ogg"},
        "视频": {".mp4", ".avi", ".mkv", ".mov", ".wmv", ".flv"},
        "压缩包": {".zip", ".rar", ".7z", ".tar", ".gz"},
        "代码": {".py", ".js", ".html", ".css", ".java", ".c", ".cpp", ".json"},
    }

    # 反转映射：扩展名 → 分类名
    ext_to_category = {}
    for category, extensions in category_map.items():
        for ext in extensions:
            ext_to_category[ext] = category

    source = Path(source_folder)
    if not source.is_dir():
        print(f"错误：'{source_folder}' 不是有效目录")
        return

    moved_count = 0

    # 只处理 source 目录下的直接文件（不递归进子目录）
    for file in source.iterdir():
        if not file.is_file():
            continue

        ext = file.suffix.lower()
        category = ext_to_category.get(ext, "其他")
        target_dir = source / category

        if dry_run:
            print(f"  [预览] {file.name}  →  {category}/")
            moved_count += 1
        else:
            target_dir.mkdir(exist_ok=True)
            dest = target_dir / file.name

            # 如果目标位置已存在同名文件，添加序号
            if dest.exists():
                stem = file.stem
                suffix = file.suffix
                counter = 1
                while dest.exists():
                    dest = target_dir / f"{stem}_{counter}{suffix}"
                    counter += 1

            shutil.move(str(file), str(dest))
            print(f"  [移动] {file.name}  →  {category}/{dest.name}")
            moved_count += 1

    mode = "预览" if dry_run else "实际"
    print(f"\n{mode}完成：共处理 {moved_count} 个文件")
    if dry_run:
        print("提示：确认无误后，将 dry_run 设为 False 即可实际执行。")


# 创建测试目录和文件，让预览示例可以直接运行
sample_folder = Path("messy_folder")
sample_folder.mkdir(exist_ok=True)
for filename in ["photo.jpg", "report.pdf", "music.mp3", "script.py", "archive.zip"]:
    path = sample_folder / filename
    if not path.exists():
        path.write_bytes(b"demo")

# 第一步：预览（不实际移动，安全检查）
organize_files("messy_folder", dry_run=True)

# 第二步：确认无误后，实际执行
# organize_files("messy_folder", dry_run=False)
```

**设计亮点解析：**

1. **`dry_run` 模式（预演模式）：** 这是一个非常好的编程习惯。任何涉及文件移动/删除的工具，都应该先提供预览，让用户确认后再实际执行。避免手滑导致文件丢失。

2. **重名文件处理：** 如果目标位置已有同名文件，自动加序号（`photo.jpg` → `photo_1.jpg`），而不是覆盖。

3. **分类可配置：** `category_map` 字典定义了分类规则，你可以根据需要自由增减类别和扩展名。

---

## 总结与进阶方向

### 本教程覆盖的核心知识

| 章节           | 核心技能                                                     |
| -------------- | ------------------------------------------------------------ |
| 文本文件读写   | `open()`、`read/readline/readlines`、`write/writelines`、`with` 语句 |
| 二进制文件读写 | `rb/wb/ab` 模式、`bytes` 类型、分块读写、文件复制            |
| 目录访问       | `os.path`、`os.walk()`、`pathlib`、`glob/rglob`              |
| 综合练习       | 统计工具、复制校验、目录扫描、批量整理                       |

### 关键最佳实践回顾

1. **使用 `open()` 时优先配合 `with`**——自动关闭，异常安全；小文件也可以使用 `Path.read_text()` / `write_text()` 等快捷方法
2. **读写文本文件时显式指定正确编码**——新建文本文件优先统一使用 `encoding="utf-8"`
3. **大文件用分块读写**——避免内存溢出
4. **用 `os.path.join()` 或 `pathlib` 的 `/` 拼接路径**——跨平台兼容
5. **危险操作先 `dry_run`**——文件移动/删除前先预览
6. **用 `try-except` 处理文件异常**——程序要健壮

### 进阶方向

掌握本教程后，推荐继续学习：

- **`csv` 模块**：读写 CSV 表格
- **`json` 模块**：读写 JSON 数据
- **`pickle` 模块**：序列化 Python 对象
- **`tempfile` 模块**：安全创建临时文件
- **`zipfile` / `tarfile`**：压缩与解压
- **`watchdog` 第三方库**：实时监控文件系统变化
