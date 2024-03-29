[TOC]

### API接口设计规范

#### 概览

##### 背景

1.  规范接口设计、非固定规范只供参考
2.  便于更新迭代、统一

##### 请求方式

| HTTP请求方式 | 普通意义 |
| :----------: | :------: |
|     GET      |   获取   |
|     POST     |   新增   |
|     PUT      |   修改   |
|    DELETE    |   删除   |

##### 参数

**请求参数**

|  位置  |                   内容                   |
| :----: | :--------------------------------------: |
| Query  |               url?查询参数               |
| Header | 请求头公共参数Token、RequestID、加密字段 |
|  Body  |              POST、PUT参数               |

**公共参数**

APP端

|   参数   |     说明     |        备注         |
| :------: | :----------: | :-----------------: |
| network  |     网络     |      WIFI、4G       |
| operator |    运营商    |    中国联通/移动    |
| platform |     平台     |    iOS、Android     |
|  system  |     系统     | ios 13.3、android 9 |
|  device  |   设备型号   |  iPhone XR、小米9   |
|   uuid   | 设备唯一标示 |                     |
| version  |  API版本号   |   v1.1.0、v1.2.0    |

WEB端

|        参数         |  说明  |  备注  |
| :-----------------: | :----: | :----: |
| appKey/access_token | 授权码 | 字符串 |

**其他**

1.  参数命名一般使用驼峰命名
2.  携带requestID追踪问题

##### 返回

| key  | value  |                    说明                     |
| :--: | :----: | :-----------------------------------------: |
| code |  数字  | 可以三部分组成平台 + 模块 + 具体code 100100 |
| msg  | 字符串 |                返回参数说明                 |
| data |  列表  |                  返回数据                   |

说明：code最好不要复杂，可以直接是0 1 代表成功、失败

##### 安全

**敏感数据**

1.  密码类需要加密传输
2.  使用非对称加密
3.  用户手机号、用户邮箱、身份证号、支付账号、邮寄地址等要进行脱敏，部分数据加 * 号处理

**接口幂等性**

1.  服务端提供一个查询前一个需要幂等性的接口结果查询接口，如果查询成功说明请求成功，否则失败
2.  服务端保证幂等性
    *   调用接口前获取全局唯一Token、调用接口放到Header
    *   服务端解析验证TOken有效性
    *   完成业务逻辑将业务逻辑与Token关联并且设置失效时间
    *   重试时不需要重新获取Token，需要使用上次的Token

#### 接口设计

##### 参数

**接口中提取公共参数**

| 公共参数  |   含意   |                          参数的意义                          |
| :-------: | :------: | :----------------------------------------------------------: |
| timestamp |   毫秒   | 1.请求时间标示 2.请求过期验证 3.该参数参与签名算法增加签名的唯一性 |
|  app_key  | 签名公钥 |        签名算法的公钥，后端通过公钥可以得到对应的私钥        |
|   sign    | 接口签名 | 通过请求的参数和定义好的签名算法生成接口签名，作用防止中间人篡改请求参数 |
|    did    |  设备ID  |   设备的唯一标示，1:数据收集 2.便于问题追踪 3.消息推送标示   |

##### 版本

1. 一般不要超过5个版本
2. URL接口携带或者请求参数携带

##### 安全

1.  过期验证timestamp
2.  签名验证sign
3.  限流
4.  重放攻击

##### 解耦合

抽象公共函数与业务代码

##### 可读性

1.  通用规则RESTAPI
2.  业务响应码和HTTP响应码
3.  接口说明message

##### 文档

#### RESTful

REST从资源的角度类审视整个网络,它将分布在网络中某个节点的资源通过URL进行标识,客户端应用通过URL来获取资源的表征,获得这些表征致使这些应用转变状态.RESTful是一种设计风格,但是逐渐的变成前后端分离的一种"标准"

##### 资源层

网络上面的实体

##### 表现层

不同实体有不同的表达表现形式

##### 状态转移

