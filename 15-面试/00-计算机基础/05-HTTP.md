[TOC]

# HTTP

## 前言

1.  HTTP
    *   超文本传输控制协议，传输数据的规范
    *   文字、图片、音频、视频传输
2.  应用层协议—请求-响应模型
    *   客户端：浏览器等HTML渲染
    *   服务端：Go、Python等构件的Web服务器
3.  URI/URL
    *   URI统一资源标示符
    *   URL统一资源定位符，是URI的子集

## HTTP核心

### 请求-响应

1.  流程
    *   DNS解析域名得到IP地址
    *   80端口TCP连接，然后发送HTTP报文，服务端接收
    *   客户端接收到服务端响应，断开TCP连接
    *   将内容渲染到浏览器
2.  特征
    *   客户端-服务器模型
    *   无状态

### 报文格式

#### 请求报文

1.  格式
    *   请求行：请求方法  URL  协议/版本
    *   请求头部
    *   空行
    *   请求实体

2.  请求方法

    GET、POST、PUT、HEAD、DELETE、OPTIONS、TRACE、CONNECT

3.  URL

    协议、主机、端口、路径、?查询、#片段

4.  版本

    HTTP1.1 [Keep-alive  Range]

#### 响应报文



### 标头

#### 通用标头

##### Cache-Control

指令影响是否走缓存服务器

1.  no-cache 
    *   请求
    *   防止从缓存中获取资源

~~~
Cache-Control: no-cache 
~~~

2.  no-store 
    *   请求
    *   不缓存资源

~~~
Cache-Control: no-store
~~~

3.  max-age 
    *   请求和响应
    *   判断资源的缓存时间和max-age的大小，比max-age小则有效
    *   max-age=0 则不走缓存服务器，转发请求到资源服务器

~~~
Cache-Control: max-age=60
~~~

4.  public/private
    *   响应
    *   public表示被任何缓存所缓存，private表示只能缓存特定的客户端

~~~
Cache-Control: public
~~~

5.  s-maxage
    *   响应
    *   与max-age功能相同，只能用于多用户公共服务器

~~~
Cache-Control： s-maxage=60
~~~

6.  min-fresh
    *   请求
    *   要求返回min-fresh时间内的数据

~~~
Cache-Control: min-fresh=60
~~~

7.  max-stable
    *   请求
    *   客户端接收缓存数据，过期也接受

~~~
Cache-Control: max-stable
~~~

8.  only-if-cached
    *   请求
    *   资源在缓存服务器本地目录才返回

~~~
Cache-Control: only-if-cached
~~~

9.  proxy-revalidate
    *   请求
    *   缓存服务器返回响应之前验证资源有效性

~~~
Cache-Control: proxy-revalidate
~~~

10.  no-transform
     *   请求和响应
     *   不能更改主体的媒体类型

~~~
Cache-Control: no-transform
~~~

##### Connection

1.  持久连接，HTTP/1.1默认使用
2.  通常和Keep-Alive一起使用

~~~
// Connection: Close
Connection: Keep-Alive
Keep-Alive: timeout=5, max=1000 // 连接时间，最大请求数
~~~

##### Date

1.  请求和响应中
2.  格林威治标准时间，比北京慢8个小时

##### Pragma

1.  遗留字段，客户端请求中, 一般使用Cache-Control即可
2.  不返回缓存资源

~~~
Pragma: no-cache
~~~

##### Trailer

1.  事先声明在报文主体后记录了哪些字段
2.  用于HTTP/1.1 分块传输

~~~
Transfer-Encoding: chunked
Trailer: Expires
~~~

##### Transfer-Encoding

1.  内容协商，规定传输报文采用的编码格式
2.  属于逐条首部，多个代理节点间可以不同，每一段消息可用不同的transfer-encoding

~~~
Transfer-Encoding: gzip, chunked
~~~

##### Upgrade

1.  升级协议
2.  服务器101 响应返回

~~~
Upgrade: Websocket
Connction: Upgrade
~~~

##### Via

1.  追踪路径
2.  通常和TRACE一起使用

#### 请求标头

##### Accept

1.  客户端能接受什么MIME类型，逗号分隔
2.  类型：文本、图片、视频、二进制文件
3.  q是权重，高到低1 — 0

~~~
Accept: text/html, application/xhtml+xml,application/xml;q=0.9,"/";q=0.8
~~~



#### 响应标头

## 内容协商

## 认证

## 缓存

## 跨域

### 问题

1.  同域：同协议、同主机、同端口
2.  跨域：上面任何一个不同都是跨域
3.  同源策略： 浏览器的同源策略限制从脚本发起跨域请求，一种安全策略
4.  跨域请求：Ajax、Fetch

