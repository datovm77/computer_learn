# C++ STL `map` 容器完全教程

---

## 📚 前言

`map` 是 C++ STL 中最重要的**关联容器**之一。它存储的是 **键值对（key-value pair）**，并且会根据**键（key）自动排序**。你可以把它想象成一本**字典**：通过"单词"（key）快速找到"释义"（value）。

> **头文件：** `#include <map>`
>
> **命名空间：** `std`

在开始之前，我们先理解一个核心概念——**`pair`（对组）**：

```cpp name=pair_basic.cpp
#include <iostream>
#include <utility> // pair 所在头文件，map 会自动包含
using namespace std;

int main()
{
    // pair 就是把两个值"捆绑"在一起
    pair<string, int> p("张三", 90);

    cout << "姓名：" << p.first  << endl;  // 访问第一个元素
    cout << "成绩：" << p.second << endl;  // 访问第二个元素

    return 0;
}
```

**输出：**

```
姓名：张三
成绩：90
```

> 🔑 **核心要点：** `map` 中的每一个元素，本质上就是一个 `pair<const Key, Value>`。

---

## 第一章：map 容器的构造和赋值

### 1.1 什么是 map？

| 特性       | 说明                                                  |
| ---------- | ----------------------------------------------------- |
| 存储内容   | 键值对 `pair<key, value>`                             |
| 排序规则   | 根据 **key** 自动升序排序（默认）                     |
| key 唯一性 | **不允许重复的 key**（如需重复 key，使用 `multimap`） |
| 底层结构   | **红黑树**（一种自平衡二叉搜索树）                    |

可以用一张图来理解：

```
        map<int, string>
        
     按 key 自动排序后：
     
     key(1) --> value("张三")
     key(2) --> value("李四")
     key(3) --> value("王五")
     key(5) --> value("赵六")
```

### 1.2 构造函数详解

`map` 提供了多种构造方式，我们从最简单的开始：

#### ① 默认构造（最常用）

```cpp name=map_construct_default.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    // 创建一个空的 map
    // key 类型为 int，value 类型为 string
    map<int, string> m;

    // 此时 m 是空的，里面没有任何元素
    cout << "map 的大小：" << m.size() << endl;  // 输出：0

    return 0;
}
```

> 📝 **语法模板：** `map<key类型, value类型> 变量名;`

#### ② 初始化列表构造（C++11 起）

```cpp name=map_construct_initlist.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    // 使用初始化列表直接赋初值
    // 每个元素用 {key, value} 的形式书写
    map<int, string> m = {
        {1, "张三"},
        {3, "王五"},
        {2, "李四"},
        {5, "赵六"}
    };

    // 遍历输出 —— 注意观察：输出会按 key 自动排序！
    for (auto it = m.begin(); it != m.end(); it++)
    {
        //       ↓ key        ↓ value
        cout << it->first << " : " << it->second << endl;
    }

    return 0;
}
```

**输出：**
```
1 : 张三
2 : 李四
3 : 王五
5 : 赵六
```

> ⚠️ **注意：** 虽然我们插入时顺序是 1, 3, 2, 5，但输出时自动按 key 排好了 1, 2, 3, 5。这就是 map 的**自动排序**特性！

#### ③ 拷贝构造

```cpp name=map_construct_copy.cpp
#include <iostream>
#include <map>
using namespace std;

void printMap(const map<int, string>& m)
{
    for (auto it = m.begin(); it != m.end(); it++)
    {
        cout << "  key=" << it->first << ", value=" << it->second << endl;
    }
    cout << endl;
}

int main()
{
    map<int, string> m1 = {
        {1, "张三"},
        {2, "李四"},
        {3, "王五"}
    };

    // 拷贝构造：用已有的 m1 来创建 m2
    // m2 是 m1 的一份完整拷贝（深拷贝）
    map<int, string> m2(m1);

    cout << "m1 的内容：" << endl;
    printMap(m1);

    cout << "m2 的内容（拷贝自 m1）：" << endl;
    printMap(m2);

    return 0;
}
```

**输出：**
```
m1 的内容：
  key=1, value=张三
  key=2, value=李四
  key=3, value=王五

m2 的内容（拷贝自 m1）：
  key=1, value=张三
  key=2, value=李四
  key=3, value=王五
```

#### ④ 迭代器区间构造

```cpp name=map_construct_iterator.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m1 = {
        {1, "张三"},
        {2, "李四"},
        {3, "王五"},
        {4, "赵六"},
        {5, "孙七"}
    };

    // 用 m1 的一部分区间 [begin, end) 来构造 m2
    // 这里取 m1 的前 3 个元素
    auto it_begin = m1.begin();   // 指向第 1 个元素
    auto it_end   = m1.begin();
    advance(it_end, 3);           // 向前移动 3 步，指向第 4 个元素

    map<int, string> m2(it_begin, it_end);

    cout << "m2 的内容（m1 的前3个元素）：" << endl;
    for (const auto& kv : m2)
    {
        cout << "  " << kv.first << " : " << kv.second << endl;
    }

    return 0;
}
```

**输出：**
```
m2 的内容（m1 的前3个元素）：
  1 : 张三
  2 : 李四
  3 : 王五
```

### 1.3 赋值操作

#### ① `operator=` 赋值

