[TOC]

### DataTypes

Redis keys 是二进制安全, 任何的二进制序列都可作为key(通常情况下的key是字符串). 以下两点注意

- [ ] key最好是业务相关的, 不建议太长、或者太短
- [ ] key最大允许512MB

value可以是string 、list 、hash (哈希) 、set和 zset (有序集合)等类型. 

不针对特别数据类型的公用命令

|    command     |       description        |
| :------------: | :----------------------: |
|    select 0    | 选择数据库, 0-15个数据库 |
|   flushdb 0    |      清空某个数据库      |
|     keys *     |       获取所有key        |
|    del xkey    |       删除某个key        |
|  exists xkey   |         是否存在         |
|   type xkey    |       value的类型        |
| expire xkey 5  |     设置过期时间(秒)     |
| pexpire xkey 5 |    设置过期时间(毫秒)    |
|    ttl xkey    |     剩余存活时间(秒)     |
|   pttl xkey    |    剩余存活时间(毫秒)    |

#### String

字符串类型是最简单的数据类型, 包括二进制数据, 但是大小不能超过512MB.

简单的设置与获取, 不存在的键即创建, 存在的键设置直接覆盖更新. 

~~~bash
set k v
get k
~~~

同时设置过期时间, ex 秒, px 毫秒; 除了在设置值的时候指定过期时间, 还可以通过上述公共命令. 

~~~bash
set k v ex 100 px 1000

# 简单命令
setex k 5 v	
~~~

nx: 键存在即设置失败, xx: 键存在才能设置, 否则失败

~~~bash
set k v nx 
# 简单命令
setnx k v

set k v xx
~~~

同时设置、获取多组; 可以减少延迟;

~~~bash
mset k1 v1 k2 v2
mget k1 k2  	  
~~~

获取同时设置, 获取旧值, 同时设置新值

~~~bash
getset k v
~~~

原子增、减; incr指令会解析"字符串"为数字

```sql
set visitor 0	      
incr visitor       	  # 将字符串转为整型, 默认加1
incrby visitor 2   	  # 加2
incrby visitor -3  	  # 减3
```

其他操作

~~~bash
strlen k		  # 获取长度
getrange k 0 2    # 切片
setrange k 1 v    # 设置
append k .cpp     # 拼接
~~~

#### List

Redis List 采用的是链表 Linked Lists(QuickList), 添加删除效率是极高的. 

添加

```mysql
rpush k v1 v2 v3	 # 可以设置单个、多个 
rpush k v1 v2 v3
llen k				 # 获取长度
```

删除

~~~bash
rpop k    			 # pop出数据在列表中被删除
lpop k 2			 # pop个数
~~~

区间获取

~~~bash
lrange k i j 		 # O(n)获取区间value i 从0开始，j可以为负数，最后一个从-1开始
~~~

lpush数据, 通过lrange k 0 10 这样的模式可以获取最新的数据

裁剪, 范围之外的会被删除

~~~bash
ltrim k i j  		 # 保留区间, 其他删除
~~~

lpush + ltrim 可以保留指定条数的数据. 

index获取值与设置

~~~bash
lindex k 0   		 # O(n)获取索引位置的值, 从0开始
lindex k -1  		 # 倒数第一个
lset k 0 v			 # 设置index位置的值
~~~

删除值

~~~bash
lrem k count v		 # 删除指定个数的值
~~~

阻塞操作

如果使用lpush 和 rpop 做这种模式做生产者消费者模型, 如果列表为空, 消费端rpop空轮训, 一直调用rpo p, 做很多无用的操作, 可采用brpop阻塞式读取并且可以设置超时时间

~~~mysql
blpop k
brpop k 2 				# 亦可设置获取个数
~~~

移动元素, 同样支持阻塞移动; left、right 表示获取元素的方向

~~~bash
lmove k1 k2 left|right left|right
blmove k1 k2 left|right left|right timeout
~~~

说明

当向聚合数据类型中添加元素时, 如果键不存在, 就先创建空的聚合类型然后添加. 当我们从聚合数据类型中移除元素时, 如果值是空的, 键自动被销毁. [List、Set、Hash、Zset、Stream]

#### Hash

无序字典, 字典的key只能是字符串, 值是一个键值对的对象; Redis 7.0 废弃压缩列表采用Listpack实现

```mysql
hset uid:100 name yao age 28

hget uid:100 name
hget uid:100 age
 
# 单个设置
hset K Field1 Value1
# 多个设置
hset K Field1 Value1 Field2 Value2
hmset K Field1 Value1 Field2 Value2

# 获取
hkeys K 				# 获取K的所有Field
hvals K					# 获取K的所有Field的Value
hget K Field1   		# 获取Field1的Value1
hgetall K       		# 获取所有
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

无序的字符串集合; 整数列表(item < 512)或者哈希表实现

```mysql
# 添加
sadd k v1
sadd k v2 v3

smembers k     # 获取所有
sismember k v1 # 查询是否有
scard k        # 获取长度（成员个数）
# 移动集合元素
smove oneset otherset v

# 随机的获取n个
spop k n      
```

集合操作, 注意数据量较大的话集合操作比较耗时造成实例阻塞

~~~mysql
# 交集
sinter k1 k2 k3 
# 结果存入新集合
sinterstore destkey k1 k2 k3
# 并集
sunion k1 k2 k3 
sunionstore destkey k1 k2 k3
# 差集
sdiff k1 k2
sdiffstore destkey k1 k2
~~~

#### Zset

一方面它是一个 set, 保证了内部 value 的唯一性, 另一方面它可以给每个 value 赋予一个 score, 代表这个 value 的排序权重

O(log(N)), 添加删除更新

压缩列表(item<128)或者跳跃表; Redis 7.0 废弃压缩列表采用Listpack实现;

```mysql
zadd key score member [[score member]...]   
zadd k 10 v1
zadd k 20 v2
zadd k 30 v3
zrange k 0 -1 withscores # 区间输出
zrevrange k 0 -1 		 # 逆序输出
zcard k          		 # 长度
zscore k v1      		 # 获取分数
zrank k v2       		 # 排名, 从0开始
zrevrank k v2    		 # 反向排名
zrangebyscore k 10 29    # 权重区间遍历
zrem k v3        		 # 删除
zremrangebyscore		 # 通过分数删除
zincrby k xscore v		 # 增加分数
```

#### Bitmap

位图, 二进制位字符串, 位置从0开始; 可认为是个bit数组

~~~sql
# 设置
setbit key offset value
setbit k 2 1
# 获取
getbit key offset
getbit k 2

# 范围计数
bitcount key start end
bitcount k
~~~

位运算

~~~sql
bitop [operations] [result] [key1] [keyn…] # op = or | and | not 
setbit b1 0 1
setbit b1 1 0
bitop or b3 b1 b2	
~~~

#### HyperLogLog

是一种用于统计基数的数据集合类型, 基数统计就是指统计一个集合中不重复的元素个数; 注意不是精确统计的;

~~~bash
pfadd k v1 v2 v3 v4
pfcount k
~~~

#### Geo

存储地理位置信息, 并对存储的信息进行操作
