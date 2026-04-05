# C++ 数据结构教程 —— 字符串（Strings）完全指南

---

## 第一章：字符串简介（P108）

### 1.1 什么是字符串？

在计算机科学中，**字符串（String）** 是由一系列字符组成的序列。它是我们日常编程中最常用的数据类型之一——用户名、密码、文章、网址……几乎所有与文字相关的数据都以字符串的形式存储。

在深入学习字符串操作之前，我们需要先理解几个基础概念。

### 1.2 ASCII 编码基础

计算机只能存储数字（0和1），那它是如何存储字母的呢？答案是**编码表**。

**ASCII（American Standard Code for Information Interchange）** 是最经典的字符编码标准。它用一个 **7位的整数（0~127）** 来表示每一个字符。

以下是你**必须**记住的关键ASCII码值：

| 字符范围  | ASCII 码值范围 | 说明                      |
| --------- | -------------- | ------------------------- |
| `A` ~ `Z` | 65 ~ 90        | 大写英文字母              |
| `a` ~ `z` | 97 ~ 122       | 小写英文字母              |
| `0` ~ `9` | 48 ~ 57        | 数字字符                  |
| 空格 ` `  | 32             | 空格符                    |
| `\0`      | 0              | 空字符（Null terminator） |

**一个至关重要的规律：** 大写字母和对应小写字母之间的差值固定为 **32**。

```
'A' = 65    'a' = 97    差值 = 32
'B' = 66    'b' = 98    差值 = 32
'Z' = 90    'z' = 122   差值 = 32
```

这个规律将在后续的大小写转换中发挥核心作用。

```cpp
#include <iostream>
using namespace std;

int main() {
    char ch = 'A';
    // char 本质上存储的是整数
    cout << "字符: " << ch << endl;          // 输出: A
    cout << "ASCII值: " << (int)ch << endl;  // 输出: 65
    
    // 字符可以参与算术运算
    cout << "'A' + 32 = " << (char)(ch + 32) << endl;  // 输出: a
    
    return 0;
}
```

### 1.3 C/C++ 中的两种字符串

在 C/C++ 中，表示字符串有**两种主要方式**：

#### ① C 风格字符串（字符数组）

C 风格字符串本质上是一个 **字符数组（char array）**，以特殊字符 **`\0`（空字符/Null Terminator）** 作为结束标志。

```cpp
// 方式一：逐个指定字符（必须手动加 '\0'）
char str1[] = {'H', 'e', 'l', 'l', 'o', '\0'};

// 方式二：使用字符串字面量（编译器自动加 '\0'）
char str2[] = "Hello";

// 方式三：指定数组大小
char str3[10] = "Hello";
// str3 的内存布局: ['H','e','l','l','o','\0','\0','\0','\0','\0']
```

**内存布局示意图：**

```
索引:   [0]  [1]  [2]  [3]  [4]  [5]
str2:   'H'  'e'  'l'  'l'  'o'  '\0'
ASCII:   72  101  108  108  111    0
```

> ⚠️ **关键概念：`\0` 的作用**
> `\0` 是字符串的"终止信号"。所有 C 风格的字符串函数（如 `strlen`、`printf` 的 `%s`）都依赖 `\0` 来判断字符串在哪里结束。如果忘记加 `\0`，程序会继续读取内存中的垃圾数据，直到碰巧遇到一个 `0` 值为止——这会导致**未定义行为**。

**字符数组 vs 字符指针：**

```cpp
// 字符数组：在栈上分配，内容可修改
char arr[] = "Hello";
arr[0] = 'h';  // ✅ 合法

// 字符指针：指向只读的字符串字面量区域
const char* ptr = "Hello";
// ptr[0] = 'h';  // ❌ 未定义行为！字符串字面量不可修改
```

#### ② C++ string 类

C++ 标准库提供了 `std::string` 类，它对 C 风格字符串进行了面向对象的封装，使用起来更安全、更方便。

```cpp
#include <string>
using namespace std;

string s1 = "Hello";          // 直接初始化
string s2("World");           // 构造函数初始化
string s3 = s1 + " " + s2;   // 拼接运算
cout << s3 << endl;           // 输出: Hello World
cout << s3.length() << endl;  // 输出: 11
```

#### 两种方式的对比

| 特性             | C 风格 `char[]`     | C++ `string`       |
| ---------------- | ------------------- | ------------------ |
| 需要手动管理内存 | ✅ 是                | ❌ 否（自动管理）   |
| 可以动态增长     | ❌ 否                | ✅ 是               |
| 以 `\0` 结尾     | ✅ 必须              | 内部自动处理       |
| 支持 `+` 拼接    | ❌ 否（需 `strcat`） | ✅ 是               |
| 性能             | 略高                | 略低（有额外开销） |
| 学习数据结构推荐 | ✅ **推荐使用**      | 辅助使用           |

> 📌 **本教程的选择：** 在数据结构的学习中，我们**主要使用 C 风格字符串（字符数组）**，因为它能让你直接接触底层的内存操作和算法细节。理解了底层原理后，使用 `std::string` 将会得心应手。

### 1.4 字符串的创建与基础操作

```cpp
#include <iostream>
using namespace std;

int main() {
    // === 创建字符串 ===
    char greeting[] = "Hello, World!";
    
    // === 逐字符遍历（使用 '\0' 作为终止条件）===
    cout << "逐字符输出: ";
    for (int i = 0; greeting[i] != '\0'; i++) {
        cout << greeting[i];
    }
    cout << endl;
    
    // === 输入字符串 ===
    char name[50];
    
    // cin >> name;        // 读取到空格就停止
    // cin.getline(name, 50);  // 读取整行（包括空格）
    
    cout << "请输入你的名字: ";
    cin.getline(name, 50);
    cout << "你好, " << name << "!" << endl;
    
    return 0;
}
```

---

## 第二章：计算字符串长度（P109）

### 2.1 什么是字符串长度？