```cpp name=map_assign_operator.cpp
#include <iostream>
#include <map>
using namespace std;

void printMap(const map<int, string>& m)
{
    for (const auto& kv : m)
    {
        cout << "  " << kv.first << " : " << kv.second << endl;
    }
    cout << endl;
}

int main()
{
    map<int, string> m1 = {
        {1, "Apple"},
        {2, "Banana"},
        {3, "Cherry"}
    };

    map<int, string> m2;

    // 使用 = 进行赋值
    m2 = m1;

    cout << "赋值后 m2 的内容：" << endl;
    printMap(m2);

    // 修改 m1 不会影响 m2（深拷贝）
    m1[1] = "Avocado";
    cout << "修改 m1 后：" << endl;
    cout << "m1[1] = " << m1[1] << endl;
    cout << "m2[1] = " << m2[1] << " （不受影响）" << endl;

    return 0;
}
```

**输出：**
```
赋值后 m2 的内容：
  1 : Apple
  2 : Banana
  3 : Cherry

修改 m1 后：
m1[1] = Avocado
m2[1] = Apple （不受影响）
```

### 1.4 构造和赋值总结表

| 方式       | 代码示例                                    | 说明             |
| ---------- | ------------------------------------------- | ---------------- |
| 默认构造   | `map<int,string> m;`                        | 创建空 map       |
| 初始化列表 | `map<int,string> m = {{1,"a"},{2,"b"}};`    | 直接给初值       |
| 拷贝构造   | `map<int,string> m2(m1);`                   | 拷贝另一个 map   |
| 区间构造   | `map<int,string> m2(m1.begin(), m1.end());` | 用迭代器区间构造 |
| `=` 赋值   | `m2 = m1;`                                  | 将 m1 赋值给 m2  |

---

## 第二章：map 容器的大小和交换

### 2.1 大小相关操作

| 函数        | 返回值   | 说明                               |
| ----------- | -------- | ---------------------------------- |
| `m.size()`  | `size_t` | 返回 map 中键值对的个数            |
| `m.empty()` | `bool`   | 判断 map 是否为空（空返回 `true`） |

#### 完整示例

```cpp name=map_size.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m;

    // ========== empty() ==========
    if (m.empty())
    {
        cout << "m 当前是空的" << endl;
    }
    cout << "当前大小：" << m.size() << endl;  // 0

    cout << "----------------------------" << endl;

    // 插入一些数据
    m.insert({1, "语文"});
    m.insert({2, "数学"});
    m.insert({3, "英语"});

    // ========== size() ==========
    if (!m.empty())
    {
        cout << "m 不为空" << endl;
        cout << "当前大小：" << m.size() << endl;  // 3
    }

    cout << "----------------------------" << endl;

    // 插入重复的 key，不会成功！
    m.insert({1, "物理"});  // key=1 已存在，插入失败

    cout << "尝试插入重复 key 后，大小：" << m.size() << endl;  // 仍然是 3
    cout << "key=1 对应的值：" << m[1] << endl;  // 仍然是"语文"

    return 0;
}
```

**输出：**
```
m 当前是空的
当前大小：0
----------------------------
m 不为空
当前大小：3
----------------------------
尝试插入重复 key 后，大小：3
key=1 对应的值：语文
```

> 🔑 **重要发现：** 当插入重复的 key 时，`insert` 不会覆盖原有值，插入直接失败，size 不变！

### 2.2 交换操作 `swap()`

`swap()` 用于**交换两个 map 的全部内容**。

```cpp name=map_swap.cpp
#include <iostream>
#include <map>
using namespace std;

void printMap(const string& name, const map<int, string>& m)
{
    cout << name << "（size=" << m.size() << "）：" << endl;
    for (const auto& kv : m)
    {
        cout << "  " << kv.first << " => " << kv.second << endl;
    }
    cout << endl;
}

int main()
{
    map<int, string> m1 = {
        {1, "C++"},
        {2, "Java"},
        {3, "Python"}
    };

    map<int, string> m2 = {
        {10, "篮球"},
        {20, "足球"}
    };

    cout << "===== 交换前 =====" << endl;
    printMap("m1", m1);
    printMap("m2", m2);

    // 交换 m1 和 m2 的内容
    m1.swap(m2);

    cout << "===== 交换后 =====" << endl;
    printMap("m1", m1);
    printMap("m2", m2);

    return 0;
}
```

**输出：**
```
===== 交换前 =====
m1（size=3）：
  1 => C++
  2 => Java
  3 => Python

m2（size=2）：
  10 => 篮球
  20 => 足球

===== 交换后 =====
m1（size=2）：
  10 => 篮球
  20 => 足球

m2（size=3）：
  1 => C++
  2 => Java
  3 => Python
```

> 📝 `swap` 的底层实现非常高效——它只是交换了两棵红黑树的根节点指针，而**不是逐个拷贝元素**，所以时间复杂度是 **O(1)**。

### 2.3 大小和交换总结表

| 函数          | 作用                | 时间复杂度 |
| ------------- | ------------------- | ---------- |
| `m.size()`    | 返回元素个数        | O(1)       |
| `m.empty()`   | 判断是否为空        | O(1)       |
| `m1.swap(m2)` | 交换两个 map 的内容 | O(1)       |

---

## 第三章：map 容器的插入和删除

这是 map 最核心、最常用的部分。map 提供了**多种插入方式**，每种都有细微差别，初学者务必仔细区分。

### 3.1 四种插入方式详解

#### ① `insert(pair<>)` —— 最经典的方式

