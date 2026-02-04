## 1  问题：CRTP 向下转换有风险吗

```cpp
```

### 1.1  动态多态的代码复用

```cpp
#include <iostream>

class Base
{
public:
    virtual void implementation()
    {
        std::cout << "Base implementation" << std::endl ;
    }
    // virtual ~Base() = default;
};

class Derived1 : public Base
{
public:
    void implementation() override // 重写
    {
        std::cout << "Derived1 implementation" << std::endl;
    }
};

class Derived2 : public Base
{
public:
    void implementation() override// 重写
    {
        std::cout << "Derived2 implementation" << std::endl;
    }
};

int main()
{
    Derived1 d1;
    Derived2 d2;

    d1.implementation();
    d2.implementation();
}
```

### 1.2  CRTP 的多态

```cpp
#include <iostream>

template <typename Derived>
class Base
{
    // static_assert(std::is_base_of_v<Base<Derived>, Derived>, "Derived must inherit from Base");
public:
    void interface()
    {
        static_cast<Derived*>(this)->implementation();
    }

    void implementation()
    {
        std::cout << "Base implementation" << std::endl;
    }
};

class Derived1 : public Base<Derived1>
{
public:
    void implementation() // 这里隐藏了 Base::implementation
    {
        std::cout << "Derived1 implementation" << std::endl;
    }
};

class Derived2 : public Base<Derived2>
{
public:
    // void implementation() // 这里隐藏了 Base::implementation
    // {
    //     std::cout << "Derived2 implementation" << std::endl;
    // }
};

int main()
{
    Derived1 d1;
    Derived2 d2;

    d1.interface();
    d2.interface();
}
```
