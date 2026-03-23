# 📘 HDU 2051–2057 Python 刷题教程

---

## 📌 目录

| #    | 题目            | 核心考点             |
| ---- | --------------- | -------------------- |
| 2051 | Bitset          | 整数转二进制         |
| 2052 | Picture         | 字符矩形绘制         |
| 2053 | Switch Game     | 因子个数与完全平方数 |
| 2054 | A == B ?        | 字符串精确比较       |
| 2055 | An Easy Problem | ASCII 码与大小写转换 |
| 2056 | Rectangles      | 矩形交集面积         |
| 2057 | A + B Again     | 十六进制加法         |

---

## 🟢 HDU 2051 — Bitset

### 📜 原题描述

**Problem Description**

给定一个正整数 N（1 ≤ N ≤ 1000），将其转换为二进制表示并输出（不含前导零）。

**Input**

输入包含多组测试数据，每组一行，包含一个正整数 N。

**Output**

对于每个 N，输出其二进制表示（不含前导零），每个输出占一行。

**Sample Input**
```
7
8
```

**Sample Output**
```
111
1000
```

---

### 💡 解题思路

将十进制整数转换为二进制字符串，核心就是不断地对 2 取余、除以 2，余数的逆序即为二进制。

Python 中有多种实现方式：
1. **手动模拟短除法**：循环取余拼接
2. **内置函数 `bin()`**：直接调用，去掉前缀 `0b`
3. **格式化字符串**：使用 `format()` 或 f-string

---

### ✅ 解法一：手动模拟短除法（暴力/基础）

```python
import sys

for line in sys.stdin:            # 逐行读取输入，直到 EOF
    n = int(line.strip())         # 去除行尾换行符，转为整数
    if n == 0:                    # 特殊情况：0 的二进制就是 "0"
        print(0)
        continue
    result = []                   # 用列表收集每一位的余数
    while n > 0:                  # 当 n 不为 0 时持续循环
        result.append(str(n % 2)) # 取 n 除以 2 的余数（0 或 1），转字符串存入列表
        n //= 2                   # n 整除 2，相当于右移一位
    result.reverse()              # 余数是从低位到高位收集的，需要反转
    print(''.join(result))        # 将列表拼接为字符串输出
```

**复杂度分析**
- **时间复杂度**：O(log N)，循环次数等于 N 的二进制位数
- **空间复杂度**：O(log N)，存储二进制各位

---

### ✅ 解法二：使用内置函数 `bin()`（推荐）

```python
import sys

for line in sys.stdin:            # 逐行读取，处理多组输入
    n = int(line.strip())         # 读取整数
    print(bin(n)[2:])             # bin(n) 返回 "0b1010" 这样的字符串
                                  # [2:] 切片去掉前缀 "0b"，只保留二进制数字部分
```

> **解释 `bin()` 函数**：
> - `bin(7)` → `'0b111'`
> - `bin(8)` → `'0b1000'`
> - `[2:]` 切片操作去掉前两个字符 `'0b'`

**复杂度分析**
- **时间复杂度**：O(log N)，`bin()` 内部也是 O(log N) 的转换
- **空间复杂度**：O(log N)

---

### ✅ 解法三：格式化字符串

```python
import sys

for line in sys.stdin:
    n = int(line.strip())
    print(f"{n:b}")               # f-string 格式化，":b" 表示以二进制格式输出
                                  # 等价于 format(n, 'b')
```

> `format(n, 'b')` 和 `f"{n:b}"` 效果完全一致，都会将整数格式化为不含前缀的二进制字符串。

**复杂度分析**
- 同解法二

---

### ⚠️ 易错提示

1. **多组输入**：题目会有多组测试数据，需要循环读取直到 EOF。在 Python 中常用 `for line in sys.stdin` 或 `while True: try ... except EOFError`。
2. **不要输出前缀 `0b`**：如果使用 `bin()`，记得切片 `[2:]`。
3. **N=0 的情况**：题目说 N ≥ 1，但养成好习惯，考虑一下 `bin(0)` → `'0b0'` → `'0'`，是正确的。

