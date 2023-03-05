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
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *event, int maxevents, int timeout);
~~~

epoll默认是水平触发. epoll_ctl把connfd放到epollfd并拷贝到内核态, 有数据时对应connfd复制到readylist; epoll_wait系统调用, 会判断readylist是否为空, 不为空则把fd信息从内核态复制到用户态数组里. 

水平触发: 没有把数据(元素)一次性全部读写完, 那么下次调用epoll_wait()时, 它还会通知你在没读写完的文件描述符上继续读写, 如果你一直不去读写, 会一直通知你. 耗费性能, 但是水平触发相对安全, 最起码事件不会丢掉. 监听端口listenfd产生connfd时要用这个, 不能把产生的connfd丢掉. 

边缘触发：没有把数据(元素)全部读写完, 那么下次调用epoll_wait()时, 它不会通知你, 也就是它只会通知你一次, 直到该文件描述符上出现第二次可读写事件才会通知, 减少了拷贝过程, 增加了性能, 相对来说, 将会产生事件丢的情况. 
