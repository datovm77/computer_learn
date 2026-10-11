# C++ 输入与输出完全教程：从基础到进阶与性能优化

---

## 前言

在 C++ 编程中，**输入（Input）与输出（Output，统称 I/O）** 是程序与用户、文件及其他系统交互的核心纽带：
- **输入**让程序获取外界的原始数据，为计算提供原料；
- **输出**将程序的处理结果、计算状态和调试信息反馈给外界。

C++ 的 I/O 体系非常丰富，但也因此常常让初学者产生困惑：
- 输入时：为什么 `cin >>` 读不到带空格的字符串？为什么混用 `cin >>` 和 `getline` 会诡异地“跳过”一行？多组数据到底怎么读到文件结尾？
- 输出时：为什么 `endl` 和 `\n` 不仅仅是换行的区别？如何优雅地控制浮点数保留几位小数、编号前导补 0、表格对齐？为什么大输入大输出的题目会频繁超时（TLE）？

本教程旨在为你提供一份**最详尽、最系统、从零基础到竞赛/实战级别的 C++ I/O 完全指南**。内容涵盖基于流对象的现代 C++ 输入输出、C 风格的高效 I/O、文件读写与重定向、格式化细节、流状态与异常控制、现代 C++20/23 新特性，以及算法竞赛中至关重要的性能优化与快读快写技巧。

**学习建议：** 每个知识点都附有完整的可运行示例代码，建议逐一在本地编辑器中编译运行，亲手修改输入数据观察输出变化，这样理解最透彻。

---

## 一、基于流对象的常用输入（推荐优先掌握）

### 1.0 什么是"流"？

在正式学习输入函数之前，你需要理解一个核心概念——**流（stream）**。

你可以把"流"想象成一条**传送带**：
- 键盘上敲入的字符，会一个接一个地排列在这条传送带上。
- 程序从传送带的前端依次取走字符进行处理。
- 取走的字符就从传送带上消失了，还没取走的继续排队等待。

C++ 中，标准输入流就是 `cin`（可以理解为 character input 的缩写），它连接着你的键盘（或者被重定向的输入源）。

当你在键盘上输入 `123 abc` 然后按下回车，传送带上实际排列的是：

```
'1' '2' '3' ' ' 'a' 'b' 'c' '\n'
```

注意最后有一个 `'\n'`（换行符），这是你按回车键产生的。这个换行符的存在，是后面很多"坑"的根源，我们会反复提到它。

---

### 1.1 `cin >>` —— 格式化输入（最常用）

#### 作用

从标准输入流中读取**一个数据项**（一个整数、一个浮点数、一个单词等）。

#### 基本行为

`cin >>` 的工作过程可以拆解为两步：
1. **跳过前导空白**：自动忽略输入流开头的所有空格、Tab（`\t`）、换行（`\n`）。
2. **读取有效字符**：从第一个非空白字符开始，按目标变量的类型读取。读字符串时通常读到下一个空白字符为止；读数字时读到不能继续组成数字的字符为止。停止处的字符通常**不会被取走**，会留在流中等待下一次读取。

#### 示例 1：读取整数

```cpp
#include <iostream>

int main() {
    int a;
    std::cout << "请输入一个整数: ";
    std::cin >> a;
    std::cout << "你输入的是: " << a << std::endl;
    return 0;
}
```

运行后输入 `42` 并回车，输出：

```
你输入的是: 42
```

#### 示例 2：连续读取多个变量

```cpp
#include <iostream>
#include <string>

int main() {
    int age;
    std::string name;
    std::cout << "请输入年龄和姓名（用空格隔开）: ";
    std::cin >> age >> name;
    std::cout << "姓名: " << name << ", 年龄: " << age << std::endl;
    return 0;
}
```

输入 `25 Alice` 并回车，输出：

```
姓名: Alice, 年龄: 25
```

**执行过程详解：**

输入后，流中的内容是：`'2' '5' ' ' 'A' 'l' 'i' 'c' 'e' '\n'`

- `cin >> age`：跳过前导空白（没有），读取 `25`，遇到空格停止。空格留在流中。
- `cin >> name`：跳过前导空白（跳过那个空格），读取 `Alice`，遇到 `'\n'` 停止。`'\n'` 留在流中。

#### 示例 3：多个数据分多行输入也没问题

```cpp
int a, b, c;
std::cin >> a >> b >> c;
```

你可以输入：

```
1 2 3
```

也可以输入：

```
1
2
3
```

甚至：

```
1    2
    3
```

效果完全一样，因为 `cin >>` 会自动跳过所有类型的空白字符。

#### 重要局限：无法读取含空格的字符串

```cpp
std::string s;
std::cin >> s;
// 输入 "Hello World"
// s 的值只有 "Hello"，"World" 还留在流里
```

这是 `cin >>` 最大的局限。如果你需要读取一整行（包括其中的空格），就要用下面要讲的 `getline`。

#### 类型不匹配时会发生什么？

```cpp
int a;
std::cin >> a;
// 如果用户输入的是 "abc" 而不是数字
```

此时 `cin` 会进入**失败状态**（fail state）。之后所有的 `cin >>` 操作都会直接跳过，不再读取任何内容。你可以用 `cin.fail()` 检查是否失败，用 `cin.clear()` 恢复状态。这属于进阶内容，但现在知道有这回事即可。

---

### 1.2 `getline(cin, string_var)` —— 读取整行（最推荐的行输入方式）

#### 作用

从输入流中读取**一整行**文本，包括其中的空格，直到遇到换行符 `'\n'` 为止。换行符会从流中取走并**丢弃**（不会存入目标字符串）。

#### 头文件

需要 `#include <string>`（通常 `#include <iostream>` 时已经间接包含了，但显式写上更规范）。

#### 基本语法

```cpp
std::string line;
std::getline(std::cin, line);
```

#### 示例 1：读取包含空格的字符串

```cpp
#include <iostream>
#include <string>

int main() {
    std::string line;
    std::cout << "请输入一行文字: ";
    std::getline(std::cin, line);
    std::cout << "你输入的是: [" << line << "]" << std::endl;
    std::cout << "长度: " << line.size() << std::endl;
    return 0;
}
```

输入 `Hello World!` 并回车，输出：

```
你输入的是: [Hello World!]
长度: 12
```

用方括号括住输出是一个好习惯，可以帮你看清字符串的首尾有没有多余的空白。

#### 示例 2：连续读取多行

```cpp
#include <iostream>
#include <string>

int main() {
    std::string line1, line2;
    std::cout << "请输入第一行: ";
    std::getline(std::cin, line1);
    std::cout << "请输入第二行: ";
    std::getline(std::cin, line2);
    std::cout << "第一行: [" << line1 << "]" << std::endl;
    std::cout << "第二行: [" << line2 << "]" << std::endl;
    return 0;
}
```

这里连续两次 `getline` 不会有任何问题，因为每次 `getline` 都会把换行符取走。

#### 核心陷阱：`cin >>` 和 `getline` 混用

这是初学者最常踩的坑之一，必须深入理解。

```cpp
#include <iostream>
#include <string>

int main() {
    int age;
    std::string name;

    std::cout << "请输入年龄: ";
    std::cin >> age;

    std::cout << "请输入姓名: ";
    std::getline(std::cin, name);

    std::cout << "年龄: " << age << std::endl;
    std::cout << "姓名: [" << name << "]" << std::endl;
    return 0;
}
```

运行后输入 `25` 并回车。你会发现程序**直接跳过了姓名的输入**，输出：

```
年龄: 25
姓名: []
```

**为什么？** 让我们分析流中发生了什么：

1. 你输入 `25` 并按回车，流中是：`'2' '5' '\n'`
2. `cin >> age` 读走了 `25`，但 `'\n'` **留在了流中**。
3. `getline(cin, name)` 一执行，它发现流中第一个字符就是 `'\n'`，于是认为"这一行已经结束了"，读到一个空字符串，把 `'\n'` 丢弃。
4. 程序继续执行，根本没给你输入姓名的机会。

**解决方案一：使用 `cin.ignore()`**

```cpp
std::cin >> age;
std::cin.ignore();          // 忽略流中的一个字符（这里必须确定它就是那个 '\n'）
std::getline(std::cin, name);    // 现在可以正常读取了
```

**解决方案二：使用 `std::ws`**（后面会详细讲）

```cpp
std::cin >> age;
std::getline(std::cin >> std::ws, name);  // ws 会跳过所有前导空白（包括 '\n'）
```

**解决方案三：全部使用 `getline`，需要数字时再转换**

```cpp
std::string age_str;
std::getline(std::cin, age_str);
int age = std::stoi(age_str);   // stoi：string to int

std::string name;
std::getline(std::cin, name);
```

这种方式比较统一，不会有残留换行符的问题。需要注意的是，`stoi` 在转换失败或数字超出范围时会抛出异常，实际项目中要做错误处理。

#### 空行输入

如果用户直接按回车（不输入任何内容），`getline` 会读到一个空字符串：

```cpp
std::string line;
std::getline(std::cin, line);
// 直接回车 → line 是 ""，line.size() 是 0
```

这是正常行为，不是错误。

---

### 1.3 `getline(cin, string_var, delim)` —— 指定分隔符

#### 作用

与上面的 `getline` 类似，但不再以换行符作为结束标志，而是以你指定的字符 `delim` 作为结束标志。

#### 语法

```cpp
std::getline(std::cin, variable, ',');
```

#### 示例 1：以逗号为分隔符

```cpp
#include <iostream>
#include <string>

int main() {
    std::string item1, item2, item3;

    std::cout << "请输入三个水果（用逗号隔开）: ";
    std::getline(std::cin, item1, ',');
    std::getline(std::cin, item2, ',');
    std::getline(std::cin, item3);       // 最后一个用默认换行符结束

    std::cout << "第一个: [" << item1 << "]" << std::endl;
    std::cout << "第二个: [" << item2 << "]" << std::endl;
    std::cout << "第三个: [" << item3 << "]" << std::endl;
    return 0;
}
```

输入 `apple,banana,cherry` 并回车，输出：

```
第一个: [apple]
第二个: [banana]
第三个: [cherry]
```

#### 示例 2：解析简单的 CSV 格式

```cpp
#include <iostream>
#include <string>

int main() {
    std::string name, age_str, city;

    // 假设输入格式是: 姓名,年龄,城市
    std::cout << "请输入（格式：姓名,年龄,城市）: ";
    std::getline(std::cin, name, ',');
    std::getline(std::cin, age_str, ',');
    std::getline(std::cin, city);   // 最后一项以换行结束

    std::cout << "姓名: " << name << std::endl;
    std::cout << "年龄: " << age_str << std::endl;
    std::cout << "城市: " << city << std::endl;
    return 0;
}
```

输入 `Alice,25,Beijing`，输出：

```
姓名: Alice
年龄: 25
城市: Beijing
```

#### 注意事项

- 分隔符字符本身会被从流中取走并丢弃，不会出现在结果字符串中。
- 如果指定了逗号作为分隔符，那么函数只会在读到逗号或到达文件结尾时停止；换行符 `'\n'` 不会自动结束这次读取。也就是说，在跨多行读取的场景下，换行符可能被读入结果字符串中。

---

### 1.4 `istringstream` —— 从字符串中提取数据

#### 为什么需要它？

有时候你面对的情况是：**一行里有多少个数据是不确定的**。

比如，用户输入一行数字，可能是 `1 2 3`，也可能是 `10 20 30 40 50`，你不知道具体有几个。

这时候的思路是：
1. 先用 `getline` 把整行读进来。
2. 再用 `istringstream` 把这个字符串当作一个"虚拟的输入流"，像用 `cin >>` 一样从里面逐个提取数据。

#### 头文件

```cpp
#include <sstream>
```

#### 基本用法

