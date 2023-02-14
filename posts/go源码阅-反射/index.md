# Go源码阅 反射


reflect 实现了运行时的反射能力，能让程序操作不同类型的对象。

reflect.TypeOf() 获取类型信息
reflect.ValueOf() 获取数据的运行时表示

反射包中的所有方法基本都是围绕着 reflect.Type 和 reflect.Value 两个类型设计的。

## 三大法则

Go 中反射的三大法则：
- 1. 从 interface{} 变量可以反射出反射对象；
- 2. 从反射对象可以获取 interface{} 变量；
- 3. 要修改反射对象，其值必须可设置

### 第一法则
可以将 Go 中的 interface{} 变量转换成反射对象。

```go
func TypeOf(i interface{}) Type {
	eface := *(*emptyInterface)(unsafe.Pointer(&i))
	return toType(eface.typ)
}

func ValueOf(i interface{}) Value {
	if i == nil {
		return Value{}
	}
	escapes(i)

	return unpackEface(i)
}
```
当我们调用
```go
func do() {
    var i int
    reflect.TypeOf(i) // 发生了类型转换
}
```
TypeOf 和 ValueOf 接收的参数类型是 interface 类型。Go 是值传递，变量在函数调用时进行类型转换。

### 第二法则
可以从反射对象获取 interface{} 变量。使用 reflect.Value.Interface() 方法

```go
v := reflect.ValueOf(1)
v.Interface().(int)
```
### 第三法则
如果需要更新一个 reflect.Vlaue，它持有的值一定是可以背更新的。

```go
func main() {
	i := 1
	v := reflect.ValueOf(&i) // 如果是 v := reflect.ValueOf(i) 不可更改
	v.Elem().SetInt(10)
	fmt.Println(i)
}
```

## 类型和值
Go 中 interface{} 类型在内部是通过 emptyInterface 结构体表示的
```go
type emptyInterface struct {
    typ *rtype // 用于表示变量的类型
    word unsafe.Pointer // 指向内部封装的数据
}
```
reflect.TypeOf 会将传入的变量转换为 emptyInterface 类型，并获取其中存储的类型信息

当我们要将一个变量转换成反射对象时，Go 语言会在编译期间完成类型转换，将变量的类型和值转换成 interface{} 并等待运行期间使用 reflect 包获取接口中存放的信息。


