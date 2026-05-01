# 🐍 Python 刷题教程（2000–2015）

> **适用人群**：有一定 Python 基础，正在刷 OJ 题目的学习者
> **目标**：快速掌握每道题的解题思路与多种实现方式，可直接作为教材复习使用

---

## 📌 刷题前必知：OJ 输入输出模板

在正式开始之前，先掌握 Python 处理 OJ 多组输入的常用模式：

```python
# 模式一：无限读取，直到 EOF（最常见）
import sys
for line in sys.stdin:
    data = line.split()
    # 处理 data ...

# 模式二：while + try/except
while True:
    try:
        line = input()
        # 处理 ...
    except EOFError:
        break

# 模式三：先读 n，再循环 n 次
n = int(input())
for _ in range(n):
    line = input()
    # 处理 ...
```

### 🔧 教学版补充：更鲁棒的输入模板（推荐）

很多 OJ 数据中可能夹带空行，或者数字行前后有多余空格。教学场景建议封装一个“跳过空行”的读取函数：

```python
import sys

def next_non_empty_line():
    """读取下一行非空内容；若到 EOF 返回 None"""
    for raw in sys.stdin:
        line = raw.strip()
        if line:
            return line
    return None

# 示例：按两行一组读取（第一行 n，第二行 n 个数）
while True:
    line_n = next_non_empty_line()
    if line_n is None:
        break

    n = int(line_n)
    nums_line = next_non_empty_line()
    if nums_line is None:
        break

    nums = list(map(int, nums_line.split()))
    nums = nums[:n]  # 如果题目保证输入规范，这句可省略
    # ... 处理 nums
```

---

## 题目 2000：ASCII 码排序

### 📝 原题描述

> 输入三个字符后，按各字符的 ASCII 码从小到大的顺序输出这三个字符。
>
> **输入**：多组测试数据，每组一行包含三个字符，字符之间用一个空格分隔。
>
> **输出**：对于每组输入数据，输出一行，字符中间用一个空格分隔。

**样例输入**：

```
q s a
```

**样例输出**：

```
a q s
```

---

### 💡 解题思路

**核心知识点**：字符比较在 Python 中天然支持。严格来说，Python 字符串比较按 **Unicode 码点** 进行；本题输入是英文字母/ASCII 字符时，效果与按 ASCII 排序一致。

- **解法一（排序法）**：将三个字符放入列表，调用 `sorted()` 排序后输出。
- **解法二（手动比较 / 冒泡）**：不用内置排序，用条件交换实现，练习基本排序逻辑。

---

### ✅ 解法一：排序法（推荐）

```python
import sys

for line in sys.stdin:                    # 持续读取每一行，直到 EOF
    chars = line.split()                  # 按空格切分，得到三个字符的列表
    chars.sort()                          # 原地排序（按 ASCII 码值升序）
    print(' '.join(chars))               # 用空格连接输出
```

**复杂度分析**：

| 维度 | 复杂度                              |
| ---- | ----------------------------------- |
| 时间 | O(1)（固定 3 个元素，排序是常数级） |
| 空间 | O(1)                                |

---

### ✅ 解法二：手动冒泡排序

```python
import sys

for line in sys.stdin:
    a, b, c = line.split()               # 读取三个字符

    # 两两比较，确保 a <= b <= c（冒泡思想）
    if a > b:                             # 如果第一个 > 第二个，交换
        a, b = b, a
    if a > c:                             # 如果第一个 > 第三个，交换
        a, c = c, a
    if b > c:                             # 如果第二个 > 第三个，交换
        b, c = c, b

    print(a, b, c)                        # print 默认用空格分隔
```

**复杂度分析**：与解法一相同，O(1)。

---

### ⚠️ 易错提示

1. **`input().split()` 返回的是字符串列表**，不是字符。但对于单字符来说效果一致。
2. **不要忘记多组输入**。很多人只处理了一组，提交后 WA。
3. Python 中字符串比较就是按 Unicode 码值，无需手动 `ord()` 转换。

---

## 题目 2001：计算两点间的距离

### 📝 原题描述

> 输入两点坐标 (x1, y1) 和 (x2, y2)，计算并输出两点间的距离。
>
> **输入**：多组测试数据，每组一行包含四个实数 x1, y1, x2, y2。
>
> **输出**：每组输出一行，结果保留两位小数。

**样例输入**：

```
0 0 0 1
```

**样例输出**：

```
1.00
```

---

### 💡 解题思路

**公式**：$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$

- **解法一**：使用 `math.sqrt()`
- **解法二**：使用幂运算 `** 0.5`
- **解法三**：使用 `math.hypot()`（最优雅）

---

### ✅ 解法一：`math.sqrt()`

```python
import sys
import math

for line in sys.stdin:
    x1, y1, x2, y2 = map(float, line.split())   # 读取四个浮点数
    dist = math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2)  # 欧几里得距离公式
    print(f"{dist:.2f}")                          # 保留两位小数
```

