# Linux 操作入门教程（WSL2 版）

> 本教程面向零基础新手，内容由浅入深，配合大量示例，可直接作为教材使用。

---

# 第一章：Linux 基础认知

---

## 1.1 Linux 是什么？

### 1.1.1 一句话理解

**Linux 是一个操作系统的内核（Kernel）**，就像 Windows 的"核心引擎"一样，它负责管理你电脑的硬件资源（CPU、内存、磁盘、网络等），并为上层应用程序提供运行环境。

但在日常使用中，我们说的"Linux"通常指的是 **基于 Linux 内核的完整操作系统**，也叫 **GNU/Linux**。

### 1.1.2 Linux 的诞生

| 时间  | 事件                                                         |
| ----- | ------------------------------------------------------------ |
| 1969  | AT&T 贝尔实验室开发了 **Unix** 操作系统                      |
| 1983  | Richard Stallman 发起 **GNU 计划**，目标是做一个自由的类 Unix 系统 |
| 1991  | 芬兰大学生 **Linus Torvalds** 编写了 Linux 内核，并开源发布  |
| 1992+ | Linux 内核 + GNU 工具 = 完整的操作系统，各种**发行版**开始涌现 |

> 💡 **关键理解**：Linux 内核本身只是一个"引擎"，它需要配合 Shell（命令行）、文件管理器、包管理器等工具才能成为一个可用的操作系统。不同的人把这些工具打包成不同的"套餐"，就形成了不同的**发行版**。

### 1.1.3 为什么要学 Linux？

- **服务器领域的绝对霸主**：全球超过 96% 的服务器运行 Linux（包括阿里云、AWS、Google Cloud）
- **开发者的标配**：Python / Node.js / Java / Go 等语言的开发生态都对 Linux 支持最好
- **免费且开源**：任何人都可以免费使用、修改、分发
- **高度可定制**：你可以精确控制系统的每一个行为
- **安全性强**：权限模型严格，病毒极少

---

## 1.2 主流发行版介绍

"发行版"（Distribution，简称 Distro）= Linux 内核 + 软件包 + 包管理器 + 桌面环境（可选）+ 默认配置

### 三大主流发行版对比

| 特性         | Ubuntu                | CentOS / Rocky Linux     | Debian               |
| ------------ | --------------------- | ------------------------ | -------------------- |
| **定位**     | 新手友好，桌面+服务器 | 企业服务器               | 稳定至上，纯社区驱动 |
| **包管理器** | `apt`（.deb 包）      | `yum` / `dnf`（.rpm 包） | `apt`（.deb 包）     |
| **更新频率** | 每 6 个月一个版本     | 跟随 RHEL 发布           | 约 2 年一个稳定版    |
| **社区支持** | ⭐⭐⭐⭐⭐ 非常活跃        | ⭐⭐⭐⭐ 企业级文档多        | ⭐⭐⭐⭐ 技术深度高      |
| **WSL 默认** | ✅ WSL 商店默认推荐    | 需手动安装               | 可安装               |
| **推荐人群** | 初学者、开发者        | 运维工程师               | 有经验的用户         |

### Ubuntu 详解（本教程使用）

Ubuntu 是目前**最流行的 Linux 桌面发行版**，也是 WSL 的默认推荐发行版。

- **基于 Debian**：继承了 Debian 的稳定性和 `apt` 包管理器
- **LTS 版本**：每 2 年发布一个长期支持版本（如 22.04 LTS、24.04 LTS），支持 5 年
- **命名规则**：年份.月份，如 `24.04` 表示 2024 年 4 月发布

```
Ubuntu 版本命名：
24.04 = 2024年4月发布（LTS = 长期支持版）
24.10 = 2024年10月发布（非LTS，短期支持）
```

### 其他值得知道的发行版

| 发行版         | 特点                                          |
| -------------- | --------------------------------------------- |
| **Fedora**     | Red Hat 的社区版，技术前沿，适合尝鲜          |
| **Arch Linux** | 极客最爱，滚动更新，高度定制                  |
| **Alpine**     | 极小体积（约 5MB），Docker 容器的首选基础镜像 |
| **Kali Linux** | 网络安全 / 渗透测试专用                       |

---

## 1.3 Linux 与 Windows 的核心区别

### 1.3.1 设计哲学对比

| 维度               | Windows                           | Linux                                   |
| ------------------ | --------------------------------- | --------------------------------------- |
| **交互方式**       | 以图形界面（GUI）为主             | 以命令行（CLI）为主，GUI 为辅           |
| **核心理念**       | "开箱即用"                        | "一切皆文件"                            |
| **配置方式**       | 注册表 + 图形化设置               | **文本配置文件**（如 `/etc/` 下的文件） |
| **软件安装**       | 下载 .exe 安装包                  | **包管理器**（如 `apt install`）        |
| **文件路径分隔符** | 反斜杠 `\`（如 `C:\Users\`）      | 正斜杠 `/`（如 `/home/user/`）          |
| **大小写敏感**     | 不敏感（`File.txt` = `file.txt`） | **敏感**（`File.txt` ≠ `file.txt`）     |
| **文件扩展名**     | 决定文件类型                      | 扩展名只是命名习惯，不决定文件类型      |
| **换行符**         | `\r\n`（CRLF）                    | `\n`（LF）                              |

### 1.3.2 "一切皆文件"——Linux 最重要的设计理念

在 Linux 中，**几乎所有东西都被抽象为文件**：

```
普通文档           → 文件
目录/文件夹        → 文件（一种特殊类型的文件）
硬盘               → /dev/sda（设备文件）
U盘                → /dev/sdb（设备文件）
键盘输入           → /dev/stdin（标准输入文件）
屏幕输出           → /dev/stdout（标准输出文件）
正在运行的进程      → /proc/1234/（虚拟文件系统中的文件）
网络连接           → /dev/net/（网络设备文件）
系统配置           → /etc/ 下的文本文件
```

**为什么这样设计？** 因为统一的文件接口意味着你可以用相同的方式（读/写）来操作所有东西，极大简化了系统编程和管理。

### 1.3.3 没有 C 盘 D 盘！Linux 的目录树

这是新手最需要适应的一点：

**Windows** 的文件系统：
```
C:\
├── Users\
├── Program Files\
├── Windows\
D:\
├── 我的文件\
├── 工作\
```
每个磁盘分区有一个**盘符**（C:、D:、E:...），它们是**平行**的。

**Linux** 的文件系统：
```
/ (根目录，一切的起点)
├── home/
├── etc/
├── var/
├── usr/
├── dev/
├── tmp/
└── ...
```
所有东西都从一个**根目录 `/`** 开始，像一棵倒过来的树。即使有多个磁盘，也会被"挂载"到这棵树的某个位置（我们后面会讲挂载）。

---

## 1.4 文件系统结构详解

下面是 Linux 最重要的目录，请仔细阅读并理解每个目录的用途：

### 目录速览表

| 目录    | 英文全称              | 用途                             | 类比 Windows                       |
| ------- | --------------------- | -------------------------------- | ---------------------------------- |
| `/`     | Root                  | 根目录，所有目录的起点           | 相当于"我的电脑"                   |
| `/home` | Home                  | 普通用户的主目录                 | `C:\Users\`                        |
| `/root` | Root Home             | root 超级管理员的主目录          | `C:\Users\Administrator\`          |
| `/etc`  | Et cetera (配置)      | **系统配置文件**                 | 类似注册表 + 控制面板              |
| `/var`  | Variable              | 经常变化的数据（日志、缓存）     | `C:\Windows\Logs\`                 |
| `/usr`  | Unix System Resources | 用户安装的程序和库               | `C:\Program Files\`                |
| `/bin`  | Binaries              | 基本系统命令（`ls`, `cp` 等）    | `C:\Windows\System32\`             |
| `/sbin` | System Binaries       | 系统管理命令（需 root）          | 系统管理工具                       |
| `/tmp`  | Temporary             | 临时文件，重启后可能清空         | `C:\Users\XXX\AppData\Local\Temp\` |
| `/dev`  | Devices               | 设备文件                         | 设备管理器                         |
| `/proc` | Processes             | 虚拟文件系统，反映内核和进程状态 | 任务管理器（底层）                 |
| `/opt`  | Optional              | 第三方软件的安装目录             | `D:\Software\`                     |
| `/mnt`  | Mount                 | 临时挂载点                       | U盘 / 移动硬盘挂载位               |
| `/boot` | Boot                  | 启动引导文件（内核等）           | 启动分区                           |

### 重点目录详解

#### `/home` —— 你的"家"

```bash
/home/
├── alice/          # 用户 alice 的主目录
│   ├── Documents/
│   ├── Downloads/
│   ├── .bashrc     # 注意：以 . 开头的是隐藏文件
│   └── .ssh/       # SSH 密钥存放处
├── bob/            # 用户 bob 的主目录
│   └── ...
```

- 每个普通用户在 `/home/用户名/` 下有自己的空间
- 你在 WSL 中登录后，默认就在 `/home/你的用户名/`
- 可以用 `~` 来代替 `/home/你的用户名/`，比如 `cd ~` 就是回家
- **隐藏文件**以 `.` 开头，用 `ls -a` 才能看到

#### `/etc` —— 系统的"控制面板"

```bash
/etc/
├── hostname          # 主机名配置
├── hosts             # 域名解析配置（类似 Windows 的 hosts 文件）
├── passwd            # 用户账户信息
├── shadow            # 用户密码（加密后的）
├── group             # 用户组信息
├── apt/              # apt 包管理器的配置
│   └── sources.list  # 软件源地址
├── nginx/            # Nginx 服务器配置
│   └── nginx.conf
├── ssh/              # SSH 服务配置
│   └── sshd_config
└── crontab           # 定时任务配置
```

**核心理解**：在 Linux 中，几乎所有软件的配置都是通过编辑 `/etc/` 下的文本文件来完成的。不像 Windows 需要图形界面点来点去。

#### `/var` —— 变动数据

```bash
/var/
├── log/              # 📋 系统日志
│   ├── syslog        # 系统日志
│   ├── auth.log      # 登录认证日志
│   └── nginx/        # Nginx 的访问和错误日志
├── cache/            # 缓存文件
│   └── apt/          # apt 下载的软件包缓存
├── www/              # Web 服务器的网站文件（有些发行版用这个路径）
└── tmp/              # 持久化的临时文件
```

#### `/usr` —— 用户程序资源

```bash
/usr/
├── bin/              # 用户可执行程序（大多数命令在这里）
├── sbin/             # 系统管理程序
├── lib/              # 库文件(.so 文件，类似 Windows 的 .dll)
├── local/            # 用户手动编译安装的软件
│   ├── bin/
│   └── lib/
├── share/            # 共享资源（文档、图标等）
└── include/          # C/C++ 头文件
```

> 💡 **小知识**：`/usr` 和 `/usr/local` 的区别
> - `/usr/bin/` 下的程序是通过包管理器（`apt`）安装的
> - `/usr/local/bin/` 下的程序是你手动编译安装的
> 这样区分是为了避免手动安装的软件和系统软件互相冲突。

### 在 WSL 中实际查看

打开你的 WSL 终端，尝试运行以下命令，亲眼看看目录结构：

```bash
# 查看根目录下有哪些目录
ls /

# 查看你的家目录
ls ~

# 查看 /etc 下的配置文件
ls /etc

# 查看当前你在哪个目录
pwd
```

---

# 第二章：WSL2 环境配置

---

## 2.1 什么是 WSL？

### 2.1.1 WSL 的定义

**WSL**（Windows Subsystem for Linux）是微软官方提供的一项功能，让你可以在 Windows 系统上**原生运行 Linux 环境**，无需安装虚拟机或双系统。

### 2.1.2 WSL1 vs WSL2

| 特性             | WSL1                                       | WSL2                                        |
| ---------------- | ------------------------------------------ | ------------------------------------------- |
| **架构**         | 翻译层（把 Linux 调用翻译成 Windows 调用） | **完整的 Linux 内核**（运行在轻量虚拟机中） |
| **兼容性**       | 部分 Linux 程序不兼容                      | 完全兼容（真正的 Linux 内核）               |
| **文件系统性能** | 访问 Windows 文件快                        | 访问 Linux 文件快，访问 Windows 文件稍慢    |
| **内存占用**     | 较低                                       | 动态分配（起步约 300MB）                    |
| **Docker 支持**  | ❌                                          | ✅ 完美支持                                  |
| **推荐**         | 已过时                                     | ✅ **强烈推荐**                              |

> 💡 **WSL2 的本质**：它在 Windows 内部运行了一个极度精简的 Hyper-V 虚拟机，里面跑着一个真正的 Linux 内核。但它和传统虚拟机（VMware）不同的是，它启动极快（约 1-2 秒），内存动态分配，而且和 Windows 深度集成。

---

## 2.2 WSL2 安装与验证

### 2.2.1 系统要求

- **Windows 10**（版本 2004 及以上）或 **Windows 11**
- 已开启虚拟化（BIOS 中的 Intel VT-x 或 AMD-V）

检查虚拟化是否开启：
```
打开任务管理器 → 性能 → CPU → 右下角查看"虚拟化：已启用"
```

### 2.2.2 一键安装（推荐）

以 **管理员身份** 打开 PowerShell 或 Windows Terminal，运行：

```powershell
wsl --install
```

这一条命令会自动完成以下所有步骤：
1. 启用"适用于 Linux 的 Windows 子系统"功能
2. 启用"虚拟机平台"功能
3. 下载并安装 WSL2 Linux 内核
4. 将 WSL2 设为默认版本
5. 下载并安装 **Ubuntu**（默认发行版）

安装完成后，**重启电脑**。

### 2.2.3 首次启动配置

重启后，Ubuntu 会自动启动（或从开始菜单找到 Ubuntu 打开），要求你设置：

```
Enter new UNIX username: yourname      # 输入你想要的用户名（小写字母）
New password: ********                  # 输入密码（输入时不会显示任何字符，这是正常的！）
Retype new password: ********           # 再输入一遍确认
```

> ⚠️ **密码不显示是正常的**：Linux 终端输入密码时，屏幕上不会出现 `***` 或任何字符，这是安全设计。放心输入后按回车即可。

### 2.2.4 验证安装

```bash
# 查看 WSL 版本
wsl --version

# 查看已安装的 Linux 发行版
wsl --list --verbose
# 或简写
wsl -l -v
```

你应该看到类似输出：

```
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

`VERSION` 列显示 `2` 表示你正在使用 WSL2。✅

如果显示的是 `1`，可以强制切换：
```powershell
wsl --set-version Ubuntu 2
```

### 2.2.5 安装其他发行版（可选）

```powershell
# 查看可安装的发行版
wsl --list --online

# 安装指定发行版
wsl --install -d Debian
wsl --install -d kali-linux
```

---

## 2.3 WSL2 与 Windows 文件互访

这是 WSL 最强大的特性之一：**Linux 和 Windows 的文件系统可以互相访问！**

### 2.3.1 从 Linux 访问 Windows 文件

Windows 的所有磁盘都被自动挂载到 `/mnt/` 目录下：

```bash
# 访问 C 盘
ls /mnt/c/

# 访问 D 盘
ls /mnt/d/

# 访问 Windows 用户桌面
ls /mnt/c/Users/你的Windows用户名/Desktop/

# 复制 Windows 文件到 Linux 主目录
cp /mnt/c/Users/你的用户名/Documents/test.txt ~/
```

> ⚠️ **性能提示**：在 WSL2 中，操作 `/mnt/c/` 下的文件性能远不如操作 Linux 原生文件系统（`/home/你的用户名/`）。**建议把项目文件放在 Linux 文件系统中**，不要放在 `/mnt/c/` 下开发。

### 2.3.2 从 Windows 访问 Linux 文件

在 Windows 文件资源管理器的地址栏输入：

```
\\wsl$
```

或者（Windows 11 更新版本）：

```
\\wsl.localhost
```

你会看到你安装的所有 Linux 发行版。点击进入后，就可以像浏览普通文件夹一样浏览 Linux 的文件系统。

你也可以直接访问特定路径：
```
\\wsl$\Ubuntu\home\你的用户名\
```

另一个方便的方式——在 WSL 终端中，直接用命令打开当前目录的资源管理器：

```bash
# 在 Windows 资源管理器中打开当前 Linux 目录
explorer.exe .
```

> 💡 **小技巧**：你可以在 Windows 资源管理器的左侧快速访问中，把 `\\wsl$\Ubuntu\home\你的用户名` 固定为快捷方式，以后一键访问。

### 2.3.3 文件系统性能建议

| 操作场景                    | 推荐存放位置             | 原因                                    |
| --------------------------- | ------------------------ | --------------------------------------- |
| Linux 项目开发              | `/home/用户名/projects/` | Linux 原生文件系统性能最好              |
| 需要 Windows 程序打开的文件 | `C:\Users\...`           | Windows 程序无法直接读取 Linux 文件系统 |
| Windows ↔ Linux 共享文件    | 按需复制                 | 避免跨文件系统操作的性能损失            |

---

## 2.4 推荐工具配置

### 2.4.1 Windows Terminal（强烈推荐）

Windows Terminal 是微软官方的现代终端应用，支持多标签页、分屏、丰富的主题。

**安装方式**：
- Windows 11：已内置
- Windows 10：从 Microsoft Store 搜索 "Windows Terminal" 安装

**为什么推荐**：
- 一个窗口同时管理 PowerShell、CMD、WSL
- 支持标签页和分屏（`Alt + Shift + D` 分屏）
- 丰富的主题和字体配置
- 支持 GPU 加速渲染

**常用快捷键**：

| 快捷键                  | 功能                                        |
| ----------------------- | ------------------------------------------- |
| `Ctrl + Shift + T`      | 新建标签页                                  |
| `Ctrl + Shift + W`      | 关闭当前标签页                              |
| `Alt + Shift + D`       | 分屏                                        |
| `Ctrl + Shift + 数字`   | 打开指定类型的终端（如 PowerShell、Ubuntu） |
| `Ctrl + +` / `Ctrl + -` | 放大/缩小字体                               |
| `Ctrl + Shift + P`      | 命令面板                                    |

**设置默认打开 WSL**：
设置 → 启动 → 默认配置文件 → 选择 Ubuntu

### 2.4.2 VS Code + Remote-WSL 插件

这是在 WSL 环境下进行开发的**最佳方式**。

