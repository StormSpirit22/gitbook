# Go, Threads, and Raft 课程笔记

## Go

只需要知道怎么写正确的并发代码，对于 happens before 不需要了解得很详细。如果需要加一个特别大的锁时也不需要考虑 cpu 的性能，这个影响很微小。

lock 原则：

1. lock 用来保护共享数据。
2.  lock 用来保护 **invariants** （不变量），比如如果要确保某个值不变，那么需要将修改该值得操作都放在同一块加锁的代码里：

    ```go
    go func() {
      for i := 0; i < 1000; i++ {
        mu.Lock()
        alice--
        bob++
        mu.Unlock()
      }
    }()
    ```

    而不是这样：

    ```go
    go func() {
      for i := 0; i < 1000; i++ {
        mu.Lock()
        alice--
        mu.Unlock()
        mu.Lock()
        bob++
        mu.Unlock()
      }
    }()
    ```

lock 主要保护某一块代码的原子性。

## Threads

以下是课程里的部分示例代码：

```go
/*
闭包的正确写法
 */
func closure() {
	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func(x int) {
			sendRpc(x)
			wg.Done()
		}(i)
	}
	wg.Wait()
}

func sendRpc(i int) {
	println(i)
}

/*
定期等待的正确写法
wait for something periodically
注意：
如果把 wait 和 periodic 里的 lock 都去掉，那么编译器可能会将 periodic 的代码优化成第一次读取 done 之后，
就一直在 for 循环里用这个值了，因为没有检测到并发限制比如 lock 等，说明这个变量可能不会被其他并发协程改变。
但实际运行的时候去掉了 lock 还是正常运行的。
*/

var mu sync.Mutex
var done bool

func wait() {
	time.Sleep(time.Second)
	println("started")
	go periodic()
	time.Sleep(5 * time.Second) // wait for a while so we can see what ticker does
	mu.Lock()
	done = true
	mu.Unlock()
	println("canceled")
	time.Sleep(3 * time.Second) // observe no output
}

func periodic() {
	for {
		println("tick")
		time.Sleep(time.Second)
		// 如果 mu 之前被加过锁，那么这里会一直阻塞到 mu.Unlock() 被调用
		mu.Lock()
		if done {
			mu.Unlock()
			return
		}
		mu.Unlock()
	}
}

/*
累加的正确写法，这里也可以用 sync.WaitGroup
 */
func update() {
	counter := 0
	var mu sync.Mutex
	for i := 0; i < 1000; i++ {
		go func() {
			mu.Lock()
			defer mu.Unlock()
			counter ++
		}()
	}
	time.Sleep(time.Second)
	mu.Lock()
	println(counter)
	mu.Unlock()
}

/*
保护不变值，加锁的错误写法
 */
func perField() {
	alice := 10000
	bob := 10000
	var mu sync.Mutex

	total := alice + bob

	go func() {
		// 这块是原子操作，但是和另一个协程一起运行就不是
		for i := 0; i < 1000; i++ {
			mu.Lock()
			alice--
			mu.Unlock()
			mu.Lock()
			bob++
			mu.Unlock()
		}
	}()

	go func() {
		for i := 0; i < 1000; i++ {
			mu.Lock()
			bob--
			mu.Unlock()
			mu.Lock()
			alice++
			mu.Unlock()
		}
	}()

	start := time.Now()
	for time.Since(start) < time.Second {
		mu.Lock()
		if alice + bob != total {
			fmt.Printf("observed violation, alice = %v, bob = %v, sum = %v\n", alice, bob, alice+bob)
		}
		mu.Unlock()
	}
}

/*
保护不变值，加锁的正确写法
unlock happens before lock
 */
func perField2() {
	alice := 10000
	bob := 10000
	var mu sync.Mutex

	total := alice + bob

	go func() {
		for i := 0; i < 1000; i++ {
			mu.Lock()
			alice--
			bob++
			mu.Unlock()
		}
	}()

	go func() {
		for i := 0; i < 1000; i++ {
			mu.Lock()
			bob--
			alice++
			mu.Unlock()
		}
	}()

	start := time.Now()
	for time.Since(start) < time.Second {
		mu.Lock()
		if alice + bob != total {
			fmt.Printf("observed violation, alice = %v, bob = %v, sum = %v\n", alice, bob, alice+bob)
		}
		mu.Unlock()
	}
}

/*
并发等待投票正确写法
*/
func waitVotes() {
	rand.Seed(time.Now().UnixNano())

	count := 0
	finished := 0
	var mu sync.Mutex

	for i := 0; i < 10; i++ {
		go func() {
			vote := requestVote()
			mu.Lock()
			defer mu.Unlock()
			if vote {
				count++
			}
			finished++
		}()
	}
	for {
		mu.Lock()

		if count >= 5 || finished == 10 {
			break
		}
		mu.Unlock()
		// 防止该 for 循环占用 100% cpu
		time.Sleep(time.Millisecond * 50)
	}
	if count >= 5 {
		println("recevied 5+ votes!")
	} else {
		println("lost")
	}
	mu.Unlock()
}

/*
并发等待投票正确写法2
*/
func waitVotes2() {
	rand.Seed(time.Now().UnixNano())

	// 共享数据 count 和 finished
	count := 0
	finished := 0
	var mu sync.Mutex
	// 将 mu 传递给 cond，成为 cond 内部的锁
	cond := sync.NewCond(&mu)

	for i := 0; i < 10; i++ {
		go func() {
			vote := requestVote()
			mu.Lock()
			defer mu.Unlock()
			if vote {
				count++
			}
			finished++
			cond.Broadcast()
		}()
	}
	// 这里其实相当于 cond.L.Lock()
	mu.Lock()
	for count < 5 && finished != 10 {
		// cond.Wait() 会一直阻塞直到调用 cond.Broadcast() 之后，然后继续循环等待
		cond.Wait()
	}
	if count >= 5 {
		println("recevied 5+ votes!")
	} else {
		println("lost")
	}
	mu.Unlock()
}

func requestVote() bool {
	return rand.Int31() % 2 == 0
}

/*
通道例子
是一种同步的数据结构
 */
func channelExample() {
	c := make(chan bool)
	go func() {
		time.Sleep(time.Second)
		<-c
		//c <- true
	}()
	start := time.Now()
	//<- c
	c <- true // blocks until other goroutine receives
	fmt.Printf("send took %v\n", time.Since(start))
}

/*
生产者消费者模式用通道实现
 */

func produceConsume() {
	c := make(chan int)
	for i := 0; i < 4; i++ {
		go doWork(c)
	}

	for {
		v := <-c
		println(v)
	}
}

func doWork(c chan int) {
	for {
		time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
		c <- rand.Int()
	}
}

/*
下面是找 bug。
 */

type RequestVoteArgs struct {
	Term int
	CandidateId	int
}
type State string
const (
	Follower State = "follower"
	Candidate = "candidate"
	Leader = "leader"
)
type Raft struct {
	mu        sync.Mutex          // Lock to protect shared access to this peer's state
	me int
	peers []int
	state State

	currentTerm int
	votedFor int
}

/*
第一个问题
 */

func (rf *Raft) AttemptElection() {
	rf.mu.Lock()
	rf.state = Candidate
	rf.currentTerm++
	rf.votedFor = rf.me
	log.Printf("[%d] attempting an election at term %d", rf.me, rf.currentTerm)
	rf.mu.Unlock()

	for _, server := range rf.peers {
		if server == rf.me {
			continue
		}
		go func(server int) {
			voteGranted := rf.CallRequestVote(server)
			if !voteGranted {
				return
			}
			// ... tally the notes
		}(server)
	}
}

func (rf *Raft) CallRequestVote(server int) bool {
	// sendRequestVote 这个方法里会有 rpc 调用，不应该把这个方法也放到锁里面，
	// 因为如果卡住，那么这个 rf.mu 就一直不会解锁, 别的地方也无法获取这个锁，造成死锁。
	rf.mu.Lock()
	defer rf.mu.Unlock()
	log.Printf("[%d] sending request vote to %d", rf.me, server)
	args := RequestVoteArgs {
		Term: rf.currentTerm,
		CandidateId: rf.me,
	}
	var reply raft.RequestVoteReply
	ok := rf.sendRequestVote(server, &args, &reply)
	log.Printf("[%d] finish sending request vote to %d", rf.me, server)
	if !ok {
		return false
	}
	return true
}

/*
第一个问题正确的写法
 */

func (rf *Raft) AttemptElection2() {
	rf.mu.Lock()
	rf.state = Candidate
	rf.currentTerm++
	rf.votedFor = rf.me
	log.Printf("[%d] attempting an election at term %d", rf.me, rf.currentTerm)
	term := rf.currentTerm
	rf.mu.Unlock()

	for _, server := range rf.peers {
		if server == rf.me {
			continue
		}
		go func(server int) {
			voteGranted := rf.CallRequestVote2(server, term)
			if !voteGranted {
				return
			}
			// ... tally the notes
		}(server)
	}
}

func (rf *Raft) CallRequestVote2(server, term int) bool {
	// 去掉锁，改成从外部获取数据
	log.Printf("[%d] sending request vote to %d", rf.me, server)
	args := RequestVoteArgs {
		Term: term,
		CandidateId: rf.me,
	}
	var reply raft.RequestVoteReply
	ok := rf.sendRequestVote(server, &args, &reply)
	log.Printf("[%d] finish sending request vote to %d", rf.me, server)
	if !ok {
		return false
	}
	return true
}

/*
第二个问题
该方法会可能出现两个同时为 leader 的情况
 */

func (rf *Raft) AttemptElection3() {
	rf.mu.Lock()
	rf.state = Candidate
	rf.currentTerm++
	rf.votedFor = rf.me
	log.Printf("[%d] attempting an election at term %d", rf.me, rf.currentTerm)
	term := rf.currentTerm
	done := false
	votes := 1
	rf.mu.Unlock()

	for _, server := range rf.peers {
		if server == rf.me {
			continue
		}
		go func(server int) {
			voteGranted := rf.CallRequestVote2(server, term)
			if !voteGranted {
				return
			}
			rf.mu.Lock()
			defer rf.mu.Unlock()
			votes++
			log.Printf("[]%d got vote from %d", rf.me, server)
			if done || votes <= len(rf.peers) / 2 {
				return
			}
			done = true
			log.Printf("[%d] we got enough votes, we are now the leader （currentTerm = %d, state = %v)! ", rf.me, rf.currentTerm, rf.state)
			rf.state = Leader
		}(server)
	}
}

/*
第二个问题的修复
 */

func (rf *Raft) AttemptElection4() {
	rf.mu.Lock()
	rf.state = Candidate
	rf.currentTerm++
	rf.votedFor = rf.me
	log.Printf("[%d] attempting an election at term %d", rf.me, rf.currentTerm)
	term := rf.currentTerm
	done := false
	votes := 1
	rf.mu.Unlock()

	for _, server := range rf.peers {
		if server == rf.me {
			continue
		}
		go func(server int) {
			voteGranted := rf.CallRequestVote2(server, term)
			if !voteGranted {
				return
			}
			rf.mu.Lock()
			defer rf.mu.Unlock()
			votes++
			log.Printf("[]%d got vote from %d", rf.me, server)
			if done || votes <= len(rf.peers) / 2 {
				return
			}
			done = true
			// 双重检测，防止现在的 state 和 term 和上面的加锁部分的环境不是一个了
			// 可能这个时候是别的协程的数据
			if rf.state != Candidate || rf.currentTerm != term {
				return
			}
			log.Printf("[%d] we got enough votes, we are now the leader （currentTerm = %d, state = %v)! ", rf.me, rf.currentTerm, rf.state)
			rf.state = Leader
		}(server)
	}
}
```

## Debugging

util.go

1.  设置 Debug = 1

    func DPrintf()

    调用 DPrintf()
2. 按住 ctrl + backslash， 会终止程序并且打印栈信息。
3.  运行 go test -race -run 2A

    检测数据竞争。
4. 在日志里加入特殊标志，表示需要注意的特殊情况发生。

./go-test-many.sh 1000 4 2A
