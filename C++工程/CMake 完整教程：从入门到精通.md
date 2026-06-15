# CMake 完整教程：从入门到精通

## 第一部分：基础概念

### 1.1 什么是 CMake？

CMake（Cross-platform Make）是一个跨平台的构建系统生成器。它不直接构建你的项目，而是生成特定平台的构建文件（如 Unix 的 Makefile、Windows 的 Visual Studio 项目文件等），然后由这些构建系统来编译你的代码。

**为什么需要 CMake？**

传统的构建方式存在问题：
- **平台依赖性**：在 Linux 上用 Makefile，在 Windows 上用 Visual Studio，代码要维护多套构建脚本
- **复杂性**：手写 Makefile 容易出错，难以维护
- **可移植性差**：换个平台就要重写构建脚本

CMake 解决了这些问题：
- **跨平台**：一次编写，到处构建
- **抽象层次高**：描述"要做什么"而非"怎么做"
- **易于维护**：语法简洁，逻辑清晰

### 1.2 CMake 工作流程

```
源代码 + CMakeLists.txt 
    ↓
cmake 命令（配置阶段）
    ↓
生成构建文件（Makefile 或项目文件）
    ↓
make/ninja/msbuild（构建阶段）
    ↓
可执行文件/库文件
```

**详细说明：**

1. **配置阶段**（cmake 命令）
   - 读取 CMakeLists.txt 文件
   - 检测编译器、系统特性
   - 生成构建系统文件
   - 创建 CMakeCache.txt 缓存配置

2. **构建阶段**（make/ninja 等）
   - 实际编译源代码
   - 链接生成最终产物

### 1.3 安装 CMake

**Linux：**
```bash
# Ubuntu/Debian
sudo apt-get install cmake

# Fedora/RHEL
sudo dnf install cmake

# 查看版本
cmake --version
```

**macOS：**
```bash
brew install cmake
```

**Windows：**
- 从官网下载安装包：https://cmake.org/download/
- 安装时选择"添加到系统 PATH"

### 1.4 基本术语

- **源码目录**（Source Directory）：包含 CMakeLists.txt 和源代码的目录
- **构建目录**（Build Directory）：存放构建产物的目录，通常命名为 `build`
- **内源构建**（In-Source Build）：在源码目录直接构建（不推荐）
- **外源构建**（Out-of-Source Build）：在独立的 build 目录构建（推荐）
- **目标**（Target）：可执行文件、库文件等构建产物
- **生成器**（Generator）：决定生成什么类型的构建文件（Makefile、Ninja、Visual Studio 等）

---

## 第二部分：CMakeLists.txt 编写

### 2.1 最简单的 CMakeLists.txt

创建项目结构：
```
my_project/
├── CMakeLists.txt
└── main.cpp
```

**main.cpp：**
```cpp
#include <iostream>

int main() {
    std::cout << "Hello, CMake!" << std::endl;
    return 0;
}
```

**CMakeLists.txt：**
```cmake
# 指定 CMake 最低版本要求
cmake_minimum_required(VERSION 3.10)

# 定义项目名称
project(MyFirstProject)

# 添加可执行文件目标
add_executable(hello main.cpp)
```

**逐行解释：**

1. `cmake_minimum_required(VERSION 3.10)`
   - 指定需要的 CMake 最低版本
   - 如果用户的 CMake 版本太低，会报错
   - 版本号影响可用的功能和命令

2. `project(MyFirstProject)`
   - 设置项目名称为 MyFirstProject
   - 自动设置变量：`PROJECT_NAME`、`PROJECT_SOURCE_DIR` 等
   - 可以指定版本号和语言：`project(MyFirstProject VERSION 1.0 LANGUAGES CXX)`

3. `add_executable(hello main.cpp)`
   - 定义一个可执行文件目标，名为 `hello`
   - 由 `main.cpp` 编译而来
   - 可以添加多个源文件：`add_executable(hello main.cpp utils.cpp)`

### 2.2 构建这个项目

```bash
# 创建构建目录
mkdir build
cd build

# 配置项目（生成构建文件）
cmake ..

# 构建项目
cmake --build .

# 运行程序
./hello
```

**详细说明：**

- `cmake ..`：`..` 表示源码目录在上一级，CMake 会在当前目录生成构建文件
- `cmake --build .`：跨平台的构建命令，等价于：
  - Linux/macOS: `make`
  - Windows (Visual Studio): `msbuild xxx.sln`

### 2.3 CMake 变量

CMake 使用变量来存储和传递信息。

**定义和使用变量：**
```cmake
# 设置变量
set(MY_VARIABLE "hello")
set(MY_NUMBER 42)
set(MY_LIST a b c)

# 使用变量（通过 ${} 语法）
message("变量值: ${MY_VARIABLE}")

# 列表操作
list(APPEND MY_LIST d e)
```

**常用预定义变量：**

```cmake
# 项目信息
PROJECT_NAME                    # 项目名称
PROJECT_SOURCE_DIR             # 项目源码根目录
PROJECT_BINARY_DIR             # 项目构建根目录

# 目录信息
CMAKE_SOURCE_DIR               # 顶层 CMakeLists.txt 所在目录
CMAKE_BINARY_DIR               # 顶层构建目录
CMAKE_CURRENT_SOURCE_DIR       # 当前处理的 CMakeLists.txt 所在目录
CMAKE_CURRENT_BINARY_DIR       # 当前 CMakeLists.txt 的构建目录

# 编译器信息
CMAKE_CXX_COMPILER             # C++ 编译器路径
CMAKE_C_COMPILER               # C 编译器路径
CMAKE_CXX_COMPILER_ID          # 编译器 ID（GNU, Clang, MSVC 等）

# 系统信息
CMAKE_SYSTEM_NAME              # 操作系统名称（Linux, Windows, Darwin）
CMAKE_SYSTEM_PROCESSOR         # 处理器架构

# 构建类型
CMAKE_BUILD_TYPE               # Debug, Release, RelWithDebInfo, MinSizeRel
```

**示例：使用变量**
```cmake
cmake_minimum_required(VERSION 3.10)
project(VariableDemo)

# 定义源文件列表
set(SOURCES 
    main.cpp
    utils.cpp
    helper.cpp
)

# 定义头文件目录
set(INCLUDE_DIRS 
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/third_party
)

add_executable(myapp ${SOURCES})

# 添加头文件搜索路径
target_include_directories(myapp PRIVATE ${INCLUDE_DIRS})

# 打印信息
message(STATUS "项目名称: ${PROJECT_NAME}")
message(STATUS "源码目录: ${PROJECT_SOURCE_DIR}")
message(STATUS "构建目录: ${PROJECT_BINARY_DIR}")
```

### 2.4 设置 C++ 标准

```cmake
cmake_minimum_required(VERSION 3.10)
project(CppStandardDemo)

# 方法 1: 全局设置 C++ 标准
set(CMAKE_CXX_STANDARD 17)           # 使用 C++17
set(CMAKE_CXX_STANDARD_REQUIRED ON)   # 必须支持该标准，否则报错
set(CMAKE_CXX_EXTENSIONS OFF)         # 禁用编译器扩展（如 GNU 扩展）

add_executable(myapp main.cpp)

# 方法 2: 针对特定目标设置
add_executable(another_app other.cpp)
set_property(TARGET another_app PROPERTY CXX_STANDARD 20)
```

**C++ 标准版本：**
- 98, 03, 11, 14, 17, 20, 23

### 2.5 编译选项

```cmake
# 全局编译选项
add_compile_options(-Wall -Wextra)

# 针对特定目标的编译选项
target_compile_options(myapp PRIVATE 
    -Wall          # 开启所有警告
    -Wextra        # 额外警告
    -Werror        # 警告视为错误
    -O2            # 优化级别
)

# 根据编译器设置不同选项
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(myapp PRIVATE -fno-rtti)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(myapp PRIVATE /W4)
endif()
```

**PRIVATE、PUBLIC、INTERFACE 关键字：**

- **PRIVATE**: 仅影响当前目标，不传播
- **PUBLIC**: 影响当前目标，且传播给依赖此目标的其他目标
- **INTERFACE**: 不影响当前目标，仅传播给依赖者

```cmake
# 示例
add_library(mylib lib.cpp)
target_compile_options(mylib PRIVATE -fPIC)      # 仅 mylib 使用
target_compile_options(mylib PUBLIC -DDEBUG)     # mylib 和依赖它的目标都使用
target_compile_options(mylib INTERFACE -DAPI)    # 仅依赖 mylib 的目标使用
```

### 2.6 条件语句

```cmake
# if-else 语句
if(WIN32)
    message("这是 Windows 系统")
elseif(APPLE)
    message("这是 macOS 系统")
elseif(UNIX)
    message("这是 Unix/Linux 系统")
else()
    message("未知系统")
endif()

# 变量存在性检查
if(DEFINED MY_VARIABLE)
    message("MY_VARIABLE 已定义")
endif()

# 字符串比较
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    message("调试模式")
endif()

# 数值比较
if(CMAKE_CXX_STANDARD GREATER_EQUAL 17)
    message("使用 C++17 或更高版本")
endif()

# 逻辑运算
if(CMAKE_SYSTEM_NAME STREQUAL "Linux" AND CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    message("Linux + GCC")
endif()

# 文件存在性
if(EXISTS "${PROJECT_SOURCE_DIR}/config.h")
    message("找到配置文件")
endif()
```

### 2.7 循环

```cmake
# foreach 循环
set(SOURCES main.cpp utils.cpp helper.cpp)

foreach(source ${SOURCES})
    message("处理源文件: ${source}")
endforeach()

# 范围循环
foreach(i RANGE 1 5)
    message("数字: ${i}")
endforeach()

# 列表迭代
set(LIBS math pthread dl)
foreach(lib ${LIBS})
    target_link_libraries(myapp ${lib})
endforeach()
```

### 2.8 函数和宏

**函数（推荐）：**
```cmake
# 定义函数
function(print_info target_name)
    message("目标名称: ${target_name}")
    message("源码目录: ${CMAKE_CURRENT_SOURCE_DIR}")
endfunction()

# 调用函数
print_info(myapp)

# 带返回值的函数
function(get_source_files result_var)
    file(GLOB sources "src/*.cpp")
    set(${result_var} ${sources} PARENT_SCOPE)  # PARENT_SCOPE 将变量传递到外层作用域
endfunction()

get_source_files(MY_SOURCES)
add_executable(myapp ${MY_SOURCES})
```

**宏：**
```cmake
# 定义宏
macro(add_test_executable test_name)
    add_executable(${test_name} ${test_name}.cpp)
    target_link_libraries(${test_name} gtest)
endmacro()

# 调用宏
add_test_executable(test_math)
add_test_executable(test_string)
```

**函数 vs 宏的区别：**
- 函数有独立作用域，变量不会泄露
- 宏是简单的文本替换，共享调用者的作用域
- 推荐使用函数，除非需要修改调用者的变量