**安装步骤**：
1. 在 Windows 上安装 [VS Code](https://code.visualstudio.com/)
2. 在 VS Code 中安装扩展：`WSL`（扩展ID：`ms-vscode-remote.remote-wsl`）

**使用方式**：

```bash
# 在 WSL 终端中，进入你的项目目录
cd ~/my-project

# 用 VS Code 打开当前目录
code .
```

第一次运行 `code .` 时，WSL 会自动下载并安装 VS Code Server。之后 VS Code 窗口会在 Windows 上打开，但**它实际上连接的是 Linux 环境**。

**你可以验证**：看 VS Code 窗口左下角，会显示 `WSL: Ubuntu`，表示你正在 WSL 环境中工作。

**这意味着**：
- 终端（`Ctrl + ``）打开的是 Linux 终端
- 文件浏览器显示的是 Linux 文件系统
- 扩展运行在 Linux 环境中
- Git 用的是 Linux 上的 Git

> 💡 **最佳实践**：VS Code 的扩展需要**分别安装在 Windows 和 WSL 中**。打开 WSL 项目后，在扩展页面你会看到某些扩展旁边有 "Install in WSL" 的按钮，点击安装即可。

---

## 2.5 常用 WSL 管理命令

这些命令需要在 **Windows 的 PowerShell / CMD** 中执行（不是在 WSL Linux 终端中）：

### 基本管理

```powershell
# 查看所有已安装的 WSL 发行版及其状态
wsl --list --verbose
wsl -l -v

# 查看 WSL 版本信息
wsl --version

# 启动默认发行版
wsl

# 启动指定发行版
wsl -d Ubuntu

# 以指定用户身份启动（比如 root）
wsl -u root
```

### 关闭与重启

```powershell
# 关闭所有正在运行的 WSL 实例（释放内存）
wsl --shutdown

# 终止指定发行版
wsl --terminate Ubuntu
wsl -t Ubuntu
```

> 💡 **日常建议**：WSL2 会占用内存且不会自动释放。如果你发现电脑变慢了，可以运行 `wsl --shutdown` 来释放内存。

### 更新与维护

```powershell
# 更新 WSL 内核到最新版
wsl --update

# 检查 WSL 状态
wsl --status

# 查看可在线安装的发行版列表
wsl --list --online
```

### 导入导出（备份/迁移）

```powershell
# 导出一个发行版为 .tar 文件（备份）
wsl --export Ubuntu D:\backup\ubuntu-backup.tar

# 导入一个发行版（恢复/克隆）
wsl --import MyUbuntu D:\WSL\MyUbuntu D:\backup\ubuntu-backup.tar

# 注销一个发行版（删除）—— 谨慎操作！
wsl --unregister Ubuntu
```

### WSL 内存限制配置（进阶）

WSL2 默认会占用较多内存。你可以通过创建配置文件来限制：

在 Windows 用户目录下创建文件 `C:\Users\你的用户名\.wslconfig`：

```ini
[wsl2]
# 限制最大内存为 4GB
memory=4GB

# 限制 CPU 核心数
processors=2

# 限制 swap 大小
swap=2GB

# 关闭页面报告（减少磁盘占用）
pageReporting=false
```

修改后需要运行 `wsl --shutdown` 重启 WSL 才能生效。

---

# 第三章：命令行基础操作

---

## 3.1 开始之前：认识你的终端

### 3.1.1 终端提示符

打开 WSL 终端后，你会看到类似这样的提示符：

```
username@hostname:~$
```

拆解一下：

| 部分       | 含义                              |
| ---------- | --------------------------------- |
| `username` | 当前登录的用户名                  |
| `@`        | 分隔符（at 的意思）               |
| `hostname` | 计算机名                          |
| `:`        | 分隔符                            |
| `~`        | 当前所在目录（`~` 代表家目录）    |
| `$`        | 普通用户提示符（root 用户是 `#`） |

示例：
```
datovm@DESKTOP-ABC:~$          # 你在家目录
datovm@DESKTOP-ABC:/etc$       # 你在 /etc 目录
root@DESKTOP-ABC:/etc#          # 你是 root 用户，在 /etc 目录
```

### 3.1.2 命令的基本格式

```
命令 [选项] [参数]
```

- **命令**：要执行的操作（如 `ls`）
- **选项**：修改命令行为，通常以 `-`（短选项）或 `--`（长选项）开头
- **参数**：命令操作的对象（如文件名、目录名）

```bash
ls                     # 命令，无选项无参数
ls -l                  # 命令 + 短选项
ls -la                 # 命令 + 多个短选项合并
ls --all               # 命令 + 长选项（等同于 -a）
ls -l /home            # 命令 + 选项 + 参数
```

### 3.1.3 获取帮助的三种方式

```bash
# 方式1：--help 选项（最常用）
ls --help

# 方式2：man 手册（最详细）
man ls           # 按 q 退出，按 空格 翻页，按 / 搜索

# 方式3：简短说明
whatis ls        # 一句话描述命令用途
```

### 3.1.4 常用快捷键（节省大量时间）

| 快捷键     | 功能                        | 使用频率 |
| ---------- | --------------------------- | -------- |
| `Tab`      | **自动补全**文件名/命令名   | ⭐⭐⭐⭐⭐    |
| `Tab Tab`  | 显示所有可能的补全项        | ⭐⭐⭐⭐     |
| `↑` / `↓`  | 浏览历史命令                | ⭐⭐⭐⭐⭐    |
| `Ctrl + C` | **终止**当前正在运行的命令  | ⭐⭐⭐⭐⭐    |
| `Ctrl + L` | 清屏（等同于 `clear` 命令） | ⭐⭐⭐⭐     |
| `Ctrl + A` | 光标跳到行首                | ⭐⭐⭐      |
| `Ctrl + E` | 光标跳到行尾                | ⭐⭐⭐      |
| `Ctrl + U` | 删除光标前的所有内容        | ⭐⭐⭐      |
| `Ctrl + K` | 删除光标后的所有内容        | ⭐⭐       |
| `Ctrl + W` | 删除光标前一个单词          | ⭐⭐⭐      |
| `Ctrl + R` | **反向搜索**历史命令        | ⭐⭐⭐⭐     |
| `Ctrl + D` | 退出当前终端                | ⭐⭐       |

> 💡 **Tab 补全是你最好的朋友**：输入文件名或命令名的前几个字母，然后按 `Tab`，系统会自动补全。如果有多个匹配项，按两次 `Tab` 会列出所有候选项。**养成疯狂按 Tab 的习惯，会让你的效率提升数倍。**

---

## 3.2 文件与目录操作

### 3.2.1 `pwd` —— 我在哪里？

```bash
pwd    # Print Working Directory（打印当前工作目录）
```

输出示例：
```
/home/datovm
```

> 任何时候不确定自己在哪个目录，就输入 `pwd`。

### 3.2.2 `ls` —— 看看有什么

`ls`（List）是你最常用的命令之一。

```bash
# 基础用法
ls                # 列出当前目录的文件和文件夹
ls /etc           # 列出指定目录的内容
```

**常用选项**：

```bash
ls -l             # 长格式（详细信息：权限、大小、日期）
ls -a             # 显示所有文件（包括隐藏文件，以 . 开头的）
ls -la            # 两个选项合并使用（最常用的组合！）
ls -lh            # -h = human-readable，文件大小显示为 KB/MB/GB
ls -lt            # 按修改时间排序（最新的在前）
ls -ltr           # 按修改时间排序（最新的在后，-r = reverse）
ls -R             # 递归列出所有子目录内容
```

**`ls -l` 输出详解**：

```
-rw-r--r-- 1 datovm datovm  4096 Apr 13 10:30 test.txt
drwxr-xr-x 2 datovm datovm  4096 Apr 13 09:15 Documents
```

拆解每一列：

```
-rw-r--r--   1   datovm  datovm   4096   Apr 13 10:30   test.txt
│            │     │       │        │        │              │
│            │     │       │        │        │              └─ 文件名
│            │     │       │        │        └─ 最后修改时间
│            │     │       │        └─ 文件大小（字节）
│            │     │       └─ 所属组
│            │     └─ 所有者
│            └─ 硬链接数
└─ 文件类型和权限
   第1位：- 普通文件 / d 目录 / l 符号链接
   后9位：所有者权限(rwx) + 组权限(rwx) + 其他人权限(rwx)
```

### 3.2.3 `cd` —— 移动到别处

```bash
cd 目录名        # Change Directory（切换目录）
```

**各种路径写法**：

```bash
# 绝对路径（以 / 开头，从根目录算起）
cd /home/datovm/Documents
cd /etc/nginx

# 相对路径（从当前位置算起）
cd Documents              # 进入当前目录下的 Documents
cd ../                    # 回到上一级目录
cd ../../                 # 回到上两级目录
cd ./test                 # 进入当前目录下的 test（./ 代表当前目录）

# 特殊路径
cd ~                      # 回到家目录（/home/你的用户名）
cd                        # 同上，不带参数也是回家
cd -                      # 回到上一次所在的目录（在两个目录间切换很方便）
cd /                      # 去到根目录
```

> 💡 **绝对路径 vs 相对路径**：
> - **绝对路径**以 `/` 开头，像街道门牌号：`北京市朝阳区XX路100号`
> - **相对路径**不以 `/` 开头，像口头说路：`前面路口左转`
> - `..` 代表上一级目录，`.` 代表当前目录

**练习**：
```bash
cd ~                  # 确保在家目录
pwd                   # 输出：/home/datovm
cd /etc               # 去 /etc
pwd                   # 输出：/etc
cd -                  # 回到上次的位置
pwd                   # 输出：/home/datovm
```

### 3.2.4 `mkdir` —— 创建目录

```bash
mkdir 目录名          # Make Directory（创建目录）
```

```bash
# 创建单个目录
mkdir projects

# 创建多级目录（父目录不存在时，-p 会自动创建）
mkdir -p projects/web/frontend
# 这会一次性创建 projects → web → frontend 三级目录

# 同时创建多个目录
mkdir dir1 dir2 dir3

# 创建并设置权限
mkdir -m 755 my_public_dir
```

> ⚠️ **常见错误**：如果不加 `-p`，当父目录不存在时会报错：
> ```
> mkdir projects/web/frontend
> mkdir: cannot create directory 'projects/web/frontend': No such file or directory
> ```
> 加上 `-p` 就没问题了。**建议养成使用 `-p` 的习惯。**

### 3.2.5 `touch` —— 创建空文件

```bash
touch 文件名         # 创建空文件（如果文件已存在，则更新修改时间）
```

```bash
# 创建单个空文件
touch hello.txt

# 创建多个空文件
touch file1.txt file2.txt file3.txt

# 在指定目录创建文件
touch ~/projects/readme.md
```

### 3.2.6 `cp` —— 复制

```bash
cp 源文件 目标位置    # Copy（复制）
```

```bash
# 复制文件
cp hello.txt hello_backup.txt          # 复制并重命名
cp hello.txt ~/backup/                 # 复制到指定目录（保持原名）
cp hello.txt ~/backup/new_name.txt     # 复制到指定目录并重命名

# 复制目录（必须加 -r，递归复制）
cp -r projects/ projects_backup/

# 常用选项
cp -i hello.txt backup/               # -i = 覆盖前询问
cp -v hello.txt backup/               # -v = 显示复制过程（verbose）
cp -r -v src/ dest/                   # 递归复制目录并显示过程
```

> ⚠️ **记住**：复制目录时**必须**加 `-r`（recursive，递归），否则会报错。

### 3.2.7 `mv` —— 移动 / 重命名

```bash
mv 源 目标           # Move（移动 / 重命名）
```

`mv` 有两个功能：**移动**和**重命名**。

```bash
# 重命名文件
mv oldname.txt newname.txt

# 重命名目录
mv old_dir/ new_dir/

# 移动文件到另一个目录
mv hello.txt ~/Documents/

# 移动并重命名
mv hello.txt ~/Documents/greeting.txt

# 移动多个文件到一个目录
mv file1.txt file2.txt file3.txt ~/Documents/

# 覆盖前询问
mv -i hello.txt ~/Documents/
```

> 💡 **和 Windows 的区别**：在 Linux 中，"重命名"不是一个单独的操作，它就是 `mv`（把文件从"旧名字"移动到"新名字"）。

### 3.2.8 `rm` —— 删除

```bash
rm 文件名            # Remove（删除）
```

> ⚠️ **Linux 没有回收站！`rm` 删除的文件无法恢复！请务必小心！**

```bash
# 删除文件
rm hello.txt

# 删除前确认
rm -i hello.txt            # 会提示 "remove hello.txt? y/n"

# 删除目录（必须加 -r）
rm -r old_dir/

# 强制删除（不提示确认）
rm -f hello.txt

# 删除目录及其所有内容（最常用但也最危险）
rm -rf old_dir/

# 显示删除过程
rm -rv old_dir/
```

**⛔ 致命命令警告**：

```bash
# 永远不要运行这个命令！！！
rm -rf /                   # 删除整个系统中的所有文件！

# 也不要运行这个！
rm -rf /*                  # 同上，删除根目录下所有内容
```

> 💡 **安全建议**：
> 1. 删除前先用 `ls` 确认要删除的内容
> 2. 养成使用 `rm -i` 的习惯
> 3. 重要数据定期备份
> 4. 可以设置别名：`alias rm='rm -i'`（在 `.bashrc` 中添加，后面会学到）

### 3.2.9 综合练习

跟着做一遍，巩固所有文件操作命令：

```bash
# 1. 确保在家目录
cd ~

# 2. 创建练习环境
mkdir -p linux_practice/notes
mkdir -p linux_practice/backup

# 3. 创建一些文件
touch linux_practice/notes/day1.txt
touch linux_practice/notes/day2.txt
touch linux_practice/readme.md

# 4. 查看目录结构
ls -R linux_practice/

# 5. 复制文件
cp linux_practice/notes/day1.txt linux_practice/backup/

# 6. 重命名文件
mv linux_practice/readme.md linux_practice/README.md

# 7. 移动文件
mv linux_practice/backup/day1.txt linux_practice/

# 8. 查看最终结构
ls -lR linux_practice/

# 9. 清理（练习完后可选）
rm -rf linux_practice/
```

---

## 3.3 查看文件内容

### 3.3.1 `cat` —— 一次性显示全部内容

```bash
cat 文件名           # Concatenate（连接/显示）
```

```bash
# 查看文件内容
cat /etc/hostname

# 显示行号
cat -n /etc/hosts

# 查看多个文件（按顺序显示）
cat file1.txt file2.txt

# 合并多个文件到一个新文件
cat file1.txt file2.txt > combined.txt
```

> 💡 **适用场景**：内容较短的文件（几十行以内）。如果文件有几百上千行，用 `cat` 会刷屏，应该用 `less`。

**实际练习**：
```bash
# 查看系统的主机名
cat /etc/hostname

# 查看本机的 hosts 解析文件（带行号）
cat -n /etc/hosts

# 查看当前系统支持的 shell
cat /etc/shells
```

### 3.3.2 `less` —— 分页查看（推荐）

```bash
less 文件名          # 分页浏览器
```

`less` 打开文件后，你可以上下翻页、搜索，非常方便：

| 按键          | 功能                                          |
| ------------- | --------------------------------------------- |
| `空格` 或 `f` | 向下翻一页                                    |
| `b`           | 向上翻一页                                    |
| `↑` `↓`       | 上下滚动一行                                  |
| `g`           | 跳到文件开头                                  |
| `G`           | 跳到文件末尾                                  |
| `/关键词`     | 向下搜索（按 `n` 跳到下一个，`N` 跳到上一个） |
| `?关键词`     | 向上搜索                                      |
| `q`           | 退出                                          |

```bash
# 查看系统日志
less /var/log/syslog

# 也常配合管道使用（后面会讲管道）
ls -la /etc | less
```

> 💡 **`less` vs `more`**：`more` 是更早期的分页工具，只能向下翻，功能较少。`less` 功能更强大（能向上翻、能搜索），记口诀：**"less is more"**（less 比 more 强）。

### 3.3.3 `head` —— 查看开头

```bash
head 文件名          # 默认显示前 10 行
```

```bash
# 显示前 10 行
head /etc/passwd

# 显示前 5 行
head -n 5 /etc/passwd
head -5 /etc/passwd            # 简写

# 显示前 100 个字节
head -c 100 /etc/passwd
```

### 3.3.4 `tail` —— 查看结尾

```bash
tail 文件名          # 默认显示后 10 行
```

```bash
# 显示后 10 行
tail /var/log/syslog

# 显示后 20 行
tail -n 20 /var/log/syslog
tail -20 /var/log/syslog       # 简写

# ⭐ 实时追踪文件变化（非常常用！监控日志）
tail -f /var/log/syslog
# 按 Ctrl + C 停止追踪
```

> 💡 **`tail -f` 是运维必备技能**：当你启动了一个服务后，想实时看它的日志输出，就用 `tail -f`。-f 的意思是 follow（跟踪）。文件有新内容写入时，会自动显示在屏幕上。

### 3.3.5 其他查看方式

```bash
# wc —— 统计行数、单词数、字节数
wc /etc/passwd                # 输出：行数 单词数 字节数
wc -l /etc/passwd             # 只统计行数

# file —— 查看文件类型
file /etc/passwd              # 输出：ASCII text
file /bin/ls                  # 输出：ELF 64-bit executable

# stat —— 查看文件详细元数据（大小、权限、修改时间等）
stat /etc/passwd
```

---

## 3.4 搜索与过滤

### 3.4.1 `find` —— 查找文件

`find` 是 Linux 中最强大的文件搜索命令。

```bash
find 搜索路径 搜索条件 [动作]
```

**按名称搜索**：
```bash
# 在当前目录及子目录中，查找名为 test.txt 的文件
find . -name "test.txt"

# 忽略大小写查找
find . -iname "test.txt"

# 使用通配符（注意要加引号）
find /home -name "*.txt"           # 查找所有 .txt 文件
find /home -name "day*"            # 查找所有以 day 开头的文件
find /etc -name "*.conf"           # 查找所有配置文件
```

**按类型搜索**：
```bash
# 只找文件（-type f）
find /home -type f -name "*.log"

# 只找目录（-type d）
find /home -type d -name "projects"

# 只找符号链接（-type l）
find /usr -type l
```

**按大小搜索**：
```bash
# 查找大于 100MB 的文件
find / -type f -size +100M

# 查找小于 1KB 的文件
find . -type f -size -1k

# 查找正好 4096 字节的文件
find . -type f -size 4096c        # c = 字节
```

**按时间搜索**：
```bash
# 查找最近 7 天内修改过的文件
find /home -type f -mtime -7

# 查找超过 30 天未修改的文件
find /home -type f -mtime +30

# 查找最近 60 分钟内修改的文件
find /home -type f -mmin -60
```

**找到后执行操作**：
```bash
# 找到 .log 文件并显示详细信息
find /var/log -name "*.log" -exec ls -lh {} \;

# 找到 .tmp 文件并删除（谨慎！）
find /tmp -name "*.tmp" -exec rm {} \;

# 更安全的写法：删除前确认
find /tmp -name "*.tmp" -ok rm {} \;
```

> 💡 **`{}` 和 `\;` 的含义**：
> - `{}` 是一个占位符，代表 find 找到的每一个文件
> - `\;` 表示 `-exec` 命令的结束
> - 整句意思是：对找到的每个文件执行后面的命令

### 3.4.2 `grep` —— 搜索文件内容

如果说 `find` 是"找文件"，那 `grep` 就是"在文件里找内容"。

```bash
grep "要搜索的内容" 文件名
```

**基础用法**：
```bash
# 在 /etc/passwd 中搜索包含 "root" 的行
grep "root" /etc/passwd

# 忽略大小写搜索
grep -i "ROOT" /etc/passwd

# 显示行号
grep -n "root" /etc/passwd

# 显示匹配行的前后 2 行上下文
grep -C 2 "root" /etc/passwd

# 显示不匹配的行（反向筛选）
grep -v "nologin" /etc/passwd         # 排除包含 nologin 的行
```

**递归搜索目录中的所有文件**：
```bash
# 在 /etc 目录及子目录中，搜索包含 "listen" 的所有文件
grep -r "listen" /etc/

# 递归搜索 + 显示行号 + 忽略大小写
grep -rni "port" /etc/nginx/

# 只显示文件名（不显示匹配内容）
grep -rl "password" /etc/
```

**使用正则表达式**：
```bash
# 搜索以 root 开头的行
grep "^root" /etc/passwd

# 搜索以 bash 结尾的行
grep "bash$" /etc/passwd

# 搜索包含数字的行
grep "[0-9]" /etc/passwd

# 使用扩展正则（-E）
grep -E "root|daemon" /etc/passwd     # 搜索 root 或 daemon
```

**实用示例**：
```bash
# 查看系统中所有可以登录的用户
grep "bash" /etc/passwd

# 查看某个服务是否在运行
ps aux | grep nginx                    # 配合管道使用

# 在代码目录中搜索 TODO 注释
grep -rn "TODO" ~/projects/
```

### 3.4.3 `|` 管道符 —— 命令之间的"接力棒"

**管道是 Linux 最精华的设计之一！**

管道符 `|` 的作用是：**把前一个命令的输出，作为后一个命令的输入**。

```
命令1 的输出 ——→ | ——→ 命令2 的输入
```

**示例**：

```bash
# 示例1：查看 /etc 目录下有多少个文件
ls /etc | wc -l
# ls /etc 的输出（文件列表）→ 作为 wc -l 的输入 → 统计行数

# 示例2：查看系统进程中是否有 nginx
ps aux | grep nginx
# ps aux 输出所有进程 → grep 从中筛选包含 nginx 的行

# 示例3：查看最占内存的前 10 个进程
ps aux --sort=-%mem | head -10
# ps aux 列出所有进程并按内存排序 → head 只取前 10 行

# 示例4：在大量输出中分页查看
ls -la /etc | less
# ls 的输出 → 通过 less 分页浏览

# 示例5：多级管道（可以连续使用多个 |）
cat /etc/passwd | grep "home" | sort | head -5
# 查看 passwd → 筛选含 "home" 的行 → 排序 → 取前 5 行
```

### 3.4.4 输出重定向

除了管道，重定向也是命令行的核心概念：

```bash
# > 将输出写入文件（覆盖原内容）
echo "Hello Linux" > hello.txt
ls -la > file_list.txt

# >> 将输出追加到文件末尾（不覆盖）
echo "Another line" >> hello.txt
date >> log.txt                        # 把当前时间追加到日志文件

# 2> 将错误输出重定向到文件
find / -name "test" 2> errors.txt      # 错误信息写入 errors.txt

# &> 将标准输出和错误输出都重定向
find / -name "test" &> all_output.txt

# /dev/null —— "黑洞"，丢弃所有输出
find / -name "test" 2>/dev/null        # 丢弃错误信息，只看正常结果
```

> 💡 **`>` vs `>>`**：
> - `>` **覆盖**：如果文件已存在，原内容全部丢失！
> - `>>` **追加**：在文件末尾添加新内容
> - **记忆方式**：一个箭头"冲进去覆盖"，两个箭头"排队追加"

**实用组合**：

```bash
# 把命令的执行记录保存到文件
ls -la /etc > ~/etc_files.txt 2>&1     # 正常输出和错误都保存

# 同时在屏幕显示和保存到文件
ls -la | tee file_list.txt             # tee 命令：一份输出两份用
ls -la | tee -a file_list.txt          # -a = append，追加模式
```

### 3.4.5 综合练习

```bash
# 1. 创建练习环境
mkdir -p ~/search_practice
cd ~/search_practice
echo "Hello World" > file1.txt
echo "hello linux" > file2.txt
echo "HELLO GREP" > file3.txt
echo "goodbye" > file4.txt
mkdir -p subdir
echo "nested hello" > subdir/file5.txt

# 2. 用 find 查找所有 .txt 文件
find . -name "*.txt"

# 3. 用 grep 搜索包含 hello 的文件（不区分大小写）
grep -ri "hello" .

# 4. 用管道统计包含 hello 的文件有多少个
grep -ri "hello" . | wc -l

# 5. 把搜索结果保存到文件
grep -ri "hello" . > search_results.txt
cat search_results.txt

# 6. 清理
cd ~
rm -rf ~/search_practice
```

---

## 3.5 补充：一些好用的小命令

| 命令       | 功能               | 示例                                                   |
| ---------- | ------------------ | ------------------------------------------------------ |
| `echo`     | 输出文本           | `echo "Hello"`                                         |
| `date`     | 显示日期时间       | `date "+%Y-%m-%d %H:%M:%S"`                            |
| `whoami`   | 显示当前用户       | `whoami`                                               |
| `hostname` | 显示主机名         | `hostname`                                             |
| `uname -a` | 显示系统信息       | `uname -a`                                             |
| `clear`    | 清屏               | `clear`（或 Ctrl+L）                                   |
| `history`  | 查看历史命令       | `history`，`history 20`                                |
| `which`    | 查看命令的位置     | `which python3`                                        |
| `alias`    | 创建命令别名       | `alias ll='ls -la'`                                    |
| `tree`     | 以树形显示目录结构 | `tree ~/projects`（需先安装：`sudo apt install tree`） |
| `sort`     | 排序               | `sort file.txt`                                        |
| `uniq`     | 去重（需先排序）   | `sort file.txt \| uniq`                                |
| `cut`      | 按列截取           | `cut -d: -f1 /etc/passwd`（取第1列）                   |

**`tree` 特别推荐**——直观展示目录结构：

```bash
# 先安装
sudo apt install tree -y

# 使用
tree ~/                    # 显示家目录树形结构
tree -L 2 /etc             # 只显示 2 层深度
tree -d ~/projects         # 只显示目录，不显示文件
```

---

## 第三章小结

恭喜你！学完第三章，你已经掌握了 Linux 命令行的核心操作：

| 技能             | 掌握的命令                       |
| ---------------- | -------------------------------- |
| 定位自己         | `pwd`                            |
| 查看内容         | `ls`（各种选项）                 |
| 移动位置         | `cd`（绝对/相对路径）            |
| 创建文件/目录    | `mkdir -p`、`touch`              |
| 复制/移动/重命名 | `cp -r`、`mv`                    |
| 删除             | `rm -rf`（小心使用！）           |
| 查看文件         | `cat`、`less`、`head`、`tail -f` |
| 搜索文件         | `find`                           |
| 搜索内容         | `grep -rni`                      |
| 管道与重定向     | `\|`、`>`、`>>`、`tee`           |

> 📝 **学习建议**：不要死记命令！打开 WSL 终端，把上面的每个命令都**亲手敲一遍**。命令行操作就像骑自行车，多练几次就形成肌肉记忆了。遇到不确定的命令，随时用 `命令 --help` 查看帮助。

---

# Linux 操作入门教程（WSL2 版）— 续篇

> 接上篇第1-3章，本篇覆盖第4-6章：用户与权限管理、文件编辑、软件包管理。

---

# 第四章：用户与权限管理

---

## 4.1 理解 Linux 的用户体系

### 4.1.1 为什么需要用户管理？

想象一下：一台 Linux 服务器上可能有多个人同时使用（开发者、运维、数据库管理员……）。如果所有人都能看到、修改所有文件，那：
- 小明不小心删了数据库配置 → 网站挂了
- 实习生看到了生产环境密码 → 安全隐患
- 客户上传的文件被其他用户覆盖 → 数据丢失

所以 Linux 设计了一套**严格的用户与权限体系**，实现"**谁能做什么、对哪些文件能做什么**"的精细控制。

### 4.1.2 三种用户角色

| 用户类型       | 用户名                 | UID   | 权限级别                             | 类比                     |
| -------------- | ---------------------- | ----- | ------------------------------------ | ------------------------ |
| **超级管理员** | `root`                 | 0     | 拥有系统的**一切权限**，可以做任何事 | Windows 的 Administrator |
| **普通用户**   | 你创建的用户名         | 1000+ | 只能操作自己的文件，受权限限制       | Windows 的标准用户       |
| **系统用户**   | `www-data`、`mysql` 等 | 1-999 | 为服务程序创建的专用用户，不能登录   | Windows 的服务账户       |

**查看当前用户信息**：

```bash
# 我是谁？
whoami
# 输出：datovm

# 我的详细身份信息
id
# 输出：uid=1000(datovm) gid=1000(datovm) groups=1000(datovm),4(adm),27(sudo)

# 拆解上面的输出：
# uid=1000(datovm)      → 用户 ID 是 1000，用户名是 datovm
# gid=1000(datovm)      → 主组 ID 是 1000，组名是 datovm
# groups=...,27(sudo)   → 还属于 sudo 组（可以使用 sudo 命令）
```

### 4.1.3 用户组

**用户组（Group）** 是把多个用户归类管理的机制。

```
假设场景：一个公司的 Linux 服务器

开发组 (developers)：   alice, bob, charlie
运维组 (ops)：          dave, eve
数据库组 (dba)：        frank

项目代码目录的权限：
- 所有者：alice（项目负责人）
- 所属组：developers（整个开发组可以读写）
- 其他人：只读
```

每个用户都有：
- **一个主组**（创建用户时自动创建，通常与用户名同名）
- **零个或多个附加组**（通过 `usermod` 添加）

```bash
# 查看当前用户属于哪些组
groups
# 输出：datovm adm sudo

# 查看指定用户属于哪些组
groups root
```

### 4.1.4 用户信息存储在哪里？

Linux 用**文本文件**存储用户信息（又是"一切皆文件"的体现）：

```bash
# /etc/passwd — 用户账户信息（所有人可读）
cat /etc/passwd
```

每一行代表一个用户，用 `:` 分隔 7 个字段：

```
datovm:x:1000:1000:,,,:/home/datovm:/bin/bash
│      │ │    │    │    │            │
│      │ │    │    │    │            └─ 登录后使用的 Shell
│      │ │    │    │    └─ 家目录
│      │ │    │    └─ 备注信息（全名等）
│      │ │    └─ 主组 GID
│      │ └─ 用户 UID
│      └─ 密码占位符（实际密码在 /etc/shadow 中）
└─ 用户名
```

```bash
# /etc/shadow — 加密后的密码（只有 root 可读）
sudo cat /etc/shadow

# /etc/group — 用户组信息
cat /etc/group
```

---

## 4.2 `root` 与 `sudo`

### 4.2.1 root 是什么？

`root` 是 Linux 中的**超级管理员**，UID 为 0，拥有**无限权力**：
- 可以读写删除**系统中的任何文件**
- 可以安装/卸载软件
- 可以添加/删除用户
- 可以修改系统配置
- 可以终止任何进程

**但日常操作中，你不应该直接用 root 用户！** 原因：
- 一个手误就可能摧毁整个系统（没有"你确定吗？"的防护）
- 安全风险：如果攻击者获取了 root 权限，后果不堪设想
- 无法追踪是谁执行了什么操作（审计困难）

### 4.2.2 `sudo` —— 临时借用超级权力

`sudo`（**S**uper **U**ser **DO**）让普通用户**临时以 root 身份执行一条命令**：

```bash
# 普通用户尝试查看 shadow 文件 → 权限不足
cat /etc/shadow
# 输出：cat: /etc/shadow: Permission denied

# 使用 sudo → 输入自己的密码后获得临时 root 权限
sudo cat /etc/shadow
# [sudo] password for datovm: ********（输入你自己的密码）
# 成功显示内容！
```

**sudo 的工作流程**：

```
普通用户执行 sudo 命令
    ↓
系统检查：该用户在 /etc/sudoers 文件中吗？
    ↓
是 → 要求输入用户自己的密码（不是 root 密码）
    ↓
密码正确 → 以 root 身份执行命令
    ↓
15 分钟内再次使用 sudo 不需要重新输入密码
```

**常用 sudo 操作**：

```bash
# 以 root 身份执行单条命令
sudo apt update

# 以 root 身份编辑文件
sudo nano /etc/hosts

# 切换为 root 用户（进入 root 的 shell）
sudo su -
# 注意：提示符从 $ 变成 #
# 执行完操作后，输入 exit 退出 root shell

# 以 root 身份打开一个新的 shell（另一种方式）
sudo -i
# 输入 exit 退出

# 以其他用户身份执行命令
sudo -u www-data whoami
# 输出：www-data

# 查看 sudo 权限缓存是否有效
sudo -v

# 清除 sudo 密码缓存（下次 sudo 需要重新输入密码）
sudo -k
```

> 💡 **最佳实践**：
> - 平时用普通用户操作
> - 需要管理员权限时用 `sudo` 加在命令前面
> - 不要长时间停留在 `sudo su` 的 root shell 中
> - WSL 中你的默认用户已经在 sudo 组中，可以直接使用 sudo

### 4.2.3 `sudo` 常见报错与解决

**错误1：用户不在 sudoers 文件中**

```
datovm is not in the sudoers file. This incident will be reported.
```

解决：需要用 root 用户把该用户加入 sudo 组：
```bash
# 用 root 执行（在 WSL 中可以用 wsl -u root 登录）
usermod -aG sudo datovm
```

**错误2：忘记加 sudo**

```bash
apt update
# 输出：E: Could not open lock file... Permission denied

# 正确做法：
sudo apt update
```

> 💡 **小技巧**：如果你执行了一条命令忘记加 `sudo`，不需要重新输入整条命令，只需要：
> ```bash
> sudo !!
> ```
> `!!` 代表"上一条命令"，`sudo !!` = 用 sudo 重新执行上一条命令。**非常实用！**

---

## 4.3 权限模型详解

### 4.3.1 权限的三个维度

Linux 对每个文件/目录设置了三组权限，分别针对三类人：

```
-rwxr-xr--
 │││ │││ │││
 │││ │││ └┴┴─ 其他人（Others）的权限
 │││ └┴┴───── 所属组（Group）的权限
 └┴┴───────── 所有者（Owner/User）的权限
```

### 4.3.2 三种权限类型

| 符号 | 含义            | 对文件的作用          | 对目录的作用               |
| ---- | --------------- | --------------------- | -------------------------- |
| `r`  | 读（Read）      | 查看文件内容          | 列出目录中的文件名（`ls`） |
| `w`  | 写（Write）     | 修改文件内容          | 在目录中创建/删除文件      |
| `x`  | 执行（Execute） | 运行文件（脚本/程序） | 进入该目录（`cd`）         |
| `-`  | 无权限          | 不能进行对应操作      | 不能进行对应操作           |

> ⚠️ **目录的权限比较特殊**，新手容易搞混：
> - 目录的 `r` 权限：允许你 `ls` 这个目录（看到里面有什么文件）
> - 目录的 `x` 权限：允许你 `cd` 进入这个目录
> - 目录的 `w` 权限：允许你在这个目录中创建/删除文件
> - **你可能有 `x` 但没有 `r`**：可以 `cd` 进去，但无法 `ls` 查看内容（需要知道确切文件名才能操作）

### 4.3.3 权限的数字表示法

每种权限对应一个数字：

```
r = 4    （读）
w = 2    （写）
x = 1    （执行）
- = 0    （无权限）
```

把三个权限的数字**相加**得到一个数字，代表一组人的权限：

```
rwx = 4 + 2 + 1 = 7  （可读、可写、可执行）
rw- = 4 + 2 + 0 = 6  （可读、可写、不可执行）
r-x = 4 + 0 + 1 = 5  （可读、不可写、可执行）
r-- = 4 + 0 + 0 = 4  （只读）
--- = 0 + 0 + 0 = 0  （无任何权限）
```

三组人的权限连起来写，就是三位数：

```
755 = rwxr-xr-x
│││
│││
│└┴─ 其他人：r-x = 5（可读可执行）
│└── 所属组：r-x = 5（可读可执行）
└─── 所有者：rwx = 7（可读可写可执行）
```

**常见权限组合**：

| 数字  | 符号表示     | 含义                             | 典型用途                     |
| ----- | ------------ | -------------------------------- | ---------------------------- |
| `755` | `rwxr-xr-x`  | 所有者完全控制，其他人可读可执行 | 目录、可执行脚本             |
| `644` | `rw-r--r--`  | 所有者可读写，其他人只读         | 普通文件、配置文件           |
| `700` | `rwx------`  | 只有所有者可以操作               | 私密目录（如 `.ssh`）        |
| `600` | `rw-------`  | 只有所有者可读写                 | SSH 私钥文件                 |
| `777` | `rwxrwxrwx`  | 所有人完全控制                   | ⚠️ **不安全，尽量避免**       |
| `000` | `----------` | 所有人无权限                     | 彻底锁死（只有 root 可操作） |

### 4.3.4 查看文件权限

```bash
ls -l
```

输出示例：
```
drwxr-xr-x 2 datovm datovm 4096 Apr 13 10:00 Documents
-rw-r--r-- 1 datovm datovm  256 Apr 13 10:30 notes.txt
-rwxr-xr-x 1 datovm datovm 1024 Apr 13 11:00 backup.sh
lrwxrwxrwx 1 datovm datovm   15 Apr 13 11:30 link -> /tmp/test
```

拆解第一列：

```
d rwx r-x r-x
│ └┬┘ └┬┘ └┬┘
│  │   │   └── 其他人权限
│  │   └────── 所属组权限
│  └────────── 所有者权限
└───────────── 文件类型
               d = 目录
               - = 普通文件
               l = 符号链接（快捷方式）
```

---

## 4.4 `chmod` —— 修改权限

```bash
chmod 权限 文件名     # Change Mode（修改文件权限）
```

### 4.4.1 数字方式（推荐，最常用）

```bash
# 设置 backup.sh 的权限为 755（所有者可读写执行，其他人可读可执行）
chmod 755 backup.sh

# 设置 config.ini 只有所有者可读写
chmod 600 config.ini

# 设置目录权限（递归，包括目录下所有文件）
chmod -R 755 my_project/

# 设置 .ssh 目录的安全权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa            # SSH 私钥必须是 600！
```

### 4.4.2 符号方式

格式：`chmod [谁][操作][权限] 文件`

- **谁**：`u`（user 所有者）、`g`（group 组）、`o`（others 其他人）、`a`（all 所有人）
- **操作**：`+`（添加）、`-`（移除）、`=`（设为）
- **权限**：`r`、`w`、`x`

```bash
# 给所有者添加执行权限
chmod u+x script.sh

# 给文件的所有者添加执行权限，移除其他人的写权限
chmod u+x,o-w file.txt

# 给所有人添加读权限
chmod a+r document.txt

# 移除组和其他人的写权限
chmod go-w config.ini

# 设定：所有者可读写执行，组可读执行，其他人无权限
chmod u=rwx,g=rx,o= script.sh
```

### 4.4.3 实际场景示例

**场景1：让脚本可以运行**

```bash
# 你写了一个 Shell 脚本
echo '#!/bin/bash' > hello.sh
echo 'echo "Hello from script!"' >> hello.sh

# 尝试运行 → 报错：权限不足
./hello.sh
# 输出：bash: ./hello.sh: Permission denied

# 查看权限——没有 x（执行）
ls -l hello.sh
# -rw-r--r-- 1 datovm datovm 40 Apr 13 12:00 hello.sh

# 添加执行权限
chmod +x hello.sh
# 或者更精确地
chmod 755 hello.sh

# 再次运行 → 成功！
./hello.sh
# 输出：Hello from script!
```

**场景2：保护敏感文件**

```bash
# 创建一个包含密码的文件
echo "my_secret_password_123" > secret.txt

# 默认权限是 644（其他人可读！不安全！）
ls -l secret.txt
# -rw-r--r-- 1 datovm datovm 24 ...

# 改为只有所有者可读写
chmod 600 secret.txt
ls -l secret.txt
# -rw------- 1 datovm datovm 24 ...
# 现在其他用户无法读取这个文件了 ✅
```

---

## 4.5 `chown` —— 修改文件所有者和组

```bash
chown 新所有者:新组 文件名     # Change Owner（修改所有者）
```

> ⚠️ **修改文件所有者需要 root 权限**，所以需要加 `sudo`。

```bash
# 修改文件的所有者
sudo chown alice file.txt

# 修改文件的所有者和所属组
sudo chown alice:developers file.txt

# 只修改所属组（冒号前留空）
sudo chown :developers file.txt

# 也可以用 chgrp 单独修改组
sudo chgrp developers file.txt

# 递归修改目录及其下所有文件
sudo chown -R www-data:www-data /var/www/html/
```

**典型应用场景**：

```bash
# 部署网站时，把网站文件的所有者改为 Web 服务器用户
sudo chown -R www-data:www-data /var/www/mysite/

# 把项目目录的组改为开发组
sudo chown -R :developers /opt/project/
sudo chmod -R 775 /opt/project/    # 组内成员都可以读写
```

---

## 4.6 多用户管理

### 4.6.1 添加用户

```bash
# 方式1：useradd（基础命令，需要手动配置）
sudo useradd alice

# 方式2：adduser（交互式，更友好，推荐！）
sudo adduser alice
```

`adduser` 会引导你完成所有设置：

```
Adding user `alice' ...
Adding new group `alice' (1001) ...
Adding new user `alice' (1001) with group `alice' ...
Creating home directory `/home/alice' ...
Copying files from `/etc/skel' ...
New password:                          # 设置密码
Retype new password:                   # 确认密码
Full Name []: Alice Wang               # 填写全名（可选，直接回车跳过）
Room Number []:                         # 回车跳过
Work Phone []:                          # 回车跳过
Home Phone []:                          # 回车跳过
Other []:                               # 回车跳过
Is the information correct? [Y/n] Y    # 确认
```

**`useradd` vs `adduser` 对比**：

| 特性         | `useradd`                | `adduser`               |
| ------------ | ------------------------ | ----------------------- |
| 类型         | 底层命令                 | 上层友好脚本            |
| 创建家目录   | 需要手动加 `-m` 选项     | 自动创建                |
| 设置密码     | 需要另外用 `passwd` 命令 | 交互式引导设置          |
| 复制初始文件 | 需手动指定               | 自动从 `/etc/skel` 复制 |
| 推荐度       | 适合脚本自动化           | ✅ 日常推荐              |

如果用 `useradd`，完整的操作是：
```bash
sudo useradd -m -s /bin/bash alice    # -m 创建家目录, -s 指定 shell
sudo passwd alice                      # 单独设置密码
```

### 4.6.2 查看用户信息

```bash
# 查看所有用户（每行一个用户）
cat /etc/passwd

# 查看所有普通用户（UID >= 1000 的）
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# 查看某个用户的详细信息
id alice

# 查看当前谁在登录
who
w              # 更详细，显示在做什么

# 查看用户的登录历史
last alice
```

### 4.6.3 修改用户

```bash
# 修改用户的 Shell
sudo usermod -s /bin/zsh alice

# 把用户添加到附加组（-a = append, -G = 附加组）
sudo usermod -aG sudo alice          # 给 alice sudo 权限
sudo usermod -aG docker alice        # 让 alice 可以使用 Docker

# 修改用户名
sudo usermod -l new_alice alice

# 锁定用户（禁止登录）
sudo usermod -L alice

# 解锁用户
sudo usermod -U alice

# 修改密码
sudo passwd alice
```

> ⚠️ **`-aG` 中的 `-a` 非常重要！** 如果不加 `-a`，用户会被从所有已有的附加组中移除，只保留你指定的组。这是一个常见的新手错误，后果很严重。

### 4.6.4 删除用户

```bash
# 删除用户（保留家目录）
sudo userdel alice

# 删除用户并删除其家目录（彻底清除）
sudo userdel -r alice

# 更友好的交互式方式
sudo deluser alice
sudo deluser --remove-home alice
```

### 4.6.5 用户组管理

```bash
# 创建新组
sudo groupadd developers

# 查看所有组
cat /etc/group

# 把用户加入组
sudo usermod -aG developers alice
sudo usermod -aG developers bob

# 从组中移除用户
sudo gpasswd -d alice developers

# 删除组
sudo groupdel developers

# 查看某个组有哪些成员
getent group developers
```

### 4.6.6 切换用户

```bash
# 切换到其他用户
su - alice
# 输入 alice 的密码
# 注意：- 号很重要，它会加载目标用户的完整环境

# 切换到 root
su -
# 输入 root 密码（WSL 中 root 默认可能没设密码，用 sudo su 代替）

# 用 sudo 切换（输入自己的密码即可）
sudo su - alice
sudo su -               # 切换到 root

# 退出当前用户，回到原来的用户
exit
```

### 4.6.7 综合练习

```bash
# 1. 创建一个新用户 testuser
sudo adduser testuser

# 2. 查看用户信息
id testuser

# 3. 创建一个用户组
sudo groupadd testgroup

# 4. 把用户加入组
sudo usermod -aG testgroup testuser

# 5. 验证
groups testuser
# 输出应包含：testuser testgroup

# 6. 切换到新用户
su - testuser
whoami
pwd
# 输出：testuser 和 /home/testuser

# 7. 退出回到你的用户
exit

# 8. 创建一个文件，修改权限做实验
touch ~/permission_test.txt
ls -l ~/permission_test.txt

# 改为只有自己可读写
chmod 600 ~/permission_test.txt
ls -l ~/permission_test.txt

# 改为所有人可读
chmod 644 ~/permission_test.txt

# 9. 清理：删除测试用户和组
sudo userdel -r testuser
sudo groupdel testgroup
```

---

# 第五章：文件编辑

---

## 5.1 编辑器概述

在 Linux 中，**修改配置 = 编辑文本文件**。所以掌握至少一个文本编辑器是必备技能。

| 编辑器      | 学习难度  | 功能强大度 | 特点                                     |
| ----------- | --------- | ---------- | ---------------------------------------- |
| **Nano**    | ⭐ 极简    | ⭐⭐         | 新手友好，界面底部有快捷键提示           |
| **Vim**     | ⭐⭐⭐⭐ 较难 | ⭐⭐⭐⭐⭐      | 效率极高，几乎所有 Linux 系统都预装      |
| **VS Code** | ⭐⭐ 简单   | ⭐⭐⭐⭐⭐      | 图形界面编辑器，通过 Remote-WSL 插件使用 |

> 💡 **学习建议**：先学 Nano 应急，再学 Vim 的基本操作（至少会保存退出），日常开发用 VS Code。但 **Vim 的基本操作必须掌握**，因为很多时候你只有一个终端（比如 SSH 连接远程服务器），Vim 是你唯一的选择。

---

## 5.2 Nano 编辑器 —— 新手友好

### 5.2.1 打开/创建文件

```bash
# 打开已有文件
nano filename.txt

# 打开并定位到特定行
nano +10 filename.txt          # 直接跳到第 10 行

# 如果文件不存在，nano 会创建一个新文件
nano new_file.txt

# 编辑系统配置文件（需要 sudo）
sudo nano /etc/hosts
```

### 5.2.2 界面说明

打开 nano 后，你会看到：

```
  GNU nano 6.2                  filename.txt

这里是文件内容区域
你可以直接输入文字
就像在记事本里打字一样



^G Help    ^O Write Out  ^W Where Is  ^K Cut       ^U Paste
^X Exit    ^R Read File  ^\ Replace   ^J Justify   ^T Spell
```

**底部两行是快捷键提示**：
- `^` 代表 `Ctrl` 键
- `^O` = `Ctrl + O`（保存文件）
- `^X` = `Ctrl + X`（退出）

### 5.2.3 常用操作

| 快捷键     | 功能               | 说明                        |
| ---------- | ------------------ | --------------------------- |
| 直接打字   | 输入/编辑文本      | 和记事本一样                |
| `Ctrl + O` | **保存**文件       | 按下后再按 Enter 确认文件名 |
| `Ctrl + X` | **退出** nano      | 如果未保存会提示是否保存    |
| `Ctrl + K` | **剪切**当前行     | 把整行剪切（可多次按）      |
| `Ctrl + U` | **粘贴**剪切的内容 |                             |
| `Ctrl + W` | **搜索**内容       | 输入关键词后回车            |
| `Ctrl + \` | **搜索并替换**     |                             |
| `Ctrl + G` | 查看帮助           |                             |
| `Ctrl + _` | 跳转到指定行号     |                             |
| `Alt + U`  | 撤销               |                             |
| `Alt + E`  | 重做               |                             |
| `Ctrl + C` | 显示当前光标位置   | 显示行号和列号              |

### 5.2.4 保存退出的完整流程

```
1. 按 Ctrl + X 退出
2. 提示：Save modified buffer? (save 修改的内容？)
3. 按 Y 保存（或 N 不保存）
4. 显示文件名，按 Enter 确认

或者分两步：
1. 先按 Ctrl + O 保存（回车确认文件名）
2. 再按 Ctrl + X 退出
```

### 5.2.5 实际操作练习

```bash
# 1. 创建并编辑一个文件
nano ~/practice.txt

# 2. 在 nano 中输入以下内容：
#    Hello, this is my first nano file.
#    I am learning Linux!
#    Today's date is: April 13, 2026.

# 3. 按 Ctrl + O，然后 Enter 保存
# 4. 按 Ctrl + X 退出

# 5. 验证内容
cat ~/practice.txt
```

---

## 5.3 Vim 编辑器 —— 效率之王

### 5.3.1 为什么要学 Vim？

- **无处不在**：几乎每台 Linux 服务器都预装了 vi 或 vim
- **效率极高**：熟练后，编辑速度远超其他编辑器
- **不需要鼠标**：纯键盘操作，在 SSH 远程终端中特别方便
- **高度可定制**：可以通过配置文件打造成 IDE

但 Vim 对新手来说**非常反直觉**，因为它有**模式**的概念。

### 5.3.2 Vim 的三种核心模式

这是理解 Vim 的**关键中的关键**：

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   ┌──────────┐    按 i/a/o    ┌──────────────┐      │
│   │          │ ─────────────→ │              │      │
│   │ 普通模式  │                │   插入模式    │      │
│   │ (Normal)  │ ←───────────── │   (Insert)   │      │
│   │          │    按 Esc      │              │      │
│   └────┬─────┘                └──────────────┘      │
│        │                                            │
│        │ 按 :                                       │
│        ↓                                            │
│   ┌──────────────┐                                  │
│   │              │                                  │
│   │  命令行模式   │                                  │
│   │ (Command)    │                                  │
│   │              │                                  │
│   └──────────────┘                                  │
│   按 Enter 执行后回到普通模式                         │
│   按 Esc 取消回到普通模式                             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

| 模式                      | 作用                     | 如何进入                    | 如何退出                     |
| ------------------------- | ------------------------ | --------------------------- | ---------------------------- |
| **普通模式（Normal）**    | 移动光标、复制粘贴、删除 | 打开文件时默认就是          | —                            |
| **插入模式（Insert）**    | 输入/编辑文字            | 在普通模式按 `i` `a` `o` 等 | 按 `Esc`                     |
| **命令行模式（Command）** | 保存、退出、搜索替换     | 在普通模式按 `:`            | 按 `Enter` 执行或 `Esc` 取消 |

> 💡 **新手最常犯的错误**：打开 Vim 后直接开始打字 → 发现打出来的不是字，而是在执行各种操作 → 慌了 → 不知道怎么退出 → 崩溃。
>
> **记住**：打开后默认是**普通模式**，按 `i` 进入插入模式后才能输入文字！

### 5.3.3 打开 Vim

```bash
# 打开/创建文件
vim filename.txt

# 打开并跳到指定行
vim +10 filename.txt

# 以只读模式打开
vim -R filename.txt
# 或使用 view 命令
view filename.txt

# 编辑系统文件
sudo vim /etc/hosts
```

> 💡 如果系统中没有 `vim`，只有 `vi`（精简版），可以安装完整版：
> ```bash
> sudo apt install vim -y
> ```

### 5.3.4 最重要的操作：保存与退出

**这是 Vim 新手生存的第一课！**

在**普通模式**下（不确定就按几次 `Esc` 确保在普通模式），输入：

| 命令   | 功能               | 说明                                      |
| ------ | ------------------ | ----------------------------------------- |
| `:w`   | **保存**（Write）  | 按 `:` 进入命令行模式，输入 `w`，按 Enter |
| `:q`   | **退出**（Quit）   | 文件未修改时可直接退出                    |
| `:wq`  | **保存并退出**     | 最常用的组合！                            |
| `:q!`  | **强制退出不保存** | 放弃所有修改                              |
| `:wq!` | **强制保存并退出** | 对只读文件有用（需要权限）                |
| `ZZ`   | 保存并退出的快捷键 | 在普通模式直接按大写 ZZ（不需要 `:`）     |

> 🆘 **紧急救命指南**：如果你在 Vim 中完全迷路了：
> 1. 按 `Esc` 键 3 次（确保回到普通模式）
> 2. 输入 `:q!` 然后按 Enter（强制退出不保存）
> 3. 你安全了 😮‍💨

### 5.3.5 插入模式 —— 输入文字

从普通模式进入插入模式有多种方式：

| 按键 | 插入位置               | 说明               |
| ---- | ---------------------- | ------------------ |
| `i`  | 光标**前面**           | **最常用**，Insert |
| `a`  | 光标**后面**           | Append             |
| `I`  | 当前**行首**           | 大写 I             |
| `A`  | 当前**行尾**           | 大写 A             |
| `o`  | 当前行**下方新建一行** | Open               |
| `O`  | 当前行**上方新建一行** | 大写 O             |

进入插入模式后：
- 左下角会显示 `-- INSERT --`
- 这时你可以像记事本一样正常输入文字
- 编辑完后，按 `Esc` 回到普通模式

### 5.3.6 普通模式 —— 移动与操作

#### 光标移动

| 按键           | 移动方式               |
| -------------- | ---------------------- |
| `h`            | ← 左移一个字符         |
| `j`            | ↓ 下移一行             |
| `k`            | ↑ 上移一行             |
| `l`            | → 右移一个字符         |
| `w`            | 跳到下一个**单词**开头 |
| `b`            | 跳到上一个**单词**开头 |
| `e`            | 跳到当前单词末尾       |
| `0`            | 跳到行首               |
| `$`            | 跳到行尾               |
| `gg`           | 跳到文件**第一行**     |
| `G`            | 跳到文件**最后一行**   |
| `10G` 或 `:10` | 跳到**第 10 行**       |
| `Ctrl + f`     | 向下翻一页             |
| `Ctrl + b`     | 向上翻一页             |

> 💡 当然，方向键 `↑↓←→` 在 Vim 中也能用，初学阶段别勉强用 hjkl。

#### 删除

| 按键        | 功能                       |
| ----------- | -------------------------- |
| `x`         | 删除光标处的一个字符       |
| `dd`        | **删除整行**（其实是剪切） |
| `3dd`       | 删除 3 行                  |
| `dw`        | 删除一个单词               |
| `d$` 或 `D` | 从光标删到行尾             |
| `d0`        | 从光标删到行首             |
| `dG`        | 从当前行删到文件末尾       |

#### 复制与粘贴

| 按键  | 功能                        |
| ----- | --------------------------- |
| `yy`  | **复制**当前行（Yank）      |
| `3yy` | 复制 3 行                   |
| `yw`  | 复制一个单词                |
| `p`   | 在光标**后面**粘贴（Paste） |
| `P`   | 在光标**前面**粘贴          |

> 💡 **dd + p 组合**：`dd` 是剪切（不是真正的删除），配合 `p` 可以实现"移动行"的效果。

#### 撤销与重做

| 按键       | 功能                             |
| ---------- | -------------------------------- |
| `u`        | **撤销**上一步操作（Undo）       |
| `Ctrl + r` | **重做**（重新执行被撤销的操作） |

#### 搜索

| 按键      | 功能             |
| --------- | ---------------- |
| `/关键词` | 向下搜索         |
| `?关键词` | 向上搜索         |
| `n`       | 跳到下一个匹配项 |
| `N`       | 跳到上一个匹配项 |

### 5.3.7 命令行模式 —— 高级操作

在普通模式下按 `:` 进入命令行模式：

```vim
:w                     " 保存
:q                     " 退出
:wq                    " 保存并退出
:q!                    " 强制退出不保存

:set number            " 显示行号
:set nonumber          " 隐藏行号

:s/旧文本/新文本/       " 替换当前行第一个匹配
:s/旧文本/新文本/g      " 替换当前行所有匹配
:%s/旧文本/新文本/g     " 替换全文所有匹配
:%s/旧文本/新文本/gc    " 替换全文所有匹配，但每次确认

:10                    " 跳到第 10 行
```

### 5.3.8 Vim 完整操作流程（新手必练）

跟着一步步做：

```bash
# 1. 用 vim 创建文件
vim ~/vim_practice.txt
```

```
# 2. 你现在在普通模式。按 i 进入插入模式
#    左下角会出现 -- INSERT --

# 3. 输入以下内容：
Hello, Vim!
This is line 2.
This is line 3.
I am learning Vim editing.
This is the last line.

# 4. 按 Esc 回到普通模式

# 5. 用 gg 跳到第一行

# 6. 用 yy 复制第一行

# 7. 用 G 跳到最后一行

# 8. 用 p 粘贴（第一行的内容会出现在最后）

# 9. 用 u 撤销刚才的粘贴

# 10. 用 /learning 搜索 "learning" 这个词

# 11. 用 dd 删除搜索到的那一行

# 12. 用 :set number 显示行号

# 13. 保存退出：按 :wq 然后 Enter
```

```bash
# 14. 验证文件内容
cat ~/vim_practice.txt
```

### 5.3.9 Vim 速查卡

按重要性排序，新手先掌握前两栏：

| 优先级 | 操作              | 按键          |
| ------ | ----------------- | ------------- |
| 🔴 必会 | 进入插入模式      | `i`           |
| 🔴 必会 | 退出插入模式      | `Esc`         |
| 🔴 必会 | 保存并退出        | `:wq` + Enter |
| 🔴 必会 | 强制退出不保存    | `:q!` + Enter |
| 🔴 必会 | 撤销              | `u`           |
| 🟡 常用 | 删除整行          | `dd`          |
| 🟡 常用 | 复制整行          | `yy`          |
| 🟡 常用 | 粘贴              | `p`           |
| 🟡 常用 | 搜索              | `/关键词`     |
| 🟡 常用 | 跳到文件开头/末尾 | `gg` / `G`    |
| 🟢 进阶 | 行尾插入          | `A`           |
| 🟢 进阶 | 下方新行插入      | `o`           |
| 🟢 进阶 | 全文替换          | `:%s/旧/新/g` |
| 🟢 进阶 | 显示行号          | `:set number` |

---

## 5.4 VS Code 联动 —— 最佳开发体验

### 5.4.1 为什么推荐 VS Code + WSL？

| 对比     | 纯终端编辑器（Vim/Nano） | VS Code + WSL         |
| -------- | ------------------------ | --------------------- |
| 学习曲线 | Vim 较陡峭               | 和 Windows 上使用一样 |
| 代码补全 | 有限                     | 强大的 IntelliSense   |
| 调试     | 困难                     | 内置可视化调试器      |
| Git 操作 | 命令行                   | 可视化 Git 面板       |
| 文件管理 | 命令行                   | 图形化文件浏览器      |
| 终端     | 另开窗口                 | 编辑器内嵌终端        |

### 5.4.2 安装与配置（回顾第二章）

确保你已经完成：
1. Windows 上安装了 VS Code
2. 安装了 `WSL` 扩展（扩展ID：`ms-vscode-remote.remote-wsl`）

### 5.4.3 使用 `code .` 命令

```bash
# 在 WSL 终端中，进入你的项目目录
cd ~/my-project

# 用 VS Code 打开当前目录
code .
```

**第一次使用时**，WSL 会自动下载并安装 VS Code Server：
```
Installing VS Code Server for x64...
Downloading: 100%
Unpacking: 100%
```

之后 VS Code 窗口会在 Windows 上打开，左下角显示 `WSL: Ubuntu`。

### 5.4.4 VS Code 中的常用操作

**打开特定文件**：
```bash
# 用 VS Code 打开一个具体文件
code ~/my-project/main.py

# 打开并跳到指定行
code --goto ~/my-project/main.py:25
```

**VS Code 内嵌终端**：
- 快捷键 `` Ctrl + ` ``（反引号）打开/关闭内嵌终端
- 这个终端就是 WSL 的 Linux 终端
- 可以直接在里面运行 Linux 命令

**VS Code 常用快捷键**：

| 快捷键             | 功能          |
| ------------------ | ------------- |
| `Ctrl + S`         | 保存          |
| `Ctrl + P`         | 快速打开文件  |
| `Ctrl + Shift + P` | 命令面板      |
| `Ctrl + F`         | 查找          |
| `Ctrl + H`         | 查找并替换    |
| `Ctrl + Shift + F` | 全局搜索      |
| `Ctrl + /`         | 切换注释      |
| `` Ctrl + ` ``     | 打开/关闭终端 |
| `Ctrl + B`         | 切换侧边栏    |
| `Ctrl + \`         | 编辑器分屏    |

### 5.4.5 VS Code + WSL 的工作流

```bash
# 1. 在 WSL 终端创建项目
mkdir -p ~/projects/my-app
cd ~/projects/my-app

# 2. 初始化项目文件
touch index.html style.css app.js

# 3. 用 VS Code 打开
code .

# 4. 在 VS Code 中编辑文件（图形界面，正常使用）

# 5. 在 VS Code 内嵌终端中运行命令
# 比如：python3 app.py 或 node server.js

# 6. 所有文件都存储在 Linux 文件系统中，性能最优
```

### 5.4.6 三种编辑器的使用场景总结

| 场景                     | 推荐编辑器                    | 原因                       |
| ------------------------ | ----------------------------- | -------------------------- |
| 快速修改一个配置文件     | **Nano**                      | 简单快速，不需要记复杂操作 |
| SSH 远程连接服务器编辑   | **Vim**                       | 服务器上通常只有 vi/vim    |
| 日常项目开发             | **VS Code**                   | 功能完整，体验最好         |
| 编辑需要 root 权限的文件 | **sudo nano** 或 **sudo vim** | VS Code 不方便直接 sudo    |
| 查看/快速修改系统日志    | **Vim**（只读模式）           | `vim -R` 或 `view`         |

---

# 第六章：软件包管理

---

## 6.1 什么是软件包管理？

### 6.1.1 Linux 安装软件的方式

在 Windows 中，安装软件的流程是：
```
百度搜索 → 找到官网 → 下载 .exe → 双击安装 → 下一步下一步 → 完成
```

在 Linux 中，有一种**更优雅**的方式——**包管理器**：
```bash
sudo apt install nginx      # 一行命令，自动下载、安装、配置
```

### 6.1.2 什么是软件包（Package）？

一个**软件包**是一个打包好的文件，包含：
- 程序的可执行文件
- 库文件
- 配置文件模板
- 文档
- 依赖关系清单（这个软件需要哪些其他软件才能运行）

### 6.1.3 什么是包管理器？

**包管理器**是管理软件包的工具，它帮你：

| 功能         | 自己手动做需要…            | 包管理器一行命令搞定   |
| ------------ | -------------------------- | ---------------------- |
| 下载软件     | 去官网找下载链接           | `apt install`          |
| 安装软件     | 编译、拷贝文件、配置环境   | 自动完成               |
| 处理依赖     | 一个个找需要的库           | 自动计算并安装所有依赖 |
| 升级软件     | 找新版本、卸载旧版、装新版 | `apt upgrade`          |
| 卸载软件     | 找到所有安装的文件逐一删除 | `apt remove`           |
| 查看已装软件 | 手动记录                   | `apt list --installed` |

### 6.1.4 不同发行版的包管理器

| 发行版                     | 包格式         | 包管理器                         | 常用命令前缀 |
| -------------------------- | -------------- | -------------------------------- | ------------ |
| **Ubuntu / Debian**        | `.deb`         | `apt`（现代）/ `apt-get`（传统） | `apt`        |
| **CentOS / RHEL / Fedora** | `.rpm`         | `yum`（旧）/ `dnf`（新）         | `dnf`        |
| **Arch Linux**             | `.pkg.tar.zst` | `pacman`                         | `pacman`     |
| **Alpine**                 | `.apk`         | `apk`                            | `apk`        |

> 本教程使用 **Ubuntu**，所以我们学习 `apt`。

### 6.1.5 `apt` vs `apt-get`

你可能在网上看到有些教程用 `apt-get`，有些用 `apt`。它们的关系：

| 特性     | `apt-get`          | `apt`                    |
| -------- | ------------------ | ------------------------ |
| 出现时间 | 更早               | 2014年推出               |
| 定位     | 传统工具           | 现代替代品               |
| 输出     | 纯文本             | 更友好（有进度条、颜色） |
| 功能     | 任务分散到多个命令 | 整合了常用功能           |
| 推荐     | 旧脚本中常见       | ✅ **日常使用推荐**       |

**一句话：日常用 `apt` 就对了。**

---

## 6.2 软件源（Repository）

### 6.2.1 什么是软件源？

软件源就是**存放软件包的服务器**。当你执行 `apt install nginx` 时，`apt` 会从软件源下载 nginx 的安装包。

Ubuntu 的默认软件源在美国，对于国内用户来说速度可能较慢。可以更换为国内镜像。

### 6.2.2 查看当前软件源

```bash
cat /etc/apt/sources.list
```

你会看到类似这样的内容：

```
deb http://archive.ubuntu.com/ubuntu/ jammy main restricted
deb http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted
deb http://archive.ubuntu.com/ubuntu/ jammy universe
...
```

每一行的格式：
```
deb <软件源URL> <Ubuntu版本代号> <软件分类>
```

- `deb`：二进制包（安装用的）
- `deb-src`：源代码包
- `jammy`：Ubuntu 22.04 的版本代号
- `main`：官方支持的自由软件
- `restricted`：官方支持的专有驱动
- `universe`：社区维护的自由软件
- `multiverse`：非自由软件

### 6.2.3 更换国内镜像源（可选，提速）

```bash
# 1. 备份原始源配置文件
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup

# 2. 编辑源文件
sudo nano /etc/apt/sources.list

# 3. 把所有 http://archive.ubuntu.com 替换为

#    阿里云：   http://mirrors.aliyun.com
#    清华大学： https://mirrors.tuna.tsinghua.edu.cn
#    中科大：   https://mirrors.ustc.edu.cn

# 你可以在 nano 中用 Ctrl+\ 进行搜索替换：
# 搜索：  archive.ubuntu.com
# 替换为：mirrors.aliyun.com

# 4. 保存退出（Ctrl+O, Enter, Ctrl+X）

# 5. 更新软件源索引
sudo apt update
```

> 💡 **WSL 中的情况**：WSL 环境一般网络还行，如果速度不慢可以不换源。网速慢时再换。

---

## 6.3 `apt` 核心命令

### 6.3.1 更新软件包索引

```bash
sudo apt update
```

**这条命令做了什么？** 它去连接软件源服务器，下载最新的软件包**列表**（哪些软件有新版本、有哪些新软件可用），但**并不安装任何东西**。

**类比**：就像刷新一下淘宝的商品列表，知道有什么新货，但还没买。

> 💡 **在安装任何软件之前，都建议先运行 `sudo apt update`**，确保软件包列表是最新的。

输出示例：
```
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]
...
Reading package lists... Done
Building dependency tree... Done
15 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

