# Python 文件操作与文本编码 完整教程

---

## 第一章 文件的概念以及文本文件和二进制文件的区别

### 1.1 什么是文件

在日常生活中，我们经常接触各种各样的文件：Word 文档、Excel 表格、图片、音乐、视频等等。在计算机中，**文件是存储在外部存储介质（如硬盘、U盘）上的数据集合**。

我们可以这样理解：
- 计算机运行时，数据存放在**内存（RAM）**中，内存的特点是**速度快**，但**断电后数据会丢失**。
- 为了**长期保存数据**，我们需要把数据写入到**硬盘**等存储设备中，而硬盘上保存数据的基本单位就是**文件**。

打个比方：
> 内存就像你桌子上的草稿纸，你工作时在上面写写画画很方便，但下班后草稿纸可能就被清理了。硬盘则像你的文件柜，把重要的东西放进去，明天来了还能找到。

### 1.2 文件的分类

在计算机中，文件从存储内容的角度可以分为两大类：

#### （1）文本文件

- 文本文件可以用**文本编辑器**（如记事本、VS Code）直接打开查看。
- 文本文件的内容本质上是一串**字符**，每个字符按照某种**编码规则**（如 ASCII、UTF-8）被转换成二进制数据存储在硬盘上。
- 常见的文本文件有：`.txt`、`.py`、`.html`、`.csv`、`.json`、`.xml` 等。

举例：你写的 Python 源代码 `hello.py`，用记事本打开后能看到一行行的代码文字，这就是典型的文本文件。

#### （2）二进制文件

- 二进制文件**不能**用文本编辑器直接查看（强行打开会看到乱码）。
- 二进制文件的内容是按照**特定的格式和规则**组织的二进制数据，只有对应的软件才能正确解析。
- 常见的二进制文件有：`.jpg`（图片）、`.mp3`（音频）、`.mp4`（视频）、`.exe`（可执行程序）、`.doc`（Word文档）等。

举例：你用记事本打开一张 `.jpg` 图片，会看到一堆乱码，因为图片数据不是按"字符"方式组织的，它有自己的数据格式。

### 1.3 两者的核心区别

| 特征           | 文本文件                   | 二进制文件                   |
| -------------- | -------------------------- | ---------------------------- |
| 存储方式       | 以字符编码的形式存储       | 以原始的字节(bytes)形式存储  |
| 能否直接阅读   | 可以用文本编辑器打开阅读   | 用文本编辑器打开会显示乱码   |
| 常见扩展名     | .txt .py .html .csv .json  | .jpg .mp3 .mp4 .exe .doc     |
| Python读取方式 | 以 `"r"` 或 `"w"` 模式打开 | 以 `"rb"` 或 `"wb"` 模式打开 |

### 1.4 本质上的统一

需要特别理解的一点是：**计算机中所有的文件，在硬盘上存储的都是二进制数据（0和1）**。之所以区分文本文件和二进制文件，是因为它们**对这些二进制数据的解释方式不同**：

- **文本文件**：二进制数据会按照某种编码表（如 UTF-8）被翻译成人类可读的字符。
- **二进制文件**：二进制数据按照该文件格式自身的规则来组织和解读。

---

## 第二章 文件操作套路以及 Python 中的对应函数和方法

### 2.1 文件操作的通用"三步走"套路

不管使用什么编程语言操作文件，基本步骤都是一样的，可以概括为**三步**：

```
第一步：打开文件
第二步：读/写文件（对文件进行操作）
第三步：关闭文件
```

这就像你去图书馆借书：
1. **打开**书（翻开封面）
2. **阅读**书的内容 或者 在笔记本上**写**东西
3. **合上**书（归还）

**为什么要关闭文件？**
- 打开文件后，操作系统会分配一些资源来维护这个文件的状态。
- 如果打开文件后不关闭，这些资源就一直被占用，可能导致**资源泄露**。
- 更重要的是，写入数据时，数据可能暂时保存在**缓冲区**中，关闭文件会刷新缓冲区，把数据交给操作系统写入文件。

### 2.2 Python 中操作文件的函数和方法

Python 提供了一个内置函数 `open()` 来打开文件，以及文件对象的多个方法来进行读写和关闭操作。

#### （1）`open()` 函数——打开文件

```python
file = open("文件名", "打开方式")
```

- `"文件名"` ：要打开的文件的路径（可以是相对路径或绝对路径）。
- `"打开方式"` ：指定以什么方式打开文件（读？写？追加？文本模式？二进制模式？），后面会详细讲。
- 返回值 `file` ：是一个**文件对象**，后续通过这个对象来操作文件。

#### （2）文件对象的常用方法

