#### defer

注册多个延迟调用，并且多个调用的顺序是FIFO，常用于一些资源的回收以及释放

1.  打开文件/关闭文件
2.  加锁/释放锁
3.  打开连接/关闭连接

概念

1.  延迟函数的在主函数结束的时候调用
2.  多个defer会压栈，先进后出执行

情况1: 延迟函数的参数

1.  延迟函数的参数在遇见defer的时候就已经确定

  2.  意思是遇见defer的延迟调用函数，会保存当前延迟函数状态

~~~go
func DeferParam() {
	i := 10
	defer fmt.Println(i)  // 10
	i++
}
~~~

情况2: 多个defer压栈执行

~~~go
func Defers() {
    // 3 2 1
	defer fmt.Println(1)
	defer fmt.Println(2)
	defer fmt.Println(3)
}
~~~

情况3: defer遇见panic

1.  遇到panic后，后面不在执行
2.  然后会执行已经注册的全部defer，并且将异常往回传递，直到异常被捕获

~~~go
package main
func f1() {
	defer println("f1-begin")
	f2()
	defer println("f1-end")
}
func f2() {
	defer println("f2-begin")
	f3()
	defer println("f2-end")
}
func f3() {
	defer println("f3-begin")
	panic(0)   
	defer println("f3-end")
}
func main() {
	f1()
}
~~~

3.  如果defer中也有panic，那么defer中的panic会覆盖main中的Panic

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

情况4: 异常捕获

~~~go
func RecoverPanic() {
	if err := recover(); err != nil {
		fmt.Println(err)
	}
}
func main() {
	defer RecoverPanic()
	panic("recover panic")
}
~~~

情况5: defer影响有名函数的返回值

1.  修改返回值
    *   return的操作不是原子操作，只是代理汇编的ret操作，即将跳转程序执行
    *   先将值压栈作为返回值
    *   然后执行跳转【defer的执行时机正是在跳转之前，所有有机会操作有名的返回值】

~~~go
func foo() (ret int) {
    defer func() {
        ret++
    }()
    return 0
}
// ret = 1
// ret = 0
// ret++
// return
~~~

2.  无法操作匿名的返回值

~~~go
func foo() int { //  还是返回1
    var i int
    defer func() {
        i++
    }()
    return 1
}
func foo() int { //  还是返回1
    var i int
    defer func() {
        i++
    }()
    return i
}
// retValue = i
// i++
// return retValue
~~~

3.  测试

~~~go
func DeferReturn() (t int) { //1: 初始值为0  3: 被修改为2
	defer func(i int) {
		fmt.Println(i)
		fmt.Println(t)  // t = 2
    }(t) // 2: 出现defer的时候参数就已经确定 t = 0 然后 i = 0
	t = 1
	return 2  // t = 2
}
~~~
