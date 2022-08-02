[TOC]

### Introduction

It’s important that you prepare your mind for the material you are about to read. The introduction will provide some thoughts and ideas, using quotes to stimulate your initial understanding of the language. 

为即将阅读的材料做好准备是很重要的. 引言将提供一些` thoughts and ideas`,使用引号来促进您对该语言的初步理解.

Somewhere Along The Line (曾几何时)

- [ ] We became impressed with programs that contain large amounts of code. 

    我们对包含大量代码的程序敬仰

- [ ] We strived to create large abstractions in our code base. 

    我们努力在代码库中创建大型抽象

- [ ] We forgot that the hardware is the platform. 

    我们忘记了硬件就是平台

- [ ] We lost the understanding that every decision comes with a cost. 

    我们失去了每一个决定都要付出代价的认识

These Days Are Gone 这些日子一去不复返了

- [ ] We can throw more hardware at the problem. 我们可以在这个问题上投入更多的硬件.
- [ ] We can throw more developers at the problem. 我们可以让更多的开发者来解决这个问题.

Open Your Mind 开放你的思想

- [ ] Technology changes quickly but people's minds change slowly. 

    技术变化很快, 但人们的思想变化很慢.

- [ ] Easy to adopt new technology but hard to adopt new ways of thinking. 

    易于采用新技术, 但难以采用新的思维方式.

Interesting Questions - What do they mean to you? 

- [ ] Is it a good program? 
- [ ] Is it an efficient program? 
- [ ] Is it correct? 
- [ ] Was it done on time? 
- [ ] What did it cost? 

Aspire To 追求、向往

- [ ] **Be a champion for quality, efficiency and simplicity. 成为质量、效率和简单性的捍卫者.** 
- [ ] **Have a point of view. 有自己的观点**
- [ ] **Value introspection and self-review. 价值观反思与自我检讨**

#### 1.1 Reading Code 																								(阅读代码)

Go is about being a language that focuses on code being readable.

Go是一种专注于代码可读性的语言.

**Quotes**

*"If most computer people lack understanding and knowledge, then what they will select* 

*will also be lacking."- Alan Kay* 

*"The software business is one of the few places we teach people to write before we* 

*teach them to read." - Tom Love (inventor of Objective C)*

*"Code is read many more times than it is written." - Dave Cheney*

*"Programming is, among other things, a kind of writing. One way to learn writing is to write, but in all other forms of writing, one also reads. We read examples both good and bad to facilitate learning. But how many programmers learn to write programs by reading programs?" - Gerald M. Weinberg*

"除其他外, 编程是一种写作. 学习写作的一种方法是写作, 但在所有其他形式的写作中, 也有阅读. 我们阅读好的和坏的例子来促进学习. 但是有多少程序员学习编写程序是通过阅读程序?"

*"Skill develops when we produce, not consume." - Katrina Owen*

#### 1.2 Legacy Software  																						(遗留软件)

Do you care about the legacy you are leaving behind?

**Quotes** 

*"There are two kinds of software projects: those that fail, and those that turn into* 

*legacy horrors." - Peter Weinberger (inventor of AWK)* 

"有两种软件项目: 一种是失败的, 另一种是变成遗留问题的. "

*"Legacy software is an unappreciated but serious problem. Legacy code may be the* 

*downfall of our civilization." - Chuck Moore (inventor of Forth)* 

"遗留软件是一个未被重视但严重的问题. 遗留代码可能是我们文明的垮台". 

*"Few programmers of any experience would contradict the assertion that most programs are modified in their lifetime. Why then do we rarely find a program that contains any evidence of having been written with an eye to subsequent modification."- Gerald M. Weinberg*

**几乎没有任何有经验的程序员会反驳大多数程序在其一生中都会被修改的说法. 那么, 为什么我们很少发现一个程序包含任何证据表明其编写时考虑到了后续修改.**

*"We think awful code is written by awful devs. But in reality, it's written by reasonable* *devs in awful circumstances." - Sarah Mei* 

