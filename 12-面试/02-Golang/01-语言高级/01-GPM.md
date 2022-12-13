[TOC]

### Goroutine调度器原理

#### 调度器的由来？

###### 解析

![Go调度器由来](./images/Go调度器由来.png)

###### 图解：内核线程模型 1:1

<img src="./images/协程.svg" alt="协程" style="zoom:150%;" />

<img src="./images/协程2.svg" alt="协程" style="zoom:150%;" />

<img src="./images/早期调度器.svg" alt="g1" style="zoom:150%;" />

#### 调度器GPM设计思想

##### GPM模型简介

###### 图解：

两级线程模型，即用户调度器实现用户线程到Kernel的“调度”，内核调度器实现KSE到CPU上的调度

<img src="./images/GMP模型.svg" alt="GMP模型" style="zoom: 150%;" />

###### 解析

![GPM模型概念](./images/GPM模型概念.png)

##### 调度器设计策略

###### 解析

![调度器设计策略](./images/调度器设计策略.png)

###### 图解

<img src="./images/GMP复用线程—偷取机制.svg" alt="GMP复用线程—偷取机制" style="zoom:150%;" />

<img src="./images/GMP复用线程—阻塞转移.svg" alt="GMP复用线程—阻塞转移" style="zoom:150%;" />

##### go gunc()过程

###### 解析

<img src="./images/go func 流程.png" alt="go func 流程" style="zoom:150%;" />

###### 图解

![GMP go func](./images/GMP go func.svg)

##### 调度器的生命周期

###### 概念

![调度器的生命周期](./images/调度器的生命周期.png)

###### 图解

![GMP 生命周期](./images/GMP 生命周期.svg)

##### 可视化GPM编程

###### trace

~~~go
package main
import (
	"fmt"
	"os"
	"runtime/trace"
)
func main() {
	// 1 创建文件
	f, err := os.Create("trace.out")  // go run 生成trace.out文件
	if err != nil {
		return
	}
	defer f.Close()
	// 2 启动
	trace.Start(f)
	// 3 业务
	fmt.Println("hello world")
	// 4 停止
	trace.Stop()
}
~~~

~~~bash
go tool trace trace.out
~~~

###### GODEBUG

~~~bash
GODEBUG=schedtrace=1000 ./trace
~~~

##### GMP场景全面分析

###### 场景1: 局部性

<img src="./images/GMP 全面分析-局部性.svg" alt="GMP 全面分析-局部性" style="zoom:150%;" />

###### 场景2: 调度切换

<img src="./images/GMP 全面分析-调度切换.svg" alt="GMP 全面分析-调度切换" style="zoom:150%;" />

###### 场景345: 队列满

<img src="./images/GMP 全面分析-队列满.svg" alt="GMP 全面分析-队列满" style="zoom:150%;" />

###### 场景6: 唤醒M

**自旋线程是要去获取G的**

<img src="./images/GMP 全面分析-唤醒M.svg" alt="GMP 全面分析-唤醒M" style="zoom:150%;" />

###### 场景7: 唤醒的M取全局G队列

<img src="./images/GMP 全面分析-唤醒M获取全局G.svg" alt="GMP 全面分析-唤醒M获取全局G" style="zoom:150%;" />

###### 场景8: 唤醒的M偷取其他P的G

<img src="./images/GMP 全面分析-唤醒M偷取其他P的G.svg" alt="GMP 全面分析-唤醒M偷取其他P的G" style="zoom:150%;" />

###### 场景9: 唤醒的M最大自旋线程数

<img src="./images/GMP 全面分析-自旋线程最大限制.svg" alt="GMP 全面分析-唤醒M偷取其他P的G" style="zoom:150%;" /> 

###### 场景10: G阻塞调用

<img src="./images/GMP 全面分析-G阻塞调用.svg" alt="GMP 全面分析-G阻塞调用" style="zoom:150%;" />

###### 场景11: 阻塞调用结束

<img src="./images/GMP 全面分析-G阻塞取消后.svg" alt="GMP 全面分析-G阻塞取消后" style="zoom:150%;" />
