# Go源码阅 同步原语和锁


锁是一种并发编程中的同步原语，它能保证多个 goroutine 在访问统一内存时不会出现竞争条件等问题。

常见的同步原语：
sync.Mutex
sync.RWMutex
sync.WaitGroup
sync.Once
sync.Cond

## Mutex
```go
// 互斥锁结构
type Mutex struct {
	state int32 // 锁的状态
	sema  uint32 // 用于控制锁状态的信号量
}
```
默认情况下，互斥锁的所有状态为都是 0，int32 中不同位表示不同的状态：
- mutexLocked - 表示互斥锁的锁定状态
- mutexWoken - 表示从正常状态被唤醒
- mutexStarving - 表示当前互斥锁进入饥饿状态
- waitersCount - 表示当前互斥锁上等待的 goroutine 个数

## 正常模式和饥饿模式
* Mutex can be in 2 modes of operations: normal and starvation.
互斥锁有两种模式：正常模式和饥饿模式

正常模式下，锁的等待者按照**先进先出 FIFO**的顺序获取锁。但是刚被唤起的 goroutine 与新创建的 goroutine 竞争时，大概率会获取不到锁。为了减少这种情况的出现，一旦 goroutine 超过 1ms 没有获取到锁，它就会从当前互斥锁切换为饥饿模式，防止部分 goroutine 被饿死

饥饿模式在 go1.9中引入的优化，目的是保证互斥锁的公平性。
在饥饿模式下，互斥锁会直接交给等待队列最前面的 goroutine。新的 goroutine 在该状态下不能获取锁，也不会进入自旋状态，只会在队列的末尾等待。
如果一个 goroutine 获得了互斥锁并且它在队列的末尾或者它的等待时间少于 1ms，当前锁的切换为正常模式。

正常模式下的互斥锁能提供更好的性能。饥饿模式能避免 goroutine 由于陷入等待无法获取锁而造成的延迟。

## 加锁和解锁

sync.Mutex.Lock
sync.Mutex.Unlock

```go
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	// Slow path (outlined so that the fast path can be inlined)
	m.lockSlow()
}
```
如果互斥锁的状态是 0 时，将 mutexLocked 位改成1。如果不是 0，会调用 lockSlow 尝试通过自旋（Spinning）等方式等待锁的释放。

大的 for 循环
步骤：
- 判断当前 goroutine 是否进入自旋
- 通过自旋等待互斥锁的释放
- 计算互斥锁的最新状态
- 更新互斥锁的状态并获取锁

自旋是一种多线程同步机制，当前线程进入自旋的过程中会一直保持 CPU 的占用，持续检查某个条件是否为真。在多核 CPU 上，自旋可以避免 goroutine 的切换，使用恰当会对性能带来很大增益。
goroutine 进入字段的条件很苛刻：
- 互斥锁只有在普通模式下才能进入自旋
- sync.runtime.canSpin 需要返回 true
  - 运行在多 CPU 的机器上
  - 当前 goroutine 为了获取该锁进入自旋的次数小于四次
  - 当前机器上至少存在一个正在运行的处理器 P 并且处理的运行队列为空

当前 goroutine 进入自旋会调用 sync_runtime_doSpin 和 runtime.procyield 并执行 30 次 PAUSE 执行

互斥锁回根据上下文计算当前互斥锁的最新状态。
计算互斥锁状态后，使用 CSA 函数 更新状态

解锁
Unlock 调用 atomic.AddInt32 来快速解锁
- 如果返回的新状态是0，当前 goroutine 解锁成功
- 如果不是0，调用 unlockSlow 开始慢解锁

慢解锁回检验锁状态的合法性，如果当前互斥锁已经背解锁过会直接抛异常，中止当前程序。


