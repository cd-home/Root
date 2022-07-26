[TOC]

### 检索数据

》SQL 必知必会

#### SELECT

1.  从哪里检索【表】
2.  检索什么【字段列】
3.  SQL语句必须是分号结尾
4.  SQL不区分大小写，可以对SQL关键字大写、表和列用小写
5.  除了正常SQL语句的空格之外，多余的空格都会被忽略

##### 单列

~~~mysql
mysql> select prod_id from products;
~~~

##### 多列

~~~mysql
mysql> select prod_id, prod_name, prod_price from products;
~~~

##### 所有

~~~mysql
mysql> select * from products;
~~~

##### 去重

~~~mysql
mysql> select distinct vend_i from products;
~~~

##### 限制数目

~~~mysql
mysql> select prod_name from products limit 5;           # 从第一行开始
mysql> select prod_name from products limit m offset n;  # m表示从m开始的n行
~~~

##### 表限定列

~~~mysql
mysql> select products.prod_name from products;				 # 限定列
mysql> select products.prod_name from crashcourse.products;  # 限定表
~~~

#### 排序

##### 单列

~~~mysql
mysql> select prod_name from products order by prod_name;
~~~

##### 多列

~~~mysql
mysql> select prod_id, prod_price, prod_name from products order by prod_price, prod_name; #多列
~~~

##### 排序方向

~~~mysql
mysql> select 
			prod_id, prod_price, prod_name 
	   from products order by prod_price desc; # 降序
	   
# DESC关键字只应用到直接位于其前面的列名
mysql> select 
			prod_id, prod_price, prod_name 
	   from products order by prod_price desc, prod_name;
	   
mysql> select prod_price from porducts order by prod_price desc limit 1;
~~~

#### 过滤

##### 比较

~~~mysql
mysql> select prod_name, prod_price from products where prod_price = 2.5;
~~~

​	支持操作符

~~~mysql
 =   <>   !=   <  >   <=   >= 
 between A and B
~~~

~~~mysql
mysql> select prod_name from products where prod_price > 10;
mysql> select prod_name from products where prod_price between 2 and 20;
~~~

##### 空值

~~~mysql
mysql> select prod_name from products where prod_price is null;
mysql> select prod_name from products where prod_price is not null;
~~~

#### 多条件组合

##### 逻辑组合

~~~mysql
mysql> select prod_name from products where vend_id = 1003 and prod_price <= 10;
mysql> select prod_name from products where vend_id = 1003 or prod_id = 1004;
mysql> select prod_name from products where vend_id = 1003 and prod_price <= 10;
~~~

​	注意 and的优先级高，在or 和 and 组合的时候可以加括号

##### in

~~~mysql
mysql> select pro_name, prod_price from products where vend_id in (1002, 1004) order by prod_id;
~~~

​	这里使用in和or的效果是一样的， in可以包含其他子句

##### not

~~~mysql
mysql> select pro_name, prod_price from products where vend_id not in (1002, 1004);
~~~

#### 通配符过滤