---

## 🟢 HDU 2052 — Picture

### 📜 原题描述

**Problem Description**

给定矩形的宽 W 和高 H，使用字符绘制一个矩形图案：
- 四个角用 `+` 表示
- 上下边框（水平边）用 `-` 表示
- 左右边框（垂直边）用 `|` 表示
- 内部填充空格

**Input**

输入包含多组测试数据，每组一行，包含两个正整数 W 和 H（1 ≤ W, H ≤ 80）。

**Output**

对于每组输入，输出对应的矩形图案。每两个矩形之间输出一个空行。

**Sample Input**
```
1 1
2 3
```

**Sample Output**
```
+-+
| |
+-+

+--+
|  |
|  |
|  |
+--+
```

---

### 💡 解题思路

按行输出矩形：
- **第 1 行和最后一行**（边框行）：`+` + W 个 `-` + `+`
- **中间的 H 行**（内容行）：`|` + W 个空格 + `|`

关键注意点：**两个矩形之间要输出一个空行**，但最后一个矩形后面不需要空行。

---

### ✅ 解法一：逐行构造输出（基础）

```python
import sys

first = True                              # 标记是否为第一个矩形，用于控制空行
for line in sys.stdin:
    line = line.strip()
    if not line:                          # 跳过空行
        continue
    w, h = map(int, line.split())         # 读取宽 W 和高 H
    
    if not first:                         # 如果不是第一个矩形，先输出一个空行分隔
        print()
    first = False                         # 第一个矩形输出后，后续都不是第一个了
    
    # 第一行：边框行
    print('+' + '-' * w + '+')            # "+" 开头，w 个 "-"，"+" 结尾
    
    # 中间 h 行：内容行
    for i in range(h):                    # 循环 h 次
        print('|' + ' ' * w + '|')        # "|" 开头，w 个空格，"|" 结尾
    
    # 最后一行：边框行（和第一行一样）
    print('+' + '-' * w + '+')
```

**复杂度分析**
- **时间复杂度**：O(W × H)，需要输出 (H+2) 行，每行长度为 (W+2)
- **空间复杂度**：O(W)，每行最长 W+2

---

### ✅ 解法二：使用列表构造后一次性输出

```python
import sys

first = True
for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    w, h = map(int, line.split())
    
    if not first:
        print()
    first = False
    
    rows = []                             # 用列表收集所有行
    border = '+' + '-' * w + '+'          # 预先构造边框行字符串
    middle = '|' + ' ' * w + '|'          # 预先构造内容行字符串
    
    rows.append(border)                   # 添加顶部边框
    rows.extend([middle] * h)             # 添加 h 行内容行
    rows.append(border)                   # 添加底部边框
    
    print('\n'.join(rows))                # 用换行符拼接所有行，一次性输出
```

**复杂度分析**
- 同解法一，但减少了 I/O 调用次数（一次 `print` 代替多次）

---

### ⚠️ 易错提示

1. **空行分隔**：两个矩形之间需要一个空行，但**最后一个矩形后面不要多输出空行**，否则可能 PE（Presentation Error）。
2. **W 和 H 的含义**：W 是内部宽度（不含边框），H 是内部高度（不含边框），总输出是 (H+2) 行 × (W+2) 列。
3. **行尾不要有多余空格**：某些 OJ 对行尾空格很敏感，但本题使用字符串拼接一般不会出问题。

---

## 🟢 HDU 2053 — Switch Game

### 📜 原题描述

**Problem Description**

有 N 个灯，编号 1 到 N，初始状态全部关闭（OFF）。

有 N 个人依次操作：
- 第 i 个人会翻转所有**编号是 i 的倍数**的灯（开变关，关变开）。

问：经过 N 轮操作后，第 N 号灯的状态是什么？

**Input**

