# Python 字符串与正则表达式完全教程

> **适用对象**：Python 初学者 | **前置要求**：了解 Python 基本变量、print 语句即可
> **学习建议**：每一段代码都请亲手敲一遍，不要复制粘贴，肌肉记忆很重要！

---

## 一、字符串基础

### 1.1 什么是字符串？

在 Python 中，**字符串（string）** 就是一串字符的序列。你可以把它想象成一串珠子，每颗珠子就是一个字符。

```python
# 三种创建方式，效果完全相同
s1 = 'hello'        # 单引号
s2 = "hello"        # 双引号
s3 = '''hello'''    # 三引号（通常用于多行字符串）

print(s1)  # hello
print(s2)  # hello
print(s3)  # hello
```

**单引号和双引号有什么区别？** 功能上没有任何区别，只是在包含引号时可以互相嵌套，避免转义：

```python
# 字符串里有单引号时，外面用双引号
msg1 = "I'm a student"

# 字符串里有双引号时，外面用单引号
msg2 = 'He said "wow"'

print(msg1)  # I'm a student
print(msg2)  # He said "wow"
```

**三引号的妙用**——可以写多行字符串：

```python
poem = '''静夜思
床前明月光，
疑是地上霜。
举头望明月，
低头思故乡。'''

print(poem)
```

输出：

```
静夜思
床前明月光，
疑是地上霜。
举头望明月，
低头思故乡。
```

---

### 1.2 索引（Index）—— 精准定位每一个字符

字符串中的每个字符都有一个"门牌号"，叫做 **索引**，从 **0** 开始编号。

```
字符串:  P   y   t   h   o   n
正向索引: 0   1   2   3   4   5
反向索引:-6  -5  -4  -3  -2  -1
```

```python
s = "Python"

# 正向索引：从左往右，从0开始
print(s[0])   # P
print(s[1])   # y
print(s[5])   # n

# 反向索引：从右往左，从-1开始
print(s[-1])  # n （最后一个字符）
print(s[-2])  # o （倒数第二个）
print(s[-6])  # P （倒数第六个，也就是第一个）
```

**常见错误——索引越界**：

```python
s = "Python"
# print(s[6])  # ❌ IndexError: string index out of range
# "Python" 只有6个字符，索引最大到5
```

> 💡 **记忆技巧**：正向从 0 开始，反向从 -1 开始。`s[0]` 是第一个，`s[-1]` 永远是最后一个。

---

### 1.3 切片（Slicing）—— 截取字符串的一部分

切片是 Python 中非常强大的功能，语法为：

```
s[start:stop:step]
```

- `start`：起始索引（**包含**）
- `stop`：结束索引（**不包含**）
- `step`：步长（默认为 1）

把它想象成"左闭右开"区间：$[start, stop)$

```python
s = "Hello, Python!"

# 基础切片
print(s[0:5])    # Hello     （索引0到4，不包含5）
print(s[7:13])   # Python    （索引7到12）

# 省略写法
print(s[:5])     # Hello     （从开头到索引4）
print(s[7:])     # Python!   （从索引7到末尾）
print(s[:])      # Hello, Python!  （完整复制）

# 带步长
print(s[::2])    # Hlo yhn   （每隔一个取一个）
print(s[0:5:2])  # Hlo       （在前5个字符中，每隔一个取一个）
```

**经典用法——字符串反转**：

```python
s = "Hello"
print(s[::-1])   # olleH  （步长为-1，从右往左取）
```

**更多切片练习**：

```python
s = "0123456789"

print(s[2:8])     # 234567
print(s[2:8:2])   # 246
print(s[8:2:-1])  # 876543   （反向切片，从8到3）
print(s[::-1])    # 9876543210
print(s[::3])     # 0369
```

> 💡 **切片不会报索引越界错误**，即使超出范围也不会出错，Python 会自动处理：
>
> ```python
> s = "Hello"
> print(s[0:100])  # Hello （超出部分自动忽略）
> print(s[-100:3]) # Hel   （超出部分自动忽略）
> ```

---

### 1.4 字符串的不可变性

这是一个**非常重要**的概念：Python 中的字符串一旦创建，就**不能修改**其中的任何字符。

```python
s = "Hello"
# s[0] = 'h'  # ❌ TypeError: 'str' object does not support item assignment
```

那我们平时"修改"字符串是怎么回事？实际上是**创建了一个新的字符串**：

```python
s = "Hello"
s = "hello"  # 这不是修改，而是让变量 s 指向了一个全新的字符串

# 正确的"修改"方式：创建新字符串
s = "Hello"
s_new = 'h' + s[1:]   # 拼接出一个新字符串
print(s_new)           # hello
print(s)               # Hello （原字符串不变！）
```

**用图来理解**：

```
第一步：s = "Hello"
  s ----> "Hello"

第二步：s = "hello"
  s ----> "hello"     （s 指向了新的字符串）
          "Hello"     （旧字符串没人用了，会被自动回收）
```

> 💡 **为什么字符串要设计成不可变的？** 这是为了安全和效率。不可变意味着字符串可以被安全地共享和缓存，Python 底层做了很多优化。

---

### 1.5 字符串拼接与重复

```python
# 用 + 拼接
first = "Hello"
second = "World"
result = first + " " + second
print(result)  # Hello World

# 用 * 重复
line = "-" * 30
print(line)  # ------------------------------

star = "★ " * 5
print(star)  # ★ ★ ★ ★ ★ 

# 注意：字符串不能和数字直接拼接
age = 18
# print("我今年" + age + "岁")  # ❌ TypeError
print("我今年" + str(age) + "岁")  # ✅ 需要先转换为字符串
```

---

### 1.6 常用字符串方法（重点）

方法就是"绑定在对象上的函数"，通过 `字符串.方法名()` 来调用。

#### 1.6.1 `split()` —— 分割字符串，得到列表

```python
# 默认按空格分割
sentence = "I love Python programming"
words = sentence.split()
print(words)  # ['I', 'love', 'Python', 'programming']

# 按指定字符分割
date = "2026-04-09"
parts = date.split("-")
print(parts)  # ['2026', '04', '09']

# 按逗号分割
csv_line = "张三,25,北京,工程师"
info = csv_line.split(",")
print(info)  # ['张三', '25', '北京', '工程师']

# 限制分割次数
text = "a-b-c-d-e"
print(text.split("-"))      # ['a', 'b', 'c', 'd', 'e']
print(text.split("-", 2))   # ['a', 'b', 'c-d-e']  只分割前2次
```

#### 1.6.2 `join()` —— 将列表拼接为字符串（split 的逆操作）

```python
# 用空格拼接
words = ['I', 'love', 'Python']
sentence = " ".join(words)
print(sentence)  # I love Python

# 用逗号拼接
items = ['苹果', '香蕉', '橘子']
result = ",".join(items)
print(result)  # 苹果,香蕉,橘子

# 用空字符串拼接（把列表中的字符连起来）
letters = ['P', 'y', 't', 'h', 'o', 'n']
word = "".join(letters)
print(word)  # Python

# 用换行符拼接
lines = ['第一行', '第二行', '第三行']
text = "\n".join(lines)
print(text)
# 第一行
# 第二行
# 第三行
```

> 💡 **`split()` 和 `join()` 是一对互逆操作**，这在文本处理中极其常用：
>
> ```python
> s = "Hello World"
> s.split()              # ['Hello', 'World']    拆开
> " ".join(s.split())    # 'Hello World'          拼回来
> ```

#### 1.6.3 `replace()` —— 替换子字符串

```python
s = "Hello World"
print(s.replace("World", "Python"))  # Hello Python

# 替换多个字符
msg = "我喜欢Java，Java是最好的语言"
print(msg.replace("Java", "Python"))  # 我喜欢Python，Python是最好的语言

# 限制替换次数
msg = "aaa-bbb-aaa-bbb"
print(msg.replace("aaa", "xxx"))      # xxx-bbb-xxx-bbb （全部替换）
print(msg.replace("aaa", "xxx", 1))   # xxx-bbb-aaa-bbb （只替换第一个）
```

> ⚠️ **记住**：replace() 不会修改原字符串（因为字符串不可变），而是返回一个新的字符串。