---

## 第三部分：项目构建

### 3.1 构建类型

CMake 支持多种构建类型：

```cmake
# 设置默认构建类型（仅适用于 Makefile、Ninja 等单配置生成器）
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release)
endif()

message(STATUS "构建类型: ${CMAKE_BUILD_TYPE}")
```

**四种标准构建类型：**

1. **Debug**：调试版本
   - 包含调试符号
   - 不优化代码
   - 常见编译选项：`-g`

2. **Release**：发布版本
   - 开启优化
   - 不包含调试符号
   - 常见编译选项：`-O3 -DNDEBUG`

3. **RelWithDebInfo**：带调试信息的发布版本
   - 开启优化
   - 包含调试符号
   - 常见编译选项：`-O2 -g -DNDEBUG`

4. **MinSizeRel**：最小尺寸发布版本
   - 优化代码大小
   - 常见编译选项：`-Os -DNDEBUG`

实际选项由编译器、工具链和 `CMAKE_<LANG>_FLAGS_<CONFIG>` 决定。

**指定构建类型：**
```bash
# 方法 1: 配置时指定
cmake -DCMAKE_BUILD_TYPE=Debug ..

# 方法 2: 在 CMakeLists.txt 中根据类型设置选项
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(myapp PRIVATE DEBUG_MODE)
endif()
```

### 3.2 生成器

生成器决定生成什么类型的构建文件。

**查看可用生成器：**
```bash
cmake --help
```

**常用生成器：**

- **Unix Makefiles**（默认，Linux/macOS）
- **Ninja**（速度快，推荐）
- **Visual Studio 16 2019**（Windows）
- **Xcode**（macOS）

**指定生成器：**
```bash
# 使用 Ninja
cmake -G Ninja ..

# 使用 Visual Studio
cmake -G "Visual Studio 16 2019" ..
```

### 3.3 多配置生成器

某些生成器（如 Visual Studio、Xcode）支持在单次配置中包含多个构建类型。

```bash
# Visual Studio: 配置时不指定类型
cmake -G "Visual Studio 16 2019" ..

# 构建时指定类型
cmake --build . --config Release
cmake --build . --config Debug
```

### 3.4 并行构建

```bash
# 使用所有 CPU 核心
cmake --build . --parallel

# 指定核心数
cmake --build . -j 4

# Makefile 方式
make -j 4
```

### 3.5 详细输出

```bash
# 查看完整编译命令
cmake --build . --verbose

# 或使用 make
make VERBOSE=1
```

### 3.6 清理构建

```bash
# 清理构建产物
cmake --build . --target clean

# 或
make clean

# 完全重新构建（删除 build 目录）
rm -rf build
mkdir build
cd build
cmake ..
cmake --build .
```

### 3.7 指定编译器

```bash
# 方法 1: 环境变量
export CC=gcc
export CXX=g++
cmake ..

# 方法 2: CMake 参数
cmake -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ ..
```

```cmake
# 方法 3: 工具链文件（适合交叉编译或固定工具链）
set(CMAKE_C_COMPILER gcc)
set(CMAKE_CXX_COMPILER g++)
```

```bash
cmake -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake ..
```

### 3.8 查找工具

```cmake
# 查找 Make 程序
find_program(MAKE_EXECUTABLE NAMES make gmake)
if(MAKE_EXECUTABLE)
    message("找到 Make: ${MAKE_EXECUTABLE}")
endif()

# 查找 Git
find_package(Git)
if(Git_FOUND)
    message("Git 版本: ${GIT_VERSION_STRING}")
endif()
```

---

## 第四部分：工程组织

### 4.1 单目录结构

最简单的项目：所有文件在同一目录。

```
simple_project/
├── CMakeLists.txt
├── main.cpp
├── utils.cpp
└── utils.h
```

**CMakeLists.txt：**
```cmake
cmake_minimum_required(VERSION 3.10)
project(SimpleProject)

set(CMAKE_CXX_STANDARD 17)

add_executable(app main.cpp utils.cpp)
```

### 4.2 标准目录结构

推荐的项目组织方式：

```
my_project/
├── CMakeLists.txt          # 顶层 CMake 文件
├── include/                # 公共头文件
│   └── mylib/
│       ├── math.h
│       └── utils.h
├── src/                    # 源文件
│   ├── CMakeLists.txt      # 子目录 CMake 文件
│   ├── main.cpp
│   ├── math.cpp
│   └── utils.cpp
├── tests/                  # 测试代码
│   ├── CMakeLists.txt
│   └── test_math.cpp
├── docs/                   # 文档
├── external/               # 第三方依赖
└── build/                  # 构建目录（不纳入版本控制）
```

**顶层 CMakeLists.txt：**
```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 17)

# 添加子目录
add_subdirectory(src)
add_subdirectory(tests)
```

**src/CMakeLists.txt：**
```cmake
# 设置头文件搜索路径
include_directories(${PROJECT_SOURCE_DIR}/include)

# 收集源文件
set(SOURCES
    main.cpp
    math.cpp
    utils.cpp
)

# 创建可执行文件
add_executable(myapp ${SOURCES})
```

### 4.3 使用 add_subdirectory

`add_subdirectory` 允许组织多层次的 CMake 文件。

**示例：**

```
project/
├── CMakeLists.txt
├── app/
│   ├── CMakeLists.txt
│   └── main.cpp
└── lib/
    ├── CMakeLists.txt
    ├── mylib.cpp
    └── mylib.h
```

**顶层 CMakeLists.txt：**
```cmake
cmake_minimum_required(VERSION 3.10)
project(MultiDirProject)

add_subdirectory(lib)
add_subdirectory(app)
```

**lib/CMakeLists.txt：**
```cmake
add_library(mylib mylib.cpp)

# 指定头文件位置（PUBLIC 使依赖此库的目标也能找到头文件）
target_include_directories(mylib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

**app/CMakeLists.txt：**
```cmake
add_executable(myapp main.cpp)

