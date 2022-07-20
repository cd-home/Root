### Introduction To Numeric Constants In Go

**Introduction**
One of the more unique features of Go is how the language implements constants. The rules for [constants](http://golang.org/ref/spec#Constants) in the language specification are unique to Go. They provide the flexibility Go needs at the compiler level to make the code we write readable and intuitive while still maintaining a type safe language.

This post will attempt to build a foundation for what numeric constants are, how they behave in their simplest form and how best to talk about them. There are a lot of little nuances, terms and concepts that can trip us up. Because of this, the post is going to take things slowly.

So if you are ready to peek under the covers just a bit, roll up your sleeve and let’s get started:

**Untyped and Typed Numeric Constants**
Constants can be declared with or without a type in Go. When we declare literal values in our code, we are actually declaring constants that are both untyped and unnamed.

The following examples show typed and untyped numeric constants that are both named and unnamed:

~~~go
const untypedInteger    = 12345
const untypedFloatingPoint = 3.141592

const typedInteger int      = 12345
const typedFloatingPoint float64 = 3.141592
~~~

The constants on the left hand side of the declaration are named constants and the literal values on the right hand side are unnamed constants.

**Kinds of Numeric Constants**
Your first instinct may be to think that typed constants use the same type system as variables, but they don’t. Constants have their own implementation for representing the values that we associate with them. Every Go [compiler](http://golang.org/cmd/) has the flexibility to implement constants as they wish, within a set of mandatory [requirements](http://golang.org/ref/spec#Constants).

When declaring a typed constant, the declared type is used to associate the type’s precision limitations. It does not change how the value is being internally represented. Because the internal representation of constants can be different between the different compilers, it is best to think of constants as having a *kind*, not a type.

Numeric constants can be one of four kinds: integer, floating-point, complex and rune:

~~~go
12345  		// kind: integer
3.141592 	// kind: floating-point
1E6    		// kind: floating-point
~~~

In the example above, we have declared three numeric constants, one of kind integer and two of kind floating-point. The form of the literal value will determine what kind the constant takes. When the form of the literal value contains a decimal or exponent, the constant is of kind floating-point. When the form does not contain a decimal or exponent, the constant is of kind integer.

**Constants Are Mathematically Exact**
Regardless of the implementation, constants are always considered to be mathematically exact. This is something that makes constants in Go unique. This is not the case in other languages like C and C++.

Integers can always be represented precisely when there is enough memory to store their entire value. Since the specification requires integer constants to have at least 256 bits of precision, we are safe in saying integer constants are mathematically exact.

To have mathematically exact floating-point numbers, there are different strategies and options that the compiler can employ. The specification does not state how the compiler must do this, it just specifies a set of mandatory requirements that need to be met.

Here are two strategies that the different Go compilers use today to implement mathematically exact floating-point numbers:

- One strategy is to represent all floating-point numbers as fractions, and use rational arithmetic on those fractions. This is what go/types does today and these floating-point numbers never have any loss of precision.
- Another strategy is to use floating-point numbers with so much precision that they appear to be exact for all practical purposes. When we use floating-point numbers with several hundred bits, the difference between exact and approximate becomes virtually non-existent. This is what the gc/gccgo compilers do today.

As developers however, it is best to not consider what internal representation is being used by the compiler, it is irrelevant. Just remember that all constants, regardless if they are declared with or without a type, use the same representation to store their values, which is not the same as variables and is mathematically exact.

**Mathematically Exact Example**
Since constants only exist during compilation, it is hard to provide an example that shows constants are mathematically exact. One way is to show how the compiler will let us declare constants of kind integer with values that are much larger than the largest integer types can support.

Here is a program that can be compiled because constants of kind integer are mathematically exact:

~~~go
package main

import "fmt"

// Much larger value than int64.
const myConst = 9223372036854775808543522345

func main() {
  fmt.Println("Will Compile")
}
~~~

If we change the constant to be of type int64, which means the constant is now bound to the precision limitations of a 64 bit integer, the program will no longer compile:

~~~go
package main

import "fmt"

// Much larger value than int64.
const myConst int64 = 9223372036854775808543522345

func main() {
  fmt.Println("Will NOT Compile")
}

Compiler Error:
./ideal.go:6: constant 9223372036854775808543522345 overflows int64


~~~

Here we can see that constants of kind integer can represent very large numbers and why we say they are mathematically exact.

**Numeric Constant Declarations**
When we declare an untyped numeric constant, there are no type constraints that must be met by the constant value:

~~~go
const untypedInteger    = 12345   	  // kind: integer
const untypedFloatingPoint = 3.141592 // kind: floating-point
~~~

In each case, the untyped constant on the left hand side of the declaration is given the same kind and value as the constant on the right.

When we declare a typed constant, the constant on the right hand side of the declaration must use a form that is compatible with the declared type on the left:

~~~go
const typedInteger int      = 12345   // kind: integer
const typedFloatingPoint float64 = 3.141592 // kind: floating-point
~~~

The value on the right hand side of the declaration must also fit into the range for the declared type. For instance, this numeric constant declaration is invalid:

~~~go
const myUint8 uint8 = 1000
~~~

uint8 only can represent numbers from 0 to 255. This is what I mean when I said earlier that the declared type is used to associate the type’s precision limitations.

**Implicit Integer Type Conversions**
In Go there are no implicit type [conversions](http://golang.org/ref/spec#Conversions) between variables. However, implicit type conversions between variables and constants can happen regularly by the compiler.

Let’s start with an implicit integer conversion:

~~~go
var myInt int = 123
~~~

In this example we have constant 123 of kind integer being implicitly converted to a value of type int. Since the form of the constant is not using a decimal point or exponent, the constant takes the kind integer. Constants of kind integer can be implicitly converted into signed and unsigned integer variables of any length as long as no truncation needs to take place.

Constants of kind floating-point can also be implicitly converted into integer variables if the constant uses a form that is compatible with the integer type:

~~~go
var myInt int = 123.0
~~~

We can also perform implicit type conversion assignments without declaring an explicit type for the variable:

~~~go
var myInt = 123
~~~

In this case, the default type of int64 is used to initialize the variable being assigned with constant 123 of kind integer.

**Implicit Floating-Point Type Conversions**
Next let’s look at an implicit floating-point conversion:

~~~go
var myFloat float64 = 0.333
~~~

This time the compiler is performing an implicit conversion between constant 0.333 of kind floating-point to a variable of type float64. Since the form of the constant is using a decimal point, the constant takes the kind floating-point. The default type for a variable initialized with a constant of kind floating-point is float64.

The compiler can also perform implicit conversions between constants of kind integer to variables of type float64:

~~~go
var myFloat float64 = 1
~~~

In this example, constant 1 of kind integer is implicitly converted to a variable of type float64.

**Kind Promotion**
Performing constant arithmetic between other constants and variables is something we do quite often in our programs. It follows the rules for [binary operators](http://golang.org/ref/spec#Operators) in the specification. The rule states that operand types must be identical unless the operation involves shifts or untyped constants.

Let’s look at an example of two constants that are multiplied together:

~~~go
var answer = 3 * 0.333
~~~

In this example we perform multiplication between constant 3 of kind integer and constant 0.333 of kind floating-point.

There is a rule in the specification about constant expressions that is specific to this operation:

*"Except for shift operation, if the operands of a binary operation are different kinds of untyped constants, …, the result use the kind that appears later in this list: integer, rune, floating-point, complex."*

Based on this rule, the result of the multiplication between these two constants will be a constant of kind floating-point. Kind floating-point is being promoted ahead of kind integer based on the rule.

**Numeric Constant Arithmetic**
Let’s continue with our multiplication example:

~~~go
var answer = 3 * 0.333
~~~

The result of the multiplication will be a new constant of kind floating-point. That constant is then assigned to the variable answer through an implicit type conversion from kind floating-point to float64.

When we divide numeric constants, the kind of the constants determine how the division is performed.

~~~go
const third = 1 / 3.0
~~~

When one of the two constants are of kind floating-point, the result of the division will also be a constant of kind floating-point. In our example we have used a decimal point to represent the constant in the denominator. This follows the rules for kind promotion that we talked about before.

Let’s take the same example but use kind integer in the denominator:

~~~go
const zero = 1 / 3
~~~

This time we are performing division between two constants of kind integer. The result of the division will be a new constant of kind integer. Since dividing 3 into the value of 1 represents a number that is less than 1, the result of this division is constant 0 of kind integer.

Let’s create a typed constant using numeric constant arithmetic:

~~~go
type Numbers int8
const One Numbers = 1
const Two     = 2 * One
~~~

Here we declare a new type called Numbers with a base type of int8. Then we declare constant One with type Numbers and assign constant 1 of kind integer. Next we declare constant Two which is promoted to type Numbers through the multiplication of constant 2 of kind integer and constant One of type Numbers.

The declaration of constant Two shows an example of a constant getting promoted not just to a user-defined type, but a user-defined type associated with a base type.

**One Practical Example**
Let’s look at one practical example right from the standard library. The time package declares this type and set of constants:

~~~go
type Duration int64

const (
  Nanosecond Duration = 1
  Microsecond     = 1000 * Nanosecond
  Millisecond     = 1000 * Microsecond
  Second        = 1000 * Millisecond
)
~~~

All of the constants declared above are constants of type Duration which have a base type of int64. Here we are declaring typed constants using constant arithmetic between constants that are typed and untyped.

Since the compiler will perform implicit conversions for constants, we can write code in Go like this:

~~~go
package main

import (
  "fmt"
  "time"
)

const fiveSeconds = 5 * time.Second

func main() {
  now := time.Now()
  lessFiveNanoseconds := now.Add(-5)
  lessFiveSeconds := now.Add(-fiveSeconds)

  fmt.Printf("Now   : %v\n", now)
  fmt.Printf("Nano   : %v\n", lessFiveNanoseconds)
  fmt.Printf("Seconds : %v\n", lessFiveSeconds)
}

// Output:
// Now   : 2014-03-27 13:30:49.111038384 -0400 EDT
// Nano   : 2014-03-27 13:30:49.111038379 -0400 EDT
// Seconds : 2014-03-27 13:30:44.111038384 -0400 EDT
~~~

The power of constants are exhibited with the method calls to Add. Let’s look at the definition of the Add method for the receiver type Time:

~~~go
func (t Time) Add(d Duration) Time
~~~

The Add method accepts a single parameter of type Duration. Let’s look closer at the method calls to Add from our program:

~~~go
var lessFiveNanoseconds = now.Add(-5)
var lessFiveMinutes = now.Add(-fiveSeconds)
~~~

The compiler is implicitly converting constant -5 into a variable of type Duration to allow the method call to happen. Constant fiveSeconds is already of type Duration thanks to the rules for constant arithmetic:

~~~go
const fiveSeconds = 5 * time.Second
~~~

The multiplication between constant 5 and time.Second results in constant fiveSeconds becoming a constant of type Duration. This is because constant time.Second is of type Duration and this type is promoted when determining the type of the result. To support the function call, the constant still needs to be implicitly converted from a constant of type Duration to a variable of type Duration.

If constants didn’t behave the way they do, these kind of assignments and function calls would always require explicit conversions. Look at what happens when we try to use a value of type int to make the same method call:

~~~go
var difference int = -5
var lessFiveNano = now.Add(difference)

Compiler Error:
./const.go:16: cannot use difference (type int) as type time.Duration in function argument
~~~

Once we use a typed integer value as the parameter for the Add method call, we received a compiler error. The compiler will not allow implicit type conversions between typed variables. For that code to compile we would need to perform an explicit type conversion:

~~~go
Add(time.Duration(difference))
~~~

Constants are the only mechanism we have to write code without the need to use explicit type conversions.

Conclusion
We take the behavior of constants for granted, which is a testament to the language designers and those who have worked hard on this feature. A lot of work and care has gone into making constants work this way and the benefits are hopefully clear.

So the next time you are working with a constant, remember you are working with something that is unique. A hidden gem buried in the compiler that doesn’t get enough credit or recognition as a unique feature of Go. Constants help make coding in Go fun and the code we write readable and intuitive. While at the same time keeping the code we write type safe.

Thanks
Thanks to [Nate Finch](https://www.ardanlabs.com/broken-link) and [Kim Shrier](http://www.westryn.net/resumes/kim.html) who have provided several reviews of the post that have helped to make sure the content and examples were accurate, flowed well and would be interesting to Go developers. I was ready to give up a few times and Nate’s encouragement kept me going.

Special thanks to Robert Griesemer and others on the Go dev team for their time and patience in teaching me the subject matter. The Go dev team is filled with an amazing group of people who really care about the community and the people who are a part of it. Thanks!!