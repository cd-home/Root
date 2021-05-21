[TOC]

# 第六章HTTP首部

## 6.1 HTTP报文首部

1.  请求报文
    *   报文首部
        *   请求行 [HTTP请求方法、URL、协议/版本]
        *   首部 [请求首部、通用首部、实体首部]
    *   空行
    *   报文主体
2.  响应报文
    *   报文首部
        *   响应行 [协议/版本 状态码 状态短语]
        *   首部 [响应首部、通用首部、实体首部]
    *   空行
    *   报文主体

## 6.2 HTTP首部字段

1.  HTTP首部传递重要信息
    *   给浏览器或者服务器提供报文主体大小、使用语言、认证信息
2.  结构

~~~
字段名: 字段值
Content-Type: text/html
Keep-Alive: timeout=15, max=100
~~~

3.  通用首部字段

|      字段名       |           说明           |      |
| :---------------: | :----------------------: | ---- |
| **Cache-Control** |       控制缓存行为       |      |
|    Connection     |         连接管理         |      |
|       Date        |       创建报文日期       |      |
|    **Pragma**     |     HTTP缓存报文指令     |      |
|      Trailer      |     报文末端首部一览     |      |
| Transfer-Encoding | 指定报文主体传输编码方式 |      |
|    **Upgrade**    |      升级为其他协议      |      |
|        Via        |      代理服务器信息      |      |
|      Waring       |         错误通知         |      |

4.  请求首部字段

|        字段名         |           说明           |      |
| :-------------------: | :----------------------: | ---- |
|        Accept         | 用户代理可处理的媒体类型 |      |
|    Accept-Charset     |        优先字符集        |      |
|    Accept-Encoding    |      优先的内容编码      |      |
|    Accpet-Language    |         优先语言         |      |
|     Authorization     |         认证信息         |      |
|        Expect         |    期待服务器特定行为    |      |
|         From          |     用户电子邮件地址     |      |
|         Host          |    请求资源所在服务器    |      |
|       If-Match        |     比较实体标记Etag     |      |
| **If-Modified-Since** |     比较资源更新时间     |      |
|   **If-None-Match**   |       比较实体标记       |      |
|       If-Range        |  发送实体的请求byte范围  |      |
|  If-Unmodified-Since  |     比较资源更新时间     |      |
|     Max-Forwards      |       最大传输跳数       |      |
|  Proxy-Authorization  |  代理服务器要求认证信息  |      |
|       **Range**       |       实体字节范围       |      |
|        Referer        |   请求URL的原始获取方    |      |
|          TE           |      传输编码优先级      |      |
|      User-Agent       |    HTTP客户端程序信息    |      |

5.  响应首部字段

|        字段        |             说明             |      |
| :----------------: | :--------------------------: | ---- |
|   Accpet-Ranges    |     是否接受字节请求范围     |      |
|        Age         |     推算资源创建经过时间     |      |
|      **Etag**      |         资源匹配信息         |      |
|      Location      |    令客户端重定向指定URL     |      |
| Proxy-Authenticate | 代理服务器堆客户端的认证信息 |      |
|    Retry-After     |    再次发起请求的时机要求    |      |
|       Server       |      HTTP服务器安装信息      |      |
|        Vary        |   代理服务器缓存的管理信息   |      |
|  WWW-Authenticate  |   服务器堆客户端的认证信息   |      |
|     **Allow**      |       资源支持HTTP方法       |      |
|  Content-Encoding  |      实体主体适用的编码      |      |
|  Content-Language  |         实体自然语言         |      |
| **Content-Length** |          主体的大小          |      |
|  Content-Location  |        替代资源的URL         |      |
|    Content-MD5     |      实体主体的报文摘要      |      |
|   Content-Range    |      实体的主体位置范围      |      |
|  **Content-Type**  |      实体主体的媒体类型      |      |
|      Expires       |   实体主体的过期的日期时间   |      |
| **Last-Modified**  |       资源最后修改时间       |      |

在 HTTP 协议通信交互中使用到的首部字段，不限于 RFC2616 中定义的 47 种首部字段。还有 Cookie、Set-Cookie 和 Content-Disposition 等在其他 RFC 中定义的首部字段，它们的使用频率也很高

