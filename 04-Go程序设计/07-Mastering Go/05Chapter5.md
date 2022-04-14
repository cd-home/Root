[TOC]

### Chapter5

#### 图与节点

> 图G(V,E)􏰓由一组有􏲜的，非􏳳的顶点集V(或者节点)以及一组􏵈E组 成
>
> 1. 有环图􏰓指其􏰢􏰛或者􏰛分顶 点集存在闭环
> 2. 在􏲏环中，没有闭环

#### 算法复杂度

> 1. 用大 O 表􏰠法􏰌描􏰎􏲑法的复杂􏱝
> 2. 优先使用内置数据结构

#### 二叉树

##### 定义

> 1. 􏰓每个节点最多有两个两个分􏳚的􏰑据􏰵􏰶
> 2. 树的深度 ，又􏳫为树􏱄的高􏱝，􏰓从􏱄的根节点􏱔所有节点的􏲮􏲯中最长􏳷的一个
> 3. 叶节点，没有子节点的节点
> 4. 平衡二叉树：一个􏱄的根节点􏱔任意两个叶节点的􏵊􏵋之􏴷不大于 1

##### 实现

~~~go
type Tree struct {
	Left  *Tree
	Value int
	Right *Tree
}
~~~

##### 优点

> 1. 平衡二叉树(搜索数)查找效率高，时间复杂度为logn
> 2. 在插入和删除情况下，需要维护树的平衡

#### 哈希表

> 1.  存储􏰑据􏰵􏱂一至多个􏰥键值对􏲈􏱒，􏰆使用一个哈希函数􏱌􏰑计􏲑出存存放正确值􏲗􏲈的􏵐桶或者槽的索引
> 2. 哈希函数应该将每个键分配到唯一的桶中
> 3. 哈希函数能输出均匀分布的哈希值，并且具有一致性􏴅􏳂
> 4. 哈􏱆表非􏰰􏳠合用于搭􏱯词􏴋或者其他这􏰬需要大􏰪􏰑据􏲄找的应用

##### 实现

~~~go
type Node struct {
	Value int
	Next  *Node
}
type HashTable struct {
	Table map[int]*Node
	Size  int
}
func hashFunction(i, size int) int {
	return i % size
}
~~~

#### 链表

##### 单链表

**定义**

> 1. 节点
> 2. 数据域
> 3. 指针域
> 4. 头、尾
> 5. 使用指􏰯进行顺序检索的速度􏱝非常􏰰快
> 6. 有序链表，为了􏲛持􏱅表的有􏰋性，必须􏰨􏲨保证节点􏳜放到􏱔正确􏲗的位置【可以通过手段优化】􏱃

**实现**

~~~go
var root = new(LinkNode)
type LinkNode struct {
	Value int
	Next  *LinkNode
}
~~~

##### 双向链表

> 1. 每个节点既􏱬􏴹有指向前一个节点的指针，又有指向下一个元素的指针
> 2. 两个方向访问, 更加容易增加删除

~~~go
var head = new(DoubleLinkedNode)
type DoubleLinkedNode struct {
	Value int
	Pre   *DoubleLinkedNode
	Next  *DoubleLinkedNode
}
~~~

#### 队列

> 1. 先进先出特点
> 2. 可用数组、链表实现

~~~go
type QNode struct {
	Value int
	Next *QNode
}
var QSize = 0
var queue = new(QNode)
~~~

#### 栈

> 1. 先进后出特点
> 2. 可用数组、链表实现

~~~go
type SNode struct {
	Value int
	Next *SNode
}
var SSize = 0
var stack = new(SNode)
~~~

#### container

> 1. heap
> 2. list
> 3. ring