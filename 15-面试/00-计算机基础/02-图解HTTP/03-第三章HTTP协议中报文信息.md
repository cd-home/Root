[TOC]

# 第三章HTTP协议中报文信息

## 3.1 HTTP报文

1.  请求报文
2.  响应报文
3.  一般又首部 + 主体组成，中间空行隔开【回车+换行】
4.  主体不是必须的



## 3.2 请求报文及响应报文结构

### 请求报文

1.  报文首部
    *   请求行
        *   请求方法 请求URL HTTP版本
    *   请求首部字段
    *   通用首部字段
    *   实体首部字段
    *   其他：Cookie等
2.  空行
3.  报文主体

### 响应报文

1.  报文首部
    *   响应行/状态行
        *   HTTP版本 响应状态码 响应短语
    *   响应首部字段
    *   通用首部字段
    *   实体首部字段
    *   其他：Cookie等
2.  空行
3.  报文主体



## 3.3 编码提升传输效率

1.  报文

    HTTP通信基本单位，8字节流传输

2.  实体

    请求或者响应有效载荷数据 实体首部+实体主体

3.  报文主体 == 实体主体

4.  压缩

    *   gzip
    *   compress
    *   deflate
    *   indenttity

5.  分块传输编码Chunked Transfer Coding



## 3.4 多种数据多部分对象集合

1.  **multipart/form-data**
2.  **multipart/byteranges**
3.  **multipart/form-data**
4.  **multipart/byteranges**

在 HTTP 报文中使用多部分对象集合时，需要在首部字段里加上 Content-type



## 3.5 获取部分内容的范围请求2

1.  断点续传，可恢复下载机制
2.  形式如下

~~~
Range: bytes=5001-10000
Range: bytes=5001-
Range: bytes=-3000, 5000-7000
~~~

3.  返回206状态码



## 3.6 内容协商返回最合适的内容

1.  内容协商机制是指客户端和服务器端就响应的资源内容进行交涉，然后提供给客户端最为适合的资源
2.  内容协商会以响应资源的语言、字符集、编码方式等作为判断的基准
3.  如下可选项
    *   **Accept**
    *   **Accept-Charset**
    *   **Accept-Encoding**
    *   **Accept-Language**
    *   **Content-Language**