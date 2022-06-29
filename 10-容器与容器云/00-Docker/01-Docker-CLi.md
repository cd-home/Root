[TOC]

### Docker-CLi

#### 镜像 					 image

##### 搜索			docker search

~~~bash
$ docker search <image-name>:<tag>
~~~

##### 拉取			docker pull

>  tag 可选, 默认latest, 并不一定代表最新

```bash
$ docker image pull <image-name>:tag
$ docker pull <image-name>:tag
```

##### 查看			docker images

IMAGE ID 或者 REPOSITORY:TAG 标示唯一镜像,  以下使用IMAGE ID表示

```bash
$ docker images	 	
$ docker image ls
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE
```

##### 删除			docker rmi

```bash
$ docker image rm <image-name>:tag
$ docker image rm IMAGEID
$ docker rmi <image-name>:tag
$ docker rmi IMAGEID
```

##### 标签			docker tag

~~~bash
$ docker tag IMAGEID <image-name>:v1.0.0
~~~

##### 查看历史	docker history

~~~bash
$ docker history IMAGEID
~~~

##### 镜像信息	docker inspect

~~~bash
$ docker inspect IMAGEID
~~~

##### 推送仓库	docker push

需要特定的标签[地址]才可以推送

~~~bash
$ docker tag demo.docker.com/demo-project:tag
$ docker login -u xxx -p xxx
$ docker push demo.docker.com/demo-project:tag
~~~

##### 保存归档	docker save

```bash
$ docker save IMAGEID -o dest_images.tar
$ docker save IMAGEID | gzip > dest_images.tar.gz
```

##### 加载			docker load

~~~bash
$ docker load -i dest_images.tar.gz
~~~

##### 构建镜像	docker build

`docker build` command builds Docker images from a Dockerfile and a "context". A build’s context is the set of files located in the specified `PATH` or `URL`. The build process can refer to any of the files in the context.

docker build 命令从 Dockerfile 和"上下文"构建 Docker 镜像. 构建的上下文是位于指定 PATH 或 URL 中的文件集. 构建过程可以引用上下文中的任何文件. 

~~~bash
$ docker build [OPTIONS] Context(PATH|URL)
~~~

通常情况下 Context 用 ., 表示当前目录下即为构建上下文(将上下文的文件发送到Docker Daemon)

**常用OPTIONS参数详细描述**

|   option    |                     | 说明                          |
| :---------: | ------------------- | ----------------------------- |
|     -f      | --file=./Dockerfile | 指定Dockerfile,默认当前目录下 |
| --no-cache  |                     | 不采用缓存构建                |
| --platform  |                     | 指定构建平台                  |
|    --rm     | --rm=true           | 构建成功后删除                |
|     -t      | --tag               | 设置构建后镜像名称            |
|  --target   | --target=           | 选择不同的构建阶段FROM        |
| --build-arg | --build-arg k=v     | 设置构建时变量, 可覆盖ARG     |
|     -o      | --output            | 构建完成输出                  |

#### 容器					container

 \<container-name>  \<container-id> 标识唯一镜像, 后续命令采用\<container-id>

##### 创建 			docker create

~~~bash
$ docker container create -it --name <container-name> <image-name>:tag
~~~

##### 运行 			docker run

Docker runs processes in isolated containers. A container is a process which runs on a host. The host may be local or remote. When an operator executes `docker run`, the container process that runs is isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.

Docker在隔离的容器中运行进程. 容器是在主机上运行的进程. 主机可以是本地的, 也可以是远程的. 当执行docker run时, 运行的容器进程是隔离的, 因为它有自己的文件系统、网络和、主机分离的独立进程树. 

**注意 容器子命令 container 省略, 大多数情况下亦是如此使用**

~~~bash
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
~~~

Example

~~~bash
$ docker run -it --name alpine_o --rm alpine:latest /bin/sh
~~~

With the `docker run [OPTIONS]` an operator can add to or override the image defaults set by a developer. And, additionally, operators can override nearly all the defaults set by the Docker runtime itself. 

