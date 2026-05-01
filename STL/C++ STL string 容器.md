# C++ STL `string` 容器完全教程

> 🎯 本教程面向 C++ 初学者，由浅入深，系统讲解 `std::string` 的核心操作。每个知识点都配有**详细注释的代码示例**。

---

## 📌 前置知识

`std::string` 本质上是 C++ 标准库对 C 风格字符串（`char[]`）的**面向对象封装**。它自动管理内存，使用起来比 `char*` 安全得多。

```cpp
#include <string>   // 必须包含此头文件
#include <iostream>
using namespace std;
```

---

## 一、string 容器 — 构造函数

`string` 提供了多种构造方式，满足不同场景的初始化需求。

### 1.1 四种核心构造方式

```cpp
#include <iostream>
#include <string>
using namespace std;

void test01() {
    // ① 默认构造：创建一个空字符串
    string s1;
    cout << "s1 = \"" << s1 << "\"" << endl;  // s1 = ""

    // ② 使用 C 风格字符串（const char*）构造
    string s2("Hello, STL!");
    cout << "s2 = " << s2 << endl;  // s2 = Hello, STL!

    // ③ 拷贝构造：用另一个 string 对象来初始化
    string s3(s2);
    cout << "s3 = " << s3 << endl;  // s3 = Hello, STL!

    // ④ 用 n 个相同字符构造
    string s4(10, 'A');
    cout << "s4 = " << s4 << endl;  // s4 = AAAAAAAAAA
}

int main() {
    test01();
    return 0;
}
```

### 1.2 补充：其他实用构造方式

```cpp
void test02() {
    string src = "Hello, World!";

    // ⑤ 截取子串构造：从 pos 位置开始，取 len 个字符
    // string(const string& str, size_t pos, size_t len)
    string s5(src, 7, 5);
    cout << "s5 = " << s5 << endl;  // s5 = World

    // ⑥ 用 C 风格字符串的前 n 个字符构造
    // string(const char* s, size_t n)
    string s6("Hello, World!", 5);
    cout << "s6 = " << s6 << endl;  // s6 = Hello

    // ⑦ 用迭代器范围构造（后面学到迭代器时会更理解）
    string s7(src.begin(), src.begin() + 5);
    cout << "s7 = " << s7 << endl;  // s7 = Hello
}
```

> 💡 **初学者建议**：先掌握前四种，它们覆盖了 90% 的使用场景。

---

## 二、string 容器 — 赋值操作

已经构造好的 `string`，可以通过赋值操作来修改其内容。

### 2.1 使用 `=` 运算符赋值

```cpp
void test03() {
    string s1, s2, s3, s4;

    // ① 用 C 风格字符串赋值
    s1 = "Hello";
    cout << "s1 = " << s1 << endl;  // Hello

    // ② 用另一个 string 对象赋值
    s2 = s1;
    cout << "s2 = " << s2 << endl;  // Hello

    // ③ 用单个字符赋值（整个字符串变成这一个字符）
    s3 = 'X';
    cout << "s3 = " << s3 << endl;  // X
}
```

### 2.2 使用 `assign()` 成员函数赋值

`assign()` 提供了更丰富的赋值方式：

```cpp
void test04() {
    string s1, s2, s3, s4;

    // ① assign(const char* s) — 用 C 风格字符串赋值
    s1.assign("Hello, C++!");
    cout << "s1 = " << s1 << endl;  // Hello, C++!

    // ② assign(const string& str) — 用另一个 string 赋值
    s2.assign(s1);
    cout << "s2 = " << s2 << endl;  // Hello, C++!

    // ③ assign(const char* s, size_t n) — 取前 n 个字符赋值
    s3.assign("Hello, C++!", 5);
    cout << "s3 = " << s3 << endl;  // Hello

    // ④ assign(size_t n, char c) — 用 n 个相同字符赋值
    s4.assign(8, 'W');
    cout << "s4 = " << s4 << endl;  // WWWWWWWW
}
```

> 💡 **`=` vs `assign()`**：日常开发中 `=` 更常用、更简洁。`assign()` 在需要**截取赋值**或**重复字符赋值**时才有优势。

