# 股票买卖问题

Leetcode 上一共有六道股票买卖的问题，一次性来看看，解题思路及模板可以直接看东哥的文章 [团灭 LeetCode 股票买卖问题](https://labuladong.gitbook.io/algo-en/v/master/dong-tai-gui-hua-xi-lie/qi-ta-jing-dian-wen-ti/tuan-mie-gu-piao-wen-ti) ，写得更详细，我这里只记录一下总结后的状态转移方程和简单的思路。

本文要介绍的题：

[买卖股票的最佳时机（简单）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/solution/)

[买卖股票的最佳时机 II（简单）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

[最佳买卖股票时机含冷冻期（中等）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

[买卖股票的最佳时机含手续费（中等）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

[买卖股票的最佳时机 III（困难）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

[买卖股票的最佳时机 IV（困难）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

## 题目解析

一共有6种变形题。我们一次性把所有状态都考虑进去。

设 `dp[i][0][k]` 为第 i 天手上没有股票且交易 k 次后的利润。

**状态转移方程**

有两种情况，分别是当前持有股票和不持有股票：

`dp[i][0][k] = max(dp[i-1][0][k], dp[i-1][1][k] + prices[i]`

`dp[i][1][k] = max(dp[i-1][1][k], dp[i-1][0][k-1] - prices[i]`

### 买卖股票的最佳时机

> 给定一个数组 prices ，它的第 i 个元素 prices\[i] 表示一支给定股票第 i 天的价格。
>
> 你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。
>
> 返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

解析：

k = 1，只能交易一次，对结果没有影响，可以省去 k，简化为

`dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])`

`dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])`，另外 `dp[i-1][0]` 肯定等于0，因为既然只能交易一次，那么你没有股票时的利润肯定是0（否则就不只是交易一次了），而 `dp[i-1][1]` 不能简化，因为持有股票时的利润肯定为负数。

所以最终 `dp[i][1] = max(dp[i-1][1], -prices[i])`，除了用dp数组求解，还可以压缩状态，用两个变量保存当天不持有股票和持有股票时的利润。

### 买卖股票的最佳时机II

> 给定一个数组 `prices` ，其中 `prices[i]` 是一支给定股票第 `i` 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。
>
> \*\*注意：\*\*你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

解析：

k = 正无穷，可以省去k，简化为：

`dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])`

`dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])`

同样也可以压缩状态，用tmp保存前一天的没有持有股票时的利润，用来计算dp1(在下次购买前必须手上没有股票)。

### 最佳买卖股票时机含冷冻期

> 给定一个整数数组，其中第 i 个元素代表了第 i 天的股票价格 。
>
> 设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:
>
> 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。

解析：

从上一题转换而来，因为中间有一天冷冻期，所以要从 i-2 的状态转移，即

`dp[i][1] = max(dp[i-1][1], dp[i-2][0] - prices[i])`。

压缩状态，可以新增一个变量保存前前一天的利润

### 买卖股票的最佳时机含手续费

> 给定一个整数数组 prices，其中第 i 个元素代表了第 i 天的股票价格 ；整数 fee 代表了交易股票的手续费用。
>
> 你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。
>
> 返回获得利润的最大值。
>
> 注意：这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。

解析：

就是第二题加了一个手续费。

### 买卖股票的最佳时机 III

> 给定一个数组，它的第 i 个元素是一支给定的股票在第 i 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润。你最多可以完成 两笔 交易。
>
> 注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

### 买卖股票的最佳时机 IV

> 给定一个整数数组 prices ，它的第 i 个元素 prices\[i] 是一支给定的股票在第 i 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润。你最多可以完成 k 笔交易。
>
> 注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

解析：

这两道题其实可以用同一套代码，分别 k = 2 和 k 是参数来解决。

**状态转移方程**

`dp[i][0][k] = max(dp[i-1][0][k], dp[i-1][1][k] + prices[i]`

`dp[i][1][k] = max(dp[i-1][1][k], dp[i-1][0][k-1] - prices[i]`

## 代码

### 买卖股票的最佳时机

```go
func maxProfit(prices []int) int {
    n := len(prices)    
    if n == 0 {
        return 0
    }

    dp0, dp1 := 0, -prices[0]

    for i := 1; i < n; i++ {
        dp0 = max(dp0, dp1 + prices[i])        
        dp1 = max(dp1, -prices[i])
    }
    return dp0
}

func max(a, b int) int {
    if a >= b {
        return a
    }
    return b
}
```

### 买卖股票的最佳时机II

```go
func maxProfit(prices []int) int {
  n := len(prices)
    if n == 0 {
        return 0
    }

    dp0, dp1 := 0, -prices[0]

    for i := 1; i < n; i++ {
        //保存前一天没有持有股票时的利润，用来计算dp1(在下次购买前必须手上没有股票)
        tmp := dp0
        dp0 = max(dp0, dp1 + prices[i])
        dp1 = max(dp1, tmp - prices[i])
    }
    return dp0
}

func max(a, b int) int {
    if a >= b {
        return a
    }
    return b
}
```

### 最佳买卖股票时机含冷冻期

```go
func maxProfit(prices []int) int {
  n := len(prices)
    if n == 0 {
        return 0
    }

    dp0, dp1 := 0, -prices[0]
  dpPre := 0
    for i := 1; i < n; i++ {
        //保存前一天没有持有股票时的利润，用来计算dp1(在下次购买前必须手上没有股票)
        tmp := dp0
        dp0 = max(dp0, dp1 + prices[i])
        dp1 = max(dp1, dpPre - prices[i])
    //因为中间有一天冷冻期，所以要从i-2的状态转移，即dp[i][1] = max(dp[i-1][1], dp[i-2][0] - prices[i])
    //保存前前一天的状态
    dpPre = tmp
    }
    return dp0
}

func max(a, b int) int {
    if a >= b {
        return a
    }
    return b
}
```

### 买卖股票的最佳时机含手续费

```go
func maxProfit(prices []int, fee int) int {
  n := len(prices)
    if n == 0 {
        return 0
    }

    dp0, dp1 := 0, -prices[0]

    for i := 1; i < n; i++ {
        //保存前一天没有持有股票时的利润，用来计算dp1(在下次购买前必须手上没有股票)
        tmp := dp0
        dp0 = max(dp0, dp1 + prices[i] - fee)
        dp1 = max(dp1, tmp - prices[i])
    }
    return dp0
}   

func max(a, b int) int {
    if a >= b {
        return a
    }
    return b
}
```

### 买卖股票的最佳时机 IV

```go
func maxProfit(k int, prices []int) int {
    n := len(prices)
    if n == 0 {
        return 0
    }
    dp := make([][][]int, n)
    for i := 0; i < n; i++ {
        dp[i] = make([][]int, 2)
        for j := 0; j < 2; j++ {
            // k 可以为 0 次，所以长度为 k+1
            dp[i][j] = make([]int, k+1)
        }
    }
    for i := 1; i <= k; i++ {
        // base case
        // k = 0的时候都是0，不用显示初始化
        dp[0][0][i] = 0
        dp[0][1][i] = -prices[0]
    }

    //dp[i][0][k] = max(dp[i-1][0][k], dp[i-1][1][k] + prices[i]
    //dp[i][1][k] = max(dp[i-1][1][k], dp[i-1][0][k][k-1] - prices[i]
    for i := 1; i < n; i++ {
        for j := 1; j <= k; j++ {
            dp[i][0][j] = max(dp[i-1][0][j], dp[i-1][1][j] + prices[i])
            dp[i][1][j] = max(dp[i-1][1][j], dp[i-1][0][j-1] - prices[i])
        }
    }
    maxRes := 0
    for i := 0; i <= k; i++ {
        maxRes = max(maxRes, dp[n-1][0][i])
    }
    return maxRes
}
func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

## 总结

这类题其实用一个状态转移方程就可以解决，每道题有些细微的变化，可以多看多做几遍就能理解了。
