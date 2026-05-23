# Python 网络编程详细教程

> 本教程为新手量身定制，由浅入深，包含完整可运行代码与详细注释。
> 学完本教程后，你将掌握 TCP/UDP 编程的核心技能，能独立开发简单的网络应用。

---

## 📖 前言：写在开始之前

亲爱的同学，欢迎来到 Python 网络编程的世界！在你开始这段旅程之前，请先了解一些必要的基础概念。

### 一、什么是网络编程？

想象一下你正在用微信和朋友聊天。当你发送"你好"两个字时，这两个字是如何穿越千山万水到达朋友手机的？答案就是：**网络**。

**网络编程**就是编写让两台（或多台）计算机能够互相通信的程序。

### 二、必须了解的核心概念

#### 1. IP 地址

IP 地址就像每台计算机在网络上的"门牌号"。常见 IP：

| IP 地址 | 含义 |
|---|---|
| `127.0.0.1` | 本机回环地址（也叫 `localhost`） |
| `0.0.0.0` | 表示绑定本机所有网卡 |
| `192.168.x.x` | 局域网常见 IP |
| `8.8.8.8` | Google 公共 DNS |

#### 2. 端口（Port）

如果说 IP 地址是门牌号，那端口就是这栋楼里的房间号。一台计算机能同时运行多个网络程序，每个程序占用不同的端口。

- **范围**：0 ~ 65535
- **0 ~ 1023**：系统保留端口（如 HTTP 的 80、HTTPS 的 443）
- **1024 ~ 49151**：用户注册端口
- **49152 ~ 65535**：动态/私有端口

> 💡 新手建议使用 **1024 以上**的端口号进行编程练习。

#### 3. TCP 与 UDP

这是两种最常见的传输协议：

| 特性 | TCP | UDP |
|---|---|---|
| 连接方式 | 面向连接（三次握手） | 无连接 |
| 可靠性 | 可靠（保证送达、有序） | 不可靠（可能丢包、乱序） |
| 速度 | 较慢 | 较快 |
| 类比 | 打电话（先拨号） | 扔纸条（直接扔出去） |
| 应用 | 文件传输、网页、邮件 | 视频流、游戏、DNS |

#### 4. 客户端/服务器模型（C/S）

- **服务器（Server）**：等待客户端连接，提供服务
- **客户端（Client）**：主动连接服务器，请求服务

> 🍔 类比：餐厅是服务器，等待顾客（客户端）光临；顾客点菜（请求），服务员上菜（响应）。

#### 5. Socket（套接字）

Socket 是网络编程的**核心概念**，可以理解为"通信的端点"。
- 一个 Socket = IP 地址 + 端口号 + 协议
- 两个 Socket 之间建立通道，就能传输数据

> ☎️ 形象地说：Socket 就像电话机，两台电话机通过电话线（网络）连接就能通话。

好，基础知识到这里就足够了，让我们开始动手吧！

---

# 🎯 课程 1：Socket 编程（TCP）

## 关卡 1：创建连接套接字（简单）

### 📋 学习目标

- 了解 Python 中的 `socket` 模块
- 学会创建 TCP 服务端和客户端的 socket 对象
- 理解服务端的工作流程：`bind` → `listen` → `accept`
- 理解客户端的工作流程：`connect`

### 1.1 Python 的 socket 模块

Python 自带 `socket` 模块，无需额外安装。使用前导入即可：

```python
import socket
```

### 1.2 创建一个 socket 对象

```python
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

让我们**逐字逐句**地解析这一行代码：

- `socket.socket()`：socket 模块下的 socket 类，调用它就能创建一个 socket 对象
- `socket.AF_INET`：地址族（Address Family），表示使用 **IPv4** 地址
  - IPv6 则使用 `socket.AF_INET6`
- `socket.SOCK_STREAM`：套接字类型，表示 **TCP** 协议
  - UDP 则使用 `socket.SOCK_DGRAM`

> 🧠 **记忆技巧**：
> - **AF** = Address Family（地址族）
> - **SOCK_STREAM** = 流式套接字（TCP，连续不断的数据流）
> - **SOCK_DGRAM** = 数据报套接字（UDP，一包一包的）

### 1.3 服务端的工作流程

服务端需要完成 **4 个核心步骤**：

```
1. 创建 socket    →   socket.socket()
2. 绑定地址       →   bind()
3. 开始监听       →   listen()
4. 接受连接       →   accept()
```

**完整代码 `server.py`：**

```python
import socket

# 第一步：创建 socket 对象
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 第二步：绑定 IP 地址和端口
# 参数是一个元组 (IP, 端口)
# '127.0.0.1' 表示本机
# 如果想让其他电脑也能连接，使用 '0.0.0.0'
server_socket.bind(('127.0.0.1', 8888))

# 第三步：开始监听，参数是等待连接队列的最大长度
server_socket.listen(5)

print("服务器已启动，等待客户端连接...")

# 第四步：接受客户端连接
# accept() 会阻塞程序（一直等），直到有客户端连接
# 返回值：(新的 socket 对象, 客户端地址元组)
client_socket, client_address = server_socket.accept()

print(f"客户端 {client_address} 已连接！")

# 关闭连接
client_socket.close()
server_socket.close()
```

**重要概念详解：**

#### 🔹 `bind()` 的参数

- 是一个元组 `(host, port)`
- `host` 是 IP 地址字符串
- `port` 是端口号整数

#### 🔹 `listen()` 的参数

- 表示**等待队列**的最大长度
- 比如 `listen(5)` 表示最多允许 5 个连接在排队等候被 `accept`

#### 🔹 `accept()` 的返回值

- 返回一个元组：`(conn, addr)`
- `conn`：一个**新的** socket 对象，专门用于与该客户端通信
- `addr`：客户端的地址 `(IP, 端口)`

#### 🔹 为什么 `accept()` 返回新的 socket？

- 原本的 `server_socket` 继续负责"接听"新连接
- 新的 `client_socket` 专门负责与已连接的客户端通信
- 类比：客服总机接到电话后，转接给专门的客服人员

### 1.4 客户端的工作流程

客户端的流程更简单，只要 **2 步**：

```
1. 创建 socket   →   socket.socket()
2. 连接服务器    →   connect()
```

**完整代码 `client.py`：**

```python
import socket

# 第一步：创建 socket 对象
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 第二步：连接服务器
# 参数也是元组 (IP, 端口)，要和服务器的一致
client_socket.connect(('127.0.0.1', 8888))

print("已连接到服务器！")

# 关闭连接
client_socket.close()
```

### 1.5 运行测试

⚠️ **重要：必须先启动服务器，再启动客户端！**

1. 打开一个终端，运行 `python server.py`
2. 看到"等待客户端连接..."后
3. 打开**另一个**终端，运行 `python client.py`
4. 服务器应该会显示 `客户端 ('127.0.0.1', 50231) 已连接！`

### 1.6 常见问题

**❓ 问题 1：报错 `Address already in use`**

**原因**：上一次程序的端口还没释放（操作系统通常会保留一段时间）。

**解决方法**：
- 换一个端口号，或者
- 在创建 socket 后添加端口重用设置：
```python
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
```

**❓ 问题 2：客户端连不上服务器**

**检查清单**：
- ✅ 服务器是否已启动？
- ✅ IP 和端口是否一致？
- ✅ 防火墙是否拦截？

### 📝 本关小结

恭喜你完成了第一关！你已经学会：
- ✓ 创建 TCP socket 对象
- ✓ 服务端的 `bind` / `listen` / `accept`
- ✓ 客户端的 `connect`

但这两个程序还没"说话"，下一关我们让它们真正对话起来。

---

## 关卡 2：客户端与服务端通信（简单）

### 📋 学习目标

- 学会使用 `send()` 和 `recv()` 发送、接收数据
- 理解字节流（bytes）与字符串的转换
- 实现一次完整的请求-响应通信

### 2.1 `send()` 和 `recv()` 详解

Socket 通信中，有两个核心方法：

| 方法 | 作用 | 参数 |
|---|---|---|
| `send(data)` | 发送数据 | data **必须是 bytes 类型** |
| `recv(bufsize)` | 接收数据 | bufsize 是一次接收的最大字节数 |

`recv` 返回的也是 **bytes 类型**。

### 2.2 bytes 与 str 的转换

**为什么要转换？**
网络上传输的本质是二进制数据（0 和 1）。字符串需要按照某种编码（如 UTF-8）转换为字节后才能传输。

```python
# 字符串 → bytes：使用 encode()
msg = "你好"
data = msg.encode('utf-8')
print(data)
# 输出：b'\xe4\xbd\xa0\xe5\xa5\xbd'

# bytes → 字符串：使用 decode()
text = data.decode('utf-8')
print(text)
# 输出：你好
```

> 💡 **快速记忆**：
> - `encode` = 编码（人话 → 机器语言）
> - `decode` = 解码（机器语言 → 人话）

### 2.3 完整的通信示例：回声程序

我们来实现一个简单的"回声"程序：客户端发送一句话，服务器原样返回。

**服务端 `server.py`：**

```python
import socket

# 创建 socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 设置端口重用（避免重启时报错）
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

# 绑定地址
server_socket.bind(('127.0.0.1', 8888))

# 开始监听
server_socket.listen(5)
print("服务器已启动，等待连接...")

# 接受连接
client_socket, client_address = server_socket.accept()
print(f"客户端 {client_address} 已连接！")

# 接收数据（最多 1024 字节）
data = client_socket.recv(1024)
received_msg = data.decode('utf-8')
print(f"收到客户端消息：{received_msg}")

# 发送响应
response = f"服务器已收到你的消息：{received_msg}"
client_socket.send(response.encode('utf-8'))

# 关闭连接
client_socket.close()
server_socket.close()
print("通信结束")
```

**客户端 `client.py`：**

```python
import socket

# 创建 socket 并连接
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect(('127.0.0.1', 8888))
print("已连接到服务器")

# 发送消息
message = "Hello, Server! 你好，服务器！"
client_socket.send(message.encode('utf-8'))
print(f"已发送：{message}")

# 接收响应
response = client_socket.recv(1024)
print(f"服务器响应：{response.decode('utf-8')}")

# 关闭连接
client_socket.close()
```

### 2.4 进阶：实现循环聊天

刚才的程序只能通信一次就结束，太简单了。我们让它能持续对话：

**服务端 `chat_server.py`：**

```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind(('127.0.0.1', 8888))
server_socket.listen(5)
print("服务器已启动...")

client_socket, addr = server_socket.accept()
print(f"客户端 {addr} 已连接")

while True:
    # 接收客户端消息
    data = client_socket.recv(1024)

    # 如果收到空数据，说明客户端断开了
    if not data:
        print("客户端断开连接")
        break

    msg = data.decode('utf-8')
    print(f"客户端：{msg}")

    # 如果是 bye，结束聊天
    if msg.lower() == 'bye':
        print("客户端请求结束聊天")
        break

    # 服务器输入回复
    reply = input("我（服务器）：")
    client_socket.send(reply.encode('utf-8'))

    if reply.lower() == 'bye':
        print("结束聊天")
        break

client_socket.close()
server_socket.close()
```

**客户端 `chat_client.py`：**

```python
import socket

client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect(('127.0.0.1', 8888))
print("已连接到服务器，开始聊天（输入 bye 结束）")

while True:
    # 输入消息
    msg = input("我（客户端）：")
    client_socket.send(msg.encode('utf-8'))

    if msg.lower() == 'bye':
        break

    # 接收服务器回复
    data = client_socket.recv(1024)
    if not data:
        print("服务器已断开")
        break
    print(f"服务器：{data.decode('utf-8')}")

client_socket.close()
```

### 2.5 ⚠️ 注意事项

1. **`recv()` 是阻塞的**：没有数据时，程序会停在这里等待。
2. **`send()` 不保证一次发完**：发送大数据时可能只发一部分。建议使用 `sendall()`；它成功返回表示数据已完整交给本机 socket 发送，失败会抛异常。
3. **缓冲区大小**：`recv(1024)` 中的 1024 是一次接收的最大字节数。如果数据超过这个大小，需要多次接收。

### 📝 本关小结

你现在已经能让两个程序"对话"了！下一关我们将处理一个 TCP 编程的核心难题——**粘包**，并实现文件传输。

---

## 关卡 3：发送字符串与文件（中等）

### 📋 学习目标

- 理解 TCP 的"粘包"问题
- 设计简单的应用层协议
- 实现大文件的可靠传输

### 3.1 什么是粘包？

**粘包问题**是 TCP 编程中最常见的"坑"。

TCP 是一个**流式协议**，数据像水流一样连续传输，**没有明确的"消息边界"**。

**举个例子：**

客户端连续发送：
```python
client.send(b"Hello")
client.send(b"World")
```

服务端的 `recv` 可能收到：
- `Hello` 一次，`World` 一次 ✅（理想情况）
- `HelloWorld` 一次粘在一起 ⚠️（这就是"粘包"）
- `Hel`、`loWor`、`ld` 分三次 ⚠️（数据被切碎）

**为什么会粘包？**

- 根本原因是 TCP 是字节流协议，没有应用层消息边界
- Nagle 算法、发送缓冲区、接收缓冲区会让多次发送的数据表现为合并或拆分

### 3.2 解决粘包：自定义协议

最经典的方案是：**在数据前加上长度信息**。

**协议设计**：

```
┌──────────────────┬──────────────────────┐
│  4 字节长度信息   │       实际数据         │
└──────────────────┴──────────────────────┘
```

例如要发送 `"Hello"`（5 字节），实际发送的数据是：
```
[0x00, 0x00, 0x00, 0x05] + [H, e, l, l, o]
```

接收时：**先读 4 字节得到长度 5，再读 5 字节得到数据**。这样就解决了边界问题。

### 3.3 整数与字节的互转：`struct` 模块

Python 中处理整数和字节的转换用 `struct`：

```python
import struct

# 将整数 5 转换为 4 字节
length_bytes = struct.pack('!I', 5)
print(length_bytes)
# 输出：b'\x00\x00\x00\x05'

# 将 4 字节转换为整数
length = struct.unpack('!I', length_bytes)[0]
print(length)
# 输出：5
```

**`struct.pack` 格式说明**：
- `!`：**网络字节序**（大端序，跨平台标准）
- `I`：**无符号整数**（4 字节）

`unpack` 返回的是元组，所以加 `[0]` 取出第一个元素。

### 3.4 发送字符串（解决粘包版）

我们将通用的"安全收发"函数封装好：

**服务端 `server.py`：**

```python
import socket
import struct

def recv_all(sock, n):
    """精确接收 n 字节数据，不多不少"""
    data = b''
    while len(data) < n:
        # 每次最多接收剩余字节数
        packet = sock.recv(n - len(data))
        if not packet:
            # 连接断开
            return None
        data += packet
    return data

def recv_msg(sock):
    """接收一条完整消息（长度+数据）"""
    # 1. 先接收 4 字节的长度
    raw_length = recv_all(sock, 4)
    if not raw_length:
        return None
    msg_length = struct.unpack('!I', raw_length)[0]
    # 2. 再接收 msg_length 字节的数据
    return recv_all(sock, msg_length)

# 创建服务器
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind(('127.0.0.1', 8888))
server_socket.listen(5)
print("等待连接...")

client_socket, addr = server_socket.accept()
print(f"连接来自 {addr}")

while True:
    msg = recv_msg(client_socket)
    if msg is None:
        print("客户端断开")
        break
    print(f"收到：{msg.decode('utf-8')}")

client_socket.close()
server_socket.close()
```

**客户端 `client.py`：**

```python
import socket
import struct

def send_msg(sock, msg_bytes):
    """发送一条消息（带 4 字节长度前缀）"""
    # 将长度打包为 4 字节，拼接在数据前
    packet = struct.pack('!I', len(msg_bytes)) + msg_bytes
    sock.sendall(packet)

client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect(('127.0.0.1', 8888))

# 连续发送多条消息（测试粘包是否解决）
messages = ["Hello", "World", "Python", "Socket 编程", "完美！"]
for m in messages:
    send_msg(client_socket, m.encode('utf-8'))
    print(f"发送：{m}")

client_socket.close()
```

运行后，服务端会**完美地分条接收**这 5 条消息，不会粘在一起！

### 3.5 文件传输

文件传输的核心思路：

```
1. 发送方先告诉对方：文件名、文件大小
2. 然后逐块读取文件并发送
3. 接收方按大小接收数据，写入文件
```

**服务端 `file_server.py`（接收文件）：**

```python
import socket
import struct

def recv_all(sock, n):
    data = b''
    while len(data) < n:
        packet = sock.recv(n - len(data))
        if not packet:
            return None
        data += packet
    return data

# 创建服务器
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind(('127.0.0.1', 8888))
server_socket.listen(5)
print("文件服务器已启动...")

client_socket, addr = server_socket.accept()
print(f"接受连接：{addr}")

# 1. 接收文件名长度（4 字节）
filename_length_data = recv_all(client_socket, 4)
filename_length = struct.unpack('!I', filename_length_data)[0]

# 2. 接收文件名
filename = recv_all(client_socket, filename_length).decode('utf-8')
print(f"准备接收文件：{filename}")

# 3. 接收文件大小（8 字节，支持大文件）
file_size_data = recv_all(client_socket, 8)
file_size = struct.unpack('!Q', file_size_data)[0]  # Q = 8 字节无符号整数
print(f"文件大小：{file_size} 字节")

# 4. 接收文件内容
save_path = f"received_{filename}"
with open(save_path, 'wb') as f:
    received_size = 0
    while received_size < file_size:
        # 每次最多接收 4096 字节，但不超过剩余字节
        chunk_size = min(4096, file_size - received_size)
        chunk = client_socket.recv(chunk_size)
        if not chunk:
            print("传输中断")
            break
        f.write(chunk)
        received_size += len(chunk)
        # 显示进度
        progress = (received_size / file_size) * 100
        print(f"\r接收进度：{progress:.2f}%", end='')

print(f"\n✅ 文件已保存为：{save_path}")
client_socket.close()
server_socket.close()
```

**客户端 `file_client.py`（发送文件）：**

```python
import socket
import struct
import os

def send_file(sock, filepath):
    # 检查文件是否存在
    if not os.path.exists(filepath):
        print(f"❌ 文件不存在：{filepath}")
        return

    # 获取文件信息
    filename = os.path.basename(filepath)
    file_size = os.path.getsize(filepath)

    # 1. 发送文件名长度（4 字节）+ 文件名
    filename_bytes = filename.encode('utf-8')
    sock.sendall(struct.pack('!I', len(filename_bytes)))
    sock.sendall(filename_bytes)

    # 2. 发送文件大小（8 字节）
    sock.sendall(struct.pack('!Q', file_size))

    # 3. 发送文件内容
    print(f"开始发送文件：{filename}（{file_size} 字节）")
    with open(filepath, 'rb') as f:
        sent_size = 0
        while True:
            chunk = f.read(4096)
            if not chunk:
                break
            sock.sendall(chunk)
            sent_size += len(chunk)
            progress = (sent_size / file_size) * 100
            print(f"\r发送进度：{progress:.2f}%", end='')

    print("\n✅ 文件发送完成！")

# 连接服务器
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect(('127.0.0.1', 8888))

# 发送文件
filepath = input("请输入要发送的文件路径：")
send_file(client_socket, filepath)

