# C++ 类模板 —— 完整知识总结

---

## 一、类模板基本语法

### 1.1 什么是类模板？

类模板是 C++ 泛型编程的核心机制之一。它允许我们编写一个**与类型无关**的类定义，在使用时再指定具体类型，由编译器生成对应的类代码。

### 1.2 基本语法

```cpp
template<typename T>  // 或 template<class T>
class MyClass {
public:
    MyClass(T value) : m_value(value) {}
    T getValue() const { return m_value; }
private:
    T m_value;
};
```

- `template<typename T>` —— 声明这是一个模板，`T` 是类型参数（占位符）。

- `typename` 和 `class` 在此处完全等价，均可使用。

- 可以有**多个类型参数**：

```cpp
template<typename NameType, typename AgeType>
class Person {
public:
    Person(NameType name, AgeType age) : m_name(name), m_age(age) {}

    void showInfo() const {
        std::cout << "姓名: " << m_name << "  年龄: " << m_age << std::endl;
    }
private:
    NameType m_name;
    AgeType m_age;
};
```

### 1.3 使用方式

```cpp
int main() {
    // 必须用尖括号显式指定类型
    Person<std::string, int> p1("张三", 28);
    p1.showInfo();

    Person<std::string, double> p2("李四", 25.5);
    p2.showInfo();

    return 0;
}
```

### 1.4 默认模板参数（C++11 起）

类模板的类型参数可以有**默认值**：

```cpp
template<typename NameType, typename AgeType = int>
class Person {
    // ...
};

// 使用时可以省略有默认值的类型参数
Person<std::string> p("王五", 30);  // AgeType 默认为 int
```

### 1.5 C++17 类模板参数推导（CTAD）

C++17 引入了**类模板参数推导（Class Template Argument Deduction）**，允许编译器从构造函数参数中自动推导模板参数：

```cpp
std::pair p(1, 2.0);             // 推导为 std::pair<int, double>
std::vector v{1, 2, 3};          // 推导为 std::vector<int>
```

> 说明：CTAD 是“按构造参数推导”，并不是“所有场景都能推导成功”；推导失败时仍需手写模板参数。

也可以为自定义类编写**推导指引（Deduction Guide）**：

```cpp
template<typename T>
class MyClass {
public:
    MyClass(T val) : m_value(val) {}
private:
    T m_value;
};

// 推导指引
MyClass(const char*) -> MyClass<std::string>;

MyClass obj("hello");  // 推导为 MyClass<std::string>，而非 MyClass<const char*>
```

### 1.6 非类型模板参数

模板参数不仅可以是类型，还可以是**编译期常量值**：

```cpp
template<typename T, std::size_t N>
class StaticArray {
public:
    T& operator[](std::size_t index) { return m_data[index]; }
    constexpr std::size_t size() const { return N; }
private:
    T m_data[N];
};

StaticArray<int, 10> arr;  // 一个包含10个int的静态数组
```

> C++20 起，非类型模板参数可以是浮点数、字面类类型（literal class type）等。

---

## 二、类模板与函数模板的区别

| 对比项       | 函数模板                 | 类模板                                             |
| ------------ | ------------------------ | -------------------------------------------------- |
| 类型推导     | ✅ 可以自动推导参数类型   | ❌ 通常需要显式指定（C++17 CTAD 除外）              |
| 默认模板参数 | C++11 起支持             | 一直都支持                                         |
| 使用语法     | 直接调用 func(arg)       | 必须 ClassName<Type> obj                           |
| 特化         | 支持全特化，不支持偏特化 | 支持全特化和偏特化                                 |
| 声明与定义   | 通常放在头文件           | 通常放在头文件（原因相同：编译器需要看到完整定义） |

### 2.1 关键区别一：类型推导

```cpp
// 函数模板 —— 自动推导
template<typename T>
T add(T a, T b) { return a + b; }

add(1, 2);      // ✅ 自动推导 T = int
add(1.5, 2.5);  // ✅ 自动推导 T = double

// 类模板 —— 必须显式指定（C++17 之前）
template<typename T>
class Calculator {
public:
    T add(T a, T b) { return a + b; }
};

Calculator<int> calc;       // ✅ 必须指定
// Calculator calc2;        // ❌ 编译错误（C++17 之前）
```

