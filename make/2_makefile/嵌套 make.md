## 1  基本语法与原理

### 1.1  基本调用语法

顶层 Makefile 通过 `make -C <subdir>` 调用子目录的构建过程

```makefile
# 基本调用格式
sub_target:
	$(MAKE) -C <子目录路径> <目标> <key=value>
```

### 1.2  `$(MAKE)` 变量

```makefile
# $(MAKE) 表示当前使用的 make 命令
# 使用 $(MAKE) 而不是 make 的原因：
# 1. 传递 make 的选项（如 -j, -k）
# 2. 兼容不同名称的 make（如 gmake, pmake）
# 3. 处理嵌套调用时的一致性

# 示例：使用 $(MAKE) 调用子目录
lib:
	$(MAKE) -C lib

# 等价于手动切换目录
lib_manual:
	cd lib && make
```

### 1.3  完整示例

```makefile
# 项目结构示例
# project/
# ├── Makefile
# ├── lib/
# │   ├── Makefile
# │   ├── math.c
# │   └── utils.c
# ├── src/
# │   ├── Makefile
# │   └── main.c
# └── test/
#     ├── Makefile
#     └── test.c

# 顶层 Makefile
SUBDIRS = lib src test

# 默认目标：按依赖顺序构建
all: lib src test

# 子目标定义
lib:
	$(MAKE) -C lib

src: lib
	$(MAKE) -C src

test: src
	$(MAKE) -C test

clean:
	for dir in $(SUBDIRS); do \
		$(MAKE) -C $$dir clean; \
	done

.PHONY: all lib src test clean
```

## 2  参数传递机制

### 2.1  命令行参数传递

```makefile
# 顶层 Makefile
build_with_debug:
	$(MAKE) -C src DEBUG=1 OPTIMIZE=0

build_with_release:
	$(MAKE) -C src DEBUG=0 OPTIMIZE=2
```

```makefile
# src/Makefile
ifeq ($(DEBUG))
	CFLAGS += -g -DDEBUG
endif

ifeq ($(OPTIMIZE))
	CFLAGS += -O2
endif
```

### 2.2  `export` 机制

```makefile
# 顶层 Makefile - 导出变量到子 Makefile

# 编译器设置
CC = gcc
CXX = g++
CFLAGS = -Wall -O2
CXXFLAGS = -std=c++11 -Wall

# 导出单个变量
export CC
export CXX

# 导出多个变量（推荐方式）
export CFLAGS CXXFLAGS

# 导出所有变量（谨慎使用）
export

# 取消导出
unexport UNDESIRED_VAR

# 子目录构建规则
lib:
	$(MAKE) -C lib

# 子 Makefile 可以直接使用导出的变量
# lib/Makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

### 2.3  环境变量传递

```makefile
# Makefile 不仅能传递变量给子 Makefile，还能设置环境变量

# 设置环境变量（会传递给子进程）
export PATH := $(shell pwd)/tools:$(PATH)

# 设置局部变量（不传递给子进程）
LOCAL_VAR = value

# 验证示例
show_env:
	@echo "PATH = $(PATH)"
	@echo "LOCAL_VAR = $(LOCAL_VAR)"
	@echo "--- 子进程环境 ---"
	$(MAKE) -C subdir show_env

# subdir/Makefile
show_env:
	@echo "子进程 PATH = $(PATH)"
	@echo "子进程 LOCAL_VAR = $(LOCAL_VAR)"
```

## 3  路径处理策略

### 3.1  顶层目录路径管理

```makefile
# 获取顶层绝对路径的几种方式

# 方式1：使用 shell pwd（推荐）
TOP_DIR := $(shell pwd)

# 方式2：使用 CURDIR（make 内置变量）
TOP_DIR := $(CURDIR)

# 方式3：使用 abspath 函数
TOP_DIR := $(abspath .)

# 导出给子目录使用
export TOP_DIR

# 定义其他路径变量
INCLUDE_DIR = $(TOP_DIR)/include
LIB_DIR = $(TOP_DIR)/lib
BIN_DIR = $(TOP_DIR)/bin

export INCLUDE_DIR LIB_DIR BIN_DIR

# 子目录中引用
# src/Makefile
INCLUDES = -I$(TOP_DIR)/include -I.
```

### 3.2  子目录路径处理

```makefile
# 子 Makefile 中的路径处理

