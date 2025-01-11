## 1. 模板

### 1.1. 概念

编程思想：==泛型编程==

模板是一个==通用摸具==，为了提高复用

C++中模板有两种：

+ 函数模板
+ 类模板

> + 模板是一个框架，不能直接使用
> + 模板通用性很高，但并非万能

### 1.2. 函数模板

建立一个通用函数，函数的返回值类型和参数类型==可以先不具体指定==，用一个==虚拟类型==代替

#### 1.2.1. 语法

```c++
template <typename T>
函数的声明或定义
```

+ `template`：模板关键字，告诉编译器，下面是定义的模板
+ `typename T`：虚拟的数据类型，也就是通用数据类型，在对函数模板使用时，会使用具体数据类型实现该虚拟数据类型
  + `typename` 定义 `T` 为虚拟数据类型的关键字，也可以使用 `class`
  + `T`只是名字，可以替换成任意的合法命名

#### 1.2.2. 使用方法

1. 自动推导

   使用定义的模板函数时，直接输入参数对函数进行调用，编译器会==根据输入的参数自动推导虚拟数据类型`T`的实现==

   + 比如下面代码中 `a` 和 `b` 的类型会自动推导出为 `int` 类型

2. 显式指定类型

   + 语法：`函数模板名<具体数据类型>(参数列表)`
	 + 尖括号中的**具体数据类型**会指定函数模板定义中的 `T` 为该数据类型

**代码**：

```c++
#include <iostream>
using namespace std;

template <typename T>
void mySwap(T &a, T &b){
    T temp = a;
    a = b;
    b = temp;
}

int main(){
    int a = 2, b = 20;
    // 自动类型推导
    swap(a, b);
    printf("%d\n",a);
    printf("%d", b);
    // 显式指定类型
    swap<int>(a, b);
    printf("%d\n",a);
    printf("%d", b);
    return 0;
}
```

#### 1.2.3. 注意事项

1. 函数模板在调用时，在使用自动类型推导时，需要保持 ==`T` 的数据类型一致性==
   1. 所以，在通过**自动类型推导**来调用函数模板时，和普通函数不一样的是，是不会发生**隐式数据类型转换**的
2. 函数模板在调用时，一定要==对 `T` 指定出具体的数据类型==
   1. 如果是自动类型推导，就需要保持 `T` 的数据类型的一致性
   1. 可以使用显式指定类型
3. 当函数模板中使用迭代器并声明迭代器对象时，由于编译器不知道对象名前面的是类型名还是迭代器容器中的属性，导致二义性错误而报错，比如：
   1. ```c++
      template<typename T>
      void printVectorTemplator(vector<T>& v) {
      	for ( vector<T>::iterator it = v.begin(); it != v.end(); ++it) { 
      		cout << *it << '\t';
      	}
      	cout << endl;
      }
      ```
	  1. 报错行：`for ( vector<T>::iterator it = v.begin(); it != v.end(); ++it) { `
	  2. 报错原因：编译器不知道 `vector<T>::iterator` 是类型名还是容器中的成员
	  3. 解决方法：使用`typename` 对该类型名进行限定

		 将源代码改为`for (typename vector<T>::iterator it = v.begin(); it != v.end(); ++it) {`

#### 1.2.4. 函数模板案例 - 不同数据类型数组排序

1. 利用函数模板封装一个排序的函数，可以对**不同数据类型**数组进行排序
2. 排序规则从大到小，排序算法为**选择排序**
3. 分别利用char数组和int数组进行测试

```c++
#include <iostream>
#include <utility>
using namespace std;

template <typename T>
void m_swap(T &a, T &b){
    T temp = a;
    a = b;
    b = temp;
}

template <typename T>
void m_sort(T a, int length){

    for(int i = 0; i!= length; ++i){
        int min = i;
        for(int j = i+1; j!= length; ++j){
            if(a[j] < a[min]) min = j;
        }
        m_swap(a[i], a[min]);
    }

}
int main(){
    int a[4] = {1,3,4,2};
    char b[4] = {'1','2', '3', '4'};
    
    // 自动类型推导
    m_sort(a, 4);
    for(int i= 0; i < 4; ++i){
        cout << a[i] << endl;
    }
    
	// 自动类型推导
    m_sort(b, 4);
    for(int i= 0; i < 4; ++i){
        cout << b[i] << endl;
    }
    return 0;
}
```

#### 1.2.5. 普通函数与函数模板的调用规则

1. 当普通函数的函数名与函数模板中的函数名相同时
   1. ==代价最小原则==：
	  1. **如果参数的数据类型符合普通函数的参数列表，优先调用普通函数**
	  2. **否则，优先调用函数模板**
   2. 显式调用模板时，可以通过==空模板参数列表 `<>`== ，告诉编译器此处调用的是函数模板
2. 函数模板也可以发生==函数重载==

> 如果已经提供函数模板，建议不要再一个写同名的普通函数，容易出现二义性

#### 1.2.6. 函数模板的局限性

模板中的 `T` 虽然是通用数据类型，但是如果调用时，传入的数据类型是特殊的数据类型，比如自定义的数据类型，就会导致虽然数据类型可以成功传入，但是在程序运行时，运算符会限制住对传入数据类型的操作，比如两个类对象做 `a == b` 的计算，编译器就会不知道应该如何计算，会照成编译报错。

解决方法：

1. 对运算符的操作
   1. **对自定义的数据类型或者类做运算符重载**
2. 对模板的操作
   1. **对具体数据类型做具体化的实现**

	  语法：

	  ```c++
      template<typename T1, ...>
      bool 函数名(T1 a, T2 b, ...){
          ...
      }
      ```

对具体数据类型做具体化的实现：

```c++
#include <iostream>
using namespace std;

class Person{
public:
    Person(int age, string name){
        m_age = age;
        m_name = name;
    }

    int m_age;
    string m_name;
};


template <typename T>
bool m_equal(T a, T b){
    if(a == b) return 1;
    else return 0;
}

template<>bool m_equal(Person a, Person b){
    if(a.m_age == b.m_age)return 1;
    else return 0;
}



void test(){
    int a = 0, b = 1;
    cout << m_equal(a, b) << endl;

    Person p(18, "张三");
    Person p1(20, "里斯");
    cout << m_equal(p, p1);
}

int main(){
    test();

    return 0;
}
```

### 1.3. 类模板

#### 1.3.1. 语法

```c++
template <typename T>
类
```

> 定义时语法和函数模板没啥区别

#### 1.3.2. 类模板使用时和函数模板的区别

1. 类模板没有自动类型推导，必须显式指定
2. 类模板的模板参数列表可以有默认值

#### 1.3.3. 类模板的成员函数

+ 普通类的成员函数在定义时创建
+ ==类模板的成员函数在**调用时**创建==

#### 1.3.4. 类模板对象做函数的参数

当类模板对象作为函数参数时，需要在传入参数时确定类模板对象中的成员属性的数据类型

有几种方式在传入参数时确定类模板对象成员属性的数据类型：

1. 直接将具体数据类型传入
2. 间接将具体数据类型传入——将具有类模板对象参数的函数转化为函数模板
   1. 将成员属性的数据类型作为模板参数
   2. 将对象对应的类作为模板参数

```c++
#include <iostream>
#include <string>
using namespace std;

template <class T1, class T2>
class Person{
public:
    Person(T1 age, T2 name){
        m_age = age;
        m_name = name;
    }

    T1 m_age;
    T2 m_name;
};

// 1. 指定参数类型传入
void printP1(Person<int, string> &p){
    cout << p.m_age << " : "  << typeid(p.m_age).name() << endl;
    cout <<  p.m_name << " : " << typeid(p.m_name).name() << endl;
}


void test1(){
    Person<int, string> p(18, "张三");
    printP1(p);
}

// 2. 函数模板化，传入的类的参数作为模板参数
template <typename T1, typename T2>
void printP2(Person<T1, T2> p){
    cout << p.m_age << " : "  << typeid(p.m_age).name() << endl;
    cout <<  p.m_name << " : " << typeid(p.m_name).name() << endl;
}

void test2(){
    Person<int, string> p(18, "张三");
    printP2(p);
}

// 3. 函数模板化，传入的类作为模板参数
template <class T>
void printP3(T p){
    cout << p.m_age << " : "  << typeid(p.m_age).name() << endl;
    cout <<  p.m_name << " : " << typeid(p.m_name).name() << endl;
}

void test3(){
    Person<int, string> p(18, "张三");
    printP3(p);
}

int main(){
    // test1();
    // test2();
    test3();

    return 0;
}
```

#### 1.3.5. 类模板的继承

和上述情况一样，同样需要指定父类中成员属性的具体数据类型

> 原因：类模板中成员属性的数据类型没有被指定，编译器无法为继承该类的子类分配内存

解决方法也同上：

1. 在继承时直接指定父类模板参数的具体类型
   1. 相当于在继承时将父类模板退化为普通的类
2. 将子类转化为模板，将父类的模板参数传递给子类

```c++
#include <iostream>
#include <string>
using namespace std;


template <class T1, class T2>
class Person{
public:
    Person(T1 age, T2 name){
        m_age = age;
        m_name = name;
    }

    T1 m_age;
    T2 m_name;
};

// 1. 指定参数类型传入
class SonOfPerson:public Person<int, string>{

};

// 2. 将子类转化为类模板
template <class T1, class T2, class T3>
class SonOfPerson2 : public Person<T1, T2>{
    public:
	T3 m_address;
};
/*
	可以看出上述代码中 T1, T2 是父类的模板参数传递给子类
	T3 是子类新加入的模板参数
*/


int main(){

    return 0;
}
```