| 方法               | 说明                               |
| ------------------ | ---------------------------------- |
| `file.read()`      | 读取文件的全部内容（文本模式返回字符串，二进制模式返回 `bytes`） |
| `file.readline()`  | 读取文件的一行内容                 |
| `file.readlines()` | 读取文件的所有行，返回一个列表（每个元素是一行） |
| `file.write(内容)` | 将指定的内容写入文件（文本模式写字符串，二进制模式写 `bytes`） |
| `file.close()`     | 关闭文件                           |

#### （3）完整的操作流程示例

```python
# 第一步：打开文件
file = open("test.txt", "r", encoding="utf-8")

# 第二步：读取文件内容
content = file.read()
print(content)

# 第三步：关闭文件
file.close()
```

### 2.3 关于文件路径的补充

在使用 `open()` 函数时，文件名可以是：

**相对路径**——相对于程序运行时的**当前工作目录**，不一定是脚本文件所在目录：
```python
file = open("test.txt", "r")           # 当前目录下的 test.txt
file = open("data/test.txt", "r")      # 当前目录下 data 文件夹中的 test.txt
```

**绝对路径**——从磁盘根目录开始的完整路径：
```python
# Windows
file = open("C:/Users/zhangsan/test.txt", "r")

# Mac / Linux
file = open("/home/zhangsan/test.txt", "r")
```

> **小提示**：在 Windows 中路径分隔符是反斜杠 `\`，但 `\` 在 Python 字符串中是转义字符。所以推荐使用正斜杠 `/` 或者原始字符串 `r"C:\Users\zhangsan\test.txt"`。

---

## 第三章 读取文件内容

### 3.1 准备工作

首先，我们手动创建一个测试文件。在程序运行时的当前工作目录下，新建一个名为 `test.txt` 的文件，用记事本打开，输入以下内容并保存：

```
Hello, Python!
你好，世界！
Life is short, I use Python.
```

如果文件中包含中文，建议保存为 UTF-8 编码；在后面的编码章节中会专门讲解如何用 `encoding="utf-8"` 避免跨平台乱码问题。

### 3.2 使用 `read()` 方法读取全部内容

`read()` 方法会一次性读取文件的**全部内容**，返回一个字符串。

```python
# 1. 打开文件
file = open("test.txt", "r", encoding="utf-8")

# 2. 读取全部内容
content = file.read()

# 3. 打印内容
print(content)

# 4. 关闭文件
file.close()
```

运行结果：
```
Hello, Python!
你好，世界！
Life is short, I use Python.
```

### 3.3 使用 `read()` 时需要注意的事项

`read()` 方法适用于**小文件**。为什么？

因为 `read()` 会把文件的全部内容一次性读入内存。如果文件非常大（比如好几个 GB），那么你的内存可能会被撑爆，导致程序崩溃。

> 对于大文件的读取方式，后面会专门讲解。

### 3.4 带参数的 `read(n)`

`read()` 方法还可以接收一个参数。在**文本模式**下，参数表示要读取的**字符数**；在**二进制模式**下，参数表示要读取的**字节数**。

```python
file = open("test.txt", "r", encoding="utf-8")

# 只读取前5个字符
content = file.read(5)
print(content)  # 输出: Hello

file.close()
```

---

## 第四章 读取文件后文件指针会发生变化

### 4.1 什么是文件指针

当我们打开一个文件后，系统会自动维护一个**文件指针（File Pointer）**，它标记着**当前读写操作的位置**。

你可以把文件指针想象成你读书时用手指指着的位置：
- 刚打开书时，手指指在**第一页第一行**。
- 你每读一行，手指就往下移一行。
- 读完全书后，手指指在**最后一页的末尾**。

### 4.2 文件指针的变化规律

```python
file = open("test.txt", "r", encoding="utf-8")

# 第一次读取——从头开始读，读取全部内容
content1 = file.read()
print("--- 第一次读取 ---")
print(content1)

# 第二次读取——此时文件指针已经在文件末尾了！
content2 = file.read()
print("--- 第二次读取 ---")
print(content2)  # 输出为空！什么都没有！

file.close()
```

运行结果：
```
--- 第一次读取 ---
Hello, Python!
你好，世界！
Life is short, I use Python.
--- 第二次读取 ---

```

**分析**：
1. 第一次调用 `read()` 时，文件指针从**文件开头**开始，一直读到**文件末尾**，所以能读到全部内容。读完后，文件指针停在了**文件末尾**。
2. 第二次调用 `read()` 时，文件指针已经在**末尾**了，它后面没有任何内容了，所以返回一个**空字符串**。

### 4.3 如何重置文件指针——`seek()` 方法

如果你想重新从头读取文件，可以使用 `seek()` 方法将文件指针**移动到指定位置**。

```python
file = open("test.txt", "r", encoding="utf-8")

# 第一次读取
content1 = file.read()
print("--- 第一次读取 ---")
print(content1)

# 将文件指针重新移到文件开头
# seek(0) 中的 0 表示文件开头的位置
file.seek(0)

