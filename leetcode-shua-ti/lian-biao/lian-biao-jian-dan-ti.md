# 链表初阶

链表通常由一连串节点组成，每个节点包含任意的实例数据（data fields）和一或两个用来指向上一个/或下一个节点的位置的链接。

链表最明显的好处就是，常规数组排列关联项目的方式可能不同于这些数据项目在记忆体或磁盘上顺序，数据的访问往往要在不同的排列

顺序中转换。而链表是一种自我指示数据类型，因为它包含指向另一个相同类型的数据的指针（链接）。链表允许插入和移除表上任意位

置上的节点，但是不允许随机存取。链表有很多种不同的类型：**单向链表，双向链表以及循环链表**。

‌

本文要介绍的题目有：‌

[删除排序链表中的重复元素（简单）](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

[反转链表（简单）](https://leetcode-cn.com/problems/reverse-linked-list/)‌

[合并两个有序链表（简单）](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

[回文链表（简单）](https://leetcode-cn.com/problems/palindrome-linked-list/)

## 题目解析

### 删除排序链表中的重复元素

> 存在一个按升序排列的链表，给你这个链表的头节点 `head` ，请你删除所有重复的元素，使每个元素 **只出现一次** 。
>
> 返回同样按升序排列的结果链表。

解析：

这题意思是删除**已排序**链表所有重复的元素，那么我们只要遍历链表的时候如果碰到当前节点的值和前一个节点的值相等，那么就将前一个节点的Next指向下一个节点即可。

### 反转链表

> 给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

解析：

这题用迭代或者递归。

迭代的方法，需要三个临时变量，pre、cur、next，分别表示前一个节点、当前节点和后一个节点，开始遍历的时候只需要用 next 存储 cur.Next，再 `cur.Next = pre，cur = next, pre = cur` 即可。

递归的方法，我们之前在二叉树模块里已经说过了，递归的思想就是”你需要相信这个函数一定能实现我们的功能“。

那我们假设 `last := reverseList(head.Next)` 已经将 head.Next 及之后的节点都反转了，如下图所示：

![](<../../.gitbook/assets/image (1) (1).png>)

由于该函数的定义是，**返回的节点是反转后的链表的头结点**，那么用 last 接收的值就是最后要返回的节点值。

此时只剩下 head 节点没有反转，我们现在要做的就是将 head 反转，即 `head.Next.Next = head, head.Next = nil`，这样就行了。这题如果跳进递归很难跳出来，最好就是基于我们的原则，相信它能完成反转，我们要做的就是处理最后一种情况和 base case。

该题的 base case 就是 `head == nil || head.Next == nil`，可以知道能通过这个条件返回的其实就只有最后一个节点，所以 last 节点一直没变。就是我们要的返回值。

### 合并两个有序链表

> 将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

解析：

两个有序链表或者两个有序数组都是同一类型的题目，用双指针解法就行。也是有迭代或者递归两种方法。

迭代的方法，初始化一个新的链表节点 head，l1 与 l2 判断大小，谁小就连在 head 后面，再循环迭代下去。

‌递归的方法，同样我们需要相信方法能完成我们的功能，我们只需要定义 base case 和处理一种情况。

base case 是 l1 == nil 则返回 l2 或 l2 == nil 则返回 l1。

要处理的情况是：如果 `l1.Val < l2.Val`，那么 `l1.Next = mergeTwoLists(l1.Next, l2)`。

什么意思呢？如果 `l1.Val < l2.Val`，那么要返回的肯定是 l1 这个节点，l1 这个节点就是新链表的第一个有序节点了，之后的事情就是 l1.Next 与 l2 继续去递归就行了。同样当 `l1.Val >= l2.Val`时，只需要将 l1 和 l2 的位置换一下。

或者我们可以将 l1 和 l2 简化为都只有 1 个节点。是不是就好理解这个代码啦，如果只有一个节点，上面的代码其实就是 `l1.Next = l2`，再返回 l1 即可。

### 回文链表

> 请判断一个链表是否为回文链表。
>
> ```
> 输入: 1->2->2->1
> 输出: true
> ```

解析：

这题最简单的方法是将链表的值都存下来放到一个数组里，然后用双指针法判断数组是否是回文的。

如果不借助数组，可以用迭代或者递归两种方法来解决。

迭代：

用快慢指针法找到链表中点，将链表断开成两部分，再把第二部分反转一下，再判断这两部分是否是相等的链表即可。方法有点繁琐且细节多，容易出错。

递归：

递归方法比较巧妙，有点类似**回溯**的思想。

left 全局变量赋值为 head， right 赋值为head，先利用递归函数将 right 递归到最后一个节点，然后比较 left 和 right 的值，如果不相等则返回 false。

接着 `left = left.Next`，返回 true，此时函数返回到上一层，right 就是倒数第二个节点，继续与第二个节点 left 比较，以此类推。

## 代码

### 删除排序链表中的重复元素

```go
func deleteDuplicates(head *ListNode) *ListNode {
    current := head 
    for current != nil {
        for current.Next != nil && current.Val == current.Next.Val {
            current.Next = current.Next.Next
        }
        current = current.Next
    }
    return head
}
```

### 反转链表

迭代：

```go
func reverseList(head *ListNode) *ListNode {
  if head == nil || head.Next == nil {
        return head
    }
    var pre *ListNode
    cur := head
    for cur != nil {
        next := cur.Next
        cur.Next = pre
        pre = cur
        cur = next
    }
    return pre
}
```

递归：

```go
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    last := reverseList(head.Next)
    head.Next.Next = head
    head.Next = nil    
    return last
}
```

### 合并两个有序链表

迭代：

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
  head := &ListNode{}
    cur := head
    for l1 != nil && l2 != nil {
        if l1.Val < l2.Val {
            cur.Next = l1
            l1 = l1.Next
        } else {
            cur.Next = l2
            l2 = l2.Next
        }
        cur = cur.Next
    }
    if l1 != nil {
        cur.Next = l1
    }
    if l2 != nil {
        cur.Next = l2
    }
    return head.Next
}
```

递归：

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
  if l1 == nil {
        return l2
    }
    if l2 == nil {
        return l1
    }
    if l1.Val < l2.Val {
        l1.Next = mergeTwoLists(l1.Next, l2)
        return l1
    } else {
        l2.Next = mergeTwoLists(l1, l2.Next)
        return l2
    }
    return nil
}
```

### 回文链表

迭代：

```go
func isPalindrome(head *ListNode) bool {
    if head == nil {
        return true
    }
    slow, fast := head, head.Next
    // fast如果初始化为head.Next则中点在slow.Next
    // fast初始化为head,则中点在slow
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
    }
    tail := reverse(slow.Next)
    slow.Next = nil
    for head != nil && tail != nil {
        if head.Val != tail.Val {
            return false
        }
        head = head.Next
        tail = tail.Next
    }
    return true
}

func reverse(head *ListNode) *ListNode {
    cur := head
    var prev *ListNode
    for cur != nil {
        next := cur.Next
        cur.Next = prev
        prev = cur
        cur = next
    }
    return prev
}
```

递归：

```go
var left *ListNode
func isPalindrome(head *ListNode) bool {
    left = head
    return traverse(head)
}

func traverse(right *ListNode) bool {
    if right == nil {
        return true
    }
    res := traverse(right.Next)
    if !res || left.Val != right.Val {
        return false
    }
    left = left.Next
    return true
}
```
