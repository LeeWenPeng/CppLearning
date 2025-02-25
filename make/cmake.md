# cmake 

```cmake
# 指定项目名
project(项目名)

#生成可执行文件
add_executable([执行文件名] [源文件1] [源文件2] ...)

# SET 指令的语法是：
# [] 中的参数为可选项, 如不需要可以不写
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])

# 指定C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)

# 给后续目标添加编译选项
add_compile_options(-g -Wunused)
add_compile_options([option1] [option2])
# 多个目标使用空格隔开
# -g 可调试
# -Wunused -W 是警告 -Wunused 为 -W unused 未使用警告

# 给指定目标添加编译选项
target_compile_options(app PUBLIC -Wall -Werror)
target_compile_options([目标名] [权限] [option1] [option2])
# 权限：PUBLIC
# -Wall
# -Werror -W error 

# 指定输出路径
# 通过宏 EXECUTABLE_OUTPUT_PATH 指定输出路径
set(EXECUTABLE_OUTPUT_PATH 路径)


# 搜索文件
# aux_source_directory 搜索路径下所有的源文件
aux_source_directory(目录路径 变量名)
# 1. 指定要搜索的目录
# 2. 将搜索到的源文件存储在变量中

file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
# GLOB 将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中。
# GLOB_RECURSE 递归搜索指定目录，将搜索到的满足条件的文件名生成一个列表，并将其存储到变量
# 要搜索的文件路径和文件类型 可加双引号，也可不加

# 制作静态库
add_library(库名称 STATIC 源文件1 [源文件2] ...) 
# 指定静态库生成位置
set(LIBRARY_OUTPUT_PATH 路径) # 也适用于动态库
# 设置静态库位置
# 如果静态库非系统提供，则需要指定一下静态库位置
link_directories(<lib path>)
# 链接静态库
# 静态库的链接应该是在生成可执行文件前
# 也就是 link_libraries 应该在 add_executable 代码之前
link_libraries(<static lib> [<static lib>...])

# 制作动态库
add_library(库名称 SHARED 源文件1 [源文件2] ...) 
# 指定动态库生成位置
set(EXECUTABLE_OUTPUT_PATH 路径)
# 链接动态库
# 动态库的链接应该是在生成可执行文件后
# 也就是 target_link_libraries 应该在 add_executable 代码之后
target_link_libraries(
    <target> 
    <PRIVATE|PUBLIC|INTERFACE> <item>... 
    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
    
    
# 字符串追加
# 使用场景：在有多个源文件夹时，需要使用多个搜索，这样就会创建出几个存放源文件的变量
# 1. set
# 将后续多个变量，追加到第一个变量中
set(变量1 ${变量1} ${变量2})
# 2. list
list(APPEND 变量名 ${变量1} ${变量2})
# APPEND 表示追加功能

# 字符串剔除
# 当搜索到的源文件中，有部分源文件是不需要的时候，可以使用 list 命令将这部分源文件去除
list(REMOVE_ITEM <list> <value> [<value> ...])
# REMOVE_ITEM 表示移除

# 定义宏
add_definitions(-D宏名称)

```

> 学习网址：
>
> https://subingwen.cn/cmake/CMake-primer/谢谢大丙
>
> [如何编译 C++ 程序：轻松搞定 CMake]:<<https://www.bilibili.com/video/BV15j411b7F8?vd_source=289a327f5dfc2718f181e3cad64ee708>
>
> 