# 第二次读取——此时文件指针已经回到开头，可以再次读取到内容
content2 = file.read()
print("--- 第二次读取 ---")
print(content2)

file.close()
```

运行结果：
```
--- 第一次读取 ---
Hello, Python!
你好，世界！
Life is short, I use Python.
--- 第二次读取 ---
Hello, Python!
你好，世界！
Life is short, I use Python.
```

### 4.4 查看当前文件指针的位置——`tell()` 方法

```python
file = open("test.txt", "r", encoding="utf-8")

print("初始位置:", file.tell())  # 0

file.read(5)  # 读取5个字符
print("读取5个字符后:", file.tell())  # 对这个纯英文开头的文件，通常会显示 5

file.close()
```

> **注意**：在二进制模式中，`tell()` 返回的是当前的**字节位置**。在文本模式中，`tell()` 返回的是一个可传给 `seek()` 的位置标记，不应简单当作“已读字符数”。对于包含中文、换行转换或不同编码的文本文件，这个数值可能和字符数不一致。

---

## 第五章 打开文件的方式以及写入和追加数据

### 5.1 `open()` 函数的打开方式一览

```python
file = open("文件名", "打开方式")
```

常用的打开方式如下表：

| 模式   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `"r"`  | **只读**模式（默认）。文件必须存在，否则报错                 |
| `"w"`  | **只写**模式。文件存在则**清空**内容后写入；文件不存在则**创建**新文件 |
| `"a"`  | **追加**模式。文件存在则在**末尾追加**内容；文件不存在则**创建**新文件 |
| `"rb"` | 以**二进制**方式**只读**打开                                 |
| `"wb"` | 以**二进制**方式**只写**打开                                 |
| `"ab"` | 以**二进制**方式**追加**打开                                 |

### 5.2 `"r"` 模式——只读（默认模式）

```python
# 以下两种写法在打开方式上等价：
file = open("test.txt", "r")
file = open("test.txt")  # 不写第二个参数，默认打开方式就是 "r"
```

如果文件中包含中文，实际读写时仍建议显式指定 `encoding="utf-8"`。

如果文件不存在，会抛出 `FileNotFoundError` 错误：
```python
file = open("不存在的文件.txt", "r")
# FileNotFoundError: [Errno 2] No such file or directory: '不存在的文件.txt'
```

### 5.3 `"w"` 模式——只写（覆盖写入）

```python
# 以 "w" 模式打开文件
file = open("output.txt", "w", encoding="utf-8")

# 写入内容
file.write("第一行内容\n")
file.write("第二行内容\n")
file.write("第三行内容\n")

# 关闭文件
file.close()
```

执行后，`output.txt` 的内容为：
```
第一行内容
第二行内容
第三行内容
```

**特别注意——`"w"` 模式的"危险性"**：

如果 `output.txt` 原本就有内容，用 `"w"` 模式打开后，**原有内容会被全部清空**！然后再写入新的内容。

```python
# 假设 output.txt 现在已有三行内容

# 再次用 "w" 模式打开
file = open("output.txt", "w", encoding="utf-8")
file.write("新的内容\n")
file.close()

# 此时 output.txt 中只有：
# 新的内容
# （之前的三行全没了！）
```

### 5.4 `"a"` 模式——追加

如果你想在文件**末尾添加内容**，而不清空原有内容，应该使用 `"a"`（append）模式。

```python
# 假设 output.txt 现在内容为 "新的内容\n"

# 以 "a" 模式打开
file = open("output.txt", "a", encoding="utf-8")
file.write("追加的第一行\n")
file.write("追加的第二行\n")
file.close()
```

执行后，`output.txt` 的内容变为：
```
新的内容
追加的第一行
追加的第二行
```

原有的内容得到了保留，新内容被追加在末尾。

### 5.5 三种模式的对比总结

| 特性         | `"r"` 只读 | `"w"` 只写     | `"a"` 追加   |
| ------------ | ---------- | -------------- | ------------ |
| 文件不存在时 | **报错**   | **自动创建**   | **自动创建** |
| 文件已存在时 | 正常打开   | **清空原内容** | 保留原内容   |
| 能否读取     | 能         | 不能           | 不能         |
| 能否写入     | 不能       | 能             | 能           |
| 写入位置     | —          | 从头开始       | 在末尾追加   |

---

## 第六章 使用 readline 分行读取大文件

### 6.1 为什么需要分行读取

前面我们学了 `read()` 方法，它会一次性把文件的全部内容读入内存。对于小文件，这完全没问题。但如果文件非常大（几百MB甚至几个GB），一次性全部读入内存显然是不现实的，可能导致内存不足、程序崩溃。

解决思路：**一行一行地读取**，每次只在内存中保留一行的数据。

### 6.2 `readline()` 方法

`readline()` 方法每次读取文件中的**一行内容**（包括行末的换行符 `\n`）。

```python
file = open("test.txt", "r", encoding="utf-8")

