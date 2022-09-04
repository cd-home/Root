[TOC]

### Linux

》一切皆文件; 常用命令的集合;

#### 目录系统

~~~bash
[root@localhost /]# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt 
opt  proc  root  run  sbin  srv  sys  tmp  usr  var
~~~

![Linux_dir](./images/Linux_dir.svg)

#### 文件属性

![Dir_File_Perm](./images/Dir_File_Perm.svg)

#### 命令总览

<img src="./images/Linux-Commands.svg" alt="Linux-Commands"  />

#### 目录管理

| command |    description     |          example           | parameter           |
| :-----: | :----------------: | :------------------------: | ------------------- |
|  mkdir  |      创建文件      |       mkdir testdir        |                     |
|  mkdir  |      递归创建      | mkdir -p testPdir/testCdir |                     |
|  rmdir  |     删除空目录     |       rmdir EmptyDir       |                     |
|   cd    |      切换目录      |          cd path           |                     |
|   cd    |       上一级       |           cd ..            |                     |
|   cd    |        上次        |            cd -            |                     |
|   cd    |        Home        |         cd ~ [cd]          |                     |
|   pwd   |      当前目录      |            pwd             |                     |
|   ls    |   查看路径下文件   |  ls [path] [-a] [-h] [-l]  | -a 隐藏文件 -l 权限 |
|   ll    | 查看路径下文件详细 |    ll [path] [-a] [-h]     | -h 文件大小         |

#### 文件管理

| command |  description   |    example     |  parameter   |
| :-----: | :------------: | :------------: | :----------: |
|  touch  |    创建文件    | touch testfile |              |
|  which  | 查找可执行文件 | which python3  |              |
|   rm    |    删除文件    |  rm testfile   |   rm *log    |
|   vim   | 创建、编辑文件 |  vim testfile  | 具体见vim.md |

#### 资源管理

| command |             description             |                 example                 |         parameter         |
| :-----: | :---------------------------------: | :-------------------------------------: | :-----------------------: |
|   rm    |                删除                 |     rm testfile、rm -rf noEmptyDir      |      -rf 强制、递归       |
|   mv    |            移动、重命名             | mv source\_[Dir\|file] dest_[Dir\|file] |                           |
|   cp    |                复制                 | mv source\_[Dir\|file] dest_[Dir\|file] |        -r 递归复制        |
|  file   |              类型检测               |             file dir\|file              |                           |
| locate  |                位置                 |            locate dir\|file             |                           |
|  stat   |              详细信息               |             stat dir\|file              |                           |
| rename  |               重命名                |           rename old new old            |                           |
|  chmod  |                权限                 |        chmod [-R] 777 Dir\|File         | 4 2 1 模式[读、写、执行]  |
|  chmod  |                权限                 |        chomd [-R] a+x Dir\| File        | [u g o a] [+ - =] [r w x] |
|         | u 用户、g 组、o其他用户、a 所有用户 |           r 读、w 写、x 执行            |         op + - =          |
|   tar   |                打包                 |      tar -cvf target.tar dest_dir       |                           |
|   tar   |                解包                 |           tar -xvf target.tar           |                           |
|   tar   |                压缩                 |   tar -zcvf target.tar.gz source_dir    |                           |
|   tar   |                查看                 |          tar -tf target.tar.gz          |                           |
|   tar   |                解压                 |   tar -zxvf target.tar.gz -C dest_dir   |                           |
|  gzip   |                压缩                 |          gzip -rv ./source_dir          |         -rv 递归          |
|  gzip   |                解压                 |             gzip -dr test/              |                           |
| gunzip  |                解压                 |           gunzip dest.tar.gz            |                           |
|   zip   |                压缩                 |         zip dest.zip source_dir         |                           |
|  unzip  |                解压                 |             unzip dest.zip              |                           |
|   ln    |                链接                 |          ln [-s]  source dest           |         -s 软链接         |

