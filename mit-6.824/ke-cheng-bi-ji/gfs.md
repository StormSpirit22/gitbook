# GFS

## 大存储的问题：

性能 -> 分块

错误 -> 容错

容错 -> 副本

副本 -> 低性能

## GFS

master 只有一个，用来管理追踪 chunk 的位置， chunk 会存储实际的数据。一个 chunk 64 MB。

Master Data 有 2 个数据映射：

filename -> arrays of chunk handles （nv，not volatile，不易失，需要存储到硬盘）

chunk handle -> list of chunk servers (v)

每个 chunk 都有一个当前版本号，master 需要记录每个 chunk 的版本号。(nv)

master 选取一个 primary chunk 签订租约，master 记录租约的过期时间。(v)

所有的 chunk 的写操作都需要在 primary chunk 序列化。

以上这些数据会存储在 master 内存中，部分（nv）会同时记录在存储在磁盘的 log 里。**只要有数据变化**就会在 log 里进行记录，每过一段时间还会创建 checkpoint ，这些也会存储到磁盘里。

当 master crash 之后，只需要将其恢复到 checkpoint 状态，然后重演之后的 log 日志即可恢复。

### 读取数据

1. 客户端发送文件名和偏移量到 master想要读取数据。
2. master 发送 chunk handle 和 server 列表给客户端。客户端会缓存给定的 chunk 对应的 server 信息。
3. 客户端向 chunk server 发送需要读取的数据，server 返回。

#### 写入数据

1. 如果没有 primary chunk，master 会找到所有拥有最新版本的 chunk，选取一个作为 primary，其他的作为 secondary server。
2. master 将版本号递增，发送新的版本号给 primary 和 secondaries，chunk servers 也会记录新的版本号到磁盘里。master 将版本号记录日志。这样在崩溃后也不会丢失版本号。（该顺序不确定，是先通知 chunk 还是先记录日志。）
3. master 和 primary 签订一个 60s 的租约，同一时间只有一个 primary。master 会将 primary 和 secondaries 告诉客户端。
4. 当 client 需要写数据时，client 会将要追加的数据发送给离它最近的一个副本，然后根据网络链路将数据发送到所有的副本。primary 和 secondaries 会将数据写到一个临时区域。
5. 等所有 primary 和 secondaries 都收到数据并回复 client 之后，primary 会选择一个偏移量（chunk 的末尾）并告知所有 副本在同样的偏移量那里追加数据。
6.  如果有些副本追加数据发生了错误，那么 primary 会告知 client ，client 可能会重新发起一个追加的任务，直到所有副本追加数据成功，primary 就会返回 yes，这个时候所有副本的某个偏移位置都会有追加的数据，但可能有些副本之前的偏移位置也有相同的数据，不过不影响。

    论文中的详细写流程：

![谷歌三大核心技术（二）Google MapReduce中文版](https://simg.open-open.com/show/ecab726c6eda85c8c1a24f053fcd57e5.bmp)

1. client 向 master 询问那个 chunkserver 获取了当前 chunk 的租约以及其他副本所在的位置。如果没有人得到租约，master 将租约授权给它选择的一个副本。
2. master 返回该主副本的标识符以及其他副本的位置。Client 为未来的变更缓存这个数据。只有当主副本没有响应或者租约到期（60s 或者 master revoke）时它才需要与 master 联系。
3. client 将数据推送给所有的副本，client 可以以任意的顺序（针对副本的顺序）进行推送。每个 chunkserver 会将数据存放在内部的 LRU buffer cache 里，直到数据被使用或者过期（缓冲流）。通过将控制流与数据流分离，我们可以通过将昂贵的数据流基于网络拓扑进行调度来提高性能，而不用考虑哪个 chunkserver 是主副本。3.2 节更深入地讨论了这点。
4. 一旦所有的副本接受到了数据，client 发送一个写请求给主副本，这个请求标识了先前推送给所有副本的数据。主副本会给它收到的所有变更(可能来自多个 client)安排一个连续的序列号来进行必需的串行化。它将这些变更根据序列号应用在本地副本上。
5. 主副本将写请求发送给所有的次副本，每个次副本以与主副本相同的串行化顺序应用这些变更。
6. 所有的次副本完成操作后向主副本返回应答
7. 主副本向 client 返回应答。任何副本碰到的错误都会返回给 client。出现错误时，该写操作可能已经在主副本以及一部分次副本上执行成功。(如果主副本失败，那么它不会安排一个序列号并且发送给其他人)。客户端请求将会被认为是失败的，被修改的区域将会处在非一致状态下。我们的客户端代码会通过重试变更来处理这样的错误。它会首先在 3 到 7 步骤间进行一些尝试后在重新从头重试这个写操作。

如果应用程序的一个写操作很大或者跨越了 chunk 的边界，GFS client 代码会将它转化为多个写操作。它们都会遵循上面的控制流程，但是可能会被来自其他 client 的操作插入或者覆盖。因此共享的文件区域可能会包含来自不同 client 的片段，虽然这些副本是一致的，因为所有的操作都按照相同的顺序在所有副本上执行成功了。但是文件区域会处在一种一致但是未定义的状态，正如 2.7 节描述的那样（一致，但是由于并发 client 获取不到全部的操作信息，所以是未定义的）。

如果同时存在 2 个 primary，这种情况成为 split brain。可能会有两种完全不同的数据副本。租约就是为了解决这个问题。 当 master 不能和 primary 通信时， master 必须等待租约到期后，才会指定一个新的 primary。因为只有当 primary 的租约到期后才会忽略 client 的请求，为了防止同时存在 2 个 primary 与 client 通信，master 必须等待租约到期。

通常有 3 个副本，一个 primary 和 2 个 secondary。
