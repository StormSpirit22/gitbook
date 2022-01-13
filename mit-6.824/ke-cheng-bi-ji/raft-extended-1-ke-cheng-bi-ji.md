# Raft Extended (1) 课程笔记

## Fault Tolerance System

MapReduce：由单一 master 来控制复制过程。

GFS：primary backup（主备方案）复制实际文件内容，通常有 3 个副本，一个 primary 和 2 个 secondary。

VMWare FT： replicated state machine（状态机转移），primary 发送指令到 backup。当 primary failed，借助 test-and-set 让 backup 接管 primary。

以上这些方案都需要一个单一的 master 来控制谁是 primary。可能会发生单点故障。

### split brain

![](<../../.gitbook/assets/raft\_1 (1).png>)

​ 图 1

如图所示。假设对于一个容错系统来说，客户端能与其中之一的服务器通信就表示正常。

C1 C2 是 vmware 的虚拟机，S1 S2 是 test-and-set 服务器。

如果发生网络分区（partition）， C1 只能与 S1 通信， C2 只能与 S2 通信，当 C1 发送一条 [test-and-set](https://zh.wikipedia.org/wiki/%E6%A3%80%E6%9F%A5%E5%B9%B6%E8%AE%BE%E7%BD%AE) 指令到 S1 里，S1 会对某个[存储器](https://zh.wikipedia.org/wiki/%E8%A8%98%E6%86%B6%E9%AB%94) 位置写入1 并返回其旧值 0，C1 就会认为自己获得了锁并接管 primary，但此时 S2 的值还是 0 。同样 C2 发送指令到 S2，C2 也成为 primary，这就发生了 split brain 情况。

对于以上的情况，要么放弃容错，等待所有服务器返回。要么就只等待一台服务器返回，放弃正确性（放弃正确性的版本就是 split brain）。

**解决 split brain 的方法主要是 majority voting system ，或称作 quorum 系统（多数派系统）：**

1. 奇数台服务器。
2. 每一步操作都需要获得大部分服务器（超过一半）的同意。这里大部分服务器指的是**所有服务器**，包括在线的宕机的，而不是仅指在线的服务器。
3. 如果有 2f + 1 的服务器，那么可以容忍 f 台机器的故障。

**majority voting system 的特征：**

1. 对于网络分区来说，至多有一个 partition 拥有大多数服务器。如果发生网络分区，就能保证不让所有的网络分区都能推进执行，也就不会发生 图 1 的情况。比如如果有 3 台服务器，网络分区中必定有一个分区里只有 1 台服务器，那么这个分区就不能推进执行，因为没有满足大多数服务器（即 2 台）都要同意的要求。
2. 两个任意的 majority 至少有 1 台服务器重叠。这样可以避免 split brain。

## Raft

![](<../../.gitbook/assets/raft\_2 (1).png>)

​ 图 2

如图 2 所示。流程：

1. C1 发送指令到 raft 集群的 leader 服务器。
2. leader 将指令追加到自己的日志中。
3. leader 将指令通过调用 AppendEntries RPC 发送给 follower 服务器。
4. follower 服务器将 leader 发送的指令加到自己的日志中，并返回。
5. leader 收到大多数服务器（包括它自己）的返回后，运行指令，返回客户端结果。
6. leader 在下次向 follower 发送 AppendEntries 时会加入 commitIndex，即已经提交了日志的索引。
7. 所有收到指令的 follower 运行索引为 commitIndex 的日志。

### Raft 接口

kv 层将指令传输给 raft 层，调用 start(command) 方法。

raft 层通知 kv 层，applyCh 通道发送 ApplyMsg(command, index) 。

**只有当 start 函数返回后，且与客户端命令一致的 ApplyMsg 通过 applyCh 传递到 kv 层时，kv 层才会执行命令并返回给客户端。**

### Leader Election

选举计时器超时 -> 开始选举

term++， 发送 RequestVotes RPC，得到大部分的选票，成为 leader，发送 AppendEntries RPC，接收到 AppendEntries 消息的服务器重置选举计时器。

**广播时间（broadcastTime） << 选举超时时间（electionTimeout） << 平均故障间隔时间（MTBF）**

每次重置选举计时器时（比如接收到心跳时）都需要**重新随机选取一个值**，而不是一直用启动时创建的随机值。这样可以避免 split vote

关于日志复制的问题详见论文 5.3 5.4。
