# Pandas 完全入门教程

---

## 前言

本教程面向零基础学习者，假设你已经安装好 Python 和 Pandas。如果没有，请先执行：

```bash
pip install pandas openpyxl
```

在所有代码示例中，我们统一使用以下导入：

```python
import pandas as pd
import numpy as np
```

---

## 第 1 章：基础数据结构

Pandas 的核心只有两个数据结构：**Series**（一维）和 **DataFrame**（二维）。理解它们是学习 Pandas 的第一步，也是最重要的一步。

---

### 1.1 Series —— 一维数据

#### 1.1.1 什么是 Series

Series 就是**一列数据**，你可以把它想象成 Excel 中的**一列**。它由两部分组成：

- **索引（index）**：每行数据的"标签"，默认是 0, 1, 2, 3...
- **值（values）**：实际存储的数据

```
索引    值
 0  →  10
 1  →  20
 2  →  30
```

#### 1.1.2 创建 Series

**方法一：从列表创建**

```python
s = pd.Series([10, 20, 30])
print(s)
```

输出：

```
0    10
1    20
2    30
dtype: int64
```

左边一列是**索引**（自动生成 0, 1, 2），右边一列是**值**。`dtype: int64` 表示数据类型是 64 位整数。

**方法二：指定索引**

```python
s = pd.Series([10, 20, 30], index=['a', 'b', 'c'])
print(s)
```

输出：

```
a    10
b    20
c    30
dtype: int64
```

现在索引不再是数字，而是我们指定的 `'a'`, `'b'`, `'c'`。

**方法三：从字典创建**

```python
d = {'苹果': 5.5, '香蕉': 3.0, '橘子': 4.2}
s = pd.Series(d)
print(s)
```

输出：

```
苹果    5.5
香蕉    3.0
橘子    4.2
dtype: float64
```

字典的**键**自动变成了**索引**，字典的**值**变成了 Series 的**值**。

**方法四：用标量（单个值）创建**

```python
s = pd.Series(100, index=['x', 'y', 'z'])
print(s)
```

输出：

```
x    100
y    100
z    100
dtype: int64
```

所有位置都填充同一个值。

#### 1.1.3 访问 Series 中的数据

```python
s = pd.Series([10, 20, 30, 40], index=['a', 'b', 'c', 'd'])

# 通过索引标签访问
print(s['a'])        # 输出: 10

# 通过位置访问（和列表一样，从 0 开始）
print(s.iloc[0])     # 输出: 10

# 访问多个元素
print(s[['a', 'c']]) # 输出 a=10, c=30

# 切片
print(s['a':'c'])    # 输出 a=10, b=20, c=30（注意：标签切片包含末尾！）
print(s[0:2])        # 输出 a=10, b=20（位置切片不包含末尾，和 Python 一致）
```

> **重要区别**：用标签切片 `'a':'c'` 是**包含末尾**的；用位置切片 `0:2` 是**不包含末尾**的。这是初学者常犯的错误。

#### 1.1.4 Series 的常用属性和方法

```python
s = pd.Series([10, 20, 30, 40], index=['a', 'b', 'c', 'd'], name='成绩')

print(s.values)     # 输出: [10 20 30 40]        → 返回 NumPy 数组
print(s.index)      # 输出: Index(['a','b','c','d']) → 返回索引对象
print(s.dtype)      # 输出: int64                 → 数据类型
print(s.shape)      # 输出: (4,)                  → 形状（4个元素）
print(s.size)       # 输出: 4                     → 元素个数
print(s.name)       # 输出: 成绩                  → Series 的名字
```

#### 1.1.5 Series 的运算

Series 支持**向量化运算**，也就是说，运算会自动作用到每一个元素上：

```python
s = pd.Series([10, 20, 30])

print(s + 5)    # 每个元素 +5  → [15, 25, 35]
print(s * 2)    # 每个元素 ×2  → [20, 40, 60]
print(s > 15)   # 每个元素和 15 比较 → [False, True, True]
```

两个 Series 之间也可以运算，**按索引对齐**：

```python
s1 = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s2 = pd.Series([10, 20, 30], index=['a', 'b', 'd'])

print(s1 + s2)
```

输出：

```
a    11.0
b    22.0
c     NaN
d     NaN
dtype: float64
```

`'c'` 只在 s1 中有，`'d'` 只在 s2 中有，对不上的位置结果就是 `NaN`（Not a Number，表示缺失值）。

---

### 1.2 DataFrame —— 二维表

#### 1.2.1 什么是 DataFrame

DataFrame 就是一张**二维表格**，和 Excel 的工作表几乎一样。它由以下部分组成：

```
          姓名    年龄    城市
  0       张三     25     北京
  1       李四     30     上海
  2       王五     28     广州
  ↑                              ↑
 行索引(index)              列名(columns)
```

- **行索引（index）**：标识每一行
- **列名（columns）**：标识每一列
- 每一**列**就是一个 Series

#### 1.2.2 创建 DataFrame

**方法一：从字典创建（最常用）**

字典的**键**变成**列名**，字典的**值**（列表）变成**每列的数据**：

```python
data = {
    '姓名': ['张三', '李四', '王五'],
    '年龄': [25, 30, 28],
    '城市': ['北京', '上海', '广州']
}
df = pd.DataFrame(data)
print(df)
```

输出：

```
   姓名  年龄  城市
0  张三   25  北京
1  李四   30  上海
2  王五   28  广州
```

**方法二：从列表的列表创建**

```python
data = [
    ['张三', 25, '北京'],
    ['李四', 30, '上海'],
    ['王五', 28, '广州']
]
df = pd.DataFrame(data, columns=['姓名', '年龄', '城市'])
print(df)
```

输出和上面一样。注意这种方式**必须手动指定 `columns`**，否则列名会变成 0, 1, 2。

**方法三：从字典的列表创建**

每个字典代表一行：

```python
data = [
    {'姓名': '张三', '年龄': 25, '城市': '北京'},
    {'姓名': '李四', '年龄': 30, '城市': '上海'},
    {'姓名': '王五', '年龄': 28, '城市': '广州'}
]
df = pd.DataFrame(data)
print(df)
```

**方法四：指定行索引**

```python
df = pd.DataFrame(
    {'姓名': ['张三', '李四', '王五'],
     '年龄': [25, 30, 28]},
    index=['row1', 'row2', 'row3']
)
print(df)
```

输出：

```
      姓名  年龄
row1  张三   25
row2  李四   30
row3  王五   28
```

#### 1.2.3 DataFrame 的基本属性

这些属性你需要**烂熟于心**，因为拿到任何数据第一步就是用它们来了解数据的"长相"：

```python
data = {
    '姓名': ['张三', '李四', '王五', '赵六'],
    '年龄': [25, 30, 28, 35],
    '城市': ['北京', '上海', '广州', '深圳'],
    '工资': [10000, 15000, 12000, 20000]
}
df = pd.DataFrame(data)

# ========== 基本属性 ==========
print(df.shape)      # (4, 4)  → 4行4列
print(df.columns)    # Index(['姓名', '年龄', '城市', '工资'], dtype='object')
print(df.index)      # RangeIndex(start=0, stop=4, step=1)
print(df.dtypes)     # 每列的数据类型
print(df.values)     # 返回二维 NumPy 数组
print(df.size)       # 16  → 总元素个数（4行×4列）
print(len(df))       # 4   → 行数
```

#### 1.2.4 快速了解数据的方法

