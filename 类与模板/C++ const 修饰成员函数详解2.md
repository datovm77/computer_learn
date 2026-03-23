# C++ `const` 修饰成员函数详解

## 一、从问题出发：为什么需要常函数？

先看一个会报错的场景：

```cpp
#include <iostream>
#include <string>

class Student {
private:
    std::string name;
    int score;

public:
    Student(std::string n, int s) : name(n), score(s) {}

    // 普通成员函数
    void printInfo() {
        std::cout << "Name: " << name << ", Score: " << score << std::endl;
    }
};

// 全局函数：打印学生信息
void displayStudent(const Student& s) {
    s.printInfo();  // ❌ 编译错误！
}

int main() {
    Student stu("Alice", 90);
    displayStudent(stu);
    return 0;
}
```

**编译错误信息：**
```
error: passing 'const Student' as 'this' argument discards qualifiers
```

### 为什么会报错？

```
┌─────────────────────────────────────────────────────────────────┐
│  const Student& s  ────>  s 是"只读"的                           │
│                                                                 │
│  s.printInfo()  ────>  printInfo() 是普通函数                    │
│                         编译器认为它"可能"会修改对象                │
│                                                                 │
│  矛盾：只读对象 vs 可能修改的函数  ────>  编译器拒绝！                 │
└─────────────────────────────────────────────────────────────────┘
```

**解决方案**：告诉编译器"这个函数不会修改对象"——这就是**常函数**！

---

## 二、常函数的语法与本质

### 2.1 基本语法

```cpp
class 类名 {
public:
    返回类型 函数名(参数列表) const {   // ← const 放在函数声明之后
        // 函数体
    }
};
```

### 2.2 修复上面的例子

```cpp
#include <iostream>
#include <string>

class Student {
private:
    std::string name;
    int score;

public:
    Student(std::string n, int s) : name(n), score(s) {}

    // ✅ 常函数：const 放在参数列表之后
    void printInfo() const {
        std::cout << "Name: " << name << ", Score: " << score << std::endl;
    }
    
    // 普通函数
    void setScore(int s) {
        score = s;
    }
};

// 现在可以正常工作了！
void displayStudent(const Student& s) {
    s.printInfo();  // ✅ 常对象调用常函数，编译通过
    // s.setScore(100);  // ❌ 常对象不能调用普通函数
}

int main() {
    Student stu("Alice", 90);
    displayStudent(stu);
    
    std::cout << "\n--- 普通对象可以调用所有函数 ---\n";
    stu.printInfo();    // ✅ 普通对象可以调用常函数
    stu.setScore(95);   // ✅ 普通对象可以调用普通函数
    
    return 0;
}
```

**输出：**
```
Name: Alice, Score: 90

--- 普通对象可以调用所有函数 ---
Name: Alice, Score: 95
```

---

## 三、常函数的本质：`this` 指针的变化

### 3.1 普通成员函数的 `this` 指针

```cpp
class Example {
public:
    void normalFunc() {
        // this 的类型是: Example* const this
        // 可以修改成员，因为 this 指向的是非 const 对象
    }
};
```

### 3.2 常函数的 `this` 指针

```cpp
class Example {
public:
    void constFunc() const {
        // this 的类型是: const Example* const this
        // 不能修改成员，因为 this 指向的是 const 对象
    }
};
```

### 3.3 图解对比

```
┌────────────────────────────────────────────────────────────────────┐
│                    普通成员函数                                      │
├────────────────────────────────────────────────────────────────────┤
│  void func() { }                                                    │
│                                                                     │
│  隐式的 this 类型:  ClassName* const this                           │
│                                                                     │
│       ┌──────────────┐                                              │
│  this ──→  [  对象   ]  ←── 可以读写                                │
│              [成员变量]                                              │
│       └──────────────┘                                              │
└────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────┐
│                    常函数 (const member function)                   │
├────────────────────────────────────────────────────────────────────┤
│  void func() const { }                                              │
│                                                                     │
│  隐式的 this 类型:  const ClassName* const this                     │
│                                                                     │
│       ┌──────────────┐                                              │
│  this ──→  [  对象   ]  ←── 只能读取，不能写入！                     │
│              [成员变量]                                              │
│       └──────────────┘                                              │
└────────────────────────────────────────────────────────────────────┘
```

