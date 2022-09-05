[TOC]

### Tutorial

Connecting And Disconnecting

~~~bash
$ mysql -h host -u user -p

> QUIT;
~~~

Entering Queries

~~~bash
$ SELECT VERSION(), CURRENT_DATE;
~~~

mysql determines where your statement ends by looking for the **terminating semicolON**, not by looking for the end of the input line. 

PS: [分号作为完整SQL的结束]

PS: [大小写不敏感，推荐关键词大些，其他(字段名、表名)小写]

#### 术语

|      术语      |              说明              |
| :------------: | :----------------------------: |
| 数据库管理系统 | SQL database management system |
|     数据库     |            database            |
|     数据表     |             table              |
|  数据行(记录)  |          rows, record          |
|  数据列(字段)  |        columns,  field         |
|    存储引擎    |         storage engine         |

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

#### DDL

Data DefinitiON Language: 操作数据库、表

##### Database

~~~sql
-- 显示所有数据库
mysql> SHOW DATABASES;

-- 创建数据库
mysql> CREATE DATABASE dbname CHARSET utf8mb4;

-- 显示数据库创建信息
mysql> SHOW CREATE DATABASE \G;

-- 修改数据库的元信息
mysql> ALTER DATABASE dbname CHARSET utf8mb4;

-- 使用数据库
-- Under Unix, database names are case-sensitive (unlike SQL keywords)
-- 数据库、以及表是 区分大小写的 (SQL 关键词是不区分大小写的, 通常使用大写)
-- 推荐 非SQL关键词小写
mysql> USE dbname;
mysql> SHOW STATUS;
mysql> SHOW GRANTS;

-- 删除数据库【危险操作, 所有的删除库、表操作都应该授权】
mysql> DROP DATABASE dbname;
~~~

##### Table

~~~sql
-- 创建表：列名 类型 约束1 约束2 约束3【列之间逗号隔开】
CREATE TABLE t_class(
	 id int(11) PRIMARY KEY AUTO_INCREMENT,
	 name VARCHAR(10) NOT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8mb4;

CREATE TABLE t_name(
    id int(11) PRIMARY KEY AUTO_INCREMENT comment "id", 	-- 自增主键
    name VARCHAR(10) NOT NULL DEFAULT 'Mike',				-- 默认值
    card VARCHAR(18) UNIQUE,								-- 唯一约束
    phONe CHAR(11) NOT NULL,
    class_id int NOT NULL,
    INDEX idx_name(name),									-- 创建普通索引			
    UNIQUE INDEX idx_phONe(phONe),							-- 创建唯一索引
    														-- 外键约束
	CONSTRAINT fk_name FOREIGN KEY(class_id) REFERENCES t_class(id) 
)ENGINE=INNODB DEFAULT CHARSET=utf8mb4;						-- 引擎与编码
	
-- 查看表信息、表结构	
mysql> SHOW tables; 			     						-- 查看所有数据表
mysql> SHOW CREATE TABLE t_name;							-- 显示创建表的信息
mysql> SHOW CREATE TABLE t_name \G; 						-- 显示创建表的信息
mysql> DESC t_name; 				 						-- 查看字段信息/表结构

-- 修改表结构
mysql> ALTER TABLE t_name ADD field_name type;	    		-- 增加字段
mysql> ALTER TABLE t_name ADD primary key(id);				-- 增加属性

mysql> ALTER TABLE t_name ADD UNIQUE(`field_name`);			-- 增加属性； 唯一约束
mysql> ALTER TABLE t_name ADD UNIQUE (`f1`, `f2`);			-- 增加属性； 联合唯一

mysql> ALTER TABLE t_name ADD UNIQUE INDEX(`f1`);			-- 增加属性； 唯一索引
mysql> ALTER TABLE t_name ADD UNIQUE INDEX(`f1`, `f2`);		-- 增加属性； 联合唯一索引

mysql> ALTER TABLE t_name DROP field_name;					-- 删除字段
mysql> ALTER TABLE t_name MODIFY field_name type;			-- 修改字段类型
mysql> ALTER TABLE t_name CHANGE old_field_name new_field_name type; -- 修改字段列名

-- 普通索引
mysql> CREATE INDEX idx_name_card ON t_name (name, card);	-- 创建索引
mysql> DROP INDEX idx_name_card ON t_name;					-- 删除索引
mysql> ALTER TABLE t_name ADD INDEX idx_name(name);			-- 添加索引

-- 唯一索引
mysql> CREATE UNIQUE INDEX idx_card ON t_name (card);		-- 创建唯一索引
mysql> SHOW INDEX from t_name \G;							-- 查看索引信息
~~~

#### DML

Data ManipulatiON Language：操作数据行

~~~sql
SELECT
INSERT
UPDATE
DELETE
EXPLAIN # SQL执行计划
LOCK
~~~

##### SELECT

~~~sql
SELECT 
	col1,
	col2
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

特殊的, * 可以查询所有的字段数据， 通常线上不要这样使用

~~~sql
SELECT * FROM 
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

#### DCL

Data CONtrol Language

~~~mysql
BEGIN
START transactiON
COMMIT
SAVEPOINT
ROLLBACK
SET TRANSACTION
~~~

Examples: 事务

~~~mysql
mysql> BEGIN;
mysql> ...
mysql> COMMIT;
~~~