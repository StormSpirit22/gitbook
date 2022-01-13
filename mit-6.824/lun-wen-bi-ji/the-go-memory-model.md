# The Go Memory Model

原文 [The Go Memory Model](https://golang.org/ref/mem)

翻译转载并修改自 [\[译\] Go References - The Go Memory Model | golang官方文档中文翻译之内存模型](https://www.pengrl.com/p/34119/)

## 前言

> 本篇译文对应的原文 标题：The Go Memory Model - Go References 作者：Go官方文档 地址：https://golang.org/ref/mem

## 目录

* 简介
* 建议
* Happens Before
* 同步
  * init function
  * 创建协程
  * 销毁协程
  * 使用channel通信
  * 锁
  * Once
* 示范错误的同步原语使用方法

## 简介

golang的内存模型指定了想要达到以下这种效果所依赖的条件。达到什么效果？即在一个协程中修改一个变量，在另外一个协程读取这个变量时需保证读取到修改后的值。

## 建议

程序如果修改被多个协程同时访问的数据，那么必须串行化这些访问操作。

为了保证串行化访问，可以使用golang的channel操作或者使用sync和sync/atomic包中的同步原语来保护数据。

如果你需要通过阅读这个文档的剩余部分才能搞明白你程序的行为，那么你有些太聪明了。

额，不要聪明过头了。

## Happens Before

在一个协程内，读写操作的真正执行顺序必须保证它们所表现出的行为和程序中指定的顺序一致。也就是说，编译器和处理器只会在不改变这个协程内的程序语意的前提下重排序这个协程内的读写操作。由于存在这种重排序，一个协程观察到的执行顺序可能会和另一个协程观察到的不同。举例来说，如果一个协程执行了`a=1; b=2;`，另一个协程可能观察到的是b变量的更新发生在a变量的更新之前。

为了说明对读写操作的要求，我们定义了`happens before`，一种golang程序对内存操作的局部执行顺序。如果事件e1`happens before`事件e2，那么我们说e2`happens after`e1。同样的，如果e1不`happens before`e2并且e1也不`happens after`e2，那么我们说e1和e2`happens concurrently`。

```
备注：
happens before定义的这种执行顺序，是一种确认的、唯一性的执行顺序。
即如果我说a happens before b，
那么a就必定是发生在b之前的，不允许出现a发生在b之后或a和b同时发生的情况出现。
换句话说，如果存在a发生在b之后或a和b同时发生的情况，那么就不能说a happens before b。

not happens before，这个需要特别注意。
如果我说a not happens before b，
那么有可能是a发生在b之后；也有可能是a和b同时发生。
只要不是a发生在b之前就行。

happens after就是happens before说法的主语和宾语调换位置后的另一种说法，这个没什么好解释的。
```

在单个协程内，`happens before`顺序就是程序中所描述的顺序。

对变量v的读操作r **allow**观察到对变量v的写操作w`（备注：这里的allow是允许的意思，即有可能观察到，也可能观察不到）`，需同时满足以下条件：

1. r`not happens before`w
2. 没有其它的`happens after`w并且`happens before`r的写操作

如果要保证对变量v的读操作r要观察到特定的对变量v的写操作w，即w是r唯一允许被观察到的写操作。简单来说，要保证r观察到w，需同时满足以下条件：

1. w`happens before`r
2. 其它对共享变量v的写操作要么`happens before`w，要么`happens after`r

这组条件的限定要比第一组的条件限定更严格些。它要求了没有其它的写操作和w/r`happens concurrently`。

在一个协程中，由于没有并发，所以这两种定义是相同的：读操作r可以观察到最近一次的写操作w。如果是多协程访问一个共享变量。那么必须用同步事件来建立起`happens before`的条件以保证读操作观察到期望的写操作。

对变量v的初始化操作，其行为和在内存模型中做一次写操作是一样的。

对变量超过一个机器字长大小的读写操作，其行为和多个单机器字长大小的操作一样，是一种未指定的顺序。

## 同步

### init function

golang程序中的所有init function运行在同一个协程中，但是这个协程可能会创建其它的协程，而这些协程可能会并发运行。

如果p包内引入了q包，那么q的init函数将完全执行完后再开始执行p的init function。

程序的入口函数`main.main`在所有init function都执行完后再执行。

### 创建协程

开启一个新协程`happens before`这个新协程的执行入口处。

比如下面这个程序：

```
var a string

func f() {
	print(a)
}

func hello() {
	a = "hello, world"
	go f()
}
```

调用hello函数后会在未来的某个时间点打印”hello, world”，这个时间点有可能是hello函数执行结束以后。

### 销毁协程

协程退出并不保证`happens before`程序中的任何事件。比如下面这个程序：

```
var a string

func hello() {
	go func() { a = "hello" }()
	print(a)
}
```

赋值语句并没有和任何同步事件相结合，所以并不能保证这个赋值语句被任何其他协程观察到。事实上，一个激进的编译器可能把整个协程语句都删除掉。

如果一个协程造成的影响需要被其他协程观察到，需使用锁或channel等同步机制来建立一个关联顺序。

### 使用channel通信

使用channel通信是多协程同步的主要方法。每次往一个特定的channel发送都和一个相关联的从channel接收相匹配，一般发送和接收在不同的协程上。

**往带缓冲的channel发送`happens before`从该channel完成接收。**`（备注：即channel的接收处会阻塞，直到其他协程往channel发送了数据）`

这个程序：

```
var c = make(chan int, 10)
var a string

func f() {
	a = "hello, world"
	c <- 0
}

func main() {
	go f()
	<-c
	print(a)
}
```

会保证打印”hello, world”。对a的写入`happens before`往channel c发送，往channel c发送`happens before`从channel c完成接收，从channel c完成接收`happens before`打印。

关闭一个channel `happens before`从channel的接收处返回0值。

在前面的例子，将`c <- 0`替换成`close(c)`，程序会保证相同的行为。

**从无缓冲channel接收`happens before`往该channel完成发送。**`（备注：即channel的发送处会阻塞，直到其他协程执行到从channel读取数据）`

这个程序（和上面的程序相比，交换了发送和接收的语句并且使用了无缓存channel）：

```
var c = make(chan int)
var a string

func f() {
	a = "hello, world"
	<-c
}
func main() {
	go f()
	c <- 0
	print(a)
}
```

同样会保证打印”hello, world”。写入a变量`happens before`从c接收，从c接收`happens before`往c完成发送，往c完成发送`happens before`打印。

如果channel是有缓冲的（比如，`c = make(chan int, 1)`），那么程序不能保证打印出”hello, world”。（可能会打印出空字符串，崩溃或者其他情况）。

第k次从初始化空间为C的channel的接收`happens before`第k+C次往channel完成发送。

这个规则概况了前面那条同样是关于带缓冲channel的规则。它允许用带缓冲channel来实现计数信号量：channel里面元素的数量和当前实际使用的信号量数量相等，channel的初始化空间大小和最大能同时使用的信号量数量相等，往channel发送一个元素相当于获取信号量，从channel接收一个元素相当于释放信号量。这是一种常见的限制并发量的手法。

这个程序为work列表的每一个元素开启了一个协程，**但是这些协程用限制初始化大小的channel保证同一时刻最多有三个work在执行**。

```
var limit = make(chan int, 3)

func main() {
	for _, w := range work {
		go func(w func()) {
			limit <- 1
			w()
			<-limit
		}(w)
	}
	select{}
}
```

> 可以用以上这种方法来控制协程数量。

### 锁

sync包实现了两种锁数据类型，sync.Mutex和sync.RWMutex。

> sync.Mutex 和 sync.RWMutex具体可以参考： https://www.jianshu.com/p/679041bdaa39

对于任何sync.Mutex或sync.RWMutex的锁变量l和两个描述次数的n和m（`n < m`），调用第n次l.Unlock()`happens before`调用第m次l.Lock()的返回。

这个程序：

```
var l sync.Mutex
var a string

func f() {
	a = "hello, world"
	l.Unlock()
}

func main() {
	l.Lock()
	go f()
	l.Lock()
	print(a)
}
```

会保证打印”hello, world”。第一次调用l.Unlock(在f函数中)`happens before`第二次调用l.Lock()(在main函数中)的返回，第二次l.Lock()的返回`happens before`打印。

对于sync.RWMutex变量l的任意调用`l.RLock`，`l.RLock`阻塞直到n次调用`l.UnLock`，并且n次`l.RUnlock` `happens before` 第n+1次调用`l.Lock`。

```
备注：
英文原文对读写锁的这句描述有些绕，个人对读写锁的理解是：
进入读锁的前提条件是，当前可以有0~n个读锁已被进入，但当前必须没有写锁已被进入
进入写锁的前提条件是，当前没有读锁已被进入，且当前也没有写锁已被进入
那么原文这句话的意思其实是，读锁需要写锁释放后才可能进入，写锁需要所有读锁释放后才可能进入
```

### Once

sync包中的Once类型提供了一种在多协程环境下做初始化工作的安全机制。多线程可以为一个特定的函数f执行`once.Do(f)`，但是只有一个`f()`会被执行，并且其他的调用会阻塞直到`f()`执行完毕。

多协程使用`once.Do(f)`，只有唯一的那个被执行的`f()`执行完并返回，然后其它的才返回。

这个程序：

```
var a string
var once sync.Once

func setup() {
	a = "hello, world"
}

func doprint() {
	once.Do(setup)
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
```

调用twoprint只会调用setup一次。setup函数会在调用print之前执行完毕。结果是”hello, world”会打印两次。

> 我这里啥都不打印，没有出现打印两次的情况

## 示范错误的同步原语使用方法

记住，读操作r可能会观察到并发写操作w的结果。即使发生了这种情况，并不意味着在r之后的读也会观察到发生在r之前的w。

这个程序：

```
var a, b int

func f() {
	a = 1
	b = 2
}

func g() {
	print(b)
	print(a)
}

func main() {
	go f()
	g()
}
```

有可能出现g函数先打印2再打印0。

> 很神奇,但是我就打印了两个00，很难复现。

这个事实使某些写法变得是不正确的。

双检锁是一种尝试避免多余同步操作的手段。比如，twoprint程序可能被错误改写成：

```
var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func doprint() {
	if !done {
		once.Do(setup)
	}
	print(a)
}

func twoprint() {
	go doprint()
	go doprint()
}
```

但是这并不能保证，在doprint函数中，观察到对done的写入就等效于观察到对a的写入。这个错误的版本可能错误的打印出一个空字符串而不是”hello, world”。

> 其实也就是说goroutine中赋值的顺序是不一定的，(只要两个值顺序没影响)。

另一种错误的写法是繁忙等待一个变量，就像：

```
var a string
var done bool

func setup() {
	a = "hello, world"
	done = true
}

func main() {
	go setup()
	for !done {
	}
	print(a)
}
```

就像前面所说，并不能保证在main函数中观察到对done的写入就意味着观察到对a的写入，所有这个程序也可能打印出一个空字符串。更糟糕的是，并不能保证对done的写入会被main函数观察到，因为两个线程间并没有使用同步事件。main函数中的循环并不能保证会结束。

还有一些关于这个主题的一些细小差别的其它场景，就像这个程序。

```
type T struct {
	msg string
}

var g *T

func setup() {
	t := new(T)
	t.msg = "hello, world"
	g = t
}

func main() {
	go setup()
	for g == nil {
	}
	print(g.msg)
}
```

即使main函数观察到了`g != nil`并且退出了这个循环，也不能保证main函数就能观察到对g.msg的修改。

所有的这些例子，解决方法都是相同的：使用显式的同步原语操作。