---

## 四、常函数内不能修改成员属性

### 4.1 错误示例

```cpp
#include <iostream>

class Counter {
private:
    int value;

public:
    Counter(int v) : value(v) {}

    // ❌ 这个常函数试图修改成员变量
    void increment() const {
        value++;  // 编译错误！
        std::cout << "Value: " << value << std::endl;
    }
};

int main() {
    Counter c(10);
    c.increment();
    return 0;
}
```

**编译错误：**
```
error: increment of member 'Counter::value' in read-only object
```

### 4.2 正确示例

```cpp
#include <iostream>

class Counter {
private:
    int value;

public:
    Counter(int v) : value(v) {}

    // ✅ 常函数只读取，不修改
    int getValue() const {
        return value;  // ✅ 读取成员变量是允许的
    }
    
    // ✅ 普通函数可以修改
    void increment() {
        value++;
    }
};

int main() {
    Counter c(10);
    
    std::cout << "初始值: " << c.getValue() << std::endl;
    c.increment();
    std::cout << "增加后: " << c.getValue() << std::endl;
    
    return 0;
}
```

**输出：**
```
初始值: 10
增加后: 11
```

---

## 五、`mutable` 关键字：常函数的例外

有些场景下，我们希望在常函数中仍能修改某些成员变量（比如用于缓存、计数等），这时可以用 `mutable`。

### 5.1 基本概念

```
┌─────────────────────────────────────────────────────────────────┐
│  mutable 修饰的成员变量                                          │
│                                                                 │
│  特点：即使在 const 对象或 const 成员函数中，也可以被修改        │
│                                                                 │
│  典型用途：                                                      │
│  • 缓存计算结果                                                  │
│  • 访问计数器                                                    │
│  • 延迟初始化                                                    │
│  • 调试/日志状态                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 实例：缓存计算结果

```cpp
#include <iostream>
#include <cmath>

class Circle {
private:
    double radius;
    
    // mutable 允许在常函数中修改
    mutable double cachedArea;
    mutable bool areaCached;

public:
    Circle(double r) : radius(r), cachedArea(0), areaCached(false) {}

    // 常函数，但可以修改 mutable 成员
    double getArea() const {
        if (!areaCached) {
            cachedArea = M_PI * radius * radius;  // ✅ 修改 mutable 成员
            areaCached = true;                     // ✅ 修改 mutable 成员
            std::cout << "[计算并缓存面积]" << std::endl;
        } else {
            std::cout << "[使用缓存的面积]" << std::endl;
        }
        return cachedArea;
    }
    
    // 修改半径需要清除缓存
    void setRadius(double r) {
        radius = r;
        areaCached = false;  // 清除缓存
    }
};

int main() {
    const Circle circle(5.0);  // 常对象！
    
    std::cout << "第一次获取面积: " << circle.getArea() << std::endl;
    std::cout << "第二次获取面积: " << circle.getArea() << std::endl;
    
    return 0;
}
```

**输出：**
```
第一次获取面积: [计算并缓存面积]
78.5398
第二次获取面积: [使用缓存的面积]
78.5398
```

### 5.3 实例：访问计数器

```cpp
#include <iostream>

class Data {
private:
    int value;
    mutable int accessCount;  // 记录访问次数

public:
    Data(int v) : value(v), accessCount(0) {}

    // 常函数：获取值的同时记录访问次数
    int getValue() const {
        accessCount++;  // ✅ 修改 mutable 成员
        return value;
    }
    
    // 获取访问次数
    int getAccessCount() const {
        return accessCount;
    }
    
    // 修改值
    void setValue(int v) {
        value = v;
    }
};

int main() {
    const Data data(42);  // 常对象
    
    // 多次读取
    for (int i = 0; i < 5; i++) {
        std::cout << "读取: " << data.getValue() << std::endl;
    }
    
    std::cout << "\n总访问次数: " << data.getAccessCount() << std::endl;
    
    // data.setValue(100);  // ❌ 常对象不能调用非常函数
    
    return 0;
}
```

**输出：**
```
读取: 42
读取: 42
读取: 42
读取: 42
读取: 42

