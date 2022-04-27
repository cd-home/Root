[TOC]

### 5. Software Design

#### 5.1 Group Different Types of Data 分组不同类型的数据

It’s important to remember that in Go the **concepts** of **sub-typing** or **sub-classing** really don't exist and these **design patterns** should be avoided

重要的是要记住，**子类型**或者**子类**的概念在Go中是不存在的， 应该避免这些**设计模式**。

The following is an **anti-pattern** I shouldn’t follow or **implement**

以下是一个我们不应该遵循或者**实施**的**反模式**

~~~go
type Animal struct {
		Name     string
		IsMammal bool
}
~~~

The Animal type is being **declared** as a **base type** that tries to define data that is common to all animals. I also attempt to provide some common **behavior** to an animal as well.

Animal 类型被**声明**为尝试定义所有动物共有的数据的**基本类型**，我也尝试为动物提供一些常见的**行为**。

~~~go
func (a *Animal) Speak() {
	fmt.Println("UGH!", 
              "My Name is", a.Name, 
              ", it is", a.IsMammal, 
              ", I am a mammal")
}
~~~

Most animals have the ability to speak in one way or the other. However, trying to apply this common behavior to just an animal doesn’t make any sense. At this point, I have no idea what sound this animal makes, so I wrote UGH.

大多数动物都有以一种或另一种方式说话的能力。 然而，试图将这种常见行为仅应用于动物是没有任何意义的。 在这一点上，我不知道这只动物发出什么声音，所以我写了 UGH。

~~~go
type Dog struct {
	Animal
	PackFactor int
}
~~~

Now the real problems begin. I’m attempting to use **embedding** to make a Dog everything an Animal is plus more. On the surface this will seem to work, but there will be problems. With that being said, a Dog does have a specific way they speak.

现在真正的问题开始了。 我正在尝试使用**嵌入**来使 Dog 成为一个有更多内容的Animal。 从表面上看，这似乎可行，但会有问题。 话虽如此，狗确实有特定的说话方式。

~~~go
func (d *Dog) Speak() {
	fmt.Println("Woof!", 
              "My Name is", d.Name,
              ", it is", d.IsMammal, 
              ", I am a mammal", 	d.PackFactor)
}
~~~

In the **implementation** of the Speak method, I can change out UGH for Woof. This is specific to how a dog speaks.

在 Speak 方法的**实现**中，我可以将 UGH 更改为 Woof。 这是狗特定的说话方式。

```go
type Cat struct {
	Animal
	ClimbFactor int
}
```

If I’m going to have a Dog that represents an Animal, then I have to have a Cat. Using embedding, a Cat is everything an Animal is plus more.

如果我要养一只代表动物的狗，那么我必须养一只猫。通过嵌入，猫就是动物的一切，再加上更多。

```go
func (c *Cat) Speak() {
	fmt.Println("Meow", 
              "My Name is", c.Name, 
              ", it is", c.IsMammal, 
              ", I am a mammal", c.ClimbFactor)
}
```

In the implementation of the Speak method, I can change out UGH for Meow. This is specific to how a cat speaks.

在 Speak 方法的实现中，我可以把 UGH 换成 Meow。 这是猫特定说话的方式。

Everything seems fine and it looks like embedding is providing the same **functionality** as **inheritance** does in other languages. Then I try to go ahead and group dogs and cats by the fact they have a common DNA of being an Animal.

一切看起来都很好，看起来嵌入提供了与其他语言中的**继承**相同的**功能**。 然后，我尝试将狗和猫归为一类，因为它们具有共同的动物 DNA。

~~~go
animals := []Animal{ 
			Dog{
				Animal: Animal{
					 Name: "Fido",
					  IsMammal: true,
				}, PackFactor: 5,
			},
			Cat{
				Animal: Animal{ 
					Name: "Milo",
					IsMammal: true,
				},
			ClimbFactor: 4, },
	}
	for _, animal := range animals {
		 animal.Speak()
	}
~~~

