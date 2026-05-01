# Python 面向对象进阶：继承与多态

> **学习目标**
> 1. 理解继承的概念和意义
> 2. 掌握继承的基础语法
> 3. 学会复写（重写）父类方法
> 4. 掌握调用父类成员的方式
> 5. 理解多态的概念和实现

---

## 一、继承的基础语法

### 1.1 什么是继承？

继承是面向对象编程的三大特性之一（封装、继承、多态）。它的核心思想是：**子类可以复用父类的属性和方法**。

**通俗理解：** 想象你开了一家连锁店。总店有一套标准的运营流程（装修风格、员工制服、核心菜品）。新开的分店不需要从零开始，直接"继承"总店的这套东西，然后再根据当地口味加点特色菜。继承就是这个"直接拿过来用"的过程。

生活中的例子：
- 「猫」是「动物」的一种 → 猫继承了动物的基本特征（会呼吸、会吃东西、会睡觉）
- 「轿车」是「汽车」的一种 → 轿车继承了汽车的基本功能（能跑、有方向盘、有刹车）
- 「大学生」是「学生」的一种 → 大学生继承了学生的基本属性（有学号、要上课、要考试）
- 「微信支付」是「支付方式」的一种 → 继承了支付的基本操作（付款、退款、查账单）
- 「电动车」是「车」的一种 → 继承了车的基本结构（轮子、座椅、车灯），但动力系统不同

在代码层面，继承让我们避免重复编写相同的代码。**不用继承的话，每个类都要把相同代码写一遍，改一个地方要改 N 个文件，容易出错。**

---

#### 1.1.1 前置知识：object 基类

在 Python 中，**所有类都直接或间接继承自 `object` 类**。`object` 是 Python 中所有类的"老祖宗"。

```python
class Foo:
    pass

# 等价于
class Foo(object):
    pass

# 两种写法完全一样，Python 3 中可以省略 (object)
print(Foo.__bases__)  # 输出: (<class 'object'>,)
```

`object` 类提供了一些基础方法，所有类都自动拥有：
- `__init__()` —— 构造方法
- `__str__()` —— 定义 `print(obj)` 时显示的内容
- `__repr__()` —— 定义对象在解释器中的显示形式
- `__eq__()` —— 定义 `==` 比较行为
- `__hash__()` —— 定义哈希值（用于字典键、集合元素）

**为什么要知道这个？**
- 当你写 `class Dog: pass` 时，Dog 已经自动继承了 object 的所有方法
- 后面学的 `super().__init__()` 最终会调用到 object 的 `__init__`
- 理解 MRO（方法解析顺序）时，object 总是在链条最末端

### 1.2 基本语法

```python
class 父类名:
    # 父类的属性和方法
    pass

class 子类名(父类名):
    # 子类特有的属性和方法
    pass
```

**示例：**

```python
class Animal:
    """动物类（父类）"""

    def __init__(self, name: str, age: int):
        self.name = name          # 初始化实例属性：名字
        self.age = age            # 初始化实例属性：年龄

    def eat(self):
        print(f"{self.name} 正在吃东西")  # 吃东西的行为

    def sleep(self):
        print(f"{self.name} 正在睡觉")    # 睡觉的行为


class Cat(Animal):
    """猫类（子类），继承自 Animal"""
    pass  # pass 表示"什么都不做"，这里故意留空，演示纯继承的效果


# 创建 Cat 对象，传入名字和年龄
# 虽然 Cat 没有定义 __init__，但它继承了 Animal 的 __init__
cat = Cat("小花", 2)

# Cat 类没有定义任何方法，但它继承了 Animal 的所有方法
cat.eat()    # 输出: 小花 正在吃东西 —— 调用的是继承来的 eat 方法
cat.sleep()  # 输出: 小花 正在睡觉 —— 调用的是继承来的 sleep 方法

print(cat.name)  # 输出: 小花 —— 访问继承来的属性
print(cat.age)   # 输出: 2 —— 访问继承来的属性
```

**要点：**
- `Cat(Animal)` 表示 `Cat` 继承自 `Animal`，括号里写父类名
- 子类自动拥有父类的所有属性和方法，不需要重复写
- 子类可以在此基础上添加自己独有的内容
- 即使子类是空的（只有 `pass`），也能使用父类的一切

**新手常见错误：**

```python
# 错误 1：忘记在括号里写父类名
class Cat:  # 这不是继承，这是独立的类，没有 Animal 的任何功能
    pass

# 错误 2：子类定义了 __init__ 但忘记调用 super().__init__()
class Cat(Animal):
    def __init__(self, name, age, color):
        self.color = color  # 忘了初始化 name 和 age！
        # 应该先写 super().__init__(name, age)

cat = Cat("小花", 2, "橘色")
print(cat.name)  # 报错！AttributeError: 'Cat' object has no attribute 'name'

# 错误 3：把继承关系搞反了
class Animal(Cat):  # 错！应该是 Cat(Animal)，不是 Animal(Cat)
    pass
# 记忆方法：子类名(父类名)，"谁是谁的一种"，猫是动物的一种 → Cat(Animal)
```

---

### 继承基础小练习

**练习 1-A：** 下面代码的输出是什么？先想一想，再运行验证。

```python
class Phone:
    def __init__(self, brand):
        self.brand = brand

    def call(self):
        print(f"{self.brand} 手机正在打电话")

class iPhone(Phone):
    pass

my_phone = iPhone("Apple")
my_phone.call()  # 输出是什么？
print(my_phone.brand)  # 输出是什么？
```

**练习 1-B：** 创建一个 `Vehicle`（车辆）类，有 `brand`（品牌）和 `speed`（速度）属性，以及 `describe()` 方法。然后创建 `Bicycle`（自行车）子类，不添加任何新内容，测试它能否使用 `Vehicle` 的所有功能。

### 1.3 子类添加自己的属性和方法

```python
class Animal:
    """动物类（父类）"""

    def __init__(self, name: str, age: int):
        self.name = name          # 设置名字
        self.age = age            # 设置年龄

    def eat(self):
        print(f"{self.name} 正在吃东西")


class Cat(Animal):
    """猫类（子类）"""

    def __init__(self, name: str, age: int, color: str):
        # 第一步：调用父类的 __init__，把 name 和 age 交给父类处理
        # 这样 self.name 和 self.age 就被正确初始化了
        super().__init__(name, age)
        # 第二步：添加子类特有的属性（父类没有的）
        self.color = color        # 颜色是猫特有的属性

    # 添加子类特有的方法（父类没有的）
    def meow(self):
        print(f"{self.name} 喵喵叫~")


# 创建 Cat 对象，需要传入三个参数
cat = Cat("小花", 2, "橘色")

# 调用继承自父类的方法 —— Cat 类里没有定义 eat，但能用
cat.eat()      # 输出: 小花 正在吃东西

# 调用子类自己的方法 —— 只有 Cat 才有 meow
cat.meow()     # 输出: 小花 喵喵叫~

# 访问属性
print(cat.name)   # 继承自父类: 小花
print(cat.age)    # 继承自父类: 2
print(cat.color)  # 子类自己的: 橘色
```

