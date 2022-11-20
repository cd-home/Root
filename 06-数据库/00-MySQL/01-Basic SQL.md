[TOC]

### Tutorial

#### 术语

|      术语      |              说明              |
| :------------: | :----------------------------: |
| 数据库管理系统 | SQL database management system |
|     数据库     |            database            |
|     数据表     |             table              |
|  数据行(记录)  |          rows, record          |
|  数据列(字段)  |        columns,  field         |
|    存储引擎    |         storage engine         |

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

#### 数据(列类型)

##### 数值类型

unsigned、signed类型修饰;  注意选择合适的类型使用;

|       类型       | 大小( byte) |    (有符号)    |  (无符号)  |       用途       |
| :--------------: | :---------: | :------------: | :--------: | :--------------: |
|   **TINYINT**    |      1      |   (-128,127)   |  (0,255)   |      小整数      |
|     SMALLINT     |      2      | (-32768,32767) | (0,65 535) |      大整数      |
|    MEDIUMINT     |      3      | -800万 ~ 800万 |   1600万   |      大整数      |
| **INT或INTEGER** |      4      |   -21亿~21亿   |    42亿    |      大整数      |
|    **BIGINT**    |      8      |                |            |       极大       |
|      FLOAT       |      4      |   FLOAT(5,2)   |            | 单精度(四舍五入) |
|      DOUBLE      |      8      |                |            |      双精度      |
|     DECIMAL      |             |                |            |   精确值(金额)   |

DECIMAL(M,D) M表示总位数、D表示小数

##### 字符串

通常情况下使用CHAR、VARCHAR、TEXT 较多

|     类型     |   大小(bytes)   |              用途               | 例子     |
| :----------: | :-------------: | :-----------------------------: | -------- |
|   **CHAR**   |      0-255      |           定长字符串            | CHAR(32) |
| **VARCHAR**  |     0-65535     |           变长字符串            |          |
|   TINYBLOB   |      0-255      | 不超过 255 个字符的二进制字符串 |          |
|   TINYTEXT   |      0-255      |          短文本字符串           |          |
|     BLOB     |    0-65 535     |     二进制形式的长文本数据      |          |
|   **TEXT**   |    0-65 535     |           长文本数据            |          |
|  MEDIUMBLOB  |  0-16 777 215   |  二进制形式的中等长度文本数据   |          |
|  MEDIUMTEXT  |  0-16 777 215   |        中等长度文本数据         |          |
|   LONGBLOB   | 0-4 294 967 295 |    二进制形式的极大文本数据     |          |
| **LONGTEXT** | 0-4 294 967 295 |          极大文本数据           |          |

##### 日期

通常情况下使用 DATETIME 类型 或者 DATE 类型

|     类型     | 大小 |                  范围                   |        格式         |         用途         |
| :----------: | :--: | :-------------------------------------: | :-----------------: | :------------------: |
|   **DATE**   |  3   |         1000-01-01 ~ 9999-12-31         |     YYYY-MM-DD      |        日期值        |
|     TIME     |  3   |        '-838:59:59'/'838:59:59'         |      HH:MM:SS       |   时间值或持续时间   |
|     YEAR     |  1   |                1901/2155                |        YYYY         |        年份值        |
| **DATETIME** |  8   | 1000-01-01 00:00:00/9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS |     混合日期时间     |
|  TIMESTAMP   |  4   |          19700101000001 ~ 2038          |   YYYYMMDDHHMMSS    | 混合日期时间值时间戳 |

##### 枚举

~~~mysql
enum('0', '1')  -- 比如性别或者其他的基本固定的选择
~~~

##### NULL

NULL;  NOT NULL

#### 约束

数据完整性：存储在数据库中的所有数据值均正确的状态

如果数据库中存储有不正确的数据值, 则该数据库称为已丧失数据完整性. 数据完整性包括: 

1.  域完整性
2.  实体完整性
3.  参考完整性（外键）

数据库通过约束来保证数据完整性. 它通过对表的行或列的数据做出限制, 来确保表的数据的

**准确性, 完整性、唯一性, 可靠性、联动性.** 

|  约束  |                   意义                   |
| :----: | :--------------------------------------: |
|  主键  | PRIMARY KEY 唯一, 自增长, 只能有一个主键 |
|  唯一  |              UNIQUE 唯一性               |
|  非空  |              非空(NOT NULL)              |
| 默认值 |              默认值 default              |
|  外键  |                 引用关系                 |