字符串长度是指从首字符到 `\0` 之前的**有效字符个数**。注意：`\0` 本身**不计入**长度。

```
字符串:  "Hello"
字符:    'H'  'e'  'l'  'l'  'o'  '\0'
索引:     0    1    2    3    4    5
长度:     5（不包含 '\0'）
```

### 2.2 手动实现字符串长度计算

**原理非常简单：** 从索引 0 开始，逐个字符向后扫描，直到遇到 `\0` 为止，记录扫描过的字符数。

```cpp
#include <iostream>
using namespace std;

// 方法一：使用 while 循环
int myStrlen_v1(const char str[]) {
    int length = 0;
    while (str[length] != '\0') {
        length++;
    }
    return length;
}

// 方法二：使用 for 循环（更简洁）
int myStrlen_v2(const char str[]) {
    int i;
    for (i = 0; str[i] != '\0'; i++);  // 注意：循环体为空
    return i;
}

// 方法三：使用指针（进阶写法）
int myStrlen_v3(const char* str) {
    const char* start = str;       // 记录起始位置
    while (*str != '\0') {
        str++;                     // 指针向后移动
    }
    return str - start;            // 尾指针 - 头指针 = 长度
}

int main() {
    char text[] = "Data Structures";
    
    cout << "方法一: " << myStrlen_v1(text) << endl;  // 15
    cout << "方法二: " << myStrlen_v2(text) << endl;  // 15
    cout << "方法三: " << myStrlen_v3(text) << endl;  // 15
    
    // 对比标准库函数
    cout << "strlen: " << strlen(text) << endl;       // 15
    
    return 0;
}
```

### 2.3 时间复杂度分析

| 操作           | 时间复杂度 | 说明                      |
| -------------- | ---------- | ------------------------- |
| 计算字符串长度 | **O(n)**   | 必须遍历每个字符直到 `\0` |

**n** 是字符串的长度。这意味着如果字符串有 100 万个字符，就需要检查 100 万次。

> 💡 **思考：** C++ 的 `std::string` 类的 `.length()` 方法的时间复杂度是 **O(1)**，因为它内部维护了一个长度变量。而 C 风格的 `strlen()` 每次调用都是 **O(n)**。所以在循环中反复调用 `strlen()` 是一个常见的性能陷阱：
> ```cpp
> // ❌ 低效写法：每次循环都调用 strlen，总时间 O(n²)
> for (int i = 0; i < strlen(str); i++) { ... }
> 
> // ✅ 高效写法：只计算一次长度
> int len = strlen(str);
> for (int i = 0; i < len; i++) { ... }
> 
> // ✅ 最佳写法：直接判断 '\0'
> for (int i = 0; str[i] != '\0'; i++) { ... }
> ```

---

## 第三章：字符串大小写转换（P110）

### 3.1 转换原理

回忆 ASCII 编码的规律：

```
'A' (65)  ←→  'a' (97)    差值 = 32
'B' (66)  ←→  'b' (98)    差值 = 32
...
'Z' (90)  ←→  'z' (122)   差值 = 32
```

因此：
- **大写 → 小写：** 字符 + 32
- **小写 → 大写：** 字符 - 32

### 3.2 实现代码

```cpp
#include <iostream>
using namespace std;

// 将整个字符串转为大写
void toUpperCase(char str[]) {
    for (int i = 0; str[i] != '\0'; i++) {
        // 先判断是否是小写字母，再转换
        if (str[i] >= 'a' && str[i] <= 'z') {
            str[i] = str[i] - 32;
        }
    }
}

// 将整个字符串转为小写
void toLowerCase(char str[]) {
    for (int i = 0; str[i] != '\0'; i++) {
        // 先判断是否是大写字母，再转换
        if (str[i] >= 'A' && str[i] <= 'Z') {
            str[i] = str[i] + 32;
        }
    }
}

// 大小写互换（Toggle Case）
void toggleCase(char str[]) {
    for (int i = 0; str[i] != '\0'; i++) {
        if (str[i] >= 'A' && str[i] <= 'Z') {
            str[i] += 32;   // 大写 → 小写
        } else if (str[i] >= 'a' && str[i] <= 'z') {
            str[i] -= 32;   // 小写 → 大写
        }
        // 非字母字符（数字、空格、标点等）不变
    }
}

int main() {
    char s1[] = "Hello World 123";
    char s2[] = "Hello World 123";
    char s3[] = "Hello World 123";
    
    toUpperCase(s1);
    cout << "大写: " << s1 << endl;   // HELLO WORLD 123
    
    toLowerCase(s2);
    cout << "小写: " << s2 << endl;   // hello world 123
    
    toggleCase(s3);
    cout << "互换: " << s3 << endl;   // hELLO wORLD 123
    
    return 0;
}
```

### 3.3 判断字符类型的条件总结

```cpp
// 是否是大写字母
bool isUpper = (ch >= 'A' && ch <= 'Z');

// 是否是小写字母
bool isLower = (ch >= 'a' && ch <= 'z');

// 是否是字母（大写或小写）
bool isAlpha = (ch >= 'A' && ch <= 'Z') || (ch >= 'a' && ch <= 'z');

// 是否是数字字符
bool isDigit = (ch >= '0' && ch <= '9');

// 是否是字母或数字
bool isAlnum = isAlpha || isDigit;
```

> 📌 C 标准库 `<cctype>` 头文件提供了这些函数的现成版本：`isupper()`、`islower()`、`isalpha()`、`isdigit()`、`toupper()`、`tolower()` 等。了解原理后可以直接使用它们。

---

## 第四章：统计字符串中的单词和元音（P111）

### 4.1 统计元音字母

英语中的**元音字母（Vowels）** 是：**A, E, I, O, U**（及其小写形式）。

**辅音字母（Consonants）** 是除元音以外的所有字母。

