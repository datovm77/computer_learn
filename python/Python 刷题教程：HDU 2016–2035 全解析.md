# 🐍 Python 刷题教程：HDU 2016–2035 全解析

> **适用对象**：有一定 Python 基础，希望快速复习、系统掌握常见题型的学习者
> **使用方式**：每题包含 **原题 → 思路 → 完整代码（逐行注释）→ 复杂度分析 → 易错提示**，可直接当教材使用

> **审校说明**：本版按“**题意等价、OJ 可提交、概念表述严格**”的标准整理。凡是 Python 中“写起来很方便、但和原题规则不完全等价”的写法，都会明确标注边界，避免误导初学者。

---

## 📑 目录

| 序号 | 题号 | 题目                        |
| ---- | ---- | --------------------------- |
| 1    | 2016 | 数据的交换输出              |
| 2    | 2017 | 字符串统计                  |
| 3    | 2018 | 母牛的故事                  |
| 4    | 2019 | 数列有序!                   |
| 5    | 2020 | 绝对值排序                  |
| 6    | 2021 | 发工资咯 :)                 |
| 7    | 2022 | 海选女主角                  |
| 8    | 2023 | 求平均成绩                  |
| 9    | 2024 | C语言合法标识符             |
| 10   | 2025 | 查找最大元素                |
| 11   | 2026 | 首字母变大写                |
| 12   | 2027 | 统计元音                    |
| 13   | 2028 | Lowest Common Multiple Plus |
| 14   | 2029 | Palindromes                 |
| 15   | 2030 | 汉字统计                    |
| 16   | 2031 | 进制转换                    |
| 17   | 2032 | 杨辉三角                    |
| 18   | 2033 | 人见人爱A+B                 |
| 19   | 2034 | 人见人爱A-B                 |
| 20   | 2035 | 人见人爱A^B                 |

---

## 1. HDU 2016 —— 数据的交换输出

### 📝 题目描述

**输入**：多组测试数据。每组第一行一个整数 `n`（`n > 0` 时继续），第二行 `n` 个整数。  
**处理**：找出这 `n` 个数中**最小的数**，将它与**第一个数**交换。  
**输出**：输出交换后的序列，数与数之间用空格分隔。  
**结束条件**：当 `n = 0` 时结束。

**样例输入**：
```
5
8 3 5 1 7
0
```
**样例输出**：
```
1 3 5 8 7
```

---

### 💡 解题思路

**核心操作**：
1. 遍历列表，找到最小值的**下标**。
2. 将该下标的元素与下标 `0` 的元素交换。
3. 输出交换后的列表。

**注意**：如果最小值本身就是第一个元素，交换等于不动，逻辑不变。

---

### ✅ 解法一：手动遍历找最小值下标（暴力 / 基础写法）

```python
import sys

for line in sys.stdin:                       # 逐行读取输入
    n = int(line.strip())                    # 读取 n
    if n == 0:                               # n 为 0 时结束
        break
    nums = list(map(int, input().split()))   # 读取 n 个整数到列表

    # ---- 手动找最小值下标 ----
    min_idx = 0                              # 假设第 0 个是最小的
    for i in range(1, n):                    # 从第 1 个开始比较
        if nums[i] < nums[min_idx]:          # 发现更小的
            min_idx = i                      # 更新最小值下标

    # ---- 交换 ----
    nums[0], nums[min_idx] = nums[min_idx], nums[0]

    # ---- 输出 ----
    print(' '.join(map(str, nums)))          # 用空格连接并输出
```

### ✅ 解法二：使用 `list.index()` + `min()`（Pythonic 写法）

```python
import sys

for line in sys.stdin:
    n = int(line.strip())
    if n == 0:
        break
    nums = list(map(int, input().split()))

    min_idx = nums.index(min(nums))          # 一行搞定：先求最小值，再找下标
    nums[0], nums[min_idx] = nums[min_idx], nums[0]  # Python 可以直接元组交换

    print(' '.join(map(str, nums)))
```

**差异说明**：
- `min(nums)` 内部会遍历一次列表 → O(n)
- `nums.index(...)` 又会遍历一次 → O(n)
- 总共遍历两次，但写法更简洁。常数级差异，在竞赛中一般可以接受。

---

### 📊 复杂度分析

| 方法        | 时间复杂度       | 空间复杂度   |
| ----------- | ---------------- | ------------ |
| 手动遍历    | O(n)             | O(1)（原地） |
| min + index | O(n)（两次遍历） | O(1)         |

### ⚠️ 易错提示

1. **多组输入**：必须用循环持续读取，直到 `n == 0`。
2. **交换时要用下标**，不要直接赋值 `nums[0] = min(nums)` 然后忘了把原来的 `nums[0]` 放回去。
3. **输出格式**：数字之间是空格，行末**不要有多余空格**。`' '.join(...)` 自动满足。

---

## 2. HDU 2017 —— 字符串统计

### 📝 题目描述

**输入**：第一行 `n`，表示测试组数；之后 `n` 行，每行一个字符串。  
**输出**：每个字符串中**数字字符**（`'0'`~`'9'`）出现的次数。

**样例输入**：
```
3
ABCD00efg
43yert
9yert45
```
**样例输出**：
```
2
2
3
```

---

### 💡 解题思路

逐字符扫描，判断是否在 `'0'` 到 `'9'` 之间，计数即可。

---

### ✅ 解法一：循环遍历 + `str.isdigit()`

```python
n = int(input())                             # 读取测试组数
for _ in range(n):                           # 循环 n 次
    s = input()                              # 读取一行字符串
    count = 0                                # 计数器初始化
    for ch in s:                             # 遍历每个字符
        if ch.isdigit():                     # 判断是否为数字字符
            count += 1                       # 是则计数 +1
    print(count)                             # 输出本行的数字个数
```

### ✅ 解法二：生成器表达式 + `sum()`（一行流）

```python
n = int(input())
for _ in range(n):
    s = input()
    # sum() 对布尔值求和：True 记为 1，False 记为 0
    print(sum(ch.isdigit() for ch in s))
```

### ✅ 解法三：正则表达式

```python
import re

n = int(input())
for _ in range(n):
    s = input()
    # re.findall 返回所有匹配的列表，长度即为个数
    print(len(re.findall(r'\d', s)))
```

---

### 📊 复杂度分析

| 方法         | 时间复杂度 | 空间复杂度       |
| ------------ | ---------- | ---------------- |
| 循环遍历     | O(len(s))  | O(1)             |
| sum + 生成器 | O(len(s))  | O(1)             |
| 正则         | O(len(s))  | O(k)，k 为匹配数 |

### ⚠️ 易错提示

1. `isdigit()` 对中文数字"一二三"也返回 `False`，这里没问题。
2. 注意区分 `isdigit()` / `isnumeric()` / `isdecimal()`——本题只需判断 ASCII 数字，三者都行。
3. 字符串可能包含空格，`input()` 不会截断中间的空格，但会去掉行末换行。

---

## 3. HDU 2018 —— 母牛的故事

### 📝 题目描述

有一头母牛，从第 2 年开始每年生一头小母牛，每头小母牛从出生后**第 4 年**开始也每年生一头小母牛。假设所有牛都不会死，求第 `n` 年共有多少头牛。  
**输入**：每行一个 `n`（`n = 0` 时结束）。  
**输出**：每行对应一个答案。

