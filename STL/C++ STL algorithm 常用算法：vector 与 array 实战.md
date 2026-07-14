# C++ STL `<algorithm>` 常用算法：`vector` 与 `array` 实战

---

`<algorithm>` 的大多数算法只关心迭代器范围，不关心数据具体放在什么容器里。`std::vector` 和 `std::array` 都提供 `begin()`、`end()`，而且都支持随机访问迭代器，所以本页的算法可以用几乎相同的写法调用：

~~~cpp
算法名(container.begin(), container.end(), 其他参数);
~~~

本文覆盖下列 20 个算法，并把重点放在四件事上：是否改写数据、返回什么、有哪些前提，以及 `vector` 与 `array` 的实际差异。

~~~text
sort              reverse           find              count
max_element       min_element       minmax_element    copy
fill              transform         for_each          remove
unique            binary_search     lower_bound       upper_bound
next_permutation  prev_permutation  rotate            accumulate
~~~

以下片段按 C++11 编写。除“完整示例”外，代码默认已包含所需头文件；请分别放进 `main` 测试，或放入独立作用域以避免变量重名。

---

## 一、先统一三个概念

### 1.1 `vector` 和 `array` 什么时候有区别？

| 项目 | `std::vector<T>` | `std::array<T, N>` |
| --- | --- | --- |
| 元素个数 | 运行时可增减 | 固定为 `N`，不能增减 |
| 内存布局 | 连续 | 连续 |
| `begin() + n` | 支持 | 支持 |
| 典型场景 | 待处理记录、动态列表 | 固定数量的数据，例如一周 7 天 |
| `remove` / `unique` 后 | 可调用 `erase` 真正删除尾部元素 | 不能缩短，只能使用新的有效范围 |

因此，排序、查找、统计、填充等算法对两者的调用方式相同；真正需要分开理解的是“长度是否可以变化”。

### 1.2 区间 `[first, last)`

STL 使用左闭右开区间：包含 `first`，不包含 `last`。`end()` 指向最后一个元素之后，不能解引用。

~~~cpp
std::vector<int> values = {10, 40, 20, 30, 50};

// 处理下标 1、2、3；首尾元素不参与排序。
std::sort(values.begin() + 1, values.begin() + 4);
// values = {10, 20, 30, 40, 50}
~~~

`[it, it)` 是合法的空区间。`vector` 和 `array` 支持 `begin() + n`，但 `n` 必须让位置保持在 `[begin(), end()]` 内。

### 1.3 常用头文件和打印函数

~~~cpp
#include <algorithm>
#include <array>
#include <functional>
#include <iostream>
#include <iterator>
#include <numeric>
#include <string>
#include <vector>

template <class Container>
void print(const Container& values) {
    for (const auto& value : values) {
        std::cout << value << ' ';
    }
    std::cout << '\n';
}
~~~

---

## 二、调整顺序：`sort`、`reverse`、`rotate`

这三个算法都会改写指定范围。原顺序还要使用时，应先复制数据。

### 2.1 `std::sort`：排序

~~~cpp
std::vector<int> scores = {88, 60, 92, 75, 75};
std::sort(scores.begin(), scores.end());
print(scores);
// 60 75 75 88 92

std::array<int, 5> priority = {3, 1, 5, 2, 4};
std::sort(priority.begin(), priority.end(), std::greater<int>());
print(priority);
// 5 4 3 2 1
~~~

`sort` 需要随机访问迭代器，`vector` 和 `array` 都满足。比较器表达的是“左侧元素是否应排在右侧元素之前”，必须满足严格弱序；不要写成 `<=` 或 `>=`。

~~~cpp
std::vector<std::string> names = {"Tom", "Alice", "Bob"};

std::sort(names.begin(), names.end(),
          [](const std::string& left, const std::string& right) {
              if (left.size() != right.size()) {
                  return left.size() < right.size();
              }
              return left < right;
          });
// Bob Tom Alice
~~~

> **注意**：`sort` 不保证相等元素原来的相对顺序。若该顺序有意义，使用 `std::stable_sort`。

### 2.2 `std::reverse`：反转

`reverse` 不排序，只把已有顺序反过来。

~~~cpp
std::array<int, 5> route = {1, 2, 3, 4, 5};
std::reverse(route.begin(), route.end());
print(route);
// 5 4 3 2 1

std::vector<int> part = {10, 20, 30, 40, 50};
std::reverse(part.begin() + 1, part.end() - 1);
print(part);
// 10 40 30 20 50
~~~

它返回 `void`，至少需要双向迭代器。

### 2.3 `std::rotate`：改变起始位置