总访问次数: 5
```

---

## 六、常对象（Const Objects）

### 6.1 什么是常对象

```cpp
const ClassName obj;      // 常对象
const ClassName& ref;     // 常引用
const ClassName* ptr;     // 指向常对象的指针
```

### 6.2 常对象的特点

```
┌─────────────────────────────────────────────────────────────────┐
│                        常对象的限制                              │
├─────────────────────────────────────────────────────────────────┤
│  ✅ 可以调用：常函数（const member functions）                   │
│  ❌ 不能调用：普通成员函数（non-const member functions）         │
│  ❌ 不能修改：非 mutable 的成员变量                              │
│  ✅ 可以读取：所有成员变量                                       │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 综合示例

```cpp
#include <iostream>
#include <string>

class Person {
private:
    std::string name;
    int age;
    mutable int greetCount;  // mutable: 可在常函数中修改

public:
    Person(std::string n, int a) : name(n), age(a), greetCount(0) {}

    // ✅ 常函数
    std::string getName() const {
        return name;
    }

    // ✅ 常函数
    int getAge() const {
        return age;
    }
    
    // ✅ 常函数，但可以修改 mutable 成员
    void greet() const {
        greetCount++;
        std::cout << "Hello! I'm " << name << ", " << age << " years old." << std::endl;
    }
    
    // ✅ 常函数
    int getGreetCount() const {
        return greetCount;
    }

    // ❌ 普通函数（非常函数）
    void setAge(int a) {
        age = a;
    }
    
    // ❌ 普通函数
    void haveBirthday() {
        age++;
        std::cout << "Happy Birthday! Now " << age << " years old." << std::endl;
    }
};

// 函数接收常引用
void showPerson(const Person& p) {
    std::cout << "=== 展示人物信息 ===" << std::endl;
    std::cout << "姓名: " << p.getName() << std::endl;      // ✅ 调用常函数
    std::cout << "年龄: " << p.getAge() << std::endl;        // ✅ 调用常函数
    p.greet();                                               // ✅ 调用常函数
    
    // p.setAge(20);      // ❌ 编译错误！常对象不能调用非常函数
    // p.haveBirthday();  // ❌ 编译错误！
}

int main() {
    std::cout << "===== 普通对象 =====" << std::endl;
    Person alice("Alice", 25);
    
    alice.greet();           // ✅ 普通对象可以调用常函数
    alice.haveBirthday();    // ✅ 普通对象可以调用普通函数
    alice.setAge(27);        // ✅ 普通对象可以调用普通函数
    std::cout << "问候次数: " << alice.getGreetCount() << std::endl;
    
    std::cout << "\n===== 常对象 =====" << std::endl;
    const Person bob("Bob", 30);  // 常对象
    
    std::cout << "姓名: " << bob.getName() << std::endl;    // ✅ 调用常函数
    bob.greet();                                            // ✅ 调用常函数
    std::cout << "问候次数: " << bob.getGreetCount() << std::endl;
    
    // bob.haveBirthday();  // ❌ 编译错误！常对象不能调用普通函数
    // bob.setAge(35);      // ❌ 编译错误！
    
    std::cout << "\n===== 常引用参数 =====" << std::endl;
    showPerson(alice);
    
    return 0;
}
```

**输出：**

```
===== 普通对象 =====
Hello! I'm Alice, 25 years old.
Happy Birthday! Now 26 years old.
问候次数: 1

===== 常对象 =====
姓名: Bob
Hello! I'm Bob, 30 years old.
问候次数: 1

===== 常引用参数 =====
=== 展示人物信息 ===
姓名: Alice
年龄: 27
Hello! I'm Alice, 27 years old.
```

---

## 七、调用权限总结

### 7.1 权限矩阵图

```
┌────────────────────────────────────────────────────────────────────────┐
│                           成员函数调用权限                              │
├────────────────┬─────────────────────┬─────────────────────────────────┤
│     对象类型    │   可以调用常函数？   │   可以调用普通函数？             │
├────────────────┼─────────────────────┼─────────────────────────────────┤
│   普通对象      │        ✅ 是        │            ✅ 是                │
│   Person p;    │                     │                                 │
├────────────────┼─────────────────────┼─────────────────────────────────┤
│   常对象        │        ✅ 是        │            ❌ 否                │
│   const Person p;│                    │                                 │
├────────────────┼─────────────────────┼─────────────────────────────────┤
│   常引用        │        ✅ 是        │            ❌ 否                │
│   const Person& p│                    │                                 │
├────────────────┼─────────────────────┼─────────────────────────────────┤
│   常指针        │        ✅ 是        │            ❌ 否                │
│   const Person* p│                    │                                 │
└────────────────┴─────────────────────┴─────────────────────────────────┘
```

