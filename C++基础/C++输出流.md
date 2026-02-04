## 1  输出流

### 1.1  标准输出流 cout

#### 1.1.1  功能

cout 是 iostream 类的一个实例，是行缓冲 I/O，连接到标准输出设备，通常是显示屏。全称为 std::cout。

使用示例

```cpp
cout << v1 << ... << endl;
```

- `<<`：流插入运算符
- `endl`：end line，输出时插入换行符并刷新流

> [!tip] cout 对象和 printf 的速度到底谁更快？
> - 通常认为使用 `cout` 对象进行输出要比 `printf` 进行输出慢的多
> - 参考博客 [C++的cin/cout为什么比C语言的scanf/printf慢](https://blog.csdn.net/zq_onlytime/article/details/44900081)，在经过上述 2.1 版本的优化后，将 stdin 和 cin 解绑后，其实二者速度已经差不多。
> - 其次，根据网上一些网友的实验，endl 也是拖累 cout 速度的一大元凶，因为 endl 会在添加换行符的同时刷新输出缓冲区，从而降低代码的运行效率。(未验证)

### 1.2  文件输出流对象

- ofstream 类支持磁盘文件输出

```cpp
// 1. 在构造函数中指定文件名，构造这个对象时，该文件将会自动打开
ofstream myFile("filename");

// 2. 调用默认构造函数，之后使用 open() 成员打开文件
ofstream myFile;
myFile.open("filename");

// 3. 在构造对象时或 open() 打开时可以指定模式
ofstream myFile("filename", mode);
```

- 参数
	- filename：指定文件路径名
	- mode：输出的模式

### 1.3  二进制文件流

- 对于不需要阅读的文件，没有必要使用文本文件，因此可以直接以二进制文件流的方式打开

```cpp
// 1. 构造文件流对象时，指定二进制输出模式
ofstream myFile("filename", ios_base::binary);
// 2. 以默认构造创建文件流对象，再使用 setmode 成员函数，修改文件流的模式
ofstream myFile("filename");
myFile.setmode(ios_base::binary);
```

- 原则上，二进制文件流不支持**对象**序列化

### 1.4  字符串输出流 ostringstream

- 将字符串作为输出流的目标，可以实现其他数据类型转换为字符串的功能
- 将字符串视为文件

#### 1.4.1  功能

- 用于构造字符串
- 支持 ofstream 类除 open、 close 外所有操作
- str() 成员函数返回当前已经构造的字符串

```cpp
ostringstream os;
os << endl;
string s = os.str();
```

## 2  插入运算符 <<

- 作用：用于传送字节到一个输出流对象

## 3  成员函数


### 3.1  成员函数：put

#### 3.1.1  功能

- 把一个字符写入到输出流中
- 失败时，调用 setstate(badbit)，设置流状态为 badbit

#### 3.1.2  函数原型

```cpp
basic_ostream& put( char_type ch );
```

- 参数
	- ch：写入字符
- 返回值：`*this`

#### 3.1.3  补充

- 与 opertor<< 不同，该函数没有为了 signed char 和 unsigned char 重载
- 与格式化输出不同，输出失败时，该函数不会设置 failbit

### 3.2  成员函数：write

#### 3.2.1  功能

- 非格式化，输出字符数组中连续位置的字符，该字符数组的首元素由 s 指定
- 字符将插入到输出序列中，直到
	- count 个字符都已经插入到输出序列中
	- 插入输出序列时失败
		- 插入失败时，将调用 setstate(badbit)

#### 3.2.2  函数原型

```cpp
basic_ostream& write( const char_type* s, 
				std::streamsize count );
```

- 参数
	- s：指向要插入的字符串
	- count：要插入的字符个数
- 返回值：`*this`

#### 3.2.3  补充

- 与 operator<< 不同，该函数没有为 signed char 和 unsigned char 提供重载
- 与格式化输出不同，该函数输出失败时，不设置 failbit

### 3.3  成员函数：seekp——慎用

#### 3.3.1  功能

- 设置当前关联的 streambuf 对象的位置指示器

#### 3.3.2  函数原型

```cpp
basic_ostream& seekp( pos_type pos );

basic_ostream& seekp( off_type off, std::ios_base::seekdir dir );
```

- 参数
	- pos：对输出位置指示器设置绝对位置
	- off：对输出位置指示器设置相对位置
	- dir：定义相对偏移的基准位置
- 返回值
	- 成功
	- 失败

#### 3.3.3  补充

dir 允许的值

| 内容  | 说明          |
| --- | ----------- |
| beg | 流的开始位置      |
| end | 流的结束位置      |
| cur | 流位置指示器的当前位置 |

### 3.4  成员函数：tellp——慎用

#### 3.4.1  功能

- 返回当前关联 streambuf 对象的输出位置指示符

#### 3.4.2  函数原型

```cpp
pos_type tellp();
```

- 参数：无
- 返回值
	- 成功：当前输出位置
	- 失败：pos_type(-1)

#### 3.4.3  补充

- 通过下行代码获取位置

```Cpp
rdbuf()->pubseekoff(0, std::ios_base::cur, std::ios_base::out)
```


### 3.5  成员函数：width

- setw() 的效果和 width() 的效果几乎一样
- 二者都是仅影响紧随其后的输出项
- 因此，在流对象进行连续输出时，如果要控制多个输出项的宽度，那么就只能使用 setw()

函数原型

```cpp
stream.width(n);
```

- 参数
	- stream：流对象
	- n：输出宽度

### 3.6  成员函数：str

- 将字符串输出流的值转换为字符串，并返回

函数使用示例

```cpp
// 构造字符串函数
inline string toString(const T &v){
	ostringstream os;
	os << v;
	os.str();
}
```

- 返回值：输出流生成的字符串

## 4  操纵符

### 4.1  功能

- **插入运算符与操纵符一起工作**
- 控制输出格式
- 定义在：ios_base 类、`<iomainp>` 头文件

### 4.2  操纵符：输出宽度 setw

功能

- 控制输出宽度
- 仅影响紧随其后的输出项
函数原型

```cpp
stream << setw(n) << ... << endl;
```

- 参数
	- stream：流对象
	- n：输出的宽度

#### 4.2.1  setw 和 width 对比

```cpp
// 1 setw
stream << setw(n) << v1 << setw(n2) << v2 << ... << endl;

// 2 width
stream.width(n);
stream << v1 << v2 << endl; // v2 是不受影响的

```

### 4.3  操纵符：dec、oct、hex

作用：设置输入输出的默认进制

### 4.4  操纵符：unitbuf、nounitbuf

作用：

- unitbuf：在输出操作后，对缓冲区进行一次刷新操作
- nounitbuf：恢复默认缓冲区刷新机制

```cpp
stream << unitbuf; // 所有输出操作后，会立即刷新缓冲区
stream << nounitbuf; // 回到正常的缓冲方式
```

### 4.5  操纵符 setiosflags、resetiosflags

作用

- setiosflags：设置输出格式
- resetiosflags：取消设置输出格式
- setiosflags 的设置是持久的，直到使用 resetiosflags 重新恢复默认值为止

使用示例

```cpp
// 设置 stream 的输出格式
stream << setiosflags(flags) << ... << endl;

// 取消设置
stream << resetiosflags(flags) << ... << endl;
```

- stream：流对象
- flags：流对象的格式标志值。如果有多个值，可用按位或运算符 `|` 进行组合

#### 4.5.1  设置对齐方式

```cpp
// 设置 stream 的输出格式为左对齐
stream << setiosflags(ios::left) << ... << endl;

// 取消左对齐
stream << resetiosflags(ios::left) << ... << endl;
```

#### 4.5.2  setiosflags 的参数

流格式标志值

| 内容                   | 说明                          |
| -------------------- | --------------------------- |
| ios_base::skipws     | 输入中跳过空白                     |
| ios_base::left       | 左对齐                         |
| ios_base::right      | 右对齐                         |
| ios_base::internal   | 规定宽度内，在前缀和数值之间，插入指定的填充符号    |
| ios_base::dec        | 以十进制格式化数值                   |
| ios_base::oct        | 以八进制格式化数值                   |
| ios_base::hex        | 以十六进制格式化数值                  |
| ios_base::showbase   | 插入前缀符号以表示整数的数制              |
| ios_base::showpoint  | 对于浮点数显示小数点以及尾部的 0            |
| ios_base::uppercase  | 对于十六进制数值显示大写 A 到 F，对于科学格式显示大写 E |
| ios_base::showpos    | 对于非负数，显示正号（`+`）|
| ios_base::scientific | 以科学格式显示浮点数值                 |
| ios_base::fixed      | 以定点格式显示浮点数值（没有指数部分）|
| ios_base::unitbuf    | 每次插入后，转储并清除缓冲区内容            |

### 4.6  操纵符：setprecision

作用

- 改变输出精度
- 对于未指定 fixed 或 sentific 时，精度值表示为有效数字的位数
- 对于设置流 fixed 或 sentific 时，精度值表示为小数点后的位数

> [!tip] 默认输出精度
> 浮点数输出精度的默认值为 6

使用示例

```cpp
stream << setprecision(n);
```

- 参数：
	- n：精度值