~~~cpp
std::rotate(first, middle, last);
~~~

调用后，`[first, middle)` 会移到末尾，`[middle, last)` 会接到前面；后半段非空时，`middle` 成为新范围的第一个元素。两段内部的相对顺序保持不变。

~~~cpp
std::array<int, 5> order = {1, 2, 3, 4, 5};

auto oldFirst = std::rotate(order.begin(), order.begin() + 2, order.end());
print(order);
// 3 4 5 1 2

std::cout << *oldFirst << '\n';
// 1
~~~

上例中返回迭代器指向原来的第一个元素。若 `first == middle`，没有发生轮转，返回值为 `last`；若 `middle == last`，返回值为 `first`。通常不需要依赖这个返回值，但在解引用前仍应确认它不是 `last`。`middle` 必须位于 `[first, last]` 内。

---

## 三、查找与统计：`find`、`count`、极值算法

### 3.1 `std::find` 与 `std::count`

`find` 返回第一个匹配元素的迭代器；找不到时返回传入的 `last`（本页完整范围示例中即 `end()`）。

~~~cpp
std::vector<int> ids = {101, 203, 305, 203};
auto pos = std::find(ids.begin(), ids.end(), 203);

if (pos != ids.end()) {
    std::cout << "下标：" << std::distance(ids.begin(), pos) << '\n';
    *pos = 204;  // 非 const 容器中，可以通过迭代器修改元素
}
~~~

`count` 会扫描完整个范围并返回数量。

~~~cpp
std::array<int, 8> dice = {6, 1, 6, 2, 6, 3, 4, 6};
auto sixCount = std::count(dice.begin(), dice.end(), 6);
std::cout << sixCount << '\n';
// 4
~~~

只需要判断“是否存在”时，`find` 通常更合适，因为它找到后就能停止。不要在 `find` 失败后解引用返回的 `last`。

### 3.2 `std::min_element`、`std::max_element`

二者返回极值的迭代器，而不是极值本身。

~~~cpp
std::vector<int> scores = {88, 60, 92, 75};

auto minIt = std::min_element(scores.begin(), scores.end());
auto maxIt = std::max_element(scores.begin(), scores.end());

if (minIt != scores.end()) {
    std::cout << "最低分：" << *minIt << '\n';
    std::cout << "最高分：" << *maxIt << '\n';
    std::cout << "最高分下标："
              << std::distance(scores.begin(), maxIt) << '\n';
}
~~~

空范围时，它们返回传入的 `last`；多个相同极值时，它们都返回第一个匹配位置。

### 3.3 `std::minmax_element`

需要同时找最小和最大值时，可直接调用一次：

~~~cpp
std::array<int, 6> temperatures = {26, 31, 28, 24, 33, 33};
auto result = std::minmax_element(temperatures.begin(), temperatures.end());

if (result.first != temperatures.end()) {
    std::cout << "最低温：" << *result.first << '\n';
    std::cout << "最高温：" << *result.second << '\n';
}
~~~

`result.first` 是最小元素，`result.second` 是最大元素。空范围时二者都等于传入的 `last`。并列时，`minmax_element` 返回第一个最小值和最后一个最大值；单独的 `max_element` 则返回第一个最大值。

---

## 四、有序范围查询：`binary_search`、`lower_bound`、`upper_bound`

三个算法的共同前提是：范围必须按照**同一个比较规则**保持有序。默认升序时，可以先 `sort`，再调用它们。

~~~cpp
std::vector<int> numbers = {40, 10, 30, 20, 20};
std::sort(numbers.begin(), numbers.end());
// 10 20 20 30 40

bool has20 = std::binary_search(numbers.begin(), numbers.end(), 20);
auto first20 = std::lower_bound(numbers.begin(), numbers.end(), 20);
auto after20 = std::upper_bound(numbers.begin(), numbers.end(), 20);

std::cout << std::boolalpha << has20 << '\n';                  // true
std::cout << std::distance(numbers.begin(), first20) << '\n'; // 1
std::cout << std::distance(numbers.begin(), after20) << '\n'; // 3
std::cout << std::distance(first20, after20) << '\n';          // 2
~~~

| 算法 | 默认升序时的含义 | 找不到时 |
| --- | --- | --- |
| `binary_search` | 是否存在目标值 | 返回 `false` |
| `lower_bound` | 第一个 `>= target` 的位置 | 返回插入位置，可能是 `end()` |
| `upper_bound` | 第一个 `> target` 的位置 | 返回插入位置，可能是 `end()` |

`lower_bound` 可直接提供 `vector` 的有序插入位置：