```python
s = "Hello"
s.replace("H", "h")   # 这行代码执行了，但结果没有保存！
print(s)               # Hello （原字符串没变）

s = s.replace("H", "h")  # ✅ 必须重新赋值
print(s)                  # hello
```

#### 1.6.4 `strip()` —— 去除首尾空白字符

```python
s = "   Hello World   "
print(s.strip())    # "Hello World"    （去除两端空白）
print(s.lstrip())   # "Hello World   " （只去除左边）
print(s.rstrip())   # "   Hello World" （只去除右边）

# 去除指定字符
s = "###Hello###"
print(s.strip("#"))   # Hello

# 实际应用：处理用户输入
username = input("请输入用户名：")  # 用户可能不小心多打空格
username = username.strip()         # 去掉首尾空格，再处理
```

> 💡 `strip()` 在处理文件读取时特别有用，因为每行末尾通常会有换行符 `\n`：
>
> ```python
> line = "Hello World\n"
> print(line.strip())  # Hello World （去掉了末尾的换行符）
> ```

#### 1.6.5 `find()` 和 `count()` —— 查找与计数

```python
s = "Hello, World! Hello, Python!"

# find() 返回第一次出现的索引，找不到返回 -1
print(s.find("Hello"))    # 0
print(s.find("Python"))   # 21
print(s.find("Java"))     # -1 （没找到）

# index() 和 find() 类似，但找不到会报错
# print(s.index("Java"))  # ❌ ValueError

# count() 统计出现次数
print(s.count("Hello"))   # 2
print(s.count("o"))       # 3
print(s.count("xyz"))     # 0
```

#### 1.6.6 `startswith()` 和 `endswith()` —— 判断开头结尾

```python
filename = "report_2026.pdf"

print(filename.startswith("report"))  # True
print(filename.endswith(".pdf"))      # True
print(filename.endswith(".txt"))      # False

# 实际应用：过滤文件类型
files = ["a.py", "b.txt", "c.py", "d.pdf"]
python_files = [f for f in files if f.endswith(".py")]
print(python_files)  # ['a.py', 'c.py']
```

---

### 1.7 字符串格式化（重点中的重点）

在实际开发中，我们经常需要将变量"嵌入"到字符串中。Python 提供了几种方式，**f-string 是最推荐的**。

#### 1.7.1 f-string（Python 3.6+，强烈推荐）

在字符串前加 `f` 或 `F`，就可以在 `{}` 中直接写 Python 表达式：

```python
name = "小明"
age = 18

# 基本用法
print(f"我叫{name}，今年{age}岁。")
# 输出：我叫小明，今年18岁。

# 可以在 {} 中写表达式
print(f"明年我就{age + 1}岁了。")
# 输出：明年我就19岁了。

# 可以调用方法
msg = "hello"
print(f"大写：{msg.upper()}")
# 输出：大写：HELLO

# 可以做计算
price = 99.5
quantity = 3
print(f"总价：{price * quantity}元")
# 输出：总价：298.5元
```

**f-string 的格式控制**——这是 f-string 的精髓：

```python
import math

# 控制小数位数
pi = math.pi
print(f"圆周率：{pi}")        # 圆周率：3.141592653589793
print(f"圆周率：{pi:.2f}")    # 圆周率：3.14   （保留2位小数）
print(f"圆周率：{pi:.4f}")    # 圆周率：3.1416 （保留4位小数）

# 百分比格式
rate = 0.8567
print(f"合格率：{rate:.1%}")   # 合格率：85.7%
print(f"合格率：{rate:.2%}")   # 合格率：85.67%

# 千位分隔符
population = 1400000000
print(f"中国人口：{population:,}")   # 中国人口：1,400,000,000

# 对齐与填充
name = "Python"
print(f"|{name:<20}|")   # |Python              | 左对齐，宽度20
print(f"|{name:>20}|")   # |              Python| 右对齐
print(f"|{name:^20}|")   # |       Python       | 居中
print(f"|{name:*^20}|")  # |*******Python*******| 居中，用*填充
print(f"|{name:-<20}|")  # |Python--------------| 左对齐，用-填充

# 整数补零
for i in range(1, 6):
    print(f"文件_{i:03d}.txt")
# 文件_001.txt
# 文件_002.txt
# 文件_003.txt
# 文件_004.txt
# 文件_005.txt
```

**f-string 中使用条件表达式**：

```python
score = 85
result = f"成绩：{score}分，{'及格' if score >= 60 else '不及格'}"
print(result)  # 成绩：85分，及格

# 在循环中使用
students = [("小明", 92), ("小红", 55), ("小刚", 78)]
for name, score in students:
    print(f"{name}: {score:3d}分 - {'✓ 及格' if score >= 60 else '✗ 不及格'}")
# 小明:  92分 - ✓ 及格
# 小红:  55分 - ✗ 不及格
# 小刚:  78分 - ✓ 及格
```

#### 1.7.2 其他格式化方式（了解即可）

```python
name = "小明"
age = 18

# 方式1：% 格式化（C语言风格，最古老）
print("我叫%s，今年%d岁。" % (name, age))

# 方式2：str.format() 方法
print("我叫{}，今年{}岁。".format(name, age))

# 方式3：f-string（推荐！）
print(f"我叫{name}，今年{age}岁。")
```

> 💡 **建议**：在 Python 3.6 以上版本中，**统一使用 f-string** 就够了，其他方式只需在阅读别人代码时能看懂即可。

---

## 二、字符串进阶

### 2.1 大小写转换方法

```python
s = "Hello, World!"

print(s.upper())       # HELLO, WORLD!    全部大写
print(s.lower())       # hello, world!    全部小写
print(s.title())       # Hello, World!    每个单词首字母大写
print(s.capitalize())  # Hello, world!    仅第一个字母大写
print(s.swapcase())    # hELLO, wORLD!    大小写互换

# 实际应用：不区分大小写的比较
user_input = "Yes"
if user_input.lower() == "yes":
    print("用户选择了是")
```

---

### 2.2 判断方法（返回 True / False）

这些方法非常适合用来做数据验证：

```python
# 判断内容类型
print("12345".isdigit())    # True  是否全是数字
print("123.45".isdigit())   # False 小数点不算数字
print("abc".isalpha())      # True  是否全是字母
print("abc123".isalnum())   # True  是否全是字母或数字
print("   ".isspace())      # True  是否全是空白字符

# 判断大小写
print("HELLO".isupper())    # True  是否全大写
print("hello".islower())    # True  是否全小写
print("Hello World".istitle())  # True  是否标题格式
```

**实际应用——简单的输入验证**：

```python
# 验证手机号（简化版：11位数字）
phone = "13800138000"
if phone.isdigit() and len(phone) == 11:
    print("手机号格式正确")
else:
    print("手机号格式错误")

# 验证用户名（只允许字母和数字）
username = "user123"
if username.isalnum() and len(username) >= 3:
    print("用户名有效")
else:
    print("用户名无效，只能包含字母和数字，至少3个字符")

# 验证密码强度
password = "Abc12345"
has_upper = any(c.isupper() for c in password)
has_lower = any(c.islower() for c in password)
has_digit = any(c.isdigit() for c in password)
long_enough = len(password) >= 8

if all([has_upper, has_lower, has_digit, long_enough]):
    print("密码强度：强")
else:
    print("密码太弱，需要包含大写、小写、数字，且至少8位")
```

---

### 2.3 转义字符

有些字符无法直接输入到字符串中，需要用反斜杠 `\` 来"转义"：

| 转义字符 | 含义          | 示例                   |
| -------- | ------------- | ---------------------- |
| `\n`     | 换行          | `"第一行\n第二行"`     |
| `\t`     | 制表符（Tab） | `"姓名\t年龄"`         |
| `\\`     | 反斜杠本身    | `"C:\\Users\\Desktop"` |
| `\'`     | 单引号        | `'I\'m fine'`          |
| `\"`     | 双引号        | `"He said \"hi\""`     |

```python
# 换行符
print("你好\n世界")
# 你好
# 世界

