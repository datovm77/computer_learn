# NumPy 完整入门教程

## 第一章：NumPy 简介与安装

### 1.1 什么是 NumPy？

NumPy（Numerical Python）是 Python 中用于科学计算的核心库。它提供了高性能的多维数组对象和用于处理这些数组的工具。NumPy 是数据科学、机器学习、深度学习等领域的基础。

**为什么需要 NumPy？**

Python 原生的列表虽然灵活，但在处理大规模数值计算时效率较低。NumPy 的优势：
- **速度快**：底层用 C 语言实现，比 Python 列表快 10-100 倍
- **内存效率高**：数组元素类型统一，内存占用更小
- **功能强大**：提供大量数学函数和线性代数运算
- **广播机制**：可以对不同形状的数组进行运算

### 1.2 安装 NumPy

打开终端或命令提示符，输入：

```bash
pip install numpy
```

验证安装：

```python
import numpy as np
print(np.__version__)  # 输出 NumPy 版本号
```

**约定俗成的导入方式**：通常使用 `np` 作为 NumPy 的别名，这是社区标准。

---

## 第二章：ndarray 数组基础

### 2.1 什么是 ndarray？

ndarray（N-dimensional array）是 NumPy 的核心数据结构，是一个多维数组对象。

**ndarray 与 Python 列表的区别**：

| 特性     | Python 列表      | NumPy ndarray          |
| -------- | ---------------- | ---------------------- |
| 元素类型 | 可以混合不同类型 | 所有元素必须是同一类型 |
| 性能     | 较慢             | 非常快                 |
| 内存占用 | 较大             | 较小                   |
| 功能     | 基础操作         | 丰富的数学运算         |

### 2.2 创建第一个数组

```python
import numpy as np

# 从 Python 列表创建一维数组
arr1 = np.array([1, 2, 3, 4, 5])
print(arr1)  # 输出: [1 2 3 4 5]
print(type(arr1))  # 输出: <class 'numpy.ndarray'>

# 从 Python 列表创建二维数组
arr2 = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2)
# 输出:
# [[1 2 3]
#  [4 5 6]]

# 从 Python 列表创建三维数组
arr3 = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
print(arr3)
# 输出:
# [[[1 2]
#   [3 4]]
#
#  [[5 6]
#   [7 8]]]
```

**理解维度**：
- **一维数组**：像一条直线，例如 `[1, 2, 3, 4]`
- **二维数组**：像一个表格（行和列），例如矩阵
- **三维数组**：像一个立方体，可以想象成多个表格堆叠

---

## 第三章：数组创建与属性

### 3.1 多种数组创建方法

#### 3.1.1 使用 np.array() 创建

```python
# 显式指定数据类型
arr_int = np.array([1, 2, 3], dtype=int)
arr_float = np.array([1, 2, 3], dtype=float)
print(arr_int)    # 输出: [1 2 3]
print(arr_float)  # 输出: [1. 2. 3.]
```

#### 3.1.2 创建特殊数组

```python
# 创建全零数组
zeros = np.zeros(5)  # 一维，5个元素
print(zeros)  # 输出: [0. 0. 0. 0. 0.]

zeros_2d = np.zeros((3, 4))  # 二维，3行4列
print(zeros_2d)
# 输出:
# [[0. 0. 0. 0.]
#  [0. 0. 0. 0.]
#  [0. 0. 0. 0.]]

# 创建全一数组
ones = np.ones((2, 3))
print(ones)
# 输出:
# [[1. 1. 1.]
#  [1. 1. 1.]]

# 创建指定值的数组
full = np.full((2, 3), 7)  # 填充数字 7
print(full)
# 输出:
# [[7 7 7]
#  [7 7 7]]

# 创建单位矩阵（对角线为1，其余为0）
identity = np.eye(3)
print(identity)
# 输出:
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]

# 创建未初始化数组（值随机，速度快）
empty = np.empty((2, 2))
print(empty)  # 输出的值是内存中的随机值
```

#### 3.1.3 创建数值序列

```python
# arange：类似 Python 的 range，但返回数组
arr_range = np.arange(10)  # 从 0 到 9
print(arr_range)  # 输出: [0 1 2 3 4 5 6 7 8 9]

arr_range2 = np.arange(5, 15)  # 从 5 到 14
print(arr_range2)  # 输出: [ 5  6  7  8  9 10 11 12 13 14]

arr_range3 = np.arange(0, 10, 2)  # 从 0 到 9，步长为 2
print(arr_range3)  # 输出: [0 2 4 6 8]

# linspace：创建等间隔的数值序列
# 语法：np.linspace(起始值, 结束值, 元素个数)
arr_linspace = np.linspace(0, 1, 5)  # 从 0 到 1，均匀分成 5 个数
print(arr_linspace)  # 输出: [0.   0.25 0.5  0.75 1.  ]

# logspace：创建等比数列
arr_logspace = np.logspace(0, 2, 5)  # 10^0 到 10^2，5个数
print(arr_logspace)  # 输出: [  1.           3.16227766  10.          31.6227766  100.        ]
```

**arange 与 linspace 的区别**：
- `arange`：指定步长，不包含结束值
- `linspace`：指定元素个数，包含结束值

#### 3.1.4 从其他数组创建

```python
# 创建与另一个数组形状相同的数组
arr = np.array([[1, 2], [3, 4]])

zeros_like = np.zeros_like(arr)  # 形状相同的全零数组
print(zeros_like)
# 输出:
# [[0 0]
#  [0 0]]

ones_like = np.ones_like(arr)  # 形状相同的全一数组
print(ones_like)
# 输出:
# [[1 1]
#  [1 1]]
```

### 3.2 数组的重要属性

```python
arr = np.array([[1, 2, 3, 4], 
                [5, 6, 7, 8], 
                [9, 10, 11, 12]])

# ndim：数组的维度数（轴的个数）
print(arr.ndim)  # 输出: 2（二维数组）

# shape：数组的形状（各维度的大小）
print(arr.shape)  # 输出: (3, 4) 表示 3 行 4 列

# size：数组中元素的总个数
print(arr.size)  # 输出: 12（3 × 4 = 12）

# dtype：数组元素的数据类型
print(arr.dtype)  # 输出: int64 或 int32（取决于系统）

# itemsize：每个元素占用的字节数
print(arr.itemsize)  # 输出: 8（int64 占 8 字节）

# nbytes：数组占用的总字节数
print(arr.nbytes)  # 输出: 96（12 个元素 × 8 字节）
```

**形状理解示例**：
```python
# 一维数组
arr1d = np.array([1, 2, 3, 4, 5])
print(arr1d.shape)  # 输出: (5,) 表示有 5 个元素

# 二维数组
arr2d = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2d.shape)  # 输出: (2, 3) 表示 2 行 3 列

# 三维数组
arr3d = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
print(arr3d.shape)  # 输出: (2, 2, 2) 表示 2 个 2×2 的矩阵
```

### 3.3 数据类型详解

NumPy 支持多种数据类型，选择合适的类型可以节省内存。

```python
# 常见数据类型
arr_int8 = np.array([1, 2, 3], dtype=np.int8)    # 8位整数（-128 到 127）
arr_int32 = np.array([1, 2, 3], dtype=np.int32)  # 32位整数
arr_int64 = np.array([1, 2, 3], dtype=np.int64)  # 64位整数

arr_float32 = np.array([1.0, 2.0], dtype=np.float32)  # 32位浮点数
arr_float64 = np.array([1.0, 2.0], dtype=np.float64)  # 64位浮点数

arr_bool = np.array([True, False, True], dtype=bool)  # 布尔类型

arr_str = np.array(['a', 'b', 'c'], dtype=str)  # 字符串类型

# 查看数据类型
print(arr_int8.dtype)  # 输出: int8

# 类型转换
arr_float = np.array([1.7, 2.3, 3.9])
arr_int = arr_float.astype(int)  # 转换为整数（会截断小数部分）
print(arr_int)  # 输出: [1 2 3]

# 字符串转数字
arr_str_num = np.array(['1', '2', '3'])
arr_num = arr_str_num.astype(int)
print(arr_num)  # 输出: [1 2 3]
```

**常用数据类型表**：

| 类型    | 类型代码 | 说明           |
| ------- | -------- | -------------- |
| int8    | i1       | 有符号8位整数  |
| int16   | i2       | 有符号16位整数 |
| int32   | i4       | 有符号32位整数 |
| int64   | i8       | 有符号64位整数 |
| uint8   | u1       | 无符号8位整数  |
| float32 | f4       | 32位浮点数     |
| float64 | f8       | 64位浮点数     |
| bool    | ?        | 布尔类型       |

---

## 第四章：数组形状变换

### 4.1 reshape：改变数组形状

`reshape` 可以在不改变数据的情况下改变数组的形状。

```python
# 一维转二维
arr = np.arange(12)  # [0, 1, 2, ..., 11]
print(arr)

arr_2d = arr.reshape(3, 4)  # 转换为 3 行 4 列
print(arr_2d)
# 输出:
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# 二维转三维
arr_3d = arr.reshape(2, 2, 3)  # 转换为 2×2×3
print(arr_3d)
# 输出:
# [[[ 0  1  2]
#   [ 3  4  5]]
#
#  [[ 6  7  8]
#   [ 9 10 11]]]

# 自动计算维度：使用 -1
arr_auto = arr.reshape(3, -1)  # 3 行，列数自动计算（12÷3=4）
print(arr_auto)
# 输出:
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

arr_auto2 = arr.reshape(-1, 2)  # 2 列，行数自动计算（12÷2=6）
print(arr_auto2.shape)  # 输出: (6, 2)
```

**重要提示**：
- `reshape` 返回的是数组的视图（view），不是复制
- 元素总数必须保持不变（例如 12 个元素可以变成 3×4 或 2×6，但不能变成 3×5）

### 4.2 flatten 与 ravel：展平数组

将多维数组转换为一维数组。

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

# flatten：返回数组的副本
arr_flat = arr.flatten()
print(arr_flat)  # 输出: [1 2 3 4 5 6]
arr_flat[0] = 999
print(arr)  # 原数组不变
# 输出:
# [[1 2 3]
#  [4 5 6]]

# ravel：返回数组的视图（通常更快）
arr_ravel = arr.ravel()
print(arr_ravel)  # 输出: [1 2 3 4 5 6]
arr_ravel[0] = 999
print(arr)  # 原数组会改变！
# 输出:
# [[999   2   3]
#  [  4   5   6]]
```

**flatten 与 ravel 的区别**：
- `flatten()`：创建副本，修改不影响原数组
- `ravel()`：创建视图，修改会影响原数组（性能更好）

### 4.3 转置：transpose 与 T

交换数组的行和列。

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr.shape)  # 输出: (2, 3)

# 使用 .T 属性
arr_t = arr.T
print(arr_t)
# 输出:
# [[1 4]
#  [2 5]
#  [3 6]]
print(arr_t.shape)  # 输出: (3, 2)

# 使用 transpose() 方法
arr_t2 = arr.transpose()
print(arr_t2)  # 结果相同

# 多维数组的转置（指定轴的顺序）
arr_3d = np.arange(24).reshape(2, 3, 4)
print(arr_3d.shape)  # 输出: (2, 3, 4)

# 交换第 0 轴和第 2 轴
arr_3d_t = arr_3d.transpose(2, 1, 0)
print(arr_3d_t.shape)  # 输出: (4, 3, 2)
```

**转置的应用场景**：在矩阵运算、机器学习中经常需要转置数据。

### 4.4 增加维度：newaxis 与 expand_dims

```python
arr = np.array([1, 2, 3, 4])
print(arr.shape)  # 输出: (4,)

# 使用 newaxis 增加维度
arr_row = arr[np.newaxis, :]  # 增加行维度，变成行向量
print(arr_row.shape)  # 输出: (1, 4)
print(arr_row)  # 输出: [[1 2 3 4]]

arr_col = arr[:, np.newaxis]  # 增加列维度，变成列向量
print(arr_col.shape)  # 输出: (4, 1)
print(arr_col)
# 输出:
# [[1]
#  [2]
#  [3]
#  [4]]

# 使用 expand_dims
arr_expand = np.expand_dims(arr, axis=0)  # 在第 0 轴增加维度
print(arr_expand.shape)  # 输出: (1, 4)

arr_expand2 = np.expand_dims(arr, axis=1)  # 在第 1 轴增加维度
print(arr_expand2.shape)  # 输出: (4, 1)
```

### 4.5 挤压维度：squeeze

删除长度为 1 的维度。

```python
arr = np.array([[[1, 2, 3]]])
print(arr.shape)  # 输出: (1, 1, 3)

arr_squeezed = arr.squeeze()
print(arr_squeezed.shape)  # 输出: (3,)
print(arr_squeezed)  # 输出: [1 2 3]

# 指定要删除的轴
arr = np.array([[[1], [2], [3]]])
print(arr.shape)  # 输出: (1, 3, 1)

arr_squeezed = arr.squeeze(axis=0)  # 删除第 0 轴
print(arr_squeezed.shape)  # 输出: (3, 1)

arr_squeezed = arr.squeeze(axis=2)  # 删除第 2 轴
print(arr_squeezed.shape)  # 输出: (1, 3)
```

### 4.6 拼接数组

#### 4.6.1 concatenate：沿指定轴拼接

```python
arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5, 6], [7, 8]])

# 沿第 0 轴（行）拼接（垂直拼接）
result = np.concatenate([arr1, arr2], axis=0)
print(result)
# 输出:
# [[1 2]
#  [3 4]
#  [5 6]
#  [7 8]]

# 沿第 1 轴（列）拼接（水平拼接）
result = np.concatenate([arr1, arr2], axis=1)
print(result)
# 输出:
# [[1 2 5 6]
#  [3 4 7 8]]
```

#### 4.6.2 vstack 和 hstack：垂直和水平堆叠

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])

# 垂直堆叠（相当于 axis=0）
v_stack = np.vstack([arr1, arr2])
print(v_stack)
# 输出:
# [[1 2 3]
#  [4 5 6]]

# 水平堆叠（相当于 axis=1）
h_stack = np.hstack([arr1, arr2])
print(h_stack)  # 输出: [1 2 3 4 5 6]
```

#### 4.6.3 stack：沿新轴堆叠

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])

# 沿第 0 轴堆叠
result = np.stack([arr1, arr2], axis=0)
print(result)
# 输出:
# [[1 2 3]
#  [4 5 6]]

# 沿第 1 轴堆叠
result = np.stack([arr1, arr2], axis=1)
print(result)
# 输出:
# [[1 4]
#  [2 5]
#  [3 6]]
```

### 4.7 分割数组

```python
arr = np.arange(12).reshape(3, 4)
print(arr)
# 输出:
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# 垂直分割（按行）
v_split = np.vsplit(arr, 3)  # 分成 3 份
print(v_split[0])  # 输出: [[0 1 2 3]]
print(v_split[1])  # 输出: [[4 5 6 7]]
print(v_split[2])  # 输出: [[ 8  9 10 11]]

# 水平分割（按列）
h_split = np.hsplit(arr, 2)  # 分成 2 份
print(h_split[0])
# 输出:
# [[0 1]
#  [4 5]
#  [8 9]]
print(h_split[1])
# 输出:
# [[ 2  3]
#  [ 6  7]
#  [10 11]]

# 使用 split 指定分割位置
result = np.split(arr, [1, 3], axis=1)  # 在索引 1 和 3 处分割
print(len(result))  # 输出: 3（分成3部分）
print(result[0])  # 列 0
print(result[1])  # 列 1-2
print(result[2])  # 列 3
```

---

## 第五章：数组运算与广播

### 5.1 数组的基本运算

NumPy 数组支持逐元素运算（element-wise operations）。

#### 5.1.1 算术运算

```python
arr1 = np.array([1, 2, 3, 4])
arr2 = np.array([10, 20, 30, 40])

# 加法
print(arr1 + arr2)  # 输出: [11 22 33 44]

# 减法
print(arr2 - arr1)  # 输出: [ 9 18 27 36]

# 乘法（逐元素相乘，不是矩阵乘法）
print(arr1 * arr2)  # 输出: [ 10  40  90 160]

# 除法
print(arr2 / arr1)  # 输出: [10. 10. 10. 10.]

# 幂运算
print(arr1 ** 2)  # 输出: [ 1  4  9 16]

# 取余
print(arr2 % 3)  # 输出: [1 2 0 1]

# 与标量运算
print(arr1 + 10)  # 输出: [11 12 13 14]
print(arr1 * 2)   # 输出: [2 4 6 8]
```

#### 5.1.2 比较运算

```python
arr1 = np.array([1, 2, 3, 4, 5])
arr2 = np.array([5, 4, 3, 2, 1])

# 逐元素比较，返回布尔数组
print(arr1 > arr2)   # 输出: [False False False  True  True]
print(arr1 == arr2)  # 输出: [False False  True False False]
print(arr1 < 3)      # 输出: [ True  True False False False]

# 使用比较结果进行筛选
print(arr1[arr1 > 3])  # 输出: [4 5]
```

#### 5.1.3 逻辑运算

```python
arr = np.array([True, False, True, False])
arr2 = np.array([True, True, False, False])

# 逻辑与
print(arr & arr2)  # 输出: [ True False False False]

# 逻辑或
print(arr | arr2)  # 输出: [ True  True  True False]

# 逻辑非
print(~arr)  # 输出: [False  True False  True]

# 应用示例：组合条件筛选
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
# 筛选大于 3 且小于 8 的元素
result = arr[(arr > 3) & (arr < 8)]
print(result)  # 输出: [4 5 6 7]
```

### 5.2 通用函数（ufunc）

NumPy 提供了大量的数学函数，这些函数对数组中的每个元素进行操作。

```python
arr = np.array([1, 4, 9, 16, 25])

# 平方根
print(np.sqrt(arr))  # 输出: [1. 2. 3. 4. 5.]

# 指数
print(np.exp(arr))  # 输出: [2.71828183e+00 5.45981500e+01 8.10308393e+03 ...]

# 对数
print(np.log(arr))   # 自然对数
print(np.log10(arr)) # 以10为底的对数

# 三角函数
arr_angle = np.array([0, np.pi/2, np.pi])
print(np.sin(arr_angle))  # 输出: [0.000000e+00 1.000000e+00 1.224647e-16]
print(np.cos(arr_angle))  # 输出: [ 1.000000e+00  6.123234e-17 -1.000000e+00]

# 四舍五入
arr_float = np.array([1.2, 2.7, 3.5, 4.8])
print(np.round(arr_float))  # 输出: [1. 3. 4. 5.]
print(np.floor(arr_float))  # 向下取整: [1. 2. 3. 4.]
print(np.ceil(arr_float))   # 向上取整: [2. 3. 4. 5.]

# 绝对值
arr_neg = np.array([-1, -2, 3, -4])
print(np.abs(arr_neg))  # 输出: [1 2 3 4]

# 符号函数
print(np.sign(arr_neg))  # 输出: [-1 -1  1 -1]
```

### 5.3 聚合函数

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

# 求和
print(np.sum(arr))  # 所有元素求和: 21
print(np.sum(arr, axis=0))  # 按列求和: [5 7 9]
print(np.sum(arr, axis=1))  # 按行求和: [ 6 15]

# 平均值
print(np.mean(arr))  # 输出: 3.5
print(np.mean(arr, axis=0))  # 按列: [2.5 3.5 4.5]

# 标准差和方差
print(np.std(arr))   # 标准差
print(np.var(arr))   # 方差

# 最大值和最小值
print(np.max(arr))   # 输出: 6
print(np.min(arr))   # 输出: 1
print(np.max(arr, axis=0))  # 按列: [4 5 6]

# 最大值和最小值的索引
print(np.argmax(arr))  # 输出: 5（展平后的索引）
print(np.argmin(arr))  # 输出: 0

# 累加和累乘
print(np.cumsum(arr))  # 累加: [ 1  3  6 10 15 21]
print(np.cumprod(arr)) # 累乘: [  1   2   6  24 120 720]

