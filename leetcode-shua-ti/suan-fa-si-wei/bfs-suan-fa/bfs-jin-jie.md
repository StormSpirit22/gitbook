# BFS 进阶

这次来看一道 BFS 的困难题，巩固一下。

本文要介绍的题：

[滑动谜题（困难）](https://leetcode-cn.com/problems/sliding-puzzle/)

## 题解

### 滑动谜题

> 在一个 2 x 3 的板上（board）有 5 块砖瓦，用数字 1\~5 来表示, 以及一块空缺用 0 来表示.
>
> 一次移动定义为选择 0 与一个相邻的数字（上下左右）进行交换.
>
> 最终当板 board 的结果是 \[\[1,2,3],\[4,5,0]] 谜板被解开。
>
> 给出一个谜板的初始状态，返回最少可以通过多少次移动解开谜板，如果不能解开谜板，则返回 -1 。
>
> ```
> 输入：board = [[1,2,3],[4,0,5]]
> 输出：1
> 解释：交换 0 和 5 ，1 步完成
> ```

解析：

像这个问题其实也是枚举所有可能性，所有可能性其实就是 0 的位置用旁边几个数字来替换。

将邻近的数字一个个尝试成新的**棋盘**加入队列，等数组变成 target 就返回。这题可以将 board 压缩成一维数组更简单，

只要找到 0 在每个位置时的邻近索引就行。比如 0 在第 0 个索引的位置时，那么它的邻近索引就是 1 和 3，如下所示：

0 n x n x x

'n' 就是 0 旁边的数字，对应到一维数组的索引就是 1 和 3，因为只有这两个数字能和它交换位置。

依次类推所有 0 的位置附近的索引：

`[1,3], [0,2,4], [1,5], [0,4], [1,3,5], [2,4]`

而每次棋盘变化其实这些邻近位置的索引是不会变的，因为这些 0 的位置就是棋盘的索引 0 - 5 的位置。

我们只需要在每次的新棋盘里找到 0 的位置然后再分别将 0 和它的邻近数字交换得到新的棋盘加入队列即可。

## 代码

### 滑动谜题

```go
func slidingPuzzle(board [][]int) int {
    var steps int
    target := []int{1, 2, 3, 4, 5, 0}
    zeroIndex := make(map[int][]int)
    zeroIndex[0] = []int{1,3}
    zeroIndex[1] = []int{0,2,4}
    zeroIndex[2] = []int{1,5}
    zeroIndex[3] = []int{0,4}
    zeroIndex[4] = []int{1,3,5}
    zeroIndex[5] = []int{2,4}

    sboard := make([]int, 6)


    k := 0
    for i := 0; i < len(board); i++ {
        for j := 0; j < len(board[i]); j++ {
            sboard[k] = board[i][j]
            k++
        }
    }
    // 队列是棋盘的数组, 每次将可能的棋盘放入队列
    q := [][]int{sboard}
    // 记录已经存在过的棋盘，因为 int 数组无法作为 key，所以我们把它转化为 string
    visitedMap := make(map[string]struct{})

    for len(q) > 0 {
        n := len(q)
        for i := 0; i < n; i++ {
            // 从队列里拿出一个棋盘
            tmpBoard := q[i]
            // 如果该棋盘和目标数组一样就返回
            if isFinish(tmpBoard, target) {
                return steps
            }
            // 找到 0 的索引
            var index int
            for ; index < len(tmpBoard); index++ {
               if tmpBoard[index] == 0 {
                   break
               }
            }
            // 0 附近的数字数组
            neighborhoods := zeroIndex[index]

            for _, j := range neighborhoods {
                // 每次都复制一个新的棋盘来交换数字，因为在这个 0 的位置只能交换一次，所以交换完了一个位置得复位
                newBoard := make([]int, len(tmpBoard))
                copy(newBoard, tmpBoard)

                newBoard[index], newBoard[j] = newBoard[j], newBoard[index]

                // 如果不存在这样的棋盘就加入到队列中
                if _, ok := visitedMap[fmt.Sprint(newBoard)]; !ok {
                   visitedMap[fmt.Sprint(newBoard)] = struct{}{}
                    q = append(q, newBoard)
                }
            }
        }
        q = q[n:]
        steps++
    }
    return -1
}

func isFinish(s, t []int) bool {
    for i := range s {
        if s[i] != t[i] {
            return false
        }
    }
    return true
}
```

## 总结

BFS 算法的核心就是枚举，把所有可能性加入到队列，然后一个个取出来，再把之后的可能加入到队列中循环处理就行，多练练就会了，熟能生巧。
