## 1  标准 C 库 IO 和 Linux 系统 IO 的关系

标准 C 库的 IO 程序和 Linux 系统的 IO 程序之间的关系为：调用关系。

![标准C库和LinuxIO的关系](标准C库和LinuxIO的关系.md|1000)

## 2  IO 类

![IO 库类型和文件](IO库类型和文件表.png)

三个独立头文件

- iostream：定义流用于读写流的基本类型
- fstream：定义了读写命名文件的类型
- sstream：读写内存 string 对象的类型

## 3  输入输出类

### 3.1  iostream 类

- 一个 iostream 对象可以是数据的源或目的
- iostream 库中定义了 cin、cout、ceer 以及 clog 对象
- fstream、 stringstream 类由 iostream 类派生。
- iostream 、 fstream 和 stringstream 类继承了 istream 和 ostream 类的功能。

```C
#include <iostream>
```

- iostream 相当于 istream + ostream

### 3.2  fstream 类

- fstream 类支持磁盘文件输入和输出
- 一个 fstream 对象具有两个逻辑子流，一个用于输入， 另一个用于输出

### 3.3  stringstream

- stringstream 类支持面向字符串的输入和输出
- 可以对同一个字符串的内容交替读写
- stringstream 对象具有两个逻辑子流，一个用于输入，另一个用于输出

### 3.4  IO 类之间的关系

- 派生关系：fstream、 stringstream 类由 iostream 类派生。
- 继承关系：输入输出类之间具有继承关系
	- 比如，ifstream 和 istringstream 都继承自 istream。

> [!note] 继承机制和模版对于 IO 类的作用
**继承机制**，使得标准库特性可以无差别应用于普通流、文件流和 string 流，以及宽字符流版本

### 3.5  宽字符

- 宽字符类型 `wchar_t`
- 数据类型和函数的名字以 w 开始

## 4  流对象

- C++ 语言不直接处理输入输出，而是通过一族定义在标准库中的类型来处理 IO
- 流对象则是介于数据的生产者和数据的消费者之间，对二者建立联系，并管理数据的流动

### 4.1  IO 对象无拷贝和赋值

- 流对象不能拷贝或赋值。
- 由于不能拷贝 IO 对象，因此，**不可以将形参或返回类型设置为流类型**。
- 进行 IO 操作的函数通常以**引用方式**传递和返回流。
- 由于读写一个 IO 对象会改变流状态，因此**传递和返回的引用不能是 const 的**。

## 5  流对象与文件操作

- 程序建立一个流对象
- 指定流对象与文件对象建立连接
- 程序操作流对象
- 流对象对设备产生作用

## 6  额外的流

### 6.1  标准错误流 `cerr`

`cerr << 变量1 [<< 变量2 [<< endl]];`

### 6.2  标准日志流

**语法：**DER}非缓冲标准错误流**，是 `clog` 类的一个实例，附属到标准输出设备，通常是显示屏。

**每个流插入到 `iostream` 都会立刻发送到标准输出设备输出。**DER}非缓冲标准错误流**`clog << 变量1 [<< 变量2 [<< endl]];`

## 7  条件状态

### 7.1  概括

- 流对象具有状态, 类型为 strm::iostate

![](file-20250823143656966.png)

- 一个流一旦发生错误，其上后续所有的 IO 操作都会失败。
- 只有当一个流处于无错状态时，才可以正常使用。
- 因此，**在使用流之前，应该检查其是否处于良好状态**。
- 确定一个流对象状态的简单方法：**将其作为一个条件使用**

```cpp
while(cin >> word)
	//todo…
```

### 7.2  条件状态类型

- iostate 类型：表达流状态的完整功能。
- 该类型对象为一个状态的**位集合**，其中又包括四种位状态
	- badbit
	- failbit
	- eofbit
	- goodbit
- 通过**位运算**能够检测或设置多个标志位

### 7.3  流状态的获取

通过如下四个函数获取当前流的状态

- eof()
- fail()
- bad()
- good()

其中，good 和 fail 可以用来确定流的整体状态，而 eof 和 bad 操作表示特定错误

#### 7.3.1  rdstate()

作用：能够通过 stream 对象的成员函数 rdstate() 来获取 stream 对象的当前状态

```cpp
// 获取 stream 的当前状态
auto state = stream.rdstate();
```

### 7.4  条件状态的管理

可以通过 clear() 和 setstate（）设置流状态

