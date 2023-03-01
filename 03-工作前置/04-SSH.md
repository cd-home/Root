### Github配置SSH

#### 流程

1. 本地生成公钥

~~~bash
# 提示可以设置密码
$ ssh-keygen -t rsa -C liyaoo1995@gmail.com 
~~~

2. .ssh

~~~bash
$ ls
authorized_keys id_rsa id_rsa.pub known_hosts
~~~

3. 复制.pub的内容添加到代码仓库大的SSH配置管理(例如GitHub)
4. SSH配置 

~~~bash
$ vim ~/.ssh/config # 添加以下内容
Host https://github.com/cd-home
HostName github.com
User cd-home
IdentityFile ~/.ssh/id_rsa
~~~

5. 添加

~~~bash
# 如果第一步设置了密码，这里需要输入密码
$ ssh-add ~/.ssh/id_rsa
~~~

6. 测试

~~~bash
$ ssh -T git@github.com 
# Hi ! You’ve successfully authenticated, but GitHub does not provide shell access.
~~~

#### Tips

配置免密登陆虚拟机

~~~bash
$ ssh-copy-id -i id_rsa.pub root@10.211.55.102
~~~
