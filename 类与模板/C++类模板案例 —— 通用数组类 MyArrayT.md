# C++类模板案例 —— 通用数组类 `MyArray<T>`

## 📋 案例目标

实现一个**通用的数组类**，它的行为类似于 `std::vector`，但我们从零开始手写，目的是深入理解以下核心 C++ 概念：

| 需求                     | 涉及的核心概念                 |
| ------------------------ | ------------------------------ |
| 存储内置类型和自定义类型 | 类模板 template<typename T>    |
| 数据存储到堆区           | new / delete[] 动态内存管理    |
| 传入数组容量             | 有参构造函数                   |
| 拷贝构造 & operator=     | 深拷贝，防止浅拷贝导致重复释放 |
| 尾插法 & 尾删法          | 成员函数设计，边界判断         |
| 通过下标访问元素         | operator[] 重载                |
| 获取元素个数和容量       | getter 成员函数                |

---

## 一、为什么需要"通用数组"？—— 从问题出发

假设你需要一个能存 `int` 的动态数组，你可能会写：

```cpp
class IntArray {
    int* m_data;
    int  m_capacity;
    int  m_size;
    // ...
};
```

现在又需要一个存 `double` 的、存 `string` 的、存自定义 `Person` 的……难道要写四个类？

**类模板**就是为了解决这个问题——**把数据类型参数化**，让编译器帮你生成不同类型的版本。

---

## 二、逐步构建 `MyArray<T>`

### 2.1 类模板的骨架

```cpp
template<typename T>
class MyArray {
private:
    T*  m_pAddress;   // 指向堆区数组的指针
    int m_capacity;   // 数组容量（最多能存多少个元素）
    int m_size;       // 当前实际存了多少个元素

public:
    // 各种构造函数、运算符重载、成员函数……
};
```

> **关键点**：`T` 是一个**类型占位符**。当你写 `MyArray<int>` 时，编译器会把所有 `T` 替换成 `int`，生成一个专门处理 `int` 的类。`MyArray<Person>` 则会生成一个专门处理 `Person` 的类。

---

### 2.2 有参构造函数 —— 在堆区开辟空间

```cpp
// 有参构造：传入容量
MyArray(int capacity) {
    m_capacity = capacity;
    m_size = 0;
    m_pAddress = new T[m_capacity];  // 在堆区申请 capacity 个 T 类型的空间
}
```

**逐行解析**：

| 代码              | 含义                                                      |
| ----------------- | --------------------------------------------------------- |
| new T[m_capacity] | 在堆区连续分配 m_capacity 个 T 类型大小的内存，返回首地址 |
| m_size = 0        | 刚创建时，数组里还没放任何元素                            |

> ⚠️ **栈区 vs 堆区**：
>

> - **栈区**：函数结束自动释放，空间有限
>
> - | 类型             | 说明                             |
>  | ---------------- | -------------------------------- |
>   | **局部变量**     | 函数内部定义的变量（非`static`） |
>  | **函数参数**     | 形参的值传递或指针传递           |
>   | **返回地址**     | 函数调用后返回的位置             |
>   | **寄存器上下文** | 调用函数前的CPU寄存器状态        |
> 
> 
> 
> - **堆区**：程序员手动管理（`new` 申请，`delete` 释放），空间大

> 我们把数据放在堆区，是因为数组可能很大，而且我们希望它的生命周期由我们自己控制。

---

### 2.3 析构函数 —— 释放堆区内存

```cpp
~MyArray() {
    if (m_pAddress != nullptr) {
        delete[] m_pAddress;    // 释放堆区数组
        m_pAddress = nullptr;   // 置空，防止野指针
        m_capacity = 0;
        m_size = 0;
    }
}
```

> **为什么用 `delete[]` 而不是 `delete`？**
>
> 
>
> - `new T[n]` 申请的是**数组**，必须用 `delete[]` 释放
>
> - `new T` 申请的是**单个对象**，用 `delete` 释放
>
> - 混用会导致**未定义行为**（可能崩溃、内存泄漏等）

---

