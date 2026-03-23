# C++ 函数模板详细教程

## 📚 目录

1. 什么是函数模板

2. 为什么需要函数模板

3. 函数模板的基本语法

4. **底层原理：模板实例化机制**

5. 模板参数推导

6. 显式模板参数

7. 函数模板特化

8. 函数模板重载

9. 高级主题与注意事项

---

## 1️⃣ 什么是函数模板

**函数模板（Function Template）** 是C++中的一种泛型编程机制， 

简单来说：

- **普通函数**：针对特定类型编写的代码

- **函数模板**：针对任意类型编写的代码模板，编译器根据实际使用的类型自动生成对应的函数

---

## 2️⃣ 为什么需要函数模板

### 问题场景

假设我们要编写一个交换两个变量值的函数：

```cpp
// 交换两个整数
void swap_int(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

// 交换两个浮点数
void swap_double(double& a, double& b) {
    double temp = a;
    a = b;
    b = temp;
}

// 交换两个字符串
void swap_string(std::string& a, std::string& b) {
    std::string temp = a;
    a = b;
    b = temp;
}
```

**问题**：

- 代码高度重复，只是类型不同

- 每增加一种类型，就要写一个新函数

- 维护困难，修改逻辑需要改多处

### 解决方案：函数模板

```cpp
template<typename T>
void mySwap(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}
```

一个模板搞定所有类型！

---

## 3️⃣ 函数模板的基本语法

### 语法格式

```cpp
template<typename T>
返回类型 函数名(参数列表) {
    // 函数体
}
```

或者使用 `class` 关键字（效果相同）：

```cpp
template<class T>
返回类型 函数名(参数列表) {
    // 函数体
}
```

> **注意**：`typename` 和 `class` 在函数模板中完全等价，推荐使用 `typename`（语义更清晰）

### 基础示例

```cpp
#include <iostream>
using namespace std;

// 函数模板：返回两个值中的较大者
template<typename T>
T getMax(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    cout << getMax(10, 20) << endl;           // T = int
    cout << getMax(3.14, 2.72) << endl;       // T = double
    cout << getMax('a', 'z') << endl;         // T = char
    
    return 0;
}
```

**输出**：

```
20
3.14
z
```

---

## 4️⃣ 🔥 底层原理：模板实例化机制

这是理解函数模板的**核心**！

### 4.1 模板不是函数

**关键认知**：

```cpp
template<typename T>
T add(T a, T b) {
    return a + b;
}
```

这段代码**不会生成任何机器码**！它只是一个模板，编译器看到它时不会立即编译。

### 4.2 实例化触发时机

只有当你**调用**函数模板时，编译器才会根据实际类型**实例化**（instantiate）出真正的函数。

```cpp
int main() {
    add(1, 2);        // 触发实例化：生成 int add(int, int)
    add(1.5, 2.5);    // 触发实例化：生成 double add(double, double)
}
```

### 4.3 实例化过程（编译器视角）

#### 阶段1：模板定义阶段

```cpp
template<typename T>
T add(T a, T b) {
    return a + b;
}
```

编译器：记录模板定义，**不生成代码**

#### 阶段2：模板使用阶段

```cpp
int result = add(10, 20);
```

编译器执行步骤：

1. **类型推导**：从参数 `10, 20` 推导出 `T = int`

2. **实例化**：用 `int` 替换模板中所有的 `T`

3. **生成函数**：

```cpp
// 编译器自动生成的代码（概念示意）
int add(int a, int b) {
    return a + b;
}
```

4. **编译函数**：将生成的函数编译成机器码

5. **调用函数**：调用实例化后的函数

### 4.4 多次实例化

```cpp
int main() {
    add(1, 2);           // 实例化 add<int>
    add(1.5, 2.5);       // 实例化 add<double>
    add(1.5f, 2.5f);     // 实例化 add<float>
}
```

编译器**生成了3个不同的函数**：

```cpp
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }
float add(float a, float b) { return a + b; }
```

每个类型对应一份独立的机器码！

### 4.5 验证实例化

我们可以通过查看符号表来验证：

```cpp
#include <iostream>
using namespace std;

template<typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    cout << add(1, 2) << endl;
    cout << add(1.5, 2.5) << endl;
    return 0;
}
```

编译后使用 `nm` 命令（Linux）或 `dumpbin` 命令（Windows）查看符号：

```
_Z3addIiET_S0_S0_     // add<int>(int, int)
_Z3addIdET_S0_S0_     // add<double>(double, double)
```

可以看到生成了两个不同的函数符号！

### 4.6 代码膨胀问题

**问题**：如果对100种类型使用模板，就会生成100个函数，导致可执行文件变大。

```cpp
add(char, char);
add(short, short);
add(int, int);
add(long, long);
// ... 每种类型都有一份代码
```