### 跨域资源共享

1.  􏱥􏱦􏱒􏱓􏱕􏱖􏰷􏷢􏰹􏰭􏹴通过添加HTTP标头来允许服务器描述Web浏览器可以读取的资源

2.  跨域资源共享有简单请求和复杂请求，复杂请求导致预检请求

3.  简单请求

    * GET/HEAD/POST
    * Content-Type: text/plain;  (multipart/form-data /  application/x-www-form-urlencoded )

4.   􏰷􏰸􏳵􏳝􏲋􏳉复杂请求

    *   除了以上的都是复杂请求，比如Content-Type: application/json

5.  解决方案：预检请求响应设置响应头

    ~~~
    Access-Control-Allow-Origin = "*"
    Access-Control-Allow-Methods = "POST, PUT, DELETE"
    Access-Control-Allow-Headers = "Content-Type"
    ~~~

    

    

## Cookie

### Cookie

1.  HTTP无状态协议，Cookie用来存储会话信息
2.  Cookie依赖浏览器，浏览器可以禁用，浏览器关闭则消失，并且有时效
3.  httpOnly可以禁止JS等脚本获取浏览器Cookie
4.  Domain和Path标识了Cookie的作用域
5.  每次请求都会携带上一次的Cookie
6.  响应设置

~~~
Set-Cookie: Key=value; Expire: xxx;
~~~

6.  请求携带

~~~
Cookie: Key=value;
~~~

### Session

1.  服务端维持信息，但是还是需要依赖Cookie

~~~
Cookie: SESSIONID=abc
~~~

2.  Session一致性问题，可以通过共享Session服务，或者JWT解决
3.  禁用Cookie可以采用URL重写的形式将Cookie放置在URL后面

## HTTPS

### 前提

1.  HTTP问题
    *   明文传输
    *   无验证
    *   无完整性校验
2.  HTTPS：安全的超文本传输控制协议  HTTP + SSL(STL) = HTTPS
    *   加密
    *   数据一致性
    *   身份认证
3.  协议HTTPS，端口443
4.  HTTPS不是一项新的应用层协议,只是HTTP的通信层接口部分由SSL或者STL代替【注意目前基本上是TSL传输安全层】
    *   HTTP和TSL通信
    *   TSL和TCP通信
5.  TSL: 密钥交换算法-签名算法-对称加密算法-摘要算法 【根本上是使用对称加密和非对称加密】
6.  对称加密
    *   加密解密使用同样的密钥
    *   保证密钥的安全就保证了数据安全，但是还是可能被截获
7.  非对称加密
    *   也称公钥加密,公钥可以任何人使用,但是私钥只能自己知道，只要自己不泄露私钥那么就是安全的
    *   但是公钥被截获，加密的数据也被截获，那么非对称加密也是不安全的

### HTTPS

1.  采用混合加密
    *   通信开始的时候，使用非对称加密解决密钥交换问题，也就是利用非对称加密传输对称加密密钥
        *   客户端非对称公钥加密【对称加密会话密钥】
        *   服务端非对称私钥解密得到【对称加密会话密钥】
    *   使用对称加密密钥进行通信
2.  完整性：采用摘要算法MD5、SHA-1 SHA-2 HMAC
3.  认证
    *   采用私钥加密，公钥解密
    *   私钥加上摘要算法，形成数字签名
4.  公钥信任：CA认证认证机构
    *   公钥内置浏览器
    *   私钥服务端持有

## HTTP2.0

## Jwt

1.  JSON Web Token
    *   认证：单点登录、可以跨域
    *   信息交换
2.  格式
    *   Header
    *   Payload
    *   Signature
3.  header：令牌类型+签名算法 然后BASE64编码

~~~
{
	"alg": "SHA256",
	"typ": "JWT"
}
~~~

4.  payload： 包含一个声明，三种类型 registered、public、private，BASE64编码

    *   registered

    ~~~
    iss(issuer)  			签发人
    exp(expiration time)    过期时间
    sub(subject)			主体
    aud(audience)			受众
    nbf(not before)			生效时间
    iat(issued At)			签发时间
    jti(JWT ID)				编号
    ~~~

    *   public：公共声明可以添加任何信息，一般添加业务相关，但是不要添加私密信息
    *   private：自定义声明

5.  singature

    *   BASE64后的header
    *   BASE64后的payload
    *   secret
    *   然后进行签名，例如HMACSHA256

6.  最后用.将三段拼接在一起



## HTTP流程

## HTTPS流程