```cpp name=map_insert_pair.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m;

    // 方式一：构造匿名 pair 对象
    m.insert(pair<int, string>(1, "张三"));

    // 方式二：使用 make_pair（自动推导类型，更简洁）
    m.insert(make_pair(2, "李四"));

    // 方式三：使用花括号初始化（C++11，最推荐！）
    m.insert({3, "王五"});

    for (const auto& kv : m)
    {
        cout << kv.first << " : " << kv.second << endl;
    }

    return 0;
}
```

**输出：**
```
1 : 张三
2 : 李四
3 : 王五
```

#### ② `insert` 的返回值 —— 判断是否插入成功

这是一个非常重要但常被忽略的知识点：

```cpp name=map_insert_return.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m;

    // insert 的返回值类型是 pair<iterator, bool>
    //   - first  : 指向插入位置（或已存在元素位置）的迭代器
    //   - second : 插入是否成功（true=成功，false=key已存在）

    auto ret1 = m.insert({1, "张三"});
    if (ret1.second)
    {
        cout << "第1次插入 key=1 成功！" << endl;
    }

    auto ret2 = m.insert({1, "李四"});  // key=1 已存在
    if (!ret2.second)
    {
        cout << "第2次插入 key=1 失败！key 已存在。" << endl;
        cout << "已存在的值为：" << ret2.first->second << endl;
    }

    return 0;
}
```

**输出：**
```
第1次插入 key=1 成功！
第2次插入 key=1 失败！key 已存在。
已存在的值为：张三
```

> 🔑 **核心结论：`insert` 不会覆盖已有的 key 对应的 value！**

#### ③ `operator[]` —— 用下标访问和插入

```cpp name=map_insert_bracket.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m;

    // 使用 [] 插入数据
    m[1] = "张三";
    m[2] = "李四";
    m[3] = "王五";

    // 使用 [] 访问数据
    cout << "m[1] = " << m[1] << endl;
    cout << "m[2] = " << m[2] << endl;

    // 【重点】[] 会覆盖已有的值！（和 insert 不同！）
    m[1] = "赵六";
    cout << "覆盖后 m[1] = " << m[1] << endl;  // 赵六

    // 【危险】访问不存在的 key 会自动创建该 key！
    cout << "m[100] = \"" << m[100] << "\"" << endl;  // 输出空字符串
    cout << "此时 size = " << m.size() << endl;        // 变成了 4！

    cout << endl << "当前 map 所有内容：" << endl;
    for (const auto& kv : m)
    {
        cout << "  " << kv.first << " => \"" << kv.second << "\"" << endl;
    }

    return 0;
}
```

**输出：**
```
m[1] = 张三
m[2] = 李四
覆盖后 m[1] = 赵六
m[100] = ""
此时 size = 4

当前 map 所有内容：
  1 => "赵六"
  2 => "李四"
  3 => "王五"
  100 => ""
```

> ⚠️ **特别警告：** `m[key]` 在 key 不存在时，会**自动插入**一个默认值的元素！
> - `int` 类型默认值为 `0`
> - `string` 类型默认值为 `""`（空字符串）
> - 这是初学者最容易踩的坑！如果你只是想查找，请用 `find()` 或 `count()`。

#### ④ `emplace()` —— 原地构造（C++11）

```cpp name=map_emplace.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m;

    // emplace 直接在 map 内部原地构造 pair
    // 避免了先构造临时 pair 再拷贝的开销
    m.emplace(1, "张三");
    m.emplace(2, "李四");
    m.emplace(3, "王五");

    // 和 insert 一样，key 重复时不会覆盖
    m.emplace(1, "赵六");

    for (const auto& kv : m)
    {
        cout << kv.first << " : " << kv.second << endl;
    }
    // key=1 仍然是"张三"

    return 0;
}
```

**输出：**
```
1 : 张三
2 : 李四
3 : 王五
```

### 3.2 四种插入方式对比

| 方式                | 代码                         | key 重复时的行为   | 性能             |
| ------------------- | ---------------------------- | ------------------ | ---------------- |
| `insert(pair)`      | `m.insert({1,"a"})`          | ❌ 不覆盖，插入失败 | 好               |
| `insert(make_pair)` | `m.insert(make_pair(1,"a"))` | ❌ 不覆盖，插入失败 | 好               |
| `operator[]`        | `m[1] = "a"`                 | ✅ **覆盖**旧值     | 好               |
| `emplace`           | `m.emplace(1,"a")`           | ❌ 不覆盖，插入失败 | 最好（原地构造） |

> 💡 **实际开发建议：**
> - 想要"不存在就插入，存在就跳过" → 用 `insert` 或 `emplace`
> - 想要"不存在就插入，存在就更新" → 用 `operator[]`

### 3.3 删除操作

#### ① `erase()` —— 三种重载

```cpp name=map_erase.cpp
#include <iostream>
#include <map>
using namespace std;

void printMap(const map<int, string>& m)
{
    for (const auto& kv : m)
    {
        cout << "  " << kv.first << " => " << kv.second << endl;
    }
    cout << "  (size=" << m.size() << ")" << endl << endl;
}

int main()
{
    map<int, string> m = {
        {1, "语文"},
        {2, "数学"},
        {3, "英语"},
        {4, "物理"},
        {5, "化学"}
    };

    cout << "初始状态：" << endl;
    printMap(m);

    // ==============================
    // 方式一：通过 key 删除（最常用）
    // ==============================
    m.erase(3);  // 删除 key=3 的元素
    cout << "删除 key=3 后：" << endl;
    printMap(m);

    // 删除不存在的 key，不会报错，什么也不发生
    m.erase(100);
    cout << "删除不存在的 key=100 后（无变化）：" << endl;
    printMap(m);

    // ==============================
    // 方式二：通过迭代器删除
    // ==============================
    auto it = m.find(2);  // 先找到 key=2 的迭代器
    if (it != m.end())
    {
        m.erase(it);  // 删除该迭代器指向的元素
    }
    cout << "通过迭代器删除 key=2 后：" << endl;
    printMap(m);

    // ==============================
    // 方式三：通过迭代器区间删除
    // ==============================
    // 删除从 begin 到 end 的所有元素（等价于 clear）
    m.erase(m.begin(), m.end());
    cout << "区间删除所有元素后：" << endl;
    printMap(m);

    return 0;
}
```

