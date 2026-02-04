## 1  vector 原理与使用详解

### 1.1  原理

#### 1.1.1  vector 底层实现

![*1000](vector%20底层原理实现与扩容机制/file-20260107205953751.png)

##### 1.1.1 核心特性

- **动态数组**：vector 本质上是一个动态增长的数组
- **内存连续**：所有元素在内存中连续存储，支持随机访问
- **自动扩容**：当容量不足时自动分配更大的内存空间

##### 1.1.2 内部结构

vector 通常由**三个指针**组成：

1. **`_M_start`**：指向已分配内存的起始位置（或第一个元素）
2. **`_M_finish`**：指向最后一个元素的下一个位置（已使用内存的结束位置）
3. **`_M_end_of_storage`**：指向已分配内存的结束位置

通过这三个指针可以计算出：

- **size** = `_M_finish - _M_start`（元素数量）
- **capacity** = `_M_end_of_storage - _M_start`（总容量）

##### 1.1.3 内存特性

- **对象大小**：在 64 位系统中，vector 对象本身大小为 24 字节（3 个指针，每个 8 字节）
- **内存位置**：vector 对象可以位于栈或堆，但其**元素数据存储在堆上**
- **访问效率**：连续内存布局提供良好的缓存局部性，访问效率高

#### 1.1.2  vector 扩容机制

##### 1.2.1 扩容触发条件

当 `size == capacity` 时，下一次插入操作会触发扩容：

```cpp
if (size == capacity) {
    // 触发扩容
    reserve(capacity * growth_factor);
}
```

##### 1.2.2 扩容步骤

1. **分配新空间**：计算新容量，分配更大的内存块
2. **迁移数据**：将原有元素复制/移动到新内存
3. **释放旧空间**：释放原有内存
4. **更新指针**：调整三个指针指向新内存

##### 1.2.3 扩容倍率差异

| 编译器 | 扩容倍率 | 特点 |
|--------|----------|------|
| **Visual Studio** | 1.5 倍 | 空间利用率较高，可复用部分旧内存 |
| **GCC/Clang** | 2 倍 | 性能更优，但空间浪费稍大 |

##### 1.2.4 扩容策略分析

- **倍数扩容优点**：
  - 均摊时间复杂度为 O(1)
  - 减少频繁扩容带来的性能开销
- **倍数扩容缺点**：
  - 存在空间浪费（平均浪费约 25%-50%）
  - 内存碎片化问题

##### 1.2.5 技术细节

- **内存分配**：使用 `malloc` 或分配器分配原始内存
- **对象构造**：采用 **placement new** 在已分配内存上构造对象
- **移动语义**：
  - C++11 起，支持移动构造和移动赋值
  - 扩容时优先调用移动操作（如果元素类型支持）

#### 1.1.3  vector 迭代器失效

##### 1.3.1 失效场景

1. **插入操作**：
   - 在任意位置插入元素，**插入点之后**的迭代器失效
   - 如果触发扩容，**所有**迭代器失效

2. **删除操作**：
   - 删除任意位置元素，**删除点之后**的迭代器失效
   - 删除最后一个元素，仅尾后迭代器失效

3. **扩容操作**：
   - 任何导致重新分配内存的操作都会使**所有**迭代器失效

##### 1.3.2 安全建议

```cpp
// 危险：迭代器可能在操作后失效
auto it = vec.begin();
vec.push_back(value);  // 可能导致扩容
*it;  // 可能访问无效内存

// 安全：每次操作后重新获取迭代器
vec.push_back(value);
auto it = vec.begin();  // 重新获取
```

### 1.2  使用

#### 1.2.1  resize 和 reserve 的区别

| 特性 | `resize(n)` | `reserve(n)` |
|------|-------------|--------------|
| **影响** | 改变 `size()` | 改变 `capacity()` |
| **元素操作** | 可能增删元素 | 不改变元素数量 |
| **内存初始化** | 新元素会初始化 | 不初始化新内存 |
| **容量变化** | 可能触发扩容 | 确保容量至少为 n |
| **用途** | 调整元素数量 | 预留内存避免多次扩容 |