**优点**：

- 没有运行时开销（不像虚函数）

- 类型安全（编译期检查）

**缺点**：

- 可执行文件体积增大

- 编译时间变长

---

## 5️⃣ 模板参数推导

编译器如何确定 `T` 的类型？

### 5.1 自动推导

```cpp
template<typename T>
void print(T value) {
    cout << value << endl;
}

int main() {
    print(100);        // T = int
    print(3.14);       // T = double
    print("Hello");    // T = const char*
}
```

### 5.2 推导规则

#### 规则1：完全匹配

```cpp
template<typename T>
void func(T a, T b) {
    // ...
}

func(1, 2);        // ✅ T = int
func(1.5, 2.5);    // ✅ T = double
func(1, 2.5);      // ❌ 错误！T 无法同时是 int 和 double
```

#### 规则2：引用和指针

```cpp
template<typename T>
void func(T& value) {}

int x = 10;
func(x);           // T = int（不是 int&）

const int y = 20;
func(y);           // T = const int
```

#### 规则3：数组退化

```cpp
template<typename T>
void func(T arr) {}

int arr[5];
func(arr);         // T = int*（数组退化为指针）
```

#### 规则4：const 限定符

```cpp
template<typename T>
void func(T value) {}

const int x = 10;
func(x);           // T = int（丢弃顶层const）
```

### 5.3 推导失败的情况

```cpp
template<typename T>
T add(T a, T b) {
    return a + b;
}

add(1, 2.5);       // ❌ 错误！T 无法推导
```

解决方法见下节。

---

## 6️⃣ 显式模板参数

### 6.1 显式指定类型

```cpp
template<typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    // 自动推导失败时，手动指定
    auto result = add<double>(1, 2.5);  // 强制 T = double
    cout << result << endl;  // 3.5
}
```

### 6.2 多个模板参数

```cpp
template<typename T1, typename T2>
void print(T1 a, T2 b) {
    cout << a << " " << b << endl;
}

int main() {
    print(100, 3.14);           // T1 = int, T2 = double
    print<int, int>(100, 3.14); // 强制 T1 = int, T2 = int
}
```

### 6.3 返回类型与参数类型不同

```cpp
template<typename T1, typename T2, typename RT>
RT add(T1 a, T2 b) {
    return static_cast<RT>(a + b);
}

int main() {
    auto result = add<int, double, double>(1, 2.5);
    cout << result << endl;  // 3.5
}
```

**问题**：返回类型无法推导，必须显式指定。

**C++11 改进**：使用 `auto` 和尾置返回类型

```cpp
template<typename T1, typename T2>
auto add(T1 a, T2 b) -> decltype(a + b) {
    return a + b;
}

// C++14 更简洁
template<typename T1, typename T2>
auto add(T1 a, T2 b) {
    return a + b;
}
```

---

## 7️⃣ 函数模板特化

### 7.1 什么是特化

当模板对某些特定类型需要特殊处理时，可以为该类型提供**特化版本**。

### 7.2 示例：比较函数

```cpp
#include <iostream>
#include <cstring>
using namespace std;

// 通用模板
template<typename T>
bool isEqual(T a, T b) {
    return a == b;
}

// 特化：针对 const char* 类型
template<>
bool isEqual<const char*>(const char* a, const char* b) {
    return strcmp(a, b) == 0;
}

int main() {
    cout << isEqual(10, 10) << endl;              // 1（使用通用模板）
    cout << isEqual("hello", "hello") << endl;    // 0（指针比较，不相等）//实际是1
    
    const char* s1 = "hello";
    const char* s2 = "hello";
    cout << isEqual(s1, s2) << endl;              // 1（使用特化版本）
    
    return 0;
}
```

### 7.3 特化语法

```cpp
// 通用模板
template<typename T>
void func(T value) {
    cout << "通用版本" << endl;
}

// 完全特化
template<>
void func<int>(int value) {
    cout << "int 特化版本" << endl;
}
```

---

## 8️⃣ 函数模板重载

函数模板可以重载，编译器根据参数选择最佳匹配。

```cpp
#include <iostream>
using namespace std;

// 版本1：单参数
template<typename T>
void print(T value) {
    cout << "单参数：" << value << endl;
}

// 版本2：双参数
template<typename T>
void print(T a, T b) {
    cout << "双参数：" << a << ", " << b << endl;
}

// 版本3：数组版本
template<typename T>
void print(T arr[], int size) {
    cout << "数组：";
    for (int i = 0; i < size; i++) {
        cout << arr[i] << " ";
    }
    cout << endl;
}

int main() {
    print(100);              // 调用版本1
    print(100, 200);         // 调用版本2
    
    int arr[] = {1, 2, 3};
    print(arr, 3);           // 调用版本3
    
    return 0;
}
```

