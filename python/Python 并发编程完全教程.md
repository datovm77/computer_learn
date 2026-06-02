# Python 并发编程完全教程

> 本笔记以 Python 多线程为主线，延伸比较多进程、GIL、分布式进程与常见并发实战，便于放在同一份并发编程教程中对照学习。

## 第一章：并发编程基础概念

### 1.1 什么是并发编程？

在开始学习具体技术之前，我们需要理解几个核心概念：

**串行执行**：程序按顺序一步步执行，就像排队买票，一个人办完才能轮到下一个。

**并发执行**：多个任务看起来同时进行，就像银行有多个窗口，可以同时服务多个客户。

**为什么需要并发编程？**
- 提高程序执行效率
- 充分利用多核CPU资源
- 改善用户体验（比如下载文件时界面不卡顿）
- 处理大量I/O操作（网络请求、文件读写等）

### 1.2 进程与线程的概念

**进程（Process）**：
- 是操作系统分配资源的基本单位
- 每个进程有独立的内存空间
- 就像一个独立运行的程序实例
- 例如：打开一个浏览器就是一个进程

**线程（Thread）**：
- 是CPU调度的基本单位
- 同一进程内的线程共享内存空间
- 就像进程内部的"小任务"
- 例如：浏览器的一个标签页可能是一个线程

**形象比喻**：
- 进程像一家工厂，有自己的厂房、设备、原材料
- 线程像工厂里的工人，共用工厂的资源，协同工作

---

## 第二章：多线程编程（Thread）

### 2.1 为什么先学线程？

线程相比进程更轻量级，创建和切换的开销更小，适合I/O密集型任务。Python的线程编程相对简单，是学习并发编程的良好起点。

### 2.2 创建第一个线程

#### 方法一：使用Thread类直接创建

```python
import threading
import time

# 定义一个简单的函数
def print_numbers():
    """打印数字1到5"""
    for i in range(1, 6):
        print(f"数字: {i}")
        time.sleep(1)  # 暂停1秒

# 创建线程
thread = threading.Thread(target=print_numbers)

# 启动线程
thread.start()

# 主线程继续执行
print("主线程继续执行")

# 等待线程执行完毕
thread.join()
print("所有线程执行完毕")
```

**代码详解**：
1. `threading.Thread(target=print_numbers)`：创建线程对象，target指定要执行的函数
2. `thread.start()`：启动线程，此时print_numbers在新线程中执行
3. `thread.join()`：主线程等待子线程执行完毕

**输出示例**：
```
主线程继续执行
数字: 1
数字: 2
数字: 3
数字: 4
数字: 5
所有线程执行完毕
```

#### 方法二：传递参数给线程函数

```python
import threading
import time

def greet(name, times):
    """问候函数，接收参数"""
    for i in range(times):
        print(f"你好, {name}! (第{i+1}次)")
        time.sleep(0.5)

# 使用args传递位置参数
thread1 = threading.Thread(target=greet, args=("张三", 3))

# 使用kwargs传递关键字参数
thread2 = threading.Thread(target=greet, kwargs={"name": "李四", "times": 3})

thread1.start()
thread2.start()

thread1.join()
thread2.join()

print("所有问候完成")
```

**关键点**：
- `args`：传递位置参数，必须是元组
- `kwargs`：传递关键字参数，必须是字典
- 两个线程会并发执行，输出可能交错

#### 方法三：继承Thread类

```python
import threading
import time

class MyThread(threading.Thread):
    """自定义线程类"""
  
    def __init__(self, name, count):
        super().__init__()  # 调用父类构造函数
        self.thread_name = name
        self.count = count
  
    def run(self):
        """线程执行的主体方法"""
        print(f"{self.thread_name} 开始执行")
        for i in range(self.count):
            print(f"{self.thread_name}: {i+1}")
            time.sleep(0.5)
        print(f"{self.thread_name} 执行完毕")

# 创建线程实例
thread1 = MyThread("线程A", 3)
thread2 = MyThread("线程B", 3)

# 启动线程
thread1.start()
thread2.start()

# 等待完成
thread1.join()
thread2.join()
```

**为什么要继承Thread类？**
- 可以封装更复杂的逻辑
- 可以添加自定义属性和方法
- 代码结构更清晰

### 2.3 线程同步问题

当多个线程访问共享资源时，可能出现数据不一致的问题。

#### 问题演示：不安全的计数器

```python
import threading

# 全局共享变量
counter = 0

def increment():
    """增加计数器"""
    global counter
    for _ in range(100000):
        counter += 1

# 创建两个线程
thread1 = threading.Thread(target=increment)
thread2 = threading.Thread(target=increment)

thread1.start()
thread2.start()

thread1.join()
thread2.join()

print(f"最终计数: {counter}")
print(f"期望计数: 200000")
```

**运行多次可能会发现**：最终计数不是200000。不同 Python 版本和运行环境下，这个竞态不一定每次都能稳定复现，但 `counter += 1` 仍然不是一个受锁保护的原子操作。

**为什么？**
`counter += 1` 实际分为三步：
1. 读取counter的值
2. 加1
3. 写回counter

两个线程可能同时读到相同的值，导致一次加法丢失。

#### 解决方案一：使用Lock（锁）

```python
import threading

counter = 0
lock = threading.Lock()  # 创建锁对象

def increment():
    """使用锁保护的增加操作"""
    global counter
    for _ in range(100000):
        lock.acquire()  # 获取锁
        try:
            counter += 1
        finally:
            lock.release()  # 释放锁

# 创建并运行线程
thread1 = threading.Thread(target=increment)
thread2 = threading.Thread(target=increment)

thread1.start()
thread2.start()

thread1.join()
thread2.join()

print(f"最终计数: {counter}")  # 现在总是200000
```

**Lock工作原理**：
- 同一时刻只有一个线程能获取锁
- 其他线程必须等待锁被释放
- 确保临界区代码的原子性

**更优雅的写法：使用with语句**

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:  # 自动获取和释放锁
            counter += 1

# 其余代码相同...
```

**with语句的优势**：
- 自动处理锁的获取和释放
- 即使发生异常也会释放锁
- 代码更简洁

### 2.4 死锁问题及避免

#### 什么是死锁？

两个或多个线程互相等待对方持有的锁，导致都无法继续执行。

```python
import threading
import time

lock1 = threading.Lock()
lock2 = threading.Lock()

def task1():
    """任务1：先获取lock1，再获取lock2"""
    print("任务1: 尝试获取lock1")
    with lock1:
        print("任务1: 获取到lock1")
        time.sleep(0.1)  # 模拟处理时间
      
        print("任务1: 尝试获取lock2")
        with lock2:
            print("任务1: 获取到lock2")

def task2():
    """任务2：先获取lock2，再获取lock1"""
    print("任务2: 尝试获取lock2")
    with lock2:
        print("任务2: 获取到lock2")
        time.sleep(0.1)
      
        print("任务2: 尝试获取lock1")
        with lock1:
            print("任务2: 获取到lock1")

# 启动线程
thread1 = threading.Thread(target=task1)
thread2 = threading.Thread(target=task2)

thread1.start()
thread2.start()

# 这个程序会卡住！
```

**死锁形成过程**：
1. 线程1获取lock1
2. 线程2获取lock2
3. 线程1等待lock2（被线程2持有）
4. 线程2等待lock1（被线程1持有）
5. 互相等待，程序卡死

#### 避免死锁的方法

**方法一：固定锁的获取顺序**

```python
import threading
import time

lock1 = threading.Lock()
lock2 = threading.Lock()

def task1():
    """统一按照lock1 -> lock2的顺序"""
    with lock1:
        print("任务1: 获取到lock1")
        time.sleep(0.1)
        with lock2:
            print("任务1: 获取到lock2")

def task2():
    """同样按照lock1 -> lock2的顺序"""
    with lock1:
        print("任务2: 获取到lock1")
        time.sleep(0.1)
        with lock2:
            print("任务2: 获取到lock2")

# 现在不会死锁了
thread1 = threading.Thread(target=task1)
thread2 = threading.Thread(target=task2)

thread1.start()
thread2.start()

thread1.join()
thread2.join()
```

**方法二：使用超时机制**

```python
import threading
import time

lock1 = threading.Lock()
lock2 = threading.Lock()

def task_with_timeout():
    """使用超时避免永久阻塞"""
    if lock1.acquire(timeout=1):  # 最多等待1秒
        try:
            print("获取到lock1")
            if lock2.acquire(timeout=1):
                try:
                    print("获取到lock2")
                    # 执行实际工作
                finally:
                    lock2.release()
            else:
                print("无法获取lock2，放弃操作")
        finally:
            lock1.release()
    else:
        print("无法获取lock1，放弃操作")

thread = threading.Thread(target=task_with_timeout)
thread.start()
thread.join()
```

### 2.5 线程间通信

#### 使用Queue（队列）

Queue是线程安全的队列，适合生产者-消费者模式。

```python
import threading
import queue
import time
import random

def producer(q, name):
    """生产者：生产数据放入队列"""
    for i in range(5):
        item = f"{name}-产品{i}"
        q.put(item)  # 放入队列
        print(f"{name} 生产了: {item}")
        time.sleep(random.uniform(0.1, 0.5))
    print(f"{name} 生产完毕")

def consumer(q, name):
    """消费者：从队列取出数据处理"""
    while True:
        try:
            # 从队列获取，超时1秒
            item = q.get(timeout=1)
            print(f"{name} 消费了: {item}")
            time.sleep(random.uniform(0.1, 0.3))
            q.task_done()  # 标记任务完成
        except queue.Empty:
            print(f"{name} 队列为空，退出")
            break

# 创建队列
q = queue.Queue(maxsize=10)  # 最多存放10个元素

# 创建生产者线程
producer1 = threading.Thread(target=producer, args=(q, "生产者1"))
producer2 = threading.Thread(target=producer, args=(q, "生产者2"))

# 创建消费者线程
consumer1 = threading.Thread(target=consumer, args=(q, "消费者1"))
consumer2 = threading.Thread(target=consumer, args=(q, "消费者2"))

# 启动所有线程
producer1.start()
producer2.start()
consumer1.start()
consumer2.start()

# 等待生产者完成
producer1.join()
producer2.join()

# 等待队列中所有任务被处理
q.join()

# 等待消费者线程退出
consumer1.join()
consumer2.join()

print("所有任务完成")
```

**Queue的关键方法**：
- `put(item)`：放入元素（队列满时阻塞）
- `get()`：取出元素（队列空时阻塞）
- `put(item, timeout=1)`：带超时的放入
- `get(timeout=1)`：带超时的取出
- `task_done()`：标记任务完成
- `join()`：等待所有任务完成

#### 使用Event（事件）

Event用于线程间的简单信号通知。

```python
import threading
import time

# 创建事件对象
event = threading.Event()

def waiter(name):
    """等待者：等待事件发生"""
    print(f"{name} 开始等待信号...")
    event.wait()  # 阻塞，直到事件被设置
    print(f"{name} 收到信号，开始工作！")

def setter():
    """设置者：发送事件信号"""
    print("设置者: 准备发送信号...")
    time.sleep(2)  # 模拟准备时间
    print("设置者: 发送信号！")
    event.set()  # 设置事件，唤醒所有等待者

# 创建多个等待线程
waiter1 = threading.Thread(target=waiter, args=("等待者1",))
waiter2 = threading.Thread(target=waiter, args=("等待者2",))
waiter3 = threading.Thread(target=waiter, args=("等待者3",))

# 创建设置线程
setter_thread = threading.Thread(target=setter)

# 启动所有线程
waiter1.start()
waiter2.start()
waiter3.start()
setter_thread.start()

# 等待完成
waiter1.join()
waiter2.join()
waiter3.join()
setter_thread.join()

print("所有线程完成")
```

**Event的方法**：
- `wait()`：阻塞直到事件被设置
- `set()`：设置事件，唤醒所有等待线程
- `clear()`：清除事件状态
- `is_set()`：检查事件是否已设置

#### 使用Condition（条件变量）

Condition比Event更灵活，可以实现复杂的通知机制。

```python
import threading
import time

condition = threading.Condition()
items = []  # 共享资源

def producer(n, consumer_count):
    """生产者"""
    for i in range(n):
        with condition:
            item = f"产品{i}"
            items.append(item)
            print(f"生产了: {item}")
            condition.notify()  # 通知一个等待的消费者
        time.sleep(0.5)
  
    # 发送结束信号，确保每个消费者都能退出
    with condition:
        for _ in range(consumer_count):
            items.append(None)
        condition.notify_all()

def consumer(name):
    """消费者"""
    while True:
        with condition:
            # 等待直到有产品
            while not items:
                print(f"{name} 等待产品...")
                condition.wait()
          
            item = items.pop(0)
            if item is None:
                print(f"{name} 收到结束信号，退出")
                break
            print(f"{name} 消费了: {item}")
      
        time.sleep(1)

# 启动线程
producer_thread = threading.Thread(target=producer, args=(5, 2))
consumer_thread1 = threading.Thread(target=consumer, args=("消费者1",))
consumer_thread2 = threading.Thread(target=consumer, args=("消费者2",))

consumer_thread1.start()
consumer_thread2.start()
producer_thread.start()

producer_thread.join()
consumer_thread1.join()
consumer_thread2.join()
```

**Condition的方法**：
- `wait()`：释放锁并等待通知
- `notify(n=1)`：唤醒n个等待线程
- `notify_all()`：唤醒所有等待线程

### 2.6 线程池（ThreadPoolExecutor）

手动管理大量线程很麻烦，线程池可以自动管理线程的创建和销毁。

```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(n):
    """模拟耗时任务"""
    print(f"任务 {n} 开始执行")
    time.sleep(1)
    result = n * n
    print(f"任务 {n} 完成，结果: {result}")
    return result

# 创建线程池，最多3个线程
with ThreadPoolExecutor(max_workers=3) as executor:
    # 提交多个任务
    futures = [executor.submit(task, i) for i in range(10)]
  
    # 获取结果
    for future in futures:
        result = future.result()  # 阻塞直到任务完成
        print(f"获取到结果: {result}")
```

**更优雅的方式：使用map**

```python
from concurrent.futures import ThreadPoolExecutor
import time

def square(n):
    """计算平方"""
    time.sleep(0.5)
    return n * n

# 使用map方法
with ThreadPoolExecutor(max_workers=3) as executor:
    numbers = [1, 2, 3, 4, 5]
    results = executor.map(square, numbers)
  
    # map返回迭代器，按顺序返回结果
    for num, result in zip(numbers, results):
        print(f"{num} 的平方是 {result}")
```

**线程池的优势**：
- 自动管理线程生命周期
- 限制并发线程数量
- 重用线程，减少创建开销
- 提供简洁的API

---

## 第三章：线程局部变量（ThreadLocal）

### 3.1 什么是ThreadLocal？

ThreadLocal为每个线程提供独立的变量副本，各线程之间互不干扰。

### 3.2 为什么需要ThreadLocal？

看一个问题场景：

```python
import threading

# 全局变量
user_name = None

def process_request(name):
    """处理请求"""
    global user_name
    user_name = name  # 设置当前用户
  
    import time
    time.sleep(0.1)  # 模拟处理时间
  
    # 获取当前用户
    print(f"处理用户: {user_name}")

# 创建多个线程模拟并发请求
thread1 = threading.Thread(target=process_request, args=("张三",))
thread2 = threading.Thread(target=process_request, args=("李四",))

thread1.start()
thread2.start()

thread1.join()
thread2.join()
```

**问题**：输出可能是：
```
处理用户: 李四
处理用户: 李四
```

两个线程共享全局变量，互相覆盖！

### 3.3 使用ThreadLocal解决

```python
import threading

# 创建ThreadLocal对象
local_data = threading.local()

def process_request(name):
    """处理请求"""
    # 为当前线程设置变量
    local_data.user_name = name
  
    import time
    time.sleep(0.1)
  
    # 获取当前线程的变量
    print(f"处理用户: {local_data.user_name}")

# 创建多个线程
thread1 = threading.Thread(target=process_request, args=("张三",))
thread2 = threading.Thread(target=process_request, args=("李四",))

thread1.start()
thread2.start()

thread1.join()
thread2.join()
```

**正确输出**：
```
处理用户: 张三
处理用户: 李四
```

### 3.4 ThreadLocal的实际应用

#### 场景一：数据库连接管理

```python
import threading
import time

# 模拟数据库连接类
class DBConnection:
    def __init__(self, thread_name):
        self.thread_name = thread_name
        print(f"为 {thread_name} 创建数据库连接")
  
    def query(self, sql):
        print(f"{self.thread_name} 执行查询: {sql}")
        return f"结果来自 {self.thread_name}"

# ThreadLocal存储每个线程的数据库连接
db_local = threading.local()

def get_connection():
    """获取当前线程的数据库连接"""
    if not hasattr(db_local, 'connection'):
        # 为当前线程创建连接
        thread_name = threading.current_thread().name
        db_local.connection = DBConnection(thread_name)
    return db_local.connection

def worker(task_id):
    """工作线程"""
    # 获取连接（每个线程都是独立的）
    conn = get_connection()
  
    # 执行查询
    result = conn.query(f"SELECT * FROM tasks WHERE id={task_id}")
    print(f"任务 {task_id} 的结果: {result}")
  
    time.sleep(0.5)
  
    # 再次获取连接（是同一个连接）
    conn2 = get_connection()
    print(f"连接是否相同: {conn is conn2}")

# 创建多个工作线程
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(i,), name=f"工作线程{i}")
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

**输出示例**：
```
为 工作线程0 创建数据库连接
工作线程0 执行查询: SELECT * FROM tasks WHERE id=0
为 工作线程1 创建数据库连接
工作线程1 执行查询: SELECT * FROM tasks WHERE id=1
为 工作线程2 创建数据库连接
工作线程2 执行查询: SELECT * FROM tasks WHERE id=2
任务 0 的结果: 结果来自 工作线程0
连接是否相同: True
任务 1 的结果: 结果来自 工作线程1
连接是否相同: True
任务 2 的结果: 结果来自 工作线程2
连接是否相同: True
```

#### 场景二：用户上下文管理

```python
import threading

# 用户上下文
context = threading.local()

def set_user(user_id, user_name):
    """设置当前用户信息"""
    context.user_id = user_id
    context.user_name = user_name

def get_current_user():
    """获取当前用户"""
    return {
        'id': getattr(context, 'user_id', None),
        'name': getattr(context, 'user_name', None)
    }

def business_logic():
    """业务逻辑"""
    user = get_current_user()
    print(f"当前用户: {user['name']} (ID: {user['id']})")
    # 执行业务操作...

def handle_request(user_id, user_name):
    """处理用户请求"""
    # 设置用户上下文
    set_user(user_id, user_name)
  
    # 调用业务逻辑
    business_logic()
  
    # 模拟更多操作
    import time
    time.sleep(0.1)
  
    # 再次获取用户信息
    user = get_current_user()
    print(f"请求完成，用户: {user['name']}")

# 模拟多个并发请求
threads = [
    threading.Thread(target=handle_request, args=(1, "张三")),
    threading.Thread(target=handle_request, args=(2, "李四")),
    threading.Thread(target=handle_request, args=(3, "王五"))
]

for t in threads:
    t.start()

for t in threads:
    t.join()
```

### 3.5 ThreadLocal的注意事项

**注意点一：线程池中的陷阱**