# 链接库
target_link_libraries(myapp mylib)
```

### 4.4 创建库

**静态库：**
```cmake
add_library(mylib STATIC 
    lib.cpp
    helper.cpp
)
```

**动态库（共享库）：**
```cmake
add_library(mylib SHARED
    lib.cpp
    helper.cpp
)
```

**接口库（仅头文件库）：**
```cmake
add_library(mylib INTERFACE)
target_include_directories(mylib INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

**对象库（中间对象）：**
```cmake
add_library(mylib_obj OBJECT lib.cpp)

# 使用对象库
add_executable(app1 main1.cpp $<TARGET_OBJECTS:mylib_obj>)
add_executable(app2 main2.cpp $<TARGET_OBJECTS:mylib_obj>)
```

### 4.5 链接库

```cmake
add_executable(myapp main.cpp)

# 链接自定义库
target_link_libraries(myapp mylib)

# 链接系统库
target_link_libraries(myapp pthread m dl)

# 链接第三方库
find_package(Boost REQUIRED)
target_link_libraries(myapp Boost::boost)

# PRIVATE/PUBLIC/INTERFACE
add_library(mylib lib.cpp)
target_link_libraries(mylib 
    PRIVATE internal_lib      # 仅 mylib 内部使用
    PUBLIC common_lib         # mylib 和链接 mylib 的目标都使用
    INTERFACE header_only_lib # 仅链接 mylib 的目标使用
)
```

### 4.6 头文件管理

```cmake
# 方法 1: 全局包含目录（不推荐，污染全局命名空间）
include_directories(${PROJECT_SOURCE_DIR}/include)

# 方法 2: 目标特定包含目录（推荐）
target_include_directories(myapp 
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/private_include
    PUBLIC ${PROJECT_SOURCE_DIR}/include
)

# 方法 3: 使用变量组织
set(PUBLIC_HEADERS
    ${PROJECT_SOURCE_DIR}/include/mylib/math.h
    ${PROJECT_SOURCE_DIR}/include/mylib/utils.h
)

set(PRIVATE_HEADERS
    ${CMAKE_CURRENT_SOURCE_DIR}/internal.h
)

add_library(mylib mylib.cpp)
target_include_directories(mylib
    PUBLIC ${PROJECT_SOURCE_DIR}/include
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}
)

# 安装头文件
install(FILES ${PUBLIC_HEADERS} DESTINATION include/mylib)
```

### 4.7 文件搜索

**手动列出文件（推荐）：**
```cmake
set(SOURCES
    main.cpp
    utils.cpp
    helper.cpp
)

add_executable(myapp ${SOURCES})
```

**使用 GLOB（不推荐用于源文件）：**
```cmake
# 通配符匹配
file(GLOB SOURCES "src/*.cpp")
file(GLOB_RECURSE HEADERS "include/*.h")  # 递归搜索

add_executable(myapp ${SOURCES})

# 问题: 普通 GLOB 不会自动触发重新配置
# 可用 CONFIGURE_DEPENDS，但核心源文件仍建议显式列出
```

**为什么不推荐 GLOB：**
- CMake 在配置阶段运行，之后添加文件不会触发重新配置
- 导致增量构建可能遗漏新文件
- 手动列出文件更明确、可控

**GLOB 适用场景：**
- 收集资源文件（图片、配置文件）
- 安装目录中的文件

```cmake
file(GLOB CONFIG_FILES "config/*.json")
install(FILES ${CONFIG_FILES} DESTINATION etc)
```

### 4.8 生成器表达式

生成器表达式会在生成构建系统或相关导出/安装上下文中延迟求值，用于条件编译。

**语法：** `$<条件:真值>`

```cmake
# 根据构建类型添加编译选项
target_compile_options(myapp PRIVATE
    $<$<CONFIG:Debug>:-g -O0>
    $<$<CONFIG:Release>:-O3>
)

# 根据编译器添加选项
target_compile_options(myapp PRIVATE
    $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra>
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
)

# 根据平台添加链接库
target_link_libraries(myapp
    $<$<PLATFORM_ID:Linux>:pthread>
    $<$<PLATFORM_ID:Windows>:ws2_32>
)

# 组合条件
target_compile_definitions(myapp PRIVATE
    $<$<AND:$<CONFIG:Debug>,$<CXX_COMPILER_ID:GNU>>:DEBUG_WITH_GCC>
)

# 目标属性
target_include_directories(myapp PRIVATE
    $<TARGET_PROPERTY:mylib,INTERFACE_INCLUDE_DIRECTORIES>
)
```

**常用生成器表达式：**

- `$<CONFIG:cfg>`：匹配构建类型
- `$<PLATFORM_ID:platform>`：匹配平台
- `$<CXX_COMPILER_ID:compiler>`：匹配编译器
- `$<TARGET_EXISTS:target>`：目标是否存在
- `$<TARGET_PROPERTY:target,prop>`：获取目标属性
- `$<BOOL:value>`：布尔转换
- `$<AND:cond1,cond2>`、`$<OR:cond1,cond2>`：逻辑运算

---

## 第五部分：安装部署

### 5.1 安装基础

`install` 命令指定哪些文件在 `make install` 时安装到哪里。

**基本语法：**
```cmake
install(TARGETS myapp
    RUNTIME DESTINATION bin        # 可执行文件
    LIBRARY DESTINATION lib        # 动态库
    ARCHIVE DESTINATION lib        # 静态库
)
```

**完整示例：**
```cmake
cmake_minimum_required(VERSION 3.10)
project(InstallDemo VERSION 1.0)

# 创建库
add_library(mylib SHARED mylib.cpp)
target_include_directories(mylib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# 创建可执行文件
add_executable(myapp main.cpp)
target_link_libraries(myapp mylib)

# 安装目标
install(TARGETS myapp mylib
    EXPORT MyAppTargets
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)

# 安装头文件
install(DIRECTORY include/ DESTINATION include)

# 安装配置文件
install(FILES config.json DESTINATION etc)
```

**执行安装：**
```bash
cmake --build . --target install

# 或
make install

# 指定安装前缀
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..
make install
```

### 5.2 安装组件

将安装内容分组，允许选择性安装。

```cmake
# 定义组件
install(TARGETS myapp
    RUNTIME DESTINATION bin
    COMPONENT applications
)

install(TARGETS mylib
    LIBRARY DESTINATION lib
    COMPONENT libraries
)

install(DIRECTORY include/
    DESTINATION include
    COMPONENT development
)

install(FILES README.md LICENSE
    DESTINATION share/doc/myproject
    COMPONENT documentation
)
```

**选择性安装：**
```bash
# 仅安装应用程序
cmake --install . --component applications

# 安装多个组件
cmake --install . --component libraries
cmake --install . --component development
```

### 5.3 导出目标

允许其他项目通过 `find_package` 找到你的项目。

**完整流程：**

```cmake
# 1. 安装目标时导出
install(TARGETS mylib myapp
    EXPORT MyProjectTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

# 2. 安装导出文件
install(EXPORT MyProjectTargets
    FILE MyProjectTargets.cmake
    NAMESPACE MyProject::
    DESTINATION lib/cmake/MyProject
)

# 3. 生成配置文件
include(CMakePackageConfigHelpers)

# 创建版本文件
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

# 配置配置文件
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake"
    INSTALL_DESTINATION lib/cmake/MyProject
)

# 安装配置文件
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake"
    DESTINATION lib/cmake/MyProject
)
```

**Config.cmake.in 文件：**
```cmake
@PACKAGE_INIT@

include("${CMAKE_CURRENT_LIST_DIR}/MyProjectTargets.cmake")

check_required_components(MyProject)
```

**使用导出的项目：**

```cmake
# 在其他项目中使用
find_package(MyProject REQUIRED)

add_executable(consumer main.cpp)
target_link_libraries(consumer MyProject::mylib)
```

### 5.4 CPack 打包

CPack 是 CMake 的打包工具，可以生成各种格式的安装包。

**基本配置：**

```cmake
# 设置包信息
set(CPACK_PACKAGE_NAME "MyProject")
set(CPACK_PACKAGE_VENDOR "Your Company")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "My awesome project")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})

# 设置包文件名
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${PROJECT_VERSION}-${CMAKE_SYSTEM_NAME}")

# 包含的文件
set(CPACK_PACKAGE_INSTALL_DIRECTORY "MyProject")

include(CPack)
```

**生成不同格式的包：**

```cmake
# 以下是不同平台/格式的互斥示例；多个生成器可写成 "DEB;RPM;TGZ"

# Linux: DEB 包
set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name <your.email@example.com>")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6 (>= 2.27)")

# Linux: RPM 包
set(CPACK_GENERATOR "RPM")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "Development/Tools")

# Windows: NSIS 安装器
set(CPACK_GENERATOR "NSIS")
set(CPACK_NSIS_DISPLAY_NAME "MyProject")
set(CPACK_NSIS_CONTACT "your.email@example.com")
set(CPACK_NSIS_HELP_LINK "https://myproject.com")

# macOS: DMG
set(CPACK_GENERATOR "DragNDrop")

# 通用: TGZ 压缩包
set(CPACK_GENERATOR "TGZ")
```

**生成包：**

```bash
# 配置并构建
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .

# 生成包
cpack

# 生成特定类型的包
cpack -G DEB
cpack -G RPM
cpack -G TGZ
```

**完整的打包示例：**

```cmake
cmake_minimum_required(VERSION 3.10)
project(PackageDemo VERSION 1.2.3)

add_executable(myapp main.cpp)

install(TARGETS myapp RUNTIME DESTINATION bin)
install(FILES README.md LICENSE DESTINATION share/doc/myapp)

# CPack 配置
set(CPACK_PACKAGE_NAME "myapp")
set(CPACK_PACKAGE_VENDOR "My Company")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "A great application")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")

# 根据平台选择生成器
if(WIN32)
    set(CPACK_GENERATOR "NSIS;ZIP")
elseif(APPLE)
    set(CPACK_GENERATOR "DragNDrop;TGZ")
else()
    set(CPACK_GENERATOR "DEB;RPM;TGZ")
endif()

include(CPack)
```

### 5.5 安装后脚本

在安装前后执行自定义操作。

```cmake
# 安装后执行脚本
install(CODE "execute_process(COMMAND ${CMAKE_COMMAND} -E echo '安装完成！')")

# 安装前执行
install(CODE "
    message(STATUS \"准备安装到 \${CMAKE_INSTALL_PREFIX}\")
" COMPONENT Runtime)

# 使用脚本文件
install(SCRIPT "${CMAKE_CURRENT_SOURCE_DIR}/post_install.cmake")
```

**post_install.cmake 示例：**

```cmake
message("执行安装后配置...")

# 创建符号链接
execute_process(
    COMMAND ${CMAKE_COMMAND} -E create_symlink
        ${CMAKE_INSTALL_PREFIX}/bin/myapp
        ${CMAKE_INSTALL_PREFIX}/bin/app
)

# 设置权限
file(CHMOD ${CMAKE_INSTALL_PREFIX}/bin/myapp PERMISSIONS
    OWNER_READ OWNER_WRITE OWNER_EXECUTE
    GROUP_READ GROUP_EXECUTE
    WORLD_READ WORLD_EXECUTE
)

message("配置完成")
```

---

## 第六部分：库管理

### 6.1 查找系统库

使用 `find_library` 查找系统中已安装的库。

```cmake
# 查找单个库
find_library(MATH_LIB m)  # 查找 libm (数学库)

if(MATH_LIB)
    message("找到数学库: ${MATH_LIB}")
    target_link_libraries(myapp ${MATH_LIB})
else()
    message(FATAL_ERROR "找不到数学库")
endif()

# 查找多个库
find_package(Threads REQUIRED)
find_library(DL_LIB dl)

target_link_libraries(myapp Threads::Threads ${DL_LIB})

# 指定搜索路径
find_library(CUSTOM_LIB
    NAMES customlib
    PATHS /usr/local/lib /opt/lib
    NO_DEFAULT_PATH  # 不搜索默认路径
)
```

### 6.2 find_package 详解

`find_package` 是查找第三方库的标准方式。

**两种模式：**

1. **Module 模式**：查找 `FindXXX.cmake` 文件
2. **Config 模式**：查找 `XXXConfig.cmake` 或 `xxx-config.cmake` 文件

**基本用法：**

```cmake
# 查找包（可选）
find_package(ZLIB)

if(ZLIB_FOUND)
    target_include_directories(myapp PRIVATE ${ZLIB_INCLUDE_DIRS})
    target_link_libraries(myapp ${ZLIB_LIBRARIES})
endif()

# 查找包（必需）
find_package(Threads REQUIRED)
target_link_libraries(myapp Threads::Threads)

# 指定版本
find_package(Boost 1.70 REQUIRED)

# 查找特定组件
find_package(Boost REQUIRED COMPONENTS system filesystem)
target_link_libraries(myapp Boost::system Boost::filesystem)
```

**查找路径：**

常用影响因素包括 `<Package>_DIR` 缓存变量、`CMAKE_PREFIX_PATH`、`<Package>_ROOT`、系统标准路径（`/usr`、`/usr/local` 等）。完整搜索顺序以 `find_package` 官方文档为准。

```cmake
# 添加自定义搜索路径
list(APPEND CMAKE_PREFIX_PATH "/opt/mylib")

find_package(MyLib REQUIRED)
```

**命令行指定路径：**

```bash
cmake -DCMAKE_PREFIX_PATH=/opt/mylib ..

# 或直接指定包的配置文件位置
cmake -DMyLib_DIR=/opt/mylib/lib/cmake/MyLib ..
```

### 6.3 常用库的 find_package

**Threads（线程库）：**

```cmake
find_package(Threads REQUIRED)
target_link_libraries(myapp Threads::Threads)
```

**OpenMP（并行计算）：**

```cmake
find_package(OpenMP REQUIRED)
target_link_libraries(myapp OpenMP::OpenMP_CXX)
```

**Boost（C++ 库集合）：**

```cmake
find_package(Boost 1.70 REQUIRED COMPONENTS 
    system 
    filesystem 
    thread
    program_options
)

target_link_libraries(myapp 
    Boost::system
    Boost::filesystem
    Boost::thread
    Boost::program_options
)
```

**OpenCV（计算机视觉）：**

```cmake
find_package(OpenCV REQUIRED)

message(STATUS "OpenCV 版本: ${OpenCV_VERSION}")
message(STATUS "OpenCV 包含目录: ${OpenCV_INCLUDE_DIRS}")

target_link_libraries(myapp ${OpenCV_LIBS})
```

**Qt（GUI 框架）：**

```cmake
find_package(Qt5 REQUIRED COMPONENTS Core Widgets Gui)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

add_executable(myapp main.cpp)

target_link_libraries(myapp 
    Qt5::Core
    Qt5::Widgets
    Qt5::Gui
)
```

**Protobuf（序列化库）：**

```cmake
find_package(Protobuf REQUIRED)

# 生成 protobuf 代码
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS message.proto)

add_executable(myapp main.cpp ${PROTO_SRCS})
target_link_libraries(myapp protobuf::libprotobuf)
```

**gRPC（RPC 框架）：**

```cmake
find_package(Protobuf REQUIRED)
find_package(gRPC REQUIRED)

add_executable(myapp main.cpp)

# 生成 gRPC 代码
protobuf_generate(TARGET myapp LANGUAGE cpp)
protobuf_generate(TARGET myapp LANGUAGE grpc GENERATE_EXTENSIONS .grpc.pb.h .grpc.pb.cc PLUGIN "protoc-gen-grpc=\$<TARGET_FILE:gRPC::grpc_cpp_plugin>")

target_link_libraries(myapp 
    gRPC::grpc++
    gRPC::grpc++_reflection
)
```

### 6.4 编写自定义 FindXXX.cmake

当第三方库没有提供 CMake 支持时，需要自己编写 Find 模块。

**目录结构：**

```
project/
├── CMakeLists.txt
├── cmake/
│   └── FindMyLib.cmake
└── src/
```

**CMakeLists.txt：**

```cmake
# 添加模块搜索路径
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

find_package(MyLib REQUIRED)
```

**cmake/FindMyLib.cmake：**

```cmake
# FindMyLib.cmake

# 查找头文件
find_path(MyLib_INCLUDE_DIR
    NAMES mylib.h
    PATHS
        /usr/include
        /usr/local/include
        /opt/mylib/include
        ${MyLib_ROOT}/include
    PATH_SUFFIXES mylib
)

# 查找库文件
find_library(MyLib_LIBRARY
    NAMES mylib libmylib
    PATHS
        /usr/lib
        /usr/local/lib
        /opt/mylib/lib
        ${MyLib_ROOT}/lib
)

# 处理版本信息（如果有版本头文件）
if(EXISTS "${MyLib_INCLUDE_DIR}/mylib_version.h")
    file(READ "${MyLib_INCLUDE_DIR}/mylib_version.h" VERSION_CONTENT)
    string(REGEX MATCH "#define MYLIB_VERSION \"([0-9.]+)\"" _ ${VERSION_CONTENT})
    set(MyLib_VERSION ${CMAKE_MATCH_1})
endif()

# 标准的 find_package 结果处理
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(MyLib
    REQUIRED_VARS MyLib_LIBRARY MyLib_INCLUDE_DIR
    VERSION_VAR MyLib_VERSION
)

# 创建导入目标（推荐方式）
if(MyLib_FOUND AND NOT TARGET MyLib::MyLib)
    add_library(MyLib::MyLib UNKNOWN IMPORTED)
    set_target_properties(MyLib::MyLib PROPERTIES
        IMPORTED_LOCATION "${MyLib_LIBRARY}"
        INTERFACE_INCLUDE_DIRECTORIES "${MyLib_INCLUDE_DIR}"
    )
endif()

# 设置输出变量（兼容旧方式）
if(MyLib_FOUND)
    set(MyLib_LIBRARIES ${MyLib_LIBRARY})
    set(MyLib_INCLUDE_DIRS ${MyLib_INCLUDE_DIR})
endif()

# 标记为高级选项（在 cmake-gui 中隐藏）
mark_as_advanced(MyLib_INCLUDE_DIR MyLib_LIBRARY)
```

**使用：**

```cmake
find_package(MyLib REQUIRED)

add_executable(myapp main.cpp)
target_link_libraries(myapp MyLib::MyLib)
```

### 6.5 pkg-config 集成

许多 Unix 库使用 pkg-config 来提供编译和链接信息。

```cmake
# 查找 pkg-config
find_package(PkgConfig REQUIRED)

# 使用 pkg-config 查找库
pkg_check_modules(GTK3 REQUIRED gtk+-3.0)

# 方式 1: 使用变量
target_include_directories(myapp PRIVATE ${GTK3_INCLUDE_DIRS})
target_link_libraries(myapp ${GTK3_LIBRARIES})
target_link_directories(myapp PRIVATE ${GTK3_LIBRARY_DIRS})
target_compile_options(myapp PRIVATE ${GTK3_CFLAGS_OTHER})

# 方式 2: 使用导入目标（推荐）
pkg_check_modules(SDL2 REQUIRED IMPORTED_TARGET sdl2)
target_link_libraries(myapp PkgConfig::SDL2)
```

**示例：集成 libcurl**

```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(CURL REQUIRED IMPORTED_TARGET libcurl)

add_executable(downloader main.cpp)
target_link_libraries(downloader PkgConfig::CURL)
```

---

## 第七部分：第三方库集成

### 7.1 使用系统安装的库

最简单的方式：库已经安装在系统中。

```cmake
find_package(ZLIB REQUIRED)

add_executable(myapp main.cpp)
target_link_libraries(myapp ZLIB::ZLIB)
```

### 7.2 使用子模块（Git Submodule）

将第三方库作为 Git 子模块包含。

**添加子模块：**

```bash
git submodule add https://github.com/fmtlib/fmt.git external/fmt
git submodule update --init --recursive
```

**项目结构：**

```
project/
├── CMakeLists.txt
├── external/
│   └── fmt/           # Git 子模块
└── src/
```

**CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# 添加子模块
add_subdirectory(external/fmt)

add_executable(myapp src/main.cpp)
target_link_libraries(myapp fmt::fmt)
```

**优点：**
- 版本明确，随项目源码一起管理
- 离线构建

**缺点：**
- 需要将依赖库源码纳入版本控制
- 构建时间增加

### 7.3 FetchContent（推荐）

CMake 3.11+ 引入的现代依赖管理方式。

**基本用法：**

```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProject)