# 读取第一行
line1 = file.readline()
print("第1行:", line1)

# 读取第二行
line2 = file.readline()
print("第2行:", line2)

# 读取第三行
line3 = file.readline()
print("第3行:", line3)

file.close()
```

运行结果（注意每行后面会多一个空行，因为 `readline()` 读取的内容包含 `\n`，而 `print` 本身也会换行）：
```
第1行: Hello, Python!

第2行: 你好，世界！

第3行: Life is short, I use Python.
```

如果不想出现多余的空行，可以使用 `end=""` 参数：
```python
print("第1行:", line1, end="")
```

### 6.3 使用 `readline()` 配合循环读取大文件

当文件行数未知时，我们可以用 `while` 循环配合 `readline()` 来逐行读取：

```python
file = open("test.txt", "r", encoding="utf-8")

while True:
    # 读取一行
    line = file.readline()

    # 判断是否读到了文件末尾
    # 当 readline() 返回空字符串时，说明已经读到文件末尾了
    if not line:
        break

    # 处理这一行（这里只是简单打印）
    print(line, end="")

file.close()
```

运行结果：
```
Hello, Python!
你好，世界！
Life is short, I use Python.
```

**这段代码的核心逻辑**：
1. 在 `while True` 的无限循环中，每次读取一行。
2. 如果 `readline()` 返回了空字符串 `""`（注意，空行返回的是 `"\n"` 而不是 `""`），说明文件已经读完了，用 `break` 跳出循环。
3. 如果读到了内容，就进行处理（打印、分析等）。

### 6.4 补充：`readlines()` 方法

`readlines()` 会一次性读取文件的**所有行**，返回一个**列表**，列表中的每个元素是文件中的一行（字符串）。

```python
file = open("test.txt", "r", encoding="utf-8")

lines = file.readlines()
print(lines)

file.close()
```

输出：
```python
['Hello, Python!\n', '你好，世界！\n', 'Life is short, I use Python.\n']
```

可以配合 `for` 循环遍历：
```python
file = open("test.txt", "r", encoding="utf-8")

for line in file.readlines():
    print(line, end="")

file.close()
```

> **注意**：`readlines()` 也是一次性读入全部内容，所以和 `read()` 一样，不适合处理超大文件。

### 6.5 最Pythonic的逐行读取方式

实际上，Python 的文件对象本身就是一个**可迭代对象**，可以直接用 `for` 循环逐行遍历，这是最简洁、最推荐的方式：

```python
file = open("test.txt", "r", encoding="utf-8")

for line in file:  # 直接遍历文件对象
    print(line, end="")

file.close()
```

这种方式在内部会自动逐行读取，**不会一次性加载全部内容到内存**，既简洁又适合处理大文件。

---

## 第七章 小文件复制

### 7.1 思路分析

复制文件的思路非常简单：
1. 打开**源文件**，读取其内容。
2. 打开（创建）**目标文件**，将读取到的内容写入。
3. 关闭两个文件。

对于小文件，我们可以用 `read()` 一次性读取全部内容，再用 `write()` 一次性写入。

### 7.2 代码实现

```python
# 1. 打开源文件（只读模式）
file_read = open("test.txt", "r", encoding="utf-8")

# 2. 读取源文件的全部内容
content = file_read.read()

# 3. 打开目标文件（只写模式，文件不存在会自动创建）
file_write = open("test_copy.txt", "w", encoding="utf-8")

# 4. 将内容写入目标文件
file_write.write(content)

# 5. 关闭两个文件
file_read.close()
file_write.close()

print("小文件复制完成！")
```

运行后，你会发现当前目录下多了一个 `test_copy.txt` 文件，其文本内容与 `test.txt` 一致。

### 7.3 复制二进制文件（如图片）

如果要复制图片、音乐等二进制文件，需要使用 `"rb"` 和 `"wb"` 模式：

```python
# 复制图片
file_read = open("photo.jpg", "rb")
content = file_read.read()

file_write = open("photo_copy.jpg", "wb")
file_write.write(content)

file_read.close()
file_write.close()

print("图片复制完成！")
```

### 7.4 小文件复制的局限性

上面的方法使用 `read()` 一次性读取了整个文件的内容。如果文件很大（比如一个 2GB 的视频），这种方式就不合适了——因为会把 2GB 的数据全部加载到内存中。

---

## 第八章 大文件复制

### 8.1 思路分析

大文件复制的核心思想是：**不要一次性读取全部内容，而是分批次读取和写入**。

具体做法：
1. 打开源文件和目标文件。
2. 循环读取源文件的一小部分内容（文本模式下比如每次读 1024 个字符，二进制模式下比如每次读 1024 个字节）。
3. 每读一部分，就写入到目标文件中。
4. 直到源文件读取完毕，关闭两个文件。

### 8.2 代码实现（文本文件）

```python
file_read = open("big_file.txt", "r", encoding="utf-8")
file_write = open("big_file_copy.txt", "w", encoding="utf-8")