# 中位数
arr = np.array([1, 3, 5, 7, 9])
print(np.median(arr))  # 输出: 5.0

# 百分位数
print(np.percentile(arr, 25))  # 25% 分位数: 3.0
print(np.percentile(arr, 75))  # 75% 分位数: 7.0
```

**axis 参数理解**：
- `axis=0`：沿着行的方向（垂直方向），对每列进行操作
- `axis=1`：沿着列的方向（水平方向），对每行进行操作
- 不指定 axis：对整个数组进行操作

```python
# axis 参数理解（续）
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

# axis=0：沿着行方向操作（对每一列）
print(np.sum(arr, axis=0))
# 计算过程：
# 列1: 1+4=5
# 列2: 2+5=7
# 列3: 3+6=9
# 输出: [5 7 9]

# axis=1：沿着列方向操作（对每一行）
print(np.sum(arr, axis=1))
# 计算过程：
# 行1: 1+2+3=6
# 行2: 4+5+6=15
# 输出: [ 6 15]

# 三维数组的 axis 理解
arr_3d = np.arange(24).reshape(2, 3, 4)
print(arr_3d.shape)  # (2, 3, 4)

print(np.sum(arr_3d, axis=0).shape)  # (3, 4) - 沿第0轴（最外层）
print(np.sum(arr_3d, axis=1).shape)  # (2, 4) - 沿第1轴（中间层）
print(np.sum(arr_3d, axis=2).shape)  # (2, 3) - 沿第2轴（最内层）
```

**记忆技巧**：axis 参数指定的是要"消失"的维度。例如，对于形状 (2, 3, 4) 的数组：
- `axis=0` 操作后，第 0 维消失，结果形状变为 (3, 4)
- `axis=1` 操作后，第 1 维消失，结果形状变为 (2, 4)

### 5.4 广播机制（Broadcasting）

广播是 NumPy 最强大的特性之一，允许不同形状的数组进行运算。

#### 5.4.1 广播的基本概念

```python
# 标量与数组的广播
arr = np.array([1, 2, 3, 4])
result = arr + 10
print(result)  # 输出: [11 12 13 14]
# 广播过程：10 被扩展为 [10, 10, 10, 10]，然后逐元素相加

# 一维与二维的广播
arr_2d = np.array([[1, 2, 3],
                   [4, 5, 6]])
arr_1d = np.array([10, 20, 30])

result = arr_2d + arr_1d
print(result)
# 输出:
# [[11 22 33]
#  [14 25 36]]
# 广播过程：arr_1d 被扩展为与 arr_2d 相同的形状
# [[10, 20, 30],
#  [10, 20, 30]]
```

#### 5.4.2 广播规则

NumPy 在两个数组间执行广播时，会按照以下规则比较它们的形状：

**规则 1**：如果两个数组的维度数不同，形状较小的数组会在前面补 1，直到两者维度数相同。

**规则 2**：如果两个数组在某个维度上的大小相同，或者其中一个大小为 1，则认为它们在该维度上兼容。

**规则 3**：如果两个数组在所有维度上都兼容，则可以进行广播；否则会报错。

**规则 4**：广播后，每个维度的大小取两个数组中该维度的最大值。

**规则 5**：在任何一个维度上大小为 1 的数组，在该维度上会被拉伸以匹配另一个数组的大小。

```python
# 示例 1：一维数组与二维数组
arr_2d = np.array([[1, 2, 3],    # 形状: (2, 3)
                   [4, 5, 6]])
arr_1d = np.array([10, 20, 30])  # 形状: (3,)

# 广播过程：
# 1. arr_1d 的形状 (3,) 补成 (1, 3)
# 2. (2, 3) 和 (1, 3) 比较：
#    - 第0维：2 和 1 兼容（1会被拉伸）
#    - 第1维：3 和 3 兼容
# 3. 结果形状：(2, 3)

result = arr_2d + arr_1d
print(result)
# 输出:
# [[11 22 33]
#  [14 25 36]]

# 示例 2：列向量与行向量
col = np.array([[1],     # 形状: (3, 1)
                [2],
                [3]])
row = np.array([10, 20, 30])  # 形状: (3,) -> (1, 3)

result = col + row
print(result)
# 输出:
# [[11 21 31]
#  [12 22 32]
#  [13 23 33]]
# 广播过程：
# col: (3, 1) -> (3, 3)
# row: (1, 3) -> (3, 3)

# 示例 3：三维数组的广播
arr_3d = np.ones((3, 4, 5))       # 形状: (3, 4, 5)
arr_2d = np.ones((4, 5))          # 形状: (4, 5) -> (1, 4, 5)
arr_1d = np.ones((5,))            # 形状: (5,) -> (1, 1, 5)

result1 = arr_3d + arr_2d  # 可以广播
print(result1.shape)  # 输出: (3, 4, 5)

result2 = arr_3d + arr_1d  # 可以广播
print(result2.shape)  # 输出: (3, 4, 5)
```

#### 5.4.3 广播失败的情况

```python
# 不兼容的形状
arr1 = np.ones((3, 4))  # 形状: (3, 4)
arr2 = np.ones((3,))    # 形状: (3,) -> (1, 3)

# 尝试相加会报错
try:
    result = arr1 + arr2
except ValueError as e:
    print(f"错误: {e}")
    # 输出: 错误: operands could not be broadcast together with shapes (3,4) (3,)

# 分析：
# arr1: (3, 4)
# arr2: (1, 3)
# 第1维：4 和 3 不兼容（都不是1，且不相等）

# 解决方法：调整形状
arr2_reshaped = arr2[:, np.newaxis]  # 形状变为 (3, 1)
result = arr1 + arr2_reshaped  # 现在可以广播了
print(result.shape)  # 输出: (3, 4)
```

#### 5.4.4 广播的实际应用

```python
# 应用 1：数据标准化（减均值，除以标准差）
data = np.array([[1, 2, 3],
                 [4, 5, 6],
                 [7, 8, 9]])

# 计算每列的均值和标准差
mean = np.mean(data, axis=0)  # 形状: (3,)
std = np.std(data, axis=0)    # 形状: (3,)

# 标准化（利用广播）
normalized = (data - mean) / std
print(normalized)
# 输出:
# [[-1.22474487 -1.22474487 -1.22474487]
#  [ 0.          0.          0.        ]
#  [ 1.22474487  1.22474487  1.22474487]]

# 应用 2：计算距离矩阵
points = np.array([[1, 2],
                   [3, 4],
                   [5, 6]])

# 计算所有点对之间的欧几里得距离
# 使用广播技巧
points_expanded = points[:, np.newaxis, :]  # 形状: (3, 1, 2)
diff = points_expanded - points             # 广播到 (3, 3, 2)
distances = np.sqrt(np.sum(diff**2, axis=2))
print(distances)
# 输出:
# [[0.         2.82842712 5.65685425]
#  [2.82842712 0.         2.82842712]
#  [5.65685425 2.82842712 0.        ]]

# 应用 3：创建网格
x = np.array([1, 2, 3])
y = np.array([10, 20, 30, 40])

# 使用广播创建网格坐标
X = x[np.newaxis, :]  # 形状: (1, 3)
Y = y[:, np.newaxis]  # 形状: (4, 1)

# 计算所有组合的和
grid_sum = X + Y
print(grid_sum)
# 输出:
# [[11 12 13]
#  [21 22 23]
#  [31 32 33]
#  [41 42 43]]

# 应用 4：批量计算
# 假设有多个样本，每个样本有多个特征
samples = np.random.rand(100, 10)  # 100个样本，每个10个特征
weights = np.random.rand(10)       # 10个权重

# 计算每个样本的加权和（利用广播）
weighted_sum = np.sum(samples * weights, axis=1)
print(weighted_sum.shape)  # 输出: (100,)
```

#### 5.4.5 广播技巧总结

```python
# 技巧 1：使用 newaxis 添加维度
arr = np.array([1, 2, 3])
print(arr.shape)  # (3,)

# 转换为行向量
row_vector = arr[np.newaxis, :]
print(row_vector.shape)  # (1, 3)

# 转换为列向量
col_vector = arr[:, np.newaxis]
print(col_vector.shape)  # (3, 1)

# 技巧 2：使用 reshape 调整形状
arr = np.array([1, 2, 3, 4])
# 转换为 (2, 2) 进行广播
arr_2d = arr.reshape(2, 2)

# 技巧 3：使用 np.expand_dims
arr = np.array([1, 2, 3])
expanded = np.expand_dims(arr, axis=0)  # 在第0维添加维度
print(expanded.shape)  # (1, 3)

# 技巧 4：验证广播兼容性的函数
def can_broadcast(shape1, shape2):
    """检查两个形状是否可以广播"""
    # 反向遍历维度
    for s1, s2 in zip(reversed(shape1), reversed(shape2)):
        if s1 != s2 and s1 != 1 and s2 != 1:
            return False
    return True

print(can_broadcast((3, 4), (4,)))    # True
print(can_broadcast((3, 4), (3,)))    # False
print(can_broadcast((3, 1, 5), (5,))) # True
```

### 5.5 矩阵运算

虽然逐元素运算很有用，但有时我们需要真正的矩阵运算。

```python
# 矩阵乘法（点积）
arr1 = np.array([[1, 2],
                 [3, 4]])
arr2 = np.array([[5, 6],
                 [7, 8]])

# 方法 1：使用 @ 运算符（推荐，Python 3.5+）
result = arr1 @ arr2
print(result)
# 输出:
# [[19 22]
#  [43 50]]
# 计算过程：
# [1*5+2*7, 1*6+2*8]   [19, 22]
# [3*5+4*7, 3*6+4*8] = [43, 50]

# 方法 2：使用 np.dot()
result = np.dot(arr1, arr2)
print(result)  # 结果相同

# 方法 3：使用 matmul()
result = np.matmul(arr1, arr2)
print(result)  # 结果相同

# 注意：* 是逐元素相乘，不是矩阵乘法
result_element = arr1 * arr2
print(result_element)
# 输出:
# [[ 5 12]
#  [21 32]]

# 向量的点积
vec1 = np.array([1, 2, 3])
vec2 = np.array([4, 5, 6])
dot_product = np.dot(vec1, vec2)
print(dot_product)  # 输出: 32 (1*4 + 2*5 + 3*6)

# 矩阵与向量的乘法
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])
vector = np.array([1, 2, 3])
result = matrix @ vector
print(result)  # 输出: [14 32]

# 批量矩阵乘法（3D 数组）
batch_matrices = np.random.rand(10, 3, 4)  # 10个 3×4 矩阵
batch_vectors = np.random.rand(10, 4)       # 10个长度为4的向量

# 使用 @ 进行批量乘法
result = batch_matrices @ batch_vectors[:, :, np.newaxis]
print(result.shape)  # 输出: (10, 3, 1)
```

---

## 第六章：随机数生成

NumPy 提供了强大的随机数生成功能，这在模拟、测试和机器学习中非常重要。

### 6.1 旧版随机数生成（np.random）

```python
# 设置随机种子（保证结果可重复）
np.random.seed(42)

# 生成 [0, 1) 之间的随机浮点数
rand_float = np.random.random(5)
print(rand_float)
# 输出: [0.37454012 0.95071431 0.73199394 0.59865848 0.15601864]

# 再次运行会得到不同结果，除非重新设置种子
rand_float2 = np.random.random(5)
print(rand_float2)  # 不同的值

# 重新设置种子，结果会重复
np.random.seed(42)
rand_float3 = np.random.random(5)
print(rand_float3)  # 与 rand_float 相同

# 生成指定形状的随机数组
rand_2d = np.random.random((3, 4))
print(rand_2d)
# 输出 3×4 的随机数组

# 生成 [0, 1) 之间的随机数（与 random() 相同）
rand_uniform = np.random.rand(2, 3)
print(rand_uniform)

# 生成标准正态分布（均值0，标准差1）
rand_normal = np.random.randn(1000)
print(f"均值: {np.mean(rand_normal):.2f}")  # 接近 0
print(f"标准差: {np.std(rand_normal):.2f}")  # 接近 1

# 生成指定范围的随机整数
rand_int = np.random.randint(1, 10, size=5)  # [1, 10) 之间的整数
print(rand_int)  # 例如: [3 8 2 6 9]

rand_int_2d = np.random.randint(0, 100, size=(3, 4))
print(rand_int_2d)

# 生成均匀分布的随机数
uniform = np.random.uniform(5, 10, size=5)  # [5, 10) 之间
print(uniform)

# 生成正态分布的随机数
normal = np.random.normal(loc=50, scale=10, size=1000)  # 均值50，标准差10
print(f"均值: {np.mean(normal):.2f}")
print(f"标准差: {np.std(normal):.2f}")

# 从数组中随机选择
arr = np.array([1, 2, 3, 4, 5])
choice = np.random.choice(arr, size=3)  # 随机选3个（可能重复）
print(choice)

choice_no_replace = np.random.choice(arr, size=3, replace=False)  # 不重复
print(choice_no_replace)

# 按概率随机选择
choice_prob = np.random.choice([1, 2, 3], size=10, p=[0.6, 0.3, 0.1])
print(choice_prob)  # 1出现的概率最高

# 随机打乱数组
arr = np.arange(10)
np.random.shuffle(arr)  # 原地打乱
print(arr)

# 返回打乱后的副本（不改变原数组）
arr = np.arange(10)
shuffled = np.random.permutation(arr)
print(shuffled)
print(arr)  # 原数组不变
```

### 6.2 新版随机数生成器（推荐）

从 NumPy 1.17 开始，推荐使用新的随机数生成器 API，它更快、更灵活。

```python
# 创建随机数生成器
rng = np.random.default_rng(seed=42)

# 生成随机数（与旧版类似的功能）
rand = rng.random(5)
print(rand)

# 生成随机整数
rand_int = rng.integers(0, 10, size=5)  # 注意是 integers，不是 randint
print(rand_int)

# 生成正态分布
normal = rng.normal(loc=0, scale=1, size=1000)
print(f"均值: {np.mean(normal):.2f}")

# 从数组中选择
arr = np.arange(10)
choice = rng.choice(arr, size=5, replace=False)
print(choice)

# 打乱数组
arr = np.arange(10)
rng.shuffle(arr)
print(arr)

# 多种分布
# 泊松分布
poisson = rng.poisson(lam=5, size=1000)
print(f"泊松分布均值: {np.mean(poisson):.2f}")

# 二项分布
binomial = rng.binomial(n=10, p=0.5, size=1000)
print(f"二项分布均值: {np.mean(binomial):.2f}")

# 指数分布
exponential = rng.exponential(scale=2, size=1000)
print(f"指数分布均值: {np.mean(exponential):.2f}")
```

### 6.3 常用概率分布

```python
rng = np.random.default_rng(seed=42)

# 1. 均匀分布 Uniform
# 所有值出现的概率相等
uniform = rng.uniform(low=0, high=10, size=1000)
print(f"均匀分布 - 最小值: {np.min(uniform):.2f}, 最大值: {np.max(uniform):.2f}")

# 2. 正态分布（高斯分布）Normal/Gaussian
# 钟形曲线，大部分值集中在均值附近
normal = rng.normal(loc=100, scale=15, size=1000)  # 均值100，标准差15
print(f"正态分布 - 均值: {np.mean(normal):.2f}, 标准差: {np.std(normal):.2f}")

# 3. 二项分布 Binomial
# n 次独立试验中成功 k 次的概率
# 例如：抛10次硬币，正面朝上的次数
binomial = rng.binomial(n=10, p=0.5, size=1000)
print(f"二项分布 - 均值: {np.mean(binomial):.2f}")

# 4. 泊松分布 Poisson
# 单位时间内事件发生的次数
# 例如：每小时到达的顾客数
poisson = rng.poisson(lam=3, size=1000)  # lambda=3（平均值）
print(f"泊松分布 - 均值: {np.mean(poisson):.2f}")

# 5. 指数分布 Exponential
# 事件之间的等待时间
# 例如：顾客到达之间的时间间隔
exponential = rng.exponential(scale=2, size=1000)
print(f"指数分布 - 均值: {np.mean(exponential):.2f}")

# 6. Beta 分布
# 值在 [0, 1] 之间，形状由两个参数控制
beta = rng.beta(a=2, b=5, size=1000)
print(f"Beta分布 - 均值: {np.mean(beta):.2f}")

# 7. Gamma 分布
gamma = rng.gamma(shape=2, scale=2, size=1000)
print(f"Gamma分布 - 均值: {np.mean(gamma):.2f}")

# 8. 卡方分布 Chi-square
chisquare = rng.chisquare(df=5, size=1000)  # 自由度为5
print(f"卡方分布 - 均值: {np.mean(chisquare):.2f}")
```

### 6.4 随机数的实际应用

```python
rng = np.random.default_rng(seed=42)

# 应用 1：生成模拟数据
# 模拟学生成绩（正态分布，均值75，标准差10）
scores = rng.normal(loc=75, scale=10, size=100)
scores = np.clip(scores, 0, 100)  # 限制在 [0, 100] 范围
print(f"平均分: {np.mean(scores):.2f}")
print(f"及格率: {np.sum(scores >= 60) / len(scores) * 100:.1f}%")

# 应用 2：数据增强（机器学习中常用）
# 给图像添加随机噪声
image = np.ones((100, 100)) * 128  # 灰度图像
noise = rng.normal(loc=0, scale=20, size=image.shape)
noisy_image = image + noise
noisy_image = np.clip(noisy_image, 0, 255)  # 限制在有效范围

# 应用 3：随机采样
# 从大数据集中随机抽取样本
data = np.arange(10000)
sample_indices = rng.choice(len(data), size=100, replace=False)
sample = data[sample_indices]
print(f"采样了 {len(sample)} 个数据点")

# 应用 4：蒙特卡洛模拟
# 估算圆周率
n_samples = 1000000
x = rng.uniform(-1, 1, n_samples)
y = rng.uniform(-1, 1, n_samples)
inside_circle = (x**2 + y**2) <= 1
pi_estimate = 4 * np.sum(inside_circle) / n_samples
print(f"π 的估计值: {pi_estimate:.4f}")

# 应用 5：生成训练/测试集分割
# 随机打乱索引并分割
data_size = 1000
indices = np.arange(data_size)
rng.shuffle(indices)

train_size = int(0.8 * data_size)
train_indices = indices[:train_size]
test_indices = indices[train_size:]
print(f"训练集大小: {len(train_indices)}, 测试集大小: {len(test_indices)}")
```

---

## 第七章：索引与切片

### 7.1 一维数组的索引与切片

```python
arr = np.array([10, 20, 30, 40, 50, 60, 70, 80, 90])

# 基本索引（从0开始）
print(arr[0])   # 输出: 10（第一个元素）
print(arr[3])   # 输出: 40（第四个元素）
print(arr[-1])  # 输出: 90（最后一个元素）
print(arr[-2])  # 输出: 80（倒数第二个元素）

# 基本切片 [start:stop:step]
print(arr[2:5])   # 输出: [30 40 50]（索引2到4）
print(arr[:5])    # 输出: [10 20 30 40 50]（从开始到索引4）
print(arr[5:])    # 输出: [60 70 80 90]（从索引5到结束）
print(arr[:])     # 输出: 整个数组（复制）

# 带步长的切片
print(arr[::2])   # 输出: [10 30 50 70 90]（每隔一个元素）
print(arr[1::2])  # 输出: [20 40 60 80]（从索引1开始，每隔一个）
print(arr[::-1])  # 输出: [90 80 70 60 50 40 30 20 10]（反转数组）
print(arr[7:2:-1])  # 输出: [80 70 60 50 40 30]（从索引7到3，倒序）

# 修改元素
arr[0] = 100
print(arr[0])  # 输出: 100

# 切片赋值
arr[0:3] = [11, 22, 33]
print(arr)  # 前三个元素被修改