### 7.2 成员变量修改权限

```
┌────────────────────────────────────────────────────────────────────────┐
│                        成员变量修改权限                                 │
├────────────────────┬─────────────────────┬────────────────────────────┤
│       场景          │  普通成员变量？      │   mutable 成员变量？        │
├────────────────────┼─────────────────────┼────────────────────────────┤
│   普通函数          │        ✅ 可修改     │          ✅ 可修改          │
├────────────────────┼─────────────────────┼────────────────────────────┤
│   常函数            │        ❌ 不可修改   │          ✅ 可修改          │
├────────────────────┼─────────────────────┼────────────────────────────┤
│   普通对象          │        ✅ 可修改     │          ✅ 可修改          │
├────────────────────┼─────────────────────┼────────────────────────────┤
│   常对象            │        ❌ 不可修改   │          ✅ 可修改          │
└────────────────────┴─────────────────────┴────────────────────────────┘
```

---

## 八、最佳实践

<details>
<summary><b>📖 点击展开：const 使用的最佳实践</b></summary>

### 1. 何时使用常函数

```cpp
class GoodExample {
private:
    int data;
    
public:
    // ✅ 所有"获取"类型的函数都应该声明为 const
    int getData() const { return data; }
    
    // ✅ 不修改状态的函数应该声明为 const
    bool isValid() const { return data > 0; }
    
    // ✅ 打印/显示函数通常应该是 const
    void print() const { std::cout << data << std::endl; }
    
    // ✅ 比较/判断函数应该是 const
    bool operator==(const GoodExample& other) const {
        return data == other.data;
    }
};
```

### 2. 函数重载：const 版本与非 const 版本

```cpp
#include <iostream>
#include <vector>

class Container {
private:
    std::vector<int> data;

public:
    Container(std::initializer_list<int> init) : data(init) {}

    // ✅ 非 const 版本：返回可修改的引用
    int& operator[](size_t index) {
        std::cout << "[非const版本]" << std::endl;
        return data[index];
    }

    // ✅ const 版本：返回只读引用
    const int& operator[](size_t index) const {
        std::cout << "[const版本]" << std::endl;
        return data[index];
    }
};

int main() {
    Container c{1, 2, 3, 4, 5};
    const Container& cref = c;
    
    c[0] = 10;       // 调用非 const 版本
    int val = cref[0]; // 调用 const 版本
    
    return 0;
}
```

### 3. 尽可能使用 const

```cpp
// ✅ 好习惯：函数参数用 const 引用避免拷贝
void processString(const std::string& str) {
    // str 不能被修改
}

// ✅ 好习惯：返回 const 指针防止外部修改
class Owner {
private:
    int* resource;
public:
    const int* getResource() const { return resource; }
};

// ✅ 好习惯：const 成员函数保证逻辑正确性
class Calculator {
public:
    // 纯计算函数，不修改任何状态
    int add(int a, int b) const { return a + b; }
};
```

</details>

---

## 九、常见陷阱与误区

<details>
<summary><b>⚠️ 点击展开：常见陷阱与误区</b></summary>

### 陷阱 1：const 位置写错

```cpp
class Mistake {
public:
    // ❌ 错误：这声明了一个返回 const int 的普通函数
    const int getValue() { return 42; }
    
    // ✅ 正确：这才是常函数
    int getValue() const { return 42; }
};
```

### 陷阱 2：忘记 const 导致无法使用

```cpp
class Mistake {
public:
    int getValue() { return 42; }  // 忘记加 const
};

void process(const Mistake& m) {
    // int x = m.getValue();  // ❌ 编译错误！
}

// 修复方案：
class Fixed {
public:
    int getValue() const { return 42; }  // 加上 const
};
```

### 陷阱 3：在常函数中返回非 const 引用

```cpp
class Mistake {
private:
    int data;
public:
    // ❌ 危险：常函数返回非 const 引用
    int& getData() const { return data; }
    // 这会破坏 const 的语义！
    // 现代 C++ 编译器可能会警告或报错
};
```

