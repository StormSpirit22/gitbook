# 栈和队列

队列是一种先进先出的数据结构，栈是一种先进后出的数据结构。

本文要介绍的题目有：

[232. 用栈实现队列（简单）](https://leetcode-cn.com/problems/implement-queue-using-stacks)

[225. 用队列实现栈（简单）](https://leetcode-cn.com/problems/implement-stack-using-queues)

[155. 最小栈（简单）](https://leetcode-cn.com/problems/min-stack/)

## 题目解析

### 用栈实现队列

> 请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（push、pop、peek、empty）：
>
> 实现 MyQueue 类：
>
> void push(int x) 将元素 x 推到队列的末尾 int pop() 从队列的开头移除并返回元素 int peek() 返回队列开头的元素 boolean empty() 如果队列为空，返回 true ；否则，返回 false 说明：
>
> 你 只能 使用标准的栈操作 —— 也就是只有 push to top, peek/pop from top, size, 和 is empty 操作是合法的。 你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。

解析：

push 方法：我们有 2 个栈 stack1 和 stack2，可以先将 stack1 的数据全部出栈到 stack2，比如 stack1 的数据是 \[3,2,1]，到了 stack2 里就是 \[1,2,3]，这时再往 stack2 入栈一个 4，此时 stack2 是 \[1,2,3,4]，然后再将 stack2 的数据全部出栈到 stack1，stack1 的数据就是 \[4,3,2,1]，出栈的顺序就是队列的顺序了。其实就是两个栈互相倒数据，把数据顺序变得和 push 的顺序相反就行。

只要完成 push 方法，stack1 就是我们的主栈，pop、top、empty 方法都只要对 stack1 操作就行了。

### 用队列实现栈

> 请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（push、top、pop 和 empty）。
>
> 实现 MyStack 类：
>
> void push(int x) 将元素 x 压入栈顶。 int pop() 移除并返回栈顶元素。 int top() 返回栈顶元素。 boolean empty() 如果栈是空的，返回 true ；否则，返回 false 。
>
> 注意：
>
> 你只能使用队列的基本操作 —— 也就是 push to back、peek/pop from front、size 和 is empty 这些操作。 你所使用的语言也许不支持队列。 你可以使用 list （列表）或者 deque（双端队列）来模拟一个队列 , 只要是标准的队列操作即可。

解析：

这题用两个队列或者一个队列都能实现，两种思路其实差不多，这里只介绍两个队列的方法。

push 方法：我们有 2 个队列 queue1 和 queue2，假设 queue1 作为主列有数据，queue2 没数据作为辅助队列，可以先将来的数据入队 queue2，然后将 queue1 的数据全部出队到 queue2，然后 queue1 与 queue2 交换即可。比如 queue1 有数据 \[3,2,1]，此时 queue2 入队 4，然后将 queue1 的数据全部出队到 queue2，则 queue2 为 \[4,3,2,1]，然后将两个队列交换，即 queue1 还是主队列有数据，queue2 还是辅助队列没有数据。这里思想也是保证队列里的数据顺序与 push 的顺序相反即可，这样就能后入队列的先出了。

同样完成 push 方法，其他的方法只要对 queue1 操作即可。

### 最小栈

> 设计一个支持 push ，pop ，top 操作，并能在常数时间内检索到最小元素的栈。
>
> push(x) —— 将元素 x 推入栈中。 pop() —— 删除栈顶的元素。 top() —— 获取栈顶元素。 getMin() —— 检索栈中的最小元素。

解析：

每次都要获取栈中的最小元素，那么我们用一个数组来表示最小栈，每次 push 的时候都把当前最小元素如 x， 与要 push 的元素如 y 比较，如果 x < y ，那么最小栈里压入 y 即当前最小值变成 y 了，否则继续压入 x，即当前的最小值还是 x。只要想通这一点这题就很简单了。

## 代码

### 用栈实现队列

```go
type MyQueue struct {
	stack1 []int
	stack2 []int
}


func Constructor() MyQueue {
	return MyQueue{stack1: []int{}, stack2: []int{}}
}


func (this *MyQueue) Push(x int)  {
	for len(this.stack1) > 0 {
		n := len(this.stack1)
		x := this.stack1[n-1]
		this.stack2 = append(this.stack2, x)
		this.stack1 = this.stack1[:n-1]
	}
	this.stack2 = append(this.stack2, x)

	for len(this.stack2) > 0 {
		n := len(this.stack2)
		x := this.stack2[n-1]
		this.stack1 = append(this.stack1, x)
		this.stack2 = this.stack2[:n-1]
	}
}


func (this *MyQueue) Pop() int {
	n := len(this.stack1)
	x := this.stack1[n-1]
	this.stack1 = this.stack1[:n-1]
	return x
}


func (this *MyQueue) Peek() int {
	return this.stack1[len(this.stack1)-1]
}


func (this *MyQueue) Empty() bool {
	return len(this.stack1) == 0
}
```

### 用队列实现栈

双队列实现：

```go
type MyStack struct {
    queue1, queue2 []int
}

/** Initialize your data structure here. */
func Constructor() (s MyStack) {
    return
}

/** Push element x onto stack. */
func (s *MyStack) Push(x int) {
    s.queue2 = append(s.queue2, x)
    for len(s.queue1) > 0 {
        s.queue2 = append(s.queue2, s.queue1[0])
        s.queue1 = s.queue1[1:]
    }
    s.queue1, s.queue2 = s.queue2, s.queue1
}

/** Removes the element on top of the stack and returns that element. */
func (s *MyStack) Pop() int {
    v := s.queue1[0]
    s.queue1 = s.queue1[1:]
    return v
}

/** Get the top element. */
func (s *MyStack) Top() int {
    return s.queue1[0]
}

/** Returns whether the stack is empty. */
func (s *MyStack) Empty() bool {
    return len(s.queue1) == 0
}
```

单队列实现：

```go
type MyStack struct {
	queue []int
}


func Constructor() MyStack {
	stack := MyStack{
		queue: []int{},
	}
	return stack
}


func (this *MyStack) Push(x int)  {
    this.queue = append(this.queue, x)
    n := len(this.queue)

    for n > 1 {
        x := this.queue[0]
        this.queue = append(this.queue, x)
        this.queue = this.queue[1:]
        n--
    }
}


func (this *MyStack) Pop() int {
	x := this.queue[0]
    this.queue = this.queue[1:]
    return x
}

func (this *MyStack) Top() int {
	return this.queue[0]
}


func (this *MyStack) Empty() bool {
	if len(this.queue) == 0 {
		return true
	}
	return false
}
```

### 最小栈

```go
type MinStack struct {
	stack []int
	minStack []int
}


func Constructor() MinStack {
	return MinStack{stack: []int{}, minStack: []int{}}
}


func (this *MinStack) Push(val int)  {
	x := this.GetMin()
	if x > val {
		this.minStack = append(this.minStack, val)
	} else {
		this.minStack = append(this.minStack, x)
	}
	this.stack = append(this.stack, val)
}


func (this *MinStack) Pop()  {
	n := len(this.stack)
	this.stack = this.stack[:n-1]
	this.minStack = this.minStack[:n-1]
}


func (this *MinStack) Top() int {
	n := len(this.stack)
	x := this.stack[n-1]
	return x
}


func (this *MinStack) GetMin() int {
	if len(this.minStack) > 0 {
		return this.minStack[len(this.minStack)-1]
	}
	return math.MaxInt32
}
```
