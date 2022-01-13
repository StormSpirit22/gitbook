# CRAQ

## zookeeper

提供 test-and-set 服务

config info

master 选举

以文件的形式存储，文件和目录被称为 **znodes**，有三种类型：

regular

ephemeral

sequential

**api**：

create(path,data,flags）排他的

delete(path, version)

exists(path, watch) watch 为 true 时会检测 path，如果有文件生成或删除等，会通知

getdata(path, watch)

setdata(path, data, version)

list(directory)

使用 zookeeper 正确设置值：

```go
for {
  x, v := getdata("f")
  if setdata("f", x+1, v) {
    break
  }
}
```

如果设置值的版本 v 和当前版本一致，说明 x 没有被改过，所以能正确设置 x+1。

上面的操作被称为微事务。

Lock

需要：

1. if create("f", ephmeral = true) return true or false
2. 如果上一步返回 false，那么 if exists("f", watch = true)
3. 上一步成功就一直 wait
4. 等待通知，如果 "f" 被删除了或者第二部 if exists 返回 false，就去第 1 步。

以上步骤来使用文件创建一个 lock。

以上两个例子都有 herd effect（羊群效应），如果有一个锁被释放，那么所有客户端都会收到通知，然后一起去第一步，然后又重新等待同一个锁。

Lock2

1. create sequential file f , ephemeral = true
2. list f\*
3. if no lower number file, return
4. if exists(next lower number, watch = true)
5. wait
6. go to 2

意思是先创建一个序列化文件 f，比如 f27，然后把所有 f\* 文件列举出来，如果没有比 27 更小的文件存在，那么 return 表示成功获取锁。否则一直 watch 比 27 小的文件，比如 f26 文件被删除了（释放锁），那么就重新去第二步，列举出所有文件，看是否有更小的序号文件存在，存在就表示还有锁没被释放，需要继续等待。

为什么需要重新去第二步 list，是因为可能前一个客户端已经失去连接或者挂了，master 会删除它创建的序列文件，但是可能前面还有其他的锁文件没释放，所以需要再 list 看有没有更小的数字文件存在。并不是前面一个文件锁释放了，就表示前面所有的文件锁都释放了。

这种方式可以避免羊群效应，因为每个文件只会等待它的上一个数字的文件释放，比如第一个文件被删除，只有创建第二个文件的线程会收到通知，然后再去循环执行。

以上的 “锁” 并不能保证原子性，因为如果一个线程获取了锁，它的工作可能只做了一半然后 crash 了，它就会马上释放锁，这时其他线程获取到锁了，会出现不可预知的问题。没有原子性保证。

## CRAQ

chain replication：

![](<../../.gitbook/assets/craq-1 (1).png>)

通常需要一个 config manager，配置中心可能使用 raft、paxos 或 zookeeper 等算法，来确定一个链的配置，即有哪些服务器构成一个链。链本身节点无法控制自己成为 head 或 tail。如果一个链 head 宕机了，那么需要 config manager 来确定下一个 head，通常是之前的 head 的下一个节点，而不是由节点自己确定。
