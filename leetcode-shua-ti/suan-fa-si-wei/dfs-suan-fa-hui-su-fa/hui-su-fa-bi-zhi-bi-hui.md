# 回溯法进阶

上一篇文章介绍了回溯法的基础，如果把上一篇文章的框架和题解完全弄懂了的话，这篇的题目应该不是特别难。这次我们来看回溯法的一些练习题，熟能生巧。

本文要介绍的题：

[全排列 II（中等）](https://leetcode-cn.com/problems/permutations-ii/)

[子集（中等）](https://leetcode-cn.com/problems/subsets/)

[子集 II（中等）](https://leetcode-cn.com/problems/subsets-ii/)

[组合（中等）](https://leetcode-cn.com/problems/combinations/)

[括号生成（中等）](https://leetcode-cn.com/problems/generate-parentheses/)

## 题解

### 全排列II

> 给定一个可包含重复数字的序列 `nums` ，**按任意顺序** 返回所有不重复的全排列。
>
> ```
> 输入：nums = [1,1,2]
> 输出：
> [[1,1,2],
>  [1,2,1],
>  [2,1,1]]
> ```

解析：

这题和全排列的套路差不多。但是因为该序列是包含重复数字的，所以要先排序再多一个判断条件：

`if i > 0 && nums[i] == nums[i-1] && !visited[i-1]` 就 continue。

加上 !visited\[i - 1] 来去重主要是通过限制一下两个相邻的重复数字的访问顺序。

举个栗子，对于两个相同的数11，我们将其命名为 1a1b, 1a表示第一个1，1b表示第二个1；

那么，不做去重的话，会有两种重复排列 1a1b, 1b1a， 我们只需要取其中任意一种排列；

为了达到这个目的，限制一下1a, 1b访问顺序即可。 比如我们只取1a1b那个排列的话，

只有当 visit nums\[i-1] 之后我们才去 visit nums\[i] ， 也就是如果 !visited\[i-1] 的话则continue。

或者 visited\[i-1] 也可以，即去掉 ! ，此时意思就是访问过了 nums\[i-1] 就 continue。

因为要去重所以肯定要先排个序啦。

### 子集

> 给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。
>
> 解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

解析：

回溯法有几种套路，一种是上面的用 **visited 标记数组来判断是否遍历过**，另一种是通过 **start 标志位来判断从哪个位置开始遍历**，

这题是子集，所以很显然应该有个标志位，不能再把之前的数字重复遍历，每次都从标志位开始遍历就能解决。

### 子集II

> 给你一个整数数组 nums ，其中可能包含重复元素，请你返回该数组所有可能的子集（幂集）。
>
> 解集 不能 包含重复的子集。返回的解集中，子集可以按 任意顺序 排列。

解析：

这题和 全排列II 类似，也是在上一题的解法上先排序加一个判断条件。

`if i > start && nums[i] == nums[i-1]` 则 continue。

当 i > start 的时候（因为 i 从 start 开始遍历）才需要比较当前元素和之前元素是否相等，相等则跳过，

### 组合

> 给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。
>
> 你可以按 **任何顺序** 返回答案。

解析：

这题和子集一样，也是一个 start 标志位即可。

### 括号生成

> 数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。
>
> ```
> 输入：n = 3
> 输出：["((()))","(()())","(())()","()(())","()()()"]
> ```

解析：

这题稍微和前面的不太一样。我们总结一下，回溯法其实就是暴力枚举，把所有情况都列举出来，然后再把有效的加入到结果数组中。

对于这题来说我们可以枚举出所有 n个 '(' 和 n 个 ')' 字符构成的序列，然后我们检查每一个是否有效即可。

回溯法最重要是定义回溯函数的参数和 base case，怎么定义参数？总结以上的几种情况：

全排列需要标记数组 visited，子集需要标记位置 start , 这都很好理解。那么看这题，先考虑是否要标记数组，标记数组的用处是让结果里不含重复值，而这个左括号和右括号其实都是重复值，所以不需要用标记数组。标记位置 start 是为了不会回头遍历，对于这题来说也不需要。

唯一不同的是括号的数量，那么我们是不是可以用 left 和 right 两个变量来统计左右括号还剩余的数量，所以我们的回溯函数先定义成这样 `backtrack(left, right int, track string)`。

那么 base case 呢？

一是 left = right = 0 时，就将 track 加入到结果数组中，这很好理解，左右括号用完了嘛。

二是 left > right 就 return ，因为按照括号对的定义，在所有有效的括号对里，左括号的数量一定是大于右括号的，即剩余的括号数 left <= right，才是有效的 string 。

三是 left 或 right 有一个小于 0 就return，注意这里如果不能是等于 0 return，因为比如 n = 3， left = 0 其实就是track = "((("，接着就去掉一个左括号，这就把 "(((" 情况漏掉了，应该在 "((((" 的时候 return，才不会漏掉 "(((" 这种情况。

## 代码

### 全排列II

```go
func permuteUnique(nums []int) [][]int {
    var res [][]int

    //要判断重复元素，需要先排个序
    sort.Ints(nums)

    var backtrack func([]int, []bool)
    backtrack = func(track []int, visited []bool) {
        //需要判断如果 track 数组和 nums 数组长度相等的时候才把 track 加入到 res 中
        if len(track) == len(nums) {
            //注意这里需要 copy 一个 slice，不然因为引用的关系， res append 会有问题
            tmp := make([]int, len(track))
            copy(tmp, track)
            res = append(res, tmp)
            return
        }
        for i := 0; i < len(nums); i++ {
            //如果已经标记过该元素则跳过
            if visited[i] {
                continue
            }
            /*
            加上 !vis[i - 1]来去重主要是通过限制一下两个相邻的重复数字的访问顺序
            举个栗子，对于两个相同的数11，我们将其命名为1a1b, 1a表示第一个1，1b表示第二个1； 
            那么，不做去重的话，会有两种重复排列 1a1b, 1b1a， 我们只需要取其中任意一种排列； 
            为了达到这个目的，限制一下1a, 1b访问顺序即可。 比如我们只取1a1b那个排列的话，
            只有当visit nums[i-1]之后我们才去visit nums[i]， 也就是如果!visited[i-1]的话则continue。
            或者 visited[i-1] 也可以，即去掉 ! ，此时意思就是访问过了 nums[i-1] 就 continue。
            */
            if i > 0 && nums[i] == nums[i-1] && !visited[i-1] {
                continue
            }

            track = append(track, nums[i])
            visited[i] = true
            //开始下一步决策树
            backtrack(track, visited)
            //回溯，将上一次添加的 nums[i] 去掉，比如 nums = [1,2,3]，第一次从 1 开始遍历决策树，回溯就把 1 去掉，从 2 开始遍历决策树，依此类推。
            //再把标记数组标记回去
            track = track[:len(track)-1]
            visited[i] = false
        }
    }
    var track []int
    visited := make([]bool, len(nums))
    backtrack(track, visited)
    return res
}
```

### 子集

```go
func subsets(nums []int) [][]int {
  var res [][]int
    var track []int

    var backtrack func([]int, int)
    // 这里需要一个 start 参数来判断从第几个元素开始遍历
    backtrack = func(track []int, start int) {
        // 因为是子集，所以这里不需要判断数组长度是否相等，而是直接添加到 res 中
        tmp := make([]int, len(track))
        copy(tmp, track)
        res = append(res, tmp)

        // 一个大循环，遍历 1，2，3 选择将哪个元素作为 track 的第一个元素
        for i := start; i < len(nums); i++ {
            // 往 track 中添加元素
            track = append(track, nums[i])
            // 注意这里，下一次添加元素就是从下一个开始遍历，使用 start 来控制还能遍历的元素。比如 track = [1]，
            // 那么下一次就只能从 [2,3] 开始遍历，相当于把已经遍历的元素剔除
            backtrack(track, i+1)
            //撤销操作
            track = track[:len(track)-1]
        }
    }
    backtrack(track, 0)
    return res
}
```

### 子集II

```go
func subsetsWithDup(nums []int) [][]int {
    var res [][]int
    sort.Ints(nums)

    var backtrack func([]int, int)
    backtrack = func(track []int, start int) {
        tmp := make([]int, len(track))
        copy(tmp, track)
        res = append(res, tmp)

        for i := start; i < len(nums); i++ {
            // 排序之后，如果再遇到重复元素，则不选择此元素
            if i > start && nums[i] == nums[i-1] {
                continue
            }
            track = append(track, nums[i])
            backtrack(track, i+1)
            track = track[:len(track)-1]
        }
    }

    var track []int
    backtrack(track, 0)
    return res
}
```

### 组合

```go
func combine(n int, k int) [][]int {
    var res [][]int
    var track []int

    var backtrack func([]int, int)
    backtrack = func(track []int, start int) {
        if len(track) == k {
            tmp := make([]int, len(track))
            copy(tmp, track)
            res = append(res, tmp)
            return
        }
        for i := start; i <= n; i++ {
            track = append(track, i)
            backtrack(track, i+1)
            track = track[:len(track)-1]
        }
    }
    backtrack(track, 1)
    return res
}
```

### 括号生成

```go
func generateParenthesis(n int) []string {
    var res []string

    var backtrack func(int, int, string)
    backtrack = func(left, right int, track string) {
        if left == 0 && right == 0 {
            res = append(res, track)
            return
        }
        if left < 0 || right < 0 {
            return
        }
        // 不合法的括号对
        if left > right {
            return
        }

        track += "("
        backtrack(left-1, right, track)
        track = track[:len(track)-1]

        track += ")"
        backtrack(left, right-1, track)
        track = track[:len(track)-1]
    }

    var track string
    backtrack(n, n, track)
    return res
}
```

## 总结

从这些题可以看出，最普通的回溯算法可以使用**标记数组**或**标记位**来避免重复遍历，在遇到数组中有重复值的时候我们还需要排序再加判断条件。碰到其他的变种问题可以思考有哪些状态是可变的，再将其作为回溯函数的参数，这也有点像动态规划。总之多练习就能熟能生巧啦。