### 2.4 浅拷贝问题 —— 为什么必须写拷贝构造和 `operator=`？

这是本案例**最重要的概念**，务必理解。

#### 什么是浅拷贝？

如果你不写拷贝构造函数，编译器会自动生成一个，它做的是**逐成员复制**（浅拷贝）：

```
对象 a:  m_pAddress = 0x1000  →  [ 1, 2, 3, ?, ? ]
                                        ↑
对象 b（浅拷贝自 a）:  m_pAddress = 0x1000  ──┘  (指向同一块内存！)
```

**灾难发生的时刻**：当 `b` 先析构，它执行 `delete[] m_pAddress`，把 `0x1000` 的内存释放了。然后 `a` 再析构，又执行 `delete[] m_pAddress`，**再次释放同一块内存** → 💥 **程序崩溃（double free）**！

#### 深拷贝的解决方案

深拷贝的核心思想：**你有你的堆区，我有我的堆区，互不干扰**。

```
对象 a:  m_pAddress = 0x1000  →  [ 1, 2, 3, ?, ? ]

对象 b（深拷贝自 a）:  m_pAddress = 0x2000  →  [ 1, 2, 3, ?, ? ]  (全新的内存！)
```

---

### 2.5 拷贝构造函数 —— 深拷贝实现

```cpp
// 拷贝构造函数
MyArray(const MyArray& other) {
    m_capacity = other.m_capacity; //当前的最大容量
    m_size = other.m_size;         //当前的容量

    // 核心：重新在堆区申请一块全新的内存
    m_pAddress = new T[m_capacity];

    // 把 other 中的每个元素逐一拷贝过来
    for (int i = 0; i < m_size; i++) {
        m_pAddress[i] = other.m_pAddress[i];
    }
}
```

**为什么参数是 `const MyArray&`？**

- `const`：拷贝源不应该被修改

- `&`（引用）：避免传值（传值本身会触发拷贝构造，形成无限递归 💀）

---

### 2.6 `operator=` 赋值运算符重载 —— 深拷贝的另一个入口

```cpp
// 赋值运算符重载
MyArray& operator=(const MyArray& other) {
    // 1. 先释放自己原有的堆区内存
    if (m_pAddress != nullptr) {
        delete[] m_pAddress;
        m_pAddress = nullptr;
    }

    // 2. 深拷贝
    m_capacity = other.m_capacity;
    m_size = other.m_size;
    m_pAddress = new T[m_capacity];

    for (int i = 0; i < m_size; i++) {
        m_pAddress[i] = other.m_pAddress[i];
    }

    // 3. 返回自身引用，支持链式赋值 a = b = c
    return *this;
}
```

**拷贝构造 vs 赋值运算符，什么时候调用谁？**

```cpp
MyArray<int> a(10);
MyArray<int> b(a);      // 拷贝构造：b 是新对象，在创建时用 a 来初始化
MyArray<int> c(5);
c = a;                  // 赋值运算符：c 已经存在，把 a 的内容赋给 c
```

> 一句话记忆：**"新生儿用拷贝构造，已有的人用赋值运算符"**

**为什么第 1 步要先释放？**

因为 `c` 在赋值之前已经通过构造函数在堆区申请了自己的内存。如果不先释放，直接让 `m_pAddress` 指向新内存，那**原来的旧内存就再也找不到了（内存泄漏）**。

**为什么返回 `MyArray&`（自身引用）？**

为了支持**链式赋值**：

```cpp
a = b = c;
// 等价于 a = (b = c);
// b = c 返回 b 的引用，然后 a = b
```

---

### 2.7 尾插法 —— `pushBack`

```cpp
// 尾插法：在数组末尾添加元素
void pushBack(const T& value) {
    if (m_size >= m_capacity) {
        // 数组已满，无法插入
        // 实际项目中可以选择扩容，这里简单处理
        return;
    }

    m_pAddress[m_size] = value;  // 在末尾位置放入新元素
    m_size++;                    // 元素个数 +1
}
```

**形象图解**（容量=5）：

