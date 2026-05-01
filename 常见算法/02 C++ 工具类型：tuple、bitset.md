# 第十五阶段：实用工具补充 —— `std::tuple` 与 `std::bitset`

> 所有容器学完了，现在补上两个在日常编程中极其常用的工具类型。它们不是容器，但经常和容器配合使用，掌握它们能让你的代码更加简洁高效。

---

# 一、`std::tuple` —— 万能的多元组

## 1.1 为什么需要 `tuple`？

在学习 STL 的过程中，你一定用过 `std::pair`。`pair` 可以把**两个**值打包在一起：

```cpp
std::pair<int, std::string> p = {1, "hello"};
std::cout << p.first << ", " << p.second << std::endl;
```

但如果你想打包**三个、四个甚至更多**值呢？`pair` 就力不从心了。

```cpp
// 如果你想存储一个学生的：学号、姓名、成绩
// 用 pair？ 不够用！
// 用 struct？ 可以，但有时候只是临时用一下，写一个 struct 太重了。
```

**`std::tuple`（元组）** 就是 `pair` 的升级版——它可以打包**任意数量、任意类型**的值。

> 📌 **头文件**：`#include <tuple>`

---

## 1.2 构造 `tuple`

### 方式一：直接构造

```cpp
#include <iostream>
#include <tuple>
#include <string>

int main() {
    // 显式指定类型
    std::tuple<int, std::string, double> student(1001, "张三", 95.5);

    // 也可以用花括号初始化（C++11）
    std::tuple<int, std::string, double> student2 = {1002, "李四", 88.0};

    return 0;
}
```

类型写在 `<>` 中，值写在 `()` 或 `{}` 中，顺序一一对应。

### 方式二：`std::make_tuple()` — 自动推导类型（推荐）

每次都要写完整的类型很麻烦。`make_tuple()` 可以**自动推导**模板参数：

```cpp
#include <iostream>
#include <tuple>
#include <string>

int main() {
    // 编译器自动推导为 tuple<int, string, double>
    auto student = std::make_tuple(1001, std::string("张三"), 95.5);

    // 注意：字符串字面量 "张三" 的类型是 const char*，不是 string
    // 如果你想让它推导为 string，需要显式写 std::string("张三")
    // 或者直接用 using namespace std::string_literals;
    // 然后写 "张三"s  （C++14 的字符串字面量后缀）

    return 0;
}
```

> ⚠️ **小陷阱**：`make_tuple("hello")` 推导出的是 `tuple<const char*>`，而不是 `tuple<string>`。如果你后续要和 `string` 做比较或赋值，可能会有问题。建议需要 `string` 时显式构造。

### 方式三：C++17 的类模板参数推导（CTAD）

C++17 起，你甚至不需要 `make_tuple()`，直接让编译器推导：

```cpp
// C++17 CTAD（Class Template Argument Deduction）
std::tuple student(1001, std::string("张三"), 95.5);
// 等价于 std::tuple<int, std::string, double>
```

这是最简洁的写法，但要求编译器支持 C++17。

---

## 1.3 访问元素：`std::get<>()`

`pair` 用 `.first` 和 `.second` 访问元素。`tuple` 有任意多个元素，不可能起那么多名字，所以用**下标**访问：

```cpp
#include <iostream>
#include <tuple>
#include <string>

int main() {
    auto student = std::make_tuple(1001, std::string("张三"), 95.5);

    // 用 std::get<下标>() 访问，下标从 0 开始
    std::cout << "学号: " << std::get<0>(student) << std::endl;  // 1001
    std::cout << "姓名: " << std::get<1>(student) << std::endl;  // 张三
    std::cout << "成绩: " << std::get<2>(student) << std::endl;  // 95.5

    // get 返回的是引用，可以直接修改
    std::get<2>(student) = 99.0;
    std::cout << "修改后成绩: " << std::get<2>(student) << std::endl;  // 99.0

    return 0;
}
```

### 关键要点

| 特性                     | 说明                                                |
| ------------------------ | --------------------------------------------------- |
| **下标从 0 开始**        | 和数组一样                                          |
| **下标必须是编译期常量** | 不能用变量，比如 `int i = 0; std::get<i>(t)` ❌      |
| **返回引用**             | 可以读，也可以写                                    |
| **越界编译报错**         | `std::get<3>(student)` 直接编译失败，不是运行时错误 |