#### 7.4.1  clear()

##### 3.1 功能

- 复位流的状态

##### 3.2 函数原型

```cpp
// 无参，复位所有的错误标志位
stream.clear();
// 有参，复位指定错误标志位
stream.clear(iostate flags);
```

- 参数
	- flags：流状态。通过位运算符进行修改

##### 补充：复位指定错误标志位

```cpp
// 例子：复位 failbit 和 badbit, 其余标志位保持不变
stream.clear(stream.rdstate() & ~cinfailbit & ~cin.badbit);
```

#### 7.4.2  setstate()

作用：设置流状态

```cpp
// 设置 cin 状态
cin.setstate(state);
```

- state：流状态，可以通过 rdstate() 设置，通过位运算修改

## 8  管理输出缓冲区

- 每个输出流都管理一个输出缓冲区，用以保存程序读写的数据。
- 有了缓冲区，操作系统可以将程序的多个输出操作组合成单一的系统级写操作，以提升性能。
- 往往在缓冲刷新时，数据才将从缓冲区内，真正的写到输出设备或文件中。

> [!note] 如果程序崩溃，输出缓冲区不会被刷新
> 如果程序异常终止，则无法正常调用 return 函数刷新缓冲区。因此，输出缓冲区内的数据可能停留在输出缓冲区内等待打印
> 当调试程序时，需要注意此点！

### 8.1  缓冲区刷新的原因

- 程序正常结束，return 将刷新缓冲区
- 缓冲区满
- 通过 endl 操作符，手动显式刷新缓冲区
- 在每个输出操作后，使用 unitbuf 设置流的内部状态，以使得流在每次写操作后都普进行一次刷新缓冲区。
- 当一个流关联到另一个流。当读写被关联的流时，关联流的缓冲区会被刷新。

### 8.2  操纵符：endl、flush、ends

#### 8.2.1  endl

作用：换行并刷新缓冲区

```cpp
// 输出 hi 并换行，然后刷新缓冲区
cout << "hi" << endl;
```

#### 8.2.2  flush

作用：刷新缓冲区，不附加任何额外字符

```cpp
// 输出 hi，然后刷新缓冲区
cout << "hi" << flush;
```

#### 8.2.3  ends

作用：向缓冲区插入一个空字符，然后刷新缓冲区

```cpp
// 输出 hi 和一个空字符，然后刷新缓冲区
cout << "hi" << ends;
```

### 8.3  操纵符： unitbuf 和 nounitbuf

作用：

- unitbuf：设置流状态，使得流对象在每次写操作后，都进行一次 flush 操作；
- nounitbuf：操作符则取消该设置。

```Cpp
cout << unitbuf; // 设置 cout，使得其每次写入操作后都刷新缓冲区
// 任何输出操作都直接刷新，无缓冲
cout << nounitbuf; // 回归正常
```

### 8.4  关联输入和输出流

- 默认情况下，cin 和 cerr 都关联到 cout。
- 因此，读写 cin 和 cerr，cout 都将刷新。

### 8.5  成员函数：tip

作用：将两个流对象关联到一起

```cpp
// 1. 将 stream1 关联到 stream2
stream1.tip(&stream2);

// 2. 取消关联
stream1.tip(nullptr);
```

- 每个流同时最多关联一个流
- 多个流可以同时关联到同一个 ostream。

## 9  成员函数

### 9.1  成员函数：open

#### 9.1.1  功能

- 打开 filename 文件并将其与流相联
- 需要指定打开的模式
	- 默认为 `std::ios_base::out`
- 成功时，调用 clear() 成员刷新流
- 失败时，调用 setstate(failbit) 成员，设置流状态为 failbit

#### 9.1.2  函数原型

```cpp
void open(const std::filesystem::path& filename, 
		std::ios_base::openmode mode = std::ios_base::out);
```

- 参数
	- filename：文件路径
	- mode：模式
- 返回值：无

#### 9.1.3  补充：mode 表

| 内容                  | 说明          |
| ------------------- | ----------- |
| ios_base::app       | 在流的末尾追加写    |
| ios_base::binary    | 二进制模式       |
| ios_base::in        | 读           |
| ios_base::out       | 写           |
| ios_base::trunc     | 打开时丢弃流的内容   |
| ios_base::ate       | 打开后立即查找流的末尾 |
| ios_base::noreplace | 独占打开        |