### 匹配优先级

1. **普通函数** > 函数模板

2. **特化模板** > 通用模板

3. **更特化的模板** > 更通用的模板

```cpp
template<typename T>
void func(T value) {
    cout << "模板版本" << endl;
}

void func(int value) {
    cout << "普通函数版本" << endl;
}

int main() {
    func(10);      // 调用普通函数（优先级更高）
    func(3.14);    // 调用模板
}
```

---

## 9️⃣ 高级主题与注意事项

### 9.1 非类型模板参数

```cpp
template<typename T, int SIZE>
void printArray(T (&arr)[SIZE]) {
    for (int i = 0; i < SIZE; i++) {
        cout << arr[i] << " ";
    }
    cout << endl;
}

int main() {
    int arr1[3] = {1, 2, 3};
    int arr2[5] = {10, 20, 30, 40, 50};
    
    printArray(arr1);  // T = int, SIZE = 3
    printArray(arr2);  // T = int, SIZE = 5
}
```

### 9.2 模板的声明与定义必须在一起

**重要**：模板不能像普通函数那样分离声明和定义！

```cpp
// ❌ 错误示例
// header.h
template<typename T>
T add(T a, T b);  // 声明

// source.cpp
template<typename T>
T add(T a, T b) {  // 定义
    return a + b;
}

// main.cpp
#include "header.h"
int main() {
    add(1, 2);  // ❌ 链接错误！
}
```

**原因**：编译器在编译 `main.cpp` 时看不到模板的定义，无法实例化。

**正确做法**：将定义放在头文件中

```cpp
// header.h
template<typename T>
T add(T a, T b) {
    return a + b;
}
```

### 9.3 typename 的双重含义

在模板中，`typename` 还有另一个用途：告诉编译器某个名称是类型。

```cpp
template<typename T>
void func() {
    typename T::value_type x;  // 告诉编译器 value_type 是类型
}
```

### 9.4 模板编译两阶段检查

1. **定义检查**：检查模板语法（不涉及 `T`）

2. **实例化检查**：检查具体类型是否支持操作

```cpp
template<typename T>
T add(T a, T b) {
    return a + b;  // 只有实例化时才检查 T 是否支持 +
}

class MyClass {};

int main() {
    add(1, 2);              // ✅ int 支持 +
    // add(MyClass(), MyClass());  // ❌ MyClass 不支持 +
}
```

---

## 📖 完整示例

```cpp
#include <iostream>
#include <string>
using namespace std;

// 1. 基础函数模板
template<typename T>
void mySwap(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}

// 2. 多参数模板
template<typename T1, typename T2>
void printPair(T1 first, T2 second) {
    cout << "(" << first << ", " << second << ")" << endl;
}

// 3. 返回类型推导
template<typename T1, typename T2>
auto multiply(T1 a, T2 b) -> decltype(a * b) {
    return a * b;
}

// 4. 数组模板
template<typename T, int N>
void printArray(T (&arr)[N]) {
    for (int i = 0; i < N; i++) {
        cout << arr[i] << " ";
    }
    cout << endl;
}

// 5. 函数模板特化
template<typename T>
T getMax(T a, T b) {
    return (a > b) ? a : b;
}

template<>
const char* getMax<const char*>(const char* a, const char* b) {
    return (strcmp(a, b) > 0) ? a : b;
}

int main() {
    // 测试 mySwap
    int x = 10, y = 20;
    mySwap(x, y);
    cout << "x = " << x << ", y = " << y << endl;
    
    // 测试 printPair
    printPair(100, 3.14);
    printPair(string("Hello"), 'A');
    
    // 测试 multiply
    cout << multiply(3, 4.5) << endl;
    
    // 测试 printArray
    int arr[] = {1, 2, 3, 4, 5};
    printArray(arr);
    
    // 测试特化
    cout << getMax(10, 20) << endl;
    cout << getMax("apple", "banana") << endl;
    
    return 0;
}
```

---

## 🎯 总结

### 核心要点

1. **函数模板是代码生成器**，不是函数本身

2. **实例化发生在编译期**，根据使用的类型生成具体函数

3. **模板定义必须可见**，通常放在头文件中

4. **编译器自动推导类型**，也可显式指定

5. **每个类型生成一份代码**，可能导致代码膨胀

### 底层机制

```
源代码 → 模板定义（不生成代码）
     ↓
   调用点
     ↓
类型推导 → 模板实例化 → 生成具体函数 → 编译成机器码
```

### 适用场景

✅ **适合使用模板**：

- 类型无关的算法（排序、查找、交换等）

- 容器类（vector、list 等）

- 需要类型安全的泛型代码

❌ **不适合使用模板**：

- 代码量很大且会被大量实例化（导致可执行文件膨胀）

- 需要运行时多态（应该用虚函数）