client_socket.close()
```

### 3.6 `struct` 格式速查表

| 符号 | 类型 | 字节数 |
|---|---|---|
| `b` | signed char | 1 |
| `B` | unsigned char | 1 |
| `h` | short | 2 |
| `H` | unsigned short | 2 |
| `i` / `l` | int / long | 4 |
| `I` / `L` | unsigned int / long | 4 |
| `q` | long long | 8 |
| `Q` | unsigned long long | 8 |

**前缀符号**：
- `<`：小端序
- `>`：大端序
- `!`：网络字节序（等同于大端序，推荐用这个）

### 📝 本关小结

本关你学会了：
- ✓ 粘包问题的原理与解决方案
- ✓ 自定义"长度+数据"协议
- ✓ 使用 `struct` 模块处理二进制数据
- ✓ 完整的文件传输实现

---

## 关卡 4：多客户端连接（中等）

### 📋 学习目标

- 理解为什么之前的服务器只能服务一个客户端
- 学会使用多线程处理并发连接
- 实现一个简单的聊天室

### 4.1 单线程服务器的局限

回顾之前的服务器代码：

```python
client_socket, addr = server_socket.accept()  # 只接受一个连接
while True:
    data = client_socket.recv(1024)  # 阻塞在这里
    # ...
```

`accept()` 只接受了一个客户端，之后程序就陷入了与该客户端的通信循环。其他客户端无法连接！

### 4.2 解决方案：多线程

**思路**：每当有新客户端连接，就开启一个新线程专门处理它。主线程继续 `accept` 接受新连接。

**Python 的 `threading` 模块入门：**

```python
import threading

# 定义一个函数
def hello(name):
    print(f"Hello, {name}!")

# 创建线程
t = threading.Thread(target=hello, args=("World",))

# 启动线程
t.start()

# 等待线程结束（可选）
t.join()
```

**关键参数**：
- `target`：线程要执行的函数
- `args`：传给函数的参数（必须是元组，单参数也要加逗号）
- `daemon=True`：设为守护线程，主线程结束时它自动结束

### 4.3 多线程服务器实现

**服务端 `multi_server.py`：**

```python
import socket
import threading

def handle_client(client_socket, addr):
    """处理单个客户端的函数（在独立线程中运行）"""
    print(f"[新连接] {addr} 已连接")

    try:
        while True:
            data = client_socket.recv(1024)
            if not data:
                break

            message = data.decode('utf-8')
            print(f"[{addr}] {message}")

            # 回复客户端
            response = f"已收到：{message}"
            client_socket.send(response.encode('utf-8'))

            if message.lower() == 'bye':
                break
    except Exception as e:
        print(f"[错误] 与 {addr} 通信出错：{e}")
    finally:
        client_socket.close()
        print(f"[断开] {addr} 已断开")

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('127.0.0.1', 8888))
    server_socket.listen(5)
    print("[启动] 服务器正在监听...")

    while True:
        # 主线程负责接受新连接
        client_socket, addr = server_socket.accept()

        # 为每个新客户端创建独立线程
        thread = threading.Thread(
            target=handle_client,
            args=(client_socket, addr)
        )
        thread.daemon = True  # 守护线程
        thread.start()

        # 显示当前活跃连接数（减 1 是因为不包括主线程）
        print(f"[活跃连接] {threading.active_count() - 1}")

if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\n服务器关闭")
```

**客户端 `multi_client.py`：**

```python
import socket

def main():
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect(('127.0.0.1', 8888))
    print("[已连接] 连接到服务器")

    while True:
        msg = input("请输入消息（bye 退出）：")
        client_socket.send(msg.encode('utf-8'))

        if msg.lower() == 'bye':
            break

        data = client_socket.recv(1024)
        print(f"服务器：{data.decode('utf-8')}")

    client_socket.close()

if __name__ == "__main__":
    main()
```

**测试方法**：
1. 启动服务端：`python multi_server.py`
2. 打开**多个终端**，每个都运行 `python multi_client.py`
3. 在各个客户端输入消息，观察服务端的反应

### 4.4 进阶实战：聊天室广播

让我们做一个更有趣的功能：**当一个客户端发消息，服务器广播给所有客户端**。

**广播服务端 `broadcast_server.py`：**

```python
import socket
import threading

# 全局变量：存储所有已连接客户端的 socket
clients = []
# 锁：保证多线程操作 clients 的安全
lock = threading.Lock()

def broadcast(message, sender_socket=None):
    """广播消息给除发送者外的所有客户端"""
    with lock:
        for client in clients[:]:  # 使用副本遍历，避免迭代时修改
            if client != sender_socket:
                try:
                    client.send(message)
                except:
                    # 发送失败，可能已断开
                    clients.remove(client)

def handle_client(client_socket, addr):
    print(f"[新连接] {addr}")

    # 添加到客户端列表
    with lock:
        clients.append(client_socket)

    # 通知所有人：有新人加入
    join_msg = f"[系统] 用户 {addr} 加入了聊天室".encode('utf-8')
    broadcast(join_msg, client_socket)

    try:
        while True:
            data = client_socket.recv(1024)
            if not data:
                break

            message = f"[{addr}] {data.decode('utf-8')}"
            print(message)
            # 广播消息
            broadcast(message.encode('utf-8'), client_socket)
    except:
        pass
    finally:
        # 从列表移除
        with lock:
            if client_socket in clients:
                clients.remove(client_socket)
        client_socket.close()
        # 通知所有人有人离开
        leave_msg = f"[系统] 用户 {addr} 离开了聊天室".encode('utf-8')
        broadcast(leave_msg)
        print(f"[断开] {addr}")

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('127.0.0.1', 8888))
    server_socket.listen(5)
    print("[启动] 聊天室服务器已启动...")

    while True:
        client_socket, addr = server_socket.accept()
        thread = threading.Thread(
            target=handle_client,
            args=(client_socket, addr)
        )
        thread.daemon = True
        thread.start()

if __name__ == "__main__":
    main()
```

**广播客户端 `broadcast_client.py`：**

```python
import socket
import threading

def receive_messages(sock):
    """持续接收服务器消息的线程"""
    while True:
        try:
            data = sock.recv(1024)
            if not data:
                break
            print(f"\n{data.decode('utf-8')}")
            print("请输入消息：", end='', flush=True)
        except:
            break

def main():
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect(('127.0.0.1', 8888))
    print("已连接到聊天室")

    # 启动接收线程（持续监听服务器消息）
    recv_thread = threading.Thread(target=receive_messages, args=(client_socket,))
    recv_thread.daemon = True
    recv_thread.start()

    try:
        # 主线程负责发送消息
        while True:
            msg = input("请输入消息：")
            if msg.lower() == 'bye':
                break
            client_socket.send(msg.encode('utf-8'))
    except KeyboardInterrupt:
        pass
    finally:
        client_socket.close()

if __name__ == "__main__":
    main()
```

### 4.5 🤔 思考题

**Q1：为什么客户端也需要多线程？**

A：因为客户端既要发送消息（`input` 阻塞），又要接收消息（`recv` 阻塞）。在同一个线程中，会出现互相阻塞的冲突。一个线程负责发送，一个负责接收，互不干扰。

**Q2：线程数会无限增长吗？**

A：是的。每个连接对应一个线程，连接数过多会消耗大量资源。生产环境通常使用：
- 线程池（`concurrent.futures.ThreadPoolExecutor`）
- 协程（`asyncio`）
- 多路复用（`select` / `epoll`）

### 📝 本关小结

你已经掌握了：
- ✓ 多线程的基本使用
- ✓ 并发服务器的实现
- ✓ 简易聊天室广播机制
- ✓ 线程锁的使用（避免数据竞争）

---

## 关卡 5：socket 传输加密图片（困难）

### 📋 学习目标

- 了解加密的基本概念
- 学会使用 AES 对称加密
- 实现图片的加密传输与解密保存

### 5.1 为什么需要加密？

网络上"裸传"数据是不安全的，可能被中途截获：

```
不加密：客户端 → [图片二进制数据] → 服务器
                       ↑
                 黑客可截获并查看
```

加密后：

```
加密：客户端 → [一堆乱码] → 服务器
                  ↑
             黑客截获也看不懂
```

### 5.2 加密基础概念

**对称加密**：加密和解密用**同一把钥匙**。
- ✅ 优点：速度快
- ❌ 缺点：密钥如何安全传递？
- 🔑 代表算法：**AES**、DES

**非对称加密**：加密用公钥，解密用私钥。
- ✅ 优点：安全
- ❌ 缺点：速度慢
- 🔑 代表算法：**RSA**

> 💡 **实际应用**：通常组合使用——先用非对称加密传递对称密钥，再用对称密钥加密大量数据。这就是 HTTPS 的核心思路。

本关使用 **AES** 对称加密（为简化流程，假设双方已经共享了密钥）。

### 5.3 安装加密库

```bash
pip install cryptography
```

`cryptography` 是 Python 中最常用的加密库。

### 5.4 AES 加密基础

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import padding
from cryptography.hazmat.backends import default_backend
import os

# 生成 256 位（32 字节）的密钥
key = os.urandom(32)

# 生成 128 位（16 字节）的 IV（初始化向量）
iv = os.urandom(16)

# 创建加密器（CBC 模式）
cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())

# CBC 模式要求明文长度是 16 字节的倍数，所以先填充
plaintext = b"Hello AES CBC"
padder = padding.PKCS7(128).padder()
padded_data = padder.update(plaintext) + padder.finalize()

# 加密
encryptor = cipher.encryptor()
ciphertext = encryptor.update(padded_data) + encryptor.finalize()

# 解密
decryptor = cipher.decryptor()
padded_plaintext = decryptor.update(ciphertext) + decryptor.finalize()
unpadder = padding.PKCS7(128).unpadder()
plaintext = unpadder.update(padded_plaintext) + unpadder.finalize()
```

**关键概念**：
- **密钥（key）**：加密的核心。AES-256 用 32 字节密钥
- **IV（Initialization Vector）**：初始化向量，**每次加密都应该不同**，增加随机性
- **模式**：CBC、ECB、CFB 等，CBC 较常用、ECB 不安全
- **块大小**：AES 块大小固定为 16 字节，明文长度必须是 16 的倍数 → 需要**填充**

### 5.5 PKCS7 填充

明文长度不是 16 的倍数时，需要在末尾填充字节：

```python
from cryptography.hazmat.primitives import padding

# 填充
padder = padding.PKCS7(128).padder()  # 128 位 = 16 字节块大小
padded_data = padder.update(data) + padder.finalize()

# 去除填充
unpadder = padding.PKCS7(128).unpadder()
data = unpadder.update(padded_data) + unpadder.finalize()
```

### 5.6 完整实现

**项目结构**：
```
crypto_image/
├── crypto_helper.py   # 加密工具
├── server.py          # 服务端（接收并解密）
├── client.py          # 客户端（加密并发送）
└── test.jpg           # 待发送的测试图片
```

#### 📄 `crypto_helper.py` — 加密工具类

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import padding
from cryptography.hazmat.backends import default_backend
import os

class AESCipher:
    def __init__(self, key):
        """
        初始化 AES 加密器
        :param key: 32 字节的密钥（256 位 AES）
        """
        if len(key) != 32:
            raise ValueError("密钥必须是 32 字节")
        self.key = key

    def encrypt(self, data):
        """
        加密数据
        :param data: 待加密的字节数据
        :return: iv + 密文（IV 在前 16 字节）
        """
        # 1. 生成随机 IV
        iv = os.urandom(16)

        # 2. PKCS7 填充
        padder = padding.PKCS7(128).padder()
        padded_data = padder.update(data) + padder.finalize()

        # 3. 加密
        cipher = Cipher(
            algorithms.AES(self.key),
            modes.CBC(iv),
            backend=default_backend()
        )
        encryptor = cipher.encryptor()
        ciphertext = encryptor.update(padded_data) + encryptor.finalize()

        # 4. 返回 IV + 密文（IV 不需要保密，解密时需要）
        return iv + ciphertext

    def decrypt(self, data):
        """
        解密数据
        :param data: iv(16字节) + 密文
        :return: 原始明文
        """
        # 1. 分离 IV 和密文
        iv = data[:16]
        ciphertext = data[16:]

        # 2. 解密
        cipher = Cipher(
            algorithms.AES(self.key),
            modes.CBC(iv),
            backend=default_backend()
        )
        decryptor = cipher.decryptor()
        padded_data = decryptor.update(ciphertext) + decryptor.finalize()

        # 3. 去除填充
        unpadder = padding.PKCS7(128).unpadder()
        return unpadder.update(padded_data) + unpadder.finalize()


# 单元测试
if __name__ == "__main__":
    key = b'0123456789abcdef0123456789abcdef'  # 32 字节
    cipher = AESCipher(key)

    original = b"Hello, this is a secret message!"
    encrypted = cipher.encrypt(original)
    decrypted = cipher.decrypt(encrypted)

    print(f"原文：{original}")
    print(f"密文：{encrypted.hex()[:40]}...")
    print(f"解密后：{decrypted}")
    assert original == decrypted, "解密结果不匹配！"
    print("✅ 测试通过")
```

#### 📄 `client.py` — 客户端

```python
import socket
import struct
import os
from crypto_helper import AESCipher

# 注意：实际项目中，密钥应通过安全方式传递，这里硬编码仅用于演示
KEY = b'0123456789abcdef0123456789abcdef'

def send_encrypted_image(filepath):
    if not os.path.exists(filepath):
        print(f"❌ 文件不存在：{filepath}")
        return

    # 1. 读取图片
    with open(filepath, 'rb') as f:
        image_data = f.read()
    print(f"📷 图片大小：{len(image_data)} 字节")

    # 2. 加密
    cipher = AESCipher(KEY)
    encrypted_data = cipher.encrypt(image_data)
    print(f"🔒 加密后大小：{len(encrypted_data)} 字节")

    # 3. 连接服务器
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(('127.0.0.1', 8888))
    print("✅ 已连接服务器")

    try:
        # 4. 发送文件名长度 + 文件名
        filename = os.path.basename(filepath).encode('utf-8')
        sock.sendall(struct.pack('!I', len(filename)))
        sock.sendall(filename)

        # 5. 发送加密数据长度（8 字节）
        sock.sendall(struct.pack('!Q', len(encrypted_data)))

        # 6. 分块发送加密数据
        sent = 0
        chunk_size = 4096
        while sent < len(encrypted_data):
            chunk = encrypted_data[sent:sent + chunk_size]
            sock.sendall(chunk)
            sent += len(chunk)
            progress = (sent / len(encrypted_data)) * 100
            print(f"\r📤 发送进度：{progress:.2f}%", end='')

        print("\n✅ 图片发送完成！")
    finally:
        sock.close()

if __name__ == "__main__":
    filepath = input("请输入图片路径：")
    send_encrypted_image(filepath)
```

#### 📄 `server.py` — 服务端

```python
import socket
import struct
import os
from crypto_helper import AESCipher

KEY = b'0123456789abcdef0123456789abcdef'

def recv_all(sock, n):
    """精确接收 n 字节数据"""
    data = b''
    while len(data) < n:
        packet = sock.recv(min(n - len(data), 4096))
        if not packet:
            return None
        data += packet
    return data

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('127.0.0.1', 8888))
    server_socket.listen(5)
    print("🚀 图片接收服务器已启动...")

    while True:
        client_socket, addr = server_socket.accept()
        print(f"\n[新连接] {addr}")

        try:
            # 1. 接收文件名长度
            filename_len_data = recv_all(client_socket, 4)
            if filename_len_data is None:
                raise ConnectionError("文件名长度接收失败")
            filename_len = struct.unpack('!I', filename_len_data)[0]

            # 2. 接收文件名
            filename_data = recv_all(client_socket, filename_len)
            if filename_data is None:
                raise ConnectionError("文件名接收失败")
            filename = filename_data.decode('utf-8')
            print(f"📷 接收文件：{filename}")

            # 3. 接收加密数据长度
            data_len_data = recv_all(client_socket, 8)
            if data_len_data is None:
                raise ConnectionError("数据长度接收失败")
            data_len = struct.unpack('!Q', data_len_data)[0]
            print(f"📦 加密数据长度：{data_len} 字节")

            # 4. 接收加密数据
            received = 0
            encrypted_data = b''
            while received < data_len:
                chunk_size = min(4096, data_len - received)
                chunk = client_socket.recv(chunk_size)
                if not chunk:
                    break
                encrypted_data += chunk
                received += len(chunk)
                progress = (received / data_len) * 100
                print(f"\r📥 接收进度：{progress:.2f}%", end='')
            print()
            if received != data_len:
                raise ConnectionError("加密数据接收不完整")

            # 5. 解密
            cipher = AESCipher(KEY)
            decrypted_data = cipher.decrypt(encrypted_data)
            print(f"🔓 解密后大小：{len(decrypted_data)} 字节")

            # 6. 保存图片
            save_path = f"received_{filename}"
            with open(save_path, 'wb') as f:
                f.write(decrypted_data)
            print(f"✅ 图片已保存为：{save_path}")

        except Exception as e:
            print(f"❌ 出错：{e}")
        finally:
            client_socket.close()

if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\n服务器关闭")
```

### 5.7 运行测试

1. 准备一张图片，放到客户端同目录下
2. 启动服务端：`python server.py`
3. 启动客户端：`python client.py`，输入图片路径
4. 检查服务端目录，应该有 `received_xxx.jpg`
5. 打开图片，确认能正常显示 ✅

### 5.8 🎓 安全性提升建议

我们的实现还有几个不足：

| 问题 | 改进方案 |
|---|---|
| 密钥硬编码 | 使用密钥管理服务（KMS）或环境变量 |
| 无完整性校验 | 添加 HMAC，或使用 AES-GCM 模式 |
| 密钥固定 | 使用 Diffie-Hellman 等密钥交换协议 |

进阶可以了解：
- **TLS/SSL**：行业标准的加密通信协议
- **Python 的 `ssl` 模块**：把普通 socket 升级为安全 socket
- **AES-GCM 模式**：既加密又验证完整性

### 📝 本关小结

本关你学会了：
- ✓ 加密的基本概念
- ✓ AES 对称加密的使用
- ✓ 加密图片的完整传输流程

🎉 **恭喜你完成了 TCP Socket 编程课程的所有关卡！**

---

# 🎯 课程 2：UDP 套接字编程

## 关卡 1：UDP 初体验（简单）

### 📋 学习目标

- 理解 UDP 与 TCP 的区别
- 学会创建 UDP 套接字
- 理解无连接通信的特点

### 1.1 UDP vs TCP 复习

| 特性 | TCP | UDP |
|---|---|---|
| 连接方式 | 面向连接 | 无连接 |
| 可靠性 | 可靠 | 不可靠 |
| 速度 | 慢 | 快 |
| 数据边界 | 字节流（有粘包） | 数据报（有边界） |
| 类比 | 寄挂号信 | 发短信 |
| 应用 | HTTP、邮件 | 视频流、游戏、DNS |

### 1.2 创建 UDP 套接字

```python
import socket
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```

注意区别：
- TCP 用 `socket.SOCK_STREAM`
- UDP 用 `socket.SOCK_DGRAM`（**DGRAM = Datagram**，数据报）

### 1.3 UDP 的两个核心方法

UDP **没有** `connect`、`listen`、`accept`，因为无连接！直接使用：

| 方法 | 作用 | 返回值 |
|---|---|---|
| `sendto(data, (host, port))` | 发送数据到指定地址 | 已发送字节数 |
| `recvfrom(bufsize)` | 接收数据 | `(data, addr)` 元组 |

### 1.4 第一个 UDP 程序

**UDP 服务端 `udp_server.py`：**

```python
import socket

# 创建 UDP 套接字
server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# 绑定地址和端口（注意：UDP 不需要 listen）
server_socket.bind(('127.0.0.1', 9999))

print("UDP 服务器已启动，等待数据...")

# 接收数据（返回 数据 + 发送者地址）
data, client_addr = server_socket.recvfrom(1024)
print(f"收到来自 {client_addr} 的数据：{data.decode('utf-8')}")

# 关闭套接字
server_socket.close()
```

**UDP 客户端 `udp_client.py`：**

```python
import socket