```python
# 查看前 N 行（默认5行）
print(df.head())
print(df.head(2))    # 只看前2行

# 查看后 N 行
print(df.tail(2))

# 查看整体信息（非常有用！）
df.info()
```

`df.info()` 的输出：

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 4 entries, 0 to 3
Data columns (total 4 columns):
 #   Column  Non-Null Count  Dtype
---  ------  --------------  -----
 0   姓名      4 non-null      object
 1   年龄      4 non-null      int64
 2   城市      4 non-null      object
 3   工资      4 non-null      int64
dtypes: int64(2), object(2)
memory usage: 256.0+ bytes
```

一眼就能看出：有多少行、每列有多少非空值、每列是什么类型。

```python
# 数值列的统计摘要
print(df.describe())
```

输出：

```
             年龄           工资
count   4.000000      4.000000
mean   29.500000  14250.000000
std     4.203173   4349.329006
min    25.000000  10000.000000
25%    27.250000  11500.000000
50%    29.000000  13500.000000
75%    31.250000  16250.000000
max    35.000000  20000.000000
```

自动给出计数、均值、标准差、最小值、四分位数、最大值。

#### 1.2.5 DataFrame 和 Series 的关系

DataFrame 的**每一列**都是一个 Series：

```python
col = df['姓名']
print(type(col))    # <class 'pandas.core.series.Series'>
print(col)
```

输出：

```
0    张三
1    李四
2    王五
3    赵六
Name: 姓名, dtype: object
```

可以看到，这个 Series 的 `name` 属性就是列名 `'姓名'`。

---

## 第 2 章：数据读取与保存

实际工作中，数据几乎不会手动创建，而是从文件中读取。Pandas 支持几十种文件格式，最常用的是 CSV、Excel 和 JSON。

---

### 2.1 CSV 文件

#### 2.1.1 什么是 CSV

CSV（Comma-Separated Values）是最简单、最通用的数据格式，用**纯文本**存储表格数据，每行一条记录，字段之间用逗号分隔：

```
姓名,年龄,城市
张三,25,北京
李四,30,上海
```

#### 2.1.2 读取 CSV

```python
# 最基本的读取
df = pd.read_csv('data.csv')

# 常用参数
df = pd.read_csv(
    'data.csv',
    encoding='utf-8',       # 编码，中文文件可能需要 'gbk' 或 'utf-8-sig'
    sep=',',                # 分隔符，默认逗号。TSV 文件用 '\t'
    header=0,               # 第几行作为列名，默认第0行（第一行）
    index_col=None,         # 哪一列作为行索引，默认不指定
    usecols=['姓名', '年龄'], # 只读取指定列（提高效率）
    nrows=100,              # 只读取前100行（大文件预览时非常有用）
    na_values=['NA', '缺失'], # 把这些值当作缺失值处理
    dtype={'年龄': int}       # 强制指定列的数据类型
)
```

**常见问题排查**：

```python
# 问题1：中文乱码
df = pd.read_csv('data.csv', encoding='gbk')         # 试试 gbk
df = pd.read_csv('data.csv', encoding='utf-8-sig')    # 试试 utf-8-sig

# 问题2：分隔符不是逗号
df = pd.read_csv('data.tsv', sep='\t')                # Tab 分隔
df = pd.read_csv('data.txt', sep='|')                 # 竖线分隔

# 问题3：文件没有表头
df = pd.read_csv('data.csv', header=None)             # 第一行也是数据
df = pd.read_csv('data.csv', header=None, names=['A', 'B', 'C'])  # 手动指定列名

# 问题4：跳过前几行（比如文件前面有注释）
df = pd.read_csv('data.csv', skiprows=3)              # 跳过前3行
```

#### 2.1.3 保存 CSV

```python
# 最基本的保存
df.to_csv('output.csv')

# 常用参数
df.to_csv(
    'output.csv',
    index=False,            # 不保存行索引（通常都加这个！）
    encoding='utf-8-sig',   # 用 Excel 打开中文不乱码
    sep=',',                # 分隔符
    columns=['姓名', '年龄']  # 只保存指定列
)
```

> **重要提示**：`index=False` 非常常用。如果不加，保存的文件会多出一列索引数字（0, 1, 2...），再次读取时会出现多余的列。

---

### 2.2 Excel 文件

#### 2.2.1 读取 Excel

```python
# 读取 .xlsx 文件（需要 openpyxl 库）
df = pd.read_excel('data.xlsx')

# 常用参数
df = pd.read_excel(
    'data.xlsx',
    sheet_name='Sheet1',    # 指定工作表名称。也可以用数字: sheet_name=0
    header=0,               # 表头行
    usecols='A:C',          # 只读取 A 到 C 列（也可以用列表: usecols=[0, 1, 2]）
    skiprows=2,             # 跳过前2行
    nrows=100               # 只读取100行
)

# 一次读取所有工作表
all_sheets = pd.read_excel('data.xlsx', sheet_name=None)
# 返回一个字典：{'Sheet1': df1, 'Sheet2': df2, ...}
for name, sheet_df in all_sheets.items():
    print(f"工作表: {name}, 行数: {len(sheet_df)}")
```

#### 2.2.2 保存 Excel

```python
# 基本保存
df.to_excel('output.xlsx', index=False)

# 保存到指定工作表
df.to_excel('output.xlsx', sheet_name='数据', index=False)

# 保存多个工作表到同一个文件
with pd.ExcelWriter('output.xlsx') as writer:
    df1.to_excel(writer, sheet_name='销售数据', index=False)
    df2.to_excel(writer, sheet_name='用户数据', index=False)
```

---

### 2.3 JSON 文件

#### 2.3.1 读取 JSON

```python
# 基本读取
df = pd.read_json('data.json')

# 如果 JSON 结构比较特殊，可以指定方向
df = pd.read_json('data.json', orient='records')
# orient 参数常用值：
#   'records' → [{"col1": val1, "col2": val2}, ...]  （最常见）
#   'columns' → {"col1": {"row1": val1}, ...}
#   'index'   → {"row1": {"col1": val1}, ...}
```

#### 2.3.2 保存 JSON

```python
df.to_json('output.json', orient='records', force_ascii=False)
# force_ascii=False → 中文正常显示，不会变成 \uXXXX
```

---

### 2.4 其他格式速查

```python
# SQL 数据库
import sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM table_name', conn)
df.to_sql('table_name', conn, if_exists='replace', index=False)

# HTML 表格
tables = pd.read_html('https://example.com/page.html')  # 返回列表，每个表格一个 DataFrame

# 剪贴板（从 Excel 复制后直接读取！）
df = pd.read_clipboard()

# Parquet（大数据常用，速度快体积小）
df = pd.read_parquet('data.parquet')
df.to_parquet('output.parquet')
```

---

## 第 3 章：基础数据操作

这一章是日常使用最频繁的部分。我们使用以下示例数据：

```python
df = pd.DataFrame({
    '姓名': ['张三', '李四', '王五', '赵六', '钱七'],
    '年龄': [25, 30, 28, 35, 22],
    '城市': ['北京', '上海', '广州', '深圳', '北京'],
    '工资': [10000, 15000, 12000, 20000, 8000],
    '部门': ['技术', '市场', '技术', '管理', '市场']
})
```

```
   姓名  年龄  城市    工资  部门
0  张三   25  北京  10000  技术
1  李四   30  上海  15000  市场
2  王五   28  广州  12000  技术
3  赵六   35  深圳  20000  管理
4  钱七   22  北京   8000  市场
```

---

### 3.1 选取数据

#### 3.1.1 选取列

```python
# 选取单列 → 返回 Series
print(df['姓名'])
# 等价写法（列名是合法变量名且不与 DataFrame 方法冲突时）
print(df.姓名)         # 不推荐，因为列名如果含空格或特殊字符就不行

