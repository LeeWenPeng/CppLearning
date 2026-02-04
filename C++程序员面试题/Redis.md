## Redis

### 1 听说过 Redis 吗？它是什么？

Redis 是**数据库**，是存储在**内存**中的数据库，所以**读写速度非常快**，因此 Redis 被广泛应用于 **缓存**方向

除此之外，Redis 还经常被用来做分布式锁，Redis 提供了多种数据类型来支持不同的业务场景，Redis 还支持事务持久化、LUA 脚本、LRU 驱动事件、多种集群方案。

### 2 Redis 的五种数据结构整理

#### 2.1 String 类型

简单动态字符串 (Simple Dynamic String, SDS)

Redis 的默认字符串类型为 SDS，SDS 等同于 C 语言种的 `char*`, 可以任意存储二进制