---

### ✅ 解法二：幂运算

```python
import sys

for line in sys.stdin:
    x1, y1, x2, y2 = map(float, line.split())
    dist = ((x2 - x1) ** 2 + (y2 - y1) ** 2) ** 0.5   # ** 0.5 等价于开根号
    print(f"{dist:.2f}")
```

---

### ✅ 解法三：`math.hypot()`（推荐）

```python
import sys
import math

for line in sys.stdin:
    x1, y1, x2, y2 = map(float, line.split())
    dist = math.hypot(x2 - x1, y2 - y1)   # hypot(dx, dy) = sqrt(dx² + dy²)
    print(f"{dist:.2f}")
```

**复杂度分析**：

| 维度 | 复杂度    |
| ---- | --------- |
| 时间 | O(1) 每组 |
| 空间 | O(1)      |

---

### ⚠️ 易错提示

1. **输出格式**：必须保留两位小数，`f"{x:.2f}"` 是最可靠的方式。
2. **`math.hypot`** 内部做了数值优化，避免大数溢出，是计算距离的首选。
3. 注意输入是浮点数，用 `float` 而非 `int`。

---

## 题目 2002：计算球体积

### 📝 原题描述

> 根据输入的半径值 r，计算球的体积。
>
> **公式**：$V = \frac{4}{3} \pi r^3$
>
> **输入**：多组测试数据，每行一个实数 r。
>
> **输出**：每行输出对应球的体积，保留三位小数。

**样例输入**：

```
1
1.5
```

**样例输出**：

```
4.189
14.137
```

---

### 💡 解题思路

直接套公式。注意 Python 中整数除法坑。

---

### ✅ 解法一：使用 `math.pi`（推荐）

```python
import sys
import math

for line in sys.stdin:
    r = float(line.strip())                         # 读取半径
    volume = (4 / 3) * math.pi * r ** 3             # 球体积公式
    print(f"{volume:.3f}")                           # 保留三位小数
```

---

### ✅ 解法二：使用 `math.pow`

```python
import sys
import math

for line in sys.stdin:
    r = float(line.strip())
    volume = (4.0 / 3.0) * math.pi * math.pow(r, 3)  # math.pow 返回浮点数
    print(f"{volume:.3f}")
```

**复杂度分析**：O(1) 时间, O(1) 空间。

---

### ⚠️ 易错提示

1. **Python 3 中 `4/3 = 1.333...`** 是浮点除法，这与 C/C++ 不同（C 中 `4/3 = 1`）。但还是建议写 `4.0 / 3.0` 以明确意图。
2. **π 的精度**：必须使用 `math.pi`，不要手写 `3.14159`，精度不够会 WA。
3. 注意保留 **三位** 小数。

---

## 题目 2003：求绝对值

### 📝 原题描述

> 求给定实数的绝对值。
>
> **输入**：多组测试数据，每行一个实数。
>
> **输出**：每行输出绝对值，保留两位小数。

**样例输入**：

```
-5
9
```

**样例输出**：

```
5.00
9.00
```

---

### 💡 解题思路

- **解法一**：内置函数 `abs()`
- **解法二**：条件判断手动实现

---

### ✅ 解法一：`abs()` 函数（推荐）

```python
import sys

for line in sys.stdin:
    x = float(line.strip())       # 读取实数
    print(f"{abs(x):.2f}")        # 取绝对值，保留两位小数
```

---

### ✅ 解法二：条件判断

```python
import sys

for line in sys.stdin:
    x = float(line.strip())
    result = x if x >= 0 else -x   # 手动：非负则取自身，否则取相反数
    print(f"{result:.2f}")
```

**复杂度分析**：O(1) 时间, O(1) 空间。

---

### ⚠️ 易错提示

1. **`-0.00` 输出问题**：`abs(-0.0)` 在 Python 中返回 `0.0`，不会出现 `-0.00`，放心使用。
2. 注意输入可能是整数或小数，统一用 `float()` 转换。

---

## 题目 2004：成绩转换

### 📝 原题描述

> 输入一个百分制成绩，将其转换为对应等级并输出。
>
> **规则**：
> - 90–100 → A
> - 80–89 → B
> - 70–79 → C
> - 60–69 → D
> - 0–59 → E
>
> **输入**：多组测试数据，每行一个整数成绩。
>
> **输出**：每行输出对应等级。

**样例输入**：

```
56
67
100
```

**样例输出**：

```
E
D
A
```

---

### 💡 解题思路

- **解法一**：if-elif-else 条件链（最直观）
- **解法二**：整数除法映射（技巧型）

> **严格题面提醒**：经典 HDU 2004 题面要求当分数不在 `0~100` 范围内时输出 `Score is error!`。教学材料中这一点不能省略，否则会把非法输入误判为 `E`。

