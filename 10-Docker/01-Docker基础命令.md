[TOC]

### Docker命令

#### 镜像

##### 搜索

~~~bash
$ docker search <image-name>:<tag>
~~~

##### 拉取

>  tag 可选, 默认latest

```bash
$ docker image pull <image-name>[:tag] 
```

##### 查看

> IMAGE ID 或者 REPOSITORY:TAG 标示唯一镜像， 以下使用IMAGE ID表示

```bash
$ docker images
$ docker image ls
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
```

##### 删除

```bash
$ docker image rm <image-name>:tag
$ docker image rm IMAGEID
# 简写
$ docker rmi <image-name>:tag
$ docker rmi IMAGEID
```

##### 保存

```bash
$ docker save IMAGEID -o xxx_images.tar
```

##### 重命名

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

##### 查看端口

~~~bash
$ docker port IMAGEID
~~~

#### 容器

##### 创建

~~~bash
$ docker create -it --name <container-name> <image-name>:tag
~~~

##### 创建&运行

```bash
docker run [options] <image-name>:tag [向启动容器中传入的命令]
```

|    option     |         全          |             说明             |          例子          | 必要 |
| :-----------: | :-----------------: | :--------------------------: | :--------------------: | ---- |
|      -i       | --interactive=false |      以交互模式运行容器      |                        |      |
|      -t       |     --tty=false     |   容器启动后会进入其命令行   |                        |      |
|    --name     |                     |       为创建的容器命名       |                        |      |
|      -v       |     --volume=[]     |    目录映射关系，可以多个    | 宿主机目录:容器中目录  |      |
| --volume-from |  --volume-from=[]   |          容器卷挂载          |                        |      |
|      -d       |   --detach=false    | 创建一个守护式容器在后台运行 |                        |      |
|      -p       |    --publish=[]     |           端口映射           |      -p 8080:8080      |      |
|      -P       | --publish-all=false |           暴露端口           |                        |      |
|      -e       |      --env=[]       |      为容器设置环境变量      |                        |      |
|     -net      |   --net="bridge"    |           设置网络           |       --net host       |      |
|     --rm      |     --rm=false      |     退出删除, 一次性容器     |                        |      |
|    --link     |      --link=[]      |        容器互联, 单机        |   --link name:alias    |      |
|   --restart   |   --restart="on"    |             重启             | no、always、on-failure |      |
|      -u       |      --user=""      |          容器用户名          |                        |      |
|      -a       |     --attach=[]     |           登陆容器           |                        |      |
|      -w       |    --workdir=[]     |         容器工作目录         |                        |      |
|      -c       |   --cpu-shares=0    |         容器CPU权重          |                        |      |
|      -m       |     --memery=""     |         容器内存上限         |                        |      |
|     --dns     |      --dns=[]       |        容器DNS服务器         |                        |      |
| --dns-search  |   --dns-search=[]   |        容器DNS搜素域         |                        |      |
| --entrypoint  |   --entrypoint=""   |        覆盖容器入口点        |                        |      |
|  --env-file   |    --env-file=[]    |         环境变量文件         |                        |      |

2. 进入容器

```bash
docker exec <container-id>
docker exec <container-name>
docker exec -it <container-id> /bin/bash 
```

3. 查看容器

```bash
docker container ls 
docker container ls --all
# 所有运行中的容器
docker ps
# 所有容器
docker ps --all
docker ps -a
```

4. 停止启动

```bash
# 停止一个已经在运行的容器
docker container stop <container-name>
docker container stop <container-id>

# 启动一个已经停止的容器
docker container start <container-name>
docker container start <container-id>

# kill掉一个已经在运行的容器
docker container kill <container-name>
docker container kill <container-id>
```

5. 删除容器

```bash
docker container rm <container-id>
docker container rm <container-name>
docker container rm -f <container-id>
docker container rm -f <container-name>
docker rm <container-id>
docker rm <container-name>
 # 删除容器并且删除数据盘
docker rm -v <container-id>
```

6. 容器保存镜像

```bash
docker commit <container-name> <image-name>
docker commit <container-id> <image-name>
docker commit -m "desp" -a "author" <container-id> <image-name>:tag
```

7. 容器导出

~~~
docker export <container-id>
docker export <container-name>
~~~

8.  其他操作

~~~bash
docker logs
docker inspect <container-name>
docker rename <container-name> <newName>
docker top
~~~

9.  数据盘

~~~bash
docker volume ls
docker volume ls -f dangling=true # 未使用的数据盘
docker volume rm volumeName
~~~

