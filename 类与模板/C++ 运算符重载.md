# C++ 运算符重载完整教程

## 📚 目录

1. 什么是运算符重载

2. 运算符重载的基本语法

3. 成员函数重载 vs 全局函数重载

4. 常见运算符重载示例

5. 特殊运算符重载

6. 运算符重载的注意事项

7. 综合实战案例

---

## 1. 什么是运算符重载

### 1.1 基本概念

**运算符重载**是C++的一个重要特性，它允许我们重新定义运算符的行为，使其能够用于自定义类型（类对象）。

**为什么需要运算符重载？**

```cpp
// 对于内置类型，运算符可以直接使用
int a = 10, b = 20;
int c = a + b;  // ✅ 完全没问题

// 但对于自定义类型...
class Point {
public:
    int x, y;
};

Point p1, p2;
Point p3 = p1 + p2;  // ❌ 编译错误！编译器不知道如何"加"两个Point
```

运算符重载就是告诉编译器：**当遇到自定义类型的运算时，应该怎么做**。

### 1.2 运算符重载的本质

**运算符重载本质上是函数调用的语法糖**。当你写 `a + b` 时，编译器实际上会转换为函数调用：

```cpp
a + b;           // 你写的
a.operator+(b);  // 编译器理解的（成员函数形式）
operator+(a, b); // 或者这样（全局函数形式）
```

---

## 2. 运算符重载的基本语法

### 2.1 语法格式

```cpp
返回类型 operator运算符(参数列表) {
    // 实现逻辑
}
```

### 2.2 第一个例子：加法运算符

```cpp
#include <iostream>
using namespace std;

class Point {
public:
    int x, y;
    
    Point(int x = 0, int y = 0) : x(x), y(y) {}
    
    // 重载 + 运算符（成员函数方式）
    Point operator+(const Point& other) const {
        return Point(this->x + other.x, this->y + other.y);
    }
    
    void display() const {
        cout << "(" << x << ", " << y << ")" << endl;
    }
};

int main() {
    Point p1(3, 4);
    Point p2(1, 2);
    
    Point p3 = p1 + p2;  // 现在可以了！
    p3.display();        // 输出: (4, 6)
    
    return 0;
}
```

**代码解析：**

- `operator+` 是函数名，表示重载 `+` 运算符

- `const Point& other`：传入右操作数（引用避免拷贝，const 保护不被修改）

- `const` 成员函数：表示不会修改调用对象

- 返回新的 `Point` 对象：保持原操作数不变

---

## 3. 成员函数重载 vs 全局函数重载

### 3.1 两种方式对比

| 特性 | 成员函数 | 全局函数 |
| --- | --- | --- |
| 参数数量 | n-1 个（this 是第一个操作数） | n 个 |
| 访问私有成员 | 可以直接访问 | 需要声明为友元 |
| 左操作数类型 | 必须是类对象 | 可以是任意类型 |

### 3.2 成员函数方式

```cpp
class Complex {
private:
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // 成员函数重载：c1 + c2 → c1.operator+(c2)
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
};
```

### 3.3 全局函数方式（使用友元）

```cpp
class Complex {
private:
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // 声明友元函数
    friend Complex operator+(const Complex& c1, const Complex& c2);
};

// 全局函数重载
Complex operator+(const Complex& c1, const Complex& c2) {
    return Complex(c1.real + c2.real, c1.imag + c2.imag);
}
```

### 3.4 什么时候用全局函数？

**当左操作数不是类对象时，必须使用全局函数：**

```cpp
class Complex {
private:
    double real, imag;
    
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // 成员函数：Complex + double
    Complex operator+(double d) const {
        return Complex(real + d, imag);
    }
    
    // 友元全局函数：double + Complex
    friend Complex operator+(double d, const Complex& c);
};

// 全局函数实现
Complex operator+(double d, const Complex& c) {
    return Complex(d + c.real, c.imag);
}

int main() {
    Complex c(1, 2);
    
    Complex r1 = c + 5.0;    // ✅ 调用成员函数
    Complex r2 = 5.0 + c;    // ✅ 调用全局函数
    
    return 0;
}
```

