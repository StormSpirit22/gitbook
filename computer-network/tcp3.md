# TCP 协议

最近问大佬同事面试一般会问什么，答曰：普通的八股文没啥意思，有时候会问一些偏的，比如什么情况下 TCP 是三次挥手？一问三不知，这里记录一下网上关于这个问题的解答。



## TCP 四次挥手

四次挥手大家都知道，上一篇文也说过，这里再回顾一下：

![在这里插入图片描述](/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp8.png)

- 客户端主动调用关闭连接的函数，于是就会发送 FIN 报文，这个 FIN 报文代表客户端不会再发送数据了，进入 FIN_WAIT_1 状态；
- 服务端收到了 FIN 报文，然后马上回复一个 ACK 确认报文，此时服务端进入 CLOSE_WAIT 状态。在收到 FIN 报文的时候，TCP 协议栈会为 FIN 包插入一个文件结束符 EOF 到接收缓冲区中，服务端应用程序可以通过 read 调用来感知这个 FIN 包，这个 EOF 会被**放在已排队等候的其他已接收的数据之后**，所以必须要得继续 read 接收缓冲区已接收的数据；
- 接着，当服务端在 read 数据的时候，最后自然就会读到 EOF，接着 **read() 就会返回 0，这时服务端应用程序如果有数据要发送的话，就发完数据后才调用关闭连接的函数，如果服务端应用程序没有数据要发送的话，可以直接调用关闭连接的函数**，这时服务端就会发一个 FIN 包，这个 FIN 报文代表服务端不会再发送数据了，之后处于 LAST_ACK 状态；
- 客户端接收到服务端的 FIN 包，并发送 ACK 确认包给服务端，此时客户端将进入 TIME_WAIT 状态；
- 服务端收到 ACK 确认包后，就进入了最后的 CLOSE 状态；
- 客户端经过 2MSL 时间之后，也进入 CLOSE 状态；

为啥要四次挥手？因为服务端发送 ACK 之后可能还有数据没发送完，不能马上发送 FIN，需要等待数据全部发送完给客户端，再发送 FIN进行关闭连接。



## TCP 关闭连接的函数

在Linux系统中，shutdown()和close()函数都用于关闭TCP连接，但它们的具体作用和区别如下：

1. close()函数:

- close函数会对socket的引用计数-1，一旦socket的引用计数被减为0，就会对socket进行彻底释放，并且会**关闭TCP两个方向的数据流**。

- 为了关闭两个方向的数据流，在数据接收方向，系统内核会将socket设置为不可读，任何读操作都会返回异常；在数据发送方向，系统内核尝试将发送缓冲区的数据发送给对端，并**最后向对端发送一个FIN报文**，接下来如果再对socket进行写操作会返回异常。

- 如果对端没有检测到socket已关闭，仍然继续发送报文，则会收到一个RST报文。如果**向这个已经收到RST的socket执行写操作**，内核会发出一个**SIGPIPE信号**给进程，该信号的默认行为是终止进程。

```c
int close(int sockfd);
```

2. shutdown()函数:

- shutdown()函数允许你关闭套接字的读取或写入方向，或者同时关闭两个方向。它不会立即关闭连接，而是根据提供的参数执行相应的操作。
- 使用shutdown()函数可以单独关闭套接字的读取通道或写入通道，或者同时关闭两个通道，而不会立即关闭整个套接字。
- shutdown()函数的常见用途之一是在多线程或多进程环境中，通过关闭套接字的写通道来通知对端不再发送数据。

```c
int shutdown(int sockfd, int how);
```

### close和shutdown的区别

1. close 会关闭连接，并**释放所有连接对应的资源**，而 shutdown 并不会释放掉套接字和所有的资源。确切地说，close用来关闭套接字，将套接字描述符（或句柄）从内存清除，**之后再也不能使用该套接字**。应用程序关闭套接字后，与该套接字相关的连接和缓存也失去了意义，TCP协议会自动触发关闭连接的操作。

2. shutdown() 用来**关闭连接**，而不是套接字，不管调用多少次 shutdown()，**套接字依然存在**，直到调用close将套接字从内存清除。(**即调用shutdown后，仍然需要调用close关闭socket**)。

   调用close关闭套接字，或调用shutdown关闭输出流时，都会向对方发送FIN包，FIN 包表示数据传输完毕，计算机收到 FIN 包就知道不会再有数据传送过来了。

