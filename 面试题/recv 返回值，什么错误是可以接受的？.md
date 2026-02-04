
```cpp
extern ssize_t recv (int __fd, void *__buf, size_t __n, int __flags);
```

- fd 文件描述符
- buf 用户态缓冲区指针
- n 预期读取最大字节数，一般为缓冲区的大小
- flags 一般为 0
- 返回值
	- 成功：>0, 实际读取的字节数
	- 对端关闭连接：0
	- 失败：-1，errno
		- EAGAIN：非阻塞模式下，表示当前接收缓冲区内无数据可读
		- EINTR：系统调用被信号打断

系统调用会将用户态切换到内核态

具体逻辑

- recv 先根据 socketFd 找到内核中这个 socket 对应的内核缓冲区
- 然后将内核缓冲区接收到的数据拷贝到 buf 中

特殊情况

- 当关闭放主动断开连接后，主动关闭方发送 fin 包
- 然后会在接收缓冲区后写入文件结束符 EOF
- read 读取到 EOF 会返回 0

socket 具有阻塞和非阻塞模式

通过 fcntl 可以设置 socket 为非阻塞

```cpp
fcntl(sockfd, F_SETFL, O_NONBLOCK);
```

阻塞和非阻塞的 socket 的差异

- 阻塞模式：如果接收缓冲区为空，recv 会阻塞
- 非阻塞模式：如果接收缓冲区为空，recv 直接返回 errno = EAGAIN 或 EWOULDBLOCK

可以通过多路复用技术，来提前检测接收缓冲区是否为空

- 注册读事件
- 如果读事件就绪，说明接收缓冲区内非空
- 接着可以对对应 fd 进行 recv 来接收数据

可接受的错误

- EAGAIN、EWOULDBLOCK
	- 含义：非阻塞模式下，表示当前接收缓冲区内无数据可读
	- 处理：等待下次可读事件，无需被关闭连接
- EINTR
	- 含义：系统调用被信号（如 SIGINT）中断
	- 处理：重新调用 recv
