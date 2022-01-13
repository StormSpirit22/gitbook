# 回溯法高阶

这次我们来看两道与回溯法相关的困难题。

本文要介绍的题：

[N 皇后（困难）](https://leetcode-cn.com/problems/n-queens/)

[解数独（困难）](https://leetcode-cn.com/problems/sudoku-solver/)

## 题解

### N皇后

> n 皇后问题 研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。
>
> 给你一个整数 n ，返回所有不同的 n 皇后问题 的解决方案。
>
> 每一种解法包含一个不同的 n 皇后问题 的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。
>
> ```
> 输入：n = 4
> 输出：[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
> 解释：如上图所示，4 皇后问题存在两个不同的解法。
> ```

解析：

这题应该是非常经典的一道题了，大学的时候就讲过。这次我们重新来做一遍。

还是用回溯法解决，我们先初始化一个 n \* n 的棋盘 board，

从上到下遍历，因为每行只能最多放置一个 Q，那么我们就一行行遍历。

然后定义回溯函数，参数应该是 board 和当前遍历的行 row，再在每一行对列进行遍历，

尝试在每个列位置放入 Q ，然后递归下一行，再回溯回来。

最后写个 isValid 方法，判断左上右上和同一列有没有冲突即可。

其实回溯的框架写出来就比较简单了，主要是 isValid 的方法要注意，分别要检查列是否冲突，左上斜线是否冲突，右上斜线是否冲突。

因为我们是一行一行遍历的，所以只需要检查以上三种情况即可。

### 解数独

> 编写一个程序，通过填充空格来解决数独问题。
>
> 数独的解法需 遵循如下规则：
>
> 数字 1-9 在每一行只能出现一次。 数字 1-9 在每一列只能出现一次。 数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。（请参考示例图） 数独部分空格内已填入了数字，空白格用 '.' 表示。

解析：

这题看起来很复杂很唬人，其实理解了了N皇后，这题其实并不是太难了。

还是回溯的套路，唯一要注意的是判断该一格可以放哪些数字，也就是 isValid 方法是正确的那这道题就没问题了。

回溯函数的参数是 board, row, col，因为这次没法像N皇后一样只遍历行就行，而是要遍历所有的格子，所以参数里 row, col 都要。

其次要注意的是之前的回溯函数基本都没返回值，因为大都是在回溯的过程中将合法的值统计起来。

但是这题说了只有一种解法，那么我们要设置一个 bool 返回值，如果找不到就返回 false， 找到了就返回 true，不用再继续找了。

最后是一些 base case：

如果 row 等于 len(board) 时其实就是所有都遍历过了。

如果 col 等于len(board) 那么应该从下一行第 0 列开始遍历。

找到了一种解法，直接返回 true。

另外 isValid 方法要判断这一行这一列是否有重复，还要判断当前数字所在的九宫格里是否有重复数字，只要写对了 isValid 方法其他的就好写了，这题主要是遍历的一些细节要特别注意，多看看就会了。

## 代码

### N皇后

```go
func solveNQueens(n int) [][]string {
    var res [][]string
    board := make([][]byte, n)

    // 初始化棋盘
    for i := 0; i < n; i++ {
        board[i] = make([]byte, n)
        for j := 0; j < n; j++ {
            board[i][j] = '.'
        }
    }

    // 路径：board 中小于 row 的那些行都已经成功放置了皇后
    // 选择列表：第 row 行的所有列都是放置皇后的选择
    // 结束条件：row 超过 board 的最后一行
    var backtrack func([][]byte, int)
    backtrack = func(board [][]byte, row int) {
        if row == len(board) {
            var tmp []string
            for i := range board {
                s := string(board[i])
                tmp = append(tmp, s)
            }
            res = append(res, tmp)
            return
        }

        for col := 0; col < n; col++ {
            if !isValid(board, row, col) {
                continue
            }
            board[row][col] = 'Q'
            backtrack(board, row+1)
            board[row][col] = '.'
        }
    }
    backtrack(board, 0)
    return res
}

func isValid(board [][]byte, row, col int) bool {
    // 检查列是否冲突
    for i := 0; i < len(board); i++ {
        if board[i][col] == 'Q' {
            return false
        }
    }

    // 检查左上斜线是否冲突
    var i, j int
    i = row - 1
    j = col - 1
    for ; i >= 0 && j >= 0; {
        if board[i][j] == 'Q' {
            return false
        }
        i--
        j--
    }

    // 检查右上斜线是否冲突
    i = row - 1
    j = col + 1
    for ; i >= 0 && j < len(board); {
        if board[i][j] == 'Q' {
            return false
        }
        i--
        j++
    }
    return true
}
```

### 解数独

```go
func solveSudoku(board [][]byte)  {
    n := len(board)
    var backtrack func ([][]byte, int, int) bool
    backtrack = func(board [][]byte, row, col int) bool {
        // 棋盘所有空间都遍历了，可以返回
        if row == n {
            return true
        }
        // 走到棋盘最右了，去下一行遍历
        if col == n {
            return backtrack(board, row+1, 0)
        }
        // 不是点的不需要改
        if board[row][col] != '.' {
            return backtrack(board, row, col+1)
        }
        // 找到所有的合法的数字
        valids := getValid(board, row, col)
        for _, v := range valids {
            board[row][col] = v
            // 这里需要判断一下，方法如果返回 true 了那么就不用回溯了
            if backtrack(board, row, col+1) {
                return true
            }
            board[row][col] = '.'
        }
        // 该种方法没有找到一条可行的路，返回 false
        return false
    }
    backtrack(board, 0, 0)
}

// 判断某个位置上能够放的数字
func getValid(board [][]byte, row, col int) []byte {
    exist := make([]bool, 10)
    var valid []byte
    // 行列
    for i := 0; i < 9; i++ {
        if board[row][i] != '.' {
            exist[board[row][i]-'0'] = true
        }
        if board[i][col] != '.' {
            exist[board[i][col]-'0'] = true
        }
        // 九宫格
        sudoku := board[(row/3)*3 + i/3][(col/3)*3 + i%3]
        if sudoku != '.' {
            exist[sudoku-'0'] = true
        }
    }

  // 返回 1-9 中所有的合法的数字
    for i := 1; i < 10; i++ {
        if !exist[i] {
            valid = append(valid, byte(i) + '0')
        }
    }
    return valid
}
```

## 总结

可以看出这两道题还是有些难度的，不过只要会了其中一道，类似的也不是太难了。核心还是回溯的框架，满足结束条件就加入到结果，在选择列表里做选择，回溯，撤销选择，然后就是要正确选取回溯函数的参数，再注意一些遍历的细节就可以了。