**样例**：
| n   | 答案 |
| --- | ---- |
| 1   | 1    |
| 2   | 2    |
| 3   | 3    |
| 4   | 4    |
| 5   | 6    |
| 6   | 9    |

---

### 💡 解题思路

**递推关系推导**：
- 第 n 年的牛 = 第 n-1 年所有牛（没有牛死）+ **新出生的牛**
- 新出生的牛 = 第 n-3 年的总牛数（因为要满 3 岁才能生，即第 n-3 年及之前出生的牛在第 n 年都能生）

所以递推公式：

```
f(n) = f(n-1) + f(n-3)    (n >= 4)
f(1) = 1, f(2) = 2, f(3) = 3
```

---

### ✅ 解法一：迭代递推（推荐）

```python
import sys

for line in sys.stdin:
    n = int(line.strip())
    if n == 0:
        break

    # 预计算到第 n 年
    if n <= 3:
        print(n)                             # 前 3 年：1, 2, 3
        continue

    # dp[i] 表示第 i 年的牛的数量
    dp = [0] * (n + 1)                       # 下标 0 不用，从 1 开始
    dp[1], dp[2], dp[3] = 1, 2, 3            # 基础值

    for i in range(4, n + 1):                # 从第 4 年开始递推
        dp[i] = dp[i - 1] + dp[i - 3]       # 递推公式

    print(dp[n])
```

### ✅ 解法二：滚动变量优化空间

```python
import sys

for line in sys.stdin:
    n = int(line.strip())
    if n == 0:
        break

    if n <= 3:
        print(n)
        continue

    # 只保留最近 3 个值
    a, b, c = 1, 2, 3    # 分别对应 f(n-3), f(n-2), f(n-1)

    for i in range(4, n + 1):
        new = c + a       # f(n) = f(n-1) + f(n-3)
        a, b, c = b, c, new  # 滑动窗口前移

    print(c)
```

### ✅ 解法三：递归 + 记忆化（教学用，不推荐竞赛）

```python
import sys
from functools import lru_cache

@lru_cache(maxsize=None)                     # 自动缓存已计算过的值
def cow(n):
    if n <= 3:
        return n
    return cow(n - 1) + cow(n - 3)

for line in sys.stdin:
    n = int(line.strip())
    if n == 0:
        break
    print(cow(n))
```

---

### 📊 复杂度分析

| 方法       | 时间复杂度 | 空间复杂度        |
| ---------- | ---------- | ----------------- |
| 迭代递推   | O(n)       | O(n)              |
| 滚动变量   | O(n)       | O(1)              |
| 记忆化递归 | O(n)       | O(n)（栈 + 缓存） |

### ⚠️ 易错提示

1. **递推公式是 `f(n-1) + f(n-3)`**，不是 `f(n-1) + f(n-2)`（那是斐波那契）。
2. "第四年开始生" → 出生后第 4 年 = 经过了 3 年成长期，所以减 3。
3. 注意输入格式：每行一个 n，`n == 0` 结束，**不是**先给出测试组数。

---

## 4. HDU 2019 —— 数列有序!

### 📝 题目描述

**输入**：多组数据。每组第一行两个整数 `n` 和 `x`，第二行 `n` 个**已从小到大排列**的整数。  
**处理**：将 `x` 插入序列中，使序列仍然有序。  
**输出**：插入后的序列。  
**结束条件**：`n == 0` 且 `x == 0` 时结束。

**样例输入**：
```
5 3
1 2 4 5 7
0 0
```
**样例输出**：
```
1 2 3 4 5 7
```

---

### 💡 解题思路

序列已经有序，只需找到 `x` 应该插入的位置，然后插入。

---

### ✅ 解法一：暴力——直接 append 后排序

```python
import sys

for line in sys.stdin:
    parts = line.split()
    n, x = int(parts[0]), int(parts[1])
    if n == 0 and x == 0:
        break

    nums = list(map(int, input().split()))    # 读取有序序列
    nums.append(x)                            # 把 x 加到末尾
    nums.sort()                               # 重新排序（暴力但简单）

    print(' '.join(map(str, nums)))
```

**时间复杂度**：O(n log n)（因为排序）。序列本身已有序，只需插入一个元素，排序有些浪费。

### ✅ 解法二：线性扫描找插入位置

```python
import sys

for line in sys.stdin:
    parts = line.split()
    n, x = int(parts[0]), int(parts[1])
    if n == 0 and x == 0:
        break

    nums = list(map(int, input().split()))

    # 从前往后找插入位置
    pos = n                                   # 默认插在最后
    for i in range(n):
        if nums[i] >= x:                      # 找到第一个 >= x 的位置
            pos = i
            break

    nums.insert(pos, x)                       # list.insert() 在 pos 处插入 x
    print(' '.join(map(str, nums)))
```

### ✅ 解法三：二分查找（更推荐）

```python
import sys
import bisect                                 # Python 标准库的二分模块

for line in sys.stdin:
    parts = line.split()
    n, x = int(parts[0]), int(parts[1])
    if n == 0 and x == 0:
        break

    nums = list(map(int, input().split()))

    bisect.insort(nums, x)                    # 二分查找 + 插入，一步到位
    # insort 内部：先 bisect_right 找位置 O(log n)，再 list.insert O(n)
    # 总体 O(n)（因为 insert 要移动元素），但查找部分是 O(log n)

    print(' '.join(map(str, nums)))
```

---

### 📊 复杂度分析

| 方法              | 时间复杂度              | 空间复杂度 |
| ----------------- | ----------------------- | ---------- |
| append + sort     | O(n log n)              | O(1) 原地  |
| 线性扫描 + insert | O(n)                    | O(1)       |
| bisect.insort     | O(n)（insert 移动元素） | O(1)       |

### ⚠️ 易错提示

1. **`bisect.insort` vs `bisect.insort_left`**：`insort` 默认等价于 `insort_right`，即相同元素插在右边。本题无影响。
2. **`list.insert(pos, x)`** 会把 `pos` 及之后的元素全部右移，所以最坏 O(n)。
3. 读取格式：`n` 和 `x` 在**同一行**。

---

## 5. HDU 2020 —— 绝对值排序

### 📝 题目描述

**输入**：多组数据。每行第一个整数 `n`（`n > 0`），后面跟 `n` 个整数。  
**处理**：将这 `n` 个整数按**绝对值**从大到小排序。  
**输出**：排序后的序列。  
**结束条件**：`n == 0`。

**样例输入**：
```
3 3 -4 2
4 0 1 2 -3
0
```
**样例输出**：
```
-4 3 2
-3 2 1 0
```

---

### 💡 解题思路

- Python 的 `sorted()` / `list.sort()` 支持 `key` 参数。
- 设置 `key=abs` 即可按绝对值排序，再设 `reverse=True` 从大到小。

---

### ✅ 解法一：内置排序 + key=abs（最 Pythonic）

```python
import sys

for line in sys.stdin:
    parts = list(map(int, line.split()))
    n = parts[0]                              # 第一个数是 n
    if n == 0:
        break
    nums = parts[1:]                          # 剩下的是 n 个整数

    # 按绝对值从大到小排序
    nums.sort(key=abs, reverse=True)

    print(' '.join(map(str, nums)))
```

### ✅ 解法二：手动实现选择排序（教学理解用）