```cpp
// ❌ 错误示范：下标不能是变量
int index = 1;
// std::get<index>(student);  // 编译错误！模板参数必须是编译期常量

// ✅ 正确：只能用字面量或 constexpr
constexpr int idx = 1;
std::get<idx>(student);  // OK
```

### 按类型访问（C++14）

如果 `tuple` 中**每个元素类型都不同**，你还可以用类型来访问：

```cpp
auto student = std::make_tuple(1001, std::string("张三"), 95.5);

// 按类型访问（要求该类型在 tuple 中唯一）
std::cout << std::get<int>(student) << std::endl;         // 1001
std::cout << std::get<std::string>(student) << std::endl;  // 张三
std::cout << std::get<double>(student) << std::endl;       // 95.5
```

> ⚠️ 如果有两个相同类型的元素，按类型访问会**编译报错**（歧义）。

```cpp
auto t = std::make_tuple(1, 2, 3.14);
// std::get<int>(t);  // ❌ 编译错误：有两个 int，不知道取哪个
```

---

## 1.4 解包：`std::tie()`

每次都 `std::get<0>()`, `std::get<1>()` 太啰嗦了。`std::tie()` 可以把 `tuple` 的值**一次性解包到多个独立变量**中：

```cpp
#include <iostream>
#include <tuple>
#include <string>

int main() {
    auto student = std::make_tuple(1001, std::string("张三"), 95.5);

    // ===== 使用 tie 解包 =====
    int id;
    std::string name;
    double score;

    std::tie(id, name, score) = student;

    std::cout << "学号: " << id << std::endl;    // 1001
    std::cout << "姓名: " << name << std::endl;  // 张三
    std::cout << "成绩: " << score << std::endl;  // 95.5

    return 0;
}
```

### `tie` 的工作原理

`std::tie(id, name, score)` 返回一个 `tuple<int&, string&, double&>`——注意是**引用**的 tuple。然后赋值运算符把右边 `student` 的每个元素分别赋值给对应的引用。

**本质上就是**：
```
id    = std::get<0>(student);
name  = std::get<1>(student);
score = std::get<2>(student);
```

### 忽略某些元素：`std::ignore`

如果你只关心部分元素，不想要的位置用 `std::ignore` 占位：

```cpp
int id;
double score;

// 只取学号和成绩，忽略姓名
std::tie(id, std::ignore, score) = student;

std::cout << "学号: " << id << std::endl;     // 1001
std::cout << "成绩: " << score << std::endl;  // 95.5
```

### `tie` 的经典用法：简化多字段比较

在写自定义排序或比较时，`tie` 非常好用：

```cpp
#include <iostream>
#include <tuple>
#include <string>
#include <vector>
#include <algorithm>

struct Student {
    std::string name;
    int age;
    double score;
};

int main() {
    std::vector<Student> students = {
        {"张三", 20, 90.5},
        {"李四", 19, 95.0},
        {"王五", 20, 90.5},
        {"赵六", 19, 88.0}
    };

    // 按 score 降序 → age 升序 → name 升序 排序
    std::sort(students.begin(), students.end(),
        [](const Student& a, const Student& b) {
            // 混合升/降序时，直接按字段写更清晰，也不容易写错
            if (a.score != b.score) return a.score > b.score;  // score 降序
            if (a.age != b.age) return a.age < b.age;          // age 升序
            return a.name < b.name;                             // name 升序
        }
    );

    for (const auto& s : students) {
        std::cout << s.name << " " << s.age << " " << s.score << std::endl;
    }

    return 0;
}
```

> 💡 `tuple` 的比较是**字典序**的：先比第一个元素，相等再比第二个，以此类推。这和 `pair` 的比较规则一模一样，只是扩展到了任意多个元素。

---

## 1.5 结构化绑定（C++17）—— 最优雅的解包方式

C++17 引入的**结构化绑定（Structured Bindings）** 是解包 `tuple` 的终极方案，也是目前最推荐的写法：

