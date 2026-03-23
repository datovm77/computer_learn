# C++ 类与对象的继承 

---

## 📚 目录

1. 继承的基本概念

2. 继承的基本语法

3. 三种继承方式详解

4. 继承中的构造与析构

5. 继承中的成员访问

6. 多继承

7. 菱形继承与虚继承

8. 继承中的多态基础

9. 实战案例

---

## 1. 继承的基本概念

### 🎯 什么是继承？

**继承**是面向对象编程（OOP）的三大特性之一（封装、继承、多态）。它允许我们基于一个已存在的类来创建新的类。

**生活中的例子：**

- 🐶 动物 → 狗 → 哈士奇

- 🚗 交通工具 → 汽车 → 轿车/SUV

- 👨 人 → 学生 → 大学生

### 🔑 核心术语

| 术语   | 说明           | 别名       |
| ------ | -------------- | ---------- |
| 基类   | 被继承的类     | 父类、超类 |
| 派生类 | 继承而来的新类 | 子类       |

### ✅ 继承的优点

1. **代码复用**：避免重复编写相同的代码

2. **扩展性好**：可以在基类基础上添加新功能

3. **层次清晰**：体现类之间的关系

4. **维护方便**：修改基类，派生类自动继承更改

---

## 2. 继承的基本语法

### 📝 语法格式

```cpp
class 派生类名 : 继承方式 基类名 {
    // 派生类新增的成员
};
```

### 🌟 第一个继承示例

```cpp
#include <iostream>
using namespace std;

// 基类：动物
class Animal {
public:
    string name;
    int age;
    
    void eat() {
        cout << name << " 正在吃东西" << endl;
    }
    
    void sleep() {
        cout << name << " 正在睡觉" << endl;
    }
};

// 派生类：狗（继承自动物）
class Dog : public Animal {
public:
    // 狗特有的行为
    void bark() {
        cout << name << " 汪汪汪！" << endl;
    }
};

// 派生类：猫（继承自动物）
class Cat : public Animal {
public:
    // 猫特有的行为
    void meow() {
        cout << name << " 喵喵喵~" << endl;
    }
};

int main() {
    Dog dog;
    dog.name = "旺财";
    dog.age = 3;
    dog.eat();      // 继承自 Animal
    dog.sleep();    // 继承自 Animal
    dog.bark();     // Dog 独有
    
    cout << "----------" << endl;
    
    Cat cat;
    cat.name = "咪咪";
    cat.age = 2;
    cat.eat();      // 继承自 Animal
    cat.meow();     // Cat 独有
    
    return 0;
}
```

**输出：**

```
旺财 正在吃东西
旺财 正在睡觉
旺财 汪汪汪！
----------
咪咪 正在吃东西
咪咪 喵喵喵~
```

### 💡 关键点

- `Dog` 和 `Cat` 都继承了 `Animal` 的 `name`、`age`、`eat()`、`sleep()`

- 同时它们各自有自己独特的方法 `bark()` 和 `meow()`

- 使用 `public` 继承方式（最常用）

---

## 3. 三种继承方式详解

C++ 提供了三种继承方式：`public`、`protected`、`private`

### 📊 成员访问权限变化表

| 基类成员  | public继承 | protected继承 | private继承 |
| --------- | ---------- | ------------- | ----------- |
| public    | public     | protected     | private     |
| protected | protected  | protected     | private     |
| private   | ❌ 不可访问 | ❌ 不可访问    | ❌ 不可访问  |

> ⚠️ **重要**：无论哪种继承方式，基类的 `private` 成员都**不能**被派生类直接访问！

### 🔍 详细示例

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    int m_A;
protected:
    int m_B;
private:
    int m_C;  // 派生类永远无法直接访问
};

// ============ public 继承 ============
class Son1 : public Base {
public:
    void func() {
        m_A = 10;   // 可访问，仍是 public
        m_B = 20;   // 可访问，仍是 protected
        // m_C = 30; // ❌ 错误！private 成员不可访问
    }
};

