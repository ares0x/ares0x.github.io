# Go源码阅 Select


在操作系统中 select 是系统调用，经常使用 select，poll 和 epoll 等函数构建 I/O 多路复用模型来提升程序性能。

Go 中的 select 能够让 goroutine 同时等待多个 channel 读写，在多个文件或者 channel 状态改变之前，select 会一直阻塞当前线程或 goroutine

格式：
select 的格式与 swithc 类似，有 case 操作，但是 case 中的表达式必须是 channel
```go
func fibonacci(c, quit chan int) {
    x, y := 0, 1
    for {
        select {
            case c <- x:
                x, y = y, x+y
            case <- quit:
                fmt.Println("quit")
                return 
        }
    }
}
```

## 现象
- select 能在 channel 上进行非阻塞的收发
- select 在遇到多个 channel 同时响应时会随机执行一种情况

### 非阻塞收发
通常情况下，select 会阻塞当前 goroutine 并等待多个 channel 中的一个达到可以收发的状态。但是如果 select 中包含 default 语句，会有以下两种情况：
- 当存在可以收发的 channel 时，直接处理该 channel 对应的 case
- 当不存在可以收发的车channel 时，执行 default 中的语句

在很多场景下我们不希望 channel 阻塞当前 goroutine，只是想看看 channel 的可读或者可写状态

```go
errCh := make(chan error, len(tasks))
wg := sync.WaitGroup()
wg.Add(len(tasks))

for i := rang tasks{
    go func() {
        defer wg.Down()
        if err := tasks[i].Run(); err != nil {
            errCh <- err
        }
    }
}
wg.Wait()

select {
case err := <- errCh:
    return err
default:
    return nil
}
// 此方法不关心几个任务执行失败，只关心是否有错误
```

### 随机执行
select 遇到多个 <- ch 同时满足可读或者可写条件时会随机选择一个 case 执行

## 结构
```go
type scase struct {
    c    *hchan
    elem unsafe.Pointer
}
```

## 原理

空 select：空的 select 会直接阻塞当前 goroutine， 导致 goroutine 进入无法被唤醒的永久休眠状态
单一阻塞：如果当前 select 中只包含一个 case，编译器会将其改写为 if 语句
```go
select {
case v, ok <-ch: // case ch <- v
    ...    
}

// 改写后
if ch == nil { // 如果ch 为空，直接休眠
    block()
}
v, ok := <-ch // case ch <- v
```
非阻塞操作：select 中仅包含两个 case，其实一个是 default 时，go 编译器认为这是一次非阻塞的收发操作。