```cpp
#include <iostream>
#include <tuple>
#include <string>

int main() {
    auto student = std::make_tuple(1001, std::string("张三"), 95.5);

    // ===== C++17 结构化绑定 =====
    auto [id, name, score] = student;
    // 等价于：
    // auto id    = std::get<0>(student);   (拷贝)
    // auto name  = std::get<1>(student);   (拷贝)
    // auto score = std::get<2>(student);   (拷贝)

    std::cout << "学号: " << id << std::endl;     // 1001
    std::cout << "姓名: " << name << std::endl;   // 张三
    std::cout << "成绩: " << score << std::endl;   // 95.5

    return 0;
}
```

### 三种绑定方式

```cpp
auto student = std::make_tuple(1001, std::string("张三"), 95.5);

// 1. 拷贝绑定 —— 修改 id/name/score 不影响 student
auto [id, name, score] = student;

// 2. 引用绑定 —— 修改 id/name/score 会同步修改 student
auto& [id2, name2, score2] = student;
score2 = 100.0;  // student 中的成绩也变成了 100.0

// 3. const 引用绑定 —— 只读，不允许修改
const auto& [id3, name3, score3] = student;
// score3 = 50.0;  // ❌ 编译错误
```

### 结构化绑定不仅限于 `tuple`！

它还能用于 `pair`、`struct`、`array`，超级方便：

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> scores = {
        {"张三", 90},
        {"李四", 85}
    };

    // 遍历 map 时，每个元素是 pair<const string, int>
    // 传统写法：
    for (const auto& p : scores) {
        std::cout << p.first << ": " << p.second << std::endl;
    }

    // C++17 结构化绑定写法（更直观！）：
    for (const auto& [name, score] : scores) {
        std::cout << name << ": " << score << std::endl;
    }

    return 0;
}
```

> 你之前在学 `map` 时可能已经见过这种写法了。现在你知道它的本质了——这就是**结构化绑定**在 `pair` 上的应用。

---

## 1.6 `tuple` 的其他实用操作

### `std::tuple_size` — 获取元素个数

```cpp
using MyTuple = std::tuple<int, std::string, double>;

// 编译期获取 tuple 的元素个数
constexpr std::size_t n = std::tuple_size<MyTuple>::value;  // 3

// C++17 简写
constexpr std::size_t n2 = std::tuple_size_v<MyTuple>;     // 3

std::cout << "元素个数: " << n << std::endl;
```

### `std::tuple_element` — 获取某个位置的类型

```cpp
using MyTuple = std::tuple<int, std::string, double>;

// 获取第 1 个元素的类型（从 0 开始）
using SecondType = std::tuple_element<1, MyTuple>::type;  // std::string

// C++14 简写
using SecondType2 = std::tuple_element_t<1, MyTuple>;     // std::string
```

> 这两个在日常编码中用得不多，但在写模板代码时非常有用。了解即可。

### `std::tuple_cat()` — 拼接多个 `tuple`

```cpp
auto t1 = std::make_tuple(1, 2);
auto t2 = std::make_tuple(std::string("hello"), 3.14);
auto t3 = std::make_tuple('A');

// 拼接成一个大 tuple
auto combined = std::tuple_cat(t1, t2, t3);
// combined 的类型: tuple<int, int, string, double, char>
// 值: (1, 2, "hello", 3.14, 'A')

std::cout << std::get<2>(combined) << std::endl;  // hello
std::cout << std::get<4>(combined) << std::endl;  // A
```

---

## 1.7 `tuple` 的实战场景

### 场景一：函数返回多个值

```cpp
#include <iostream>
#include <tuple>
#include <string>

// 返回多个值：错误码、错误消息、结果数据
std::tuple<int, std::string, double> divide(double a, double b) {
    if (b == 0.0) {
        return {-1, "除数不能为零", 0.0};
    }
    return {0, "成功", a / b};
}

int main() {
    auto [code, msg, result] = divide(10.0, 3.0);

    if (code == 0) {
        std::cout << msg << ": " << result << std::endl;  // 成功: 3.33333
    } else {
        std::cout << "错误: " << msg << std::endl;
    }

    auto [code2, msg2, result2] = divide(10.0, 0.0);
    if (code2 != 0) {
        std::cout << "错误: " << msg2 << std::endl;  // 错误: 除数不能为零
    }

    return 0;
}
```

### 场景二：作为 `map` 的值

```cpp
#include <iostream>
#include <map>
#include <tuple>
#include <string>