---

## 4. 常见运算符重载示例

### 4.1 算术运算符（+, -, *, /）

```cpp
class Fraction {
private:
    int numerator;   // 分子
    int denominator; // 分母
    
public:
    Fraction(int n = 0, int d = 1) : numerator(n), denominator(d) {}
    
    // 加法
    Fraction operator+(const Fraction& other) const {
        return Fraction(
            numerator * other.denominator + other.numerator * denominator,
            denominator * other.denominator
        );
    }
    
    // 减法
    Fraction operator-(const Fraction& other) const {
        return Fraction(
            numerator * other.denominator - other.numerator * denominator,
            denominator * other.denominator
        );
    }
    
    // 乘法
    Fraction operator*(const Fraction& other) const {
        return Fraction(
            numerator * other.numerator,
            denominator * other.denominator
        );
    }
    
    // 除法
    Fraction operator/(const Fraction& other) const {
        return Fraction(
            numerator * other.denominator,
            denominator * other.numerator
        );
    }
};
```

### 4.2 比较运算符（==, !=, <, >, <=, >=）

```cpp
class Student {
private:
    string name;
    int score;
    
public:
    Student(string n, int s) : name(n), score(s) {}
    
    // 等于
    bool operator==(const Student& other) const {
        return score == other.score;
    }
    
    // 不等于
    bool operator!=(const Student& other) const {
        return !(*this == other);  // 复用 ==
    }
    
    // 小于
    bool operator<(const Student& other) const {
        return score < other.score;
    }
    
    // 大于
    bool operator>(const Student& other) const {
        return other < *this;  // 复用 <
    }
    
    // 小于等于
    bool operator<=(const Student& other) const {
        return !(other < *this);  // 复用 <
    }
    
    // 大于等于
    bool operator>=(const Student& other) const {
        return !(*this < other);  // 复用 <
    }
};
```

### 4.3 赋值运算符（=）

**这是非常重要的运算符！** 默认的赋值运算符是浅拷贝，当类中有指针成员时，可能导致问题。

```cpp
class String {
private:
    char* data;
    int length;
    
public:
    String(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
    }
    
    // 拷贝构造函数
    String(const String& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
    }
    
    // 赋值运算符重载 ⭐
    String& operator=(const String& other) {
        // 1. 检查自我赋值
        if (this == &other) {
            return *this;
        }
        
        // 2. 释放旧资源
        delete[] data;
        
        // 3. 分配新资源并拷贝
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        
        // 4. 返回当前对象引用（支持链式赋值）
        return *this;
    }
    
    // 析构函数
    ~String() {
        delete[] data;
    }
};

int main() {
    String s1("Hello");
    String s2("World");
    String s3;
    
    s3 = s2 = s1;  // 链式赋值
    
    return 0;
}
```

**赋值运算符重载要点：**

1. ✅ 检查自我赋值（`if (this == &other)`）

2. ✅ 释放旧资源

3. ✅ 深拷贝新数据

4. ✅ 返回 `*this` 的引用（支持链式赋值）

### 4.4 复合赋值运算符（+=, -=, *=, /=）

```cpp
class Vector2D {
public:
    double x, y;
    
    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}
    
    // += 运算符
    Vector2D& operator+=(const Vector2D& other) {
        x += other.x;
        y += other.y;
        return *this;  // 返回引用
    }
    
    // + 运算符可以基于 += 实现
    Vector2D operator+(const Vector2D& other) const {
        Vector2D result = *this;  // 拷贝当前对象
        result += other;          // 使用 +=
        return result;
    }
};
```

---

## 5. 特殊运算符重载

### 5.1 自增自减运算符（++, --）

**前置和后置的区分是重点！**

