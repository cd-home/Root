[TOC]

### 问题7: Golang如何进行模块与项目依赖管理？【Go Mod】

#### Go Moudules

1.  Go语言依赖解决方案
2.  Go1.11版本发布，Go1.14正式推荐在生产环境使用
3.  淘汰GOPATH模式

#### GOPATH模式

##### 模式

固定三个目录的工作空间

1.  src，程序源码
2.  bin，可执行文件
3.  pkg，下载的包

~~~
GOPATH="/Users/apple-liyao/go/WorkSpace"
~~~

##### 缺点

1.  没有版本管理的概念
2.  无法制定当前项目引用的第三方的版本号
3.  无法同步一致的第三方版本号

#### Go Modules模式

1.  go mod环境变量

~~~
GO111MODULE=on // auto on off 
GOPROXY="https://goproxy.cn,direct"           // 七牛
GOPROXY="https://mirrors.aliyun.com/goproxy"  // 阿里
~~~

2.  设置

~~~bash
go env -w GO111MODULE=on
go env -w GOPROXY="https://goproxy.cn,direct"  # direct 回到模块源地址去抓取
go env -w GOPRIVATE="*.example.com"  # 配置私有的仓库，就不会通过GOPROXY下载
~~~

3.  go mod 命令

|      命令       |             作用             |
| :-------------: | :--------------------------: |
|   go mod init   |        生成go.mod文件        |
| go mod download |      下载go.mod中的依赖      |
|   go mod tidy   |        整理现有的依赖        |
|  go mod graph   |         查看依赖结构         |
|   go mod edit   |        编辑go.mod文件        |
|  go mod vender  | 导出项目所有依赖到vender目录 |
|  go mod verify  |    检查一个模块是否被篡改    |
|   go mod why    |    查看为什么依赖某个模块    |

#### 使用Go Modules

1.  设置环境变量
2.  初始化项目

~~~bash
go mod init github.com/liyao/learn_go
~~~

3.  mod信息

~~~
module gin-lover

go 1.13

require (
	github.com/dgrijalva/jwt-go v3.2.0+incompatible
	github.com/gin-gonic/gin v1.4.0
	github.com/go-sql-driver/mysql v1.4.1
	github.com/jinzhu/gorm v1.9.11
	github.com/kr/pretty v0.1.0 // indirect  间接依赖kr，本项目没有直接使用kr
	gopkg.in/go-playground/assert.v1 v1.2.1
)
~~~

4.  sum文件，保障项目的模块不会被篡改

#### 修改项目模块的版本依赖关系

1.  go mod edit

~~~bash
go mod edit -replace=github.com/gin-gonic/gin@v1.4.0=github.com/gin-gonic/gin@v1.5.0
~~~

2.  replace

~~~
replace github.com/gin-gonic/gin v1.4.0 => github.com/gin-gonic/gin v1.5.0
~~~