// ============ protected 继承 ============
class Son2 : protected Base {
public:
    void func() {
        m_A = 10;   // 可访问，变为 protected
        m_B = 20;   // 可访问，仍是 protected
        // m_C = 30; // ❌ 错误！private 成员不可访问
    }
};

// ============ private 继承 ============
class Son3 : private Base {
public:
    void func() {
        m_A = 10;   // 可访问，变为 private
        m_B = 20;   // 可访问，变为 private
        // m_C = 30; // ❌ 错误！private 成员不可访问
    }
};

int main() {
    Son1 s1;
    s1.m_A = 100;    // ✅ public成员，类外可访问
    // s1.m_B = 200; // ❌ protected成员，类外不可访问
    
    Son2 s2;
    // s2.m_A = 100; // ❌ 变成protected，类外不可访问
    
    Son3 s3;
    // s3.m_A = 100; // ❌ 变成private，类外不可访问
    
    return 0;
}
```

### 🎨 图解记忆法

```
基类成员权限
                 ↓
    ┌────────────┬────────────┬────────────┐
    │   public   │ protected  │  private   │
    └────────────┴────────────┴────────────┘
           ↓            ↓            ↓
┌──────────────────────────────────────────────┐
│  public 继承:    public    protected    ❌    │
│  protected继承:  protected protected    ❌    │
│  private继承:    private   private      ❌    │
└──────────────────────────────────────────────┘
```

### 📌 使用建议

- **`public` 继承**：最常用，表示 "is-a" 关系（狗是动物）

- **`protected` 继承**：较少用，限制对外接口

- **`private` 继承**：很少用，表示 "用...实现" 关系

---

## 4. 继承中的构造与析构

### 🔄 构造与析构的顺序

当创建派生类对象时：

1. **先**调用基类构造函数

2. **再**调用派生类构造函数

3. **先**调用派生类析构函数

4. **再**调用基类析构函数

> 💡 记忆口诀：**构造从父到子，析构从子到父**（像穿衣服和脱衣服）

### 📝 示例代码

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    Base() {
        cout << "Base 构造函数" << endl;
    }
    ~Base() {
        cout << "Base 析构函数" << endl;
    }
};

class Son : public Base {
public:
    Son() {
        cout << "Son 构造函数" << endl;
    }
    ~Son() {
        cout << "Son 析构函数" << endl;
    }
};

int main() {
    cout << "=== 创建对象 ===" << endl;
    Son son;
    cout << "=== 对象销毁 ===" << endl;
    return 0;
}
```

**输出：**

```
=== 创建对象 ===
Base 构造函数
Son 构造函数
=== 对象销毁 ===
Son 析构函数
Base 析构函数
```

### 🎯 带参数的构造函数

当基类没有默认构造函数时，派生类必须**显式调用**基类的带参构造函数：

```cpp
#include <iostream>
using namespace std;

class Person {
public:
    string m_Name;
    int m_Age;
    
    // 没有默认构造函数！
    Person(string name, int age) {
        m_Name = name;
        m_Age = age;
        cout << "Person 带参构造" << endl;
    }
};

class Student : public Person {
public:
    int m_Score;
    
    // 必须使用初始化列表调用基类构造函数
    Student(string name, int age, int score) 
        : Person(name, age)  // 👈 显式调用基类构造函数
    {
        m_Score = score;
        cout << "Student 带参构造" << endl;
    }
    
    void showInfo() {
        cout << "姓名：" << m_Name 
             << "，年龄：" << m_Age 
             << "，成绩：" << m_Score << endl;
    }
};

int main() {
    Student stu("张三", 18, 95);
    stu.showInfo();
    return 0;
}
```

**输出：**

```
Person 带参构造
Student 带参构造
姓名：张三，年龄：18，成绩：95
```

### ⚠️ 常见错误

```cpp
// ❌ 错误写法
Student(string name, int age, int score) {
    Person(name, age);  // 这只是创建了一个临时对象，不是调用基类构造
    m_Score = score;
}

// ✅ 正确写法（使用初始化列表）
Student(string name, int age, int score) : Person(name, age) {
    m_Score = score;
}
```