### 陷阱 4：mutable 滥用

```cpp
class BadDesign {
private:
    int importantData;
    mutable int cache;  // ✅ 合理的 mutable 用法
    
    mutable int dataBackup;  // ❌ 不要用 mutable 绕过 const 限制
                             // 如果需要在常函数中修改核心数据，
                             // 重新考虑设计是否合理
};
```

</details>

---

## 十、实战练习

### 练习题：设计一个矩形类

```cpp
#include <iostream>

class Rectangle {
private:
    double width;
    double height;
    mutable double cachedArea;      // 缓存面积
    mutable bool areaIsValid;       // 缓存是否有效

public:
    // 构造函数
    Rectangle(double w, double h) 
        : width(w), height(h), cachedArea(0), areaIsValid(false) {}
    
    // TODO: 实现以下成员函数
    // 1. getWidth() - 常函数，返回宽度
    // 2. getHeight() - 常函数，返回高度
    // 3. getArea() - 常函数，返回面积（使用缓存）
    // 4. setWidth(double) - 设置宽度（需要使缓存失效）
    // 5. setHeight(double) - 设置高度（需要使缓存失效）
    // 6. isSquare() - 常函数，判断是否为正方形
    // 7. print() - 常函数，打印矩形信息
    
    // 在此编写你的代码...
};

int main() {
    const Rectangle rect1(5.0, 3.0);
    Rectangle rect2(4.0, 4.0);
    
    // 测试你的实现...
    
    return 0;
}
```

<details>
<summary><b>✅ 点击查看参考答案</b></summary>

```cpp
#include <iostream>
#include <iomanip>

class Rectangle {
private:
    double width;
    double height;
    mutable double cachedArea;
    mutable bool areaIsValid;

public:
    Rectangle(double w, double h) 
        : width(w), height(h), cachedArea(0), areaIsValid(false) {}
    
    // 1. 常函数：获取宽度
    double getWidth() const { return width; }
    
    // 2. 常函数：获取高度
    double getHeight() const { return height; }
    
    // 3. 常函数：获取面积（带缓存）
    double getArea() const {
        if (!areaIsValid) {
            cachedArea = width * height;
            areaIsValid = true;
            std::cout << "[计算面积]" << std::endl;
        } else {
            std::cout << "[使用缓存]" << std::endl;
        }
        return cachedArea;
    }
    
    // 4. 设置宽度
    void setWidth(double w) {
        width = w;
        areaIsValid = false;
    }
    
    // 5. 设置高度
    void setHeight(double h) {
        height = h;
        areaIsValid = false;
    }
    
    // 6. 常函数：判断是否为正方形
    bool isSquare() const {
        return width == height;
    }
    
    // 7. 常函数：打印信息
    void print() const {
        std::cout << "矩形: " << width << " x " << height 
                  << ", 面积: " << getArea()
                  << (isSquare() ? " (正方形)" : "") << std::endl;
    }
};

int main() {
    std::cout << "=== 常对象测试 ===" << std::endl;
    const Rectangle rect1(5.0, 3.0);
    rect1.print();
    rect1.print();  // 第二次应该使用缓存
    
    std::cout << "\n=== 普通对象测试 ===" << std::endl;
    Rectangle rect2(4.0, 4.0);
    rect2.print();
    
    std::cout << "\n修改尺寸..." << std::endl;
    rect2.setWidth(6.0);
    rect2.print();  // 缓存应该失效，重新计算
    
    std::cout << "\n=== 权限测试 ===" << std::endl;
    // rect1.setWidth(10);  // ❌ 常对象不能调用非常函数
    rect2.setWidth(10);     // ✅ 普通对象可以调用
    
    return 0;
}
```

</details>

---

## 总结

| 概念        | 语法                  | 作用                                       |
| ----------- | --------------------- | ------------------------------------------ |
| **常函数**  | `void func() const`   | 承诺不修改对象状态，`this` 变成 `const T*` |
| **常对象**  | `const ClassName obj` | 只能调用常函数，不能被修改                 |
| **mutable** | `mutable int x`       | 允许在常函数/常对象中修改该成员            |

**核心记忆口诀：**
> 常函数不能改成员，mutable 是个例外门；  
> 常对象只能调常函数，普通对象全都行！

如果有任何疑问，欢迎继续提问！