### 2.2 关键区别二：偏特化

```cpp
// 类模板支持偏特化
template<typename T1, typename T2>
class MyPair {
public:
    void info() { std::cout << "通用版本" << std::endl; }
};

// 偏特化：当第二个参数为 int 时
template<typename T1>
class MyPair<T1, int> {
public:
    void info() { std::cout << "第二参数特化为 int" << std::endl; }
};

// 全特化
template<>
class MyPair<std::string, int> {
public:
    void info() { std::cout << "完全特化为 string + int" << std::endl; }
};
```

```cpp
MyPair<double, double> a;   // 通用版本
MyPair<double, int>    b;   // 偏特化版本
MyPair<std::string, int> c; // 全特化版本（更优先）
```

> **函数模板不支持偏特化**，但可以通过函数重载来达到类似效果。

---

## 三、类模板中成员函数的创建时机

### 3.1 核心原则：**延迟实例化（Lazy Instantiation）**

类模板的成员函数**不是在模板定义时就全部生成**，而是在**该成员函数真正被用到（如调用）时**才实例化。

```cpp
template<typename T>
class TestClass {
public:
    void funcA() {
        T obj;
        obj.methodOnlyInClassA();  // 只有 A 类才有此方法
    }

    void funcB() {
        T obj;
        obj.methodOnlyInClassB();  // 只有 B 类才有此方法
    }
};

class A {
public:
    void methodOnlyInClassA() { std::cout << "A" << std::endl; }
};

class B {
public:
    void methodOnlyInClassB() { std::cout << "B" << std::endl; }
};
```

```cpp
int main() {
    TestClass<A> testA;
    testA.funcA();   // ✅ 正常编译和运行
    // testA.funcB(); // ❌ 调用时才实例化，此时发现 A 没有 methodOnlyInClassB，编译错误

    TestClass<B> testB;
    // testB.funcA(); // ❌ 同理，B 没有 methodOnlyInClassA
    testB.funcB();   // ✅ 正常

    return 0;
}
```

### 3.2 为什么这很重要？

1. **灵活性**：同一个类模板可以包含针对不同类型的成员函数，只要用户不同时调用冲突的函数，就不会编译错误。

2. **编译效率**：未调用的成员函数不会被实例化，减少编译产物。

3. **注意**：这也意味着，**与模板参数相关的错误可能在未实例化前“隐藏”**，直到某次实例化时才暴露。

### 3.3 `if constexpr` 的配合（C++17）

```cpp
template<typename T>
class SmartProcessor {
public:
    void process() {
        if constexpr (std::is_integral_v<T>) {
            std::cout << "处理整数类型" << std::endl;
        } else if constexpr (std::is_floating_point_v<T>) {
            std::cout << "处理浮点类型" << std::endl;
        } else {
            std::cout << "处理其他类型" << std::endl;
        }
    }
};
```

`if constexpr` 在编译期决定分支，被丢弃的分支**不会被实例化**，从根本上避免了类型不匹配的问题。

---

## 四、类模板对象做函数参数

类模板实例化后就是一个具体类型，可以作为函数参数传递。共有**三种方式**：

### 4.1 方式一：指定传入类型（最常用✅）

```cpp
template<typename T1, typename T2>
class Person {
public:
    Person(T1 name, T2 age) : m_name(name), m_age(age) {}
    void showInfo() const {
        std::cout << "姓名: " << m_name << " 年龄: " << m_age << std::endl;
    }
    T1 m_name;
    T2 m_age;
};

// 直接指定具体类型
void printPerson1(const Person<std::string, int>& p) {
    p.showInfo();
}
```

- **优点**：直观明了，编译器可以做完整检查。

- **缺点**：只能接受特定类型的实例。

- **实际开发中使用最广泛**。

### 4.2 方式二：参数模板化

```cpp
template<typename T1, typename T2>
void printPerson2(const Person<T1, T2>& p) {
    p.showInfo();
    // 可通过 typeid 查看推导出的类型
    std::cout << "T1 类型: " << typeid(T1).name() << std::endl;
    std::cout << "T2 类型: " << typeid(T2).name() << std::endl;
}
```

- **优点**：更灵活，可接受不同类型参数的 `Person`。