# 获取当前子目录路径
CURRENT_DIR := $(shell pwd)

# 相对于顶层目录的路径
RELATIVE_PATH := $(subst $(TOP_DIR)/,,$(CURRENT_DIR))

# 构建输出路径
OBJ_DIR = $(TOP_DIR)/build/$(RELATIVE_PATH)
BIN_DIR = $(TOP_DIR)/bin

# 自动创建目录
$(OBJ_DIR)/%.o: %.c
	@mkdir -p $(@D)
	$(CC) $(CFLAGS) -c $< -o $@
```

## 4  并行构建处理

### 4.1  继承并行参数

```makefile
# 默认情况下，子 make 会继承父 make 的 -j 参数
# 顶层执行：make -j8
# 子目录也会使用 8 个作业

# 显式控制子目录并行度
parallel_build:
	$(MAKE) -C lib -j4
	$(MAKE) -C src -j4
	$(MAKE) -C test -j2

# 限制总并行度
limited_parallel:
	$(MAKE) -j4 -C lib
	$(MAKE) -j4 -C src
	$(MAKE) -j4 -C test
```

### 4.2  依赖关系与并行

```makefile
# 正确的依赖关系确保并行构建的正确性

# 有依赖关系 - 串行执行（lib 完成后再执行 src）
all: lib src

lib:
	$(MAKE) -C lib

src: lib
	$(MAKE) -C src

# 无依赖关系 - 可以并行执行
independent_targets: target1 target2

target1:
	$(MAKE) -C dir1

target2:
	$(MAKE) -C dir2

# 混合依赖
complex_deps: lib1 lib2
	$(MAKE) -C app  # 等待 lib1 和 lib2 都完成

lib1:
	$(MAKE) -C lib1

lib2:
	$(MAKE) -C lib2
```

## 5  错误处理机制

### 5.1  忽略子目录错误

```makefile
# 使用 - 前缀忽略子 make 的错误
optional_targets:
	-$(MAKE) -C experimental_feature  # 失败也不会停止
	-$(MAKE) -C optional_module
	$(MAKE) -C required_module       # 失败会停止

# 使用 -k 继续构建其他目标
continue_on_error:
	$(MAKE) -k -C dir1
	$(MAKE) -k -C dir2

# 组合使用
robust_build:
	-$(MAKE) -k -C unstable
	$(MAKE) -C stable
```

### 5.2  调试输出控制

```makefile
# 调试子目录构建过程
debug_build:
	$(MAKE) -C src --debug=v  # 详细调试信息
	$(MAKE) -C lib --debug=b  # 基本调试信息

# 只打印命令不执行
dry_run:
	$(MAKE) -C src -n
	$(MAKE) -C lib -n

# 显示变量值
show_vars:
	$(MAKE) -p -C src | grep "^CC\|^CFLAGS"
```

## 6  自动遍历子目录

### 6.1  简单遍历方法

```makefile
# 使用 shell 循环
SUBDIRS = lib src test tools

all:
	for dir in $(SUBDIRS); do \
		$(MAKE) -C $$dir || exit 1; \
	done

clean:
	for dir in $(SUBDIRS); do \
		$(MAKE) -C $$dir clean; \
	done

# 并行版本
parallel_all:
	$(foreach dir,$(SUBDIRS),$(MAKE) -C $(dir) & ) wait
```

### 6.2  模式规则遍历

```makefile
# 使用模式规则自动处理
SUBDIRS = lib src test

# 所有子目录目标
SUBDIR_TARGETS = $(addsuffix _all, $(SUBDIRS))

all: $(SUBDIR_TARGETS)

# 模式规则：%_all 对应子目录的 all 目标
%_all:
	$(MAKE) -C $* all

# 清理目标
clean:
	$(foreach dir,$(SUBDIRS),$(MAKE) -C $(dir) clean;)

.PHONY: all clean $(SUBDIR_TARGETS)
```

### 6.3  动态发现子目录

```makefile
# 自动发现所有包含 Makefile 的子目录
SUBDIRS := $(shell find . -maxdepth 2 -name "Makefile" | sed 's|/Makefile||' | sort)

# 过滤掉不需要的目录
EXCLUDE_DIRS = . ./build ./dist
SUBDIRS := $(filter-out $(EXCLUDE_DIRS), $(SUBDIRS))