# 选取多列 → 返回 DataFrame（注意双层方括号）
print(df[['姓名', '工资']])
```

输出：

```
   姓名    工资
0  张三  10000
1  李四  15000
2  王五  12000
3  赵六  20000
4  钱七   8000
```

#### 3.1.2 选取行 —— `loc` 与 `iloc`

这是 Pandas 最核心的操作之一，**必须彻底搞清楚**。

|            | `loc`                            | `iloc`                             |
| ---------- | -------------------------------- | ---------------------------------- |
| 含义       | **l**abel **loc**ation（按标签） | **i**nteger **loc**ation（按位置） |
| 用什么访问 | 行/列的**名称**（标签）          | 行/列的**数字位置**（从0开始）     |
| 切片特点   | **包含**末尾                     | **不包含**末尾                     |

**`loc` —— 按标签选取**

```python
# 选取单行
print(df.loc[0])           # 选取索引标签为 0 的行（返回 Series）

# 选取多行
print(df.loc[[0, 2, 4]])   # 选取索引为 0, 2, 4 的行

# 选取行和列
print(df.loc[0, '姓名'])           # 第0行的"姓名"列 → '张三'
print(df.loc[0:2, '姓名':'城市'])  # 第0~2行（含2），"姓名"到"城市"列（含"城市"）
```

`df.loc[0:2, '姓名':'城市']` 的输出：

```
   姓名  年龄  城市
0  张三   25  北京
1  李四   30  上海
2  王五   28  广州
```

注意第2行也被包含了！

**`iloc` —— 按位置选取**

```python
# 选取单行
print(df.iloc[0])          # 第0个位置的行

# 选取多行
print(df.iloc[[0, 2, 4]])  # 位置 0, 2, 4

# 选取行和列（全部用数字）
print(df.iloc[0, 0])            # 第0行第0列 → '张三'
print(df.iloc[0:2, 0:3])        # 第0~1行（不含2），第0~2列（不含3）
```

`df.iloc[0:2, 0:3]` 的输出：

```
   姓名  年龄  城市
0  张三   25  北京
1  李四   30  上海
```

注意第2行**没有**被包含！

**对比记忆**：

```python
# loc：用标签，切片包含末尾
df.loc[0:2]     # → 第0、1、2行（3行）

# iloc：用位置，切片不包含末尾
df.iloc[0:2]    # → 第0、1行（2行）
```

**实用组合技巧**：

```python
# loc：选指定行和指定列
df.loc[[0, 3], ['姓名', '工资']]

# iloc：选指定行位置和指定列位置
df.iloc[[0, 3], [0, 3]]

# 混合使用：先用 iloc 选行，再用列名
df.iloc[0:3][['姓名', '工资']]  # 也可以，但不如 loc/iloc 直接指定高效
```

---

### 3.2 增删改列

#### 3.2.1 新增列

```python
# 方法一：直接赋值（最常用）
df['奖金'] = [1000, 2000, 1500, 3000, 800]
print(df)

# 方法二：基于已有列计算
df['总收入'] = df['工资'] + df['奖金']

# 方法三：给所有行赋同一个值
df['国家'] = '中国'

# 方法四：用 insert 在指定位置插入列
df.insert(
    loc=2,              # 插入位置（第2列的位置，即第3列之前）
    column='性别',       # 列名
    value=['男', '男', '男', '男', '女']  # 值
)
```

#### 3.2.2 修改列

```python
# 修改整列
df['年龄'] = [26, 31, 29, 36, 23]    # 所有人年龄+1

# 修改单个值
df.loc[0, '城市'] = '天津'            # 把张三的城市改为天津

# 批量修改（满足条件的行）
df.loc[df['年龄'] > 30, '部门'] = '高管'  # 年龄>30的人，部门改为"高管"

# 修改列名
df = df.rename(columns={'工资': '月薪', '城市': '所在城市'})

# 一次性修改所有列名
df.columns = ['name', 'age', 'city', 'salary', 'dept']
```

> **关于 `inplace` 参数**：
> - `inplace=True`：在原 DataFrame 上直接修改，不返回新对象
> - `inplace=False`（默认）：返回一个新的 DataFrame，原 DataFrame 不变
> - 现代 Pandas 编程风格更推荐不用 `inplace`，而是用赋值：`df = df.rename(...)`

#### 3.2.3 删除列和行

```python
# 删除列
df = df.drop(columns=['奖金'])           # 删除"奖金"列
df = df.drop(columns=['奖金', '国家'])   # 删除多列

# 另一种写法
df = df.drop('奖金', axis=1)             # axis=1 表示列

# 删除行
df = df.drop(index=[0, 2])              # 删除索引为 0 和 2 的行
df = df.drop(index=df[df['年龄'] < 25].index)  # 删除年龄<25 的行

# 用 del 删除列（直接修改原DataFrame，无法撤销）
del df['国家']

# 用 pop 删除列（删除并返回该列）
bonus = df.pop('奖金')  # df 中不再有"奖金"列，bonus 是一个 Series
```

---

### 3.3 排序

#### 3.3.1 按值排序 `sort_values`

```python
# 按单列排序（默认升序）
df_sorted = df.sort_values('工资')
print(df_sorted)

# 降序
df_sorted = df.sort_values('工资', ascending=False)

# 按多列排序（先按部门升序，部门相同再按工资降序）
df_sorted = df.sort_values(['部门', '工资'], ascending=[True, False])

# 处理缺失值的排序位置
df_sorted = df.sort_values('工资', na_position='first')  # NaN 排在最前
# na_position='last' 是默认值，NaN 排在最后
```

#### 3.3.2 按索引排序 `sort_index`

```python
# 按行索引排序
df_sorted = df.sort_index()               # 升序
df_sorted = df.sort_index(ascending=False) # 降序

# 按列名排序（把列从左到右按字母排列）
df_sorted = df.sort_index(axis=1)
```

#### 3.3.3 获取最大/最小值的行

```python
# 工资最高的 3 个人
print(df.nlargest(3, '工资'))

# 年龄最小的 2 个人
print(df.nsmallest(2, '年龄'))
```

---

## 第 4 章：数据清洗

真实世界的数据往往是"脏"的：有缺失值、有重复行、数据类型不对。数据清洗是数据分析中最耗时也最重要的一步。

---

### 4.1 缺失值处理

#### 4.1.1 制造一份含缺失值的数据

```python
df = pd.DataFrame({
    '姓名': ['张三', '李四', '王五', '赵六', None],
    '年龄': [25, None, 28, 35, 22],
    '城市': ['北京', '上海', None, '深圳', '北京'],
    '工资': [10000, 15000, 12000, None, None]
})
print(df)
```

输出：

```
     姓名    年龄    城市       工资
0    张三  25.0    北京  10000.0
1    李四   NaN    上海  15000.0
2    王五  28.0   None  12000.0
3    赵六  35.0    深圳      NaN
4   None  22.0    北京      NaN
```

`NaN` 和 `None` 在 Pandas 中都表示**缺失值**。

#### 4.1.2 检测缺失值

```python
# 检测每个单元格是否是缺失值（返回同形状的 True/False 表）
print(df.isnull())       # 或 df.isna()  ← 两者完全等价