include(FetchContent)

# 声明依赖
FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 9.1.0  # 指定版本或分支
)

# 使依赖可用
FetchContent_MakeAvailable(fmt)

add_executable(myapp main.cpp)
target_link_libraries(myapp fmt::fmt)
```

**多个依赖：**

```cmake
include(FetchContent)

# 声明多个依赖
FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG v1.14.0
)

FetchContent_Declare(
    json
    GIT_REPOSITORY https://github.com/nlohmann/json.git
    GIT_TAG v3.11.2
)

FetchContent_Declare(
    spdlog
    GIT_REPOSITORY https://github.com/gabime/spdlog.git
    GIT_TAG v1.12.0
)

# 一次性加载所有依赖
FetchContent_MakeAvailable(googletest json spdlog)

add_executable(myapp main.cpp)
target_link_libraries(myapp 
    gtest_main
    nlohmann_json::nlohmann_json
    spdlog::spdlog
)
```

**从本地路径或 URL 加载：**

```cmake
# 从本地路径
FetchContent_Declare(
    mylib
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/external/mylib
)

# 从压缩包
FetchContent_Declare(
    archive_lib
    URL https://example.com/lib-1.0.tar.gz
    URL_HASH SHA256=abc123...
)

FetchContent_MakeAvailable(mylib archive_lib)
```

**控制下载行为：**

```cmake
FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 9.1.0
    GIT_SHALLOW TRUE       # 浅克隆，加快速度
    GIT_PROGRESS TRUE      # 显示下载进度
)
```

**配置依赖的选项：**

```cmake
# 在 FetchContent_MakeAvailable 前设置选项
set(FMT_INSTALL OFF CACHE BOOL "")  # 不安装 fmt，避免覆盖用户缓存可不加 FORCE

FetchContent_MakeAvailable(fmt)
```

### 7.4 ExternalProject

对于复杂的第三方库，ExternalProject 提供更多控制。

```cmake
include(ExternalProject)

ExternalProject_Add(
    libpng
    URL https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz
    PREFIX ${CMAKE_BINARY_DIR}/external/libpng
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    BUILD_COMMAND ${CMAKE_COMMAND} --build .
    INSTALL_COMMAND ${CMAKE_COMMAND} --install .
)

# 获取安装路径
ExternalProject_Get_Property(libpng INSTALL_DIR)

# 创建导入目标
add_library(PNG::PNG STATIC IMPORTED)
file(MAKE_DIRECTORY "${INSTALL_DIR}/include")
set_target_properties(PNG::PNG PROPERTIES
    IMPORTED_LOCATION ${INSTALL_DIR}/lib/libpng.a
    INTERFACE_INCLUDE_DIRECTORIES ${INSTALL_DIR}/include
)

# 确保依赖关系
add_dependencies(PNG::PNG libpng)

# 使用
add_executable(myapp main.cpp)
target_link_libraries(myapp PNG::PNG)
```

**下载和应用补丁：**

```cmake
ExternalProject_Add(
    somelib
    URL https://example.com/somelib.tar.gz
    PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/patches/fix.patch
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)
```

**区别：FetchContent vs ExternalProject**

| 特性       | FetchContent       | ExternalProject  |
| ---------- | ------------------ | ---------------- |
| 配置时机   | 配置阶段           | 构建阶段         |
| 集成方式   | 直接集成到当前项目 | 独立构建         |
| 使用复杂度 | 简单               | 复杂             |
| 适用场景   | 现代 CMake 项目    | 任何构建系统的库 |

### 7.5 vcpkg 集成

vcpkg 是微软推出的 C++ 包管理器。

**安装 vcpkg：**

```bash
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh  # Linux/macOS
.\bootstrap-vcpkg.bat  # Windows
```

**安装库：**

```bash
./vcpkg install fmt
./vcpkg install boost
./vcpkg install opencv
```

**CMake 集成：**

```bash
# 方法 1: 使用工具链文件
cmake -DCMAKE_TOOLCHAIN_FILE=/path/to/vcpkg/scripts/buildsystems/vcpkg.cmake ..

# 方法 2: 全局集成（Windows）
vcpkg integrate install
```

**CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.15)
project(MyProject)

# vcpkg 会自动处理 find_package
find_package(fmt CONFIG REQUIRED)
find_package(Boost REQUIRED COMPONENTS system filesystem)

add_executable(myapp main.cpp)
target_link_libraries(myapp 
    fmt::fmt
    Boost::system
    Boost::filesystem
)
```

**使用 vcpkg.json（清单模式）：**

项目结构：

```
project/
├── CMakeLists.txt
├── vcpkg.json        # 依赖清单
└── src/
```

**vcpkg.json：**

```json
{
  "name": "myproject",
  "version": "1.0.0",
  "dependencies": [
    "fmt",
    "boost-system",
    "boost-filesystem",
    "nlohmann-json"
  ]
}
```

**构建：**

```bash
cmake -DCMAKE_TOOLCHAIN_FILE=/path/to/vcpkg/scripts/buildsystems/vcpkg.cmake ..
cmake --build .
```

vcpkg 会自动安装清单中的依赖。

### 7.6 Conan 集成

Conan 是另一个流行的 C++ 包管理器。

**安装 Conan：**

```bash
pip install conan
```

**创建 conanfile.txt：**

```ini
[requires]
fmt/9.1.0
boost/1.81.0
poco/1.12.4

[generators]
CMakeDeps
CMakeToolchain

[options]
boost:shared=False
```

**集成到 CMake：**

```bash
# 安装依赖
conan install . --output-folder=build --build=missing

# 配置项目
cmake -DCMAKE_TOOLCHAIN_FILE=build/conan_toolchain.cmake -B build

# 构建
cmake --build build
```

**CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.15)
project(MyProject)

find_package(fmt REQUIRED)
find_package(Boost REQUIRED COMPONENTS system filesystem)

add_executable(myapp main.cpp)
target_link_libraries(myapp 
    fmt::fmt
    Boost::system
    Boost::filesystem
)
```

### 7.7 头文件库（Header-Only Libraries）

许多现代 C++ 库是纯头文件库，集成非常简单。

**方式 1: 直接包含**

```cmake
# 添加包含目录
include_directories(${CMAKE_SOURCE_DIR}/external/json/include)

