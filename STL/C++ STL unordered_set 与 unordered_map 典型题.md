# C++ STL 专项教程：`unordered_set` / `unordered_map` 配合典型题目

------

## 第一章：基础回顾——它们是什么，为什么用它们

在正式刷题之前，我们先把基础打牢。

### 1.1 哈希表的核心思想

`unordered_set` 和 `unordered_map` 的底层都是**哈希表（Hash Table）**。

普通数组的查找需要从头遍历，时间复杂度是 O(n)。哈希表的思路是：把"键"通过一个**哈希函数**转换成一个数组下标，然后直接跳到那个位置存取数据。理想情况下，**查找、插入、删除的时间复杂度都是 O(1)**。

```
键 "apple"  →  哈希函数  →  下标 7  →  直接访问
键 "banana" →  哈希函数  →  下标 2  →  直接访问
```

这就是为什么哈希容器比 `set`/`map`（底层是红黑树，O(log n)）更快——代价是**元素无序**。

### 1.2 两个容器的职责区分

| 容器                  | 存什么       | 解决什么问题                 |
| --------------------- | ------------ | ---------------------------- |
| `unordered_set<T>`    | 只存**键**   | "这个元素**存不存在**？"     |
| `unordered_map<K, V>` | 存**键值对** | "这个键对应的**值是什么**？" |

记住这个区分，后面选容器时就不会迷茫。

### 1.3 头文件与声明

```cpp
#include <unordered_set>
#include <unordered_map>
#include <iostream>
#include <vector>
#include <string>
using namespace std;
```

------

## 第二章：`unordered_set` 详解

### 2.1 基本操作速查

```cpp
unordered_set<int> s;

// ① 插入
s.insert(10);
s.insert(20);
s.insert(10); // 重复插入，无效果！set 中只会有一个 10

// ② 查找（最核心的操作）
if (s.count(10)) {           // count 返回 0 或 1
    cout << "10 存在" << endl;
}
if (s.find(99) == s.end()) { // find 返回迭代器，找不到时返回 end()
    cout << "99 不存在" << endl;
}

// ③ 删除
s.erase(10);

// ④ 大小与判空
cout << s.size() << endl;   // 元素数量
cout << s.empty() << endl;  // 是否为空

// ⑤ 遍历（顺序不固定！）
for (int x : s) {
    cout << x << " ";
}
```

> ⚠️ **关键特性**：`unordered_set` 中的元素**自动去重**，同一个值永远只存一份。

### 2.2 用初始化列表构造

```cpp
unordered_set<string> words = {"apple", "banana", "apple", "cherry"};
// 结果中只有 3 个元素：apple、banana、cherry
cout << words.size(); // 输出 3
```

------

## 第三章：`unordered_map` 详解

### 3.1 基本操作速查

```cpp
unordered_map<string, int> mp;

// ① 插入 / 赋值（最常用写法）
mp["apple"] = 3;
mp["banana"] = 5;
mp["apple"] = 10; // 覆盖，apple 的值现在是 10

// ② 用 insert 插入（键已存在时不会覆盖）
mp.insert({"cherry", 7});
mp.insert({"apple", 99}); // 无效！apple 已存在，值不变

// ③ 查找
if (mp.count("apple")) {        // 检查键是否存在
    cout << mp["apple"] << endl; // 输出 10
}

auto it = mp.find("banana");
if (it != mp.end()) {
    cout << it->first << ": " << it->second << endl; // banana: 5
}

// ④ 删除
mp.erase("banana");

// ⑤ 遍历
for (auto& [key, val] : mp) {   // C++17 结构化绑定，强烈推荐
    cout << key << " -> " << val << endl;
}
```

### 3.2 `[]` 操作符的"隐式创建"陷阱

这是新手最容易踩的坑，必须单独说清楚：

```cpp
unordered_map<string, int> mp;

// 用 [] 访问一个不存在的键时，会自动创建它，初始值为 0！
cout << mp["xyz"] << endl; // 输出 0，同时 mp 里多了一个 "xyz" -> 0

cout << mp.size(); // 输出 1，不是 0！
```

**正确做法**：在只想查询时，用 `count` 或 `find` 先判断存在性：

```cpp
// 安全的读取方式
if (mp.count("xyz")) {
    cout << mp["xyz"];
}
// 或者用 .at()，键不存在时会抛出异常而不是静默创建
// cout << mp.at("xyz"); // 抛出 std::out_of_range
```