# 统计每列有多少缺失值（最常用！）
print(df.isnull().sum())
```

输出：

```
姓名    1
年龄    1
城市    1
工资    2
dtype: int64
```

```python
# 缺失值占比
print(df.isnull().sum() / len(df) * 100)

# 输出：
# 姓名    20.0
# 年龄    20.0
# 城市    20.0
# 工资    40.0

# 整个 DataFrame 是否有任何缺失值
print(df.isnull().values.any())   # True

# 每列是否有缺失值
print(df.isnull().any())
```

#### 4.1.3 删除缺失值 `dropna`

```python
# 删除含有任何缺失值的行（只要这一行有一个 NaN 就删掉）
df_clean = df.dropna()
print(df_clean)          # 只剩下张三（第0行），因为其他行都有 NaN

# 删除全部都是缺失值的行（一行全是 NaN 才删）
df_clean = df.dropna(how='all')

# 指定列：只看"工资"列，有缺失就删
df_clean = df.dropna(subset=['工资'])

# 指定列：看"姓名"和"年龄"列，任一缺失就删
df_clean = df.dropna(subset=['姓名', '年龄'])

# 阈值：至少要有 3 个非缺失值才保留
df_clean = df.dropna(thresh=3)

# 删除含有缺失值的列
df_clean = df.dropna(axis=1)  # axis=1 表示对列操作
```

#### 4.1.4 填充缺失值 `fillna`

```python
# 用固定值填充
df_filled = df.fillna(0)                     # 所有 NaN 填 0
df_filled = df.fillna('未知')                # 所有 NaN 填"未知"

# 对不同列用不同值填充（最常用）
df_filled = df.fillna({
    '姓名': '匿名',
    '年龄': df['年龄'].mean(),    # 用年龄的平均值填充
    '城市': '未知',
    '工资': df['工资'].median()   # 用工资的中位数填充
})

# 向前填充（用前一行的值填充）
df_filled = df.ffill()   # forward fill

# 向后填充（用后一行的值填充）
df_filled = df.bfill()   # backward fill

# 限制连续填充的个数
df_filled = df.ffill(limit=1)  # 最多连续填充1个
```

**选择删除还是填充？**

| 场景                | 建议                |
| ------------------- | ------------------- |
| 缺失比例很小（<5%） | 直接删除            |
| 缺失比例较大        | 用合适的值填充      |
| 数值列              | 用均值/中位数/0填充 |
| 分类列（如城市）    | 用众数或"未知"填充  |
| 时间序列            | 向前/向后填充       |

---

### 4.2 重复值处理

#### 4.2.1 检测重复值

```python
df = pd.DataFrame({
    '姓名': ['张三', '李四', '张三', '王五', '李四'],
    '年龄': [25, 30, 25, 28, 30],
    '城市': ['北京', '上海', '北京', '广州', '上海']
})

# 检测完全重复的行
print(df.duplicated())
```

输出：

```
0    False
1    False
2     True    ← 这行和第0行完全一样
3    False
4     True    ← 这行和第1行完全一样
dtype: bool
```

默认保留**第一次**出现的行（`keep='first'`），后续重复的标记为 `True`。

```python
# 保留最后一个，前面的标记为重复
print(df.duplicated(keep='last'))

# 所有重复行都标记为 True（不保留任何一个）
print(df.duplicated(keep=False))

# 只看指定列是否重复
print(df.duplicated(subset=['姓名']))

# 有多少重复行
print(df.duplicated().sum())  # 2
```

#### 4.2.2 删除重复值

```python
# 删除完全重复的行，保留第一个
df_clean = df.drop_duplicates()

# 保留最后一个
df_clean = df.drop_duplicates(keep='last')

# 基于指定列去重
df_clean = df.drop_duplicates(subset=['姓名'])

# 基于多列去重
df_clean = df.drop_duplicates(subset=['姓名', '城市'])
```

---

### 4.3 数据类型转换 `astype`

#### 4.3.1 查看数据类型

```python
print(df.dtypes)
```

常见数据类型：

| Pandas 类型  | 说明                 | 示例           |
| ------------ | -------------------- | -------------- |
| `int64`      | 整数                 | 1, 2, 100      |
| `float64`    | 浮点数               | 3.14, 2.0      |
| `object`     | 字符串（或混合类型） | '张三', '北京' |
| `bool`       | 布尔值               | True, False    |
| `datetime64` | 日期时间             | 2024-01-01     |
| `category`   | 分类类型             | 性别（男/女）  |

#### 4.3.2 类型转换

```python
df = pd.DataFrame({
    '年龄': ['25', '30', '28'],       # 注意：这里是字符串！
    '工资': ['10000.5', '15000', '12000'],
    '是否在职': [1, 0, 1]
})

print(df.dtypes)
# 年龄       object   ← 本应是数字，但读进来变成了字符串
# 工资       object
# 是否在职    int64

# 转为整数
df['年龄'] = df['年龄'].astype(int)

# 转为浮点数
df['工资'] = df['工资'].astype(float)

# 转为布尔值
df['是否在职'] = df['是否在职'].astype(bool)

# 转为字符串
df['年龄'] = df['年龄'].astype(str)

# 转为分类类型（节省内存，适合值种类少的列）
df['城市'] = df['城市'].astype('category')
```

**安全转换 `pd.to_numeric`**

当数据中混有无法转换的值时，`astype` 会报错，而 `pd.to_numeric` 可以优雅处理：

```python
s = pd.Series(['1', '2', '三', '4'])

# astype(int) 会报错！
# s.astype(int)  → ValueError

# 安全转换
s_num = pd.to_numeric(s, errors='coerce')   # 无法转换的变成 NaN
print(s_num)
# 0    1.0
# 1    2.0
# 2    NaN    ← '三' 无法转成数字，变成 NaN
# 3    4.0

# errors 参数：
#   'coerce'  → 无法转换的变成 NaN
#   'ignore'  → 无法转换的保持原样（列会变成 object 类型）
#   'raise'   → 报错（默认）
```

---

## 第 5 章：数据筛选

筛选数据是数据分析中最频繁的操作。Pandas 使用**布尔索引**来实现筛选，理解了它的原理，就能应对任何筛选需求。

---

### 5.1 布尔索引原理

```python
df = pd.DataFrame({
    '姓名': ['张三', '李四', '王五', '赵六', '钱七'],
    '年龄': [25, 30, 28, 35, 22],
    '城市': ['北京', '上海', '广州', '深圳', '北京'],
    '工资': [10000, 15000, 12000, 20000, 8000]
})
```

**第一步：创建布尔条件**

```python
condition = df['年龄'] > 27
print(condition)
```

输出：

```
0    False
1     True
2     True
3     True
4    False
Name: 年龄, dtype: bool
```

这是一个**布尔 Series**，长度和 DataFrame 的行数一样。

**第二步：用布尔 Series 筛选**

```python
result = df[condition]
print(result)
```

输出：

```
   姓名  年龄  城市    工资
1  李四   30  上海  15000
2  王五   28  广州  12000
3  赵六   35  深圳  20000
```

`True` 对应的行保留，`False` 对应的行丢弃。

**合起来写（最常见的写法）**：

```python
result = df[df['年龄'] > 27]
```

### 5.2 常用筛选操作

```python
# 等于
df[df['城市'] == '北京']

# 不等于
df[df['城市'] != '北京']

# 大于、小于、大于等于、小于等于
df[df['工资'] > 10000]
df[df['工资'] >= 10000]
df[df['年龄'] < 30]
df[df['年龄'] <= 30]

# 判断是否在列表中（类似 SQL 的 IN）
df[df['城市'].isin(['北京', '上海'])]