---

## 5. 继承中的成员访问

### 5.1 同名成员处理

当派生类和基类有同名成员时：

- **派生类成员**：直接访问

- **基类成员**：加作用域 `Base::`

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    int m_A = 100;
    
    void func() {
        cout << "Base::func()" << endl;
    }
    
    void func(int a) {
        cout << "Base::func(int a) = " << a << endl;
    }
};

class Son : public Base {
public:
    int m_A = 200;  // 同名成员变量
    
    void func() {   // 同名成员函数
        cout << "Son::func()" << endl;
    }
};

int main() {
    Son son;
    
    // --- 访问成员变量 ---
    cout << "Son 的 m_A = " << son.m_A << endl;        // 200（派生类）
    cout << "Base 的 m_A = " << son.Base::m_A << endl; // 100（基类）
    
    cout << "----------" << endl;
    
    // --- 访问成员函数 ---
    son.func();           // Son::func()
    son.Base::func();     // Base::func()
    son.Base::func(10);   // Base::func(int a) = 10
    
    // son.func(10);      // ❌ 错误！Son::func() 隐藏了所有 Base::func
    
    return 0;
}
```

**输出：**

```
Son 的 m_A = 200
Base 的 m_A = 100
----------
Son::func()
Base::func()
Base::func(int a) = 10
```

### ⚠️ 重要概念：隐藏（Hiding）

> 当派生类定义了与基类同名的函数时，基类中**所有同名函数**（包括重载版本）都会被**隐藏**！

```cpp
// 如果想调用基类的重载函数，必须加基类作用域
son.Base::func(10);  // ✅ 正确
son.func(10);        // ❌ 错误，Son中的func()隐藏了Base的所有func
```

### 5.2 同名静态成员

静态成员的访问方式有两种：**通过对象** 或 **通过类名**

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    static int m_A;
    static void func() {
        cout << "Base::func()" << endl;
    }
};
int Base::m_A = 100;

class Son : public Base {
public:
    static int m_A;
    static void func() {
        cout << "Son::func()" << endl;
    }
};
int Son::m_A = 200;

int main() {
    Son son;
    
    // ========= 通过对象访问 =========
    cout << "Son 的 m_A = " << son.m_A << endl;
    cout << "Base 的 m_A = " << son.Base::m_A << endl;
    
    cout << "----------" << endl;
    
    // ========= 通过类名访问 =========
    cout << "Son::m_A = " << Son::m_A << endl;
    cout << "Base::m_A = " << Son::Base::m_A << endl;  // 👈 注意写法
    
    cout << "----------" << endl;
    
    // ========= 静态函数 =========
    Son::func();
    Son::Base::func();
    
    return 0;
}
```

**输出：**

```
Son 的 m_A = 200
Base 的 m_A = 100
----------
Son::m_A = 200
Base::m_A = 100
----------
Son::func()
Base::func()
```

---

## 6. 多继承

C++ 支持一个类同时继承多个基类。

### 📝 语法

```cpp
class 派生类 : 继承方式 基类1, 继承方式 基类2, ... {
    // 类体
};
```

### 🌟 示例

```cpp
#include <iostream>
using namespace std;

class Phone {
public:
    void call() {
        cout << "打电话" << endl;
    }
};

class Camera {
public:
    void takePhoto() {
        cout << "拍照片" << endl;
    }
};

class MP3 {
public:
    void playMusic() {
        cout << "播放音乐" << endl;
    }
};

// 智能手机：同时继承多个功能
class SmartPhone : public Phone, public Camera, public MP3 {
public:
    void surfInternet() {
        cout << "上网冲浪" << endl;
    }
};

int main() {
    SmartPhone phone;
    
    phone.call();        // 来自 Phone
    phone.takePhoto();   // 来自 Camera
    phone.playMusic();   // 来自 MP3
    phone.surfInternet(); // SmartPhone 独有
    
    return 0;
}
```

### ⚠️ 多继承的问题：同名成员二义性

