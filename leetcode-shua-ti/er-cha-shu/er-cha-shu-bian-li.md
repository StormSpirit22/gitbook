---
description: 二叉树简单遍历
---

# 二叉树遍历

对于普通的二叉树遍历大家肯定都觉得这不是很简单吗，递归就解决了，但是对于基础还是要巩固一下。

前序遍历顺序：root节点，left节点，right节点。

中序遍历顺序：left节点，root节点，right节点。

后序遍历顺序：left节点，right节点，root节点。

可以看出前中后序其实就是root节点在哪个位置的遍历顺序。

另外还有层次遍历： 根据二叉树每一层的顺序来遍历。

### 二叉树前序遍历

递归代码：

```go
func preorderRecursive(root *TreeNode) []int {
  var res []int
  if root == nil {
    res = append(res, root.Val)
    res = append(res, preorderRecursive(root.Left)...)
    res = append(res, preorderRecursive(root.Right)...)
  }
  return res
}
```

是不是很简单？但是非递归的代码能一口气写出来吗？非递归遍历需要借助栈来实现：

```go
func preorderTraversal(root *TreeNode) []int {
  if root == nil {
    return nil
  }
  var result []int
  var stack []*TreeNode
  for root != nil || len(stack) > 0 {
    for root != nil {
      result = append(result, root.Value)
      stack = append(stack, root)
      root = root.Left
    }
    node := stack[len(stack) - 1]
    stack = stack[:len(stack) - 1]
    root = node.Right
  }
  return result
}
```

解释：

因为是前序遍历，所以先将 root 值放入数组，然后深度遍历 left 节点，当 left 节点为 null 时，出栈，再将right 节点赋值给 root 。比如有如下二叉树：

```
     3
   /   \
  9     20
 / \   /  \
2   8 15   7
```

先遍历 left 节点将 3、9、2 入栈，此时 result 数组： \[3, 9, 2] 。

接着 stack 出栈一个元素，node 此时为 2 节点，root = node.right 即为 nil ，接着下次循环，root = nil，直接出栈一个元素 9， root 为节点 8，接着循环，result数组就会变成 \[3, 9, 2, 8]，接着再出栈 3， 循环遍历右子树，这样最后 result 数组的值就是前序遍历的最终结果：\[3, 9, 2, 8, 20, 15, 7]。

### 二叉树中序遍历

递归代码：

```go
func inorderRecursive(root *TreeNode) []int {
  var res []int
  if root == nil {
    res = append(res, preorderRecursive(root.Left)...)
    res = append(res, root.Val)
    res = append(res, preorderRecursive(root.Right)...)
  }
  return res
}
```

非递归代码：

```go
func inorderTraversal(root *TreeNode) []int {
  if root == nil {
    return nil
  }
  var result []int
  var stack []*TreeNode
  for root != nil || len(stack) > 0 {
    for root != nil {
      stack = append(stack, root)
      root = root.Left
    }
    node := stack[len(stack) - 1]
    stack = stack[:len(stack) - 1]
    result = append(result, root.Value)
    root = node.Right
  }
  return result
}
```

解释：就是把result append的位置换了一下。

### 二叉树后序遍历

递归代码：

```go
func postorderTraversal(root *TreeNode) []int {
  var res []int
  if root == nil {
    res = append(res, preorderRecursive(root.Left)...)
    res = append(res, preorderRecursive(root.Right)...)
    res = append(res, root.Val)
  }
  return res
}
```

非递归代码：

```go
func postorderTraversal(root *TreeNode) []int {
  if root == nil {
    return nil
  }
  var result []int
  var stack []*TreeNode
  var lastVisited *TreeNode
  for root != nil || len(stack) > 0 {
    for root != nil {
      stack = append(stack, root)
      root = root.Left
    }
    node := stack[len(stack) - 1]
    if node.Right == nil || node.Right == lastVisited {
      stack = stack[:len(stack) - 1]
      result = append(result, node.Value)
      lastVisited = node
    } else {
      root = node.Right
    }
  }
  return result
}
```

解释：

在前序和中序遍历中，遍历的顺序都是左根右，只需要将值加入数组的位置变化一下就行。

但是后序遍历需要先访问左右子树的值，然后再访问根的值，但是在访问右子树之前，根节点就已经入栈了。

解决方法是先获取栈顶节点但不出栈，看如果这个节点是叶子节点，那么出栈，或者不是叶子节点但它的右子树已经访问过了再出栈，这样访问顺序就是左右根了。

所以后序遍历需要添加一个lastVisited的标记位，标记该节点已经遍历过。

### 层次遍历

递归代码：

```go
var result [][]int
func levelOrderRecursive(root *TreeNode) [][]int {
  if root == nil {
    return nil
  }
  levelOrderRecursiveHelper(root, 1)
  return result
}
​
func levelOrderRecursiveHelper(root *TreeNode, level int) {
  if root == nil {
    return
  }
  if len(result) < level {
    result = append(result, []int{})
  }
  result[level-1] = append(result[level-1], root.Value)
  levelOrderRecursiveHelper(root.Left, level + 1)
  levelOrderRecursiveHelper(root.Right, level + 1)
}
​
```

非递归代码：

```go
func levelOrderTraversal(root *TreeNode) []int {
  if root == nil {
    return nil
  }
  var result []int
  var queue []*TreeNode
  queue = append(queue, root)
  for len(queue) > 0 {
    l := len(queue)
    for i := 0; i < l; i ++ {
      node := queue[0]
      queue = queue[1:]
      result = append(result, node.Value)
      if node.Left != nil {
        queue = append(queue, node.Left)
      }
      if node.Right != nil {
        queue = append(queue, node.Right)
      }
    }
  }
  return result
}
```

解释: 层次遍历递归解法借助了level参数来判断递归到哪一层。非递归解法就比较常规了，用队列来解决。本质上都是bfs算法。

#### 总结

虽然二叉树遍历是最基础的，但是想完全掌握递归和非递归的解法还是需要一些时间的，大家都学废了吗？