# 判断是否不在列表中
df[~df['城市'].isin(['北京', '上海'])]    # ~ 是取反

# 判断字符串是否包含某子串
df[df['姓名'].str.contains('三')]         # 姓名中包含"三"的行

# 判断是否为缺失值
df[df['工资'].isnull()]     # 工资为空的行
df[df['工资'].notnull()]    # 工资不为空的行

# 区间筛选（between 方法，包含两端）
df[df['年龄'].between(25, 30)]   # 25 <= 年龄 <= 30
```

---

### 5.3 多条件筛选

多个条件用逻辑运算符组合：

| 逻辑      | Python 关键字 | Pandas 中用 |
| --------- | ------------- | ----------- |
| 且（AND） | `and`         | `&`         |
| 或（OR）  | `or`          | `\|`        |
| 非（NOT） | `not`         | `~`         |

> **重要**：Pandas 中**不能用** `and` / `or` / `not`，必须用 `&` / `|` / `~`，并且**每个条件必须用小括号包起来**！

```python
# 年龄大于25 且 工资大于10000
df[(df['年龄'] > 25) & (df['工资'] > 10000)]

# 城市是北京 或 城市是上海
df[(df['城市'] == '北京') | (df['城市'] == '上海')]

# 不是北京的
df[~(df['城市'] == '北京')]

# 三个条件组合：年龄>25 且 工资>10000 且 城市不是深圳
df[(df['年龄'] > 25) & (df['工资'] > 10000) & (df['城市'] != '深圳')]
```

**常见错误**：

```python
# ❌ 错误写法（没有小括号）
df[df['年龄'] > 25 & df['工资'] > 10000]
# 会报错或得到错误结果，因为 & 的优先级高于 >

# ✅ 正确写法（加上小括号）
df[(df['年龄'] > 25) & (df['工资'] > 10000)]
```

### 5.4 使用 `query` 方法（更简洁的写法）

```python
# 等价于 df[(df['年龄'] > 25) & (df['工资'] > 10000)]
df.query('年龄 > 25 and 工资 > 10000')

# 可以用变量
min_age = 25
df.query('年龄 > @min_age')    # @符号引用外部变量

# 字符串条件
df.query('城市 == "北京"')      # 注意内层用双引号
df.query('城市 in ["北京", "上海"]')
```

`query` 的优势是在条件复杂时代码更易读。

---

## 第 6 章：分组与聚合

分组聚合类似于 SQL 的 `GROUP BY`，是数据分析的核心操作之一。

---

### 6.1 `groupby` 基础

#### 6.1.1 概念

`groupby` 的思想是 **Split - Apply - Combine**（拆分 - 应用 - 合并）：

```
原始数据          拆分(Split)          应用(Apply)        合并(Combine)
                  ┌─ 技术组 ─┐      ┌─ 技术组均值 ─┐
全部员工  →  按部门分  ├─ 市场组 ─┤  →   ├─ 市场组均值 ─┤   →   汇总结果
                  └─ 管理组 ─┘      └─ 管理组均值 ─┘
```

#### 6.1.2 基本用法

```python
df = pd.DataFrame({
    '姓名': ['张三', '李四', '王五', '赵六', '钱七', '孙八'],
    '部门': ['技术', '市场', '技术', '管理', '市场', '技术'],
    '城市': ['北京', '上海', '广州', '深圳', '北京', '上海'],
    '工资': [10000, 15000, 12000, 20000, 8000, 11000],
    '年龄': [25, 30, 28, 35, 22, 27]
})
```

```python
# 按部门分组，计算平均工资
print(df.groupby('部门')['工资'].mean())
```

输出：

```
部门
管理    20000.0
市场    11500.0
技术    11000.0
Name: 工资, dtype: float64
```

**拆解理解**：

```python
# 第1步：创建分组对象
grouped = df.groupby('部门')
print(type(grouped))  # <class 'pandas.core.groupby.DataFrameGroupBy'>

# 第2步：选择列
grouped_salary = grouped['工资']

# 第3步：应用聚合函数
result = grouped_salary.mean()

# 以上三步合起来就是：
result = df.groupby('部门')['工资'].mean()
```

#### 6.1.3 查看分组内容

```python
grouped = df.groupby('部门')

# 查看分了几组
print(grouped.ngroups)    # 3

# 查看每组包含哪些行的索引
print(grouped.groups)
# {'管理': [3], '市场': [1, 4], '技术': [0, 2, 5]}

# 遍历每个组
for name, group in grouped:
    print(f'\n=== {name} ===')
    print(group)

# 取出某一组
print(grouped.get_group('技术'))
```

#### 6.1.4 多列分组

```python
# 按部门和城市两级分组
result = df.groupby(['部门', '城市'])['工资'].mean()
print(result)
```

输出：

```
部门  城市
管理  深圳    20000.0
市场  上海    15000.0
      北京     8000.0
技术  上海    11000.0
      北京    10000.0
      广州    12000.0
Name: 工资, dtype: float64
```

这是一个**多级索引**的 Series。如果想变回普通 DataFrame：

```python
result = df.groupby(['部门', '城市'])['工资'].mean().reset_index()
print(result)
```

输出：

```
   部门  城市      工资
0  管理  深圳  20000.0
1  市场  上海  15000.0
2  市场  北京   8000.0
3  技术  上海  11000.0
4  技术  北京  10000.0
5  技术  广州  12000.0
```

`reset_index()` 会把分组的列从索引恢复为普通列，这是一个非常常用的操作。

---

### 6.2 聚合函数

#### 6.2.1 常用内置聚合函数

```python
grouped = df.groupby('部门')['工资']

print(grouped.sum())       # 求和
print(grouped.mean())      # 均值
print(grouped.median())    # 中位数
print(grouped.min())       # 最小值
print(grouped.max())       # 最大值
print(grouped.count())     # 计数（不含 NaN）
print(grouped.std())       # 标准差
print(grouped.var())       # 方差
print(grouped.first())     # 每组的第一个值
print(grouped.last())      # 每组的最后一个值
print(grouped.nunique())   # 每组的唯一值个数
```

#### 6.2.2 `agg` 方法 —— 灵活聚合

`agg` 方法允许你同时应用多个聚合函数，或对不同列应用不同的函数。

```python
# 对工资列同时求均值、总和、最大值
result = df.groupby('部门')['工资'].agg(['mean', 'sum', 'max'])
print(result)
```

输出：

```
            mean    sum    max
部门
管理    20000.0  20000  20000
市场    11500.0  23000  15000
技术    11000.0  33000  12000
```

```python
# 对不同列应用不同的聚合函数
result = df.groupby('部门').agg({
    '工资': ['mean', 'sum'],     # 工资：求均值和总和
    '年龄': 'mean',              # 年龄：只求均值
    '姓名': 'count'              # 姓名：计数（就是人数）
})
print(result)
```

输出：

```
          工资              年龄   姓名
        mean    sum       mean  count
部门
管理  20000.0  20000  35.000000      1
市场  11500.0  23000  26.000000      2
技术  11000.0  33000  26.666667      3
```

```python
# 使用自定义函数
result = df.groupby('部门')['工资'].agg(
    平均工资='mean',
    最高工资='max',
    工资极差=lambda x: x.max() - x.min()   # 自定义：最大值-最小值
)
print(result)
```

输出：

```
      平均工资  最高工资  工资极差
