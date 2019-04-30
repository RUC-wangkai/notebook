# 关于gin的思考

在学习使用 _gin_ 时，偶尔搜到了这一篇文章，文中提到了 golang 的 _net/http_ 库已经把http的大部分功能都实现了，而且做的都还不错，为什么还会出现那么多的go框架，这些go框架是对 _net/http_ 哪部分进行的改进。
附上文章的链接 [gin源码阅读之一 – net/http的大概流程](http://www.haohongfan.com/2019/02/gin%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8B%E4%B8%80-net/http%E7%9A%84%E5%A4%A7%E6%A6%82%E6%B5%81%E7%A8%8B/)

实际上 _gin_ 实现了 **ServeHttp** 函数继承了 **Handle** 得以将数据流向 _gin_ 中。

首先我们来看看 _net/http_ 是怎样开启一个服务的，直接上代码


```
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello, Go HTTP Server"))
    })
    http.HandleFunc("/ping", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("ping success"))
    })
    if err := http.ListenAndServe(":12345", nil); err != nil {
        fmt.Println(err)
    }
}
```

这样我们就可以访问 _http://localhost:12345/_ 以及 _http://localhost:12345/ping_ 但这种路由规则太过于简单，而且处理/ping，其他路由都会匹配到/，上面的用法只能用于一些简单的使用。有一种比较好的做法是自己实现一个 **http.Handle**,而实现**http.Handle**也比较简单，只需要实现 **ServeHTTP**函数就可以了，如下：

```
type MyHandler struct{} // 实现 http.Handler 接口的 ServeHTTP 方法

func (mh MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if r.URL.Path == "/abc" {
        w.Write([]byte("abc"))
        return
    }
    if r.URL.Path == "/xyz" {
        w.Write([]byte("xyz"))
        return
    }
    w.Write([]byte("index"))
    // 这里你可以写自己的路由匹配规则
}

func main() {
    if err := http.ListenAndServe(":12345", MyHandle{}); err != nil {
        fmt.Println("start http server fail:", err)
    }
}
```
这样就可以在**ServeHTTP**中处理复杂的路由匹配，而且写法也优雅了不少。
而**HTTP**过程的操作主要是针对客户端发来的请求数据在 ***http.Request**，和返回给客户端的 **http.ResponseWriter** 两部分，我们可以在**ServeHTTP**中对这两部分做更多的文章。
这也是很多go框架所做的工作。

我们来简单看看 _gin_ 是怎么做的，首先使用 _gin_ 起一个服务很简单：
```
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080
}
```
我们到**r.Run()**里面看看，

```
// Run attaches the router to a http.Server and starts listening and serving HTTP requests.
// It is a shortcut for http.ListenAndServe(addr, router)
// Note: this method will block the calling goroutine indefinitely unless an error happens.
func (engine *Engine) Run(addr ...string) (err error) {
	defer func() { debugPrintError(err) }()

	address := resolveAddress(addr)
	debugPrint("Listening and serving HTTP on %s\n", address)
	err = http.ListenAndServe(address, engine)
	return
}
```
我们可以看到 _gin_ 调用了**http.ListenAndServe**，而且他自己实现了**Handle**接口，也就是Engine里实现了**ServeHTTP**,看这里：
```
// ServeHTTP conforms to the http.Handler interface.
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	c := engine.pool.Get().(*Context)
	c.writermem.reset(w)
	c.Request = req
	c.reset()

	engine.handleHTTPRequest(c)

	engine.pool.Put(c)
}
```

## 更多的参考资料
[gin源码阅读之一 – net/http的大概流程](http://www.haohongfan.com/2019/02/gin%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8B%E4%B8%80-net/http%E7%9A%84%E5%A4%A7%E6%A6%82%E6%B5%81%E7%A8%8B/)

[GO 开发 HTTP](http://fuxiaohei.me/2016/9/20/go-and-http-server.html)