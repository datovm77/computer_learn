# C++ 构造函数与析构函数

## C++ 中的构造函数（Constructor）

构造函数是在对象创建时自动调用的特殊成员函数，用于初始化对象的成员变量。

**特点：**

- 函数名与类名完全相同
- 没有返回类型（连 void 也不写）
- 可以重载（多个构造函数，参数不同）
- 对象创建时自动调用

**示例：**

```cpp
class Person {
private:
    string name;
    int age;
    int* ptr;  // 动态分配的指针

public:
    // 无参构造函数（默认构造函数）
    Person() {
        name = "未命名";
        age = 0;
        ptr = nullptr;
        cout << "无参构造函数被调用" << endl;
    }

    // 有参构造函数
    Person(string n, int a) {
        name = n;
        age = a;
        ptr = new int(100);  // 动态分配内存
        cout << "有参构造函数被调用：" << name << endl;
    }

    // 拷贝构造函数
    Person(const Person& p) {
        name = p.name;
        age = p.age;
        ptr = new int(*p.ptr);  // 深拷贝
        cout << "拷贝构造函数被调用" << endl;
    }

    void display() {
        cout << "姓名：" << name << ", 年龄：" << age << endl;
    }
};
```

## C++ 中的析构函数（Destructor）

析构函数在对象生命周期结束时自动调用，用于释放资源、清理工作。

**特点：**

- 函数名是类名前加波浪号 `~`
- 没有返回类型，没有参数
- 不能重载（只能有一个析构函数）
- 对象销毁时自动调用
- 主要用于释放动态分配的内存、关闭文件、释放锁等

**示例：**

```cpp
class Person {
private:
    string name;
    int* ptr;

public:
    Person(string n) {
        name = n;
        ptr = new int(100);
        cout << "构造：" << name << endl;
    }

    // 析构函数
    ~Person() {
        cout << "析构：" << name << endl;
        delete ptr;  // 释放动态分配的内存
        ptr = nullptr;
    }
};

int main() {
    Person p1("张三");
    {
        Person p2("李四");
    }  // p2 在这里被销毁，析构函数自动调用
    
    cout << "主函数继续执行" << endl;
    return 0;
}  // p1 在这里被销毁
```

**输出：**

```
构造：张三
构造：李四
析构：李四
主函数继续执行
析构：张三
```

------

# Python 中的对应函数

## Python 中的 `__init__` 方法（初始化方法）

Python 中的 `__init__` 方法相当于 C++ 的构造函数，但严格来说它是**初始化方法**而非构造方法。

**特点：**

- 方法名固定为 `__init__`
- 第一个参数必须是 `self`（指向实例本身）
- 在对象创建后自动调用
- 可以有多个参数，但 Python 不支持传统意义的重载

**示例：**

```python
class Person:
    # 初始化方法（相当于 C++ 构造函数）
    def __init__(self, name="未命名", age=0):
        self.name = name
        self.age = age
        self.data = [1, 2, 3]  # Python 自动管理内存
        print(f"初始化：{self.name}")
    
    def display(self):
        print(f"姓名：{self.name}, 年龄：{self.age}")

# 使用
p1 = Person()                    # 使用默认参数
p2 = Person("张三", 25)          # 传入参数
p3 = Person(name="李四", age=30) # 关键字参数
```

**输出：**

```
初始化：未命名
初始化：张三
初始化：李四
```

**注意：** Python 通过默认参数和可变参数实现类似重载的效果：

```python
class Person:
    def __init__(self, *args, **kwargs):
        if len(args) == 0:
            self.name = "未命名"
            self.age = 0
        elif len(args) == 1:
            self.name = args[0]
            self.age = 0
        else:
            self.name = args[0]
            self.age = args[1]
        print(f"初始化：{self.name}")
```

## Python 中的 `__del__` 方法（析构方法）

Python 中的 `__del__` 方法相当于 C++ 的析构函数，但由于 Python 的垃圾回收机制，使用场景和行为有很大不同。

**特点：**

- 方法名固定为 `__del__`
- 当对象即将被销毁时调用
- 调用时机不确定（由垃圾回收器决定）
- 很少需要显式定义（Python 自动管理内存）

**示例：**

```python
class Person:
    def __init__(self, name):
        self.name = name
        print(f"初始化：{self.name}")
    
    def __del__(self):
        print(f"析构：{self.name}")

# 使用
p1 = Person("张三")
p2 = Person("李四")
del p2  # 显式删除
print("主程序继续执行")
# p1 在程序结束时被销毁
```

**可能的输出：**

```
初始化：张三
初始化：李四
析构：李四
主程序继续执行
析构：张三
```