部门
管理  20000.0   20000       0
市场  11500.0   15000    7000
技术  11000.0   12000    2000
```

#### 6.2.3 `size()` 与 `count()` 的区别

```python
# size()：每组的行数（包含 NaN）
df.groupby('部门').size()

# count()：每列非 NaN 的个数
df.groupby('部门').count()
```

---

## 第 7 章：数据合并

在实际工作中，数据往往分散在多个表中，需要合并在一起分析。

---

### 7.1 `concat` —— 拼接

`concat` 是简单地把多个 DataFrame **纵向**或**横向**拼接在一起。

#### 7.1.1 纵向拼接（上下堆叠）

```python
df1 = pd.DataFrame({
    '姓名': ['张三', '李四'],
    '年龄': [25, 30]
})

df2 = pd.DataFrame({
    '姓名': ['王五', '赵六'],
    '年龄': [28, 35]
})

# 纵向拼接
result = pd.concat([df1, df2])
print(result)
```

输出：

```
   姓名  年龄
0  张三   25
1  李四   30
0  王五   28
1  赵六   35
```

注意索引重复了（有两个 0 和两个 1）！用 `ignore_index=True` 重建索引：

```python
result = pd.concat([df1, df2], ignore_index=True)
print(result)
```

输出：

```
   姓名  年龄
0  张三   25
1  李四   30
2  王五   28
3  赵六   35
```

**列不一致的情况**：

```python
df1 = pd.DataFrame({'姓名': ['张三'], '年龄': [25]})
df2 = pd.DataFrame({'姓名': ['李四'], '城市': ['上海']})

result = pd.concat([df1, df2], ignore_index=True)
print(result)
```

输出：

```
   姓名    年龄    城市
0  张三  25.0   NaN
1  李四   NaN   上海
```

列不一致时，缺少的列会用 NaN 填充。

#### 7.1.2 横向拼接（左右并排）

```python
df1 = pd.DataFrame({'姓名': ['张三', '李四'], '年龄': [25, 30]})
df2 = pd.DataFrame({'城市': ['北京', '上海'], '工资': [10000, 15000]})

result = pd.concat([df1, df2], axis=1)
print(result)
```

输出：

```
   姓名  年龄  城市    工资
0  张三   25  北京  10000
1  李四   30  上海  15000
```

`axis=1` 表示沿列方向拼接（横向）。

---

### 7.2 `merge` —— 关联合并（类似 SQL JOIN）

`merge` 是基于**共同的列**把两张表关联起来，和 SQL 中的 JOIN 是一回事。

#### 7.2.1 基本用法

```python
# 员工表
employees = pd.DataFrame({
    '员工ID': [1, 2, 3, 4, 5],
    '姓名': ['张三', '李四', '王五', '赵六', '钱七'],
    '部门ID': [101, 102, 101, 103, 102]
})

# 部门表
departments = pd.DataFrame({
    '部门ID': [101, 102, 103, 104],
    '部门名称': ['技术部', '市场部', '管理部', '财务部']
})

# 合并：通过"部门ID"关联
result = pd.merge(employees, departments, on='部门ID')
print(result)
```

输出：

```
   员工ID  姓名  部门ID  部门名称
0      1  张三    101   技术部
1      3  王五    101   技术部
2      2  李四    102   市场部
3      5  钱七    102   市场部
4      4  赵六    103   管理部
```

每个员工都关联上了对应的部门名称。注意财务部（104）没出现，因为没有员工属于它。

#### 7.2.2 四种合并方式（how 参数）

```python
# 内连接（默认）：只保留两边都有匹配的行
pd.merge(employees, departments, on='部门ID', how='inner')

# 左连接：保留左表所有行，右表没匹配上的填 NaN
pd.merge(employees, departments, on='部门ID', how='left')

# 右连接：保留右表所有行，左表没匹配上的填 NaN
pd.merge(employees, departments, on='部门ID', how='right')

# 外连接：保留两边所有行，没匹配上的都填 NaN
pd.merge(employees, departments, on='部门ID', how='outer')
```

用图示理解：

```
左表 employees:                右表 departments:
员工ID  部门ID                    部门ID  部门名称
  1      101                       101   技术部
  2      102                       102   市场部
  3      101                       103   管理部
  4      103                       104   财务部  ← 左表没有
  5      102

inner: 只有 101, 102, 103 的行（两边都有的）
left:  员工表的所有行都保留（左表全保留）
right: 部门表的所有行都保留（右表全保留，财务部也在，员工信息为 NaN）
outer: 所有行都保留（两边全保留）
```

```python
# 右连接示例
result = pd.merge(employees, departments, on='部门ID', how='right')
print(result)
```

输出：

```
   员工ID    姓名  部门ID  部门名称
0    1.0    张三    101   技术部
1    3.0    王五    101   技术部
2    2.0    李四    102   市场部
3    5.0    钱七    102   市场部
4    4.0    赵六    103   管理部
5    NaN   NaN    104   财务部    ← 没有员工，用 NaN 填充
```

#### 7.2.3 关联列名不同

两张表的关联列名字不同时：

```python
employees = pd.DataFrame({
    '员工ID': [1, 2, 3],
    '姓名': ['张三', '李四', '王五'],
    'dept_id': [101, 102, 101]       # 注意列名是 dept_id
})

departments = pd.DataFrame({
    '部门编号': [101, 102, 103],       # 注意列名是 部门编号
    '部门名称': ['技术部', '市场部', '管理部']
})

# 用 left_on 和 right_on 分别指定
result = pd.merge(
    employees, departments,
    left_on='dept_id',        # 左表用 dept_id
    right_on='部门编号',       # 右表用 部门编号
    how='inner'
)
print(result)
```

输出：

```
   员工ID  姓名  dept_id  部门编号  部门名称
0      1  张三      101     101   技术部
1      3  王五      101     101   技术部
2      2  李四      102     102   市场部
```

会出现两列（`dept_id` 和 `部门编号`），内容一样，可以手动删除一列。

#### 7.2.4 基于索引合并

```python
# 左表用列，右表用索引
pd.merge(employees, departments, left_on='dept_id', right_index=True)

# 两边都用索引
pd.merge(df1, df2, left_index=True, right_index=True)
```

#### 7.2.5 处理重名列

两张表有同名但含义不同的列时：

```python
df1 = pd.DataFrame({'ID': [1, 2], '分数': [90, 80]})
df2 = pd.DataFrame({'ID': [1, 2], '分数': [85, 95]})

result = pd.merge(df1, df2, on='ID', suffixes=('_期中', '_期末'))
print(result)
```

输出：

```
   ID  分数_期中  分数_期末
0   1       90       85
1   2       80       95
```

`suffixes` 参数给重名列加上后缀以区分。

---

## 第 8 章：时间序列基础

时间序列数据在金融、日志分析、物联网等领域非常常见。

---

### 8.1 datetime 转换

#### 8.1.1 创建时间数据

```python
# 从字符串转换为 datetime（最常用）
df = pd.DataFrame({
    '日期': ['2024-01-01', '2024-01-15', '2024-02-01', '2024-03-10'],
    '销售额': [1000, 1500, 1200, 1800]
})

print(df.dtypes)
# 日期     object    ← 是字符串！不是日期！
# 销售额    int64

# 转换为 datetime 类型
df['日期'] = pd.to_datetime(df['日期'])
print(df.dtypes)
# 日期     datetime64[ns]    ← 现在是日期类型了
# 销售额    int64
```

#### 8.1.2 处理各种日期格式

```python
# Pandas 能自动识别很多常见格式
pd.to_datetime('2024-01-15')
pd.to_datetime('2024/01/15')
pd.to_datetime('15-Jan-2024')
pd.to_datetime('January 15, 2024')

