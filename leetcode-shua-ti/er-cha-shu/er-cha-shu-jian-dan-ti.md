---
description: 二叉树的简单题
---

# 二叉树初阶

上一篇文章里已经介绍了各种二叉树的遍历方式，可以看到对于二叉树来说，**递归**是最简洁并且好理解的遍历方式。

所以对于二叉树的题，我们首先要想能不能用递归解决。而对于递归，我们不需要去考虑这个函数是否能实现我们要的功能，而是要**相信**函数一定能实现我们的功能，然后定义 base case 即可。

学以致用，现在先来看看 leetcode 上二叉树的简单题，练练手。下面会先介绍这些题目的思路，ac代码集中放到最后。建议读者先自己去做一下想一想，然后手写代码，如果直接先看代码，没有经过自己的思考，第二次碰到可能还是不会做。

下面选取了一些题目，虽然是 easy 难度，但是如果是第一次做也是不容易能马上写出来的，所以还是要多刷题！^\_^

本文要介绍的题目有：

[二叉树的最大深度（简单）](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

[对称的二叉树（简单）](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

[二叉树的镜像（简单）](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

[二叉树的最近公共祖先（简单）](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

[平衡二叉树（简单）](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)

[二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree/)



## 题目解析

### 二叉树的最大深度

> 输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

解析：

这题比较简单，直接对左右子树递归，取最大值然后+1即可。

### 对称的二叉树

> 请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。

解析：

要判断是不是对称二叉树，并不是简单地递归去判断左右子树的值是否相等。

比如第一个例子，到第二层的时候就不是判断左右子树的值相等了，而是需要判断 `left.left.val == right.right.val` 和 `left.right.val == right.left.val`。

那这怎么做呢？我们可以用一个辅助函数，参数值是 left, right， 每次只要判断 left.val 是否等于 right.val ，然后再将 left.left 和 right.right， left.right 和 right.left 递归就好了。

### 二叉树的镜像

> 请完成一个函数，输入一个二叉树，该函数输出它的镜像。

解析：

这题就是使用前序遍历，先将root左右子树节点交换，然后再继续遍历。

### 二叉树的最近公共祖先

> 给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。
>
> 百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

解析：

我们在最开始就有说递归的思想，就是要相信这个函数能实现我们的功能，然后定义base case。

这题有几个 base case 需要判断，如果 root == nil 或 root == p 或 root == q，那么直接返回 root ，因为如果root 是 p 或 q 中的任意一个节点，那么 p、q 的最近公共祖先肯定就是 root。

如果不是上述情况，那么 p、q 肯定在 root 的子树中。这样我们分别递归左右子树，返回的结果可以判断有三种情况，p、q 都在 root 左子树中，则返回左子树的遍历结果，或者都在右子树中，返回右子树的遍历结果，或者 p、q 分别存在左右子树，则返回 root。

你可能会疑惑这样找出来的公共祖先深度是否是最大的。其实是最大的，因为我们是自底向上从叶子节点开始更新的，所以在所有满足条件的公共祖先中一定是深度最大的祖先先被访问到。如果还是不理解，可以先看一下最后的代码。

### 平衡二叉树

> 输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

解析：

我们可以先计算 root 节点的左子树的最大深度和右子树的最大深度，然后判断两者相差是否大于 1。

但是仅仅这样判断是不够的，因为你不知道左子树或者右子树里面会不会有深度相差大于 1 的情况，所以还需要继续递归左右子树去判断。所以需要对每个节点的左右子树判断是否深度相差大于 1 即可。

### 二叉树的直径

> 给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过也可能不穿过根结点。
>
>  
>
> 示例 :
> 给定二叉树
>
>           1
>          / \
>         2   3
>        / \     
>       4   5    
> 返回 3, 它的长度是路径 [4,2,1,3] 或者 [5,2,1,3]。
>

解析：

我们可以计算出每个节点的左右子树的最大深度，然后相加就是经过这个节点的直径，比较出来这些直径里的最大值返回即可。比如节点 1 的左右子树的最大深度是 2 和 1，相加就是 3 。可以定义一个递归函数，返回值是这个节点的的最大深度，比如节点 2 的最大深度就是 2，这样节点 1 的左子树最大长度就是 2 了。而节点的最大深度就是左右子树的最大深度值 + 1，即 `max(left, right) + 1`。

## 代码

### 二叉树的最大深度

```go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    return max(maxDepth(root.Left), maxDepth(root.Right)) + 1
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 对称的二叉树

```go
func isSymmetric(root *TreeNode) bool {
  if root == nil {
    return true
  }
  return helper(root.Left, root.Right)
}

func helper(l, r *TreeNode) bool {
  if l == nil && r == nil {
    return true
  }
  if l == nil || r == nil || l.Val != r.Val {
    return false
  }
  return helper(l.Left, r.Right) && helper(l.Right, r.Left)
}
```

### 二叉树的镜像

```go
func mirrorTree(root *TreeNode) *TreeNode {
  if root == nil || (root.Left == nil && root.Right == nil) {
    return root
  }
  root.Left, root.Right = root.Right, root.Left
  mirrorTree(root.Left)
  mirrorTree(root.Right)
  return root
}
```

### 二叉树的最近公共祖先

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
  if root == nil {
    return nil
  }
  if root == p || root == q {
    return root
  }
  left := lowestCommonAncestor(root.Left, p, q)
  right := lowestCommonAncestor(root.Right, p, q)
  if left != nil && right != nil {
    return root
  }
  if left == nil {
    return right
  }
  return left
}
```

### 平衡二叉树

```go
func isBalanced(root *TreeNode) bool {
    if root == nil {
        return true
    }
    left := float64(depth(root.Left))
    right := float64(depth(root.Right))
    if math.Abs(left - right) > 1.0 {
        return false
    }
    return isBalanced(root.Left) && isBalanced(root.Right)
}

func depth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    return max(depth(root.Left), depth(root.Right)) + 1
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

```

### 二叉树的直径

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func diameterOfBinaryTree(root *TreeNode) int {
    var res int

    var depth func(*TreeNode) int
    depth = func(root *TreeNode) int {
        if root == nil {
            return 0
        }

        // 这里得到节点左右子树最大的长度，比如节点 1 左子树最大长度为 2，右子树最大长度为 1
        left := depth(root.Left)
        right := depth(root.Right)

        res = max(res, left+right)
        // 返回节点深度
        return max(left, right)+1
    }
    depth(root)
    return res
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