---

### ✅ 解法一：if-elif-else（推荐）

```python
import sys

for line in sys.stdin:
    score = int(line.strip())                 # 读取整数成绩

    if score < 0 or score > 100:              # 非法成绩
        print('Score is error!')
    elif 90 <= score <= 100:                  # 90~100 → A
        print('A')
    elif 80 <= score <= 89:                   # 80~89  → B
        print('B')
    elif 70 <= score <= 79:                   # 70~79  → C
        print('C')
    elif 60 <= score <= 69:                   # 60~69  → D
        print('D')
    else:                                     # 0~59   → E
        print('E')
```

---

### ✅ 解法二：整数除法映射

```python
import sys

for line in sys.stdin:
    score = int(line.strip())

    if not 0 <= score <= 100:
        print('Score is error!')
        continue

    # score // 10 得到十位数字：10->A, 9->A, 8->B, 7->C, 6->D, 其余->E
    grade_map = {10: 'A', 9: 'A', 8: 'B', 7: 'C', 6: 'D'}
    grade = grade_map.get(score // 10, 'E')   # 不在字典中的默认为 'E'
    print(grade)
```

**复杂度分析**：O(1) 时间, O(1) 空间。

---

### ⚠️ 易错提示

1. **100 分的情况**：`100 // 10 = 10`，所以字典中必须包含 `10: 'A'`，否则会输出 `E`。
2. **边界值**：确保 60、70、80、90 这些边界归到正确的区间。
3. **非法成绩必须单独处理**：小于 0 或大于 100 时输出 `Score is error!`，不能直接归入 `E`。
4. 注意 Python 不需要 `break`（不像 C 的 `switch`）。

---

## 题目 2005：第几天？

### 📝 原题描述

> 给定一个日期，格式为 YYYY/MM/DD，求该日期是该年的第几天。
>
> **输入**：多组测试数据，每行一个日期。
>
> **输出**：每行输出该日期是当年的第几天。

**样例输入**：

```
2000/1/1
2000/3/1
```

**样例输出**：

```
1
61
```

---

### 💡 解题思路

**核心知识点**：闰年判定 + 每月天数累加。

- **解法一**：手动累加（基础练习）
- **解法二**：使用 `datetime` 库（推荐实战）

**闰年条件**：能被 4 整除且不能被 100 整除，**或** 能被 400 整除。

---

### ✅ 解法一：手动累加

```python
import sys

def is_leap(year):
    """判断是否为闰年"""
    return (year % 4 == 0 and year % 100 != 0) or (year % 400 == 0)

for line in sys.stdin:
    parts = line.strip().split('/')            # 按 '/' 分割
    year, month, day = int(parts[0]), int(parts[1]), int(parts[2])

    # 每月天数表（索引0不使用，1~12月）
    days_in_month = [0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

    if is_leap(year):                          # 闰年二月多一天
        days_in_month[2] = 29

    total = day                                # 先加上当月的天数
    for i in range(1, month):                  # 再累加前面每个月的天数
        total += days_in_month[i]

    print(total)
```

---

### ✅ 解法二：`datetime` 库（推荐）

```python
import sys
from datetime import date

for line in sys.stdin:
    parts = line.strip().split('/')
    year, month, day = int(parts[0]), int(parts[1]), int(parts[2])

    target = date(year, month, day)                # 构建目标日期
    first_day = date(year, 1, 1)                   # 当年第一天
    diff = (target - first_day).days + 1           # 差值 + 1（1月1日是第1天）
    print(diff)
```

**复杂度分析**：

| 维度 | 复杂度                              |
| ---- | ----------------------------------- |
| 时间 | O(1) 每组（月份最多 12 个，常数级） |
| 空间 | O(1)                                |

---

### ⚠️ 易错提示

1. **1 月 1 日是第 1 天**，不是第 0 天！用 `datetime` 时记得 `+1`。
2. **闰年判断不能漏 400 的情况**：2000 年是闰年，1900 年不是。
3. 输入可能是 `2000/1/1` 而不是 `2000/01/01`，用 `split('/')` + `int()` 就不怕前导零问题。

---

## 题目 2006：求奇数的乘积

### 📝 原题描述

> 给定 n 个整数，要求你求出其中所有奇数的乘积。
>
> **输入**：多组测试数据，每组第一行为 n，第二行为 n 个整数。
>
> **输出**：每组输出所有奇数的乘积。（保证至少有一个奇数）

**样例输入**：

```
3
1 2 3
```

**样例输出**：

```
3
```

---

### 💡 解题思路

- **解法一**：循环遍历 + 条件判断
- **解法二**：函数式编程风格（`reduce` + `filter`）

---

### ✅ 解法一：循环（推荐）