3. close存在引用计数的概念，并不一定导致该套接字不可用，而shutdown则不会管引用计数，直接**使得该套接字不可用**，**如果有别的进程企图使用该套接字，将会受到影响**。



## 什么情况会出现三次挥手？

当被动关闭方（即服务端）在 TCP 挥手过程中，「**没有数据要发送」并且「开启了 TCP 延迟确认机制」，那么第二和第三次挥手就会合并传输，这样就出现了三次挥手。**

![image-20240616193031493](/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp9.png)

因为 TCP 延迟确认机制是默认开启的，所以导致我们抓包时，看见三次挥手的次数比四次挥手还多。

> 什么是 TCP 延迟确认机制？

当发送没有携带数据的 ACK，它的网络效率也是很低的，因为它也有 40 个字节的 IP 头 和 TCP 头，但却没有携带数据报文。 为了解决 ACK 传输效率低问题，所以就衍生出了 **TCP 延迟确认**。 TCP 延迟确认的策略：

- 当有响应数据要发送时，ACK 会随着响应数据一起立刻发送给对方
- 当没有响应数据要发送时，ACK 将会延迟一段时间，以等待是否有响应数据可以一起发送
- 如果在延迟等待发送 ACK 期间，对方的第二个数据报文又到达了，这时就会立刻发送 ACK

<img src="/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp10.png" alt="img" style="zoom: 67%;" />

> 怎么关闭 TCP 延迟确认机制？

如果要关闭 TCP 延迟确认机制，可以在 Socket 设置里启用 TCP_QUICKACK。



## 验证

Go 由于不支持直接设置 TCP_QUICKACK 的值，得调用 syscall 包设置。linux 默认开启延迟确认，即 TCP_QUICKACK 默认值 0 为开启，1 为不开启。但是经过测试，同一份代码在 mac 和 linux 上运行的结果不一样，证明在 macOS 上不支持 tcp 延迟确认，即每次都是四次挥手，而 linux 默认开启，每次都是三次挥手。

### 实验一

macOS 和 linux 运行的同一份代码：

#### 服务端

```go
package main

import (
	"fmt"
	"net"
)

func main() {
	// Listen for incoming connections on port 9000
	ln, err := net.Listen("tcp", ":9000")
	if err != nil {
		fmt.Println(err)
		return
	}

	// Accept incoming connections and handle them
	for {
		conn, err := ln.Accept()
		if err != nil {
			fmt.Println(err)
			continue
		}

		// Handle the connection in a new goroutine
		go handleConnection(conn)
	}
}

func handleConnection(conn net.Conn) {
	// Close the connection when we're done
	defer conn.Close()

	// Read incoming data
	buf := make([]byte, 1024)
	_, err := conn.Read(buf)
	if err != nil {
		fmt.Println(err)
		return
	}

	// Print the incoming data
	fmt.Printf("Received: %s", buf)
}
```

#### 客户端

```go
package main

import (
	"fmt"
	"log"
	"net"
)

func main() {

	raddr, err := net.ResolveTCPAddr("tcp", "localhost:9000")
	if err != nil {
		log.Fatal(err)
	}

	// Connect to the server
	conn, err := net.DialTCP("tcp", nil, raddr)
	if err != nil {
		fmt.Println(err)
		return
	}

	// 设置延迟确认
	conn.SetNoDelay(false)

	// Send some data to the server
	_, err = conn.Write([]byte("Hello, server!"))
	if err != nil {
		fmt.Println(err)
		return
	}

	// Close the connection
	conn.Close()
}
```



本地运行 wireshark 抓包，注意因为客户端和服务端都在本地运行，所以需要抓本机的回环包，需要开启这个：

![image-20240616210633311](/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp11.png)

macOS 运行的结果：

![image-20240616210743813](/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp12.png)

看抓包确实是四次挥手。

Linux 运行的结果，使用命令抓回环包：

```shell
tcpdump -nnvv -i lo port 9000 -w /tmp/tcp.cap
```

![image-20240616210830497](/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp13.png)

变成三次挥手了，少了一个单独的 ACK 包。

### 实验二

在 Linux 环境下关闭 tcp 延迟确认。不过 go 的 syscall 包在 mac 环境下找不到 syscall.TCP_QUICKACK，可见 macOS 确实不支持该设置。但是在 linux 的包可以找到：

![image-20240616212343125](/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp16.png)

![image-20240616212136194](/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp15.png)

#### 服务端

