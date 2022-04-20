[TOC]

### Linux

> 资源 https://linuxtools-rst.readthedocs.io/zh_CN/latest/index.html

#### 帮助命令

~~~bash
$ info [command]
$ man  [command]
$ whereis [command]
~~~

#### 文件及目录管理

1. 基本操作

> 创建、查询、删除、移动、查看文件内容

~~~bash
$ mkdir testdir

# 递归创建
$ mkdir -p testPdir/testCdir

# 查看当前目录下文件个数
$ find ./ | wc -l 

$ rm file
# 强制删除
$ rm -rf noEmptyDir
# 匹配删除 == find ./ name "*log -exec rm {}"
$ rm *log

$ mv file/Dir toPath

# 通常情况下: -r 一般是递归、-f是强制
$ cp -r sourceDir destDir
~~~

2. 切换路径

~~~bash
$ cd abspath
$ cd -
$ cd or cd ~
$ cd ./xxpath # 相对路径
$ cd ..  	  # 上级目录
$ pwd
~~~

3. 查看

~~~bash
$ ls
$ ls -l       # 权限
$ ls -rlt     # 按照时间排序
$ ls | cat -n # 添加行号
$ ls -a		  # 隐藏文件
$ ls -lh      # 文件详细大小等 
~~~

4. 查找目录及文件

~~~bash
$ find ./ -name "Git*" | xargs file
$ find ./ -name "*.go"
# 递归当前目录删除.png文件
$ find ./ -name "*.png" -exec rm {}
~~~

5. 查看文件内容

~~~bash
$ cat -n
$ ls -al | more

# 前
$ head -10 filename

# 后
$ tail -5 filname

# 查看差别
$ diff file1 file2

# 动态显示文本最新信息
$ tail -f api.log
~~~

6. 查找文件内容

~~~bash
$ egrep "error: auth"  api.log
~~~

7. 文件、目录权限

~~~bash
$ chmod 777 file/Dir
$ chomd -R 
$ chomd a+x run.sh
~~~

8. 创建软、硬链接

~~~bash
$ ln cc ccHardLinkAgain
$ ln -s cc ccSoftLinkAgain
~~~

9. 管道

~~~bash
$ ls -lh | head -n 2
~~~

10. 重定向

~~~bash
$ head -n 10 1.txt > 2.txt  # >> 追加
~~~

11. 权限修改(TODO)

~~~bash
# 读写、执行权限
chmod +x fileName
# 改变拥有者
chown -R 
~~~

12. 链接（别名）

~~~bash
# 硬链接， 删除一个，另一个仍然可使用
ln xx xxother
# 软链接，不能删除源，否则无法使用
ln -c yy yyother
~~~

13. 重定向

~~~bash
echo "Hello" > 1.txt
# 追加
echo "Hello" >> 1.txt
# 清空文件
:> 1.txt
~~~

14. 管道

~~~bash
# 批量命令执行，前面的结果应用到后面的命令
ls -al | egrep opt
~~~

15. 组合多命令

~~~bash
# 前面成功，后面才执行
cd .. && pwd
# 前面失败，后面才执行
./proc || ls 
# 串联多命令，无关联关系
ls ; pwd
~~~

#### 文本处理

1. 文件find 查找

~~~bash
# 查找多个 
$ find . \( -name "*.txt" -o -name "*.pdf" \) -print

# 正则查找 特殊符号需要转义 -iregex： 忽略大小写的正则 
$ find . -regex  '.*\(\.txt|\.pdf\)$'

# 否定查找
$ find . ! -name "*.pdf"
$ find . ! -name "*.pdf"

# 类型定制搜索 
# type d f l  目录、文件、链接
# -maxdepth 深度
$ file xxfile  # 检测文件类型
$ find . -maxdepth 1 -type f

# 时间定制搜索
# 最近第7天
$ find . -atime 7 -type f -print
# 最近7天内
$ find . -atime -7 -type f -print
# 7天前
$ find . -atime +7 type f -print

# 定制权限查找
$ find . -type f -perm 644 -print

# 定制用户查找
$ find . -user weber -print
~~~

2. grep 文本搜索

~~~bash
# 统计数量
$ grep -c 'error: ' api.log
# 显示匹配行、以及行号 -i 忽略大小写
$ grep -n -i 'error: ' api.log

# 递归搜索
$ grep "class" . -R -n

# 综合 tr 转换
$ cat LOG.* | tr a-z A-Z | grep "FROM " | grep "WHERE" > b
~~~

3. xargs

~~~bash
# 单行输出
$ cat file.txt| xargs

# 多行输出 -n 每行的字数 -d 定义定界符 -0：指定0为输入定界符
$ cat single.txt | xargs -n 3

# 查找行数
$ find source_dir/ -type f -name "*.cpp" -print0 |xargs -0 wc -l
~~~

4. 排序

~~~bash
# -n 数字排序 -r 逆序 -d 字典排序 -k 按第几行排序
$ sort -n file.txt
~~~

5. 消除重复行

~~~bash
# 消除重复行
$ sort unsort.txt | uniq

# 统计各行在文件中出现次数
$ sort unsort.txt | uniq -c

# 找出重复行
$ sort unsort.txt | uniq -d
~~~