```cpp
vector<int> vec;
vec.reserve(100);   // 预留100个元素的空间，size=0
vec.resize(50);     // size=50，前50个元素值初始化
vec.resize(100);    // size=100，新增50个元素值初始化
```

#### 1.2.2  在 vector 中间插入元素

**操作步骤**：
1. **检查容量**：如果 `size == capacity`，先扩容
2. **移动元素**：将插入点后的所有元素向后移动
3. **构造元素**：在插入位置构造新元素
4. **更新大小**：`size++`

**时间复杂度**：O(n)，需要移动后续所有元素

```cpp
// 在位置pos插入元素value
void insert(vector<T>& vec, size_t pos, const T& value) {
    if (vec.size() == vec.capacity()) {
        // 扩容
    }
    // 移动元素
    for (size_t i = vec.size(); i > pos; --i) {
        vec[i] = move(vec[i-1]);
    }
    // 构造新元素
    new (&vec[pos]) T(value);
    vec._M_finish++;
}
```

#### 1.2.3  在 vector 中间删除元素

**操作步骤**：
1. **析构元素**：调用要删除元素的析构函数
2. **移动元素**：将删除点后的元素向前移动
3. **更新大小**：`size--`

**时间复杂度**：O(n)，需要移动后续所有元素

```cpp
// 删除位置pos的元素
void erase(vector<T>& vec, size_t pos) {
    // 析构要删除的元素
    vec[pos].~T();
    
    // 移动后续元素
    for (size_t i = pos; i < vec.size() - 1; ++i) {
        vec[i] = move(vec[i+1]);
    }
    
    // 更新大小
    vec._M_finish--;
}
```

#### 1.2.4  完全清空 vector 的方法

##### 2.4.1 方法比较

```cpp
vector<int> vec(100, 42);

// 方法1：clear() - 只清空元素，不释放内存
vec.clear();           // size=0, capacity不变

// 方法2：shrink_to_fit() - 请求释放未使用内存
vec.shrink_to_fit();   // capacity可能减小到size

// 方法3：swap技巧 - 彻底释放所有内存
vector<int>().swap(vec); // vec变为空，原内存完全释放

// 方法4：C++11移动赋值
vec = vector<int>();   // 同swap技巧
```

##### 2.4.2 推荐做法

```cpp
// 如果不再需要vector，彻底释放内存
vector<T>().swap(vec);

// 如果还要继续使用，但想释放多余内存
vec.clear();
vec.shrink_to_fit();
```

#### 1.2.5  push_back() 和 emplace_back() 的区别

##### 2.5.1 核心差异

- **`push_back`**：先构造对象，再拷贝/移动到 vector
- **`emplace_back`**：直接在 vector 内存中构造对象

##### 2.5.2 示例对比

```cpp
class Complex {
public:
    Complex(int a, int b, string c) { /* … */ }
};

vector<Complex> vec;

// push_back: 需要构造临时对象
vec.push_back(Complex(1, 2, "hello"));  // 构造临时对象 + 移动构造

// emplace_back: 直接构造，无临时对象
vec.emplace_back(1, 2, "hello");        // 直接构造，效率更高
```

##### 2.5.3 性能影响

- 对于简单类型（int, double 等）：两者性能相同
- 对于复杂类型：`emplace_back` 避免临时对象构造，性能更优

#### 1.2.6  vector erase 优化技巧

##### 2.6.1 传统删除（保持顺序）

```cpp
// 删除所有值为val的元素（保持顺序）
vec.erase(
    remove(vec.begin(), vec.end(), val),
    vec.end()
);
```

##### 2.6.2 快速删除（不保持顺序）

```cpp
// 快速删除位置i的元素（顺序可能改变）
template<typename T>
void unordered_erase(vector<T>& vec, size_t i) {
    if (i < vec.size() - 1) {
        swap(vec[i], vec.back());  // 与最后一个元素交换
    }
    vec.pop_back();                // 删除最后一个元素
}

// 删除所有满足条件的元素
for (size_t i = 0; i < vec.size();) {
    if (should_remove(vec[i])) {
        unordered_erase(vec, i);  // 不递增i
    } else {
        ++i;
    }
}
```

