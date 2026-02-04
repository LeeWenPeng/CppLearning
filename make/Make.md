## 1  make

### 1.1  介绍

make 是一个工具，主要用于**处理 C/C++ 的编译工作**。

但不只 C/C++，**所有能在终端运行的编译器/解释器编程语言**都可以处理。

除此之外，make 可以完成各种**基于依赖链的文件更新工作**。换句话说，当某些文件进行了更新工作时，同步更新所有依赖该文件的文件的工作。

## 2  Makefile

### 2.1  Makefile 文件命名的相关规则

make 工具根据 Makefile 文件中的规则对文件进行更新。

#### 2.1.1  自动查找

Make 会**自动查找** Makefile 文件，查找顺序为：**GNUmakefile -> makefile -> Makefile**

- GNUmakefile：不建议使用，只有 GNU make 会识别，其他版本 make 工具不会识别
- makefile：可以使用
- Makefile：推荐使用

> Make 会执行查找到的第一个 Makefile 文件，如果三个文件都没有找到，就会报错

#### 2.1.2  手动查找

可以手动指定文件名

```cpp
make -f filename
make --file=filename
```

> [!tip]
> 手动指定后，Make 就会使用指定文件，就不会通过自动查找 makefile 文件执行

### 2.2  Makefile 文件内容组成

一个 Makefile 文件由五种类型的内容组成：显式规则、隐式规则、变量定义、指令和注释

- **显式规则**：显示指明何时以及如何生成或更新目录文件，显式规则包括目标、依赖和更新方法三部分
- **隐式规则**：根据文件自动推导如何从依赖生成或更新目标文件
- **变量定义**：定义变量并指定值，值都是字符串，类似于 C 语言宏定义 `#define`，在使用时将值展开到引用位置（字符串替换）
- **指令**：在 make 读取 Makefile 时做一些特别操作，包括
	1. 读取（包括）另一个 Makefile 文件（类似于 C 语言中的 `#include`）
	2. 确定是否使用或略过 Makefile 文件中的部分内容（类似于 C 语言中的 `#if`)
	3. 定义多行变量
- **注释**：一行当中 `#` 后的内容都是注释，不会被 make 执行，make 只有单行注释，如果需要使用 `#` 符号，则需要使用转义字符 `\#`

Makefile 文件中，**规则书写顺序为目标的依赖顺序**。换句话说，其书写顺序，与编译顺序相反。

### 2.3  工作原理

make 的核心是依赖**关系树**和**时间戳**检查。

- 比较目标和先决条件的修改时间
- 如果先决条件更新，或目标不存在，则执行 recipe 进行更新

### 2.4  make 执行 Makefile 文件的两个阶段

GUN make 分为两个阶段来执行 Makefile 文件：

第一阶段（读取阶段）：

- 将 Makefile 文件的内容读取到 make 的内存中
- 根据 Makefile 文件构建变量
- 在程序内构建起显式规则、隐式规则
- 建立目标和依赖之间的依赖图

第二阶段（目标更新阶段）：

- 用第一阶段构建起来的数据确定需要更新的目标，执行对应的更新方法

## 3  规则

一个 Makefile 文件是由许多条规则构成，而一条规则为一个完整编译型单元，由**目标、目标的依赖与目标的更新方法**三部分组成。

### 3.1  规则语法（多行写法）

通过对依赖（prerequisites）执行方法（recipe），从而生成目标（target）

```makefile
target ...: prerequisites ...
	recipe
	...
	...
```

- target：目标，要生成的文件或执行的动作名称
- prerequisites：依赖，也就是先决条件，生成目标所需要的文件或者目标，在 `:` 后列出
- recipe：方法，一系列的 Shell 命令，以 tab 开头，用于从依赖中生成目标

> [! tip] 规则的递归执行
方法在执行之前，先检查规则中的依赖是否存在
> - 存在，直接执行方法；
> - 不存在，则向下检查，查找生成依赖文件的规则，并执行该规则以生成缺失依赖。

### 3.2  规则语法（一行写法）

```makefile
target: prerequisites; recipe; ...; ...
```

- 需要使用分号 `;`，将依赖与方法，方法与方法之间隔开

## 4  目标

一个目标就是一个文件，而一行规则的目的为生成或更新目标文件。

### 4.1  最终目标

一个 Makefile 往往同时拥有多个目标，但**最终目标**只允许有一个。

一般情况下，make 工具会将 Makefile 文件中**第一行规则中的目标**作为最终目标。

最终目标可以通过关键字 `.DEFAULT_GOAL` 手动指定：

```Makefile
.DEFAULT_GOAL = target名
```

### 4.2  目标文件的更新

make 工具会根据目标文件或依赖文件的**最终修改时间**判断是否需要执行更新目标文件的方法。

执行更新目标文件方法的条件：

- **目标文件不存在**
- **目标文件最后修改时间早于其依赖文件中任一文件的最后修改时间**

> 换句话说，当目标文件所有的依赖文件没有更新时，目标文件将不会重新编译。

最终目标对应的更新方法，以及最终目标直接或间接依赖的目标的更新方法，都将默认执行。

当一个目标与最终目标没有直接或间接的关系时，该目标所在的规则不会自动执行，需要手动指定：

```shell
make target名 # 例子： make clean
```

> [!advance] Makefile 文件编写的建议
> - 编译链接两步走
> - 将目标文件的依赖写完整

### 4.3  伪目标

**不生成和更新任何文件的目标**为伪目标

比如，Makefile 文件往往会在最后写入的 `clean` 目标。

可以通过关键字 `.PHONY` 指定伪目标：

```Makefile
.PHONY: 目标1 目标2 ...
```

> 通过将目标设置为 `.PHONY` 的依赖，可以使得该目标设置为伪目标。同时，如果目标文件存在，则该目标文件会被 make 工具忽略掉。

#### 4.3.1  伪目标的作用

根据伪目标的定义可知，伪目标必然是与最终目标没有任何的依赖关系。也就是说，伪目标所在规则中的方法必须要手动执行。

手动执行伪目标方法的代码为

```shell
make target_name
```

而为 make 工具手动指定 Makefile 文件的代码为

```shell
make Makefile_name
```

二者的形式都为

```shell
make name
```

显而易见，当伪目标名称与项目内某文件重名时，这里存在一个歧义问题。而，在 make 工具中，手动指定 Makefile 文件命令的优先级更高。也就是说，**当伪目标名称与项目内某文件重名时，将无法手动执行伪目标方法。**

因此，**需要使用 `.PHONY` 关键字手动指定伪目标。**当伪目标被指定后，使用 `make name` 代码遇到重名问题时，将会跳过该文件，执行伪目标方法。

#### 4.3.2  伪目标的新用法

通过**将某个目标指定为伪目标**的方式，可以使得在每次调用 make 时，该目标都默认重新编译。

**原因**：

目标的本质为文件，而通过将目标设置为伪目标，使得 make 工具在执行时默认跳过该文件。因此，对于 make 工具而言，该目标文件是不存在的。从而使得在每次调用 make 时，该目标都默认重新编译。

