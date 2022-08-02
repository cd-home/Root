### Language Mechanics On Memory Profiling

### Prelude

This is the third post in a four part series that will provide an understanding of the mechanics and design behind pointers, stacks, heaps, escape analysis and value/pointer semantics in Go. This post focuses on heaps and escape analysis.

Index of the four part series:
\1) [Language Mechanics On Stacks And Pointers](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html)
\2) [Language Mechanics On Escape Analysis](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html)
\3) [Language Mechanics On Memory Profiling](https://www.ardanlabs.com/blog/2017/06/language-mechanics-on-memory-profiling.html)
\4) [Design Philosophy On Data And Semantics](https://www.ardanlabs.com/blog/2017/06/design-philosophy-on-data-and-semantics.html)

Watch this video to see a live demo of this code:
[GopherCon Singapore (2017) - Escape Analysis](https://engineers.sg/video/go-concurrency-live-gophercon-sg-2017--1746)

### Introduction

In the previous post, I taught the basics on escape analysis by using an example that shared a value up a goroutine’s stack. What I did not show you are other scenarios that can cause values to escape. To help you with this, I am going to debug a program that is allocating in surprising ways.

### The Program

I wanted to learn more about the `io` package so I gave myself a quick project. Given a stream of bytes, write a function that can find the string `elvis` and replace it with the capitalized version of the string `Elvis`. We are talking about the King here, so his name should always be capitalized.

Here is a link to the solution:
https://play.golang.org/p/n_SzF4Cer4

Here is a link to the benchmarks:
https://play.golang.org/p/TnXrxJVfLV

*The code listing has two different functions that solve this problem. This post will focus its attention on the `algOne` function since it is using the `io` package. Use the `algTwo` function to experiment with memory and cpu profiles on your own.*

Here is the input data we are going to use and expected output the `algOne` function is expected to produce.

**Listing 1**

```
Input:
abcelvisaElvisabcelviseelvisaelvisaabeeeelvise l v i saa bb e l v i saa elvi
selvielviselvielvielviselvi1elvielviselvis

Output:
abcElvisaElvisabcElviseElvisaElvisaabeeeElvise l v i saa bb e l v i saa elvi
selviElviselvielviElviselvi1elviElvisElvis
```

Here is a listing the of `algOne` function in its entirety.

**Listing 2**

```
 80 func algOne(data []byte, find []byte, repl []byte, output *bytes.Buffer) {
 81
 82     // Use a bytes Buffer to provide a stream to process.
 83     input := bytes.NewBuffer(data)
 84
 85     // The number of bytes we are looking for.
 86     size := len(find)
 87
 88     // Declare the buffers we need to process the stream.
 89     buf := make([]byte, size)
 90     end := size - 1
 91
 92     // Read in an initial number of bytes we need to get started.
 93     if n, err := io.ReadFull(input, buf[:end]); err != nil {
 94         output.Write(buf[:n])
 95         return
 96     }
 97
 98     for {
 99
100         // Read in one byte from the input stream.
101         if _, err := io.ReadFull(input, buf[end:]); err != nil {
102
103             // Flush the reset of the bytes we have.
104             output.Write(buf[:end])
105             return
106         }
107
108         // If we have a match, replace the bytes.
109         if bytes.Compare(buf, find) == 0 {
110             output.Write(repl)
111
112             // Read a new initial number of bytes.
113             if n, err := io.ReadFull(input, buf[:end]); err != nil {
114                 output.Write(buf[:n])
115                 return
116             }
117
118             continue
119         }
120
121         // Write the front byte since it has been compared.
122         output.WriteByte(buf[0])
123
124         // Slice that front byte out.
125         copy(buf, buf[1:])
126     }
127 }
```

What I want to know is how well this function performs and what kind of pressure is it placing on the heap. To learn this we are going to run a benchmark.

### Benchmarking

Here is the benchmark function I wrote that calls into the `algOne` function to perform the data stream processing.

**Listing 3**

```
15 func BenchmarkAlgorithmOne(b *testing.B) {
16     var output bytes.Buffer
17     in := assembleInputStream()
18     find := []byte("elvis")
19     repl := []byte("Elvis")
20
21     b.ResetTimer()
22
23     for i := 0; i < b.N; i++ {
24         output.Reset()
25         algOne(in, find, repl, &output)
26     }
27 }
```

With this benchmark function in place, we can run it through `go test` using the `-bench`, `-benchtime` and `-benchmem` switches.

**Listing 4**

```
$ go test -run none -bench AlgorithmOne -benchtime 3s -benchmem
BenchmarkAlgorithmOne-8    	2000000 	     2522 ns/op       117 B/op  	      2 allocs/op
```

After running the benchmark we can see that the `algOne` function is allocating 2 values worth a total of 117 bytes per operation. This is great, but we need to know what lines of code in the function are causing these allocations. To learn this, we need to generate profiling data for this benchmark.

### Profiling

To generate the profile data, we are going to run the benchmark again but this time ask for a memory profile by using the `-memprofile` switch.

**Listing 5**

```
$ go test -run none -bench AlgorithmOne -benchtime 3s -benchmem -memprofile mem.out
BenchmarkAlgorithmOne-8    	2000000 	     2570 ns/op       117 B/op  	      2 allocs/op
```

Once the benchmark is finished, the test tool has produced two new files.

**Listing 6**

```
~/code/go/src/.../memcpu
$ ls -l
total 9248
-rw-r--r--  1 bill  staff      209 May 22 18:11 mem.out       (NEW)
-rwxr-xr-x  1 bill  staff  2847600 May 22 18:10 memcpu.test   (NEW)
-rw-r--r--  1 bill  staff     4761 May 22 18:01 stream.go
-rw-r--r--  1 bill  staff      880 May 22 14:49 stream_test.go
```

The source code is in a folder called `memcpu` with the `algOne` function inside of `stream.go` and the benchmark function inside of `stream_test.go`. The two new files that were produced are called `mem.out` and `memcpu.test`. The `mem.out` file contains the profile data and the `memcpu.test` file, named after the folder, contains a test binary we need to have access to symbols when looking at the profile data.

With the profile data and test binary in place, we can now run the `pprof` tool to study the profile data.

**Listing 7**

```
$ go tool pprof -alloc_space memcpu.test mem.out
Entering interactive mode (type "help" for commands)
(pprof) _
```

When profiling memory and looking for “low hanging fruit”, you want to use the `-alloc_space` option instead of the default `-inuse_space` option. This will show you where every allocation is happening regardless if it is still in memory or not at the time you take the profile.

From the `(pprof)` prompt, we can inspect the `algOne` function using the `list` command. This command takes a regular expression as an argument to find the function(s) you want to look at.

**Listing 8**

```
(pprof) list algOne
Total: 335.03MB
ROUTINE ======================== .../memcpu.algOne in code/go/src/.../memcpu/stream.go
 335.03MB   335.03MB (flat, cum)   100% of Total
        .          .     78:
        .          .     79:// algOne is one way to solve the problem.
        .          .     80:func algOne(data []byte, find []byte, repl []byte, output *bytes.Buffer) {
        .          .     81:
        .          .     82: // Use a bytes Buffer to provide a stream to process.
 318.53MB   318.53MB     83: input := bytes.NewBuffer(data)
        .          .     84:
        .          .     85: // The number of bytes we are looking for.
        .          .     86: size := len(find)
        .          .     87:
        .          .     88: // Declare the buffers we need to process the stream.
  16.50MB    16.50MB     89: buf := make([]byte, size)
        .          .     90: end := size - 1
        .          .     91:
        .          .     92: // Read in an initial number of bytes we need to get started.
        .          .     93: if n, err := io.ReadFull(input, buf[:end]); err != nil || n < end {
        .          .     94:       output.Write(buf[:n])
(pprof) _
```

Based on this profile, we now know `input` and the backing array of the `buf` slice is allocating to the heap. Since `input` is a pointer variable, the profile is really saying that the `bytes.Buffer` value that the `input` pointer points to is allocating. So let’s focus on the `input` allocation first and understand why it is allocating.

We could assume it is allocating because the function call to `bytes.NewBuffer` is sharing the `bytes.Buffer` value it creates up the call stack. However, the existence of a value in the `flat` column (the first column in the pprof output) tells me that the value is allocating because the `algOne` function is sharing it in a way to cause it to escape.

I know the `flat` column represents in function allocations because look at what the `list` command shows for the `Benchmark` function which is calling into `algOne`.

**Listing 9**

```
(pprof) list Benchmark
Total: 335.03MB
ROUTINE ======================== .../memcpu.BenchmarkAlgorithmOne in code/go/src/.../memcpu/stream_test.go
        0   335.03MB (flat, cum)   100% of Total
        .          .     18: find := []byte("elvis")
        .          .     19: repl := []byte("Elvis")
        .          .     20:
        .          .     21: b.ResetTimer()
        .          .     22:
        .   335.03MB     23: for i := 0; i < b.N; i++ {
        .          .     24:       output.Reset()
        .          .     25:       algOne(in, find, repl, &output)
        .          .     26: }
        .          .     27:}
        .          .     28:
(pprof) _
```

Since there is only a value in the `cum` column (the second column), this is telling me that the `Benchmark` function is allocating nothing directly. All allocations are happening from function calls that are being made inside that loop. You can see all the allocation numbers from these two `list` calls match.

We still don’t know why the `bytes.Buffer` value is allocating. This is where the `-gcflags "-m -m"` switch on the `go build` command will come in handy. The profiler can only tell you what values are escaping, but the build command can tell you why.

### Compiler Reporting

Let’s ask the compiler what decisions it’s making as it relates to escape analysis on the code.

**Listing 10**

```
$ go build -gcflags "-m -m"
```

This command produces a lot of output. We just need to search the output for anything that has `stream.go:83` since `stream.go` is the name of the file that contains this code and line 83 contains the construction of the `bytes.buffer` value. After the search we find 6 lines.

**Listing 11**

```
./stream.go:83: inlining call to bytes.NewBuffer func([]byte) *bytes.Buffer { return &bytes.Buffer literal }

./stream.go:83: &bytes.Buffer literal escapes to heap
./stream.go:83:   from ~r0 (assign-pair) at ./stream.go:83
./stream.go:83:   from input (assigned) at ./stream.go:83
./stream.go:83:   from input (interface-converted) at ./stream.go:93
./stream.go:83:   from input (passed to call[argument escapes]) at ./stream.go:93
```

The first line we found for `stream.go:83` is interesting.

**Listing 12**

```
./stream.go:83: inlining call to bytes.NewBuffer func([]byte) *bytes.Buffer { return &bytes.Buffer literal }
```

It confirms that the `bytes.Buffer` value didn’t escape because it was passed up the call stack. This is because the call to `bytes.NewBuffer` was never called, the code inside the function was inlined.

So this is the piece of code I wrote:

**Listing 13**

```
83     input := bytes.NewBuffer(data)
```

Due to the compiler choosing to inline the `bytes.NewBuffer` function call, the code I wrote is converted to this:

**Listing 14**

```
input := &bytes.Buffer{buf: data}
```

That means the `algOne` function is constructing the `bytes.Buffer` value directly. So now the question is, what is causing the value to escape from the `algOne` stack frame? That answer is in the other 5 lines we found in the report.

**Listing 15**

```
./stream.go:83: &bytes.Buffer literal escapes to heap
./stream.go:83:   from ~r0 (assign-pair) at ./stream.go:83
./stream.go:83:   from input (assigned) at ./stream.go:83
./stream.go:83:   from input (interface-converted) at ./stream.go:93
./stream.go:83:   from input (passed to call[argument escapes]) at ./stream.go:93
```

What these lines are telling us, is that the code at line 93 is causing the escape. The `input` variable is being assigned to an interface value.

### Interfaces

I don’t remember making an assignment to an interface value at all in the code. However, if you look at line 93, it becomes clear what is happening.

**Listing 16**

```
 93     if n, err := io.ReadFull(input, buf[:end]); err != nil {
 94         output.Write(buf[:n])
 95         return
 96     }
```

The call to `io.ReadFull` is causing the interface assignment. If you look the definition of the `io.ReadFull` function, you can see how it is accepting the `input` value through an interface type.

**Listing 17**

```
type Reader interface {
      Read(p []byte) (n int, err error)
}

func ReadFull(r Reader, buf []byte) (n int, err error) {
      return ReadAtLeast(r, buf, len(buf))
}
```

It appears that passing the `bytes.Buffer` address down the call stack and storing it inside of the `Reader` interface value is causing an escape. Now we know there is a cost to using an interface: allocation and indirection. So, if it’s not clear how an interface makes the code better, you probably don’t want to use one. Here are some guidelines that I follow to validate the use of interfaces in my code.

Use an interface when:

- users of the API need to provide an implementation detail.
- API’s have multiple implementations they need to maintain internally.
- parts of the API that can change have been identified and require decoupling.

Don’t use an interface:

- for the sake of using an interface.
- to generalize an algorithm.
- when users can declare their own interfaces.

Now we can ask ourselves, does this algorithm really need the `io.ReadFull` function? The answer is no because the `bytes.Buffer` type has a method set that we can use. Using methods against a value that a function owns can prevent allocations.

Let’s make the code change to remove the `io` package and use the `Read` method directly against the `input` variable.

*This code change removes the need to import the `io` package, so to keep all the line numbers the same, I am using the blank identifier against the `io` package import. This will allow the import to stay in the list.*

**Listing 18**

```
 12 import (
 13     "bytes"
 14     "fmt"
 15     _ "io"
 16 )

 80 func algOne(data []byte, find []byte, repl []byte, output *bytes.Buffer) {
 81
 82     // Use a bytes Buffer to provide a stream to process.
 83     input := bytes.NewBuffer(data)
 84
 85     // The number of bytes we are looking for.
 86     size := len(find)
 87
 88     // Declare the buffers we need to process the stream.
 89     buf := make([]byte, size)
 90     end := size - 1
 91
 92     // Read in an initial number of bytes we need to get started.
 93     if n, err := input.Read(buf[:end]); err != nil || n < end {
 94         output.Write(buf[:n])
 95         return
 96     }
 97
 98     for {
 99
100         // Read in one byte from the input stream.
101         if _, err := input.Read(buf[end:]); err != nil {
102
103             // Flush the reset of the bytes we have.
104             output.Write(buf[:end])
105             return
106         }
107
108         // If we have a match, replace the bytes.
109         if bytes.Compare(buf, find) == 0 {
110             output.Write(repl)
111
112             // Read a new initial number of bytes.
113             if n, err := input.Read(buf[:end]); err != nil || n < end {
114                 output.Write(buf[:n])
115                 return
116             }
117
118             continue
119         }
120
121         // Write the front byte since it has been compared.
122         output.WriteByte(buf[0])
123
124         // Slice that front byte out.
125         copy(buf, buf[1:])
126     }
127 }
```

When we run the benchmark against this code change, we can see that the allocation for the `bytes.Buffer` value is gone.

**Listing 19**

```
$ go test -run none -bench AlgorithmOne -benchtime 3s -benchmem -memprofile mem.out
BenchmarkAlgorithmOne-8    	2000000 	     1814 ns/op         5 B/op  	      1 allocs/op
```

We also see there was a performance improvement of about ~29%. The code went from `2570 ns/op` to `1814 ns/op`. With this solved, we can now focus on the allocation of the backing array for the `buf` slice. If we use the profiler once more against the new profile data we just produced, we should be able to identify what is causing the remaining allocation.

**Listing 20**

```
$ go tool pprof -alloc_space memcpu.test mem.out
Entering interactive mode (type "help" for commands)
(pprof) list algOne
Total: 7.50MB
ROUTINE ======================== .../memcpu.BenchmarkAlgorithmOne in code/go/src/.../memcpu/stream_test.go
     11MB       11MB (flat, cum)   100% of Total
        .          .     84:
        .          .     85: // The number of bytes we are looking for.
        .          .     86: size := len(find)
        .          .     87:
        .          .     88: // Declare the buffers we need to process the stream.
     11MB       11MB     89: buf := make([]byte, size)
        .          .     90: end := size - 1
        .          .     91:
        .          .     92: // Read in an initial number of bytes we need to get started.
        .          .     93: if n, err := input.Read(buf[:end]); err != nil || n < end {
        .          .     94:       output.Write(buf[:n])
```

The only allocation left is on line 89, which is for the backing array of the slice.

### Stack Frames

We want to know why the backing array for `buf` is allocating? Let’s run `go build` again using the `-gcflags "-m -m"` option and search for `stream.go:89`.

**Listing 21**

```
$ go build -gcflags "-m -m"
./stream.go:89: make([]byte, size) escapes to heap
./stream.go:89:   from make([]byte, size) (too large for stack) at ./stream.go:89
```

The report says the backing array is “too large for stack”. This message is very misleading. It’s not that the backing array is too large, but that the compiler doesn’t know what the size of the backing array is compile time.

Values can only be placed on the stack if the compiler knows the size of the value at compile time. This is because the size of every stack frame, for every function, is calculated at compile time. If the compiler doesn’t know the size of a value, it is placed on the heap.

To show this, let’s temporarily hard code the size of the slice to `5` and run the benchmark again.

**Listing 22**

```
 89     buf := make([]byte, 5)
```

This time when we run the benchmark the allocations are gone.

**Listing 23**

```
$ go test -run none -bench AlgorithmOne -benchtime 3s -benchmem
BenchmarkAlgorithmOne-8    	3000000 	     1720 ns/op         0 B/op  	      0 allocs/op
```

If you look at the compiler report once more, you will see nothing is escaping.

**Listing 24**

```
$ go build -gcflags "-m -m"
./stream.go:83: algOne &bytes.Buffer literal does not escape
./stream.go:89: algOne make([]byte, 5) does not escape
```

Obviously we can’t hard code the size of the slice, so we will need to live with the 1 allocation for this algorithm.

### Allocations and Performance

Compare the different performance gains we achieved throughout each refactoring.

**Listing 25**

```
Before any optimization
BenchmarkAlgorithmOne-8    	2000000 	     2570 ns/op       117 B/op  	      2 allocs/op

Removing the bytes.Buffer allocation
BenchmarkAlgorithmOne-8    	2000000 	     1814 ns/op         5 B/op  	      1 allocs/op

Removing the backing array allocation
BenchmarkAlgorithmOne-8    	3000000 	     1720 ns/op         0 B/op  	      0 allocs/op
```

We got ~29% increase in performance by removing the allocation of the `bytes.Buffer` and ~33% speed up when all the allocations were removed. Allocations are a place where application performance can suffer.

### Conclusion

Go has some amazing tooling that allows you to understand the decisions the compiler is making as it relates to escape analysis. Based on this information, you can refactor code to be sympathetic with keeping values on the stack that don’t need to be on the heap. You are not going to write zero allocation software but you want to minimize allocations when possible.

That being said, never write code with performance as your first priority because you don’t want to be guessing about performance. Write code that optimizes for correctness as your first priority. This means focus on integrity, readability and simplicity first. Once you have a working program, identify if the program is fast enough. If it’s not fast enough, then use the tooling the language provides to find and fix your performance issues.