### 6.3.2 升级已安装的软件

```bash
# 升级所有已安装的软件包到最新版本
sudo apt upgrade

# 升级前先更新索引（推荐组合）
sudo apt update && sudo apt upgrade -y
# -y = 自动确认（不需要手动输入 Y）

# 查看哪些软件包可以升级
apt list --upgradable
```

**`upgrade` vs `full-upgrade` vs `dist-upgrade`**：

| 命令               | 行为                                   | 安全级别 |
| ------------------ | -------------------------------------- | -------- |
| `apt upgrade`      | 升级已安装的包，**不删除**任何包       | ✅ 安全   |
| `apt full-upgrade` | 升级已安装的包，如需要**可能删除**旧包 | ⚠️ 注意   |
| `apt dist-upgrade` | 同 full-upgrade（传统名称）            | ⚠️ 注意   |

### 6.3.3 安装软件

```bash
sudo apt install 软件包名
```

```bash
# 安装单个软件
sudo apt install vim

# 安装多个软件（空格分隔）
sudo apt install vim git curl wget

# 安装时自动确认（不用手动输入 Y）
sudo apt install nginx -y

# 安装指定版本
sudo apt install nginx=1.18.0-0ubuntu1

# 只下载不安装
sudo apt install --download-only nginx

# 模拟安装（看看会装什么，但不真正安装）
sudo apt install --simulate nginx
# 或简写
sudo apt install -s nginx
```