When I try to do this, the compiler complains that **a Dog and Cat are not an Animal** and this is true. **Embedding isn’t the same as inheritance and this is the pattern I need to stay away from**. A Dog is a Dog, a Cat a Cat, and an Animal an Animal. I can’t pass Dog’s and Cat’s around as if they are Animal’s because they are not.

嵌入与继承不同，我们应该避免的设计模式。

This kind of mechanic is also not very flexible. It requires configuration by the developer and this isn’t very flexible unless I have access to the code and can make configuration changes over time.

这种机制也不是很灵活。 它需要开发人员进行配置，这不是很灵活，除非我可以访问代码并且可以随着时间的推移进行配置更改。

If this is not how we can **construct** a collection of Dog’s and Cat’s, how can we do this in Go? It’s not about grouping through common DNA, **it’s about grouping through common behavior**. **Behavior is the key**.

如果这不是我们**构建** Dog's 和 Cat's 集合的方法，那么我们如何在 Go 中做到这一点？ 这不是通过共同的 DNA 进行分组，而是**通过共同的行为进行分组**。 **行为是关键**。

~~~go
type Speaker interface {
		Speak()
}
~~~

If I use an **interface**, then I can define the common method set of behavior that I want to group different types of data against.

如果我使用一个**接口**，那么我可以定义我想要对不同类型的数据进行分组的行为的公共方法集。

~~~go
speakers := []Speaker{ 
			&Dog{
				Animal: Animal{
					 Name: "Fido",
					 IsMammal: true,
				}, PackFactor: 5,
			},
			&Cat{
				Animal: Animal{ 
					Name: "Milo",
					IsMammal: true,
				},
			ClimbFactor: 4, },
	}
	for _, speaker := range speakers {
		 speaker.Speak()
	}
~~~

In the new code, I can now group Dogs and Cats together based on their common set of behavior, which is the fact that Dogs and Cats can speak.

在新代码中，我现在可以根据狗和猫的共同行为集将它们分组在一起，这就是狗和猫可以说话的事实。

In fact, the Animal type is really **type pollution** because declaring a type just to share a set of **common state** is a smell and should be avoided.

事实上，Animal 类型确实是**类型污染**，因为声明一个类型只是为了共享一组**公共状态**，应该避免。

~~~go
type Dog struct {
		Name       string
		IsMammal   bool
		PackFactor int
}
type Cat struct {
		Name        string
		IsMammal    bool
		ClimbFactor int
}
~~~

In this particular case, I would rather see the Animal type removed and the fields copied and pasted into the Dog and Cat types. Later I will have notes about better patterns that **eliminate** these scenarios from happening.

在这种特殊情况下，我宁愿看到 Animal 类型被删除，字段被复制并粘贴到 Dog 和 Cat 类型中。 稍后我将记录关于**消除**这些情况发生的更好模式的注释。

Here are the code smells from the original code: 原始代码的问题

1. The Animal type provides an abstraction layer of reusable state

   Animal 类型提供了可重用状态的抽象层

2. The program never needs to create or solely use a value of Animal type

   程序永远不需要创建或单独使用 Animal 类型的值

3. The implementation of the Speak method for the Animal type is generalized

   Animal 类型的 Speak 方法的实现被泛化

4. The Speak method for the Animal type is never going to be called.

   Animal 类型的 Speak 方法永远不会被调用

Guidelines around declaring types: 关于声明类型的指南

1. Declare types that represent something new or unique

   声明代表新事物或独特事物的类型

2. Don't create aliases just for readability

   不要仅仅为了可读性而创建别名

3. Validate that a value of any type is created or used on its own

   验证任何类型的值是自行创建或使用的

4. **Embed types not because I need the state, but because we need the behavior**

   **嵌入类型不是因为我需要状态，而是因为我们需要行为**

5. If I am not thinking about behavior, I’m locking myself into the design that I can’t grow in the future without cascading code changes

   如果我不考虑行为，我会将自己锁定在不进行级联代码更改的情况下我无法在未来发展的设计中

