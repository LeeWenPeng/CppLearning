## 1  explicit 关键字的作用

`explicit` 关键字用于**防止构造函数的隐式转换**。

## 2  具体示例说明

### 2.1  没有 explicit 的情况（允许隐式转换）

```cpp
class MyClass {
public:
    MyClass(int value) {  // 允许隐式转换
        this->value = value;
    }
    int value;
};

void func(MyClass obj) {
    // 处理对象
}

int main() {
    MyClass obj1(10);     // 正确：显式构造
    MyClass obj2 = 20;    // 正确：隐式转换！int 被隐式转换为 MyClass
    func(30);             // 正确：隐式转换！30 被隐式转换为 MyClass
    return 0;
}
```

### 2.2  使用 explicit 的情况（禁止隐式转换）

```cpp
class MyClass {
public:
    explicit MyClass(int value) {  // 禁止隐式转换
        this->value = value;
    }
    int value;
};

void func(MyClass obj) {
    // 处理对象
}

int main() {
    MyClass obj1(10);     // 正确：显式构造
    MyClass obj2 = 20;    // 错误！不允许隐式转换
    func(30);             // 错误！不允许隐式转换
    func(MyClass(30));    // 正确：显式构造
    return 0;
}
```

## 3  在 WebServer 项目中的具体应用

在我们之前分析的代码中：

```cpp
class Epoller {
public:
    explicit Epoller(int maxEvent = 1024);  // 使用 explicit
    // …
};
```

### 3.1  为什么要在 Epoller 中使用 explicit？

**防止意外的隐式转换**：

```cpp
// 如果没有 explicit，可能发生这种情况：
void SomeFunction(Epoller epoller) {
    // 使用 epoller
}

int main() {
    SomeFunction(1024);  // 如果没有 explicit，1024 会被隐式转换为 Epoller 对象
                         // 这通常不是我们想要的行为！
    
    // 使用 explicit 后，必须显式创建对象：
    SomeFunction(Epoller(1024));  // 正确：显式构造
    return 0;
}
```

## 4  使用建议

**应该在以下情况下使用 explicit**：

- 单参数构造函数（除了拷贝构造函数）
- 多参数构造函数（C++11 以后）
- 当你希望调用者明确知道他们在创建对象时

**不应该使用 explicit 的情况**：

- 拷贝构造函数
- 移动构造函数
- 当你确实需要隐式转换时