```
pushBack(10):  [ 10,  ?,  ?,  ?,  ? ]    size=1
pushBack(20):  [ 10, 20,  ?,  ?,  ? ]    size=2
pushBack(30):  [ 10, 20, 30,  ?,  ? ]    size=3
                              ↑
                          m_size 指向下一个空位
```

**为什么参数是 `const T&`？**

- `const`：我们只是读取 `value` 的值，不会修改它

- `&`（引用）：如果 `T` 是一个很大的自定义类型（比如包含很多成员的类），传引用可以**避免不必要的拷贝**，提高性能

---

### 2.8 尾删法 —— `popBack`

```cpp
// 尾删法：删除数组末尾的元素
void popBack() {
    if (m_size == 0) {
        // 数组为空，无法删除
        return;
    }

    m_size--;  // 逻辑上删除：让用户访问不到最后一个元素
}
```

> **注意**：这里并没有真正清除数据或释放内存，只是把 `m_size` 减 1。这是因为：
>
> 
>
> 1. 那个位置的内存本来就是我们申请的数组的一部分，不需要单独释放
>
> 2. 下次 `pushBack` 时会**覆盖**那个位置的旧数据
>
> 3. 用户通过 `operator[]` 和 `getSize()` 访问时，永远只能访问 `[0, m_size)` 范围，所以"逻辑删除"就够了

---

### 2.9 `operator[]` —— 通过下标访问元素

```cpp
// 下标运算符重载
T& operator[](int index) {
    return m_pAddress[index];
}
```

**为什么返回 `T&`（引用）？**

因为我们希望支持**通过下标修改元素**：

```cpp
MyArray<int> arr(10);
arr.pushBack(100);
arr[0] = 200;        // 如果返回的是值（T），这里修改的就是临时副本，原数组不变
                     // 返回引用（T&），修改的就是数组中的真实元素
```

---

### 2.10 获取大小和容量

```cpp
// 获取当前元素个数
int getSize() const {
    return m_size;
}

// 获取数组容量
int getCapacity() const {
    return m_capacity;
}
```

> **`const` 成员函数**：函数签名后面的 `const` 表示"这个函数不会修改对象的任何成员变量"。这是一个好习惯——对于只读操作，都应该标记为 `const`。

---

## 三、完整实现代码

