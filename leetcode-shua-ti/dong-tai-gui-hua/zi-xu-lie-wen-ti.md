# 子序列问题

这次来看看子序列问题，子序列问题也是比较常见的动态规划问题。

本文要介绍的题：

[最大子序和（简单）](https://leetcode-cn.com/problems/maximum-subarray/)

[最长公共子序列（中等）](https://leetcode-cn.com/problems/longest-common-subsequence/)

[两个字符串的删除操作（中等）](https://leetcode-cn.com/problems/delete-operation-for-two-strings/)

[两个字符串的最小ASCII删除和（中等）](https://leetcode-cn.com/problems/minimum-ascii-delete-sum-for-two-strings/)

[最长回文子序列（中等）](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

[最长回文子串（中等）](https://leetcode-cn.com/problems/longest-palindromic-substring/)

[石子游戏（中等）](https://leetcode-cn.com/problems/stone-game/)

## 题目解析

### 最大子序和

> 给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

解析：

这题看上去和滑动窗口有点关系，但是并不好用滑动窗口算法来解，因为数组中可能包含负数。

用动态规划就简单了。

**状态**

设 `dp[i]` 为前 i 个数的最大子序和。

**转移方程**

`dp[i] = max(nums[i], dp[i-1] + nums[i])`

因为 `dp[i-1]` 可能是负数，所以可以直接舍弃前面所有的子序和，那就从 nums\[i] 从头开始计算子序和。

**返回**

取 dp 数组中最大值返回即可。

### 最长公共子序列

> 给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。
>
> 一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
>
> 例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。 两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。

解析：

**状态**

二维数组dp，`dp[i][j]`的意思是当 text1 的长度为 i 和当 text2 的长度为 j 时，最长公共子序列的长度。设 m = len(text1)，n = len(text2)，因为 dp 是包含 text1 text2 长度为 0 的情况，所以 dp 的长度应该是 m + 1 和 n + 1。

**转移方程**

i 从 1 遍历到 m，j 从 1 遍历到n，有两种情况：

当 text1\[i-1] = text2\[j-1] 时， 则 `dp[i][j] = 1 + dp[i-1][j-1]`。

否则 `dp[i][j] = max(dp[i][j-1], dp[i-1][j])` ，即分别保留 text1 的当前值或者 text2 的值，取这两种情况的公共子序列最长。

**初始化**

base case 是 `dp[0][j] = 0，dp[i][0] = 0`。

**返回**

`dp[m][n]`

### 两个字符串的删除操作

> 给定两个单词 _word1_ 和 _word2_，找到使得 _word1_ 和 _word2_ 相同所需的最小步数，每步可以删除任意一个字符串中的一个字符。
>
> ```
> 输入: "sea", "eat"
> 输出: 2
> 解释: 第一步将"sea"变为"ea"，第二步将"eat"变为"ea"
> ```

解析：

其实这道题就可以直接使用上一题的代码，先找出最长公共子序列，然后再用两个单词的长度减去它即可。或者可以用动态规划算法。具体可以看下道题的解释，后面也有代码解释。

### 两个字符串的最小ASCII删除和

> 给定两个字符串`s1, s2`，找到使两个字符串相等所需删除字符的ASCII值的最小和。
>
> ```
> 输入: s1 = "sea", s2 = "eat"
> 输出: 231
> 解释: 在 "sea" 中删除 "s" 并将 "s" 的值(115)加入总和。
> 在 "eat" 中删除 "t" 并将 116 加入总和。
> 结束时，两个字符串相等，115 + 116 = 231 就是符合条件的最小和。
> ```

解析：

这道题也借用最长公共子序列来解，先算出两个字符串的公共子序列的字符 ASCII 和 lcs\_sum，再将两个字符串的 ASCII 和减去 2 \* lcs\_sum 即可。

或者这道题也可以不借助 lsc，而用传统的动态规划来求解。

**状态**

`dp[i][j]` 数组表示从 s1 第 i 位 和 s2 第 j 位的最小删除和。

**转移方程**

和之前类似，也是两种情况：

当 s1\[i-1] = s2\[j-1] 时，则 `dp[i][j] = dp[i-1][j-1]`，相同则不用删除。

否则 `dp[i][j] = min(dp[i-1][j] + int(s1[i-1]), dp[i][j-1] + int(s2[j-1]))`，不相同则分别删除 s1 或 s2 的当前值，再加上删除的值。

**初始化**

Base case 是当 s2 为空字符串时，`dp[i][0]`就等于`dp[i-1][0]`加上s1的当前值，意思是 s2 为空，那么肯定是要删除 s1 的所有字符，那么对应到代码就是 `dp[i][0] = dp[i-1][0] + int(s1[i-1])`， s1 为空也是如此。

**返回**

`dp[m][n]`。

### 最长回文子序列

> 给你一个字符串 s ，找出其中最长的回文子序列，并返回该序列的长度。
>
> 子序列定义为：不改变剩余字符顺序的情况下，删除某些字符或者不删除任何字符形成的一个序列。
>
> ```
> 输入：s = "bbbab"
> 输出：4
> 解释：一个可能的最长回文子序列为 "bbbb" 。
> ```

解析：

**状态**

`dp[i][j]` 表示 s 的第 i 个字符到第 j 个字符组成的子串中，最长的回文序列长度是多少。

**转移方程** 如果 s 的第 i 个字符和第 j 个字符相同的话

`dp[i][j] = dp[i + 1][j - 1] + 2`

如果 s 的第 i 个字符和第 j 个字符不同的话

`dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])`

我们初始化了对角线，而 i, j 又是依赖于 i+1 j-1，即矩阵的下方和左方，这样我们可以斜着遍历或者往矩阵的上方和右方遍历。

![](<../../.gitbook/assets/lcs (1).png>)

这里选择第二种遍历顺序，i 从最后一个字符开始往前遍历，j 从 i + 1 开始往后遍历，这样可以保证每个子问题都已经算好了。

**初始化** `dp[i][i] = 1` 单个字符的最长回文序列是 1

**返回** `dp[0][n - 1]`

### 最长回文子串

> 给你一个字符串 `s`，找到 `s` 中最长的回文子串。
>
> \`\`\` 输入：s = "babad" 输出："bab" 解释："aba" 同样是符合题意的答案。

解析：

这道题有多种解法，这里选取其中两种来看。

**第一种解法：动态规划。**

**状态**

`dp[i][j]`表示`s[i:j]`是否是回文串

**转移方程**

如果 `s[i] == s[j]`，那么`dp[i][j] = dp[i+1][j-1]，`即两边相等再看中间的串，如果 i, j 之间的长度 <= 2，则 `dp[i][j] = true`，

因为可能是 aba 或 aa 的情况。

如果 `s[i] != s[j]`，那么`dp[i][j] = false`

遍历顺序还是和上一题一样。

**初始化** `dp[i][i] = 1` 单个字符的最长回文子串为 true

**返回** 记录每次回文子串的开始索引 start 和回文子串长度 length，返回 `s[start:start+maxLen+1]`

**第二种解法：中心扩散算法。**

我们知道回文串一定是对称的，所以我们可以每次循环选择一个中心，进行左右扩展，判断左右字符是否相等即可。

由于存在奇数的字符串和偶数的字符串，所以我们需要从一个字符开始扩展，或者从两个字符之间开始扩展，所以总共有 n+n-1 个中心。

可以看下代码更容易理解，当然这题最重要还是了解动态规划的解法，毕竟最通用。

### 石子游戏

> 亚历克斯和李用几堆石子在做游戏。偶数堆石子**排成一行**，每堆都有正整数颗石子 `piles[i]` 。
>
> 游戏以谁手中的石子最多来决出胜负。石子的总数是奇数，所以没有平局。
>
> 亚历克斯和李轮流进行，亚历克斯先开始。 每回合，玩家从行的开始或结束处取走整堆石头。 这种情况一直持续到没有更多的石子堆为止，此时手中石子最多的玩家获胜。
>
> 假设亚历克斯和李都发挥出最佳水平，当亚历克斯赢得比赛时返回 `true` ，当李赢得比赛时返回 `false` 。

解析：

**状态**

`dp[i][j]`表示从第 i 到 j 堆石头，Alex（先手）能领先Lee（后手）的最大分值。

**转移方程**

两种情况：

Alex拿走第 i 堆，则相当于变成：Alex初始分数为 piles\[i]，第 i+1 到 j 堆，且 Lee 先手。此时计算的情况相反，所以 Alex 能领先 Lee 的最大分值 `dp[i+1][j]`要 加上负号。下同。 `dp[i][j] = piles[i] + (-dp[i+1][j])`

Alex拿走第 j 堆，则相当于变成：Alex 初始分数为 piles\[j] ，第 i 到 j-1 堆，且 Lee 先手。 `dp[i][j] = piles[j] + (-dp[i][j-1])`

二者取大。

即 `dp[i][j] = max(piles[i] - dp[i+1][j], piles[j] - dp[i][j-1])`

**初始化**

如果只有一堆，那么领先的最大分值就是 piles\[i]

`dp[i][i] = piles[i]`

**返回**

`dp[0][n-1] > 0`

当然直接return true也可以。先手的必赢。

## 代码

### 最大子序和

普通版本：

```go
func maxSubArray(nums []int) int {
    maxRes := math.MinInt64
    dp := make([]int, len(nums))
    for i := range nums {
        if i > 0 {
            dp[i] = max(nums[i], dp[i-1] + nums[i])
        } else {
            dp[i] = nums[i]
        }
        maxRes = max(maxRes, dp[i])
    }
    return maxRes
}

func max (a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

状态压缩：

```go
//状态压缩
func maxSubArray(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    pre, cur, maxRes := nums[0], nums[0], nums[0]
    for i := 1; i < len(nums); i++ {
        cur = max(nums[i], pre + nums[i])
        maxRes = max(maxRes, cur)
        pre = cur
    }
    return maxRes
}

func max (a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 最长公共子序列

```go
func longestCommonSubsequence(text1 string, text2 string) int {
    //返回的结果为dp[m][n]
    m, n := len(text1), len(text2)
    //初始化dp，base case为当text1长度为0或text2长度为0时，dp[0][j] = 0，dp[i][0] = 0，初始化就是0，所以不用特意再赋值
    dp := make([][]int, m+1)
    for i := 0; i < len(dp); i++ {
        dp[i] = make([]int, n+1)
    }
    //有2种情况，
    //1: text1[i] = text2[j] 则 dp[i][j] = 1 + dp[i-1][j-1]
    //2: text1[i] != text2[j] 则 dp[i][j] = max(dp[i][j-1], dp[i-1][j]) 分别保留text1的当前值或者text2的值，取最大
    //O(n^2)，必须遍历text1和text2
    for i := 1; i < m+1; i++ {
        for j := 1; j < n+1; j++ {
            if text1[i-1] == text2[j-1] {
                dp[i][j] = 1 + dp[i-1][j-1]
            } else {
                dp[i][j] = max(dp[i][j-1], dp[i-1][j])
            }
        }
    }
    return dp[m][n]
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 两个字符串的删除操作

lcs 解法：

```go
func minDistance(word1 string, word2 string) int {
    l := longestCommonSubsequence(word1, word2)
    return len(word1) - l + len(word2) - l
 }

func longestCommonSubsequence(text1 string, text2 string) int {
    m, n := len(text1), len(text2)
    dp := make([][]int, m+1)
    for i := 0; i < m+1; i++ {
        dp[i] = make([]int, n+1)
        for j := 0; j < n+1; j++ {
            if i == 0 || j == 0 {
                dp[i][j] = 0
            }
        }
    }

    for i := 1; i < m+1; i++ {
        for j := 1; j < n+1; j++ {
           if text1[i-1] == text2[j-1] {
               dp[i][j] = 1 + dp[i-1][j-1]
           } else {
               dp[i][j] = max(dp[i][j-1], dp[i-1][j])
           }
        }
    }
    return dp[m][n]
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

动态规划解法：

```go
func minDistance(word1 string, word2 string) int {
	m, n := len(word1), len(word2)

	dp := make([][]int, m+1)
	for i := 0; i < m+1; i++ {
		dp[i] = make([]int, n+1)
		dp[i][0] = i
	}

	for j := range dp[0] {
		dp[0][j] = j
	}

	// dp[i][j]: word1 在索引 i 和 word2 在索引 j 时需要删除的字符个数
	// base case: 当 word1 是空或 word2 是空时，值就是另一个字符串的长度，dp[0][j] = j, dp[i][0] = i
	// func:
	// 1. if word1[i] == word2[j] 则不需要删除。dp[i][j] = dp[i-1][j-1]
	// 2. else 选择删除 word1 和 word2 中的某一个的当前字符，选最小值并+1。 dp[i][j] = min(dp[i-1][j], dp[j-1][i]) + 1
	for i := 1; i < m+1; i++ {
		for j := 1; j < n+1; j++ {
			if word1[i-1] == word2[j-1] {
				dp[i][j] = dp[i-1][j-1]
			} else {
				dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + 1
			}
		}
	}
	return dp[m][n]
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
```

### 两个字符串的最小ASCII删除和

```go
func minimumDeleteSum(s1 string, s2 string) int {
    m, n := len(s1), len(s2)
    //dp数组表示从s1、s2第“几”位的最小删除和，要返回的是dp[m][n]，为什么要从1开始呢？
    //因为要用0表示s1、s2为空的情况。
    dp := make([][]int, m+1)
    for i := 0; i < m+1; i++ {
        dp[i] = make([]int, n+1)
    }
    for i := 1; i < m+1; i++ {
        //当s2为空字符串时，dp[i][0]就等于dp[i-1][0]加上s1的当前值
        dp[i][0] = dp[i-1][0] + int(s1[i-1])
    }
    for j := 1; j < n+1; j++ {
        dp[0][j] = dp[0][j-1] + int(s2[j-1])
    }
    var i, j int
    for i = 1; i < m+1; i++ {
        for j = 1; j < n+1; j++ {
            if s1[i-1] != s2[j-1] {
                dp[i][j] = min(dp[i-1][j] + int(s1[i-1]), dp[i][j-1] + int(s2[j-1]))
            } else {
                dp[i][j] = dp[i-1][j-1]
            }
        }
    }
    return dp[m][n]
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### 最长回文子序列

```go
func longestPalindromeSubseq(s string) int {
    n := len(s)
    dp := make([][]int, n)
    for i := 0; i < n; i++ {
        dp[i] = make([]int, n)
        dp[i][i] = 1
    }
    for i := n - 1; i >= 0; i-- {
        for j := i + 1; j < n; j++ {
            if s[i] == s[j] {
                dp[i][j] = dp[i+1][j-1] + 2
            } else {
                dp[i][j] = max(dp[i][j-1], dp[i+1][j])
            }
        }
    }
    return dp[0][n-1]
}
func max(a, b int) int {
    if a < b {
        return b
    }
    return a
}
```

还有一种 i，j 颠倒的写法，本质上是一样的：

```go
/*
dp[i][j]: s 在 [i,j] 的范围内的最长回文子序列
base: dp[i][i] = 1, if i > j dp[i][j] = -1
func:
1. if s[i] == s[j] dp[i][j] = dp[i+1][j-1]+2
2. if s[i] != s[j] dp[i][j] = max(dp[i+1][j], dp[i][j-1])
由于 i，j 依赖于 i+1 和 j-1，而且最终目标是算出 dp[0][n-1]，
所以应该是从左上角开始，根据对角线从右往左遍历，再从上往下，最终得出 dp[0][n-1]
*/
func longestPalindromeSubseq(s string) int {
    n := len(s)

    dp := make([][]int, n)
    for i := range dp {
        dp[i] = make([]int, n)
        dp[i][i] = 1
    }

    // 应该先从右往左遍历，然后从上往下，所以 j 是外循环，i 是内循环
    // j = 0 时都是 0，所以不用遍历
    for j := 1; j < n; j++ {
        // 如果 i >= j 则不处理
        for i := j-1; i >= 0; i-- {
            if s[i] == s[j] {
                dp[i][j] = dp[i+1][j-1] + 2
            } else {
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])
            }
        }
    }
    return dp[0][n-1]
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 最长回文子串

动态规划：

```go
func longestPalindrome(s string) string {
    n := len(s)
    if n <= 1 {
        return s
    }
    // dp[i][j]表示s[i:j]是否是回文串
    dp := make([][]bool, n)
    for i := 0; i < n; i++ {
        dp[i] = make([]bool, n)
        //s[i][i]只有一个字符时都是回文，这是base case
        dp[i][i] = true
    }
    // 回文串最大长度
    maxLen := 0
    // 字串开始索引
    start := 0
    /*
    状态转移方程有2种情况：
    1. 如果s[i] == s[j]，那么dp[i][j] = dp[i+1][j-1]，即两边相等再看中间的串
    2. 如果s[2] != s[j]，那么dp[i][j] = false
    我们初始化了对角线，而 i,j 又是依赖于 i+1 j-1，即矩阵的下方和左方，这样我们可以往矩阵的上方和右方遍历，
    */
    // i 从下往上遍历，对角线 dp[n-1][n-1] 已经赋值过了，所以从 n-2 开始遍历
    for i := n-2; i >= 0; i-- {
        // j 从左往右遍历，从 i+1 开始到 n
        for j := i+1; j < n; j++ {
            length := j - i
            if s[i] == s[j] {
                //如果是aba、或aa这种情况
                if length <= 2 {
                    dp[i][j] = true
                } else {
                    dp[i][j] = dp[i+1][j-1]
                }
            } else {
                dp[i][j] = false
            }
            //更新maxLen和start
            if dp[i][j] && length > maxLen {
                maxLen = length
                start = i
            }
        }
    }
    return s[start:start+maxLen+1]
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

中心扩散算法：

```go
func longestPalindrome(s string) string {
	var maxLength int
	var res string
	// 中心扩散算法
	for i := 0; i < len(s); i++ {
		l1, r1 := helper(i, i, s)
		l2, r2 := helper(i, i+1, s)
		len1, len2 := r1 - l1 + 1, r2 - l2 + 1
		if len1 > len2 {
			if maxLength < len1 {
				maxLength = len1
				res = s[l1:r1+1]
			}
		} else if maxLength < len2 {
			maxLength = len2
			res = s[l2:r2+1]
		}
	}
	return res
}

func helper(l, r int, s string) (int, int) {
	for l >= 0 && r < len(s) && s[l] == s[r] {
		l --
		r ++
	}
	return l+1, r-1
}
```

### 石子游戏

```go
func stoneGame(piles []int) bool {
	/*
	dp[i][j] 表示在 piles[i][j] 里 alice 领先 bob 多少分
	base: dp[i][i] = i
	func: dp[i][j] = max(piles[i] - dp[i+1][j], piles[j] - dp[i][j-1])
	 */
	n := len(piles)
	dp := make([][]int, n)
	for i := range dp {
		dp[i] = make([]int, n)
		dp[i][i] = piles[i]
	}
	// 向右和向上遍历
	for i := n-1; i >= 0; i-- {
		for j := i+1; j < n; j++ {
			dp[i][j] = max(piles[i] - dp[i+1][j], piles[j] - dp[i][j-1])
		}
	}
	return dp[0][n-1] > 0
}


func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```

压缩空间：

```go
func stoneGame(piles []int) bool {
    length := len(piles)
    dp := make([]int, length)
    for i := 0; i < length; i++ {
        dp[i] = piles[i]
    }
    for i := length - 2; i >= 0; i-- {
        for j := i + 1; j < length; j++ {
            dp[j] = max(piles[i] - dp[j], piles[j] - dp[j - 1])
        }
    }
    return dp[length - 1] > 0
}

func max(x, y int) int {
    if x > y {
        return x
    }
    return y
}
```

## 总结

子序列的几题其实套路都很像，就是几种情况分别枚举，两者相等或者两者不相等，要么保留这个要么保留那个，取最大或最小。石子游戏其实是个很经典的博弈类的游戏，但是不专门开一篇写，就也放这了。
