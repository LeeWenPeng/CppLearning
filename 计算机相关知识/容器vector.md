# 容器 `vector`

## １　定义和初始化

```c++
// 默认初始化vector，创建一个指定的空vector对象
vector <T> v1;

// 拷贝创建
vector <T> v2(v1);
vector <T> v2 = v1;

//创建指定数量的元素
vector <T> v3(n, val); //创建n个val值的v3

vector <T> v4(n);//创建具有n个<T>类型的对象，每个对象的值为

// 列表初始化 C++11
vector <T> v5{a, b, c, ...};
vector <T> v5 = {a, b, c, ...};
```

1. 使用`=`进行初始化，实际上执行的是拷贝初始化，把等号右边的拷贝到新创建的对象中

   1. 本质上，是因为 `vector` 容器提供了 `=` 运算符的重写

2. vector初始化主要分为使用**花括号**和**圆括号**两种。

   1. 花括号：表示想要列表初始化，也就是为新创建的对象提供初始值
      
      > 如果花括号提供的数值无法作为列表初始化值，则会将这些数值作为构造函数参数使用
   2. 圆括号：表示想要提供构造参数

**c++11新特性**：

1. `vector`声明写法：
   当`vector`内的对象为`vector`对象时
    + 旧写法：`vector<vector<int> >`，右尖括号和元素对象类型之间必须要有一个空格
    + 新写法：`vector<vector<int>>`
2. `vector`列表初始化

## 2 添加元素——`push_back()`

作用：将一个值当作`vector`对象的尾元素，压到(push)`vector`对象的尾部(back)。

语法：

```c++
vector<int> v1;
v1.push_back(1);
```

**注意：**

1. `vector` **使用技巧**

    除非要创建的`vector`对象中的所有元素值都一样，否则在创建时不应该预设`vector`的大小

2. `vector`容器**如何申请内存空间**

   1. `vector`容器只会在**执行`insert`操作，`size`和`capacity`相等时**，或者**调用`resize`和`reserve`时给定大小超过当前的`capacity`**这两情况时，才可能重新分配内存空间

        > 也就是当需求大于当前容量时, `vector`容器才会重新分配内存空间

   2. `vector`每次分配内存空间总是**将当前容量翻倍**

      > 只有迫不得已的时候才可以分配新的内存空间

   3. 标准库实现中`vector`增长的具体情况为：
      1. 默认的内存分配机制

            ０　１　２　４　８

      2. 使用`reserve(n)`修改预分配内存空间

         $$
         \text{每次新增空间的大小}　>= \max(ivec.capacity(), n)
         $$

## 3 `vector`关于分配内存空间的几个方法

### 1 `size()`和`capacity()`方法

`size()`方法

作用：查询容器中已有元素个数

`capacity()`方法

作用：查询容器当前内存所能容纳的元素个数

二者区别：

对于`vector`容器`ivec`而言，$ivec.capacity()　= ivec.size() + 预留空间$，其中预留空间大于等于０．

当容器的`capacity`等于它的`size`时，如果再执行`insert`操作，则需要重新为这个容器分配内存空间，之后，`capacity`又将大于`size`

所以，一个容器的`capacity`一定大于等于它的`size`

### 2 `resize()`和`reserve()`方法

`resize(n)`方法

作用：重新设置容器可以容纳元素的大小为`n`

`reserve(n)`方法

作用：修改容器预分配空间大小**至少**为`n`个元素大小

这两个方法都不回收容器所占内存空间

### 3 `shrink_to_fit()`方法

`shrink_to_fit()`

作用：请求回收预留空间

> 不一定生效
>
> 如果生效，那么之后，`ivec.capacity() == ivec.size()`
>

## 4 vector 容器其余操作

1　删除操作

