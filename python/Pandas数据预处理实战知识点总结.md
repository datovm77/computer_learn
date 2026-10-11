# Pandas 数据预处理超全核心指南（原理剖析 + 实战精要 + 避坑指南）

> **适用对象**：数据分析初学者、Python 数据清洗实战进阶  
> **练习背景**：基于《Pandas 数据预处理 Part 1》实战环境，全面系统拆解核心考点与原理  
> **核心工具库**：`pandas`, `numpy`, `matplotlib.pyplot`

---

## 目录
1. [为什么要进行数据预处理？](#一为什么要进行数据预处理)
2. [缺失值（空值）处理详解](#二缺失值空值处理详解)
3. [重复值处理详解](#三重复值处理详解)
4. [异常值（离群点）检测与清洗详解](#四异常值离群点检测与清洗详解)
5. [数据拼接与格式规范](#五数据拼接与格式规范)
6. [Matplotlib 画图中文乱码原理与终极解决方案](#六matplotlib-画图中文乱码原理与终极解决方案)
7. [初学者高频踩坑与核心 FAQ](#七初学者高频踩坑与核心-faq)
8. [今日实战练习题目逐题逐行代码精解](#八今日实战练习题目逐题逐行代码精解)

---

## 一、为什么要进行数据预处理？

真实世界中的数据往往是“脏”的（Dirty Data）：
- **缺失（Incomplete）**：问卷用户漏填、传感器信号丢失、数据传输丢包。
- **重复（Duplicate）**：网络爬虫重复抓取、用户重复提交订单、系统多次导入。
- **异常（Noisy / Outlier）**：录入失误（把年龄 18 输成 180）、设备故障极端读数。

> **数据分析界的黄金法则**：*Garbage In, Garbage Out（输入垃圾，产出的只能是垃圾）*。  
> 预处理的目的就是把“脏数据”变成规范、干净、可直接用于建模和统计分析的“高质量数据”。

---

## 二、缺失值（空值）处理详解

### 1. 缺失值的表现形式
在 Pandas 中，缺失值主要有两种表示：
- `np.nan` 或 `float('nan')`：浮点型空值，源自 NumPy，表示“不是一个数字”。
- `None`：Python 自带的空对象，在 Pandas 数值列中通常会被自动转为 `NaN`。

### 2. 缺失值的检测：`notnull()` 与 `isnull()`

#### （1）函数对比
| 函数 | 作用 | 遇空值返回 | 遇正常值返回 |
| :--- | :--- | :--- | :--- |
| `pd.isnull(obj)` / `obj.isnull()` | 判断元素是否为缺失值 | `True` | `False` |
| `pd.notnull(obj)` / `obj.notnull()` | 判断元素是否为有效非空值 | `False` | `True` |

> **提示**：`isna()` 与 `isnull()` 功能完全一致；`notna()` 与 `notnull()` 完全一致。

#### （2）实战代码
```python
import pandas as pd
import numpy as np

s = pd.Series([1, None, np.nan])

# 检查非空：结果为 [True, False, False]
s.notnull()

# 检查空值：结果为 [False, True, True]
s.isnull()
```

---

### 3. 删除缺失值：`dropna()`

#### （1）常用核心参数
```python
df.dropna(axis=0, how='any', thresh=None, subset=None, inplace=False)
```
- **`axis`**：
  - `0` 或 `'index'`（默认）：按行删除，如果某行有缺失值，整行删掉。
  - `1` 或 `'columns'`：按列删除，如果某列有缺失值，整列删掉。
- **`how`**：
  - `'any'`（默认）：只要有一个缺失值就删。
  - `'all'`：该行（或该列）**全部都是缺失值**时才删。
- **`subset`**：指定只在某些特定列中检查缺失值。例如 `subset=['书名']`，只有书名为空才删，作者为空不删。
- **`inplace`**：
  - `False`（默认）：返回删除后的新 DataFrame，原数据不受影响。
  - `True`：直接在原对象上修改，不返回新对象。

#### （2）实战代码
```python
df_obj = pd.DataFrame({
    "类别": ['小说', '散文随笔', '青春文学', '传记'],
    "书名": [np.nan, '《皮囊》', '《旅程结束时》', '《老舍自传》'],
    "作者": ["老舍", None, "张其鑫", "老舍"]
})

# 删除只要含有缺失值的行（第0行和第1行将被剔除）
clean_df = df_obj.dropna()
```

---

### 4. 填充缺失值：`fillna()`

删除数据会导致样本量缩减；如果数据很宝贵，我们通常选择**填充（Imputation）**。

#### （1）常用填充方式
1. **全局统一填充标量**：
   ```python
   df.fillna(0)     # 将所有缺失值填为 0
   df.fillna(10)    # 将所有缺失值填为 10
   ```
2. **字典映射（不同列填不同值）**：
   ```python
   # B 列填 4，D 列填 6
   df.fillna({'B': 4, 'D': 6})
   ```
3. **基于邻近值的填充（时间序列常用）**：
   - `method='ffill'`（Forward Fill，前向填充）：用缺失值**上方（前面）**的最近一个有效值填补。
   - `method='bfill'`（Backward Fill，后向填充）：用缺失值**下方（后面）**的最近一个有效值填补。
   ```python
   df.fillna(method='ffill')
   ```
4. **统计量填充（均值、中位数、众数）**：
   - 数值型数据常使用**均值（mean）**或**中位数（median）**填充。
   ```python
   # 计算常住人口均值
   mean_val = df['常住人口（万人）'].mean()
   # 原地填充
   df['常住人口（万人）'].fillna(mean_val, inplace=True)
   ```

---

## 三、重复值处理详解

在数据集中，由于多次抓取或重复录入，经常出现完全相同或某唯一标识（如学号、身份证号、订单号）相同的多行记录。

### 1. 检测重复值：`duplicated()`

#### （1）函数原理
```python
df.duplicated(subset=None, keep='first')
```
- **`subset`**：指定考虑哪些列。如果不填，默认整行所有列都相同才算重复。
- **`keep` 参数三种取值的关键区别**：
  - `'first'`（默认，从前往后）：保留第一次出现的记录，**第二次及后续出现的标记为 `True`（重复）**。
  - `'last'`（从后向前）：保留最后一次出现的记录，**前面的重复项标记为 `True`（重复）**。
  - `False`：只要有重复，**所有参与重复的行全被标记为 `True`**。

#### （2）图解示例
假设数据为：`[A, B, A, C]`
- 使用 `keep='first'`：返回 `[False, False, True, False]`（第3个 A 是多余的）
- 使用 `keep='last'`：返回 `[True, False, False, False]`（第1个 A 是多余的）
- 使用 `keep=False`：返回 `[True, False, True, False]`（两个 A 统统标为重复）

```python
# 实战：从后向前查找和判断是否有重复值
person_info.duplicated(keep='last')
```

### 2. 删除重复值：`drop_duplicates()`
```python
# 默认保留第一次出现的数据，删除后续重复项
person_info.drop_duplicates()

# 原地修改原数据
person_info.drop_duplicates(inplace=True)
```

---

## 四、异常值（离群点）检测与清洗详解

异常值是指数据集中明显偏离其他观测值的样本点。异常值可能是输入错误（如体重大于 1000kg），也可能是真实的极端表现。

常用的两种检测方法：
1. **$3\sigma$ 原则（参数方法，需正态分布）**
2. **箱线图 / IQR 法（非参数方法，通用稳健）**

---

### 1. 基于 $3\sigma$ 原则（拉依达准则）的检测

#### （1）理论依据
根据正态分布（高斯分布）的概率密度性质：
- 数值落在 $(\mu - \sigma, \mu + \sigma)$ 区间的概率约为 **68.27%**
- 数值落在 $(\mu - 2\sigma, \mu + 2\sigma)$ 区间的概率约为 **95.45%**
- 数值落在 $(\mu - 3\sigma, \mu + 3\sigma)$ 区间的概率约为 **99.73%**

超出 $(\mu - 3\sigma, \mu + 3\sigma)$ 范围的概率仅有 **0.27%**，属于极小概率事件。一旦出现，在工程上通常判定为异常。

```
              μ-3σ    μ-2σ    μ-1σ     μ     μ+1σ    μ+2σ    μ+3σ
────────────────┼───────┼───────┼───────┼───────┼───────┼───────► 数值
  [异常点 <]   │ ◄────── 99.73% 正常数据区间 ──────► │   [> 异常点]
```

#### （2）代码实现与逐步剖析
```python
def three_sigma(ser1): 
    # 第一步：计算均值 μ
    mean_value = ser1.mean()
    
    # 第二步：计算标准差 σ
    std_value = ser1.std()
    
    # 第三步：构建布尔掩码（Boolean Mask）
    # 只要数值小于 μ-3σ 或者大于 μ+3σ，即标记为 True
    rule = (mean_value - 3 * std_value > ser1) | (mean_value + 3 * std_value < ser1)
    
    # 第四步：提取异常值所在的行索引与数据
    index = np.arange(ser1.shape[0])[rule]
    outrange = ser1.iloc[index]
    
    return outrange
```

> **特别注意**：
> - Pandas 条件并列时必须使用按位运算符 `|`（或）、`&`（且），**不能使用** Python 的 `or` / `and`。
> - 每个判断条件必须用小括号 `()` 括起来，因为比较运算符的优先级低于按位运算符。

---

### 2. 基于箱线图（Boxplot / IQR 四分位距）的检测与处理

$3\sigma$ 原则的软肋在于：如果数据本身严重偏态或者已经有极大异常值，均值 $\mu$ 和标准差 $\sigma$ 自身就会被拉偏。  
因此在实际工程中，**箱线图 IQR 法（Tukey 法）**更为广泛通用。

#### （1）核心统计量与计算公式
1. **$Q1$（第一四分位数 / 25%分位数）**：数据排序后位于 25% 处的数值。
   $$\text{q1} = \text{ser.quantile}(0.25)$$
2. **$Q3$（第三四分位数 / 75%分位数）**：数据排序后位于 75% 处的数值。
   $$\text{q3} = \text{ser.quantile}(0.75)$$
3. **$IQR$（四分位距 / Interquartile Range）**：中间 50% 数据的跨度范围。
   $$IQR = Q3 - Q1$$
4. **正常范围边界（内限）**：
   - 下限（Lower Bound）：$Q1 - 1.5 \times IQR$
   - 上限（Upper Bound）：$Q3 + 1.5 \times IQR$

凡是小于下限或大于上限的数据点，均视为异常值。

#### （2）找出异常值
```python
q1 = df['B'].quantile(0.25)
q3 = df['B'].quantile(0.75)
iqr = q3 - q1

# 提取异常值
outliers = df['B'][(df['B'] < q1 - 1.5 * iqr) | (df['B'] > q3 + 1.5 * iqr)]
```

#### （3）异常值处理策略

##### 策略 A：用剔除异常值后的均值替换
先算出所有正常数据的均值，再把异常值更新为该均值：
```python
# 筛选正常数据的均值
clean_mean = df['B'][(df['B'] >= q1 - 1.5 * iqr) & (df['B'] <= q3 + 1.5 * iqr)].mean()

# 将异常值赋值为 clean_mean
df.loc[(df['B'] < q1 - 1.5 * iqr) | (df['B'] > q3 + 1.5 * iqr), 'B'] = clean_mean
```

##### 策略 B：缩尾截断替代（Winsorization / 盖帽法）
当异常值不能删除（要保留记录行），且希望削弱极端值对整体分布的影响时：
- 大于上限的，用**上限**替代；
- 小于下限的，用**下限**替代。

```python
for col in ['GDP（亿元）', '常住人口（万人）']:
    q1 = df[col].quantile(0.25)
    q3 = df[col].quantile(0.75)
    iqr = q3 - q1
    low = q1 - 1.5 * iqr
    high = q3 + 1.5 * iqr
    
    # 截断处理
    df.loc[df[col] < low, col] = low
    df.loc[df[col] > high, col] = high
```

---

## 五、数据拼接与格式规范

### 1. 文件读取编码安全
国内的 Excel 导出的 CSV 文件经常使用 `GBK` 或 `GB2312` 编码。
```python
# 默认 encoding='utf-8'，遇到 GBK 文件会报 UnicodeDecodeError
# 安全做法：指定 encoding='gbk'
df = pd.read_csv('北京地区信息.csv', encoding='gbk')
```

### 2. 数值精度控制
```python
# 将常住人口列保留两位小数
df['常住人口（万人）'] = df['常住人口（万人）'].round(2)
```

### 3. 数据集垂直堆叠合并：`pd.concat`
当两个表字段结构相同，分别代表不同省份/地区的数据时：
```python
# ignore_index=True 会重置合并后的连续行索引 (0, 1, 2, ...)
merged_df = pd.concat([df_bj, df_tj], ignore_index=True)
```

---

## 六、Matplotlib 画图中文乱码原理与终极解决方案

### 1. 为什么会乱码？
Matplotlib 是国外开发的绘图引擎，其内置的默认英文字体（DejaVu Sans）**没有中文字形库**。  
当你在绘图时传入中文字符（例如 `'GDP（亿元）'`、`'行政面积（K㎡）'`）时：
1. 字体引擎在字符集中查不到该汉字的 Unicode 编码，会产生 `RuntimeWarning: Glyph ... missing from current font.` 警告；
2. 在 Jupyter Notebook 中，该警告会以鲜艳的**粉红/红色背景框**显示，常被误认为是代码报错；
3. 输出的图表轴标签全部变成空心方框 `□□`。

### 2. 终极解决方案
在任何画图代码执行前，配置全局黑体中文字体：
```python
import matplotlib.pyplot as plt

# 1. 设置默认字体为 SimHei（黑体）
plt.rcParams['font.sans-serif'] = ['SimHei']

# 2. 解决保存图像时负号 '-' 被解释为乱码方块的问题
plt.rcParams['axes.unicode_minus'] = False
```

配置后，无论是箱线图、折线图还是柱状图，中文均可完美呈现。

---

## 七、初学者高频踩坑与核心 FAQ

### Q1: 为什么我执行了 `df.dropna()`，再查看 `df` 缺失值依然存在？
- **解答**：Pandas 的绝大多数数据清洗方法默认是**非原地修改（not in-place）**的，它会返回一个全新的副本。必须重新赋值 `df = df.dropna()`，或者传入参数 `df.dropna(inplace=True)`。

### Q2: 为什么条件筛选时不能用 `and` / `or`，必须用 `&` / `|`？
- **解答**：Python 原生的 `and` 和 `or` 只能对单个布尔标量（Boolean Scalar）进行短路求值；而 Pandas Series 是由一组值构成的向量，必须使用按位与 `&` 和按位或 `|` 进行元素级（Element-wise）的逻辑比较。

### Q3: 为什么修改数据时出现 `SettingWithCopyWarning`？
- **解答**：如果你使用链式切片（如 `df[df['A']>2]['B'] = 10`），Pandas 无法确认你修改的是原数据还是临时视图。  
  **安全规范写法**：一律使用 `.loc[行条件, 列名]`，例如：
  ```python
  df.loc[df['A'] > 2, 'B'] = 10
  ```

---

## 八、今日实战练习题目逐题逐行代码精解

以下为本次练习 01.ipynb 中的全部实战题目代码精解，方便对照复习：

### 题目 1：检查 series_obj 是否不为空值或缺失值
```python
# 原题代码：
from pandas import DataFrame, Series
import pandas as pd
from numpy import NaN
series_obj = Series([1, None, NaN])

# 核心解答：
pd.notnull(series_obj)
# 解释：对序列每个元素检测，1 返回 True，None 和 NaN 返回 False
```

### 题目 2：检查 series_obj 中是否有空值或缺失值
```python
# 核心解答：
pd.isnull(series_obj)
# 解释：与 notnull 相反，1 返回 False，None 和 NaN 返回 True
```

### 题目 3：删除数据集中缺失值和有空值的行
```python
# 核心解答：
df_obj.dropna()
# 解释：默认 axis=0，包含空值的行将被过滤剔除
```

### 题目 4：使用 10 替换 df_obj 中缺失值
```python
# 核心解答：
df_obj.fillna(10)
# 解释：将 DataFrame 中所有 NaN 替换为数值 10
```

### 题目 5：B 列缺失值填充 4，D 列缺失值填充 6
```python
# 核心解答：
df_obj.fillna({'B': 4, 'D': 6})
# 解释：使用列名字典映射，针对不同列定制不同填充值
```

### 题目 6：使用前向填充的方式替换空值或缺失值
```python
# 核心解答：
df.fillna(method='ffill')
# 解释：ffill 即 forward fill，用每一列上方最近的非空值填充当前空值
```

### 题目 7：从后向前查找和判断是否有重复值
```python
# 核心解答：
person_info.duplicated(keep='last')
# 解释：keep='last' 会从下往上看，将上面的重复行标记为 True
```

### 题目 8：删除重复值
```python
# 核心解答：
person_info.drop_duplicates()
# 解释：去除重复行，只保留第一次出现的记录
```

### 题目 9：实现基于 3σ 原则的异常值检测函数
```python
# 核心解答：
def three_sigma(ser1): 
    mean_value = ser1.mean()
    std_value = ser1.std()
    rule = (mean_value - 3 * std_value > ser1) | (mean_value + 3 * std_value < ser1)
    index = np.arange(ser1.shape[0])[rule]
    outrange = ser1.iloc[index]
    return outrange
```

### 题目 10：利用箱型图检测方法找出 B 列的异常值
```python
# 核心解答：
q1 = df['B'].quantile(0.25)
q3 = df['B'].quantile(0.75)
iqr = q3 - q1
outlier_b = df['B'][(df['B'] < q1 - 1.5 * iqr) | (df['B'] > q3 + 1.5 * iqr)]
outlier_b
# 解释：计算出 1.5*IQR 边界，利用布尔切片筛选出超过边界的值（本题中为 40）
```

### 题目 11：将 B 列中的异常值用去除异常值后的均值替代
```python
# 核心解答：
mean_val = df['B'][(df['B'] >= q1 - 1.5 * iqr) & (df['B'] <= q3 + 1.5 * iqr)].mean()
df.loc[(df['B'] < q1 - 1.5 * iqr) | (df['B'] > q3 + 1.5 * iqr), 'B'] = mean_val
df
# 解释：去除 40 后剩下的数值 [2, 3, 2] 均值为 2.333333，将 40 替换为该均值
```

### 题目 12：画出北京地区信息的箱线图并进行异常值上下限替代
```python
# 核心解答：
bj_info.boxplot(column=['GDP（亿元）', '常住人口（万人）'])
plt.show()
for col in ['GDP（亿元）', '常住人口（万人）']:
    q1 = bj_info[col].quantile(0.25)
    q3 = bj_info[col].quantile(0.75)
    iqr = q3 - q1
    low = q1 - 1.5 * iqr
    high = q3 + 1.5 * iqr
    bj_info.loc[bj_info[col] < low, col] = low
    bj_info.loc[bj_info[col] > high, col] = high
bj_info
# 解释：将极端偏大的值压缩至上限 high，极端偏小的值压缩至下限 low（缩尾处理）
```

### 题目 13：对两地信息数据进行合并
```python
# 核心解答：
pd.concat([bj_info, tj_info], ignore_index=True)
# 解释：将北京数据和天津数据纵向合并为 32 行的大表，并重置序号
```

---
*本文档已永久保存在您的 Windows 桌面上，随时可供查阅与复习。*
