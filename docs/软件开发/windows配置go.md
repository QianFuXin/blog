---
tags: ["windows", "go"]
---

# 安装

[官网](https://go.dev/dl/)

# 环境变量

已默认配置好

- 默认的gopath是用户账户下的go文件夹

- 默认的goroot是安装的位置

# goland开启demo

- 新建go项目
- 安装包

  `go get -u github.com/gin-gonic/gin`

- 新建main.go

```go
package main

import (
"github.com/gin-gonic/gin"
)

func main() {
r := gin.Default()
r.GET("/ping", func(c *gin.Context) {
c.JSON(200, gin.H{
"message": "hello world",
})
})
r.Run() //listen and serve on 0.0.0.0:8080
}
```

- 运行

```shell
#运行
go run main.go
# 打包（在gopath的bin目录中，比如C:\Users\username\go\bin）
go install
```
