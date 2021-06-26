+++
title = "load test with dynamic payload by tsenart/vegeta"
date = 2021-03-11T03:08:11+09:00
tags = ["go", "vegeta"]
draft = false
+++

負荷テストツール`vegeta`のコマンドでの使い方ぐはググるとたくさん出てくるが、ライブラリとして仕様している記事が少なく手こずったのでメモ代わりに記事にする。


## やりたいこと

リクエスト毎にRequestの内容を動的に変えて、負荷をかけたかった。

- Headerに持つタイムスタンプがリクエスト時のタイムスタンプになるように。
- userIdはuniqueになるように。(同じユーザだと重複リクエストを弾く処理があったため。)

具体的には下記のような感じでリクエストを送りたい。

```json
{"header": {"timestamp":"1234"}, "body": {"userId": "lunar-1"}}
{"header": {"timestamp":"1234"}, "body": {"userId": "lunar-2"}}
{"header": {"timestamp":"1235"}, "body": {"userId": "lunar-3"}}
...
{"header": {"timestamp":"1240"}, "body": {"userId": "lunar-n"}}
```


vegeta.Targeterは `func(*vegeta.Target) error` である。  
また、vegeta.Attackは、vegeta.Targeterをgoroutineで都度呼び出して、Targeter内からTargetに基づきリクエストしていることから、都度Targetが変わるようなTargeterを返す関数を作成し、それをAttack時に呼び出せば良さそう。

ってことが、 [[QUESTION] Sending requests with dynamic body #330](https://github.com/tsenart/vegeta/issues/330#issuecomment-417230380)に書いてあって何度も行き来した結果やっと理解できた...。

上記を拝借して以下のようにすることで目的を達成できた。  


```go
	type yourRequestBody struct {
		UserID string
	}

	targeter := func(id uint64) vegeta.Targeter {
		return func(t *vegeta.Target) error {
			execTime := time.Now()
			requestTimestamp := execTime.Unix()

			req := &http.Request{
				Method: "POST",
				URL: &url.URL{
					Scheme: "http",
					Host:   "localhost",
					Path:   "api/v1/yourendpoint/",
				},
				Header: map[string][]string{
					"X-RequestTimestamp": []string{strconv.Itoa(int(requestTimestamp))},
				},
			}

			t.Header = req.Header
			t.Method = req.Method
			t.URL = req.URL.String()
			t.Body, _ = json.Marshal(
				&yourRequestBody{
					UserID: fmt.Sprintf("lunar-%d-%d", requestTimestamp, atomic.AddUint64(&id, 1)),
				},
			)
			return nil
		}
	}(0)


	rate := vegeta.Rate{Freq: 10, Per: time.Second}
	duration := 10 * time.Second
	var metrics vegeta.Metrics
	attacker := vegeta.NewAttacker()
	for res := range attacker.Attack(targeter, rate, duration, "Big Bang!") {
		metrics.Add(res)
	}
```


結構、苦戦した...。