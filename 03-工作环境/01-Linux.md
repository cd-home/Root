[TOC]

### Linux

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
grep -c 'error: ' api.log
# 显示匹配行、以及行号 -i 忽略大小写
grep -n -i 'error: ' api.log

# 递归搜索
grep "class" . -R -n
~~~

#### 磁盘管理

#### 进程管理

~~~bash
$ ps -ef
$ top
$ lsof -i:3306
$ kill pid 	   # kill -9 pid
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

