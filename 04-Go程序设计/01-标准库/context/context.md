[TOC]

### context

#### What？

1. g一旦开启后, 除了本身的逻辑执行外, 外部无法干涉
    - 在Go中无法直接杀死g
    - g的关闭一般采用select + channle的方式
2. 同样的无法传递信息给g, 或者取消
3. g中还可能开辟新的g, 也面临同样的问题
4. 新增context包, 作为g的上下文, 传递进g中, 追踪g
    - 取消控制
    - 超时控制
    - 传值

![context](./images/context.svg)

#### Why?

##### Context

~~~go
type Context interface {
	Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
~~~

1. Done返回一个只读的channel, 所有关闭的效果都是通过这个只读的channle
2. 当关闭这个只读的channle时, ctx.Done就会读出来strcut{}

##### canceler

```go
 type canceler interface {
     cancel(removeFromParent bool, err error)
     Done() <-chan struct{}
 }
```

1.  实现了这个接口的context就认为该ctx是可以取消的
2.  没有在Context定义的原因是, 取消的操作并不是一定需要实现的
3.  取消ctx, 与之相关的ctx也需要取消, 所有的都同时监控这个Done返回的只读channle, 一旦关闭就会广播出去, 然后所有的ctx都会收到

##### emptyCtx

~~~go
type emptyCtx int
func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}
func (*emptyCtx) Done() <-chan struct{} {
	return nil
}
func (*emptyCtx) Err() error {
	return nil
}
func (*emptyCtx) Value(key interface{}) interface{} {
	return nil
}
~~~

1.  没有具体实现, 是一个空的context, 没有值, 没有超时, 没有取消
2.  用它来包装两个根Ctx

~~~go
var (
	background = new(emptyCtx)
	todo       = new(emptyCtx)
)
// Background returns a non-nil, empty Context. It is never canceled, has no
// values, and has no deadline. It is typically used by the main function,
func Background() Context {
	return background
}
// TODO returns a non-nil, empty Context. Code should use context.TODO when
// it's unclear which Context to use or it is not yet available 
func TODO() Context {
	return todo
}
~~~

##### cancelCtx

~~~go
type cancelCtx struct {
	Context
    // 保护字段
	mu       sync.Mutex   			// protects following fields
	done     chan struct{} 			// created lazily, closed by first cancel call
	children map[canceler]struct{}  // set to nil by the first cancel call
	err      error            		// set to non-nil by the first cancel call
} 
~~~

​	是一个取消的Ctx, 实现了canceler接口

1.  Done方法实现

~~~go
func (c *cancelCtx) Done() <-chan struct{} { 
    // 只读的channle, 配合select使用
	c.mu.Lock()
    // 创建cancelCtx的时候, done一开始就是nil的, 用到的时候才会创建
	if c.done == nil {
		c.done = make(chan struct{})
	}
	d := c.done
	c.mu.Unlock()
	return d
}
~~~

2.  cancel方法实现
    *   取消方法的目的就是关闭done, 达到广播效果, select可以监听
    *   关闭它所有的子节点
    *   从父节点删除自己

~~~go
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
    // 必须要传一个取消的错误类型进来
	if err == nil {
		panic("context: internal error: missing cancel error")
	}
	c.mu.Lock()
	if c.err != nil {
		c.mu.Unlock()
		return // already canceled
	}
	c.err = err
    // 关闭 done channle
	if c.done == nil {
		c.done = closedchan
	} else {
		close(c.done)
	}
    // 遍历它所有的子节点, 调用取消方法
	for child := range c.children {
		child.cancel(false, err)
	}
    // 子节点置il
	c.children = nil
	c.mu.Unlock()
	// 从父节点移除自己
	if removeFromParent {
		removeChild(c.Context, c)
	}
}
// removeChild removes a context from its parent.
func removeChild(parent Context, child canceler) {
	p, ok := parentCancelCtx(parent)
	if !ok {
		return
	}
	p.mu.Lock()
	if p.children != nil {
		delete(p.children, child)
	}
	p.mu.Unlock()
}
~~~

3. WithCancel

    传入一个根Ctx, 一般是BackgroundCtx

~~~go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
    // 返回子ctx, 和取消函数, true代表从父节点移除, Canceled是错误类型
	return &c, func() { c.cancel(true, Canceled) }
}
~~~

4.  创建一个可取消的子Ctx

~~~go
func newCancelCtx(parent Context) cancelCtx {
    // 其他字段done children都是新建的
	return cancelCtx{Context: parent}
}
~~~

5.  挂载在parant上