#### 文件查看

~~~bash
# ------------------------------------[所有]----------------------------------------
# 获取所有
$ cat -n
 
# ------------------------------------[局部]----------------------------------------
# 前 n 行
$ head -10 testfile

# 后 n 行
$ tail -5 testfile

# ------------------------------------[动态]----------------------------------------
# 动态显示文本最新信息
$ tail -f api.log
$ tail -n +20 api.log

# 查看差别
$ diff testfile1 testfile2

# 翻页查看
$ more testfile
$ less testfile

# ------------------------------------[匹配]----------------------------------------
# 匹配查看
$ egrep "error: auth"  api.log

# ------------------------------------[过滤]----------------------------------------
# 通常作用在其他命令的管道的结果: ps -ef | grep nginx
# 显示匹配行、以及行号 -i 忽略大小写
$ grep -n -i 'error: ' api.log
# 匹配多个模式:
$ grep -e "error" -e "info" testfile
~~~

#### 文件编辑

~~~bash
# ------------------------------------[vim 查看、编辑]-------------------------------
$ vim testfile

# ------------------------------------[sed 查看、编辑]-------------------------------
# 首处替换
$ sed 's/text/replace_text/' testfile

# 全局替换 输出内容 -g
$ sed 's/text/replace_text/g' testfile

# 替换文件内容
$ sed -i 's/text/repalce_text/g' testfile

# 移除空白行
$ sed '/^$/d' testfile

# 变量替换 []
$ echo this is en example | sed 's/\w+/[&]/g'
~~~

#### 用户管理

组、权限信息 /etc/group /etc/passwd 

~~~bash
# ------------------------------------[用户组管理]-----------------------------------
# 创建组
$ groupadd gx

# 删除组
$ groupdel g1

# 查看组、查看用户的组
$ groups [user]

# -G 用户加入多个组 -g 用户加入新的组
$ groups -G g1,g2 u1

# ------------------------------------[用户管理]-------------------------------------
# 增加用户
$ useradd ux 
# 增加用户并且添加到组
$ useradd u1 -G g1,g2

# 删除用户
$ userdel [-r] u1

# 修改用户所在组
$ usermod -G g3 u2

# 修改密码
$ passwd
$ passwd u1

# 切换root用户
$ su  
# 切换普通用户
$ su other_user

# root用户执行
$ sudo command
~~~

#### 硬件管理

~~~bash
# 磁盘空间
$ df -h

# 当前目录所占空间
$ du -sh
$ du -sh `ls` | sort
~~~

#### 进程管理

~~~bash
# report a snapshot of the current processes.
# 查找正在运行进程信息
$ ps -ef

# 以完整的格式显示所有的进程
$ ps -ajx

# 查找进程
$ pgrep -l mysql
$ pidof progress_name

# 使用某个端口的进程
$ lsof -i :3306

# 杀死
$ kill -l
$ kill pid 	   

# display sorted information about processes
# 进程实时监控 
$ top

# 分析线程堆栈
$ pmap pid
~~~

#### 性能监控

~~~bash
# ------------------------------------[CPU]-------------------------------------
# 两个参数表示监控的频率, 比如例子中的1和2, 表示每秒采样一次, 总共采样2次
$ sar 1 2 
# 内存使用情况
$ sar -r 1 2 
# 页面交换
$ sar -W 1 2

# ------------------------------------[内存]-------------------------------------
$ free -m
~~~

#### 网络工具

~~~bash
# ------------------------------------[curl]---------------------------------------
$ curl url 

# ------------------------------------[wget]---------------------------------------
# -c 断点下载 –limit-rate :下载限速 -o：指定日志文件
$ wget url 

# ------------------------------------[telnet]-------------------------------------
# telent xinetd
$ telnet host 

# ------------------------------------[SSH]----------------------------------------
# 登陆服务器 ssh-keygen 为 ssh 生成、管理和转换认证密钥，它支持 RSA 和 DSA 两种认证密钥
$ ssh host
$ ssh [-p PORT] USER@HOST