add_executable(myapp main.cpp)
```

**方式 2: 接口库（推荐）**

```cmake
# 创建接口库
add_library(json INTERFACE)
target_include_directories(json INTERFACE 
    ${CMAKE_SOURCE_DIR}/external/json/include
)

add_executable(myapp main.cpp)
target_link_libraries(myapp json)
```

**方式 3: FetchContent**

```cmake
include(FetchContent)

FetchContent_Declare(
    json
    GIT_REPOSITORY https://github.com/nlohmann/json.git
    GIT_TAG v3.11.2
)

FetchContent_MakeAvailable(json)

add_executable(myapp main.cpp)
target_link_libraries(myapp nlohmann_json::nlohmann_json)
```

### 7.8 预编译库

使用预编译的二进制库。

**项目结构：**

```
project/
├── CMakeLists.txt
├── include/
│   └── mylib.h
├── lib/
│   ├── libmylib.a      # 静态库
│   └── libmylib.so     # 动态库
└── src/
    └── main.cpp
```

**CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.10)
project(PrecompiledDemo)

# 创建导入库
add_library(mylib STATIC IMPORTED)

# 设置库位置
set_target_properties(mylib PROPERTIES
    IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/lib/libmylib.a
    INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_SOURCE_DIR}/include
)

# 如果是动态库
add_library(mylib_shared SHARED IMPORTED)
set_target_properties(mylib_shared PROPERTIES
    IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/lib/libmylib.so
    INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_SOURCE_DIR}/include
)

add_executable(myapp src/main.cpp)
target_link_libraries(myapp mylib)
```

### 7.9 处理依赖的依赖

当第三方库本身有依赖时：

```cmake
# A 依赖 B，B 依赖 C

# 确保顺序正确
find_package(C REQUIRED)
find_package(B REQUIRED)
find_package(A REQUIRED)

add_executable(myapp main.cpp)

# 链接时顺序也很重要
target_link_libraries(myapp 
    A::A    # A 会自动拉入 B 和 C（如果正确声明了依赖）
)

# 如果自动依赖不生效，手动指定
target_link_libraries(myapp A::A B::B C::C)
```

---

## 第八部分：高级技巧与最佳实践

### 8.1 Option 选项

为用户提供配置选项。

```cmake
# 定义选项
option(BUILD_SHARED_LIBS "构建动态库" ON)
option(ENABLE_TESTS "启用测试" OFF)
option(USE_SYSTEM_LIBS "使用系统库而非捆绑版本" OFF)

# 使用选项
if(BUILD_SHARED_LIBS)
    add_library(mylib SHARED lib.cpp)
else()
    add_library(mylib STATIC lib.cpp)
endif()

if(ENABLE_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

# 打印选项
message(STATUS "BUILD_SHARED_LIBS: ${BUILD_SHARED_LIBS}")
message(STATUS "ENABLE_TESTS: ${ENABLE_TESTS}")
```

**命令行设置选项：**

```bash
cmake -DBUILD_SHARED_LIBS=OFF -DENABLE_TESTS=ON ..
```

### 8.2 缓存变量

缓存变量保存在 CMakeCache.txt 中，可被用户修改。

```cmake
# 定义缓存变量
set(MY_VAR "default_value" CACHE STRING "描述信息")

# 类型: BOOL, STRING, PATH, FILEPATH
set(ENABLE_FEATURE ON CACHE BOOL "启用某功能")
set(INSTALL_DIR "/usr/local" CACHE PATH "安装路径")
set(CONFIG_FILE "config.json" CACHE FILEPATH "配置文件")

# FORCE 强制覆盖缓存
set(MY_VAR "new_value" CACHE STRING "" FORCE)
```

### 8.3 目标属性

目标（target）有许多属性可以设置。

```cmake
add_executable(myapp main.cpp)

# 设置单个属性
set_target_properties(myapp PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
    OUTPUT_NAME "MyApplication"     # 输出文件名
    RUNTIME_OUTPUT_DIRECTORY bin    # 输出目录
)

# 设置多个目标的属性
set_target_properties(app1 app2 app3 PROPERTIES
    CXX_STANDARD 20
)

# 获取属性
get_target_property(APP_NAME myapp OUTPUT_NAME)
message("应用名称: ${APP_NAME}")

# 追加属性
set_property(TARGET myapp APPEND PROPERTY
    COMPILE_DEFINITIONS DEBUG_MODE=1
)
```

**常用属性：**

- `OUTPUT_NAME`: 输出文件名
- `VERSION`: 库版本
- `SOVERSION`: 共享库版本
- `POSITION_INDEPENDENT_CODE`: 是否生成位置无关代码（PIC）
- `COMPILE_DEFINITIONS`: 预处理器定义
- `LINK_FLAGS`: 链接器标志

### 8.4 配置文件生成

根据 CMake 变量生成配置头文件。

**config.h.in：**

```cpp
#ifndef CONFIG_H
#define CONFIG_H

#define PROJECT_NAME "@PROJECT_NAME@"
#define PROJECT_VERSION "@PROJECT_VERSION@"
#define PROJECT_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define PROJECT_VERSION_MINOR @PROJECT_VERSION_MINOR@

#cmakedefine ENABLE_FEATURE_X
#cmakedefine01 USE_FEATURE_Y

#define INSTALL_PREFIX "@CMAKE_INSTALL_PREFIX@"

#endif
```

**CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject VERSION 2.5.1)

option(ENABLE_FEATURE_X "启用功能 X" ON)
option(USE_FEATURE_Y "使用功能 Y" OFF)

# 配置头文件
configure_file(
    ${CMAKE_SOURCE_DIR}/config.h.in
    ${CMAKE_BINARY_DIR}/config.h
    @ONLY  # 仅替换 @VAR@，不替换 ${VAR}
)

# 添加生成的头文件路径
include_directories(${CMAKE_BINARY_DIR})

add_executable(myapp main.cpp)
```

**main.cpp：**

```cpp
#include <iostream>
#include "config.h"

int main() {
    std::cout << "项目: " << PROJECT_NAME << std::endl;
    std::cout << "版本: " << PROJECT_VERSION << std::endl;
  
#ifdef ENABLE_FEATURE_X
    std::cout << "功能 X 已启用" << std::endl;
#endif

#if USE_FEATURE_Y
    std::cout << "功能 Y 已启用" << std::endl;
#endif
  
    return 0;
}
```

**configure_file 的替换规则：**

- `@VAR@` 或 `${VAR}`：替换为变量值
- `#cmakedefine VAR`：如果 VAR 为真，生成 `#define VAR`，否则注释掉
- `#cmakedefine01 VAR`：生成 `#define VAR 0` 或 `#define VAR 1`

### 8.5 测试支持

使用 CTest 进行测试。

**项目结构：**

```
project/
├── CMakeLists.txt
├── src/
│   ├── CMakeLists.txt
│   └── math.cpp
├── include/
│   └── math.h
└── tests/
    ├── CMakeLists.txt
    ├── test_math.cpp
    └── test_utils.cpp
```

**顶层 CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.10)
project(MathProject)

set(CMAKE_CXX_STANDARD 17)

# 启用测试
enable_testing()

add_subdirectory(src)
add_subdirectory(tests)
```

**src/CMakeLists.txt：**

```cmake
add_library(mathlib math.cpp)
target_include_directories(mathlib PUBLIC ${PROJECT_SOURCE_DIR}/include)
```

**tests/CMakeLists.txt：**

```cmake
# 添加测试可执行文件
add_executable(test_math test_math.cpp)
target_link_libraries(test_math mathlib)

add_executable(test_utils test_utils.cpp)
target_link_libraries(test_utils mathlib)

# 注册测试
add_test(NAME MathTest COMMAND test_math)
add_test(NAME UtilsTest COMMAND test_utils)

# 设置测试属性
set_tests_properties(MathTest PROPERTIES 
    TIMEOUT 10              # 超时时间（秒）
    PASS_REGULAR_EXPRESSION "All tests passed"  # 通过条件
)
```

**运行测试：**

```bash
# 构建
cmake --build .

# 运行所有测试
ctest

# 详细输出
ctest --verbose

# 运行特定测试
ctest -R Math  # 运行名称匹配 Math 的测试

# 并行运行
ctest -j 4
```

**使用 Google Test：**

```cmake
# 使用 FetchContent 获取 GTest
include(FetchContent)
FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG v1.14.0
)

# 防止 GTest 覆盖父项目的编译选项（Windows）
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(googletest)

# 启用测试
enable_testing()

# 添加测试
add_executable(math_test test_math.cpp)
target_link_libraries(math_test 
    mathlib
    gtest_main  # 包含 main 函数
)

# 自动发现测试
include(GoogleTest)
gtest_discover_tests(math_test)
```

**test_math.cpp：**

```cpp
#include <gtest/gtest.h>
#include "math.h"

TEST(MathTest, Addition) {
    EXPECT_EQ(add(2, 3), 5);
    EXPECT_EQ(add(-1, 1), 0);
}

TEST(MathTest, Multiplication) {
    EXPECT_EQ(multiply(3, 4), 12);
    EXPECT_EQ(multiply(0, 5), 0);
}
```

### 8.6 代码覆盖率

使用 gcov/lcov 生成覆盖率报告（GCC/Clang）。

```cmake
# 添加覆盖率编译选项
option(ENABLE_COVERAGE "启用代码覆盖率" OFF)

if(ENABLE_COVERAGE)
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
        target_compile_options(mathlib PRIVATE --coverage)
        target_link_options(mathlib PRIVATE --coverage)
    else()
        message(WARNING "代码覆盖率仅支持 GCC 和 Clang")
    endif()
endif()
```

**生成覆盖率报告：**

```bash
# 配置并启用覆盖率
cmake -DCMAKE_BUILD_TYPE=Debug -DENABLE_COVERAGE=ON ..

# 构建并运行测试
cmake --build .
ctest

# 生成覆盖率报告
lcov --capture --directory . --output-file coverage.info
lcov --remove coverage.info '/usr/*' '*/tests/*' --output-file coverage_filtered.info
genhtml coverage_filtered.info --output-directory coverage_report

# 查看报告
open coverage_report/index.html
```

### 8.7 静态分析

集成静态分析工具。

**Clang-Tidy：**

```cmake
# 启用 clang-tidy
option(ENABLE_CLANG_TIDY "启用 clang-tidy" OFF)

if(ENABLE_CLANG_TIDY)
    find_program(CLANG_TIDY_EXE NAMES clang-tidy)
  
    if(CLANG_TIDY_EXE)
        set(CMAKE_CXX_CLANG_TIDY 
            ${CLANG_TIDY_EXE};
            -checks=*,-fuchsia-*,-google-*,-llvm-*;
            -header-filter=.*;
        )
        message(STATUS "启用 clang-tidy: ${CLANG_TIDY_EXE}")
    else()
        message(WARNING "找不到 clang-tidy")
    endif()
endif()
```

**cppcheck：**

```cmake
find_program(CPPCHECK_EXE NAMES cppcheck)

