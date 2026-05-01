# C++ const 修饰成员函数详解

## 一、为什么需要 const 成员函数?

在开始之前,先理解一个场景:

```cpp
class Student {
private:
    string name;
    int age;
public:
    void showInfo() {
        cout << "Name: " << name << ", Age: " << age << endl;
    }
};

int main() {
    const Student stu;  // 常对象
    stu.showInfo();     // 编译错误!
}
```

**问题**: 为什么常对象不能调用 `showInfo()`? 明明这个函数只是显示信息,并没有修改数据啊!

**原因**: 编译器不知道 `showInfo()` 会不会修改成员变量,为了安全,禁止常对象调用普通成员函数。

**解决方案**: 用 const 告诉编译器"这个函数保证不修改成员变量"。

------

## 二、常函数基础

### 2.1 常函数的语法

```cpp
class Student {
private:
    string name;
    int age;
public:
    // 常函数:在函数参数列表后加 const
    void showInfo() const {
        cout << "Name: " << name << ", Age: " << age << endl;
    }
    
    // 注意 const 的位置
    // 返回值 函数名(参数) const { }
    //                      ^^^^^ const 在这里
};
```

### 2.2 常函数的本质

常函数中的 `this` 指针类型发生了变化:

```cpp
// 普通成员函数
void showInfo() {
    // this 的类型是: Student* const
    // 指针本身不能改,但可以修改指向的内容
}

// 常成员函数
void showInfo() const {
    // this 的类型是: const Student* const
    // 指针本身不能改,指向的内容也不能改
}
```

------

## 三、常函数的限制

### 3.1 不能修改成员变量

```cpp
class Person {
private:
    string name;
    int age;
public:
    void setAge(int a) const {
        age = a;  // ❌ 编译错误!常函数不能修改成员变量
    }
    
    void showInfo() const {
        cout << name << ", " << age << endl;  // ✅ 可以读取
    }
};
```

### 3.2 不能调用非 const 成员函数

```cpp
class MyClass {
private:
    int value;
public:
    void modify() {
        value = 100;
    }
    
    void print() const {
        modify();  // ❌ 编译错误!
        // 常函数不能调用非常函数,因为非常函数可能修改成员
    }
    
    void display() const {
        cout << value << endl;
    }
    
    void process() const {
        display();  // ✅ 可以调用其他常函数
    }
};
```

------

## 四、mutable 关键字

### 4.1 为什么需要 mutable?

有时候,某些成员变量的修改**不影响对象的逻辑状态**,比如:

- 缓存数据
- 访问计数器
- 调试信息

```cpp
class Article {
private:
    string title;
    string content;
    int viewCount;  // 访问次数
    
public:
    void read() const {
        viewCount++;  // ❌ 编译错误!
        // 虽然只是计数,但在常函数中不能修改
        cout << content << endl;
    }
};
```

### 4.2 使用 mutable

```cpp
class Article {
private:
    string title;
    string content;
    mutable int viewCount;  // 用 mutable 修饰
    mutable bool cached;    // 可以有多个 mutable 成员
    
public:
    Article() : viewCount(0), cached(false) {}
    
    void read() const {
        viewCount++;  // ✅ 现在可以了!
        cached = true;
        cout << "阅读次数: " << viewCount << endl;
        cout << content << endl;
    }
    
    int getViewCount() const {
        return viewCount;  // 常函数读取 mutable 变量
    }
};
```

### 4.3 实际应用示例

```cpp
class ExpensiveCalculation {
private:
    double data;
    mutable double cachedResult;  // 缓存计算结果
    mutable bool isCalculated;    // 是否已计算
    
public:
    ExpensiveCalculation(double d) : data(d), isCalculated(false) {}
    
    double getResult() const {
        if (!isCalculated) {
            // 模拟耗时计算
            cachedResult = data * data * data;
            isCalculated = true;
            cout << "执行了复杂计算" << endl;
        } else {
            cout << "使用缓存结果" << endl;
        }
        return cachedResult;
    }
};

int main() {
    const ExpensiveCalculation calc(5.0);
    cout << calc.getResult() << endl;  // 执行计算
    cout << calc.getResult() << endl;  // 使用缓存
}
```

