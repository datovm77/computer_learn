# C++中的浅拷贝与深拷贝详解

## 一、从最基础开始：什么是拷贝？

在C++中，当你创建一个对象的副本时，就发生了拷贝。比如：

```cpp
int a = 10;
int b = a;  // 这就是拷贝，b得到了a的值
```

对于基本数据类型（int、double等），拷贝很简单直接。但对于类对象，事情就复杂了。

## 二、默认的拷贝行为

当你定义一个类时，如果不特别说明，C++会自动为你生成一个**默认拷贝构造函数**，它会逐个拷贝类的每个成员变量。

```cpp
class Simple {
public:
    int x;
    double y;
    
    Simple(int a, double b) : x(a), y(b) {}
    // 编译器会自动生成拷贝构造函数
};

Simple obj1(10, 3.14);
Simple obj2 = obj1;  // 使用默认拷贝构造函数，x和y都被拷贝
```

这种情况下没有问题，因为所有成员都是简单类型。

## 三、问题出现了：指针成员

当类中包含指针成员时，麻烦就来了。看这个例子：

```cpp
class Array {
public:
    int* data;
    int size;
    
    Array(int s) {
        size = s;
        data = new int[size];  // 在堆上分配内存
        for(int i = 0; i < size; i++) {
            data[i] = 0;
        }
    }
    
    ~Array() {
        delete[] data;  // 释放内存
    }
};

Array arr1(5);
arr1.data[0] = 100;

Array arr2 = arr1;  // 默认拷贝
arr2.data[0] = 200;

// 这时arr1.data[0]是什么？答案是200！
```

### 这就是**浅拷贝**（Shallow Copy）

浅拷贝只是拷贝了指针的**地址值**，而不是指针指向的**内容**。结果是：

- `arr1.data` 和 `arr2.data` 指向**同一块内存**
- 修改其中一个会影响另一个
- 更严重的是：当 `arr1` 和 `arr2` 销毁时，同一块内存会被 `delete` **两次**，导致程序崩溃！

用图来理解：

```
浅拷贝后：
arr1.data ----→ [内存区域: 100, 0, 0, 0, 0]
                     ↑
arr2.data -----------┘  (两个指针指向同一块内存)
```

## 四、解决方案：深拷贝（Deep Copy）

深拷贝的核心思想是：不仅拷贝指针本身，还要**拷贝指针指向的内容**，为新对象分配独立的内存。

```cpp
class Array {
public:
    int* data;
    int size;
    
    Array(int s) {
        size = s;
        data = new int[size];
        for(int i = 0; i < size; i++) {
            data[i] = 0;
        }
    }
    
    // 自定义拷贝构造函数（深拷贝）
    Array(const Array& other) {
        size = other.size;
        data = new int[size];  // 分配新的内存
        for(int i = 0; i < size; i++) {
            data[i] = other.data[i];  // 拷贝每个元素
        }
    }
    
    ~Array() {
        delete[] data;
    }
};

Array arr1(5);
arr1.data[0] = 100;

Array arr2 = arr1;  // 使用自定义的拷贝构造函数
arr2.data[0] = 200;

// 现在arr1.data[0]仍然是100，arr2.data[0]是200
```

深拷贝后的情况：

```
arr1.data ----→ [内存区域1: 100, 0, 0, 0, 0]

arr2.data ----→ [内存区域2: 200, 0, 0, 0, 0]
                (完全独立的内存)
```

## 五、拷贝赋值运算符

除了拷贝构造函数，还需要注意**拷贝赋值运算符**：

```cpp
Array arr1(5);
Array arr2(3);
arr2 = arr1;  // 这里调用的是赋值运算符，不是拷贝构造函数
```

完整的深拷贝类应该这样写：

```cpp
class Array {
public:
    int* data;
    int size;
    
    Array(int s) {
        size = s;
        data = new int[size];
        for(int i = 0; i < size; i++) {
            data[i] = 0;
        }
    }
    
    // 拷贝构造函数
    Array(const Array& other) {
        size = other.size;
        data = new int[size];
        for(int i = 0; i < size; i++) {
            data[i] = other.data[i];
        }
    }
    
    // 拷贝赋值运算符
    Array& operator=(const Array& other) {
        if(this == &other) {  // 防止自我赋值
            return *this;
        }
        
        delete[] data;  // 释放旧内存
        
        size = other.size;
        data = new int[size];  // 分配新内存
        for(int i = 0; i < size; i++) {
            data[i] = other.data[i];
        }
        
        return *this;
    }
    
    ~Array() {
        delete[] data;
    }
};
```

## 六、总结对比

| 特性         | 浅拷贝               | 深拷贝                   |
| ------------ | -------------------- | ------------------------ |
| 拷贝指针成员 | 只拷贝地址值         | 拷贝指针指向的内容       |
| 独立性       | 共享同一块内存       | 完全独立的内存           |
| 相互影响     | 修改一个会影响另一个 | 互不影响                 |
| 内存安全     | 可能重复释放（崩溃） | 安全                     |
| 性能         | 快（只拷贝指针）     | 慢（需要分配和拷贝内存） |

## 七、经验法则

**"三法则"（Rule of Three）**：如果你的类需要自定义以下三者之一，通常需要自定义全部三个：

1. 析构函数
2. 拷贝构造函数
3. 拷贝赋值运算符

如果你的类管理动态内存（使用 `new/delete`），几乎肯定需要实现深拷贝。

现在你应该理解了浅拷贝和深拷贝的本质区别。有什么不清楚的地方吗？