**要点：**
- `super().__init__(name, age)` 用于调用父类的构造方法，初始化继承来的属性
- 子类可以在父类基础上添加新的属性和方法
- `super()` 是 Python 内置函数，返回父类的代理对象
- **写子类 `__init__` 时，第一行就应该调用 `super().__init__()`，养成习惯**

**通俗理解 `super()`：** 想象你在装修房子。`super().__init__()` 就是"先把爸爸的装修方案执行一遍"（铺地板、刷墙），然后你再在爸爸的基础上加自己的东西（装投影仪、买游戏机）。如果跳过爸爸的方案，地板都没铺，你的投影仪往哪装？

**新手常见错误：**

```python
# 错误：子类 __init__ 忘记调用 super().__init__()
class Cat(Animal):
    def __init__(self, name, age, color):
        # 忘了写 super().__init__(name, age)！
        self.color = color

cat = Cat("小花", 2, "橘色")
cat.eat()        # 报错！因为 self.name 从未被赋值
# AttributeError: 'Cat' object has no attribute 'name'

# 正确写法：
class Cat(Animal):
    def __init__(self, name, age, color):
        super().__init__(name, age)  # 先初始化父类的属性
        self.color = color           # 再添加自己的属性
```

---

### 子类扩展小练习

**练习 1-C：** 创建一个 `BankAccount`（银行账户）类，有 `owner`（户主）和 `balance`（余额）属性，以及 `deposit(amount)`（存款）方法。然后创建 `SavingsAccount`（储蓄账户）子类，添加 `interest_rate`（利率）属性和 `add_interest()`（计息）方法。

**练习 1-D：** 下面代码有什么问题？找出并修复。

```python
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade

class CollegeStudent(Student):
    def __init__(self, name, grade, major):
        self.major = major  # 这里有问题！

s = CollegeStudent("小明", "大一", "计算机")
print(s.name)  # 会报错吗？
```

### 1.4 多层继承

继承可以形成链条：爷爷 → 爸爸 → 儿子

```python
class Animal:
    """动物"""

    def __init__(self, name: str):
        self.name = name        # 最基础的属性：名字

    def breathe(self):
        print(f"{self.name} 正在呼吸")  # 最基础的行为：呼吸


class Dog(Animal):
    """狗，继承自 Animal"""

    def bark(self):
        print(f"{self.name} 汪汪叫")  # 狗特有的行为：叫


class Husky(Dog):
    """哈士奇，继承自 Dog"""

    def destroy_house(self):
        print(f"{self.name} 正在拆家")  # 哈士奇特有的行为：拆家


# 创建哈士奇对象
husky = Husky("二哈")

# 可以调用所有祖先类的方法 —— 哈士奇会呼吸（继承自Animal）、会叫（继承自Dog）、会拆家（自己的）
husky.breathe()        # 继承自 Animal: 二哈 正在呼吸
husky.bark()           # 继承自 Dog: 二哈 汪汪叫
husky.destroy_house()  # 自己的方法: 二哈 正在拆家
```

**继承链：** `Husky → Dog → Animal → object`

在 Python 中，所有类都直接或间接继承自 `object` 类（Python 的根类）。

**生活类比：** 多层继承就像家族传承。爷爷会做饭，爸爸继承了爷爷的厨艺并学会了开车，你继承了爸爸的厨艺和开车技能，又自己学会了编程。你会的东西是三代人能力的叠加。

**新手常见错误：**

```python
# 错误：想调用最顶层父类的方法，但跳过了中间层
class Husky(Dog):
    def __init__(self, name):
        # 如果 Dog 也有 __init__，应该先调用 Dog 的，而不是直接跳到 Animal
        Animal.__init__(self, name)  # 不推荐！跳过了 Dog 的初始化
        # 应该用 super().__init__(name) 来按 MRO 顺序调用
```

---

### 多层继承小练习

**练习 1-E：** 创建一个三层继承链：`Appliance`（电器）→ `KitchenAppliance`（厨房电器）→ `Blender`（搅拌机）。每层添加自己特有的属性和方法。测试搅拌机能否使用最顶层 `Appliance` 的方法。

### 1.5 查看继承关系

```python
class Animal:
    pass

class Dog(Animal):
    pass

class Husky(Dog):
    pass

# 查看类的继承链（MRO: Method Resolution Order，方法解析顺序）
# MRO 决定了 Python 查找方法时的顺序：从左到右，找到就停
print(Husky.__mro__)
# 输出: (<class 'Husky'>, <class 'Dog'>, <class 'Animal'>, <class 'object'>)
# 顺序：先找 Husky → 再找 Dog → 再找 Animal → 最后找 object

# 判断继承关系：issubclass(子类, 父类) → 是否是"子类-父类"关系
print(issubclass(Husky, Animal))   # True  —— 哈士奇是动物的一种
print(issubclass(Husky, Dog))      # True  —— 哈士奇是狗的一种
print(issubclass(Dog, Husky))      # False —— 狗不一定是哈士奇

# 判断实例关系：isinstance(对象, 类) → 对象是否是该类（或其子类）的实例
husky = Husky("二哈")
print(isinstance(husky, Husky))    # True  —— 是哈士奇
print(isinstance(husky, Dog))      # True  —— 也是狗（哈士奇是狗的子类）
print(isinstance(husky, Animal))   # True  —— 也是动物
```

**`issubclass` vs `isinstance` 的区别：**
- `issubclass(A, B)` —— 判断的是**类与类**之间的关系："A 是不是 B 的子类？"
- `isinstance(obj, A)` —— 判断的是**对象与类**之间的关系："obj 是不是 A 的实例？"

**生活类比：**
- `issubclass(哈士奇类, 狗类)` → "哈士奇这个品种是不是属于狗？" → 是
- `isinstance(我家那只二哈, 狗)` → "我家养的这只二哈是不是狗？" → 是

### 1.6 多重继承

Python 支持一个类同时继承多个父类：