输入包含多组测试数据，每组一行，包含一个正整数 N（1 ≤ N ≤ 2^31 - 1，即 int 范围）。

**Output**

对于每个 N，如果第 N 号灯最终是开的，输出 `1`；否则输出 `0`。

**Sample Input**
```
1
2
3
```

**Sample Output**
```
1
0
0
```

---

### 💡 解题思路

**关键数学分析**：

第 N 号灯被翻转的次数 = N 的因子个数。

- 第 i 个人翻转编号是 i 的倍数的灯。灯 N 被翻转，当且仅当 i 是 N 的因子。
- 如果因子个数是**奇数**，灯最终是**开的**（翻转奇数次 = 开）。
- 如果因子个数是**偶数**，灯最终是**关的**（翻转偶数次 = 关）。

**什么时候因子个数是奇数？**

一般因子成对出现：如果 d 是 N 的因子，那么 N/d 也是，它们是一对。
唯一例外：当 d = N/d，即 d² = N 时，这个因子只算一次。

所以，**当且仅当 N 是完全平方数时，N 的因子个数为奇数**。

结论：**判断 N 是否为完全平方数**。

---

### ✅ 解法一：暴力模拟（仅用于理解，会超时）

```python
import sys

for line in sys.stdin:
    n = int(line.strip())
    
    # 统计 n 的因子个数
    count = 0                         # 因子计数器
    for i in range(1, n + 1):         # 遍历 1 到 n
        if n % i == 0:                # 如果 i 能整除 n，则 i 是 n 的因子
            count += 1
    
    # 因子个数为奇数则灯亮（1），否则灯灭（0）
    print(1 if count % 2 == 1 else 0)
```

**复杂度分析**
- **时间复杂度**：O(N)，对每个测试用例遍历 1 到 N
- 当 N 接近 2^31 时，这个解法会严重超时！

---

### ✅ 解法二：优化统计因子（O(√N)）

```python
import sys

for line in sys.stdin:
    n = int(line.strip())
    
    count = 0
    i = 1
    while i * i <= n:                 # 只遍历到 √N
        if n % i == 0:                # i 是 n 的因子
            count += 1                # 计入因子 i
            if i != n // i:           # 如果 i 和 n/i 不相等，还有配对因子
                count += 1            # 计入配对因子 n/i
        i += 1
    
    print(1 if count % 2 == 1 else 0)
```

**复杂度分析**
- **时间复杂度**：O(√N)

---

### ✅ 解法三：直接判断完全平方数（最优，推荐）

```python
import sys
import math

for line in sys.stdin:
    n = int(line.strip())
    
    s = int(math.isqrt(n))            # 计算 n 的整数平方根（Python 3.8+）
                                      # math.isqrt(n) 返回 ⌊√n⌋
    if s * s == n:                    # 如果平方根的平方等于 n，说明 n 是完全平方数
        print(1)                      # 灯是开的
    else:
        print(0)                      # 灯是关的
```

> **为什么用 `math.isqrt()` 而不是 `int(math.sqrt())`？**
>
> `math.sqrt()` 返回浮点数，对于大整数可能有精度问题。例如 `math.sqrt(10**18)` 可能不精确。
> `math.isqrt()` 是纯整数运算，不会有精度损失，推荐使用。

**复杂度分析**
- **时间复杂度**：严格来说可记为 O(log N)（按整数位数计）；在本题 `int` 范围内，也可以近似视为常数级
- **空间复杂度**：O(1)

---

### ⚠️ 易错提示

1. **不要真的去模拟所有 N 个人翻转灯**：N 最大约 2×10⁹，直接模拟铁定超时。
2. **浮点精度**：如果用 `int(n ** 0.5)`，对于大数可能得到错误结果。务必用 `math.isqrt()`。
3. **理解因子与完全平方数的关系**：这是经典数学结论，值得记忆。

---

## 🟡 HDU 2054 — A == B ?

### 📜 原题描述

**Problem Description**