安装过程中你会看到：
```
Reading package lists... Done
Building dependency tree... Done
The following additional packages will be installed:     ← 自动安装的依赖
  libnginx-mod-http-echo nginx-common nginx-core
The following NEW packages will be installed:
  nginx nginx-common nginx-core
0 upgraded, 3 newly installed, 0 to remove.
Need to get 592 kB of archives.                         ← 需要下载的大小
After this operation, 1,588 kB of additional disk space will be used.  ← 安装后占用空间
Do you want to continue? [Y/n]                          ← 确认（输入 Y 或直接回车）
```

### 6.3.4 卸载软件

```bash
# 卸载软件（保留配置文件）
sudo apt remove nginx

# 卸载软件并删除配置文件（更彻底）
sudo apt purge nginx

# 卸载不再需要的依赖包（清理孤立依赖）
sudo apt autoremove

# 组合：彻底卸载并清理
sudo apt purge nginx -y && sudo apt autoremove -y
```

**`remove` vs `purge` 的区别**：

```
remove：
  ✅ 删除程序本身
  ❌ 保留 /etc/ 下的配置文件
  适用场景：可能以后还会重装，想保留配置

purge：
  ✅ 删除程序本身
  ✅ 删除配置文件
  适用场景：彻底不要了，干干净净
```