**输出：**
```
初始状态：
  1 => 语文
  2 => 数学
  3 => 英语
  4 => 物理
  5 => 化学
  (size=5)

删除 key=3 后：
  1 => 语文
  2 => 数学
  4 => 物理
  5 => 化学
  (size=4)

删除不存在的 key=100 后（无变化）：
  1 => 语文
  2 => 数学
  4 => 物理
  5 => 化学
  (size=4)

通过迭代器删除 key=2 后：
  1 => 语文
  4 => 物理
  5 => 化学
  (size=3)

区间删除所有元素后：
  (size=0)
```

#### ② `clear()` —— 清空所有元素

```cpp name=map_clear.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m = {
        {1, "A"},
        {2, "B"},
        {3, "C"}
    };

    cout << "清空前 size = " << m.size() << endl;   // 3

    m.clear();

    cout << "清空后 size = " << m.size() << endl;   // 0
    cout << "是否为空：" << (m.empty() ? "是" : "否") << endl;  // 是

    return 0;
}
```

#### ③ 在遍历中安全删除

这是一个非常经典的面试和实战问题：

```cpp name=map_erase_while_iterate.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m = {
        {1, "语文"},
        {2, "数学"},
        {3, "英语"},
        {4, "物理"},
        {5, "化学"}
    };

    // 需求：删除所有 key 为偶数的元素
    
    // ❌ 错误写法（会导致迭代器失效！）：
    // for (auto it = m.begin(); it != m.end(); it++)
    // {
    //     if (it->first % 2 == 0)
    //         m.erase(it);  // it 已失效，之后 it++ 是未定义行为！
    // }

    // ✅ 正确写法：
    for (auto it = m.begin(); it != m.end(); /* 注意这里没有 it++ */)
    {
        if (it->first % 2 == 0)
        {
            it = m.erase(it);  // erase 返回下一个有效迭代器
        }
        else
        {
            it++;  // 不删除时才手动前进
        }
    }

    cout << "删除偶数 key 后：" << endl;
    for (const auto& kv : m)
    {
        cout << "  " << kv.first << " => " << kv.second << endl;
    }

    return 0;
}
```

**输出：**
```
删除偶数 key 后：
  1 => 语文
  3 => 英语
  5 => 化学
```

### 3.4 插入和删除总结表

| 操作         | 函数                     | 说明                       |
| ------------ | ------------------------ | -------------------------- |
| 插入         | `m.insert({key, value})` | key 存在则失败             |
| 插入         | `m[key] = value`         | key 存在则覆盖             |
| 插入         | `m.emplace(key, value)`  | key 存在则失败（原地构造） |
| 按 key 删除  | `m.erase(key)`           | 删除指定 key 的元素        |
| 按迭代器删除 | `m.erase(it)`            | 删除迭代器指向的元素       |
| 区间删除     | `m.erase(begin, end)`    | 删除 [begin, end) 区间     |
| 清空         | `m.clear()`              | 删除所有元素               |

---

## 第四章：map 容器的查找和统计

### 4.1 `find()` —— 查找指定 key

```cpp name=map_find.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m = {
        {1, "语文"},
        {2, "数学"},
        {3, "英语"}
    };

    // find(key) 返回一个迭代器
    //   - 找到了 → 返回指向该元素的迭代器
    //   - 没找到 → 返回 m.end()

    // ===== 查找存在的 key =====
    auto it = m.find(2);
    if (it != m.end())
    {
        cout << "找到了！key=" << it->first 
             << ", value=" << it->second << endl;
    }
    else
    {
        cout << "没找到 key=2" << endl;
    }

    // ===== 查找不存在的 key =====
    auto it2 = m.find(99);
    if (it2 != m.end())
    {
        cout << "找到了！" << endl;
    }
    else
    {
        cout << "没找到 key=99" << endl;
    }

    return 0;
}
```

**输出：**
```
找到了！key=2, value=数学
没找到 key=99
```

> 🔑 **为什么用 `find()` 而不是 `m[key]` 来查找？**
> 因为 `m[key]` 在 key 不存在时会**自动插入一个默认值**，这不是我们想要的！
> `find()` 则安全得多——不存在时只会返回 `end()`，不会修改 map。

### 4.2 `count()` —— 统计指定 key 的个数

```cpp name=map_count.cpp
#include <iostream>
#include <map>
using namespace std;

int main()
{
    map<int, string> m = {
        {1, "语文"},
        {2, "数学"},
        {3, "英语"}
    };

    // count(key) 返回 key 出现的次数
    // 对于 map，结果只会是 0 或 1（因为 key 唯一）
    // 对于 multimap，结果可能大于 1

    cout << "key=2 出现的次数：" << m.count(2) << endl;   // 1
    cout << "key=99 出现的次数：" << m.count(99) << endl;  // 0

    // count 常用于快速判断 key 是否存在
    int targetKey = 3;
    if (m.count(targetKey))  // 非0即true
    {
        cout << "key=" << targetKey << " 存在！" << endl;
    }
    else
    {
        cout << "key=" << targetKey << " 不存在！" << endl;
    }

    return 0;
}
```