```cpp
#include <iostream>
#include <string>
using namespace std;

// =============================================
// 自定义数据类型：用于测试 MyArray<Person>
// =============================================
class Person {
public:
    string m_name;
    int m_age;

    Person() : m_name(""), m_age(0) {}

    Person(const string& name, int age) : m_name(name), m_age(age) {}
};

// =============================================
// 通用数组类模板
// =============================================
template<typename T>
class MyArray {
private:
    T*  m_pAddress;   // 指向堆区数组的指针
    int m_capacity;   // 容量
    int m_size;       // 当前元素个数

public:
    // ---------- 有参构造 ----------
    MyArray(int capacity) {
        cout << "MyArray 有参构造调用" << endl;
        m_capacity = capacity;
        m_size = 0;
        m_pAddress = new T[m_capacity];
    }

    // ---------- 拷贝构造（深拷贝）----------
    MyArray(const MyArray& other) {
        cout << "MyArray 拷贝构造调用" << endl;
        m_capacity = other.m_capacity;
        m_size = other.m_size;
        m_pAddress = new T[m_capacity];

        for (int i = 0; i < m_size; i++) {
            m_pAddress[i] = other.m_pAddress[i];
        }
    }

    // ---------- 赋值运算符重载（深拷贝）----------
    MyArray& operator=(const MyArray& other) {
        cout << "MyArray operator= 调用" << endl;

        // 先释放原有内存
        if (m_pAddress != nullptr) {
            delete[] m_pAddress;
            m_pAddress = nullptr;
        }

        // 深拷贝
        m_capacity = other.m_capacity;
        m_size = other.m_size;
        m_pAddress = new T[m_capacity];

        for (int i = 0; i < m_size; i++) {
            m_pAddress[i] = other.m_pAddress[i];
        }

        return *this;
    }

    // ---------- 尾插法 ----------
    void pushBack(const T& value) {
        if (m_size >= m_capacity) {
            cout << "数组已满，无法插入！" << endl;
            return;
        }
        m_pAddress[m_size] = value;
        m_size++;
    }

    // ---------- 尾删法 ----------
    void popBack() {
        if (m_size == 0) {
            cout << "数组为空，无法删除！" << endl;
            return;
        }
        m_size--;
    }

    // ---------- 下标访问 ----------
    T& operator[](int index) {
        return m_pAddress[index];
    }

    // ---------- 获取当前元素个数 ----------
    int getSize() const {
        return m_size;
    }

    // ---------- 获取容量 ----------
    int getCapacity() const {
        return m_capacity;
    }

    // ---------- 析构函数 ----------
    ~MyArray() {
        if (m_pAddress != nullptr) {
            delete[] m_pAddress;
            m_pAddress = nullptr;
            m_capacity = 0;
            m_size = 0;
        }
    }
};

// =============================================
// 打印 int 数组
// =============================================
void printIntArray(
) {
    for (int i = 0; i < arr.getSize(); i++) {
        cout << arr[i] << " ";
    }
    cout << endl;
}

// =============================================
// 打印 Person 数组
// =============================================
void printPersonArray(MyArray<Person>& arr) {
    for (int i = 0; i < arr.getSize(); i++) {
        cout << "姓名：" << arr[i].m_name
             << "  年龄：" << arr[i].m_age << endl;
    }
}

// =============================================
// 测试1：内置数据类型
// =============================================
void test01() {
    cout << "========== 测试1：内置类型 int ==========" << endl;

    MyArray<int> arr(5);

    // 尾插
    for (int i = 0; i < 5; i++) {
        arr.pushBack(i * 10);
    }

    cout << "arr 的内容：";
    printIntArray(arr);
    cout << "arr 的容量：" << arr.getCapacity() << endl;
    cout << "arr 的大小：" << arr.getSize() << endl;

    // 尾删
    arr.popBack();
    cout << "尾删后 arr 的内容：";
    printIntArray(arr);
    cout << "尾删后 arr 的大小：" << arr.getSize() << endl;

    // 拷贝构造
    MyArray<int> arr2(arr);
    cout << "arr2（拷贝自 arr）的内容：";
    printIntArray(arr2);

    // 赋值运算符
    MyArray<int> arr3(100);
    arr3 = arr;
    cout << "arr3（赋值自 arr）的内容：";
    printIntArray(arr3);
}

// =============================================
// 测试2：自定义数据类型
// =============================================
void test02() {
    cout << "\n========== 测试2：自定义类型 Person ==========" << endl;

    MyArray<Person> pArr(10);

    Person p1("孙悟空", 999);
    Person p2("猪八戒", 800);
    Person p3("沙悟净", 700);
    Person p4("唐僧",   30);
    Person p5("白龙马", 500);

    pArr.pushBack(p1);
    pArr.pushBack(p2);
    pArr.pushBack(p3);
    pArr.pushBack(p4);
    pArr.pushBack(p5);

    cout << "pArr 的内容：" << endl;
    printPersonArray(pArr);
    cout << "pArr 的容量：" << pArr.getCapacity() << endl;
    cout << "pArr 的大小：" << pArr.getSize() << endl;

    // 通过下标修改
    pArr[0].m_name = "齐天大圣";
    cout << "\n修改后 pArr[0]：" << pArr[0].m_name
         << "  " << pArr[0].m_age << endl;
}

int main() {
    test01();
    test02();
    return 0;
}
```

---

## 四、运行结果

