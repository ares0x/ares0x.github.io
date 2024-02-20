# gin 框架 - 路由


## 简介
路由是指当客户端请求达到后端服务器后，根据请求的 HTTP 请求方法和 URL，找到对应的处理函数进行处理的过程。路由由 HTTP 请求方法，URL 和请求处理器组成。HTTP 请求方法指 HTTP 协议中定义的请求类型，常见的包括 GET、POST、PUT、DELETE 等。URL 指请求的资源在服务器上的路径，通常是由一个或多个单词或数字组成，在一定规则约束下可带上特定字符，例如斜杠、短横线等。请求处理器是指针对某一请求的响应处理函数。

## 基本使用

### 基本路由
```go
package main

import "github.com/gin-gonic/gin"

func main() {
    g := gin.Default()
    g.Handle("GET", "/hello", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Hello World!",
        })
    })
    g.Run(":8080")
}
```
使用 g.Handle() 方法可以定义针对特定请求方法的路由。Handle() 方法的第一个参数是 HTTP 请求方法，第二个参数是 URL 路径，第三个参数是请求处理器函数。在上面的例子中，我们定义了一个针对 HTTP GET 方法、URL 路径为 /hello 的路由。当用户请求该路径时，路由器会调用”请求处理器函数”返回相应的 JSON 信息。

除了上述方法外，我们还可以使用GET()、POST()、DELETE()等快捷方法定义路由。比如：
```go
router.GET("/hello", func(c *gin.Context) {
    c.JSON(200, gin.H{
        "message": "Hello World!",
    })
})
```

### 路由组
路由组是将多个路由集成为一组的一种方法，主要用于拆分应用的路由实现。这样可以使路由变得更加清晰可读，并可以把相关路由分组到同一个地方进行管理。在 gin 框架中，使用 Group() 方法来创建路由组。下面是一个创建路由组的例子：
```go
package main

import "github.com/gin-gonic/gin"

func main() {
    g := gin.Default()
    apiGroup := g.Group("/api")
    {
        apiGroup.GET("/hello", func(c *gin.Context) {
            c.JSON(200, gin.H{
                "message": "Hello World!",
            })
        })
    }
    g.Run(":8080")
}
```
在这个例子中，我们创建了一个名为 apiGroup 的路由组，它包含了所有以 /api 开头的路径。那么，请求 /api/hello 路径时，仍然会调用定义的处理函数返回所需的 JSON 信息。这种方法可以把多个路由按照一定规律分组起来，便于管理和增加可读性。

### 中间件
中间件是在执行路由处理函数之前或之后处理请求和响应的函数。中间件可以被用来写日志和验证等较为通用的操作，可以帮助程序员更好地维护Web应用。在 gin 框架中，使用 Use() 方法注册中间件。
```go
package main

import "github.com/gin-gonic/gin"

func main() {
    g := gin.Default()
    g.Use(func(c *gin.Context) {
        // 中间件处理逻辑
        c.Next()
    })
    g.GET("/hello", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Hello World!",
        })
    })
    g.Run(":8080")
}
```
使用中间件可以实现一些预处理，例如，如果用户想要访问特定路由，需要先通过账号密码验证。我们可以编写一个最简单的验证中间件：
```go
func auth() gin.HandlerFunc {
    return func(c *gin.Context) {
        if user, pass, ok := c.Request.BasicAuth(); !ok || user != "admin" || pass != "123456" {
            c.Header("WWW-Authenticate", `Basic realm="My Realm"`)
            c.AbortWithStatus(401)
        } else {
            c.Next()
        }
    }
}
```
调用 auth() 方法创建一个基础中间件，实现验证用户名密码的功能。当我们访问 /hello 获取数据时，会先调用中间件检测权限，如果权限验证失败，返回 401 状态码。

### 完整实例
下面是一个完整的 Gin 路由示例代码：
```go
package main

import "github.com/gin-gonic/gin"

func main() {
    g := gin.Default()
    g.GET("/hello", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Hello World!",
        })
    })
    g.GET("/user/:name", func(c *gin.Context) {
        name := c.Param("name")
        c.JSON(200, gin.H{
            "message": "Hello," + name,
        })
    })
    v1 := g.Group("/api/v1", Auth())
    {
        v1.GET("/users", func(c *gin.Context) {
            c.JSON(200, gin.H{
                "message": "list all users",
            })
        })
        v1.GET("/users/:id", func(c *gin.Context) {
            id := c.Param("id")
            c.JSON(200, gin.H{
                "message": "get user " + id,
            })
        })
    }
    g.Run(":8080")
}

func Auth() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 鉴权逻辑
        c.Next()
    }
}

```
在上述示例中，我们定义了一个基本路由和一个带参数的路由，还使用了路由组，并用一个基础鉴权中间件包装它们。首先，在 http://localhost:8080/hello 路径下，返回了 ”Hello World!” JSON 信息。在 /user/:name 路径下，把传进来的 name 连接在字符串前面返回。在 /api/v1 路径下的路由被授权。这是一个简洁明了的例子，可以用来学习如何使用Gin框架。

## gin 路由原理

```go
type Engine struct{
        ...
    trees            methodTrees
        ...
}

type methodTree struct {
	method string
	root   *node
}

// methodTree 中的每个节点
type node struct {
	path      string
	indices   string // 索引
	wildChild bool
	nType     nodeType // 节点类型：static，root，param，catchAll
	priority  uint32 // 优先级
	children  []*node // 子节点
	handlers  HandlersChain // HandlersChain 定义了一个 HandlerFunc 切片
	fullPath  string // 完整路径
}

func New() *Engine {
     engine := &Engine{
        trees:                  make(methodTrees, 0, 9),
     }
}
```
在 Engine 结构中有一个名为 trees 的 methodTrees，它是 gin 路由结构中的方法树。在调用 New() 方法创建一个 Engine 时，会构建一个 cap 为 9 的 methodTrees slice。gin 会为 POST，GET 等请求分别创建一棵方法树。
每当新增一个 handler，会调用 addRoute 方法
```go
func (engine *Engine) addRoute(method, path string, handlers HandlersChain) {
	assert1(path[0] == '/', "path must begin with '/'")
	assert1(method != "", "HTTP method can not be empty")
	assert1(len(handlers) > 0, "there must be at least one handler")

	debugPrintRoute(method, path, handlers)

	root := engine.trees.get(method)
	if root == nil {
		root = new(node)
		root.fullPath = "/"
		engine.trees = append(engine.trees, methodTree{method: method, root: root})
	}
    // 将其加入到路由树中
	root.addRoute(path, handlers)

	// Update maxParams
	if paramsCount := countParams(path); paramsCount > engine.maxParams {
		engine.maxParams = paramsCount
	}

	if sectionsCount := countSections(path); sectionsCount > engine.maxSections {
		engine.maxSections = sectionsCount
	}
}
```