6. Question types that are aliases or abstractions for an existing type

7. Question types whose sole purpose is to share common state

#### 5.2 Don’t Design With Interfaces 不要使用接口进行设计

Unfortunately, too many developers attempt to solve problems in the abstract first. They focus on interfaces right away and this leads to interface pollution. As a developer, I exist in one of two modes: a **programmer** and then an **engineer**.

不幸的是，太多的开发人员试图先抽象地解决问题。 他们立即关注接口，这会导致接口污染。 作为开发人员，我存在两种模式之一：**程序员**，然后是**工程师**。

When I am programming, I am focused on getting a piece of code to work. Trying to solve the problem and break down walls. Prove that my initial ideas work. That is all I care about. This programming must be done in the concrete and is never production ready.

**当我编程时，我专注于让一段代码工作。 试图解决问题。 证明我最初的想法是可行的。 这就是我所关心的。 这种编程必须具体完成，并且永远不会为生产做好准备**。

Once I have a prototype of code that solves the problem, then I need to switch to engineering mode. I need to focus on how to write the code at a **micro-level for my data semantics**, **macro-level for mental models**, readability, and maintainability. I need to think about errors and fail states. So much more.

一旦我有了解决问题的代码原型，我就需要切换到工程模式。 我需要专注于如何在**微观层面**为我的数据语义编写代码，在**宏观层面**为心智模型、可读性和可维护性编写代码。 我需要考虑错误和失败状态。 等。

This work is done in a cycle of **refactoring**. Refactoring for readability, efficiency, abstraction, and for testability. Abstracting is one of a few refactors that need to be performed. This works best when I start with a piece of **concrete** code and then DISCOVER the interfaces that are needed. Don’t apply abstractions unless they are absolutely necessary.

这项工作是在一个**重构**周期中完成的。 为可读性、效率、抽象和可测试性进行重构。 抽象是需要执行的少数重构之一。 当我从一段**具体**代码开始，然后发现所需的接口时，这种方法效果最好。 除非绝对必要，否则不要应用抽象。

Every problem I solve with code is a data problem requiring me to write data transformations. If I don’t understand the data, I don’t understand the problem. If I don’t understand the problem, I can’t write any code. **Starting with a concrete solution that is based on the concrete data structures is critical.**

我用代码解决的每个问题都是需要我编写数据转换。 如果我不了解数据，我就不了解问题所在。 如果我不理解问题，我就无法编写任何代码。 从基于具体数据结构的具体解决方案开始至关重要。

As Rob Pike said,

"Data dominates. If you've chosen the right data structures and organized things well, the algorithms will almost always be self-evident"

数据占主导地位。如果你选择了正确的数据结构，并且组织得很好，那么算法几乎总是不言而喻的

When is abstraction necessary? When I see a place in the code where the data could change and I want to **minimize the cascading code effects** that would result. I might use abstraction to help make code testable but I should try to avoid this if possible. The best testable functions are functions that take raw data in and send raw data out. It shouldn’t matter where the data is coming from or going.

什么时候需要抽象？当我看到代码中某个地方的数据可能会发生变化，我想尽量减少可能产生的级联代码效应。我可能会使用抽象来帮助代码可测试，但如果可能的话，我应该尽量避免这种情况。最好的可测试函数是接收原始数据并发送原始数据的函数。数据从哪里来或去哪里都不重要。

In the end, start with a concrete solution to every problem. Even if the bulk of that is just programming. Then discover the interfaces that are absolutely required for the code today.

最后，从每个问题的具体解决方案开始。 即使其中大部分只是编程。 然后发现今天的代码绝对需要的接口。

Don’t design with interfaces, discover them

不要用接口进行设计，要发现它们

#### 5.3 Composition 组合

The best way to take advantage of embedding is through the **compositional design pattern**. The idea is to compose larger types from smaller types and focus on the composition of behavior.

利用嵌入的最佳方式是通过**组合设计模式**。 这个想法是从较小的类型组合更大的类型，并专注于行为的组合。