```cpp
class Counter {
private:
    int count;
    
public:
    Counter(int c = 0) : count(c) {}
    
    // 前置 ++（++c）
    Counter& operator++() {
        ++count;
        return *this;  // 返回引用，可以继续操作
    }
    
    // 后置 ++（c++）—— 用 int 占位符区分
    Counter operator++(int) {
        Counter temp = *this;  // 保存原值
        ++count;               // 增加
        return temp;           // 返回原值
    }
    
    // 前置 --（--c）
    Counter& operator--() {
        --count;
        return *this;
    }
    
    // 后置 --（c--）
    Counter operator--(int) {
        Counter temp = *this;
        --count;
        return temp;
    }
    
    int getCount() const { return count; }
};

int main() {
    Counter c(5);
    
    cout << (++c).getCount() << endl;  // 6（先加后用）
    cout << (c++).getCount() << endl;  // 6（先用后加）
    cout << c.getCount() << endl;      // 7
    
    return 0;
}
```

**关键区别：**

| 类型 | 参数 | 返回类型 | 语义 |
| --- | --- | --- | --- |
| 前置 ++c | 无参数 | 引用 Counter& | 先增后用 |
| 后置 c++ | int 占位符 | 值 Counter | 先用后增 |

### 5.2 输入输出运算符（<<, >>）

**必须用全局函数重载！** 因为左操作数是流对象。

```cpp
#include <iostream>
using namespace std;

class Point {
private:
    int x, y;
    
public:
    Point(int x = 0, int y = 0) : x(x), y(y) {}
    
    // 声明友元
    friend ostream& operator<<(ostream& os, const Point& p);
    friend istream& operator>>(istream& is, Point& p);
};

// 输出运算符重载
ostream& operator<<(ostream& os, const Point& p) {
    os << "(" << p.x << ", " << p.y << ")";
    return os;  // 返回流引用，支持链式输出
}

// 输入运算符重载
istream& operator>>(istream& is, Point& p) {
    cout << "Enter x and y: ";
    is >> p.x >> p.y;
    return is;  // 返回流引用，支持链式输入
}

int main() {
    Point p1(3, 4), p2(1, 2);
    
    cout << p1 << " and " << p2 << endl;  // (3, 4) and (1, 2)
    
    Point p3;
    cin >> p3;
    cout << "You entered: " << p3 << endl;
    
    return 0;
}
```

### 5.3 下标运算符（[]）

```cpp
class IntArray {
private:
    int* data;
    int size;
    
public:
    IntArray(int n) : size(n) {
        data = new int[n]();  // 初始化为0
    }
    
    ~IntArray() {
        delete[] data;
    }
    
    // 非 const 版本（可读可写）
    int& operator[](int index) {
        if (index < 0 || index >= size) {
            throw out_of_range("Index out of range");
        }
        return data[index];
    }
    
    // const 版本（只读）
    const int& operator[](int index) const {
        if (index < 0 || index >= size) {
            throw out_of_range("Index out of range");
        }
        return data[index];
    }
    
    int getSize() const { return size; }
};

int main() {
    IntArray arr(5);
    
    arr[0] = 10;  // 使用非 const 版本
    arr[1] = 20;
    
    cout << arr[0] << endl;  // 10
    
    const IntArray constArr(3);
    cout << constArr[0] << endl;  // 使用 const 版本
    
    return 0;
}
```

### 5.4 函数调用运算符（()）—— 仿函数

重载 `()` 运算符的类对象称为**仿函数（Functor）**，可以像函数一样调用。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// 仿函数类
class Adder {
private:
    int value;
    
public:
    Adder(int v) : value(v) {}
    
    // 重载 ()
    int operator()(int x) const {
        return x + value;
    }
};

// 用于排序的仿函数
class DescendingCompare {
public:
    bool operator()(int a, int b) const {
        return a > b;  // 降序
    }
};