```cpp
class Base1 {
public:
    int m_A = 100;
};

class Base2 {
public:
    int m_A = 200;
};

class Son : public Base1, public Base2 {
};

int main() {
    Son son;
    // cout << son.m_A << endl;  // ❌ 二义性错误！
    
    cout << son.Base1::m_A << endl;  // ✅ 100
    cout << son.Base2::m_A << endl;  // ✅ 200
    
    return 0;
}
```

### 📌 多继承使用建议

> 多继承容易产生二义性问题，实际开发中建议**谨慎使用**。很多语言（如Java、C#）干脆不支持多继承。

---

## 7. 菱形继承与虚继承

### 🔷 什么是菱形继承？

```
Animal
       /      \
     Sheep    Camel
       \      /
        Alpaca（羊驼）
```

羊驼同时继承了羊和骆驼，而羊和骆驼又都继承自动物。

### ❌ 菱形继承的问题

```cpp
#include <iostream>
using namespace std;

class Animal {
public:
    int m_Age;
};

class Sheep : public Animal {};
class Camel : public Animal {};

class Alpaca : public Sheep, public Camel {};

int main() {
    Alpaca alpaca;
    
    // alpaca.m_Age = 10;  // ❌ 二义性！有两份 m_Age
    
    alpaca.Sheep::m_Age = 10;  // 羊的年龄
    alpaca.Camel::m_Age = 20;  // 骆驼的年龄
    
    // 问题：羊驼需要两个年龄吗？浪费资源且不合理！
    cout << "通过Sheep: " << alpaca.Sheep::m_Age << endl;
    cout << "通过Camel: " << alpaca.Camel::m_Age << endl;
    
    return 0;
}
```

### ✅ 解决方案：虚继承

使用 `virtual` 关键字进行虚继承，确保基类只有一份：

```cpp
#include <iostream>
using namespace std;

class Animal {
public:
    int m_Age;
};

// 使用虚继承
class Sheep : virtual public Animal {};
class Camel : virtual public Animal {};

class Alpaca : public Sheep, public Camel {};

int main() {
    Alpaca alpaca;
    
    // 现在只有一份 m_Age，不存在二义性
    alpaca.m_Age = 10;  // ✅ 可以直接访问
    
    alpaca.Sheep::m_Age = 20;
    alpaca.Camel::m_Age = 30;  // 修改的是同一份数据
    
    cout << "m_Age = " << alpaca.m_Age << endl;               // 30
    cout << "通过Sheep: " << alpaca.Sheep::m_Age << endl;     // 30
    cout << "通过Camel: " << alpaca.Camel::m_Age << endl;     // 30
    
    return 0;
}
```

### 🔍 虚继承的原理（了解即可）

虚继承通过**虚基类指针（vbptr）**和**虚基类表（vbtable）**来实现数据共享：

```
            ┌─────────────┐
            │   Animal    │
            │   m_Age     │
            └─────────────┘
                  ↑
         ┌───────┴───────┐
    vbptr│              │vbptr
    ┌────┴────┐    ┌────┴────┐
    │  Sheep  │    │  Camel  │
    └────┬────┘    └────┬────┘
         └───────┬───────┘
            ┌────┴────┐
            │ Alpaca  │
            └─────────┘
```

---

## 8. 继承中的多态基础

### 🎭 什么是多态？

多态允许我们用**基类指针/引用**来操作**派生类对象**，并根据实际对象类型调用对应的函数。

### 📝 虚函数（Virtual Function）

使用 `virtual` 关键字声明的函数：

