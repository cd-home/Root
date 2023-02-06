[TOC]

### Redis

基于内存的高性能kv数据存储系统, 支持多种数据结构.

- [ ] 支持持久化、过期策略
- [ ] 发布订阅模式
- [ ] 管道、事务
- [ ] 支持Lua脚本
- [ ] 分布式锁
- [ ] 哨兵、主从部署
- [ ] 集群分片

#### How install

》Linux ARM64 单机版安装

~~~bash
$ yum install redis
$ systemctl enable redis --now
~~~

配置文件地址conf

~~~bash
/etc/redis.conf
~~~

或者可通过启动服务

~~~bash
$ redis-server /etc/redis.conf
~~~

#### Redis-cli

客户端连接

~~~bash
$ redis-cli -h 127.0.0.1 -p 6379
~~~