给定两个数 A 和 B，判断它们是否相等。

**注意**：A 和 B 可能是整数或浮点数，可能有前导零、尾部零，数值可能非常大（超出标准整数/浮点数范围）。

**Input**

输入包含多组测试数据，每组一行，包含两个由空格分隔的数 A 和 B。

**Output**

对于每组输入，如果 A 和 B 数值相等，输出 `YES`；否则输出 `NO`。

**Sample Input**
```
1 1
2 3
0 0
1.0 1
```

**Sample Output**
```
YES
NO
YES
YES
```

---

### 💡 解题思路

这道题看似简单，实则暗藏陷阱！

不能简单地用 `float()` 比较，因为：
- 数可能非常长，超出浮点精度
- `1.0` 和 `1` 应该相等
- `00123` 和 `123` 应该相等
- `1.10` 和 `1.1` 应该相等

解题关键：**对字符串进行标准化处理**，把不同写法但数值相同的表示统一成同一种“规范形式”，再比较字符串。

标准化规则：
1. 先处理正负号，便于后续统一操作
2. 整数部分去掉前导零（但至少保留一个 `0`）
3. 小数部分去掉尾部无意义的零
4. 如果小数部分删完为空，就连同小数点一起去掉（如 `1.000` → `1`）
5. `-0`、`-0.0` 等都应规范成 `0`

> 下面这份“手动标准化”代码默认输入是**普通十进制写法**（不含科学计数法）。如果题目扩展到 `1e3` 这类表示，优先使用 `Decimal`。

---

### ✅ 解法一：手动标准化字符串（稳妥）

```python
import sys

def normalize(s):
    """
    将普通十进制字符串标准化。
    例：'0012.3400' -> '12.34'，'-0.000' -> '0'
    """
    sign = ''
    if s.startswith('-'):
        sign = '-'
        s = s[1:]
    elif s.startswith('+'):
        s = s[1:]

    if '.' in s:
        int_part, frac_part = s.split('.', 1)
    else:
        int_part, frac_part = s, ''

    int_part = int_part.lstrip('0') or '0'
    frac_part = frac_part.rstrip('0')

    if frac_part:
        s = int_part + '.' + frac_part
    else:
        s = int_part

    if s == '0':
        sign = ''

    return sign + s

for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    parts = line.split()
    if len(parts) < 2:
        continue
    a, b = parts[0], parts[1]
    
    if normalize(a) == normalize(b):  # 标准化后比较
        print("YES")
    else:
        print("NO")
```

**复杂度分析**
- **时间复杂度**：O(L)，L 为字符串长度，`lstrip`、`rstrip` 和字符串比较都是线性的
- **空间复杂度**：O(L)

---

### ✅ 解法二：利用 Python 的 `Decimal` 模块（简洁但需注意）

```python
import sys
from decimal import Decimal, InvalidOperation

for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    parts = line.split()
    if len(parts) < 2:
        continue
    a_str, b_str = parts[0], parts[1]
    
    try:
        a = Decimal(a_str)            # Decimal 可以精确表示任意精度的十进制数
        b = Decimal(b_str)            # 不会有浮点精度损失
        if a == b:                    # Decimal 的比较是精确的
            print("YES")
        else:
            print("NO")
    except InvalidOperation:          # 如果输入不合法
        print("NO")
```

> **为什么用 `Decimal` 而不用 `float`？**
>
> `float` 是 64 位浮点数，精度约 15-17 位有效数字。如果输入的数有几百位，`float` 会丢失精度导致误判。
> `Decimal` 使用十进制高精度表示，更适合这类“按十进制语义比较”的题目。

> **补充提醒**：
>
> `Decimal` 解法写起来更短，但教程中保留“手动标准化”版本仍然很有价值，因为它能帮助读者真正理解“为什么 `1.0` 和 `1` 相等”。

**复杂度分析**
- **时间复杂度**：O(L)
- **空间复杂度**：O(L)

---

