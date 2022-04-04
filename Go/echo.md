# Echo
## http 요청 및 응답
```go
//컨트롤러라 보면됨
package homeController

import (
	// "net/http"
	
	"github.com/labstack/echo/v4"
)

// 핸들러 메서드라 보면됨
func HandleHome(c echo.Context) error {
    // return c.Attachment("server-file-name.jpg", "client-file-name.jpg") // 파일 다운로드
    // return c.String(http.StatusOK, "Hello, World!") // 문자열
    // return c.JSON(http.StatusOK, 구조체) // json
    // return c.Redirect(http.StatusFound, "/") // 리다이렉트
    return c.File("home.html") // html 파일, 상대경로
}
```
```go
package main

import (
	// "net/http"
	"study/homeController"
	"github.com/labstack/echo/v4"
)

func main() {
	e := echo.New()
	e.GET("/", homeController.HandleHome)
    // e.POST("/", homeController.HandleHome)
    // e.PUT("/", homeController.HandleHome)
    // e.DELETE("/", homeController.HandleHome)
    // e.PATCH("/", homeController.HandleHome)
	e.Logger.Fatal(e.Start(":1323")) // 포트번호
}
```
* Echo는 go의 웹 프레임워크이다.
* 스프링의 컨트롤러에 해당한다고 보면된다.