#### 1.3.6. 类模板成员函数的类外实现

代码：

```c++
#include <iostream>
#include <string>
using namespace std;


template <class T1, class T2>
class Person{
public:
    Person(T1 age, T2 name);

    void printP();

    T1 m_age;
    T2 m_name;
};

// 1. 将成员函数转化为函数模板
template <class T1, class T2>
Person<T1, T2>::Person(T1 age, T2 name){ // 2. 添加类作用域，而且在类作用域后添加上参数列表
    this->m_age = age;
    this->m_name = name;
}

template <class T1, class T2>
void Person<T1, T2>::printP(){
    cout << "姓名" << this->m_name << endl;
    cout << "年龄" << this->m_age << endl;
}

void test(){
    Person<int, string> p(19, "张三");
    p.printP();
}

int main(){
    test();

    return 0;
}
```

> 实现重点：
>
> 1. 模板：
>    1. 类外实现类模板内的成员函数，首先需要==将成员函数转化为函数模板==
> 2. 作用域：
>    1. 类外实现类内成员函数需要==加上类的作用域==，指定这个函数是某个类内的成员函数
>    2. 当类是类模板时，需要在添加作用域时，==添加上模板参数==，指定这个函数所在的类是类模板
> 	  1. 参数列表和类模板的参数列表相同

#### 1.3.7. 类模板分文件编写

问题：==类模板中的成员函数在调用时创建，在文件链接时，链接不到==

解决方法：

1. 不再引入头文件，而是**直接引入实现了头文件的源文件** `*.cpp*`
2. **将头文件和实现写入到同一个文件当中**，并将文件后缀写为`*.hpp*`
   1. 这个后缀只是一种约定，写成`*.cpp`也无所谓，但写成这样是为了告诉浏览器，这个是关于类模板的一个定义实现文件

##### 第一种解决方法代码

`Person.c`

```c++
#include <iostream>
#include <string>
using namespace std;

// Person 类模板定义
template <class T1, class T2>
class Person{
public:
    Person(T1 age, T2 name);

    void printP();

    T1 m_age;
    T2 m_name;
};
```

`Person.cpp`

```c++
#pragma once 
#include "Person.h"


// Person 类模板的类外实现
template <class T1, class T2>
Person<T1, T2>::Person(T1 age, T2 name){
    this->m_age = age;
    this->m_name = name;
}

template <class T1, class T2>
void Person<T1, T2>::printP(){
    cout << "姓名" << this->m_name << endl;
    cout << "年龄" << this->m_age << endl;
}

```

`main.cpp`

```c++
#include <iostream>
#include <string>
using namespace std;
#include "Person.cpp" // 直接引入实现后的源文件

void test(){
    Person<int, string> p(19, "张三");
    p.printP();
}

int main(){
    test();

    return 0;
}
```

##### 第二种解决方法代码

`Person.hpp`

将类模板定义和类模板的实现放到同一个`hpp`文件中

```c++
#include <iostream>
#include <string>
using namespace std;

// Person 类模板定义
template <class T1, class T2>
class Person{
public:
    Person(T1 age, T2 name);

    void printP();

    T1 m_age;
    T2 m_name;
};

// Person 类模板的类外实现
template <class T1, class T2>
Person<T1, T2>::Person(T1 age, T2 name){
    this->m_age = age;
    this->m_name = name;
}

template <class T1, class T2>
void Person<T1, T2>::printP(){
    cout << "姓名" << this->m_name << endl;
    cout << "年龄" << this->m_age << endl;
}
```

`main.cpp`

```c++
#include <iostream>
#include <string>
using namespace std;
#include "Person.hpp" // 引入 hpp 文件

void test(){
    Person<int, string> p(19, "张三");
    p.printP();
}

int main(){
    test();

    return 0;
}
```

#### 1.3.8. 类模板与友元函数

> 首先，回忆一下**友元函数**：
>
> ​		在类的开始，使用`friend`关键字声明的函数，是该类的友元函数，这个函数可以访问该类中的私有属性
>
>
>
> 然后，回忆一下友元函数实现的细节：
>
> 1. 在定义友元函数时，首先需要在类中声明该友元函数，也就是说，该函数的实现应先于该类的实现
>    1. 全局函数之所以不在意这个，是因为全局函数存储在全局区，是在函数编译时，首先被编译的
> 2. 友元函数需要访问该类，也就是说，友元函数的实现应该在类的声明之后

类模板使用友元函数又根据友元函数的实现方式分为两种：

1. 友元函数在类模板类内实现
2. 友元函数在类模板类外实现

##### 第一种：友元函数类内实现

这种情况就很简单，直接写就OK

```c++
#include <string>
#include <iostream>
using namespace std;

// Person 类模板定义
template <class T1, class T2>
class Person{
    // 友元函数应该是个函数模板，因为Person 类是一个类模板，如果友元函数不是函数模板，则无法与类模板传入的参数类型保持一致
    friend void printPerson(Person<T1, T2> &p){
    cout << p.m_age << endl;
    cout << p.m_name << endl;
}
public:
    Person(T1 age, T2 name);

    void printP();
private:
    T1 m_age;
    T2 m_name;
};

// Person 类模板的类外实现
template <class T1, class T2>
Person<T1, T2>::Person(T1 age, T2 name){
    this->m_age = age;
    this->m_name = name;
}


void test(){
    Person<int, string> p(19, "张三");
    printPerson(p);
}

int main(){
    test();

    return 0;
}
```

##### 第二种：友元函数类外实现

1. 类声明在最开始
2. 友元函数定义
3. 类定义，类内的友元函数的声明也在这一步写

或者更复杂一些：

1. 类声明
2. 友元函数的函数声明
3. 类定义，友元声明
4. 友元函数定义

```c++
#include <string>
#include <iostream>
using namespace std;

// 类模板声明
template <class T1, class T2>
class Person;

// 友元函数的类外实现
// 友元函数的声明应该在类模板完整定义之前
// 由于友元函数的参数为类模板对象，所以其声明也应该在类模板声明之后
template <class T1, class T2>
void printPerson(Person<T1, T2> &p){
    cout << p.m_age << endl;
    cout << p.m_name << endl;
}

// Person 类模板定义
template <class T1, class T2>
class Person{
    // 友元函数声明
    friend void printPerson<>(Person<T1, T2> &p);
public:
    Person(T1 age, T2 name);

    void printP();
private:
    T1 m_age;
    T2 m_name;
};

// Person 类模板的类外实现
template <class T1, class T2>
Person<T1, T2>::Person(T1 age, T2 name){
    this->m_age = age;
    this->m_name = name;
}


void test(){
    Person<int, string> p(19, "张三");
    printPerson(p);
}

int main(){
    test();

    return 0;
}
```

### 1.4. 案例 - 通用数组

`MyArray.hpp`

```c++
#pragma once
#include <string>
#include <iostream>
using namespace std;

/*
1. 构造函数和析构函数不是程序员能够主动调用的函数，而是在对象创建和生命周期结束时，会被编译器主动调用
2. 需要做什么功能时，需要提前将功能的流程给计划好

*/
template <class T>
class MyArray{
public:
    MyArray<T>(int num);// 构造函数
    MyArray<T>(MyArray<T> &array);
    ~MyArray<T>();
public:
    // 尾插法
    void RearInsert(T value);
    // 尾删法
    void RearDelete();
    // 重写 = 运算符，防止浅拷贝问题
    MyArray<T>& operator=(const MyArray<T> &array);
    // 重写 [] 运算符，根据下标获取数据
    T& operator[](int num);

    // 获取当前元素个数
    int getCounts();
    // 获取当前数组长度;
    int getLength();
private:
    T* m_value;// 数组
    int m_length;// 数组长度
    int m_counts;// 当前元素个数
};

// 有参构造
template <class T>
MyArray<T>::MyArray(int num){
    cout << "调用有参构造" << endl;
    this->m_value = new T[num];
    this->m_length = num;
    this->m_counts = 0;
}
// 拷贝构造
template <class T>
MyArray<T>::MyArray(MyArray &array){
    cout << "调用拷贝构造" << endl;
    this->m_length = array.m_length;
    this->m_value = new T[array.m_length];
    for(int i = 0; i != array.m_counts; ++i){
        this->m_value[i] = array.m_value[i];
    }
    this->m_counts = array.m_counts;
    
}

// 尾插法
template <class T>
void MyArray<T>::RearInsert(T value){
    if (this->m_counts >= this->m_length) {
        return;
    }else {
        this->m_value[this->m_counts++] = value;
    }
}

// 尾删法
template <class T>
void MyArray<T>::RearDelete(){
    if(this->m_counts == 0){
        cout << "数组中没有数据, 删除失败" << endl;
        return;
    }
    --this->m_counts;
}

// 重写 = 运算符，防止浅拷贝
template <class T>
MyArray<T>& MyArray<T>::operator=(const MyArray<T> &array){
    // 判断当前类是否有数据
    if(this->m_counts != 0){
        // 初始化
        delete [] this->m_value;
        this->m_value = NULL;
        this->m_counts = 0;
         this->m_length = 0;
    }

    this->m_length = array.m_length;
    this->m_value = new T[this->m_length];
    for(int i = 0; i != array.m_counts; ++i){
        this->m_value[i] = array.m_value[i];
    }
    this->m_counts = array.m_counts;
    return *this;
}

// 获取当前元素个数
template <class T>
inline int MyArray<T>::getCounts(){
    return this->m_counts;
}

// 获取当前数组长度
template <class T>
inline int MyArray<T>::getLength(){
    return this->m_length;
}

    // 析构
    template <class T>
    MyArray<T>::~MyArray<T>(){
        if(this->m_value!=NULL){
            for(int i = 0; i != m_counts; ++i){
                delete this->m_value[i];
            }
        }
        delete [] this->m_value;
        this->m_value = NULL;
    }

template <class T>
T& MyArray<T>::operator[](int num){
    return this->m_value[num];
}
```

