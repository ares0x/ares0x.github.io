# Go源码阅读 数组


## 概述
数组是由**相同类型**元素的集合组成的数据结构，计算机会为数组分配一块连续的内存来保存其中的元素。可以通过数组中元素的索引快速访问特定元素。
从两个维度描述数组：数组中存储的元素类型和数组最大能存储的元素个数。
- Go 数组在初始化后大小无法改变。


```go
// cmd/compile/internal/types/type.go NewArray
// NewArray returns a new fixed-length array Type.
func NewArray(elem *Type, bound int64) *Type {
	if bound < 0 {
		Fatalf("NewArray: invalid bound %v", bound)
	}
	t := New(TARRAY)
	t.Extra = &Array{Elem: elem, Bound: bound}
	t.SetNotInHeap(elem.NotInHeap())
	return t
}
```

## 初始化
Go 数组有两种初始化方式：
1. 显式初始化
```go
arr := [2]int{1,2}
```
2. 隐式初始化 (此方式会在编译器期 - 类型检查阶段被转换成第一种，编译器会对数组大小推导)
```go
arr := [...]int{1,2}
```

如果数组的元素数量小于或者等于4时，会直接将数组中的元素放在栈上；当数组元素数量大于4时，会将数组元素放在静态区，并在运行时取出。

无论是在栈上还是静态区，数组在内存中都是一连串的内存空间。可以通过指向数组开头的指针，元素的数量以及元素类型占的空间大小表示数组。

数组越界也是在类型检查期间被检查出。



