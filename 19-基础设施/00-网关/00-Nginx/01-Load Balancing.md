[TOC]

### LB

#### HTTP Load Balancing

~~~nginx
http {
    # Maybe Include From conf.d http
    upstream backend {
        server xxip:port;
    } 
    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
~~~

#### HTTP UDP Load Balancing

~~~nginx
stream {
    upstream stream_backend {
        
    }
    
    server {
        listen ;
        proxy_pass ;
    }
}
~~~



#### 负载均衡算法

1. round-robin

    ==默认的是使用轮询算法==,每个请求按照不同的顺序逐一分配到不同的后端服务器,可以设置轮询的权重weight

    权重越大,服务器被访问的概率就越高

    ~~~nginx
    upstream backend {
        server 127.0.0.1:8000 weight=3;
        server 127.0.0.1:8001;
    }
    ~~~

2. least_conn

    请求会被发送到活跃连接数最少的服务器上

    ~~~nginx
    upstream backend {
        least_conn;
        server 127.0.0.1:8000;
        server 127.0.0.1:8001;
    }
    ~~~

3. ip_hash

    按照访问的ip的哈希结果分配请求,来自同一个ip的用户请求会被分配到固定的后端的服务器

    ~~~nginx
    upstream backend {
        ip_hash;
        server 127.0.0.1:8000;
        server 127.0.0.1:8001;
    }
    ~~~

4. hash

    按照某个键的hansh结果分配

    ~~~nginx
    upstream backend {
        hash $request_uri;      # 请求的hash
        server 127.0.0.1:8000;
        server 127.0.0.1:8001;
    }
    ~~~