# 注意：切片返回的是视图（view），不是副本
arr_slice = arr[2:5]
arr_slice[0] = 999
print(arr)  # 原数组也被修改了！

# 如果需要副本，使用 copy()
arr_copy = arr[2:5].copy()
arr_copy[0] = 777
print(arr)  # 原数组不受影响
```

### 7.2 二维数组的索引与切片

```python
arr_2d = np.array([[1, 2, 3, 4],
                   [5, 6, 7, 8],
                   [9, 10, 11, 12]])

# 基本索引
print(arr_2d[0, 0])  # 输出: 1（第1行第1列）
print(arr_2d[1, 2])  # 输出: 7（第2行第3列）
print(arr_2d[-1, -1])  # 输出: 12（最后一行最后一列）

# 获取整行
print(arr_2d[0])  # 输出: [1 2 3 4]（第一行）
print(arr_2d[1])  # 输出: [5 6 7 8]（第二行）

# 使用切片获取多行
print(arr_2d[0:2])  # 输出前两行
# [[1 2 3 4]
#  [5 6 7 8]]

# 获取整列（使用 : 表示所有行）
print(arr_2d[:, 0])  # 输出: [ 1  5  9]（第一列）
print(arr_2d[:, 2])  # 输出: [ 3  7 11]（第三列）

# 使用切片获取多列
print(arr_2d[:, 1:3])  # 输出第2和第3列
# [[ 2  3]
#  [ 6  7]
#  [10 11]]

# 同时切片行和列
print(arr_2d[0:2, 1:3])  # 前两行，第2和第3列
# [[2 3]
#  [6 7]]

# 步长切片
print(arr_2d[::2, ::2])  # 每隔一行，每隔一列
# [[ 1  3]
#  [ 9 11]]

# 反转行
print(arr_2d[::-1, :])
# [[ 9 10 11 12]
#  [ 5  6  7  8]
#  [ 1  2  3  4]]

# 反转列
print(arr_2d[:, ::-1])
# [[ 4  3  2  1]
#  [ 8  7  6  5]
#  [12 11 10  9]]

# 修改子区域
arr_2d[0:2, 0:2] = 0
print(arr_2d)
# [[ 0  0  3  4]
#  [ 0  0  7  8]
#  [ 9 10 11 12]]
```

### 7.3 布尔索引

布尔索引允许我们使用布尔数组来选择满足特定条件的元素。

```python
# 创建示例数组
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# 创建布尔掩码（mask）
mask = arr > 5
print(mask)
# 输出: [False False False False False  True  True  True  True  True]

# 使用布尔掩码筛选元素
result = arr[mask]
print(result)  # 输出: [ 6  7  8  9 10]

# 简写方式（一步完成）
result = arr[arr > 5]
print(result)  # 输出: [ 6  7  8  9 10]

# 多个条件组合
# 与运算 &
result = arr[(arr > 3) & (arr < 8)]
print(result)  # 输出: [4 5 6 7]

# 或运算 |
result = arr[(arr < 3) | (arr > 8)]
print(result)  # 输出: [ 1  2  9 10]

# 非运算 ~
result = arr[~(arr > 5)]
print(result)  # 输出: [1 2 3 4 5]

# 注意：必须使用 &、| 和 ~，不能使用 and、or 和 not

# 二维数组的布尔索引
arr_2d = np.array([[1, 2, 3, 4],
                   [5, 6, 7, 8],
                   [9, 10, 11, 12]])

# 筛选大于5的元素
result = arr_2d[arr_2d > 5]
print(result)  # 输出: [ 6  7  8  9 10 11 12]（一维数组）

# 按行筛选
# 筛选第一列大于5的行
mask = arr_2d[:, 0] > 5
print(mask)  # 输出: [False False  True]
result = arr_2d[mask]
print(result)
# 输出:
# [[ 9 10 11 12]]

# 按条件修改元素
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
arr[arr > 5] = 0  # 将大于5的元素设为0
print(arr)  # 输出: [1 2 3 4 5 0 0 0 0 0]

# 按条件进行运算
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
arr[arr % 2 == 0] *= 10  # 将偶数乘以10
print(arr)  # 输出: [ 1 20  3 40  5 60  7 80  9 100]
```

#### 7.3.1 where 函数

`np.where()` 是一个非常有用的函数，可以根据条件选择元素。

```python
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# 语法：np.where(条件, 满足条件时的值, 不满足条件时的值)
result = np.where(arr > 5, 'big', 'small')
print(result)
# 输出: ['small' 'small' 'small' 'small' 'small' 'big' 'big' 'big' 'big' 'big']

# 数值示例
result = np.where(arr > 5, arr, 0)  # 大于5保持，否则设为0
print(result)  # 输出: [ 0  0  0  0  0  6  7  8  9 10]

# 二维数组
arr_2d = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

result = np.where(arr_2d > 5, 1, 0)  # 大于5设为1，否则为0
print(result)
# 输出:
# [[0 0 0]
#  [0 0 1]
#  [1 1 1]]

# 只传入条件，返回满足条件的索引
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
indices = np.where(arr > 5)
print(indices)  # 输出: (array([5, 6, 7, 8, 9]),)
print(arr[indices])  # 输出: [ 6  7  8  9 10]

# 二维数组返回行和列索引
arr_2d = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])
rows, cols = np.where(arr_2d > 5)
print(f"行索引: {rows}")  # 输出: 行索引: [1 2 2 2]
print(f"列索引: {cols}")  # 输出: 列索引: [2 0 1 2]
print(arr_2d[rows, cols])  # 输出: [6 7 8 9]
```

#### 7.3.2 实用的布尔函数

```python
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# any：是否有任何元素满足条件
print(np.any(arr > 5))  # 输出: True（有元素大于5）
print(np.any(arr > 100))  # 输出: False（没有元素大于100）

# all：是否所有元素都满足条件
print(np.all(arr > 0))  # 输出: True（所有元素都大于0）
print(np.all(arr > 5))  # 输出: False（不是所有元素都大于5）

# 二维数组按轴操作
arr_2d = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

# 每列是否有大于5的元素
print(np.any(arr_2d > 5, axis=0))  # 输出: [False  True  True]

# 每行是否所有元素都大于3
print(np.all(arr_2d > 3, axis=1))  # 输出: [False  True  True]

# 统计满足条件的元素个数
arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
count = np.sum(arr > 5)  # 布尔值会被转换为0和1
print(f"大于5的元素有 {count} 个")  # 输出: 大于5的元素有 5 个

# 计算满足条件的元素百分比
percentage = np.mean(arr > 5) * 100
print(f"大于5的元素占 {percentage}%")  # 输出: 大于5的元素占 50.0%
```

### 7.4 花式索引（Fancy Indexing）

使用整数数组进行索引，可以以任意顺序选择元素。

```python
# 一维数组的花式索引
arr = np.array([10, 20, 30, 40, 50, 60, 70, 80, 90])

# 使用索引数组
indices = np.array([0, 2, 4, 6])
result = arr[indices]
print(result)  # 输出: [10 30 50 70]

# 可以重复索引
indices = np.array([0, 0, 2, 2, 4])
result = arr[indices]
print(result)  # 输出: [10 10 30 30 50]

# 可以改变顺序
indices = np.array([4, 2, 0])
result = arr[indices]
print(result)  # 输出: [50 30 10]

# 二维数组的花式索引
arr_2d = np.array([[1, 2, 3, 4],
                   [5, 6, 7, 8],
                   [9, 10, 11, 12],
                   [13, 14, 15, 16]])

# 选择特定的行
row_indices = np.array([0, 2, 3])
result = arr_2d[row_indices]
print(result)
# 输出:
# [[ 1  2  3  4]
#  [ 9 10 11 12]
#  [13 14 15 16]]

# 同时指定行和列索引
row_indices = np.array([0, 1, 2])
col_indices = np.array([0, 1, 2])
result = arr_2d[row_indices, col_indices]
print(result)  # 输出: [1 6 11]（对角线元素）

# 选择矩形区域（使用 ix_）
rows = np.array([0, 2])
cols = np.array([1, 3])
result = arr_2d[np.ix_(rows, cols)]
print(result)
# 输出:
# [[ 2  4]
#  [10 12]]

# 花式索引与切片结合
result = arr_2d[[0, 2], 1:3]  # 第0和第2行，第2和第3列
print(result)
# 输出:
# [[ 2  3]
#  [10 11]]

# 使用花式索引修改元素
arr = np.array([10, 20, 30, 40, 50])
indices = np.array([0, 2, 4])
arr[indices] = 0
print(arr)  # 输出: [ 0 20  0 40  0]

# 注意：花式索引返回的是副本，不是视图
arr = np.array([10, 20, 30, 40, 50])
indices = np.array([0, 2, 4])
arr_fancy = arr[indices]
arr_fancy[0] = 999
print(arr)  # 输出: [10 20 30 40 50]（原数组不变）
```

#### 7.4.1 花式索引的高级应用

```python
# 应用 1：重新排列数组
arr = np.arange(12).reshape(4, 3)
print("原数组:")
print(arr)
# [[ 0  1  2]
#  [ 3  4  5]
#  [ 6  7  8]
#  [ 9 10 11]]

# 反转行顺序
print("反转行:")
print(arr[[3, 2, 1, 0]])

# 应用 2：提取多个不连续的区域
arr = np.arange(20).reshape(4, 5)
# 提取第1、3行的第0、2、4列
rows = np.array([0, 2])
cols = np.array([0, 2, 4])
result = arr[np.ix_(rows, cols)]
print(result)
# [[ 0  2  4]
#  [10 12 14]]

# 应用 3：根据另一个数组的值进行索引
data = np.array([10, 20, 30, 40, 50])
labels = np.array([0, 2, 1, 2, 0])  # 标签

# 统计每个标签的数据
label_0 = data[labels == 0]
print(f"标签0的数据: {label_0}")  # 输出: [10 50]

# 应用 4：实现 one-hot 编码
labels = np.array([0, 2, 1, 2, 0])
n_classes = 3
one_hot = np.zeros((len(labels), n_classes))
one_hot[np.arange(len(labels)), labels] = 1
print(one_hot)
# [[1. 0. 0.]
#  [0. 0. 1.]
#  [0. 1. 0.]
#  [0. 0. 1.]
#  [1. 0. 0.]]
```

### 7.5 索引与切片总结

```python
# 1. 基本索引
arr = np.arange(10)
print(arr[5])  # 单个元素

# 2. 切片
print(arr[2:7])  # 连续的元素
print(arr[::2])  # 带步长

# 3. 布尔索引
print(arr[arr > 5])  # 满足条件的元素

# 4. 花式索引
print(arr[[0, 2, 4]])  # 指定位置的元素

# 5. 组合使用
arr_2d = np.arange(20).reshape(4, 5)

# 布尔索引 + 切片
mask = arr_2d[:, 0] > 5
print(arr_2d[mask, 1:3])

# 花式索引 + 切片
print(arr_2d[[0, 2], 1:4])

# 6. 视图 vs 副本
# 基本切片返回视图
view = arr[2:7]
view[0] = 999
print(arr)  # 原数组改变

# 布尔索引和花式索引返回副本
arr = np.arange(10)
copy = arr[arr > 5]
copy[0] = 999
print(arr)  # 原数组不变

# 显式创建副本
arr_copy = arr[2:7].copy()
arr_copy[0] = 777
print(arr)  # 原数组不变
```

### 7.6 实际应用示例

```python
# 示例 1：数据清洗 - 替换异常值
data = np.array([1, 2, 3, 100, 5, 6, -50, 8, 9, 10])

# 将小于0或大于50的值替换为中位数
median = np.median(data[(data >= 0) & (data <= 50)])
data[(data < 0) | (data > 50)] = median
print(data)  # 输出: [ 1.  2.  3.  6.  5.  6.  6.  8.  9. 10.]

# 示例 2：图像处理 - 阈值化
image = np.random.randint(0, 256, (5, 5))  # 模拟灰度图像
print("原图像:")
print(image)

# 二值化：大于128设为255，否则设为0
binary_image = np.where(image > 128, 255, 0)
print("二值化后:")
print(binary_image)

# 示例 3：数据分析 - 按条件统计
scores = np.array([85, 92, 78, 95, 88, 76, 90, 83, 79, 94])

# 统计各分数段的人数
excellent = np.sum(scores >= 90)  # 优秀
good = np.sum((scores >= 80) & (scores < 90))  # 良好
pass_count = np.sum((scores >= 60) & (scores < 80))  # 及格
fail = np.sum(scores < 60)  # 不及格

print(f"优秀: {excellent}人, 良好: {good}人, 及格: {pass_count}人, 不及格: {fail}人")

# 示例 4：时间序列 - 滑动窗口
data = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
window_size = 3

# 计算滑动窗口平均值
windows = []
for i in range(len(data) - window_size + 1):
    window = data[i:i+window_size]
    windows.append(np.mean(window))

print(np.array(windows))  # 输出: [2. 3. 4. 5. 6. 7. 8. 9.]

# 示例 5：机器学习 - 特征选择
# 模拟数据：100个样本，5个特征
X = np.random.randn(100, 5)
y = np.random.randint(0, 2, 100)

# 选择前3个特征
X_selected = X[:, [0, 1, 2]]
print(f"原特征形状: {X.shape}, 选择后: {X_selected.shape}")

# 选择满足条件的样本
# 假设我们只要第一个特征大于0的样本
mask = X[:, 0] > 0
X_filtered = X[mask]
y_filtered = y[mask]
print(f"过滤前: {X.shape}, 过滤后: {X_filtered.shape}")
```

---

## 第八章：综合练习

### 练习 1：创建和操作数组

```python
# 1. 创建一个 5×5 的数组，值为 1 到 25
arr = np.arange(1, 26).reshape(5, 5)
print("数组:")
print(arr)

# 2. 提取边界元素（第一行、最后一行、第一列、最后一列）
border = np.concatenate([
    arr[0, :],      # 第一行
    arr[-1, :],     # 最后一行
    arr[1:-1, 0],   # 第一列（去掉角）
    arr[1:-1, -1]   # 最后一列（去掉角）
])
print("边界元素:", border)

# 3. 将对角线元素设为 0
arr[np.arange(5), np.arange(5)] = 0  # 主对角线
arr[np.arange(5), np.arange(4, -1, -1)] = 0  # 副对角线
print("修改后:")
print(arr)
```

### 练习 2：统计分析

```python
# 生成模拟数据：100个学生，5门课程的成绩
np.random.seed(42)
scores = np.random.randint(60, 100, (100, 5))

# 1. 计算每门课程的平均分
course_means = np.mean(scores, axis=0)
print("每门课程的平均分:", course_means)

# 2. 计算每个学生的平均分
student_means = np.mean(scores, axis=1)
print("前10个学生的平均分:", student_means[:10])

# 3. 找出平均分最高的学生
best_student = np.argmax(student_means)
print(f"平均分最高的是第 {best_student+1} 名学生，平均分: {student_means[best_student]:.2f}")

# 4. 找出每门课程成绩最高的学生
for i in range(5):
    best_in_course = np.argmax(scores[:, i])
    print(f"第{i+1}门课程最高分: 第{best_in_course+1}名学生, {scores[best_in_course, i]}分")

# 5. 统计每门课程的及格率
pass_rates = np.mean(scores >= 80, axis=0) * 100
for i, rate in enumerate(pass_rates):
    print(f"第{i+1}门课程优秀率: {rate:.1f}%")
```

### 练习 3：图像处理模拟

```python
# 创建一个模拟图像（10×10 灰度图）
np.random.seed(42)
image = np.random.randint(0, 256, (10, 10))
print("原始图像:")
print(image)

# 1. 图像增强：对比度拉伸
min_val = np.min(image)
max_val = np.max(image)
enhanced = ((image - min_val) / (max_val - min_val) * 255).astype(int)
print("增强后:")
print(enhanced)

# 2. 边缘检测（简单差分）
edges_h = np.abs(np.diff(image, axis=1))  # 水平边缘
edges_v = np.abs(np.diff(image, axis=0))  # 垂直边缘
print("水平边缘形状:", edges_h.shape)
print("垂直边缘形状:", edges_v.shape)

# 3. 图像平滑（3×3 均值滤波）
smoothed = np.zeros_like(image, dtype=float)
for i in range(1, 9):
    for j in range(1, 9):
        window = image[i-1:i+2, j-1:j+2]
        smoothed[i, j] = np.mean(window)
print("平滑后的中心区域:")
print(smoothed[1:9, 1:9].astype(int))
```

### 练习 4：矩阵运算

```python
# 1. 创建两个矩阵
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# 2. 矩阵乘法
C = A @ B
print("A @ B =")
print(C)

# 3. 计算矩阵的转置
print("A的转置:")
print(A.T)

# 4. 计算矩阵的行列式（需要是方阵）
det_A = np.linalg.det(A)
print(f"A的行列式: {det_A}")

# 5. 计算矩阵的逆
inv_A = np.linalg.inv(A)
print("A的逆矩阵:")
print(inv_A)

# 验证：A × A^(-1) = I
identity = A @ inv_A
print("A × A^(-1):")
print(identity)

# 6. 解线性方程组 Ax = b
b = np.array([5, 11])
x = np.linalg.solve(A, b)
print(f"方程组 Ax = b 的解: {x}")

# 验证
print(f"验证 Ax = {A @ x}")
```

### 练习 5：数据模拟与分析

```python
# 模拟一个实验：测量值服从正态分布
np.random.seed(42)
true_value = 100  # 真实值
measurements = np.random.normal(loc=true_value, scale=5, size=1000)

# 1. 计算统计量
print(f"平均值: {np.mean(measurements):.2f}")
print(f"标准差: {np.std(measurements):.2f}")
print(f"中位数: {np.median(measurements):.2f}")
print(f"最小值: {np.min(measurements):.2f}")
print(f"最大值: {np.max(measurements):.2f}")

# 2. 计算置信区间（95%）
lower = np.percentile(measurements, 2.5)
upper = np.percentile(measurements, 97.5)
print(f"95% 置信区间: [{lower:.2f}, {upper:.2f}]")

# 3. 去除异常值（超过3个标准差）
mean = np.mean(measurements)
std = np.std(measurements)
filtered = measurements[np.abs(measurements - mean) <= 3 * std]
print(f"去除异常值前: {len(measurements)} 个数据")
print(f"去除异常值后: {len(filtered)} 个数据")
print(f"去除异常值后的平均值: {np.mean(filtered):.2f}")

# 4. 计算误差
errors = measurements - true_value
mae = np.mean(np.abs(errors))  # 平均绝对误差
rmse = np.sqrt(np.mean(errors**2))  # 均方根误差
print(f"平均绝对误差 (MAE): {mae:.2f}")
print(f"均方根误差 (RMSE): {rmse:.2f}")
```

---

## 第九章：NumPy 最佳实践与技巧

### 9.1 性能优化技巧

```python
import time

# 技巧 1：使用 NumPy 向量化操作而不是循环
n = 1000000

# 低效的 Python 循环
start = time.time()
result_loop = []
for i in range(n):
    result_loop.append(i ** 2)
time_loop = time.time() - start
print(f"Python 循环耗时: {time_loop:.4f} 秒")

# 高效的 NumPy 向量化
start = time.time()
arr = np.arange(n)
result_numpy = arr ** 2
time_numpy = time.time() - start
print(f"NumPy 向量化耗时: {time_numpy:.4f} 秒")
print(f"速度提升: {time_loop / time_numpy:.1f} 倍")

# 技巧 2：预分配数组而不是动态增长
# 低效：动态增长
start = time.time()
arr = np.array([])
for i in range(10000):
    arr = np.append(arr, i)
time_append = time.time() - start
print(f"动态增长耗时: {time_append:.4f} 秒")

# 高效：预分配
start = time.time()
arr = np.zeros(10000)
for i in range(10000):
    arr[i] = i
time_preallocate = time.time() - start
print(f"预分配耗时: {time_preallocate:.4f} 秒")

# 技巧 3：使用 in-place 操作
arr = np.arange(1000000)