while True:
    # 每次读取 1024 个字符
    content = file_read.read(1024)

    # 如果读取的内容为空，说明文件已读完
    if not content:
        break

    # 将读取的内容写入目标文件
    file_write.write(content)

file_read.close()
file_write.close()

print("大文件复制完成！")
```

### 8.3 代码实现（二进制文件，如视频、图片等）

```python
file_read = open("big_video.mp4", "rb")
file_write = open("big_video_copy.mp4", "wb")

while True:
    # 每次读取 1024 个字节
    content = file_read.read(1024)

    # 如果读取的内容为空，说明文件已读完
    if not content:
        break

    # 将读取的内容写入目标文件
    file_write.write(content)

file_read.close()
file_write.close()

print("大文件（二进制）复制完成！")
```

### 8.4 小文件复制 vs 大文件复制 对比

| 特点       | 小文件复制                 | 大文件复制                     |
| ---------- | -------------------------- | ------------------------------ |
| 读取方式   | `read()` 一次性全部读取    | `read(1024)` 分批次读取        |
| 内存占用   | 一次性占用与文件等大的内存 | 内存中只保留一个小块的数据     |
| 适用场景   | 文件较小（几KB到几MB）     | 任意大小的文件                 |
| 代码复杂度 | 简单                       | 稍复杂（多了循环）             |

> **实际开发建议**：即使文件不大，使用大文件复制的方式也完全没有问题，而且更加安全和通用。如果你不确定文件大小，建议统一使用大文件复制的方式。

> 如果需要**逐字节完全一致**地复制任意文件，优先使用二进制模式（`"rb"` / `"wb"`）或标准库 `shutil.copyfile()`。文本模式会进行编码解码，某些平台上还可能处理换行符。

### 8.5 使用 readline 逐行复制大文本文件

对于文本文件，也可以逐行读取和写入：

```python
file_read = open("big_file.txt", "r", encoding="utf-8")
file_write = open("big_file_copy.txt", "w", encoding="utf-8")

while True:
    line = file_read.readline()

    if not line:
        break

    file_write.write(line)

file_read.close()
file_write.close()
```

---

## 第九章 导入 os 模块，执行文件和目录管理操作

### 9.1 os 模块简介

Python 的 `os` 模块提供了与**操作系统交互**的功能，包括文件管理和目录管理。使用前需要先导入：

```python
import os
```

### 9.2 文件操作

#### （1）重命名文件

```python
import os

# os.rename("原文件名", "新文件名")
os.rename("test.txt", "test_renamed.txt")
```

#### （2）删除文件

```python
import os

# os.remove("文件名")
os.remove("test_copy.txt")
```

> **注意**：`os.remove()` 删除的文件不会进入回收站，而是**直接删除**。普通用户通常不能从回收站恢复，使用时一定要小心。

#### （3）判断文件是否存在

```python
import os

# os.path.exists() 判断文件或目录是否存在
print(os.path.exists("test.txt"))  # True 或 False
```

### 9.3 目录（文件夹）操作

#### （1）获取当前工作目录

```python
import os

print(os.getcwd())  # 输出当前工作目录的绝对路径
# 例如: /Users/zhangsan/projects/python_demo
```

#### （2）改变当前工作目录

```python
import os

os.chdir("/Users/zhangsan/Desktop")
print(os.getcwd())  # /Users/zhangsan/Desktop
```

#### （3）列出目录下的所有文件和子目录

```python
import os

file_list = os.listdir(".")  # "." 表示当前目录
print(file_list)
# 例如: ['test.txt', 'output.txt', 'data', 'main.py']
```

#### （4）创建目录

```python
import os

# 创建一个新目录
os.mkdir("new_folder")

# 创建多层目录（如果父目录不存在也一并创建）
os.makedirs("level1/level2/level3")
```

#### （5）删除目录

```python
import os

# 删除一个空目录
os.rmdir("new_folder")

# 注意：os.rmdir() 只能删除空目录
# 如果目录中有文件或子目录，会报错
```

如果要删除非空目录，可以使用 `shutil` 模块：
```python
import shutil

shutil.rmtree("level1")  # 递归删除 level1 目录及其所有内容，慎用
```

#### （6）判断是否为文件或目录

```python
import os

print(os.path.isfile("test.txt"))   # True，是文件
print(os.path.isdir("test.txt"))    # False，不是目录

print(os.path.isfile("new_folder")) # False，不是文件
print(os.path.isdir("new_folder"))  # True，是目录
```

### 9.4 os.path 常用方法

`os.path` 是 `os` 模块中专门用于处理路径的子模块，提供了很多实用的方法：

```python
import os

