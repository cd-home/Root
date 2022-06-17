[TOC]

### Shell

> 命令行工具、Linux命令、默认Bash终端

#### 系统工作

##### echo

~~~bash
echo "Hello Shell"
~~~

##### date

> 获取时间、亦可设置时间

~~~bash
date "+%Y-%m-%d %H:%M:%S %j" 
date -s "20220505 17:09:00"
~~~

##### reboot

> 重启系统

##### poweroff

> 关闭系统

##### wget

> 下载网络文件

~~~bash
wget host # -b 后台 -P 下载指定目录 -t 最大尝试次数 -c 断点续传 -p 下载页面 -r 递归下载
~~~

##### ps

> 查看进程状态(当前静态)

~~~bash
ps -aux
~~~

##### top

> 查看进程状态(动态)

~~~bash
top
~~~

##### pidof

> 某个进程服务的PID

~~~bash
pidof xxservice
~~~

##### kill

> 杀死进程

~~~bash
kill PID
~~~

##### killall

> 杀死服务的所有进程

~~~bash
killall xxservice
~~~

#### 系统检测

##### ifconfig

> 获取网卡、网络配置 (网卡名称、IP地址、MAC地址)

##### uname

> 系统信息

~~~bash
uname -a
cat /etc/centos-release
~~~

##### uptime

> 系统负载

##### free

> 系统内存使用量

~~~bash
free -h
~~~

##### who

> 当前登录用户、以及打开终端

##### history

> 执行的历史命令

~~~bash
history
!10 # 执行显示的历史命令
~~~

#### 工作目录

##### pwd

> 显示当前所在目录路径(工作目录)

~~~bash
pwd
~~~

##### cd

> 切换工作路径

~~~bash
cd relpath or abspath
cd ..	# 上一级
cd ~	# home
cd -	# last
~~~

##### ls

> 显示目录下文件信息

~~~bash
ls -l
ls -al	# -a 隐藏文件
~~~

#### 文本编辑

##### cat

> 获取文本内容