if(CPPCHECK_EXE)
    set(CMAKE_CXX_CPPCHECK 
        ${CPPCHECK_EXE};
        --enable=all;
        --suppress=missingIncludeSystem;
    )
endif()
```

**.clang-tidy 配置文件：**

```yaml
Checks: >
  -*,
  clang-analyzer-*,
  cppcoreguidelines-*,
  modernize-*,
  performance-*,
  readability-*

CheckOptions:
  - key: readability-identifier-naming.ClassCase
    value: CamelCase
  - key: readability-identifier-naming.FunctionCase
    value: camelBack
```

### 8.8 预编译头文件

加速编译时间。

```cmake
# CMake 3.16+
add_executable(myapp main.cpp utils.cpp helper.cpp)

# 设置预编译头
target_precompile_headers(myapp PRIVATE
    <iostream>
    <vector>
    <string>
    <memory>
    "common.h"
)

# 重用预编译头
add_executable(another_app other.cpp)
target_precompile_headers(another_app REUSE_FROM myapp)
```

**对于库：**

```cmake
add_library(mylib lib.cpp)
target_precompile_headers(mylib PUBLIC
    <algorithm>
    <functional>
)

# 使用库的目标会自动获得预编译头
add_executable(myapp main.cpp)
target_link_libraries(myapp mylib)
```

### 8.9 Unity Build（统一构建）

将多个源文件合并编译，减少编译时间。

```cmake
# CMake 3.16+
add_executable(myapp 
    main.cpp
    file1.cpp
    file2.cpp
    file3.cpp
)

# 启用 Unity Build
set_target_properties(myapp PROPERTIES
    UNITY_BUILD ON
    UNITY_BUILD_BATCH_SIZE 10  # 每批合并的文件数
)
```

**全局启用：**

```cmake
set(CMAKE_UNITY_BUILD ON)
set(CMAKE_UNITY_BUILD_BATCH_SIZE 10)
```

### 8.10 交叉编译

为不同平台编译。

**工具链文件（toolchain.cmake）：**

```cmake
# ARM Linux 交叉编译
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

# 指定交叉编译器
set(CMAKE_C_COMPILER arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)

# 设置查找路径
set(CMAKE_FIND_ROOT_PATH /usr/arm-linux-gnueabihf)

# 调整搜索行为
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

**使用工具链：**

```bash
cmake -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake ..
cmake --build .
```

**Android 交叉编译：**

```bash
cmake \
    -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
    -DANDROID_ABI=arm64-v8a \
    -DANDROID_PLATFORM=android-21 \
    ..
```

### 8.11 多语言项目

CMake 支持 C、C++、Fortran、CUDA 等多种语言。

**C 和 C++ 混合：**

```cmake
project(MixedProject LANGUAGES C CXX)

add_library(c_lib lib.c)
add_library(cxx_lib lib.cpp)

add_executable(myapp main.cpp)
target_link_libraries(myapp c_lib cxx_lib)
```

**CUDA 支持：**

```cmake
cmake_minimum_required(VERSION 3.18)
project(CudaProject LANGUAGES CXX CUDA)

add_executable(cuda_app main.cpp kernel.cu)

# 设置 CUDA 标准
set_target_properties(cuda_app PROPERTIES
    CUDA_STANDARD 14
    CUDA_SEPARABLE_COMPILATION ON
)

# CUDA 架构
set_property(TARGET cuda_app PROPERTY CUDA_ARCHITECTURES 75 80)
```

### 8.12 自定义命令

在构建过程中执行自定义操作。

**add_custom_command：**

```cmake
# 生成源文件
add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/generated.cpp
    COMMAND python3 ${CMAKE_SOURCE_DIR}/generate.py -o ${CMAKE_BINARY_DIR}/generated.cpp
    DEPENDS ${CMAKE_SOURCE_DIR}/generate.py
    COMMENT "生成源文件"
)

add_executable(myapp main.cpp ${CMAKE_BINARY_DIR}/generated.cpp)
```

**add_custom_target：**

```cmake
# 创建文档目标
add_custom_target(docs
    COMMAND doxygen ${CMAKE_SOURCE_DIR}/Doxyfile
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMENT "生成文档"
)

# 格式化代码
add_custom_target(format
    COMMAND clang-format -i ${CMAKE_SOURCE_DIR}/src/*.cpp
    COMMENT "格式化代码"
)

# 构建特定目标
# make docs
# make format
```

**依赖关系：**

```cmake
add_custom_target(prepare
    COMMAND echo "准备构建环境"
)

add_executable(myapp main.cpp)
add_dependencies(myapp prepare)  # myapp 依赖 prepare
```

### 8.13 打印和调试

调试 CMake 脚本的技巧。

```cmake
# 打印消息
message(STATUS "这是状态信息")      # -- 这是状态信息
message(WARNING "这是警告")         # CMake Warning: 这是警告
message(FATAL_ERROR "这是错误")     # CMake Error: 这是错误（停止配置）

# 打印变量
message(STATUS "CMAKE_CXX_COMPILER = ${CMAKE_CXX_COMPILER}")
message(STATUS "PROJECT_SOURCE_DIR = ${PROJECT_SOURCE_DIR}")

# 打印所有变量
get_cmake_property(_variableNames VARIABLES)
foreach(_variableName ${_variableNames})
    message(STATUS "${_variableName}=${${_variableName}}")
endforeach()

# 打印目标属性
get_target_property(SOURCES myapp SOURCES)
message(STATUS "myapp 的源文件: ${SOURCES}")

# 调试模式
set(CMAKE_VERBOSE_MAKEFILE ON)  # 显示完整编译命令

# 打印列表
set(MY_LIST a b c d)
message(STATUS "列表: ${MY_LIST}")           # a;b;c;d
list(JOIN MY_LIST ", " MY_LIST_STR)
message(STATUS "列表字符串: ${MY_LIST_STR}") # a, b, c, d
```

### 8.14 平台特定代码

根据平台执行不同操作。

```cmake
# 检测操作系统
if(WIN32)
    message(STATUS "Windows 平台")
    target_compile_definitions(myapp PRIVATE PLATFORM_WINDOWS)
    target_sources(myapp PRIVATE windows_specific.cpp)
elseif(APPLE)
    message(STATUS "macOS 平台")
    target_compile_definitions(myapp PRIVATE PLATFORM_MACOS)
    target_sources(myapp PRIVATE macos_specific.cpp)
elseif(UNIX)
    message(STATUS "Linux/Unix 平台")
    target_compile_definitions(myapp PRIVATE PLATFORM_LINUX)
    target_sources(myapp PRIVATE linux_specific.cpp)
endif()

# 检测编译器
if(MSVC)
    target_compile_options(myapp PRIVATE /W4 /WX)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(myapp PRIVATE -Wall -Wextra -Werror)
endif()

# 检测架构
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(STATUS "64 位系统")
else()
    message(STATUS "32 位系统")
endif()
```

### 8.15 模块化 CMake 代码

将常用功能提取为模块。

**项目结构：**

```
project/
├── CMakeLists.txt
├── cmake/
│   ├── CompilerWarnings.cmake
│   ├── Sanitizers.cmake
│   └── StaticAnalysis.cmake
└── src/
```

**cmake/CompilerWarnings.cmake：**

```cmake
function(set_project_warnings target)
    set(MSVC_WARNINGS
        /W4     # 警告级别 4
        /WX     # 警告视为错误
        /w14640 # 线程安全静态变量
    )

    set(CLANG_WARNINGS
        -Wall
        -Wextra
        -Wshadow
        -Wnon-virtual-dtor
        -Wold-style-cast
        -Wcast-align
        -Wunused
        -Woverloaded-virtual
        -Wpedantic
        -Wconversion
        -Wsign-conversion
        -Werror
    )

    set(GCC_WARNINGS
        ${CLANG_WARNINGS}
        -Wmisleading-indentation
        -Wduplicated-cond
        -Wduplicated-branches
        -Wlogical-op
    )

    if(MSVC)
        set(WARNINGS ${MSVC_WARNINGS})
    elseif(CMAKE_CXX_COMPILER_ID MATCHES ".*Clang")
        set(WARNINGS ${CLANG_WARNINGS})
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        set(WARNINGS ${GCC_WARNINGS})
    endif()

    target_compile_options(${target} PRIVATE ${WARNINGS})
endfunction()
```

**cmake/Sanitizers.cmake：**

```cmake
function(enable_sanitizers target)
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
        option(ENABLE_ASAN "启用 Address Sanitizer" OFF)
        option(ENABLE_UBSAN "启用 Undefined Behavior Sanitizer" OFF)
        option(ENABLE_TSAN "启用 Thread Sanitizer" OFF)

        set(SANITIZERS "")

        if(ENABLE_ASAN)
            list(APPEND SANITIZERS "address")
        endif()

        if(ENABLE_UBSAN)
            list(APPEND SANITIZERS "undefined")
        endif()

        if(ENABLE_TSAN)
            list(APPEND SANITIZERS "thread")
        endif()

        if(SANITIZERS)
            list(JOIN SANITIZERS "," SANITIZERS_STR)
            target_compile_options(${target} PRIVATE -fsanitize=${SANITIZERS_STR})
            target_link_options(${target} PRIVATE -fsanitize=${SANITIZERS_STR})
        endif()
    endif()
endfunction()
```

**顶层 CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.15)
project(ModularProject)

# 添加模块路径
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# 导入模块
include(CompilerWarnings)
include(Sanitizers)

add_executable(myapp main.cpp)

# 使用模块函数
set_project_warnings(myapp)
enable_sanitizers(myapp)
```

---

## 第九部分：完整项目示例

### 9.1 中型项目结构

一个实际的完整项目示例。

**项目结构：**

```
ImageProcessor/
├── CMakeLists.txt
├── README.md
├── LICENSE
├── .gitignore
├── cmake/
│   ├── CompilerWarnings.cmake
│   └── FindSomeLib.cmake
├── include/
│   └── imageproc/
│       ├── image.h
│       ├── filters.h
│       └── utils.h
├── src/
│   ├── CMakeLists.txt
│   ├── image.cpp
│   ├── filters.cpp
│   └── utils.cpp
├── apps/
│   ├── CMakeLists.txt
│   └── main.cpp
├── tests/
│   ├── CMakeLists.txt
│   ├── test_image.cpp
│   └── test_filters.cpp
├── docs/
│   ├── Doxyfile
│   └── README.md
├── external/
│   └── (第三方依赖)
└── scripts/
    └── build.sh