```cpp
#include <iostream>
#include <string>
#include <sstream>

int main() {
    std::string line;
    std::cout << "请输入若干整数（空格隔开）: ";
    std::getline(std::cin, line);

    std::istringstream iss(line);   // 用读到的字符串创建一个字符串输入流
    int num;
    int sum = 0;
    int count = 0;

    while (iss >> num) {       // 像用 cin >> 一样逐个提取
        sum += num;
        count++;
        std::cout << "读到: " << num << std::endl;
    }

    std::cout << "共 " << count << " 个数，总和 = " << sum << std::endl;
    return 0;
}
```

输入 `10 20 30 40`，输出：

```
读到: 10
读到: 20
读到: 30
读到: 40
共 4 个数，总和 = 100
```

#### `while (iss >> num)` 是怎么结束的？

`iss >> num` 每次成功读取一个整数就返回 `true`（准确说是返回流对象本身，它可以被隐式转换为 `bool`）。当字符串中的数据全部读完，再次尝试读取时会失败，返回 `false`，循环结束。

#### 示例 2：一行中混合多种类型

```cpp
#include <iostream>
#include <string>
#include <sstream>

int main() {
    std::string line = "Alice 25 3.85";
    std::istringstream iss(line);

    std::string name;
    int age;
    double gpa;

    iss >> name >> age >> gpa;

    std::cout << "姓名: " << name << std::endl;
    std::cout << "年龄: " << age << std::endl;
    std::cout << "GPA: " << gpa << std::endl;
    return 0;
}
```

输出：

```
姓名: Alice
年龄: 25
GPA: 3.85
```

#### 示例 3：配合 getline 处理多行、每行不定个数的数据

这是竞赛和实际开发中非常常见的模式：

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <vector>