```python
import sys

while True:
    try:
        n = int(input())                          # 读取整数个数
        nums = list(map(int, input().split()))[:n]  # 严格按 n 个整数处理

        product = 1                                # 乘积初始化为 1（乘法单位元）
        for x in nums:
            if x % 2 != 0:                         # 判断奇数
                product *= x                       # 累乘

        print(product)
    except EOFError:
        break
```

---

### ✅ 解法二：`reduce` + `filter`

```python
from functools import reduce
import operator

while True:
    try:
        n = int(input())
        nums = list(map(int, input().split()))[:n]

        odds = filter(lambda x: x % 2 != 0, nums)    # 筛选所有奇数
        product = reduce(operator.mul, odds)           # 连乘
        print(product)
    except EOFError:
        break
```

**复杂度分析**：

| 维度 | 复杂度                                                |
| ---- | ----------------------------------------------------- |
| 时间 | O(n) 每组                                             |
| 空间 | O(n)（存储列表），解法二 filter 是惰性求值可降到 O(1) |

---

### ⚠️ 易错提示

1. **乘积初始化为 1**，不是 0！（0 乘任何数都是 0）
2. **负奇数也是奇数**：`-3 % 2` 在 Python 中等于 `1`（Python 余数与除数同号），所以 `x % 2 != 0` 可以正确判断正负奇数。
3. 题目保证至少有一个奇数，不需要特判。
4. **读取了 n 就尽量按 n 处理**：教学中建议使用 `nums[:n]`，避免输入行里多出数字时影响结果。

---

## 题目 2007：平方和与立方和

### 📝 原题描述

> 给定区间 [m, n]，求区间内所有偶数的平方和与所有奇数的立方和。
>
> **输入**：多组测试数据，每行两个整数 m 和 n（m 不一定小于 n）。
>
> **输出**：每行输出偶数的平方和与奇数的立方和，用一个空格分隔。

**样例输入**：

```
1 3
```

**样例输出**：

```
4 28
```

> 解释：偶数 {2}，平方和 = 4；奇数 {1, 3}，立方和 = 1 + 27 = 28

---

### 💡 解题思路

注意 m 和 n 的大小关系不确定，需要先确定区间范围。

- **解法一**：循环遍历
- **解法二**：推导式一行搞定

---

### ✅ 解法一：循环

```python
import sys

for line in sys.stdin:
    m, n = map(int, line.split())

    if m > n:                                    # m 不一定小于 n，需要交换
        m, n = n, m

    even_sq_sum = 0                              # 偶数平方和
    odd_cb_sum = 0                               # 奇数立方和

    for i in range(m, n + 1):                    # 遍历闭区间 [m, n]
        if i % 2 == 0:
            even_sq_sum += i ** 2                # 偶数：累加平方
        else:
            odd_cb_sum += i ** 3                 # 奇数：累加立方

    print(even_sq_sum, odd_cb_sum)
```

---

### ✅ 解法二：推导式

```python
import sys

for line in sys.stdin:
    m, n = map(int, line.split())
    if m > n:
        m, n = n, m

    even_sq = sum(i ** 2 for i in range(m, n + 1) if i % 2 == 0)   # 偶数平方和
    odd_cb = sum(i ** 3 for i in range(m, n + 1) if i % 2 != 0)    # 奇数立方和
    print(even_sq, odd_cb)
```

**复杂度分析**：

| 维度 | 复杂度        |
| ---- | ------------- |
| 时间 | O(n - m) 每组 |
| 空间 | O(1)          |

---

### ⚠️ 易错提示

1. **m 和 n 的大小关系不确定**！必须先处理，否则 `range(m, n+1)` 在 m > n 时为空。
2. **0 是偶数**，如果区间包含 0，0² = 0 会被加入偶数平方和（不影响结果但要理解）。
3. 注意是**闭区间** [m, n]，`range` 右端要 `+1`。

---

## 题目 2008：数值统计

### 📝 原题描述

> 统计给定 n 个数中，负数、零和正数的个数。
>
> **输入**：多组测试数据，每组第一行为 n（n=0 结束），第二行为 n 个数。
>
> **输出**：每组输出负数个数、零的个数、正数个数，用一个空格分隔。

**样例输入**：

```
6
0 1 2 3 -1 0
0
```

**样例输出**：

```
1 2 3
```

---

### 💡 解题思路

- **解法一**：循环计数
- **解法二**：列表推导式 + `sum`

---

### ✅ 解法一：循环计数

```python
while True:
    try:
        n = int(input())
        if n == 0:                                   # n=0 表示结束
            break

        nums = list(map(float, input().split()))[:n]  # 严格按 n 个数据统计
        neg = zero = pos = 0                         # 初始化三个计数器

        for x in nums:
            if x < 0:
                neg += 1
            elif x == 0:
                zero += 1
            else:
                pos += 1

        print(neg, zero, pos)
    except EOFError:
        break
```

---

### ✅ 解法二：推导式

