[TOC]

### Basic Linux Cmd

#### SSH

##### ssh

~~~bash
ssh 192.168.46.133
~~~

##### scp

1. Upload

~~~bash
scp local_path remote_addr:remote_path
scp -r local_path remote_addr:remote_path
~~~

2. Download

~~~bash
scp remote_addr:remote_path local_path 
scp -r remote_addr:remote_path local_path 
~~~

#### Progress

##### ps

> 静态进程信息

~~~bash
$ ps -ef
UID          PID   	PPID  		C 				 STIME 				 					TTY    
启动进程用户  进程ID  父进程ID  CPU利用率 进程启动时系统时间 	进程启动时的终端设备	
TIME 					CMD
运行进程累计 CPU时间启动程序名称
~~~

##### top

> 实时探测进程

~~~bash
~~~

##### kill

> 终止进程

~~~bash
kill -9 # 1 HUP 2 INT 3 结束运行 9 无条件终止
~~~

