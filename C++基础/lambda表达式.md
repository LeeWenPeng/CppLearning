## 1  基本语法

```cpp
[捕获列表](参数列表) -> 返回类型{函数体};
```

## 2  捕获列表

- 值捕获，直接对变量现在的值进行复制，然后保留副本
	- 捕获的范围为 lambda 表达式定义行之前的行
	- 对范围内，在捕获列表中的变量最新的值进行拷贝一个副本
	- 因此，即使通过 mutable 关键字修饰后，对该变量进行修改，也仅仅修改的副本的值，不会修改原值
- 引用捕获
	- 捕获的范围为 lambda 所在的整个作用域
	- 类似于引用传递，是将原变量的地址进行传递。
		- 因此，其捕获的值始终为调用 lambda 表达式前的最新值。
		- 而且，对于被捕获变量的修改，也会实际上修改 lambda 表达式外的原变量
- 右值捕获
	- c++ 14 后，允许捕获的成员用任意表达式进行初始化，被声明的捕获变量类型会根据表达式进行判断，判断方式和 auto 相同
- 特殊情况
	- `[]`：空捕获列表
	- `[=]`：隐式捕获，全部变量进行值传递
	- `[&]`：隐式捕获，全部变量进行引用传递
	- `[=, identifier_list]`：包含在 identifier_list 中的变量采用引用传递，除此之外的所有变量采用值传递
		- `\=`：隐式捕获，值传递
		- `identifier_list`：逗号分割的名字列表，包含 0 个或多个来自所在函数的变量
	- `[&, identifier_list]`：包含在 identifier_list 中的变量采用值传递，除此之外的所有变量采用引用传递
		- `&`：隐式捕获，引用传递
		- `identifier_list`：逗号分割的名字列表，包含 0 个或多个来自所在函数的变量

## 3  泛型 lambda

c++14 后，lambda 函数的形参可以使用 auto 来产生意义上的泛型

```cpp
void test1(){
	cout << "test1" << endl;
	auto add = [](auto x, auto y){
		return x+y;
	};
	
	cout << add(1, 2) << endl;	
	cout << add(1.1, 2.2) << endl;
}

int main(){
	test1();
	return 0;
}
```

## 4  可变 lambda

- 采用值捕获的方式时，lambda 不能修改被捕获变量的值。但是通过 mutable 关键字修饰，就可以修改。
	- 只修改 lambda 表达式内被捕获变量的值，不会更新 lambda 表达式外原变量的值
- 采用引用捕获的方式时，lambda 可以修改被不会变量的值，同时更新 lambda 表达式外原变量的值

```cpp
#include <algorithm>
#include <cstdio>
#include <iostream>
#include <utility>
using namespace std;

void test1(){
	cout << "test1" << endl;
	
    int v = 5;
    auto ff = [v]() {
        // return ++v; // 此处报错
        return v+1;
    };

    v = 0;
    auto j = ff(); 
    cout << v << endl; // v = 0
    cout << j << endl; // j = 6
}

void test2(){
	cout << "test2" << endl;
	
    int v = 5;
    auto ff = [v]() mutable{
        return ++v;
    };

    v = 0;
    auto j = ff(); 
    cout << v << endl; // v = 0
    cout << j << endl; // j = 6
}

void test3(){
    cout << "test3" << endl;

    int v = 5;
    auto ff=[&v](){
        return ++v;
    };

    v = 0;
    auto j = ff(); 
    cout << v << endl; // v = 1
    cout << j << endl; // j = 1;
}


int main(){
	test1();
    test2();
    test3();
    return 0;
}
```