```python
import sys

for line in sys.stdin:
    parts = list(map(int, line.split()))
    n = parts[0]
    if n == 0:
        break
    nums = parts[1:]

    # 选择排序：每次选出绝对值最大的放到前面
    for i in range(n):
        max_idx = i
        for j in range(i + 1, n):
            if abs(nums[j]) > abs(nums[max_idx]):
                max_idx = j
        nums[i], nums[max_idx] = nums[max_idx], nums[i]

    print(' '.join(map(str, nums)))
```

---

### 📊 复杂度分析

| 方法     | 时间复杂度 | 空间复杂度      |
| -------- | ---------- | --------------- |
| 内置排序 | O(n log n) | O(n)（Timsort） |
| 选择排序 | O(n²)      | O(1)            |

### ⚠️ 易错提示

1. **`key=abs`** 是关键，不要写成 `key=lambda x: -abs(x)` 再正序排——虽然结果一样但画蛇添足。
2. 输入的 `n` 和数据在**同一行**，用 `parts[1:]` 截取。
3. 排序是**降序**（从大到小），别忘了 `reverse=True`。

---

## 6. HDU 2021 —— 发工资咯 :)

### 📝 题目描述

**输入**：每组第一行 `n`，后面 `n` 个整数分别表示每位老师的工资。  
**处理**：每位老师的工资按 **100、50、10、5、2、1** 的面额拆分，计算最少需要多少张/枚钱。  
**输出**：输出这 `n` 位老师合计最少需要的张数（每组一行）。  
**结束条件**：`n == 0`。

**样例输入**：
```
3
1 2 3
0
```
**样例输出**：
```
4
```

> 解释：1 元、2 元、3 元分别需要 1、1、2 张，总计 4 张。

---

### 💡 解题思路

经典的**贪心**问题：优先使用面值大的钞票。  
面额：`[100, 50, 10, 5, 2, 1]`  
对每个工资，依次除以面额，累加商，余数继续往下；最后把所有老师的张数求和输出。

---

### ✅ 解法一：标准贪心

```python
import sys

denominations = [100, 50, 10, 5, 2, 1]      # 面额从大到小

for line in sys.stdin:
    n = int(line.strip())
    if n == 0:
        break
    salaries = list(map(int, input().split()))

    total = 0                                 # 全体老师总张数
    for salary in salaries:
        count = 0                             # 本位老师所需张数
        remain = salary                       # 剩余金额
        for d in denominations:               # 从最大面额开始
            count += remain // d              # 该面额能用几张
            remain %= d                       # 取余，剩下的继续
        total += count

    print(total)
```

### ✅ 解法二：使用 `divmod` 简化

```python
import sys

denoms = [100, 50, 10, 5, 2, 1]

for line in sys.stdin:
    n = int(line.strip())
    if n == 0:
        break
    salaries = list(map(int, input().split()))

    total = 0
    for salary in salaries:
        count = 0
        for d in denoms:
            q, salary = divmod(salary, d)     # divmod 同时返回商和余数
            count += q
        total += count

    print(total)
```

### ✅ 解法三：推导式写法（紧凑版）

```python
import sys

denoms = (100, 50, 10, 5, 2, 1)

for line in sys.stdin:
    n = int(line.strip())
    if n == 0:
        break

    salaries = list(map(int, input().split()))

    def bills(x):
        cnt = 0
        for d in denoms:
            q, x = divmod(x, d)
            cnt += q
        return cnt

    print(sum(bills(s) for s in salaries))
```

---

### 📊 复杂度分析

| 方法     | 时间复杂度           | 空间复杂度           |
| -------- | -------------------- | -------------------- |
| 每个工资 | O(6) ≈ O(1)          | O(1)                 |
| 每组数据 | O(n)（n 为老师人数） | O(n)（存储工资列表） |

### ⚠️ 易错提示

1. 面额列表**必须从大到小排列**，否则贪心不对。
2. 输出是**总张数一行**，不是每位老师各一行。
3. 工资保证是正整数，不需要处理小数；结束标记由 `n == 0` 给出。

---

## 7. HDU 2022 —— 海选女主角

### 📝 题目描述

**输入**：多组数据。每组第一行两个整数 `m`（行数）、`n`（列数），后面 `m` 行每行 `n` 个整数，构成 `m×n` 矩阵。  
**处理**：找出矩阵中**绝对值最大**的元素（保证唯一），输出其**行号、列号、值**（行列从 1 开始）。  
**结束条件**：EOF。

**样例输入**：
```
2 3
1 -2 3
4 -5 6
```
**样例输出**：
```
2 3 6
```

---

### 💡 解题思路

遍历整个矩阵，维护绝对值最大的元素及其位置。

---

### ✅ 解法一：双重循环遍历

```python
import sys

while True:
    try:
        m, n = map(int, input().split())      # 读取行数和列数
    except EOFError:                          # 读到文件末尾退出
        break

    matrix = []
    for i in range(m):
        row = list(map(int, input().split()))
        matrix.append(row)

    # 初始化：假设第一个元素绝对值最大
    max_val = matrix[0][0]
    max_r, max_c = 0, 0

    for i in range(m):
        for j in range(n):
            if abs(matrix[i][j]) > abs(max_val):
                max_val = matrix[i][j]        # 记录原值（不是绝对值）
                max_r, max_c = i, j

    # 输出行号、列号（从 1 开始），以及原值
    print(f"{max_r + 1} {max_c + 1} {max_val}")
```

### ✅ 解法二：压平矩阵 + max with key

```python
import sys

while True:
    try:
        m, n = map(int, input().split())
    except EOFError:
        break

    matrix = []
    for i in range(m):
        matrix.append(list(map(int, input().split())))

    # 构造 (绝对值, 行, 列, 原值) 的列表，取 max
    best = max(
        (abs(matrix[i][j]), i, j, matrix[i][j])
        for i in range(m) for j in range(n)
    )
    # best = (max_abs, row, col, original_value)

    print(f"{best[1] + 1} {best[2] + 1} {best[3]}")
```

---

### 📊 复杂度分析

| 方法       | 时间复杂度 | 空间复杂度 |
| ---------- | ---------- | ---------- |
| 双重循环   | O(m × n)   | O(m × n)   |
| 压平 + max | O(m × n)   | O(m × n)   |

### ⚠️ 易错提示

1. **输出原值**，不是绝对值！比较时用绝对值，输出时用原始值。
2. 行列号**从 1 开始**，不是 0。
3. 多组数据，用 `try-except EOFError` 处理结束。

---

## 8. HDU 2023 —— 求平均成绩

### 📝 题目描述

**输入**：多组数据。每组第一行两个整数 `n`（学生数）、`m`（课程数）。之后 `n` 行，每行 `m` 个成绩。  
**输出**：
1. 每个学生的平均成绩（保留 2 位小数），一行输出，空格分隔。
2. 每门课的平均成绩（保留 2 位小数），一行输出，空格分隔。
3. 所有课成绩**都不低于**该科平均分的学生人数。

每两组输出之间空一行。

**样例输入**：
```
2 2
80 90
85 95
```
**样例输出**：
```
85.00 90.00
82.50 92.50
0
```

---

### 💡 解题思路

1. 读入 `n × m` 的成绩矩阵。
2. 按行求平均 → 学生平均成绩。
3. 按列求平均 → 课程平均成绩。
4. 遍历每个学生，检查其**每门课**的成绩是否 ≥ 该课平均分。

