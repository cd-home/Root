[TOC]

### Transport Data

HTTP方式传输数据

#### Header

- [ ] [请求] 自定义 请求头 Header, 例如 Auth: Token xxxxx
- [ ] [响应] Cookie, 服务端设置后, 下次浏览器可自动携带返回

#### Request

##### GET URL

1. Content-Type类型

~~~
Content-Type: application/x-www-form-urlencoded;
~~~

2. GET请求方法参数传递键值对

~~~
?name=yao&age=27
~~~

##### POST Body

服务端通常是根据首部中的 **Content-Type** 字段来获知请求中的消息主体是用何种方式编码, 再对主体进行解析

###### URL

1. Content-Type类型是如下形式

~~~
Content-Type: application/x-www-form-urlencoded;
~~~

2. POST参数传递: 以key1=value1&key2=value2封装到body

###### JSON

1. Content-Type类型是如下形式

~~~
Content-Type: application/json
~~~

2. POST参数传递：将数据序列化到body

###### File: multipart/form-data

1. 最初的HTTP协议没有规定上传文件的功能, 后续添加Content-Type类型, 允许上传文件
2. POST方法
3. 可以提交普通键值对, 也可以提交(上传)(多个)文件

~~~
Content-Type: multipart/form-data; boundary=<calculated when request is sent>
~~~

~~~html
<form action="/upload" method="POST" enctype="multipart/form-data">
     <input type="file" name="file" />
     <button type="submit">上传</button>
</form>
~~~

boundary, 是增加的分割符号, 如下是body

~~~
--{boundary}
Content-Disposition: form-data; name="key"
\r\n
value
--{boundary}
Content-Disposition: form-data; name="test.txt"; filename="test.txt"
Content-Type: text/plain
\r\n
<文件内容>
--{boundary}--
\r\n
~~~

注意如下消息头：作为multipart/form-data请求的消息头

~~~
Content-Disposition: form-data; name="title"
~~~

###### Binary

1. Content-Type

~~~
Content-Type: application/octet-stream
~~~

2. 二进制数据写入Body即可

#### Response

##### JSON

1. 设置响应头

~~~
Content-Type: application/json
~~~

2. 序列化语言数据为JSON字符串数据

##### File

1. 设置响应头

~~~
Content-Disposition: inline								 # 网页显示
Content-Disposition: attachment							 # 下载
Content-Disposition: attachment; filename="filename.jpg" # 下载, 附带默认文件名
~~~

2. 文件二进制数据写入Response

##### Binary

1. 设置响应头

~~~
Content-Type: application/octet-stream
~~~

2. 文件二进制数据写入Response

除了以上的Content-Type外还有其他常用的MIME类型

~~~
text/plain
text/html; charset=utf-8
image/gif
image/jpeg
mage/png
application/pdf
pplication/msword
~~~