int main() {
    std::string line;

    std::cout << "请逐行输入数字，每行若干个，输入空行结束:" << std::endl;

    while (std::getline(std::cin, line)) {
        if (line.empty()) break;   // 空行退出

        std::istringstream iss(line);
        int num;
        std::vector<int> row;

        while (iss >> num) {
            row.push_back(num);
        }

        std::cout << "这一行有 " << row.size() << " 个数: ";
        for (int x : row) {
            std::cout << x << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

输入：

```
1 2 3
10 20
100
（空行，直接回车）
```

输出：

```
这一行有 3 个数: 1 2 3
这一行有 2 个数: 10 20
这一行有 1 个数: 100
```

#### 本质理解

`istringstream` 不是一个新的"输入函数"，它是一个**把字符串变成流**的工具。它实现了和 `cin` 一样的接口（`>>` 运算符、`getline` 等都能用），所以你学会了 `cin >>`，就等于学会了 `istringstream >>`。

---

### 1.5 `cin.peek()` 与 `cin.putback()` —— 输入流的"偷看"与"放回"

在手写简单的词法分析器、计算器或解析复合输入格式时，常常需要“先看看下一个字符是什么，再决定用什么方式读取”，这时候 `cin.peek()` 和 `cin.putback()` 极其有用。

#### 1. `cin.peek()` —— 偷看下一个字符
- **作用**：查看输入流传送带最前端的下一个字符，但**不把它从流中取走**。
- **返回值**：字符的 ASCII 整数值（`int` 类型），若到达末尾则返回 `EOF`。

```cpp
#include <iostream>
#include <cctype>
#include <string>

int main() {
    std::cout << "请输入一个数字或单词: ";

    // 先偷看第一个有效字符（跳过前导空格）
    while (std::isspace(std::cin.peek())) {
        std::cin.get();
    }

    char nextChar = std::cin.peek();
    if (std::isdigit(nextChar)) {
        int num;
        std::cin >> num;
        std::cout << "检测到数字输入: " << num << std::endl;
    } else {
        std::string word;
        std::cin >> word;
        std::cout << "检测到文本输入: [" << word << "]" << std::endl;
    }

    return 0;
}
```

输入 `12345`，输出：
```
检测到数字输入: 12345
```
输入 `Hello`，输出：
```
检测到文本输入: [Hello]
```

#### 2. `cin.putback(ch)` —— 把字符塞回流中
- **作用**：如果已经用 `cin.get(ch)` 取出了一个字符，发现取错了，可以把字符塞回到输入流的前端，下次读取会再次读到它。

```cpp
char ch;
std::cin.get(ch);
if (ch == '#') {
    // 发现是特殊标记，特殊处理
} else {
    std::cin.putback(ch); // 不是特殊标记，放回去让后续的正常读取流程读取
    std::string rest;
    std::cin >> rest;
}
```

---

### 1.6 流状态与错误恢复 —— `fail()`, `eof()`, `clear()`

在程序运行过程中，如果用户的输入与预期的类型不匹配（例如要求输入整数却敲了 `abc`），流就会产生“错误状态”，后续所有输入都会被直接拒绝。理解流的状态位是写出健壮代码的关键。

#### 流的四大状态位
每个流对象内部都有 4 个标志位：
1. **`good()`**：一切正常，没有发生任何错误。
2. **`eof()`**：已经读到了输入流的末尾（End Of File）。
3. **`fail()`**：发生了格式错误（如把字母读入 `int`）或提取失败，但流本身没有严重损坏，**可以恢复**。
4. **`bad()`**：发生了不可逆的严重底层错误（如硬件损坏、读取非法物理内存），通常无法恢复。

#### 示例：遭遇输入错误并手动修复

```cpp
#include <iostream>
#include <limits>

int main() {
    int x;
    std::cout << "请输入一个整数: ";
    std::cin >> x;

    if (std::cin.fail()) {
        std::cout << "【检测到错误】输入的内容不是有效整数！" << std::endl;
        
        // 步骤 1：清除错误状态标志位（让流重新回到正常工作模式）
        std::cin.clear();

        // 步骤 2：清空留在流中导致出错的“垃圾”字符（一直丢弃直到换行符）
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');

        std::cout << "流已恢复正常，请重新输入一个整数: ";
        std::cin >> x;
    }

    std::cout << "最终成功读取: " << x << std::endl;
    return 0;
}
```

> [!WARNING]
> 修复流错误时，**必须 `cin.clear()` 和 `cin.ignore(...)` 双管齐下**：
> - 只调用 `cin.clear()`：流的状态恢复了，但那串错误的字符（如 `abc`）还在流传送带上，下一次读还会再次失败！
> - 只调用 `cin.ignore()`：流还处于 `fail` 状态，根本不会执行 `ignore` 操作！

---

### 1.7 多组数据循环输入与 EOF 处理

在各大 OJ（如 LeetCode、Codeforces、洛谷、牛客）的算法题中，“多组输入，直到文件末尾结束”是最基本的题型。

#### 1. 为什么 `while (cin >> x)` 可以当作循环条件？

```cpp
#include <iostream>

int main() {
    int a, b;
    // 只要成功读入 a 和 b，就继续循环
    while (std::cin >> a >> b) {
        std::cout << "a + b = " << a + b << std::endl;
    }
    return 0;
}
```

**底层原理**：
1. `cin >> a >> b` 的返回值是 `cin` 这个流对象本身（所以才能链式调用）。
2. 在 `while (...)` 条件判断中，流对象会被隐式转换为 `bool` 类型。
3. 只要流处于正常状态（`!cin.fail()`），转换出的 `bool` 就是 `true`；一旦到达文件结尾（EOF）或者读取失败，就会转换为 `false`，循环自然终止。

#### 2. 逐行读取直到文件结尾

```cpp
#include <iostream>
#include <string>

int main() {
    std::string line;
    // 每次读取一行，遇到文件末尾退出
    while (std::getline(std::cin, line)) {
        std::cout << "读取到: " << line << std::endl;
    }
    return 0;
}
```

#### 3. 如何在终端手动输入 EOF 触发结束？
在本地命令行终端运行上述循环程序时，如果不按特殊按键，程序会一直等待输入。如何模拟文件结尾呢？
- **Windows** 终端：按下 **`Ctrl + Z`** 然后按回车。
- **Linux / macOS / WSL** 终端：按下 **`Ctrl + D`**。

---

## 二、C 语言风格的输入函数

以下函数在 C++ 中也可以使用，但通常只在与 C 代码混用、处理字符数组、或追求极致性能的特定场景下才会用到。**作为初学者，优先掌握第一节的内容即可**，这一节作为知识补充。

---

### 2.1 `cin.getline(char_array, size)` —— C 风格的行输入

#### 与 `getline(cin, string)` 的区别

| 特性             | `getline(cin, string)`    | `cin.getline(char[], size)` |
| ---------------- | ------------------------- | --------------------------- |
| 存储目标         | `std::string`（自动扩容） | 字符数组（固定大小）        |
| 是否需要指定长度 | 不需要                    | 必须指定                    |
| 输入过长时       | 自动扩容                  | 会截断，并可能进入失败状态  |
| 推荐程度         | **推荐**                  | 仅在需要字符数组时使用      |

#### 语法

```cpp
char buffer[100];
std::cin.getline(buffer, 100);
// 最多读取 99 个字符（第 100 个位置留给 '\0'，即字符串结束标记）
```

#### 示例

```cpp
#include <iostream>

int main() {
    char name[50];
    std::cout << "请输入你的全名: ";
    std::cin.getline(name, 50);
    std::cout << "你好, " << name << "!" << std::endl;
    return 0;
}
```

输入 `Zhang San`，输出：

```
你好, Zhang San!
```

#### 第三个参数：自定义分隔符

```cpp
char buf[100];
std::cin.getline(buf, 100, ',');   // 读到逗号或满99个字符为止
```

#### 输入过长时会发生什么？

如果输入超过了 `size - 1` 个字符，`cin.getline` 会：
1. 只读取前 `size - 1` 个字符。
2. 将 `cin` 置为失败状态（`failbit`）。
3. 后续的输入操作都会失败，直到你调用 `cin.clear()`。

这就是为什么我们推荐使用 `getline(cin, string)`——`string` 可以自动扩容，通常不需要你手动处理缓冲区大小。

---

### 2.2 `cin.get()` —— 读取单个字符

#### 作用

从输入流中读取**一个字符**，包括空格、Tab、换行等空白字符。这一点和 `cin >>` 不同——`cin >>` 在读取 `char` 时会跳过空白，而 `cin.get()` 不会。

#### 两种用法

**用法一：参数形式**

```cpp
char ch;
std::cin.get(ch);   // 读取一个字符存入 ch
```

**用法二：返回值形式**

```cpp
int c = std::cin.get();   // 返回字符值（int 类型），失败时返回 EOF
```

返回 `int` 而不是 `char` 是有原因的：`EOF`（End Of File）是一个特殊值（通常是 -1），它不在普通字符的取值范围内，所以需要用 `int` 来接收。

#### 示例 1：对比 `cin >>` 和 `cin.get()` 读取字符

```cpp
#include <iostream>

int main() {
    char ch;

    // 用 cin >> 读字符
    std::cout << "cin >> 测试，请输入（试试先输入空格再输入a）: ";
    std::cin >> ch;
    std::cout << "读到: [" << ch << "]" << std::endl;

    // 清理流
    std::cin.ignore(1000, '\n');

    // 用 cin.get() 读字符
    std::cout << "cin.get() 测试，请输入（试试先输入空格再输入a）: ";
    std::cin.get(ch);
    std::cout << "读到: [" << ch << "]" << std::endl;

    return 0;
}
```

第一次输入 `  a`（两个空格加a），`cin >>` 跳过空格，读到 `a`。
第二次输入 `  a`（两个空格加a），`cin.get()` 读到第一个空格 ` `。

#### 示例 2：逐字符读取直到换行

```cpp
#include <iostream>

int main() {
    std::cout << "请输入一行文字: ";
    char ch;
    while (std::cin.get(ch) && ch != '\n') {
        std::cout << "[" << ch << "] ";
    }
    std::cout << std::endl;
    return 0;
}
```

输入 `Hi!`，输出：

```
[H] [i] [!]
```

#### `cin.get()` 的另一个用途：暂停程序

在某些环境中（比如 Windows 命令行），程序运行完窗口会立即关闭。可以在程序末尾加上：

```cpp
std::cin.get();   // 等待用户按回车
```

但现代 IDE 通常不需要这样做。

---

### 2.3 `getchar()` —— C 标准库的单字符读取

#### 作用

从标准输入（`stdin`）读取一个字符，返回字符值（`int` 类型），读取失败或到达文件末尾时返回 `EOF`。

#### 头文件

```cpp
#include <cstdio>   // C++ 中推荐用这个
// 或者 #include <stdio.h>   // C 风格
```

#### 与 `cin.get()` 的关系

功能几乎完全一样。`getchar()` 是 C 语言的函数，`cin.get()` 是 C++ 的成员函数。在 C++ 中两者都可以用，但不建议在同一个程序中混用 C 风格和 C++ 风格的 I/O。

#### 示例：统计输入中的元音字母个数

```cpp
#include <cstdio>
#include <cctype>

int main() {
    int c;
    int count = 0;

    printf("请输入一行文字: ");
    while ((c = getchar()) != '\n' && c != EOF) {
        int lower = std::tolower(c);
        if (lower == 'a' || lower == 'e' || lower == 'i' ||
            lower == 'o' || lower == 'u') {
            count++;
        }
    }
    printf("元音字母个数: %d\n", count);
    return 0;
}
```

输入 `Hello World`，输出：

```
元音字母个数: 3
```

#### 为什么用 `int` 接收而不是 `char`？

```cpp
int c = getchar();      // 正确
// char c = getchar();  // 危险！可能无法正确判断 EOF
```

`EOF` 通常定义为 `-1`。如果你用 `char` 接收：
- 在 `char` 是有符号的系统上，值为 255 的字符（`0xFF`）会被误判为 EOF。
- 在 `char` 是无符号的系统上，EOF（`-1`）会被截断成 255，永远不等于 EOF，导致无限循环。

所以，**用 `int` 接收 `getchar()` 的返回值是必须的**。

---

### 2.4 `gets()` 和 `fgets()` —— 了解即可，不推荐使用

#### `gets()` —— 已被废弃

```cpp
char buf[100];
gets(buf);   // 危险！不要使用！
```

`gets()` 从标准输入读取一行到字符数组中，但它**不检查数组边界**。如果用户输入超过数组容量，就会造成缓冲区溢出（buffer overflow），这是严重的安全漏洞。

- C11 标准已经正式移除了 `gets()`。
- C++14 标准也已经移除。
- 现代编译器会给出警告甚至错误。

**结论：永远不要使用 `gets()`。**

#### `fgets()` —— 相对安全但不够方便

```cpp
#include <cstdio>
#include <cstring>

int main() {
    char buf[100];
    printf("请输入: ");
    if (fgets(buf, 100, stdin) != nullptr) {   // 最多读99个字符，第三个参数指定输入源
        printf("你输入的是: [%s]\n", buf);
    }
    return 0;
}
```

输入 `Hello`，输出：

```
你输入的是: [Hello
]
```

注意输出中 `Hello` 后面有个换行——这是因为 `fgets()` **会保留末尾的换行符**，而 `getline` 不会。这是一个经常被忽略的差异。

如果你需要去掉末尾的换行符：

```cpp
#include <cstring>

if (fgets(buf, 100, stdin) != nullptr) {
    // 手动去掉换行符
    std::size_t len = strlen(buf);
    if (len > 0 && buf[len - 1] == '\n') {
        buf[len - 1] = '\0';
    }
}
```

**结论：在 C++ 中，直接用 `getline(cin, string)` 即可，没有必要使用 `fgets()`。**

---

### 2.5 `scanf` —— C 语言格式化输入（算法竞赛高频工具）

在传统的 C 语言及算法竞赛中，`scanf` 凭借其**无需配置即可达到的高速读取能力**以及**强大的格式匹配语法**，被广泛使用。

#### 头文件

```cpp
#include <cstdio>   // C++ 推荐写法
// 或 #include <stdio.h>
```

#### 基本语法与取地址符 `&`

```cpp
int a;
double b;
char str[20];

scanf("%d", &a);       // 普通变量必须传地址 &
scanf("%lf", &b);      // double 必须用 %lf！
scanf("%s", str);      // 字符数组名本身就是首地址，不需要写 &
```

> [!IMPORTANT]
> - 读取普通变量（`int`, `double`, `char` 等）时，**千万不要漏写 `&`**，否则会导致程序崩溃（段错误 Segmentation Fault）。
> - **浮点数差异**：`scanf` 读取 `double` **必须使用 `%lf`**（如果写 `%f` 会导致数据错误解析）；而读取 `float` 则使用 `%f`。

#### 常用格式占位符一览

| 占位符   | 对应数据类型             | 说明                                             |
| :------- | :----------------------- | :----------------------------------------------- |
| `%d`     | `int`                    | 有符号十进制整数                                 |
| `%lld`   | `long long`              | 64位有符号整数                                   |
| `%u`     | `unsigned int`           | 无符号十进制整数                                 |
| `%f`     | `float`                  | 单精度浮点数                                     |
| `%lf`    | `double`                 | 双精度浮点数（**不可错写为 `%f`**）              |
| `%c`     | `char`                   | 读取**单个字符（不会跳过空格与换行！）**          |
| `%s`     | `char[]`                 | 读取连续非空白字符串，遇到空格或换行停止         |
| `%x`/`%o`| `int`                    | 十六进制 / 八进制整数输入                        |

#### 示例 1：指定特殊分隔符读取

`scanf` 最强大的地方在于可以通过格式串直接跳过固定格式中的标点符号。

```cpp
#include <cstdio>

int main() {
    int year, month, day;
    printf("请输入日期（例如 2026-09-27）: ");
    
    // 格式串中的 '-' 会强制匹配输入流中的 '-' 字符并自动跳过
    scanf("%d-%d-%d", &year, &month, &day);

    printf("解析成功: 年=%d, 月=%02d, 日=%02d\n", year, month, day);
    return 0;
}
```

输入 `2026-09-27`，输出：
```
解析成功: 年=2026, 月=09, 日=27
```

#### 核心陷阱：`%c` 会吞噬回车与空格！

与 `cin >> ch`（自动跳过前导空白）不同，`scanf("%c", &ch)` 会原汁原味地读入下一个字符，即使它是空格、Tab 或回车换行符！

```cpp
int n;
char ch;
scanf("%d", &n);
scanf("%c", &ch); // 惨剧：读到了输入完 n 后按下的 '\n'！
```

**终极解决方案：在 `%c` 前加一个空格！**

```cpp
scanf("%d", &n);
scanf(" %c", &ch); // 注意 %c 前面有一个空格！
```
格式串中的空格表示：**“在这里跳过 0 个或多个空白字符（包括空格、Tab、换行符），直到遇到下一个有效字符才开始读取”**。这是一个非常实用的黑科技技巧。

#### 高级特性：扫描集与忽略项

1. **`%*` 抑制符（读取但不保存）**：
   ```cpp
   int a, b;
   // 跳过中间的一个数字
   scanf("%d %*d %d", &a, &b); // 输入 "10 999 20" -> a=10, b=20 (999被丢弃)
   ```

2. **扫描集 `%[...]`（简易正则读取整行）**：
   ```cpp
   char line[100];
   // %[^\n] 表示读取所有不是 '\n' 的字符，相当于 C 语言版本的整行读取
   scanf("%[^\n]", line);
   ```

#### 示例 2：多组输入的 EOF 判定

在算法竞赛中，用 `scanf` 读到文件结尾是经典写法：

```cpp
#include <cstdio>

int main() {
    int a, b;
    // scanf 成功读取时返回读取成功的变量个数
    // 到达文件末尾返回 EOF（即 -1）
    while (scanf("%d%d", &a, &b) == 2) {
        printf("%d\n", a + b);
    }
    return 0;
}
```

> [!TIP]
> 算法竞赛中经常能看到 `while (~scanf("%d%d", &a, &b))` 这种写法。
> 原理是：`EOF` 的值是 `-1`，二进制补码全为 1；按位取反 `~(-1)` 的结果为 `0`（布尔假）。当没有读到文件尾时，返回值大于等于 0，按位取反非 0（布尔真），所以循环会一直进行。

---

## 三、文件输入与输出

程序不一定总是从键盘读取或向屏幕输出。很多时候，数据需要持久化保存在文件中，或者从已经准备好的文件中批量读取。

C++ 提供了专门的文件流库 `<fstream>`：
- `ifstream`（input file stream）：专门用于从文件读取数据。
- `ofstream`（output file stream）：专门用于向文件写入数据。
- `fstream`：同时支持文件的读写操作。

它们的用法与 `cin`、`cout` 极其相似，几乎具有完全一致的操作符与接口。

### 3.1 `ifstream` + `>>` / `getline` —— 基本文件读取

#### 头文件

```cpp
#include <fstream>
```

#### 示例 1：从文件读取数据

假设有一个文件 `data.txt`，内容如下：

```
42
Hello World
3.14
```

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::ifstream fin("data.txt");   // 打开文件

    // 检查文件是否成功打开
    if (!fin) {
        std::cout << "无法打开文件！" << std::endl;
        return 1;
    }

    int number;
    std::string line;
    double pi;

    fin >> number;           // 读取整数 42
    fin.ignore();            // 跳过 42 后面的换行符
    std::getline(fin, line);      // 读取整行 "Hello World"
    fin >> pi;               // 读取浮点数 3.14

    std::cout << "整数: " << number << std::endl;
    std::cout << "字符串: " << line << std::endl;
    std::cout << "浮点数: " << pi << std::endl;

    fin.close();             // 关闭文件（可选，析构时会自动关闭）
    return 0;
}
```

输出：

```
整数: 42
字符串: Hello World
浮点数: 3.14
```

#### 关键点

- `ifstream fin("data.txt")` 创建一个文件输入流并打开文件。
- `fin` 的使用方式和 `cin` 完全一样：`fin >> x`、`getline(fin, line)` 等。
- **一定要检查文件是否成功打开**。`if (!fin)` 或 `if (!fin.is_open())` 都可以。
- `fin.close()` 关闭文件。其实当 `fin` 变量离开作用域（比如函数结束）时，析构函数会自动关闭文件，但显式关闭是好习惯。

#### 示例 2：逐行读取整个文件

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::ifstream fin("poem.txt");
    if (!fin) {
        std::cout << "文件打开失败！" << std::endl;
        return 1;
    }

    std::string line;
    int lineNum = 0;
    while (std::getline(fin, line)) {
        lineNum++;
        std::cout << lineNum << ": " << line << std::endl;
    }

    fin.close();
    return 0;
}
```

如果 `poem.txt` 内容是：

```
白日依山尽
黄河入海流
欲穷千里目
更上一层楼
```

输出：

```
1: 白日依山尽
2: 黄河入海流
3: 欲穷千里目
4: 更上一层楼
```

`while (getline(fin, line))` 会在文件读完（到达 EOF）时自动结束循环。

#### 示例 3：从文件读取结构化数据

假设 `students.txt` 中每行是一个学生的信息，格式为"姓名 年龄 成绩"：

```
Alice 20 95.5
Bob 21 88.0
Charlie 19 92.3
```

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::ifstream fin("students.txt");
    if (!fin) return 1;

    std::string name;
    int age;
    double score;

    while (fin >> name >> age >> score) {
        std::cout << name << " 同学，" << age << " 岁，成绩 " << score << std::endl;
    }

    return 0;
}
```

输出：

```
Alice 同学，20 岁，成绩 95.5
Bob 同学，21 岁，成绩 88
Charlie 同学，19 岁，成绩 92.3
```

`while (fin >> name >> age >> score)` 会在任一次读取失败（文件结束或格式错误）时结束循环。

---

### 3.2 读取整个文件到一个字符串

有时你需要一次性把整个文件的内容读入一个 `string` 中，而不是逐行处理。

#### 方法一：使用 `istreambuf_iterator`

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <iterator>

int main() {
    std::ifstream fin("file.txt");
    if (!fin) return 1;

    // 用迭代器范围构造 string
    std::string content(
        (std::istreambuf_iterator<char>(fin)),    // 起始位置：文件开头
        std::istreambuf_iterator<char>()          // 结束位置：文件末尾（默认构造）
    );

    std::cout << "文件内容 (" << content.size() << " 字节):" << std::endl;
    std::cout << content << std::endl;

    return 0;
}
```

**注意第一个参数外面多了一层括号**——这是为了避免 C++ 的"最令人困惑的解析"（Most Vexing Parse）问题。没有那层括号，编译器可能会把这一行理解成函数声明而不是变量定义。

#### 方法二：使用 `read()`（适合二进制文件）

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    std::ifstream fin("file.txt", std::ios::binary);   // 以二进制模式打开
    if (!fin) return 1;

    // 获取文件大小
    fin.seekg(0, std::ios::end);          // 移动到文件末尾
    std::streampos end = fin.tellg();     // 获取当前位置（即文件大小）
    if (end == std::streampos(-1)) return 1;
    std::size_t size = static_cast<std::size_t>(end);
    fin.seekg(0, std::ios::beg);          // 移回文件开头

    // 读取全部内容
    std::string content(size, '\0');      // 创建足够大的字符串
    if (!content.empty()) {
        fin.read(&content[0], static_cast<std::streamsize>(content.size()));  // 读取全部字节到字符串中
    }

    std::cout << "文件大小: " << size << " 字节" << std::endl;
    std::cout << content << std::endl;

    return 0;
}
```

这种方式在处理二进制文件（图片、音频等）或需要精确控制读取字节数时特别有用。

#### 两种方法的比较

| 特性                 | `istreambuf_iterator` | `read()`               |
| -------------------- | --------------------- | ---------------------- |
| 代码简洁度           | 较简洁                | 稍复杂                 |
| 适用场景             | 文本文件              | 文本和二进制文件均可   |
| 性能                 | 好                    | 通常更快（一次性读取） |
| 是否需要知道文件大小 | 不需要                | 需要先获取             |

---

### 3.3 `ofstream` + `<<` —— 基本文件写入与打开模式

#### 基本用法

`ofstream`（输出文件流）的使用方式和 `cout` 几乎一模一样，只是把 `cout` 换成了你定义的文件流对象：

```cpp
#include <iostream>
#include <fstream>