- 本质上是**函数模板 + 类模板**的配合。

### 4.3 方式三：整个类模板化

```cpp
template<typename T>
void printPerson3(const T& p) {
    p.showInfo();
    std::cout << "T 类型: " << typeid(T).name() << std::endl;
}
```

- **优点**：最通用，可以接受任何“满足调用需求”的类型（常称鸭子类型风格）。

- **缺点**：类型约束较弱（C++20 `concepts` 可以解决此问题）。

### 4.4 C++20 Concepts 约束（扩展）

```cpp
template<typename T>
concept HasShowInfo = requires(const T& t) {
    t.showInfo();
};

void printPerson4(const HasShowInfo auto& p) {
    p.showInfo();
}
```

使用 Concepts 可以在编译期给出**清晰的错误信息**，而不是传统模板的"模板错误瀑布"。

### 4.5 调用示例

```cpp
int main() {
    Person<std::string, int> p("张三", 25);

    printPerson1(p);    // 指定传入类型
    printPerson2(p);    // 参数模板化
    printPerson3(p);    // 整个类模板化

    return 0;
}
```

---

## 五、类模板与继承

### 5.1 基本规则

当父类是类模板时，子类在继承时**必须指明父类中模板参数的类型**（或者子类自身也是模板）。

#### 情况一：子类是普通类，必须指定父类模板参数

```cpp
template<typename T>
class Base {
public:
    T m_value;
};

// 子类继承时必须指定 T 的具体类型
class Derived : public Base<int> {
    // m_value 的类型为 int
};
```

**为什么必须指定？** —— 因为编译器需要知道 `Base` 的内存布局来确定 `Derived` 的大小。如果不指定 `T`，编译器无法计算 `sizeof(Derived)`。

#### 情况二：子类也是模板，灵活传递类型参数

```cpp
template<typename T>
class Base {
public:
    T m_value;
};

template<typename T1, typename T2>
class Derived : public Base<T1> {
public:
    T2 m_extra;
};

Derived<std::string, int> d;
// d.m_value 是 std::string 类型
// d.m_extra 是 int 类型
```

### 5.2 继承中的模板特化

```cpp
template<typename T>
class Base {
public:
    virtual void process(T val) {
        std::cout << "Base: " << val << std::endl;
    }
    virtual ~Base() = default;
};

// 子类可以对特定类型进行特化
template<>
class Base<std::string> {
public:
    virtual void process(std::string val) {
        std::cout << "Base<string>特化: " << val << std::endl;
    }
    virtual ~Base() = default;
};
```

### 5.3 CRTP —— 奇异递归模板模式（Curiously Recurring Template Pattern）

这是类模板与继承结合的高级用法，是**静态多态**的经典实现：

```cpp
template<typename Derived>
class Base {
public:
    void interface() {
        // 安全地将 this 转换为 Derived 类型，调用子类方法
        static_cast<Derived*>(this)->implementation();
    }
};

class ConcreteA : public Base<ConcreteA> {
public:
    void implementation() {
        std::cout << "ConcreteA 的实现" << std::endl;
    }
};

class ConcreteB : public Base<ConcreteB> {
public:
    void implementation() {
        std::cout << "ConcreteB 的实现" << std::endl;
    }
};
```

CRTP 的优势：

- **零开销多态**：没有虚函数表的运行时开销。

- 常用于：静态接口强制、mixin 模式、编译期多态。

> C++23 引入了 **`this` 推导（Deducing `this`）**，可以更优雅地替代部分 CRTP 用法。

---

## 六、类模板成员函数的类外实现

### 6.1 基本语法

在类外实现类模板的成员函数时，**每个函数前都必须加模板声明**，并且需要用**完整的类名（含模板参数）**来限定：

```cpp
template<typename T1, typename T2>
class Person {
public:
    Person(T1 name, T2 age);
    void showInfo() const;
private:
    T1 m_name;
    T2 m_age;
};

// 构造函数 —— 类外实现
template<typename T1, typename T2>
Person<T1, T2>::Person(T1 name, T2 age) : m_name(name), m_age(age) {}

// 成员函数 —— 类外实现
template<typename T1, typename T2>
void Person<T1, T2>::showInfo() const {
    std::cout << "姓名: " << m_name << " 年龄: " << m_age << std::endl;
}
```