# 如果格式特殊，手动指定 format（更快更准确）
pd.to_datetime('01-15-2024', format='%m-%d-%Y')
pd.to_datetime('20240115', format='%Y%m%d')

# format 中的常用代码：
# %Y → 四位年份 (2024)
# %m → 两位月份 (01-12)
# %d → 两位日期 (01-31)
# %H → 小时 (00-23)
# %M → 分钟 (00-59)
# %S → 秒 (00-59)
```

#### 8.1.3 处理转换失败

```python
dates = ['2024-01-01', '2024-02-30', 'not_a_date']

# errors='coerce'：无法解析的变成 NaT（Not a Time，时间的缺失值）
result = pd.to_datetime(dates, errors='coerce')
print(result)
# DatetimeIndex(['2024-01-01', 'NaT', 'NaT'], dtype='datetime64[ns]', freq=None)
```

#### 8.1.4 提取日期的各个部分

```python
df['日期'] = pd.to_datetime(df['日期'])

# 使用 .dt 访问器
df['年'] = df['日期'].dt.year
df['月'] = df['日期'].dt.month
df['日'] = df['日期'].dt.day
df['星期几'] = df['日期'].dt.dayofweek     # 0=周一, 6=周日
df['星期名'] = df['日期'].dt.day_name()     # Monday, Tuesday, ...
df['第几周'] = df['日期'].dt.isocalendar().week
df['季度'] = df['日期'].dt.quarter
df['是否月末'] = df['日期'].dt.is_month_end
```

#### 8.1.5 日期计算

```python
# 日期加减
df['日期'] + pd.Timedelta(days=7)       # 加7天
df['日期'] - pd.Timedelta(hours=3)      # 减3小时

# 两个日期之差
df['间隔天数'] = df['日期'] - df['日期'].iloc[0]   # 每个日期和第一个日期的差

# 结果是 Timedelta 类型，取天数：
df['间隔天数'].dt.days
```

#### 8.1.6 将日期设为索引

```python
df = df.set_index('日期')
print(df)
```

输出：

```
            销售额
日期
2024-01-01   1000
2024-01-15   1500
2024-02-01   1200
2024-03-10   1800
```

设为索引后，可以用日期来选取数据：

```python
# 选取某一天
df.loc['2024-01-15']

# 选取某个月
df.loc['2024-01']

# 选取某个时间范围
df.loc['2024-01-01':'2024-02-01']
```

---

### 8.2 `resample` —— 时间重采样

`resample` 可以改变时间序列的频率，比如把日数据汇总为月数据。**前提：日期必须是索引。**

#### 8.2.1 生成示例数据

```python
# 生成日期范围
dates = pd.date_range('2024-01-01', periods=90, freq='D')  # 90天的日数据
df = pd.DataFrame({
    '销售额': np.random.randint(100, 500, 90),
    '订单数': np.random.randint(5, 30, 90)
}, index=dates)

print(df.head())
```

输出（随机数据，仅示意）：

```
            销售额  订单数
2024-01-01    342      15
2024-01-02    189      27
2024-01-03    456      8
2024-01-04    123      22
2024-01-05    267      11
```

#### 8.2.2 降采样（高频 → 低频）

```python
# 按月汇总
monthly = df.resample('ME').sum()
print(monthly)
```

输出：

```
            销售额  订单数
2024-01-31    8234    456
2024-02-29    7891    423
2024-03-31    9012    489
```

常用频率代码：

| 代码           | 含义         |
| -------------- | ------------ |
| `'D'`          | 每天         |
| `'W'`          | 每周         |
| `'ME'`         | 每月（月末） |
| `'MS'`         | 每月（月初） |
| `'QE'`         | 每季度       |
| `'YE'`         | 每年         |
| `'h'`          | 每小时       |

```python
# 按周汇总
weekly = df.resample('W').sum()

# 按月求均值
monthly_avg = df.resample('ME').mean()

# 按月使用多个聚合函数
monthly_agg = df.resample('ME').agg({
    '销售额': 'sum',     # 销售额求总和
    '订单数': 'mean'     # 订单数求均值
})
```

#### 8.2.3 升采样（低频 → 高频）

```python
# 月数据 → 日数据
monthly = df.resample('ME').sum()
daily = monthly.resample('D').ffill()   # 向前填充（每天的值等于当月的值）
```

---

## 第 9 章：简单数据分析流程

本章通过一个完整案例，串联前面所有知识点。我们将按照 **读取数据 → 清洗 → 分组统计 → 输出结果** 的流程进行一次完整的数据分析。

---

### 9.1 案例背景

假设我们有一份电商销售数据 `sales.csv`，包含以下字段：

| 字段     | 说明                    |
| -------- | ----------------------- |
| 订单ID   | 唯一标识                |
| 日期     | 下单日期                |
| 商品分类 | 电子产品/服装/食品/家居 |
| 商品名称 | 具体商品名              |
| 数量     | 购买数量                |
| 单价     | 单件价格                |
| 城市     | 买家所在城市            |

### 9.2 第一步：生成模拟数据并读取

由于我们没有真实文件，先用代码生成模拟数据：

```python
import pandas as pd
import numpy as np

# 设置随机种子保证可复现
np.random.seed(42)

n = 500  # 500条订单

categories = ['电子产品', '服装', '食品', '家居']
products = {
    '电子产品': ['手机', '耳机', '平板', '充电器'],
    '服装': ['T恤', '牛仔裤', '外套', '运动鞋'],
    '食品': ['零食大礼包', '坚果', '咖啡', '巧克力'],
    '家居': ['台灯', '抱枕', '收纳盒', '香薰']
}
cities = ['北京', '上海', '广州', '深圳', '杭州', '成都']

# 生成数据
data = {
    '订单ID': [f'ORD{str(i).zfill(5)}' for i in range(1, n + 1)],
    '日期': pd.date_range('2024-01-01', periods=n, freq='8h'),
    '商品分类': np.random.choice(categories, n),
    '数量': np.random.randint(1, 10, n),
    '城市': np.random.choice(cities, n)
}

# 根据分类匹配商品名称
data['商品名称'] = [np.random.choice(products[cat]) for cat in data['商品分类']]

# 根据分类设定价格范围
price_ranges = {'电子产品': (200, 5000), '服装': (50, 500), '食品': (10, 100), '家居': (30, 300)}
data['单价'] = [round(np.random.uniform(*price_ranges[cat]), 2) for cat in data['商品分类']]

df = pd.DataFrame(data)

# 人为制造一些"脏"数据
df.loc[np.random.choice(n, 15), '数量'] = np.nan       # 15个缺失值
df.loc[np.random.choice(n, 10), '单价'] = np.nan       # 10个缺失值
df.loc[np.random.choice(n, 5), '城市'] = np.nan        # 5个缺失值

# 制造重复行
dup_rows = df.iloc[:8].copy()
df = pd.concat([df, dup_rows], ignore_index=True)

# 保存为 CSV（模拟真实场景）
df.to_csv('sales.csv', index=False, encoding='utf-8-sig')
print("数据已生成并保存为 sales.csv")
```

**现在正式开始分析——从读取 CSV 开始：**

```python
# ========== 读取数据 ==========
df = pd.read_csv('sales.csv', encoding='utf-8-sig')