# 构建所有发现的子目录
all: $(SUBDIRS)

$(SUBDIRS):
	$(MAKE) -C $@

.PHONY: all $(SUBDIRS)
```

## 7  完整的项目示例

### 7.1  项目结构

```cpp
complex-project/
├── Makefile              # 顶层 Makefile
├── config.mk             # 公共配置
├── include/              # 公共头文件
│   ├── common.h
│   └── config.h
├── libs/                 # 第三方库
│   ├── Makefile
│   └── …
├── core/                 # 核心模块
│   ├── Makefile
│   ├── module1/
│   └── module2/
├── apps/                 # 应用程序
│   ├── app1/
│   ├── app2/
│   └── Makefile
└── tests/                # 测试
    └── Makefile
```

### 7.2  顶层 Makefile

```makefile
# 顶层 Makefile - complex-project/Makefile

# 包含公共配置
include config.mk

# 定义顶层目录
TOP_DIR := $(CURDIR)
export TOP_DIR

# 定义子目录及其依赖关系
SUBDIRS = libs core apps tests

# 导出编译配置
export CC CXX
export CFLAGS CXXFLAGS LDFLAGS LDLIBS
export BUILD_TYPE

# 默认目标
all: libs core apps

# 子目录目标
libs:
	@echo "=== 构建库文件 ==="
	$(MAKE) -C libs

core: libs
	@echo "=== 构建核心模块 ==="
	$(MAKE) -C core

apps: core
	@echo "=== 构建应用程序 ==="
	$(MAKE) -C apps

tests: core
	@echo "=== 构建测试 ==="
	$(MAKE) -C tests

# 清理
clean:
	@echo "=== 清理所有构建文件 ==="
	for dir in $(SUBDIRS); do \
		if [ -f $$dir/Makefile ]; then \
			$(MAKE) -C $$dir clean; \
		fi \
	done

# 安装
install: apps
	@echo "=== 安装应用程序 ==="
	$(MAKE) -C apps install

# 测试
test: tests
	@echo "=== 运行测试 ==="
	$(MAKE) -C tests run

# 重新构建
rebuild: clean all

# 帮助信息
help:
	@echo "可用目标:"
	@echo "  make all      - 构建所有目标（默认）"
	@echo "  make clean    - 清理构建文件"
	@echo "  make install  - 安装应用程序"
	@echo "  make test     - 运行测试"
	@echo "  make rebuild  - 重新构建"
	@echo "  make help     - 显示此帮助"

.PHONY: all clean install test rebuild help $(SUBDIRS)
```

### 7.3  公共配置文件

```makefile
# config.mk - 公共配置

# 构建类型
BUILD_TYPE ?= release

# 编译器
CC = gcc
CXX = g++

# 编译选项
CFLAGS = -Wall -I$(TOP_DIR)/include
CXXFLAGS = -std=c++11 -Wall -I$(TOP_DIR)/include

# 根据构建类型调整选项
ifeq ($(BUILD_TYPE),debug)
	CFLAGS += -g -DDEBUG -O0
	CXXFLAGS += -g -DDEBUG -O0
else ifeq ($(BUILD_TYPE),release)
	CFLAGS += -O2 -DNDEBUG
	CXXFLAGS += -O2 -DNDEBUG
endif

# 链接选项
LDFLAGS = -L$(TOP_DIR)/libs/build
LDLIBS = -lm -lpthread

# 安装路径
PREFIX ?= /usr/local
BINDIR ?= $(PREFIX)/bin
LIBDIR ?= $(PREFIX)/lib
INCLUDEDIR ?= $(PREFIX)/include

export PREFIX BINDIR LIBDIR INCLUDEDIR
```

### 7.4  子目录 Makefile 示例

```makefile
# core/Makefile - 核心模块

# 使用顶层传递的变量
include $(TOP_DIR)/config.mk

# 模块特定配置
MODULE_NAME = core
MODULE_DIR = $(TOP_DIR)/core

