# TCP Keepalive 和 HTTP Keep-Alive

HTTP 和 TCP 都有 Keep-Alive 机制，分别都有什么区别呢？

## HTTP Keep-Alive

HTTP 是基于 TCP 传输协议实现的，客户端需要与服务端进行 HTTP 通信时，需要先建立 TCP 连接，客户端再发送 HTTP 请求，服务端收到再响应。因此最开始 HTTP 1.0 的建连并传输的机制就是：

- 请求 1：client TCP 建连 server，发送数据，关闭 TCP 连接。
- 请求 2：client TCP 建连 server，发送数据，关闭 TCP 连接。

每次请求都需要重新建连，这样非常浪费资源和时间。

于是出现了 Keep-Alive 机制。

### Keep-Alive

Keep-Alive 解决的核心问题是： 一定时间内，同一域名多次请求数据，**只建立一次 TCP 连接**，其他请求可复用每一次建立的连接通道，以达到提高请求效率的问题。这样使用同一个 TCP 连接来发送接收多个 HTTP 请求的方法成为 **HTTP 长连接**。在 HTTP 1.0 默认是关闭的，如果需要开启 Keep-Alive，必须在请求头添加：

```http
Connection: Keep-Alive
```

从 HTTP 1.1 开始默认开启了 Keep-Alive 机制，现在大多数浏览器都默认使用 HTTP 1.1 。多次 http 请求效果如下图所示：

![image-20240928173642159](/Users/zonst/Library/Application Support/typora-user-images/image-20240928173642159.png)



`Keep-Alive`还是存在如下问题：

- 串行的文件传输。
- 同域并行请求限制带来的阻塞（6~8）个

**在HTTP协议中，Keep-Alive属性保持连接的时间长短是由服务端决定的，通常配置都是在几十秒左右。**有些如 tomcat 默认是 60 秒。

### 管线化(pipeline)

HTTP 管线化可以克服同域并行请求限制带来的阻塞，它是建立在**持久连接**之上，是把所有请求一并发给服务器，但是服务器需要按照**顺序一个一个响应**，而不是等到一个响应回来才能发下一个请求，这样就节省了很多请求到服务器的时间。不过，HTTP 管线化**仍旧**有阻塞的问题，若上一响应迟迟不回，**后面的响应**都会被阻塞到。

![image-20240928173824181](/Users/zonst/Library/Application Support/typora-user-images/image-20240928173824181.png)



### 多路复用

多路复用代替原来的序列和阻塞机制。所有就是请求的都是通过一个 TCP 连接并发完成。因为在多路复用之前所有的传输是基于基础文本的，在多路复用中是基于二进制数据帧的传输、消息、流，所以可以做到乱序的传输。多路复用对同一域名下所有请求都是基于流，所以不存在同域并行的阻塞。多次请求如下图：

![image-20240928174649310](/Users/zonst/Library/Application Support/typora-user-images/image-20240928174649310.png)

## TCP 的 Keepalive

TCP 的 Keepalive 其实是 TCP 的保活机制，如果对端程序是正常工作的。当 TCP 保活的探测报文发送给对端,对端会正常响应，这样 TCP 保活时间会被重置，等待下一个 TCP 保活时间的到来，

如果对端主机宕机（注意不是进程崩溃，进程崩溃后操作系统在回收进程资源的时候，会发送 FIN 报文，而主机宕机则是无法感知的，所以需要TCP 保活机制来探测对方是不是发生了主机宕机），或对端由于其他原因导致报文不可达。当 TCP 保活的探测报文发送给对端后，石沉大海，没有响应，连续几次，达到保活探测次数后，TCP 会报告该TCP连接已经失效。

所以，TCP 保活机制可以在双方没有数据交互的情况，通过探测报文，来确定对方的 TCP 连接是否存活，这个工作是在内核完成的。

<img src="https://cdn.xiaolincoding.com//mysql/other/87e138ae9f2438c8f4e2c9c46ec40b95.png" alt="TCP 保活机制" style="zoom:50%;" />



## 总结

HTTP 的 Keep-Alive 也叫 HTTP 长连接，该功能是由「应用程序」实现的，可以使得用同一个 TCP 连接来发送和接收多个 HTTP 请求/应答，减少了 HTTP 短连接带来的多次 TCP 连接建立和释放的开销。

TCP 的 Keepalive 也叫 TCP 保活机制，该功能是由「内核」实现的，当客户端和服务端长达一定时间没有进行数据交互时，内核为了确保该连接是否还有效，就会发送探测报文，来检测对方是否还在线，然后来决定是否要关闭该连接。