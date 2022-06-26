[TOC]

### Dockerfile

> 构建镜像

#### 构建上下文

> 构建当前目录就是构建上下文（上下文下所有内容打包到Docker引擎，除.dockerignore），一般情况下Dockerfile也在此目录下

#### FROM

> 指定基础镜像, 注意只能FROM开头，指令大写； 推荐使用alpine镜像；

~~~dockerfile
FROM alpine
~~~

一些经典的镜像：nginx、ubuntu、alpine(linux)、scratch(虚拟空白镜像)、golang

#### LABEL

> 添加标签、记录信息

#### WORKDIR

> 指定工作目录， 通常使用绝对路径

~~~dockerfile
WORKDIR /app/build/
~~~

#### ENV

> 设置环境变量，容器运行时可用

~~~dockerfile
ENV Key Value
# 以下方式可设置多个
ENV Key=Value Key1=Value1
~~~

#### AGR

> 构建时变量(会有FROM作用域), 第一种形式需要在build的时候传入

~~~dockerfile
ARG Key
# 以下方式可设置多个
ARG Key=Value Key1=Value1
~~~

#### EXPOSE

> 暴露端口, 只是一个申明

~~~dockerfile
EXPOSE 80
~~~

#### RUN

> 执行Shell命令，注意每一个RUN都会建立镜像层，所以RUN命令尽量写在一起，减少构建的层

~~~dockerfile
# 通常使用第一种形式，可以复合 && 执行多条 \ 可换行
RUN command			
RUN ['executable', 'param1', 'param1']
~~~

#### COPY

> 构建上下文原因，原文件命令使用的是相对路径(基于上下文)

~~~dockerfile
COPY . .
# 多阶段构建使用，参考下文
COPY --from=Builder /app/build/app /app/release/
~~~

#### ADD

> 基本作用和COPY一致，但是多了URL下载功能、解压功能，下载的在后面不能删除，基本上不推荐使用ADD

~~~dockerfile
ADD ./scripts /app/release/
~~~

#### CMD

> 容器启动命令以及参数, docker run 最后指定的命令即是此， 并且run可以覆盖CMD

~~~dockerfile
CMD command param1 param2
# 推荐第二种形式, 注意可执行文件位置
CMD ["executable", "param1", "param2"...]
~~~

#### ENTRYPOINT

> 容器入口点，指定容器启动以及参数(可通过--entrypoint覆盖)，如果设置了该指令，那么CMD的指令即会作为ENTRYPOINT的参数
>
> 一般认为ENTRYPOINT是主要命令，而CMD作为参数

~~~dockerfile
ENTRYPOINT ["./app", "param1", "param2"]
# 同时存在的情况下
ENTRYPOINT CMD
~~~

docker-entrypoint.sh

> TODO

#### VOLUME

> 数据卷挂载, 避免容器存储层发生写操作

~~~dockerfile
VOLUME /data /data
VOLUME ['/data', '/data']
~~~

#### 多阶段构建

~~~
(base) Li-Yao:docker-oninsm apple$ tree
.
├── Dockerfile
├── go.mod
└── main.go
0 directories, 3 files
~~~

~~~dockerfile
FROM golang:alpine AS Builder
LABEL stage=Build
WORKDIR /app/build/
COPY . .
ENV GOOS=linux CGO_ENABLED=0 GOARCH=amd64
RUN go build -o app -ldflags="-s -w" main.go

FROM alpine:3.14
LABEL stage=Run
WORKDIR /app/release/
COPY --from=Builder /app/build/ /app/release/
EXPOSE 8080
RUN apk update && \
	apk add --no-cache \
	vim && \
	curl && \
	ca-certificates && \
	bash && \
	tzdata && \
	ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
	echo Asia/Shanghai > /etc/timezone
EXPOSE 8999
ENTRYPOINT ["./app"]
~~~

~~~bash
$ docker build -t webServer:v1.0.0 -f Dockerfile . 
~~~

**注意：. 代表的是构建上下文，-f 指定Dockerfile**

#### 最佳实践

- [x] .dockerignore
- [ ] 可读性、注释、分行
- [ ] 选择合适的基础镜像，避免安装不需要的包
- [ ] 多阶段构建，减少容器体积
- [ ] 一个容器允许一个进程
- [ ] 镜像层数尽量少（RUN、COPY、ADD 会增加层数）