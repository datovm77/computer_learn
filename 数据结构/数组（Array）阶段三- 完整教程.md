# 阶段三：数组（Array）— 完整教程

---

## 第一章：数组简介

### 1.1 什么是数组？

**数组（Array）** 是最基本、最重要的数据结构之一。它是一组**相同类型**的元素，存储在**连续的内存空间**中，并通过**索引（Index）** 来快速访问每个元素。

你可以把数组想象成一排紧挨着的储物柜：

```
索引:    [0]    [1]    [2]    [3]    [4]
内容:   | 10  |  20  |  30  |  40  |  50  |
地址:  0x100  0x104  0x108  0x10C  0x110
```

每个储物柜都有一个编号（索引），你可以直接凭编号打开对应的柜子，而不需要从头一个个找。

### 1.2 数组的核心特征

| 特征                     | 说明                                                            |
| ------------------------ | --------------------------------------------------------------- |
| **同质性**               | 数组中所有元素的数据类型必须相同（全是 `int`、全是 `float` 等） |
| **连续存储**             | 元素在内存中是紧密排列的，中间没有间隙                          |
| **随机访问**             | 可以通过索引在 **O(1)** 时间内直接访问任意元素                  |
| **固定大小**（传统数组） | 创建时大小确定，之后不能改变                                    |
| **零索引**               | C/C++ 中数组索引从 0 开始                                       |

### 1.3 为什么需要数组？

假设你要存储 100 个学生的成绩。没有数组，你需要声明 100 个变量：

```cpp
int score1, score2, score3, /* ... */ score100; // 噩梦！
```

有了数组，一行搞定：

```cpp
int scores[100]; // 优雅！
```

数组的本质是**将一组相关数据聚合在一起管理**，这使得我们可以用循环来批量处理数据，而不是逐个操作。

### 1.4 数组在内存中的样子

当你声明 `int A[5] = {2, 4, 6, 8, 10};` 时，内存中的布局如下（假设 `int` 占 4 字节，起始地址为 200）：

```
地址:    200    204    208    212    216
          ┌──────┬──────┬──────┬──────┬──────┐
    A --> │   2  │   4  │   6  │   8  │  10  │
          └──────┴──────┴──────┴──────┴──────┘
索引:      [0]    [1]    [2]    [3]    [4]
```

**地址计算公式：**

$$\text{Address}(A[i]) = \text{Base Address} + i \times \text{sizeof(element)}$$

例如：`A[3]` 的地址 = 200 + 3 × 4 = **212**

这就是为什么数组能做到 **O(1) 随机访问**——给定索引，通过简单的算术运算就能直接计算出元素的内存地址，不需要遍历。

---

## 第二章：数组的声明与初始化

### 2.1 声明语法

在 C/C++ 中，数组声明的基本语法为：

```cpp
数据类型 数组名[大小];
```

```cpp
int A[5];       // 声明一个包含 5 个 int 的数组
float B[10];    // 声明一个包含 10 个 float 的数组
char C[20];     // 声明一个包含 20 个 char 的数组
```

### 2.2 初始化方式

#### 方式一：声明时完整初始化

```cpp
int A[5] = {2, 4, 6, 8, 10};
```

#### 方式二：声明时部分初始化（剩余元素自动置零）

```cpp
int A[5] = {2, 4}; // A = {2, 4, 0, 0, 0}
```

#### 方式三：全部初始化为零

```cpp
int A[5] = {0}; // A = {0, 0, 0, 0, 0}
int A[5] = {};  // C++ 中同样有效，全部置零
```

#### 方式四：省略大小（编译器自动推断）

```cpp
int A[] = {1, 2, 3, 4, 5}; // 编译器推断大小为 5
```

#### 方式五：声明后逐个赋值

```cpp
int A[5];
A[0] = 2;
A[1] = 4;
// ...
```

#### 方式六：循环赋值

```cpp
int A[5];
for (int i = 0; i < 5; i++) {
    A[i] = (i + 1) * 2; // A = {2, 4, 6, 8, 10}
}
```

### 2.3 C++ 现代写法：`std::array`

C++11 引入了 `std::array`，它是对原生数组的封装，更安全、功能更丰富：

```cpp
#include <array>
#include <iostream>

std::array<int, 5> A = {2, 4, 6, 8, 10};

// 支持 .size()、.at()（带边界检查）、范围 for 循环等
std::cout << A.size();      // 5
std::cout << A.at(2);       // 6（越界会抛异常）
for (int x : A) {
    std::cout << x << " ";  // 2 4 6 8 10
}
```

> **教学说明**：在学习数据结构时，我们主要使用原生数组和指针来理解底层原理。在实际工程中，优先使用 `std::array`（固定大小）或 `std::vector`（动态大小）。

### 2.4 常见错误

```cpp
// ❌ 错误：使用变量作为数组大小（C++ 标准不允许 VLA）
int n;
std::cin >> n;
int A[n]; // 不是标准 C++！部分编译器作为扩展支持，但不可移植

// ✅ 正确做法 1：使用动态分配（记得 delete[]）
int* A = new int[n];
// ... 使用 A
delete[] A;

// ✅ 正确做法 2（更推荐）：使用 std::vector
std::vector<int> B(n);
```

```cpp
// ❌ 错误：越界访问
int A[5] = {1, 2, 3, 4, 5};
std::cout << A[5]; // 未定义行为！有效索引是 0~4
```

---

## 第三章：静态数组与动态数组

这是理解数组的**核心分水岭**。你必须清楚：数组可以创建在**栈（Stack）** 上，也可以创建在**堆（Heap）** 上。

### 3.1 静态数组（Stack 上的数组）

```cpp
int A[5] = {1, 2, 3, 4, 5};
```

- **存储位置**：栈（Stack）
- **大小**：编译时确定，运行时不可改变
- **生命周期**：随函数调用结束而自动销毁
- **无需手动释放**

```
┌─────────────────────────────────┐
│           Stack 栈               │
│                                 │
│   ┌───┬───┬───┬───┬───┐        │
│   │ 1 │ 2 │ 3 │ 4 │ 5 │  A    │
│   └───┴───┴───┴───┴───┘        │
│                                 │
└─────────────────────────────────┘
```

### 3.2 动态数组（Heap 上的数组）

```cpp
int* A = new int[5]; // 在堆上分配 5 个 int 的空间
A[0] = 1; A[1] = 2; A[2] = 3; A[3] = 4; A[4] = 5;

// 使用完毕后必须手动释放！
delete[] A;
A = nullptr; // 好习惯：释放后将指针置空
```

- **存储位置**：堆（Heap）
- **大小**：运行时决定，可以用变量指定
- **生命周期**：直到你手动 `delete[]` 才释放
- **必须手动释放**，否则内存泄漏

```
┌──────────────────────┐     ┌──────────────────────────┐
│      Stack 栈         │     │         Heap 堆           │
│                      │     │                          │
│   A (指针) ──────────┼────>│ ┌───┬───┬───┬───┬───┐   │
│   [存储堆地址]       │     │ │ 1 │ 2 │ 3 │ 4 │ 5 │   │
│                      │     │ └───┴───┴───┴───┴───┘   │
└──────────────────────┘     └──────────────────────────┘
```

### 3.3 关键对比

| 特性     | 静态数组    | 动态数组               |
| -------- | ----------- | ---------------------- |
| 声明     | `int A[5];` | `int* A = new int[5];` |
| 存储位置 | 栈          | 堆                     |
| 大小     | 编译时固定  | 运行时决定             |
| 释放     | 自动        | 手动 `delete[]`        |
| 可否扩容 | ❌ 不可以    | ✅ 可以（通过重新分配） |

### 3.4 完整演示

