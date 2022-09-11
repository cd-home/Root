[TOC]

## DataTypes

### Key

key通常是字符串(二进制安全)),是业务相关的,并且不建议使用太长、或者太短的Key;

当我们向一个聚合数据类型中添加元素时, 如果目标键不存在, 就在添加元素前创建空的聚合数据类型. 当我们从聚合数据类型中移除元素时, 如果值仍然是空的, 键自动被销毁. [没有元素key就会被清除]

### Value

Value可以是string 、list 、hash (哈希) 、set和 zset (有序集合)

#### String

value可以是任何类型的字符串, 但是长度不超过512MB

```bash
# 单k-v
set k v
set k 100 ex 10   # 创建时设置过期时间
# 不存在就创建
set lock_key unique_value nx px 10000

# 获取
get k

# 设置
setnx k v 		  # 不存在就创建否则不创建

# 多key
mset k1 v1 k2 v2
mget k1 k2  	  # 返回列表

setex k 5 v		  # SET 同时设置过期时间
```

计数器

```sql
set visitor 0	      
# 原子操作
incr visitor       	  #  将字符串转为整型, 默认加1
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

字符串列表, Linked Lists(QuickList)

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

其他操作

```mysql
rpush k v1 v2 v3
lindex k 1   		# O(n) 获取索引位置的value
lindex k -1  		# 倒数第一个
ltrim k i j  		# 保留区间, 其他删除
lrange k i j 		# O(n)获取区间value i 从0开始，j可以为负数，最后一个从-1开始
lset k i v
lrem k i v
```

生产消费模式

如果使用lpush 和 pop 做这种模式, 如果列表为空, rpop就会空轮训, brpop阻塞式读取并且可以设置超时时间

~~~mysql
brpop k 2
brpoppush k otherk
~~~

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

#### zset

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