---

### ✅ 解法：矩阵操作

```python
import sys

first = True                                  # 控制空行输出

while True:
    try:
        n, m = map(int, input().split())
    except EOFError:
        break

    if not first:                             # 两组输出之间空一行
        print()
    first = False

    # 读取成绩矩阵
    scores = []
    for i in range(n):
        row = list(map(int, input().split()))
        scores.append(row)

    # ---- 第 1 行输出：每个学生的平均成绩 ----
    student_avg = []
    for i in range(n):
        avg = sum(scores[i]) / m              # 第 i 个学生的平均
        student_avg.append(avg)
    print(' '.join(f"{a:.2f}" for a in student_avg))

    # ---- 第 2 行输出：每门课的平均成绩 ----
    course_avg = []
    for j in range(m):
        total = sum(scores[i][j] for i in range(n))  # 第 j 门课的总分
        avg = total / n
        course_avg.append(avg)
    print(' '.join(f"{a:.2f}" for a in course_avg))

    # ---- 第 3 行输出：所有科目都 >= 该科均分的学生数 ----
    count = 0
    for i in range(n):
        # 检查学生 i 的每门课成绩
        all_pass = True
        for j in range(m):
            if scores[i][j] < course_avg[j]:  # 低于该科平均分
                all_pass = False
                break                         # 有一门不及格就跳出
        if all_pass:
            count += 1
    print(count)
```

### ✅ 简化写法（使用 `all()` + 列表推导）

```python
import sys

first = True

while True:
    try:
        n, m = map(int, input().split())
    except EOFError:
        break

    if not first:
        print()
    first = False

    scores = [list(map(int, input().split())) for _ in range(n)]

    # 学生平均
    stu_avg = [sum(row) / m for row in scores]
    print(' '.join(f"{a:.2f}" for a in stu_avg))

    # 课程平均
    crs_avg = [sum(scores[i][j] for i in range(n)) / n for j in range(m)]
    print(' '.join(f"{a:.2f}" for a in crs_avg))

    # 统计满足条件的学生
    count = sum(
        1 for i in range(n)
        if all(scores[i][j] >= crs_avg[j] for j in range(m))
    )
    print(count)
```

---

### 📊 复杂度分析

| 操作         | 时间复杂度   |
| ------------ | ------------ |
| 学生平均     | O(n × m)     |
| 课程平均     | O(n × m)     |
| 统计合格学生 | O(n × m)     |
| **总计**     | **O(n × m)** |

### ⚠️ 易错提示

1. **两组数据之间空一行**，但最后一组后面**不能空行**。用 `first` 标志控制。
2. **保留 2 位小数**：用 `f"{val:.2f}"` 或 `"%.2f" % val`。
3. **是否 ≥ 该科平均分**：用的是**课程平均分**，不是学生自己的平均分！
4. 浮点数比较：本题精度要求不高，直接 `>=` 即可。

---

## 9. HDU 2024 —— C语言合法标识符

### 📝 题目描述

**输入**：第一行 `n`，之后 `n` 行，每行一个字符串。  
**处理**：判断每个字符串是否是**合法的 C 语言标识符**。  
**规则**：
- 由字母、数字、下划线组成。
- **不能以数字开头**。
- **不能为空字符串**。

**输出**：合法输出 `yes`，否则输出 `no`。

**样例输入**：
```
3
12abc
abc
_a1
```
**样例输出**：
```
no
yes
yes
```

---

### 💡 解题思路

检查两个条件：
1. 首字符：必须是 **ASCII 字母** 或下划线。
2. 其余字符：必须是 **ASCII 字母、数字或下划线**。

这里要特别注意：题目说的是 **C 语言标识符**。  
因此“字母”和“数字”应按 **ASCII 规则** 理解，不能直接把 Python 的 Unicode 字符分类规则照搬过来。

---

### ✅ 解法一：手动逐字符检查

```python
n = int(input())
for _ in range(n):
    s = input()

    if len(s) == 0:                           # 空串不合法
        print("no")
        continue

    def is_ascii_letter(ch):
        return ('a' <= ch <= 'z') or ('A' <= ch <= 'Z')

    # 检查首字符
    if not (is_ascii_letter(s[0]) or s[0] == '_'):
        print("no")
        continue

    # 检查剩余字符
    valid = True
    for ch in s[1:]:
        if not (is_ascii_letter(ch) or ('0' <= ch <= '9') or ch == '_'):
            valid = False
            break

    print("yes" if valid else "no")
```

### ✅ 解法二：正则表达式（ASCII 规则清晰）

```python
import re

n = int(input())
for _ in range(n):
    s = input()
    # ^[a-zA-Z_]    → 首字符：ASCII 字母或下划线
    # [a-zA-Z0-9_]* → 后续字符：ASCII 字母、数字、下划线
    if re.fullmatch(r'[a-zA-Z_][a-zA-Z0-9_]*', s):
        print("yes")
    else:
        print("no")
```

### ⚠️ 补充说明：`str.isidentifier()` 为什么不适合作为本题标准答案

```python
n = int(input())
for _ in range(n):
    s = input()
    print(s.isidentifier())
```

`isidentifier()` 只能说明“它像不像 **Python 标识符**”，不能说明“它是不是 **C 语言合法标识符**”。  
例如它会接受很多非 ASCII 字符，因此**不建议**把它当本题标准解。

---

### 📊 复杂度分析

| 方法       | 时间复杂度 | 空间复杂度 |
| ---------- | ---------- | ---------- |
| 逐字符检查 | O(len(s))  | O(1)       |
| 正则       | O(len(s))  | O(1)       |

### ⚠️ 易错提示

1. **输入可能包含空格**！如 `"hello world"` 应判为 `no`。注意 `input()` 读取整行（含空格）。
2. **空串**也要处理。正则的 `+` 和 `*` 不同：首字符用 `[a-zA-Z_]`（必须有一个），后续用 `*`。
3. `isalpha()`、`isalnum()`、`isidentifier()` 都带有 **Unicode 语义**，严格按本题做时要优先使用 **ASCII 明确判断**。
4. C 语言标识符不能是关键字，但本题**通常不考关键字过滤**；若题面明确要求，再额外判断。

---

## 10. HDU 2025 —— 查找最大元素

### 📝 题目描述

**输入**：多个字符串（每行一个），直到 EOF。  
**处理**：找出字符串中 ASCII 值最大的字符，在它**每次出现**的位置后面插入 `"(max)"`。  
**输出**：处理后的字符串。

**样例输入**：
```
abcdefgfedcba
aaa
```
**样例输出**：
```
abcdefg(max)fedcba
a(max)a(max)a(max)
```

---

### 💡 解题思路

1. 先找出最大字符 `max_ch = max(s)`。
2. 遍历字符串，遇到 `max_ch` 就在后面追加 `"(max)"`。

---

### ✅ 解法一：列表拼接

```python
import sys

for line in sys.stdin:
    s = line.rstrip('\n')                     # 仅去掉行末换行，保留其余空白
    if s == "":                              # 空行原样输出
        print()
        continue

    max_ch = max(s)                           # 找出最大字符
    result = []
    for ch in s:
        result.append(ch)                     # 先加入原字符
        if ch == max_ch:
            result.append("(max)")            # 如果是最大字符，追加标记
    print(''.join(result))
```