`main.cpp`

```c++
#include <string>
#include <iostream>
#include "MyArray.hpp"
using namespace std;

class Person{
    
    public:
    Person(int age, string name);
    int age;
    string name;
};

Person::Person(int age, string name){
    this->age = age;
    this->name = name;
}

int main(){
    Person* p1 = new Person(18, "张三");
    Person* p2 = new Person(19, "张三哥哥");
    Person* p3 = new Person(20, "藏三");

    MyArray<Person*> array(3);
    array.RearInsert(p1);
    array.RearInsert(p2);
    cout << array[1]->age << endl;
    return 0;
}
```

## 2. STL 初识

### 2.1. STL 基础

#### 2.1.1. STL概念

**STL(Standard Template Library)** 标准模板库

大概组成：

1. **容器（container）**
2. **算法(algorithm)**
3. **迭代器(iterator)**

**容器**和**算法**之间用**迭代器**进行连接

#### 2.1.2. 六大组件

1 容器：各种数据结构

2 算法：常用算法

3 迭代器：容器算法之间的胶合剂

4 仿函数：行为类似函数，作为算法的某种策略

​	重载 () 运算符后，使得类的行为类似于函数

5 适配器：用来修饰容器、仿函数或迭代器接口

6 空间配置器：对空间的配置和管理

#### 2.1.3. 容器

常用的数据结构：数组、树、链表、映射表、栈、队列、集合等

分类：

1. 关联式容器：二叉树结构，数据之间没有严格的物理关系
2. 序列式容器：强调数据的顺序，容器中每一个值都有**固定**的位置

#### 2.1.4. 算法

分类：

1. 质变算法：涉及到对数据修改操作的算法
2. 非质变算法：查找，排序等不涉及对数据修改操作的算法

#### 2.1.5. 迭代器

容器和算法之间的桥梁，用于访问容器中的数据且不暴露容器的内部表示

每一种容器都有其专有的迭代器

使用方式类似于指针

常用迭代器有五种：

1. 输入迭代器
2. 输出迭代器
3. 正向迭代器：
   1. 只支持向前遍历，也就是`++`操作，也支持`*p`
	  1. 也就是上文说的，使用方式类似于指针，甚至可以直接当作指针来看待
   2. 在正向迭代器之间支持`==` 与 `!=` 逻辑运算
4. 双向迭代器
   1. 拥有正向迭代器的全部功能，而且支持向后遍历，也就是`--`操作
5. 随机访问迭代器
   1. 支持双向迭代器的全部功能的同时，支持随机访问，也就是`p+i`、`p-i`的操作
   2. 在随机访问迭代器之间支持`>`、`>=`、`<`、`<=`、`"=="`、`"!="`逻辑运算

### 2.2. String 容器

> 与 C 语言中的字符串 `char*` 的区别：
>
> + C 语言中的 `char*` 本质上是一个字符数组，或者说是一个指针
>
> + string 的本质是一个类，类中维护着 `char*`，是一个`char*`的容器
> + string相当于 const char[]

#### 2.2.1. 构造函数

1. 无参构造`string();`
   1. 返回一个空字符对象
2. 有参构造`string(const char* s);`
   1. 参数为字符串`s`
3. 拷贝构造，`string(const string& s);`
4. 有参构造，`string(int n, char c)`
   1. 使用 n 个字符 `c`组成字符串

#### 2.2.2. 赋值

![image-20230927164408819](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230927164408819.png)

> 前三个是对运算符 `=` 的重写，后四个是成员函数
>
> 这些函数的作用是为了防止浅拷贝

#### 2.2.3. `string` 字符串拼接

在字符串的末尾追加字符

![image-20230927164810903](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230927164810903.png)

#### 2.2.4. `string` 字符串的查找和替换

![image-20230927164921024](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230927164921024.png)

find

`int position = str1.find(str2, pos=0, )`

在 `str1` 中查询 `str2` 的起始位置下标，如果不存在返回 `-1`

`rfind` 是从右往左找—— `rfind` —— `reverse find`

replace

`int str2 = str2.replace(pos, n, str1)`

`str2` 中的字符，从`pos` 开始的 `n` 个字符替换成`str1`

1. 替换前和替换后 `str2` 的地址保持不变，也就是说，替换的操作是在 `str2` 本身上进行的。
2. 替换操作并不是按字符单位来进行的，而是子串替换
   1. 也就是说，无论 `n`多大，都会将`str1`替换掉`[pos, pos+n-1]`的字符

#### 2.2.5. 字符串的比较

![image-20230930112354240](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230930112354240.png)

比较方式：==根据字符串的 ASCII 值进行比较==

`str1.compare(str2)`

+ `0`：当 `str1 == str2` 返回值为 `0`
+ `1` ：当 `str1 > str2` 返回值为 `1`
+ `-1`：当`str1 < str2` 返回值为 `-1`

#### 2.2.6. 字符存取

从 `string` 中访问单个字符的方式有两种：

1. 通过 `[]` 运算符：`char& oprator[](int n);`
2. 通过`at` 方法：`char& at(int n);`

> 也能通过上述两种方式对字符串进行修改

#### 2.2.7. 字符串插入与删除

![image-20230930112754925](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230930112754925.png)

#### 2.2.8. string 字串

从 string 对象中获取字符串子串

![image-20230930112951981](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230930112951981.png)

### 2.3. vector容器

[[STL - vector容器]]

### 2.4. deque容器

#### 2.4.1. 介绍

##### 功能

双端数组，可以对头部进行插入删除操作，支持随机访问

##### 与vector容器的区别

+ `vector`容器对于头部的插入删除效率低，数据量越大效率越低
+ `deque`相对而言，对于头部插入删除速度比`vector`快
+ `vector`访问元素时的速度比`vector`快，这个与二者的内部实现有关

![[双端队列_deque|1000]]

##### 内部机制

deque内部有个**中控器**，维护每段缓冲区中的内容，缓冲区中存放真实数据

中控器维护每个缓冲区的地址，使其像一片连续的内存空间

#### 2.4.2. deque构造函数

##### 函数原型

+ `deque<T>` //默认构造
+ `deque(beg, end);` //构造时，并将\[beg, end)区间内元素复制到本身
+ `deque(n, elem);` //构造时，并将n个elem区间内元素复制到本身
+ `deque(const deque &deq);` // 拷贝构造

##### 代码

```C++
#include<iostream>
#include<deque>
using namespace std;

// deque
void printDeque(const deque<int> &deq){
    for(deque<int>::const_iterator it = deq.begin(); it!=deq.end(); ++it){ // 迭代器的类型为 const_iterator
        cout << *it << " ";
    }
    cout << endl;
}
void test01(){
    deque<int> d1; // 默认构造
    for(int i = 0; i < 10; i++){
        d1.push_back(i);
    }

    printDeque(d1);
    deque<int> d2(d1.begin(), d1.end()); // 迭代器赋值
    printDeque(d2);
    deque<int> d3(10,100); // n elem 赋值
    printDeque(d3);
    deque<int> d4(d1); // 拷贝构造
    printDeque(d4);
}

int main(){
    test01();
    return 0;
}
```

#### 2.4.3. 赋值方式

##### 函数原型

+ `deque& operator=(const deque& deq);` // 重载等号操作符
+ `assign(beg, end)` // 将\[beg, end)区间内元素复制到本身
+ `assign(n, elem)`// 将n个elem复制到本身

##### 代码

```C++
#include<iostream>
#include<deque>
using namespace std;

// deque 赋值操作
void printDeque(const deque<int> &deq){
    for(deque<int>::const_iterator it = deq.begin(); it!=deq.end(); ++it){ // 迭代器的类型为 const_iterator
        cout << *it << " ";
    }
    cout << endl;
}
void test01(){
    deque<int> d1; // 默认构造
    for(int i = 0; i < 10; i++){
        d1.push_back(i);
    }
    printDeque(d1);
    deque<int> d2;
    d2 = d1;
    printDeque(d2);
    deque<int> d3;
    d3.assign(d1.begin(), d1.end());
    printDeque(d3);
    deque<int> d4;
    d4.assign(4,5);
    printDeque(d4);
}

int main(){
    test01();
    return 0;
}
```

> 和 `vector`的赋值方式几乎一致

#### 2.4.4. deque大小操作

##### 函数原型

+ `deque.empty();`
+ `deque.size();`
+ `deque.resize(num);`// 重新指定容器长度为`num`，如果容器变长，则以默认值填充新位置；如果容器变短，且删除末尾超出的元素
+ `deque.resize(num, elem);`// 重新指定容器长度为`num`，如果容器变长，则以`elem`填充新位置；如果容器变短，且删除末尾超出的元素

#### 2.4.5. deque插入和删除

##### 函数原型

双端插入/删除

+ `push_back(elem);`//后插
+ `push_front(elem);`//前插
+ `pop_back();`
+ `pop_front();`

指定位置插入/删除

+ `insert(pos, elem);` // 在pos处插入elem，返回新数据的位置
+ `insert(pos, n, elem);`//在pos位置处插入n个elem，无返回值
+ `insert(pos, beg, end);`//在pos位置插入区间内数据，无返回值
+ `clear();`// 清空容器
+ `erase(beg, end);`// 删除区间内数据，返回下一个数据的位置
+ `erase(pos);`// 删除pos位置的数据，返回下一个数据的位置

