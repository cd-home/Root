### Character Sets

#### Charset

~~~mysql
mysql> show CHARSET;
| utf8mb4  | UTF-8 Unicode | utf8mb4_0900_ai_ci | 4 |
~~~

注意排序规则: ai 不区分重音  ci 不区分大小写

~~~mysql
mysql> show variables  like 'character_set_server';
+----------------------+---------+
| Variable_name        | Value   |
+----------------------+---------+
| character_set_server | utf8mb4 |
+----------------------+---------+
1 row in set (0.00 sec)

mysql> show variables like 'collation_server';
+------------------+--------------------+
| Variable_name    | Value              |
+------------------+--------------------+
| collation_server | utf8mb4_0900_ai_ci |
+------------------+--------------------+
1 row in set (0.00 sec)

# 查看支持的排序规则
mysql> show COLLATION like 'utf8%';
~~~

注意: 可在服务器级别、数据库级别、表级别设置、列级别(字符串列)