```python
from concurrent.futures import ThreadPoolExecutor
import threading

local_data = threading.local()

def task(value):
    try:
        # 检查是否有旧值；hasattr不会漏掉0、空字符串等值
        if hasattr(local_data, 'value'):
            print(f"警告：线程有旧值 {local_data.value}")
      
        local_data.value = value
        print(f"设置值: {value}")
    finally:
        # 线程池会重用线程，任务结束时应删除线程本地状态
        if hasattr(local_data, 'value'):
            del local_data.value

with ThreadPoolExecutor(max_workers=2) as executor:
    # 提交6个任务到2个线程
    for i in range(6):
        executor.submit(task, i)
```

**建议**：
- 在任务开始时初始化ThreadLocal变量
- 在线程池中，任务结束时用 `finally` 删除或重置ThreadLocal变量
- `contextvars` 更适合异步上下文传播，不会自动解决线程池复用线程带来的旧状态

---

## 第四章：多进程编程（Process）

### 4.1 为什么需要多进程？

**线程的局限性**：
- Python有全局解释器锁（GIL），同一时刻只有一个线程执行Python字节码
- 线程适合I/O密集型任务，不适合CPU密集型任务
- 多核CPU的计算能力无法被多线程充分利用

**进程的优势**：
- 每个进程有独立的GIL，可以真正并行
- 适合CPU密集型任务
- 更好地利用多核CPU

### 4.2 创建第一个进程

#### 基本用法

```python
import multiprocessing
import time
import os

def worker(name):
    """工作进程"""
    print(f"进程 {name} 开始，PID: {os.getpid()}")
    time.sleep(2)
    print(f"进程 {name} 完成")

if __name__ == '__main__':
    print(f"主进程 PID: {os.getpid()}")
  
    # 创建进程
    process = multiprocessing.Process(target=worker, args=("子进程",))
  
    # 启动进程
    process.start()
  
    print("主进程继续执行")
  
    # 等待进程完成
    process.join()
  
    print("所有进程完成")
```

**重要**：多进程代码必须放在 `if __name__ == '__main__':` 内，否则在Windows上会出错！

**为什么？**
- Windows创建进程是通过导入主模块实现的
- 如果不加保护，会导致无限递归创建进程

#### 传递参数

```python
import multiprocessing

def calculate(x, y, operation):
    """计算函数"""
    if operation == 'add':
        result = x + y
    elif operation == 'multiply':
        result = x * y
    else:
        result = None
  
    print(f"计算结果: {x} {operation} {y} = {result}")
    return result

if __name__ == '__main__':
    # 使用args传递参数
    p1 = multiprocessing.Process(target=calculate, args=(10, 5, 'add'))
  
    # 使用kwargs传递参数
    p2 = multiprocessing.Process(
        target=calculate,
        kwargs={'x': 10, 'y': 5, 'operation': 'multiply'}
    )
  
    p1.start()
    p2.start()
  
    p1.join()
    p2.join()
```

### 4.3 进程间通信

进程有独立的内存空间，不能直接共享变量，需要特殊的通信机制。

#### 方法一：使用Queue（队列）

```python
import multiprocessing
import time

def producer(queue, name):
    """生产者进程"""
    for i in range(5):
        item = f"{name}-数据{i}"
        queue.put(item)
        print(f"{name} 生产: {item}")
        time.sleep(0.5)
    print(f"{name} 生产完毕")

def consumer(queue, name):
    """消费者进程"""
    while True:
        try:
            item = queue.get(timeout=2)  # 2秒超时
            print(f"{name} 消费: {item}")
            time.sleep(0.3)
        except:
            print(f"{name} 队列为空，退出")
            break

if __name__ == '__main__':
    # 创建进程安全的队列
    queue = multiprocessing.Queue()
  
    # 创建生产者进程
    p1 = multiprocessing.Process(target=producer, args=(queue, "生产者1"))
    p2 = multiprocessing.Process(target=producer, args=(queue, "生产者2"))
  
    # 创建消费者进程
    c1 = multiprocessing.Process(target=consumer, args=(queue, "消费者1"))
    c2 = multiprocessing.Process(target=consumer, args=(queue, "消费者2"))
  
    # 启动所有进程
    p1.start()
    p2.start()
    c1.start()
    c2.start()
  
    # 等待生产者完成
    p1.join()
    p2.join()
  
    # 等待消费者完成
    c1.join()
    c2.join()
  
    print("所有进程完成")
```

**Queue的关键特性**：
- 进程安全，多个进程可以同时操作
- 底层使用管道和锁实现
- 与`queue.Queue`（线程队列）API类似

**Queue的方法**：
- `put(item)`：放入元素
- `get()`：取出元素
- `put(item, timeout=1)`：带超时
- `get(timeout=1)`：带超时
- `empty()`：检查是否为空（不可靠）
- `qsize()`：获取大小（近似值）

#### 方法二：使用Pipe（管道）

Pipe创建一对连接对象，用于双向通信。

```python
import multiprocessing
import time

def sender(conn):
    """发送进程"""
    messages = ["消息1", "消息2", "消息3"]
  
    for msg in messages:
        print(f"发送: {msg}")
        conn.send(msg)
        time.sleep(1)
  
    # 发送结束信号
    conn.send("END")
    conn.close()

def receiver(conn):
    """接收进程"""
    while True:
        msg = conn.recv()  # 阻塞直到接收到数据
        print(f"接收: {msg}")
      
        if msg == "END":
            break
      
        # 发送响应
        conn.send(f"已收到: {msg}")
  
    conn.close()

if __name__ == '__main__':
    # 创建管道，返回两端连接对象
    parent_conn, child_conn = multiprocessing.Pipe()
  
    # 创建进程
    p1 = multiprocessing.Process(target=sender, args=(parent_conn,))
    p2 = multiprocessing.Process(target=receiver, args=(child_conn,))
  
    p1.start()
    p2.start()
  
    # 主进程也可以通信
    # 注意：parent_conn和child_conn不应该同时在同一进程中使用
  
    p1.join()
    p2.join()
  
    print("通信完成")
```

**Pipe vs Queue**：

| 特性     | Pipe                         | Queue               |
| -------- | ---------------------------- | ------------------- |
| 通信方式 | 点对点                       | 多对多              |
| 性能     | 更快                         | 稍慢                |
| 适用场景 | 两个进程通信                 | 多进程生产者-消费者 |
| 安全性   | 需手动同步（多个进程同时写） | 内置同步            |

**Pipe的高级用法**：

```python
import multiprocessing
import time

def bidirectional_worker(conn, name):
    """双向通信的工作进程"""
    # 发送消息
    conn.send(f"你好，我是{name}")
  
    # 接收消息
    msg = conn.recv()
    print(f"{name} 收到: {msg}")
  
    # 再次发送
    conn.send(f"{name}：收到了！")
  
    conn.close()

if __name__ == '__main__':
    conn1, conn2 = multiprocessing.Pipe()
  
    p = multiprocessing.Process(target=bidirectional_worker, args=(conn2, "子进程"))
    p.start()
  
    # 主进程接收
    msg = conn1.recv()
    print(f"主进程收到: {msg}")
  
    # 主进程发送
    conn1.send("你好，子进程！")
  
    # 主进程再次接收
    msg = conn1.recv()
    print(f"主进程收到: {msg}")
  
    conn1.close()
    p.join()
```

#### 方法三：使用Value和Array（共享内存）

共享内存是最快的进程间通信方式，但需要手动同步。

**使用Value共享单个值**：

```python
import multiprocessing
import time

def increment(shared_value, lock):
    """增加共享值"""
    for _ in range(1000):
        with lock:  # 必须使用锁
            shared_value.value += 1

if __name__ == '__main__':
    # 创建共享值
    # 'd' 表示double（浮点数），也可以用 'i' 表示int
    shared_value = multiprocessing.Value('i', 0)
  
    # 创建锁
    lock = multiprocessing.Lock()
  
    # 创建多个进程
    processes = []
    for _ in range(5):
        p = multiprocessing.Process(target=increment, args=(shared_value, lock))
        processes.append(p)
        p.start()
  
    # 等待所有进程完成
    for p in processes:
        p.join()
  
    print(f"最终值: {shared_value.value}")
    print(f"期望值: 5000")
```

**Value的类型码**：
- `'i'`：int (整数)
- `'d'`：double (浮点数)
- `'f'`：float (单精度浮点)
- `'c'`：char (字符)
- `'b'`：signed char (有符号字节)

**使用Array共享数组**：

```python
import multiprocessing

def modify_array(shared_array, index, value):
    """修改共享数组"""
    shared_array[index] = value
    print(f"进程设置 array[{index}] = {value}")

if __name__ == '__main__':
    # 创建共享数组
    # 'i' 表示整数类型，10 是数组大小
    shared_array = multiprocessing.Array('i', 10)
  
    # 初始化数组
    for i in range(10):
        shared_array[i] = 0
  
    print(f"初始数组: {list(shared_array)}")
  
    # 创建进程修改数组
    processes = []
    for i in range(10):
        p = multiprocessing.Process(
            target=modify_array,
            args=(shared_array, i, i * i)
        )
        processes.append(p)
        p.start()
  
    # 等待完成
    for p in processes:
        p.join()
  
    print(f"最终数组: {list(shared_array)}")
```

**共享复杂数据结构**：

```python
import multiprocessing
from ctypes import Structure, c_int, c_double

# 定义结构体
class Point(Structure):
    _fields_ = [('x', c_int), ('y', c_int)]

def modify_point(shared_point, x, y):
    """修改共享的点"""
    shared_point.x = x
    shared_point.y = y
    print(f"进程设置点为 ({x}, {y})")

if __name__ == '__main__':
    # 创建共享结构体
    shared_point = multiprocessing.Value(Point)
  
    # 初始化
    shared_point.x = 0
    shared_point.y = 0
  
    print(f"初始点: ({shared_point.x}, {shared_point.y})")
  
    # 创建进程修改
    p = multiprocessing.Process(target=modify_point, args=(shared_point, 10, 20))
    p.start()
    p.join()
  
    print(f"最终点: ({shared_point.x}, {shared_point.y})")
```

#### 方法四：使用Manager（管理器）

Manager提供更高级的共享对象，包括list、dict等。

```python
import multiprocessing
import time

def add_to_list(shared_list, items):
    """向共享列表添加元素"""
    for item in items:
        shared_list.append(item)
        print(f"添加: {item}")
        time.sleep(0.1)

def add_to_dict(shared_dict, key, value):
    """向共享字典添加元素"""
    shared_dict[key] = value
    print(f"设置: {key} = {value}")

if __name__ == '__main__':
    # 创建Manager
    with multiprocessing.Manager() as manager:
        # 创建共享列表
        shared_list = manager.list()
      
        # 创建共享字典
        shared_dict = manager.dict()
      
        # 创建进程操作共享列表
        p1 = multiprocessing.Process(
            target=add_to_list,
            args=(shared_list, [1, 2, 3])
        )
        p2 = multiprocessing.Process(
            target=add_to_list,
            args=(shared_list, [4, 5, 6])
        )
      
        # 创建进程操作共享字典
        p3 = multiprocessing.Process(
            target=add_to_dict,
            args=(shared_dict, "name", "张三")
        )
        p4 = multiprocessing.Process(
            target=add_to_dict,
            args=(shared_dict, "age", 25)
        )
      
        # 启动所有进程
        p1.start()
        p2.start()
        p3.start()
        p4.start()
      
        # 等待完成
        p1.join()
        p2.join()
        p3.join()
        p4.join()
      
        print(f"\n最终列表: {list(shared_list)}")
        print(f"最终字典: {dict(shared_dict)}")
```

**Manager支持的类型**：
- `list`：共享列表
- `dict`：共享字典
- `Queue`：共享队列
- `Lock`：锁
- `Event`：事件
- `Condition`：条件变量
- `Namespace`：命名空间对象

**使用Namespace**：

```python
import multiprocessing

def worker(ns, lock, name):
    """工作进程"""
    with lock:
        ns.counter += 1
        ns.name = name
        print(f"进程 {name}: counter = {ns.counter}")

if __name__ == '__main__':
    with multiprocessing.Manager() as manager:
        # 创建命名空间
        ns = manager.Namespace()
        ns.counter = 0
        ns.name = ""
        lock = manager.Lock()
      
        # 创建进程
        processes = []
        for i in range(5):
            p = multiprocessing.Process(target=worker, args=(ns, lock, f"进程{i}"))
            processes.append(p)
            p.start()
      
        # 等待完成
        for p in processes:
            p.join()
      
        print(f"\n最终状态:")
        print(f"counter = {ns.counter}")
        print(f"name = {ns.name}")
```

**性能对比**：

| 方法        | 速度 | 复杂度 | 适用场景     |
| ----------- | ---- | ------ | ------------ |
| Pipe        | 最快 | 简单   | 两进程通信   |
| Queue       | 快   | 中等   | 多进程通信   |
| Value/Array | 很快 | 中等   | 共享简单数据 |
| Manager     | 较慢 | 简单   | 共享复杂对象 |

### 4.4 进程池（Pool）

进程池自动管理多个进程，非常适合批量任务处理。

#### 基本用法

```python
import multiprocessing
import time

def square(n):
    """计算平方"""
    print(f"进程 {multiprocessing.current_process().name} 计算 {n}^2")
    time.sleep(0.5)
    return n * n

if __name__ == '__main__':
    # 创建进程池，进程数为CPU核心数
    with multiprocessing.Pool() as pool:
        numbers = [1, 2, 3, 4, 5, 6, 7, 8]
      
        # 使用map方法
        results = pool.map(square, numbers)
      
        print(f"\n结果: {results}")
```

**输出示例**（4核CPU）：
```
进程 ForkPoolWorker-1 计算 1^2
进程 ForkPoolWorker-2 计算 2^2
进程 ForkPoolWorker-3 计算 3^2
进程 ForkPoolWorker-4 计算 4^2
进程 ForkPoolWorker-1 计算 5^2
进程 ForkPoolWorker-2 计算 6^2
进程 ForkPoolWorker-3 计算 7^2
进程 ForkPoolWorker-4 计算 8^2

结果: [1, 4, 9, 16, 25, 36, 49, 64]
```

#### Pool的方法详解

**方法一：map（批量处理）**

```python
import multiprocessing

def process_data(x):
    return x * 2

if __name__ == '__main__':
    with multiprocessing.Pool(processes=4) as pool:
        data = [1, 2, 3, 4, 5]
        # map会阻塞直到所有任务完成
        results = pool.map(process_data, data)
        print(results)  # [2, 4, 6, 8, 10]
```

**方法二：map_async（异步批量处理）**

```python
import multiprocessing
import time

def slow_square(x):
    time.sleep(1)
    return x * x

if __name__ == '__main__':
    with multiprocessing.Pool(processes=4) as pool:
        # 异步提交任务
        result_object = pool.map_async(slow_square, [1, 2, 3, 4, 5])
      
        print("任务已提交，主进程可以做其他事情...")
      
        # 做其他事情
        time.sleep(0.5)
        print("主进程做了一些工作")
      
        # 等待结果（阻塞）
        results = result_object.get()
        print(f"结果: {results}")
```

**方法三：apply（单个任务）**

```python
import multiprocessing

def add(x, y):
    return x + y

if __name__ == '__main__':
    with multiprocessing.Pool(processes=2) as pool:
        # apply 阻塞直到任务完成
        result = pool.apply(add, (10, 20))
        print(f"结果: {result}")  # 30
```

**方法四：apply_async（异步单个任务）**

```python
import multiprocessing
import time

def multiply(x, y):
    time.sleep(1)
    return x * y

if __name__ == '__main__':
    with multiprocessing.Pool(processes=2) as pool:
        # 异步提交多个任务
        result1 = pool.apply_async(multiply, (3, 4))
        result2 = pool.apply_async(multiply, (5, 6))
        result3 = pool.apply_async(multiply, (7, 8))
      
        print("任务已提交")
      
        # 获取结果
        print(f"结果1: {result1.get()}")  # 12
        print(f"结果2: {result2.get()}")  # 30
        print(f"结果3: {result3.get()}")  # 56
```

**方法五：starmap（传递多个参数）**

```python
import multiprocessing

def power(base, exponent):
    return base ** exponent

if __name__ == '__main__':
    with multiprocessing.Pool(processes=4) as pool:
        # starmap 可以传递多个参数
        args = [(2, 3), (3, 4), (4, 5), (5, 6)]
        results = pool.starmap(power, args)
        print(results)  # [8, 81, 1024, 15625]
```

#### 回调函数

```python
import multiprocessing
import time

def task(x):
    """耗时任务"""
    time.sleep(1)
    return x * x

def on_success(result):
    """成功回调"""
    print(f"任务完成，结果: {result}")

def on_error(error):
    """错误回调"""
    print(f"任务失败: {error}")

if __name__ == '__main__':
    with multiprocessing.Pool(processes=2) as pool:
        # 使用回调函数
        for i in range(5):
            pool.apply_async(
                task,
                args=(i,),
                callback=on_success,
                error_callback=on_error
            )
      
        # 关闭池，不再接受新任务
        pool.close()
      
        # 等待所有任务完成
        pool.join()
      
        print("所有任务完成")
```

#### 进度追踪

```python
import multiprocessing
from tqdm import tqdm  # 需要安装: pip install tqdm
import time

def slow_task(x):
    time.sleep(0.1)
    return x * x

if __name__ == '__main__':
    numbers = list(range(100))
  
    with multiprocessing.Pool(processes=4) as pool:
        # 使用imap显示进度
        results = list(tqdm(
            pool.imap(slow_task, numbers),
            total=len(numbers),
            desc="处理中"
        ))
      
        print(f"\n完成，处理了 {len(results)} 个任务")
```

#### 实际案例：批量图像处理

```python
import multiprocessing
import os
import time

def process_image(image_path):
    """模拟图像处理"""
    filename = os.path.basename(image_path)
    print(f"处理 {filename}...")
  
    # 模拟耗时操作
    time.sleep(0.5)
  
    # 返回处理结果
    return {
        'path': image_path,
        'status': 'success',
        'size': len(filename)  # 模拟处理后的大小
    }

if __name__ == '__main__':
    # 模拟图像文件列表
    image_files = [f"image_{i}.jpg" for i in range(20)]
  
    print(f"开始处理 {len(image_files)} 个图像文件...\n")
  
    start_time = time.time()
  
    # 使用进程池并行处理
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(process_image, image_files)
  
    end_time = time.time()
  
    # 统计结果
    success_count = sum(1 for r in results if r['status'] == 'success')
  
    print(f"\n处理完成:")
    print(f"- 总数: {len(results)}")
    print(f"- 成功: {success_count}")
    print(f"- 耗时: {end_time - start_time:.2f}秒")
```

### 4.5 进程同步

进程也需要同步机制来协调操作。

#### 使用Lock

```python
import multiprocessing
import time

def worker(lock, shared_value, worker_id):
    """使用锁的工作进程"""
    for i in range(3):
        # 获取锁
        lock.acquire()
        try:
            print(f"进程 {worker_id} 获取锁")
            current = shared_value.value
            time.sleep(0.1)  # 模拟处理
            shared_value.value = current + 1
            print(f"进程 {worker_id} 更新值为 {shared_value.value}")
        finally:
            lock.release()
            print(f"进程 {worker_id} 释放锁")

if __name__ == '__main__':
    lock = multiprocessing.Lock()
    shared_value = multiprocessing.Value('i', 0)
  
    processes = []
    for i in range(3):
        p = multiprocessing.Process(
            target=worker,
            args=(lock, shared_value, i)
        )
        processes.append(p)
        p.start()
  
    for p in processes:
        p.join()
  
    print(f"\n最终值: {shared_value.value}")
```