# 拼接路径（推荐使用，能自动处理不同操作系统的路径分隔符）
path = os.path.join("/Users/zhangsan", "documents", "test.txt")
print(path)  # /Users/zhangsan/documents/test.txt

# 获取文件名
print(os.path.basename("/Users/zhangsan/test.txt"))  # test.txt

# 获取目录名
print(os.path.dirname("/Users/zhangsan/test.txt"))   # /Users/zhangsan

# 获取文件扩展名
print(os.path.splitext("test.txt"))  # ('test', '.txt')
```

### 9.5 综合示例：整理文件

下面是一个实际的小案例——列出当前目录下所有的 `.txt` 文件：

```python
import os

# 获取当前目录下的所有内容
file_list = os.listdir(".")

for file_name in file_list:
    # 判断是否是文件（排除目录）
    if os.path.isfile(file_name):
        # 判断扩展名是否为 .txt
        if file_name.endswith(".txt"):
            print(file_name)
```

---

## 第十章 文本文件的编码方式——ASCII 和 UTF-8

### 10.1 计算机如何存储文字

我们知道，计算机底层只认识 0 和 1（二进制）。那么，计算机是怎么存储和显示文字的呢？

答案是通过**编码表（字符集）**——人们制定了一套规则，规定每个字符对应一个唯一的数字编号，计算机存储这个编号的二进制形式，显示时再根据编号找到对应的字符。

### 10.2 ASCII 编码

**ASCII**（American Standard Code for Information Interchange，美国信息交换标准代码）是最早、最基础的编码标准。

#### 主要特点：
- 使用 **1 个字节（8 位）** 来存储一个字符
- 由于最高位固定为 0，实际只用了 **7 位**，所以总共能表示 **128** 个字符（编号 0~127）
- 包含的字符：英文大小写字母（A-Z, a-z）、数字（0-9）、标点符号、以及一些控制字符（如换行、制表符）

#### ASCII 编码表（部分常用字符）：

| 字符 | ASCII 编号 | 二进制    |
| ---- | ---------- | --------- |
| `0`  | 48         | 0011 0000 |
| `A`  | 65         | 0100 0001 |
| `B`  | 66         | 0100 0010 |
| `a`  | 97         | 0110 0001 |
| `b`  | 98         | 0110 0010 |

#### 一些有用的规律：
- 大写字母 A 的编号是 65，B 是 66，……Z 是 90
- 小写字母 a 的编号是 97，b 是 98，……z 是 122
- 大写字母和对应的小写字母之间**相差 32**（比如 A=65，a=97，97-65=32）
- 数字字符 '0' 的编号是 48

#### ASCII 的局限性：

ASCII 只有 128 个字符，只能表示英文字母、数字和少量符号。它无法表示中文、日文、韩文、阿拉伯文等其他语言的文字。

### 10.3 UTF-8 编码

为了解决 ASCII 无法表示全球各种语言文字的问题，Unicode 标准给世界上几乎所有的字符都分配了一个唯一的编号。

而 **UTF-8** 是 Unicode 最常用的一种**实现方式（编码方案）**。

#### UTF-8 的主要特点：

- **变长编码**：不同的字符使用不同长度的字节来存储
  - 英文字母：**1 个字节**（与 ASCII 完全兼容）
  - 常见中文字符：**3 个字节**
  - 某些生僻字符或 emoji：**4 个字节**
- **向后兼容 ASCII**：ASCII 中的 128 个字符，在 UTF-8 中的编码和 ASCII 完全一样
- 目前是**互联网上最主流**的编码方式

#### 举例：

| 字符 | UTF-8 占用字节数 | 说明                    |
| ---- | ---------------- | ----------------------- |
| `A`  | 1 字节           | 英文字母，与 ASCII 一样 |
| `9`  | 1 字节           | 数字字符                |
| `中` | 3 字节           | 常见中文字符            |
| `😀`  | 4 字节           | emoji 表情              |

### 10.4 ASCII 与 UTF-8 的对比

| 特性             | ASCII                     | UTF-8                  |
| ---------------- | ------------------------- | ---------------------- |
| 能表示的字符     | 128个（英文、数字、符号） | 几乎全球所有字符       |
| 每个字符占用空间 | 固定 1 字节               | 1~4 字节不等（变长）   |
| 是否兼容 ASCII   | —                         | 完全兼容               |
| 适用场景         | 纯英文内容                | 多语言内容（推荐使用） |

### 10.5 Python 中查看字符的编码

```python
# ord() 函数：获取字符的 Unicode 编码（十进制数字）
print(ord("A"))   # 65
print(ord("a"))   # 97
print(ord("中"))  # 20013