------

## 第四章：典型题目精讲——频次统计

频次统计是 `unordered_map` 最经典的用法，核心模式只有一行：

```cpp
mp[element]++; // 每遇到一个元素，它的计数加一
```

### 4.1 题目：统计字符频率

**题目描述**：给定一个字符串，统计每个字符出现的次数，并找出出现最多的字符。

```cpp
#include <unordered_map>
#include <iostream>
#include <string>
using namespace std;

int main() {
    string s = "programming";

    // 第一步：建立频率表
    unordered_map<char, int> freq;
    for (char c : s) {
        freq[c]++; // 核心：第一次访问自动创建并初始化为0，然后+1
    }

    // 第二步：打印所有字符的频率
    for (auto& [ch, cnt] : freq) {
        cout << ch << ": " << cnt << endl;
    }

    // 第三步：找出现次数最多的字符
    char maxChar = ' ';
    int maxCount = 0;
    for (auto& [ch, cnt] : freq) {
        if (cnt > maxCount) {
            maxCount = cnt;
            maxChar = ch;
        }
    }
    cout << "最多的字符: " << maxChar << "，出现 " << maxCount << " 次" << endl;

    return 0;
}
```

**输出结果**：

```
出现次数最多的字符: r，出现 2 次（或 g、m，因为同样出现2次）
```

**思路拆解**：

1. 遍历字符串，`freq[c]++` 利用了 `[]` 自动初始化为 0 的特性，第一次见到字符时从 0 开始计数。
2. 再遍历 map 找最大值，这是标准的"一次建表，多次查询"模式。

------

### 4.2 题目：找出出现次数超过一半的数（多数元素）

**题目描述**：给定一个大小为 n 的数组，找出其中出现次数**超过 n/2** 的元素（保证一定存在）。

```cpp
#include <unordered_map>
#include <vector>
#include <iostream>
using namespace std;

int majorityElement(vector<int>& nums) {
    unordered_map<int, int> freq;
    int n = nums.size();

    for (int x : nums) {
        freq[x]++;
        // 边统计边判断，找到后立刻返回
        if (freq[x] > n / 2) {
            return x;
        }
    }
    return -1; // 理论上不会到这里
}

int main() {
    vector<int> nums = {2, 2, 1, 1, 1, 2, 2};
    cout << majorityElement(nums) << endl; // 输出 2
    return 0;
}
```

**为什么可以边统计边返回**：题目保证答案存在，所以一旦某个数的计数超过 n/2，它一定是答案，可以提前结束，提升效率。

------

### 4.3 题目：前 K 个高频元素

**题目描述**：给定整数数组 nums 和整数 k，返回出现频率前 k 高的元素。

```cpp
#include <unordered_map>
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

vector<int> topKFrequent(vector<int>& nums, int k) {
    // 第一步：统计频率
    unordered_map<int, int> freq;
    for (int x : nums) {
        freq[x]++;
    }

    // 第二步：把 map 中的 (值, 频率) 对转入 vector，方便排序
    vector<pair<int, int>> items(freq.begin(), freq.end());

    // 第三步：按频率从大到小排序
    sort(items.begin(), items.end(), [](const pair<int,int>& a, const pair<int,int>& b) {
        return a.second > b.second; // 注意是按 second（频率）降序
    });

    // 第四步：取前 k 个
    vector<int> result;
    for (int i = 0; i < k; i++) {
        result.push_back(items[i].first);
    }
    return result;
}

int main() {
    vector<int> nums = {1, 1, 1, 2, 2, 3};
    int k = 2;
    auto res = topKFrequent(nums, k);
    for (int x : res) cout << x << " "; // 输出 1 2
    return 0;
}
```

**核心技巧**：`map` 不能直接排序，需要先转成 `vector<pair<int,int>>`，再用自定义比较函数排序。这是频率统计题的通用套路。

------

## 第五章：典型题目精讲——去重

去重是 `unordered_set` 最自然的用途，因为它天生不存重复元素。

### 5.1 题目：数组去重

**题目描述**：给定数组，返回去重后的所有元素（顺序不要求）。