### ⚠️ 易错提示

1. **绝对不能用 `float` 转换后比较**：这是本题最大的坑！`float` 精度有限，大数或高精度小数会出错。
2. **前导零**：`00123` 和 `123` 应判为相等。
3. **尾部零**：`1.10` 和 `1.1`、`1.0` 和 `1` 应判为相等。
4. **全是零的情况**：`0`、`00`、`0.0`、`0.00` 都应该相等。
5. **负零**：部分 OJ 可能测试 `-0` 和 `0` 的情况（虽然较少见）。

---

## 🟢 HDU 2055 — An Easy Problem

### 📜 原题描述

**Problem Description**

定义函数 f(x)：
- 如果 x 是大写字母，f(x) = x 在大写字母中的序号（A=1, B=2, ..., Z=26）
- 如果 x 是小写字母，f(x) = x 在小写字母中的序号的负值（a=-1, b=-2, ..., z=-26）

给定一个字符 x 和一个整数 y，计算 f(x) + y 的值。

**Input**

第一行一个整数 T，表示测试用例数。

接下来 T 行，每行包含一个字符 x 和一个整数 y，以空格分隔。

**Output**

对于每组输入，输出 f(x) + y。

**Sample Input**
```
2
A 1
a 2
```

**Sample Output**
```
2
1
```

> 解释：
> - f('A') = 1, 1 + 1 = 2
> - f('a') = -1, -1 + 2 = 1

---

### 💡 解题思路

核心就是字符与 ASCII 码的转换：
- 大写字母 `A` 的 ASCII 码是 65，`B` 是 66，...
- 小写字母 `a` 的 ASCII 码是 97，`b` 是 98，...

公式：
- 大写：`f(x) = ord(x) - ord('A') + 1`
- 小写：`f(x) = -(ord(x) - ord('a') + 1)`

---

### ✅ 解法一：使用 `ord()` 和条件判断

```python
T = int(input())                      # 读取测试用例数
for _ in range(T):                    # 循环 T 次
    parts = input().split()           # 读取一行并按空格分割
    x = parts[0]                      # 第一个部分是字符 x
    y = int(parts[1])                 # 第二个部分是整数 y
    
    if x.isupper():                   # 判断 x 是否为大写字母
        fx = ord(x) - ord('A') + 1    # 大写字母：A→1, B→2, ..., Z→26
    else:                             # x 是小写字母
        fx = -(ord(x) - ord('a') + 1) # 小写字母：a→-1, b→-2, ..., z→-26
    
    print(fx + y)                     # 输出 f(x) + y
```

**复杂度分析**
- **时间复杂度**：O(T)，每个用例 O(1)
- **空间复杂度**：O(1)

---

### ✅ 解法二：统一公式计算

```python
T = int(input())
for _ in range(T):
    parts = input().split()
    x = parts[0]
    y = int(parts[1])
    
    if 'A' <= x <= 'Z':                           # 大写字母范围判断
        fx = ord(x) - 64                           # ord('A') = 65，65 - 64 = 1
    else:                                          # 小写字母
        fx = -(ord(x) - 96)                        # ord('a') = 97，-(97 - 96) = -1
    
    print(fx + y)
```

> **小技巧**：`ord('A')` = 65 = 64 + 1，所以 `ord(x) - 64` 等价于 `ord(x) - ord('A') + 1`，写起来更简洁。

---

### ✅ 解法三：利用字典映射

```python
import string

# 预构建映射字典
upper_map = {c: i + 1 for i, c in enumerate(string.ascii_uppercase)}
# {'A': 1, 'B': 2, ..., 'Z': 26}
lower_map = {c: -(i + 1) for i, c in enumerate(string.ascii_lowercase)}
# {'a': -1, 'b': -2, ..., 'z': -26}
char_map = {**upper_map, **lower_map}  # 合并两个字典

T = int(input())
for _ in range(T):
    parts = input().split()
    x = parts[0]
    y = int(parts[1])
    
    print(char_map[x] + y)            # 直接查字典得到 f(x)，加上 y
```

