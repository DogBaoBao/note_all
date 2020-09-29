# 一拳搞定 Gin \| 日志

go 语言不像 java 有 slf4j，我们在开发项目时通常都会选择使用专业的日志库来记录项目中的日志，go语言常用的日志库有`zap`、`logrus`等。



## 现状

默认启动 Gin，我们会在控制台看到：

```bash
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /debug/pprof/             --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/cmdline      --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/profile      --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] POST   /debug/pprof/symbol       --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/symbol       --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/trace        --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/allocs       --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/block        --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/goroutine    --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/heap         --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/mutex        --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /debug/pprof/threadcreate --> github.com/gin-contrib/pprof.pprofHandler.func1 (3 handlers)
[GIN-debug] GET    /api/v1/ping              --> gdcloud/pkg/router.Ping (3 handlers)
...
[GIN-debug] Listening and serving HTTP on :8899
```

会显示所有的接口和启动的端口等信息。

控制台输出日志只能应付我们开发环境的使用，如果项目要上线必须有日志文件（毕竟在 k8s 里面容器是无状态运行，很容易结束自己的生命周期）