## 6.3 HTTP通用首部

1.  **Cache-Control** 

    **控制浏览器的缓存行为**

~~~
Cache-Control: private, max-age=0, no-cache
~~~

​	缓存请求指令

~~~
no-cache          不缓存， 我不要缓存服务器的，我要服务器资源，防止从缓存中拿到过期数据
no-store          不缓存或响应任何内容
max-age=[秒]  必须 响应最大的Age值
....
~~~

​	 缓存响应指令

~~~
public            可向任意方提供缓存
private           仅向特定用户返回响应
no-cache          服务器返回，缓存服务器不能进行缓存
no-store
max-age=[秒]  必须 响应最大Age值
s-maxage=[秒] 必须 公共缓存响应最大Age值
~~~

2.  具体的说明

    *   public 明确指出其他的用户也可以利用缓存

    *   private 缓存服务器只会对特定用户提供资源缓存的服务,其他用户发送的请求,代理服务器不会返回缓存

    *   no-cache 

        *   防止从缓存中返回过期的资源
        *   客户端发送的请求中包括no-cache，则表示客户端不接收缓存过的响应，会转发给源服务器获取
        *   服务端返回的响应中包括no-cache，则表示源服务器以后也将不再对缓存服务器请求中提出的资源有效性进行确认，且禁止其对响应资源进行缓存操作

    *   no-store

        *   暗示请求或响应中包含机密信息
        *   规定缓存不能在本地存储请求或响应的任一部分，no-store才是真正的不进行缓存

    *   s-maxage

        *   指定缓存期限

    *   max-age

        *   客户端发送请求包含max-age，判断max-age是否小于缓存资源的缓存时间，小于则直接返回缓存资源，当max-age=0时，缓存服务器直接转发请求给服务器
        *   当服务器返回的响应中包含 max-age 指令时，缓存服务器将不对资源的有效性再作确认，而 max-age 数值代表资源保存为缓存的最长时间
        *    HTTP/1.1 版本的缓存服务器遇到同时存在 Expires 首部字段的情况时，会优先处理 max-age 指令

    *   Connection 

        *   **控制不再转发给代理的首部字段**

        ~~~
        Connection: Upgrade  // 删除这个字段
        ~~~

        *   **管理持久连接**

        ~~~
        Keep-Alive: timeout=10, max=500 // HTTP1.0需要加上
        Connection: Keep-Alive
        ~~~

    *   Date

        *   创建HTTP报文的日期

        ~~~
        Date: Tue, 03 Jul 2012 04:40:59 GMT
        ~~~

    *   Pragma

        *   HTTP1.0遗留问题，不返回缓存的资源
        *   通常和Cache-Control一起使用

        ~~~
        Cache-Control: no-cache
        Pragma: no-cache
        ~~~

    *   Upgrade

        *   首部字段 Upgrade 用于检测 HTTP 协议及其他协议是否可使用更高的版本进行通信
        *   对于附有首部字段 Upgrade 的请求，服务器可用 101 Switching Protocols 状态码作为响应返回

## 6.4 HTTP请求首部字段

1.  Accept 接受的媒体类型

~~~
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
~~~

~~~
application/xml;q=0.9   // q = 0 - 1  代表权重 
~~~

2.  Accept-Charset 接受的字符集

~~~
Accept-Charset: iso-8859-5, unicode-1-1;q=0.8
~~~

3.  Accept-Encoding 接受的压缩方式

~~~
Accept-Encoding: gzip, deflate
~~~

4.  Accept-Language   接受的语言

~~~
Accept-Language: zh-cn,zh;q=0.7,en-us,en;q=0.3
~~~

5.  Authorization 认证

~~~
Authorization: Basic dWVub3NlbjpwYXNzd29yZA==
~~~

6.  **If-Match**

    *   附加条件请求，服务器判断满足条件后才执行请求
    *   使用星号（*）指定 If-Match 的字段值. 服务器将会忽略 ETag 的值，只要资源存在就处理请求

7.  **If-None-Match**

    *   只有在 If-None-Match 的字段值与 ETag 值不一致时，可处理该请求
    *   所以可以用来获取最新的资源

