## 1  const 常量

`const` 可以修饰变量，使其成为常量，==必须在声明时初始化，之后不能修改==。

```cpp
const datatype k = v;
```

## 2  const 指针

### 2.1  常量指针

常量指针：指针为常量，指针指向地址中存放的值可以修改，但指针本身的指向不能修改

```cpp
int* const p = &v;
```

### 2.2  指针指向常量

指针指向常量：指针指向地址中存放的值不能修改，但指针的指向可以修改。

```cpp
const int* p = &v;
```

### 2.3  常量指针指向常量

指针指向地址内存放的值以及指针的指向都不可以修改

```cpp
const int* const p = &v;
```

## 3  const 修饰数组

数组内元素不可以修改

```cpp
const int arr[] = {1,2,3};
```

## 4  const 修饰类

### 4.1  const 成员变量

const 修饰成员变量，只能在构造函数的初始化列表中赋值

```cpp
class MyClass {
private:
    const int a;  // const 数据成员
public:
    MyClass(int x) : a(x) {}  // 初始化
};
```

### 4.2  const 成员函数和 const 对象

- const 成员函数不会修改类对象的数据成员
- const 类对象的数据成员不能修改，只能调用 const 成员函数

```cpp
class MyClass {
public:
    void func() const {  // const 成员函数
        // cannot modify member variables
    }
};

const MyClass obj;
obj.func();  // 正确，可以调用 const 成员函数
```

- 注意：const 修饰成员函数时，是放在 `()` 和 `{` 之间的

## 5  const 修饰函数

### 5.1  const 修饰函数返回值

函数返回值不能修改

### 5.2  const 修饰函数参数

函数参数在函数内部不能修改

### 5.3  const 可以作为重载要素

函数重载要求的是函数的参数列表不同

- 类型不同
- 参数数量不同
- 是否被 const 修饰

```cpp
void func(int a) {}
void func(const int a) {}
func(5); // 调用 void func(int a)
```

## 6  const_cast

- 用于去除 变量的 const 属性
