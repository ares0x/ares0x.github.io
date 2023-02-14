# Go源码阅 接口


## 概述

**Go 语言的接口不是任意类型**

接口的本质是引入一个新的中间层，调用方可以通过接口与具体实现的分离，解除上下游的耦合。
定义良好的接口能够隔离底层的实现，让我们将重点放在当前的代码片段中。

如，数据查询操作
```go
type UserInfo struct {}

type Query interface {
	QueryUserInfo(userName string) (UserInfo, error)
}

func GetQuery(userName string) Query {
	if userName != "" {
		return &MysqlQuery{}
	}
	return &RedisQuery{}
}

type MysqlQuery struct{}

func NewMysqlQuery() Query {
	return &MysqlQuery{}
}

func (m *MysqlQuery) QueryUserInfo(userName string) (UserInfo, error) {
	var userInfo = UserInfo{}
	return userInfo, nil
}

type RedisQuery struct{}

func NewRedisQuery() Query {
	return &RedisQuery{}
}

func (r *RedisQuery) QueryUserInfo(userName string) (UserInfo, error) {
	var userInfo = UserInfo{}
	return userInfo, nil
}

func main() {
	var userName = "ares"
	GetQuery(userName).QueryUserInfo(userName)
	NewMysqlQuery().QueryUserInfo(userName)
	NewRedisQuery().QueryUserInfo(userName)
}

```

### 隐式接口

最简单的 error interface
```go
type error interface {
    Error() string
}
```
如果要实现 error 接口，只需要实现 Error() string 方法即可
```go
type RPCError struct {
    Code int64
    Message string
}

func NewRPCError(code int64, msg string) error{
    return &RPCError{
        Code: code, 
        Message:msg,
    }
}

func (e *RPCError) Error() string {
    return fmt.Sprintf("%s, code=%d", e.Message, e.Code)
}

func main() {
    var rpcError error = NewRPCError(500, "internal error")
    err := AsError(rpcErr)
    fmt.Println(err)
}

func AsErr(err error) error {
    return err
}

```
所以，在 Go 中的接口都是隐式的，只要实现了接口中方法，就实现了这个 interface 

### 类型
接口（interface）时 Go 中的一种类型，能够出现在变量的定义，函数的入参和返回值中并对它们进行约束。Go 中的 interface 大概可分为两种，一种是带有一组方法的，一种是不带任何方法的。
第一种用 runtime.iface 表示，第二种用 runtime.eface 表示。

在 Go 中 interface 类型不是**任意类型**。

```go
package main

func main() {
    type Test struct{}
    v := Test{}
    print(v)
}

// print 只接收 interface{} 类型的值，调用 print 时对 v 进行类型转换，将 Test 类型转换为 interface{} 类型。
func print(v interface{}) {
    println(v)
}
```

### 指针和接口
对于一个结构体，在实现接口时可以选择接受者的类型：结构体或者结构体指针。
如果结构体实现接口，则使用结构体初始化变量和使用结构体指针初始化变量都可以通过；
如果结构体指针实现接口，使用结构体初始化变量不通过，使用结构体指针初始化变量可以通过。
使用结构体定义结构，结构体指针初始化变量时，指针能够隐式地获取指向的结构体。

总结：
使用指针实现接口时，只有指针类型的变量才会实现该接口；使用结构体实现接口时，指针和结构体类型都会实现该接口。

### nil 和 non-nil

```go
package main

import(
    "fmt"
)

type Student struct{}

func NilOrNot(v interface{}) bool {
    return v == nil
}

// s 初始化后是指针的零值 nil，在作为参数传递到 NilOrNot 后，做了隐式类型转换
func main() {
    var s *Student
    fmt.Println(s == nil) // true
    fmt.Println(NilOrNot(s)) // false
}
```
上述中，除了向方法传入参数外，变量的赋值也会触发隐式类型转换。*Student 转换为 interface 类型，转换后的变量不仅包含转换前的变量，还包含变量的类型信息 Student,所以转换后与 nil 不想等。

## 数据结构

不包含方法的 interface，结构中只包含指向底层数据和类型的指针。

```go
type eface struct {
    _type *_type
    data unsafe.Pointer
}
```
runtime._type 是 Go 语言类型的运行时表示。

```go
type _type struct {
	size       uintptr // 存储了类型占用的内存空间，为内存空间的分配提供信息
	ptrdata    uintptr
	hash       uint32 //该字段可以帮助我们快速确定类型是否相等
	tflag      tflag
	align      uint8
	fieldAlign uint8
	kind       uint8
	equal      func(unsafe.Pointer, unsafe.Pointer) bool // 用于判断当前类型的多个对象是否相等
	gcdata     *byte
	str        nameOff
	ptrToThis  typeOff
}
```

包含方法的 interface
```go
type iface struct {
    tab *itab
    data unsafe.Pointer
}
```

```go
type itab struct { // 32 字节
	inter *interfacetype
	_type *_type
	hash  uint32 // 对 _type.hash 的拷贝，将 interface 转为具体类型时，可以使用该字段快速判断目标类型和具体类型是否一致
	_     [4]byte
	fun   [1]uintptr // 一个动态大小的数组，是一个用于动态派发的虚函数表，存储了一组函数指针
}
```

## 类型转换

Go 语言的编译器会在编译期间将一些需要**动态派发**的方法调用改写成目标方法的直接调用，以减少性能的额外消耗。

```go
package main

type Duck interface {
	Quack()
}

type Cat struct {
	Name string
}

//go:noinline
func (c Cat) Quack() {
	println(c.Name + " meow")
}

func main() {
	var c Duck = Cat{Name: "draven"}
	c.Quack()
}
```
步骤：（指针与结构体类似）
- 初始化 Cat 结构体；
- 完成从 Cat 到 Duck 接口的类型转换
- 调用接口的 Quack 方法。


### 类型断言

### 动态派发
动态派发是在运行期间选择具体多态操作执行的过程，是面向对象语言中的常见特性。
Go 不是严格意义上的面向对象语言，但是接口的引入为它带来了动态派发这一特性。调用接口类型的方法时，如果编译期间不能确定接口的类型，Go 语言会在运行期间决定具体调用该方法的哪个实现。