# 制表符（对齐用）
print("姓名\t年龄\t城市")
print("张三\t25\t北京")
print("李四\t30\t上海")
# 姓名    年龄    城市
# 张三    25      北京
# 李四    30      上海

# 反斜杠
print("文件路径：C:\\Users\\Desktop\\file.txt")
# 文件路径：C:\Users\Desktop\file.txt
```

**原始字符串（Raw String）**——在字符串前加 `r`，所有反斜杠都不转义：

```python
# 普通字符串
print("C:\new\test")    # C: 这里换行了，因为 \n 被转义了
                         # ew	est  （\t 也被转义了）

# 原始字符串
print(r"C:\new\test")   # C:\new\test （所有内容原样输出）

# 在正则表达式中特别有用（后面会详细讲）
import re
pattern = r"\d+"  # 推荐写法
```

---

### 2.4 字符串编码

计算机底层只认识数字（0和1），字符串需要"编码"才能存储和传输。

```python
# ord() 获取字符的 Unicode 编码值
print(ord('A'))    # 65
print(ord('a'))    # 97
print(ord('0'))    # 48
print(ord('中'))   # 20013

# chr() 将编码值转回字符
print(chr(65))     # A
print(chr(97))     # a
print(chr(20013))  # 中

# 编码与解码
s = "你好Python"
encoded = s.encode("utf-8")     # 字符串 → 字节串
print(encoded)                   # b'\xe4\xbd\xa0\xe5\xa5\xbdPython'
print(type(encoded))             # <class 'bytes'>

decoded = encoded.decode("utf-8")  # 字节串 → 字符串
print(decoded)                      # 你好Python
```

> 💡 **实际建议**：在 Python 3 中，字符串默认使用 Unicode 编码，绝大多数情况下你不需要手动处理编码。只有在读写文件、网络传输时才需要注意编码问题，记住 **用 UTF-8 就对了**。

---

## 三、正则表达式（re 模块）

### 3.1 什么是正则表达式？

**正则表达式（Regular Expression，简称 regex 或 re）** 是一种用特殊语法描述"文本模式"的工具。

打个比方：

- `split(",")` 只能按固定的逗号分割
- 正则表达式可以说"帮我找到所有以数字开头、后面跟着字母的内容"

它就像一个**超级搜索框**，可以用"模式"来匹配文本。

```python
import re  # 使用正则表达式需要导入 re 模块

# 一个简单的例子：找出所有数字
text = "我今年18岁，身高175cm，体重70kg"
numbers = re.findall(r"\d+", text)
print(numbers)  # ['18', '175', '70']
```

---

### 3.2 基本语法——元字符

正则表达式有一套自己的"语法符号"，叫做**元字符**。我们一个一个来学。

#### 3.2.1 普通字符——直接匹配自身

```python
import re

text = "Hello World, Hello Python"
result = re.findall(r"Hello", text)
print(result)  # ['Hello', 'Hello']
```

#### 3.2.2 `.` —— 匹配任意单个字符（除了换行符）

```python
text = "cat cot cut c_t c9t"
result = re.findall(r"c.t", text)
print(result)  # ['cat', 'cot', 'cut', 'c_t', 'c9t']

# .只匹配一个字符
text = "ct cat caat"
result = re.findall(r"c.t", text)
print(result)  # ['cat']  （ct没有中间字符，caat有两个中间字符，都不匹配）
```

#### 3.2.3 `\d` `\w` `\s` —— 预定义字符集

| 符号 | 含义                                 | 等价写法         |
| ---- | ------------------------------------ | ---------------- |
| `\d` | 一个数字                             | `[0-9]`          |
| `\D` | 一个非数字                           | `[^0-9]`         |
| `\w` | 一个"单词字符"（字母、数字、下划线） | `[a-zA-Z0-9_]`   |
| `\W` | 一个非单词字符                       | `[^a-zA-Z0-9_]`  |
| `\s` | 一个空白字符（空格、Tab、换行等）    | `[ \t\n\r\f\v]`  |
| `\S` | 一个非空白字符                       | `[^ \t\n\r\f\v]` |

```python
import re

text = "电话：010-12345678，手机：138-0013-8000"

# \d 匹配数字
print(re.findall(r"\d", text))    
# ['0', '1', '0', '1', '2', '3', '4', '5', '6', '7', '8', '1', '3', '8', '0', '0', '1', '3', '8', '0', '0', '0']

# \d+ 匹配连续数字（+号表示一个或多个，后面会讲）
print(re.findall(r"\d+", text))   
# ['010', '12345678', '138', '0013', '8000']

# \w 匹配单词字符
text2 = "user_01@test.com"
print(re.findall(r"\w+", text2))  
# ['user_01', 'test', 'com']
```

#### 3.2.4 量词 —— `*` `+` `?` `{n,m}`

量词用来指定前一个字符/模式出现的次数：

| 量词    | 含义      | 举例                                            |
| ------- | --------- | ----------------------------------------------- |
| `*`     | 0次或多次 | `ab*` 匹配 `a`, `ab`, `abb`, `abbb`...          |
| `+`     | 1次或多次 | `ab+` 匹配 `ab`, `abb`, `abbb`...（不匹配 `a`） |
| `?`     | 0次或1次  | `ab?` 匹配 `a` 或 `ab`                          |
| `{n}`   | 恰好 n 次 | `a{3}` 匹配 `aaa`                               |
| `{n,}`  | 至少 n 次 | `a{2,}` 匹配 `aa`, `aaa`, `aaaa`...             |
| `{n,m}` | n 到 m 次 | `a{2,4}` 匹配 `aa`, `aaa`, `aaaa`               |

```python
import re

# * : 0次或多次
print(re.findall(r"go*d", "gd god good goood"))
# ['gd', 'god', 'good', 'goood']

# + : 1次或多次
print(re.findall(r"go+d", "gd god good goood"))
# ['god', 'good', 'goood']   （gd 不匹配，因为至少要1个o）

# ? : 0次或1次
print(re.findall(r"colou?r", "color colour"))
# ['color', 'colour']   （u出现0次或1次都行）

# {n} : 恰好n次
print(re.findall(r"\d{3}", "1 12 123 1234"))
# ['123', '123']   （1234中的前3位也匹配）

# {n,m} : n到m次
print(re.findall(r"\d{2,4}", "1 12 123 1234 12345"))
# ['12', '123', '1234', '1234']
```

**贪婪 vs 非贪婪匹配**：

默认情况下，量词是"贪婪"的，会尽可能多地匹配。在量词后加 `?` 可以变成"非贪婪"模式：

```python
import re

text = "<h1>标题</h1>"

# 贪婪匹配（默认）：尽可能多
print(re.findall(r"<.+>", text))
# ['<h1>标题</h1>']  （从第一个<匹配到最后一个>）

# 非贪婪匹配：尽可能少
print(re.findall(r"<.+?>", text))
# ['<h1>', '</h1>']   （遇到第一个>就停止）
```

#### 3.2.5 `[]` —— 字符集合

方括号 `[]` 定义一个字符集合，匹配其中的**任意一个**字符：

```python
import re

# [abc] 匹配a或b或c
print(re.findall(r"[abc]", "apple banana cherry"))
# ['a', 'b', 'a', 'a', 'a', 'c']

# [a-z] 匹配任意小写字母
print(re.findall(r"[a-z]+", "Hello World 123"))
# ['ello', 'orld']

# [A-Za-z] 匹配任意字母
print(re.findall(r"[A-Za-z]+", "Hello World 123"))
# ['Hello', 'World']

# [0-9] 等价于 \d
print(re.findall(r"[0-9]+", "abc123def456"))
# ['123', '456']

# [^...] 取反：匹配不在集合中的字符
print(re.findall(r"[^0-9]+", "abc123def456"))
# ['abc', 'def']   （匹配非数字部分）

# 匹配元音字母
print(re.findall(r"[aeiou]", "Hello World"))
# ['e', 'o', 'o']
```

#### 3.2.6 `()` —— 分组捕获

圆括号 `()` 用来将模式分组，并且可以提取其中的内容：

```python
import re