#### 2.4.6. deque数据存取

##### 函数原型

+ `at(int index);` // 返回索引位置的数据
+ `operator[];`
+ `front();`
+ `back();`

#### 2.4.7. deque 排序

##### 算法

+ `sort(iterator beg, iterator end, _prod);`
[[#sort]]

### 2.5. stack 容器

#### 2.5.1. 介绍

##### 概念

**先进后出**的数据结构，只有一个出口

> 栈不允许有遍历行为，只有栈顶元素能被外界访问到

![[栈_stack|1000]]

#### 2.5.2. 常用接口

##### 构造

+ `stack<T> stk;`
+ `stack(const stack &stk);`

##### 赋值操作

+ `stack &operator=(const stack &stk);`

##### 数据存取

+ `push(elem);`
+ `pop();`
+ `top();`

##### 大小操作

+ `empty();`
+ `size();`

#### 2.5.3. 代码

```C++
#include<iostream>
#include<stack>
using namespace std;

// 栈 stack 
// 先进后出
void test01(){
    stack<int> s;

    s.push(10);
    s.push(20);
    s.push(30);
    s.push(40);

    cout<< "栈当前大小为：" << s.size() << endl;
    while(!s.empty()){
        cout <<"当前栈顶元素为："<< s.top()<< endl;
        s.pop();
    }
    cout<< "栈当前大小为：" << s.size() << endl;
}

int main(){
    test01();
    return 0;
}
```

### 2.6. queue容器

#### 2.6.1. 介绍

概念：队列，先进先出(First In First Out)

![[队列_queue|1000]]

> 尾端入队，头端出队，只有双端元素能被访问，**不允许遍历**

#### 2.6.2. 函数原型

##### 构造函数

+ `queue<T> que;`
+ `queue(const queue &que);`

##### 赋值操作

+ `queue& operator(const queue& que);`

##### 数据存取操作

+ `push(elem);`
+ `pop();`
+ `back();`
+ `front();`

##### 大小操作

+ `empty();`
+ `size();`

#### 2.6.3. 代码

```C++
#include<iostream>
#include<queue>
#include<string>
using namespace std;

// 队列 queue
// 先进先出
class Person{
    public:
    Person(string name, int age){
        this->m_name = name;
        this->m_age = age;
    }

    string m_name;
    int m_age;
};
ostream& operator<<(ostream& cout, const Person & p){
    cout << "姓名：" << p.m_name << "; 年龄："<<p.m_age << endl;
    return cout; // 链式编程
}
void test01(){
    queue<Person> q;

    Person p1("SWK", 500);
    Person p2("TS", 45);
    Person p3("ZBJ", 1000);
    Person p4("SS", 1500);

    q.push(p1);
    q.push(p2);
    q.push(p3);
    q.push(p4);

    while(!q.empty()){
        cout << q.front()<< "..." << q.back() << endl;
        q.pop();
    }
}

int main(){
    test01();
    return 0;
}
```

> [!note] 左移运算符的重载
> [[lesson05 - c++核心编程#2 左移运算符重载 <<]]

### 2.7. list 链表

#### 2.7.1. 介绍

##### 概念

list容器是双向循环链表

链表：将数据链式存储。在存储单元上非连续的，数据元素的逻辑顺序通过指针实现

由于list容器的存储方式并非是连续的内存空间，因此list的迭代器只支持**前移**和**后移**，属于**双向迭代器**

##### 优点

+ 动态存储，不会造成内存浪费和溢出
+ 链表执行插入和删除为`O(1)`级别，不需要移动大量元素

##### 缺点

+ 指针灵活，但是空间(指针域)和时间(遍历)耗费大

> [!note] vector 和 list 关于迭代器
> 在list容器中，插入删除操作不会造成原有迭代器失效
> 在vector容器中，插入和删除操作会造成原有迭代器失效

#### 2.7.2. 构造函数

##### 函数原型

+ `list<T> lst;`
+ `list(beg, end);`
+ `list(n, elem);`
+ `list(const list &lst);`

#### 2.7.3. 赋值和交换

##### 函数原型

+ `assign(beg, end);`
+ `assign(n, elem);`
+ `list & operator=(const list & lst);`
+ `swap(lst);`

#### 2.7.4. 大小操作

#### 2.7.5. 函数原型

+ `size();`
+ `empty();`
+ `resize(num);` //空余部分用默认值填充
+ `resize(num, elem);` // 空余部分用 elem填充

#### 2.7.6. 插入和删除

##### 函数原型

+ `push_back(elem);`
+ `pop_back();`
+ `push_front(elem);`
+ `pop_front();`
+ `insert(pos, elem);` //返回新元素位置
+ `insert(pos, n, elem);`//无返回
+ `insert(pos, beg, elem);`//无返回
+ `clear();`
+ `erase(beg, end);`//返回下一位置
+ `erase(pos);` // 删除pos位置上的元素，返回下一位置
+ `remove(elem);` // 删除容器中所有匹配的元素

#### 2.7.7. 数据存取

##### 函数原型

+ `front();`
+ `back();`

#### 2.7.8. 反转和排序

##### 函数原型

+ `reverse();`
+ `sort();` // list 容器内部成员算法

> [!danger]
> 所有不支持随机存取迭代器的容器都不能使用标准算法库内的算法
> 不支持随机存取迭代器的容器，内部会提供对应算法

#### 2.7.9. 排序案例

```C++
#include<iostream>
#include<list>
#include<string>
using namespace std;

// 链表 list
// 按照年龄升序，按照身高降序
class Person{
    public:
    Person(string name, int age, int height){
        this->m_name = name;
        this->m_age = age;
        this->m_height = height;
    }

    string m_name;
    int m_age;
    int m_height;
};
ostream& operator<<(ostream& cout, const Person & p){
    cout << "姓名：" << p.m_name << "; 年龄：" << p.m_age << "; 身高为" << p.m_height <<  endl;
    return cout; // 链式编程
}

//指定排序规则
class AgeS{
    public:
    bool operator()(const Person&p1, const Person& p2){
        if(p1.m_age == p2.m_age) {
            return p1.m_height > p2.m_height;
        }
        return p1.m_age < p2.m_age;
    }
};
void test01(){
    list<Person> l;

    Person p1("SWK", 500, 175);
    Person p2("TS", 45, 180);
    Person p3("ZBJ", 1000, 170);
    Person p4("SS", 1500, 190);
    Person p5("SWK2", 500, 173);
    Person p6("TS2", 45, 185);
    Person p7("ZBJ2", 1000, 123);
    Person p8("SS2", 1500, 141);
    Person p9("SWK3", 500, 1235);
    Person p10("TS3", 45, 1124);
    Person p11("ZBJ3", 1000, 124);
    Person p12("SS3", 1500, 1512);

    l.push_back(p1);
    l.push_back(p2);
    l.push_back(p3);
    l.push_back(p4);
    l.push_back(p5);
    l.push_back(p6);
    l.push_back(p7);
    l.push_back(p8);
    l.push_back(p9);
    l.push_back(p10);
    l.push_back(p11);
    l.push_back(p12);

    for(Person p:l){
        cout << p << " ";
    }
    cout << endl;

    l.sort(AgeS()); // 自定义类型对象，需要自定义排序器
    for(Person p:l){
        cout << p << " ";
    }
    cout << endl;
}

int main(){
    test01();
    return 0;
}
```

### 2.8. set/multiset容器

#### 2.8.1. 介绍

##### 概念

所有元素自动被排序

##### 本质

set/multiset属于**关联式容器**，底层使用**二叉树**实现(红黑树)

set容器和multset容器区别

+ set不允许有重复元素，因此不允许插入重复数值
+ multiset允许有重复元素

#### 2.8.2. 构造和赋值

##### 函数原型

构造

+ `set<T> st;`
+ `set(const set& st);`

赋值

+ `set& operator=(const set &st);`

#### 2.8.3. 大小和交换

##### 函数原型

+ `size();`
+ `empty();`
+ `swap(st);`

#### 2.8.4. 插入和删除

##### 函数原型

+ `pair<iterator, bool> ret= insert(elem);`
+ `clear();`
+ `erase(pos)`
+ `erase(beg, end);`
+ `erase(elem);`

> [!note]
> + 插入自动排序
> + 插入重复数值时会插入失败
> + set 插入时会返回`pair<iterator, bool>`，其中第二个值为是否插入成功
> + multiset 插入时会返回`iterator`

#### 2.8.5. 查找和统计

##### 函数原型

+ `find(key);` // 存在返回元素迭代器，否则返回结束迭代器；
+ `count(key);` // 统计元素个数

#### 2.8.6. set和multiset的区别

+ set不允许插入重复值，multiset可以
+ set 插入数据时会返回插入的结果，表示是否插入成功
+ multiset插入时不会检测数据，因此可以插入重复数据

#### 2.8.7. pair创建

函数原型

+ `pair<type, type> p(value1, value2);`
+ `pair<type, type> p = make_pair(value1, value2);`

数据获取

+ `first`
+ `seconde`

#### 2.8.8. 排序

##### 7.1 内置数据类型

set容器默认的规则为**从小到大**，可以通过**仿函数**修改排序规则

> [!note]
> 这里传入`list`的成员函数`sort()`的是仿函数，而非函数对象

###### 代码

```C++
#include<iostream>
#include<set>
#include<string>
using namespace std;

// set 内置数据类型 自定义排序

//从大到小
class MyCompara{
public:
    bool operator()(const int& v1,const int& v2)const{ // 需要const
        return v1 > v2;
    }
};
void test01(){
    set<int> st;

    st.insert(10);
    st.insert(40);
    st.insert(20);
    st.insert(50);
    st.insert(30);
    st.insert(80);

    for(set<int>::iterator it = st.begin(); it != st.end(); ++it){
        cout << *it << " ";
    }
    cout << endl;

    // 指定排序规则
    set<int, MyCompara> st2;

    st2.insert(10);
    st2.insert(40);
    st2.insert(20);
    st2.insert(50);
    st2.insert(30);
    st2.insert(80);

    for(set<int, MyCompara>::iterator it = st2.begin(); it != st2.end(); ++it){
        cout << *it << " ";
    }
    cout << endl;
}

int main(){
    test01();
    return 0;
}
```

##### 7.2 自定义数据类型

###### 概念

+ set容器插入自定义类型数据时，需要**自定义排序规则**，也就是谓词。否则，会报错。
+ 谓词重载的`()`运算符方法，需要是`const`方法，也就是`bool operator()(type &args) const {…}`

###### 代码

```C++
#include<iostream>
#include<set>
#include<string>
using namespace std;

// set 自定义类型数据插入
class Person{
    public:
    Person(string name, int age){
        this->m_name = name;
        this->m_age = age;
    }

    string m_name;
    int m_age;
};
ostream& operator<<(ostream& cout, const Person & p){
    cout << "姓名：" << p.m_name << "; 年龄：" << p.m_age  <<  endl;
    return cout; // 链式编程
}

//指定排序规则
class AgeS{
    public:
    bool operator()(const Person&p1, const Person& p2) const{// 需要const
        return p1.m_age < p2.m_age;
    }
};

void test01(){
    set<Person, AgeS> st;

    Person p1("LB", 24);
    Person p2("GY", 28);
    Person p3("ZF", 25);
    Person p4("ZY", 21);

    st.insert(p1);
    st.insert(p2);
    st.insert(p3);
    st.insert(p4);

    for(set<Person>::iterator it = st.begin(); it != st.end(); ++it){
        cout << *it << " ";
    }
    cout << endl;
}

int main(){
    test01();
    return 0;
}
```

### 2.9. map容器

#### 2.9.1. 介绍

##### 概念

+ map中的所有元素都是`pair`
+ `pair`中的`first`为key(键值)，`second`为value(值)
+ 所有元素根据元素的key自动排序

##### 本质

map/multimap属于**关联式容器**，底层结构是用**二叉树实现**

##### 优点

+ 可以根据key快速索引到value

##### map/multimap的区别

+ map不允许有重复的key
+ multimap允许有重复的key

#### 2.9.2. 构造和赋值

+ `map<T1, T2> mp;`
+ `map(const map &mp);`
+ `map & operator=(const map &mp);`

#### 2.9.3. 大小和交换

+ `size();`
+ `empty();`
+ `swap();` // 成员函数

#### 2.9.4. 插入和删除

+ `insert(elem);`
+ `clear();`
+ `erase(pos);`
+ `erase(beg, end);`
+ `erase(key);`

插入的四种方式

```C++
#include<iostream>
#include<map>
#include<string>
using namespace std;

// map insert

template<class T1>
class PrintMap{
    public:
    void operator()(const T1& m) const {
        for(typename T1::const_iterator it = m.begin(); it!= m.end(); ++it){
            cout << it->first << " : " << it->second << endl;
        }
        cout << endl;
    }
};
void test01(){
    map<int, int> mp;

    // 第一种
    mp.insert(pair<int ,int>(1, 10));

    // 第二种
    mp.insert(make_pair(2, 20)); 

    // 第三种
    mp.insert(map<int, int>::value_type(3, 30));

    // 第四种 使用 [] 插入
    mp[4] = 40;
    // 插入时，插入错误时，不会插入失败，而是会修改 key =i 对应的 value;
    // 索引时，当key = i 值不存在时，会自动插入mp[i] = 0;
    cout << mp[5] << endl;  // 

    PrintMap< map<int, int>> p;
    p(mp);
}

int main(){
    test01();
    return 0;
}
```

>[!note]
>+ map类型的数据，可以通过`[]`进行索引和插入
>+ 类似于数组索引的方法，key对应数组下标，value对应数组下标对应的值
>+ map不允许插入重复key，但当使用`[]`插入时，如果误插入重复key，不会插入失败，而是会被认为该操作是对该key对应的值的修改
>+ 当使用`[]`访问map中的元素时，如果对应的key不存在，则默认插入`(key, 0)`

#### 2.9.5. 查找和统计

+ `find(key);`查找key是否存在，存在则返回key元素迭代器，否则返回结束迭代器
+ `count(key);`统计key的个数

#### 2.9.6. 排序

##### 概念

+ 默认排序规则为从小到大排序
+ 可以通过仿函数修改排序规则
+ 对于自定义类型必须指定排序规则

##### 代码

```C++
#include<iostream>
#include<map>
#include<string>
using namespace std;

// map insert
// print
template<class T1>
class PrintMap{
    public:
    void operator()(const T1& m) const {
        for(typename T1::const_iterator it = m.begin(); it!= m.end(); ++it){
            cout << it->first << " : " << it->second << endl;
        }
        cout << endl;
    }
};
// 内置数据类型 排序
template<class T1>
class MyCompara{
    public:
    bool operator()(const T1& v1, const T1& v2) const { // const 
        return v1 > v2;
    }
};

// 自定义数据类型 排序
class Person{
    public:
    Person(string name, int age, int height){
        this->m_name = name;
        this->m_age = age;
        this->m_height = height;
    }

    string m_name;
    int m_age;
    int m_height;
};
ostream& operator<<(ostream& cout,const Person & p1){
    cout << p1.m_name << " : " << p1.m_age << " : " << p1.m_height;
    return cout ;
}
template<class T1>
class MyCompara2{
    public:
    bool operator()(const T1& v1, const T1& v2) const { // const 
        if(v1.m_age == v2.m_age){
            return v1.m_height > v2.m_height;
        }
        return v1.m_age > v2.m_age;
    }
};

// 内置数据类型
void test01(){
    map<int, int, MyCompara<int>> mp;

    // 第一种
    mp.insert(pair<int ,int>(1, 10));

    // 第二种
    mp.insert(make_pair(2, 20)); 

    // 第三种
    mp.insert(map<int, int>::value_type(3, 30));

    // 第四种
    mp[4] = 40;

    cout << mp[5] << endl;

    PrintMap< map<int, int, MyCompara<int>>> p;
    p(mp);
}

// 自定义数据类型
void test02(){
     map<Person, int, MyCompara2<Person>> mp;  // 必须改变排序规则

     Person p1("asdad", 12, 241);
     Person p2("a123ad", 13, 221);
     Person p3("a415dad", 14, 212);

     mp.insert(map<Person, int>::value_type(p1, 1));
     mp.insert(map<Person, int>::value_type(p2, 1));
     mp.insert(map<Person, int>::value_type(p3, 1));

     PrintMap<map<Person, int, MyCompara2<Person>>> p;
     p(mp);
}
int main(){
    test02();
    return 0;
```

### 2.10. 案例1 评委打分

#### 2.10.1. 代码

```C++
#include<iostream>
#include<deque>
#include<vector>
#include<algorithm>
#include<numeric>
using namespace std;

// 评委打分
void printDeque(const deque<int> &deq){
    for(deque<int>::const_iterator it = deq.begin(); it!=deq.end(); ++it){
        cout << *it << " ";
    }
    cout << endl;
}
void printVector(const vector<int> &vec){
    for(vector<int>::const_iterator it = vec.begin(); it!=vec.end(); ++it){
        cout << *it << " ";
    }
    cout << endl;
}
void test01(){
    vector<deque<int>> grade;
    vector<int> gradeAvg;
    for(int i = 0; i < 5; ++i){
        deque<int> deq;
        grade.push_back(deq);
        for(int j = 0; j < 10; ++j){
            grade[i].push_back((int)rand()%10);
        }
    }
    for(int i = 0; i < 5; ++i){
        sort(grade[i].begin(), grade[i].end());
        grade[i].pop_back();
        grade[i].pop_front();
        printDeque(grade[i]);
    }
    for(int i = 0; i < 5; ++i){
        int avg = accumulate(grade[i].begin(), grade[i].end(), 0)/8;
        gradeAvg.push_back(avg);
    }
    printVector(gradeAvg);
    
}

int main(){
    srand((unsigned int) time(NULL));
    test01();
    return 0;
}
```

### 2.11. 案例2 员工分组

#### 2.11.1. 代码

```C++
#include<iostream>
#include<map>
#include<string>
#include<vector>
using namespace std;

// print
template<class T1>
class PrintMap{
    public:
    void operator()(const T1& m) const {
        for(typename T1::const_iterator it = m.begin(); it!= m.end(); ++it){
            cout << it->first << " : " << it->second << endl;
        }
        cout << endl;
    }
};

// 自定义数据类型 排序
class Person{
    public:
    Person(string name){
        this->m_name = name;
    }

    string m_name;
    int m_money;
    int m_bmid;
};
ostream& operator<<(ostream& cout,const Person & p1){
    cout << p1.m_name << " : " << p1.m_money ;
    return cout ;
}
template<class T1>
class MyCompara2{
    public:
    bool operator()(const T1& v1, const T1& v2) const { // const 
        if(v1.m_age == v2.m_age){
            return v1.m_height > v2.m_height;
        }
        return v1.m_age > v2.m_age;
    }
};



// 自定义数据类型
void test01(){
    vector<Person> v1;
    string nameSeed = "ABCDEFGHIJ";
    for(int i = 0; i < 10; ++i){
        // string pName = "员工_" + nameSeed[i];
        // "员工_" 字符串在C++ 中默认为char*类型，并非为string类型
        string pName = string("员工_") + nameSeed[i];
        Person p(pName);
        v1.push_back(p);
    }
    for(vector<Person>::iterator it = v1.begin(); it!=v1.end(); ++it){
        it->m_bmid = rand()%3;
        it->m_money = rand()%500 + 1000;
    }

    map<int, string> empolyee;
    empolyee.insert(map<int, string>::value_type(0, "策划"));
    empolyee.insert(map<int, string>::value_type(1, "美术"));
    empolyee.insert(map<int, string>::value_type(2, "研发"));

    multimap<int, Person> mp;

    for(vector<Person>::iterator it = v1.begin(); it!=v1.end(); ++it){
        mp.insert(multimap<int, Person>::value_type(it->m_bmid, *it));
    }
    for(int i = 0; i < 3; ++i){
        cout << empolyee[i] << endl;
        int count = mp.count(i);
        int j = 0;
        for( multimap<int, Person>::iterator it = mp.find(i);it != mp.end() && j < count; ++it, ++j){
            cout << it->second << " ";
        }
        cout << endl;
    }
    

}
int main(){
    test01();
    return 0;
}
```

## 3. 函数对象

### 3.1. 介绍

#### 3.1.1. 概念

+ 重载**函数调用操作符**的类，其对象称为函数对象
+ 函数对象使用重载后的`()`时，类似于函数，因此称为仿函数

#### 3.1.2. 本质

函数对象(仿函数)本质上是一个**类**，而非函数

#### 3.1.3. 使用

+ 使用时类似于普通函数，有调用行为，可以有参数和返回值
+ 有普通函数所不具有的**状态**的概念
+ 可以作为参数传递

### 3.2. 谓词

+ 返回bool类型的仿函数为谓词
+ 如果operator()接受一个参数，称之为一元谓词
+ 如果operator()接受两个参数，称之为二元谓词

## 4. 内建函数对象

### 4.1. 概念

STL内建了一些函数对象

### 4.2. 分类

+ 算术仿函数
+ 关系仿函数
+ 逻辑仿函数

### 4.3. 用法

+ 仿函数的功能与一般函数完全相同
+ 需要引入头文件`#include<functional>`

## 5. 内建函数对象：算术仿函数

### 5.1. 功能描述

+ 实现四则运算
+ 其中 `negate`为一元运算，其余为二元函数

### 5.2. 仿函数原型

+ `template<class T> T plus <T>` //加法
+ … // 减乘除取模都和加法一致
+ `template<class T> T negate <T>` // 取反

### 5.3. 代码

```c++
#include<iostream>
#include<functional>//内建函数对象头文件
using namespace std;
// negate 一元仿函数 取反仿函数
void test01(){
	negate<int> n;
	cout << n(50) <<endl;
}
// plus 二元仿函数 加法仿函数
void test02(){
	plus<int>p; // 只允许传入同等数据类型，因为模版中只给出一个数据类型TP
	cout << p(10, 20) << endl;
}
int main(){
	test02();
	return 0;
}
```

## 6. 内建函数对象：关系仿函数

### 6.1. 功能描述

实现关系对比操作

### 6.2. 仿函数原型

+ `template<class T> bool greater<T>` //大于
+ … 其余类似

### 6.3. 代码

```C++
#include<iostream>
#include<functional>//内建函数对象头文件
#include<vector>
#include<algorithm>
using namespace std;

// 内建函数对象 关系仿函数
// 大于 greater
class MyCompara{
    public:
    bool operator()(int v1, int v2){
        return v1 > v2;
    }
};

void test01(){
    vector<int> v;
    v.push_back(10);
    v.push_back(30);
    v.push_back(50);
    v.push_back(40);
    v.push_back(20);

    for(vector<int>::iterator it= v.begin(); it!= v.end();++it){
        cout << *it << " ";
    }
    cout << endl;

    //降序
    sort(v.begin(), v.end(), MyCompara());

    for(vector<int>::iterator it= v.begin(); it!= v.end();++it){
        cout << *it << " ";
    }
    cout << endl;
}

void test02(){
    vector<int> v;
    v.push_back(10);
    v.push_back(30);
    v.push_back(50);
    v.push_back(40);
    v.push_back(20);

    // greater 内建函数对象
    sort(v.begin(), v.end(), greater<int>()); 
     for(vector<int>::iterator it= v.begin(); it!= v.end();++it){
        cout << *it << " ";
    }
    cout << endl;
}

int main(){
    test02();
    return 0;
}
```

## 7. 内建函数对象：逻辑仿函数

### 7.1. 功能描述

实现逻辑运算

### 7.2. 函数原型

+ `template<class T> bool logical_and<T>` // 与
+ `template<class T> bool logical_or<T>` // 或
+ `template<class T> bool logical_not<T>` //非

> 逻辑仿函数在实际开发中用不到

### 7.3. 代码

```C++
#include<iostream>
#include<functional>//内建函数对象头文件
#include<vector>
#include<algorithm>
using namespace std;

// 内建函数对象 逻辑仿函数
// logical_not
void test01(){
    vector<bool> v;
    v.push_back(1);
    v.push_back(0);
    v.push_back(1);
    v.push_back(0);

    for(vector<bool>::iterator it = v.begin(); it!=v.end(); ++it){
        cout << *it <<" ";
    }
    cout << endl;

    // 运用逻辑非 将容器v内容搬运到容器v2中，并执行取反操作
    vector<bool> v2;
    v2.resize(v.size());
    transform(v.begin(), v.end(), v2.begin(), logical_not<bool>());
    for(vector<bool>::iterator it = v2.begin(); it!=v2.end(); ++it){
        cout << *it <<" ";
    }
      cout << endl;
}

int main(){
    test01();
    return 0;
}
```

## 8. 常用算法

+ 主要有头文件`<algorithm>`、`<functional>`、`<numeric>`
+ `<algorithm>`：所有STL头文件中最大的一个，范围涉及比较、交换、查找、遍历操作、复制、修改等
+ `<functional>`：定义了一些模版类，用以声明函数对象
+ `<numeric>`：体积很小，只包括几个序列上进行的简单数学运算的模版函数

### 8.1. 常用遍历算法

#### 8.1.1. for_each

##### 概念描述

实现遍历容器

##### 函数原型

`for_each(iterator beg, iterator end, _func);`

+ `beg`：起始迭代器
+ `end`：结束迭代器
+ `_func`：函数或函数对象

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
// 常用遍历算法 for_each

// 普通函数
void print01(int val){
	cout << val << " ";
}

// 仿函数
class print02{
	public:
	void operator()(int val){
		cout << val << " ";
	}
};

void test01(){
	vector<int> v;
	for(int i = 0; i < 10; ++i){
		v.push_back(i);
	}
	for_each(v.begin(), v.end(), print01);
	cout << endl;
	for_each(v.begin(), v.end(), print02()); // 仿函数需要是函数对象
	cout<< endl;
}
int main(){
	test01();
	return 0;
}
```

#### 8.1.2. transform

##### 概念

搬运容器内元素到另一个容器中

##### 函数原型

`transform(iterator beg1, iterator end1, iterator beg2, _func);`

+ `beg1`：源容器初始迭代器
+ `end1`：源容器结束迭代器
+ `beg2`：目标容器初始迭代器
+ `_func`：函数或函数对象

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用遍历算法 transform

// 仿函数
class MyTransform{
    public:
    int operator()(int val){
        return val+100; // 可以在搬运的过程中对搬运的元素做额外操作
    }
};
class Myprint{
    public:
    void operator()(int val){
        cout << val << " ";
    }
};
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }
    vector<int> vTarget;
    vTarget.resize(v.size());// 目标容器需要提前设置容器大小

    transform(v.begin(), v.end(),vTarget.begin(), MyTransform());
    for_each(vTarget.begin(),vTarget.end(), Myprint());
    cout << endl;
}

int main(){
    test01();
    return 0;
}
```

### 8.2. 常用查找算法

#### 8.2.1. find

##### 概念

查找指定元素，找到返回指定元素的迭代器，找不到返回结束迭代器

##### 函数原型

`find(iterator beg, iterator end, value);`

+ `beg`：起始迭代器
+ `end`：结束迭代器
+ `value`：查找的元素

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
#include<string>
using namespace std;

// 常用查找算法 find

// 查找内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }

//    vector<int>::iterator it =  find(v.begin(), v.end(), 5);
    vector<int>::iterator it =  find(v.begin(), v.end(), 50);
   if(it == v.end()){
    cout << "没有找到" << endl;
   }else{
    cout << "找到了"<< *it << endl;
   }
}
// 查找自定义数据类型
class Person{
    public:
    Person(string name, int age){
        m_name = name;
        m_age = age;
    }
    // 重载 == 使底层find知道如何比较 !!!!
    bool operator==(const Person& p){
        return p.m_age == this->m_age && p.m_name == this->m_name;
    }
    string m_name;
    int m_age;
};
void test02(){
    vector<Person> v;
    Person p1("aaa", 10);
    Person p2("bbb", 20);
    Person p3("ccc", 30);
    Person p4("ddd", 40);


    v.push_back(p1);
    v.push_back(p2);
    v.push_back(p3);
    v.push_back(p4);

    Person pp("bbb", 20);
    vector<Person>::iterator it =  find(v.begin(), v.end(), pp);
    if(it != v.end()){
        cout << "找到了" << endl;
    }else{
        cout << "没找到" << endl;
    }
}

int main(){
    test02();
    return 0;
}
```

#### 8.2.2. find_if

##### 概念

按条件查找元素，找到返回指定位置迭代器，否则返回结束迭代器

##### 函数原型

`find_if(iterator beg, iterator end, _pred)`

+ 开始迭代器
+ 结束迭代器
+ `_pred`函数或谓词(返回bool类型的仿函数)

[[#谓词]]

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用查找算法 find_if

// 查找内置数据类型
class GreaterFive{
public:
    bool operator()(int val){
        return val > 5;
    }
};
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }

   vector<int>::iterator it =  find_if(v.begin(), v.end(), GreaterFive());
   if(it == v.end()){
    cout << "没有找到" << endl;
   }else{
    cout << "找到了："<< *it << endl;
   }
}
// 查找自定义数据类型
class Person{
    public:
    Person(string name, int age){
        m_name = name;
        m_age = age;
    }

    string m_name;
    int m_age;
};
class PersonGreater20{
public:
    bool operator()(Person &p){
        return p.m_age > 20;
    }
};
void test02(){
    vector<Person> v;
    Person p1("aaa", 10);
    Person p2("bbb", 20);
    Person p3("ccc", 30);
    Person p4("ddd", 40);


    v.push_back(p1);
    v.push_back(p2);
    v.push_back(p3);
    v.push_back(p4);

    vector<Person>::iterator it =  find_if(v.begin(), v.end(), PersonGreater20());
    if(it != v.end()){
        cout << "找到了:" << it->m_name << endl;
    }else{
        cout << "没找到" << endl;
    }
}

int main(){
    test02();
    return 0;
}
```

#### 8.2.3. adjacent_find

##### 概念

查找相邻重复元素，返回相邻重复元素的第一个位置的迭代器

##### 函数原型

`adjacent_find(iterator beg, iterator end);`

+ `beg`：开始迭代器
+ `end`：结束迭代器

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用查找算法 adjacent_find
void test01(){
    vector<int> v;
    v.push_back(0);
    v.push_back(2);
    v.push_back(0);
    v.push_back(5);
    v.push_back(1);
    v.push_back(1);
    v.push_back(7);
    v.push_back(0);

    vector<int>::iterator it =  adjacent_find(v.begin(), v.end());
    if(it == v.end()){
        cout << "没有找到" << endl;
    }else{
        cout << "找到了："<< *it << endl;
    }
}

int main(){
    test01();
    return 0;
}
```

#### 8.2.4. binary_search

##### 描述

查找指定元素是否存在，存在返回`true`，否则返回`false`

> [!note] 注意
> 必须在有序序列中应用，如果是无序列表则结果未知

##### 函数原型

`bool binary_search(iterator beg, iterator end, value);`

+ `beg`：起始迭代器
+ `end`：结束迭代器
+ `value`：要查找的元素

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用查找算法 binary_search
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }

    bool ifFind =  binary_search(v.begin(), v.end(), 9);
    if(ifFind){
        cout << "找到了" << endl;
    }else{
        cout << "没有找到" << endl;
    }
}

int main(){
    test01();
    return 0;
}
```

#### 8.2.5. count

##### 概念

统计元素出现的次数

##### 函数原型

`count(iterator beg, iterator end, value);`

+ `beg`：起始迭代器
+ `end`：结束迭代器
+ `value`：统计的元素

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用查找算法 count 

// 统计内置数据类型
void test01(){
    vector<int> v;
    v.push_back(10);
    v.push_back(40);
    v.push_back(30);
    v.push_back(40);
    v.push_back(20);
    v.push_back(30);
    v.push_back(60);

    int countN =  count(v.begin(), v.end(), 40);
    cout << "40的个数为：" << countN << endl;
}

class Person{
public:
    Person(string name, int age){
        this->m_age = age;
        this->m_name = name;
    }
    bool operator==(const Person &p){ // 参数需要加const
        return p.m_age == this->m_age ;
    }

    string m_name;
    int m_age;
};
// 统计自定义数据类型
void test02(){
    vector<Person> v;

    Person p1("刘备", 35);
    Person p2("章鱼", 34);
    Person p3("张飞", 33);
    Person p4("赵云", 33);
    Person p5("韩信", 100);
    Person p6("马良", 33);

    v.push_back(p1);
    v.push_back(p2);
    v.push_back(p3);
    v.push_back(p4);
    v.push_back(p5);
    v.push_back(p6);

    Person p("诸葛亮", 33);

    int countN = count(v.begin(), v.end(), p);
    cout << "年龄为33岁的人有" << countN << "个"<<endl;
}

int main(){
    test02();
    return 0;
}
```

#### 8.2.6. count_if

##### 概念

按条件统计指定元素的个数

##### 函数原型

`count_if(iterator beg, iterator end, _prod);`

+ `beg`
+ `end`
+ `_prod`：函数或函数对象（谓词）

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用查找算法 count 
class Greater20{
  public:
  bool operator()(int val){
    return val > 20;
  }  
};
// 统计内置数据类型
void test01(){
    vector<int> v;
    v.push_back(10);
    v.push_back(40);
    v.push_back(30);
    v.push_back(40);
    v.push_back(20);
    v.push_back(30);
    v.push_back(60);

    int countN =  count_if(v.begin(), v.end(), Greater20());
    cout << "大于20的数的个数：" << countN << endl;
}

class Person{
public:
    Person(string name, int age){
        this->m_age = age;
        this->m_name = name;
    }
    bool operator==(const Person &p){ // 参数需要加const
        return p.m_age == this->m_age ;
    }

    string m_name;
    int m_age;
};
class PersonAgeGreater33{
    public:
    bool operator()(const Person &p){
        return p.m_age > 33;
    }
};
// 统计自定义数据类型
void test02(){
    vector<Person> v;

    Person p1("刘备", 35);
    Person p2("章鱼", 34);
    Person p3("张飞", 33);
    Person p4("赵云", 33);
    Person p5("韩信", 100);
    Person p6("马良", 33);
    Person p7("诸葛亮", 39);

    v.push_back(p1);
    v.push_back(p2);
    v.push_back(p3);
    v.push_back(p4);
    v.push_back(p5);
    v.push_back(p6);
    v.push_back(p7);

    int countN = count_if(v.begin(), v.end(), PersonAgeGreater33());
    cout << "年龄大于33的人数有：" << countN << "个"<<endl;
}

int main(){
    test02();
    return 0;
}
```

### 8.3. 常用排序算法

#### 8.3.1. sort

##### 概念

对容器内元素进行排序

##### 函数原型

`sort(iterator beg, iterator end, _pred);`

+ `beg`开始迭代器
+ `end`结束迭代器
+ `_pred`谓词，可以忽略，默认为从小到大排序

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用排序算法 sort 
class MyCompara{
  public:
  bool operator()(int a, int b){
    return a>b;
  }  
};
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    v.push_back(10);
    v.push_back(40);
    v.push_back(30);
    v.push_back(40);
    v.push_back(20);
    v.push_back(30);
    v.push_back(60);

    sort(v.begin(), v.end(), MyCompara());
    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;

    sort(v.begin(), v.end(), greater<int>());
    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
}

class Person{
public:
    Person(string name, int age){
        this->m_age = age;
        this->m_name = name;
    }
    bool operator==(const Person &p){ // 参数需要加const
        return p.m_age == this->m_age ;
    }
    bool operator>(const Person &p)const{ // 参数需要加const 函数体需要加const
        return this->m_age == p.m_age ;
    }

    string m_name;
    int m_age;
};

// 自定义数据类型
class MyCompara2{
public:
    bool operator()(const Person & p1, const Person & p2){
        return p1.m_age > p2.m_age;
    }
};
class MyPrint2{
    public:
    void operator()(const Person & p){
        cout << "姓名为："<<p.m_name << ", "<< "年龄为："<<p.m_age<< endl;
    }
};
void test02(){
    vector<Person> v;

    Person p1("刘备", 35);
    Person p2("章鱼", 34);
    Person p3("张飞", 33);
    Person p4("赵云", 33);
    Person p5("韩信", 100);
    Person p6("马良", 33);
    Person p7("诸葛亮", 39);

    v.push_back(p1);
    v.push_back(p2);
    v.push_back(p3);
    v.push_back(p4);
    v.push_back(p5);
    v.push_back(p6);
    v.push_back(p7);

    sort(v.begin(), v.end(), MyCompara2());
    for_each(v.begin(), v.end(), MyPrint2());
    cout << endl;

    sort(v.begin(), v.end(), greater<Person>());
    for_each(v.begin(), v.end(), MyPrint2());
    cout << endl;
}

int main(){
    test02();
    return 0;
}
```

#### 8.3.2. random_shuffle

##### 概念

洗牌，指定范围内的元素随机排序

##### 函数原型

`random_shuffle(iterator beg, iterator end);`

+ `beg`开始迭代器
+ `end`结束迭代器

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
#include<random>
#include<ctime>
using namespace std;

// 常用排序算法 random_shuffle
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }

    random_shuffle(v.begin(), v.end());
    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
}



int main(){
	srand((unsigned int) time(NULL)); // 随机种子
    test01();
    return 0;
}
```

#### 8.3.3. shuffle

##### 概念

洗牌，指定范围内的元素随机排序

##### 函数原型

`shuffle(iterator beg, iterator end, random_engine)`

+ `beg`
+ `end`
+ `random_engine`：随机数生成器，`default_random_engine(time(NULL))`

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
#include<random>
#include<ctime>
using namespace std;

// 常用排序算法 random_shuffle
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }

    shuffle(v.begin(), v.end(), default_random_engine(time(NULL)));
    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
}