int main() {
    // 存储学生信息：学号 → (姓名, 年龄, 成绩)
    std::map<int, std::tuple<std::string, int, double>> students;

    students[1001] = {"张三", 20, 95.5};
    students[1002] = {"李四", 19, 88.0};

    for (const auto& [id, info] : students) {
        const auto& [name, age, score] = info;
        std::cout << id << " - " << name
                  << ", 年龄: " << age
                  << ", 成绩: " << score << std::endl;
    }

    return 0;
}
```

---

## 1.8 `tuple` vs `pair` vs `struct` 总结

| 特性     | `pair`              | `tuple`            | `struct`             |
| -------- | ------------------- | ------------------ | -------------------- |
| 元素数量 | 固定 2 个           | 任意个             | 任意个               |
| 访问方式 | `.first`, `.second` | `std::get<N>()`    | `obj.成员名`         |
| 可读性   | 一般                | 较差（下标无语义） | **最好**（有名字）   |
| 定义成本 | 低                  | 低                 | 较高（需要写定义）   |
| 适用场景 | 临时打包 2 个值     | 临时打包多个值     | 需要复用、语义清晰时 |

> 💡 **选择建议**：
> - 临时返回 2 个值 → `pair`
> - 临时返回 3 个及以上值 → `tuple`
> - 在多处使用、需要清晰语义 → 定义 `struct`

---
---

# 二、`std::bitset` —— 固定大小的位集合

## 2.1 为什么需要 `bitset`？

在编程中，我们经常需要处理**位（bit）级别**的操作：

- 标记一组开关的开/关状态（比如 32 个灯泡，哪些亮了？）
- 权限管理（读/写/执行权限的组合）
- 算法竞赛中的状态压缩
- 高效存储大量布尔值

你可以用 `int` 或 `long long` 做位运算（`& | ^ ~ << >>`），但有几个痛点：

1. **位数有限**：`int` 最多 32 位，`long long` 最多 64 位
2. **操作不直观**：`flags & (1 << 3)` 这种写法可读性差
3. **没有方便的工具函数**：要计数有多少个 1？要全部翻转？自己写

`std::bitset` 就是为了解决这些问题而生的——它是**固定大小的位集合**，提供了直观的操作接口。

> 📌 **头文件**：`#include <bitset>`

---

## 2.2 构造 `bitset`

`bitset` 的大小是**编译期常量**（写在模板参数里），构造方式有以下几种：

### 方式一：默认构造（全 0）

```cpp
#include <iostream>
#include <bitset>

int main() {
    std::bitset<8> b;  // 8 位，全部为 0
    std::cout << b << std::endl;  // 输出: 00000000

    return 0;
}
```

> 📌 `bitset` 可以直接用 `<<` 输出，非常方便！输出时**高位在左，低位在右**（和我们写二进制数的习惯一样）。

### 方式二：用整数构造

```cpp
#include <iostream>
#include <bitset>

int main() {
    std::bitset<8> b1(42);
    // 42 的二进制是 00101010
    std::cout << b1 << std::endl;  // 输出: 00101010

    std::bitset<8> b2(0b11001100);  // C++14 二进制字面量
    std::cout << b2 << std::endl;   // 输出: 11001100

    std::bitset<4> b3(0xFF);  // 0xFF = 11111111，但只有 4 位
    std::cout << b3 << std::endl;   // 输出: 1111（高位被截断）

    std::bitset<16> b4(42);  // 16 位，42 = 0000000000101010
    std::cout << b4 << std::endl;   // 输出: 0000000000101010（高位补 0）

    return 0;
}
```

**规则**：
- 如果整数的二进制位数 **>** `bitset` 大小 → 高位被截断
- 如果整数的二进制位数 **<** `bitset` 大小 → 高位补 0

### 方式三：用字符串构造

```cpp
#include <iostream>
#include <bitset>
#include <string>

int main() {
    // 用 std::string 构造
    std::bitset<8> b1(std::string("11001010"));
    std::cout << b1 << std::endl;   // 输出: 11001010

    // 用 C 风格字符串构造
    std::bitset<8> b2("11001010");
    std::cout << b2 << std::endl;   // 输出: 11001010

    // 字符串比 bitset 短 → 右边对齐（低位对齐），高位补 0
    std::bitset<8> b3("1010");
    std::cout << b3 << std::endl;   // 输出: 00001010
    // 等价于：字符串 "1010" 表示的二进制数，放入 8 位中

    return 0;
}
```

