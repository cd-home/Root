### Design Philosophy On Data And Semantics

### Prelude

This is the final post in a four part series discussing the mechanics and design behind pointers, stacks, heaps, escape analysis and value/pointer semantics in Go. This post focuses on data and the design philosophies of applying value/pointer semantics in your code.

Index of the four part series:
\1) [Language Mechanics On Stacks And Pointers](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html)
\2) [Language Mechanics On Escape Analysis](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html)
\3) [Language Mechanics On Memory Profiling](https://www.ardanlabs.com/blog/2017/06/language-mechanics-on-memory-profiling.html)
\4) [Design Philosophy On Data And Semantics](https://www.ardanlabs.com/blog/2017/06/design-philosophy-on-data-and-semantics.html)

### Design Philosophies

***“Value semantics keep values on the stack, which reduces pressure on the Garbage Collector (GC). However, value semantics require various copies of any given value to be stored, tracked and maintained. Pointer semantics place values on the heap, which can put pressure on the GC. However, pointer semantics are efficient because only one value needs to be stored, tracked and maintained.” - Bill Kennedy\***

A consistent use of value/pointer semantics, for a given type of data, is critical if you want to maintain integrity and readability throughout your software. Why? Because, if you are changing the semantics for a piece of data as it is passed between functions, you are making it difficult to maintain a clear and consistent mental model of the code. The larger the code base and the team becomes, the more bugs, data races and side effects will creep unseen into the code base.

I want to start with a set of design philosophies that will drive the guidelines for choosing one semantic over the other.

#### Mental Models

***“Let’s imagine a project that’s going to end up with a million lines of code or more. The probability of those projects being successful in the United States these days is very low - well under 50%. That’s debatable”. - Tom Love (inventor of Objective C)\***

Tom has also mentioned that a box of copy paper can hold 100k lines of code. Take a second to let that sink in. For what percentage of the code in that box could you maintain a mental model of?

I believe asking a single developer to maintain a mental model of more than one ream of paper (~10k lines of code) is already asking quite a bit. But, if we go ahead assume ~10k lines of code per developer, it would take a team of 100 developers to maintain on a code base that hits a million lines of code. That is 100 people that need to be coordinated, grouped, tracked and in constant communication. Now look at your current team of 1 to 10 devs. How well are you doing with this at a much smaller scale? At 10k lines of code per developer, is the size of your code base in line with the size of the team?

#### Debugging

***“The hardest bugs are those where your mental model of the situation is just wrong, so you can’t see the problem at all” - Brian Kernighan\***

I don’t believe in using a debugger until you have lost your mental model of the code and are now wasting effort trying to understand the problem. Debuggers are evil when they are abused, and you know that you are abusing a debugger when it becomes the first reaction to any observable bug.

If you have a problem in production, what are you going to ask for? That’s right, the logs. If the logs are not working for you during development, they are certainly not going to work for you when that production bug hits. Logs require having a mental model of the code base, such that you can read code to find bugs.

#### Readability

***“C is the best balance I’ve ever seen between power and expressiveness. You can do almost anything you want to do by programming fairly straightforwardly and you will have a very good mental model of what’s going to happen on the machine; you can predict reasonably well how quickly it’s going to run, you understand what’s going on ….” - Brian Kernighan\***

I believe this quote by Brian applies just as well to Go. Maintaining this “mental model” is everything. It drives integrity, readability and simplicity. These are the cornerstones of well-written software that allow it to be maintained and last over time. Writing code that maintains a consistent use of value/pointer semantics for a given type of data is an important way to achieve this.

#### Data Oriented Design

***“If you don’t understand the data, you don’t understand the problem. This is because all problems are unique and specific to the data you are working with. When the data is changing, your problems are changing. When your problems are changing, the algorithms (data transformations) needs to change with it.” - Bill Kennedy\***

Think about this. Every problem you work on is a data transformation problem. Every function you write and every program you run takes some input data and produces some output data. Your mental model of software is, from this perspective, an understanding of these data transformations (i.e., how they are organized and applied in the code base). A “less is more” attitude is critical to solving problems with fewer layers, statements, generalizations, less complexity and less effort. This makes everything easier on you and your teams, but it also makes it easier for the hardware to execute these data transformations.

#### Type (Is Life)

***“Integrity means that every allocation, every read of memory and every write of memory is accurate, consistent and efficient. The type system is critical to making sure we have this micro level of integrity.” - William Kennedy\***

If data drives everything you do, then the types that represent the data are critical. In my world “Type Is Life”, because type provide the compiler with the ability to ensure the integrity of the data. Type also drives and dictates the semantic rules code must respect for the data that it operates on. This is where the proper use of value/pointer semantics starts: with types.

#### Data (With Capability)

***“Methods are valid when it is practical or reasonable for a piece of data to have a capability.” - William Kennedy\***

The idea of value/pointer semantics doesn’t hit Go developers in the face until they have to make a decision about the receiver type for a method. It’s a question I see come up quite a bit: should I use a value receiver or pointer receiver? Once I hear this question, I know that the developer doesn’t have a good grasp of these semantics.

The purpose of methods is to give a piece of data capability. Think about that. A piece of data can have the capability to do something. I always want the focus to be on the data because it is the data that drives the functionality of your programs. Data drives the algorithms you write, the encapsulations you put in place and the performance you can achieve.

#### Polymorphism

***“Polymorphism means that you write a certain program and it behaves differently depending on the data that it operates on.” - Tom Kurtz (inventor of BASIC)\***

I love what Tom said in that quote above. A function can behave differently depending on the data it operates on. That the behavior of data is what decouples functions from the concrete data types they can accept and work with. This is one core reason for a piece of data to have a capability. It is this idea that is the cornerstone of architecting and designing systems that can adapt to change.

#### Prototype First Approach

***“Unless the developer has a really good idea of what the software is going to be used for, there’s a very high probability that the software will turn out badly. If the developers don’t know and understand the application well, then it’s crucial to get as much user input and experience as possible.” - Brian Kernighan\***

I want you to always focus first on understanding the concrete data and algorithms you need to get data transformations working to solve problems. Take this prototype first approach and write concrete implementations that could also be deployed in production (if it is reasonable and practical to do so). Once a concrete implementation is working and once you’ve learned what works and doesn’t work, focus on refactoring to decouple the implementation from the concrete data by giving the data capability.

### Semantic Guidelines

You must decide which semantic, value or pointer, is going to be used for a particular data type at the time the type is declared. API’s that accepts or return data of that type must respect the semantic chosen for the type. API’s are not allowed to dictate or change semantics. They must know what semantic is being used for the data and conform to that. This is, at least partially, how consistency across a large code base can be achieved.

Here are the basic guidelines:

- At the time you declare a type you must decide what semantic is being used.
- Functions and methods must respect the semantic choice for the given type.
- Avoid having method receivers that use different semantics than those corresponding to a given type.
- Avoid having functions that accept/return data using different semantics than those corresponding to the given type.
- Avoid changing the semantic for a given type.

*There are a few exceptions to these guidelines with the biggest being unmarshaling. Unmarshaling always requires the use of pointer semantics. Marshaling and unmarshaling seem to always be exceptions to the rule.*

How do you chose one semantic over the other for a given type? These guidelines will help you answer the question. Below we will apply the guidelines in certain scenarios:

#### Built-In Types

Go’s built-in types represent numeric, textual and boolean data. These types should be handled using value semantics. Don’t use pointers to share values of these types unless you have a very good reason.

As an example, look at these function declarations from the `strings` package.

**Listing 1**

```
func Replace(s, old, new string, n int) string
func LastIndex(s, sep string) int
func ContainsRune(s string, r rune) bool
```

All of these functions use value semantics in the design of the API.

#### Reference Types

Reference types represent the slice, map, interface, function and channel types in the language. These types should be using value semantics because they have been designed to stay on the stack and minimize heap pressure. They allow each function to have their own copy of the value, as opposed to each function call causing a potential allocation. This is possible because these values contain a pointer that share the underlying data structure between calls.

Don’t use pointers to share values of these types unless you have a very good reason. Sharing a slice or map value down the call stack into an `Unmarshal` function could be one exception. As an example, look at these two types declared in the `net` package.

**Listing 2**

```
type IP []byte
type IPMask []byte
```

The `IP` and `IPMask` types are both based on a slice of bytes. This means they are both reference types and they should follow the value semantic rules. Here is a method named `Mask` that was declared for the `IP` type that accepts a value of `IPMask`.

**Listing 3**

```
func (ip IP) Mask(mask IPMask) IP {
    if len(mask) == IPv6len && len(ip) == IPv4len && allFF(mask[:12]) {
        mask = mask[12:]
    }
    if len(mask) == IPv4len && len(ip) == IPv6len && bytesEqual(ip[:12], v4InV6Prefix) {
        ip = ip[12:]
    }
    n := len(ip)
    if n != len(mask) {
        return nil
    }
    out := make(IP, n)
    for i := 0; i < n; i++ {
        out[i] = ip[i] & mask[i]
    }
    return out
}
```

Notice that this method is a mutation operation and is using a value semantic API style. It uses a value of `IP` as the receiver and depending on the `IPMask` value that is passed in, creates a new `IP` value and returns a copy of it back to the caller. The method is respecting the fact that you use value semantics for references types.

This is the same for the built-in function `append`.

**Listing 4**

```
var data []string
data = append(data, "string")
```

The `append` function is using value semantics for this mutation operation. You pass a slice value into `append` and after the mutation it returns a new slice value.

The exception is always unmarshaling, which requires pointer semantics.

**Listing 5**

```
func (ip *IP) UnmarshalText(text []byte) error {
  	if len(text) == 0 {
  		*ip = nil
  		return nil
  	}
  	s := string(text)
  	x := ParseIP(s)
  	if x == nil {
  		return &ParseError{Type: "IP address", Text: s}
  	}
  	*ip = x
  	return nil
  }
```

The `UnmarshalText` method is implementing the `encoding.TextUnmarshaler` interface. If pointer semantics were not used, it wouldn’t work. But this is ok because it is usually safe to share a value. Outside of unmarshaling, you should raise a flag if pointer semantics are being used for a reference type.

#### User Defined Types

This is where you will need to make the most decisions. You must decide at the time you are declaring a type what semantic will be used.

What if I asked you to write the API for the `time` package and I gave you this type.

**Listing 6**

```
type Time struct {
    sec  int64
    nsec int32
    loc  *Location
}
```

What semantic would you use?

Look at the implementation of this type in the `Time` package along with the factory function `Now`.

**Listing 7**

```
func Now() Time {
  	sec, nsec := now()
  	return Time{sec + unixToInternal, nsec, Local}
  }
```

A factory function is one of the most important functions for a type because it tells you what semantic is being chosen. The `Now` function is making it clear that value semantics are in play. This function creates a value of type `Time` and returns a copy of this value back to the caller. Sharing `Time` values is not necessary and they don’t need to end up on the heap.

Also, look at the `Add` method, which is a mutation operation.

**Listing 8**

```
func (t Time) Add(d Duration) Time {
  	t.sec += int64(d / 1e9)
  	nsec := t.nsec + int32(d%1e9)
  	if nsec >= 1e9 {
  		t.sec++
  		nsec -= 1e9
  	} else if nsec < 0 {
  		t.sec--
  		nsec += 1e9
  	}
  	t.nsec = nsec
  	return t
  }
```

Again you can see that the `Add` method is respecting the semantic chosen for the type. The `Add` method is using a value receiver to operate on its own copy of the `Time` value, where that copy of the `Time` value is being used to make the call. It then mutates its own copy and returns a new copy of the `Time` value back to the call.

Here is a function that accepts `Time` values:

**Listing 9**

```
func div(t Time, d Duration) (qmod2 int, r Duration) {
```

Once again, value semantics are being used to accept a value of type `Time`. The only use of pointer semantics for the `Time` API are these `Unmarshal` related functions:

**Listing 10**

```
func (t *Time) UnmarshalBinary(data []byte) error {
func (t *Time) GobDecode(data []byte) error {
func (t *Time) UnmarshalJSON(data []byte) error {
func (t *Time) UnmarshalText(data []byte) error {
```

Most of the time your ability to use value semantics is limiting. It isn’t correct or reasonable to make copies of the data as it passes from function to function. Changes to the data need to be isolated to a single value and shared. This is when pointer semantics need to be used. If you are not 100% sure it is correct or reasonable to make copies, then use pointer semantics.

Look at the factory function for the `File` type in the `os` package.

**Listing 11**

```
func Open(name string) (file *File, err error) {
    return OpenFile(name, O_RDONLY, 0)
}
```

The `Open` function is returning a pointer of type `File`. This means you should be using pointer semantics and always share `File` values. Changing the semantic from pointer to value could be devastating to your program. When a function shares a value with you, you should assume that you are not allowed to make a copy of the value pointed to by the pointer. If you do, results will be undefined.

Looking at more of the API you will see the consistent use of pointer semantics.

**Listing 12**

```
func (f *File) Chdir() error {
    if f == nil {
        return ErrInvalid
    }
    if e := syscall.Fchdir(f.fd); e != nil {
        return &PathError{"chdir", f.name, e}
    }
    return nil
}
```

The `Chdir` method uses pointer semantics even though the `File` value is never mutated. The method must respect the semantic convention for the type.

**Listing 13**

```
func epipecheck(file *File, e error) {
    if e == syscall.EPIPE {
        if atomic.AddInt32(&file.nepipe, 1) >= 10 {
            sigpipe()
        }
    } else {
        atomic.StoreInt32(&file.nepipe, 0)
    }
}
```

Here is a function named `epipecheck` and it’s using pointer semantics to accept the `File` value. Again, notice the consistent use of pointer semantics for values of type `File`.

### Conclusion

The consistent use of value/pointer semantics is something I look for in code reviews. It helps you keep code consistent and predictable over time. It also allows everyone to maintain a clear and consistent mental model of the code. As the code base and the team gets larger, consistent use of value/pointer semantics becomes even more important.

What’s amazing about Go is that the choice between pointer and value semantics goes beyond the declaration of receivers and function parameters. It’s all over the language, from how `for range` works to the mechanics of interfaces, function values and slices. In future posts, I will show how value/pointer semantics show up in these different parts of the language.