```

**顶层 CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.15)
project(ImageProcessor 
    VERSION 1.0.0
    DESCRIPTION "图像处理库"
    LANGUAGES CXX
)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 选项
option(BUILD_SHARED_LIBS "构建动态库" ON)
option(BUILD_APPS "构建应用程序" ON)
option(BUILD_TESTS "构建测试" ON)
option(BUILD_DOCS "构建文档" OFF)

# 输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# 添加 CMake 模块路径
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# 查找依赖
find_package(OpenCV REQUIRED)

# 添加子目录
add_subdirectory(src)

if(BUILD_APPS)
    add_subdirectory(apps)
endif()

if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

# 安装配置
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# 配置文件
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/ImageProcessorConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/ImageProcessorConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ImageProcessor
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/ImageProcessorConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

# 安装配置文件
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/ImageProcessorConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/ImageProcessorConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ImageProcessor
)

# 打包
set(CPACK_PACKAGE_NAME "ImageProcessor")
set(CPACK_PACKAGE_VENDOR "MyCompany")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.md")
include(CPack)
```

**src/CMakeLists.txt：**

```cmake
# 定义库源文件
set(IMAGEPROC_SOURCES
    image.cpp
    filters.cpp
    utils.cpp
)

set(IMAGEPROC_HEADERS
    ${PROJECT_SOURCE_DIR}/include/imageproc/image.h
    ${PROJECT_SOURCE_DIR}/include/imageproc/filters.h
    ${PROJECT_SOURCE_DIR}/include/imageproc/utils.h
)

# 创建库
add_library(imageproc ${IMAGEPROC_SOURCES})

# 设置目标属性
set_target_properties(imageproc PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
    PUBLIC_HEADER "${IMAGEPROC_HEADERS}"
)

# 包含目录
target_include_directories(imageproc
    PUBLIC
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
)

# 链接依赖
target_link_libraries(imageproc 
    PUBLIC ${OpenCV_LIBS}
)
# ImageProcessorConfig.cmake.in 中应使用 find_dependency(OpenCV)

# 编译选项
if(MSVC)
    target_compile_options(imageproc PRIVATE /W4)
else()
    target_compile_options(imageproc PRIVATE -Wall -Wextra)
endif()

# 安装
install(TARGETS imageproc
    EXPORT ImageProcessorTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/imageproc
)

# 导出目标
install(EXPORT ImageProcessorTargets
    FILE ImageProcessorTargets.cmake
    NAMESPACE ImageProcessor::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ImageProcessor
)
```

**apps/CMakeLists.txt：**

```cmake
add_executable(imageproc_app main.cpp)

target_link_libraries(imageproc_app PRIVATE imageproc)

install(TARGETS imageproc_app
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
```

**tests/CMakeLists.txt：**

```cmake
include(FetchContent)

FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG v1.14.0
)

set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

# 测试可执行文件
add_executable(image_tests
    test_image.cpp
    test_filters.cpp
)

target_link_libraries(image_tests
    imageproc
    gtest_main
)

# 自动发现测试
include(GoogleTest)
gtest_discover_tests(image_tests)
```

### 9.2 构建脚本

**scripts/build.sh：**

```bash
#!/bin/bash

set -e  # 遇到错误立即退出

BUILD_TYPE=${1:-Release}
BUILD_DIR="build_${BUILD_TYPE}"

echo "构建类型: ${BUILD_TYPE}"
echo "构建目录: ${BUILD_DIR}"

# 创建构建目录
mkdir -p ${BUILD_DIR}
cd ${BUILD_DIR}

# 配置
cmake .. \
    -DCMAKE_BUILD_TYPE=${BUILD_TYPE} \
    -DBUILD_TESTS=ON \
    -DBUILD_APPS=ON

# 构建
cmake --build . --parallel $(nproc)

# 运行测试
ctest --output-on-failure

echo "构建成功！"
```

---

## 第十部分：常见问题与解决方案

### 10.1 找不到库

**问题：** `find_package(XXX)` 找不到库

**解决方案：**

```cmake
# 1. 指定搜索路径
list(APPEND CMAKE_PREFIX_PATH "/opt/mylib")

# 2. 设置环境变量
# export CMAKE_PREFIX_PATH="/opt/mylib:$CMAKE_PREFIX_PATH"

# 3. 直接指定配置文件位置
cmake -DMyLib_DIR=/opt/mylib/lib/cmake/MyLib ..

# 4. 检查是否安装了开发包（Linux）
# sudo apt install libmylib-dev
```

### 10.2 链接顺序问题

**问题：** undefined reference 错误

**解决方案：**

```cmake
# 库的链接顺序很重要
# 静态库通常应把依赖者放在前、被依赖库放在后

# 正确
target_link_libraries(myapp libA libB)  # 如果 libA 依赖 libB

# 错误
target_link_libraries(myapp libB libA)

# 或者使用 PRIVATE/PUBLIC 自动处理依赖
add_library(libA a.cpp)
target_link_libraries(libA PUBLIC libB)  # libB 会自动传递给依赖 libA 的目标

add_executable(myapp main.cpp)
target_link_libraries(myapp libA)  # libB 自动链接
```

### 10.3 头文件找不到

**问题：** fatal error: xxx.h: No such file or directory

**解决方案：**

```cmake
# 检查包含目录是否正确设置
target_include_directories(myapp PRIVATE
    ${PROJECT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}
)

# 或使用 PUBLIC 使依赖者也能看到
add_library(mylib lib.cpp)
target_include_directories(mylib PUBLIC
    ${PROJECT_SOURCE_DIR}/include
)
```

### 10.4 生成器表达式不生效

**问题：** 生成器表达式在配置阶段没有值

**解决方案：**

```cmake
# 错误：在配置阶段使用
set(MY_VAR $<CONFIG:Debug>)  # 不会工作

# 正确：在目标属性中使用
target_compile_definitions(myapp PRIVATE
    $<$<CONFIG:Debug>:DEBUG_MODE>
)
```

### 10.5 中文路径问题

**问题：** Windows 下中文路径导致编译失败

**解决方案：**

```cmake
# 1. 避免使用中文路径

# 2. 设置编码（MSVC）
if(MSVC)
    add_compile_options(/utf-8)
endif()

# 3. 使用 file(TO_CMAKE_PATH)
set(MY_PATH "D:/中文目录/project")
file(TO_CMAKE_PATH "${MY_PATH}" MY_PATH_CMAKE)
```

### 10.6 缓存问题

**问题：** 修改 CMakeLists.txt 后没有生效

**解决方案：**

```bash
# 删除缓存
rm CMakeCache.txt

# 或完全重新构建
rm -rf build
mkdir build
cd build
cmake ..
```

### 10.7 并行构建失败

**问题：** 使用 `-j` 并行构建时出错

**解决方案：**

```cmake
# 普通库依赖用 target_link_libraries 声明
add_library(libA a.cpp)
add_library(libB b.cpp)

# libB 依赖 libA
target_link_libraries(libB libA)
```

如果并行失败来自生成文件或自定义命令，需在 `add_custom_command(DEPENDS ...)` 或 `add_dependencies()` 中显式声明依赖。

---

## 总结与最佳实践

### 核心最佳实践