使用 `docker run [OPTIONS]` 可以添加或覆盖开发人员设置的镜像默认值.  此外, 还可以覆盖几乎所有由 Docker 运行时本身设置的默认值. 

**常用OPTIONS参数详细描述**

|    option    |                    |                         说明                         |
| :----------: | :----------------: | :--------------------------------------------------: |
|      -d      |   --detach=true    |                 守护式容器,后台运行                  |
|      -i      | --interactive=true |                   交互模式运行容器                   |
|      -t      |     --tty=true     |                分配一个终端, 通常-it                 |
|    --name    |      --name=       |            为创建的容器命名,标识唯一容器             |
|      -p      |    --publish=[]    |                       端口映射                       |
|    --net     |  --network=bridge  |   设置网络模式bridge、host、container、define_net    |
|    --link    |      --link=       |                      链接到网络                      |
|      -e      | -e k1=v1 -e k2=v2  |                  为容器设置环境变量                  |
| --entrypoint |   --entrypoint=    |                    覆盖容器入口点                    |
|  --env-file  |    ---env-file=    |             通过环境变量文件设置环境变量             |
|  --restart   |     --restart=     | no、always、on-failure[:max-retries]、unless-stopped |
|     --rm     |     --rm=true      |           退出删除, 一次性容器. 无重启策略           |
|      -v      | --volume=src:dest  |              host:container 数据卷挂载               |
|      -w      |   --workdir="/"    |                 设置工作路径,默认是/                 |

**注意：env-file使用key=value解析, 不能使用引号**

**说明：run options 部分参数可以覆盖Dockerfile**

1. [COMMAND] [ARG...] => CMD 
2. --entrypoint                 => ENTRYPOINT
3. -e / --env-file              => ENV 
4. --expose                          => EXPOSE
5. -v                                        => VALUME

**重启策略Policy细节**

1. always: 

    无论退出状态如何, 始终重新启动容器, 除非主动停止; 

    Docker 守护进程重启, 亦会重启策略为always的容器

2. unless-stopped: 

    无论退出状态如何, 始终重启容器, 包括在守护进程启动时, 除非容器在 Docker 守护进程停止之前进入停止状态

3. on-failure[:max-retries] 

    仅当容器以非零退出状态退出时才重新启动. 

    可选限制 Docker守护程序尝试重新启动的次数 on-failure:3

##### 进入容器	docker exec

```bash
$ docker exec <container-id>
$ docker exec -it <container-id> /bin/bash 
```

##### 查看容器	docker ps

```bash
$ docker container ls 
$ docker container ls --all
# 所有运行中的容器
$ docker ps
# 所有容器
$ docker ps --all
$ docker ps -a
```

##### 停止启动	docker start/stop/restart/kill

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

##### 删除容器	docker rm

通常删除容器, 1 停止 2 删除 两步

```bash
$ docker rm <container-id>
# 强制删除
$ docker rm -f <container-id>
# 删除容器并且删除数据盘
$ docker rm -v <container-id>  
```

##### 保存镜像	docker commit

从容器创建新镜像

```bash
$ docker commit <container-id> <image-name>:tag
$ docker commit -m "desp" -a "author" <container-id> <image-name>:tag
```

##### 容器导出	docker export

~~~bash
$ docker export <container-id> > dest_container.tar
~~~

##### 容器导入	docker import

导入容器快照到镜像库

~~~bash
$ cat dest_container.tar | docker import - test/xx:v1.0.0
$ docker import URL example/imagerepo
~~~

##### 容器信息	docker inspect

~~~bash
$ docker inspect <container-id>
~~~

##### 容器名字	docker rename

~~~bash
$ docker rename <container-id> newContainerName
~~~

##### 容器日志	docker logs

~~~bash
$ docker logs -f
~~~

##### 容器端口	docker port

~~~bash
$ docker port <container-id>
~~~

##### 复制文件	docker cp

~~~bash
$ docker cp <container-id>:src dest
~~~