# 创建新数组
start = time.time()
result = arr + 1
time_new = time.time() - start

# in-place 操作
start = time.time()
arr += 1
time_inplace = time.time() - start
print(f"创建新数组耗时: {time_new:.4f} 秒")
print(f"in-place 操作耗时: {time_inplace:.4f} 秒")
```

### 9.2 内存管理

```python
# 技巧 1：了解视图和副本
arr = np.arange(10)

# 切片是视图（共享内存）
view = arr[2:7]
print(f"原数组内存地址: {arr.__array_interface__['data'][0]}")
print(f"视图内存地址: {view.__array_interface__['data'][0]}")

# 检查是否共享内存
print(f"是否共享内存: {np.shares_memory(arr, view)}")

# 副本（独立内存）
copy = arr[2:7].copy()
print(f"副本内存地址: {copy.__array_interface__['data'][0]}")
print(f"是否共享内存: {np.shares_memory(arr, copy)}")

# 技巧 2：选择合适的数据类型节省内存
# 默认整数是 int64（8字节）
arr_int64 = np.arange(1000000)
print(f"int64 内存占用: {arr_int64.nbytes / 1024 / 1024:.2f} MB")

# 如果数据范围小，可以使用 int32 或 int16
arr_int16 = np.arange(1000000, dtype=np.int16)
print(f"int16 内存占用: {arr_int16.nbytes / 1024 / 1024:.2f} MB")

# 技巧 3：删除不需要的大数组
large_arr = np.zeros((10000, 10000))
print(f"大数组内存: {large_arr.nbytes / 1024 / 1024:.2f} MB")
del large_arr  # 显式删除
```

### 9.3 常见陷阱

```python
# 陷阱 1：浮点数比较
a = 0.1 + 0.2
b = 0.3
print(a == b)  # 输出: False（浮点精度问题）

# 正确方法：使用 isclose 或 allclose
print(np.isclose(a, b))  # 输出: True
arr1 = np.array([0.1, 0.2, 0.3])
arr2 = np.array([0.1, 0.2, 0.30000001])
print(np.allclose(arr1, arr2))  # 输出: True

# 陷阱 2：整数除法
arr = np.array([1, 2, 3, 4, 5])
result = arr / 2  # 自动转换为浮点数
print(result)  # 输出: [0.5 1.  1.5 2.  2.5]

arr_int = np.array([1, 2, 3, 4, 5], dtype=int)
result_int = arr_int // 2  # 整数除法
print(result_int)  # 输出: [0 1 1 2 2]


# 陷阱 3：广播的意外行为
arr_2d = np.array([[1, 2, 3], [4, 5, 6]])
arr_1d = np.array([10, 20])

# 可能的意图：每行加上对应的值
# 错误的做法
try:
    result = arr_2d + arr_1d  # 形状不兼容会报错
except ValueError as e:
    print(f"错误: {e}")

# 正确的做法：调整形状
arr_1d_reshaped = arr_1d[:, np.newaxis]  # 变为 (2, 1)
result = arr_2d + arr_1d_reshaped
print("正确的结果:")
print(result)
# [[11 12 13]
#  [24 25 26]]

# 陷阱 4：修改切片影响原数组
arr = np.arange(10)
subset = arr[2:7]
subset[0] = 999
print(arr)  # 原数组被修改了！
# [ 0  1 999  3  4  5  6  7  8  9]

# 避免方法：使用 copy()
arr = np.arange(10)
subset = arr[2:7].copy()
subset[0] = 999
print(arr)  # 原数组不受影响
# [0 1 2 3 4 5 6 7 8 9]

# 陷阱 5：数组赋值的类型转换
arr_float = np.array([1.5, 2.7, 3.9])
arr_int = np.array([1, 2, 3], dtype=int)

# 浮点数赋值给整数数组会被截断
arr_int[0] = 5.7
print(arr_int)  # 输出: [5 2 3]（5.7 被截断为 5）

# 陷阱 6：布尔运算符的优先级
arr = np.array([1, 2, 3, 4, 5])

# 错误的写法（优先级问题）
# result = arr > 2 & arr < 5  # 这会报错或得到错误结果

# 正确的写法（使用括号）
result = (arr > 2) & (arr < 5)
print(result)  # [False False  True  True False]

# 陷阱 7：混淆 axis 参数
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

# axis=0 是沿着行的方向（每列）
print(np.sum(arr, axis=0))  # [5 7 9]

# axis=1 是沿着列的方向（每行）
print(np.sum(arr, axis=1))  # [ 6 15]

# 陷阱 8：NaN 和 Inf 的处理
arr = np.array([1, 2, np.nan, 4, np.inf])

# 普通运算会传播 NaN
print(np.sum(arr))  # nan
print(np.mean(arr))  # nan

# 使用 nan-safe 函数
print(np.nansum(arr))  # inf（忽略 NaN）
print(np.nanmean(arr))  # inf

# 检测 NaN 和 Inf
print(np.isnan(arr))  # [False False  True False False]
print(np.isinf(arr))  # [False False False False  True]
print(np.isfinite(arr))  # [ True  True False  True False]

# 陷阱 9：数组形状的误解
arr = np.array([1, 2, 3])
print(arr.shape)  # (3,) 一维数组

arr_2d = np.array([[1, 2, 3]])
print(arr_2d.shape)  # (1, 3) 二维数组（一行三列）

# 这两个是不同的！
print(arr.ndim)  # 1
print(arr_2d.ndim)  # 2

# 陷阱 10：使用 == 比较数组
arr1 = np.array([1, 2, 3])
arr2 = np.array([1, 2, 3])

# == 返回的是逐元素比较结果（布尔数组）
print(arr1 == arr2)  # [ True  True  True]

# 要检查数组是否完全相等，使用 array_equal
print(np.array_equal(arr1, arr2))  # True

# 在条件语句中使用数组会有歧义
if arr1.all():  # 检查所有元素是否为真
    print("所有元素都为真")
```

### 9.4 代码风格与可读性

```python
# 技巧 1：使用有意义的变量名
# 不好的写法
a = np.array([85, 90, 78, 92])
b = np.mean(a)

# 好的写法
student_scores = np.array([85, 90, 78, 92])
average_score = np.mean(student_scores)

# 技巧 2：添加注释说明复杂操作
data = np.random.randn(100, 5)

# 标准化数据：减去均值，除以标准差
mean = np.mean(data, axis=0)
std = np.std(data, axis=0)
normalized_data = (data - mean) / std

# 技巧 3：将复杂操作分解为多个步骤
# 不好的写法（一行完成所有操作）
result = np.sum(np.sqrt(np.abs(data[data > 0])))

# 好的写法（分步骤）
positive_data = data[data > 0]  # 筛选正数
abs_positive = np.abs(positive_data)  # 取绝对值
sqrt_values = np.sqrt(abs_positive)  # 平方根
result = np.sum(sqrt_values)  # 求和

# 技巧 4：使用函数封装重复操作
def normalize(data):
    """标准化数据（Z-score normalization）
  
    参数:
        data: ndarray, 输入数据
  
    返回:
        normalized: ndarray, 标准化后的数据
    """
    mean = np.mean(data, axis=0)
    std = np.std(data, axis=0)
    return (data - mean) / std

# 使用函数
data1 = np.random.randn(100, 5)
data2 = np.random.randn(50, 5)
norm_data1 = normalize(data1)
norm_data2 = normalize(data2)

# 技巧 5：使用常量而不是魔法数字
# 不好的写法
if score >= 90:
    grade = 'A'

# 好的写法
GRADE_A_THRESHOLD = 90
GRADE_B_THRESHOLD = 80
GRADE_C_THRESHOLD = 70

if score >= GRADE_A_THRESHOLD:
    grade = 'A'
elif score >= GRADE_B_THRESHOLD:
    grade = 'B'
```

### 9.5 调试技巧

```python
# 技巧 1：检查数组的基本信息
def inspect_array(arr, name="Array"):
    """打印数组的详细信息"""
    print(f"\n{name} 信息:")
    print(f"  形状: {arr.shape}")
    print(f"  维度: {arr.ndim}")
    print(f"  数据类型: {arr.dtype}")
    print(f"  元素总数: {arr.size}")
    print(f"  内存占用: {arr.nbytes} 字节")
    print(f"  最小值: {np.min(arr)}")
    print(f"  最大值: {np.max(arr)}")
    print(f"  平均值: {np.mean(arr):.2f}")
    print(f"  标准差: {np.std(arr):.2f}")
    print(f"  包含 NaN: {np.any(np.isnan(arr))}")
    print(f"  包含 Inf: {np.any(np.isinf(arr))}")

# 使用示例
data = np.random.randn(100, 5)
inspect_array(data, "随机数据")

# 技巧 2：使用 set_printoptions 控制打印输出
# 设置打印选项
np.set_printoptions(
    precision=2,      # 小数点后保留2位
    suppress=True,    # 抑制科学计数法
    threshold=10,     # 超过10个元素时省略显示
    edgeitems=3       # 省略时显示的边缘元素数
)

large_array = np.random.randn(100)
print(large_array)  # 只显示前3个和后3个元素

# 恢复默认设置
np.set_printoptions()

# 技巧 3：检查形状兼容性
def check_shape_compatibility(arr1, arr2, operation="operation"):
    """检查两个数组是否可以进行操作"""
    try:
        # 尝试广播
        np.broadcast_shapes(arr1.shape, arr2.shape)
        print(f"{operation}: 形状兼容")
        print(f"  arr1: {arr1.shape}")
        print(f"  arr2: {arr2.shape}")
        return True
    except ValueError as e:
        print(f"{operation}: 形状不兼容")
        print(f"  arr1: {arr1.shape}")
        print(f"  arr2: {arr2.shape}")
        print(f"  错误: {e}")
        return False

# 使用示例
arr1 = np.ones((3, 4))
arr2 = np.ones((4,))
check_shape_compatibility(arr1, arr2, "加法运算")

# 技巧 4：逐步调试复杂表达式
data = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# 复杂表达式
# result = np.sum(data[data > 5] ** 2)

# 逐步调试
print("原始数据:")
print(data)

mask = data > 5
print("\n布尔掩码:")
print(mask)

filtered = data[mask]
print("\n筛选后的数据:")
print(filtered)

squared = filtered ** 2
print("\n平方后:")
print(squared)

result = np.sum(squared)
print(f"\n最终结果: {result}")

# 技巧 5：使用 assert 验证假设
def process_data(arr):
    """处理数据，要求输入必须是二维数组"""
    # 验证输入
    assert arr.ndim == 2, f"期望二维数组，实际得到 {arr.ndim} 维"
    assert arr.shape[1] == 5, f"期望5列，实际得到 {arr.shape[1]} 列"
    assert not np.any(np.isnan(arr)), "数据中包含 NaN"
  
    # 处理数据
    return np.mean(arr, axis=0)

# 使用示例
try:
    data = np.random.randn(10, 3)
    result = process_data(data)
except AssertionError as e:
    print(f"验证失败: {e}")
```

### 9.6 与其他库的集成

```python
# NumPy 与 Python 列表的转换
python_list = [1, 2, 3, 4, 5]
numpy_array = np.array(python_list)  # 列表转数组
back_to_list = numpy_array.tolist()  # 数组转列表
print(f"类型: {type(back_to_list)}")  # <class 'list'>

# 多维数组转列表
arr_2d = np.array([[1, 2], [3, 4]])
list_2d = arr_2d.tolist()
print(list_2d)  # [[1, 2], [3, 4]]

# NumPy 与文件 I/O
# 保存数组到文件
data = np.random.randn(100, 5)

# 方法 1：保存为二进制文件（.npy）
np.save('data.npy', data)
loaded_data = np.load('data.npy')
print(f"加载的数据形状: {loaded_data.shape}")

# 方法 2：保存为文本文件
np.savetxt('data.txt', data, delimiter=',', fmt='%.2f')
loaded_text = np.loadtxt('data.txt', delimiter=',')
print(f"从文本加载的形状: {loaded_text.shape}")

# 方法 3：保存多个数组（.npz）
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
np.savez('arrays.npz', first=arr1, second=arr2)
loaded = np.load('arrays.npz')
print(f"第一个数组: {loaded['first']}")
print(f"第二个数组: {loaded['second']}")

# 压缩保存
np.savez_compressed('arrays_compressed.npz', first=arr1, second=arr2)
```

---

## 第十章：实战项目

### 项目 1：简单的数据分析

```python
# 场景：分析一个月的温度数据

# 1. 生成模拟数据（30天的温度，单位：摄氏度）
np.random.seed(42)
days = np.arange(1, 31)
temperatures = np.random.normal(loc=25, scale=5, size=30)
temperatures = np.clip(temperatures, 15, 35)  # 限制在合理范围

print("=== 温度数据分析 ===\n")

# 2. 基本统计
print(f"平均温度: {np.mean(temperatures):.1f}°C")
print(f"最高温度: {np.max(temperatures):.1f}°C (第 {np.argmax(temperatures)+1} 天)")
print(f"最低温度: {np.min(temperatures):.1f}°C (第 {np.argmin(temperatures)+1} 天)")
print(f"温度标准差: {np.std(temperatures):.1f}°C")
print(f"温度范围: {np.ptp(temperatures):.1f}°C")  # ptp = peak to peak

# 3. 分类统计
hot_days = np.sum(temperatures >= 30)
warm_days = np.sum((temperatures >= 25) & (temperatures < 30))
cool_days = np.sum((temperatures >= 20) & (temperatures < 25))
cold_days = np.sum(temperatures < 20)

print(f"\n温度分布:")
print(f"  炎热 (≥30°C): {hot_days} 天")
print(f"  温暖 (25-30°C): {warm_days} 天")
print(f"  凉爽 (20-25°C): {cool_days} 天")
print(f"  寒冷 (<20°C): {cold_days} 天")

# 4. 趋势分析
# 将数据分为三个时段：初期、中期、末期
early = temperatures[:10]
middle = temperatures[10:20]
late = temperatures[20:]

print(f"\n趋势分析:")
print(f"  初期平均: {np.mean(early):.1f}°C")
print(f"  中期平均: {np.mean(middle):.1f}°C")
print(f"  末期平均: {np.mean(late):.1f}°C")

# 5. 计算连续高温天数
consecutive_hot = 0
max_consecutive = 0
for temp in temperatures:
    if temp >= 30:
        consecutive_hot += 1
        max_consecutive = max(max_consecutive, consecutive_hot)
    else:
        consecutive_hot = 0

print(f"\n最长连续高温天数: {max_consecutive} 天")

# 6. 异常值检测（3σ原则）
mean = np.mean(temperatures)
std = np.std(temperatures)
outliers = temperatures[np.abs(temperatures - mean) > 2 * std]
print(f"\n异常温度 (超过2个标准差): {len(outliers)} 天")
if len(outliers) > 0:
    print(f"  异常温度值: {outliers}")
```

### 项目 2：图像处理基础

```python
# 场景：简单的图像操作

# 1. 创建一个模拟的灰度图像（50×50）
np.random.seed(42)
image = np.random.randint(0, 256, (50, 50), dtype=np.uint8)

print("=== 图像处理 ===\n")
print(f"图像大小: {image.shape}")
print(f"像素值范围: [{np.min(image)}, {np.max(image)}]")

# 2. 图像增强
# 直方图均衡化（简化版）
def histogram_equalization(img):
    """简单的直方图均衡化"""
    # 计算累积分布函数
    hist, bins = np.histogram(img.flatten(), 256, [0, 256])
    cdf = hist.cumsum()
    cdf_normalized = cdf * 255 / cdf[-1]
  
    # 使用线性插值
    img_equalized = np.interp(img.flatten(), bins[:-1], cdf_normalized)
    return img_equalized.reshape(img.shape).astype(np.uint8)

enhanced = histogram_equalization(image)
print(f"\n增强后像素值范围: [{np.min(enhanced)}, {np.max(enhanced)}]")

# 3. 图像滤波（简单均值滤波）
def mean_filter(img, kernel_size=3):
    """均值滤波"""
    pad = kernel_size // 2
    padded = np.pad(img, pad, mode='edge')
    filtered = np.zeros_like(img, dtype=float)
  
    for i in range(img.shape[0]):
        for j in range(img.shape[1]):
            window = padded[i:i+kernel_size, j:j+kernel_size]
            filtered[i, j] = np.mean(window)
  
    return filtered.astype(np.uint8)

filtered = mean_filter(image, kernel_size=3)
print(f"滤波后的图像大小: {filtered.shape}")

# 4. 边缘检测（简单梯度法）
def simple_edge_detection(img):
    """简单的边缘检测"""
    # 计算水平和垂直梯度
    gx = np.abs(np.diff(img, axis=1))
    gy = np.abs(np.diff(img, axis=0))
  
    # 梯度幅值（简化处理）
    edges = np.zeros_like(img)
    edges[:, :-1] += gx
    edges[:-1, :] += gy
  
    return edges

edges = simple_edge_detection(image)
print(f"检测到的边缘点数: {np.sum(edges > 50)}")

# 5. 图像二值化
threshold = np.mean(image)
binary = np.where(image > threshold, 255, 0).astype(np.uint8)
white_pixels = np.sum(binary == 255)
black_pixels = np.sum(binary == 0)
print(f"\n二值化:")
print(f"  阈值: {threshold:.1f}")
print(f"  白色像素: {white_pixels} ({white_pixels/binary.size*100:.1f}%)")
print(f"  黑色像素: {black_pixels} ({black_pixels/binary.size*100:.1f}%)")

# 6. 图像旋转（90度）
rotated_90 = np.rot90(image)
rotated_180 = np.rot90(image, k=2)
rotated_270 = np.rot90(image, k=3)
print(f"\n旋转90度后的形状: {rotated_90.shape}")

# 7. 图像翻转
flipped_lr = np.fliplr(image)  # 左右翻转
flipped_ud = np.flipud(image)  # 上下翻转
print(f"翻转后的形状: {flipped_lr.shape}")
```

### 项目 3：时间序列分析

```python
# 场景：股票价格分析

# 1. 生成模拟股票价格数据（100天）
np.random.seed(42)
days = np.arange(100)
# 生成带有趋势和噪声的价格
trend = 100 + 0.5 * days
noise = np.random.normal(0, 2, 100)
prices = trend + noise + 10 * np.sin(days / 10)  # 加入周期性波动
prices = np.maximum(prices, 50)  # 价格不能低于50

print("=== 股票价格分析 ===\n")
print(f"起始价格: {prices[0]:.2f}")
print(f"结束价格: {prices[-1]:.2f}")
print(f"总涨幅: {((prices[-1] - prices[0]) / prices[0] * 100):.2f}%")

# 2. 移动平均线
def moving_average(data, window):
    """计算移动平均"""
    return np.convolve(data, np.ones(window)/window, mode='valid')

ma5 = moving_average(prices, 5)   # 5日均线
ma10 = moving_average(prices, 10)  # 10日均线
ma20 = moving_average(prices, 20)  # 20日均线

print(f"\n移动平均线:")
print(f"  5日均线当前值: {ma5[-1]:.2f}")
print(f"  10日均线当前值: {ma10[-1]:.2f}")
print(f"  20日均线当前值: {ma20[-1]:.2f}")

# 3. 收益率计算
returns = np.diff(prices) / prices[:-1]  # 日收益率
print(f"\n收益率统计:")
print(f"  平均日收益率: {np.mean(returns)*100:.3f}%")
print(f"  收益率标准差: {np.std(returns)*100:.3f}%")
print(f"  最大单日涨幅: {np.max(returns)*100:.2f}%")
print(f"  最大单日跌幅: {np.min(returns)*100:.2f}%")

# 4. 波动率（标准差）
volatility_5d = np.array([np.std(prices[max(0,i-5):i+1]) 
                          for i in range(len(prices))])
print(f"\n5日波动率:")
print(f"  当前: {volatility_5d[-1]:.2f}")
print(f"  平均: {np.mean(volatility_5d):.2f}")

