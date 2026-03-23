# C++ 多态详解教程

## 一、什么是多态？

**多态**（Polymorphism）是面向对象编程的三大特性之一（封装、继承、多态），它的字面意思是"多种形态"。在C++中，多态允许我们用同一个接口调用不同的实现，使程序更加灵活和可扩展。

### 1.1 多态的两种类型

- **静态多态**（编译时多态）：通过函数重载和运算符重载实现

- **动态多态**（运行时多态）：通过继承和虚函数实现 ← **本教程重点**

---

## 二、为什么需要多态？

让我们通过一个实际场景来理解：

```cpp
// 不使用多态的传统做法
class Animal {
public:
    void speak() {
        cout << "动物在叫" << endl;
    }
};

class Cat : public Animal {
public:
    void speak() {
        cout << "喵喵喵" << endl;
    }
};

class Dog : public Animal {
public:
    void speak() {
        cout << "汪汪汪" << endl;
    }
};

// 调用函数
void doSpeak(Animal &animal) {
    animal.speak();  // 问题：这里只会调用Animal的speak()
}

int main() {
    Cat cat;
    Dog dog;
    
    doSpeak(cat);  // 输出：动物在叫 （我们期望：喵喵喵）
    doSpeak(dog);  // 输出：动物在叫 （我们期望：汪汪汪）
    
    return 0;
}
```

**问题出现了**：尽管我们传入的是Cat和Dog对象，但调用的却是父类Animal的speak()方法。这不是我们想要的！

---

## 三、多态的实现 - 虚函数

### 3.1 使用 `virtual` 关键字

要实现动态多态，只需在父类的函数前加上 `virtual` 关键字：

```cpp
class Animal {
public:
    // 将speak()声明为虚函数
    virtual void speak() {
        cout << "动物在叫" << endl;
    }
};

class Cat : public Animal {
public:
    // 子类重写（override）虚函数
    void speak() {
        cout << "喵喵喵" << endl;
    }
};

class Dog : public Animal {
public:
    void speak() {
        cout << "汪汪汪" << endl;
    }
};

// 调用函数（使用父类引用或指针）
void doSpeak(Animal &animal) {
    animal.speak();  // 现在会调用实际对象的speak()
}

int main() {
    Cat cat;
    Dog dog;
    
    doSpeak(cat);  // 输出：喵喵喵 ✓
    doSpeak(dog);  // 输出：汪汪汪 ✓
    
    return 0;
}
```

### 3.2 多态的两个必要条件

1. **有继承关系**

2. **子类重写父类的虚函数**（重写 = 函数返回值、函数名、参数列表完全相同）

### 3.3 多态的使用方式

- **父类指针**或**父类引用** 指向子类对象

```cpp
Animal *animal1 = new Cat();    // 父类指针
Animal &animal2 = dog;          // 父类引用

animal1->speak();  // 喵喵喵
animal2.speak();   // 汪汪汪
```

---

## 四、多态的底层原理

### 4.1 虚函数表（VTable）

当类中有虚函数时，编译器会为这个类创建一个**虚函数表**（Virtual Table，简称VTable）：

```cpp
class Animal {
public:
    virtual void speak() { cout << "动物在叫" << endl; }
    virtual void eat() { cout << "动物在吃" << endl; }
};
```

**内存布局**：

```
Animal对象:
+------------------+
| vptr (虚函数指针) |  ← 指向虚函数表
+------------------+
| 其他成员变量      |
+------------------+

虚函数表:
+------------------+
| &Animal::speak   |
+------------------+
| &Animal::eat     |
+------------------+
```

### 4.2 子类的虚函数表

```cpp
class Cat : public Animal {
public:
    void speak() { cout << "喵喵喵" << endl; }  // 重写
    // eat()未重写，使用父类的
};
```

**Cat的虚函数表**：

```
+------------------+
| &Cat::speak      |  ← 被重写的函数地址
+------------------+
| &Animal::eat     |  ← 未重写，保留父类地址
+------------------+
```

### 4.3 动态绑定过程

```cpp
Animal *animal = new Cat();
animal->speak();
```

执行流程：

1. 通过对象的 `vptr` 找到虚函数表

2. 在虚函数表中查找 `speak()` 的地址

3. 调用找到的函数（Cat::speak）

这就是**运行时多态** - 在程序运行时才确定调用哪个函数！

---

## 五、纯虚函数与抽象类

### 5.1 纯虚函数

如果父类中的虚函数没有意义，可以定义为**纯虚函数**：

```cpp
class AbstractCalculator {
public:
    int m_A;
    int m_B;
    
    // 纯虚函数：在函数后加 = 0
    virtual int calculate() = 0;
};
```

### 5.2 抽象类

包含纯虚函数的类称为**抽象类**，抽象类有两个特点：

1. **无法实例化对象**

2. **子类必须重写纯虚函数**，否则子类也是抽象类

```cpp
// AbstractCalculator calculator; // 错误！抽象类不能实例化

class AddCalculator : public AbstractCalculator {
public:
    int calculate() {  // 必须重写
        return m_A + m_B;
    }
};

class SubCalculator : public AbstractCalculator {
public:
    int calculate() {
        return m_A - m_B;
    }
};
```

