# はじめに

Go 言語のフレームワークである [Echo](https://echo.labstack.com/) を使ってバックエンドを実装する際の、ルーティング に関する備忘録です。
私はこうやったということを残すものであり、ベストプラクティスではありませんが、みなさまの参考情報となれば幸いです。


# 環境と各ツール

|                                                                | バージョン | 備考                                                 |
| -------------------------------------------------------------- | --------- | ---------------------------------------------------- |
| [Go](https://go.dev/)                                          | v1.22.1   | `go version` で確認                                  |
| [Echo](https://echo.labstack.com/)                             | v4.12.0   | サーバ起動時のログで確認                              |


# ルーティング
## ルーティングを管理する処理

API を実行するための URL と、その API が実行されたときに実際に行われる処理を定義します。
ここでは `routing.go` というファイルに `testapi` というパッケージを読み込ませて、`GET`, `POST`, `PUT`, `DELETE` の各処理へのルーティングを設定しています。

```go:routing.go
package router

import (
    // 振り分ける API
	"restapi/modules/testapi"

	"github.com/labstack/echo/v4"
)

/**
 * API のルーティングを管理する
 */
func Routing(e *echo.Echo) {
	e.GET("/testapi/all", testapi.All)       // 全件取得
	e.GET("/testapi/:id", testapi.Content)   // ID を指定して取得
	e.POST("/testapi", testapi.Register)     // 登録
	e.PUT("/testapi/:id", testapi.Update)    // ID を指定して更新
	e.DELETE("/testapi/:id", testapi.Delete) // ID を指定して削除
}
```

パッケージ `testapi` には

- All
- Content
- Register
- Update
- Delete

という処理( 関数 )を定義しています。( 各処理については後述の [API](#API) にコードを載せてあります )
これらの処理と対応する URL を指定したものが上記です。

## echo 起動時にルーティングを実行する

`routing.go` でルーティングを定義しましたが、それだけでは Echo 上でルーティングが機能してくれません。
機能してもらうには次のように Echo 起動時にルーティングを実行してやります。

```go:server.go
package main

import (
	// (..略..)

	"restapi/router" // ルーティングパッケージ

	"github.com/labstack/echo/v4"
)

function main() {
	e := echo.New()

	// (..略..)

	// ルーティングパッケージにある Routing を実行することでルーティングが機能する
	router.Routing(e)

	// (..略..)
}

```

# API 

ルーティングで指定している各処理は( 本記事で扱っている構成では )次のようになっています。

```go:testapi.go
package testapi

import (
	"log"
	"net/http"

	"github.com/labstack/echo/v4"
)

/**
 * 全件取得
 */
func All(c echo.Context) error {
	// 全件取得できたという体でレスポンスを返却する
	return c.String(http.StatusOK, "All Contents\n")
}

/**
 * 指定された ID に紐づくデータを取得
 */
func Content(c echo.Context) error {
	id := c.Param("id")

	// 指定されたデータを取得できたという体でレスポンスを返却する
	return c.String(http.StatusOK, "Contens, id="+id+"\n")
}

/**
 * 登録
 */
func Register(c echo.Context) error {
	log.Println("exec post::contents.Regsiter.")

	// 指定されたデータを登録できたという体でレスポンスを返却する
	return c.String(http.StatusOK, "Register OK\n")
}

/**
 * 指定された ID に紐づくデータを更新
 */
func Update(c echo.Context) error {
	id := c.Param("id")
	log.Println("exec put::contents.Update. id=" + id)

	// 指定されたデータを更新できたという体でレスポンスを返却する
	return c.String(http.StatusOK, "Update, id="+id+"\n")
}

/**
 * 指定された ID に紐づくデータを削除
 */
func Delete(c echo.Context) error {
	id := c.Param("id")
	log.Println("exec delete::contens.Delete. id=" + id)

	// 指定されたデータを削除できたという体でレスポンスを返却する
	return c.String(http.StatusOK, "Delete, id="+id+"\n")
}
```

# curl で試してみる

ルーティングした各処理を curl で実行してみます。
指定する URL はこちらです。( 下記は [ルーティングを管理する処理](#ルーティングを管理する処理) でコードを抜粋したものです )

```go:routing.goから抜粋
/**
 * API のルーティングを管理する
 */
func Routing(e *echo.Echo) {
	e.GET("/testapi/all", testapi.All)       // 全件取得
	e.GET("/testapi/:id", testapi.Content)   // ID を指定して取得
	e.POST("/testapi", testapi.Register)     // 登録
	e.PUT("/testapi/:id", testapi.Update)    // ID を指定して更新
	e.DELETE("/testapi/:id", testapi.Delete) // ID を指定して削除
}
```


**全件取得( `/testapi/all` )を試す**

```bash:/testapi/all
% curl http://localhost:8080/testapi/all
All Contents
```

**ID を指定して取得( `/testapi/:id` ) を試す**

```bash
% curl http://localhost:8080/testapi/asdasdasd # asdasdasd は任意の文字列
Contens, id=asdasdasd
```

**登録( `/testapi` ) を試す**

```bash
% curl -X POST -H "Content-Type: application/json" -d '{"content": "hjodfsdfsdf"}' localhost:8080/testapi
Register OK
```

**ID を指定して更新( `/testapi/:id` ) を試す**

```bash
% curl -X PUT -H "Content-Type: application/json" -d '{content: "hjodfsdfsdf"}' localhost:8080/testapi/asdasdasd
Update, id=asdasdasd
```

**ID を指定して削除( `/testapi/:id` ) を試す**

```bash
% curl -X DELETE -H "Content-Type: application/json" localhost:8080/testapi/dsdwewqas
Delete, id=dsdwewqas
```

# ソースコード

今回扱ったコードはこちらにアップしてあります。ご興味あればご覧ください。

- [server.go](https://github.com/ksh-fthr/react-and-echo-work/blob/main/restapi/api/server.go)
- [routing.go](https://github.com/ksh-fthr/react-and-echo-work/blob/main/restapi/api/router/routing.go)
- [testapi.go](https://github.com/ksh-fthr/react-and-echo-work/blob/main/restapi/api/modules/testapi/testapi.go)

# 参考

- 公式リファレンス
  - [Guide](https://echo.labstack.com/docs/category/guide)
  - [Routing](https://echo.labstack.com/docs/routing) 
