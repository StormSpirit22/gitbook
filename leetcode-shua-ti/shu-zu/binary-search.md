# 二分搜索

二分搜索模板，根据 [我写了首诗，让你闭着眼睛也能写对二分搜索](https://labuladong.gitee.io/algo/2/21/61/) 改编而来。

## 模板

```go
package main

import "fmt"

func main() {
	nums := []int{1,2,3,3,3,5,6}
	fmt.Println(binarySearch(nums, 3))		// 输出索引 3
	fmt.Println(leftBound(nums, 3))			// 输出索引 2
	fmt.Println(rightBound(nums, 3))			// 输出索引 4
}

// 基础二分搜索
func binarySearch(nums []int, target int) int {
	left, right := 0, len(nums)-1

	for left <= right {
		mid := left + (right - left) / 2
		if nums[mid] ==  target {
			return mid
		} else if nums[mid] < target {
			left = mid + 1
		} else if nums[mid] > target {
			right = mid - 1
		}
	}
	return -1
}

// 寻找左侧边界的二分搜索
func leftBound(nums []int, target int) int {
	if len(nums) == 0 {
		return -1
	}
	left, right := 0, len(nums)
	for left < right {
		mid := left + (right - left) / 2
		if nums[mid] == target {
			right = mid
		} else if nums[mid] < target {
			left = mid + 1
		} else if nums[mid] > target {
			right = mid
		}
	}
	return left
}

// 寻找右侧边界的二分搜索
func rightBound(nums []int, target int) int {
	if len(nums) == 0 {
		return -1
	}

	left, right := 0, len(nums)
	for left < right {
		mid := left + (right - left) / 2
		if nums[mid] == target {
			left = mid + 1
		} else if nums[mid] < target {
			left = mid + 1
		} else if nums[mid] > target {
			right = mid
		}
	}
	return left - 1
}
```

这里面基础的二分搜索模板最简单也是用的最多的，这个必须得记住，也没什么太多细节。而寻找左侧和右侧边界的模板不太好记（如果能完全理解就更容易记住，可以参考上面的文章），实在不行就先用基础二分搜索找出目标值，然后往左遍历数组找到左侧边界，或者往右遍历找到右侧边界，这样也行，只要能做出来就行。

下面看看 2 道需要使用二分搜索解决的题目。

本文要介绍的题目有：

[875. 爱吃香蕉的珂珂（中等）](https://leetcode-cn.com/problems/koko-eating-bananas/)

[1011. 在D天内送达包裹的能力（中等）](https://leetcode-cn.com/problems/capacity-to-ship-packages-within-d-days/)



## 题目解析

### 爱吃香蕉的珂珂

> 珂珂喜欢吃香蕉。这里有 N 堆香蕉，第 i 堆中有 piles[i] 根香蕉。警卫已经离开了，将在 H 小时后回来。
>
> 珂珂可以决定她吃香蕉的速度 K （单位：根/小时）。每个小时，她将会选择一堆香蕉，从中吃掉 K 根。如果这堆香蕉少于 K 根，她将吃掉这堆的所有香蕉，然后这一小时内不会再吃更多的香蕉。  
>
> 珂珂喜欢慢慢吃，但仍然想在警卫回来前吃掉所有的香蕉。
>
> 返回她可以在 H 小时内吃掉所有香蕉的最小速度 K（K 为整数）。

解析：

能吃掉所有香蕉的速度应该是在一个区间内，而题目所求的就是这个区间的左边界，可以用二分法。

其中搜索区间的最小值是 1， 最大值是 max(piles) 这样求符合条件的值。因为一次最多吃掉 max(piles) 的就足够了，不需要吃更多了。这里可以直接套用上面的寻找左侧边界的模板，验证通过。

### 在 D 天内送达包裹的能力

> 传送带上的包裹必须在 days 天内从一个港口运送到另一个港口。
>
> 传送带上的第 i 个包裹的重量为 weights[i]。每一天，我们都会按给出重量（weights）的顺序往传送带上装载包裹。我们装载的重量不会超过船的最大运载重量。
>
> 返回能在 days 天内将传送带上的所有包裹送达的船的最低运载能力。

解析：

与上题类似，只是这里二分搜索的最小值应该是 max(weights)，因为船至少要能载起来所有货物，不然有货物载不起来肯定不行。最大值就是 sum(weights)。



## 代码

### 爱吃香蕉的珂珂

```go
func minEatingSpeed(piles []int, h int) int {
	n := len(piles)
	sort.Ints(piles)
	left, right := 1, piles[n-1]

	for left < right {
		mid := left + (right - left) / 2
		hours := getHours(mid, piles)
		// fmt.Printf("mid %d, hours %d \n", mid, hours)
		if hours == h {
			right = mid
		} else if hours < h {
			// 时间少了说明速度快了，可以减慢一下速度，缩小 mid
			right = mid
		} else if hours > h {
			// 速度慢了，需要加快一下速度，多吃点，增大 mid
			left = mid + 1
		}
	}
	return left
}

func getHours(num int, piles []int) int {
	var sum int
	for _, p := range piles {
        sum += (p + num - 1) / num
	}
	return sum
}
```

### 在 D 天内送达包裹的能力

```go
func shipWithinDays(weights []int, days int) int {
	//sort.Ints(weights)
	var sum int
	var maxWeight int
	for _, w := range weights {
		sum += w
		if maxWeight < w {
			maxWeight = w
		}
	}
	left, right := maxWeight, sum
	for left < right {
		mid := left + (right - left) / 2
		d := getDays(mid, weights)
		// fmt.Printf("mid %d, days %d, left %d, right %d \n", mid, d, left, right)
		if d == days {
			right = mid
		} else if d < days {
			right = mid
		} else if d > days {
			left = mid + 1
		}
	}
	return left
}

func getDays(ship int, weights []int) int {
	var days int
	for i := 0; i < len(weights); {
		days++
		count := weights[i]
		for count <= ship {
			i++
			if i == len(weights) {
				break
			}
			count += weights[i]
		}
	}
	return days
}
```

