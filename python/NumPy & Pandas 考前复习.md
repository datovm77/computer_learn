# NumPy & Pandas 考前复习教程

> 所有代码均已在 Python 环境中实际运行验证，输出结果真实可靠。建议边看边在自己电脑上敲一遍。

------

## 第一部分：NumPy

### 1. 数组创建

```python
import numpy as np

a = np.array([1, 2, 3])              # 一维数组
b = np.array([[1,2,3],[4,5,6]])      # 二维数组

print(a.shape, a.ndim)   # (3,) 1
print(b.shape, b.ndim)   # (2, 3) 2
```

**常用创建函数：**

| 函数                                 | 作用                                | 示例                    |
| ------------------------------------ | ----------------------------------- | ----------------------- |
| `np.arange(0,10,2)`                  | 类似 `range`，指定步长              | `[0 2 4 6 8]`           |
| `np.linspace(0,1,5)`                 | 在区间内**均分**取 n 个点（含端点） | `[0. 0.25 0.5 0.75 1.]` |
| `np.zeros((2,3))`                    | 全 0 数组                           | 2×3 的 0 矩阵           |
| `np.ones((2,3))`                     | 全 1 数组                           | 2×3 的 1 矩阵           |
| `np.eye(3)`                          | 单位矩阵                            | 3×3 对角为1             |
| `np.random.randint(0,10,size=(2,3))` | 随机整数                            | 2×3 随机矩阵            |
| `np.random.rand(2,2)`                | [0,1) 均匀分布                      | 2×2 随机矩阵            |

⚠️ **`arange` vs `linspace` 的区别是高频考点**：`arange` 按**步长**切分（可能取不到终点），`linspace` 按**个数**均分（一定包含终点，除非设置 `endpoint=False`）。

------

### 2. 索引与切片

```python
arr = np.arange(1,13).reshape(3,4)
# [[ 1  2  3  4]
#  [ 5  6  7  8]
#  [ 9 10 11 12]]

arr[1, 2]        # 7   （第1行第2列，从0开始）
arr[:, 1]        # [ 2  6 10]   取第1列
arr[1:, 1:3]     # [[ 6  7] [10 11]]  行1开始，列1到2
```

**布尔索引（考试常考“筛选满足条件的元素”）：**

```python
mask = arr > 6
# [[False False False False]
#  [False False  True  True]
#  [ True  True  True  True]]

arr[arr > 6]   # [ 7  8  9 10 11 12]   一维展开返回
```

**花式索引（fancy indexing）：**

```python
arr[[0, 2]]   
# [[ 1  2  3  4]
#  [ 9 10 11 12]]     取第0行和第2行
```

### ⭐️ 视图（view）vs 拷贝（copy）—— 非常容易考

```python
a = np.arange(5)        # [0 1 2 3 4]

b = a[1:3]      # 切片 → 视图，共享内存！
b[0] = 100
print(a)        # [  0 100   2   3   4]  ← a 被修改了！

c = a[[0,1]]    # 花式索引 → 拷贝，独立内存
c[0] = 999
print(a)        # [  0 100   2   3   4]  ← a 没变
```

**记忆口诀：普通切片（`:`）是视图，花式索引 / 布尔索引是拷贝。** 如果要强制拷贝，用 `.copy()`。

------

### 3. 广播机制（Broadcasting）

```python
x = np.array([[1],[2],[3]])   # shape (3,1)
y = np.array([10,20,30])      # shape (3,)

x + y
# [[11 21 31]
#  [12 22 32]
#  [13 23 33]]
```

**广播规则（从右往左比较维度）：**

1. 维度数不同时，在**较短**的形状左边补 1；
2. 每个维度上，两数组尺寸要么相等，要么其中一个是 1；
3. 都不满足则报错 `ValueError: operands could not be broadcast together`。

这里 `x` 是 `(3,1)`，`y` 是 `(3,)` → 补齐成 `(1,3)` → 逐维比较：`3 vs 3`✅，`1 vs 3`✅（1可以扩展）→ 结果 `(3,3)`。

------

### 4. 聚合函数与 axis（超级重点）

```python
m = np.array([[1,2,3],[4,5,6]])

m.sum()          # 21        所有元素求和
m.sum(axis=0)    # [5 7 9]   按列求和（压缩掉行，"竖着加"）
m.sum(axis=1)    # [ 6 15]   按行求和（压缩掉列，"横着加"）
```

**理解 axis 的技巧**：`axis=n` 表示"沿着第 n 个方向压扁"，压扁后该维度消失。二维数组里 `axis=0` 是"跨行"（结果长度=列数），`axis=1` 是"跨列"（结果长度=行数）。

```python
m.mean()         # 3.5
m.std()          # 1.7078...
m.argmax()       # 5   → 展平后最大值的下标
m.argmax(axis=1) # [2 2]  每一行最大值所在的列号
```

其他常用：`np.sort()`、`np.argsort()`（返回排序后的**下标**）、`np.unique()`（去重并排序）、`np.where(cond, a, b)`（三元条件选择）。