# 5. 支撑位和阻力位（简化计算）
# 找出局部最小值和最大值
support_levels = []
resistance_levels = []

for i in range(5, len(prices)-5):
    window = prices[i-5:i+5]
    if prices[i] == np.min(window):
        support_levels.append(prices[i])
    if prices[i] == np.max(window):
        resistance_levels.append(prices[i])

if support_levels:
    print(f"\n支撑位: {np.mean(support_levels):.2f}")
if resistance_levels:
    print(f"阻力位: {np.mean(resistance_levels):.2f}")

# 6. 交易信号（简单策略）
# 当短期均线上穿长期均线时买入，下穿时卖出
signals = np.zeros(len(ma20))
positions = np.zeros(len(ma20))

# 对齐两个均线（都从第20天开始）
ma5_aligned = ma5[15:]  # ma5 从第5天开始，需要跳过15天
ma20_aligned = ma20

for i in range(1, len(ma20_aligned)):
    if ma5_aligned[i] > ma20_aligned[i] and ma5_aligned[i-1] <= ma20_aligned[i-1]:
        signals[i] = 1  # 买入信号
    elif ma5_aligned[i] < ma20_aligned[i] and ma5_aligned[i-1] >= ma20_aligned[i-1]:
        signals[i] = -1  # 卖出信号

buy_signals = np.sum(signals == 1)
sell_signals = np.sum(signals == -1)
print(f"\n交易信号:")
print(f"  买入信号: {buy_signals} 次")
print(f"  卖出信号: {sell_signals} 次")

# 7. 最大回撤
cummax = np.maximum.accumulate(prices)
drawdown = (prices - cummax) / cummax
max_drawdown = np.min(drawdown)
print(f"\n最大回撤: {max_drawdown*100:.2f}%")
```

### 项目 4：科学计算 - 数值积分

```python
# 场景：使用蒙特卡洛方法计算积分

print("=== 蒙特卡洛积分 ===\n")

# 1. 计算圆的面积（验证方法）
def monte_carlo_circle(n_samples):
    """使用蒙特卡洛方法估算圆的面积"""
    np.random.seed(42)
    x = np.random.uniform(-1, 1, n_samples)
    y = np.random.uniform(-1, 1, n_samples)
  
    # 判断点是否在圆内
    inside = (x**2 + y**2) <= 1
    pi_estimate = 4 * np.sum(inside) / n_samples
  
    return pi_estimate

# 使用不同的样本数
for n in [100, 1000, 10000, 100000]:
    pi_est = monte_carlo_circle(n)
    error = abs(pi_est - np.pi)
    print(f"样本数 {n:6d}: π ≈ {pi_est:.6f}, 误差 = {error:.6f}")

# 2. 计算定积分 ∫[0,1] x^2 dx = 1/3
def monte_carlo_integral(func, a, b, n_samples):
    """蒙特卡洛积分"""
    np.random.seed(42)
    x = np.random.uniform(a, b, n_samples)
    y = func(x)
    integral = (b - a) * np.mean(y)
    return integral

# 定义被积函数
def f(x):
    return x**2

true_value = 1/3
for n in [100, 1000, 10000, 100000]:
    estimate = monte_carlo_integral(f, 0, 1, n)
    error = abs(estimate - true_value)
    print(f"\n样本数 {n:6d}: ∫x²dx ≈ {estimate:.6f}, 误差 = {error:.6f}")

# 3. 多维积分
print("\n\n=== 多维积分 ===")
print("计算球体积积分")

def monte_carlo_sphere(n_samples):
    """计算单位球体积"""
    np.random.seed(42)
    x = np.random.uniform(-1, 1, n_samples)
    y = np.random.uniform(-1, 1, n_samples)
    z = np.random.uniform(-1, 1, n_samples)
  
    inside = (x**2 + y**2 + z**2) <= 1
    volume = 8 * np.sum(inside) / n_samples  # 立方体体积是8
  
    return volume

true_volume = 4 * np.pi / 3  # 理论值
for n in [1000, 10000, 100000]:
    volume_est = monte_carlo_sphere(n)
    error = abs(volume_est - true_volume)
    print(f"样本数 {n:6d}: V ≈ {volume_est:.6f}, 误差 = {error:.6f}")
```

---

## 总结

恭喜你完成了 NumPy 的完整学习之旅！让我们回顾一下你所掌握的内容：

### 你现在可以：

1. **创建和操作数组**
   - 使用多种方法创建数组
   - 理解数组的形状、维度和数据类型
   - 进行形状变换和数组重组
2. **执行数组运算**
   - 进行逐元素运算
   - 理解和应用广播机制
   - 使用通用函数和聚合函数
   - 进行矩阵运算
3. **生成随机数**
   - 使用各种概率分布
   - 进行随机采样和模拟
4. **索引和切片**
   - 使用基本索引和切片
   - 应用布尔索引筛选数据
   - 使用花式索引灵活选择元素
5. **编写高效代码**
   - 应用性能优化技巧
   - 管理内存
   - 避免常见陷阱
   - 调试复杂的数组操作

6. **解决实际问题**
   - 数据分析和统计
   - 图像处理基础
   - 时间序列分析
   - 科学计算和数值模拟

---

## 第十一章：进阶主题

### 11.1 结构化数组

结构化数组允许你在单个数组中存储不同类型的数据，类似于数据库表。

```python
# 创建结构化数组
# 定义数据类型
dt = np.dtype([('name', 'U10'),      # Unicode字符串，最长10个字符
               ('age', 'i4'),         # 32位整数
               ('height', 'f4'),      # 32位浮点数
               ('grade', 'U1')])      # 单个字符

# 创建数组
students = np.array([
    ('Alice', 20, 165.5, 'A'),
    ('Bob', 21, 175.0, 'B'),
    ('Charlie', 19, 170.5, 'A'),
    ('David', 22, 180.0, 'C')
], dtype=dt)

print("学生数据:")
print(students)

# 按字段访问
print("\n所有学生的姓名:")
print(students['name'])

print("\n所有学生的年龄:")
print(students['age'])

# 按条件筛选
print("\n年龄大于20的学生:")
print(students[students['age'] > 20])

print("\n成绩为A的学生:")
print(students[students['grade'] == 'A'])

# 排序
sorted_by_height = np.sort(students, order='height')
print("\n按身高排序:")
print(sorted_by_height)

# 多字段排序
sorted_multi = np.sort(students, order=['grade', 'age'])
print("\n按成绩和年龄排序:")
print(sorted_multi)

# 修改字段值
students['age'] += 1  # 所有学生年龄+1
print("\n一年后的年龄:")
print(students['age'])

# 添加计算字段（需要创建新数组）
# 计算BMI（假设身高单位是cm，需要转换）
bmi_data = students['height'] / 100  # 这里简化了BMI计算

# 创建更复杂的结构
dt_extended = np.dtype([
    ('name', 'U10'),
    ('age', 'i4'),
    ('scores', 'f4', (3,))  # 三门课程的成绩
])

students_scores = np.array([
    ('Alice', 20, [85, 90, 88]),
    ('Bob', 21, [78, 82, 80]),
    ('Charlie', 19, [92, 95, 93])
], dtype=dt_extended)

print("\n\n带成绩的学生数据:")
print(students_scores)

# 访问嵌套数据
print("\nAlice的成绩:")
print(students_scores[0]['scores'])

# 计算每个学生的平均成绩
avg_scores = np.mean(students_scores['scores'], axis=1)
print("\n平均成绩:")
for name, avg in zip(students_scores['name'], avg_scores):
    print(f"{name}: {avg:.2f}")
```

### 11.2 掩码数组（Masked Arrays）

掩码数组用于处理包含无效或缺失值的数据。

```python
import numpy.ma as ma

# 创建带有无效值的数据
data = np.array([1, 2, -999, 4, 5, -999, 7, 8])

# 创建掩码数组，标记无效值
masked_data = ma.masked_equal(data, -999)
print("掩码数组:")
print(masked_data)
print("\n掩码:")
print(masked_data.mask)

# 计算时自动忽略被掩码的值
print(f"\n平均值（忽略-999）: {masked_data.mean()}")
print(f"总和（忽略-999）: {masked_data.sum()}")

# 使用条件创建掩码
data = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
masked_data = ma.masked_where(data > 7, data)
print("\n\n大于7的值被掩码:")
print(masked_data)
print(f"平均值: {masked_data.mean()}")

# 二维掩码数组
data_2d = np.array([[1, 2, -999, 4],
                     [5, -999, 7, 8],
                     [9, 10, 11, -999]])

masked_2d = ma.masked_equal(data_2d, -999)
print("\n\n二维掩码数组:")
print(masked_2d)

# 按行计算平均值
row_means = masked_2d.mean(axis=1)
print("\n每行平均值:")
print(row_means)

# 按列计算平均值
col_means = masked_2d.mean(axis=0)
print("\n每列平均值:")
print(col_means)

# 填充掩码值
filled = masked_2d.filled(0)  # 用0填充被掩码的值
print("\n用0填充后:")
print(filled)

# 用平均值填充
filled_mean = masked_2d.filled(masked_2d.mean())
print("\n用平均值填充后:")
print(filled_mean)

# 压缩掉被掩码的值
compressed = masked_2d.compressed()
print("\n压缩后的一维数组:")
print(compressed)

# 掩码数组的运算
data1 = ma.array([1, 2, 3, 4], mask=[0, 0, 1, 0])
data2 = ma.array([5, 6, 7, 8], mask=[0, 1, 0, 0])
result = data1 + data2
print("\n\n掩码数组相加:")
print(f"data1: {data1}")
print(f"data2: {data2}")
print(f"相加结果: {result}")
print(f"结果的掩码: {result.mask}")
```

### 11.3 记录数组（Record Arrays）

记录数组是结构化数组的扩展，允许使用属性访问字段。

```python
# 使用 np.rec.fromarrays 创建
names = ['Alice', 'Bob', 'Charlie']
ages = [20, 21, 19]
heights = [165.5, 175.0, 170.5]

# 创建记录数组
rec_array = np.rec.fromarrays([names, ages, heights],
                               names='name,age,height')

print("记录数组:")
print(rec_array)

# 使用属性访问（比字典访问更方便）
print("\n所有姓名:")
print(rec_array.name)

print("\n所有年龄:")
print(rec_array.age)

# 仍然可以使用索引
print("\n第一个记录:")
print(rec_array[0])

# 属性访问和索引结合
print("\nBob的身高:")
print(rec_array[1].height)

# 使用 recfromcsv 从 CSV 读取（模拟）
csv_data = """name,age,height
Alice,20,165.5
Bob,21,175.0
Charlie,19,170.5"""

# 实际应用中会从文件读取
# rec_array = np.recfromcsv('data.csv')
```

### 11.4 内存映射文件

处理大型数据集时，内存映射允许你将磁盘上的数据作为数组访问，无需全部加载到内存。

```python
# 创建一个大型数组并保存到磁盘
large_data = np.random.randn(1000, 1000)
np.save('large_data.npy', large_data)

# 使用内存映射打开
# mode='r' 只读，'r+' 读写，'w+' 创建并写入
mmap_data = np.load('large_data.npy', mmap_mode='r')

print(f"数据形状: {mmap_data.shape}")
print(f"数据类型: {mmap_data.dtype}")
print(f"内存占用: {mmap_data.nbytes / 1024 / 1024:.2f} MB")

# 只访问需要的部分（不会全部加载到内存）
subset = mmap_data[100:200, 100:200]
print(f"\n子集形状: {subset.shape}")
print(f"子集平均值: {np.mean(subset):.4f}")

# 创建内存映射数组
# 直接在磁盘上创建数组，适合非常大的数据
fp = np.memmap('test.mmap', dtype='float32', mode='w+', shape=(1000, 1000))
fp[:] = np.random.randn(1000, 1000)
fp.flush()  # 确保写入磁盘

# 再次打开
fp_read = np.memmap('test.mmap', dtype='float32', mode='r', shape=(1000, 1000))
print(f"\n从内存映射读取的数据形状: {fp_read.shape}")

# 清理临时文件
import os
os.remove('large_data.npy')
os.remove('test.mmap')
```

### 11.5 日期时间处理

NumPy 提供了专门的日期时间数据类型。

```python
# 创建日期时间数组
dates = np.array(['2024-01-01', '2024-02-01', '2024-03-01'], dtype='datetime64')
print("日期数组:")
print(dates)

# 日期时间运算
one_week = np.timedelta64(7, 'D')  # 7天
future_dates = dates + one_week
print("\n一周后的日期:")
print(future_dates)

# 日期范围
date_range = np.arange('2024-01-01', '2024-01-31', dtype='datetime64[D]')
print(f"\n一月的日期数量: {len(date_range)}")
print("前5天:")
print(date_range[:5])

# 不同的时间单位
# 'D' 天, 'h' 小时, 'm' 分钟, 's' 秒
dates_hours = np.arange('2024-01-01T00', '2024-01-01T24', dtype='datetime64[h]')
print(f"\n一天的小时数: {len(dates_hours)}")

# 工作日计算
start_date = np.datetime64('2024-01-01')
end_date = np.datetime64('2024-01-31')
# busday_count 计算工作日
workdays = np.busday_count(start_date, end_date)
print(f"\n1月的工作日数: {workdays}")

# 生成工作日序列
workday_range = np.busday_offset('2024-01-01', np.arange(10))
print("\n接下来的10个工作日:")
print(workday_range)

# 日期差值
date1 = np.datetime64('2024-06-15')
date2 = np.datetime64('2024-01-01')
diff = date1 - date2
print(f"\n日期差: {diff}")
print(f"天数: {diff / np.timedelta64(1, 'D')}")

# 时间序列数据处理示例
# 生成一年的日期
dates = np.arange('2024-01-01', '2025-01-01', dtype='datetime64[D]')
# 生成对应的数据（例如温度）
np.random.seed(42)
temperatures = 20 + 10 * np.sin(np.arange(len(dates)) * 2 * np.pi / 365) + \
               np.random.randn(len(dates)) * 3

# 按月统计
# 提取月份
months = dates.astype('datetime64[M]')
unique_months = np.unique(months)

print("\n\n每月平均温度:")
for month in unique_months[:6]:  # 只显示前6个月
    mask = months == month
    avg_temp = np.mean(temperatures[mask])
    print(f"{month}: {avg_temp:.1f}°C")

# 按星期几统计
weekdays = np.busday_offset(dates, 0).astype('datetime64[D]')  # 标准化到工作日
is_weekend = np.is_busday(dates) == False
weekend_temps = temperatures[is_weekend]
weekday_temps = temperatures[~is_weekend]

print(f"\n周末平均温度: {np.mean(weekend_temps):.1f}°C")
print(f"工作日平均温度: {np.mean(weekday_temps):.1f}°C")
```

### 11.6 线性代数进阶

```python
# 特征值和特征向量
A = np.array([[4, 2],
              [1, 3]])

eigenvalues, eigenvectors = np.linalg.eig(A)
print("矩阵 A:")
print(A)
print(f"\n特征值:")
print(eigenvalues)
print(f"\n特征向量:")
print(eigenvectors)

# 验证：A * v = λ * v
for i in range(len(eigenvalues)):
    v = eigenvectors[:, i]
    lam = eigenvalues[i]
    left = A @ v
    right = lam * v
    print(f"\n特征值 {lam:.4f} 的验证:")
    print(f"A * v = {left}")
    print(f"λ * v = {right}")

# 奇异值分解 (SVD)
A = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9],
              [10, 11, 12]])

U, s, Vt = np.linalg.svd(A, full_matrices=False)
print("\n\n奇异值分解:")
print(f"U 形状: {U.shape}")
print(f"奇异值: {s}")
print(f"V^T 形状: {Vt.shape}")

# 重构原矩阵
S = np.diag(s)
A_reconstructed = U @ S @ Vt
print("\n重构的矩阵:")
print(A_reconstructed)
print("\n原矩阵:")
print(A)
print(f"\n重构误差: {np.linalg.norm(A - A_reconstructed):.10f}")

# QR 分解
A = np.array([[1, 2],
              [3, 4],
              [5, 6]], dtype=float)

Q, R = np.linalg.qr(A)
print("\n\nQR 分解:")
print("Q (正交矩阵):")
print(Q)
print("\nR (上三角矩阵):")
print(R)

# 验证：A = Q * R
print("\nQ * R:")
print(Q @ R)
print("\n原矩阵 A:")
print(A)

# 验证 Q 是正交矩阵：Q^T * Q = I
print("\nQ^T * Q (应该是单位矩阵):")
print(Q.T @ Q)

# 矩阵的范数
A = np.array([[1, 2],
              [3, 4]])

print("\n\n矩阵范数:")
print(f"Frobenius 范数: {np.linalg.norm(A, 'fro')}")
print(f"最大奇异值 (2-范数): {np.linalg.norm(A, 2)}")
print(f"最大列和 (1-范数): {np.linalg.norm(A, 1)}")
print(f"最大行和 (∞-范数): {np.linalg.norm(A, np.inf)}")

# 矩阵的秩
A1 = np.array([[1, 2, 3],
               [4, 5, 6],
               [7, 8, 9]])
rank1 = np.linalg.matrix_rank(A1)
print(f"\n矩阵的秩: {rank1}")

A2 = np.array([[1, 2],
               [3, 4],
               [5, 6]])
rank2 = np.linalg.matrix_rank(A2)
print(f"满秩矩阵的秩: {rank2}")

# 最小二乘解
# 求解超定方程组 Ax = b
A = np.array([[1, 1],
              [1, 2],
              [1, 3],
              [1, 4]])
b = np.array([6, 5, 7, 10])

x, residuals, rank, s = np.linalg.lstsq(A, b, rcond=None)
print("\n\n最小二乘解:")
print(f"x = {x}")
print(f"残差平方和: {residuals}")

# 验证
print(f"\nA * x = {A @ x}")
print(f"b = {b}")
```

### 11.7 多项式运算

```python
# 创建多项式 p(x) = 2x^2 + 3x + 1
# 系数从高次到低次排列
coeffs = [2, 3, 1]
p = np.poly1d(coeffs)

print("多项式:")
print(p)

# 计算多项式的值
x = 2
print(f"\np({x}) = {p(x)}")

# 多项式数组计算
x_array = np.array([0, 1, 2, 3])
y_array = p(x_array)
print(f"\np({x_array}) = {y_array}")

# 多项式求根
roots = np.roots(coeffs)
print(f"\n多项式的根: {roots}")

# 验证根
for root in roots:
    print(f"p({root:.4f}) = {p(root):.10f}")

# 从根构造多项式
p_from_roots = np.poly(roots)
print(f"\n从根构造的多项式系数: {p_from_roots}")

# 多项式乘法
p1 = np.poly1d([1, 2])  # x + 2
p2 = np.poly1d([1, 3])  # x + 3
p_product = p1 * p2
print(f"\n({p1}) * ({p2}) = {p_product}")

# 多项式微分
p = np.poly1d([3, 0, 2, 1])  # 3x^3 + 2x + 1
p_deriv = np.polyder(p)
print(f"\n原多项式: {p}")
print(f"导数: {p_deriv}")

# 多项式积分
p_integ = np.polyint(p)
print(f"不定积分: {p_integ}")

# 多项式拟合
# 生成带噪声的数据
x = np.linspace(0, 10, 50)
y = 2*x**2 + 3*x + 1 + np.random.randn(50) * 5

# 拟合二次多项式
coeffs_fitted = np.polyfit(x, y, 2)
p_fitted = np.poly1d(coeffs_fitted)

print(f"\n\n拟合的多项式: {p_fitted}")
print(f"真实系数: [2, 3, 1]")
print(f"拟合系数: {coeffs_fitted}")