# 创建 UDP 套接字
client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# 直接发送（不需要 connect！）
message = "Hello, UDP Server!".encode('utf-8')
client_socket.sendto(message, ('127.0.0.1', 9999))
print("数据已发送")

client_socket.close()
```

### 1.5 流程对比

**TCP 服务端流程**：
```
socket() → bind() → listen() → accept() → recv() → send() → close()
```

**UDP 服务端流程**：
```
socket() → bind() → recvfrom() → sendto() → close()
```

> ✨ UDP 简化了很多步骤！

### 1.6 ⚠️ 重要特性

1. **无需 `listen` 和 `accept`**：UDP 不建立连接
2. **每次发送都指定目标地址**：`sendto` 必须带目标
3. **数据报有边界**：发送 100 字节，接收方一次 `recvfrom` 就能拿到完整的 100 字节
4. **可能丢失**：UDP 不保证送达

### 📝 本关小结

你已经会用 UDP 了！是不是比 TCP 简单很多？

---

## 关卡 2：UDP 数据收发（简单）

### 📋 学习目标

- 实现 UDP 的双向通信
- 实现持续的数据收发
- 体验 UDP 天然的多客户端支持

### 2.1 双向通信

让服务器也能回复客户端。

**服务端 `udp_echo_server.py`：**

```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(('127.0.0.1', 9999))
print("UDP 回声服务器已启动...")

while True:
    # 接收数据
    data, client_addr = server_socket.recvfrom(1024)
    msg = data.decode('utf-8')
    print(f"[{client_addr}] {msg}")

    # 回复客户端
    response = f"已收到：{msg}"
    server_socket.sendto(response.encode('utf-8'), client_addr)

server_socket.close()
```

**客户端 `udp_echo_client.py`：**

```python
import socket

client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_addr = ('127.0.0.1', 9999)

print("UDP 客户端启动，输入 bye 退出")

while True:
    msg = input("我：")
    if not msg:
        continue

    # 发送
    client_socket.sendto(msg.encode('utf-8'), server_addr)

    if msg.lower() == 'bye':
        break

    # 接收回复
    data, _ = client_socket.recvfrom(1024)
    print(f"服务器：{data.decode('utf-8')}")

client_socket.close()
```

### 2.2 ✨ UDP 天然支持多客户端

注意一个有趣的现象：**UDP 服务器不需要多线程就能处理多个客户端！**

**为什么？**

因为 UDP 是无连接的，服务器只需要：
1. 在 `recvfrom` 时获取发送者地址
2. 用 `sendto` 回复
3. 不需要为每个客户端维护独立连接

**测试**：
1. 启动服务端
2. **同时**启动多个客户端
3. 服务器能依次响应所有客户端的消息

### 2.3 ⚠️ 注意事项

1. **缓冲区大小**：`recvfrom(1024)` 中的 1024 是缓冲区大小。**如果数据报超过这个大小，会被截断！**
2. **数据报最大大小**：UDP 理论上最大 65507 字节，但实际网络中受 MTU 限制（一般约 1472 字节），过大会被分片可能丢失
3. **乱序**：UDP 不保证顺序，发送 1、2、3，可能收到 2、1、3

### 📝 本关小结

UDP 的收发非常简单。但要时刻记住它的特性：**无连接、不可靠、无序**。

---

## 关卡 3：UDP 计算丢包率（简单）

### 📋 学习目标

- 理解丢包的概念
- 学会设计带序号的数据包
- 实现简单的丢包率统计

### 3.1 什么是丢包？

UDP 不保证数据送达。发送 100 个包，可能只收到 95 个，丢包率就是 5%。

**为什么会丢包？**
- 网络拥塞
- 路由器丢弃数据包
- 信号干扰（无线网络）
- 接收方处理不过来

### 3.2 统计丢包率的思路

```
1. 发送方给每个包加上序号（1, 2, 3, ...）
2. 总共发送 N 个包
3. 接收方记录所有收到的序号
4. 丢包率 = (N - 收到数) / N × 100%
```

### 3.3 完整实现

**发送方 `udp_send.py`：**

```python
import socket
import time

client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_addr = ('127.0.0.1', 9999)

TOTAL_PACKETS = 100

print(f"开始发送 {TOTAL_PACKETS} 个数据包...")
for i in range(1, TOTAL_PACKETS + 1):
    # 构造数据包：序号 | 内容（用 | 分隔）
    message = f"{i}|这是第 {i} 个数据包"
    client_socket.sendto(message.encode('utf-8'), server_addr)
    print(f"已发送：第 {i} 个")
    time.sleep(0.01)  # 稍稍延时，避免发送过快被丢

# 发送结束标志，并带上总包数；真实网络中 END 也可能丢失，接收方仍应设置超时
client_socket.sendto(f"END|{TOTAL_PACKETS}".encode('utf-8'), server_addr)
print("✅ 发送完成")
client_socket.close()
```

**接收方 `udp_recv.py`：**

```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(('127.0.0.1', 9999))
server_socket.settimeout(5)
print("等待接收数据...")

received_seqs = set()  # 已收到的序号集合
expected = None  # 发送方声明的期望总数

while True:
    try:
        data, addr = server_socket.recvfrom(1024)
    except socket.timeout:
        print("等待超时，停止统计")
        break
    msg = data.decode('utf-8')

    if msg.startswith("END|"):
        expected = int(msg.split('|', 1)[1])
        break

    # 解析序号
    seq_str, content = msg.split('|', 1)
    seq = int(seq_str)
    received_seqs.add(seq)

    print(f"收到：第 {seq} 个")

# 计算丢包率
if expected is None:
    expected = max(received_seqs) if received_seqs else 0
received = len(received_seqs) # 实际收到的数量
lost = expected - received    # 丢失的包数
loss_rate = (lost / expected) * 100 if expected > 0 else 0

print("\n========== 统计结果 ==========")
print(f"应收数据包：{expected}")
print(f"实收数据包：{received}")
print(f"丢失数据包：{lost}")
print(f"丢包率：{loss_rate:.2f}%")

# 显示丢失的序号
all_seqs = set(range(1, expected + 1))
lost_seqs = all_seqs - received_seqs
if lost_seqs:
    print(f"丢失的序号：{sorted(lost_seqs)}")

server_socket.close()
```

### 3.4 模拟丢包测试

在本地测试，通常不会真的丢包（本地通信很稳定）。要模拟丢包，可以让接收方按概率丢弃部分包：

```python
# udp_recv_loss.py（模拟丢包版本）
import socket
import random

server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(('127.0.0.1', 9999))
server_socket.settimeout(5)
print("等待接收数据（模拟 10% 丢包）...")

LOSS_RATE = 0.1  # 模拟 10% 的丢包率
received_seqs = set()
expected = None

while True:
    try:
        data, addr = server_socket.recvfrom(1024)
    except socket.timeout:
        print("等待超时，停止统计")
        break
    msg = data.decode('utf-8')

    if msg.startswith("END|"):
        expected = int(msg.split('|', 1)[1])
        break

    seq_str, content = msg.split('|', 1)
    seq = int(seq_str)

    # 模拟丢包：按概率丢弃
    if random.random() < LOSS_RATE:
        print(f"💥 模拟丢失：第 {seq} 个")
        continue

    received_seqs.add(seq)
    print(f"✅ 收到：第 {seq} 个")

# 统计
if expected is None:
    expected = max(received_seqs) if received_seqs else 0
received = len(received_seqs)
lost = expected - received
loss_rate = (lost / expected) * 100 if expected > 0 else 0

print(f"\n应收：{expected}，实收：{received}，丢失：{lost}，丢包率：{loss_rate:.2f}%")
server_socket.close()
```

### 📝 本关小结

你已经学会：
- ✓ 数据包序号的设计
- ✓ 丢包率的计算
- ✓ 简单的丢包模拟

---

## 关卡 4：创建 UDP Ping 程序（简单）

### 📋 学习目标

- 理解 ping 程序的工作原理
- 实现 RTT（往返时间）计算
- 处理超时

### 4.1 什么是 ping？

`ping` 是网络中最常用的工具，用于：
- 测试网络是否通畅
- 测量网络延迟（**RTT**，Round-Trip Time，往返时间）
- 检测丢包率

**工作原理**：

```
1. 客户端发送一个数据包 → 记录时间 t1
2. 服务器收到后立即原样回复
3. 客户端收到回复 → 记录时间 t2
4. RTT = t2 - t1
```

> 💡 **真实的 ping 使用 ICMP 协议**，但 ICMP 需要管理员权限。我们用 UDP 实现一个**类似** ping 的工具。

### 4.2 超时机制

如果数据包丢失，客户端会一直等待。需要设置超时：

```python
sock.settimeout(1.0)  # 设置 1 秒超时

try:
    data, addr = sock.recvfrom(1024)
except socket.timeout:
    print("超时！")
```

`settimeout(秒数)` 设置 socket 的超时时间，超过后会抛出 `socket.timeout` 异常。

### 4.3 完整实现

**服务端 `ping_server.py`（响应 ping）：**

```python
import socket
import random
import time

server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(('127.0.0.1', 9999))
print("Ping 服务器已启动...")

LOSS_RATE = 0.2  # 模拟 20% 丢包率

while True:
    try:
        data, client_addr = server_socket.recvfrom(1024)

        # 模拟丢包：以一定概率不回复
        if random.random() < LOSS_RATE:
            print(f"💥 模拟丢包，不回复 {client_addr}")
            continue

        # 模拟随机网络延迟
        delay = random.uniform(0, 0.1)
        time.sleep(delay)

        # 原样回复（这就是 echo）
        server_socket.sendto(data, client_addr)
        print(f"✅ 已回复 {client_addr}")
    except KeyboardInterrupt:
        break

server_socket.close()
```

**客户端 `ping_client.py`（发起 ping）：**

```python
import socket
import time
import sys

def ping(target_host, target_port, count=10, timeout=1.0):
    """
    模拟 ping 程序
    :param target_host: 目标主机 IP
    :param target_port: 目标端口
    :param count: ping 多少次
    :param timeout: 超时时间（秒）
    """
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.settimeout(timeout)

    target = (target_host, target_port)
    print(f"正在 ping {target_host}:{target_port}")
    print("=" * 50)

    sent = 0       # 已发送数量
    received = 0   # 已接收数量
    rtts = []      # 所有的 RTT

    try:
        for i in range(1, count + 1):
            sent += 1
            send_time = time.time()
            message = f"PING_{i}_{send_time}"

            try:
                # 发送
                sock.sendto(message.encode('utf-8'), target)

                # 接收回复
                data, addr = sock.recvfrom(1024)
                recv_time = time.time()

                # 计算 RTT（转毫秒）
                rtt = (recv_time - send_time) * 1000
                rtts.append(rtt)
                received += 1

                print(f"序号 {i}: rtt={rtt:.2f} ms")

            except socket.timeout:
                print(f"序号 {i}: 请求超时")

            time.sleep(0.5)  # 间隔 0.5 秒，避免发送过快

    except KeyboardInterrupt:
        print("\n用户中断")

    finally:
        sock.close()

    # 输出统计
    print("=" * 50)
    print("===== ping 统计 =====")
    print(f"已发送：{sent}，已接收：{received}")
    loss_rate = ((sent - received) / sent) * 100 if sent > 0 else 0
    print(f"丢包率：{loss_rate:.1f}%")

    if rtts:
        print(f"最小 RTT: {min(rtts):.2f} ms")
        print(f"最大 RTT: {max(rtts):.2f} ms")
        print(f"平均 RTT: {sum(rtts) / len(rtts):.2f} ms")

if __name__ == "__main__":
    # 支持命令行参数：python ping_client.py 127.0.0.1 9999 20
    host = sys.argv[1] if len(sys.argv) > 1 else '127.0.0.1'
    port = int(sys.argv[2]) if len(sys.argv) > 2 else 9999
    count = int(sys.argv[3]) if len(sys.argv) > 3 else 10

    ping(host, port, count)
```

### 4.4 使用方法

```bash
# 启动服务端
python ping_server.py

# 启动客户端（默认 ping 10 次）
python ping_client.py

# 自定义主机、端口和次数
python ping_client.py 127.0.0.1 9999 20
```

**输出示例**：
```
正在 ping 127.0.0.1:9999
==================================================
序号 1: rtt=12.34 ms
序号 2: 请求超时
序号 3: rtt=8.92 ms
...

===== ping 统计 =====
已发送：20，已接收：16
丢包率：20.0%
最小 RTT: 5.23 ms
最大 RTT: 95.67 ms
平均 RTT: 25.43 ms
```

### 4.5 优化思路

进一步改进：
- 显示标准差等更详细的统计
- 实时计算（不只是最后统计）
- 支持 IPv6
- 添加 TTL（生存时间）
- 模拟 ICMP 的 ECHO REQUEST/REPLY 格式

### 📝 本关小结

你已经会写一个简单但功能完整的 ping 程序了！

---

## 关卡 5：模拟简单聊天室（中等）

### 📋 学习目标

- 综合应用 UDP 的特性
- 实现多用户管理
- 实现消息广播
- 设计简单的应用层协议（JSON）

### 5.1 聊天室设计

**功能需求**：
- 用户加入聊天室（带昵称）
- 发送消息，所有人都能看到
- 查看在线用户
- 用户离开聊天室

### 5.2 协议设计

为了简化，我们使用 **JSON 格式**来传输消息（带结构化字段，易扩展）：

```json
{
    "type": "join",
    "name": "Alice",
    "content": "..."
}
```

**消息类型**：

| `type` | 含义 | 字段 |
|---|---|---|
| `join` | 加入聊天室 | name |
| `leave` | 离开聊天室 | （无） |
| `message` | 发送消息 | content |
| `system` | 系统消息（服务端→客户端） | content |
| `list` | 请求/返回在线列表 | users |

### 5.3 服务端实现

**`chat_server.py`：**

```python
import socket
import json
from datetime import datetime

# 创建 UDP 套接字
server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(('127.0.0.1', 9999))
print("🚀 聊天室服务器已启动...")

# 用户管理：地址 → 昵称
users = {}

def broadcast(message, exclude=None):
    """广播消息给所有用户（可排除某个地址）"""
    data = json.dumps(message).encode('utf-8')
    for addr in users:
        if addr != exclude:
            server_socket.sendto(data, addr)

def send_to(addr, message):
    """发送消息给指定用户"""
    data = json.dumps(message).encode('utf-8')
    server_socket.sendto(data, addr)

while True:
    try:
        data, addr = server_socket.recvfrom(4096)
        msg = json.loads(data.decode('utf-8'))
        msg_type = msg.get('type')

        # ===== 处理加入 =====
        if msg_type == 'join':
            name = msg['name']
            users[addr] = name
            print(f"[{datetime.now()}] ✅ {name} 加入了聊天室")

            # 通知所有人
            broadcast({
                'type': 'system',
                'content': f"{name} 加入了聊天室"
            })

            # 欢迎新用户
            send_to(addr, {
                'type': 'system',
                'content': f"欢迎！当前在线 {len(users)} 人"
            })

        # ===== 处理离开 =====
        elif msg_type == 'leave':
            name = users.get(addr, '未知用户')
            if addr in users:
                del users[addr]
            print(f"[{datetime.now()}] ❌ {name} 离开了聊天室")

            broadcast({
                'type': 'system',
                'content': f"{name} 离开了聊天室"
            })

        # ===== 处理普通消息 =====
        elif msg_type == 'message':
            name = users.get(addr)
            if name:
                content = msg['content']
                print(f"[{datetime.now()}] {name}: {content}")

                # 广播给除发送者外的所有人
                broadcast({
                    'type': 'message',
                    'name': name,
                    'content': content
                }, exclude=addr)

        # ===== 处理在线列表请求 =====
        elif msg_type == 'list':
            send_to(addr, {
                'type': 'list',
                'users': list(users.values())
            })

    except KeyboardInterrupt:
        print("\n🛑 服务器关闭")
        break
    except Exception as e:
        print(f"❌ 错误：{e}")

server_socket.close()
```

### 5.4 客户端实现

**`chat_client.py`：**

```python
import socket
import json
import threading

# 创建 UDP 套接字
client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_addr = ('127.0.0.1', 9999)

# 控制是否继续运行
running = True

def receive_messages():
    """接收消息的线程"""
    global running
    while running:
        try:
            data, addr = client_socket.recvfrom(4096)
            msg = json.loads(data.decode('utf-8'))
            msg_type = msg.get('type')

            if msg_type == 'system':
                print(f"\n📢 [系统] {msg['content']}")
            elif msg_type == 'message':
                print(f"\n💬 [{msg['name']}] {msg['content']}")
            elif msg_type == 'list':
                print("\n👥 [在线用户]")
                for name in msg['users']:
                    print(f"  - {name}")

            # 重新打印输入提示符
            print("> ", end='', flush=True)
        except Exception as e:
            if running:
                print(f"接收错误：{e}")
            break

def send_message(msg):
    """发送消息"""
    data = json.dumps(msg).encode('utf-8')
    client_socket.sendto(data, server_addr)

def main():
    global running

    # 输入昵称
    name = input("请输入你的昵称：").strip()
    if not name:
        print("昵称不能为空")
        return

    # 加入聊天室
    send_message({'type': 'join', 'name': name})

    # 启动接收消息的线程
    recv_thread = threading.Thread(target=receive_messages)
    recv_thread.daemon = True
    recv_thread.start()

    # 显示帮助
    print("\n========= 欢迎来到聊天室 =========")
    print("💬 输入消息聊天")
    print("📋 输入 /list 查看在线用户")
    print("🚪 输入 /quit 退出")
    print("==================================\n")

    try:
        while True:
            content = input("> ").strip()
            if not content:
                continue

            if content == '/quit':
                send_message({'type': 'leave'})
                break
            elif content == '/list':
                send_message({'type': 'list'})
            else:
                send_message({
                    'type': 'message',
                    'content': content
                })
    except KeyboardInterrupt:
        send_message({'type': 'leave'})
    finally:
        running = False
        client_socket.close()
        print("\n👋 已退出聊天室")

if __name__ == "__main__":
    main()
```

### 5.5 测试聊天室

1. 启动服务器：`python chat_server.py`
2. 启动多个客户端（在不同终端）：`python chat_client.py`
3. 每个客户端输入不同昵称
4. 测试发送消息、`/list` 查看用户、`/quit` 退出

### 5.6 🚀 进一步改进的方向

实际聊天室还可以加入：

| 功能 | 实现思路 |
|---|---|
| 私聊 | 在 type 中加入 `private`，指定接收者 |
| 多房间 | 维护多个用户组 |
| 心跳检测 | 客户端定期发心跳，服务器记录最后活跃时间 |
| 历史消息 | 服务器保存最近 N 条消息，新用户加入时推送 |
| 持久化 | 用户、消息保存到文件或数据库 |

### 5.7 心跳检测扩展思路

```python
# 在服务器中维护每个用户的最后活跃时间
import time

last_active = {}  # addr -> 最后活跃时间

# 每次收到消息时更新
last_active[addr] = time.time()

# 启动一个线程定期检查
def check_timeout():
    while True:
        time.sleep(10)
        now = time.time()
        for addr in list(last_active.keys()):
            if now - last_active[addr] > 30:
                # 超过 30 秒无活动，视为掉线
                name = users.get(addr, '未知')
                print(f"⏰ {name} 超时掉线")
                # 清理用户
                ...
