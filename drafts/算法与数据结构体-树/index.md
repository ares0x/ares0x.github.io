# 算法与数据结构体 树


## 二叉树


### 常用的算法

#### 二叉树的前中后序遍历 (递归)

```go
// 前序（根 -> 左 -> 右）
func preOrder(root *TreeNode) []int{
	result := make([]int, 0)
	if root == nil {
		return result
	}
	var order func(node *TreeNode)
	order = func(node *TreeNode) {
		if node == nil {
			return 
		}
		result = append(result, node.val)
		order(node.left)
		order(node.right)
	}
	order(root)
	return result
} 

// 中序（左 -> 根 -> 右）
func inOrder(root *TreeNode) []int{
	result := make([]int, 0)
	if root == nil {
		return result
	}
	var order func(node *TreeNode)
	order = func(node *TreeNode) {
		if node == nil {
			return 
		}
		order(node.left)
		result = append(result, node.val)
		order(node.right)
	}
	order(root)
	return result
}

// 后序 (左 -> 右 -> 根)
func postOrder(root *TreeNode) []int{
	result := make([]int, 0)
	if root == nil {
		return result
	}
	var order func(node *TreeNode)
	order = func(node *TreeNode) {
		if node == nil {
			return 
		}
		order(node.left)
		order(node.right)
		result = append(result, node.val)
	}
	order(root)
	return result
}
```

#### 二叉树的前中后序遍历 (非递归)






#### 二叉树的层次遍历 (BFS)

```go
func bfs(root *TreeNode) [][]int{
	result := make([][]int,0)
	if root == nil {
		return result
	}
	queue := []*TreeNode{root}
	for len(queue) != 0 {
		size := len(queue)
		tmp := make([]int, 0)
		for i := 0; i < size; i++ {
			node := queue[0]
			queue := queue[1:]
			tmp := append(tmp, node.value)
			if node.left != nil {
				queue = append(queue, node.left)
			}
			if node.right != nil {
				queue = append(queue, node.right)
			}
		}
		result = append(result, tmp)
	}
	return result
}
```

#### 二叉树的深度遍历 (DFS)

```go
func dfs (root *TreeNode) [][]int{
	result := make([][]int,0)
	if root == nil {
		return result
	}
	stack := []*TreeNode{root}
	for len(stack) != 0 {
		tmp := make([]int, 0)
		size := len(stack)
		for i := 0; i < size; i++ {
			node := stack[len(stack)-1]
			stack = stack[:len(stack)-1]
			tmp = append(tmp,node.val)
			if node.right != nil {
				stack = append(stack, node.right)
			}
			if node.left != nil {
				stack = append(stack, node. left)
			}
		}
		result = append(result, tmp)
	}
	return result
}
```

### 二叉搜索树

- 空树也是一颗二叉搜索树

特征：
* 左子树上所有节点均小于它的根节点的值；
* 右子树上所有节点均大于它的根节点的值；

中序遍历二叉搜素树，升序排列