**复杂度分析**
- 字典查找 O(1)，整体 O(T)

---

### ⚠️ 易错提示

1. **输入格式**：字符和数字之间用空格分隔。`input().split()` 可以正确处理。
2. **大小写判断**：不要搞反！大写是正数，小写是负数。
3. **y 可能是负数**：`int(parts[1])` 可以正确处理负数。
4. **不要忘记加 1**：`ord('A') - ord('A')` 等于 0，但 A 对应的是 1，不是 0。

---

## 🟡 HDU 2056 — Rectangles

### 📜 原题描述

**Problem Description**

在一个二维平面上，有两个矩形。每个矩形由其**左下角**和**右上角**的坐标给出。

求两个矩形交集区域的面积。如果没有交集，面积为 0.00。

**Input**

输入包含多组测试数据。每组一行，包含 8 个浮点数：
`x1 y1 x2 y2 x3 y3 x4 y4`

其中 `(x1, y1)` 和 `(x2, y2)` 是第一个矩形的两个对角顶点，`(x3, y3)` 和 `(x4, y4)` 是第二个矩形的两个对角顶点。

**注意**：题目给出的不一定是"左下角"和"右上角"，可能是任意对角！需要自行确定左下角和右上角。

**Output**

对于每组输入，输出交集面积，保留 2 位小数。

**Sample Input**
```
1.00 1.00 3.00 3.00 2.00 2.00 4.00 4.00
5.00 5.00 13.00 13.00 4.00 4.00 12.50 12.50
```

**Sample Output**
```
1.00
56.25
```

---

### 💡 解题思路

**矩形交集的经典算法**：

两个轴对齐矩形的交集仍然是一个矩形（或为空）。交集矩形的范围为：
- x 方向：`[max(左边界1, 左边界2), min(右边界1, 右边界2)]`
- y 方向：`[max(下边界1, 下边界2), min(上边界1, 上边界2)]`

如果交集的宽或高 ≤ 0，则没有交集，面积为 0。

**预处理**：由于输入的两个点可能是任意对角，需要先确定每个矩形的左下角和右上角。

---

### ✅ 解法一：标准解法

```python
import sys

for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    nums = list(map(float, line.split()))  # 读取 8 个浮点数
    x1, y1, x2, y2 = nums[0], nums[1], nums[2], nums[3]  # 矩形 1 的两个对角顶点
    x3, y3, x4, y4 = nums[4], nums[5], nums[6], nums[7]  # 矩形 2 的两个对角顶点
    
    # 确定矩形 1 的左下角和右上角
    # 左下角：(min(x1,x2), min(y1,y2))
    # 右上角：(max(x1,x2), max(y1,y2))
    left1  = min(x1, x2)                  # 矩形 1 左边界
    right1 = max(x1, x2)                  # 矩形 1 右边界
    down1  = min(y1, y2)                  # 矩形 1 下边界
    up1    = max(y1, y2)                  # 矩形 1 上边界
    
    # 确定矩形 2 的左下角和右上角
    left2  = min(x3, x4)
    right2 = max(x3, x4)
    down2  = min(y3, y4)
    up2    = max(y3, y4)
    
    # 计算交集矩形的范围
    inter_left  = max(left1, left2)        # 交集左边界 = 两个左边界的较大值
    inter_right = min(right1, right2)      # 交集右边界 = 两个右边界的较小值
    inter_down  = max(down1, down2)        # 交集下边界 = 两个下边界的较大值
    inter_up    = min(up1, up2)            # 交集上边界 = 两个上边界的较小值
    
    # 计算交集的宽和高
    width  = inter_right - inter_left      # 交集宽度
    height = inter_up - inter_down         # 交集高度
    
    # 如果宽或高 <= 0，说明没有交集
    if width > 0 and height > 0:
        area = width * height
    else:
        area = 0.0
    
    print(f"{area:.2f}")                   # 保留 2 位小数输出
```