int main() {
    // 默认模式：如果文件不存在则创建；如果已存在则清空原内容重新写入
    std::ofstream fout("output.txt");

    if (!fout.is_open()) {
        std::cout << "无法创建或打开文件！" << std::endl;
        return 1;
    }

    fout << "姓名: 张三" << std::endl;
    fout << "成绩: " << 98.5 << std::endl;
    fout << "录取状态: " << std::boolalpha << true << std::endl;

    fout.close(); // 关闭流
    std::cout << "数据已成功写入 output.txt" << std::endl;
    return 0;
}
```

#### 文件打开模式（Open Mode）

在构造函数或 `open()` 时，可以传入第二个参数来指定打开模式：

```cpp
std::ofstream fout("log.txt", std::ios::app); // 以追加模式打开
```

常见模式标志位（定义在 `std::ios` 中）：

| 模式标志位    | 作用说明                                                 |
| :------------ | :------------------------------------------------------- |
| `ios::in`     | 以只读方式打开（`ifstream` 默认）                        |
| `ios::out`    | 以只写方式打开，通常伴随清空已有内容（`ofstream` 默认）  |
| `ios::app`    | **追加模式（Append）**：所有写入操作始终追加到文件末尾   |
| `ios::trunc`  | **截断模式（Truncate）**：如果文件已存在，清空其全部内容 |
| `ios::binary` | **二进制模式**：不进行回车换行符转换，原样字节读写       |

多个标志位可以用按位或运算符 `|` 组合：
```cpp
// 以二进制追加模式打开
std::ofstream fout("data.bin", std::ios::out | std::ios::app | std::ios::binary);
```

#### 示例：日志追加写入

```cpp
#include <iostream>
#include <fstream>
#include <string>

void logMessage(const std::string& msg) {
    // 使用 ios::app，每次写入都会追加在文件最后，不会覆盖旧日志
    std::ofstream logFile("app.log", std::ios::app);
    if (logFile) {
        logFile << "[LOG] " << msg << std::endl;
    }
}

int main() {
    logMessage("程序启动");
    logMessage("执行核心算法模块");
    logMessage("程序安全退出");
    std::cout << "日志记录完成，请查看 app.log" << std::endl;
    return 0;
}
```

---

### 3.4 `fstream` —— 读写一体与指针定位

如果你需要对同一个文件既读又写，可以使用 `fstream`。

#### 文件指针概念
文件流在内部维护着指针：
- **`seekg()`** / **`tellg()`**：用于输入流（**g** 表示 get，读取指针）。
- **`seekp()`** / **`tellp()`**：用于输出流（**p** 表示 put，写入指针）。

#### 指针移动基准
- `ios::beg`：相对文件开头（Begin）
- `ios::cur`：相对当前指针位置（Current）
- `ios::end`：相对文件末尾（End）

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() {
    // 读写模式打开
    std::fstream file("test.txt", std::ios::in | std::ios::out | std::ios::trunc);

    // 1. 写入两行数据
    file << "Line1: ABCDEFG" << std::endl;
    file << "Line2: 1234567" << std::endl;

    // 2. 将读取指针移回文件开头
    file.seekg(0, std::ios::beg);

    // 3. 读取第一行
    std::string s;
    std::getline(file, s);
    std::cout << "读取第一行: " << s << std::endl;

    // 4. 定位到文件开头后第 7 个字节处进行覆盖写入
    file.seekp(7, std::ios::beg);
    file << "XYZ";

    // 5. 重新读取查看修改结果
    file.seekg(0, std::ios::beg);
    while (std::getline(file, s)) {
        std::cout << s << std::endl;
    }

    file.close();
    return 0;
}
```

输出：
```
读取第一行: Line1: ABCDEFG
Line1: XYZDEFG
Line2: 1234567
```

---

### 3.5 算法竞赛重器：`freopen` 重定向标准输入输出

在 CSP-J/S、NOIP、ACM-ICPC 练习或日常算法刷题中，往往需要测试极其庞大的测试用例（比如几万行数据）。手敲键盘输入显然不可行，而改写成 `ifstream` 又得改动所有的 `cin` 代码。

**`freopen` 完美解决了这个问题！** 它可以将原本指向键盘/屏幕的标准流，重定向到文件。

#### 语法

```cpp
#include <cstdio> // 包含 freopen

freopen("in.txt", "r", stdin);   // 把键盘输入重定向到 in.txt 文件
freopen("out.txt", "w", stdout); // 把屏幕输出重定向到 out.txt 文件
```

#### 完整示例与本地调试技巧

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    // 本地调试神技巧：利用宏判断
    // 只有在本地运行（没有定义 ONLINE_JUDGE 宏）时才重定向到文件
    // 提交到 OJ（如洛谷、Codeforces）时，OJ 会自动定义 ONLINE_JUDGE，这段代码会自动被忽略！
#ifndef ONLINE_JUDGE
    freopen("in.txt", "r", stdin);
    freopen("out.txt", "w", stdout);
#endif

    int n;
    if (std::cin >> n) {
        std::vector<int> a(n);
        for (int i = 0; i < n; ++i) {
            std::cin >> a[i];
        }
        std::sort(a.begin(), a.end());
        for (int i = 0; i < n; ++i) {
            std::cout << a[i] << (i == n - 1 ? '\n' : ' ');
        }
    }

    return 0;
}
```

#### `freopen` 的绝妙优点
1. **零侵入性**：你的业务核心代码完全使用标准的 `cin >>` 和 `cout <<`，不需要改写成 `fin` 或 `fout`。
2. **免除重复粘贴**：把题目的大样例直接扔到 `in.txt`，运行后直接去 `out.txt` 看结果或使用 diff 工具对比答案。
3. **安全提交**：配合 `#ifndef ONLINE_JUDGE`，提交代码时哪怕忘了删掉文件重定向语句，也绝不会导致 OJ 报 `Runtime Error` 或 `Wrong Answer`。

---

## 四、特殊场景的输入处理

这一节介绍两个用于解决特定问题的工具，它们不是独立的输入方式，而是配合前面讲过的输入方法使用的辅助工具。

---

### 4.1 `std::ws` —— 跳过空白字符的操纵符

#### 问题场景

前面在 1.2 节讲过，`cin >>` 和 `getline` 混用时会出现残留换行符的问题。`std::ws` 是解决这个问题的另一种方式。

#### 什么是 `ws`？

`ws` 是一个**输入操纵符**（input manipulator），它的作用是：从输入流中提取并丢弃所有前导空白字符（空格、Tab、换行），直到遇到第一个非空白字符为止。

#### 基本用法

```cpp
#include <iostream>
#include <string>

int main() {
    int age;
    std::string name;

    std::cout << "请输入年龄: ";
    std::cin >> age;

    std::cout << "请输入姓名: ";
    std::getline(std::cin >> std::ws, name);    // ws 跳过了残留的 '\n' 以及可能的其他空白

    std::cout << "年龄: " << age << ", 姓名: [" << name << "]" << std::endl;
    return 0;
}
```

输入 `25`（回车）`Alice`（回车），输出：

```
年龄: 25, 姓名: [Alice]
```

**`cin >> ws` 的执行过程：**
1. `cin >> age` 之后，流中残留 `'\n'`。
2. `cin >> ws` 从流中取走这个 `'\n'`（以及任何后续的空白字符）。
3. `getline` 正常读取下一行。

#### `ws` vs `cin.ignore()` 的选择

| 方式                  | 效果             | 区别                             |
| --------------------- | ---------------- | -------------------------------- |
| `cin.ignore()`        | 忽略一个字符     | 只跳过一个字符，适合确定只剩一个换行符的情况 |
| `cin.ignore(n, '\n')` | 忽略直到换行符   | 可以跳过整行残留内容             |
| `cin >> ws`           | 跳过所有前导空白 | 如果用户输入了多个空行，全部跳过 |

一般来说：
- 如果你确定流里只剩一个换行符，用 `cin.ignore()` 即可。
- 如果你不确定有多少空白需要跳过，`cin >> ws` 更简便。
- 但要注意，`cin >> ws` 会把用户故意输入的前导空格也跳过，如果你需要保留前导空格，就不要用它。

---

### 4.2 `cin.ignore()` —— 忽略指定数量的字符

#### 语法

```cpp
std::cin.ignore();                    // 忽略1个字符
std::cin.ignore(n);                   // 忽略n个字符
std::cin.ignore(n, delim);            // 忽略最多n个字符，或者直到遇到 delim 为止（delim也被忽略）
```

#### 最常用的形式

```cpp
#include <limits>

std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
```

这行代码的意思是：忽略流中的字符，直到遇到换行符 `'\n'` 为止（换行符也被忽略）。`numeric_limits<streamsize>::max()` 是一个极大的数字，表示"不管有多少字符，全部忽略，直到遇到换行"。

#### 示例 1：解决 `cin >>` 后 `getline` 的问题

```cpp
#include <iostream>
#include <string>
#include <limits>

int main() {
    int num;
    std::string line;

    std::cout << "请输入一个数字: ";
    std::cin >> num;

    // 忽略剩余的所有字符直到换行（包括换行本身）
    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');

    std::cout << "请输入一行文字: ";
    std::getline(std::cin, line);

    std::cout << "数字: " << num << std::endl;
    std::cout << "文字: [" << line << "]" << std::endl;
    return 0;
}
```

#### 示例 2：跳过用户的错误输入

