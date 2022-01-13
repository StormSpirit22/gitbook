# BFS 初阶

这篇主要来介绍 BFS 的思想及一些题。

**广度优先搜索算法**（英语：Breadth-First Search，缩写为BFS），又译作**宽度优先搜索**，或**横向优先搜索**，是一种[图形搜索算法](https://zh.wikipedia.org/wiki/%E6%90%9C%E7%B4%A2%E7%AE%97%E6%B3%95)。简单的

说，BFS是从\[根节点]\([https://zh.wikipedia.org/wiki/树\_(数据结构)#术语)开始，沿着树的宽度遍历树的\[节点\](https://zh.wikipedia.org/wiki/节点)。如果所有节点均被访问，则算法中止。](https://zh.wikipedia.org/wiki/%E6%A0%91\_\(%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84\)#%E6%9C%AF%E8%AF%AD\)%E5%BC%80%E5%A7%8B%EF%BC%8C%E6%B2%BF%E7%9D%80%E6%A0%91%E7%9A%84%E5%AE%BD%E5%BA%A6%E9%81%8D%E5%8E%86%E6%A0%91%E7%9A%84\[%E8%8A%82%E7%82%B9]\(https://zh.wikipedia.org/wiki/%E8%8A%82%E7%82%B9\)%E3%80%82%E5%A6%82%E6%9E%9C%E6%89%80%E6%9C%89%E8%8A%82%E7%82%B9%E5%9D%87%E8%A2%AB%E8%AE%BF%E9%97%AE%EF%BC%8C%E5%88%99%E7%AE%97%E6%B3%95%E4%B8%AD%E6%AD%A2%E3%80%82)

其实最简单的一个例子就是用队列实现的二叉树的层次遍历，BFS 算法基本都是围绕着队列这一数据结构来进行的。

BFS 的核心思想应该不难理解的，就是把一些问题抽象成图，从一个点开始，向四周开始扩散。一般来说，我们写 BFS 算法都是用「队列」这种数据结构，每次将一个节点周围的所有节点加入队列。

BFS 相对 DFS 的最主要的区别是：**BFS 找到的路径一定是最短的，但代价就是空间复杂度比 DFS 大很多**。

这里改编了一下东哥的算法模板， go 版本 BFS 模板：

```go
// 计算从起点 start 到终点 target 的最近距离
func BFS(start Node, target Node) int {
    var q []Node // 核心数据结构
    visited := make(map[Node]struct) // 避免走回头路

    q = append(q, start) // 将起点加入队列
    visited[start] = struct{}{}
    var step int // 记录扩散的步数

    for (q is not empty) {
        n := len(q)
        /* 将当前队列中的所有节点向四周扩散 */
        for (i := 0; i < n; i++) {
            cur := q[len(q)-1]
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step
            /* 将 cur 的相邻节点加入队列 */
            for (x is cur.adj())
                if (x not in visited) {
                    q = append(q, x)
                    visited[x] = struct{}{}
                }
        }
        /* 划重点：更新步数在这里 */
        step++
    }
}
```

队列 `q` 就不说了，BFS 的核心数据结构；`cur.adj()` 泛指 `cur` 相邻的节点，比如说二维数组中，`cur` 上下左右四面的位置就是相邻节

点；`visited` 的主要作用是防止走回头路，大部分时候都是必须的，但是像一般的二叉树结构，没有子节点到父节点的指针，不会走回

头路就不需要 `visited`。

本文要介绍的题：

[二叉树的最小深度（简单）](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)

[打开转盘锁（中等）](https://leetcode-cn.com/problems/open-the-lock/)

## 题解

### 二叉树的最小深度

> 给定一个二叉树，找出其最小深度。
>
> 最小深度是从根节点到最近叶子节点的最短路径上的节点数量。
>
> \*\*说明：\*\*叶子节点是指没有子节点的节点。
>
> ```
> 输入：root = [3,9,20,null,null,15,7]
> 输出：2
> ```

解析：

这题就是很标准的 BFS 算法题，我们使用层次遍历，一层一层遍历，如果某一层出现叶子节点那么它必然是最近的叶子节点，直接返回深度即可。

当然这题其实也可以用 DSF 来做，下面两种解法代码都有。

### 打开转盘锁

> 你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有10个数字： `'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'` 。每个拨轮可以自由旋转：例如把 `'9'` 变为 `'0'`，`'0'` 变为 `'9'` 。每次旋转都只能旋转一个拨轮的一位数字。
>
> 锁的初始数字为 `'0000'` ，一个代表四个拨轮的数字的字符串。
>
> 列表 `deadends` 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。
>
> 字符串 `target` 代表可以解锁的数字，你需要给出解锁需要的最小旋转次数，如果无论如何不能解锁，返回 `-1` 。

解析：

这题刚开始可能不知道该怎么下手，那我们先考虑最简单的情况。

如果我们把所有可能的情况都枚举出来再去和 target 对比。

先从 0000 开始，对于 0000 来说，它有 8 种可能的情况，分别是对这四个数字往上拨或者往下拨。

那么是不是可以把这 8 种情况看作是 0000 这个节点的 8 个相邻的节点，然后用 BFS 的套路来解决呢？

肯定是可以的。我们先将 0000 加入到队列中，然后按照层次遍历树的代码来写就行了，注意这里需要用一个 visitedMap 来标记数字是

否已经转出来过，不然你对同一个位置往上拨往下拨能一直拨下去，还要加上 deadends 和 target 的判断。

另外还有一种时间空间上更简化的方法：双向 BFS 算法。

先从 0000 开始遍历，然后从 target 开始遍历，之后循环，当找到两个遍历的队列有重合的数字时就返回，意味着就找到了一条联通起点和终点的最短路径。

「双向 BFS」的基本实现思路如下：

创建「两个队列」分别用于两个方向的搜索；

创建「两个哈希表」用于「解决相同节点重复搜索」和「记录转换次数」；

为了尽可能让两个搜索方向“平均”，每次从队列中取值进行扩展时，先判断哪个队列容量较少；

如果在搜索过程中「搜索到对方搜索过的节点」，说明找到了最短路径。

双向 BFS 只是一种优化，而且并不是所有 BFS 的题都能这样优化，其实主要能掌握普通的 BFS 算法题模板就可以啦。

## 代码

### 二叉树的最小深度

BFS

```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    depth := 1
    var queue []*TreeNode
    queue = append(queue, root)
    for len(queue) > 0 {
        n := len(queue)
        for i := 0; i < n; i++ {
            tmp := queue[i]
            if tmp.Left == nil && tmp.Right == nil {
                return depth
            }
            if tmp.Left != nil {
                queue = append(queue, tmp.Left)
            }
            if tmp.Right != nil {
                queue = append(queue, tmp.Right)
            }
        }
        queue = queue[n:]
        depth++
    }
    return depth
}
```

DFS

```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    return minDepthDfsHelper(root, 0)
}

// 递归找到左子树的长度和右子树的长度，选最小值
func minDepthDfsHelper(root *TreeNode, depth int) int {
    if root.Left == nil && root.Right == nil {
        return depth + 1
    }
    left, right := math.MaxInt32, math.MaxInt32
    if root.Left != nil {
        left = minDepthDfsHelper(root.Left, depth + 1)
    }
    if root.Right != nil {
        right = minDepthDfsHelper(root.Right, depth + 1)
    }
    return min(left, right)
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### 打开转盘锁

BFS

```go
func openLock(deadends []string, target string) int {
    steps := 0
    var queue []string
    queue = append(queue, "0000")

    visitedMap := make(map[string]struct{})
    deadendMap := make(map[string]struct{})
    for _, d := range deadends {
        deadendMap[d] = struct{}{}
    }

    for len(queue) > 0 {
        n := len(queue)

        for i := 0; i < n; i++ {
            tmp := queue[i]
            if tmp == target {
                return steps
            }
            // 判断是否会被永久锁定。注意不能将 visitedMap 放在这里判断，
            // 因为数组里所有值其实已经被标记过了，放在这里判断会走不下去。
            if _, ok := deadendMap[tmp]; ok {
                continue
            }
            // 对 4 个数字进行旋转
            for j := 0; j < 4; j++ {
                rotated := plusOne(tmp, j)
                // 在这里判断是否被标记过，如果已经标记过就不加入队列中
                if _, ok := visitedMap[rotated]; !ok {
                    // 必须在这里就把相邻的节点标记了，在这里其实就是已经遍历过了节点，所以要标记。
                    visitedMap[rotated] = struct{}{}
                    queue = append(queue, rotated)
                }
                rotated2 := minusOne(tmp, j)
                if _, ok := visitedMap[rotated2]; !ok {
                    visitedMap[rotated2] = struct{}{}
                    queue = append(queue, rotated2)
                }
            }
        }
        // 对队列里所有情况都试了一遍之后其实才是旋转了 1 次
        steps ++
        queue = queue[n:]
    }
    return -1
}

func plusOne(code string, i int) string {
    codeSlice := []byte(code)
    if codeSlice[i] == '9' {
        codeSlice[i] = '0'
    } else {
        codeSlice[i] += 1
    }
    return string(codeSlice)
}

func minusOne(code string, i int) string {
    codeSlice := []byte(code)
    if codeSlice[i] == '0' {
        codeSlice[i] = '9'
    } else {
        codeSlice[i] -= 1
    }
    return string(codeSlice)
}
```

双向 BFS

```go
func openLock2(deadends []string, target string) int {
    steps := 0

    q1 := make(map[string]struct{})
    q2 := make(map[string]struct{})
    visitedMap := make(map[string]struct{})
    deadendMap := make(map[string]struct{})
    for _, d := range deadends {
        deadendMap[d] = struct{}{}
    }

    q1["0000"] = struct{}{}
    q2[target] = struct{}{}

    for len(q1) > 0 && len(q2) > 0 {
        // 我们每次都选择一个较小的集合进行扩散，那么占用的空间增长速度就会慢一些，效率就会高一些
        if len(q1) > len(q2) {
            q1, q2 = q2, q1
        }
        temp := make(map[string]struct{})
        for k := range q1 {
            if _, ok := deadendMap[k]; ok {
                continue
            }
            // 两个 map 有重合数字就表示找到一条能够到 target 的路
            if _, ok := q2[k]; ok {
                return steps
            }
            visitedMap[k] = struct{}{}

            for i := 0; i < 4; i++ {
                rotated := plusOne(k, i)
                // 在这里判断是否被标记过，如果已经标记过就不加入队列中
                if _, ok := visitedMap[rotated]; !ok {
                    temp[rotated] = struct{}{}
                    // 注意不能像上一种方法一样在这里给 visitedMap 赋值，因为第一次是从 q1 开始 BFS，第二次就是从 q2 BFS，
                    // 如果在这里就赋值了，在第二次遍历的时候就不会把和 q1 相邻的节点加入到队列中（因为已经被标记过了），
                    // 以致于 q1 q2 永远无法找到，相同的值，所以应该放在上面，遍历到哪个节点就标记。
                }
                rotated2 := minusOne(k, i)
                if _, ok := visitedMap[rotated2]; !ok {
                    temp[rotated2] = struct{}{}
                }
            }
        }
        // 交换 q1 q2，下一次就开始遍历 q2
        q1 = q2
        q2 = temp
        steps++
    }
    return -1
}

func plusOne(code string, i int) string {
    codeSlice := []byte(code)
    if codeSlice[i] == '9' {
        codeSlice[i] = '0'
    } else {
        codeSlice[i] += 1
    }
    return string(codeSlice)
}

func minusOne(code string, i int) string {
    codeSlice := []byte(code)
    if codeSlice[i] == '0' {
        codeSlice[i] = '9'
    } else {
        codeSlice[i] -= 1
    }
    return string(codeSlice)
}
```

## 总结

BFS 算法和 DFS 算法相比，空间复杂度高很多，因为每次都要把每一层的数据放到内存中再进行遍历，一层层往下推。但是 BFS 算法找最短路径之类的问题走的步数是最少的，所以还是看情况选择算法。
