[TOC]

### Layout Example

》常见的目录说明

#### `/cmd`

项目的应用入口

每个应用程序的目录名应该与你想要的可执行文件的名称相匹配(例如, `/cmd/myapp`)

通常有一个小的 `main` 函数, 从 `/internal` 和 `/pkg` 目录导入和调用代码, 除此之外没有别的东西

1. 不要在这个目录中放置太多代码
2. 如果你认为代码可以导入并在其他项目中使用, 那么它应该位于 `/pkg` 目录中
3. 如果代码不是可重用的, 或者你不希望其他人重用它, 请将该代码放到 `/internal` 目录中

#### `/internal`

私有应用程序和库代码

1. 这是不希望其他人在其应用程序或库中导入代码
2. 注意: 这个布局模式是由 Go 编译器本身执行的有关更多细节, 请参阅Go 1.4 [`release notes`](https://golang.org/doc/go1.4#internalpackages) 
3. 注意: 你并不局限于顶级 `internal` 目录, 在项目树的任何级别上都可以有多个`internal`目录

你可以选择向 internal 包中添加一些额外的结构, 以分隔共享和非共享的内部代码这不是必需的(特别是对于较小的项目)

你的实际应用程序代码可以放在 `/internal/app` 目录下(例如 `/internal/app/myapp`), 这些应用程序共享的代码可以放在 `/internal/pkg` 目录下(例如 `/internal/pkg/myprivlib`)

#### `/pkg`

外部应用程序可以使用的库代码

例如 `/pkg/mypubliclib`)其他项目会导入这些库, 所以在这里放东西之前要三思)

如果你的应用程序项目真的很小, 并且额外的嵌套并不能增加多少价值(除非你真的想要), 那就不要使用它当它变得足够大时, 你的根目录会变得非常繁琐时(尤其是有很多非 Go 应用组件时), 请考虑一下

#### `/vendor`

应用程序依赖项(手动管理或使用你喜欢的依赖项管理工具, 如新的内置 [`Go Modules`](https://github.com/golang/go/wiki/Modules) 功能)

`go mod vendor` 命令将为你创建 `/vendor` 目录请注意, 如果未使用默认情况下处于启用状态的 Go 1.14, 则可能需要在 `go build` 命令中添加 `-mod=vendor` 标志

如果你正在构建一个库, 那么不要提交你的应用程序依赖项

国内模块代理功能默认是被墙的, 七牛云有维护专门的的[`模块代理`](https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md) 

#### `/api`

OpenAPI/Swagger 规范, JSON 模式文件, 协议定义文件

#### `/web`

特定于 Web 应用程序的组件:静态 Web 资产、服务器端模板和 SPAs

#### `/configs`

配置文件模板或默认配置

#### `/init`

System init（systemd, upstart,sysv）和 process manager/supervisor（runit, supervisor）配置

#### `/scripts`

执行各种构建、安装、分析等操作的脚本; 这些脚本保持了根级别的 Makefile 变得小而简单

#### `/build`

打包和持续集成

1. 容器( Docker )、操作系统(deb、rpm、pkg)包配置和脚本放在 `/build/package` 目录下
2. 将你的 CI配置和脚本放在 `/build/ci` 目录中请注意

#### `/deployments`

1. IaaS、PaaS、系统和容器编配部署配置和模板(docker-compose、kubernetes/helm)
2. 注意, 在一些存储库中(特别是使用 kubernetes 部署的应用程序), 这个目录被称为 `/deploy`

#### `/test`

额外的外部测试应用程序和测试数据

你可以随时根据需求构造 `/test` 目录对于较大的项目, 有一个数据子目录是有意义的例如, 你可以使用 `/test/data` 或 `/test/testdata` (如果你需要忽略目录中的内容)请注意, Go 还会忽略以“.”或“_”开头的目录或文件, 因此在如何命名测试数据目录方面有更大的灵活性

#### `/docs`

设计和用户文档(除了 godoc 生成的文档之外)

#### `/tools`

这个项目的支持工具注意, 这些工具可以从 `/pkg` 和 `/internal` 目录导入代码

#### `/examples`

你的应用程序和/或公共库的示例

#### `/third_party`

外部辅助工具, 分叉代码和其他第三方工具(例如 Swagger UI)

#### `/assets`

与存储库一起使用的其他资产(图像、徽标等)
