[TOC]

### Install

》Linux Centos8 arm64

》需要安装与Erlang版本匹配的 https://www.rabbitmq.com/which-erlang.html

#### Erlang

~~~bash
$ wget https://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm
$ rpm -Uvh erlang-solutions-2.0-1.noarch.rpm 
$ rpm --import https://packages.erlang-solutions.com/rpm/erlang_solutions.asc

$ yum list | grep erlang
$ yum install erlang-24.0.4-1.el8.aarch64
~~~

#### Rabbitmq

~~~bash
$ yum install -y socat
$ wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.35/rabbitmq-server-3.8.35-1.el8.noarch.rpm

$ rpm -Uvh rabbitmq-server-3.8.35-1.el8.noarch.rpm 
$ yum install -y rabbitmq-server
~~~

systemclt 配置启动

Dash界面

~~~bash
$ rabbitmq-plugins enable rabbitmq_management
$ rabbitmqctl add_user [username] [password] 
$ rabbitmqctl set_user_tags [username] [administrator]
$ rabbitmqctl set_permissions -p / [username] ".*" ".*" ".*"
~~~

四种角色

1. administrator：可以登录控制台、查看所有信息、并对rabbitmq进行管理
2. monToring：监控者；登录控制台，查看所有信息
3. policymaker：策略制定者；登录控制台指定策略
4. managment：普通管理员；登录控制

~~~bash
$ rabbitmqctl change_ password 用户名 新密码
$ rabbitmqctl delete_user 用户名
$ rabbitmqctl list_users
~~~

Container(TODO)

~~~bash
~~~