```cpp
#include <iostream>
using namespace std;

void countVowelsAndConsonants(const char str[]) {
    int vowels = 0;
    int consonants = 0;
    
    for (int i = 0; str[i] != '\0'; i++) {
        char ch = str[i];
        
        // 检查是否是元音（需同时考虑大小写）
        if (ch == 'a' || ch == 'e' || ch == 'i' || ch == 'o' || ch == 'u' ||
            ch == 'A' || ch == 'E' || ch == 'I' || ch == 'O' || ch == 'U') {
            vowels++;
        }
        // 如果是字母但不是元音，那就是辅音
        else if ((ch >= 'A' && ch <= 'Z') || (ch >= 'a' && ch <= 'z')) {
            consonants++;
        }
        // 其他字符（空格、数字、标点等）不统计
    }
    
    cout << "元音字母数: " << vowels << endl;
    cout << "辅音字母数: " << consonants << endl;
}

int main() {
    char text[] = "How are you";
    countVowelsAndConsonants(text);
    // 元音: o, a, e, o, u → 5
    // 辅音: H, w, r, y → 4
    return 0;
}
```

### 4.2 统计单词数量

**关键观察：** 单词之间由空格分隔。统计单词数量的核心思想是：**数"从空格到非空格的切换次数"**。

有多种思路，这里介绍最经典的一种：

**方法：统计"空格 → 非空格"的转换次数**

```cpp
#include <iostream>
using namespace std;

int countWords(const char str[]) {
    int words = 0;
    
    for (int i = 0; str[i] != '\0'; i++) {
        // 当前字符是非空格，且前一个字符是空格（或这是第一个字符）
        // 说明一个新单词开始了
        if (str[i] != ' ' && (i == 0 || str[i - 1] == ' ')) {
            words++;
        }
    }
    
    return words;
}

int main() {
    char text1[] = "How are you";
    char text2[] = "  Hello   World  ";   // 多余空格的情况
    char text3[] = "";                     // 空字符串
    char text4[] = "SingleWord";           // 单个单词
    
    cout << "\"" << text1 << "\": " << countWords(text1) << " 个单词" << endl;  // 3
    cout << "\"" << text2 << "\": " << countWords(text2) << " 个单词" << endl;  // 2
    cout << "\"" << text3 << "\": " << countWords(text3) << " 个单词" << endl;  // 0
    cout << "\"" << text4 << "\": " << countWords(text4) << " 个单词" << endl;  // 1
    
    return 0;
}
```

**逐步执行追踪（以 `"How are you"` 为例）：**

```
i=0: str[0]='H' (非空格), i==0 → 新单词! words=1
i=1: str[1]='o' (非空格), str[0]='H' (非空格) → 不是新单词
i=2: str[2]='w' (非空格), str[1]='o' (非空格) → 不是新单词
i=3: str[3]=' ' (空格) → 跳过
i=4: str[4]='a' (非空格), str[3]=' ' (空格) → 新单词! words=2
i=5: str[5]='r' (非空格), str[4]='a' (非空格) → 不是新单词
i=6: str[6]='e' (非空格), str[5]='r' (非空格) → 不是新单词
i=7: str[7]=' ' (空格) → 跳过
i=8: str[8]='y' (非空格), str[7]=' ' (空格) → 新单词! words=3
i=9: str[9]='o' → 不是新单词
i=10: str[10]='u' → 不是新单词
i=11: str[11]='\0' → 循环结束

结果: 3 个单词 ✅
```

---

## 第五章：验证字符串（P112）

### 5.1 什么是字符串验证？

**字符串验证（String Validation）** 是指检查字符串中的所有字符是否都符合某个特定规则。

常见的验证场景：
- 用户名只能包含字母和数字
- 密码必须包含大写、小写、数字和特殊字符
- 输入的数据只能是纯数字

### 5.2 常见验证函数

```cpp
#include <iostream>
using namespace std;

// 验证：字符串是否全部为字母
bool isAllAlpha(const char str[]) {
    for (int i = 0; str[i] != '\0'; i++) {
        if (!((str[i] >= 'A' && str[i] <= 'Z') || 
              (str[i] >= 'a' && str[i] <= 'z'))) {
            return false;  // 发现非字母字符，立即返回 false
        }
    }
    return true;  // 全部是字母
}

// 验证：字符串是否全部为数字
bool isAllDigit(const char str[]) {
    for (int i = 0; str[i] != '\0'; i++) {
        if (!(str[i] >= '0' && str[i] <= '9')) {
            return false;
        }
    }
    return true;
}

// 验证：字符串是否只包含字母和数字（Alphanumeric）
bool isAlphanumeric(const char str[]) {
    for (int i = 0; str[i] != '\0'; i++) {
        char ch = str[i];
        if (!((ch >= 'A' && ch <= 'Z') || 
              (ch >= 'a' && ch <= 'z') || 
              (ch >= '0' && ch <= '9'))) {
            return false;
        }
    }
    return true;
}

// 验证密码强度：至少包含一个大写、一个小写、一个数字、一个特殊字符
bool isStrongPassword(const char str[]) {
    bool hasUpper = false;
    bool hasLower = false;
    bool hasDigit = false;
    bool hasSpecial = false;
    
    for (int i = 0; str[i] != '\0'; i++) {
        char ch = str[i];
        if (ch >= 'A' && ch <= 'Z')      hasUpper = true;
        else if (ch >= 'a' && ch <= 'z') hasLower = true;
        else if (ch >= '0' && ch <= '9') hasDigit = true;
        else                              hasSpecial = true;
    }
    
    return hasUpper && hasLower && hasDigit && hasSpecial;
}

int main() {
    cout << boolalpha;  // 让 bool 输出 true/false 而非 1/0
    
    cout << "isAllAlpha(\"Hello\"): "   << isAllAlpha("Hello") << endl;    // true
    cout << "isAllAlpha(\"Hello1\"): "  << isAllAlpha("Hello1") << endl;   // false
    cout << "isAllDigit(\"12345\"): "   << isAllDigit("12345") << endl;    // true
    cout << "isAllDigit(\"123a5\"): "   << isAllDigit("123a5") << endl;    // false
    cout << "isStrongPassword(\"Abc1!\"): " << isStrongPassword("Abc1!") << endl;  // true
    cout << "isStrongPassword(\"abc\"): "   << isStrongPassword("abc") << endl;    // false
    
    return 0;
}
```

