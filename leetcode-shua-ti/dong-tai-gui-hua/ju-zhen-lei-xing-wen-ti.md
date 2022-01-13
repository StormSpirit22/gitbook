# 矩阵类型问题

这次来看看矩阵类型问题，矩阵问题也是比较常见的动态规划问题。

这类问题基本都是一个二维数组来解决，dp 的意思其实不需要像之后的一些题目去仔细定义，i、j 的意思比较明确，状态转移方程也比较容易找出来。

本文要介绍的题：

[三角形最小路径和（中等）](https://leetcode-cn.com/problems/triangle/)

[最小路径和（中等）](https://leetcode-cn.com/problems/minimum-path-sum/)

[不同路径（中等）](https://leetcode-cn.com/problems/unique-paths/)

[不同路径 II（中等）](https://leetcode-cn.com/problems/unique-paths-ii/)

## 题目解析

### 三角形最小路径和

> 给定一个三角形 triangle ，找出自顶向下的最小路径和。
>
> 每一步只能移动到下一行中相邻的结点上。相邻的结点 在这里指的是 下标 与 上一层结点下标 相同或者等于 上一层结点下标 + 1 的两个结点。也就是说，如果正位于当前行的下标 i ，那么下一步可以移动到下一行的下标 i 或 i + 1 。

解析：

首先先明确 dp 数组的含义，三角形每个数字的坐标都有行列，那么设 `dp[i][j]`为在第 i 行第 j 列的数字到**最底下一层**的最小路径和。

我们先想**自底向上**的方法，那么很明显最底下一层就是 base case，对于最底下一层来说，它的最小路径和就是它自己。

明确了 base case 我们接着往“上”推，对于倒数第二层来说，它的最小路径和是不是它下面两个“腿”的最小值加上它自己？

这样就能推出动态转移方程：`dp[i][j] = min(dp[i+1][j], dp[i+1][j+1]) + triangle[i][j]`，即对于 `dp[i][j]`来说，它的下一层的可能性就两种：`dp[i+1][j]` 和 `dp[i+1][j+1]`，找出最小值。

因为我们是要找第一个节点自顶向下的最小路径和， 所以最后返回 `dp[0][0]` 即可。这就是**自底向上**的解法。

第二种解法是**自顶向下**的解法，即计算出每个元素到**顶部**的最小路径和，这个时候返回的就是最后一层数据中的最小值。

这个状态转移方程是 `dp[i][j] = min(dp[i-1][j], dp[i-1][j-1]) + triangle[i][j]`，但是这个解法对于这题需要判断条件，因为有的元素的上层可能只有一个元素，所以相比第一种解法要麻烦一些。

### 最小路径和

> 给定一个包含非负整数的 m x n 网格 grid ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
>
> 说明：每次只能向下或者向右移动一步。

解析：

和上一题类似，设 `dp[i][j]` 为第 i 行第 j 列的数字到右下角的最小路径和。

还是用**自底向上**的方法，base case 就是右下角 `dp[m-1][n-1] = grid[m-1][n-1]`。

因为每次只能向下或者向右移动一步，所以对于一个不在边界上的格子来说，`dp[i][j] = min(dp[i+1][j], dp[i][j+1]) + grid[i][j]`，还有两种情况分别是在最后一行或最后一列的情况，那就只能往右移动或者往下移动了，需要单独判断。

### 不同路径

> 一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。
>
> 机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。
>
> 问总共有多少条不同的路径？

解析：这题其实是上一题的简化版，要注意的是如果在最后一行最后一列的情况下，其实只有一种走法，即 `dp[m-1][x] = 1 ， dp[x][n-1] = 1`。

### 不同路径II

> 一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
>
> 机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。
>
> 现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？

解析：

这题比上题多加了个障碍物的选项。其实也是和上一题类似，状态转移方程也是一样的。只是需要多加判断条件，障碍物那个格子的 dp值肯定是 0 ，没有一种走法能从障碍物走到右下角。然后在最后一行最后一列的情况下，不是简单的将 dp 值赋值为 1，因为可能有障碍物，应该是将右边的和下面的格子的 dp 值赋值给它，即 `dp[m-1][x] = dp[m-1][x+1]，dp[x][n-1] = dp[x+1][n-1]`。另外这道题是可以用状态压缩的，只用一维数组就可以解决。可以看一下代码。

## 代码

### 三角形最小路径和

自底向上：

```go
func minimumTotal(triangle [][]int) int {
    dp := make([][]int, len(triangle))
    for i := range dp {
        dp[i] = make([]int, len(triangle[i]))
        for j := 0; j < len(dp[i]); j++ {
            dp[i][j] = triangle[i][j]
        }
    }
    //自底向上，倒序
    for i := len(dp)-2; i >= 0; i-- {
        for j := 0; j < len(dp[i]); j++ {
            dp[i][j] = min(dp[i+1][j], dp[i+1][j+1]) + triangle[i][j]
        }
    }
    return dp[0][0]
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

自顶向下：

```go
func minimumTotal(triangle [][]int) int {
    dp := make([][]int, len(triangle))
    for i := range dp {
        dp[i] = make([]int, len(triangle[i]))
        for j := 0; j < len(dp[i]); j++ {
            dp[i][j] = triangle[i][j]
        }
    }
    // 自顶向下
  // i = 1， j = 0 从第二行第一个开始遍历
    for i := 1; i < len(dp); i++ {
        for j := 0; j < len(dp[i]); j++ {
            // j = 0 的时候上层没有左元素
            // j = len(dp[i])-1 的时候上层没有右元素
            if j == 0 {
                dp[i][j] = dp[i-1][j] + triangle[i][j]
            } else if j == len(dp[i])-1 {
                dp[i][j] = dp[i-1][j-1] + triangle[i][j]
            } else {
                dp[i][j] = min(dp[i-1][j], dp[i-1][j-1]) + triangle[i][j]
            }
        }
    }
    n := len(dp)-1
    minRes := math.MaxInt16
    for i := 0; i < len(dp[n]); i++ {
        if minRes > dp[n][i] {
            minRes = dp[n][i]
        }
    }
    return minRes
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### 最小路径和

```go
func minPathSum(grid [][]int) int {
    dp := make([][]int, len(grid))
    for i := 0; i < len(grid); i++ {
        dp[i] = make([]int, len(grid[i]))
        for j := 0; j < len(dp[i]); j++ {
            dp[i][j] = grid[i][j]
        }
    }

    for i := len(dp) - 1; i >= 0; i-- {
        for j := len(dp[i]) - 1; j >= 0; j-- {
            if i == len(dp) - 1 && j == len(dp[i]) - 1 {
                continue
            }
            if i == len(dp) - 1 {
                dp[i][j] = dp[i][j+1] + grid[i][j]
            } else if j == len(dp[i]) - 1 {
                dp[i][j] = dp[i+1][j] + grid[i][j]
            } else {
                dp[i][j] = min(dp[i+1][j], dp[i][j+1]) + grid[i][j]
            }
        }
    }
    return dp[0][0]
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### 不同路径

```go
func uniquePaths(m int, n int) int {
    dp := make([][]int, m)
    for i := 0; i < m; i++ {
        dp[i] = make([]int, n)
    }

    for i := m-1; i >= 0; i-- {
        for j := n-1; j >= 0; j-- {
            //在最右和最底一层只有一种走法
            if i == m-1 || j == n-1 {
                dp[i][j] = 1
            } else {
                //否则就是向下或者向右两种走法
                dp[i][j] = dp[i+1][j] + dp[i][j+1]
            }
        }
    }
    return dp[0][0]
}
```

### 不同路径II

二维数组解法：

```go
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    if len(obstacleGrid) == 0 || obstacleGrid[0][0] == 1 {
        return 0
    }
    m := len(obstacleGrid)
    n := len(obstacleGrid[0])

    dp := make([][]int, m)
    for i := 0; i < m; i++ {
        dp[i] = make([]int, n)
        for j := 0; j < n; j++ {
            dp[i][j] = 1
        }
    }

    for i := m-1; i >= 0; i-- {
        for j := n-1; j >= 0; j-- {
            if obstacleGrid[i][j] == 1 {
                dp[i][j] = 0
                continue
            }
            // 右下角的 base case， 已经初始化过了
            if i == m-1 && j == n-1 {
                continue
            }
            //在最底和最右一层只有一种走法，就是等于相邻的那个格子的路径
            if i == m-1 {
                dp[i][j] = dp[i][j+1]                
            } else if j == n-1 {
                dp[i][j] = dp[i+1][j]
            } else {
                //否则就是向下或者向右两种走法
                dp[i][j] = dp[i+1][j] + dp[i][j+1]
            }
        }
    }
    return dp[0][0]
}
```

状态压缩解法（参考官方题解）：

```go
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    n, m := len(obstacleGrid), len(obstacleGrid[0])
    f := make([]int, m)
    if obstacleGrid[0][0] == 0 {
        f[0] = 1
    }
    for i := 0; i < n; i++ {
        for j := 0; j < m; j++ {
            if obstacleGrid[i][j] == 1 {
                f[j] = 0
                continue
            }
            if j - 1 >= 0 && obstacleGrid[i][j-1] == 0 {
                f[j] += f[j-1]
            }
        }
    }
    return f[len(f)-1]
}
```

## 总结

做完这几道题其实就可以发现，这一类型的题目都是一样的套路，都存在一定的递推性质，当前问题的方案数取决于子问题的方案数。所以在遇到求方案数的问题时，我们可以往动态规划的方向考虑。
