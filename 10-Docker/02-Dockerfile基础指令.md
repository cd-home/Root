[TOC]

### Dockerfile

> 构建镜像

#### 构建上下文

> 构建当前目录就是构建上下文（上下文下所有内容打包到Docker引擎，除.dockerignore），一般情况下Dockerfile也在此目录下

#### FROM

> 指定基础镜像, 注意只能FROM开头，指令大写

~~~dockerfile
FROM alpine
~~~

一些经典的镜像：nginx、ubuntu、alpine(linux)、scratch(虚拟空白镜像)、golang

#### WORKDIR

> 指定工作目录

~~~dockerfile
WORKDIR /app/build/
~~~

#### RUN

> 执行Shell命令，注意每一个RUN都会建立镜像层，所以RUN命令尽量写在一起，减少构建的层

#### COPY

> 构建上下文原因，原文件命令使用的是相对路径(基于上下文)

#### ADD

> 基本作用和COPY一致，但是多了解压功能，但是不推荐使用ADD

#### CMD

> 容器启动命令以及参数, docker run 最后指定的命令即是此， 由此可以通过其覆盖

~~~dockerfile
CMD ["./app", "param1", "param2"]
~~~

#### ENTRYPOINT

> 容器入口点，指定容器启动以及参数(可通过--entrypoint覆盖)，如果设置了该指令，那么CMD的指令即会作为ENTRYPOINT的参数

~~~dockerfile
ENTRYPOINT ["./app", "param1", "param2"]
~~~

#### ENV

> 设置环境变量

~~~dockerfile
ENV Key Value
ENV Key=Value
~~~

#### EXPOSE

> 暴露端口, 只是一个申明

~~~dockerfile
EXPOSE 80
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

FROM scratch
LABEL stage=Run
WORKDIR /app/release/
COPY --from=Builder /app/build/app /app/release/
EXPOSE 8999
ENTRYPOINT ["./app"]
~~~