```cpp
#include <unordered_set>
#include <vector>
#include <iostream>
using namespace std;

vector<int> removeDuplicates(vector<int>& nums) {
    unordered_set<int> seen(nums.begin(), nums.end()); // 直接用迭代器构造，一步完成去重
    return vector<int>(seen.begin(), seen.end());       // 转回 vector
}

int main() {
    vector<int> nums = {4, 3, 2, 7, 8, 2, 3, 1};
    auto res = removeDuplicates(nums);
    for (int x : res) cout << x << " ";
    return 0;
}
```

------

### 5.2 题目：找出数组中只出现一次的数

**题目描述**：给定数组，其中除一个元素外，其余元素都出现了两次，找出那个只出现一次的元素。

```cpp
#include <unordered_set>
#include <vector>
#include <iostream>
using namespace std;

int singleNumber(vector<int>& nums) {
    unordered_set<int> s;
    for (int x : nums) {
        if (s.count(x)) {
            s.erase(x);   // 第二次见到：从集合中删除（抵消）
        } else {
            s.insert(x);  // 第一次见到：加入集合
        }
    }
    return *s.begin(); // 最后集合里只剩那个单独的元素
}

int main() {
    vector<int> nums = {4, 1, 2, 1, 2};
    cout << singleNumber(nums) << endl; // 输出 4
    return 0;
}
```

**思路解读**：把集合当"抵消器"用——第一次见到加进去，第二次见到删掉，最后剩下的就是答案。这个思路非常巧妙。

> 注：此题用位运算异或（XOR）更优雅，但 set 的解法更直观易懂，适合初学阶段理解问题。

------

### 5.3 题目：最长连续序列（去重 + 集合查询的综合运用）

**题目描述**：给定一个未排序的整数数组，找出数字连续的最长序列（不要求序列元素在原数组中连续），要求时间复杂度 O(n)。

例如：`[100, 4, 200, 1, 3, 2]` → 最长连续序列是 `[1, 2, 3, 4]`，长度为 4。

```cpp
#include <unordered_set>
#include <vector>
#include <iostream>
using namespace std;

int longestConsecutive(vector<int>& nums) {
    unordered_set<int> numSet(nums.begin(), nums.end()); // 去重，同时支持 O(1) 查询

    int longest = 0;

    for (int x : numSet) {
        // 关键优化：只从"序列起点"开始数
        // 如果 x-1 存在，说明 x 不是起点，跳过
        if (numSet.count(x - 1)) continue;

        // x 是起点，向后延伸
        int currentNum = x;
        int currentLen = 1;
        while (numSet.count(currentNum + 1)) {
            currentNum++;
            currentLen++;
        }

        longest = max(longest, currentLen);
    }

    return longest;
}

int main() {
    vector<int> nums = {100, 4, 200, 1, 3, 2};
    cout << longestConsecutive(nums) << endl; // 输出 4
    return 0;
}
```

**为什么是 O(n) 而不是 O(n²)**：

- 如果每个元素都去延伸，看起来是 O(n²)。
- 但加了 `if (numSet.count(x - 1)) continue;` 之后，**每个元素只会作为起点被延伸一次**。
- 因为只有序列的第一个元素（左边没有邻居的）才会触发 while 循环。
- 所有元素加起来被"经过"的总次数是 O(n)。

这是 `unordered_set` 支持 O(1) 查询带来的关键收益。

------

## 第六章：典型题目精讲——两数之和

两数之和是 `unordered_map` 最经典的面试题，几乎是哈希表的代名词。

### 6.1 题目：两数之和（基础版）

**题目描述**：给定整数数组 nums 和目标值 target，找出数组中**两个数**之和等于 target 的下标。

**暴力解法（O(n²)，先理解再优化）**：

```cpp
// 暴力：枚举所有组合
for (int i = 0; i < n; i++)
    for (int j = i+1; j < n; j++)
        if (nums[i] + nums[j] == target)
            return {i, j};
```

**哈希表解法（O(n)）**：

```cpp
#include <unordered_map>
#include <vector>
#include <iostream>
using namespace std;

vector<int> twoSum(vector<int>& nums, int target) {
    // key: 数组中的值，value: 该值的下标
    unordered_map<int, int> seen;

    for (int i = 0; i < nums.size(); i++) {
        int complement = target - nums[i]; // 我需要的"另一半"

        if (seen.count(complement)) {
            // 找到了！complement 的下标和当前下标 i 就是答案
            return {seen[complement], i};
        }

        // 还没找到，把当前数记录下来，供后续元素查询
        seen[nums[i]] = i;
    }

    return {}; // 题目保证有解，不会到这里
}

int main() {
    vector<int> nums = {2, 7, 11, 15};
    int target = 9;
    auto res = twoSum(nums, target);
    cout << res[0] << ", " << res[1] << endl; // 输出 0, 1
    return 0;
}
```