### 5.3 验证的通用模式

所有字符串验证都遵循一个**通用的编程模式**：

```cpp
bool validate(const char str[]) {
    for (int i = 0; str[i] != '\0'; i++) {
        if (/* 当前字符不满足条件 */) {
            return false;   // "短路"返回：一旦发现不合格，立即判定失败
        }
    }
    return true;  // 所有字符都通过检查
}
```

这种"**遇到反例立即返回 false，全部通过才返回 true**"的模式非常高效，因为不合格的字符串通常很快就能被检测出来。

---

## 第六章：反转字符串（P113）

### 6.1 问题描述

将字符串 `"HELLO"` 反转为 `"OLLEH"`。

### 6.2 方法一：使用辅助数组

**思路：** 创建一个新数组，将原字符串从后向前复制到新数组中。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

void reverseString_v1(char str[]) {
    int len = strlen(str);
    char temp[100];   // 辅助数组（假设长度够用）
    
    // 从原字符串尾部开始，逐个复制到辅助数组
    for (int i = 0; i < len; i++) {
        temp[i] = str[len - 1 - i];
    }
    temp[len] = '\0';  // 别忘了终止符！
    
    // 将结果复制回原数组
    strcpy(str, temp);
}
```

**执行过程追踪：**

```
原字符串: "HELLO"  (len = 5)

i=0: temp[0] = str[4] = 'O'
i=1: temp[1] = str[3] = 'L'
i=2: temp[2] = str[2] = 'L'
i=3: temp[3] = str[1] = 'E'
i=4: temp[4] = str[0] = 'H'
temp[5] = '\0'

temp = "OLLEH" ✅
```

### 6.3 方法二：原地反转（双指针法）⭐

**思路：** 使用两个指针，一个从头开始（`left`），一个从尾开始（`right`），两个指针相向移动，每次交换它们指向的字符。

这是**最优解**，不需要额外空间。

```cpp
void reverseString_v2(char str[]) {
    int left = 0;
    int right = strlen(str) - 1;
    
    while (left < right) {
        // 交换 str[left] 和 str[right]
        char temp = str[left];
        str[left] = str[right];
        str[right] = temp;
        
        left++;
        right--;
    }
}
```

**执行过程追踪（以 `"HELLO"` 为例）：**

```
初始状态:
  [H] [E] [L] [L] [O]
   ↑               ↑
  left=0         right=4

第1次交换: 交换 H 和 O
  [O] [E] [L] [L] [H]
       ↑       ↑
     left=1  right=3

第2次交换: 交换 E 和 L
  [O] [L] [L] [E] [H]
           ↑↑
       left=2, right=2

left == right，循环结束（中间元素不需要动）

结果: "OLLEH" ✅
```

**偶数长度的情况（以 `"ABCD"` 为例）：**

```
初始: [A] [B] [C] [D]    left=0, right=3
交换: [D] [B] [C] [A]    left=1, right=2
交换: [D] [C] [B] [A]    left=2, right=1
                          left > right，循环结束

结果: "DCBA" ✅
```

### 6.4 复杂度对比

| 方法           | 时间复杂度 | 空间复杂度 |
| -------------- | ---------- | ---------- |
| 辅助数组       | O(n)       | O(n)       |
| 双指针原地反转 | O(n)       | **O(1)** ⭐ |

---

## 第七章：字符串比较与回文检查（P114）

### 7.1 字符串比较

**字符串比较** 是逐字符按 ASCII 值进行比较的过程，类似于字典中单词的排序规则（**字典序/Lexicographic Order**）。

**比较规则：**
1. 从第一个字符开始，逐一比较两个字符串对应位置的字符
2. 如果某个位置的字符不同，ASCII 值更大的字符串"更大"
3. 如果一个字符串是另一个的前缀，则较长的字符串"更大"
4. 如果所有字符都相同，两个字符串"相等"

```cpp
#include <iostream>
using namespace std;

// 手动实现字符串比较
// 返回值: 0 表示相等，正数表示 s1 > s2，负数表示 s1 < s2
int myStrcmp(const char s1[], const char s2[]) {
    int i = 0;
    
    // 逐字符比较，直到发现不同或某个字符串结束
    while (s1[i] != '\0' && s2[i] != '\0') {
        if (s1[i] != s2[i]) {
            return s1[i] - s2[i];  // 返回差值
        }
        i++;
    }
    
    // 走到这里说明其中一个或两个字符串已结束
    // 如果两个都结束了 → 相等（返回 0）
    // 如果只有 s2 结束 → s1 更长 → s1 > s2（返回正数）
    // 如果只有 s1 结束 → s2 更长 → s1 < s2（返回负数）
    return s1[i] - s2[i];
}

int main() {
    cout << myStrcmp("abc", "abc") << endl;   //  0 (相等)
    cout << myStrcmp("abd", "abc") << endl;   //  1 (d > c)
    cout << myStrcmp("abc", "abd") << endl;   // -1 (c < d)
    cout << myStrcmp("abc", "abcd") << endl;  // 负数 (abc 更短)
    
    return 0;
}
```

### 7.2 什么是回文？

**回文（Palindrome）** 是指正着读和反着读都一样的字符串。

例如：
- `"madam"` → 反转 → `"madam"` ✅ 回文
- `"racecar"` → 反转 → `"racecar"` ✅ 回文
- `"hello"` → 反转 → `"olleh"` ❌ 不是回文

### 7.3 回文检查方法

#### 方法一：反转后比较

```cpp
#include <iostream>
#include <cstring>
using namespace std;

