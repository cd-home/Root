[TOC]

# 第二章 简单的HTTP

## 2.1 HTTP通信

1.  HTTP用于客户端和服务端通信的协议
2.  必然有一端是客服端：请求文本、图像等资源
3.  必然是有一端是服务端：响应客户端的资源
4.  HTTP协议可以明确区分客户端和服务端

## 2.2 通过请求和响应的交换达成通信

1.  HTTP协议规定请求从客户端发出，服务端响应请求并且返回
2.  请求报文有以下组成
    *   请求方法
    *   请求URL
    *   协议/版本
    *   可选请求头  Key: Value
    *   空行
    *   内容实体

~~~
GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Mode: navigate
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: BAIDUID=FCB9FE8C8B028EC060BEF28CAE48BC9A:FG=1;
~~~

3.  响应报文
    *   协议/版本 
    *   状态码 状态短语
    *   响应头
    *   空行
    *   响应实体

~~~
HTTP/1.1 200 OK
Cache-Control: private
Connection: keep-alive
Content-Encoding: gzip
Content-Type: text/html;charset=utf-8
Date: Sun, 12 Jul 2020 09:13:52 GMT
Expires: Sun, 12 Jul 2020 09:13:52 GMT
Server: BWS/1.1
Set-Cookie: path=/; domain=.baidu.com
Strict-Transport-Security: max-age=172800
Traceid: 1594545232057327489016884404931870793513
X-Ua-Compatible: IE=Edge,chrome=1
Transfer-Encoding: chunked
~~~

## 2.3 HTTP是无状态的

1.  HTTP协议本身不对请求和响应的通信状态进行保存，即不对已经发送过的请求和返回的响应做持久化
2.  更快的处理事务，确保协议的可伸缩性
3.  由于现代Web业务的发展，需要有状态的服务，所以引入了Cookie机制

## 2.4 请求URL

1.  HTTP协议采用URL定位网络上的资源
2.  协议、服务器地址、资源
3.  URL还可以携带一些信息

## 2.5 告知服务器意图的方法

1.  GET

    *   获取资源

    *   注意GET方法获取资源有HTTP缓存问题

        *   强制缓存

        *   协商缓存

2.  POST 

    *   传输实体主体

    *   GET也可以传输主体，但是一般不用GET

3.  PUT 

    *   传输文件

    *   HTTP1.1 PUT方法不带验证机制，一般的网站不使用PUT

4.  HEAD

    *   获取报文的首部、不返回主体
    *   确认URL的有效性以及资源更新日期

5.  DELETE

    *   删除资源
    *   HTTP1.1 DELETE方法不带验证机制，一般的网站不使用

6.  OPTIONS

    *   询问支持的方法
    *   简单请求
        *   GET/HEAD/POST
        *   请求体非JSON
    *   复杂请求
        *   不符合简单请求就是复杂请求
    *   复杂请求情况下，就会发送OPTIONS请求，询问允许的方法以及是否可以跨域

7.  TRACE

    *   追踪请求的
    *   不常用，容易引发XST攻击

8.  CONNECT

    *   要求用隧道协议连接代理
    *   使用隧道协议建立TCP通信，主要使用SSL和TLS加密后用隧道进行通信



## 2.6 持久连接

1.  HTTP初始版本
    *   建立TCP连接，HTTP请求响应，然后断开
    *   如果要再次请求又必须建立TCP连接，造成通信量的开销
2.  HTTP持久连接
    *   持久连接的好处就是不用频繁的建立TCP连接，避免客重复建立和断开的开销，减轻服务器负载
    *   HTTP1.1中都是默认使用持久化连接，这样一次web页面请求的效率提高并且资源消耗减少

~~~
keep-alive: connection
~~~

3.  管线化
    *   不用等待请求的响应，可以继续发送请求，可以使得请求并行化
    *   但是一个请求必须对应一个响应

## 2.7 Cookie状态管理

1.  无状态情况下，减少资源和内存消耗
2.  Cookie技术通过在请求和响应报文中写入Cookie信息来控制客户端的状态
3.  Cookie技术
    *   服务器在响应报文中设置一个**Set-Cookie的响应头**，通知客户端保存Cookie信息
    *   下一次访问，客户端在请求报文中**携带Cookie信息**
    *   服务器得到Cookie信息，检测Cookie和服务器上的记录，然后得到之前的状态信息