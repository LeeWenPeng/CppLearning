## 1  三个重要的输入流类

- istream：适合用于顺序文本模式输入。cin 是其实例。
- ifstream：支持磁盘文件输入
- istringstream：字符串输入流

## 2  输入流对象

### 2.1  标准输入流 cin

`std::cin` 是 isotream 的一个实例，附属到标准输入设备，通常是键盘。

```cpp
cin >> v1 >> v2;
```

- `>>`：流提取运算符

#### 2.1.1  优化 cin 和 cout 效率代码

```cpp
std::ios::sync_with_stdio(false);
std::cin.tie(0); // 取消 cin 和 cout 的关联
```

### 2.2  文件流对象

```cpp
// 1 构造时，指定文件名，构造该对象时，文件自动打开
ifstream myFile("filename");

// 2 默认构造，然后使用 open 函数打开
ifstream myFile;
myFile.open("filename");

// 3 打开文件时，可以指定模式
ifstream myFile("filename", mode);
```

- 参数
	- filename：指定文件名
	- mode 打开模式

#### 2.2.1  补充 mode 表

| 内容                  | 说明          |
| ------------------- | ----------- |
| ios_base::app       | 在流的末尾追加写    |
| ios_base::binary    | 二进制模式       |
| ios_base::in        | 读           |
| ios_base::out       | 写           |
| ios_base::trunc     | 打开时丢弃流的内容   |
| ios_base::ate       | 打开后立即查找流的末尾 |
| ios_base::noreplace | 独占打开        |

### 2.3  字符串输入流对象 istringstream

- 将字符串作为文本输入流的源，可以将字符串转换为其他数据类型
- 用于从字符串读取数据
- 在构造函数中设置要读取的字符串
- 支持 ifstream 类除了 open 、close 外的所有操作

```cpp
// 将字符串转换为数值
inline T fromString(const string& str){
	istringstream is(str);
	T v;
	is >> v;
	return v;
}
```

## 3  提取运算符 >>

- 提取运算符 >> 支持对于所有标准 C++ 数据类型
- 从输入流对象获取字节最容易的方法
- ios 类中的操纵符很多都可以应用于输入流。
	- 但大多数不建议使用
	- 最值得使用的几个操纵符为：dec、oct、hex

## 4  成员函数

### 4.1  open

#### 4.1.1  功能

- 打开一个文件，将其与文件流关联起来

#### 4.1.2  函数原型

```cpp
void open( const char* filename, [std::ios_base::openmode] mode = [std::ios_base::in]);
```

- 参数
	- filename：文件名
	- mode：文件模式

### 4.2  get

#### 4.2.1  功能

- 从流中提取一个或多个字节

#### 4.2.2  函数原型

```cpp
int_type get();
```

- 功能
	- 读取一个可用字节并返回。
- 参数
- 返回值
	- 成功：提取到的字节
	- 错误：返回 `Traits::eof()`，设置 failbit 和 eofbit

```cpp
basic_istream& get(char_type& ch);
```

- 功能：读取一个可用字节，并将其存储在 ch 中
- 参数 ch：要将结果写入的字符引用
- 返回值：`*this`

```cpp
// 等价于 get(s, count, widen('\n'));
basic_istream& get(char_type* s, std::streamsize count);

basic_istream& get(char_type* s, std::streamsize count, char_type delim);
```

- 功能：
	- 读取多个字符，并将它们存储在 s 中
	- 读取结束的事件
		- 最多读取 std::max(0, count - 1) 个字符
		- 下一个可用字符等于 delim
		- 文件结束
- 参数
	- s：指向用于存储结果的字符串指针
	- count：s 的大小
	- delim：停止提取的分隔字符。它不被提取也不被存储。
- 返回值：`*this

```cpp
// 等价于 get(strbuf, widen('\n'));
basic_istream& get(basic_streambuf& strbuf);

basic_istream& get(basic_streambuf& strbuf, char_type delim);
```

- 功能：
	- 读取字符并将它们插入到指定的输出流对象 strbuf 控制的输出序列中。
	- 读取结束事件
		- 文件结束
		- 插入失败
		- 下一个可用字符等于 delim
		- 发生异常
- 参数
	- strbuf：输出流缓冲区
	- delim：停止提取的分隔字符。它不被提取也不被存储。
- 返回值：`*this

#### 4.2.3  补充

- 如果没有提取字符，则调用 setstate（failbit）。
- 所有版本都将 gcount（）的值设置为提取的字符数。

### 4.3  getline

#### 4.3.1  功能

- 从流中提取多个字符，直到一行结束 (`\n`) 或者遇到提取终止符 delim
- 提取结束事件
	- 文件结束
	- 下一个提取的可用字符 c 等于 delim
	- count < 0 或者是 提取到 count - 1 个字符

#### 4.3.2  函数原型

```cpp
// 等价于 getline(s, count, widen('\n'));
basic_istream& getline( char_type* s, [std::streamsize] count);

basic_istream& getline( char_type* s, [std::streamsize] count, char_type delim );
```

- 参数
	- s：指向用于存储结果的字符串指针
	- count：s 的大小
	- delim：停止提取的分隔字符。
		- 它被提取， 被 gcount() 统计，但是不被存储。
- 返回值：`*this`

### 4.4  read

#### 4.4.1  功能

- 从流中提取多个字符
- 提取结束事件
	- count 个字节被提取并且存储
	- 文件结束

#### 4.4.2  函数原型

```cpp
basic_istream& read( char_type* s, [std::streamsize] count );
```

- 参数
	- s：指向用于存储目标字节的字符数组的指针
	- count：读取字节数
- 返回值：`*this`

#### 4.4.3  补充

### 4.5  seekg

#### 4.5.1  功能

- 设置当前关联的流对象的输入位置指示器

#### 4.5.2  函数原型

```cpp
basic_istream& seekg( pos_type pos );

basic_istream& seekg( off_type off, [std::ios_base::seekdir] dir);
```

- 参数
	- pos：绝对位置
	- off：相对位置
	- dir：偏移基准
		- beg：流的起始位置
		- end：流的结尾位置
		- cur：当前位置
- 返回值：`*this`

#### 4.5.3  补充

- `seekg(n)` 不一定等同于 `seek(n, beg)`。

### 4.6  tellg

#### 4.6.1  功能

- 返回当前流对象的输入位置指示器

#### 4.6.2  函数原型

```cpp
pos_type tellg();
```

- 参数
- 返回值
	- 成功：指针的当前位置
	- 失败：pos_type(-1)

#### 4.6.3  补充

### 4.7  close

#### 4.7.1  功能

- 关闭与输入流相联的文件

#### 4.7.2  函数原型

```cpp
void close()
```
