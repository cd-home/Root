[TOC]

#### DDL

Data Definition Language: 操作数据库、表

##### CREATE DATABAE

通常需要指定字符集与排序规则. 

-- 显示所有数据库

~~~mysql
mysql> SHOW DATABASES;
~~~

-- 创建数据库, 数据库设置会影响表的设置(创建表时也可指定, 覆盖数据库的设置, 列也可以指定, 优先级依次增高)

~~~mysql
mysql> CREATE DATABASE [IF NOT EXISTS] dbname CHARACTER SET = utf8mb4 COLLATE = xxx;
mysql> CREATE DATABASE [IF NOT EXISTS] dbname CHARSET = utf8mb4 COLLATE = xxx;
-- 排序规则注意一般情况下需要区分大小写 _cs
~~~

-- 显示数据库创建信息

~~~mysql
mysql> SHOW CREATE DATABASE dbname \G;
~~~

-- 修改数据库的元信息

~~~mysql
mysql> ALTER DATABASE dbname CHARSET utf8mb4;
~~~

-- 使用数据库

~~~mysql
-- Under Unix, database names are case-sensitive (unlike SQL keywords)
-- 数据库、以及表是 区分大小写的 (SQL 关键词是不区分大小写的, 通常使用大写)
-- 推荐 非SQL关键词小写
mysql> USE dbname;
mysql> SHOW STATUS;
mysql> SHOW GRANTS;
~~~

-- 删除数据库【危险操作, 所有的删除库、表操作都应该授权】

~~~sql
mysql> DROP DATABASE dbname;
~~~

##### CREATE TABLE 

~~~mysql
mysql> CREATE TABLE t_name IF NOT EXISTS (
    
	col_name column_definition
    
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4  PARTITION BY KEY(col_3) PARTITIONS 4;

-- 具体看分区表模式
-- PARTITION BY HASH(col_1);
-- PARTITION BY RANGE(year_col) (
     PARTITION p0 VALUES LESS THAN (1991),
     PARTITION p1 VALUES LESS THAN (1995),
     PARTITION p2 VALUES LESS THAN (1999),
     PARTITION p3 VALUES LESS THAN (2002),
     PARTITION p4 VALUES LESS THAN (2006),
     PARTITION p5 VALUES LESS THAN MAXVALUE
);

column_definition: {
    data_type [NOT NULL | NULL] [DEFAULT default_value]
      [AUTO_INCREMENT] [UNIQUE [KEY]] [[PRIMARY] KEY]
      [COMMENT 'string']
      [COLLATE collation_name]
      [reference_definition]
}
~~~

每个 table 只能有一个`AUTO_INCREMENT`列, 必须对其进行索引, 并且不能具有`DEFAULT`值

唯一索引, 其中所有键列必须定义为`NOT NULL`。如果未将它们显式声明为`NOT NULL`, 则 MySQL 会如此隐式(无声地)声明它们。一个 table 只能有一个`PRIMARY KEY`。 `PRIMARY KEY`的名称始终为`PRIMARY`, 因此不能用作任何其他种类的索引的名称

如果您没有`PRIMARY KEY`并且应用程序在 table 中要求`PRIMARY KEY`, 则 MySQL 返回的第一个`UNIQUE`索引将没有`NULL`列作为`PRIMARY KEY`; `PRIMARY KEY`可以是多列索引

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
~~~

###### FOREIGN KEY

Belong to

~~~mysql
CREATE TABLE user (
    id INT NOT NULL,
    company_id INT,
    index c_ind (company_id),
    -- 指定外键的名字, 不指定默认创建
    CONSTRAINT `fk_xxname` FOREIGN KEY (company_id) REFERENCES company(id) ON DELETE SET NULL
) ENGINE=INNODB;

CREATE TABLE company (
    id INT,
    PRIMARY KEY (id)
) ENGINE=INNODB;
~~~

Has One

~~~mysql
CREATE TABLE user (
    id INT NOT NULL,
    PRIMARY KEY (id)
) ENGINE=INNODB;

CREATE TABLE id_card (
    id INT,
    user_id INT,
    UNIQUE u_ind (user_id),
    FOREIGN KEY (user_id) REFERENCES user(id) ON DELETE CASCADE
) ENGINE=INNODB;
~~~

Has Many

~~~sql
CREATE TABLE parent (
    id INT NOT NULL,
    PRIMARY KEY (id)
) ENGINE=INNODB;

CREATE TABLE child (
    id INT,
    parent_id INT,
    INDEX par_ind (parent_id),
    FOREIGN KEY (parent_id) REFERENCES parent(id) ON DELETE CASCADE
) ENGINE=INNODB;
-- 1:1 设置外键字段为唯一索引即可
~~~

Many To Many

```sql
CREATE TABLE product (
    id INT NOT NULL,
    category INT NOT NULL, 
    price DECIMAL,
    PRIMARY KEY(category, id)
)   ENGINE=INNODB;

CREATE TABLE customer (
    id INT NOT NULL,
    PRIMARY KEY (id)
)   ENGINE=INNODB;

-- 第三张表表示关系
CREATE TABLE product_order (
    no INT NOT NULL AUTO_INCREMENT,
    product_category INT NOT NULL,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,

    PRIMARY KEY(no),
    INDEX (product_category, product_id),
    INDEX (customer_id),

    FOREIGN KEY (product_category, product_id) 
    REFERENCES product(category, id)
    ON UPDATE CASCADE ON DELETE RESTRICT,

    FOREIGN KEY (customer_id)
      REFERENCES customer(id)
    
)   ENGINE=INNODB;
```

删除被参考表(主表)的记录, 参考表的记录的动作时如何的

on update [no action |set null |  cascade | restrict]

on delete [no action | set null | cascade |  restrict ]

cascade表示及联更新或者删除

[主键表中被参考字段更新, 外键表中对应行相应更新；如果为on delete cascade, 主键表中的记录被删除, 外键表中对应行相应删除]

restrict 不允许删除[更新] 当在父表(即外键的来源表)中删除[更新]对应记录时, 首先检查该记录是否有对应外键, 如果有则不允许删除[更新]

但InnoDB拒绝包含ON DELETE SET DEFAULT或ON UPDATE SET DEFAULT子句的表定义.

###### ALTER TABLE

~~~mysql
-- 查看表信息、表结构	
mysql> SHOW TABLES; 			     						-- 查看所有数据表
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
mysql> ALTER TABLE t_name ADD INDEX idx_name(name);			-- 添加索引
~~~

###### CREATE INDEX

~~~mysql
-- 普通索引
mysql> CREATE INDEX idx_name_card ON t_name (name, card);	-- 创建索引
mysql> DROP INDEX idx_name_card ON t_name;					-- 删除索

-- 唯一索引
mysql> CREATE UNIQUE INDEX idx_card ON t_name (card);		-- 创建唯一索引
mysql> SHOW INDEX from t_name \G;							-- 查看索引信息
~~~
