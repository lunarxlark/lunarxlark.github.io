---
title : Custome Logger using LoggerWithConfig in Echo
tags : ["go", "echo"]
date : 2020-11-10T02:51:09+09:00
published : true
---

WebFramework `Echo`のLoggerを弄っていて気付いた点があったのでメモ。

[sample code](https://github.com/lunarxlark/sample-echo)

## 箇条書き

1. responseログにHeader `X-Any-Header`を出力したい場合、ログフォーマットに`"any_header":${header:x-any-header}`を指定すればいいよ
2. responseログにQueryParamerter `?anyparam=xxx`を出力したい場合、ログフォーマットに`"any_query":${query:anyparam}`を指定すればいいよ
3. デフォルトで出力されるカラム'id'はHeader `X-Request-Id` を指定すると出力されるよ
4. Skipperを設定することで、path毎にログ出力する/しない等を設定出来るよ

1, 2, 3はEchoのそういう機能だってことで特にありません。
4は結構便利だなと思いました。以下のサンプルコードでは、/healthcheckに来た場合とCI/CD等での自動テスト時にはresponseログを出力しないようにする設定例です。


## sample

```go
// Middleware
e.Use(middleware.LoggerWithConfig(customLogger()))

// Handler
e.GET("/greet", greet)
e.GET("/healthcheck", healthcheck)

var ResponseLogForamt = `{` +
	`"time":"${time_custom}",` +
	`"id":"${id}",` + `"remote_ip":"${remote_ip}",` +
	`"host":"${host}",` +
	`"method":"${method}",` +
	`"uri":"${uri}",` +
	`"user_agent":"${user_agent}",` +
	`"status":${status},` +
	`"error":"${error}",` +
	`"latency":${latency},` +
	`"latency_human":"${latency_human}",` +
	`"bytes_in":${bytes_in},` +
	`"bytes_out":${bytes_out},` +
	`"forwarded-for":"${header:x-forwarded-for}",` +
	`"same-as-id":${header:X-Request-Id},` +
	`"query":${query:lang}` +
	`}`

func customLogger() middleware.LoggerConfig {
	cl := middleware.DefaultLoggerConfig
	cl.Skipper = customeSkipper
	cl.Format = ResponseLogForamt
	cl.CustomTimeFormat = "2006/01/02 15:04:05.00000"

	return cl
}

func customeSkipper(c echo.Context) bool {
	if c.Path() == "/healthcheck" {
		return true
	}
	if os.Getenv("ENV") == "auto-test" {
		return true
	} else {
		return false
	}
}

func greet(c echo.Context) error {
	switch c.QueryParam("lang") {
	case "jp":
		return c.String(http.StatusOK, "こんにちは\n")
	case "en":
		return c.String(http.StatusOK, "Hello World\n")
	default:
		return c.String(http.StatusOK, "ウホホイ!!\n")
	}
}

func healthcheck(c echo.Context) error {
	return c.String(http.StatusOK, "I'm fine\n")
}
```

```bash
-- request
$ curl \
  -X GET \
  -H "X-Forwarded-For: achoo" \
  -H "X-Request-Id: requestid123" \
  "http://localhost/greet?lang=en"

-- response log
{
  "time":"2020/11/10 00:17:09.28534",
  "id":"requestid123",
  "remote_ip":"achoo",
  "host":"localhost",
  "method":"GET",
  "uri":"/greet?lang=en",
  "user_agent":"curl/7.64.1",
  "status":200,
  "error":"",
  "latency":18699,
  "latency_human":"18.699µs",
  "bytes_in":0,
  "bytes_out":12,
  "forwarded-for":"achoo",
  "same-as-id":requestid123,
  "query":en
}
```