```python
class Flyable:
    """会飞的"""

    def fly(self):
        print(f"{self.name} 正在飞")  # 注意：这里用到了 self.name，但 Flyable 自己没有 __init__


class Swimmable:
    """会游泳的"""

    def swim(self):
        print(f"{self.name} 正在游泳")  # 同样用到了 self.name


class Duck(Flyable, Swimmable):
    """鸭子，既会飞又会游泳
    括号里的顺序很重要：先 Flyable，后 Swimmable
    Python 会按照这个顺序查找方法
    """

    def __init__(self, name: str):
        self.name = name  # name 在 Duck 自己的 __init__ 中初始化


duck = Duck("唐老鸭")
duck.fly()    # 输出: 唐老鸭 正在飞   —— 从 Flyable 继承
duck.swim()   # 输出: 唐老鸭 正在游泳 —— 从 Swimmable 继承
```

**生活类比：** 多重继承就像一个人同时是"程序员"和"吉他手"。他既有写代码的能力，又有弹吉他技能。两种能力来自不同的"父类"，但都属于同一个人。

**括号里的顺序很重要：** `Duck(Flyable, Swimmable)` 和 `Duck(Swimmable, Flyable)` 不一样。如果两个父类有同名方法，Python 按括号里的顺序优先查找。你可以用 `Duck.__mro__` 查看实际的查找顺序。

**注意：** 多重继承可能导致方法冲突（菱形继承问题），实际开发中应谨慎使用，优先使用单继承。

---

### super() 在多重继承中的行为

当涉及多重继承时，`super()` 的行为比单继承复杂得多。它不是简单地调用"父类"的方法，而是按照 **MRO 顺序**调用"下一个类"的方法。

```python
class A:
    def __init__(self):
        print("A.__init__")
        super().__init__()  # 调用 MRO 中 A 的下一个类（这里是 object）

class B(A):
    def __init__(self):
        print("B.__init__")
        super().__init__()  # 调用 MRO 中 B 的下一个类（这里是 A）

class C(A):
    def __init__(self):
        print("C.__init__")
        super().__init__()  # 调用 MRO 中 C 的下一个类

class D(B, C):
    def __init__(self):
        print("D.__init__")
        super().__init__()  # 调用 MRO 中 D 的下一个类（这里是 B）

# 查看 MRO 顺序
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

d = D()
# 输出:
# D.__init__
# B.__init__
# C.__init__
# A.__init__
```

**关键理解：**
- `super()` 不是"调用父类的方法"，而是"调用 MRO 链中下一个类的方法"
- 在 `B` 中的 `super().__init__()` 不是调用 `A`，而是调用 MRO 中 B 后面的 `C`
- 这保证了菱形继承中每个类的 `__init__` 只被调用一次

**生活类比：** 想象一条流水线：D → B → C → A。每个环节做完自己的事，用 `super()` 把工作传给流水线的下一个环节，而不是"传给自己的上级"。流水线的顺序由 MRO 决定。

**新手建议：** 多重继承中 `super()` 的行为容易让人困惑。如果你的项目不需要多重继承，尽量用单继承，简单清晰。

---

### 多重继承小练习

**练习 1-F：** 创建一个 `FlyingFish`（飞鱼）类，同时继承 `Fish`（鱼）和 `Bird`（鸟）。`Fish` 有 `swim()` 方法，`Bird` 有 `fly()` 方法。测试飞鱼能否同时游泳和飞行。

**练习 1-G：** 思考下面代码的输出，先想一想再运行验证。

```python
class A:
    def hello(self):
        print("Hello from A")

class B(A):
    def hello(self):
        print("Hello from B")

class C(A):
    def hello(self):
        print("Hello from C")

class D(B, C):
    pass

d = D()
d.hello()  # 输出什么？为什么？
print(D.__mro__)  # 查看 MRO，理解查找顺序
```

---

## 二、复写父类成员和调用父类成员

### 2.1 什么是复写（Override）？

**复写**（也叫重写、覆盖）是指子类重新定义父类中已有的方法。当子类调用该方法时，会优先使用子类自己的版本。

**通俗理解：** 爸爸说"我们家的传统是过年吃饺子"。你长大后说"我们家过年改吃火锅了"。这就是复写 —— 你用新规则覆盖了旧规则。但爸爸家的规则还在，只是你这一支执行的是新规则。

```python
class Animal:
    def speak(self):
        print("动物发出声音")  # 父类的默认实现：通用的叫声


class Dog(Animal):
    # 复写父类的 speak 方法 —— 用狗的叫声替换通用叫声
    def speak(self):
        print("汪汪汪！")     # Dog 自己的实现


class Cat(Animal):
    # 复写父类的 speak 方法 —— 用猫的叫声替换通用叫声
    def speak(self):
        print("喵喵喵！")     # Cat 自己的实现


# 创建对象
dog = Dog()    # Dog 实例
cat = Cat()    # Cat 实例

# 调用 speak 方法 —— Python 找到的是子类自己定义的版本
dog.speak()  # 输出: 汪汪汪！（调用的是 Dog 类的 speak，不是 Animal 的）
cat.speak()  # 输出: 喵喵喵！（调用的是 Cat 类的 speak，不是 Animal 的）

# 如果 Cat 没有复写 speak，就会用 Animal 的默认版本
# 复写 = "我不要父类的实现，我要自己写一个"
```

**要点：**
- 复写只需要在子类中定义同名方法即可，不需要任何特殊关键字
- 子类对象调用方法时，Python 会按照 MRO 顺序查找，优先使用最近的定义
- 复写是多态的基础 —— 同一个方法名，不同类有不同的行为
- **复写不影响父类本身** —— Dog 复写了 speak，但 Animal 的 speak 还是原来的

**新手常见错误：**

```python
# 错误：方法名写错，以为复写了其实没有
class Dog(Animal):
    def Speak(self):  # 大写了 S！Python 区分大小写
        print("汪汪汪！")

dog = Dog()
dog.speak()  # 输出: 动物发出声音 —— 调用的是 Animal 的 speak，不是 Dog 的 Speak

# 错误：方法签名（参数）不一致
class Animal:
    def speak(self, volume):  # 父类有两个参数
        print(f"动物发出声音，音量{volume}")

class Dog(Animal):
    def speak(self):  # 子类只有一个参数！参数数量不匹配
        print("汪汪汪！")

# 这种情况下，如果调用 dog.speak(10) 会报错，因为 Dog 的 speak 不接受参数
```

### 2.2 复写的查找顺序（MRO）

```python
class A:
    def show(self):
        print("A.show")

class B(A):
    def show(self):
        print("B.show")

class C(B):
    pass  # C 没有复写 show，所以会继续向上查找

class D(C):
    def show(self):
        print("D.show")


# 查看 MRO —— 方法查找顺序
print(D.__mro__)
# (<class 'D'>, <class 'C'>, <class 'B'>, <class 'A'>, <class 'object'>)

# 情况 1：D 有自己的 show → 直接用 D 的
d = D()
d.show()  # 输出: D.show（D 类自己有定义，不会继续找）

# 情况 2：C 没有 show → 沿着 MRO 向上找，找到 B 的 show
c = C()
c.show()  # 输出: B.show（C 没有，向上找到 B）
```

