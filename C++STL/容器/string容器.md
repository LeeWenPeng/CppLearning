## 1  字符串:string

作用：

- 是 STL 的类型
- 用于表示一串字符
- 存储和操作类字符的对象序列

语法：

```C++
template<
    class CharT,
    class Traits = std::char_traits<CharT>,
    class Allocator = std::allocator<CharT>
> class basic_string;
```

- CharT：字符类型
- Traits：指定该字符类型上的操作
- Allocator：用于分配内部内存

- string 字符串需要引入头文件
	- `#include <string>`
	- `#include<bits/stdc++.h`

## 2  字符和字符串

### 2.1  字符和字符串的区别

- 被单引号包裹的为字符
- 被双引号包裹的为字符串
- 字符串末尾有结束符 `\0`

### 2.2  字符串和字符的所占内存空间大小不同

```C++
printf("%d %d\n", sizeof('a'), sizeof("a"));
// 1 2
```

## 3  字符数组和字符串

- C 语言中的 `char*` 本质上是一个字符数组，或者说是一个指针
- C++ 风格的 string 的本质是一个类，类中维护着 `char*`，是一个 `char*` 的容器

```c++
#include <iostream>
#include <string>
using namespace std;

int main() {

	//C风格的字符串
	char ch[] = "abcde";
	cout << ch << endl;

	//C++风格的字符串
	string st = "abcde";
	cout << st << endl;

	system("pause");
	return 0;
}
```

## 4  返回空字符

返回值类型为 string

```C++

//1 
return ""; 

//2
return {};

//3
return std::string();
```

### 4.1  现象

方法 2/3 的效率要大于方法 1

### 4.2  原因

- 核心原因：语义不同
- 方法一：返回一个 C 风格字符串构造的 string 变量。因此，编译器需要调用 `const char*` 的构造函数
- 方法二/三：直接调用 `std::string` 的默认构造，并返回
- 显而易见，方法一多了转换的步骤

## 5  成员

### 5.1  构造函数

1. 无参构造 `string();`
   1. 返回一个空字符对象
2. 有参构造 `string(const char* s);`
   1. 参数为字符串 `s`
3. 拷贝构造，`string(const string& s);`
4. 有参构造，`string(int n, char c)`
   1. 使用 n 个字符 `c` 组成字符串

### 5.2  赋值

![image-20230927164408819](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20230927164408819.png)

- 这些函数的作用是为了防止浅拷贝

### 5.3  `string` 字符串拼接

在字符串的末尾追加字符

![image-20230927164810903](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20230927164810903.png)

### 5.4  `string` 字符串的查找和替换

![image-20230927164921024](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20230927164921024.png)

#### 5.4.1  find

在 `str1` 中查询 `str2` 的起始位置下标，如果不存在返回 `-1`

```C++
size_type find( const CharT* s, size_type pos, size_type count ) const;
```

- 参数
	- s：要查找的子串
	- pos：查找的起始位置
	- count：子串 s 中可用范围，$[s, s+count)$
		- 相当于对 s 进一步截取，只保留前 count 个字符
- 返回值
	- 成功：找到的子串第一个字符的位置
	- 失败：npos
- `rfind`：reverse find，从右往左找

#### 5.4.2  replace

`str2` 中的字符，从 `pos` 开始的 `n` 个字符替换成 `str1`

```c++
int pos = str2.repalce(pos, n, str1)
```

- 替换前和替换后 `str2` 的地址保持不变，也就是说，替换的操作是在 `str2` 本身上进行的。
- 替换操作并不是按字符单位来进行的，而是子串替换
	- 也就是说，无论 `n` 多大，都会将 `str1` 替换掉 `[pos, pos+n-1]` 的字符

### 5.5  字符串的比较

![image-20230930112354240](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20230930112354240.png)

- 参数
	- s：被比较的字符串
- 返回值
	- -1：小于 s
	- 0: 与 s 相等
	- 1：大于 s
比较方式：==根据字符串的 ASCII 值进行比较==

### 5.6  字符存取

从 `string` 中访问单个字符的方式有两种：

1. 通过 `[]` 运算符：`char& oprator[](int n);`
2. 通过 `at` 方法：`char& at(int n);`

> 也能通过上述两种方式对字符串进行修改，但尽量不要。

### 5.7  字符串插入与删除

![image-20230930112754925](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20230930112754925.png)

### 5.8  获取 string 子串

从 string 对象中获取字符串子串

![image-20230930112951981](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20230930112951981.png)