int main(){
    test01();
    return 0;
}
```

#### 8.3.4. merge

##### 概念

将两个容器合并到一起，并存储到第三个容器中

>[!note]
>+ 两个容器必须是有序的
>
>+ 合成的容器也是有序的
>
>+ 目标容器需要提前分配内存，且分配的内存需要大于两个源容器所占内存

##### 函数原型

`merge(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest);`

+ `beg1`容器1起始迭代器
+ `end1`容器1结束迭代器
+ `beg2`容器2起始迭代器
+ `end2`容器2结束迭代器
+ `dest`目标容器的起始迭代器

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用排序算法 reverse
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }
    vector<int> v2;
    for(int i = 1; i < 20; ++i){
        v2.push_back(i);
    }
    vector<int> v3;
    v3.resize(v.size()+v2.size()); // 需要提前分配内存

    merge(v.begin(), v.end(), v2.begin(), v2.end(), v3.begin());

    for_each(v3.begin(), v3.end(), MyPrint());
    cout << endl;
}
int main(){
	test01();
	return 0;
}
```

#### 8.3.5. reverse

##### 概念

将容器中元素次序反转

##### 函数原型

`reverse(iterator beg, iterator end)`

+ `beg`
+ `end`

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用排序算法 merge

class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }
    reverse(v.begin(), v.end());

    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
}
int main(){
	test01();
	return 0;
}
```

### 8.4. 常用拷贝和替换算法

#### 8.4.1. copy

##### 概念

容器内指定范围的元素拷贝到另一个容器中

>[!note]
>需要提前给目标容器开辟空间

##### 函数原型

`copy(iterator beg1, iterator end1, iterator dest);`

+ `beg1`起始迭代器
+ `end1`结束迭代器
+ `dest`目标容器起始迭代器

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
#include<random>
using namespace std;

// 常用拷贝和替换算法 copy
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }
    vector<int>v2;
    v2.resize(v.size());
    copy(v.begin(), v.end(), v2.begin());

    for_each(v2.begin(), v2.end(), MyPrint());
    cout << endl;
}
int main(){
	test01();
	return 0;
}
```