```cpp
#include <iostream>

int main() {
    // === 静态数组 ===
    int staticArr[5] = {10, 20, 30, 40, 50};
    std::cout << "静态数组: ";
    for (int i = 0; i < 5; i++) {
        std::cout << staticArr[i] << " ";
    }
    std::cout << "\n";

    // === 动态数组 ===
    int size;
    std::cout << "请输入动态数组大小: ";
    std::cin >> size;

    int* dynamicArr = new int[size];
    for (int i = 0; i < size; i++) {
        dynamicArr[i] = (i + 1) * 10;
    }

    std::cout << "动态数组: ";
    for (int i = 0; i < size; i++) {
        std::cout << dynamicArr[i] << " ";
    }
    std::cout << "\n";

    delete[] dynamicArr;   // 必须释放！
    dynamicArr = nullptr;

    return 0;
}
```

---

## 第四章：如何增加数组大小（动态扩容）

### 4.1 核心问题

数组一旦创建，其大小就固定了——无论是栈上还是堆上。如果我们需要更多空间怎么办？

**答案：创建一个更大的新数组，把旧数据复制过去，然后释放旧数组。**

这就是**动态扩容**的基本思想，也是 `std::vector` 内部的核心机制。

### 4.2 扩容算法步骤

```
1. 创建一个更大的新数组（例如原来的 2 倍）
2. 将旧数组的所有元素复制到新数组
3. 释放旧数组的内存
4. 让指针指向新数组
```

图解：

```
旧数组（大小 5）：
┌───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │
└───┴───┴───┴───┴───┘
        │
        │ 复制
        ▼
新数组（大小 10）：
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │   │   │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

然后释放旧数组，指针指向新数组。
```

### 4.3 代码实现

```cpp
#include <iostream>

int main() {
    int oldSize = 5;
    int newSize = 10;

    // 1. 创建旧数组并填充数据
    int* arr = new int[oldSize]{1, 2, 3, 4, 5};

    // 2. 创建更大的新数组
    int* newArr = new int[newSize]{};  // 零初始化

    // 3. 复制旧数据到新数组
    for (int i = 0; i < oldSize; i++) {
        newArr[i] = arr[i];
    }

    // 4. 释放旧数组
    delete[] arr;

    // 5. 让 arr 指向新数组
    arr = newArr;
    newArr = nullptr;  // newArr 不再需要

    // 现在 arr 指向大小为 10 的数组，前 5 个元素是旧数据
    std::cout << "扩容后的数组: ";
    for (int i = 0; i < newSize; i++) {
        std::cout << arr[i] << " ";
    }
    // 输出: 1 2 3 4 5 0 0 0 0 0

    delete[] arr;
    return 0;
}
```

### 4.4 时间复杂度分析

- 扩容一次的代价：**O(n)**（需要复制 n 个元素）
- 这就是为什么 `std::vector` 采用**倍增策略**（每次扩容翻倍），使得**均摊（Amortized）** 插入的时间复杂度为 **O(1)**

> **深入理解**：如果每次只增加 1 个位置，那么连续插入 n 个元素的总复制次数为 1+2+3+...+n = O(n²)。而如果每次翻倍，总复制次数为 1+2+4+8+...+n ≈ 2n = O(n)，均摊后每次插入只需 O(1)。这是一个非常经典的均摊分析案例。

---

## 第五章：二维数组

### 5.1 什么是二维数组？

二维数组可以看作"数组的数组"，或者直观地理解为一个**矩阵（表格）**。

```cpp
int A[3][4]; // 3 行 4 列的矩阵
```

```
          列0   列1   列2   列3
行0 →   [  1  |  2  |  3  |  4  ]
行1 →   [  5  |  6  |  7  |  8  ]
行2 →   [  9  | 10  | 11  | 12  ]
```

### 5.2 声明与初始化

```cpp
// 方式一：完整初始化
int A[3][4] = {
    {1,  2,  3,  4},
    {5,  6,  7,  8},
    {9, 10, 11, 12}
};

// 方式二：平铺初始化（按行填充）
int B[3][4] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};

// 方式三：部分初始化（未指定元素为 0）
int C[3][4] = {{1, 2}, {5}};
// C = {{1,2,0,0}, {5,0,0,0}, {0,0,0,0}}

// 访问元素
std::cout << A[1][2]; // 第 1 行第 2 列 → 7
```

### 5.3 二维数组在堆上的创建

```cpp
// 方法一：指针数组
int** A = new int*[3]; // 3 个 int* 指针
for (int i = 0; i < 3; i++) {
    A[i] = new int[4]; // 每行 4 个 int
}

// 使用
A[1][2] = 7;

// 释放（注意顺序：先释放每一行，再释放指针数组）
for (int i = 0; i < 3; i++) {
    delete[] A[i];
}
delete[] A;
```

注意这种方式的内存**不一定连续**：

```
Stack                Heap
┌─────┐
│  A ─┼────→ ┌────────┐
└─────┘      │ A[0] ──┼──→ [1][2][3][4]  （可能在堆的某处）
             │ A[1] ──┼──→ [5][6][7][8]  （可能在堆的另一处）
             │ A[2] ──┼──→ [9][10][11][12]
             └────────┘
```

```cpp
// 方法二：一维数组模拟二维（内存连续，性能更好）
int rows = 3, cols = 4;
int* A = new int[rows * cols];

// 访问 A[i][j] → A[i * cols + j]
A[1 * 4 + 2] = 7; // 等价于 A[1][2] = 7

delete[] A;
```

### 5.4 遍历二维数组

```cpp
int A[3][4] = {
    {1,  2,  3,  4},
    {5,  6,  7,  8},
    {9, 10, 11, 12}
};

// 行优先遍历
for (int i = 0; i < 3; i++) {       // 外层循环：行
    for (int j = 0; j < 4; j++) {   // 内层循环：列
        std::cout << A[i][j] << "\t";
    }
    std::cout << "\n";
}
```

---

## 第六章：编译器的数组表示与地址计算公式

这一章是理论重点，是考试和面试的常考内容。

### 6.1 编译器如何存储二维数组？

虽然我们在逻辑上把二维数组看作一个矩阵，但**内存是一维的**。编译器必须把二维矩阵"展平"成一维来存储。有两种方式：

#### 行主序（Row Major Order）

**先存第 0 行的所有元素，再存第 1 行，以此类推。**

C/C++、Java、Python 默认使用行主序。

```
逻辑视图：            内存中的实际存储：
[1  2  3  4]          [1][2][3][4][5][6][7][8][9][10][11][12]
[5  6  7  8]           ──行0──── ──行1──── ───行2────
[9 10 11 12]
```

#### 列主序（Column Major Order）

**先存第 0 列的所有元素，再存第 1 列，以此类推。**

Fortran、MATLAB 默认使用列主序。

```
逻辑视图：            内存中的实际存储：
[1  2  3  4]          [1][5][9][2][6][10][3][7][11][4][8][12]
[5  6  7  8]           ─列0── ──列1─── ──列2─── ──列3──
[9 10 11 12]
```

### 6.2 行主序地址公式

对于一个 `m × n` 的二维数组 `A[m][n]`（索引从 0 开始）：

$$\text{Address}(A[i][j]) = \text{Base} + (i \times n + j) \times w$$

其中：
- `Base` = 数组起始地址
- `i` = 行索引
- `j` = 列索引
- `n` = 每行的列数
- `w` = 每个元素的字节数（`sizeof`）

**直觉理解**：要到达 `A[i][j]`，需要跳过 `i` 整行（每行 `n` 个元素）再加上当前行的 `j` 个元素。

**例题**：数组 `int A[3][4]`，基地址为 200，`int` 占 4 字节。求 `A[2][1]` 的地址。

