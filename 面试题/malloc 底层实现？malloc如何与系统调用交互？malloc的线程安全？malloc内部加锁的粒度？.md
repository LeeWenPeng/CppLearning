## 1  malloc 的底层实现

### 1.1  大内存分配

- 对于大内存（>128KB），采用 `mmap` 分配，采用 `munmap` 释放
- 对于小内存 (<=128KB)，按照 `1.3  分配和释放` 的过程进行分配

### 1.2  核心数据结构

- arena 线程级内存管理容器，维护管理多个 chunk 和 bins
- chunk 是内存分配的基本单位，包含已分配块和空闲块
	- 已分配块：用于存储用户数据
	- 空闲块：按大小存入 bins 中
	- top chunk：arena 顶部未分配的备用块，bins 无合适内存时使用
- bins 用于复用空闲块，通过链表快速复用空闲块，减少系统调用
	- fast_bin：16-80 字节的小内存，单链表，LIFO 策略，不合并相邻块
	- unsorted_bin：刚释放的 small 或 large 块，临时缓存，下次分配优先
	- small_bin：小于 512 字节的块，双向链表，FIFO，合并相邻块
	- large_bin：大于等于 512 字节的块，合并相邻块，按大小递减排序，支持快速查找

### 1.3  分配和释放

#### 1.3.1  分配

1. 优先查找当前线程的 arena
2. 接着依次检查
	1. fast_bin
	2. unsorted_bin
	3. small_bin
	4. large_bin
3. 如果找到合适的 chunk，则直接分配
4. 否则，切割 top chunk
5. 如果 top chunk 不足
	1. 如果是主线程，则通过 brk 扩展堆段（对堆段的扩充）
	2. 如果是子线程，则通过 mmap 分配新的 heap（从文件映射区直接重新分配新的内存）

#### 1.3.2  释放

1. 小内存（<=80KB），直接加入到 fast_bin
2. 其他内存（>80KB）
	1. 先检查相邻块是否空闲，并进行内存的合并
	2. 然后，先加入到 unsorted_bin
	3. 在分配内存的过程中，检索 small_bin 和 large_bin 失败后，会将 unsorted 中的内存块转移到 small_bin 和 large_bin 中

## 2  malloc 线程安全

- arena 的数量限制
	- 32 bits 系统： 2 \* 核心数
	- 64 bits 系统： 8 \* 核心数
- 线程数量未超过 arena 数量限制时，每个线程和一个 arena 对应，每一个 arena 一个互斥锁（细粒度锁），避免全局竞争
- 线程数量超过 arena 数量限制时，线程需要共享 arena，从而导致锁竞争
- 分配大内存 (>128KB) 时，或者子线程创建 arena 时，通过 mmap 分配，无需经过 arena 锁

## 3  malloc 如何与系统调用动态交互

- 大内存分配时，通过 mmap，大内存释放通过 munmap
- 小内存
	- 主线程通过 brk 分配
	- 子线程则通过 mmap 分配内存空间
- 内存拓展
	- 内存不足时，主线程通过 brk 对堆段空间进行扩展，也通过 brk 释放内存空间
	- 子进程则通过 mmap 分配新的 heap