"我们认为糟糕的代码是由糟糕的开发人员编写的. 但实际上,它是由合理的开发人员在糟糕的情况下编写的."

"*There are many reasons why programs are built the way they are, although we may* *fail to recognize the multiplicity of reasons because we usually look at code from the* *outside rather than by reading it. When we do read code, we find that some of it gets* *written because of machine limitations, some because of language limitations, some* *because of programmer limitations, some because of historical accidents, and some* *because of specifications—both essential and inessential." - Gerald M. Weinberg* 

程序按照它们的方式构建的原因有很多,尽管我们可能无法识别出多种原因,因为我们通常从外部而不是通过阅读来查看代码. 当我们确实阅读代码时,我们发现其中一些是因为机器限制而编写的,一些是因为语言限制，一些是因为程序员的限制, 一些是因为历史事故, 还有一些是因为规范——无论是必要的还是非必要的. 

#### 1.3 Mental Models  																							(思维/心智模型)

You must constantly make sure your mental model of your projects are clear. When you can't remember where a piece of logic is or you can't remember how something works, you are losing your mental model of the code. This is a clear indication that refactoring is a must. Focus time on structuring code that provides the best mental model possible and code review for this as well. 

你必须不断地确保你的项目思维模型是清晰的.  当你不记得一段逻辑在哪里或者你不记得某事是如何工作的时, 你就失去了代码的思维模型.  这清楚地表明重构是必须的. 将时间集中在构建可提供最佳思维模型的代码上, 并为此进行代码审查. 

How much code in that box do you think you can maintain a mental model of in your head? I believe asking a single developer to **maintain** a mental model of more than one ream of paper in that box (~10k lines of code) is asking a lot. If you do the math, then it takes a team of 100 people to work on a code base that hits a million lines of code. That is 100 people that need to be coordinated, grouped, tracked and in a constant feedback loop of communication.

你认为这个盒子里有多少代码可以在你的脑海中**维持**一个心智模型?我认为,要求一个开发人员在那个盒子里维护一个包含超过一令纸(约10k行代码)的心智模型要求很高. 如果你做数学运算, 那么需要一个由100人组成的团队来处理一个达到一百万行代码的代码库. 也就是说, 有100人需要协调、分组、跟踪, 并处于一个不断反馈的沟通循环中.

**Quotes** 

*"Let's imagine a project that's going to end up with a million lines of code or more. The* *probability of those projects being successful in the United States these days is very* *low - well under 50%. That's debatable." - Tom Love (inventor of Objective C)* 

"让我们想象一个项目最终将有 100 万行或更多代码. 这些项目如今在美国成功的可能性非常低 - 远低于 50%. 这是值得商榷的".

*"100k lines of code fit inside a box of paper." - Tom Love (inventor of Objective C)* 

"10 万行代码可以放在一盒纸里."

"*One of our many problems with thinking is 'cognitive load': the number of things we* *can pay attention to at once. The cliche is 7±2, but for many things it is even less. We* *make progress by making those few things be more powerful." - Alan Kay* 

"我们思考的许多问题之一是'认知负荷'：即我们一次可以关注的事情的数量. 陈词滥调是7±2, 但在许多情况下甚至更少. 我们取得进步通过关注的更少有效的事情."

*"The hardest bugs are those where your mental model of the situation is just wrong, so* *you can't see the problem at all." - Brian Kernighan* 

"最难的错误的情况是那些你的心理模型是错误的, 所以你根本看不到问题所在. " - 布赖恩·克尼汉

*"Everyone knows that debugging is twice as hard as writing a program in the first* *place. So if you're as clever as you can be when you write it, how will you ever debug* *it?" - Brian Kernighan* 

"每个人都知道调试的难度是编写程序的两倍. 因此, 如果你在编写程序时尽可能聪明, 你将如何调试它?."

*"Debuggers don't remove bugs. They only show them in slow motion." - Unknown* 

