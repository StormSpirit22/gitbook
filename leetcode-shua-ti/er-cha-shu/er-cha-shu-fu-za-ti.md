---
description: 二叉树的困难题
---

# 二叉树高阶

好了，二叉树的简单题、中等题我们都做过了一部分，基本的思路都了解了，这里看一下leetcode上的二叉树的复杂题，放心，只有一题。

本文要介绍的题目有：

[二叉树的序列化与反序列化（困难）](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree/)

## 题目解析

### 二叉树的序列化与反序列化

> 序列化是将一个数据结构或者对象转换为连续的比特位的操作，进而可以将转换后的数据存储在一个文件或者内存中，同时也可以通过网络传输到另一个计算机环境，采取相反方式重构得到原数据。
>
> 请设计一个算法来实现二叉树的序列化与反序列化。这里不限定你的序列 / 反序列化算法执行逻辑，你只需要保证一个二叉树可以被序列化为一个字符串并且将这个字符串反序列化为原始的树结构。
>
> 提示: 输入输出格式与 LeetCode 目前使用的方式一致，详情请参阅 LeetCode 序列化二叉树的格式。你并非必须采取这种方式，你也可以采用其他的方法解决这个问题。

解析：

这道题更类似于设计题，可以先想想序列化怎么做，序列化我们之前在重复子树中已经做过类似的，这次是把整棵树都序列化，也大同小异。

先介绍递归的方法，使用前序遍历，我们假定如果节点为nil，则用“#”来代替，节点之间用“,”来分隔。

我们需要一个变量data，当root == nil时，data += "#" + ","，否则 data += root.Val，然后递归root.Left和root.Right，这样序列化就完成了。

比如题目中的例子，root = \[1,2,3,null,null,4,5]，序列化之后应该返回 “1,2,#,#,3,4,#,#,5,#,# ” 这样才对。

接着介绍反序列化。同样，我们需要声明一个全局变量nodes，作为strings.Split(data, ",")的接收者。

即将每个节点放到数组中，然后拿出数组第一个值，并在数组中去掉这个值，然后以这个值初始化root节点，接着使用nodes去递归赋值root.Left和root.Right即可。

后序遍历也是可以的，大家可以自己去试试，但是中序遍历不行，因为中序遍历的序列化无法确定root值是哪个。

再介绍迭代的方法，也就是层次遍历，层次遍历就是借助队列来实现，序列化比较简单，这里跳过。要注意的是反序列化也需要借助队列，出队一个元素，需要再入队后面两个元素，即left，right。

## 代码

### 二叉树的序列化与反序列化（前序遍历）

```go
type Codec struct {
}
​
func Constructor() Codec {
  c := Codec{}
  return c
}
​
// Serializes a tree to a single string.
func (this *Codec) serialize(root *TreeNode) string {
  data := serializeHelper(root)
  data = data[:len(data)-1]
  return data
}
​
func serializeHelper(root *TreeNode) string {
  var data string
  if root == nil {
    data += "#" + ","
    return data
  }
  data += strconv.Itoa(root.Val) + ","
  data += serializeHelper(root.Left)
  data += serializeHelper(root.Right)
  return data
}
​
//需要一个全局变量nodes，如果helper是通过传值来进行nodes的传递，那么回溯时nodes应该被去掉的值没有被去掉，
//而全局变量可以避免这个问题，或者在Codec结构体里声明一个nodes变量，helper作为成员函数也是可以的。
var nodes []string
// Deserializes your encoded data to tree.
func (this *Codec) deserialize(data string) *TreeNode {
  nodes = strings.Split(data, ",")
  return deserializeHelper()
}
​
func deserializeHelper() *TreeNode {
  if len(nodes) == 0 {
    return nil
  }
  v := nodes[0]
  nodes = nodes[1:]
  if v == "#" {
    return nil
  }
  num, _ := strconv.Atoi(v)
  root := &TreeNode{Val: num}
  root.Left = deserializeHelper()
  root.Right = deserializeHelper()
  return root
}
​
```

### 二叉树的序列化与反序列化（层次遍历）

```go
type Codec struct {
    
}
​
func Constructor() Codec {
    return Codec{}
}
​
// Serializes a tree to a single string.
func (this *Codec) serialize(root *TreeNode) string {
    if root == nil {
        return ""
    }
    var res string
    queue := list.New()
    queue.PushBack(root)
    for queue.Len() > 0 {
        n := queue.Len()
        for i := 0; i < n; i++ {
            tmp := queue.Front()
            tmpNode := tmp.Value.(*TreeNode)
            queue.Remove(tmp)
            if tmpNode == nil {
                res += "#,"
            } else {
                res += strconv.Itoa(tmpNode.Val) + ","
                queue.PushBack(tmpNode.Left)
                queue.PushBack(tmpNode.Right)
            }
        }
    }
    res = res[:len(res)-1]
    return res
}
​
// Deserializes your encoded data to tree.
func (this *Codec) deserialize(data string) *TreeNode {   
    if len(data) == 0 {
        return nil
    }
    str := strings.Split(data, ",")
    if len(str) == 0 {
        return nil
    }
    t, _ := strconv.Atoi(str[0])
    root := &TreeNode{Val: t}
    queue := list.New()
    queue.PushBack(root)
    for i := 1; i < len(str); i++ {
        tmp := queue.Front()
        tmpNode := tmp.Value.(*TreeNode)
        queue.Remove(tmp)
        if str[i] != "#" {
            a, _ := strconv.Atoi(str[i])
            tmpNode.Left = &TreeNode{Val: a}
            queue.PushBack(tmpNode.Left)
        }
        i++
        if str[i] != "#" {
            a, _ := strconv.Atoi(str[i])
            tmpNode.Right = &TreeNode{Val: a}
            queue.PushBack(tmpNode.Right)
        }
    }
    return root
}
```
