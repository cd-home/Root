[TOC]

### More Types

#### bitmap

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

#### Stream