**查找规则：** Python 按照 MRO 列表从左到右查找，找到第一个定义该方法的类就使用它。就像问路：先问 D，D 说"我不知道"，再问 C，C 也说"我不知道"，再问 B，B 说"我知道！"，就用 B 的答案。

**生活类比：** 你问一个问题，先问你自己 → 你不知道 → 问你爸 → 你爸也不知道 → 问你爷爷 → 爷爷知道。找到答案就停，不会继续往上问。

### 2.3 调用父类方法的方式

有时候，我们复写父类方法，不是要完全替换它，而是想在父类方法的基础上**增加新功能**。这时就需要先调用父类的原方法。

#### 方式一：`super()` 调用（推荐）

```python
class Animal:
    def __init__(self, name: str, age: int):
        self.name = name    # 初始化名字
        self.age = age      # 初始化年龄

    def info(self):
        print(f"名字: {self.name}, 年龄: {self.age}")


class Dog(Animal):
    def __init__(self, name: str, age: int, breed: str):
        # 用 super() 调用父类的 __init__，把 name 和 age 交给父类处理
        super().__init__(name, age)
        # 子类自己处理 breed（品种）属性
        self.breed = breed

    def info(self):
        # 先调用父类的 info 方法，打印基本信息
        super().info()
        # 在父类基础上添加新内容
        print(f"品种: {self.breed}")


dog = Dog("旺财", 3, "金毛")
dog.info()
# 输出:
# 名字: 旺财, 年龄: 3        ← 这是 super().info() 输出的
# 品种: 金毛                   ← 这是 Dog 自己添加的
```

#### 方式二：显式指定父类名调用

```python
class Dog(Animal):
    def __init__(self, name: str, age: int, breed: str):
        # 显式指定父类名来调用 —— 需要手动传 self
        Animal.__init__(self, name, age)
        self.breed = breed
```

**两种方式的区别：**
- `super()` 会按照 MRO 顺序自动查找父类，推荐使用。不需要传 `self`，会自动处理
- 显式指定父类名会硬编码调用某个特定类，在多重继承时可能有问题（可能跳过某个类的初始化）

#### 方式三：`super()` 的其他用法 —— 获取父类方法的返回值

```python
class Base:
    def greet(self):
        return "Hello from Base"

class Child(Base):
    def greet(self):
        # 先获取父类方法的返回值，存到变量里
        parent_greet = super().greet()
        # 在父类返回值的基础上做加工
        return f"{parent_greet} and Child"


child = Child()
print(child.greet())
# 输出: Hello from Base and Child
# 父类说 "Hello from Base"，子类加上 "and Child"
```

**要点：** `super()` 不只能调用 void 方法，也能拿到返回值，像普通函数调用一样用就行。

### 2.4 复写属性

除了方法，属性也可以被复写：

```python
class Config:
    """配置类 —— 定义默认配置"""
    max_connections = 100   # 默认最大连接数
    timeout = 30            # 默认超时时间

    def show(self):
        print(f"最大连接数: {self.max_connections}, 超时: {self.timeout}s")


class ProductionConfig(Config):
    """生产环境配置 —— 需要更高的性能"""
    # 复写类属性：把默认值替换为生产环境的值
    max_connections = 1000
    timeout = 60


class DevelopmentConfig(Config):
    """开发环境配置 —— 不需要太高，省资源"""
    # 复写类属性：把默认值替换为开发环境的值
    max_connections = 10
    timeout = 5


# 不同环境使用不同配置
config = ProductionConfig()
config.show()  # 输出: 最大连接数: 1000, 超时: 60s —— 使用生产环境的值

dev_config = DevelopmentConfig()
dev_config.show()  # 输出: 最大连接数: 10, 超时: 5s —— 使用开发环境的值

# 注意：Config 本身的值没有被改变
default = Config()
default.show()  # 输出: 最大连接数: 100, 超时: 30s —— 还是默认值
```

**生活类比：** 公司规定"上班时间 9:00"（父类属性）。分公司可以复写这个规定："我们上班时间 10:00"（子类属性复写）。总部的规定没变，只是这个分公司执行自己的规则。

---

### 复写小练习

**练习 2-A：** 创建一个 `Vehicle` 基类，有 `max_speed` 属性和 `describe()` 方法。创建 `SportsCar`（跑车）和 `Truck`（卡车）子类，分别复写 `max_speed` 属性和 `describe()` 方法。

**练习 2-B：** 下面代码有什么问题？

```python
class Parent:
    def do_something(self, x):
        print(f"Parent: {x}")

class Child(Parent):
    def do_something(self):  # 注意参数
        super().do_something()  # 这里会报错吗？
        print("Child done")
```

### 2.5 完整示例：游戏角色系统

```python
class Character:
    """游戏角色基类"""

    def __init__(self, name: str, hp: int, attack: int):
        self.name = name
        self.hp = hp
        self.attack = attack
        self.is_alive = True

    def take_damage(self, damage: int):
        """受到伤害"""
        self.hp -= damage
        if self.hp <= 0:
            self.hp = 0
            self.is_alive = False
            print(f"{self.name} 已阵亡！")
        else:
            print(f"{self.name} 受到 {damage} 点伤害，剩余 HP: {self.hp}")

    def attack_enemy(self, enemy):
        """攻击敌人"""
        if not self.is_alive:
            print(f"{self.name} 已阵亡，无法攻击！")
            return
        print(f"{self.name} 攻击 {enemy.name}，造成 {self.attack} 点伤害")
        enemy.take_damage(self.attack)

    def status(self):
        """显示状态"""
        status = "存活" if self.is_alive else "阵亡"
        print(f"[{self.name}] HP: {self.hp}, 状态: {status}")


class Warrior(Character):
    """战士，继承自 Character"""

    def __init__(self, name: str, hp: int, attack: int, armor: int):
        super().__init__(name, hp, attack)
        self.armor = armor

    # 复写 take_damage 方法，加入护甲减伤
    def take_damage(self, damage: int):
        reduced_damage = max(1, damage - self.armor)  # 护甲减伤，最少受到1点伤害
        print(f"护甲减免了 {damage - reduced_damage} 点伤害")
        # 调用父类的 take_damage
        super().take_damage(reduced_damage)

    # 战士特有的技能
    def shield_bash(self, enemy):
        """盾击：造成攻击伤害并眩晕"""
        if not self.is_alive:
            return
        print(f"{self.name} 使用盾击攻击 {enemy.name}！")
        enemy.take_damage(self.attack + 5)


class Mage(Character):
    """法师，继承自 Character"""

    def __init__(self, name: str, hp: int, attack: int, mana: int):
        super().__init__(name, hp, attack)
        self.mana = mana

    # 复写 attack_enemy 方法，法师使用魔法攻击
    def attack_enemy(self, enemy):
        if not self.is_alive:
            print(f"{self.name} 已阵亡，无法攻击！")
            return
        magic_damage = self.attack * 2  # 魔法伤害是物理攻击的2倍
        print(f"{self.name} 对 {enemy.name} 施放魔法，造成 {magic_damage} 点魔法伤害")
        enemy.take_damage(magic_damage)

    # 法师特有的技能
    def heal(self):
        """治疗术"""
        heal_amount = 30
        self.hp += heal_amount
        print(f"{self.name} 使用治疗术，恢复 {heal_amount} 点 HP，当前 HP: {self.hp}")


# 测试
warrior = Warrior("亚瑟", 100, 20, 5)
mage = Mage("甘道夫", 80, 15, 100)

warrior.status()
mage.status()

print("\n--- 战斗开始 ---\n")

warrior.attack_enemy(mage)
mage.attack_enemy(warrior)
warrior.shield_bash(mage)
mage.heal()

print("\n--- 战斗结束 ---\n")

warrior.status()
mage.status()
```

