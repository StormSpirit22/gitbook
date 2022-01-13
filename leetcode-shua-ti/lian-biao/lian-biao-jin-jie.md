# 链表高阶

这次来看看链表难度更大一点的题，其实了解了会发现并不难。

‌

本文要介绍的题目有：‌

[反转链表 II（中等）](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

[K 个一组翻转链表（困难）](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

[复制带随机指针的链表（中等）](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

## 题目解析

### 反转链表II

> 给你单链表的头指针 head 和两个整数 left 和 right ，其中 left <= right 。请你反转从位置 left 到位置 right 的链表节点，返回 反转后的链表 。
>
> ```
> 输入：head = [1,2,3,4,5], left = 2, right = 4
> 输出：[1,4,3,2,5]
> ```

解析：

需要一个辅助节点 dummy node，最后返回 dummy.Next。

这题比反转整个链表要复杂些，我们可以用几个变量把要反转的中间链表的前节点和后节点保存起来，然后拼接。

或者我们可以把这个问题拆解成两部分，先遍历到 left 的位置，然后我们反转以 left 为头节点的链表前 right-left+1 个节点。

先写一个 `func reverseN(head *ListNode, n int) *ListNode` 函数，表示**反转链表的前 n 个节点**，那么我们遍历原链表到 left 位置。

然后调用 `newHead := reverseN(left, right-left+1)`，再将 left 前一个节点 pre 链接到返回的节点就可以了，即 `pre.Next = newHead`。这样逻辑比普通迭代要清晰很多。

### K个一组翻转链表

> 给你一个链表，每 _k_ 个节点一组进行翻转，请你返回翻转后的链表。
>
> _k_ 是一个正整数，它的值小于或等于链表的长度。
>
> 如果节点总数不是 _k_ 的整数倍，那么请将最后剩余的节点保持原有顺序。
>
> **进阶：**
>
> * 你可以设计一个只使用常数额外空间的算法来解决此问题吗？
> * **你不能只是单纯的改变节点内部的值**，而是需要实际进行节点交换。
>
> ```
> 输入：head = [1,2,3,4,5], k = 3
> 输出：[3,2,1,4,5]
> ```

解析：

这题就得借助上一题的**反转链表的前 n 个节点**方法了，不然太麻烦。遍历链表，不断地调用 reverseN 函数即可。要注意的是需要先对 k 个节点进行遍历，如果不够 k 个节点反转就直接返回了。

### 复制带随机指针的链表

> 给你一个长度为 n 的链表，每个节点包含一个额外增加的随机指针 random ，该指针可以指向链表中的任何节点或空节点。
>
> 构造这个链表的 深拷贝。 深拷贝应该正好由 n 个 全新 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 next 指针和 random 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。复制链表中的指针都不应指向原链表中的节点 。
>
> 例如，如果原链表中有 X 和 Y 两个节点，其中 X.random --> Y 。那么在复制链表中对应的两个节点 x 和 y ，同样有 x.random --> y 。
>
> 返回复制链表的头节点。
>
> 用一个由 n 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 \[val, random\_index] 表示：
>
> val：一个表示 Node.val 的整数。 random\_index：随机指针指向的节点索引（范围从 0 到 n-1）；如果不指向任何节点，则为 null 。 你的代码 只 接受原链表的头节点 head 作为传入参数。
>
> ```
> 输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
> 输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
> ```

解析：

这题很经典，如果没做过估计很难想出来，官方有三种解法，我这里就选最后一种最好理解的来做。

简单的说就是在原链表中每个节点后面都跟上一个clone节点，val和上个节点一样，next 指向 cur.Next，cur.Next 再指向 clone。

之后处理 random 节点，重新遍历新链表，很巧妙的点是 `cur.Next.Random = cur.Random.Next`，只要理解这个就好办了。

最后再把两个链表拆分，拆分过程也是有技巧的，具体可以看下代码。

## 代码

### 反转链表II

普通迭代

```go
func reverseBetween(head *ListNode, m int, n int) *ListNode {
    if head == nil {
        return nil
    }
     // 因为头节点有可能发生变化，使用虚拟头节点可以避免复杂的分类讨论
    dummy := &ListNode{Next: head}
    head = dummy
    pre := head
    for i := 0; i < m; i++ {
        pre = head
        head =  head.Next
    }
    tail := head
    var last, next *ListNode
    for i := m; i <= n; i++ {
        if head != nil {
            next = head.Next
            head.Next = last
            last = head
            head = next
        }
    }
    tail.Next = next
    pre.Next = last
    return dummy.Next
}
```

拆分方法

```go
func reverseBetween(head *ListNode, m int, n int) *ListNode {
    if head == nil {
        return nil
    }
     // 因为头节点有可能发生变化，使用虚拟头节点可以避免复杂的分类讨论
    dummy := &ListNode{Next: head}
    head = dummy
    pre := head
    for i := 0; i < m; i++ {
        pre = head
        head =  head.Next
    }
    // 反转前 n-m+1 个节点
    newHead := reverseN(head, n-m+1)
    pre.Next = newHead
    return dummy.Next
}

func reverseN(head *ListNode, n int) *ListNode {
    if head == nil {
        return nil
    }
    pre := head
    var cur *ListNode
    for i := 0; head != nil && i < n; i++ {
        next := head.Next
        head.Next = cur
        cur = head
        head = next
    }
    pre.Next = head
    return cur
}
```

### K个一组翻转链表

```go
func reverseKGroup(head *ListNode, k int) *ListNode {
    dummy := &ListNode{Next: head}
    pre := dummy
    for head != nil {
        test := pre
        for i := 0; i < k; i++ {
            test = test.Next
            if test == nil {
                return dummy.Next
            }
        }
        pre.Next = reverseN(head, k)
        pre = head
        head = head.Next
    }
    return dummy.Next
}

func reverseN(head *ListNode, n int) *ListNode {
    if head == nil {
        return nil
    }
    pre := head
    var cur *ListNode
    for i := 0; head != nil && i < n; i++ {
        next := head.Next
        head.Next = cur
        cur = head
        head = next
    }
    pre.Next = head
    return cur
}
```

### 复制带随机指针的链表

```go
func copyRandomList(head *Node) *Node {
    if head == nil {
        return nil
    }
    cur := head
    for cur != nil {
        cloneNode := &Node{Val: cur.Val, Next: cur.Next}
        cur.Next = cloneNode
        cur = cur.Next.Next
    }
    cur = head
    for cur != nil {
        if cur.Random != nil {
            cur.Next.Random = cur.Random.Next
        }
        cur = cur.Next.Next
    }
    newHead := head.Next
    cur = head
    for cur != nil && cur.Next != nil {
        tmp := cur.Next
        cur.Next = cur.Next.Next
        cur = tmp
    }
    return newHead
}
```