$$\text{Address}(A[2][1]) = 200 + (2 \times 4 + 1) \times 4 = 200 + 9 \times 4 = 200 + 36 = 236$$

验证：

```
A[0][0]=200, A[0][1]=204, A[0][2]=208, A[0][3]=212,
A[1][0]=216, A[1][1]=220, A[1][2]=224, A[1][3]=228,
A[2][0]=232, A[2][1]=236 ✓
```

#### 如果索引不从 0 开始？

假设数组声明为 `A[l1..u1][l2..u2]`（下界 `l`，上界 `u`），则：

$$\text{Address}(A[i][j]) = \text{Base} + [(i - l_1) \times (u_2 - l_2 + 1) + (j - l_2)] \times w$$

### 6.3 列主序地址公式

对于 `m × n` 的数组 `A[m][n]`（索引从 0 开始）：

$$\text{Address}(A[i][j]) = \text{Base} + (j \times m + i) \times w$$

**直觉理解**：在列主序中，先按列存储。要到达 `A[i][j]`，需要跳过 `j` 整列（每列 `m` 个元素）再加上当前列的 `i` 个元素。

**例题**：同样 `int A[3][4]`，基地址 200，求 `A[2][1]` 在列主序下的地址：

$$\text{Address}(A[2][1]) = 200 + (1 \times 3 + 2) \times 4 = 200 + 5 \times 4 = 220$$

#### 通用形式（索引从 `l` 开始）：

$$\text{Address}(A[i][j]) = \text{Base} + [(j - l_2) \times (u_1 - l_1 + 1) + (i - l_1)] \times w$$

### 6.4 对比记忆

|                | 行主序                      | 列主序                      |
| -------------- | --------------------------- | --------------------------- |
| 外层跳过       | 跳过的**行**数 × 每行元素数 | 跳过的**列**数 × 每列元素数 |
| 公式 (0-based) | `Base + (i*n + j)*w`        | `Base + (j*m + i)*w`        |
| 谁先变         | j 先变（同一行内列号递增）  | i 先变（同一列内行号递增）  |
| 使用者         | C/C++, Java, Python         | Fortran, MATLAB             |

---

## 第七章：n 维数组与三维数组公式

### 7.1 三维数组

三维数组可以想象为"多个矩阵叠在一起"：

```cpp
int A[2][3][4]; // 2 个 3×4 的矩阵
```

```
层 0：                层 1：
┌─────────────┐      ┌─────────────┐
│  1  2  3  4 │      │ 13 14 15 16 │
│  5  6  7  8 │      │ 17 18 19 20 │
│  9 10 11 12 │      │ 21 22 23 24 │
└─────────────┘      └─────────────┘
```

### 7.2 三维数组地址公式（行主序，0-based）

对于 `A[d1][d2][d3]`：

$$\text{Address}(A[i][j][k]) = \text{Base} + (i \times d_2 \times d_3 + j \times d_3 + k) \times w$$

**直觉理解**：
- 跳过 `i` 个完整的二维矩阵（每个矩阵 `d2 × d3` 个元素）
- 在当前矩阵中，跳过 `j` 行（每行 `d3` 个元素）
- 在当前行中，偏移 `k` 个位置

**例题**：`int A[2][3][4]`，基地址 = 100。求 `A[1][2][3]` 的地址：

$$\text{Address} = 100 + (1 \times 3 \times 4 + 2 \times 4 + 3) \times 4 = 100 + (12 + 8 + 3) \times 4 = 100 + 92 = 192$$

### 7.3 n维数组通用公式

对于 n 维数组 `A[d1][d2]...[dn]`，元素 `A[i1][i2]...[in]` 的地址（行主序，0-based）：

$$\text{Address} = \text{Base} + \left(\sum_{k=1}^{n} i_k \times \prod_{j=k+1}^{n} d_j \right) \times w$$

换一种更直观的写法：

$$\text{offset} = i_1 \times (d_2 \times d_3 \times ... \times d_n) + i_2 \times (d_3 \times d_4 \times ... \times d_n) + ... + i_{n-1} \times d_n + i_n$$

本质就是一个**多进制数转十进制**的过程。每一维就像一个"进制位"，权重是后续所有维度大小的乘积。

---

## 第八章：数组抽象数据类型（ADT）

### 8.1 什么是 ADT？

**抽象数据类型（Abstract Data Type, ADT）** 定义了数据结构的：
1. **数据（Data）** ——存储什么
2. **操作（Operations）** ——能做什么

ADT 只关心"做什么"，不关心"怎么做"。它是接口，不是实现。

### 8.2 数组 ADT 的定义

```
ADT Array:
    Data:
        - 一个用于存储元素的数组空间 A
        - 数组的总容量 size
        - 数组中实际存储的元素个数 length

    Operations:
        - Display()         显示所有元素
        - Append(x)         在末尾添加元素 x
        - Insert(index, x)  在指定位置插入元素 x
        - Delete(index)     删除指定位置的元素
        - Search(x)         搜索元素 x
        - Get(index)        获取指定位置的元素
        - Set(index, x)     设置指定位置的元素
        - Max()             返回最大值
        - Min()             返回最小值
        - Sum() / Avg()     返回总和 / 平均值
        - Reverse()         反转数组
        - Shift() / Rotate()移位 / 旋转
        - IsSorted()        检查是否有序
        - Merge(arr2)       合并两个有序数组
        - Union / Intersection / Difference  集合操作
```

### 8.3 用结构体实现数组 ADT

```cpp
struct Array {
    int* A;       // 指向堆上数组空间的指针
    int size;     // 总容量
    int length;   // 当前元素个数
};
```

**关键理解**：`size` 和 `length` 是不同的！

```
size = 10（总容量为 10）
length = 5（实际存了 5 个元素）

┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 2 │ 4 │ 6 │ 8 │10 │   │   │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
 [0] [1] [2] [3] [4] [5] [6] [7] [8] [9]
 ←── length=5 ───→
 ←──────── size=10 ──────────────────→
```

### 8.4 完整 ADT 框架代码

```cpp
#include <iostream>

struct Array {
    int* A;
    int size;
    int length;
};

// 显示数组
void Display(const Array& arr) {
    std::cout << "数组元素: ";
    for (int i = 0; i < arr.length; i++) {
        std::cout << arr.A[i] << " ";
    }
    std::cout << "\n";
    std::cout << "长度: " << arr.length << ", 容量: " << arr.size << "\n";
}

// 在末尾添加元素
void Append(Array& arr, int x) {
    if (arr.length < arr.size) {
        arr.A[arr.length] = x;
        arr.length++;
    } else {
        std::cout << "数组已满,无法添加!\n";
    }
}

int main() {
    Array arr;
    arr.size = 10;
    arr.length = 0;
    arr.A = new int[arr.size]{};

    Append(arr, 5);
    Append(arr, 10);
    Append(arr, 15);
    Display(arr);
    // 输出: 数组元素: 5 10 15
    //       长度: 3, 容量: 10

    delete[] arr.A;
    return 0;
}
```

---

## 第九章：数组中插入元素

### 9.1 在索引 `index` 处插入元素 `x`

**核心思路**：将 `index` 及之后的所有元素向后移动一位，然后将 `x` 放到 `index` 位置。

```
插入前:  [2, 4, 6, 8, 10, _, _, _]     length=5
在 index=2 插入 x=5

步骤1: 从后往前移动元素
         [2, 4, 6, 8, 10, _, _, _]
                      ↓        ↓
         [2, 4, 6, 6, 8, 10, _, _]  // 先移 10，再移 8，再移 6

步骤2: 放入新元素
         [2, 4, 5, 6, 8, 10, _, _]     length=6
```