### 5.3 实际应用示例

```cpp
// 使用多态实现计算器
AbstractCalculator* calculator = nullptr;

calculator = new AddCalculator();
calculator->m_A = 10;
calculator->m_B = 20;
cout << calculator->calculate() << endl;  // 30
delete calculator;

calculator = new SubCalculator();
calculator->m_A = 10;
calculator->m_B = 20;
cout << calculator->calculate() << endl;  // -10
delete calculator;
```

**优势**：

- 代码组织清晰，结构性强

- 可扩展性好（添加新运算只需新增子类）

- 符合开闭原则（对扩展开放，对修改封闭）

---

## 六、虚析构函数与纯虚析构函数

### 6.1 为什么需要虚析构？

```cpp
class Animal {
public:
    Animal() { cout << "Animal构造" << endl; }
    ~Animal() { cout << "Animal析构" << endl; }
};

class Cat : public Animal {
public:
    string *m_Name;
    
    Cat(string name) {
        m_Name = new string(name);
        cout << "Cat构造" << endl;
    }
    
    ~Cat() {
        if (m_Name != nullptr) {
            delete m_Name;
            m_Name = nullptr;
        }
        cout << "Cat析构" << endl;
    }
};

int main() {
    Animal *animal = new Cat("Tom");
    delete animal;  // 问题：只调用了Animal析构，没有调用Cat析构！
    
    return 0;
}
```

**输出**：

```
Animal构造
Cat构造
Animal析构    ← 只析构了父类，内存泄漏！
```

### 6.2 解决方案：虚析构函数

```cpp
class Animal {
public:
    Animal() { cout << "Animal构造" << endl; }
    
    // 将析构函数声明为虚函数
    virtual ~Animal() { 
        cout << "Animal析构" << endl; 
    }
};

// Cat类不变

int main() {
    Animal *animal = new Cat("Tom");
    delete animal;  // 现在会正确调用两个析构函数
    
    return 0;
}
```

**输出**：

```
Animal构造
Cat构造
Cat析构      ← 正确调用
Animal析构
```

### 6.3 纯虚析构函数

```cpp
class Animal {
public:
    // 纯虚析构（需要声明也需要实现）
    virtual ~Animal() = 0;
};

// 纯虚析构必须要有实现（与纯虚函数不同）
Animal::~Animal() {
    cout << "Animal纯虚析构" << endl;
}

class Cat : public Animal {
public:
    ~Cat() {
        cout << "Cat析构" << endl;
    }
};
```

**记忆要点**：

- 如果有子类对象指向堆区，父类析构函数必须是虚析构或纯虚析构

- 纯虚析构函数需要提供实现（因为子类析构时会调用父类析构）

---

## 七、多态案例：组装电脑

### 7.1 需求分析

电脑由CPU、显卡、内存条组成，不同厂商生产不同的零件，我们需要组装不同配置的电脑。

### 7.2 代码实现

```cpp
#include <iostream>
using namespace std;

// ========== 抽象零件类 ==========
class CPU {
public:
    virtual void calculate() = 0;  // 纯虚函数
};

class VideoCard {
public:
    virtual void display() = 0;
};

class Memory {
public:
    virtual void storage() = 0;
};

// ========== Intel厂商 ==========
class IntelCPU : public CPU {
public:
    void calculate() {
        cout << "Intel CPU开始计算" << endl;
    }
};

class IntelVideoCard : public VideoCard {
public:
    void display() {
        cout << "Intel显卡开始显示" << endl;
    }
};

class IntelMemory : public Memory {
public:
    void storage() {
        cout << "Intel内存条开始存储" << endl;
    }
};

// ========== Lenovo厂商 ==========
class LenovoCPU : public CPU {
public:
    void calculate() {
        cout << "Lenovo CPU开始计算" << endl;
    }
};

class LenovoVideoCard : public VideoCard {
public:
    void display() {
        cout << "Lenovo显卡开始显示" << endl;
    }
};

class LenovoMemory : public Memory {
public:
    void storage() {
        cout << "Lenovo内存条开始存储" << endl;
    }
};

// ========== 电脑类 ==========
class Computer {
public:
    CPU *m_cpu;
    VideoCard *m_vc;
    Memory *m_mem;
    
    Computer(CPU *cpu, VideoCard *vc, Memory *mem) {
        m_cpu = cpu;
        m_vc = vc;
        m_mem = mem;
    }
    
    void work() {
        m_cpu->calculate();
        m_vc->display();
        m_mem->storage();
    }
    
    ~Computer() {
        // 释放零件
        if (m_cpu != nullptr) {
            delete m_cpu;
            m_cpu = nullptr;
        }
        if (m_vc != nullptr) {
            delete m_vc;
            m_vc = nullptr;
        }
        if (m_mem != nullptr) {
            delete m_mem;
            m_mem = nullptr;
        }
    }
};

// ========== 测试 ==========
int main() {
    cout << "========== 第一台电脑 ==========" << endl;
    Computer *computer1 = new Computer(
        new IntelCPU(), 
        new IntelVideoCard(), 
        new IntelMemory()
    );
    computer1->work();
    delete computer1;
    
    cout << "\n========== 第二台电脑 ==========" << endl;
    Computer *computer2 = new Computer(
        new LenovoCPU(), 
        new LenovoVideoCard(), 
        new LenovoMemory()
    );
    computer2->work();
    delete computer2;
    
    cout << "\n========== 混合配置电脑 ==========" << endl;
    Computer *computer3 = new Computer(
        new IntelCPU(), 
        new LenovoVideoCard(), 
        new IntelMemory()
    );
    computer3->work();
    delete computer3;
    
    return 0;
}
```