### 4.4  特殊目标

本文内提及的许多关键字的本质为特殊目标。

如 `.PHONY` 其实是一个目标，而 `.PHONY: 目标1 目标2 …` 其实是一条规则，没有方法的规则。

同样的还有下面提及的 `.SILENT`

## 5  依赖

依赖分为**普通依赖与 order-only 依赖**两种类型。

**两种依赖通过 `|` 分隔**，左边为普通依赖列表，右边为 order-only 列表。

```makefile
target : 普通依赖1 普通依赖2 ... | order-only1 order-only2 ...
```

### 5.1  普通依赖

- 如果这一依赖是由其他规则生成的文件，那么当执行到该规则时，会先执行生成这一依赖的规则。
- 如果任何一个依赖目标最终修改时间比目标晚，则重新编译生成目标文件。

### 5.2  order-only 依赖

- 当依赖文件不存在时，会执行对应方法生成。
- **依赖文件的更新不会导致目标文件的更新**。
	- 检查目标文件更新时，不会检查 order-only 依赖文件的更新

### 5.3  指定依赖路径

当 Makefile 文件与依赖文件处于不同的目录时，需要通过 `VPATH` 或 `vpath` 指定依赖文件的查找路径。

#### 5.3.1  通过变量 `VPATH` 指定搜索路径

`VPATH` 是一个全局变量，用于指定**所有文件类型**的全局搜索路径。

```makefile
VPATH = dirPath1:dirPath2:...
```

- dirPath：就是需要搜索的目录路径，可以是相对路径，也可以是绝对路径
- 目录之间用 `:` 隔开

设置 VPATH 之后，make 工具在建立依赖图时，便会从给出的路径中搜索

##### 查找顺序

- 当 `make` 需要解析某个依赖文件（如 `foo.c`）时，首先检查当前目录是否存在。
- 如果不存在，按 `VPATH` 定义的**目录顺序**依次查找（如先 `src/`，再 `lib/`）。
- 如果多个目录中存在同名文件，`make` 会使用第一个找到的文件。

#### 5.3.2  通过 `vpath` 指令指定搜索路径

`vpath` 用法比 `VPATH` 使用更加灵活，可以为不同文件类型指定不同的搜索路径

```makefile
vpath <pattern> <directories>
```

- `<pattern>`：通配符模式
- `<directories>`：目录列表，用空格或冒号分隔

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

##### 查找顺序

- `vpath` 的路径优先级高于全局变量 `VPATH`
- 搜索顺序按照 `vpath` 声明的顺序查找

## 6  方法

方法的本质为一系列 Shell 命令。

### 6.1  方法的执行工具

这些命令需要交给终端执行，所以需要符合 Shell 语法。**默认使用的 Shell 为 sh**，可以通过 `SHELL` 变量来手动指定 Shell。

### 6.2  方法的执行方式

默认的执行方式为**对每一条命令调用独立 Shell 进程执行**。可以通过添加关键字 `.ONESHELL`，使得所有的 Shell 命令调用同一个 Shell 进程中执行。

```makefile
.ONESHELL:
```

### 6.3  方法执行时的错误处理

当方法中有多条 Shell 语句时，将**顺序执行每一条 Shell 命令，直到遇到错误停止**。

通过在命令前添加 `-` 的方式，使得 make 工具**忽略 Shell 命令执行错误的情况**，以继续顺序执行之后的 shell 命令。

```makefile
target: prerequisites ...
	-Shell 命令1
	-Shell 命令2
	Shell 命令3
```

### 6.4  取消 Shell 命令回显的两种方式

> [!tip] Shell 命令的回显问题
在执行 Shell 命令时，终端通常会先会打印 Shell 命令，然后再打印 Shell 命令的执行结果。

1 在具体命令前添加 `@`

```makefile
target: prerequisites ...
	@Shell命令 操作 参数
```

2 使用关键字 `.SILENT`，将不需要打印命令的目标，设置为 `.SILENT` 的依赖

```makefile
.SILENT: target1 target2 ...
```

## 7  模式匹配

### 7.1  基本概念

#### 7.1.1  **模式规则（Pattern Rules）**

- **语法**：使用 `%` 作为通配符，匹配文件名中的任意字符串。
- **示例**：将所有的 `.c` 文件编译为 `.o` 文件

  ```makefile
  %.o: %.c
      $(CC) -c $< -o $@
  ```

  - `%.o`：匹配任意以 `.o` 结尾的目标文件。
  - `%.c`：匹配对应的 `.c` 源文件。
  - `$<` 和 `$@` 是自动变量，分别代表依赖列表中的第一个文件和目标文件名。

#### 7.1.2  **通配符（Wildcard）**

- **`*` 和 `?`**：在目标和命令中使用，匹配文件名。

  ```makefile
  # 匹配所有 .c 文件
  SRC_FILES = $(wildcard *.c)
  # 将 .c 替换为 .o
  OBJ_FILES = $(patsubst %.c,%.o,$(SRC_FILES))
  ```

### 7.2  自动变量