```

### 📝 本关小结

🎉 **恭喜你完成了本套教程的所有关卡！**

你已经掌握了：
- ✓ TCP/UDP 的核心概念与区别
- ✓ Socket 编程的标准流程
- ✓ 粘包问题与协议设计
- ✓ 多线程并发处理
- ✓ 数据加密传输
- ✓ 多人聊天室的完整实现

---

# 🎓 总结：你的下一步

完成这套教程后，你已经具备了 Python 网络编程的扎实基础。接下来可以学习：

| 方向 | 学习内容 |
|---|---|
| 🚀 异步编程 | `asyncio` 模块，更高效的并发 |
| 🌐 HTTP 编程 | `requests`、`Flask`、`FastAPI` |
| 🔌 WebSocket | 实时双向通信 |
| 📦 分布式系统 | RPC、消息队列（Kafka、RabbitMQ） |
| 🛡️ 网络安全 | TLS/SSL、HTTPS 实现 |
| 📊 高性能 IO | `select`、`epoll`、IO 多路复用 |

## 学习建议

1. **多动手**：每个关卡的代码都亲自运行一遍，并尝试修改
2. **多调试**：使用 `print` 或调试器观察数据流向
3. **多实践**：把每个关卡的小项目扩展成更完整的应用
4. **多查阅**：遇到问题查看官方文档：https://docs.python.org/3/library/socket.html

# Python 网络编程详细教程（续）

> 上一部分我们学习了 TCP 和 UDP 的基础。本部分将进入**进阶**与**工程化**领域，学完后你将能开发**生产级**的网络应用。

---

# 🎯 课程 3：网络编程进阶

## 关卡 1：多线程 Socket（中等）

### 📋 学习目标

- 深入理解多线程在网络编程中的作用
- 学会使用线程池（`ThreadPoolExecutor`）
- 了解 Python 的 GIL 对网络编程的影响
- 学会用锁解决线程安全问题

### 1.1 回顾：为什么需要多线程？

我们在课程 1 第 4 关已经接触过多线程。回忆一下：

**单线程服务器的问题**：
```python
client_socket, addr = server_socket.accept()  # 接受一个连接
while True:
    data = client_socket.recv(1024)  # 阻塞在这里
    # 其他客户端无法连接！
```

**多线程的解决方案**：每个客户端一个线程。

### 1.2 多线程的问题

虽然能解决并发问题，但简单的多线程也有缺点：

| 问题 | 说明 |
|---|---|
| 🐢 **资源消耗** | 每个线程占用约 1MB 内存，1000 个连接就是 1GB |
| 🔀 **频繁切换** | 线程过多导致上下文切换开销大 |
| 🔒 **GIL 限制** | Python 的全局解释器锁，同一时刻只有一个线程执行 Python 字节码 |
| ⚠️ **难以管理** | 大量线程创建/销毁，难以监控 |

> 💡 **关于 GIL**：网络编程中线程大部分时间在等待 IO（如 `recv`、`send`），GIL 在 IO 等待时会释放，所以多线程在网络编程中**依然有效**。

### 1.3 解决方案：线程池

**线程池**预先创建一批线程，让它们**重复使用**，避免频繁创建/销毁。

Python 提供了 `concurrent.futures.ThreadPoolExecutor`：

```python
from concurrent.futures import ThreadPoolExecutor

# 创建一个最大 10 个线程的池
executor = ThreadPoolExecutor(max_workers=10)

# 提交任务
executor.submit(some_function, arg1, arg2)

# 关闭池
executor.shutdown()
```

### 1.4 线程池版本的服务器

**服务端 `pool_server.py`：**

```python
import socket
from concurrent.futures import ThreadPoolExecutor
import threading

# 创建线程池：最多 10 个工作线程
executor = ThreadPoolExecutor(max_workers=10)

# 全局计数（注意线程安全）
active_clients = 0
lock = threading.Lock()  # 保护 active_clients

def handle_client(client_socket, addr):
    global active_clients

    # 加锁修改计数
    with lock:
        active_clients += 1
        current = active_clients
    print(f"[新连接] {addr}，当前活跃：{current}")

    try:
        while True:
            data = client_socket.recv(1024)
            if not data:
                break
            msg = data.decode('utf-8', errors='ignore')
            print(f"[{addr}] {msg}")

            # 回复
            response = f"已收到（来自线程 {threading.current_thread().name}）：{msg}"
            client_socket.send(response.encode('utf-8'))

            if msg.lower().strip() == 'bye':
                break
    except ConnectionResetError:
        print(f"[{addr}] 连接被重置")
    except Exception as e:
        print(f"[错误] {addr}: {e}")
    finally:
        client_socket.close()
        with lock:
            active_clients -= 1
        print(f"[断开] {addr}")

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('127.0.0.1', 8888))
    server_socket.listen(100)  # 允许更多排队
    print("[启动] 线程池服务器（最大 10 个并发）...")

    try:
        while True:
            client_socket, addr = server_socket.accept()
            # 把任务提交到线程池，自动复用线程
            executor.submit(handle_client, client_socket, addr)
    except KeyboardInterrupt:
        print("\n[关闭] 等待所有任务完成...")
        executor.shutdown(wait=True)
        server_socket.close()

if __name__ == "__main__":
    main()
```

### 1.5 压力测试

写个客户端模拟多个并发连接：

**`stress_test.py`：**

```python
import socket
import threading
import time

def client_task(i):
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect(('127.0.0.1', 8888))

        # 发送 3 条消息
        for j in range(3):
            msg = f"客户端 {i} 的第 {j+1} 条消息"
            sock.send(msg.encode('utf-8'))
            response = sock.recv(1024)
            print(f"[客户端 {i}] {response.decode('utf-8')}")
            time.sleep(0.5)

        sock.send(b'bye')
        sock.close()
    except Exception as e:
        print(f"[客户端 {i}] 错误：{e}")