**输出：**
```
key=2 出现的次数：1
key=99 出现的次数：0
key=3 存在！
```

### 4.3 `find()` vs `count()` vs `operator[]` 对比

| 方法         | 返回值       | key 不存在时       | 是否修改 map | 适用场景          |
| ------------ | ------------ | ------------------ | ------------ | ----------------- |
| `find(key)`  | 迭代器       | 返回 `end()`       | ❌ 不修改     | 需要获取 value 时 |
| `count(key)` | 0 或 1       | 返回 0             | ❌ 不修改     | 只判断存不存在时  |
| `m[key]`     | value 的引用 | **自动插入默认值** | ⚠️ **会修改** | 确定要读写时      |

### 4.4 综合实战：学生成绩管理系统

让我们用一个完整的例子来综合运用本教程的所有知识：

```cpp name=student_score_system.cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

// 打印所有学生成绩
void printAll(const map<string, int>& scores)
{
    cout << "\n========== 成绩单 ==========" << endl;
    if (scores.empty())
    {
        cout << "  （暂无数据）" << endl;
    }
    else
    {
        for (const auto& kv : scores)
        {
            cout << "  " << kv.first << " : " << kv.second << " 分" << endl;
        }
        cout << "  共 " << scores.size() << " 名学生" << endl;
    }
    cout << "============================\n" << endl;
}

int main()
{
    map<string, int> scores;

    // ===== 1. 插入学生成绩 =====
    cout << "【步骤1】录入学生成绩" << endl;
    scores.insert({"Alice", 92});
    scores.insert({"Bob", 85});
    scores.insert({"Charlie", 78});
    scores.emplace("David", 95);
    scores["Eve"] = 88;
    printAll(scores);
    // 注意：输出按姓名（string）的字典序自动排序！

    // ===== 2. 查找某个学生 =====
    cout << "【步骤2】查找学生 Bob" << endl;
    auto it = scores.find("Bob");
    if (it != scores.end())
    {
        cout << "  找到了！" << it->first << " 的成绩是 " << it->second << " 分" << endl;
    }

    // ===== 3. 修改成绩 =====
    cout << "\n【步骤3】将 Charlie 的成绩改为 90" << endl;
    scores["Charlie"] = 90;  // [] 会覆盖
    printAll(scores);

    // ===== 4. 判断学生是否存在 =====
    cout << "【步骤4】判断 Frank 是否存在" << endl;
    if (scores.count("Frank") == 0)
    {
        cout << "  Frank 不在成绩单中" << endl;
    }

    // ===== 5. 删除某个学生 =====
    cout << "\n【步骤5】删除 David" << endl;
    scores.erase("David");
    printAll(scores);

    // ===== 6. 删除不及格的学生（假设 < 80 为不及格）=====
    // 先加一个不及格的学生来演示
    scores["Frank"] = 55;
    scores["Grace"] = 42;
    cout << "【步骤6】加入 Frank(55分)、Grace(42分)，然后删除所有不及格学生" << endl;
    printAll(scores);

    for (auto iter = scores.begin(); iter != scores.end(); )
    {
        if (iter->second < 80)
        {
            cout << "  删除不及格学生：" << iter->first 
                 << "（" << iter->second << "分）" << endl;
            iter = scores.erase(iter);
        }
        else
        {
            iter++;
        }
    }
    printAll(scores);

    // ===== 7. 交换 =====
    cout << "【步骤7】与另一份空成绩单交换" << endl;
    map<string, int> emptyScores;
    scores.swap(emptyScores);

    cout << "scores 交换后：";
    printAll(scores);

    cout << "emptyScores 交换后：";
    printAll(emptyScores);

    return 0;
}
```

**输出：**
```
【步骤1】录入学生成绩

========== 成绩单 ==========
  Alice : 92 分
  Bob : 85 分
  Charlie : 78 分
  David : 95 分
  Eve : 88 分
  共 5 名学生
============================

【步骤2】查找学生 Bob
  找到了！Bob 的成绩是 85 分

【步骤3】将 Charlie 的成绩改为 90

========== 成绩单 ==========
  Alice : 92 分
  Bob : 85 分
  Charlie : 90 分
  David : 95 分
  Eve : 88 分
  共 5 名学生
============================

【步骤4】判断 Frank 是否存在
  Frank 不在成绩单中

【步骤5】删除 David

========== 成绩单 ==========
  Alice : 92 分
  Bob : 85 分
  Charlie : 90 分
  Eve : 88 分
  共 4 名学生
============================

【步骤6】加入 Frank(55分)、Grace(42分)，然后删除所有不及格学生

========== 成绩单 ==========
  Alice : 92 分
  Bob : 85 分
  Charlie : 90 分
  Eve : 88 分
  Frank : 55 分
  Grace : 42 分
  共 6 名学生
============================

  删除不及格学生：Frank（55分）
  删除不及格学生：Grace（42分）

========== 成绩单 ==========
  Alice : 92 分
  Bob : 85 分
  Charlie : 90 分
  Eve : 88 分
  共 4 名学生
============================

【步骤7】与另一份空成绩单交换
scores 交换后：
========== 成绩单 ==========
  （暂无数据）
============================

emptyScores 交换后：
========== 成绩单 ==========
  Alice : 92 分
  Bob : 85 分
  Charlie : 90 分
  Eve : 88 分
  共 4 名学生
============================
```

