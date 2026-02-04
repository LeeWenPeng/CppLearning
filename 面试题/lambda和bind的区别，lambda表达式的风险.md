## 1  lambda 表达式和 bind 的区别

### 1.1  可读性

- lambda 表达式直接展示函数的调用逻辑、参数的传递清晰
- bind 依赖占位符，映射参数位置，需要查看原函数声明

### 1.2  重载函数支持

- lambda 通过常规函数调用自动匹配重载版本
- bind 无法自动区分重载函数，需要通过 `static_cast` 显示类型转换函数指针类型
- 同时，lambda 表达式可以在 STL 算法中使用，但 bind 很难应用于 STL 容器

#### 1.2.1  普通函数和成员函数的重载

```cpp
#include <iostream>
#include <functional>
#include <vector>
#include <algorithm>

class Calculator{
public:
    int calc(int a, int b) const{
        std::cout << "calc(int a, int b): \n";
        return a + b;
    }

    double calc(double a, double b) const{
        std::cout << "calc(double a, double b): \n";
        return a + b;
    }

    std::string calc(const std::string& a, const std::string& b) const{
        std::cout << "calc(std::string a, std::string b): \n";
        return a + b;
    }
};

int main(){
    Calculator c;

    std::cout << "bind =====================" << std::endl;

	// 1
    auto bind_int = std::bind(
        static_cast<int(Calculator::*)(int, int) const>(&Calculator::calc),
        &c, 1, 2
    );
    int res = bind_int();
    std::cout << "res = " << res << std::endl;

	// 2
    auto bind_double = std::bind(
        static_cast<double(Calculator::*)(double, double) const>(&Calculator::calc),
        &c, 1.0, 2.1
    );
    double res2 = bind_double();
    std::cout << "res2 = " << res2 << std::endl;
	
	// 3
    using calcString = std::string(Calculator::*)(const std::string&, const std::string&) const;
    auto bind_string = std::bind(
        static_cast<calcString>(&Calculator::calc),
        &c, 
        std::placeholders::_1,
        std::placeholders::_2
    );
    std::string res3 = bind_string("a", "b");
    std::cout << "res3 = " << res3 << std::endl;
    std::cout << "\n";

    std::cout << "lambda =====================" << std::endl;
    
    // 4
    auto lambda_int = [&c](int a, int b){
        return c.calc(a, b);
    }; 
    int res4 = lambda_int(1, 2); 
    std::cout << "res4 = " << res4 << std::endl;

	// 5
    auto lambda_double = [&c](double a, double b){
        return c.calc(a, b);
    };
    double res5 = lambda_double(2.1, 2.2);
    std::cout << "res5 = " << res5 << std::endl;

	// 6
    auto lambda_string = [&c](const std::string& a,const std::string& b){
        return c.calc(a, b);
    };
    std::string res6 = lambda_string("std::string a", "std::string b");
    std::cout << "res6 = " << res6 << std::endl;

    return 0;
}
```

#### 1.2.2  模板函数的重载

```cpp
#include <iostream>
#include <functional>
#include <type_traits>

// 模板重载函数
template<typename T>
void process(T value) {
    std::cout << "process(T): " << value << std::endl;
}

// 特化版本
template<>
void process<int>(int value) {
    std::cout << "process<int>: " << value << std::endl;
}

// 非模板重载
void process(double value) {
    std::cout << "process(double): " << value << std::endl;
}

int main() {
    std::cout << "=== bind 处理模板重载 ===" << std::endl;
    
    // bind 无法直接绑定模板函数
    // auto b1 = std::bind(process<int>, 10);  // 错误：process<int> 不是函数
    
    // 必须使用函数指针或包装器
    void (*ptr_int)(int) = process<int>;
    auto b1 = std::bind(ptr_int, std::placeholders::_1);
    b1(10);  // process<int>: 10
    
    // 对于 double 版本，同样需要处理重载
    auto b2 = std::bind(static_cast<void(*)(double)>(process), std::placeholders::_1);
    b2(3.14);  // process(double): 3.14
    
    std::cout << "\n=== lambda 处理模板重载 ===" << std::endl;
    
    // lambda 可以轻松处理模板函数
    auto l1 = [](int x) {
        process(x);  // 调用 process<int>
    };
    
    auto l2 = [](double x) {
        process(x);  // 调用 process<double>
    };
    
    auto l3 = [](auto x) {  // C++14 通用 lambda
        process(x);  // 根据 x 的类型选择正确的重载
    };
    
    l1(20);     // process<int>: 20
    l2(2.718);  // process(double): 2.718
    l3(30);     // process<int>: 30
    l3(3.14159);// process(double): 3.14159
    
    return 0;
}
```

### 1.3  性能

- lambda 的函数调用可在编译期时内联优化
- bind 通过函数指针调用，编译期难以内联优化

### 1.4  移动捕捉（C++11）

- C++11 中，lambda 不支持移动捕捉，也就是不支持初始化捕获，需要结合 bind 模拟移动捕获
- C++14 中，lambda 可以直接移动捕获
- C++20 中，模板 lambda+ 完美转发捕获

### 1.5  结论

- **优先使用 lambda 表达式**
- lambda 表达式更易读、效率高，且避免 bind 的复杂语法以及潜在陷阱

## 2  lambda 表达式引用捕获的风险

1. 悬挂引用
	1. 当 lambda 表达式捕获局部变量的引用，当局部变量的生命周期结束时，该变量将被摧毁。此时调用 lambda 表达式，将访问无效内存，从而导致未定义的行为错误。
	2. 有两种典型情况
		1. lambda 返回捕获的局部变量引用
		2. lambda 生命周期长于捕获对象
	3. 解决方案
		1. 使用值捕获，创建变量副本
		2. 智能指针捕获，延长变量生命周期
		3. 使用移动捕获，转移变量所有全到 lambda
2. 多线程竞争
	1. 若 lambda 表达式被多个线程调用，且通过引用修改捕获变量，则需要手动同步。
		1. 也就是说，本质上是共享资源竞争的问题
	2. 解决方案
		1. 原子操作，保证线程安全
		2. 互斥锁，对资源进行互斥保护
		3. 线程局部存储，取消资源的共享性

## 3  lambda 使用建议

- 默认使用显式值捕获，而非隐式引用捕获
- 捕获成员变量或 this 指针时，创建副本或值，而非直接捕获
- 需要共享时，使用智能指针
- 捕获大对象时，使用移动捕获，避免数据的拷贝
- 明确 lambda 对象拥有传入的可调用对象的所有权