# 同时启动 20 个客户端（超过线程池大小）
threads = []
for i in range(20):
    t = threading.Thread(target=client_task, args=(i,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()

print("✅ 所有客户端完成")
```

观察现象：虽然有 20 个客户端，但服务器只用 10 个线程处理（线程池），后续客户端会**排队等待**有空闲线程。

### 1.6 ⚠️ 线程安全：锁的使用

多线程同时修改共享数据会出问题：

```python
# 危险！多个线程同时执行可能丢失更新
counter = 0
def increment():
    global counter
    counter += 1  # 实际是 read → modify → write 三步
```

**正确做法**：

```python
lock = threading.Lock()
counter = 0

def increment():
    global counter
    with lock:  # 进入临界区
        counter += 1  # 同一时刻只有一个线程执行
```

**常见的需要加锁场景**：
- 共享字典/列表的修改
- 全局计数器
- 文件写入
- 数据库连接

### 1.7 📊 ThreadPoolExecutor 的高级用法

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

# 1. 用 with 语法自动管理
with ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(task, i) for i in range(10)]

    # 按完成顺序处理结果
    for future in as_completed(futures):
        result = future.result()
        print(result)

# 2. map 方法：类似 map() 函数
with ThreadPoolExecutor() as executor:
    results = executor.map(task, range(10))
    for r in results:
        print(r)
```

### 📝 本关小结

- ✓ 使用 `ThreadPoolExecutor` 管理线程
- ✓ 线程安全：用 `Lock` 保护共享数据
- ✓ 理解 GIL 对网络编程的影响有限

---

## 关卡 2：非阻塞 Socket（中等）

### 📋 学习目标

- 理解阻塞与非阻塞的区别
- 学会设置 socket 为非阻塞模式
- 学会处理 `BlockingIOError` 异常
- 理解非阻塞编程的"轮询"模式

### 2.1 什么是阻塞？

之前我们写的代码都是**阻塞**的：

```python
data = sock.recv(1024)  # 程序在这里"卡住"，等待数据到达
```

`recv` 没有数据时，程序会**停止执行**，直到数据来了才继续。

**问题**：单线程下，一个 `recv` 阻塞会让整个程序无法做其他事。

### 2.2 非阻塞：立即返回

设置 socket 为**非阻塞模式**后：

```python
sock.setblocking(False)
```

- `recv` 没有数据时，**立即抛出异常**（`BlockingIOError`），不再等待
- `accept` 没有新连接时，**立即抛出异常**
- `send` 缓冲区满时，**立即抛出异常**

这样程序可以在不阻塞的情况下检查 IO 状态，做其他事情。

### 2.3 阻塞 vs 非阻塞 对比

| 模式 | 行为 |
|---|---|
| **阻塞** | 等待操作完成才返回 |
| **非阻塞** | 立即返回，没准备好就抛异常 |

```python
# 阻塞模式（默认）
data = sock.recv(1024)  # 卡住等待

# 非阻塞模式
try:
    data = sock.recv(1024)  # 立即返回
except BlockingIOError:
    pass  # 没有数据，跳过去做别的
```

### 2.4 非阻塞服务器实现

**服务端 `nonblock_server.py`：**

```python
import socket
import time

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind(('127.0.0.1', 8888))
server_socket.listen(100)

# 关键：设置为非阻塞
server_socket.setblocking(False)

print("[启动] 非阻塞服务器...")

# 存储所有客户端连接
client_sockets = []

try:
    while True:
        # 1. 尝试接受新连接（非阻塞）
        try:
            client_socket, addr = server_socket.accept()
            client_socket.setblocking(False)  # 客户端 socket 也要设为非阻塞
            client_sockets.append((client_socket, addr))
            print(f"[新连接] {addr}，当前 {len(client_sockets)} 个客户端")
        except BlockingIOError:
            # 没有新连接，正常
            pass

        # 2. 遍历所有客户端，尝试接收数据
        for client_socket, addr in client_sockets[:]:  # 用副本避免迭代时删除
            try:
                data = client_socket.recv(1024)
                if not data:
                    # 客户端断开
                    print(f"[断开] {addr}")
                    client_sockets.remove((client_socket, addr))
                    client_socket.close()
                    continue

                msg = data.decode('utf-8', errors='ignore')
                print(f"[{addr}] {msg}")

                # 回复
                response = f"已收到：{msg}".encode('utf-8')
                try:
                    client_socket.send(response)
                except BlockingIOError:
                    print(f"[发送缓冲区满] {addr}")
            except BlockingIOError:
                # 这个客户端没数据，跳过
                continue
            except ConnectionResetError:
                print(f"[重置] {addr}")
                client_sockets.remove((client_socket, addr))
                client_socket.close()

        # 3. 短暂休眠，避免 CPU 100% 占用
        time.sleep(0.01)

except KeyboardInterrupt:
    print("\n[关闭] 服务器")
    for client_socket, _ in client_sockets:
        client_socket.close()
    server_socket.close()
```

### 2.5 ⚠️ 非阻塞的两大坑

#### 🔴 坑 1：CPU 占用 100%

```python
while True:
    try:
        data = sock.recv(1024)
    except BlockingIOError:
        pass  # 没数据
    # 这里直接进入下一次循环！CPU 飞起！
```

**解决**：加 `time.sleep(0.01)` 或使用下一关的 `select`。

#### 🔴 坑 2：send 也可能失败

```python
sock.send(big_data)  # 阻塞模式也可能只发一部分；非阻塞模式更常见
# 要完整发送，需要检查返回值循环发送，或在阻塞模式下使用 sendall()
```

**正确的非阻塞 send**：

```python
def send_all_nonblock(sock, data):
    total_sent = 0
    while total_sent < len(data):
        try:
            sent = sock.send(data[total_sent:])
            total_sent += sent
        except BlockingIOError:
            time.sleep(0.001)  # 缓冲区满，稍等再试
```

### 2.6 客户端用阻塞即可

客户端通常无需非阻塞：

```python
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('127.0.0.1', 8888))

for i in range(5):
    sock.send(f"消息 {i}".encode())
    print(sock.recv(1024).decode())

sock.close()
```

### 2.7 非阻塞模式的真正威力

单纯轮询非常低效。非阻塞 socket **真正的用武之地**是配合 **IO 多路复用**（下一关）。

操作系统提供了高效的"批量监听"机制：
- `select`（最简单，跨平台）
- `poll`（Linux 改进版）
- `epoll`（Linux 高性能版）
- `kqueue`（macOS/BSD）

> 🎯 **核心思想**：让操作系统告诉你哪些 socket 准备好了，避免无效的轮询。

### 📝 本关小结

- ✓ 理解阻塞 vs 非阻塞
- ✓ 学会 `setblocking(False)` 和处理 `BlockingIOError`
- ✓ 了解非阻塞编程的"轮询"模式及其问题
- ✓ 期待下一关用 `select` 高效解决问题

---

## 关卡 3：select / IO 多路复用（困难）

### 📋 学习目标

- 理解 IO 多路复用的概念
- 学会使用 `select` 模块
- 学会使用更现代的 `selectors` 模块
- 实现高性能的并发服务器

### 3.1 什么是 IO 多路复用？

**核心思想**：用**一个线程**同时监听**多个 socket**，哪个有数据就处理哪个。

**类比**：
- ❌ 多线程：一个保安看一个门，10 个门就要 10 个保安
- ✅ IO 多路复用：一个保安看监控屏幕，10 个门只要 1 个保安

**优势**：
- 内存占用低（不需要每个连接一个线程）
- 没有线程切换开销
- 能支持上万并发连接

### 3.2 select 的工作原理

```
你（程序）：操作系统大佬，请帮我监控这一堆 socket
            告诉我哪些有数据可读、哪些可写
操作系统：好的，等等...... 这几个有数据了，这几个可写
你：好嘞，我去处理这几个，其他的继续监控
```

`select` 函数签名：

```python
import select

readable, writable, exceptional = select.select(rlist, wlist, xlist, timeout)
```

| 参数 | 含义 |
|---|---|
| `rlist` | 监控**可读**的 socket 列表 |
| `wlist` | 监控**可写**的 socket 列表 |
| `xlist` | 监控**异常**的 socket 列表 |
| `timeout` | 超时秒数（None 表示永久等待） |

返回三个列表：**已准备好**的 socket。

### 3.3 select 版本服务器

**`select_server.py`：**

```python
import socket
import select

# 创建服务端 socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind(('127.0.0.1', 8888))
server_socket.listen(100)
server_socket.setblocking(False)  # 非阻塞

print("[启动] select 服务器...")

# 监控列表：初始只有服务端 socket
# 服务端 socket 可读 = 有新连接
sockets_to_read = [server_socket]
# 用字典保存 socket → 地址 的映射
addresses = {}

try:
    while True:
        # 调用 select，阻塞等待任意 socket 就绪（最多 1 秒）
        readable, _, exceptional = select.select(sockets_to_read, [], sockets_to_read, 1.0)

        # 处理可读的 socket
        for sock in readable:
            if sock is server_socket:
                # 服务端 socket 可读 = 有新连接
                client_socket, addr = server_socket.accept()
                client_socket.setblocking(False)
                sockets_to_read.append(client_socket)
                addresses[client_socket] = addr
                print(f"[新连接] {addr}，总连接数：{len(sockets_to_read) - 1}")
            else:
                # 客户端 socket 可读 = 有数据
                addr = addresses[sock]
                try:
                    data = sock.recv(1024)
                    if data:
                        msg = data.decode('utf-8', errors='ignore').strip()
                        print(f"[{addr}] {msg}")
                        # 回复
                        sock.send(f"echo: {msg}\n".encode('utf-8'))
                    else:
                        # 客户端关闭
                        print(f"[断开] {addr}")
                        sockets_to_read.remove(sock)
                        del addresses[sock]
                        sock.close()
                except ConnectionResetError:
                    print(f"[重置] {addr}")
                    sockets_to_read.remove(sock)
                    del addresses[sock]
                    sock.close()

        # 处理异常的 socket
        for sock in exceptional:
            addr = addresses.get(sock, "?")
            print(f"[异常] {addr}")
            sockets_to_read.remove(sock)
            del addresses[sock]
            sock.close()

except KeyboardInterrupt:
    print("\n[关闭]")
    for sock in sockets_to_read:
        sock.close()
```

**测试**：用之前的 `stress_test.py`，或者用 `telnet`：

```bash
telnet 127.0.0.1 8888
```

启动多个 telnet，一个服务器进程就能服务全部！

### 3.4 select 的局限

| 问题 | 说明 |
|---|---|
| ⚠️ **数量限制** | 在 Linux 默认最多监控 1024 个 |
| 🐢 **效率不高** | 每次都要遍历所有 socket，O(n) 复杂度 |
| 🔄 **每次重新传入** | 调用时要传完整列表 |

Linux 提供了 `epoll`，效率是 O(1)，但接口较底层。

### 3.5 selectors：更现代的 API

Python 提供了 `selectors` 模块，自动选择系统上最优的方式（Linux 用 epoll，macOS 用 kqueue）：

```python
import selectors

# 自动选择最佳实现
sel = selectors.DefaultSelector()

# 注册：监控某个 socket 的某种事件
sel.register(sock, selectors.EVENT_READ, data=callback)

# 等待事件
events = sel.select(timeout)
for key, mask in events:
    # 处理事件
    pass

# 取消注册
sel.unregister(sock)
```

### 3.6 selectors 版本服务器（推荐写法）

**`selectors_server.py`：**

```python
import socket
import selectors
import types  # 用于创建简单的命名空间

sel = selectors.DefaultSelector()

def accept_client(server_socket):
    """处理新连接"""
    client_socket, addr = server_socket.accept()
    print(f"[新连接] {addr}")
    client_socket.setblocking(False)

    # 用 SimpleNamespace 存储这个连接的数据
    data = types.SimpleNamespace(
        addr=addr,
        inbox=b'',   # 已接收/待解析的数据
        outbox=b''   # 待发送的数据
    )

    # 初始只监听读；有待发送数据时再临时监听写，避免可写事件持续触发
    sel.register(client_socket, selectors.EVENT_READ, data=data)

def service_client(key, mask):
    """处理客户端 IO"""
    sock = key.fileobj
    data = key.data

    # 可读
    if mask & selectors.EVENT_READ:
        try:
            recv_data = sock.recv(1024)
            if recv_data:
                msg = recv_data.decode('utf-8', errors='ignore').strip()
                print(f"[{data.addr}] {msg}")
                # 准备好回复（放入 outbox）
                data.outbox += f"echo: {msg}\n".encode('utf-8')
                sel.modify(sock, selectors.EVENT_READ | selectors.EVENT_WRITE, data=data)
            else:
                # 客户端关闭
                print(f"[断开] {data.addr}")
                sel.unregister(sock)
                sock.close()
                return
        except ConnectionResetError:
            print(f"[重置] {data.addr}")
            sel.unregister(sock)
            sock.close()
            return

    # 可写
    if mask & selectors.EVENT_WRITE:
        if data.outbox:
            # 有数据要发
            try:
                sent = sock.send(data.outbox)
                data.outbox = data.outbox[sent:]  # 剩下没发完的
                if not data.outbox:
                    sel.modify(sock, selectors.EVENT_READ, data=data)
            except BlockingIOError:
                pass

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('127.0.0.1', 8888))
    server_socket.listen(100)
    server_socket.setblocking(False)

    # 注册服务端 socket，只关心读事件（新连接）
    # data=None 用于区分服务端和客户端
    sel.register(server_socket, selectors.EVENT_READ, data=None)

    print("[启动] selectors 服务器...")

    try:
        while True:
            # 等待事件，timeout=None 表示永久等待
            events = sel.select(timeout=None)
            for key, mask in events:
                if key.data is None:
                    # 服务端 socket 有事件 = 新连接
                    accept_client(key.fileobj)
                else:
                    # 客户端 socket 有事件
                    service_client(key, mask)
    except KeyboardInterrupt:
        print("\n[关闭]")
    finally:
        sel.close()
        server_socket.close()

if __name__ == "__main__":
    main()
```

### 3.7 性能对比

| 方案 | 1000 并发表现 | 10000 并发表现 |
|---|---|---|
| 单线程阻塞 | ❌ 不可能 | ❌ 不可能 |
| 一连接一线程 | ⚠️ 慢、内存高 | ❌ 崩溃 |
| 线程池（100） | ✅ 良好 | ⚠️ 排队 |
| select | ✅ 良好 | ⚠️ 慢 |
| epoll/selectors | ✅ 优秀 | ✅ 优秀 |

**著名的 C10K 问题**：如何在单机支持 10000 个并发连接？答案就是 IO 多路复用 + 非阻塞。

### 📝 本关小结

- ✓ 理解 IO 多路复用的核心思想
- ✓ 学会用 `select` 和 `selectors`
- ✓ 单线程也能服务大量并发连接

---

## 关卡 4：简单 HTTP 服务器（困难）

### 📋 学习目标

- 理解 HTTP 协议的基本格式
- 用 Socket 从零实现一个 HTTP 服务器
- 支持 GET 和 POST 方法
- 支持返回静态文件

### 4.1 HTTP 协议速览

HTTP 是基于 TCP 的应用层协议。客户端（浏览器）和服务器之间约定一种"格式"来通信。

**请求格式**：
```
GET /index.html HTTP/1.1\r\n           ← 请求行（方法 路径 协议版本）
Host: www.example.com\r\n              ← 请求头
User-Agent: Mozilla/5.0\r\n
Accept: text/html\r\n
\r\n                                   ← 空行（重要！表示头结束）
（这里是请求体，GET 一般无）
```

**响应格式**：
```
HTTP/1.1 200 OK\r\n                    ← 状态行（协议 状态码 状态描述）
Content-Type: text/html\r\n            ← 响应头
Content-Length: 123\r\n
\r\n                                   ← 空行
<html><body>Hello</body></html>        ← 响应体
```

**常见状态码**：

| 状态码 | 含义 |
|---|---|
| 200 | OK，成功 |
| 301/302 | 重定向 |
| 400 | Bad Request，请求格式错 |
| 404 | Not Found，资源不存在 |
| 500 | Internal Server Error，服务器错 |

### 4.2 最简单的 HTTP 服务器

**`simple_http.py`：**

```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind(('127.0.0.1', 8080))
server_socket.listen(5)
print("[启动] HTTP 服务器：http://127.0.0.1:8080")

while True:
    client_socket, addr = server_socket.accept()
    # 接收请求
    request = client_socket.recv(4096).decode('utf-8', errors='ignore')
    print(f"\n=========== 收到请求 ===========\n{request}")

    # 构造响应
    body = "<html><body><h1>Hello, World!</h1><p>这是我的第一个 HTTP 服务器</p></body></html>"
    response = (
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html; charset=utf-8\r\n"
        f"Content-Length: {len(body.encode('utf-8'))}\r\n"
        "Connection: close\r\n"
        "\r\n"
        f"{body}"
    )

    client_socket.send(response.encode('utf-8'))
    client_socket.close()
```

**测试**：
1. 运行服务器
2. 浏览器打开 `http://127.0.0.1:8080`
3. 看到 "Hello, World!" 🎉

### 4.3 解析请求

为了支持不同的请求，需要解析请求行：

```python
def parse_request(request_str):
    """解析 HTTP 请求，返回 (方法, 路径, 头部字典, 请求体)"""
    # 分割头和体（空行分隔）
    if '\r\n\r\n' in request_str:
        header_part, body = request_str.split('\r\n\r\n', 1)
    else:
        header_part = request_str
        body = ''

    lines = header_part.split('\r\n')

    # 第一行是请求行
    request_line = lines[0]
    parts = request_line.split(' ')
    if len(parts) < 3:
        return None, None, {}, ''
    method, path, version = parts[0], parts[1], parts[2]

    # 后续行是头部
    headers = {}
    for line in lines[1:]:
        if ':' in line:
            key, value = line.split(':', 1)
            headers[key.strip().lower()] = value.strip()

    return method, path, headers, body
```

### 4.4 进阶版：支持路由和静态文件

我们做一个能：
- 根据路径返回不同内容（路由）
- 返回静态文件（HTML、图片）
- 支持 POST 请求
- 返回 JSON

**目录结构**：
```
http_server/
├── server.py
└── static/
    ├── index.html
    ├── about.html
    └── logo.png
```

**`static/index.html`** 示例：
```html
<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>首页</title></head>
<body>
    <h1>欢迎来到我的网站</h1>
    <p><a href="/about">关于</a> | <a href="/api/time">API（JSON）</a></p>
    <img src="/static/logo.png" alt="logo">
    <form action="/api/echo" method="POST">
        <input name="message" placeholder="输入消息">
        <button>发送</button>
    </form>
</body>
</html>
```

**`server.py`：**

```python
import socket
import os
import json
import threading
import mimetypes
import html
from datetime import datetime
from urllib.parse import unquote, parse_qs

STATIC_DIR = 'static'

def parse_request(request_bytes):
    """解析 HTTP 请求"""
    try:
        request_str = request_bytes.decode('utf-8', errors='ignore')
    except:
        return None

    if '\r\n\r\n' in request_str:
        header_part, body = request_str.split('\r\n\r\n', 1)
    else:
        header_part, body = request_str, ''

    lines = header_part.split('\r\n')
    if not lines:
        return None

    parts = lines[0].split(' ')
    if len(parts) < 3:
        return None

    method, path, version = parts[0], parts[1], parts[2]

    # 解析查询参数：/page?key=value
    query = {}
    if '?' in path:
        path, query_str = path.split('?', 1)
        query = parse_qs(query_str)
        query = {k: v[0] for k, v in query.items()}  # 简化

    path = unquote(path)  # URL 解码

    # 解析头
    headers = {}
    for line in lines[1:]:
        if ':' in line:
            k, v = line.split(':', 1)
            headers[k.strip().lower()] = v.strip()

    return {
        'method': method,
        'path': path,
        'version': version,
        'headers': headers,
        'query': query,
        'body': body
    }

def make_response(status_code, body, content_type='text/html; charset=utf-8', extra_headers=None):
    """构造 HTTP 响应"""
    status_text = {
        200: 'OK', 301: 'Moved Permanently', 302: 'Found',
        400: 'Bad Request', 404: 'Not Found', 405: 'Method Not Allowed',
        500: 'Internal Server Error'
    }.get(status_code, 'Unknown')

    if isinstance(body, str):
        body = body.encode('utf-8')

    response_headers = [
        f"HTTP/1.1 {status_code} {status_text}",
        f"Content-Type: {content_type}",
        f"Content-Length: {len(body)}",
        f"Date: {datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')}",
        "Server: MyPyServer/1.0",
        "Connection: close",
    ]
    if extra_headers:
        for k, v in extra_headers.items():
            response_headers.append(f"{k}: {v}")

    header_bytes = ('\r\n'.join(response_headers) + '\r\n\r\n').encode('utf-8')
    return header_bytes + body

def serve_static(path):
    """提供静态文件"""
    # 安全：防止 ../ 路径攻击
    safe_path = path.lstrip('/')
    base_dir = os.path.abspath(STATIC_DIR)
    file_path = os.path.abspath(os.path.join(base_dir, safe_path.replace('static/', '', 1)))

    if os.path.commonpath([base_dir, file_path]) != base_dir or not os.path.isfile(file_path):
        return make_response(404, "<h1>404 Not Found</h1>")

    # 推断 MIME 类型
    content_type, _ = mimetypes.guess_type(file_path)
    if not content_type:
        content_type = 'application/octet-stream'

    with open(file_path, 'rb') as f:
        body = f.read()

    return make_response(200, body, content_type=content_type)

def handle_request(req):
    """根据请求分发处理（路由）"""
    method = req['method']
    path = req['path']

    # ========== 路由 ==========
    if path == '/':
        return serve_static('index.html')

    elif path == '/about':
        body = "<h1>关于</h1><p>这是用 socket 手写的 HTTP 服务器</p><a href='/'>返回</a>"
        return make_response(200, body)

    elif path == '/api/time':
        # 返回 JSON
        data = {
            'time': datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
            'message': 'Hello from API'
        }
        return make_response(200, json.dumps(data, ensure_ascii=False),
                             content_type='application/json; charset=utf-8')

    elif path == '/api/echo' and method == 'POST':
        # 解析表单数据
        form = parse_qs(req['body'])
        message = html.escape(form.get('message', [''])[0])
        body = f"<h1>收到消息</h1><p>{message}</p><a href='/'>返回</a>"
        return make_response(200, body)

    elif path.startswith('/static/'):
        return serve_static(path)

    else:
        return make_response(404, "<h1>404 Not Found</h1><a href='/'>返回首页</a>")

def recv_http_request(sock):
    """读取完整 HTTP 头，并按 Content-Length 继续读取请求体"""
    data = b''
    while b'\r\n\r\n' not in data:
        chunk = sock.recv(4096)
        if not chunk:
            return data
        data += chunk

    header, body = data.split(b'\r\n\r\n', 1)
    headers = {}
    for line in header.decode('iso-8859-1', errors='ignore').split('\r\n')[1:]:
        if ':' in line:
            k, v = line.split(':', 1)
            headers[k.strip().lower()] = v.strip()
    content_length = int(headers.get('content-length', 0))
    while len(body) < content_length:
        chunk = sock.recv(min(4096, content_length - len(body)))
        if not chunk:
            break
        body += chunk
    return header + b'\r\n\r\n' + body

def handle_client(client_socket, addr):
    """处理单个客户端连接"""
    try:
        request_bytes = recv_http_request(client_socket)
        if not request_bytes:
            return

        req = parse_request(request_bytes)
        if req is None:
            response = make_response(400, "<h1>400 Bad Request</h1>")
        else:
            print(f"[{datetime.now()}] {addr[0]} - {req['method']} {req['path']}")
            try:
                response = handle_request(req)
            except Exception as e:
                print(f"[错误] {e}")
                response = make_response(500, f"<h1>500 Internal Server Error</h1><pre>{e}</pre>")

        client_socket.sendall(response)
    except Exception as e:
        print(f"[处理错误] {e}")
    finally:
        client_socket.close()

def main():
    # 确保 static 目录存在
    os.makedirs(STATIC_DIR, exist_ok=True)

    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('127.0.0.1', 8080))
    server_socket.listen(10)
    print("🚀 HTTP 服务器启动：http://127.0.0.1:8080")

    try:
        while True:
            client_socket, addr = server_socket.accept()
            # 多线程处理
            t = threading.Thread(target=handle_client, args=(client_socket, addr))
            t.daemon = True
            t.start()
    except KeyboardInterrupt:
        print("\n[关闭]")
        server_socket.close()

if __name__ == "__main__":
    main()
```

### 4.5 测试

1. 创建 `static/index.html`（内容见上面）
2. 启动服务器：`python server.py`
3. 浏览器访问：
   - `http://127.0.0.1:8080/` — 首页
   - `http://127.0.0.1:8080/about` — 关于页
   - `http://127.0.0.1:8080/api/time` — JSON API
   - 在首页表单输入文字提交 — 测试 POST

### 4.6 🎓 进一步思考

我们手写的 HTTP 服务器虽然能用，但还很简陋。生产中应该使用：

| 工具 | 用途 |
|---|---|
| `http.server` | Python 内置的简单服务器 |
| `Flask` | 轻量级 Web 框架 |
| `FastAPI` | 现代异步框架 |
| `Django` | 全功能 Web 框架 |
| `nginx` | 反向代理 / 静态文件服务 |

但理解底层原理后，再用这些框架会更加得心应手！

### 📝 本关小结

- ✓ 理解 HTTP 协议的请求/响应格式
- ✓ 从 socket 层手动实现 HTTP 服务器
- ✓ 支持路由、静态文件、JSON API、POST 表单

---

## 关卡 5：文件传输系统（困难）

### 📋 学习目标

- 实现完整的文件传输系统
- 支持文件列表、上传、下载
- 实现断点续传
- 添加文件完整性校验（MD5）

### 5.1 系统设计

**功能列表**：

| 命令 | 功能 |
|---|---|
| `LIST` | 列出服务器上的文件 |
| `UPLOAD <filename>` | 上传文件 |
| `DOWNLOAD <filename>` | 下载文件 |
| `DOWNLOAD <filename> [offset]` | 带偏移量的断点续传下载 |
| `DELETE <filename>` | 删除文件 |
| `QUIT` | 退出 |

**协议设计**：

为了简单，使用"长度前缀 + JSON 命令"格式：

```
┌─────────────┬──────────────────┐
│ 4 字节长度  │  JSON 命令体     │
└─────────────┴──────────────────┘
```

文件传输部分用原始字节，按长度精确接收。

### 5.2 协议工具函数

**`protocol.py`：**

```python
import struct
import json

MAX_FRAME_SIZE = 1024 * 1024  # 限制 JSON 命令最大 1MB，避免恶意长度耗尽内存

def send_msg(sock, msg_dict):
    """发送 JSON 消息（带 4 字节长度前缀）"""
    data = json.dumps(msg_dict, ensure_ascii=False).encode('utf-8')
    sock.sendall(struct.pack('!I', len(data)) + data)

def recv_msg(sock):
    """接收 JSON 消息"""
    raw_len = recv_n(sock, 4)
    if not raw_len:
        return None
    length = struct.unpack('!I', raw_len)[0]
    if length > MAX_FRAME_SIZE:
        return None
    data = recv_n(sock, length)
    if not data:
        return None
    return json.loads(data.decode('utf-8'))

def recv_n(sock, n):
    """精确接收 n 字节"""
    data = b''
    while len(data) < n:
        chunk = sock.recv(min(n - len(data), 65536))
        if not chunk:
            return None
        data += chunk
    return data

def send_file_data(sock, filepath, offset=0, chunk_size=65536):
    """发送文件内容（可指定起始偏移）"""
    with open(filepath, 'rb') as f:
        f.seek(offset)
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            sock.sendall(chunk)

def recv_file_data(sock, filepath, total_size, offset=0, chunk_size=65536):
    """接收文件数据并写入"""
    mode = 'ab' if offset > 0 else 'wb'
    with open(filepath, mode) as f:
        received = 0
        while received < total_size:
            to_recv = min(chunk_size, total_size - received)
            chunk = sock.recv(to_recv)
            if not chunk:
                return False
            f.write(chunk)
            received += len(chunk)
            # 显示进度
            pct = (received / total_size) * 100
            print(f"\r  进度：{pct:.1f}% ({received}/{total_size})", end='', flush=True)
        print()
    return True

def calc_md5(filepath, chunk_size=65536):
    """计算文件 MD5"""
    import hashlib
    md5 = hashlib.md5()
    with open(filepath, 'rb') as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            md5.update(chunk)
    return md5.hexdigest()
```

### 5.3 服务端实现

**`file_server.py`：**

```python
import socket
import os
import threading
from protocol import send_msg, recv_msg, send_file_data, recv_file_data, calc_md5

SERVER_DIR = 'server_files'
os.makedirs(SERVER_DIR, exist_ok=True)

def safe_join(base_dir, filename):
    """把文件名限制在 base_dir 内，防止 ../ 路径穿越"""
    base = os.path.abspath(base_dir)
    target = os.path.abspath(os.path.join(base, filename))
    if os.path.commonpath([base, target]) != base:
        raise ValueError("非法文件名")
    return target

def handle_client(client_socket, addr):
    print(f"[新连接] {addr}")

    try:
        while True:
            # 接收命令
            req = recv_msg(client_socket)
            if not req:
                break

            cmd = req.get('cmd')
            print(f"[{addr}] {cmd}")

            # ========== LIST ==========
            if cmd == 'LIST':
                files = []
                for f in os.listdir(SERVER_DIR):
                    full = os.path.join(SERVER_DIR, f)
                    if os.path.isfile(full):
                        files.append({
                            'name': f,
                            'size': os.path.getsize(full)
                        })
                send_msg(client_socket, {'status': 'ok', 'files': files})

            # ========== DOWNLOAD ==========
            elif cmd == 'DOWNLOAD':
                filename = req['filename']
                offset = req.get('offset', 0)  # 断点续传起点
                try:
                    filepath = safe_join(SERVER_DIR, filename)
                except ValueError:
                    send_msg(client_socket, {'status': 'error', 'message': '非法文件名'})
                    continue

                if not os.path.isfile(filepath):
                    send_msg(client_socket, {'status': 'error', 'message': '文件不存在'})
                    continue

                file_size = os.path.getsize(filepath)
                if offset > file_size:
                    send_msg(client_socket, {'status': 'error', 'message': 'offset 超过文件大小'})
                    continue

                md5 = calc_md5(filepath)
                send_msg(client_socket, {
                    'status': 'ok',
                    'size': file_size,
                    'offset': offset,
                    'md5': md5
                })

                # 发送文件内容
                send_file_data(client_socket, filepath, offset=offset)
                print(f"[发送完成] {filename}")

            # ========== UPLOAD ==========
            elif cmd == 'UPLOAD':
                filename = req['filename']
                size = req['size']
                md5_expected = req.get('md5')
                try:
                    filepath = safe_join(SERVER_DIR, filename)
                except ValueError:
                    send_msg(client_socket, {'status': 'error', 'message': '非法文件名'})
                    continue

                send_msg(client_socket, {'status': 'ready'})

                # 接收文件内容
                success = recv_file_data(client_socket, filepath, size)
                if success:
                    # MD5 校验
                    md5_actual = calc_md5(filepath)
                    if md5_expected and md5_actual != md5_expected:
                        send_msg(client_socket, {'status': 'error', 'message': 'MD5 不匹配'})
                        os.remove(filepath)
                    else:
                        send_msg(client_socket, {'status': 'ok', 'md5': md5_actual})
                        print(f"[接收完成] {filename}")
                else:
                    send_msg(client_socket, {'status': 'error', 'message': '传输中断'})

            # ========== DELETE ==========
            elif cmd == 'DELETE':
                filename = req['filename']
                try:
                    filepath = safe_join(SERVER_DIR, filename)
                except ValueError:
                    send_msg(client_socket, {'status': 'error', 'message': '非法文件名'})
                    continue
                if os.path.isfile(filepath):
                    os.remove(filepath)
                    send_msg(client_socket, {'status': 'ok'})
                else:
                    send_msg(client_socket, {'status': 'error', 'message': '文件不存在'})

            # ========== QUIT ==========
            elif cmd == 'QUIT':
                send_msg(client_socket, {'status': 'bye'})
                break

            else:
                send_msg(client_socket, {'status': 'error', 'message': '未知命令'})

    except Exception as e:
        print(f"[错误] {addr}: {e}")
    finally:
        client_socket.close()
        print(f"[断开] {addr}")

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('127.0.0.1', 9000))
    server_socket.listen(10)
    print(f"🚀 文件服务器启动：127.0.0.1:9000")
    print(f"📁 文件目录：{os.path.abspath(SERVER_DIR)}")

    try:
        while True:
            client_socket, addr = server_socket.accept()
            t = threading.Thread(target=handle_client, args=(client_socket, addr))
            t.daemon = True
            t.start()
    except KeyboardInterrupt:
        print("\n[关闭]")
        server_socket.close()

if __name__ == "__main__":
    main()
```

### 5.4 客户端实现

**`file_client.py`：**

```python
import socket
import os
from protocol import send_msg, recv_msg, send_file_data, recv_file_data, calc_md5

CLIENT_DIR = 'client_downloads'
os.makedirs(CLIENT_DIR, exist_ok=True)

def safe_join(base_dir, filename):
    base = os.path.abspath(base_dir)
    target = os.path.abspath(os.path.join(base, filename))
    if os.path.commonpath([base, target]) != base:
        raise ValueError("非法文件名")
    return target

class FileClient:
    def __init__(self, host='127.0.0.1', port=9000):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((host, port))
        print(f"✅ 已连接 {host}:{port}")

    def list_files(self):
        send_msg(self.sock, {'cmd': 'LIST'})
        resp = recv_msg(self.sock)
        if resp['status'] == 'ok':
            print("\n📁 服务器文件列表：")
            print("-" * 50)
            for f in resp['files']:
                size_kb = f['size'] / 1024
                print(f"  {f['name']:<30} {size_kb:>10.2f} KB")
            print("-" * 50)
        else:
            print(f"❌ {resp.get('message')}")

    def download(self, filename, resume=False):
        try:
            save_path = safe_join(CLIENT_DIR, filename)
        except ValueError:
            print("❌ 非法文件名")
            return

        # 断点续传：检查本地已下载部分
        offset = 0
        if resume and os.path.exists(save_path):
            offset = os.path.getsize(save_path)
            print(f"🔄 检测到本地已有 {offset} 字节，从此处继续")

        send_msg(self.sock, {'cmd': 'DOWNLOAD', 'filename': filename, 'offset': offset})
        resp = recv_msg(self.sock)
        if resp['status'] != 'ok':
            print(f"❌ {resp.get('message')}")
            return

        total_size = resp['size']
        remaining = total_size - offset
        print(f"📥 开始下载 {filename}（共 {total_size} 字节，剩余 {remaining}）")

        if recv_file_data(self.sock, save_path, remaining, offset=offset):
            # MD5 校验
            expected_md5 = resp.get('md5')
            actual_md5 = calc_md5(save_path)
            if expected_md5 == actual_md5:
                print(f"✅ 下载完成且 MD5 校验通过：{save_path}")
            else:
                print(f"⚠️ MD5 校验失败！可能文件已损坏")
        else:
            print("❌ 下载中断（可用 resume 续传）")

    def upload(self, filepath):
        if not os.path.isfile(filepath):
            print(f"❌ 文件不存在：{filepath}")
            return

        filename = os.path.basename(filepath)
        size = os.path.getsize(filepath)
        md5 = calc_md5(filepath)

        print(f"📤 准备上传 {filename}（{size} 字节）")
        send_msg(self.sock, {'cmd': 'UPLOAD', 'filename': filename, 'size': size, 'md5': md5})

        resp = recv_msg(self.sock)
        if resp['status'] != 'ready':
            print(f"❌ {resp.get('message')}")
            return

        # 发送文件
        send_file_data(self.sock, filepath)

        # 等待结果
        result = recv_msg(self.sock)
        if result['status'] == 'ok':
            print(f"✅ 上传成功，服务器 MD5：{result['md5']}")
        else:
            print(f"❌ {result.get('message')}")

    def delete(self, filename):
        send_msg(self.sock, {'cmd': 'DELETE', 'filename': filename})
        resp = recv_msg(self.sock)
        if resp['status'] == 'ok':
            print(f"✅ 已删除 {filename}")
        else:
            print(f"❌ {resp.get('message')}")

    def quit(self):
        send_msg(self.sock, {'cmd': 'QUIT'})
        try:
            recv_msg(self.sock)
        except:
            pass
        self.sock.close()
        print("👋 再见")

def main():
    client = FileClient()
    print("\n========== 文件传输系统 ==========")
    print("命令：list / download <file> / resume <file>")
    print("      upload <path> / delete <file> / quit")
    print("=" * 35)

    while True:
        try:
            cmd_line = input("\n>>> ").strip()
            if not cmd_line:
                continue

            parts = cmd_line.split(maxsplit=1)
            cmd = parts[0].lower()

            if cmd == 'list':
                client.list_files()
            elif cmd == 'download' and len(parts) > 1:
                client.download(parts[1])
            elif cmd == 'resume' and len(parts) > 1:
                client.download(parts[1], resume=True)
            elif cmd == 'upload' and len(parts) > 1:
                client.upload(parts[1])
            elif cmd == 'delete' and len(parts) > 1:
                client.delete(parts[1])
            elif cmd == 'quit':
                client.quit()
                break
            else:
                print("❌ 未知命令")
        except KeyboardInterrupt:
            print("\n^C")
            break
        except Exception as e:
            print(f"❌ 错误：{e}")
            break

if __name__ == "__main__":
    main()
```

### 5.5 测试断点续传

测试方法：
1. 启动服务器，往 `server_files` 放一个大文件（如 100MB）
2. 启动客户端，开始 `download big.zip`
3. **中途按 Ctrl+C 中断**
4. 重新启动客户端，用 `resume big.zip`
5. 观察从断点处继续下载

### 5.6 🎓 还可以改进

| 改进 | 思路 |
|---|---|
| 用户认证 | 加入登录系统 |
| 文件权限 | 区分公开/私有 |
| 上传断点续传 | 服务端记录已收字节 |
| 并发上传下载 | 同一连接多任务 |
| 加密传输 | TLS（下个课程） |
| 压缩传输 | gzip 压缩节省带宽 |

### 📝 本关小结

🎉 你已经完成了一个**完整的文件传输系统**！包含：
- ✓ 自定义协议（长度前缀 + JSON）
- ✓ 大文件传输
- ✓ 断点续传
- ✓ MD5 完整性校验
- ✓ 多客户端支持

---

# 🎯 课程 4：安全与工程化

## 关卡 1：数据加密传输（中等）

### 📋 学习目标

- 理解 TLS/SSL 的作用
- 学会用 Python 的 `ssl` 模块
- 生成自签名证书
- 实现 HTTPS 服务器

### 1.1 为什么需要 TLS？

我们在课程 1 第 5 关用了 AES 加密图片。但那种方式有几个问题：

- ❌ 密钥硬编码在代码里
- ❌ 没有身份认证（你知道对方是真服务器吗？）
- ❌ 没有完整性校验
- ❌ 自己写加密容易出错

**TLS（Transport Layer Security）** 是行业标准：
- ✅ 自动密钥交换（Diffie-Hellman）
- ✅ 服务器身份认证（数字证书）
- ✅ 数据加密（AES）
- ✅ 数据完整性（HMAC）

> 💡 **HTTPS** 就是 HTTP + TLS。`https://` 网站都用 TLS。

### 1.2 TLS 握手过程（简化版）

```
客户端 ──── "我支持 AES、SHA256...你呢？" ────> 服务器
       <──── "选 AES + SHA256，这是我的证书" ────
       （客户端验证证书）
       ──── "我用你的公钥加密一个随机数发给你" ────>
       （双方根据这个随机数生成会话密钥）
       <──── "握手完成，开始加密通信" ────>
```

### 1.3 生成自签名证书

正式的证书需要从 CA（证书颁发机构）购买。开发测试可以自己签发。

**用 OpenSSL 命令生成（需要安装 openssl）**：

```bash
# 生成私钥和证书（一行搞定，包含现代客户端校验需要的 SAN）
openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 365 -subj "/CN=localhost" -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

生成两个文件：
- `key.pem`：私钥（**绝不能泄露**）
- `cert.pem`：证书（包含公钥）

**用 Python 生成**（不需要 openssl）：

```python
# generate_cert.py
from cryptography import x509
from cryptography.x509.oid import NameOID
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa
from datetime import datetime, timedelta
import ipaddress

# 1. 生成 RSA 私钥
private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)

# 2. 构造证书
subject = issuer = x509.Name([
    x509.NameAttribute(NameOID.COUNTRY_NAME, "CN"),
    x509.NameAttribute(NameOID.COMMON_NAME, "localhost"),
])

cert = (x509.CertificateBuilder()
    .subject_name(subject)
    .issuer_name(issuer)
    .public_key(private_key.public_key())
    .serial_number(x509.random_serial_number())
    .not_valid_before(datetime.utcnow())
    .not_valid_after(datetime.utcnow() + timedelta(days=365))
    .add_extension(
        x509.SubjectAlternativeName([
            x509.DNSName("localhost"),
            x509.IPAddress(ipaddress.ip_address("127.0.0.1")),
        ]),
        critical=False
    )
    .sign(private_key, hashes.SHA256())
)

# 3. 保存
with open("key.pem", "wb") as f:
    f.write(private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=serialization.NoEncryption()
    ))