```cpp
#include <iostream>
using namespace std;

class Animal {
public:
    // 虚函数
    virtual void speak() {
        cout << "动物在叫" << endl;
    }
};

class Dog : public Animal {
public:
    void speak() override {  // override 是可选的，但推荐加上
        cout << "汪汪汪！" << endl;
    }
};

class Cat : public Animal {
public:
    void speak() override {
        cout << "喵喵喵~" << endl;
    }
};

// 使用基类指针的函数
void makeAnimalSpeak(Animal* animal) {
    animal->speak();  // 根据实际类型调用对应的 speak()
}

int main() {
    Dog dog;
    Cat cat;
    
    makeAnimalSpeak(&dog);  // 汪汪汪！
    makeAnimalSpeak(&cat);  // 喵喵喵~
    
    // 使用基类指针
    Animal* ptr = &dog;
    ptr->speak();  // 汪汪汪！
    
    ptr = &cat;
    ptr->speak();  // 喵喵喵~
    
    return 0;
}
```

### 🆚 有无 virtual 的区别

```cpp
// ❌ 没有 virtual 时（静态绑定，编译时决定）
class Animal {
public:
    void speak() { cout << "动物在叫" << endl; }
};

Animal* ptr = new Dog();
ptr->speak();  // 输出：动物在叫（总是调用 Animal::speak）
```

```cpp
// ✅ 有 virtual 时（动态绑定，运行时决定）
class Animal {
public:
    virtual void speak() { cout << "动物在叫" << endl; }
};

Animal* ptr = new Dog();
ptr->speak();  // 输出：汪汪汪！（根据实际类型调用）
```

### 🔐 纯虚函数与抽象类

当基类的函数没有实现意义时，可以声明为**纯虚函数**：

```cpp
class Shape {
public:
    // 纯虚函数（= 0 表示没有实现）
    virtual double getArea() = 0;
    virtual double getPerimeter() = 0;
};

// 包含纯虚函数的类是抽象类，不能实例化
// Shape shape;  // ❌ 错误！

class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    
    // 必须实现所有纯虚函数
    double getArea() override {
        return 3.14159 * radius * radius;
    }
    
    double getPerimeter() override {
        return 2 * 3.14159 * radius;
    }
};

class Rectangle : public Shape {
private:
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    
    double getArea() override {
        return width * height;
    }
    
    double getPerimeter() override {
        return 2 * (width + height);
    }
};
```

### 🗑️ 虚析构函数

> **重要**：当使用基类指针删除派生类对象时，基类的析构函数必须是虚函数！

```cpp
class Animal {
public:
    Animal() { cout << "Animal 构造" << endl; }
    
    // ✅ 虚析构函数
    virtual ~Animal() { cout << "Animal 析构" << endl; }
};

class Dog : public Animal {
public:
    Dog() { 
        cout << "Dog 构造" << endl;
        m_Data = new int[100];  // 申请堆内存
    }
    
    ~Dog() { 
        cout << "Dog 析构" << endl;
        delete[] m_Data;  // 释放堆内存
    }
private:
    int* m_Data;
};

int main() {
    Animal* animal = new Dog();
    delete animal;  // 如果没有虚析构，Dog的析构不会被调用 → 内存泄漏！
    return 0;
}
```

**正确输出（有虚析构）：**

```
Animal 构造
Dog 构造
Dog 析构
Animal 析构
```

---

## 9. 实战案例

### 📦 案例：员工管理系统

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

// ============ 抽象基类：员工 ============
class Employee {
protected:
    string m_Name;
    int m_Id;
    double m_BaseSalary;
    
public:
    Employee(string name, int id, double salary)
        : m_Name(name), m_Id(id), m_BaseSalary(salary) {}
    
    virtual ~Employee() {}  // 虚析构函数
    
    // 纯虚函数：计算工资
    virtual double calculateSalary() = 0;
    
    // 纯虚函数：显示信息
    virtual void showInfo() = 0;
    
    string getName() { return m_Name; }
};

// ============ 普通员工 ============
class RegularEmployee : public Employee {
private:
    int m_WorkDays;  // 出勤天数
    
public:
    RegularEmployee(string name, int id, double salary, int days)
        : Employee(name, id, salary), m_WorkDays(days) {}
    
    double calculateSalary() override {
        // 日薪 * 出勤天数
        return (m_BaseSalary / 22) * m_WorkDays;
    }
    
