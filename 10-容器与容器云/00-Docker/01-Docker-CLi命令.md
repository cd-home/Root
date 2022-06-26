[TOC]

### Docker基础命令

#### 镜像

##### 搜索

~~~bash
$ docker search <image-name>:<tag>
~~~

##### 拉取

>  tag 可选, 默认latest, 并不一定代表最新

```bash
$ docker image pull <image-name>[:tag] 
```

##### 查看

> IMAGE ID 或者 REPOSITORY:TAG 标示唯一镜像， 以下使用IMAGE ID表示

```bash
$ docker images	 	
$ docker image ls
REPOSITORY            TAG                 IMAGE ID            CREATED           SIZE
```

##### 删除

```bash
$ docker image rm <image-name>:tag
$ docker image rm IMAGEID
$ docker rmi <image-name>:tag
$ docker rmi IMAGEID
```

##### 标签

~~~bash
$ docker tag IMAGEID <image-name>:v1
~~~

##### 查看历史

~~~bash
$ docker history IMAGEID
~~~

##### 详细信息

~~~bash
$ docker inspect IMAGEID
~~~

##### 标签

~~~bash
$ docker tag <new-image-name>:tag
$ docker tag demo.docker.com/demo-project:tag
~~~

##### 推送仓库

>   需要特定的标签[地址]才可以推送

~~~bash
$ docker login -u xxx -p xxx
$ docker push demo.docker.com/demo-project:tag
~~~

##### 保存归档

```bash
$ docker save IMAGEID -o xxx_images.tar
$ docker save IMAGEID | gzip > xxx_images.tar.gz
```

##### 加载

~~~bash
$ docker load -i xxx_images.tar.gz
~~~

#### 容器

> 以下 \<container-name>  \<container-id> 标识唯一镜像, 并且容器子命令 container 可以省略

##### 创建

> 镜像创建容器

~~~bash
$ docker container create -it --name <container-name> <image-name>:tag
~~~

##### 创建&运行

~~~bash
$ docker container run [options] <image-name>:tag [向启动容器中传入的CMD命令]
~~~

**注意， 后面的容器子命令 container 省略, 大多数情况下亦是如此使用**

Example

~~~bash
$ docker container run -it --rm alpine:latest /bin/sh
~~~

**TODO 常用参数详细描述**

###### --name 

指定容器名, 需要显式指定



|    option    |         全          |             说明             |              例子              |
| :----------: | :-----------------: | :--------------------------: | :----------------------------: |
|      -i      | --interactive=true  |      以交互模式运行容器      |                                |
|      -t      |     --tty=true      |   容器启动后会进入其命令行   |                                |
|    --name    |                     |       为创建的容器命名       |                                |
|      -d      |   --detach=false    | 创建一个守护式容器在后台运行 |                                |
|      -p      |    --publish=[]     |      端口映射, 可以多个      |          -p 8080:8080          |
|      -P      | --publish-all=false |           暴露端口           |                                |
|      -e      |        --env        |      为容器设置环境变量      |              k=v               |
|     --rm     |      --rm=true      |     退出删除, 一次性容器     |                                |
| --entrypoint |   --entrypoint=""   |        覆盖容器入口点        |                                |
|  --env-file  |     --env-file      |         环境变量文件         |                                |
|  --restart   |                     |           重启策略           | always、unless-stop、on-failed |

**注意：env-file使用key=value解析，不能使用引号**

##### 进入容器

```bash
$ docker exec <container-id>
$ docker exec -it <container-id> /bin/bash 
```

##### 查看容器

```bash
$ docker container ls 
$ docker container ls --all
# 所有运行中的容器
$ docker ps
# 所有容器
$ docker ps --all
$ docker ps -a
```

##### 停止启动

```bash
# 停止一个已经在运行的容器, 不会损坏容器与其中文件
$ docker stop <container-id>

# 启动一个已经停止的容器
$ docker start <container-id>

# 重启启一个已经停止的容器
$ docker restart <container-id>

# kill掉一个已经在运行的容器
$ docker kill <container-id>
```

##### 删除容器

> 通常删除容器, 1 停止 2 删除 两步

```bash
$ docker rm <container-id>
# 强制删除
$ docker rm -f <container-id>
# 删除容器并且删除数据盘
$ docker rm -v <container-id>  
```

##### 保存镜像

```bash
$ docker commit <container-id> <image-name>:tag
$ docker commit -m "desp" -a "author" <container-id> <image-name>:tag
```

##### 容器导出

~~~bash
$ docker export <container-id> > xx_container.tar
~~~

##### 容器导入

> 导入容器快照到镜像库

~~~bash
$ cat xx_container.tar | docker import - test/xx:v1.0.0
$ docker import URL example/imagerepo
~~~

##### 容器信息

~~~bash
$ docker inspect <container-id>
$ docker rename <container-id> <newName>
~~~

##### 查看端口

~~~bash
$ docker port IMAGEID
~~~
