# `std::array` vs `std::vector` 性能差异深度解析

## 一、最本质的差异：内存模型

这是一切性能差异的根源。

### std::array — 栈上的连续内存

```cpp
std::array<int, 4> arr = {1, 2, 3, 4};
```

内存布局（栈帧内）：

```
栈 (Stack)
┌─────────────────────────────┐
│  arr[0] │ arr[1] │ arr[2] │ arr[3] │
│    1    │    2   │    3   │    4   │
│  地址:  1000     1004     1008    100C
└─────────────────────────────┘
```

**大小在编译期确定，直接内嵌在栈帧里，就是一块裸数组。**

------

### std::vector — 堆上的动态内存

```cpp
std::vector<int> vec = {1, 2, 3, 4};
```

内存布局：

```
栈 (Stack)                        堆 (Heap)
┌──────────────────────┐         ┌──────────────────────────────┐
│  ptr ──────────────────────────►  1  │  2  │  3  │  4  │(空余)│
│  size = 4            │         └──────────────────────────────┘
│  capacity = 8        │
└──────────────────────┘
  3个指针/整数字段（24字节）
```

vector 本身在栈上只是一个控制块，真正的数据在堆上。

------

## 二、第一层：堆分配的代价（最显著的差异）

### `new` / `malloc` 并非"免费"

当 vector 构造时，必须调用堆分配器：

```cpp
// vector 内部大致等价于：
T* data = static_cast<T*>(::operator new(n * sizeof(T)));
```

堆分配器在底层做了什么？

```
1. 进入 malloc（glibc ptmalloc / jemalloc / tcmalloc）
2. 查找合适的 free chunk（可能需要遍历 bin 链表）
3. 如果没有合适的 chunk → 调用 brk() 或 mmap() 系统调用
4. 更新内部元数据（header、footer、链表指针）
5. 返回指针
```

**典型开销：**

- 快路径（thread-local cache 命中）：~20-50ns
- 慢路径（需要系统调用）：~1000ns+
- `std::array` 栈分配：**0ns**（编译期确定，仅移动栈指针，一条指令）

------

## 三、第二层：指针间接寻址（Pointer Indirection）

### array 的访问

```cpp
arr[i]
// 汇编：
// mov eax, [rbp - 16 + i*4]   // 直接从栈地址读取，一次内存访问
```

### vector 的访问

```cpp
vec[i]
// 汇编：
// mov rax, [rbp - 32]          // 第1次：读取 ptr（栈→ptr）
// mov eax, [rax + i*4]         // 第2次：读取数据（ptr→堆）
```

vector 比 array **多一次内存间接寻址**。

虽然现代 CPU 有强大的寄存器缓存，但在循环中每次都要维护这个指针，且指针本身有时会被 spill 到内存。

------

## 四、第三层：缓存局部性（Cache Locality）——真正重要的地方

### CPU 缓存体系

```
寄存器    ~0.3ns
L1 Cache  ~1ns    (32KB-64KB，每次加载 64 字节 cache line)
L2 Cache  ~5ns    (256KB-1MB)
L3 Cache  ~20ns   (8MB-32MB)
主内存    ~100ns
堆内存的数据需要经历这整条链路
```

### std::array 的缓存行为

```
栈上已有数据，在函数调用时大概率已经在 L1/L2 Cache 中。
局部变量天然具有空间和时间局部性。

arr 的相邻元素必然在同一（或相邻）cache line 中：
[  arr[0] arr[1] arr[2] arr[3] arr[4] arr[5] arr[6] arr[7]  ]
 ←────────────── 64 字节 cache line ──────────────────►
```

### std::vector 的缓存行为

```
控制块在栈上（L1 friendly）
数据块在堆上（可能在 cache 的任何位置）

如果你有多个 vector：
  vec1 → 堆地址 0x7f8a1000
  vec2 → 堆地址 0x2b3c4000  ← 完全不同的 cache line！

访问 vec1 后访问 vec2，必然 cache miss。
```