int main() {
    // 使用仿函数
    Adder add5(5);
    cout << add5(10) << endl;  // 15
    cout << add5(20) << endl;  // 25
    
    // 在 STL 算法中使用仿函数
    vector<int> nums = {3, 1, 4, 1, 5, 9, 2, 6};
    sort(nums.begin(), nums.end(), DescendingCompare());
    
    for (int n : nums) {
        cout << n << " ";  // 9 6 5 4 3 2 1 1
    }
    
    return 0;
}
```

### 5.5 类型转换运算符

```cpp
class Fraction {
private:
    int numerator;
    int denominator;
    
public:
    Fraction(int n = 0, int d = 1) : numerator(n), denominator(d) {}
    
    // 转换为 double
    operator double() const {
        return static_cast<double>(numerator) / denominator;
    }
    
    // 转换为 bool（用于条件判断）
    operator bool() const {
        return numerator != 0;
    }
};

int main() {
    Fraction f(3, 4);
    
    double d = f;  // 隐式转换为 double
    cout << d << endl;  // 0.75
    
    if (f) {  // 隐式转换为 bool
        cout << "Fraction is non-zero" << endl;
    }
    
    Fraction zero(0, 1);
    if (!zero) {
        cout << "Fraction is zero" << endl;
    }
    
    return 0;
}
```

### 5.6 指针相关运算符（*, ->）

用于实现智能指针等类。

```cpp
template<typename T>
class SmartPointer {
private:
    T* ptr;
    
public:
    SmartPointer(T* p = nullptr) : ptr(p) {}
    
    ~SmartPointer() {
        delete ptr;
    }
    
    // 解引用运算符
    T& operator*() const {
        return *ptr;
    }
    
    // 箭头运算符
    T* operator->() const {
        return ptr;
    }
    
    // 禁止拷贝
    SmartPointer(const SmartPointer&) = delete;
    SmartPointer& operator=(const SmartPointer&) = delete;
};

class MyClass {
public:
    void sayHello() { cout << "Hello!" << endl; }
    int value = 42;
};

int main() {
    SmartPointer<MyClass> sp(new MyClass());
    
    (*sp).sayHello();     // 使用 *
    sp->sayHello();       // 使用 ->
    cout << sp->value << endl;  // 42
    
    return 0;
}
```

---

## 6. 运算符重载的注意事项

### 6.1 不能重载的运算符

| 运算符 | 说明 |
| --- | --- |
| . | 成员访问 |
| .* | 成员指针访问 |
| :: | 作用域解析 |
| ?: | 三元条件 |
| sizeof | 大小计算 |
| typeid | 类型识别 |

### 6.2 必须用成员函数重载的运算符

| 运算符 | 说明 |
| --- | --- |
| = | 赋值 |
| [] | 下标 |
| () | 函数调用 |
| -> | 指针访问 |
| ->* | 指针成员访问 |

### 6.3 推荐用成员函数还是全局函数？

| 运算符类型 | 推荐方式 | 原因 |
| --- | --- | --- |
| 一元运算符 | 成员函数 | 只有一个操作数 |
| =, [], (), -> | 成员函数 | 语言规定必须 |
| +=, -= 等复合赋值 | 成员函数 | 会修改左操作数 |
| <<, >> | 全局函数 | 左操作数是流 |
| 二元算术和比较 | 成员或全局都可 | 看是否需要类型转换 |

### 6.4 重载原则

1. **保持语义一致**：`+` 应该表示加法，不要用它做减法

2. **保持返回类型合理**：比较运算符返回 `bool`，算术运算符返回对象

3. **支持链式操作**：`<<`、`=`、`+=` 等应返回引用

4. **成对重载**：如果重载了 `==`，通常也应重载 `!=`

---

## 7. 综合实战案例

### 7.1 完整的复数类

```cpp
#include <iostream>
#include <cmath>
using namespace std;

class Complex {
private:
    double real;
    double imag;
    
public:
    // 构造函数
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    
    // ========== 算术运算符 ==========
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
    
    Complex operator-(const Complex& other) const {
        return Complex(real - other.real, imag - other.imag);
    }
    
    Complex operator*(const Complex& other) const {
        return Complex(
            real * other.real - imag * other.imag,
            real * other.imag + imag * other.real
        );
    }
    