> ⚠️ **注意**：字符串中只能包含 `'0'` 和 `'1'`，否则会抛出 `std::invalid_argument` 异常。

### 方式四：字符串部分构造（高级）

```cpp
#include <iostream>
#include <bitset>
#include <string>

int main() {
    std::string s = "1111000010101010";

    // 从位置 4 开始，取 8 个字符
    std::bitset<8> b(s, 4, 8);
    // 取到的是 "00001010"
    std::cout << b << std::endl;   // 输出: 00001010

    // 从位置 0 开始，取 4 个字符
    std::bitset<8> b2(s, 0, 4);
    // 取到的是 "1111"
    std::cout << b2 << std::endl;  // 输出: 00001111

    return 0;
}
```

`bitset(string, pos, len)` 的参数含义：
- `pos`：从字符串的第 `pos` 个字符开始
- `len`：取 `len` 个字符

---

## 2.3 访问和修改单个位

### `operator[]` — 下标访问

```cpp
#include <iostream>
#include <bitset>

int main() {
    std::bitset<8> b("11001010");
    //                76543210  ← 注意：下标 0 是最右边的位（最低位）！

    std::cout << "第 0 位: " << b[0] << std::endl;  // 0（最右边）
    std::cout << "第 1 位: " << b[1] << std::endl;  // 1
    std::cout << "第 7 位: " << b[7] << std::endl;  // 1（最左边）

    // 修改单个位
    b[0] = 1;
    std::cout << b << std::endl;  // 11001011

    return 0;
}
```

> ⚠️ **重要**：下标 0 对应**最低位（最右边）**，不是最高位！这和输出时的视觉顺序**相反**。
>
> ```
> 输出：   1 1 0 0 1 0 1 0
> 下标：   7 6 5 4 3 2 1 0
> ```

### `test()` — 安全的访问（带边界检查）

```cpp
std::bitset<8> b("11001010");

std::cout << b.test(1) << std::endl;  // true (1)
std::cout << b.test(0) << std::endl;  // false (0)

// b.test(8);  // ❌ 运行时抛出 std::out_of_range 异常！
// b[8];       // ⚠️ 未定义行为！不会抛异常，但结果不可预测
```

| 访问方式    | 越界时的行为                    |
| ----------- | ------------------------------- |
| `b[i]`      | ❌ 未定义行为（不检查边界）      |
| `b.test(i)` | ✅ 抛出 `std::out_of_range` 异常 |

---

## 2.4 批量修改操作

### `set()` — 置为 1

```cpp
#include <iostream>
#include <bitset>

int main() {
    std::bitset<8> b;  // 00000000

    b.set(0);     // 将第 0 位设为 1 → 00000001
    b.set(3);     // 将第 3 位设为 1 → 00001001
    b.set(7);     // 将第 7 位设为 1 → 10001001
    std::cout << b << std::endl;  // 10001001

    b.set(0, false);  // set 也可以设为 0！第二个参数控制值
    std::cout << b << std::endl;  // 10001000

    b.set();  // 不传参数 → 全部置为 1
    std::cout << b << std::endl;  // 11111111

    return 0;
}
```

### `reset()` — 置为 0

```cpp
std::bitset<8> b("11111111");

b.reset(0);    // 将第 0 位设为 0 → 11111110
b.reset(7);    // 将第 7 位设为 0 → 01111110
std::cout << b << std::endl;  // 01111110

b.reset();     // 不传参数 → 全部置为 0
std::cout << b << std::endl;  // 00000000
```

### `flip()` — 翻转（0 变 1，1 变 0）

```cpp
std::bitset<8> b("11001010");

b.flip(0);     // 翻转第 0 位 → 11001011
b.flip(7);     // 翻转第 7 位 → 01001011
std::cout << b << std::endl;  // 01001011

b.flip();      // 不传参数 → 全部翻转
std::cout << b << std::endl;  // 10110100
```

### 汇总表

