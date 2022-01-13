# Primary-Backup Replication 课程笔记

## replication - 容错

可以解决 fail-stop 故障

不能解决 bugs

### replication 方法：

**状态转移（state transfer）**：primary 发送自己的一个拷贝给副本同步，包括内存等所有所需的信息 -> memory。expensive, 可以处理多核和并发的情况。

**状态机副本（replicated state machine）**：primary 发送一些客户端的操作 -> operations，inputs，events 。cheap, 无法处理多核和并发的情况。

### 问题：

1. 同步哪些状态？
2. 同步到哪种程度？比如延迟
3. 在 primary 故障时，客户端切换
4. 发现异常
5. 当一个副本发生故障时，需要马上创建一个新的副本，这时就需要整个状态转移。

非确定性事件：

1. inputs - packets - data + interupt（输入 -> 网络包 -> 包含的数据和中断的时间点）
2. wired instructions（随机事件，比如随机数、当前时间）
3. multi-core

log entry：instruction number，type，data

output rule

primary 或 backup 需要（可能是第三方存储里）做 test-and-set 原子操作来判断是否能接管服务，保证了当前只有一个 primary。