# 基本分组
text = "2026-04-09"
match = re.search(r"(\d{4})-(\d{2})-(\d{2})", text)
if match:
    print(match.group())   # 2026-04-09 （完整匹配）
    print(match.group(1))  # 2026       （第1个括号）
    print(match.group(2))  # 04         （第2个括号）
    print(match.group(3))  # 09         （第3个括号）

# findall 配合分组——只返回括号中的内容
text = "张三:90分, 李四:85分, 王五:92分"
result = re.findall(r"(\w+):(\d+)分", text)
print(result)  # [('张三', '90'), ('李四', '85'), ('王五', '92')]
```

#### 3.2.7 `^` 和 `$` —— 锚点

```python
import re

# ^ 匹配字符串开头
print(re.findall(r"^Hello", "Hello World"))   # ['Hello']
print(re.findall(r"^Hello", "Say Hello"))     # []（Hello不在开头）

# $ 匹配字符串结尾
print(re.findall(r"World$", "Hello World"))   # ['World']
print(re.findall(r"World$", "World Peace"))   # []（World不在结尾）

# ^ 和 $ 配合使用：精确匹配整个字符串
print(bool(re.match(r"^\d{11}$", "13800138000")))  # True  （11位数字）
print(bool(re.match(r"^\d{11}$", "1380013800")))    # False （只有10位）
print(bool(re.match(r"^\d{11}$", "1380013800a")))   # False （含字母）
```

#### 3.2.8 `|` —— 或操作

```python
import re

text = "I have a cat and a dog"
print(re.findall(r"cat|dog", text))  # ['cat', 'dog']

# 配合分组使用
text = "image.jpg photo.png icon.gif document.pdf"
print(re.findall(r"\w+\.(?:jpg|png|gif)", text))
# ['image.jpg', 'photo.png', 'icon.gif']
# (?:...) 是非捕获分组，表示分组但不提取
```

---

### 3.3 常用语法速查表

| 语法     | 含义     | 示例模式   | 匹配示例      |
| -------- | -------- | ---------- | ------------- |
| `.`      | 任意字符 | `a.c`      | abc, a1c, a_c |
| `\d`     | 数字     | `\d{3}`    | 123, 456      |
| `\w`     | 单词字符 | `\w+`      | hello, user_1 |
| `\s`     | 空白     | `a\sb`     | a b           |
| `*`      | 0+次     | `ab*`      | a, ab, abb    |
| `+`      | 1+次     | `ab+`      | ab, abb       |
| `?`      | 0或1次   | `ab?`      | a, ab         |
| `{n,m}`  | n到m次   | `\d{3,5}`  | 123, 12345    |
| `[abc]`  | 字符集   | `[aeiou]`  | a, e, i       |
| `[^abc]` | 取反     | `[^0-9]`   | a, _, @       |
| `(...)`  | 捕获分组 | `(\d+)`    | 提取数字      |
| `^`      | 开头     | `^Hello`   | Hello...      |
| `$`      | 结尾     | `end$`     | ...end        |
| `\|`     | 或       | `cat\|dog` | cat, dog      |

---

### 3.4 常用函数

#### 3.4.1 `re.search()` —— 搜索第一个匹配

```python
import re

text = "我的邮箱是 test@example.com，备用邮箱是 backup@gmail.com"

# search 只返回第一个匹配
match = re.search(r"\w+@\w+\.\w+", text)
if match:
    print(match.group())  # test@example.com
    print(match.start())  # 5  （匹配开始的索引位置）
    print(match.end())    # 21 （匹配结束的索引位置）
    print(match.span())   # (5, 21)
```

#### 3.4.2 `re.findall()` —— 找出所有匹配

```python
import re

text = "我的邮箱是 test@example.com，备用邮箱是 backup@gmail.com"

# findall 返回所有匹配的列表
emails = re.findall(r"\w+@\w+\.\w+", text)
print(emails)  # ['test@example.com', 'backup@gmail.com']

# 有分组时，返回分组内容
pairs = re.findall(r"(\w+)@(\w+\.\w+)", text)
print(pairs)  # [('test', 'example.com'), ('backup', 'gmail.com')]
```

#### 3.4.3 `re.sub()` —— 替换匹配内容

```python
import re

# 基本替换
text = "我的电话是 13800138000，另一个是 13900139000"
result = re.sub(r"\d{11}", "***已隐藏***", text)
print(result)  
# 我的电话是 ***已隐藏***，另一个是 ***已隐藏***

# 只替换指定次数
result = re.sub(r"\d{11}", "***", text, count=1)
print(result)
# 我的电话是 ***，另一个是 13900139000

# 使用函数进行复杂替换
def mask_phone(match):
    phone = match.group()
    return phone[:3] + "****" + phone[7:]

text = "联系方式：13800138000"
result = re.sub(r"\d{11}", mask_phone, text)
print(result)  # 联系方式：138****8000
```

#### 3.4.4 `re.match()` —— 从字符串开头匹配

```python
import re

# match 只从字符串开头匹配
print(re.match(r"\d+", "123abc"))   # <re.Match object; span=(0, 3), match='123'>
print(re.match(r"\d+", "abc123"))   # None （开头不是数字，匹配失败）

# 对比 search（在整个字符串中搜索）
print(re.search(r"\d+", "abc123"))  # <re.Match object; span=(3, 6), match='123'>
```

> 💡 **`match` vs `search`**：
> - `match`：只看开头，"这个字符串是不是以...开头？"
> - `search`：全文搜索，"这个字符串里有没有...？"

#### 3.4.5 `re.split()` —— 按模式分割

```python
import re

# 按多种分隔符分割
text = "苹果,香蕉;橘子 西瓜|葡萄"
result = re.split(r"[,;| ]+", text)
print(result)  # ['苹果', '香蕉', '橘子', '西瓜', '葡萄']

# 对比字符串的split：只能按固定字符分割
print(text.split(","))  # ['苹果', '香蕉;橘子 西瓜|葡萄']  只能按逗号
```

---

### 3.5 简单匹配案例

#### 案例 1：匹配邮箱

```python
import re

text = """
请联系以下邮箱：
张三: zhangsan@company.com
李四: lisi_01@gmail.com
王五: wangwu@pku.edu.cn
客服: support@my-company.org
"""

# 简单版邮箱正则
pattern = r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+"
emails = re.findall(pattern, text)
for email in emails:
    print(email)
# zhangsan@company.com
# lisi_01@gmail.com
# wangwu@pku.edu.cn
# support@my-company.org
```

#### 案例 2：匹配手机号

```python
import re

text = "张三 13812345678 李四 15987654321 座机 010-88886666"

# 匹配手机号（1开头，11位数字）
phones = re.findall(r"1[3-9]\d{9}", text)
print(phones)  # ['13812345678', '15987654321']
```

#### 案例 3：匹配日期

```python
import re

text = "项目开始日期2026-04-09，截止日期2026/12/31，备注日期20261231"

# 匹配 YYYY-MM-DD 或 YYYY/MM/DD 格式
dates = re.findall(r"\d{4}[-/]\d{2}[-/]\d{2}", text)
print(dates)  # ['2026-04-09', '2026/12/31']

# 提取年月日
for d in dates:
    match = re.search(r"(\d{4})[-/](\d{2})[-/](\d{2})", d)
    print(f"年:{match.group(1)} 月:{match.group(2)} 日:{match.group(3)}")
# 年:2026 月:04 日:09
# 年:2026 月:12 日:31
```

#### 案例 4：匹配 IP 地址

```python
import re

text = "服务器IP：192.168.1.100，网关：192.168.1.1，DNS：8.8.8.8"

# 简单版（可能匹配到不合法的IP，但基本够用）
ips = re.findall(r"\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}", text)
print(ips)  # ['192.168.1.100', '192.168.1.1', '8.8.8.8']
```

#### 案例 5：提取 HTML 标签中的内容

```python
import re

html = "<title>Python教程</title><p>这是一个段落</p><a>链接文字</a>"

# 提取所有标签中的文字
contents = re.findall(r"<\w+>(.*?)</\w+>", html)
print(contents)  # ['Python教程', '这是一个段落', '链接文字']