### 6.3.5 搜索软件包

```bash
# 搜索软件包
apt search nginx

# 搜索结果太多？用 grep 筛选
apt search nginx | grep "^nginx"

# 查看软件包的详细信息
apt show nginx
```

`apt show` 输出示例：
```
Package: nginx
Version: 1.18.0-6ubuntu14.4
Priority: optional
Section: web
Origin: Ubuntu
Installed-Size: 44.3 kB
Depends: nginx-core | nginx-full | nginx-light | nginx-extras
Description: small, powerful, scalable web/proxy server
```

### 6.3.6 查看已安装的软件

```bash
# 列出所有已安装的软件包
apt list --installed

# 太多了？搜索特定的
apt list --installed | grep nginx

# 查看软件包安装了哪些文件
dpkg -L nginx

# 查看某个文件属于哪个软件包
dpkg -S /usr/sbin/nginx
```

### 6.3.7 清理缓存

```bash
# 清理下载的 .deb 安装包缓存
sudo apt clean            # 清除所有缓存（释放最多空间）
sudo apt autoclean        # 只清除过时版本的缓存

# 删除不再需要的依赖
sudo apt autoremove

# 查看缓存占用了多少空间
du -sh /var/cache/apt/archives/
```

### 6.3.8 `apt` 命令速查表

| 命令                    | 功能                 | 需要 sudo |
| ----------------------- | -------------------- | --------- |
| `apt update`            | 更新软件包列表       | ✅         |
| `apt upgrade`           | 升级所有已安装包     | ✅         |
| `apt install 包名`      | 安装软件             | ✅         |
| `apt remove 包名`       | 卸载软件（保留配置） | ✅         |
| `apt purge 包名`        | 彻底卸载             | ✅         |
| `apt autoremove`        | 清理孤立依赖         | ✅         |
| `apt search 关键词`     | 搜索软件包           | ❌         |
| `apt show 包名`         | 查看包详情           | ❌         |
| `apt list --installed`  | 列出已安装的包       | ❌         |
| `apt list --upgradable` | 列出可升级的包       | ❌         |
| `apt clean`             | 清理缓存             | ✅         |

---

## 6.4 实战：在 WSL 中安装常用软件

### 6.4.1 安装前的准备

```bash
# 第一步：永远先更新软件包列表
sudo apt update

# 第二步（可选）：升级已安装的软件
sudo apt upgrade -y
```

### 6.4.2 实战一：安装 Nginx（Web 服务器）

```bash
# 1. 安装 Nginx
sudo apt install nginx -y

# 2. 查看安装的版本
nginx -v
# 输出类似：nginx version: nginx/1.18.0 (Ubuntu)

# 3. 启动 Nginx
sudo service nginx start

# 4. 查看 Nginx 状态
sudo service nginx status
# 应该显示 * nginx is running

# 5. 验证：在 WSL 中访问
curl http://localhost
# 你应该看到一大段 HTML 代码，包含 "Welcome to nginx!"

# 6. 在 Windows 浏览器中访问
#    打开浏览器，输入 http://localhost
#    你应该看到 Nginx 的欢迎页面！🎉

# 7. 停止 Nginx
sudo service nginx stop

# 8. 其他管理命令
sudo service nginx restart    # 重启
sudo service nginx reload     # 重新加载配置（不中断服务）
```

> 💡 **WSL2 中的 service 命令**：WSL2 不使用 `systemd`（传统 Linux 的服务管理器），而是用 `service` 命令来管理服务。但较新版本的 WSL2 已支持 systemd，你可以在 `/etc/wsl.conf` 中启用：
> ```ini
> [boot]
> systemd=true
> ```
> 重启 WSL 后就可以使用 `systemctl` 命令了。

### 6.4.3 实战二：安装 Python

```bash
# 1. 查看是否已安装 Python
python3 --version
# Ubuntu 通常预装了 Python3

# 2. 如果没有，安装 Python3 和 pip
sudo apt install python3 python3-pip python3-venv -y

# 3. 验证
python3 --version
# Python 3.10.12（或其他版本）

pip3 --version
# pip 22.0.2 from /usr/lib/python3/dist-packages/pip

# 4. 测试 Python
python3 -c "print('Hello from Python in WSL!')"

# 5. 创建虚拟环境（推荐的 Python 项目管理方式）
mkdir -p ~/projects/python-test
cd ~/projects/python-test
python3 -m venv venv               # 创建虚拟环境
source venv/bin/activate            # 激活虚拟环境
# 提示符前面会出现 (venv)
pip install requests                # 在虚拟环境中安装包
deactivate                          # 退出虚拟环境
```

### 6.4.4 实战三：安装 Node.js

> ⚠️ **不要直接 `apt install nodejs`！** Ubuntu 官方源中的 Node.js 版本通常很旧。推荐使用 **nvm**（Node Version Manager）安装。

```bash
# 方法一：使用 nvm（推荐！）

# 1. 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# 2. 重新加载 shell 配置
source ~/.bashrc

# 3. 验证 nvm 安装
nvm --version

# 4. 安装最新 LTS 版本的 Node.js
nvm install --lts

# 5. 验证
node -v          # 例如：v20.11.0
npm -v           # 例如：10.2.4

# 6. 测试
node -e "console.log('Hello from Node.js in WSL!')"
```

```bash
# 方法二：使用 NodeSource 官方源

# 1. 添加 NodeSource 的软件源（以 Node.js 20 为例）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# 2. 安装 Node.js
sudo apt install nodejs -y

# 3. 验证
node -v
npm -v
```

### 6.4.5 实战四：安装其他常用工具

```bash
# 一次性安装开发常用工具
sudo apt install -y \
    git \              # 版本控制
    curl \             # HTTP 请求工具
    wget \             # 下载工具
    htop \             # 进程监控（比 top 好用）
    tree \             # 树形显示目录结构
    zip unzip \        # 压缩/解压缩
    net-tools \        # 网络工具（ifconfig 等）
    build-essential    # C/C++ 编译工具链（make, gcc 等）
```

验证安装：
```bash
git --version
curl --version
htop --version
tree --version
```

---

## 6.5 其他安装方式（了解）

除了 `apt`，Linux 中还有其他安装软件的方式：

### 6.5.1 使用 `.deb` 安装包

有些软件提供 `.deb` 下载包（类似 Windows 的 `.exe`）：

```bash
# 下载 .deb 包
wget https://example.com/software.deb

# 安装 .deb 包
sudo dpkg -i software.deb

# 如果有依赖问题，修复依赖
sudo apt install -f
```

### 6.5.2 使用 Snap（容器化安装）

```bash
# Snap 是 Ubuntu 推出的另一种软件包格式
sudo snap install vlc
sudo snap install code --classic    # VS Code
```

### 6.5.3 从源代码编译安装

```bash
# 这是最"Linux"的方式，适合源里没有的软件
# 通用流程：
./configure        # 检查系统环境、生成编译配置
make               # 编译
sudo make install  # 安装到系统
```

> 💡 **初学者建议**：优先使用 `apt` 安装。如果 `apt` 里没有，再考虑其他方式。

---

## 6.6 本章总结

| 操作           | 命令                                            |
| -------------- | ----------------------------------------------- |
| 更新软件包列表 | `sudo apt update`                               |
| 升级所有软件   | `sudo apt upgrade -y`                           |
| 安装软件       | `sudo apt install 包名 -y`                      |
| 卸载软件       | `sudo apt remove 包名` 或 `sudo apt purge 包名` |
| 搜索软件       | `apt search 关键词`                             |
| 清理           | `sudo apt autoremove && sudo apt clean`         |

**你的日常操作流程**：

```bash
# 每隔几天执行一次，保持系统最新
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```

---

# Linux 操作入门教程（WSL2 版）— 第三篇

> 接上篇第4-6章，本篇覆盖第7-10章：进程与系统监控、网络基础、Shell 脚本入门、综合实战项目。

---

# 第七章：进程与系统监控

---

## 7.1 什么是进程？

### 7.1.1 进程的概念

**程序（Program）** 和 **进程（Process）** 的区别：

| 概念 | 含义                             | 类比           |
| ---- | -------------------------------- | -------------- |
| 程序 | 存储在磁盘上的可执行文件（静态） | 菜谱           |
| 进程 | 程序运行起来后的实例（动态）     | 正在做的那道菜 |

一个程序可以同时运行多个进程。比如你打开了两个终端窗口，就有两个 `bash` 进程在运行。

### 7.1.2 进程的关键属性

每个进程都有以下信息：

| 属性        | 说明                           | 示例                     |
| ----------- | ------------------------------ | ------------------------ |
| **PID**     | 进程 ID，系统分配的唯一编号    | `1234`                   |
| **PPID**    | 父进程 ID（谁启动了这个进程）  | `1`                      |
| **USER**    | 进程的所有者（以谁的身份运行） | `datovm`                 |
| **状态**    | 运行中、休眠、停止、僵尸等     | `R`（运行）、`S`（休眠） |
| **CPU%**    | 占用的 CPU 百分比              | `2.5%`                   |
| **MEM%**    | 占用的内存百分比               | `1.2%`                   |
| **COMMAND** | 启动这个进程的命令             | `/usr/sbin/nginx`        |

### 7.1.3 进程的状态

```
进程的生命周期：

创建 → 就绪(R) → 运行(R) → 结束
              ↑         ↓
              └── 阻塞(S/D) ──→ （等待 I/O、信号等）
```

| 状态码 | 含义               | 说明                               |
| ------ | ------------------ | ---------------------------------- |
| `R`    | Running / Runnable | 正在运行或等待 CPU 分配            |
| `S`    | Sleeping           | 可中断休眠（等待事件，如用户输入） |
| `D`    | Disk Sleep         | 不可中断休眠（等待磁盘 I/O）       |
| `T`    | Stopped            | 被暂停（收到 SIGSTOP 信号）        |
| `Z`    | Zombie             | 僵尸进程（已结束但父进程尚未回收） |

### 7.1.4 前台进程与后台进程

```bash
# 前台运行（默认）—— 占用终端，必须等它结束
ping google.com
# 按 Ctrl+C 终止

# 后台运行 —— 加 & 符号，不占用终端
ping google.com > /dev/null &
# 立即返回提示符，输出 [1] 12345（作业号和 PID）

# 查看当前终端的后台作业
jobs
# 输出：[1]+  Running    ping google.com > /dev/null &

# 把后台作业调回前台
fg %1             # %1 是作业号

# 把前台程序放到后台
# 先按 Ctrl+Z 暂停当前前台程序
# 然后输入 bg 让它在后台继续运行
bg %1
```

**完整流程演示**：
```bash
# 1. 启动一个前台程序
sleep 300                    # 前台休眠 300 秒

# 2. 按 Ctrl+Z 暂停它
# 输出：[1]+  Stopped     sleep 300

# 3. 让它在后台继续运行
bg %1
# 输出：[1]+ sleep 300 &

# 4. 查看后台作业
jobs
# 输出：[1]+  Running     sleep 300 &

# 5. 把它调回前台
fg %1

# 6. 按 Ctrl+C 终止
```

---

## 7.2 查看进程

### 7.2.1 `ps` —— 进程快照

`ps`（Process Status）显示当前时刻的进程信息（快照，不会自动刷新）。

```bash
# 只显示当前终端的进程
ps
# 输出：
#   PID TTY          TIME CMD
#  1234 pts/0    00:00:00 bash
#  5678 pts/0    00:00:00 ps
```

**最常用的写法——`ps aux`**：

```bash
ps aux
```

```
USER       PID %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
root         1  0.0  0.1 167324 11080 ?     Ss   10:00   0:01 /sbin/init
root        23  0.0  0.0   2616  1524 ?     Sl   10:00   0:00 /init
datovm     142  0.0  0.1  11640  7956 pts/0 Ss   10:01   0:00 -bash
www-data   890  0.0  0.1  55288  5432 ?     S    10:05   0:00 nginx: worker
datovm    1023  0.0  0.0  10760  3344 pts/0 R+   10:30   0:00 ps aux
```

**各列含义**：

| 列名      | 含义                                  |
| --------- | ------------------------------------- |
| `USER`    | 进程所有者                            |
| `PID`     | 进程 ID                               |
| `%CPU`    | CPU 使用率                            |
| `%MEM`    | 内存使用率                            |
| `VSZ`     | 虚拟内存大小（KB）                    |
| `RSS`     | 实际占用的物理内存（KB）              |
| `TTY`     | 关联的终端（`?` 表示无终端/后台服务） |
| `STAT`    | 进程状态                              |
| `START`   | 启动时间                              |
| `TIME`    | 累计 CPU 使用时间                     |
| `COMMAND` | 启动命令                              |

**STAT 列的详细含义**：

| 符号 | 含义                         |
| ---- | ---------------------------- |
| `S`  | 休眠中                       |
| `R`  | 运行中                       |
| `s`  | Session leader（会话首进程） |
| `l`  | 多线程                       |
| `+`  | 前台进程                     |
| `<`  | 高优先级                     |
| `N`  | 低优先级                     |

**`ps` 的常用组合**：

```bash
# 查看所有进程（最常用！）
ps aux

# 搜索特定进程（配合 grep）
ps aux | grep nginx
ps aux | grep python3

# 注意：grep 本身也会出现在结果中，排除它：
ps aux | grep nginx | grep -v grep

# 以树形结构显示进程关系（父子关系一目了然）
ps auxf

# 只显示特定用户的进程
ps -u datovm

# 按 CPU 使用率排序（降序）
ps aux --sort=-%cpu | head -10

# 按内存使用率排序（降序）
ps aux --sort=-%mem | head -10

# 查看特定 PID 的进程信息
ps -p 1234
```

### 7.2.2 `top` —— 实时监控

`top` 是 Linux 内置的实时进程监控工具（类似 Windows 的任务管理器）。

```bash
top
```

界面解读：

```
top - 10:30:05 up  0:30,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  42 total,   1 running,  41 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.2 sy,  0.0 ni, 99.5 id,  0.0 wa,  0.0 hi,  0.0 si
MiB Mem :   7951.6 total,   6543.2 free,    802.4 used,    606.0 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   6893.2 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    1 root      20   0  167324  11080   8196 S   0.0   0.1   0:01.23 systemd
  890 www-data  20   0   55288   5432   3600 S   0.3   0.1   0:02.15 nginx
  ...
```

**头部信息解读**：

| 行    | 含义                                                         |
| ----- | ------------------------------------------------------------ |
| 第1行 | 当前时间、运行时长、登录用户数、**负载均衡**（1/5/15分钟平均） |
| 第2行 | 总进程数、运行中/休眠/停止/僵尸进程数                        |
| 第3行 | CPU 使用情况：us=用户空间, sy=内核空间, id=空闲, wa=等待I/O  |
| 第4行 | 内存使用情况                                                 |
| 第5行 | 交换分区使用情况                                             |

> 💡 **负载均衡（Load Average）**：
> - 数值表示 CPU 等待队列的平均长度
> - 单核 CPU：`1.0` 表示满载，`0.5` 表示 50% 负载
> - 4 核 CPU：`4.0` 才算满载
> - 一般来说，负载 < CPU 核数 就是健康的

**`top` 中的交互快捷键**：