### 9.2 代码实现

```cpp
void Insert(Array& arr, int index, int x) {
    // 边界检查
    if (index < 0 || index > arr.length) {
        std::cout << "无效的索引!\n";
        return;
    }
    if (arr.length >= arr.size) {
        std::cout << "数组已满!\n";
        return;
    }

    // 从后往前，逐个后移
    for (int i = arr.length; i > index; i--) {
        arr.A[i] = arr.A[i - 1];
    }

    arr.A[index] = x;
    arr.length++;
}
```

### 9.3 时间复杂度

| 场景           | 时间复杂度                       |
| -------------- | -------------------------------- |
| 在**末尾**插入 | **O(1)**（不需要移动任何元素）   |
| 在**开头**插入 | **O(n)**（所有元素都要后移一位） |
| 在**中间**插入 | **O(n)**（平均移动 n/2 个元素）  |
| **最坏情况**   | **O(n)**                         |
| **最好情况**   | **O(1)**                         |

---

## 第十章：从数组中删除元素

### 10.1 删除索引为 `index` 的元素

**核心思路**：与插入相反。保存要删除的值，将 `index` 之后的元素全部前移一位。

```
删除前:  [2, 4, 5, 6, 8, 10]     length=6
删除 index=2 的元素 (值为 5)

步骤1: 保存被删除的值 deleted = A[2] = 5

步骤2: 从 index 开始，逐个前移
         [2, 4, 6, 8, 10, 10]  // 6前移，8前移，10前移

步骤3: length--
         [2, 4, 6, 8, 10, _]     length=5
```

### 10.2 代码实现

```cpp
int Delete(Array& arr, int index) {
    if (index < 0 || index >= arr.length) {
        std::cout << "无效的索引!\n";
        return -1; // 或使用 std::optional
    }

    int deleted = arr.A[index]; // 保存被删除的值

    // 从 index 开始，逐个前移
    for (int i = index; i < arr.length - 1; i++) {
        arr.A[i] = arr.A[i + 1];
    }

    arr.length--;
    return deleted;
}
```

### 10.3 时间复杂度

与插入完全相同：
- 删除末尾元素：**O(1)**
- 删除开头元素：**O(n)**
- 平均情况：**O(n)**

---

## 第十一章：线性搜索

### 11.1 基本线性搜索

**思路**：从头到尾逐个检查，找到目标就返回索引。

```cpp
int LinearSearch(const Array& arr, int key) {
    for (int i = 0; i < arr.length; i++) {
        if (arr.A[i] == key) {
            return i; // 找到了，返回索引
        }
    }
    return -1; // 没找到
}
```

**时间复杂度**：
- 最好情况：O(1)（第一个就是）
- 最坏情况：O(n)（最后一个或不存在）
- 平均情况：O(n)

### 11.2 改进线性搜索

如果某些元素被频繁搜索，我们可以让它们"靠前"，从而加快后续搜索。有两种策略：

#### 策略一：交换法（Transposition）

找到元素后，将它与**前一个元素**交换位置。这样每搜索一次，目标就向前移动一步。

```cpp
int LinearSearchTransposition(Array& arr, int key) {
    for (int i = 0; i < arr.length; i++) {
        if (arr.A[i] == key) {
            if (i > 0) { // 不是第一个才交换
                std::swap(arr.A[i], arr.A[i - 1]);
                return i - 1; // 返回交换后的新位置
            }
            return i;
        }
    }
    return -1;
}
```

**优点**：渐进式改进，稳定
**缺点**：需要多次搜索才能把元素移到前面

#### 策略二：前移法（Move to Front / Head）

找到元素后，直接将它移到**数组最前面**。

```cpp
int LinearSearchMoveToFront(Array& arr, int key) {
    for (int i = 0; i < arr.length; i++) {
        if (arr.A[i] == key) {
            if (i > 0) {
                std::swap(arr.A[i], arr.A[0]);
                return 0;
            }
            return i;
        }
    }
    return -1;
}
```

**优点**：一步到位，高频元素立刻获得最快访问速度
**缺点**：如果不同的元素交替被搜索，频繁交换可能导致性能下降

---

## 第十二章：二分查找

### 12.1 前提条件

**二分查找只能用于已排序的数组！**

### 12.2 核心思想

每次比较中间元素，根据比较结果将搜索范围缩小一半。

```
在有序数组 [3, 6, 8, 12, 14, 17, 25, 29, 31, 36, 42, 53] 中查找 14

第一轮: low=0, high=11, mid=5
        A[5]=17, 14 < 17 → 搜索左半部分, high=4

第二轮: low=0, high=4, mid=2
        A[2]=8, 14 > 8 → 搜索右半部分, low=3

第三轮: low=3, high=4, mid=3
        A[3]=12, 14 > 12 → low=4

第四轮: low=4, high=4, mid=4
        A[4]=14 → 找到了！返回索引 4
```

### 12.3 迭代实现

```cpp
int BinarySearch(const Array& arr, int key) {
    int low = 0;
    int high = arr.length - 1;

    while (low <= high) {
        int mid = low + (high - low) / 2; // 防止溢出！
        
        if (arr.A[mid] == key) {
            return mid;
        } else if (key < arr.A[mid]) {
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return -1; // 未找到
}
```

> **注意**：计算 `mid` 时使用 `low + (high - low) / 2` 而不是 `(low + high) / 2`，是为了防止 `low + high` 超过 `int` 最大值导致整数溢出。这是一个经典的细节。

### 12.4 递归实现

```cpp
int BinarySearchRecursive(const int A[], int low, int high, int key) {
    if (low > high) {
        return -1;
    }

    int mid = low + (high - low) / 2;

    if (A[mid] == key) {
        return mid;
    } else if (key < A[mid]) {
        return BinarySearchRecursive(A, low, mid - 1, key);
    } else {
        return BinarySearchRecursive(A, mid + 1, high, key);
    }
}
```

### 12.5 时间复杂度分析

每次比较后，搜索范围减半：

```
n → n/2 → n/4 → n/8 → ... → 1
```

假设经过 k 轮比较后剩 1 个元素：

$$\frac{n}{2^k} = 1 \implies n = 2^k \implies k = \log_2 n$$

| 场景     | 时间复杂度                   |
| -------- | ---------------------------- |
| 最好情况 | **O(1)**（中间元素就是目标） |
| 最坏情况 | **O(log n)**                 |
| 平均情况 | **O(log n)**                 |

**空间复杂度**：
- 迭代版：O(1)
- 递归版：O(log n)（递归栈深度）

### 12.6 平均情况的精确分析

对于一个大小为 n 的有序数组（取 n = 2^k - 1 使树完全平衡），我们可以画一棵**二分查找树**：

```
对于 n=15 的数组（索引 0~14）：

                    第 1 次比较找到
                        [7]
                       /    \
              第 2 次  [3]    [11]  第 2 次
                      / \    /  \
            第 3 次 [1] [5] [9] [13]  第 3 次
                   /\ /\ /\  /\
            第 4 次 [0][2][4][6][8][10][12][14]  第 4 次
```

- 1 个元素需要 1 次比较
- 2 个元素需要 2 次比较
- 4 个元素需要 3 次比较
- 8 个元素需要 4 次比较

平均比较次数：

$$\text{Avg} = \frac{1 \times 1 + 2 \times 2 + 3 \times 4 + 4 \times 8}{15} = \frac{1 + 4 + 12 + 32}{15} = \frac{49}{15} \approx 3.27$$

一般化公式（对于 n = 2^k - 1）：

$$\text{Avg} = \frac{\sum_{i=1}^{k} i \times 2^{i-1}}{2^k - 1} \approx \log_2 n - 1$$

