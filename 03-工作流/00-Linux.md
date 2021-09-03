[TOC]

### Linux

#### 帮助命令

~~~bash
$ info [command]
$ man  [command]
$ whereis [command]
~~~

#### 文件夹文件

1. 基本操作

> 创建、查询、删除、移动、查看文件内容

~~~bash
$ mkdir testdir
$ mkdir -p testPdir/testCdir

$ find ./ | wc -l. # 查看当前目录下文件个数

$ rm file
$ rm -rf noEmptyDir
$ rm *log

$ mv file/Dir toPath

$ cp -r sourceDir destDir
~~~

说明: -r 一般是递归、-f是强制

2. 切换

~~~bash
$ cd path
$ cd -
$ cd or cd ~
$ cd ..  # 上级目录
$ pwd
~~~

3. 查看

~~~bash
$ ls
$ ls -l.      # 权限
$ ls -rlt     # 按照时间排序
$ ls | cat -n # 添加行号
$ ls -a		# 隐藏文件
$ ls -lh      # 文件详细大小等 
~~~

4. 查找目录及文件

~~~bash
$ find ./ -name "Git*" | xargs file
$ find ./ -name "*.go"
$ find ./ -name "*.png" -exec rm {} \;  # 递归当前目录删除.png文件
~~~

5. 查看文件内容

~~~bash
$ cat -n
$ ls -al | more
# 前
$ head -1 filename
$ head -10 **
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

#### 文本处理

1. find

~~~bash
$ find . ! -name "*.pdf"
$ find . ! -name "*.pdf"
~~~

#### 磁盘管理

#### 进程管理

~~~bash
$ ps -ef
$ top
$ lsof -i:3306
$ kill pid # kill -9 pid
$ pmap pid
~~~

#### 性能监控

#### 网络工具

~~~bash
$ netstat -a  # all port
$ netstat -at # tcp port
$ netstat -l  # running

$ wget url 
$ ssh user@host
$ sftp user@host

$ scp localpath ID@host:path
$ scp -r ID@site:path localpath
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
