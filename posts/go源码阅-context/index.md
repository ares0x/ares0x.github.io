# Go源码阅 Context


context.Context 在 go 中被用于设置截至时间，同步信号，传递请求相关值的结构体。
context.Context 是一个 interface，定义了 4 个方法
```go
type Context interface {
    Deadline() (deadline time.Time, ok bool) //返回 context.Context 被取消的时间，即完成工作的截止时间
    Done() <-chan struct{} // 返回一个 channel，该 channel 会在当前工作完成或者上下文被取消后关闭
    Err() error // 返回 context.Context 结束的原因，它只会在 Done 方法对应的 Channel 关闭时返回非空的值。如果 context.Context 被取消，返回 Canceled 错误；context.Context 超时，返回 DeadlineExceed 错误
    Value(key interface{}) interface{} // 从 context.Context 中获取键对应的值。对于同一个上下文来说，多次调用 Value 并传入相同 Key 会返回相同的结果。
}
```

context.Context 最大的作用是：在 goroutine 构成的树形结构中对信号进行同步，以减少计算资源的浪费。

多个 goroutine 同时订阅 ctx.Done 管道中的消息，一旦接受到取消信号就立刻通知当前正在执行的工作

context.Background 和 context.TODO 都是通过 new(emptyCtx) 语句初始化的，指向私有结构体 context.emptyCtx 的指针。
backgounrd 和 TODO 互为别名，只是在使用和语义上稍有不同
- context.Background 是上下文的默认值，所有的上下文从它衍生出来
- context.TODO 应该尽在不确定该使用哪种上下文时使用。


context.withCancel 函数能够从 context.Context 中衍生出一个新的子上下文，并返回用于取消该上下文的函数。一旦执行返回的取消函数，当前上下文和它的子上下文都会被取消（所有的goroutine 都会同步收到这一信号）
