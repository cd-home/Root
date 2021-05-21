[TOC]

### 编程设计及通用理论

#### 理论1: 流 ？ IO操作？阻塞？epoll与IO多路复用？

##### 流

1.  可以进行IO操作的对象
2.  文件、管道、套接字
3.  流的入口：文件描述符

##### IO操作

1.  对流的读写操作

##### 阻塞

1.  阻塞等待，啥也不干，浪费资源，不占CPU时间片，不能处理多个IO请求
2.  非阻塞轮询，瞎忙，做无用功，占CPU
3.  IO多路复用，处理多个IO请求
    *   既能阻塞等待不浪费资源
    *   又可以监听大量的IO状态

##### 解决方案

1.  阻塞+多线程/多进程, 高资源消耗
2.  非阻塞忙轮询，CPU占用高
3.  select 多路复用
    *   多个平台可用
    *   有监听的数目大小限制
    *   不会精确的告知哪些准备好，需要去遍历全部的流

~~~
for {
	select [stream1，stream2...]  // 阻塞监听流，有数据就会通知
	// 需要遍历全部的流
	for stream in [stream1，stream2...] {
		if stream has data {
			// code
		}
	}
}
~~~

##### epoll

###### 概念

*   Linux操作系统
*   **通知有哪些IO操作和数目准备好**，只关心活跃的连接，无需遍历所有的描述符
*   能处理大量的链接请求，系统可以打开文件的数目

~~~
for {
	// 监听可使用的流 
	Can_Do_Stream[] = epoll_wait(epoll_fd)
	// 处理
	for stram in Can_Do_Stream[] {
		// code
	}
}
~~~

###### API

1.  创建epoll

~~~c
int create_epoll(int size);    // 监听数目
int epfd = create_epoll(1000); // epfd 作为红黑树的根节点
~~~

2.  控制

~~~c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
~~~

3.  等待epoll

~~~c
int epoll_wait(int epfd, struct epoll_event *event, int maxevents, int timeout);
~~~

