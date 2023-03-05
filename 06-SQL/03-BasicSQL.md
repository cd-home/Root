[TOC]

### Tutorial

Connecting And Disconnecting

~~~bash
$ mysql -h host -u user -p

> QUIT;
~~~

Entering Queries

~~~mysql
$ SELECT USER(), VERSION(), CURRENT_DATE;
~~~

Tips: 分号作为完整SQL的结束; 大小写不敏感, 推荐关键词大些, 其他(字段名、表名)小写; \c 结尾表示不想执行当前SQL.

#### 创建选择数据库

查看Databases

~~~mysql
$ SHOW DATABASES;
~~~

创建数据库

~~~mysql
$ CREATE DATABASE test;
~~~

选择数据库

~~~mysql
$ USE test;
~~~

#### 创建数据表

查看数据表

~~~mysql
$ SHOW TABLES:
~~~

创建数据表

~~~mysql
$ CREATE TABLE pet (
	name VARCHAR(20),
	owner VARCHAR(20),
	species VARCHAR(20),
	sex CHAR(1),
	birth DATE,
	death DATE
);
~~~

VARCHAR 动态长度的字符, CHAR固定长度字符. 

查看表的列信息

~~~mysql
$ DESCRIBE pet;
~~~

#### 导入数据

VALUES 可以导入多条数据.

~~~mysql
$ INSERT INTO pet VALUES 
	('Puffball','Diane','hamster','f','1999-03-30',NULL), 
	('luffy','Harold','cat','f','1993-02-04', NULL), 
	('Bowser','Diane','dog','m','1979-08-31','1995-07-29');
~~~

#### 查询所有数据

\* 获取所有的列

~~~mysql
$ SELECT * FROM pet;
~~~

#### 获取特定行

WHERE 筛选满足条件的行.  

~~~mysql
$ SELECT * FROM pet WHERE name = 'luffy';
~~~

常见条件有: =, !=, >, <, <=, >=, in, not in, between and, like, is null, is not null; 条件连接有 and, or;

注意: NULL 值需要使用is, is not; NULL ASC排序是在前,DESC 排序在后. 

对于复杂的筛选, 可以加括号

~~~mysql
$ SELECT * FROM pet WHERE (species = 'cat' AND sex = 'm') OR (species = 'dog' AND sex = 'f');
~~~

#### 获取特定列

不采用*, 直接采用需要获取的列名即可.

~~~mysql
$ SELECT name, birth FROM pet;
$ SELECT owner FROM pet;
# 去重
$ SELECT DISTINCT owner FROM pet;
~~~

#### 排序

默认的排序是升序 ASC, DESC表示降序

~~~mysql
$ SELECT name, birth FROM pet ORDER BY birth;
$ SELECT name, birth FROM pet ORDER BY birth DESC;
# 可以排序多个列
$ SELECT name, birth FROM pet ORDER BY birth ASC, death DESC;
~~~

#### 日期计算

~~~mysql
$ SELECT name, birth, TIMESTAMPDIFF(YEAR, birth, CURDATE()) AS age FROM pet;
$ SELECT name, birth, TIMESTAMPDIFF(YEAR, birth, CURDATE()) AS age FROM pet ORDER BY age DESC;
$ SELECT name, birth, YEAR(birth), MONTH(birth), DAY(birth) FROM pet;
~~~

主要需要知道一些日期函数的使用即可.  例如还有DAYOFMONTH(), WEEK(), DATE_ADD()

时间+- 计算

~~~mysql
$ SELECT '2022-11-20' + INTERVAL 1 DAY;
~~~

可选 +-, 可选SECOND, MINUTE, HOUR, DAY, YEAR

####  模式匹配

like模糊匹配, 支持%多个字符, _ 单个字符. 

~~~mysql
SELECT * FROM pet WHERE name LIKE 'b%';
~~~

亦可以采用支持正则函数, 支持perl正则语法

~~~mysql
SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b');
~~~

#### 计算行数

统计行数, 或者统计分组后每组行数.

~~~mysql
$ SELECT COUNT(*) FROM pet;
$ SELECT owner, COUNT(*) FROM pet GROUP BY owner;
~~~

#### 分组

#### 连表

#### 总结

~~~mysql
SELECT 
	col1,
	col2 as another_name
FROM　T　　　　  

LEFT JOIN 	-- JOIN 、RIGHT JOIN

WHERE     	
			-- 条件筛选 = != > >= < <= 
			-- in ()  
			-- not in ()
			-- is null 
			-- is NOT NULL
			-- like (%  _)
			-- 条件连接 and or 
			
GROUP BY    -- 分组 (SUM、AVG等) 

HAVING　　   -- 分组后条件筛选　  

ORDER BY　　 -- 排序

LIMIT 		-- 限制条数 LIMIT offset, size | LIMIT size

-- 连接其他的SELECT 结果集
UNION 		-- 去重
UNION ALL   -- 不去重
~~~

#### DML

Data Manipulation Language：操作数据行

~~~sql
SELECT
INSERT
UPDATE
DELETE
EXPLAIN # SQL执行计划
LOCK
~~~

##### INSERT

~~~sql
INSERT INTO T(Field1, Field2, ...) VALUES(v1, v2, ....)
~~~

##### UPDATE

~~~sql
UPDATE T SET Field1=NewValue, Field2=NewValue2 WHERE [cONditiONs]
~~~

##### DELETE

~~~sql
DELETE FROM T WHERE [cONditiONs]
~~~