```python
while True:
    try:
        n = int(input())
        if n == 0:
            break

        nums = list(map(float, input().split()))[:n]

        neg  = sum(1 for x in nums if x < 0)        # 负数个数
        zero = sum(1 for x in nums if x == 0)        # 零的个数
        pos  = sum(1 for x in nums if x > 0)         # 正数个数

        print(neg, zero, pos)
    except EOFError:
        break
```

**复杂度分析**：O(n) 时间, O(n) 空间（存储列表）。

---

### ⚠️ 易错提示

1. **终止条件是 `n == 0`**，不是 EOF！但为了鲁棒性，也建议加上 `try/except`。
2. **输入可能是浮点数**（不同 OJ 有不同版本），用 `float` 更安全。
3. **不要忘记输出格式**：三个数之间用空格分隔。
4. **读取了 n 后应与数据个数对应**：教学实现可写成 `nums = nums[:n]`，保证统计范围与题意一致。

---

## 题目 2009：求数列的和

### 📝 原题描述

> 数列的第一项为 n，以后各项为前一项的平方根，求该数列的前 m 项之和。
>
> **输入**：多组测试数据，每行两个正数 n 和 m（m 为正整数）。
>
> **输出**：每行输出数列前 m 项之和，保留两位小数。

**样例输入**：

```
81 4
```

**样例输出**：

```
94.73
```

> 解释：81 + 9 + 3 + 1.732... = 94.73

---

### 💡 解题思路

递推即可：每一项 = 前一项的平方根。

---

### ✅ 解法一：循环递推（推荐）

```python
import sys
import math

for line in sys.stdin:
    parts = line.split()
    n = float(parts[0])            # 首项
    m = int(parts[1])              # 项数

    total = 0.0                    # 累加和
    current = n                    # 当前项

    for _ in range(m):
        total += current           # 先累加
        current = math.sqrt(current)   # 再更新为下一项（前一项的平方根）

    print(f"{total:.2f}")
```

---

### ✅ 解法二：列表构建后求和

```python
import sys
import math

for line in sys.stdin:
    parts = line.split()
    n, m = float(parts[0]), int(parts[1])

    seq = [n]                               # 初始化数列，首项为 n
    for _ in range(m - 1):                  # 再生成 m-1 项
        seq.append(math.sqrt(seq[-1]))      # 每项 = 上一项的平方根

    print(f"{sum(seq):.2f}")                # 求和输出
```

**复杂度分析**：O(m) 时间, O(1) / O(m) 空间。

---

### ⚠️ 易错提示

1. **首项 n 可能是浮点数**，不要用 `int()`。
2. 循环中要**先累加再更新**，否则会错过首项或多算一项。
3. 保留两位小数。

---

## 题目 2010：水仙花数

### 📝 原题描述

> 输出给定区间 [m, n] 内的所有水仙花数。
>
> **水仙花数**：一个三位数，其各位数字的立方和等于该数本身。
> 例如：153 = 1³ + 5³ + 3³
>
> **输入**：多组测试数据，每行两个正整数 m 和 n（100 ≤ m ≤ n ≤ 999）。
>
> **输出**：每组输出一行，输出区间内的水仙花数，用空格分隔。若没有则输出 `no`。

**样例输入**：

```
100 400
```

**样例输出**：

```
153 370 371
```

---

### 💡 解题思路

- **解法一**：循环 + 取位运算
- **解法二**：字符串拆位

---

### ✅ 解法一：取位运算（推荐）

```python
import sys

for line in sys.stdin:
    m, n = map(int, line.split())
    result = []                              # 收集水仙花数

    for num in range(m, n + 1):
        hundreds = num // 100                # 百位
        tens = (num // 10) % 10              # 十位
        ones = num % 10                      # 个位

        if hundreds ** 3 + tens ** 3 + ones ** 3 == num:   # 判断水仙花数
            result.append(str(num))

    if result:
        print(' '.join(result))              # 有水仙花数则输出
    else:
        print('no')                          # 没有则输出 no
```

---

### ✅ 解法二：字符串拆位

```python
import sys

for line in sys.stdin:
    m, n = map(int, line.split())
    result = []

    for num in range(m, n + 1):
        digits = str(num)                          # 转字符串
        cube_sum = sum(int(d) ** 3 for d in digits)  # 每位数字的立方和
        if cube_sum == num:
            result.append(str(num))

    print(' '.join(result) if result else 'no')
```

**复杂度分析**：O(n - m) 时间, O(1) 空间（输出用列表是为了格式化）。

---

### ⚠️ 易错提示

1. **水仙花数只限三位数**（100–999），本题区间也在此范围内。
2. **没有水仙花数时要输出 `no`**，不要输出空行。
3. 所有水仙花数只有四个：**153, 370, 371, 407**，可以用来验证。

---

## 题目 2011：多项式求和

### 📝 原题描述

