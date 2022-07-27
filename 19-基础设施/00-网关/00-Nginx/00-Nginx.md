[TOC]

## Nginx

》http://nginx.org/en/docs/

Nginx是一款高性能的HTTP服务器和反向代理服务器(负载均衡)

1. 跨平台：Nginx 可以在大多数服务器编译运行
2. 非阻塞、高并发连接
3. 事件驱动: 通信机制采用epoll模型,支持更多的的并发连接
4. 内存消耗小
5. 稳定性高

官方测试能够支撑5万并发连接,实际生产环境中2～3万并发连接数

代理

基本图示如下

![代理服务器](images/代理服务器.svg)

### Install

Linux

安装(yum管理)

~~~bash
$ apt-get install nginx
$ yum install nginx
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

### Controlling

At Runtime -s [signal]

~~~bash
$ nginx -s stop
# 重载配置文件
$ nginx -s reload 
$ nginx -s quit
~~~

### Config

/etc/nginx  (or /usr/local/etc/nginx, /usr/local/nginx/conf)

~~~bash
conf.d	fastcgi_params	mime.types  modules  nginx.conf  scgi_params  uwsgi_params
~~~

To make the configuration easier to maintain, we recommend that you split it into a set of feature‑specific files stored in the **/etc/nginx/conf.d** directory and use the [include] directive in the main **nginx.conf** file to reference the contents of the feature‑specific files.

~~~nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}

# top‑level directives
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;
    
	# 可以更加的细化
    # include /etc/nginx/conf.d/*.conf;
    include conf.d/http;
}

# TCP or UDP
# top‑level directives
stream {
    ...
    include conf.d/stream;
}
~~~

更改配置文件后, 可以检测是否正确

~~~bash
$ nginx -t 	
~~~
