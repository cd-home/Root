[TOC]

### Docker Compose

> 单引擎模式下进行多容器部署与管理

#### 声明式配置文件yaml

~~~yaml
version: "3"
services:
  redis:
    image: redis:alpine
    container_name: redis
    # CMD
    command: redis-server --requirepass 123456x
    # 网络
    networks:
      counter-net:
  web:
    depends_on:
      - redis
    build:
      # 构建上下文
      context: app
      dockerfile: app/Dockerfile
    container_name: webapp
    environment:
      ENV: DEBUG
    expose:
      - 8080
    ports:
      - "8080:8080"
    networks:
      counter-net:
networks:
  counter-net:
    driver: bridge
~~~

#### 生命周期管理命令

~~~bash
$ docker compose start 
$ docker compose stop
$ docker compose restart
$ docker compose up [-d]
$ docker compose down
$ docker compose ps
$ docker compose top
~~~