---

## 第五章：自定义排序 (⭐ 必考重点)

`map` 默认情况按照 key 的**升序**（从小到大）排序，相当于 `map<Key, Value, less<Key>>`。如果有特殊需求（如降序排列，或自定义类型作为 key），我们需要提供**自定义排序规则**。

### 5.1 降序排列

要实现降序排序，我们只需要在模板参数中提供 `greater<Type>` 即可：

```cpp name=map_sort_desc.cpp
#include <iostream>
#include <map>
#include <functional> // greater 所在的头文件
using namespace std;

int main()
{
    // 使用 greater<int> 实现降序
    map<int, string, greater<int>> m = {
        {1, "A"}, {5, "B"}, {3, "C"}
    };

    for (const auto& kv : m) {
        cout << kv.first << " : " << kv.second << endl;
    }
    // 输出: 5 : B \n 3 : C \n 1 : A
    return 0;
}
```

### 5.2 自定义类型做 key

当 `struct` 或 `class` 作为 `map` 的 key 时，必须提供排序规则，否则编译器不知道该如何让红黑树节点排序！
你可以通过**仿函数（Functor）**来完成：

```cpp name=map_custom_key.cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

struct Student {
    string name;
    int age;
};

// 自定义比较函数（仿函数）
// 要求：必须重载 operator()，且通常加 const 修饰
struct StudentCmp {
    bool operator()(const Student& a, const Student& b) const {
        return a.age < b.age; // 按年龄升序排
    }
};

int main()
{
    // 第三个参数传入我们自定义的仿函数 StudentCmp
    map<Student, int, StudentCmp> scores;

    scores.insert({{"张三", 18}, 90});
    scores.insert({{"李四", 22}, 85});
    scores.insert({{"王五", 20}, 95});

    for (const auto& kv : scores) {
        cout << kv.first.name << " (" << kv.first.age << "岁) : " << kv.second << "分" << endl;
    }
    // 输出会按照 18, 20, 22 岁的顺序排列

    return 0;
}
```

---

## 第六章：安全访问与现代特性

随着 C++ 标准的发展，`map` 也有了许多更好用、更安全的特性。

### 6.1 `at()`：安全的下标访问 (C++11)

我们之前学过，`operator[]` 当 key 不存在时会自动插入。那如果你只想**安全地查看**，又怕不小心修改了 `map` 怎么办？用 `at()`！

```cpp name=map_at.cpp
#include <iostream>
#include <map>
#include <stdexcept>
using namespace std;

int main() {
    map<int, string> m = {{1, "A"}};

    cout << m.at(1) << endl; // 正常输出 A
    
    try {
        cout << m.at(99) << endl; // key=99 不存在，抛出 out_of_range 异常
    } catch (const out_of_range& e) {
        cout << "捕获异常：" << e.what() << endl;
        cout << "此时 size=" << m.size() << "（没有像 [] 那样插入新元素！）" << endl;
    }

    return 0;
}
```

> 🔑 **对比：** `at()` 可以在 **const map** 上使用，而 `operator[]` 坚决不可以（因为它可能会修改 map）。

### 6.2 `contains()`：判断 key 是否存在 (C++20)

以前判断 key 存不存在，要写 `if (m.count(key) > 0)` 或 `if (m.find(key) != m.end())`。C++20 提供了一针见血的成员函数：

```cpp name=map_contains.cpp
map<int, string> m = {{1, "A"}};
if (m.contains(1)) {
    cout << "存在！" << endl;
}
```

### 6.3 结构化绑定 (C++17)

遍历 `map` 时不用再到处找 `->first` 和 `->second` 啦，代码更易读！

```cpp name=map_structured_binding.cpp
map<int, string> m = {{1, "A"}, {2, "B"}};

// C++17 特性：直接将 first 绑定到 key，second 绑定到 val
for (const auto& [key, val] : m) {
    cout << key << " : " << val << endl;
}
```

---

## 第七章：C++17 高级插入方式

针对日常开发中"如果不存在就...，如果存在就..."的痛点，C++17 引入了两个神作：

### 7.1 `insert_or_assign()`：插入或更新

- **如果不存**：插入新元素。
- **如果已存在**：更新对应的 value。
它比先判存再 `m[key]=val` 更高效，且会返回一个 pair 告诉你究竟是发生了**插入**还是**赋值**。

### 7.2 `try_emplace()`：尝试原地构造

`emplace` 即使在 key 已经存在（插入失败）时，也会先构造出包含 value 的临时对象，浪费性能。
`try_emplace` **只有在 key 确实不存在时**，才会去构造 value，是目前安全且性能最高的单元素插入方式！

### 7.3 全系单点插入方式对比

| 方式               | key 不存在 | key 已存在     | 存在时是否构造冗余 value？ |
|--------------------| ---------- | -------------- | -------------------------- |
| `insert`           | 插入       | 失败           | ✅ 会                       |
| `emplace`          | 插入       | 失败           | ✅ 会                       |
| `operator[]`       | 插入默认值 | **覆盖旧值**   | —                          |
| `insert_or_assign` | 插入       | **覆盖旧值**   | —                          |
| `try_emplace`      | 插入       | 失败           | ❌ **不会（性能最优）**     |

---

## 第八章：范围查找

由于 `map` 底层是红黑树（有序结构），我们可以非常快地找到一个区间：