# 源文件
SRCS = $(wildcard *.c) $(wildcard module1/*.c) $(wildcard module2/*.c)
OBJS = $(SRCS:.c=.o)

# 输出目录
OBJ_DIR = $(TOP_DIR)/build/$(MODULE_NAME)
LIB_DIR = $(TOP_DIR)/libs/build

# 目标文件路径
OBJS_WITH_PATH = $(addprefix $(OBJ_DIR)/,$(OBJS))

# 目标库
TARGET = $(LIB_DIR)/lib$(MODULE_NAME).a

# 默认目标
all: $(TARGET)

# 创建库
$(TARGET): $(OBJS_WITH_PATH)
	@mkdir -p $(LIB_DIR)
	$(AR) rcs $@ $^
	@echo "构建库: $@"

# 编译规则
$(OBJ_DIR)/%.o: %.c
	@mkdir -p $(@D)
	$(CC) $(CFLAGS) -c $< -o $@

# 清理
clean:
	rm -rf $(OBJ_DIR)
	rm -f $(TARGET)

.PHONY: all clean
```

## 8  常见问题与解决方案

### 8.1  问题排查表

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 子目录未执行 | 路径错误或子目录无 Makefile | 检查 `-C` 路径，确保子目录有 Makefile |
| 变量未传递 | 未使用 `export` | 使用 `export` 导出变量，或通过命令行传递 |
| 并行构建失败 | 子目录间有未声明的依赖 | 明确声明依赖关系，或串行执行有依赖的部分 |
| 清理不彻底 | 子目录清理目标未定义 | 确保每个子 Makefile 都有 `clean` 目标 |
| 环境变量丢失 | 使用了局部变量而非环境变量 | 使用 `export` 设置环境变量 |
| 路径问题 | 相对路径在子目录中解析错误 | 使用 `$(TOP_DIR)` 等绝对路径 |

### 8.2  调试技巧

```makefile
# 添加调试信息的技巧

# 1. 显示调用的命令
debug_call:
	@echo "正在调用: $(MAKE) -C subdir"
	$(MAKE) -C subdir

# 2. 显示变量值
debug_vars:
	@echo "TOP_DIR = $(TOP_DIR)"
	@echo "SUBDIRS = $(SUBDIRS)"
	@echo "CFLAGS = $(CFLAGS)"

# 3. 追踪执行流程
trace:
	$(MAKE) --trace -C src

# 4. 测试子目录是否存在
check_dirs:
	@for dir in $(SUBDIRS); do \
		if [ ! -f $$dir/Makefile ]; then \
			echo "警告: $$dir 没有 Makefile"; \
		fi \
	done
```

## 9  进阶技巧

### 9.1  条件构建

```makefile
# 根据条件跳过某些子目录
BUILD_TESTS ?= 1
BUILD_DOCS ?= 0

SUBDIRS = libs core apps

ifeq ($(BUILD_TESTS))
	SUBDIRS += tests
endif

ifeq ($(BUILD_DOCS))
	SUBDIRS += docs
endif

all: $(SUBDIRS)

$(SUBDIRS):
	$(MAKE) -C $@
```

### 9.2  性能优化

```makefile
# 并行构建优化
# 根据 CPU 核心数自动设置并行度
NUM_CORES := $(shell nproc 2>/dev/null || sysctl -n hw.ncpu 2>/dev/null || echo 4)

# 为每个子目录设置适当的并行度
PARALLEL_LIBS = $(shell echo $$(( $(NUM_CORES) / 4 )) )
PARALLEL_APPS = $(shell echo $$(( $(NUM_CORES) / 2 )) )

all: libs apps

libs:
	$(MAKE) -C libs -j$(PARALLEL_LIBS)

apps: libs
	$(MAKE) -C apps -j$(PARALLEL_APPS)
```

## 10  最佳实践总结

1. **始终使用 `$(MAKE)`**：确保选项传递和兼容性
2. **明确声明依赖关系**：避免并行构建时的竞争条件
3. **统一变量管理**：使用公共配置文件或顶层导出
4. **处理路径一致性**：使用绝对路径或统一的相对路径基准
5. **提供完整的生命周期目标**：确保每个子目录都有 `all`、`clean` 等目标
6. **错误处理要健壮**：合理使用 `-` 和 `-k` 选项
7. **文档化构建流程**：在顶层 Makefile 中提供清晰的帮助信息
8. **支持灵活的构建配置**：允许通过变量控制构建选项

递归构建是大型项目管理的重要手段，合理设计可以显著提高项目的可维护性和构建效率。