> 多项式的描述为：
> $$S = 1 - \frac{1}{2} + \frac{1}{3} - \frac{1}{4} + \cdots$$
>
> 求该多项式的前 n 项之和。
>
> **输入**：第一行是测试数据组数 `m`，接下来 `m` 行每行一个正整数 `n`。
>
> **输出**：每组输出一行前 n 项和，保留两位小数。（第 k 项 = `(-1)^(k+1) / k`）

**样例输入**：

```
2
1
3
```

**样例输出**：

```
1.00
0.83
```

---

### 💡 解题思路

第 k 项（从 1 开始）= $\frac{(-1)^{k+1}}{k}$，即奇数项为正、偶数项为负。

这道题最容易错的不是求和本身，而是**输入格式**：经典题面第一行先给测试组数，后面才是每组的 `n`。如果直接按 EOF 逐行处理，在一些 OJ 上会把第一行的组数误当成要求和的项数。

- **解法一**：循环累加 + 符号交替
- **解法二**：推导式

---

### ✅ 解法一：循环 + 符号变量（推荐）

```python
t = int(input())

for _ in range(t):
    n = int(input().strip())

    total = 0.0
    sign = 1                              # 符号变量，初始为正

    for k in range(1, n + 1):
        total += sign / k                 # 当前项 = 符号 / k
        sign = -sign                      # 符号交替

    print(f"{total:.2f}")
```

---

### ✅ 解法二：推导式

```python
t = int(input())

for _ in range(t):
    n = int(input().strip())
    total = sum((-1) ** (k + 1) / k for k in range(1, n + 1))   # 通项公式直接求和
    print(f"{total:.2f}")
```

**复杂度分析**：O(n) 时间, O(1) 空间。

---

### ⚠️ 易错提示

1. **除法必须是浮点除法**。Python 3 中 `1/2 = 0.5` 没问题，但要注意不要用 `//`。
2. **符号变量法比 `(-1)**k` 更快**，避免了每次都做幂运算。
3. **首行是测试组数 `m`**，不要误写成“读到 EOF 为止”。
4. 保留两位小数。

---

## 题目 2012：素数判定

### 📝 原题描述

> 对于表达式 $x^2 + x + 41$，当整数 x 在某个区间 [a, b] 内变化时，判断结果是否都为素数。
>
> **输入**：多组测试数据，每行两个整数 `a, b`，当 `a = b = 0` 时结束。
>
> **输出**：如果区间 [a, b] 内所有 `x^2 + x + 41` 都是素数，输出 `OK`；否则输出 `Sorry`。

**样例输入**：

```
0 1
40 40
0 0
```

**样例输出**：

```
OK
Sorry
```

---

### 💡 解题思路

**核心**：判断一个数是否为素数。

> **数学小知识**：$x^2 + x + 41$ 在 `x = 0~39` 时确实都会得到素数，但这不意味着本题答案永远是 `OK`。例如 `x = 40` 时，结果是 `40^2 + 40 + 41 = 1681 = 41^2`，并不是素数。

---

### ✅ 解法一：逐个判断

```python
import math

def is_prime(num):
    """判断 num 是否是素数"""
    if num < 2:                              # 小于 2 的不是素数
        return False
    if num == 2:                             # 2 是素数
        return True
    if num % 2 == 0:                         # 偶数不是素数
        return False
    for i in range(3, int(math.isqrt(num)) + 1, 2):   # 只检查奇数因子到 √num
        if num % i == 0:
            return False
    return True

while True:
    try:
        a, b = map(int, input().split())
        if a == 0 and b == 0:                # 终止条件
            break

        if a > b:
            a, b = b, a                      # 教学版补充：即使输入顺序颠倒也能处理

        all_prime = True
        for x in range(a, b + 1):
            val = x * x + x + 41             # 计算表达式值
            if not is_prime(val):
                all_prime = False
                break                        # 发现一个非素数即可终止

        print("OK" if all_prime else "Sorry")
    except EOFError:
        break
```

---

### ✅ 解法二：利用 `all()` 函数

```python
import math

def is_prime(num):
    if num < 2:
        return False
    if num == 2:
        return True
    if num % 2 == 0:
        return False
    for i in range(3, int(math.isqrt(num)) + 1, 2):
        if num % i == 0:
            return False
    return True

while True:
    try:
        a, b = map(int, input().split())
        if a == 0 and b == 0:
            break

        if a > b:
            a, b = b, a

        # all() + 生成器：所有值都是素数才返回 True，短路求值
        result = all(is_prime(x * x + x + 41) for x in range(a, b + 1))
        print("OK" if result else "Sorry")
    except EOFError:
        break
```

**复杂度分析**：

| 维度 | 复杂度                          |
| ---- | ------------------------------- |
| 时间 | O((b-a) × √V)，V 为表达式最大值 |
| 空间 | O(1)                            |