# chr() 函数：根据 Unicode 编码获取对应的字符
print(chr(65))     # A
print(chr(97))     # a
print(chr(20013))  # 中
```

---

## 第十一章 怎样在 Python 3.x 中使用中文

### 11.1 Python 2.x vs Python 3.x 的编码差异

这里做一个简单的历史背景说明，帮助你理解为什么编码问题在 Python 中这么重要。

- **Python 2.x**：默认使用 **ASCII** 编码。如果代码中直接写中文，会报错 `SyntaxError: Non-ASCII character`。
- **Python 3.x**：源代码文件默认按 **UTF-8** 处理。可以直接在代码中使用中文，不会报错。

由于我们现在学习的是 **Python 3.x**，所以使用中文相对简单很多。但了解背后的原理仍然非常重要。

### 11.2 Python 3.x 中直接使用中文

在 Python 3.x 中，你可以放心地在代码中使用中文：

```python
# 中文变量名（不推荐，但语法上允许）
名字 = "张三"

# 中文字符串（最常见的用法）
name = "张三"
print("你好，" + name)  # 你好，张三

# 中文注释
# 这是一段中文注释，完全没有问题
```

### 11.3 读写包含中文的文件

虽然 Python 3.x 本身支持中文，但在**读写文件**时，仍然可能遇到编码问题。关键在于：**文件本身的编码方式**和**你打开文件时指定的编码方式**要一致。

#### 可能出现的问题

如果你的文件是用 UTF-8 编码保存的，但打开时使用了其他编码（比如 GBK），就会出现乱码或报错。

```python
# 正确的做法：明确指定编码方式
file = open("chinese.txt", "r", encoding="utf-8")
content = file.read()
print(content)
file.close()
```

#### `open()` 函数的 `encoding` 参数

```python
file = open("文件名", "打开方式", encoding="编码方式")
```

常用的编码方式：
- `"utf-8"` ：最常用，推荐
- `"gbk"` ：很多简体中文 Windows 环境常见的本地编码
- `"ascii"` ：只支持英文

#### 在不同操作系统上的默认编码

- **Mac / Linux**：默认编码通常是 UTF-8，一般不会有问题
- **Windows**：默认编码取决于系统区域设置和 Python 运行配置，很多简体中文环境常见的是 **GBK/CP936**，这就是为什么在 Windows 上读取 UTF-8 文件时容易出现乱码

**最佳实践**：如果你能决定文件格式，优先把文本文件统一保存为 UTF-8，并在打开文件时显式指定 `encoding="utf-8"`。如果文件来自外部，就要按它的实际编码来打开。

```python
# UTF-8 文件的推荐写法——明确指定 encoding
file = open("data.txt", "r", encoding="utf-8")
```

### 11.4 写入中文到文件

```python
# 写入中文到文件
file = open("chinese_output.txt", "w", encoding="utf-8")
file.write("你好，世界！\n")
file.write("Python 编程很有趣！\n")
file.close()

# 读取验证
file = open("chinese_output.txt", "r", encoding="utf-8")
print(file.read())
file.close()
```

输出：
```
你好，世界！
Python 编程很有趣！
```

### 11.5 补充：Python 2.x 中使用中文的方法（了解即可）

在 Python 2.x 中，需要在文件的**第一行**添加编码声明：

```python
# -*- coding: utf-8 -*-

print "你好，世界！"
```

或者：
```python
# coding=utf-8

print "你好，世界！"
```

> 现在我们使用 Python 3.x，不需要这个声明了。但如果你看到别人的代码中有这一行，现在你知道它的作用了。

---

## 第十二章 Python 3.x 处理中文字符串

### 12.1 Python 3.x 中字符串的本质

在 Python 3.x 中：
- **`str` 类型**：存储的是 **Unicode 字符**。不管是英文还是中文，都是 `str` 类型。
- **`bytes` 类型**：存储的是**原始的字节数据**。

```python
# str 类型——Unicode字符串
s = "你好"
print(type(s))  # <class 'str'>

# bytes 类型——字节序列
b = b"hello"
print(type(b))  # <class 'bytes'>
```

### 12.2 `str` 和 `bytes` 之间的转换

#### `str` → `bytes`：使用 `encode()` 编码

```python
s = "你好"

# 将字符串编码为 UTF-8 格式的字节序列
b_utf8 = s.encode("utf-8")
print(b_utf8)       # b'\xe4\xbd\xa0\xe5\xa5\xbd'
print(len(b_utf8))  # 6  （这两个常见中文字符各占3个字节，共6字节）

# 将字符串编码为 GBK 格式的字节序列
b_gbk = s.encode("gbk")
print(b_gbk)       # b'\xc4\xe3\xba\xc3'
print(len(b_gbk))  # 4  （GBK中每个中文字符占2个字节，2个字符共4字节）
```

#### `bytes` → `str`：使用 `decode()` 解码

```python
b = b'\xe4\xbd\xa0\xe5\xa5\xbd'