### ✅ 解法二：字符串的 `replace` 方法

```python
import sys

for line in sys.stdin:
    s = line.rstrip('\n')
    if s == "":
        print()
        continue

    max_ch = max(s)
    # 将每个 max_ch 替换为 max_ch + "(max)"
    print(s.replace(max_ch, max_ch + "(max)"))
```

这种写法非常简洁：`str.replace(old, new)` 替换所有出现的 old。

---

### 📊 复杂度分析

| 方法     | 时间复杂度 | 空间复杂度 |
| -------- | ---------- | ---------- |
| 列表拼接 | O(n)       | O(n)       |
| replace  | O(n)       | O(n)       |

### ⚠️ 易错提示

1. 最大字符可能**出现多次**，每次都要加 `"(max)"`。
2. `max(s)` 比较的是 ASCII 值 / Unicode 码点，对纯英文字母来说就是字典序最大的。
3. 多组输入到 EOF，用 `for line in sys.stdin`。

---

## 11. HDU 2026 —— 首字母变大写

### 📝 题目描述

**输入**：多行英文句子（直到 EOF）。  
**处理**：将每个单词的首字母变为大写，其余不变。  
**输出**：处理后的句子。

**样例输入**：
```
i am a student
```
**样例输出**：
```
I Am A Student
```

---

### 💡 解题思路

"单词"由**空格**分隔。最稳妥的做法是：
- 维护一个标志 `new_word`，表示当前位置是否处在“一个单词的开头”；
- 遇到空格就把 `new_word` 置为 `True`；
- 遇到非空格字符时，如果 `new_word=True`，就把它转成大写。

这种写法和题意最一致，也不会被 `title()`、`capitalize()` 的附加规则误伤。

---

### ✅ 解法一：状态机扫描（最推荐，最贴题）

```python
import sys

for line in sys.stdin:
    s = line.rstrip('\n')
    result = []
    new_word = True                           # 行首视为新单词开头

    for ch in s:
        if ch == ' ':
            result.append(ch)
            new_word = True                   # 空格后下一个非空格字符是单词首字母
        else:
            if new_word:
                result.append(ch.upper())
                new_word = False
            else:
                result.append(ch)

    print(''.join(result))
```

### ✅ 解法二：`split(' ')` + 手动首字母大写

```python
import sys

for line in sys.stdin:
    s = line.rstrip('\n')
    if s == "":
        print()
        continue

    words = s.split(' ')                      # 按空格分割
    result = []
    for word in words:
        if word:                              # 非空单词
            result.append(word[0].upper() + word[1:])  # 首字母大写 + 其余部分
        else:
            result.append(word)               # 空串保持不变（处理连续空格）
    print(' '.join(result))
```

### ⚠️ 补充说明：`str.title()` / `capitalize()` 的边界

```python
s = "it's a test"
print(s.title())               # It'S A Test
print(' '.join(w.capitalize() for w in s.split(' ')))  # It's A Test
```

`title()` 会把“非字母字符后”的字母也视作单词开头；  
`capitalize()` 会把一个单词中**除首字母外的其他字母全部转成小写**。  
因此这两个方法都适合作为“技巧补充”，但**不建议作为本题标准提交代码**。

---

### 📊 复杂度分析

推荐的两种做法均为 **O(n)**，n 为字符串长度。

### ⚠️ 易错提示

1. 题目按**空格分词**，不是按“所有非字母字符”分词，所以 `title()` 不是完全等价的题解。
2. `str.capitalize()` 会把**非首字母**变小写；若原串中存在大写字母，信息会被改掉。
3. `split(' ')` 和 `split()` 的区别：`split()` 会合并连续空格，`split(' ')` 不会。本题若想尽量保留原空格布局，应使用 `split(' ')` 或状态机扫描。

---

## 12. HDU 2027 —— 统计元音

### 📝 题目描述

**输入**：第一行 `n`，之后 `n` 行字符串。  
**输出**：对每个字符串，依次输出 `a, e, i, o, u` 各出现的次数。  
**注意格式**：每组测试数据输出 5 行，**组与组之间再空一行**。

**样例输入**：
```
2
aeiou
my name is ignatius
```
**样例输出**：
```
a:1
e:1
i:1
o:1
u:1

a:2
e:1
i:3
o:0
u:1
```

---

### 💡 解题思路

对每个字符串，统计 5 个元音字母各出现几次。

---

### ✅ 解法一：`str.count()` 方法

```python
n = int(input())
for case in range(n):
    s = input()
    for vowel in 'aeiou':                     # 依次统计 5 个元音
        print(f"{vowel}:{s.count(vowel)}")    # count() 返回出现次数
    if case != n - 1:                         # 组与组之间空一行
        print()
```

> `str.count(sub)` 内部遍历一次字符串，5 个元音调用 5 次，总计 O(5n) = O(n)。

### ✅ 解法二：一次遍历 + 字典计数

```python
n = int(input())
for case in range(n):
    s = input()
    counts = {v: 0 for v in 'aeiou'}         # 初始化 5 个计数器
    for ch in s:                              # 遍历一次
        if ch in counts:                      # 如果是元音
            counts[ch] += 1
    for vowel in 'aeiou':                     # 按顺序输出
        print(f"{vowel}:{counts[vowel]}")
    if case != n - 1:
        print()
```

### ✅ 解法三：`collections.Counter`

```python
from collections import Counter

n = int(input())
for case in range(n):
    s = input()
    c = Counter(s)                            # 统计所有字符频次
    for vowel in 'aeiou':
        print(f"{vowel}:{c.get(vowel, 0)}")   # get 防止 KeyError
    if case != n - 1:
        print()
```

---

### 📊 复杂度分析

所有方法均为 **O(n)**（n 为字符串长度）。

### ⚠️ 易错提示

1. **只统计小写元音**，本题输入一般不含大写。如果题目可能含大写，需要做 `.lower()` 处理。
2. 输出格式是 `a:1`，冒号后面**没有空格**。
3. **组与组之间要空一行**。很多代码逻辑是对的，但会因为漏掉这个空行格式而 `Presentation Error / Wrong Answer`。
4. **注意读入**：`input()` 可以读取含空格的行，适合本题。

---

## 13. HDU 2028 —— Lowest Common Multiple Plus

### 📝 题目描述

**输入**：多组数据。每组第一行一个 `n`，第二个数到第 n+1 个数为 `n` 个正整数。  
**输出**：这 `n` 个数的**最小公倍数（LCM）**。

> 更准确地说，每行输入是 `n a1 a2 ... an`。

**样例输入**：
```
2 4 6
3 2 5 7
```
**样例输出**：
```
12
70
```

---

### 💡 解题思路

**核心公式**：  
- `lcm(a, b) = a * b // gcd(a, b)`  
- 多个数的 LCM：从左到右依次取 LCM → `lcm(a, b, c) = lcm(lcm(a, b), c)`

---

### ✅ 解法一：math.gcd + 手写 lcm（Python 3.8 及以下）

```python
import sys
from math import gcd

def lcm(a, b):
    return a * b // gcd(a, b)                 # LCM 公式

for line in sys.stdin:
    parts = list(map(int, line.split()))
    n = parts[0]
    nums = parts[1:]

    result = nums[0]                          # 从第一个数开始
    for i in range(1, n):
        result = lcm(result, nums[i])         # 逐个累积 LCM

    print(result)
```