**复杂度分析**
- **时间复杂度**：O(1)，每组数据只做常数次运算
- **空间复杂度**：O(1)

---

### ✅ 解法二：封装为函数，更清晰

```python
import sys

def rect_intersection_area(x1, y1, x2, y2, x3, y3, x4, y4):
    """
    计算两个矩形交集面积。
    (x1,y1)-(x2,y2) 为矩形1的对角，(x3,y3)-(x4,y4) 为矩形2的对角。
    """
    # 标准化：确定每个矩形的 [left, right] x [down, up]
    r1_left,  r1_right = min(x1, x2), max(x1, x2)
    r1_down,  r1_up    = min(y1, y2), max(y1, y2)
    r2_left,  r2_right = min(x3, x4), max(x3, x4)
    r2_down,  r2_up    = min(y3, y4), max(y3, y4)
    
    # 交集范围
    overlap_x = max(0, min(r1_right, r2_right) - max(r1_left, r2_left))
    overlap_y = max(0, min(r1_up, r2_up) - max(r1_down, r2_down))
    # max(0, ...) 确保了当没有交集时宽/高为 0
    
    return overlap_x * overlap_y

for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    nums = list(map(float, line.split()))
    area = rect_intersection_area(*nums)   # 用 * 解包列表作为参数
    print(f"{area:.2f}")
```

> **技巧**：`max(0, value)` 可以简洁地将负值截断为 0，避免了额外的 `if` 判断。

---

### ⚠️ 易错提示

1. **对角顶点不一定是"左下"和"右上"**：必须用 `min/max` 重新确定每个矩形的四个边界。这是最容易犯错的地方！
2. **浮点数输出格式**：用 `:.2f` 保留两位小数。
3. **面积为 0 的情况**：仅相切（共用一条边或一个点）时，面积也是 0。`width > 0 and height > 0` 可以正确处理。
4. **不要混淆 8 个数的顺序**：仔细对应哪 4 个属于第一个矩形、哪 4 个属于第二个。

---

## 🟢 HDU 2057 — A + B Again

### 📜 原题描述

**Problem Description**

给定两个**十六进制整数** A 和 B（可能是负数），计算 A + B，结果以十六进制输出。

**Input**

输入包含多组测试数据，每组一行，包含两个十六进制整数 A 和 B。

A 和 B 的绝对值不超过 2^63 - 1（long long 范围）。十六进制字母使用大写 A-F。

**Output**

对于每组输入，以大写十六进制输出 A + B。如果结果为负数，前面加负号 `-`。

**Sample Input**
```
1 9
A B
-A +B
```

**Sample Output**
```
A
15
1
```

---

### 💡 解题思路

Python 可以用 `int(s, 16)` 将十六进制字符串（含负号）转换为整数，加法后再用 `hex()` 或格式化转回十六进制。

需要注意的细节：
- 输入的十六进制字母可能是大写或小写
- 输出要求大写
- 负数前面要加 `-`
- `hex()` 会输出 `0x` 前缀和小写字母

---

### ✅ 解法一：直接利用 `int()` 和格式化（推荐）

```python
import sys

for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    parts = line.split()
    a = int(parts[0], 16)             # 将十六进制字符串转为整数
                                      # int() 第二个参数 16 表示输入是十六进制
                                      # 自动处理负号，如 int("-A", 16) = -10
    b = int(parts[1], 16)             # 同上
    
    result = a + b                    # 整数加法
    
    if result < 0:                    # 如果结果为负
        print('-' + format(-result, 'X'))
        # format(-result, 'X')：
        #   - 取绝对值 -result（此时为正数）
        #   - 'X' 格式说明符：输出大写十六进制，不带 0x 前缀
        # 最前面手动加上 '-'
    else:
        print(format(result, 'X'))    # 正数或零，直接大写十六进制输出
```

