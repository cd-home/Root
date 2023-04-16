[TOC]

### DML

#### DELETE 

~~~mysql
>mysql DELETE FROM t_name WHERE [conditions]

-- 多表连接删除 t1 t2
>mysql DELETE FROM t1, t2 USING t1 INNER JOIN t2 INNER JOIN t3
WHERE t1.id=t2.id AND t2.id=t3.id;
~~~