1. **使用外源构建（Out-of-Source Build）**
   
   ```bash
      mkdir build
      cd build
      cmake ..
   ```
   
   2. **使用现代 CMake（3.15+）**
      - 优先使用 `target_*` 命令而非全局命令
      - 使用 `target_link_libraries`、`target_include_directories` 等
   
   3. **明确依赖关系**
      ```cmake
      # 推荐
      target_link_libraries(myapp PRIVATE mylib)
       
      # 不推荐
      link_libraries(mylib)  # 全局影响
      ```
   
   4. **使用 PRIVATE/PUBLIC/INTERFACE**
      ```cmake
      # PRIVATE: 仅当前目标使用
      # PUBLIC: 当前目标和依赖者都使用
      # INTERFACE: 仅依赖者使用
       
      target_include_directories(mylib
          PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/internal
          PUBLIC ${PROJECT_SOURCE_DIR}/include
      )
      ```
   
   5. **生成器表达式用于条件编译**
      ```cmake
      target_compile_options(myapp PRIVATE
          $<$<CONFIG:Debug>:-g -O0>
          $<$<CONFIG:Release>:-O3>
      )
      ```
   
   6. **使用 FetchContent 管理依赖**
      ```cmake
      include(FetchContent)
      FetchContent_Declare(fmt GIT_REPOSITORY ... GIT_TAG ...)
      FetchContent_MakeAvailable(fmt)
      ```
   
   7. **提供安装和导出**
      ```cmake
      install(TARGETS mylib EXPORT MyLibTargets ...)
      install(EXPORT MyLibTargets ...)
      ```
   
   8. **版本管理**
      ```cmake
      project(MyProject VERSION 1.2.3)
      ```
   
   9. **选项提供灵活性**
      ```cmake
      option(BUILD_TESTS "构建测试" ON)
      option(BUILD_SHARED_LIBS "构建动态库" OFF)
      ```
   
   10. **文档和注释**
       ```cmake
       # 说明复杂的配置逻辑
       # 标注为什么这样做，而不仅仅是做了什么
       ```
   
   ### 代码组织原则
   
   1. **按功能分层**
      - 顶层：项目配置、选项、子目录
      - 中层：库定义、应用程序
      - 底层：具体的源文件、测试
   
   2. **每个子目录有自己的 CMakeLists.txt**
      ```
      project/
      ├── CMakeLists.txt          # 主配置
      ├── src/CMakeLists.txt      # 库/应用
      ├── tests/CMakeLists.txt    # 测试
      └── docs/CMakeLists.txt     # 文档
      ```
   
   3. **分离接口和实现**
      ```cmake
      # 公共头文件放在 include/
      # 私有头文件和实现放在 src/
      target_include_directories(mylib
          PUBLIC ${PROJECT_SOURCE_DIR}/include
          PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}
      )
      ```
   
   ### 避免的陷阱
   
   1. **不要使用 GLOB 自动收集源文件**
      ```cmake
      # 不推荐
      file(GLOB SOURCES "*.cpp")
       
      # 推荐：明确列出
      set(SOURCES
          main.cpp
          utils.cpp
          helper.cpp
      )
      ```
   
   2. **不要使用全局命令**
      ```cmake
      # 不推荐
      include_directories(${SOME_DIR})
      link_libraries(somelib)
      add_definitions(-DDEBUG)
       
      # 推荐：针对特定目标
      target_include_directories(myapp PRIVATE ${SOME_DIR})
      target_link_libraries(myapp somelib)
      target_compile_definitions(myapp PRIVATE DEBUG)
      ```
   
   3. **不要在 project() 之前设置编译器**
      ```cmake
      # 错误
      set(CMAKE_CXX_COMPILER g++)
      project(MyProject)
       
      # 正确：使用命令行或工具链文件
      # cmake -DCMAKE_CXX_COMPILER=g++ ..
      ```
   
   4. **不要假设路径分隔符**
      ```cmake
      # 不推荐
      set(MY_PATH ${PROJECT_SOURCE_DIR}/include)
       
      # 推荐：使用 CMake 路径操作
      set(MY_PATH "${PROJECT_SOURCE_DIR}/include")
      file(TO_CMAKE_PATH "${MY_PATH}" MY_PATH)
      ```
   
   5. **不要忽略构建类型**
      ```cmake
      # 设置默认构建类型
      if(NOT CMAKE_BUILD_TYPE)
          set(CMAKE_BUILD_TYPE Release)
      endif()
      ```
   
   ### 性能优化技巧
   
   1. **使用 Ninja 生成器**
      ```bash
      cmake -G Ninja ..
      ninja
      ```
   
   2. **启用并行构建**
      ```bash
      cmake --build . --parallel $(nproc)
      ```
   
   3. **使用预编译头**
      ```cmake
      target_precompile_headers(myapp PRIVATE pch.h)
      ```
   
   4. **Unity Build**
      ```cmake
      set_target_properties(myapp PROPERTIES UNITY_BUILD ON)
      ```
   
   5. **ccache 加速**
      ```bash
      export CMAKE_CXX_COMPILER_LAUNCHER=ccache
      cmake ..
      ```
   
   ### 调试技巧
   
   1. **查看详细编译命令**
      ```bash
      cmake --build . --verbose
      # 或
      make VERBOSE=1
      ```
   
   2. **打印变量调试**
      ```cmake
      message(STATUS "CMAKE_CXX_FLAGS = ${CMAKE_CXX_FLAGS}")
      ```
   
   3. **检查目标属性**
      ```cmake
      get_target_property(SOURCES myapp SOURCES)
      message("Sources: ${SOURCES}")
      ```
   
   4. **使用 CMake GUI**
      ```bash
      cmake-gui .
      # 或
      ccmake .
      ```
   
   ### 版本控制
   
   **.gitignore 示例：**
   
   ```gitignore
   # 构建目录
   build/
   build_*/
   cmake-build-*/
   
   # CMake 生成文件
   CMakeCache.txt
   CMakeFiles/
   cmake_install.cmake
   Makefile
   *.cmake
   !CMakeLists.txt
   !*Config.cmake.in
   
   # 编译产物
   *.o
   *.a
   *.so
   *.dll
   *.exe
   
   # IDE 文件
   .vscode/
   .idea/
   *.user
   *.suo
   
   # 测试和覆盖率
   Testing/
   coverage/
   *.gcda
   *.gcno
   ```
   
   ### 文档化
   
   **README.md 模板：**
   
   ```markdown
   # 项目名称
   
   ## 依赖
   
   - CMake 3.15+
   - C++17 编译器
   - OpenCV 4.x
   
   ## 构建
   
   ```bash
   mkdir build
   cd build
   cmake ..
   cmake --build .
   ```
   
   ## 配置选项
   
   - `BUILD_TESTS`: 构建测试（默认: ON）
   - `BUILD_SHARED_LIBS`: 构建动态库（默认: OFF）
   - `CMAKE_BUILD_TYPE`: 构建类型（Debug/Release）
   
   ## 使用
   
   ```cmake
   find_package(MyProject REQUIRED)
   target_link_libraries(your_app MyProject::mylib)
   ```
   ```
   
   ### 持续集成示例
   
   **.github/workflows/cmake.yml：**
   
   ```yaml
   name: CMake Build
   
   on: [push, pull_request]
   
   jobs:
     build:
       runs-on: ${{ matrix.os }}
       strategy:
         matrix:
           os: [ubuntu-latest, windows-latest, macos-latest]
           build_type: [Debug, Release]
   
       steps:
       - uses: actions/checkout@v3
         with:
           submodules: recursive
   
       - name: 安装依赖 (Ubuntu)
         if: matrix.os == 'ubuntu-latest'
         run: |
           sudo apt-get update
           sudo apt-get install -y libopencv-dev
   
       - name: 配置
         run: cmake -B build -DCMAKE_BUILD_TYPE=${{ matrix.build_type }} -DBUILD_TESTS=ON
   
       - name: 构建
         run: cmake --build build --config ${{ matrix.build_type }}
   
       - name: 测试
         run: ctest --test-dir build -C ${{ matrix.build_type }} --output-on-failure
   ```
   
   ### 完整的项目模板
   
   **minimal-cmake-template/ (最小模板)**
   
   ```
   minimal-project/
   ├── CMakeLists.txt
   ├── README.md
   ├── LICENSE
   ├── .gitignore
   ├── include/
   │   └── mylib/
   │       └── mylib.h
   ├── src/
   │   ├── CMakeLists.txt
   │   └── mylib.cpp
   ├── apps/
   │   ├── CMakeLists.txt
   │   └── main.cpp
   └── tests/
       ├── CMakeLists.txt
       └── test_main.cpp
   ```
   
   **CMakeLists.txt (顶层)：**
   
   ```cmake
   cmake_minimum_required(VERSION 3.15)
   project(MinimalProject 
       VERSION 1.0.0
       DESCRIPTION "A minimal CMake project template"
       LANGUAGES CXX
   )
   
   # C++ 标准
   set(CMAKE_CXX_STANDARD 17)
   set(CMAKE_CXX_STANDARD_REQUIRED ON)
   set(CMAKE_CXX_EXTENSIONS OFF)
   
   # 选项
   option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
   option(BUILD_TESTS "Build tests" ON)
   
   # 包含 GNU 安装目录标准
   include(GNUInstallDirs)
   
   # 子目录
   add_subdirectory(src)
   add_subdirectory(apps)
   
   if(BUILD_TESTS)
       enable_testing()
       add_subdirectory(tests)
   endif()
   
   # 导出配置
   install(EXPORT ${PROJECT_NAME}Targets
       FILE ${PROJECT_NAME}Targets.cmake
       NAMESPACE ${PROJECT_NAME}::
       DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
   )
   
   # 包配置
   include(CMakePackageConfigHelpers)
   
   configure_package_config_file(
       ${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
       ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
       INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
   )
   
   write_basic_package_version_file(
       ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
       VERSION ${PROJECT_VERSION}
       COMPATIBILITY SameMajorVersion
   )
   
   install(FILES
       ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
       ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
       DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
   )
   ```
   
   ### 学习资源
   
   **官方文档：**
   - CMake 官方文档：https://cmake.org/cmake/help/latest/
   - CMake Tutorial：https://cmake.org/cmake/help/latest/guide/tutorial/
   
   **推荐书籍：**
   - "Professional CMake: A Practical Guide" by Craig Scott
   - "Mastering CMake" by Ken Martin and Bill Hoffman
   
   **在线资源：**
   - Modern CMake：https://cliutils.gitlab.io/modern-cmake/
   - Effective Modern CMake：https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1
   - CMake Examples：https://github.com/ttroy50/cmake-examples
   
   **社区：**
   - CMake Discourse：https://discourse.cmake.org/
   - Stack Overflow：标签 `cmake`
   
   ### 快速参考
   
   **常用命令速查：**
   
   ```bash
   # 配置
   cmake -B build                          # 在 build 目录配置
   cmake -S . -B build                     # 明确指定源码和构建目录
   cmake -DCMAKE_BUILD_TYPE=Release ..     # 指定构建类型
   cmake -G Ninja ..                       # 指定生成器
   
   # 构建
   cmake --build build                     # 构建
   cmake --build build --target myapp      # 构建特定目标
   cmake --build build --parallel 4        # 并行构建
   cmake --build build --verbose           # 详细输出
   
   # 安装
   cmake --install build                   # 安装
   cmake --install build --prefix /usr     # 指定安装前缀
   
   # 测试
   ctest --test-dir build                  # 运行测试
   ctest --test-dir build --verbose        # 详细输出
   ctest --test-dir build -R MyTest        # 运行匹配的测试
   
   # 打包
   cpack --config build/CPackConfig.cmake  # 生成安装包
   
   # 清理
   cmake --build build --target clean      # 清理构建产物
   ```
   
   **常用 CMake 变量：**
   
   ```cmake
   # 路径
   CMAKE_SOURCE_DIR              # 顶层源码目录
   CMAKE_BINARY_DIR              # 顶层构建目录
   CMAKE_CURRENT_SOURCE_DIR      # 当前 CMakeLists.txt 所在目录
   CMAKE_CURRENT_BINARY_DIR      # 当前构建目录
   PROJECT_SOURCE_DIR            # 当前项目源码目录
   PROJECT_BINARY_DIR            # 当前项目构建目录
   
   # 编译器
   CMAKE_C_COMPILER              # C 编译器
   CMAKE_CXX_COMPILER            # C++ 编译器
   CMAKE_CXX_STANDARD            # C++ 标准版本
   
   # 构建类型
   CMAKE_BUILD_TYPE              # Debug/Release/RelWithDebInfo/MinSizeRel
   
   # 系统信息
   CMAKE_SYSTEM_NAME             # 操作系统名称
   CMAKE_SYSTEM_PROCESSOR        # 处理器架构
   WIN32                         # Windows 系统
   APPLE                         # macOS 系统
   UNIX                          # Unix/Linux 系统
   
   # 输出目录
   CMAKE_RUNTIME_OUTPUT_DIRECTORY  # 可执行文件输出目录
   CMAKE_LIBRARY_OUTPUT_DIRECTORY  # 动态库输出目录
   CMAKE_ARCHIVE_OUTPUT_DIRECTORY  # 静态库输出目录
   
   # 安装路径
   CMAKE_INSTALL_PREFIX          # 安装前缀
   CMAKE_INSTALL_BINDIR          # bin 目录
   CMAKE_INSTALL_LIBDIR          # lib 目录
   CMAKE_INSTALL_INCLUDEDIR      # include 目录
   ```
   
   ---
   
   ## 结语
   
   CMake 是一个强大且灵活的构建系统生成器，掌握它需要时间和实践。本教程从基础概念开始，逐步深入到高级特性和最佳实践，涵盖了：
   
   1. **基础概念**：理解 CMake 的工作原理和术语
   2. **CMakeLists.txt 编写**：掌握基本语法和命令
   3. **项目构建**：学会配置、构建和安装
   4. **工程组织**：如何组织大型项目
   5. **安装部署**：打包和分发你的软件
   6. **库管理**：查找和使用第三方库
   7. **第三方库集成**：现代依赖管理方法
   8. **高级技巧**：测试、文档、CI/CD 等
   
   **学习建议：**
   
   1. **从小项目开始**：先写一个简单的 "Hello World" 项目
   2. **逐步增加复杂度**：添加库、测试、多目录结构
   3. **阅读他人的代码**：查看优秀开源项目的 CMake 配置
   4. **实践是关键**：遇到问题就查文档，动手解决
   5. **使用现代 CMake**：学习 3.15+ 的新特性和最佳实践
   
   **记住：**
   - CMake 是工具，不是目的——关注解决问题，而非炫技
   - 保持简单——不要过度工程化
   - 为未来考虑——写可维护、可扩展的配置
   - 文档很重要——让其他人（包括未来的你）能理解你的配置
   
   祝你在 CMake 的学习之旅中顺利！如果有任何问题，参考官方文档、社区资源，或实际动手尝试。CMake 的强大之处在于其灵活性，熟练掌握后，你将能轻松管理任何规模的 C/C++ 项目。