所以平均情况的时间复杂度确实是 **O(log n)**。

### 12.7 线性搜索 vs 二分查找

|                            | 线性搜索         | 二分查找       |
| -------------------------- | ---------------- | -------------- |
| 前提                       | 无需排序         | 必须排序       |
| 最坏时间                   | O(n)             | O(log n)       |
| 实现复杂度                 | 简单             | 稍复杂         |
| 适用场景                   | 小数组、无序数组 | 大数组、已排序 |
| n=1,000,000 时最坏比较次数 | 1,000,000        | ~20            |

---

## 第十三章：Get、Set、Max、Min、Sum、Avg 操作

这些是数组的基础操作，实现简单但非常常用。

### 13.1 Get 和 Set

```cpp
// 获取指定索引的元素
int Get(const Array& arr, int index) {
    if (index >= 0 && index < arr.length) {
        return arr.A[index];
    }
    return -1; // 错误处理（实际项目中应使用异常或 std::optional）
}

// 设置指定索引的元素
void Set(Array& arr, int index, int x) {
    if (index >= 0 && index < arr.length) {
        arr.A[index] = x;
    }
}
```

**时间复杂度**：O(1) —— 直接通过索引访问，这是数组的最大优势。

### 13.2 Max 和 Min

```cpp
#include <climits>

int Max(const Array& arr) {
    if (arr.length == 0) return INT_MIN; // 空数组

    int maxVal = arr.A[0];
    for (int i = 1; i < arr.length; i++) {
        if (arr.A[i] > maxVal) {
            maxVal = arr.A[i];
        }
    }
    return maxVal;
}

int Min(const Array& arr) {
    if (arr.length == 0) return INT_MAX;

    int minVal = arr.A[0];
    for (int i = 1; i < arr.length; i++) {
        if (arr.A[i] < minVal) {
            minVal = arr.A[i];
        }
    }
    return minVal;
}
```

**时间复杂度**：O(n) —— 必须遍历整个数组。

### 13.3 Sum 和 Avg

```cpp
int Sum(const Array& arr) {
    int total = 0;
    for (int i = 0; i < arr.length; i++) {
        total += arr.A[i];
    }
    return total;
}

double Avg(const Array& arr) {
    if (arr.length == 0) return 0.0;
    return static_cast<double>(Sum(arr)) / arr.length;
}
```

**时间复杂度**：O(n)

---

## 第十四章：数组的反转与移动操作

### 14.1 反转数组

#### 方法一：使用辅助数组

```cpp
void ReverseWithAux(Array& arr) {
    int* B = new int[arr.length];

    // 反向复制到辅助数组
    for (int i = arr.length - 1, j = 0; i >= 0; i--, j++) {
        B[j] = arr.A[i];
    }

    // 复制回原数组
    for (int i = 0; i < arr.length; i++) {
        arr.A[i] = B[i];
    }

    delete[] B;
}
```

- 时间复杂度：O(n)
- 空间复杂度：O(n)

#### 方法二：原地交换（双指针法）— 最优解

```cpp
void Reverse(Array& arr) {
    int left = 0;
    int right = arr.length - 1;

    while (left < right) {
        std::swap(arr.A[left], arr.A[right]);
        left++;
        right--;
    }
}
```

```
原数组:     [2, 3, 5, 8, 10]
             L→            ←R

第 1 轮:    [10, 3, 5, 8, 2]      交换 2 和 10
                L→      ←R

第 2 轮:    [10, 8, 5, 3, 2]      交换 3 和 8
                   L=R            L 和 R 相遇，结束

结果:       [10, 8, 5, 3, 2]
```

- 时间复杂度：O(n)
- 空间复杂度：**O(1)** ✓ 最优

### 14.2 左移操作（Shift Left）

所有元素向左移一位，第一个元素丢失，末尾补零。

```cpp
void ShiftLeft(Array& arr) {
    if (arr.length == 0) return;

    for (int i = 0; i < arr.length - 1; i++) {
        arr.A[i] = arr.A[i + 1];
    }
    arr.A[arr.length - 1] = 0; // 末尾补零
}
```

```
[2, 4, 6, 8, 10] → [4, 6, 8, 10, 0]
```

### 14.3 左旋转操作（Rotate Left）

与左移类似，但第一个元素不丢失，而是移到末尾（循环）。

```cpp
void RotateLeft(Array& arr) {
    if (arr.length == 0) return;

    int first = arr.A[0]; // 保存第一个元素
    for (int i = 0; i < arr.length - 1; i++) {
        arr.A[i] = arr.A[i + 1];
    }
    arr.A[arr.length - 1] = first; // 放到末尾
}
```

```
[2, 4, 6, 8, 10] → [4, 6, 8, 10, 2]
```

右移和右旋转建议给出完整实现，避免读者自行补全时出现越界或覆盖问题：

```cpp
void ShiftRight(Array& arr) {
    if (arr.length == 0) return;

    for (int i = arr.length - 1; i > 0; i--) {
        arr.A[i] = arr.A[i - 1];
    }
    arr.A[0] = 0; // 开头补零
}

void RotateRight(Array& arr) {
    if (arr.length == 0) return;

    int last = arr.A[arr.length - 1];
    for (int i = arr.length - 1; i > 0; i--) {
        arr.A[i] = arr.A[i - 1];
    }
    arr.A[0] = last;
}
```

---

## 第十五章：检查数组是否有序

### 15.1 判断升序

```cpp
bool IsSorted(const Array& arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        if (arr.A[i] > arr.A[i + 1]) {
            return false; // 发现逆序对，不是升序
        }
    }
    return true;
}
```

**时间复杂度**：O(n)

### 15.2 有序数组的插入优化

如果我们知道数组是有序的，那么在插入新元素时可以使用**插入排序**的思想，保持有序性：

```cpp
void InsertSorted(Array& arr, int x) {
    if (arr.length >= arr.size) return;

    int i = arr.length - 1;
    // 从后往前，将比 x 大的元素后移
    while (i >= 0 && arr.A[i] > x) {
        arr.A[i + 1] = arr.A[i];
        i--;
    }
    arr.A[i + 1] = x;
    arr.length++;
}
```

如果数组是**降序**，判定条件需要反过来：

```cpp
void InsertSortedDescending(Array& arr, int x) {
    if (arr.length >= arr.size) return;

    int i = arr.length - 1;
    while (i >= 0 && arr.A[i] < x) {
        arr.A[i + 1] = arr.A[i];
        i--;
    }
    arr.A[i + 1] = x;
    arr.length++;
}
```

```
有序数组:   [3, 7, 12, 18, 25]
插入 10:
           [3, 7, 12, 18, 25, _]
                    ↓        ↓
           [3, 7, 12, 12, 18, 25]  // 25后移, 18后移, 12后移
                  ↓
           [3, 7, 10, 12, 18, 25]  // 放入 10
```

---

## 第十六章：合并两个有序数组

### 16.1 核心思想

给定两个**已排序**的数组 A 和 B，将它们合并为一个新的**有序**数组 C。

**方法**：使用三个指针，分别指向 A、B、C 的当前位置。每次比较 A 和 B 的当前元素，将较小的放入 C。

### 16.2 图解

```
A = [2, 6, 10, 15, 25]     i → 从头开始
B = [3, 4, 7, 18, 20]      j → 从头开始
C = [_, _, _, _, _, _, _, _, _, _]  k → 从头开始

比较 A[0]=2 和 B[0]=3 → 2 < 3, C[0]=2, i++
比较 A[1]=6 和 B[0]=3 → 6 > 3, C[1]=3, j++
比较 A[1]=6 和 B[1]=4 → 6 > 4, C[2]=4, j++
比较 A[1]=6 和 B[2]=7 → 6 < 7, C[3]=6, i++
...
C = [2, 3, 4, 6, 7, 10, 15, 18, 20, 25]
```