#### 8.4.2. replace

##### 概念

将容器中指定范围的旧元素改为新元素

##### 函数原型

`replace(iterator begn, iterator end, oldValue, newValue);`

+ `beg`起始迭代器
+ `end`结束迭代器
+ `oldValue`旧元素
+ `newValue`新元素

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用拷贝和替换算法 replace
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }
   
    replace(v.begin(), v.end(), 5, 500);

    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
}
int main(){
	test01();
	return 0;
	system("pause");
}
```

#### 8.4.3. replace_if

##### 概念

将区间内满足条件的元素，替换成指定元素

##### 函数原型

`replace_if(iterator beg, iterator end, _pred, newValue)`

+ `beg`
+ `end`
+ `_pred`：谓词
+ `newValue`

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用拷贝和替换算法 replace
class MyCompara{
  public:
  bool operator()(int a){
    return a>=5;
  }  
};
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }
    cout << "替换前：";
    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
    
    replace_if(v.begin(), v.end(), MyCompara(), 500);
    
    cout << "替换后：";
    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
}
```

#### 8.4.4. swap

##### 概念

互换两个容器中的元素

>[!note]
>+ 两个容器的类型需要一致
>+ 两个容器即使大小不一样，也可以直接交换

##### 函数原型

`swap(container c1, container c2);`

