# 打家劫舍问题

## 打家劫舍问题

Leetcode 上一共有三道打家劫舍的问题，一次性来看看，解题思路可以看看东哥的文章 [经典动态规划：打家劫舍系列问题](https://mp.weixin.qq.com/s?\_\_biz=MzAxODQxMDM0Mw==\&mid=2247484800\&idx=1\&sn=1016975b9e8df0b8f6df996a5fded0af\&scene=21#wechat\_redirect) 。我这里也提供了简单的思路。

本文要介绍的题：

[打家劫舍（简单）](https://leetcode-cn.com/problems/house-robber)

[打家劫舍II（中等）](https://leetcode-cn.com/problems/house-robber-ii)

[打家劫舍III（中等）](https://leetcode-cn.com/problems/house-robber-iii)

## 题目解析

### 打家劫舍

> 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

解析：

**状态**

`dp[i]` 为一共有 i 家时能偷盗的最大总金额

**转移方程**

`dp[i] = max(dp[i-1], dp[i-2] + nums[i])`，即第 i 家的金额等于偷上一家的总金额和上上一家的金额加上第 i 家的金额。还可以用 2个变量存储 `dp[i-1]` 和 `dp[i-2]` ，进行状态压缩。

**初始化**

`dp[0] = nums[0]`

`dp[1] = max(nums[0], nums[1])`

**返回**

`dp[n-1]`

### 打家劫舍II

> 你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，今晚能够偷窃到的最高金额。

解析：

这题其实就是 2 种情况，偷第一家和不偷第一家，取最大值，可以直接拿上一题的代码作为 helper 函数，即

`max(robHelper(nums[:n-1]), robHelper(nums[1:]))`

### 打家劫舍III

> 在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。
>
> 计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。

解析：

这题涉及到树，那么肯定要用递归。要注意题目并没有说到了某一层必须得把左右子树的全偷了，可以只偷左子树或者右子树，所以左右子树要分开计算。

**转移方程**

对于root节点来说，就是两种情况：

选择root节点，那么就不能选择 root.Left 和 root.Right。

要么不选择 root 节点的值，可以选择 root.Left 或者不选择 root.Left，可以选择 root.Right 或者不选择 root.Right （小偷可以 root、root.Left，root.Right 都不偷）。这样可以得出状态转移方程：

`f[node] = node->val + g[node->left] + g[node->right];` `​g[node] = max(f[node->left], g[node->left]) + max(f[node->right], g[node->right]);`

f 表示选择，g 表示不选择。

最后我们需要定义一个 dfs 函数，深度遍历 left，right，返回值是一个int数组，\[]int{selectd, nonSelectd}，即选择root节点和不选择root 节点的两个值。

## 代码

### 打家劫舍

dp 数组：

```go
//dp[i] = max(dp[i-1], dp[i-2] + nums[i])
func rob(nums []int) int {
    n := len(nums)
    if n == 1 {
        return nums[0]
    }
    if n == 2 {
        return max(nums[0], nums[1])
    }
    // dp := make([]int, n)
    dp := make([]int, n)
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    for i := 2; i < n; i++ {
        dp[i] = max(dp[i-1], dp[i-2] + nums[i])
    }
    return dp[n-1]
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

状态压缩：

```go
//dp[i] = max(dp[i-1], dp[i-2] + nums[i])
func rob(nums []int) int {
    n := len(nums)
    if n == 1 {
        return nums[0]
    }
    if n == 2 {
        return max(nums[0], nums[1])
    }
    // dp := make([]int, n)
    dp0 := nums[0]
    dp1 := max(nums[0], nums[1])
    dp2 := dp1
    for i := 2; i < n; i++ {
        dp2 = max(dp1, dp0 + nums[i])
        dp0 = dp1
        dp1 = dp2
    }
    return dp2
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 打家劫舍II

```go
//dp[i][0] = max(dp[i-1], dp[i-2] + nums[i])
func rob(nums []int) int {
    n := len(nums)
    if n == 1 {
        return nums[0]
    }
    if n == 2 {
        return max(nums[0], nums[1])
    }
    //要么偷第一家，那么就是计算num[0:n-1]，要么不偷第一家，即nums[1:]
    return max(robHelper(nums[:n-1]), robHelper(nums[1:]))
}

func robHelper(nums []int) int {
    n := len(nums)
    dp := make([]int, n)
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    for i := 2; i < n; i++ {
        dp[i] = max(dp[i-1], dp[i-2] + nums[i])
    }
    return dp[n-1]
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 打家劫舍III

```go
func rob(root *TreeNode) int {
    v := dfs(root)
    //返回是否选择root节点的最大值
    return max(v[0], v[1])
}

/* 返回一个大小为 2 的数组 arr
arr[0] 表示不抢 root 的话，得到的最大钱数
arr[1] 表示抢 root 的话，得到的最大钱数 */
func dfs(root *TreeNode) []int {
    if root == nil {
        return []int{0, 0}
    }
    l, r := dfs(root.Left), dfs(root.Right)
     // 抢，下家就不能抢了
    selected := root.Val + l[0] + r[0]
    //如果选择不偷root这个节点，那么就要选择 "选择左节点" 和 "不选择左节点"的最大值，因为一层不一定需要左右节点一起偷，所以左右节点要分开判断最大值
    nonSelected := max(l[0], l[1]) + max(r[0], r[1])
    return []int{nonSelected, selected}
}


func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

## 总结

前两道题是比较简单的动态规划问题，第三道题稍微有点不一样，结合了树的遍历，这里选择用深度遍历来做。
