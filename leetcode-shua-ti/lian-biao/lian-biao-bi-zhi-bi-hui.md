# 链表进阶

这次看看链表难度稍微大一点的题。

本文要介绍的题目有：‌

[删除排序链表中的重复元素 II（中等）](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

[删除链表的倒数第 N 个结点（中等）](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

[分隔链表（中等）](https://leetcode-cn.com/problems/partition-list/)

[排序链表（中等）](https://leetcode-cn.com/problems/sort-list/)

[旋转链表（中等）](https://leetcode-cn.com/problems/rotate-list/)

## 题目解析

### 删除排序链表中的重复元素II

> 存在一个按升序排列的链表，给你这个链表的头节点 head ，请你删除链表中所有存在数字重复情况的节点，只保留原始链表中 没有重复出现 的数字。
>
> 返回同样按升序排列的结果链表。

解析：

需要一个辅助节点 dummy node，最后返回 dummy.Next。

如果 `cur.Next == cur.Next.Next`，则用 x 记录这个重复的值 ，然后第二层循环迭代判断节点值是否等于 x，等于则删除节点， `cur.Next = cur.Next.Next` ，如果遇到不重复的值了则继续下一个节点遍历 `cur = cur.Next`。

### 删除链表的倒数第 N 个节点

> 给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。
>
> \*\*进阶：\*\*你能尝试使用一趟扫描实现吗？

解析：

还是需要一个辅助节点，使用 slow，fast 双指针，fast 先跑 n 步，再一起跑，fast 为 nil 时，slow 就到了 n 的前一个节点，删除即可。

这就是一趟扫描的方法。

### 分隔链表

> 给你一个链表的头节点 head 和一个特定值 x ，请你对链表进行分隔，使得所有 小于 x 的节点都出现在 大于或等于 x 的节点之前。
>
> 你应当 保留 两个分区中每个节点的初始相对位置。
>
> ```
> 输入：head = [1,4,3,2,5,2], x = 3
> 输出：[1,2,2,4,3,5]
> ```

解析：

我们可以维护两个链表，一个链表的节点都是比 x 小，另一个都比 x 大，最后拼接两个链表即可。不过这里有个细节，就是大链表最后next 要赋值为nil，不然可能成环，用题目的例子就会出现这种问题。另外就是遍历及拼接的细节，可以看下代码。

### 排序链表

> 给你链表的头结点 head ，请将其按 升序 排列并返回 排序后的链表 。
>
> 进阶：
>
> 你可以在 O(n log n) 时间复杂度和常数级空间复杂度下，对链表进行排序吗？
>
> ```
> 输入：head = [4,2,1,3]
> 输出：[1,2,3,4]
> ```

解析：

归并排序，先找到链表中点，分成两个链表，再递归，最后合并两个有序链表。

但是里面有几点细节需要注意，一是找中点节点，二是递归base条件判断，这两点没注意好很容易出错。总之多写就会啦。

### 旋转链表

> 给你一个链表的头节点 `head` ，旋转链表，将链表每个节点向右移动 `k` 个位置。
>
> ```
> 输入：head = [1,2,3,4,5], k = 2
> 输出：[4,5,1,2,3]
> ```

解析：

先计算出链表长度 count，再将链表最后一个节点连接到头结点，将链表成环。 简化 k 值，k = k % count，

遍历到 count-1-k 个节点再断开链表即可。

## 代码

### 删除排序链表中的重复元素II

```go
func deleteDuplicates(head *ListNode) *ListNode {
  if head == nil {
        return nil
    }
    dummy := &ListNode{Val: -1, Next: head}
    cur := dummy
    for cur.Next != nil && cur.Next.Next != nil {
        if cur.Next.Val == cur.Next.Next.Val {
            x := cur.Next.Val
            for cur.Next != nil && cur.Next.Val == x {
                cur.Next = cur.Next.Next
            }
        } else {
            cur = cur.Next
        }
    }
    return dummy.Next
}
```

### 删除链表的倒数第 N 个节点

```go
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    prev := &ListNode{Next: head}
    slow, fast := prev, head
    for i := 0; i < n; i++ {
        fast = fast.Next
    }
    for fast != nil {
        fast = fast.Next
        slow = slow.Next
    }
    slow.Next = slow.Next.Next
    return prev.Next
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

### 分隔链表

```go
func partition(head *ListNode, x int) *ListNode {
    small := &ListNode{}
    smallHead := small
    large := &ListNode{}
    largeHead := large
    for head != nil {
        if head.Val < x {
            small.Next = head
            small = small.Next
        } else {
            large.Next = head
            large = large.Next
        }
        head = head.Next
    }
    // 这里许要主动赋值为 nil，不然可能成环
    large.Next = nil
    // 连接两个链表，此时 small 和 large 分别为两个链表的最后一个节点
    small.Next = largeHead.Next
    return smallHead.Next
}
```

### 排序链表

```go
func sortList(head *ListNode) *ListNode {
    return mergeSort(head)
}

func mergeSort(head *ListNode) *ListNode {
    //需要判断head和head.Next，不然会死循环。因为如果链表只有一个值，那么会一直递归下去。
    if head == nil || head.Next == nil {
        return head
    }
    mid := findMiddle(head)
    tail := mid.Next
    mid.Next = nil
    left := mergeSort(head)
    right := mergeSort(tail)
    return mergeTwoLists(left, right)
}

func findMiddle(head *ListNode) *ListNode {
    //这里注意fast应该赋值为head.Next，因为如果传入只有2个节点的链表，mid一直会返回第二个节点，造成死循环。
    slow, fast := head, head.Next
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
    }
    return slow
}

func mergeTwoLists(l, r *ListNode) *ListNode {
    head := &ListNode{}
    tmp := head
    for l != nil && r != nil {
        if l.Val < r.Val {
            tmp.Next = l
            l = l.Next
        } else {
            tmp.Next = r
            r = r.Next
        }
        tmp = tmp.Next
    }
    if l != nil {
        tmp.Next = l
    }
    if r != nil {
        tmp.Next = r
    }
    return head.Next
}
```

### 旋转链表

```go
func rotateRight(head *ListNode, k int) *ListNode {
    if head == nil || k == 0 {
        return head
    }
    count := 1
    cur := head
    for cur.Next != nil {
        count++
        cur = cur.Next
    }
    cur.Next = head
    k = k % count
    cur = head
    for i := 0; i < count-1-k; i++ {
        cur = cur.Next
    }
    newHead := cur.Next
    cur.Next = nil
    return newHead
}
```