### 6.2 要点总结

| 要素     | 说明                                              |
| -------- | ------------------------------------------------- |
| 模板声明 | 每个函数都要写 template<typename T1, typename T2> |
| 类名限定 | 使用 Person<T1, T2>:: 而非 Person::               |
| 返回类型 | 如果返回类型涉及模板参数，也要完整写出            |
| 定义位置 | 通常仍然放在头文件中（原因见下一节）              |

### 6.3 返回类型中使用类模板（易错点）

```cpp
template<typename T>
class Container {
public:
    struct Iterator {
        // ...
    };

    Iterator begin();
    Iterator end();
};

// 类外实现时，返回类型中的 Iterator 需要完整限定
template<typename T>
typename Container<T>::Iterator Container<T>::begin() {
    // ...
}
```

> C++11 的**尾置返回类型**可以简化：
>
>
> ```cpp
> template<typename T>
> auto Container<T>::begin() -> Iterator {
>  // 进入类作用域后，可以直接使用 Iterator
> }
> ```

补充：`typename Container<T>::Iterator` 中的 `typename` 不能省略，因为这里是“依赖名”，编译器需要它来确认这是一个类型。

---

## 七、类模板分文件编写

### 7.1 问题描述

当类模板的声明放在 `.h` 文件，定义放在 `.cpp` 文件时，会出现**链接错误（Linker Error）**：

```
error LNK2019: 无法解析的外部符号 ...
```

### 7.2 原因分析

```
编译流程：
  ┌─────────────┐           ┌─────────────┐
  │  Person.h   │           │  Person.cpp  │
  │ (声明)       │           │ (定义)       │
  └──────┬──────┘           └──────┬──────┘
         │                         │
    被 main.cpp                编译为
    #include                   Person.obj
         │                         │
  ┌──────┴──────┐                  │
  │  main.cpp   │                  │
  │ Person<string, int> p;        │
  └──────┬──────┘                  │
         │                         │
    编译为 main.obj                │
    (需要 Person<string,int>       │
     的具体实现)                  │
         │                         │
         └────── 链接 ──────────────┘
                   ❌ Person.obj 中没有
                   Person<string,int> 的实例！
```

**根本原因**：模板不是真正的代码，只是代码的"蓝图"。编译 `Person.cpp` 时，编译器不知道应该为哪些类型生成实例。编译 `main.cpp` 时，编译器能看到声明但看不到定义。链接时就找不到实现。

### 7.3 解决方案

#### 方案一：将声明和定义写在同一个头文件（推荐✅）

将文件后缀改为 `.hpp`（约定俗成，不是强制），表明这是一个包含模板实现的头文件：

**Person.hpp**

```cpp
#pragma once
#include <iostream>
#include <string>

template<typename T1, typename T2>
class Person {
public:
    Person(T1 name, T2 age);
    void showInfo() const;
private:
    T1 m_name;
    T2 m_age;
};

// 定义也在同一文件中
template<typename T1, typename T2>
Person<T1, T2>::Person(T1 name, T2 age) : m_name(name), m_age(age) {}

template<typename T1, typename T2>
void Person<T1, T2>::showInfo() const {
    std::cout << "姓名: " << m_name << " 年龄: " << m_age << std::endl;
}
```

**main.cpp**

```cpp
#include "Person.hpp"

int main() {
    Person<std::string, int> p("张三", 25);
    p.showInfo();
    return 0;
}
```

#### 方案二：显式实例化（Explicit Instantiation）

在 `.cpp` 文件末尾显式告诉编译器要生成哪些类型的实例：

**Person.cpp**

```cpp
#include "Person.h"

template<typename T1, typename T2>
Person<T1, T2>::Person(T1 name, T2 age) : m_name(name), m_age(age) {}

template<typename T1, typename T2>
void Person<T1, T2>::showInfo() const {
    std::cout << "姓名: " << m_name << " 年龄: " << m_age << std::endl;
}

// 显式实例化
template class Person<std::string, int>;
template class Person<std::string, double>;
```

- **优点**：保持了声明/定义分离的传统结构，减少头文件膨胀。

- **缺点**：必须预先知道所有要用到的类型组合，失去了模板的通用性。