# 计算 R² 分数
y_pred = p_fitted(x)
ss_res = np.sum((y - y_pred)**2)
ss_tot = np.sum((y - np.mean(y))**2)
r_squared = 1 - (ss_res / ss_tot)
print(f"\nR² 分数: {r_squared:.4f}")
```

### 11.8 傅里叶变换

```python
# 生成信号：三个正弦波的叠加
sample_rate = 1000  # 采样率 1000 Hz
duration = 1.0      # 持续时间 1 秒
t = np.linspace(0, duration, int(sample_rate * duration))

# 信号：50Hz + 120Hz + 200Hz
freq1, freq2, freq3 = 50, 120, 200
signal = (np.sin(2 * np.pi * freq1 * t) +
          0.5 * np.sin(2 * np.pi * freq2 * t) +
          0.3 * np.sin(2 * np.pi * freq3 * t))

print("=== 傅里叶变换示例 ===\n")
print(f"信号长度: {len(signal)} 个样本")
print(f"采样率: {sample_rate} Hz")

# 执行 FFT
fft_result = np.fft.fft(signal)
frequencies = np.fft.fftfreq(len(signal), 1/sample_rate)

# 计算幅值谱
magnitude = np.abs(fft_result)

# 只取正频率部分
positive_freq_idx = frequencies > 0
frequencies_positive = frequencies[positive_freq_idx]
magnitude_positive = magnitude[positive_freq_idx]

# 找出主要频率成分
peak_indices = np.argsort(magnitude_positive)[-3:]  # 最大的3个峰值
peak_freqs = frequencies_positive[peak_indices]

print("\n检测到的主要频率:")
for freq in sorted(peak_freqs):
    print(f"  {freq:.1f} Hz")

# 逆傅里叶变换
signal_reconstructed = np.fft.ifft(fft_result)
reconstruction_error = np.max(np.abs(signal - signal_reconstructed.real))
print(f"\n重构误差: {reconstruction_error:.10f}")

# 滤波示例：去除高频成分
cutoff_freq = 150  # Hz
fft_filtered = fft_result.copy()
fft_filtered[np.abs(frequencies) > cutoff_freq] = 0

signal_filtered = np.fft.ifft(fft_filtered).real
print(f"\n滤波后保留了 {np.sum(fft_filtered != 0)} 个频率成分")

# 二维 FFT（图像处理）
# 创建一个简单的图像模式
x = np.linspace(-5, 5, 128)
y = np.linspace(-5, 5, 128)
X, Y = np.meshgrid(x, y)
image = np.sin(2*np.pi*X/2) + np.sin(2*np.pi*Y/3)

# 执行 2D FFT
fft_2d = np.fft.fft2(image)
fft_2d_shifted = np.fft.fftshift(fft_2d)  # 将零频率移到中心

# 幅值谱
magnitude_2d = np.abs(fft_2d_shifted)

print(f"\n\n2D FFT:")
print(f"原图像形状: {image.shape}")
print(f"频谱形状: {magnitude_2d.shape}")
print(f"最大幅值: {np.max(magnitude_2d):.2f}")
```

### 11.9 性能分析与优化

```python
import time

print("=== 性能分析 ===\n")

# 1. 向量化 vs 循环
n = 1000000
a = np.random.rand(n)
b = np.random.rand(n)

# Python 循环
start = time.time()
result_loop = np.zeros(n)
for i in range(n):
    result_loop[i] = a[i] + b[i]
time_loop = time.time() - start

# NumPy 向量化
start = time.time()
result_vectorized = a + b
time_vectorized = time.time() - start

print(f"加法运算 ({n} 个元素):")
print(f"  循环: {time_loop:.4f} 秒")
print(f"  向量化: {time_vectorized:.4f} 秒")
print(f"  速度提升: {time_loop/time_vectorized:.1f}x")

# 2. ufunc 性能
# 使用内置 ufunc
start = time.time()
result_ufunc = np.sqrt(a)
time_ufunc = time.time() - start

# 使用 Python 函数
start = time.time()
result_python = np.array([np.sqrt(x) for x in a])
time_python = time.time() - start

print(f"\n平方根运算:")
print(f"  ufunc: {time_ufunc:.4f} 秒")
print(f"  列表推导: {time_python:.4f} 秒")
print(f"  速度提升: {time_python/time_ufunc:.1f}x")

# 3. 广播 vs 显式循环
arr_2d = np.random.rand(1000, 1000)
arr_1d = np.random.rand(1000)

# 使用广播
start = time.time()
result_broadcast = arr_2d + arr_1d
time_broadcast = time.time() - start

# 显式循环
start = time.time()
result_loop = np.zeros_like(arr_2d)
for i in range(arr_2d.shape[0]):
    result_loop[i, :] = arr_2d[i, :] + arr_1d
time_loop = time.time() - start

print(f"\n广播运算 (1000×1000 + 1000):")
print(f"  广播: {time_broadcast:.4f} 秒")
print(f"  循环: {time_loop:.4f} 秒")
print(f"  速度提升: {time_loop/time_broadcast:.1f}x")

# 4. 内存布局的影响（行优先 vs 列优先）
arr = np.random.rand(5000, 5000)

# 按行求和（C 连续）
start = time.time()
sum_rows = np.sum(arr, axis=1)
time_rows = time.time() - start

# 按列求和（需要跳跃访问）
start = time.time()
sum_cols = np.sum(arr, axis=0)
time_cols = time.time() - start

print(f"\n内存布局影响 (5000×5000):")
print(f"  按行求和: {time_rows:.4f} 秒")
print(f"  按列求和: {time_cols:.4f} 秒")

# 5. 预分配 vs 动态增长
n = 10000

# 动态增长
start = time.time()
arr_dynamic = np.array([])
for i in range(n):
    arr_dynamic = np.append(arr_dynamic, i)
time_dynamic = time.time() - start

# 预分配
start = time.time()
arr_preallocated = np.zeros(n)
for i in range(n):
    arr_preallocated[i] = i
time_preallocated = time.time() - start

print(f"\n数组构建 ({n} 个元素):")
print(f"  动态增长: {time_dynamic:.4f} 秒")
print(f"  预分配: {time_preallocated:.4f} 秒")
print(f"  速度提升: {time_dynamic/time_preallocated:.1f}x")

# 6. 使用 numexpr（如果安装了）
try:
    import numexpr as ne
  
    a = np.random.rand(1000000)
    b = np.random.rand(1000000)
    c = np.random.rand(1000000)
  
    # NumPy
    start = time.time()
    result_numpy = a**2 + b**2 + c**2
    time_numpy = time.time() - start
  
    # numexpr
    start = time.time()
    result_numexpr = ne.evaluate('a**2 + b**2 + c**2')
    time_numexpr = time.time() - start
  
    print(f"\n复杂表达式 (a²+b²+c²):")
    print(f"  NumPy: {time_numpy:.4f} 秒")
    print(f"  numexpr: {time_numexpr:.4f} 秒")
    print(f"  速度提升: {time_numpy/time_numexpr:.1f}x")
except ImportError:
    print("\nnumexpr 未安装，跳过测试")

# 7. 视图 vs 副本的性能影响
large_array = np.random.rand(10000, 10000)

# 创建视图（快）
start = time.time()
view = large_array[::2, ::2]
time_view = time.time() - start

# 创建副本（慢）
start = time.time()
copy = large_array[::2, ::2].copy()
time_copy = time.time() - start

print(f"\n创建子数组 (10000×10000 的一半):")
print(f"  视图: {time_view:.6f} 秒")
print(f"  副本: {time_copy:.4f} 秒")
```

---

## 第十二章：与其他库的协作

### 12.1 Pandas 集成

```python
# NumPy 是 Pandas 的基础
# Pandas 的 Series 和 DataFrame 底层都是 NumPy 数组

# 示例：NumPy 数组转 Pandas
import pandas as pd

# 1. 创建 DataFrame
data = np.array([[1, 2, 3],
                 [4, 5, 6],
                 [7, 8, 9]])

df = pd.DataFrame(data, 
                  columns=['A', 'B', 'C'],
                  index=['row1', 'row2', 'row3'])

print("从 NumPy 数组创建 DataFrame:")
print(df)

# 2. DataFrame 转 NumPy 数组
arr = df.values  # 或 df.to_numpy()
print("\nDataFrame 转回 NumPy 数组:")
print(arr)
print(f"类型: {type(arr)}")

# 3. 使用结构化数组创建 DataFrame
dt = np.dtype([('name', 'U10'), ('age', 'i4'), ('score', 'f4')])
structured_arr = np.array([
    ('Alice', 20, 85.5),
    ('Bob', 21, 90.0),
    ('Charlie', 19, 78.5)
], dtype=dt)

df_structured = pd.DataFrame(structured_arr)
print("\n从结构化数组创建 DataFrame:")
print(df_structured)

# 4. Series 与 NumPy 数组
series = pd.Series(np.random.randn(5), 
                   index=['a', 'b', 'c', 'd', 'e'])
print("\nPandas Series:")
print(series)

# Series 的底层数组
arr_from_series = series.values
print("\nSeries 的底层 NumPy 数组:")
print(arr_from_series)

# 5. NumPy 函数直接应用于 Pandas
df = pd.DataFrame(np.random.randn(5, 3), columns=['A', 'B', 'C'])
print("\n原始 DataFrame:")
print(df)

# 应用 NumPy 函数
df_sqrt = np.sqrt(np.abs(df))  # 对绝对值开方
print("\nNumPy 函数应用后:")
print(df_sqrt)

# 6. 组合使用：数据分析示例
# 创建时间序列数据
dates = pd.date_range('2024-01-01', periods=100)
values = np.cumsum(np.random.randn(100)) + 100  # 随机游走

ts = pd.Series(values, index=dates)

# 使用 NumPy 计算移动平均
window = 10
moving_avg = np.convolve(ts.values, np.ones(window)/window, mode='valid')

print(f"\n时间序列长度: {len(ts)}")
print(f"移动平均序列长度: {len(moving_avg)}")
print(f"最近的移动平均值: {moving_avg[-5:]}")

# 7. 处理缺失值
df_with_nan = pd.DataFrame({
    'A': [1, 2, np.nan, 4, 5],
    'B': [np.nan, 2, 3, 4, 5],
    'C': [1, 2, 3, 4, np.nan]
})

print("\n包含缺失值的 DataFrame:")
print(df_with_nan)

# 使用 NumPy 的 nanmean 计算均值
col_means = np.array([np.nanmean(df_with_nan[col]) 
                      for col in df_with_nan.columns])
print("\n各列均值（忽略 NaN）:")
print(dict(zip(df_with_nan.columns, col_means)))

# 8. 高级索引
df = pd.DataFrame(np.random.randn(5, 3), 
                  columns=['A', 'B', 'C'],
                  index=['r1', 'r2', 'r3', 'r4', 'r5'])

# 布尔索引（Pandas 风格）
mask = df['A'] > 0
filtered_df = df[mask]
print("\n布尔索引后的 DataFrame:")
print(filtered_df)

# 底层仍然是 NumPy 操作
print("\n掩码数组（NumPy）:")
print(mask.values)
```

### 12.2 Matplotlib 可视化

```python
import matplotlib.pyplot as plt

print("=== Matplotlib 与 NumPy 集成 ===\n")

# 1. 基本绘图
x = np.linspace(0, 2*np.pi, 100)
y = np.sin(x)

plt.figure(figsize=(10, 6))
plt.plot(x, y, label='sin(x)')
plt.xlabel('x')
plt.ylabel('y')
plt.title('正弦函数')
plt.legend()
plt.grid(True)
plt.savefig('sine_wave.png', dpi=100, bbox_inches='tight')
plt.close()
print("✓ 保存了 sine_wave.png")

# 2. 多条曲线
x = np.linspace(0, 2*np.pi, 100)
y1 = np.sin(x)
y2 = np.cos(x)
y3 = np.sin(x) * np.cos(x)

plt.figure(figsize=(10, 6))
plt.plot(x, y1, label='sin(x)', linewidth=2)
plt.plot(x, y2, label='cos(x)', linewidth=2)
plt.plot(x, y3, label='sin(x)cos(x)', linewidth=2, linestyle='--')
plt.xlabel('x')
plt.ylabel('y')
plt.title('三角函数')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('trig_functions.png', dpi=100, bbox_inches='tight')
plt.close()
print("✓ 保存了 trig_functions.png")

# 3. 散点图
np.random.seed(42)
x = np.random.randn(100)
y = 2*x + np.random.randn(100)*0.5

plt.figure(figsize=(8, 6))
plt.scatter(x, y, alpha=0.5, c=y, cmap='viridis')
plt.colorbar(label='y value')
plt.xlabel('x')
plt.ylabel('y')
plt.title('散点图示例')
plt.grid(True, alpha=0.3)
plt.savefig('scatter_plot.png', dpi=100, bbox_inches='tight')
plt.close()
print("✓ 保存了 scatter_plot.png")

# 4. 直方图
data = np.random.randn(10000)

plt.figure(figsize=(10, 6))
plt.hist(data, bins=50, alpha=0.7, edgecolor='black')
plt.axvline(np.mean(data), color='red', linestyle='--', 
            linewidth=2, label=f'Mean: {np.mean(data):.2f}')
plt.axvline(np.median(data), color='green', linestyle='--', 
            linewidth=2, label=f'Median: {np.median(data):.2f}')
plt.xlabel('Value')
plt.ylabel('Frequency')
plt.title('正态分布直方图')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('histogram.png', dpi=100, bbox_inches='tight')
plt.close()
print("✓ 保存了 histogram.png")

# 5. 热力图（2D 数据）
data_2d = np.random.randn(20, 20)

plt.figure(figsize=(8, 6))
plt.imshow(data_2d, cmap='coolwarm', aspect='auto', interpolation='nearest')
plt.colorbar(label='Value')
plt.xlabel('X')
plt.ylabel('Y')
plt.title('热力图')
plt.savefig('heatmap.png', dpi=100, bbox_inches='tight')
plt.close()
print("✓ 保存了 heatmap.png")

# 6. 等高线图
x = np.linspace(-3, 3, 100)
y = np.linspace(-3, 3, 100)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

plt.figure(figsize=(8, 6))
contour = plt.contour(X, Y, Z, levels=15, cmap='viridis')
plt.clabel(contour, inline=True, fontsize=8)
plt.colorbar(contour, label='Z value')
plt.xlabel('X')
plt.ylabel('Y')
plt.title('等高线图')
plt.savefig('contour_plot.png', dpi=100, bbox_inches='tight')
plt.close()
print("✓ 保存了 contour_plot.png")

# 7. 子图
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 子图 1: 线图
x = np.linspace(0, 10, 100)
axes[0, 0].plot(x, np.sin(x))
axes[0, 0].set_title('正弦波')
axes[0, 0].grid(True)

# 子图 2: 条形图
categories = ['A', 'B', 'C', 'D']
values = np.random.randint(10, 100, 4)
axes[0, 1].bar(categories, values, color='skyblue')
axes[0, 1].set_title('条形图')

# 子图 3: 散点图
x = np.random.randn(50)
y = np.random.randn(50)
axes[1, 0].scatter(x, y, alpha=0.6)
axes[1, 0].set_title('散点图')
axes[1, 0].grid(True)

# 子图 4: 箱线图
data = [np.random.randn(100) for _ in range(4)]
axes[1, 1].boxplot(data, labels=['A', 'B', 'C', 'D'])
axes[1, 1].set_title('箱线图')

plt.tight_layout()
plt.savefig('subplots.png', dpi=100, bbox_inches='tight')
plt.close()
print("✓ 保存了 subplots.png")

# 8. 3D 绘图
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')

# 创建数据
theta = np.linspace(-4*np.pi, 4*np.pi, 100)
z = np.linspace(-2, 2, 100)
r = z**2 + 1
x = r * np.sin(theta)
y = r * np.cos(theta)

ax.plot(x, y, z, linewidth=2)
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
ax.set_title('3D 螺旋线')
plt.savefig('3d_plot.png', dpi=100, bbox_inches='tight')
plt.close()
print("✓ 保存了 3d_plot.png")

print("\n所有图像已保存完成！")
```

### 12.3 SciPy 科学计算

```python
from scipy import stats, optimize, integrate, interpolate, signal

print("=== SciPy 与 NumPy 集成 ===\n")

# 1. 统计分析
data = np.random.randn(1000)

# 正态性检验
statistic, p_value = stats.normaltest(data)
print(f"正态性检验:")
print(f"  统计量: {statistic:.4f}")
print(f"  p-value: {p_value:.4f}")
print(f"  结论: {'数据服从正态分布' if p_value > 0.05 else '数据不服从正态分布'}")

# 描述性统计
desc = stats.describe(data)
print(f"\n描述性统计:")
print(f"  样本数: {desc.nobs}")
print(f"  均值: {desc.mean:.4f}")
print(f"  方差: {desc.variance:.4f}")
print(f"  偏度: {desc.skewness:.4f}")
print(f"  峰度: {desc.kurtosis:.4f}")

# 2. 假设检验
# t 检验
group1 = np.random.randn(50) + 0.5
group2 = np.random.randn(50)

t_stat, p_value = stats.ttest_ind(group1, group2)
print(f"\n独立样本 t 检验:")
print(f"  t 统计量: {t_stat:.4f}")
print(f"  p-value: {p_value:.4f}")
print(f"  结论: {'两组有显著差异' if p_value < 0.05 else '两组无显著差异'}")

# 3. 优化问题
# 最小化函数
def rosenbrock(x):
    """Rosenbrock 函数"""
    return (1 - x[0])**2 + 100*(x[1] - x[0]**2)**2

# 初始猜测
x0 = np.array([0, 0])

# 优化
result = optimize.minimize(rosenbrock, x0, method='BFGS')
print(f"\n函数优化:")
print(f"  初始点: {x0}")
print(f"  最优点: {result.x}")
print(f"  最小值: {result.fun:.10f}")
print(f"  是否成功: {result.success}")

# 4. 数值积分
# 计算定积分 ∫[0,π] sin(x) dx = 2
def integrand(x):
    return np.sin(x)

result, error = integrate.quad(integrand, 0, np.pi)
print(f"\n数值积分:")
print(f"  ∫[0,π] sin(x)dx = {result:.10f}")
print(f"  估计误差: {error:.10e}")
print(f"  真实值: 2.0")

# 5. 插值
# 原始数据点
x = np.array([0, 1, 2, 3, 4, 5])
y = np.array([0, 1, 4, 9, 16, 25])

# 创建插值函数
f_linear = interpolate.interp1d(x, y, kind='linear')
f_cubic = interpolate.interp1d(x, y, kind='cubic')

# 在更密集的点上插值
x_new = np.linspace(0, 5, 50)
y_linear = f_linear(x_new)
y_cubic = f_cubic(x_new)

print(f"\n插值:")
print(f"  原始数据点数: {len(x)}")
print(f"  插值后数据点数: {len(x_new)}")
print(f"  线性插值在 x=2.5 的值: {f_linear(2.5):.2f}")
print(f"  三次插值在 x=2.5 的值: {f_cubic(2.5):.2f}")

# 6. 信号处理
# 生成带噪声的信号
t = np.linspace(0, 1, 1000)
clean_signal = np.sin(2*np.pi*5*t)  # 5Hz 信号
noise = np.random.randn(1000) * 0.5
noisy_signal = clean_signal + noise

# 使用 Savitzky-Golay 滤波器平滑信号
filtered_signal = signal.savgol_filter(noisy_signal, 
                                       window_length=51, 
                                       polyorder=3)

# 计算信噪比改善
snr_original = 10 * np.log10(np.var(clean_signal) / np.var(noise))
snr_filtered = 10 * np.log10(
    np.var(clean_signal) / np.var(filtered_signal - clean_signal)
)

print(f"\n信号处理:")
print(f"  原始信噪比: {snr_original:.2f} dB")
print(f"  滤波后信噪比: {snr_filtered:.2f} dB")
print(f"  改善: {snr_filtered - snr_original:.2f} dB")

# 7. 卷积
# 信号卷积
signal1 = np.array([1, 2, 3, 4, 5])
kernel = np.array([0.2, 0.5, 0.3])

