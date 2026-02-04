## 1  vector 容器概念

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

![image-20230930181910686](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程/img/image-20230930181910686.png)

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
**c++11新特性**：

1. `vector`声明写法：
   当`vector`内的对象为`vector`对象时
	+ 旧写法：`vector<vector<int> >`，右尖括号和元素对象类型之间必须要有一个空格
	+ 新写法：`vector<vector<int>>`

## 2  定义和初始化

### 2.1  概念

```C++
// 1 默认构造函数，创建一个指定的空vector对象
// v1 是一个空的 vector，潜在的元素是 T 类型的，执行默认初始化
vector<T> v1;

// 2 拷贝构造
vector<T> v2(v1);
vector<T> v2 = v1;

// 3 创建指定数量元素， n 个 elem，其中，v3的元素值为val；
vector<T> v3(n, val);

// 4 值初始化，只提供元素数量，不提供元素值，只支持直接初始化，v4的元素值为类型默认初始值
vector<T> v4(n);

// 5 列表初始化 C++11 新特性
// 提供初始元素值的列表，只能列表初始化，包含初始值个数的元素，每个元素被赋予相应初始值
vector<T> v5{a,b,c,d,...};
vector<T> v5 = {a,b,c,d,...};
// vector<T> v5(a,b,c,d,...); // wrong!

// 6 拷贝构造的另一种形式，根据迭代器指定了范围
vector<T> v6(v5.beigin(), v5.end());
```

+ 拷贝构造时，俩个 vector **容器内元素数据类型必须一致**
  + 提供初始元素值的列表，**只能**将初始值放到花括号内进行**列表初始化**，而不能放在圆括号内
+ 使用`=`进行初始化，实际上执行的是拷贝初始化，把等号右边的拷贝到新创建的对象中
	+ 本质上，是因为 `vector` 容器提供了 `=` 运算符的重写
+ vector初始化主要分为使用**花括号**和**圆括号**两种。
	+ 花括号：表示想要列表初始化，也就是为新创建的对象提供初始值

> 如果花括号提供的数值无法作为列表初始化值，则会将这些数值作为构造函数参数使用

   3. 圆括号：表示想要提供构造参数

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

### 2.2  代码

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

## 3  向 vector 对象中添加元素

push_back()

将一个值 **压到(push)** 到 vector 的 **尾端(back)**

> [!question] 要不要给vector容器设置初始值？
> + 如果容器内所有值都一样时，可以在定义容器时设置；
> + 否则，更有效的方法是先**定义一个空的 vector 对象**，再在具体运行时添加具体值。

>[!warning] 范围for语句遍历时，不应该改变其遍历序列大小

### 3.1  vector 容器

![image-20231004124656135](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231004124656135.png)

1. 和数组十分相似，被称为**单端数组**
   2. 单端数组的原因是：vector只能在数组末端进行插入删除的操作
3. 可以**动态扩展**
4. 其迭代器是支持随机访问的迭代器

> vector 容器的动态扩展：
>
> 1. 并不是在原空间后接入新空间，而是，重新寻找一个更大的内存空间，将原内存中的数据复制到新内存空间中，并释放原空间
>
> 2. **找到的新空间的内存大小往往是原空间的 `2` 倍**

#### 3.1.1  构造函数

四种构造方法：

+ 默认无参构造
+ 区间构造
  + `vector(v.begin(), v.end())`
  + 前闭后开，`[v.begin(), v.end())`
+ n个elem
+ 拷贝构造

![image-20231004124816778](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231004124816778.png)

```c++
#include <string>
#include <iostream>
using namespace std;
#include <vector>

template <typename T>
void printVector(vector<T> v){
    for(typename vector<T>::iterator it = v.begin(); it!=v.end(); ++it){
        cout << *it << "\t";
    }
    cout << endl;
}

int main(){

    // 通过默认构造函数
    vector<int> v1;
    for(int i = 0; i < 10; ++i){
        v1.push_back(i);
    }
    printVector(v1);

    // 通过区间
    vector<int> v2(v1.begin(), v1.end());
    printVector(v2);

    // n 个 elem
    vector<int> v3(10, 100);
    printVector(v3);

    // 拷贝构造函数
    vector<int> v4(v3);
    printVector(v4);
    
    return 0;
}
```

```c++  
for(typename vector<T>::iterator it = v.begin(); it!=v.end(); ++it)
```

> 注意上一行代码，其中**关键字 `typename` 是必不可少的**
>
> **原因**：
>
> ​	`vector<T>::iterator` 编译器无法确定这个是类型还是容器成员变量，二义性错误
>
> **解决方案**：
>
> 1. 添加关键字 `typename` 告诉编译器，这个是类型
>
> 2. 将函数模板修改成普通函数
>
>    如下行代码，直接将`int`类型注入，这行代码是不会报错的

```c++
for(vector<int>::iterator it = v.begin(); it!=v.end(); ++it){
	//todo...
}
```

#### 3.1.2  赋值操作

两种方式：

