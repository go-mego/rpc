# JSON RPC [![GoDoc](https://godoc.org/github.com/go-mego/jsonrpc?status.svg)](https://godoc.org/github.com/go-mego/jsonrpc)

雖然 Mego 本身是基於 HTTP 的 RESTful 引擎，但同時也可以使用 JSON RPC 作為處理、回應基礎。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)
	* [方法定義](#方法定義)
    * [錯誤處理](#錯誤處理)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/jsonrpc
```

# 使用方式

將 `jsonrpc.New` 傳入 Mego 引擎的 `Use` 來作為全域中介軟體即可在所有路由中使用 JSON RPC 的相關功能。

```go
package main

import (
	"github.com/go-mego/jsonrpc"
	"github.com/go-mego/mego"
)

func main() {
	m := mego.New()
	// 將 JSON RPC 作為全域中介軟體。
	m.Use(jsonrpc.New())
	m.Run()
}
```

JSON RPC 也能夠僅用於單一路由中，這讓你有單個進入點。

```go
func main() {
	m := mego.New()
	// 將 JSON RPC 用於單個路由中。
	m.POST("/rpc", jsonrpc.New(), func(r *jsonrpc.RPC) {
		// ...
	})
	m.Run()
}
```

## 方法定義

透過 `NewFunc` 建立新的方法，並在其中處理、回傳接收到的資料。

```go
func main() {
	m := mego.New()
	m.POST("/rpc", jsonrpc.New(), func(r *jsonrpc.RPC) {
		// 以 `NewFunc` 建立一個 `add` 方法，並接收兩個 `int` 形態的參數。
		r.NewFunc("add", func(a int, b int) (interface{}, error) {
			// ...
		})
	})
	m.Run()
}
```

方法的接收參數也可以是一個本地的建構體。

```go
// 初始化一個欲映射的本地建構體，用以接收 JSON RPC 方法的請求資料。
type User struct {
	Username string `json:"username"`
	Password string `json:"password"`
}

func main() {
	m := mego.New()
	m.POST("/rpc", jsonrpc.New(), func(r *jsonrpc.RPC) {
		// 在 `NewFunc` 中用上指定的建構體表示資料欲映射到該建構體供本地使用。
		r.NewFunc("login", func(user *User) (interface{}, error) {
			// user.Username, user.Password ...
		})
	})
	m.Run()
}
```

## 錯誤處理

方法的最後能夠回傳 `errors.New` 來表示一個最基本的錯誤發生了。

```go
func main() {
	m := mego.New()
	m.POST("/rpc", jsonrpc.New(), func(r *jsonrpc.RPC) {
		r.NewFunc("add", func() (interface{}, error) {
			// 透過 `errors.New` 建立最基本的 JSON RPC `-32603` 錯誤。
			return nil, errors.New("這是一個最基本的通用錯誤。")
		})
	})
	m.Run()
}
```

如果要表明錯誤的資料和自訂狀態碼，則可以考慮使用 `r.Error` 來產生一個更詳細的錯誤回應。

```go
func main() {
	m := mego.New()
	m.POST("/rpc", jsonrpc.New(), func(r *jsonrpc.RPC) {
		r.NewFunc("add", func() (interface{}, error) {
			// 透過 `r.Error` 建立一個較詳細的自訂錯誤、並帶有錯誤資料。
			return nil, r.Error(1, "帳號密碼不符。", map[string]string{
				"mismatch_field": "username",
			})
		})
	})
	m.Run()
}
```