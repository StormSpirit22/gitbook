# 持久化

Redis为持久化提供了两种方式：

* RDB：在指定的时间间隔能对你的数据进行快照存储。
* AOF：记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据。

## 持久化的配置

### RDB的配置

```yaml
# 时间策略
save 900 1
save 300 10
save 60 10000

# 文件名称
dbfilename dump.rdb

# 文件保存路径
dir /home/work/app/redis/data/

# 如果持久化出错，主进程是否停止写入
stop-writes-on-bgsave-error yes

# 是否压缩
rdbcompression no

# 导入时是否检查
rdbchecksum yes
```

* `save 900 1` 表示900s内如果有1条是写入命令，就触发产生一次快照，可以理解为进行一次备份
* `save 300 10` 表示300s内有10条写入，就产生快照

想要禁用RDB配置，只需要在save的最后一行写上：`save ""`

### AOF的配置

```yaml
# 是否开启aof
appendonly yes

# 文件名称
appendfilename "appendonly.aof"

# 同步方式
appendfsync everysec

# aof重写期间是否同步
no-appendfsync-on-rewrite no

# 重写触发配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 加载aof时如果有错如何处理
aof-load-truncated yes

# 文件重写策略
aof-rewrite-incremental-fsync yes
```

`ppendfsync everysec` 它其实有三种模式:

* always：把每个写命令都立即同步到aof，很慢，但是很安全
* everysec：每秒同步一次，是折中方案
* no：redis不处理交给OS来处理，非常快，但是也最不安全

一般情况下都采用 **everysec** 配置，这样可以兼顾速度与安全，最多损失1s的数据。

## RDB的原理

在Redis中RDB持久化的触发分为两种：自己手动触发与Redis定时触发。

**针对RDB方式的持久化，手动触发可以使用：**

* save：会阻塞当前Redis服务器，直到持久化完成，线上应该禁止使用。
* bgsave：Redis调用fork()，产生一个子进程。子进程把数据写到一个临时的RDB文件。当子进程写完新的RDB文件后，把旧的RDB文件替换掉。

**优点**：RDB文件是一个很简洁的单文件，它保存了某个时间点的Redis数据，很适合用于做备份。你可以设定一个时间点对RDB文件进行归档，这样就能在需要的时候很轻易的把数据恢复到不同的版本。比起AOF，在数据量比较大的情况下，RDB的启动速度更快。

**缺点**：RDB容易造成数据的丢失。假设每5分钟保存一次快照，如果Redis因为某些原因不能正常工作，那么从上次产生快照到Redis出现问题这段时间的数据就会丢失了。RDB使用fork()产生子进程进行数据的持久化，如果数据比较大的话可能就会花费点时间，造成Redis停止服务几毫秒。如果数据量很大且CPU性能不是很好的时候，停止服务的时间甚至会到1秒。

## AOF的原理

AOF的整个流程大体来看可以分为两步，一步是命令的实时写入（如果是 `appendfsync everysec` 配置，会有1s损耗），第二步是对aof文件的重写。重启Redis时，AOF里的命令会被重新执行一次，重建数据。

对于增量追加到文件这一步主要的流程是：命令写入=>追加到aof\_buf =>同步到aof磁盘。那么这里为什么要先写入buf在同步到磁盘呢？如果实时写入磁盘会带来非常高的磁盘IO，影响整体性能。

**aof重写**是在新的 aof 文件上会写入能重建当前数据集的最小操作命令的集合。

aof 重写过程：Redis调用fork()，产生一个子进程。子进程把新的AOF写到一个临时文件里。主进程持续把新的变动写到内存里的buffer，同时也会把这些新的变动写到旧的AOF里，这样即使重写失败也能保证数据的安全。当子进程完成文件的重写后，主进程会获得一个信号，然后把内存里的buffer追加到子进程生成的那个新AOF里，新的AOF文件替换旧的AOF文件

**优点**：比RDB可靠。你可以制定不同的fsync策略：不进行fsync、每秒fsync一次和每次查询进行fsync。默认是每秒fsync一次。这意味着你最多丢失一秒钟的数据。

**缺点**：在相同的数据集下，AOF文件的大小一般会比RDB文件大。

**手动触发：** `bgrewriteaof`，**自动触发** 就是根据配置规则来触发，当然自动触发的整体时间还跟Redis的定时任务频率有关系。

redis 4.0以上支持混合持久化，aof重写时会先用rdb的方式在aof文件中写入快照，然后追加后续aof命令。

## 从持久化中恢复数据

![](<../../.gitbook/assets/persist-1 (1).png>)
