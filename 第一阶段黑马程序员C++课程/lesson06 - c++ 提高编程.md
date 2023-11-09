# 1 模板

## 1 概念

编程思想：==泛型编程==

模板是一个==通用摸具==，为了提高复用

C++中模板有两种：

+ 函数模板
+ 类模板

> + 模板是一个框架，不能直接使用
> + 模板通用性很高，但并非万能

## 2 函数模板

建立一个通用函数，函数的返回值类型和参数类型==可以先不具体指定==，用一个==虚拟类型==代替

### 1 语法

```c++
template <typename T>
函数的声明或定义
```

+ `template`：模板关键字，告诉编译器，下面是定义的模板
+ `typename T`：虚拟的数据类型，也就是通用数据类型，在对函数模板使用时，会使用具体数据类型实现该虚拟数据类型
  + `typename` 定义 `T` 为虚拟数据类型的关键字，也可以使用 `class`
  + `T`只是名字，可以替换成任意的合法命名

### 2 使用方法

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

### 3 注意事项

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


### 4 函数模板案例 - 不同数据类型数组排序

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

### 5 普通函数与函数模板的调用规则                                               

1. 当普通函数的函数名与函数模板中的函数名相同时
   1. ==代价最小原则==：
      1. **如果参数的数据类型符合普通函数的参数列表，优先调用普通函数**
      2. **否则，优先调用函数模板**
   2. 显式调用模板时，可以通过==空模板参数列表 `<>`== ，告诉编译器此处调用的是函数模板
2. 函数模板也可以发生==函数重载==

> 如果已经提供函数模板，建议不要再一个写同名的普通函数，容易出现二义性



### 6 函数模板的局限性

模板中的 `T` 虽然是通用数据类型，但是如果调用时，传入的数据类型是特殊的数据类型，比如自定义的数据类型，就会导致虽然数据类型可以成功传入，但是在程序运行时，运算符会限制住对传入数据类型的操作，比如两个类对象做 `a == b` 的计算，编译器就会不知道应该如何计算，会照成编译报错。



解决方法：

1. 对运算符的操作

   1. **对自定义的数据类型或者类做运算符重载**