# ------------------------------------[ip]-----------------------------------------
$ ip addr
$ ifconfig
$ ip route
$ ip link

# ------------------------------------[route]--------------------------------------
# 网络路由表查看与设置
$ route

# ------------------------------------[host]---------------------------------------
# 域名测试
$ host Domain
$ host IP
$ nslookup Domain

# ------------------------------------[ping]---------------------------------------
# 网络联通性测试
$ ping IP
# 追踪数据包
$ traceroute IP

# ------------------------------------[sftp]---------------------------------------
# 本机   lls lcd
# 服务器 ls cd
# put filename 
# get filename
$ sftp host

# ------------------------------------[scp]----------------------------------------
# 复制
$ scp localpath host:path
# 下载
$ scp -r site:path localpath

# ------------------------------------[netstat]------------------------------------
# 查询网络服务和端口 t=tcp u=udp p=program
$ netstat -a   # all port
$ netstat -at  # tcp port 
$ netstat -au  # tcp port
$ netstat -l   # running
$ netstat -lt  # running
$ netstat -lu  # running

# 协议统计信息
$ netstat -s
$ netstat -st
$ netstat -su

# 网络服务查询端口
$ netstat -altup | grep 6379

# ------------------------------------[firewalld]----------------------------------
# ------------------------------------[iptables]-----------------------------------
~~~

#### 环境变量

~~~bash
# 获取环境变量
$ env
# 全局 		  
/etc/profile /etc/bashrc

# 用户目录下私有 
~/.profile  ~/.bashrc 

# 登陆系统获得shell进程, 加载环境变量过程 
1. /etc/profile 
2. ~/.bash_profile 
3. ~/.bash_login 
4. ~/.profile 
5. ~/.bashrc
~~~

#### 系统管理

~~~bash
# 开关机
$ rebbot
$ shutdown
# 立即重启
$ shutdown -r now
# 立即关机
$ shutdown -h now 

# 系统版本
$ uname -a

# CPU信息
$ cat /proc/cpuinfo

# 内存信息
$ cat /proc/meminfo

# 系统架构
$ arch

# 日期
$ date
$ date +%Y%m%d%H%M%S

# 系统限制信息
$ ulimit -a 

# 定时任务
$ at
$ crontab

# 文件系统挂载
$ mount
$ mount /dev/hda1 /mnt
$ umount

# 系统服务管理
$ systemctl [stop | start | restart | status | enable | disable] 
$ systemctl daemon-reload
~~~

#### 软件安装

》通常情况下可以更新yum的镜像源(见Linux System)

~~~bash
$ yum search
$ yum list
$ yum install 
$ yum update
$ yum remove
~~~

#### 补充

重定向

~~~bash
$ echo "Hello" > 1.txt
# 追加
$ echo "Hello" >> 1.txt
# 清空文件
$ :> 1.txt
~~~

多命令

~~~bash
# 前面成功, 后面才执行
$ cd .. && pwd
# 前面失败, 后面才执行
$ ./app || ls 
# 串联多命令, 无关联关系
$ ls ; pwd
~~~

**管道**

~~~bash
# 批量命令执行, 前面的结果应用到后面的命令
$ ls -lh | head -n 2
~~~

**文件描述符**

Linux系统预留可三个文件描述符：0、1和2，他们的意义如下所示：

~~~bash
0——标准输入（stdin）
1——标准输出（stdout）
2——标准错误（stderr）
~~~

~~~bash
# 特殊的黑洞设备
/dev/null
$ 2 > /dev/ull
$ >/dev/null 2>&1
$ 2>&1 >/dev/null
~~~

#### 问题排查

~~~bash
$ strace -f -o ./strace.out excute_dev
~~~

#### zsh

》Oh My Zsh 安装与使用即可
