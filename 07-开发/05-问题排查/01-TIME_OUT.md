[TOC]

#### TIME_WAIT排查

#### 问题

线上发现大量的请求无法得到服务器响应

~~~bash
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
~~~

##### 问题来源

1. HTTP协议是基于TCP协议的，并且是由服务器主动断开连接
2. TCP四次挥手，TIME_WAIT出现在主动断开连接的一方，也就是服务器房出现大量TIME_WAIT
3. 采用了Nginx反向代理，Nginx与服务器之间采用短连接的方式，造成大量的TIME_WAIT

##### 解决办法

vim /etc/sysctl.conf   /sbin/sysctl -p

~~~bash
# SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击 默认为0
net.ipv4.tcp_syncookies = 1 
# 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0
net.ipv4.tcp_tw_reuse = 1 
# 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0
net.ipv4.tcp_tw_recycle = 1 
# 修改系默认的 TIMEOUT 时间
net.ipv4.tcp_fin_timeout = 30
~~~

补充命令

~~~bash
ps aux | grep process_name
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
lsof -n|awk '{print $2}'|sort|uniq -c |sort -nr|more
ls /proc/xpid/fd | wc -l
~~~

