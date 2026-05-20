# 25级大数据1班 C++ 练习题汇总

> 使用方法：先读题面和样例，再看每题后的“答案解析”。解析按“建模思路 / 核心算法 / 输出易错点 / 参考实现提示”组织，适合作为写代码前的解题提纲。

## 文档导航

| 章节 | 主题 | 题目数 |
| --- | --- | --- |
| [实验11](#实验11-继承基础) | 继承基础 | 5 |
| [实验10](#实验10-期中综合练习) | 期中综合练习 | 5 |
| [实验9](#实验9-期中模拟) | 期中模拟 | 4 |
| [实验8](#实验8-静态成员与友元) | 静态成员与友元 | 5 |
| [实验7](#实验7-构造与拷贝构造) | 构造与拷贝构造 | 5 |
| [实验6](#实验6-构造与析构) | 构造与析构 | 5 |
| [实验5](#实验5-类与对象) | 类与对象 | 5 |
| [实验4](#实验4-引用与结构) | 引用与结构 | 5 |
| [实验三](#实验三-指针2) | 指针2 | 5 |

---

## 实验11. 继承基础

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1268` |
| 开放时间 | 2026-05-12T10:03:00+08:00 至 2026-05-19T00:00:00+08:00 |
| 本节题目数 | 5 |

---

### 1. 圆和圆柱体计算（继承）（20 分）

- 题目 ID: `68`
- 限制: 1s / 128MB

#### 题目描述

定义一个CPoint点类，包含数据成员x,y（坐标点）。

以CPoint为基类，派生出一个圆形类CCircle，增加数据成员r(半径）和一个计算圆面积的成员函数。

再以CCircle做为直接基类，派生出一个圆柱体类CCylinder，增加数据成员h(高)和一个计算体积的成员函数。

生成圆和圆柱体对象，调用成员函数计算面积或体积并输出结果。

#### 输入格式

输入圆的圆心位置、半径

输入圆柱体圆心位置、半径、高

#### 输出格式

输出圆的圆心位置 半径

输出圆面积

输出圆柱体的圆心位置 半径 高

输出圆柱体体积

#### 样例输入
```text
0 0 1
1 1 2 3
```

#### 样例输出
```text
Circle:(0,0),1
Area:3.14
Cylinder:(1,1),2,3
Volume:37.68
```

#### 答案解析

- **建模思路**：建立 `CPoint -> CCircle -> CCylinder` 三层继承。`CPoint` 保存圆心坐标，`CCircle` 增加半径并计算面积，`CCylinder` 增加高并计算体积。
- **核心算法**：圆面积为 `3.14 * r * r`，圆柱体体积为 `底面积 * h`。因为样例输出 `3.14` 和 `37.68`，这里直接使用 `pi = 3.14`。
- **输出易错点**：格式必须是 `Circle:(x,y),r` 和 `Cylinder:(x,y),r,h`，冒号、括号、逗号都不能多空格。

#### 参考实现提示

先用构造函数初始化基类部分，再在派生类中调用基类成员函数取得面积；若数据成员设为 `protected`，派生类访问会更直接。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class CPoint {
  protected:
    double x, y;

  public:
    CPoint(double a = 0, double b = 0) : x(a), y(b) {}
};
class CCircle : public CPoint {
  protected:
    double r;

  public:
    CCircle(double x = 0, double y = 0, double r = 0) : CPoint(x, y), r(r) {}
    double area() const {
        return 3.14 * r * r;
    }
    void print() const {
        cout << "Circle:(" << x << "," << y << ")," << r << endl;
    }
};
class CCylinder : public CCircle {
    double h;

  public:
    CCylinder(double x = 0, double y = 0, double r = 0, double h = 0) : CCircle(x, y, r), h(h) {}
    double volume() const {
        return area() * h;
    }
    void print() const {
        cout << "Cylinder:(" << x << "," << y << ")," << r << "," << h << endl;
    }
};
int main() {
    double x, y, r, x2, y2, r2, h;
    cin >> x >> y >> r >> x2 >> y2 >> r2 >> h;
    CCircle c(x, y, r);
    CCylinder cy(x2, y2, r2, h);
    c.print();
    cout << "Area:" << c.area() << endl;
    cy.print();
    cout << "Volume:" << cy.volume() << endl;
    return 0;
}
```

---

### 2. 三维空间的点（继承）（20 分）

- 题目 ID: `69`
- 限制: 1s / 128MB

#### 题目描述

定义一个平面上的点C2D类，它含有一个getDistance()的成员函数，计算该点到原点的距离；从C2D类派生出三维空间的点C3D类，它的getDistance()成员函数计算该点到原点的距离。试分别生成一个C2D和C3D的对象，计算它们到原点的距离。

三维空间的两点（x, y, z）和（x1, y1, z1）的距离公式如下：

[(x-x1)^2+(y-y1)^2+(z-z1)^2]^(1/2)

#### 输入格式

第一行二维坐标点位置

第二行三维坐标点位置1

第三行三维坐标点位置2

#### 输出格式

第一行二维坐标点位置到原点的距离

第二行三维坐标点位置1到原点的距离

第三行三维坐标点位置2到原点的距离

第四行三维坐标点位置2赋值给二维坐标点变量后，二维坐标点到原点的距离

#### 样例输入
```text
3 4
3 4 5
6 8 8
```

#### 样例输出
```text
5
7.07107
12.8062
10
```

#### 答案解析

- **建模思路**：`C2D` 保存 `x,y`，成员函数 `getDistance()` 返回二维点到原点距离；`C3D` 继承 `C2D`，增加 `z`，重新定义 `getDistance()` 返回三维距离。
- **核心算法**：二维距离为 `sqrt(x*x + y*y)`，三维距离为 `sqrt(x*x + y*y + z*z)`。最后一行把三维点赋值给二维点变量，会发生对象切片，只保留 `x,y`，所以样例中 `(6,8,8)` 变成二维点后距离为 `10`。
- **输出易错点**：不要强制固定小数位，默认 `cout` 输出即可匹配样例的 `7.07107`、`12.8062`。

#### 参考实现提示

在 `C3D` 中可复用基类保存的 `x,y`，但赋值给 `C2D` 时 `z` 会被舍弃，这是本题考察点。

#### 完整可运行参考代码

```cpp
#include <cmath>
#include <iostream>
using namespace std;
class C2D {
  protected:
    double x, y;

  public:
    C2D(double x = 0, double y = 0) : x(x), y(y) {}
    double getDistance() const {
        return sqrt(x * x + y * y);
    }
};
class C3D : public C2D {
    double z;

  public:
    C3D(double x = 0, double y = 0, double z = 0) : C2D(x, y), z(z) {}
    double getDistance() const {
        return sqrt(x * x + y * y + z * z);
    }
};
int main() {
    double x, y, z;
    cin >> x >> y;
    C2D a(x, y);
    cin >> x >> y >> z;
    C3D b(x, y, z);
    cin >> x >> y >> z;
    C3D c(x, y, z);
    C2D d = c;
    cout << a.getDistance() << endl
         << b.getDistance() << endl
         << c.getDistance() << endl
         << d.getDistance() << endl;
    return 0;
}
```

---

### 3. 学生成绩计算（继承）（20 分）

- 题目 ID: `175`
- 限制: 1s / 128MB

#### 题目描述

定义Person类具有姓名、年龄等属性，具有输出基本信息的display函数。

选修《面向对象程序设计》课程的学生在Person类的基础上，派生出子类：免听生和非免听生。子类继承父类成员，新增其他成员、改写display函数。

非免听生具有平时成绩、考试成绩和总评成绩三个属性，总评成绩根据（平时成绩*40%+考试成绩*60%）计算的结果，85分（包含）以上为A，75分（包含）-85分（不包含）为B，65分（包含）-75分（不包含）为C，60分（包含）-65分（不包含）为D，60分（不包含）以下为F。

免听生只有考试成绩和总评成绩两个属性，总评成绩100%根据考试成绩对应上述等级制成绩。

定义上述类并编写主函数，输入类型符号，若输入R，根据学生基本信息、平时成绩和考试成绩，建立非免听生对象，若输入S，根据学生基本信息、考试成绩，建立免听生对象。计算学生的总评成绩，并输出。

#### 输入格式

测试次数t

随后每行输入学生类型 相关信息，姓名的最大字符长度为20

#### 输出格式

每个学生基本信息和总评成绩

#### 样例输入
```text
2
R cindy 18 100 100
S sandy 28 59
```

#### 样例输出
```text
cindy 18 A
sandy 28 F
```

#### 答案解析

- **建模思路**：`Person` 保存姓名和年龄，并提供基础输出；非免听生类保存平时成绩、考试成绩；免听生类只保存考试成绩。
- **核心算法**：非免听生总评分数为 `平时*0.4 + 考试*0.6`，免听生总评分数就是考试成绩。再按区间转成等级：`>=85 A`，`>=75 B`，`>=65 C`，`>=60 D`，否则 `F`。
- **输出易错点**：每个学生输出 `姓名 年龄 等级`。等级判断要从高到低写，避免 `85` 被提前归入低等级。

#### 参考实现提示

读入类型字符后分支创建对象：`R` 读平时和考试两个成绩，`S` 只读考试成绩。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;

class Person {
  protected:
    string name;
    int age;

  public:
    Person(string n = "", int a = 0) : name(n), age(a) {}
};

char getGrade(double score) {
    if (score >= 85)
        return 'A';
    if (score >= 75)
        return 'B';
    if (score >= 65)
        return 'C';
    if (score >= 60)
        return 'D';
    return 'F';
}

class RegularStudent : public Person {
    double usualScore, examScore;

  public:
    RegularStudent(string n, int a, double usual, double exam)
        : Person(n, a), usualScore(usual), examScore(exam) {}

    void display() const {
        double total = usualScore * 0.4 + examScore * 0.6;
        cout << name << ' ' << age << ' ' << getGrade(total) << endl;
    }
};

class SpecialStudent : public Person {
    double examScore;

  public:
    SpecialStudent(string n, int a, double exam) : Person(n, a), examScore(exam) {}

    void display() const {
        cout << name << ' ' << age << ' ' << getGrade(examScore) << endl;
    }
};

int main() {
    int t;
    cin >> t;
    while (t--) {
        char type;
        string name;
        int age;
        cin >> type >> name >> age;

        if (type == 'R') {
            double usual, exam;
            cin >> usual >> exam;
            RegularStudent student(name, age, usual, exam);
            student.display();
        } else {
            double exam;
            cin >> exam;
            SpecialStudent student(name, age, exam);
            student.display();
        }
    }
    return 0;
}
```

---

### 4. 时钟模拟(继承）（20 分）

- 题目 ID: `73`
- 限制: 1s / 128MB

#### 题目描述

定义计数器类，包含保护数据成员value,公有函数increment计数加1。

定义循环计数器继承计数器类，增加私有数据成员：最小值minValue，maxValue,

重写公有函数increment，使得value在minValue~maxValue区间内循环+1。

定义时钟类，数据成员是私有循环计数器对象小时hour、分钟minute、秒second，公有函数time(int s)计算当前时间经过s秒之后的时间，即hour，minute，second的新value值。

定义时钟类对象，输入当前时间和经过的秒数,调用time函数计算新时间。

根据题目要求，增加必要的构造函数、析构函数和其他所需函数。

因为clock和time是系统内置函数，为了避免重名，请不要使用clock或者time作为类名或者函数名

#### 输入格式

第一行测试次数n

2行一组，第一行为当前时间（小时 分钟 秒），第二行为经过的秒数。

#### 输出格式

输出n行

每行对应每组当前时间和经过秒数后计算得到的新时间（小时：分钟：秒）。

#### 样例输入
```text
2
8 19 20
20
23 30 0
1801
```

#### 样例输出
```text
8:19:40
0:0:1
```

#### 答案解析

- **建模思路**：`Counter` 负责普通加一，`CycleCounter` 在最小值到最大值之间循环加一。时钟类内部组合三个循环计数器：小时、分钟、秒。
- **核心算法**：最直接的写法是把当前时间转成总秒数 `h*3600 + m*60 + s`，加上经过秒数后对 `24*3600` 取模，再还原为时、分、秒。
- **输出易错点**：样例不要求补零，输出格式是 `小时:分钟:秒`，例如 `0:0:1`。

#### 参考实现提示

虽然题目要求设计计数器继承关系，但计算函数内部使用“总秒数法”更稳定；类设计符合题意即可。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class Counter {
  protected:
    int value;

  public:
    Counter(int v = 0) : value(v) {}
    virtual void increment() {
        value++;
    }
    int getValue() const {
        return value;
    }
};
class CycleCounter : public Counter {
    int minValue, maxValue;

  public:
    CycleCounter(int mn = 0, int mx = 0, int v = 0) : Counter(v), minValue(mn), maxValue(mx) {}
    void setValue(int v) {
        value = v;
    }
    void increment() {
        value++;
        if (value > maxValue)
            value = minValue;
    }
};
class MyClock {
    CycleCounter h, m, s;

  public:
    MyClock(int hh, int mm, int ss) : h(0, 23, hh), m(0, 59, mm), s(0, 59, ss) {}
    void add(int sec) {
        int total = h.getValue() * 3600 + m.getValue() * 60 + s.getValue() + sec;
        total %= 86400;
        h.setValue(total / 3600);
        m.setValue(total % 3600 / 60);
        s.setValue(total % 60);
    }
    void print() const {
        cout << h.getValue() << ":" << m.getValue() << ":" << s.getValue() << endl;
    }
};
int main() {
    int n;
    cin >> n;
    while (n--) {
        int h, m, s, sec;
        cin >> h >> m >> s >> sec;
        MyClock c(h, m, s);
        c.add(sec);
        c.print();
    }
    return 0;
}
```

---

### 5. 新旧身份证(继承)（20 分）

- 题目 ID: `74`
- 限制: 3s / 128MB

#### 题目描述

按下述方式定义一个日期类CDate和描述15位身份证号的旧身份证类COldId:

class CDate
{

private:
	int year, month, day;
public:
	CDate(int, int, int);
	bool check(); //检验日期是否合法
	bool isLeap();
	void print();
};

class COldId
{
protected:
	char* pId15, * pName; //15位身份证号码，姓名
	CDate birthday; //出生日期
public:
	COldId(char* pIdVal, char* pNameVal, CDate& day);
	bool check(); //验证15位身份证是否合法
	void print();
	~COldId();
};

然后以COldId为基类派生18位身份证号的新身份证类CNewId,并增加3个数据成员:pId18(18位号码)、issueDay(签发日期)和validYear(有效期，年数)，并重新定义check()和print()。

15位身份证号扩展为18位身份证号的规则为：前6位号码保持一致，年份由2位变为4位（在前面加上19，例如88变为1988），剩余号码都保持一致，再加上第18位校验码。

身份证第18位校验码的生成方法：

1、将身份证号码前17位数分别乘以7,9,10,5,8,4,2,1,6,3,7,9,10,5,8,4,2。然后将其相加。

2、将17位数字与系数乘加的和除以11，得到余数。

3、余数与校验码的对应关系为1,0,X,9,8,7,6,5,4,3,2。也即：如果余数是3，身份证第18位就是9。如果余数是2，身份证的最后一位号码就是X。

主函数定义一个派生类对象，并用派生类对象调用check()，若返回false则输出“illegal id”否则调用print()输出身份证信息。check()对身份证合法性进行验证的规则：

1. 确认18位号码是从15位号码扩展的，且第18位校验码正确.

2. 身份证中的出生日期合法.

3. 身份证号码中不含非法字符.

4. 身份证号码的长度正确.

5. 身份证目前处于有效期内，假设当前日期为2021年11月9日。

6. 签发日期合法

#### 输入格式

测试数据的组数 t

第一个人姓名、出生日期年月日、15位身份证号码、18位身份证号码、签发日期年月日、有效期(100年按长期处理)

第二个人姓名、出生日期年月日、15位身份证号码、18位身份证号码、签发日期年月日、有效期(100年按长期处理)

......

姓名的最大字符长度为20

#### 输出格式

第一个人姓名

第一个人18位身份证号信息(号码、签发日期和有效期）或"illegal id"

第二个人姓名

第二个人18位身份证号信息(号码、签发日期和有效期）或"illegal id"

......

#### 样例输入
```text
10
AAAA 1988 2 28 440301880228113 440301198802281133 2006 1 20 20
BBBB 1997 4 30 440301980808554 440301199808085541 2015 2 2 10
CCCC 1920 5 8 530102200508011 53010219200508011X 1980 3 4 30
DDDD 1980 1 1 340524800101001 340524198001010012 1998 12 11 20
EEEE 1988 11 12 110203881112034 110203198811120340 2007 2 29 20 
FFFF 1964 11 15 432831641115081 432831196411150810 2015 8 7 100
GGGG 1996 12 10 44030196121010 44030119961210109 2014 6 7 20
HHHH 1988 7 21 440301880721X12 44030119880721X122 2006 5 11 20
IIII 1976 3 30 440301760330098 440301197603300983 2003 4 15 20
JJJJ 1955 9 5 440301550905205 440301195509051052 2004 6 4 100
```

#### 样例输出
```text
AAAA
440301198802281133 2006年1月20日 20年
BBBB
illegal id
CCCC
illegal id
DDDD
illegal id
EEEE
illegal id
FFFF
432831196411150810 2015年8月7日 长期
GGGG
illegal id
HHHH
illegal id
IIII
illegal id
JJJJ
illegal id
```

#### 答案解析

- **建模思路**：`CDate` 负责日期合法性；`COldId` 保存 15 位号码、姓名和生日；`CNewId` 继承旧身份证，并增加 18 位号码、签发日期和有效期。
- **核心算法**：先由 15 位号生成应有的前 17 位：前 6 位不变，中间生日年份补 `19`，后 9 位保留；再按权重数组计算校验码，和输入的第 18 位比较。
- **合法性检查**：15 位前提是长度正确且全数字；18 位前 17 位必须全数字，最后一位允许数字或 `X`；身份证生日、输入生日、签发日期都要合法；按 2021-11-09 判断是否仍在有效期内，`100` 年表示长期。
- **输出易错点**：每组先输出姓名；合法时输出 `18位号码 签发日期 有效期`，有效期 `100` 输出 `长期`，否则输出 `20年` 这类格式。

#### 参考实现提示

建议把“日期合法”“计算校验码”“由 15 位扩展 17 位”“判断有效期”拆成小函数，主 `check()` 只组合这些判断。

#### 完整可运行参考代码

```cpp
#include <cctype>
#include <iostream>
#include <string>
using namespace std;

class CDate {
    int year, month, day;

  public:
    CDate(int y = 0, int m = 0, int d = 0) : year(y), month(m), day(d) {}

    bool isLeap() const {
        return year % 400 == 0 || (year % 4 == 0 && year % 100 != 0);
    }

    bool check() const {
        int days[13] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
        if (year <= 0 || month < 1 || month > 12)
            return false;
        if (isLeap())
            days[2] = 29;
        return day >= 1 && day <= days[month];
    }

    int y() const {
        return year;
    }
    int m() const {
        return month;
    }
    int d() const {
        return day;
    }
    int number() const {
        return year * 10000 + month * 100 + day;
    }

    void print() const {
        cout << year << "年" << month << "月" << day << "日";
    }
};

bool allDigit(const string &s) {
    for (char c : s) {
        if (!isdigit((unsigned char)c))
            return false;
    }
    return true;
}

class COldId {
  protected:
    string id15, name;
    CDate birthday;

  public:
    COldId(string id, string n, CDate b) : id15(id), name(n), birthday(b) {}

    virtual bool check() const {
        if (id15.size() != 15 || !allDigit(id15) || !birthday.check())
            return false;
        int yy = stoi(id15.substr(6, 2));
        int mm = stoi(id15.substr(8, 2));
        int dd = stoi(id15.substr(10, 2));
        return birthday.y() == 1900 + yy && birthday.m() == mm && birthday.d() == dd;
    }
};

class CNewId : public COldId {
    string id18;
    CDate issueDay;
    int validYear;

    char calcCheckCode(const string &first17) const {
        int weight[17] = {7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2};
        char code[11] = {'1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2'};
        int sum = 0;
        for (int i = 0; i < 17; i++)
            sum += (first17[i] - '0') * weight[i];
        return code[sum % 11];
    }

    bool isInValidPeriod() const {
        if (validYear == 100)
            return true;
        CDate endDay(issueDay.y() + validYear, issueDay.m(), issueDay.d());
        return endDay.number() >= 20211109;
    }

  public:
    CNewId(string n, CDate birth, string oldId, string newId, CDate issue, int valid)
        : COldId(oldId, n, birth), id18(newId), issueDay(issue), validYear(valid) {}

    bool check() const override {
        if (!COldId::check())
            return false;
        if (id18.size() != 18 || !allDigit(id18.substr(0, 17)))
            return false;
        if (!(isdigit((unsigned char)id18[17]) || id18[17] == 'X'))
            return false;
        if (!issueDay.check() || !isInValidPeriod())
            return false;

        string expected17 = id15.substr(0, 6) + "19" + id15.substr(6);
        return id18.substr(0, 17) == expected17 && id18[17] == calcCheckCode(expected17);
    }

    void print() const {
        cout << name << endl;
        cout << id18 << ' ';
        issueDay.print();
        cout << ' ';
        if (validYear == 100)
            cout << "长期" << endl;
        else
            cout << validYear << "年" << endl;
    }
};

int main() {
    int t;
    cin >> t;
    while (t--) {
        string name, oldId, newId;
        int y, m, d, iy, im, id, valid;
        cin >> name >> y >> m >> d >> oldId >> newId >> iy >> im >> id >> valid;
        CNewId person(name, CDate(y, m, d), oldId, newId, CDate(iy, im, id), valid);
        if (person.check())
            person.print();
        else
            cout << name << endl << "illegal id" << endl;
    }
    return 0;
}
```

---

## 实验10. 期中综合练习

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1261` |
| 开放时间 | 2026-05-09T10:15:00+08:00 至 2026-05-12T00:00:00+08:00 |
| 本节题目数 | 5 |

---

### 1. 向量1（类和对象）（20 分）

- 题目 ID: `112`
- 限制: 1s / 128MB

#### 题目描述

n个有序数a1,a2,...,an组成的数组称为n维向量。 为n维向量定义CVector类，包含私有数据成员：

int *data；//存储n维向量

int n； //向量维数。

方法有：

（1）无参构造函数，设置n=5,data的数据分别为0,1,2,3,4；

（2）构造函数，用形参n1和数组a初始化n和data的数据；

（3）输出函数，按格式输出n维向量的值；

（4）析构函数。

主函数输入数据，生成CVector对象并调用输出函数测试。

#### 输入格式

输入n

输入n维向量

#### 输出格式

分别调用无参和带参构造函数生成2个CVector对象，输出它们的值。

#### 样例输入
```text
6
10 1 2 3 4 5
```

#### 样例输出
```text
0 1 2 3 4
10 1 2 3 4 5
```

#### 答案解析

- **建模思路**：`CVector` 用 `int* data` 保存动态数组，用 `n` 保存维数。无参构造固定生成 `0 1 2 3 4`，有参构造根据输入数组复制数据。
- **核心算法**：构造函数里分配 `new int[n]`，逐个复制元素；析构函数中 `delete[] data`。
- **输出易错点**：输出一行向量，每个数字之间一个空格，行末不要额外输出说明文字。

#### 参考实现提示

只要类里有动态数组，就要考虑析构；后续题会继续要求拷贝构造，建议从本题就写清楚资源归属。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class CVector {
    int *data, n;

  public:
    CVector() : n(5) {
        data = new int[n];
        for (int i = 0; i < n; i++)
            data[i] = i;
    }
    CVector(int n1, int a[]) : n(n1) {
        data = new int[n];
        for (int i = 0; i < n; i++)
            data[i] = a[i];
    }
    CVector(const CVector &o) : n(o.n) {
        data = new int[n];
        for (int i = 0; i < n; i++)
            data[i] = o.data[i];
    }
    ~CVector() {
        delete[] data;
    }
    void print() const {
        for (int i = 0; i < n; i++) {
            if (i)
                cout << ' ';
            cout << data[i];
        }
        cout << endl;
    }
};
int main() {
    int n;
    cin >> n;
    int *a = new int[n];
    for (int i = 0; i < n; i++)
        cin >> a[i];
    CVector v1;
    CVector v2(n, a);
    v1.print();
    v2.print();
    delete[] a;
    return 0;
}
```

---

### 2. 向量2（友元及拷贝构造）（20 分）

- 题目 ID: `113`
- 限制: 1s / 128MB

#### 题目描述

在题目向量1的代码上添加类CVector的友元函数add，计算两个向量的和(对应分量相加)。

add定义如下:

```text
CVector add(const CVector v1, const CVector v2) //函数头不可修改。
```

主函数输入数据，生成两个向量对象v1,v2，调用add(v1, v2).print()输出向量v1 + v2的计算结果。（假设print()为CVector类中的输出函数。）

可根据需要，为类CVector添加拷贝构造函数及其它成员函数。

#### 输入格式

第一行，输入测试次数t

每组测试数据格式如下:

向量维数n

第一个n维向量值

第二个n维向量值

#### 输出格式

对每组测试数据，输出两个n维向量与它们的和

#### 样例输入
```text
2
3
1 2 3
4 5 6
5
1 2 3 4 5
-1 2 4 6 10
```

#### 样例输出
```text
1 2 3
4 5 6
5 7 9
1 2 3 4 5
-1 2 4 6 10
0 4 7 10 15
```

#### 答案解析

- **建模思路**：沿用 `CVector`，增加友元函数 `add(const CVector v1, const CVector v2)` 返回一个新向量。
- **核心算法**：由于参数是按值传递，会调用拷贝构造函数；如果没有深拷贝，析构时会重复释放同一块数组。本题必须写拷贝构造和赋值逻辑，至少保证拷贝构造深拷贝。
- **输出易错点**：每组输出三行：第一个向量、第二个向量、求和结果。

#### 参考实现提示

在 `add` 中创建临时数组或临时 `CVector`，逐项相加后返回；维数按题意相同处理即可。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class CVector {
    int *data, n;

  public:
    CVector(int n = 0) : n(n) {
        data = n ? new int[n] : nullptr;
    }
    CVector(int n, int a[]) : n(n) {
        data = new int[n];
        for (int i = 0; i < n; i++)
            data[i] = a[i];
    }
    CVector(const CVector &o) : n(o.n) {
        data = n ? new int[n] : nullptr;
        for (int i = 0; i < n; i++)
            data[i] = o.data[i];
    }
    CVector &operator=(const CVector &o) {
        if (this != &o) {
            int *p = o.n ? new int[o.n] : nullptr;
            for (int i = 0; i < o.n; i++)
                p[i] = o.data[i];
            delete[] data;
            data = p;
            n = o.n;
        }
        return *this;
    }
    ~CVector() {
        delete[] data;
    }
    void print() const {
        for (int i = 0; i < n; i++) {
            if (i)
                cout << ' ';
            cout << data[i];
        }
        cout << endl;
    }
    friend CVector add(const CVector v1, const CVector v2);
};
CVector add(const CVector v1, const CVector v2) {
    CVector r(v1.n);
    for (int i = 0; i < v1.n; i++)
        r.data[i] = v1.data[i] + v2.data[i];
    return r;
}
int main() {
    int t;
    cin >> t;
    while (t--) {
        int n;
        cin >> n;
        int *a = new int[n], *b = new int[n];
        for (int i = 0; i < n; i++)
            cin >> a[i];
        for (int i = 0; i < n; i++)
            cin >> b[i];
        CVector v1(n, a), v2(n, b);
        v1.print();
        v2.print();
        add(v1, v2).print();
        delete[] a;
        delete[] b;
    }
    return 0;
}
```

---

### 3. 向量3（静态成员）（20 分）

- 题目 ID: `114`
- 限制: 1s / 128MB

#### 题目描述

为向量1题目实现的CVector类添加私有静态成员sum，在初始化对象的同时，统计所有对象的n维向量和sum。

主函数生成多个对象，测试向量和。

可根据需要自行添加需要的静态成员函数，添加非静态成员函数不得分。

#### 输入格式

测试次数t

每组测试数据格式如下：

输入m,表示n维向量的数目

后跟m行，每行格式：向量维数n  n维向量值

#### 输出格式

对每组测试数据，先按输入顺序输出m个向量，每个向量一行；最后一行输出所有向量的分量和sum。

#### 样例输入
```text
2
2
5 1 2 3 4 5
3 4 5 6
3
2 1 2 
3 10 20 30
2 11 22
```

#### 样例输出
```text
1 2 3 4 5
4 5 6
30
1 2
10 20 30
11 22
96
```

#### 答案解析

- **建模思路**：静态成员 `sum` 用来统计当前一组测试里所有向量元素的总和，而不是统计对象个数。
- **核心算法**：每创建一个 `CVector`，构造函数把本向量所有元素累加到 `CVector::sum`。每组测试开始前要把 `sum` 清零。
- **输出易错点**：题目样例会先输出每个向量，再输出总和。因此对象创建或打印顺序要和输入顺序一致。

#### 参考实现提示

可增加 `static void resetSum()` 和 `static int getSum()`，题目允许添加静态成员函数，不建议用非静态函数返回总和。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class CVector {
    int *data, n;
    static int sum;

  public:
    CVector(int n, int a[]) : n(n) {
        data = new int[n];
        for (int i = 0; i < n; i++) {
            data[i] = a[i];
            sum += a[i];
        }
    }
    CVector(const CVector &) = delete;
    CVector &operator=(const CVector &) = delete;
    ~CVector() {
        delete[] data;
    }
    void print() const {
        for (int i = 0; i < n; i++) {
            if (i)
                cout << ' ';
            cout << data[i];
        }
        cout << endl;
    }
    static void reset() {
        sum = 0;
    }
    static int getSum() {
        return sum;
    }
};
int CVector::sum = 0;
int main() {
    int t;
    cin >> t;
    while (t--) {
        CVector::reset();
        int m;
        cin >> m;
        CVector **v = new CVector *[m];
        for (int i = 0; i < m; i++) {
            int n;
            cin >> n;
            int *a = new int[n];
            for (int j = 0; j < n; j++)
                cin >> a[j];
            v[i] = new CVector(n, a);
            delete[] a;
        }
        for (int i = 0; i < m; i++)
            v[i]->print();
        cout << CVector::getSum() << endl;
        for (int i = 0; i < m; i++)
            delete v[i];
        delete[] v;
    }
    return 0;
}
```

---

### 4. 向量4（类复合）（20 分）

- 题目 ID: `115`
- 限制: 1s / 128MB

#### 题目描述

为向量1题目中实现的CVector类增加成员函数float Average()，计算n维向量的平均值并返回。

定义CStudent类，私有数据成员为：

string name;  // 姓名

CVector score;  // n个成绩

（1）添加构造函数，用形参name1、n1、数组a1初始化CStudent类对象。

（2）添加输出函数，按样例格式输出CStudent对象值。

主函数输入数据，测试CStudent对象。

#### 输入格式

输入多行，每行格式为：学生姓名 科目n   n个成绩

#### 输出格式

对每行测试数据，生成学生对象，输出如下数据：

学生姓名 n个成绩 成绩的平均值(保留2位小数)

#### 样例输入
```text
wangwu  5  90 80 70 100 90
lisi 3 100 90 100
```

#### 样例输出
```text
wangwu 90 80 70 100 90 86.00
lisi 100 90 100 96.67
```

#### 答案解析

- **建模思路**：`CStudent` 内部包含一个 `CVector score`，这是类复合。学生类负责姓名，向量类负责保存成绩和计算平均值。
- **核心算法**：`CVector::Average()` 遍历成绩求和后除以 `n`；`CStudent::print()` 先输出姓名，再调用成绩向量输出成绩，最后输出平均值。
- **输出易错点**：平均值保留两位小数，使用 `fixed << setprecision(2)`；输入行数未知，使用 `while (cin >> name >> n)` 读到文件结束。

#### 参考实现提示

由于 `CStudent` 构造函数要初始化成员对象，推荐使用初始化列表：`CStudent(...): name(name1), score(n1, a1) {}`。

#### 完整可运行参考代码

```cpp
#include <iomanip>
#include <iostream>
#include <string>
using namespace std;
class CVector {
    int *data, n;

  public:
    CVector(int n = 0, int a[] = nullptr) : n(n) {
        data = n ? new int[n] : nullptr;
        for (int i = 0; i < n; i++)
            data[i] = a[i];
    }
    CVector(const CVector &o) : n(o.n) {
        data = n ? new int[n] : nullptr;
        for (int i = 0; i < n; i++)
            data[i] = o.data[i];
    }
    CVector &operator=(const CVector &o) {
        if (this != &o) {
            int *p = o.n ? new int[o.n] : nullptr;
            for (int i = 0; i < o.n; i++)
                p[i] = o.data[i];
            delete[] data;
            data = p;
            n = o.n;
        }
        return *this;
    }
    ~CVector() {
        delete[] data;
    }
    double Average() const {
        double s = 0;
        for (int i = 0; i < n; i++)
            s += data[i];
        return s / n;
    }
    void print() const {
        for (int i = 0; i < n; i++)
            cout << ' ' << data[i];
    }
};
class CStudent {
    string name;
    CVector score;

  public:
    CStudent(string n, int m, int a[]) : name(n), score(m, a) {}
    void print() const {
        cout << name;
        score.print();
        cout << ' ' << fixed << setprecision(2) << score.Average() << endl;
    }
};
int main() {
    string name;
    int n;
    while (cin >> name >> n) {
        int *a = new int[n];
        for (int i = 0; i < n; i++)
            cin >> a[i];
        CStudent(name, n, a).print();
        delete[] a;
    }
    return 0;
}
```

---

### 5. 向量5（友元类）（20 分）

- 题目 ID: `116`
- 限制: 1s / 128MB

#### 题目描述

（1）在向量CVector类的代码上，定义n阶矩阵类CMatrix，包含私有数据成员data存储矩阵数据，n存储矩阵阶数。

（2）将CMatrix定义为CVector的友元类。

（3）为CMatrix添加成员函数：CVector multi(const CVector &v1)，计算n阶矩阵与n维向量v1的乘积。

（4）为CMatrix添加成员函数，判定矩阵与向量v1是否可计算乘积。

（5）为CMatrix添加需要的构造函数、析构函数和其它成员函数。

主函数输入数据，测试矩阵与向量的乘积。

动态创建n阶矩阵示例代码如下：

int n;
int** data;
int i, j;
cin >> n;
// 先创建n行
data = new int* [n];
// 再创建n列
for (i = 0; i < n; i++)
{
    data[i] = new int[n];
}
// 写入矩阵
for (i = 0; i < n; i++)
{
    for (j = 0; j < n; j++)
    {
        cin >> data[i][j];
    }

}

#### 输入格式

测试次数t

对每组测试数据，格式如下

第一行，矩阵阶数n  

n阶矩阵

向量维数m

m维向量

#### 输出格式

对每组测试数据，若矩阵与向量不能计算乘积，输出error；否则输出计算结果

#### 样例输入
```text
1
3
1 0 0
0 1 0
0 0 1
3
1 2 3
```

#### 样例输出
```text
1 2 3
```

#### 答案解析

- **建模思路**：`CMatrix` 是 `CVector` 的友元类，因此矩阵成员函数可以直接访问向量的 `n` 和 `data`。
- **核心算法**：只有矩阵阶数 `n` 等于向量维数 `m` 时才能相乘。结果向量第 `i` 项为 `sum(matrix[i][j] * v[j])`。
- **输出易错点**：不能相乘时只输出 `error`；可以相乘时输出结果向量一行。

#### 参考实现提示

二维动态数组要逐行 `new`，析构时也要逐行 `delete[]`，最后释放行指针数组。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class CMatrix;
class CVector {
    int *data, n;

  public:
    CVector(int n = 0) : n(n) {
        data = n ? new int[n] : nullptr;
    }
    CVector(int n, int a[]) : n(n) {
        data = new int[n];
        for (int i = 0; i < n; i++)
            data[i] = a[i];
    }
    CVector(const CVector &o) : n(o.n) {
        data = n ? new int[n] : nullptr;
        for (int i = 0; i < n; i++)
            data[i] = o.data[i];
    }
    CVector &operator=(const CVector &o) {
        if (this != &o) {
            int *p = o.n ? new int[o.n] : nullptr;
            for (int i = 0; i < o.n; i++)
                p[i] = o.data[i];
            delete[] data;
            data = p;
            n = o.n;
        }
        return *this;
    }
    ~CVector() {
        delete[] data;
    }
    void print() const {
        for (int i = 0; i < n; i++) {
            if (i)
                cout << ' ';
            cout << data[i];
        }
        cout << endl;
    }
    friend class CMatrix;
};
class CMatrix {
    int **data, n;

  public:
    CMatrix(int n) : n(n) {
        data = new int *[n];
        for (int i = 0; i < n; i++) {
            data[i] = new int[n];
            for (int j = 0; j < n; j++)
                cin >> data[i][j];
        }
    }
    ~CMatrix() {
        for (int i = 0; i < n; i++)
            delete[] data[i];
        delete[] data;
    }
    bool can(const CVector &v) const {
        return n == v.n;
    }
    CVector multi(const CVector &v) const {
        CVector r(n);
        for (int i = 0; i < n; i++) {
            r.data[i] = 0;
            for (int j = 0; j < n; j++)
                r.data[i] += data[i][j] * v.data[j];
        }
        return r;
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        int n;
        cin >> n;
        CMatrix m(n);
        int k;
        cin >> k;
        int *a = new int[k];
        for (int i = 0; i < k; i++)
            cin >> a[i];
        CVector v(k, a);
        if (m.can(v))
            m.multi(v).print();
        else
            cout << "error" << endl;
        delete[] a;
    }
    return 0;
}
```

---

## 实验9. 期中模拟

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1247` |
| 开放时间 | 2026-04-28T10:11:00+08:00 至 2026-05-09T00:00:00+08:00 |
| 本节题目数 | 4 |

---

### 1. 机器人变身（类与对象）（25 分）

- 题目 ID: `119`
- 限制: 1s / 128MB

#### 题目描述

编写一个机器人类，包含属性有机器名、血量、伤害值、防御值、类型和等级。其中血量、伤害和防御和等级、类型相关：

普通型机器人，类型为N，血量、伤害、防御是等级的5倍

攻击型机器人，类型为A，伤害是等级的10倍，其他属性和普通的一样

防御型机器人，类型为D，防御是等级的10倍，其他属性和普通的一样

生命型机器人，类型为H，血量是等级的50倍，其他属性和普通的一样。

机器人操作包括：打印、各个属性的获取和设置方法，构造函数可有可无，根据需要自行编写，

编写一个全局函数用于机器人变身，使得各种类型机器人能够相互转变。参数包括机器人对象指针和变身后的机器人类型，功能是修改机器人类型，并更改相关的属性。如果变身类型和机器人原来的类型不同，则执行变身功能，并返回true；如果变身类型和原来类型相同，则不执行变身，并返回false.

要求所有数据成员都是私有属性，用C++语言和面向对象设计思想来编程实现上述要求

#### 输入格式

第一行输入t，表示要执行t次机器人变身

接着每两行，一行输入一个机器人的属性，包括机器名、类型、等级，机器名最大长度为20，另一行输入变身类型

依次类推输入

#### 输出格式

每行输出变身后的机器人信息，要求调用机器人的打印方法来输出，即使机器人不变身也输出

属性输出依次为：名称、类型、等级、血量、伤害、防御

最后一行输出执行变身的次数

#### 样例输入
```text
3
X001 N 5
H
X002 A 5
D
X003 D 5
D
```

#### 样例输出
```text
X001--H--5--250--25--25
X002--D--5--25--25--50
X003--D--5--25--25--50
The number of robot transform is 2
```

#### 答案解析

- **建模思路**：机器人类保存名称、类型、等级、血量、伤害、防御。属性值完全由类型和等级决定，因此设置类型后应统一重新计算属性。
- **核心算法**：普通型 `N` 三项都是 `5*level`；攻击型 `A` 的伤害是 `10*level`；防御型 `D` 的防御是 `10*level`；生命型 `H` 的血量是 `50*level`。
- **输出易错点**：全局变身函数只有在新类型不同于旧类型时返回 `true`，最后统计真正变身的次数；每次都要打印机器人状态。

#### 参考实现提示

写一个私有成员函数 `update()` 根据 `type` 和 `level` 刷新三项属性，构造和变身都调用它。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
class Robot {
    string name;
    char type;
    int level, hp, atk, def;
    void update() {
        hp = atk = def = level * 5;
        if (type == 'A')
            atk = level * 10;
        else if (type == 'D')
            def = level * 10;
        else if (type == 'H')
            hp = level * 50;
    }

  public:
    Robot(string n, char t, int l) : name(n), type(t), level(l) {
        update();
    }
    char getType() const {
        return type;
    }
    void setType(char t) {
        type = t;
        update();
    }
    void print() const {
        cout << name << "--" << type << "--" << level << "--" << hp << "--" << atk << "--" << def
             << endl;
    }
};
bool transform(Robot *p, char t) {
    if (p->getType() == t)
        return false;
    p->setType(t);
    return true;
}
int main() {
    int t, cnt = 0;
    cin >> t;
    while (t--) {
        string n;
        char tp, to;
        int lv;
        cin >> n >> tp >> lv >> to;
        Robot r(n, tp, lv);
        if (transform(&r, to))
            cnt++;
        r.print();
    }
    cout << "The number of robot transform is " << cnt << endl;
    return 0;
}
```

---

### 2. 虚拟电话（构造与析构）（25 分）

- 题目 ID: `120`
- 限制: 1s / 128MB

#### 题目描述

虚拟电话包含属性：电话号、状态、机主姓名。

1、电话号是一个类，它包含号码和类型，其中号码是整数类型，类型用单个字母表示用户类别，A表示政府，B表示企业、C表示个人。类操作包括构造、属性的获取和设置等方法，根据需要自行编写。

2、状态用一个数字表示，1表示在用，0表示未用，

3、机主姓名是一个字符串

 

电话操作包括：构造、析构、打印和查询。

1、构造函数需要考虑复合类成员的构造，并且输出提示信息。假设电话号码为12345678，则构造函数输出"12345678 constructed."

2、打印是输出电话的相关信息，其中如果电话状态是在用则输出use；状态是未用则输出unuse，输出格式看示例。

3、析构函数是输出提示信息。假设电话号为12345678，则析构函数输出"12345678 destructed. "

4、查询操作是根据给定的号码查询电话，如果电话自身号码和给定号码不相同，则返回0；如果电话自身号码和给定号码相同，则返回1

 

用C++和面向对象思想实现以下要求：

1、输入相关数据，创建三个电话对象，并通过构造方法初始化。

2、输入若干个电话号码，通过查询操作查询这些号码是否在三个电话对象中，如果不存在输出"wrong number."，存在则调用打印操作输出电话信息，具体看输出样例。

#### 输入格式

头三行输入三个电话信息，每行包括电话号码、电话类型、状态、机主姓名，机主姓名最大长度为20

接着一行输入t，表示有t个号码要查询

接着t行输入t个电话号码

#### 输出格式

先输出三个电话对象的构造提示；再输出t个号码的查询结果；程序结束时按对象析构顺序输出析构提示。

#### 样例输入
```text
80000001 A 1 tom
80000002 B 0 ken
80000003 C 1 mary
3
50000002
80000003
80000002
```

#### 样例输出
```text
80000001 constructed.
80000002 constructed.
80000003 constructed.
wrong number.
Phone=80000003--Type=C--State=use--Owner=mary
Phone=80000002--Type=B--State=unuse--Owner=ken
80000003 destructed.
80000002 destructed.
80000001 destructed.
```

#### 答案解析

- **建模思路**：电话号码本身设计成一个类，电话类组合电话号码对象、状态和机主姓名。构造和析构都要按号码输出提示。
- **核心算法**：先创建 3 个电话对象，因此会先输出 3 行 constructed。查询时逐个比较号码，找到则打印电话信息，否则输出 `wrong number.`。
- **输出易错点**：析构顺序和对象创建顺序相反，若定义为普通数组，程序结束会自动按反序析构，正好符合样例。

#### 参考实现提示

状态 `1` 输出 `use`，状态 `0` 输出 `unuse`；查询函数可返回布尔值，打印函数只负责格式。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
class PhoneNumber {
    int number;
    char type;

  public:
    PhoneNumber(int n = 0, char t = 'A') : number(n), type(t) {}
    int getNumber() const {
        return number;
    }
    char getType() const {
        return type;
    }
};
class Phone {
    PhoneNumber num;
    int state;
    string owner;

  public:
    Phone(int n, char t, int s, string o) : num(n, t), state(s), owner(o) {
        cout << num.getNumber() << " constructed." << endl;
    }
    ~Phone() {
        cout << num.getNumber() << " destructed." << endl;
    }
    bool query(int n) const {
        return n == num.getNumber();
    }
    void print() const {
        cout << "Phone=" << num.getNumber() << "--Type=" << num.getType()
             << "--State=" << (state ? "use" : "unuse") << "--Owner=" << owner << endl;
    }
};
int main() {
    int n[3], st[3];
    char tp[3];
    string ow[3];
    for (int i = 0; i < 3; i++)
        cin >> n[i] >> tp[i] >> st[i] >> ow[i];
    Phone p1(n[0], tp[0], st[0], ow[0]), p2(n[1], tp[1], st[1], ow[1]),
        p3(n[2], tp[2], st[2], ow[2]);
    Phone *p[3] = {&p1, &p2, &p3};
    int q;
    cin >> q;
    while (q--) {
        int x;
        cin >> x;
        bool ok = false;
        for (int i = 0; i < 3; i++)
            if (p[i]->query(x)) {
                p[i]->print();
                ok = true;
                break;
            }
        if (!ok)
            cout << "wrong number." << endl;
    }
    return 0;
}
```

---

### 3. 银行账户（拷贝构造）（25 分）

- 题目 ID: `121`
- 限制: 1s / 128MB

#### 题目描述

银行账户包括余额、利率、号码、类型等属性，其中号是固定8位整数，类型表示个人还是企业账户，如果是个人用P标识，企业用E标识。

账户又分为活期账户和定期账户，活期利率为0.5%，定期利率为1.5%。

账户操作有计息、查询操作。计息操作是根据利率计算利息，并把利息增加到余额中，并做相关信息输出。查询操作是输出账户的全部信息。

创建一个活期账户，并通过构造函数初始化各个属性。然后通过拷贝构造来创建一个定期账户，信息与活期账户基本相同，不同之处包括：定期账户号码是活期账户号码加50000000（7个0）；利率是定期利率。

要求所有数据成员都是私有属性

用C++语言的类与对象思想来编写程序实现上述要求

#### 输入格式

输入测试个数t，表示有t个活期账户

先输入一个活期账户的账户号码、账户类型、余额（根据输入创建活期账户和定期账户）

接着输入两个操作符，第一个操作符对活期账户操作，第二个操作符对定期账户进行操作。若输入大写字母C表示计息操作，若输入大写字母P表示查询操作

以此类推输入其他账户的信息和操作

#### 输出格式

每两行输出一对活期账户和定期账户的操作结果。

#### 样例输入
```text
2
12345678 P 10000
C P
23456789 E 20000
P C
```

#### 样例输出
```text
Account=12345678--sum=10050
Account=62345678--Person--sum=10000--rate=0.015
Account=23456789--Enterprise--sum=20000--rate=0.005
Account=73456789--sum=20300
```

#### 答案解析

- **建模思路**：先创建活期账户，再用拷贝构造生成定期账户。定期账户复制原账户类型和余额，但账号加 `50000000`，利率改为 `0.015`。
- **核心算法**：活期利率为 `0.005`，定期利率为 `0.015`。操作符 `C` 表示计息并输出 `Account=账号--sum=新余额`；`P` 表示查询并输出账号、账户类型、余额和利率。
- **输出易错点**：账户类型 `P` 输出 `Person`，`E` 输出 `Enterprise`；利息计算后余额要更新。

#### 参考实现提示

拷贝构造函数中不能简单完全复制账号和利率，需要按题意转换成定期账户。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class Account {
    int no;
    char type;
    double sum, rate;

  public:
    Account(int n, char t, double s) : no(n), type(t), sum(s), rate(0.005) {}
    Account(const Account &a) : no(a.no + 50000000), type(a.type), sum(a.sum), rate(0.015) {}
    void calc() {
        sum += sum * rate;
        cout << "Account=" << no << "--sum=" << sum << endl;
    }
    void print() const {
        cout << "Account=" << no << "--" << (type == 'P' ? "Person" : "Enterprise")
             << "--sum=" << sum << "--rate=" << rate << endl;
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        int no;
        char tp, op1, op2;
        double s;
        cin >> no >> tp >> s >> op1 >> op2;
        Account a(no, tp, s), b(a);
        if (op1 == 'C')
            a.calc();
        else
            a.print();
        if (op2 == 'C')
            b.calc();
        else
            b.print();
    }
    return 0;
}
```

---

### 4. 电视遥控（静态+友元）【期中模拟】（25 分）

- 题目 ID: `122`
- 限制: 1s / 128MB

#### 题目描述

电视机包含音量、模式、频道号等属性，其中模式分为TV和DVD两种。电视机在TV模式下，将播放相应频道的内容；在DVD模式下，电视机使用统一的频道号播放DVD的内容，频道号统一为99。另外，电视机采用静态成员的方法共享两个数据：播放电视的电视机数量和播放DVD的电视机数量，初始都为0。

 

电视机操作包括打印、相关静态函数、属性的获取和设置等，根据需要自行编写。

现编写一个遥控器函数，通过友元方法对电视机进行控制，它的参数包括电视机对象、模式、变化音量、频道号，无返回值。函数操作包括：

1、对电视机对象进行模式设置，如果设置为DVD模式，则频道号参数一定是99；如果设置TV模式，则要把频道号设置相应的值。

2、根据变化音量进行调整，例如原有音量为50，现输入变化音量为-30，则50-30=20，音量最终为20。音量值最低为0，最高为100，低于0时置为0，高于100时置为100。

3、更新当前播放电视和播放DVD的电视机数量

4、调用电视机对象的打印方法输出电视相关信息

提示：如果电视机原来模式和参数传递的模式是相同的，那么实际操作就是调整音量、切换频道和输出信息。

注意：函数第一个参数必须是一个电视机对象，不可以是整数类型，可以是对象、或对象指针、或对象引用，根据需要自行编写。

用动态数组方法创建n台电视机，从1开始编号，频道号为编号，音量初始为50，模式为TV，然后通过遥控器函数对电视机进行控制。

所有类的数据成员都是私有属性。请使用C++语言和面向对象思想来实现上述要求

#### 输入格式

第一行输入n，表示有n台电视台

第二行输入t，表示将执行t次遥控操作

接着输入t行，每行依次输入电视机编号i、模式k、频道号x和变化音量，其中i表示第i台电视机，k为1表示TV模式，k为2表示DVD模式。

#### 输出格式

每行输出执行遥控操作后的电视机信息

最后一行输出当前播放电视和播放DVD的电视机数量。

具体格式看样例

#### 样例输入
```text
10
5
3 1 11 20
4 2 99 -20
5 2 99 80
5 1 55 -60
6 2 99 -70
```

#### 样例输出
```text
第3号电视机--TV模式--频道11--音量70
第4号电视机--DVD模式--频道99--音量30
第5号电视机--DVD模式--频道99--音量100
第5号电视机--TV模式--频道55--音量40
第6号电视机--DVD模式--频道99--音量0
播放电视的电视机数量为8
播放DVD的电视机数量为2
```

#### 答案解析

- **建模思路**：电视类保存编号、音量、模式、频道；静态成员保存当前 TV 模式数量和 DVD 模式数量。遥控器函数作为友元直接修改电视对象状态。
- **核心算法**：初始化时所有电视都是 TV 模式，所以 `tvCount=n`、`dvdCount=0`。每次遥控若模式改变，要先减少旧模式计数，再增加新模式计数。音量调整后必须限制在 `[0,100]`。
- **输出易错点**：DVD 模式频道固定为 `99`；TV 模式使用输入频道；最后输出两行当前数量。

#### 参考实现提示

把模式切换、音量调整、打印拆开写，静态计数只在模式真正改变时更新。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class TV {
    int id, vol, mode, ch;
    static int tvn, dvdn;

  public:
    TV(int i = 0) : id(i), vol(50), mode(1), ch(i) {
        if (i)
            tvn++;
    }
    void setId(int i) {
        id = i;
        vol = 50;
        mode = 1;
        ch = i;
        tvn++;
    }
    void print() const {
        cout << "第" << id << "号电视机--" << (mode == 1 ? "TV模式" : "DVD模式") << "--频道" << ch
             << "--音量" << vol << endl;
    }
    static void printCount() {
        cout << "播放电视的电视机数量为" << tvn << endl << "播放DVD的电视机数量为" << dvdn << endl;
    }
    friend void remote(TV &, int, int, int);
};
int TV::tvn = 0;
int TV::dvdn = 0;
void remote(TV &x, int m, int c, int dv) {
    if (x.mode != m) {
        if (x.mode == 1)
            TV::tvn--;
        else
            TV::dvdn--;
        if (m == 1)
            TV::tvn++;
        else
            TV::dvdn++;
    }
    x.mode = m;
    x.ch = m == 2 ? 99 : c;
    x.vol += dv;
    if (x.vol < 0)
        x.vol = 0;
    if (x.vol > 100)
        x.vol = 100;
    x.print();
}
int main() {
    int n, t;
    cin >> n;
    TV *a = new TV[n + 1];
    for (int i = 1; i <= n; i++)
        a[i].setId(i);
    cin >> t;
    while (t--) {
        int i, k, x, v;
        cin >> i >> k >> x >> v;
        remote(a[i], k, x, v);
    }
    TV::printCount();
    delete[] a;
    return 0;
}
```

---

## 实验8. 静态成员与友元

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1230` |
| 开放时间 | 2026-04-21T10:15:00+08:00 至 2026-04-28T00:00:00+08:00 |
| 本节题目数 | 5 |

---

### 1. 旅馆顾客统计（静态成员）（20 分）

- 题目 ID: `59`
- 限制: 1s / 128MB

#### 题目描述

编写程序，统计某旅馆住宿客人的总数和收入总额。要求输入客人的姓名，输出客人编号（2015+顺序号，顺序号4位，如第1位为0001，第2位为0002，依此类推）、姓名、总人数以及收入总额。总人数和收入总额用静态成员，其他属性采用普通的数据成员。旅馆类声明如下：

class Hotel
{
private:
    static int totalCustNum; // 顾客总人数
    static float totalEarning; // 旅店总收入
    static float rent; // 每个顾客的房租
    char *customerName; // 顾客姓名
    int customerId; // 顾客编号
public:
    // totalCustNum++，customerId按照totalCustNum生成
    Hotel(char* customer);
    ~Hotel(); //记得delete customerName
    void Display(); //相应输出顾客姓名、顾客编号、总人数、总收入
};

#### 输入格式

第1行：输入旅馆单个顾客房租

第2行开始，依次输入顾客姓名，0表示输入结束, 姓名的最大字符长度为20

#### 输出格式

每行依次输出顾客信息和当前旅馆信息。包括顾客姓名，顾客编号，旅馆当前总人数，旅馆当前总收入。

#### 样例输入
```text
150  
张三 李四 王五 0
```

#### 样例输出
```text
张三 20150001 1 150
李四 20150002 2 300
王五 20150003 3 450
```

#### 答案解析

- **建模思路**：每个顾客是一个 `Hotel` 对象，顾客总数、总收入、房租是所有对象共享的数据，所以使用静态成员。
- **核心算法**：构造顾客对象时 `totalCustNum++`，顾客编号为 `20150000 + totalCustNum`，总收入增加一次房租。
- **输出易错点**：编号要保持 8 位效果，例如第一个是 `20150001`；每读入一个姓名就立即输出当前统计。

#### 参考实现提示

姓名是 `char*` 时构造函数里要动态分配并复制，析构函数释放；也可用 `string` 简化，但要注意题目给出的类声明。

#### 完整可运行参考代码

```cpp
#include <cstring>
#include <iostream>
using namespace std;
class Hotel {
    static int totalCustNum;
    static float totalEarning;
    static float rent;
    char *customerName;
    int customerId;

  public:
    static void setRent(float r) {
        rent = r;
    }
    Hotel(char *customer) {
        customerName = new char[strlen(customer) + 1];
        strcpy(customerName, customer);
        totalCustNum++;
        customerId = 20150000 + totalCustNum;
        totalEarning += rent;
    }
    Hotel(const Hotel &) = delete;
    Hotel &operator=(const Hotel &) = delete;
    ~Hotel() {
        delete[] customerName;
    }
    void Display() {
        cout << customerName << ' ' << customerId << ' ' << totalCustNum << ' ' << totalEarning
             << endl;
    }
};
int Hotel::totalCustNum = 0;
float Hotel::totalEarning = 0;
float Hotel::rent = 0;
int main() {
    float r;
    cin >> r;
    Hotel::setRent(r);
    char name[25];
    while (cin >> name && strcmp(name, "0") != 0) {
        Hotel h(name);
        h.Display();
    }
    return 0;
}
```

---

### 2. 银行账户（静态成员与友元函数）（20 分）

- 题目 ID: `57`
- 限制: 1s / 128MB

#### 题目描述

银行账户类的基本描述如下：

class Account
{
private:
    static float count; // 账户总余额
    static float interestRate;
    string accno, accname;
    float balance;
public:
    Account(string ac, string na, float ba);
    ~Account();
    void deposit(float amount); // 存款
    void withdraw(float amount); // 取款
    float getBalance(); // 获取账户余额
    void show(); // 显示账户所有基本信息
    static float getCount(); // 获取账户总余额
    static void setInterestRate(float rate); // 设置利率
    static float getInterestRate(); // 获取利率
};

要求如下：

实现该银行账户类

为账户类Account增加一个友元函数，实现账户结息，要求输出结息后的余额（结息余额=账户余额+账户余额*利率）。友元函数声明形式为 friend void update(Account& a);

在main函数中，定义一个Account类型的指针数组，让每个指针指向动态分配的Account对象，并调用成员函数测试存款、取款、显示等函数，再调用友元函数测试进行结息。

大家可以根据实际需求在类内添加其他方法，也可以修改成员函数的参数设置

#### 输入格式

第1行：利率
第2行：账户数目n
第3行开始，每行依次输入一个账户的“账号”、“姓名”、“余额”、存款数，取款数。

#### 输出格式

第1行开始，每行输出一个账户的相关信息，包括账号、姓名、存款后的余额、存款后结息余额、取款后余额。

最后一行输出所有账户的余额。

#### 样例输入
```text
0.01
3
201501 张三 10000 1000 2000
201502 李四 20000 2000 4000
201503 王二 80000 4000 6000
```

#### 样例输出
```text
201501 张三 11000 11110 9110
201502 李四 22000 22220 18220
201503 王二 84000 84840 78840
106170
```

#### 答案解析

- **建模思路**：每个账户保存自身余额，静态成员 `count` 保存所有账户当前总余额，静态利率由输入设置。
- **核心算法**：先存款并输出存款后余额；调用友元函数 `update(Account& a)` 结息，余额变为 `balance + balance * interestRate`；再取款并输出取款后余额。
- **输出易错点**：总余额 `count` 必须随着存款、结息、取款同步变化，最后输出的是所有账户最终余额之和。

#### 参考实现提示

友元函数能直接改 `balance`，但同时别忘了更新静态总余额，否则最后一行会错。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
class Account {
    static float count, interestRate;
    string accno, accname;
    float balance;

  public:
    Account(string ac, string na, float ba) : accno(ac), accname(na), balance(ba) {
        count += balance;
    }
    void deposit(float a) {
        balance += a;
        count += a;
    }
    void withdraw(float a) {
        balance -= a;
        count -= a;
    }
    float getBalance() {
        return balance;
    }
    string no() {
        return accno;
    }
    string name() {
        return accname;
    }
    static float getCount() {
        return count;
    }
    static void setInterestRate(float r) {
        interestRate = r;
    }
    friend void update(Account &a);
};
float Account::count = 0;
float Account::interestRate = 0;
void update(Account &a) {
    float x = a.balance * Account::interestRate;
    a.balance += x;
    Account::count += x;
}
int main() {
    float rate;
    int n;
    cin >> rate >> n;
    Account::setInterestRate(rate);
    while (n--) {
        string no, name;
        float b, d, w;
        cin >> no >> name >> b >> d >> w;
        Account a(no, name, b);
        a.deposit(d);
        cout << a.no() << ' ' << a.name() << ' ' << a.getBalance() << ' ';
        update(a);
        cout << a.getBalance() << ' ';
        a.withdraw(w);
        cout << a.getBalance() << endl;
    }
    cout << Account::getCount() << endl;
    return 0;
}
```

---

### 3. 距离计算（友元函数）（20 分）

- 题目 ID: `56`
- 限制: 1s / 128MB

#### 题目描述

Point类的基本形式如下：

class Point
{
private:
    double x, y;
public:
    Point(double xx, double yy); // 构造函数
};

请完成如下要求：

1.实现Point类；

2.为Point类增加一个友元函数double Distance(Point &a, Point &b),用于计算两点之间的距离。直接访问Point对象的私有数据进行计算。

3.编写main函数，输入两点坐标值，计算两点之间的距离。

#### 输入格式

第1行：输入需计算距离的点对的数目

第2行开始，每行依次输入两个点的x和y坐标

#### 输出格式

每行依次输出一组点对之间的距离（结果直接取整数部分,不四舍五入 ）

#### 样例输入
```text
3
1 0 2 1
2 3 2 4
2 2 4 4
```

#### 样例输出
```text
1
1
2
```

#### 答案解析

- **建模思路**：`Point` 封装 `x,y`，友元函数 `Distance(Point&, Point&)` 直接访问私有坐标。
- **核心算法**：两点距离为 `sqrt((x1-x2)^2 + (y1-y2)^2)`。
- **输出易错点**：题目要求“直接取整数部分，不四舍五入”，所以把 `double` 强制转换成 `int` 输出即可。

#### 参考实现提示

不要使用 `round` 或设置小数精度；样例 `sqrt(8)=2.828` 输出 `2`。

#### 完整可运行参考代码

```cpp
#include <cmath>
#include <iostream>
using namespace std;
class Point {
    double x, y;

  public:
    Point(double xx = 0, double yy = 0) : x(xx), y(yy) {}
    friend double Distance(Point &a, Point &b);
};
double Distance(Point &a, Point &b) {
    return sqrt((a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y));
}
int main() {
    int t;
    cin >> t;
    while (t--) {
        double x1, y1, x2, y2;
        cin >> x1 >> y1 >> x2 >> y2;
        Point a(x1, y1), b(x2, y2);
        cout << (int)Distance(a, b) << endl;
    }
    return 0;
}
```

---

### 4. 复数运算（友元函数）（20 分）

- 题目 ID: `58`
- 限制: 1s / 128MB

#### 题目描述

复数类的声明如下：

class Complex
{
private:
	double real; // 实部
	double imag; // 虚部
public:
	Complex();
	Complex(double r, double i);
	// 友元函数，复数c1 + c2(二参数对象相加)
	friend Complex addCom(const Complex& c1, const Complex& c2);
	// 友元函数，输出类对象c的有关数据(c为参数对象)
	friend void outCom(const Complex& c);
};

 

要求如下：

1.     实现复数类和友元函数addCom和outCom。

2.   参考addCom函数为复数类增加一个友元函数minusCom，用于实现两个复数的减法

3.     在main函数中，通过友元函数，实现复数的加减法和复数的输出。

#### 输入格式

第1行：第1个复数的实部和虚部

第2行：需进行运算的次数，注意：是连续运算。具体结果可参考样例

    第3行开始，每行输入运算类型，以及参与运算的复数的实部与虚部。“+”表示复数相加，“-”表示复数相减。

#### 输出格式

每行输出复数运算后的结果，复数输出格式为“(实部,虚部)”。

#### 样例输入
```text
10 10
3
+ 20 10
- 15 5
+ 5 25
```

#### 样例输出
```text
(30,20)
(15,15)
(20,40)
```

#### 答案解析

- **建模思路**：`Complex` 保存实部和虚部，友元函数实现加法、减法和输出。
- **核心算法**：连续运算时维护一个“当前复数”。读到 `+` 就把当前复数加上新复数，读到 `-` 就减去新复数，每次更新后输出。
- **输出易错点**：输出格式固定为 `(实部,虚部)`，样例不带额外空格，也不写 `i`。

#### 参考实现提示

`outCom` 只负责输出当前对象；不要每次都从初始复数重新计算。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class Complex {
    double real, imag;

  public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}
    friend Complex addCom(const Complex &c1, const Complex &c2);
    friend Complex minusCom(const Complex &c1, const Complex &c2);
    friend void outCom(const Complex &c);
};
Complex addCom(const Complex &a, const Complex &b) {
    return Complex(a.real + b.real, a.imag + b.imag);
}
Complex minusCom(const Complex &a, const Complex &b) {
    return Complex(a.real - b.real, a.imag - b.imag);
}
void outCom(const Complex &c) {
    cout << '(' << c.real << ',' << c.imag << ')' << endl;
}
int main() {
    double r, i;
    cin >> r >> i;
    Complex cur(r, i);
    int n;
    cin >> n;
    while (n--) {
        char op;
        cin >> op >> r >> i;
        Complex x(r, i);
        cur = op == '+' ? addCom(cur, x) : minusCom(cur, x);
        outCom(cur);
    }
    return 0;
}
```

---

### 5. 日期时间合并输出（友元函数）（20 分）

- 题目 ID: `62`
- 限制: 1s / 128MB

#### 题目描述

已知日期类Date，有属性：年、月、日，其他成员函数根据需要自行编写，注意该类没有输出的成员函数

已知时间类Time，有属性：时、分、秒，其他成员函数根据需要自行编写，注意该类没有输出的成员函数

现在编写一个全局函数把时间和日期的对象合并起来一起输出，

函数原型为：void display(const Date &d, const Time &t)

函数输出要求为：

1、时分秒输出长度固定2位，不足2位补0

2、年份输出长度固定为4位，月和日的输出长度固定2位，不足2位补0

例如2017年3月3日19时5分18秒

则输出为：2017-03-03 19:05:18

 

程序要求

1、把函数display作为时间类、日期类的友元

2、分别创建一个日期对象和时间对象，保存日期的输入和时间的输入

3、调用display函数实现日期和时间的合并输出

#### 输入格式

第一行输入t表示有t组示例

接着一行输入三个整数，表示年月日

再接着一行输入三个整数，表示时分秒

依次输入t组示例

#### 输出格式

每行输出一个日期和时间合并输出结果

输出t行

#### 样例输入
```text
2
2017 3 3
19 5 18
1988 12 8
5 16 4
```

#### 样例输出
```text
2017-03-03 19:05:18
1988-12-08 05:16:04
```

#### 答案解析

- **建模思路**：`Date` 保存年月日，`Time` 保存时分秒，二者都把 `display` 声明为友元函数。
- **核心算法**：`display(const Date&, const Time&)` 同时访问两个对象私有成员，组合输出完整时间戳。
- **输出易错点**：年份宽度 4，月日时分秒宽度 2，不足补 `0`，需要 `setfill('0') << setw(...)`。

#### 参考实现提示

设置 `setfill('0')` 后会持续生效；如果后续还输出普通数字，可以再改回 `setfill(' ')`。

#### 完整可运行参考代码

```cpp
#include <iomanip>
#include <iostream>
using namespace std;
class Time;
class Date {
    int y, m, d;

  public:
    Date(int y = 0, int m = 0, int d = 0) : y(y), m(m), d(d) {}
    friend void display(const Date &, const Time &);
};
class Time {
    int h, m, s;

  public:
    Time(int h = 0, int m = 0, int s = 0) : h(h), m(m), s(s) {}
    friend void display(const Date &, const Time &);
};
void display(const Date &d, const Time &t) {
    cout << setfill('0') << setw(4) << d.y << '-' << setw(2) << d.m << '-' << setw(2) << d.d << ' '
         << setw(2) << t.h << ':' << setw(2) << t.m << ':' << setw(2) << t.s << endl;
}
int main() {
    int n;
    cin >> n;
    while (n--) {
        int y, mo, d, h, mi, s;
        cin >> y >> mo >> d >> h >> mi >> s;
        display(Date(y, mo, d), Time(h, mi, s));
    }
    return 0;
}
```

---

## 实验7. 构造与拷贝构造

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1220` |
| 开放时间 | 2026-04-14T10:15:00+08:00 至 2026-04-21T00:00:00+08:00 |
| 本节题目数 | 5 |

---

### 1. Equation(类与对象+构造)（20 分）

- 题目 ID: `45`
- 限制: 1s / 128MB

#### 题目描述

建立一个类Equation，表达方程ax2+bx+c=0。类中至少包含以下方法：

1、无参构造（abc默认值为1.0、1.0、0）与有参构造函数，用于初始化a、b、c的值；

2、set方法，用于修改a、b、c的值

3、getRoot方法，求出方程的根。

一元二次方程的求根公式如下：

一元二次方程的求解分三种情况，如下：

#### 输入格式

输入测试数据的组数t

第一组a、b、c

第二组a、b、c

#### 输出格式

输出方程的根，结果到小数点后2位

在C++中，输出指定精度的参考代码如下：

```cpp
#include <iostream>
#include <iomanip> //必须包含这个头文件
using namespace std;
int main()
{
    double a = 3.14;
    cout << fixed << setprecision(3) << a << endl; //输出小数点后3位
    return 0;
}
```

#### 样例输入
```text
3
2 4 2
2 2 2
2 8 2
```

#### 样例输出
```text
x1=x2=-1.00
x1=-0.50+0.87i x2=-0.50-0.87i
x1=-0.27 x2=-3.73
```

#### 答案解析

- **建模思路**：`Equation` 保存 `a,b,c`，构造函数和 `set` 负责初始化，`getRoot` 负责分类输出根。
- **核心算法**：判别式 `delta=b*b-4*a*c`。`delta==0` 输出两个相等实根；`delta>0` 输出两个实根；`delta<0` 输出共轭复根，实部 `-b/(2a)`，虚部 `sqrt(-delta)/(2a)`。
- **输出易错点**：所有数保留两位小数。复根格式如 `x1=-0.50+0.87i x2=-0.50-0.87i`。

#### 参考实现提示

本题输入都是二次方程场景，通常不需要处理 `a==0`；若想更稳，可额外判断。

#### 完整可运行参考代码

```cpp
#include <cmath>
#include <iomanip>
#include <iostream>
using namespace std;
class Equation {
    double a, b, c;

  public:
    Equation(double a = 1, double b = 1, double c = 0) : a(a), b(b), c(c) {}
    void set(double aa, double bb, double cc) {
        a = aa;
        b = bb;
        c = cc;
    }
    void getRoot() {
        double d = b * b - 4 * a * c;
        cout << fixed << setprecision(2);
        if (fabs(d) < 1e-9) {
            cout << "x1=x2=" << -b / (2 * a) << endl;
        } else if (d > 0) {
            double x1 = (-b + sqrt(d)) / (2 * a), x2 = (-b - sqrt(d)) / (2 * a);
            cout << "x1=" << x1 << " x2=" << x2 << endl;
        } else {
            double real = -b / (2 * a), imag = sqrt(-d) / (2 * a);
            cout << "x1=" << real << "+" << imag << "i x2=" << real << "-" << imag << "i" << endl;
        }
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        double a, b, c;
        cin >> a >> b >> c;
        Equation e(a, b, c);
        e.getRoot();
    }
    return 0;
}
```

---

### 2. 对象是怎样构造的(拷贝构造函数)（20 分）

- 题目 ID: `50`
- 限制: 1s / 128MB

#### 题目描述

某个类包含一个整型数据成员.程序运行时若输入0表示用缺省方式定义一个类对象;输入1及一个整数表示用带一个参数的构造函数构造一个类对象;输入2及一个整数表示构造2个类对象，一个用输入的参数构造，另一个用前一个对象构造。试完成该类的定义和实现。

#### 输入格式

测试数据的组数 t

第一组数

第二组数

......

#### 输出格式

第一个对象构造输出

第二个对象构造输出

......

#### 样例输入
```text
3
0
1 10
2 20
```

#### 样例输出
```text
Constructed by default, value = 0
Constructed using one argument constructor, value = 10
Constructed using one argument constructor, value = 20
Constructed using copy constructor, value = 20
```

#### 答案解析

- **建模思路**：类中只有一个整数成员，但三个构造函数要分别输出不同提示：默认构造、单参构造、拷贝构造。
- **核心算法**：输入 `0` 创建默认对象；输入 `1 x` 创建单参对象；输入 `2 x` 先用 `x` 创建对象，再用该对象拷贝构造第二个对象。
- **输出易错点**：构造函数中直接输出指定句子，主函数不要重复输出。

#### 参考实现提示

`2 x` 会输出两行，因为先调用单参构造，再调用拷贝构造。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class Test {
    int value;

  public:
    Test() : value(0) {
        cout << "Constructed by default, value = " << value << endl;
    }
    Test(int v) : value(v) {
        cout << "Constructed using one argument constructor, value = " << value << endl;
    }
    Test(const Test &t) : value(t.value) {
        cout << "Constructed using copy constructor, value = " << value << endl;
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        int op, x;
        cin >> op;
        if (op == 0) {
            Test a;
        } else if (op == 1) {
            cin >> x;
            Test a(x);
        } else {
            cin >> x;
            Test a(x);
            Test b(a);
        }
    }
    return 0;
}
```

---

### 3. 电话号码升位(拷贝构造函数)（20 分）

- 题目 ID: `51`
- 限制: 1s / 128MB

#### 题目描述

定义一个电话号码类CTelNumber,包含1个字符指针数据成员,以及构造、析构、打印及拷贝构造函数。

字符指针是用于动态创建一个字符数组，然后保存外来输入的电话号码

构造函数的功能是为对象设置键盘输入的7位电话号码，

拷贝构造函数的功能是用原来7位号码的对象升位为8位号码对象,也就是说拷贝构造的对象是源对象的升级.电话升位的规则是原2、3、4开头的电话号码前面加8，原5、6、7、8开头的前面加2。

注意:电话号码只能全部是数字字符，且与上述情况不符的输入均为非法

#### 输入格式

测试数据的组数 t

第一个7位号码

第二个7位号码

......

#### 输出格式

第一个号码升位后的号码

第二个号码升位后的号码

......

如果号码升级不成功，则输出报错信息，具体看示例

#### 样例输入
```text
3
6545889
3335656
565655
```
```text
2
1234567
22a2567
```

#### 样例输出
```text
26545889
83335656
Illegal phone number
```
```text
Illegal phone number
Illegal phone number
```

#### 答案解析

- **建模思路**：原对象保存 7 位号码，拷贝构造函数根据原号码生成升位后的 8 位号码。
- **核心算法**：先检查长度必须为 7 且全部是数字；首位为 `2/3/4` 时前面加 `8`，首位为 `5/6/7/8` 时前面加 `2`，其他首位非法。
- **输出易错点**：非法号码输出 `Illegal phone number`；合法时只输出升位后的号码。

#### 参考实现提示

因为成员是字符指针，构造、拷贝构造和析构都要做深拷贝/释放，不能直接共享同一个指针。

#### 完整可运行参考代码

```cpp
#include <cctype>
#include <cstring>
#include <iostream>
using namespace std;
class CTelNumber {
    char *p;
    bool ok;

  public:
    CTelNumber(const char *s) {
        p = new char[strlen(s) + 1];
        strcpy(p, s);
        ok = true;
    }
    CTelNumber(const CTelNumber &o) {
        ok = o.ok && strlen(o.p) == 7;
        for (int i = 0; o.p[i] && ok; i++)
            if (!isdigit((unsigned char)o.p[i]))
                ok = false;
        if (!ok || !strchr("2345678", o.p[0])) {
            p = new char[1];
            p[0] = '\0';
            ok = false;
        } else {
            p = new char[9];
            p[0] = strchr("234", o.p[0]) ? '8' : '2';
            strcpy(p + 1, o.p);
        }
    }
    ~CTelNumber() {
        delete[] p;
    }
    void print() const {
        if (ok)
            cout << p << endl;
        else
            cout << "Illegal phone number" << endl;
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        char s[50];
        cin >> s;
        CTelNumber old(s), now(old);
        now.print();
    }
    return 0;
}
```

---

### 4. 软件备份(拷贝构造函数)（20 分）

- 题目 ID: `52`
- 限制: 1s / 128MB

#### 题目描述

软件作为一种对象也可以用类来描述，软件的属性包括软件名称、类型(分别用O、T和B表示原版、试用版还是备份)、有效截止日期(用CDate类子对象表示)和存储介质(分别用D、H和U表示光盘、磁盘和U盘)等。软件拷贝可通过拷贝构造函数来实现，此时在拷贝构造函数中软件类型改成“B”, 存储介质改为"H"，其它不变。试完成该类的拷贝构造、构造和打印(包括从2015年4月7日算起有效期还有多少天，是否过期)成员函数的实现。

当输入软件有效截止日期是0年0月0日，表示无日期限制，为unlimited；当输入日期在2015年4月7日之前，则是过期，表示为expired；如果输入日期在2015年4月7日之后，则显示之后的剩余天数。具体输出信息看输出范例。

附CDate类的实现：

class CDate
{
    private:
        int year, month, day;
    public:
        CDate(int y, int m, int d);
        bool isLeapYear();
        int getYear();
        int getMonth();
        int getDay();
        int getDayofYear();
};

CDate::CDate(int y, int m, int d)
{
    year = y, month = m,day = d;
}

bool CDate::isLeapYear()
{
    return (year % 4 == 0 && year % 100 != 0) || year % 400 == 0;
}

int CDate::getYear()
{
    return year;
}

int CDate::getMonth()
{
    return month;
}

int CDate::getDay()
{
    return day;
}

int CDate::getDayofYear()
{
    int i, sum = day;
    int a[13] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
    if (isLeapYear())
    {
        a[2]++;
    }
    // 求日期的天数
    for (i = 0; i < month; i++)
    {
        sum += a[i];
    }
    return sum;
}

#### 输入格式

测试数据的组数 t

第一个软件名称

第一个软件类型  第一个软件介质类型  第一个软件有效期年 月 日

第二个软件名称

第二个软件类型 第二个软件介质类型 第二个软件有效期年 月 日

......

注意：软件名称最大长度为30；

#### 输出格式

name: 第一个软件名称

type: 第一个软件类型

media: 第一个软件介质类型

第一个软件2015-4-7后的有效天数

 

name: 第一个软件名称

type: backup

media: hard disk

第一个软件2015-4-7后的有效天数

......

#### 样例输入
```text
3
Photoshop_CS5
O D 0 0 0
Audition_3.0
B U 2015 2 3
Visual_Studio_2010
T H 2015 5 5
```
```text
2
Photoshop_CS5
O D 2015 4 8
Audition_3.0
B U 2023 4 7
```

#### 样例输出
```text
name:Photoshop_CS5
type:original
media:optical disk
this software has unlimited use

name:Photoshop_CS5
type:backup
media:hard disk
this software has unlimited use

name:Audition_3.0
type:backup
media:USB disk
this software has expired

name:Audition_3.0
type:backup
media:hard disk
this software has expired

name:Visual_Studio_2010
type:trial
media:hard disk
this software is going to be expired in 28 days

name:Visual_Studio_2010
type:backup
media:hard disk
this software is going to be expired in 28 days
```
```text
name:Photoshop_CS5
type:original
media:optical disk
this software is going to be expired in 1 days

name:Photoshop_CS5
type:backup
media:hard disk
this software is going to be expired in 1 days

name:Audition_3.0
type:backup
media:USB disk
this software is going to be expired in 2922 days

name:Audition_3.0
type:backup
media:hard disk
this software is going to be expired in 2922 days
```

#### 答案解析

- **建模思路**：软件类组合 `CDate` 截止日期，保存名称、类型和介质。拷贝构造表示制作备份：类型强制为 `B`，介质强制为 `H`。
- **核心算法**：把当前日期固定为 `2015-4-7`。截止日期为 `0 0 0` 输出 unlimited；早于当前日期输出 expired；晚于当前日期则计算相差天数。
- **输出易错点**：原软件和备份软件都要输出，中间有空行；类型和介质要转成完整英文，如 `original`、`backup`、`hard disk`。

#### 参考实现提示

计算日期差可以写一个 `daysFromZero(date)`，把年月日转成累计天数后相减，闰年逐年统计最稳。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
class CDate {
    int y, m, d;

  public:
    CDate(int y = 0, int m = 0, int d = 0) : y(y), m(m), d(d) {}
    bool leap(int yy) const {
        return yy % 400 == 0 || (yy % 4 == 0 && yy % 100 != 0);
    }
    int days() const {
        if (y == 0 && m == 0 && d == 0)
            return -1;
        int sum = 0;
        for (int i = 1; i < y; i++)
            sum += leap(i) ? 366 : 365;
        int a[13] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
        if (leap(y))
            a[2] = 29;
        for (int i = 1; i < m; i++)
            sum += a[i];
        return sum + d;
    }
};
class Software {
    string name;
    char type, media;
    CDate date;
    string ts() const {
        return type == 'O' ? "original" : type == 'T' ? "trial" : "backup";
    }
    string ms() const {
        return media == 'D' ? "optical disk" : media == 'H' ? "hard disk" : "USB disk";
    }
    void left() const {
        int end = date.days(), now = CDate(2015, 4, 7).days();
        if (end == -1)
            cout << "this software has unlimited use" << endl;
        else if (end < now)
            cout << "this software has expired" << endl;
        else
            cout << "this software is going to be expired in " << end - now << " days" << endl;
    }

  public:
    Software(string n, char t, char m, CDate d) : name(n), type(t), media(m), date(d) {}
    Software(const Software &s) : name(s.name), type('B'), media('H'), date(s.date) {}
    void print() const {
        cout << "name:" << name << endl << "type:" << ts() << endl << "media:" << ms() << endl;
        left();
        cout << endl;
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        string n;
        char tp, md;
        int y, m, d;
        cin >> n >> tp >> md >> y >> m >> d;
        Software a(n, tp, md, CDate(y, m, d)), b(a);
        a.print();
        b.print();
    }
    return 0;
}
```

---

### 5. 手机服务（构造+拷贝构造+堆）（20 分）

- 题目 ID: `53`
- 限制: 1s / 128MB

#### 题目描述

设计一个类来实现手机的功能。它包含私有属性：号码类型、号码、号码状态、停机日期；包含方法：构造、拷贝构造、打印、停机。

1、号码类型表示用户类别，只用单个字母，A表示机构，B表示企业、C表示个人

2、号码是11位整数，用一个字符串表示

3、号码状态用一个数字表示，1、2、3分别表示在用、未用、停用

4、停机日期是一个日期对象指针，在初始化时该成员指向空，该日期类包含私有属性年月日，以及构造函数和打印函数等

----------------------------------------

 

5、构造函数的作用就是接受外来参数，并设置各个属性值,并输出提示信息，看示例输出

6、拷贝构造的作用是复制已有对象的信息，并输出提示信息，看示例输出。

      想一下停机日期该如何复制，没有停机如何复制？？已经停机又如何复制？？

 

7、打印功能是把对象的所有属性都输出，输出格式看示例

8、停机功能是停用当前号码，参数是停机日期，无返回值，操作是把状态改成停用，并将停机日期指针创建为动态对象，并根据参数来设置停机日期，最后输出提示信息，看示例输出

-------------------------------------------

 

要求：在主函数中实现号码备份的功能，对已有的虚拟手机号的所有信息进行复制，并将号码类型改成D表示备份；将手机号码末尾加字母X

#### 输入格式

第一行输入t表示有t个号码

第二行输入6个参数，包括号码类型、号码、状态、停机的年、月、日，用空格隔开

依次输入t行

#### 输出格式

每个示例依次输出：构造新号码提示、原号码信息、拷贝构造提示、备份号码信息、停机提示、原号码停机后的信息。

每个示例之间用短划线（四个）分割开，看示例输出

#### 样例输入
```text
2
A 15712345678 1 2023 1 1
B 13287654321 2 2012 12 12
```
```text
1
C 15674561389 3 2020 1 2
```

#### 样例输出
```text
Construct a new phone 15712345678
类型=机构||号码=15712345678||State=在用
Construct a copy of phone 15712345678
类型=备份||号码=15712345678X||State=在用
Stop the phone 15712345678
类型=机构||号码=15712345678||State=停用||停机日期=2023.1.1
----
Construct a new phone 13287654321
类型=企业||号码=13287654321||State=未用
Construct a copy of phone 13287654321
类型=备份||号码=13287654321X||State=未用
Stop the phone 13287654321
类型=企业||号码=13287654321||State=停用||停机日期=2012.12.12
----
```
```text
Construct a new phone 15674561389
类型=个人||号码=15674561389||State=停用
Construct a copy of phone 15674561389
类型=备份||号码=15674561389X||State=停用
Stop the phone 15674561389
类型=个人||号码=15674561389||State=停用||停机日期=2020.1.2
----
```

#### 答案解析

- **建模思路**：手机号类保存类型、号码、状态和一个可能为空的停机日期指针。拷贝构造生成备份号码，类型改为 `D`，号码末尾加 `X`。
- **核心算法**：构造时停机日期指针先为空；调用停机函数时把状态改为 `3`，并在堆上创建日期对象。若复制一个已有停机日期的对象，要深拷贝日期。
- **输出易错点**：每组依次输出构造提示、原号码信息、拷贝提示、备份号码信息、停机提示、停机后的原号码信息，最后输出 `----`。

#### 参考实现提示

析构函数中释放停机日期指针；拷贝构造不能让两个对象共享同一个日期指针。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
class Date {
    int y, m, d;

  public:
    Date(int y = 0, int m = 0, int d = 0) : y(y), m(m), d(d) {}
    Date(const Date &o) : y(o.y), m(o.m), d(o.d) {}
    void print() const {
        cout << y << '.' << m << '.' << d;
    }
};
class Phone {
    char type;
    string num;
    int state;
    Date *stopDate;
    string ts() const {
        return type == 'A' ? "机构" : type == 'B' ? "企业" : type == 'C' ? "个人" : "备份";
    }
    string ss() const {
        return state == 1 ? "在用" : state == 2 ? "未用" : "停用";
    }

  public:
    Phone(char t, string n, int s) : type(t), num(n), state(s), stopDate(nullptr) {
        cout << "Construct a new phone " << num << endl;
    }
    Phone(const Phone &p)
        : type('D'), num(p.num + "X"), state(p.state),
          stopDate(p.stopDate ? new Date(*p.stopDate) : nullptr) {
        cout << "Construct a copy of phone " << p.num << endl;
    }
    Phone &operator=(const Phone &) = delete;
    ~Phone() {
        delete stopDate;
    }
    void print() const {
        cout << "类型=" << ts() << "||号码=" << num << "||State=" << ss();
        if (stopDate) {
            cout << "||停机日期=";
            stopDate->print();
        }
        cout << endl;
    }
    void stop(int y, int m, int d) {
        cout << "Stop the phone " << num << endl;
        state = 3;
        delete stopDate;
        stopDate = new Date(y, m, d);
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        char tp;
        string n;
        int s, y, m, d;
        cin >> tp >> n >> s >> y >> m >> d;
        Phone a(tp, n, s);
        a.print();
        Phone b(a);
        b.print();
        a.stop(y, m, d);
        a.print();
        cout << "----" << endl;
    }
    return 0;
}
```

---

## 实验6. 构造与析构

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1206` |
| 开放时间 | 2026-04-07T10:15:00+08:00 至 2026-04-14T00:00:00+08:00 |
| 本节题目数 | 5 |

---

### 1. Point(类与构造)（20 分）

- 题目 ID: `31`
- 限制: 1s / 128MB

#### 题目描述

下面是一个平面上的点的类定义，请在类外实现它的所有方法，并生成点测试它。

#### 输入格式

测试数据的组数 t

第一组测试数据点p1的x坐标   第一组测试数据点p1的y坐标  第一组测试数据点p2的x坐标   第一组测试数据点p2的y坐标

..........

#### 输出格式

输出p1到p2的距离，保留两位小数。详情参考输出样例。

在C++中，输出指定精度的参考代码如下：

```cpp
#include <iostream>
#include <iomanip> //必须包含这个头文件
using namespace std;
int main()
{
    double a = 3.14;
    cout << fixed << setprecision(2) << a << endl; //输出小数点后2位
    return 0;
}
```

#### 样例输入
```text
2
1 2 3 4
-1 0.5 -2 5
```

#### 样例输出
```text
Distance of Point(1.00,2.00) to Point(3.00,4.00) is 2.83
Distance of Point(-1.00,0.50) to Point(-2.00,5.00) is 4.61
```

#### 答案解析

- **建模思路**：`Point` 保存平面坐标，构造函数初始化坐标，成员函数计算到另一个点的距离。
- **核心算法**：距离公式为 `sqrt((x1-x2)^2 + (y1-y2)^2)`。
- **输出易错点**：坐标和距离都保留两位小数，格式固定为 `Distance of Point(...,...) to Point(...,...) is ...`。

#### 参考实现提示

输出前统一使用 `fixed << setprecision(2)`，这样坐标中的整数也会显示成 `1.00`。

#### 完整可运行参考代码

```cpp
#include <cmath>
#include <iomanip>
#include <iostream>
using namespace std;
class Point {
    double x, y;

  public:
    Point(double x = 0, double y = 0) : x(x), y(y) {}
    double dis(const Point &p) const {
        return sqrt((x - p.x) * (x - p.x) + (y - p.y) * (y - p.y));
    }
    void print() const {
        cout << "Point(" << fixed << setprecision(2) << x << "," << y << ")";
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        double x1, y1, x2, y2;
        cin >> x1 >> y1 >> x2 >> y2;
        Point a(x1, y1), b(x2, y2);
        cout << "Distance of ";
        a.print();
        cout << " to ";
        b.print();
        cout << " is " << fixed << setprecision(2) << a.dis(b) << endl;
    }
    return 0;
}
```

---

### 2. Date(类与构造)（20 分）

- 题目 ID: `32`
- 限制: 1s / 128MB

#### 题目描述

下面是一个日期类的定义，请在类外实现其所有的方法，并在主函数中生成对象测试之。

 

注意，在判断明天日期时，要加入跨月、跨年、闰年的判断

例如9月30日的明天是10月1日，12月31日的明天是第二年的1月1日

2月28日的明天要区分是否闰年，闰年则是2月29日，非闰年则是3月1日

#### 输入格式

测试数据的组数t

第一组测试数据的年 月 日

..........

要求第一个日期的年月日初始化采用构造函数，第二个日期的年月日初始化采用setDate方法，第三个日期又采用构造函数，第四个日期又采用setDate方法，以此类推。

#### 输出格式

输出今天的日期

输出明天的日期

#### 提示

C++中设置填充字符的代码参考如下：

cout << setfill('0') << setw(2) << month; //设置宽度为2，前面补'0'

需要头文件#include <iomanip>

#### 样例输入
```text
4
2012 1 3
2012 2 28
2012 3 31
2012 4 30
```

#### 样例输出
```text
Today is 2012/01/03
Tomorrow is 2012/01/04
Today is 2012/02/28
Tomorrow is 2012/02/29
Today is 2012/03/31
Tomorrow is 2012/04/01
Today is 2012/04/30
Tomorrow is 2012/05/01
```

#### 答案解析

- **建模思路**：`Date` 类保存年月日，构造函数和 `setDate` 都可初始化日期，成员函数负责输出今天和计算明天。
- **核心算法**：用月份天数数组处理跨月；闰年条件为能被 400 整除，或能被 4 整除但不能被 100 整除。若日期是 12 月 31 日，明天年份加一。
- **输出易错点**：年月日中的月和日固定两位补零，例如 `2012/01/03`。

#### 参考实现提示

题目要求奇偶次初始化方式不同：第 1、3、5... 个用构造函数，第 2、4、6... 个用 `setDate`。

#### 完整可运行参考代码

```cpp
#include <iomanip>
#include <iostream>
using namespace std;
class Date {
    int y, m, d;
    bool leap() const {
        return y % 400 == 0 || (y % 4 == 0 && y % 100 != 0);
    }
    int mdays() const {
        int a[13] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
        return m == 2 && leap() ? 29 : a[m];
    }

  public:
    Date(int y = 0, int m = 0, int d = 0) : y(y), m(m), d(d) {}
    void setDate(int yy, int mm, int dd) {
        y = yy;
        m = mm;
        d = dd;
    }
    void printToday() const {
        cout << "Today is " << setfill('0') << setw(4) << y << '/' << setw(2) << m << '/' << setw(2)
             << d << endl;
    }
    void printTomorrow() const {
        Date t = *this;
        t.d++;
        if (t.d > t.mdays()) {
            t.d = 1;
            t.m++;
            if (t.m > 12) {
                t.m = 1;
                t.y++;
            }
        }
        cout << "Tomorrow is " << setfill('0') << setw(4) << t.y << '/' << setw(2) << t.m << '/'
             << setw(2) << t.d << endl;
    }
};
int main() {
    int t;
    cin >> t;
    for (int i = 1; i <= t; i++) {
        int y, m, d;
        cin >> y >> m >> d;
        Date date;
        if (i % 2)
            date = Date(y, m, d);
        else
            date.setDate(y, m, d);
        date.printToday();
        date.printTomorrow();
    }
    return 0;
}
```

---

### 3. 分数类（类与构造）（20 分）

- 题目 ID: `33`
- 限制: 1s / 128MB

#### 题目描述

完成下列分数类的实现:

```text
class CFraction
{
private:
    int fz, fm;
public:
    CFraction(int fz_val, int fm_val);
    CFraction add(const CFraction &r);
    CFraction sub(const CFraction &r);
    CFraction mul(const CFraction &r);
    CFraction div(const CFraction &r);
    int getGCD(); // 求对象的分子和分母的最大公约数
    void print();
};
```

求两数a、b的最大公约数可采用辗转相除法，又称欧几里得算法，其步骤为:

1. 交换a, b使a > b;
2. 用a除b得到余数r,若r=0,则b为最大公约数,退出.
3. 若r不为0,则用b代替a, r代替b,此时a,b都比上一次的小,问题规模缩小了;
4. 继续第2步。

注意：如果分母是1的话，也按“分子/1”的方式输出。

#### 输入格式

测试数据的组数 t

第一组第一个分数

第一组第二个分数

第二组第一个分数

第二组第二个分数

......

#### 输出格式

第一组两个分数的和

第一组两个分数的差

第一组两个分数的积

第一组两个分数的商

第二组两个分数的和

第二组两个分数的差

第二组两个分数的积

第二组两个分数的商

......

#### 样例输入
```text
3
1/2
2/3
3/4
5/8
21/23
8/13
```

#### 样例输出
```text
7/6
-1/6
1/3
3/4

11/8
1/8
15/32
6/5

457/299
89/299
168/299
273/184
```

#### 答案解析

- **建模思路**：`CFraction` 保存分子 `fz` 和分母 `fm`，四则运算返回新的分数对象。
- **核心算法**：加减通分，乘法分子乘分子、分母乘分母，除法乘以倒数。每次结果都用最大公约数约分，并保证负号通常放在分子上。
- **输出易错点**：即使分母为 1，也按 `分子/1` 输出；每组四个结果后有一个空行。

#### 参考实现提示

输入形如 `1/2`，可以用 `char slash` 接住中间的 `/`：`cin >> a >> slash >> b`。

#### 完整可运行参考代码

```cpp
#include <cstdlib>
#include <iostream>
using namespace std;
class CFraction {
    int fz, fm;
    int gcd(int a, int b) {
        a = abs(a);
        b = abs(b);
        while (b) {
            int r = a % b;
            a = b;
            b = r;
        }
        return a ? a : 1;
    }
    void simp() {
        if (fm < 0) {
            fm = -fm;
            fz = -fz;
        }
        int g = gcd(fz, fm);
        fz /= g;
        fm /= g;
    }

  public:
    CFraction(int a = 0, int b = 1) : fz(a), fm(b) {
        simp();
    }
    CFraction add(const CFraction &r) {
        return CFraction(fz * r.fm + r.fz * fm, fm * r.fm);
    }
    CFraction sub(const CFraction &r) {
        return CFraction(fz * r.fm - r.fz * fm, fm * r.fm);
    }
    CFraction mul(const CFraction &r) {
        return CFraction(fz * r.fz, fm * r.fm);
    }
    CFraction div(const CFraction &r) {
        return CFraction(fz * r.fm, fm * r.fz);
    }
    void print() {
        cout << fz << '/' << fm << endl;
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        int a, b, c, d;
        char ch;
        cin >> a >> ch >> b >> c >> ch >> d;
        CFraction x(a, b), y(c, d);
        x.add(y).print();
        x.sub(y).print();
        x.mul(y).print();
        x.div(y).print();
        cout << endl;
    }
    return 0;
}
```

---

### 4. Point_Array(类+构造+对象数组)（20 分）

- 题目 ID: `34`
- 限制: 1s / 128MB

#### 题目描述

上面是我们曾经练习过的一个习题，请在原来代码的基础上作以下修改：

1、增加自写的析构函数；

2、将getDisTo方法的参数修改为getDisTo(const Point &p)；

3、根据输出的内容修改相应的构造函数。

然后在主函数中根据用户输入的数目建立Point数组，求出数组内距离最大的两个点之间的距离值。

#### 输入格式

测试数据的组数 t

第一组点的个数

第一个点的 x 坐标   y坐标

第二个点的 x坐标  y坐标

............

#### 输出格式

输出每组中距离最大的两个点以及其距离（存在多个距离都是最大值的情况下，输出下标排序最前的点组合。比如如果p[0]和p[9]、p[4]和p[5]之间的距离都是最大值，那么前一个是答案，因为p[0]排序最前）

创建每个 `Point` 对象时输出 `Constructor.`；释放对象数组时每个对象输出 `Distructor.`。

...........

在C++中，输出指定精度的参考代码如下：

```cpp
#include <iostream>
#include <iomanip> //必须包含这个头文件
using namespace std;
int main()
{
    double a = 3.141596;
    cout << fixed << setprecision(3) << a << endl; //输出小数点后3位
    return 0;
}
```

#### 样例输入
```text
2
4
0 0
5 0
5 5
2 10
3
-1 -8
0 9
5 0
```

#### 样例输出
```text
Constructor.
Constructor.
Constructor.
Constructor.
The longest distance is 10.44,between p[1] and p[3].
Distructor.
Distructor.
Distructor.
Distructor.
Constructor.
Constructor.
Constructor.
The longest distance is 17.03,between p[0] and p[1].
Distructor.
Distructor.
Distructor.
```

#### 答案解析

- **建模思路**：动态创建 `Point` 对象数组，每个对象构造时输出 `Constructor.`，数组释放时每个对象析构输出 `Distructor.`。
- **核心算法**：双重循环枚举所有点对 `(i,j)`，计算距离，记录最大距离。因为要输出下标排序最前的组合，只在 `dis > maxDis` 时更新，不在相等时更新。
- **输出易错点**：距离保留两位小数；析构提示题面拼写是 `Distructor.`，要按样例写。

#### 参考实现提示

用 `new Point[n]` 创建对象数组时会调用 n 次默认构造；如果坐标要由输入设置，构造后再调用 `set`。

#### 完整可运行参考代码

```cpp
#include <cmath>
#include <iomanip>
#include <iostream>
using namespace std;
class Point {
    double x, y;

  public:
    Point() {
        x = y = 0;
        cout << "Constructor." << endl;
    }
    ~Point() {
        cout << "Distructor." << endl;
    }
    void set(double a, double b) {
        x = a;
        y = b;
    }
    double getDisTo(const Point &p) const {
        return sqrt((x - p.x) * (x - p.x) + (y - p.y) * (y - p.y));
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        int n;
        cin >> n;
        Point *p = new Point[n];
        for (int i = 0; i < n; i++) {
            double x, y;
            cin >> x >> y;
            p[i].set(x, y);
        }
        double mx = -1;
        int a = 0, b = 1;
        for (int i = 0; i < n; i++)
            for (int j = i + 1; j < n; j++) {
                double d = p[i].getDisTo(p[j]);
                if (d > mx) {
                    mx = d;
                    a = i;
                    b = j;
                }
            }
        cout << "The longest distance is " << fixed << setprecision(2) << mx << ",between p[" << a
             << "] and p[" << b << "]." << endl;
        delete[] p;
    }
    return 0;
}
```

---

### 5. Stack(类与构造)（20 分）

- 题目 ID: `35`
- 限制: 1s / 128MB

#### 题目描述

上面是栈类的定义，栈是一种具有先进后出特点的线性表，请根据注释，完成类中所有方法的实现，并在主函数中测试之。

堆栈类的说明如下：

1. 堆栈的数据实际上是保存在数组a中，而a开始是一个指针，在初始化时，根据实际需求将a动态创建为数组，数组长度根据构造函数的参数决定。

2.size实际上就是数组的长度，当使用无参构造则size为10，当使用有参构造则size为s、

3.top表示数组下标，也表示数组中下一个存放数据的空白位置。

4.push操作表示堆栈的数组存放一个数据，例如一开始数组为空，则top为0，当有数据要入栈时，把数据存放在a[top]的位置，然后top加1指向下一个空白位置、数据进栈只能从栈顶进。

5.pop操作表示一个数据要出栈，数据出栈只能从栈顶出，先把top减1指向栈顶数据，然后把数据返回。

6.判断堆栈空的条件是top是否等于0，判断堆栈满的条件是top是否等于size

#### 输入格式

测试数据的组数 t

第一个栈的大小

第一个栈的元素列表，将该列表的元素依次进栈

..........

#### 输出格式

每组先输出构造提示，再将栈元素依次出栈并输出一行，最后输出析构提示。

#### 样例输入
```text
2
5
1 2 3 4 5
7
-1 2 8 0 -3 1 3
```

#### 样例输出
```text
Constructor.
5 4 3 2 1
Destructor.
Constructor.
3 1 -3 0 8 2 -1
Destructor.
```

#### 答案解析

- **建模思路**：栈类用动态数组保存元素，`size` 是容量，`top` 指向下一个可写位置。
- **核心算法**：`push` 把元素放到 `a[top]` 后 `top++`；`pop` 先 `top--`，再返回 `a[top]`。全部入栈后连续出栈，输出顺序自然是输入的逆序。
- **输出易错点**：构造时输出 `Constructor.`，析构时输出 `Destructor.`；每组数字输出一行。

#### 参考实现提示

本题输入数量刚好等于栈大小，但仍建议写 `isFull()` 和 `isEmpty()`，逻辑更清楚。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
class Stack {
    int *a, size, top;

  public:
    Stack(int s = 10) : size(s), top(0) {
        a = new int[size];
        cout << "Constructor." << endl;
    }
    ~Stack() {
        delete[] a;
        cout << "Destructor." << endl;
    }
    bool full() {
        return top == size;
    }
    bool empty() {
        return top == 0;
    }
    void push(int x) {
        if (!full())
            a[top++] = x;
    }
    int pop() {
        return a[--top];
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        int n;
        cin >> n;
        Stack st(n);
        for (int i = 0, x; i < n; i++) {
            cin >> x;
            st.push(x);
        }
        for (int i = 0; i < n; i++) {
            if (i)
                cout << ' ';
            cout << st.pop();
        }
        cout << endl;
    }
    return 0;
}
```

---

## 实验5. 类与对象

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1199` |
| 开放时间 | 2026-03-31T10:15:00+08:00 至 2026-04-07T00:00:00+08:00 |
| 本节题目数 | 5 |

---

### 1. 对象数组（类和对象）（20 分）

- 题目 ID: `27`
- 限制: 1s / 128MB

#### 题目描述

课堂上我们谈到类这个概念，比如第一题我们有学生类这个抽象的概念，成千上万个学生都具有同样的属性，但针对某个具体学生来说，他/她具有自己的鲜明个性，比如计算机专业的王海同学，信息工程学院的李慧同学，那么我们是定义一个该类的变量：Student  student; 假设该类包含姓名、学号、性别、所属学院、联系电话等；在程序运行过程，把变量student赋予不同值就可以让它表示是王海还是李慧，尝试定义一个学生数组（比如四个元素大小，请思考此时四个对象是如何初始化的呢？），然后输入四个不同学生各项属性给数组里学生初始化（最好定义一个输入函数），然后输出该学生对象数组的各对象的姓名。

#### 输入格式

输入数组元素的大小

依次每行输入每个对象的各项属性值，各属性值最大长度不超过30

#### 输出格式

每行输出一个学生类对象的姓名

#### 样例输入
```text
3
tom 2013333333 男 计算机学院 13766666666
Marry 2012222222 女 信息工程学院 13555555555
John  2014444444 男 管理学院 13888888888
```

#### 样例输出
```text
tom
Marry
John
```

#### 答案解析

- **建模思路**：定义 `Student` 类保存姓名、学号、性别、学院、电话，创建学生对象数组。
- **核心算法**：先读数组大小 `n`，再循环读入每个学生的五项属性，最后只输出每个对象的姓名。
- **输出易错点**：学院、姓名等字段中样例没有空格，用 `cin` 读取即可；每个姓名单独一行。

#### 参考实现提示

可写 `input()` 和 `printName()` 两个成员函数，让主函数只负责循环调用。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
class Student {
    string name, id, sex, school, phone;

  public:
    void input() {
        cin >> name >> id >> sex >> school >> phone;
    }
    void printName() {
        cout << name << endl;
    }
};
int main() {
    int n;
    cin >> n;
    Student *s = new Student[n];
    for (int i = 0; i < n; i++)
        s[i].input();
    for (int i = 0; i < n; i++)
        s[i].printName();
    delete[] s;
    return 0;
}
```

---

### 2. 存折类定义（类与对象）（20 分）

- 题目 ID: `156`
- 限制: 1s / 128MB

#### 题目描述

定义一个存折类CAccount，存折类具有帐号（account, long）、姓名（name,char[10])、余额（balance,float）等数据成员，可以实现存款（deposit,操作成功提示“saving ok!”)、取款（withdraw，操作成功提示“withdraw ok!”）和查询余额（check）的操作，取款金额必须在余额范围内，否则提示“sorry! over limit!”。编写主函数，建立这个类的对象并测试，输入账号、姓名、余额后，按照查询余额、存款、查询余额、取款、查询余额的顺序调用类方法并输出。

#### 输入格式

第一个存折的账号、姓名、余额

存款金额

取款金额

第二个存折的账号、姓名、余额

存款金额

取款金额

#### 输出格式

第一个存折的账户余额

存款操作结果

账户余额

取款操作结果

账户余额

第二个存折的账户余额

存款操作结果

账户余额

取款操作结果

账户余额

#### 样例输入
```text
9111 Tom 1000
500
1000
92220 John 500
500
1500
```

#### 样例输出
```text
Tom's balance is 1000
saving ok!
Tom's balance is 1500
withdraw ok!
Tom's balance is 500
John's balance is 500
saving ok!
John's balance is 1000
sorry! over limit!
John's balance is 1000
```

#### 答案解析

- **建模思路**：`CAccount` 保存账号、姓名和余额，成员函数完成存款、取款和查询余额。
- **核心算法**：存款直接增加余额并输出 `saving ok!`；取款前判断金额是否超过余额，足够才扣款并输出 `withdraw ok!`，否则余额不变并输出 `sorry! over limit!`。
- **输出易错点**：题目固定测试两个账户，每个账户按“查询、存款、查询、取款、查询”的顺序输出。

#### 参考实现提示

余额样例都是整数，使用 `float` 或 `double` 都可；输出不要强制两位小数。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
class CAccount {
    long account;
    string name;
    float balance;

  public:
    CAccount(long a, string n, float b) : account(a), name(n), balance(b) {}
    void deposit(float x) {
        balance += x;
        cout << "saving ok!" << endl;
    }
    void withdraw(float x) {
        if (x <= balance) {
            balance -= x;
            cout << "withdraw ok!" << endl;
        } else
            cout << "sorry! over limit!" << endl;
    }
    void check() {
        cout << name << "'s balance is " << balance << endl;
    }
};
int main() {
    for (int i = 0; i < 2; i++) {
        long a;
        string n;
        float b, d, w;
        cin >> a >> n >> b >> d >> w;
        CAccount acc(a, n, b);
        acc.check();
        acc.deposit(d);
        acc.check();
        acc.withdraw(w);
        acc.check();
    }
    return 0;
}
```

---

### 3. 身体评估（类与对象）（20 分）

- 题目 ID: `162`
- 限制: 1s / 128MB

#### 题目描述

评估成年人身体健康有多个指标，包括BMI、体脂率等

设计一个身体健康类，包含私有成员：姓名、身高(米)、体重(公斤)，腰围(厘米)，实现两个公有方法如下：

BMI方法，返回BMI数值(整数，四舍五入到个位)，计算公式BMI= 体重 / 身高的平方

体脂率方法，返回体脂率数值(浮点数)，计算过程如下：

1）参数a=腰围（cm）×0.74

2）参数b=体重（kg）×0.082+34.89

3）体脂肪重量（kg）=a－b

4）体脂率 = 体脂肪重量÷体重

其它方法根据需要自行定义

#### 输入格式

第一行输入t表示有t个测试实例

第二行起，每行输入四个参数：姓名、身高、体重、腰围，姓名的最大长度不超过20

依次输入t行

#### 输出格式

输出t行，每行输出一个实例的BMI和体脂率，BMI四舍五入到个位，体脂率小数数值精确到小数点后2位

 

在C++中，输出指定精度的参考代码如下：

```cpp
#include <iostream>
#include <iomanip> //必须包含这个头文件
using namespace std;
int main()
{
    double a = 3.14;
    cout << fixed << setprecision(2) << a << endl; //输出小数点后2位
    return 0;
}
```

#### 样例输入
```text
2
张高 1.85 78.5 85.2
李圆 1.55 67.6 77.3
```

#### 样例输出
```text
张高的BMI指数为23--体脂率为0.28
李圆的BMI指数为28--体脂率为0.25
```

#### 答案解析

- **建模思路**：身体健康类保存姓名、身高、体重、腰围，成员函数分别返回 BMI 和体脂率。
- **核心算法**：`BMI = 体重 / (身高*身高)`，四舍五入到整数可用 `round`；体脂率按题面四步公式计算。
- **输出易错点**：体脂率保留两位小数，格式是 `姓名的BMI指数为xx--体脂率为0.xx`。

#### 参考实现提示

需要包含 `<cmath>` 使用 `round`，包含 `<iomanip>` 控制体脂率精度。

#### 完整可运行参考代码

```cpp
#include <cmath>
#include <iomanip>
#include <iostream>
#include <string>
using namespace std;
class Body {
    string name;
    double height, weight, waist;

  public:
    Body(string n, double h, double w, double wa) : name(n), height(h), weight(w), waist(wa) {}
    int BMI() {
        return (int)round(weight / (height * height));
    }
    double fat() {
        double a = waist * 0.74, b = weight * 0.082 + 34.89;
        return (a - b) / weight;
    }
    void print() {
        cout << name << "的BMI指数为" << BMI() << "--体脂率为" << fixed << setprecision(2) << fat()
             << endl;
    }
};
int main() {
    int t;
    cin >> t;
    while (t--) {
        string n;
        double h, w, wa;
        cin >> n >> h >> w >> wa;
        Body(n, h, w, wa).print();
    }
    return 0;
}
```

---

### 4. 点和圆 (类与对象)（20 分）

- 题目 ID: `30`
- 限制: 1s / 128MB

#### 题目描述

设计一个点类Point，包含属性：x坐标和y坐标，方法：设定坐标（setPoint），获取x坐标（getX），获取y坐标（getY）

设计一个圆类Circle，包含属性：圆心坐标x和y、半径r；方法包括：

1. 设定圆心（setCenter），设置圆心x坐标和y坐标

2. 设定半径（setRadius），设置半径长度

3. 计算面积（getArea），计算公式：面积=3.14*r*r

4. 计算周长（getLength），计算公式：周长=2*3.14*r

5. 包含（contain），判断一个圆是否包含一个点，计算圆心到这个点的距离，然后和半径做比较，大于则不包含，小于等于则包含

注意：提交代码时必须用注释划分出三个区域：类定义、类实现、主函数，如下

```text
//-----类定义------
class XXX
{
    // 写类定义代码
};

//----类实现------
void XXX::process()
{
    // 写类定义代码
};

//-----主函数-----
int main()
{
    //自定义一些变量
    //创建一个圆对象和一个点对象
    //输入圆对象和点对象的属性数值，并做初始化
    //输出圆的面积和圆的周长
    //输出圆是否包含点，包含则输出yes，否则输出no
    return 0;
}
```

#### 输入格式

第一行输入圆的三个整数参数：圆心的x和y坐标，半径

第二行输入点的两个整数参数：x和y坐标

#### 输出格式

第一行输出圆的面积和周长，结果之间用空格隔开，输出精度到小数点后2位

第二行输出圆是否包含点，包含则输出yes，否则输出no

 

在C++中，输出指定精度的参考代码如下：

```cpp
#include <iostream>
#include <iomanip> //必须包含这个头文件
using namespace std;
int main()
{
    double a = 3.14;
    cout << fixed << setprecision(3) << a << endl; //输出小数点后3位
    return 0;
}
```

#### 提示

求两点距离的公式 dis =sqrt [ (x1-x2)^2  + (y1-y2)^2 ] ， ^2表示平方，sqrt表示开平方根，本公式只是表示含义，不是真实代码

在C++使用sqrt函数可以求平方根，头文件包含cmath

#### 样例输入
```text
1 1 1
2 2
```

#### 样例输出
```text
3.14 6.28
no
```

#### 答案解析

- **建模思路**：`Point` 类保存点坐标；`Circle` 类保存圆心和半径，并提供面积、周长和包含判断。
- **核心算法**：面积 `3.14*r*r`，周长 `2*3.14*r`。判断包含时计算点到圆心距离，若 `distance <= r` 输出 `yes`，否则输出 `no`。
- **输出易错点**：面积和周长保留两位小数；题目要求代码用注释划分类定义、类实现、主函数三个区域。

#### 参考实现提示

包含判断可以避免开根号：比较 `(dx*dx + dy*dy) <= r*r`，结果等价且更简单。

#### 完整可运行参考代码

```cpp
#include <iomanip>
#include <iostream>
using namespace std;
class Point {
    int x, y;

  public:
    void setPoint(int a, int b) {
        x = a;
        y = b;
    }
    int getX() {
        return x;
    }
    int getY() {
        return y;
    }
};
class Circle {
    int x, y, r;

  public:
    void setCenter(int a, int b) {
        x = a;
        y = b;
    }
    void setRadius(int rr) {
        r = rr;
    }
    double getArea() {
        return 3.14 * r * r;
    }
    double getLength() {
        return 2 * 3.14 * r;
    }
    bool contain(Point &p) {
        int dx = p.getX() - x, dy = p.getY() - y;
        return dx * dx + dy * dy <= r * r;
    }
};
int main() {
    int x, y, r, px, py;
    cin >> x >> y >> r >> px >> py;
    Circle c;
    c.setCenter(x, y);
    c.setRadius(r);
    Point p;
    p.setPoint(px, py);
    cout << fixed << setprecision(2) << c.getArea() << ' ' << c.getLength() << endl;
    cout << (c.contain(p) ? "yes" : "no") << endl;
    return 0;
}
```

---

### 5. 最胖的加菲（类与对象+数组）（20 分）

- 题目 ID: `160`
- 限制: 1s / 128MB

#### 题目描述

有一群猫猫，每只猫都有自己的名称和体重。

用类来描述猫，名称和体重都是私有属性，要求加入属性的get方法。其他函数根据需要自己定义

 

创建一个动态的猫对象数组，存储各只猫的名称和体重

根据猫的体重对数组做升序排序，并输出排序后每只猫的名称

 

题目涉及的数值均用整数处理

#### 输入格式

第一行输入n表示有n只猫

第二行输入一只猫的名称和体重，名称的最大长度为30

依次输入n行

#### 输出格式

输出一行，输出排序后的猫的名称

#### 样例输入
```text
4
巧克力胖三斤 1500
自来水瘦八两 400
芝士蛋糕肥六斤 3000
蔬菜沙拉轻四两 200
```

#### 样例输出
```text
蔬菜沙拉轻四两 自来水瘦八两 巧克力胖三斤 芝士蛋糕肥六斤
```

#### 答案解析

- **建模思路**：猫类保存名称和体重，动态对象数组保存所有猫。
- **核心算法**：按体重升序排序，然后输出排序后的猫名。可以使用冒泡排序、选择排序，也可以用 `sort` 配合比较函数。
- **输出易错点**：输出在一行，名字之间用一个空格，最后一个名字后是否有空格通常不影响，但建议不要多输出。

#### 参考实现提示

比较函数只比较体重；如果题目没有说明并列规则，保持原输入顺序更稳。

#### 完整可运行参考代码

```cpp
#include <algorithm>
#include <iostream>
#include <string>
using namespace std;
class Cat {
    string name;
    int weight;

  public:
    void input() {
        cin >> name >> weight;
    }
    string getName() const {
        return name;
    }
    int getWeight() const {
        return weight;
    }
};
bool cmp(const Cat &a, const Cat &b) {
    return a.getWeight() < b.getWeight();
}
int main() {
    int n;
    cin >> n;
    Cat *c = new Cat[n];
    for (int i = 0; i < n; i++)
        c[i].input();
    stable_sort(c, c + n, cmp);
    for (int i = 0; i < n; i++) {
        if (i)
            cout << ' ';
        cout << c[i].getName();
    }
    cout << endl;
    delete[] c;
    return 0;
}
```

---

## 实验4. 引用与结构

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1194` |
| 开放时间 | 2026-03-24T10:15:00+08:00 至 2026-03-31T00:00:00+08:00 |
| 本节题目数 | 5 |

---

### 1. 三数论大小（引用）（20 分）

- 题目 ID: `19`
- 限制: 1s / 128MB

#### 题目描述

输入三个整数，然后按照从大到小的顺序输出数值。

要求：定义一个函数，无返回值，函数参数是三个整数参数的引用，例如int &a, int &b, int &c。在函数内对三个参数进行排序。主函数调用这个函数进行排序。

要求：不能直接对三个整数进行排序，必须通过函数而且是引用的方法。

要求：输出必须在主函数进行。

#### 输入格式

第一行输入t表示有t个测试实例

第二行起，每行输入三个整数

输入t行

#### 输出格式

每行按照从大到小的顺序输出每个实例，三个整数之间用单个空格隔开

#### 样例输入
```text
3
2 4 6
88 99 77
111 333 222
```

#### 样例输出
```text
6 4 2
99 88 77
333 222 111
```

#### 答案解析

- **建模思路**：排序函数接收三个 `int&`，在函数内交换实参本身，主函数输出排序后的变量。
- **核心算法**：通过三次比较交换即可：保证 `a>=b`，再保证 `a>=c`，最后保证 `b>=c`。
- **输出易错点**：排序函数不能输出，输出必须在主函数进行。

#### 参考实现提示

引用参数能直接修改主函数变量，不需要返回数组或结构体。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
void sort3(int &a, int &b, int &c) {
    if (a < b)
        swap(a, b);
    if (a < c)
        swap(a, c);
    if (b < c)
        swap(b, c);
}
int main() {
    int t;
    cin >> t;
    while (t--) {
        int a, b, c;
        cin >> a >> b >> c;
        sort3(a, b, c);
        cout << a << ' ' << b << ' ' << c << endl;
    }
    return 0;
}
```

---

### 2. 求最大值最小值（引用）（20 分）

- 题目 ID: `134`
- 限制: 1s / 128MB

#### 题目描述

编写函数void find(int *num,int n,int &minIndex,int &maxIndex)，求数组num(元素为num[0]，num[1]，...，num[n-1]）中取最小值、最大值的元素下标minIndex,maxIndex（若有相同最值，取第一个出现的下标。）

输入n，动态分配n个整数空间，输入n个整数，调用该函数求数组的最小值、最大值下标。

改变函数find功能不计分。

要求：在main函数中按样例格式输出结果，不能直接在find函数中输出。

#### 输入格式

测试次数

每组测试数据一行：数据个数n，后跟n个整数

#### 输出格式

每组测试数据输出两行，分别是最小值、最大值及其下标。具体格式见样例。多组测试数据之间以空行分隔。

#### 样例输入
```text
2
5 10 20 40 -100 40
10 23 12 -32 4 6 230 100 90 -120 15
```

#### 样例输出
```text
min=-100 minindex=3
max=40 maxindex=2

min=-120 minindex=8
max=230 maxindex=5
```

#### 答案解析

- **建模思路**：动态数组保存输入数据，`find` 函数通过引用参数返回最小值下标和最大值下标。
- **核心算法**：初始化 `minIndex=maxIndex=0`，从下标 1 开始遍历。只在发现严格更小或严格更大时更新，这样相同最值会保留第一次出现的位置。
- **输出易错点**：每组输出两行，多组之间有一个空行。

#### 参考实现提示

函数原型固定为 `void find(int *num,int n,int &minIndex,int &maxIndex)`，不要在函数里直接输出。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
void find(int *num, int n, int &minIndex, int &maxIndex) {
    minIndex = maxIndex = 0;
    for (int i = 1; i < n; i++) {
        if (num[i] < num[minIndex])
            minIndex = i;
        if (num[i] > num[maxIndex])
            maxIndex = i;
    }
}
int main() {
    int t;
    cin >> t;
    for (int k = 0; k < t; k++) {
        int n;
        cin >> n;
        int *a = new int[n];
        for (int i = 0; i < n; i++)
            cin >> a[i];
        int mn, mx;
        find(a, n, mn, mx);
        cout << "min=" << a[mn] << " minindex=" << mn << endl;
        cout << "max=" << a[mx] << " maxindex=" << mx << endl;
        if (k != t - 1)
            cout << endl;
        delete[] a;
    }
    return 0;
}
```

---

### 3. 小票输入输出（结构体）（20 分）

- 题目 ID: `133`
- 限制: 1s / 128MB

#### 题目描述

现在人的消费习惯大多是刷卡消费，商家会通过POS机回执一个小票，包含商家名称、终端号、操作员、发卡方、有效期、卡号、交易时间、消费金额等信息，把商家信息定义为一个Struct结构，按照要求输出相应的格式小票。

#### 输入格式

第一行输入消费次数（刷卡次数）

 第二行依次输入小票包含的各种属性，最大长度不超过30.

第三行与第二行类似，以此类推。。。

#### 输出格式

根据输入信息，依次输出各次刷卡信息$

#### 样例输入
```text
2
TianHong 00001 01 CCB 21/06 6029071012345678 2016/3/13 1000.00
Cindy 00002 02 CCB 21/07 6029071055558888 2015/3/13 50.00
```

#### 样例输出
```text
Name: TianHong
Terminal: 00001 operator: 01
Card Issuers: CCB Validity: 21/06
CardNumber: 6029********5678
Traded: 2016/3/13
Costs: $1000.00

Name: Cindy
Terminal: 00002 operator: 02
Card Issuers: CCB Validity: 21/07
CardNumber: 6029********8888
Traded: 2015/3/13
Costs: $50.00
```

#### 答案解析

- **建模思路**：用结构体保存商家名称、终端号、操作员、发卡方、有效期、卡号、交易时间、金额。
- **核心算法**：卡号脱敏输出前 4 位和后 4 位，中间固定输出 `********`。
- **输出易错点**：每张小票之间空一行；金额前有 `$`，输入金额作为字符串保存能避免小数格式变化。

#### 参考实现提示

卡号最大长度固定时，用 `substr(0,4)` 和 `substr(len-4)` 组合最方便。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
struct Receipt {
    string name, terminal, op, issuer, valid, card, date, cost;
};
int main() {
    int t;
    cin >> t;
    for (int i = 0; i < t; i++) {
        Receipt r;
        cin >> r.name >> r.terminal >> r.op >> r.issuer >> r.valid >> r.card >> r.date >> r.cost;
        cout << "Name: " << r.name << endl;
        cout << "Terminal: " << r.terminal << " operator: " << r.op << endl;
        cout << "Card Issuers: " << r.issuer << " Validity: " << r.valid << endl;
        cout << "CardNumber: " << r.card.substr(0, 4) << "********"
             << r.card.substr(r.card.size() - 4) << endl;
        cout << "Traded: " << r.date << endl;
        cout << "Costs: $" << r.cost << endl;
        if (i != t - 1)
            cout << endl;
    }
    return 0;
}
```

---

### 4. 谁是老二（结构体）（20 分）

- 题目 ID: `20`
- 限制: 1s / 128MB

#### 题目描述

定义一个结构体，包含年月日，表示一个学生的出生日期。然后在一群学生的出生日期中找出谁的出生日期排行第二

要求：出生日期的存储必须使用结构体，不能使用其他类型的数据结构。

要求程序全过程对出生日期的输入、访问、输出都必须使用结构。

#### 输入格式

第一行输入t表示有t个出生日期

每行输入三个整数，分别表示年、月、日

依次输入t个实例

#### 输出格式

输出排行第二老的出生日期，按照年-月-日的格式输出

#### 样例输入
```text
6
1980 5 6
1981 8 3
1980 3 19
1980 5 3
1983 9 12
1981 11 23
```
```text
5
1980 4 1
1981 8 3
1980 3 31
1983 9 12
1981 11 23
```

#### 样例输出
```text
1980-5-3
```
```text
1980-4-1
```

#### 答案解析

- **建模思路**：结构体保存年月日。年龄越大，出生日期越早，因此要找“第二老”就是按日期从早到晚排序后取第 2 个。
- **核心算法**：比较日期时依次比较年、月、日。可以排序整个数组，也可以遍历维护最早和第二早。
- **输出易错点**：输出格式为 `年-月-日`，不补零。

#### 参考实现提示

如果用排序，比较函数返回 `a` 是否早于 `b`；样例中 `1980-5-3` 早于 `1980-5-6`。

#### 完整可运行参考代码

```cpp
#include <algorithm>
#include <iostream>
using namespace std;
struct Date {
    int y, m, d;
};
bool earlier(const Date &a, const Date &b) {
    if (a.y != b.y)
        return a.y < b.y;
    if (a.m != b.m)
        return a.m < b.m;
    return a.d < b.d;
}
int main() {
    int n;
    cin >> n;
    Date *a = new Date[n];
    for (int i = 0; i < n; i++)
        cin >> a[i].y >> a[i].m >> a[i].d;
    sort(a, a + n, earlier);
    cout << a[1].y << '-' << a[1].m << '-' << a[1].d << endl;
    delete[] a;
    return 0;
}
```

---

### 5. 抄袭查找（结构体+指针+函数）（20 分）

- 题目 ID: `21`
- 限制: 1s / 128MB

#### 题目描述

已知一群学生的考试试卷，要求对试卷内容进行对比，查找是否有抄袭。

每张试卷包含：学号（整数类型）、题目1答案（字符串类型）、题目2答案（字符串类型）、题目3答案（字符串类型）

要求：使用结构体来存储试卷的信息。定义一个函数，返回值为一个整数，参数是两个结构体指针，函数操作是比较两张试卷的每道题目的答案，如果相同题号的答案相似度达到或超过90%，那么就认为有抄袭，函数返回抄袭题号，否则返回0。相似度是指在同一题目中，两个答案的逐个位置上的字符两两比较，相同的数量大于等于任一个答案的长度的90%，就认为抄袭。

#### 输入格式

第一行输入t表示有t张试卷

第二行输入第1张试卷的学生学号

第三行输入第1张试卷的题目1答案，最大长度不超过100

第四行输入第1张试卷的题目2答案，最大长度不超过100

第五行输入第1张试卷的题目3答案，最大长度不超过100

每张试卷对应4行输入

依次输入t张试卷的数据

#### 输出格式

在一行中，把发现抄袭的两个学号和题目号输出，只输出第一次发现抄袭的题号，数据之间用单个空格隔开

如果发现是题目1抄袭，题目号为1，以此类推

输出顺序按照输入的学号顺序进行输出

#### 样例输入
```text
5
2088150555
aabcdef11
ZZ887766dd
cc33447799ZZ
2088150333
abcdef00
AABBCCDDEE
ZZ668899cc
2088150111
AABBCCDDEE
ZZ668899cc
abcdef00
2088150222
AABBCFDDeE
ZZ889966dd
abcdef000
2088150444
aabcdef00
AABBCDDDEE
cc668899ZZ
```

#### 样例输出
```text
2088150333 2088150444 2
2088150111 2088150222 3
```

#### 答案解析

- **建模思路**：每张试卷结构体保存学号和 3 个答案字符串。比较函数接收两个结构体指针，返回第一次判定抄袭的题号。
- **核心算法**：对同题号两个答案逐位比较，统计相同字符数。只要相同数量大于等于任一答案长度的 90%，就认为该题抄袭。
- **输出易错点**：要检查所有试卷对，输出发现抄袭的两个学号和题号；同一对只输出第一次发现的题号。

#### 参考实现提示

“任一个答案长度的 90%”可理解为达到较短或较长长度阈值时要按题目测试要求小心处理，常见写法是 `same >= len1*0.9 || same >= len2*0.9`。

#### 完整可运行参考代码

```cpp
#include <iostream>
#include <string>
using namespace std;
struct Paper {
    int id;
    string ans[3];
};
bool similar(const string &a, const string &b) {
    int same = 0, n = min(a.size(), b.size());
    for (int i = 0; i < n; i++)
        if (a[i] == b[i])
            same++;
    return same * 10 >= 9 * (int)a.size() || same * 10 >= 9 * (int)b.size();
}
int check(Paper *a, Paper *b) {
    for (int i = 0; i < 3; i++)
        if (similar(a->ans[i], b->ans[i]))
            return i + 1;
    return 0;
}
int main() {
    int n;
    cin >> n;
    Paper *p = new Paper[n];
    for (int i = 0; i < n; i++) {
        cin >> p[i].id;
        for (int j = 0; j < 3; j++)
            cin >> p[i].ans[j];
    }
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++) {
            int q = check(&p[i], &p[j]);
            if (q)
                cout << p[i].id << ' ' << p[j].id << ' ' << q << endl;
        }
    delete[] p;
    return 0;
}
```

---

## 实验三. 指针2

| 项目 | 内容 |
| --- | --- |
| 测验 ID | `1180` |
| 开放时间 | 2026-03-17T10:15:00+08:00 至 2026-03-24T00:00:00+08:00 |
| 本节题目数 | 5 |

---

### 1. 月份查询（指针数组）（20 分）

- 题目 ID: `13`
- 限制: 1s / 128MB

#### 题目描述

已知每个月份的英文单词如下，要求创建一个指针数组，数组中的每个指针指向一个月份的英文字符串，要求根据输入的月份数字输出相应的英文单词

1月 January 

2月 February

3月 March

4月 April

5月 May

6月 June

7月 July

8月 August

9月 September

10月 October

11月 November

12月 December

#### 输入格式

第一行输入t表示t个测试实例

接着每行输入一个月份的数字

依次输入t行

#### 输出格式

每行输出相应的月份的字符串，若没有这个月份的单词，输出error

#### 样例输入
```text
3
5
11
15
```

#### 样例输出
```text
May
November
error
```

#### 答案解析

- **建模思路**：定义 `char* month[12]` 或 `const char* month[12]` 保存 12 个月英文名。
- **核心算法**：输入月份 `m`，若 `1<=m<=12` 输出 `month[m-1]`，否则输出 `error`。
- **输出易错点**：数组下标从 0 开始，1 月对应下标 0。

#### 参考实现提示

字符串常量建议用 `const char*`，避免修改字面量。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
int main() {
    const char *m[12] = {"January", "February", "March",     "April",   "May",      "June",
                         "July",    "August",   "September", "October", "November", "December"};
    int t;
    cin >> t;
    while (t--) {
        int x;
        cin >> x;
        if (x >= 1 && x <= 12)
            cout << m[x - 1] << endl;
        else
            cout << "error" << endl;
    }
    return 0;
}
```

---

### 2. 字符串比较（指针与字符）（20 分）

- 题目 ID: `14`
- 限制: 1s / 128MB

#### 题目描述

编写一个函数比较两个字符串，参数是两个字符指针（要求显式定义，例如char *S, char *T)，比较字符串S和T的大小。如果S大于T，则返回1，如果S小于T则返回-1，如果S与T相等则返回0。

比较规则：

1.把两个字符串的相同位置上的字符进行比较，字符的大小比较以ASCII值为准

2.在比较中，如果字符串S的字符大于字符串T的字符的数量超过小于的数量，则认为S大于T，如果等于则S等于T，如果小于则S小于T

例如S为aaccdd，T为eebbbb，每个位置比较得到S前两个字母都小于T，但后4个字母都大于T，最终认为S大于T

3.如果两个字符串长度不同，则更长的字符串为大

 

在主函数中输入两个字符串，并调用该函数进行判断，在判断函数中必须使用函数参数的指针进行字符比较

#### 输入格式

输入t表示有t个测试实例

接着每两行输入两个字符串，字符串的最大长度不超过20

依次输入t个实例

#### 输出格式

每行输出一个实例的比较结果

#### 样例输入
```text
3
aaccdd
eebbbb
AAbb++
aaEE*-
zznnkk
aaaaaaa
```
```text
2
aaaaaaa
zznnkk
eebbbb
aaccdd
```

#### 样例输出
```text
1
0
-1
```
```text
1
-1
```

#### 答案解析

- **建模思路**：函数参数必须是字符指针，通过移动指针访问字符，而不是用数组下标完成核心比较。
- **核心算法**：如果长度不同，长字符串更大；长度相同则逐位比较，统计 `S[i]>T[i]` 的次数和 `S[i]<T[i]` 的次数，次数多者更大，相等则返回 0。
- **输出易错点**：这里不是标准字典序比较，不能直接用 `strcmp` 替代。

#### 参考实现提示

可先用指针遍历求长度，再在长度相等时同步移动两个指针统计大小次数。

#### 完整可运行参考代码

```cpp
#include <cstring>
#include <iostream>
using namespace std;
int cmp(char *S, char *T) {
    int ls = strlen(S), lt = strlen(T);
    if (ls > lt)
        return 1;
    if (ls < lt)
        return -1;
    int gt = 0, ltc = 0;
    while (*S && *T) {
        if (*S > *T)
            gt++;
        else if (*S < *T)
            ltc++;
        S++;
        T++;
    }
    if (gt > ltc)
        return 1;
    if (gt < ltc)
        return -1;
    return 0;
}
int main() {
    int t;
    cin >> t;
    while (t--) {
        char a[25], b[25];
        cin >> a >> b;
        cout << cmp(a, b) << endl;
    }
    return 0;
}
```

---

### 3. 密钥加密法（指针应用）（20 分）

- 题目 ID: `18`
- 限制: 1s / 128MB

#### 题目描述

有一种方式是使用密钥进行加密的方法，就是对明文的每个字符使用密钥上对应的密码进行加密，最终得到密文

例如明文是abcde，密钥是234，那么加密方法就是a对应密钥的2，也就是a偏移2位转化为c；明文b对应密钥的3，就是b偏移3位转化为e，同理c偏移4位转化为g。这时候密钥已经使用完，那么又重头开始使用。因此明文的d对应密钥的2，转化为f，明文的e对应密钥的3转化为h。所以明文abcde，密钥234，经过加密后得到密文是cegfh。

如果字母偏移的位数超过26个字母范围，则循环偏移，例如字母z偏移2位，就是转化为b，同理字母x偏移5位就是转化为c

要求：使用三个指针p、q、s分别指向明文、密钥和密文，然后使用指针p和q来访问每个位置的字符，进行加密得到密文存储在指针s指向的位置。

除了变量定义和输入数据，其他过程都不能使用数组下标法，必须使用三个指针来访问明文、密钥和密文。

提示：当指针q已经移动到密钥的末尾，但明文仍然没有结束，那么q就跳回密钥头

#### 输入格式

第一行输入t表示有t个测试实例

第二行输入一个字符串，表示第一个实例的明文, 字符串的最大长度不超过20

第三行输入一个数字串，表示第一个实例的密钥，数字串的最大长度不超过20

依次输入t个实例

#### 输出格式

每行输出加密后的密文

#### 样例输入
```text
2
abcde
234
XenOS
56
```

#### 样例输出
```text
cegfh
CksUX
```

#### 答案解析

- **建模思路**：三个指针分别指向明文、密钥和密文。加密过程中移动明文指针和密文指针，密钥指针到末尾后回到开头。
- **核心算法**：密钥字符 `'2'` 转成整数 `2`，字母在大小写各自范围内循环右移。大写字母按 `A-Z`，小写字母按 `a-z`。
- **输出易错点**：明文大小写要保持；密钥用完要循环使用。

#### 参考实现提示

字符偏移可写成 `(ch-base+shift)%26 + base`，其中 `base` 是 `'A'` 或 `'a'`。

#### 完整可运行参考代码

```cpp
#include <cstring>
#include <iostream>
using namespace std;
int main() {
    int t;
    cin >> t;
    while (t--) {
        char plain[25], key[25], cipher[25];
        cin >> plain >> key;
        char *p = plain, *s = cipher;
        int klen = strlen(key), i = 0;
        while (*p) {
            int sh = key[i % klen] - '0';
            char base = (*p >= 'A' && *p <= 'Z') ? 'A' : 'a';
            *s = char((*p - base + sh) % 26 + base);
            p++;
            s++;
            i++;
        }
        *s = '\0';
        cout << cipher << endl;
    }
    return 0;
}
```

---

### 4. 矩阵左转（指针与数组）（20 分）

- 题目 ID: `11`
- 限制: 1s / 128MB

#### 题目描述

输入一个2*3的矩阵，将这个矩阵向左旋转90度后输出

比如现在有2*3矩阵 ：

1 2 3

4 5 6 

向左旋转90度后的矩阵变为：

3 6

2 5

1 4

要求：除了矩阵创建和数据输入可以使用数组和数组下标的方法，其他过程对矩阵的任何访问都必须使用指针

提示：m行n列的二维矩阵，第i行第j列的元素与首元素的距离为i*n+j，序号从0开始计算

#### 输入格式

第一行输入t表示有t个测试实例

连续两行输入一个2*3的矩阵的数据

依次输入t个实例

#### 输出格式

依次输出左转后的矩阵结果

在输出的每行中，每个数据之间都用空格隔开，最后一个数据后面也带有空格

#### 样例输入
```text
2
1 2 3
4 5 6
4 5 6
7 8 9
```

#### 样例输出
```text
3 6 
2 5 
1 4 
6 9 
5 8 
4 7 
```

#### 答案解析

- **建模思路**：原矩阵固定为 `2*3`，左转 90 度后变成 `3*2`。
- **核心算法**：新矩阵第 `i` 行第 `j` 列对应原矩阵第 `j` 行第 `2-i` 列。也就是依次输出原矩阵第 3 列、第 2 列、第 1 列。
- **输出易错点**：每个数字后面都有一个空格，每行结束换行。

#### 参考实现提示

题目要求除输入外用指针访问，可用 `*(p + row*3 + col)` 访问原矩阵元素。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
int main() {
    int t;
    cin >> t;
    while (t--) {
        int a[2][3];
        for (int i = 0; i < 2; i++)
            for (int j = 0; j < 3; j++)
                cin >> a[i][j];
        int *p = &a[0][0];
        for (int col = 2; col >= 0; col--) {
            for (int row = 0; row < 2; row++)
                cout << *(p + row * 3 + col) << ' ';
            cout << endl;
        }
    }
    return 0;
}
```

---

### 5. 动态矩阵（指针与堆内存分配）（20 分）

- 题目 ID: `16`
- 限制: 1s / 128MB

#### 题目描述

未知一个整数矩阵的大小，在程序运行时才会输入矩阵的行数m和列数n

要求使用指针，结合new方法，动态创建一个二维数组，并求出该矩阵的最小值和最大值，可以使用数组下标法。

不能先创建一个超大矩阵，然后只使用矩阵的一部分空间来进行数据访问、

创建的矩阵大小必须和输入的行数m和列数n一样

#### 输入格式

第一行输入t表示t个测试实例

第二行输入两个数字m和n，表示第一个矩阵的行数和列数

第三行起，连续输入m行，每行n个数字，表示输入第一个矩阵的数值

依次输入t个实例

#### 输出格式

每行输出一个实例的最小值和最大值

#### 提示

建立动态二维数组：

对应释放空间：

#### 样例输入
```text
2
2 3
33 22 11
66 88 55
3 4
19 38 45 14
22 65 87 31
91 35 52 74
```

#### 样例输出
```text
11 88
14 91
```

#### 答案解析

- **建模思路**：行数和列数运行时才知道，所以用 `new` 动态创建二维数组，不能开固定大数组凑合。
- **核心算法**：读入第一个元素时初始化最大值和最小值，之后每读一个元素就更新。也可以先完整读入矩阵再遍历。
- **输出易错点**：每组输出 `最小值 最大值` 一行；处理完一组后要释放所有动态内存。

#### 参考实现提示

释放顺序是先逐行 `delete[] a[i]`，再 `delete[] a`。

#### 完整可运行参考代码

```cpp
#include <iostream>
using namespace std;
int main() {
    int t;
    cin >> t;
    while (t--) {
        int m, n;
        cin >> m >> n;
        int **a = new int *[m];
        int mn = 0, mx = 0;
        for (int i = 0; i < m; i++) {
            a[i] = new int[n];
            for (int j = 0; j < n; j++) {
                cin >> a[i][j];
                if (i == 0 && j == 0)
                    mn = mx = a[i][j];
                else {
                    if (a[i][j] < mn)
                        mn = a[i][j];
                    if (a[i][j] > mx)
                        mx = a[i][j];
                }
            }
        }
        cout << mn << ' ' << mx << endl;
        for (int i = 0; i < m; i++)
            delete[] a[i];
        delete[] a;
    }
    return 0;
}
```