convolved = signal.convolve(signal1, kernel, mode='same')
print(f"\n卷积:")
print(f"  原信号: {signal1}")
print(f"  卷积核: {kernel}")
print(f"  卷积结果: {convolved}")

# 8. 峰值检测
# 生成带峰值的信号
x = np.linspace(0, 10, 1000)
y = np.sin(x) + 0.5*np.sin(3*x) + np.random.randn(1000)*0.1

# 检测峰值
peaks, properties = signal.find_peaks(y, height=0.5, distance=50)

print(f"\n峰值检测:")
print(f"  检测到 {len(peaks)} 个峰值")
print(f"  峰值位置（前5个）: {peaks[:5]}")
print(f"  峰值高度（前5个）: {y[peaks[:5]]}")

# 9. 频谱分析
# 生成组合信号
fs = 1000  # 采样频率
t = np.arange(0, 1, 1/fs)
signal_data = (np.sin(2*np.pi*50*t) +   # 50 Hz
               0.5*np.sin(2*np.pi*120*t))  # 120 Hz

# 计算频谱
frequencies, power = signal.periodogram(signal_data, fs)

# 找出主要频率
peak_indices = signal.find_peaks(power)[0]
main_freqs = frequencies[peak_indices]

print(f"\n频谱分析:")
print(f"  信号长度: {len(signal_data)}")
print(f"  采样频率: {fs} Hz")
print(f"  检测到的主要频率: {main_freqs[main_freqs > 0][:5]} Hz")
```

### 12.4 Scikit-learn 机器学习

```python
from sklearn import datasets, model_selection, preprocessing
from sklearn.linear_model import LinearRegression, LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score, mean_squared_error, r2_score

print("=== Scikit-learn 与 NumPy 集成 ===\n")

# 1. 数据准备
# 生成分类数据
X_class, y_class = datasets.make_classification(
    n_samples=1000, 
    n_features=20, 
    n_informative=15,
    n_redundant=5,
    random_state=42
)

print(f"分类数据集:")
print(f"  样本数: {X_class.shape[0]}")
print(f"  特征数: {X_class.shape[1]}")
print(f"  类别分布: {np.bincount(y_class)}")

# 生成回归数据
X_reg, y_reg = datasets.make_regression(
    n_samples=1000,
    n_features=10,
    noise=10,
    random_state=42
)

print(f"\n回归数据集:")
print(f"  样本数: {X_reg.shape[0]}")
print(f"  特征数: {X_reg.shape[1]}")

# 2. 数据预处理
# 标准化
scaler = preprocessing.StandardScaler()
X_scaled = scaler.fit_transform(X_class)

print(f"\n数据标准化:")
print(f"  原始数据均值: {np.mean(X_class, axis=0)[:3]}")
print(f"  标准化后均值: {np.mean(X_scaled, axis=0)[:3]}")
print(f"  原始数据标准差: {np.std(X_class, axis=0)[:3]}")
print(f"  标准化后标准差: {np.std(X_scaled, axis=0)[:3]}")

# 3. 分割训练集和测试集
X_train, X_test, y_train, y_test = model_selection.train_test_split(
    X_class, y_class, test_size=0.2, random_state=42
)

print(f"\n数据分割:")
print(f"  训练集大小: {X_train.shape[0]}")
print(f"  测试集大小: {X_test.shape[0]}")

# 4. 分类模型
# 逻辑回归
log_reg = LogisticRegression(random_state=42, max_iter=1000)
log_reg.fit(X_train, y_train)
y_pred_lr = log_reg.predict(X_test)
accuracy_lr = accuracy_score(y_test, y_pred_lr)

print(f"\n逻辑回归:")
print(f"  准确率: {accuracy_lr:.4f}")
print(f"  系数形状: {log_reg.coef_.shape}")

# 决策树
dt = DecisionTreeClassifier(random_state=42)
dt.fit(X_train, y_train)
y_pred_dt = dt.predict(X_test)
accuracy_dt = accuracy_score(y_test, y_pred_dt)

print(f"\n决策树:")
print(f"  准确率: {accuracy_dt:.4f}")
print(f"  树深度: {dt.get_depth()}")
print(f"  叶子节点数: {dt.get_n_leaves()}")

# 5. 回归模型
X_train_reg, X_test_reg, y_train_reg, y_test_reg = \
    model_selection.train_test_split(X_reg, y_reg, test_size=0.2, random_state=42)

lin_reg = LinearRegression()
lin_reg.fit(X_train_reg, y_train_reg)
y_pred_reg = lin_reg.predict(X_test_reg)

mse = mean_squared_error(y_test_reg, y_pred_reg)
r2 = r2_score(y_test_reg, y_pred_reg)

print(f"\n线性回归:")
print(f"  均方误差 (MSE): {mse:.4f}")
print(f"  R² 分数: {r2:.4f}")
print(f"  系数: {lin_reg.coef_[:5]}")

# 6. 交叉验证
from sklearn.model_selection import cross_val_score

cv_scores = cross_val_score(log_reg, X_class, y_class, cv=5)

print(f"\n5折交叉验证:")
print(f"  各折准确率: {cv_scores}")
print(f"  平均准确率: {np.mean(cv_scores):.4f}")
print(f"  标准差: {np.std(cv_scores):.4f}")

# 7. 特征重要性（决策树）
feature_importance = dt.feature_importances_
top_features = np.argsort(feature_importance)[-5:][::-1]

print(f"\n特征重要性（前5个）:")
for i, idx in enumerate(top_features, 1):
    print(f"  {i}. 特征 {idx}: {feature_importance[idx]:.4f}")

# 8. 混淆矩阵
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, y_pred_lr)
print(f"\n混淆矩阵:")
print(cm)

# 计算各项指标
tn, fp, fn, tp = cm.ravel()
precision = tp / (tp + fp)
recall = tp / (tp + fn)
f1 = 2 * (precision * recall) / (precision + recall)

print(f"\n分类指标:")
print(f"  精确率 (Precision): {precision:.4f}")
print(f"  召回率 (Recall): {recall:.4f}")
print(f"  F1 分数: {f1:.4f}")

# 9. 网格搜索
from sklearn.model_selection import GridSearchCV

param_grid = {
    'max_depth': [3, 5, 7, 10],
    'min_samples_split': [2, 5, 10],
    'min_samples_leaf': [1, 2, 4]
}

grid_search = GridSearchCV(
    DecisionTreeClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)

grid_search.fit(X_train, y_train)

print(f"\n网格搜索:")
print(f"  最佳参数: {grid_search.best_params_}")
print(f"  最佳分数: {grid_search.best_score_:.4f}")

# 10. 使用最佳模型预测
best_model = grid_search.best_estimator_
y_pred_best = best_model.predict(X_test)
accuracy_best = accuracy_score(y_test, y_pred_best)

print(f"  测试集准确率: {accuracy_best:.4f}")
```

### 12.5 OpenCV 图像处理

```python
# 注意：需要安装 opencv-python
# pip install opencv-python