| 方法                     | 单个位（传下标）        | 全部位（不传参数） |
| ------------------------ | ----------------------- | ------------------ |
| `set(pos)` / `set()`     | 将第 `pos` 位设为 1     | 全部设为 1         |
| `set(pos, val)`          | 将第 `pos` 位设为 `val` | —                  |
| `reset(pos)` / `reset()` | 将第 `pos` 位设为 0     | 全部设为 0         |
| `flip(pos)` / `flip()`   | 翻转第 `pos` 位         | 全部翻转           |

---

## 2.5 查询操作

### `count()` — 统计 1 的个数

```cpp
std::bitset<8> b("11001010");
std::cout << b.count() << std::endl;  // 4（有 4 个 1）
```

> 等价于手动位运算中的 `__builtin_popcount()`，但 `bitset` 的 `count()` 支持任意位数！

### `size()` — 获取总位数

```cpp
std::bitset<8> b;
std::cout << b.size() << std::endl;  // 8（就是模板参数）
```

### `any()` — 是否存在至少一个 1？

```cpp
std::bitset<8> b1("00000000");
std::bitset<8> b2("00010000");

std::cout << b1.any() << std::endl;  // false（全是 0）
std::cout << b2.any() << std::endl;  // true（至少有一个 1）
```

### `none()` — 是否全部为 0？

```cpp
std::bitset<8> b1("00000000");
std::bitset<8> b2("00010000");

std::cout << b1.none() << std::endl;  // true（全是 0）
std::cout << b2.none() << std::endl;  // false
```

### `all()` — 是否全部为 1？（C++11）

```cpp
std::bitset<8> b1("11111111");
std::bitset<8> b2("11110000");

std::cout << b1.all() << std::endl;  // true（全是 1）
std::cout << b2.all() << std::endl;  // false
```

### 三者关系

```
any()  == !none()     // 有 1 ↔ 不全是 0
none() == !any()      // 全是 0 ↔ 没有 1
all()  == (count() == size())  // 全是 1 ↔ 1 的个数等于总位数
```

---

## 2.6 位运算操作

`bitset` 支持所有常见的位运算符，用法和整数的位运算**完全一致**，但对于超过 64 位的 `bitset` 也能正常工作：

### 基本位运算

```cpp
#include <iostream>
#include <bitset>

int main() {
    std::bitset<8> a("11001010");
    std::bitset<8> b("10101100");

    // 按位与 AND
    std::cout << "a & b = " << (a & b) << std::endl;   // 10001000

    // 按位或 OR
    std::cout << "a | b = " << (a | b) << std::endl;   // 11101110

    // 按位异或 XOR
    std::cout << "a ^ b = " << (a ^ b) << std::endl;   // 01100110

    // 按位取反 NOT
    std::cout << "~a    = " << (~a) << std::endl;       // 00110101

    return 0;
}
```

### 复合赋值运算

```cpp
std::bitset<8> a("11001010");
std::bitset<8> b("10101100");

a &= b;    // a = a & b
a |= b;    // a = a | b
a ^= b;    // a = a ^ b
```

### 移位运算

```cpp
std::bitset<8> a("00110100");

std::cout << (a << 2) << std::endl;  // 11010000（左移 2 位，右边补 0）
std::cout << (a >> 3) << std::endl;  // 00000110（右移 3 位，左边补 0）

a <<= 1;   // a = a << 1
std::cout << a << std::endl;         // 01101000
```

> 💡 `bitset` 的移位总是**逻辑移位**（补 0），没有算术移位的符号位问题。

---

## 2.7 类型转换

### `to_ulong()` / `to_ullong()` — 转换为整数

```cpp
#include <iostream>
#include <bitset>

int main() {
    std::bitset<8> b("11001010");

    unsigned long val = b.to_ulong();
    std::cout << val << std::endl;  // 202

    unsigned long long val2 = b.to_ullong();  // C++11
    std::cout << val2 << std::endl;  // 202

    return 0;
}
```

> ⚠️ 如果 `bitset` 的值超出了 `unsigned long` / `unsigned long long` 的范围，会抛出 `std::overflow_error` 异常。

### `to_string()` — 转换为字符串

```cpp
std::bitset<8> b("11001010");

std::string s = b.to_string();
std::cout << s << std::endl;  // "11001010"

// 可以自定义 0 和 1 的字符
std::string s2 = b.to_string('O', 'X');
std::cout << s2 << std::endl;  // "XXOOXOXO"
```