---

## 三、string 容器 — 字符串拼接

拼接是最常用的字符串操作之一。`string` 提供了两种主要方式。

### 3.1 使用 `+=` 运算符

```cpp
void test05() {
    string s = "I";

    // ① 拼接 C 风格字符串
    s += " love";
    cout << s << endl;  // I love

    // ② 拼接单个字符
    s += ' ';
    cout << s << endl;  // I love 

    // ③ 拼接另一个 string 对象
    string s2 = "C++!";
    s += s2;
    cout << s << endl;  // I love C++!
}
```

### 3.2 使用 `append()` 成员函数

```cpp
void test06() {
    string s = "Hello";

    // ① append(const char* s) — 追加 C 风格字符串
    s.append(", ");
    cout << s << endl;  // Hello, 

    // ② append(const char* s, int n) — 追加前 n 个字符
    s.append("World!!!", 5);  // 只追加 "World"
    cout << s << endl;  // Hello, World

    // ③ append(const string& str) — 追加另一个 string
    string s2 = "!!!";
    s.append(s2);
    cout << s << endl;  // Hello, World!!!

    // ④ append(const string& str, int pos, int n) — 从 str 的 pos 位置截取 n 个字符追加
    string s3 = "xxxABCxxx";
    string s4 = "Result: ";
    s4.append(s3, 3, 3);  // 从位置 3 开始取 3 个字符 "ABC"
    cout << s4 << endl;   // Result: ABC
}
```

> 💡 **`+=` vs `append()`**：和赋值类似，`+=` 更简洁直观；`append()` 在需要**部分截取拼接**时更灵活。

---

## 四、string 容器 — 字符串查找和替换

### 4.1 查找：`find()` 和 `rfind()`

```cpp
void test07() {
    string s = "Hello, World! Hello, C++!";

    // ============ find：从左往右查找 ============

    // ① 查找子串首次出现的位置（返回下标，找不到返回 string::npos）
    size_t pos1 = s.find("Hello");
    cout << "find(\"Hello\") = " << pos1 << endl;  // 0

    // ② 从指定位置开始查找
    size_t pos2 = s.find("Hello", 1);  // 从下标 1 开始找
    cout << "find(\"Hello\", 1) = " << pos2 << endl;  // 14

    // ③ 查找单个字符
    size_t pos3 = s.find('W');
    cout << "find('W') = " << pos3 << endl;  // 7

    // ④ 查找不存在的内容
    size_t pos4 = s.find("Java");
    if (pos4 == string::npos) {
        cout << "\"Java\" 未找到！" << endl;  // 未找到
    }

    // ============ rfind：从右往左查找 ============

    // rfind 查找的是子串最后一次出现的位置
    size_t pos5 = s.rfind("Hello");
    cout << "rfind(\"Hello\") = " << pos5 << endl;  // 14
}
```

> ⚠️ **关键区别**：
>
> 
>
> - `find()` → 从**左往右**找，返回**第一次**出现的位置
>
> - `rfind()` → 从**右往左**找，返回**最后一次**出现的位置
>
> - 找不到时，两者都返回 `string::npos`（通常是一个极大的值）

### 4.2 替换：`replace()`

```cpp
void test08() {
    string s = "Hello, World!";

    // replace(pos, n, str)
    // 从位置 pos 开始，将接下来的 n 个字符替换为 str
    s.replace(7, 5, "C++");
    cout << s << endl;  // Hello, C++!
    // 注意：原来 "World" 是 5 个字符，"C++" 是 3 个字符
    // replace 不要求替换前后长度相同，string 会自动调整

    // 再来一个例子：替换后的字符串更长
    string s2 = "I like cats";
    s2.replace(7, 4, "dinosaurs");
    cout << s2 << endl;  // I like dinosaurs
}
```

> 💡 **replace 的要点**：
>
> 
>
> - 第一个参数：**起始位置**（下标）
>
> - 第二个参数：要被替换的**字符个数**
>
> - 第三个参数：**新的字符串**（长度可以不同）