# 第一步永远是：看看数据长什么样
print("="*50)
print("数据前5行：")
print(df.head())
print(f"\n数据形状：{df.shape}（{df.shape[0]}行, {df.shape[1]}列）")
print(f"\n数据信息：")
df.info()
print(f"\n数值列统计：")
print(df.describe())
```

---

### 9.3 第二步：数据清洗

```python
# ========== 数据清洗 ==========

# 1. 检查缺失值
print("\n各列缺失值数量：")
print(df.isnull().sum())
print(f"\n总缺失值：{df.isnull().sum().sum()}")

# 2. 处理缺失值
# 数量：用该商品分类的中位数填充
df['数量'] = df.groupby('商品分类')['数量'].transform(
    lambda x: x.fillna(x.median())
)

# 单价：用该商品分类的中位数填充
df['单价'] = df.groupby('商品分类')['单价'].transform(
    lambda x: x.fillna(x.median())
)

# 城市：用"未知"填充
df['城市'] = df['城市'].fillna('未知')

# 验证缺失值已处理完毕
print("\n处理后各列缺失值：")
print(df.isnull().sum())

# 3. 检查并删除重复行
print(f"\n重复行数量：{df.duplicated().sum()}")
df = df.drop_duplicates()
print(f"删除重复后行数：{len(df)}")

# 4. 数据类型转换
df['日期'] = pd.to_datetime(df['日期'])
df['数量'] = df['数量'].astype(int)

# 5. 新增计算列
df['总金额'] = df['数量'] * df['单价']
df['总金额'] = df['总金额'].round(2)

# 6. 提取时间维度
df['月份'] = df['日期'].dt.month
df['星期'] = df['日期'].dt.day_name()

print("\n清洗完成！最终数据形状：", df.shape)
print(df.head())
```

---

### 9.4 第三步：分组统计分析

```python
# ========== 数据分析 ==========

# ---- 分析1：各商品分类的销售概况 ----
category_stats = df.groupby('商品分类').agg(
    订单数=('订单ID', 'count'),
    总销售额=('总金额', 'sum'),
    平均订单金额=('总金额', 'mean'),
    总销量=('数量', 'sum')
).round(2)
category_stats = category_stats.sort_values('总销售额', ascending=False)
print("\n=== 各分类销售概况 ===")
print(category_stats)

# ---- 分析2：各城市的销售排名 ----
city_stats = df.groupby('城市').agg(
    订单数=('订单ID', 'count'),
    总销售额=('总金额', 'sum')
).sort_values('总销售额', ascending=False).round(2)
print("\n=== 各城市销售排名 ===")
print(city_stats)

# ---- 分析3：月度销售趋势 ----
monthly_trend = df.set_index('日期').resample('ME').agg(
    订单数=('订单ID', 'count'),
    总销售额=('总金额', 'sum')
).round(2)
print("\n=== 月度销售趋势 ===")
print(monthly_trend)

# ---- 分析4：热销商品 TOP 10 ----
top_products = df.groupby('商品名称').agg(
    总销量=('数量', 'sum'),
    总销售额=('总金额', 'sum')
).sort_values('总销售额', ascending=False).head(10).round(2)
print("\n=== 热销商品 TOP 10 ===")
print(top_products)

# ---- 分析5：各分类在各城市的销售额（交叉分析） ----
cross_table = df.pivot_table(
    values='总金额',
    index='商品分类',
    columns='城市',
    aggfunc='sum'
).round(2)
print("\n=== 分类×城市 销售额交叉表 ===")
print(cross_table)
```

---

### 9.5 第四步：输出结果

```python
# ========== 输出结果 ==========

# 方式1：保存清洗后的数据
df.to_csv('sales_cleaned.csv', index=False, encoding='utf-8-sig')

# 方式2：多个分析结果保存到同一个 Excel 文件的不同工作表
with pd.ExcelWriter('sales_report.xlsx') as writer:
    category_stats.to_excel(writer, sheet_name='分类销售概况')
    city_stats.to_excel(writer, sheet_name='城市销售排名')
    monthly_trend.to_excel(writer, sheet_name='月度趋势')
    top_products.to_excel(writer, sheet_name='热销商品TOP10')
    cross_table.to_excel(writer, sheet_name='交叉分析')

print("\n报告已保存！")
print(" - sales_cleaned.csv（清洗后的数据）")
print(" - sales_report.xlsx（分析报告，含5个工作表）")
```

---

### 9.6 完整流程总结

```
读取数据           pd.read_csv() / pd.read_excel()
   ↓
了解数据           df.head() / df.info() / df.describe() / df.shape
   ↓
处理缺失值         df.isnull().sum() → df.fillna() / df.dropna()
   ↓
处理重复值         df.duplicated().sum() → df.drop_duplicates()
   ↓
类型转换           pd.to_datetime() / df.astype()
   ↓
新增计算列         df['新列'] = 表达式
   ↓
数据筛选           df[条件] / df.query()
   ↓
分组聚合           df.groupby().agg()
   ↓
时间重采样         df.resample()
   ↓
输出结果           df.to_csv() / df.to_excel()
```

---

## 附录：常用速查表

### 创建数据

| 操作                 | 代码                                                |
| -------------------- | --------------------------------------------------- |
| 从字典创建 DataFrame | `pd.DataFrame({'A': [1,2], 'B': [3,4]})`            |
| 从列表创建 Series    | `pd.Series([1, 2, 3])`                              |
| 生成日期序列         | `pd.date_range('2024-01-01', periods=30, freq='D')` |

### 读写文件

| 操作     | 代码                                              |
| -------- | ------------------------------------------------- |
| 读 CSV   | `pd.read_csv('file.csv', encoding='utf-8')`       |
| 写 CSV   | `df.to_csv('file.csv', index=False)`              |
| 读 Excel | `pd.read_excel('file.xlsx', sheet_name='Sheet1')` |
| 写 Excel | `df.to_excel('file.xlsx', index=False)`           |

### 查看数据

| 操作     | 代码            |
| -------- | --------------- |
| 前 N 行  | `df.head(N)`    |
| 形状     | `df.shape`      |
| 列名     | `df.columns`    |
| 数据类型 | `df.dtypes`     |
| 统计信息 | `df.info()`     |
| 数值摘要 | `df.describe()` |

### 选取数据

| 操作       | 代码                                |
| ---------- | ----------------------------------- |
| 选列       | `df['列名']` / `df[['列1', '列2']]` |
| 按标签选   | `df.loc[行标签, 列标签]`            |
| 按位置选   | `df.iloc[行号, 列号]`               |
| 条件筛选   | `df[df['列'] > 值]`                 |
| 多条件筛选 | `df[(条件1) & (条件2)]`             |

### 数据清洗

| 操作       | 代码                    |
| ---------- | ----------------------- |
| 缺失值统计 | `df.isnull().sum()`     |
| 删除缺失行 | `df.dropna()`           |
| 填充缺失值 | `df.fillna(值)`         |
| 删除重复行 | `df.drop_duplicates()`  |
| 类型转换   | `df['列'].astype(类型)` |

### 分组与合并

| 操作     | 代码                                          |
| -------- | --------------------------------------------- |
| 分组聚合 | `df.groupby('列').agg({'列2': 'sum'})`        |
| 纵向拼接 | `pd.concat([df1, df2], ignore_index=True)`    |
| 关联合并 | `pd.merge(df1, df2, on='共同列', how='left')` |

---

以上就是 Pandas 从入门到基础实战的完整教程。每个知识点都在第 9 章的案例中得到了实际应用。建议你在学习每一章后，都回到第 9 章的对应步骤中看看它是如何被用起来的，这样才能真正把知识串联成技能。
