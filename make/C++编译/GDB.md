## 1  介绍

GDB（GNU Debugger） 是 GNU 项目开发的一款命令行调试工具，主要用于分析和修复程序中的错误（如崩溃、逻辑错误、内存泄漏等）。它支持 C、C++、Go、Rust 等多种编程语言，是 Linux/Unix 开发者调试程序的核心工具之一。

## 2  核心功能

1. **控制程序执行**

	- 启动程序，暂停在指定位置（断点）。
	- 逐行执行代码（单步调试）。
	- 修改变量值，动态改变程序行为。

2. **诊断程序错误**

	- 分析程序崩溃原因（如段错误 `Segmentation Fault`）。
	- 检测内存泄漏、非法内存访问。
	- 查看函数调用栈，定位错误源头。

3. **动态观察程序状态**

	- 查看变量、寄存器、内存内容。
	- 监控条件断点（如变量值变化时暂停）。

## 3  GDB 基本使用步骤

### 3.1  **编译程序时启用调试信息**

   ```bash
   gcc -g -Wall fileName.c -o fileName
   ```

- `-O`：优化选项，调试编译时，往往会关闭掉
- `-g`：调试选项。生成一个带有调试信息的可执行程序。
	- 在可执行文件中加入源代码信息。
	- 必须保证 gdb 能够找到源文件。
- `-Wall`：不影响程序的情况下选项打开所有 warning

### 3.2  **启动 GDB**

   ```bash
gdb ./需要调试的文件
   ```

## 4  **常用命令**

### 4.1  退出和帮助

```gdb
# 退出GDB
q/quit

# 帮助手册
help <options>
```

`<options>`：

- `all`：所有命令
- `command`：指定命令

### 4.2  设置和获取参数

```gdb
# 设置主程序参数
set args nums...
# 显示参数
show agrs
```

- 多个参数值之间使用空格隔开

### 4.3  查看代码

```gdb
l/list [文件名:][<行号/函数名>]

# 示例
l bubble.cpp:10
l select.cpp:selectSort
```

- 默认文件：main 函数所在文件
- 默认开始行号：0(+n\*10) 行
- 默认显示行数：10 行

### 4.4  设置断点

```gdb
# 1 设置断点
b/break [文件名:]<行号/函数名>

# 2 查看断点信息
i/info b/break

# 3 删除断点
d/del/delete 断点编号

# 4 设置断点可用
ena/enable 断点编号

# 5 设置断点不可用
dis/disable 断点编号

# 6 设置条件断点，一般用于循环
b/break [文件名:]<行号/函数名> if condition
```

### 4.5  调试相关

```gdb
# 1 运行gdb
start # 程序停留在第一行
run # 运行到断点停止

# 2 继续运行，直到遇到下一个断点或程序结束
c/continue

# 3 向下执行一行代码
n/next # 不进入函数体
s/step # 进入函数体
finish # 跳出函数体 

# 4 变量操作
set var 变量名=变量值 # 设置变量值
p/print 变量名 # 打印变量值
ptype 变量名 # 打印变量类型

# 5 自动变量
display num
i/info # 打印自动变量
undisplay ID # 取消自动变量

# 6 跳出操作
finish # 跳出函数体
until # 跳出循环
```

- 程序停留在某一行时，该行为未执行行

### 4.6  反汇编命令

```gdb
# 查看函数调用栈。其中包含每一个函数的的调用信息
bt # 查看函数调用栈，其中包含每一个函数的调用信息

# 反汇编某一行
info line n # n 行号； 查询第 n 行的信息，其中包含代码的起始位置和结束位置
disassmeble begin_address end_address

# 反汇编某一个函数
disassmeble 函数名 # 反汇编某函数，没有 C_C++ 代码
disassmeble /m 函数名 # 反汇编某函数，代码与汇编一一对应
```

## 5  **高级功能**

1. **多线程调试**
   - `info threads`：查看所有线程。
   - `thread <ID>`：切换到指定线程。

2. **内存检查**
   - `x/<格式> <地址>`：检查内存内容（如 `x/4xw 0x7fffffffe220`）。

3. **脚本化调试**
   - 将调试命令写入脚本文件，批量执行：

	 ```bash
     gdb -x commands.gdb ./my_program
     ```

4. **图形化界面**
   - 使用 GDB 前端工具（如 [**DDD**](https://www.gnu.org/software/ddd/) 或 [**CGDB**](https://cgdb.github.io/)）提升可视化体验。

## 6  调试 core 文件

### 6.1  生成 core 文件

```bash
# 查询系统限制参数
ulimit -a

# 查看 core 文件限制
ulimit -c

# 修改 core 文件数量为不限制
ulimit -c ulimited

# 运行需要调试的程序以生成 core 文件
./需要调试的程序
```

### 6.2  编译 core 文件

```sh
g++ 需要调试的程序 生成的core文件
```

```gdb
bt # 查看函数调用栈
```

## 7  GDB 多进程调试

使用 GDB 调试时，GDB 默认只跟踪一个进程，可以在 fork 函数调用前，通过指令设置 GDB 跟踪父进程或者子进程。

### 7.1  设置跟踪父进程或者子进程

```gdb
set follow-fork-mode [parent | child]

 // 查看 GDB 跟踪的进程是父进程还是子进程
show follow-fork-mode
```

- 默认为父进程

### 7.2  设置调试模式

```gdb
set detach-on-fork [on | off]

// 查看调试模式
show detach-on-fork
```

- 默认为 on
- on：表示调试当前进程时，其他进程继续执行
- off：调试当前进程的时候，其他进程被 GDB 挂起
	- 禁止自动分离非跟踪的进程（允许同时调试父子进程）

### 7.3  查看调试的进程

```bash
info inferiors
```

- 获取进程编号

### 7.4  切换当前调试的进程

```bash
inferior Num
```

- Num：通过上一个步骤获取的编号

### 7.5  使进程脱离 GDB 调试

```bash
detach inferiors id
```
