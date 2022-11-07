[TOC]

### Redis

什么场景下使用Redis(缓存)?

Redis的网络模型(单线程、Epoll), 为什么那么快?

Redis基本的数据类型, 你常用的是什么, 在什么场景下使用?

Redis其他的类型有了解么?

Redis持久化机制?

Redis内存满, 数据的淘汰策略有哪些?

Redis分布式锁有了解么, 用在什么场景?

Redis执行Lua脚本有接触么, 用在什么场景?

Redis管道是什么, 有什么作用?

Redis能做延迟队列吗, 如何实现?

Redis发布订阅模式有了解吗?

Redis的事务了解么?

Redis如何实现CAS? 事务、Lua脚本

CAS用来解决高并发情况下, 丢失更新的问题, 也就是写偏序. [GET, IF, SET] 等多个过程. [关系型数据库乐观模式也可以使用CAS]

1. watch key, 一旦key被修改则事务回滚. 事务中的命令会进入队列, 最后靠exec执行.
2. Lua脚本亦可进行compare and set的操作

Redis缓存雪崩、穿透、击穿是什么意思, 如何解决?

Redis与数据库如何保证一致性?

Redis部署模式, 高可用方案?

Redis集群的选举机制有了解么, Raft协议(分布式共识算法)简单描述一下?
