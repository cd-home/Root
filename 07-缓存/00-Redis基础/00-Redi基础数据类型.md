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

> 列表是Linked List 实现，也就是链表, 头尾插入删除非常快O(1)

当我们向一个聚合数据类型中添加元素时，如果目标键不存在，就在添加元素前创建空的聚合数据类型。

当我们从聚合数据类型中移除元素时，如果值仍然是空的，键自动被销毁。

常见命令

队列

```mysql
 rpush k v1 v2 v3	 # 可以设置单个、多个 
 llen k
 lpop k				 # pop出来的数据被自动删除, 内存被回收
```

栈

```mysql
 rpush k v1 v2 v3
 rpop k
```

操作

```mysql
 rpush k v1 v2 v3
 lindex k 1   		# O(n) 获取索引位置的value
 lindex k -1  		# 倒数第一个
 ltrim k i j  		# 保留区间, 其他删除
 lrange k i j 		# O(n)获取区间value i 从0开始，j可以为负数，最后一个从-1开始（分页案例）
 lset k i v
 lrem k i v
```

生产消费模式

> 如果使用lpush 和 pop 做这种模式，列表为空，那么rpop就会空轮训, brpop设置超时时间

~~~mysql
brpop list 2
~~~

#### Hash

> 无序字典, 字典的key只能是字符串, 值是一个键值对的对象

常用命令

```mysql
hset uid:100 name liyao age 25
hget uid:100 name
hget uid:100 age
 
# 单个设置
hset K Field1 Value1
# 多个设置
hset K Field1 Value1 Field2 Value2
hmset K Field1 Value1 Field2 Value2

# 获取
hkeys K 		# 获取K的所有Field
hvals K			# 获取K的所有Field的Vlaue
hget K Field1
hgetall K
hmget K Field1 Field2
hlen K

# 删除
hdel K k1

# 存在
hexists K Field1

# 存在即不设置
hsetnx K Field1 Value2
```

同字符串对象一样, hash 结构中的单个子 key 也可以进行计数, 它对应的指令是 `hincrby`

```
hincrby K Field2 1
hincrbyfloat K Field1 1.2
```

#### Set

内部的键值对是无序的唯一的

常用命令

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

