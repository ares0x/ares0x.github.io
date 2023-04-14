# Go 源码阅读 - 环境配置


- 该系列是我在搭配 [Go 语言设计与实现](https://draveness.me/golang/) 阅读 Go 源码时的笔记，侧重于记录，文笔可能比较潦草。

## 环境说明
* 机器配置：Apple M1 Pro
* Go 版本： go1.16.5
* 编辑器：Goland, VIM, VS Code
* GOPATH：/Users/ares/workspace/go

## 建立目录
在 $GOPATH 下建立一个目录，用于放置下载的源码以及对应的测试代码。我建立了一个名为 version 的目录，在该目录下存放 go1.16.5 版本源码以及测试项目，如图所示
![version 目录](/images/tree.png)

## 下载go源码
在 https://go.dev/dl/ 中下载与本地 go version 版本一致的源码，注意（Mac M系列芯片的要选择 Arch 为ARM64 的）

```shell
tar -zxvf go1.16.5.darwin-arm64.tar.gz -C $GOPATH/version
```

## 配置 Goland
使用 Goland 打开下载的源码，配置 GOROOT 为本地 go env 下的 go 版本（编译过程中要使用其中的一些工具）
建立一个对应 go version 的 practice 项目，将项目的 GOROOT 配置为源码所在路径，该项目用于测试修改后的源码
![goland 配置](/images/goland-config.png)

## 编译 Go 源码

```shell
cd $GOPATH/version/go1.16.5/src (GOPATH 替换为本地 GOPATH所在路径)
./make.bash
```

## 执行测试

可以通过修改源码中的任意位置的逻辑，来测试是否生效
```go
func (e *errorString) Error() string {
    return "ares0x" + e.s
}

```
我将 src/errors/errors.go 中的 Error() 方法修改为上图所示，执行编译，并建立了 err.go 的文件

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := errors.New("hello")
	fmt.Println(err)
}
```
代码输出结果为：

```shell
ares0xhello
```
由此可得，源代码修改，编译成功

