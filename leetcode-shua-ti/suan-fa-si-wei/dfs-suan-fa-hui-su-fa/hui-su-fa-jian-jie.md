# 回溯法初阶

## 回溯法

回溯法是动态规划的一个分支，也是一种递归的思想，**解决一个回溯问题，实际上就是一个决策树的遍历过程**。回溯其实是使用的DFS的思路，但是它要主动撤销你刚刚做的选择，即尝试所有的可能性。

在回溯的算法里，你只需要思考 3 个问题：

1、路径：也就是已经做出的选择。

2、选择列表：也就是你当前可以做的选择。

3、结束条件：也就是到达决策树底层，无法再做选择的条件。

回溯法是有代码模板的，我们先看一下模板，再通过一个例子来介绍回溯法的基本的思想。

```go
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

本文要介绍的题：

[全排列](https://leetcode-cn.com/problems/permutations/)

## 题解

### 全排列

> 给定一个不含重复数字的数组 `nums` ，返回其 **所有可能的全排列** 。你可以 **按任意顺序** 返回答案。
>
> ```
> 输入：nums = [1,2,3]
> 输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
> ```

解析：

题目意思非常简单，给出一个数组的全排列，就是给出一个数组排列的所有可能性，我们可以尝试按顺序列举出每一种可能性，画出一个决策树（该图参考了[题解](https://leetcode-cn.com/problems/permutations/solution/hui-su-suan-fa-python-dai-ma-java-dai-ma-by-liweiw/)）：

![](<../../../.gitbook/assets/jueceshu (1).png>)

可以看到这个图很清晰的表达出回溯的意思，比如先选择 1，再选择 2， 最后选择 3，这样得到一个序列 \[1,2,3]，然后我们需要依次撤销上面几种情况转而选择其他数字。我们直接看一下该题的解法：

```go
func permute(nums []int) [][]int {
	var res [][]int

	n := len(nums)
	var backtrack func([]int, map[int]bool)
	backtrack = func(track []int, visited map[int]bool) {
		if n == len(track) {
			tmp := make([]int, n)
			copy(tmp, track)
			res = append(res, tmp)
			return
		}

		for i := range nums {
			if !visited[i] {
				track = append(track, nums[i])
				visited[i] = true
				backtrack(track, visited)
				track = track[:len(track)-1]
				visited[i] = false
			}
		}
	}
	visited := make(map[int]bool)
	backtrack([]int{}, visited)
	return res
}
```

我们定义了一个 backtrack 的回溯函数，参数是一个序列slice。

在该函数里，最核心的代码其实就是对 nums 遍历，如果 track 的长度等于 nums，也就是找到了其中一个序列，那么我们就把该 slice 添加到返回数组中。这也就是 DFS 的 base case 。

如果 track 的长度小于 nums，如果 track 里没有该元素，那么就将遍历到的 nums\[i] 加入到 track 中，接着递归，然后下一步就是撤销上一步的操作，这和我们的算法思路是一样的，即如果这次从 1 开始遍历，那么下次就会把 1 从 track 中去掉，track 的第一个元素就是 2 了。即决策树选择其他可能性。

然后在 backtrack 参数中加一个 visited map，不用每次都遍历 track 来判断是否存在该元素。

代码模板可以用上面那套，或者自己简化再总结出自己喜欢的模板，适合自己的才是最好的。

## 总结

回溯算法其实就是尽可能地去选择所有可能性，核心思想就是 做选择，递归，再撤销选择，记住这个，再多练几道题就 ok 了。

值得注意的是回溯法时间复杂度都不可能低于 O(N!)，因为穷举整棵决策树是无法避免的。**这也是回溯算法的一个特点，不像动态规划存在重叠子问题可以优化，回溯算法就是纯暴力穷举，复杂度一般都很高**。

而回溯法和动态规划的最大区别在于，动态规划一般都能用 dp table 或者备忘录进行剪枝优化，因为有重叠子问题，而要使用回溯法的时候一般都没有重叠子问题，就是枚举思路，所以复杂度一般很高。

最后关于是使用回溯法还是动态规划，我们可以从决策树入手，看情况使用哪种方法更好，都是熟能生巧，多做题慢慢就会有思路啦。
