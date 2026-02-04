## 1  Redis 线程

- redis-server
	- 所有的命令处理
	- 网络事件监听
- bio-close-file：异步关闭大文件
- bio-aof-fsync：异步 aof 刷盘
- bio-lazy-free：异步删除大内存
- jemalloc-bg-threads：jemalloc 后台线程，帮助管理内存
- io-threads：io 线程，read\write、decode、encode

## 2  结论

- Redis 本身不是单线程
- 但是命令处理流程都在一个线程 (redis-server) 中处理的
- 所以，可以勉强说其是单线程

## 3  为什么 redis 的命令是单线程的

- redis 并非 CPU 密集型，因为 redis 是内存数据库
- 因此不需要多线程

## 4  如果采用多线程，有什么问题？

- 加锁复杂，锁的粒度不好控制。
	- 因为 redis 具有多个对象类型，而且每个对象类型具有多种数据结构实现
- 频繁的 CPU 上下文切换，抵消多线程优点，而且使编程复杂度提高
- 单线程的局限
	- 不能够有耗时操作，以防止影响响应性能

## 5  单线程为什么很快？采用了什么机制

- 内存数据库
- 数据结构高效
	- string
		- int
		- raw
		- embstr
	- list
		- quick：节点采用 ziplist
	- hash
		- ziplist：小数据量
		- dict：大数据量
	- set
		- intset：整数
		- dict：大数据量
	- zset
		- ziplist：小数据量
		- skiplist：大数据量
- reactor 网络模型
	- IO 多路复用
	- 非阻塞 IO
- 优化：
	- 耗时阻塞操作，另起线程处理
		- 异步关闭大文件，主要是 RDB 文件的关闭
		- 异步释放大内存
		- 异步刷盘，主要是指 aof 刷盘
		- IO 多线程，将 read/write，decode，encode 这些阻塞操作放在多线程中操作
	- 将大的操作拆分成小的操作：
		- dict 扩容复制时，采用渐进式 REHASH
			- 分散在每步操作中，移动一个数据
			- 空闲时，rehash 挪动 1ms 的数据
		- 对象类型根据节点数量，动态选择不同的数据结构实现
			- 好处：在空间和时间上进行均衡
