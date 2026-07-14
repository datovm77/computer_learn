# OJC 实验 12–18：题目与参考答案

> 本文保留每题的核心要求、输入输出、样例和必要的题面代码框架，删除重复提示与无意义空行。实验 12–17 附参考代码；实验 18 在测验截止前仅给出详细思路和伪代码。

## 文档信息

- 生成时间：2026-07-12 15:06:18
- 题目范围：实验 12–18
- 测验数量：7
- 题目数量：35
- 原始数据：`contest_questions_group81_20260712_145830.json`

## 目录

| 实验 | 主题             | 题目数 |
| ---: | ---------------- | -----: |
|   12 | 多重继承         |      5 |
|   13 | 继承与多态       |      5 |
|   14 | 运算符重载       |      5 |
|   15 | 运算符重载应用   |      5 |
|   16 | 函数模板与类模板 |      5 |
|   17 | 期末综合练习     |      5 |
|   18 | 期末模拟         |      5 |

## 实验12. 多重继承

> 测验 ID：`1276` · 时间：2026-05-19T10:10:00+08:00 至 2026-05-26T00:00:00+08:00 · 共 5 题

### 12.1 交通工具（多重继承）

> 题目 ID：`72` · 20 分 · 限制：1s / 128MB

#### 题目要求

1、建立如下的类继承结构：

1)一个车类CVehicle作为基类，具有max_speed、speed、weight等数据成员，display()等成员函数

2)从CVehicle类派生出自行车类CBicycle，添加属性：高度height

3)从CVehicle类派生出汽车类CMotocar，添加属性：座位数seat_num

4)从CBicycle和CMotocar派生出摩托车类CMotocycle

2、分别定义以上类的构造函数、输出函数display及其他函数（如需要）。

3、在主函数中定义各种类的对象，并测试之，通过对象调用display函数产生输出。

#### 输入格式

第一行：最高速度 速度 重量 第二行：高度 第三行：座位数

#### 输出格式

第一行：Vehicle: 第二行及以后各行：格式见Sample

#### 样例输入

```text
100 60 20
28
2
```

#### 样例输出

```text
Vehicle:
max_speed:100
speed:60
weight:20

Bicycle:
max_speed:100
speed:60
weight:20
height:28

Motocar:
max_speed:100
speed:60
weight:20
seat_num:2

Motocycle:
max_speed:100
speed:60
weight:20
height:28
seat_num:2
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

class CVehicle {
protected:
    int max_speed, speed, weight;
public:
    CVehicle(int ms, int s, int w) : max_speed(ms), speed(s), weight(w) {}
    void displayBase() const {
        cout << "max_speed:" << max_speed << endl;
        cout << "speed:" << speed << endl;
        cout << "weight:" << weight << endl;
    }
};

class CBicycle : virtual public CVehicle {
protected:
    int height;
public:
    CBicycle(int ms, int s, int w, int h) : CVehicle(ms, s, w), height(h) {}
    void display() const {
        cout << "Bicycle:" << endl;
        displayBase();
        cout << "height:" << height << endl;
    }
};

class CMotocar : virtual public CVehicle {
protected:
    int seat_num;
public:
    CMotocar(int ms, int s, int w, int seat) : CVehicle(ms, s, w), seat_num(seat) {}
    void display() const {
        cout << "Motocar:" << endl;
        displayBase();
        cout << "seat_num:" << seat_num << endl;
    }
};

class CMotocycle : public CBicycle, public CMotocar {
public:
    CMotocycle(int ms, int s, int w, int h, int seat)
        : CVehicle(ms, s, w), CBicycle(ms, s, w, h), CMotocar(ms, s, w, seat) {}
    void display() const {
        cout << "Motocycle:" << endl;
        displayBase();
        cout << "height:" << height << endl;
        cout << "seat_num:" << seat_num << endl;
    }
};

int main() {
    int maxSpeed, speed, weight, height, seat;
    cin >> maxSpeed >> speed >> weight >> height >> seat;
    CVehicle v(maxSpeed, speed, weight);
    CBicycle b(maxSpeed, speed, weight, height);
    CMotocar c(maxSpeed, speed, weight, seat);
    CMotocycle m(maxSpeed, speed, weight, height, seat);

    cout << "Vehicle:" << endl;
    v.displayBase();
    cout << endl;
    b.display();
    cout << endl;
    c.display();
    cout << endl;
    m.display();
    return 0;
}
```

---

### 12.2 在职研究生（多重继承）

> 题目 ID：`70` · 20 分 · 限制：1s / 128MB

#### 题目要求

1、建立如下的类继承结构：

1)  定义一个人员类CPeople，其属性（保护类型）有：姓名、性别、年龄；

2)  从CPeople类派生出学生类CStudent，添加属性：学号和入学成绩；

3)  从CPeople类再派生出教师类CTeacher，添加属性：职务、部门；

4)  从CStudent和CTeacher类共同派生出在职研究生类CGradOnWork，添加属性：研究方向、导师；

2、分别定义以上类的构造函数、输出函数print及其他函数（如需要）。

3、在主函数中定义各种类的对象，并测试之。

#### 输入格式

第一行：姓名性别年龄，姓名的最大字符长度为20

第二行：学号成绩

第三行：职务部门，职务和部门的最大字符长度均为20

第四行：研究方向导师，研究方向和导师的最大长度均为20

#### 输出格式

第一行：People:

第二行及以后各行：格式见Sample

#### 样例输入

```text
wang-li m 23
2012100365 92.5
assistant computer
robot zhao-jun
```

#### 样例输出

```text
People:
Name: wang-li
Sex: m
Age: 23

Student:
Name: wang-li
Sex: m
Age: 23
No.: 2012100365
Score: 92.5

Teacher:
Name: wang-li
Sex: m
Age: 23
Position: assistant
Department: computer

GradOnWork:
Name: wang-li
Sex: m
Age: 23
No.: 2012100365
Score: 92.5
Position: assistant
Department: computer
Direction: robot
Tutor: zhao-jun
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
#include <string>
using namespace std;

class CPeople {
protected:
    string name, sex;
    int age;
public:
    CPeople(string n, string s, int a) : name(n), sex(s), age(a) {}
    void printPeopleFields() const {
        cout << "Name: " << name << endl;
        cout << "Sex: " << sex << endl;
        cout << "Age: " << age << endl;
    }
};

class CStudent : virtual public CPeople {
protected:
    string no;
    double score;
public:
    CStudent(string n, string s, int a, string no1, double sc)
        : CPeople(n, s, a), no(no1), score(sc) {}
    void printStudentFields() const {
        printPeopleFields();
        cout << "No.: " << no << endl;
        cout << "Score: " << score << endl;
    }
};

class CTeacher : virtual public CPeople {
protected:
    string position, department;
public:
    CTeacher(string n, string s, int a, string p, string d)
        : CPeople(n, s, a), position(p), department(d) {}
    void printTeacherFields() const {
        printPeopleFields();
        cout << "Position: " << position << endl;
        cout << "Department: " << department << endl;
    }
};

class CGradOnWork : public CStudent, public CTeacher {
private:
    string direction, tutor;
public:
    CGradOnWork(string n, string s, int a, string no1, double sc, string p, string d, string dir, string tu)
        : CPeople(n, s, a), CStudent(n, s, a, no1, sc), CTeacher(n, s, a, p, d),
          direction(dir), tutor(tu) {}
    void printAll() const {
        printStudentFields();
        cout << "Position: " << position << endl;
        cout << "Department: " << department << endl;
        cout << "Direction: " << direction << endl;
        cout << "Tutor: " << tutor << endl;
    }
};

int main() {
    string name, sex, no, position, department, direction, tutor;
    int age;
    double score;
    cin >> name >> sex >> age >> no >> score >> position >> department >> direction >> tutor;

    CPeople p(name, sex, age);
    CStudent s(name, sex, age, no, score);
    CTeacher t(name, sex, age, position, department);
    CGradOnWork g(name, sex, age, no, score, position, department, direction, tutor);

    cout << "People:" << endl; p.printPeopleFields(); cout << endl;
    cout << "Student:" << endl; s.printStudentFields(); cout << endl;
    cout << "Teacher:" << endl; t.printTeacherFields(); cout << endl;
    cout << "GradOnWork:" << endl; g.printAll();
    return 0;
}
```

---

### 12.3 商旅信用卡（多重继承）

> 题目 ID：`76` · 20 分 · 限制：1s / 128MB

#### 题目要求

某旅游网站（假设旅程网）和某银行推出旅游综合服务联名卡—旅程信用卡，兼具旅程会员卡和银行信用卡功能。

旅程会员卡，有会员卡号（int）、旅程积分（int），通过会员卡下订单，按订单金额累计旅程积分。

信用卡，有卡号（int）、姓名（string)、额度（int)、账单金额（float)、信用卡积分（int）。------请注意数据类型

信用卡消费金额m，若超额度，则不做操作；否则，账单金额+m，额度-m，信用卡积分按消费金额累加。

信用卡退款m，账单金额-m，信用卡积分减去退款金额。

通过旅程信用卡在旅程网下单，旅程积分和信用卡积分双重积分（即旅程积分和信用卡积分同时增加）。

旅程信用卡可以按      旅程积分：信用卡积分= 1：2    的比例将信用卡积分兑换为旅程积分。

初始假设信用卡积分、旅程积分、账单金额为0。

根据上述内容，定义旅程会员卡类、信用卡类，从两者派生出旅程信用卡类并定义三个类的构造函数和其它所需函数。

生成旅程信用卡对象，输入卡信息，调用对象成员函数完成旅程网下单、信用卡刷卡、信用卡退款、信用卡积分兑换为旅程积分等操作。

#### 输入格式

第一行：输入旅程会员卡号 信用卡号 姓名 额度

第二行：测试次数n

第三行到第n+2行，每行：操作 金额或积分

o   m（旅程网下订单，订单金额m）

c   m（信用卡消费，消费金额m）