~~~go
func propagateCancel(parent Context, child canceler) {
	if parent.Done() == nil {
		return // parent is never canceled
	}
    // 必须判断parent的类型 *cancelCtx *timerCtx *valueCtx,  找到可取消的parentCtx
	if p, ok := parentCancelCtx(parent); ok {
		p.mu.Lock()
		if p.err != nil {  // parent has already been canceled
			child.cancel(false, p.err)
		} else {
            // 第一次为nil
			if p.children == nil {
				p.children = make(map[canceler]struct{})
			}
            // 添加到children
			p.children[child] = struct{}{}
		}
		p.mu.Unlock()
	} else {
        // 如果没有找到可取消的parentCtx, 就监控父节点和子节点的取消信号
		go func() {
			select {
			case <-parent.Done():
				child.cancel(false, parent.Err())
			case <-child.Done():
			}
		}()
	}
}
~~~

6.  断言parent的类型

~~~go
func parentCancelCtx(parent Context) (*cancelCtx, bool) {
	for {
		switch c := parent.(type) {
		case *cancelCtx:
			return c, true
		case *timerCtx:
			return &c.cancelCtx, true
		case *valueCtx:
			parent = c.Context
		default:
			return nil, false
		}
	}
}
~~~

##### timerCtx

~~~go
type timerCtx struct {
	cancelCtx         // 组合取消
	timer *time.Timer // Under cancelCtx.mu.
	deadline time.Time
}
~~~

1.  withTimeout

~~~go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
~~~

2.  WithDeadline

~~~go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
	if cur, ok := parent.Deadline(); ok && cur.Before(d) {
		// The current deadline is already sooner than the new one.
		return WithCancel(parent)
	}
    // timeCtx对象
	c := &timerCtx{
		cancelCtx: newCancelCtx(parent),
		deadline:  d,
	}
    // 挂载
	propagateCancel(parent, c)
	dur := time.Until(d)
    // 已经超时
	if dur <= 0 {
		c.cancel(true, DeadlineExceeded) // deadline has already passed
		return c, func() { c.cancel(false, Canceled) }
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	if c.err == nil {
        // 定时启动
		c.timer = time.AfterFunc(dur, func() {
			c.cancel(true, DeadlineExceeded)
		})
	}
	return c, func() { c.cancel(true, Canceled) }
}
~~~

time包

~~~go
// 定时启动
func AfterFunc(d Duration, f func()) *Timer {
	t := &Timer{
		r: runtimeTimer{
			when: when(d),
			f:    goFunc,
			arg:  f,
		},
	}
	startTimer(&t.r)
	return t
}// 开辟一个g去启动超时
func goFunc(arg interface{}, seq uintptr) {
	go arg.(func())()
}
~~~

3.  cancel实现

~~~go
func (c *timerCtx) cancel(removeFromParent bool, err error) {
    // 取消
	c.cancelCtx.cancel(false, err)
	if removeFromParent {
		removeChild(c.cancelCtx.Context, c)
	}
	c.mu.Lock()
    // 时间没有到之前停止
	if c.timer != nil {
		c.timer.Stop()
		c.timer = nil
	}
	c.mu.Unlock()
}
~~~

##### valueCtx

~~~go
type valueCtx struct {
	Context
	key, val interface{}
}
~~~

1.  withValue

~~~go
func WithValue(parent Context, key, val interface{}) Context {
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}
~~~

2.  Value

~~~go
func (c *valueCtx) Value(key interface{}) interface{} {
    //  递归去获取然后比较, 所以key必须要求可比较
	if c.key == key {
		return c.val
	}
	return c.Context.Value(key)
}
~~~

#### How?

1.  使用

~~~go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) 
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, val interface{}) Context 
~~~

2.  第一个参数

~~~go
func Background() Context
~~~

3.  建议

*   context应该作为函数的第一个参数, 一般命名为ctx
*   不要传入nil的context
*   context中应该是一些共享数据
*   context是并发安全的

Example

websocket每隔一秒后端向前端发送数据

~~~go
func WorkerLoaction(ctx context.Context) {
    for {
        SendData()
        select {
        case <-ctx.Done():
            return
        case <-time.After(time.Second * 1):
        }
    }
}
~~~

2.  合适的时机可以取消

~~~go
ctx, cancel := context.WithTimeout(context.Background(), time.Hour * 1)
go WorkerLocation(ctx)
// later
cancel()
~~~

3.  例子2

~~~go
package main
import (
	"context"
	"fmt"
	"time"
)
func DoWorkTimeOut(ctx context.Context) {
	ch := make(chan struct{}, 1)
    // 有问题, 这个还是会执行完成
	go func() {
		// 模拟耗时任务
		time.Sleep(time.Second * 4)
    // 回阻塞住, 所以ch cache-1
		ch <- struct{}{}
	}()
	select {
	// 监听超时, 超时后会写入通道
	case <-ctx.Done():
		fmt.Println("timeout")
		return
	case <-ch:
		fmt.Println("work done")
	}
}
func main() {
	ctx, cancle := context.WithTimeout(context.Background(), time.Second * 3)
	defer cancle()  // 万无一失

	go DoWorkTimeOut(ctx)

	time.Sleep(time.Second * 10)
	fmt.Println("main finish")
}
~~~