**输出结果：**
```
[亚瑟] HP: 100, 状态: 存活
[甘道夫] HP: 80, 状态: 存活

--- 战斗开始 ---

亚瑟 攻击 甘道夫，造成 20 点伤害
甘道夫 受到 20 点伤害，剩余 HP: 60
甘道夫 对 亚瑟 施放魔法，造成 30 点魔法伤害
护甲减免了 5 点伤害
亚瑟 受到 25 点伤害，剩余 HP: 75
亚瑟 使用盾击攻击 甘道夫！
甘道夫 受到 25 点伤害，剩余 HP: 35
甘道夫 使用治疗术，恢复 30 点 HP，当前 HP: 65

--- 战斗结束 ---

[亚瑟] HP: 75, 状态: 存活
[甘道夫] HP: 65, 状态: 存活
```

**代码分析：**
- `Warrior` 复写了 `take_damage`，加入了护甲减伤逻辑，并用 `super()` 调用父类的原方法
- `Mage` 复写了 `attack_enemy`，改为魔法伤害，并添加了 `heal` 技能
- 子类在复写时，可以在父类方法的基础上增加功能，而不是完全替换

---

## 三、多态

### 3.1 什么是多态？

**多态**（Polymorphism）是指同一个方法在不同类中有不同的实现。调用者不需要知道对象的具体类型，只需要调用同名方法，不同对象会表现出不同的行为。

**通俗理解：** 你去餐厅点菜，说"来一份主食"。服务员不需要知道你具体要米饭、面条还是馒头 —— 他只需要知道"主食"这个接口。不同的主食有不同的做法，但对外的称呼都是"主食"。这就是多态：同一个请求，不同对象有不同的响应。

生活中的例子：
- 同样是「叫」，猫叫「喵」，狗叫「汪」，鸭叫「嘎」 —— 都是"叫"，但每种动物叫法不同
- 同样是「移动」，鱼用游，鸟用飞，人用走 —— 都是"移动"，但方式不同
- 同样是「支付」，支付宝扫码、微信扫码、刷卡 —— 都是"支付"，但流程不同
- 同样是「保存」，保存到本地文件、保存到数据库、保存到云盘 —— 都是"保存"，但目标不同
- 同样是「打开」，打开 Word 文档、打开 PDF、打开图片 —— 都是"打开"，但程序不同

### 3.2 多态的实现

```python
class Animal:
    def speak(self):
        # raise NotImplementedError 表示"这个方法必须被子类实现"
        # 如果子类忘记实现，调用时会报错，提醒开发者
        raise NotImplementedError("子类必须实现 speak 方法")


class Dog(Animal):
    def speak(self):
        return "汪汪汪！"    # 狗的叫声


class Cat(Animal):
    def speak(self):
        return "喵喵喵！"    # 猫的叫声


class Duck(Animal):
    def speak(self):
        return "嘎嘎嘎！"    # 鸭的叫声


# 多态的体现：同一个函数，处理不同类型的对象
def animal_speak(animal: Animal):
    """让动物叫 —— 不关心具体是什么动物，只要会 speak 就行"""
    print(animal.speak())  # 调用传入对象的 speak 方法


# 传入不同类型的对象 —— 函数代码完全一样，但输出不同
dog = Dog()      # 创建狗
cat = Cat()      # 创建猫
duck = Duck()    # 创建鸭

animal_speak(dog)   # 输出: 汪汪汪！ —— 调用 Dog 的 speak
animal_speak(cat)   # 输出: 喵喵喵！ —— 调用 Cat 的 speak
animal_speak(duck)  # 输出: 嘎嘎嘎！ —— 调用 Duck 的 speak

# 甚至可以传入列表，批量处理
animals = [Dog(), Cat(), Duck()]
for animal in animals:
    animal.speak()  # 每个动物用自己版本的 speak
```

**关键点：**
- `animal_speak` 函数的参数类型是 `Animal`，表示"任何动物都行"
- 传入 `Dog`、`Cat`、`Duck` 对象都可以正常工作
- 函数不需要知道具体是哪个子类，只调用 `speak` 方法
- **新增动物类型时，只需要创建新子类实现 `speak`，不需要修改 `animal_speak` 函数**

**新手常见错误：**

```python
# 错误：子类忘记实现抽象方法
class Cat(Animal):
    pass  # 没有实现 speak！

cat = Cat()
cat.speak()  # 报错！NotImplementedError: 子类必须实现 speak 方法
# 这正是 raise NotImplementedError 的好处 —— 提醒你忘了实现

# 错误：方法名拼写不一致
class Dog(Animal):
    def bark(self):  # 写成了 bark，不是 speak！
        return "汪汪汪！"

dog = Dog()
animal_speak(dog)  # 报错！因为 Dog 没有 speak 方法，只有 bark
```

### 3.3 多态的实际应用