*"Fixing bugs is just a side effect. Debuggers are for exploration." - @Deech (Twitter)* 

"修复bug只是一个副作用, 调试器是用来探索的."

#### 1.4 Productivity vs Performance  																(生产力 vs 性能)

Productivity and performance both matter, but in the past you couldn’t have both. You needed to choose one over the other. We naturally gravitated to productivity, with the idea or hope that the hardware would resolve our performance problems for free. This movement towards productivity has resulted in the design of programming languages that produce sluggish software that is outpacing the hardware’s ability to make them faster.

生产力和性能都很重要, 但在过去你不可能两者兼得. 你需要选择一个.  我们自然而然地被生产力所吸引, 有这样的想法或希望, 硬件将免费解决我们的性能问题. 这种向生产力发展的趋势导致了编程语言的设计, 这些语言产生的软件速度较慢, 失去了硬件使其更快的能力. 

By following Go’s idioms and a few guidelines, we can write code that can be reasoned about by anyone who looks at it. We can write software that simplifies, minimizes and reduces the amount of code we need to solve the problems we are working on. We don’t have to choose productivity over performance or performance over productivity anymore. We can have both. 

通过遵循Go的习惯用法和一些指导原则, 我们可以编写任何人都可以理解的代码. 我们可以编写简化、最小化和减少解决我们正在研究的问题所需的代码量的软件. 我们不再需要选择生产力而不是性能, 或者选择性能而不是生产力. 我们可以两者兼得. 

**Quotes** 

*"The hope is that the progress in hardware will cure all software ills. However, a critical observer may observe that software manages to outgrow hardware in size and sluggishness. Other observers had noted this for some time before, indeed the trend was becoming obvious as early as 1987." - Niklaus Wirth*

"我们希望硬件的进步能够治愈所有的软件弊病. 然而, 一位批判性的观察者可能会观察到, 软件在规模和缓慢性方面都超过了硬件. 其他观察者之前已经注意到了这一点, 事实上, 早在1987年, 这一趋势就变得很明显."

*"The most amazing achievement of the computer software industry is its continuing cancellation of the steady and staggering gains made by the computer hardware industry." - Henry Petroski (2015)*

"计算机软件行业最令人惊讶的成就是它不断地取消了计算机硬件行业所取得的稳定和惊人的成就."

*"The hardware folks will not put more cores into their hardware if the software isn’t going to use them, so, it is this balancing act of each other staring at each other, and we are hoping that Go is going to break through on the software side.” - Rick Hudson (2015)*

"如果软件不使用它们, 硬件人员就不会在硬件中放入更多内核, 因此, 这是一种相互凝视的平衡行为, 我们希望Go能够在软件方面取得突破."

*"C is the best balance I've ever seen between power and expressiveness. You can do almost anything you want to do by programming fairly straightforwardly and you will have a very good mental model of what's going to happen on the machine; you can predict reasonably well how quickly it's going to run, you understand what's going on." - Brian Kernighan (2000)*

"C是我见过的力量和表现力之间的最佳平衡. 通过相当直接的编程, 你几乎可以做任何你想做的事情, 你会对机器上发生的事情有一个很好的心理模型; 你可以很好地预测它的运行速度, 你知道发生了什么. "

*"The trend in programming language design has been to create languages that enhance software reliability and programmer productivity. What we should do is develop languages alongside sound software engineering practices so the task of developing reliable programs is distributed throughout the software lifecycle, especially into the early phases of system design." - Al Aho (2009)*

"编程语言设计的趋势是创建能够提高软件可靠性和程序员生产力的语言. 我们应该做的是在良好的软件工程实践中开发语言, 以便开发可靠程序的任务分布在整个软件生命周期中, 尤其是在系统设计的早期阶段."

#### 1.5 Correctness vs Performance  																(正确性与性能)

You want to write code that is optimized for correctness. Don't make coding decisions based on what you think might perform better. You must benchmark or profile to know if code is not fast enough. Then and only then should you optimize for performance. This can't be done until you have something working. 