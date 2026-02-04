## 1  参考文件

[线程手册](https://zh.cppreference.com/w/cpp/thread/thread.html)

## 2  头文件

```cpp
#include <thread>
```

## 3  语法

```cpp

/* 
	默认构造
*/
thread() noexcept;

/* 
	移动构造
*/
thread( thread&& other ) noexcept;

/* 
	初始化构造函数 
	参数
		fn 可调用对象
		args fn 的参数
		
	细节
	- explicit 防止隐式转换
	- F&& 万能引用
	- Args&&... 可变参数，万能引用
*/
template< class F, class… Args >  
explicit thread( F&& f, Args&&… args );

/*
	拷贝构造
	拷贝构造函数被禁用，thread 不允许拷贝构造
*/
thread( const thread& ) = delete;

```

## 4  std::thread 成员函数详解

### 4.1  线程状态生命周期

```cpp
创建线程 → 可连接(joinable) → { join() 或 detach() } → 不可连接(!joinable)
    ↑
默认构造
（不可连接）
```

### 4.2  成员函数详细说明

#### 4.2.1  `get_id()` - 获取线程标识符

```cpp
std::thread::id get_id() const noexcept;
```

**作用**：
- 返回当前线程对象所关联的线程的标识符
- 如果线程对象没有关联任何执行线程，则返回默认构造的 `std::thread::id`（表示 " 没有线程 "）

**返回值**：
- `std::thread::id` 类型对象，表示线程的唯一标识符
- 可比较、可哈希、可输出

**示例**：

```cpp
#include <iostream>
#include <thread>
#include <vector>

void worker(int id) {
    std::cout << "线程 " << id << " 的ID: " << std::this_thread::get_id() << std::endl;
}

int main() {
    std::thread t1(worker, 1);
    std::thread t2(worker, 2);
    
    std::cout << "主线程ID: " << std::this_thread::get_id() << std::endl;
    std::cout << "t1线程ID: " << t1.get_id() << std::endl;
    std::cout << "t2线程ID: " << t2.get_id() << std::endl;
    
    t1.join();
    t2.join();
    
    // join后获取ID
    std::cout << "join后t1线程ID: " << t1.get_id() << std::endl;  // 输出: 0 或 thread::id of a non-executing thread
    return 0;
}
```

#### 4.2.2  `joinable()` - 检查线程是否可连接

```cpp
bool joinable() const noexcept;
```

**作用**：
- 检查线程对象是否标识一个活跃的执行线程
- 即判断是否可以对线程调用 `join()` 或 `detach()`

**返回值**：
- `true`：线程对象关联着一个活跃的线程，可以调用 `join()` 或 `detach()`
- `false`：线程对象没有关联任何线程，不能调用 `join()` 或 `detach()`

**为 `false` 的情况**：
1. 默认构造的线程对象（未关联任何线程）
2. 已被移动的线程对象（资源已转移）
3. 已调用过 `join()` 的线程
4. 已调用过 `detach()` 的线程

**示例**：

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void task() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
}

int main() {
    std::thread t1;  // 默认构造，不可连接
    std::cout << "t1 joinable? " << (t1.joinable() ? "yes" : "no") << std::endl;  // no
    
    std::thread t2(task);  // 关联线程，可连接
    std::cout << "t2 joinable? " << (t2.joinable() ? "yes" : "no") << std::endl;  // yes
    
    t2.join();  // 等待线程结束
    std::cout << "After join, t2 joinable? " << (t2.joinable() ? "yes" : "no") << std::endl;  // no
    
    // 错误示例：对不可连接的线程调用join
    // t1.join();  // 会抛出 std::system_error
    
    return 0;
}
```

#### 4.2.3  `join()` - 等待线程完成

```cpp
void join();
```

**作用**：
- 阻塞当前线程，直到 `*this` 所标识的线程执行完毕
- 调用后，线程对象变为不可连接状态

**重要规则**：
1. 一个线程只能被 `join()` 一次
2. 必须在 `joinable()` 为 `true` 时调用
3. 线程对象析构前必须调用 `join()` 或 `detach()`，否则程序会终止（调用 `std::terminate()`）

**异常**：
- 如果线程不可连接，抛出 `std::system_error` 异常
- 如果当前线程不能等待给定线程，抛出 `std::system_error` 异常

**示例**：

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void slow_task(int id) {
    std::cout << "线程 " << id << " 开始工作…" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "线程 " << id << " 完成工作!" << std::endl;
}

int main() {
    std::cout << "主线程启动" << std::endl;
    
    std::thread worker(slow_task, 1);
    
    std::cout << "主线程正在等待工作线程完成…" << std::endl;
    
    if (worker.joinable()) {
        worker.join();  // 主线程在此阻塞，直到worker完成
    }
    
    std::cout << "工作线程已完成，主线程继续" << std::endl;
    
    // 多个线程顺序执行
    std::thread t1(slow_task, 1);
    t1.join();  // 等待t1完成
    
    std::thread t2(slow_task, 2);
    t2.join();  // 等待t2完成
    
    return 0;
}
```