```cpp
#include <iostream>
#include <limits>

int main() {
    int num;

    while (true) {
        std::cout << "请输入一个整数: ";
        if (std::cin >> num) {
            break;   // 输入成功，跳出循环
        }
        // 输入失败（用户输入了非数字）
        std::cout << "输入无效，请重试！" << std::endl;
        std::cin.clear();   // 清除错误状态
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');   // 丢弃错误输入
    }

    std::cout << "你输入的是: " << num << std::endl;
    return 0;
}
```

如果用户输入 `abc`，程序会提示"输入无效"并重新要求输入，而不是陷入无限循环。

这里 `cin.clear()` 和 `cin.ignore()` 配合使用：
- `cin.clear()` 清除 `cin` 的失败状态，使其恢复正常工作。
- `cin.ignore(...)` 把流中残留的无效输入丢掉，否则下次 `cin >> num` 还是会读到 `abc` 然后再次失败。

---

## 五、C++ 输出方式与格式控制完全指南

程序不仅需要正确读取数据，还需要以规范、清晰、高效的格式将结果展现给用户或评测机。在算法竞赛与工程开发中，格式控制（如保留小数、对齐补零、进制转换）以及输出性能控制（如换行符的选择、缓冲区的刷新）至关重要。

---

### 5.0 输出的核心机制：输出流与缓冲区（Buffer）

前面我们将输入流比作从键盘通往程序的“传送带”。同理，**输出流是一条从程序通向屏幕或文件的输出传送带**。

在这个过程中，有一个至关重要的中间站——**缓冲区（Buffer）**：
- 当你执行 `cout << "Hello";` 时，字符并不是立即一个一个发送给显示器硬件，而是先暂存在内存中的**输出缓冲区**里。
- 只有当满足特定条件时，缓冲区中的数据才会被真正“倒”出来（写入硬件显示），这个动作称为**刷新缓冲区（Flush）**。

#### 什么时候会触发缓冲区刷新？
1. **缓冲区写满了**：自动批量刷新，效率最高。
2. **程序正常退出**：程序结束时会自动刷新所有未输出的数据。
3. **遇到强制刷新指令**：例如调用 `cout << flush;` 或者使用了 `cout << endl;`。
4. **输入流被读取前**：在默认情况下，`cin` 绑定了 `cout`，每次执行 `cin >>` 时都会先自动刷新 `cout`（保证提示语能第一时间显示在屏幕上）。

理解了缓冲区，我们就能解开 C++ 初学者最大的疑惑之一：`endl` 和 `'\n'` 到底有什么区别？

---

### 5.1 `cout <<` 与换行之争：`'\n'` vs `endl`

#### 基本语法与自动类型推导

`cout` 是标准输出流对象，利用左移运算符 `<<` 可以输出几乎所有内置数据类型，并且支持链式拼接：

```cpp
#include <iostream>
#include <string>

int main() {
    int age = 18;
    double score = 99.5;
    std::string name = "张三";

    // 链式输出不同类型的数据，无需手动指定占位符
    std::cout << "姓名: " << name << ", 年龄: " << age << ", 成绩: " << score << "\n";
    return 0;
}
```

#### 核心差异：`'\n'` 与 `endl`

很多教材和教学视频教大家输出换行时都写 `cout << endl;`，但在性能敏感场景（如算法竞赛大量输出时），这却是一个致命的“性能杀手”。

| 对比维度   | `cout << '\n';`                       | `cout << endl;`                           |
| :--------- | :------------------------------------ | :---------------------------------------- |
| 实际动作   | 只向缓冲区写入一个换行字符 `\n`       | 写入换行符 `\n`，**并立即强制刷新缓冲区** |
| 底层开销   | 极小（纯内存操作）                    | 很大（触发一次昂贵的系统调用系统 I/O）    |
| 执行速度   | **极快**                              | **极慢**（在大规模循环下易导致 TLE 超时） |
| 推荐使用场景 | **强烈推荐日常与算法竞赛默认使用**    | 仅在交互式程序或必须立即看到输出时使用    |

#### 速度实测对比
如果在循环中输出 10 万行数字：
- 使用 `cout << i << endl;`：通常耗时 **0.5 秒 ~ 1.5 秒** 以上。
- 使用 `cout << i << '\n';`：通常耗时 **0.02 秒 ~ 0.05 秒**（速度快了近 20~50 倍！）。

> [!IMPORTANT]
> **黄金法则**：在平时的代码编写、尤其是算法竞赛刷题中，**养成默认使用 `'\n'` 代替 `endl` 的好习惯**！

---

### 5.2 诊断与日志：`cerr` 与 `clog`

C++ 标准库在 `<iostream>` 中除了提供 `cout`（标准输出），还提供了另外两个常用的输出流对象：`cerr` 和 `clog`。

#### 三者对比一览

| 流对象 | 全称 | 对应标准设备 | 是否带缓冲区？ | 主要用途 |
| :--- | :--- | :--- | :---: | :--- |
| **`cout`** | Character Output | 标准输出（stdout） | **带缓冲** | 输出正常的程序结果 |
| **`cerr`** | Character Error | 标准错误（stderr） | **无缓冲（立即打印）** | 打印严重错误、异常、崩溃前调试信息 |
| **`clog`** | Character Log | 标准错误（stderr） | **带缓冲** | 打印日志、跟踪运行过程 |

#### 为什么需要 `cerr`？
因为 `cerr` 是**无缓冲**的！如果程序在某一行发生了严重的段错误（Crash）崩溃退出，`cout` 缓冲区里的内容可能还没来得及倒出来程序就已经死了，导致你看不到最后的日志；而 `cerr` 会立刻将信息打印到屏幕上，确保调试信息不丢失。

```cpp
#include <iostream>

int main() {
    std::cout << "这是一条普通输出" << std::endl;
    std::cerr << "[错误] 配置文件丢失！" << std::endl;
    std::clog << "[日志] 系统运行正常。" << std::endl;
    return 0;
}
```

#### 在命令行中将普通输出与错误分流
在 Linux / Mac / WSL 中，你可以把正常输出和错误信息分别保存到不同文件：
```bash
./my_program > output.txt 2> error.log
```
- `>`（即 `1>`）：重定向 `cout` 的标准输出到 `output.txt`。
- `2>`：重定向 `cerr` / `clog` 的标准错误到 `error.log`。

---

### 5.3 浮点数输出格式控制

在算法题和日常程序中，浮点数格式输出（如保留 2 位小数、科学计数法）非常高频。核心工具是 `<iomanip>` 中的格式控制操纵符。

#### 5.3.1 `setprecision(n)` —— 设置有效数字位数

##### 头文件

```cpp
#include <iomanip>
```

##### 默认含义：控制“有效数字”位数

单独使用 `setprecision(n)` 时，`n` 表示**有效数字位数**，不是小数点后位数。

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double x = 123.456789;

    std::cout << std::setprecision(3) << x << std::endl;
    std::cout << std::setprecision(6) << x << std::endl;

    return 0;
}
```

输出：

```
123
123.457
```

这里 `setprecision(3)` 输出的是 3 位有效数字，所以结果是 `123`，不是 `123.457`。

#### 什么是有效数字？

有效数字是从第一个非零数字开始计数的数字位数。

| 数值       | 3 位有效数字的输出 |
| ---------- | ------------------ |
| `123.456`  | `123`              |
| `12.3456`  | `12.3`             |
| `1.23456`  | `1.23`             |
| `0.012345` | `0.0123`           |

这就是为什么只写 `setprecision(n)` 时，有时看起来不像“保留 n 位小数”。

---

#### 5.3.2 `fixed + setprecision(n)` —— 保留小数点后 n 位

如果题目要求“保留 n 位小数”，最常用写法是：

```cpp
std::cout << std::fixed << std::setprecision(n) << x << std::endl;
```

其中：
- `fixed` 表示使用普通小数形式输出，不使用科学计数法。
- `setprecision(n)` 在 `fixed` 状态下表示**小数点后保留 n 位**。

##### 示例：保留 2 位小数

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double pi = 3.1415926;
    double money = 12;

    std::cout << std::fixed << std::setprecision(2) << pi << std::endl;
    std::cout << std::fixed << std::setprecision(2) << money << std::endl;

    return 0;
}
```

输出：

```
3.14
12.00
```

这是算法题中最常用的浮点输出方式。

##### 常见模板

```cpp
double ans;
std::cout << std::fixed << std::setprecision(6) << ans << std::endl;
```

如果题目要求误差不超过 `1e-6`，通常输出 6 到 10 位小数都可以。例如：

```cpp
std::cout << std::fixed << std::setprecision(10) << ans << std::endl;
```

---

#### 5.3.3 `scientific` —— 科学计数法输出

如果希望用科学计数法输出，可以使用 `scientific`：

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double x = 123456789.0;

    std::cout << std::scientific << std::setprecision(3) << x << std::endl;

    return 0;
}
```

输出：

```
1.235e+08
```

在 `scientific` 状态下，`setprecision(n)` 也表示**小数点后 n 位**。

---

#### 5.3.4 `fixed` 的作用范围与失效方式

`fixed`、`scientific`、`setprecision` 这类格式设置会一直影响后续输出，直到你主动修改它。它们不会因为一行输出结束、遇到 `endl`、进入下一条 `cout` 语句而自动失效。

```cpp
double x = 3.1415926;

std::cout << std::fixed << std::setprecision(2) << x << std::endl;      // 3.14
std::cout << x << std::endl;                                  // 3.14，仍然受 fixed 和 setprecision(2) 影响
std::cout << std::defaultfloat << std::setprecision(6) << x << std::endl; // 3.14159，恢复默认格式
```

也就是说，输出格式是“粘住的”（sticky）。一旦设置了 `fixed` 或 `setprecision`，后面的 `cout` 也会继续沿用这个设置。

##### 作用范围：只影响当前流对象

`fixed` 作用在某个具体的输出流对象上。你对 `cout` 设置了 `fixed`，只会影响 `cout`，不会自动影响 `cerr`、`clog` 或某个文件输出流 `ofstream`。

```cpp
#include <iostream>
#include <fstream>
#include <iomanip>

int main() {
    double x = 3.1415926;
    std::ofstream fout("out.txt");

    std::cout << std::fixed << std::setprecision(2);
    std::cout << x << std::endl;   // 3.14

    std::cerr << x << std::endl;   // 3.14159，cerr 没有被设置 fixed
    fout << x << std::endl;   // 3.14159，fout 也没有被设置 fixed

    return 0;
}
```

##### 如何让 `fixed` 失效？

常用方法一：使用 `defaultfloat` 恢复默认浮点格式。

```cpp
std::cout << std::fixed << std::setprecision(2) << 3.1415926 << std::endl;      // 3.14
std::cout << std::defaultfloat << std::setprecision(6) << 3.1415926 << std::endl; // 3.14159
```

常用方法二：使用 `unsetf` 取消浮点格式标志。

```cpp
std::cout << std::fixed << std::setprecision(2);
std::cout << 3.1415926 << std::endl;  // 3.14

std::cout.unsetf(std::ios::floatfield);  // 取消 fixed/scientific 浮点格式标志，回到默认浮点格式
std::cout << std::setprecision(6);
std::cout << 3.1415926 << std::endl;     // 3.14159
```

如果你在函数中临时修改输出格式，最好先保存旧格式，用完后恢复，避免影响调用者后续输出。

```cpp
void printMoney(double x) {
    std::ios::fmtflags oldFlags = std::cout.flags();
    std::streamsize oldPrecision = std::cout.precision();

    std::cout << std::fixed << std::setprecision(2) << x << std::endl;

    std::cout.flags(oldFlags);
    std::cout.precision(oldPrecision);
}
```

---

#### 5.3.5 `cout.precision(n)` —— 成员函数写法

除了 `setprecision(n)`，也可以使用 `cout.precision(n)`：

```cpp
double x = 3.1415926;

std::cout.precision(4);
std::cout << x << std::endl;  // 默认格式下：4 位有效数字

