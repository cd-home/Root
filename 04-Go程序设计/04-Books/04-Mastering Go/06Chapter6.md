[TOC]

### 06Chapter6

> 1.  Go package，是Go组织、交付、使用代码的方式
> 2. 函数是基本组成

#### Go packages

> 1. Go所有东西都以packages形式传递
> 2. 组合相关函数、变量
> 3. 所有的packages必须在main package中编译

#### 函数

##### main

> 特殊的main入口函数，无参数无返回值

~~~go
func main() {
    
}
~~~

##### 匿名函数

> 1. 小型功能实现
> 2. 可以赋值变量、作为参数、作为返回值
> 3. 匿名函数也叫做闭包

#### 参数

> 1. 参数传递是值传递
> 2. 值作为参数
> 3. 指针作为参数
> 4. 函数作为参数

#### 返回值

> 1. 返回值
> 2. 返回指针
> 3. 返回函数

##### 多值返回

~~~go
func Foo() (bool, error) {
    
}
func Bar() (b bool, e error) {
    
}
func Goo() (a, b int) {
    
}
~~~