+ `c1`容器1
+ `c2`容器2

##### 代码

```C++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

// 常用拷贝和替换算法 swap
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i < 10; ++i){
        v.push_back(i);
    }
    vector<int> v2;
    for(int i = 100; i < 1000; ++i){
        v2.push_back(i);
    }
    cout << "交换前：" << endl;
    cout << "v1: ";
    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
    cout << "v2: ";
    for_each(v2.begin(), v2.end(), MyPrint());
    cout << endl;

    // v.resize(v2.size());
    swap(v, v2);

    cout << "交换后："<<endl;
    cout << "v1: ";
    for_each(v.begin(), v.end(), MyPrint());
    cout << endl;
    cout << "v2: ";
    for_each(v2.begin(), v2.end(), MyPrint());
    cout << endl;
}

```

### 8.5. 常用算术生成算法

头文件为`#include<numeric>`

#### 8.5.1. accumulate

##### 概念

计算区间内元素累计总和

##### 函数原型

`accumulate(iterator beg, iterator end, value);`

+ `beg`起始迭代器
+ `end`结束迭代器
+ `value`起始值

##### 代码

```C++
#include<iostream>
#include<vector>
#include<numeric>// 算术生成算法的头文件
using namespace std;

// 常用算术生成算法 accumulate
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i = 0; i <= 100; ++i){
        v.push_back(i);
    }
    int a = accumulate(v.begin(), v.end(), 0);
    cout << a << endl;
}
```