try:
    import cv2
  
    print("=== OpenCV 与 NumPy 集成 ===\n")
  
    # 1. 创建图像（NumPy 数组）
    # 灰度图像
    gray_image = np.zeros((200, 300), dtype=np.uint8)
    gray_image[50:150, 100:200] = 255  # 白色矩形
  
    print(f"灰度图像:")
    print(f"  形状: {gray_image.shape}")
    print(f"  数据类型: {gray_image.dtype}")
    print(f"  像素值范围: [{gray_image.min()}, {gray_image.max()}]")
  
    # 彩色图像（BGR 格式）
    color_image = np.zeros((200, 300, 3), dtype=np.uint8)
    color_image[50:150, 100:200] = [0, 0, 255]  # 红色矩形（BGR）
  
    print(f"\n彩色图像:")
    print(f"  形状: {color_image.shape}")
    print(f"  通道数: {color_image.shape[2]}")
  
    # 2. 图像操作
    # 绘制图形
    canvas = np.ones((400, 400, 3), dtype=np.uint8) * 255  # 白色背景
    cv2.rectangle(canvas, (50, 50), (150, 150), (0, 255, 0), 2)
    cv2.circle(canvas, (250, 100), 50, (255, 0, 0), -1)
    cv2.line(canvas, (50, 250), (350, 250), (0, 0, 255), 3)
  
    print(f"\n绘制的画布形状: {canvas.shape}")
  
    # 3. 图像变换
    # 缩放
    resized = cv2.resize(canvas, (200, 200))
    print(f"\n缩放后形状: {resized.shape}")
  
    # 旋转
    center = (canvas.shape[1]//2, canvas.shape[0]//2)
    rotation_matrix = cv2.getRotationMatrix2D(center, 45, 1.0)
    rotated = cv2.warpAffine(canvas, rotation_matrix, 
                             (canvas.shape[1], canvas.shape[0]))
    print(f"旋转后形状: {rotated.shape}")
  
    # 4. 颜色空间转换
    # BGR 到 HSV
    hsv_image = cv2.cvtColor(color_image, cv2.COLOR_BGR2HSV)
    print(f"\nHSV 图像形状: {hsv_image.shape}")
  
    # BGR 到灰度
    gray_from_color = cv2.cvtColor(color_image, cv2.COLOR_BGR2GRAY)
    print(f"灰度图像形状: {gray_from_color.shape}")
  
    # 5. 图像滤波
    # 创建带噪声的图像
    noisy_image = np.random.randint(0, 256, (200, 300), dtype=np.uint8)
  
    # 高斯模糊
    blurred = cv2.GaussianBlur(noisy_image, (5, 5), 0)
  
    # 中值滤波
    median_filtered = cv2.medianBlur(noisy_image, 5)
  
    print(f"\n滤波效果:")
    print(f"  原始图像标准差: {np.std(noisy_image):.2f}")
    print(f"  高斯模糊后标准差: {np.std(blurred):.2f}")
    print(f"  中值滤波后标准差: {np.std(median_filtered):.2f}")

    # 6. 边缘检测
    # Canny 边缘检测
    gray_for_edges = np.random.randint(0, 256, (200, 300), dtype=np.uint8)
    edges = cv2.Canny(gray_for_edges, 50, 150)
  
    print(f"\n边缘检测:")
    print(f"  原始图像形状: {gray_for_edges.shape}")
    print(f"  边缘图像形状: {edges.shape}")
    print(f"  检测到的边缘像素数: {np.sum(edges > 0)}")
    print(f"  边缘像素占比: {np.sum(edges > 0) / edges.size * 100:.2f}%")
  
    # 7. 形态学操作
    # 创建二值图像
    binary_image = np.zeros((200, 300), dtype=np.uint8)
    cv2.rectangle(binary_image, (50, 50), (250, 150), 255, -1)
  
    # 定义结构元素
    kernel = np.ones((5, 5), np.uint8)
  
    # 膨胀
    dilated = cv2.dilate(binary_image, kernel, iterations=1)
  
    # 腐蚀
    eroded = cv2.erode(binary_image, kernel, iterations=1)
  
    # 开运算（先腐蚀后膨胀）
    opening = cv2.morphologyEx(binary_image, cv2.MORPH_OPEN, kernel)
  
    # 闭运算（先膨胀后腐蚀）
    closing = cv2.morphologyEx(binary_image, cv2.MORPH_CLOSE, kernel)
  
    print(f"\n形态学操作:")
    print(f"  原始白色像素: {np.sum(binary_image == 255)}")
    print(f"  膨胀后白色像素: {np.sum(dilated == 255)}")
    print(f"  腐蚀后白色像素: {np.sum(eroded == 255)}")
  
    # 8. 直方图均衡化
    # 创建低对比度图像
    low_contrast = np.random.normal(128, 20, (200, 300)).astype(np.uint8)
  
    # 均衡化
    equalized = cv2.equalizeHist(low_contrast)
  
    print(f"\n直方图均衡化:")
    print(f"  原始图像像素值范围: [{low_contrast.min()}, {low_contrast.max()}]")
    print(f"  均衡化后像素值范围: [{equalized.min()}, {equalized.max()}]")
    print(f"  原始图像标准差: {np.std(low_contrast):.2f}")
    print(f"  均衡化后标准差: {np.std(equalized):.2f}")
  
    # 9. 阈值处理
    gray_test = np.random.randint(0, 256, (200, 300), dtype=np.uint8)
  
    # 固定阈值
    _, binary_fixed = cv2.threshold(gray_test, 127, 255, cv2.THRESH_BINARY)
  
    # Otsu 自适应阈值
    _, binary_otsu = cv2.threshold(gray_test, 0, 255, 
                                    cv2.THRESH_BINARY + cv2.THRESH_OTSU)
  
    # 自适应阈值
    binary_adaptive = cv2.adaptiveThreshold(gray_test, 255, 
                                            cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                                            cv2.THRESH_BINARY, 11, 2)
  
    print(f"\n阈值处理:")
    print(f"  固定阈值二值化像素: {np.sum(binary_fixed == 255)}")
    print(f"  Otsu 阈值二值化像素: {np.sum(binary_otsu == 255)}")
    print(f"  自适应阈值二值化像素: {np.sum(binary_adaptive == 255)}")
  
    # 10. 轮廓检测
    # 创建包含形状的图像
    contour_image = np.zeros((200, 300), dtype=np.uint8)
    cv2.rectangle(contour_image, (50, 50), (150, 150), 255, -1)
    cv2.circle(contour_image, (250, 100), 40, 255, -1)
  
    # 检测轮廓
    contours, hierarchy = cv2.findContours(contour_image, 
                                           cv2.RETR_EXTERNAL, 
                                           cv2.CHAIN_APPROX_SIMPLE)
  
    print(f"\n轮廓检测:")
    print(f"  检测到的轮廓数: {len(contours)}")
  
    for i, contour in enumerate(contours):
        area = cv2.contourArea(contour)
        perimeter = cv2.arcLength(contour, True)
        print(f"  轮廓 {i+1}: 面积={area:.0f}, 周长={perimeter:.2f}")
  
    # 11. 图像混合
    img1 = np.ones((200, 300, 3), dtype=np.uint8) * 100
    img2 = np.ones((200, 300, 3), dtype=np.uint8) * 200
  
    # 加权混合
    alpha = 0.3
    blended = cv2.addWeighted(img1, alpha, img2, 1-alpha, 0)
  
    print(f"\n图像混合:")
    print(f"  图像1 平均值: {np.mean(img1):.2f}")
    print(f"  图像2 平均值: {np.mean(img2):.2f}")
    print(f"  混合后平均值: {np.mean(blended):.2f}")
    print(f"  理论值: {100*alpha + 200*(1-alpha):.2f}")
  
    # 12. 图像金字塔
    # 创建测试图像
    pyramid_img = np.random.randint(0, 256, (256, 256, 3), dtype=np.uint8)
  
    # 高斯金字塔（下采样）
    pyramid_down = cv2.pyrDown(pyramid_img)
    pyramid_down_2 = cv2.pyrDown(pyramid_down)
  
    print(f"\n图像金字塔:")
    print(f"  原始图像: {pyramid_img.shape}")
    print(f"  下采样1次: {pyramid_down.shape}")
    print(f"  下采样2次: {pyramid_down_2.shape}")
  
    # 拉普拉斯金字塔
    pyramid_up = cv2.pyrUp(pyramid_down)
    laplacian = cv2.subtract(pyramid_img, pyramid_up[:256, :256])
  
    print(f"  拉普拉斯金字塔形状: {laplacian.shape}")
  
    # 13. 颜色空间分离与合并
    color_img = np.random.randint(0, 256, (100, 100, 3), dtype=np.uint8)
  
    # 分离通道
    b, g, r = cv2.split(color_img)
  
    print(f"\n通道分离:")
    print(f"  B 通道平均值: {np.mean(b):.2f}")
    print(f"  G 通道平均值: {np.mean(g):.2f}")
    print(f"  R 通道平均值: {np.mean(r):.2f}")
  
    # 合并通道
    merged = cv2.merge([b, g, r])
  
    # 验证
    print(f"  原始图像与合并图像是否相同: {np.array_equal(color_img, merged)}")
  
    # 14. 图像统计信息
    test_img = np.random.randint(0, 256, (200, 300, 3), dtype=np.uint8)
  
    print(f"\n图像统计信息:")
    print(f"  形状: {test_img.shape}")
    print(f"  数据类型: {test_img.dtype}")
    print(f"  大小（字节）: {test_img.nbytes}")
    print(f"  最小值: {test_img.min()}")
    print(f"  最大值: {test_img.max()}")
    print(f"  平均值: {test_img.mean():.2f}")
    print(f"  标准差: {test_img.std():.2f}")
  
    # 每个通道的统计
    for i, color in enumerate(['B', 'G', 'R']):
        channel = test_img[:, :, i]
        print(f"  {color} 通道平均值: {channel.mean():.2f}")
  
    # 15. 图像掩码操作
    # 创建图像和掩码
    img = np.random.randint(0, 256, (200, 300, 3), dtype=np.uint8)
    mask = np.zeros((200, 300), dtype=np.uint8)
    cv2.circle(mask, (150, 100), 80, 255, -1)  # 圆形掩码
  
    # 应用掩码
    masked_img = cv2.bitwise_and(img, img, mask=mask)
  
    print(f"\n掩码操作:")
    print(f"  掩码中非零像素: {np.sum(mask > 0)}")
    print(f"  原图像非零像素: {np.sum(img > 0)}")
    print(f"  掩码后非零像素: {np.sum(masked_img > 0)}")
  
    print("\n✓ OpenCV 集成示例完成")
  
except ImportError:
    print("OpenCV 未安装，跳过 OpenCV 示例")
    print("安装命令: pip install opencv-python")
```

---

## 第十三章：实用技巧总结

### 13.1 代码优化清单

```python
print("=== NumPy 代码优化清单 ===\n")

# 1. 优先使用向量化操作
print("1. 向量化 vs 循环\n")

n = 100000
a = np.random.rand(n)
b = np.random.rand(n)

# ❌ 不推荐：使用循环
import time
start = time.time()
result = np.zeros(n)
for i in range(n):
    result[i] = a[i] * b[i]
time_loop = time.time() - start

# ✅ 推荐：使用向量化
start = time.time()
result = a * b
time_vectorized = time.time() - start

print(f"  循环: {time_loop:.4f}s")
print(f"  向量化: {time_vectorized:.4f}s")
print(f"  速度提升: {time_loop/time_vectorized:.0f}x\n")

# 2. 预分配数组
print("2. 预分配 vs 动态增长\n")

n = 10000

# ❌ 不推荐：动态增长
start = time.time()
arr = np.array([])
for i in range(n):
    arr = np.append(arr, i)
time_dynamic = time.time() - start

# ✅ 推荐：预分配
start = time.time()
arr = np.zeros(n)
for i in range(n):
    arr[i] = i
time_preallocated = time.time() - start

print(f"  动态增长: {time_dynamic:.4f}s")
print(f"  预分配: {time_preallocated:.4f}s")
print(f"  速度提升: {time_dynamic/time_preallocated:.0f}x\n")

# 3. 使用 inplace 操作
print("3. Inplace 操作\n")

arr = np.random.rand(1000000)

# ❌ 创建新数组
start = time.time()
result = arr * 2
time_new = time.time() - start

# ✅ 原地修改（节省内存）
start = time.time()
arr *= 2
time_inplace = time.time() - start

print(f"  创建新数组: {time_new:.4f}s")
print(f"  原地修改: {time_inplace:.4f}s\n")

# 4. 使用视图而不是副本
print("4. 视图 vs 副本\n")

large_array = np.random.rand(10000, 10000)

# ✅ 视图（快速，共享内存）
start = time.time()
view = large_array[::2, ::2]
time_view = time.time() - start

# ❌ 副本（慢，复制内存）
start = time.time()
copy = large_array[::2, ::2].copy()
time_copy = time.time() - start

print(f"  视图: {time_view:.6f}s")
print(f"  副本: {time_copy:.4f}s\n")

# 5. 选择合适的数据类型
print("5. 数据类型选择\n")

# ❌ 使用默认 float64
arr_float64 = np.random.rand(1000000)
print(f"  float64 内存: {arr_float64.nbytes / 1024 / 1024:.2f} MB")

# ✅ 使用 float32（如果精度足够）
arr_float32 = np.random.rand(1000000).astype(np.float32)
print(f"  float32 内存: {arr_float32.nbytes / 1024 / 1024:.2f} MB")
print(f"  内存节省: {(1 - arr_float32.nbytes/arr_float64.nbytes)*100:.0f}%\n")

# 6. 使用专门的函数
print("6. 使用专门的函数\n")

arr = np.random.randn(1000000)

# ❌ 通用方法
start = time.time()
result1 = (arr - arr.mean()) / arr.std()
time_manual = time.time() - start

# ✅ 专门的函数
from scipy import stats
start = time.time()
result2 = stats.zscore(arr)
time_specialized = time.time() - start

print(f"  手动计算: {time_manual:.4f}s")
print(f"  专门函数: {time_specialized:.4f}s\n")

# 7. 广播优化
print("7. 广播技巧\n")

arr_2d = np.random.rand(1000, 1000)
arr_1d = np.random.rand(1000)

# ✅ 使用广播
start = time.time()
result = arr_2d + arr_1d
time_broadcast = time.time() - start

# ❌ 显式扩展
start = time.time()
arr_1d_expanded = np.tile(arr_1d, (1000, 1))
result = arr_2d + arr_1d_expanded
time_tile = time.time() - start

print(f"  广播: {time_broadcast:.4f}s")
print(f"  显式扩展: {time_tile:.4f}s\n")

# 8. 使用 ufunc 的 out 参数
print("8. 使用 out 参数\n")

a = np.random.rand(1000000)
b = np.random.rand(1000000)

# ❌ 创建新数组
start = time.time()
result = np.add(a, b)
time_new = time.time() - start

# ✅ 使用 out 参数（复用数组）
result = np.empty_like(a)
start = time.time()
np.add(a, b, out=result)
time_out = time.time() - start

print(f"  创建新数组: {time_new:.4f}s")
print(f"  使用 out: {time_out:.4f}s\n")
```

### 13.2 调试技巧汇总

```python
print("=== NumPy 调试技巧汇总 ===\n")

# 1. 设置错误处理模式
print("1. 错误处理设置\n")

# 设置浮点数错误处理
old_settings = np.seterr(all='warn')  # 所有错误都发出警告

# 测试除零
arr = np.array([1, 2, 0])
with np.errstate(divide='ignore', invalid='ignore'):
    result = 1 / arr
    print(f"  除零结果: {result}")

# 恢复设置
np.seterr(**old_settings)

# 2. 使用 numpy.testing 进行断言
print("\n2. 数值测试\n")

from numpy.testing import assert_almost_equal, assert_array_equal

a = np.array([1.0, 2.0, 3.0])
b = np.array([1.0, 2.0, 3.00001])

try:
    assert_array_equal(a, b)
except AssertionError:
    print("  ✓ 检测到数组不完全相等")

# 允许小误差
assert_almost_equal(a, b, decimal=4)
print("  ✓ 在精度范围内相等\n")

# 3. 检查数组属性
def inspect_detailed(arr, name="Array"):
    """详细检查数组"""
    print(f"{name} 详细信息:")
    print(f"  形状: {arr.shape}")
    print(f"  维度: {arr.ndim}")
    print(f"  大小: {arr.size}")
    print(f"  数据类型: {arr.dtype}")
    print(f"  字节顺序: {arr.dtype.byteorder}")
    print(f"  是否连续: {arr.flags['C_CONTIGUOUS']}")
    print(f"  是否可写: {arr.flags['WRITEABLE']}")
    print(f"  基对象: {arr.base is not None}")
    print(f"  内存地址: {hex(arr.__array_interface__['data'][0])}")
  
    if arr.size > 0:
        print(f"  数值范围: [{np.min(arr)}, {np.max(arr)}]")
        print(f"  平均值: {np.mean(arr):.4f}")
        print(f"  标准差: {np.std(arr):.4f}")
        print(f"  是否有 NaN: {np.any(np.isnan(arr))}")
        print(f"  是否有 Inf: {np.any(np.isinf(arr))}")
    print()

arr = np.random.randn(3, 4)
inspect_detailed(arr, "测试数组")

# 4. 追踪内存使用
print("4. 内存使用追踪\n")

def memory_usage(arr):
    """计算数组内存使用"""
    bytes_used = arr.nbytes
    kb = bytes_used / 1024
    mb = kb / 1024
  
    print(f"  内存使用:")
    print(f"    字节: {bytes_used:,}")
    print(f"    KB: {kb:.2f}")
    print(f"    MB: {mb:.4f}")

large_arr = np.random.rand(1000, 1000)
memory_usage(large_arr)

# 5. 性能分析
print("\n5. 性能分析\n")

def profile_operation(func, *args, iterations=100):
    """性能分析函数"""
    times = []
    for _ in range(iterations):
        start = time.time()
        func(*args)
        times.append(time.time() - start)
  
    times = np.array(times)
    print(f"  运行 {iterations} 次:")
    print(f"    平均时间: {np.mean(times)*1000:.4f} ms")
    print(f"    最小时间: {np.min(times)*1000:.4f} ms")
    print(f"    最大时间: {np.max(times)*1000:.4f} ms")
    print(f"    标准差: {np.std(times)*1000:.4f} ms")

arr = np.random.rand(1000)
profile_operation(np.sum, arr)

# 6. 数据验证函数
print("\n6. 数据验证\n")

def validate_data(arr, name="Data"):
    """验证数据质量"""
    issues = []
  
    # 检查 NaN
    nan_count = np.sum(np.isnan(arr))
    if nan_count > 0:
        issues.append(f"包含 {nan_count} 个 NaN")
  
    # 检查 Inf
    inf_count = np.sum(np.isinf(arr))
    if inf_count > 0:
        issues.append(f"包含 {inf_count} 个 Inf")
  
    # 检查异常值（3σ原则）
    if arr.size > 0:
        mean = np.mean(arr)
        std = np.std(arr)
        outliers = np.sum(np.abs(arr - mean) > 3 * std)
        if outliers > 0:
            issues.append(f"包含 {outliers} 个异常值（3σ）")
  
    if issues:
        print(f"  {name} 数据质量问题:")
        for issue in issues:
            print(f"    - {issue}")
    else:
        print(f"  ✓ {name} 数据质量良好")

# 测试
clean_data = np.random.randn(100)
validate_data(clean_data, "干净数据")

dirty_data = np.array([1, 2, np.nan, 4, np.inf, 100, 7, 8])
validate_data(dirty_data, "脏数据")

# 7. 可视化调试
print("\n7. 快速可视化检查\n")

def quick_plot_check(arr, name="Array"):
    """快速绘制数组用于调试"""
    if arr.ndim == 1:
        print(f"  {name} 是一维数组，长度 {len(arr)}")
        print(f"  前5个值: {arr[:5]}")
        print(f"  后5个值: {arr[-5:]}")
    elif arr.ndim == 2:
        print(f"  {name} 是二维数组，形状 {arr.shape}")
        print(f"  左上角 3×3:")
        print(arr[:3, :3])
    else:
        print(f"  {name} 是 {arr.ndim} 维数组，形状 {arr.shape}")

arr_1d = np.arange(20)
arr_2d = np.random.rand(10, 10)

quick_plot_check(arr_1d, "一维数组")
print()
quick_plot_check(arr_2d, "二维数组")
```

### 13.3 常用代码片段

```python
print("\n=== 常用代码片段 ===\n")

# 1. 数据归一化
print("1. 数据归一化方法\n")

data = np.random.rand(100) * 100

# Min-Max 归一化到 [0, 1]
normalized_minmax = (data - data.min()) / (data.max() - data.min())
print(f"  Min-Max: [{normalized_minmax.min():.2f}, {normalized_minmax.max():.2f}]")

# Z-score 标准化
normalized_zscore = (data - data.mean()) / data.std()
print(f"  Z-score: 均值={normalized_zscore.mean():.2f}, 标准差={normalized_zscore.std():.2f}")

# 缩放到指定范围 [a, b]
a, b = -1, 1
normalized_range = a + (data - data.min()) * (b - a) / (data.max() - data.min())
print(f"  范围缩放: [{normalized_range.min():.2f}, {normalized_range.max():.2f}]\n")

# 2. 缺失值处理
print("2. 缺失值处理\n")

data_with_nan = np.array([1, 2, np.nan, 4, 5, np.nan, 7, 8])

# 删除 NaN
data_no_nan = data_with_nan[~np.isnan(data_with_nan)]
print(f"  删除 NaN: {data_no_nan}")

# 用均值填充
data_filled_mean = data_with_nan.copy()
data_filled_mean[np.isnan(data_filled_mean)] = np.nanmean(data_filled_mean)
print(f"  均值填充: {data_filled_mean}")

# 前向填充
data_ffill = data_with_nan.copy()
mask = np.isnan(data_ffill)
idx = np.where(~mask, np.arange(len(mask)), 0)
np.maximum.accumulate(idx, out=idx)
data_ffill[mask] = data_ffill[idx[mask]]
print(f"  前向填充: {data_ffill}\n")

# 3. 滑动窗口
print("3. 滑动窗口操作\n")

data = np.arange(10)
window_size = 3

# 方法1: 使用 stride tricks
from numpy.lib.stride_tricks import sliding_window_view
windows = sliding_window_view(data, window_size)
print(f"  滑动窗口:\n{windows}")
print(f"  窗口均值: {windows.mean(axis=1)}\n")

# 4. 查找最接近的值
print("4. 查找最接近的值\n")

arr = np.array([1.1, 2.3, 3.5, 4.7, 5.9])
target = 3.2

# 找到最接近的值
closest_idx = np.argmin(np.abs(arr - target))
closest_value = arr[closest_idx]
print(f"  数组: {arr}")
print(f"  目标值: {target}")
print(f"  最接近的值: {closest_value} (索引 {closest_idx})\n")

# 5. 分组统计
print("5. 分组统计\n")

# 数据和分组标签
data = np.array([10, 20, 15, 25, 30, 18, 22])
groups = np.array([0, 1, 0, 1, 0, 1, 0])

# 按组计算均值
unique_groups = np.unique(groups)
group_means = np.array([data[groups == g].mean() for g in unique_groups])
print(f"  数据: {data}")
print(f"  分组: {groups}")
print(f"  各组均值: {group_means}\n")

# 6. 批量处理
print("6. 批量处理大数组\n")

large_data = np.random.rand(10000)
batch_size = 1000

results = []
for i in range(0, len(large_data), batch_size):
    batch = large_data[i:i+batch_size]
    # 处理批次
    batch_result = batch.mean()
    results.append(batch_result)

print(f"  数据大小: {len(large_data)}")
print(f"  批次大小: {batch_size}")
print(f"  批次数: {len(results)}")
print(f"  各批次均值: {results}\n")

# 7. 条件赋值
print("7. 条件赋值\n")

arr = np.array([1, -2, 3, -4, 5, -6])

# 使用 where
result = np.where(arr > 0, arr, 0)  # 负数变为0
print(f"  原数组: {arr}")
print(f"  条件赋值: {result}")

# 使用 clip
clipped = np.clip(arr, 0, 3)  # 限制在 [0, 3]
print(f"  截断到 [0,3]: {clipped}\n")

# 8. 数组去重并保持顺序
print("8. 去重保持顺序\n")

arr = np.array([3, 1, 2, 3, 4, 2, 5])

# 方法1: 使用 unique 的 return_index
_, idx = np.unique(arr, return_index=True)
unique_ordered = arr[np.sort(idx)]
print(f"  原数组: {arr}")
print(f"  去重后: {unique_ordered}\n")

# 9. 矩阵的行列操作
print("9. 矩阵行列操作\n")

matrix = np.random.randint(1, 10, (4, 5))
print(f"  原矩阵:\n{matrix}\n")

# 删除指定行和列
matrix_no_row = np.delete(matrix, 1, axis=0)  # 删除第2行
print(f"  删除第2行:\n{matrix_no_row}\n")

matrix_no_col = np.delete(matrix, [0, 2], axis=1)  # 删除第1和第3列
print(f"  删除第1和第3列:\n{matrix_no_col}\n")

# 10. 快速创建对角矩阵
# 对角矩阵
diag = np.diag([1, 2, 3, 4])
print(f"  对角矩阵:\n{diag}\n")

# 上三角矩阵
upper = np.triu(np.ones((4, 4)))
print(f"  上三角:\n{upper}\n")

# 下三角矩阵
lower = np.tril(np.ones((4, 4)))
print(f"  下三角:\n{lower}\n")
```

---

## 第十四章：快速参考

### 14.1 核心函数速查表

```python
print("=== NumPy 核心函数速查 ===\n")

# 数组创建
print("数组创建:")
print("  np.array() - 从列表创建")
print("  np.zeros(), np.ones() - 全0/全1数组")
print("  np.arange(), np.linspace() - 等差数列")
print("  np.random.rand(), randn() - 随机数组\n")

# 数组属性
print("数组属性:")
print("  .shape, .ndim, .size, .dtype")
print("  .itemsize, .nbytes, .T\n")

# 数组操作
print("数组操作:")
print("  reshape, flatten, ravel")
print("  concatenate, stack, split")
print("  sort, argsort, unique\n")

# 数学运算
print("数学运算:")
print("  +, -, *, /, ** - 逐元素运算")
print("  @ - 矩阵乘法")
print("  sum, mean, std, min, max")
print("  sin, cos, exp, log, sqrt\n")

# 索引切片
print("索引切片:")
print("  arr[i], arr[i:j], arr[::step]")
print("  arr[mask] - 布尔索引")
print("  arr[[1,3,5]] - 花式索引\n")

# 线性代数
print("线性代数:")
print("  np.linalg.inv, det, eig")
print("  np.linalg.solve, lstsq")
print("  np.dot, np.matmul\n")
```

### 14.2 性能对比总结

```python
print("=== 性能优化总结 ===\n")

print("速度提升技巧:")
print("  1. 向量化 > 循环 (10-100x)")
print("  2. 广播 > 显式扩展 (2-5x)")
print("  3. 预分配 > 动态增长 (100x+)")
print("  4. 视图 > 副本 (1000x+)")
print("  5. inplace 操作 > 创建新数组\n")

print("内存优化技巧:")
print("  1. 使用合适的数据类型 (float32 vs float64)")
print("  2. 使用视图避免复制")
print("  3. 及时删除不需要的数组")
print("  4. 使用内存映射处理大文件\n")
```

### 14.3 常见错误与解决

```python
print("=== 常见错误与解决方案 ===\n")

print("1. 维度不匹配")
print("   错误: ValueError: operands could not be broadcast")
print("   解决: 检查数组形状，使用 reshape 或 newaxis\n")

print("2. 数据类型错误")
print("   错误: TypeError: cannot perform operation")
print("   解决: 使用 astype() 转换类型\n")

print("3. 内存不足")
print("   错误: MemoryError")
print("   解决: 使用生成器、分批处理、内存映射\n")

print("4. 索引越界")
print("   错误: IndexError: index out of bounds")
print("   解决: 检查数组大小和索引范围\n")

print("5. 除零警告")
print("   警告: RuntimeWarning: divide by zero")
print("   解决: 使用 np.errstate 或添加条件判断\n")
```

### 14.4 最佳实践清单

```python
print("=== NumPy 最佳实践 ===\n")

print("✓ 代码风格:")
print("  - 优先使用向量化操作")
print("  - 避免显式 Python 循环")
print("  - 使用有意义的变量名")
print("  - 添加必要的注释\n")

print("✓ 性能:")
print("  - 预分配数组空间")
print("  - 使用视图而非副本")
print("  - 选择合适的数据类型")
print("  - 利用广播机制\n")

print("✓ 内存管理:")
print("  - 及时释放不用的数组")
print("  - 大数据使用内存映射")
print("  - 注意视图与副本的区别\n")

print("✓ 调试:")
print("  - 经常检查数组形状和类型")
print("  - 使用 numpy.testing 进行单元测试")
print("  - 处理 NaN 和 Inf")
print("  - 设置合适的错误处理模式\n")

print("✓ 代码可读性:")
print("  - 使用命名索引而非数字")
print("  - 将复杂操作分解为简单步骤")
print("  - 编写可重用的函数")
print("  - 添加类型提示（Python 3.5+）\n")
```

---

## 总结与下一步

```python
print("=" * 60)
print("NumPy 完整教程总结")
print("=" * 60)

print("\n你已经学习了:")
print("  ✓ NumPy 基础概念和数组创建")
print("  ✓ 索引、切片和高级索引")
print("  ✓ 数组操作和变形")
print("  ✓ 数学和统计运算")
print("  ✓ 广播机制")
print("  ✓ 线性代数")
print("  ✓ 随机数生成")
print("  ✓ 文件 I/O")
print("  ✓ 性能优化")
print("  ✓ 与其他库的集成")
print("  ✓ 实用技巧和最佳实践")

print("\n下一步学习建议:")
print("  1. 实践项目: 数据分析、科学计算、图像处理")
print("  2. 深入学习: Pandas (数据分析)")
print("  3. 深入学习: SciPy (科学计算)")
print("  4. 深入学习: Matplotlib (可视化)")
print("  5. 深入学习: Scikit-learn (机器学习)")
print("  6. 阅读 NumPy 官方文档和源码")

print("\n推荐资源:")
print("  - NumPy 官方文档: https://numpy.org/doc/")
print("  - NumPy 用户指南: https://numpy.org/doc/stable/user/")
print("  - NumPy GitHub: https://github.com/numpy/numpy")
print("  - Python 数据科学手册")
print("  - 利用 Python 进行数据分析")

print("\n" + "=" * 60)
print("祝你在 NumPy 和数据科学的道路上越走越远！")
print("=" * 60)
```

---

## 附录：完整示例项目

```python
print("\n=== 综合示例：股票数据分析 ===\n")

# 模拟股票数据
np.random.seed(42)
days = 252  # 一年交易日
dates = np.arange('2024-01-01', '2025-01-01', dtype='datetime64[D]')[:days]
prices = 100 + np.cumsum(np.random.randn(days) * 2)  # 随机游走

print(f"数据概览:")
print(f"  交易日数: {days}")
print(f"  起始价格: ${prices[0]:.2f}")
print(f"  结束价格: ${prices[-1]:.2f}")
print(f"  最高价: ${prices.max():.2f}")
print(f"  最低价: ${prices.min():.2f}")

# 计算收益率
returns = np.diff(prices) / prices[:-1]
print(f"\n收益率统计:")
print(f"  平均日收益率: {np.mean(returns)*100:.4f}%")
print(f"  收益率标准差: {np.std(returns)*100:.4f}%")
print(f"  夏普比率: {np.mean(returns)/np.std(returns)*np.sqrt(252):.2f}")

# 移动平均
ma_20 = np.convolve(prices, np.ones(20)/20, mode='valid')
ma_50 = np.convolve(prices, np.ones(50)/50, mode='valid')

print(f"\n移动平均:")
print(f"  20日均线当前值: ${ma_20[-1]:.2f}")
print(f"  50日均线当前值: ${ma_50[-1]:.2f}")

# 波动率
rolling_std = np.array([prices[i-20:i].std() for i in range(20, len(prices))])
annualized_vol = rolling_std[-1] * np.sqrt(252)
print(f"\n波动率:")
print(f"  当前波动率: {annualized_vol:.2f}%")

# 最大回撤
cummax = np.maximum.accumulate(prices)
drawdown = (prices - cummax) / cummax
max_drawdown = drawdown.min()
print(f"\n风险指标:")
print(f"  最大回撤: {max_drawdown*100:.2f}%")

print("\n分析完成！")
```

**本 NumPy 完整教程到此结束。祝学习愉快！** 🎉