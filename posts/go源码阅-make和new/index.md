# Go源码阅 Make和new


make: 初始化内置的数据结构，包括切片，哈希表和 Channel
new: 根据传入的类型分配一片内存空间并返回指向这片内存空间的指针

make
```go
slice := make([]int,10) // 一个包含data，cap和len的结构体 reflect.SliceHeader
hash := make(map[int]bool, 10) // 一个指向 runtime.hmap 结构体的指针
ch := make(chan int, 5) // 一个指向 runtime.hchan 结构体的指针
```

new 只能接收类型作为参数，然后返回一个指向该类型的指针（创建一个指向该类型零值的指针）
```go
// 方式一
i := new(int)

// 方式二
var v int
i := &v
// 方式一和二等价
```

## make
在编译期-类型检查阶段，Go 会将 make 关键字的 OMAKE 节点根据不同的参数类型转换成 OMAKESLICE，OMAKEMAP 和 OMAKECHAN 三种不同类型的节点，这些节点调用不同的函数来初始化数据结构

## new
new 方法会在中间代码生成阶段通过以下两个函数处理：
- gc.callnew 将关键字转换成 ONEWOBJ 类型的节点
- gc.state.expr 根据申请空间的大小分为两种情况
  - 如果申请的空间为 0，返回一个表示空指针的 zerobase 变量
  - 遇到其他情况时将关键字转换为 runtime.newobject 函数