~~~cpp
std::vector<int> sorted = {10, 20, 20, 30};
auto insertPos = std::lower_bound(sorted.begin(), sorted.end(), 25);
sorted.insert(insertPos, 25);
// {10, 20, 20, 25, 30}
~~~

`array` 同样可以查询位置，但长度固定，不能插入新元素。

~~~cpp
std::array<int, 5> fixed = {10, 20, 30, 40, 50};
auto pos = std::lower_bound(fixed.begin(), fixed.end(), 35);
std::cout << std::distance(fixed.begin(), pos) << '\n';
// 3
~~~

如果排序使用了比较器，查询时也要使用同一个比较器：

~~~cpp
std::vector<int> values = {10, 40, 20, 30};
std::sort(values.begin(), values.end(), std::greater<int>());

bool has30 = std::binary_search(
    values.begin(), values.end(), 30, std::greater<int>());

auto firstAtMost30 = std::lower_bound(
    values.begin(), values.end(), 30, std::greater<int>());
~~~

降序时，`lower_bound` 的含义由比较器决定，不能再套用“第一个大于等于”的默认升序说法。对 `vector` 和 `array`，这组查询通常只需对数级比较。

---

## 五、复制、填充与处理元素：`copy`、`fill`、`transform`、`for_each`

### 5.1 `std::copy`

复制到普通目标迭代器时，目标范围必须已经足够大。

~~~cpp
std::vector<int> source = {10, 20, 30, 40, 50};
std::array<int, 5> backup = {};

std::copy(source.begin(), source.end(), backup.begin());
print(backup);
// 10 20 30 40 50
~~~

空的 `vector` 没有可写元素，使用 `back_inserter` 让 `copy` 自动追加：

~~~cpp
std::vector<int> copied;
std::copy(source.begin(), source.end(), std::back_inserter(copied));
print(copied);
// 10 20 30 40 50
~~~

> **注意**：`reserve()` 只增加容量，不增加元素个数。只调用 `reserve()` 后，不能把 `dst.begin()` 当作可写范围；应先 `resize()` 或使用 `back_inserter`。目标起点落在源区间内部时，应按移动方向使用 `copy_backward` 或临时容器。

### 5.2 `std::fill`

`fill` 为已有范围的每个元素赋同一个值，不会扩容。

~~~cpp
std::array<int, 7> attendance = {};
std::fill(attendance.begin(), attendance.end(), -1);
std::fill(attendance.begin() + 5, attendance.end(), 0);
print(attendance);
// -1 -1 -1 -1 -1 0 0

std::vector<std::string> seats(4);
std::fill(seats.begin(), seats.end(), "空位");
~~~

### 5.3 `std::transform`

一元版本将一个输入范围转换到目标范围。目标范围需要提前准备好。

~~~cpp
std::vector<int> scores = {58, 76, 95, 100, 40};
std::vector<int> adjusted(scores.size());

std::transform(scores.begin(), scores.end(), adjusted.begin(),
               [](int score) {
                   return std::min(score + 5, 100);
               });
print(adjusted);
// 63 81 100 100 45
~~~

不需要保留原值时，可以原地处理：

~~~cpp
std::transform(scores.begin(), scores.end(), scores.begin(),
               [](int score) {
                   return score * score;
               });
~~~

二元版本可合并两个等长序列：

~~~cpp
std::array<int, 3> midterm = {80, 90, 70};
std::array<int, 3> finalExam = {85, 88, 95};
std::array<int, 3> total = {};

std::transform(midterm.begin(), midterm.end(),
               finalExam.begin(), total.begin(),
               [](int left, int right) {
                   return left + right;
               });
// total = {165, 178, 165}
~~~

第二个输入范围至少要和第一个一样长，算法不会替你检查越界。

### 5.4 `std::for_each`

仅读取元素时可按值传参，也可使用 `const T&` 避免复制；需要修改原元素时使用 `T&`。

~~~cpp
std::array<int, 4> coupons = {10, 20, 30, 40};

std::for_each(coupons.begin(), coupons.end(), [](int& value) {
    value += 5;
});
print(coupons);
// 15 25 35 45
~~~

只遍历整个容器时，范围 `for` 往往更直接；`for_each` 适合明确处理一段迭代器区间，或需要传入现成函数、Lambda 的场景。

---

## 六、删除与去重：`remove`、`unique`

这两个算法不会调用容器的删除接口。它们会把要保留的元素移到前面，并返回新的逻辑结尾。

### 6.1 `std::remove`

对 `vector`，需要接着调用 `erase` 才会真的缩短容器：

