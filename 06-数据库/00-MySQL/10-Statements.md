#### DDL

Data DefinitiON Language: 操作数据库、表

##### Database

~~~sql
-- 显示所有数据库
mysql> SHOW DATABASES;

-- 创建数据库, 数据库是区分大小写的; 推荐小写
mysql> CREATE DATABASE dbname CHARSET utf8mb4;

-- 显示数据库创建信息
mysql> SHOW CREATE DATABASE dbname \G;

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