```
========== 测试1：内置类型 int ==========
MyArray 有参构造调用
arr 的内容：0 10 20 30 40 
arr 的容量：5
arr 的大小：5
尾删后 arr 的内容：0 10 20 30 
尾删后 arr 的大小：4
MyArray 拷贝构造调用
arr2（拷贝自 arr）的内容：0 10 20 30 
MyArray 有参构造调用
MyArray operator= 调用
arr3（赋值自 arr）的内容：0 10 20 30 

========== 测试2：自定义类型 Person ==========
MyArray 有参构造调用
pArr 的内容：
姓名：孙悟空  年龄：999
姓名：猪八戒  年龄：800
姓名：沙悟净  年龄：700
姓名：唐僧  年龄：30
姓名：白龙马  年龄：500
pArr 的容量：10
pArr 的大小：5

修改后 pArr[0]：齐天大圣  999
```

---

## 五、核心概念总结图

```
MyArray<T> 对象
                    ┌────────────────────────┐
  栈区              │  m_pAddress ──────┐     │
                    │  m_capacity = 5   │     │
                    │  m_size = 3       │     │
                    └───────────────────│─────┘
                                       │
                                       ▼
  堆区              ┌─────┬─────┬─────┬─────┬─────┐
         (new T[5]) │ T_0 │ T_1 │ T_2 │  ?  │  ?  │
                    └─────┴─────┴─────┴─────┴─────┘
                      ↑ 已使用(size=3) ↑  ↑ 未使用 ↑
```

---

## 六、易错点 & 进阶思考

### 6.1 `Person` 为什么需要默认构造函数？

```cpp
m_pAddress = new T[m_capacity];
```

当执行 `new T[n]` 时，编译器需要调用 `T` 的**默认构造函数**来初始化数组中的每个元素。如果 `Person` 只有 `Person(string, int)` 而**没有**无参的 `Person()`，编译器会报错：

```
error: no matching function for call to 'Person::Person()'
```

所以 `Person` 类必须提供一个默认构造函数（哪怕是空的）。

### 6.2 `operator=` 中没有处理自我赋值

如果有人写了 `arr = arr;`，当前的实现会**先把自己的内存释放掉，再从已被释放的自己身上拷贝** → 💥 未定义行为！

改进方案：加一个**自我赋值检查**：

```cpp
MyArray& operator=(const MyArray& other) {
    if (this == &other) {       // 如果是自己赋值给自己
        return *this;           // 直接返回，什么都不做
    }
    // ... 后续代码不变
}
```

### 6.3 `operator[]` 没有做越界检查

当前实现中，如果用户写 `arr[100]` 但数组只有 5 个元素，程序会**直接访问非法内存**，导致不可预知的行为。

改进方案：

```cpp
T& operator[](int index) {
    if (index < 0 || index >= m_size) {
        // 可以抛出异常，或者做其他错误处理
        throw out_of_range("MyArray: 下标越界！");
    }
    return m_pAddress[index];
}
```

### 6.4 生命周期对照表

| 时机                | 调用什么  | 做了什么                        |
| ------------------- | --------- | ------------------------------- |
| MyArray<int> a(10); | 有参构造  | 堆区 new int[10]                |
| MyArray<int> b(a);  | 拷贝构造  | 堆区 new int[10]，复制 a 的数据 |
| c = a;              | operator= | 先 delete[] 旧的，再 new + 复制 |
| 离开作用域          | 析构函数  | delete[] 释放堆区               |

---

## 七、知识点关系图

```
类模板 MyArray<T>
    │
    ├── 构造函数
    │     ├── 有参构造 ──→ new T[capacity]（堆区分配）
    │     └── 拷贝构造 ──→ 深拷贝（防止 double free）
    │
    ├── 析构函数 ──→ delete[]（释放堆区）
    │
    ├── 运算符重载
    │     ├── operator= ──→ 释放旧内存 + 深拷贝 + return *this
    │     └── operator[] ──→ 返回 T&（支持读写）
    │
    ├── 成员函数
    │     ├── pushBack ──→ 尾部插入，size++
    │     ├── popBack  ──→ 逻辑删除，size--
    │     ├── getSize  ──→ 返回 m_size
    │     └── getCapacity ──→ 返回 m_capacity
    │
    └── 核心原则
          ├── 谁申请谁释放（RAII）
          ├── 深拷贝避免共享堆区
          └── 自定义类型需要默认构造函数
```

---

