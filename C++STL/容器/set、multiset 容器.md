## 1  set、multiset 容器

特点：数据插入时，就会自动排序

本质：set 和 multiset 是关联式容器，底层是二叉树

区别：

- set 不允许有重复数据
- multiset 允许有重复数据

### 1.1  构造函数和赋值方式

#### 1.1.1  构造函数

set 容器存在两种构造方法：

1. 默认的无参构造
2. 拷贝构造

```cpp
set<typename Key, typename Compare, typename Alloc> s1;
```

- key 是 set 容器的数据类型
- compare 是 set 容器的比较器，默认是 less

#### 1.1.2  set 容器的赋值方式

使用 set 容器中重载的赋值运算符进行赋值

```c++
set<int> s1;
s1.insert(3);
s1.insert(4);
s1.insert(1);
s1.insert(2);
s1.insert(4);
s1.insert(4);

// 1. set 容器中插入数据，会在插入时排序
// 2. set 容器不允许有重复的值，所以后面两个 4 的 insert 被忽略
printSet(s1); // 1 2 3 4

set<int> s2 = s1; // 使用 = 赋值

set<int>s3(s1); // 拷贝构造
```

### 1.2  大小和交换

![image-20231008082051409](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231008082051409.png)

### 1.3  插入和删除

![image-20231008082303936](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231008082303936.png)

> 在 set 容器中，由于插入数据时，数据会自动排序，所以插入数据的顺序是没有太大意义的 (插入时不删除)，使用 `erase` 函数根据 `pos` 删除数据时，还是根据排好序之后的顺序进行删除，比如 `erase(s1.begin())` 一定是删除最小的元素

### 1.4  查找和统计

![image-20231008082740460](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231008082740460.png)

> 对于 set 容器而言，由于不允许重复数据，`count(key)` 的结果是 0 或者是 1

### 1.5  set 和 multiset 的区别

- set 不允许插入重复的值
- multiset 允许插入重复的值

#### 1.5.1  set 容器的 insert 函数

```c++
#if __cplusplus >= 201103L
      std::pair<iterator, bool> 
      insert(value_type&& __x)
      {
	std::pair<typename _Rep_type::iterator, bool> __p =
	  _M_t._M_insert_unique(std::move(__x));
	return std::pair<iterator, bool>(__p.first, __p.second);
      }
#endif
```

> 上述代码中可以看到：
>
> 1. insert 函数的返回值为 `pair<iterator, bool>`
> 
>    1. 第一个 iterator 返回的是插入数据的位置，也就是指向插入数据的迭代器
>    2. `bool ` 返回的是插入的数据是否重复，如果重复就插入失败
> 
> 2. set 容器的 insert 函数的本质是调用 `_M_t._M_insert_unique(std::move(__x));`
> 
> 唯一值插入

#### 1.5.2  multiset 容器的 insert 函数

```c++
#if __cplusplus >= 201103L
      iterator
      insert(value_type&& __x)
      { return _M_t._M_insert_equal(std::move(__x)); }
#endif
```

> 返回值是指向插入数据的迭代器，说明这里的插入函数没有进行重复值检查

#### 1.5.3  验证实验

```c++
#include <crtdefs.h>
#include <iostream>
#include <algorithm>
#include <stdlib.h>
#include <set>
using namespace std;

void printList(const set<int> &d1){
    for(typename set<int>::const_iterator it = d1.begin(); it != d1.end(); ++it){
        cout << *it << "\t";
    }
    cout << endl;
}

void test1(){
    set<int> s1;

    std::pair<set<int>::iterator, bool> ret = s1.insert(10);
    if(ret.second == 1){
        cout << "插入成功：" << *(ret.first) << endl;
    }else{
        cout << "插入失败" << endl;
    }

    ret = s1.insert(10);
    if(ret.second == 1){
        cout << "插入成功：" << *(ret.first) << endl;
    }else{
        cout << "插入失败" << endl;
    }

    multiset<int> s2;

    multiset<int>::iterator it = s2.insert(10);

    cout << "插入数据为：" << *it << endl;
    
}

int main(){
    test1();
    return 0;
}
```

### 1.6  pair 对组的创建和使用

#### 1.6.1  创建

![image-20231008085443460](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231008085443460.png)

> 1. 默认有参构造
> 2. 使用 `make_pair` 方法

#### 1.6.2  使用

1. `s1.first()` 获取对组的第一个数据
2. `s1.second()` 获取对组的第二个数据

```c++
#include <iostream>
#include <string>
#include <utility>
using namespace std;

void printPair(pair<string, int> &p){
    cout << "姓名：" << p.first << "\t年龄：" << p.second << endl; 
}
void test1(){
    
    pair<string, int> p1;
    p1 = {"张三", 19};
    printPair(p1);

    pair<string, int> p2("李四", 18);
    printPair(p2);

    pair<string, int> p3 = make_pair("王五", 20);
    printPair(p3);
}

int main(){
    test1();
    return 0;
}
```

### 1.7  set 比较器

set 模板的模板参数有三个：

```c++
set<typename Key, typename Compare, typename Alloc>
```

![image-20231008092311974](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231008092311974.png)

如上图所示：

1. 声明的 set 容器的数据类型 `key`
2. 比较器 `Compare`，默认值为 `less`
3. `Alloc ` 是 set 容器的分配器

比较器的功能为设置排序规则，主要体现在两方面：

1. 设置排序的规则
2. 设置排序的数据

> 1. 针对于第一条，体现在我们可以使 set 容器的数据从升序改为降序
> 2. 针对于第二条，体现在对于结构化数据，我们可以选择其中某一些数据作为排序数据
> 3. 第一条和第二条结合，可以使排序规则更多样，比如设置针对于哪一种数据作为第一排序数据，排序方式为升序或是降序，设置哪一个数据为第二排序数据，排序方式为升序或是降序等等

> ==这里的比较器设置和标准算法中的 sort 中的比较器设置本质上是一样的==，区别是：
>
> - 标准算法 sort 中的比较器是函数的参数，所以定义的比较器可以是函数，也可以是仿函数
>   - 如果定义的比较器是函数，sort 函数的参数为 "`比较器的函数名`"
>   - 如果定义的比较器是仿函数，sort 函数的参数为 "`比较器的类名()`"
> - 而这里的比较器，放在了 set 模板中的模板参数中，模板参数一定是类型名，所以，这里的比较器只能使用仿函数，写入的参数为 "`比较器的类名`"

```c++
#include <iostream>
#include <set>
#include <string>
#include <utility>
using namespace std;

class MyCompare{
    public:
    // 需要定义成常函数
    bool operator() (int a, int b)const {
        return (a > b) ;
    }
};
void printSet(set<int, MyCompare> &s){
    for(set<int, MyCompare>::iterator it = s.begin(); it!=s.end(); ++it){
        cout << *it << " ";
    }
    cout << endl;
}
void test1(){
    set<int,MyCompare> s1;

    s1.insert(10);
    s1.insert(20);
    s1.insert(30);
    s1.insert(40);

    printSet(s1); // 40 30 20 10 降序
    
}

int main(){
    test1();
    return 0;
}
```

比较器：使用的是仿函数

```c++
class MyCompare{
    public:
    // 需要定义成常函数
    bool operator() (int a, int b)const {
        return (a > b) ;
    }
};
```

如果上述代码中的比较器，没有定义成常函数，则会报如下错误

![image-20231008091310918](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231008091310918.png)

报错中明确指出，比较器需要是 `const` 类型