~~~cpp
std::vector<int> values = {1, 0, 2, 0, 3, 0, 4};

auto newEnd = std::remove(values.begin(), values.end(), 0);
values.erase(newEnd, values.end());
print(values);
// 1 2 3 4
~~~

常见写法如下：

~~~cpp
values.erase(std::remove(values.begin(), values.end(), 0), values.end());
~~~

`array` 不能缩短，只能使用 `[begin(), newEnd)`：

~~~cpp
std::array<int, 6> fixed = {1, 0, 2, 0, 3, 0};
auto newEnd = std::remove(fixed.begin(), fixed.end(), 0);

for (auto it = fixed.begin(); it != newEnd; ++it) {
    std::cout << *it << ' ';
}
std::cout << '\n';
// 1 2 3
~~~

`newEnd` 之后的元素仍然可以解引用，但值处于未指定状态，不能当作删除后的结果。若 `array` 需要一个真正变短的结果，可以构造新的 `vector`：

~~~cpp
std::vector<int> kept(fixed.begin(), newEnd);
~~~

### 6.2 `std::unique`

`unique` 只处理相邻重复项：

~~~cpp
std::vector<int> values = {1, 1, 2, 2, 2, 3, 1, 1};

auto newEnd = std::unique(values.begin(), values.end());
values.erase(newEnd, values.end());
print(values);
// 1 2 3 1
~~~

对 `array`，同样只使用新的有效前缀：

~~~cpp
std::array<int, 6> fixed = {1, 1, 2, 2, 3, 3};
auto uniqueEnd = std::unique(fixed.begin(), fixed.end());

for (auto it = fixed.begin(); it != uniqueEnd; ++it) {
    std::cout << *it << ' ';
}
std::cout << '\n';
// 1 2 3
~~~

若业务要求完全去重，常用 `sort + unique`：

~~~cpp
std::vector<int> ids = {3, 1, 2, 3, 2, 1, 4};

std::sort(ids.begin(), ids.end());
ids.erase(std::unique(ids.begin(), ids.end()), ids.end());
print(ids);
// 1 2 3 4
~~~

这套写法会改变原有顺序。需要保留首次出现顺序时，不能直接这样处理。

和 `remove` 一样，`unique` 返回的 `newEnd` 之后也只是有效但值未指定的区域。对 `array`，只能把 `[begin(), newEnd)` 当作去重后的结果。

---

## 七、排列与汇总：`next_permutation`、`prev_permutation`、`accumulate`

### 7.1 `std::next_permutation`

~~~cpp
std::array<int, 3> order = {1, 2, 3};

bool hasNext = std::next_permutation(order.begin(), order.end());
print(order);
std::cout << std::boolalpha << hasNext << '\n';
// 1 3 2
// true
~~~

当当前排列已经是最大字典序时，它返回 `false`，并把范围改为最小字典序。枚举全部排列时，应从最小排列开始：

~~~cpp
std::vector<int> order = {1, 2, 3};
std::sort(order.begin(), order.end());

do {
    print(order);
} while (std::next_permutation(order.begin(), order.end()));
~~~

元素数量为 `n` 时，排列数最多是 `n!`，枚举前要先评估数据规模。

### 7.2 `std::prev_permutation`

~~~cpp
std::array<int, 3> order = {3, 2, 1};

bool hasPrev = std::prev_permutation(order.begin(), order.end());
print(order);
// 3 1 2
~~~

它与 `next_permutation` 相反：没有更小排列时返回 `false`，并把范围变为最大字典序。使用自定义比较器时，排序和排列函数也要使用同一个比较器。

### 7.3 `std::accumulate`

`accumulate` 位于 `<numeric>`，不是 `<algorithm>`。它按顺序把当前结果与下一个元素合并。

~~~cpp
std::vector<int> scores = {88, 60, 92, 75};

long long sum = std::accumulate(scores.begin(), scores.end(), 0LL);
double average = scores.empty()
    ? 0.0
    : static_cast<double>(sum) / scores.size();
~~~

初值决定累积过程的类型。即使最后用 `long long` 接收结果，初值写成 `0` 时仍可能先按 `int` 溢出；需要宽类型时使用 `0LL`。

~~~cpp
std::array<int, 4> factors = {2, 3, 4, 5};

int product = std::accumulate(factors.begin(), factors.end(), 1,
                              std::multiplies<int>());
std::cout << product << '\n';
// 120
~~~

---

## 八、完整示例：清理编号、去重、查询与统计

下面假设业务规则明确要求“编号不能重复”，并以 `-1` 表示无效记录。

