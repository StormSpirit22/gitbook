# 数据库篇

## 池化技术

数据库连接池。

go 协程池。

一个非常简单的协程池：

```go
package main

import (
	"fmt"
	"time"
)

type Pool struct {
	work chan func()   // 任务
	sem  chan struct{} // 数量
}
func New(size int) *Pool {
	return &Pool{
		work: make(chan func()),
		sem:  make(chan struct{}, size),
	}
}

func (p *Pool) NewTask(task func()) {
	select {
	case p.work <- task:
    // 这里会等待协程运行完任务后才会进入
		fmt.Println("task in queue")
	case p.sem <- struct{}{}:
    // 如果超过 channel 的 buffer 则会阻塞住，不会创建新的 channel
		go p.worker(task)
	}
}

func (p *Pool) worker(task func()) {
	defer func() { <-p.sem }()
	// 复用协程，不会退出
	for {
		fmt.Println("run task")
		task()
		task = <-p.work
	}
}

func main()  {
	pool := New(3)

	for i := 0; i < 4; i++{
		i := i
		pool.NewTask(func(){
			//time.Sleep(100 * time.Millisecond)
			fmt.Println(i)
		})
	}

	// 保证所有的协程都执行完毕
	time.Sleep(1 * time.Second)
}
```

可以参考

[如何手动实现一个协程池？](https://www.cnblogs.com/wongbingming/p/13091091.html)

[100 行写一个 go 的协程池 (任务池)](https://segmentfault.com/a/1190000021468353)

## 主从分离

### 主从读写分离

两个关键点：

1. 一个是数据的拷贝，我们称为主从复制；
2. 在主从分离的情况下，我们 **如何屏蔽主从分离带来的访问数据库方式的变化**，让开发同学像是在使用单一数据库一样。

#### 主从复制

MySQL 的主从复制是依赖于 binlog 的，也就是记录 MySQL 上的所有变化并以二进制形式保存在磁盘上二进制日志文件。**主从复制就是将 binlog 中的数据从主库传输到从库上 ** ，一般这个过程是异步的，即主库上的操作不会等待 binlog 同步的完成。

**主从复制的过程是这样的： **

1. 首先从库在连接到主节点时会创建一个 IO 线程，用以请求主库更新的 binlog，并且把接收到的 binlog 信息写入一个叫做 relay log 的日志文件中
2. 而主库也会创建一个 log dump 线程来发送 binlog 给从库；
3. 同时，从库还会创建一个 SQL 线程读取 relay log 中的内容，并且在从库中做回放，最终实现主从的一致性。这是一种比较常见的主从复制方式。

在这个方案中，使用独立的 log dump 线程是一种异步的方式，可以避免对主库的主体更新流程产生影响，而从库在接收到信息后并不是写入从库的存储中，是写入一个 relay log，是避免写入从库实际存储会比较耗时，最终造成从库和主库延迟变长。

![img](../../.gitbook/assets/hc-db-1.png)

binlog，relaylog 可参考：[MySQL中的binlog和relay-log结构完全详解](https://www.51cto.com/article/626540.html)

**一般一个主库最多挂 3～5 个从库**。

主从数据库存在延迟，一般可以使用以下方法：

1. **第一种方案是数据的冗余。**

   你可以在发送消息队列时不仅仅发送微博 ID，而是发送队列处理机需要的所有微博信息，借此避免从数据库中重新查询数据。

2. **第二种方案是使用缓存。**

   我可以在同步写数据库的同时，也把微博的数据写入到 Memcached 缓存里面，这样队列处理机在获取微博信息的时候会优先查询缓存，这样也可以保证数据的一致性。

3. **最后一种方案是查询主库。**

   我可以在队列处理机中不查询从库而改为查询主库。不过，这种方式使用起来要慎重，要明确查询的量级不会很大，是在主库的可承受范围之内，否则会对主库造成比较大的压力。

#### 如何访问数据库

可以借助一些公开库，比如：https://github.com/flike/kingshard



## 分库分表