```python
from abc import ABC, abstractmethod
# ABC = Abstract Base Class（抽象基类）
# abstractmethod = 装饰器，标记"这个方法必须被子类实现"


class Shape(ABC):
    """形状抽象基类 —— 不能直接创建 Shape 对象，只能创建它的子类"""

    @abstractmethod
    def area(self) -> float:
        """计算面积（抽象方法，子类必须实现）"""
        pass  # 没有具体实现，留给子类

    @abstractmethod
    def perimeter(self) -> float:
        """计算周长（抽象方法，子类必须实现）"""
        pass

    def description(self) -> str:
        """描述形状（普通方法，子类可以选择性复写）"""
        return f"这是一个形状，面积={self.area():.2f}，周长={self.perimeter():.2f}"


class Circle(Shape):
    """圆形 —— 必须实现 area 和 perimeter"""

    def __init__(self, radius: float):
        self.radius = radius    # 半径

    def area(self) -> float:
        return 3.14159 * self.radius ** 2     # 圆面积 = π × r²

    def perimeter(self) -> float:
        return 2 * 3.14159 * self.radius      # 圆周长 = 2 × π × r


class Rectangle(Shape):
    """矩形 —— 必须实现 area 和 perimeter"""

    def __init__(self, width: float, height: float):
        self.width = width      # 宽
        self.height = height    # 高

    def area(self) -> float:
        return self.width * self.height        # 矩形面积 = 宽 × 高

    def perimeter(self) -> float:
        return 2 * (self.width + self.height)  # 矩形周长 = 2 × (宽 + 高)


class Triangle(Shape):
    """三角形（已知三边长度）"""

    def __init__(self, a: float, b: float, c: float):
        self.a = a    # 边 a
        self.b = b    # 边 b
        self.c = c    # 边 c

    def area(self) -> float:
        # 海伦公式：已知三边求面积
        s = self.perimeter() / 2  # 半周长
        return (s * (s - self.a) * (s - self.b) * (s - self.c)) ** 0.5

    def perimeter(self) -> float:
        return self.a + self.b + self.c  # 三边之和


# 多态应用：统一处理不同形状
def print_shape_info(shape: Shape):
    """打印形状信息 —— 不关心具体是哪种形状"""
    print(shape.description())  # 调用对应形状的 area 和 perimeter


# 创建不同形状
shapes = [
    Circle(5),           # 半径 5 的圆
    Rectangle(4, 6),     # 宽 4 高 6 的矩形
    Triangle(3, 4, 5)    # 三边 3, 4, 5 的三角形
]

# 统一处理 —— 同一个循环，不同形状用各自的公式计算
for shape in shapes:
    print_shape_info(shape)

# 输出:
# 这是一个形状，面积=78.54，周长=31.42
# 这是一个形状，面积=24.00，周长=20.00
# 这是一个形状，面积=6.00，周长=12.00

# 错误示例：不能创建抽象类的实例
# shape = Shape()  # 报错！TypeError: Can't instantiate abstract class Shape
# 因为 Shape 的 area 和 perimeter 是抽象方法，没有具体实现
```

**抽象基类 vs 普通类：**

| 特性 | 普通类 | 抽象基类（ABC） |
|------|--------|-----------------|
| 能否直接创建实例 | 能 | 不能 |
| 抽象方法 | 没有这个概念 | 子类必须实现，否则报错 |
| 用途 | 通用类 | 定义"规范"或"接口" |

**生活类比：** 抽象基类就像"行业标准"。比如"食品标准"规定了所有食品必须标注"生产日期、保质期、配料表"。你不能直接买一个"食品标准"，但所有具体食品（面包、牛奶、饼干）都必须满足这个标准。

### 3.4 鸭子类型（Duck Typing）

Python 的多态不依赖于继承关系，而是依赖于对象是否具有某个方法。这就是**鸭子类型**（Duck Typing）：

> "如果它走起来像鸭子，叫起来像鸭子，那它就是鸭子。"
> —— 意思是：Python 不关心你是什么类型，只关心你能不能做这件事。

```python
class Dog:
    def speak(self):
        return "汪汪汪！"    # Dog 有 speak 方法

class Cat:
    def speak(self):
        return "喵喵喵！"    # Cat 有 speak 方法

class Car:
    def speak(self):
        return "滴滴滴！"    # Car 也有 speak 方法（喇叭声）


# 这三个类没有任何继承关系（没有共同父类），但都可以传入这个函数
def make_sound(obj):
    # Python 不检查 obj 是什么类型，只检查 obj 有没有 speak 方法
    print(obj.speak())


make_sound(Dog())   # 汪汪汪！ —— Dog 有 speak，OK
make_sound(Cat())   # 喵喵喵！ —— Cat 有 speak，OK
make_sound(Car())   # 滴滴滴！ —— Car 有 speak，OK

# 如果传入一个没有 speak 方法的对象：
class Table:
    pass

# make_sound(Table())  # 报错！AttributeError: 'Table' object has no attribute 'speak'
# 因为 Table 没有 speak 方法
```

**要点：**
- Python 不检查对象的类型，只检查对象是否有对应的方法
- 这和 C++/Java 的多态不同，Python 更灵活 —— 不需要继承关系也能实现多态
- 只要有同名方法，就可以实现多态

**生活类比：** 你要找人帮你"翻译"。你不关心对方是老师、学生还是 AI，只要他能翻译就行。这就是鸭子类型 —— 不看身份，看能力。

**鸭子类型 vs 继承多态：**

| 特性 | 继承多态 | 鸭子类型 |
|------|----------|----------|
| 是否需要继承关系 | 需要 | 不需要 |
| 是否需要共同父类 | 需要 | 不需要 |
| 灵活性 | 较低 | 很高 |
| 安全性 | 较高（有类型检查） | 较低（运行时才发现错误） |
| 适用场景 | 大型项目，需要严格规范 | 脚本、快速开发 |

### 3.5 `isinstance` 与多态

有时候我们需要根据对象类型执行不同逻辑：

```python
class Notification:
    """通知基类"""
    def send(self, message: str):
        raise NotImplementedError  # 子类必须实现

class EmailNotification(Notification):
    """邮件通知"""
    def send(self, message: str):
        print(f"发送邮件: {message}")  # 邮件发送逻辑

class SMSNotification(Notification):
    """短信通知"""
    def send(self, message: str):
        print(f"发送短信: {message}")  # 短信发送逻辑

class PushNotification(Notification):
    """推送通知"""
    def send(self, message: str):
        print(f"发送推送: {message}")  # 推送发送逻辑


def send_notification(notification: Notification, message: str):
    """发送通知 —— 根据类型做额外处理"""
    # isinstance 判断对象类型，执行不同的预处理逻辑
    if isinstance(notification, EmailNotification):
        print("正在通过邮件渠道发送...")
    elif isinstance(notification, SMSNotification):
        print("正在通过短信渠道发送...")
    elif isinstance(notification, PushNotification):
        print("正在通过推送渠道发送...")

    # 最终都调用 send 方法 —— 这部分是多态
    notification.send(message)


# 测试
send_notification(EmailNotification(), "你的订单已发货")
# 输出: 正在通过邮件渠道发送...
#       发送邮件: 你的订单已发货

send_notification(SMSNotification(), "验证码: 123456")
# 输出: 正在通过短信渠道发送...
#       发送短信: 验证码: 123456

send_notification(PushNotification(), "你有一条新消息")
# 输出: 正在通过推送渠道发送...
#       发送推送: 你有一条新消息
```