#### 使用Semaphore（信号量）

信号量限制同时访问资源的进程数。

```python
import multiprocessing
import time
import random

def access_resource(semaphore, worker_id):
    """访问受限资源"""
    print(f"进程 {worker_id} 等待访问资源...")
  
    # 获取信号量
    semaphore.acquire()
  
    try:
        print(f"进程 {worker_id} 开始使用资源")
        # 模拟使用资源
        time.sleep(random.uniform(1, 2))
        print(f"进程 {worker_id} 使用资源完毕")
    finally:
        # 释放信号量
        semaphore.release()

if __name__ == '__main__':
    # 创建信号量，最多允许2个进程同时访问
    semaphore = multiprocessing.Semaphore(2)
  
    processes = []
    for i in range(6):
        p = multiprocessing.Process(
            target=access_resource,
            args=(semaphore, i)
        )
        processes.append(p)
        p.start()
  
    for p in processes:
        p.join()
  
    print("所有进程完成")
```

**输出示例**（每次最多2个进程同时执行）：
```
进程 0 等待访问资源...
进程 1 等待访问资源...
进程 2 等待访问资源...
进程 0 开始使用资源
进程 1 开始使用资源
进程 3 等待访问资源...
进程 4 等待访问资源...
进程 5 等待访问资源...
进程 0 使用资源完毕
进程 2 开始使用资源
进程 1 使用资源完毕
进程 3 开始使用资源
...
```

#### 使用Event

```python
import multiprocessing
import time

def waiter(event, name):
    """等待事件的进程"""
    print(f"{name} 开始等待...")
    event.wait()  # 阻塞直到事件被设置
    print(f"{name} 收到信号，开始工作！")

def setter(event):
    """设置事件的进程"""
    print("准备发送信号...")
    time.sleep(3)
    print("发送信号！")
    event.set()

if __name__ == '__main__':
    event = multiprocessing.Event()
  
    # 创建等待进程
    waiters = []
    for i in range(3):
        p = multiprocessing.Process(target=waiter, args=(event, f"等待者{i}"))
        waiters.append(p)
        p.start()
  
    # 创建设置进程
    setter_process = multiprocessing.Process(target=setter, args=(event,))
    setter_process.start()
  
    # 等待所有进程
    for p in waiters:
        p.join()
    setter_process.join()
  
    print("所有进程完成")
```

### 4.6 进程间的异常处理

```python
import multiprocessing
import time
import traceback

def risky_task(x):
    """可能出错的任务"""
    time.sleep(0.5)
  
    if x == 5:
        raise ValueError(f"不喜欢数字 {x}！")
  
    return x * x

def safe_task(x):
    """安全包装的任务"""
    try:
        return risky_task(x)
    except Exception as e:
        # 返回错误信息而不是抛出异常
        return {
            'error': str(e),
            'traceback': traceback.format_exc()
        }

if __name__ == '__main__':
    numbers = list(range(10))
  
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(safe_task, numbers)
  
    # 检查结果
    for i, result in enumerate(results):
        if isinstance(result, dict) and 'error' in result:
            print(f"任务 {i} 失败: {result['error']}")
        else:
            print(f"任务 {i} 成功: {result}")
```

---

## 第五章：进程与线程对比

### 5.1 根本区别

| 特性     | 进程                 | 线程                         |
| -------- | -------------------- | ---------------------------- |
| 内存空间 | 独立的内存空间       | 共享进程的内存空间           |
| 创建开销 | 大（需要复制内存）   | 小（共享内存）               |
| 切换开销 | 大                   | 小                           |
| 通信方式 | IPC（Queue、Pipe等） | 直接共享变量                 |
| GIL影响  | 不受影响，真正并行   | 受GIL限制                    |
| 适用场景 | CPU密集型任务        | I/O密集型任务                |
| 崩溃影响 | 进程独立，互不影响   | 线程异常通常只终止该线程，但可能破坏共享状态或导致整体逻辑失败 |

### 5.2 性能对比实验

#### CPU密集型任务

```python
import time
import threading
import multiprocessing

def cpu_intensive_task(n):
    """CPU密集型：计算质数"""
    count = 0
    for i in range(2, n):
        is_prime = True
        for j in range(2, int(i ** 0.5) + 1):
            if i % j == 0:
                is_prime = False
                break
        if is_prime:
            count += 1
    return count

def test_sequential():
    """串行执行"""
    start = time.time()
  
    results = []
    for _ in range(4):
        result = cpu_intensive_task(10000)
        results.append(result)
  
    end = time.time()
    print(f"串行执行耗时: {end - start:.2f}秒")
    return results

def test_multithreading():
    """多线程执行"""
    start = time.time()
  
    threads = []
    results = []
  
    def worker():
        result = cpu_intensive_task(10000)
        results.append(result)
  
    for _ in range(4):
        t = threading.Thread(target=worker)
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    end = time.time()
    print(f"多线程执行耗时: {end - start:.2f}秒")
    return results

def test_multiprocessing():
    """多进程执行"""
    start = time.time()
  
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(cpu_intensive_task, [10000] * 4)
  
    end = time.time()
    print(f"多进程执行耗时: {end - start:.2f}秒")
    return results

if __name__ == '__main__':
    print("=== CPU密集型任务性能对比 ===\n")
  
    test_sequential()
    test_multithreading()
    test_multiprocessing()
```

**典型输出**（4核CPU）：

```
=== CPU密集型任务性能对比 ===

串行执行耗时: 4.50秒
多线程执行耗时: 4.48秒  # 几乎没有提升！
多进程执行耗时: 1.25秒  # 显著提升！
```

**结论**：CPU密集型任务应使用多进程，多线程因为GIL的存在无法真正并行。

#### I/O密集型任务

```python
import time
import threading
import multiprocessing
import requests  # 需要安装: pip install requests

def io_intensive_task(url):
    """I/O密集型：网络请求"""
    try:
        response = requests.get(url, timeout=5)
        return len(response.content)
    except:
        return 0

def test_sequential_io():
    """串行执行I/O任务"""
    urls = ['https://www.python.org'] * 10
  
    start = time.time()
    results = []
    for url in urls:
        result = io_intensive_task(url)
        results.append(result)
  
    end = time.time()
    print(f"串行I/O执行耗时: {end - start:.2f}秒")
    return results

def test_multithreading_io():
    """多线程执行I/O任务"""
    urls = ['https://www.python.org'] * 10
  
    start = time.time()
  
    results = []
    threads = []
  
    def worker(url):
        result = io_intensive_task(url)
        results.append(result)
  
    for url in urls:
        t = threading.Thread(target=worker, args=(url,))
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    end = time.time()
    print(f"多线程I/O执行耗时: {end - start:.2f}秒")
    return results

def test_multiprocessing_io():
    """多进程执行I/O任务"""
    urls = ['https://www.python.org'] * 10
  
    start = time.time()
  
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(io_intensive_task, urls)
  
    end = time.time()
    print(f"多进程I/O执行耗时: {end - start:.2f}秒")
    return results

if __name__ == '__main__':
    print("=== I/O密集型任务性能对比 ===\n")
  
    test_sequential_io()
    test_multithreading_io()
    test_multiprocessing_io()
```

**典型输出**：
```
=== I/O密集型任务性能对比 ===

串行I/O执行耗时: 5.20秒
多线程I/O执行耗时: 0.55秒  # 显著提升！
多进程I/O执行耗时: 0.60秒  # 也有提升，但不如线程
```

**结论**：I/O密集型任务应优先使用多线程，因为：
1. 线程创建开销小
2. I/O操作会释放GIL，不受GIL限制
3. 线程切换快，通信方便

### 5.3 选择决策树

```
你的任务是什么类型？
│
├─ CPU密集型（大量计算）
│  └─ 使用多进程 (multiprocessing.Pool)
│
├─ I/O密集型（网络、文件、数据库）
│  ├─ 简单并发
│  │  └─ 使用多线程 (threading.Thread 或 ThreadPoolExecutor)
│  │
│  └─ 大量并发（成百上千）
│     └─ 使用异步编程 (asyncio) - 超出本教程范围
│
└─ 混合型
   └─ 进程 + 线程混合使用
```

### 5.4 混合使用示例

```python
import multiprocessing
import threading
import time

def cpu_task(x):
    """CPU密集型子任务"""
    total = 0
    for i in range(1000000):
        total += i
    return total * x

def io_task(url):
    """I/O密集型子任务"""
    time.sleep(0.1)  # 模拟网络延迟
    return f"从 {url} 获取的数据"

def hybrid_worker(worker_id):
    """混合型工作单元：先做I/O，再做CPU计算"""
    print(f"工作单元 {worker_id} 开始")
  
    # 第一步：使用线程并行处理多个I/O任务
    urls = [f"url_{i}" for i in range(5)]
    threads = []
    results = []
  
    def fetch(url):
        data = io_task(url)
        results.append(data)
  
    for url in urls:
        t = threading.Thread(target=fetch, args=(url,))
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    print(f"工作单元 {worker_id} 完成I/O，获取了 {len(results)} 个结果")
  
    # 第二步：对获取的数据进行CPU密集型处理
    computation_result = cpu_task(worker_id)
  
    return {
        'worker_id': worker_id,
        'io_count': len(results),
        'computation': computation_result
    }

if __name__ == '__main__':
    print("=== 混合型任务：进程 + 线程 ===\n")
  
    start = time.time()
  
    # 使用进程池处理多个工作单元
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(hybrid_worker, range(4))
  
    end = time.time()
  
    print(f"\n所有工作单元完成:")
    for result in results:
        print(f"  工作单元 {result['worker_id']}: "
              f"I/O任务 {result['io_count']} 个, "
              f"计算结果 {result['computation']}")
  
    print(f"\n总耗时: {end - start:.2f}秒")
```

### 5.5 内存使用对比

```python
import multiprocessing
import threading
import os
import psutil  # 需要安装: pip install psutil

def get_memory_usage():
    """获取当前进程内存使用（MB）"""
    process = psutil.Process(os.getpid())
    return process.memory_info().rss / 1024 / 1024

def memory_intensive_task():
    """占用内存的任务"""
    # 创建一个大列表
    data = [i for i in range(1000000)]
    return len(data)

def test_thread_memory():
    """测试多线程内存使用"""
    print("=== 多线程内存使用 ===")
  
    initial_memory = get_memory_usage()
    print(f"初始内存: {initial_memory:.2f} MB")
  
    threads = []
    for _ in range(10):
        t = threading.Thread(target=memory_intensive_task)
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    final_memory = get_memory_usage()
    print(f"最终内存: {final_memory:.2f} MB")
    print(f"增加内存: {final_memory - initial_memory:.2f} MB\n")

def process_memory_task():
    """进程版本的任务"""
    memory_intensive_task()

def test_process_memory():
    """测试多进程内存使用"""
    print("=== 多进程内存使用 ===")
  
    initial_memory = get_memory_usage()
    print(f"主进程初始内存: {initial_memory:.2f} MB")
  
    processes = []
    for _ in range(10):
        p = multiprocessing.Process(target=process_memory_task)
        processes.append(p)
        p.start()
  
    for p in processes:
        p.join()
  
    final_memory = get_memory_usage()
    print(f"主进程最终内存: {final_memory:.2f} MB")
    print(f"主进程增加内存: {final_memory - initial_memory:.2f} MB")
    print("注意：每个子进程都有独立的内存空间\n")

if __name__ == '__main__':
    test_thread_memory()
    test_process_memory()
```

**典型输出**：
```
=== 多线程内存使用 ===
初始内存: 45.23 MB
最终内存: 85.67 MB
增加内存: 40.44 MB

=== 多进程内存使用 ===
主进程初始内存: 45.89 MB
主进程最终内存: 46.12 MB
主进程增加内存: 0.23 MB
注意：每个子进程都有独立的内存空间
```

**分析**：
- 线程共享同一进程地址空间；示例中每个线程会临时分配自己的列表，任务结束后可被回收
- 进程独立内存，10个进程各有一份数据（实际总内存使用更大）
- 主进程内存几乎不增加，因为数据在子进程中；主进程最终RSS也不代表运行期间的总峰值内存

---

## 第六章：全局解释器锁（GIL）详解

### 6.1 什么是GIL？

**GIL（Global Interpreter Lock）**是默认CPython解释器中的一个互斥锁，确保同一时刻通常只有一个线程执行Python字节码；free-threaded 构建除外。

**为什么需要GIL？**
- CPython的内存管理不是线程安全的
- GIL简化了CPython的实现
- 保护内部数据结构不被并发修改

### 6.2 GIL的工作机制

```python
import threading
import time
import sys

def show_gil_behavior():
    """演示GIL的行为"""
  
    def count(n, name):
        """计数任务"""
        start = time.time()
        counter = 0
      
        for i in range(n):
            counter += 1
      
        end = time.time()
        print(f"{name} 计数到 {counter}, 耗时: {end - start:.4f}秒")
  
    print("=== 单线程执行 ===")
    start = time.time()
    count(50000000, "任务")
    end = time.time()
    print(f"总耗时: {end - start:.4f}秒\n")
  
    print("=== 双线程执行 ===")
    start = time.time()
  
    t1 = threading.Thread(target=count, args=(25000000, "线程1"))
    t2 = threading.Thread(target=count, args=(25000000, "线程2"))
  
    t1.start()
    t2.start()
  
    t1.join()
    t2.join()
  
    end = time.time()
    print(f"总耗时: {end - start:.4f}秒")
    print("注意：两个线程总耗时可能比单线程还长！\n")

if __name__ == '__main__':
    show_gil_behavior()
```

**典型输出**：
```
=== 单线程执行 ===
任务 计数到 50000000, 耗时: 2.1234秒
总耗时: 2.1234秒

=== 双线程执行 ===
线程1 计数到 25000000, 耗时: 1.2500秒
线程2 计数到 25000000, 耗时: 1.2800秒
总耗时: 2.5000秒
注意：两个线程总耗时可能比单线程还长！
```

**为什么多线程更慢？**
1. 线程切换开销
2. GIL的获取和释放开销
3. 两个线程无法真正并行执行

### 6.3 GIL的释放时机

Python解释器会在以下情况释放GIL：

#### 1. I/O操作时

```python
import threading
import time

def io_operation():
    """I/O操作会释放GIL"""
    print(f"线程 {threading.current_thread().name} 开始I/O")
  
    # 模拟I/O操作（文件、网络等）
    time.sleep(1)  # sleep会释放GIL
  
    print(f"线程 {threading.current_thread().name} 完成I/O")

if __name__ == '__main__':
    start = time.time()
  
    threads = []
    for i in range(5):
        t = threading.Thread(target=io_operation, name=f"线程{i}")
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    end = time.time()
    print(f"\n总耗时: {end - start:.2f}秒")
    print("注意：5个线程几乎同时完成（约1秒），而不是5秒")
```

**输出**：
```
线程 线程0 开始I/O
线程 线程1 开始I/O
线程 线程2 开始I/O
线程 线程3 开始I/O
线程 线程4 开始I/O
线程 线程0 完成I/O
线程 线程1 完成I/O
线程 线程2 完成I/O
线程 线程3 完成I/O
线程 线程4 完成I/O

总耗时: 1.01秒
注意：5个线程几乎同时完成（约1秒），而不是5秒
```

#### 2. 执行主动释放GIL的C扩展时

```python
import threading
import numpy as np  # NumPy的底层用C实现
import time

def numpy_computation():
    """演示可能主动释放GIL的NumPy操作"""
    name = threading.current_thread().name
    print(f"{name} 开始计算")
  
    # NumPy的部分大型矩阵运算会主动释放GIL
    a = np.random.rand(1000, 1000)
    b = np.random.rand(1000, 1000)
    c = np.dot(a, b)  # 矩阵乘法
  
    print(f"{name} 完成计算")

if __name__ == '__main__':
    start = time.time()
  
    threads = []
    for i in range(4):
        t = threading.Thread(target=numpy_computation, name=f"线程{i}")
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    end = time.time()
    print(f"\n总耗时: {end - start:.2f}秒")
    print("部分NumPy计算可以并行，因为底层C代码会主动释放GIL")
```

并非所有C扩展或所有NumPy操作都会释放GIL，是否能并行取决于扩展内部实现。

#### 3. 每执行一定数量的字节码

Python 3中，解释器会按 `sys.getswitchinterval()` 指定的时间间隔考虑切换线程，默认通常是0.005秒（5毫秒）。这是期望切换间隔，不保证精确，也不是固定执行若干字节码后切换。

```python
import sys
import threading

def show_switch_interval():
    """显示线程切换间隔"""
    interval = sys.getswitchinterval()
    print(f"当前线程切换间隔: {interval}秒")
  
    # 可以修改切换间隔
    sys.setswitchinterval(0.001)  # 1毫秒
    new_interval = sys.getswitchinterval()
    print(f"新的线程切换间隔: {new_interval}秒")
  
    # 恢复默认值
    sys.setswitchinterval(0.005)

if __name__ == '__main__':
    show_switch_interval()
```

### 6.4 绕过GIL的方法

#### 方法一：使用多进程

```python
import multiprocessing
import time

def cpu_bound_task(n):
    """CPU密集型任务"""
    total = 0
    for i in range(n):
        total += i * i
    return total

if __name__ == '__main__':
    n = 10000000
  
    # 多进程不受GIL限制
    print("=== 使用多进程 ===")
    start = time.time()
  
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(cpu_bound_task, [n // 4] * 4)
  
    end = time.time()
    print(f"耗时: {end - start:.2f}秒")
    print(f"结果总和: {sum(results)}")
```

#### 方法二：使用C扩展

使用Cython将Python代码编译为C代码：

```python
# 文件名: fast_compute.pyx (Cython代码)
"""
def fast_sum(int n):
    cdef long long total = 0
    cdef int i
  
    with nogil:  # 释放GIL
        for i in range(n):
            total += i
  
    return total
"""

# Python代码使用编译后的模块
# from fast_compute import fast_sum
# import threading
#
# def worker():
#     result = fast_sum(100000000)
#     print(f"结果: {result}")
#
# # 多个线程可以真正并行
# threads = [threading.Thread(target=worker) for _ in range(4)]
# for t in threads: t.start()
# for t in threads: t.join()
```

#### 方法三：使用其他Python实现

- **Jython**：基于JVM，没有GIL
- **IronPython**：基于.NET，没有GIL
- **PyPy**：有GIL，但JIT编译器可能更快

### 6.5 GIL对不同任务类型的影响总结

```python
import threading
import multiprocessing
import time
import requests

def demonstrate_gil_impact():
    """演示GIL对不同任务的影响"""
  
    # CPU密集型任务
    def cpu_task():
        total = sum(i * i for i in range(5000000))
        return total
  
    # I/O密集型任务
    def io_task():
        time.sleep(0.5)
        return "done"
  
    print("=== CPU密集型任务 ===")
  
    # 单线程
    start = time.time()
    for _ in range(4):
        cpu_task()
    single_time = time.time() - start
    print(f"单线程: {single_time:.2f}秒")
  
    # 多线程（受GIL限制）
    start = time.time()
    threads = [threading.Thread(target=cpu_task) for _ in range(4)]
    for t in threads: t.start()
    for t in threads: t.join()
    multi_thread_time = time.time() - start
    print(f"多线程: {multi_thread_time:.2f}秒")
    print(f"加速比: {single_time / multi_thread_time:.2f}x (接近1，说明没有加速)\n")
  
    print("=== I/O密集型任务 ===")
  
    # 单线程
    start = time.time()
    for _ in range(4):
        io_task()
    single_time = time.time() - start
    print(f"单线程: {single_time:.2f}秒")
  
    # 多线程（I/O操作释放GIL）
    start = time.time()
    threads = [threading.Thread(target=io_task) for _ in range(4)]
    for t in threads: t.start()
    for t in threads: t.join()
    multi_thread_time = time.time() - start
    print(f"多线程: {multi_thread_time:.2f}秒")
    print(f"加速比: {single_time / multi_thread_time:.2f}x (接近4，说明有明显加速)")

if __name__ == '__main__':
    demonstrate_gil_impact()
```