8.  **If-Modified-Since**

    *   资源在 Since时间后更新过就，处理返回 Last-Modified
    *   没有更新过，返回304，让HTTP缓存处理

    ~~~
    If-Modified-Since: Thu, 15 Apr 2004 00:00:00 GMT
    ~~~

9.  **If-Unmodified-Since**

    *   资源在指定日期之后，未发生更新，才做处理请求
    *   发生了更新，返回 412 Fail

10.  **If-Range**

     *   If-Range的值和Etag一致，那么就处理范围请求

     ~~~
     If-Range: "etag"
     Range: bytes=5001-10000
     ~~~

11.  Max-Forwards
     *   代理服务器转发次数
12.  Proxy-Authorization
13.  **Range**
     *   范围请求 206 Partial Content 
     *   无法处理返回 200 OK 全部资源
14.  Referer 原始URL
15.  **User-Agent 客户端种类**

## 6.5 响应首部字段

1.  Accept-Ranges

    *   能否接受范围请求

    ~~~
    Accpet-Ranges: bytes  // 可以 
    Accpet-Ranges: none   // 不能接受    
    ~~~

2.  Age

    *   源服务器告诉客户端在多久前创建了响应

    ~~~
    Age: 600
    ~~~

3.  ETag

    *   资源被缓存时，就会被分配唯一性标识，服务器为资源分配的唯一标示
    *   资源更新，ETag就会更新

4.  Location

    *   引导客户端重定向到新的URL

5.  Retry-After

    *   服务器告诉客户端多久之后再来访问

6.  Server

    *   服务器名称信息

7.  Vary

## 6.6 实体首部字段

实体首部字段是包含在请求报文和响应报文中的**实体部分**所使用的首部

1.  Allow

    *   允许的请求方法

    ~~~
    Allow: GET, HEAD
    ~~~

2.  Content-Encoding

    *   实体压缩

    ~~~
    Content-Encoding: gzip
    ~~~

3.  Content-Language

4.  Content-Length

5.  Content-MD5

    *   报文主体 + MD5 + base编码
    *   **客户端会对接收的报文主体执行相同的 MD5 算法，然后与首部字段 Content-MD5 的字段值比较**
    *   目的在于检查报文主体在传输过程中是否保持完整，以及确认传输到达

6.  Content-Range

    ~~~
    Content-Range: bytes 5001-10000/10000
    ~~~

7.  Content-Type

    ~~~
    Content-Type: text/html; charset=UTF-8
    ~~~

8.  Expires
    *   首部字段 Expires 会将资源失效的日期告知客户端。缓存服务器在接收到含有首部字段 Expires 的响应后，会以缓存来应答请求，在 Expires 字段值指定的时间之前，响应的副本会一直被保存。当超过指定的时间后，缓存服务器在请求发送过来时，会转向源服务器请求资源
    *   源服务器不希望缓存服务器对资源缓存时，最好在 Expires 字段内写入与首部字段 Date 相同的时间值
    *   当首部字段 Cache-Control 有指定 max-age 指令时，比起首部字段 Expires，会优先处理 max-age 指令

9.  Last-Modified
    *   资源最后修改时间

## 6.7 为Cookie服务的字段

Cookie 的工作机制是用户识别及状态管理

| Set-Cookie | 开始状态管理所使用的Cookie信息 | 响应首部字段 |
| ---------- | ------------------------------ | ------------ |
| Cookie     | 服务器接收到的Cookie信息       | 请求首部字段 |

1.  Set-Cookie

    | 属性         | 说明                                                         |
    | ------------ | ------------------------------------------------------------ |
    | name=value   | 赋予 Cookie 的名称和其值（必需项）                           |
    | expires=DATE | Cookie 的有效期（若不明确指定则默认为浏览器关闭前为止）      |
    | path=PATH    | 将服务器上的文件目录作为Cookie的适用对象（不指定默认为文档所在的文件目录） |
    | domain=域名  | 作为 Cookie 适用对象的域名 （若不指定则默认为创建 Cookie 的服务器的域名） |
    | Secure       | 仅在 HTTPS 安全通信时才会发送 Cookie                         |
    | HttpOnly     | 加以限制，使 Cookie 不能被 JavaScript 脚本访问               |