> **`format()` 的格式说明符**：
> - `'x'`：小写十六进制，如 `a5f`
> - `'X'`：大写十六进制，如 `A5F`
> - `'b'`：二进制
> - `'o'`：八进制

**复杂度分析**
- **时间复杂度**：O(L)，L 为十六进制字符串长度
- **空间复杂度**：O(L)

---

### ✅ 解法二：使用 `hex()` 函数

```python
import sys

for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    parts = line.split()
    a = int(parts[0], 16)
    b = int(parts[1], 16)
    
    result = a + b
    
    if result >= 0:
        # hex(result) 返回如 "0x1a5f"，去掉前缀 "0x" 并转大写
        print(hex(result)[2:].upper())
    else:
        # hex(result) 对负数返回如 "-0x1a5f"
        # 去掉 "-0x" 前缀（前3个字符），转大写，再加回负号
        print('-' + hex(result)[3:].upper())
```

> **`hex()` 对负数的行为**：
> ```python
> hex(-10)   # → '-0xa'
> hex(10)    # → '0xa'
> ```
> 负数会在前面加 `-`，所以负数需要切掉前 3 个字符 `-0x`。

**复杂度分析**
- 同解法一

---

### ✅ 解法三：f-string 简洁写法

```python
import sys

for line in sys.stdin:
    line = line.strip()
    if not line:
        continue
    a, b = line.split()
    s = int(a, 16) + int(b, 16)       # 直接计算
    
    # f-string 格式化
    if s < 0:
        print(f"-{-s:X}")             # 负数：负号 + 绝对值的大写十六进制
    else:
        print(f"{s:X}")               # 非负数：直接大写十六进制
```

---

### ⚠️ 易错提示

1. **输入可能带正负号**：`int("+B", 16)` 和 `int("-A", 16)` 都能正确处理，Python 的 `int()` 很智能。
2. **输出必须大写**：用 `'X'` 而不是 `'x'`。
3. **结果为 0 的情况**：`format(0, 'X')` = `'0'`，输出正确。
4. **不要输出 `0x` 或 `0X` 前缀**：OJ 要求纯十六进制数字。
5. **Python 整数无上限**：Python 自带大整数支持，不用担心溢出问题（这是 Python 做这题的天然优势）。

---

## 📊 总结速查表

| 题号  |      题名       |    核心考点     |        最优解关键技术         | 时间复杂度 |
| :---: | :-------------: | :-------------: | :---------------------------: | :--------: |
| 2051  |     Bitset      |    进制转换     |     `bin()` / `f"{n:b}"`      |  O(log N)  |
| 2052  |     Picture     |   字符画输出    |     字符串乘法 `'-' * w`      |   O(W×H)   |
| 2053  |   Switch Game   | 数论/完全平方数 |        `math.isqrt()`         |  O(log N)  |
| 2054  |    A == B ?     | 字符串处理/精度 |    `Decimal` 或手动标准化     |    O(L)    |
| 2055  | An Easy Problem |  ASCII 码转换   |      `ord()` + 条件判断       |    O(1)    |
| 2056  |   Rectangles    |  几何/矩形交集  |         区间交集公式          |    O(1)    |
| 2057  |   A + B Again   |  十六进制运算   | `int(s,16)` + `format(n,'X')` |    O(L)    |

---

> **💡 Python 刷 OJ 常用技巧回顾**：
>
> | 技巧           | 代码                                      |
> | -------------- | ----------------------------------------- |
> | 多组输入循环   | `for line in sys.stdin:`                  |
> | 整数进制转换   | `bin()`, `oct()`, `hex()`, `int(s, base)` |
> | 任意精度十进制 | `from decimal import Decimal`             |
> | 精确整数平方根 | `math.isqrt(n)` (Python 3.8+)             |
> | 格式化输出     | `f"{value:.2f}"`, `format(n, 'X')`        |
> | 字符 ↔ ASCII   | `ord('A')` = 65, `chr(65)` = 'A'          |