```go
package main

import (
	"fmt"
	"net"
	"syscall"
)

func main() {
	listener, err := net.Listen("tcp", ":9000")
	if err != nil {
		fmt.Println("Error starting server:", err)
		return
	}
	defer listener.Close()
	fmt.Println("Server is listening on port 9000")

	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("Error accepting connection:", err)
			continue
		}
		go handleConnection(conn)
	}
}

const TCP_QUICKACK = 0xc // darwin 系统没定义，自己定义

func handleConnection(conn net.Conn) {
	defer conn.Close()

	// 获取文件描述符
	tcpConn, ok := conn.(*net.TCPConn)
	if !ok {
		fmt.Println("Failed to get TCP connection")
		return
	}
	file, err := tcpConn.File()
	if err != nil {
		fmt.Println("Error getting file descriptor:", err)
		return
	}
	defer file.Close()

	// 设置 TCP_QUICKACK 选项
	fd := int(file.Fd())
	err = syscall.SetsockoptInt(fd, syscall.IPPROTO_TCP, TCP_QUICKACK, 1)
	if err != nil {
		fmt.Println("Error setting TCP_QUICKACK:", err)
		return
	}

	// 读取数据
	buf := make([]byte, 1024)
	n, err := conn.Read(buf)
	if err != nil {
		fmt.Println("Error reading data:", err)
		return
	}
	fmt.Printf("Received: %sn", string(buf[:n]))

}
```

#### 客户端

```go
package main

import (
	"fmt"
	"net"
	"syscall"
)

const TCP_QUICKACK = 0xc // darwin 系统没定义，自己定义

func main() {
	conn, err := net.Dial("tcp", "localhost:9000")
	if err != nil {
		fmt.Println("Error connecting to server:", err)
		return
	}

	// 获取文件描述符
	tcpConn, ok := conn.(*net.TCPConn)
	if !ok {
		fmt.Println("Failed to get TCP connection")
		return
	}
	file, err := tcpConn.File()
	if err != nil {
		fmt.Println("Error getting file descriptor:", err)
		return
	}
	defer file.Close()

	// 设置 TCP_QUICKACK 选项
	fd := int(file.Fd())
	err = syscall.SetsockoptInt(fd, syscall.IPPROTO_TCP, TCP_QUICKACK, 1)
	if err != nil {
		fmt.Println("Error setting TCP_QUICKACK:", err)
		return
	}

	// 发送数据
	_, err = conn.Write([]byte("Hello, server"))
	if err != nil {
		fmt.Println("Error sending data:", err)
		return
	}

	conn.Close()
}
```

抓包结果：

![image-20240616211236224](/Users/zonst/Documents/Github/gitbook/.gitbook/assets/tcp14.png)

手动设置 TCP_QUICKACK 为 1 后变成四次挥手了。

要注意设置 TCP_QUICKACK 并不是永久的，所以每次读取数据的时候，如果想要立刻回 ACK，那就得在每次读取数据之后，重新设置 TCP_QUICKACK。但是一般在实际应用中不会有人这么做，就使用默认的系统设置即可。

> TCP_QUICKACK (since Linux 2.4.4)    Enable  quickack  mode  if  set or disable quickack mode if cleared.  In quickack mode, acks are sent immediately, rather than delayed if    needed in accordance to normal TCP operation.  This flag is not permanent, it only enables a switch to or from quickack mode.  Subsequent    operation  of  the  TCP  protocol will once again enter/leave quickack mode depending on internal protocol processing and factors such as    delayed ack timeouts occurring and data transfer.  This option should not be used in code intended to be portable.
>
> TCP_QUICKACK（自Linux 2.4.4起） 如果设置了此选项，则启用快速确认模式，如果清除了此选项，则禁用快速确认模式。在快速确认模式下，立即发送确认，而不是根据正常的TCP操作需要延迟发送确认。这个标志不是永久的，它只是启用或禁用快速确认模式的开关。随后的TCP协议操作将根据内部协议处理和延迟确认超时等因素，再次进入或离开快速确认模式。这个选项不应该在用于可移植的代码中使用。



## 总结

当被动关闭方在 TCP 挥手过程中，如果「没有数据要发送」，同时「没有开启 TCP_QUICKACK（默认情况就是没有开启，没有开启 TCP_QUICKACK，等于就是在使用 TCP 延迟确认机制）」，那么第二和第三次挥手就会合并传输，这样就出现了三次挥手。

**所以，出现三次挥手现象，是因为 TCP 延迟确认机制导致的。**