# 同时提取标签名和内容
pairs = re.findall(r"<(\w+)>(.*?)</\1>", html)
print(pairs)  # [('title', 'Python教程'), ('p', '这是一个段落'), ('a', '链接文字')]
# \1 是反向引用，表示和第1个分组匹配的内容一致
```

---

## 四、列表与字符串结合

### 4.1 `split()` ↔ `join()` 双向转换

这对"黄金搭档"是文本处理中最常见的操作模式：

```python
# 场景1：清理多余空格
text = "  Hello   World    Python  "
clean = " ".join(text.split())
print(clean)  # "Hello World Python"
# 原理：split() 默认按任意空白分割且自动去掉首尾空白 → join用单个空格拼回

# 场景2：交换姓名顺序
name = "张 三"
parts = name.split()
reversed_name = " ".join(reversed(parts))
print(reversed_name)  # "三 张"

# 场景3：CSV数据转换
csv_line = "张三,25,北京"
parts = csv_line.split(",")
tsv_line = "\t".join(parts)
print(tsv_line)  # 张三	25	北京

# 场景4：路径处理
path = "/home/user/documents/file.txt"
parts = path.split("/")
print(parts)      # ['', 'home', 'user', 'documents', 'file.txt']
filename = parts[-1]
print(filename)   # file.txt
```

---

### 4.2 列表推导式处理文本

**列表推导式**是 Python 中非常优雅的语法，可以用一行代码完成对列表的过滤和转换：

基本语法：`[表达式 for 变量 in 可迭代对象 if 条件]`

```python
# 基础：将字符串列表全部转大写
words = ["hello", "world", "python"]
upper_words = [w.upper() for w in words]
print(upper_words)  # ['HELLO', 'WORLD', 'PYTHON']

# 过滤：只保留长度大于3的单词
words = ["I", "am", "a", "Python", "programmer"]
long_words = [w for w in words if len(w) > 3]
print(long_words)  # ['Python', 'programmer']

# 转换+过滤：提取数字字符串并转为整数
data = ["123", "abc", "456", "def", "789"]
numbers = [int(x) for x in data if x.isdigit()]
print(numbers)  # [123, 456, 789]

# 处理每一行文本
raw_lines = ["  Hello  \n", "  World  \n", "  Python  \n"]
clean_lines = [line.strip() for line in raw_lines]
print(clean_lines)  # ['Hello', 'World', 'Python']

# 提取首字母
names = ["Alice", "Bob", "Charlie", "David"]
initials = [name[0] for name in names]
print(initials)        # ['A', 'B', 'C', 'D']
print("".join(initials))  # ABCD
```

**更多实用示例**：

```python
# 批量文件重命名逻辑
files = ["report.txt", "data.csv", "image.png", "note.txt"]
txt_files = [f for f in files if f.endswith(".txt")]
print(txt_files)  # ['report.txt', 'note.txt']

# 将 "key=value" 格式解析为字典
pairs = ["name=张三", "age=25", "city=北京"]
result = dict(p.split("=") for p in pairs)
print(result)  # {'name': '张三', 'age': '25', 'city': '北京'}

# 句子中每个单词的长度
sentence = "The quick brown fox jumps over the lazy dog"
word_lengths = [(w, len(w)) for w in sentence.split()]
print(word_lengths)
# [('The', 3), ('quick', 5), ('brown', 5), ('fox', 3), ('jumps', 5), 
#  ('over', 4), ('the', 3), ('lazy', 4), ('dog', 3)]
```

---

### 4.3 列表常用方法补充

这些方法在文本处理中经常用到：

```python
# ========== append() 添加元素 ==========
results = []
for word in "Hello World Python".split():
    if len(word) > 4:
        results.append(word)
print(results)  # ['Hello', 'World', 'Python']

# ========== extend() 批量添加 ==========
all_words = []
sentences = ["Hello World", "Python is great"]
for s in sentences:
    all_words.extend(s.split())
print(all_words)  # ['Hello', 'World', 'Python', 'is', 'great']

# ========== sort() 排序 ==========
words = ["banana", "apple", "cherry", "date"]
words.sort()
print(words)  # ['apple', 'banana', 'cherry', 'date']（字母顺序）

words.sort(key=len)
print(words)  # ['date', 'apple', 'banana', 'cherry']（按长度排序）

words.sort(key=len, reverse=True)
print(words)  # ['cherry', 'banana', 'apple', 'date']（按长度降序）

# ========== sorted() 不修改原列表 ==========
nums = [3, 1, 4, 1, 5, 9]
new_nums = sorted(nums)
print(nums)      # [3, 1, 4, 1, 5, 9]  （原列表不变）
print(new_nums)  # [1, 1, 3, 4, 5, 9]  （返回新列表）

# ========== 去重但保持顺序 ==========
words = ["apple", "banana", "apple", "cherry", "banana"]
seen = []
for w in words:
    if w not in seen:
        seen.append(w)
print(seen)  # ['apple', 'banana', 'cherry']
```

---

## 五、集合（set）应用

### 5.1 什么是集合？

集合（set）有三个核心特性：

1. **无序**：没有索引，不能通过 `s[0]` 访问
2. **不重复**：自动去除重复元素
3. **可变**：可以添加或删除元素

```python
# 创建集合
s1 = {1, 2, 3, 4, 5}
s2 = set([1, 2, 2, 3, 3, 3])  # 从列表创建，自动去重
print(s2)  # {1, 2, 3}

# 注意：空集合必须用 set()，不能用 {}
empty_set = set()     # ✅ 空集合
empty_dict = {}       # ⚠️ 这是空字典！
print(type(empty_set))   # <class 'set'>
print(type(empty_dict))  # <class 'dict'>
```

---

### 5.2 去重——集合最常见的用途

```python
# 列表去重
names = ["张三", "李四", "张三", "王五", "李四", "张三"]
unique_names = list(set(names))
print(unique_names)  # ['王五', '张三', '李四'] （顺序可能不同！）

# 如果需要保持原顺序去重
names = ["张三", "李四", "张三", "王五", "李四", "张三"]
unique_ordered = list(dict.fromkeys(names))
print(unique_ordered)  # ['张三', '李四', '王五'] （保持首次出现的顺序）

# 字符串中的不重复字符
text = "hello world"
unique_chars = set(text)
print(unique_chars)  # {'h', 'e', 'l', 'o', ' ', 'w', 'r', 'd'}
print(f"不重复字符数：{len(unique_chars)}")  # 8

# 统计一段文本中有多少个不同的单词
article = "the cat sat on the mat the cat ate the rat"
words = article.split()
unique_words = set(words)
print(f"总词数：{len(words)}")         # 10
print(f"不重复词数：{len(unique_words)}")  # 6
print(f"不重复的词：{unique_words}")
# {'cat', 'on', 'ate', 'sat', 'the', 'mat', 'rat'}
```

---

### 5.3 集合运算——交集、并集、差集

这是集合的"超能力"，非常适合用来做数据分析和比较。

```python
# 两个班级的选课情况
class_a = {"数学", "物理", "化学", "英语", "编程"}
class_b = {"数学", "英语", "历史", "地理", "编程"}

# 交集（两个班都选了的课）
both = class_a & class_b                  # 运算符写法
both = class_a.intersection(class_b)      # 方法写法
print(f"两个班都选了：{both}")  
# {'数学', '英语', '编程'}

# 并集（所有被选过的课）
all_courses = class_a | class_b           # 运算符写法
all_courses = class_a.union(class_b)      # 方法写法
print(f"所有课程：{all_courses}")
# {'数学', '物理', '化学', '英语', '编程', '历史', '地理'}

# 差集（A班选了但B班没选的）
only_a = class_a - class_b               # 运算符写法
only_a = class_a.difference(class_b)     # 方法写法
print(f"仅A班选了：{only_a}")  
# {'物理', '化学'}

only_b = class_b - class_a
print(f"仅B班选了：{only_b}")  
# {'历史', '地理'}

