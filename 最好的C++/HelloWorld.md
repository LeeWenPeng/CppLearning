## 1  创建项目目录

```bash
.
├── build.sh
├── CMakeLists.txt
└── src
    └── Main.cpp
```

## 2  CMakeLists.txt

```cmake
# 指定 cmake 版本
cmake_minimum_required(VERSION 4.1.2)

# 指定项目名
project(HelloWorld)

# 设置编译标志
# 1. 使用默认的编译项 ${CMAKE_CXX_FLAGS}
# 2. 显示所有的错误和警告 -Wall -Werror
# 3. 指定 C++ 版本为 14 -std=c++14
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -std=c++14")

# 设置源文件目录变量 source_dir
set(source_dir "${PROJECT_SOURCE_DIR}/src/")

# 指定要编译的源文件
file (GLOB source_files "${source_dir}/*.cpp")

# 创建可执行文件
add_executable(HelloWorld ${source_files})
```

## 3  执行 cmake 的脚本——build.sh

```bash
#!/bin/sh

cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Debug
```