### ✅ 解法二：Python 3.9+ 使用 `math.lcm`

```python
import sys
from math import lcm

for line in sys.stdin:
    parts = list(map(int, line.split()))
    n = parts[0]
    nums = parts[1:]

    print(lcm(*nums))                         # math.lcm 直接支持多个参数
```

### ✅ 解法三：`functools.reduce` + lcm

```python
import sys
from math import gcd
from functools import reduce

for line in sys.stdin:
    parts = list(map(int, line.split()))
    nums = parts[1:]

    result = reduce(lambda a, b: a * b // gcd(a, b), nums)
    print(result)
```

---

### 📊 复杂度分析

| 方法     | 时间复杂度          | 空间复杂度 |
| -------- | ------------------- | ---------- |
| 逐个 lcm | O(n × log(max_val)) | O(1)       |
| reduce   | O(n × log(max_val)) | O(1)       |

> `gcd` 的时间复杂度为 O(log(min(a,b)))。

### ⚠️ 易错提示

1. **整数溢出**：Python 有大整数支持，不用担心。但 C/C++ 需要注意。
2. **`gcd(0, x)` = `x`**，但本题保证正整数，无需处理 0。
3. 输入格式：`n` 和数列在**同一行**。

---

## 14. HDU 2029 —— Palindromes（回文串判断）

### 📝 题目描述

**输入**：第一行 `n`，之后 `n` 行字符串。  
**输出**：每行输出 `yes`（是回文串）或 `no`（不是）。

**样例输入**：
```
3
level
abcba
noon
```
**样例输出**：
```
yes
yes
yes
```

---

### 💡 解题思路

回文串 = 正读和反读一样。

---

### ✅ 解法一：切片反转比较（最 Pythonic）

```python
n = int(input())
for _ in range(n):
    s = input()
    print("yes" if s == s[::-1] else "no")    # s[::-1] 反转字符串
```

> `s[::-1]` 的时间和空间复杂度都是 O(n)。

### ✅ 解法二：双指针法

```python
n = int(input())
for _ in range(n):
    s = input()
    left, right = 0, len(s) - 1              # 两端指针
    is_palindrome = True
    while left < right:
        if s[left] != s[right]:               # 发现不匹配
            is_palindrome = False
            break
        left += 1                             # 向中间靠拢
        right -= 1
    print("yes" if is_palindrome else "no")
```

> 双指针空间 O(1)，不需要创建反转副本。

---

### 📊 复杂度分析

| 方法     | 时间复杂度 | 空间复杂度 |
| -------- | ---------- | ---------- |
| 切片反转 | O(n)       | O(n)       |
| 双指针   | O(n)       | O(1)       |

### ⚠️ 易错提示

1. 本题是**纯字符串**回文，不需要忽略大小写或特殊字符。
2. **空串是回文串**（本题一般不会输入空串）。
3. 单个字符也是回文串。

---

## 15. HDU 2030 —— 汉字统计

### 📝 题目描述

**输入**：第一行 `n`，之后 `n` 行文本。  
**输出**：每行文本中**汉字的个数**。

> 这题是经典老题，题面提示通常是“**从汉字机内码的特点考虑**”。  
> 也就是说，**原题的主流判题思路更接近 GBK/本地编码下的双字节统计**，不是现代 Unicode 文本处理题。  
> 因此在 Python 教学里要分清：
> 1. **OJ 推荐提交版**：按旧题字节语义处理；
> 2. **Unicode 扩展版**：适合现代文本处理知识补充，但不一定与老题判题完全一致。

---

### 💡 解题思路

如果按 HDU 老题的思路做，可以把一行文本先按 `gbk` 编码成字节串，再统计其中的“双字节汉字”。  
在本题常见数据中，英文字符占 1 字节，汉字占 2 字节，因此可以顺着字节流扫描。

---

### ✅ 解法一：按老题判题语义统计（推荐提交版）

```python
import sys

n = int(sys.stdin.readline().strip())
for _ in range(n):
    s = sys.stdin.readline().rstrip('\n')

    # HDU 老题更接近“本地编码下双字节汉字统计”
    raw = s.encode('gbk')
    i = 0
    count = 0

    while i < len(raw):
        if raw[i] > 127:                      # 非 ASCII，按双字节字符处理
            count += 1
            i += 2
        else:
            i += 1

    print(count)
```

### ✅ 解法二：Unicode 范围判断（现代 Python 文本处理版）

```python
n = int(input())
for _ in range(n):
    s = input()
    print(sum(1 for ch in s if '\u4e00' <= ch <= '\u9fff'))
```

> 这一版适合讲 Unicode 基础，但**不一定与 HDU 老题评测完全一致**。

---

### 📊 复杂度分析

两种做法本质上都是线性扫描，时间复杂度均为 **O(L)**，`L` 为文本长度（字符数或字节数）。

### ⚠️ 易错提示

1. **HDU 原题更建议按旧题编码语义处理**，也就是优先使用“GBK 双字节统计”版。
2. Unicode 范围 `\u4e00` ~ `\u9fff` 只覆盖**基本区汉字**，更适合现代文本处理教学，不保证和老 OJ 完全同判。
3. “非 ASCII 双字节字符就算一个汉字”是基于本题常见数据分布的竞赛写法；若扩展到更一般的中文文本，中文标点等字符需要单独区分。

---

## 16. HDU 2031 —— 进制转换

### 📝 题目描述

**输入**：每行两个整数 `N` 和 `R`（常见题面为 `2 <= R <= 16`，且 `R != 10`）。  
**处理**：将 `N` 转换为 `R` 进制表示。  
**输出**：转换后的 R 进制数。负数要加负号。  
- 10~15 用 `A`~`F` 表示。

**样例输入**：
```
7 2
23 12
-4 3
```
**样例输出**：
```
111
1B
-11
```

---

### 💡 解题思路

经典的「**短除法**」：
1. 取 `N` 的绝对值。
2. 反复对 `R` 取余，余数就是当前最低位。
3. 将 `N` 整除 `R`，继续。
4. 最后把余数**逆序**拼接。
5. 如果原始 `N` 为负，加负号。

---

### ✅ 解法一：手动短除法

```python
import sys

digits = "0123456789ABCDEF"                   # 进制字符表

for line in sys.stdin:
    N, R = map(int, line.split())

    if N == 0:                                # 特殊处理 0
        print(0)
        continue

    negative = N < 0                          # 记录是否为负
    N = abs(N)                                # 取绝对值

    result = []
    while N > 0:
        result.append(digits[N % R])          # 取余数对应的字符
        N //= R                               # 整除

    if negative:
        result.append('-')                    # 加负号

    result.reverse()                          # 反转得到正确顺序
    print(''.join(result))
```

### ✅ 解法二：递归写法（教学用）

```python
import sys

digits = "0123456789ABCDEF"

def convert(n, r):
    if n == 0:
        return ""                             # 递归终止
    return convert(n // r, r) + digits[n % r] # 高位先递归，低位后拼接

for line in sys.stdin:
    N, R = map(int, line.split())
    if N == 0:
        print(0)
        continue

    sign = '-' if N < 0 else ''
    print(sign + convert(abs(N), R))
```

### ✅ 解法三：利用 Python 内置函数（部分进制）

