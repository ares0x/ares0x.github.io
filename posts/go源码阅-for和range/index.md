# Go源码阅 For和range


for 循环两种形式：三段式循环和 range
```go
func do() {
    slice := []int{1,2,3}
    // 三段式
    for i := 0; i < len(slice); i++ {
        // do something
    }
    // range
    for idx, val := range slice {
        // do something
    }
}
```
range 可以帮助我们快速遍历数组，切片，哈希表以及 Channel 等集合类型。
for 循环可以让代码中的数据和逻辑分离，同一份代码能够多次复用相同的处理逻辑。

在汇编语言中，无论是近点的 for 循环还是 for-range 循环都会使用 JMP 等命令跳回循环体的开始位置复用代码。（for-range最终会被转换成普通的for循环格式）

## 现象

### 没有循环永动机
```go
func main() {
    arr := []int{1,2,3}
    for _, v := range arr {
        arr = arr.append(arr,i)
    }
    fmt.Println(arr) // 会输出 1 2 3 1 2 3
}
```

### 指针
```go
func main() {
	arr := []int{1, 2, 3}
	newArr := []*int{}
	for _, v := range arr {
		newArr = append(newArr, &v) // 用 &arr[i] 替代 &v
	}
	for _, v := range newArr {
		fmt.Println(*v) // 会输出 3 3 3
	}
}
```

### 遍历清空数组
```go
func main() {
    arr := []int{1,2,3}
    for i, _ := range arr {
       arr[i] = 0 
    }
}
```
遍历清空耗费性能，最快的方式是直接清空这片内存中的内容。 上述方法最后回调用 runtime.memclrNoHeapPointers 清空切片中的数据。

### 随机遍历

Go 在使用 range 遍历哈希表示，每次输出的内容顺序不一定完全一致。

```go
func main() {
    hash := map[string]int{
        "1":1,
        "2":2,
        "3":3,
    }

    for k, v := range hash {
        println(k,v)
    }
}

```

Go 在运行时为哈希表的遍历引入了不确定性。

### 经典循环
经典循环是类似 for i := 0; i < 10; i++ 的格式
```go
for Ninit; Left; Right {
    NBody
}
```
经典循环在 Go 编译器看来是一个 OFOR 类型的节点，节点由以下四个部分组成：
- 1. 初始化循环的 Ninit ;(for i := 0)
- 2. 循环的继续条件 Left ; (i < 5)
- 3. 循环体结束时执行的 Right ;(i++)
- 3. 循环体 NBody；

### 范围循环
范围循环是 for-range 格式。编译器会在编译期间将所有的 for-range 循环变成经典循环。即，将 ORANGE 类型节点转换为 OFOR 节点（gc.walkrange）

#### 数组和切片
range 的三种格式：
- for range arr : 只遍历数组和切片，不关心索引和数据
- for i := range arr : 遍历数组和切片，只关心索引的情况
- for i, v := range arr : 遍历数据和切片，关心索引和数据

对于所有的 range 循环，Go 都会在编译期间将原切片或者数组赋值给一个新变量 ha，在赋值过程中发生了拷贝，在编译期间预先通过 len 获取了原切片或者数组的长度，所以在循环中追加新的元素不会改变循环执行的次数。

同时需要遍历索引和元素的 range，Go 会额外创建一个新的 v2 变量用于存储切片中的元素，循环中使用 v2 会在每一次迭代被重新赋值（覆盖）。
```
ha := a
hv1 := 0
hn := len(ha)
v1 := hv1
v2 := nil
for ; hv1 < hn; hv1++ {
    tmp := ha[hv1]
    v1, v2 = hv1, tmp
    ...
}
```

```go
func main() {
	arr := []int{1,2,3}
	for i, v := range arr {
		fmt.Printf("index:%d,  addr:%p\n",i, &v)
	}
}
// index:0,  addr:0x140000200d0
// index:1,  addr:0x140000200d0
// index:2,  addr:0x140000200d0
```
在循环中获取返回变量的地址都完全相同。所以，如果想要访问数组中元素所在的地址，不应该直接获取range 返回的变量地址 &v2，应该获取&a[idx]

#### 哈希表
Go 在设计哈希表的遍历时不想让使用者依赖固定的遍历顺序，引入了随机数（faskrangd()）来保证遍历的随机性。
遍历哈希时会使用 runtime.mapiternext
- 在待遍历的桶为空时，选择需要遍历的新桶
- 在不存在待遍历的桶时，返回 (nil,nil) 键值对并终止遍历

哈希的遍历顺序：
- 首先选出一个正常桶开始遍历
- 随后遍历溢出桶
- 最后依次按照索引顺序比你哈希表中的其他桶，直到所有的桶都被遍历完成

#### 字符串
字符串遍历与数组，切片和哈希表类似，在遍历时会获取字符串索引对应的字节并将字节转换成 rune。在遍历字符串时拿到的值都是 rune 类型。

#### 通道
for v := range ch {}
循环会使用 <- ch 从管道中取出等待处理的值，该操作会调用 runtime.chanrecv2 并阻塞当前协程。runttime.chanrecv2 返回时会根据布尔值 hb 判断当前值是否存在
- 如果不存在当前值，意味着 channel 已经被关闭
- 如果存在当前值，会赋值，重新陷入阻塞等待新数据