1. 赋值运算符`=`重载

   `v2 = v1`

2. `assign`方法
   3. 通过区间——左闭右开

	  `v3.assign(InputIterator first, InputIterator last);`

   4. n 个 elem

	  `v4.assign(size_type n, const value_type &val);`

```c++
#include <string>
#include <iostream>
using namespace std;
#include <vector>

template <typename T>
void printVector(vector<T> v){
    for(typename vector<T>::iterator it = v.begin(); it!=v.end(); ++it){
        cout << *it << "\t";
    }
    cout << endl;
}

int main(){

    vector<int> v1;
    for(int i = 0; i < 10; ++i){
        v1.push_back(i);
    }
    printVector<int>(v1);

    // 1 通过赋值运算符重写
    vector<int> v2;
    v2 = v1;
    printVector<int>(v2);

    // 2 通过 assign 方法 
    
    // 2.1 区间
    vector<int> v3;
    v3.assign(v1.begin(), v1.end());
    printVector<int>(v3);

    // 2.2 n 个 elem
    vector<int> v4;
    v4.assign(10, 100);
    printVector<int>(v4);
    
    return 0;
}
```

#### 3.1.3  容量和大小

容量：容器所能容纳元素的个数

大小：容器中当前已有元素的个数

`vector` 容器提供了关于容器容量和大小的接口：

+ `ivec.empty()`
  + 判断容器是否为空
+ `ivec.capacity()`
  + 返回容器所能容纳的元素的个数
  + 同时也代表着当前容器所占内存空间的大小为 `ivec.capacity() * sizeof(type)`

	> 如果没有设置预分配空间大小，则下次预分配空间大小为 `ivec.capacity()`

+ `ivec.size()`
  + 返回容器中当前已有元素个数
  + `ivec.size() <= ivec.capacity()`

	> 关于容器收缩空间，则往往是指使容器容量收缩为当前容器所拥有元素大小，也即使 `ivec.capacity() == ivec.size()`
	>
	> 方法有多种，这里举出两种：
	>
	> 1. 使用`shrink_to_fit()` 函数，请求回收内存空间
	> 2. 使用 `sway()` 函数和匿名对象，进行内存空间的收缩

+ `resize(int num, elem=default)`
  + 重设容器所能容纳元素的大小

  	> 注意，`resize(num)` 函数是重新改变容器所能容纳元素的个数为 `num`，也是就说，对于不同大小的 `num`，将有不同的几种情况：(令当前的`ivec.size()` 为 `n`， 而当前的 `ivec.capacity` 为 `m`，`n<=m`)
  	>
  	> 1. `num < n`
  	> + `ivec.size() == num`，但`ivec.capacity()`大小不变
  	> + 相当于从当前容器`ivec`中截留了`num`个元素
  	> + 也就是， 保留了当前容器`ivec`中的`[0, num)`对应下标的元素，而将`[num, ivec.size())`对应下标的元素给删除掉了
  	> 2. `num == n`
  	> + `ivec.size()`和`ivec.capacity()`大小都不变
  	> + 相当于代码不生效
  	> 3. `n < num < m`
  	> + `ivec.size() == num`，但`ivec.capacity()`大小不变
  	> + 且，由于`n < num`，所以需要使用填充元素使当前已有元素个数达到`num`个
  	> 4. `num == m`
  	> + 同上，只不过将`size` 的大小，填充至和`capacity`的大小相等
  	> 5. `m < num `
  	> + 首先，`ivec`需要**重新分配内存空间**
  	> + 重新分配的内存空间大小将以`m`为基数，以`2`为系数倍增，直到`ivec.capacity() >= num`
  	> + 如果使用`reserve(int num1)`函数设置了预分配空间大小，则在分配内存空间时，将以`num1`和`m`中较大的值为基数
  	> + 其次，容器中将填充元素使`ivec.size() == num`

+ `reserve(int num)`
  + 重设容器预分配空间大小
  + 用于减少重新分配内存空间的次数

	> 有两种情况：
	>
	> 1. `num <= ivec.capacity()`
	> + 该函数不起任何作用
	> 2. `num > ivec.capacity()`
	> + `ivec`**重新分配内存空间**，以使得`ivec.capacity() == num`

+ `shrink_to_fit()`
  + 请求回收内存空间:，注意是请求，也就是说，这一条代码不一定生效

#### 3.1.4  数据访问

1. 使用容器**迭代器**
2. 使用 `at(int index)` 方法
3. 使用` []` 运算符
4. 特殊：
   5. 使用`front()` 方法获取容器中第一个元素
   6. 使用`back()` 方法获取容器中最后一个元素

#### 3.1.5  容器互换

`vector`提供`swap()`接口，使得两容器中的元素可以互换

##### 特殊用途：收缩内存

```c++
 vector<int>(v1).swap(v1);
 ```

