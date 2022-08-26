[TOC]

### Docker Compose

》单引擎模式下进行多容器部署与管理

#### 声明式配置文件yaml

~~~yaml
version: "3"
services:
  redis:
  	# 镜像 (变量: 读环境变量或者当前.env)
    image: redis:${REDIS_VERSION}
    container_name: redis
    # CMD
    command: redis-server --requirepass 123456x
    # 网络
    networks:
      - counter-net
  web:
    depends_on:
      - redis
    build:
      # 构建上下文
      context: app
      dockerfile: app/Dockerfile
    container_name: webapp
    args:
    	arch: amd64
    # 环境变量
    env_file: 
    	- ./base.env
    	- ./dev.env
    environment:
      	mode: DEBUG
      	app: admin
    # 同 EXPOSE 暴露端口
    expose:
      - 8080
    # 端口映射, 建议使用引号模式
    ports:
      - "8080:8080"
    restart: always
    networks:
      - counter-net
    # 数据卷挂载
    volumes:
      - /var/run/logs:/var/run/logs
    # 同 WORKDIR  
    working_dir: /app/release/
    # 同 ENTRYPOINT
    entrypoint: ["./app"]
# 网络
networks:
  counter-net:
    driver: bridge
~~~

#### 管理命令

~~~bash
$ docker compose start 
$ docker compose stop
$ docker compose restart
$ docker compose up [-d]
$ docker compose down
$ docker compose ps
$ docker compose top
$ docker compose images
$ docker compose logs
$ docker compose exec
$ docker compose port
# 指定容器名称, 就无法scale
$ docker compose scale web=3 db=2
~~~

Options

| option |                     |                    |
| :----: | ------------------- | ------------------ |
|   -f   | --file Name         | docker-compose文件 |
|   -p   | --project-name Name | 项目名称           |

Example Simple Dockerfile

~~~dockerfile
FROM golang:alpine AS Builder
WORKDIR /app/build/
ENV GOOS=linux CGO_ENABLED=0 GOARCH=amd64 GOPROXY=https://goproxy.cn,direct
COPY . .
RUN go build -o app -ldflags="-s -w" main.go

FROM scratch
WORKDIR /app/release/
COPY --from=0 /app/build/app /app/release/
EXPOSE 8999
ENTRYPOINT ["./app"]
~~~

在Dockerfile中设置的命令, 无必要在Compose中重复设置
