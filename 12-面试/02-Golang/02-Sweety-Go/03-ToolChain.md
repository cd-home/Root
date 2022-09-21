[TOC]

### Toolchain(工具链)

#### Gcflags

>   考点：逃逸分析

~~~go
apple-liyao@MacBook-Air Go-Interview % go build -gcflags "-m -l -N" 20-http.go
# command-line-arguments
./20-http.go:16:12: leaking param: w
./20-http.go:16:35: leaking param content: r
./20-http.go:17:13: Hello ... argument does not escape
./20-http.go:17:19: r.URL.Host escapes to heap
./20-http.go:18:13: Hello ... argument does not escape
./20-http.go:18:13: w escapes to heap
~~~

#### Benchmark

>   考点：基准测试

~~~go
func Fib(n int) int {
	if n == 1 {
		return 0
	}
	if n == 2 {
		return 1
	}
	return Fib(n-1) + Fib(n-2)
}
func BenchmarkFib10(b *testing.B) {
	for n := 0; n < b.N; n++ {
		Fib(10)
	}
}
~~~

#### package management

>   考点：包管理

1.  vender
2.  GOPATH
3.  Go MOD

#### GOMAXPROCS

>   考点：设置P的数量; 执行核心的数量；

~~~go
func main() {
	const GOMAXPROCS = 1
	runtime.GOMAXPROCS(GOMAXPROCS)

	var wg sync.WaitGroup
	wg.Add(2)

	go func() {
		defer wg.Done()
		for char := 0; char < 26; char++ {
			fmt.Printf("%c", 'A' + char)
		}
	}()

	go func() {
		defer wg.Done()
		for char := 0; char < 26; char++ {
			fmt.Printf("%c", 'a' + char)
		}
	}()

	wg.Wait()
}
~~~

#### Shared object

>   考点：打包源码包为插件使用

~~~go
// str.go
package main
import "strings"
func UpperCase(s string) string {
	return strings.ToUpper(s)
}

// go build -buildmode=plugin -o str.so str.go
// main
import (
	"fmt"
	"log"
	"plugin"
)
func main() {
	p, err := plugin.Open("str.so")
	if err != nil {
		log.Panicf("plugin.Open: %s\n", err)
	}
	f, err := p.Lookup("UpperCase")
	if err != nil {
		log.Panicf("Lookup UpperCase: %s\n", err)
	}
	UpperCase, ok := f.(func(string) string)
	if !ok {
		log.Panicf("UpperCase assertion: %s\n", err)
	}
	s := UpperCase("hello")
	fmt.Println(s)
}
~~~

#### GODEBUG

>   考点：开启GODEBUG模式

~~~bash
GODEBUG='gctrace=1' ./test
~~~

#### GOROOT GOPATH

>   考点：GOROOT、GOAPTH

#### Pprof

>   考点：http server的pprof

~~~go
// http://127.0.0.1:8080/debug/pprof
import (
	"net/http"
	_ "net/http/pprof"
)
func main() {
    // 然后点击页面中的profile，下载生成
    log.Fatal(http.ListenAndServe(":8080", nil))
}
~~~

~~~bash
Add `import _ "net/http/pprof"` in the main.go
Run the following commands to get info correspondingly:
CPU profile:
go tool pprof http://localhost:6060/debug/pprof/profile --second N
heap profile:
go tool pprof http://localhost:6060/debug/pprof/heap
goroutine blocking profile:
go tool pprof http://localhost:6060/debug/pprof/block
~~~