# 对称差集（只被一个班选了的课，即"不共同"的课）
either = class_a ^ class_b
either = class_a.symmetric_difference(class_b)
print(f"仅一个班选了：{either}")
# {'物理', '化学', '历史', '地理'}
```

**图解集合运算**：

```
        class_a              class_b
    ┌─────────────┐     ┌─────────────┐
    │  物理  化学  │     │  历史  地理  │
    │       ┌─────┼─────┼─────┐       │
    │       │数学 │     │编程 │       │
    │       │英语 │     │     │       │
    │       └─────┼─────┼─────┘       │
    └─────────────┘     └─────────────┘
    
    交集 (&)   = {数学, 英语, 编程}     （重叠部分）
    并集 (|)   = 所有区域
    A - B      = {物理, 化学}           （A独有）
    B - A      = {历史, 地理}           （B独有）
    对称差 (^) = {物理, 化学, 历史, 地理}（不重叠部分）
```

**集合的基本操作**：

```python
s = {1, 2, 3}

# 添加元素
s.add(4)
print(s)  # {1, 2, 3, 4}

# 添加重复元素——不会报错，但也不会添加
s.add(3)
print(s)  # {1, 2, 3, 4} （没有变化）

# 删除元素
s.remove(4)     # 删除指定元素，不存在会报 KeyError
s.discard(10)   # 删除指定元素，不存在也不报错（更安全）

# 判断元素是否在集合中（速度极快！）
print(3 in s)   # True
print(10 in s)  # False

# 判断子集和超集
a = {1, 2}
b = {1, 2, 3, 4}
print(a.issubset(b))    # True  a是b的子集
print(b.issuperset(a))  # True  b是a的超集
```

---

## 六、综合应用

现在让我们把前面学的所有知识串联起来，做几个实际的项目练习。

### 6.1 文本清洗

在真实项目中，拿到的原始数据往往很"脏"，需要清洗后才能使用。

```python
import re

def clean_text(raw_text):
    """
    文本清洗函数：
    1. 去除首尾空白
    2. 去除HTML标签
    3. 去除多余空格
    4. 去除特殊符号
    5. 统一小写
    """
    text = raw_text
    
    # 第1步：去除首尾空白
    text = text.strip()
    print(f"去除首尾空白：{text!r}")
    
    # 第2步：去除HTML标签
    text = re.sub(r"<[^>]+>", "", text)
    print(f"去除HTML标签：{text!r}")
    
    # 第3步：将多个空白字符替换为单个空格
    text = re.sub(r"\s+", " ", text)
    print(f"合并空白字符：{text!r}")
    
    # 第4步：去除特殊符号（只保留中文、英文、数字、空格和基本标点）
    text = re.sub(r"[^\w\s\u4e00-\u9fff，。！？、；：""''（）]", "", text)
    print(f"去除特殊符号：{text!r}")
    
    # 第5步：去除首尾可能多余的空格
    text = text.strip()
    
    return text


# 测试
raw = """
   <p>  Hello,   World!!!  </p>
   <br/>这是一个@#$测试文本~~~  
   <b>包含各种   乱七八糟的   格式</b>   
"""

result = clean_text(raw)
print(f"\n最终结果：{result}")
```

输出：

```
去除首尾空白：'<p>  Hello,   World!!!  </p>\n   <br/>这是一个@#$测试文本~~~  \n   <b>包含各种   乱七八糟的   格式</b>'
去除HTML标签：'  Hello,   World!!!  \n   这是一个@#$测试文本~~~  \n   包含各种   乱七八糟的   格式'
合并空白字符：' Hello, World!!! 这是一个@#$测试文本~~~ 包含各种 乱七八糟的 格式'
去除特殊符号：' Hello World 这是一个测试文本 包含各种 乱七八糟的 格式'

最终结果：Hello World 这是一个测试文本 包含各种 乱七八糟的 格式
```

**另一个实用场景——清洗用户输入数据**：

```python
import re

def clean_phone_numbers(raw_data):
    """
    从杂乱的文本中提取并标准化手机号
    """
    # 先去除所有的空格、横线、括号
    cleaned = re.sub(r"[\s\-\(\)（）]", "", raw_data)
    
    # 提取11位手机号
    phones = re.findall(r"1[3-9]\d{9}", cleaned)
    
    # 去重并保持顺序
    seen = set()
    unique_phones = []
    for p in phones:
        if p not in seen:
            seen.add(p)
            unique_phones.append(p)
    
    return unique_phones


raw_data = """
联系人列表：
张三: 138-0013-8000
李四: (139) 0013 9000
王五: 138 0013 8000   (和张三相同)
赵六: 15012345678
孙七: 010-88886666 (座机不要)
"""

phones = clean_phone_numbers(raw_data)
for i, phone in enumerate(phones, 1):
    print(f"手机号{i}: {phone}")
# 手机号1: 13800138000
# 手机号2: 13900139000
# 手机号3: 15012345678
```

---

### 6.2 正则提取信息

#### 案例 1：从简历文本中提取关键信息

```python
import re

resume = """
个人简历
姓名：张三
性别：男
年龄：28
手机：13800138000
邮箱：zhangsan@example.com
学历：本科
工作经验：5年
期望薪资：15000-20000元/月
技能：Python, Java, SQL, Docker
"""

# 提取所有 "key：value" 格式的信息
pairs = re.findall(r"(\S+)：(\S+)", resume)
info = dict(pairs)
print("=== 提取到的信息 ===")
for key, value in info.items():
    print(f"  {key}: {value}")

# 提取薪资范围
salary_match = re.search(r"(\d+)-(\d+)元", resume)
if salary_match:
    low = int(salary_match.group(1))
    high = int(salary_match.group(2))
    avg = (low + high) / 2
    print(f"\n期望薪资范围：{low}-{high}元")
    print(f"平均期望薪资：{avg:.0f}元")

# 提取技能列表
skill_match = re.search(r"技能：(.+)", resume)
if skill_match:
    skills = [s.strip() for s in skill_match.group(1).split(",")]
    print(f"\n技能列表：{skills}")
    print(f"技能数量：{len(skills)}项")
```

输出：

```
=== 提取到的信息 ===
  姓名: 张三
  性别: 男
  年龄: 28
  手机: 13800138000
  邮箱: zhangsan@example.com
  学历: 本科
  工作经验: 5年
  期望薪资: 15000-20000元/月
  技能: Python,

期望薪资范围：15000-20000元
平均期望薪资：17500元

技能列表：['Python', 'Java', 'SQL', 'Docker']
技能数量：4项
```

#### 案例 2：解析日志文件

```python
import re

log_data = """
2026-04-09 10:23:45 [INFO] User login: user_id=1001, ip=192.168.1.100
2026-04-09 10:24:12 [ERROR] Database connection failed: timeout after 30s
2026-04-09 10:25:01 [INFO] User login: user_id=1002, ip=192.168.1.101
2026-04-09 10:26:33 [WARNING] High memory usage: 85%
2026-04-09 10:27:15 [ERROR] File not found: /data/report.csv
2026-04-09 10:28:00 [INFO] User login: user_id=1001, ip=192.168.1.100
"""

# 提取所有错误信息
errors = re.findall(r"\[ERROR\]\s*(.+)", log_data)
print("=== 错误日志 ===")
for err in errors:
    print(f"  ❌ {err}")

# 提取所有登录的用户ID和IP
logins = re.findall(r"User login: user_id=(\d+), ip=([\d.]+)", log_data)
print("\n=== 登录记录 ===")
for user_id, ip in logins:
    print(f"  用户{user_id} 从 {ip} 登录")

# 统计各级别日志数量
levels = re.findall(r"\[(INFO|ERROR|WARNING)\]", log_data)
print("\n=== 日志级别统计 ===")
for level in set(levels):
    count = levels.count(level)
    print(f"  {level}: {count}条")

# 提取时间戳
timestamps = re.findall(r"\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}", log_data)
print(f"\n日志时间范围：{timestamps[0]} 到 {timestamps[-1]}")
```

输出：

```
=== 错误日志 ===
  ❌ Database connection failed: timeout after 30s
  ❌ File not found: /data/report.csv

=== 登录记录 ===
  用户1001 从 192.168.1.100 登录
  用户1002 从 192.168.1.101 登录
  用户1001 从 192.168.1.100 登录

=== 日志级别统计 ===
  WARNING: 1条
  INFO: 3条
  ERROR: 2条

