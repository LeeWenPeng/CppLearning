## 1 vector 容器概念

表示对象的集合，其中所有对象的类型都相同。

集合中每个对象都有一个对应索引，索引用于访问对象。

+ 头文件`#include<vector>`
+ 命名空间`using std::vector;`

vector 是一个**类模版**，在使用前需要**实例化**。

vector容器特点

1. 和数组十分相似，被称为**单端数组**
2. 可以**动态扩展**
3. 其迭代器是**支持随机访问**的迭代器

> [!note] vector 容器的动态扩展：
>
> 1. 并不是在原空间后接入新空间，而是，重新寻找一个更大的内存空间，将原内存中的数据复制到新内存空间中，并释放原空间
>
> 2. 找到的新空间的内存大小往往是原空间的 `2` 倍

![image-20230930181910686](image-20230930181910686.png)

+ 单端数组：只允许尾端进行插入删除操作
	+ `push_back()` 在尾端插入
	+ `pop_back()` 在尾端删除
+ 提供接口：
	+ `front()` 第一个元素
	+ `back()` 最后一个元素
	+ `insert()` 插入接口
	+ `erase()` 删除接口
+ 迭代器：
	+ `beigin()` 指向第一个元素的第一个位置
	+ `end()` 指向最后一个元素的后一个位置
	+ `rbegin()` 指向最后一个元素的位置
	+ `rend()` 指向第一个元素的前一个位置

## 2 定义和初始化

### 1 概念

```C++
// v1 是一个空的 vector，潜在的元素是 T 类型的，执行默认初始化
vector<T> v1;
// 拷贝构造
vector<T> v2(v1);
vector<T> v2 = v1;
// n 个 elem，其中，v3的元素值为val；
vector<T> v3(n, val);
// 值初始化，只提供元素数量，不提供元素值，只支持直接初始化，v4的元素值为类型默认初始值
vector<T> v4(n);
// 提供初始元素值的列表，只能列表初始化，包含初始值个数的元素，每个元素被赋予相应初始值
vector<T> v5{a,b,c,d,...};
vector<T> v5 = {a,b,c,d,...};
// vector<T> v5(a,b,c,d,...); // wrong!
// 拷贝构造的另一种形式，根据迭代器指定了范围
vector<T> v6(v5.beigin(), v5.end());
```

>  + 拷贝构造时，俩个 vector **容器内元素数据类型必须一致**
>  + 提供初始元素值的列表，**只能**将初始值放到花括号内进行**列表初始化**，而不能放在圆括号内

>[!BUG] 值初始化的限制
>+ 如果，vector 类型不支持默认初始化，则必须提供初始值
>+ 只支持**直接初始化**

>[!important] 初始化时，花括号和圆括号的区别
>+ 圆括号：提供值是为了**构造**vector对象
>+ 花括号：
>	+ 一般是**列表初始化**。在初始化的过程会将可能将花括号内的值当成是元素初始值列表
>	+ 当花括号内的元素**无法**进行**列表初始化时**，则是为了**构造**vector

>[!note] vector 直接初始化的三种情况
>1. 初始值已知，且数量较少
>2. 初始值是另一个 vector 对象的副本
>3. 所有元素的初始值都一样

### 2 代码

```c++
#include <iostream>
using namespace std;
#include <vector>

void printVector(vector<int> &v) {
	for ( vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
		cout << *it << " ";
	}
	cout << endl;
}

// 将上述函数改为函数模板效果
template<typename T>
void printVectorTemplator(vector<T>& v) {
	// 需要注意下一行代码，需要使用 typename 限定一下类型名
	// 如果不使用 typename 编译器会不知道 vector<T>::iterator 是类型名还是容器中的成员，会编译报错
	for (typename vector<T>::iterator it = v.begin(); it != v.end(); ++it) {
		cout << *it << '\t';
	}
	cout << endl;
}
// vector 容器 构造函数
void test01() {
	// 默认初始化vector对象，并赋值
	vector<int> v1; // 默认构造
	for (int i = 0; i != 10; ++i) {
		v1.push_back(i);
	}
	printVector(v1);

	vector<int> v2(v1.begin(), v1.end()); // 通过迭代器区间传值
	printVector(v2);

	vector<int> v3(10, 100); // 通过 n  个 elem
	printVector(v3);

	vector<int> v4(v3); // 通过拷贝构造函数
	printVector(v4);
}
int main() {
	test01();
	return 0;
}
```

## 3 向 vector 对象中添加元素

push_back()

将一个值 **压到(push)** 到 vector 的 **尾端(back)**

> [!question] 要不要给vector容器设置初始值？
> + 如果容器内所有值都一样时，可以在定义容器时设置；
> + 否则，更有效的方法是先**定义一个空的 vector 对象**，再在具体运行时添加具体值。

>[!warning] 范围for语句遍历时，不应该改变其遍历序列大小