**逐步模拟执行过程**（以 `nums = [2, 7, 11, 15], target = 9` 为例）：

```
i=0: nums[0]=2, complement=9-2=7, seen 里有 7 吗？没有。把 {2: 0} 存入 seen。
     seen = {2: 0}

i=1: nums[1]=7, complement=9-7=2, seen 里有 2 吗？有！下标是 0。
     返回 {0, 1}。✓
```

**为什么是 O(n)**：每个元素只被访问一次，每次 `count` 和 `[]` 操作都是 O(1)。

------

### 6.2 变体：三数之和为零

**题目描述**：找出数组中所有不重复的三元组 [a, b, c]，使得 a + b + c = 0。

```cpp
#include <vector>
#include <algorithm>
#include <unordered_set>
#include <iostream>
using namespace std;

vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end()); // 先排序，方便去重
    vector<vector<int>> result;
    int n = nums.size();

    for (int i = 0; i < n - 2; i++) {
        // 跳过重复的第一个数
        if (i > 0 && nums[i] == nums[i-1]) continue;

        // 固定 nums[i]，用哈希集合在 i 之后找两数之和为 -nums[i]
        unordered_set<int> seen;
        int target = -nums[i];

        for (int j = i + 1; j < n; j++) {
            int complement = target - nums[j];
            if (seen.count(complement)) {
                result.push_back({nums[i], complement, nums[j]});
                // 跳过重复的第三个数
                while (j + 1 < n && nums[j] == nums[j+1]) j++;
            }
            seen.insert(nums[j]);
        }
    }
    return result;
}

int main() {
    vector<int> nums = {-1, 0, 1, 2, -1, -4};
    auto res = threeSum(nums);
    for (auto& v : res) {
        cout << "[" << v[0] << ", " << v[1] << ", " << v[2] << "]" << endl;
    }
    // 输出: [-1, -1, 2] 和 [-1, 0, 1]
    return 0;
}
```

**思路**：固定一个数，把三数之和转化为两数之和，这是很常见的降维策略。

------

### 6.3 变体：两数之和——输入有序数组

如果数组已经排好序，还需要哈希表吗？这时候双指针更优，但对比能帮助理解哈希表的适用场景：

```cpp
// 有序数组：双指针 O(n) 且空间 O(1)
vector<int> twoSumSorted(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return {left + 1, right + 1}; // 返回1-indexed
        else if (sum < target) left++;
        else right--;
    }
    return {};
}
```

**结论**：

- 无序数组 → 用哈希表，O(n) 时间，O(n) 空间。
- 有序数组 → 用双指针，O(n) 时间，O(1) 空间。

------

## 第七章：典型题目精讲——分组

分组问题的核心是：**找到一个"规范形式（canonical form）"作为哈希表的键**，把能归为同一组的元素都映射到同一个键下。

### 7.1 题目：字母异位词分组

**题目描述**：给定字符串数组，将所有字母异位词（组成字母相同、顺序不同的词）分组。

例如：`["eat","tea","tan","ate","nat","bat"]` → `[["bat"],["nat","tan"],["ate","eat","tea"]]`

**关键思路**：`"eat"` 和 `"tea"` 和 `"ate"` 排序后都是 `"aet"`，用这个排序后的字符串作为 key！

```cpp
#include <unordered_map>
#include <vector>
#include <string>
#include <algorithm>
#include <iostream>
using namespace std;

vector<vector<string>> groupAnagrams(vector<string>& strs) {
    // key: 排序后的字符串（规范形式），value: 所有属于这组的原字符串
    unordered_map<string, vector<string>> groups;

    for (const string& s : strs) {
        string key = s;
        sort(key.begin(), key.end()); // "eat" → "aet"，"tea" → "aet"
        groups[key].push_back(s);    // 归入同一组
    }

    // 把 map 中的所有组提取出来
    vector<vector<string>> result;
    for (auto& [key, group] : groups) {
        result.push_back(group);
    }
    return result;
}

int main() {
    vector<string> strs = {"eat","tea","tan","ate","nat","bat"};
    auto res = groupAnagrams(strs);
    for (auto& group : res) {
        cout << "[ ";
        for (auto& s : group) cout << s << " ";
        cout << "]" << endl;
    }
    return 0;
}
```