**注意：** 过度使用 `isinstance` 判断类型会破坏多态的优势。大多数情况下，应该让子类自己处理差异，而不是在外部判断类型。上面的例子中，"渠道发送"的逻辑其实可以放到子类的 `send` 方法里，这样就不需要 `isinstance` 判断了。

**什么时候用 `isinstance`？**
- 需要根据类型做**完全不同的操作**时（不是同名方法的不同实现）
- 处理外部传入的数据，需要**安全检查**时
- 调试和日志记录时

**什么时候不用 `isinstance`？**
- 同一个方法在不同类中有不同实现 → 用多态
- 子类自己能处理差异 → 让子类处理

### 3.6 综合示例：支付系统

```python
from abc import ABC, abstractmethod
from datetime import datetime


class Payment(ABC):
    """支付方式抽象基类 —— 定义了所有支付方式必须具备的能力"""

    @abstractmethod
    def pay(self, amount: float) -> bool:
        """支付，返回是否成功 —— 每种支付方式必须实现"""
        pass

    @abstractmethod
    def refund(self, amount: float) -> bool:
        """退款，返回是否成功 —— 每种支付方式必须实现"""
        pass

    def get_receipt(self, amount: float) -> str:
        """获取收据 —— 默认实现，子类可以复写"""
        return f"支付金额: ¥{amount:.2f}, 时间: {datetime.now()}"


class Alipay(Payment):
    """支付宝 —— 实现了 pay 和 refund"""

    def __init__(self, account: str):
        self.account = account    # 支付宝账号

    def pay(self, amount: float) -> bool:
        # 支付宝的具体支付逻辑（这里简化为打印）
        print(f"支付宝 ({self.account}) 支付 ¥{amount:.2f}")
        return True  # 模拟支付成功

    def refund(self, amount: float) -> bool:
        print(f"支付宝 ({self.account}) 退款 ¥{amount:.2f}")
        return True


class WechatPay(Payment):
    """微信支付 —— 实现了 pay 和 refund"""

    def __init__(self, openid: str):
        self.openid = openid      # 微信 openid

    def pay(self, amount: float) -> bool:
        print(f"微信支付 ({self.openid}) 支付 ¥{amount:.2f}")
        return True

    def refund(self, amount: float) -> bool:
        print(f"微信支付 ({self.openid}) 退款 ¥{amount:.2f}")
        return True


class CreditCard(Payment):
    """信用卡 —— 实现了 pay 和 refund，并复写了 get_receipt"""

    def __init__(self, card_number: str, holder: str):
        self.card_number = card_number  # 卡号
        self.holder = holder            # 持卡人

    def pay(self, amount: float) -> bool:
        print(f"信用卡 ({self.card_number}) 支付 ¥{amount:.2f}")
        return True

    def refund(self, amount: float) -> bool:
        print(f"信用卡 ({self.card_number}) 退款 ¥{amount:.2f}")
        return True

    def get_receipt(self, amount: float) -> str:
        """复写父类方法，添加信用卡特有信息（持卡人）"""
        base_receipt = super().get_receipt(amount)  # 先获取父类的收据内容
        return f"{base_receipt}\n持卡人: {self.holder}"  # 再加上持卡人信息


# 多态应用：统一的支付处理函数
def process_payment(payment: Payment, amount: float):
    """处理支付 —— 不关心具体是哪种支付方式"""
    print(f"\n--- 开始支付 ---")
    if payment.pay(amount):                    # 调用对应支付方式的 pay 方法
        print(payment.get_receipt(amount))     # 获取收据（信用卡会多一行持卡人）
        print("支付成功！")
    else:
        print("支付失败！")


def process_refund(payment: Payment, amount: float):
    """处理退款 —— 不关心具体是哪种支付方式"""
    print(f"\n--- 开始退款 ---")
    if payment.refund(amount):                 # 调用对应支付方式的 refund 方法
        print("退款成功！")
    else:
        print("退款失败！")


# 测试 —— 创建不同支付方式的对象
alipay = Alipay("user@example.com")
wechat = WechatPay("wxid_123456")
credit_card = CreditCard("6222-xxxx-xxxx-1234", "张三")

# 统一处理不同支付方式 —— 同一个函数，不同的行为
process_payment(alipay, 99.99)        # 用支付宝支付
process_payment(wechat, 199.50)       # 用微信支付
process_payment(credit_card, 299.00)  # 用信用卡支付

process_refund(alipay, 50.00)         # 用支付宝退款
```

**输出结果：**

```
--- 开始支付 ---
支付宝 (user@example.com) 支付 ¥99.99
支付金额: ¥99.99, 时间: 2026-04-29 22:00:00  # 时间随运行时间变化
支付成功！

--- 开始支付 ---
微信支付 (wxid_123456) 支付 ¥199.50
支付金额: ¥199.50, 时间: 2026-04-29 22:00:00
支付成功！

--- 开始支付 ---
信用卡 (6222-xxxx-xxxx-1234) 支付 ¥299.00
支付金额: ¥299.00, 时间: 2026-04-29 22:00:00
持卡人: 张三
支付成功！

--- 开始退款 ---
支付宝 (user@example.com) 退款 ¥50.00
退款成功！
```

**设计要点：**
- `process_payment` 和 `process_refund` 函数不关心具体的支付方式
- 只要传入的对象是 `Payment` 的子类，就能正常工作
- 新增支付方式时，只需创建新的子类，不需要修改处理函数
- **这就是多态的核心价值：代码对扩展开放，对修改关闭（开闭原则）**

### 3.7 多态的更多实际应用场景

多态在实际开发中无处不在，以下是几个常见场景：

**场景一：日志系统**
```python
class Logger:
    def log(self, message: str):
        raise NotImplementedError

class ConsoleLogger(Logger):
    """输出到控制台"""
    def log(self, message: str):
        print(f"[控制台] {message}")

class FileLogger(Logger):
    """写入文件"""
    def __init__(self, filename: str):
        self.filename = filename

    def log(self, message: str):
        with open(self.filename, "a") as f:
            f.write(f"{message}\n")

class RemoteLogger(Logger):
    """发送到远程服务器"""
    def log(self, message: str):
        print(f"[远程] 发送到服务器: {message}")

# 使用：程序运行时可以切换日志输出方式，不需要修改业务代码
def do_something(logger: Logger):
    logger.log("开始执行任务")
    # ... 业务逻辑
    logger.log("任务完成")
```