---

## 2.8 `bitset` 的相等比较

```cpp
std::bitset<8> a("11001010");
std::bitset<8> b("11001010");
std::bitset<8> c("00001010");

std::cout << (a == b) << std::endl;  // true
std::cout << (a != c) << std::endl;  // true
```

> 📌 只支持 `==` 和 `!=`，**不支持** `<`, `>`, `<=`, `>=`。如果需要比较大小，先转换为整数。

---

## 2.9 实战示例

### 示例一：用 `bitset` 实现权限管理

```cpp
#include <iostream>
#include <bitset>
#include <string>

// 定义权限位
enum Permission {
    READ    = 0,   // 第 0 位：读权限
    WRITE   = 1,   // 第 1 位：写权限
    EXECUTE = 2,   // 第 2 位：执行权限
    DELETE  = 3    // 第 3 位：删除权限
};

int main() {
    std::bitset<4> adminPerm;    // 管理员权限
    std::bitset<4> userPerm;     // 普通用户权限

    // 设置管理员：拥有全部权限
    adminPerm.set();  // 1111

    // 设置普通用户：只有读和执行权限
    userPerm.set(READ);
    userPerm.set(EXECUTE);
    // userPerm = 0101

    std::cout << "管理员权限: " << adminPerm << std::endl;  // 1111
    std::cout << "用户权限:   " << userPerm << std::endl;    // 0101

    // 检查权限
    if (userPerm.test(WRITE)) {
        std::cout << "用户有写权限" << std::endl;
    } else {
        std::cout << "用户没有写权限" << std::endl;  // ← 输出这个
    }

    // 给用户添加写权限
    userPerm.set(WRITE);
    std::cout << "添加写权限后: " << userPerm << std::endl;  // 0111

    // 撤销用户的执行权限
    userPerm.reset(EXECUTE);
    std::cout << "撤销执行权限后: " << userPerm << std::endl;  // 0011

    // 查看用户有几个权限
    std::cout << "用户拥有 " << userPerm.count() << " 个权限" << std::endl;  // 2

    return 0;
}
```

### 示例二：埃拉托斯特尼筛法（素数筛）

`bitset` 在素数筛中特别好用，因为它按位存储布尔状态，通常比 `bool[]` 更省内存；与 `vector<bool>` 都属于位压缩思路：

```cpp
#include <iostream>
#include <bitset>

int main() {
    constexpr int N = 100;
    std::bitset<N + 1> is_prime;

    is_prime.set();            // 先假设全部是素数（置为 1）
    is_prime.reset(0);         // 0 不是素数
    is_prime.reset(1);         // 1 不是素数

    for (int i = 2; i * i <= N; ++i) {
        if (is_prime.test(i)) {
            // i 是素数，将 i 的所有倍数标记为非素数
            for (int j = i * i; j <= N; j += i) {
                is_prime.reset(j);
            }
        }
    }

    // 输出所有素数
    std::cout << N << " 以内的素数：" << std::endl;
    for (int i = 2; i <= N; ++i) {
        if (is_prime.test(i)) {
            std::cout << i << " ";
        }
    }
    std::cout << std::endl;

    // 统计素数个数
    std::cout << "共 " << is_prime.count() << " 个素数" << std::endl;

    return 0;
}
```

### 示例三：状态压缩（算法竞赛常见）

用 `bitset` 表示集合，进行子集枚举等操作：

```cpp
#include <iostream>
#include <bitset>
#include <vector>
#include <string>

int main() {
    // 假设有 4 个物品
    std::vector<std::string> items = {"苹果", "香蕉", "橙子", "葡萄"};
    int n = items.size();

    // 枚举所有子集（2^4 = 16 种）
    std::cout << "所有子集：" << std::endl;
    for (int mask = 0; mask < (1 << n); ++mask) {
        std::bitset<4> selected(mask);
        std::cout << selected << " → { ";

        for (int i = 0; i < n; ++i) {
            if (selected.test(i)) {
                std::cout << items[i] << " ";
            }
        }
        std::cout << "}" << std::endl;
    }

    return 0;
}
```