#### 方案三：在 `.h` 中 include `.cpp`（不推荐⚠️）

```cpp
// Person.h 末尾
#include "Person.cpp"
```

这本质上等价于方案一，但容易引起混淆和重复包含问题。

### 7.4 最佳实践

| 场景                   | 推荐方案                          |
| ---------------------- | --------------------------------- |
| 通用类模板库（如 STL） | 方案一（.hpp 头文件）             |
| 仅内部使用、类型有限   | 方案二（显式实例化）              |
| 大型项目减少编译时间   | 方案二 + extern template（C++11） |

**C++11 `extern template` 示例**：

```cpp
// Person.hpp 中
extern template class Person<std::string, int>;  // 告诉编译器：不要在这个翻译单元实例化

// Person.cpp 中
template class Person<std::string, int>;  // 只在这里实例化
```

这可以避免多个翻译单元重复实例化同一个模板，加速编译。

---

## 八、类模板与友元

### 8.1 全局函数做友元 —— 类内实现（简单✅）

```cpp
template<typename T1, typename T2>
class Person {
    // 友元函数直接在类内定义
    friend void printPerson(const Person<T1, T2>& p) {
        std::cout << "姓名: " << p.m_name << " 年龄: " << p.m_age << std::endl;
    }

public:
    Person(T1 name, T2 age) : m_name(name), m_age(age) {}

private:
    T1 m_name;
    T2 m_age;
};
```

```cpp
int main() {
    Person<std::string, int> p("张三", 25);
    printPerson(p);  // ✅ 正常调用
    return 0;
}
```

> **注意**：这里的 `printPerson` 实际上**不是**模板函数，而是一个普通函数。每当 `Person` 被实例化为一个新类型时，就会生成一个新的 `printPerson` 重载。
>
> 另外，这类友元通常依赖 ADL（实参相关查找）被找到，所以示例里可以直接写 `printPerson(p)`。

### 8.2 全局函数做友元 —— 类外实现（复杂⚠️）

类外实现时，需要进行**前置声明**，步骤较多：

```cpp
// ===== 第一步：前置声明类模板 =====
template<typename T1, typename T2>
class Person;

// ===== 第二步：前置声明友元函数模板 =====
template<typename T1, typename T2>
void printPerson(const Person<T1, T2>& p);

// ===== 第三步：类模板定义 =====
template<typename T1, typename T2>
class Person {
    // 注意：这里声明友元时要加 <> 表示这是一个函数模板
    friend void printPerson<>(const Person<T1, T2>& p);

public:
    Person(T1 name, T2 age) : m_name(name), m_age(age) {}

private:
    T1 m_name;
    T2 m_age;
};

// ===== 第四步：友元函数模板的定义 =====
template<typename T1, typename T2>
void printPerson(const Person<T1, T2>& p) {
    std::cout << "姓名: " << p.m_name << " 年龄: " << p.m_age << std::endl;
}
```

### 8.3 为什么类外实现这么复杂？

```
友元声明 friend void printPerson<>(...)

  编译器在这里需要知道：
  1. printPerson 是什么？—— 需要前置声明
  2. printPerson 是函数模板吗？—— 需要 <> 表明
  3. Person 是什么？—— printPerson 的参数中用到了 Person，需要前置声明

  所以必须按顺序：Person前置声明 → printPerson前置声明 → Person定义 → printPerson定义
```

### 8.4 友元类模板

```cpp
template<typename T>
class Container;

template<typename T>
class Printer {
public:
    void print(const Container<T>& c) {
        // 可以访问 Container 的 private 成员
        std::cout << "Value: " << c.m_data << std::endl;
    }
};

template<typename T>
class Container {
    friend class Printer<T>;  // 只有相同类型参数的 Printer 是友元
public:
    Container(T data) : m_data(data) {}
private:
    T m_data;
};
```

### 8.5 实际建议

> **建议优先使用"友元函数类内实现"的方式**，因为语法简单，不容易出错。如果友元函数逻辑复杂，可以让友元函数内部调用公有接口来减少直接访问 private 成员。

---

## 九、类模板案例 —— 通用数组类封装（需求分析）

### 9.1 需求描述

设计一个**通用的数组类**，能够存储任意类型的数据，要求：

1. 可以使用**构造函数**传入数组容量。

