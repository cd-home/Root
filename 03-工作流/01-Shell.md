[TOC]

### 快速入门Shell

#### 注释

~~~shell
#!/usr/bin/env bash

# 单行注释
:<<EOF

EOF
~~~

#### echo

~~~shell
#!/bin/bash
echo "Hello Wolrd" > Hello.txt

printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234 
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543 
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876 
~~~

#### Hello World

~~~shell
#!/usr/bin/env bash
echo "Hello World"
~~~

\#!用来声明当前脚本用什么解释器执行

**说明**

1.  脚本必须有执行权限

~~~bash
chmod +x 00.sh
~~~

2.  执行方式

~~~bash
./00.sh
/bin/bash 00.sh
~~~

#### 变量

~~~bash
#!/usr/bin/env bash
name="li yao"
echo ${name}

age=25
# 设置只读变量
readonly age
# age=23
echo ${age}
~~~

说明

1.  定义变量

~~~
命名只能使用英文字母，数字和下划线，首个字符不能以数字开头
中间不能有空格，可以使用下划线（_
不能使用标点符号。
不能使用bash里的关键字（可用help命令查看保留关键字）
~~~

2.  使用变量

~~~bash
${name}
~~~

3.  变量：局部变量、环境变量、Shell变量

#### 字符串

~~~shell
#!/usr/bin/env bash
name="li yao"
str1="hello"
# 拼接
echo "${str1}, ${name}"
# 长度
echo ${#name}
# 子字符串
echo ${name:2:4}
~~~

#### 数组

~~~shell
#!/bin/bash

array=(1 2 3 "God Yao")
# 获取元素
echo ${array[1]}
# 长度
echo ${#array[@]}
echo ${#array[*]}
#  获取元素长度
echo ${#array[3]}
~~~

#### 命令行参数

~~~shell
#!/bin/bash
# 获取单个命令行参数
echo $1
echo $2

# 参数个数
echo $# 

# 获取所有参数, 可以遍历
echo $@

# 获取所有的命令行参数，作为一个字符串显示
echo $*

# 当前进程id
echo $$

# 退出状态
echo $?
~~~

~~~bash
./04.sh li yao 25
~~~

#### 算术运算符

>   原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用

~~~shell
#!/bin/bash

value=`expr 10 + 2`
echo ${value}

value=`expr 20 - 3`
echo ${value}

value=`expr 20 \* 4`
echo ${value}

value=`expr 30 / 5`
echo ${value}

value=`expr 10 % 4`
echo ${value}

other=${value}

# 判断
if [[ ${value} == ${other} ]]
then
    echo "=="
fi

if [[ ${value} != ${other} ]]
then
    echo "!="
fi
~~~

#### 关系运算符

>   只支持数字，如果字符串是数字组成，也可以

~~~shell
#!/bin/bash

a=10
b=20
# ==
if [[ ${a} -eq ${b} ]]
then
    echo "a==b"
else
    echo "a!=b"
fi

# !=
if [[ ${a} -ne ${b} ]]
then
    echo "a != b"
else
    echo "a==b"
fi

# >
if [[ ${a} -gt ${b} ]]
then
    echo "a > b"
else
    echo "a !> b"
fi

# <
if [[ ${a} -lt ${b} ]]
then
    echo "a < b"
else
    echo "a !< b"
fi

# <=
if [[ ${a} -ge ${b} ]]
then
    echo "a >= b"
else
    echo "a < b"
fi

# <=
if [[ ${a} -le ${b} ]]
then
   echo "a <= b"
else
    echo "a > b"
fi
~~~

#### 布尔运算符

>   常规的布尔运算符有：!、&&、||

~~~shell
#!/bin/bash

num=20

if [[ ${num} -lt 30 && ${num} -gt 10 ]]
then
    echo "10 < num < 30"
fi
~~~

#### 字符串运算

~~~shell
str1="abc"
str2="abd"

if [[ ${str1} = ${str2} ]]
then
    echo "str1 == str2"
elif [[ ${str1} != ${str2} ]]
then
    echo "str1 != str2"
fi

if [[ -z ${str1} ]]
then
    echo "字符串长度为0"
else
    echo "字符串长度不为0"
fi

if [[ -n ${str2} ]]
then
    echo "字符串长度不为0"
fi

if [[ ${str1} ]]
then
    echo "字符串不为空"
fi
~~~

#### 文件测试

| 操作符  |                             说明                             |
| :-----: | :----------------------------------------------------------: |
| -d file |           检测文件是否是目录，如果是，则返回 true            |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true |
| -r file |            检测文件是否可读，如果是，则返回 true             |
| -w file |            检测文件是否可写，如果是，则返回 true             |
| -x file |           检测文件是否可执行，如果是，则返回 true            |
| -s file |    检测文件是否为空（文件大小是否大于0），不为空返回 true    |
| -e file |      检测文件（包括目录）是否存在，如果是，则返回 true       |

~~~shell
#!/bin/bash
file="/Users/apple-liyao/Desktop/GodExample/shell/00.sh"
if [[ -x ${file} ]]
then
    echo "可执行"
fi
~~~

#### 流程控制

##### if

~~~shell
#!/bin/bash

a=10
b=20

if [[ ${a} == ${b} ]]
then
    echo "a == b"
else
    echo "a != b"
fi
# test 命令可以代替 [[ ]]
if test ${a} -lt 5
then
    echo "a > 5"
elif [[ ${b} -lt 30 ]]
then
    echo "b < 30"
fi
~~~

##### for

~~~shell
array=(1 2 3 4 5 6 7)
for item in ${array[*]} ;
do
    echo ${item}
done

for (( ; ; ))
~~~

##### while

~~~shell
time=0
while test ${time} -lt 10
do
    echo ${time}
    time=`expr ${time} + 1`
done

time=0
while(( $time < 5 ))
do
    echo ${time}
    # 执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量
    let "time++"
done

time=0
until test ! ${time} -lt 10
do
    echo ${time}
    time=`expr ${time} + 1`
done

# 死循环
while :
do
done
while true; 
do
done
~~~

##### case

~~~shell
#!/bin/bash
echo "键盘输入数字1-4"
read num
case ${num} in
    1) echo "1";;
    2) echo "2";;
    3) echo "3";;
    4) echo "4";;
esac

case ${num} in
    1|2|3|4|5|6|7|8|9) echo ${num};;
    *) echo "over"
        break
    ;;
esac
~~~

#### Shell函数

~~~shell
Foo() {
    echo $1
    echo $2
    echo $3
    echo $4
    echo "function"
    for item in $@ ; do
        echo ${item}
    done
}
# 调用以及传递参数
Foo $@
~~~

#### 输出\重定向

~~~shell
echo "Hello World" > 1.txt
echo "Hello World追加" >> 1.txt
~~~

#### 引用外部Shell

~~~shell
# 用 . 引用外部sh文件
. ./09.sh
~~~