**典型输出**：
```
=== CPU密集型任务 ===
单线程: 2.50秒
多线程: 2.48秒
加速比: 1.01x (接近1，说明没有加速)

=== I/O密集型任务 ===
单线程: 2.00秒
多线程: 0.51秒
加速比: 3.92x (接近4，说明有明显加速)
```

### 6.6 优化建议

**根据任务类型选择方案**：

```python
"""
任务分类决策指南：

1. CPU密集型（大量计算）
   ├─ 纯Python代码
   │  └─ 使用 multiprocessing
   │
   └─ 可以用C/Cython优化
      └─ 使用 Cython + nogil 或 numba

2. I/O密集型（网络、文件、数据库）
   ├─ 并发量小（几十个）
   │  └─ 使用 threading.Thread 或 ThreadPoolExecutor
   │
   └─ 并发量大（成百上千）
      └─ 使用 asyncio (异步编程)

3. 混合型
   └─ 使用 multiprocessing + threading
      ├─ 进程处理CPU任务
      └─ 每个进程内用线程处理I/O任务
"""
```

---

## 第七章：并发性能优化实战

### 7.1 性能分析工具

#### 使用cProfile分析性能瓶颈

```python
import cProfile
import pstats
import io

def analyze_performance():
    """性能分析示例"""
  
    def slow_function():
        total = 0
        for i in range(1000000):
            total += i ** 2
        return total
  
    def fast_function():
        return sum(i ** 2 for i in range(1000000))
  
    # 创建性能分析器
    profiler = cProfile.Profile()
  
    # 开始分析
    profiler.enable()
  
    # 执行要分析的代码
    result1 = slow_function()
    result2 = fast_function()
  
    # 停止分析
    profiler.disable()
  
    # 输出结果
    s = io.StringIO()
    ps = pstats.Stats(profiler, stream=s).sort_stats('cumulative')
    ps.print_stats()
  
    print(s.getvalue())

if __name__ == '__main__':
    analyze_performance()
```

#### 使用line_profiler逐行分析

```python
# 需要安装: pip install line_profiler
# 使用装饰器标记要分析的函数

# @profile  # 取消注释后使用 kernprof -l -v script.py 运行
def process_data(data):
    """处理数据的函数"""
    result = []
  
    # 慢操作
    for item in data:
        result.append(item ** 2)
  
    # 快操作
    filtered = [x for x in result if x > 100]
  
    return filtered

if __name__ == '__main__':
    data = list(range(10000))
    result = process_data(data)
    print(f"处理完成，结果数量: {len(result)}")
```

### 7.2 常见性能陷阱

#### 陷阱一：频繁创建和销毁线程/进程

```python
import threading
import time
from concurrent.futures import ThreadPoolExecutor

def task(n):
    """简单任务"""
    return n * n

# ❌ 错误：频繁创建线程
def bad_approach():
    """每次都创建新线程"""
    start = time.time()
  
    results = []
    for i in range(100):
        t = threading.Thread(target=lambda: results.append(task(i)))
        t.start()
        t.join()
  
    end = time.time()
    print(f"频繁创建线程: {end - start:.4f}秒")

# ✅ 正确：使用线程池
def good_approach():
    """使用线程池复用线程"""
    start = time.time()
  
    with ThreadPoolExecutor(max_workers=10) as executor:
        results = list(executor.map(task, range(100)))
  
    end = time.time()
    print(f"使用线程池: {end - start:.4f}秒")

if __name__ == '__main__':
    bad_approach()
    good_approach()
```

#### 陷阱二：过度使用锁

```python
import threading

# ❌ 错误：锁粒度太大
class BadCounter:
    def __init__(self):
        self.value = 0
        self.lock = threading.Lock()
  
    def increment(self):
        with self.lock:
            # 整个方法都在锁内，包括不需要保护的操作
            temp = self.value
            # 模拟一些不需要锁保护的计算
            result = temp + 1
            self.value = result

# ✅ 正确：最小化锁的范围
class GoodCounter:
    def __init__(self):
        self.value = 0
        self.lock = threading.Lock()
  
    def increment(self):
        # 先做不需要保护的计算
        temp_result = 1  # 一些计算
      
        # 只在必要时获取锁
        with self.lock:
            self.value += temp_result

def test_counter(counter_class, name):
    """测试计数器性能"""
    import time
  
    counter = counter_class()
  
    def worker():
        for _ in range(10000):
            counter.increment()
  
    start = time.time()
  
    threads = [threading.Thread(target=worker) for _ in range(10)]
    for t in threads: t.start()
    for t in threads: t.join()
  
    end = time.time()
    print(f"{name}: {end - start:.4f}秒, 最终值: {counter.value}")

if __name__ == '__main__':
    test_counter(BadCounter, "粗粒度锁")
    test_counter(GoodCounter, "细粒度锁")
```

#### 陷阱三：在循环中使用锁

```python
import threading
import time

data = list(range(10000))
lock = threading.Lock()

# ❌ 错误：每次迭代都获取锁
def bad_process():
    """循环内频繁获取锁"""
    start = time.time()
  
    result = []
    for item in data:
        with lock:
            result.append(item * 2)
  
    end = time.time()
    print(f"循环内加锁: {end - start:.4f}秒")

# ✅ 正确：批量处理后一次性加锁
def good_process():
    """批量处理后加锁"""
    start = time.time()
  
    # 先处理数据（不需要锁）
    temp_result = [item * 2 for item in data]
  
    # 一次性更新共享资源
    result = []
    with lock:
        result.extend(temp_result)
  
    end = time.time()
    print(f"批量后加锁: {end - start:.4f}秒")

if __name__ == '__main__':
    bad_process()
    good_process()
```

### 7.3 优化模式

#### 模式一：生产者-消费者（解耦）

```python
import threading
import queue
import time
import random

def optimized_producer_consumer():
    """优化的生产者-消费者模式"""
  
    # 使用有界队列避免内存暴涨
    task_queue = queue.Queue(maxsize=100)
    result_queue = queue.Queue()
  
    def producer(queue, count):
        """生产者：生成任务"""
        for i in range(count):
            task = f"任务{i}"
            queue.put(task)
            time.sleep(0.01)  # 模拟生产时间
        print("生产者完成")
  
    def consumer(task_queue, result_queue, worker_id):
        """消费者：处理任务"""
        while True:
            try:
                task = task_queue.get(timeout=1)
              
                # 处理任务
                time.sleep(random.uniform(0.01, 0.05))
                result = f"{task} 被工作者{worker_id}处理"
              
                result_queue.put(result)
                task_queue.task_done()
              
            except queue.Empty:
                print(f"工作者{worker_id}退出")
                break
  
    start = time.time()
  
    # 启动生产者
    producer_thread = threading.Thread(
        target=producer,
        args=(task_queue, 50)
    )
    producer_thread.start()
  
    # 启动多个消费者
    consumers = []
    for i in range(3):
        t = threading.Thread(
            target=consumer,
            args=(task_queue, result_queue, i)
        )
        consumers.append(t)
        t.start()
  
    # 等待生产者完成
    producer_thread.join()
  
    # 等待所有任务被处理
    task_queue.join()
  
    # 等待消费者退出
    for t in consumers:
        t.join()
  
    end = time.time()
  
    # 收集结果
    results = []
    while not result_queue.empty():
        results.append(result_queue.get())
  
    print(f"\n处理完成:")
    print(f"- 任务总数: 50")
    print(f"- 结果数量: {len(results)}")
    print(f"- 总耗时: {end - start:.2f}秒")

if __name__ == '__main__':
    optimized_producer_consumer()
```

#### 模式二：工作窃取（Work Stealing）

当某些工作线程空闲时，可以从其他线程的任务队列中"窃取"任务。

```python
import threading
import queue
import time
import random

class WorkStealingPool:
    """工作窃取线程池"""
  
    def __init__(self, num_workers):
        self.num_workers = num_workers
        self.worker_queues = [queue.Queue() for _ in range(num_workers)]
        self.workers = []
        self.global_queue = queue.Queue()
        self.completed = []
        self.lock = threading.Lock()
  
    def worker(self, worker_id):
        """工作线程"""
        my_queue = self.worker_queues[worker_id]
      
        while True:
            task = None
          
            # 1. 先尝试从自己的队列获取任务
            try:
                task = my_queue.get_nowait()
            except queue.Empty:
                pass
          
            # 2. 如果自己队列为空，尝试从全局队列获取
            if task is None:
                try:
                    task = self.global_queue.get_nowait()
                except queue.Empty:
                    pass
          
            # 3. 如果全局队列也为空，尝试从其他工作线程"窃取"
            if task is None:
                for i, other_queue in enumerate(self.worker_queues):
                    if i != worker_id:
                        try:
                            task = other_queue.get_nowait()
                            print(f"工作者{worker_id} 从工作者{i}窃取了任务")
                            break
                        except queue.Empty:
                            continue
          
            # 4. 如果所有队列都为空，等待一段时间后退出
            if task is None:
                try:
                    task = self.global_queue.get(timeout=0.5)
                except queue.Empty:
                    print(f"工作者{worker_id} 退出")
                    break
          
            # 处理任务
            if task == "STOP":
                break
          
            result = self.process_task(task, worker_id)
          
            with self.lock:
                self.completed.append(result)
  
    def process_task(self, task, worker_id):
        """处理任务（模拟不同的处理时间）"""
        sleep_time = random.uniform(0.01, 0.1)
        time.sleep(sleep_time)
        return f"{task} 由工作者{worker_id}完成 (耗时{sleep_time:.3f}秒)"
  
    def submit(self, task, preferred_worker=None):
        """提交任务"""
        if preferred_worker is not None:
            self.worker_queues[preferred_worker].put(task)
        else:
            self.global_queue.put(task)
  
    def start(self):
        """启动所有工作线程"""
        for i in range(self.num_workers):
            t = threading.Thread(target=self.worker, args=(i,))
            self.workers.append(t)
            t.start()
  
    def shutdown(self):
        """关闭线程池"""
        # 简化示例：真实实现应先确认所有普通任务完成，再发送停止信号；
        # 否则STOP可能先被空闲线程取走，导致其他工作队列中的任务未处理完。
        # 发送停止信号
        for _ in range(self.num_workers):
            self.global_queue.put("STOP")
      
        # 等待所有工作线程完成
        for t in self.workers:
            t.join()

def test_work_stealing():
    """测试工作窃取模式"""
    pool = WorkStealingPool(num_workers=4)
    pool.start()
  
    # 提交任务（某些工作者的任务更多）
    for i in range(50):
        # 前30个任务都分配给工作者0（模拟不平衡）
        if i < 30:
            pool.submit(f"任务{i}", preferred_worker=0)
        else:
            pool.submit(f"任务{i}")
  
    # 关闭并等待
    pool.shutdown()
  
    print(f"\n完成任务数: {len(pool.completed)}")

if __name__ == '__main__':
    test_work_stealing()
```

#### 模式三：流水线（Pipeline）

将复杂任务分解为多个阶段，每个阶段由不同的线程/进程处理。

```python
import threading
import queue
import time

class Pipeline:
    """流水线模式"""
  
    def __init__(self):
        self.stage1_queue = queue.Queue(maxsize=10)
        self.stage2_queue = queue.Queue(maxsize=10)
        self.stage3_queue = queue.Queue(maxsize=10)
        self.results = []
  
    def stage1_process(self):
        """阶段1：数据获取"""
        for i in range(20):
            data = f"原始数据{i}"
            print(f"阶段1: 获取 {data}")
            self.stage1_queue.put(data)
            time.sleep(0.05)
      
        # 发送结束信号
        self.stage1_queue.put(None)
  
    def stage2_process(self):
        """阶段2：数据处理"""
        while True:
            data = self.stage1_queue.get()
          
            if data is None:
                self.stage2_queue.put(None)
                break
          
            # 处理数据
            processed = f"{data} -> 已处理"
            print(f"  阶段2: 处理 {data}")
            self.stage2_queue.put(processed)
            time.sleep(0.08)
  
    def stage3_process(self):
        """阶段3：数据存储"""
        while True:
            data = self.stage2_queue.get()
          
            if data is None:
                self.stage3_queue.put(None)
                break
          
            # 存储数据
            stored = f"{data} -> 已存储"
            print(f"    阶段3: 存储 {data}")
            self.results.append(stored)
            time.sleep(0.03)
  
    def run(self):
        """运行流水线"""
        # 创建三个阶段的线程
        t1 = threading.Thread(target=self.stage1_process, name="阶段1")
        t2 = threading.Thread(target=self.stage2_process, name="阶段2")
        t3 = threading.Thread(target=self.stage3_process, name="阶段3")
      
        start = time.time()
      
        # 启动所有阶段
        t1.start()
        t2.start()
        t3.start()
      
        # 等待完成
        t1.join()
        t2.join()
        t3.join()
      
        end = time.time()
      
        print(f"\n流水线完成:")
        print(f"- 处理数据: {len(self.results)} 条")
        print(f"- 总耗时: {end - start:.2f}秒")
        print(f"- 如果串行执行需要: {20 * (0.05 + 0.08 + 0.03):.2f}秒")

if __name__ == '__main__':
    pipeline = Pipeline()
    pipeline.run()
```

#### 模式四：Map-Reduce

经典的分布式计算模式，也适用于单机多进程。

```python
import multiprocessing
from functools import reduce
import time

def map_reduce_example():
    """Map-Reduce模式示例：统计词频"""
  
    # 模拟大量文本数据
    documents = [
        "Python is great. Python is powerful.",
        "Java is popular. Java is mature.",
        "Python is simple. Java is complex.",
        "Python and Java are both useful.",
    ] * 100  # 复制100次模拟大数据
  
    def map_function(document):
        """Map阶段：提取单词"""
        words = document.lower().replace('.', '').split()
        word_counts = {}
        for word in words:
            word_counts[word] = word_counts.get(word, 0) + 1
        return word_counts
  
    def reduce_function(count1, count2):
        """Reduce阶段：合并统计"""
        result = count1.copy()
        for word, count in count2.items():
            result[word] = result.get(word, 0) + count
        return result
  
    start = time.time()
  
    # Map阶段：并行处理
    with multiprocessing.Pool(processes=4) as pool:
        mapped_results = pool.map(map_function, documents)
  
    # Reduce阶段：合并结果
    final_result = reduce(reduce_function, mapped_results)
  
    end = time.time()
  
    # 显示结果
    print("词频统计结果（前10个）:")
    sorted_words = sorted(final_result.items(), key=lambda x: x[1], reverse=True)
    for word, count in sorted_words[:10]:
        print(f"  {word}: {count}")
  
    print(f"\n总耗时: {end - start:.2f}秒")

if __name__ == '__main__':
    map_reduce_example()
```

#### 模式五：Future模式（承诺/期货）

提交任务后立即返回一个"期货"对象，稍后可以获取结果。

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time
import random

def fetch_data(url):
    """模拟数据获取（耗时不同）"""
    time.sleep(random.uniform(0.5, 2.0))
    return f"来自 {url} 的数据 (大小: {random.randint(100, 1000)}KB)"

def future_pattern_demo():
    """Future模式演示"""
  
    urls = [f"http://example.com/page{i}" for i in range(10)]
  
    print("=== 提交所有任务 ===")
    start = time.time()
  
    with ThreadPoolExecutor(max_workers=5) as executor:
        # 提交任务，返回Future对象
        future_to_url = {
            executor.submit(fetch_data, url): url 
            for url in urls
        }
      
        print(f"已提交 {len(future_to_url)} 个任务\n")
      
        # 按完成顺序处理结果
        print("=== 按完成顺序获取结果 ===")
        for future in as_completed(future_to_url):
            url = future_to_url[future]
            try:
                result = future.result()
                print(f"✓ {url}: {result}")
            except Exception as e:
                print(f"✗ {url}: 错误 - {e}")
  
    end = time.time()
    print(f"\n总耗时: {end - start:.2f}秒")

if __name__ == '__main__':
    future_pattern_demo()
```

### 7.4 批量操作优化

#### 批量数据库操作

```python
import threading
import time
from typing import List

class DatabaseMock:
    """模拟数据库"""
  
    def __init__(self):
        self.data = []
        self.lock = threading.Lock()
  
    def insert_one(self, item):
        """单条插入（慢）"""
        time.sleep(0.01)  # 模拟网络延迟
        with self.lock:
            self.data.append(item)
  
    def insert_many(self, items: List):
        """批量插入（快）"""
        time.sleep(0.01)  # 只有一次网络延迟
        with self.lock:
            self.data.extend(items)

def compare_batch_operations():
    """对比单条操作vs批量操作"""
  
    items = [f"数据{i}" for i in range(1000)]
  
    # 方法1：单条插入
    print("=== 方法1: 单条插入 ===")
    db1 = DatabaseMock()
    start = time.time()
  
    for item in items:
        db1.insert_one(item)
  
    end = time.time()
    print(f"耗时: {end - start:.2f}秒")
    print(f"插入数量: {len(db1.data)}\n")
  
    # 方法2：批量插入
    print("=== 方法2: 批量插入 ===")
    db2 = DatabaseMock()
    start = time.time()
  
    batch_size = 100
    for i in range(0, len(items), batch_size):
        batch = items[i:i + batch_size]
        db2.insert_many(batch)
  
    end = time.time()
    print(f"耗时: {end - start:.2f}秒")
    print(f"插入数量: {len(db2.data)}")
    print(f"批次数: {len(items) // batch_size}")

if __name__ == '__main__':
    compare_batch_operations()
```

**典型输出**：
```
=== 方法1: 单条插入 ===
耗时: 10.15秒
插入数量: 1000

=== 方法2: 批量插入 ===
耗时: 0.11秒
插入数量: 1000
批次数: 10
```

#### 批量网络请求

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def fetch_url(url):
    """模拟网络请求"""
    time.sleep(0.1)
    return f"内容来自 {url}"

def batch_fetch_with_limit():
    """限制并发数的批量请求"""
  
    urls = [f"http://example.com/api/{i}" for i in range(100)]
  
    print("=== 批量请求（限制并发数） ===")
    start = time.time()
  
    results = []
  
    # 使用线程池限制并发数为10
    with ThreadPoolExecutor(max_workers=10) as executor:
        # 提交所有任务
        futures = {executor.submit(fetch_url, url): url for url in urls}
      
        # 收集结果
        for future in as_completed(futures):
            url = futures[future]
            try:
                result = future.result()
                results.append(result)
              
                # 显示进度
                if len(results) % 10 == 0:
                    print(f"已完成: {len(results)}/{len(urls)}")
                  
            except Exception as e:
                print(f"错误 {url}: {e}")
  
    end = time.time()
  
    print(f"\n完成:")
    print(f"- 总请求: {len(urls)}")
    print(f"- 成功: {len(results)}")
    print(f"- 耗时: {end - start:.2f}秒")
    print(f"- 如果串行: {len(urls) * 0.1:.2f}秒")

if __name__ == '__main__':
    batch_fetch_with_limit()
```

### 7.5 缓存与避免重复计算