> 上述代码为使用`sway()`进行内存收缩的核心代码，可以分解为如下几步
>
> 1. 使用拷贝构造创建匿名对象 `vector<int>(v1)`，假设匿名对象为 `v2`
>    1. `v2.capacity() == v1.size()`
> 2. 交换内存空间 `v2.sway(v1)`
>    1. `v1.capacity() == v2.capacity() == v1.size()`
> 3. 编译器释放匿名对象
>    1. 匿名对象在该行代码执行完毕后，会被编译器自动释放掉内存空间

测试完整代码：

```c++
#include <string>
#include <iostream>
using namespace std;
#include <vector>

template <typename T>
void printVector(vector<T> v){
    for(typename vector<T>::iterator it = v.begin(); it!=v.end(); ++it){
        cout << *it << "\t";
    }
    cout << endl;
}

int main(){

    vector<int> v1;
    for(int i = 0; i < 10; ++i){
        v1.push_back(i);
    }
    printVector<int>(v1);
    cout << v1.capacity() << endl; // 16
    cout << v1.size() << endl; // 10
    cout << &v1[0]<<endl;
    cout << endl;

    v1.reserve(20);

    printVector<int>(v1);
    cout << v1.capacity() << endl; // 20
    cout << v1.size() << endl; // 10
    cout << &v1[0]<<endl;
    cout << endl;

    // 使用 shrink_to_fit() 进行内存的收缩
    // v1.shrink_to_fit();

    // printVector<int>(v1);
    // cout << v1.capacity() << endl;
    // cout << v1.size() << endl;
    // cout << &v1[0]<<endl;
    // cout << endl;

    // 使用 sway 进行内存收缩
    vector<int>(v1).swap(v1);

    printVector<int>(v1);
    cout << v1.capacity() << endl; // 10
    cout << v1.size() << endl; // 10
    cout << &v1[0]<<endl;
    cout << endl;
    
    return 0;
}
```

#### 3.1.6  预留空间

使用`reserve()`函数，设置容器预留空间

> **如何统计`vector`容器扩展的次数?**
>
> 根据容器首地址改变的次数来进行统计

 ```c++
 #include <string>
 #include <iostream>
 using namespace std;
 #include <vector>
 
 int main(){
 
     vector<int> v1;
     int *p = NULL;
     int num = 0;
     for(int i = 0; i < 10000; ++i){
         v1.push_back(i);
         if(p != &v1[0]){
             p = &v1[0];
             ++num;
         }
     }
     cout << num << endl; // 15
     return 0;
 }
```

## 4  添加元素——`push_back()`

作用：将一个值当作`vector`对象的尾元素，压到(push)`vector`对象的尾部(back)。

语法：

```c++
vector<int> v1;
v1.push_back(1);
```

## 5  Vector 内存空间相关规律

>[!note] 建议！
>除非要创建的`vector`对象中的所有元素值都一样，否则在创建时不应该预设`vector`的大小

### 5.1  重新分配内存空间的情况

+ `vector`容器只会在**执行`insert`操作，`size`和`capacity`相等时**，或者**调用`resize`和`reserve`时给定大小超过当前的`capacity`**这两情况时，才可能重新分配内存空间
	+ 也就是，当需求大于当前容量时，vector 容器才会重新分配内存空间
	+ 只有迫不得已的时候才可以分配新的内存空间

### 5.2  vector 的内存空间容量如何增加

+ vector 每次分配内存空间时，总是将当前容量翻倍
+ 因此，标准库实现中`vector`增长的具体情况有两种
	1. 默认的内存分配机制：０　１　２　４　８
	2. 通过`reserve(n)`手动修改预分配内存空间

$$
         \text{每次真实新增空间的大小}　>= \max(ivec.capacity(), n)
$$

## 6  vector 关于分配内存空间的几个方法

### 6.1  `size()`和`capacity()`方法

#### 6.1.1  `size()`方法

作用：查询容器中已有元素个数

#### 6.1.2  `capacity()`方法

作用：查询容器当前内存所能容纳的元素个数

二者区别：

对于`vector`容器`ivec`而言，$ivec.capacity()　= ivec.size() + 预留空间$，其中预留空间大于等于0

当容器的`capacity`等于它的`size`时，如果再执行`insert`操作，则需要重新为这个容器分配内存空间，之后，`capacity`又将大于`size`

所以，一个容器的`capacity`一定大于等于它的`size`

### 6.2  `resize()`和`reserve()`方法

#### 6.2.1  resize(n)

作用：重新设置容器可以容纳元素的大小为`n`

函数原型

```C++
resize(n)
```

#### 6.2.2  `reserve(n)`方法

作用：修改容器预分配空间大小**至少**为`n`个元素大小

+ 这两个方法都不回收容器所占内存空间

### 6.3  `shrink_to_fit()`方法

作用：请求回收预留空间

+ 该函数只是向内核建议回收，因此，其不一定生效
+ 如果生效，那么，vector 容器所占用的内存空间将缩小为容器中当前所有元素的容量，也就是，`ivec.capacity() == ivec.size()`

## 7  vector 容器其余操作

1　删除操作
