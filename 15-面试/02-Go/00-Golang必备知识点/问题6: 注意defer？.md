[TOC]

### 问题6: 注意defer

#### defer执行顺序

1.  defer的函数会先压栈
2.  延迟到函数执行完成后依次出栈执行

#### defer和return

1.  return之后的会先执行
2.  defer会在函数执行完成后依次出栈执行

#### defer遇见panic

1.  强制defer出栈
2.  panic会传递到defer，如果不处理那么就会一直传递，直到被捕获
3.  defer在panic之后依然有效，可以进行资源回收

~~~go
func f1()  {
	println(1)
}
func f2()  {
    // 捕获异常，如果不捕获那么程序就会崩溃
	if err := recover(); err != nil {
		fmt.Println(err)
	}
	println(2)
}
func f3()  {
	println(3)
}
func main() {
	defer f1()
	defer f2()
	defer f3()
	panic("11111")
}
~~~

#### defer中包含panic

~~~go
func main() {
	defer func() {
        // 只有最后一个panic会被捕获
		if err := recover(); err != nil {
			fmt.Println(err)
		} else {
			fmt.Println("fatal")
		}
	}()
	defer func() {
		panic("defer panic") // defer panic 将main panic覆盖
	}()
	panic("main panic")
}
~~~

#### defer下函数包含子函数

~~~go
func Function(index, value int) int {
	fmt.Println("index = ", index)
	return index
}
func main() {
	defer Function(1, Function(3, 0))
	defer Function(2, Function(4, 0))
}
// 3 4 2 1
~~~

#### defer真题

~~~go
func DeferF1(i int) (t int) {
	t = i
	defer func() {
		t += 3
	}()
	return
}

func DeferF2(i int) int {
	t := i
	defer func() {
		t += 3
	}()
	return t
}

func DeferF3() (t int) {
	defer func(i int) {
		fmt.Println(i)
		fmt.Println(t)
	}(t)
	t = 1
	return 2
}
~~~