6. 统计

~~~bash
# -l 统计行  -w 统计词 -c 统计字符
$ wc -l file.txt
~~~

7. 文本替换

~~~bash
# 首处替换
$ sed 's/text/replace_text/' file.txt

# 全局替换 输出内容
$ sed 's/text/replace_text/g' file.txt
# 替换文件内容
$ sed -i 's/text/repalce_text/g' file.txt
# 移除空白行
$ sed '/^$/d' file.txt
# 变量替换 []
$ echo this is en example | sed 's/\w+/[&]/g'
~~~

8. awk

~~~bash
# awk ' BEGIN{ statements } {statement} END{ statements } '
# statement 是对每行做操作
# $0 当前行、NR行数、NF字段数
$ awk 'BEGIN {print "begin"} {print NR, $0} END {print "end"}' README.m

# 综合应用
$ ps -fe| grep msv8 | grep -v MFORWARD | awk '{print $2}' | xargs kill -9;
~~~

#### 磁盘管理

1. 磁盘空间

~~~bash
$ df -h
~~~

2. 当前目录所占空间

~~~bash
$ du -sh
$ du -sh `ls` | sort
~~~

3. 打包, 解包

~~~bash
# 打包
tar -cvf target.tar xxfilespath
# 解包
tar -xvf target.tar
~~~

4. 压缩, 解压

~~~bash
$ gzip xxfile.tar
$ gunzip xxfile.tar.gz
~~~

#### 进程管理

1. 查找

~~~bash
# 查找正在运行进程信息
$ ps -ef
# 以完整的格式显示所有的进程
$ ps -ajx
# 查找进程
$ pgrep -l mysql
# 端口查进程,得到PID
$ lsof -i:3306
$ ps -ef | grep PID
~~~

2. 杀死

~~~bash
$ kill pid 	   # kill -9 pid
~~~

3. 进程监控(TODO)

~~~bash
# 进入交互页面 m 内存 P cpu
$ top
# 分析线程堆栈
$ pmap pid
~~~

#### 性能监控

1. CPU

~~~bash
# 两个参数表示监控的频率，比如例子中的1和2，表示每秒采样一次，总共采样2次
$ sar 1 2 
# 内存使用情况
$ sar -r 1 2 
# 页面交换
$ sar -W 1 2
~~~

2. 内存

~~~bash
$ free -m
~~~

#### 网络工具

1. 查询网络服务和端口

~~~bash
$ netstat -a  # all port
$ netstat -at # tcp port
$ netstat -l  # running

# 网络服务查询端口
$ netstat -antp | grep 6379
~~~

2. 网络路由

~~~bash
$ route -n
$ ping IP
$ traceroute IP
$ host Domain
$ host IP
~~~

3. 下载

~~~bash
# -c 断点下载 –limit-rate :下载限速 -o：指定日志文件
$ wget url 
~~~

4. 远程

~~~bash
# 登陆服务器 exit 推出
$ ssh host

# 文件 quit 退出
# 本机   lls lcd
# 服务器 ls cd
# put filename 
# get filename
$ sftp host
~~~

5. 网络复制

~~~bash
# 复制
$ scp localpath host:path
# 下载
$ scp -r site:path localpath
~~~

#### 用户管理工具

1. 新增用户

~~~bash
$ useradd -m username
$ userdel -r username
# 切换用户
$ su username 
# 密码 /etc/shadow
~~~

2. 用户组

~~~bash
$ groups
# -G 用户加入多个组 -g 用户加入新的组
$ groups -G groupname username
# 组、权限信息 /etc/passwd /etc/group
~~~

3. 用户权限

~~~bash
$ chmod userMark [+|-] permissionMark
# userMark 			u 用户、g 组、o其他用户、a 所有用户
# permissionMark	r 读、w 写、x 执行
$ chmod a+x

# 数字方式设置 
# 三位八进制数字的形式来表示权限，第一位指定属主的权限，第二位指定组权限，第三位指定其他用户的权限
# 每位通过4(读)、2(写)、1(执行)三种数值的和来确定权限

$ chomd 740 main

# 更改目录拥有者
$ chown -R otheruser ./dir
~~~

4. 环境变量

~~~bash
# 全局 		  /etc/profile /etc/bashrc 
# 用户目录下私有 ~/.profile  ~/.bashrc 
# 登陆系统获得shell进程, 加载环境变量
# 1. /etc/profile 2. ~/.bash_profile 3. ~/.bash_login 4. ~/.profile 5. ~/.bashrc
~~~

#### 系统管理

1. 系统版本

~~~bash
$ uname -a
~~~

2. cpu

~~~bash
$ cat /proc/cpuinfo
~~~

3. 内存信息

~~~bash
$ cat /proc/meminfo
~~~

4. 架构

~~~bash
$ arch
~~~

5. 日期

~~~bash
$ date
$ date +%Y%m%d%H%M%S
~~~

6. 资源限制

~~~bash
$ ulimit -a 
# 句柄
$ ulimit -n
~~~

#### 程序调试

##### GBD

##### DLV

~~~bash
$ pstrack <program-pid>
$ strace -p <process-pid>
$ objdump -d programmer
$ size a.out
~~~

#### 性能优化