2. 通过**拷贝构造函数**和 `operator=` 实现深拷贝，防止浅拷贝问题。

3. 提供**尾插法**（`push_back`）和**尾删法**（`pop_back`）。

4. 可以通过**下标运算符** `[]` 访问数组元素。

5. 可以获取当前数组中有效元素个数（`size`）和容量（`capacity`）。

6. 存储**内置类型**和**自定义类型**的数据。

### 9.2 设计分析

```
MyArray<T>
├── 私有成员
│   ├── T* m_pAddress      // 指向堆区的数组指针
│   ├── int m_capacity     // 数组容量
│   └── int m_size         // 当前有效元素个数
│
├── 构造与析构
│   ├── MyArray(int capacity)                       // 有参构造
│   ├── MyArray(const MyArray& other)               // 拷贝构造（深拷贝）
│   ├── MyArray(MyArray&& other) noexcept           // 移动构造（C++11）
│   └── ~MyArray()                                  // 析构，释放堆区内存
│
├── 运算符重载
│   ├── MyArray& operator=(const MyArray& other)    // 赋值运算符（深拷贝）
│   ├── MyArray& operator=(MyArray&& other)         // 移动赋值（C++11）
│   └── T& operator[](int index)                    // 下标访问
│
├── 数据操作
│   ├── void push_back(const T& val)                // 尾插
│   ├── void pop_back()                              // 尾删
│   ├── int size() const                             // 有效元素个数
│   └── int capacity() const                         // 容量
│
└── 注意事项
    ├── 深拷贝：防止堆区内存重复释放
    ├── 移动语义：高效转移资源
    └── 存储自定义类型时需要 T 有默认构造函数
```

---

## 十、类模板案例 —— 数组类封装（上）

### 10.1 头文件 `MyArray.hpp`

```cpp
#pragma once
#include <iostream>
#include <stdexcept>
#include <algorithm>    // std::copy
#include <utility>      // std::move, std::exchange

template<typename T>
class MyArray {
public:
    // ========== 构造函数 ==========

    // 有参构造：指定容量
    explicit MyArray(int capacity)
        : m_capacity(capacity)
        , m_size(0)
        , m_pAddress(capacity > 0 ? new T[capacity]{} : nullptr)   // 先判容量，避免非法容量参与分配
    {
        if (capacity <= 0) {
            throw std::invalid_argument("容量必须大于0");
        }
    }

    // 拷贝构造：深拷贝
    MyArray(const MyArray& other)
        : m_capacity(other.m_capacity)
        , m_size(other.m_size)
        , m_pAddress(new T[other.m_capacity])
    {
        std::copy(other.m_pAddress, other.m_pAddress + other.m_size, m_pAddress);
    }

    // 移动构造（C++11）：窃取资源
    MyArray(MyArray&& other) noexcept
        : m_capacity(std::exchange(other.m_capacity, 0))
        , m_size(std::exchange(other.m_size, 0))
        , m_pAddress(std::exchange(other.m_pAddress, nullptr))
    {
    }

    // ========== 析构函数 ==========
    ~MyArray() {
        delete[] m_pAddress;
        m_pAddress = nullptr;
        m_capacity = 0;
        m_size = 0;
    }

    // ========== 赋值运算符 ==========

    // 拷贝赋值：使用 copy-and-swap 惯用法
    MyArray& operator=(const MyArray& other) {
        if (this != &other) {
            MyArray temp(other);             // 拷贝
            swap(*this, temp);               // 交换
        }                                    // temp 析构，释放旧资源
        return *this;
    }

    // 移动赋值
    MyArray& operator=(MyArray&& other) noexcept {
        if (this != &other) {
            delete[] m_pAddress;
            m_capacity = std::exchange(other.m_capacity, 0);
            m_size     = std::exchange(other.m_size, 0);
            m_pAddress = std::exchange(other.m_pAddress, nullptr);
        }
        return *this;
    }

    // 友元 swap（配合 copy-and-swap）
    friend void swap(MyArray& a, MyArray& b) noexcept {
        using std::swap;
        swap(a.m_capacity, b.m_capacity);
        swap(a.m_size, b.m_size);
        swap(a.m_pAddress, b.m_pAddress);
    }

    // ========== 下标运算符 ==========

    T& operator[](int index) {
        if (index < 0 || index >= m_size) {
            throw std::out_of_range("下标越界！");
        }
        return m_pAddress[index];
    }

    const T& operator[](int index) const {
        if (index < 0 || index >= m_size) {
            throw std::out_of_range("下标越界！");
        }
        return m_pAddress[index];
    }

    // ========== 数据操作 ==========

    // 尾插
    void push_back(const T& val) {
        if (m_size >= m_capacity) {
            throw std::overflow_error("数组已满，无法插入！");
        }
        m_pAddress[m_size] = val;
        ++m_size;
    }

    // 尾插（移动版本）
    void push_back(T&& val) {
        if (m_size >= m_capacity) {
            throw std::overflow_error("数组已满，无法插入！");
        }
        m_pAddress[m_size] = std::move(val);
        ++m_size;
    }

    // 尾删
    void pop_back() {
        if (m_size == 0) {
            throw std::underflow_error("数组为空，无法删除！");
        }
        --m_size;
    }

    // ========== 属性获取 ==========

    [[nodiscard]] int size() const noexcept { return m_size; }
    [[nodiscard]] int capacity() const noexcept { return m_capacity; }
    [[nodiscard]] bool empty() const noexcept { return m_size == 0; }

private:
    T*  m_pAddress;    // 堆区数组指针
    int m_capacity;    // 容量
    int m_size;        // 有效元素个数
};
```