bool isPalindrome_v1(const char str[]) {
    int len = strlen(str);
    char reversed[100];
    
    // 创建反转副本
    for (int i = 0; i < len; i++) {
        reversed[i] = str[len - 1 - i];
    }
    reversed[len] = '\0';
    
    // 比较原始和反转后的字符串
    return strcmp(str, reversed) == 0;
}
```

#### 方法二：双指针法（推荐）⭐

**不需要反转！** 直接用两个指针从两端向中间检查。

```cpp
bool isPalindrome_v2(const char str[]) {
    int left = 0;
    int right = strlen(str) - 1;
    
    while (left < right) {
        if (str[left] != str[right]) {
            return false;  // 发现不对称，不是回文
        }
        left++;
        right--;
    }
    
    return true;  // 所有对称位置都匹配
}
```

**执行追踪（`"madam"`）：**

```
  m   a   d   a   m
  ↑               ↑     left=0, right=4: 'm'=='m' ✅
      ↑       ↑         left=1, right=3: 'a'=='a' ✅
          ↑              left=2, right=2: left==right, 循环结束
→ 是回文 ✅
```

**执行追踪（`"hello"`）：**

```
  h   e   l   l   o
  ↑               ↑     left=0, right=4: 'h'!='o' ❌
→ 不是回文
```

### 7.4 复杂度对比

| 方法       | 时间复杂度 | 空间复杂度 |
| ---------- | ---------- | ---------- |
| 反转后比较 | O(n)       | O(n)       |
| 双指针法   | O(n)       | **O(1)** ⭐ |

---

## 第八章：查找字符串中的重复字符（P115）

### 8.1 问题描述

给定一个字符串，找出其中所有出现次数超过 1 次的字符，并输出它们各自出现的次数。

例如：`"finding"` 中，`'i'` 出现 2 次，`'n'` 出现 2 次。

### 8.2 方法一：暴力比较法

**思路：** 对每个字符，遍历整个字符串统计它出现的次数。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

void findDuplicates_brute(const char str[]) {
    int len = strlen(str);
    
    for (int i = 0; i < len; i++) {
        int count = 1;
        
        // 统计 str[i] 在后续位置出现的次数
        for (int j = i + 1; j < len; j++) {
            if (str[i] == str[j]) {
                count++;
            }
        }
        
        // 只在第一次发现时输出（避免重复输出）
        // 检查 str[i] 是否在 i 之前已经出现过
        bool alreadyPrinted = false;
        for (int k = 0; k < i; k++) {
            if (str[k] == str[i]) {
                alreadyPrinted = true;
                break;
            }
        }
        
        if (count > 1 && !alreadyPrinted) {
            cout << "'" << str[i] << "' 出现了 " << count << " 次" << endl;
        }
    }
}

int main() {
    findDuplicates_brute("finding");
    return 0;
}
```

**时间复杂度：O(n²)**——太慢了！

### 8.3 方法二：哈希表法（计数数组）⭐

**核心思想：** 利用 ASCII 码值作为数组索引，用一个大小为 128 的数组充当"哈希表"，直接记录每个字符出现的次数。

这是**字符串处理中最常用、最重要的技巧之一**。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

void findDuplicates_hash(const char str[]) {
    // ASCII 码范围 0~127，创建一个计数数组
    int count[128] = {0};  // 全部初始化为 0
    
    // 第一趟：遍历字符串，统计每个字符的出现次数
    for (int i = 0; str[i] != '\0'; i++) {
        count[(int)str[i]]++;  // 以字符的 ASCII 值作为索引
    }
    
    // 第二趟：遍历计数数组，输出出现次数 > 1 的字符
    for (int i = 0; i < 128; i++) {
        if (count[i] > 1) {
            cout << "'" << (char)i << "' 出现了 " << count[i] << " 次" << endl;
        }
    }
}

int main() {
    char text[] = "finding";
    cout << "字符串 \"" << text << "\" 中的重复字符:" << endl;
    findDuplicates_hash(text);
    return 0;
}
```

**执行过程追踪（`"finding"`）：**

```
遍历 "finding":
  'f'(102): count[102] = 1
  'i'(105): count[105] = 1
  'n'(110): count[110] = 1
  'd'(100): count[100] = 1
  'i'(105): count[105] = 2   ← 'i' 第二次出现
  'n'(110): count[110] = 2   ← 'n' 第二次出现
  'g'(103): count[103] = 1

输出: count > 1 的字符:
  'i' 出现了 2 次
  'n' 出现了 2 次