```python
import threading
import time
from functools import lru_cache

class CachedComputation:
    """带缓存的计算类"""
  
    def __init__(self):
        self.cache = {}
        self.lock = threading.Lock()
        self.hit_count = 0
        self.miss_count = 0
  
    def expensive_computation(self, n):
        """耗时计算"""
        # 检查缓存
        with self.lock:
            if n in self.cache:
                self.hit_count += 1
                return self.cache[n]
          
            self.miss_count += 1
      
        # 执行计算
        time.sleep(0.1)  # 模拟耗时操作
        result = sum(i * i for i in range(n))
      
        # 存入缓存
        with self.lock:
            self.cache[n] = result
      
        return result
  
    def stats(self):
        """显示缓存统计"""
        total = self.hit_count + self.miss_count
        hit_rate = self.hit_count / total if total > 0 else 0
      
        print(f"缓存统计:")
        print(f"  命中: {self.hit_count}")
        print(f"  未命中: {self.miss_count}")
        print(f"  命中率: {hit_rate:.2%}")

def test_cache():
    """测试缓存效果"""
  
    comp = CachedComputation()
  
    # 模拟多个线程访问相同的数据
    def worker(values):
        for v in values:
            result = comp.expensive_computation(v)
  
    # 生成有重复的数据
    import random
    data_sets = [
        [random.randint(1, 20) for _ in range(50)]
        for _ in range(10)
    ]
  
    start = time.time()
  
    threads = []
    for data in data_sets:
        t = threading.Thread(target=worker, args=(data,))
        threads.append(t)
        t.start()
  
    for t in threads:
        t.join()
  
    end = time.time()
  
    comp.stats()
    print(f"\n总耗时: {end - start:.2f}秒")

# 使用装饰器实现缓存（更简单）
@lru_cache(maxsize=128)
def fibonacci(n):
    """斐波那契数列（带缓存）"""
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

def test_lru_cache():
    """测试LRU缓存"""
  
    print("\n=== 使用@lru_cache装饰器 ===")
  
    start = time.time()
  
    # 计算多个斐波那契数
    results = [fibonacci(i) for i in range(35)]
  
    end = time.time()
  
    print(f"计算 fibonacci(0) 到 fibonacci(34)")
    print(f"耗时: {end - start:.4f}秒")
    print(f"缓存信息: {fibonacci.cache_info()}")

if __name__ == '__main__':
    test_cache()
    test_lru_cache()
```

---

## 第八章：分布式进程（Distributed Process）

### 8.1 什么是分布式进程？

分布式进程允许Python程序在多台机器上运行，突破单机限制。

**应用场景**：
- 大规模数据处理
- 分布式爬虫
- 科学计算
- 机器学习训练

### 8.2 使用multiprocessing.managers实现

#### 简单的分布式任务队列

```python
# ===== 服务器端 (server.py) =====
import multiprocessing
from multiprocessing.managers import BaseManager
import queue

# 创建任务队列和结果队列
task_queue = queue.Queue()
result_queue = queue.Queue()

class QueueManager(BaseManager):
    """队列管理器"""
    pass

# 注册队列到管理器
QueueManager.register('get_task_queue', callable=lambda: task_queue)
QueueManager.register('get_result_queue', callable=lambda: result_queue)

def start_server():
    """启动服务器"""
    # 创建管理器实例
    manager = QueueManager(address=('', 5000), authkey=b'secret')
  
    # 启动管理器服务
    server = manager.get_server()
  
    print("服务器启动在端口 5000...")
    print("等待客户端连接...")
  
    # 添加任务到队列
    for i in range(20):
        task_queue.put(f"任务{i}")
  
    print(f"已添加 20 个任务")
  
    # 运行服务器
    server.serve_forever()

if __name__ == '__main__':
    start_server()
```

```python
# ===== 客户端 (client.py) =====
import multiprocessing
from multiprocessing.managers import BaseManager
import time

class QueueManager(BaseManager):
    """队列管理器（客户端）"""
    pass

# 注册队列（客户端只需要声明，不需要callable）
QueueManager.register('get_task_queue')
QueueManager.register('get_result_queue')

def process_task(task):
    """处理任务"""
    time.sleep(1)  # 模拟处理
    return f"{task} 已完成"

def start_client():
    """启动客户端"""
    # 连接到服务器
    manager = QueueManager(address=('localhost', 5000), authkey=b'secret')
    manager.connect()
  
    print("已连接到服务器")
  
    # 获取队列
    task_queue = manager.get_task_queue()
    result_queue = manager.get_result_queue()
  
    # 处理任务
    while True:
        try:
            task = task_queue.get(timeout=5)
            print(f"获取任务: {task}")
          
            result = process_task(task)
            result_queue.put(result)
          
            print(f"完成任务: {result}")
          
        except:
            print("没有更多任务，退出")
            break

if __name__ == '__main__':
    start_client()
```

#### 收集结果的监控程序

```python
# ===== 监控程序 (monitor.py) =====
from multiprocessing.managers import BaseManager
import time

class QueueManager(BaseManager):
    pass

QueueManager.register('get_task_queue')
QueueManager.register('get_result_queue')

def monitor_results():
    """监控任务结果"""
    manager = QueueManager(address=('localhost', 5000), authkey=b'secret')
    manager.connect()
  
    result_queue = manager.get_result_queue()
  
    print("开始监控结果...")
  
    results = []
    while len(results) < 20:
        try:
            result = result_queue.get(timeout=2)
            results.append(result)
            print(f"收到结果: {result} (总计: {len(results)}/20)")
        except:
            print("等待更多结果...")
            time.sleep(1)
  
    print("\n所有任务完成！")
    for r in results:
        print(f"  - {r}")

if __name__ == '__main__':
    monitor_results()
```

**使用方法**：
1. 在一台机器上运行 `python server.py`
2. 在其他机器上运行 `python client.py`（可以多个）
3. 在任意机器上运行 `python monitor.py` 查看结果

### 8.3 使用Celery实现分布式任务

Celery是专业的分布式任务队列框架。

```python
# 安装: pip install celery redis

# ===== celery_config.py =====
from celery import Celery

# 创建Celery应用
app = Celery(
    'tasks',
    broker='redis://localhost:6379/0',  # 消息代理
    backend='redis://localhost:6379/0'   # 结果后端
)

# 配置
app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='Asia/Shanghai',
    enable_utc=True,
)

@app.task
def add(x, y):
    """简单的加法任务"""
    import time
    time.sleep(2)  # 模拟耗时操作
    return x + y

@app.task
def process_data(data):
    """数据处理任务"""
    import time
    time.sleep(1)
    return f"处理完成: {data}"
```

```python
# ===== 提交任务 (submit_tasks.py) =====
from celery_config import add, process_data
import time

def submit_tasks():
    """提交任务到Celery"""
  
    print("=== 提交任务 ===")
  
    # 异步执行任务
    result1 = add.delay(4, 6)
    result2 = add.delay(10, 20)
  
    print(f"任务1 ID: {result1.id}")
    print(f"任务2 ID: {result2.id}")
  
    # 等待结果
    print("\n等待结果...")
    print(f"任务1 结果: {result1.get(timeout=10)}")
    print(f"任务2 结果: {result2.get(timeout=10)}")
  
    # 批量提交任务
    print("\n=== 批量提交任务 ===")
    tasks = []
    for i in range(10):
        task = process_data.delay(f"数据{i}")
        tasks.append(task)
  
    # 等待所有任务完成
    print("等待所有任务完成...")
    for i, task in enumerate(tasks):
        result = task.get(timeout=30)
        print(f"任务{i}: {result}")

if __name__ == '__main__':
    submit_tasks()
```

**启动Celery Worker**：
```bash
# 在命令行运行
celery -A celery_config worker --loglevel=info
```

### 8.4 使用Ray实现分布式计算

Ray是现代的分布式计算框架，特别适合机器学习。

```python
# 安装: pip install ray

import ray
import time

# 初始化Ray
ray.init()

@ray.remote
def compute_heavy_task(x):
    """CPU密集型任务"""
    time.sleep(1)
    return sum(i * i for i in range(x))

@ray.remote
class Counter:
    """分布式计数器"""
  
    def __init__(self):
        self.value = 0
  
    def increment(self):
        self.value += 1
        return self.value
  
    def get_value(self):
        return self.value

def ray_example():
    """Ray使用示例"""
  
    print("=== Ray远程函数 ===")
  
    # 提交多个任务
    futures = [compute_heavy_task.remote(1000000) for _ in range(10)]
  
    # 等待结果
    results = ray.get(futures)
    print(f"结果: {results[0:3]}...")
  
    print("\n=== Ray Actor（分布式对象） ===")
  
    # 创建分布式对象
    counter = Counter.remote()
  
    # 调用方法
    futures = [counter.increment.remote() for _ in range(10)]
    results = ray.get(futures)
  
    # 获取最终值
    final_value = ray.get(counter.get_value.remote())
    print(f"最终计数: {final_value}")
  
    # 关闭Ray
    ray.shutdown()

if __name__ == '__main__':
    ray_example()
```

---

## 第九章：实战项目

### 9.1 多线程网络爬虫

```python
import threading
import queue
import requests
from bs4 import BeautifulSoup
import time
from urllib.parse import urljoin, urlparse
import logging

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(threadName)s - %(message)s'
)

class WebCrawler:
    """多线程网络爬虫"""
  
    def __init__(self, start_urls, max_workers=5, max_depth=2):
        self.start_urls = start_urls
        self.max_workers = max_workers
        self.max_depth = max_depth
      
        # 任务队列：存储 (url, depth)
        self.task_queue = queue.Queue()
      
        # 结果队列
        self.result_queue = queue.Queue()
      
        # 已访问的URL集合（线程安全）
        self.visited = set()
        self.visited_lock = threading.Lock()
      
        # 统计信息
        self.stats = {
            'total_pages': 0,
            'successful': 0,
            'failed': 0
        }
        self.stats_lock = threading.Lock()
  
    def is_valid_url(self, url):
        """检查URL是否有效"""
        try:
            parsed = urlparse(url)
            return bool(parsed.netloc) and bool(parsed.scheme)
        except:
            return False
  
    def get_domain(self, url):
        """获取URL的域名"""
        return urlparse(url).netloc
  
    def fetch_page(self, url):
        """获取网页内容"""
        try:
            headers = {
                'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
            }
            response = requests.get(url, headers=headers, timeout=10)
            response.raise_for_status()
            return response.text
        except Exception as e:
            logging.error(f"获取 {url} 失败: {e}")
            return None
  
    def parse_links(self, html, base_url):
        """解析页面中的链接"""
        try:
            soup = BeautifulSoup(html, 'html.parser')
            links = []
          
            for link in soup.find_all('a', href=True):
                href = link['href']
                # 转换为绝对URL
                absolute_url = urljoin(base_url, href)
              
                # 只爬取同域名的链接
                if (self.is_valid_url(absolute_url) and 
                    self.get_domain(absolute_url) == self.get_domain(base_url)):
                    links.append(absolute_url)
          
            return links
        except Exception as e:
            logging.error(f"解析链接失败: {e}")
            return []
  
    def extract_content(self, html, url):
        """提取页面内容"""
        try:
            soup = BeautifulSoup(html, 'html.parser')
          
            # 提取标题
            title = soup.find('title')
            title_text = title.string if title else 'No title'
          
            # 提取文本内容（简化版）
            paragraphs = soup.find_all('p')
            content = ' '.join([p.get_text() for p in paragraphs[:5]])  # 前5段
          
            return {
                'url': url,
                'title': title_text,
                'content': content[:200],  # 前200字符
                'content_length': len(content)
            }
        except Exception as e:
            logging.error(f"提取内容失败: {e}")
            return None
  
    def worker(self):
        """工作线程"""
        while True:
            try:
                # 从队列获取任务，设置超时避免死锁
                url, depth = self.task_queue.get(timeout=5)
              
                # 检查是否已访问
                with self.visited_lock:
                    if url in self.visited:
                        self.task_queue.task_done()
                        continue
                    self.visited.add(url)
              
                logging.info(f"正在爬取: {url} (深度: {depth})")
              
                # 获取页面
                html = self.fetch_page(url)
              
                if html:
                    # 提取内容
                    content = self.extract_content(html, url)
                    if content:
                        self.result_queue.put(content)
                      
                        with self.stats_lock:
                            self.stats['successful'] += 1
                  
                    # 如果未达到最大深度，继续爬取链接
                    if depth < self.max_depth:
                        links = self.parse_links(html, url)
                        for link in links[:10]:  # 限制每页最多10个链接
                            with self.visited_lock:
                                if link not in self.visited:
                                    self.task_queue.put((link, depth + 1))
                else:
                    with self.stats_lock:
                        self.stats['failed'] += 1
              
                # 更新统计
                with self.stats_lock:
                    self.stats['total_pages'] += 1
              
                # 礼貌延迟
                time.sleep(0.5)
              
                # 标记任务完成
                self.task_queue.task_done()
              
            except queue.Empty:
                # 队列为空，退出
                logging.info(f"线程退出")
                break
            except Exception as e:
                logging.error(f"工作线程错误: {e}")
                self.task_queue.task_done()
  
    def crawl(self):
        """开始爬取"""
        logging.info(f"开始爬取，工作线程数: {self.max_workers}")
      
        # 添加起始URL
        for url in self.start_urls:
            self.task_queue.put((url, 0))
      
        start_time = time.time()
      
        # 创建并启动工作线程
        workers = []
        for i in range(self.max_workers):
            t = threading.Thread(target=self.worker, name=f"Worker-{i}")
            t.start()
            workers.append(t)
      
        # 等待所有任务完成
        self.task_queue.join()
      
        # 等待所有线程退出
        for t in workers:
            t.join()
      
        end_time = time.time()
      
        # 收集结果
        results = []
        while not self.result_queue.empty():
            results.append(self.result_queue.get())
      
        # 打印统计信息
        logging.info("\n=== 爬取完成 ===")
        logging.info(f"总页面数: {self.stats['total_pages']}")
        logging.info(f"成功: {self.stats['successful']}")
        logging.info(f"失败: {self.stats['failed']}")
        logging.info(f"耗时: {end_time - start_time:.2f}秒")
      
        return results

def main():
    """主函数"""
    # 起始URL（请替换为实际URL）
    start_urls = [
        'https://example.com',
    ]
  
    # 创建爬虫
    crawler = WebCrawler(
        start_urls=start_urls,
        max_workers=5,
        max_depth=2
    )
  
    # 开始爬取
    results = crawler.crawl()
  
    # 显示部分结果
    print("\n=== 爬取结果（前5条）===")
    for i, result in enumerate(results[:5], 1):
        print(f"\n{i}. {result['title']}")
        print(f"   URL: {result['url']}")
        print(f"   内容长度: {result['content_length']}")
        print(f"   预览: {result['content'][:100]}...")

if __name__ == '__main__':
    main()
```

### 9.2 多进程图像处理器

```python
import multiprocessing
from multiprocessing import Pool
import os
from PIL import Image
import time
from pathlib import Path

class ImageProcessor:
    """多进程图像处理器"""
  
    def __init__(self, input_dir, output_dir, num_processes=None):
        self.input_dir = Path(input_dir)
        self.output_dir = Path(output_dir)
        self.num_processes = num_processes or multiprocessing.cpu_count()
      
        # 创建输出目录
        self.output_dir.mkdir(parents=True, exist_ok=True)
  
    def get_image_files(self):
        """获取所有图像文件"""
        extensions = ['.jpg', '.jpeg', '.png', '.bmp', '.gif']
        image_files = []
      
        for ext in extensions:
            image_files.extend(self.input_dir.glob(f'*{ext}'))
            image_files.extend(self.input_dir.glob(f'*{ext.upper()}'))
      
        return image_files
  
    @staticmethod
    def resize_image(args):
        """调整图像大小（静态方法供进程池使用）"""
        input_path, output_path, size = args
      
        try:
            # 打开图像
            img = Image.open(input_path)
          
            # 计算新尺寸（保持宽高比）
            img.thumbnail(size, Image.Resampling.LANCZOS)
          
            # 保存
            img.save(output_path, quality=85, optimize=True)
          
            return {
                'status': 'success',
                'input': str(input_path),
                'output': str(output_path),
                'original_size': Image.open(input_path).size,
                'new_size': img.size
            }
        except Exception as e:
            return {
                'status': 'error',
                'input': str(input_path),
                'error': str(e)
            }
  
    @staticmethod
    def add_watermark(args):
        """添加水印"""
        input_path, output_path, watermark_text = args
      
        try:
            from PIL import ImageDraw, ImageFont
          
            # 打开图像
            img = Image.open(input_path)
          
            # 创建绘图对象
            draw = ImageDraw.Draw(img)
          
            # 设置字体（使用默认字体）
            try:
                font = ImageFont.truetype("arial.ttf", 36)
            except:
                font = ImageFont.load_default()
          
            # 计算水印位置（右下角）
            text_bbox = draw.textbbox((0, 0), watermark_text, font=font)
            text_width = text_bbox[2] - text_bbox[0]
            text_height = text_bbox[3] - text_bbox[1]
          
            x = img.width - text_width - 10
            y = img.height - text_height - 10
          
            # 绘制水印（带阴影效果）
            draw.text((x+2, y+2), watermark_text, fill='black', font=font)
            draw.text((x, y), watermark_text, fill='white', font=font)
          
            # 保存
            img.save(output_path, quality=85)
          
            return {
                'status': 'success',
                'input': str(input_path),
                'output': str(output_path)
            }
        except Exception as e:
            return {
                'status': 'error',
                'input': str(input_path),
                'error': str(e)
            }
  
    @staticmethod
    def convert_format(args):
        """转换图像格式"""
        input_path, output_path, target_format = args
      
        try:
            # 打开并转换
            img = Image.open(input_path)
          
            # 如果是PNG转JPG，需要处理透明度
            if target_format.lower() == 'jpg' and img.mode in ('RGBA', 'LA', 'P'):
                background = Image.new('RGB', img.size, (255, 255, 255))
                if img.mode == 'P':
                    img = img.convert('RGBA')
                background.paste(img, mask=img.split()[-1])
                img = background
          
            # 保存为新格式
            img.save(output_path, format=target_format, quality=85)
          
            return {
                'status': 'success',
                'input': str(input_path),
                'output': str(output_path),
                'format': target_format
            }
        except Exception as e:
            return {
                'status': 'error',
                'input': str(input_path),
                'error': str(e)
            }
  
    def batch_resize(self, size=(800, 600)):
        """批量调整图像大小"""
        print(f"\n=== 批量调整图像大小 ===")
        print(f"目标尺寸: {size}")
      
        image_files = self.get_image_files()
        print(f"找到 {len(image_files)} 个图像文件")
      
        # 准备任务参数
        tasks = []
        for img_path in image_files:
            output_path = self.output_dir / f"resized_{img_path.name}"
            tasks.append((img_path, output_path, size))
      
        # 使用进程池处理
        start_time = time.time()
      
        with Pool(processes=self.num_processes) as pool:
            results = pool.map(self.resize_image, tasks)
      
        end_time = time.time()
      
        # 统计结果
        success_count = sum(1 for r in results if r['status'] == 'success')
      
        print(f"\n处理完成:")
        print(f"- 成功: {success_count}")
        print(f"- 失败: {len(results) - success_count}")
        print(f"- 耗时: {end_time - start_time:.2f}秒")
      
        return results
  
    def batch_watermark(self, text="© 2024"):
        """批量添加水印"""
        print(f"\n=== 批量添加水印 ===")
        print(f"水印文字: {text}")
      
        image_files = self.get_image_files()
        print(f"找到 {len(image_files)} 个图像文件")
      
        # 准备任务参数
        tasks = []
        for img_path in image_files:
            output_path = self.output_dir / f"watermarked_{img_path.name}"
            tasks.append((img_path, output_path, text))
      
        # 使用进程池处理
        start_time = time.time()
      
        with Pool(processes=self.num_processes) as pool:
            results = pool.map(self.add_watermark, tasks)
      
        end_time = time.time()
      
        # 统计结果
        success_count = sum(1 for r in results if r['status'] == 'success')
      
        print(f"\n处理完成:")
        print(f"- 成功: {success_count}")
        print(f"- 失败: {len(results) - success_count}")
        print(f"- 耗时: {end_time - start_time:.2f}秒")
      
        return results
  
    def batch_convert(self, target_format='JPEG'):
        """批量转换格式"""
        print(f"\n=== 批量转换格式 ===")
        print(f"目标格式: {target_format}")
      
        image_files = self.get_image_files()
        print(f"找到 {len(image_files)} 个图像文件")
      
        # 准备任务参数
        tasks = []
        ext = '.jpg' if target_format.upper() == 'JPEG' else f'.{target_format.lower()}'
      
        for img_path in image_files:
            output_path = self.output_dir / f"{img_path.stem}{ext}"
            tasks.append((img_path, output_path, target_format))
      
        # 使用进程池处理
        start_time = time.time()
      
        with Pool(processes=self.num_processes) as pool:
            results = pool.map(self.convert_format, tasks)
      
        end_time = time.time()
      
        # 统计结果
        success_count = sum(1 for r in results if r['status'] == 'success')
      
        print(f"\n处理完成:")
        print(f"- 成功: {success_count}")
        print(f"- 失败: {len(results) - success_count}")
        print(f"- 耗时: {end_time - start_time:.2f}秒")
      
        return results

def create_sample_images(output_dir, count=10):
    """创建示例图像用于测试"""
    from PIL import ImageDraw
    import random
  
    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)
  
    print(f"创建 {count} 个示例图像...")
  
    for i in range(count):
        # 创建随机颜色的图像
        width, height = random.randint(800, 1200), random.randint(600, 900)
        color = (random.randint(0, 255), random.randint(0, 255), random.randint(0, 255))
      
        img = Image.new('RGB', (width, height), color)
        draw = ImageDraw.Draw(img)
      
        # 画一些随机图形
        for _ in range(5):
            x1, y1 = random.randint(0, width), random.randint(0, height)
            x2, y2 = random.randint(0, width), random.randint(0, height)
            shape_color = (random.randint(0, 255), random.randint(0, 255), random.randint(0, 255))
            draw.rectangle([x1, y1, x2, y2], fill=shape_color)
      
        # 保存
        img.save(output_path / f"sample_{i}.jpg")
  
    print(f"示例图像已创建在 {output_dir}")

def main():
    """主函数"""
    # 创建示例图像
    create_sample_images('sample_images', count=20)
  
    # 创建图像处理器
    processor = ImageProcessor(
        input_dir='sample_images',
        output_dir='processed_images',
        num_processes=4
    )
  
    # 批量调整大小
    processor.batch_resize(size=(800, 600))
  
    # 批量添加水印
    processor.batch_watermark(text="© My Photo")
  
    # 批量转换格式
    # processor.batch_convert(target_format='PNG')

if __name__ == '__main__':
    main()
```