**重要警告：** 在 Python 中，`__del__` 的调用时机是不确定的，不应该依赖它来进行重要的清理工作。应该使用上下文管理器（`with` 语句）。

------

# 详细对比

## 1. 命名与语法

| 特性       | C++            | Python                             |
| ---------- | -------------- | ---------------------------------- |
| 构造函数名 | 与类名相同     | 固定为 `__init__`                  |
| 析构函数名 | `~类名`        | 固定为 `__del__`                   |
| 返回类型   | 无返回类型     | 无返回语句（或返回 None）          |
| 参数       | 可以有任意参数 | `__init__` 第一个参数必须是 `self` |

## 2. 调用时机

**C++：**

- **构造函数：** 对象创建时立即调用（栈对象在声明时，堆对象在 `new` 时）
- **析构函数：** 对象销毁时立即调用（栈对象离开作用域时，堆对象 `delete` 时）
- 调用时机完全确定和可预测

```cpp
{
    Person p1("张三");  // 立即调用构造函数
}  // 立即调用析构函数
```

**Python：**

- **`__init__`：** 对象创建后立即调用（在 `__new__` 之后）
- **`__del__`：** 对象引用计数为 0 时调用，但具体时机由垃圾回收器决定
- `__del__` 调用时机不确定

```python
p1 = Person("张三")  # 立即调用 __init__
del p1  # 可能调用 __del__，但不保证立即调用
```

## 3. 重载支持

**C++：** 支持函数重载

```cpp
class Person {
public:
    Person() { }                        // 无参构造
    Person(string n) { name = n; }      // 单参数构造
    Person(string n, int a) {           // 双参数构造
        name = n; 
        age = a; 
    }
};
```

**Python：** 不支持传统重载，使用默认参数和可变参数

```python
class Person:
    def __init__(self, name="未命名", age=0):
        self.name = name
        self.age = age
    
    # 或者使用可变参数
    def __init__(self, *args, **kwargs):
        # 根据参数数量和类型处理
        pass
```

## 4. 内存管理

**C++：** 需要手动管理内存

```cpp
class Person {
private:
    int* data;
public:
    Person() {
        data = new int[100];  // 分配内存
    }
    
    ~Person() {
        delete[] data;  // 必须手动释放
        data = nullptr;
    }
};
```

**Python：** 自动垃圾回收

```python
class Person:
    def __init__(self):
        self.data = [0] * 100  # 自动管理内存
    
    # 通常不需要 __del__
    # Python 会自动回收内存
```

## 5. 资源清理的最佳实践

**C++：** 使用 RAII（Resource Acquisition Is Initialization）

```cpp
class FileHandler {
private:
    FILE* file;
public:
    FileHandler(const char* filename) {
        file = fopen(filename, "r");
    }
    
    ~FileHandler() {
        if (file) {
            fclose(file);  // 自动关闭文件
        }
    }
};

// 使用
{
    FileHandler fh("test.txt");
    // 使用文件
}  // 离开作用域，自动关闭文件
```

**Python：** 使用上下文管理器（`with` 语句）

```python
class FileHandler:
    def __init__(self, filename):
        self.file = open(filename, 'r')
    
    def __enter__(self):
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()  # 保证关闭

# 使用
with FileHandler("test.txt") as f:
    # 使用文件
    pass
# 自动调用 __exit__，关闭文件
```

## 6. 拷贝构造

**C++：** 有专门的拷贝构造函数

```cpp
class Person {
public:
    Person(const Person& other) {  // 拷贝构造函数
        name = other.name;
        age = other.age;
    }
};

Person p1("张三", 25);
Person p2 = p1;  // 调用拷贝构造函数
```

**Python：** 使用 `copy` 模块

```python
import copy

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

p1 = Person("张三", 25)
p2 = copy.copy(p1)       # 浅拷贝
p3 = copy.deepcopy(p1)   # 深拷贝
```

## 总结对比表

| 方面            | C++                  | Python                          |
| --------------- | -------------------- | ------------------------------- |
| **构造/初始化** | 构造函数，名称同类名 | `__init__` 方法                 |
| **析构/清理**   | 析构函数 `~类名`     | `__del__` 方法（少用）          |
| **调用确定性**  | 完全确定             | `__del__` 不确定                |
| **内存管理**    | 手动管理             | 自动垃圾回收                    |
| **重载**        | 支持函数重载         | 用默认参数模拟                  |
| **资源清理**    | RAII 模式            | 上下文管理器                    |
| **使用频率**    | 几乎总是需要         | `__init__` 常用，`__del__` 少用 |

**核心建议：**

- C++ 中析构函数非常重要，用于资源管理
- Python 中应该使用上下文管理器而不是 `__del__` 来管理资源
- Python 的垃圾回收机制使得很少需要关心对象销毁的细节