```

### 8.4 方法对比

| 方法                | 时间复杂度 | 空间复杂度 | 说明     |
| ------------------- | ---------- | ---------- | -------- |
| 暴力比较            | O(n²)      | O(1)       | 简单但慢 |
| **哈希表/计数数组** | **O(n)**   | O(1)*      | **推荐** |

> *空间固定为 128（ASCII字符集大小），与输入无关，因此空间复杂度也可以认为是 O(1)。

### 8.5 只针对小写字母的优化

如果已知字符串只包含小写英文字母（`a~z`），可以只用大小为 26 的数组：

```cpp
void findDuplicates_lowercase(const char str[]) {
    int count[26] = {0};
    
    for (int i = 0; str[i] != '\0'; i++) {
        count[str[i] - 'a']++;  // 'a'→0, 'b'→1, ..., 'z'→25
    }
    
    for (int i = 0; i < 26; i++) {
        if (count[i] > 1) {
            cout << "'" << (char)(i + 'a') << "' 出现了 " << count[i] << " 次" << endl;
        }
    }
}
```

`str[i] - 'a'` 这个映射非常常见：将字母 `'a'~'z'` 映射到索引 `0~25`。

---

## 第九章：使用位运算查找字符串中的重复字符（P116）

### 9.1 前置知识：位运算基础

在学习位运算方法之前，我们需要先理解几个关键的**位运算操作**：

| 运算       | 符号 | 含义           | 示例                  |
| ---------- | ---- | -------------- | --------------------- |
| 按位与 AND | `&`  | 两个都为1才是1 | `1010 & 1100 = 1000`  |
| 按位或 OR  | `\|` | 任一为1就是1   | `1010 \| 1100 = 1110` |
| 左移       | `<<` | 所有位向左移动 | `1 << 3 = 1000 = 8`   |
| 右移       | `>>` | 所有位向右移动 | `1000 >> 2 = 10 = 2`  |

**`1 << n` 的含义：** 生成一个"只有第 `n` 位为 1，其余全为 0"的数。

```
1 << 0 = 0000...0001   (第0位为1)
1 << 1 = 0000...0010   (第1位为1)
1 << 2 = 0000...0100   (第2位为1)
1 << 3 = 0000...1000   (第3位为1)
...
1 << 25 = 0000...10000000000000000000000000  (第25位为1)
```

### 9.2 核心思想：用一个整数的每一位代表一个字母

一个 `int` 有 32 位（bit），小写字母一共 26 个（`a~z`）。我们可以让每一位代表一个字母是否已经出现过：

```
位编号:  ... 4  3  2  1  0
对应字母:    e  d  c  b  a
```

- 如果第 `k` 位为 **0** → 字母 `'a' + k` **没出现过**
- 如果第 `k` 位为 **1** → 字母 `'a' + k` **出现过**

**这样，一个 32 位整数就相当于一个能追踪 26 个字母的"存在表"！**

### 9.3 算法步骤

```
初始化: H = 0  (所有位都是0，表示没有字母出现过)

对字符串中的每个字符 ch:
    1. 计算位置: x = ch - 'a'    (例如 'c'-'a'=2)
    2. 生成掩码: mask = 1 << x   (例如 1<<2 = 00...100)
    3. 检查: 如果 (H & mask) != 0 → 第 x 位已经是1 → ch 之前出现过 → 重复！
    4. 标记: H = H | mask          (将第 x 位设为1，记录 ch 已出现)
```

### 9.4 完整代码实现

```cpp
#include <iostream>
using namespace std;

void findDuplicates_bitwise(const char str[]) {
    int H = 0;   // 哈希位图（Bit Map），初始全0
    
    for (int i = 0; str[i] != '\0'; i++) {
        int x = str[i] - 'a';    // 字母对应的位编号 (0~25)
        int mask = 1 << x;        // 生成掩码
        
        if (H & mask) {
            // 第 x 位已经是 1，说明之前出现过 → 重复字符！
            cout << "重复字符: '" << str[i] << "'" << endl;
        } else {
            // 第 x 位是 0，说明第一次出现 → 标记为已出现
            H = H | mask;
        }
    }
}

int main() {
    findDuplicates_bitwise("finding");
    return 0;
}
```

### 9.5 执行过程详细追踪

以 `"finding"` 为例：

```
初始: H = 00000000000000000000000000  (26位简写)

处理 'f': x = 5,  mask = 000...0100000
  H & mask = 0 → 第一次出现
  H = 000...0100000

处理 'i': x = 8,  mask = 000...100000000
  H & mask = 0 → 第一次出现  
  H = 000...100100000

处理 'n': x = 13, mask = 00...10000000000000
  H & mask = 0 → 第一次出现
  H = 00...10000100100000

处理 'd': x = 3,  mask = 000...0001000
  H & mask = 0 → 第一次出现
  H = 00...10000100101000

处理 'i': x = 8,  mask = 000...100000000
  H & mask ≠ 0 → 第8位已经是1！ → 重复！输出 'i'

处理 'n': x = 13, mask = 00...10000000000000
  H & mask ≠ 0 → 第13位已经是1！ → 重复！输出 'n'

处理 'g': x = 6,  mask = 000...01000000
  H & mask = 0 → 第一次出现
  H = 00...10000101101000
```

输出结果：
```
重复字符: 'i'
重复字符: 'n'
```

### 9.6 避免重复输出的改进

上面的代码在 `"aaa"` 这样的输入中会多次输出 `'a'`。如果想**每个重复字符只输出一次**，可以用两个位图：

```cpp
void findDuplicates_bitwise_v2(const char str[]) {
    int seen = 0;       // 记录"已见过的字符"
    int reported = 0;   // 记录"已报告过的重复字符"
    
    for (int i = 0; str[i] != '\0'; i++) {
        int mask = 1 << (str[i] - 'a');
        
        if (seen & mask) {
            // 之前见过
            if (!(reported & mask)) {
                // 但还没报告过
                cout << "重复字符: '" << str[i] << "'" << endl;
                reported |= mask;  // 标记为已报告
            }
        } else {
            seen |= mask;  // 标记为已见
        }
    }
}
```

### 9.7 位运算方法 vs 计数数组方法

| 比较维度   | 计数数组                 | 位运算                         |
| ---------- | ------------------------ | ------------------------------ |
| 空间       | 26 个 int = **104 字节** | 1 个 int = **4 字节**          |
| 能统计次数 | ✅ 是                     | ❌ 否（只能判断是否重复）       |
| 适用字符集 | 任意大小                 | 最多 32 种字符（int 位数限制） |
| 代码复杂度 | 简单直观                 | 需要理解位运算                 |
| 适用场景   | 通用                     | 仅限小写字母/有限字符集        |

> 📌 **考试/面试建议：** 面试官提问这道题时，如果你能先说出计数数组法，再优化到位运算法，会是一个很好的加分项。

---

## 第十章：检查两个字符串是否为变位词（P117）

### 10.1 什么是变位词（Anagram）？

**变位词** 是指两个字符串由**完全相同的字符组成，只是排列顺序不同**。

例如：
- `"listen"` 和 `"silent"` ✅ 是变位词
- `"decimal"` 和 `"medical"` ✅ 是变位词
- `"hello"` 和 `"world"` ❌ 不是变位词
- `"aab"` 和 `"abb"` ❌ 不是变位词（字符数量不同）

### 10.2 方法一：排序后比较

**思路：** 如果两个字符串是变位词，那么将它们各自排序后，结果应该完全相同。

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

bool isAnagram_sort(char s1[], char s2[]) {
    int len1 = strlen(s1);
    int len2 = strlen(s2);
    
    // 长度不同一定不是变位词
    if (len1 != len2) return false;
    
    // 排序两个字符串
    sort(s1, s1 + len1);
    sort(s2, s2 + len2);
    
    // 比较排序后的结果
    return strcmp(s1, s2) == 0;
}

int main() {
    char a[] = "listen";
    char b[] = "silent";
    
    if (isAnagram_sort(a, b)) {
        cout << "是变位词" << endl;
    } else {
        cout << "不是变位词" << endl;
    }
    // 输出: 是变位词
    return 0;
}
```