    Complex operator/(const Complex& other) const {
        double denominator = other.real * other.real + other.imag * other.imag;
        return Complex(
            (real * other.real + imag * other.imag) / denominator,
            (imag * other.real - real * other.imag) / denominator
        );
    }
    
    // 一元负号
    Complex operator-() const {
        return Complex(-real, -imag);
    }
    
    // ========== 复合赋值运算符 ==========
    Complex& operator+=(const Complex& other) {
        real += other.real;
        imag += other.imag;
        return *this;
    }
    
    Complex& operator-=(const Complex& other) {
        real -= other.real;
        imag -= other.imag;
        return *this;
    }
    
    // ========== 比较运算符 ==========
    bool operator==(const Complex& other) const {
        return real == other.real && imag == other.imag;
    }
    
    bool operator!=(const Complex& other) const {
        return !(*this == other);
    }
    
    // ========== 类型转换 ==========
    // 获取模长
    double magnitude() const {
        return sqrt(real * real + imag * imag);
    }
    
    // ========== 友元函数 ==========
    friend ostream& operator<<(ostream& os, const Complex& c);
    friend istream& operator>>(istream& is, Complex& c);
    
    // 支持 double + Complex
    friend Complex operator+(double d, const Complex& c);
    friend Complex operator*(double d, const Complex& c);
};

// 输出
ostream& operator<<(ostream& os, const Complex& c) {
    os << c.real;
    if (c.imag >= 0) os << "+";
    os << c.imag << "i";
    return os;
}

// 输入
istream& operator>>(istream& is, Complex& c) {
    cout << "Enter real and imaginary parts: ";
    is >> c.real >> c.imag;
    return is;
}

// double + Complex
Complex operator+(double d, const Complex& c) {
    return Complex(d + c.real, c.imag);
}

// double * Complex
Complex operator*(double d, const Complex& c) {
    return Complex(d * c.real, d * c.imag);
}

// ========== 测试 ==========
int main() {
    Complex c1(3, 4);
    Complex c2(1, 2);
    
    cout << "c1 = " << c1 << endl;           // 3+4i
    cout << "c2 = " << c2 << endl;           // 1+2i
    cout << "c1 + c2 = " << c1 + c2 << endl; // 4+6i
    cout << "c1 - c2 = " << c1 - c2 << endl; // 2+2i
    cout << "c1 * c2 = " << c1 * c2 << endl; // -5+10i
    cout << "c1 / c2 = " << c1 / c2 << endl; // 2.2+0.4i
    cout << "-c1 = " << -c1 << endl;         // -3-4i
    cout << "|c1| = " << c1.magnitude() << endl; // 5
    
    cout << "2 + c1 = " << 2 + c1 << endl;   // 5+4i
    cout << "2 * c1 = " << 2 * c1 << endl;   // 6+8i
    
    Complex c3 = c1;
    c3 += c2;
    cout << "c3 after += : " << c3 << endl;  // 4+6i
    
    cout << "c1 == c2: " << (c1 == c2) << endl; // 0 (false)
    cout << "c1 != c2: " << (c1 != c2) << endl; // 1 (true)
    
    return 0;
}
```

---

## 📋 总结速查表

| 运算符 | 成员/全局 | 返回类型 | 参数 |
| --- | --- | --- | --- |
| +, -, *, / | 都可 | 新对象 | const T& |
| +=, -=, *=, /= | 成员 | T& | const T& |
| = | 成员（必须） | T& | const T& |
| ++, --（前置） | 成员 | T& | 无 |
| ++, --（后置） | 成员 | T | int |
| ==, !=, < 等 | 都可 | bool | const T& |
| [] | 成员（必须） | T& / const T& | 索引 |
| () | 成员（必须） | 任意 | 任意 |
| <<, >> | 全局 | 流引用 | 流引用, T |
| *, -> | 成员 | T& / T* | 无 |

---

希望这个教程对你学习 C++ 运算符重载有帮助！如果有任何问题，欢迎随时问我 🎉