输出：
```
所有子集：
0000 → { }
0001 → { 苹果 }
0010 → { 香蕉 }
0011 → { 苹果 香蕉 }
0100 → { 橙子 }
0101 → { 苹果 橙子 }
0110 → { 香蕉 橙子 }
0111 → { 苹果 香蕉 橙子 }
1000 → { 葡萄 }
1001 → { 苹果 葡萄 }
1010 → { 香蕉 葡萄 }
1011 → { 苹果 香蕉 葡萄 }
1100 → { 橙子 葡萄 }
1101 → { 苹果 橙子 葡萄 }
1110 → { 香蕉 橙子 葡萄 }
1111 → { 苹果 香蕉 橙子 葡萄 }
```

---

## 2.10 `bitset` 的局限性

| 局限                   | 说明                                                 |
| ---------------------- | ---------------------------------------------------- |
| **大小必须编译期确定** | `std::bitset<n>` 的 `n` 必须是常量，不能是运行时变量 |
| **不能动态改变大小**   | 创建后位数固定，不能 push/pop                        |
| **不支持大小比较**     | 只有 `==` 和 `!=`，没有 `<` `>`                      |
| **不支持迭代器**       | 不能用范围 for 遍历，只能用下标                      |

> 💡 如果需要**运行时决定大小**的位集合，可以考虑 `std::vector<bool>`（虽然它也有自己的问题）或者 Boost 库的 `dynamic_bitset`。

---

## 2.11 `bitset` 完整 API 速查表

| 操作       | 方法                       | 说明                             |
| ---------- | -------------------------- | -------------------------------- |
| **构造**   | `bitset<N>()`              | 全 0                             |
|            | `bitset<N>(unsigned long)` | 从整数构造                       |
|            | `bitset<N>(string)`        | 从 "01" 字符串构造               |
| **访问**   | `b[i]`                     | 访问第 i 位（不检查边界）        |
|            | `b.test(i)`                | 访问第 i 位（检查边界）          |
| **修改**   | `b.set()` / `b.set(i)`     | 全部/指定位 置 1                 |
|            | `b.reset()` / `b.reset(i)` | 全部/指定位 置 0                 |
|            | `b.flip()` / `b.flip(i)`   | 全部/指定位 翻转                 |
| **查询**   | `b.count()`                | 1 的个数                         |
|            | `b.size()`                 | 总位数                           |
|            | `b.any()`                  | 是否至少有一个 1                 |
|            | `b.none()`                 | 是否全部为 0                     |
|            | `b.all()`                  | 是否全部为 1（C++11）            |
| **位运算** | `& \| ^ ~ << >>`           | 与、或、异或、取反、左移、右移   |
|            | `&= \|= ^= <<= >>=`        | 对应的复合赋值                   |
| **转换**   | `b.to_ulong()`             | 转 `unsigned long`               |
|            | `b.to_ullong()`            | 转 `unsigned long long`（C++11） |
|            | `b.to_string()`            | 转字符串                         |
| **比较**   | `== !=`                    | 相等/不等比较                    |
| **I/O**    | `cin >> b` / `cout << b`   | 输入/输出                        |

---

# 三、阶段总结

恭喜你！到这里，STL 中最常用的容器和工具类型就全部学完了。来做个回顾：

| 工具          | 本质       | 核心特点                       | 典型用途                             |
| ------------- | ---------- | ------------------------------ | ------------------------------------ |
| `std::pair`   | 2 元组     | `.first`, `.second`            | map 的元素、临时返回 2 个值          |
| `std::tuple`  | N 元组     | `get<>()`, `tie()`, 结构化绑定 | 函数返回多个值、临时打包数据         |
| `std::bitset` | 固定位集合 | 位操作 + 查询接口              | 状态标记、权限管理、素数筛、状态压缩 |

这三个工具不是容器（不存储可变数量的元素），而是**实用类型（Utility Types）**。它们和容器一起使用，能让你的代码更简洁、更高效。

> 📌 **下一步学习建议**：
> - 多在实际练习中使用 `tuple` 的结构化绑定（遍历 map 时用 `auto& [key, value]`）
> - 用 `bitset` 重做一些之前用 `bool` 数组做的题目，感受节省空间的效果
> - 尝试在函数返回值中使用 `tuple` 代替输出参数