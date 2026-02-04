## 1  list 容器

list 是一个双向循环链表

> 对数据结构上链表的实现

![image-20231007105627546](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007105627546.png)

链表不是连续的存储空间，所以链表的迭代器是双向迭代器，支持向前遍历和向后遍历，但不支持随机访问

链表的优点：

- 内存是动态存储分配，不会造成内存溢出和内存泄漏
- 插入删除操作是时间复杂度为 1 的操作，只需要修改对应节点指针

缺点：

- 不支持随机访问，遍历需要挨个元素遍历
- 需要额外存储指针，空间消耗相对较大

### 1.1  构造函数

![image-20231007105959488](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007105959488.png)

> 与其他 STL 容器， 比如 vector 容器的构造方法一样

### 1.2  赋值操作和交换

![image-20231007110138970](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007110138970.png)

> 和 vector 容器一样

swap 接口的两种调用方法：

1. `L1.swap(L2)`
2. `swap(L1, L2)`

### 1.3  大小

![image-20231007110515014](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007110515014.png)

### 1.4  插入删除

![image-20231007122701966](../../第一阶段黑马程序员C++课程/lesson06%20-%20c++%20提高编程.assets/image-20231007122701966.png)

`remove(elem)` 会删除容器中所有与 `elem` 值匹配的元素

### 1.5  数据存取

由于链表不是存储内存不连续，所以无法通过运算符 `[]` 或者是 `at` 方法访问其元素

只能通过双向迭代器，或者是指针逐个遍历，或者是 `list` 容器提供的接口：

- `front()`
- `back()`

### 1.6  翻转和排序

- `reserve()` 翻转链表
- `sort()` 链表排序

#### 1.6.1  sort 函数

是 list 容器中的成员函数，默认是升序排序，可以提供比较器的方式，修改排序为降序

```c++
#include <crtdefs.h>
#include <iostream>
#include <deque>
#include <algorithm>
#include <list>
#include <stdlib.h>
#include <vector>
using namespace std;

class Person{
    public:
    Person(string s){
        this->m_name = s;
        this->m_score = -1; // 初始为 -1
    }
    Person(string s, int score){
        this->m_name = s;
        this->m_score = score; 
    }

    string m_name;
    int m_score;
};

void printList(const list<Person> &d1){
    for(typename list<Person>::const_iterator it = d1.begin(); it != d1.end(); ++it){
        cout << it->m_name << " : " << it->m_score << " \t";
    }
    cout << endl;
}

bool myCompara(Person &p1, Person &p2){

    return p1.m_score > p2.m_score;

}

void test1(){
    list<Person> l1;

    Person p1("李一", 12);
    Person p2("李二", 52);
    Person p3("李三", 21);
    Person p4("李四", 64);
    Person p5("李五", 19);

    l1.push_back(p1);
    l1.push_back(p2);
    l1.push_back(p3);
    l1.push_back(p4);
    l1.push_back(p5);

    printList(l1);
    l1.sort(myCompara);
    printList(l1);
}

int main(){
    test1();
    return 0;
}
```

> 1. 比较器其实就是比较规则，比如
> 
>    ```c++
>    bool myCompare(int a, int b){
>        return a>b;
>    }
>    ```
> 
> 上述代码就很明显，如果 `a>b`，则返回真，否则返回假
>
> 这个比较器提供的比较规则就是前面的数据要大于后面的数据，换而言之，这个排序应该是降序排序
>
> 1. 如果列表中存储的是自定义数据类型，同样可以通过提供比较器的方式进行排序，比如上述的代码：
> 
>    ```c++
>    bool myCompara(Person &p1, Person &p2){
>        return p1.m_score > p2.m_score;
>    }
>    ```
> 
> ==对于自定义数据类型的列表，则必须要提供比较器，否则会报错==
>
> 1. 通过比较器可以提供更为复杂的规则，比如
> 
>    ```c++
>    bool myCompara(Person &p1, Person &p2){
>        // 先按照分数进行降序排序，再按照年龄升序排序
>        if(p1.m_score == p2.m_score){
>            return p1.m_age < p2.m_age;
>        }
>        return p1.m_score > p2.m_score;
>    
>    }
>    ```
> 
> 2. 这里是 list 容器中的 sort 函数，这个函数的参数