---

### ⚠️ 易错提示

1. **终止条件是 `a == b == 0`**，不是 EOF。
2. **素数判断易错点**：
   - 1 不是素数
   - 2 是素数
   - 检查到 `√num` 即可，不需要到 `num-1`
3. **不要把“0~39 都成立”误读成“任意区间都成立”**。`x = 40` 时就会得到合数 1681。
4. **`math.isqrt()`**（Python 3.8+）返回整数平方根，比 `int(math.sqrt())` 更准确，避免浮点误差。

---

## 题目 2013：蟠桃记

### 📝 原题描述

> 孙悟空吃桃问题：猴子第一天摘了若干桃子，当即吃了一半多一个；第二天又吃了剩下的一半多一个……以此类推，到第 n 天早上只剩下 1 个。求第一天共摘了多少个桃子。
>
> **输入**：多组测试数据，每行一个正整数 n。
>
> **输出**：每行输出第一天的桃子总数。

**样例输入**：

```
2
4
```

**样例输出**：

```
4
22
```

---

### 💡 解题思路

**关键推导**：设第 i 天有 `a[i]` 个桃子。

- 吃掉一半多一个后剩余：`a[i+1] = a[i] / 2 - 1`
- 反推：`a[i] = (a[i+1] + 1) * 2`

从第 n 天的 1 个桃子倒推回第一天。

---

### ✅ 解法一：循环反推（推荐）

```python
import sys

for line in sys.stdin:
    n = int(line.strip())

    peaches = 1                              # 第 n 天剩 1 个

    for _ in range(n - 1):                   # 反推 n-1 次
        peaches = (peaches + 1) * 2          # 逆运算

    print(peaches)
```

---

### ✅ 解法二：递归

```python
import sys

def reverse_peach(day, target_day):
    """递归：从 target_day 反推到 day"""
    if day == target_day:
        return 1                             # 最后一天剩 1 个
    return (reverse_peach(day + 1, target_day) + 1) * 2   # 逆推

for line in sys.stdin:
    n = int(line.strip())
    print(reverse_peach(1, n))
```

**复杂度分析**：O(n) 时间, O(1) / O(n) 空间。

---

### ⚠️ 易错提示

1. **反推公式**：是 `(剩余 + 1) * 2`，不是 `剩余 * 2 + 1`。
   - 正向：吃了一半多一个 → `剩 = 总/2 - 1`
   - 反向：`总 = (剩 + 1) × 2`
2. **循环次数是 `n-1`**，不是 `n`（从第 n 天推到第 1 天，共 n-1 步）。
3. 递归解法在 n 较大时可能栈溢出，实际 OJ 中 n 不会太大，但循环更稳健。

---

## 题目 2014：青年歌手大奖赛\_评委会打分

### 📝 原题描述

> 青年歌手大奖赛中，评委会给出的分数去掉一个最高分和一个最低分后，求平均分。
>
> **输入**：多组测试数据，每组第一行为评委人数 n (n > 2)，第二行为 n 个评分。
>
> **输出**：每组输出平均分，保留两位小数。

**样例输入**：

```
6
10 9 8 7.5 9.5 8
```

**样例输出**：

```
8.63
```

---

### 💡 解题思路

- **解法一**：排序后切片
- **解法二**：直接用 `sum - max - min`（推荐，O(n)）

---

### ✅ 解法一：排序后切片

```python
while True:
    try:
        n = int(input())
        scores = list(map(float, input().split()))[:n]

        scores.sort()                                  # 排序
        trimmed = scores[1:-1]                         # 去掉第一个(最小)和最后一个(最大)
        avg = sum(trimmed) / len(trimmed)              # 求平均
        print(f"{avg:.2f}")
    except EOFError:
        break
```

---

### ✅ 解法二：`sum - max - min`（推荐）

```python
while True:
    try:
        n = int(input())
        scores = list(map(float, input().split()))[:n]

        total = sum(scores) - max(scores) - min(scores)   # 总分减最高减最低
        avg = total / (n - 2)                              # 剩余人数 = n-2
        print(f"{avg:.2f}")
    except EOFError:
        break
```

**复杂度分析**：

| 解法        | 时间       | 空间 |
| ----------- | ---------- | ---- |
| 排序        | O(n log n) | O(n) |
| sum-max-min | O(n)       | O(n) |

---

### ⚠️ 易错提示

1. **分母是 `n - 2`**，不是 `n`（去掉了两个分数）。
2. **输入是浮点数**（如 `7.5`），必须用 `float`。
3. 如果有多个最高/最低分相同，**只去掉各一个**，`max()` 和 `min()` 只返回一个值，正合题意。

---

## 题目 2015：偶数求和

### 📝 原题描述