#### 1.2.7  C++11/14/17/20 新增功能

##### 2.7.1 C++11 新增

1. **移动语义**：移动构造和移动赋值
2. **原地构造**：`emplace()`, `emplace_back()`
3. **内存管理**：`shrink_to_fit()`
4. **新的迭代器**：`cbegin()`, `cend()`, `crbegin()`, `crend()`
5. **初始化列表**：`vector<int> vec = {1, 2, 3};`

##### 2.7.2 C++14/17/20 增强

- **constexpr 支持**：部分操作可在编译期计算
- **数据访问**：`data()` 成员函数
- **异常规范**：`noexcept` 标记

#### 1.2.8  构造函数效率对比

##### 2.8.1 构造函数类型

```cpp
// 1. 默认构造（最高效）
vector<T> vec1;

// 2. 带大小构造（次之）
vector<T> vec2(n);           // n个默认构造的元素

// 3. 带大小和值构造
vector<T> vec3(n, value);    // n个value的拷贝

// 4. 拷贝构造（深拷贝）
vector<T> vec4(other_vec);

// 5. 移动构造（最高效的"拷贝"）
vector<T> vec5(move(other_vec));

// 6. 初始化列表构造
vector<T> vec6 = {a, b, c};
```

##### 2.8.2 效率排序（从高到低）

1. **移动构造**：只转移资源所有权，O(1)
2. **默认构造**：无任何分配和构造
3. **带大小构造**：一次内存分配 + n 次默认构造
4. **移动赋值**：同移动构造
5. **拷贝构造**：深拷贝，O(n)

##### 2.8.3 使用建议

```cpp
// 返回vector的最佳实践（C++11起）
vector<int> create_vector() {
    vector<int> result;
    // … 填充数据
    return result;  // RVO或移动构造，无开销
}

// 传递vector到函数
void process(const vector<int>& vec);  // 只读访问，无拷贝
void process(vector<int>&& vec);       // 移动语义，转移所有权
```

### 1.3  最佳实践与注意事项

#### 1.3.1  预分配内存

```cpp
// 如果知道元素数量，预先分配内存
vector<int> vec;
vec.reserve(expected_size);  // 避免多次扩容
for (int i = 0; i < expected_size; ++i) {
    vec.push_back(i);
}
```

#### 1.3.2  元素类型选择

- **存储对象**：适合小型、频繁访问的类型
- **存储指针**：适合大型对象或多态类型
- **存储智能指针**：自动管理生命周期（推荐）

#### 1.3.3  避免迭代器失效

```cpp
// 危险：在遍历中修改容器
for (auto it = vec.begin(); it != vec.end(); ++it) {
    if (condition(*it)) {
        vec.erase(it);  // it失效，后续++it未定义
    }
}

// 正确：使用erase返回的新迭代器
for (auto it = vec.begin(); it != vec.end(); ) {
    if (condition(*it)) {
        it = vec.erase(it);  // erase返回下一个有效迭代器
    } else {
        ++it;
    }
}
```

#### 1.3.4  性能优化技巧

1. **使用 `reserve()`** 减少扩容次数
2. **优先使用 `emplace_back()`** 避免临时对象
3. **考虑使用 `std::array`** 如果大小固定
4. **批量操作** 使用算法库（`insert` 范围版本）

### 1.4  总结

vector 是 C++ 中最重要和最常用的容器之一，理解其内部原理和正确使用方式对于编写高效、安全的 C++ 代码至关重要。关键点包括：

1. **理解扩容机制**：避免不必要的扩容开销
2. **注意迭代器失效**：确保代码的安全性
3. **合理选择操作方法**：根据场景选择 `push_back`/`emplace_back`、`resize`/`reserve` 等
4. **利用现代 C++ 特性**：移动语义、原地构造等
5. **遵循最佳实践**：预分配内存、避免中间插入/删除等
  