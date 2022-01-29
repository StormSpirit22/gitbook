# Zookeeper

## 线性一致

如果执行历史整体可以按照一个顺序排列，且排列顺序与客户端请求的实际顺序（real time order）相符合，并且每一个读操作都看到的是最近一次写入的值，那么它是线性一致的。

![](<../../.gitbook/assets/zkbj-1.png>)w先设置为0，再设置成2，读取2，设置成1，读取1。所以这是一个 线性一致的运行过程。

![](<../../.gitbook/assets/zkbj-2.png>)

![](<../../.gitbook/assets/zkbj-3.png>)

client1 和 client2 看到不同的情况，并且运行过程有个环，不是线性一致。

![](<../../.gitbook/assets/zkbj-4.png>)

线性一致的系统永远不会返回过时的数据。

![](<../../.gitbook/assets/zkbj-5.png>)

C2 在第一次发送 read 请求时，没有收到回复，然后在箭头处重新发送请求，服务器应该要能记住客户端上次发送的请求的返回结果，然后直接返回，而不是再运行一次再返回，服务器要能识别出重复的请求。

## Why ZK

1. API general-purpose coodination service
2. n 个机器是否能提升 n 倍性能

![](<../../.gitbook/assets/zkbj-6.png>)

这种系统机器越多性能越低，性能瓶颈都在 leader 那台机器上。

zookeeper 不保证线性一致，可能会读取到过时的数据。会把只读请求发送到副本服务器，可能不是最新的。但是性能会提高。

## ZK 保证

1. 写线性一致
2.  FIFO 客户端顺序（per client，即对于每一个客户端有 FIFO，而不保证全局命令的 FIFO）。如果同一个客户端先后发送了一个 write 和 read 请求， read 会等 write 执行完成之后再返回。

    writes client specified order

![](<../../.gitbook/assets/zkbj-7.png>)


正常的写配置与读配置。

![](<../../.gitbook/assets/zkbj-8.png>)


需要加一个 watch 来看配置文件是否被更改。
