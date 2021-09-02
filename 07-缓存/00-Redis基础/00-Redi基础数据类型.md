[TOC]

## Redis

### 前言

- [x] Redis是基于内存的Key-Value数据存储系统，一般用作缓存、消息中间件
- [ ] 支持Lua脚本、事务
- [x] 支持持久化
- [x] 哨兵、主从部署

### Key

> key通常是字符串，是业务相关的，并且不建议使用太长、或者太短的Key

#### Value

> Value可以是string (字符串)、list (列表)、set (集合)、hash (哈希) 和 zset (有序集合)
>
> bitmap(位图)、hyperloglogs、geospatial(地理空间)

### 数据类型

> 数据类型指的是Value的类型

#### String

> 所有的数据类型的键都是以唯一字符串作为key, 通过key获取value

通用方法

~~~sql
keys *    # 线上禁用
del k	  # 删除
exists k  # 是否存在
type k    # 返回类型
ttl	k	  # 返回剩余存活时间
~~~

单k-v

```sql
set k v
set k 100 ex 10   # 创建时设置过期时间
get k
setnx k v 		  # 不存在就创建否则不创建
```

多k-v

```sql
mset k1 v1 k2 v2
mget k1 k2  	 # 返回列表
```

设置过期

```sql
expire k 5  	 # 5s
setex k 5 v		 # SET 同时设置过期时间
```

计数

```sql
 set age 25		# 注意，虽然看似SET的是数字，实际上已经被转换为了字符串
 incr age       # 默认加1(原子操作), 将字符串转为整型
 incrby age 2   # 加2
 incrby age -3  # 减3
```

其他

```
 strlen k
 getrange k 0 2
 setrange k 1 v
 append k .cpp
```

#### List

1. Redis中的列表是链表, 插入删除非常快O(1)

2. pop出来的数据被自动删除, 内存被回收

3. Redis的List通常被用做异步队列使用, 将需要延后处理的任务结构体序列化成字符串塞进Redis的列表, 然后消费

4. 常见命令

    - 队列

    ```
     rpush k v1 v2 v3
     llen k
     lpop k
    ```

    - 栈

    ```
     rpush k v1 v2 v3
     rpop k
    ```

    - 慢操作

    ```
     rpush k v1 v2 v3
     lindex k 1   # O(n) 获取索引位置的value
     lindex k -1  # 倒数第一个
     ltrim k i j  # 保留区间, 其他删除
     lrange k i j # O(n)获取区间value
     lset k i v
     lrem k i v
    ```

#### hash

1. 无序字典, 字典的key只能是字符串

2. 内部实现, 数组 + 链表二维结构. 第一维 hash 的数组位置碰撞时, 就会将碰撞的元素使用链表串接起来

    第一维是数组, 第二维是链表, hash的内容key和value存放在链表中, 数组里存放的是链表的头指针,通过key查找元素时, 先计算key的hashcode, 然后用hashcode对数组的长度进行取模定位到链表的表头, 再对链表进行遍历获取到相应的value值, 链表的作用就是用来将产生了「hash碰撞」的元素串起来

1. rehash

    为了让哈希表的负载因子维持在一个合理的范围之内,  当哈希表保存的键值对数量太多或者太少时,  程序需要对哈希表的大小进行相应的扩展或者收缩.Redis 为了高性能, 不能堵塞服务, 所以采用了渐进式 rehash 策略

2. 当hash的最后一个元素被移除, 该内存被回收

3. hash的存储空间消耗同等情况下要高于字符串

4. 常用命令

    ```
     hset K k1 v1
     hset K k2 v2
     hset K k3 v3
     hgetall K
     hlen K
     hget K k1
     hset K k2 Newv2
     hmset K k1 v1 k2 v2 k3 v3
    ```

    同字符串对象一样, hash 结构中的单个子 key 也可以进行计数, 它对应的指令是 `hincrby`

    ```
     hincrby K k1 1
    ```

#### Set

1. 内部的键值对是无序的唯一的

2. 内部实现相当于一个特殊的字典, 字典中所有的 value 都是一个值`NULL`

3. 当集合中最后一个元素移除之后, 数据结构自动删除, 内存被回收

4. 常用命令

    ```
     sadd k v1
     sadd k v2 v3
     smembers k     # 获取所有
     sismember k v1 # 查询是否有
     scard k        # 获取长度
     spop k         # 弹出一个
    ```

#### zset

1. 一方面它是一个 set, 保证了内部 value 的唯一性

2. 另一方面它可以给每个 value 赋予一个 score, 代表这个 value 的排序权重

3. zset底层实现使用了两个数据结构, 第一个是hash, 第二个是跳跃列表

    - hash的作用就是关联元素value和权重score, 保障元素value的唯一性,可以通过元素value找到相应的score值
    - 跳跃列表的目的在于给元素value排序, 根据score的范围获取元素列表

4. 最后一个 value 被移除后, 数据结构自动删除, 内存被回收

5. 常用命令

    ```
     zadd k 10 v1
     zadd k 20 v2
     zadd k 30 v3
     zrange k 0 -1    # 区间输出
     zrevrange k 0 -1 # 逆序输出
     zcard k          # 长度
     zscore k v1      # 获取权重
     zrank k v2       # 排名
     zrevrank k v2    # 反向排名
     zrangebyscore k 10 29 # 权重区间遍历
     zrem k v3        # 删除
    ```

