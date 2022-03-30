### Github配置SSH

#### 流程

1. 本地生成公钥

~~~bash
ssh-keygen -t rsa -C liyaoo1995@gmail.com # 提示可以设置密码
~~~

2. .ssh

~~~bash
$ ls
authorized_keys id_rsa id_rsa.pub known_hosts
~~~

3. 复制.pub的内容添加到Github的SSH配置
4. ssh配置 

~~~bash
$ vim ~/.ssh/config # 添加以下内容
Host https://github.com/GodYao1995
HostName github.com
User GodYao1995
IdentityFile ~/.ssh/id_rsa
~~~

5. 添加

~~~bash
$ ssh-add ~/.ssh/id_rsa	# 如果第一步设置了密码，这里需要输入密码
~~~

6. 测试

~~~bash
$ ssh -T git@github.com 
# Hi ! You’ve successfully authenticated, but GitHub does not provide shell access.
~~~

#### 多账号

可以在config中配置多个账号

~~~bash
# 个人账号xxxxx@foxmail.com
Host https://github.com/GodYao1995
HostName github.com
User GodYao1995
IdentityFile ~/.ssh/id_rsa_a

# 公司账号xxx@xxx.com
Host xx.gitlab.com
HostName gitlab.com
User ABC
IdentityFile ~/.ssh/id_rsa_b
~~~