日志时间范围：2026-04-09 10:23:45 到 2026-04-09 10:28:00
```

---

### 6.3 简单词频统计

这是自然语言处理（NLP）领域最基础的任务之一。

#### 英文词频统计

```python
import re

text = """
Python is a great programming language. Python is easy to learn.
Many people love Python because Python is powerful and Python is fun.
Programming with Python is enjoyable. I love programming in Python.
"""

# 第1步：文本预处理
text_clean = text.lower()                         # 统一小写
words = re.findall(r"[a-z]+", text_clean)         # 提取所有单词
print(f"总词数：{len(words)}")
print(f"不重复词数：{len(set(words))}")

# 第2步：手动统计词频
word_count = {}
for word in words:
    if word in word_count:
        word_count[word] += 1
    else:
        word_count[word] = 1

# 第3步：按频率排序
sorted_words = sorted(word_count.items(), key=lambda x: x[1], reverse=True)

# 第4步：输出结果
print("\n=== 词频统计 Top 10 ===")
print(f"{'单词':<15} {'次数':>4} {'占比':>8}")
print("-" * 30)
for word, count in sorted_words[:10]:
    percentage = count / len(words) * 100
    bar = "█" * count
    print(f"{word:<15} {count:>4} {percentage:>7.1f}% {bar}")
```

输出：

```
总词数：30
不重复词数：14

=== 词频统计 Top 10 ===
单词              次数       占比
------------------------------
python             6   20.0% ██████
is                 5   16.7% █████
programming        3   10.0% ███
love               2    6.7% ██
a                  1    3.3% █
great              1    3.3% █
language           1    3.3% █
easy               1    3.3% █
to                 1    3.3% █
learn              1    3.3% █
```

#### 中文词频统计（简易版）

中文没有空格分隔单词，所以需要"分词"。这里我们先用简单的单字统计和手动分词：

```python
import re

text = """
Python是一门优秀的编程语言。Python简单易学，Python功能强大。
许多人喜欢Python，因为Python能做很多事情。
学习Python是一个很好的选择。编程让生活更美好。
"""

# ========== 方法1：单字频率统计 ==========
# 提取所有中文字符
chinese_chars = re.findall(r"[\u4e00-\u9fff]", text)
print(f"中文字符总数：{len(chinese_chars)}")
print(f"不重复字符数：{len(set(chinese_chars))}")

# 统计字频
char_count = {}
for char in chinese_chars:
    char_count[char] = char_count.get(char, 0) + 1
    # .get(key, 0) 等价于：如果key存在返回其值，否则返回0

sorted_chars = sorted(char_count.items(), key=lambda x: x[1], reverse=True)
print("\n=== 高频汉字 Top 10 ===")
for char, count in sorted_chars[:10]:
    print(f"  '{char}' 出现了 {count} 次")


# ========== 方法2：简单的手动中文分词 ==========
# 定义一个简单的词典
dictionary = ["Python", "编程", "语言", "简单", "易学", "功能", "强大",
              "喜欢", "学习", "选择", "生活", "美好", "很多", "事情", "优秀"]

# 统计每个词出现的次数
print("\n=== 基于词典的词频统计 ===")
word_freq = {}
for word in dictionary:
    count = text.count(word)
    if count > 0:
        word_freq[word] = count

sorted_freq = sorted(word_freq.items(), key=lambda x: x[1], reverse=True)
for word, count in sorted_freq:
    print(f"  {word}: {count}次")
```

> 💡 **进阶提示**：真正的中文分词通常使用 `jieba` 库：
>
> ```python
> # pip install jieba
> import jieba
> words = jieba.lcut("Python是一门优秀的编程语言")
> print(words)  # ['Python', '是', '一门', '优秀', '的', '编程', '语言']
> ```

#### 使用 Counter 简化统计

Python 标准库中有个 `Counter` 类，可以大大简化词频统计：

```python
from collections import Counter
import re

text = """
Python is a great programming language. Python is easy to learn.
Many people love Python because Python is powerful and Python is fun.
"""

words = re.findall(r"[a-z]+", text.lower())

# 一行搞定词频统计！
counter = Counter(words)

# 获取最常见的5个词
print("最常见的5个词：")
for word, count in counter.most_common(5):
    print(f"  {word}: {count}次")

# Counter 的其他有用功能
print(f"\n'python'出现了 {counter['python']} 次")
print(f"总词数：{sum(counter.values())}")

# 两段文本的词频可以直接相加
text2 = "python is the best language for data science"
counter2 = Counter(text2.lower().split())
total = counter + counter2
print(f"\n合并后'python'出现了 {total['python']} 次")
```

---

### 6.4 综合项目：文本分析器

最后，让我们把所有知识融合到一个完整的项目中：

```python
import re
from collections import Counter

def analyze_text(text, title="文本分析报告"):
    """
    综合文本分析工具
    功能：
    1. 基本统计（字符数、行数、句子数等）
    2. 词频统计
    3. 信息提取（邮箱、网址、数字等）
    4. 文本清洗
    """
    
    print("=" * 50)
    print(f"  {title}")
    print("=" * 50)
    
    # ============================
    # 1. 基本统计
    # ============================
    print("\n【一、基本统计】")
    
    total_chars = len(text)
    chars_no_space = len(text.replace(" ", "").replace("\n", ""))
    lines = text.strip().split("\n")
    non_empty_lines = [line for line in lines if line.strip()]
    
    # 用正则拆分句子
    sentences = re.split(r"[.!?。！？]+", text)
    sentences = [s.strip() for s in sentences if s.strip()]
    
    # 提取英文单词
    english_words = re.findall(r"[a-zA-Z]+", text)
    
    # 提取中文字符
    chinese_chars = re.findall(r"[\u4e00-\u9fff]", text)
    
    # 提取数字
    numbers = re.findall(r"\d+\.?\d*", text)
    
    print(f"  总字符数（含空格）：{total_chars}")
    print(f"  总字符数（不含空格）：{chars_no_space}")
    print(f"  总行数：{len(lines)}")
    print(f"  非空行数：{len(non_empty_lines)}")
    print(f"  句子数：{len(sentences)}")
    print(f"  英文单词数：{len(english_words)}")
    print(f"  中文字符数：{len(chinese_chars)}")
    print(f"  数字个数：{len(numbers)}")
    
    # ============================
    # 2. 词频统计
    # ============================
    print("\n【二、英文词频统计 Top 10】")
    
    # 清洗并统计
    words_lower = [w.lower() for w in english_words]
    
    # 停用词（常见但无意义的词）
    stop_words = {"the", "a", "an", "is", "are", "was", "were", "in", "on",
                  "at", "to", "for", "of", "and", "or", "but", "it", "this",
                  "that", "with", "as", "by", "from", "be", "has", "have",
                  "had", "do", "does", "did", "not", "no", "i", "we", "you",
                  "he", "she", "they", "my", "your", "his", "her", "its"}
    
    # 去掉停用词
    meaningful_words = [w for w in words_lower if w not in stop_words and len(w) > 1]
    word_counter = Counter(meaningful_words)
    
    if word_counter:
        max_count = word_counter.most_common(1)[0][1]
        print(f"  {'单词':<15} {'次数':>4}  可视化")
        print(f"  {'-'*40}")
        for word, count in word_counter.most_common(10):
            bar_length = int(count / max_count * 20)
            bar = "█" * bar_length
            print(f"  {word:<15} {count:>4}  {bar}")
    else:
        print("  （未找到有意义的英文单词）")
    
    # ============================
    # 3. 信息提取
    # ============================
    print("\n【三、信息提取】")
    
    # 邮箱
    emails = re.findall(r"[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+", text)
    if emails:
        print(f"  📧 邮箱（{len(emails)}个）：")
        for e in set(emails):
            print(f"     {e}")
    
    # 网址
    urls = re.findall(r"https?://[^\s<>\"']+", text)
    if urls:
        print(f"  🔗 网址（{len(urls)}个）：")
        for u in set(urls):
            print(f"     {u}")
    
    # 手机号
    phones = re.findall(r"1[3-9]\d{9}", text)
    if phones:
        print(f"  📱 手机号（{len(phones)}个）：")
        for p in set(phones):
            masked = p[:3] + "****" + p[7:]
            print(f"     {masked}")
    
    # 日期
    dates = re.findall(r"\d{4}[-/年]\d{1,2}[-/月]\d{1,2}[日]?", text)
    if dates:
        print(f"  📅 日期（{len(dates)}个）：")
        for d in dates:
            print(f"     {d}")
    
    if not any([emails, urls, phones, dates]):
        print("  （未提取到特殊信息）")
    
    # ============================
    # 4. 独特词汇
    # ============================
    print("\n【四、词汇丰富度】")
    if words_lower:
        unique_ratio = len(set(words_lower)) / len(words_lower) * 100
        print(f"  总词数：{len(words_lower)}")
        print(f"  不重复词数：{len(set(words_lower))}")
        print(f"  词汇丰富度：{unique_ratio:.1f}%")
        
        # 只出现一次的词
        hapax = [word for word, count in word_counter.items() if count == 1]
        if hapax:
            print(f"  仅出现一次的词（前10个）：{', '.join(hapax[:10])}")
    
    print("\n" + "=" * 50)
    print("  分析完成！")
    print("=" * 50)


