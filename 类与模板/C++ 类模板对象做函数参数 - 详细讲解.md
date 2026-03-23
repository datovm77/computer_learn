# C++ 类模板对象做函数参数 - 详细讲解

## 一、基础概念回顾

在开始之前，先理解什么是类模板对象：

```cpp
// 定义一个类模板
template<class T>
class Person {
public:
    T m_Age;
    Person(T age) : m_Age(age) {}
};

// 创建类模板对象
Person<int> p1(18);      // 这就是类模板实例化出的对象
Person<string> p2("20");  // 不同类型的对象
```

当我们需要将这些对象传递给函数时，有**三种方式**。

------

## 二、方式一：指定传入的类型（最常用、最直观）

### 2.1 概念说明

直接在函数参数中**明确指定**类模板对象的具体类型，这是最直接的方式。

### 2.2 代码示例

```cpp
#include <iostream>
#include <string>
using namespace std;

// 类模板定义
template<class NameType, class AgeType>
class Person {
public:
    NameType m_Name;
    AgeType m_Age;
    
    Person(NameType name, AgeType age) {
        m_Name = name;
        m_Age = age;
    }
    
    void showPerson() {
        cout << "姓名: " << m_Name << ", 年龄: " << m_Age << endl;
    }
};

// 方式一：指定传入类型
// 直接告诉函数：我要接收的是 Person<string, int> 类型的对象
void printPerson1(Person<string, int>& p) {
    p.showPerson();
    cout << "这是指定类型的方式" << endl;
}

void test01() {
    Person<string, int> p("张三", 18);
    printPerson1(p);  // 只能传入 Person<string, int> 类型
}
```

### 2.3 特点分析

**优点：**

- 简单明了，一目了然
- 类型安全，编译期检查严格
- 适合明确知道参数类型的场景

**缺点：**

- 灵活性差，只能接收指定类型的对象
- 如果需要不同类型，就要写多个函数重载

------

## 三、方式二：参数模板化（灵活性中等）

### 3.1 概念说明

将**函数也设计成模板函数**，但只将类模板中的**部分参数**变为模板参数，保持类模板的结构。

### 3.2 代码示例

```cpp
// 方式二：参数模板化
// 将 Person 类中的模板参数 T1 和 T2 变为函数模板的参数
template<class T1, class T2>
void printPerson2(Person<T1, T2>& p) {
    p.showPerson();
    cout << "这是参数模板化的方式" << endl;
    
    // 可以查看 T1 和 T2 的类型信息
    cout << "T1 的类型: " << typeid(T1).name() << endl;
    cout << "T2 的类型: " << typeid(T2).name() << endl;
}

void test02() {
    Person<string, int> p1("李四", 20);
    printPerson2(p1);  // 自动推导出 T1=string, T2=int
    
    Person<string, double> p2("王五", 25.5);
    printPerson2(p2);  // 自动推导出 T1=string, T2=double
    
    // 也可以显式指定类型
    printPerson2<string, int>(p1);
}
```

### 3.3 深入理解

```cpp
// 更复杂的例子：可以对模板参数进行操作
template<class T1, class T2>
void comparePerson(Person<T1, T2>& p1, Person<T1, T2>& p2) {
    if (p1.m_Age > p2.m_Age) {
        cout << p1.m_Name << " 年龄更大" << endl;
    } else {
        cout << p2.m_Name << " 年龄更大" << endl;
    }
}

void test03() {
    Person<string, int> p1("赵六", 30);
    Person<string, int> p2("孙七", 25);
    comparePerson(p1, p2);
}
```

### 3.4 特点分析

**优点：**

- 灵活性强，可以接收不同类型参数的类模板对象
- 可以在函数内部获取和使用类型信息
- 一个函数可以处理多种类型组合

**缺点：**

- 必须是 Person 类型，不能是其他类模板
- 语法稍微复杂一些

------

## 四、方式三：整个类模板化（最灵活）

### 4.1 概念说明

将**整个类对象的类型**都作为模板参数，这样函数不仅可以接收不同参数的 Person 对象，理论上还可以接收其他类模板对象。

### 4.2 代码示例

```cpp
// 方式三：整个类模板化
// T 代表整个类型，可以是 Person<string, int> 或任何其他类型
template<class T>
void printPerson3(T& p) {
    p.showPerson();
    cout << "这是整个类模板化的方式" << endl;
    cout << "T 的类型: " << typeid(T).name() << endl;
}

void test04() {
    Person<string, int> p1("周八", 22);
    printPerson3(p1);  // T 被推导为 Person<string, int>
    
    Person<char*, double> p2("吴九", 28.5);
    printPerson3(p2);  // T 被推导为 Person<char*, double>
}
```

### 4.3 高级应用