with open("cert.pem", "wb") as f:
    f.write(cert.public_bytes(serialization.Encoding.PEM))

print("✅ 证书已生成：key.pem, cert.pem")
```

### 1.4 TLS 服务器

Python 的 `ssl` 模块可以把普通 socket"包装"成 TLS socket。

**`tls_server.py`：**

```python
import socket
import ssl

# 1. 创建 SSL 上下文
context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
# 加载服务器的证书和私钥
context.load_cert_chain(certfile='cert.pem', keyfile='key.pem')

# 2. 创建普通 socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_socket.bind(('127.0.0.1', 8443))
server_socket.listen(5)
print("🔒 TLS 服务器启动：127.0.0.1:8443")

while True:
    raw_socket, addr = server_socket.accept()
    try:
        # 3. 用 SSL 包装
        client_socket = context.wrap_socket(raw_socket, server_side=True)
        print(f"[新 TLS 连接] {addr}")

        data = client_socket.recv(1024)
        print(f"[{addr}] {data.decode('utf-8')}")

        response = f"安全回复：{data.decode('utf-8')}"
        client_socket.send(response.encode('utf-8'))

        client_socket.close()
    except ssl.SSLError as e:
        print(f"[SSL 错误] {e}")
        raw_socket.close()
```

### 1.5 TLS 客户端

**`tls_client.py`：**

```python
import socket
import ssl

# 创建 SSL 上下文（客户端）
context = ssl.create_default_context()

# 自签名证书也应该加载并验证；生产环境通常加载系统/CA 信任链
context.load_verify_locations('cert.pem')
context.check_hostname = True
context.verify_mode = ssl.CERT_REQUIRED

# 创建并包装 socket
raw_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket = context.wrap_socket(raw_socket, server_hostname='localhost')

# 连接
client_socket.connect(('127.0.0.1', 8443))
print("🔒 已建立 TLS 连接")
print(f"使用协议：{client_socket.version()}")
print(f"加密算法：{client_socket.cipher()}")

# 发送数据（全程加密）
client_socket.send(b"Hello over TLS!")
response = client_socket.recv(1024)
print(f"收到：{response.decode('utf-8')}")

client_socket.close()
```

**运行**：
```bash
# 1. 生成证书
python generate_cert.py
# 2. 启动服务器
python tls_server.py
# 3. 启动客户端
python tls_client.py
```

输出应该显示加密协议和密码套件，证明通信是加密的。

### 1.6 验证证书的正确做法

```python
# 正式环境的客户端
context = ssl.create_default_context()
# 加载 CA 证书（或服务器证书）
context.load_verify_locations('cert.pem')
context.check_hostname = True
context.verify_mode = ssl.CERT_REQUIRED
```

如果证书无效（过期、域名不匹配、不受信任），连接会失败。

### 1.7 SSL 错误常见原因

| 错误 | 原因 |
|---|---|
| `CERTIFICATE_VERIFY_FAILED` | 证书不受信任（自签名开发环境应加载证书；禁用验证只适合临时实验） |
| `WRONG_VERSION_NUMBER` | 协议版本不匹配 |
| `SSLEOFError` | 对方意外关闭连接 |
| `Hostname mismatch` | 证书的域名与连接的不一致 |

### 📝 本关小结

- ✓ 理解 TLS 的作用与握手流程
- ✓ 生成自签名证书
- ✓ 用 `ssl` 模块实现加密通信
- ✓ 学会证书验证的正确方法

---

## 关卡 2：身份认证（中等）

### 📋 学习目标

- 理解为什么不能存明文密码
- 学会用哈希 + 盐存储密码
- 实现简单的 Token 认证
- 了解 JWT

### 2.1 密码存储的演进

**❌ 第一版：明文存密码**
```
用户名: alice
密码: 123456    ← 数据库泄露后所有用户裸奔
```

**⚠️ 第二版：MD5/SHA 哈希**
```
密码: e10adc3949ba59abbe56e057f20f883e  (MD5 of 123456)
```
问题：彩虹表攻击（黑客预先计算常见密码的哈希）。

**✅ 第三版：加盐哈希**
```
每个用户独立的盐: x7Kj2pNm
存储: SHA256(盐 + 密码) = a4f8c...
```
即使密码相同，盐不同，哈希值也不同。

**🎯 第四版（推荐）：bcrypt / scrypt / argon2**
专为密码设计的慢哈希算法，**故意设计得慢**以抵抗暴力破解。

### 2.2 使用 bcrypt

```bash
pip install bcrypt
```

```python
import bcrypt

# 注册：哈希密码
password = "my_secret_password"
hashed = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())
print(hashed)  # b'$2b$12$...'
# 存到数据库

# 登录：验证
input_password = "my_secret_password"
if bcrypt.checkpw(input_password.encode('utf-8'), hashed):
    print("✅ 密码正确")
else:
    print("❌ 密码错误")
```

bcrypt 自带盐，存储时直接存哈希即可。

### 2.3 简单的 Token 认证

登录成功后，服务器签发一个 **Token**，客户端后续请求都带上它。

**Token 设计**：

```
随机字符串：a8sdK2lP9qR3xZ7v...
```

服务器维护：`{token: 用户信息, 过期时间}`。

**`auth_server.py`：**

```python
import socket
import threading
import json
import struct
import secrets
import time
import bcrypt

# ========== 数据存储（实际应用应使用数据库） ==========
users = {}     # username -> hashed_password
tokens = {}    # token -> {'username': ..., 'expire': ...}
TOKEN_EXPIRE = 3600  # Token 1 小时过期

lock = threading.Lock()

# ========== 协议函数 ==========
def send_msg(sock, msg):
    data = json.dumps(msg).encode('utf-8')
    sock.sendall(struct.pack('!I', len(data)) + data)

def recv_n(sock, n):
    data = b''
    while len(data) < n:
        chunk = sock.recv(n - len(data))
        if not chunk:
            return None
        data += chunk
    return data

def recv_msg(sock):
    raw = recv_n(sock, 4)
    if not raw:
        return None
    length = struct.unpack('!I', raw)[0]
    data = recv_n(sock, length)
    if not data:
        return None
    return json.loads(data.decode('utf-8'))

# ========== 业务逻辑 ==========
def register(username, password):
    with lock:
        if username in users:
            return {'status': 'error', 'message': '用户名已存在'}
        hashed = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())
        users[username] = hashed
    return {'status': 'ok', 'message': '注册成功'}

def login(username, password):
    with lock:
        hashed = users.get(username)
        if not hashed:
            return {'status': 'error', 'message': '用户不存在'}
        if not bcrypt.checkpw(password.encode('utf-8'), hashed):
            return {'status': 'error', 'message': '密码错误'}
        # 生成 token
        token = secrets.token_hex(32)
        tokens[token] = {
            'username': username,
            'expire': time.time() + TOKEN_EXPIRE
        }
    return {'status': 'ok', 'token': token, 'expires_in': TOKEN_EXPIRE}

def verify_token(token):
    """验证 token，返回用户名或 None"""
    with lock:
        info = tokens.get(token)
        if not info:
            return None
        if time.time() > info['expire']:
            del tokens[token]
            return None
        return info['username']

def logout(token):
    with lock:
        if token in tokens:
            del tokens[token]
    return {'status': 'ok'}

# ========== 客户端处理 ==========
def handle_client(client_socket, addr):
    print(f"[新连接] {addr}")
    try:
        while True:
            req = recv_msg(client_socket)
            if not req:
                break

            action = req.get('action')
            print(f"[{addr}] {action}")

            if action == 'register':
                resp = register(req['username'], req['password'])
            elif action == 'login':
                resp = login(req['username'], req['password'])
            elif action == 'logout':
                resp = logout(req.get('token', ''))
            elif action == 'profile':
                # 需要认证才能访问
                username = verify_token(req.get('token', ''))
                if username:
                    resp = {'status': 'ok', 'username': username, 'data': '这是受保护的数据'}
                else:
                    resp = {'status': 'error', 'message': 'Token 无效或已过期'}
            else:
                resp = {'status': 'error', 'message': '未知操作'}

            send_msg(client_socket, resp)
    except Exception as e:
        print(f"[错误] {e}")
    finally:
        client_socket.close()

def main():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('127.0.0.1', 9000))
    server_socket.listen(10)
    print("🔐 认证服务器启动")

    while True:
        client, addr = server_socket.accept()
        t = threading.Thread(target=handle_client, args=(client, addr))
        t.daemon = True
        t.start()

if __name__ == "__main__":
    main()
```

### 2.4 客户端

**`auth_client.py`：**

```python
import socket
import json
import struct

class AuthClient:
    def __init__(self, host='127.0.0.1', port=9000):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((host, port))
        self.token = None

    def _send(self, msg):
        data = json.dumps(msg).encode('utf-8')
        self.sock.sendall(struct.pack('!I', len(data)) + data)

    def _recv(self):
        raw = self._recv_n(4)
        if not raw:
            return None
        length = struct.unpack('!I', raw)[0]
        data = self._recv_n(length)
        if not data:
            return None
        return json.loads(data.decode('utf-8'))

    def _recv_n(self, n):
        data = b''
        while len(data) < n:
            chunk = self.sock.recv(n - len(data))
            if not chunk:
                return None
            data += chunk
        return data

    def register(self, username, password):
        self._send({'action': 'register', 'username': username, 'password': password})
        return self._recv()

    def login(self, username, password):
        self._send({'action': 'login', 'username': username, 'password': password})
        resp = self._recv()
        if resp['status'] == 'ok':
            self.token = resp['token']
        return resp

    def profile(self):
        self._send({'action': 'profile', 'token': self.token})
        return self._recv()

    def logout(self):
        self._send({'action': 'logout', 'token': self.token})
        resp = self._recv()
        self.token = None
        return resp

def main():
    c = AuthClient()
    print("==== 认证系统 ====")
    print("命令：register / login / profile / logout / quit")

    while True:
        cmd = input("\n>>> ").strip().lower()
        try:
            if cmd == 'register':
                u = input("用户名：")
                p = input("密码：")
                print(c.register(u, p))
            elif cmd == 'login':
                u = input("用户名：")
                p = input("密码：")
                print(c.login(u, p))
            elif cmd == 'profile':
                print(c.profile())
            elif cmd == 'logout':
                print(c.logout())
            elif cmd == 'quit':
                break
        except Exception as e:
            print(f"❌ {e}")
            break

if __name__ == "__main__":
    main()
```

### 2.5 进阶：JWT（JSON Web Token）

之前的 token 需要服务器存储，多服务器部署时不方便。**JWT** 是一种**自包含**的 token：

```
JWT 结构：xxxxx.yyyyy.zzzzz
         头部. 载荷. 签名
```

- **头部**：算法类型
- **载荷**：用户信息、过期时间
- **签名**：用密钥签名，保证未被篡改

```bash
pip install pyjwt
```

```python
import jwt
import time

SECRET = "your-secret-key"

# 生成 JWT
payload = {
    'username': 'alice',
    'exp': time.time() + 3600  # 1 小时后过期
}
token = jwt.encode(payload, SECRET, algorithm='HS256')
print(token)

# 验证 JWT
try:
    data = jwt.decode(token, SECRET, algorithms=['HS256'])
    print(data)  # {'username': 'alice', 'exp': ...}
except jwt.ExpiredSignatureError:
    print("Token 已过期")
except jwt.InvalidTokenError:
    print("Token 无效")
```

JWT 优势：服务器**无需存储**，验签即可知道是否伪造。

### 📝 本关小结

- ✓ 用 bcrypt 安全存储密码
- ✓ 实现 Token 认证
- ✓ 了解 JWT 的原理

---

## 关卡 3：异常处理与断线重连（中等）

### 📋 学习目标

- 掌握网络编程的各种异常
- 学会优雅地处理异常
- 实现心跳检测
- 实现自动重连（含退避算法）

### 3.1 网络编程的常见异常

| 异常 | 原因 |
|---|---|
| `ConnectionRefusedError` | 服务器没启动或拒绝连接 |
| `ConnectionResetError` | 对方异常关闭连接 |
| `ConnectionAbortedError` | 本地中断了连接 |
| `BrokenPipeError` | 向已关闭的连接写数据 |
| `socket.timeout` | 操作超时 |
| `socket.gaierror` | 域名解析失败 |
| `OSError` | 其他网络错误（如网络不可达） |

### 3.2 系统性的异常处理

```python
import socket

