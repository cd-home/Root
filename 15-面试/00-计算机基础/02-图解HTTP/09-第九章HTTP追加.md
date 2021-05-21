# 第九章 HTTP追加

## WebScoket

Web 浏览器与 Web 服务器之间全双工通信标准

通信过程中可互相发送 JSON、XML、HTML 或图片等任意格式的数据

## 特点

1.  推送功能
2.  减少通信量

## 连接

1.  握手请求, 需要使用HTTP的Upgrade字段，升级协议

~~~
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
~~~

2.   握手响应

~~~
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
~~~

​	Sec-WebSocket-Accept 的字段值是由握手请求中的 Sec-WebSocket-Key 的字段值生成的

3.  成功握手确立 WebSocket 连接之后，通信时不再使用 HTTP 的数据帧，而采用 WebSocket 独立的数据帧