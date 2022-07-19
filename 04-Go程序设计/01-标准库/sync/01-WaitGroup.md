[TOC]

### WaitGroup

#### How To Use

- [ ] WaitGroup 等待一个 goroutines 完成
- [ ] "main goroutine"  调用 Add 方法设置等待goroutines的数量
- [ ] 每一个goroutine允许结束的时候调用 Done 方法
- [ ] 与此同时, Wait 方法阻塞直到所有的goroutines完成

~~~go
// A WaitGroup waits for a collection of goroutines to finish.
// The main goroutine calls Add to set the number of
// goroutines to wait for. Then each of the goroutines
// runs and calls Done when finished. At the same time,
// Wait can be used to block until all goroutines have finished.
//
// A WaitGroup must not be copied after first use.
type WaitGroup struct {
	noCopy noCopy

	// 64-bit value: high 32 bits are counter, low 32 bits are waiter count.
	// 64-bit atomic operations require 64-bit alignment, but 32-bit
	// compilers do not ensure it. So we allocate 12 bytes and then use
	// the aligned 8 bytes in them as state, and the other 4 as storage
	// for the sema.
	state1 [3]uint32
}
~~~

**注意: 在第一次使用之后, 一个WaitGroup不能被复制**

Example

~~~go
func TestWaitForAllGroutines(t *testing.T) {
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			fmt.Println(i)
			time.Sleep(100 * time.Millisecond)
		}(i)
	}
	wg.Wait()
}
~~~

#### 扩展1: wg-Copy

如上所述, 在第一次使用之后, 一个WaitGroup不能被复制; 

原因是: **变量资源本身带状态且针对的操作是成对出现的不能被复制**

Example

~~~go
func doSomething(wg sync.WaitGroup) {
	defer wg.Done()
	fmt.Println("done")
}

func TestCopyWaitGroup(t *testing.T) {
	var wg sync.WaitGroup
	wg.Add(1)
	doSomething(wg)
	wg.Wait()
}
~~~

wg传递参数被复制一份,导致看似函数中wg.Done()调用, 实际上wg不是同一个, 导致操作Add-Done操作不匹配, 最终导致死锁的出现. 所以在使用WaitGroup时要注意不能被复制. 

可以传递指针解决(指针共享同一个变量)

~~~go
func doSomething(wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Println("done")
}

func TestNoCopyWaitGroup(t *testing.T) {
	var wg sync.WaitGroup
	wg.Add(1)
	doSomething(&wg)
	wg.Wait()
}
~~~

#### 扩展2: noCopy

noCopy 组合在WaitGroup中, 目的就是为了解决"错误复制问题"

~~~go
// noCopy may be embedded into structs which must not be copied
// after the first use.
//
// See https://golang.org/issues/8005#issuecomment-190753527
// for details.
type noCopy struct{}

// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}
~~~

此问题无法在编译期检测出来, 但是 go vet 可以静态检测出带有noCopy的结构

~~~bash
➜  sync git:(master) ✗ go vet wait_group_test.go
# command-line-arguments_test
./wait_group_test.go:23:21: 
	doSomething passes lock by value: sync.WaitGroup contains sync.noCopy
./wait_group_test.go:31:14: 
	call of doSomething copies lock value: sync.WaitGroup contains sync.noCopy
~~~

#### Conclusion

- [x] WaitGroup是用来同步等待一组goroutines完成
- [x] WaitGroup在第一次使用后不可被复制(会导致操作不匹配)
- [x] 如果不能保证上述问题, 请使用 go vet 静态检测

#### 扩展3: Source Code Analysis

TODO