------

## 五、第四层：编译器优化——差异最深刻的地方

### 编译器对 array 的视角

```cpp
std::array<int, 4> arr = {1, 2, 3, 4};
int sum = arr[0] + arr[1] + arr[2] + arr[3];
```

因为大小是编译期常量，编译器可以：

**1. 完全展开循环（Loop Unrolling）**

```cpp
// for (auto x : arr) sum += x;
// 编译器直接展开成：
sum = arr[0] + arr[1] + arr[2] + arr[3];
// 零循环开销！
```

**2. SIMD 向量化**

```
编译器知道有 4 个 int，恰好 128 bits：
movdqu xmm0, [arr_addr]          // 一次加载 4 个 int
phaddd xmm0, xmm0                // SIMD 水平加法
// 4个加法 → 1条指令！
```

**3. 常量折叠（Constant Folding）**

```cpp
constexpr std::array<int, 4> arr = {1, 2, 3, 4};
// 编译器直接在编译期算出结果，运行时根本不存在这段代码！
```

### 编译器对 vector 的视角

```cpp
std::vector<int> vec = {1, 2, 3, 4};
// 编译器必须假设：
// - 大小运行时才知道
// - 数据在堆上某个不确定地址
// - 可能存在 aliasing（别名）问题
```

由于大小不是编译期常量，很多优化无法进行，或效果大打折扣。

------

## 六、第五层：析构与生命周期开销

```cpp
// array：离开作用域，栈指针回退，0开销
{
    std::array<int, 1000> arr;
} // ← 一条 add rsp, 4000 指令，完成！

// vector：离开作用域，必须调用析构函数
{
    std::vector<int> vec(1000);
} // ← 调用 free()/delete[] 归还堆内存，涉及堆元数据更新
```

------

## 七、实际汇编对比（最直观）

```cpp
// 测试代码
int sum_array() {
    std::array<int, 4> a = {1, 2, 3, 4};
    int s = 0;
    for (auto x : a) s += x;
    return s;
}

int sum_vector() {
    std::vector<int> v = {1, 2, 3, 4};
    int s = 0;
    for (auto x : v) s += x;
    return s;
}
```

**`sum_array` 的 -O2 汇编（GCC）：**

```asm
sum_array():
    mov eax, 10     ; 编译器直接算出 1+2+3+4=10
    ret             ; 两条指令！！
```

**`sum_vector` 的 -O2 汇编（GCC）：**

```asm
sum_vector():
    push   rbx
    sub    rsp, 48
    ; ... 调用 operator new ...
    ; ... 初始化 vector 控制块 ...
    ; ... 循环遍历 ...
    ; ... 调用 operator delete ...
    ; 约 40+ 条指令
    pop    rbx
    ret
```

差距是**量级级别**的。

------

## 八、总结：性能差异全景图

| 维度           | std::array               | std::vector          |
| -------------- | ------------------------ | -------------------- |
| **内存分配**   | 零开销（栈）             | 堆分配（~20-1000ns） |
| **内存访问**   | 直接寻址（1次）          | 指针间接（2次）      |
| **缓存行为**   | 天然热缓存               | 可能冷缓存           |
| **编译器优化** | 循环展开、SIMD、常量折叠 | 受限                 |
| **析构开销**   | 零（栈回退）             | free() 调用          |
| **大小信息**   | 编译期已知               | 运行期才知           |

------

## 九、什么时候 vector 不可避免？

性能差异客观存在，但 `array` 的核心限制是**大小必须编译期确定**。凡是需要运行时动态大小的场景（用户输入、文件读取、不确定数量的数据），就只能用 `vector`。

> **经验准则：** 如果大小在编译期已知且较小（几个到几百个元素），优先 `std::array`；需要动态大小才用 `std::vector`。不要用 `vector` 只是因为"习惯"。