- `lower_bound(key)`：返回指向 **第一个键值 >= key** 的元素的迭代器。
- `upper_bound(key)`：返回指向 **第一个键值 > key** 的元素的迭代器。
- `equal_range(key)`：一次性返回包含 `{lower_bound, upper_bound}` 的 pair。

**范围查找口诀：** `[lower_bound(a), upper_bound(b))` = 所有满足 `a <= key <= b` 的元素区间。

---

## 第九章：multimap 简介

`map` 不允许重复 key，如果在业务中遇到"一对多"的场景（如某门课有多个学生，或者同一天有多个相同的课程），应使用 `multimap`：

| 特性         | `map`          | `multimap`         |
|------------- | -------------- | ------------------ |
| key 可重复？ | ❌ 不可         | ✅ 可以             |
| `operator[]` | ✅ 支持         | ❌ 不支持 (存在歧义) |
| `at()`       | ✅ 支持         | ❌ 不支持           |

---

## 第十章：unordered_map 简介与实战

虽然 `map` 极其强大，但它底层使用了**红黑树**，这意味着每次插入、查找的时间复杂度都是 $O(\log N)$。当我们需要处理海量数据，且完全**不需要关心键的顺序**时，`map` 这份排序的开销就显得没必要了。

这时候，`unordered_map` 闪亮登场！

### 10.1 什么是 unordered_map？

`unordered_map` 翻译为“无序映射”，它的底层是通过**哈希表（Hash Table）**实现的。

> 💡 **核心重点：** `unordered_map` 的用法（包括 `insert`、`operator[]`、`find`、`count`、`size` 等等）与 `map` **几乎完全一样**！唯一的区别就是你遍历它的时候，元素是乱序的（哈希桶顺序）。

| 特性 | `map` | `unordered_map` |
| --- | --- | --- |
| **底层结构** | 红黑树 | 哈希表 |
| **排序** | ✅ 按 key 自动排序 | ❌ 完全无序 |
| **时间复杂度** | 插入/查找/删除 $O(\log N)$ | 平均 $O(1)$，最坏 $O(N)$ |
| **头文件** | `<map>` | `<unordered_map>` |
| **适用场景** | 需要数据的有序性（如范围查找） | 只需要最高效的单点查找和统计 |

> **提示：** 除非题目或业务明确要求保证数据有序，否则在只需要进行简单的 Key-Value 映射和统计时，优先选用 `unordered_map`。

### 10.2 实战演练一：LeetCode 1. 两数之和

这是在算法面试中最经典的一道题，也是体会 `unordered_map`“空间换时间”思想的最佳案例。

**题目背景：**
给定一个整数集合 `nums` 和一个目标值 `target`，请你在该集合中找出**和为目标值**的那两个整数，并返回它们的数组下标。（你可以假设每种输入只会对应一个答案，且同样的元素不能被重复利用。）

**分析：**
- 暴力解法：两层 `for` 循环嵌套，时间复杂度 $O(N^2)$。慢！
- 我们要找某个数 `x`，它满足 `x = target - nums[i]`。
- 如果我们把遍历过的数字及其下标存起来，每次只要“回头看一眼”字典里有没有刚好等于 `x` 的数字就可以了。
- 这个快速查询的需求，完美契合 `unordered_map`。

```cpp name=learn-40_two_sum.cpp
#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        // 哈希表：
        // key：数组中的数值（我们需要根据数值来查找它是否存在）
        // value：该数值在数组中的下标
        unordered_map<int, int> m;

        // 一遍边遍历，边保存字典
        for (int i = 0; i < nums.size(); ++i) {
            // 我们当前遇到 nums[i]，想找与之配对的 target - nums[i] 是否已经在字典中
            auto it = m.find(target - nums[i]);
            
            // 如果找到了
            if (it != m.end()) {
                // it->second 是配对数字的下标，i 是当前的下标
                return {it->second, i};
            }
            
            // 如果没找到，就把当前数字及下标“登记”到字典中，给后面的人用
            m.emplace(nums[i], i);
        }
        return {}; // 找不到的情况（题目保证了必有解，这里为了编译不出错）
    }
};
```

**为什么这道题它是神作？**
由于 `unordered_map` 查找平均只需 $O(1)$，我们只需**一次遍历**即可找到答案。这直接把时间复杂度从 $O(N^2)$ 降维打击到了 **$O(N)$**！

---

### 10.3 实战演练二：P1918 保龄球

我们再来看一道进阶版的工程类/竞赛类输入输出场景题目。当你需要**应对海量数据查询**时，`unordered_map` 是保命神技。

**题目背景：**
有 $n$ 个保龄球摆成一排，每个保龄球上都有一个数字标识（数字可能极大）。现在有 $q$ 次询问，每次询问一个数字求它所在的位置（从 1 开始算），如果不存在输出 0。

**分析：**
1. 试想如果 $n=100000$，$q=100000$，每次都在数组里从头找，$O(N \cdot Q)$ 绝对超时(TLE)。
2. 我们需要预先建立一张快查表：`数字标识 -> 位置`。
3. 同样，我们**毫不犹豫使用 `unordered_map`**。

下面是规范的解题写法和讲解：