### 9.3 并发下载管理器

```python
import threading
import queue
import requests
import time
from pathlib import Path
from urllib.parse import urlparse
import hashlib

class DownloadManager:
    """并发下载管理器"""
  
    def __init__(self, max_workers=5, output_dir='downloads'):
        self.max_workers = max_workers
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(parents=True, exist_ok=True)
      
        # 下载队列
        self.download_queue = queue.Queue()
      
        # 结果队列
        self.result_queue = queue.Queue()
      
        # 统计信息
        self.stats = {
            'total': 0,
            'completed': 0,
            'failed': 0,
            'total_bytes': 0
        }
        self.stats_lock = threading.Lock()
      
        # 进度信息
        self.progress = {}
        self.progress_lock = threading.Lock()
  
    def get_filename_from_url(self, url):
        """从URL提取文件名"""
        parsed = urlparse(url)
        filename = Path(parsed.path).name
      
        if not filename:
            # 使用URL的hash作为文件名
            filename = hashlib.md5(url.encode()).hexdigest()
      
        return filename
  
    def download_file(self, url, filename=None):
        """下载单个文件"""
        try:
            # 确定文件名
            if filename is None:
                filename = self.get_filename_from_url(url)
          
            output_path = self.output_dir / filename
          
            # 开始下载
            print(f"[{threading.current_thread().name}] 开始下载: {url}")
          
            response = requests.get(url, stream=True, timeout=30)
            response.raise_for_status()
          
            # 获取文件大小
            total_size = int(response.headers.get('content-length', 0))
          
            # 初始化进度
            with self.progress_lock:
                self.progress[url] = {
                    'downloaded': 0,
                    'total': total_size,
                    'filename': filename
                }
          
            # 分块下载
            downloaded_size = 0
            chunk_size = 8192
          
            with open(output_path, 'wb') as f:
                for chunk in response.iter_content(chunk_size=chunk_size):
                    if chunk:
                        f.write(chunk)
                        downloaded_size += len(chunk)
                      
                        # 更新进度
                        with self.progress_lock:
                            self.progress[url]['downloaded'] = downloaded_size
          
            # 更新统计
            with self.stats_lock:
                self.stats['completed'] += 1
                self.stats['total_bytes'] += downloaded_size
          
            with self.progress_lock:
                self.progress.pop(url, None)
          
            print(f"[{threading.current_thread().name}] 完成: {filename} ({downloaded_size} bytes)")
          
            return {
                'status': 'success',
                'url': url,
                'filename': filename,
                'size': downloaded_size,
                'path': str(output_path)
            }
          
        except Exception as e:
            with self.stats_lock:
                self.stats['failed'] += 1
          
            with self.progress_lock:
                self.progress.pop(url, None)
          
            print(f"[{threading.current_thread().name}] 失败: {url} - {e}")
          
            return {
                'status': 'error',
                'url': url,
                'error': str(e)
            }
  
    def worker(self):
        """工作线程"""
        while True:
            try:
                # 从队列获取任务
                task = self.download_queue.get(timeout=5)
              
                if task is None:  # 停止信号
                    break
              
                url, filename = task
              
                # 下载文件
                result = self.download_file(url, filename)
                self.result_queue.put(result)
              
                # 标记任务完成
                self.download_queue.task_done()
              
                # 礼貌延迟
                time.sleep(0.5)
              
            except queue.Empty:
                break
            except Exception as e:
                print(f"Worker错误: {e}")
  
    def progress_monitor(self):
        """进度监控线程"""
        while True:
            time.sleep(2)
          
            with self.progress_lock:
                if not self.progress:
                    break
              
                print("\n=== 下载进度 ===")
                for url, info in list(self.progress.items()):
                    downloaded = info['downloaded']
                    total = info['total']
                    filename = info['filename']
                  
                    if total > 0:
                        percent = (downloaded / total) * 100
                        print(f"{filename}: {downloaded}/{total} ({percent:.1f}%)")
                    else:
                        print(f"{filename}: {downloaded} bytes")
  
    def download_all(self, urls):
        """批量下载"""
        print(f"=== 开始批量下载 ===")
        print(f"文件数量: {len(urls)}")
        print(f"工作线程: {self.max_workers}\n")
      
        # 更新统计
        with self.stats_lock:
            self.stats['total'] = len(urls)
      
        # 添加任务到队列
        for url in urls:
            if isinstance(url, tuple):
                self.download_queue.put(url)
            else:
                self.download_queue.put((url, None))
      
        start_time = time.time()
      
        # 启动工作线程
        workers = []
        for i in range(self.max_workers):
            t = threading.Thread(target=self.worker, name=f"Downloader-{i}")
            t.start()
            workers.append(t)
      
        # 启动进度监控线程
        monitor = threading.Thread(target=self.progress_monitor, name="Monitor")
        monitor.start()
      
        # 等待所有任务完成
        self.download_queue.join()
      
        # 发送停止信号
        for _ in range(self.max_workers):
            self.download_queue.put(None)
      
        # 等待所有线程结束
        for t in workers:
            t.join()
      
        monitor.join()
      
        end_time = time.time()
      
        # 收集结果
        results = []
        while not self.result_queue.empty():
            results.append(self.result_queue.get())
      
        # 打印统计信息
        print("\n=== 下载完成 ===")
        print(f"总数: {self.stats['total']}")
        print(f"成功: {self.stats['completed']}")
        print(f"失败: {self.stats['failed']}")
        print(f"总大小: {self.stats['total_bytes'] / 1024 / 1024:.2f} MB")
        print(f"耗时: {end_time - start_time:.2f}秒")
        print(f"平均速度: {self.stats['total_bytes'] / (end_time - start_time) / 1024:.2f} KB/s")
      
        return results

def main():
    """主函数"""
    # 示例下载链接（请替换为实际链接）
    urls = [
        'https://www.python.org/static/img/python-logo.png',
        'https://www.python.org/static/favicon.ico',
        # 添加更多URL...
    ]
  
    # 创建下载管理器
    manager = DownloadManager(max_workers=3, output_dir='downloads')
  
    # 开始下载
    results = manager.download_all(urls)
  
    # 显示结果
    print("\n=== 下载详情 ===")
    for result in results:
        if result['status'] == 'success':
            print(f"✓ {result['filename']}: {result['size']} bytes")
        else:
            print(f"✗ {result['url']}: {result['error']}")

if __name__ == '__main__':
    main()
```

### 9.4 实时日志分析系统

```python
import threading
import queue
import time
import random
from datetime import datetime
from collections import defaultdict, deque
import re

class LogAnalyzer:
    """实时日志分析系统"""
    
    def __init__(self, num_workers=3):
        self.num_workers = num_workers
        
        # 日志队列
        self.log_queue = queue.Queue(maxsize=1000)
        
        # 分析结果
        self.results = {
            'total_logs': 0,
            'error_count': 0,
            'warning_count': 0,
            'info_count': 0,
            'ip_stats': defaultdict(int),
            'url_stats': defaultdict(int),
            'error_messages': deque(maxlen=10),  # 最近10条错误
            'response_times': []
        }
        self.results_lock = threading.Lock()
        
        # 运行标志
        self.running = False
    
    def parse_log_line(self, log_line):
        """解析日志行"""
        try:
            # 示例日志格式: 
            # 2024-01-15 10:30:45 INFO 192.168.1.100 GET /api/users 200 0.123s
            # 2024-01-15 10:30:46 ERROR 192.168.1.101 GET /api/data 500 0.456s Database connection failed
            
            pattern = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (\w+) ([\d\.]+) (\w+) ([\w/]+) (\d+) ([\d\.]+)s(.*)'
            match = re.match(pattern, log_line)
            
            if match:
                timestamp, level, ip, method, url, status, response_time, message = match.groups()
                
                return {
                    'timestamp': timestamp,
                    'level': level,
                    'ip': ip,
                    'method': method,
                    'url': url,
                    'status': int(status),
                    'response_time': float(response_time),
                    'message': message.strip()
                }
            
            return None
            
        except Exception as e:
            print(f"解析日志失败: {e}")
            return None
    
    def analyze_log(self, log_data):
        """分析日志数据"""
        if not log_data:
            return
        
        with self.results_lock:
            # 更新总数
            self.results['total_logs'] += 1
            
            # 按级别统计
            level = log_data['level']
            if level == 'ERROR':
                self.results['error_count'] += 1
                self.results['error_messages'].append({
                    'timestamp': log_data['timestamp'],
                    'url': log_data['url'],
                    'message': log_data['message']
                })
            elif level == 'WARNING':
                self.results['warning_count'] += 1
            elif level == 'INFO':
                self.results['info_count'] += 1
            
            # IP统计
            self.results['ip_stats'][log_data['ip']] += 1
            
            # URL统计
            self.results['url_stats'][log_data['url']] += 1
            
            # 响应时间统计
            self.results['response_times'].append(log_data['response_time'])
            
            # 只保留最近10000条响应时间
            if len(self.results['response_times']) > 10000:
                self.results['response_times'] = self.results['response_times'][-10000:]
    
    def worker(self):
        """日志分析工作线程"""
        while self.running:
            got_item = False
            try:
                # 从队列获取日志
                log_line = self.log_queue.get(timeout=1)
                got_item = True
                
                # 解析并分析
                log_data = self.parse_log_line(log_line)
                if log_data:
                    self.analyze_log(log_data)
                
            except queue.Empty:
                continue
            except Exception as e:
                print(f"Worker错误: {e}")
            finally:
                if got_item:
                    self.log_queue.task_done()
    
    def reporter(self):
        """报告线程：定期输出统计信息"""
        while self.running:
            time.sleep(5)  # 每5秒报告一次
            
            with self.results_lock:
                self.print_report()
    
    def print_report(self):
        """打印分析报告"""
        print("\n" + "="*60)
        print(f"实时日志分析报告 - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
        print("="*60)
        
        # 基本统计
        print(f"\n【总体统计】")
        print(f"  总日志数: {self.results['total_logs']}")
        print(f"  INFO: {self.results['info_count']}")
        print(f"  WARNING: {self.results['warning_count']}")
        print(f"  ERROR: {self.results['error_count']}")
        
        # 响应时间统计
        if self.results['response_times']:
            avg_time = sum(self.results['response_times']) / len(self.results['response_times'])
            max_time = max(self.results['response_times'])
            min_time = min(self.results['response_times'])
            
            print(f"\n【响应时间】")
            print(f"  平均: {avg_time:.3f}秒")
            print(f"  最大: {max_time:.3f}秒")
            print(f"  最小: {min_time:.3f}秒")
        
        # Top IP
        if self.results['ip_stats']:
            print(f"\n【Top 5 访问IP】")
            top_ips = sorted(self.results['ip_stats'].items(), 
                           key=lambda x: x[1], reverse=True)[:5]
            for ip, count in top_ips:
                print(f"  {ip}: {count} 次")
        
        # Top URL
        if self.results['url_stats']:
            print(f"\n【Top 5 访问URL】")
            top_urls = sorted(self.results['url_stats'].items(), 
                            key=lambda x: x[1], reverse=True)[:5]
            for url, count in top_urls:
                print(f"  {url}: {count} 次")
        
        # 最近错误
        if self.results['error_messages']:
            print(f"\n【最近错误】")
            for err in list(self.results['error_messages'])[-3:]:
                print(f"  [{err['timestamp']}] {err['url']}: {err['message']}")
        
        print("="*60)
    
    def add_log(self, log_line):
        """添加日志到队列"""
        try:
            self.log_queue.put(log_line, block=False)
        except queue.Full:
            print("日志队列已满，丢弃日志")
    
    def start(self):
        """启动分析系统"""
        print("启动日志分析系统...")
        self.running = True
        
        # 启动工作线程
        self.workers = []
        for i in range(self.num_workers):
            t = threading.Thread(target=self.worker, name=f"Analyzer-{i}")
            t.daemon = True
            t.start()
            self.workers.append(t)
        
        # 启动报告线程
        self.report_thread = threading.Thread(target=self.reporter, name="Reporter")
        self.report_thread.daemon = True
        self.report_thread.start()
        
        print(f"已启动 {self.num_workers} 个分析线程和1个报告线程")
    
    def stop(self):
        """停止分析系统"""
        print("\n正在停止日志分析系统...")

        # 等待队列清空
        self.log_queue.join()
        
        # 队列处理完后再停止worker，避免join时无人消费
        self.running = False
        
        # 等待线程结束
        for t in self.workers:
            t.join(timeout=2)
        
        self.report_thread.join(timeout=2)
        
        # 打印最终报告
        print("\n最终报告:")
        with self.results_lock:
            self.print_report()

def generate_sample_logs(analyzer, duration=30):
    """生成示例日志（模拟实时日志流）"""
    
    ips = ['192.168.1.' + str(i) for i in range(100, 110)]
    urls = ['/api/users', '/api/data', '/api/login', '/api/logout', '/api/products']
    levels = ['INFO', 'WARNING', 'ERROR']
    methods = ['GET', 'POST', 'PUT', 'DELETE']
    error_messages = [
        'Database connection failed',
        'Timeout waiting for response',
        'Invalid authentication token',
        'Resource not found',
        'Internal server error'
    ]
    
    start_time = time.time()
    log_count = 0
    
    print(f"开始生成日志流（持续 {duration} 秒）...\n")
    
    while time.time() - start_time < duration:
        # 随机生成日志
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        level = random.choices(levels, weights=[70, 20, 10])[0]  # 70% INFO, 20% WARNING, 10% ERROR
        ip = random.choice(ips)
        method = random.choice(methods)
        url = random.choice(urls)
        status = random.choice([200, 201, 400, 404, 500]) if level == 'ERROR' else 200
        response_time = random.uniform(0.01, 2.0)
        
        message = ''
        if level == 'ERROR':
            message = ' ' + random.choice(error_messages)
        
        log_line = f"{timestamp} {level} {ip} {method} {url} {status} {response_time:.3f}s{message}"
        
        # 添加到分析器
        analyzer.add_log(log_line)
        
        log_count += 1
        
        # 随机延迟（模拟日志到达间隔）
        time.sleep(random.uniform(0.01, 0.1))
    
    print(f"\n日志流结束，共生成 {log_count} 条日志")

def main():
    """主函数"""
    # 创建日志分析器
    analyzer = LogAnalyzer(num_workers=3)
    
    # 启动分析系统
    analyzer.start()
    
    try:
        # 生成示例日志
        generate_sample_logs(analyzer, duration=30)
        
        # 等待一段时间让最后的日志处理完
        time.sleep(2)
        
    except KeyboardInterrupt:
        print("\n收到中断信号...")
    finally:
        # 停止分析系统
        analyzer.stop()

if __name__ == '__main__':
    main()
```

### 9.5 并发数据库操作系统