~~~cpp
#include <algorithm>
#include <iostream>
#include <numeric>
#include <vector>

int main() {
    std::vector<int> ids = {105, -1, 103, 101, 103, 108, -1, 105};

    // 1. 删除无效编号。
    ids.erase(std::remove(ids.begin(), ids.end(), -1), ids.end());

    // 2. 排序并去重。这里会按升序重排编号。
    std::sort(ids.begin(), ids.end());
    ids.erase(std::unique(ids.begin(), ids.end()), ids.end());
    // ids = {101, 103, 105, 108}

    // 3. 在有序范围中查询。
    bool has105 = std::binary_search(ids.begin(), ids.end(), 105);
    auto firstAtLeast104 = std::lower_bound(ids.begin(), ids.end(), 104);

    // 4. 统计与极值。
    long long checksum = std::accumulate(ids.begin(), ids.end(), 0LL);
    auto range = std::minmax_element(ids.begin(), ids.end());

    std::cout << "编号：";
    for (int id : ids) {
        std::cout << id << ' ';
    }
    std::cout << '\n';
    std::cout << "是否有 105：" << std::boolalpha << has105 << '\n';
    std::cout << "校验和：" << checksum << '\n';

    if (firstAtLeast104 != ids.end()) {
        std::cout << "第一个不小于 104 的编号："
                  << *firstAtLeast104 << '\n';
    }
    if (range.first != ids.end()) {
        std::cout << "最小编号：" << *range.first
                  << "，最大编号：" << *range.second << '\n';
    }
}
~~~

这里的排序既为去重服务，也为二分查询提供前提。若业务需要保留录入顺序，应使用其他去重策略，而不是直接排序。

---

## 九、20 个算法速查表

| 算法 | 是否改写输入范围 | 返回值 | 关键前提或注意点 |
| --- | --- | --- | --- |
| `sort` | 是 | `void` | 需要随机访问迭代器；比较器应满足严格弱序 |
| `reverse` | 是 | `void` | 只反转，不排序 |
| `find` | 否 | 迭代器 | 找不到时等于传入的 `last` |
| `count` | 否 | 数量 | 必须扫描完整个范围 |
| `max_element` | 否 | 最大元素迭代器 | 空范围返回 `last` |
| `min_element` | 否 | 最小元素迭代器 | 空范围返回 `last` |
| `minmax_element` | 否 | 一对迭代器 | 空范围为 `{last, last}`；并列时取第一个最小、最后一个最大 |
| `copy` | 源范围否；目标范围会写入 | 目标尾后迭代器 | 目标要有足够空间，或用 `back_inserter` |
| `fill` | 是 | `void` | 只写已有范围，不扩容 |
| `transform` | 源范围通常不变；目标范围会写入 | 目标尾后迭代器 | 目标范围要可写；二元版本要保证第二个输入足够长 |
| `for_each` | 取决于函数参数 | 函数对象 | 用 `T&` 才能改写元素 |
| `remove` | 会整理元素 | 新逻辑结尾 | `vector` 需接 `erase`；`array` 只能使用有效前缀 |
| `unique` | 会整理元素 | 新逻辑结尾 | 只处理相邻重复项 |
| `binary_search` | 否 | `bool` | 范围须按同一比较规则有序 |
| `lower_bound` | 否 | 迭代器 | 默认升序时为第一个 `>= target` |
| `upper_bound` | 否 | 迭代器 | 默认升序时为第一个 `> target` |
| `next_permutation` | 是 | `bool` | 最大排列后变回最小排列 |
| `prev_permutation` | 是 | `bool` | 最小排列前变回最大排列 |
| `rotate` | 是 | 迭代器 | `middle` 是新范围的新起点 |
| `accumulate` | 否 | 累积结果 | 位于 `<numeric>`；初值决定累积类型 |

---

## 十、练习建议

1. 用 `std::array<int, 7>` 保存一周步数，分别求最大值、最小值和平均值。
2. 用 `std::vector<int>` 保存待处理编号，删除 `-1`，排序去重，再用 `lower_bound` 查询插入位置。
3. 对一组已排序的成绩，使用 `lower_bound` 和 `upper_bound` 统计某个分数出现的次数。
4. 用 `next_permutation` 输出 `{1, 2, 3}` 的全部排列。
5. 分别对 `vector` 与 `array` 使用 `remove`，观察“容器长度是否变化”和“有效范围”的差别。

实际写代码时，先确认三件事：数据是否有序、算法会不会改写元素、结果是迭代器还是数值。把这三点判断清楚后，这些算法就可以稳定地组合使用。
