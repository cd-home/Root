[TOC]

### Install MySQL On Linux

》Linux Centos ARM64

~~~bash
$ yum install -y mysql mysql-server mysql-devel

# or download rpm package
$ wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
$ yum install mysql-community-server

# or
$ sudo yum localinstall mysql80-community-release-el8-{version-number}.noarch.rpm
~~~

查看版本信息

~~~bash
[root@liyao ~]# rpm -qi mysql-server
Name        : mysql-server
Version     : 8.0.26
Release     : 1.module_el8.4.0+915+de215114
Architecture: aarch64
Install Date: 2022年08月30日 星期二 10时51分41秒
Group       : Unspecified
Size        : 166699966
License     : GPLv2 with exceptions and LGPLv2 and BSD
Signature   : RSA/SHA256, 2021年09月01日 星期三 17时13分27秒, Key ID 05b555b38483c65d
Source RPM  : mysql-8.0.26-1.module_el8.4.0+915+de215114.src.rpm
Build Date  : 2021年09月01日 星期三 16时25分03秒
Build Host  : aarch64-05.mbox.centos.org
Relocations : (not relocatable)
Packager    : CentOS Buildsys <bugs@centos.org>
Vendor      : CentOS
URL         : http://www.mysql.com
Summary     : The MySQL server and related files
Description :
MySQL is a multi-user, multi-threaded SQL database server. MySQL is a
client/server implementation consisting of a server daemon (mysqld)
and many different client programs and libraries. This package contains
the MySQL server and some accompanying files and directories.
~~~

启动

~~~bash
$ systemctl start mysqld
# 查看下服务启动顺序, 以及启动前做的准备, 停止后做的收尾
$ systemctl status mysqld
~~~

命令工具

~~~bash
[root@liyao ~]# ls /usr/bin/ | grep mysql
mysql
mysqladmin
mysqlbinlog
mysqlcheck
mysql_config
mysql_config_editor
mysqld_pre_systemd
mysqldump
mysqldumpslow
mysqlimport
mysql_migrate_keyring
mysqlpump
mysql_secure_installation
mysqlshow
mysqlslap
mysql_ssl_rsa_setup
mysql_tzinfo_to_sql
mysql_upgrade

[root@liyao ~]# which mysqld
/usr/sbin/mysqld
~~~

设置密码, 初始密码在 /var/log/mysqld.log

登陆

~~~bash
$ mysql -u root -p
$ mysql -h127.0.0.1 -uroot -P3306 -p

# 退出 or exit;
>quit; 

# 必须重制密码(长度、大小写、特殊字符)
> ALTER USER USER() IDENTIFIED BY 'Mysql8-primary@2023';
> ALTER USER USER() IDENTIFIED BY 'Mysql8-secondary1@2023';
> ALTER USER USER() IDENTIFIED BY 'Mysql8-secondary2@2023';
~~~

配置文件 cnf

~~~bash
[root@liyao mysql]# mysql --help | grep my.cnf
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
/etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf 
~~~

查看下配置

~~~bash
vim /etc/my.cnf.d/mysql-server.cnf
~~~

~~~bash
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysql/mysqld.log
pid-file=/run/mysqld/mysqld.pid
~~~

可以看到datadir的位置, 也可以换种方式获取

~~~mysql
mysql> show variables like "datadir";
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.00 sec)
~~~

Tips: 支持like模糊查询

~~~mysql
mysql> show variables like 'max_connections';
mysql> show variables like 'default_storage_engine';
~~~

#### 术语

|      术语      |              说明              |
| :------------: | :----------------------------: |
| 数据库管理系统 | SQL database management system |
|     数据库     |            database            |
|     数据表     |             table              |
|  数据行(记录)  |          rows, record          |
|  数据列(字段)  |        columns,  field         |
|    存储引擎    |         storage engine         |

#### 主从复制

修改配置(修改了配置重启)

~~~bash
# primary
server-id=1
log-bin=mysql-bin

# secondary1
server-id=2

# secondary2
server-id=3
~~~

创建复制用户

~~~bash
> create user 'replicator'@'%' identified by 'Mysql8-replicator@2023';
> grant all privileges on *.* to 'replicator'@'%' with grant option;
> flush privileges;
~~~

master节点

~~~
> reset master;	
> show master status;
~~~

slave

复制节点需要登陆一次复制用户.

~~~bash
> stop slave;
> reset slave;
> change master to master_host='10.211.55.61',master_user='replicator',master_port=3306,master_password='Mysql8-replicator@2023',master_log_file='mysql-bin.000001',master_log_pos=157;

> set global read_only=1;
> show global variables like '%read_only%';
~~~

