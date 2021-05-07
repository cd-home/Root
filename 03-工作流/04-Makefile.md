[TOC]

### Makefile

> 自动化编译，make命令用来解释Makefile

> 大型需要很多编译以及编译的顺序，提高效率和准确、便捷

#### 规则

> 1. 文件名称Makefile
> 2. 每条规则就明确两件事：构建目标的前置条件是什么，以及如何构建

~~~makefile
<target>: <prerequisites> 
[tab] <commands>
~~~

#### 说明

> 1. target是文件或者伪目标由于本身学习Makefile不用与编译C系列项目，所以target默认都是伪目标
> 2. commands是一行或者多行Linux/Shell命令, command前面是tab (如果是一行想要在多行，可以 \ )

#### 例子

> 注释采用 # 

~~~makefile
# 声明是伪目标，就不会去检测是否有clean这个文件
.PHONY: all
all:
	@echo "Hello World"
~~~

#### 执行

> 1. make target
> 2. 不指定默认执行第一个伪目标
> 3. @会忽略每条命令的打印

~~~bash
make all
~~~

#### 前置条件

> 多个前置条件使用空格分开

~~~makefile
check:
    [check command]
build: check
	[build command]
~~~

#### 通配符

~~~makefile
clean:
	@rm -f *.o
~~~

#### 定义变量

> 变量需要放在 $( ) 之中, 才可以被引用

~~~makefile
VERSION = "1.0.0"
test:
	@echo $(VERSION)
	@echo $$HOME    # shell变量需要$转义

# 其他赋值
Var = Value   # 允许定义后，赋值
Var := Value  # 只允许当前定义赋值
Var ?= Value  # 为空才赋值
Var += Value  # 追加到尾端
~~~

#### 判断

~~~makefile
if eq ($(CC), gcc)
else
endif
~~~

#### 循环

~~~makefile
LIST = one two three
n = 5
test:
	@for (( i = 0; i < $(n); i++ )); do \
	    echo $$i; \
	done

	@for i in $(LIST); do \
		echo $$i; \
	done
~~~

#### 函数

> 执行Shell命令

~~~makefile
TIME = $(shell date +%Y%m%d%H%M)
~~~

#### 注意

> 每command是独立的，如果需要用到前面的comand的变量可以如下

~~~makefile
check: export MODE=DEBUG
check:
	@echo $$MODE
~~~