---

## 五、string 容器 — 字符串比较

### 5.1 使用 `compare()` 成员函数

`compare()` 按 ASCII 码逐字符比较，返回值的含义：

| 返回值 | 含义           |
| ------ | -------------- |
| 0      | 两个字符串相等 |
| > 0    | 调用者大于参数 |
| < 0    | 调用者小于参数 |

```cpp
void test09() {
    string s1 = "hello";
    string s2 = "hello";
    string s3 = "world";
    string s4 = "Hello";  // 注意大写 'H'

    // ① 相等
    int result1 = s1.compare(s2);
    cout << "\"hello\" vs \"hello\" = " << result1 << endl;  // 0

    // ② 小于（'h' 的 ASCII 是 104，'w' 是 119）
    int result2 = s1.compare(s3);
    cout << "\"hello\" vs \"world\" = " << result2 << endl;  // < 0

    // ③ 大于（'h' 是 104，'H' 是 72）
    int result3 = s1.compare(s4);
    cout << "\"hello\" vs \"Hello\" = " << result3 << endl;  // > 0
}
```

### 5.2 更推荐：直接使用关系运算符

```cpp
void test10() {
    string s1 = "apple";
    string s2 = "banana";

    // 直接用 ==, !=, <, >, <=, >= 比较，更直观！
    if (s1 == s2) {
        cout << "相等" << endl;
    } else if (s1 < s2) {
        cout << "\"" << s1 << "\" < \"" << s2 << "\"" << endl;
        // 输出：apple < banana（因为 'a' < 'b'）
    }

    // 判断相等时，== 比 compare() 直观得多
    string password = "abc123";
    if (password == "abc123") {
        cout << "密码正确！" << endl;
    }
}
```

> 💡 **实际开发建议**：
>
> 
>
> - 判断**是否相等** → 直接用 `==`
>
> - 判断**大小关系** → 直接用 `<`、`>` 等
>
> - `compare()` 主要在需要**三路比较**（小于/等于/大于一次性判断）时使用

---

## 六、string 容器 — 字符存取

如何访问和修改字符串中**单个字符**？

### 6.1 两种方式：`[]` 和 `at()`

```cpp
void test11() {
    string s = "Hello";

    // ============ 读取字符 ============

    // ① 使用 [] 下标方式访问
    for (size_t i = 0; i < s.size(); ++i) {
        cout << s[i] << " ";
    }
    cout << endl;  // H e l l o

    // ② 使用 at() 成员函数访问
    for (size_t i = 0; i < s.size(); ++i) {
        cout << s.at(i) << " ";
    }
    cout << endl;  // H e l l o

    // ============ 修改单个字符 ============

    // ③ 用 [] 修改
    s[0] = 'h';
    cout << s << endl;  // hello

    // ④ 用 at() 修改
    s.at(4) = 'O';
    cout << s << endl;  // hellO
}
```

### 6.2 `[]` 与 `at()` 的关键区别

```cpp
void test12() {
    string s = "Hello";

    // [] 越界访问：行为未定义（可能崩溃，也可能返回垃圾值，不会报错提示）
    // cout << s[100];  // ⚠️ 未定义行为！不要这样做

    // at() 越界访问：抛出 std::out_of_range 异常（更安全）
    try {
        char c = s.at(100);  // 越界！
    } catch (const out_of_range& e) {
        cout << "捕获异常: " << e.what() << endl;
        // 输出类似：捕获异常: invalid string position
    }
}
```

> 💡 **选哪个？**
>
> 
>
> - `[]`：**更快**（不做越界检查），适合你确认下标合法时使用
>
> - `at()`：**更安全**（越界时抛异常），适合不确定下标是否合法时使用
>
> - 现代 C++ 中，如果只是遍历，更推荐 **range-based for**：

```cpp
void test13() {
    string s = "Hello";

    // 只读遍历
    for (char c : s) {
        cout << c << " ";
    }
    cout << endl;  // H e l l o

    // 可修改遍历（注意引用 &）
    for (char& c : s) {
        c = toupper(c);  // 转大写
    }
    cout << s << endl;  // HELLO
}
```