```python
import sys

# Python 内置只支持 2/8/16 进制
# 对于任意进制还是需要手动实现
# 但可以作为验证工具

for line in sys.stdin:
    N, R = map(int, line.split())
    if N == 0:
        print(0)
        continue

    sign = '-' if N < 0 else ''
    N = abs(N)
    result = []
    digits = "0123456789ABCDEF"
    while N:
        N, rem = divmod(N, R)                 # divmod 同时返回商和余
        result.append(digits[rem])
    print(sign + ''.join(reversed(result)))
```

---

### 📊 复杂度分析

| 方法   | 时间复杂度  | 空间复杂度        |
| ------ | ----------- | ----------------- |
| 短除法 | O(log_R(N)) | O(log_R(N))       |
| 递归   | O(log_R(N)) | O(log_R(N))（栈） |

### ⚠️ 易错提示

1. **N 可以为负数**！必须先取绝对值再转换，最后加负号。
2. **N = 0 要特殊处理**，否则 while 循环不会执行，输出空串。
3. **余数的顺序是反的**，别忘了 `reverse()`。
4. 10 以上用大写 `A`~`F`。

---

## 17. HDU 2032 —— 杨辉三角

### 📝 题目描述

**输入**：每行一个正整数 `n`（`n ≤ 30`），直到 EOF。  
**输出**：`n` 层杨辉三角，数之间用空格分隔。每组输出后**空一行**。

```
1
1 1
1 2 1
1 3 3 1
1 4 6 4 1
```

---

### 💡 解题思路

- 第 `i` 行有 `i` 个数（从 1 开始）。
- 每行首尾为 `1`。
- 中间元素 `tri[i][j] = tri[i-1][j-1] + tri[i-1][j]`。

---

### ✅ 解法一：二维列表逐行生成

```python
import sys

for line in sys.stdin:
    n = int(line.strip())
    tri = []
    for i in range(n):
        row = [1] * (i + 1)                   # 先全部填 1
        for j in range(1, i):                 # 中间部分由上一行推导
            row[j] = tri[i - 1][j - 1] + tri[i - 1][j]
        tri.append(row)
        print(' '.join(map(str, row)))         # 逐行输出
    print()                                    # 每组后空一行
```

### ✅ 解法二：只保留上一行（滚动数组，省空间）

```python
import sys

for line in sys.stdin:
    n = int(line.strip())
    prev = []
    for i in range(n):
        curr = [1] * (i + 1)
        for j in range(1, i):
            curr[j] = prev[j - 1] + prev[j]
        print(' '.join(map(str, curr)))
        prev = curr                            # 滚动：当前行变上一行
    print()
```

### ✅ 解法三：利用组合数公式 C(n,k)

```python
import sys
from math import comb                         # Python 3.8+

for line in sys.stdin:
    n = int(line.strip())
    for i in range(n):
        # 第 i 行（0-indexed）的第 j 个数 = C(i, j)
        row = [comb(i, j) for j in range(i + 1)]
        print(' '.join(map(str, row)))
    print()
```

> `comb(i, j)` 直接计算组合数 C(i,j)，无需递推。

---

### 📊 复杂度分析

| 方法       | 时间复杂度              | 空间复杂度 |
| ---------- | ----------------------- | ---------- |
| 二维列表   | O(n²)                   | O(n²)      |
| 滚动数组   | O(n²)                   | O(n)       |
| 组合数公式 | O(n²)（每次 comb O(k)） | O(n)       |

### ⚠️ 易错提示

1. **每组输出后空一行**，注意别漏 `print()`。
2. 中间元素的递推循环是 `range(1, i)`，当 `i=0` 或 `i=1` 时不会执行，正好首尾为 1。
3. `n ≤ 30` 时数值最大约 C(29,14) ≈ 1.7亿，Python 没有溢出问题。

---

## 18. HDU 2033 —— 人见人爱A+B

### 📝 题目描述

**输入**：第一行 `T` 组测试。每组一行 6 个整数 `h1 m1 s1 h2 m2 s2`，表示两个时间（时 分 秒）。  
**输出**：两个时间相加后的结果，格式 `H M S`。

**样例输入**：
```
2
1 2 3 4 5 6
34 45 56 12 23 34
```
**样例输出**：
```
5 7 9
47 9 30
```

---

### 💡 解题思路

将两组时间的秒、分、时分别相加，处理**进位**：
- 秒 ≥ 60 → 进 1 到分
- 分 ≥ 60 → 进 1 到时

---

### ✅ 解法一：模拟进位

```python
T = int(input())
for _ in range(T):
    h1, m1, s1, h2, m2, s2 = map(int, input().split())

    s = s1 + s2                                # 秒相加
    m = m1 + m2                                # 分相加
    h = h1 + h2                                # 时相加

    # 处理秒的进位
    m += s // 60                               # 秒超过60的部分进到分
    s %= 60                                    # 秒保留余数

    # 处理分的进位
    h += m // 60                               # 分超过60的部分进到时
    m %= 60                                    # 分保留余数

    print(h, m, s)
```

### ✅ 解法二：全部转为秒再换算

```python
T = int(input())
for _ in range(T):
    h1, m1, s1, h2, m2, s2 = map(int, input().split())

    total_sec = (h1 * 3600 + m1 * 60 + s1) + (h2 * 3600 + m2 * 60 + s2)

    h = total_sec // 3600
    m = (total_sec % 3600) // 60
    s = total_sec % 60

    print(h, m, s)
```

### ✅ 解法三：使用 `divmod` 链式进位

```python
T = int(input())
for _ in range(T):
    h1, m1, s1, h2, m2, s2 = map(int, input().split())

    carry, s = divmod(s1 + s2, 60)            # divmod 返回 (商, 余数)
    carry2, m = divmod(m1 + m2 + carry, 60)
    h = h1 + h2 + carry2

    print(h, m, s)
```

---

### 📊 复杂度分析

所有方法均为 **O(1)**（每组数据常数时间）。

### ⚠️ 易错提示

1. **不需要对 24 取模**！题目没有说结果不超过 24 小时，直接输出总小时数。
2. 进位顺序：**先处理秒，再处理分**。不能反过来。
3. 输出格式是 `H M S`，用空格分隔。`print(h, m, s)` 默认空格分隔。

---

## 19. HDU 2034 —— 人见人爱A-B

### 📝 题目描述

**输入**：每行两个整数 `n`、`m`，表示集合 A 有 `n` 个元素、集合 B 有 `m` 个元素。之后一行 `n` 个整数为 A，再一行 `m` 个整数为 B。  
**处理**：求 `A - B`（差集：属于 A 但不属于 B 的元素）。  
**输出**：将差集**从小到大排序**输出。若差集为空，输出 `NULL`。  
**结束条件**：`n == 0` 且 `m == 0`。

**样例输入**：
```
3 3
1 2 3
1 4 7
0 0
```
**样例输出**：
```
2 3
```

---

### 💡 解题思路

- **差集 A - B**：A 中去掉所有出现在 B 中的元素。
- Python 有原生的 `set` 支持差集运算。

---

### ✅ 解法一：使用 `set` 差集运算（最 Pythonic）

```python
import sys

for line in sys.stdin:
    n, m = map(int, line.split())
    if n == 0 and m == 0:
        break

    A = set(map(int, input().split()))         # 读入集合 A
    B = set(map(int, input().split()))         # 读入集合 B

    diff = sorted(A - B)                       # A - B 差集，排序输出

    if diff:
        print(' '.join(map(str, diff)))
    else:
        print("NULL")
```

