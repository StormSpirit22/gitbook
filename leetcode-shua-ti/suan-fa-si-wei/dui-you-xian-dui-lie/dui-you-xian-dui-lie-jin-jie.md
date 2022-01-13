# 堆（优先队列）进阶

本文要介绍的题：

[数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

[前 K 个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

[超级丑数](https://leetcode-cn.com/problems/super-ugly-number/)

[丑数 II](https://leetcode-cn.com/problems/ugly-number-ii/)

## 题解

### 数组中的第 K 个最大元素

> 给定整数数组 nums 和整数 k，请返回数组中第 k 个最大的元素。
>
> 请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。
>
> 示例 1:
>
> ```
> 输入: [3,2,1,5,6,4] 和 k = 2
> 输出: 5
> ```

解析：

这题一看不是很简单吗，直接排个序然后遍历第 k 个元素就行了。但这题如果用标准库的排序来做就不是中等难度了。题目主要就是想考

排序，快速排序或者堆排序都行。快速排序在这里就不说了，后面代码只写一个堆排序。

### 前 K 个高频元素

> 给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。
>
> **示例 1:**
>
> ```
> 输入: nums = [1,1,1,2,2,3], k = 2
> 输出: [1,2]
> ```

解析：

这题明显就是考优先队列，出现频率就是队列的优先级，用标准库里的堆构造一个优先队列，一个个出队列即可。

### 超级丑数

> 超级丑数 是一个正整数，并满足其所有质因数都出现在质数数组 primes 中。
>
> 给你一个整数 n 和一个整数数组 primes ，返回第 n 个 超级丑数 。
>
> 题目数据保证第 n 个 超级丑数 在 32-bit 带符号整数范围内。
>
> 示例 1：
>
> ```
> 输入：n = 12, primes = [2,7,13,19]
> 输出：32 
> 解释：给定长度为 4 的质数数组 primes = [2,7,13,19]，前 12 个超级丑数序列为：[1,2,4,7,8,13,14,16,19,26,28,32] 。
> ```

解析：

开始看这题有点绕。其实意思就是 primes 数组里的数字循环相乘即可，这样出来的数不就是所有的质因数都在 primes 数组中了。我们

先构造一个堆，插入一个元素 1，然后从 0 遍历到 n，每次取出堆顶的元素，然后将该元素与 primes 数组中的数相乘，再分别加入堆

中，要注意这里可能出现重复的数字，所以要加一个 visited 标记是否加入过堆。由于堆的数据结构比较复杂，当 n 特别大时可能会超、

时，就算拿官方题解放进去也一样超时😂。只能用动态规划的方法来提交，但这里就不做介绍了，可以看看[题解](https://leetcode-cn.com/problems/super-ugly-number/solution/chao-ji-chou-shu-by-leetcode-solution-uzff/)。

### 丑数II

> 给你一个整数 `n` ，请你找出并返回第 `n` 个 **丑数** 。
>
> **丑数** 就是只包含质因数 `2`、`3` 和/或 `5` 的正整数。
>
> 示例 1：
>
> ```
> 输入：n = 10
> 输出：12
> 解释：[1, 2, 3, 4, 5, 6, 8, 9, 10, 12] 是由前 10 个丑数组成的序列。
> ```

解析：

这题就比上题还简单了，primes 数组就是 \[2,3,5]。其他的不变。这题和上一题其实还可以用动态规划的方法来做，这里不做介绍。

## 代码

### 数组中的第 K 个最大元素

```go
func findKthLargest(nums []int, k int) int {
    heapSize := len(nums)
    buildMaxHeap(nums, heapSize)
    for i := len(nums) - 1; i >= len(nums) - k + 1; i-- {
        nums[0], nums[i] = nums[i], nums[0]
        heapSize--
        maxHeapify(nums, 0, heapSize)
    }
    return nums[0]
}

func buildMaxHeap(a []int, heapSize int) {
    for i := heapSize/2; i >= 0; i-- {
        maxHeapify(a, i, heapSize)
    }
}

func maxHeapify(a []int, i, heapSize int) {
    l, r, largest := i * 2 + 1, i * 2 + 2, i
    if l < heapSize && a[l] > a[largest] {
        largest = l
    }
    if r < heapSize && a[r] > a[largest] {
        largest = r
    }
    if largest != i {
        a[i], a[largest] = a[largest], a[i]
        maxHeapify(a, largest, heapSize)
    }
}
```

或者直接借助标准库的 heap

```go
func findKthLargest(nums []int, k int) int {
    h := &IntHeap{}
    var res int
    for i := range nums {
        heap.Push(h, nums[i])
    }
    for i := 0; i < k; i++ {
        res = heap.Pop(h).(int)
    }
    return res
}

// IntHeap 是一个由整数组成的最大堆。
type IntHeap []int

func (h IntHeap) Len() int {return len(h)}
func (h IntHeap) Less(i, j int) bool {return h[i] > h[j]}
func (h IntHeap) Swap(i, j int) {h[i], h[j] = h[j], h[i]}


func (h *IntHeap) Push(x interface{}) {
    *h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0:n-1]
    return x
}
```

### 前 K 个高频元素

```go
func topKFrequent(nums []int, k int) []int {
    pq := PQs{}
    numMap := make(map[int]int)
    for _, n := range nums {
        numMap[n] += 1
    }
    for k, v := range numMap {
        f := &Frequency{frequency: v, num: k}
        heap.Push(&pq, f)
    }
    var res []int
    for i := 0; i < k; i++ {
        f := heap.Pop(&pq).(*Frequency)
        res = append(res, f.num)
    }
    return res
}

type PQs []*Frequency
type Frequency struct {
    num int
    frequency int
}

func (p PQs) Len() int { return len(p)}
func (p PQs) Less(i, j int) bool { return p[i].frequency > p[j].frequency }
func (p PQs) Swap(i, j int) { p[i], p[j] = p[j], p[i] }
func (p *PQs) Push(x interface{}) {
    *p = append(*p, x.(*Frequency))
}
func (p *PQs) Pop() interface{} {
    old := *p
    n := len(old)
    x := old[n-1]
    *p = old[:n-1]
    return x
}
```

### 超级丑数

```go
func nthSuperUglyNumber(n int, primes []int) int {
    h := hp{}
    heap.Push(&h, 1)
    visited := make(map[int]bool)
    var ugly int
    for i := 0; i < n; i++ {
        ugly = heap.Pop(&h).(int)
        for j := range primes {
            next := ugly * primes[j]
            // 判断是否加入过
            if !visited[next] {
                heap.Push(&h, next)
                visited[next] = true
            }
        }
    }
    return ugly
}

type hp []int
func (h hp) Len() int {return len(h)}
func (h hp) Less(i, j int) bool {return h[i] < h[j]}
func (h hp) Swap(i, j int) {h[i], h[j] = h[j], h[i]}

func (h *hp) Push(x interface{}) {
    *h = append(*h, x.(int))
}

func (h *hp) Pop() interface{} {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}
```

### 丑数II

```go
func nthUglyNumber(n int) int {
    count := 1
    primes := []int{2, 3, 5}
    h := hp{1}
    ans := 1
    visited := make(map[int]bool)
    for count <= n {
        ans = heap.Pop(&h).(int)
        if count == n {
            return ans
        }
        for _, p := range primes {
            tmp := ans * p
            if !visited[tmp] {
                heap.Push(&h, tmp)
                visited[tmp] = true
            }
        }
        count++
    }
    return ans
}

type hp []int
func (h hp) Len() int {return len(h)}
func (h hp) Less(i, j int) bool {return h[i] < h[j]}
func (h hp) Swap(i, j int) {h[i], h[j] = h[j], h[i]}

func (h *hp) Push(x interface{}) {
    *h = append(*h, x.(int))
}

func (h *hp) Pop() interface{} {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}
```