### 6.3 补充：`front()` 和 `back()`

```cpp
void test14() {
    string s = "Hello";

    // front() 返回第一个字符的引用
    cout << "首字符: " << s.front() << endl;  // H

    // back() 返回最后一个字符的引用
    cout << "尾字符: " << s.back() << endl;  // o

    // 可以直接修改
    s.front() = 'J';
    s.back() = '!';
    cout << s << endl;  // Jell!
}
```

---

## 七、string 容器 — 字符串插入和删除

### 7.1 插入：`insert()`

```cpp
void test15() {
    string s = "Hello!";

    // ① insert(pos, str) — 在位置 pos 前插入字符串
    s.insert(5, ", World");
    cout << s << endl;  // Hello, World!

    // ② insert(pos, n, char) — 在位置 pos 前插入 n 个字符
    string s2 = "AC";
    s2.insert(1, 3, 'B');   // 在位置 1 插入 3 个 'B'
    cout << s2 << endl;     // ABBBC

    // ③ insert(pos, str, start, len) — 插入 str 从 start 开始的 len 个字符
    string s3 = "Hello!";
    string src = "xxx World xxx";
    s3.insert(5, src, 3, 6);   // 从 src 的位置 3 开始取 6 个字符 " World"
    cout << s3 << endl;        // Hello World!
}
```

### 7.2 删除：`erase()`

```cpp
void test16() {
    string s = "Hello, World!";

    // ① erase(pos, n) — 从位置 pos 开始删除 n 个字符
    s.erase(5, 7);   // 从位置 5 删除 7 个字符 ", World" 被删掉
    cout << s << endl;  // Hello!

    // ② erase() / clear() — 清空字符串
    string s2 = "Something";
    s2.erase();          // 等价于 s2.clear()
    cout << "s2 为空: " << s2.empty() << endl;  // 1 (true)

    // ③ 使用迭代器删除单个字符
    string s3 = "Hellol";  // 多了一个 'l'
    s3.erase(s3.begin() + 4);  // 删除位置 4 的字符
    cout << s3 << endl;  // Hello

    // ④ 使用迭代器范围删除
    string s4 = "Hello, World!";
    s4.erase(s4.begin() + 5, s4.end());  // 删除位置 5 到末尾
    cout << s4 << endl;  // Hello
}
```

### 7.3 补充：`push_back()` 和 `pop_back()`

```cpp
void test17() {
    string s = "Hell";

    // push_back：在末尾追加一个字符
    s.push_back('o');
    cout << s << endl;  // Hello

    // pop_back：删除末尾一个字符
    s.pop_back();
    cout << s << endl;  // Hell
}
```

---

## 八、string 容器 — 子串获取

`substr()` 用于从字符串中**截取**一段子串，返回一个**新的 `string` 对象**。

### 8.1 基本用法

```cpp
void test18() {
    string s = "Hello, World!";

    // substr(pos, len) — 从位置 pos 开始截取 len 个字符
    string sub1 = s.substr(0, 5);
    cout << sub1 << endl;  // Hello

    string sub2 = s.substr(7, 5);
    cout << sub2 << endl;  // World

    // 如果不传 len，或 len 超出范围，就截取到字符串末尾
    string sub3 = s.substr(7);
    cout << sub3 << endl;  // World!
}
```

### 8.2 实战：解析邮箱地址

这是 `substr` 最经典的应用场景之一 —— 配合 `find` 来分割字符串：

```cpp
void test19() {
    string email = "zhangsan@gmail.com";

    // 第一步：找到 '@' 的位置
    size_t atPos = email.find('@');

    if (atPos != string::npos) {
        // 第二步：截取用户名（从 0 到 @ 之前）
        string username = email.substr(0, atPos);
        cout << "用户名: " << username << endl;  // zhangsan

        // 第三步：截取域名（从 @ 之后到末尾）
        string domain = email.substr(atPos + 1);
        cout << "域名: " << domain << endl;  // gmail.com
    }
}
```

### 8.3 实战：按分隔符拆分字符串

