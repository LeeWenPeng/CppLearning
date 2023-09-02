# C++ 基础语法

## 1、在 main 执行之前和 main 执行之后执行的代码可能是什么？

main 函数**执行之前**，主要是初始化系统相关资源:

+ 设置栈指针
+ 初始化静态 static 变量和 global 全局变量, 即 **.data** 段的内容
+ 将未初始化的全局变量赋初值：数值型 short, int, long等为 0,  bool 为 FALSE, 指针为 null 等等, 即为 **.bss** 段的内容
+ 全局对象初始化, 在 main 之前调用构造函数, 这是可能会执行的一些代码,
+ 将 main 函数的参数 `argc`, `argv` 等传递给 main 函数, 然后才真正运行 main 函数
+ `__attribute__((constructor))`

main 函数执行之后:

+ 全局对象的析构函数会在 main 函数之后执行;
+ 可以用 atexit 注册一个函数, 它会在 main 之后执行;
+ `__attribute__((destructor))`