q   m (信用卡退款，退款金额m）

t    m（积分兑换，m信用卡积分兑换为旅程积分）

#### 输出格式

输出所有操作后旅程信用卡的信息：

旅程号   旅程积分

信用卡号  姓名   账单金额   信用卡积分

#### 样例输入

```text
1000 2002  lili  3000
4
o 212.5
c 300
q 117.4
t 200
```

#### 样例输出

```text
1000 312
2002 lili 395.1 195
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
#include <string>
using namespace std;

class TravelCard {
protected:
    int travelNo, travelPoint;
public:
    TravelCard(int no) : travelNo(no), travelPoint(0) {}
};

class CreditCard {
protected:
    int creditNo, limit, creditPoint;
    string name;
    double bill;
public:
    CreditCard(int no, string n, int lim) : creditNo(no), limit(lim), creditPoint(0), name(n), bill(0) {}
    bool consume(double m) {
        if (m > limit) return false;
        bill += m;
        limit -= (int)m;
        creditPoint += (int)m;
        return true;
    }
    void refund(double m) {
        bill -= m;
        creditPoint -= (int)m;
    }
};

class TravelCreditCard : public TravelCard, public CreditCard {
public:
    TravelCreditCard(int tNo, int cNo, string n, int lim) : TravelCard(tNo), CreditCard(cNo, n, lim) {}
    void order(double m) {
        if (consume(m)) travelPoint += (int)m; // 下单同时得到两类积分
    }
    void exchange(int p) {
        if (p <= creditPoint) {
            creditPoint -= p;
            travelPoint += p / 2;
        }
    }
    void print() const {
        cout << travelNo << ' ' << travelPoint << endl;
        cout << creditNo << ' ' << name << ' ' << bill << ' ' << creditPoint << endl;
    }
};

int main() {
    int tNo, cNo, limit, n;
    string name;
    cin >> tNo >> cNo >> name >> limit >> n;
    TravelCreditCard card(tNo, cNo, name, limit);
    while (n--) {
        char op;
        double m;
        cin >> op >> m;
        if (op == 'o') card.order(m);
        else if (op == 'c') card.consume(m);
        else if (op == 'q') card.refund(m);
        else if (op == 't') card.exchange((int)m);
    }
    card.print();
    return 0;
}
```

---

### 12.4 拯救小明（多继承+友元）

> 题目 ID：`301` · 20 分 · 限制：1s / 128MB

#### 题目要求

小明同学有着严重的拖延症，每次老师布置的作业都要到快要截止的时候才会开始动手完成，因此现在有着许许多多的作业完成。你是小明的好朋友，请帮小明找出最紧急的作业（即最早截止的作业）。

要求如下：

1.定义一个日期类Date，包括三个protected成员数据year, month, day;

2.定义一个时间类Time，包括三个protected成员数据hour, minute, second（24小时制）；

3.以Date类和Time类为基类，创建一个作业类Work，包括新增成员：int id;  // 作业的id

4.定义一个友元函数bool before(const Work& w1,const Work& w2);  // 判断作业w1的时间是否早于作业w2的时间。

#### 输入格式

输入若干作业，每个作业占一行（作业id 年 月 日 时 分 秒）

当输入0时结束，相应的结果不要输出。

#### 输出格式

时间最靠前的作业。

#### 样例输入

```text
1 2021 9 25 4 5 6
2 2020 6 13 5 7 8
3 2021 8 21 16 7 9
5 2022 7 8 9 10 11
4 2021 7 26 15 25 30
0
```

#### 样例输出

```text
The urgent Work is No.2: 2020/06/13 05:07:08
```

#### 完整参考代码（带注释）

```cpp
#include <iomanip>
#include <iostream>
using namespace std;

class Date {
protected:
    int year, month, day;
public:
    Date(int y = 0, int m = 0, int d = 0) : year(y), month(m), day(d) {}
};

class Time {
protected:
    int hour, minute, second;
public:
    Time(int h = 0, int m = 0, int s = 0) : hour(h), minute(m), second(s) {}
};

class Work : public Date, public Time {
private:
    int id;
public:
    Work(int i = 0, int y = 0, int mo = 0, int d = 0, int h = 0, int mi = 0, int s = 0)
        : Date(y, mo, d), Time(h, mi, s), id(i) {}
    friend bool before(const Work &a, const Work &b);
    void print() const {
        cout << "The urgent Work is No." << id << ": "
             << year << "/" << setw(2) << setfill('0') << month << "/"
             << setw(2) << day << " " << setw(2) << hour << ":"
             << setw(2) << minute << ":" << setw(2) << second << endl;
    }
};

bool before(const Work &a, const Work &b) {
    if (a.year != b.year) return a.year < b.year;
    if (a.month != b.month) return a.month < b.month;
    if (a.day != b.day) return a.day < b.day;
    if (a.hour != b.hour) return a.hour < b.hour;
    if (a.minute != b.minute) return a.minute < b.minute;
    return a.second < b.second;
}

int main() {
    int id, y, m, d, h, mi, s;
    cin >> id;
    if (id == 0) return 0;
    cin >> y >> m >> d >> h >> mi >> s;
    Work urgent(id, y, m, d, h, mi, s);
    while (cin >> id && id != 0) {
        cin >> y >> m >> d >> h >> mi >> s;
        Work cur(id, y, m, d, h, mi, s);
        if (before(cur, urgent)) urgent = cur;
    }
    urgent.print();
    return 0;
}
```

---

### 12.5 计算宝宝帐户收益(多重继承)

> 题目 ID：`77` · 20 分 · 限制：1s / 128MB

#### 题目要求

定义一个类 CPeople，具有身份号码 (id, char[20]) 和姓名 (name, char[10]) 两个数据成员，从 CPeople 类中再派生出 CInternetUser 类和 CBankCustomer 类，然后再从 CInternetUser 和 CBankCustomer 多重继承派生出 CInternetBankCustomer 类。

CInternetUser 类有登录密码 (password, char[20]) 属性和注册 registerUser（设置 id、name 和 password），登录 login（判断输入的 id 与 password 是否与对象注册的相同）成员函数。

CBankCustomer 类有余额 (balance, double) 属性和开户 openAccount（设置客户姓名和 id），存款 deposit，取款 withdraw 以及缺省的构造函数。

CInternetBankCustomer 类包括有余额、前一日余额、当日收益、今日万元收益和上一日万元收益等 5 个数据成员，成员函数有缺省构造函数，存款和取款，设置万元收益，计算当日收益，登陆 login（判断输入的 id 和密码是否与互联网用户的相同，同时从 CBankCustomer 继承的用户姓名和 id 要与从 CInternetUser 继承的相同）。CInternetBankCustomer 类对象当日存款不计算收益，第 2 天开始才能计算收益，当日取款部分无收益。

可参照如下所示的主函数来测试并设计输入数据：

void main()
{
int t, no_of_days, i;
char i_xm[20], i_id[20], i_mm[20], b_xm[20], b_id[20], ib_id[20], ib_mm[20];
double money, interest;
char op_code;

//输入测试案例数t
cin >> t;
while (t--)
{
//输入互联网用户注册时的用户名,id,登陆密码
cin >> i_xm >> i_id >> i_mm;

//输入银行开户用户名,id
cin >> b_xm >> b_id;

//输入互联网用户登陆时的id,登陆密码
cin >> ib_id >> ib_mm;

CInternetBankCustomer ib_user;

ib_user.registerUser(i_xm, i_id, i_mm);
ib_user.openAccount(b_xm, b_id);

if (ib_user.login(ib_id, ib_mm) == 0)  //互联网用户登陆,若id与密码不符;以及银行开户姓名和id与互联网开户姓名和id不同
{
cout << "Password or ID incorrect" << endl;
continue;
}

//输入天数
cin >> no_of_days;
for (i=0; i < no_of_days; i++)
{
//输入操作代码, 金额, 当日万元收益
cin >> op_code >> money >> interest;
switch (op_code)
{
case 'S':  //从银行向互联网金融帐户存入
case 's':
if (ib_user.deposit(money) == 0)
{
cout << "Bank balance not enough" << endl;
continue;
}
break;
case 'T':  //从互联网金融转入银行帐户
case 't':
if (ib_user.withdraw(money) == 0)
{
cout << "Internet bank balance not enough" << endl;
continue;
}
break;
case 'D':  //直接向银行帐户存款
case 'd':
ib_user.CBankCustomer::deposit(money);
break;
case 'W':  //直接从银行帐户取款
case 'w':
if (ib_user.CBankCustomer::withdraw(money) == 0)
{
cout << "Bank balance not enough" << endl;
continue;
}
break;
default:
cout << "Illegal input" << endl;
continue;
}
ib_user.setInterest(interest);
ib_user.calculateProfit();
//输出用户名,id
//输出银行余额
//输出互联网金融账户余额
ib_user.print();
}
}
}

#### 输入格式

输入用户例数

输入第1个互联网用户注册时的用户名,id,登陆密码

输入第1个用户银行开户用户名,id

输入第1个互联网用户登陆时的id,登陆密码

输入第1个用户操作天数

循环输入操作代码(S,T,D,W)  金额  当日万元收益
......

#### 输出格式

输出第1个用户名,id
输出第1个用户银行余额
输出第1个互联网金融账户余额
......

#### 样例输入

```text
2
zhangsan 1234567890 222222
zhangsan 1234567890
1234567890 222222
4
D 15000 0
s 8000 1.5
T 3000 1.55
w 2000 0
lisi 2014150000 abcdef
lisi 2014150000
2014150000 123456
```

#### 样例输出

```text
Name: zhangsan ID: 1234567890
Bank balance: 15000
Internet bank balance: 0

Name: zhangsan ID: 1234567890
Bank balance: 7000
Internet bank balance: 8000

Name: zhangsan ID: 1234567890
Bank balance: 10000
Internet bank balance: 5001.2

Name: zhangsan ID: 1234567890
Bank balance: 8000
Internet bank balance: 5001.98

Password or ID incorrect
```

#### 关键代码（带注释）

建议把下面成员函数嵌入题目要求的类结构中；它覆盖登录、转账和收益计算的核心。

```cpp
// CInternetBankCustomer 内部建议维护：
// bankBalance: 银行余额
// internetBalance: 互联网金融余额
// lastBalance: 前一日可计息余额
// currentInterest: 今日万元收益

bool login(const string &inputId, const string &inputPwd) {
    // 互联网注册 id/密码正确，并且银行开户姓名/id 与互联网注册姓名/id 相同
    return inputId == internetId && inputPwd == password
        && bankId == internetId && bankName == internetName;
}

bool deposit(double money) { // S: 银行转入互联网账户
    if (bankBalance < money) return false;
    bankBalance -= money;
    internetBalance += money;
    // money 是当日转入，不进入 todayBase
    return true;
}

bool withdraw(double money) { // T: 互联网账户转回银行
    if (internetBalance < money) return false;
    internetBalance -= money;
    bankBalance += money;
    // 当日取出部分无收益，因此 lastBalance 也要相应扣除，但不能小于 0
    if (lastBalance >= money) lastBalance -= money;
    else lastBalance = 0;
    return true;
}

void calculateProfit() {
    double profit = lastBalance * currentInterest / 10000.0;
    internetBalance += profit;
    // 今天结束后，当前余额成为明天可计息余额
    lastBalance = internetBalance;
}
```

---

## 实验13. 继承与多态

> 测验 ID：`1287` · 时间：2026-05-26T10:09:00+08:00 至 2026-06-02T00:00:00+08:00 · 共 5 题

### 13.1 图形面积（虚函数与多态）

> 题目 ID：`64` · 20 分 · 限制：1s / 128MB

#### 题目要求

编写一个程序，定义抽象基类Shape，在Shape类中定义虚函数area()；由它派生出3个派生类：Circle(圆形)、Square(正方形)、Rectangle(矩形)。用虚函数分别计算几种图形面积。

1、要求输出结果保留两位小数。

2、要求用基类指针数组，使它每一个元素指向每一个派生类对象。

#### 输入格式

测试数据的组数 t

第一组测试数据中圆的半径

第一组测测试数据中正方形的边长

第一组测试数据中矩形的长、宽

.......

第 t 组测试数据中圆的半径

第 t 组测测试数据中正方形的边长

第 t 组测试数据中矩形的长、宽

#### 输出格式

第一组数据中圆的面积

第一组数据中正方形的面积

第一组数据中矩形的面积

......

第 t 组数据中圆的面积

第 t 组数据中正方形的面积

第 t 组数据中矩形的面积

#### 样例输入

```text
2
1.2
2.3
1.2 2.3
2.1
3.2
1.23 2.12
```

#### 样例输出

```text
4.52
5.29
2.76
13.85
10.24
2.61
```

#### 完整参考代码（带注释）

```cpp
#include <iomanip>
#include <iostream>
using namespace std;

class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
    double r;
public:
    Circle(double r1) : r(r1) {}
    double area() const override { return 3.14 * r * r; }
};

class Square : public Shape {
    double a;
public:
    Square(double a1) : a(a1) {}
    double area() const override { return a * a; }
};

class Rectangle : public Shape {
    double a, b;
public:
    Rectangle(double x, double y) : a(x), b(y) {}
    double area() const override { return a * b; }
};

int main() {
    int t;
    cin >> t;
    cout << fixed << setprecision(2);
    while (t--) {
        double r, a, x, y;
        cin >> r >> a >> x >> y;
        Circle c(r);
        Square s(a);
        Rectangle rec(x, y);
        Shape *p[3] = {&c, &s, &rec};
        for (int i = 0; i < 3; ++i) cout << p[i]->area() << endl;
    }
    return 0;
}
```

---

### 13.2 汽车收费（虚函数和多态）

> 题目 ID：`180` · 20 分 · 限制：1s / 128MB

#### 题目要求

现在要开发一个系统，实现对多种汽车的收费工作。 汽车基类框架如下所示：

```cpp
class Vehicle
{ 
protected:
     string no; //编号
public:
    virtual void display()=0; //应收费用
}
```

以Vehicle为基类，构建出Car、Truck和Bus三个类。

Car的收费公式为： 载客数*8+重量*2

Truck的收费公式为：重量*5

Bus的收费公式为： 载客数*30

生成上述类并编写主函数，要求主函数中有一个基类指针Vehicle *pv;用来做测试用。

主函数根据输入的信息，相应建立Car,Truck或Bus类对象，对于Car给出载客数和重量，Truck给出重量，Bus给出载客数。假设载客数和重量均为整数。

#### 输入格式

第一行表示测试次数。从第二行开始，每个测试用例占一行，每行数据意义如下：汽车类型（1为car，2为Truck，3为Bus）、编号、基本信息（Car是载客数和重量，Truck给出重量，Bus给出载客数）。

#### 输出格式

车的编号、应缴费用

#### 样例输入

```text
4
1 002 20 5
3 009 30
2 003 50
1 010 17 6
```

#### 样例输出

```text
002 170
009 900
003 250
010 148
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
#include <string>
using namespace std;

class Vehicle {
protected:
    string no;
public:
    Vehicle(string n) : no(n) {}
    virtual void display() = 0;
    virtual ~Vehicle() {}
};

class Car : public Vehicle {
    int passenger, weight;
public:
    Car(string n, int p, int w) : Vehicle(n), passenger(p), weight(w) {}
    void display() override { cout << no << ' ' << passenger * 8 + weight * 2 << endl; }
};

class Truck : public Vehicle {
    int weight;
public:
    Truck(string n, int w) : Vehicle(n), weight(w) {}
    void display() override { cout << no << ' ' << weight * 5 << endl; }
};

class Bus : public Vehicle {
    int passenger;
public:
    Bus(string n, int p) : Vehicle(n), passenger(p) {}
    void display() override { cout << no << ' ' << passenger * 30 << endl; }
};

int main() {
    int t;
    cin >> t;
    while (t--) {
        int type, a, b;
        string no;
        cin >> type >> no;
        Vehicle *pv = nullptr;
        if (type == 1) { cin >> a >> b; pv = new Car(no, a, b); }
        else if (type == 2) { cin >> a; pv = new Truck(no, a); }
        else { cin >> a; pv = new Bus(no, a); }
        pv->display();
        delete pv;
    }
    return 0;
}
```

---

### 13.3 动物园（虚函数与多态）

> 题目 ID：`66` · 20 分 · 限制：1s / 128MB

#### 题目要求

某个动物园内，有老虎、狗、鸭子和猪等动物，动物园的管理员为每个动物都起了一个名字，并且每个动物都有年龄、体重等信息。每到喂食的时候，不同的动物都会叫唤(speak)。每种动物的叫唤声均不同，老虎的叫唤声是“AOOO”，狗的叫唤声是“WangWang”，鸭子的叫唤声是“GAGA”，猪的叫唤声是“HENGHENG”。

定义一个Animal的基类，Animal类有函数Speak()，并派生老虎、狗、鸭子和猪类，其能发出不同的叫唤声（用文本信息输出叫声）。

编写程序，输入动物名称、名字、年龄，让动物园内的各种动物叫唤。

要求：只使用一个基类指针，指向生成的对象并调用方法。

#### 输入格式

测试案例的个数

第一种动物的名称  名字  年龄

第二种动物的名称  名字 年龄

......

#### 输出格式

输出相应动物的信息

如果不存在该种动物，输出There is no 动物名称 in our Zoo.  ，具体输出参考样例输出

#### 样例输入

```text
4
Tiger Jumpjump 10
Pig Piglet 4
Rabbit labi 3
Duck tanglaoya 8
```

#### 样例输出

```text
Hello,I am Jumpjump,AOOO.
Hello,I am Piglet,HENGHENG.
There is no Rabbit in our Zoo.
Hello,I am tanglaoya,GAGA.
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
#include <string>
using namespace std;

class Animal {
protected:
    string name;
    int age;
public:
    Animal(string n, int a) : name(n), age(a) {}
    virtual string speak() const = 0;
    void print() const { cout << "Hello,I am " << name << "," << speak() << "." << endl; }
    virtual ~Animal() {}
};

class Tiger : public Animal { public: Tiger(string n, int a): Animal(n,a) {} string speak() const override { return "AOOO"; } };
class Dog   : public Animal { public: Dog(string n, int a): Animal(n,a) {} string speak() const override { return "WangWang"; } };
class Duck  : public Animal { public: Duck(string n, int a): Animal(n,a) {} string speak() const override { return "GAGA"; } };
class Pig   : public Animal { public: Pig(string n, int a): Animal(n,a) {} string speak() const override { return "HENGHENG"; } };

int main() {
    int t;
    cin >> t;
    while (t--) {
        string type, name;
        int age;
        cin >> type >> name >> age;
        Animal *p = nullptr;
        if (type == "Tiger") p = new Tiger(name, age);
        else if (type == "Dog") p = new Dog(name, age);
        else if (type == "Duck") p = new Duck(name, age);
        else if (type == "Pig") p = new Pig(name, age);

        if (p) { p->print(); delete p; }
        else cout << "There is no " << type << " in our Zoo." << endl;
    }
    return 0;
}
```

---

### 13.4 支票账户（虚函数与多态）

> 题目 ID：`65` · 20 分 · 限制：1s / 128MB

#### 题目要求

某银行的支票账户分为两类，一类为基本支票账户BaseAccount，另一类为具有透支保护特性的BasePlus支票账户。

BaseAccount支票账户的信息包括：客户姓名(name)、账户(account)、当前结余(balance)；BaseAccount支票账户可以执行的操作包括：存款(deposit)、取款(withdraw)、显示账户信息(display)。注意：取款金额不能透支，否则显示出错信息“insufficient”。

BasePlus支票账户除包含BaseAccount的所有信息外，还包括以下信息：透支上限(默认为5000)，当前可透支额度(limitSum)；BasePlus支票账户可执行的操作与BaseAccount相同，但有两种操作的实现不同：(1)对于取款操作，可以在透支上限范围内透支，超过则显示出错信息“insufficient”；(2)对于显示操作，必须显示BasePlus的其他信息。

请实现BaseAccount类和BasePlus类，其中BasePlus类继承于BaseAccount类，注意BaseAccount账户名称以BA开头，BasePlus账户名称以BP开头。

要求只使用一个基类指针，指向所建立的对象，然后使用指针调用类中的方法。

#### 输入格式

测试案例组数 t

第一组测试数据：

第一行输入账户信息：姓名 帐号 当前余额

第二行输入四个整数，表示对账户按顺序存款、取款、存款、取款

第二组测试数据：

.........

#### 输出格式

输出BaseAccount的信息

输出BasePlus的信息

#### 样例输入

```text
4
Tom BA008 1000
1000 2000 1000 1200
Bob BP009 1000
1000 2000 1000 7000
May BA001 2000
500 1000 500 1000
Lily BP002 1000
500 2000 500 3000
```

#### 样例输出

```text
insufficient
Tom BA008 Balance:1000
insufficient
Bob BP009 Balance:1000 limit:5000
May BA001 Balance:1000
Lily BP002 Balance:0 limit:2000
```

#### 关键代码（带注释）

下面是账户类核心实现；主函数按账号前缀 `BA/BP` 创建对象并依次执行存、取、存、取即可。

```cpp
class BaseAccount {
protected:
    string name, account;
    int balance;
public:
    BaseAccount(string n, string a, int b) : name(n), account(a), balance(b) {}
    virtual void deposit(int m) { balance += m; }
    virtual void withdraw(int m) {
        if (balance < m) cout << "insufficient" << endl;
        else balance -= m;
    }
    virtual void display() {
        cout << name << ' ' << account << " Balance:" << balance << endl;
    }
    virtual ~BaseAccount() {}
};

class BasePlus : public BaseAccount {
    int limitSum;
public:
    BasePlus(string n, string a, int b) : BaseAccount(n, a, b), limitSum(5000) {}
    void withdraw(int m) override {
        if (balance + limitSum < m) {
            cout << "insufficient" << endl;
            return;
        }
        if (balance >= m) balance -= m;
        else {
            limitSum -= (m - balance); // 不足部分使用透支额度
            balance = 0;
        }
    }
    void display() override {
        cout << name << ' ' << account << " Balance:" << balance << " limit:" << limitSum << endl;
    }
};
```

---

### 13.5 进位与借位（虚函数和多态）

> 题目 ID：`295` · 20 分 · 限制：1s / 128MB

#### 题目要求

某小学二年级的数学老师在教学生整数加减法运算时发现：班上的同学可以分成三类，第一类可以正确地完成加减法运算(GroupA)；第二类可以正确地完成加法运算，但对于减法运算来说，总是忘记借位的处理(GroupB)；第三类总是忘记加法的进位，也总是忘记减法的借位(GroupC)。（提示：小学二年级还没学负数。）

现在请模拟当老师在课堂提问某位同学时，同学会给出的回答。

实现时请基于下面的基类框架：

```cpp
class Group
{
public:
	virtual int add(int x, int y) = 0; // 输出加法的运算结果
	virtual int sub(int x, int y) = 0; // 输出减法的运算结果
}
```

构建出GroupA, GroupB和GroupC三个派生类:

并编写主函数，要求主函数中有一个基类Group指针，通过该指针统一地进行add和sub运算。

#### 输入格式

第一行表示测试次数。从第二行开始，每个测试用例占一行，每行数据意义如下：学生类别（1为第一类学生，2为第二类学生，3为第三类学生）、第一个数、第二个数。

#### 输出格式

运算后的结果

#### 样例输入

```text
3
1 79+81
2 81-79
3 183+69
```

#### 样例输出

```text
160
12
142
```

#### 关键代码（带注释）

关键是模拟“忘记进位/借位”的竖式计算。

```cpp
int addNoCarry(int a, int b) {
    int base = 1, ans = 0;
    while (a > 0 || b > 0) {
        int d = (a % 10 + b % 10) % 10; // 忘记进位，只保留个位
        ans += d * base;
        a /= 10; b /= 10; base *= 10;
    }
    return ans;
}

int subNoBorrow(int a, int b) {
    int base = 1, ans = 0;
    while (a > 0 || b > 0) {
        int x = a % 10, y = b % 10;
        int d = x >= y ? x - y : x + 10 - y; // 不向高位借 1
        ans += d * base;
        a /= 10; b /= 10; base *= 10;
    }
    return ans;
}

// GroupA: + 用 a+b，- 用 a-b
// GroupB: + 用 a+b，- 用 subNoBorrow(a,b)
// GroupC: + 用 addNoCarry(a,b)，- 用 subNoBorrow(a,b)
```

---

## 实验14. 运算符重载

> 测验 ID：`1296` · 时间：2026-06-02T10:15:00+08:00 至 2026-06-09T00:00:00+08:00 · 共 5 题

### 14.1 分数的加减乘除（运算符重载）

> 题目 ID：`79` · 20 分 · 限制：1s / 128MB

#### 题目要求

Fraction类的基本形式如下：

// 定义Fraction类

class Fraction

{

private:

int fz, fm;

public:

Fraction(int = 0, int = 1);

Fraction(const Fraction&);

Fraction operator+(Fraction);

Fraction operator-(Fraction);

Fraction operator*(Fraction);

Fraction operator/(Fraction);

void set(int = 0, int = 1);

void display();

};

要求如下：

编写main函数，初始化两个Fraction对象的，计算它们之间的加减乘除，计算结果不用化简。

#### 输入格式

第1行：依次输入第1个和第2个Fraction对象的分子和分母值。

#### 输出格式

每行依次分别输出加减乘除计算后的Fraction对象（直接输出分数值，不需要约简）。

#### 样例输入

```text
1 3 2 5
```

#### 样例输出

```text
fraction=11/15
fraction=-1/15
fraction=2/15
fraction=5/6
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

class Fraction {
private:
    int fz, fm;
public:
    Fraction(int a = 0, int b = 1) : fz(a), fm(b) {}
    Fraction(const Fraction &other) : fz(other.fz), fm(other.fm) {}

    Fraction operator+(Fraction x) { return Fraction(fz * x.fm + x.fz * fm, fm * x.fm); }
    Fraction operator-(Fraction x) { return Fraction(fz * x.fm - x.fz * fm, fm * x.fm); }
    Fraction operator*(Fraction x) { return Fraction(fz * x.fz, fm * x.fm); }
    Fraction operator/(Fraction x) { return Fraction(fz * x.fm, fm * x.fz); }

    void set(int a = 0, int b = 1) {
        fz = a;
        fm = b;
    }

    void display() {
        cout << "fraction=" << fz << "/" << fm << endl;
    }
};

int main() {
    int a, b, c, d;
    cin >> a >> b >> c >> d;
    Fraction f1(a, b), f2(c, d);
    (f1 + f2).display();
    (f1 - f2).display();
    (f1 * f2).display();
    (f1 / f2).display();
    return 0;
}
```

---

### 14.2 复数的加减乘运算（运算符重载）

> 题目 ID：`80` · 20 分 · 限制：1s / 128MB

#### 题目要求

定义一个复数类，通过重载运算符：+、-、*，实现两个复数之间的各种运算。

```cpp
class Complex
{
private:
    float real, image;
public:
    Complex(float x = 0, float y = 0);
    friend Complex operator+(Complex&, Complex&);
    friend Complex operator-(Complex&, Complex&);
    friend Complex operator*(Complex&, Complex&);
    void show();
};
```

要求如下：

1.实现Complex类；

2.编写main函数，初始化两个Complex对象，计算它们之间的加减乘，并输出结果。

复数相乘的运算规则

设z1=a+bi，z2=c+di(a、b、c、d∈R)是任意两个复数，那么它们的积(a+bi)(c+di)=(ac-bd)+(bc+ad)i.

#### 输入格式

第1行：输入两个数值，分别为第一个Complex对象的实部和虚部。

第2行：输入两个数值，分别为第二个Complex对象的实部和虚部。

#### 输出格式

第1行：两个Complex对象相加后的输出结果。

第2行：两个Complex对象相减后的输出结果。

第3行：两个Complex对象相乘后的输出结果。

#### 样例输入

```text
10 20
50 40
```

#### 样例输出

```text
Real=60 Image=60
Real=-40 Image=-20
Real=-300 Image=1400
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

class Complex {
private:
    float real, image;
public:
    Complex(float x = 0, float y = 0) : real(x), image(y) {}

    friend Complex operator+(Complex &a, Complex &b) {
        return Complex(a.real + b.real, a.image + b.image);
    }
    friend Complex operator-(Complex &a, Complex &b) {
        return Complex(a.real - b.real, a.image - b.image);
    }
    friend Complex operator*(Complex &a, Complex &b) {
        return Complex(a.real * b.real - a.image * b.image,
                       a.image * b.real + a.real * b.image);
    }

    void show() {
        cout << "Real=" << real << " Image=" << image << endl;
    }
};

int main() {
    float a, b, c, d;
    cin >> a >> b >> c >> d;
    Complex x(a, b), y(c, d);
    (x + y).show();
    (x - y).show();
    (x * y).show();
    return 0;
}
```

---

### 14.3 向量的加减（运算符重载）

> 题目 ID：`94` · 20 分 · 限制：1s / 128MB

#### 题目要求

设向量X=(x1,x2,…,xn)和Y=(y1,y2…,yn),它们之间的加、减分别定义为：

X+Y=(x1+y1,x2+y2,…,xn+yn)

X-Y=(x1-y1,x2-y2,…,xn-yn)

编程序定义向量类Vector ,重载运算符“+”、“-”,实现向量之间的加、减运算;并编写print函数作为向量的输出操作。

Vector类的基本形式如下：

```cpp
class Vector
{
private:
    int vec[5];
public:
    Vector(int v[]);
    Vector();
    Vector(const Vector& obj);
    Vector operator +(const Vector& obj);
    Vector operator -(const Vector& obj);
    void print();
};
```

要求如下：

1.实现Vector类；

2.编写main函数，初始化两个Vector对象的，计算它们之间的加减，并输出结果。

#### 输入格式

第1行：输入5个int类型的值，初始化第一个Vector对象。

第2行: 输入5个int类型的值，初始化第二个Vector对象。

#### 输出格式

第1行：2个Vector对象相加后的输出结果。

第2行：2个Vector对象相减后的输出结果。

#### 样例输入

```text
-4 1 0 10 5 
-11 8 10 17 -6 
```

#### 样例输出

```text
-15 9 10 27 -1 
7 -7 -10 -7 11 
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

class Vector {
private:
    int vec[5];
public:
    Vector() {
        for (int i = 0; i < 5; ++i) vec[i] = 0;
    }
    Vector(int v[]) {
        for (int i = 0; i < 5; ++i) vec[i] = v[i];
    }
    Vector(const Vector &obj) {
        for (int i = 0; i < 5; ++i) vec[i] = obj.vec[i];
    }
    Vector operator+(const Vector &obj) {
        Vector ans;
        for (int i = 0; i < 5; ++i) ans.vec[i] = vec[i] + obj.vec[i];
        return ans;
    }
    Vector operator-(const Vector &obj) {
        Vector ans;
        for (int i = 0; i < 5; ++i) ans.vec[i] = vec[i] - obj.vec[i];
        return ans;
    }
    void print() {
        for (int i = 0; i < 5; ++i) cout << vec[i] << ' ';
        cout << endl;
    }
};

int main() {
    int a[5], b[5];
    for (int i = 0; i < 5; ++i) cin >> a[i];
    for (int i = 0; i < 5; ++i) cin >> b[i];
    Vector x(a), y(b);
    (x + y).print();
    (x - y).print();
    return 0;
}
```

---

### 14.4 矩阵相乘（运算符重载）

> 题目 ID：`83` · 20 分 · 限制：1s / 128MB

#### 题目要求

定义一个矩阵类MyMatrix，并且在类中进行运算符重定义，用*实现矩阵相乘。要求必须对运算符进行重载，如果用函数如multiply（matrix，matrix）去实现矩阵之间的运算一律记0分。

必须包含以下析构函数，其中n是矩阵的阶数，data是存放矩阵数据的二维动态数组：

MyMatrix::~MyMatrix() { // 释放空间 for (int i = 0; i < n; i++) { delete[] data[i]; } delete[] data; }

#### 输入格式

第一行输入所需要的矩阵个数c;

第二行输入矩阵的阶数n，即矩阵是一个n*n的矩阵;

第三行开始依次输入c个矩阵.

#### 输出格式

c个矩阵相乘的结果

#### 样例输入

```text
2
2
1 2
1 2
1 0
1 1
```

#### 样例输出

```text
3 2
3 2
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

class MyMatrix {
private:
    int n;
    int **data;

    void allocate() {
        data = new int *[n];
        for (int i = 0; i < n; ++i) data[i] = new int[n];
    }

public:
    MyMatrix(int n1 = 0) : n(n1), data(nullptr) {
        if (n) allocate();
    }

    MyMatrix(const MyMatrix &other) : n(other.n), data(nullptr) {
        allocate();
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                data[i][j] = other.data[i][j];
    }

    MyMatrix &operator=(const MyMatrix &other) {
        if (this == &other) return *this;
        for (int i = 0; i < n; ++i) delete[] data[i];
        delete[] data;
        n = other.n;
        allocate();
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                data[i][j] = other.data[i][j];
        return *this;
    }

    ~MyMatrix() {
        for (int i = 0; i < n; ++i) delete[] data[i];
        delete[] data;
    }

    void input() {
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                cin >> data[i][j];
    }

    MyMatrix operator*(const MyMatrix &b) const {
        MyMatrix ans(n);
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                ans.data[i][j] = 0;
                for (int k = 0; k < n; ++k) ans.data[i][j] += data[i][k] * b.data[k][j];
            }
        }
        return ans;
    }

    void print() const {
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (j) cout << ' ';
                cout << data[i][j];
            }
            cout << endl;
        }
    }
};

int main() {
    int c, n;
    cin >> c >> n;
    MyMatrix ans(n);
    ans.input();
    for (int i = 1; i < c; ++i) {
        MyMatrix cur(n);
        cur.input();
        ans = ans * cur;
    }
    ans.print();
    return 0;
}
```

---

### 14.5 最小生日差值计算（运算符重载）

> 题目 ID：`342` · 20 分 · 限制：1s / 200MB

#### 题目要求

定义一个学生类Student，包含该学生的姓名、出生年、月、日 ，重定义 “-”号实现两个学生之间相差多少天的比较。并利用重载的“-”运算符，求所有学生中年龄相差最小的两个人的名字以及相差天数。

#### 输入格式

第一行：输入所需要输入的学生个数；

第二行开始，依次输入每个学生的姓名、出生年、月、日。

#### 输出格式

输出年龄相差最小的两个人的名字以及相差天数，名字的输出顺序按输入的先后，天数大于等于0。

#### 样例输入

```text
3
Tom 1995 1 1
Joe 1995 2 28
Jimmy 1996 1 8
```

#### 样例输出

```text
Tom和Joe年龄相差最小，为58天。
```

#### 完整参考代码（带注释）

```cpp
#include <cmath>
#include <iostream>
#include <string>
#include <vector>
using namespace std;

bool leap(int y) {
    return y % 400 == 0 || (y % 4 == 0 && y % 100 != 0);
}

int daysBeforeYear(int y) {
    int ans = 0;
    for (int i = 1; i < y; ++i) ans += leap(i) ? 366 : 365;
    return ans;
}

int dayOfYear(int y, int m, int d) {
    int md[] = {0,31,28,31,30,31,30,31,31,30,31,30,31};
    if (leap(y)) md[2] = 29;
    int ans = d;
    for (int i = 1; i < m; ++i) ans += md[i];
    return ans;
}

class Student {
public:
    string name;
    int y, m, d;
    Student(string n, int yy, int mm, int dd) : name(n), y(yy), m(mm), d(dd) {}
    int serial() const { return daysBeforeYear(y) + dayOfYear(y, m, d); }
    int operator-(const Student &other) const {
        return abs(serial() - other.serial());
    }
};

int main() {
    int n;
    cin >> n;
    vector<Student> s;
    for (int i = 0; i < n; ++i) {
        string name;
        int y, m, d;
        cin >> name >> y >> m >> d;
        s.push_back(Student(name, y, m, d));
    }

    int best = s[0] - s[1], a = 0, b = 1;
    for (int i = 0; i < n; ++i) {
        for (int j = i + 1; j < n; ++j) {
            int diff = s[i] - s[j];
            if (diff < best) {
                best = diff;
                a = i;
                b = j;
            }
        }
    }
    cout << s[a].name << "和" << s[b].name << "年龄相差最小，为" << best << "天。" << endl;
    return 0;
}
```

---

## 实验15. 运算符重载应用

> 测验 ID：`1308` · 时间：2026-06-09T10:15:00+08:00 至 2026-06-16T00:00:00+08:00 · 共 5 题

### 15.1 三维坐标点的平移（运算符重载）

> 题目 ID：`81` · 20 分 · 限制：1s / 128MB

#### 题目要求

定义一个三维点Point类，利用友元函数重载"++"和"--"运算符，并区分这两种运算符的前置和后置运算。

要求如下：

1.实现Point类；

2.编写main函数，初始化1个Point对象，将这个对象++或--后赋给另外一个对象，并输出计算后对象的坐标信息。

#### 输入格式

第1行：输入三个int类型的值，分别为一个Point对象p1的x,y,z坐标。

#### 输出格式

第1行：Point对象p1后置++之后的坐标信息输出。

第2行：Point对象p1后置++操作后赋给另外一个Point对象p2的坐标信息。

第3行开始，依次输出前置++，后置--，前置--运算的坐标信息，输出格式与后置++一样。

即：

p2=p1++;

第1行输出p1

第2行输出p2

p1恢复原值

p2=++p1;

第3行输出p1

第4行输出p2

p1恢复原值

p2=p1--;

第5行输出p1

第6行输出p2

p1恢复原值

p2=--p1;

第7行输出p1

第8行输出p2

#### 提示

原值是最初输入的数值

第1行是p1后置++后，再输出

第2行是p1恢复原值，接着p1后置++同时复制给p2，p2输出

第3、4行是p1恢复原值，p1前置++同时输出，然后p1再输出

第5、6行是p1恢复原值，p1后置--后，再输出，接着输出一次原值

第7、8行是p1恢复原值，p1前置--同时输出，然后p1再输出

#### 样例输入

```text
10 20 30
```

#### 样例输出

```text
x=11 y=21 z=31
x=10 y=20 z=30
x=11 y=21 z=31
x=11 y=21 z=31
x=9 y=19 z=29
x=10 y=20 z=30
x=9 y=19 z=29
x=9 y=19 z=29
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

class Point {
private:
    int x, y, z;
public:
    Point(int a = 0, int b = 0, int c = 0) : x(a), y(b), z(c) {}

    friend Point operator++(Point &p) { // 前置++
        ++p.x; ++p.y; ++p.z;
        return p;
    }
    friend Point operator++(Point &p, int) { // 后置++
        Point old = p;
        ++p.x; ++p.y; ++p.z;
        return old;
    }
    friend Point operator--(Point &p) { // 前置--
        --p.x; --p.y; --p.z;
        return p;
    }
    friend Point operator--(Point &p, int) { // 后置--
        Point old = p;
        --p.x; --p.y; --p.z;
        return old;
    }

    void show() const {
        cout << "x=" << x << " y=" << y << " z=" << z << endl;
    }
};

int main() {
    int x, y, z;
    cin >> x >> y >> z;
    Point p1, p2;

    p1 = Point(x, y, z);
    p2 = p1++;
    p1.show();
    p2.show();

    p1 = Point(x, y, z);
    p2 = ++p1;
    p1.show();
    p2.show();

    p1 = Point(x, y, z);
    p2 = p1--;
    p1.show();
    p2.show();

    p1 = Point(x, y, z);
    p2 = --p1;
    p1.show();
    p2.show();
    return 0;
}
```

---

### 15.2 货币加减（运算符重载）

> 题目 ID：`191` · 20 分 · 限制：1s / 128MB

#### 题目要求

定义CMoney类，包含元、角、分三个数据成员，友元函数重载‘+’、'-'，实现货币的加减运算（假设a-b中a的金额始终大于等于b的金额），

重载输入及输出，实现货币的输入，输出。

读入最初的货币值，对其不断进行加、减操作，输出结果。

可根据需要，为CMoney类添加构造函数或其它成员函数。

#### 输入格式

测试次数

每组测试数据格式如下：

第一行，初始货币：元 角 分

第二行开始，每行一个操作：add 元 角 分（加）、minus 元 角 分（减）、stop（结束）

#### 输出格式

对每组测试数据，输出操作终止后的货币金额，具体输出格式见样例。

#### 提示

<<和>>都需要重载

#### 样例输入

```text
2
0 0 0
add 48 9 0
minus 0 5 3
add 18 6 8
add 12 1 2
stop
10 2 5
add 5 8 0
add 32 1 2
minus 10 5 9
minus 37 5 8
stop
```

#### 样例输出

```text
79元1角7分
0元0角0分
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
#include <string>
using namespace std;

class CMoney {
private:
    int total; // 统一用“分”保存，进位退位最简单
public:
    CMoney(int y = 0, int j = 0, int f = 0) : total(y * 100 + j * 10 + f) {}

    CMoney operator+(const CMoney &b) const {
        CMoney ans;
        ans.total = total + b.total;
        return ans;
    }
    CMoney operator-(const CMoney &b) const {
        CMoney ans;
        ans.total = total - b.total;
        return ans;
    }

    friend istream &operator>>(istream &in, CMoney &m) {
        int y, j, f;
        in >> y >> j >> f;
        m.total = y * 100 + j * 10 + f;
        return in;
    }

    friend ostream &operator<<(ostream &out, const CMoney &m) {
        out << m.total / 100 << "元" << m.total % 100 / 10 << "角" << m.total % 10 << "分";
        return out;
    }
};

int main() {
    int t;
    cin >> t;
    while (t--) {
        CMoney cur;
        cin >> cur;
        string op;
        while (cin >> op && op != "stop") {
            CMoney x;
            cin >> x;
            if (op == "add") cur = cur + x;
            else if (op == "minus") cur = cur - x;
        }
        cout << cur << endl;
    }
    return 0;
}
```

---

### 15.3 时钟调整（运算符前后增量）

> 题目 ID：`93` · 20 分 · 限制：1s / 128MB

#### 题目要求

假定一个时钟包含时、分、秒三个属性，取值范围分别为0~11，0~59，0~59，具体要求如下：

1、用一元运算符++，并且是前增量的方法，实现时钟的调快操作。例如要把时钟调快5秒，则执行5次”  ++<对象> “ 的操作

2、用一元运算符--，并且是后增量的方法，实现时钟的调慢操作。例如要把时钟调慢10秒，则执行10次” <对象>-- “的操作

3、用构造函数的方法实现时钟对象的初始化，用输出函数实现时钟信息的输出

clock和time是系统内部函数，所以不要用来做类名或者其他

#### 输入格式

第一行输入时钟的当前时间时、分、秒

第二行输入t表示有t个示例

第三行输入t个整数x，如果x为正整数，则表示执行调快操作，使用重载运算符++；如果x为负整数，则表示执行调慢操作，使用重载运算符--；如果x为0，则不执行操作，直接输出

每次的调快或调慢操作都是承接上一次调整后的结果进行，例如先调快10秒，再调慢2秒，那么调慢2秒是接着调快10秒后的结果进行的

#### 输出格式

每行输出每个时钟调整操作后的时分秒

#### 样例输入

```text
11 58 46
4
5 70 -22 -55
```

#### 样例输出

```text
11:58:51
0:0:1
11:59:39
11:58:44
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

class MyClock {
private:
    int h, m, s;

    void normalize(int delta) {
        int total = h * 3600 + m * 60 + s + delta;
        int mod = 12 * 3600;
        total = (total % mod + mod) % mod;
        h = total / 3600;
        m = total % 3600 / 60;
        s = total % 60;
    }

public:
    MyClock(int hh = 0, int mm = 0, int ss = 0) : h(hh), m(mm), s(ss) {}
    MyClock &operator++() { normalize(1); return *this; } // 前置++：调快 1 秒
    MyClock operator--(int) { MyClock old = *this; normalize(-1); return old; } // 后置--：调慢 1 秒
    void print() const { cout << h << ":" << m << ":" << s << endl; }
};

int main() {
    int h, m, s, t;
    cin >> h >> m >> s >> t;
    MyClock c(h, m, s);
    while (t--) {
        int x;
        cin >> x;
        if (x > 0) while (x--) ++c;
        else while (x++) c--;
        c.print();
    }
    return 0;
}
```

---

### 15.4 X的放大与缩小（运算符重载）

> 题目 ID：`190` · 20 分 · 限制：1s / 128MB

#### 题目要求

X字母可以放大和缩小，变为n行X（n=1,3,5,7,9,...,21）。例如，3行x图案如下：

现假设一个n行（n>0，奇数）X图案，遥控器可以控制X图案的放大与缩小。遥控器有5个按键，1）show，显示当前X图案；2）show++, 显示当前X图案，再放大图案，n+2；3）++show，先放大图案，n+2，再显示图案；4）show--，显示当前X图案，再缩小图案，n-2；5）--show，先缩小图案，n-2，再显示图案。假设X图案的放大和缩小在1-21之间。n=1时，缩小不起作用，n=21时，放大不起作用。

用类CXGraph表示X图案及其放大、缩小、显示。主函数模拟遥控器，代码如下，不可修改。请补充CXGraph类的定义和实现。

```cpp
int main()
{
    int t, n;
    string command;
    cin >> n;
    CXGraph xGraph(n);
    cin >> t;
    while (t--)
    {
        cin >> command;
        if (command == "show++")
        {
            cout << xGraph++ << endl;
        }
        else if(command == "++show")
        {
            cout << ++xGraph << endl;
        }
        else if (command == "show--")
        {
            cout << xGraph-- << endl;
        }
        else if (command == "--show")
        {
            cout << --xGraph << endl;
        }
        else if (command == "show")
        {
            cout << xGraph << endl;
        }
    }
    return 0;
}
```

#### 输入格式

第一行n，大于0的奇数，X图案的初始大小。

第二行，操作次数

每个操作一行，为show、show++、show--、--show、++show之一，具体操作含义见题目。

#### 输出格式

对每个操作，输出对应的X图案。

#### 样例输入

```text
3
5
show
show++
show++
++show
--show
```

#### 样例输出

```text
XXX
 X
XXX

XXX
 X
XXX

XXXXX
 XXX
  X
 XXX
XXXXX

XXXXXXXXX
 XXXXXXX
  XXXXX
   XXX
    X
   XXX
  XXXXX
 XXXXXXX
XXXXXXXXX

XXXXXXX
 XXXXX
  XXX
   X
  XXX
 XXXXX
XXXXXXX
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
#include <string>
using namespace std;

class CXGraph {
private:
    int n;
    void enlarge() { if (n < 21) n += 2; }
    void shrink() { if (n > 1) n -= 2; }

public:
    CXGraph(int n1 = 1) : n(n1) {}

    CXGraph &operator++() { enlarge(); return *this; }      // ++show
    CXGraph operator++(int) { CXGraph old = *this; enlarge(); return old; } // show++
    CXGraph &operator--() { shrink(); return *this; }       // --show
    CXGraph operator--(int) { CXGraph old = *this; shrink(); return old; }  // show--

    friend ostream &operator<<(ostream &out, const CXGraph &g) {
        for (int i = 0; i < g.n; ++i) {
            int blank = i < g.n - 1 - i ? i : g.n - 1 - i;
            int cnt = g.n - 2 * blank;
            for (int j = 0; j < blank; ++j) out << ' ';
            for (int j = 0; j < cnt; ++j) out << 'X';
            if (i != g.n - 1) out << '\n';
        }
        return out;
    }
};

int main() {
    int t, n;
    string command;
    cin >> n;
    CXGraph xGraph(n);
    cin >> t;
    while (t--) {
        cin >> command;
        if (command == "show++") cout << xGraph++ << endl;
        else if (command == "++show") cout << ++xGraph << endl;
        else if (command == "show--") cout << xGraph-- << endl;
        else if (command == "--show") cout << --xGraph << endl;
        else if (command == "show") cout << xGraph << endl;
    }
    return 0;
}
```

---

### 15.5 矩形关系（运算符重载）

> 题目 ID：`188` · 20 分 · 限制：1s / 128MB

#### 题目要求

假设坐标采用二维平面坐标。

定义点类CPoint，包含属性x,y（整型）。方法有：带参构造函数，getX，getY分别返回点的x坐标，y坐标。

定义矩形类CRectangle，包含属性：矩形的左上角坐标leftPoint，右下角坐标rightPoint。类中方法有：

1）带参构造函数，初始化矩形的左上角、右下角

2）重载>运算符，参数为CPoint点对象，假设为p，若p在矩形内，返回true,否则返回false。

3）重载>运算符，第一个矩形若包含第二个矩形（部分边界可以相等），返回true，否则返回false。（要求该函数调用2）实现）

4）重载==运算符，判断两个矩形是否一致，返回true或false。

5）重载*运算符，判断两个矩形是否有重叠部分，返回true或false。

6）重载类型转换运算符，计算矩形的面积并返回，面积是整型。

7）重载<<运算符，输出矩形的两个角坐标，具体格式见样例。

输入2个矩形，计算面积，判断矩形的关系。主函数如下，不可修改。

```cpp
int main()
{
    int t, x1, x2, y1, y2;
    cin >> t;
    while (t--)
    {
        // 矩形1的左上角、右下角
        cin >> x1 >> y1 >> x2 >> y2;
        CRectangle rect1(x1, y1, x2, y2);
        // 矩形2的左上角、右下角
        cin >> x1 >> y1 >> x2 >> y2;
        CRectangle rect2(x1, y1, x2, y2);
        // 输出矩形1的坐标及面积
        cout << "矩形1:" << rect1 << " " << (int)rect1 << endl;
        // 输出矩形2的坐标及面积
        cout << "矩形2:" << rect2 << " " << (int)rect2 << endl;
        if (rect1 == rect2)
        {
            cout << "矩形1和矩形2相等" << endl;
        }
        else if (rect2 > rect1)
        {
            cout << "矩形2包含矩形1" << endl;
        }
        else if (rect1 > rect2)
        {
            cout << "矩形1包含矩形2" << endl;
        }
        else if (rect1 * rect2)
        {
            cout << "矩形1和矩形2相交" << endl;
        }
        else
        {
            cout << "矩形1和矩形2不相交" << endl;
        }
        cout << endl;
    }
    return 0;
}
```

可根据需要，添加构造函数和析构函数。

#### 输入格式

测试次数

每组测试数据如下：

矩形1的左上角、右下角坐标

矩形2的左上角、右下角坐标

#### 输出格式

每组测试数据输出如下，中间以空行分隔：

矩形1的坐标和面积（具体格式见样例）

矩形2的坐标和面积（具体格式见样例）

矩形1和矩形2的关系（矩形1包含矩形2、矩形2包含矩形1、矩形1和矩阵2相等、矩形1和矩形2相交、矩形1和矩形2不相交）

#### 样例输入

```text
2
1 4 4 1
2 3 3 2
1 4 4 1
0 3 5 2
```

#### 样例输出

```text
矩形1:1 4 4 1 9
矩形2:2 3 3 2 1
矩形1包含矩形2

矩形1:1 4 4 1 9
矩形2:0 3 5 2 5
矩形1和矩形2相交
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

class CPoint {
private:
    int x, y;
public:
    CPoint(int x1 = 0, int y1 = 0) : x(x1), y(y1) {}
    int getX() const { return x; }
    int getY() const { return y; }
};

class CRectangle {
private:
    CPoint leftPoint, rightPoint; // 左上角、右下角
public:
    CRectangle(int x1, int y1, int x2, int y2) : leftPoint(x1, y1), rightPoint(x2, y2) {}

    bool operator>(const CPoint &p) const {
        return leftPoint.getX() <= p.getX() && p.getX() <= rightPoint.getX()
            && rightPoint.getY() <= p.getY() && p.getY() <= leftPoint.getY();
    }

    bool operator>(const CRectangle &r) const {
        return (*this > r.leftPoint) && (*this > r.rightPoint);
    }

    bool operator==(const CRectangle &r) const {
        return leftPoint.getX() == r.leftPoint.getX()
            && leftPoint.getY() == r.leftPoint.getY()
            && rightPoint.getX() == r.rightPoint.getX()
            && rightPoint.getY() == r.rightPoint.getY();
    }

    bool operator*(const CRectangle &r) const {
        // 两个投影都重叠即相交。
        bool xOverlap = !(rightPoint.getX() < r.leftPoint.getX() || r.rightPoint.getX() < leftPoint.getX());
        bool yOverlap = !(leftPoint.getY() < r.rightPoint.getY() || r.leftPoint.getY() < rightPoint.getY());
        return xOverlap && yOverlap;
    }

    operator int() const {
        return (rightPoint.getX() - leftPoint.getX()) * (leftPoint.getY() - rightPoint.getY());
    }

    friend ostream &operator<<(ostream &out, const CRectangle &r) {
        out << r.leftPoint.getX() << ' ' << r.leftPoint.getY() << ' '
            << r.rightPoint.getX() << ' ' << r.rightPoint.getY();
        return out;
    }
};

int main() {
    int t, x1, x2, y1, y2;
    cin >> t;
    while (t--) {
        cin >> x1 >> y1 >> x2 >> y2;
        CRectangle rect1(x1, y1, x2, y2);
        cin >> x1 >> y1 >> x2 >> y2;
        CRectangle rect2(x1, y1, x2, y2);
        cout << "矩形1:" << rect1 << " " << (int)rect1 << endl;
        cout << "矩形2:" << rect2 << " " << (int)rect2 << endl;
        if (rect1 == rect2) cout << "矩形1和矩形2相等" << endl;
        else if (rect2 > rect1) cout << "矩形2包含矩形1" << endl;
        else if (rect1 > rect2) cout << "矩形1包含矩形2" << endl;
        else if (rect1 * rect2) cout << "矩形1和矩形2相交" << endl;
        else cout << "矩形1和矩形2不相交" << endl;
        cout << endl;
    }
    return 0;
}
```

---

## 实验16. 函数模板与类模板

> 测验 ID：`1322` · 时间：2026-06-16T10:15:00+08:00 至 2026-06-23T00:00:00+08:00 · 共 5 题

### 16.1 元素查找（函数模板）

> 题目 ID：`98` · 20 分 · 限制：1s / 128MB

#### 题目要求

1024x768

Normal
0

7.8 磅
0
2

false
false
false

/* Style Definitions */
table.MsoNormalTable
{mso-style-name:普通表格;
mso-tstyle-rowband-size:0;
mso-tstyle-colband-size:0;
mso-style-noshow:yes;
mso-style-parent:"";
mso-padding-alt:0cm 5.4pt 0cm 5.4pt;
mso-para-margin:0cm;
mso-para-margin-bottom:.0001pt;
mso-pagination:widow-orphan;
font-size:10.0pt;
font-family:"Times New Roman";
mso-ansi-language:#0400;
mso-fareast-language:#0400;
mso-bidi-language:#0400;}

编写一个在数组中进行查找的函数模板，其中数组为具有n个元素，类型为T，要查找的元素为key。

Normal
0

7.8 磅
0
2

false
false
false

MicrosoftInternetExplorer4

/* Style Definitions */
table.MsoNormalTable
{mso-style-name:普通表格;
mso-tstyle-rowband-size:0;
mso-tstyle-colband-size:0;
mso-style-noshow:yes;
mso-style-parent:"";
mso-padding-alt:0cm 5.4pt 0cm 5.4pt;
mso-para-margin:0cm;
mso-para-margin-bottom:.0001pt;
mso-pagination:widow-orphan;
font-size:10.0pt;
font-family:"Times New Roman";
mso-ansi-language:#0400;
mso-fareast-language:#0400;
mso-bidi-language:#0400;}

注意：必须使用模板函数

#### 输入格式

第一行输入t表示有t个测试实例

第二行先输入一个大写字母表示数组类型，I表示整数类型，D表示双精度数类型，C表示字符型，S表示字符串型；然后输入n表示数组长度。

第三行输入n个数据

第四行输入key

依次输入t个实例

#### 输出格式

每行输出一个结果，找到输出key是数组中的第几个元素（从1开始），找不到输出0

#### 样例输入

```text
4
I 5
5 3 51 27 9
27
D 3
-11.3 25.42 13.2
2.7
C 6
a b g e u q
a
S 4
sandy david eason cindy
cindy
```

#### 样例输出

```text
4
0
1
4
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
#include <string>
using namespace std;

template <class T>
int findPos(T a[], int n, T key) {
    for (int i = 0; i < n; ++i)
        if (a[i] == key) return i + 1; // 题目要求从 1 开始
    return 0;
}

template <class T>
void solveOne(int n) {
    T *a = new T[n];
    for (int i = 0; i < n; ++i) cin >> a[i];
    T key;
    cin >> key;
    cout << findPos(a, n, key) << endl;
    delete[] a;
}

int main() {
    int t;
    cin >> t;
    while (t--) {
        char type;
        int n;
        cin >> type >> n;
        if (type == 'I') solveOne<int>(n);
        else if (type == 'D') solveOne<double>(n);
        else if (type == 'C') solveOne<char>(n);
        else if (type == 'S') solveOne<string>(n);
    }
    return 0;
}
```

---

### 16.2 谁的票数最高（函数模板）

> 题目 ID：`99` · 20 分 · 限制：1s / 128MB

#### 题目要求

1024x768

Normal
0

7.8 磅
0
2

false
false
false

/* Style Definitions */
table.MsoNormalTable
{mso-style-name:普通表格;
mso-tstyle-rowband-size:0;
mso-tstyle-colband-size:0;
mso-style-noshow:yes;
mso-style-parent:"";
mso-padding-alt:0cm 5.4pt 0cm 5.4pt;
mso-para-margin:0cm;
mso-para-margin-bottom:.0001pt;
mso-pagination:widow-orphan;
font-size:10.0pt;
font-family:"Times New Roman";
mso-ansi-language:#0400;
mso-fareast-language:#0400;
mso-bidi-language:#0400;}

某小镇要票选镇长，得票最高者当选。但由于投票机制不健全，导致每届投票时，候选人在投票系统的识别码类型不一致。请编写函数模板，能针对多种类型的数据，查找出得票最高的元素。其中，每届投票的选票有n张，识别码类型为T

注意：必须使用模板函数

Normal
0

7.8 磅
0
2

false
false
false

MicrosoftInternetExplorer4

/* Style Definitions */
table.MsoNormalTable
{mso-style-name:普通表格;
mso-tstyle-rowband-size:0;
mso-tstyle-colband-size:0;
mso-style-noshow:yes;
mso-style-parent:"";
mso-padding-alt:0cm 5.4pt 0cm 5.4pt;
mso-para-margin:0cm;
mso-para-margin-bottom:.0001pt;
mso-pagination:widow-orphan;
font-size:10.0pt;
font-family:"Times New Roman";
mso-ansi-language:#0400;
mso-fareast-language:#0400;
mso-bidi-language:#0400;}

#### 输入格式

第一行输入t表示有t个测试实例

第二行先输入一个大写字母表示识别码类型，I表示整数类型，C表示字符型，S表示字符串型；然后输入n表示数组长度。

第三行输入n个数据

依次输入t个实例

#### 输出格式

每行输出一个结果，分别输出当选者的识别码和得票数，以空格分开。

#### 样例输入

```text
3
I 10
5 3 5 2 9 7 3 7 2 3
C 8
a b a e b e e q
S 5
sandy david eason cindy cindy
```

#### 样例输出

```text
3 3
e 3
cindy 2
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
#include <map>
#include <string>
#include <vector>
using namespace std;

template <class T>
void winner(int n) {
    vector<T> a(n);
    map<T, int> cnt;
    T best;
    int bestCnt = -1;
    for (int i = 0; i < n; ++i) {
        cin >> a[i];
        cnt[a[i]]++;
        // 只在严格更多时更新，票数相同保留先达到最高票的候选。
        if (cnt[a[i]] > bestCnt) {
            bestCnt = cnt[a[i]];
            best = a[i];
        }
    }
    cout << best << ' ' << bestCnt << endl;
}

int main() {
    int t;
    cin >> t;
    while (t--) {
        char type;
        int n;
        cin >> type >> n;
        if (type == 'I') winner<int>(n);
        else if (type == 'C') winner<char>(n);
        else if (type == 'S') winner<string>(n);
    }
    return 0;
}
```

---

### 16.3 抢票验证码（函数模板）

> 题目 ID：`331` · 20 分 · 限制：1s / 128MB

#### 题目要求

对于某抢票系统，存在一个长度为六的验证码，验证码可以为整形，字符型，浮点型，我们认为验证码是有效的当且仅当验证码是非递减序列（递增或者相等）。现请你利用函数模板，完成对验证码的检验

#### 输入格式

测试数据有多组，每组测试数据给出验证码类型以及一串验证码

#### 输出格式

输出验证的结果

#### 样例输入

```text
c
a b c d e f
i
1 2 3 4 5 6
f
1.1 1.2 1.3 4 5 6
c
f e v a c s
```

#### 样例输出

```text
Valid
Valid
Valid
Invalid
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

template <class T>
bool valid(T a[]) {
    for (int i = 0; i < 5; ++i)
        if (a[i] > a[i + 1]) return false; // 非递减：允许相等
    return true;
}

template <class T>
void solve() {
    T a[6];
    for (int i = 0; i < 6; ++i) cin >> a[i];
    cout << (valid(a) ? "Valid" : "Invalid") << endl;
}

int main() {
    char type;
    while (cin >> type) {
        if (type == 'c') solve<char>();
        else if (type == 'i') solve<int>();
        else if (type == 'f') solve<double>();
    }
    return 0;
}
```

---

### 16.4 矩阵类模板（类模板）

> 题目 ID：`102` · 20 分 · 限制：1s / 128MB

#### 题目要求

设计一个矩阵类模板Matrix，支持任意数据类型的数据。

要求至少包含2个成员函数：矩阵转置函数transport、以及打印输出函数print

编写main函数进行测试，调用类的成员函数完成转置和输出。

#### 输入格式

第一行先输入t，表示有t个测试用例

从第二行开始输入每个测试用例的数据。

首先输入数据类型，I表示int，D表示double，C表示char，接着输入两个参数m和n，分别表示矩阵的行和列

接下来输入矩阵的元素，一共m行，每行n个数据

#### 输出格式

输出转置后的矩阵

#### 样例输入

```text
2
I 2 3
1 2 3
4 5 6
C 3 3
a b c
d e f
g h i
```

#### 样例输出

```text
1 4
2 5
3 6
a d g
b e h
c f i
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

template <class T>
class Matrix {
private:
    int m, n;
    T **data;
public:
    Matrix(int r, int c) : m(r), n(c) {
        data = new T *[m];
        for (int i = 0; i < m; ++i) data[i] = new T[n];
    }
    ~Matrix() {
        for (int i = 0; i < m; ++i) delete[] data[i];
        delete[] data;
    }
    void input() {
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                cin >> data[i][j];
    }
    void transport() const {
        for (int j = 0; j < n; ++j) {
            for (int i = 0; i < m; ++i) {
                if (i) cout << ' ';
                cout << data[i][j];
            }
            cout << endl;
        }
    }
};

template <class T>
void solve(int m, int n) {
    Matrix<T> mat(m, n);
    mat.input();
    mat.transport();
}

int main() {
    int t;
    cin >> t;
    while (t--) {
        char type;
        int m, n;
        cin >> type >> m >> n;
        if (type == 'I') solve<int>(m, n);
        else if (type == 'D') solve<double>(m, n);
        else if (type == 'C') solve<char>(m, n);
    }
    return 0;
}
```

---

### 16.5 简单类模板(类模板)

> 题目 ID：`101` · 20 分 · 限制：1s / 128MB

#### 题目要求

定义一个列表类，该列表包含属性：数值列表（用长度为100的数组表示），数据长度（实际的数据个数）；包含的方法：初始化、插入、删除、打印，方法定义为：

1）初始化，接受外来参数，把数据保存在数值列表中，未使用的列表部分全部初始化为-1

2）插入，接受外来参数的插入位置和插入数值，插入位置从0开始计算，注意从插入位置开始，原有数据都要往后移动一位，且数据长度+1

3）删除，接受外来参数的删除位置，删除位置从0开始计算，注意从删除位置后一位开始，原有数据都要往前移动一位，且数据长度-1

4）打印，把包含的数据按位置顺序输出一行，数据之间单个空格隔开

使用类模板的方法，使得这个类支持整数int类型和浮点数double类型

#### 输入格式

第一行先输入参数n表示有n个数据，接着输入n个整数

第二行输入两个参数，表示插入位置和插入数值，数值为整数

第三行输入删除位置

第四行先输入参数n表示有n个数据，接着输入n个浮点数

第五行输入两个参数，表示插入位置和插入数值，数值为浮点数

第六行输入删除位置

#### 输出格式

针对头三行输入，分别执行初始化、插入操作和删除操作，调用打印方法输出列表包含的整数数据

针对接着的三行输入，分别执行初始化、插入操作和删除操作，调用打印方法输出列表包含的浮点数数据

#### 样例输入

```text
5 11 22 33 44 55
2 888
4
5 1.1 2.2 3.3 4.4 5.5
2 88.8
3
```

#### 样例输出

```text
11 22 888 33 55
1.1 2.2 88.8 4.4 5.5
```

#### 完整参考代码（带注释）

```cpp
#include <iostream>
using namespace std;

template <class T>
class List {
private:
    T data[100];
    int len;
public:
    List() : len(0) {
        for (int i = 0; i < 100; ++i) data[i] = -1;
    }
    void init(int n) {
        len = n;
        for (int i = 0; i < len; ++i) cin >> data[i];
        for (int i = len; i < 100; ++i) data[i] = -1;
    }
    void insert(int pos, T value) {
        for (int i = len; i > pos; --i) data[i] = data[i - 1];
        data[pos] = value;
        ++len;
    }
    void erase(int pos) {
        for (int i = pos; i + 1 < len; ++i) data[i] = data[i + 1];
        --len;
    }
    void print() const {
        for (int i = 0; i < len; ++i) {
            if (i) cout << ' ';
            cout << data[i];
        }
        cout << endl;
    }
};

int main() {
    int n, pos;

    List<int> li;
    cin >> n;
    li.init(n);
    int iv;
    cin >> pos >> iv;
    li.insert(pos, iv);
    cin >> pos;
    li.erase(pos);
    li.print();

    List<double> ld;
    cin >> n;
    ld.init(n);
    double dv;
    cin >> pos >> dv;
    ld.insert(pos, dv);
    cin >> pos;
    ld.erase(pos);
    ld.print();
    return 0;
}
```

---

## 实验17. 期末综合练习

> 测验 ID：`1320` · 时间：2026-06-23T10:15:00+08:00 至 2026-06-30T00:00:00+08:00 · 共 5 题

### 17.1 音像制品（类与对象）

> 题目 ID：`159` · 20 分 · 限制：1s / 128MB

#### 题目要求

某商店出租音像制品，制品信息包括：类型、名称、租金单价、状态。

其中类型用单个数字表示，对应关系为：1-黑胶片，2-CD，3-VCD，4-DVD

名称是字符串，存储制品的名称信息

租金单价表示每天租金价格

状态用单个数字表示，0是未出租，1是已出租

商店提供业务操作包括

1. 初始化(使用构造方法)，从键盘输入音像制品的信息，并设置到对象中

2. 查询print，输出音像制品的信息

3. 计算租金fee，参数是租借的天数，输出租金总价，如果未出租则提示，具体输出信息看示范

请定义音像制品类，并创建相应的对象来完成操作

题目涉及的数值均用整数处理

#### 输入格式

第一行输入n表示有n个音像制品

每个音像制品对应两行输入

一行输入一个音像制品的多个参数，具体为：类型、名称、租金单价（正整数）、状态

一行输入操作命令，如果输入为0表示查询操作，非0则表示查询并且计算租金费用，租用天数就是这个非0值

依次输入2n行

#### 输出格式

根据每个音像制品的操作命令，调用相应的操作，然后输出相关结果

输出样式看示范。

#### 样例输入

```text
4
1 AAA 43 1
0
2 BBB 19 0
3
3 CCC 27 1
5
4 DDD 32 1
7
```

#### 样例输出

```text
黑胶片[AAA]已出租
CD[BBB]未出租
未产生租金
VCD[CCC]已出租
当前租金为135
DVD[DDD]已出租
当前租金为224
```

#### 关键代码（带注释）

下面给出类设计与核心成员函数，并说明 main 中的调用顺序。

```cpp
class Video {
private:
    int type, price, status;
    string name;

    string typeName() const {
        string names[] = {"", "黑胶片", "CD", "VCD", "DVD"};
        return names[type];
    }

public:
    Video(int t, string n, int p, int s) : type(t), price(p), status(s), name(n) {}

    void print() const {
        cout << typeName() << "[" << name << "]" << (status ? "已出租" : "未出租") << endl;
    }

    void fee(int days) const {
        if (!status) cout << "未产生租金" << endl;
        else cout << "当前租金为" << price * days << endl;
    }
};

// 使用方式：每个对象先 print()；如果命令非 0，再 fee(command)。
```

---

### 17.2 OOP双人决斗（多重继承）

> 题目 ID：`300` · 20 分 · 限制：1s / 128MB

#### 题目要求

写一个Node2D基类，属性有位置location(String)

一个Body子类继承自Node2D，属性初始生命值maxHealth，当前生命值health，防御力defense

一个Weapon子类也继承自Node2D，属性有武器名w_name，武器伤害damage

一个Player多继承自Body和Weapon，属性有名字name,方法有attack，对目标造成伤害

在主函数创建两个Player，p1、p2，判断在p1首先开始攻击的情况下谁会获胜

我们规定，每次造成的伤害等于damage减去defense

#### 输入格式

输入：地点location,玩家1的名字、生命值、防御力、武器名、武器伤害，玩家2的名字、生命值、防御力、武器名、武器伤害

#### 输出格式

输出：获胜信息

#### 样例输入

```text
palace
p1 30 5 bow 30
p2 50 10 sword 20
```

#### 样例输出

```text
p1 deal 20 damage to p2
p2 still have 30 health

p2 deal 15 damage to p1
p1 still have 15 health

p1 deal 20 damage to p2
p2 still have 10 health

p2 deal 15 damage to p1
p2 defeated p1 by sword in palace
```

#### 关键代码（带注释）

下面给出战斗循环和攻击函数的关键写法。

```cpp
class Player : public Body, public Weapon {
public:
    string name;

    int attack(Player &target) {
        int hurt = damage - target.defense;
        if (hurt < 0) hurt = 0; // 防御高于攻击时按 0 伤害处理更稳妥
        target.health -= hurt;
        return hurt;
    }
};

void duel(Player &p1, Player &p2, const string &location) {
    Player *attacker = &p1, *target = &p2;
    while (true) {
        int hurt = attacker->attack(*target);
        cout << attacker->name << " deal " << hurt << " damage to " << target->name << endl;
        if (target->health <= 0) {
            cout << attacker->name << " defeated " << target->name
                 << " by " << attacker->w_name << " in " << location << endl;
            break;
        }
        cout << target->name << " still have " << target->health << " health" << endl << endl;
        swap(attacker, target); // 双方轮流攻击，p1 先手
    }
}
```

---

### 17.3 宠物的生长（虚函数和多态）

> 题目 ID：`181` · 20 分 · 限制：1s / 128MB

#### 题目要求

需要开发一个系统，对宠物的生长状态进行管理。给出下面的基类框架：

class Pet

{ protected:

string name;//姓名

float length;//身长

float weight;//体重

CDate current;//开始记录时间

（日期类CDate包含年、月、日三个私有数据，其他方法根据需要自拟。）

public:

virtual void display(CDate day)=0;//输出目标日期时宠物的身长和体重

}

以Pet为基类，构建出Cat和Dog两个类:

Cat一天身长加0.1，体重加0.2。

Dog一天身长加0.2，体重加0.1。

生成上述类并编写主函数，要求主函数中有一个基类指针Pet *pt，用于测试子类数据。

主函数根据输入的信息，相应建立Cat类对象或Dog类对象，并给出测量日期时宠物的身长和体重。

#### 输入格式

第一行为测试次数

第二行是开始记录日期

从第三行起，每个测试用例占一行，每行给出宠物的基本信息：宠物的类型（1为Cat，2为Dog）、名字、身长、体重、最后测量的日期。

#### 输出格式

要求输出目标日期宠物姓名、身长和体重（结果要求保留小数点后2位）。若测量日期小于开始日期，输出”error”。

#### 样例输入

```text
3
2019 5 5
1 tony 10 10 2018 12 30
2 jerry 5 6 2019 5 10
1 tom 3 4 2019 6 1
```

#### 样例输出

```text
error
jerry after 5 day: length=6.00,weight=6.50
tom after 27 day: length=5.70,weight=9.40
```

#### 关键代码（带注释）

下面给出日期差和多态输出的核心代码。

```cpp
bool leap(int y) {
    return y % 400 == 0 || (y % 4 == 0 && y % 100 != 0);
}

int serial(int y, int m, int d) {
    int md[] = {0,31,28,31,30,31,30,31,31,30,31,30,31};
    if (leap(y)) md[2] = 29;
    int ans = d;
    for (int i = 1; i < y; ++i) ans += leap(i) ? 366 : 365;
    for (int i = 1; i < m; ++i) ans += md[i];
    return ans;
}

class Pet {
protected:
    string name;
    double length, weight;
    int startDay;
public:
    virtual void display(int targetDay) = 0;
    virtual ~Pet() {}
};

class Cat : public Pet {
public:
    void display(int targetDay) override {
        int d = targetDay - startDay;
        if (d < 0) { cout << "error" << endl; return; }
        cout << fixed << setprecision(2)
             << name << " after " << d << " day: length=" << length + 0.1 * d
             << ",weight=" << weight + 0.2 * d << endl;
    }
};

class Dog : public Pet {
public:
    void display(int targetDay) override {
        int d = targetDay - startDay;
        if (d < 0) { cout << "error" << endl; return; }
        cout << fixed << setprecision(2)
             << name << " after " << d << " day: length=" << length + 0.2 * d
             << ",weight=" << weight + 0.1 * d << endl;
    }
};
```

---

### 17.4 集合（运算符重载）

> 题目 ID：`187` · 20 分 · 限制：1s / 128MB

#### 题目要求

集合是由一个或多个确定的元素所构成的整体。集合的运算有并、交、相对补等。

集合A和集合B的交集：由属于A且属于B的相同元素组成的集合。

集合A和集合B的并集：由所有属于集合A或属于集合B的元素所组成的集合。

集合B关于集合A的相对补集，记做A-B：由属于A而不属于B的元素组成的集合。

假设集合A={10，20，30}，集合B={1，10，50，8}。则A与B的并是{10，20，30,1,50,8}，A与B的交是{10}，B关于A的相对补集是{20,30}。

定义整数集合类CSet，属性包括：集合中的元素个数n，整型指针data存储集合中的元素。

方法有：重载输出，按样例格式输出集合中的元素。

重载+运算符，求集合A和集合B的并集，并返回结果集合。

重载-运算符，求集合B关于集合A的相对补集，并返回结果集合。

重载*运算符，求集合A和集合B的交集，并返回结果集合。

主函数输入集合A、B的数据，计算集合的并、交、相对补。

可根据题目，为CSet类添加需要的成员函数。

#### 输入格式

测试次数

每组测试数据两行，格式如下：

第一行：集合A的元素个数和元素

第二行：集合B的元素个数和元素

#### 输出格式

每组测试数据输出如下：

第一行：集合A

第二行：集合B

第三行：A和B的并

第四行：A和B的交

第五行：B关于A的相对补集 与 A关于B的相对补集的并，即(A-B)+(B-A)

每组测试数据间以空行分隔。

#### 样例输入

```text
2
3 10 20 30
4 10 1 2 3
5 100 2 3 4 -10
6 -34 12 2 4 90 100
```

#### 样例输出

```text
A:10 20 30
B:10 1 2 3
A+B:10 20 30 1 2 3
A*B:10
(A-B)+(B-A):20 30 1 2 3

A:100 2 3 4 -10
B:-34 12 2 4 90 100
A+B:100 2 3 4 -10 -34 12 90
A*B:100 2 4
(A-B)+(B-A):3 -10 -34 12 90
```

#### 关键代码（带注释）

下面给出集合运算符的核心实现。

```cpp
class CSet {
private:
    vector<int> data;

    bool has(int x) const {
        for (int v : data) if (v == x) return true;
        return false;
    }

public:
    CSet operator+(const CSet &b) const { // 并集：先 A 后 B，去重
        CSet ans = *this;
        for (int v : b.data)
            if (!ans.has(v)) ans.data.push_back(v);
        return ans;
    }

    CSet operator-(const CSet &b) const { // A-B：A 中有且 B 中没有
        CSet ans;
        for (int v : data)
            if (!b.has(v)) ans.data.push_back(v);
        return ans;
    }

    CSet operator*(const CSet &b) const { // 交集：按 A 的顺序
        CSet ans;
        for (int v : data)
            if (b.has(v)) ans.data.push_back(v);
        return ans;
    }
};

// 对称差按题意输出：(A-B)+(B-A)。
```

---

### 17.5 最贵的书（重载+友元+引用）

> 题目 ID：`192` · 20 分 · 限制：1s / 128MB

#### 题目要求

定义CBook，属性包含书名（string），编者（string）、售价（double）,出版社（string）。方法有：重载输入、输出。

定义友元函数find(CBook *book, int n, int &max1index,int &max2index)查找n本书中售价最高、次高的两本书，并通过引用返回其下标。若有相同售价最高、次高的两本书，按输入顺序输出第一本、第二本。

输入n，输入n本书的信息，调用上述友元函数，求价格最高的两本书下标，并按样例格式输出书信息。

#### 输入格式

测试次数

每组测试数据格式如下：

n

n行书信息(书名,编者,售价,出版社)

#### 输出格式

每组测试数据输出两行：

第一行：售价最高的书信息。

第二行：售价次高的书信息。

具体输出格式见样例，售价保留两位小数。书中间以空格分隔。

#### 提示

读取‘,’前的字符用 getline进行输入，如下所示

#include <string>

string  name;

getline(cin, name, ',');

#### 样例输入

```text
1
5
python从入门到精通,艾里克.马瑟斯,62.00,人民邮电出版社
Java并发编程实战,盖茨,54.5,机械工业出版社
Effective Java中文版,约书亚.布洛克,94,机械工业出版社
重构 改善既有代码的设计,马丁.福勒,122.6,人民邮电出版社
活用数据：驱动业务的数据分析实战,陈哲,61.4,电子工业出版社
```

#### 样例输出

```text
重构 改善既有代码的设计
马丁.福勒
122.60
人民邮电出版社

Effective Java中文版
约书亚.布洛克
94.00
机械工业出版社
```

#### 关键代码（带注释）

下面给出逗号分隔输入、友元查找和格式化输出的核心代码。

```cpp
class CBook {
private:
    string name, author, press;
    double price;
public:
    friend istream &operator>>(istream &in, CBook &b) {
        string priceText;
        getline(in, b.name, ',');
        getline(in, b.author, ',');
        getline(in, priceText, ',');
        getline(in, b.press);
        b.price = stod(priceText);
        return in;
    }

    friend ostream &operator<<(ostream &out, const CBook &b) {
        out << b.name << '\n' << b.author << '\n'
            << fixed << setprecision(2) << b.price << '\n'
            << b.press << '\n';
        return out;
    }

    friend void find(CBook *book, int n, int &max1index, int &max2index);
};

void find(CBook *book, int n, int &max1index, int &max2index) {
    max1index = 0;
    max2index = 1;
    if (book[max2index].price > book[max1index].price) swap(max1index, max2index);
    for (int i = 2; i < n; ++i) {
        if (book[i].price > book[max1index].price) {
            max2index = max1index;
            max1index = i;
        } else if (book[i].price > book[max2index].price) {
            max2index = i;
        }
    }
}
```

---

## 实验18. 期末模拟

> 测验 ID：`1349` · 时间：2026-06-30T10:10:00+08:00 至 2026-07-13T00:00:00+08:00 · 共 5 题

> **说明：** 实验 18 的截止时间是 2026-07-13 00:00（UTC+8）。下列内容保留完整题目要求，并给出足以自行完成和核对的类设计、公式、边界条件与伪代码，但不放置可直接提交的完整源码。

### 18.1 一、会员积分（期末模拟）

> 题目 ID：`124` · 20 分 · 限制：1s / 128MB

#### 题目要求

某电商网站的会员分为：普通、贵宾两个级别

普通会员类Member，包含编号、姓名、积分三个属性，编号和积分是整数，姓名是字符串

操作包括构造、打印、积分累加、积分兑换，操作定义如下：

1、积分累加add，是根据消费金额累加积分，无返回值，参数是消费金额（整数），积分根据消费金额按1比1的比例累加。

2、积分兑换exchange，是按照每100积分换1元的比例，把积分兑换成现金。参数是要兑换的积分数量，返回值是兑换的现金数量。

注意：兑换积分数量不足100的部分是不能兑换的，例如会员原有500积分，要兑换积分数量为450，则450/100=4，最终用400积分兑换4元，会员余100积分。

3、打印是输出会员信息，格式参考输出样例

贵宾会员类VIP，继承了普通会员的属性与操作，新增两个属性：累加比例(整数)、兑换比例(整数)。并且重定义了所有操作：

1、积分累加中，积分按累加比例进行累加。例如累加比例是2，消费金额100元，则累加积分=100*2=200

2、积分兑换中，按照兑换比例的数值把积分抵扣消费金额。例如兑换比例是90，会员原有500积分，要兑换积分数量为420，则420/90=4，最终用360积分兑换4元，会员余140积分。

3、打印是输出会员信息，格式参考输出样例

程序要求

1、采用继承机制实现上述会员关系

2、打印、积分累加和积分兑换都采用虚函数方式，来实现运行多态性

3、派生的构造函数必须考虑基类属性的构造。

4、必须采用以下代码框架，在提示的地方增加代码，其他地方不能修改。

上述所有类属性都不是public，用面向对象思想和C++语言实现上述要求

----参考代码----

```cpp
// 普通会员类
class Member 
{  
	// ....代码自行编写
};

// 贵宾会员类
class VIP .... 
{  
	// ....代码自行编写
};

int main()
{
	// 创建一个基类对象指针
	Member* pm;
	// ....其他变量自行编写

	// 输入数据，创建普通会员对象mm
	// 使用指针pm执行以下操作：
	// 1、pm指向普通会员对象mm
	// 2、输入数据，通过pm执行积分累加和积分兑换
	// 3、通过pm调用打印方法输出

	// 输入数据，创建贵宾会员对象vv
	// 使用指针pm执行以下操作：
	// 1、pm指向贵宾会员对象vv
	// 2、输入数据，通过pm执行积分累加和积分兑换
	// 3、通过pm调用打印方法输出

	return 0;
}
```

#### 输入格式

第一行输入普通会员的三个信息：编号、姓名、积分

第二行输入两个操作信息：消费金额、积分兑换数量，表示普通会员执行一次积分累加，一次积分兑换

第三行输入贵宾会员的五个信息：编号、姓名、积分、累加比例、兑换比例

第四行输入两个操作信息：消费金额、积分兑换数量，表示贵宾会员执行一次积分累加，一次积分兑换

#### 输出格式

第一行输出普通会员执行两个操作后的信息，要求调用打印方法

第二行输出贵宾会员执行两个操作后的信息，要求调用打印方法

#### 样例输入

```text
1001 John 500
244 300 
8001 Jane 300 2 90
100 420
```

#### 样例输出

```text
普通会员1001--John--444
贵宾会员8001--Jane--140
```

#### 解题思路与伪代码

**类与多态结构**

- `Member` 保存编号、姓名、积分；`add`、`exchange`、`print` 都应声明为虚函数。
- `VIP` 公有继承 `Member`，新增累加比例和兑换比例，并重写三个虚函数。
- 普通会员累加：`积分 += 消费金额`；VIP 累加：`积分 += 消费金额 × 累加比例`。
- 兑换时只处理兑换比例的整数倍。可兑换现金数应同时受“申请兑换积分”和“账户现有积分”限制。

**核心计算**

```text
单位积分 = 普通会员取 100，VIP 取兑换比例
现金数 = min(申请积分 / 单位积分, 当前积分 / 单位积分)
实际扣分 = 现金数 × 单位积分
当前积分 -= 实际扣分
返回现金数
```

通过 `Member*` 依次指向普通会员和 VIP 对象，再调用虚函数，即可体现运行时多态。输出中的会员类型、连字符和字段顺序必须与样例完全一致。

---

### 18.2 二、金属加工（期末模拟）

> 题目 ID：`125` · 20 分 · 限制：1s / 128MB

#### 题目要求

在金属加工中，金属具有硬度、重量、体积的属性（都是整数），包括四种操作：

1、合并，每两块金属可以合并成一块新的金属。新金属的重量等于原两块金属的重量之和，体积和硬度也类似计算。

2、巨化，金属通过熔炼风吹的方法会巨化，体积变大n倍，重量和硬度不变

3、硬化，在金属中加入高分子材料可以硬化金属，每提升硬度一级，重量和体积都增加10%。

4、软化，在金属中加入特殊化学溶液可以降低金属硬度，每降低硬度一级，重量和体积都减少10%

用类来描述金属，用运算符重载方式实现金属的四种操作，并定义打印函数，具体要求如下

1、用加法运算符、友元的方式实现合并

2、用乘法运算符、友元的方式实现巨化，含两个参数：金属对象、巨化倍数

3、用++运算符、成员函数、前增量的方式实现硬化

4、用--运算符、成员函数、后增量的方式实现软化

5、打印函数用来输出金属的信息，输出格式看参考样本

操作中所有属性的运算结果都只保留整数部分。

上述所有类属性都不是public，用面向对象思想和C++语言实现上述要求

#### 输入格式

第一行输入第一块金属的信息，包括硬度、重量、体积

第二行输入第二块金属的信息，包括硬度、重量、体积

第三行输入一个参数n，表示巨化n倍

#### 输出格式

第一行输出两块金属合并后的信息

第二行输出第一块金属巨化n倍的信息

第三行输出第一块金属提升硬度一级后的信息

第四行输出第二块金属降低硬度一级后的信息

#### 样例输入

```text
3 3000 300
5 5000 500
2
```

#### 样例输出

```text
硬度8--重量8000--体积800
硬度3--重量3000--体积600
硬度4--重量3300--体积330
硬度4--重量4500--体积450
```

#### 解题思路与伪代码

**运算符设计**

- `Metal` 私有保存硬度、重量、体积，三个属性均为整数。
- 友元 `operator+`：三个属性分别相加，返回新对象，不修改两个操作数。
- 友元 `operator*`：只把体积乘以倍数，硬度和重量保持不变。
- 前置 `operator++`：硬度加 1，重量和体积变为原值的 110%，修改并返回当前对象。
- 后置 `operator--(int)`：先保留旧对象，再让硬度减 1、重量和体积变为原值的 90%；后置形式返回旧值。

**取整规则**

重量和体积是整数，计算 110% 或 90% 后直接截去小数部分。为避免先做整数除法，先乘再除，例如：

```text
增加 10%：newValue = oldValue × 110 / 100
减少 10%：newValue = oldValue × 90 / 100
```

注意题目要求区分前置 `++` 和后置 `--` 的函数签名，打印格式为“硬度…--重量…--体积…”。

---

### 18.3 三、加密模板（期末模拟）

> 题目 ID：`126` · 20 分 · 限制：1s / 128MB

#### 题目要求

加密机制包括明文、密文、密钥。用密钥对明文进行加密后就得到密文。

在古典加密机制中，偏离值是一种常见的方法，加密过程为

1、在已知数据中找出最大值

2、用最大值减去各个数值，得到相应的偏离值

3、偏离值加上密钥就得到密文

例如明文为1 2 3 4 5，密钥是10，加密过程为：

1、找出明文的最大值是5

2、用5减去明文的各个数值，得到偏离值4 3 2 1 0

3、用偏离值加上密钥，得到密文14 13 12 11 10

定义一个函数模板，名为max，参数包括数组和数组长度，返回值是数组中的最大值，要求支持整数、浮点数和字符三种类型。

用类模板定义一个加密类，包含四个属性：明文、密文、密钥、长度，前三个属性都是同一种类型，长度是整数。长度是指明文的长度。

类模板包含操作构造、加密、打印，说明如下：

1、加密是调用函数模板max得到数组最大值，按照前面的方法使用最大值和密钥进行加密，得到密文

2、打印是输出密文

要求类模板支持整数、浮点数和字符三种类型。

参考代码给出了加密类界面（只支持整数类型）、主函数（支持三种数据类型），程序要求

1、根据要求编写函数模板max

2、使用类模板方法改造加密类界面，不能增加任何属性和操作，必须在类外实现构造函数和加密方法

3、主函数不能有任何修改

上述所有类属性都不是public，用面向对象思想和C++语言实现上述要求

----参考代码----

```cpp
//只支持整数类型的加密类界面
class Cryption
{
private:
	int ptxt[100]; //明文
	int ctxt[100]; //密文
	int key; //密钥
	int len; //长度
public:
	Cryption(int tk, int tt[], int n); //参数依次对应密钥、明文、长度
	void encrypt(); //加密
	void print() //打印，无需改造
	{
		int i;
		for (i = 0; i < len - 1; i++)
		{
			cout << ctxt[i] << " ";
		}
		cout << ctxt[i] << endl;
	}
};

//支持三种类型的主函数
int main()
{
	int i;
	int length; //长度
	int ik, itxt[100];
	double dk, dtxt[100];
	char ck, ctxt[100];
	//整数加密
	cin >> ik >> length;
	for (i = 0; i < length; i++)
	{
		cin >> itxt[i];
	}
	Cryption<int> ic(ik, itxt, length);
	ic.encrypt();
	ic.print();
	//浮点数加密
	cin >> dk >> length;
	for (i = 0; i < length; i++)
	{
		cin >> dtxt[i];
	}
	Cryption<double> dc(dk, dtxt, length);
	dc.encrypt();
	dc.print();
	//字符加密
	cin >> ck >> length;
	for (i = 0; i < length; i++)
	{
		cin >> ctxt[i];
	}
	Cryption<char> cc(ck, ctxt, length);
	cc.encrypt();
	cc.print();

	return 0;
}
```

#### 输入格式

第一行输入整数类型的信息，包括密钥、长度、明文

第二行输入浮点数类型的信息，包括密钥、长度、明文

第三行输入字符类型的信息，包括密钥、长度、明文

#### 输出格式

三行分别输出三种类型的密文

#### 样例输入

```text
10 5 1 2 3 4 5
11.11 4 1.1 2.2 3.3 4.4
O 3 a b c
```

#### 样例输出

```text
14 13 12 11 10
14.41 13.31 12.21 11.11
Q P O
```

#### 解题思路与伪代码

**模板结构**

1. 定义函数模板 `max(array, length)`，从第一个元素开始比较并返回最大值。
2. 将 `Cryption` 改成类模板，明文数组、密文数组和密钥使用同一个模板类型，长度仍为整数。
3. 构造函数必须在类外实现：复制密钥、长度及全部明文元素。
4. `encrypt` 在类外实现：先调用函数模板取得最大明文，再逐项计算密文。

**加密公式**

```text
maximum = max(明文数组, 长度)
对每个 i：密文[i] = maximum - 明文[i] + 密钥
```

同一套模板会分别实例化为 `int`、`double`、`char`。字符参与加减时按字符编码运算，结果再存回字符类型，因此样例中的 `a b c`、密钥 `O` 会得到 `Q P O`。不要改动题目给出的 `main` 和打印逻辑。

---

### 18.4 四、加湿风扇（期末模拟）

> 题目 ID：`127` · 20 分 · 限制：1s / 128MB

#### 题目要求

已知家电有编号、功率的属性，属性都是整数，操作包括构造和打印等

电风扇继承家电的特点，新增两个属性（整数）：风向和风力，其中风向为0表示定向吹风，状态为1表示旋转吹风。

风扇包含两个新操作：风向控制和风力控制

1、风向控制含一个整数参数，无返回，把风向设置为参数值，参数为0表示定向吹风，为1表示旋转吹风。

2、风力控制含一个整数参数，无返回，把风力设置为参数值，参数表示风力级别，例如1级、2级、3级等。

加湿器继承家电的特点，新增两个属性（浮点数）：实际水容量和最大水容量

新增操作是预警，无参数，返回值为整数，当实际水容量不小于最大水容量的50%，则返回1；小于50%且不小于10%则返回2，小于10%则返回3

加湿风扇继承了风扇和加湿器的特点，新增属性档位（整数）

新增操作调整档位，含一个参数，无返回值，先设置档位为参数值，再调用风向控制和风力控制来设置相关属性，包括：

1、参数为0，不做其他属性修改

2、参数为1，设置定向吹风且风力1级

3、参数为2，设置旋转吹风且风力2级

4、参数为3，设置旋转吹风且风力3级

档位只可能是0、1、2、3四个数值，其他数值忽略。

加湿风扇重载打印操作，输出格式参考样本。输出要求如下：

1、如果风向为0，则输出定向吹风，风向为1则输出旋转吹风。

2、调用预警操作，并根据返回结果1、2、3输出不同信息，分别是：水量正常、水量偏低、水量不足

程序要求

1、采用虚拟继承机制实现上述电器的关系，明确谁是虚基类、基类、派生类

2、基类和派生类的构造要考虑虚基类、基类的属性构造

上述所有类属性都不是public，用面向对象思想和C++语言实现上述要求

#### 输入格式

第一行输入t，表示有t个实例

第二行输入一个加湿风扇的信息，依次包括编号、功率、风向、风力、实际水容量、最大水容量 档位

第三行输入一个参数，表示调档操作的档位，然后执行调档操作。

以此类推，输入t个实例

#### 输出格式

对于每个实例，调用打印操作输出加湿风扇的最终状态

#### 样例输入

```text
3
1001 1000 1 2 3 4 0
1
2002 2000 0 1 1 12 0
3
3003 3000 0 3 2 10 0
0
```

#### 样例输出

```text
加湿风扇--档位1
编号1001--功率1000W
定向吹风--风力1级
实际水容量3升--水量正常
加湿风扇--档位3
编号2002--功率2000W
旋转吹风--风力3级
实际水容量1升--水量不足
加湿风扇--档位0
编号3003--功率3000W
定向吹风--风力3级
实际水容量2升--水量偏低
```

#### 解题思路与伪代码

**继承结构**

```text
                 Appliance（家电）
                 /             \
      Fan（虚继承）          Humidifier（虚继承）
                 \             /
                  HumidifyingFan
```

- `Fan` 与 `Humidifier` 都应 `virtual public Appliance`，从而让最终对象只保留一份编号和功率。
- 最终派生类构造时由它直接初始化虚基类 `Appliance`，同时初始化两个直接基类和档位。
- 调档规则：0 只更新档位；1 设置定向、1 级；2 设置旋转、2 级；3 设置旋转、3 级；其他值忽略。

**水量预警**

```text
actual >= maximum × 0.5  -> 1（水量正常）
actual >= maximum × 0.1  -> 2（水量偏低）
否则                       -> 3（水量不足）
```

判断顺序必须从 50% 开始，否则区间容易重叠。档位为 0 时保留对象输入时已有的风向和风力。打印时严格按照样例的四行顺序输出。

---

### 18.5 五、计重转换（期末模拟）

> 题目 ID：`128` · 20 分 · 限制：1s / 128MB

#### 题目要求

目前国际计重最基本的单位是克。在古代各个国家的计重单位是不同的。

中国使用斤、两、钱来表示重量，其中1斤=10两，1两=10钱

中国计重单位与克的关系为：1斤=500克，1两=50克，1钱=5克

英国使用磅、盎司、打兰来表示重量，其中1磅=16盎司，1盎司=16打兰

英国计重单位与克的关系为：1磅=512克，1盎司=32克，1打兰=2克

以下参考代码包含了抽象类Weight，中国计重和英国计重都继承了抽象类。

中国计重类新增了斤、两、钱三个属性，并新增了一个操作：计重转换Convert。

Convert能够把输入的克数转成中国计重，例如1234克转成2斤4两6钱4克，并且把数值放入斤、两、钱、克四个属性中

英国计重类新增了磅、盎司、打兰三个属性，并新增了两个操作：

1、计重转换Convert，功能与上述类似，例如2345克转成4磅9盎司4打兰1克，并且把数值放入对应的四个属性中

2、计重等价，重载类型转换运算符，实现将英国计重类的对象转换成中国计重类的对象，例如英国计重类对象en（2磅2盎司11打兰1克）等价于（转换成）中国计重类对象cn（2斤2两2钱1克）。

程序要求如下

1、参考代码框架不能做任何修改，在要求的地方添加代码

2、主函数不能有任何修改

以上数据纯粹为题目设计，方便计算，实际换算数据是不同的。

上述所有类属性都不是public，用面向对象思想和C++语言实现上述要求

----参考代码----

```cpp
class CN; //提前声明
class EN; //提前声明

// 抽象类
class Weight
{
protected:
	char kind[20]; //计重类型
	int gram; // 克
public:
	Weight(const char tk[] = "no name", int tg = 0)
	{
		strcpy(kind, tk);
		gram = tg;
	}
	virtual void print(ostream& out) = 0; // 输出不同类型的计重信息
};

// 中国计重
class CN : public Weight
{
	// ....类定义自行编写
};

// 英国计重
class EN : public Weight
{
	// ....类定义自行编写
};

// 以全局函数方式重载输出运算符，代码3-5行....自行编写
// 重载函数包含两个参数：ostream流对象、Weight类对象，参数可以是对象或对象引用
// 重载函数必须调用参数Weight对象的print方法

// 主函数
int main()
{
	int tw;
	// 创建一个中国计重类对象cn
	// 构造参数对应斤、两、钱、克、类型，其中克和类型是对应基类属性gram和kind
	CN cn(0, 0, 0, 0, "中国计重");
	cin >> tw;
	cn.Convert(tw); // 把输入的克数转成中国计重
	cout << cn;

	// 创建英国计重类对象en
	// 构造参数对应磅、盎司、打兰、克、类型，其中克和类型是对应基类属性gram和kind
	EN en(0, 0, 0, 0, "英国计重");
	cin >> tw;
	en.Convert(tw); // 把输入的克数转成英国计重
	cout << en;
	cn = en; // 把英国计重转成中国计重
	cout << cn;
	return 0;
}
```

#### 输入格式

第一行输入一个克数，调用中国计重转换，把克数转成中国计重

第二行输入一个克数，调用英国计重转换，把克数转成英国计重，并调用计重等价把英国计重转成中国计重

#### 输出格式

根据主函数运行输出

#### 样例输入

```text
1234
2345
```

#### 样例输出

```text
中国计重:2斤4两6钱4克
英国计重:4磅9盎司4打兰1克
中国计重:4斤6两9钱0克
```

#### 解题思路与伪代码

**多态与转换设计**

- `Weight` 是抽象基类，保存类型名称和余下的克数，`print` 为纯虚函数。
- `CN` 保存斤、两、钱；`EN` 保存磅、盎司、打兰；两者分别实现 `Convert` 和 `print`。
- 全局输出运算符接收 `Weight&`（或 `const Weight&`，取决于 `print` 的签名），内部只调用虚函数 `print`，从而统一输出两种计重对象。

**中国计重分解**

```text
斤 = total / 500；total %= 500
两 = total / 50； total %= 50
钱 = total / 5；  克 = total % 5
```

**英国计重分解**

```text
磅 = total / 512；total %= 512
盎司 = total / 32；total %= 32
打兰 = total / 2； 克 = total % 2
```

英国对象转换为中国对象时，先把英国各字段还原成总克数：`磅×512 + 盎司×32 + 打兰×2 + 克`，再调用中国计重的分解逻辑。为适配题目中的 `cn = en`，在 `EN` 中定义到 `CN` 的类型转换运算符。

---