> `A - B` 是 Python set 的**差集运算符**，等价于 `A.difference(B)`。

### ✅ 解法二：手动遍历过滤

```python
import sys

for line in sys.stdin:
    n, m = map(int, line.split())
    if n == 0 and m == 0:
        break

    A = list(map(int, input().split()))
    B = set(map(int, input().split()))         # B 转 set 加速查找

    # 从 A 中筛选出不在 B 中的元素
    diff = sorted(x for x in A if x not in B)

    if diff:
        print(' '.join(map(str, diff)))
    else:
        print("NULL")
```

> 将 B 转为 `set`，`in` 操作从 O(m) 降到 O(1)。

### ✅ 解法三：列表推导 + 排序（纯暴力）

```python
import sys

for line in sys.stdin:
    n, m = map(int, line.split())
    if n == 0 and m == 0:
        break

    A = list(map(int, input().split()))
    B = list(map(int, input().split()))        # B 保持列表

    diff = sorted([x for x in A if x not in B])  # 每次 in 都是 O(m)

    print(' '.join(map(str, diff)) if diff else "NULL")
```

---

### 📊 复杂度分析

| 方法               | 时间复杂度                     | 空间复杂度 |
| ------------------ | ------------------------------ | ---------- |
| set 差集           | O(n + m + k log k)，k=差集大小 | O(n + m)   |
| 手动过滤（B为set） | O(n + m + k log k)             | O(n + m)   |
| 暴力（B为list）    | O(n × m + k log k)             | O(n + m)   |

### ⚠️ 易错提示

1. **差集为空输出 `NULL`**，不是 `EMPTY` 或空行。
2. **结果要排序**，从小到大。
3. A 中可能有重复元素（如果用 `set(A)` 会自动去重）。根据题意，一般 A 中元素不重复。如果可能重复，需注意是否保留重复。
4. 注意本题每组数据**先读 A 再读 B**，各在单独一行。

---

## 20. HDU 2035 —— 人见人爱A^B

### 📝 题目描述

**输入**：每行两个正整数 `A` 和 `B`。  
**处理**：求 `A^B`（A 的 B 次方）的**最后三位数**。  
**结束条件**：`A == 0` 且 `B == 0`。

**样例输入**：
```
2 10
0 0
```
**样例输出**：
```
24
```

> 解释：2^10 = 1024，最后三位是 `024`，但输出为整数所以是 `24`。

---

### 💡 解题思路

"最后三位数" = 对 1000 取模。  
直接算 `A^B` 可能非常大，但 Python 支持大整数，可以先算再取模。  
更好的做法是使用**快速幂取模**或 Python 内置的 `pow(a, b, mod)`。

---

### ✅ 解法一：Python 内置 `pow(a, b, mod)`（最推荐）

```python
import sys

for line in sys.stdin:
    A, B = map(int, line.split())
    if A == 0 and B == 0:
        break
    # pow(A, B, 1000) 内部使用快速幂取模，效率极高
    print(pow(A, B, 1000))
```

> **`pow(a, b, mod)`** 是 Python 内置的三参数幂运算，**自动执行快速幂取模**，时间复杂度 O(log B)。这是 Python 的一个强大特性。

### ✅ 解法二：先算大数再取模（暴力，Python 可行）

```python
import sys

for line in sys.stdin:
    A, B = map(int, line.split())
    if A == 0 and B == 0:
        break
    print((A ** B) % 1000)                     # 先算 A^B 再取模
```

> Python 的大整数不会溢出，但 `A^B` 可能极大（如 `999^999999`），计算非常慢。**竞赛中不推荐**。

### ✅ 解法三：手写快速幂取模（教学理解用）

```python
import sys

def fast_pow_mod(base, exp, mod):
    """快速幂取模：计算 base^exp % mod"""
    result = 1                                 # 初始结果
    base %= mod                                # 先对 base 取模

    while exp > 0:
        if exp % 2 == 1:                       # 如果当前位是 1
            result = (result * base) % mod     # 乘入结果
        exp //= 2                              # 指数右移一位
        base = (base * base) % mod            # 底数平方

    return result

for line in sys.stdin:
    A, B = map(int, line.split())
    if A == 0 and B == 0:
        break
    print(fast_pow_mod(A, B, 1000))
```

**快速幂原理**：
- 将指数 B 看作二进制，例如 B=10 → 二进制 `1010`
- `A^10 = A^8 × A^2`（对应二进制中为 1 的位）
- 每一步将底数平方（`A → A² → A⁴ → A⁸`），遇到指数为奇数时乘入结果
- 全程取模，防止数值爆炸

---

### 📊 复杂度分析

| 方法              | 时间复杂度       | 空间复杂度  |
| ----------------- | ---------------- | ----------- |
| `pow(A, B, 1000)` | O(log B)         | O(1)        |
| `A ** B % 1000`   | O(B)（大数乘法） | O(B 的位数) |
| 手写快速幂        | O(log B)         | O(1)        |

### ⚠️ 易错提示

1. **输出是整数**，不是补零的三位字符串。`024` 输出为 `24`，`000` 输出为 `0`。
2. **`pow(A, B, 1000)` 通常最推荐**，一行即可完成快速幂取模。
3. 暴力算 `A ** B` 在 B 很大时会**超时**，务必用快速幂或 `pow` 三参数形式。
4. 注意 `A % 1000` 可能为 0，但 `0` 的 B 次方仍是 0，合法。

---

## 📚 全卷总结

| 题号 | 核心考点          | 推荐 Python 技巧              |
| ---- | ----------------- | ----------------------------- |
| 2016 | 查找最小值 + 交换 | `min()` + `index()`           |
| 2017 | 字符统计          | `str.isdigit()` / `sum()`     |
| 2018 | 递推 / DP         | `f(n) = f(n-1) + f(n-3)`      |
| 2019 | 有序插入          | `bisect.insort()`             |
| 2020 | 自定义排序        | `sort(key=abs, reverse=True)` |
| 2021 | 贪心              | `divmod` + 面额列表           |
| 2022 | 矩阵遍历          | `max()` with key              |
| 2023 | 矩阵行/列统计     | 列表推导 + `all()`            |
| 2024 | 字符串合法性      | ASCII 逐字符判断 / 正则       |
| 2025 | 字符串替换        | `str.replace()`               |
| 2026 | 首字母大写        | 状态机扫描 / `split(' ')`     |
| 2027 | 元音统计          | `str.count()` / `Counter`     |
| 2028 | 最小公倍数        | `math.gcd` → `lcm`            |
| 2029 | 回文判断          | `s == s[::-1]`                |
| 2030 | 汉字统计          | GBK 双字节统计 / Unicode 范围 |
| 2031 | 进制转换          | 短除法 + `divmod`             |
| 2032 | 杨辉三角          | 递推 / `math.comb`            |
| 2033 | 时间加法          | `divmod` 进位                 |
| 2034 | 集合差集          | `set` 差集 `A - B`            |
| 2035 | 快速幂取模        | `pow(A, B, 1000)`             |

> **刷题核心建议**：先写暴力确保正确，再优化。Python 的内置函数（`min`, `max`, `sorted`, `bisect`, `pow`, `set` 运算）是竞赛利器，熟练掌握可以大幅减少代码量和出错概率。祝刷题顺利！🎯