详情请见 变量 [9.6.1  自动变量](#9.6.1%20自动变量)

### 7.3  **静态模式规则（Static Pattern Rules）**

- **作用**：限定模式规则的适用范围，避免全局匹配。

  ```makefile
  # 仅对 $(OBJ_FILES) 列表中的文件应用模式规则
  $(OBJ_FILES): %.o: %.c
      $(CC) -c $< -o $@
  ```

### 7.4  **跨目录匹配**

- **`vpath` 指令**：指定搜索依赖文件的目录。

  ```makefile
  # 告诉 Make 在 src/ 和 lib/ 目录中查找 .c 文件
  vpath %.c src/:lib/

  %.o: %.c
      $(CC) -c $< -o $@
  ```

### 7.5  优先级规则

  - **显式规则 > 静态模式规则 > 隐式规则**。
  - 如果多个模式匹配同一目标，优先执行第一个匹配的规则。
- **示例**：

  ```makefile
  # 显式规则优先
  special.o: special.c
      $(CC) -c -O2 $< -o $@

  # 通用模式规则
  %.o: %.c
      $(CC) -c $< -o $@
  ```

## 8  多目标和多依赖

规则的目的为生成或更新目标文件。因此，

- 一条规则可以有多个目标，可以是组合多目标，也可以是独立多目标；
- 对于一个目标，也可以写多条规则

### 8.1  独立多目标

在目标位置上写入独立的多个目标，称作独立多目标写法。

这种情况下，当 make 执行到该规则时，会**对每一个目标都独立执行一次方法**。

本质是对每个目标分开写独立规则的一种简化。

#### 8.1.1  使用独立多目标对 Makefile 文件进行

```makefile
.DEFALUT_GOAL = sudoku
objs = block.o command.o i18n.o input.o main.o scene.o

sudoku: $(objs)
	g++ $^ -o $@

block.o: block.cpp block.h common.h color.h display_symbol.h
command.o: command.cpp scene.h common.h block.h command.h
i18n.o: i18n.cpp i18n.h
input.o: input.cpp common.h i18n.h utility.inl
main.o: main.cpp i18n.h input.h common.h scene.h block.h command.h system_env.hpp utility.inl
scene.o: scene.cpp scene.h common.h block.h command.h display_symbol.h i18n.h utility.inl color.h

$(objs): 
	g++ -c $(@:%.o=%.cpp)

.PHONY: clean

clean:
	-rm $(objs)
	rm sudoku

```

> [!tip] 存在问题
> 因为每个目标都具有各自不同的依赖，因此，在这条规则里依赖如何写，就成了一个问题
> 解决方法：
> - 不写依赖
> - 全部写入
> - 通过其他规则写依赖
> - **静态模式规则**可以解决一部分问题

### 8.2  组合多目标

在 **Makefile** 中，可以通过 **组合多目标** 的写法将多个目标组合成一个整体。使用 `&` 符号将多个目标组合，表示这些目标共享同一个依赖关系和构建命令，并且在构建时只执行一次命令。这种方式有效地避免了重复执行构建步骤。

```makefile
# 1
target1 & target2 & target3 & ...: dependencies
    command

# 2
targets = target1 target2 target3 ...
$(targets)&: dependencies
	command
```

#### 8.2.1  行为特点

1. **多个目标组合为一个整体**：
	- Make 会将目标 `target1`、`target2` 和 `target3` 视为一个整体。
	- **构建命令只执行一次**，但生成的文件可能需要满足所有目标。

2. **共享依赖**：
	- 所有目标共享同一组依赖。

3. **常见场景**：********
	- 当多个目标对应同一个构建命令时，使用这种写法可以避免重复执行命令。

#### 8.2.2  示例

##### **1. 多目标共享依赖**

假设你有三个目标 `file1.out`、`file2.out` 和 `file3.out`，它们依赖于 `source.txt` 并且共享相同的构建逻辑：

```makefile
file1.out & file2.out & file3.out: source.txt
    cp source.txt file1.out file2.out file3.out
```

运行 `make` 时，`cp` 命令只会执行一次，但会生成三个文件 `file1.out`、`file2.out` 和 `file3.out`。

##### **2. 多目标生成同一文件**

假设目标 `a.out` 和 `b.out` 实际上对应同一个文件：

```makefile
a.out & b.out: main.c
    gcc main.c -o a.out
    ln -sf a.out b.out
```

这里，`gcc` 只会执行一次，生成 `a.out`，然后创建一个符号链接 `b.out` 指向 `a.out`。

#### 8.2.3  注意事项

1. **命令只执行一次**： 如果组合的目标需要单独的文件或结果，但命令只生成一个文件，会导致目标无法正常满足。例如：

```makefile
a & b: main.c
gcc main.c -o a
```

- 在这种情况下，`b` 不会被真正创建，可能引发问题。

1. **不支持并行编译**： 使用组合多目标时，`make -j` 的并行编译模式可能无法正确处理这些目标，因为它们被视为一个整体。
2. **仅适用于 GNU Make**： 组合多目标写法是 GNU Make 的扩展功能，其他 Make 实现可能不支持。

### 8.3  多规则

Makefile 中对相同目标写多条规则，但 make 工具在解析 Makefile 文件时，会将对同一目标的多条规则进行合并。合并的具体规则如下：

#### 8.3.1  依赖合并

- **所有规则中的依赖会被合并**，即将不同规则中列出的依赖项统一收集。
- **没有重复依赖**：如果某个依赖项在多条规则中重复出现，Make 会自动去重。

#### 8.3.2  方法合并

- **只保留最后一条规则中的构建命令**。
- 前面规则中定义的构建命令会被忽略。

#### 8.3.3  注意事项

1. **显式规则优先级高**： 如果存在显式规则和隐式规则的冲突，Make 会优先使用显式规则。
2. **避免混乱**： 多条规则合并可能会导致维护和调试困难，建议尽量将目标规则集中写在一起。
3. **合理分离依赖和方法**： 如果目标的依赖和命令较多，可以用变量和模式规则来管理，避免手动合并的复杂性。

### 8.4  静态模式规则

用于在一组特定目标上应用同一个构建模式。这种规则允许你明确地列出目标，并为它们定义共同的依赖和构建指令。

> 本质：通过使用 `%` 进行文件模式匹配来推导出对应的依赖。

语法：

```makefile
targets : target-pattern: prereq-patterns
	recipe
	...
```

- targets：列出应用规则的目标
- target-pattern：定义目标的通配模式
- prereq-patterns：定义依赖项的通配模式
- recipe：更新方法

#### 8.4.1  静态模式规则与通用模式规则的区别

- **静态模式规则**只对明确列出的目标起作用。
- **通用模式规则**（如 `%.o: %.c`）会自动匹配所有符合模式的目标。

#### 8.4.2  静态模式规则的应用场景

1. 编译特定的对象文件
2. **处理多个目标的不同依赖**

#### 8.4.3  示例

通过静态模式，对 objs 变量中所有的 `.o` 文件，设置 `.cpp` 和 `.h` 依赖，以及方法。

```makefile
$(objs): %.o : %.cpp %.h
	g++ -c $<
```

1. 对每一个对象独立进行下面的两步模式匹配
2. 在 `objs` 中模式匹配 `%.o`
3. 然后，生成字符串 `%.cpp` 和 `%.h`

#### 8.4.4  通过静态模式对 Makefile 进行修改

```makefile
.DEFALUT_GOAL = sudoku
objs = block.o command.o i18n.o input.o  scene.o

sudoku: $(objs) main.o
	g++ -o $@ $^

block.o:  common.h color.h display_symbol.h
command.o: scene.h common.h block.h 
input.o:  common.h i18n.h utility.inl
scene.o: common.h block.h command.h display_symbol.h i18n.h utility.inl color.h

$(objs): %.o : %.cpp %.h
	g++ -c $<

main.o: main.cpp i18n.h input.h common.h scene.h block.h command.h system_env.hpp utility.inl
	g++ -c $<

.PHONY: clean

clean:
	-rm $(objs)
	rm sudoku

```

> 虽然我还是觉得通过多规则写依赖更靠谱

## 9  变量

Makefile 变量类似于 C 语言中的宏定义，对变量引用的本质为**字符串替换**。也就是，在执行变量所在语句时，会将变量替换成变量对应值的字符串。与 C 语言中宏定义不同的是，Makefile 变量允许重新赋值。

### 9.1  变量名的命名规则

- 区分大小写
- 不能含有 `:`、`#`、`=`

### 9.2  变量定义

#### 9.2.1  单行变量的定义

```makefile
# 1
变量名1 = 变量值1 变量名2 = 变量值3

# 2
变量名1 := 变量值1 变量名2 := 变量值3

# 3
变量名1 ::= 变量值1 变量名2 ::= 变量值3
```

- **变量值的数据类型只有字符串类型**

#### 9.2.2  多行变量的定义

使用关键字 `define`，可以定义多行变量

```makefile
# 1 默认的赋值模式为 =，可以省略不写
define 变量名 # 默认为 =
值1
值2
...
endef

# 2 但仍可以使用单行变量定义中的其他赋值符号
define 变量名 :=
值1
值2
...
endef

define 变量名 +=
值1
值2
...
endef
```

#### 9.2.3  取消变量定义

```makefile
undefine 变量名
```

#### 9.2.4  变量覆盖

Makefile 文件中定义的变量，可以在执行 make 命令时进行覆盖

```shell
make 变量名 = 变量值
```

通过关键字 `override`，可以限制这种行为：

```makefile
override 变量名 = 值
```

### 9.3  变量的引用方式

使用 `$` 符号引用变量名

```makefile
# 1
$(变量名)

# 2
${变量名}
```

#### 9.3.1  打印 `$` 字符

`$` 在 Makefile 文件中是特殊字符，是引用变量的符号，不能直接打印。如果想要在变量值中使用 `$`，就使用 `$$`。

换句话说，`$$` 是 `$` 的转义字符

```makefile
echo $(info $(objs))
```

#### 9.3.2  系统变量可以直接引用

系统中的环境变量可以在 Makefile 中直接使用

```makefile
all:
	$(info $(HOME))
```

#### 9.3.3  变量替换引用

对 var2 中的值进行字符串替换操作后，再赋值给 var 变量

```makefile
var = $(var2:%字符1=%字符2)
```

```makefile
# 例子
files != ls *.cpp
objs = $(files:%.cpp=%.o)
all:
	$(info $(objs)) 
# 打印结果为：block.o command.o i18n.o input.o main.o scene.o
```

> 只能替换结尾部分

### 9.4  变量的展开时机

#### 9.4.1  变量和函数的展开

make 通过两种方式来展开变量：**立即展开（Immediate Expansion）**和**延迟展开（Lazy Expansion）**zy Expansion）**。

##### （1）立即展开

在 Makefile 的读取阶段，Make 会立即对 `:=` 定义的变量进行展开，也就是说，在赋值时，变量的值会立即计算并赋给该变量。这意味着，变量的值会在 Makefile 解析时计算好，而不会等到实际使用时才计算。

例子：

```makefile
    VAR := $(shell echo Hello)
```

在这个例子中，`VAR` 的值会在 Makefile 被解析时就立刻被计算，`$(shell echo Hello)` 会在读取 Makefile 时执行，而不是在执行规则时。

##### （2）延迟展开

在延迟展开中，变量在被引用时才会展开，而不是在赋值时。这通常通过 `=` 来定义变量。

例子：

```makefile
    VAR = $(shell echo Hello)
```

在这个例子中，`VAR` 的值不会在 Makefile 解析时立即计算，而是直到实际使用 `VAR` 时才会执行 `$(shell echo Hello)`。

#### 9.4.2  直接展开和延迟展开的区别

1. 直接展开

```makefile
file := $(objs)

all:
	$(info $(file))

objs = block.o command.o i18n.o input.o main.o scene.o
```

结果：打印空

1. 延迟展开

```makefile
file = $(objs)

all:
	$(info $(file))

objs = block.o command.o i18n.o input.o main.o scene.o
```

结果：打印 block.o command.o i18n.o input.o main.o scene.o

 1. 分析

- 直接展开是在读取阶段时进行的展开。也就是，在 make 工具读取到变量时，就直接对其变量值进行计算，并将其进行展开。例子中，在 file 变量定义时，就将其展开为 obsj，而这时，obsj 还未定义，也就是一个空串。因此，打印为空。
- 延迟展开是在变量使用时进行的展开。也就是，引用变量时，再将变量进行展开。例子中，在 file 变量被引用时，也就是 `$(info $(file))`，首先将其展开成 `$(objs)`，然后再将 `objs` 展开。最后，成功打印。

#### 9.4.3  二次展开

通过关键字 `.SECONDEXPANSION` 可以将变量设置为延迟展开

```makefile
.SECONDEXPANSION:
target: $$(var)
```

- 规则前需要添加 `.SECONDEXPANSION` 伪目标
- 变量引用部分需要使用 `$$`

> [!question] 如果使用了 `$$`，但没有在此之前添加 `.SECONDEXPANSION` 伪目标，会如何？
> 结果：终端处报错，错误原因为不存在该 var
> 原因：`$$` 将转义成 `$`，并将 `$(var)` 发送给 sh，但 sh 无法识别该变量

> [!question] 如果使用了 `.SECONDEXPANSION`，但没有使用 `$$`，会如何？
> 结果：不会将变量设置为延迟展开
> 原因：相当于正常引用了变量

### 9.5  变量的赋值

#### 9.5.1  简单赋值（直接展开）

```makefile
# 1
变量名 := 变量值

# 2
变量名 ::= 变量值
```

#### 9.5.2  递归展开赋值（延迟展开）

```makefile
变量名 = 变量值
```

#### 9.5.3  条件赋值

在对变量赋值前进行一个判断，如果变量已经定义过，则不再赋值。如果变量没有定义，那么就定义该变量并直接赋值。

```makefile
变量名 ?= 变量值
```

例子：

```makefile
var := 10

var ?= 100

all: 
	@echo $(var) # 打印结果为 10
```

#### 9.5.4  追加赋值

在原有值后，追加新值。

```makefile
变量名 += 变量值
```

#### 9.5.5  Shell 命令赋值

执行 Shell 命令，并将返回的值赋值给变量

```makefile
变量名 != 有返回值的Shell 命令
```

### 9.6  特殊变量

#### 9.6.1  自动变量

|  代码   | 解释                                                                  |
| :---: | ------------------------------------------------------------------- |
| `$@`  | + 本条规则的目标名；<br>+ 如果目标为归档文件的成员，则为归档文件名；<br>+ 多目标的模式规则中，为导致本条规则执行的目标名 |
| `$<`  | 本条规则中，第一个依赖                                                         |
| `$?`  | 所有比目标更新的先决条件，以空格隔开                                                  |
| `$^`  | 所有简单依赖文件名，文件名不会重复且不包括 order-only 依赖                                 |
| `$+`  | 所有简单依赖文件名，包括重复的文件名，但不包括 order-only 依赖                               |
| `$\|` | 所有 order-only 依赖名                                                   |
| `$*`  | 目标文件名的主干部分，也就是不包括后缀名                                                |
| `$%`  | + 如果目标不是归档文件，则为空<br>+ 如果目标是归档文件成员，则为对应的成员文件                         |
| `$@D` | 对应变量所在的目录路径，但是不包括最后的 `/`                                            |
| `$@F` | 对应变量去除目录部分的文件名                                                      |

> 有改动的意思就是其修改时间晚于目标文件的修改时间

#### 9.6.2  预定义变量

|    常用变量    |      作用      |    默认值     |                     补充                     |
| :--------: | :----------: | :--------: | :----------------------------------------: |
|    `CC`    |   C 语言编译器程序   |    `cc`    |                                            |
|   `CXX`    |   C++ 编译程序   |   `g++`    |                                            |
|    `AR`    |     归档程序     |    `ar`    |                                            |
|   `CPP`    |  C 语言预处理程序   | `$(CC) -E` |                                            |
|    `RM`    |   删除文件的程序    |  `rm -f`   |                                            |
|  `CFLAGS`  |  传递给 C 编译器的选项  |     无      |               `-O Iinclude`                |
| `CXXFLAGS` | 传递给 C++ 编译器的选项 |     无      |  + `-std=c++11`<br>+ `-fexec-charset=GBK`  |
| `CPPFLAGS` |   C 预处理的选项    |     无      |                                            |
| `LDFLAGS`  |     链接选项     |            |                    `-L`                    |
|  `LDLIBS`  |   链接需要用到的库   |            | + `Ikernel32`<br>+ `Iuser32`<br>+ `Igdi32` |

> [!note]
> 在 Makefile 里，**和**，而 **`CXX` 代表 C++**

### 9.7  绑定目标的变量 - 变量作用域

Makefile 中的变量一般是**`CPP` 代表 C 语言预处理程序**，也就是说，**全局变量**。

在变量定义时，在变量名前添加 `目标:` 的方式，**在变量定义后，可以在 Makefile 文件中任意位置对变量进行引用**。

```makefile
目标名: 变量名 = 值
```

#### 9.7.1  一个目标一次只能绑定一个变量

```makefile
目标名: 变量名1 = 值 变量名2 = 值
```

上述代码中，`变量名1` 为变量名，而第一个等号后的所有字符，都将作为值存在，也就是，`值 变量名2 = 值`。

#### 9.7.2  一个变量一次可以绑定多个目标

使用 `%` 的模式匹配串，**绑定目标和变量，限制该变量的使用范围在该目标内**

```makefile
%字符串: 变量名 = 值
```

例子

```makefile
global_var = This is a global var!

.PHONY = all

all: first.c second.o third.o

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

- global_var 全局变量
- target_var 绑定到 first.c
- target2_var 绑定到所有的 .o 文件，也就是 second.o 和 third.o

## 10  函数

### 10.1  字符串处理函数

#### 10.1.1  subst

功能：文本中字符串的替换，返回替换后的文本

语法：

```makefile
$(subst target,replacement,text)
```

- 使用 replacement 替换掉 text 中的 target
- target 需要替换掉的源内容
- replacement 替换后的目标内容
- text 代处理的内容，可以是任意字符串

> [!important] 逗号后不要有空格，除非需要
> 参数列表中，空格将会被看作字符处理，包括逗号后的空格。

例子

```makefile
objs = main.o hello.o
srcs = $(subst .o,.cpp,$(objs))
headers = $(subst .cpp,.h,$(srcs))

all:
	$(info $(srcs))
	$(info $(headers))
```

> [!note] 与变量替换引用的比较
> - 变量替换引用只能替换结尾的字符
> - subst 更加灵活，可以替换任意字符

#### 10.1.2  patsubst

功能：模式替换，返回被替换后的文本

语法：

```makefile
$(patsubst pattern,replacement,text)
```

- pattern 需要被替换的模式
- replacement 替换后的目标模式
- text 代处理的内容，各项内容需要用空格隔开

例子

```makefile
objs = main.o hello.o
srcs = $(patsubst %.o,%.cpp,$(objs))
headers = $(patsubst %.cpp,%.h,$(srcs))

all:
	$(info $(srcs))
	$(info $(headers))
```

> [!note] 与 subst 的比较
> - subst 的匹配是以**可以将变量与所有匹配到的目标进行绑定**为基础的，会将所有匹配到的源文本都替换成目标文本
> - patsubst 的模式匹配是以空格分割的**单个字符**为基础的，将匹配到的单词替换为目标文本
> - 也就是说，有一个字符串中间有一段子串与需要替换的文本相同，subst 会进行替换，而 patsubst 不会进行替换
> - 因此，在使用 patsubst 时，需要将每一项用空格隔开

```makefile
srcs = a.cpp b.cppc.cpp
headers = $(subst .cpp,.h,$(srcs))
headers2 = $(patsubst %.cpp,%.h,$(srcs))

all:
	$(info $(headers))# a.h b.hc.h
	$(info $(headers2))# a.h b.cppc.h
```

- 在 subst 中，由于是文本替换，能够成功将 .cpp 替换成 .h。
- 而在 patsubst 中，由于 b.cpp **单词** c.cpp 中间没有用空格分开，因此，`b.cppc.cpp` 被看作为一个单词。

#### 10.1.3  strip

功能：去除字符串中的空格，并返回去除空格后的文本。具体如下：

- 去除字符串头部和尾部的空格
- 去除字符串中间连续的多个空格，用一个空格替换

语法：

```makefile
$(strip string)
```

例子

```makefile
files = aa hello.cpp              main.cpp      test.cpp     
files := $(subst aa,        ,$(files))
files2 = $(strip $(files))

all:
	$(info $(files)) #          hello.cpp              main.cpp      test.cpp     
	$(info $(files2)) # hello.cpp main.cpp test.cpp
```

#### 10.1.4  findstring

功能：查找字符串，找到后返回对应字符串，否则返回空

语法：

```makefile
$(findstring find,string)
```

- find 需要查找的字符串
- string 查找域

例子

```makefile
files = hello.cpp main.cpp test.cpp
find = $(findstring hel,$(files))
find2 = $(findstring HEL,$(files))

all:
	$(info $(find)) # hel
	$(info $(find2)) # 空
```

> 区分大小写

#### 10.1.5  filter、filter-out

功能：从文本中筛选出符合模式的内容并返回

语法：

```makefile
$(filter pattern...,text)
```

- pattern 模式，可以有多个，用空格隔开
- text 查找域

例子

```makefile
files = hello.cpp main.cpp test.cpp main.o test.o hello.h
files2 = $(filter %.o %.h,$(files))
all:
	$(info $(files2))
```

> filter-out 的语法基本和 filter 一致，功能相反，从文本中过滤掉符合模式的内容，并将其余内容返回

#### 10.1.6  sort

功能：将文本中的单词按字典顺序排序，并且移除重复项

语法：

```makefile
$(sort list)
```

- list 需要排序的内容

sort 以**和**为基础单位，因此，需要排序的内容中，每一项之间需要**单词**

#### 10.1.7  word

功能：用于返回文本中第 N 个单词

语法：

```makefile
$(word N,text)
```

- N，单词的位置，从 1 开始
	- 如果小于 1，则报错
	- 如果大于总单词数，则返回空串
- text 搜索域

#### 10.1.8  wordlist

功能：用于返回文本中指定范围内的所有单词

语法：

```makefile
$(wordlist beg,end,text)
```

- beg 起始位置
- end 结束位置
- text 搜索域
- 返回 $[\text{beg}, \text{end}]$ 范围内的所有单词
	- beg 和 end 从 1 开始
	- 如果 beg 大于 end，则返回空串
	- 如果 beg 大于单词总数，则返回空串
	- 如果 end 大于单词总数，则返回 $[\text{start}, \text{单词总数}]$ 返回内的所有单词

#### 10.1.9  words

功能：返回文本中的单词数

语法：

```makefile
$(words text)
```

#### 10.1.10  firstword、lastword

功能：返回文本中第一个\最后一个单词

语法：

```makefile
$(firstword text)

$(lastword text)
```

#### 10.1.11  foreach

功能：对文本中每一个单词进行处理，并返回处理后的文本

语法：

```makefile
$(foreach each,list,process)
```

- each 文本中的每一项
- list 文本，也就是单词列表
- process 需要对 each 进行的操作

例子

```makefile
objs = block.o command.o i18n.o input.o main.o scene.o
objs_new = $(foreach each,$(objs),$(addprefix hello,$(suffix $(each))))

.PHONY: all
all:
	$(info $(objs_new))
```

#### 10.1.12  let

功能：将一个文本中的单词逐项分配给指定的变量。**用空格隔开**

```makefile
$(let vars,[list],proc)
```

- vars：变量名，可以有多个，以空格隔开
- list：待处理字符串，单词列表
- proc：可以在这里对变量进行一些额外操作，操作的结果为 let 函数的返回值

##### 变量与列表项的对应关系

- 如果变量个数 **GUN make 4.4 以上支持** 列表项数：每个变量依次分配一个列表项。
- 如果变量个数 **等于** 列表项数：多余的变量会被赋值为空（或默认值，例如空字符串）。
- 如果变量个数 **多于** 列表项数：
	- 前 `n-1` 个变量分别获得一个列表项。
	- 最后一个变量获得列表中剩余的所有项（以列表形式）。

例子

```makefile
files = main.cpp src/block.cpp 
f2 = $(let v1 v2 v3,$(files),$(v3))

all:
	$(info $(f2))
```

### 10.2  文件名处理函数

#### 10.2.1  dir

用于返回文件的目录路径

```makefile
$(dir files)
```

- files 需要返回目录的文件名，可以有多个，空格隔开

并不是通过扫描，然后返回文件的真实地址，而是根据 files 中给出的文件的路径进行返回

```makefile
files =  block.h command.h i18n.h hus/input.h hhh/main.h hhh/scene.h
VPATH = src:include
files2 = $(files:%.o=%.h)
result = $(dir $(files2))
all:
	$(info $(files2))
	$(info $(result)) # ./ ./ ./ hus/ hhh/ hhh/
```

比如上述代码中的目录根本就不存在，但是仍然成功打印出来

所以，其本质是**少于**

#### 10.2.2  notdir

返回除目录外的文件名

```makefile
$(notdir files)
```

- files 代操作文件，可以有多个，空格隔开

> dir 和 notdir 刚好组成原本的 files 内的文件：
> - dir 只保留 files 中的目录部分的字符串
> - not dir 只保留 files 中文件名部分的字符串

#### 10.2.3  suffix

返回文件后缀名，如果没有后缀名返回空

```makefile
$(suffix files)
```

#### 10.2.4  basename

返回文件名除后缀名外的部分

```makefile
$(basename files)
```

- 只去除后缀名，如果 files 中文件名前有目录的话，仍然会保留

> basename 和 suffix 刚好组成原本的 files 内的文件：
> - suffix 只保留 files 中的后缀部分
> - basename 保留 files 中文件名除后缀部分外的其余部分

#### 10.2.5  addsuffix

给文件添加后缀

```makefile
$(addsuffix, sfx,files)
```

- sfx 要添加的后缀
- files

在 files 中每一个单词后追加一串字符串，相当于：

```cpp
for(string file : files){
	file += sfx;
}
```

#### 10.2.6  addprefix

给文件添加前缀

```makefile
$(addprefix, pfx,files)
```

仍然是简单直接的字符串追加，相当于：

```cpp
for(string file : files){
	file = pfx + file;
}
```

#### 10.2.7  join

将两个列表中的单词一对一连接，当两个列表长度不同时，则多出来的部分原样返回

```makefile
$(join files1, files2)
```

### 10.3  文件检索函数

#### 10.3.1  wildcard

返回符合搜索条件的文件列表

```makefile
$(wildcard condition)
```

- wildcard 是一个搜索操作，返回的是真实的文件，目录也被视为文件

> [!note] 通配符
> 文件操作时可以使用通配符，比如 `*`

例子

```makefile
files = $(wildcard *.cpp src/*.cpp include/*.h)
files2 =  $(wildcard *.cpp)
files3 =  $(wildcard *)
all:
	$(info $(files))
	$(info $(files2))
	$(info $(files3))
```

#### 10.3.2  realpath

返回文件的绝对路径。如果文件不存在，返回空。

```makefile
$(realpath files)
```

#### 10.3.3  abspath

返回文件的绝对路径。如果文件不存在，以当前路径作为文件路径，并返回。

```makefile
$(abspath files)
```

### 10.4  条件函数

#### 10.4.1  if

条件判断，如果条件展开不是空串，则返回真的部分，否则返回假的部分

```makefile
$(if condition,then-part,else-part)
```

- condition 条件部分
- then-part 条件非空时，返回 then-part
- else-part 条件为空时，返回 else-part
	- 可省略为空

#### 10.4.2  or

返回条件中第一个不为空的部分

```makefile
$(or condition1[,condition2[,condition3[...]]])
```

#### 10.4.3  and

如果条件按中有一个空串，则返回空，否则返回最后一个条件

```makefile
$(and condition1[,condition2[,condition3[...]]])
```

#### 10.4.4  intcmp

功能：比较两个整数的大小，并返回操作结果 (**针对于文件名的字符串操作**)

语法：

```makefile
$(intcmp lhs,rhs[,lt-part[,eq-part[,gt-part]]])
```

- lhs 第一个数
- rhs 第二个数
- lt-part：
	- 当 lhs < rhs，返回 lt-part
	- 否则返回空
- eq-part：
	- 当 lhs < rhs，返回 lt-part
	- 当 lhs == rhs，返回 eq-part
	- 否则返回空
- gt-part：
	- 当 lhs < rhs，返回 lt-part
	- 当 lhs == rhs，返回 eq-part
	- lhs > rhs 返回 gt-part
	- 否则返回空
- 如果仅有两个参数，则 lhs == rhs 时，返回数值，否则返回空

```makefile
all:
	$(info $(intcmp 1,1))
	$(info $(intcmp 1,2,less))
	$(info $(intcmp 2,2,less,equal))
	$(info $(intcmp 3,2,less,equal,greater))
```

### 10.5  变量信息检索函数

#### 10.5.1  value 函数

功能：

- 对于非立即展开的变量，可以查看变量的定义；
- 对于立即展开的变量，直接返回变量值。

语法：

```makefile
$(value variable)
```

例子

```makefile
func = $(dir $(realpath $(1))) $(dir $(realpath $(2)))
res = $(call func,main.cpp,m.txt)
res2 := $(call func,main.cpp,m.txt)

.PHONY: all
all:
	$(info $(value res))
	$(info $(value res2))
```

#### 10.5.2  origin 函数

功能：查看变量的定义来源

语法：

```makefile
$(origin variable)
```

- variable 变量名，不是变量的引用，是变量的

返回值：

| 变量类型  | 返回值         |
| ----- | ----------- |
| 未定义变量 | undefined   |
| 默认变量  | default     |
| 环境变量  | environment |
| 自定义变量 | file        |
| 自动变量  | automatic   |

例子

```makefile
res = $(origin abc) # undefined
res2 = $(origin CC) # default
res3 = $(origin HOME) # environment
res4 = $(origin res3) # file
res5 = $(origin <) # automatic

all:
	$(info $(res))
	$(info $(res2))
	$(info $(res3))
	$(info $(res4))
	$(info $(res5))
```

#### 10.5.3  flavor

功能：查看变量的赋值方式

语法：

```makefile
flavor variable
```

返回值：

| 变量     | 返回值       |
| ------ | --------- |
| 未定义    | undefined |
| 不是立即展开 | recursive |
| 立即展开   | simple    |

### 10.6  信息提示函数

#### 10.6.1  error

提示错误信息，终止 make 运行

```makefile
$(error text)
```

- text 提示信息

---

#### 10.6.2  warning

提示警告信息，但 make 不会停止运行

```makefile
$(warning text)
```

- text 提示信息

---

#### 10.6.3  info

输出提供的信息

```makefile
$(info text)
```

- text 提示信息

### 10.7  其他重要函数

#### 10.7.1  file

功能：对文件进行读写操作的函数

语法：

```makefile
$(file op filename[,text])
```

- op 操作
	- `>` 覆盖
	- `>>` 追加
	- `<` 读
- filename 需要操作的文件
- text 写入的文本内容
	- 当 op 为 `<` 时，不需要该参数

> 如果文件不存在，则会新建文件

例子

```makefile
text1 = This is first!
text2 = Something Write to File heihei
$(file > m.txt,$(text1))
$(file >> m.txt,$(text2))

res = $(file < m.txt)

.PHONY: all
all:
	$(info $(res))
```

#### 10.7.2  eval

功能：将文本生成 Makefile 的内容

- 设置 eval 文本变量
- 使用 eval 函数访问变量
- 相当于将 eval 函数直接替换成了 eval 文本变量，并将文本直接作为 Makefile 代码执行
语法：

```makefile
define eval_target = 
...代码
endef

$(eval $(eval_target))
```

#### 10.7.3  shell

功能：用于执行 shell 指令

语法：

```makefile
$(shell shell语句)
```

类似于

```makefile
shell_res != shell语句
$(shell_res)
```

### 10.8  自定义函数

将复杂表达式写成一个变量，类似于编程语言里的自定义函数

```makefile
function_name = function($(1), $(2),...,$(n))
```

- 等号左侧为函数名，call 通过函数名调用函数
- 等号右侧为函数体，其运行结果是函数的返回值
- 函数中可以用 $(n) 来传递第 n 个参数

#### 10.8.1  call

功能： 调用函数，包括自定义函数

语法：

```makefile
$(call funcname,args...)
```

- funcname 函数，可以是自定义函数
- args 参数，至少有一个，以逗号隔开

例子

```makefile
func = $(dir $(realpath $(1))) $(dir $(realpath $(2)))
res = $(call func,main.cpp,m.txt)

all:
	$(info $(res))
```

## 11  隐式规则

### 11.1  什么是隐式规则？

隐式规则是 **GUN make 4.4 以上版本支持** 的内置规则，用于处理常见的文件生成过程（例如从源文件生成目标文件）。当 Makefile 中没有明确指定某个目标的生成规则时，**make** 会尝试应用这些隐式规则。

### 11.2  隐式规则的工作机制

当 **make** 发现某个目标没有明确的规则时，它会：

1. 检查是否有适用于该目标的隐式规则。
2. 如果找到合适的隐式规则，则应用该规则。
3. 使用内置变量（如 `$(CC)`、`$(CFLAGS)` 等）来执行规则中的命令。

### 11.3  隐式规则的常见场景

隐式规则的主要作用是**make**，例如：

1. **针对某些常用模式的文件生成过程**：

   ```makefile
   %.o: %.c
       $(CC) $(CFLAGS) -c $< -o $@
   ```

   这是一个典型的隐式规则，表示所有 `.c` 文件可以通过相同的方式生成对应的 `.o` 文件。

   隐式规则是没有明写在 Makefile 文件中的。

2. **从 `.c` 文件编译生成 `.o` 文件**

   ```makefile
   %.o: %.cpp
       $(CXX) $(CPPFLAGS) $(CXXFLAGS) -c
   ```

> [!note]
> - `.cpp` 和 `.o` 编译为 `.cc`，也是相同更新方法

1. **从 `.C` 文件生成 `.o` 文件**

   ```makefile
   %: %.o
	   $(CC) $(LDFLAGS) $^ $(LOADLIBES) $(LDLIBS) -o $@
   ```

> [!note]
> - 默认的链接使用的工具为 `.o`，是针对 C 语言的链接
> - 进行 C++ 语言的链接时，需要指定编译工具为 `CC`。
> ```makefile
> # 1 通过修改CC变量值指定编译工具
> CC=g++
> # 2 手动指定方法，使用 CXX 变量
> %.o: %.cpp
> 	$(CXX) -c $<
> ```

1. **`g++` 文件链接为可执行文件**（`.y` 工具）：

   ```makefile
   %.c: %.y
       $(YACC) $(YFLAGS) $< -o $@
   ```

2. **从 `.c` 文件生成 `yacc` 文件**（`.tex` 工具）：

   ```makefile
   %.dvi: %.tex
       $(TEX) $< -o $@
   ```

---

#### 11.3.1  链接库文件所需的变量设置

实验条件：

- 将 .cpp 文件打包成 src/libxxx.a

实验目的：

不再通过.o 文件进行链接将程序打包成 .a 文件后进行链接

```makefile
# 1 指定库依赖的寻找路径
vpath %.a PATH

# 2 指定库文件所在位置
LDFLAGS += -LPATH

# 3 指定库文件名称
LDLIBS += -l

# 4 指定依赖
main: libxxx.a ...
```

- 库文件名为，`.dvi`
	- `TeX` 为前缀
	- `libxxx.a` 为后缀
	- `lib` 为库文件简名
- 库文件需要
	- 指定依赖
		- 在依赖中写库文件全名 `.a`
	- 指定依赖路径
		- `xxx`
	- 指定库文件所在位置
		- `libxxx.a`
		- 和上述依赖路径为一个 `vpath %.a PATH`
	- 指定库文件名称：
		- 使用关键字 `LDFLAGS += -LPATH` 指定库名字时，使用库文件简名，也就是需要去除前缀和后缀
		- `PATH`

### 11.4  打印变量默认值

```makefile
make -p | grep LDFLAGS
```

### 11.5  如何查看 make 的隐式规则？

可以使用以下命令查看系统中内置的隐式规则：

```bash
make -p
```

这会列出 **从 `LDLIBS` 文件生成 `LDLIBS += -lxxx` 文件** 所有的默认规则和变量。

### 11.6  禁用隐式规则

如果希望完全禁用隐式规则（例如为了调试或简化构建过程），可以使用以下选项：

```bash
make --no-builtin-rules
```

隐式规则只有在没有显式规则时，才会执行。而且，当相同目标的规则有多条时，会将依赖合并，而更新方法只保留最后一个有效更新方法。因此，**make**

```makefile
%.o: %.c
```

### 11.7  示例

以下是一个简化的 Makefile，利用隐式规则自动处理 `.o` 文件的生成：

#### 11.7.1  Makefile

```makefile
CC = gcc
CFLAGS = -Wall -g

# 显式规则，只写出最终目标
app: main.o utils.o
    $(CC) $^ -o $@

# 隐式规则：不需要显式写如何生成 .o 文件，make 会自动处理
```

执行 `make app` 时，隐式规则会自动处理 `.c` 文件到 `.o` 文件的编译过程。无需手动写出每个 `.o` 文件的生成规则。

## 12  引入其他 Makefile 文件

**禁用某一条隐式规则的方法，写入一条具有相同对象和依赖的显式规则。**，效果如同**使用 `include` 指令可以读入其他 Makefile 文件的内容**

```makefile
-include mk1 mk2 ...

# 允许使用通配符
-include mk*
```

- 引入文件可以有多个，以空格隔开
- 如果查找不到，就会报错，停止 make 运行。因此，在命令前添加 `include`，在报错时跳过 [[#6 关于方法的一些问题#错误处理]]

> [!note]
> - 使用 `-` 指令，相当于，在使用该指令的位置处，插入对应的 Makefile 文件代码
> - 类似于 C++ 中的 `include` 关键字

### 12.1  使用 include 和多规则更新 Makefile

不再直接写入 cpp 文件需要的多条依赖。

- 借助 shell 语句自动获取依赖，存入到 `include` 文件中；
- 采用 `.d` 插入依赖

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

### 12.2  Makefile 文件内容组成

一个 Makefile 文件由五种类型的内容组成：显式规则、隐式规则、变量定义、指令和注释

- **在 `include` 位置用对应的文件内容替换**：显示指明何时以及如何生成或更新目录文件，显式规则包括目标、依赖和更新方法三部分
- **显式规则**：根据文件自动推导如何从依赖生成或更新目标文件
- **隐式规则**：定义变量并指定值，值都是字符串，类似于 C 语言宏定义 `#define`，在使用时将值展开到引用位置（字符串替换）
- **变量定义**：在 make 读取 Makefile 时做一些特别操作，包括
	1. 读取（包括）另一个 Makefile 文件（类似于 C 语言中的 `#include`）
	2. 确定是否使用或略过 Makefile 文件中的部分内容（类似于 C 语言中的 `#if`)
	3. 定义多行变量
- **指令**：一行当中 `#` 后的内容都是注释，不会被 make 执行，make 只有单行注释，如果需要使用 `#` 符号，则需要使用转义字符 `\#`

Makefile 文件中，**注释**。换句话说，其书写顺序，与编译顺序相反。

## 13  嵌套 make

在大型项目中，将项目划分为多个子项目并为每个子项目单独创建 `Makefile` 是一种常见的做法。通过在顶层的 `Makefile` 中调用子项目的 `Makefile`，可以实现递归构建。这种方式便于组织和管理复杂的项目。

### 13.1  基本语法

```makefile
sub_target:
	$(MAKE) -C <子目录路径> <目标> <key=value>
```

- **规则书写顺序为目标的依赖顺序** 是一个内置变量，指代当前的 `MAKE` 命令。
- **`make`**：切换到子目录执行 Makefile。
- **`-C <dir>`**：指定子目录 Makefile 的目标（如 `<目标>`）。非必填。

```shell
$(MAKE) -C Sub1Dir
```

  等价于：

```shell
cd Sub1Dir && make
```

### 13.2  参数传递

#### 13.2.1  命令行传递

```makefile
sub_target:
	$(MAKE) -C <子目录路径> <key=value>
```

 `clean`：向子 Makefile 传递参数。子 Makefile 中可以通过 `<key=value>` 读取参数。非必填。等价于 `$(key)`。

#### 13.2.2  export

作用：`export` 用于将变量从一个顶层 `export` 传递到子 `Makefile` 中，从而在子项目中使用这些变量。

**`Makefile`**

```makefile
# 传递变量 VAR_NAME
export VAR_NAME

# 传递所有变量
```

- 如果 `export` 后不指定变量名，则所有当前已定义的变量都会被传递给子 `Makefile`。
- `export` 指令不仅能传递 `Makefile` 中的变量，还能将这些变量转化为 **传递变量**，从而可以被外部的命令或脚本读取。

**环境变量**

```makefile
# 取消变量 `VAR_NAME` 的传递
unexport VAR_NAME

# 取消所有变量的传递
unexport
```

### 13.3  处理路径问题

(1) 获取顶层绝对路径

在顶层 Makefile 中定义：

```makefile
TOP_DIR := $(shell pwd)
export TOP_DIR
```

子 Makefile 可通过 `$(TOP_DIR)` 引用顶层目录。

(2) 子目录中的路径处理

在子 Makefile 中，所有路径均为 **取消传递变量** 的路径。例如：

```makefile
INCLUDE_DIR = $(TOP_DIR)/include  # 正确引用顶层目录
```

### 13.4  并行构建支持

在顶层使用 `make -jN` 时，子 `make` 默认继承并行参数。如需显式控制：

```makefile
all:
	$(MAKE) -C lib -j4
	$(MAKE) -C src -j4
```

### 13.5  错误处理与调试

(1) 忽略子目录错误

```makefile
optional_build:
	-$(MAKE) -C experimental  # 前缀 "-" 忽略错误
```

(2) 调试输出

```makefile
debug_build:
	$(MAKE) -C src --debug=v
```

### 13.6  替代方案：自动遍历子目录

若子目录数量多且结构统一，可使用循环：

```makefile
SUBDIRS = lib src test

all: $(SUBDIRS)

$(SUBDIRS):
	$(MAKE) -C $@

clean:
	for dir in $(SUBDIRS); do \
		$(MAKE) -C $$dir clean; \
	done

.PHONY: all clean
```

### 13.7  常见问题与解决

| 问题                          | 解决方案                                                                 |
|-------------------------------|--------------------------------------------------------------------------|
| 子目录 Makefile 未执行        | 确保 `-C` 后路径正确，且子目录存在 Makefile。                           |
| 变量未传递到子目录            | 使用 `export` 导出变量，或通过命令行传递（如 `$(MAKE) VAR=value`）。    |
| 并行构建 (-j) 失败            | 检查子 Makefile 中的依赖是否完整，避免资源竞争。                        |
| 清理目标 (`clean`) 未执行     | 确保子 Makefile 中定义了 `clean` 目标，且未被 `.PHONY` 排除。           |

---

### 13.8  总结

嵌套 `make` 的核心方法：

1. 使用 `$(MAKE) -C <dir>` 调用子目录 Makefile。
2. 通过 `export` 传递全局变量（如编译器、编译选项）。
3. 显式定义目标依赖关系（如 `src: lib`）。
4. 统一管理清理操作（递归调用 `clean`）。

通过合理设计嵌套结构，可实现模块化、可维护的构建系统。
