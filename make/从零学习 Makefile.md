## 1. 前言

课程网址：[从零开始学Makefile](https://www.bilibili.com/video/BV1Bv4y1J7QT/?spm_id_from=333.337.search-card.all.click&vd_source=d9e178b992882410dc0927d40741958a)

make 手册：[make 文档](https://www.gnu.org/software/make/manual/make.html)

目的：面对大型项目多文件管理的情况

>[!summary] make (GUN make) 项目构建工具
管理项目内哪些文件需要更新以及如何更新

>[!summary] makefile
> 规则文件，告诉 make 如何进行编译和管理

### 1.1. 实验素材

1. [2048小游戏](https://github.com/plibither8/2048.cpp)
2. [数独小游戏](https://github.com/mayerui/sudoku)

### 1.2. 信息

| Suffix | File Contains |
| ------ | ------------- |
| `.a`   | 静态库文件         |
| `.c`   | C 语言源文件       |
| `.h`   | C 语言头文件       |
| `.i`   | C 语言预处理文件     |
| `.o`   | 目标文件          |
| `.s`   | 汇编语言文件        |
| `.so`  | 动态库文件         |

## 2. 2

## 3. 最简单的Makefile

### 3.1. C++ 源文件：hello.cpp

```cpp
#include <iostream>
using namespace std;

int main(){
    cout << "Hello World!" << endl;
    return 0;
}
```

### 3.2. Makefile 文件

```makefile
hello: hello.cpp
	g++ -o hello hello.cpp
```

更复杂写法

```makefile
hello: hello.o
	g++ -o hello hello.o
hello.o: hello.cpp
	g++ -c hello.cpp
```

>[!important]
>缩进处是tab缩进，而非空格缩进。

### 3.3. 命令执行

```shell
# 调用make
make

# 执行文件
./hello
```

### 3.4. 规则

![[Make#3 规则]]

## 4. Makefile 文件

### 4.1. Makefile 文件的命令与指定

[[Make#2.1 Makefile 文件的命名规则与查找方式]]

### 4.2. Makefile 文件内容组成

[[Make#2.2 Makefile 文件内容组成]]

## 5. 一个稍微复杂的 Makefile

针对数独游戏素材的Makefile文件编写

![[数独软件的makefile文件|1000]]

在 Makefile 文件中code编写的顺序是依赖顺序，是与实际编译顺序相反的

具体来说：

+ 首先，先写出最终目标
+ 然后依次解决总目标的依赖

### 5.1. 代码

```makefile
.DEFALUT_GOAL = sudoku

sudoku: block.o command.o i18n.o input.o main.o scene.o
	g++ -o sudoku block.o command.o i18n.o input.o main.o scene.o 

block.o: block.cpp block.h common.h color.h display_symbol.h
	g++ -c block.cpp

command.o: command.cpp scene.h common.h block.h command.h
	g++ -c command.cpp 

i18n.o: i18n.cpp i18n.h
	g++ -c i18n.cpp

input.o: input.cpp common.h i18n.h utility.inl
	g++ -c input.cpp

main.o: main.cpp i18n.h input.h common.h scene.h block.h command.h system_env.hpp utility.inl
	g++ -c main.cpp

scene.o: scene.cpp scene.h common.h block.h command.h display_symbol.h i18n.h utility.inl color.h
	g++ -c scene.cpp

clean:
	rm block.o command.o i18n.o input.o main.o scene.o
	rm sudoku

```

>+ 查看 cpp 文件生成.o文件所依赖文件的命令`g++ -MM 文件名.cpp`
>+ hpp 后缀文件不需要编译
>+ 依赖中的头文件可以省略掉，但不建议
>+ 如果目标与最终目标没有直接或间接的依赖关系，就不会自动执行，比如上述的`clean`目标
>+ **如果依赖没有改变，make工具将不会重新编译和链接**

>[!question] 为什么不建议省略掉依赖中的头文件？
>+ 依赖中的头文件省略掉，不会影响初次编译
>+ 但，如果不将头文件加入到头文件中，那么在头文件修改后，引入头文件的文件将不会重新编译

>[!question] 为何 Makefile 要分成编译和链接两步
>+ **如果依赖没有改变，make工具将不会重新编译和链接**
>+ 如果编译和链接没有分开，那么只要有文件改变，所有的文件都需要重新编译重新链接
>+ 将编译和链接分开后，只需要对更新的文件进行重新的编译，最后进行重新链接

## 6. 伪目标

![[Make#伪目标]]

## 7. 依赖类型

依赖分为普通依赖与 order-only 依赖两种类型。

两种依赖通过`|`分隔，左边为普通依赖列表，右边为order-only列表。

```makefile
target : 普通依赖1 普通依赖2 ... | order-only1 order-only2 ...
```

### 7.1. 普通依赖特点

+ 如果这一依赖是由其他规则生成的文件，那么当执行到该规则时，会先执行生成这一依赖的规则。
+ 如果任何一个依赖目标最终修改时间比目标晚，则重新编译生成目标文件。

### 7.2. order-only 依赖特点

+ 当依赖文件不存在时，会执行对应方法生成。
+ **依赖文件的更新不会导致目标文件的更新**。
	+ 检查目标文件更新时，不会检查 order-only 依赖文件的更新

## 8. 关于方法的一些问题

### 8.1. Shell 命令的执行问题

方法的本质为一系列 Shell 命令。这些命令需要交给终端执行，所以需要符合 Shell 语法。**默认使用的 Shell 为 sh**，可以通过`SHELL`变量来手动指定 Shell。

对于方法中的多条 Shell 命令，

+ 默认的执行方式为**对每一条命令调用独立 Shell 进程执行**。
+ 可以通过添加关键字`.ONESHELL`，使得所有的Shell 命令调用同一个 Shell 进程中执行

```cpp
.ONESHELL:
```

### 8.2. Shell 命令的回显问题

在执行Shell命令时，首先会打印 Shell 命令，然后再打印 Shell 命令的执行结果。

取消Shell命令回显的两种方式：

1 在具体命令前添加`@`

```makefile
target: prerequisites ...
	@Shell命令 操作 参数
```

2 使用关键字`.SILENT`，将不需要打印命令的目标，设置为`.SILENT`的依赖

```makefile
.SILENT: target1 target2 ...
```

### 8.3. 错误处理

当方法中有多条 Shell 语句时，

+ 只有当一条语句正确执行后，之后的语句才能执行；
+ 当一条语句执行错误时，则之后的所有语句都将不会执行。

通过在命令前添加`-`的方式，使得当命令执行错误时，make工具忽略错误，继续执行之后的 shell 命令。

```makefile
target: prerequisites ...
	-Shell 命令1
	-Shell 命令2
	Shell 命令3
```

## 9. 使用变量简化

[[Make#7 变量]]

## 10. Makefile执行过程

[[Make#变量的展开时机]]

## 11. 变量赋值

[[Make#变量的赋值]]

## 12. 定义多行变量

[[Make#多行变量的定义]]

## 13. 取消变量定义

[[Make#取消变量定义]]

## 14. 环境变量的使用

系统中环境变量在 Makefile 中是可以直接使用的

## 15. 变量替换引用

对 var2 中的值进行字符串替换操作后，再赋值给 var 变量

```makefile
var = $(var2:%字符1=%字符2)
```

例子：

```makefile
files != ls *.cpp
objs = $(files:%.cpp=%.o)
all:
	$(info $(objs)) 
# 打印结果为：block.o command.o i18n.o input.o main.o scene.o
```

## 16. 变量覆盖

Makefile 文件中定义的变量，可以在执行 make 命令时进行覆盖

```shell
make 变量名 = 变量值
```

通过关键字`override`，可以限制这种行为：

```makefile
override 变量名 = 值
```

## 17. 绑定目标的变量

Makefile 中的变量一般是全局变量，也就是说，在变量定义后，可以在Makefile文件中任意位置对变量进行引用。

在变量定义时，在变量名前添加`目标:`的方式，绑定目标和变量，限制该变量的使用范围在该目标内。

```makefile
目标名: 变量名 = 值
```

### 17.1. 一次只能绑定一个变量

```makefile
目标名: 变量名1 = 值 变量名2 = 值
```

上述代码中，`变量名1`为变量名，而第一个等号后的所有字符，都将作为值存在，也就是，`值 变量名2 = 值`。

### 17.2. 配合模式匹配使用

使用`%`的模式匹配串，**将变量与所有匹配到的目标进行绑定**

```makefile
%字符串: 变量名 = 值
```

### 17.3. 例子

```makefile
global_var = This is a global var!

.PHONY = all

all: first.c second.c third.c

first.c: target_var = This is target var! 

%.o: target2_var = This is target var!

first.c:
	@echo first.c global_var = $(global_var)
	@echo first.c target_var = $(target_var)
	@echo first.c target2_var = $(target2_var)
second.o:
	@echo second.c global_var = $(global_var)
	@echo second.c target_var = $(target_var)
	@echo second.c target2_var = $(target2_var)

third.o:
	@echo third.c global_var = $(global_var)
	@echo third.c target_var = $(target_var)
	@echo third.c target2_var = $(target2_var)

```

在上述代码中

+ global_var 全局变量
+ target_var 绑定到 first.c
+ target2_var 绑定到所有的 .o 文件，也就是 second.o 和 third.o

## 18. 自动变量

|  代码   | 解释                                                                  |
| :---: | ------------------------------------------------------------------- |
| `$@`  | + 本条规则的目标名；<br>+ 如果目标为归档文件的成员，则为归档文件名；<br>+ 多目标的模式规则中，为导致本条规则执行的目标名 |
| `$<`  | 本条规则中，第一个依赖                                                         |
| `$?`  | 依赖中有改动的文件，以空格隔开                                                     |
| `$^`  | 所有简单依赖文件名，文件名不会重复且不包括order-only依赖                                   |
| `$+`  | 所有简单依赖文件名，包括重复的文件名，但不包括order-only依赖                                 |
| `$\|` | 所有order-only依赖名                                                     |
| `$*`  | 目标文件名的主干部分，也就是不包括后缀名                                                |
| `$%`  | + 如果目标不是归档文件，则为空<br>+ 如果目标是归档文件成员，则为对应的成员文件                         |
| `$@D` | 对应变量所在的目录路径，但是不包括最后的`/`                                             |
| `$@F` | 对应变量去除目录部分的文件名                                                      |

> 有改动的意思就是其修改时间晚于目标文件的修改时间

使用自动变量简化后的 Makefile 文件

```makefile
.DEFALUT_GOAL = sudoku
objs = block.o command.o i18n.o input.o main.o scene.o

sudoku: $(objs)
	g++ -o sudoku $^

block.o: block.cpp block.h common.h color.h display_symbol.h
	g++ -c $<

command.o: command.cpp scene.h common.h block.h command.h
	g++ -c $<

i18n.o: i18n.cpp i18n.h
	g++ -c $<

input.o: input.cpp common.h i18n.h utility.inl
	g++ -c $<

main.o: main.cpp i18n.h input.h common.h scene.h block.h command.h system_env.hpp utility.inl
	g++ -c $<

scene.o: scene.cpp scene.h common.h block.h command.h display_symbol.h i18n.h utility.inl color.h
	g++ -c $<

clean:
	-rm $(objs)
	rm sudoku

```

## 19. 二次展开

通过关键字`.SECONDEXPANSION`可以将变量设置为延迟展开

```makefile
.SECONDEXPANSION:
target: $$(var)
```

+ 规则前需要添加`.SECONDEXPANSION`伪目标
+ 变量引用部分需要使用`$$`

>[!question] 如果使用了`$$`，但没有在此之前添加`.SECONDEXPANSION`伪目标，会如何？
>结果：相当于使用了`$`的转义字符，终端处报错，错误原因为不存在该var
>原因：`$$`将转义成`$`，并将`$(var)`发送给 sh，但 sh 无法识别该变量

>[!question] 如果使用了`.SECONDEXPANSION`，但没有使用`$$`，会如何？
>结果：不会将变量设置为延迟展开
>原因：相当于正常引用了变量

## 20. 指定依赖路径

项目结构的改变

+ 将头文件置入`include`文件夹内
+ 将代码源文件置入`src`文件夹内

当 Makefile 文件与头文件和源文件处于不同的文件夹时，需要指定依赖路径：

### 20.1. 通过变量 VPATH 指定搜索路径

```makefile
VPATH = dirPath1 : dirPath2 : ...
```

+ dirPath：就是需要搜索的目录路径，可以是相对路径，也可以是绝对路径
+ 多个目录之间用 `:` 隔开

设置VPATH之后，make工具在建立依赖图时，便会从给出的路径中搜索

#### 20.1.1. 为源文件指定头文件搜索路径

make 会将更新方法传递给 sh 执行，sh 将调用 g++ 对源文件进行编译，这时就会触发源文件编译时找不到头文件的错误。因为，我们将源文件放入了 src 文件夹，而头文件则放入了 include 文件夹。因此，需要在更新方法里为 g++ 工具指出头文件所在目录路径。

```makefile
target: prereq
	g++ ... -Iinclude
```

#### 20.1.2. 使用 VPATH 修改 Makefile 文件

```makefile
.DEFALUT_GOAL = sudoku
objs = block.o command.o i18n.o input.o main.o scene.o

VPATH = src:include

sudoku: $(objs)
	g++ -o $@ $^

block.o:  block.cpp block.h common.h color.h display_symbol.h
command.o: command.cpp command.h scene.h common.h block.h 
i18n.o: i18n.cpp i18n.h
input.o:  input.cpp common.h i18n.h utility.inl
main.o:  main.cpp i18n.h input.h common.h scene.h block.h command.h system_env.hpp utility.inl
scene.o: scene.cpp scene.h common.h block.h command.h display_symbol.h i18n.h utility.inl color.h

$(objs) : 
	g++ -c $< -Iinclude

.PHONY: clean

clean:
	-rm $(objs)
	rm sudoku
```

### 20.2. 通过 vpath 指令指定搜索路径

vpath 用法比 VPATH 使用更加灵活，可以为make指定文件类型的搜索路径

```makefile
vpath pattern dirPath
```

例子

```makefile
# .h 文件夹在 include 目录内搜索
vpath %.h include

# .h 文件夹在 include/header 目录内搜索
vpath %.h include:header

# 所有文件在 src 中搜索
vpath % src

# inputh.cpp 文件在 src 中搜索
vpath input.cpp src
```

#### 20.2.1. 使用 vpath 修改 Makefile 文件

```makefile
.DEFALUT_GOAL = sudoku
objs = block.o command.o i18n.o input.o main.o scene.o

# 大写的 VPATH 变量
# VPATH = dirPath : dirPath : dirPath : ...
# VPATH = src:include

# .cpp 文件在 src 中搜索
# 其他文件在 include 中搜索
vpath %.cpp src
vpath % include


sudoku: $(objs)
	g++ -o $@ $^

block.o:  block.cpp block.h common.h color.h display_symbol.h
command.o: command.cpp command.h scene.h common.h block.h 
i18n.o: i18n.cpp i18n.h
input.o:  input.cpp common.h i18n.h utility.inl
main.o:  main.cpp i18n.h input.h common.h scene.h block.h command.h system_env.hpp utility.inl
scene.o: scene.cpp scene.h common.h block.h command.h display_symbol.h i18n.h utility.inl color.h

# 为 g++ 工具指定 .h 文件的搜索路径
$(objs) : 
	g++ -c $< -Iinclude

.PHONY: clean

clean:
	-rm $(objs)
	rm sudoku

```

## 21. 条件判断

### 21.1. ifdef 和 ifndef

判断变量是否已经定义

```makefile
ifdef var
# todo... 如果变量var已经定义需要做的事情
else ifdef var2
# todo... 如果变量var2已经定义需要做的事情
else
# todo... 如果变量未定义
endif
```

+ 允许嵌套
+ `ifndef` 和 `ifdef`相反

### 21.2. ifeq

判断两个值是否相等

```makefile
# 下面一行代码中 ifeq 和 () 之间有一个空格，不能省略
ifeq (var1, var2)
# 
else
# 
enif

# 除了使用括号外，也可以单独对每一个变量或字符串使用引号
# 引号可以是单引号也可以是双引号
ifeq 'var1' 'var2'

ifeq "var1" "var2"

ifeq 'var1' "var2"
```

+ `ifeq`和`()`之间需要有一个空格
+ 允许嵌套
+ `ifneq` 的判断和 `ifeq` 相反

## 22. 字符处理函数

[[Make#8.1 字符串处理函数]]

## 23. 24

[[Make]]

[[Make]]

## 24. 自动推导与隐式规则

## 25. 引入其他 Makefile 文件

**使用 `include` 指令可以读入其他 Makefile 文件的内容**，效果如同**在 `include` 位置用对应的文件内容替换**

```makefile
-include mk1 mk2 ...

# 允许使用通配符
-include mk*
```

+ 引入文件可以有多个，以空格隔开
+ 如果查找不到，就会报错，停止 make 运行。因此，在命令前添加`-`，在报错时跳过[[#6 关于方法的一些问题#错误处理]]

>[!note]
>+ 使用 `include` 指令，相当于，在使用该指令的位置处，插入对应的 Makefile 文件代码
>+ 类似于 C++ 中的 `include` 关键字

### 25.1. 使用 include 和多规则更新 Makefile

不再直接写入 cpp 文件需要的多条依赖。

+ 借助 shell 语句自动获取依赖，存入到 `.d` 文件中；
+ 采用 `include` 插入依赖

```makefile
.DEFALUT_GOAL = sudoku
objs = block.o command.o i18n.o input.o main.o scene.o

# 通过 vpath 指令设置文件搜索
vpath %.cpp src
vpath %.h include
vpath %.inl include

# 隐式规则变量
# 设置编译器
# CC = g++
# 指定头文件位置
CXXFLAGS += -Iinclude

sudoku: $(objs)
	$(CXX) -o $@ $^

%.o:%.cpp
	$(CXX) -c $< $(CXXFLAGS)

include $(objs:%.o=%.d)

%.d: %.cpp
	@-rm $@
	$(CXX) -MM $< > $@.temp $(CXXFLAGS)
	@sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.temp >$@
	@-rm $@.temp

.PHONY: clean
clean:
	-$(RM) $(objs:%.o=%.d)
	-$(RM)  $(objs)
	$(RM) sudoku

```

### 25.2. Makefile 文件内容组成

一个Makefile文件由五种类型的内容组成：显式规则、隐式规则、变量定义、指令和注释

+ **显式规则**：显示指明何时以及如何生成或更新目录文件，显式规则包括目标、依赖和更新方法三部分
+ **隐式规则**：根据文件自动推导如何从依赖生成或更新目标文件
+ **变量定义**：定义变量并指定值，值都是字符串，类似于C语言宏定义`#define`，在使用时将值展开到引用位置（字符串替换）
+ **指令**：在 make 读取 Makefile 时做一些特别操作，包括
	1. 读取（包括）另一个 Makefile 文件（类似于C语言中的`#include`）
	2. 确定是否使用或略过Makefile文件中的部分内容（类似于C语言中的`#if`)
	3. 定义多行变量
+ **注释**：一行当中 `#` 后的内容都是注释，不会被make执行，make只有单行注释，如果需要使用`#`符号，则需要使用转义字符`\#`

Makefile 文件中，**规则书写顺序为目标的依赖顺序**。换句话说，其书写顺序，与编译顺序相反。

## 26. 嵌套 make

在大型项目中，将项目划分为多个子项目并为每个子项目单独创建 `Makefile` 是一种常见的做法。通过在顶层的 `Makefile` 中调用子项目的 `Makefile`，可以实现递归构建。这种方式便于组织和管理复杂的项目。

### 26.1. 语法

```makefile
.PHONY: all clean

all: sub1 sub2

sub1:
	$(MAKE) -C Sub1Dir

sub2:
	$(MAKE) -C Sub2Dir

clean:
	clean:
	$(MAKE) clean -C Sub1Dir
	$(MAKE) clean -C Sub2Dir
```

+ **`MAKE`** 是一个内置变量，指代当前的 `make` 命令。
+ 使用 `$(MAKE)` 确保可以递归调用 `make`，而不会丢失选项或参数。
+ **`-C Path`**：指令告诉 `make` 进入指定目录（`Path`）执行该目录中的 `Makefile`。

```shell
$(MAKE) -C Sub1Dir
```

  等价于：

```shell
cd Sub1Dir && make
```

### 26.2. export

作用：`export` 用于将变量从一个顶层`Makefile` 传递到子 `Makefile` 中，从而在子项目中使用这些变量。

语法

**传递变量**

```makefile
# 传递变量 VAR_NAME
export VAR_NAME

# 传递所有变量
```

+ 如果 `export` 后不指定变量名，则所有当前已定义的变量都会被传递给子 `Makefile`。
+ `export` 指令不仅能传递 `Makefile` 中的变量，还能将这些变量转化为 **环境变量**，从而可以被外部的命令或脚本读取。

**取消传递变量**

```makefile
# 取消变量 `VAR_NAME` 的传递
unexport VAR_NAME

# 取消所有变量的传递
unexport
```

### 26.3. 实验

#### 26.3.1. 实验条件

文件结构

```CSS
./
├── bin
├── include
│   ├── block.h
│   ├── color.h
│   ├── command.h
│   ├── common.h
│   ├── display_symbol.h
│   ├── i18n.h
│   ├── input.h
│   ├── scene.h
│   ├── system_env.hpp
│   └── utility.inl
├── lib
│   ├── block.cpp
│   ├── command.cpp
│   ├── i18n.cpp
│   ├── input.cpp
│   └── scene.cpp
└── src
    └── main.cpp
```

#### 26.3.2. 实验步骤

##### 1 `lib` 文件夹中创建 `Makefile` 文件

```Makefile
cpps := $(wildcard *.cpp)
objs := $(cpps:%.cpp=%.o)

CXXFLAGS += -I../include -fexec-charset=GBK -finput-charset=UTF-8

libsudoku.a:$(objs)
	$(AR) rcs $@ $^


.PHONY: clean
clean:
	$(RM) *.o *.a
```

##### 2 `src` 文件夹中创建 `Makefile` 文件

```makefile
cpps := $(wildcard *.cpp)
objs := $(cpps:%.cpp=%.o)

CXXFLAGS += -I../include 

libsudoku.a:$(objs)
	$(AR) rcs $@ $^


.PHONY: clean
clean:
	$(RM) *.o *.a
```

##### 3 编写总 Makefile

```Makefile
.PHONY: subsrc sublib clean

subsrc: sublib
	$(MAKE) -C src

sublib:
	$(MAKE) -C lib

clean:
	$(MAKE) clean -C src
	$(MAKE) clean -C lib
```