### 16.3 代码实现

```cpp
Array Merge(const Array& arr1, const Array& arr2) {
    Array result;
    result.size = arr1.length + arr2.length;
    result.length = 0;
    result.A = new int[result.size];

    int i = 0, j = 0, k = 0;

    // 两个数组都有剩余元素时，比较并放入较小的
    while (i < arr1.length && j < arr2.length) {
        if (arr1.A[i] < arr2.A[j]) {
            result.A[k++] = arr1.A[i++];
        } else {
            result.A[k++] = arr2.A[j++];
        }
    }

    // 将剩余元素全部放入
    while (i < arr1.length) {
        result.A[k++] = arr1.A[i++];
    }
    while (j < arr2.length) {
        result.A[k++] = arr2.A[j++];
    }

    result.length = k;
    return result;
}
```

### 16.4 复杂度分析

- **时间复杂度**：O(m + n)，其中 m 和 n 分别是两个数组的长度
- **空间复杂度**：O(m + n)，需要额外的结果数组

> **核心洞察**：合并操作是**归并排序（Merge Sort）** 的核心步骤。归并排序先递归地将数组分成两半分别排序，然后用合并操作将两个有序的半部分合并成一个完整的有序数组。

---

## 第十七章：数组的集合操作

数组可以被看作集合，我们可以在其上实现集合论中的基本操作。以下操作假设**两个数组都已排序**。

### 17.1 并集（Union）

**定义**：A ∪ B = 包含 A 和 B 中所有不重复元素的集合。

```
A = [2, 5, 6, 10, 15]
B = [3, 5, 7, 10, 18]
A ∪ B = [2, 3, 5, 6, 7, 10, 15, 18]
```

**实现思路**：类似合并操作，但遇到相同元素时只放入一次。

```cpp
Array Union(const Array& arr1, const Array& arr2) {
    Array result;
    result.size = arr1.length + arr2.length;
    result.A = new int[result.size];

    int i = 0, j = 0, k = 0;

    while (i < arr1.length && j < arr2.length) {
        if (arr1.A[i] < arr2.A[j]) {
            result.A[k++] = arr1.A[i++];
        } else if (arr1.A[i] > arr2.A[j]) {
            result.A[k++] = arr2.A[j++];
        } else {
            // 相等！只放入一次
            result.A[k++] = arr1.A[i++];
            j++; // 跳过 B 中的重复元素
        }
    }

    // 处理剩余元素
    while (i < arr1.length) result.A[k++] = arr1.A[i++];
    while (j < arr2.length) result.A[k++] = arr2.A[j++];

    result.length = k;
    return result;
}
```

### 17.2 交集（Intersection）

**定义**：A ∩ B = 同时存在于 A 和 B 中的元素。

```
A = [2, 5, 6, 10, 15]
B = [3, 5, 7, 10, 18]
A ∩ B = [5, 10]
```

```cpp
Array Intersection(const Array& arr1, const Array& arr2) {
    Array result;
    result.size = std::min(arr1.length, arr2.length);
    result.A = new int[result.size];

    int i = 0, j = 0, k = 0;

    while (i < arr1.length && j < arr2.length) {
        if (arr1.A[i] < arr2.A[j]) {
            i++;
        } else if (arr1.A[i] > arr2.A[j]) {
            j++;
        } else {
            // 相等！放入结果
            result.A[k++] = arr1.A[i++];
            j++;
        }
    }

    result.length = k;
    return result;
}
```

### 17.3 差集（Difference）

**定义**：A - B = 存在于 A 中但不存在于 B 中的元素。

```
A = [2, 5, 6, 10, 15]
B = [3, 5, 7, 10, 18]
A - B = [2, 6, 15]
```

```cpp
Array Difference(const Array& arr1, const Array& arr2) {
    Array result;
    result.size = arr1.length;
    result.A = new int[result.size];

    int i = 0, j = 0, k = 0;

    while (i < arr1.length && j < arr2.length) {
        if (arr1.A[i] < arr2.A[j]) {
            // A 的元素不在 B 中 → 放入结果
            result.A[k++] = arr1.A[i++];
        } else if (arr1.A[i] > arr2.A[j]) {
            j++; // B 的元素不相关，跳过
        } else {
            // 相等！不放入结果（因为 B 中也有）
            i++;
            j++;
        }
    }

    // A 的剩余元素都不在 B 中
    while (i < arr1.length) {
        result.A[k++] = arr1.A[i++];
    }

    result.length = k;
    return result;
}
```

### 17.4 三种操作的统一模式

| 情况         | 并集          | 交集     | 差集(A-B)     |
| ------------ | ------------- | -------- | ------------- |
| A[i] < B[j]  | **放入** A[i] | 跳过 A   | **放入** A[i] |
| A[i] > B[j]  | **放入** B[j] | 跳过 B   | 跳过 B        |
| A[i] == B[j] | 放入**一次**  | **放入** | **都跳过**    |
| A 有剩余     | 全部放入      | 不管     | 全部放入      |
| B 有剩余     | 全部放入      | 不管     | 不管          |

三种操作的时间复杂度都是 **O(m+n)**。

---

# 阶段三：数组（Array）— 完整教程（下篇：可选强化挑战）

---

## 第十八章：查找单个缺失元素（P99）

### 18.1 问题描述

给定一个从 1 到 n 的**连续自然数**序列，其中恰好**缺少一个**数字，存储在一个长度为 n-1 的数组中。找出缺失的那个数字。

```
输入: [1, 2, 3, 4, 6, 7, 8, 9, 10]   （缺了 5）
输出: 5
```

### 18.2 方法一：求和公式法

1~n 的总和公式：$S = \frac{n(n+1)}{2}$

用公式总和减去数组实际总和，差值就是缺失的数。

```cpp
long long FindMissing(const int A[], int length, int n) {
    long long expectedSum = 1LL * n * (n + 1) / 2;
    long long actualSum = 0;
    for (int i = 0; i < length; i++)
        actualSum += A[i];
    return expectedSum - actualSum;
}
```

```
n = 10, expectedSum = 55
actualSum = 1+2+3+4+6+7+8+9+10 = 50
缺失 = 55 - 50 = 5 ✓
```

**时间 O(n)，空间 O(1)**。

> **注意**：当 n 很大时，`n*(n+1)/2` 可能溢出 `int`。实际工程中可用 `long long`，或采用下面的方法。

### 18.3 方法二：逐项差值法

因为数组是有序的（1, 2, 3, ..., 缺一个, ..., n），我们可以检查 `A[i]` 是否等于 `i+1`（如果从 1 开始）。

```cpp
int FindMissing_v2(const int A[], int length) {
    for (int i = 0; i < length; i++) {
        if (A[i] != i + 1)
            return i + 1; // 这个位置应该是 i+1，但不是
    }
    return length + 1; // 缺的是最后一个数
}
```

**时间 O(n)，空间 O(1)**。

---

## 第十九章：查找多个缺失元素（P100~P101）

### 19.1 问题描述

数组来自 1~n 的自然数序列，但可能缺少**多个**数字。找出所有缺失的数字。

```
输入: [1, 3, 4, 7, 9, 10]    n=10
缺失: 2, 5, 6, 8
```

### 19.2 方法一：逐项对比法（数组已排序）

依次检查：数组中第 i 个位置的值是否等于期望值。

