## 1  CMAKE

- 不区分大小写
- `CMakeLists.txt` 是推荐的文件命名

## 2  CMake 编译建议

```bash
# 错误做法：污染源码目录
cmake .

# 正确做法：分离编译输出
mkdir build   # 在项目根目录创建
cd build
cmake ..      # 从build目录中运行，指定上级目录为源码根目录
make
```

## 3  基础命令

### 3.1  cmake_minimum_required

```cpp
# 指定使用的 CMake 最低版本号
cmake_minimum_required(VERSION tag)
```

- tag：该 CMakeLists.txt 所支持的最低版本号

### 3.2  project

定义项目信息

```cmake
project(MyProject
    VERSION 1.0.0      # 项目版本号
    DESCRIPTION "一个示例项目"  # 项目描述
    LANGUAGES C CXX    # 指定编程语言（C和C++）
)
```

- 该指令自动定义了四个变量
	- `PROJECT_NAME`：项目名称 (MyProject)
	- `${PROJECT_BINARY_DIR}`：项目源码根目录（绝对路径）
	- `${PROJECT_SOURCE_DIR}`：项目编译目录（绝对路径）
	- `CMAKE_PROJECT_VERSION`：项目版本号

### 3.3  set

SET 指令可以用来显式的定义变量，如果有多个 value，通过空格隔开

```cmake
# 基本用法
set(MY_VARIABLE "Hello World")

# 设置多个值（列表）
set(SOURCES 
    src/main.c
    src/utils.c
    src/config.c
)

# 设置缓存变量（用户可修改）
set(ENABLE_FEATURE ON CACHE BOOL "启用某个功能")
```

### 3.4  message

这个指令用于向终端输出用户定义的信息

```cmake
message(STATUS "配置信息：${PROJECT_NAME}")
message(WARNING "这是一条警告")
message(AUTHOR_WARNING "作者警告")
message(SEND_ERROR "错误信息，继续执行")
message(FATAL_ERROR "致命错误，立即停止")  # 最严重
```

### 3.5  add_executable

生成可执行文件

```cmake
# 简单用法
add_executable(my_app src/main.c)

# 使用变量
set(APP_SOURCES src/main.c src/utils.c)
add_executable(my_app ${APP_SOURCES})

# 生成多个可执行文件
add_executable(app1 src/app1.c)
add_executable(app2 src/app2.c)
```

### 3.6  add_subdirectory

此指令用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置。

```cmake
# 基本用法
add_subdirectory(src)
add_subdirectory(lib)

# 指定输出目录
add_subdirectory(src bin)  # 二进制文件输出到bin目录

# 排除编译（默认不编译，可通过目标依赖触发）
add_subdirectory(tests EXCLUDE_FROM_ALL)
```

- EXCLUDE_FROM_ALL: 将这个目录从编译过程中排除

## 4  安装配置

### 4.1  安装命令概览

|命令类型|用途|示例|
|---|---|---|
|**TARGETS**|安装目标（可执行文件、库）|`install(TARGETS myapp DESTINATION bin)`|
|**FILES**|安装普通文件|`install(FILES README.md DESTINATION share/doc)`|
|**PROGRAMS**|安装可执行脚本|`install(PROGRAMS script.sh DESTINATION bin)`|
|**DIRECTORY**|安装整个目录|`install(DIRECTORY data/ DESTINATION share/data)`|

### 4.2  详细安装配置

```cmake
#  安装可执行文件
install(TARGETS my_app
    RUNTIME DESTINATION bin # 可执行文件安装到 bin/
    LIBRARY DESTINATION lib # 动态库安装到 lib/
    ARCHIVE DESTINATION lib/static # 静态库安装到 lib/static/
)

# 安装头文件
install(FILES include/myheader.h
    DESTINATION include/${PROJECT_NAME}
)

# 安装整个目录
install(DIRECTORY docs/
    DESTINATION share/doc/${PROJECT_NAME}
    PATTERN "*.tmp" EXCLUDE # 排除临时文件
    PATTERN "*.md" PERMISSIONS OWNER_READ OWNER_WRITE
)

# 6 安装时根据配置区分
install(TARGETS my_app
    CONFIGURATIONS Debug
    RUNTIME DESTINATION bin/debug
    OPTIONAL # 如果目标不存在也不报错
)
```

- TARGETS：目标
- FILES：文件
- DIRECTORY：目录
- SCRIPT：脚本
- CODE：源文件
- EXPORT  

## 5  库管理

### 5.1  创建和使用库

#### 5.1.1  引入库文件

```cmake
# 根目录 CMakeLists.txt
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# 添加子目录
add_subdirectory(lib)    # 包含库的源码
add_subdirectory(src)    # 包含可执行文件的源码
```

#### 5.1.2  创建库文件

```cmake
# lib/CMakeLists.txt - 创建库

# 添加源文件
set(LIB_SOURCES
    module1.c
    module2.c
    utils.c
)

# 创建动态库（共享库）
add_library(mylib SHARED ${LIB_SOURCES})

# 创建静态库（可选）
add_library(mylib_static STATIC ${LIB_SOURCES})
set_target_properties(mylib_static PROPERTIES OUTPUT_NAME mylib)

# 设置库版本
set_target_properties(mylib PROPERTIES
    VERSION 1.0.0
    SOVERSION 1
)
```

**`add_library`**：
- `SHARED`：创建动态库（.so, .dll）
- `STATIC`：创建静态库（.a, .lib）
- `MODULE`：插件模块，不被链接，运行时加载

#### 5.1.3  链接库文件

```cmake
# src/CMakeLists.txt - 使用库
# 添加可执行文件
add_executable(myapp main.c)

# 添加头文件路径
target_include_directories(mylib
    PRIVATE  # 私有头文件（实现细节）
        src/
        internal/
    
    PUBLIC   # 公共头文件（接口）
        include/
        ${CMAKE_CURRENT_SOURCE_DIR}/api/
    
    INTERFACE # 纯头文件库的接口
        # 这个例子中未使用
)

# 链接库文件
target_link_libraries(myapp 
    PRIVATE mylib  # 链接前面创建的库
)

# 或者链接外部库
find_package(OpenSSL REQUIRED)
target_link_libraries(myapp
    PRIVATE OpenSSL::SSL
    PRIVATE OpenSSL::Crypto
)
```

- `PRIVATE`：仅当前目标使用
- `PUBLIC`：当前目标和链接此目标的其他目标都使用
- `INTERFACE`：仅给链接此目标的其他目标使用

> [!note] 链接库文件时，库名同时匹配到静态库和动态库，会链接哪个？
> - 会优先链接动态库
> - 可以通过写全名的方式，指定链接静态库，例如，`libname.a`