```cpp
// 定义另一个类模板
template<class T>
class Student {
public:
    T m_Score;
    Student(T score) : m_Score(score) {}
    void showPerson() {
        cout << "分数: " << m_Score << endl;
    }
};

void test05() {
    Person<string, int> p("张三", 18);
    Student<double> s(95.5);
    
    // 同一个函数可以接收不同的类模板对象
    printPerson3(p);  // 处理 Person 对象
    printPerson3(s);  // 处理 Student 对象
}
```

### 4.4 特点分析

**优点：**

- 最大灵活性，可以接收任何类型的对象
- 泛型编程的极致体现
- 代码复用性最高

**缺点：**

- 类型约束最弱，可能接收不希望的类型
- 需要确保传入对象有函数中调用的成员（如 showPerson）

------

## 五、三种方式对比总结

```cpp
#include <iostream>
#include <string>
using namespace std;

template<class NameType, class AgeType>
class Person {
public:
    NameType m_Name;
    AgeType m_Age;
    Person(NameType name, AgeType age) : m_Name(name), m_Age(age) {}
    void showPerson() {
        cout << "姓名: " << m_Name << ", 年龄: " << m_Age << endl;
    }
};

// 方式一：指定传入类型
void printPerson1(Person<string, int>& p) {
    cout << "【方式一】";
    p.showPerson();
}

// 方式二：参数模板化
template<class T1, class T2>
void printPerson2(Person<T1, T2>& p) {
    cout << "【方式二】";
    p.showPerson();
}

// 方式三：整个类模板化
template<class T>
void printPerson3(T& p) {
    cout << "【方式三】";
    p.showPerson();
}

int main() {
    Person<string, int> p("李明", 25);
    
    cout << "=== 三种方式演示 ===" << endl;
    printPerson1(p);
    printPerson2(p);
    printPerson3(p);
    
    return 0;
}
```

### 对比表格

| 特性         | 方式一：指定类型 | 方式二：参数模板化 | 方式三：整个类模板化 |
| ------------ | ---------------- | ------------------ | -------------------- |
| **灵活性**   | ★☆☆              | ★★☆                | ★★★                  |
| **类型安全** | ★★★              | ★★★                | ★★☆                  |
| **易读性**   | ★★★              | ★★☆                | ★☆☆                  |
| **使用场景** | 类型固定         | 同类不同参数       | 完全泛型             |
| **推荐指数** | 日常开发         | 中等复杂度         | 高级应用             |

------

## 六、实战建议

### 6.1 选择指南

```cpp
// 场景1：明确知道类型 → 使用方式一
void processUser(Person<string, int>& user) {
    // 处理用户信息，类型固定
}

// 场景2：需要处理多种参数组合 → 使用方式二
template<class T1, class T2>
void saveToDatabase(Person<T1, T2>& person) {
    // 保存不同类型组合的 Person 到数据库
}

// 场景3：编写通用工具函数 → 使用方式三
template<class T>
void serialize(T& obj) {
    // 序列化任意对象
}
```

### 6.2 注意事项

1. **方式一**：最常用，90% 的场景都够用
2. **方式二**：STL 容器常用这种方式（如 `vector<T>`）
3. **方式三**：需要配合 SFINAE 或 Concepts（C++20）进行类型约束

------

## 七、完整示例代码

```cpp
#include <iostream>
#include <string>
using namespace std;

template<class NameType, class AgeType>
class Person {
public:
    NameType m_Name;
    AgeType m_Age;
    
    Person(NameType name, AgeType age) : m_Name(name), m_Age(age) {}
    
    void showPerson() {
        cout << "姓名: " << m_Name << ", 年龄: " << m_Age << endl;
    }
};

// 方式一实现
void printPerson1(Person<string, int>& p) {
    cout << "\n【方式一：指定传入类型】" << endl;
    p.showPerson();
}

// 方式二实现
template<class T1, class T2>
void printPerson2(Person<T1, T2>& p) {
    cout << "\n【方式二：参数模板化】" << endl;
    p.showPerson();
    cout << "T1类型: " << typeid(T1).name() << endl;
    cout << "T2类型: " << typeid(T2).name() << endl;
}

// 方式三实现
template<class T>
void printPerson3(T& p) {
    cout << "\n【方式三：整个类模板化】" << endl;
    p.showPerson();
    cout << "T的类型: " << typeid(T).name() << endl;
}

int main() {
    // 测试方式一
    Person<string, int> p1("张三", 18);
    printPerson1(p1);
    
    // 测试方式二
    Person<string, int> p2("李四", 20);
    Person<char*, double> p3("王五", 25.5);
    printPerson2(p2);
    printPerson2(p3);
    
    // 测试方式三
    Person<string, int> p4("赵六", 30);
    printPerson3(p4);
    
    return 0;
}
```

这三种方式从简单到复杂，从具体到抽象，掌握它们能让你在 C++ 泛型编程中游刃有余！