## 1  deque 容器

双端数组，可以在头部和尾部都进行插入和删除

> 如果是 `vector` 容器进行在头部的插入操作，就需要重新开辟空间；同时，对于 `vector` 容器，删除操作无论开不开辟新的空间，都将会是一个 $\Omicron(n) $ 级别的操作。
>
> 所以：
>
> 1. 如果是需要进行在头部的插入删除操作，`deque` 是比 `vector` 更优选的操作
> 2. `vector` 访问元素的速度要快于 `deque`，这是因为二者的内部实现不同

![image-20231007085221168](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007085221168.png)

> 提供的接口和 vector 基本一致，唯一区别是 `deque` 可以进行头部的插入删除

### 1.1  构造函数

> 和 vector 构造方式一样

```c++
// 静态迭代器
// 如果在函数入口处用 `const` 关键字限定容器不可以修改，那么使用的容器的迭代器也应该是 `const` 版
// 因为可以简单把迭代器当作容器的指针，那就很显然了，如果使用正常的迭代器类型区遍历`const`类型容器，显然会存在一个类型不符的错误
// 必须要记得， const 是可以作为函数重载的条件的
#include <iostream>
#include <deque>
using namespace std;

template <typename T>
void printDeque(const deque<T> &d1){
    // 注意！下一行代码中的 it 的类型 不再是 typename deque<T>::iterator
    // 而是 typename deque<T>::const_iterator 静态迭代器
    for(typename deque<T>::const_iterator it = d1.begin(); it != d1.end(); ++it){
        cout << *it << "\t";
    }
    cout << endl;
}

void test1(){
    deque<int> d1;
    for(int i = 0; i < 10; ++i){
        d1.push_back(i);
    }
    printDeque(d1);
}

int main(){
    test1();
    return 0;
}
```

### 1.2  赋值

![image-20231007092240721](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007092240721.png)

> 和 vector 容器基本一样

### 1.3  deque 容器大小

![image-20231007092359246](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007092359246.png)

> - 和 vector 操作几乎一样，区别是 deque 容器没有容量的概念
> - 一共三种操作：
>   - 判空 `empty`
>   - 返回大小 `size`
>   - 改变大小 `resize`

### 1.4  插入删除

![image-20231007092630998](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007092630998.png)

> 上述所有操作中的位置是迭代器

### 1.5  数据存取

> 和 vector 操作一样

1. 迭代器
2. 操作符 `[]` 的重载
3. `at` 函数
4. 两个特殊接口：
   1. `front()`
   2. `back()`

### 1.6  排序

使用 `STL algorithm` 中的 `sort` 函数

![image-20231007093408473](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007093408473.png)

```c++
#include <iostream>
#include <deque>
#include <algorithm>
using namespace std;

template <typename T>
void printDeque(const deque<T> &d1){
    for(typename deque<T>::const_iterator it = d1.begin(); it != d1.end(); ++it){
        cout << *it << "\t";
    }
    cout << endl;
}

void test1(){
    deque<int> d1;
    d1.push_back(1);
    d1.push_back(2);
    d1.push_back(3);
    d1.push_front(100);
    d1.push_front(200);
    d1.push_front(300);
    printDeque(d1); // 300 200 100 1 2 3

    //排序 需要包含头文件 <algorithm>
    // sort(ideque.begin(), ideque.end());
    // 默认是从小到大排序
    sort(d1.begin(), d1.end());

    printDeque(d1); // 1       2       3       100     200     300
}

int main(){
    test1();
    return 0;
}
```

> 支持随机访问迭代器的容器都可以使用这个 `sort` 函数进行排序，比如 `vector`
>
> 补充：==所有不支持随机访问迭代器的容器，都不可以使用标准算法==