### 10.2 关键设计说明

| 设计要点      | 说明                                                 |
| ------------- | ---------------------------------------------------- |
| explicit 构造 | 防止隐式的 int → MyArray 转换                        |
| 深拷贝        | 使用 std::copy 而非手动循环，更安全也更清晰          |
| 移动语义      | 使用 std::exchange 安全地转移资源并将源置零          |
| copy-and-swap | 赋值运算符的经典惯用法，同时保证异常安全和自赋值安全 |
| [[nodiscard]] | C++17 属性，提醒调用者不要忽略返回值                 |
| noexcept      | 移动操作标记为 noexcept，使 STL 容器能优先使用移动   |
| const 重载 [] | 提供 const 版本，使 const 对象也能访问元素           |

---

## 十一、类模板案例 —— 数组类封装（下）

### 11.1 测试：存储内置类型

```cpp
#include "MyArray.hpp"
#include <iostream>

void testBuiltInType() {
    std::cout << "===== 测试内置类型 =====" << std::endl;

    MyArray<int> arr(10);

    // 尾插
    for (int i = 0; i < 10; ++i) {
        arr.push_back(i * 10);
    }

    // 遍历
    std::cout << "数组内容: ";
    for (int i = 0; i < arr.size(); ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;

    std::cout << "容量: " << arr.capacity() << std::endl;
    std::cout << "大小: " << arr.size() << std::endl;

    // 尾删
    arr.pop_back();
    std::cout << "尾删后大小: " << arr.size() << std::endl;

    // 测试拷贝构造
    MyArray<int> arr2(arr);
    std::cout << "拷贝后 arr2 大小: " << arr2.size() << std::endl;

    // 测试赋值运算符
    MyArray<int> arr3(5);
    arr3 = arr;
    std::cout << "赋值后 arr3 大小: " << arr3.size() << std::endl;

    // 测试移动构造
    MyArray<int> arr4(std::move(arr));
    std::cout << "移动后 arr4 大小: " << arr4.size() << std::endl;
    std::cout << "移动后 arr 大小: " << arr.size() << std::endl;  // 0
}
```

### 11.2 测试：存储自定义类型