**时间复杂度：O(n log n)**（排序的开销）

### 10.3 方法二：计数数组法（推荐）⭐

**思路：** 统计两个字符串中每个字符出现的次数，如果完全一致，就是变位词。

可以更巧妙地只用一个计数数组：**遇到 s1 的字符就 +1，遇到 s2 的字符就 -1**。最终如果所有计数都为 0，说明两者完美抵消——它们是变位词。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

bool isAnagram_count(const char s1[], const char s2[]) {
    // 长度不同一定不是变位词
    if (strlen(s1) != strlen(s2)) return false;
    
    int count[26] = {0};  // 假设只有小写字母
    
    // 遍历两个字符串（它们等长）
    for (int i = 0; s1[i] != '\0'; i++) {
        count[s1[i] - 'a']++;   // s1 的字符 +1
        count[s2[i] - 'a']--;   // s2 的字符 -1
    }
    
    // 检查所有计数是否归零
    for (int i = 0; i < 26; i++) {
        if (count[i] != 0) {
            return false;  // 有字符数量不匹配
        }
    }
    
    return true;
}

int main() {
    cout << boolalpha;
    cout << isAnagram_count("listen", "silent") << endl;    // true
    cout << isAnagram_count("decimal", "medical") << endl;  // true
    cout << isAnagram_count("hello", "world") << endl;      // false
    cout << isAnagram_count("aab", "abb") << endl;          // false
    return 0;
}
```

**执行追踪（`"listen"` vs `"silent"`）：**

```
初始: count = [0, 0, 0, ..., 0]  (26个0)

i=0: s1='l'(+1) → count[11]=+1    s2='s'(-1) → count[18]=-1
i=1: s1='i'(+1) → count[8]=+1     s2='i'(-1) → count[8]=0
i=2: s1='s'(+1) → count[18]=0     s2='l'(-1) → count[11]=0
i=3: s1='t'(+1) → count[19]=+1    s2='e'(-1) → count[4]=-1
i=4: s1='e'(+1) → count[4]=0      s2='n'(-1) → count[13]=-1
i=5: s1='n'(+1) → count[13]=0     s2='t'(-1) → count[19]=0

最终 count 全为 0 → 是变位词 ✅
```

### 10.4 复杂度对比

| 方法         | 时间复杂度 | 空间复杂度 | 是否修改原字符串 |
| ------------ | ---------- | ---------- | ---------------- |
| 排序比较     | O(n log n) | O(1)       | ✅ 会修改         |
| **计数数组** | **O(n)**   | O(1)*      | ❌ 不修改         |

---

## 第十一章：字符串排列（P118）

### 11.1 问题描述

给定一个字符串，生成它的**所有排列（Permutations）**。

例如，字符串 `"ABC"` 的所有排列为：
```
ABC, ACB, BAC, BCA, CAB, CBA
```

对于长度为 n 的字符串，排列总数为 **n!（n 的阶乘）**：
- 3 个字符 → 3! = 6 种排列
- 4 个字符 → 4! = 24 种排列
- 5 个字符 → 5! = 120 种排列

### 11.2 递归思想详解

字符串排列是一个经典的**递归**问题。我们来逐步拆解这个思路：

**核心递归思想：**

要生成 `"ABC"` 的所有排列，我们可以：
1. 固定第 1 个字符为 `'A'`，然后生成 `"BC"` 的所有排列 → `A`+`BC`, `A`+`CB`
2. 固定第 1 个字符为 `'B'`，然后生成 `"AC"` 的所有排列 → `B`+`AC`, `B`+`CA`
3. 固定第 1 个字符为 `'C'`，然后生成 `"AB"` 的所有排列 → `C`+`AB`, `C`+`BA`

**"固定"的操作实际上就是"交换"**：把每个字符轮流交换到当前位置来固定。

### 11.3 递归树可视化

```
                        permute("ABC", 0)
                       /        |        \
              swap(0,0)     swap(0,1)    swap(0,2)
               固定'A'       固定'B'      固定'C'
                 /              |             \
         perm("ABC",1)   perm("BAC",1)   perm("CBA",1)
           /      \          /    \          /      \
      swap(1,1) swap(1,2) s(1,1) s(1,2)  s(1,1) s(1,2)
       固定'B'   固定'C'  固定'A'  固定'C'  固定'B'  固定'A'
         |        |        |       |        |       |
       "ABC"    "ACB"    "BAC"   "BCA"    "CBA"   "CAB"
```

### 11.4 完整代码实现

```cpp
#include <iostream>
#include <cstring>
using namespace std;

// 交换两个字符
void swapChar(char& a, char& b) {
    char temp = a;
    a = b;
    b = temp;
}