def safe_connect(host, port, timeout=5):
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(timeout)
        sock.connect((host, port))
        return sock
    except ConnectionRefusedError:
        print(f"❌ 服务器 {host}:{port} 拒绝连接")
    except socket.timeout:
        print(f"❌ 连接超时")
    except socket.gaierror:
        print(f"❌ 无法解析域名 {host}")
    except OSError as e:
        print(f"❌ 网络错误：{e}")
    return None
```

### 3.3 心跳检测

**问题**：TCP 连接看起来还在，但对方可能已经掉线（拔网线、电脑死机），你的 `recv` 会一直等下去。

**解决**：定期发心跳包，确认对方还活着。

```
客户端 ── PING ──> 服务器
       <── PONG ──
   （30 秒收不到 PONG 就视为断线）
```

### 3.4 实现心跳

**服务端片段**：

```python
import time

def handle_client(sock, addr):
    sock.settimeout(60)  # 60 秒收不到任何数据就视为断线
    last_active = time.time()

    while True:
        try:
            data = sock.recv(1024).decode('utf-8')
            if not data:
                break

            if data == 'PING':
                sock.send(b'PONG')
                last_active = time.time()
            else:
                # 处理业务
                pass
        except socket.timeout:
            print(f"[{addr}] 心跳超时，断开")
            break
        except ConnectionResetError:
            break

    sock.close()
```

**客户端心跳线程**：

```python
import threading
import time

def heartbeat(sock, interval=10):
    """每 interval 秒发一次心跳"""
    while True:
        try:
            sock.send(b'PING')
            time.sleep(interval)
        except Exception:
            print("[心跳] 连接已断开")
            break

# 启动心跳线程
threading.Thread(target=heartbeat, args=(sock,), daemon=True).start()
```

### 3.5 自动重连：指数退避

断线后立即重连不是好选择：
- 服务器可能还没恢复
- 大量客户端同时重连会"打爆"服务器

**指数退避**：
```
第 1 次失败：等 1 秒
第 2 次失败：等 2 秒
第 3 次失败：等 4 秒
第 4 次失败：等 8 秒
...
最大等待 60 秒
```

### 3.6 健壮的可重连客户端

**`robust_client.py`：**

```python
import socket
import threading
import time
import random

class RobustClient:
    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.sock = None
        self.connected = False
        self.running = True
        self.reconnect_attempts = 0
        self.max_reconnect_delay = 60
        self.heartbeat_interval = 10

    def connect(self):
        """连接（带重试）"""
        while self.running and not self.connected:
            try:
                print(f"🔄 尝试连接 {self.host}:{self.port}（第 {self.reconnect_attempts + 1} 次）")
                self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                self.sock.settimeout(5)
                self.sock.connect((self.host, self.port))
                self.sock.settimeout(30)  # 业务超时改为 30s
                self.connected = True
                self.reconnect_attempts = 0
                print("✅ 连接成功")

                # 启动心跳和接收线程
                threading.Thread(target=self._heartbeat, daemon=True).start()
                threading.Thread(target=self._receive_loop, daemon=True).start()

            except Exception as e:
                self.reconnect_attempts += 1
                # 指数退避，加随机抖动避免雪崩
                delay = min(2 ** self.reconnect_attempts, self.max_reconnect_delay)
                delay += random.uniform(0, 1)
                print(f"❌ 连接失败：{e}，{delay:.1f} 秒后重试")
                time.sleep(delay)

    def _heartbeat(self):
        """心跳循环"""
        while self.connected and self.running:
            try:
                self.sock.send(b'PING\n')
                time.sleep(self.heartbeat_interval)
            except Exception:
                print("⚠️ 心跳失败")
                self._on_disconnect()
                break

    def _receive_loop(self):
        """接收循环"""
        buffer = b''
        while self.connected and self.running:
            try:
                data = self.sock.recv(4096)
                if not data:
                    print("⚠️ 服务器关闭连接")
                    self._on_disconnect()
                    break
                buffer += data
                # 按行处理
                while b'\n' in buffer:
                    line, buffer = buffer.split(b'\n', 1)
                    self._handle_message(line.decode('utf-8'))
            except socket.timeout:
                # 超时不一定是断线，继续
                continue
            except Exception as e:
                print(f"⚠️ 接收错误：{e}")
                self._on_disconnect()
                break

    def _handle_message(self, msg):
        if msg == 'PONG':
            pass  # 心跳响应，忽略
        else:
            print(f"📨 收到：{msg}")

    def _on_disconnect(self):
        """断线处理"""
        if not self.connected:
            return
        self.connected = False
        if self.sock:
            try:
                self.sock.close()
            except:
                pass
        print("🔌 已断开，准备重连...")
        if self.running:
            threading.Thread(target=self.connect, daemon=True).start()

    def send(self, msg):
        """发送消息（断线时丢弃或缓存）"""
        if not self.connected:
            print("⚠️ 未连接，消息未发送")
            return False
        try:
            self.sock.send((msg + '\n').encode('utf-8'))
            return True
        except Exception as e:
            print(f"❌ 发送失败：{e}")
            self._on_disconnect()
            return False

    def stop(self):
        self.running = False
        self.connected = False
        if self.sock:
            try:
                self.sock.close()
            except:
                pass

def main():
    client = RobustClient('127.0.0.1', 8888)
    client.connect()

    print("\n输入消息发送（quit 退出）")
    try:
        while True:
            msg = input("> ")
            if msg == 'quit':
                break
            client.send(msg)
    except KeyboardInterrupt:
        pass
    finally:
        client.stop()
        print("👋 再见")

if __name__ == "__main__":
    main()
```

**测试自动重连**：
1. 启动一个简单的 echo 服务器
2. 启动客户端 → 正常通信
3. 关掉服务器 → 客户端显示重试，开始指数退避
4. 重新启动服务器 → 客户端自动重连成功

### 3.7 ✨ 优雅关闭

```python
import signal
import sys

def signal_handler(sig, frame):
    print("\n收到退出信号，正在清理...")
    client.stop()
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)   # Ctrl+C
signal.signal(signal.SIGTERM, signal_handler)  # kill 命令
```

### 📝 本关小结

- ✓ 各种网络异常的处理
- ✓ 心跳检测机制
- ✓ 指数退避重连算法
- ✓ 优雅关闭

---

## 关卡 4：日志系统（简单）

### 📋 学习目标

- 理解日志的重要性
- 学会使用 Python 的 `logging` 模块
- 配置日志级别、格式、输出位置
- 学会日志文件分割

### 4.1 为什么不用 print？

`print` 的缺点：
- ❌ 无法区分严重程度
- ❌ 无法控制输出位置（只能终端）
- ❌ 调试完要手动删除
- ❌ 没有时间戳、模块名等元信息
- ❌ 难以分析（无结构）

**日志系统**优势：
- ✅ 分级别（DEBUG/INFO/WARNING/ERROR/CRITICAL）
- ✅ 可输出到文件、终端、网络
- ✅ 可按时间或大小自动分割
- ✅ 含时间戳、模块、行号等
- ✅ 一份代码，不同环境用不同配置

### 4.2 五个日志级别

| 级别 | 数值 | 用途 |
|---|---|---|
| DEBUG | 10 | 调试详情 |
| INFO | 20 | 一般信息 |
| WARNING | 30 | 警告（默认级别） |
| ERROR | 40 | 错误 |
| CRITICAL | 50 | 严重错误 |

**重要**：设置某级别，只输出 >= 该级别的日志。

### 4.3 logging 基础用法

```python
import logging

# 简单配置：输出到终端，INFO 级别及以上
logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s - %(levelname)s - %(message)s')

logging.debug("调试信息")    # 不显示（DEBUG < INFO）
logging.info("正常信息")     # ✅
logging.warning("警告")     # ✅
logging.error("错误")       # ✅
logging.critical("严重错误") # ✅
```

输出：
```
2024-01-15 10:30:15,123 - INFO - 正常信息
2024-01-15 10:30:15,124 - WARNING - 警告
2024-01-15 10:30:15,125 - ERROR - 错误
2024-01-15 10:30:15,126 - CRITICAL - 严重错误
```

### 4.4 格式化占位符

| 占位符 | 含义 |
|---|---|
| `%(asctime)s` | 时间 |
| `%(levelname)s` | 级别（INFO 等） |
| `%(name)s` | logger 名 |
| `%(filename)s` | 源文件名 |
| `%(lineno)d` | 行号 |
| `%(funcName)s` | 函数名 |
| `%(message)s` | 日志内容 |
| `%(thread)d` | 线程 ID |
| `%(threadName)s` | 线程名 |

### 4.5 完整的日志配置

```python
import logging
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

def setup_logger(name='myapp', log_file='app.log', level=logging.INFO):
    """配置 logger"""
    # 创建 logger
    logger = logging.getLogger(name)
    logger.setLevel(logging.DEBUG)  # logger 级别最低，由 handler 控制
    logger.handlers.clear()  # 避免重复添加

    # 格式
    fmt = logging.Formatter(
        '%(asctime)s [%(levelname)s] [%(threadName)s] '
        '%(filename)s:%(lineno)d - %(message)s'
    )

    # ===== Handler 1：控制台（INFO 以上）=====
    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    console.setFormatter(fmt)
    logger.addHandler(console)

    # ===== Handler 2：文件（DEBUG 以上，按大小分割）=====
    # 文件最大 10MB，保留 5 个历史文件
    file_handler = RotatingFileHandler(
        log_file, maxBytes=10*1024*1024, backupCount=5, encoding='utf-8'
    )
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(fmt)
    logger.addHandler(file_handler)

    # ===== Handler 3：错误日志单独存档 =====
    error_handler = RotatingFileHandler(
        'error.log', maxBytes=10*1024*1024, backupCount=5, encoding='utf-8'
    )
    error_handler.setLevel(logging.ERROR)
    error_handler.setFormatter(fmt)
    logger.addHandler(error_handler)

    return logger

# 使用
logger = setup_logger()
logger.debug("详细调试")     # 只在文件
logger.info("正常运行")      # 终端 + 文件
logger.warning("警告")       # 终端 + 文件
logger.error("出错了")       # 终端 + 文件 + 错误日志
logger.critical("严重错误")  # 同上
```

### 4.6 按时间分割

```python
from logging.handlers import TimedRotatingFileHandler

handler = TimedRotatingFileHandler(
    'app.log',
    when='midnight',      # 每天午夜分割
    interval=1,           # 间隔 1 天
    backupCount=7,        # 保留 7 天
    encoding='utf-8'
)
```

每天生成：`app.log.2024-01-15`, `app.log.2024-01-16`...

### 4.7 在网络应用中使用

**`logged_server.py`：**

```python
import socket
import threading
import logging
from logging.handlers import RotatingFileHandler

# ===== 配置日志 =====
def setup_logger():
    logger = logging.getLogger('chat_server')
    logger.setLevel(logging.DEBUG)
    fmt = logging.Formatter('%(asctime)s [%(levelname)s] [%(threadName)s] %(message)s')

    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    console.setFormatter(fmt)

    file_handler = RotatingFileHandler('server.log', maxBytes=5*1024*1024, backupCount=3, encoding='utf-8')
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(fmt)

    logger.addHandler(console)
    logger.addHandler(file_handler)
    return logger

log = setup_logger()

def handle_client(client_socket, addr):
    log.info(f"新连接：{addr}")
    try:
        while True:
            data = client_socket.recv(1024)
            if not data:
                log.info(f"客户端正常断开：{addr}")
                break
            msg = data.decode('utf-8', errors='ignore').strip()
            log.debug(f"收到来自 {addr} 的消息：{msg}")

            try:
                client_socket.send(f"echo: {msg}\n".encode('utf-8'))
            except Exception as e:
                log.error(f"发送失败 to {addr}: {e}")
                break
    except ConnectionResetError:
        log.warning(f"连接重置：{addr}")
    except Exception as e:
        log.exception(f"处理 {addr} 时发生异常")  # exception 会自动打印堆栈
    finally:
        client_socket.close()

def main():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(('127.0.0.1', 8888))
    server.listen(10)
    log.info("服务器启动，监听 127.0.0.1:8888")

    try:
        while True:
            client, addr = server.accept()
            t = threading.Thread(target=handle_client, args=(client, addr), name=f"Worker-{addr[1]}")
            t.daemon = True
            t.start()
    except KeyboardInterrupt:
        log.info("收到中断信号，关闭服务器")
    finally:
        server.close()

if __name__ == "__main__":
    main()
```

### 4.8 💡 最佳实践

1. **不要打印密码、密钥**等敏感信息
2. 使用 `logger.exception()` 在 `except` 块中打印**完整堆栈**
3. 不同模块用不同 logger：`logging.getLogger(__name__)`
4. 生产环境用 INFO 级别，调试时改 DEBUG
5. 日志文件路径用绝对路径，避免运行目录变化

### 📝 本关小结

- ✓ 配置完整的日志系统
- ✓ 日志分级与多输出
- ✓ 自动文件分割

---

## 关卡 5：完整聊天系统（困难）

### 📋 学习目标

这是**最终大 BOSS**！综合应用之前学的所有知识，开发一个完整的聊天系统。

**功能清单**：
- ✅ 用户注册、登录、Token 认证
- ✅ 多房间（频道）支持
- ✅ 公屏消息 + 私聊
- ✅ 在线用户列表
- ✅ 历史消息
- ✅ TLS 加密通信
- ✅ 完整日志系统
- ✅ 异常处理与断线重连

### 5.1 项目结构

```
chat_system/
├── server/
│   ├── server.py          # 服务器主程序
│   ├── auth.py            # 认证模块
│   ├── chatroom.py        # 房间管理
│   └── protocol.py        # 协议
├── client/
│   ├── client.py          # 客户端
│   └── protocol.py        # 协议（同上）
├── certs/
│   ├── cert.pem
│   └── key.pem
└── data/
    ├── users.json         # 用户数据
    └── server.log
```

### 5.2 协议设计

所有消息用 JSON，长度前缀。**消息类型**：

| type | 方向 | 说明 |
|---|---|---|
| register | C→S | 注册 |
| login | C→S | 登录 |
| logout | C→S | 退出登录 |
| join_room | C→S | 加入房间 |
| leave_room | C→S | 离开房间 |
| message | C→S | 发送消息 |
| private | C→S | 私聊 |
| list_rooms | C→S | 房间列表 |
| list_users | C→S | 在线用户 |
| history | C→S | 历史消息 |
| ping | C→S | 心跳 |
| response | S→C | 通用响应 |
| broadcast | S→C | 广播消息 |
| notify | S→C | 通知（用户加入/离开） |
| private_msg | S→C | 收到私聊 |
| pong | S→C | 心跳响应 |

### 5.3 `protocol.py`

```python
import struct
import json

MAX_FRAME_SIZE = 1024 * 1024  # 限制单条 JSON 消息最大 1MB，避免恶意长度耗尽内存

def send_msg(sock, msg):
    """发送 JSON 消息"""
    data = json.dumps(msg, ensure_ascii=False).encode('utf-8')
    sock.sendall(struct.pack('!I', len(data)) + data)

def recv_msg(sock):
    """接收 JSON 消息（None 表示连接关闭）"""
    raw = recv_n(sock, 4)
    if not raw:
        return None
    length = struct.unpack('!I', raw)[0]
    if length > MAX_FRAME_SIZE:
        return None
    data = recv_n(sock, length)
    if not data:
        return None
    return json.loads(data.decode('utf-8'))

def recv_n(sock, n):
    data = b''
    while len(data) < n:
        chunk = sock.recv(min(n - len(data), 65536))
        if not chunk:
            return None
        data += chunk
    return data
```

### 5.4 `auth.py`（认证模块）

```python
import os
import json
import bcrypt
import secrets
import time
import threading

USERS_FILE = 'data/users.json'
TOKEN_EXPIRE = 3600 * 24  # 24 小时

class AuthManager:
    def __init__(self):
        self.users = {}      # username -> hashed_password
        self.tokens = {}     # token -> {'username': ..., 'expire': ...}
        self.lock = threading.Lock()
        self._load_users()

    def _load_users(self):
        os.makedirs(os.path.dirname(USERS_FILE), exist_ok=True)
        if os.path.exists(USERS_FILE):
            try:
                with open(USERS_FILE, 'r', encoding='utf-8') as f:
                    self.users = json.load(f)
            except:
                self.users = {}

    def _save_users(self):
        with open(USERS_FILE, 'w', encoding='utf-8') as f:
            json.dump(self.users, f, ensure_ascii=False, indent=2)

    def register(self, username, password):
        with self.lock:
            if not username or not password:
                return False, '用户名或密码为空'
            if len(username) < 3 or len(username) > 20:
                return False, '用户名长度必须 3-20'
            if username in self.users:
                return False, '用户名已存在'
            hashed = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt()).decode('utf-8')
            self.users[username] = hashed
            self._save_users()
            return True, '注册成功'

    def login(self, username, password):
        with self.lock:
            hashed = self.users.get(username)
            if not hashed:
                return None, '用户不存在'
            if not bcrypt.checkpw(password.encode('utf-8'), hashed.encode('utf-8')):
                return None, '密码错误'
            token = secrets.token_hex(32)
            self.tokens[token] = {
                'username': username,
                'expire': time.time() + TOKEN_EXPIRE
            }
            return token, '登录成功'

    def verify(self, token):
        with self.lock:
            info = self.tokens.get(token)
            if not info:
                return None
            if time.time() > info['expire']:
                del self.tokens[token]
                return None
            return info['username']

    def logout(self, token):
        with self.lock:
            if token in self.tokens:
                del self.tokens[token]
```

### 5.5 `chatroom.py`（房间管理）

```python
import threading
import time
from collections import deque

MAX_HISTORY = 50

class Room:
    def __init__(self, name):
        self.name = name
        self.members = {}                # username -> client_socket
        self.history = deque(maxlen=MAX_HISTORY)
        self.lock = threading.Lock()

    def add_member(self, username, sock):
        with self.lock:
            self.members[username] = sock

    def remove_member(self, username):
        with self.lock:
            if username in self.members:
                del self.members[username]

    def get_members(self):
        with self.lock:
            return list(self.members.keys())

    def add_message(self, username, content):
        msg = {
            'username': username,
            'content': content,
            'time': time.strftime('%H:%M:%S')
        }
        with self.lock:
            self.history.append(msg)
        return msg

    def get_history(self):
        with self.lock:
            return list(self.history)


class RoomManager:
    def __init__(self):
        self.rooms = {'大厅': Room('大厅')}
        self.user_room = {}  # username -> room_name
        self.lock = threading.Lock()

    def get_or_create(self, name):
        with self.lock:
            if name not in self.rooms:
                self.rooms[name] = Room(name)
            return self.rooms[name]

    def list_rooms(self):
        with self.lock:
            return [
                {'name': r.name, 'count': len(r.members)}
                for r in self.rooms.values()
            ]

    def join(self, username, sock, room_name):
        # 先离开旧房间
        self.leave(username)
        room = self.get_or_create(room_name)
        room.add_member(username, sock)
        with self.lock:
            self.user_room[username] = room_name
        return room

    def leave(self, username):
        with self.lock:
            room_name = self.user_room.pop(username, None)
        if room_name and room_name in self.rooms:
            self.rooms[room_name].remove_member(username)
            return room_name
        return None

    def get_user_room(self, username):
        with self.lock:
            room_name = self.user_room.get(username)
            return self.rooms.get(room_name) if room_name else None