```python
import threading
import queue
import time
import sqlite3
from contextlib import contextmanager
from typing import List, Dict, Any

class ConnectionPool:
    """数据库连接池"""
    
    def __init__(self, db_path, pool_size=5):
        self.db_path = db_path
        self.pool_size = pool_size
        self.pool = queue.Queue(maxsize=pool_size)
        self.lock = threading.Lock()
        
        # 初始化连接池
        self._initialize_pool()
    
    def _initialize_pool(self):
        """初始化连接池"""
        for _ in range(self.pool_size):
            conn = sqlite3.connect(self.db_path, check_same_thread=False)
            conn.row_factory = sqlite3.Row  # 返回字典格式
            self.pool.put(conn)
    
    @contextmanager
    def get_connection(self):
        """获取数据库连接（上下文管理器）"""
        conn = self.pool.get()
        try:
            yield conn
        finally:
            self.pool.put(conn)
    
    def close_all(self):
        """关闭所有连接"""
        while not self.pool.empty():
            conn = self.pool.get()
            conn.close()

class DatabaseWorker:
    """数据库操作工作类"""
    
    def __init__(self, connection_pool):
        self.pool = connection_pool
        self.stats = {
            'queries': 0,
            'inserts': 0,
            'updates': 0,
            'errors': 0
        }
        self.stats_lock = threading.Lock()
    
    def execute_query(self, sql, params=None):
        """执行查询"""
        try:
            with self.pool.get_connection() as conn:
                cursor = conn.cursor()
                
                if params:
                    cursor.execute(sql, params)
                else:
                    cursor.execute(sql)
                
                results = cursor.fetchall()
                
                with self.stats_lock:
                    self.stats['queries'] += 1
                
                return [dict(row) for row in results]
                
        except Exception as e:
            with self.stats_lock:
                self.stats['errors'] += 1
            print(f"查询错误: {e}")
            return None
    
    def execute_insert(self, sql, params):
        """执行插入"""
        try:
            with self.pool.get_connection() as conn:
                cursor = conn.cursor()
                cursor.execute(sql, params)
                conn.commit()
                
                with self.stats_lock:
                    self.stats['inserts'] += 1
                
                return cursor.lastrowid
                
        except Exception as e:
            with self.stats_lock:
                self.stats['errors'] += 1
            print(f"插入错误: {e}")
            return None
    
    def execute_batch_insert(self, sql, data_list):
        """批量插入"""
        try:
            with self.pool.get_connection() as conn:
                cursor = conn.cursor()
                cursor.executemany(sql, data_list)
                conn.commit()
                
                with self.stats_lock:
                    self.stats['inserts'] += len(data_list)
                
                return cursor.rowcount
                
        except Exception as e:
            with self.stats_lock:
                self.stats['errors'] += 1
            print(f"批量插入错误: {e}")
            return None
    
    def execute_update(self, sql, params):
        """执行更新"""
        try:
            with self.pool.get_connection() as conn:
                cursor = conn.cursor()
                cursor.execute(sql, params)
                conn.commit()
                
                with self.stats_lock:
                    self.stats['updates'] += 1
                
                return cursor.rowcount
                
        except Exception as e:
            with self.stats_lock:
                self.stats['errors'] += 1
            print(f"更新错误: {e}")
            return None

class ConcurrentDBSystem:
    """并发数据库操作系统"""
    
    def __init__(self, db_path, num_workers=5):
        self.db_path = db_path
        self.num_workers = num_workers
        
        # 创建连接池
        self.pool = ConnectionPool(db_path, pool_size=num_workers)
        
        # 创建数据库工作者
        self.worker = DatabaseWorker(self.pool)
        
        # 任务队列
        self.task_queue = queue.Queue()
        
        # 结果队列
        self.result_queue = queue.Queue()
        
        # 初始化数据库
        self._initialize_database()
    
    def _initialize_database(self):
        """初始化数据库表"""
        with self.pool.get_connection() as conn:
            cursor = conn.cursor()
            
            # 创建用户表
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS users (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    username TEXT NOT NULL,
                    email TEXT NOT NULL,
                    age INTEGER,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            ''')
            
            # 创建订单表
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS orders (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    user_id INTEGER,
                    product TEXT NOT NULL,
                    amount REAL NOT NULL,
                    status TEXT DEFAULT 'pending',
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                    FOREIGN KEY (user_id) REFERENCES users (id)
                )
            ''')
            
            conn.commit()
            
        print("数据库初始化完成")
    
    def worker_thread(self):
        """工作线程"""
        while True:
            try:
                task = self.task_queue.get(timeout=5)
                
                if task is None:  # 停止信号
                    break
                
                task_type, args = task
                
                # 执行任务
                if task_type == 'query':
                    result = self.worker.execute_query(*args)
                elif task_type == 'insert':
                    result = self.worker.execute_insert(*args)
                elif task_type == 'batch_insert':
                    result = self.worker.execute_batch_insert(*args)
                elif task_type == 'update':
                    result = self.worker.execute_update(*args)
                else:
                    result = None
                
                self.result_queue.put(result)
                self.task_queue.task_done()
                
            except queue.Empty:
                break
            except Exception as e:
                print(f"工作线程错误: {e}")
    
    def batch_insert_users(self, users_data: List[Dict]):
        """批量插入用户"""
        print(f"\n批量插入 {len(users_data)} 个用户...")
        
        sql = "INSERT INTO users (username, email, age) VALUES (?, ?, ?)"
        data_list = [(u['username'], u['email'], u['age']) for u in users_data]
        
        start_time = time.time()
        
        # 分批插入
        batch_size = 100
        for i in range(0, len(data_list), batch_size):
            batch = data_list[i:i + batch_size]
            self.task_queue.put(('batch_insert', (sql, batch)))
        
        # 等待完成
        self.task_queue.join()
        
        end_time = time.time()
        
        print(f"批量插入完成，耗时: {end_time - start_time:.2f}秒")
    
    def concurrent_queries(self, num_queries=100):
        """并发查询"""
        print(f"\n执行 {num_queries} 个并发查询...")
        
        sql = "SELECT * FROM users WHERE age > ? LIMIT 10"
        
        start_time = time.time()
        
        # 提交查询任务
        for i in range(num_queries):
            age = 20 + (i % 30)
            self.task_queue.put(('query', (sql, (age,))))
        
        # 等待完成
        self.task_queue.join()
        
        end_time = time.time()
        
        print(f"并发查询完成，耗时: {end_time - start_time:.2f}秒")
        print(f"平均每个查询: {(end_time - start_time) / num_queries * 1000:.2f}毫秒")
    
    def concurrent_updates(self, num_updates=100):
        """并发更新"""
        print(f"\n执行 {num_updates} 个并发更新...")
        
        sql = "UPDATE users SET age = age + 1 WHERE id = ?"
        
        start_time = time.time()
        
        # 提交更新任务
        for i in range(1, num_updates + 1):
            self.task_queue.put(('update', (sql, (i,))))
        
        # 等待完成
        self.task_queue.join()
        
        end_time = time.time()
        
        print(f"并发更新完成，耗时: {end_time - start_time:.2f}秒")
    
    def run_workers(self):
        """启动所有工作线程"""
        workers = []
        for i in range(self.num_workers):
            t = threading.Thread(target=self.worker_thread, name=f"DBWorker-{i}")
            t.start()
            workers.append(t)
        
        return workers
    
    def stop_workers(self, workers):
        """停止所有工作线程"""
        for _ in range(self.num_workers):
            self.task_queue.put(None)
        
        for t in workers:
            t.join()
    
    def print_stats(self):
        """打印统计信息"""
        print("\n=== 数据库操作统计 ===")
        print(f"查询次数: {self.worker.stats['queries']}")
        print(f"插入次数: {self.worker.stats['inserts']}")
        print(f"更新次数: {self.worker.stats['updates']}")
        print(f"错误次数: {self.worker.stats['errors']}")
    
    def cleanup(self):
        """清理资源"""
        self.pool.close_all()
        print("\n数据库连接池已关闭")

def generate_sample_users(count):
    """生成示例用户数据"""
    import random
    
    first_names = ['张', '李', '王', '刘', '陈', '杨', '赵', '黄', '周', '吴']
    last_names = ['伟', '芳', '娜', '秀英', '敏', '静', '丽', '强', '磊', '洋']
    
    users = []
    for i in range(count):
        username = random.choice(first_names) + random.choice(last_names) + str(i)
        email = f"user{i}@example.com"
        age = random.randint(18, 60)
        
        users.append({
            'username': username,
            'email': email,
            'age': age
        })
    
    return users

def main():
    """主函数"""
    # 创建并发数据库系统
    db_system = ConcurrentDBSystem(db_path='concurrent_test.db', num_workers=5)
    
    # 启动工作线程
    workers = db_system.run_workers()
    
    try:
        # 1. 批量插入用户
        users = generate_sample_users(1000)
        db_system.batch_insert_users(users)
        
        # 2. 并发查询
        db_system.concurrent_queries(num_queries=200)
        
        # 3. 并发更新
        db_system.concurrent_updates(num_updates=500)
        
        # 4. 打印统计
        db_system.print_stats()
        
    finally:
        # 停止工作线程
        db_system.stop_workers(workers)
        
        # 清理资源
        db_system.cleanup()

if __name__ == '__main__':
    main()
```

```python
if __name__ == '__main__':
    main()
```

### 9.6 分布式任务调度器

```python
import threading
import queue
import time
import heapq
from datetime import datetime, timedelta
from typing import Callable, Any, Optional
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(threadName)s - %(message)s'
)

class Task:
    """任务对象"""
  
    def __init__(self, task_id, func, args=None, kwargs=None, 
                 priority=5, delay=0, interval=None):
        self.task_id = task_id
        self.func = func
        self.args = args or ()
        self.kwargs = kwargs or {}
        self.priority = priority
        self.interval = interval  # 如果设置，则为周期任务
      
        # 计算执行时间
        self.scheduled_time = time.time() + delay
      
        # 任务状态
        self.status = 'pending'  # pending, running, completed, failed
        self.result = None
        self.error = None
        self.retry_count = 0
        self.max_retries = 3
  
    def __lt__(self, other):
        """用于优先队列排序"""
        if self.scheduled_time == other.scheduled_time:
            return self.priority < other.priority
        return self.scheduled_time < other.scheduled_time
  
    def execute(self):
        """执行任务"""
        try:
            self.status = 'running'
            logging.info(f"执行任务: {self.task_id}")
          
            self.result = self.func(*self.args, **self.kwargs)
            self.status = 'completed'
          
            logging.info(f"任务完成: {self.task_id}")
            return True
          
        except Exception as e:
            self.error = str(e)
            self.status = 'failed'
            logging.error(f"任务失败: {self.task_id} - {e}")
            return False
  
    def should_retry(self):
        """是否应该重试"""
        return self.status == 'failed' and self.retry_count < self.max_retries
  
    def reschedule(self):
        """重新调度（用于周期任务）"""
        if self.interval:
            self.scheduled_time = time.time() + self.interval
            self.status = 'pending'
            return True
        return False

class TaskScheduler:
    """分布式任务调度器"""
  
    def __init__(self, num_workers=5):
        self.num_workers = num_workers
      
        # 任务优先队列（最小堆）
        self.task_heap = []
        self.heap_lock = threading.Lock()
      
        # 任务映射（用于快速查找）
        self.tasks = {}
        self.tasks_lock = threading.Lock()
      
        # 工作队列
        self.work_queue = queue.PriorityQueue()
      
        # 完成的任务
        self.completed_tasks = []
        self.completed_lock = threading.Lock()
      
        # 统计信息
        self.stats = {
            'total_tasks': 0,
            'completed': 0,
            'failed': 0,
            'retried': 0
        }
        self.stats_lock = threading.Lock()
      
        # 运行标志
        self.running = False
  
    def add_task(self, task_id, func, args=None, kwargs=None, 
                 priority=5, delay=0, interval=None):
        """添加任务"""
        task = Task(task_id, func, args, kwargs, priority, delay, interval)
      
        with self.heap_lock:
            heapq.heappush(self.task_heap, task)
      
        with self.tasks_lock:
            self.tasks[task_id] = task
      
        with self.stats_lock:
            self.stats['total_tasks'] += 1
      
        logging.info(f"添加任务: {task_id}, 延迟: {delay}秒, 优先级: {priority}")
  
    def schedule_at(self, task_id, func, scheduled_time, args=None, kwargs=None):
        """在指定时间执行任务"""
        delay = (scheduled_time - datetime.now()).total_seconds()
        if delay < 0:
            delay = 0
      
        self.add_task(task_id, func, args, kwargs, delay=delay)
  
    def schedule_periodic(self, task_id, func, interval, args=None, kwargs=None):
        """添加周期任务"""
        self.add_task(task_id, func, args, kwargs, interval=interval)
  
    def dispatcher_thread(self):
        """调度线程：将到期的任务分发给工作队列"""
        logging.info("调度线程启动")
      
        while self.running:
            try:
                current_time = time.time()
                tasks_to_dispatch = []
              
                # 检查堆顶的任务
                with self.heap_lock:
                    while self.task_heap and self.task_heap[0].scheduled_time <= current_time:
                        task = heapq.heappop(self.task_heap)
                        tasks_to_dispatch.append(task)
              
                # 将任务加入工作队列
                for task in tasks_to_dispatch:
                    self.work_queue.put((task.priority, task))
                    logging.info(f"分发任务: {task.task_id}")
              
                # 短暂休眠
                time.sleep(0.1)
              
            except Exception as e:
                logging.error(f"调度线程错误: {e}")
  
    def worker_thread(self):
        """工作线程：执行任务"""
        while self.running:
            try:
                # 从工作队列获取任务
                priority, task = self.work_queue.get(timeout=1)
              
                # 执行任务
                success = task.execute()
              
                if success:
                    # 任务成功
                    with self.stats_lock:
                        self.stats['completed'] += 1
                  
                    with self.completed_lock:
                        self.completed_tasks.append(task)
                  
                    # 如果是周期任务，重新调度
                    if task.reschedule():
                        with self.heap_lock:
                            heapq.heappush(self.task_heap, task)
                        logging.info(f"周期任务重新调度: {task.task_id}")
              
                else:
                    # 任务失败
                    if task.should_retry():
                        # 重试
                        task.retry_count += 1
                        task.scheduled_time = time.time() + (2 ** task.retry_count)  # 指数退避
                      
                        with self.heap_lock:
                            heapq.heappush(self.task_heap, task)
                      
                        with self.stats_lock:
                            self.stats['retried'] += 1
                      
                        logging.info(f"任务将重试 ({task.retry_count}/{task.max_retries}): {task.task_id}")
                  
                    else:
                        # 最终失败
                        with self.stats_lock:
                            self.stats['failed'] += 1
                      
                        with self.completed_lock:
                            self.completed_tasks.append(task)
              
                self.work_queue.task_done()
              
            except queue.Empty:
                continue
            except Exception as e:
                logging.error(f"工作线程错误: {e}")
  
    def start(self):
        """启动调度器"""
        logging.info(f"启动任务调度器，工作线程数: {self.num_workers}")
        self.running = True
      
        # 启动调度线程
        self.dispatcher = threading.Thread(target=self.dispatcher_thread, name="Dispatcher")
        self.dispatcher.start()
      
        # 启动工作线程
        self.workers = []
        for i in range(self.num_workers):
            t = threading.Thread(target=self.worker_thread, name=f"Worker-{i}")
            t.start()
            self.workers.append(t)
  
    def stop(self):
        """停止调度器"""
        logging.info("正在停止任务调度器...")
        self.running = False
      
        # 等待调度线程
        self.dispatcher.join(timeout=5)
      
        # 等待工作线程
        for t in self.workers:
            t.join(timeout=5)
      
        logging.info("任务调度器已停止")
  
    def get_task_status(self, task_id):
        """获取任务状态"""
        with self.tasks_lock:
            task = self.tasks.get(task_id)
            if task:
                return {
                    'task_id': task.task_id,
                    'status': task.status,
                    'result': task.result,
                    'error': task.error,
                    'retry_count': task.retry_count
                }
        return None
  
    def print_stats(self):
        """打印统计信息"""
        print("\n" + "="*60)
        print("任务调度统计")
        print("="*60)
      
        with self.stats_lock:
            print(f"总任务数: {self.stats['total_tasks']}")
            print(f"已完成: {self.stats['completed']}")
            print(f"失败: {self.stats['failed']}")
            print(f"重试次数: {self.stats['retried']}")
      
        with self.heap_lock:
            pending = len(self.task_heap)
      
        print(f"待执行: {pending}")
        print(f"工作队列: {self.work_queue.qsize()}")
        print("="*60)

# ===== 示例任务函数 =====

def sample_task(task_name, duration=1):
    """示例任务"""
    logging.info(f"开始执行: {task_name}")
    time.sleep(duration)
    logging.info(f"完成: {task_name}")
    return f"{task_name} 的结果"

def failing_task(fail_times=2):
    """会失败的任务（用于测试重试）"""
    import random
    if random.random() < 0.7:  # 70%概率失败
        raise Exception("任务执行失败")
    return "成功"

def data_processing_task(data_id):
    """数据处理任务"""
    time.sleep(0.5)
    return f"处理完成: 数据{data_id}"

def periodic_cleanup_task():
    """周期性清理任务"""
    logging.info("执行清理任务...")
    time.sleep(0.5)
    return "清理完成"

def send_report_task(report_type):
    """发送报告任务"""
    logging.info(f"发送{report_type}报告...")
    time.sleep(1)
    return f"{report_type}报告已发送"

def main():
    """主函数"""
    # 创建调度器
    scheduler = TaskScheduler(num_workers=5)
  
    # 启动调度器
    scheduler.start()
  
    try:
        # 1. 添加立即执行的任务
        print("\n添加立即执行的任务...")
        for i in range(5):
            scheduler.add_task(
                task_id=f"immediate_{i}",
                func=sample_task,
                args=(f"任务{i}",),
                priority=5
            )
      
        time.sleep(3)
      
        # 2. 添加延迟执行的任务
        print("\n添加延迟执行的任务...")
        scheduler.add_task(
            task_id="delayed_1",
            func=sample_task,
            args=("延迟任务1",),
            delay=2,
            priority=1  # 高优先级
        )
      
        scheduler.add_task(
            task_id="delayed_2",
            func=sample_task,
            args=("延迟任务2",),
            delay=2,
            priority=10  # 低优先级
        )
      
        time.sleep(4)
      
        # 3. 添加周期任务
        print("\n添加周期任务...")
        scheduler.schedule_periodic(
            task_id="periodic_cleanup",
            func=periodic_cleanup_task,
            interval=3  # 每3秒执行一次
        )
      
        time.sleep(10)
      
        # 4. 添加会失败的任务（测试重试）
        print("\n添加会失败的任务（测试重试）...")
        for i in range(3):
            scheduler.add_task(
                task_id=f"failing_{i}",
                func=failing_task,
                priority=5
            )
      
        time.sleep(5)
      
        # 5. 添加批量数据处理任务
        print("\n添加批量数据处理任务...")
        for i in range(20):
            scheduler.add_task(
                task_id=f"data_{i}",
                func=data_processing_task,
                args=(i,),
                priority=i % 10  # 不同优先级
            )
      
        time.sleep(8)
      
        # 6. 定时任务
        print("\n添加定时任务...")
        scheduled_time = datetime.now() + timedelta(seconds=3)
        scheduler.schedule_at(
            task_id="scheduled_report",
            func=send_report_task,
            scheduled_time=scheduled_time,
            args=("每日",)
        )
      
        time.sleep(5)
      
        # 打印统计信息
        scheduler.print_stats()
      
        # 查询特定任务状态
        print("\n查询任务状态:")
        for task_id in ['immediate_0', 'delayed_1', 'failing_1', 'periodic_cleanup']:
            status = scheduler.get_task_status(task_id)
            if status:
                print(f"  {task_id}: {status['status']}")
      
        # 等待所有任务完成
        print("\n等待剩余任务完成...")
        time.sleep(5)
      
    except KeyboardInterrupt:
        print("\n收到中断信号...")
  
    finally:
        # 停止调度器
        scheduler.stop()
      
        # 最终统计
        scheduler.print_stats()

if __name__ == '__main__':
    main()
```

### 9.7 高性能Web服务器（基础版）