#### 4.2.4  `detach()` - 分离线程

```cpp
void detach();
```

**作用**：
- 将线程对象与其关联的执行线程分离
- 分离后，线程在后台独立运行（守护线程）
- 调用后，线程对象变为不可连接状态

**特点**：
1. 分离的线程无法再被 `join()`
2. 分离的线程无法调用 `get_id()`
3. 分离的线程调用 `joinable()`，会得到结果 false
4. 分离的线程在退出时会自动释放资源
5. 主线程退出时，所有未完成的分离线程会被终止

**使用场景**：
- **不需要等待结果**的**长时间运行任务**
- 日志记录、监控、后台清理等任务
- 守护进程

**警告**：
- 确保分离的线程不会访问已销毁的对象（如局部变量）
- 避免数据竞争和悬空指针
- 在 C++ 程序中，当主线程（`main` 函数）结束时：
	- **所有未完成的 `detach` 线程都会被强制终止**
	- C++ 运行时不会等待这些后台线程完成
	- 程序直接退出，所有资源被回收
**示例**：

```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <vector>

void background_task(int id) {
    for (int i = 0; i < 5; ++i) {
        std::cout << "后台线程 " << id << ": " << i << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    std::cout << "后台线程 " << id << " 结束" << std::endl;
}

int main() {
    std::cout << "主线程开始" << std::endl;
    
    // 正确：使用全局变量或堆分配内存
    static int shared_data = 42;
    
    std::thread t1([&shared_data]() {
        for (int i = 0; i < 3; ++i) {
            std::cout << "t1: shared_data = " << shared_data << std::endl;
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    });
    
    t1.detach();  // 分离线程，使其在后台运行
    
    // 错误示例：访问局部变量（危险！）
    {
        int local_var = 100;
        std::thread t2([&local_var]() {
            // 危险！local_var可能在t2运行时已被销毁
            std::this_thread::sleep_for(std::chrono::seconds(1));
            // 访问已销毁的local_var -> 未定义行为！
            // std::cout << local_var << std::endl;
        });
        t2.detach();
    }  // local_var在此被销毁，但t2可能还在运行
    
    std::cout << "主线程继续执行…" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    
    std::cout << "主线程结束" << std::endl;
    // 注意：主线程结束时，所有分离的线程会被终止
    
    return 0;
}
```

### 4.3  this_thread 分组

[this_thread CPP 手册](cplusplus.com/reference/thread/this_thread/)

#### 4.3.1  get_id

```cpp
std::this_thread::get_id()
```

作用：返回当前线程 ID

示例

```cpp
#include <thread>
#include <chrono>
#include <iostream>

std::thread::id main_thread_id = std::this_thread::get_id();

void is_main_thread(){
    if(main_thread_id == std::this_thread::get_id()){
        std::cout << "this is main thread" << std::endl;
    }else{
        std::cout << "this is not the main thread" << std::endl;
    }
}

int main(){
    is_main_thread();
    std::thread t(is_main_thread);
    t.join();
    return 0;
}
/*
output:
    this is main thread
    this is not the main thread
*/
```

#### 4.3.2  yield

作用：让出 CPU，避免空转浪费资源。线程告诉调度器：" 我现在没事做，先运行其他线程吧 "

示例

```cpp
#include <thread>
#include <atomic>
#include <iostream>

std::atomic<bool> ready(false); // 原子变量，确保多线程环境下的安全访问

void count1m(int id){
    while (!ready) {
        std::this_thread::yield(); // 让出 CPU 时间片
    }
    for(volatile int i = 0; i < 1000000; ++i){} // 模拟耗时任务; volatile 禁止编译器优化
    std::cout << id;
}

int main(){
    std::thread threads[10];
    std::cout << "race of 10 threads that count to 1 million\n";

    for(int i  = 0; i < 10; ++i) threads[i] = std::thread(count1m, i);
    ready = true;
    for(auto &t : threads) t.join();
    std::cout << "\n";
    return 0;
}
/*
output:
    this is main thread
    this is not the main thread
*/
```

#### 4.3.3  sleep_for

```cpp
template< class Rep, class Period >
void sleep_for( const std::chrono::duration<Rep, Period>& sleep_duration );
```

作用：在指定的 sleep_duration 内阻塞当前线程

- 由于调度或资源争用的延迟，阻塞时间可能超过 sleep_duration
- 建议使用稳定时钟，而非系统时钟

参数

- sleep_duration：阻塞时长

示例