| 按键       | 功能                          |
| ---------- | ----------------------------- |
| `q`        | 退出 top                      |
| `h`        | 显示帮助                      |
| `P`        | 按 **CPU** 使用率排序（默认） |
| `M`        | 按**内存**使用率排序          |
| `T`        | 按运行**时间**排序            |
| `k`        | 杀死进程（输入 PID）          |
| `r`        | 调整进程优先级（输入 PID）    |
| `1`        | 显示每个 CPU 核心的使用率     |
| `c`        | 显示完整命令路径              |
| `f`        | 选择显示哪些列                |
| `d` 或 `s` | 修改刷新间隔（默认 3 秒）     |

### 7.2.3 `htop` —— 更好用的 top

`htop` 是 `top` 的增强版，界面更友好，支持鼠标操作、颜色显示、横向滚动。

```bash
# 安装 htop
sudo apt install htop -y

# 运行
htop
```

**htop 的优势**：

| 特性     | top      | htop          |
| -------- | -------- | ------------- |
| 颜色     | 单色     | ✅ 彩色        |
| CPU 图表 | 数字     | ✅ 柱状图      |
| 鼠标操作 | ❌        | ✅ 可点击      |
| 横向滚动 | ❌        | ✅ 查看长命令  |
| 进程树   | 需要按键 | ✅ F5 切换     |
| 搜索进程 | 不方便   | ✅ F3 搜索     |
| 杀进程   | 输入 PID | ✅ F9 选择信号 |

**htop 快捷键**：

| 按键        | 功能         |
| ----------- | ------------ |
| `F1`        | 帮助         |
| `F2`        | 设置         |
| `F3`        | 搜索进程     |
| `F4`        | 过滤         |
| `F5`        | 树形视图     |
| `F6`        | 排序方式     |
| `F9`        | 杀死进程     |
| `F10` / `q` | 退出         |
| `↑↓`        | 上下选择进程 |
| `Space`     | 标记进程     |

---

## 7.3 终止进程

### 7.3.1 信号（Signal）的概念

Linux 通过**信号**来控制进程。信号是操作系统发给进程的一种通知。

| 信号编号 | 信号名    | 含义     | 效果                               |
| -------- | --------- | -------- | ---------------------------------- |
| `2`      | `SIGINT`  | 中断     | `Ctrl+C` 发送的信号，优雅终止      |
| `9`      | `SIGKILL` | 强制杀死 | **无法被捕获或忽略**，立即终止     |
| `15`     | `SIGTERM` | 终止     | 默认的 kill 信号，允许进程善后     |
| `18`     | `SIGCONT` | 继续     | 恢复被暂停的进程                   |
| `19`     | `SIGSTOP` | 停止     | `Ctrl+Z` 发送的信号，暂停进程      |
| `1`      | `SIGHUP`  | 挂起     | 终端关闭时发送，常用于重新加载配置 |

### 7.3.2 `kill` —— 发送信号给进程

```bash
kill [信号] PID
```

```bash
# 发送默认信号 SIGTERM（15）——优雅终止
kill 1234

# 发送 SIGKILL（9）——强制终止（杀不掉的进程用这个）
kill -9 1234

# 发送 SIGHUP（1）——让进程重新加载配置
kill -1 1234              # 例如让 Nginx 重新读取配置文件

# 查看所有可用信号
kill -l
```

**操作流程**：
```bash
# 1. 先找到目标进程的 PID
ps aux | grep nginx

# 2. 记下 PID（比如是 890）

# 3. 先尝试优雅终止
kill 890

# 4. 等几秒，检查是否已终止
ps aux | grep nginx

# 5. 如果还在运行，强制终止
kill -9 890
```

### 7.3.3 `killall` —— 按名称杀死进程

```bash
killall 进程名
```

```bash
# 按名称终止所有匹配的进程
killall nginx

# 强制终止
killall -9 python3

# 只终止特定用户的进程
killall -u datovm python3

# 交互式确认
killall -i nginx         # 每个进程都会问你 y/n
```

### 7.3.4 `pkill` —— 按模式杀死进程

```bash
# 按名称（支持部分匹配）
pkill nginx

# 按用户
pkill -u datovm

# 按终端
pkill -t pts/0           # 终止 pts/0 终端的所有进程
```

### 7.3.5 实际场景

**场景1：Python 脚本卡住了**
```bash
# 找到卡住的 Python 进程
ps aux | grep python
# datovm  2345  99.0  5.0  ... python3 compute.py

# 优雅终止
kill 2345

# 如果无效，强制终止
kill -9 2345
```

**场景2：端口被占用**
```bash
# 查看占用 8080 端口的进程
sudo lsof -i :8080
# COMMAND  PID    USER   FD   TYPE DEVICE ... NAME
# node     3456   datovm 22u  IPv4 ...       *:8080

# 终止该进程
kill 3456
```

---

## 7.4 磁盘与内存监控

### 7.4.1 `df` —— 磁盘空间使用情况

```bash
df                  # Disk Free
```

```bash
# 人类友好的显示（KB/MB/GB）
df -h

# 输出示例：
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sdc       1007G   12G  944G   2% /
# tmpfs           3.9G     0  3.9G   0% /mnt/wsl
# C:\             476G  289G  188G  61% /mnt/c
# D:\             466G  120G  346G  26% /mnt/d
```

| 列         | 含义          |
| ---------- | ------------- |
| Filesystem | 文件系统/分区 |
| Size       | 总大小        |
| Used       | 已使用        |
| Avail      | 可用          |
| Use%       | 使用百分比    |
| Mounted on | 挂载点        |

```bash
# 只看特定文件系统
df -h /

# 只看特定类型的文件系统（排除临时文件系统）
df -h -x tmpfs -x devtmpfs

# 查看 inode 使用情况（文件数量限制）
df -i
```

> 💡 **何时需要关注？** 当 `Use%` 超过 **80%** 时就需要注意了，超过 **90%** 应该立即清理。磁盘满会导致系统异常、服务崩溃。

### 7.4.2 `du` —— 目录/文件大小

```bash
du                  # Disk Usage
```

```bash
# 查看当前目录的大小
du -sh .
# 输出：152M    .

# 查看指定目录的大小
du -sh /var/log
# 输出：24M     /var/log

# 查看目录中每个子目录的大小
du -sh /var/*
# 4.0K    /var/backups
# 72M     /var/cache
# 4.0K    /var/crash
# 24M     /var/log
# ...

# 查看目录中最大的 10 个文件/目录
du -ah /var | sort -rh | head -10

# 查看当前目录下每个子目录的大小（只到第1层）
du -h --max-depth=1
```

> 💡 **常用命令**：当磁盘快满时，用以下命令找出最大的目录：
> ```bash
> sudo du -h --max-depth=1 / | sort -rh | head -20
> ```

### 7.4.3 `free` —— 内存使用情况

```bash
free                # 默认以 KB 显示

free -m             # 以 MB 显示
free -h             # 以人类友好格式显示（推荐）
```

输出示例：
```
              total        used        free      shared  buff/cache   available
Mem:          7.8Gi       802Mi       6.4Gi        0Mi       606Mi       6.7Gi
Swap:         2.0Gi          0B       2.0Gi
```

**各列含义**：

| 列           | 含义                           |
| ------------ | ------------------------------ |
| `total`      | 总内存                         |
| `used`       | 已使用的内存                   |
| `free`       | 完全空闲的内存                 |
| `shared`     | 共享内存                       |
| `buff/cache` | 缓冲区/缓存占用的内存          |
| `available`  | **实际可用的内存**（最重要！） |

> 💡 **important 区分**：
> - `free` 是"完全空闲"的内存，看起来可能很少
> - `buff/cache` 是 Linux 拿来做文件缓存的内存，需要时会自动释放
> - **`available` 才是你真正可用的内存量**，等于 `free` + 可释放的 `buff/cache`
> - 所以看到 `free` 很少不要慌，看 `available` 就好

```bash
# 持续监控内存（每 2 秒刷新）
watch -n 2 free -h
# 按 Ctrl+C 停止
```

### 7.4.4 WSL 内存特殊说明

**WSL2 的内存与 Windows 共享**，有一些特殊行为：

1. **WSL2 默认最多使用 Windows 总内存的 50%**（或 8GB，取较小值）
2. **WSL2 的内存不会主动归还给 Windows**（在旧版本中），可能导致 `vmmem` 进程在 Windows 任务管理器中占用大量内存

**查看 WSL 的内存占用**（在 Windows PowerShell 中）：
```powershell
# 查看 vmmem 进程（就是 WSL 虚拟机的内存占用）
Get-Process vmmem
```

**限制 WSL 内存**（在 Windows 中）：

编辑 `C:\Users\你的用户名\.wslconfig`：
```ini
[wsl2]
memory=4GB       # 最多使用 4GB
swap=2GB         # 交换空间 2GB
```

修改后重启 WSL：
```powershell
wsl --shutdown
```

**手动释放 WSL 内存**：
```bash
# 在 WSL 中清理缓存
sudo sh -c "echo 3 > /proc/sys/vm/drop_caches"
```

或者直接重启 WSL（在 PowerShell 中）：
```powershell
wsl --shutdown
# 然后重新打开 WSL
```

---

## 7.5 其他监控工具

### 7.5.1 `uptime` —— 系统运行时间

```bash
uptime
# 输出：10:30:05 up 2:30, 1 user, load average: 0.00, 0.01, 0.05
#        当前时间  运行时长  登录用户数    1/5/15分钟平均负载
```

### 7.5.2 `lsof` —— 查看打开的文件

```bash
# 查看哪个进程在使用某个文件
lsof /var/log/syslog

# 查看哪个进程在占用某个端口
sudo lsof -i :80
sudo lsof -i :8080

# 查看某个进程打开了哪些文件
lsof -p 1234

# 查看某个用户打开的所有文件
lsof -u datovm
```

### 7.5.3 `iostat` —— 磁盘 I/O 监控

```bash
# 安装
sudo apt install sysstat -y

# 查看磁盘 I/O
iostat

# 每 2 秒刷新
iostat 2
```

### 7.5.4 本章练习

```bash
# 1. 启动一个后台进程
sleep 600 &

# 2. 用 ps 找到它
ps aux | grep sleep

# 3. 记下 PID，用 kill 终止
kill <PID>

# 4. 查看磁盘使用情况
df -h

# 5. 查看内存使用情况
free -h

# 6. 打开 htop 观察系统
htop
# 在 htop 中尝试：
#   按 F6 排序
#   按 F3 搜索 "bash"
#   按 F5 查看进程树
#   按 q 退出
```

---

# 第八章：网络基础

---

## 8.1 Linux 网络基础概念

### 8.1.1 必须知道的网络术语

| 术语             | 解释                                    | 类比           |
| ---------------- | --------------------------------------- | -------------- |
| **IP 地址**      | 设备在网络中的唯一标识                  | 家庭住址       |
| **端口（Port）** | 一台设备上不同服务的"门牌号"（0-65535） | 公寓楼的房间号 |
| **DNS**          | 将域名（如 google.com）翻译成 IP 地址   | 电话簿         |
| **TCP**          | 可靠的连接协议（确保数据完整送达）      | 挂号信         |
| **UDP**          | 快速但不保证送达的协议                  | 普通信         |
| **HTTP/HTTPS**   | 网页传输协议，默认端口 80/443           | —              |
| **SSH**          | 安全远程连接协议，默认端口 22           | —              |
| **localhost**    | 本机地址，等同于 `127.0.0.1`            | "家"           |

**常见端口号**：

| 端口 | 服务         | 说明           |
| ---- | ------------ | -------------- |
| 22   | SSH          | 远程登录       |
| 80   | HTTP         | 网页（不加密） |
| 443  | HTTPS        | 网页（加密）   |
| 3306 | MySQL        | 数据库         |
| 5432 | PostgreSQL   | 数据库         |
| 6379 | Redis        | 缓存           |
| 8080 | 备用 HTTP    | 开发常用       |
| 3000 | Node.js 默认 | 开发常用       |

---

## 8.2 网络查看命令

### 8.2.1 `ip addr` —— 查看网络接口和 IP 地址

```bash
ip addr
# 或简写
ip a
```

输出示例：
```
1: lo: <LOOPBACK,UP> mtu 65536
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500
    inet 172.25.123.45/20 brd 172.25.127.255 scope global eth0
    inet6 fe80::215:5dff:fe12:3456/64 scope link
```

**解读**：

| 接口   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `lo`   | **回环接口**（Loopback），IP 是 `127.0.0.1`，用于本机自己访问自己 |
| `eth0` | **以太网接口**，这里的 IP 是 `172.25.123.45`，是 WSL 的虚拟网卡 |

```bash
# 只看 IP 地址（精简版）
ip -4 addr show eth0
# 或更简洁
hostname -I
```

> 💡 **旧版命令 `ifconfig`**：很多教程还在用 `ifconfig`，它需要额外安装：
> ```bash
> sudo apt install net-tools -y
> ifconfig
> ```
> 建议用新命令 `ip addr`，但知道 `ifconfig` 的存在即可。

### 8.2.2 `ping` —— 网络连通性测试

```bash
ping 目标
```

```bash
# 测试能否连接到外网
ping google.com
# PING google.com (142.250.185.206) 56(84) bytes of data.
# 64 bytes from ... : icmp_seq=1 ttl=115 time=15.2 ms
# 64 bytes from ... : icmp_seq=2 ttl=115 time=14.8 ms
# 按 Ctrl+C 停止

# 只 ping 4 次
ping -c 4 google.com

# ping 本机
ping localhost

# ping 指定 IP
ping 192.168.1.1
```

**输出解读**：
- `time=15.2 ms`：往返时间（延迟），越小越好
- `ttl=115`：数据包生存时间
- `icmp_seq`：序列号，连续表示没有丢包

**如果 ping 不通**：
```
ping: google.com: Name or service not known     → DNS 问题
Request timeout                                  → 网络不通或对方防火墙阻止
```

### 8.2.3 `curl` —— HTTP 请求工具

`curl` 是一个强大的命令行 HTTP 客户端，能发送各种 HTTP 请求。

```bash
# 获取网页内容
curl https://httpbin.org/ip
# 输出你的公网 IP

# 只看 HTTP 响应头
curl -I https://www.google.com

# 下载文件
curl -o filename.ext https://example.com/file.ext

# 跟随重定向（-L）
curl -L https://example.com

# 发送 POST 请求
curl -X POST https://httpbin.org/post -d "name=linux&age=35"

# 发送 JSON 数据
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"name": "linux", "version": 6}'

# 显示详细的请求/响应信息
curl -v https://www.google.com

# 安静模式（不显示进度条）
curl -s https://httpbin.org/ip

# 测试本地服务
curl http://localhost:80
curl http://localhost:3000/api/health
```

### 8.2.4 `wget` —— 下载工具

```bash
# 下载文件
wget https://example.com/file.tar.gz

# 指定保存文件名
wget -O newname.tar.gz https://example.com/file.tar.gz

# 断点续传（下载中断后继续）
wget -c https://example.com/large-file.zip

# 后台下载
wget -b https://example.com/large-file.zip
# 下载日志保存在 wget-log 中

# 下载整个网站（谨慎使用）
wget -r -np https://example.com/docs/
```

### 8.2.5 `ss` / `netstat` —— 查看网络连接和端口

```bash
# 查看所有监听的端口（推荐用 ss）
ss -tlnp
# -t = TCP
# -l = 监听状态
# -n = 显示端口号（不转换成服务名）
# -p = 显示进程信息
```

输出示例：
```
State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
LISTEN  0       511     0.0.0.0:80          0.0.0.0:*          users:(("nginx",pid=890,...))
LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=456,...))
```

```bash
# 查看所有连接（包含已建立的）
ss -tanp

# 查看 UDP 端口
ss -ulnp

# 旧命令 netstat（需要 net-tools）
sudo netstat -tlnp
```

### 8.2.6 `nslookup` / `dig` —— DNS 查询

```bash
# 查询域名对应的 IP
nslookup google.com
# 输出：
# Name: google.com
# Address: 142.250.185.206

# 更详细的 DNS 查询（需安装 dnsutils）
sudo apt install dnsutils -y
dig google.com

# 反向查询（IP → 域名）
nslookup 8.8.8.8
```

---

## 8.3 防火墙：`ufw`

### 8.3.1 什么是防火墙？

防火墙控制哪些网络流量可以进入或离开你的系统。

`ufw`（Uncomplicated Firewall）是 Ubuntu 提供的**简化版防火墙工具**。

### 8.3.2 基本操作

```bash
# 查看防火墙状态
sudo ufw status
# 输出：Status: inactive（默认是关闭的）

# 启用防火墙
sudo ufw enable
# 提示让你确认（因为可能断开 SSH），输入 y

# 关闭防火墙
sudo ufw disable

# 查看详细状态
sudo ufw status verbose

# 查看带编号的规则
sudo ufw status numbered
```

### 8.3.3 添加和删除规则

```bash
# 允许 SSH 连接（端口 22）
sudo ufw allow 22
# 或
sudo ufw allow ssh

# 允许 HTTP（端口 80）
sudo ufw allow 80
sudo ufw allow http

# 允许 HTTPS（端口 443）
sudo ufw allow 443
sudo ufw allow https

# 允许特定端口范围
sudo ufw allow 3000:3100/tcp

# 允许特定 IP 访问
sudo ufw allow from 192.168.1.100

# 允许特定 IP 访问特定端口
sudo ufw allow from 192.168.1.100 to any port 22

# 拒绝某个端口
sudo ufw deny 8080

# 删除规则（先查看编号）
sudo ufw status numbered
sudo ufw delete 3            # 删除编号为 3 的规则

# 重置所有规则
sudo ufw reset
```

### 8.3.4 典型的防火墙配置

```bash
# Web 服务器的标准配置
sudo ufw default deny incoming     # 默认拒绝所有入站
sudo ufw default allow outgoing    # 默认允许所有出站
sudo ufw allow ssh                 # 允许 SSH
sudo ufw allow http                # 允许 HTTP
sudo ufw allow https               # 允许 HTTPS
sudo ufw enable                    # 启用

# 查看最终结果
sudo ufw status
```

> 💡 **WSL 中的防火墙**：WSL 环境中，防火墙的作用有限（因为网络走的是 Windows 的网络栈）。但了解 `ufw` 的使用对你将来管理真正的 Linux 服务器非常重要。

---

## 8.4 WSL 网络特性

### 8.4.1 WSL2 的网络架构

```
┌──────────────────────────────────────────┐
│  Windows 主机                            │
│                                          │
│   浏览器 ──→ localhost:80 ──→──┐         │
│                                │         │
│  ┌─────────────────────────────┼───────┐ │
│  │  WSL2 虚拟机               ↓       │ │
│  │                                     │ │
│  │    Nginx 监听 0.0.0.0:80            │ │
│  │    IP: 172.25.xxx.xxx               │ │
│  │                                     │ │
│  └─────────────────────────────────────┘ │
│                                          │
└──────────────────────────────────────────┘
```

**关键点**：
- WSL2 有自己的 IP 地址（通常是 `172.x.x.x`），和 Windows 不在同一个网段
- **但 Windows 可以通过 `localhost` 直接访问 WSL 中的服务**（微软做了自动端口转发）
- 这意味着：WSL 中启动一个 Web 服务器监听 `0.0.0.0:80`，Windows 浏览器直接访问 `http://localhost:80` 就能看到

### 8.4.2 从 Windows 访问 WSL 服务

**大多数情况下，直接用 `localhost` 就行**：

```bash
# 在 WSL 中启动一个简单的 Python HTTP 服务器
cd /tmp
echo "<h1>Hello from WSL!</h1>" > index.html
python3 -m http.server 8080
```

然后在 Windows 浏览器中访问：`http://localhost:8080`

你应该能看到 "Hello from WSL!" 🎉

### 8.4.3 查看 WSL 的 IP

```bash
# 在 WSL 中查看自己的 IP
ip addr show eth0 | grep "inet "
# inet 172.25.123.45/20 ...

# 或者简单点
hostname -I
```

```powershell
# 在 Windows PowerShell 中查看 WSL 的 IP
wsl hostname -I
```

### 8.4.4 从 WSL 访问 Windows 服务

如果 Windows 上运行着某个服务（比如数据库），WSL 需要通过 Windows 的 IP 来访问：

```bash
# 获取 Windows 主机的 IP（从 WSL 的角度）
cat /etc/resolv.conf | grep nameserver | awk '{print $2}'
# 输出类似：172.25.112.1

# 用这个 IP 访问 Windows 上的服务
curl http://172.25.112.1:3000
```

---

## 8.5 实战：WSL 启动 Nginx + Windows 浏览器访问

