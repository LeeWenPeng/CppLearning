C++ 11 后引入了 emplace_back，其为就地构造，不用构造后再拷贝到容器中，减少了内存拷贝和移动，因此效率更高

## 1  push_back

```cpp
vector<string> testVec;
testVec.push_back(string(16, 'a'));
```

两次构造一次析构

- string 进行一次构造
- vector 内会创建一个新的 string 对象，第二次构造
- string 构造的临时对象会被析构

## 2  emplace_back

直接在 vector 内构建一个新对象，省略一次构造和一次析构

## 3  push_back 和 emplace_back 的选择建议

- **默认使用 push_back**，因为它意图更清晰
- 当需要**传递构造参数**而非对象时，使用 **emplace_back**
- 在**性能敏感**且构造开销大的场景，优先使用 **emplace_back**
- 关注**异常安全**和**代码可读性**，必要时选择 **push_back**