标准库没有内置 `split` 函数，但用 `find` + `substr` 可以轻松实现：

```cpp
#include <vector>

void test20() {
    string data = "apple,banana,cherry,date";
    string delimiter = ",";
    vector<string> tokens;

    size_t start = 0;
    size_t end = 0;

    while ((end = data.find(delimiter, start)) != string::npos) {
        tokens.push_back(data.substr(start, end - start));
        start = end + delimiter.size();
    }
    // 别忘了最后一个元素（后面没有分隔符了）
    tokens.push_back(data.substr(start));

    // 输出结果
    for (const auto& token : tokens) {
        cout << "[" << token << "]" << endl;
    }
    // [apple]
    // [banana]
    // [cherry]
    // [date]
}
```

---

## 📝 知识总结速查表

| 操作 | 常用方法                         | 说明                     |
| ---- | -------------------------------- | ------------------------ |
| 构造 | string s("abc") / string s(n, c) | 多种方式创建字符串       |
| 赋值 | s = "abc" / s.assign(...)        | = 最常用                 |
| 拼接 | s += "abc" / s.append(...)       | += 最常用                |
| 查找 | s.find("abc") / s.rfind("abc")   | 找不到返回 string::npos  |
| 替换 | s.replace(pos, n, "new")         | 替换 pos 开始的 n 个字符 |
| 比较 | s1 == s2 / s1.compare(s2)        | == 最直观                |
| 存取 | s[i] / s.at(i)                   | at() 带越界检查          |
| 插入 | s.insert(pos, "abc")             | 在 pos 前插入            |
| 删除 | s.erase(pos, n) / s.pop_back()   | 删除指定范围或末尾       |
| 子串 | s.substr(pos, len)               | 返回新的 string          |

---

## 🧩 额外补充：常用辅助函数

```cpp
void test_extras() {
    string s = "Hello, World!";

    // size() / length()：获取字符串长度（两者完全等价）
    cout << "长度: " << s.size() << endl;    // 13
    cout << "长度: " << s.length() << endl;  // 13

    // empty()：判断是否为空
    cout << "是否为空: " << s.empty() << endl;  // 0 (false)

    // clear()：清空字符串
    s.clear();
    cout << "清空后: \"" << s << "\", 是否为空: " << s.empty() << endl;  // "", 1

    // resize()：重新设置大小
    string s2 = "Hello";
    s2.resize(10, '!');  // 扩大到 10，用 '!' 填充
    cout << s2 << endl;  // Hello!!!!!

    s2.resize(3);  // 缩小到 3，多余的直接截断
    cout << s2 << endl;  // Hel
}
```

---

## 🔥 综合实战练习

把今天学到的内容串联起来：

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

int main() {
    // 1. 构造
    string greeting("Welcome to C++ STL!");

    // 2. 查找 + 子串
    size_t pos = greeting.find("C++");
    string keyword = greeting.substr(pos, 3);
    cout << "找到关键字: " << keyword << endl;  // C++

    // 3. 替换
    greeting.replace(pos, 3, "Modern C++17");
    cout << "替换后: " << greeting << endl;  // Welcome to Modern C++17 STL!

    // 4. 拼接
    greeting += " Let's go!";
    cout << "拼接后: " << greeting << endl;

    // 5. 字符存取与修改
    greeting.at(0) = 'w';  // 改成小写
    cout << "修改后: " << greeting << endl;

    // 6. 插入
    greeting.insert(0, ">>> ");
    cout << "插入后: " << greeting << endl;

    // 7. 删除末尾多余内容
    size_t excl = greeting.rfind("!");
    if (excl != string::npos) {
        greeting.erase(excl + 1);  // 只保留到第一个 ! 为止
    }
    cout << "最终: " << greeting << endl;

    // 8. 比较
    string a = "abc", b = "abd";
    cout << (a < b ? "abc < abd" : "abc >= abd") << endl;  // abc < abd

    return 0;
}
```

---

> 补充

![image-20260223225205383](C:\Users\datovm\AppData\Roaming\Typora\typora-user-images\image-20260223225205383.png)