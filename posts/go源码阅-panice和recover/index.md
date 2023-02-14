# Go源码阅 Panice和recover


- panic 能够改变程序的控制流，调用 panic 后会立刻停止执行当前函数的剩余代码，并在当前 goroutine 中低轨执行调用方 defer
- revocer 可以终止 panic 造成的程序崩溃。它是一个只能在 defer 中发挥作用的函数。

现象：
- panic 只会触发当前 goroutine 的 defer
- recover 只有在 defer 中调用才会生效
- panic 允许在 defer 中嵌套多次调用

defer 关键字对应的 runtime.deferproc 会将延迟调用函数与调用方所在 goroutine 进行关联。当程序崩溃时只会调用当前 goroutine 的延迟函数。

```go
func main() {
	defer fmt.Println("in main")
	if err := recover(); err != nil {
		fmt.Println(err)
	}

	panic("unknown err")
}
```
上述程序中程序没有正常退出。recover 只有在发生 panic 之后调用才会生效。所以 recover 应该在 defer 中使用

多次调用 panic 也不会影响 defer 函数的正常执行。


## 数据结构
```go
type _panic struct {
	argp      unsafe.Pointer // pointer to arguments of deferred call run during panic; cannot move - known to liblink
	arg       interface{}    // argument to panic
	link      *_panic        // link to earlier panic
	pc        uintptr        // where to return to in runtime if this panic is bypassed
	sp        unsafe.Pointer // where to return to in runtime if this panic is bypassed
	recovered bool           // whether this panic is over
	aborted   bool           // the panic was aborted
	goexit    bool
}
```
argp 是指向 defer 调用时参数的指针
arg 是调用 panic 时传入的参数
link 指向了更早调用的 runtime._panic 结构 （panic 可以被连续调用）
recovered 表示当前 runtimme._panic 是否被 recover 回复
aborted 表示当前 panic 是否被强行终止

## 原理
panic 终止程序的原理：
编译器将关键字 panic 转换为 runtime.gopanic，并按照以下步骤执行：
- 1. 创建新的 runtime._panic 并添加到当前所在 goroutine 的 _panic 链表最前面
- 2. 在循环中不断从当前 goroutine 的 _defer 链表中获取 runtime._defer 并调用 runtime._reflectcall 运行延迟调用
- 3. 调用 runtime.fatalpanic 中止整个程序


## 崩溃恢复

编译器将 recover 转换为 runtime.gorecover。
如果当前 goroutine 并没有调用 panic，函数会继续执行；