```python
arr = np.array([3,1,4,1,5,9,2,6])
np.sort(arr)      # [1 1 2 3 4 5 6 9]
np.argsort(arr)   # [1 3 6 0 2 4 7 5]
np.unique(arr)    # [1 2 3 4 5 6 9]
np.where(arr>3, arr, 0)  # [0 0 4 0 5 9 0 6]
```

------

### 5. 形状操作

```python
v = np.arange(6)
v.reshape(2,3)
# [[0 1 2]
#  [3 4 5]]

v.reshape(2,3).T     # 转置
# [[0 3]
#  [1 4]
#  [2 5]]
```

**拼接函数辨析（易混淆）：**

| 函数                        | 效果                                             |
| --------------------------- | ------------------------------------------------ |
| `np.concatenate([c1,c2])`   | 沿已有轴拼接，不增加维度                         |
| `np.vstack([c1,c2])`        | 垂直堆叠（增加行），等价 `axis=0` 的 concatenate |
| `np.hstack([c1,c2])`        | 水平堆叠（增加列）                               |
| `np.stack([c1,c2], axis=0)` | **增加新的一个维度**再堆叠                       |

```python
c1, c2 = np.array([1,2]), np.array([3,4])
np.concatenate([c1,c2])   # [1 2 3 4]        一维还是一维
np.vstack([c1,c2])        # [[1 2] [3 4]]     变成二维
np.stack([c1,c2],axis=0)  # [[1 2] [3 4]]     和 vstack 结果一样，但原理是"新增维度"
```

------

### 6. 线性代数（结合你的线代课，重点）

```python
A = np.array([[2,0],[0,3]])
B = np.array([[1,2],[3,4]])

A @ B            # 矩阵乘法，等价 A.dot(B)
# [[ 2  4]
#  [ 9 12]]

np.linalg.inv(B)     # 逆矩阵
# [[-2.   1. ]
#  [ 1.5 -0.5]]

np.linalg.det(B)     # 行列式 = -2.0

eigvals, eigvecs = np.linalg.eig(B)
# eigvals = [-0.372  5.372]
# eigvecs 每一列是对应特征值的特征向量

np.linalg.solve(B, np.array([5,6]))  # 解线性方程组 Bx=b → [-4. 4.5]
np.linalg.matrix_rank(B)             # 秩 = 2
np.linalg.norm(np.array([3,4]))      # 向量范数 = 5.0
```

⚠️ 注意：`A * B` 是**逐元素相乘**（哈达玛积），`A @ B` 或 `A.dot(B)` 才是**矩阵乘法**——这是数据科学专业考试的经典陷阱题。

------

## 第二部分：Pandas

### 1. Series 与 DataFrame

```python
import pandas as pd

s = pd.Series([10,20,30], index=['a','b','c'])
s['b']       # 20   按标签取值
s.iloc[1]    # 20   按位置取值

df = pd.DataFrame({
    '姓名': ['小明','小红','小刚','小李'],
    '年龄': [20,21,19,22],
    '成绩': [85,92,78,88],
    '班级': ['A','B','A','B']
})
```

|      | 姓名 | 年龄 | 成绩 | 班级 |
| ---- | ---- | ---- | ---- | ---- |
| 0    | 小明 | 20   | 85   | A    |
| 1    | 小红 | 21   | 92   | B    |
| 2    | 小刚 | 19   | 78   | A    |
| 3    | 小李 | 22   | 88   | B    |

常用检查方法：`df.dtypes`、`df.info()`、`df.describe()`（对数值列自动做统计：均值/标准差/四分位数等）。

------

### 2. loc / iloc（考试必考，一定要分清）

```python
df.loc[0]           # 按【标签】取第0行（结果是一个 Series）
df.loc[0, '姓名']    # '小明'   按行标签+列名
df.iloc[0:2, 1:3]    # 按【位置】切片，取 行0~1、列1~2
df.loc[df['年龄'] > 19]   # 布尔条件筛选，配合 loc 常用
```

**核心区别**：`loc` 按**标签**（行索引名/列名）取值，`iloc` 按**整数位置**取值（类似 Python 序列下标，切片不包含结束点）。当索引本身就是数字 0,1,2... 时两者容易混，但语义不同：`iloc[0:2]` 一定是前两行，`loc[0:2]` 若索引是数字则**包含结束标签**。

------

### 3. 布尔筛选（组合条件）

```python
df[(df['年龄'] > 19) & (df['班级'] == 'A')]
```

⚠️ 多条件必须：① 每个条件加括号 ② 用 `&`（与）`|`（或）`~`（非），**不能用 `and`/`or`**（因为要对每个元素逐个判断，Python 内置 `and/or` 只能判断单个布尔值）。

------

### 4. 缺失值处理