#### 8.5.2. fill

##### 概念

向容器中填充指定的元素

##### 函数原型

`fill(iterator beg, iterator end, value);`

+ `beg`
+ `end`
+ `value`：要填充的值

##### 代码

```C++
#include<iostream>
#include<vector>
#include<numeric>// 算术生成算法的头文件
#include<algorithm>
using namespace std;

// 常用算术生成算法 fill
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    v.resize(10);
    fill(v.begin(), v.end(), 100);    
    for_each(v.begin(), v.end(), MyPrint());
}
```

### 8.6. 常用集合成算法

#### 8.6.1. set_intersection

##### 概念

求两个容器的交集，并存入第三个容器中，并返回交集中最后一个位置的迭代器

> [!note]
> + 两个容器内的元素必须有序
> + 目标容器必须提前分配内存
> + 返回值为交集最后一个元素在目标容器中的迭代器

##### 函数原型

`iterator itEnd = set_intersection(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest`

+ `beg1`容器1起始迭代器
+ `end1`容器1结束迭代器
+ `beg2`容器2起始迭代器
+ `end2`容器2结束迭代器
+ `dest`目标容器起始迭代器

##### 代码

```C++
#include<iostream>
#include<vector>
#include<numeric>// 算术生成算法的头文件
#include<algorithm>
using namespace std;

// 常用集合算法 set_intersection
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i =0; i < 10; ++i){
        v.push_back(i);
    }
    vector<int> v2;
    for(int i =5; i < 70; ++i){
        v2.push_back(i);
    }
    vector<int> vTarget;
    vTarget.resize(min(v.size(), v2.size()));
    vector<int>::iterator it = set_intersection(v.begin(), v.end(), v2.begin(), v2.end(), vTarget.begin());
    for_each(vTarget.begin(), itEnd, MyPrint());
}
```

#### 8.6.2. set_union

##### 概念

求两个集合的并集

> [!note]
> + 两个容器内的元素必须有序
> + 目标容器必须提前分配内存
> + 返回值为交集最后一个元素在目标容器中的迭代器

##### 函数原型

`iteratro itEnd = set_union(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest);`

+ `beg1`：容器1起始迭代器
+ `end1`：容器1结束迭代器
+ `beg2`：容器2起始迭代器
+ `end2`：容器2结束迭代器
+ `dest`：目标容器起始迭代器
+ `itEnd`：目标容器中并集最后一个元素的迭代器

##### 代码

```C++
#include<iostream>
#include<vector>
#include<numeric>// 算术生成算法的头文件
#include<algorithm>
using namespace std;

// 常用集合算法 set_union
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i =0; i < 10; ++i){
        v.push_back(i);
    }
    vector<int> v2;
    for(int i =5; i < 70; ++i){
        v2.push_back(i);
    }
    vector<int> vTarget;
    vTarget.resize(v.size()+ v2.size());
    vector<int>::iterator itEnd = set_union(v.begin(), v.end(), v2.begin(), v2.end(), vTarget.begin());
    for_each(vTarget.begin(), itEnd, MyPrint());

}
int main(){
    test01();
    return 0;
    system("pause");
}
```

#### 8.6.3. set_difference

##### 概念

求两个集合的差集

>[!Note]
> + 两个容器内的元素必须有序
> + 目标容器必须提前分配内存
> + 返回值为交集最后一个元素在目标容器中的迭代器
> + 需注意是求谁对谁的交集，容器之间的顺序不同结果不同

##### 函数原型

`iterator itEnd = set_difference(iterator beg1, iterator end1， iterator beg2, iterator end2, iterator dest);`

+ `beg1`：容器1起始迭代器
+ `end1`：容器1结束迭代器
+ `beg2`：容器2起始迭代器
+ `end2`：容器2结束迭代器
+ `dest`：目标容器起始迭代器
+ `itEnd`：目标容器中并集最后一个元素的迭代器

##### 代码

```C++
#include<iostream>
#include<vector>
#include<numeric>// 算术生成算法的头文件
#include<algorithm>
using namespace std;

// 常用集合算法 set_union
class MyPrint{
public:
    void operator()(int a){
        cout << a << " ";
    }
};
// 内置数据类型
void test01(){
    vector<int> v;
    for(int i =0; i < 10; ++i){
        v.push_back(i);
    }
    vector<int> v2;
    for(int i =5; i < 70; ++i){
        v2.push_back(i);
    }
    vector<int> vTarget;
    vTarget.resize(max(v.size(), v2.size()));
    vector<int>::iterator itEnd = set_difference(v.begin(), v.end(), v2.begin(), v2.end(), vTarget.begin());
    for_each(vTarget.begin(), itEnd, MyPrint());
    cout << endl;

    vector<int> vTarget2;
    vTarget2.resize(max(v.size(), v2.size()));
    vector<int>::iterator itEnd2 = set_difference(v2.begin(), v2.end(), v.begin(), v.end(), vTarget2.begin());
    for_each(vTarget2.begin(), itEnd2, MyPrint());
    cout << endl;
}
```
