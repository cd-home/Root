[TOC]

### 认证

HTTP是无状态协议

#### Cookie

##### 背景

HTTP协议是无状态的, 即是不记录连接, 每次请求都是独立的

##### 解决问题

解决用户与服务端交互问题, 如登陆状态、购物车、记录信息、追踪用户的行为等。简单来说就是存储信息。

##### 定义

Cookie是服务端发送给浏览器的数据(Set-Cookie)

~~~
Set-Cookie: Key=value; Expire: xxx;
~~~

浏览器提供存储Cookie的功能, 并且在再次发出请求时会携带上当前域下的Cookie(请求头中)发送到服务端

~~~
Cookie: Key=value;
~~~

##### 问题

Cookie存储的数据量并不是很大(4K), 所以如果要存储一些较大的数据, 可以采用localStorage等, 或者其他客户端提供的存储方式。

Cookie受限与浏览器的同源策略。

Cookie是有时效限制。

1. 支持会话时效Expires、并且关闭浏览器Cookie就会销毁。
2. 支持持久性Max-Age

Cookie可以被浏览器禁止。

Cookie可以设置(通过服务端)是否可被Javascript操作。

Cookie 可设置同站点是否发送Cookie, 以避免CSRF。

~~~
Set-Cookie: key=value; SameSite=Strict
~~~

同时, 也可采用以下的方法, 来避免攻击

1. 对用户输入进行过滤来阻止 XSS
2. 任何敏感操作都需要确认
3. 用于敏感信息的 Cookie 只能拥有较短的生命周期

##### 总结

1. Cookie用来保持会话
2. 不应当用来传输存储重要敏感数据
3. 设置Cookie的实效
4. 设置Secure、以及HttpOnly等安全属性

#### Session

依赖Cookie实现SESSIONID存储, 后端通常采用单独的会话服务器

1. SESSION-ID
2. SESSION-DATA

~~~
Cookie: SESSIONID=abc
~~~

Session一致性问题, 可以通过共享Session服务, 或者JWT解决

禁用Cookie可以采用URL重写的形式将Cookie放置在URL后面

#### Token

服务端生成的令牌、可用于携带信息、鉴权

#### Jwt

定义

JSON Web Token

1.  功能
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

4. payload： 包含一个声明, 三种类型 registered、public、private, BASE64编码

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

    *   public：公共声明可以添加任何信息, 一般添加业务相关, 但是不要添加私密信息
    *   private：自定义声明

5. singature

    *   BASE64后的header
    *   BASE64后的payload
    *   secret
    *   然后进行签名, 例如HMACSHA256

6. 最后用.将三段拼接在一起

功能

失效与退出, 此时的JWT 变成了有状态的[不要太严格的强调Token的无状态性]

1. Token黑名单, 定期清理
2. Token白名单
3. 版本号, JWT【用户ID+版本】与存储的用户ID对比, 适用于单设备

#### 总结

- [ ] Session模式是服务端保持会话状态的方法, 通过在Cookie中设置SESSIONID来鉴别用户, 前端无须做操作
- [ ] Token令牌, 服务端生成返回, 浏览器可以在请求头中携带, 常用来做接口的Access Token
- [ ] Jwt, "无状态的[服务端不负责存储信息]", 客户端在请求头中携带, 服务端可以鉴权、获取用户信息, 【**多端适配**】

总之, 虽然模式不同, 但是最终的目的都是为了**传递信息**, 在不同的场景下选择不同的模式即可