```bash
# 1. 安装 Nginx（如果还没装）
sudo apt install nginx -y

# 2. 创建自定义网页
sudo bash -c 'cat > /var/www/html/index.html << EOF
<!DOCTYPE html>
<html>
<head>
    <title>My WSL Server</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
        }
        .container {
            text-align: center;
            padding: 40px;
            background: rgba(255,255,255,0.1);
            border-radius: 20px;
            backdrop-filter: blur(10px);
        }
        h1 { font-size: 3em; margin-bottom: 10px; }
        p { font-size: 1.3em; opacity: 0.8; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🐧 Hello from WSL2!</h1>
        <p>Nginx is running inside your Linux subsystem.</p>
        <p>Server Time: $(date)</p>
    </div>
</body>
</html>
EOF'

# 3. 启动 Nginx
sudo service nginx start

# 4. 验证 Nginx 运行状态
sudo service nginx status

# 5. 本地测试
curl http://localhost

# 6. 打开 Windows 浏览器，访问 http://localhost
#    你应该看到一个漂亮的紫色渐变页面！🎉

# 7. 实验结束后停止 Nginx
sudo service nginx stop
```

---

# 第九章：Shell 脚本入门

---

## 9.1 什么是 Shell 脚本？

### 9.1.1 概念

**Shell 脚本**就是把一系列 Linux 命令写在一个文件里，然后一次性执行。

就像 Windows 的 `.bat` 批处理文件，但功能强大得多。

**为什么需要 Shell 脚本？**
- **自动化**：不用每次手动输入一堆命令
- **可复用**：写一次，到处运行
- **可维护**：比起口头传达"先执行这个再执行那个"，脚本更可靠
- **定时执行**：配合 `crontab`，可以让脚本定时自动运行

### 9.1.2 第一个脚本

```bash
# 1. 创建脚本文件
nano ~/hello.sh
```

输入以下内容：

```bash
#!/bin/bash
# 这是我的第一个 Shell 脚本
# 以 # 开头的行是注释

echo "Hello, World!"
echo "当前时间: $(date)"
echo "当前用户: $(whoami)"
echo "当前目录: $(pwd)"
```

```bash
# 2. 保存退出（Ctrl+O, Enter, Ctrl+X）

# 3. 添加执行权限
chmod +x ~/hello.sh

# 4. 运行脚本
~/hello.sh
# 或者
bash ~/hello.sh    # 不需要执行权限也能运行
```

输出：
```
Hello, World!
当前时间: Mon Apr 13 22:00:00 CST 2026
当前用户: datovm
当前目录: /home/datovm
```

### 9.1.3 `#!/bin/bash` 是什么？

第一行 `#!/bin/bash` 叫做 **Shebang**（也叫 Hashbang）：
- `#!` 告诉系统"用后面指定的程序来解释这个脚本"
- `/bin/bash` 表示用 Bash 解释器来运行

常见的 Shebang：
```bash
#!/bin/bash         # 用 Bash 运行
#!/bin/sh           # 用系统默认 Shell 运行
#!/usr/bin/env bash # 更通用的写法（自动查找 bash 的路径）
#!/usr/bin/python3  # Python 脚本
#!/usr/bin/env node # Node.js 脚本
```

---

## 9.2 变量

### 9.2.1 定义和使用变量

```bash
#!/bin/bash

# 定义变量（等号两边不能有空格！）
name="Linux"
version=6
today=$(date +%Y-%m-%d)       # 命令的输出赋值给变量

# 使用变量（用 $ 或 ${} 引用）
echo "操作系统: $name"
echo "版本: ${version}"
echo "今天是: $today"

# ⚠️ 常见错误：等号两边加了空格
# name = "Linux"    # ❌ 错误！会被解释为命令
# name="Linux"      # ✅ 正确！
```

### 9.2.2 变量类型

```bash
#!/bin/bash

# 字符串
greeting="Hello World"

# 数字（Bash 中变量默认都是字符串，做运算需要特殊语法）
count=42

# 命令结果
current_dir=$(pwd)
file_count=$(ls | wc -l)

# 环境变量（系统预定义的）
echo "主目录: $HOME"
echo "当前用户: $USER"
echo "Shell 类型: $SHELL"
echo "系统路径: $PATH"
```

### 9.2.3 字符串操作

```bash
#!/bin/bash

name="Linux"
full_name="GNU/Linux"

# 字符串拼接
greeting="Hello, "$name"!"
echo $greeting                    # Hello, Linux!

# 获取字符串长度
echo "长度: ${#name}"             # 长度: 5

# 字符串截取
echo "${full_name:0:3}"           # GNU（从位置0开始，取3个字符）
echo "${full_name:4}"             # Linux（从位置4开始到末尾）

# 字符串替换
path="/home/user/file.txt"
echo "${path/user/datovm}"        # /home/datovm/file.txt（替换第一个匹配）
echo "${path%.txt}"               # /home/user/file（删除末尾的 .txt）
echo "${path##*/}"                # file.txt（删除最后一个 / 前的所有内容）
```

### 9.2.4 特殊变量

```bash
#!/bin/bash

echo "脚本名: $0"                # 脚本自身的名字
echo "第一个参数: $1"             # 第一个命令行参数
echo "第二个参数: $2"             # 第二个命令行参数
echo "所有参数: $@"               # 所有参数（常用）
echo "参数个数: $#"               # 参数的数量
echo "上一条命令的退出码: $?"      # 0=成功, 非0=失败
echo "当前进程 PID: $$"
```

使用示例：
```bash
# 保存为 greet.sh 后运行：
# ./greet.sh Alice Bob
# 输出：
# 脚本名: ./greet.sh
# 第一个参数: Alice
# 第二个参数: Bob
# 所有参数: Alice Bob
# 参数个数: 2
```

### 9.2.5 用户输入

```bash
#!/bin/bash

# 读取用户输入
echo -n "请输入你的名字: "
read username
echo "你好, $username!"

# 带提示符的 read
read -p "请输入年龄: " age
echo "你今年 $age 岁"

# 读取密码（不显示输入）
read -s -p "请输入密码: " password
echo ""
echo "密码长度: ${#password}"

# 设置超时（5秒内没输入就用默认值）
read -t 5 -p "5秒内输入名字（默认Guest）: " name
name=${name:-Guest}    # 如果 name 为空，则设为 Guest
echo "欢迎, $name"
```

---

## 9.3 条件判断 `if`

### 9.3.1 基本语法

```bash
if [ 条件 ]; then
    # 条件为真时执行
    命令1
    命令2
elif [ 另一个条件 ]; then
    # 第一个条件为假，这个条件为真时执行
    命令3
else
    # 所有条件都为假时执行
    命令4
fi
```

> ⚠️ **空格非常重要！ `[` 后面和 `]` 前面必须有空格！**
> ```bash
> if [ $a -eq 1 ]; then    # ✅ 正确
> if [$a -eq 1]; then      # ❌ 错误！
> ```

### 9.3.2 数值比较

| 运算符 | 含义                        | 示例            |
| ------ | --------------------------- | --------------- |
| `-eq`  | 等于 (equal)                | `[ $a -eq $b ]` |
| `-ne`  | 不等于 (not equal)          | `[ $a -ne $b ]` |
| `-gt`  | 大于 (greater than)         | `[ $a -gt $b ]` |
| `-lt`  | 小于 (less than)            | `[ $a -lt $b ]` |
| `-ge`  | 大于等于 (greater or equal) | `[ $a -ge $b ]` |
| `-le`  | 小于等于 (less or equal)    | `[ $a -le $b ]` |

```bash
#!/bin/bash

read -p "请输入你的成绩: " score

if [ "$score" -ge 90 ]; then
    echo "优秀！🏆"
elif [ "$score" -ge 80 ]; then
    echo "良好 👍"
elif [ "$score" -ge 60 ]; then
    echo "及格 ✅"
else
    echo "不及格 ❌ 继续努力！"
fi
```

> 💡 **为什么变量要加双引号？** `"$score"` 而不是 `$score`：如果用户没有输入任何内容，`$score` 为空，`[ -ge 90 ]` 会语法错误，而 `[ "" -ge 90 ]` 虽然也会报错但更安全。**养成给变量加引号的习惯。**

### 9.3.3 字符串比较

| 运算符      | 含义         | 示例               |
| ----------- | ------------ | ------------------ |
| `=` 或 `==` | 字符串相等   | `[ "$a" = "$b" ]`  |
| `!=`        | 字符串不相等 | `[ "$a" != "$b" ]` |
| `-z`        | 字符串为空   | `[ -z "$a" ]`      |
| `-n`        | 字符串不为空 | `[ -n "$a" ]`      |

```bash
#!/bin/bash

read -p "请输入操作系统名: " os

if [ "$os" = "linux" ] || [ "$os" = "Linux" ]; then
    echo "Linux 大法好！🐧"
elif [ "$os" = "windows" ] || [ "$os" = "Windows" ]; then
    echo "Windows 也不错"
elif [ -z "$os" ]; then
    echo "你没有输入任何东西"
else
    echo "你输入的是: $os"
fi
```

### 9.3.4 文件测试

| 运算符    | 含义                       |
| --------- | -------------------------- |
| `-e 文件` | 文件**存在**               |
| `-f 文件` | 是**普通文件**             |
| `-d 文件` | 是**目录**                 |
| `-r 文件` | 文件**可读**               |
| `-w 文件` | 文件**可写**               |
| `-x 文件` | 文件**可执行**             |
| `-s 文件` | 文件**不为空**（大小 > 0） |

```bash
#!/bin/bash

file="/etc/passwd"

if [ -e "$file" ]; then
    echo "文件 $file 存在 ✅"
    
    if [ -r "$file" ]; then
        echo "  可读 ✅"
    fi
    if [ -w "$file" ]; then
        echo "  可写 ✅"
    else
        echo "  不可写 ❌"
    fi
else
    echo "文件 $file 不存在 ❌"
fi

# 实用：检查目录是否存在，不存在就创建
dir="$HOME/backup"
if [ ! -d "$dir" ]; then
    echo "目录不存在，正在创建..."
    mkdir -p "$dir"
    echo "创建完成：$dir"
else
    echo "目录已存在：$dir"
fi
```

### 9.3.5 逻辑运算

```bash
# 在 [ ] 内部
[ 条件1 -a 条件2 ]       # AND（且）
[ 条件1 -o 条件2 ]       # OR（或）
[ ! 条件 ]               # NOT（非）

# 在 [ ] 外部（推荐）
[ 条件1 ] && [ 条件2 ]    # AND
[ 条件1 ] || [ 条件2 ]    # OR

# 使用双括号 [[ ]]（Bash 增强版，更灵活）
[[ $a -gt 0 && $a -lt 100 ]]    # AND
[[ $a = "hello" || $a = "hi" ]] # OR
```

### 9.3.6 `[[ ]]` vs `[ ]`

| 特性                 | `[ ]` (test)        | `[[ ]]` (Bash 增强)      |
| -------------------- | ------------------- | ------------------------ |
| POSIX 兼容           | ✅ 所有 Shell 都支持 | ❌ 仅 Bash/Zsh            |
| `&&` `\|\|` 用在内部 | ❌                   | ✅                        |
| 模式匹配             | ❌                   | ✅ `[[ $a == h* ]]`       |
| 正则匹配             | ❌                   | ✅ `[[ $a =~ ^[0-9]+$ ]]` |
| 变量不加引号         | 可能出错            | 安全                     |

**推荐在 Bash 脚本中使用 `[[ ]]`**。

---

## 9.4 循环

### 9.4.1 `for` 循环

```bash
# 基本语法
for 变量 in 列表; do
    命令
done
```

**示例**：

```bash
#!/bin/bash

# 遍历一组值
for fruit in apple banana cherry; do
    echo "水果: $fruit"
done

# 遍历数字范围
for i in {1..5}; do
    echo "数字: $i"
done

# 带步长的范围
for i in {0..20..5}; do
    echo "数字: $i"        # 0, 5, 10, 15, 20
done

# C 风格 for 循环
for ((i=1; i<=5; i++)); do
    echo "计数: $i"
done

# 遍历文件
for file in /etc/*.conf; do
    echo "配置文件: $file"
done

# 遍历命令输出
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "用户: $user"
done
```

### 9.4.2 `while` 循环

```bash
# 基本语法
while [ 条件 ]; do
    命令
done
```

```bash
#!/bin/bash

# 基本计数
count=1
while [ $count -le 5 ]; do
    echo "第 $count 次循环"
    count=$((count + 1))    # 数学运算用 $((...))
done

# 逐行读取文件（非常常用！）
while IFS= read -r line; do
    echo "行内容: $line"
done < /etc/hosts

# 无限循环（常用于监控脚本）
while true; do
    echo "系统时间: $(date)"
    sleep 5                 # 每 5 秒执行一次
    # 按 Ctrl+C 停止
done
```

### 9.4.3 循环控制

```bash
#!/bin/bash

# break —— 跳出循环
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        echo "到 5 了，退出循环"
        break
    fi
    echo "当前: $i"
done

# continue —— 跳过本次，继续下一次
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue              # 跳过偶数
    fi
    echo "奇数: $i"           # 只打印奇数
done
```

---

## 9.5 函数

```bash
#!/bin/bash

# 定义函数
greet() {
    local name=$1            # local 声明局部变量
    echo "你好, $name! 欢迎学习 Shell 脚本"
}

# 带返回值的函数
add() {
    local a=$1
    local b=$2
    local sum=$((a + b))
    echo $sum                # 通过 echo 返回结果
}

# 调用函数
greet "Alice"
greet "Bob"

# 获取函数返回值
result=$(add 10 20)
echo "10 + 20 = $result"

# 检查命令是否成功的函数
check_command() {
    if command -v "$1" &> /dev/null; then
        echo "✅ $1 已安装（$(command -v $1)）"
        return 0
    else
        echo "❌ $1 未安装"
        return 1
    fi
}

check_command git
check_command nginx
check_command nonexistent_tool
```

---

## 9.6 实战：自动备份脚本

这是一个完整的、实用的备份脚本。逐段解释：

```bash
#!/bin/bash
#
# 自动备份脚本
# 功能：将指定目录压缩备份，保留最近 7 天的备份
# 用法：./backup.sh /path/to/source
#

# ========== 配置 ==========
BACKUP_DIR="$HOME/backups"          # 备份文件存放目录
KEEP_DAYS=7                         # 保留最近几天的备份
DATE=$(date +%Y%m%d_%H%M%S)        # 当前日期时间，格式：20260413_220000
LOG_FILE="$BACKUP_DIR/backup.log"   # 日志文件

# ========== 函数 ==========

# 日志函数
log() {
    local message="[$(date '+%Y-%m-%d %H:%M:%S')] $1"
    echo "$message"
    echo "$message" >> "$LOG_FILE"
}

# 检查参数
check_args() {
    if [ $# -eq 0 ]; then
        echo "用法: $0 <要备份的目录>"
        echo "示例: $0 /home/datovm/projects"
        exit 1
    fi

    if [ ! -d "$1" ]; then
        echo "错误: 目录 '$1' 不存在！"
        exit 1
    fi
}

# ========== 主逻辑 ==========

# 检查参数
check_args "$@"

SOURCE_DIR="$1"
DIR_NAME=$(basename "$SOURCE_DIR")
BACKUP_FILE="$BACKUP_DIR/${DIR_NAME}_${DATE}.tar.gz"

# 创建备份目录
mkdir -p "$BACKUP_DIR"

log "========== 开始备份 =========="
log "源目录: $SOURCE_DIR"
log "备份文件: $BACKUP_FILE"

# 执行压缩备份
tar -czf "$BACKUP_FILE" -C "$(dirname "$SOURCE_DIR")" "$DIR_NAME" 2>> "$LOG_FILE"

# 检查备份是否成功
if [ $? -eq 0 ]; then
    FILE_SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    log "✅ 备份成功！文件大小: $FILE_SIZE"
else
    log "❌ 备份失败！请检查日志"
    exit 1
fi

# 清理过期备份
log "正在清理 ${KEEP_DAYS} 天前的旧备份..."
deleted_count=0
while IFS= read -r old_file; do
    rm -f "$old_file"
    log "  已删除: $(basename "$old_file")"
    deleted_count=$((deleted_count + 1))
done < <(find "$BACKUP_DIR" -name "${DIR_NAME}_*.tar.gz" -mtime +$KEEP_DAYS)

log "已清理 $deleted_count 个过期备份"

# 显示当前备份文件
log "当前备份列表:"
ls -lh "$BACKUP_DIR"/${DIR_NAME}_*.tar.gz 2>/dev/null | while read -r line; do
    log "  $line"
done

log "========== 备份完成 =========="
```

**使用方式**：
```bash
# 1. 保存上面的内容到 backup.sh
nano ~/backup.sh
# （粘贴上面的内容）

# 2. 添加执行权限
chmod +x ~/backup.sh

# 3. 创建一些测试文件
mkdir -p ~/test_project
echo "Hello" > ~/test_project/file1.txt
echo "World" > ~/test_project/file2.txt

# 4. 运行备份
~/backup.sh ~/test_project

# 5. 查看备份结果
ls -lh ~/backups/
```

---

## 9.7 定时任务：`crontab`

### 9.7.1 什么是 Cron？

`cron` 是 Linux 的定时任务调度器，可以按照设定的时间自动执行命令或脚本。

### 9.7.2 Crontab 时间格式

```
┌──────────── 分钟 (0-59)
│ ┌────────── 小时 (0-23)
│ │ ┌──────── 日期 (1-31)
│ │ │ ┌────── 月份 (1-12)
│ │ │ │ ┌──── 星期 (0-7, 0和7都是星期天)
│ │ │ │ │
* * * * *  命令
```

**特殊符号**：

| 符号 | 含义   | 示例                                         |
| ---- | ------ | -------------------------------------------- |
| `*`  | 任意值 | `* * * * *`（每分钟）                        |
| `,`  | 列表   | `1,15,30 * * * *`（每小时的第1、15、30分钟） |
| `-`  | 范围   | `1-5 * * * *`（第1到第5分钟）                |
| `/`  | 间隔   | `*/5 * * * *`（每5分钟）                     |

**常用示例**：

```bash
# 每分钟执行
* * * * * /path/to/script.sh

# 每小时执行（每小时的第0分钟）
0 * * * * /path/to/script.sh

# 每天凌晨 2:30 执行
30 2 * * * /path/to/script.sh

# 每天上午 9 点和下午 6 点执行
0 9,18 * * * /path/to/script.sh

# 每周一到周五上午 9 点执行
0 9 * * 1-5 /path/to/script.sh

# 每月 1 号凌晨 0 点执行
0 0 1 * * /path/to/script.sh

# 每 5 分钟执行
*/5 * * * * /path/to/script.sh

# 每 30 分钟执行
*/30 * * * * /path/to/script.sh
```

### 9.7.3 管理定时任务

```bash
# 编辑当前用户的定时任务（打开编辑器）
crontab -e
# 第一次会让你选择编辑器，选 nano（对新手最友好）

# 查看当前用户的定时任务
crontab -l

# 删除当前用户的所有定时任务（谨慎！）
crontab -r

# 编辑其他用户的定时任务（需要 root）
sudo crontab -u username -e
```

### 9.7.4 实用示例

```bash
# 编辑定时任务
crontab -e

# 在打开的编辑器中添加以下内容：

# 每天凌晨 3 点执行备份脚本
0 3 * * * /home/datovm/backup.sh /home/datovm/projects >> /home/datovm/backups/cron.log 2>&1

# 每 10 分钟记录一次系统资源状态
*/10 * * * * echo "$(date): $(free -h | grep Mem)" >> /home/datovm/system_monitor.log

# 每周日凌晨清理临时文件
0 0 * * 0 find /tmp -mtime +7 -delete
```

> 💡 **注意事项**：
> - Cron 任务使用的是**绝对路径**，因为 cron 不会加载你的 `.bashrc`
> - 输出重定向 `>> log 2>&1` 把标准输出和错误都记录到日志
> - 在 WSL 中，cron 服务可能没有自动启动

### 9.7.5 在 WSL 中启动 Cron

```bash
# 检查 cron 是否在运行
sudo service cron status

# 启动 cron
sudo service cron start

# 验证
sudo service cron status
# 应该显示 * cron is running
```

### 9.7.6 WSL 开机自启脚本

WSL 不像传统 Linux 服务器那样有完整的开机流程，但可以通过 `/etc/wsl.conf` 配置启动命令：

```bash
# 编辑 wsl.conf
sudo nano /etc/wsl.conf
```

添加以下内容：
```ini
[boot]
# 启用 systemd（推荐，较新版本 WSL 支持）
systemd=true

# 也可以用 command 指定启动命令
# command = service cron start && service nginx start
```

如果不支持 systemd，可以用另一种方法——在 Windows 的任务计划程序中，设置在登录时运行：

```powershell
# 在 PowerShell 中执行
wsl -u root service cron start
```

或者手动在每次打开 WSL 时执行：

```bash
# 在 ~/.bashrc 末尾添加（不太推荐，但简单有效）
# 检查 cron 是否运行，没运行就启动它
if ! pgrep -x "cron" > /dev/null; then
    sudo service cron start > /dev/null 2>&1
fi
```

