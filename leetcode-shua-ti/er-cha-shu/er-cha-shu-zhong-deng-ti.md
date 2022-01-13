---
description: 二叉树的中等题
---

# 二叉树进阶

上次已经做完了二叉树的简单题，这次来看看难度为中等的题。

首先有一个点需要讲清楚，就是二叉树的解法基本都是通过遍历。

**那么如何判断我们应该用前序还是中序还是后序遍历的框架**？

**根据题意，思考一个二叉树节点需要做什么，到底用什么遍历顺序就清楚了**。

先看题，最好先点到题目里做一遍。\\

本文要介绍的题目有：

[二叉树展开为链表（中等）](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/)

[填充每个节点的下一个右侧节点指针（中等）](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)

[最大二叉树（中等）](https://leetcode-cn.com/problems/maximum-binary-tree/)

[从前序与中序遍历序列构造二叉树（中等）](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

[从中序与后序遍历序列构造二叉树（中等）](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

[寻找重复的子树（中等）](https://leetcode-cn.com/problems/find-duplicate-subtrees/)

## 题目解析

### 二叉树展开为链表

> 给你二叉树的根结点 root ，请你将它展开为一个单链表：
>
> 展开后的单链表应该同样使用 TreeNode ，其中 right 子指针指向链表中下一个结点，而左子指针始终为 null 。 展开后的单链表应该与二叉树 先序遍历 顺序相同。

解析：

想象一个最简单的情况，对于一个 root 节点来说，假设它的左右子树都只有一个节点，那么把树展开为链表，应该是先把 root.Right 保存下来，然后 `root.Right = root.Left` ，最后遍历新的右子树到它的最右节点，然后把之前的右子树接上去，这一块逻辑应该比较清晰。

然后就是递归处理这一块代码就行了，有个问题就是这题用前序遍历还是后序呢？

其实这两种都可以通过，区别在于需要的时间和空间不同，可以想象一下，如果是前序遍历，先将整个左子树拼接到右边，再去遍历整个右子树，整个右子树会拉得特别长，再一个个递归下去，这耗费的时间和空间就特别多了。

而如果使用后序遍历，先遍历左右子树，分别把左右子树处理完成后，最后处理root节点，这样使用的时间和空间会少很多。

即前序遍历是自顶向下的方法，而后序遍历是自底向上的方法，再通俗一点地说，对于这道题，使用后序遍历是使用归并的思想，先处理小的情况，最后把两边处理好的子树统一拼接起来，这样效率上会快很多。

这个思路是递归的解法，迭代的解法可以去看官方解答。

### 填充每个节点的下一个右侧节点指针

> 给定一个 **完美二叉树** ，其所有叶子节点都在同一层，每个父节点都有两个子节点。填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 NULL。
>
> 初始状态下，所有 next 指针都被设置为 NULL。

解析：

这题同样也是想象一个最简单的情况，假设有这样一个前序遍历的二叉树，\[1, 2, 3]，是不是非常简单，只需要把2节点的next指针指向3，即 left.Next = right。

我们再延伸一层，比如题目中的例子，不仅左右子树各自next指针要赋值，它们之间的节点5的next要指向右子树的节点6，这应该怎么做呢？

其实很简单，有点像简单题里的对称二叉树的遍历方式，

只需要将 root.Left.Right 和 root.Right.Left 代入到上面的表达式 `left.Next = right` 中 left 和 right 中即可。

即对于root来说，left 是它的左子树，right 是它的右子树，我们需要遍历 left.left，left.right、 left.right，right.left、right.left，right.right 即可。

### 最大二叉树

> 给定一个不含重复元素的整数数组 nums 。一个以此数组直接递归构建的 最大二叉树 定义如下：
>
> 二叉树的根是数组 nums 中的最大元素。 左子树是通过数组中 最大值左边部分 递归构造出的最大二叉树。 右子树是通过数组中 最大值右边部分 递归构造出的最大二叉树。 返回有给定数组 nums 构建的 最大二叉树 。

解析：

这题描述就告诉用递归了。

那么还是考虑最简单情况，比如 nums = \[3,2,1]，我们先找到最大的值为 3，索引为 0，然后构造 root 节点，root.Left 为 3 左边的部分。

即递归参数为 nums\[:0] 即空，root.Right的递归参数为 nums\[0:]，这样就行了，用递归很容易做出来。

### 从前序与中序遍历序列构造二叉树

> 根据一棵树的前序遍历与中序遍历构造二叉树。
>
> **注意:** 你可以假设树中没有重复的元素。

解析：

这类题先要找到根节点，再通过根节点找到左子树和右子树的两种遍历序列，作为参数递归下去构造二叉树即可。

在这题里需要先通过前序遍历的根节点的值，找到中序遍历根节点的索引，再根据中序遍历找到左右子树的长度，即计算出从0到根节点的中序遍历数组的**长度**，或从根节点到数组末尾的**长度**。

### 从中序与后序遍历序列构造二叉树

> 根据一棵树的中序遍历与后序遍历构造二叉树。
>
> **注意:** 你可以假设树中没有重复的元素。

解析：

和上道题类似，只不过后序遍历递归时，右子树区间是左子树的长度到后序遍历数组的长度-1。-1就是把root节点去掉。

### 寻找重复的子树

> 给定一棵二叉树，返回所有重复的子树。对于同一类的重复子树，你只需要返回其中任意**一棵**的根结点即可。
>
> 两棵树重复是指它们具有相同的结构以及相同的结点值。

解析：

这题不是简单的遍历就能完成，它需要记录每个子树，然后再看是否重复，返回重复的子树。

那么怎么记录子树呢？我们可以将二叉树序列化，其实只需要前序与后序遍历即可，即`root.Val + "," + left + "," + right 或 left + "," + right + "," + root.Val`，对于节点为nil我们返回"#"。但是不可以使用中序遍历来进行序列化，因为对于像下面这种情况，中序遍历的序列是一样的，但是对于题目来说这不是同一类的重复子树：

```
   0           0
  /              \
 0                0
```

中序遍历：#0#0# 和 #0#0#

前序遍历：00### 和 0#0##

后序遍历：##0#0 ###00

然后我们就可以将序列化的值存到map里，如果有重复就加入到结果数组中。

## 代码

### 二叉树展开为链表

```go
func flatten(root *TreeNode)  {
    if root == nil {
        return 
    }
    flatten(root.Left)
    flatten(root.Right)
    left := root.Left
    right := root.Right
​
    root.Right = left
    root.Left = nil
​
    tmp := root
    for tmp.Right != nil {
        tmp = tmp.Right
    }
    tmp.Right = right
}
```

### 填充每个节点的下一个右侧节点指针

```go
func connect(root *Node) *Node {
    if root == nil {
        return nil
    }
    left := root.Left 
    right := root.Right
    connectTwo(left, right)
    return root
}
​
func connectTwo(left, right *Node) {
    if left != nil {
        left.Next = right
        connectTwo(left.Left, left.Right)
        connectTwo(left.Right, right.Left)
        connectTwo(right.Left, right.Right)
    }
}
```

### 最大二叉树

```go
func constructMaximumBinaryTree(nums []int) *TreeNode {
    if len(nums) == 0 {
        return nil
    }
    maxIndex := findMaxIndex(nums)
    root := &TreeNode{Val: nums[maxIndex]}
    root.Left = constructMaximumBinaryTree(nums[:maxIndex])
    root.Right = constructMaximumBinaryTree(nums[maxIndex+1:])
    return root
}
​
func findMaxIndex(nums []int) int {
    max := -1
    var res int
    for i := range nums {
        if nums[i] > max {
            max = nums[i]
            res = i
        }
    }
    return res
}
```

### 从前序与中序遍历序列构造二叉树

```go
func buildTree(preorder []int, inorder []int) *TreeNode {
	if len(preorder) == 0 {
		return nil
	}
	root := &TreeNode{Val: preorder[0]}

	var index int
	for i := range inorder {
		if inorder[i] == preorder[0] {
			index = i
			break
		}
	}
	root.Left = buildTree(preorder[1:1+index], inorder[:index])
	root.Right = buildTree(preorder[1+index:], inorder[index+1:])
	return root
}
```

### 从中序与后序遍历序列构造二叉树

```go
func buildTree(inorder []int, postorder []int) *TreeNode {
	if len(inorder) == 0 {
		return nil
	}
	n := len(postorder)
	root := &TreeNode{Val: postorder[n-1]}
	var index int
	for i := range inorder {
		if inorder[i] == root.Val {
			index = i
			break
		}
	}
	postorder = postorder[:n-1]
	root.Left = buildTree(inorder[:index], postorder[:index])
	root.Right = buildTree(inorder[index+1:], postorder[index:])

	return root
}
```

### 寻找重复的子树

```go
var treeMap map[string]*TreeNode
var res []*TreeNode
func findDuplicateSubtrees(root *TreeNode) []*TreeNode {
    res = []*TreeNode{}
    treeMap = make(map[string]*TreeNode)
    traverse(root)
    return res
}
​
func traverse(root *TreeNode) string {
    if root == nil {
        return "#"
    }
    var str string
    left := traverse(root.Left)
    right := traverse(root.Right)
    str = left + "," + right + "," + strconv.Itoa(root.Val)
    if v, ok := treeMap[str]; ok {
        if v != nil {
            res = append(res, v)
        }
        treeMap[str] = nil
    } else {
        treeMap[str] = root
    }
    return str
}
```