**场景二：数据导出**
```python
class Exporter:
    def export(self, data: list) -> str:
        raise NotImplementedError

class CSVExporter(Exporter):
    def export(self, data: list) -> str:
        # 转成 CSV 格式
        return "\n".join([",".join(row) for row in data])

class JSONExporter(Exporter):
    def export(self, data: list) -> str:
        # 转成 JSON 格式
        import json
        return json.dumps(data)

class HTMLExporter(Exporter):
    def export(self, data: list) -> str:
        # 转成 HTML 表格
        rows = "".join([f"<tr>{''.join(f'<td>{c}</td>' for c in row)}</tr>" for row in data])
        return f"<table>{rows}</table>"

# 使用：同一个数据，用户选择导出格式
def save_report(data: list, exporter: Exporter):
    content = exporter.export(data)  # 不关心具体格式，调用 export 就行
    print(content)
```

**场景三：数据库连接（Python 标准库中的真实例子）**
```python
# Python 的数据库接口规范 DB-API 2.0 就是用多态设计的
# 不同数据库（SQLite、MySQL、PostgreSQL）都实现了相同的接口

import sqlite3

# 连接对象都有 cursor()、commit()、close() 方法
conn = sqlite3.connect(":memory:")
cursor = conn.cursor()
cursor.execute("CREATE TABLE test (id INTEGER, name TEXT)")
cursor.execute("INSERT INTO test VALUES (1, 'Alice')")
conn.commit()
conn.close()

# 如果换成 MySQL，只需要改连接方式，后面的 cursor、execute、commit 代码完全一样
# 这就是多态的好处：换数据库不需要改业务代码
```

**场景四：游戏中的"策略模式"**
```python
class AttackStrategy:
    def execute(self, attacker, target):
        raise NotImplementedError

class MeleeAttack(AttackStrategy):
    """近战攻击"""
    def execute(self, attacker, target):
        damage = attacker.attack * 1.5
        print(f"{attacker.name} 近战攻击 {target.name}，造成 {damage} 伤害")
        target.hp -= damage

class RangedAttack(AttackStrategy):
    """远程攻击"""
    def execute(self, attacker, target):
        damage = attacker.attack * 0.8
        print(f"{attacker.name} 远程攻击 {target.name}，造成 {damage} 伤害")
        target.hp -= damage

class MagicAttack(AttackStrategy):
    """魔法攻击"""
    def execute(self, attacker, target):
        damage = attacker.attack * 2.0
        print(f"{attacker.name} 魔法攻击 {target.name}，造成 {damage} 伤害")
        target.hp -= damage

# 使用：角色可以在运行时切换攻击策略
# hero.attack_strategy = MeleeAttack()   # 换成近战
# hero.attack_strategy = MagicAttack()   # 换成魔法
```

**总结：多态的应用模式**
- **策略模式**：同一操作，不同实现（支付、攻击、排序算法）
- **工厂模式**：根据条件创建不同类型的对象
- **插件系统**：主程序定义接口，插件实现具体功能
- **适配器模式**：统一不同接口，让它们可以互换使用

---

## 四、本章小结

### 继承
- 继承让子类复用父类的代码，减少重复
- `super()` 用于调用父类方法，推荐使用
- Python 支持多层继承（A → B → C）和多重继承（C 同时继承 A 和 B）
- `issubclass()` 判断类与类的关系，`isinstance()` 判断对象与类的关系
- 所有类最终都继承自 `object`（Python 的根类）
- 括号里的顺序在多重继承中很重要，决定了 MRO

### 复写
- 子类可以重新定义父类的方法（同名方法覆盖），不需要特殊关键字
- 复写时可以用 `super()` 调用父类的原方法，在父类基础上增加功能
- Python 按照 MRO 顺序查找方法，找到第一个就停
- 复写不影响父类本身，只影响子类及其后代

### 多态
- 同一个方法在不同类中有不同实现
- 调用者不需要知道对象的具体类型，只调用同名方法
- Python 的多态基于鸭子类型，更灵活 —— 不需要继承关系
- 抽象基类（ABC）可以强制子类实现特定方法，适合定义"规范"
- 多态的实际应用：支付系统、图形计算、文件处理、通知系统等

### 新手速查表

| 语法 | 含义 | 示例 |
|------|------|------|
| `class B(A)` | B 继承 A | `class Dog(Animal)` |
| `super().__init__()` | 调用父类构造方法 | `super().__init__(name)` |
| `super().method()` | 调用父类方法 | `super().info()` |
| `issubclass(B, A)` | B 是不是 A 的子类 | `issubclass(Dog, Animal)` → True |
| `isinstance(obj, A)` | obj 是不是 A 的实例 | `isinstance(dog, Animal)` → True |
| `__mro__` | 查看方法解析顺序 | `Dog.__mro__` |
| `@abstractmethod` | 标记抽象方法 | 子类必须实现 |
| `raise NotImplementedError` | 提醒必须实现 | 子类忘记实现会报错 |

---

## 五、练习题

### 练习 1：员工管理系统
创建一个员工类体系：
- `Employee` 基类：包含姓名、工号、基本工资
- `Manager` 子类：添加奖金属性，复写工资计算方法
- `Developer` 子类：添加加班费属性，复写工资计算方法
- 编写一个函数，传入员工列表，打印每个人的工资信息（体现多态）

### 练习 2：图形面积计算
创建一个图形类体系：
- `Shape` 抽象基类
- `Circle`、`Rectangle`、`Triangle` 子类
- 编写一个函数，接受图形列表，计算总面积

### 练习 3：动物叫声模拟器
创建多个动物类，每个类都有 `speak()` 方法，但实现不同。
编写一个函数，传入动物列表，让它们依次叫。

### 练习 4：文件处理器（鸭子类型）
创建 `TextFile`、`CSVFile`、`JSONFile` 三个类（不需要继承同一个父类），每个类都有 `read()` 和 `write(data)` 方法。
编写一个 `process_file(file)` 函数，调用 file 的 read 和 write 方法。验证鸭子类型。

### 练习 5：游戏角色扩展
基于本章的游戏角色示例，添加以下内容：
- `Healer`（治疗师）子类：有自己的 `heal()` 方法，复写 `attack_enemy` 为"不攻击，改为治疗"
- `Archer`（弓箭手）子类：有 `arrows`（箭矢）属性，攻击时消耗箭矢，箭矢为 0 时无法攻击
- 编写一个 `battle_round(characters)` 函数，让所有角色轮流攻击同一个敌人

### 练习 6：配置管理器（属性复写）
创建一个 `Config` 基类，定义默认配置项（如 `debug=False`, `port=8080`, `host="localhost"`）。
创建 `DevConfig`、`TestConfig`、`ProdConfig` 子类，分别复写这些配置项。
编写一个函数，接受配置对象，打印所有配置信息。