**执行过程模拟**：

```
"eat" → 排序 → "aet" → groups["aet"] = ["eat"]
"tea" → 排序 → "aet" → groups["aet"] = ["eat", "tea"]
"tan" → 排序 → "ant" → groups["ant"] = ["tan"]
"ate" → 排序 → "aet" → groups["aet"] = ["eat", "tea", "ate"]
"nat" → 排序 → "ant" → groups["ant"] = ["tan", "nat"]
"bat" → 排序 → "abt" → groups["abt"] = ["bat"]
```

------

### 7.2 题目：按首字母分组

**题目描述**：给定字符串列表，按首字母将它们分组。

```cpp
#include <unordered_map>
#include <vector>
#include <string>
#include <iostream>
using namespace std;

unordered_map<char, vector<string>> groupByFirstLetter(vector<string>& words) {
    unordered_map<char, vector<string>> groups;
    for (const string& w : words) {
        if (!w.empty()) {
            groups[w[0]].push_back(w); // 首字母作为 key
        }
    }
    return groups;
}

int main() {
    vector<string> words = {"apple", "ant", "banana", "avocado", "blueberry", "cherry"};
    auto groups = groupByFirstLetter(words);
    for (auto& [ch, group] : groups) {
        cout << ch << ": ";
        for (auto& w : group) cout << w << " ";
        cout << endl;
    }
    return 0;
}
```

------

### 7.3 题目：同构字符串判断（分组思维的延伸）

**题目描述**：判断两个字符串是否同构。例如 `"egg"` 和 `"add"` 同构（e→a, g→d），`"foo"` 和 `"bar"` 不同构（o 要映射到两个不同字母）。

```cpp
#include <unordered_map>
#include <string>
#include <iostream>
using namespace std;

bool isIsomorphic(string s, string t) {
    if (s.size() != t.size()) return false;

    unordered_map<char, char> sToT, tToS; // 双向映射，防止两个字符映射到同一个

    for (int i = 0; i < s.size(); i++) {
        char sc = s[i], tc = t[i];

        // 检查 s→t 方向
        if (sToT.count(sc)) {
            if (sToT[sc] != tc) return false; // 冲突！
        } else {
            sToT[sc] = tc;
        }

        // 检查 t→s 方向（防止 "ab" 和 "aa" 这种情况）
        if (tToS.count(tc)) {
            if (tToS[tc] != sc) return false;
        } else {
            tToS[tc] = sc;
        }
    }
    return true;
}

int main() {
    cout << isIsomorphic("egg", "add") << endl;  // 1 (true)
    cout << isIsomorphic("foo", "bar") << endl;  // 0 (false)
    cout << isIsomorphic("paper", "title") << endl; // 1 (true)
    return 0;
}
```

**为什么需要双向映射**：

- 单向 s→t：`"ab"` 和 `"aa"` 会错误地判断为同构（a→a, b→a，但 t 中 a 要对应 s 中两个字符）。
- 加上 t→s 方向，就能捕获这种"多对一"的非法情况。

------

## 第八章：综合进阶——嵌套与组合使用

### 8.1 `unordered_map<string, unordered_set<string>>`：值也是集合

**场景**：构建课程和选课学生的映射，每个课程对应一个学生集合（自动去重）。

```cpp
#include <unordered_map>
#include <unordered_set>
#include <string>
#include <vector>
#include <iostream>
using namespace std;

int main() {
    // 选课记录：{学生, 课程}
    vector<pair<string, string>> enrollments = {
        {"Alice", "Math"}, {"Bob", "Math"}, {"Alice", "CS"},
        {"Charlie", "Math"}, {"Bob", "CS"}, {"Alice", "Math"} // Alice重复选Math
    };

    unordered_map<string, unordered_set<string>> courseStudents;

    for (auto& [student, course] : enrollments) {
        courseStudents[course].insert(student); // set 自动去重
    }

    for (auto& [course, students] : courseStudents) {
        cout << course << ": ";
        for (auto& s : students) cout << s << " ";
        cout << "(" << students.size() << " 人)" << endl;
    }
    // Math: Alice Bob Charlie (3 人)
    // CS: Alice Bob (2 人)
    return 0;
}
```

