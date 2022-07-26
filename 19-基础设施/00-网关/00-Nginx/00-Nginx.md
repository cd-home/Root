[TOC]

## Nginx

Nginx是一款高性能的 HTTP服务器和反向代理服务器

1. 跨平台：Nginx 可以在大多数服务器编译运行
2. 非阻塞、高并发连接：数据复制时,磁盘I/O的第一阶段是非阻塞的
3. 内存消耗小
4. 稳定性高
5. 事件驱动: 通信机制采用epoll模型,支持更多的的并发连接,并发数越大(不超过最大)可以最大化发挥epoll模型

官方测试能够支撑5万并发连接,实际生产环境中2～3万并发连接数.(Nginx使用epoll模型)

**作用**

1. 负载均衡,降低负载,提高吞吐量
2. 静态服务器,缓存静态内容
3. 隐藏服务端,保护作用
4. 容灾处理(转发到正常的服务器上)
5. 恢复添加

说明

1. 代理服务器

    一般是指局域网内部的机器通过代理服务器发送请求到互联网上的服务器

    代理服务器一般作用在客户端

2. 正向代理

    作为客户端的代理向服务器发送请求获取响应,客户端是不透明的,服务器端是不知道访问它的是客户端还是代理

3. 反向代理

    作为服务端的代理,接收客户端的请求然后转发到服务器获取到数据后,返回给客户端,服务端是不透明的

    所以客户端是不知道它访问的是客户端还是代理

4. 基本图示如下

![代理服务器](images/代理服务器.svg)

由反向代理于防火墙的情况下,可以有效的保护原始的服务器

### 安装

mac

~~~bash
 brew install nginx
~~~

服务启动

~~~bash
brew services start nginx
brew services restart nginx
brew services stop nginx
~~~

Linux

安装(yum管理)

~~~bash
sudo apt-get install nginx
~~~

服务器(systemctl管理)

~~~bash
sudo service nginx start
sudo service nginx restart
sudo service nginx stop
~~~

Docker

~~~bash
docker pull nginx:latest
docker run -d -p 80:80 --name ng nginx:latest 
~~~
