## 1  头文件

```cpp
#include <functional>
```

## 2  function

### 2.1  功能

类模板 std::function 是一个通用的多态函数包装器

- 可以存储、复制、调用任何允许拷贝构造的目标函数、lambda 表达式、bind 表达式和其他函数对象，以及指向成员函数和数据成员的指针
- function 存储的可调用对象被称为目标。不包含目标的 function 是空的 function。空的 function 在编译时会导致 std：：bad_function_call 异常

### 2.2  函数原型

```cpp
template< class >
class function; /* undefined */

template< class R, class… Args >
class function<R(Args…)>;
```

- 类型
	- R：函数返回值类型
	- Args…：要传递给函数的参数类型

### 2.3  补充

> [!note]
> 在 C++ 中禁止将返回的引用绑定到一个临时对象。因此，当 std::function 的实例被声明为返回引用时，不应该从一个返回纯右值的函数或函数对象（包括没有尾随返回类型的 lambda 表达式）进行初始化。

### 2.4  示例

```cpp
#include <functional>
#include <string>
#include <iostream>
using namespace std;
// 普通函数
void func1(int a){
    cout << a << endl;
}

// 成员函数
class A{
public:
    A(string name):_name(name){}

    void func3(int i) const {
        cout << _name <<  ", "<< i <<endl;
    }


    void func3(string name) const {
        cout << name << endl;
    }

private:
    string _name;
};

int main(){
    // 保存普通函数
    cout << "main1\n";
    std::function<void(int a)> func1_;
    func1_ = func1;
    func1_(2);
    cout << "\n";

    // 保存 lambda 表达式
    cout << "main2\n";
    std::function<void()> func2_ = [](){cout << "lambda\n"; };
    func2_();
    cout << "\n";
    
    // 保存成员函数
    /*
        std::function<R(args…)> f = …
            args 中第一个参数为类对象的引用
            之后为要传递给成员函数的参数类型
        
        f(args…)
            args 中第一个参数为类对象
            之后为要传递给成员函数的参数
    */
    // 绑定函数重载版本
    // 通过 static_cast 指定重载版本
    cout << "main3\n";
    std::function<void(const A&, int)> func3_ = static_cast<void(A::*)(int) const>(&A::func3);
    A a("lee");
    func3_(a, 3);
    cout << "\n";

    cout << "main4\n";
    std::function<void (const A&, string)> func4_ = static_cast<void(A::*)(string) const>(&A::func3);
    A a2("lee");
    func4_(a2, "king");
    cout << "\n";

    // 2. 使用 lambda 表达式包装
    cout << "main5\n";
    std::function<void (const A&, int)> func5_ = [](const A& obj, int i){
        obj.func3(i);
    };
    A a3("lee");
    func4_(a2, "wang");
    cout << "\n";

    cout << "main6\n";
    std::function<void (const A&, string)> func6_ = [](const A& obj, string name){
        obj.func3(name);
    };
    A a4("lee");
    func4_(a2, "daWang");
    cout << "\n";
    return 0;
}
```

## 3  bind

### 3.1  功能

### 3.2  函数原型

```cpp
template< class F, class… Args >
/* unspecified */ bind( F&& f, Args&&… args );

template< class R, class F, class… Args >
/* unspecified */ bind( F&& f, Args&&… args );
```

- 参数
	- F：可调用对象
	- args：要绑定的参数列表，未绑定的参数可以被 `std::placeholders` 命名空间下的 `_1`、`_2`、`_3`…所替换
- 返回值
	- 成功：一个函数对象

### 3.3  补充

已经被 lambda 表达式所取代

### 3.4  示例

```cpp
#include <functional>
#include <iostream>
using namespace std;

void func_1(int a, int b, int c){
    cout <<"print: a=" << a << ", b=" << b << ", c=" << c <<endl;
}

void func_2(int a, int b){
    a++;
    b++;
    cout <<"print: a=" << a << ", b=" << b << endl;
}

class A{
public:
    void func_3(int a, int b){
        cout << "a= " << a << ", b= " << b << endl;
    }
};

int main(){
#if 0

    cout << "std::bind(func_1, 1, 2, 3)" << endl;
    auto f1 = std::bind(func_1, 1, 2, 3);
    f1();

    cout << "std::bind(func_1, \
            std::placeholders::_1, \
            std::placeholders::_2, \
            300);" << endl;
    auto f3 = std::bind(func_1, 
            std::placeholders::_1, 
            std::placeholders::_2, 
            300);
    f3(1, 2);
#endif

#if 1
    cout << "std::bind(func_1, 1, 2, 3)" << endl;
    A a;
    auto f1 = std::bind(&A::func_3, &a, 1, 2);
    f1();

    auto f3 = std::bind(&A::func_3,
        &a, 
        std::placeholders::_1, 
        std::placeholders::_2);
    f3(1, 2);
#endif
    return 0;
}
```