```python
import socket
import threading
import queue
import time
import os
from pathlib import Path
from urllib.parse import unquote
import mimetypes

class ThreadPool:
    """线程池"""
  
    def __init__(self, num_workers):
        self.num_workers = num_workers
        self.task_queue = queue.Queue()
        self.workers = []
        self.running = False
  
    def start(self):
        """启动线程池"""
        self.running = True
        for i in range(self.num_workers):
            t = threading.Thread(target=self._worker, name=f"Worker-{i}")
            t.daemon = True
            t.start()
            self.workers.append(t)
  
    def _worker(self):
        """工作线程"""
        while self.running:
            try:
                task, args = self.task_queue.get(timeout=1)
                task(*args)
                self.task_queue.task_done()
            except queue.Empty:
                continue
            except Exception as e:
                print(f"Worker错误: {e}")
  
    def submit(self, task, *args):
        """提交任务"""
        self.task_queue.put((task, args))
  
    def shutdown(self):
        """关闭线程池"""
        self.running = False
        for t in self.workers:
            t.join(timeout=2)

class HTTPServer:
    """高性能HTTP服务器"""
  
    def __init__(self, host='0.0.0.0', port=8000, num_workers=10, document_root='www'):
        self.host = host
        self.port = port
        self.num_workers = num_workers
        self.document_root = Path(document_root)
      
        # 创建文档根目录
        self.document_root.mkdir(exist_ok=True)
      
        # 线程池
        self.thread_pool = ThreadPool(num_workers)
      
        # 统计信息
        self.stats = {
            'requests': 0,
            'bytes_sent': 0,
            'errors': 0
        }
        self.stats_lock = threading.Lock()
      
        # 服务器socket
        self.server_socket = None
        self.running = False
  
    def parse_request(self, request_data):
        """解析HTTP请求"""
        try:
            lines = request_data.decode('utf-8').split('\r\n')
            request_line = lines[0]
          
            method, path, protocol = request_line.split()
          
            # 解析headers
            headers = {}
            for line in lines[1:]:
                if ':' in line:
                    key, value = line.split(':', 1)
                    headers[key.strip()] = value.strip()
          
            return {
                'method': method,
                'path': unquote(path),
                'protocol': protocol,
                'headers': headers
            }
        except Exception as e:
            print(f"解析请求失败: {e}")
            return None
  
    def get_content_type(self, file_path):
        """获取文件的Content-Type"""
        mime_type, _ = mimetypes.guess_type(file_path)
        return mime_type or 'application/octet-stream'
  
    def build_response(self, status_code, status_text, headers=None, body=b''):
        """构建HTTP响应"""
        response = f"HTTP/1.1 {status_code} {status_text}\r\n"
      
        # 默认headers
        default_headers = {
            'Server': 'PythonHTTPServer/1.0',
            'Content-Length': str(len(body)),
            'Connection': 'close'
        }
      
        if headers:
            default_headers.update(headers)
      
        for key, value in default_headers.items():
            response += f"{key}: {value}\r\n"
      
        response += "\r\n"
      
        return response.encode('utf-8') + body
  
    def serve_file(self, file_path):
        """提供文件服务"""
        try:
            with open(file_path, 'rb') as f:
                content = f.read()
          
            headers = {
                'Content-Type': self.get_content_type(str(file_path))
            }
          
            return self.build_response(200, 'OK', headers, content)
      
        except FileNotFoundError:
            body = b'<h1>404 Not Found</h1>'
            headers = {'Content-Type': 'text/html'}
            return self.build_response(404, 'Not Found', headers, body)
      
        except Exception as e:
            body = f'<h1>500 Internal Server Error</h1><p>{e}</p>'.encode('utf-8')
            headers = {'Content-Type': 'text/html'}
            return self.build_response(500, 'Internal Server Error', headers, body)
  
    def handle_client(self, client_socket, client_address):
        """处理客户端请求"""
        try:
            # 接收请求
            request_data = client_socket.recv(4096)
          
            if not request_data:
                return
          
            # 解析请求
            request = self.parse_request(request_data)
          
            if not request:
                response = self.build_response(
                    400, 'Bad Request',
                    {'Content-Type': 'text/html'},
                    b'<h1>400 Bad Request</h1>'
                )
                client_socket.sendall(response)
                return
          
            print(f"[{client_address[0]}] {request['method']} {request['path']}")
          
            # 更新统计
            with self.stats_lock:
                self.stats['requests'] += 1
          
            # 处理请求
            if request['method'] == 'GET':
                # 构建文件路径
                path = request['path'].lstrip('/')
              
                if not path or path.endswith('/'):
                    path += 'index.html'
              
                file_path = self.document_root / path
              
                # 安全检查：防止目录遍历攻击
                try:
                    file_path = file_path.resolve()
                    self.document_root.resolve()
                  
                    if not str(file_path).startswith(str(self.document_root.resolve())):
                        raise ValueError("非法路径")
              
                except:
                    response = self.build_response(
                        403, 'Forbidden',
                        {'Content-Type': 'text/html'},
                        b'<h1>403 Forbidden</h1>'
                    )
                    client_socket.sendall(response)
                    return
              
                # 提供文件
                response = self.serve_file(file_path)
                client_socket.sendall(response)
              
                # 更新统计
                with self.stats_lock:
                    self.stats['bytes_sent'] += len(response)
          
            else:
                # 不支持的方法
                response = self.build_response(
                    405, 'Method Not Allowed',
                    {'Content-Type': 'text/html'},
                    b'<h1>405 Method Not Allowed</h1>'
                )
                client_socket.sendall(response)
      
        except Exception as e:
            print(f"处理请求错误: {e}")
          
            with self.stats_lock:
                self.stats['errors'] += 1
          
            try:
                response = self.build_response(
                    500, 'Internal Server Error',
                    {'Content-Type': 'text/html'},
                    f'<h1>500 Internal Server Error</h1><p>{e}</p>'.encode('utf-8')
                )
                client_socket.sendall(response)
            except:
                pass
      
        finally:
            client_socket.close()
  
    def stats_reporter(self):
        """统计报告线程"""
        while self.running:
            time.sleep(10)
          
            with self.stats_lock:
                print(f"\n=== 服务器统计 ===")
                print(f"总请求数: {self.stats['requests']}")
                print(f"发送字节: {self.stats['bytes_sent'] / 1024:.2f} KB")
                print(f"错误数: {self.stats['errors']}")
                print("="*30 + "\n")
  
    def start(self):
        """启动服务器"""
        # 创建socket
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server_socket.bind((self.host, self.port))
        self.server_socket.listen(128)
      
        print(f"HTTP服务器启动在 http://{self.host}:{self.port}")
        print(f"文档根目录: {self.document_root.absolute()}")
        print(f"工作线程数: {self.num_workers}\n")
      
        # 启动线程池
        self.thread_pool.start()
      
        # 启动统计线程
        self.running = True
        stats_thread = threading.Thread(target=self.stats_reporter, name="StatsReporter")
        stats_thread.daemon = True
        stats_thread.start()
      
        try:
            while True:
                # 接受连接
                client_socket, client_address = self.server_socket.accept()
              
                # 提交到线程池处理
                self.thread_pool.submit(self.handle_client, client_socket, client_address)
      
        except KeyboardInterrupt:
            print("\n正在关闭服务器...")
      
        finally:
            self.stop()
  
    def stop(self):
        """停止服务器"""
        self.running = False
      
        if self.server_socket:
            self.server_socket.close()
      
        self.thread_pool.shutdown()
      
        print("服务器已关闭")

def create_sample_website(document_root):
    """创建示例网站"""
    root = Path(document_root)
    root.mkdir(exist_ok=True)
  
    # 创建index.html
    index_html = """<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Python HTTP Server</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>欢迎使用Python HTTP服务器</h1>
        <p>这是一个多线程HTTP服务器示例</p>
        <ul>
            <li><a href="about.html">关于</a></li>
            <li><a href="test.txt">测试文件</a></li>
        </ul>
    </div>
    <script src="script.js"></script>
</body>
</html>
"""
  
    # 创建style.css
    style_css = """body {
    font-family: Arial, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    margin: 0;
    padding: 20px;
}

.container {
    max-width: 800px;
    margin: 50px auto;
    background: white;
    padding: 30px;
    border-radius: 10px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.3);
}

h1 {
    color: #667eea;
}

a {
    color: #764ba2;
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}
"""
  
    # 创建about.html
    about_html = """<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>关于 - Python HTTP Server</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>关于本服务器</h1>
        <p>这是一个使用Python实现的多线程HTTP服务器</p>
        <p><a href="index.html">返回首页</a></p>
    </div>
</body>
</html>
"""
  
    # 创建script.js
    script_js = """console.log('Python HTTP Server loaded');

document.addEventListener('DOMContentLoaded', function() {
    console.log('Page ready');
});
"""
  
    # 创建test.txt
    test_txt = """这是一个测试文本文件。
Test file for Python HTTP Server.
"""
  
    # 写入文件
    (root / 'index.html').write_text(index_html, encoding='utf-8')
    (root / 'style.css').write_text(style_css, encoding='utf-8')
    (root / 'about.html').write_text(about_html, encoding='utf-8')
    (root / 'script.js').write_text(script_js, encoding='utf-8')
    (root / 'test.txt').write_text(test_txt, encoding='utf-8')
  
    print(f"示例网站已创建在 {root.absolute()}")

def main():
    """主函数"""
    # 创建示例网站
    create_sample_website('www')
  
    # 创建并启动服务器
    server = HTTPServer(host='0.0.0.0', port=8000, num_workers=10)
    server.start()

if __name__ == '__main__':
    main()
```

---

## 第十章：性能分析与调优

### 10.1 性能分析工具

#### 使用cProfile进行性能分析

```python
import cProfile
import pstats
import io
from pstats import SortKey

def performance_test_function():
    """需要分析的函数"""
    result = []
    for i in range(10000):
        result.append(i ** 2)
    return result

def profile_function():
    """使用cProfile分析函数"""
    profiler = cProfile.Profile()
    profiler.enable()
  
    # 执行需要分析的代码
    performance_test_function()
  
    profiler.disable()
  
    # 输出结果
    s = io.StringIO()
    ps = pstats.Stats(profiler, stream=s).sort_stats(SortKey.CUMULATIVE)
    ps.print_stats(10)  # 打印前10个最耗时的函数
  
    print(s.getvalue())

if __name__ == '__main__':
    profile_function()
```

#### 使用line_profiler进行逐行分析

```python
# 安装: pip install line-profiler

# 使用装饰器标记要分析的函数
@profile  # 需要使用 kernprof 运行
def slow_function():
    total = 0
    for i in range(1000):
        for j in range(1000):
            total += i * j
    return total

# 运行方式:
# kernprof -l -v script.py
```

#### 使用memory_profiler分析内存

```python
# 安装: pip install memory-profiler

from memory_profiler import profile

@profile
def memory_intensive_function():
    """内存密集型函数"""
    a = [i for i in range(1000000)]
    b = [i * 2 for i in range(1000000)]
    c = a + b
    return c

if __name__ == '__main__':
    memory_intensive_function()

# 运行: python -m memory_profiler script.py
```

### 10.2 并发性能监控

```python
import threading
import time
import psutil
import os
from collections import deque

class PerformanceMonitor:
    """并发性能监控器"""
  
    def __init__(self, interval=1):
        self.interval = interval
        self.running = False
      
        # 存储历史数据
        self.cpu_history = deque(maxlen=60)
        self.memory_history = deque(maxlen=60)
        self.thread_history = deque(maxlen=60)
      
        # 进程对象
        self.process = psutil.Process(os.getpid())
  
    def monitor_thread(self):
        """监控线程"""
        while self.running:
            try:
                # CPU使用率
                cpu_percent = self.process.cpu_percent(interval=0.1)
              
                # 内存使用
                memory_info = self.process.memory_info()
                memory_mb = memory_info.rss / 1024 / 1024
              
                # 线程数
                thread_count = self.process.num_threads()
              
                # 记录历史
                self.cpu_history.append(cpu_percent)
                self.memory_history.append(memory_mb)
                self.thread_history.append(thread_count)
              
                time.sleep(self.interval)
              
            except Exception as e:
                print(f"监控错误: {e}")
  
    def start(self):
        """启动监控"""
        self.running = True
        self.monitor = threading.Thread(target=self.monitor_thread, daemon=True)
        self.monitor.start()
  
    def stop(self):
        """停止监控"""
        self.running = False
        if self.monitor:
            self.monitor.join(timeout=2)
  
    def get_stats(self):
        """获取统计信息"""
        if not self.cpu_history:
            return None
      
        return {
            'cpu': {
                'current': self.cpu_history[-1],
                'avg': sum(self.cpu_history) / len(self.cpu_history),
                'max': max(self.cpu_history)
            },
            'memory': {
                'current': self.memory_history[-1],
                'avg': sum(self.memory_history) / len(self.memory_history),
                'max': max(self.memory_history)
            },
            'threads': {
                'current': self.thread_history[-1],
                'max': max(self.thread_history)
            }
        }
  
    def print_report(self):
        """打印报告"""
        stats = self.get_stats()
        if not stats:
            print("暂无数据")
            return
      
        print("\n=== 性能监控报告 ===")
        print(f"CPU使用率: {stats['cpu']['current']:.1f}% "
              f"(平均: {stats['cpu']['avg']:.1f}%, 峰值: {stats['cpu']['max']:.1f}%)")
        print(f"内存使用: {stats['memory']['current']:.1f}MB "
              f"(平均: {stats['memory']['avg']:.1f}MB, 峰值: {stats['memory']['max']:.1f}MB)")
        print(f"线程数: {stats['threads']['current']} "
              f"(峰值: {stats['threads']['max']})")
        print("="*25)

# 测试示例
def cpu_intensive_task():
    """CPU密集任务"""
    total = 0
    for i in range(1000000):
        total += i ** 2
    return total

def main():
    monitor = PerformanceMonitor(interval=0.5)
    monitor.start()
  
    # 模拟并发任务
    threads = []
    for i in range(10):
        t = threading.Thread(target=cpu_intensive_task)
        t.start()
        threads.append(t)
  
    # 等待完成
    for t in threads:
        t.join()
  
    time.sleep(1)
    monitor.print_report()
    monitor.stop()

if __name__ == '__main__':
    main()
```

### 10.3 性能优化技巧

```python
import threading
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

# ===== 技巧1: 使用线程池替代手动管理线程 =====

# 不推荐
def bad_way():
    threads = []
    for i in range(100):
        t = threading.Thread(target=lambda x: time.sleep(0.1), args=(i,))
        t.start()
        threads.append(t)
  
    for t in threads:
        t.join()

# 推荐
def good_way():
    with ThreadPoolExecutor(max_workers=10) as executor:
        futures = [executor.submit(time.sleep, 0.1) for i in range(100)]
        for future in as_completed(futures):
            future.result()

# ===== 技巧2: 使用局部变量减少锁竞争 =====

class Counter:
    def __init__(self):
        self.count = 0
        self.lock = threading.Lock()
  
    # 不推荐：频繁加锁
    def increment_bad(self, times):
        for _ in range(times):
            with self.lock:
                self.count += 1
  
    # 推荐：批量更新
    def increment_good(self, times):
        local_count = 0
        for _ in range(times):
            local_count += 1
      
        with self.lock:
            self.count += local_count

# ===== 技巧3: 使用队列避免锁 =====

import queue

class TaskProcessor:
    def __init__(self):
        self.task_queue = queue.Queue()
        self.results = []
        self.lock = threading.Lock()  # 仅用于结果
  
    def worker(self):
        while True:
            try:
                task = self.task_queue.get(timeout=1)
                if task is None:
                    break
              
                # 处理任务
                result = task * 2
              
                # 存储结果
                with self.lock:
                    self.results.append(result)
              
                self.task_queue.task_done()
            except queue.Empty:
                break
  
    def process(self, tasks):
        # 添加任务
        for task in tasks:
            self.task_queue.put(task)
      
        # 启动工作线程
        workers = []
        for _ in range(4):
            t = threading.Thread(target=self.worker)
            t.start()
            workers.append(t)
      
        # 等待完成
        self.task_queue.join()
      
        # 发送停止信号
        for _ in range(4):
            self.task_queue.put(None)
      
        for t in workers:
            t.join()

# ===== 技巧4: 使用线程本地存储 =====

thread_local = threading.local()

def process_with_connection():
    # 每个线程都有自己的连接
    if not hasattr(thread_local, 'connection'):
        thread_local.connection = create_connection()
  
    return thread_local.connection.query()

def create_connection():
    return f"Connection-{threading.current_thread().name}"

# ===== 技巧5: 避免GIL影响 =====

# CPU密集型：使用multiprocessing
from multiprocessing import Pool

def cpu_task(n):
    return sum(i * i for i in range(n))

def use_multiprocessing():
    with Pool(4) as pool:
        results = pool.map(cpu_task, [1000000] * 10)

# I/O密集型：使用threading
def io_task():
    time.sleep(0.1)
    return "done"

def use_threading():
    with ThreadPoolExecutor(max_workers=10) as executor:
        results = list(executor.map(lambda _: io_task(), range(10)))

# ===== 性能对比测试 =====

def benchmark():
    print("=== 性能对比测试 ===\n")
  
    # 测试1: 手动线程 vs 线程池
    print("1. 手动管理线程 vs 线程池")
  
    start = time.time()
    bad_way()
    print(f"手动管理: {time.time() - start:.3f}秒")
  
    start = time.time()
    good_way()
    print(f"线程池: {time.time() - start:.3f}秒\n")
  
    # 测试2: 频繁加锁 vs 批量更新
    print("2. 频繁加锁 vs 批量更新")
  
    counter = Counter()
  
    start = time.time()
    threads = [threading.Thread(target=counter.increment_bad, args=(10000,)) 
               for _ in range(10)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    print(f"频繁加锁: {time.time() - start:.3f}秒")
  
    counter = Counter()
    start = time.time()
    threads = [threading.Thread(target=counter.increment_good, args=(10000,)) 
               for _ in range(10)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    print(f"批量更新: {time.time() - start:.3f}秒")

if __name__ == '__main__':
    benchmark()
```

### 10.4 常见性能瓶颈与解决方案

```python
"""
常见性能问题及解决方案总结
"""

# 问题1: 过多线程导致上下文切换开销
# 解决方案: 使用合适大小的线程池
MAX_WORKERS = min(32, os.cpu_count() * 2)  # 经验值

# 问题2: 锁竞争激烈
# 解决方案: 
# - 减少临界区大小
# - 使用线程安全队列（queue.Queue内部使用锁和条件变量）
# - 分段锁

# 问题3: GIL限制CPU密集型性能
# 解决方案:
# - 使用multiprocessing
# - 使用C扩展（Cython, numba）
# - 使用asyncio处理I/O

# 问题4: 死锁
# 解决方案:
# - 统一加锁顺序
# - 使用timeout
# - 需要同一线程重入同一把锁时使用RLock；多锁死锁仍要靠统一顺序等策略解决

# 问题5: 内存泄漏
# 解决方案:
# - 及时清理大对象
# - 使用weakref
# - 定期监控内存

print("""
性能优化最佳实践:

1. 先测量再优化
2. I/O密集用threading，CPU密集用multiprocessing
3. 使用线程池管理线程
4. 减少锁竞争，优先使用队列
5. 避免过度优化，保持代码可读性
""")
```

---

## 总结

本教程涵盖了Python并发编程的核心内容：

**第一部分：基础**
- 进程与线程的概念
- threading模块基础
- 线程同步机制（Lock, RLock, Semaphore等）

**第二部分：高级应用**
- 线程池与任务队列
- 生产者消费者模式
- 多进程编程
- 异步编程（asyncio）作为大量 I/O 并发时的延伸选择

**第三部分：实战项目**
- 多线程网络爬虫
- 图像批处理
- 下载管理器
- 日志分析系统
- HTTP服务器

**第四部分：优化**
- 性能监控工具
- 优化技巧
- 常见问题解决

通过学习这些内容，你应该能够：
✓ 理解并发编程的核心概念
✓ 选择合适的并发模型
✓ 编写高性能并发程序
✓ 解决实际项目中的并发问题
