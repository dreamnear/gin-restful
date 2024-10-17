# gin-restful
gin-restful 是为了方便使用 gin 开发 restful api 而创建的库。  
可以通过向 Api 注册 Resource 的形式开发 restful api。  
由于英语不好，文档和注释用韩文编写。

## 概要
为了更方便地使用 gin 开发 restful api，  
可以通过向 Api 注册 Resource 的形式进行开发。  
Resource 可以是任何结构体，当 url 被调用时，  
会调用与 http method 同名的方法。  
只需注册 Resource，自动分析 Handler Method 的参数，  
生成 url 并注册到 gin。

## 示例
```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/hwangseonu/gin-restful"
	"net/http"
)

//用于实现 restful api 的 SampleResource 结构体。
//嵌入 gin-restful 的 Resource 结构体指针。
//可以为每个 Handler 应用不同的 Middleware。
type SampleResource struct {
	*gin_restful.Resource
}

//用于测试 Json Body 的结构体。
type Data struct {
	Name string `json:"name" validate:"required,notblank"`
}

//当 GET 请求到 SampleResource 的 Url 时执行的 Handler。
//返回 gin.H(json) 和状态码。
//将 path variable 的 name 放入 json 返回。
func (r SampleResource) Get(name string) (gin.H, int) {
	return gin.H{
		"name": name,
	}, http.StatusOK
}

//当 POST 请求到 SampleResource 的 Url 时执行的 Handler。
//将 json body 接收为 Data 结构体。
//返回 gin.H 和状态码。
//将接收到的 payload 原样返回。
func (r SampleResource) Post(c *gin.Context, json Data) (Data, int) {
	return json, 200
}

//用于测试 Middleware 的 Sample Middleware。
//在控制台输出 "Hello, World"。
func SampleMiddleware(c *gin.Context) {
	println("Hello, World")
}

//在 "/" 地址创建 Api 实例。
//创建 SampleResource 实例并注册到 Api "/samples" 地址。
//为 SampleResource GET handler 注册 SampleMiddleware。
//在 5000 端口运行 gin 服务器。
func main() {
	r := gin.Default()
	v1 := gin_restful.NewApi(r, "/")
	res := SampleResource{gin_restful.InitResource()}
	res.AddMiddleware(SampleMiddleware, http.MethodGet)
	v1.AddResource(res, "/samples")
	_ = r.Run(":5000")
}