```cpp
void FindMultipleMissing_Sorted(const int A[], int length, int n) {
    int expected = 1; // 期望的下一个数
    int i = 0;

    while (expected <= n) {
        if (i < length && A[i] == expected) {
            i++; // 匹配，继续
        } else {
            std::cout << expected << " "; // 缺失！
        }
        expected++;
    }
}
```

```
A = [1, 3, 4, 7, 9, 10], n=10

expected=1: A[0]=1 ✓, i=1
expected=2: A[1]=3 ≠ 2, 输出 2
expected=3: A[1]=3 ✓, i=2
expected=4: A[2]=4 ✓, i=3
expected=5: A[3]=7 ≠ 5, 输出 5
expected=6: A[3]=7 ≠ 6, 输出 6
expected=7: A[3]=7 ✓, i=4
expected=8: A[4]=9 ≠ 8, 输出 8
expected=9: A[4]=9 ✓, i=5
expected=10: A[5]=10 ✓, i=6
输出: 2 5 6 8 ✓
```

**时间 O(n)，空间 O(1)**。

### 19.3 方法二：哈希表法（数组无序时使用）

当数组**无序**时，无法用上面的方法。使用哈希表（或布尔数组）标记出现过的数字。

```cpp
#include <unordered_set>

void FindMultipleMissing_Unsorted(const int A[], int length, int n) {
    std::unordered_set<int> present(A, A + length);

    for (int i = 1; i <= n; i++) {
        if (present.find(i) == present.end())
            std::cout << i << " ";
    }
}
```

或者用更高效的布尔数组：

```cpp
void FindMultipleMissing_BitArray(const int A[], int length, int n) {
    // 用 vector<bool> 作为位标记（也可用 new bool[n+1]）
    std::vector<bool> seen(n + 1, false);

    for (int i = 0; i < length; i++) {
        if (A[i] >= 1 && A[i] <= n) {
            seen[A[i]] = true;
        }
    }

    for (int i = 1; i <= n; i++)
        if (!seen[i])
            std::cout << i << " ";
}
```

**时间 O(n)，空间 O(n)**。

> **取舍**：有序数组用方法一（O(1) 空间），无序数组用方法二（O(n) 空间）。

---

## 第二十章：在有序数组中查找重复项（P102~P103）

### 20.1 基础版：找出所有重复元素

有序数组中，重复元素一定相邻。

```cpp
void FindDuplicates_Sorted(const int A[], int length) {
    for (int i = 0; i < length - 1; i++) {
        if (A[i] == A[i + 1]) {
            int val = A[i];
            int count = 1;
            // 统计连续相同元素的个数
            while (i < length - 1 && A[i] == A[i + 1]) {
                count++;
                i++;
            }
            std::cout << val << " 出现了 " << count << " 次\n";
        }
    }
}
```

```
A = [2, 3, 3, 5, 5, 5, 8, 10, 10]
输出:
3 出现了 2 次
5 出现了 3 次
10 出现了 2 次
```

**时间 O(n)，空间 O(1)**。

### 20.2 进阶版：利用索引差值

如果数组元素来自 1~n 的范围且已排序，可以利用一个性质：**如果没有重复，`A[i]` 应等于 `i + 起始值偏移`**。当 `A[i] == A[i-1]` 时，说明该值重复。

另一种更巧妙的方式——对于连续整数序列，如果 `A[i] - A[i-1] == 0`，则重复；如果  `A[i] - A[i-1] > 1`，则中间有缺失：

```cpp
void AnalyzeSortedArray(const int A[], int length) {
    if (length <= 1) return;

    for (int i = 1; i < length; i++) {
        int diff = A[i] - A[i - 1];
        if (diff == 0) {
            std::cout << A[i] << " 重复\n";
        } else if (diff > 1) {
            // 中间缺失了 diff-1 个数
            for (int j = A[i - 1] + 1; j < A[i]; j++)
                std::cout << j << " 缺失\n";
        }
    }
}
```

---

## 第二十一章：在无序数组中查找重复项（P104）

### 21.1 问题描述

数组无序，找出所有重复元素及其出现次数。

### 21.2 方法一：暴力双循环（不推荐）

```cpp
// 时间 O(n²)，空间 O(1)
void FindDup_Brute(const int A[], int length) {
    std::vector<bool> visited(length, false);

    for (int i = 0; i < length; i++) {
        if (visited[i]) continue;

        int count = 1;
        for (int j = i + 1; j < length; j++) {
            if (A[i] == A[j]) {
                count++;
                visited[j] = true;
            }
        }
        if (count > 1)
            std::cout << A[i] << " 出现了 " << count << " 次\n";
    }
}
```

### 21.3 方法二：哈希表（推荐）

```cpp
#include <unordered_map>

void FindDup_Hash(const int A[], int length) {
    std::unordered_map<int, int> freq;

    for (int i = 0; i < length; i++)
        freq[A[i]]++;

    for (const auto& [val, count] : freq) {
        if (count > 1)
            std::cout << val << " 出现了 " << count << " 次\n";
    }
}
```

**时间 O(n)，空间 O(n)**。

### 21.4 方法三：计数数组法（元素范围已知时最优）

如果元素值在 `0 ~ maxVal` 范围内，可以用计数数组：

```cpp
void FindDup_Counting(const int A[], int length, int maxVal) {
    int* count = new int[maxVal + 1]{}; // 零初始化

    for (int i = 0; i < length; i++) {
        if (A[i] >= 0 && A[i] <= maxVal) {
            count[A[i]]++;
        }
    }

    for (int i = 0; i <= maxVal; i++)
        if (count[i] > 1)
            std::cout << i << " 出现了 " << count[i] << " 次\n";

    delete[] count;
}
```

```
A = [8, 3, 6, 4, 6, 5, 6, 8, 2, 7]  maxVal=8
count: [0,0,1,1,1,1,3,1,2]
          索引: 0 1 2 3 4 5 6 7 8
输出: 6 出现了 3 次, 8 出现了 2 次
```

**时间 O(n+maxVal)，空间 O(maxVal)**。

---

## 第二十二章：查找满足条件的元素对（P105~P106）

### 22.1 问题描述

给定数组和目标值 `target`，找出所有满足 `A[i] + A[j] == target` 的元素对。

```
A = [1, 3, 4, 5, 6, 8, 9, 10, 12, 14]   target = 10
元素对: (1,9), (4,6)
```

### 22.2 方法一：暴力法（无序数组）

```cpp
void FindPairs_Brute(const int A[], int length, int target) {
    for (int i = 0; i < length - 1; i++)
        for (int j = i + 1; j < length; j++)
            if (A[i] + A[j] == target)
                std::cout << "(" << A[i] << ", " << A[j] << ")\n";
}
```

**时间 O(n²)**。

### 22.3 方法二：哈希集合法（无序数组，推荐）

```cpp
#include <unordered_set>

void FindPairs_Hash(const int A[], int length, int target) {
    std::unordered_set<int> seen;

    for (int i = 0; i < length; i++) {
        int complement = target - A[i];
        if (seen.count(complement))
            std::cout << "(" << complement << ", " << A[i] << ")\n";
        seen.insert(A[i]);
    }
}
```

**原理**：遍历数组，对每个元素 `x`，检查 `target - x` 是否已经在集合中。

```
target = 10, A = [1, 3, 4, 5, 6, 8, 9, 10, 12, 14]

i=0: x=1,  comp=9,  seen={} → 无          seen={1}
i=1: x=3,  comp=7,  seen={1} → 无         seen={1,3}
i=2: x=4,  comp=6,  seen={1,3} → 无       seen={1,3,4}
i=3: x=5,  comp=5,  seen={1,3,4} → 无     seen={1,3,4,5}
i=4: x=6,  comp=4,  seen 中有 4! → 输出 (4,6)
i=6: x=9,  comp=1,  seen 中有 1! → 输出 (1,9)
...
```

