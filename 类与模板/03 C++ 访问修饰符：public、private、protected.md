# C++中的访问修饰符（**private**、**public**、**protected**）

在C++中，**private**、**public**、**protected** 是三种访问修饰符(访问控制符),用来控制类成员的访问权限。

## 1. **public(公有)**

**特点:**

- 类的外部可以直接访问
- 派生类可以访问
- 类内部可以访问

**使用场景:** 提供给外部使用的接口函数和需要公开的成员

```cpp
class Student {
public:
    string name;  // 公有成员变量
    
    void setAge(int a) {  // 公有成员函数
        age = a;
    }
    
    int getAge() {
        return age;
    }
    
private:
    int age;
};

int main() {
    Student stu;
    stu.name = "张三";  // ✓ 可以直接访问
    stu.setAge(20);     // ✓ 可以调用公有函数
    // stu.age = 20;    // ✗ 错误!不能直接访问private成员
}
```

## 2. **private(私有)**

**特点:**

- 只能在类内部访问
- 类的外部不能访问
- 派生类也不能直接访问

**使用场景:** 隐藏类的内部实现细节,实现数据封装

```cpp
class BankAccount {
private:
    double balance;      // 私有成员,外部无法直接修改
    string password;
    
    bool verifyPassword(string pwd) {  // 私有函数
        return password == pwd;
    }
    
public:
    BankAccount(double initial) {
        balance = initial;
        password = "123456";
    }
    
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;  // 类内部可以访问private成员
        }
    }
    
    double getBalance() {
        return balance;
    }
};

int main() {
    BankAccount account(1000);
    account.deposit(500);           // ✓ 可以调用
    cout << account.getBalance();   // ✓ 可以调用
    // account.balance = 99999;     // ✗ 错误!不能直接访问
    // account.verifyPassword("123"); // ✗ 错误!私有函数不能访问
}
```

## 3. **protected(保护)**

**特点:**

- 类内部可以访问
- 派生类(子类)可以访问
- 类的外部不能访问

**使用场景:** 需要在继承体系中共享,但不想对外公开的成员

```cpp
class Person {
protected:
    string name;
    int age;
    
    void showInfo() {  // protected函数
        cout << "姓名: " << name << ", 年龄: " << age << endl;
    }
    
public:
    Person(string n, int a) : name(n), age(a) {}
};

class Employee : public Person {
private:
    double salary;
    
public:
    Employee(string n, int a, double s) : Person(n, a) {
        salary = s;
    }
    
    void display() {
        // ✓ 派生类可以访问基类的protected成员
        cout << "员工: " << name << ", " << age << "岁, 薪水: " << salary << endl;
        showInfo();  // ✓ 可以调用protected函数
    }
};

int main() {
    Employee emp("李四", 30, 8000);
    emp.display();       // ✓ 可以调用
    // emp.name = "王五"; // ✗ 错误!外部不能访问protected成员
    // emp.showInfo();   // ✗ 错误!外部不能调用protected函数
}
```

## 4. 继承中的访问控制

继承方式会影响基类成员在派生类中的访问属性:

```cpp
class Base {
public:
    int pub_mem;
protected:
    int prot_mem;
private:
    int priv_mem;
};

// 公有继承 (最常用)
class D1 : public Base {
    // pub_mem  → public
    // prot_mem → protected
    // priv_mem → 不可访问
};

// 保护继承
class D2 : protected Base {
    // pub_mem  → protected
    // prot_mem → protected
    // priv_mem → 不可访问
};

// 私有继承
class D3 : private Base {
    // pub_mem  → private
    // prot_mem → private
    // priv_mem → 不可访问
};
```

## 5. 完整示例

```cpp
class Animal {
protected:
    string name;
    int age;
    
    void breathe() {  // 子类可用,但外部不可见
        cout << "呼吸中..." << endl;
    }
    
private:
    string species;  // 只有Animal类内部可访问
    
public:
    Animal(string n, int a, string s) : name(n), age(a), species(s) {}
    
    void eat() {
        cout << name << "正在进食" << endl;
    }
    
    string getSpecies() {
        return species;  // 通过public函数访问private成员
    }
};

class Dog : public Animal {
private:
    string breed;
    
public:
    Dog(string n, int a, string b) : Animal(n, a, "犬科"), breed(b) {}
    
    void bark() {
        // ✓ 可以访问protected成员
        cout << name << " (品种: " << breed << ") 正在叫: 汪汪!" << endl;
        breathe();  // ✓ 可以调用protected函数
        // cout << species;  // ✗ 不能访问基类的private成员
    }
};

int main() {
    Dog dog("旺财", 3, "哈士奇");
    dog.eat();   // ✓ public函数
    dog.bark();  // ✓ public函数
    
    // dog.name = "小黑";  // ✗ protected成员外部不可访问
    // dog.breathe();     // ✗ protected函数外部不可调用
    
    cout << "物种: " << dog.getSpecies() << endl;  // ✓ 通过public接口访问
}
```

## 6. 访问权限总结表

| 访问位置 | public | protected | private |
| -------- | ------ | --------- | ------- |
| 类内部   | ✓      | ✓         | ✓       |
| 派生类   | ✓      | ✓         | ✗       |
| 类外部   | ✓      | ✗         | ✗       |

## 7. 最佳实践

1. **数据成员通常设为private**,通过public的getter/setter访问
2. **接口函数设为public**,供外部调用
3. **需要被继承使用的成员设为protected**
4. **遵循封装原则**,隐藏实现细节
5. **默认情况**: class默认是private, struct默认是public

```cpp
class Example {
    int x;  // 默认private
public:
    int y;  // 显式public
};

struct Example2 {
    int x;  // 默认public
private:
    int y;  // 显式private
};
```

这三种访问修饰符是C++实现**封装**这一面向对象特性的核心机制,合理使用可以提高代码的安全性和可维护性。