**输出**：

```
========== 第一台电脑 ==========
Intel CPU开始计算
Intel显卡开始显示
Intel内存条开始存储

========== 第二台电脑 ==========
Lenovo CPU开始计算
Lenovo显卡开始显示
Lenovo内存条开始存储

========== 混合配置电脑 ==========
Intel CPU开始计算
Lenovo显卡开始显示
Intel内存条开始存储
```

### 7.3 设计优势

1. **扩展性强**：添加新厂商（如AMD）只需新增子类，不修改现有代码

2. **灵活组合**：可以自由搭配不同厂商的零件

3. **代码复用**：Computer类不依赖具体厂商，高度抽象

---

## 八、重要知识点总结

### 8.1 关键概念对比

| 概念     | 说明                 | 语法                     |
| -------- | -------------------- | ------------------------ |
| 虚函数   | 可以被子类重写的函数 | virtual void func()      |
| 纯虚函数 | 必须被子类重写的函数 | virtual void func() = 0  |
| 抽象类   | 包含纯虚函数的类     | 不能实例化               |
| 虚析构   | 确保子类正确析构     | virtual ~ClassName()     |
| 纯虚析构 | 抽象类+虚析构        | virtual ~ClassName() = 0 |

### 8.2 多态使用原则

✅ **DO（应该做）**：

- 父类指针/引用指向子类对象

- 在基类中声明虚函数

- 有堆区数据时使用虚析构

❌ **DON'T（不应该做）**：

- 不要将构造函数声明为虚函数

- 不要在构造/析构函数中调用虚函数

- 普通成员函数不需要都声明为虚函数（有性能开销）

### 8.3 虚函数性能考虑

- 虚函数调用比普通函数慢一点（需要查表）

- 每个对象多占用一个指针的空间（vptr）

- 但这点性能损失换来的灵活性是值得的

---

## 九、进阶主题

### 9.1 override关键字（C++11）

明确表明重写虚函数，编译器会检查：

```cpp
class Animal {
public:
    virtual void speak() { }
};

class Cat : public Animal {
public:
    // 使用override关键字，防止拼写错误
    void speak() override {  // 如果父类没有此虚函数，编译错误
        cout << "喵喵喵" << endl;
    }
};
```

### 9.2 final关键字（C++11）

防止类被继承或虚函数被重写：

```cpp
class Animal {
public:
    virtual void speak() final {  // 不允许子类重写
        cout << "动物在叫" << endl;
    }
};

class Dog final : public Animal {  // 不允许再被继承
    // ...
};
```

### 9.3 多重继承中的虚函数

```cpp
class A {
public:
    virtual void func() { cout << "A" << endl; }
};

class B {
public:
    virtual void func() { cout << "B" << endl; }
};

class C : public A, public B {
public:
    void func() override {  // 同时重写A和B的func
        cout << "C" << endl;
    }
};
```

---

## 十、常见错误与调试

### 错误1：忘记添加virtual

```cpp
class Animal {
public:
    void speak() { }  // ❌ 没有virtual
};

class Cat : public Animal {
public:
    void speak() { }
};

Animal *a = new Cat();
a->speak();  // 只会调用Animal::speak()
```

### 错误2：参数不匹配

```cpp
class Animal {
public:
    virtual void speak(int x) { }
};

class Cat : public Animal {
public:
    void speak() { }  // ❌ 参数不同，不是重写，是重载
};
```

### 错误3：忘记虚析构

```cpp
class Animal {
public:
    ~Animal() { }  // ❌ 应该是 virtual ~Animal()
};

Animal *a = new Cat();
delete a;  // 只析构Animal，Cat的资源泄漏
```

---

## 十一、练习题

### 基础练习

创建一个形状类Shape，包含计算面积的纯虚函数，然后创建Circle（圆形）和Rectangle（矩形）子类。

### 进阶练习

设计一个饮料制作系统：

- 抽象类：Beverage（煮水→冲泡→倒杯子→加料）

- 子类：Coffee（冲泡咖啡粉、加糖）

- 子类：Tea（冲泡茶叶、加柠檬）

提示：使用模板方法模式和多态。

---

## 总结

多态是C++最强大的特性之一，它让我们能够编写更加灵活、可扩展的代码。记住三个关键点：

1. **继承** + **虚函数** = **多态**

2. 父类指针/引用 指向子类对象

3. 有堆区数据就用虚析构

多态的本质是**接口重用**和**实现可替换**，这是面向对象设计的精髓。继续练习，你会越来越熟练！加油！🚀