std::cout << std::fixed;
std::cout.precision(4);
std::cout << x << std::endl;  // fixed 格式下：小数点后 4 位
```

这两种写法效果类似：

```cpp
std::cout << std::setprecision(4);
std::cout.precision(4);
```

算法题中更常见的是 `setprecision`，因为它可以直接写在输出语句里。

---

#### 5.3.6 控制精度不是改变变量本身

输出精度只影响“显示出来的样子”，不会改变变量内部保存的值。

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double x = 3.1415926;

    std::cout << std::fixed << std::setprecision(2) << x << std::endl; // 显示 3.14
    std::cout << std::fixed << std::setprecision(5) << x << std::endl; // 仍然可以显示 3.14159

    return 0;
}
```

`setprecision(2)` 并没有把 `x` 变成 `3.14`，只是输出时按 2 位小数显示。

如果你真的需要把数值四舍五入到某个位数再参与后续计算，就不能只依赖输出格式，而要使用 `round` 等函数。但在大多数算法题中，只需要控制最终输出即可。

---

#### 5.3.7 四舍五入式保留 vs 直接截断

控制输出精度时，最容易混淆的是两件事：
- `fixed << setprecision(n)` 是**输出时按 n 位小数显示**，通常表现为四舍五入。
- 直接截断是**先把数值处理成截断后的结果**，再输出。

##### 方式一：输出时四舍五入显示

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double x = 3.149;

    std::cout << std::fixed << std::setprecision(2) << x << std::endl;

    return 0;
}
```

输出：

```
3.15
```

这里的 `x` 本身仍然是 `3.149`，只是显示时保留 2 位小数。注意，浮点数在计算机中是二进制近似值，如果数值刚好卡在 `x.xx5` 这种边界，输出结果可能和十进制直觉不完全一致。

##### 方式二：真正把数值四舍五入到指定小数位

如果你不仅要显示，还要得到一个四舍五入后的新数值，可以使用 `round`。

```cpp
#include <iostream>
#include <iomanip>
#include <cmath>

int main() {
    double x = 3.149;
    double y = std::round(x * 100) / 100.0;

    std::cout << std::fixed << std::setprecision(2) << y << std::endl;

    return 0;
}
```

输出：

```
3.15
```

其中 `100` 表示保留 2 位小数；如果保留 3 位小数，就使用 `1000`。

##### 方式三：直接截断，不四舍五入

如果题目明确要求“截断”“舍去后面的位数”，就不能用 `setprecision` 直接输出，因为它会四舍五入。可以用 `trunc`：

```cpp
#include <iostream>
#include <iomanip>
#include <cmath>

int main() {
    double x = 3.149;
    double y = std::trunc(x * 100) / 100.0;

    std::cout << std::fixed << std::setprecision(2) << y << std::endl;

    return 0;
}
```

输出：

```
3.14
```

`trunc` 的含义是向 0 方向截断。对正数来说就是直接去掉多余小数位；对负数来说要特别注意：

| 表达式                      | 结果    | 含义             |
| --------------------------- | ------- | ---------------- |
| `trunc(3.149 * 100) / 100`  | `3.14`  | 正数直接截断     |
| `trunc(-3.149 * 100) / 100` | `-3.14` | 负数向 0 截断    |
| `floor(-3.149 * 100) / 100` | `-3.15` | 负数向更小值取整 |

所以：
- 想“保留 n 位小数并四舍五入显示”，用 `fixed << setprecision(n)`。
- 想“得到四舍五入后的新数值”，用 `round(x * base) / base`。
- 想“直接截断，不四舍五入”，用 `trunc(x * base) / base`。

---

#### 5.3.8 与 C 风格 `printf` 的对比

C++ 的 `cout` 写法：

```cpp
std::cout << std::fixed << std::setprecision(2) << x << std::endl;
```

C 风格的 `printf` 写法：

```cpp
printf("%.2f\n", x);
```

两者都可以输出小数点后 2 位。C++ 中如果使用 `printf`，需要包含：

```cpp
#include <cstdio>
```

在 C++ 程序中，建议优先统一使用 `cout` / `cin`，不要频繁混用 `cout` 和 `printf`，除非你明确知道自己在做什么。

---

### 5.4 字段宽度、填充与对齐

在输出表格、对齐数据、输出固定格式（如日期时间 `09:05:07`）时，宽度与对齐控制必不可少。

#### 5.4.1 `setw(n)` —— 设置输出宽度

##### 头文件

```cpp
#include <iomanip>
```

##### 作用

`setw(n)` 用来设置**下一个输出项的最小宽度**。

如果实际输出内容长度小于 `n`，就用填充字符补齐；如果实际输出内容长度大于或等于 `n`，则正常完整输出，**不会截断内容**。

##### 示例 1：默认右对齐

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << "[" << std::setw(5) << 42 << "]" << std::endl;
    std::cout << "[" << std::setw(5) << "Hi" << "]" << std::endl;
    std::cout << "[" << std::setw(5) << "HelloWorld" << "]" << std::endl;

    return 0;
}
```

输出：

```
[   42]
[   Hi]
[HelloWorld]
```

可以看到：
- `42` 长度为 2，不足 5 位，左边补 3 个空格。
- `Hi` 长度为 2，不足 5 位，左边补 3 个空格。
- `HelloWorld` 长度超过 5，直接完整输出，不会被截断。

##### `setw` 只影响紧跟着的一个输出项

这是 `setw` 最重要的特点：它不是“粘住的”，只对**下一个**被输出的值生效。

```cpp
std::cout << std::setw(5) << 1 << 2 << 3 << std::endl;
```

输出：

```
    123
```

只有 `1` 被设置了宽度，`2` 和 `3` 不受影响。如果希望每一列都占固定宽度，必须每次都写 `setw`：

```cpp
std::cout << std::setw(5) << 1 << std::setw(5) << 2 << std::setw(5) << 3 << std::endl;
```

输出：

```
    1    2    3
```

##### 示例 2：按列输出表格

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::left << std::setw(10) << "Name"
         << std::right << std::setw(6) << "Score" << std::endl;

    std::cout << std::left << std::setw(10) << "Alice"
         << std::right << std::setw(6) << 95 << std::endl;

    std::cout << std::left << std::setw(10) << "Bob"
         << std::right << std::setw(6) << 8 << std::endl;

    return 0;
}
```

输出：

```
Name       Score
Alice          95
Bob             8
```

这类写法常用于调试时打印数组、矩阵、统计表。

---

#### 5.4.2 `setfill(ch)` —— 设置填充字符

##### 作用

`setfill(ch)` 用来设置当输出宽度不够时，用什么字符补齐。默认填充字符是空格 `' '`。

最常见的用法是配合 `setw` 进行**前导补 0**。

##### 示例 1：编号补 0

```cpp
#include <iostream>
#include <iomanip>

int main() {
    int id = 7;

    std::cout << std::setfill('0') << std::setw(4) << id << std::endl;

    return 0;
}
```

输出：

```
0007
```

这里 `setw(4)` 表示最小宽度为 4，`setfill('0')` 表示不足的位置用字符 `'0'` 补齐。

##### 示例 2：时间格式

```cpp
#include <iostream>
#include <iomanip>

int main() {
    int h = 9, m = 5, s = 7;

    std::cout << std::setfill('0')
         << std::setw(2) << h << ":"
         << std::setw(2) << m << ":"
         << std::setw(2) << s << std::endl;

    return 0;
}
```

输出：

```
09:05:07
```

注意：因为 `setw` 只影响下一个输出项，所以 `h`、`m`、`s` 前面都要分别写 `setw(2)`；但 `setfill('0')` 会持续生效，所以只需要写一次。

##### `setfill` 是“粘住的”

与 `setw` 不同，`setfill` 会一直影响同一个输出流，直到你重新设置填充字符。

```cpp
std::cout << std::setfill('0') << std::setw(4) << 7 << std::endl;  // 0007
std::cout << std::setw(4) << 8 << std::endl;                  // 0008，仍然用 0 填充

std::cout << std::setfill(' ') << std::setw(4) << 9 << std::endl;  //    9，恢复为空格填充
```

所以，如果你只是临时补 0，后面又要正常按空格对齐，记得恢复：

```cpp
std::cout << std::setfill(' ');
```

---

#### 5.4.3 对齐方式：`left`、`right`、`internal`

##### `right`：右对齐（默认）

默认情况下，输出是右对齐的，填充字符放在左边。

```cpp
std::cout << "[" << std::setw(5) << 42 << "]" << std::endl;
```

输出：

```
[   42]
```

##### `left`：左对齐

`left` 表示左对齐，填充字符放在右边。

```cpp
std::cout << std::left << "[" << std::setw(5) << 42 << "]" << std::endl;
```

输出：

```
[42   ]
```

`left` 常用于输出文本列，例如姓名、城市名等。

##### `internal`：符号在最左，填充在符号和数字之间

当输出带符号数字并且使用 `0` 填充时，`internal` 很有用。

```cpp
#include <iostream>
#include <iomanip>

int main() {
    std::cout << std::setfill('0');

    std::cout << std::right << std::setw(6) << -42 << std::endl;
    std::cout << std::internal << std::setw(6) << -42 << std::endl;

    return 0;
}
```

输出：

```
000-42
-00042
```

一般来说，如果你想让负数看起来像“符号 + 前导 0 + 数字”，可以使用 `internal`。

##### 对齐方式也是“粘住的”

`left`、`right`、`internal` 和 `fixed` 一样，会一直影响同一个输出流，直到你改成其他对齐方式。

```cpp
std::cout << std::left << std::setw(8) << "A" << std::endl;
std::cout << std::setw(8) << "B" << std::endl;       // 仍然左对齐
std::cout << std::right << std::setw(8) << "C" << std::endl; // 改回右对齐
```

---

#### 5.4.4 `setw` 在输入中的少见用法

`setw` 也可以用于输入，不过实际开发和算法题中用得不多。它的主要作用是限制下一次读取的最大宽度，常见于读取 C 风格字符数组时避免数组越界。

```cpp
#include <iostream>
#include <iomanip>

int main() {
    char name[10];

    std::cin >> std::setw(10) >> name;  // 最多读取 9 个有效字符，最后 1 个位置留给 '\0'
    std::cout << name << std::endl;

    return 0;
}
```

如果输入：

```
Alexanderson
```

`name` 中会存入：

```
Alexander
```

最后一个位置用于存放字符串结束标记 `'\0'`。

对于 `std::string`，`setw(n)` 也能限制下一次 `>>` 最多读取 `n` 个字符，但这种写法不如直接使用 `getline` 或自己处理字符串常见。

```cpp
std::string s;
std::cin >> std::setw(5) >> s;   // 最多读取 5 个字符到 s
```

需要注意：用于输入时，`setw` 同样只影响下一次读取操作。

---

### 5.5 整数进制转换输出：`dec`、`hex`、`oct` 与 `bitset`

除了以十进制输出数字，C++ 还能非常方便地以十六进制、八进制或二进制输出。

#### 1. 十进制、十六进制、八进制操纵符

- `dec`：十进制（默认）
- `hex`：十六进制
- `oct`：八进制
- `showbase`：显示进制前缀（十六进制显示 `0x`，八进制显示 `0`）
- `uppercase`：十六进制字母大写显示（如 `0X1A` 代替 `0x1a`）

```cpp
#include <iostream>
#include <iomanip>

int main() {
    int val = 255;

    std::cout << "默认十进制: " << val << std::endl;
    std::cout << "十六进制: " << std::hex << val << std::endl;
    std::cout << "带前缀十六进制: " << std::showbase << val << std::endl;
    std::cout << "大写十六进制: " << std::uppercase << val << std::endl;
    std::cout << "八进制: " << std::oct << val << std::endl;

    // 恢复十进制、取消前缀、取消大写（这些设置都是粘住的！）
    std::cout << std::dec << std::noshowbase << std::nouppercase;
    std::cout << "恢复十进制: " << val << std::endl;

    return 0;
}
```

输出：
```
默认十进制: 255
十六进制: ff
带前缀十六进制: 0xff
大写十六进制: 0XFF
八进制: 0377
恢复十进制: 255
```

#### 2. 二进制输出神器：`std::bitset`

流操纵符中没有直接提供 `bin`，但标准库提供了 `<bitset>` 头文件，它是输出二进制最清晰的方法：

```cpp
#include <iostream>
#include <bitset>