```cpp
#include <chrono>
#include <iostream>
#include <thread>
 
int main()
{
    using namespace std::chrono_literals;
 
    std::cout << "Hello waiter\n" << std::flush;
 
    const auto start = std::chrono::high_resolution_clock::now();
    std::this_thread::sleep_for(2000ms);
    const auto end = std::chrono::high_resolution_clock::now();
    const std::chrono::duration<double, std::milli> elapsed = end - start;
 
    std::cout << "Waited " << elapsed.count() << "ms\n";
}

/*
    output
        Hello waiter
        Waited 2000.06ms
*/
```

#### 4.3.4  sleep_until

```cpp
template< class Clock, class Duration >  
void sleep_until( const [std::chrono::time_point]<Clock, Duration>& sleep_time );
```

- 作用：阻止当前线程的执行，直到达到指定的 sleep_time。
	- 时钟必须符合时钟要求。
	- 建议使用与 sleep_time 绑定的时钟，
- 参数
	- sleep_time：阻塞结束的时间
示例

```cpp
#include <chrono>
#include <iostream>
#include <ratio>
#include <thread>

// 获取稳定时钟的当前时间
auto now() { 
    return std::chrono::steady_clock::now(); 
}

// 计算 sleep_time
auto awake_time(){
    using std::chrono::operator""ms;
    return now() + 2000ms;
}

int main(){
    std::cout << "Hello Waiter!\n" << std::flush;
    
    const auto start{now()};
    std::this_thread::sleep_until(awake_time());
    std::chrono::duration<double, std::milli> elapsed{ now() - start }; // 计算阻塞时间

    std::cout << "Waited " << elapsed.count() << "ms\n";
}

/*
    output
        Hello Waiter!
        Waited 2000.17ms
*/
```

### 4.4  完整示例：综合使用

```cpp
#include <iostream>
#include <thread>
#include <chrono>
#include <vector>
#include <mutex>

std::mutex cout_mutex;

void print_safe(const std::string& msg) {
    std::lock_guard<std::mutex> lock(cout_mutex);
    std::cout << msg << std::endl;
}

void time_consuming_task(int id) {
    print_safe("线程 " + std::to_string(id) + " 开始");
    std::this_thread::sleep_for(std::chrono::seconds(id));
    print_safe("线程 " + std::to_string(id) + " 结束");
}

int main() {
    std::cout << "=== 线程管理示例 ===" << std::endl;
    
    // 1. 创建线程并获取ID
    std::thread t1(time_consuming_task, 1);
    std::cout << "t1 ID: " << t1.get_id() << std::endl;
    
    // 2. 检查是否可连接
    if (t1.joinable()) {
        std::cout << "t1 is joinable" << std::endl;
    }
    
    // 3. 创建多个线程
    std::vector<std::thread> threads;
    for (int i = 2; i <= 4; ++i) {
        threads.emplace_back(time_consuming_task, i);
    }
    
    // 4. 分离一个线程（后台运行）
    std::thread background([]() {
        for (int i = 0; i < 3; ++i) {
            print_safe("后台线程运行中…");
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
        print_safe("后台线程结束");
    });
    background.detach();
    
    // 5. 等待所有线程完成（除已分离的）
    t1.join();
    print_safe("t1 已join");
    
    for (auto& t : threads) {
        t.join();
    }
    print_safe("所有线程已join");
    
    // 6. 检查线程状态
    std::thread temp(time_consuming_task, 0);
    std::cout << "join前，temp joinable? " << (temp.joinable() ? "yes" : "no") << std::endl;
    temp.join();
    std::cout << "join后，temp joinable? " << (temp.joinable() ? "yes" : "no") << std::endl;
    std::cout << "join后，temp ID: " << temp.get_id() << std::endl;
    
    std::cout << "=== 主线程结束 ===" << std::endl;
    
    // 主线程暂停，以便看到后台线程的输出
    std::this_thread::sleep_for(std::chrono::seconds(4));
    
    return 0;
}
```

### 4.5  最佳实践与注意事项

#### 4.5.1  必须遵循的规则

```cpp
std::thread t(some_function);

// 在t销毁前，必须满足以下条件之一：
if (t.joinable()) {
    t.join();    // 选项1：等待线程完成
    // 或
    t.detach();  // 选项2：分离线程
}
// 否则，std::terminate() 会被调用
```

#### 4.5.2  RAII 包装类示例

```cpp
class ThreadGuard {
    std::thread& t;
public:
    explicit ThreadGuard(std::thread& t_) : t(t_) {}
    
    ~ThreadGuard() {
        if (t.joinable()) {
            t.join();  // 或根据需求选择其他策略
        }
    }
    
    ThreadGuard(const ThreadGuard&) = delete;
    ThreadGuard& operator=(const ThreadGuard&) = delete;
};

void use_thread_guard() {
    std::thread t([]() {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        std::cout << "任务完成" << std::endl;
    });
    
    ThreadGuard g(t);  // 确保线程会被正确join
    // 即使发生异常，g的析构函数也会调用t.join()
}
```

