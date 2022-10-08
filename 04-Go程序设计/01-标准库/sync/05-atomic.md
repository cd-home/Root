[TOC]

### atomic

Package atomic provides low-level atomic memory primitives useful for implementing synchronization algorithms.

提供了低级别内存原语, 用于实现同步算法.  适用的范围仅限于"低级别"的同步, 其主要是对变量进行如下操作:

- [ ] add
- [ ] load
- [ ] store
- [ ] compare and swap [CAS]
- [ ] swap

atomic通过CPU在硬件层面实现原子操作无须加锁, 性能较好, 但应用的范围有限. 只用于同步访问整数与指针.

其他采用mutex(代码临界区)或者channle实现同步更为普遍.

#### How To Use

##### Add

》对变量进行增原子操作, delta为负即减操作

~~~go
var num int64 = 0
var wg sync.WaitGroup

func main() {
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func() {
			atomic.AddInt64(&num, 1)
			wg.Done()
		}()
	}
	wg.Wait()
	fmt.Println(num)
}
~~~

##### Load

原子加载,防止读取过程中, 其他的协程修改

~~~go
atomic.LoadInt64(&num)
~~~

##### store

原子设置新值, 无须关注原值是什么

~~~go
atomic.StoreInt64(&num, i)
~~~

##### swap

直接交换值, 无须关注原值是什么, 返回原值

~~~go
atomic.SwapInt64(&num, val)
~~~

##### compare and swap

比较然后交换

~~~go
atomic.CompareAndSwapInt64(&num, oldVal, newVal)
~~~

##### atomic.Value

原子的 Store and Load Any Value

~~~go
func main() {
	config := make(map[string]string)
	config["name"] = "yao"

	var anyValue atomic.Value
	anyValue.Store(config)

	val := anyValue.Load()
	c, ok := val.(map[string]string)
	if ok {
		fmt.Println(c["name"])
	}
}
~~~

#### Ex

原子操作是执行过程中不能被中断的操作. 通常是由CPU原子指令实现; 锁通常是由调度器实现, 用来保护一段代码逻辑;

竞争条件是由于异步的访问共享资源, 并试图同时读写该资源而导致的, 使用互斥锁和通道的思路都是在线程获得到访问权后阻塞其他线程对共享内存的访问, 而使用原子操作解决数据竞争问题则是利用了其不可被打断的特性. 