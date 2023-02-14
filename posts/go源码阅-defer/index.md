# Go源码阅 Defer


defer 会在当前函数返回前执行传入的函数。常被用于关闭文件描述符，关闭数据库连接以及解锁资源。
作为 go 中的关键字，defer 的实现是由编译器和运行时共同完成的。
defer 的常见场景是在函数调用结束后完成一些收尾工作。

```go
defer tx.Rollback()
defer file.Close()
```

defer 传入的函数不是在退出代码块的作用域时执行的，只会在当前函数和方法返回之前被调用。

runtime._defer 结构体是延迟调用链表上的一个元素，所有的结构体会停过 link 字段串联成链表，每次有新的 defer 函数加入进来，会放到链表的头部，所以 FILO
```go
type _defer struct {
    siz       int32 // 参数和结果的内存大小
	started   bool 
	openDefer bool  // 开放编码
	sp        uintptr // 栈指针
	pc        uintptr // 调用方法的程序计数器
	fn        *funcval // defer 关键词中传入的函数
	_panic    *_panic // 触发延迟调用的结构体
	link      *_defer 
}
```
堆分配，栈分配和开放编码是处理 defer 关键字的三种方法（中间代码生成阶段）

### 堆上分配
堆上分配的 runtime._defer 结构体是默认的兜底方案，表示 defer 在编译器看来是函数调用

栈上分配是 1.13 做的优化

开放编码也是一种优化 defer 关键字的方法，开放编码只会在满足以下条件时启用：
- 1. 函数的 defer 数量少于或者等于 8 个
- 2. 函数的 defer 关键字不能在循环中执行
- 3. 函数的 return 语句与 defer 语句的乘积小于或者等于 15 个


**defer 函数的传入参数在定义时就已经明确，不论穿传入的参数时变量，表达式还是函数语句，都会先计算出实参结果，再随 defer 入栈**

```go
func main() {
    i := 1
    defer fmt.Println(i) // 1
    i++
    return
}
```

当 defer 类似闭包使用时，访问的总是循环中最后一个值
```go
func main() {
    for i := 0; i < 5; i++ {
        defer func() {
            fmt.Println(i) // 输出5
        }
    }
}
```