------

## 五、常对象

### 5.1 常对象的定义

```cpp
const MyClass obj1;           // 方式1
MyClass const obj2;           // 方式2(不推荐,但合法)
const MyClass obj3(10);       // 带参数构造
const MyClass* ptr = new MyClass();  // 指向常对象的指针
```

### 5.2 常对象的限制

```cpp
class Book {
private:
    string title;
    int pages;
    mutable int readCount;
    
public:
    Book(string t, int p) : title(t), pages(p), readCount(0) {}
    
    void setPages(int p) {
        pages = p;
    }
    
    void showInfo() const {
        readCount++;
        cout << title << ", " << pages << "页" << endl;
    }
    
    int getPages() const {
        return pages;
    }
};

int main() {
    const Book book("C++编程", 500);
    
    book.showInfo();      // ✅ 可以调用常函数
    cout << book.getPages() << endl;  // ✅ 可以调用常函数
    
    book.setPages(600);   // ❌ 编译错误!常对象不能调用非常函数
}
```

------

## 六、综合示例

```cpp
#include <iostream>
#include <string>
using namespace std;

class BankAccount {
private:
    string accountNumber;
    double balance;
    mutable int queryCount;  // 查询次数(不影响账户状态)
    
public:
    BankAccount(string num, double bal) 
        : accountNumber(num), balance(bal), queryCount(0) {}
    
    // 常函数:查询余额
    double getBalance() const {
        queryCount++;  // mutable 变量可以修改
        cout << "[查询次数: " << queryCount << "]" << endl;
        return balance;
    }
    
    // 常函数:显示信息
    void displayInfo() const {
        queryCount++;
        cout << "账号: " << accountNumber << endl;
        cout << "余额: " << balance << endl;
    }
    
    // 非常函数:存款
    void deposit(double amount) {
        balance += amount;
    }
    
    // 非常函数:取款
    bool withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;
    }
};

int main() {
    // 普通对象
    BankAccount acc1("123456", 1000.0);
    acc1.deposit(500);           // ✅ 可以调用所有函数
    acc1.displayInfo();          // ✅
    
    cout << "-------------------" << endl;
    
    // 常对象(只读账户)
    const BankAccount acc2("789012", 2000.0);
    acc2.displayInfo();          // ✅ 可以调用常函数
    cout << acc2.getBalance() << endl;  // ✅
    
    // acc2.deposit(100);        // ❌ 不能调用非常函数
    // acc2.withdraw(50);        // ❌ 不能调用非常函数
    
    return 0;
}
```

**输出:**

```
[查询次数: 1]
账号: 123456
余额: 1500
-------------------
[查询次数: 1]
账号: 789012
余额: 2000
[查询次数: 2]
2000
```

------

## 七、重要规则总结

| 场景       | 能否调用常函数     | 能否调用非常函数   |
| ---------- | ------------------ | ------------------ |
| 普通对象   | ✅ 能               | ✅ 能               |
| 常对象     | ✅ 能               | ❌ 不能             |
| 常函数内部 | ✅ 能调用其他常函数 | ❌ 不能调用非常函数 |

**记忆口诀:**

- **常对象只能调常函数**
- **常函数不能改数据**(除了 mutable)
- **常函数不能调非常函数**

------

## 八、实践建议

1. **养成好习惯**: 不修改成员变量的函数都加 `const`
2. **const 正确性**: 设计类时就要考虑哪些函数应该是 const
3. **mutable 谨慎用**: 只用于不影响对象逻辑状态的成员
4. **接口设计**: 提供 const 和非 const 两个版本的函数(如有需要)

```cpp
class Container {
private:
    int* data;
public:
    // const 版本:返回 const 引用
    const int& at(int index) const {
        return data[index];
    }
    
    // 非 const 版本:返回非 const 引用
    int& at(int index) {
        return data[index];
    }
};
```

