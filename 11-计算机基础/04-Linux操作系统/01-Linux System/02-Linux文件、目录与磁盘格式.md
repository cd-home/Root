[TOC]

### Linux文件、目录与磁盘格式

#### 文件权限与目录配置

 **一般将文件可存取的身份分为三个类别，分别是 owner/group/others，且三种身份各有 read/write/execute 等权限**

1. 拥有者
2. 群组
3. 其他

文件权限

~~~bash
$ ls -lh
$ ll
# 权限 连接数 拥有者 群组 最后修改时间 名字
drwxrwxr-x. 2 admin admin 6 Jun 20 01:57 testperm
-rw-rw-r--. 1 admin admin 0 Jun 20 01:57 testperm.txt
~~~

**第一列代表权限，第一个字符代表类型**

~~~bash
- 文件
d 目录
l 链接文件
b 可随机存储设备
c 一次性读取设备
~~~

后面每三个权限一组(无权限为-)，第一组是拥有者、第二组是群组、第三组是其他人

~~~bash
r 可读
w 可写
x 可执行
~~~

改变所属群组(只能修改存在的群组, /etc/group)

~~~bash
$ chgrp xxgroup [-R] dir/file
~~~

改变拥有者

~~~bash
$ chown xxUser [-R] dir/file
~~~

改变权限

**亦可用数字表示发表示权限：4 2 1**

~~~bash
# owner = rwx = 4+2+1 = 7
# group = rwx = 4+2+1 = 7
# others= --- = 0+0+0 = 0
$ chmod [-R] 666 dir/file
~~~

符号改变权限

~~~bash
# user group others, all = u, g, o, a 来代表三种身份的权限 a 则代表 all 亦即全部的
# r w x 读 写 可执行
# + 加入权限
# - 除去权限
# = 设置权限
# 已有权限上增加x
$ chmod u+x build.sh 
# 直接赋权限
$ chmod u=rwx build.sh
# 去掉权限
$ chmod a-x build.sh
~~~

**注意：开放进入目录权限(cd)是给x, 不是给r**, w不要随便给, w可以删除、修改、更新、新增