> 有一个长度为 `n` 的偶数序列：`2, 4, 6, ..., 2n`。把它按顺序每 `m` 个分成一组，求每组的平均值。
>
> **输入**：多组测试数据，每行两个正整数 n 和 m。
>
> **输出**：每组按分组顺序输出各组平均值，组与组之间用空格分隔，最后不足 `m` 个的也要输出其平均值。

**样例输入**：

```
4 2
```

**样例输出**：

```
3 7
```

> 解释：偶数序列 {2, 4, 6, 8}，分组 {2, 4}, {6, 8}，平均值分别是 3 和 7。

---

### 💡 解题思路

1. 先生成偶数序列 `2, 4, ..., 2n`
2. 按顺序每 `m` 个分组
3. 计算每组平均值
4. 用空格连接输出

---

### ✅ 解法一：循环分组（推荐）

```python
import sys

for line in sys.stdin:
    n, m = map(int, line.split())

    evens = [2 * i for i in range(1, n + 1)]   # 生成序列 2, 4, 6, ..., 2n

    avgs = []
    for i in range(0, len(evens), m):            # 每 m 个一组
        group = evens[i:i + m]                   # 切片取组（最后一组可能不足 m 个）
        avgs.append(str(sum(group) // len(group)))  # 每组平均值一定是整数

    print(' '.join(avgs))                        # 用空格连接输出
```

---

### ✅ 解法二：边累加边输出

```python
import sys

for line in sys.stdin:
    n, m = map(int, line.split())

    evens = list(range(2, 2 * n + 1, 2))         # 所有偶数：2, 4, ..., 2n
    results = []
    group_sum = 0                                # 当前组的总和
    count = 0

    for num in evens:
        group_sum += num                         # 累加当前偶数
        count += 1
        if count == m:                           # 满 m 个，记录并重置
            results.append(str(group_sum // count))
            group_sum = 0
            count = 0

    if count > 0:                                # 处理最后不满 m 个的情况
        results.append(str(group_sum // count))

    print(' '.join(results))
```

**复杂度分析**：O(n) 时间, O(n) 空间。

---

### ⚠️ 易错提示

1. **这里求的是平均值，不是组和**。如果直接输出 `sum(group)`，会把题意做错。
2. **序列是 `2, 4, 6, ..., 2n`**，不是“1 到 n 之间的偶数”。例如 `n = 4` 时序列应为 `2, 4, 6, 8`。
3. **最后一组可能不足 `m` 个**，平均值要除以该组的实际元素个数。
4. **输出分隔符是空格**，不是逗号；用 `' '.join(...)` 最稳妥。

---

## 📊 总结速查表

| 题号 | 题目           | 核心知识点     | 关键函数/技巧                      |
| ---- | -------------- | -------------- | ---------------------------------- |
| 2000 | ASCII 码排序   | 排序、字符比较 | `sort()`, `sorted()`               |
| 2001 | 两点距离       | 数学公式       | `math.hypot()`, `f"{x:.2f}"`       |
| 2002 | 球体积         | 数学公式       | `math.pi`, `** 3`                  |
| 2003 | 绝对值         | 内置函数       | `abs()`                            |
| 2004 | 成绩转换       | 条件分支       | `if-elif`, `dict.get()`            |
| 2005 | 第几天         | 日期计算、闰年 | `datetime.date`, 闰年判定          |
| 2006 | 奇数乘积       | 循环、条件筛选 | `reduce()`, `filter()`             |
| 2007 | 平方和与立方和 | 区间遍历       | 推导式, `m > n` 交换               |
| 2008 | 数值统计       | 计数           | 三变量计数                         |
| 2009 | 数列求和       | 递推           | `math.sqrt()`                      |
| 2010 | 水仙花数       | 取位运算       | `// 100`, `% 10`                   |
| 2011 | 多项式求和     | 交错级数       | 首行组数 + 符号交替 `sign = -sign` |
| 2012 | 素数判定       | 素数检测       | `math.isqrt()`, 试除法             |
| 2013 | 蟠桃记         | 逆向推导       | `(x + 1) * 2`                      |
| 2014 | 评委打分       | 去极值平均     | `sum - max - min`                  |
| 2015 | 偶数求和       | 分组平均值     | 切片 `[i:i+m]`, `' '.join()`       |

---

## 🎯 刷题通用建议

1. **多组输入是 OJ 标配**：务必用循环 + `try/except EOFError` 或 `sys.stdin` 处理。
2. **格式输出最易丢分**：保留几位小数、用什么分隔符、有没有末尾空格，一定要精准。
3. **Python 3 vs Python 2**：OJ 通常都支持 Python 3，但要确认。`print` 是函数、`/` 是浮点除法——这些都是 Python 3 特性。
4. **善用 Python 特性**：推导式、`sorted()`、`sum()`、`map()`、`zip()` 等内置工具能让代码更简洁。
5. **先暴力后优化**：先保证正确性，再考虑效率。这些基础题规模小，暴力通常就能 AC。