通过某种手段操作服务器，让服务器发生状态转移，基于表现层

基于HTTP协议的：GET、PUT、POST、DELETE等

##### 总结

客户端通过HTTP协议动词，对服务端资源的状态转移操作，可以称为RESTful架构

##### 设计风格

1.  HTTPS: HTTP安全传输协议, 推荐API与用户的通信协议,总是使用HTTPS

2.  域名体现API

```
 https://api.example.com   # 尽量部属在专属的域名下（存在跨域的问题）
 https://example.org/api/  # 如果api很简单可以部署在主域名下
```

3.  版本

```
 # 版本号尽量放置在URL中
 https://api.example.com/v1/  
 # 请求头中（跨域存在发送多次请求）
 Accept: vnd.example-com.foo+json; version=2.0
```

4. 路径

   资源作为网址,只能有名词,不能有动词,而且所用的名词往往与数据库的表名对应; API中的名词应该使用复数

```
 https://api.example.com/v1/zoos
 https://api.example.com/v1/animals
 https://api.example.com/v1/employees
```

5. HTTP请求方式

   根据请求方式的不同进行不同的操作

|  方式  |             意义              |
| :----: | :---------------------------: |
|  GET   |           获取资源            |
|  POST  |           新增资源            |
|  PUT   |  修改资源/提供全部数据[替换]  |
| DELETE |           删除资源            |
| PATCH  | 修改资源/提供修改的数据[更新] |

5. 过滤条件

   如果数据量很大的情况,可以传递搜索/过滤的条件

```
 ?limit=10：指定返回记录的数量
 ?offset=10：指定返回记录的开始位置
 ?page=2&per_page=100：指定第几页,以及每页的记录数
 ?sortby=name&order=asc：指定返回结果按照哪个属性排序,以及排序顺序
 ?animal_type_id=1：指定筛选条件
```

6. 状态码

   针对不同的操作以及结果返回不同的状态码

| 状态码 |        意义        |
| :----: | :----------------: |
|  200   |    成功返回资源    |
|  201   |    成功新建资源    |
|  202   |  请求进入后台队列  |
|  204   |    成功删除资源    |
|  400   |      错误请求      |
|  401   |       无权限       |
|  403   | 有权限/访问被禁止  |
|  404   |     找不到资源     |
|  406   |   请求格式不可得   |
|  410   | 请求数据被永久删除 |
|  422   |  创建对象验证错误  |
|  500   |     服务器错误     |

6.  错误处理

```json
 {
     code: 4002,
     msg: "Invalid API key"
 }
```

7.  返回结果

针对不同操作,服务器向用户返回的结果

|            操作             |   意义   |           返回值           |
| :-------------------------: | :------: | :------------------------: |
|       GET /collection       | 获取所有 | 返回资源对象的列表（数组） |
|  GET /collection/resource   | 获取单个 |      返回单个资源对象      |
|      POST /collection       |   新增   |    返回新生成的资源对象    |
|  PUT /collection/resource   |   修改   |     返回完整的资源对象     |
| PATCH /collection/resource  |   修改   |     返回完整的资源对象     |
| DELETE /collection/resource |   删除   |       返回一个空文档       |

```json
 {
     code: 200,
     data: []
 }
```

8. 超媒体

   返回结果中提供链接,连向其他API方法,使得用户不查文档,也知道下一步应该做什么

```python
{
	"link": {
    	"rel":   "collection https://www.example.com/zoos",
   		"href":  "https://api.example.com/zoos",
   		"title": "List of zoos",
   		"type":  "application/vnd.yourformat+json"
 	}
}
```

9. 返回数据格式

   JSON,  Javascript对象表示法,流行的轻量级数据交换格式

```json
 json = {
     "key1": 1,
     "key2": "value2",
     "key3": [1, 2, "value3"],
     "key4": {
         "k1": "v1",
         "k2": [1, 2, 3],
     },
     "key5": null,
     "key6": false
 }
```
