### Tutorial

Install And Config

~~~bash
/etc/my.cnf
/etc/mysql/my.cnf
~~~

Connecting And Disconnecting

~~~bash
$ mysql -h host -u user -p

> QUIT;
~~~

Entering Queries

~~~bash
$ SELECT VERSION(), CURRENT_DATE;
~~~

mysql determines where your statement ends by looking for the **terminating semicolon**, not by looking for the end of the input line. 

PS: [分号作为完整SQL的结束]