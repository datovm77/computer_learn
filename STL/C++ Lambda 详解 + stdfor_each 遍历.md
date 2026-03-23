# C++ Lambda 详解 + `std::for_each` 遍历

---

## 一、什么是 Lambda？

Lambda 是 C++11 引入的**匿名函数**（闭包），可以在需要函数的地方直接"内联"定义，无需单独命名。

Lambda 的完整语法：

```
[ 捕获列表 ] ( 参数列表 ) -> 返回类型 { 函数体 }
```

---

## 二、Lambda 语法逐部分拆解

### 1. 最简单的 Lambda

```cpp
auto greet = []() {
    std::cout << "Hello, Lambda!" << std::endl;
};
greet(); // 调用，输出: Hello, Lambda!
```

### 2. 带参数的 Lambda

```cpp
auto add = [](int a, int b) -> int {
    return a + b;
};

std::cout << add(3, 5) << std::endl; // 输出: 8
```

> **返回类型可以省略**，编译器会自动推导：
>
>
> ```cpp
> auto add = [](int a, int b) { return a + b; }; // 推导返回 int
> ```

---

## 三、捕获列表（最重要！）

捕获列表 `[]` 决定了 Lambda **能否访问外部变量**，以及**如何访问**。

| 捕获方式 | 含义                       |
| -------- | -------------------------- |
| []       | 不捕获任何外部变量         |
| [x]      | 值捕获 x（复制一份，只读） |
| [&x]     | 引用捕获 x（可修改原变量） |
| [=]      | 值捕获所有外部变量         |
| [&]      | 引用捕获所有外部变量       |
| [=, &x]  | 其余值捕获，x 引用捕获     |
| [this]   | 捕获当前类的 this 指针     |

### 值捕获 vs 引用捕获 示例

```cpp
#include <iostream>

int main() {
    int count = 0;

    // 值捕获：Lambda 内部是 count 的副本，修改不影响外部
    auto byValue = [count]() mutable {
        count++;                          // mutable 才能修改副本
        std::cout << "内部: " << count << std::endl;
    };

    // 引用捕获：直接修改外部 count
    auto byRef = [&count]() {
        count++;
        std::cout << "内部: " << count << std::endl;
    };

    byValue(); // 内部: 1
    std::cout << "外部 count: " << count << std::endl; // 外部 count: 0 ← 未改变！

    byRef();   // 内部: 1
    std::cout << "外部 count: " << count << std::endl; // 外部 count: 1 ← 已改变！

    return 0;
}
```

> ⚠️ **注意**：值捕获默认是 `const` 的，要在 Lambda 内修改副本必须加 `mutable` 关键字。

---

## 四、`std::for_each` + Lambda 遍历

`std::for_each` 定义在 `<algorithm>` 头文件中，语法如下：

```cpp
std::for_each(开始迭代器, 结束迭代器, 可调用对象);
```

### 基础示例：打印所有元素

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5};

    std::for_each(nums.begin(), nums.end(), [](int n) {
        std::cout << n << " ";
    });
    // 输出: 1 2 3 4 5

    return 0;
}
```

### 修改元素（引用参数）

```cpp
std::vector<int> nums = {1, 2, 3, 4, 5};

std::for_each(nums.begin(), nums.end(), [](int& n) {
    n *= 2; // 直接修改原元素
});

// nums 现在是: {2, 4, 6, 8, 10}
```

### 配合捕获列表：求总和

```cpp
std::vector<int> nums = {1, 2, 3, 4, 5};
int sum = 0;

std::for_each(nums.begin(), nums.end(), [&sum](int n) {
    sum += n; // 引用捕获 sum，累加
});

std::cout << "总和: " << sum << std::endl; // 总和: 15
```

### 复杂示例：统计 + 修改

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> nums = {3, 1, 4, 1, 5, 9, 2, 6};
    int evenCount = 0;
    int threshold = 4;

    std::for_each(nums.begin(), nums.end(), [&evenCount, threshold](int& n) {
        if (n % 2 == 0) evenCount++;   // 统计偶数个数
        if (n > threshold) n = threshold; // 超过阈值的截断
    });

    std::cout << "偶数个数: " << evenCount << std::endl; // 偶数个数: 3

    for (int n : nums) std::cout << n << " ";
    // 输出: 3 1 4 1 4 4 2 4

    return 0;
}
```

---

## 五、`std::for_each` vs Range-based for 对比

```cpp
std::vector<int> nums = {1, 2, 3};

// ✅ Range-based for（简洁，日常首选）
for (int n : nums) {
    std::cout << n << " ";
}

// ✅ std::for_each + lambda（更函数式，可复用 lambda）
auto printer = [](int n) { std::cout << n << " "; };
std::for_each(nums.begin(), nums.end(), printer);

// ✅ std::for_each 的优势：可以指定范围！
// 只遍历前3个元素：
std::for_each(nums.begin(), nums.begin() + 3, printer);
```

| 特性         | Range-based for | std::for_each + Lambda |
| ------------ | --------------- | ---------------------- |
| 代码简洁性   | ⭐⭐⭐             | ⭐⭐                     |
| 可复用性     | ❌               | ✅ Lambda 可复用        |
| 指定部分范围 | ❌               | ✅                      |
| 与算法组合   | ❌               | ✅                      |

---

## 六、Lambda 进阶：泛型 Lambda（C++14）

C++14 开始，参数类型可以用 `auto`，实现泛型：

```cpp
// 泛型 lambda，可以接受任意类型
auto printAny = [](auto val) {
    std::cout << val << " ";
};

std::vector<int>    ints   = {1, 2, 3};
std::vector<double> doubles = {1.1, 2.2, 3.3};
std::vector<std::string> strs = {"a", "b", "c"};

std::for_each(ints.begin(),    ints.end(),    printAny);
std::for_each(doubles.begin(), doubles.end(), printAny);
std::for_each(strs.begin(),    strs.end(),    printAny);
```

---

## 七、实战综合示例

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>

struct Student {
    std::string name;
    double score;
};

int main() {
    std::vector<Student> students = {
        {"Alice", 92.5},
        {"Bob",   78.0},
        {"Carol", 88.5},
        {"Dave",  55.0}
    };

    double passLine = 60.0;
    int passCount = 0;

    std::for_each(students.begin(), students.end(),
        [&passCount, passLine](const Student& s) {
            std::cout << s.name << ": " << s.score;
            if (s.score >= passLine) {
                std::cout << " ✅";
                passCount++;
            } else {
                std::cout << " ❌";
            }
            std::cout << std::endl;
        }
    );

    std::cout << "\n通过人数: " << passCount << "/" << students.size() << std::endl;

    return 0;
}
```

**输出：**

```
Alice: 92.5 ✅
Bob: 78 ✅
Carol: 88.5 ✅
Dave: 55 ❌

通过人数: 3/4
```

---

## 总结

```
Lambda = [ 捕获 ] ( 参数 ) -> 返回类型 { 函数体 }
```

| 要点          | 说明                              |
| ------------- | --------------------------------- |
| [=] / [&]     | 值/引用捕获所有外部变量           |
| mutable       | 允许修改值捕获的副本              |
| auto 参数     | C++14 泛型 Lambda                 |
| std::for_each | 比 range-for 更灵活，支持指定范围 |

Lambda 是现代 C++ 的核心特性，后续学 `std::sort`、`std::find_if`、`std::transform` 等算法都会大量用到它！