int main() {
    int n = 42;

    // bitset<位数>(数值)
    std::cout << "8 位二进制:  " << std::bitset<8>(n) << std::endl;
    std::cout << "16 位二进制: " << std::bitset<16>(n) << std::endl;

    return 0;
}
```

输出：
```
8 位二进制:  00101010
16 位二进制: 0000000000101010
```

---

### 5.6 布尔值与正负号显示：`boolalpha` 与 `showpos`

#### 1. 布尔值输出形式

默认情况下，`cout` 输出 `bool` 类型变量时会输出数字 `1` 或 `0`。使用 `boolalpha` 可以让其直接显示为单词 `true` 或 `false`。

```cpp
#include <iostream>

int main() {
    bool flag1 = true, flag2 = false;

    std::cout << flag1 << " " << flag2 << std::endl;             // 1 0
    std::cout << std::boolalpha << flag1 << " " << flag2 << std::endl; // true false
    std::cout << std::noboolalpha << flag1 << std::endl;              // 1 (恢复默认)

    return 0;
}
```

#### 2. 正数显示正号 `+`

有时为了对比正负数（如温度、涨跌幅），希望正数也带上 `+` 号：

```cpp
#include <iostream>

int main() {
    int diff = 15;
    std::cout << std::showpos << diff << std::endl;   // +15
    std::cout << std::noshowpos << diff << std::endl; // 15 (恢复默认)
    return 0;
}
```

---

### 5.7 `ostringstream` —— 格式化构造字符串

#### 为什么需要它？
有时你并不是想立刻把内容打印到屏幕上，而是想**在内存中拼装出一个复杂的格式化字符串**，供后续网络发送、日志保存或 UI 显示。

`ostringstream`（头文件 `<sstream>`）提供了与 `cout` 完全一致的接口，但所有输出都会写入到一个内存缓冲区中，最后通过 `.str()` 提取出完整的 `std::string`。

#### 示例：组装学生报告卡字符串

```cpp
#include <iostream>
#include <sstream>
#include <iomanip>
#include <string>

std::string makeStudentReport(const std::string& name, int id, double gpa) {
    std::ostringstream oss;

    oss << "【学籍卡】"
        << "学号: " << std::setfill('0') << std::setw(6) << id << " | "
        << "姓名: " << std::left << std::setw(10) << name << " | "
        << "GPA: " << std::fixed << std::setprecision(2) << gpa;

    return oss.str(); // 导出为 std::string
}

int main() {
    std::string report = makeStudentReport("李华", 42, 3.856);
    std::cout << report << std::endl;
    return 0;
}
```

输出：
```
【学籍卡】学号: 000042 | 姓名: 李华       | GPA: 3.86
```

---

### 5.8 C 语言风格输出：`printf`、`putchar`、`puts` 与 `sprintf`

虽然 C++ 推荐使用类型安全的 `cout`，但 C 风格的输出函数由于语法紧凑、在旧代码和部分竞赛代码中依然极其常见。

#### 1. `printf` 格式化占位符体系

```cpp
#include <cstdio>
```

基本格式串语法：`%[标志][最小宽度][.精度][长度修饰符]类型转换符`

| 占位符   | 含义                           | 示例代码                       | 样例输出     |
| :------- | :----------------------------- | :----------------------------- | :----------- |
| `%d`     | 有符号十进制 `int`             | `printf("%d", 123)`            | `123`        |
| `%lld`   | 64位有符号 `long long`         | `printf("%lld", 12345678901LL)`| `12345678901`|
| `%u`     | 无符号十进制 `unsigned int`    | `printf("%u", 3000000000U)`    | `3000000000` |
| `%f`     | 浮点数（保留默认 6 位小数）    | `printf("%f", 3.14)`           | `3.140000`   |
| `%.2f`   | **保留 2 位小数（四舍五入）**  | `printf("%.2f", 3.14159)`      | `3.14`       |
| `%05d`   | **最小宽度 5，前导补 0**       | `printf("%05d", 42)`           | `00042`      |
| `%-8s`   | **最小宽度 8，左对齐**         | `printf("[%-8s]", "abc")`      | `[abc     ]` |
| `%x`     | 十六进制小写                   | `printf("%x", 255)`            | `ff`         |
| `%X`     | 十六进制大写                   | `printf("%X", 255)`            | `FF`         |
| `%c`     | 单个字符                       | `printf("%c", 'A')`            | `A`          |
| `%s`     | C 风格字符串 `char*`           | `printf("%s", "Hello")`        | `Hello`      |

> [!CAUTION]
> - `printf` **输出 `std::string` 时必须调用 `.c_str()`**：
>   ```cpp
>   std::string s = "Hello";
>   // printf("%s", s); // 严重错误！类型不匹配，未定义行为
>   printf("%s", s.c_str()); // 正确
>   ```
> - `printf` 输出 `double` 和 `float` 都可以用 `%f`（`double` 会自动提升），但写 `%lf` 也是合法的。

#### 2. `putchar(ch)` —— 极速单字符输出

输出单个字符的最轻量级函数，内部没有任何格式解析开销，常用于快写（Fast Write）函数：

```cpp
#include <cstdio>

int main() {
    putchar('A');
    putchar('\n');
    return 0;
}
```

#### 3. `puts(str)` —— 极简字符串行输出

`puts` 接收一个字符指针 `const char*`，并**自动在末尾追加一个换行符 `\n`**。它的速度非常快，非常适合输出结果为 `"YES"` / `"NO"` 的题目：

```cpp
#include <cstdio>

int main() {
    // 自动换行，无需手动写 \n
    puts("YES");
    puts("NO");
    return 0;
}
```

#### 4. `sprintf` 与安全的 `snprintf`

将格式化内容输出到字符数组（内存缓冲区）：

```cpp
#include <cstdio>

int main() {
    char buf[50];
    int score = 98;
    // 建议使用 snprintf 指定缓冲区大小，防止缓冲区溢出
    snprintf(buf, sizeof(buf), "Student Score: %03d", score);
    puts(buf); // 输出: Student Score: 098
    return 0;
}
```

---

### 5.9 现代 C++ 革命性格式化：C++20 `<format>` 与 C++23 `std::print`

长期以来，C++ 开发者一直在 `cout`（安全但语法冗长繁琐）和 `printf`（语法紧凑但不安全且不支持自定义类型）之间纠结。现代 C++ 彻底解决了这一百年痛点！

#### 1. C++20 `std::format`（类似 Python f-string）

需要编译器支持 C++20，包含头文件 `<format>`：

```cpp
#include <iostream>
#include <format>
#include <string>

int main() {
    std::string name = "Alice";
    int id = 7;
    double balance = 1234.567;

    // 占位符使用 {}，支持类似 Python 的格式化语法
    std::string msg = std::format("ID: {:04d}, 用户: {}, 余额: {:.2f}", id, name, balance);
    std::cout << msg << std::endl;

    return 0;
}
```

输出：
```
ID: 0007, 用户: Alice, 余额: 1234.57
```

**语法亮点**：
- `{}`：默认按变量类型自动匹配。
- `{:04d}`：补 0 到 4 位。
- `{:.2f}`：浮点数保留 2 位小数。
- `{:>10}` / `{:<10}`：右对齐 / 左对齐并设置宽度。

#### 2. C++23 `std::print` 与 `std::println`（终极输出方案）

在 C++23 中，包含 `<print>` 头文件后，可以直接使用 `print` / `println` 打印，无需任何管道符号或临时对象，性能直接拉满：

```cpp
#include <print>

int main() {
    // println 自动换行
    std::println("Hello, {}! 成绩是 {:.1f}", "Bob", 95.5);
    return 0;
}
```

---

### 5.10 常用输出格式操纵符速查表

| 需求分类       | 操纵符 / 语法                                | 作用与示例                                    | 是否粘住（Sticky） |
| :------------- | :------------------------------------------- | :-------------------------------------------- | :----------------: |
| **浮点精度**   | `setprecision(n)`                            | 默认表示有效数字位数                          | 是                 |
|                | `fixed << setprecision(n)`                   | **小数点后固定保留 n 位（四舍五入）**         | 是                 |
|                | `scientific`                                 | 使用科学计数法                                | 是                 |
|                | `defaultfloat`                               | 恢复默认浮点显示                              | 是                 |
| **字段宽度**   | `setw(n)`                                    | 设置下一个输出项最小宽度（内容超长不截断）    | **否（仅下一次）** |
| **空白填充**   | `setfill(ch)`                                | 设置不足宽度时的填充字符（配合 `setw` 补 0）  | 是                 |
| **对齐方式**   | `left`                                       | 左对齐（填充在右侧）                          | 是                 |
|                | `right`                                      | 右对齐（默认，填充在左侧）                    | 是                 |
|                | `internal`                                   | 符号居左，填充字符夹在符号与数值之间          | 是                 |
| **进制转换**   | `dec` / `hex` / `oct`                        | 十进制 / 十六进制 / 八进制输出                | 是                 |
|                | `showbase` / `noshowbase`                    | 显示/关闭进制前缀（`0x` / `0`）               | 是                 |
|                | `uppercase` / `nouppercase`                  | 十六进制字母大写（`0XFF`）/ 小写              | 是                 |
|                | `bitset<N>(val)`                             | 借助 `<bitset>` 输出 N 位二进制字符串         | 单次表达式         |
| **布尔与符号** | `boolalpha` / `noboolalpha`                  | 输出 `true`/`false` 还是 `1`/`0`              | 是                 |
|                | `showpos` / `noshowpos`                      | 正数前显示 `+` 号                             | 是                 |
| **换行控制**   | `'\n'`                                       | 仅输出换行字符，性能极高                      | 单次               |
|                | `endl`                                       | 输出换行符并**强制刷新缓冲区**，性能损耗大    | 单次               |

---

## 六、I/O 性能深度优化（算法竞赛与高并发必备）

在各类算法竞赛（ACM / ICPC、Codeforces、LeetCode、洛谷、蓝桥杯）中，常常遇到数据规模在 $10^5 \sim 10^7$ 的题目。如果 I/O 方式使用不当，即使算法时间复杂度完全正确，也会因为读写耗时过长而被判定为 **TLE（Time Limit Exceeded，超时）**。

---

### 6.1 为什么 C++ 的 `cin / cout` 默认很慢？

很多初学者甚至部分高校课程认为“C++ 的 `cin/cout` 比 C 语言的 `scanf/printf` 慢几倍甚至几十倍”。**这个结论是片面的！** 默认情况下它们确实慢，但根本原因并非 C++ 流机制本身性能不行，而是标准库做出了以下两项安全妥协：

#### 妥协一：与 C 标准 I/O 的同步机制（Synchronization）
为了允许程序员在同一个程序中自由交替混用 `cin` 和 `scanf`、`cout` 和 `printf` 且不会出现输出乱序，C++ 的流底层默认绑定了 C 语言的 `stdio` 缓冲区。每一次 `cin >>` 或 `cout <<` 都需要进行繁琐的同步检查，产生巨大的性能损耗。

#### 妥协二：`cin` 与 `cout` 的相互绑定（Tie）
在默认情况下，`cin` 是绑定（tied）到 `cout` 上的。也就是说：**每次调用 `cin >>` 准备读取数据前，系统都会强行执行一次 `cout.flush()` 刷新输出缓冲区**。这对于控制台互动程序很友好（确保在用户输入前能看到提示语），但在算法竞赛中，这种频繁的缓冲区刷新极其致命。

---

### 6.2 经典三步加速法：解除同步与解绑

只需在 `main()` 函数的最开始加上两行代码，`cin/cout` 的速度就可以**瞬间提升到超越 `scanf/printf` 的级别**！

```cpp
#include <iostream>

