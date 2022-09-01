[TOC]

### 数据传递

#### 前置知识

- [x] HTTP 报文结构
- [x] 请求方法
- [x] Content-Type

#### Header

- [x] Cookie
- [x] 自定义Header

#### URL?

要求

1. Content-Type类型

~~~
Content-Type: application/x-www-form-urlencoded;
~~~

2. GET请求方法参数传递

~~~
www.params.com:8080/api/v1?name=mike
~~~

#### Body

> 服务端通常是根据首部中的 **Content-Type** 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析

##### URL编码

1. Content-Type类型是如下形式

~~~
Content-Type: application/x-www-form-urlencoded;
~~~

2. POST参数传递: 以key1=value1&key2=value2封装到body

##### JSON

1. Content-Type类型是如下形式

~~~
Content-Type: application/json
~~~

2. POST参数传递：将数据序列化到body

##### File: multipart/form-data

1. 最初的HTTP协议没有规定上传文件的功能，后续添加Content-Type类型，允许上传文件
2. 必须是POST方法
3. 可以提交普通键值对，也可以提交(上传)(多个)文件键值对

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

注意如下消息头：要么作为multipart/form-data请求的消息头

~~~
Content-Disposition: form-data; name="title"
~~~

要么作为文件下载的响应头

~~~
Content-Disposition: inline								 # 网页显示
Content-Disposition: attachment							 # 下载
Content-Disposition: attachment; filename="filename.jpg" # 下载，附带默认文件名
~~~