```cpp
#include <string>

class Person {
public:
    Person() = default;  // ⚠️ 必须有默认构造函数！因为 new T[capacity]{} 需要它

    Person(std::string name, int age)
        : m_name(std::move(name)), m_age(age) {}

    // 用于输出
    friend std::ostream& operator<<(std::ostream& os, const Person& p) {
        os << "[" << p.m_name << ", " << p.m_age << "]";
        return os;
    }

private:
    std::string m_name;
    int m_age{0};
};

void testCustomType() {
    std::cout << "\n===== 测试自定义类型 =====" << std::endl;

    MyArray<Person> personArr(5);

    personArr.push_back(Person("张三", 25));
    personArr.push_back(Person("李四", 30));
    personArr.push_back(Person("王五", 28));
    personArr.push_back(Person("赵六", 22));
    personArr.push_back(Person("钱七", 35));

    std::cout << "人员列表:" << std::endl;
    for (int i = 0; i < personArr.size(); ++i) {
        std::cout << "  " << personArr[i] << std::endl;
    }

    std::cout << "容量: " << personArr.capacity() << std::endl;
    std::cout << "大小: " << personArr.size() << std::endl;

    // 测试异常处理
    try {
        personArr.push_back(Person("多余", 99));  // 超出容量
    } catch (const std::overflow_error& e) {
        std::cout << "捕获异常: " << e.what() << std::endl;
    }

    try {
        std::cout << personArr[10] << std::endl;   // 越界
    } catch (const std::out_of_range& e) {
        std::cout << "捕获异常: " << e.what() << std::endl;
    }
}
```

### 11.3 主函数

```cpp
int main() {
    testBuiltInType();
    testCustomType();
    return 0;
}
```

### 11.4 预期输出

```
===== 测试内置类型 =====
数组内容: 0 10 20 30 40 50 60 70 80 90
容量: 10
大小: 10
尾删后大小: 9
拷贝后 arr2 大小: 9
赋值后 arr3 大小: 9
移动后 arr4 大小: 9
移动后 arr 大小: 0

===== 测试自定义类型 =====
人员列表:
  [张三, 25]
  [李四, 30]
  [王五, 28]
  [赵六, 22]
  [钱七, 35]
容量: 5
大小: 5
捕获异常: 数组已满，无法插入！
捕获异常: 下标越界！
```

### 11.5 重要注意事项

| 注意事项     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| 默认构造函数 | 在本实现里使用了 `new T[n]{}`，因此 `T` 需要可默认构造；若改为“原始内存 + 定位 new”的设计，可放宽该限制 |
| 深拷贝       | 如果自定义类型内部也有堆区内存，该类型自身也需要正确实现拷贝/移动语义 |
| 异常安全     | 使用 try-catch 测试边界条件                                  |
| 移动语义     | push_back(T&& val) 重载使得临时对象可以被移动而非拷贝，提升性能 |
| const 正确性 | 提供 const 版本的 operator[]、属性获取方法标记 const         |

---

## 📌 全局知识图谱总结

```
                    ┌─────────────────────┐
                    │     类模板基本语法    │
                    │  template<typename T>│
                    └─────────┬───────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
   ┌──────────┴──────┐  ┌────┴─────┐  ┌──────┴──────────┐
   │ 与函数模板区别   │  │ 成员函数  │  │  默认模板参数    │
   │ - 无自动推导     │  │ 创建时机  │  │  CTAD (C++17)   │
   │ - 支持偏特化     │  │ 延迟实例化│  │  非类型模板参数  │
   └─────────────────┘  └──────────┘  └─────────────────┘
              │
    ┌─────────┼──────────────┐
    │         │              │
┌───┴───┐ ┌──┴──────┐ ┌─────┴─────┐
│做函数  │ │与继承    │ │  与友元    │
│参数    │ │- 指定类型│ │- 类内实现  │
│3种方式 │ │- 传递参数│ │- 类外实现  │
│+概念   │ │- CRTP   │ │- 前置声明  │
└───────┘ └─────────┘ └───────────┘
              │
    ┌─────────┼──────────┐
    │         │          │
┌───┴────┐ ┌─┴──────┐ ┌─┴──────────┐
│类外实现 │ │分文件   │ │ 实战案例    │
│完整类名 │ │编写     │ │ MyArray    │
│模板声明 │ │.hpp方案 │ │ 深拷贝+移动 │
└────────┘ └────────┘ └────────────┘
```

---

以上就是 C++ 类模板从基础到进阶的完整知识总结。核心要点：

1. **类模板是"蓝图"**，不是真正的类，实例化后才生成真正的类。

2. **成员函数延迟实例化**，只有调用时才编译。

3. **分文件编写**推荐使用 `.hpp` 头文件方案。

4. **友元函数类内实现**远比类外实现简单。

5. 现代 C++ 推荐结合**移动语义、`concepts`、`if constexpr`** 等特性，写出更安全高效的模板代码。