---

# 第十章：综合实战项目

---

## 10.1 项目一：在 WSL 部署静态网站

### 目标

在 WSL 中搭建 Nginx 服务器，部署一个自定义的静态网站，然后在 Windows 浏览器中访问。

### 步骤

```bash
# ========== 1. 安装 Nginx ==========
sudo apt update && sudo apt install nginx -y

# ========== 2. 创建网站目录 ==========
sudo mkdir -p /var/www/mysite

# ========== 3. 创建网站文件 ==========
sudo bash -c 'cat > /var/www/mysite/index.html << "HTMLEOF"
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的 WSL 网站</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: "Segoe UI", Arial, sans-serif;
            background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
            color: #fff;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        .card {
            background: rgba(255,255,255,0.05);
            border: 1px solid rgba(255,255,255,0.1);
            border-radius: 20px;
            padding: 40px 60px;
            backdrop-filter: blur(10px);
            text-align: center;
            max-width: 600px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            background: linear-gradient(to right, #f7971e, #ffd200);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        p { line-height: 1.8; opacity: 0.8; margin: 10px 0; }
        .tech-stack {
            display: flex;
            gap: 15px;
            justify-content: center;
            margin-top: 20px;
            flex-wrap: wrap;
        }
        .badge {
            background: rgba(255,255,255,0.1);
            padding: 8px 20px;
            border-radius: 30px;
            font-size: 0.9em;
            border: 1px solid rgba(255,255,255,0.15);
        }
        .footer { margin-top: 40px; opacity: 0.4; font-size: 0.85em; }
    </style>
</head>
<body>
    <div class="card">
        <h1>🐧 WSL2 静态网站</h1>
        <p>这个网站运行在 WSL2 的 Nginx 服务器上</p>
        <p>从 Windows 浏览器通过 localhost 访问</p>
        <div class="tech-stack">
            <span class="badge">Ubuntu</span>
            <span class="badge">Nginx</span>
            <span class="badge">WSL2</span>
            <span class="badge">HTML/CSS</span>
        </div>
    </div>
    <p class="footer">Powered by Linux on Windows 💪</p>
</body>
</html>
HTMLEOF'
```

```bash
# ========== 4. 配置 Nginx ==========
sudo bash -c 'cat > /etc/nginx/sites-available/mysite << "CONFEOF"
server {
    listen 80;
    server_name localhost;

    root /var/www/mysite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # 启用 gzip 压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
}
CONFEOF'

# 启用站点（创建符号链接）
sudo ln -sf /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

# 删除默认站点（可选）
sudo rm -f /etc/nginx/sites-enabled/default

# 测试配置语法
sudo nginx -t
# 应该输出：syntax is ok / test is successful

# ========== 5. 启动 Nginx ==========
sudo service nginx restart

# ========== 6. 验证 ==========
curl http://localhost
# 应该返回你的 HTML 内容

echo ""
echo "✅ 部署完成！请在 Windows 浏览器中访问 http://localhost"
```

打开 Windows 浏览器，输入 `http://localhost`，你将看到你的自定义静态网站！

---

## 10.2 项目二：VS Code + WSL 完整开发环境搭建

### 目标

搭建一个可以进行 Python + Node.js 开发的完整本地环境。

### 步骤

```bash
# ========== 1. 基础工具安装 ==========
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget build-essential

# ========== 2. 配置 Git ==========
git config --global user.name "你的名字"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main
git config --global core.autocrlf input    # 重要！处理 Windows/Linux 换行符差异

# 验证
git config --list

# ========== 3. 安装 Python ==========
sudo apt install -y python3 python3-pip python3-venv

# 设置 pip 镜像源（加速下载）
mkdir -p ~/.pip
cat > ~/.pip/pip.conf << 'EOF'
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF

# 验证
python3 --version
pip3 --version

# ========== 4. 安装 Node.js（通过 nvm）==========
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
source ~/.bashrc
nvm install --lts

# 设置 npm 镜像源（可选，国内加速）
npm config set registry https://registry.npmmirror.com

# 验证
node -v
npm -v

# ========== 5. 安装其他有用工具 ==========
sudo apt install -y \
    htop tree zip unzip \
    jq \                   # JSON 处理工具
    ripgrep                # 超快的搜索工具

# ========== 6. 配置 Shell 环境 ==========
# 添加一些实用别名到 .bashrc
cat >> ~/.bashrc << 'EOF'

# ========= 自定义别名 =========
alias ll='ls -alFh'
alias la='ls -A'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'
alias gs='git status'
alias gl='git log --oneline -10'
alias ga='git add .'
alias gc='git commit -m'
alias gp='git push'
alias cls='clear'
alias ports='ss -tlnp'
alias myip='hostname -I'

# 让 rm 更安全
alias rm='rm -i'

# 彩色 grep
alias grep='grep --color=auto'
EOF

# 重新加载配置
source ~/.bashrc

# ========== 7. 创建标准项目结构 ==========
mkdir -p ~/projects
cd ~/projects

echo ""
echo "✅ 开发环境搭建完成！"
echo "   Python: $(python3 --version)"
echo "   Node.js: $(node -v)"
echo "   npm: $(npm -v)"
echo "   Git: $(git --version)"
echo ""
echo "💡 使用方式："
echo "   cd ~/projects && mkdir my-app && cd my-app && code ."
```

---

## 10.3 项目三：Windows ↔ Linux 自动同步备份脚本

### 目标

编写一个脚本，将 Windows 中指定目录的文件自动同步备份到 WSL 的 Linux 文件系统中。

### 脚本

```bash
#!/bin/bash
#
# sync_backup.sh
# 功能：将 Windows 指定目录同步到 Linux 备份目录
# 支持增量备份（只复制有变化的文件）
#

# ==================== 配置区 ====================

# Windows 源目录（通过 /mnt/c 访问）
# 修改为你自己的目录！
WIN_SOURCE="/mnt/c/Users/datovm/Documents/important_files"

# Linux 备份目标目录
BACKUP_BASE="$HOME/win_backup"

# 日志文件
LOG_FILE="$BACKUP_BASE/sync.log"

# 保留几份历史快照
MAX_SNAPSHOTS=5

# ==================== 函数区 ====================

# 带时间戳的日志
log() {
    local timestamp
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] $1" | tee -a "$LOG_FILE"
}

# 检查源目录是否存在
check_source() {
    if [ ! -d "$WIN_SOURCE" ]; then
        log "❌ 错误: 源目录不存在: $WIN_SOURCE"
        log "   请修改脚本中的 WIN_SOURCE 变量"
        exit 1
    fi
}

# 获取目录大小
get_dir_size() {
    du -sh "$1" 2>/dev/null | cut -f1
}

# ==================== 主逻辑 ====================

# 创建备份基础目录
mkdir -p "$BACKUP_BASE"

log "========================================"
log "  开始 Windows → Linux 同步备份"
log "========================================"

# 检查源
check_source

# 创建今天的快照目录
SNAPSHOT_DIR="$BACKUP_BASE/snapshot_$(date +%Y%m%d_%H%M%S)"
LATEST_LINK="$BACKUP_BASE/latest"

log "源目录:   $WIN_SOURCE"
log "目标目录: $SNAPSHOT_DIR"

# 使用 rsync 进行增量备份
# --archive：保留权限、时间等
# --verbose：显示详情
# --delete：删除目标中源已不存在的文件
# --link-dest：与上一次备份比较，未变化的文件使用硬链接（节省空间）
if [ -L "$LATEST_LINK" ]; then
    # 有上一次备份，使用增量备份
    log "检测到上次备份，执行增量备份..."
    rsync -av --delete \
        --link-dest="$LATEST_LINK" \
        "$WIN_SOURCE/" "$SNAPSHOT_DIR/" \
        >> "$LOG_FILE" 2>&1
else
    # 首次备份，完整复制
    log "首次备份，执行完整复制..."
    rsync -av --delete \
        "$WIN_SOURCE/" "$SNAPSHOT_DIR/" \
        >> "$LOG_FILE" 2>&1
fi

# 检查 rsync 结果
if [ $? -eq 0 ]; then
    log "✅ 同步完成！"
else
    log "⚠️  同步过程中出现警告，请检查日志"
fi

# 更新 latest 符号链接（指向最新的备份快照）
rm -f "$LATEST_LINK"
ln -s "$SNAPSHOT_DIR" "$LATEST_LINK"
log "已更新 latest 链接 → $(basename "$SNAPSHOT_DIR")"

# 计算备份大小
backup_size=$(get_dir_size "$SNAPSHOT_DIR")
log "本次备份大小: $backup_size"

# ==================== 清理旧快照 ====================

# 获取所有快照目录（按时间排序）
snapshot_count=$(find "$BACKUP_BASE" -maxdepth 1 -type d -name "snapshot_*" | wc -l)

if [ "$snapshot_count" -gt "$MAX_SNAPSHOTS" ]; then
    delete_count=$((snapshot_count - MAX_SNAPSHOTS))
    log "当前有 $snapshot_count 个快照，超过上限 $MAX_SNAPSHOTS，清理 $delete_count 个旧快照..."

    find "$BACKUP_BASE" -maxdepth 1 -type d -name "snapshot_*" | \
        sort | \
        head -n "$delete_count" | \
        while IFS= read -r old_snapshot; do
            rm -rf "$old_snapshot"
            log "  已删除: $(basename "$old_snapshot")"
        done
else
    log "当前有 $snapshot_count 个快照（上限 $MAX_SNAPSHOTS），无需清理"
fi

# ==================== 汇总报告 ====================

log ""
log "========== 备份汇总 =========="
log "  源目录大小:   $(get_dir_size "$WIN_SOURCE")"
log "  备份目录大小: $(get_dir_size "$BACKUP_BASE")"
log "  快照数量:     $(find "$BACKUP_BASE" -maxdepth 1 -type d -name "snapshot_*" | wc -l)"
log ""

# 列出所有快照
log "  现有快照:"
find "$BACKUP_BASE" -maxdepth 1 -type d -name "snapshot_*" | sort | while IFS= read -r snap; do
    log "    $(basename "$snap")  ($(get_dir_size "$snap"))"
done

log ""
log "========================================"
log "  备份任务完成 🎉"
log "========================================"
```

### 完整使用流程

```bash
# 1. 创建脚本
nano ~/sync_backup.sh
# （粘贴上面完整的脚本内容）

# 2. 修改脚本中的 WIN_SOURCE 为你自己的 Windows 目录
# 例如：WIN_SOURCE="/mnt/c/Users/datovm/Documents/study"

# 3. 安装 rsync（通常已预装）
sudo apt install rsync -y

# 4. 添加执行权限
chmod +x ~/sync_backup.sh

# 5. 先创建一些 Windows 测试文件（从 WSL 中）
mkdir -p /mnt/c/Users/$(whoami)/Documents/important_files
echo "文件1内容" > /mnt/c/Users/$(whoami)/Documents/important_files/note1.txt
echo "文件2内容" > /mnt/c/Users/$(whoami)/Documents/important_files/note2.txt

# 6. 运行脚本
~/sync_backup.sh

# 7. 查看备份结果
ls -la ~/win_backup/
ls -la ~/win_backup/latest/

# 8. 查看日志
cat ~/win_backup/sync.log
```

### 设置定时自动同步

```bash
# 编辑定时任务
crontab -e

# 添加以下内容：每天中午 12 点和晚上 8 点自动备份
0 12,20 * * * /home/datovm/sync_backup.sh

# 确保 cron 服务运行
sudo service cron start
```

---

## 10.4 综合知识梳理：一个完整的 Linux 运维日常

学完所有章节后，下面是一个**模拟运维场景**，把前面学到的知识融会贯通。

### 场景：你接手了一台新的 WSL 环境，需要从零配置

```bash
# ============================================================
# 阶段 1：了解系统信息（第一章 + 第七章）
# ============================================================

# 查看系统版本
cat /etc/os-release

# 查看内核版本
uname -a

# 查看 CPU 信息
lscpu | head -15

# 查看内存
free -h

# 查看磁盘
df -h

# 查看当前用户
whoami && id

# ============================================================
# 阶段 2：系统更新与基础工具安装（第六章）
# ============================================================

# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装基础工具
sudo apt install -y \
    vim nano git curl wget \
    htop tree zip unzip \
    net-tools dnsutils \
    build-essential \
    rsync jq

# ============================================================
# 阶段 3：用户与安全配置（第四章）
# ============================================================

# 创建一个开发者账户
sudo adduser developer

# 创建开发者组
sudo groupadd devteam

# 把用户加入组
sudo usermod -aG devteam developer
sudo usermod -aG sudo developer     # 赋予 sudo 权限

# 创建共享项目目录
sudo mkdir -p /opt/projects
sudo chown root:devteam /opt/projects
sudo chmod 775 /opt/projects

# 验证
ls -ld /opt/projects
# drwxrwxr-x 2 root devteam 4096 ... /opt/projects

# ============================================================
# 阶段 4：安装和配置 Nginx（第六章 + 第八章）
# ============================================================

# 安装
sudo apt install nginx -y

# 创建网站目录
sudo mkdir -p /var/www/myapp
sudo chown -R www-data:devteam /var/www/myapp
sudo chmod -R 775 /var/www/myapp

# 创建一个简单的页面
echo "<h1>Server is running!</h1>" | sudo tee /var/www/myapp/index.html

# 配置 Nginx
sudo nano /etc/nginx/sites-available/myapp
# （输入 server 块配置，参考 10.1）

# 启用站点
sudo ln -sf /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default

# 测试并重启
sudo nginx -t && sudo service nginx restart

# 验证
curl -s http://localhost | head -5

# ============================================================
# 阶段 5：网络检查（第八章）
# ============================================================

# 查看监听的端口
ss -tlnp

# 检查 Nginx 是否在监听 80 端口
ss -tlnp | grep :80

# 测试外网连通性
ping -c 3 google.com

# ============================================================
# 阶段 6：配置自动备份（第九章）
# ============================================================

# 创建备份脚本（参考 9.6 的脚本）
nano ~/backup.sh
chmod +x ~/backup.sh

# 设置定时任务
crontab -e
# 添加：0 3 * * * /home/datovm/backup.sh /var/www/myapp

# 启动 cron
sudo service cron start

# 验证定时任务
crontab -l

# ============================================================
# 阶段 7：日常监控（第七章）
# ============================================================

# 查看系统负载
uptime

# 查看占用资源最多的进程
ps aux --sort=-%mem | head -10

# 交互式监控
htop

# 查看磁盘使用
df -h
du -h --max-depth=1 /var/log | sort -rh | head -10

# 查看 Nginx 日志
sudo tail -20 /var/log/nginx/access.log
sudo tail -20 /var/log/nginx/error.log

# 实时追踪日志
sudo tail -f /var/log/nginx/access.log
```

---

## 10.5 常见问题与故障排查清单

### 速查表：遇到问题先查这里

| 问题           | 排查命令                                      | 解决方法                              |
| -------------- | --------------------------------------------- | ------------------------------------- |
| 命令找不到     | `which 命令名`                                | `sudo apt install 包名`               |
| 权限不够       | `ls -la 文件名`                               | `sudo` 或 `chmod`                     |
| 磁盘满了       | `df -h` → `du -h --max-depth=1 / \| sort -rh` | 删除大文件/旧日志                     |
| 内存不足       | `free -h` → `ps aux --sort=-%mem \| head`     | 终止占内存的进程，或 `wsl --shutdown` |
| 端口被占用     | `ss -tlnp \| grep :端口号`                    | `kill` 占用端口的进程                 |
| 服务没启动     | `sudo service 服务名 status`                  | `sudo service 服务名 start`           |
| 网络不通       | `ping 8.8.8.8` 和 `ping google.com`           | 区分 DNS 问题和网络问题               |
| WSL 卡死       | （从 Windows PowerShell）`wsl --shutdown`     | 重启 WSL                              |
| apt 更新失败   | `sudo apt update` 报错                        | 检查 `/etc/apt/sources.list`，换源    |
| 定时任务没执行 | `sudo service cron status`                    | `sudo service cron start`             |

### 阅读错误信息的技巧

```bash
# 错误示例 1：
bash: ./script.sh: Permission denied
# 含义：没有执行权限
# 解决：chmod +x script.sh

# 错误示例 2：
E: Could not get lock /var/lib/dpkg/lock-frontend
# 含义：另一个 apt 进程正在运行
# 解决：等它完成，或 sudo kill 那个进程

# 错误示例 3：
No space left on device
# 含义：磁盘满了
# 解决：df -h 查看 → du 找大文件 → 删除不需要的文件

# 错误示例 4：
bind: Address already in use
# 含义：端口被占用
# 解决：ss -tlnp | grep :端口号 → kill 占用的进程
```

---

## 10.6 学习路线建议

恭喜你完成了全部十章的学习！以下是继续深入的建议路线：

### 你已经掌握的技能

| ✅ 已学     | 具体内容                                 |
| ---------- | ---------------------------------------- |
| 基础认知   | Linux 概念、发行版、文件系统结构         |
| 环境搭建   | WSL2 安装、配置、工具链                  |
| 命令行操作 | 文件管理、内容查看、搜索过滤、管道重定向 |
| 用户权限   | root/sudo、chmod/chown、用户管理         |
| 文件编辑   | Nano、Vim 基础、VS Code 联动             |
| 软件管理   | apt 全套操作                             |
| 进程管理   | ps、top、htop、kill                      |
| 网络基础   | ip、ping、curl、ufw、WSL 网络            |
| Shell 脚本 | 变量、条件、循环、函数、定时任务         |
| 实战项目   | Nginx 部署、开发环境、备份脚本           |

### 下一步可以学什么

```
当前位置 ──→ 以下方向任选

├── 🐳 Docker 容器化
│     WSL2 完美支持 Docker，学习容器化部署
│
├── 🔐 SSH 与远程管理
│     SSH 密钥认证、远程服务器管理、SCP/SFTP 文件传输
│
├── 📊 系统运维进阶
│     systemd 服务管理、日志分析（journalctl）、性能调优
│
├── 🌐 网络进阶
│     iptables 深入、反向代理、负载均衡、HTTPS 证书配置
│
├── 📝 Shell 脚本进阶
│     正则表达式深入、awk/sed 文本处理、复杂脚本架构
│
└── ☁️ 云服务器实战
      购买云服务器（阿里云/AWS），把 WSL 学到的全部用上
```

---

## 全教程速查命令索引

最后附上一份按用途分类的**速查命令表**，方便你日后随时翻阅：

### 文件操作
```bash
ls -lah                    # 详细列出文件
cd ~/projects              # 切换目录
mkdir -p dir/sub           # 创建多级目录
touch file.txt             # 创建空文件
cp -r src/ dest/           # 复制目录
mv old new                 # 移动/重命名
rm -rf dir/                # 删除目录（慎用）
find . -name "*.txt"       # 查找文件
grep -rni "keyword" .      # 搜索文件内容
```

### 文件查看
```bash
cat file.txt               # 查看全部内容
less file.txt              # 分页查看
head -20 file.txt          # 查看前 20 行
tail -f log.txt            # 实时追踪文件
wc -l file.txt             # 统计行数
```

### 用户与权限
```bash
sudo command               # 以 root 身份执行
chmod 755 file             # 修改权限
chown user:group file      # 修改所有者
sudo adduser username      # 创建用户
sudo usermod -aG grp user  # 添加到组
```

### 系统监控
```bash
ps aux | grep name         # 查找进程
htop                       # 交互式监控
kill -9 PID                # 强制终止进程
df -h                      # 磁盘使用
free -h                    # 内存使用
uptime                     # 系统负载
```

### 软件管理
```bash
sudo apt update            # 更新源
sudo apt install pkg -y    # 安装软件
sudo apt remove pkg        # 卸载软件
apt search keyword         # 搜索软件
```

### 网络
```bash
ip a                       # 查看 IP
ping -c 4 host             # 测试连通性
curl http://url            # HTTP 请求
ss -tlnp                   # 查看监听端口
sudo ufw allow 80          # 放行端口
```

### WSL 专用（在 PowerShell 中执行）
```powershell
wsl -l -v                  # 查看 WSL 实例
wsl --shutdown             # 关闭所有 WSL
wsl --update               # 更新 WSL
wsl --export Ubuntu file   # 导出/备份
```

---

至此，**Linux 操作入门教程（WSL2 版）** 全十章内容全部完成！🎉

**核心学习建议**：
1. **不要死记硬背**——打开终端，每个命令亲手敲一遍
2. **善用 Tab 补全和 `--help`**——它们是你最好的老师
3. **遇到问题先 Google/搜索错误信息**——Linux 社区的答案非常丰富
4. **多写 Shell 脚本**——自动化是 Linux 的灵魂
5. **考虑买一台便宜的云服务器练手**——真实环境会让你学得更快