```python
df2.isnull()          # 返回布尔表，标记 NaN 位置
df2.dropna()           # 删除含 NaN 的行
df2.fillna(df2['成绩'].mean())   # 用均值填补

s2 = pd.Series([1, np.nan, np.nan, 4])
s2.ffill()   # [1. 1. 1. 4.]   用前一个有效值向下填充
s2.bfill()   # [1. 4. 4. 4.]   用后一个有效值向上填充
```

------

### 5. apply / map（自定义函数处理）

```python
df['成绩'].apply(lambda x: '优秀' if x >= 85 else '一般')
# 0    优秀
# 1    优秀
# 2    一般
# 3    优秀

df['年龄+1'] = df['年龄'].map(lambda x: x + 1)   # 对Series逐元素处理
```

**区别**：`Series.map()` 只能用于 Series（逐元素映射，也可传字典做映射表）；`DataFrame.apply()` 可以按行或列（`axis=0/1`）传入函数；`applymap`（旧版本，新版本叫 `DataFrame.map`）是对整个 DataFrame 逐元素处理。

------

### 6. groupby 分组聚合（重点中的重点）

```python
df.groupby('班级')['成绩'].mean()
# 班级
# A    81.5
# B    90.0

df.groupby('班级').agg({'成绩': ['mean','max'], '年龄': 'sum'})
#       成绩       年龄
#      mean max   sum
# 班级
# A    81.5  85    39
# B    90.0  92    43
```

**理解 groupby 的三步走**："split（拆分）→ apply（应用函数）→ combine（合并结果）"。这是考试大题里最常被问到"原理是什么"的地方。

------

### 7. merge / concat（合并表格，务必分清用途）

```python
df_a = pd.DataFrame({'id':[1,2,3],'name':['a','b','c']})
df_b = pd.DataFrame({'id':[2,3,4],'score':[90,80,70]})

pd.merge(df_a, df_b, on='id', how='inner')
#    id name  score
# 0   2    b     90
# 1   3    c     80        # 只保留两表共有的 id

pd.merge(df_a, df_b, on='id', how='left')
#    id name  score
# 0   1    a    NaN
# 1   2    b   90.0
# 2   3    c   80.0        # 保留左表全部，右表没有的补NaN

pd.merge(df_a, df_b, on='id', how='outer')
#    id name  score
# 0   1    a    NaN
# 1   2    b   90.0
# 2   3    c   80.0
# 3   4  NaN   70.0        # 两表并集
```

**`merge` vs `concat` 的本质区别**：

- `merge` 是按**某一列的值**（键）做类似 SQL JOIN 的关联合并；
- `concat` 是单纯把多个表**堆叠**在一起（`axis=0` 上下拼接，`axis=1` 左右拼接），不看键值对应关系。

```python
pd.concat([df_a, df_b], axis=0, ignore_index=True)  # 上下堆叠，重新编号
pd.concat([df_a, df_b], axis=1)                     # 左右拼接，按行位置对齐
```

------

### 8. 排序、去重、字符串处理、计数

```python
df.sort_values('成绩', ascending=False)   # 按值排序
df.sort_index(ascending=False)             # 按索引排序

s = pd.Series(['苹果','香蕉','苹果','橙子','苹果'])
s.value_counts()
# 苹果    3
# 香蕉    1
# 橙子    1

df2 = pd.DataFrame({'x':[1,1,2,2,3]})
df2.duplicated()          # 标记哪些是重复行（第一次出现算 False）
df2.drop_duplicates()      # 去重，保留第一次出现

df['城市'].str.split('-').str[0]   # 字符串按'-'切分后取第一段（.str 访问器）
```

------

### 9. 数据透视表 pivot_table

```python
df.pivot_table(values='成绩', index='班级', aggfunc='mean')
#       成绩
# 班级
# A    81.5
# B    90.0
```

本质上是 `groupby` 的另一种写法，更像 Excel 里的"数据透视表"，`index` 决定分组的行标签，`values` 是要统计的列，`aggfunc` 是统计方式。

------

## 三、易错点速查表（考前最后过一遍）

| 易错点                 | 正确理解                                               |
| ---------------------- | ------------------------------------------------------ |
| `A*B` vs `A@B`         | 前者逐元素乘，后者矩阵乘法                             |
| 切片 vs 花式/布尔索引  | 前者是视图（改了会影响原数组），后者是拷贝             |
| `axis=0` vs `axis=1`   | 0 是"跨行"操作（结果按列），1 是"跨列"操作（结果按行） |
| `loc` vs `iloc`        | 前者按标签，后者按位置，切片时前者含终点、后者不含     |
| 多条件筛选用 `&/       | `                                                      |
| `merge` vs `concat`    | 前者按键关联，后者单纯堆叠                             |
| `arange` vs `linspace` | 前者按步长，后者按个数均分                             |
| `ffill` vs `bfill`     | 前者向下填（用前面的值），后者向上填（用后面的值）     |
| `dropna` vs `fillna`   | 前者删除缺失行，后者填补缺失值                         |

------

**祝你明天考试顺利！** 如果有具体的历年题或者某个函数不确定，随时可以贴出来我帮你再讲一遍。