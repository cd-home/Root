[TOC]

### Linux

#### help

~~~bash
info [command]
man  [command]
whereis [command]
~~~

#### Dir、File

1. 基本操作

> 创建、查询、删除、移动、查看文件内容

~~~bash
mkdir testdir
mkdir -p testPdir/testCdir

find ./ | wc -l. # 查看当前目录下文件个数

rm file
rm -rf noEmptyDir
rm *log

mv file/Dir toPath

cp -r sourceDir destDir
~~~

说明: -r 一般是递归、-f是强制

2. 切换

~~~bash
cd path
cd -
cd or cd ~
cd ..  # 上级目录
pwd
~~~

3. 查看

~~~bash
ls
ls -l.      # 权限
ls -rlt     # 按照时间排序
ls | cat -n # 添加行号
ls -a
~~~

4. 查找目录及文件

~~~bash
find ./ -name "Git*" | xargs file
find ./ -name "*.go"
find ./ -name "*.png" -exec rm {} \;  # 递归当前目录删除.png文件
~~~

5. 查看文件内容

~~~bash
cat -n
ls -al | more
# 前
head -1 filename
head -10 **
# 后
tail -5 filname
# 查看差别
diff file1 file2
# 动态显示文本最新信息
tail -f api.log
~~~

6. 查找文件内容

~~~bash
egrep "error: auth"  api.log
~~~

7. 文件、目录权限

~~~bash
chmod 777 file/Dir
chomd -R 
chomd a+x run.sh
~~~

8. 创建软、硬链接

~~~bash
ln cc ccHardLinkAgain
ln -s cc ccSoftLinkAgain
~~~

