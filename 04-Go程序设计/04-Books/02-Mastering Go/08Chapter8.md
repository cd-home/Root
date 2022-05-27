[TOC]

### 08Chapter8

#### Go unix系统编程

##### Unix进程

1. 进程是一个执行环境[用户数据、系统数据、运行中获取的资源]
2. 程序是一个二进制文件[包含指令与初始化指令时用到的数据、进程的用户数据]
3. 用户进程、守护进程、系统进程
4. C通过fork、Go通过goroutines

##### Flag包

> 传递命令行参数进入程序，控制行为

~~~go
func main() {
	var manyNames  NamesFlag
	minusk := flag.Int("k", 0, "An int")
	minusO := flag.String("o", "nike", "name")
	flag.Var(&manyNames, "names", "names list")

	flag.Parse()
	fmt.Println("-k", *minusk)
	fmt.Println("-o", *minusO)

	for i, item := range manyNames.GetNames() {
		fmt.Println(i ,item)
	}
	// Args returns the non-flag command-line arguments.
	for index, val := range flag.Args() {
		fmt.Println(index, val)
	}

}
~~~

##### io.Reader

###### 缓冲

> 1. 缓冲与无缓冲文件输入和输出
> 2. 关键数据使用无缓冲；缓冲可能存在脏数据或者断电数据丢失

###### bufio

> 内部任然使用io.Reader和i o.Writer对象，非常适合读区文件