```cpp name=learn-41.cpp
#include <iostream>
#include <unordered_map>

void test01()
{
    // 如果想要在算法竞赛进一步提速，可以加上面两句:
    // std::ios::sync_with_stdio(false);
    // std::cin.tie(nullptr);

    int n;
    std::cin >> n;
    
    // 构造哈希表
    // key：保龄球上面的数字
    std::unordered_map<int, int> m;
    
    // 第一步：录入所有保龄球数据
    for (int i = 0; i < n; i++)
    {
        int temp;
        std::cin >> temp;
        
        // 由于题目要求输出的位置从 1 开始算，所以 value 是 i + 1。
        // 使用 emplace 避免额外开销
        m.emplace(temp, i + 1);
    }
    
    int times; // 询问次数
    std::cin >> times;
    
    // 第二步：处理所有的狂风暴雨般的询问
    for (int i = 0; i < times; ++i)
    {
        int target;
        std::cin >> target;
        
        // 极速查找，平均时间复杂度 O(1)
        auto it = m.find(target);
        
        if (it != m.end())
        {
            // 找到了，it->first 是数字，it->second 就是位置
            std::cout << it->second << '\n';
        }
        else
        {
            // find 到底了没找到，说明没这个球，按要求输出 0
            std::cout << '0' << '\n';
        }
    }
}

int main()
{
    test01();
    return 0;
}
```

> **小结：**
> - 「两数之和」 教会了我们如何利用 `unordered_map` “边遍历边记录，避免回头路”。
> - 「保龄球问题」教会了我们面对海量查询如何 “一次建库，万次秒查”。
> 这两大法宝，是处理所有以查找为核心诉求的题目的必杀技！

---

## 附录 A：map 常用操作速查表

| 分类     | 操作         | 代码                              | 时间复杂度 |
| -------- | ------------ | --------------------------------- | ---------- |
| **构造** | 默认构造     | `map<K,V> m;`                     | —          |
|          | 初始化列表   | `map<K,V> m = {{k1,v1},{k2,v2}};` | O(n log n) |
|          | 拷贝构造     | `map<K,V> m2(m1);`                | O(n)       |
| **赋值** | 赋值运算符   | `m2 = m1;`                        | O(n)       |
| **大小** | 元素个数     | `m.size()`                        | O(1)       |
|          | 是否为空     | `m.empty()`                       | O(1)       |
| **交换** | 交换内容     | `m1.swap(m2)`                     | O(1)       |
| **插入** | insert       | `m.insert({k, v})`                | O(log n)   |
|          | emplace      | `m.emplace(k, v)`                 | O(log n)   |
|          | 下标访问     | `m[k] = v`                        | O(log n)   |
| **删除** | 按 key 删除  | `m.erase(key)`                    | O(log n)   |
|          | 按迭代器删除 | `m.erase(it)`                     | 均摊 O(1)  |
|          | 区间删除     | `m.erase(begin, end)`             | O(n)       |
|          | 清空         | `m.clear()`                       | O(n)       |
| **查找** | 查找         | `m.find(key)`                     | O(log n)   |
|          | 统计         | `m.count(key)`                    | O(log n)   |
| **访问** | 安全访问     | `m.at(key)`                       | O(log n)   |
| **判断** | 是否存在     | `m.contains(key)` (C++20)         | O(log n)   |
| **范围查**| 下界         | `m.lower_bound(key)`              | O(log n)   |
|          | 上界         | `m.upper_bound(key)`              | O(log n)   |
|          | 等值范围     | `m.equal_range(key)`              | O(log n)   |
| **进阶** | 插入或更新   | `m.insert_or_assign(k, v)` (C++17)| O(log n)   |
|          | 尝试原地构造 | `m.try_emplace(k, args)` (C++17)  | O(log n)   |

## 附录 B：map vs unordered_map 对比

| 特性           | `map`         | `unordered_map`      |
| -------------- | ------------- | -------------------- |
| 底层结构       | 红黑树        | 哈希表               |
| 元素有序性     | ✅ 按 key 有序 | ❌ 无序               |
| 查找/插入/删除 | O(log n)      | 平均 O(1)，最差 O(n) |
| 头文件         | `<map>`       | `<unordered_map>`    |
| 适用场景       | 需要有序遍历  | 只需要快速查找       |

## 附录 C：初学者常见错误总结

| 错误                              | 后果                           | 正确做法                 |
| --------------------------------- | ------------------------------ | ------------------------ |
| 用 `m[key]` 查找不存在的 key      | 自动插入默认值元素，map 被修改 | 用 `find()` 或 `count()` |
| 用 `insert` 期望更新已有 key 的值 | 插入失败，旧值不变             | 用 `m[key] = newValue`   |
| 遍历中直接 `erase(it)` 后 `it++`  | 迭代器失效，未定义行为         | `it = m.erase(it)`       |
| 忘记 map 会自动排序               | 数据顺序与插入顺序不一致       | 明确知道 map 按 key 排序 |

---

> 📌 **学完本教程，你应该掌握了：**
> 1. map 的基本概念（键值对、自动排序、key 唯一）
> 2. 四种构造方式和赋值操作
> 3. `size()`、`empty()`、`swap()` 的使用
> 4. 四种插入方式的区别（特别是 `insert` vs `operator[]`）
> 5. 三种删除方式以及遍历中安全删除的技巧
> 6. `find()` 和 `count()` 的使用与区别
> 7. 为什么不能用 `operator[]` 来做查找
> 8. 自定义排序规则（仿函数、自定义类型做 key）
> 9. `at()` 的安全访问 与 `contains()` 的现代判断
> 10. C++17 的结构化绑定、`insert_or_assign`、`try_emplace`
> 11. `lower_bound` / `upper_bound` 范围查找
> 12. multimap 与 map 的区别
> 13. `unordered_map` 的原理与实战场景应用