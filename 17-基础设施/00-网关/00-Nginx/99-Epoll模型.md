### 高性能

#### 核心模型

多进程的工作方式(也支持多线程的模式),主流的方式是多进程的模式,多进程有很多的优点

![nginx模型](./images/nginx%E6%A8%A1%E5%9E%8B.svg?lastModify=1658821803)



说明

1. nginx启动后,master进程管理worker进程,监控与重启worker
2. 多个worker之间是平等竞争来自客户端的请求,各个worker之间互相独立
3. worker的数目一般设置为CPU的核心数目

处理请求

1. nginx启动,建立socket的listen,然后从master进程fork出worker进程

2. 每个worker都有自己的socket,每个socket监听的都是同一个IP与端口（但是每个socket是不同的）

3. 请求到来,所有的worker都收到了通知,但是只有一个worker能接受到,其他的失败

    > nginx通过锁的机制,让只有一个worker接受该请求
    >
    > 同一时刻,就只会有一个进程在accpet连接,这样就不会有惊群问题了

4. 读取请求、解析请求、处理请求、返回数据

worker连接池

1. 每个worker都有一个独立的连接池,连接池的大小是worker_connections,但是这里的连接池不是以往我们理解的连接池,而是一个连接的链表结构,简单来说只是表示连接的数目,建立连接取出,用完后再放回去
2. 最大的连接数worker_connections * worker_processes/2 （作为反向代理）因为作为反向代理服务器,每个并发会建立与客户端的连接和与后端服务的连接,会占用两个连接