2. 对模板的操作

   1. **对具体数据类型做具体化的实现**

      语法：

      ```c++
      template<>bool 函数名(具体数据类型 a, 具体数据类型 b){
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

## 3 类模板

### 1 语法：

```c++
template <typename T>
类
```

> 定义时语法和函数模板没啥区别

### 2 类模板使用时和函数模板的区别

1. 类模板没有自动类型推导，必须显式指定
2. 类模板的模板参数列表可以有默认值

### 3 类模板的成员函数

+ 普通类的成员函数在定义时创建
+ ==类模板的成员函数在**调用时**创建==

### 4 类模板对象做函数的参数

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

### 5 类模板的继承

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

### 6 类模板成员函数的类外实现

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
>       1. 参数列表和类模板的参数列表相同

### 7 类模板分文件编写

问题：==类模板中的成员函数在调用时创建，在文件链接时，链接不到==

解决方法：

1. 不再引入头文件，而是**直接引入实现了头文件的源文件** `*.cpp*`
2. **将头文件和实现写入到同一个文件当中**，并将文件后缀写为`*.hpp*`
   1. 这个后缀只是一种约定，写成`*.cpp`也无所谓，但写成这样是为了告诉浏览器，这个是关于类模板的一个定义实现文件

#### 第一种解决方法代码：

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

#### 第二种解决方法代码：

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

### 8 类模板与友元函数

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

#### 第一种：友元函数类内实现

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



#### 第二种：友元函数类外实现

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

## 4 案例 - 通用数组

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

# 2 STL 初识

## 1 STL概念

**STL(Standard Template Library)** 标准模板库

大概组成：

1. **容器（container）**
2. **算法(algorithm)**
3. **迭代器(iterator)**

**容器**和**算法**之间用**迭代器**进行连接

## 2 六大组件

1 容器：各种数据结构

2 算法：常用算法

3 迭代器：容器算法之间的胶合剂

4 仿函数：行为类似函数，作为算法的某种策略

​	重载 () 运算符后，使得类的行为类似于函数

5 适配器：用来修饰容器、仿函数或迭代器接口

6 空间配置器：对空间的配置和管理

## 3 容器

常用的数据结构：数组、树、链表、映射表、栈、队列、集合等

分类：

1. 关联式容器：二叉树结构，数据之间没有严格的物理关系
2. 序列式容器：强调数据的顺序，容器中每一个值都有**固定**的位置

## 4 算法

分类：

1. 质变算法：涉及到对数据修改操作的算法
2. 非质变算法：查找，排序等不涉及对数据修改操作的算法

## 5 迭代器

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
   2. 在随机访问迭代器之间支持`>`、`>=`、`<`、`<=`、`==`、`!=`逻辑运算 

# 3 String 容器

> 与 C 语言中的字符串 `char*` 的区别：
>
> + C 语言中的 `char*` 本质上是一个字符数组，或者说是一个指针
>
> + string 的本质是一个类，类中维护着 `char*`，是一个`char*`的容器

## 1 构造函数

1. 无参构造`string();`
   1.  返回一个空字符对象
2. 有参构造`string(const char* s);`
   1. 参数为字符串`s` 
3. 拷贝构造，`string(const string& s);`
4. 有参构造，`string(int n, char c)`
   1. 使用 n 个字符 `c`组成字符串

## 2 赋值

![image-20230927164408819](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230927164408819.png)

> 前三个是对运算符 `=` 的重写，后四个是成员函数
>
> 这些函数的作用是为了防止浅拷贝

## 3 `string` 字符串拼接

在字符串的末尾追加字符

![image-20230927164810903](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230927164810903.png)

## 4 `string` 字符串的查找和替换

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



## 5 字符串的比较

![image-20230930112354240](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230930112354240.png)

比较方式：==根据字符串的 ASCII 值进行比较==

`str1.compare(str2)`

+ `0`：当 `str1 == str2` 返回值为 `0`
+ `1` ：当 `str1 > str2` 返回值为 `1`
+ `-1`：当`str1 < str2` 返回值为 `-1`



## 6 字符存取

从 `string` 中访问单个字符的方式有两种：

1. 通过 `[]` 运算符：`char& oprator[](int n);`
2. 通过`at` 方法：`char& at(int n);`

> 也能通过上述两种方式对字符串进行修改

## 7 字符串插入与删除

![image-20230930112754925](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230930112754925.png)

## 8 string 字串

从 string 对象中获取字符串子串

![image-20230930112951981](lesson06%20-%20c++%20%E6%8F%90%E9%AB%98%E7%BC%96%E7%A8%8B.assets/image-20230930112951981.png)

# 4 vector 容器

1. 和数组十分相似，被称为**单端数组**
2. 可以**动态扩展**
3. 其迭代器是支持随机访问的迭代器

> vector 容器的动态扩展：
>
> 1. 并不是在原空间后接入新空间，而是，重新寻找一个更大的内存空间，将原内存中的数据复制到新内存空间中，并释放原空间
>
> 2. 找到的新空间的内存大小往往是原空间的 `2` 倍

![image-20230930181910686](lesson06 - c++ 提高编程/img/image-20230930181910686.png)

+   单端数组：只允许尾端进行插入删除操作

    +   `push_back()` 在尾端插入
    +   `pop_back()` 在尾端删除

+   提供接口：

    +   `front()` 第一个元素 

    +   `back()` 最后一个元素
    +   `insert()` 插入接口
    +   `erase()` 删除接口

+   迭代器：
    +   `beigin()` 指向第一个元素的第一个位置
    +   `end()` 指向最后一个元素的后一个位置
    +   `rbegin()` 指向最后一个元素的位置
    +   `rend()` 指向第一个元素的前一个位置  

## 1 构造函数

![image-20230930192026397](lesson06 - c++ 提高编程/img/image-20230930192026397.png)

### 1 默认构造函数

```c++
// 声明
vector<int> v;
// 初始化
for(int i =0; i!=10; ++i){
    v.push_back(i);
}
```



###  2 通过迭代器传值构造

语法：`vector<T> v2(v1.beigin(), v1.end());`

### 3  n 个 elem 构造

语法：`vector<T> v(n, elem);`

### 4 复制构造函数

语法：`vector<T> v(vector<T> v2);`

### 5 代码

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

## 2 赋值操作

给 vector 容器赋值 

1.   重载赋值符号 `=`
2.   `assign` 方法：
     1.   `assign(beg, end);`
     2.   `assign(n, elem);`