    void showInfo() override {
        cout << "【普通员工】" << endl;
        cout << "  姓名：" << m_Name << endl;
        cout << "  工号：" << m_Id << endl;
        cout << "  出勤：" << m_WorkDays << " 天" << endl;
        cout << "  工资：" << calculateSalary() << " 元" << endl;
    }
};

// ============ 销售员工 ============
class SalesEmployee : public Employee {
private:
    double m_Sales;       // 销售额
    double m_Commission;  // 提成比例
    
public:
    SalesEmployee(string name, int id, double salary, double sales, double comm)
        : Employee(name, id, salary), m_Sales(sales), m_Commission(comm) {}
    
    double calculateSalary() override {
        // 底薪 + 销售提成
        return m_BaseSalary + m_Sales * m_Commission;
    }
    
    void showInfo() override {
        cout << "【销售员工】" << endl;
        cout << "  姓名：" << m_Name << endl;
        cout << "  工号：" << m_Id << endl;
        cout << "  销售额：" << m_Sales << " 元" << endl;
        cout << "  提成比例：" << m_Commission * 100 << "%" << endl;
        cout << "  工资：" << calculateSalary() << " 元" << endl;
    }
};

// ============ 经理 ============
class Manager : public Employee {
private:
    double m_Bonus;  // 奖金
    
public:
    Manager(string name, int id, double salary, double bonus)
        : Employee(name, id, salary), m_Bonus(bonus) {}
    
    double calculateSalary() override {
        return m_BaseSalary + m_Bonus;
    }
    
    void showInfo() override {
        cout << "【经理】" << endl;
        cout << "  姓名：" << m_Name << endl;
        cout << "  工号：" << m_Id << endl;
        cout << "  底薪：" << m_BaseSalary << " 元" << endl;
        cout << "  奖金：" << m_Bonus << " 元" << endl;
        cout << "  工资：" << calculateSalary() << " 元" << endl;
    }
};

// ============ 主程序 ============
int main() {
    vector<Employee*> employees;
    
    // 添加不同类型的员工
    employees.push_back(new RegularEmployee("张三", 1001, 6000, 20));
    employees.push_back(new RegularEmployee("李四", 1002, 5500, 22));
    employees.push_back(new SalesEmployee("王五", 2001, 4000, 50000, 0.05));
    employees.push_back(new Manager("赵六", 3001, 15000, 5000));
    
    cout << "========== 员工信息 ==========" << endl << endl;
    
    double totalSalary = 0;
    for (Employee* emp : employees) {
        emp->showInfo();  // 多态调用
        totalSalary += emp->calculateSalary();
        cout << endl;
    }
    
    cout << "==============================" << endl;
    cout << "本月工资总额：" << totalSalary << " 元" << endl;
    
    // 释放内存
    for (Employee* emp : employees) {
        delete emp;
    }
    
    return 0;
}
```

**输出：**

```
========== 员工信息 ==========

【普通员工】
  姓名：张三
  工号：1001
  出勤：20 天
  工资：5454.55 元

【普通员工】
  姓名：李四
  工号：1002
  出勤：22 天
  工资：5500 元

【销售员工】
  姓名：王五
  工号：2001
  销售额：50000 元
  提成比例：5%
  工资：6500 元

【经理】
  姓名：赵六
  工号：3001
  底薪：15000 元
  奖金：5000 元
  工资：20000 元

==============================
本月工资总额：37454.5 元
```

---

## 📝 总结

| 知识点       | 要点                                       |
| ------------ | ------------------------------------------ |
| 继承语法     | class 派生类 : 继承方式 基类               |
| 三种继承方式 | public（最常用）、protected、private       |
| 构造析构顺序 | 构造：父→子；析构：子→父                   |
| 同名成员     | 派生类优先，访问基类需加 Base::            |
| 多继承       | 语法简单但易产生二义性                     |
| 菱形继承     | 使用虚继承 virtual 解决                    |
| 多态         | 虚函数 virtual，允许基类指针调用派生类方法 |
| 抽象类       | 包含纯虚函数 = 0，不能实例化               |
| 虚析构       | 基类指针删除派生类对象时必须使用           |