------

### 8.2 题目综合练习——词语模式匹配

**题目描述**：给定模式串 pattern 和字符串 str，判断 str 是否符合 pattern。

例如：`pattern = "abba"`, `str = "dog cat cat dog"` → true

```cpp
#include <unordered_map>
#include <unordered_set>
#include <string>
#include <sstream>
#include <vector>
#include <iostream>
using namespace std;

bool wordPattern(string pattern, string s) {
    // 先把 s 按空格分割成单词数组
    vector<string> words;
    istringstream iss(s);
    string word;
    while (iss >> word) words.push_back(word);

    if (pattern.size() != words.size()) return false;

    unordered_map<char, string> charToWord;   // 字符 → 单词
    unordered_map<string, char> wordToChar;   // 单词 → 字符（防多对一）

    for (int i = 0; i < pattern.size(); i++) {
        char c = pattern[i];
        string& w = words[i];

        if (charToWord.count(c) && charToWord[c] != w) return false;
        if (wordToChar.count(w) && wordToChar[w] != c) return false;

        charToWord[c] = w;
        wordToChar[w] = c;
    }
    return true;
}

int main() {
    cout << wordPattern("abba", "dog cat cat dog") << endl; // 1
    cout << wordPattern("abba", "dog cat cat fish") << endl; // 0
    cout << wordPattern("aaaa", "dog cat cat dog") << endl; // 0
    return 0;
}
```

------

## 第九章：常见错误与注意事项汇总

### 错误一：遍历时修改容器

```cpp
// ❌ 错误：遍历时删除元素，行为未定义
for (auto it = mp.begin(); it != mp.end(); it++) {
    if (it->second == 0) mp.erase(it); // 危险！
}

// ✅ 正确：erase 返回下一个有效迭代器
for (auto it = mp.begin(); it != mp.end(); ) {
    if (it->second == 0) it = mp.erase(it);
    else it++;
}
```

### 错误二：自定义类型作为键忘记提供哈希函数

```cpp
// ❌ 编译报错：pair<int,int> 没有默认哈希函数
unordered_set<pair<int,int>> s;

// ✅ 正确：提供自定义哈希
struct PairHash {
    size_t operator()(const pair<int,int>& p) const {
        return hash<int>()(p.first) ^ (hash<int>()(p.second) << 32);
    }
};
unordered_set<pair<int,int>, PairHash> s;
```

### 错误三：误用 `[]` 进行存在性检查

```cpp
// ❌ 错误：会意外创建键
if (mp["unknown_key"]) { ... } // 创建了 "unknown_key": 0

// ✅ 正确
if (mp.count("unknown_key")) { ... }
```

------

## 第十章：复杂度与选型速查表

| 操作     | `unordered_set/map` | `set/map` |
| -------- | ------------------- | --------- |
| 插入     | O(1) 平均           | O(log n)  |
| 查找     | O(1) 平均           | O(log n)  |
| 删除     | O(1) 平均           | O(log n)  |
| 遍历顺序 | **无序**            | **有序**  |

**选型建议**：

- 只需要"存在/不存在" → `unordered_set`
- 需要"键对应的值" → `unordered_map`
- 需要**有序遍历**或**范围查询** → 改用 `set`/`map`
- 键是自定义类型 → 需要手写哈希函数

------

## 总结：四类题的解题模板

```cpp
// ① 频次统计
unordered_map<T, int> freq;
for (auto& x : data) freq[x]++;

// ② 去重
unordered_set<T> seen(data.begin(), data.end());

// ③ 两数之和（查"我需要的另一半"）
unordered_map<int, int> seen; // 值 → 下标
for (int i = 0; i < n; i++) {
    int need = target - nums[i];
    if (seen.count(need)) return {seen[need], i};
    seen[nums[i]] = i;
}

// ④ 分组（设计规范形式作为key）
unordered_map<KeyType, vector<T>> groups;
for (auto& x : data) {
    KeyType key = canonicalForm(x); // 排序、首字母等
    groups[key].push_back(x);
}
```

掌握这四个模板，配合上面的例题反复练习，`unordered_set` 和 `unordered_map` 就真正内化了。祝学习顺利！