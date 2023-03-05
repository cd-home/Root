[TOC]

### InnoDB

#### Engine

》默认为InnoDB

~~~mysql
mysql> show engines;
| Engine | Support | Comment | Transactions |XA | Savepoints |
| InnoDB | DEFAULT | Supports transactions, row-level locking, and foreign keys| YES | YES  | YES |
~~~

事务、回滚

~~~mysql
CREATE TABLE test(
	
)ENGINE=InnoDB;
~~~

~~~mysql
ALTER TABLE test ENGINE=InnoDB
~~~

#### DCL

Data Control Language

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