**时间 O(n)，空间 O(n)**。

### 22.4 方法三：双指针法（有序数组，最优）

```cpp
void FindPairs_TwoPointers(const int A[], int length, int target) {
    int left = 0, right = length - 1;

    while (left < right) {
        int sum = A[left] + A[right];
        if (sum == target) {
            std::cout << "(" << A[left] << ", " << A[right] << ")\n";
            left++;
            right--;
        } else if (sum < target) {
            left++;   // 和太小，左指针右移增大
        } else {
            right--;  // 和太大，右指针左移减小
        }
    }
}
```

```
A = [1, 3, 4, 5, 6, 8, 9, 10, 12, 14]  target=10
     L→                              ←R

1+14=15 > 10 → R--
1+12=13 > 10 → R--
1+10=11 > 10 → R--
1+9=10 == 10 → 输出 (1,9), L++, R--
3+8=11 > 10 → R--
3+6=9 < 10 → L++
4+6=10 == 10 → 输出 (4,6), L++, R--
5=5, L>=R → 结束
```

**时间 O(n)，空间 O(1)** ✓ 最优。但要求数组已排序。

---

## 第二十三章：单次遍历查找最大值和最小值（P107）

### 23.1 问题描述

在一次遍历中同时找出数组的最大值和最小值。

### 23.2 朴素方法（两次遍历）

```cpp
int max = Max(arr); // O(n)
int min = Min(arr); // O(n)
// 总共 2(n-1) 次比较
```

### 23.3 优化：单次遍历

```cpp
void FindMaxMin(const int A[], int length, int& outMax, int& outMin) {
    if (length == 0) {
        outMax = INT_MIN;
        outMin = INT_MAX;
        return;
    }

    outMax = outMin = A[0];

    for (int i = 1; i < length; i++) {
        if (A[i] > outMax)
            outMax = A[i];
        else if (A[i] < outMin) // 注意用 else if！
            outMin = A[i];
    }
}
```

**关键优化**：使用 `else if` 而非两个独立 `if`。如果元素比 max 还大，它不可能是 min，无需再比。

- 最好情况（升序）：n-1 次比较
- 最坏情况（降序）：2(n-1) 次比较
- 平均：约 1.5(n-1) 次比较

### 23.4 进阶：成对比较法

更高效的方法——每次取两个元素，先互相比较，再分别与 max/min 比较。每对只需 3 次比较（而非 4 次）。

```cpp
void FindMaxMin_Pairs(const int A[], int length, int& outMax, int& outMin) {
    if (length == 0) {
        outMax = INT_MIN;
        outMin = INT_MAX;
        return;
    }

    int i;
    // 初始化
    if (length % 2 == 0) {
        // 偶数个：先比较前两个
        if (A[0] > A[1]) { outMax = A[0]; outMin = A[1]; }
        else             { outMax = A[1]; outMin = A[0]; }
        i = 2;
    } else {
        // 奇数个：第一个既是 max 也是 min
        outMax = outMin = A[0];
        i = 1;
    }

    // 每次处理两个元素
    for (; i < length - 1; i += 2) {
        int bigger, smaller;
        if (A[i] > A[i + 1]) {       // 1 次比较
            bigger = A[i];
            smaller = A[i + 1];
        } else {
            bigger = A[i + 1];
            smaller = A[i];
        }
        if (bigger > outMax)  outMax = bigger;   // 1 次比较
        if (smaller < outMin) outMin = smaller;  // 1 次比较
    }
    // 每对 3 次比较，共 ⌈3n/2⌉ - 2 次比较，是理论最优
}
```

```
A = [3, 8, 1, 9, 2, 7]

初始化（偶数）：比较 3 vs 8 → max=8, min=3

第一对 (1, 9)：1 vs 9 → smaller=1, bigger=9
    9 > 8 → max=9;  1 < 3 → min=1

第二对 (2, 7)：2 vs 7 → smaller=2, bigger=7
    7 < 9 → 不变;   2 > 1 → 不变

结果: max=9, min=1 ✓
总比较次数: 1 + 3 + 3 = 7 次（而非朴素法的 10 次）
```

**比较次数**：$\lceil 3n/2 \rceil - 2$，这是**理论最优**，不可能用更少的比较次数同时找到最大和最小值。

---

## 第二十四章：教学严审补充（工程可用性）

### 24.1 代码片段最小头文件清单

为了让示例“复制即编译”，建议在整篇示例中按需补齐头文件：

```cpp
#include <iostream>      // std::cout, std::cin
#include <array>         // std::array
#include <vector>        // std::vector, std::vector<bool>
#include <unordered_set> // std::unordered_set
#include <unordered_map> // std::unordered_map
#include <algorithm>     // std::swap, std::min
#include <climits>       // INT_MIN, INT_MAX
```

### 24.2 返回新数组的所有权约定

`Merge`、`Union`、`Intersection`、`Difference` 都在堆上分配了 `result.A`。这意味着调用者必须负责释放，否则会泄漏。

```cpp
void Destroy(Array& arr) {
    delete[] arr.A;
    arr.A = nullptr;
    arr.size = 0;
    arr.length = 0;
}
```

调用示例：

```cpp
Array c = Merge(a, b);
Display(c);
Destroy(c); // 使用后释放
```

> 工程建议：如果进入 C++ 工程实践，优先考虑 `std::vector<int>` 替代手写 `new[]/delete[]`，减少资源管理错误。

---

## 知识总结

### 各操作复杂度速查表

| 操作                     | 时间复杂度         | 空间复杂度 |
| ------------------------ | ------------------ | ---------- |
| 随机访问 Get/Set         | O(1)               | O(1)       |
| 末尾插入 Append          | O(1)               | O(1)       |
| 任意位置插入 Insert      | O(n)               | O(1)       |
| 任意位置删除 Delete      | O(n)               | O(1)       |
| 线性搜索                 | O(n)               | O(1)       |
| 二分查找（已排序）       | O(log n)           | O(1)       |
| Max / Min / Sum          | O(n)               | O(1)       |
| 反转 Reverse（双指针）   | O(n)               | O(1)       |
| 检查有序                 | O(n)               | O(1)       |
| 合并两个有序数组         | O(m+n)             | O(m+n)     |
| 集合操作（有序）         | O(m+n)             | O(m+n)     |
| 动态扩容（一次）         | O(n)               | O(n)       |
| 查找缺失元素（求和法）   | O(n)               | O(1)       |
| 查找重复（哈希法）       | O(n)               | O(n)       |
| 元素对（双指针，有序）   | O(n)               | O(1)       |
| 同时求 Max+Min（成对法） | O(n)，~1.5n 次比较 | O(1)       |

### 数组的优缺点总结

| 优点               | 缺点                     |
| ------------------ | ------------------------ |
| O(1) 随机访问      | 插入/删除需移动元素 O(n) |
| 内存连续，缓存友好 | 大小固定，扩容代价高     |
| 实现简单           | 空间可能浪费（预分配）   |

> **核心认识**：数组擅长**读取**（O(1) 随机访问），不擅长**修改结构**（插入/删除 O(n)）。当需要频繁插入删除时，应考虑**链表**——这正是下一阶段要学习的内容。

---

**全部教程完成！** 三篇内容覆盖了数组从基础概念到高级挑战的全部知识点：

- **上篇（第1~10章）**：基础概念、声明、静态/动态、扩容、二维数组、地址公式、ADT、插入删除
- **中篇（第11~17章）**：线性搜索、二分查找、基础函数、反转移动、有序检测、合并、集合操作
- **下篇（第18~23章）**：缺失元素、重复元素、元素对查找、同时求最大最小值