# ============ 测试 ============

sample_text = """
Python Programming News - April 2026

Python continues to dominate the programming world in 2026. 
According to the latest survey, Python is used by over 15 million 
developers worldwide. Python is especially popular in data science, 
machine learning, and web development.

The Python Software Foundation announced Python 3.14 on 2026-03-15, 
which includes significant performance improvements. Many developers 
are excited about the new features.

For more information, visit https://www.python.org or contact 
the community at info@python.org. You can also reach the Chinese 
community coordinator at zhangsan@python.org.cn or call 13800138000.

Python is not just a programming language, it's a community. 
Whether you're building web applications, analyzing data, or 
creating machine learning models, Python has the tools you need.
Python makes programming fun and accessible for everyone.
"""

analyze_text(sample_text, "Python 新闻文本分析")
```

输出：

```
==================================================
  Python 新闻文本分析
==================================================

【一、基本统计】
  总字符数（含空格）：815
  总字符数（不含空格）：678
  总行数：18
  非空行数：14
  句子数：11
  英文单词数：120
  中文字符数：0
  数字个数：6

【二、英文词频统计 Top 10】
  单词              次数  可视化
  ----------------------------------------
  python             10  ████████████████████
  programming         4  ████████
  community           2  ████
  developers          2  ████
  learning            2  ████
  machine             2  ████
  data                2  ████
  web                 2  ████
  new                 2  ████
  language            2  ████

【三、信息提取】
  📧 邮箱（2个）：
     info@python.org
     zhangsan@python.org.cn
  🔗 网址（1个）：
     https://www.python.org
  📱 手机号（1个）：
     138****8000
  📅 日期（1个）：
     2026-03-15

【四、词汇丰富度】
  总词数：120
  不重复词数：79
  词汇丰富度：65.8%
  仅出现一次的词（前10个）：continues, dominate, world, according, latest

==================================================
  分析完成！
==================================================
```

---

## 课后练习

### 练习 1：基础题

```python
# 1. 给定字符串 s = "Hello, Python World!"
#    - 用切片取出 "Python"
#    - 将整个字符串反转
#    - 统计字母 'o' 出现了几次

# 2. 给定列表 fruits = ["  Apple ", "banana  ", "  Cherry"]
#    - 去除每个元素的首尾空格
#    - 全部转为小写
#    - 用 " | " 连接成一个字符串
```

### 练习 2：正则表达式

```python
# 1. 从以下文本中提取所有的价格（数字，可能有小数点）：
#    text = "苹果5.5元一斤，香蕉3元一斤，葡萄12.8元一斤"

# 2. 验证以下字符串是否是合法的邮箱地址：
#    emails = ["test@example.com", "invalid@", "user.name@domain.co.uk", "@no.com"]

# 3. 将以下文本中的手机号中间四位替换为****：
#    text = "张三13812345678，李四15987654321"
```

### 练习 3：综合应用

```python
# 给定一段英文文章（可以自己找一段），完成：
# 1. 统计总单词数和不重复单词数
# 2. 找出最长的5个单词
# 3. 找出出现次数最多的5个单词（去掉停用词）
# 4. 统计每个句子的平均单词数
```

---

## 参考答案（建议先自己做再看）

### 练习 1 参考答案

```python
# 题目1
s = "Hello, Python World!"
print(s[7:13])       # Python
print(s[::-1])       # !dlroW nohtyP ,olleH
print(s.count('o'))  # 3

# 题目2
fruits = ["  Apple ", "banana  ", "  Cherry"]
cleaned = [f.strip().lower() for f in fruits]
print(cleaned)                 # ['apple', 'banana', 'cherry']
print(" | ".join(cleaned))     # apple | banana | cherry
```

### 练习 2 参考答案

```python
import re

# 题目1
text = "苹果5.5元一斤，香蕉3元一斤，葡萄12.8元一斤"
prices = re.findall(r"\d+\.?\d*", text)
print(prices)  # ['5.5', '3', '12.8']

# 题目2
emails = ["test@example.com", "invalid@", "user.name@domain.co.uk", "@no.com"]
pattern = r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$"
for email in emails:
    is_valid = bool(re.match(pattern, email))
    print(f"  {email:<30} {'✅ 合法' if is_valid else '❌ 不合法'}")

# 题目3
text = "张三13812345678，李四15987654321"
result = re.sub(r"(1\d{2})\d{4}(\d{4})", r"\1****\2", text)
print(result)  # 张三138****5678，李四159****4321
```

---

## 总结思维导图

```
Python 字符串与正则表达式
│
├── 字符串基础
│   ├── 索引：s[0], s[-1]
│   ├── 切片：s[start:stop:step]
│   ├── 不可变性：不能修改，只能创建新的
│   ├── 常用方法
│   │   ├── split() / join()    ← 互逆操作，最常用！
│   │   ├── replace()           ← 替换
│   │   ├── strip()             ← 去首尾空白
│   │   ├── find() / count()    ← 查找与计数
│   │   └── startswith() / endswith()  ← 判断开头结尾
│   └── 格式化：f"...{变量}..."  ← 强烈推荐！
│
├── 字符串进阶
│   ├── 大小写：upper() / lower() / title()
│   ├── 判断：isdigit() / isalpha() / isalnum()
│   ├── 转义字符：\n \t \\ 以及 r"原始字符串"
│   └── 编码：ord() / chr() / encode() / decode()
│
├── 正则表达式 (import re)
│   ├── 元字符：. \d \w \s
│   ├── 量词：* + ? {n,m}
│   ├── 字符集：[abc] [a-z] [^0-9]
│   ├── 分组：() 捕获  (?:) 非捕获
│   ├── 锚点：^ 开头  $ 结尾
│   ├── 常用函数
│   │   ├── search()   ← 找第一个
│   │   ├── findall()  ← 找所有
│   │   ├── sub()      ← 替换
│   │   ├── match()    ← 从开头匹配
│   │   └── split()    ← 按模式分割
│   └── 贪婪 vs 非贪婪：.*  vs  .*?
│
├── 列表与字符串
│   ├── split() ↔ join() 互转
│   ├── 列表推导式处理文本
│   └── append() / sort() / sorted()
│
├── 集合 (set)
│   ├── 去重：list(set(data))
│   ├── 交集 &  并集 |  差集 -
│   └── 对称差集 ^
│
└── 综合应用
    ├── 文本清洗
    ├── 信息提取（邮箱、手机号、日期等）
    └── 词频统计（Counter）
```

---

> **学完这份教程后，你应该能够**：
> 1. 熟练操作 Python 字符串的索引、切片和各种方法
> 2. 使用 f-string 优雅地格式化输出
> 3. 编写正则表达式来匹配、搜索和替换文本
> 4. 使用列表推导式和集合来高效处理文本数据
> 5. 完成简单的文本清洗和词频统计任务
>
> **下一步学习建议**：文件读写（用今天学的技能处理真实文件）→ 字典进阶 → 函数与模块 → 面向对象编程