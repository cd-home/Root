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