```

### 5.6 服务器主程序 `server.py`

```python
import socket
import ssl
import threading
import logging
import os
import sys
from logging.handlers import RotatingFileHandler

# 添加上级路径以便导入
sys.path.insert(0, os.path.dirname(__file__))
from protocol import send_msg, recv_msg
from auth import AuthManager
from chatroom import RoomManager

# ===== 日志配置 =====
def setup_logger():
    os.makedirs('data', exist_ok=True)
    logger = logging.getLogger('chat')
    logger.setLevel(logging.DEBUG)
    fmt = logging.Formatter('%(asctime)s [%(levelname)s] [%(threadName)s] %(message)s')

    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    console.setFormatter(fmt)
    logger.addHandler(console)

    fh = RotatingFileHandler('data/server.log', maxBytes=10*1024*1024, backupCount=5, encoding='utf-8')
    fh.setLevel(logging.DEBUG)
    fh.setFormatter(fmt)
    logger.addHandler(fh)
    return logger

log = setup_logger()

# ===== 全局组件 =====
auth = AuthManager()
rooms = RoomManager()

# 在线连接：sock -> username
online_users = {}            # username -> sock
online_lock = threading.Lock()

# ===== 业务处理 =====
def broadcast_to_room(room, msg, exclude_user=None):
    """向房间所有人广播"""
    for username, member_sock in list(room.members.items()):
        if username != exclude_user:
            try:
                send_msg(member_sock, msg)
            except Exception as e:
                log.error(f"广播失败 → {username}: {e}")

def send_response(sock, status, message='', **extra):
    """发送通用响应"""
    msg = {'type': 'response', 'status': status, 'message': message}
    msg.update(extra)
    try:
        send_msg(sock, msg)
    except Exception as e:
        log.error(f"发送响应失败：{e}")

def handle_client(client_socket, addr):
    log.info(f"新连接：{addr}")
    username = None
    token = None

    try:
        while True:
            req = recv_msg(client_socket)
            if not req:
                log.info(f"客户端断开：{addr}")
                break

            msg_type = req.get('type')
            log.debug(f"[{addr}] 收到 type={msg_type}")

            # ========== 心跳 ==========
            if msg_type == 'ping':
                send_msg(client_socket, {'type': 'pong'})
                continue

            # ========== 注册 ==========
            if msg_type == 'register':
                ok, message = auth.register(req.get('username'), req.get('password'))
                send_response(client_socket, 'ok' if ok else 'error', message)
                continue

            # ========== 登录 ==========
            if msg_type == 'login':
                # 已登录的话先登出
                if username:
                    _handle_logout(username, token, client_socket, revoke_token=True)
                    username = None

                resume_token = req.get('token')
                if resume_token:
                    verified = auth.verify(resume_token)
                    new_token = resume_token if verified else None
                    message = '恢复登录成功' if verified else 'Token 无效或已过期'
                    req['username'] = verified
                else:
                    new_token, message = auth.login(req.get('username'), req.get('password'))
                if new_token:
                    username = req['username']
                    token = new_token
                    with online_lock:
                        # 同一用户重复登录：踢掉之前的
                        if username in online_users:
                            old_sock = online_users[username]
                            try:
                                send_msg(old_sock, {
                                    'type': 'notify',
                                    'content': '你的账号在别处登录，本连接被强制退出'
                                })
                                old_sock.shutdown(socket.SHUT_RDWR)
                            except:
                                pass
                        online_users[username] = client_socket
                    send_response(client_socket, 'ok', message, token=token, username=username)
                    log.info(f"用户登录：{username} from {addr}")

                    # 自动加入大厅
                    room = rooms.join(username, client_socket, '大厅')
                    broadcast_to_room(room, {
                        'type': 'notify',
                        'content': f"📥 {username} 加入了 {room.name}"
                    }, exclude_user=username)
                else:
                    send_response(client_socket, 'error', message)
                continue

            # ========== 以下都需要登录 ==========
            if not username:
                # 还未登录，校验 token
                req_token = req.get('token')
                if req_token:
                    verified = auth.verify(req_token)
                    if verified:
                        username = verified
                        token = req_token
                        with online_lock:
                            online_users[username] = client_socket
                if not username:
                    send_response(client_socket, 'error', '请先登录')
                    continue

            # ========== 登出 ==========
            if msg_type == 'logout':
                _handle_logout(username, token, client_socket, revoke_token=True)
                send_response(client_socket, 'ok', '已退出')
                username = None
                token = None
                continue

            # ========== 加入房间 ==========
            if msg_type == 'join_room':
                room_name = req.get('room', '大厅')
                # 先通知旧房间
                old_room = rooms.get_user_room(username)
                if old_room:
                    broadcast_to_room(old_room, {
                        'type': 'notify',
                        'content': f"📤 {username} 离开了 {old_room.name}"
                    }, exclude_user=username)

                room = rooms.join(username, client_socket, room_name)
                send_response(client_socket, 'ok', f'已加入 {room.name}',
                              room=room.name, members=room.get_members())
                broadcast_to_room(room, {
                    'type': 'notify',
                    'content': f"📥 {username} 加入了 {room.name}"
                }, exclude_user=username)
                log.info(f"{username} 加入房间 {room.name}")
                continue

            # ========== 发送消息（房间内）==========
            if msg_type == 'message':
                content = req.get('content', '').strip()
                if not content:
                    continue
                room = rooms.get_user_room(username)
                if not room:
                    send_response(client_socket, 'error', '你不在任何房间')
                    continue
                msg = room.add_message(username, content)
                broadcast_to_room(room, {
                    'type': 'broadcast',
                    'username': msg['username'],
                    'content': msg['content'],
                    'time': msg['time'],
                    'room': room.name,
                })
                log.info(f"[{room.name}] {username}: {content}")
                continue

            # ========== 私聊 ==========
            if msg_type == 'private':
                target = req.get('to')
                content = req.get('content', '').strip()
                if not target or not content:
                    send_response(client_socket, 'error', '参数错误')
                    continue
                with online_lock:
                    target_sock = online_users.get(target)
                if not target_sock:
                    send_response(client_socket, 'error', f'{target} 不在线')
                    continue
                try:
                    send_msg(target_sock, {
                        'type': 'private_msg',
                        'from': username,
                        'content': content
                    })
                    send_response(client_socket, 'ok', '已发送')
                    log.info(f"私聊 {username} → {target}: {content}")
                except Exception as e:
                    log.error(f"私聊发送失败：{e}")
                    send_response(client_socket, 'error', '发送失败')
                continue

            # ========== 房间列表 ==========
            if msg_type == 'list_rooms':
                send_response(client_socket, 'ok', rooms_list=rooms.list_rooms())
                continue

            # ========== 用户列表 ==========
            if msg_type == 'list_users':
                room = rooms.get_user_room(username)
                if room:
                    send_response(client_socket, 'ok', users=room.get_members(),
                                  room=room.name)
                continue

            # ========== 历史消息 ==========
            if msg_type == 'history':
                room = rooms.get_user_room(username)
                if room:
                    send_response(client_socket, 'ok', history=room.get_history())
                continue

            send_response(client_socket, 'error', f'未知类型：{msg_type}')

    except ConnectionResetError:
        log.warning(f"连接重置：{addr}")
    except Exception as e:
        log.exception(f"处理 {addr} 异常")
    finally:
        if username:
            _handle_logout(username, token, client_socket, revoke_token=False)
        try:
            client_socket.close()
        except:
            pass
        log.info(f"清理连接：{addr}")

def _handle_logout(username, token, client_socket=None, revoke_token=True):
    """统一登出逻辑"""
    if not username:
        return
    with online_lock:
        if client_socket is not None and online_users.get(username) is not client_socket:
            return
    # 通知房间其他人
    room = rooms.get_user_room(username)
    if room:
        broadcast_to_room(room, {
            'type': 'notify',
            'content': f"📤 {username} 离线了"
        }, exclude_user=username)
    rooms.leave(username)
    with online_lock:
        if username in online_users and (client_socket is None or online_users.get(username) is client_socket):
            del online_users[username]
    if token and revoke_token:
        auth.logout(token)
    log.info(f"用户离线：{username}")

def main():
    use_tls = '--tls' in sys.argv

    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind(('0.0.0.0', 8888))
    server_socket.listen(50)

    if use_tls:
        context = ssl.create_default_context(ssl.Purpose.CLIENT_AUTH)
        context.load_cert_chain(certfile='certs/cert.pem', keyfile='certs/key.pem')
        log.info("🔒 启用 TLS")
    else:
        context = None

    log.info(f"🚀 聊天服务器启动 :8888 {'(TLS)' if use_tls else '(明文)'}")

    try:
        while True:
            raw_sock, addr = server_socket.accept()
            try:
                if context:
                    client_sock = context.wrap_socket(raw_sock, server_side=True)
                else:
                    client_sock = raw_sock

                t = threading.Thread(
                    target=handle_client, args=(client_sock, addr),
                    name=f"Worker-{addr[1]}", daemon=True
                )
                t.start()
            except ssl.SSLError as e:
                log.warning(f"TLS 握手失败 {addr}: {e}")
                raw_sock.close()
    except KeyboardInterrupt:
        log.info("收到中断信号")
    finally:
        server_socket.close()
        log.info("服务器已关闭")

if __name__ == "__main__":
    main()
```

### 5.7 客户端 `client.py`

```python
import socket
import ssl
import threading
import time
import sys
import os
import random
import logging

sys.path.insert(0, os.path.dirname(__file__))
from protocol import send_msg, recv_msg

# 简单日志
logging.basicConfig(level=logging.INFO, format='%(asctime)s [%(levelname)s] %(message)s')
log = logging.getLogger('client')

class ChatClient:
    def __init__(self, host='127.0.0.1', port=8888, use_tls=False):
        self.host = host
        self.port = port
        self.use_tls = use_tls
        self.sock = None
        self.connected = False
        self.running = True
        self.username = None
        self.token = None
        self.lock = threading.Lock()
        self.reconnect_attempts = 0

    def connect(self):
        """带重试的连接"""
        while self.running and not self.connected:
            try:
                log.info(f"连接 {self.host}:{self.port}（第 {self.reconnect_attempts + 1} 次）")
                raw = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                raw.settimeout(5)
                raw.connect((self.host, self.port))

                if self.use_tls:
                    ctx = ssl.create_default_context()
                    ctx.load_verify_locations('certs/cert.pem')
                    ctx.check_hostname = True
                    ctx.verify_mode = ssl.CERT_REQUIRED
                    self.sock = ctx.wrap_socket(raw, server_hostname=self.host)
                else:
                    self.sock = raw

                self.sock.settimeout(30)
                self.connected = True
                self.reconnect_attempts = 0
                log.info("✅ 连接成功")

                # 启动接收线程
                threading.Thread(target=self._recv_loop, daemon=True).start()
                threading.Thread(target=self._heartbeat, daemon=True).start()

                # 自动用 token 重新登录
                if self.token:
                    self._send({'type': 'login', 'token': self.token})
                return True
            except Exception as e:
                self.reconnect_attempts += 1
                delay = min(2 ** self.reconnect_attempts, 30) + random.uniform(0, 1)
                log.error(f"连接失败：{e}，{delay:.1f}s 后重试")
                time.sleep(delay)
        return False

    def _send(self, msg):
        try:
            with self.lock:
                send_msg(self.sock, msg)
            return True
        except Exception as e:
            log.error(f"发送失败：{e}")
            self._on_disconnect()
            return False

    def _recv_loop(self):
        while self.connected and self.running:
            try:
                msg = recv_msg(self.sock)
                if not msg:
                    self._on_disconnect()
                    break
                self._handle_message(msg)
            except socket.timeout:
                continue
            except Exception as e:
                log.error(f"接收异常：{e}")
                self._on_disconnect()
                break

    def _heartbeat(self):
        while self.connected and self.running:
            time.sleep(15)
            if self.connected:
                self._send({'type': 'ping'})

    def _handle_message(self, msg):
        t = msg.get('type')
        if t == 'pong':
            return
        elif t == 'broadcast':
            print(f"\n💬 [{msg['time']}] [{msg['username']}] {msg['content']}")
        elif t == 'private_msg':
            print(f"\n🔒 [私聊][{msg['from']}] {msg['content']}")
        elif t == 'notify':
            print(f"\n📢 [系统] {msg['content']}")
        elif t == 'response':
            if msg['status'] == 'ok':
                # 登录成功后保存 token
                if 'token' in msg:
                    self.token = msg['token']
                    self.username = msg.get('username')
                self._print_response(msg)
            else:
                print(f"\n❌ {msg.get('message')}")
        self._prompt()

    def _print_response(self, msg):
        if 'users' in msg:
            print(f"\n👥 [{msg.get('room', '')}] 在线 {len(msg['users'])} 人：{', '.join(msg['users'])}")
        elif 'rooms_list' in msg:
            print("\n📋 房间列表：")
            for r in msg['rooms_list']:
                print(f"  - {r['name']} ({r['count']} 人)")
        elif 'history' in msg:
            print("\n📜 历史消息：")
            for m in msg['history']:
                print(f"  [{m['time']}] [{m['username']}] {m['content']}")
        elif 'room' in msg:
            print(f"\n✅ {msg.get('message', '')}")
            if msg.get('members'):
                print(f"   在线：{', '.join(msg['members'])}")
        else:
            if msg.get('message'):
                print(f"\n✅ {msg['message']}")

    def _prompt(self):
        print("> ", end='', flush=True)

    def _on_disconnect(self):
        if not self.connected:
            return
        self.connected = False
        try:
            self.sock.close()
        except:
            pass
        log.warning("连接断开")
        if self.running:
            threading.Thread(target=self.connect, daemon=True).start()

    def stop(self):
        self.running = False
        self.connected = False
        if self.sock:
            try:
                self.sock.close()
            except:
                pass

    # ===== 用户命令 =====
    def register(self, u, p):
        self._send({'type': 'register', 'username': u, 'password': p})

    def login(self, u, p):
        self._send({'type': 'login', 'username': u, 'password': p})

    def logout(self):
        self._send({'type': 'logout'})
        self.token = None
        self.username = None

    def say(self, content):
        self._send({'type': 'message', 'content': content})

    def private(self, target, content):
        self._send({'type': 'private', 'to': target, 'content': content})

    def join_room(self, room):
        self._send({'type': 'join_room', 'room': room})

    def list_rooms(self):
        self._send({'type': 'list_rooms'})

    def list_users(self):
        self._send({'type': 'list_users'})

    def history(self):
        self._send({'type': 'history'})

def show_help():
    print("""
====================== 命令帮助 ======================
/register <用户名> <密码>     注册
/login <用户名> <密码>        登录
/logout                       退出登录
/join <房间名>                加入房间
/rooms                        列出房间
/users                        当前房间用户
/history                      历史消息
/pm <用户> <内容>             私聊
/quit                         退出
不以 / 开头的输入将作为消息发送到当前房间
=====================================================
""")

def main():
    use_tls = '--tls' in sys.argv
    host = '127.0.0.1'

    client = ChatClient(host=host, use_tls=use_tls)
    if not client.connect():
        return

    show_help()

    try:
        while True:
            line = input("> ").strip()
            if not line:
                continue

            if not line.startswith('/'):
                # 普通消息
                client.say(line)
                continue

            parts = line.split(maxsplit=2)
            cmd = parts[0][1:]

            if cmd == 'help':
                show_help()
            elif cmd == 'register' and len(parts) >= 3:
                client.register(parts[1], parts[2])
            elif cmd == 'login' and len(parts) >= 3:
                client.login(parts[1], parts[2])
            elif cmd == 'logout':
                client.logout()
            elif cmd == 'join' and len(parts) >= 2:
                client.join_room(parts[1])
            elif cmd == 'rooms':
                client.list_rooms()
            elif cmd == 'users':
                client.list_users()
            elif cmd == 'history':
                client.history()
            elif cmd == 'pm' and len(parts) >= 3:
                client.private(parts[1], parts[2])
            elif cmd == 'quit':
                client.logout()
                time.sleep(0.5)
                break
            else:
                print("❌ 未知命令，输入 /help 查看")
    except (KeyboardInterrupt, EOFError):
        pass
    finally:
        client.stop()
        print("\n👋 再见！")

if __name__ == "__main__":
    main()
```

### 5.8 运行步骤

```bash
# 1. 安装依赖
pip install bcrypt cryptography

# 2. （可选）生成 TLS 证书（前面 generate_cert.py），放到 certs/

# 3. 启动服务器（明文版，仅限本机学习；密码和 token 会明文传输）
python server/server.py
# 推荐启用 TLS：
# python server/server.py --tls

# 4. 启动多个客户端
python client/client.py
# TLS 版本：
# python client/client.py --tls
```

### 5.9 使用流程示例

```
# 客户端 A
> /register alice 123456
✅ 注册成功
> /login alice 123456
✅ 登录成功
   在线：alice

# 客户端 B
> /register bob 123456
> /login bob 123456
📢 [系统] 📥 bob 加入了 大厅   ← A 看到通知

# A 发消息
> 大家好啊！
# B 看到：
💬 [10:23:45] [alice] 大家好啊！

# 私聊
> /pm alice 嗨 Alice
# A 看到：
🔒 [私聊][bob] 嗨 Alice

# 创建/加入新房间
> /join 摸鱼频道
✅ 已加入 摸鱼频道
```

### 5.10 🎉 你已完成所有挑战

恭喜你！通过这套教程，你已经从零开始，构建了一个：
- 支持**TLS 加密**通信
- 拥有**完整用户系统**
- 支持**多房间、私聊、广播**
- 具备**心跳与自动重连**
- 拥有**专业日志系统**
- **生产级别**的聊天系统！

---

# 🎓 总结与展望

## 你已掌握的技能树

```
🎯 Python 网络编程
├── 🔵 基础
│   ├── TCP Socket 编程
│   ├── UDP Socket 编程
│   ├── 客户端/服务端模型
│   └── 协议设计（粘包处理）
├── 🟢 进阶
│   ├── 多线程并发
│   ├── 非阻塞 IO
│   ├── IO 多路复用（select/selectors）
│   └── HTTP 协议实现
├── 🟡 工程化
│   ├── TLS/SSL 加密
│   ├── 用户认证（bcrypt/Token/JWT）
│   ├── 异常处理与重连
│   └── 日志系统
└── 🟣 综合项目
    ├── 文件传输系统
    └── 完整聊天系统
```

## 进一步学习

| 方向 | 推荐资源 |
|---|---|
| 🚀 **异步编程** | `asyncio`, `aiohttp` |
| 🌐 **Web 框架** | Flask, FastAPI, Django |
| 📡 **WebSocket** | `websockets` 库 |
| ⚡ **高性能** | `uvloop`, Gunicorn |
| 📦 **消息队列** | RabbitMQ, Kafka, Redis |
| 🐳 **部署运维** | Docker, Kubernetes, Nginx |
| 🛡️ **网络安全** | 渗透测试、OWASP Top 10 |
| 🌐 **协议分析** | Wireshark, tcpdump |

## 💪 给你的最后建议

1. **多写代码**：网络编程必须动手，光看不会
2. **多调试**：用 `print` / `logging` / 调试器观察程序行为
3. **看源码**：研究 `socket`、`requests`、`Flask` 的实现
4. **做项目**：尝试做个 IM、爬虫、游戏服务器
5. **读 RFC**：HTTP/2、TLS 1.3 等协议规范是金矿
6. **加入社区**：Stack Overflow、GitHub、技术论坛

> 🌟 **写代码 ≠ 理解原理。但理解了原理，再用任何框架都得心应手。**
>
> 网络的世界没有终点，恭喜你踏出了坚实的第一步！
>
> **Happy hacking! 💻🚀**
