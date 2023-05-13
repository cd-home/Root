[TOC]

### Cluster

#### Master-Slave

》主从模式, 复制数据到从节点, 读写分离, 降低Master压力; 缺点是可用性不强, 主机宕机的话, 造成写不可用.

![master-slave](./images/master-slave.svg)

从节点启动后, 发送SYNC命令启动同步. 主节点执行BGSAVE命令保存快照(RDB), 传输到从节点. 在传输过程, 以及从节点加载快照过程, 主节点的写命令会存储到缓冲区,  后续发送缓冲区命令. 此后的写命令都会直接同步到从节点

#### Sentinel

》哨兵模式, 健康监控, 故障转移; 选举新的Master

![sentienl](./images/sentienl.svg)

- [x] 监控主节点、从节点是否正常运行
- [ ] 通过API提醒Redis服务器故障
- [ ] 自动故障转移[Automatic Failover]

Ping 超时 主观下线-> 询问其他哨兵, 客观下线->Leader 选举(Raft) 健康、备份完整度、稳定性(启动周期、心跳检测) ->设置为新的主节点 Slave No one ->其他从节点设置为新节点的从节点 Slavaof

#### Cluster

三主三从

采用systemctl管理

~~~bash
$ redis-cli --cluster create 10.211.55.61:6379 10.211.55.62:6379 10.211.55.63:6379 10.211.55.61:6380 10.211.55.62:6380 10.211.55.63:6380 --cluster-replicas 1
~~~

主要的配置修改, 三台机器

master

~~~
dir /var/lib/redis/master
port 6379
bind * -::*
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
~~~

slave

~~~
dir /var/lib/redis/slave
port 6380
bind * -::*
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
~~~