# 将字节序列解码为字符串（必须使用正确的编码方式）
s = b.decode("utf-8")
print(s)  # 你好
```

> **重要**：编码和解码时必须使用**相同的编码方式**，否则会出现乱码或报错。用 UTF-8 编码的字节，必须用 UTF-8 来解码。

### 12.3 中文字符串的长度

在 Python 3.x 中，`len()` 函数对 `str` 类型计算的是**字符数**（不是字节数）：

```python
s1 = "hello"
print(len(s1))  # 5（5个英文字符）

s2 = "你好"
print(len(s2))  # 2（2个中文字符）

s3 = "hello你好"
print(len(s3))  # 7（5个英文 + 2个中文 = 7个字符）
```

如果想知道**字节数**，需要先编码为 `bytes`：
```python
s = "你好"

print(len(s.encode("utf-8")))  # 6 字节（这里两个常见中文字符各占3字节）
print(len(s.encode("gbk")))    # 4 字节（GBK下每个中文2字节）
```

### 12.4 中文字符串的常用操作

中文字符串在 Python 3.x 中和英文字符串的操作完全一致：

```python
s = "我爱Python编程"

# 切片
print(s[0:4])    # 我爱Py
print(s[-2:])    # 编程

# 查找
print(s.find("Python"))   # 2（从索引2开始出现）
print("Python" in s)      # True

# 替换
print(s.replace("Python", "Java"))  # 我爱Java编程

# 分割
info = "张三-18-北京"
parts = info.split("-")
print(parts)  # ['张三', '18', '北京']

# 拼接
words = ["你好", "世界"]
result = " ".join(words)
print(result)  # 你好 世界

# 统计
print(s.count("编"))  # 1
```

### 12.5 实际应用示例

下面是一个综合示例，读取一个中文文本文件并统计其中的中文字符数：

```python
# 打开包含中文的文件
file = open("article.txt", "r", encoding="utf-8")
content = file.read()
file.close()

# 统计中文字符的数量
chinese_count = 0
for char in content:
    # 判断一个字符是否是中文
    # 中文字符的 Unicode 范围大致是 \u4e00 到 \u9fff
    if '\u4e00' <= char <= '\u9fff':
        chinese_count += 1

print(f"文件中共有 {chinese_count} 个中文字符")
```

---

## 总结与补充

### 使用 `with` 语句管理文件（推荐）

在前面的所有示例中，我们都需要手动调用 `file.close()` 来关闭文件。如果程序在 `close()` 之前发生了异常，文件就不会被正确关闭，导致资源泄露。

Python 提供了 `with` 语句来自动管理文件的打开和关闭：

```python
# 使用 with 语句，不需要手动调用 close()
# 当 with 代码块执行完毕后，文件会自动关闭
with open("test.txt", "r", encoding="utf-8") as file:
    content = file.read()
    print(content)

# 执行到这里时，文件已经自动关闭了
# 即使代码块中发生了异常，文件也会被正确关闭
```

这是 Python 中操作文件的**最佳实践**，推荐在实际开发中始终使用 `with` 语句。

用 `with` 语句改写大文件复制的代码：

```python
with open("source.txt", "r", encoding="utf-8") as f_read, \
     open("target.txt", "w", encoding="utf-8") as f_write:

    while True:
        content = f_read.read(1024)
        if not content:
            break
        f_write.write(content)

print("复制完成！")
```

### 知识体系回顾

```
文件操作
├── 文件的概念
│   ├── 文本文件（可读字符，编码存储）
│   └── 二进制文件（原始字节，特定格式）
│
├── 文件操作三步走
│   ├── 打开文件：open()
│   ├── 读写文件：read() / readline() / write()
│   └── 关闭文件：close()（推荐用 with 语句）
│
├── 打开模式
│   ├── "r" 只读（默认）
│   ├── "w" 只写（覆盖）
│   ├── "a" 追加
│   └── 加 "b" 后缀表示二进制模式（rb, wb, ab）
│
├── 文件指针
│   ├── 读写后指针自动后移
│   ├── seek() 移动指针
│   └── tell() 查看指针位置
│
├── 文件复制
│   ├── 小文件：read() 一次性读写
│   └── 大文件：read(n) 循环分批读写
│
├── 文件/目录管理：os 模块
│   ├── 文件：rename / remove / path.exists
│   └── 目录：mkdir / rmdir / listdir / getcwd / chdir
│
└── 文本编码
    ├── ASCII：128个字符，1字节
    ├── UTF-8：全球字符，变长编码（1-4字节）
    ├── open() 时指定 encoding="utf-8"
    └── str ↔ bytes：encode() / decode()
```

---

以上就是 Python 文件操作与文本编码的完整教程。建议你按照章节顺序，在自己的电脑上逐一运行每个代码示例，动手实践是掌握这些知识最有效的方式。