// 生成字符串的所有排列
// str: 字符串数组
// left: 当前递归处理的起始位置
// right: 字符串最后一个字符的位置
void permute(char str[], int left, int right) {
    // 基准情况：当 left == right 时，所有位置都已固定，输出当前排列
    if (left == right) {
        cout << str << endl;
        return;
    }
    
    // 递归步骤：让每个字符轮流出现在 left 位置
    for (int i = left; i <= right; i++) {
        // 第1步：将 str[i] 交换到 str[left] 位置（固定当前位置的字符）
        swapChar(str[left], str[i]);
        
        // 第2步：递归处理 left+1 到 right 的子问题
        permute(str, left + 1, right);
        
        // 第3步：回溯（Backtrack）—— 撤销交换，恢复原状
        // 这一步至关重要！确保下一次循环时字符串回到交换前的状态
        swapChar(str[left], str[i]);
    }
}

int main() {
    char str[] = "ABC";
    int n = strlen(str);
    
    cout << "\"" << str << "\" 的所有排列:" << endl;
    permute(str, 0, n - 1);
    
    return 0;
}
```

**输出：**
```
"ABC" 的所有排列:
ABC
ACB
BAC
BCA
CBA
CAB
```

### 11.5 递归执行过程逐步追踪

```
调用: permute("ABC", 0, 2)
│
├─ i=0: swap(0,0) → "ABC"
│  └─ permute("ABC", 1, 2)
│       ├─ i=1: swap(1,1) → "ABC" → 输出 "ABC" ✅
│       └─ i=2: swap(1,2) → "ACB" → 输出 "ACB" ✅ → 回溯 → "ABC"
│  回溯 → "ABC"
│
├─ i=1: swap(0,1) → "BAC"
│  └─ permute("BAC", 1, 2)
│       ├─ i=1: swap(1,1) → "BAC" → 输出 "BAC" ✅
│       └─ i=2: swap(1,2) → "BCA" → 输出 "BCA" ✅ → 回溯 → "BAC"
│  回溯: swap(0,1) → "ABC"
│
├─ i=2: swap(0,2) → "CBA"
│  └─ permute("CBA", 1, 2)
│       ├─ i=1: swap(1,1) → "CBA" → 输出 "CBA" ✅
│       └─ i=2: swap(1,2) → "CAB" → 输出 "CAB" ✅ → 回溯 → "CBA"
│  回溯: swap(0,2) → "ABC"
```

### 11.6 回溯（Backtracking）为什么至关重要？

在上面的代码中，每次递归调用**之后**都有一句 `swapChar(str[left], str[i])`——这就是**回溯**。

如果没有回溯会怎样？看这个例子：

```
第一轮 i=1: swap(0,1) → "BAC"  →  递归处理完毕
没有回溯: 字符串此时仍然是 "BAC"
第二轮 i=2: swap(0,2) → "CAB"  ← 这里基于"BAC"交换，结果不对！
                                   正确的应该是基于"ABC"交换得到"CBA"
```

**回溯确保每次循环迭代都从"相同的起始状态"出发。**

### 11.7 处理重复字符

如果字符串包含重复字符（如 `"AAB"`），上面的算法会产生重复排列。解决办法是在**交换之前检查：从 left 到 i-1 之间是否已经有相同的字符被放到 left 位置上**。

```cpp
bool shouldSwap(char str[], int left, int i) {
    for (int k = left; k < i; k++) {
        if (str[k] == str[i])
            return false;  // 字符相同，跳过以避免重复排列
    }
    return true;
}

void permuteUnique(char str[], int left, int right) {
    if (left == right) {
        cout << str << endl;
        return;
    }
    for (int i = left; i <= right; i++) {
        if (shouldSwap(str, left, i)) {
            swapChar(str[left], str[i]);
            permuteUnique(str, left + 1, right);
            swapChar(str[left], str[i]);  // 回溯
        }
    }
}
```

### 11.8 复杂度分析

| 指标       | 值            | 说明                         |
| ---------- | ------------- | ---------------------------- |
| 时间复杂度 | **O(n × n!)** | n! 种排列，每种输出需 O(n)   |
| 空间复杂度 | **O(n)**      | 递归调用栈深度为 n           |
| 排列总数   | **n!**        | 3→6, 4→24, 5→120, 10→3628800 |

> ⚠️ 字符串排列的复杂度是**阶乘级**的，增长极快。当 n ≥ 10 时输出量已经非常庞大，实际应用中需要谨慎。

---

## 总结与知识图谱

### 核心技巧回顾

| 编号 | 主题          | 核心技巧               | 最优时间 | 最优空间 |
| ---- | ------------- | ---------------------- | -------- | -------- |
| P109 | 计算长度      | 遍历至 `\0`            | O(n)     | O(1)     |
| P110 | 大小写转换    | ±32 利用 ASCII 差值    | O(n)     | O(1)     |
| P111 | 统计单词/元音 | 状态转换检测           | O(n)     | O(1)     |
| P112 | 验证字符串    | 逐字符判断 + 短路返回  | O(n)     | O(1)     |
| P113 | 反转字符串    | **双指针法**           | O(n)     | O(1)     |
| P114 | 回文检查      | **双指针法**           | O(n)     | O(1)     |
| P115 | 查找重复字符  | **计数数组（哈希表）** | O(n)     | O(1)     |
| P116 | 位运算查重复  | **位图（Bitmap）**     | O(n)     | O(1)     |
| P117 | 变位词检查    | **计数数组 +/- 抵消**  | O(n)     | O(1)     |
| P118 | 字符串排列    | **递归 + 回溯**        | O(n·n!)  | O(n)     |

### 反复出现的核心模式

1. **`\0` 终止遍历** —— 几乎所有 C 风格字符串操作的基础
2. **双指针法** —— 反转、回文检查的首选
3. **计数数组 `count[26]` / `count[128]`** —— 字符频率统计的万能工具
4. **位图 Bitmap** —— 用单个整数追踪集合，极致空间优化
5. **递归 + 回溯** —— 排列/组合问题的通用框架

这些模式不仅在字符串中重要，在后续学习**链表、树、图**等数据结构时同样会反复用到。掌握好这些基础，后面的学习会事半功倍。