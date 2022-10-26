[TOC]

### Redis

- [ ] 基于内存的Key-Value数据存储系统
- [ ] 支持持久化、过期
- [ ] 发布订阅
- [ ] 管道
- [ ] 支持Lua脚本
- [ ] 事务
- [ ] 分布式锁
- [ ] 哨兵、主从部署
- [ ] 集群

#### How install

》Linux ARM64

~~~bash
$ yum install redis
$ systemctl enable redis --now
~~~

配置文件地址conf

~~~bash
/etc/redis.conf
~~~

#### Redis-cli

~~~bash
$ redis-cli -h 127.0.0.1 -p 6379
~~~

|    command    |        description        |
| :-----------: | :-----------------------: |
|   select 0    | 选择数据库, 0-15个数据库  |
|   flushdb 0   | 清空某个数据库 [危险操作] |
|    keys *     |        获取所有key        |
|   del xkey    |        删除某个key        |
|  exists xkey  |         是否存在          |
|   type xkey   |        value的类型        |
|   ttl xkey    |       剩余存活时间        |
| expire xkey 5 |       设置过期时间        |