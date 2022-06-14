### Alpine Linux

> Alpine Linux是一个面向安全应用的轻量级Linux发行版

#### APK(包管理工具)

Example(Dockerfile)

~~~dockerfile
RUN apk update && \
	apk add --no-cache \
	vim && \
	curl && \
	ca-certificates && \
	bash && \
	tzdata && \
	ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
	echo Asia/Shanghai > /etc/timezone
~~~

#### 扩展

##### tzdata

> Time Zone Database发布的组件之一,
>
> 全称time zone and daylight-saving time(DST) data，供各个Linux系统安装以读取Time Zone Database中数据