[TOC]

### 编程设计及通用理论

#### 流

可以进行IO操作的对象如: 文件、管道、套接字; 流的入口: 文件描述符

#### IO

对流的读写操作

#### 阻塞

阻塞等待, 浪费资源, 不占CPU时间片, 不能处理多个IO请求

非阻塞轮询, CPU轮训

IO多路复用, 处理多个IO请求, 既能阻塞等待不浪费资源, 又可以监听大量的IO状态

#### select

select 多路复用: 轮训机制, 有监听的数目大小限制, 性能随着文件描述符增多而下降

~~~
for {
	select [stream1, stream2...]  // 阻塞监听流, 有数据就会通知
	// 需要遍历全部的流
	for stream in [stream1, stream2...] {
		if stream has data {
			// code
		}
	}
}
~~~

#### epoll

支持Linux系统; 事件回调机制, 将就绪的文件描述符加入到就绪队列中,epoll_wait返回直接访问就绪队列就知道哪些文件描述符就绪

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

API

创建epoll

~~~c
int create_epoll(int size);    // 监听数目
int epfd = create_epoll(1000); // epfd 作为红黑树的根节点
~~~

控制

~~~c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
~~~

等待epoll

~~~c
int epoll_wait(int epfd, struct epoll_event *event, int maxevents, int timeout);
~~~