#### 4.5.3  错误处理模式

```cpp
void safe_thread_operation() {
    std::thread t([]() { /* 任务代码 */ });
    
    try {
        // 可能抛出异常的操作
        do_something_risky();
        
        if (t.joinable()) {
            t.join();
        }
    } catch (…) {
        if (t.joinable()) {
            t.join();  // 异常时也确保join
        }
        throw;  // 重新抛出异常
    }
}
```

### 4.6  常见问题解答

#### 4.6.1  Q1: `join()` 和 `detach()` 的区别是什么？

**A1**:
- `join()`: 等待线程完成，线程对象变为不可连接
- `detach()`: 线程分离运行，线程对象变为不可连接，线程结束后自动清理

#### 4.6.2  Q2: 为什么析构时要调用 `join()` 或 `detach()`？

**A2**: 这是 C++ 线程安全模型的一部分，确保不会意外地让线程在后台运行并访问已销毁的数据。

#### 4.6.3  Q3: 如何避免悬空引用？

**A3**:
- 使用值捕获而不是引用捕获
- 将数据移到线程中（C++11 以后）
- 使用 `std::shared_ptr` 共享数据
- 确保线程生命周期不超过数据生命周期

#### 4.6.4  Q4: 何时使用 `detach()`？

**A4**: 当线程执行独立的后台任务，且不需要与主线程同步时使用。但要确保线程不会访问即将销毁的资源。

这个详细的文档涵盖了 `std::thread` 的主要成员函数，包括正确用法、常见陷阱和最佳实践。

## 5  thread 构造

构造 thread 时，需要传入线程函数

- 普通全局/静态函数
- lambda 表达式
- 类成员函数

### 5.1  普通全局/静态函数

```cpp
std::thread t(&func, arg);
```

参数

- `&func`：函数地址。由于函数名就是函数地址，因此，这里的 `&` 可以省略
- `arg`：函数参数

### 5.2  lambda 表达式

```cpp
std::thread t([&]{ obj.func(arg); });
```

参数

- `[&]{ obj.func(arg);`： lambda 表达式

### 5.3  类成员函数

```cpp
std::thread t(&T::func, &obj, arg);
```

参数

- `&T::func`：成员函数地址
- `&obj`：调用成员函数的对象地址
- `arg`：成员函数参数

## 6  简单线程创建

```cpp
#include <chrono>
#include <thread>
#include <iostream>
using namespace std;

// 不传入参数
void func1(){
    cout << "func1" << endl;
}

// 传入 2 个值
void func2(int a, int b){
    cout << "func2 a + b = "<< a + b << endl;
}

// 传入引用
void func3(int &c){
    cout << "func3 c = " << &c << endl;
    c += 10;
}

class A{
public:
    void setName(string s){
        this->_name = s;
    }
    void func4(int a){
        cout << "func4 name = " << _name << ", age = " << a <<endl; 
    }
private:
    string _name;
};

// detach
void func5(){
    cout << "func5 into sleep" << endl;
    std::this_thread::sleep_for(std::chrono::seconds(1)); // 休眠
    cout << "func5 leave" << endl;
}

// move
void func6(){
    cout << "this is func6" << endl;
}

int main(){
    // 1. 传入 0 个值
    cout << "main1\n";
    std::thread t1(&func1); // 只传递函数
    t1.join(); // 阻塞等待执行结束

    // 2.  传入两个值
    cout << "\n\nmain2\n";
    int a = 10;
    int b = 20;
    std::thread t2(&func2, a, b);
    t2.join();

    // 3. 传入引用
    cout << "\n\nmain3\n";
    int c = 110;
    std::thread t3(&func3, std::ref(c)); // 传入引用需要使用 ref
    t3.join();
    cout << "c = " << &c << ": " << c << endl;

    // 4. 传入类函数
    cout << "\n\nmain4\n";
    A* a_ptr = new A();
    a_ptr->setName("darren");
    std::thread t4(&A::func4, a_ptr, 20); // 取地址符不能省略
    t4.join();

    // 5. detach
    cout << "\n\nmain5\n";
    std::thread t5(func5); // 由于函数名指向函数首地址，因此，这里 & 符号可以省略
    t5.detach(); // 脱离
    cout << "main5 end" << endl;
    // 打印结果
    // main5
    // main5 end
    // func5 into sleep
    // 问题：缺失 func5 leave
    // 分析：detach，如果主线程退出，则子线程直接退出

    // 6. move
    cout << "\n\nmain6\n";
    std::thread t6_1(func6); 
    std::thread t6_2(std::move(t6_1));  // 将 t6_1 所有权移交给 t6_2
    t6_1.join(); // terminate called after throwing an instance of 'std::system_error'
    t6_2.join(); 
}
```