int main() {
    // 1. 关闭与 C stdio 的同步（核心加速代码）
    std::ios::sync_with_stdio(false);

    // 2. 解除 cin 与 cout 的绑定（避免每次读入前自动刷新输出）
    std::cin.tie(nullptr);

    // 3. （可选）解除 cout 自身绑定的输出流
    std::cout.tie(nullptr);

    // 从此处开始，所有的读入输出都使用 cin 和 cout
    int n;
    while (std::cin >> n) {
        std::cout << n * 2 << '\n'; // 务必使用 '\n' 代替 endl！
    }

    return 0;
}
```

> [!CAUTION]
> #### 唯一的终极铁律与避坑指南
> 一旦执行了 `ios::sync_with_stdio(false);`，**绝对不要在同一个程序中混用任何 C 风格的 I/O 函数**（如 `scanf`、`printf`、`getchar`、`putchar`、`puts`）！
> 否则两套独立的缓冲区会导致数据完全错位、输出顺序彻底颠倒！

---

### 6.3 极致性能：快读（Fast I/O）与快写模板

当题目输入数据量达到 $10^6$ 甚至 $10^7$（例如百万级大图遍历、海量长整型输入），即使是加速后的 `cin` 也可能面临压线超时的风险。此时，算法竞赛选手会使用基于底层字符扫描的**快读（Fast Read）**与**快写（Fast Write）**。

#### 1. 经典快读模板（基于 `getchar`）

原理：利用极速的 `getchar()` 逐个读取字符，跳过空白符，识别负号，手动通过数值累加 `x = (x << 1) + (x << 3) + (ch ^ 48)` 构建整数。

```cpp
#include <cstdio>
#include <cctype>

// 模板函数：支持 int, long long, unsigned 等所有整型
template <typename T>
inline void read(T& x) {
    x = 0;
    int f = 1;
    int ch = getchar();

    // 1. 跳过非数字字符，同时检测是否有负号
    while (!std::isdigit(ch)) {
        if (ch == '-') f = -1;
        ch = getchar();
    }

    // 2. 累加数值位：(x << 1) + (x << 3) 等价于 x * 10
    //    ch ^ 48 等价于 ch - '0'
    while (std::isdigit(ch)) {
        x = (x << 1) + (x << 3) + (ch ^ 48);
        ch = getchar();
    }
    x *= f;
}
```

#### 2. 经典快写模板（基于 `putchar`）

原理：数字转换为字符时通常从个位开始取余，顺序是反的。利用递归或栈将其反转后，通过 `putchar()` 极速输出。

```cpp
// 模板函数：递归快写整数
template <typename T>
inline void write(T x) {
    if (x < 0) {
        putchar('-');
        x = -x;
    }
    if (x > 9) {
        write(x / 10);
    }
    putchar(x % 10 + '0');
}

// 辅助快速输出换行
template <typename T>
inline void writeln(T x) {
    write(x);
    putchar('\n');
}
```

#### 3. 终极性能怪兽：基于 `fread` 的成块读取（卡常神器）

如果题目时限卡到极限（例如 1000 万个整数要在 1 秒内读入），`getchar` 每次调用函数的开销依然存在。此时最强武器是 `fread`：**一次性将几兆甚至上百兆的数据整块装入大内存缓冲区**，性能直接达到硬件上限。

```cpp
#include <cstdio>
#include <cctype>

namespace FastIO {
    const int BUF_SIZE = 1 << 20; // 1MB 缓冲区
    char buf[BUF_SIZE];
    int p1 = 0, p2 = 0;

    // 自定义高效 getchar
    inline char getChar() {
        if (p1 == p2) {
            p2 = fread(buf, 1, BUF_SIZE, stdin);
            p1 = 0;
            if (p1 == p2) return EOF;
        }
        return buf[p1++];
    }

    template <typename T>
    inline void read(T& x) {
        x = 0;
        int f = 1;
        char ch = getChar();
        while (!std::isdigit(ch)) {
            if (ch == '-') f = -1;
            ch = getChar();
        }
        while (std::isdigit(ch)) {
            x = (x << 1) + (x << 3) + (ch ^ 48);
            ch = getChar();
        }
        x *= f;
    }
}
```

---

## 七、总结与全景速查表

### 7.1 各输入方式全景横向对比

| 方法                       | 头文件       | 适用场景                       | 是否跳前导空白 | 是否吃掉换行符 | 速度评级 | 安全性 | 推荐指数 |
| :------------------------- | :----------- | :----------------------------- | :------------: | :------------: | :------: | :----: | :------: |
| **`cin >>`**               | `<iostream>` | 读取单个数值、无空格单词       |       是       |     留在流     |  极快*   | 极高   | ★★★★★    |
| **`getline(cin, str)`**    | `<string>`   | 读取包含空格的整行文本         |       否       |   取走并丢弃   |  极快*   | 极高   | ★★★★★    |
| **`getline(..., delim)`**  | `<string>`   | 读取以逗号等特定字符分隔的字段 |       否       |     不适用     |   极快   | 极高   | ★★★★☆    |
| **`istringstream`**        | `<sstream>`  | 内存字符串逐个解析不定项数据   |    同 `>>`     |     不适用     |    快    | 极高   | ★★★★☆    |
| **`scanf`**                | `<cstdio>`   | C 风格格式化匹配（如指定日期） |   是（除%c）   |     留在流     |   极快   |  中等  | ★★★★☆    |
| **`cin.get(ch)`**          | `<iostream>` | 精确读取下一个字符（含空白）   |       否       |     不适用     |    快    |  极高  | ★★★☆☆    |
| **`getchar()`**            | `<cstdio>`   | 极速读取下一个字符             |       否       |     不适用     |  超极快  |  中等  | ★★★★☆    |
| **`cin.getline(buf, n)`**  | `<iostream>` | C 风格字符数组整行读取         |       否       |   取走并丢弃   |    快    | 需控长 | ★★☆☆☆    |
| **`fgets`**                | `<cstdio>`   | C 风格安全行读取               |       否       |  保留在字符串  |    快    |  良好  | ★★☆☆☆    |
| **`ifstream`**             | `<fstream>`  | 各种形式的文件批量读取         |   同对应流方法 |   同对应流方法 |   极快   |  极高  | ★★★★★    |
| **Fast I/O 快读**          | 自定义       | 百万级海量数据的极端性能场景   |       是       |     不适用     |   封神   |  高    | 算法特选 |

*\*注：`cin` 和 `getline` 的“极快”建立在开启 `sync_with_stdio(false)` 并解绑的前提下。*

---

### 7.2 各输出方式全景横向对比

| 方法                       | 头文件       | 适用场景                       | 是否支持链式 | 格式化灵活性 | 速度评级 | 推荐指数 |
| :------------------------- | :----------- | :----------------------------- | :----------: | :----------: | :------: | :------: |
| **`cout <<` (`\n`)**       | `<iostream>` | 通用业务、算法竞赛默认输出     |      是      | 搭配 iomanip |   极快   | ★★★★★    |
| **`printf`**               | `<cstdio>`   | 紧凑排版、指定小数与补零       |      否      |  占位符强大  |   极快   | ★★★★☆    |
| **`puts(str)`**            | `<cstdio>`   | 字符串输出并自动追加换行       |      否      |      无      |  超极快  | ★★★★☆    |
| **`putchar(ch)`**          | `<cstdio>`   | 单个字符快速打出               |      否      |      无      |   极高   | ★★★★☆    |
| **`ostringstream`**        | `<sstream>`  | 拼装格式化字符串（内存操作）   |      是      | 搭配 iomanip |    快    | ★★★★☆    |
| **`ofstream`**             | `<fstream>`  | 文件持久化写入（支持覆盖/追加）|      是      | 搭配 iomanip |   极快   | ★★★★★    |
| **`std::format` (C++20)**  | `<format>`   | 类似 Python f-string 优雅排版  |      否      |    极强大    |   极快   | ★★★★★    |
| **`std::println` (C++23)** | `<print>`    | 现代 C++ 终极控制台格式化打印  |      否      |    极强大    |   极快   | ★★★★★    |
| **Fast Write 快写**        | 自定义       | 百万级海量数据极限输出         |      否      |    手写转换  |   封神   | 算法特选 |

---

### 7.3 I/O 选型决策树

#### 1. 输入选型决策树
```
需要读取输入 ──┬─ 普通数字/无空格单词？ ──────────────────────────────→ cin >> var （竞赛记得加速）
               │
               ├─ 包含空格的一整行？ ─────────────────────────────────→ getline(cin, str)
               │
               ├─ 一行中数量不固定的整数/单词？ ──────────────────────→ getline 读入整行 + istringstream 提取
               │
               ├─ 单个字符（需要保留空格和换行符）？ ─────────────────→ cin.get(ch) 或 ch = getchar()
               │
               ├─ 固定格式文本（如 "2026-09-27"）？ ──────────────────→ scanf("%d-%d-%d") 或 getline 搭配 ','
               │
               ├─ 来自本地文件而非控制台？ ───────────────────────────→ freopen 重定向（首选）或 ifstream
               │
               ├─ cin >> 后紧接着要用 getline？ ──────────────────────→ 务必先 cin.ignore() 或 getline(cin >> ws, str)
               │
               └─ 数据量达 10^6 ~ 10^7，要求极致耗时？ ───────────────→ 快读 read(x)
```

#### 2. 输出选型决策树
```
需要产生输出 ──┬─ 日常与算法题通用输出？ ─────────────────────────────→ cout << ans << '\n'（坚决不用 endl）
               │
               ├─ 控制小数位数、宽度、补零对齐？ ─────────────────────→ cout << fixed << setprecision(n) << setw(w)
               │                                                      （或使用 printf("%.2f", x)）
               │
               ├─ 仅输出单个结果字符串并换行（如 "YES"/"NO"）？ ──────→ puts("YES")
               │
               ├─ 严重错误报警、崩溃前追踪日志？ ─────────────────────→ cerr << "[Error] ..." << endl（无缓冲立刻刷出）
               │
               ├─ 组装复杂字符串用于存盘或传参？ ─────────────────────→ ostringstream 或 std::format (C++20)
               │
               ├─ 保存到文件持久化？ ─────────────────────────────────→ ofstream fout("file.txt", ios::app)
               │
               └─ 算法题输出量在 10^6 以上面临卡常？ ─────────────────→ 快写 write(x) + putchar('\n')
```

---

### 7.4 初学者与竞赛避坑黄金法则

1. **`endl` 禁令**：永远记住 `endl` 等于 `'\n' + flush`。没有特殊交互需求，一律写 `'\n'`。
2. **加速双剑客**：算法竞赛第一行必写 `ios::sync_with_stdio(false); cin.tie(nullptr);`。
3. **不同体系切忌混用**：关闭 `sync_with_stdio` 之后，**绝不能在同一个代码里同时使用 `cin/cout` 和 `scanf/printf`**。
4. **换行残留陷阱**：`cin >>` 之后调用 `getline`，一定要用 `cin.ignore()` 或者 `cin >> ws` 滤掉残留的换行符。
5. **格式粘连意识**：`fixed`, `setprecision`, `setfill`, `hex`, `left` 是**粘住的**，设置一次会一直影响后续输出；而 `setw` **只生效一次**！
6. **`double` 输入输出差异**：使用 `scanf` 读入 double **必须是 `%lf`**；而 `printf` 输出 double 用 `%f` 或 `%lf` 均可。
7. **`EOF` 判定机制**：算法题多组数据循环输入，推荐 `while (cin >> a >> b)` 或 `while (scanf("%d%d", &a, &b) == 2)`。

